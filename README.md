# @pipeworx/jp-fsa-actions

Has the Japan Financial Services Agency (金融庁) taken an administrative
disposition against this financial firm: which order, when, under which law,
for what cause? Rows are the agency's own collection of dispositions issued by
the agency and the Local Finance Bureaus since fiscal 2002. The enforcement
counterpart of `jp-fsa-licensed` (who is registered) and `jp-fsa-warnings`
(unregistered operators warned).

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2014+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `jp_fsa_actions_search(name?, corporate_number?, action_type?, limit?)` —
  exact or partial name match after Unicode folding, old-form kanji folding and
  dropping corporate-form words, and/or exact 13-digit corporate number. Each
  match is the agency's row: fiscal year, announcement date (ISO), the 解除の有無
  column as published, business category, name, corporate number where the row
  has one (entries since 2016), law, type and content of the order, main cause and
  trigger. Also returns `result` (`hit` / `no_hit`), `total_matches`, `list_size`,
  `covers` (earliest and latest announcement), `omitted_persons` and `data_as_of`.

## Auth

Keyless.

## Data sources

- Japan Financial Services Agency (金融庁), collection of administrative
  dispositions against financial institutions (行政処分事例集). Read from the
  publisher on request and held in the serving process for an hour (~0.4 MB);
  the list's own "as of" date is `data_as_of`.

## Behaviour worth knowing

- About 2,800 rows: securities and investment firms (most), banks, insurers and
  insurance agencies, money lenders, payment and prepaid issuers.
- Not in it: Securities and Exchange Surveillance Commission recommendations
  (surcharge orders, criminal accusations) and actions by other agencies.
- Rows that read like individuals (no corporate number and a bare short kanji
  name, a personal name in parentheses, or an intermediary row) are not served and are
  counted in `omitted_persons`. Older rows often omit 株式会社, so a plain
  trade name is kept. The rule is heuristic: a company with a bare 2-4 kanji name
  and no corporate number (pre-2016 rows) can be dropped.
- A header row that changes, a list that parses to implausibly few rows, or a
  non-spreadsheet answer is refused rather than read as zero rows.
- A hit says the order was announced, not that it is still in force; read the
  published 解除の有無 value and the agency's own page.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "jp-fsa-actions": {
      "url": "https://gateway.pipeworx.io/jp-fsa-actions/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/jp-fsa-actions/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2014+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/jp_fsa_actions_search \
  -H 'Content-Type: application/json' \
  -d '{"name":"野村證券"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/jp_fsa_actions_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-jp-fsa-actions` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/jp-fsa-actions/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "jp-fsa-actions": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-jp-fsa-actions"]
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
ask_pipeworx({ question: "your question about Jp Fsa Actions data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
