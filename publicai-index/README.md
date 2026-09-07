# Overview

**PublicAI Index** is the LLM benchmark aggregator: one score from every recognised public leaderboard, standardized onto one scale, with launch-post figures marked ✱ and kept out of the headline. It is built from everyone’s benchmarks and none of our own.

Live at [publicai.io/model-index](https://publicai.io/model-index).

## What it is

- **Recognised leaderboards** (unmarked): LMArena Text, Artificial Analysis (Intelligence Index, GDPval-AA, AA-Briefcase), Terminal-Bench, ARC-AGI-2, LiveBench. Read from each publisher’s page; every figure links back to it.
- **Reports ✱**: launch posts and third-party write-ups. Indexed so brand-new models appear before the boards list them, but marked, discounted and never in the Overall ranking.
- **Catalogs**: OpenRouter (callable ids, context, price) and Hugging Face (parameter counts). Never scored.

## What it is not

- PublicAI runs no benchmarks. We normalize and weight; we do not score.
- Weights are fixed and published. They are not user-adjustable.
- A snapshot, not a live feed. The date is on every page and in every API response.

## Six rules

1. **Standardize, discount uncertainty.** Every figure is z-scored across the models its source lists and mapped to 0–100; a published error bar lowers its weight.
2. **Shrink, never impute.** A missing figure stays missing; thin evidence is pulled toward 50.
3. **Weight by a published scheme.** Each board’s share of the Overall index is fixed and stated with its reason.
4. **Two publishers to rank.** Overall needs boards from two independent publishers; a domain ranks on its own board evidence.
5. **Reports ✱ stay out of the headline.** They shape domain columns at a tenth of a board’s weight.
6. **Estimate the rest, and say so.** A model with no Overall score gets an anchored estimate, shown as ~55✱, never ranked.

See [Method](method.md) for the definitions.
