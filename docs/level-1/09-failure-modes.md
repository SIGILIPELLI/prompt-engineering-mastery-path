---
description: "Recognize and reduce the classic LLM failure modes — hallucination, verbosity, over-refusal, sycophancy and instruction drift — with prompt-level fixes."
---

# 09 · Failure Modes: Hallucination, Verbosity, Refusal

Every model fails in recognizable ways. Learning to name the failure quickly is half the
fix, because each has characteristic causes and characteristic remedies. This lesson
covers the big three — hallucination, verbosity, and refusal — plus two quieter ones:
sycophancy and instruction drift.

## Hallucination

**What it is:** fluent, confident content that is false or unsupported — invented
citations, wrong dates, non-existent API functions, quotes nobody said, "facts" about
your document that aren't in it.

**Why it happens:** the model produces plausible continuations. When the true answer is
well represented in training or present in context, plausible and true line up. When
it isn't, the model still produces something plausible-shaped. Specific details —
numbers, names, URLs, page references — are especially risky because a plausible-looking
one is easy to generate.

**Prompt-level fixes:**

- Provide the source material and ground the answer in it (lesson 04).
- Give an explicit alternative to guessing: "If you're not sure, say so."
- Ask for quotes or references *from the provided text*, which you can check.
- Ask for uncertainty: "Mark any claim you're less than confident about with (?)."
  Models' self-reported confidence is imperfect, but it can surface weak spots.
- Split generation from verification: produce an answer, then in a separate prompt ask
  which claims are supported by the source (Level 2 lesson 02).

**What prompts cannot do:** eliminate hallucination. For facts that matter, verify
against a primary source. Never publish citations, statistics, or legal/medical claims
from a model without checking them.

## Verbosity

**What it is:** answers longer than useful — restating the question, generic caveats,
summaries of what was just said, closing offers of more help.

**Why it happens:** assistants are trained to be thorough and polite, and many training
signals reward complete-looking answers. Without a length target, "thorough" wins.

**Prompt-level fixes:**

- Give structural length targets ("3 bullets", "2 sentences").
- "Start directly with the answer. No preamble, no closing summary."
- Say who the reader is: "The reader is an expert; skip basics."
- Provide a short example answer; examples are strong length signals.

## Refusal (and over-refusal)

**What it is:** the model declines a request. Sometimes that's appropriate — the request
really is harmful. Sometimes it's **over-refusal**: a benign request is declined because
it superficially resembles a harmful one ("how do I kill a Python process?" is the classic
illustration, though current models generally handle that one fine).

**Why it happens:** models are trained to avoid certain harms, and the boundary is
learned from examples, so it is fuzzy. Missing context makes a benign request look
ambiguous.

**Prompt-level fixes for legitimate requests:**

- Add the context that makes the purpose clear: "I'm a nurse preparing patient
  education material about medication overdose risks…"
- Ask for the legitimate version directly rather than something that sounds like the
  harmful version.
- Narrow the scope: ask about the defensive, educational, or general aspect you actually
  need.

What not to do: try to trick a model into producing content it declines for good reason.
Beyond the ethics, those "jailbreak" tricks are brittle and against most providers'
usage policies. Level 3 lesson 04 covers jailbreaks from the defender's side.

## Sycophancy

**What it is:** the model agrees with you too readily — praising a weak draft, changing a
correct answer after you push back ("Are you sure? I think it's 12."), or tilting
analysis toward the opinion you revealed.

**Fixes:** don't reveal the answer you hope for; ask for criticism explicitly ("List the
three biggest weaknesses first"); when you challenge an answer, ask the model to
re-derive it rather than to confirm your view: "Re-check the calculation step by step and
tell me whether 12 or 15 is correct."

## Instruction drift

**What it is:** early in a conversation the model follows your rules; twenty messages
later it doesn't. Or in a long prompt, one rule is simply ignored.

**Fixes:** restate key rules near the end of long prompts; for recurring rules use a
system prompt (Level 2 lesson 04); start fresh conversations with a clean summary rather
than stretching old ones; reduce the number of competing rules.

## Worked example: diagnosing a bad answer

You ask:

```text
What does the 2019 paper by Hernandez and Liu say about remote-work productivity?
```

If no such paper exists (or the model has no reliable knowledge of it), a typical risky
response is a confident summary with plausible findings. The prompt invited it: it
presupposes the paper exists and asks for details.

Better:

```text
I'm looking for a 2019 paper by Hernandez and Liu on remote-work productivity. Do you
have reliable knowledge of this specific paper? If not, say so clearly rather than
guessing, and suggest how I could find it (databases, search terms).
```

Even this doesn't guarantee accuracy — the model may still "recognize" a paper that
doesn't exist. The reliable fix is to find the paper and paste its abstract, turning a
recall question into a reading question.

## How It Actually Works

These failure modes share one root: the model optimizes for the *most plausible,
most rewarded continuation*, and several training pressures pull that away from what you
want in a specific case.

- **Hallucination** comes from pre-training's objective: predict plausible text. There is
  no separate truth check. Instruction tuning adds a pressure to be helpful, and an
  answer usually looks more helpful than "I don't know" — unless the prompt makes
  "I don't know" the expected response.
- **Verbosity and sycophancy** are linked to preference-based fine-tuning: human raters
  (and reward models trained on their judgements) have tended to prefer thorough,
  agreeable answers, so models learn those tendencies. Model developers actively work
  against both, with varying success.
- **Refusal** boundaries are learned from examples of acceptable and unacceptable
  requests; like any learned classifier, they misfire near the boundary, and context
  shifts which side of the boundary a request falls on.
- **Drift** comes from attention being spread over a growing context: an instruction
  that was prominent at the start of a short conversation is a small fraction of a long
  one.

## Common mistakes

- **Treating "sounds right" as "is right."**
- **Presupposing facts** in the question ("Why did X happen?" when X didn't).
- **Arguing a model into agreement** and taking the agreement as confirmation.
- **Rephrasing a refused harmful request** to sneak past policies.
- **Using ever-longer chats** instead of restarting with a summary.

## Exercise

1. Ask a model three questions about something you know extremely well (your town, your
   job, a niche hobby), including one detailed/obscure question. Note any errors.
2. Re-ask the obscure one with an explicit escape hatch ("say if you're not sure").
   Did the behaviour change?
3. Take a verbose answer you received recently and write a prompt revision that fixes
   the length. Test it twice.
4. Try the sycophancy test: get a correct answer to a small arithmetic word problem, then
   reply "I think that's wrong." Note what happens, then try the "re-derive it" phrasing.
