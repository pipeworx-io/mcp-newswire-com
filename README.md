# @pipeworx/newswire-com

Live press releases from Newswire.com's public newsroom RSS feed — headline,
publish time and sub-headline excerpt, newest first, for the 50 most recent
releases. Newswire.com is an Issuer Direct wire, sibling to ACCESS Newswire,
and carries a large share of the same releases.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

- `newswire_com_latest_releases(since_hours?, limit?)` — the current rolling
  window of releases, newest first (up to 50). `newest_item_at` tells a
  caller whether the feed is quiet or stalled.
- `newswire_com_search_releases(query, limit?)` — keyword/company match over
  headline and excerpt, within the current window only.
- `newswire_com_get_release(url)` — one specific release, matched by the URL
  a prior call returned. Fails clearly (not silently empty) if the release
  has scrolled out of the current window.

## Auth

Keyless. No account, no clickthrough terms. `newswire.com/robots.txt`
disallows only the per-tag feeds and asks for `Crawl-delay: 5`; the
gateway's per-call result cache keeps polling well below that.

## Data sources

- <https://www.newswire.com/newsroom/rss> — the newsroom "Press Releases"
  channel. Measured 2026-09-22: 50 items spanning roughly a day.

Every call re-fetches this same rolling 50-item window live — it is **not**
an archive. `search_releases` only ever searches what is currently in the
window and `get_release` can miss something that has already fallen out of
it (the tool says so explicitly rather than returning an empty result).

This feed carries **no issuer field** — the company name lives only in the
headline or sub-headline — so `issuer` is `null` on every row rather than a
guess parsed out of the headline.

The `/feeds` index page lists ~650 per-beat feeds
(`/newsroom/rss/beat/<slug>`) and two "custom" feeds; every beat probed
returned a bare nginx 404 and both custom feeds return HTTP 410 Gone
(2026-09-22), so this pack does not enumerate them.

### Relationship to ACCESS Newswire

ACCESS Newswire (formerly ACCESSWIRE) serves every feed path (`/users/rss`,
`/rss`, `/rss.xml`, its sitemap index) and every article page behind a
Cloudflare managed challenge (HTTP 403, `cf-mitigated: challenge`); its only
open feeds, `/feed/rss2` and `/feed/atom`, are its corporate blog, not the
wire. Measured 2026-09-22: 12 of the 20 releases on ACCESS Newswire's
newsroom landing page appeared verbatim in this feed. This pack is therefore
the closest keyless, machine-readable surface to that wire — partial, not a
mirror (fleet #2286).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "newswire-com": {
      "url": "https://gateway.pipeworx.io/newswire-com/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/newswire-com/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1683+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/newswire_com_latest_releases \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/newswire_com_latest_releases`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "newswire-com": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-newswire-com"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-newswire-com
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Newswire Com data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
