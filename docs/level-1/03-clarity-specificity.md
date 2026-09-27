---
description: "How to turn vague prompt words like 'better', 'short' and 'professional' into specific, checkable criteria a model can actually meet."
---

# 03 · Clarity & Specificity

"Make it better." "Keep it short." "Sound professional." These feel like instructions,
but each hides a decision you have not made yet. The model will make that decision for
you — usually by picking the most average interpretation. Specificity is the habit of
making the decision yourself and writing it down.

## The "new colleague" test

Imagine handing your prompt to a capable new colleague who started today. They are
smart, but they know nothing about your project, audience, or taste. Could they do the
task well without asking you a single question? If they would need to ask "how long?",
"for whom?", or "what counts as done?", the model needs those answers too.

## Replace vague words with criteria

| Vague | Specific |
|-------|----------|
| short | "3 bullet points, each under 15 words" |
| professional | "formal register, no exclamation marks, no slang, address the reader as 'you'" |
| simple | "readable by a 12-year-old; define any term a non-specialist wouldn't know" |
| better | "fewer passive sentences; the main recommendation in the first sentence" |
| detailed | "cover causes, impact, and two mitigation options, one paragraph each" |
| a few | "exactly 5" |

A good test: could a second person check whether the output met the instruction? "Under
100 words" is checkable. "Concise" is not.

## Say what to do, not only what to avoid

Negative instructions ("don't be verbose", "don't use jargon") tell the model where not
to go without telling it where to go. They also put the unwanted concept into the
context. Pair every "don't" with a "do":

```text
Weak:   Don't use technical jargon.
Better: Explain in everyday words. If a technical term is unavoidable (e.g. "API"),
        define it in brackets the first time you use it.
```

Negative instructions are still useful for hard limits ("Do not include customer
names"). Just don't rely on them to describe the output you want.

## Specify the success criteria

For anything non-trivial, tell the model what a good answer achieves:

```text
Write a one-paragraph explanation of compound interest for a personal-finance blog.

A good answer:
- uses one concrete example with round numbers the reader can check mentally
- explains why growth speeds up over time
- does not recommend any specific financial product
- is 90-130 words
```

This reads like a checklist — deliberately. You will reuse these exact criteria in
Level 3 when you grade outputs.

## Worked example: sharpening a request step by step

Starting point:

```text
Give me some ideas for team building.
```

Typical result: a long generic list (escape rooms, trust falls, cooking classes) that
fits no particular team.

Sharpen one dimension at a time:

1. *Who:* "for a remote software team of 8 people across 3 time zones"
2. *Constraint:* "that can be done in a 45-minute video call, with no purchases"
3. *Goal:* "aimed at helping people who rarely work together get to know each other"
4. *Format:* "Give 5 ideas. For each: name, one-sentence description, what to prepare."
5. *Exclusion with alternative:* "Avoid anything competitive or requiring personal
   disclosures; prefer collaborative problem-solving."

Final prompt:

```text
Suggest team-building activities for a remote software team of 8 people across
3 time zones. Each must fit in a 45-minute video call and require no purchases.
Goal: help people who rarely work together get to know each other.
Prefer collaborative problem-solving; avoid anything competitive or that asks people
to share personal information.

Give 5 ideas. For each: a name, a one-sentence description, and what the organiser
needs to prepare.
```

The model now has a much narrower target, and you can check every output against it.

## When not to be specific

Specificity narrows the output. Sometimes you want breadth — early brainstorming,
exploring angles you haven't thought of. In that case be specific about the *process*
instead ("give 15 very different ideas, including some unusual ones") and leave the
content open. Over-specifying also has a cost: every constraint is something the model
must juggle, and a long list of minor rules can crowd out the important ones. Keep the
rules that you would actually reject an output for breaking.

## How It Actually Works

A vague prompt is compatible with a huge range of continuations. The model's output
reflects the probability-weighted "centre" of that range — the typical response to a
typical version of your request, as seen in training. That centre is generic by
construction.

Each specific constraint conditions the prediction: it removes continuations that
conflict with it, and the probability mass moves to the ones that remain. Numeric and
structural constraints ("5 ideas", "one sentence each") are particularly effective
because they are easy for the model to track while generating — they correspond to
visible patterns in the text it is producing. Subjective constraints ("engaging") are
weak because there is no crisp pattern to track.

Exact counts are still not guaranteed: models track "roughly 100 words" better than
"exactly 100 words", because they do not count tokens-to-words precisely. If a hard
limit matters, verify it (with code if the prompt is part of an application).

## Common mistakes

- **Adjectives instead of criteria**: "engaging", "high-quality", "comprehensive".
- **Undefined relative terms**: "shorter" (than what?), "more formal" (how much?).
- **Only negative instructions**, with no description of what you do want.
- **Burying the one constraint that matters** in a list of ten trivial ones.
- **Demanding precision the model cannot deliver** (exact word counts) and not checking.

## Exercise

1. Write down five vague words you often use in prompts.
2. For each, write a specific, checkable replacement for one real task of yours.
3. Take one prompt and add a "A good answer:" checklist of 3–5 criteria.
4. Run it, then score the output against your own checklist. Which criterion did the
   model miss most often across three runs? Rewrite that line and try again.
