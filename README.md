<div align="center">

# Cipher & Row - Freight Verification MCP

**LLM-ready freight data for AI agents.** Live FMCSA carrier & broker verification,
multi-signal 0-100 trust scores, double-brokering and chameleon-carrier fraud signals,
and a monitored partner watchlist - as native [Model Context Protocol](https://modelcontextprotocol.io)
tools an autonomous dispatch or brokerage agent can call directly.

![MCP](https://img.shields.io/badge/Model_Context_Protocol-2025--06--18-05e599)
![Auth](https://img.shields.io/badge/auth-OAuth_2.1_%2B_PKCE-blue)
![Tools](https://img.shields.io/badge/tools-8_Phase_1-informational)
![Surface](https://img.shields.io/badge/surface-read--only_%2B_watchlist-success)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

`https://mcp.cipherandrow.com/mcp`

</div>

---

This repository is the **open, public interface** for the Cipher & Row MCP server: the
exact tool JSON Schemas, copy-paste client configs, and integration guides. The
**verification engine, scoring model, and infrastructure are proprietary** (see
[`NOTICE`](./NOTICE)); this repo is the contract you build against, kept in sync with the
live server.

> **Who this is for:** engineers building **AI agent carrier vetting**, autonomous
> dispatch, brokerage automation, and freight-fraud-prevention systems - and security
> reviewers evaluating how the server resists prompt injection.

## Why it exists

An AI agent in freight reads untrusted text all day and can be talked into fraud - a fake
carrier, a spoofed broker, a double-brokering scam. The fix is to never let the model
*decide* trust from the conversation, but to *verify* against authoritative records the
attacker can't reach. This MCP server is that **air-gapped verification layer**: you
verify by **DOT / MC / CAB number** against FMCSA, surety-bond, sanctions, and registry
data, and get back structured evidence plus a canonical trust score - not a persuadable
opinion. Full model: [`docs/PROMPT-INJECTION-DEFENSE.md`](./docs/PROMPT-INJECTION-DEFENSE.md).

## Quickstart

**Claude Code**

```bash
claude mcp add --transport http cipher-row https://mcp.cipherandrow.com/mcp
# then run /mcp in a session and choose Authenticate
```

**Claude Desktop / stdio clients** - paste [`mcp-config/claude-desktop.json`](./mcp-config/claude-desktop.json)
into your `claude_desktop_config.json`.

**Cursor, Windsurf, others** - see [`mcp-config/cursor-and-other-clients.md`](./mcp-config/cursor-and-other-clients.md).

**Raw HTTP** - OAuth 2.1 + PKCE, or an `Authorization: Bearer cr_live_...` key:

```bash
curl -sS https://mcp.cipherandrow.com/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer cr_live_YOUR_KEY" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
        "name":"cr_verify_carrier","arguments":{"dot_number":"53467"}}}'
```

Auth in depth: [`docs/AUTHENTICATION.md`](./docs/AUTHENTICATION.md).

## The 8 Phase 1 tools

| Tool | Does | Surface |
| --- | --- | --- |
| [`cr_verify_carrier`](./schemas/cr_verify_carrier.schema.json) | Live FMCSA carrier profile + fraud signals + 0-100 trust score | read-only |
| [`cr_verify_broker`](./schemas/cr_verify_broker.schema.json) | Broker profile + surety bond + trust score (US FMCSA or Canadian registry) | read-only |
| [`cr_search_registry`](./schemas/cr_search_registry.schema.json) | Identity search to disambiguate before verifying | read-only |
| [`cr_session_status`](./schemas/cr_session_status.schema.json) | Your identity, role, tier, rate-limit headroom, available tools | read-only |
| [`cr_add_partner`](./schemas/cr_add_partner.schema.json) | Watch a counterparty for authority/insurance/bond/safety changes | write (your data) |
| [`cr_list_partners`](./schemas/cr_list_partners.schema.json) | List your watchlist + change events since a timestamp | read-only |
| [`cr_set_notifications`](./schemas/cr_set_notifications.schema.json) | Tune alert change-types / delivery channels | write (your data) |
| [`cr_remove_partner`](./schemas/cr_remove_partner.schema.json) | Stop watching a partner (soft delete; history preserved) | destructive (your data) |

Full manifest: [`schemas/tools.json`](./schemas/tools.json) · Human reference:
[`docs/TOOLS.md`](./docs/TOOLS.md).

**The surface is verification + a watchlist. No tool posts a load, moves money, or
messages a counterparty** - so even a hijacked agent's blast radius is bounded by design.

## Repository map

```
├── schemas/            JSON Schemas (draft 2020-12) for all 8 tools + tools.json manifest
├── mcp-config/         Copy-paste client configs (Claude Desktop/Code, Cursor, generic)
├── docs/
│   ├── PROMPT-INJECTION-DEFENSE.md   Air-gapped verification: how the server resists injection
│   ├── API-TO-MCP.md                 Append-only audit trail + multi-signal scores over MCP
│   ├── AUTHENTICATION.md             OAuth 2.1 + PKCE and API-key auth
│   ├── RATE-LIMITS.md                Per-tier, per-category ceilings
│   └── TOOLS.md                      Human-readable tool reference
├── examples/
│   ├── vet-a-carrier.md              End-to-end disambiguate → verify → decide → monitor
│   └── sample-responses/             Illustrative response shapes
├── SECURITY.md         Responsible disclosure + security posture
├── LICENSE             MIT (this interface repo)
└── NOTICE              What is open here vs. what stays proprietary
```

## How it verifies (the short version)

1. **Disambiguate** with `cr_search_registry` (identity only, no score).
2. **Verify** with `cr_verify_carrier` / `cr_verify_broker` - profile, fraud signals, and
   a canonical **0-100 trust score** with a per-factor breakdown, computed **server-side**
   from independent regulated signals. The entity being scored has no channel to inflate
   its own score.
3. **Decide** in your agent, on the evidence and verdict - not on the conversation.
4. **Monitor** with `cr_add_partner` so authority, insurance, and bond changes reach you
   between deals. Every verification call is written to a **server-side audit log**.

Details: [`docs/API-TO-MCP.md`](./docs/API-TO-MCP.md).

## Coverage

- **United States** - live FMCSA authority, insurance (BIPD/cargo/bond), safety and
  inspection history, surety bonds (BMC-84/85), sanctions screening.
- **Canada** - federal + cross-border FMCSA on every lookup; provincial depth varies by
  province. Canadian **broker** verification is available over MCP (`cab_id`); Canadian
  **carrier** intake is a REST write, not an MCP tool.

## Security

- **Air-gapped verification** - trust decisions are grounded in out-of-band authoritative
  data, not in the agent's persuadable context.
- **Typed output is structured evidence**; third-party free-text fields are data to
  reason over, never commands to execute.
- **OAuth 2.1 + PKCE** with scoped, server-enforced role gating and **per-key rate limits.**
- **EIN excluded** from the agent door; inputs sanitized; payloads capped.
- **Audit log** of every verification call (which tool, which account, when, outcome).

Report a vulnerability: [`SECURITY.md`](./SECURITY.md).

## License

The contents of this repository - schemas, configs, and documentation - are released under
the [MIT License](./LICENSE) so you can freely build against, redistribute, and adapt the
interface. The Cipher & Row verification server, scoring model, and engine are **not** open
source; see [`NOTICE`](./NOTICE).

---

**Setup + live examples:** <https://www.cipherandrow.com/mcp/setup> ·
**Partnerships / review keys:** partnerships@cipherandrow.com

<sub>Keywords: MCP freight verification · LLM-ready freight data · AI agent carrier vetting ·
FMCSA carrier lookup API · broker verification · double-brokering fraud detection ·
chameleon carrier · freight trust score · Model Context Protocol logistics · autonomous
dispatch agent.</sub>
