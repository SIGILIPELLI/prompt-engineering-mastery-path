---
description: "What context to give a model — audience, purpose, background facts, source material — how to delimit it, and how to avoid drowning the task."
---

# 04 · Giving Context

The single biggest upgrade most people can make to their prompts is not clever wording;
it is telling the model what it cannot possibly know. Context is the difference between
"write an email declining the meeting" producing something usable and something you have
to rewrite from scratch.

## Four kinds of context

1. **Audience** — who will read or use the output? A board, a customer, a new
   developer, a 10-year-old? Audience decides vocabulary, depth, and tone at once.
2. **Purpose** — what will the output be *used for*? "So the reader can decide whether
   to approve the budget" is more useful than "a summary".
3. **Situation and background** — the facts of your world: what happened, what has
   been tried, constraints you are under, relationships between people.
4. **Source material** — the actual documents, data, code, or notes the task is about.

Purpose is the most under-used. The same meeting notes should be summarized differently
for "someone who needs to know what they must do by Friday" and "an auditor checking
that the decision was properly approved".

## Delimiting source material

When you paste material, mark exactly where it starts and ends, and say what it is:

```text
Below are notes from a customer interview (in <interview>) and our current feature list
(in <features>). Identify which customer needs are not covered by any existing feature.

<interview>
...
</interview>

<features>
...
</features>
```

Delimiters matter for three reasons: the model can tell which text is which; you can
refer to each piece by name; and if the pasted text contains instructions ("ignore the
above and…"), clear framing reduces — though does not eliminate — the chance they are
followed. Level 3 lesson 04 covers that risk properly.

## Grounding: "use only what I gave you"

If the answer must come from your material, say so and say what to do when it isn't
there:

```text
Answer the question using only the policy text in <policy>. If the policy does not
answer it, reply exactly: "The policy doesn't cover this." Quote the sentence you
relied on.
```

This does not make hallucination impossible, but it gives the model an explicit,
acceptable alternative to guessing — and asking for a quote gives *you* a way to check.
Answering questions over large document collections is its own discipline: see the
[RAG Mastery Path](https://sigilipelli.github.io/rag-mastery-path/).

## How much context is enough?

More is not automatically better. Irrelevant context costs tokens and can pull the model
toward details that don't matter (a stray remark in pasted notes can end up
over-emphasized in a summary). A good rule: include context that would change the
answer. If removing a paragraph would not change what a good human expert writes, it
probably doesn't need to be there.

A quick checklist before sending:

- Would a new colleague know **who** this is for?
- Would they know **why** it's needed and what happens next?
- Do they have the **facts** they'd otherwise have to invent?
- Is the **material** clearly marked?

## Worked example: context for a technical explanation

```text
Explain what a database index is.
```

Result: a correct, generic explanation, pitched somewhere in the middle — probably too
technical for a manager and too basic for an engineer.

With context:

```text
I'm a product manager. Our engineers say a report page is slow because a query "needs
an index" and it'll take a day to add safely. I need to understand enough to judge
whether that's reasonable and to explain the delay to a customer.

Explain what a database index is and why adding one to a large, live table might take
care, in under 200 words, with an everyday analogy. Then give me one sentence I could
say to the customer.
```

Now the model knows the audience (a non-engineer), the purpose (judge a plan and explain
a delay), and the required outputs. The explanation shifts toward the trade-offs that
matter (write cost, locking on a busy table) instead of internals.

## Context in conversations

In a chat, earlier messages are context too. That helps — you can build up background
over several turns — but it also means stale or wrong information from earlier keeps
influencing later answers. If a conversation has drifted, starting a fresh chat with a
clean summary of the relevant facts often works better than correcting it repeatedly.

## How It Actually Works

Everything in the context window shapes the prediction, because attention lets each
generated token draw on any earlier token. Context works by shifting probabilities: once
"I'm a product manager" is present, continuations that use analogies and skip internals
become more likely, because in training data explanations addressed to non-specialists
look like that.

This is also why irrelevant context can hurt. The model has no reliable way to know that
a paragraph is irrelevant unless you tell it; the paragraph still influences attention.
Research on long inputs has also found that models can use information at the start and
end of a long context more reliably than information in the middle — so the placement of
key facts matters as material grows (Level 3 lesson 07).

Grounding instructions work by making "I don't know"-style continuations acceptable and
likely. Without an explicit alternative, the most plausible continuation of a question
is an answer, whether or not the material supports one.

## Common mistakes

- **Leaving out the purpose**, so the model optimizes for a generic reader.
- **Pasting without delimiting**, especially multiple documents.
- **Dumping everything**, including irrelevant history that distracts.
- **Grounding without an escape hatch**: "use only the document" with no instruction for
  when the document doesn't contain the answer.
- **Trusting stale conversation context** after the facts have changed.

## Exercise

1. Choose a task where you'd normally paste a document (notes, a report, an article).
2. Write the prompt three ways: (a) task only; (b) task + audience + purpose; (c) (b) plus
   the material in tags and a grounding instruction with an escape hatch.
3. Ask one question the material does *not* answer. Compare how (a) and (c) handle it.
4. Record in your notebook which piece of context changed the output most.
