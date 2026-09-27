---
description: "Level 4 of prompt engineering: prompts in production, regression tests, cost and latency, agents, domain prompting, ethics, team libraries, RAG vs fine-tuning."
---

# Level 4 · Master <span class="level-badge">Production & Practice</span>

Goal: treat prompts as long-lived production artifacts — versioned, tested, costed,
reviewed and shared — and develop the judgement to know when a prompt isn't the right
tool at all.

By now you can design, evaluate and defend a prompt. Level 4 is about everything around
the prompt when it lives inside a product or a team for months: managing versions and
model upgrades, catching regressions, balancing quality against cost and latency,
writing prompts for agents that act over many steps, adapting technique to specific
domains (code, data analysis, writing), handling bias and ethical risk, and building a
shared library and style guide so a whole team prompts well. It closes with the question
every prompt engineer eventually faces — should this be solved with retrieval or
fine-tuning instead? — and a capstone that packages a production-ready prompt end to end.

## Modules

1. [Prompts as Production Artifacts](01-prompts-in-production.md) — storing, versioning, reviewing and releasing prompts
2. [Regression Testing Prompts](02-regression-testing.md) — golden sets, CI checks, and surviving model upgrades
3. [Cost & Latency Tradeoffs](03-cost-latency.md) — where tokens and time go, and prompt-level levers to reduce them
4. [Prompting for Agents](04-prompting-agents.md) — instructions for models that plan, use tools and act over many steps
5. [Prompting for Code](05-prompting-for-code.md) — specifying, generating, reviewing and testing code with models
6. [Prompting for Data Analysis & Writing](06-data-and-writing.md) — two domains with their own traps and techniques
7. [Ethics, Bias & Responsible Prompting](07-ethics-bias.md) — how prompts introduce or reduce bias, and responsible defaults
8. [Team Prompt Libraries & Style Guides](08-team-prompt-libraries.md) — sharing prompts, conventions, ownership and review
9. [When Not to Prompt: RAG & Fine-Tuning](09-beyond-prompting.md) — choosing between prompting, retrieval and fine-tuning
10. [Capstone — A Production Prompt Package](10-capstone.md) — a complete, tested, documented prompt ready to ship

## Before you start

- Levels 1–3, especially evaluation (L3·01–03) and the eval harness project (L3·10),
  which Level 4 reuses.
- Basic familiarity with Git is helpful for lessons 01–02 and 08; the
  [GitHub & Git Mastery Path](https://sigilipelli.github.io/github-mastery-path/) covers it.
