# Tool reference

The Cipher & Row MCP server exposes **8 Phase 1 tools**: verification, registry search,
session introspection, and a monitored partner watchlist. All are read-only except the
watchlist writes, and none can post loads, move money, or message a counterparty.

Machine-readable schemas: [`../schemas/`](../schemas/) · Full manifest:
[`../schemas/tools.json`](../schemas/tools.json).

Connect via `https://mcp.cipherandrow.com/mcp` (OAuth or Bearer key). The server resolves
the calling agent from the token - no `agent_key` argument is part of the advertised tool
schemas below (confirmed against a live `tools/list` call 2026-07-12).

Legend: 🟢 read-only · 🟡 write (your data) · 🔴 destructive (your data)

Four tools also return an MCP UI widget resource for clients that support the Apps SDK
`_meta.ui.resourceUri` convention - see [MCP UI widgets](#mcp-ui-widgets) below. Clients
that don't support it simply ignore the field and use the plain JSON result.

---

## 🟢 cr_verify_carrier

Verify a **US motor carrier** by USDOT or MC number against live FMCSA data, at full
dashboard parity.

| Argument | Type | Required | Notes |
| --- | --- | --- | --- |
| `dot_number` | string | one of dot/mc | The carrier's USDOT number |
| `mc_number` | string | one of dot/mc | The carrier's MC number |

**Returns:** complete carrier profile (operating status, authority history, fleet,
insurance detail incl. BIPD/cargo/bond flags + expiry, safety rating, 24-month inspections
and crashes, complaints, MCS-150, cargo/operation classifications, contact + addresses);
`fraud_signals` (7-signal set), `double_broker_signals`, `advisory_flags`; a legacy
`risk_assessment` verdict (`PROCEED`/`CAUTION`/`BLOCK`); and a canonical 0-100
`trust_score` with per-factor `breakdown` and `data_confidence`. `trust_score` is `null`
if the carrier has never been looked up on any Cipher & Row surface.

**Notes:** Role-gated - carrier-role accounts cannot call this. FMCSA data cached 24h;
trust score and signals recompute on every call. EIN is excluded from the agent door.
Counts against `read`. Returns a `trust-scorecard` UI widget - see
[MCP UI widgets](#mcp-ui-widgets).

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
  "name":"cr_verify_carrier","arguments":{"dot_number":"53467"}}}
```

Example response shape: [`../examples/sample-responses/cr_verify_carrier.json`](../examples/sample-responses/cr_verify_carrier.json)

---

## 🟢 cr_verify_broker

Verify a **freight broker** (US via FMCSA, or Canadian via the CR registry) and get the
canonical trust score.

| Argument | Type | Required | Notes |
| --- | --- | --- | --- |
| `mc_number` | string | one of mc/dot/cab | US broker MC number |
| `dot_number` | string | one of mc/dot/cab | US broker USDOT number |
| `cab_id` | string | one of mc/dot/cab | Canadian broker, format `CAB-XX-NNNNNNN` |

**Returns (US):** live FMCSA broker profile, authority status, full surety bond detail
(`broker.bond`: BMC-84/85 type, surety company, amount, effective/cancellation dates,
classification flags, freshness), `fraud_signals`, `double_broker_signals`,
`advisory_flags`, `risk_assessment`, and the 0-100 `trust_score` block.
**Returns (CA):** Canadian registry profile (province, verification band, CTQ status) with
the same canonical trust score.

**Notes:** Use before accepting loads from an unknown broker. Counts against `read`.
Returns a `trust-scorecard` UI widget - see [MCP UI widgets](#mcp-ui-widgets).

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
  "name":"cr_verify_broker","arguments":{"mc_number":"131029"}}}
```

Example response shape: [`../examples/sample-responses/cr_verify_broker.json`](../examples/sample-responses/cr_verify_broker.json)

---

## 🟢 cr_search_registry

Disambiguate an entity **before** verifying it. Identity only - no trust score.

| Argument | Type | Required | Notes |
| --- | --- | --- | --- |
| `query` | string | ✅ | Legal name or identifier (MC, USDOT, CR ID, ...) |
| `country` | enum | | `auto` (default) · `US` · `CA` |
| `entity_type` | enum | | `carrier` · `broker` · `any` (default) |

**Returns:** a list of matches - legal name, identifiers, country, location, and claim
status. Then call `cr_verify_carrier`/`cr_verify_broker` on the chosen match.
Counts against `read`. Returns a `registry-picker` UI widget - see
[MCP UI widgets](#mcp-ui-widgets).

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
  "name":"cr_search_registry",
  "arguments":{"query":"bison transport","country":"CA","entity_type":"carrier"}}}
```

Example response shape: [`../examples/sample-responses/cr_search_registry.json`](../examples/sample-responses/cr_search_registry.json)

---

## 🟢 cr_session_status

Introspect the server and your account. Takes no arguments.

**Returns:** server build/version, your agent identity, company logistics role,
subscription tier, per-category rate-limit headroom, and `available_tools` - the
definitive role- and scope-filtered list your connection can call (computed with the
same filter `tools/list` uses, so the two always agree) - plus `granted_scopes` so you
can self-diagnose a missing-scope mismatch. Counts against `read`.

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
  "name":"cr_session_status","arguments":{}}}
```

---

## 🟡 cr_add_partner

Add a carrier or broker to your **monitored watchlist**.

| Argument | Type | Required | Notes |
| --- | --- | --- | --- |
| `partner_kind` | enum | ✅ | `carrier` · `broker` (carrier accounts: broker only) |
| `dot_number` | string | | Carrier USDOT |
| `mc_number` | string | | US broker MC |
| `cab_id` | string | | Canadian broker `CAB-XX-NNNNNNN` |
| `legal_name` | string | | Optional; backfilled if omitted |
| `change_types` | string[] | | `authority`, `insurance`, `bond`, `safety_rating`, `address`, `phone`, `chameleon_flagged` |
| `delivery_channels` | string[] | | `in_app`, `email`, `sms` |

**Returns:** the created/updated watch. Idempotent - re-adding upserts. Provide
`dot_number` when `partner_kind=carrier`, `mc_number` when `partner_kind=broker`. Default
change types are the six standard ones (`chameleon_flagged` is opt-in); default channels
are `in_app` + `email`. Counts against `setup`.

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
  "name":"cr_add_partner","arguments":{
    "partner_kind":"broker","mc_number":"131029",
    "change_types":["authority","bond"],"delivery_channels":["email"]}}}
```

---

## 🟢 cr_list_partners

List your watchlist and recent change events.

| Argument | Type | Required | Notes |
| --- | --- | --- | --- |
| `since` | string | | ISO 8601 - fold in events fired after this time |
| `partner_kind` | enum | | Filter `carrier`/`broker` |
| `limit` | number | | Default 50, max 200 |

**Returns:** active partners with current status and `last_event_at`; with `since`, the
events per partner. The hourly partner-watch-diff cron drives change detection. Counts
against `read`. Returns a `watchlist-table` UI widget - see
[MCP UI widgets](#mcp-ui-widgets).

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
  "name":"cr_list_partners","arguments":{"since":"2026-05-01T00:00:00Z"}}}
```

---

## 🟡 cr_set_notifications

Tune alert preferences - one watch, or the company default.

| Argument | Type | Required | Notes |
| --- | --- | --- | --- |
| `partner_id` | string | | Omit to set the company default for future watches |
| `change_types` | string[] | | Replace the watched change types |
| `delivery_channels` | string[] | | Replace the delivery channels |

**Returns:** the effective settings after the update. Counts against `setup`.

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
  "name":"cr_set_notifications","arguments":{"delivery_channels":["in_app","email"]}}}
```

---

## 🔴 cr_remove_partner

Stop watching a partner. **Soft-deletes** the watch (`active=false`); past alert history is
preserved. Reversible - re-add with `cr_add_partner` to resume watching.

| Argument | Type | Required | Notes |
| --- | --- | --- | --- |
| `partner_id` | string | one of | Preferred |
| `partner_kind` + `dot_number`/`mc_number`/`cab_id` | | one of | Identifier fallback |

**Returns:** confirmation. Counts against `setup`.

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
  "name":"cr_remove_partner","arguments":{"partner_kind":"broker","mc_number":"131029"}}}
```

---

## MCP UI widgets

Four tools advertise an MCP UI widget resource via `_meta.ui.resourceUri` in their
`tools/list` entry (Apps-SDK-style convention). A client that renders MCP UI resources
shows the widget inline; any other client ignores `_meta` and just uses the plain JSON
result - nothing about the tool call itself changes.

| Widget resource | Returned by |
| --- | --- |
| `ui://cr/trust-scorecard.html` | `cr_verify_carrier`, `cr_verify_broker` |
| `ui://cr/registry-picker.html` | `cr_search_registry` |
| `ui://cr/watchlist-table.html` | `cr_list_partners` |

---

## A note on Canadian carrier intake

Submitting Canadian **carrier** registration data is a write to the registry and is **not**
part of the MCP surface - use the REST endpoint documented at
<https://www.cipherandrow.com/mcp/setup>. Canadian **broker** verification *is* available
over MCP via `cr_verify_broker` with a `cab_id`.

---

Security model: [PROMPT-INJECTION-DEFENSE.md](./PROMPT-INJECTION-DEFENSE.md) ·
Rate limits: [RATE-LIMITS.md](./RATE-LIMITS.md) ·
Auth: [AUTHENTICATION.md](./AUTHENTICATION.md)
