# Changelog

This changelog tracks the **public MCP tool surface** as reflected in this repository. The
live server at `https://mcp.cipherandrow.com/mcp` is the source of truth; entries here
record surface-level changes that affect integrators.

The format follows [Keep a Changelog](https://keepachangelog.com/); the server reports its
build under `serverInfo.version` (call `cr_session_status` to read it live).

## 2026-07-12 - schema refresh + UI widgets

- Synced `schemas/tools.json` and all 8 `schemas/*.schema.json` files against a live
  `tools/list` call. Tool names are unchanged, but descriptions and `inputSchema`s are
  materially richer than the prior docs (bond sub-fields, fraud-signal breakdowns,
  `data_confidence`, `available_tools`/`granted_scopes` on `cr_session_status`).
- **Removed `agent_key` from every tool's advertised schema.** OAuth/Bearer is the only
  connection path reflected in the current tool surface; see [TOOLS.md](docs/TOOLS.md).
- **New:** `cr_verify_carrier`, `cr_verify_broker`, `cr_search_registry`, and
  `cr_list_partners` now return an MCP UI widget resource (`_meta.ui.resourceUri`) for
  clients that support the Apps SDK convention - see
  [MCP UI widgets](docs/TOOLS.md#mcp-ui-widgets).

## Server 2.1.0 - dashboard parity

- `cr_verify_carrier` and `cr_verify_broker` return the **full dashboard-parity payload**:
  complete carrier profile (insurance detail, 24-month inspections/crashes, MCS-150,
  complaints, classifications, contact), broker surety-bond detail, and the fraud/identity
  signal set (`fraud_signals`, `double_broker_signals`, `advisory_flags`).
- Canonical **0-100 trust score** with per-factor breakdown and `data_confidence` rides
  inline on every verification.

## Tool surface - 8 Phase 1 tools

The public surface is verification + a monitored watchlist:

- `cr_verify_carrier`, `cr_verify_broker`, `cr_search_registry`, `cr_session_status`
- `cr_add_partner`, `cr_list_partners`, `cr_set_notifications`, `cr_remove_partner`

### Naming note

Earlier tool names were renamed to the current, well-scoped set; the previous names
(`cr_fetch_carrier`, `cr_fetch_broker`, `cr_search`, `cr_network_status`) are **retired**.
A client calling a retired name receives an explicit error naming its replacement. Use the
current names above.

### Not on the public surface

Deal, fleet, and driver tools are **Phase 2** and are not part of the public Phase 1
surface. Canadian **carrier** intake is a REST write, not an MCP tool.
