---
description: "Level 1 project: engineer a reusable prompt for a real recurring task, with a test set, versioned iterations, and a documented before/after."
---

# 10 · Project — A Reusable Prompt, Before & After

This project pulls Level 1 together. You will take one real task you do repeatedly and
engineer a prompt for it that you can reuse for months — with evidence that it is better
than what you started with. The walkthrough below uses a sample task so you can see every
step; then you repeat the process on your own.

## What you'll produce

1. A **task card**: what the prompt is for, who reads the output, what "good" means.
2. A **test set** of 8 inputs.
3. A **before** prompt (how you'd have written it last month) and its results.
4. At least **three logged iterations**.
5. An **after** prompt, saved as a reusable template with a `{placeholder}` for the input.
6. A short **before/after write-up**: what changed and why it works.

## Sample task: turning messy notes into a client follow-up email

A freelance designer has calls with clients and scribbles notes. After each call, they
send a follow-up email confirming what was agreed. It takes 15 minutes each time.

### Step 1 — the task card

```text
Task: Turn my raw call notes into a follow-up email to the client.
Reader: the client (non-designer, busy, reads on phone).
Good means:
  - confirms every decision in the notes, nothing invented
  - lists what I will deliver and when, and anything I need from them
  - friendly, professional, under 180 words
  - plain text (my email client doesn't render Markdown)
Bad means: invented dates/prices, missing a request, sounding like a template.
```

### Step 2 — the test set

Eight sets of notes, deliberately varied:

| # | Case |
|---|------|
| 1 | Clean, complete notes |
| 2 | Very short notes (3 lines) |
| 3 | Notes with a price mentioned |
| 4 | Notes with no deadlines stated |
| 5 | Notes where the client asked for something I declined |
| 6 | Notes with a typo'd name and abbreviations |
| 7 | Notes that include a personal aside that shouldn't be in the email |
| 8 | Notes where I need two things from the client |

Example — test input #4:

```text
call w/ Marta (Riverside Bakery) - logo refresh
- likes option B, wants warmer colours
- keep the wheat icon
- she'll send photos of the shopfront
- I do 2 revised versions
```

### Step 3 — the "before" prompt

```text
Write a follow-up email from these notes:
{notes}
```

Run it on all 8. Typical problems to look for (record what *you* actually see):

- Deadlines or meeting dates appearing that aren't in the notes (especially on #4).
- Markdown bullets and bold text.
- Generic sign-offs and a subject line you didn't ask for.
- The personal aside in #7 leaking into the email.
- The declined request in #5 phrased as if it were agreed.

Score each output against the task card: pass or fail, with a one-line reason.

### Step 4 — iterate (one change at a time)

**v2 — grounding and escape hatch.** Most common failure: invented details.

```text
Use only information in the notes. If a deadline isn't stated, don't give one;
instead ask the client to confirm timing.
```

**v3 — format.** Most common remaining failure: Markdown and subject line.

```text
Output only the email body in plain text: no subject line, no Markdown.
Use short paragraphs; if listing deliverables, use lines starting with "- ".
```

**v4 — structure and exclusions.** Remaining failures: #5 and #7.

```text
Structure: 1) thanks + one-line recap, 2) what we agreed, 3) what I'll deliver,
4) what I need from you, 5) sign-off "Best, Jo".
If the notes say I declined something, mention it briefly and politely.
Leave out anything personal or unrelated to the project.
```

Log each version and its score in your notebook.

### Step 5 — the "after" prompt

```text
Write a follow-up email to a client based on my raw call notes in <notes>.

About me: I'm Jo, a freelance graphic designer. The client is not a designer and will
likely read this on a phone.

Rules:
- Use only information in the notes. Never invent dates, prices, or deliverables.
  If timing isn't stated, ask the client to confirm it.
- If the notes say I declined a request, mention it briefly and politely.
- Leave out anything personal or unrelated to the project.
- Friendly and professional; under 180 words.

Structure:
1. Thanks and a one-line recap of the call.
2. What we agreed.
3. What I'll deliver (lines starting with "- ").
4. What I need from the client (lines starting with "- "), if anything.
5. Sign-off: "Best, Jo"

Output only the email body in plain text — no subject line, no Markdown.

<notes>
{notes}
</notes>
```

### Step 6 — the before/after write-up

Keep it short and evidence-based. For example:

```text
Before: 2/8 passed. Main failures: invented deadlines (4 cases), Markdown (8),
personal aside leaked (#7).
After (v4): 7/8 passed across two runs each. Remaining: #2 is sometimes too thin to
be useful — acceptable; the fix is better notes, not a better prompt.
Biggest single improvement: the grounding rule with an explicit "ask to confirm timing"
alternative.
```

Your numbers will differ — use the ones you actually observe.

## How It Actually Works

The finished prompt works because each part removes a specific degree of freedom that the
model previously filled with a generic guess:

- The **about-me/reader** context moves tone from "average business email" to "designer to
  non-designer on a phone".
- The **grounding rule with an alternative** changes the most likely continuation when a
  fact is missing from "invent one" to "ask".
- The **numbered structure** fixes the order of content, which in turn makes it easy for
  the model to track whether it has covered everything — each section is a visible
  checkpoint in its own output.
- The **format line at the end**, right before the notes, sits close to where generation
  starts, which is where format instructions are most reliably followed.

The test set is what makes this engineering rather than tinkering: without it, you'd
have no way to know whether v4 was better than v1 or just different.

## Common mistakes

- **Choosing a one-off task.** Pick something you'll repeat; reuse is the point.
- **A test set of easy cases only.** The awkward cases are where prompts fail.
- **Judging by the best output** instead of the pass rate across all inputs.
- **Hard-coding one input's details** into the template during tuning.
- **Skipping the write-up.** Future-you won't remember why a rule is there.

## Exercise

Complete the project on your own recurring task:

1. Write the task card.
2. Build a test set of 8 realistic inputs, at least 3 of them awkward.
3. Record the before prompt and its pass rate.
4. Iterate at least three times, one change each, logging results.
5. Save the final template with a `{placeholder}` and write the before/after summary.
6. Share the template with a colleague and ask them to run it on one input of theirs.
   Did it work for someone else's input? That's the real test of reusability.
