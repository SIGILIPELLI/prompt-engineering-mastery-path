---
description: "How to use few-shot examples in prompts: choosing diverse examples, formatting them consistently, avoiding copying and label bias, and zero- vs few-shot."
---

# 06 · Few-Shot Examples

Describing what you want is hard. Showing it is often easier. A **few-shot prompt**
includes a handful of worked examples — input paired with the desired output — before
the real input. It is one of the most reliable techniques there is, especially for
format, tone, and fuzzy categories that are hard to define in words.

"Zero-shot" means no examples; "one-shot" means one; "few-shot" usually means two to
roughly ten.

## A basic few-shot prompt

```text
Rewrite each product review as a one-line, neutral summary of the main point.

Review: "Honestly the best kettle I've owned!!! Boils super fast but the lid is a bit stiff."
Summary: Boils quickly; lid is stiff.

Review: "Arrived broken. Customer service replaced it within 2 days though, so 3 stars."
Summary: Arrived damaged; replacement was fast.

Review: "It's fine. Does what it says. Kinda loud."
Summary: Works as described; noisy.

Review: "{new review}"
Summary:
```

Notice what the examples teach without saying it: summaries are short, drop emotion,
use a semicolon to join two points, and don't include star ratings. Writing all of that
as rules would take longer and still be less clear.

## Choosing examples

Good example sets are:

- **Representative** — they look like the real inputs you'll get, including messy ones.
- **Diverse** — they cover different cases (positive, negative, mixed; short, long). If
  all examples are positive reviews, the model may struggle with negative ones.
- **Correct** — a wrong example is a strong wrong signal. Check them carefully.
- **Edge-aware** — include one tricky case and show how you want it handled (e.g. a
  review that is sarcastic, or empty).

## The two big risks

**1. Copying.** Models tend to imitate surface features of the examples: length,
opening words, even specific phrases. If every example summary starts with "Works…", many
outputs will too. Vary the surface of your examples so only the features you want are
consistent.

**2. Label and order bias.** In classification, if four of your five examples are
labelled `positive`, outputs may skew positive. The last example can also have extra
influence. Balance labels across examples and shuffle their order; if possible, test
more than one ordering.

## Formatting examples consistently

Use one exact pattern for every example and for the final input. The final line should
set up the model to produce only the part you want:

```text
Input: ...
Output: ...

Input: ...
Output: ...

Input: {real input}
Output:
```

In chat interfaces you can also present examples as tagged blocks:

```text
<example>
<input>...</input>
<output>...</output>
</example>
```

Tell the model the examples are illustrations: "The examples show the format and level
of detail; don't reuse their content."

## Zero-shot or few-shot?

Modern instruction-tuned models handle many tasks zero-shot. Try zero-shot first when the
task is common and easy to describe ("translate to French"). Add examples when:

- the output format is unusual or strict;
- the categories are fuzzy or specific to your organisation;
- the tone is hard to describe ("like our brand voice");
- zero-shot outputs vary in ways a description hasn't fixed.

Examples cost tokens on every request. In applications that run a prompt thousands of
times, that cost is real (Level 4 lesson 03).

## Worked example: fixing an inconsistent classifier

Zero-shot prompt:

```text
Classify this support message as billing, technical, or account.
Message: "I can't log in since I changed my card details"
```

This message plausibly fits all three categories. Different runs may label it
differently, and that's a sign your categories need a decision, not just a model.
Decide the rule ("login problems are `account`, even if payment is mentioned"), then
teach it with examples that include that exact ambiguity:

```text
Classify each support message into exactly one category: billing, technical, account.
Answer with the category word only.

Message: "Why was I charged twice this month?"
Category: billing

Message: "The export button does nothing when I click it."
Category: technical

Message: "I can't log in after updating my payment method."
Category: account

Message: "How do I change the email on my profile?"
Category: account

Message: "The invoice PDF won't download, it just spins."
Category: technical

Message: "{message}"
Category:
```

Labels are balanced-ish, one example covers the ambiguous case, and the final line asks
for exactly one word.

## How It Actually Works

Few-shot prompting relies on what researchers call **in-context learning**: the model's
ability to pick up a pattern from examples in its context and continue it, without any
change to its weights. Mechanistically, attention lets the model compare the new input
with the earlier inputs and draw on what followed them. Researchers have identified
attention patterns (sometimes called induction heads) that implement a basic form of
"find where this appeared before, and copy what came next".

That copying mechanism explains both the power and the risks. Examples are a very strong
signal for format and style because format is exactly the kind of surface pattern that
copying picks up. It's also why label imbalance, repeated phrases, and example order leak
into outputs. Studies of in-context learning have found that the *format* and *label
space* of examples often matter more than whether each example label is perfectly
correct — but wrong labels still hurt, so don't rely on that.

## Common mistakes

- **All examples too similar**, so the model can't generalize to other kinds of input.
- **Unbalanced labels** in classification examples.
- **Examples that contradict the instructions** (instructions say "one sentence",
  example has three). Examples usually win.
- **Leaking real data**: using real customer messages as examples in a shared prompt.
  Anonymize them.
- **Too many examples** for a simple task, wasting tokens and over-constraining style.

## Exercise

1. Pick a small classification or rewriting task you actually have.
2. Write a zero-shot prompt and collect 10 realistic test inputs, including 2 tricky ones.
3. Run zero-shot on all 10 and note errors.
4. Add 3–5 examples chosen to cover the errors you saw, with balanced labels and varied
   wording. Re-run on the same 10 inputs.
5. Shuffle the example order and run once more. Did any answers change?
