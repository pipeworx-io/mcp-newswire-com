# @pipeworx/newswire-com

Live press releases from Newswire.com's public newsroom RSS feed — headline,
publish time and sub-headline excerpt, newest first, for the 50 most recent
releases. Newswire.com is an Issuer Direct wire, sibling to ACCESS Newswire,
and carries a large share of the same releases.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/newswire_com_latest_releases \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/newswire_com_latest_releases`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-newswire-com` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/newswire-com/mcp` endpoint shown above. It needs network access;
there is no offline mode.

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

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

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
