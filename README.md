# @pipeworx/dice

Live event, venue and lineup listings from DICE (dice.fm) — city-by-city
browse of on-sale gigs, DJ sets, parties, comedy, theatre and talks, plus
full detail (dates, venue, starting price, billed artists) for any one event.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1800+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `dice_cities(query?)` — the full list of cities DICE sells tickets in
  worldwide, optionally filtered by a name/country substring. Each city's
  `id` + `perm_name` is what `dice_browse` needs.
- `dice_browse(city, category?)` — events currently on sale in one city,
  optionally filtered by category (e.g. "DJ", "Comedy", "Theatre"). Returns
  name, status, dates, venue and starting price for each event.
- `dice_event(event_id)` — full detail for one event: description, venue +
  address, dates, on-sale window, starting price, billed lineup, and
  highlights (age restriction, promoter). Needs the `id` from `dice_browse`.

## Auth

Keyless. `api.dice.fm` answers 403 without a key and is never called; this
pack reads the same server-rendered pages any anonymous visitor's browser
loads.

## Data sources

- DICE (dice.fm) — the platform's own public event, venue and city listings,
  as served to any anonymous visitor. No attendee/guest list or buyer contact
  data is exposed by any tool here; only public event, venue and billed-artist
  information.

## Notes

- `dice_browse`'s result set is whatever DICE's own server-rendered page ships
  (typically 20-30 events). `page=`/`nextCursor` on the browse page were both
  probed live and neither advances past the first batch — deeper paging goes
  through `api.dice.fm`, which this pack cannot reach keylessly. Results are
  not artificially truncated below what the page itself returns.
- `dice_event` takes the bare event id (24-char hex) from `dice_browse`'s `id`
  field. The bare-id event URL 308-redirects to the canonical slugged URL; a
  plain `fetch()` follows it, so the bare id is all callers need.
- City resolution in `dice_browse` accepts a city name, `perm_name`, or the
  full `perm_name-id` browse slug (e.g. "new_york-5bbf4db0f06331478e9b2c59"),
  matched case-insensitively against the live city directory (not a
  hardcoded list) — so DICE adding or renaming a city needs no code change
  here.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "dice": {
      "url": "https://gateway.pipeworx.io/dice/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/dice/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1800+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/dice_cities \
  -H 'Content-Type: application/json' \
  -d '{"query":"new york"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/dice_cities`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "dice": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-dice"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-dice
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Dice data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
