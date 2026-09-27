---
description: "Using a language model to grade outputs: judge prompt design, pairwise vs pointwise scoring, known biases (position, length, self-preference) and calibration."
---

# 02 · LLM-as-Judge and Its Biases

Programmatic checks can't tell you whether a summary is faithful or a reply is helpful.
Humans can, but they're slow and expensive. **LLM-as-judge** uses a model, given a
rubric, to score outputs. It's widely used because it scales — and widely misused because
judges have systematic biases that can quietly distort your conclusions.

## Two judging formats

**Pointwise:** score one output against a rubric.

```text
You are grading a customer-support reply.

Rubric — Faithfulness to policy:
4: every statement about policy is supported by <policy>
3: one minor imprecision that wouldn't mislead the customer
2: one statement that could mislead the customer
1: contradicts the policy or invents a policy

<policy>...</policy>
<reply>...</reply>

First list each policy-related statement in the reply and whether <policy> supports it.
Then output the score on the last line as: SCORE: <1-4>
```

**Pairwise:** compare two outputs and pick the better one.

```text
Here are two replies (A and B) to the same customer message, and the rubric.
Which reply better satisfies the rubric? Explain briefly, then output
WINNER: A, WINNER: B, or WINNER: TIE.
```

Pairwise comparisons are often easier for judges (and humans) to do consistently than
absolute scores, and they fit naturally with A/B tests (lesson 03).

## Known biases

These have been documented in published research on LLM judges and are worth assuming
until your own checks show otherwise:

- **Position bias:** in pairwise comparisons, a judge may favour the first (or second)
  option regardless of content.
- **Verbosity bias:** longer answers tend to be rated higher even when not better.
- **Self-preference:** a judge may favour outputs that resemble its own style, or that
  were produced by the same model family.
- **Surface-feature bias:** confident tone, formatting, and fluent language can mask
  factual errors.
- **Limited expertise:** a judge can't reliably verify facts it doesn't know or maths it
  can't do.

## Mitigations

- **Swap positions:** run each pairwise comparison twice with A and B swapped; count a
  win only if it's consistent, otherwise treat it as a tie.
- **Narrow criteria:** judge one criterion per call with an anchored rubric, rather than
  "overall quality".
- **Reasoning before the score:** ask for the evidence first, then the score on a fixed
  final line.
- **Give the judge the reference material** (source document, expected answer) so it
  checks rather than guesses.
- **Control length:** add "Do not reward length; a shorter answer that meets the rubric
  is better" and check your results for correlation with length.
- **Use a different model family as judge** when comparing prompts on a given model,
  where practical.
- **Calibrate against humans** (below).

## Calibrating a judge

Before trusting a judge, measure it. Have a human score a sample of outputs (say 30–50),
run the judge on the same outputs, and measure agreement. Simple percent agreement is a
start; **Cohen's kappa** corrects for agreement that would happen by chance:

```python
from collections import Counter

human = ["pass", "pass", "fail", "pass", "fail", "pass", "pass", "fail", "pass", "pass"]
judge = ["pass", "pass", "fail", "pass", "pass", "pass", "pass", "fail", "fail", "pass"]

n = len(human)
observed = sum(h == j for h, j in zip(human, judge)) / n
hc, jc = Counter(human), Counter(judge)
expected = sum(hc[k] * jc[k] for k in set(human) | set(judge)) / n**2
kappa = (observed - expected) / (1 - expected)

print(f"agreement={observed:.2f} chance={expected:.2f} kappa={kappa:.2f}")
disagreements = [i for i, (h, j) in enumerate(zip(human, judge)) if h != j]
print("look at cases:", disagreements)
```

Output:

```text
agreement=0.80 chance=0.58 kappa=0.52
look at cases: [4, 8]
```

80% raw agreement sounds good, but much of it is expected by chance when most outputs
pass; kappa of about 0.5 indicates only moderate agreement. Read the disagreement cases:
they usually reveal an ambiguous rubric line or a blind spot in the judge. Fix, re-run,
re-measure. (With 10 items this is only an illustration; use more for real decisions.)

## Worked example: detecting position bias

Run 20 pairwise comparisons where A and B are **identical** outputs. An unbiased judge
should say TIE or split roughly evenly. If it picks "A" in most of them, it has a
position preference. Then check your real comparisons were run in both orders.

## How It Actually Works

A judge is a model doing a classification task (Level 2 lesson 06) over long, complex
inputs — so everything about classification applies: its label is driven by learned
associations, and inputs near a boundary are unstable. Judges were trained on the same
kind of preference data that makes assistants verbose and agreeable, so they inherit
related preferences: longer, more confident, better-formatted answers "look" more like
the answers that were rewarded. Position bias arises because the order of options is
part of the input sequence, and nothing forces the model to treat positions
symmetrically.

Swapping positions, narrowing criteria, and providing references each remove a way for
superficial features to substitute for the actual judgement you want. Calibration against
humans tells you how much residual bias remains.

## Common mistakes

- **Using a judge without measuring its agreement with humans.**
- **One-shot "rate this 1–10 for quality".**
- **Pairwise comparisons in one order only.**
- **Letting the judge grade facts it can't verify** without a reference.
- **Treating judge scores as ground truth** in reports.

## Exercise

1. Write a pointwise judge prompt for one criterion of your Level 3 lesson 01 rubric.
2. Score 20 outputs yourself (pass/fail), then run the judge on the same outputs.
3. Compute agreement and kappa with the script above; read every disagreement.
4. Revise the rubric or judge prompt once and re-measure.
5. Run the identical-pair position-bias test with a pairwise prompt and report the result.
