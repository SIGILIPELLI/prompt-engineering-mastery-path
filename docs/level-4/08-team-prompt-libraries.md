---
description: "Build a shared team prompt library and style guide: entry format, naming, ownership, review, discoverability, and linting prompts for common issues."
---

# 08 · Team Prompt Libraries & Style Guides

When everyone on a team writes prompts independently, the same problems get solved five
times, good techniques stay in one person's notebook, and nobody knows which prompt
version the product runs. A **prompt library** makes good prompts shareable; a **style
guide** makes new prompts consistent; **ownership and review** keep both healthy.

## Two kinds of library

- **Personal-use library:** reusable prompts for everyday tasks (meeting summaries,
  email drafts, code review checklists) that people paste into chat tools. Lives in a
  wiki, shared doc, or the chat tool's saved-prompt feature.
- **Production library:** prompts that run inside products. Lives in version control
  with tests (lessons 01–02).

Both benefit from the same entry format.

## A library entry

```markdown
# Meeting notes → action items   (v1.4, owner: @ops-enablement)

**Use when:** you have raw notes or a transcript and need decisions + owners + dates.
**Don't use for:** legal or HR meetings (use the "confidential minutes" prompt).
**Inputs:** notes (paste in <notes>), your name (optional, to highlight your items).
**Output:** plain text, three sections; "None" when empty.
**Tested on:** 12 sample meetings (see link); known weakness: unnamed owners.

## Prompt
<the template, with {placeholders}>

## Example
Input excerpt → output excerpt (anonymized)

## Changelog
- 1.4 — added "unassigned" rule for missing owners
```

"Use when / don't use for", tested-on, and known weaknesses are what separate a library
from a pile of snippets.

## A prompt style guide

Keep it short — a page or two. Example conventions:

1. **Structure:** task first; context; material in XML-style tags; format last.
2. **Delimiters:** tags named for their content (`<email>`, `<policy>`), never generic
   (`<text1>`).
3. **Every "don't" has a "do instead".**
4. **Missing information:** every template states what to output when data is missing.
5. **No secrets or personal data** in templates; placeholders only.
6. **Placeholders:** `{snake_case}` names describing the content.
7. **Tone:** calm, specific instructions; no all-caps emphasis.
8. **Output contracts:** JSON outputs list every key, type, and allowed values.
9. **Metadata:** owner, version, model, settings, eval link for production prompts.
10. **Vendor neutrality:** avoid relying on one model's quirks unless the prompt is
    explicitly model-specific (and labelled so).

## Linting prompts

Some style-guide rules can be checked automatically — in CI for production prompts, or as
a quick script for the shared library:

```python
import re

RULES = [
    (r"\{[a-z_]*\d+[a-z_]*\}", "placeholder names should describe content, not number it"),
    (r"\b[A-Z]{5,}\b", "avoid all-caps emphasis"),
    (r"(?i)\b(api[_-]?key|password|secret)\s*[:=]", "possible secret in prompt"),
    (r"<(text|input|data)\d*>", "use descriptive tag names"),
]

def lint(name: str, text: str) -> list[str]:
    issues = [f"{name}: {msg}" for pattern, msg in RULES if re.search(pattern, text)]
    if "don't" in text.lower() and "instead" not in text.lower():
        issues.append(f"{name}: has a 'don't' rule but no 'instead' alternative")
    return issues

prompts = {
    "summary_v1": "Summarize <text1>{text1}</text1>. NEVER use bullet points. Don't be long.",
    "summary_v2": "Summarize the report in <report>{report_text}</report> in 3 sentences.",
}
for name, text in prompts.items():
    print(lint(name, text) or f"{name}: clean")
```

Output:

```text
['summary_v1: placeholder names should describe content, not number it', 'summary_v1: avoid all-caps emphasis', 'summary_v1: use descriptive tag names', "summary_v1: has a 'don't' rule but no 'instead' alternative"]
summary_v2: clean
```

Lint rules are heuristics; treat hits as prompts for review, not automatic rejections.

## Ownership, review and hygiene

- **Every entry has an owner** who answers questions and reviews changes.
- **Changes go through review** — a lightweight pull request or a doc suggestion — with
  evidence from the test cases.
- **Deprecate, don't delete:** mark outdated prompts as deprecated with a pointer to the
  replacement.
- **Review periodically**, especially after model changes; prompts written for older
  models often carry workarounds that are no longer needed.
- **Collect feedback:** a simple "worked / didn't work + example" form per entry surfaces
  problems quickly.

## Worked example: rolling out a library to a 20-person team

1. Collect the ten most-used prompts from people's notebooks (ask in a team meeting).
2. For each, write a task card and a small test set (Level 1 project format); keep the
   best version, merge good ideas from others.
3. Publish entries in one place with the standard format; announce with two examples of
   before/after improvements.
4. Assign owners; hold a 30-minute monthly review of feedback and changes.
5. Track adoption informally (which entries get used, which get feedback) and prune.

Wider organizational rollout — training, governance, and change management — is covered
in the [AI Tools Mastery Path](https://sigilipelli.github.io/ai-tools-mastery-path/).

## How It Actually Works

A library works for the same reasons code libraries do: it concentrates testing and
improvement effort on shared components, so one person's fix benefits everyone. A style
guide reduces variation between prompts, which makes them easier to read, review, and
debug — and consistent conventions (tag naming, missing-data rules, format-last ordering)
mean that techniques proven on one prompt transfer to others. Ownership prevents the
tragedy of the commons: without a named owner, shared prompts decay as models, products
and needs change.

## Common mistakes

- **A dump of snippets** with no "use when", tests, or owner.
- **Style guides too long to read.**
- **No path for feedback** from people using the prompts.
- **Deleting old versions** that someone still depends on.
- **Never revisiting prompts** after a model upgrade.

## Exercise

1. Gather five prompts you and colleagues use regularly.
2. Write library entries for each using the entry format.
3. Draft a one-page prompt style guide with 8–10 rules for your team.
4. Adapt the linter to two of your rules and run it on all five prompts; fix what it
   finds, or refine the rule if the hits were false alarms.
