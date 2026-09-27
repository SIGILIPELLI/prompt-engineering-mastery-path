---
description: "Understand where LLM cost and latency come from — input, output and reasoning tokens, calls per task — and the prompt-level levers that reduce them."
---

# 03 · Cost & Latency Tradeoffs

A prompt that's excellent but costs too much per call or makes users wait ten seconds
may still be the wrong prompt. Cost and latency are properties of your prompt design,
not just of the model you pick. This lesson shows where they come from and which levers
you control. Prices and speeds change often and differ by provider, so it uses formulas
and placeholders rather than real prices.

## Where cost comes from

For most hosted APIs, cost per call is roughly:

```text
cost ≈ input_tokens × input_price + output_tokens × output_price
```

- **Output tokens usually cost more** per token than input tokens.
- **Reasoning tokens** (from reasoning models or chain-of-thought) are generally billed as
  output, even when hidden.
- **Cached input** (lesson section below) may be billed at a discount where supported.
- **Calls per task** multiply everything: a 3-step chain with one retry and a judge is
  5 calls.

## Where latency comes from

- **Time to first token** grows with input length (the model must process the prompt)
  and with queueing at the provider.
- **Generation time** grows with output length — tokens are produced one at a time.
- **Sequential calls** add up; parallel calls don't (if independent).

So long outputs hurt latency more than long inputs, and chains hurt latency more than
single calls.

## A cost model you can reason with

```python
def monthly_cost(calls_per_day, in_tokens, out_tokens, in_price_per_m, out_price_per_m,
                 cached_share=0.0, cache_discount=0.0):
    """Estimated monthly cost. Prices are per million tokens; cached_share is the fraction
    of input tokens served from cache, billed at (1 - cache_discount) of the input price."""
    eff_in = in_price_per_m * (1 - cached_share * cache_discount)
    per_call = (in_tokens * eff_in + out_tokens * out_price_per_m) / 1_000_000
    return per_call * calls_per_day * 30

# Placeholder prices (units per million tokens) — substitute your provider's current rates.
IN_P, OUT_P = 1.0, 4.0
base = monthly_cost(10_000, in_tokens=3_000, out_tokens=500, in_price_per_m=IN_P, out_price_per_m=OUT_P)
fewer_examples = monthly_cost(10_000, 1_800, 500, IN_P, OUT_P)
shorter_output = monthly_cost(10_000, 3_000, 250, IN_P, OUT_P)
cached = monthly_cost(10_000, 3_000, 500, IN_P, OUT_P, cached_share=0.8, cache_discount=0.9)
for name, v in [("baseline", base), ("fewer examples", fewer_examples),
                ("half the output", shorter_output), ("80% cached prefix", cached)]:
    print(f"{name:<18} {v:8.1f}")
```

Output:

```text
baseline             1500.0
fewer examples       1140.0
half the output      1200.0
80% cached prefix     852.0
```

With these placeholder prices, trimming 1,200 input tokens and halving output tokens have
similar effects, and caching a stable prefix has the largest. With your real prices and
token counts the ranking may differ — that's why it's worth modelling.

## Prompt-level levers

**Shrink input:**

- Remove instructions that don't change behaviour (verify with your eval).
- Use fewer or shorter few-shot examples; test whether the last examples add anything.
- Send only relevant context (retrieval instead of whole documents).
- Strip boilerplate from pasted material.

**Shrink output:**

- Ask for exactly what's needed: JSON fields instead of prose; labels instead of
  explanations.
- Set length targets and a sensible max-tokens cap.
- Skip visible reasoning where it doesn't improve accuracy (Level 2 lesson 01), or use a
  reasoning setting with a lower effort level where offered.

**Structure for caching:**

- Many providers offer **prompt caching** for repeated prefixes. Put stable content
  (system prompt, tool definitions, examples, reference documents) first and variable
  content last, so the prefix is identical across calls. Caching rules and minimum sizes
  differ by provider.

**Fewer, smarter calls:**

- Route easy inputs to a smaller, cheaper model and hard ones to a larger model (a
  cascade), with a classifier or confidence check deciding.
- Run independent chain steps in parallel.
- Cache final results for repeated identical inputs.
- Use batch APIs (where offered) for non-urgent workloads; they're typically cheaper in
  exchange for slower turnaround.

**Perceived latency:**

- Stream output to users so they see text immediately.
- Put the most useful information first in the output format.

## Worked example: trimming a classifier

A classifier prompt has 12 few-shot examples (about 1,500 tokens) and returns a reason
plus a label. Experiments on the eval set:

1. Remove examples in groups, re-running the eval each time. Suppose accuracy holds with
   5 examples chosen to cover the tricky boundaries — the others added little.
2. Drop the free-text reason from production output, keeping it only in eval runs where
   you debug failures. (If the reason-first ordering was helping accuracy, keep a short
   reason — test it.)
3. Reorder so the system prompt and examples form a stable cached prefix.

Each change is validated by the regression gate (lesson 02) — cost savings that break
must-pass cases don't count.

## How It Actually Works

Processing input is parallel: the model can compute over all prompt tokens at once, so
long inputs add cost and some latency but relatively little per token. Generating output
is sequential: each new token requires another pass through the model, which is why
output tokens dominate latency and are priced higher. Prompt caching works because the
internal computations for a prefix (stored as a key-value cache in the attention layers)
can be reused when a later request starts with exactly the same tokens — any change early
in the prompt invalidates everything after it, which is why stable content must come
first.

## Common mistakes

- **Optimizing cost without re-running evals.**
- **Variable content at the start** of the prompt, defeating caching.
- **Verbose outputs** consumed only by code.
- **Large models for every call** when most inputs are easy.
- **Using real prices from memory** instead of the provider's current price page.

## Exercise

1. Measure the actual input and output token counts of one production-style prompt
   (most APIs return usage in the response).
2. Plug them into `monthly_cost` with your provider's current prices and your expected
   volume.
3. Try two levers (e.g. fewer examples, shorter output) and measure their eval impact.
4. Write a one-paragraph recommendation with estimated savings and quality impact.
