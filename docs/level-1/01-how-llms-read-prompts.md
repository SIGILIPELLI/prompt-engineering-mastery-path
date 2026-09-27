---
description: "An intuitive model of how LLMs process a prompt: tokens, the context window, next-token prediction, sampling, and what that means for how you write."
---

# 01 · How a Model Reads Your Prompt

You do not need to understand transformer mathematics to write good prompts. You do
need a working mental model of what happens between pressing Enter and seeing words
appear, because almost every piece of prompting advice in this course follows from it.
This lesson builds that model in four pieces: tokens, the context window, next-token
prediction, and sampling.

## Piece 1: text becomes tokens

A model does not read letters or words. Before anything else, your text is split into
**tokens** — chunks drawn from a fixed vocabulary of tens of thousands of pieces. Common
English words are often a single token; rarer words, names, and code identifiers are
split into several; spaces and punctuation are usually attached to neighbouring pieces.

A rough, illustrative split (the exact split depends on the model's tokenizer):

```text
"Summarize the quarterly report"   ->  ["Summ", "arize", " the", " quarterly", " report"]
"antidisestablishmentarianism"     ->  ["ant", "idis", "establish", "ment", "arian", "ism"]
"customer_id_v2"                   ->  ["customer", "_id", "_v", "2"]
```

Practical consequences:

- **Length is measured in tokens, not words.** For English prose a common rule of thumb
  is roughly three-quarters of a word per token, but code, numbers, and non-English
  text can use noticeably more tokens per word. Limits and prices are per token.
- **Character-level tasks are awkward.** "How many r's are in *strawberry*?" or "reverse
  this string" ask about units the model never directly sees. Models have improved at
  these, but they remain a classic weak spot. If you need exact character operations,
  use code.
- **Spelling and spacing can matter.** `JSON` and ` json` may be different tokens. This
  rarely matters for everyday prompts but explains occasional oddities with formats.

## Piece 2: the context window is the model's entire world

Everything the model uses to produce a reply must fit in its **context window**: the
system instructions, the conversation so far, any documents you pasted, and the reply
being generated. Modern windows are large — often hundreds of thousands of tokens —
but they are finite, and more importantly *the model has nothing else*.

It does not remember your previous chat unless the application re-sends it. It cannot
see a file you mentioned but did not paste. It does not know your company's naming
conventions or your manager's preferences. When a model "ignores" something, the first
question to ask is: *was it actually in the context?*

```text
Context window (one request)
+-------------------------------------------------------------+
| system / instructions | conversation history | your message |  -> reply tokens
+-------------------------------------------------------------+
```

In chat applications, older messages may be trimmed or summarized when a conversation
gets long, so something you said an hour ago may genuinely be gone.

## Piece 3: the model predicts the next token

At its core, a language model does one thing: given a sequence of tokens, it produces a
probability for every token in its vocabulary being the *next* one. To generate a reply,
the system picks a token, appends it, and asks again — one token at a time until it
produces a stop signal or hits a length limit.

Instruction-tuned assistants have been further trained so that the most likely
continuation of "a user asked X" is a helpful answer to X. But the mechanism is still
continuation. That is why:

- **The beginning of an answer steers the rest.** Once the model has written "There are
  three main reasons:", it is now very likely to produce three reasons.
- **Patterns in the prompt get continued.** If your examples are formatted as
  `Input: … / Output: …`, the model tends to keep that format.
- **The model has no separate "fact lookup".** It produces text that is plausible given
  its training and the context. Plausible and true usually coincide for well-known
  facts; they diverge for obscure details, recent events, and specifics like citations,
  which is where hallucinations come from (lesson 09).

## Piece 4: sampling adds variation

The model produces probabilities; something must choose a token. With **temperature** at
or near zero, the system mostly picks the highest-probability token, so outputs are more
repeatable. Higher temperatures flatten the distribution so less likely tokens are picked
more often, producing more varied — sometimes more creative, sometimes more erratic —
text. Chat interfaces usually hide this setting; APIs expose it. Level 3 lesson 09 covers
sampling properly.

The key point for now: **the same prompt can give different answers on different runs.**
Judging a prompt by a single output is like judging a die by a single roll.

## Worked example: reading a prompt the way the model does

Consider this prompt:

```text
write something about our new product for the newsletter
```

Walk through it with the four pieces in mind:

1. *Tokens:* short, cheap, fine.
2. *Context:* the model does not know what "our" company is, what the product is, who
   reads the newsletter, or how long "something" should be. None of that is in the
   window.
3. *Prediction:* the most plausible continuation of such a request, with no specifics,
   is generic marketing copy — enthusiastic adjectives, invented features, perhaps a
   placeholder like "[Product Name]".
4. *Sampling:* run it three times and you will likely get three different generic
   pieces.

Now a version written with the mechanism in mind:

```text
Write a 120-150 word item for our monthly customer newsletter announcing a new product.

About us: Brightline is a small company that sells refillable cleaning products online.
Readers: existing customers who already buy our refills; friendly, practical tone.

Product facts (use only these, do not invent others):
- Name: Brightline Glass Spray Refill, 1 litre
- Refills the existing spray bottle 2 times
- Same formula as our current glass spray
- Available from 3 March

End with one sentence telling readers where to find it: the "Refills" page of our website.
```

Every added line fills a gap the model would otherwise fill by guessing. The explicit
"use only these" line reduces invented features. The length target gives the model a
concrete stopping point.

## How It Actually Works

Inside the model, each token is turned into a vector (a list of numbers). A stack of
**attention** layers then lets every position look back at every earlier position and
mix in information from the ones that seem relevant. By the final layer, the vector at
the last position encodes a compressed picture of "everything so far, as it bears on
what comes next", and a final layer turns it into scores over the vocabulary.

Two properties of this matter for prompting:

- **Attention is learned, not guaranteed.** The model decides from patterns in training
  which earlier tokens matter. Clear structure — headings, labels, delimiters — makes
  it easier to attend to the right parts. A key instruction buried in a long paragraph
  competes with everything around it.
- **Generation is causal.** Each new token can only look backwards. The model cannot
  plan the whole answer and then write it; any "planning" happens through the text it
  has already produced. This is why asking a model to reason *before* it answers can
  help on some tasks (Level 2 lesson 01), and why an answer that starts badly rarely
  recovers.

Finally, the model's knowledge comes from training data up to some cutoff date, plus
whatever you put in the context. It has no live connection to the world unless the
application gives it tools such as search.

## Common mistakes

- **Assuming shared memory.** Referring to "the document" or "like last time" when
  neither is in the current context.
- **Counting in words.** Setting a hard limit of "under 4,000 tokens" for pasted
  material but measuring in words, then being surprised when it is cut off.
- **Trusting one sample.** Deciding a prompt "works" (or doesn't) from one reply.
- **Asking for character-exact operations** (counting letters, exact string reversal)
  and relying on the answer without checking.
- **Treating fluent output as verified output.** Fluency is what the model is best at;
  it says nothing about accuracy.

## Exercise

1. Take a prompt you used recently that gave a disappointing result. List every piece of
   information the model would have needed that was not actually in the context.
2. Rewrite the prompt to include that information.
3. Run the old and new versions three times each. Write two sentences on how much the
   outputs varied between runs of the *same* prompt, compared with the difference
   between the two prompts.
4. Optional: if your model provider publishes a tokenizer tool, paste a paragraph of
   prose and a paragraph of code and compare their token counts per word.
