---
description: "What role prompts like 'You are an expert editor' really change, where personas help (tone, framing, audience) and why they can't add knowledge."
---

# 05 · Roles & Personas — and Their Limits

"You are a world-class copywriter." "Act as a senior Python developer." Role prompts are
the most famous prompting technique, and also the most over-credited. This lesson
explains what a persona actually does, when it earns its place, and when it is a
distraction from the context the model really needs.

## What a role changes

A role line is a compact way of setting several things at once:

- **Register and vocabulary** — a "pediatric nurse explaining to a parent" uses different
  words than "a clinical pharmacologist writing for peers".
- **Framing and priorities** — "act as a skeptical reviewer" makes criticism the expected
  output rather than praise.
- **Format conventions** — "as a technical writer" nudges toward headings, steps and
  consistent terminology.

In short, a role shifts *style and stance*. That can be genuinely useful.

## What a role cannot change

A role cannot give the model knowledge it doesn't have, or facts about your situation.
"You are a senior tax advisor" does not make the model know your country's current tax
rules or your client's circumstances. It may, however, make the answer *sound* more
authoritative — which is worse, because confident tone hides uncertainty.

Evaluations of expert-persona prompts on factual and reasoning tasks have generally found
small or inconsistent effects on accuracy. Treat any claim that a particular role phrase
reliably boosts correctness with suspicion, and test it on your own task.

## Better: role + context + task

A role is most useful as a short preface to real context:

```text
Weak:
You are an expert marketing strategist. Write a launch plan.

Stronger:
You are helping a two-person team launch a paid budgeting app for freelancers.
Take the stance of an experienced, pragmatic marketer who is skeptical of expensive
channels.

Context: budget of about 2,000 in local currency for the first three months; no
existing audience; the founders can each spend ~5 hours a week on marketing.

Task: propose a 3-month launch plan with at most 4 channels. For each channel say
why it fits the budget and time constraint, and one metric to watch.
```

The role line here does one job — it sets a stance (pragmatic, skeptical of spend). The
rest of the prompt carries the information.

## Useful role patterns

- **Stance roles**: "Critique this as a skeptical reviewer would. List weaknesses first."
- **Audience-inversion roles**: "You are explaining to a smart 14-year-old."
- **Perspective roles for critique**: "Read this contract clause as the *other* party's
  lawyer would. What would they object to?"
- **Simulation for practice**: "Play a hiring manager interviewing me for a data-analyst
  role. Ask one question at a time and wait for my answer." (Useful — but remember it's a
  simulation, not an insight into what real interviewers think.)

## Where personas go wrong

- **Fake credentials.** Asking a model to be "a licensed doctor" or "a lawyer" can
  produce advice formatted as if it were professional judgement. Use models for
  understanding and drafting; check high-stakes answers with a qualified human.
- **Persona drift.** In long conversations, a character voice may fade or intensify.
  For applications, put the stance in the system prompt (Level 2 lesson 04) rather than
  relying on one line at the start of a chat.
- **Stereotyped voices.** "Talk like a [nationality/profession]" invites caricature.
  Describe the tone you want directly instead ("warm, plain-spoken, short sentences").
- **Roles that fight the task.** "You are a poet" plus "output valid JSON" pulls in
  two directions.

## Worked example: testing whether a role helps

Task: review a short project update for problems.

```text
Version A:
Review the project update in <update> and list any problems.

Version B:
You are a project sponsor who has been burned by optimistic status reports before.
Review the project update in <update>. List anything that is vague, unsupported by a
number or date, or that hides a risk. Quote the phrase each time.
```

Version B combines a stance with *explicit criteria*. If you want to know which part
matters, test a Version C that keeps the criteria and drops the role. In many cases C
performs close to B, which tells you the criteria, not the persona, were doing the work.
That is a useful habit: when a prompt has several changes at once, isolate them (lesson
08).

## How It Actually Works

During training, a model sees text written by many kinds of people in many contexts.
A role statement shifts the prediction toward continuations that resemble text
associated with that role in the training data — its vocabulary, structure, and typical
concerns. Instruction tuning adds another layer: the model has learned to adopt personas
on request because doing so was rewarded as helpful.

What the role does *not* do is retrieve a hidden store of expert knowledge. The model's
parameters are the same whether or not you say "expert". If the knowledge is there, a
clear question usually surfaces it without a persona; if it isn't, a persona only changes
how confidently the gap is covered. That is why the combination of role (for stance)
plus explicit context and criteria (for substance) is more reliable than role alone.

## Common mistakes

- **Using a role as a substitute for context** ("You are an expert in our codebase").
- **Stacking superlatives** ("world-class, award-winning, genius…"); they add length, not
  quality.
- **Believing authority of tone**: persona outputs sound confident regardless of accuracy.
- **Unstated stance**: "act as a reviewer" without saying what to look for.
- **Keeping a role that no longer fits** after the task changes mid-conversation.

## Exercise

1. Take a critique task (review a paragraph you wrote, a plan, or some code).
2. Write three versions: role only; criteria only; role + criteria.
3. Run each twice. For each output, count how many of the actual issues it caught.
4. Write down which version you'd keep and whether the role earned its place.
