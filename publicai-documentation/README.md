---
description: >-
  PublicAI works on the two things the AI industry keeps private: the data
  models learn from, and the numbers models are judged by.
---

# 🚀 Overview

A model is shaped by two things almost nobody outside a lab gets to see: the data it learned from, and the evidence about how good it is. Who produced that data, whether they agreed, whether they were paid. Which benchmarks, run by whom, weighted how.

PublicAI builds one product for each, and both of them in the open.

## 🛰️ Trajector

An open-source command-line tool that turns your own Claude Code sessions into compensated data contributions.

With your explicit per-project consent it routes that project's API traffic through a proxy on your own machine, records the exchange, masks secrets locally, and uploads only the redacted result. Nothing is captured until you opt a project in — and a project you have not opted in is not merely excluded by policy, the code path to record it does not exist.

What comes out is real coding-agent work, the messy multi-turn kind that is hard to buy and impossible to fake. AI labs license it; the people who produced it are the ones paid.

→ [Install it](publicai-trajector-cli/installation.md) · [Data and privacy](publicai-trajector-cli/data-and-privacy.md)

## 📊 PublicAI Index

The public LLM leaderboards, aggregated into one ranking — and no benchmark of our own.

Every score traces back to a board someone else published. Figures from a launch post rather than a recognised board are marked ✱ and can never rank a model alone. The weighting is printed beside the table rather than kept as a trade secret, because a ranking you cannot audit is a ranking you should not cite.

Free to read, free to query, and built for machines as much as people: a JSON API, an MCP server, and a feed of what changed.

→ [Read the Index](publicai-index/README.md) · [MCP for agents](publicai-index/mcp.md)

