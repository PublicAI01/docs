# Method

## Standardization

For each measure _m_ with raw figures _x_ over the models it lists:

`z = (x − mean) / sd`, `score = clamp(50 + 15·z, 0, 100)`

A measure with fewer than two figures is dropped: there is nothing to standardize against.

## Confidence

Where a source publishes an error bar `e` and the measure’s spread is `r = max − min`:

`confidence = clamp(1 / (1 + 16·(e/r)²), 0.15, 1)`

No error bar means confidence 1. Absence is not treated as evidence of a wide one.

## Weighted mean with shrinkage

For a scope (Overall, a category or a domain) with measures `i`, weights `wᵢ`, confidences `cᵢ` and standardized scores `sᵢ`:

`prior = PRIOR_FRACTION · W_scope`, with `PRIOR_FRACTION = 0.25` and `W_scope` the sum of weights available in the scope

`score = (Σ wᵢ·cᵢ·sᵢ + prior·50) / (Σ wᵢ·cᵢ + prior)`

Missing measures are excluded from the sums, never imputed.

## Weights

| Board                                  | Overall share | Also shapes                                    |
| -------------------------------------- | ------------- | ---------------------------------------------- |
| LMArena Text                           | 25            | Human preference                               |
| Artificial Analysis Intelligence Index | 25            | General intelligence                           |
| LiveBench                              | 20            | Reasoning, Coding, Mathematics, … (8 measures) |
| Terminal-Bench                         | 15            | Agentic coding                                 |
| ARC-AGI-2                              | 15            | Abstract reasoning                             |
| GDPval-AA, AA-Briefcase                | —             | Professional tasks, Agentic knowledge work     |
| Any report ✱ measure                   | —             | its domain, at 10 (a board measure carries 25) |

## Eligibility

- **Overall**: scored by boards from ≥ 2 independent publishers. Publishers, not boards: Artificial Analysis’s three boards are one voice.
- **Category / domain**: any model a recognised board has measured in that scope. Reports alone never rank.

## Anchored estimate

For a model with no Overall score: on every measure it shares with models that have one, its figure is placed among theirs and their Overall index is read at that position (linear between neighbours). Outside their range the nearest anchor is a bound, shown as ≤ or ≥. Placements are averaged by measure weight, discounted where fewer than six anchors support them. Shown as `~55✱`; never a rank.

## Identity

Sources spell one model several ways. Names are reduced to a bag of tokens with reasoning-effort tiers, run settings, vendor prefixes and board badges removed; the highest published tier per source is indexed and the exact label kept. Every merge is logged in the build report.

## Data contract

`index.json` — `generatedAt`, `benchmarks[]` (id, name, publisher, url, retrievedAt, metric, category, domain, kind, source, group, caveat?), `models[]` (id, name, org, access?, size?), `scores[]` (modelId, benchmarkId, raw, stderr?, sourceLabel, scaffold?), `excluded[]`, `catalogs[]`. The same shape is validated by the pipeline before a snapshot ships.
