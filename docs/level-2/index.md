---
description: "Level 2 of prompt engineering: reasoning prompts, chaining, JSON output, system prompts, style control, extraction, summarization and templates."
---

# Level 2 · Intermediate <span class="level-badge">Techniques</span>

Goal: build a toolbox of techniques that work across models and tasks, and learn when
each one is — and isn't — worth using.

Level 1 was about writing one good prompt. Level 2 is about the patterns that show up
again and again in real work: getting a model to reason before answering, splitting big
jobs into chains of small prompts, producing machine-readable output, writing system
prompts, controlling voice and length precisely, and the everyday workhorse tasks of
extraction, classification, summarization, and transformation. The level ends with a
project that combines several techniques into a small pipeline.

From here on, lessons include short Python examples. They use only the standard
library and a `fake_model()` stand-in so they run without an API key; swap in a real
model call when you're ready. How to call specific provider APIs is covered in the
[LLM Dev Mastery Path](https://sigilipelli.github.io/llm-dev-mastery-path/) — this course
stays focused on what goes *into* the call.

## Modules

1. [Reasoning Prompts & Chain-of-Thought](01-reasoning-prompts.md) — asking for reasoning, when it helps, when it doesn't, and reasoning models
2. [Decomposition & Prompt Chaining](02-decomposition-chaining.md) — splitting a task into steps whose outputs feed each other
3. [Structured Output: JSON & Schemas](03-structured-output.md) — reliable machine-readable output, validation, and repair
4. [System Prompts](04-system-prompts.md) — what belongs in a system prompt versus the user turn, and how to organize one
5. [Controlling Tone, Length & Style](05-tone-length-style.md) — style specifications, voice samples, and reading-level targets
6. [Extraction & Classification Prompts](06-extraction-classification.md) — pulling fields out of text and assigning labels consistently
7. [Summarization Prompts](07-summarization.md) — purpose-driven summaries, faithfulness, and long documents
8. [Prompt Templates & Variables](08-templates-variables.md) — turning prompts into safe, reusable templates
9. [Transformation Prompts: Rewrite, Translate, Convert](09-transformation-prompts.md) — changing form without changing meaning
10. [Project — A Support-Ticket Triage Chain](10-project-triage-chain.md) — classify, extract, and draft replies in a tested three-step chain

## Before you start

- Complete Level 1, or be comfortable with its ideas: context, delimiting, few-shot
  examples, format templates, and iterating against a small test set.
- Python 3.9 or newer for the optional code examples.
