# Using the Index

## Rank by

**Overall** ranks models scored by boards from two independent publishers. A **category** (Agents, Coding, Reasoning, Knowledge, General, Human preference) or a **domain** inside it ranks every model a recognised board has measured there; rows placed only by report figures carry a ✱ after their number.

When no recognised board measures a domain yet, a note above the table says so: every figure there is from one publication, a comparison set its publisher chose. Read those positions as “within that set”.

## Filters

| Filter            | What it does                                                                                                                                                                                                         |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Search            | Model name or organisation                                                                                                                                                                                           |
| Model line        | Claude, GPT, Qwen, K2, … read off the name                                                                                                                                                                           |
| Size              | Small ≤ 15B · Medium 15–100B · Large 100B–1T · Very large > 1T · Undisclosed. Counted from the weights on Hugging Face for open models, read from the name otherwise; closed models are undisclosed, never estimated |
| Coverage          | Minimum number of recognised boards; “Any coverage” includes report-only models                                                                                                                                      |
| Include reports ✱ | Off: report figures leave every score and report-only models are hidden                                                                                                                                              |
| Must include      | Only models scored by the chosen sources                                                                                                                                                                             |

## Reading a row

- **Index** — the Overall score. `~55✱` is an estimate; `≤` / `≥` mean the model sat outside the range of the models it could be compared with.
- **Sources** — one mark per recognised board (filled = scored; hover for the figure), a ✱ with a count for reports, and `n/N` coverage.
- **provisional** — fewer than two independent publishers.

## The model card

Open a row for: facts about the model (organisation, size, open weights, context, first listed), scores by category → domain, **how to call it** (OpenRouter id, vendor site, weights) and every source figure with the exact label the source printed.

## Sharing

Every filter lives in the URL. **Share on X / LinkedIn / Copy link** share exactly the current view; the link previews as a card of that view’s top ten. **Export chart** downloads the same top ten as a PNG.
