---
description: "Level 3 of prompt engineering: test sets, rubrics, LLM-as-judge, A/B tests, prompt injection defense, guardrails, tool calling, long context, multimodal."
---

# Level 3 · Advanced <span class="level-badge">Evidence & Safety</span>

Goal: replace "it seems better" with evidence, and make prompts that hold up against
hostile or unusual input.

Levels 1 and 2 used small, informal test sets. That is enough for personal prompts, but
not for a prompt that runs thousands of times inside a product, where one percent of
failures is a lot of unhappy users. This level teaches the measurement side of prompt
engineering — test sets, rubrics, automated graders and their biases, and fair
comparisons between prompt versions — and the defensive side: prompt injection,
jailbreaks, and guardrails. It also covers prompting for three situations that have their
own rules: tool calling, very long contexts, and images. It ends with a small evaluation
harness you build and run yourself.

## Modules

1. [Evaluating Prompts: Test Sets & Rubrics](01-evaluating-prompts.md) — building a test set, writing rubrics, and choosing metrics
2. [LLM-as-Judge and Its Biases](02-llm-as-judge.md) — automated grading with a model, and how to keep it honest
3. [A/B Testing Prompts](03-ab-testing-prompts.md) — comparing two prompts fairly, and telling real gains from noise
4. [Prompt Injection & Jailbreaks (Defensive View)](04-prompt-injection.md) — how attacks work conceptually and how to reduce the risk
5. [Guardrails Around Prompts](05-guardrails.md) — input and output checks, least privilege, and human review
6. [Prompting for Tool & Function Calling](06-tool-calling-prompts.md) — writing tool descriptions and instructions a model uses correctly
7. [Long-Context Strategies](07-long-context.md) — placement, structure, chunking, and quoting for large inputs
8. [Multimodal Prompting](08-multimodal-prompting.md) — prompting with images, charts, screenshots and documents
9. [Sampling Settings & Determinism](09-sampling-settings.md) — temperature, top-p, max tokens, and what "deterministic" really means
10. [Project — A Small Prompt Eval Harness](10-project-eval-harness.md) — a runnable harness with checks, a judge interface, and a comparison report

## Before you start

- Levels 1–2, especially iteration (L1·08), structured output (L2·03) and
  classification (L2·06).
- Python 3.9+. All code runs with the standard library and a mock model.
- Security note: lesson 04 describes attack *categories* so you can defend against them.
  It deliberately contains no working attack payloads.
