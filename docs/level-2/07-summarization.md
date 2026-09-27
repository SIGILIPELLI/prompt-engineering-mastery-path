---
description: "Write summarization prompts that serve a purpose: audience-driven summaries, faithfulness checks, extractive vs abstractive, and summarizing long documents."
---

# 07 · Summarization Prompts

"Summarize this" is probably the most common prompt in the world, and it almost always
under-specifies the job. A summary is a selection: out of everything in the source, what
should survive? That depends entirely on who will read the summary and what they'll do
with it.

## Purpose first

Compare three summaries of the same 20-page vendor contract:

- For the **finance lead**: payment terms, price changes, penalties, total commitment.
- For the **engineering lead**: service levels, support hours, data export, integration
  obligations.
- For **legal**: liability caps, termination, jurisdiction, unusual clauses.

Same source, almost no overlap. A purpose-driven prompt:

```text
Summarize the contract in <contract> for our finance lead, who must decide whether to
sign by Friday. Cover: total cost over the term, payment schedule, price-increase
clauses, and any penalties or fees. For each point, cite the clause number.
If one of these topics isn't addressed in the contract, say "Not addressed".
Max 200 words.
```

## Extractive vs abstractive

- **Extractive** summaries reuse sentences or phrases from the source — safer, easier to
  verify, sometimes choppy.
- **Abstractive** summaries rephrase in new words — smoother, but each rephrasing is a
  chance to distort.

You can ask for a mix: "Summarize in your own words, but quote exact wording for any
numbers, dates, obligations, or commitments."

## Faithfulness

The main failure of summaries isn't style — it's **unfaithfulness**: adding a claim that
isn't in the source, merging two facts into a wrong one, dropping a qualifier ("may"
becomes "will"), or flipping a negation. Prompt-level defenses:

- "Include only information stated in the source; do not add background knowledge."
- "Preserve qualifiers such as *may*, *up to*, *except*, and *unless*."
- Ask for clause/paragraph references for each point.
- Use a separate verification prompt (below).

## A verification prompt

```text
Below are a source document (in <source>) and a summary of it (in <summary>).
For each sentence of the summary, answer:
- SUPPORTED: the source clearly states this
- PARTIAL: the source states something related but the summary changes or overstates it
- UNSUPPORTED: the source doesn't say this
Give the sentence number, the verdict, and the source sentence you compared it with.
```

This is not a guarantee — the checker is also a model — but separating writing from
checking catches many errors that a single pass misses (see Level 3 lesson 02 for the
biases of model-based checking).

## Long documents

When the source is too long to fit comfortably — or so long that details in the middle
get neglected — use **map-reduce**:

1. **Map:** split the document into sections; summarize each with the same
   purpose-driven prompt, keeping section references.
2. **Reduce:** combine the section summaries into the final summary, with the same
   audience and purpose.

A variant, **refine**, walks the document in order, updating a running summary with each
new section — better for narratives, slower because it's sequential. Chunking details
(where to split, overlap) are covered in Level 3 lesson 07 and, for retrieval systems,
the [RAG Mastery Path](https://sigilipelli.github.io/rag-mastery-path/).

## Worked example: meeting notes for someone who was absent

```text
I missed this meeting. From the transcript in <transcript>, tell me:
1. Decisions made (with who made them).
2. Anything assigned to me (I'm "Lee").
3. Open questions that are still unresolved.
4. Anything surprising or contentious, in one or two sentences.
Skip small talk and status updates that didn't lead to a decision.
If a section is empty, write "None".
```

The numbered questions define the selection criteria. "Anything assigned to me" is
something no generic summary would prioritize, and it's the part Lee needs most.

## How It Actually Works

A summary is generated token by token, conditioned on the source and the instruction.
The model doesn't first select sentences and then compress them; the selection emerges
from what the instruction makes relevant. Purpose and audience instructions change which
parts of the source receive the most attention and which facts the continuation draws on.

Unfaithfulness arises because abstractive rewriting leans on the model's general
language knowledge: when rephrasing, a common collocation or a typical phrasing can replace
the source's unusual one, and qualifiers and negations are especially fragile because
dropping them still yields fluent text. Requiring quotes for numbers and
obligations pins those tokens to the source text, where copying is much more reliable
than paraphrase.

For long inputs, the model's use of information can be uneven across positions, which is
why map-reduce — giving each section a turn at the start of a short context — improves
coverage of the middle of long documents.

## Common mistakes

- **"Summarize this"** with no audience or purpose.
- **No length or coverage target**, so the summary is either a sentence or a rewrite.
- **Losing qualifiers and negations.**
- **Trusting summaries of documents you haven't looked at** for high-stakes decisions.
- **Summarizing a summary of a summary** in long chains, compounding drift.

## Exercise

1. Take a long document you know well (a report, a policy, a long article).
2. Write two purpose-driven summary prompts for two different readers. Compare how little
   the outputs overlap.
3. Run the verification prompt on one summary. Check every PARTIAL or UNSUPPORTED verdict
   yourself — was the checker right?
4. Note one qualifier or negation that was lost or altered, if any, and add a rule to
   prevent it.
