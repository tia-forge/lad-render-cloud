# LAD Render Cloud — SimaticML → ladder diagram over MCP

**LAD Render Cloud** renders a TIA Portal **SimaticML** block (FC / FB / OB) into a **PNG ladder
diagram** — real ladder layout (contacts, coils, timers/counters, branch merges, jumpers, network
titles, interface declaration table), not syntax highlighting and not a code screenshot.

Use it so an AI agent's generated ladder logic can actually be *looked at*, on a machine that has no
TIA Portal installed.

> This repository is **documentation and client configuration only**. The service runs at
> `tia.hubwiz.com`; its implementation is not open source.

- **Endpoint**: `https://tia.hubwiz.com/render/mcp`
- **Transport**: Streamable HTTP (stateless, POST JSON-RPC)
- **Auth**: `Authorization: Bearer render_live_…` (the `X-API-Key` header is also accepted)
- **Dashboard / API keys**: https://tia.hubwiz.com/render
- **Registry entry**: `io.github.tia-forge/lad-render-cloud` ([official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=lad-render-cloud))

## What it does

| Tool | Purpose | Billing |
|---|---|---|
| `render_version` | Free: renderer version, supported languages / block types, size and input limits | free |
| `render_simaticml` | Take one SimaticML block, return its PNG ladder diagram | 1 credit |

`initialize` / `tools/list` / `render_version` are free; every other `tools/call` costs 1 credit, and
one call submits exactly one block.

## Inputs and limits

- **Accepted**: FC / FB / OB blocks in **LAD** or **SCL**, including blocks where networks mix both.
- **Rejected with an explicit error**: GlobalDB / InstanceDB / UDT / tag tables (no ladder diagram to
  draw) → `UNSUPPORTED_DOCUMENT`; FBD / GRAPH / STL → `UNSUPPORTED_LANGUAGE`.
- **Options**: `network` renders a single 0-based network; `scale` (1–3) enlarges the image.
- **Limits**: input XML 4 MiB, MCP request body 8 MiB, image 32,000,000 pixels with no side over
  20,000 px, PNG 12 MiB, rate limit 120 requests/minute. Too tall, or `scale` too large → render one
  network instead of the whole block.

## How to get SimaticML

In TIA Portal: right-click the block → **Export** → **XML**. The saved
`<Document><SW.Blocks.FC|FB|OB>…` text is the SimaticML this service expects. No TIA Portal at hand?
Render one of the published example XMLs first.

## Quick start

### Claude Code

```bash
claude mcp add --transport http lad-render https://tia.hubwiz.com/render/mcp \
  --header "Authorization: Bearer render_live_YOUR_KEY"
claude mcp list
```

### Cursor / VS Code / any JSON-configured client

```json
{
  "mcpServers": {
    "lad-render": {
      "url": "https://tia.hubwiz.com/render/mcp",
      "headers": { "Authorization": "Bearer render_live_YOUR_KEY" }
    }
  }
}
```

### First calls

```jsonc
// free — what does this renderer support right now?
{ "name": "render_version", "arguments": {} }

// one block → PNG (costs 1 credit)
{ "name": "render_simaticml", "arguments": { "xml": "<Document>…</Document>" } }

// a single network, 2× larger
{ "name": "render_simaticml", "arguments": { "xml": "<Document>…</Document>", "network": 0, "scale": 2 } }
```

## Pricing

| Pack | Price | Per render |
|---|---|---|
| Free trial (new API key) | 100 renders | — |
| 1,000 renders | ¥9.90 | ≈ ¥0.0099 |
| 5,000 renders | ¥29.90 | ≈ ¥0.0060 |
| 15,000 renders | ¥69.90 | ≈ ¥0.0047 |

No subscription, no monthly fee; balance is applied immediately, and `render_version` /
`tools/list` never consume credits.

## Scope

- Renders text only: it does not connect to TIA Portal, does not read or write a project, and never
  downloads to a device. For on-premise project work use the **TiaLink** MCP gateway
  (https://github.com/tia-forge/tialink) instead.
- Output is for reading and review — it does not replace compiling the block in TIA Portal or
  commissioning on site.

## Related

- **TiaLink** — TIA Portal MCP gateway, on your machine: https://github.com/tia-forge/tialink
- **LadDoc Cloud** — LadDoc IR → importable SimaticML: https://github.com/tia-forge/laddoc-cloud
- **PLC Cloud Sim** — validate and step-run LAD/SCL in the cloud: https://github.com/tia-forge/plc-cloud-sim
