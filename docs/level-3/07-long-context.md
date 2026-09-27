---
description: "Strategies for prompts with very long inputs: where to put instructions, structuring documents, quote-first answering, chunking, and when to retrieve instead."
---

# 07 · Long-Context Strategies

Context windows have grown enormously; you can now paste whole books, codebases, or
hundreds of pages of reports into a single prompt. But fitting in the window is not the
same as being used well. With very long inputs, models can miss details, confuse
documents, or follow the wrong instruction. This lesson covers how to prompt with long
contexts effectively.

## Known behaviours of long contexts

- **Position effects:** research on long-context use (sometimes summarized as "lost in
  the middle") has found models often use information near the beginning or end of a
  long input more reliably than information in the middle. Newer models have improved
  here, but it's still worth designing around.
- **Distraction:** irrelevant material competes for attention and can reduce accuracy
  even when the relevant passage is present.
- **Instruction dilution:** instructions at the top of a very long prompt become a tiny
  fraction of it.
- **Cost and latency** scale with input length (Level 4 lesson 03).

## Placement

A widely recommended pattern for long inputs:

```text
[brief task statement]

<documents>
  <document index="1" source="Q3-board-report.pdf"> ... </document>
  <document index="2" source="Q3-finance-appendix.xlsx (as CSV)"> ... </document>
</documents>

[full instructions and the question, restated here, after the material]
```

Putting the question after the documents means the model reads the material and then
immediately encounters what to do with it. Several providers' documentation recommends
this ordering for long inputs; test it on your task.

## Structure the material

- Wrap each document in its own tags with metadata (source, date, author).
- Give sections numbers or IDs so the model can cite them.
- Remove boilerplate (navigation text, repeated headers and footers, legal footers on
  every page) before pasting.
- Convert tables to a clean text format (CSV or Markdown) where possible.

## Quote first, then answer

For question answering over long documents, ask the model to first extract the relevant
passages, then answer from them:

```text
First, find the passages in <documents> most relevant to the question and copy them
word for word inside <quotes>, each with its document index. If nothing is relevant,
write <quotes>none</quotes>.
Then answer the question in <answer> using only the quotes. If the quotes are "none",
say the documents don't answer the question.

Question: What reasons did the board give for delaying the Lisbon office?
```

This narrows the model's focus to a small set of text before it composes the answer, and
gives you verifiable evidence (you can check the quotes exist in the source).

## When the input doesn't fit — or shouldn't

If the material exceeds the window, or you're asking many questions over a large
corpus, don't paste everything:

- **Map-reduce** (Level 2 lesson 07): process chunks independently, then combine.
- **Retrieval (RAG):** index the documents and include only the relevant chunks per
  question. Covered in depth in the
  [RAG Mastery Path](https://sigilipelli.github.io/rag-mastery-path/).
- **Prompt caching:** if the same long prefix is reused across many requests, several
  providers can cache it to cut cost and latency (Level 4 lesson 03).

## Test long-context behaviour directly

A simple "needle" test: insert a specific, unusual fact at different positions in a long
document and ask for it. This script builds such test documents so you can run them
against your model:

```python
filler = [f"Paragraph {i}: routine operational update with no notable changes."
          for i in range(1, 201)]
needle = "Note: the backup generator test is rescheduled to 14 October."

tests = []
for position in (0.1, 0.5, 0.9):
    doc = filler.copy()
    idx = int(len(doc) * position)
    doc.insert(idx, needle)
    tests.append({"position": position, "index": idx, "document": "\n".join(doc)})

for t in tests:
    print(t["position"], t["index"], len(t["document"].split()), "words")
```

Output:

```text
0.1 20 1810 words
0.5 100 1810 words
0.9 180 1810 words
```

Real documents are more varied than this filler, and needle tests measure retrieval of
one fact, not reasoning across many — so treat them as a smoke test. The more realistic
test is your own task, with the key fact placed at different positions.

## Worked example: comparing two long contracts

```text
Two supplier contracts are in <contract_a> and <contract_b>.

For each topic below, quote the relevant clause from each contract (with clause number),
then state the practical difference in one sentence. If a contract has no clause on the
topic, write "not addressed".

Topics: payment terms; termination notice; liability cap; data ownership.

Output a table: Topic | Contract A (quote) | Contract B (quote) | Difference
```

Per-topic quoting forces the model to locate each clause in each document explicitly —
instead of relying on a general impression — and keeps the two documents from blending.

## How It Actually Works

Attention lets each generated token draw on every earlier token, but the model has to
*learn* where to look, and the relevant signal becomes harder to find as the haystack
grows. Training data also shapes position effects: in most documents, important
information clusters at the start (titles, abstracts) and end (conclusions), and the
recency of the instructions right before the answer makes them salient. Models trained
specifically for long contexts reduce but don't remove these effects.

Quote-first prompting works because copying is a strong, reliable model behaviour:
once relevant passages are reproduced near the end of the context, the answer is being
generated right next to its evidence, rather than from a passage 100,000 tokens back.

## Common mistakes

- **Pasting everything** because it fits.
- **Instructions only at the top** of a very long prompt.
- **Unlabelled documents** that the model blends together.
- **Leaving in boilerplate** that inflates length and distracts.
- **Assuming a large window means perfect recall.**

## Exercise

1. Assemble a long document from your own work (or combine several public reports).
2. Write five questions whose answers sit at different positions — beginning, middle,
   end — and one that isn't answered at all.
3. Ask them with (a) instructions only at the top and (b) documents first, question
   restated after, with quote-first answering.
4. Check the quotes exist in the source and compare accuracy between (a) and (b).
