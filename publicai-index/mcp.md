# MCP

The Index is an MCP server — Streamable HTTP, stateless, no key:

```json
{
  "mcpServers": {
    "publicai-index": { "url": "https://publicai.io/model-index/mcp" }
  }
}
```

For stdio-only clients: `npx -y mcp-remote https://publicai.io/model-index/mcp`.

## Tools

| Tool             | Arguments                                                                                    | Returns                                                                                             |
| ---------------- | -------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `describe_index` | —                                                                                            | Method, weighting, sources, scopes, catalogs, limits. Call first when unsure which scope to ask for |
| `rank_models`    | `scope`, `org`, `family`, `size`, `minBoards`, `reports`, `limit`, `openWeights`, `callable` | Ranked rows with scores, coverage and the recommended access channel                                |
| `get_model`      | `model` (id or name)                                                                         | Everything the index holds on one model, every figure with its source URL                           |

Same read model as the page and the API; every answer carries `generatedAt`.

## Reading answers

- `index` null and `estimatedIndex` set → an estimate ✱, never a rank.
- `rankedInScope` false → placed by report figures only.
- `warning` present → the scope has no recognised-board measure yet; positions are within a publisher-chosen set.
- `access.recommended` → OpenRouter id and base URL where listed; `alternatives` hold the vendor site and open weights.
