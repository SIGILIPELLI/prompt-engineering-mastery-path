---
description: "How to write system prompts: what belongs there versus in the user turn, a practical section structure, priorities, and keeping them testable."
---

# 04 · System Prompts

Most chat APIs separate messages into roles: a **system** message (sometimes called a
developer message or instructions), then alternating **user** and **assistant** messages.
The system prompt sets standing instructions for the whole conversation — who the
assistant is for, what it should and shouldn't do, and how it should format answers.
Consumer chat apps expose a version of this as "custom instructions" or project
instructions.

## What belongs where

| System prompt | User turn |
|---------------|-----------|
| Stable facts about the product, audience, and purpose | The specific request |
| Behaviour rules that apply to every turn | Material for this request |
| Default output format and tone | One-off format changes |
| How to handle out-of-scope requests | Examples specific to this request |

A good test: if it would be the same for every user and every message, it's a system
prompt candidate. If it changes per request, it belongs in the user turn.

## A practical structure

There's no official format, but organized sections are easier for models to follow and
for humans to maintain:

```text
# Purpose
You are the help assistant inside "Ledgerly", a bookkeeping app for small businesses.
Users are business owners, usually not accountants.

# What you can help with
- How to use Ledgerly features (invoices, expenses, bank feeds, reports).
- General explanations of bookkeeping terms.

# What you must not do
- Give tax or legal advice specific to the user's situation. Instead, explain the general
  concept and suggest they confirm with an accountant.
- Guess at features. If you don't know whether Ledgerly can do something, say so and
  point to the Help Centre.

# Style
- Plain language. Define accounting terms on first use.
- Short answers first (2–4 sentences); offer detail only if asked.
- Use numbered steps for any how-to instructions.

# Reference
<feature_list>
...current features, one per line...
</feature_list>
```

## Writing rules that hold up

- **Explain the why.** "Don't give personalised tax advice, because users may act on it
  without an accountant and rules vary by location" helps the model generalize to cases
  your rule didn't list.
- **Give the alternative.** Every "don't" should say what to do instead.
- **Prioritize explicitly** if rules can conflict: "If brevity conflicts with safety
  information, include the safety information."
- **Keep it as short as it can be.** Every sentence competes for attention; remove rules
  that never change behaviour on your test set.
- **Avoid shouting.** ALL-CAPS and "CRITICAL!!!" were a common workaround for older,
  less attentive models. With current models they can cause over-application (the model
  applies the rule where it shouldn't). Calm, specific wording usually works better.

## The system prompt is not a security boundary

Users can ask the model to reveal or ignore its instructions, and content the model reads
(documents, web pages) can contain instructions too. Models are trained to give system
instructions priority, but that priority is a tendency, not a guarantee. So:

- Don't put secrets (API keys, internal URLs, private data) in a system prompt.
- Don't rely on "never reveal these instructions" for anything that matters.
- Enforce real permissions in code, not in prose (Level 3 lessons 04–05).

## Worked example: fixing a system prompt with a test

Symptom: users ask "Can I deduct my car?" and the assistant gives a confident,
jurisdiction-specific answer.

Old rule:

```text
Don't give tax advice.
```

The model reads "Can I deduct my car?" as a general question and answers. Revised rule:

```text
- Questions about whether something is deductible, taxable, or allowed are personal tax
  questions. Explain the general idea in one or two sentences (for example, what
  "deductible expense" means), then say that the answer depends on their location and
  circumstances and recommend confirming with an accountant. Don't state that a specific
  item is or isn't deductible.
```

Then add "Can I deduct my car?", "Is my home office deductible?" and two similar
questions to the test set, and check them every time the system prompt changes.

## How It Actually Works

Under the hood, the API formats all messages into one token sequence using a chat
template with special role markers; the model sees the system text first, then the
conversation. During instruction tuning, models are trained on conversations where
system-level instructions should persist across turns and generally take precedence over
conflicting user requests. Some model developers describe an explicit **instruction
hierarchy** (platform policies, then developer/system instructions, then user
instructions, with content from tools and documents treated as data) and train models to
follow it.

Because this is learned behaviour, it's strong but imperfect, and it can weaken over very
long conversations as the system prompt becomes a smaller fraction of the context. That
is why testing, restating critical rules near where they apply, and enforcing hard limits
in code remain important.

## Common mistakes

- **Putting per-request data in the system prompt** so it goes stale.
- **Rules without reasons or alternatives.**
- **Contradictions between sections** ("be concise" and "always explain fully").
- **Secrets in the system prompt.**
- **Changing it without re-running tests.** Small edits can have broad effects.

## Exercise

1. Write a system prompt for an assistant you'd find useful (a study helper for one
   subject, a writing assistant for your team, a helper for a hobby club).
2. Use the section structure above; include at least two "don't" rules with reasons and
   alternatives.
3. Write 8 test messages: 4 normal, 2 out of scope, 2 that tempt it to break a rule.
4. Run all 8; revise the prompt for any failures; re-run all 8 after each change.
