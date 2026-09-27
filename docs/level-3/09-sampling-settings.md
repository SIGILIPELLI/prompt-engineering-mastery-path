---
description: "What temperature, top-p, max tokens and stop sequences do, how to choose them per task, and why even temperature 0 isn't fully deterministic."
---

# 09 · Sampling Settings & Determinism

Two identical prompts can give different outputs, and the settings you pass alongside a
prompt shape how different. This lesson explains the common generation parameters at
the level you need to use them well, and clears up a frequent misunderstanding about
"deterministic" output.

## From probabilities to text

At each step the model outputs a score for every token in its vocabulary. These are
turned into probabilities, and a **sampler** picks one. The settings below change how the
sampler picks.

## Temperature

Temperature rescales the scores before they become probabilities:

- **Low (near 0):** the distribution sharpens; the top token dominates. Output is focused
  and repeatable-ish.
- **Around 1:** the model's "natural" distribution.
- **Higher:** the distribution flattens; less likely tokens get picked more often. More
  variety — and more mistakes and drift.

This small script shows the effect on a toy distribution of four candidate next words:

```python
import math

def softmax_with_temperature(scores: dict, t: float) -> dict:
    scaled = {k: v / t for k, v in scores.items()}
    m = max(scaled.values())
    exps = {k: math.exp(v - m) for k, v in scaled.items()}
    z = sum(exps.values())
    return {k: round(e / z, 3) for k, e in exps.items()}

scores = {"reliable": 2.0, "robust": 1.5, "solid": 1.0, "delightful": -1.0}
for t in (0.2, 1.0, 1.5):
    print(t, softmax_with_temperature(scores, t))
```

Output:

```text
0.2 {'reliable': 0.918, 'robust': 0.075, 'solid': 0.006, 'delightful': 0.0}
1.0 {'reliable': 0.494, 'robust': 0.3, 'solid': 0.182, 'delightful': 0.025}
1.5 {'reliable': 0.423, 'robust': 0.303, 'solid': 0.217, 'delightful': 0.057}
```

At 0.2 the top word almost always wins; at 1.5 the unlikely "delightful" is chosen more
than twice as often as at 1.0. Over hundreds of tokens, small shifts compound into very
different texts.

## Top-p and top-k

- **Top-k** keeps only the k most likely tokens and samples among them.
- **Top-p** (nucleus sampling) keeps the smallest set of tokens whose probabilities add up
  to p (e.g. 0.9), and samples among them.

Both cut off the long tail of unlikely tokens. A common guideline is to adjust
temperature *or* top-p, not both at once, so you can reason about the effect. Not every
provider exposes every parameter, and some reasoning models fix or ignore sampling
settings — check your provider's documentation.

## Other settings that affect prompts

- **Max tokens (output limit):** a hard cap. If reached, output stops mid-sentence (or
  mid-JSON!). It's a cost safeguard, not a length instruction — say the length you want in
  the prompt, and set max tokens comfortably above it.
- **Stop sequences:** strings that end generation when produced, e.g. stop at
  `</answer>`. Useful for bounded formats.
- **Seed:** some APIs accept a seed to make sampling more reproducible; it's typically
  documented as best-effort.

## Choosing settings by task

| Task | Typical starting point |
|------|------------------------|
| Extraction, classification, JSON, code | Low temperature |
| Q&A over documents | Low temperature |
| General writing | Provider default |
| Brainstorming, varied options | Default or somewhat higher; ask for variety in the prompt |
| Evals of a production prompt | The same settings as production |

These are starting points, not rules — test on your task.

## "Temperature 0 means deterministic" — not quite

Setting temperature to 0 (greedy decoding) removes sampling randomness in principle, but
in practice identical requests can still produce different outputs on hosted APIs.
Sources include floating-point differences in how computations are batched and ordered
on accelerator hardware (small differences can flip which token is on top when two are
nearly tied), and model or infrastructure updates behind the same model name. Design
for this:

- Run evals more than once for important decisions.
- Pin model versions where possible.
- Validate outputs in code instead of assuming a fixed string.
- Cache results if you need exact repeatability for a given input.

## Worked example: getting varied options without chaos

You want 10 different taglines. Raising temperature a lot produces variety but also
nonsense. Often better: keep a moderate temperature and ask for variety explicitly.

```text
Write 10 taglines for a bicycle repair café. Make them genuinely different from each
other: vary the angle (community, sustainability, saving money, learning a skill, humour),
the length (3–10 words), and the structure (question, imperative, statement).
Numbered list, no explanations.
```

Explicit diversity instructions give *structured* variety; temperature gives *random*
variety. You usually want the former.

## How It Actually Works

Temperature divides every score (logit) by t before the softmax. Dividing by a small t
magnifies the gaps between scores, so exponentiation makes the top token overwhelming;
dividing by a large t shrinks the gaps, pushing probabilities toward uniform. Top-p and
top-k then truncate the distribution before sampling. Because generation is sequential,
one unusual early token changes the context for every later token — this is why
high-temperature outputs can drift far off topic while low-temperature ones stay close
to the most typical path.

Low temperature isn't "more accurate" in general: if the model's top choice is wrong,
greedy decoding reliably picks the wrong answer. It is more *consistent*, which is what
you want for extraction and evaluation.

## Common mistakes

- **Cranking temperature to get creativity** instead of prompting for variety.
- **Using max tokens as a length control**, truncating output mid-structure.
- **Evaluating at different settings than production.**
- **Assuming temperature 0 gives identical outputs** forever.
- **Tuning temperature and top-p together** without understanding either.

## Exercise

1. Run the temperature script with your own toy scores; find the temperature where the
   second-best token passes 40% probability.
2. With a real model, run one extraction prompt 5 times at low temperature and 5 times at
   a high setting; compare consistency.
3. Run a brainstorming prompt at default temperature with and without explicit diversity
   instructions; count how many genuinely distinct ideas each produces.
