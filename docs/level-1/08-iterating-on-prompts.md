---
description: "A disciplined loop for improving prompts: small test sets, one change at a time, diagnosing failures, and keeping a prompt notebook of versions."
---

# 08 · Iterating on a Prompt

Nobody writes a great prompt on the first attempt. The difference between people who get
consistently good results and people who don't is rarely talent — it's that the first
group iterates deliberately while the second group changes five things at once, tries it
once, and decides based on a feeling.

## The loop

```text
  write v1 ──► run on a few test inputs ──► look at every output
     ▲                                             │
     │                                             ▼
  change ONE thing  ◄── name the most common failure
```

1. **Collect a few test inputs** before you tune anything — typically 5–10 for a personal
   task. Include easy, typical, and awkward cases.
2. **Run the prompt on all of them.** Look at every output, not just the first.
3. **Name the most common failure** in a sentence: "It invents due dates", "Summaries are
   twice as long as asked".
4. **Change one thing** aimed at that failure.
5. **Re-run on the same inputs**, and compare.

## Why one change at a time

If you add a role, an example, and a length limit together and the output improves, you
don't know which change helped — or whether one of them made things worse and was masked
by the others. Changes also interact: adding a long example can quietly undo an earlier
length instruction. One change per version keeps cause and effect visible.

## Diagnose before you fix

Match the failure to the kind of fix:

| Symptom | Likely cause | Try |
|---------|--------------|-----|
| Generic, could be about anything | Missing context | Add audience, purpose, facts (lesson 04) |
| Right content, wrong shape | Format underspecified | Template or example (lessons 06–07) |
| Invents details | No grounding / no escape hatch | "Use only…; if missing say…" |
| Ignores a rule | Rule buried or conflicting | Move it later, remove conflicts, make it concrete |
| Inconsistent across runs | Ambiguous task | Decide the rule; add an example of the ambiguous case |
| Too long | Vague length, polite padding | Structural limit; "no preamble" |

## The prompt notebook

Keep a plain text file (or a spreadsheet) per recurring task:

```text
## Task: weekly status summary

### v1 (2026-03-02)
<prompt text>
Tested on: notes-A, notes-B, notes-C, notes-D, notes-E
Result: 3/5 ok. Fails: invents owners when notes don't name one (B, D).

### v2 (2026-03-02)
Change: added "If no owner is named, write 'Owner: unassigned'."
Result: 5/5 owners correct. New issue: C now has a preamble line.

### v3 ...
```

This takes a minute per version and saves you from re-discovering the same fixes. It also
becomes the seed of the test set you'll formalize in Level 3.

## Knowing when to stop

Stop iterating when the prompt meets your success criteria on your test inputs across a
couple of runs each, and remaining failures are rare and cheap to fix by hand. Chasing
the last few percent on a personal task is usually not worth it. For prompts inside
applications, the bar is higher and the evaluation more formal — that is Level 3.

Also stop and rethink when fixes keep piling up. If you've added fifteen special-case
rules, the task may be too big for one prompt (Level 2 lesson 02 covers splitting it) or
may need different information rather than different wording.

## Worked example: three iterations

Task: turn customer feedback emails into a one-line summary plus a sentiment label.

**v1**

```text
Summarize this feedback and say if it's positive or negative.
{email}
```

Run on 6 emails. Failures: outputs vary between one sentence and a paragraph; mixed
feedback is forced into positive or negative; two outputs start with "Sure!".

Most common failure: inconsistent shape. **v2** changes only format:

```text
Summarize the customer feedback in <email>.
Output exactly two lines:
Summary: <one sentence, max 20 words>
Sentiment: <positive | negative>

<email>{email}</email>
```

Re-run: shape is now consistent on all 6. Remaining failure: mixed emails (2 of 6) get
labels that flip between runs. **v3** changes only the label set:

```text
Sentiment: <positive | negative | mixed>
Use "mixed" when the email contains both clear praise and a clear complaint.
```

Re-run: mixed emails are labelled `mixed` consistently. Stop — the remaining variation is
in summary wording, which is fine.

## How It Actually Works

Iteration works because prompt changes act on the model's output *distribution*, not on
a single output. A single run is one sample from that distribution; several runs over
several inputs give you a rough picture of it. When you change one thing and re-run on
the same inputs, you're approximating a controlled experiment: same inputs, one variable.

It also works because failures cluster. A model's mistakes on a task are usually not
random; they come from a small number of ambiguities or missing facts in the prompt that
affect many inputs at once. Fixing the most common failure first gives the biggest gain,
the same way fixing the most common bug in software does.

Be aware of the limits of small tests: with 6 inputs, "5 out of 6" versus "6 out of 6"
may just be noise. Level 3 lesson 03 explains how to tell a real improvement from luck.

## Common mistakes

- **Tuning on one input**, then being surprised the prompt fails on the next one.
- **Changing many things at once.**
- **Only looking at the first output** in a batch.
- **Not saving versions**, so you can't return to the one that worked.
- **Adding rules forever** instead of reconsidering the task design.

## Exercise

1. Pick a task you'll do more than five times this month.
2. Gather 6 realistic inputs, including 2 awkward ones.
3. Write v1, run all 6, and name the most common failure.
4. Do at least three iterations, changing one thing each time, and log each version in
   your prompt notebook with its result.
5. Write one sentence on which change had the largest effect.
