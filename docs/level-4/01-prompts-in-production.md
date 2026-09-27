---
description: "Manage prompts like code in production: storing them as files, semantic versioning, metadata, code review, staged rollout, rollback and change logs."
---

# 01 · Prompts as Production Artifacts

In a prototype, a prompt is a string in the code. In production, that string determines
what thousands of users see, and changing one word can change behaviour across the whole
product. Mature teams manage prompts with the same discipline as code: versioned,
reviewed, tested, released gradually, and reversible.

## Store prompts as files, not scattered strings

Keep each prompt in its own file, next to (or separate from) the code that uses it:

```text
prompts/
  ticket_classifier/
    prompt.txt          # the template
    meta.yaml           # version, owner, model, settings, changelog
    tests.jsonl         # the regression set (lesson 02)
  reply_drafter/
    system.txt
    user.txt
    meta.yaml
    tests.jsonl
```

Benefits: prompt changes show up as readable diffs in code review; non-engineers can
propose edits; tests live beside what they test; and you can find every prompt in the
product with one directory listing.

Some teams use a hosted prompt-management tool instead, which adds a UI, versioning and
deployment without code releases. The principles below apply either way.

## Metadata that travels with the prompt

```yaml
name: ticket_classifier
version: 2.3.0
owner: support-platform team
model: <provider/model-name, pinned version>
settings: {temperature: 0, max_output_tokens: 200}
inputs: [ticket_text]
output: JSON {reason, category, priority}
eval: tests.jsonl — last run 2026-09-20, category acc 0.94, priority acc 0.91
changelog:
  - 2.3.0: added rule for login+payment tickets (fixes #412)
  - 2.2.1: wording fix, no behaviour change expected (eval unchanged)
  - 2.0.0: output changed from plain label to JSON (breaking)
```

A prompt's behaviour depends on the **combination** of text, model version, and settings.
Record all three; a prompt that was tested on one model version is an untested prompt on
another.

## Versioning

Semantic versioning adapts well:

- **Major:** the output contract changes (new format, new fields, removed labels) —
  downstream code must change.
- **Minor:** behaviour changes intentionally (new rule, new example) with the same
  contract.
- **Patch:** wording clean-up expected to have no behavioural effect — *verified* by
  running the eval.

Log the prompt version with every model call, so any output in production can be traced
to the exact prompt that produced it.

## Review

Prompt changes deserve code review, with prompt-specific questions:

- What failure does this change fix? Is there a test case for it?
- What else could it affect? (Rules often have side effects on other slices.)
- Did the eval run, and how did per-slice results change?
- Does it add length (cost, latency) — and is that justified?
- Does it introduce anything sensitive (internal names, data) into the prompt?

## Release and rollback

- **Staged rollout:** serve the new version to a small share of traffic first, watching
  quality and guardrail metrics (Level 3 lesson 03).
- **Feature flags / config:** select the prompt version by configuration so you can
  switch without a code deploy.
- **Rollback:** keep the previous version deployable; rolling back should be a config
  change.
- **Model upgrades are releases too:** a provider's new model version is a change to
  every prompt using it — run the full regression suite before switching (lesson 02).

## Worked example: a small prompt loader

```python
import hashlib
from pathlib import Path
import tempfile

def load_prompt(root: Path, name: str) -> dict:
    """Load a prompt and derive a content hash, so logs record exactly what ran."""
    text = (root / name / "prompt.txt").read_text(encoding="utf-8")
    version = (root / name / "VERSION").read_text(encoding="utf-8").strip()
    digest = hashlib.sha256(text.encode("utf-8")).hexdigest()[:12]
    return {"name": name, "version": version, "hash": digest, "text": text}

with tempfile.TemporaryDirectory() as d:
    root = Path(d)
    (root / "ticket_classifier").mkdir()
    (root / "ticket_classifier" / "prompt.txt").write_text("Classify: {ticket}\n", encoding="utf-8")
    (root / "ticket_classifier" / "VERSION").write_text("2.3.0\n", encoding="utf-8")
    p = load_prompt(root, "ticket_classifier")
    print(p["name"], p["version"], p["hash"])
```

Output:

```text
ticket_classifier 2.3.0 934948c0e573
```

The hash catches the classic mistake of editing a prompt without bumping its version:
if two logs show the same version but different hashes, someone changed the text.

## How It Actually Works

A prompt in production is effectively configuration that controls a probabilistic
component. Small textual changes can shift the output distribution in ways that aren't
visible from the diff — a new rule meant for one case subtly changes how the model
handles others. Code-style practices work because they make changes *explicit*
(diffs), *attributable* (versions, owners, hashes), *verifiable* (tests), and
*reversible* (rollback). Recording the model version alongside the prompt matters
because the "program" is really the pair (prompt, model); upgrading the model changes the
interpreter under your program.

## Common mistakes

- **Prompts embedded in many code files**, impossible to audit.
- **No model version pinned** in the prompt's metadata.
- **"Tiny wording changes"** shipped without an eval run.
- **No per-call logging** of prompt version.
- **Rollback that requires a code deploy.**

## Exercise

1. Move one prompt you own (or your Level 2 project prompts) into the directory
   structure above with `meta.yaml` including a changelog.
2. Implement `load_prompt` (or similar) and log name, version and hash with each call.
3. Make a patch-level wording change, run your eval, and record in the changelog whether
   behaviour actually stayed the same.
4. Write a one-paragraph rollout and rollback plan for your next minor change.
