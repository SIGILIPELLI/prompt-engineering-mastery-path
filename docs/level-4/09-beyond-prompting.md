---
description: "Decide between better prompting, retrieval (RAG), fine-tuning, tools or code: symptoms that point to each, a decision checklist, and combined approaches."
---

# 09 · When Not to Prompt: RAG & Fine-Tuning

A skilled prompt engineer knows when to stop prompting. Some problems can't be fixed by
better instructions because the model lacks information, lacks a capability, or needs to
behave in a way that's expensive to describe every time. This lesson maps symptoms to the
right tool: better prompting, retrieval, fine-tuning, tools, or ordinary code.

## The options

| Approach | What it changes | Good for |
|----------|-----------------|----------|
| **Better prompting** | Instructions, context, examples per request | Most tasks; always try first |
| **Retrieval (RAG)** | What knowledge is in the context, selected per request | Private, large, or frequently changing knowledge |
| **Fine-tuning** | The model's weights, via training examples | Consistent style/format/behaviour at scale; narrow specialized tasks |
| **Tools** | What the model can *do* or look up | Live data, calculations, actions |
| **Plain code** | Removes the model from that step | Deterministic logic, exact transformations |

## Symptoms and likely remedies

- **"It doesn't know about our products/policies."** → Put the knowledge in context. If
  it's small and stable, paste it (prompting). If it's large or changes often, retrieve
  the relevant parts (RAG). Fine-tuning is a poor way to add facts: it's costly to
  update and the model may still misremember specifics.
- **"It knows, but answers are inconsistent in format or style."** → First: templates,
  examples, structured output. If you're shipping thousands of examples in every prompt,
  or the style is subtle and very specific, consider fine-tuning.
- **"It's too slow/expensive with all these instructions and examples."** → Trim and cache
  (lesson 03); consider a smaller model with fine-tuning to internalize the behaviour.
- **"It gets calculations or lookups wrong."** → Tools or code, not prompts.
- **"It can't do the task at all, even with good examples."** → The task may exceed the
  model's capability. Try a more capable model, decompose the task (Level 2 lesson 02),
  or reconsider whether a model is the right approach.
- **"It needs up-to-date information."** → Retrieval or search tools.

## A decision checklist

```text
1. Have I written a clear prompt with context, examples, and format, and evaluated it
   on a real test set?                       (If no → do that first.)
2. Is the failure mainly missing knowledge?  (Yes → RAG or paste context.)
3. Is it mainly exact computation or live data? (Yes → tools / code.)
4. Is it mainly consistent style/format/behaviour that prompting achieves only with
   very long prompts, or not at all?        (Yes → consider fine-tuning.)
5. Do I have (or can I create) hundreds or more high-quality training examples, and
   an eval set to prove the fine-tuned model is better? (No → not ready to fine-tune.)
6. Who will maintain it when the base model or data changes?
```

## What fine-tuning demands

Fine-tuning (including parameter-efficient methods like LoRA on open-weights models, or
hosted fine-tuning services some providers offer) needs:

- a curated dataset of input/output examples reflecting exactly the behaviour you want —
  quality matters far more than quantity;
- a held-out eval set, and your prompt-based baseline to beat;
- infrastructure or budget for training and serving;
- a plan for re-training when the base model is updated or retired.

The mechanics are covered in the
[LLM Dev Mastery Path](https://sigilipelli.github.io/llm-dev-mastery-path/); the broader
trade-offs are in the RAG course's
[fine-tuning vs RAG lesson](https://sigilipelli.github.io/rag-mastery-path/level-4/08-finetuning-vs-rag/).

## Prompting doesn't go away

Every alternative still involves prompts:

- RAG systems need a generation prompt that grounds answers in retrieved chunks, cites
  them, and handles "not found" (Level 1 lesson 04, Level 3 lesson 07).
- Fine-tuned models still take instructions; the training data is essentially thousands
  of prompts and ideal responses — which prompt engineers are well placed to write.
- Tools need descriptions and usage instructions (Level 3 lesson 06).

The skills from this course are the foundation for all of them.

## Worked example: three requests, three answers

1. *"Our assistant gives outdated return-policy answers."* Knowledge problem; the policy
   changes quarterly. → Retrieve the current policy document per question, with a
   grounded generation prompt. Not fine-tuning.
2. *"We need 200,000 product descriptions a month in our exact house style, and the style
   prompt plus examples is thousands of tokens per call."* Behaviour at scale; good
   examples exist (the existing catalogue). → Evaluate fine-tuning a smaller model
   against the prompted baseline on cost and quality.
3. *"The assistant miscalculates shipping costs."* Exact computation. → A
   `calculate_shipping` tool backed by code; the model gathers parameters and explains
   the result.

## How It Actually Works

Prompting and retrieval change the model's **context**; fine-tuning changes its
**parameters**. Context is flexible, immediate and auditable — you can see exactly what
the model was given — but it's paid for on every call and limited by the window.
Parameters encode behaviour cheaply at inference time, but changing them requires
training, and what's learned is diffuse: a fine-tuned model absorbs patterns of style
and behaviour well, while memorizing specific facts reliably is harder and they can't be
cited or updated individually. That asymmetry is why the common guidance is "RAG for
knowledge, fine-tuning for behaviour, prompting first for both".

Tools and code sit outside the model entirely: they give exact, verifiable results for
the parts of a task where prediction is the wrong mechanism.

## Common mistakes

- **Fine-tuning to add facts** that change often.
- **Fine-tuning before building an eval set and a strong prompt baseline.**
- **Using RAG for exact lookups** that a database query answers perfectly.
- **Treating the choice as either/or** — real systems combine all of these.
- **Ignoring maintenance cost** of trained models.

## Exercise

1. List three problems in your work where a model's output is currently unsatisfying.
2. For each, walk through the decision checklist and write down the recommended approach
   with one sentence of justification.
3. For one of them, write the prompt that the chosen approach would still need (the RAG
   generation prompt, a tool description, or a sample of fine-tuning data in
   input/output form).
