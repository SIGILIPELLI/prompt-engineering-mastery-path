---
description: "Getting reliable JSON from language models: schema-in-prompt, native structured-output modes, validation, repair loops, and designing model-friendly schemas."
---

# 03 · Structured Output: JSON & Schemas

The moment a prompt's output is read by code rather than a person, "mostly right" is not
good enough. A missing brace or an extra sentence before the JSON crashes the parser.
This lesson covers how to prompt for structured output, how provider features help, and
why you must validate regardless.

## Three levels of enforcement

1. **Prompt-only:** describe the structure in the prompt, with an example. Works with any
   model; well-formed output is likely but not guaranteed.
2. **JSON mode:** many APIs offer a setting that forces syntactically valid JSON. The
   *shape* (which keys, which types) is still up to the prompt.
3. **Schema-constrained output:** several providers and open-source inference engines can
   constrain generation to a JSON Schema you supply, so output matches the schema's
   structure. Supported schema features vary by provider — check the docs.

Even at level 3, values can be *wrong*: a schema can force `"priority"` to be one of
`low|medium|high`, but not that the model picked the right one. Validation of content is
still your job.

## Prompting for JSON

```text
Extract the event details from the message in <message>.

Return a single JSON object with exactly these keys:
{
  "title": string,
  "date": string in YYYY-MM-DD format, or null if not stated,
  "start_time": string in 24-hour HH:MM, or null,
  "location": string, or null,
  "attendees": array of first names (may be empty)
}
Output only the JSON object. No code fences, no commentary.

<message>
Hey all — design review moved to Thursday 14 May, 3pm, room 4B. Ana and Raj please come.
</message>
```

Key moves: every key is listed with its type and format; missing values have a defined
representation (`null`, empty array); and the output is restricted to the object itself.
Note that the message gives no year — a good prompt would supply today's date in context
or accept `null`; otherwise the model must guess the year.

## Designing model-friendly schemas

- **Flat beats deeply nested.** Fewer levels, fewer places to go wrong.
- **Enums for categories.** `"sentiment": "positive" | "negative" | "mixed"` rather than
  free text.
- **Explicit nulls.** Decide how "not found" is represented and say it.
- **Descriptive key names.** `due_date_iso` tells the model more than `d`.
- **Order keys for reasoning.** If you want a short justification, put a `"reason"` key
  *before* the `"label"` key, so the model writes the reasoning first (lesson 01).
- **Avoid asking for computed values** the model is bad at (sums over many items); compute
  them in code from extracted fields.

## Validate and repair

Always parse and validate. On failure, a common pattern is one repair attempt: send the
error back and ask for a corrected object.

```python
import json

REQUIRED = {"title": str, "date": (str, type(None)), "attendees": list}

def validate(raw: str) -> tuple[dict | None, str | None]:
    """Parse raw model output and check required keys/types. Return (obj, error)."""
    text = raw.strip()
    if text.startswith("```"):                     # tolerate code fences
        text = text.strip("`").removeprefix("json").strip()
    try:
        obj = json.loads(text)
    except json.JSONDecodeError as e:
        return None, f"invalid JSON: {e.msg} at position {e.pos}"
    if not isinstance(obj, dict):
        return None, "top level must be an object"
    for key, typ in REQUIRED.items():
        if key not in obj:
            return None, f"missing key: {key}"
        if not isinstance(obj[key], typ):
            return None, f"wrong type for {key}"
    return obj, None

outputs = [
    '{"title": "Design review", "date": null, "attendees": ["Ana", "Raj"]}',
    'Sure! Here is the JSON: {"title": "Design review"}',
    '```json\n{"title": "Design review", "date": "2026-05-14", "attendees": "Ana"}\n```',
]
for raw in outputs:
    print(validate(raw))
```

Output:

```text
({'title': 'Design review', 'date': None, 'attendees': ['Ana', 'Raj']}, None)
(None, 'invalid JSON: Expecting value at position 0')
(None, 'wrong type for attendees')
```

A repair prompt for the failed cases might be:

```text
Your previous output could not be used: {error}.
Return the corrected JSON object only, following the same key list.
```

Limit repairs to one or two attempts; if it still fails, log it and fall back (skip the
item, flag it for a human). In production, a library such as Pydantic (Python) or a JSON
Schema validator gives you richer checks than the hand-rolled one above.

## Worked example: a schema that encourages careful labels

```text
Classify the review in <review>. Return JSON:
{
  "evidence": "short quote(s) from the review that support your label",
  "aspects": array of zero or more of ["price", "quality", "delivery", "support"],
  "sentiment": "positive" | "negative" | "mixed"
}
```

Because `evidence` comes first, the model commits to specific text before choosing a
label. Because `aspects` is a closed list, you can aggregate results across thousands of
reviews without cleaning free text.

## How It Actually Works

Without constraints, JSON is just another text pattern the model has seen countless times
in training, so it's usually produced well — but nothing prevents a stray token such as a
friendly preamble or a trailing comment. **Constrained decoding** fixes this at generation
time: at each step, the inference engine masks out every token that would make the output
invalid according to a grammar derived from your schema, so the model can only choose
among valid continuations. That guarantees the structure but not the semantic
correctness of values — the model still picks the most likely valid token, which may be
the wrong enum value.

Key order matters because generation is left to right. Fields generated early become
context for fields generated later; putting evidence or reasoning before a decision field
lets the decision condition on it.

## Common mistakes

- **Parsing without validating** types and required keys.
- **No representation for missing data**, so the model invents values.
- **Free-text categories** where an enum would do.
- **Assuming constrained output means correct output.**
- **Infinite repair loops** instead of a bounded retry and a fallback.

## Exercise

1. Choose a text source you care about (job ads, recipes, receipts, meeting invites).
2. Design a flat JSON schema with at least one enum and one nullable field.
3. Write the prompt and run it on 5 varied inputs, including one where most fields are
   missing.
4. Adapt the `validate` function above to your schema and run all 5 outputs through it.
5. If any fail, write a repair prompt and test whether one retry fixes them.
