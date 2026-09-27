---
description: "Prompting with images, screenshots, charts and scanned documents: describing what to look at, grounding in visible evidence, and known visual weak spots."
---

# 08 · Multimodal Prompting

Many current models accept images alongside text, and some accept audio, video, or PDF
documents directly. Multimodal prompting uses all the same principles as text prompting —
clear task, context, format, examples — plus a few specific to visual input: telling the
model where to look, separating what is *seen* from what is *inferred*, and knowing the
visual tasks models still struggle with.

## Typical tasks

- **Describe / caption:** alt text for accessibility, product descriptions.
- **Read / extract:** text from screenshots, receipts, forms, whiteboard photos (OCR-like).
- **Interpret charts and diagrams:** trends, comparisons, anomalies.
- **Compare images:** before/after, design variants.
- **Reason about UI:** "what's wrong in this screenshot of the error dialog?"

## Principles

**Say what the image is and why you're sharing it.**

```text
The image is a photo of a paper receipt from a restaurant, taken on a phone at an angle.
I need to file it as an expense.
Extract: merchant name, date, total amount, currency, and tip if shown.
Return JSON; use null for anything not legible. Don't compute or estimate missing values.
```

**Direct attention.** Refer to regions and elements: "the table in the lower half",
"the red error banner at the top", "the y-axis labels".

**Separate observation from inference.**

```text
First list what you can directly see in the chart (axis labels, units, values you can
read). Then, separately, state what you infer from it, marking each inference as such.
```

This two-part structure mirrors quote-first answering (lesson 07): observation acts as
evidence the inference must rest on.

**Ask for uncertainty on legibility.** "If a digit is unclear, write it as `?` rather than
guessing." Guessed digits in amounts and dates are a common, costly error.

## Known weak spots

Capabilities improve quickly, but these areas have historically been error-prone and are
worth testing on your model:

- **Exact counting** of many similar objects.
- **Precise spatial relations** and measurements ("how many pixels apart", "which line is
  longer" when close).
- **Reading small, low-contrast, rotated, or handwritten text.**
- **Reading exact values from charts** without labelled data points; estimates from bar
  heights can be off.
- **Dense tables** in images, especially with merged cells.
- **Fine visual differences** between two similar images.

When exactness matters, prefer the underlying data (the CSV behind the chart, the text
behind the screenshot) over the image.

## Practical tips

- **Resolution and cropping:** providers resize large images; a small text region in a
  large photo may become illegible. Crop to the region of interest, or send a crop and a
  full view.
- **Multiple images:** label them in text ("Image 1: before; Image 2: after") and refer
  to those labels.
- **Order:** several providers suggest placing images before the question about them;
  check your provider's guidance and test.
- **Privacy:** screenshots and photos often contain more than you intend — names, email
  addresses, faces, notifications. Crop or redact before sending.
- **Alt text:** for accessibility work, specify purpose and length ("one sentence that
  conveys what matters in the context of the article, not every detail").

## Worked example: chart interpretation with evidence

```text
The image is a line chart from our monthly metrics deck showing weekly active users.

1. Observations: list the chart title, axis labels and units, the time range, and the
   approximate values at the start, the end, and any obvious peak or dip. Mark values as
   approximate if they're read from the line rather than printed labels.
2. Inferences: describe the overall trend and anything unusual. Each inference must refer
   to a specific observation from step 1.
3. Questions: list anything a careful analyst would want to check that the chart alone
   can't answer (e.g. changes in how "active" is defined).
Plain text, numbered sections.
```

The "Questions" section is a useful habit: it steers the model away from confidently
explaining a dip it has no information about.

## Worked example: UI bug triage from a screenshot

```text
Screenshot 1 shows our checkout page on a phone. A user reported they "couldn't pay".
Describe any visible problems that could block payment (e.g. disabled buttons, error
messages, overlapping elements, cut-off fields). Quote any on-screen text exactly.
If nothing visible would block payment, say so. Don't speculate about backend causes.
```

## How It Actually Works

In most current multimodal models, an image is converted by a vision encoder into a
sequence of embedding vectors — "visual tokens" — covering patches of the image. These
are placed into the same sequence as your text tokens, and the language model attends
over both. That's why text prompting techniques carry over: the image becomes part of the
context, and your words steer which parts are attended to.

It also explains the weak spots. Images are typically resized to a fixed budget of
patches, so fine detail can be lost before the model ever "sees" it. Counting and precise
measurement require exact, systematic comparison across many patches, which is not what
the encoder-plus-prediction pipeline is naturally good at. Reading chart values means
estimating continuous positions from patch features — approximate by nature.

## Common mistakes

- **No context about the image's purpose.**
- **Trusting extracted digits** from blurry images without a legibility flag.
- **Asking for exact counts or measurements** and not verifying.
- **Sending full-resolution screenshots** where the relevant text is tiny after resizing.
- **Accidentally sharing private data** visible in screenshots.

## Exercise

1. Take three images from your own work: a chart, a screenshot, and a photo of printed
   or handwritten text.
2. For each, write a prompt using the observation/inference split and a legibility rule.
3. Check every extracted number and quoted string against the image yourself.
4. Try the photo again after cropping to the text region. Did accuracy change?
