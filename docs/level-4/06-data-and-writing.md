---
description: "Domain prompting for data analysis (compute in code, not in prose) and for writing (outlines, drafts, editing passes, keeping the author's voice)."
---

# 06 · Prompting for Data Analysis & Writing

Two of the most common professional uses of language models sit at opposite ends. Data
analysis needs exactness: a wrong number is simply wrong. Writing needs judgement and
voice: there's no single right answer, but there are many generic ones. Each domain has
its own traps and its own techniques.

## Part 1 — Data analysis

### The core rule: compute in code, reason in words

Models are fluent with numbers in text but unreliable at arithmetic over many values —
summing a column of 200 figures "by reading" is exactly the kind of task that produces a
confident wrong answer. The robust pattern:

- Let the model **plan the analysis and write code** (SQL, Python/pandas, spreadsheet
  formulas).
- **Run the code** (yourself, or via a code-execution tool the assistant provides).
- Let the model **interpret the computed results**, which you pass back.

```text
I have a CSV with columns: order_date (YYYY-MM-DD), region, product, units, unit_price.
About 50,000 rows. I want to know whether the average order value changed after the
price increase on 2026-04-01, overall and per region.

1. List the assumptions and possible pitfalls (e.g. seasonality, mix changes) before
   writing code.
2. Write pandas code to compute: AOV before vs after, per region, with order counts.
   Don't compute any numbers yourself; I'll run the code and paste the results.
```

### Give the schema, not the whole dataset

Paste column names, types, a few example rows, and what each column means. That's usually
enough for correct code, avoids context bloat, and avoids sending sensitive data to the
model unnecessarily.

### Interpretation prompts that stay honest

```text
Here are the computed results in <results>. Write three findings for a regional sales
manager. For each: the number(s) it's based on (quoted exactly from <results>), what it
suggests, and one alternative explanation that the data can't rule out.
Don't claim causation; this is observational data.
```

Asking for alternative explanations counteracts a model's tendency to produce a tidy
story, and quoting numbers keeps interpretation tied to the actual output.

### Checking a model's numbers

Whenever a model states numbers derived from data, verify them against computed output.
A tiny example of the habit:

```python
rows = [("North", 3, 120.0), ("South", 5, 80.0), ("North", 2, 45.0), ("East", 4, 60.0)]
revenue = {}
for region, units, price in rows:
    revenue[region] = revenue.get(region, 0) + units * price

model_claim = {"North": 450.0, "South": 400.0, "East": 240.0}   # numbers stated in a draft
for region, value in revenue.items():
    status = "ok" if abs(value - model_claim[region]) < 0.01 else f"WRONG (actual {value})"
    print(region, status)
```

Output:

```text
North ok
South ok
East ok
```

Here the stated figures are correct — the point is that checking took three lines and
removed the doubt.

## Part 2 — Writing

### Separate the stages

Asking for a finished piece in one prompt tends to produce competent, generic prose.
Professional writing happens in stages, and prompts work better that way too:

1. **Brief:** purpose, reader, key message, what the reader should do next.
2. **Outline:** ask for 2–3 alternative structures; pick or merge.
3. **Draft:** section by section, supplying your own facts, examples and opinions.
4. **Edit passes**, each with one focus: structure; clarity; cutting; tone; facts.

### Bring the substance

The model can't supply your experience, data, or point of view. The more specific
material you give — anecdotes, figures, the argument you actually want to make — the
less the draft falls back on generic filler.

```text
Draft the section "Why we moved off nightly batch jobs" (250–300 words) for our
engineering blog. Use my notes in <notes> for all facts and examples; don't add
technical claims that aren't in the notes. Voice: first person plural, dry humour,
short paragraphs. Open with the 3 a.m. incident described in the notes.
```

### Editing prompts that respect the author

```text
Edit the draft in <draft> for clarity only. Keep my voice, my examples and my structure.
Don't add new points. Return the edited text, then a list of every change that altered
meaning (should be none) and every sentence you cut, so I can restore it.
```

Diffing the result (Level 2 lesson 09) shows exactly what changed.

### Feedback instead of rewriting

Often the most valuable use isn't generating text but critiquing it:

```text
Read my draft as a skeptical reader from <audience>. Where would you stop reading? Which
claims would you not believe without evidence? What question would you still have at
the end? Don't rewrite anything.
```

### Honesty and disclosure

When model-assisted writing is published, follow your organization's and publication's
policies on disclosure, check every factual claim, and make sure you can stand behind
every sentence as your own.

## How It Actually Works

Arithmetic over many values requires exact, step-by-step manipulation of digits spread
across many tokens — something next-token prediction does unreliably at scale, even when
it handles small calculations well. Code delegates that to a deterministic interpreter
and leaves the model the parts it's good at: translating a question into an analysis
plan, and translating results back into language.

In writing, the model's default output is the statistical centre of similar texts it has
seen — which is, by definition, generic. Staged prompting and specific material move the
output away from that centre: an outline you chose, facts only you have, and a voice
sample each add constraints the average text doesn't satisfy.

## Common mistakes

- **Asking a model to compute totals or averages** directly from pasted data.
- **Pasting whole datasets** when the schema and a sample would do.
- **Accepting causal stories** from observational data.
- **One-shot "write the article"** prompts with no material from you.
- **Edits that silently change meaning**, unreviewed.

## Exercise

1. Data: take a real CSV (or a public dataset). Write an analysis-plan-plus-code prompt,
   run the code yourself, then write an interpretation prompt with alternative
   explanations. Verify every number in the final text.
2. Writing: take a piece you need to write. Do the four stages — brief, outline, draft
   section, two single-focus edit passes — and diff each edit.
3. Compare with a single "write it" prompt. Which would you actually publish?
