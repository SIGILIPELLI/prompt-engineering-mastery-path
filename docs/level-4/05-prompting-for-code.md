---
description: "Prompting models to write, explain, review and test code: precise specs, giving the right code context, test-first prompts, and verifying generated code."
---

# 05 · Prompting for Code

Code is one of the domains where models are most useful — and where their mistakes are
most easily hidden behind code that looks right. Code has one enormous advantage over
prose: it can be *executed and tested*. Good code prompting leans on that advantage at
every step.

## Specify like a ticket, not a wish

```text
Weak:
write a function to parse dates

Strong:
Write a Python 3.11 function `parse_date(s: str) -> datetime.date | None`.
- Accept "YYYY-MM-DD", "DD/MM/YYYY" and "D Mon YYYY" (e.g. "3 Mar 2026"), English
  month abbreviations only.
- Return None for anything else, including impossible dates like 31/02/2026.
- Standard library only; prefer clear code over clever one-liners.
- Include a docstring and 8 pytest tests covering each format, invalid dates, surrounding
  whitespace, and the empty string.
```

The strong version states language and version, signature, accepted inputs, error
behaviour, constraints, and how it will be verified.

## Give the right context

Models write better code when they can see how it will fit in:

- relevant existing code (interfaces, a similar function, data models) — not the whole
  repository;
- conventions (naming, error handling, logging style, test framework);
- versions of key libraries, since APIs change across versions;
- what already exists, so it doesn't reinvent helpers.

```text
<conventions>
- Errors: raise domain exceptions from app/errors.py; never return error strings.
- Logging: use the module logger, no print().
- Tests: pytest, fixtures in tests/conftest.py.
</conventions>
<existing_code path="app/errors.py"> ... </existing_code>
```

## Test-first prompting

Ask for tests before (or alongside) the implementation, review the tests yourself, then
ask for the implementation that makes them pass. Tests are shorter and easier to review
than code, and they turn your intent into something executable.

```text
Step 1: Write pytest tests for the behaviour described in <spec>. Don't write the
implementation. Include edge cases you think I might have missed, each with a comment
explaining why.
```

## Review and explanation prompts

For code review, narrow the focus and require evidence:

```text
Review the diff in <diff> for correctness bugs only (not style). For each issue:
file and line, what input triggers it, what happens, and a suggested fix.
If you find no correctness bugs, say so — don't pad with style comments.
```

For explanation, name your audience and purpose: "Explain what this function does to a
developer new to this codebase, then list any behaviour that would surprise them."

## Verify, always

Generated code can call functions that don't exist in the library version you use,
mishandle edge cases, or introduce security problems (injection, unsafe deserialization,
secrets in code). Verification habits:

- Run it. Run the tests. Add your own tests for edge cases.
- Check unfamiliar API calls against the library's documentation.
- Be suspicious of dependencies the model suggests — confirm the package exists, is the
  one you think it is, and is maintained, before installing.
- Run linters, type checkers and security scanners as you would for human code.
- Review generated code with the same bar as a colleague's pull request.

## Worked example: turning a bug report into a fix prompt

```text
Bug: `apply_discount(total, code)` returns a negative total when a fixed-amount coupon
exceeds the order total.

Code is in <code>; existing tests in <tests>.

1. Write a failing pytest test that reproduces the bug.
2. Propose the smallest fix: totals must never go below 0; keep existing behaviour
   for all current tests.
3. List any other inputs that could produce a negative total after your fix.

<code> ... </code>
<tests> ... </tests>
```

The prompt asks for a reproduction first (verifiable), a minimal fix (reviewable), and
adjacent risks (the model's breadth, used as a checklist for you).

Here's what the resulting test and fix might look like — verified by running it:

```python
def apply_discount(total: float, code: dict) -> float:
    if code["type"] == "percent":
        total = total * (1 - code["value"] / 100)
    elif code["type"] == "fixed":
        total = total - code["value"]
    return max(round(total, 2), 0.0)

def test_fixed_coupon_larger_than_total_is_clamped():
    assert apply_discount(20.0, {"type": "fixed", "value": 25}) == 0.0

def test_percent_coupon_unchanged():
    assert apply_discount(80.0, {"type": "percent", "value": 25}) == 60.0

test_fixed_coupon_larger_than_total_is_clamped()
test_percent_coupon_unchanged()
print("both tests pass")
```

Output:

```text
both tests pass
```

## How It Actually Works

Models learned to code from large amounts of public source code, documentation and
discussion, so they are strong at common patterns and popular libraries — and weaker at
rare libraries, recent API changes, and your private codebase's conventions, which they
only know if you put them in context. When a model "invents" a function, it's producing
a plausible name by analogy with similar APIs, the code-shaped version of hallucination.

Tests change the game because they convert "looks plausible" into "demonstrably behaves".
Coding agents that can run tests use exactly this loop — generate, execute, read errors,
revise — which is why they can outperform a single-shot prompt: execution feedback is an
external source of truth the model otherwise lacks.

## Common mistakes

- **Vague specs** with no signature, inputs, or error behaviour.
- **Too much or too little context** — the whole repo, or none of it.
- **Accepting code without running it.**
- **Installing suggested packages without checking them.**
- **Asking "is this code correct?"** instead of asking for specific failure-inducing
  inputs.

## Exercise

1. Pick a small function you need (or a real bug). Write a strong spec using the template
   above.
2. Use test-first prompting: get tests, review and fix them yourself, then get the
   implementation.
3. Run the tests. Add two edge-case tests the model missed.
4. Ask for a correctness-only review of the final code; verify each reported issue by
   writing a test for it.
