---
description: "Split complex tasks into focused prompts chained together: when to decompose, how to pass outputs between steps, and how to debug a chain step by step."
---

# 02 · Decomposition & Prompt Chaining

When a prompt grows past a dozen rules and still fails in new ways each time you fix it,
the problem is usually not wording. The task is too big for one prompt. **Decomposition**
splits it into smaller subtasks; **prompt chaining** runs them in sequence, feeding each
step's output into the next.

## Signs you should split a prompt

- It asks for several different *kinds* of work at once (analyse, decide, then write).
- Fixing one part keeps breaking another.
- You can't tell which part of the prompt caused a failure.
- Some parts need different settings (a strict extraction step vs a creative writing step).
- Intermediate results would be useful to check, cache, or show to someone.

## A simple chain

Task: turn a long customer interview transcript into a prioritized list of product
improvements with supporting quotes.

```text
Step 1 — Extract
Read the transcript in <transcript>. List every distinct problem or wish the customer
expressed. For each, include one verbatim quote. Output as numbered lines:
<n>. <problem> | "<quote>"

Step 2 — Group
Here is a list of customer problems with quotes (in <problems>). Group duplicates and
near-duplicates. Output each group with a short name and the numbers it contains.

Step 3 — Prioritize and write
Here are grouped customer problems (in <groups>) and our current roadmap (in <roadmap>).
Write a prioritized list of up to 5 improvements not already on the roadmap. For each,
cite the supporting quote(s) exactly as given.
```

Each step has one job, a clear input, and a clear output format. You can inspect the
extraction before trusting the priorities.

## Patterns of decomposition

- **Sequential pipeline:** extract → transform → write (above).
- **Generate then critique:** draft in one call, critique against criteria in a second,
  revise in a third. Often improves quality for writing and code.
- **Routing:** a first prompt classifies the input; the next prompt depends on the class
  (a billing question goes to a billing-specific prompt).
- **Map then reduce:** run the same prompt on each chunk of a long document, then combine
  results in a final prompt (Level 3 lesson 07).
- **Verify:** after generating an answer, a separate prompt checks each claim against
  the source.

## Passing data between steps

- **Use structured intermediate formats** — numbered lines or JSON (lesson 03) — so the
  next step and your code can parse them.
- **Pass only what the next step needs.** A later step that sees the full transcript
  plus the extraction may start re-extracting on its own.
- **Carry identifiers forward** (item numbers, IDs) so you can trace any final claim back
  to its source.
- **Validate between steps** in code: is the JSON valid? Are all quotes actually present
  in the transcript?

## A tiny chain in Python

This runs end to end with a stand-in model function so you can see the plumbing. The
"model" here is a deterministic fake; replace `fake_model` with a real API call later.

```python
def fake_model(prompt: str) -> str:
    """Stand-in for an LLM call: returns canned text based on the step label."""
    if prompt.startswith("STEP:extract"):
        return ('1. Export is slow | "exports take forever"\n'
                '2. Wants dark mode | "my eyes hurt at night"\n'
                '3. Export times out | "big exports just fail"')
    if prompt.startswith("STEP:group"):
        return "Export performance: 1, 3\nDark mode: 2"
    raise ValueError("unknown step")

def verify_quotes(extracted: str, transcript: str) -> list[str]:
    """Return quotes that do not appear verbatim in the transcript."""
    quotes = [line.split('"')[1] for line in extracted.splitlines() if '"' in line]
    return [q for q in quotes if q not in transcript]

transcript = ("Honestly exports take forever, and big exports just fail. "
              "Also my eyes hurt at night, is there a dark theme?")

step1 = fake_model("STEP:extract\n<transcript>" + transcript + "</transcript>")
bad = verify_quotes(step1, transcript)
print("unverified quotes:", bad)
step2 = fake_model("STEP:group\n<problems>" + step1 + "</problems>")
print(step2)
```

Output:

```text
unverified quotes: []
Export performance: 1, 3
Dark mode: 2
```

The `verify_quotes` check is the important idea: code between steps catches a
hallucinated quote before it propagates to the final report.

## Worked example: generate, critique, revise

```text
Call 1 (draft):
Write a 150-word announcement of our office move for all staff, using the facts in
<facts>.

Call 2 (critique):
Here is a draft announcement (in <draft>) and the facts it should be based on (in
<facts>). List: (a) any statement not supported by the facts; (b) any fact missing from
the draft; (c) anything unclear for someone who has never visited the new office.
Output as three short lists. Don't rewrite the draft.

Call 3 (revise):
Revise the draft in <draft> to fix every issue in <critique>. Keep it under 150 words.
Output only the revised announcement.
```

Keeping the critique separate from the rewrite matters: a model asked to "check and fix"
in one go tends to rewrite without really checking.

## How It Actually Works

A single prompt carrying many subtasks splits the model's attention across all of them,
and every generated token has to serve several goals at once. Shorter, focused prompts
give each call one objective, which is easier to satisfy and easier to evaluate. Chaining
also turns hidden intermediate state into visible text you can inspect, validate, and
correct — the same benefit as chain-of-thought, but with a program (or you) in the loop
between steps.

The cost is more calls, more latency, and the risk that errors compound: if step 1 misses
a problem, step 3 can't recover it. That is why validation between steps and a test set
that checks each step separately are part of the pattern, not optional extras.

Chains are the simplest form of what are often called *workflows*; when the model itself
decides which step to run next, you are building an agent (Level 4 lesson 04, and in
depth in the [LLM Dev Mastery Path](https://sigilipelli.github.io/llm-dev-mastery-path/)).

## Common mistakes

- **Splitting too finely**, so each call lacks the context to do its job.
- **Unstructured hand-offs** that the next step misreads.
- **No checks between steps**, letting a step-1 error flow to the output.
- **Passing everything to every step**, recreating the original overloaded prompt.
- **Evaluating only the final output**, which hides which step is weak.

## Exercise

1. Take a task you currently do with one big prompt (report writing, research summary,
   code review).
2. Break it into 2–4 steps; write each step's prompt with a defined input and output
   format.
3. Run the chain on three inputs and inspect every intermediate output.
4. Add one automatic check between two steps (even a simple one, like the quote check).
5. Compare the final quality with your original single prompt on the same inputs.
