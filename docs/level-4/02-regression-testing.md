---
description: "Regression-test prompts: golden sets, tolerance thresholds instead of exact matches, CI gating, flaky-case handling, and safely migrating to a new model."
---

# 02 · Regression Testing Prompts

A **regression** is when something that used to work stops working. For prompts,
regressions come from three directions: someone edits the prompt, someone edits the code
around it, or the model underneath changes. Regression tests catch all three — if they're
designed for the fact that model outputs aren't exact.

## Golden sets

A **golden set** is a curated subset of your test cases whose expected behaviour is agreed
and stable. It's smaller than your full eval set and runs on every change. Good golden
cases:

- cover every output category and every important slice;
- include every bug that has previously reached users;
- include adversarial cases (injection, off-topic, empty input);
- have checks that are cheap and reliable (mostly programmatic).

## Assertions that tolerate variation

Exact string matching fails on harmless wording changes. Assert *properties* instead:

| Instead of… | Assert… |
|-------------|---------|
| exact reply text | contains the tracking number; under 120 words; no URLs outside allowlist |
| exact summary | mentions each required decision (keyword or judge check) |
| exact JSON | parses; required fields; label equals expected |
| "same as before" | judge prefers new ≥ old, or rubric score ≥ threshold |

## Thresholds, not perfection

Because outputs vary, gate on rates with tolerances: "category accuracy ≥ 0.92 on the
golden set, and no must-pass case fails". Two tiers work well:

- **Must-pass cases** (safety, known bugs, contract): any failure blocks the change.
- **Aggregate metrics**: must stay above a threshold and not drop more than an agreed
  margin from the current baseline.

## A tiny regression gate

```python
BASELINE = {"accuracy": 0.93, "format_ok": 1.00}
MAX_DROP = 0.02

def gate(results: list[dict]) -> tuple[bool, list[str]]:
    msgs = []
    must = [r for r in results if r["must_pass"] and not r["passed"]]
    for r in must:
        msgs.append(f"must-pass failed: {r['id']}")
    acc = sum(r["passed"] for r in results) / len(results)
    fmt = sum(r["format_ok"] for r in results) / len(results)
    if acc < BASELINE["accuracy"] - MAX_DROP:
        msgs.append(f"accuracy {acc:.2f} < {BASELINE['accuracy'] - MAX_DROP:.2f}")
    if fmt < BASELINE["format_ok"]:
        msgs.append(f"format_ok {fmt:.2f} < {BASELINE['format_ok']:.2f}")
    return (not msgs), msgs

results = [
    {"id": f"g{i}", "must_pass": i < 3, "passed": True, "format_ok": True} for i in range(20)
]
results[7]["passed"] = False
print(gate(results))
results[1]["passed"] = False
print(gate(results))
```

Output:

```text
(True, [])
(False, ['must-pass failed: g1', 'accuracy 0.90 < 0.91'])
```

The first run passes (one non-critical miss: 95% accuracy is above the 91% floor). The
second fails on two grounds: a must-pass case broke, and accuracy fell below the floor.

## Running in CI

- Run the golden set on every pull request that touches a prompt, its template code, or
  model configuration.
- Run the full eval set nightly or before releases.
- Cache model responses keyed by (prompt hash, model version, settings, input) so reruns
  of unchanged cases cost nothing.
- Store results as artifacts so reviewers can read failures, not just a red/green badge.
- Keep API keys in the CI system's secrets store, never in the repository.

## Flaky cases

Some cases pass sometimes and fail sometimes. Don't just delete them — they sit on real
decision boundaries. Options: run them several times and assert a pass rate; tighten the
prompt so they become stable; or, if the ambiguity is genuine, accept either answer and
document why.

## Model migrations

When a provider releases a new model or retires an old one, treat the switch like a
major release:

1. Run the full eval set on the current and new model with the current prompt.
2. Compare per-slice and case by case (Level 3 lesson 03). Expect some wins and some
   losses; newer is not automatically better *for your prompt*.
3. Read the changed cases. Prompts often contain workarounds for the old model's quirks
   that are unnecessary or harmful for the new one — remove them and re-test.
4. Watch for changed defaults: verbosity, formatting habits, refusal behaviour, and
   tool-calling style.
5. Roll out gradually with monitoring, and keep the old configuration ready for rollback
   until the provider's retirement date.

## Worked example: a regression that wasn't in the diff

A team adds one line to a summarizer prompt: "Be concise." Golden set: format and
accuracy unchanged. But a must-pass case — a transcript where a safety decision appears
only in the last minute — now fails: the shorter summaries drop the final decision. The
fix is not to remove "Be concise", but to make coverage explicit ("Always include every
decision, even if brief"). Without that must-pass case, the regression would have
shipped.

## How It Actually Works

Regression testing for prompts is statistical quality control. Each run of the golden
set is a sample; thresholds and tolerances define how much deviation from the baseline
counts as a real change rather than noise. Must-pass cases act as hard invariants — the
equivalent of unit tests for the behaviours you can never afford to lose — while
aggregate thresholds catch broad drift. Caching and pinned model versions make runs
reproducible enough that a change in results can be attributed to a change in inputs
rather than randomness.

Model migrations are the hardest case because every prompt's "interpreter" changes at
once; prompts tuned against one model's quirks encode assumptions that silently stop
holding.

## Common mistakes

- **Exact-match assertions** on free text.
- **No must-pass tier**, so critical failures are averaged away.
- **Deleting flaky cases** instead of understanding them.
- **Switching model versions** without a full eval.
- **Keeping old-model workarounds** after migration.

## Exercise

1. Pick 15 cases from your eval set as a golden set; mark 3–5 as must-pass.
2. Write property-based assertions for each (no exact text matches).
3. Adapt `gate()` and set a baseline from a current run.
4. Make a deliberately harmful prompt edit and confirm the gate fails; then revert.
5. If you have access to two model versions, run your golden set on both and write a
   short migration note.
