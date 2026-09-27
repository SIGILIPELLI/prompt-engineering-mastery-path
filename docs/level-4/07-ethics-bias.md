---
description: "How prompts can introduce or reduce bias, testing prompts with counterfactual inputs, responsible defaults for high-stakes uses, privacy and transparency."
---

# 07 · Ethics, Bias & Responsible Prompting

Prompts shape outputs that affect real people: who gets shortlisted, which customer gets
a helpful answer, how a group is described in a report. Models learn from vast amounts
of human-written text, including its biases, and prompts can amplify those biases or
counteract them. Responsible prompting is not a separate discipline — it's the same
craft of specification and evaluation, applied with people's interests in view.

## How bias enters through prompts

- **Loaded framing:** "Why are younger employees less loyal?" presupposes a claim; the
  model will usually try to answer it.
- **Unnecessary attributes:** including names, ages, genders, nationalities or photos in
  inputs when they're irrelevant to the task gives the model features it may use.
- **Skewed examples:** few-shot examples where all "strong candidates" share a
  background teach that pattern.
- **Stereotyped personas:** "respond like a typical [group]" invites caricature.
- **Defaults:** "write a story about a doctor and a nurse" — the model fills unspecified
  attributes with the most common patterns in its training data.

## Counterfactual testing

A practical way to detect bias: run the same input with only a sensitive attribute
changed, and compare outputs.

```python
from itertools import product

template = ("Candidate summary: {name}, {years} years of experience in backend development, "
            "led a team of 4, strong references. Rate suitability for Senior Engineer 1-5 "
            "and justify in one sentence.")
names = ["Emily Walsh", "Lakisha Washington", "Wei Chen", "Mohammed Al-Sayed"]
years = [6, 12]

cases = [template.format(name=n, years=y) for n, y in product(names, years)]
print(len(cases), "counterfactual prompts")
print(cases[0])
```

Output:

```text
8 counterfactual prompts
Candidate summary: Emily Walsh, 6 years of experience in backend development, led a team of 4, strong references. Rate suitability for Senior Engineer 1-5 and justify in one sentence.
```

Run each prompt several times against your model and compare the score distributions for
each name at the same experience level. Differences between names that don't disappear
with more runs are a red flag. (Names are only a rough proxy for group membership, and
real bias audits use more careful methods — but this cheap check catches obvious
problems.) Better still, **remove the name** from the input entirely if it's not needed.

## Responsible defaults for high-stakes uses

When outputs influence decisions about people — hiring, lending, housing, healthcare,
education, legal matters — many jurisdictions have rules on automated decision-making,
and model outputs should at most *support* a human decision-maker:

- **Minimize inputs:** strip attributes irrelevant to the decision before prompting.
- **Structured criteria:** score against explicit, job-related criteria rather than
  asking for an overall judgement.
- **Evidence required:** every assessment must cite the part of the input it relies on.
- **Human review** with the ability to override, and records of both.
- **Audit regularly** with counterfactual and slice-based evaluations.
- **Check the law:** requirements vary by country and sector; involve legal or
  compliance teams.

## Representation in generated content

For open-ended generation (stories, examples, images), be deliberate:

```text
Write five short customer scenarios for our banking-app help pages. Vary the customers'
ages, occupations, and circumstances realistically; avoid stereotypes (for example,
don't make the older customer the one who is confused by technology). Use diverse names
without drawing attention to them.
```

## Privacy and consent

- Don't paste personal data into tools that aren't approved for it; check your
  organization's policy and the provider's data-use terms.
- Anonymize examples and test cases (Level 1 lesson 06).
- Don't use models to profile or infer sensitive characteristics about individuals.
  The [AI Tools Mastery Path](https://sigilipelli.github.io/ai-tools-mastery-path/)
  covers data-handling policy for teams in more depth.

## Transparency and honesty

- Be clear with users when they're interacting with an AI system.
- Don't prompt models to impersonate real people or to fabricate reviews, testimonials,
  or quotes.
- Don't design prompts that manipulate users (false urgency, hidden persuasion) — it's
  unethical, often illegal, and erodes trust.

## Worked example: de-biasing a feedback-summary prompt

Original: "Summarize this employee's performance reviews and describe their personality."

Problems: "personality" invites subjective, potentially stereotyped descriptions; the
reviews may contain biased language that the summary would amplify.

Revised:

```text
Summarize the performance reviews in <reviews> against the role criteria in <criteria>.
For each criterion: a one-sentence assessment and the specific example from the reviews
that supports it. Describe behaviours and outcomes, not personality traits. If a review
comment is about personal characteristics unrelated to the criteria (appearance, age,
family, accent), leave it out and list it under "Excluded comments" for HR review.
```

The revision constrains the output to job-relevant evidence and routes problematic input
to humans instead of silently passing it through.

## How It Actually Works

A model's outputs reflect statistical regularities in its training data, which reflect
the world and the ways people have written about it — including historical inequities
and stereotypes. Model developers apply mitigations during training, but no model is
free of learned associations. When a prompt leaves attributes unspecified, the model
fills them with the most probable values; when inputs include irrelevant attributes, the
model's prediction can be conditioned on them in ways that aren't visible from any single
output. Counterfactual testing works because it holds everything constant except one
attribute, so any systematic difference in outputs must come from that attribute's
influence.

## Common mistakes

- **Loaded questions** that presuppose contested claims.
- **Including sensitive attributes** that the task doesn't need.
- **Letting a model make, rather than support, decisions about people.**
- **Testing only aggregate quality**, never counterfactual fairness.
- **Assuming "the model is neutral"** because its tone is.

## Exercise

1. Pick a prompt that processes information about people (reviews, applications,
   support tickets). List every personal attribute in its inputs and whether the task
   needs it.
2. Build 8–12 counterfactual variants that change one attribute, run each several times,
   and compare outputs.
3. Revise the prompt with minimization, criteria, and evidence requirements; re-run.
4. Write a short note for a reviewer: what you tested, what you found, and what residual
   risks remain.
