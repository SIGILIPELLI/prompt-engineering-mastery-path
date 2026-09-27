---
description: "The six building blocks of an effective prompt — task, context, input, constraints, output format, examples — and how to order and label them."
---

# 02 · Anatomy of a Good Prompt

Most weak prompts are not badly worded; they are *missing parts*. Once you know the
parts, you can look at any prompt and spot the gap in seconds. This lesson names six
building blocks. Not every prompt needs all six, but every prompt should leave out a
block on purpose, not by accident.

## The six blocks

| Block | Question it answers | Example line |
|-------|--------------------|--------------|
| **Task** | What should the model do? | "Draft a reply to the customer email below." |
| **Context** | Why, for whom, in what situation? | "We are a 5-person bookkeeping firm; the customer is a long-time client." |
| **Input** | What material should it work on? | The email itself, clearly delimited. |
| **Constraints** | What must it do or avoid? | "Do not promise a refund; do not mention pricing." |
| **Output format** | What shape should the answer take? | "Plain text, under 150 words, no subject line." |
| **Examples** | What does good look like? | One past reply you liked. |

The **task** is the only mandatory block. Everything else exists to remove guesswork.

## Putting them together

A before/after for a common request:

```text
Before:
reply to this email

Hi, I sent my documents two weeks ago and haven't heard anything. Is my tax return
done? I need it for a mortgage application. - Priya
```

```text
After:
Task: Draft a reply to the client email below.

Context:
- We are a small bookkeeping and tax firm. Priya is a long-standing client.
- Her return is in progress; we are waiting on one missing document from her:
  the 2025 interest statement from her savings bank.
- Typical turnaround once we have everything is 5 business days.

Constraints:
- Be warm and brief; acknowledge the delay without over-apologising.
- Ask clearly for the missing document.
- Do not promise a specific completion date.

Output format: plain text email body only, under 120 words, signed "Sam".

<email>
Hi, I sent my documents two weeks ago and haven't heard anything. Is my tax return
done? I need it for a mortgage application. - Priya
</email>
```

With the first version, the model has to invent the status of the return. A typical
response either vaguely reassures ("we're working on it!") or makes up a date. The
second version gives it the one fact that matters — the missing document — so the
useful reply becomes the most likely reply.

## Ordering the blocks

There is no single correct order, but some patterns hold up well across models:

- **Put the task early**, so everything after it is read in light of what you want.
  For very long inputs, it also helps to **restate the task at the end**, right before
  the model starts writing (see Level 3 lesson 07).
- **Keep instructions and material separate.** Use labels (`Context:`), Markdown
  headings, or XML-style tags (`<email> … </email>`) so the model can tell "things to
  do" from "things to work on". Tags are especially useful when the input itself
  contains instructions-like text ("Please reply by Friday").
- **Put format last** among the instructions; it is the thing the model acts on as it
  begins writing.

## Labels and delimiters

Any consistent delimiter works: triple quotes, `---` lines, Markdown headings, or tags.
Tags have two advantages: they have an explicit end (`</email>`), and you can refer to
them by name ("using only the facts in `<notes>`…").

```text
Summarize the meeting notes in <notes> for someone who missed the meeting.
List decisions first, then open questions.

<notes>
...pasted notes...
</notes>
```

## Worked example: diagnosing a prompt by its missing blocks

```text
Make this better:
"Our app lets you track habits. It has streaks and reminders. Download now."
```

- Task: "make better" — better *how*? More persuasive? Shorter? More formal?
- Context: missing — where does this text appear? An app store listing? A tweet?
- Constraints: missing — can it add features? Change the call to action?
- Format: missing — one version or several? How long?

A repaired version:

```text
Rewrite the app store short description below to be more specific and appealing to
people who have tried habit apps before and given up.

Constraints: max 80 characters; mention only features listed; keep a call to action.
Give 3 alternatives as a numbered list.

<description>Our app lets you track habits. It has streaks and reminders. Download now.</description>
```

## How It Actually Works

Why do labels and structure help? Instruction-tuned models were trained on huge numbers
of documents and conversations where structure carries meaning: headings introduce
sections, quoted material is distinct from commentary, lists enumerate parallel items.
When your prompt uses the same conventions, the model can lean on those learned
associations to decide which tokens are instructions and which are data.

Explicit blocks also reduce the number of things the model must *infer*. Every inference
is a place where the model falls back on the most generic plausible choice from its
training — the average newsletter, the average email. Filling a block replaces an
average guess with your specific answer. That is the whole trick: specifics in, specifics
out.

Finally, the end of the prompt sits right next to where generation begins, so the model
attends to it strongly. That is why format instructions and a restated task often work
best near the end, especially after long pasted material.

## Common mistakes

- **All task, no context.** "Write a cover letter" with no job, no candidate, no tone.
- **Instructions mixed into the material.** Pasting a document and typing instructions
  halfway through it; the model may treat your instruction as part of the document.
- **Contradictory blocks.** "Be detailed" in the task and "under 50 words" in the format.
  The model will satisfy one and violate the other unpredictably.
- **Unlabelled multiple inputs.** Pasting two documents back to back and asking to
  "compare them" without marking where one ends.
- **Over-stuffing.** Adding every possible block to a trivial request. A quick question
  deserves a quick prompt.

## Exercise

Pick three prompts you use regularly (or invent three realistic ones: an email reply,
a summary, a piece of code). For each:

1. Label which of the six blocks it currently contains.
2. Decide which missing blocks would actually change the result, and add only those.
3. Wrap any pasted material in named tags.
4. Run old and new versions twice each and note the differences in your prompt notebook.
