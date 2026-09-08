---
description: >-
  Two things the AI industry keeps private: the data models learn from, and the
  numbers they are judged by. PublicAI works on both, in the open.
---

# 📖 What PublicAI Builds

Modern models are shaped by two things almost nobody outside a lab gets to see. The first is the data they learn from — who produced it, whether that person agreed, whether they were paid. The second is the evidence about how good they are — which benchmarks, run by whom, weighted how.

PublicAI builds one product for each.

***

## 🛰️ Trajector — the data side

[Trajector](publicai-trajector-cli/README.md) is an open-source command-line tool that turns your own Claude Code sessions into compensated data contributions.

Run `trajector enable` inside a project and that project's API traffic is routed through a proxy on your own machine. The proxy forwards every request verbatim and records it on the side; secrets are masked locally, and only the redacted result is uploaded.

* **Consent is per project, and it is structural.** Nothing is captured until you opt a project in. A project you have not opted in is not merely excluded by policy — the code path to record it does not exist.
* **Forwarding is sacred.** Any failure on the recording side never interrupts your work. Streaming responses pass through unbuffered.
* **Credentials never touch disk.** `Authorization` and `x-api-key` are not written to any file, in any state.
* **Consent is revocable.** `disable`, `logout` and `uninstall` each undo a different amount, immediately.

What comes out is real coding-agent work — the messy, multi-turn kind that is hard to buy and impossible to fake — contributed knowingly and paid for. AI labs license it; the people who produced it are the ones compensated.

Start at [Installation](publicai-trajector-cli/installation.md), or read [Data and privacy](publicai-trajector-cli/data-and-privacy.md) first if that is the part you care about.

***

## 📊 PublicAI Index — the measurement side

The [PublicAI Index](https://docs.publicai.io/index) aggregates the public LLM leaderboards into a single ranking, and runs no benchmark of its own.

Every score traces back to a leaderboard someone else published. Figures that come from a launch post or a write-up rather than a recognised board are marked ✱ and can never rank a model on their own. The weighting is printed next to the table rather than kept as a trade secret, because a ranking you cannot audit is a ranking you should not cite.

It is free to read, free to query, and built to be used by machines as much as people: a plain JSON API, an MCP server, an RSS feed of what changed. See [MCP](https://docs.publicai.io/index/mcp) for the four tools, or [Method](https://docs.publicai.io/index/method) for how the scores are actually computed.

***

## Why both

They are the two ends of the same argument. Trajector is about where a model's training data comes from and who gets paid for it. The Index is about who decides a model is good and whether you are allowed to check their work. Neither is a market PublicAI wants to own — both are things that ought to be public, and currently are not.

***

The `$PUBLIC` token is documented separately: [Tokenomics](https://docs.publicai.io/token).
