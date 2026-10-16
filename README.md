# @pipeworx/us-drought-monitor

Weekly U.S. drought-severity statistics — percent of area and percent of
population in categories None, D0 (abnormally dry) through D4 (exceptional
drought) — for the nation, each state, and each county.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1819+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `us_drought_national(area?, measure?, start_date?, end_date?)` — national
  drought statistics for the contiguous U.S. (`conus`), the contiguous states
  plus Alaska/Hawaii/Puerto Rico (`total`), or both at once (`us`). Answers
  "what percent of the contiguous US is in drought right now?"
- `us_drought_state(state, measure?, start_date?, end_date?)` — drought
  statistics for one state or territory, given by name, postal abbreviation,
  or 2-digit FIPS code.
- `us_drought_county(state?, county?, fips?, measure?, start_date?, end_date?)`
  — drought statistics for one county, matched by state + county name or by
  5-digit FIPS code.

All three categories are **cumulative**, which is the Drought Monitor's own
convention: `d1_pct` means "at least D1" (D1 through D4 combined), not D1
alone. `measure` picks `area` (percent of land area, default) or `population`
(percent of the area's population). Omit the date arguments for the latest
published map; pass `start_date`/`end_date` (YYYY-MM-DD or M/D/YYYY) for a
range of weekly maps.

The map is valid every Tuesday and the Center releases it the following
Thursday — every response carries `map_date`, `valid_start`, `valid_end`, and
`data_as_of` so a caller never has to guess which week a figure belongs to.

## Auth

Keyless. The upstream data service has no registration or key requirement.

## Data sources

- National Drought Mitigation Center (University of Nebraska-Lincoln), with
  the USDA and NOAA — the U.S. Drought Monitor's weekly drought-severity
  statistics (comprehensive area and population percent by category, for the
  nation, states, and counties).

## Notes

- Counties are matched by name against the full county list the publisher
  returns for a state (rather than carrying a ~3,100-row FIPS table in the
  pack), so a county name that exists in more than one form (e.g. "St. Louis"
  vs. "St. Louis City") should be disambiguated with the 5-digit `fips`
  argument if the name match is ambiguous.
- An area/state/county with no map in the requested window comes back from
  the upstream as an empty (but successful) result — the tools report that as
  `found: false` with a reason, not a thrown error.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "us-drought-monitor": {
      "url": "https://gateway.pipeworx.io/us-drought-monitor/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/us-drought-monitor/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1819+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/us_drought_national \
  -H 'Content-Type: application/json' \
  -d '{"area":"conus"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/us_drought_national`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "us-drought-monitor": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-us-drought-monitor"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-us-drought-monitor
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Us Drought Monitor data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
