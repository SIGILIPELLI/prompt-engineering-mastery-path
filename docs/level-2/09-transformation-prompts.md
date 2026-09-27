---
description: "Prompts that change form without changing meaning: rewriting, simplifying, translating, and converting formats, with invariants and diff-based checks."
---

# 09 · Transformation Prompts: Rewrite, Translate, Convert

A transformation task takes an input and changes its *form* while keeping its *meaning*:
rewrite for a new audience, fix grammar, translate, turn prose into a table, convert a
CSV to JSON, convert passive to active voice. These are some of the tasks models are best
at — and the risk is always the same: meaning quietly changes along with form.

## Name the invariants

The most important line in a transformation prompt says what must **not** change:

```text
Rewrite the paragraph in <text> so it's readable by customers with no technical
background.

Must stay exactly the same: all numbers, dates, product names, and the order of the
steps.
Must stay equivalent: every obligation and limitation (for example "must", "only if",
"up to").
May change: vocabulary, sentence structure, and length (within ±30%).
```

Listing invariants explicitly turns an open-ended rewrite into a constrained one, and
gives you a checklist for verification.

## Light-touch vs heavy rewrites

Say how much change you want. "Proofread" and "rewrite" sit at opposite ends:

| Level | Instruction |
|-------|-------------|
| Proofread | "Fix only spelling, grammar and punctuation. Don't change wording otherwise." |
| Copy-edit | "Also fix unclear sentences and inconsistent terms; keep the author's voice." |
| Rewrite | "Restructure freely for clarity; keep all facts and claims." |

Without this, models tend to over-edit — "improving" wording you liked.

## Translation

Models are strong translators for widely used languages and weaker for less-resourced
ones. Prompts help most with:

- **Locale and register:** "Brazilian Portuguese, informal 'você', for a consumer app."
- **Glossaries:** "Translate 'workspace' as 'espace de travail'; keep 'Dashboard' in
  English."
- **Do-not-translate lists:** product names, code, placeholders like `{name}`.
- **Format preservation:** "Keep Markdown formatting and line breaks exactly."

For anything published or legally meaningful, have a fluent human review it. A useful
cheap check is back-translation (translate the result back and compare meaning), though
it can miss errors that happen to round-trip.

## Format conversion

Prose → table, CSV → JSON, bullet notes → formal minutes. For conversions a program can
do (CSV → JSON), prefer code — it's exact and free. Use a model when the source is messy
or semi-structured:

```text
Convert the free-text shipping notes in <notes> into CSV with the header:
order_id,carrier,tracking_number,ship_date
Use ISO dates. Leave a field empty if not stated. One row per order; output only CSV.
```

## Checking a transformation with a diff

For proofreading and light edits, a word-level diff shows exactly what changed — so you
can review changes instead of rereading the whole text:

```python
import difflib

original = "The refund will be issued within 14 days, only if the item is unused."
edited   = "The refund will be issued within 14 days if the item is unused."

a, b = original.split(), edited.split()
for op in difflib.SequenceMatcher(None, a, b).get_opcodes():
    tag, i1, i2, j1, j2 = op
    if tag != "equal":
        print(tag, a[i1:i2], "->", b[j1:j2])
```

Output:

```text
replace ['days,', 'only'] -> ['days']
```

A "proofread" silently dropped *only* — changing a strict condition into a looser one.
This is exactly the kind of meaning change a diff catches in seconds and a quick
reread misses.

You can automate invariant checks too: extract all numbers from input and output with a
regular expression and compare the sets.

## Worked example: policy to plain language

```text
Rewrite the HR policy excerpt in <policy> as a short FAQ for employees.

Invariants:
- Keep every number, deadline and eligibility condition exactly.
- Keep every "must"/"must not" as a clear requirement.
Format: 4–6 questions a new employee would ask, each answered in 1–3 sentences.
After the FAQ, list any policy sentence you could not fit into the FAQ, verbatim,
under "Not covered above:".
```

The final instruction is a safety valve: rather than silently dropping hard-to-phrase
content, the model is asked to surface it.

## How It Actually Works

Transformation tasks play to the model's strengths because most of the output is
strongly determined by the input: the model is largely copying meaning while re-sampling
form. Attention can align each output segment with the corresponding input segment,
similar to how translation systems align sentences.

The danger comes from the "re-sampling form" part. When the model rewrites a phrase, it
generates a fluent alternative, and fluency is judged by general language patterns, not
by strict equivalence to the source. Small function words — *only*, *not*, *up to*,
*unless* — carry large meaning but little "fluency weight", so dropping them rarely makes
the output sound wrong. That's why invariants must be named and why diffs and number
checks are so effective as verification.

## Common mistakes

- **Not specifying the edit depth** (proofread vs rewrite).
- **No invariants**, especially for numbers and conditions.
- **Using a model for exact mechanical conversions** code could do perfectly.
- **Translating placeholders and code** that must stay unchanged.
- **Reviewing by rereading** rather than by diffing.

## Exercise

1. Take a paragraph with at least two numbers and one condition ("only if", "unless").
2. Ask a model to "make it clearer" with no invariants, then again with explicit
   invariants. Diff both against the original.
3. Write a small script that extracts all numbers from input and output and reports any
   difference; run it on both versions.
4. If you speak two languages, translate a short passage with and without a glossary and
   judge which is better.
