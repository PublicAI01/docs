# Subscriptions

## For people

- **Index Weekly** — a Monday email: the top ten, the biggest movers, new entrants and new sources, one chart. Sign up in the sidebar of the [Index](https://publicai.io/model-index).
- **RSS** — [`publicai.io/model-index/feed.xml`](https://publicai.io/model-index/feed.xml): one item per snapshot, listing what changed.
- **Watch a view** — _planned_: subscribe to any filtered view (its URL) and receive only that view's card and changes.

## For agents

- **Poll** — `GET /model-index/api?since=YYYY-MM-DD` or the MCP tool `whats_new({ since })`. See [API](api.md).
- **Push** — _planned_: register a webhook with a filter (`family`, `scope`, threshold); each snapshot that matches is POSTed as signed JSON.

## Cadence

Snapshots refresh daily at 06:00 UTC; the feed and `since` report only what changed. Index Weekly goes out Mondays at 09:00 UTC.
