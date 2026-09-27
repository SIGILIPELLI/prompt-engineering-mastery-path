---
description: "Guardrails around LLM prompts: input screening, output validation, topic and PII filters, fallbacks, human review, and measuring false positives."
---

# 05 · Guardrails Around Prompts

A **guardrail** is any check outside the main prompt that constrains what goes into the
model or what comes out of it. Prompts ask the model to behave; guardrails verify that it
did, and decide what happens when it didn't. They're how you get predictable behaviour
from an unpredictable component.

## Where guardrails sit

```text
user input ──► [input checks] ──► prompt + model ──► [output checks] ──► user / next step
                    │                                       │
                    └────────► block / redirect / flag ◄────┘
```

## Input guardrails

- **Scope checks:** is the request within what this assistant is for? A cheap
  classifier prompt (or a small model) can route off-topic requests to a polite
  standard reply.
- **Injection/jailbreak screening:** flag likely attempts (lesson 04).
- **PII handling:** detect and redact personal data you don't need before it reaches the
  model or your logs (e.g. card numbers, government IDs). Pattern-matching catches
  structured identifiers; names and addresses need more sophisticated detection.
- **Size and format limits:** reject or truncate inputs that are too long; normalize
  encodings.

## Output guardrails

- **Structure:** JSON parses; fields and enums are valid (Level 2 lesson 03).
- **Content rules:** no disallowed topics; no promises your policy forbids ("guaranteed
  refund"); no URLs outside an allowlist.
- **Grounding:** quoted text appears in the source; cited IDs exist.
- **Safety moderation:** many providers offer moderation endpoints or classifiers for
  harmful content categories.
- **PII leakage:** the output doesn't contain personal data it shouldn't.

## What to do when a guardrail fires

Every guardrail needs a defined outcome:

| Outcome | When |
|---------|------|
| **Retry** (with the error fed back) | Format errors; one or two attempts max |
| **Fallback response** | Off-topic, repeated failures ("I can't help with that here; try…") |
| **Redact and continue** | PII that isn't needed |
| **Escalate to a human** | High-stakes or ambiguous cases |
| **Block and log** | Clear policy violations |

## A small output guardrail

```python
import re

ALLOWED_DOMAINS = {"help.example.com"}
FORBIDDEN = [r"\bguarantee(d)?\b", r"\brefund (is|will be) (approved|issued)\b"]
CARD = re.compile(r"\b(?:\d[ -]?){13,16}\b")

def check_reply(reply: str) -> list[str]:
    problems = []
    for url_domain in re.findall(r"https?://([^/\s]+)", reply):
        if url_domain.lower() not in ALLOWED_DOMAINS:
            problems.append(f"link to non-allowed domain: {url_domain}")
    for pattern in FORBIDDEN:
        if re.search(pattern, reply, re.IGNORECASE):
            problems.append(f"forbidden promise: /{pattern}/")
    if CARD.search(reply):
        problems.append("possible card number in output")
    return problems

replies = [
    "You can request a refund at https://help.example.com/refunds.",
    "Your refund will be approved today, guaranteed! Details: https://bit.example.net/x",
]
for r in replies:
    print(check_reply(r) or "ok")
```

Output:

```text
ok
['link to non-allowed domain: bit.example.net', 'forbidden promise: /\\bguarantee(d)?\\b/', 'forbidden promise: /\\brefund (is|will be) (approved|issued)\\b/']
```

Simple rules like these are fast, cheap, and explainable. Use model-based checks for
things rules can't capture, such as tone or subtle policy violations.

## Measure your guardrails

Guardrails make mistakes in both directions:

- **False negatives:** harmful or broken outputs get through.
- **False positives:** good outputs get blocked, frustrating users.

Build a small labelled set for each guardrail (should-block and should-pass examples)
and measure both rates. An over-eager guardrail — say, a topic filter that blocks every
message mentioning "medication" in a pharmacy's assistant — can be worse than none.

## Human review

For high-stakes domains (medical, legal, financial, HR decisions) or irreversible
actions, the most reliable guardrail is a person. Design review to be efficient: show
the reviewer the input, the output, which checks fired, and the relevant source text.

## Worked example: a guardrail plan for a travel-policy assistant

```text
Input:
- Scope classifier: travel policy & booking how-to → continue; else polite redirect.
- Redact passport numbers and card numbers before the model call.
Output:
- Must cite a policy section ID that exists in the policy index.
- No monetary limits in the reply unless the cited section contains that number.
- No links except the intranet travel page.
On failure: one retry with the error; then "I'm not certain — please check section
{nearest_section} or contact travel@ (link)".
Measurement: 40 labelled cases per check; review false positives weekly.
```

## How It Actually Works

Guardrails work because they change the *system's* failure behaviour without requiring
the model to be perfect. The model is probabilistic; deterministic checks around it
catch specific, well-defined failures with certainty, and model-based checks add a second,
partly independent opinion for fuzzier ones. If the main model and a checker fail on
different cases, stacking them reduces the overall failure rate — but only to the extent
their errors are independent. Two calls to the same model with similar prompts tend to
share blind spots, which is why deterministic rules and different models/approaches add
the most value.

## Common mistakes

- **Guardrails with no defined outcome** ("we log it" and nothing else).
- **Unmeasured false-positive rates.**
- **Relying only on model self-checks** that share the main model's blind spots.
- **Logging raw PII** while carefully redacting it from prompts.
- **Unbounded retries** that multiply cost and latency.

## Exercise

1. List the three worst things your assistant (or Level 2 project) could output.
2. Write a programmatic check for at least two of them; adapt `check_reply`.
3. Build 10 should-block and 10 should-pass examples per check and measure both error
   rates.
4. Define the outcome (retry, fallback, escalate, block) for each check.
