---
description: "When chain-of-thought and step-by-step prompts improve answers, when they don't, how reasoning models change the picture, and how to separate reasoning from output."
---

# 01 · Reasoning Prompts & Chain-of-Thought

Ask a model a multi-step question and demand the answer immediately, and it has to
produce the result in its first few tokens. Ask it to work through the problem first,
and it can use its own written steps as scaffolding. That is the core idea of
**chain-of-thought (CoT)** prompting — and also the key to understanding when it helps
and when it's just extra text.

## The basic technique

```text
Without reasoning:
A shop sells notebooks at 3 for 4.50. A teacher needs 40 notebooks for her class.
How much will she pay? Answer with the amount only.

With reasoning:
A shop sells notebooks at 3 for 4.50. A teacher needs 40 notebooks for her class.
The shop only sells in packs of 3. Work through the problem step by step, then give
the final amount on its own line starting with "Answer:".
```

The second version gives the model room to figure out that 40 notebooks needs 14 packs
(13 packs is only 39), and 14 × 4.50 = 63.00. When forced to answer instantly, a model is
more likely to compute 40 × 1.50 = 60.00 and miss the pack constraint.

## Ways to ask for reasoning

- **Zero-shot CoT:** "Think step by step before answering." Simple and often enough.
- **Structured reasoning:** name the steps you want: "First list the constraints. Then
  check each option against them. Then choose."
- **Few-shot CoT:** include worked examples that show the reasoning, not just the answer.
  Useful when you want a particular reasoning style (e.g. always check units).
- **Reason-then-answer format:** reasoning inside `<thinking>` tags (or a "Reasoning:"
  section), final answer in `<answer>` tags, so a program can extract just the answer.

## When reasoning helps

Reasoning prompts tend to help on tasks where a person would also need scratch paper:

- multi-step arithmetic and word problems;
- logic puzzles and constraint checking ("which of these meeting times works for all
  five people?");
- comparing several options against several criteria;
- tasks where a hidden detail changes the answer (like the pack size above);
- code reasoning ("what does this function return for input X?").

## When it doesn't help (or hurts)

- **Simple lookups and classifications.** "Is this email spam?" rarely benefits, and
  reasoning costs extra tokens and time.
- **Knowledge the model lacks.** Reasoning can't create a missing fact; it can produce a
  long, confident path to a wrong one.
- **Style and creative tasks.** "Think step by step" before a poem mostly adds latency.
- **Tight latency or cost budgets.** Reasoning tokens are output tokens — typically the
  more expensive kind — and they arrive before the answer.
- **Rationalization risk.** Written reasoning isn't always a faithful account of how the
  model reached its answer. Researchers have shown cases where models' stated reasoning
  omits factors that actually influenced the answer. Treat reasoning as a useful
  artifact, not proof.

## Reasoning models

Several providers now offer **reasoning models** (sometimes called "thinking" models)
that are trained to produce extended internal reasoning before answering, often with a
setting to control how much. With these:

- You usually **don't need** "think step by step" — they reason by default, and some
  providers advise against micromanaging the steps.
- Clear goals, constraints, and success criteria matter more than reasoning scaffolds.
- The reasoning may be hidden, summarized, or returned separately, depending on the
  provider. Check its documentation before depending on it.

A practical rule: with a standard model, try explicit reasoning prompts on multi-step
tasks; with a reasoning model, spend your effort on describing the problem well.

## Worked example: option comparison with structured reasoning

```text
We need to pick one of three project-management tools for a 12-person team.

Hard requirements:
1. Supports guest access for 2 external contractors.
2. Has a Gantt/timeline view.
3. Costs under 150 per month for 12 users.

<options>
Tool A: 10 per user/month. Timeline view. Guests: not supported.
Tool B: 11 per user/month. Timeline view on all plans. Guests: free, unlimited.
Tool C: flat 99/month up to 15 users. Timeline view only on the 199/month plan.
        Guests: supported.
</options>

In <thinking>, check each option against each hard requirement one at a time, showing
the arithmetic for cost. Then in <answer>, name the tool that meets all requirements, or
say "none" and explain which requirement fails for each.
```

Working it by hand: A fails guest access; B costs 12 × 11 = 132 (under 150), has
timeline and guests — passes; C at 99 lacks the timeline view and at 199 exceeds the
budget — fails. The correct answer is Tool B. The structured check-each-requirement
instruction makes it much harder for the model to skip C's plan detail.

## Extracting the answer in code

```python
import re

def final_answer(response: str) -> str | None:
    """Pull the text inside <answer>...</answer>, or None if missing."""
    m = re.search(r"<answer>(.*?)</answer>", response, re.DOTALL)
    return m.group(1).strip() if m else None

demo = "<thinking>A fails guests. B: 12*11=132 ok...</thinking>\n<answer>Tool B</answer>"
print(final_answer(demo))
print(final_answer("no tags here"))
```

Output:

```text
Tool B
None
```

Always handle the `None` case: sometimes a model will skip the tags, and your program
should detect that rather than show the raw reasoning to users.

## How It Actually Works

Generation is sequential: each token is predicted using a fixed amount of computation
over the tokens before it. A question requiring several dependent steps can exceed what
the model can reliably compute "in one go" for the first answer token. Writing
intermediate steps changes the situation: each step becomes part of the context, so later
tokens can attend to already-computed partial results. The written reasoning is, in
effect, external working memory.

That also explains the limits. If a step depends on knowledge the model doesn't have,
writing it down doesn't help. And if an early step is wrong, later steps condition on the
error — which is why a long chain can be confidently wrong. Techniques like sampling
several reasoning paths and taking the most common final answer (often called
**self-consistency**) exploit the fact that errors in independent paths tend to differ
while correct answers agree — at the cost of multiple calls.

Reasoning models are trained, often with reinforcement learning on problems with
checkable answers, to produce useful intermediate reasoning on their own, which is why
the explicit instruction matters less for them.

## Common mistakes

- **Adding "think step by step" to everything**, including trivial tasks.
- **Showing reasoning to end users** when they only need the answer.
- **Parsing the answer from free text** instead of asking for a marked final line/tag.
- **Treating the explanation as the true cause** of the answer.
- **Micromanaging reasoning models** with rigid step lists when a clear problem
  statement would do better.

## Exercise

1. Write three word problems of your own with a hidden constraint (like the pack size).
   Work out the correct answers by hand.
2. Run each with "answer only" and with a reason-then-answer prompt, three times each.
   Record correctness.
3. Run a simple classification task both ways and compare. Did reasoning help there?
4. Write a one-paragraph rule of thumb, based on your own results, for when you'll use
   reasoning prompts.
