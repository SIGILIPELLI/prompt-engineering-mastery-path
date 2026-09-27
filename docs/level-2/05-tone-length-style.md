---
description: "Precise control of tone, length and style: style specs, voice samples, reading-level targets, banned-phrase lists, and checking style with simple metrics."
---

# 05 · Controlling Tone, Length & Style

"Make it sound less robotic" is one of the most common requests people make of models —
and one of the least effective. Style is controllable, but only when you describe it in
terms the model can act on. This lesson turns taste into specifications.

## Describe style along dimensions

Instead of one adjective, specify several concrete dimensions:

| Dimension | Example setting |
|-----------|-----------------|
| Formality | "Conversational; contractions are fine; no slang" |
| Sentence length | "Mostly short sentences; vary length; max ~25 words" |
| Person & address | "Second person ('you'); we = the company" |
| Vocabulary | "Everyday words; no buzzwords like 'leverage', 'synergy', 'seamless'" |
| Warmth | "Friendly but not effusive; no exclamation marks" |
| Structure | "Lead with the point; one idea per paragraph" |
| Confidence | "State facts plainly; hedge only where genuinely uncertain" |

A compact **style spec** combining these is reusable across prompts and people.

## Show a voice sample

Descriptions only go so far. A short sample of the target voice is often the strongest
signal:

```text
Write the product update in the same voice as the sample in <voice_sample>: match its
sentence length, level of formality and humour. Do not reuse its content or phrases.

<voice_sample>
We moved the export button. It was hiding in Settings, which, in hindsight, was a
strange place to keep it. It now lives at the top of every report.
</voice_sample>
```

The instruction not to reuse phrases matters; otherwise models tend to borrow
distinctive lines from the sample.

## Reading level and audience

Reading-level targets are useful when paired with an audience:

```text
Rewrite for adults with no medical background, at roughly the reading level of a
general-interest newspaper. Keep all numbers and dosing instructions exactly as written.
```

Note the second sentence: when simplifying, specify what must not change.

## Length, revisited

From Level 1: prefer structural limits. For longer pieces, set a length per section:

```text
Intro: 2 sentences. Three sections of ~80 words each, with a one-line heading.
No conclusion section.
```

When a model keeps writing too much, the cause is often *content* scope, not wording:
tell it what to leave out ("skip background; the reader knows the product").

## Avoiding the "assistant voice"

Many people recognize a generic AI register: stock openings, inflated adjectives,
forced summaries, lists of three everywhere. To steer away from it:

- Ban specific phrases you dislike *and* describe the replacement behaviour.
- Ask for concrete details from the provided material rather than general claims.
- Ask for the draft to open with the most specific fact available.
- Provide a real sample of writing you like.

Banned-phrase lists work best when short and specific; a very long list can itself
become a distraction.

## Measuring style with code

Some style properties are measurable. This script reports sentence lengths, exclamation
marks, and banned words — useful when you're comparing prompt versions across many
outputs:

```python
import re

BANNED = {"leverage", "seamless", "delve", "synergy", "game-changer"}

def style_report(text: str) -> dict:
    sentences = [s for s in re.split(r"(?<=[.!?])\s+", text.strip()) if s]
    words = re.findall(r"[A-Za-z'-]+", text.lower())
    lengths = [len(re.findall(r"[A-Za-z'-]+", s)) for s in sentences]
    return {
        "sentences": len(sentences),
        "avg_words_per_sentence": round(sum(lengths) / len(lengths), 1),
        "longest_sentence": max(lengths),
        "exclamations": text.count("!"),
        "banned_found": sorted(BANNED & set(words)),
    }

draft = ("Our new dashboard lets you leverage real-time insights! "
         "It is fast. Reports load in under two seconds, even for large accounts, "
         "and you can share them with one click.")
print(style_report(draft))
```

Output:

```text
{'sentences': 3, 'avg_words_per_sentence': 9.7, 'longest_sentence': 18, 'exclamations': 1, 'banned_found': ['leverage']}
```

Metrics like these don't capture quality, but they turn vague complaints ("too
salesy") into something you can track across versions.

## Worked example: one message, three audiences

Facts: scheduled maintenance, Saturday 02:00–04:00 UTC, the app will be read-only,
no data will be lost.

```text
Write three versions of a maintenance notice using only the facts in <facts>:
1. In-app banner: max 20 words, neutral.
2. Email to business customers: 60–80 words, formal, includes the times in UTC and
   a suggestion to plan exports beforehand.
3. Status-page entry: one factual line with times; no reassurance language.
Label each version with its number.
```

Specifying each audience's constraints in one place lets you compare the variants side
by side and keeps facts consistent across them.

## How It Actually Works

Style is distributed across every token choice — word frequency, sentence boundaries,
punctuation — so it is shaped by everything in the context: instructions, samples, even
the style of your own prompt. A prompt written in terse, plain sentences tends to pull
the output that way; a rambling prompt pulls the other way. Voice samples work
particularly well because they supply the dense, low-level statistical cues (rhythm,
word choice) that an adjective like "punchy" only gestures at.

The default "assistant voice" is itself a learned style: fine-tuning on preferred
responses creates recognizable habits, and without strong cues the model falls back on
them. Explicit specs and samples give it stronger cues to follow instead.

## Common mistakes

- **Single-adjective style instructions** ("make it engaging").
- **Voice samples without a "don't copy content" instruction.**
- **Simplifying without protecting facts** like numbers and names.
- **Huge banned-word lists** instead of describing the target.
- **Writing the prompt in the opposite style** of the output you want.

## Exercise

1. Write a style spec (6–8 lines, using the dimensions table) for a real voice you need:
   your team's docs, your newsletter, your own emails.
2. Find a 60–100 word sample that exemplifies it.
3. Generate the same piece three ways: no style guidance; spec only; spec + sample.
4. Run `style_report` (or your own checks) on each and read them aloud. Which is closest,
   and which dimension is still off?
