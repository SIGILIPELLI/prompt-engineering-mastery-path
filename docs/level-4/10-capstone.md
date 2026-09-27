---
description: "Capstone: ship a production-ready prompt package — spec, versioned templates, eval and golden sets, guardrails, cost estimate, bias check and runbook."
---

# 10 · Capstone — A Production Prompt Package

The capstone brings the whole course together. You'll choose a realistic task and
deliver a **prompt package**: everything a team would need to run a prompt in production
with confidence — not just the prompt text, but its specification, tests, safeguards,
cost model, and operating instructions. A reviewer should be able to pick up your package
and understand what it does, how well, and how to change it safely.

## Choose a task

Pick one that you can test realistically. Options if you need a starting point:

- **Support reply assistant:** classifies an incoming message, retrieves the relevant
  help article (paste a small set of articles as the "knowledge base"), and drafts a
  grounded reply for a human agent to approve.
- **Meeting-to-actions pipeline:** turns transcripts into decisions, owners and dates
  as JSON, then a message for the team chat.
- **Research brief generator:** takes 3–5 provided sources and produces a sourced brief
  for a named audience, with every claim cited to a source.
- **Code review helper:** reviews diffs for a specific class of bug in a codebase you
  know, with evidence per finding.

## Deliverables

Organize the package like this:

```text
capstone/
  README.md            # overview, how to run, results summary
  SPEC.md              # task card, users, success criteria, non-goals, risks
  prompts/
    <step>/prompt.txt  # one directory per prompt in the chain
    <step>/meta.yaml   # version, owner, model, settings, changelog
  evals/
    cases.jsonl        # ≥ 40 tagged cases, with a held-out split
    golden.jsonl       # ≥ 15 cases, must-pass flagged
    rubric.md          # anchored criteria for judged checks
    run_eval.py        # extends your Level 3 harness
    results/           # saved runs: raw outputs + reports
  guardrails.py        # input/output checks with defined outcomes
  COST.md              # token measurements and monthly estimate
  FAIRNESS.md          # counterfactual tests and findings (if people are involved)
  RUNBOOK.md           # monitoring, rollback, model-migration steps
```

### 1. Specification (Level 1)

Task card: who uses it, what "good" means as checkable criteria, what it must never do,
and explicit non-goals.

### 2. Prompt design (Levels 1–2)

- Decompose into steps where it helps; each step has a defined input and output
  contract (JSON where code consumes it).
- System prompt with purpose, rules with reasons and alternatives, and format.
- Templates with descriptive placeholders; untrusted content in named tags.
- Document each design decision in one line in `meta.yaml` or the README ("examples
  cover the refund/cancellation boundary because…").

### 3. Evaluation (Level 3)

- ≥ 40 cases with tags, including edge, adversarial (descriptive injection cases) and
  "not answerable" cases; a held-out split you only run at the end.
- Programmatic checks first; a calibrated judge for what code can't check — report its
  agreement with your own labels on at least 20 cases.
- At least two prompt versions compared with per-slice results and a sign test.

### 4. Safety and guardrails (Level 3)

- Threat model: what private data, untrusted content, and external actions are involved?
- Input and output guardrails with measured false-positive and false-negative rates on
  small labelled sets, and a defined outcome for each.
- Human approval for anything consequential.

### 5. Production readiness (Level 4)

- Versioned prompts with metadata and changelogs; prompt version and hash logged per
  call.
- Golden set and a regression gate script with must-pass cases and thresholds.
- Cost and latency: measured token counts, a monthly estimate at a stated volume with
  your provider's current prices, and at least one optimization with its eval impact.
- Fairness: counterfactual tests if the task involves people.
- Runbook: metrics to watch, how to roll back, and a checklist for migrating to a new
  model version.

## Suggested schedule

| Week | Focus |
|------|-------|
| 1 | Spec, test cases (label them yourself), first prompt version, first eval run |
| 2 | Iterate with evidence; decomposition; structured outputs; judge calibration |
| 3 | Guardrails, injection cases, fairness checks, cost measurements |
| 4 | Regression gate, runbook, held-out evaluation, README and results write-up |

## Assessment rubric

Score yourself (or ask a peer) on each criterion, 1–4:

| Criterion | 4 looks like |
|-----------|--------------|
| Specification | Success criteria are checkable; non-goals and risks are explicit |
| Prompt craft | Clear structure, delimiters, grounded instructions, justified examples |
| Evaluation rigour | Tagged, varied cases; calibrated judge; paired comparison; held-out result reported honestly |
| Safety | Threat model matches design; guardrails measured; least privilege |
| Operability | Versioning, logging, regression gate, rollback and migration steps all present and runnable |
| Cost awareness | Measured tokens, realistic estimate, one validated optimization |
| Honesty | Results reported as measured, including failures and limitations |

The last criterion matters most. A package that reports "82% on the held-out set; fails
on multi-issue messages; here's why" is more valuable than one that claims perfection.

## Worked example: a README results section

```markdown
## Results (model: <name/version>, temperature 0, 3 runs per case)

| Metric                  | v1   | v2   | v3 (shipped) |
|-------------------------|------|------|--------------|
| Category accuracy       | ...  | ...  | ...          |
| Grounded reply (judge)  | ...  | ...  | ...          |
| Format errors           | ...  | ...  | ...          |
| Held-out accuracy       |  –   |  –   | ...          |

v3 vs v2: better on N cases, worse on M, sign-test p = ...
Known weaknesses: messages containing two unrelated issues (slice "multi", 60%).
Judge agreement with author labels: ... (kappa ...), 25 cases.
```

Fill in your own measured numbers — the value of the table is that every figure came from
a saved run in `evals/results/`.

## How It Actually Works

A production prompt is a socio-technical system: text, a model, code around it, data
flowing through it, and people relying on it. Each deliverable controls a different way
it can fail. The spec prevents building the wrong thing; the prompt design makes the
right behaviour likely; evals measure how likely; guardrails catch what gets through;
versioning, regression gates and runbooks keep it working as everything around it
changes. None of these is sufficient alone — together they turn a clever prompt into
something a team can depend on.

## Common mistakes

- **Polishing the prompt and skipping the evidence.**
- **A test set built after the prompt**, only from cases it already handles.
- **Reporting only the best run** or only the non-held-out results.
- **Guardrails with no measured error rates.**
- **A runbook nobody has tried** — do a practice rollback.

## Exercise

Complete the capstone:

1. Choose your task and write `SPEC.md`.
2. Build the prompts, eval sets, harness and guardrails following the deliverables list.
3. Run at least two prompt versions and the held-out evaluation; save all runs.
4. Complete `COST.md`, `FAIRNESS.md` (if applicable) and `RUNBOOK.md`.
5. Write the README with an honest results section, and score the package against the
   rubric. Ask someone else to follow your runbook to perform a rollback — then fix
   whatever confused them.
