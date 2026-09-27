---
description: "Level 1 of prompt engineering: how models read prompts, prompt anatomy, clarity, context, personas, few-shot, formatting, iteration and failure modes."
---

# Level 1 · Entry <span class="level-badge">Foundations</span>

Goal: stop treating prompting as guesswork. By the end of this level you should be able
to take a vague request ("summarize this", "write me an email"), turn it into a prompt
that produces a useful result most of the time, and explain *why* each part of the
prompt is there.

Three ideas run through every lesson in this level:

1. **The model only sees text.** It has no access to your intentions, your earlier
   conversations in another window, or the document you forgot to paste. Whatever is
   not in the context does not exist for it.
2. **The model continues patterns.** It predicts what text is likely to come next given
   everything before it. Good prompts make the *useful* continuation the most likely
   one.
3. **One good result proves little.** Outputs vary from run to run. A prompt is good
   when it works across many inputs and many runs — which is why lesson 08 introduces
   a simple habit of testing before you trust.

## Modules

1. [How a Model Reads Your Prompt](01-how-llms-read-prompts.md) — tokens, the context window, next-token prediction and sampling, at an intuitive level
2. [Anatomy of a Good Prompt](02-anatomy-of-a-prompt.md) — task, context, input, constraints, output format and examples, and how to order them
3. [Clarity & Specificity](03-clarity-specificity.md) — replacing vague words with checkable criteria
4. [Giving Context](04-giving-context.md) — audience, purpose, background material, and delimiting what you paste
5. [Roles & Personas — and Their Limits](05-roles-personas.md) — what "You are an expert…" actually changes, and what it cannot
6. [Few-Shot Examples](06-few-shot-examples.md) — choosing, ordering and formatting examples without over-steering
7. [Asking for a Format](07-output-formatting.md) — lists, tables, Markdown, fixed templates and length
8. [Iterating on a Prompt](08-iterating-on-prompts.md) — one change at a time, a small test set, and a prompt notebook
9. [Failure Modes: Hallucination, Verbosity, Refusal](09-failure-modes.md) — recognizing each and the prompt-level fixes that help
10. [Project — A Reusable Prompt, Before & After](10-project-reusable-prompt.md) — take a real recurring task and engineer a prompt you can reuse

## What you need before starting

- Access to at least one chat-based language model. Any mainstream assistant or an
  open-weights model running locally will do. Free tiers are enough for this level.
- A plain text file or note to keep your prompt versions in.
- No programming is required for Level 1. A few lessons include optional Python
  snippets (standard library only) for readers who want them.

## A note on the examples

Prompts in this course are shown in fenced blocks labelled `text`. When a lesson says
what a model "typically" does, that is a qualitative description of common behaviour
across current instruction-tuned models, not a recorded transcript. Try the prompts
yourself; your model may behave differently, and noticing *how* it differs is part of
the skill.
