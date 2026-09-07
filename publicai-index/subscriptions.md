# Subscriptions

_Planned. Shipping in this order._

## For people

- **Index Weekly** — a Monday email: biggest movers, new entrants, new boards, one chart. Sign up on the Index page.
- **Watch a view** — subscribe to any filtered view (its URL) and receive only that view’s card and changes.
- **RSS** — `https://publicai.io/model-index/feed.xml`, one item per snapshot.

## For agents

- **Poll** — `GET /model-index/api/changes?since=YYYY-MM-DD`: new models, rank moves, new sources since that date. `ETag` supported. MCP tool `whats_new({ since })`.
- **Push** — register a webhook with a filter (`family`, `scope`, threshold); each snapshot that matches is POSTed as signed JSON (HMAC-SHA256 in `X-PublicAI-Signature`). Register via `POST /model-index/api/subscriptions` or the MCP tool `subscribe({ url, filter })`.

## Cadence

Snapshots refresh daily; notifications go out only when something changed. Weekly digest on Mondays. Instant: a new board is added, a watched model moves three places or more, a watched model gains an Overall rank.
