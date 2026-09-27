---
description: "How to ask for the output shape you need — lists, tables, Markdown, fill-in templates, length targets — and how to verify format with a quick script."
---

# 07 · Asking for a Format

A correct answer in the wrong shape still costs you time: you reformat the table, delete
the preamble, trim the conclusion nobody asked for. Format instructions are among the
easiest wins in prompting because format is a surface pattern — exactly what models are
best at following.

## Say where the output will go

The best format instruction often names the destination:

- "…as a bulleted list I can paste into Slack" (short lines, no headings)
- "…as a Markdown table with columns Task, Owner, Due"
- "…as plain text for an SMS, under 160 characters"
- "…as CSV with a header row, no commentary before or after"

The destination implies many small conventions you would otherwise list one by one.

## Templates: show the exact skeleton

When the shape matters, give the skeleton and ask the model to fill it:

```text
Summarize the incident report in <report> using exactly this template:

**What happened:** <one sentence>
**Impact:** <who was affected and for how long>
**Root cause:** <one sentence, or "Unknown">
**Follow-ups:**
- <action> — <owner>

<report>
...
</report>
```

Fill-in templates remove almost all ambiguity. Include placeholder guidance inside the
angle brackets, and state what to write when information is missing ("Unknown") so the
model doesn't invent it.

## Controlling length

Length instructions work best when they are concrete and structural:

- Words or sentences: "2–3 sentences", "about 150 words".
- Structure: "5 bullets, each under 12 words".
- Relative to input: "about one-tenth the length of the original".

Models follow ranges more reliably than exact counts. If you need a hard limit
(a character-limited field), check it after generation — by eye, or with a line of code.

## Removing the chatter

Chat assistants often add a friendly preamble ("Sure! Here's…") and a closing offer
("Let me know if…"). When you need clean output:

```text
Output only the table. Do not add any text before or after it.
```

Starting the answer for the model also helps, where your interface allows it. In an
API that accepts a partial assistant message (sometimes called prefilling), beginning the
reply with the first characters of the format — like `|` for a table or `{` for JSON —
strongly anchors the shape. Not every provider or interface supports this.

## Markdown or not?

Many chat interfaces render Markdown, so assistants use it by default: headings, bold,
bullets. That's great in a chat window and noise in an email, a spreadsheet cell, or a
text field in your app. Be explicit: "plain text, no Markdown formatting".

## Worked example: a meeting summary that pastes cleanly

```text
Below are raw notes from our weekly team meeting.

Produce a summary to paste into our team chat. Format:
- First line: "Weekly sync — " followed by the date in the notes.
- Then a line "Decisions:" and one bullet (using "•") per decision.
- Then "Action items:" and one bullet per item in the form "• Name: task (due date)".
  If no due date is stated, write "(no date)".
- Plain text only: no Markdown headings, no bold, no text after the last bullet.

<notes>
...
</notes>
```

Each rule maps to something that would otherwise be inconsistent: the date line, the
bullet character, the "no date" case, the trailing chatter.

## Checking format with a few lines of Python

When the output feeds a program, check the format rather than hoping. This snippet
validates the action-item lines from the example above:

```python
import re

ACTION = re.compile(r"^• [A-Z][\w .'-]*: .+ \((\d{4}-\d{2}-\d{2}|no date)\)$")

def check_actions(text: str) -> list[str]:
    """Return the action-item lines that don't match the required format."""
    lines = text.splitlines()
    try:
        start = lines.index("Action items:") + 1
    except ValueError:
        return ["missing 'Action items:' line"]
    return [ln for ln in lines[start:] if ln.strip() and not ACTION.match(ln)]

sample = """Weekly sync — 2026-03-02
Decisions:
• Move launch to April
Action items:
• Ana: draft release notes (2026-03-06)
• Ben: check pricing page (no date)
• update the FAQ"""

print(check_actions(sample))
```

Running it prints:

```text
['• update the FAQ']
```

The last line has no owner or date, so it's flagged. The same idea — a small, automatic
format check — becomes a key part of evaluation in Level 3.

## How It Actually Works

Formats are sequences of highly predictable tokens: after `| Task | Owner |` a table
separator row is extremely likely; after `•` and a name, a colon is likely if every
earlier bullet had one. Once the model has produced the first element of a structure,
continuing it is easy — the hard part is getting it started in the right shape. That is
why templates, a pre-started answer, and a clear final instruction are so effective:
they fix the first few tokens, and the pattern carries the rest.

Length is harder because the model does not have a precise running count of words.
It has learned rough associations between phrases like "in two sentences" and the shape
of short answers, and it can see structural boundaries (bullets, paragraphs) as it writes
them. Structural limits ("3 bullets") are therefore more reliable than numeric word
limits.

## Common mistakes

- **Describing a format in prose** when a skeleton would be clearer.
- **No instruction for missing information**, so the template gets filled with guesses.
- **Asking for "just the answer"** but pasting examples that include explanations.
- **Assuming Markdown renders** everywhere the output will be pasted.
- **Trusting exact counts** without checking.

## Exercise

1. Choose an output you regularly reformat by hand (a summary, a list, a table).
2. Write a template prompt with an explicit skeleton, a missing-information rule, and a
   "no text before or after" line.
3. Run it on three different inputs.
4. Optional: write a small check (regex or a few lines of Python) that flags outputs
   breaking your format, and run it on the three outputs.
