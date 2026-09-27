---
description: "Compare two prompt versions fairly: paired offline tests, repeated runs, a simple sign test for noise, and cautious online A/B tests with guardrail metrics."
---

# 03 · A/B Testing Prompts

You've changed a prompt, and your eval score went from 84% to 88%. Is the new version
better? Maybe. With a small test set and sampling randomness, a four-point change can be
pure noise. This lesson shows how to compare prompts fairly and how to tell whether a
difference is likely to be real.

## Offline comparisons: paired by design

The strongest offline design is **paired**: run both prompts on the *same* test cases,
with the same model and settings, and compare case by case. Pairing removes the
variation that comes from some cases simply being harder than others.

For each case, record one of three outcomes:

- A better (A passes, B fails — or judge prefers A in both orders)
- B better
- no difference (both pass, both fail, or inconsistent judge)

Cases with no difference tell you nothing about which is better. The comparison lives in
the cases where they differ.

## Is the difference real? A sign test

A simple, assumption-light check: if A and B were truly equal, each differing case would
be equally likely to favour A or B, like a coin flip. The **sign test** asks how
surprising your split would be under that assumption.

```python
from math import comb

def sign_test(b_wins: int, a_wins: int) -> float:
    """Two-sided p-value for 'B and A are equally good', using only differing cases."""
    n = b_wins + a_wins
    k = max(b_wins, a_wins)
    tail = sum(comb(n, i) for i in range(k, n + 1)) / 2**n
    return min(1.0, 2 * tail)

# 50-case test set: new prompt B wins 7 cases, old prompt A wins 3, 40 tie
print(f"B 7 vs A 3:  p = {sign_test(7, 3):.3f}")
# Larger set, same proportions: B wins 28, A wins 12
print(f"B 28 vs A 12: p = {sign_test(28, 12):.3f}")
```

Output:

```text
B 7 vs A 3:  p = 0.344
B 28 vs A 12: p = 0.017
```

The same 70/30 split is weak evidence with 10 differing cases and much stronger with 40.
A p-value isn't the probability that B is better; it's how often you'd see a split at
least this lopsided if the prompts were equal. The practical lesson: small test sets can
only detect large differences. Grow the test set before celebrating small gains.

## Controlling other sources of variation

- **Same model version** for both arms. Pin versions where the provider allows.
- **Same settings**: temperature, max tokens, system prompt.
- **Repeat runs** when temperature > 0: run each case several times per prompt and use
  the pass rate per case.
- **Randomize order** of judge comparisons (lesson 02).
- **Don't peek and tweak**: if you edit B after seeing some results, re-run everything.

## Online A/B tests

For prompts in a live product, you can split real traffic: some users get A, some get B.
This measures what matters most — real outcomes — but requires care:

- **Randomize by user**, not by request, so each person has a consistent experience.
- **Choose one primary metric** in advance (task completion, thumbs-up rate, escalation
  to a human, resolved-without-reply rate).
- **Add guardrail metrics** that must not get worse: complaint rate, safety flags,
  latency, cost per request.
- **Run long enough** to cover weekly patterns and to collect enough outcomes; decide the
  duration before you start.
- **Ramp gradually**: start B at a small percentage and watch guardrails.

Online tests have real users on the other end. Test risky changes offline first.

## Worked example: a prompt that's better *and* worse

A new support prompt (B) improves the judged helpfulness pass rate compared with A on the
offline set, and the sign test on the differing cases is convincing. But the per-slice
breakdown shows B loses on the "refund request" slice: its warmer phrasing sometimes
implies a refund is guaranteed. Options: fix B's refund handling and re-test, or ship B
with a refund-specific rule. What you shouldn't do is ship B on the aggregate alone.

## How It Actually Works

A prompt comparison is a small experiment with two sources of randomness: which cases
are in your test set (sampling of inputs) and what the model generates (sampling of
outputs). Paired designs cancel much of the first; repeated runs average out the second.
Statistical tests quantify the leftover uncertainty. The sign test is used here because
it needs almost no assumptions — just that each differing case is an independent
"vote" — which suits small, messy prompt evals. With larger datasets, you may prefer
confidence intervals on the difference in pass rates or bootstrap resampling; the
principle is the same.

Online tests add a third source of variation — users themselves — and a new danger:
metrics can move for reasons unrelated to the prompt (seasonality, a product launch).
Randomization is what protects you: it makes those outside factors affect both arms
equally.

## Common mistakes

- **Comparing scores from different test sets or model versions.**
- **Declaring victory on a tiny difference** from a small test set.
- **One run per case at high temperature.**
- **Only aggregate metrics**, missing slice regressions.
- **Online tests without guardrail metrics.**

## Exercise

1. Take a prompt and a plausible improvement (one change).
2. Run both on your test set (at least 30 cases) and record per-case outcomes.
3. Count A-wins, B-wins and ties; run the sign test.
4. Break results down by slice. Write a short decision note: ship, don't ship, or
   investigate — and why.
