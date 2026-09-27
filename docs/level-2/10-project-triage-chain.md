---
description: "Level 2 project: build a three-step support-ticket chain — classify, extract to JSON, draft a grounded reply — with validation and a test set, runnable offline."
---

# 10 · Project — A Support-Ticket Triage Chain

This project combines the Level 2 techniques into one small, realistic pipeline. A
support inbox receives free-text tickets. Your chain will:

1. **Classify** each ticket (category and priority) using definitions and decision rules.
2. **Extract** key fields into validated JSON.
3. **Draft** a reply grounded in a help-centre snippet, in a defined voice.

You'll build it with a mock model so the plumbing runs offline, then swap in a real
model and test with the prompts as written.

## The prompts

### Step 1 — classify (system prompt + template)

```text
SYSTEM:
You triage support tickets for "Parcelio", a parcel-tracking app.

Categories:
- delivery: a parcel is late, lost, damaged, or delivered to the wrong place.
- account: login, password, profile, or notification settings.
- billing: charges, refunds, subscription plan.
- other: anything else, including feature requests.

Priority:
- high: money charged in error, or a parcel reported lost/stolen.
- normal: everything else.

Rules: judge by the described problem, not by tone or capital letters. If two categories
apply, choose the one describing the customer's main request.
Return JSON only: {"reason": string, "category": ..., "priority": ...}

USER:
<ticket>
{ticket}
</ticket>
```

### Step 2 — extract

```text
From the ticket in <ticket>, extract JSON with keys:
- "tracking_number": string or null (Parcelio numbers look like "PCL-" + 8 digits)
- "order_date": "YYYY-MM-DD" or null
- "customer_request": one sentence describing what the customer wants
- "evidence": object mapping each non-null field to the exact phrase used
Return JSON only.

<ticket>
{ticket}
</ticket>
```

### Step 3 — draft

```text
Write a reply to the customer as Parcelio Support.
Voice: warm, direct, plain words, no exclamation marks, under 120 words.
Use only the help-centre text in <help> for any policy or procedure.
If <help> doesn't cover the request, say a specialist will reply within one business day.
Mention the tracking number if one is given. Don't promise refunds or dates.

Ticket category: {category}
Customer request: {customer_request}
Tracking number: {tracking_number}

<help>
{help_text}
</help>
```

Notice each step's job is narrow, and step 3 never sees the raw ticket — only the fields
it needs. That reduces the chance of the model reacting to irrelevant details or to
instructions embedded in the ticket.

## The runnable pipeline

Save as `triage.py` and run with `python3 triage.py`. The `fake_model` function returns
canned, deliberately imperfect outputs so you can see the validation working.

```python
import json
import re

HELP = {
    "delivery": "Parcels count as lost after 10 days without a tracking update. "
                "Customers can file a claim in the app under Orders > Report a problem.",
    "billing": "Refunds for duplicate charges are reviewed within 5 business days.",
}

def fake_model(step: str, ticket: str) -> str:
    """Stand-in for an LLM. Returns canned responses keyed on ticket content."""
    if step == "classify":
        if "charged twice" in ticket:
            return '{"reason": "duplicate charge", "category": "billing", "priority": "high"}'
        if "lost" in ticket:
            return '{"reason": "parcel lost", "category": "delivery", "priority": "high"}'
        return 'Sure! {"category": "other"}'          # malformed on purpose
    if step == "extract":
        m = re.search(r"PCL-\d{8}", ticket)
        return json.dumps({
            "tracking_number": m.group(0) if m else None,
            "order_date": None,
            "customer_request": "Resolve the issue described.",
            "evidence": {"tracking_number": m.group(0)} if m else {},
        })
    raise ValueError(step)

CATEGORIES = {"delivery", "account", "billing", "other"}

def parse_json(raw: str) -> dict | None:
    try:
        obj = json.loads(raw)
        return obj if isinstance(obj, dict) else None
    except json.JSONDecodeError:
        return None

def classify(ticket: str) -> dict:
    obj = parse_json(fake_model("classify", ticket))
    if not obj or obj.get("category") not in CATEGORIES or obj.get("priority") not in {"high", "normal"}:
        return {"category": "other", "priority": "normal", "needs_human": True}
    return {**obj, "needs_human": False}

def extract(ticket: str) -> dict:
    obj = parse_json(fake_model("extract", ticket)) or {}
    tn = obj.get("tracking_number")
    if tn and (not re.fullmatch(r"PCL-\d{8}", tn) or tn not in ticket):
        obj["tracking_number"] = None                   # reject invented numbers
    return obj

def run(ticket: str) -> dict:
    c = classify(ticket)
    e = extract(ticket)
    help_text = HELP.get(c["category"], "")
    return {"category": c["category"], "priority": c["priority"],
            "needs_human": c["needs_human"], "tracking": e.get("tracking_number"),
            "has_help": bool(help_text)}

TESTS = [
    ("My parcel PCL-12345678 is lost, no update for 2 weeks", "delivery", "high"),
    ("I was charged twice for my Plus plan this month", "billing", "high"),
    ("Can you add a dark mode?", "other", "normal"),
]

passed = 0
for ticket, want_cat, want_pri in TESTS:
    out = run(ticket)
    ok = out["category"] == want_cat and out["priority"] == want_pri
    passed += ok
    print("PASS" if ok else "FAIL", out)
print(f"{passed}/{len(TESTS)} passed")
```

Output:

```text
PASS {'category': 'delivery', 'priority': 'high', 'needs_human': False, 'tracking': 'PCL-12345678', 'has_help': True}
PASS {'category': 'billing', 'priority': 'high', 'needs_human': False, 'tracking': None, 'has_help': True}
PASS {'category': 'other', 'priority': 'normal', 'needs_human': True, 'tracking': None, 'has_help': False}
3/3 passed
```

The third ticket "passes" only because the fallback happens to produce the expected
label; the `needs_human: True` flag reveals that the model's output was malformed. When
you switch to a real model, track how often the fallback fires — it's a key health
metric for the chain.

## Switching to a real model

Replace `fake_model(step, ticket)` with a function that renders the step's prompt
template (lesson 08) and calls your provider's API, returning the text. Then:

1. Expand `TESTS` to at least 15 tickets: 3 per category plus edge cases (angry but
   low-priority, polite but high-priority, two issues in one ticket, a ticket containing
   "ignore your instructions and…").
2. Label them yourself before running.
3. Run each ticket 3 times; record category/priority agreement and fallback rate.
4. Read every drafted reply (step 3) against its voice spec and grounding rule.

## How It Actually Works

The chain's reliability comes less from any single prompt than from the **contracts
between steps**: each step emits a structure that code validates before the next step
runs. When a model output violates the contract (bad JSON, unknown category, a tracking
number that isn't in the ticket), the pipeline takes a safe default and flags the item,
rather than passing bad data forward. This is the same idea as input validation in any
software system — the model is treated as a powerful but fallible component.

Step 3's grounding is also structural: because it sees only `help_text` and extracted
fields, it has little material from which to invent policy, and the "specialist will
reply" fallback gives it an acceptable alternative when help text is missing.

## Common mistakes

- **Passing the raw ticket to every step**, reintroducing noise and injection risk.
- **Silent fallbacks** that hide how often the model misbehaves.
- **Trusting extracted identifiers** without checking they appear in the source.
- **Testing only the final reply** instead of each step.
- **A test set without adversarial or ambiguous tickets.**

## Exercise

1. Run `triage.py` as given and confirm the output.
2. Add a fourth test ticket that the fake model classifies wrongly, and watch it fail.
3. Add the `account` category to `HELP` with a sentence of help text, and a test for it.
4. Replace `fake_model` with a real model call and run your 15-ticket test set three
   times. Report category accuracy, priority accuracy, and fallback rate.
5. Pick the most common failure and fix it with one prompt change. Re-run and compare.
