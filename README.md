# mcp-octopart

Octopart MCP — electronic component search via the Nexar Supply API (GraphQL)

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `octopart_search` | Search Octopart for electronic components by manufacturer part number (MPN) or keyword. Returns matching parts with manufacturer, category, median price, total availability, and distributor offers (stock, price breaks, MOQ, lead time). Example: octopart_search({ q: "STM32F407", limit: 5, _apiKey: "client_id:client_secret" }) |
| `octopart_multi_match` | Batch-match a list of manufacturer part numbers in one call — ideal for BOM (bill of materials) enrichment. Each query resolves to the best-matching part(s) with distributor offers. Example: octopart_multi_match({ mpns: ["SY89832UMG", "SY89874UMG"], limit: 1, _apiKey: "client_id:client_secret" }) |
| `octopart_part` | Get full detail for a single electronic part by exact MPN: every distributor seller with offers (price breaks, inventory level, MOQ, factory lead time), technical specs, availability, datasheet URL, and Octopart page. Example: octopart_part({ mpn: "LM317T", _apiKey: "client_id:client_secret" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "octopart": {
      "url": "https://gateway.pipeworx.io/octopart/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/octopart/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/octopart_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "octopart": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-octopart"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-octopart
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Octopart data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
