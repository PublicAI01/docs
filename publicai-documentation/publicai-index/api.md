# API

Base: `https://publicai.io/model-index/api`. Plain GET, JSON, no key. Responses carry `generatedAt`; cache for an hour.

## Describe

`GET /model-index/api` — method, weighting with rationale, every source with measures and caveats, categories → domains (usable as scopes), catalogs, limits.

## Rank

`GET /model-index/api?scope=coding&limit=10`

| Param       | Values                                                                                             | Default |
| ----------- | -------------------------------------------------------------------------------------------------- | ------- |
| `scope`     | `overall`, a category, a domain, or `category:X` / `domain:X` to settle a name used at both levels | overall |
| `org`       | organisation name                                                                                  | —       |
| `family`    | model line: `Claude`, `GPT`, `Qwen`, `K2`, …                                                       | —       |
| `size`      | `small` `medium` `large` `xlarge` `undisclosed`                                                    | —       |
| `minBoards` | minimum recognised boards; `0` includes report-only models                                         | 2       |
| `reports`   | `false` drops report figures and report-only models                                                | true    |
| `q`         | text match on name or organisation                                                                 | —       |
| `must`      | comma-separated source ids that must have scored the model; `any-report` for any ✱                 | —       |
| `limit`     | 1–100                                                                                              | 20      |

Each row: `position`, `name`, `org`, `family`, `index`, `estimatedIndex`, `scopeScore` (when a scope is asked for), `ranked`, `rankedInScope`, `covered/coverable`, `reports`, `size`, `sizeTier`, `agreement`, `access` (recommended channel with model id and base URL, alternatives, the rule).

When no recognised board measures the scope, the response carries `scopeBoards: 0`, `scopeReports` and a `warning`.

## What changed

`GET /model-index/api?since=2026-09-01&minDelta=3` — sources added or removed, models that entered the Overall ranking, moves of `minDelta` places or more, models no longer listed, between the last snapshot on or before `since` and the latest. Poll this instead of re-reading the ranking; snapshots refresh daily.

## Badge

`GET /model-index/badge?model=<id>[&scope=domain:Tool use]` — an SVG showing the live position (`#1 · 69.1`, `Tool use #18✱ · 59.4`, `provisional`, `~55✱ provisional`). Cacheable for an hour; the tooltip carries the snapshot date.

## One model

`GET /model-index/api?model=claude-fable-5-1` (id or name) — everything above plus `byCategory`, `byDomain`, `catalog` (context, prices, modalities, open weights) and `figures[]`: every source figure with `source`, `kind`, `measure`, `label` (as printed), `raw`, `rawLabel`, `stderr`, `normalized`, `url`, `retrievedAt`.

Unknown or ambiguous names return `error` and `candidates`.

## Rate limits

Cloudflare limits bursts from one IP (HTTP 429, `Retry-After`). Cache responses; the snapshot changes at most daily.
