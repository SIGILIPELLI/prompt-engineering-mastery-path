---
description: "Design extraction and classification prompts that stay consistent: label definitions, decision rules for edge cases, abstention, and measuring agreement."
---

# 06 · Extraction & Classification Prompts

Extraction (pull specific facts out of text) and classification (assign text to a
category) are the quiet workhorses of applied prompting. They power ticket routing,
document triage, survey analysis, and data cleaning. They're also where inconsistency
hurts most: a label that flips between runs corrupts every count built on it.

## Classification: define the labels, not just name them

A label name is not a definition. "Urgent" means different things to different people —
and to a model it means whatever the average of its training data suggests.

```text
Classify the IT support request in <request> into exactly one priority.

Definitions:
- P1: a whole team or a customer-facing system cannot work, with no workaround.
- P2: one person cannot work, or a team is slowed but has a workaround.
- P3: an inconvenience or a request for something new; work can continue normally.

Decision rules:
- If the request mentions a security concern (phishing, lost laptop, suspicious login),
  classify as P1 regardless of other factors.
- Judge by the described impact, not by words like "URGENT" or "ASAP".
- If there isn't enough information to decide between two levels, choose the higher one
  and set "needs_followup" to true.

Return JSON: {"reason": "<one sentence>", "priority": "P1"|"P2"|"P3",
"needs_followup": true|false}
```

Notice the three ingredients: **definitions** tied to observable facts, **decision rules**
for the known tricky cases, and a policy for **uncertainty**.

## Let the model abstain

Forcing a choice when the input doesn't fit any class produces confident garbage. Offer
an explicit way out — `"other"`, `"unclear"`, or a `needs_followup` flag — and monitor how
often it's used. A sudden rise in "other" is often your first sign that incoming data has
changed.

## Extraction: fields, formats, and provenance

```text
From the job posting in <posting>, extract:
- job_title (string)
- salary_min, salary_max (numbers, in the posting's currency, annual; null if not stated)
- currency (ISO code like "EUR", or null)
- remote_policy: "remote" | "hybrid" | "onsite" | "unspecified"
- required_years_experience (number or null)
- evidence: for each non-null field, the exact phrase you took it from

Rules: do not convert currencies. If a range is given per hour or per month, return null
for salary fields and explain in evidence.
```

Two habits make extraction trustworthy:

1. **Normalize formats in the prompt** (ISO dates, ISO currency codes, enums).
2. **Ask for provenance** (the source phrase) so a program can verify that the phrase
   exists in the input, and a human can audit disagreements quickly.

## Consistency: measuring agreement

Run the same classification prompt several times on the same inputs. If labels change
between runs, your prompt has ambiguity (or your temperature is high). This snippet
measures run-to-run agreement and finds unstable items:

```python
from collections import Counter

# labels[i] = labels for input i across 3 runs of the same prompt
labels = {
    "t1": ["P2", "P2", "P2"],
    "t2": ["P1", "P2", "P1"],
    "t3": ["P3", "P3", "P3"],
    "t4": ["P2", "P3", "P3"],
    "t5": ["P1", "P1", "P1"],
}

unstable = {}
for item, runs in labels.items():
    top, count = Counter(runs).most_common(1)[0]
    if count < len(runs):
        unstable[item] = dict(Counter(runs))

stable_share = 1 - len(unstable) / len(labels)
print(f"stable items: {stable_share:.0%}")
print("unstable:", unstable)
```

Output:

```text
stable items: 60%
unstable: {'t2': {'P1': 2, 'P2': 1}, 't4': {'P2': 1, 'P3': 2}}
```

Read the unstable items. Almost always they fall into a gap between two definitions —
exactly where a new decision rule is needed.

## Worked example: fixing an ambiguous boundary

Suppose `t4` is: "The shared printer on floor 3 is jammed; people are walking to floor 2."
Is that P2 (team slowed, workaround exists) or P3 (inconvenience)? Both definitions
arguably apply. Decide — say, "an inconvenience affecting a team with an easy workaround
is P3" — and add it as a decision rule and as a few-shot example. Re-run and check that
`t4` is now stable *and* that no previously stable items changed.

## How It Actually Works

A classification prompt makes the model produce a label token (or a few) after reading
the input. The label chosen is the one with the highest probability under the model's
learned associations, shaped by your definitions. When an input sits near the boundary
between two definitions, the probabilities for the two labels are close, and small
factors — sampling randomness, example order, wording — tip it either way. That's what
run-to-run instability measures: closeness to a boundary.

Decision rules move the boundary to where you want it and push probabilities apart, so
borderline inputs become clear cases. Asking for a short reason before the label (as in
the JSON above) lets the label condition on an explicit judgement. Some APIs can also
return token probabilities (log-probs) for the label, which gives a direct confidence
signal — availability varies by provider.

## Common mistakes

- **Label names without definitions.**
- **No "other/unclear" option**, forcing wrong labels.
- **Letting the input's wording drive the label** ("URGENT!!!" becomes P1).
- **Extraction without provenance**, so errors can't be checked.
- **Asking the model to normalize or convert** values (currencies, units) that code
  should handle.

## Exercise

1. Choose a classification task with 3–5 labels. Write definitions and at least two
   decision rules.
2. Build 15 test inputs, including 5 you think are borderline. Label them yourself first.
3. Run the prompt 3 times on all 15; use the agreement script to find unstable items.
4. Add a decision rule for the largest cluster of unstable items; re-run and compare both
   stability and agreement with your own labels.
