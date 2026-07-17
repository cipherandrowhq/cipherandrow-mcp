# From REST API to MCP: LLM-ready freight data for AI agents

This document explains how Cipher & Row exposes its verification platform - the same
**audit logging** and **multi-signal risk scores** that power the dashboard and
REST API - to AI agents through the **Model Context Protocol (MCP)**. It is written for
engineers building **AI agent carrier vetting**, autonomous dispatch, and
freight-fraud-prevention systems who want **LLM-ready freight data** without wiring a
REST client, mapping FMCSA fields, or standing up their own trust model.

---

## Why MCP, not "just an API"

A REST API gives an application *endpoints*. MCP gives a *language model* **tools** -
each with a name, a description, and a typed JSON Schema - that the model can discover and
call on its own. The translation matters:

| REST API (for apps) | MCP (for AI agents) |
| --- | --- |
| You read docs, pick endpoints, write a client | The agent reads `tools/list` and self-selects |
| Fields are yours to interpret | Tool descriptions tell the model *when* and *why* to call |
| You build retry / auth / pagination | The client + server handle transport, OAuth, limits |
| Output is JSON you post-process | Output is structured **evidence** the model reasons over |
| Trust logic lives in your code | The 0-100 trust score is computed server-side |

Cipher & Row runs **one trust engine behind three doors** - dashboard, REST API, and MCP
- so an answer an agent gets over MCP is the *same* answer a human sees in the dashboard
("full dashboard parity"). No second-class agent data.

---

## What gets translated

### 1. Multi-signal risk scores → `trust_score` on every verification

The platform computes a canonical **0-100 trust score** for carriers and brokers from
independent, regulated signals - operating authority, insurance and surety-bond status,
safety and inspection history, sanctions/watchlist screening, and identity/chameleon
(reincarnation) checks. Over MCP, that score rides **inline** on the verification
response, with:

- `trust_score` - the canonical 0-100 value (null if the entity has never been looked up
  on any Cipher & Row surface).
- a **per-factor breakdown** - which signals contributed, so the agent can explain the
  score, not just quote it.
- `data_confidence` - how complete the underlying data was.
- a legacy `risk_assessment` verdict (`PROCEED` / `CAUTION` / `BLOCK`) for agents that
  want a single categorical gate.
- `fraud_signals` - the same seven-signal set the dashboard's fraud card renders
  (sanctions, industry fraud reports, authority, out-of-service/bond, insurance/bond
  classification, safety, chameleon), plus raw double-brokering signals and advisory
  flags.

The scoring model itself is proprietary; what's exposed is the **result and its factors**
- exactly what an agent needs to make and justify a decision. See
[TOOLS.md](./TOOLS.md) for the full response shape.

### 2. Server-side audit log → provable due-diligence

Every authenticated tool call is written to a server-side **audit log** - which tool,
which account, when, and the outcome. This is a compliance and dispute-defense aid: you
have a record that a verification was run, by whom, and at what time, rather than
reconstructing it after the fact.

Over MCP this shows up two ways:

- **Verification is a recorded event.** Calling `cr_verify_carrier` / `cr_verify_broker`
  is itself logged, so an agent's due-diligence is provable.
- **Monitoring is continuous.** `cr_add_partner` subscribes a counterparty to change
  detection (authority, insurance, bond, safety rating, address, phone, and opt-in
  chameleon alerts); `cr_list_partners` returns the event history since any timestamp.
  The audit trail becomes a live feed, not just an archive.

### 3. FMCSA + registry lookups → two clean tools

The REST platform fans out across FMCSA sub-endpoints, a carrier/broker registry, surety
bond records, and Canadian provincial/federal signals. MCP collapses that into:

- `cr_search_registry` - disambiguate first (identity only, no score).
- `cr_verify_carrier` / `cr_verify_broker` - the full, metered verification with the trust
  score and signals.

The agent never sees the fan-out; it sees two tools and a clean result.

---

## The verification lifecycle over MCP

```
 1. discover      client calls tools/list  ──►  8 typed tools, self-describing
 2. authenticate  OAuth 2.1 + PKCE (or Bearer key)  ──►  caller identity resolved
 3. disambiguate  cr_search_registry("bison transport", CA)  ──►  identity matches
 4. verify        cr_verify_carrier(dot_number)  ──►  profile + fraud signals + trust_score
                  └─ the call is written to the server-side audit log
 5. decide        your agent's policy gates on trust_score / risk_assessment / signals
 6. monitor       cr_add_partner(...)  ──►  authority/insurance/bond changes push to you
 7. review        cr_list_partners(since=...)  ──►  event history for the record
```

Steps 4-7 are the difference between a one-time API call and a **durable trust
relationship** an agent can maintain across many loads.

---

## Keywords, plainly

For teams searching for these capabilities: this is **LLM-ready freight data** delivered
over **MCP** for **AI agent carrier vetting** and **broker verification** - live **FMCSA
carrier lookup**, **double-brokering** and **chameleon-carrier** fraud signals, a
canonical **carrier/broker trust score**, a **server-side audit log** for compliance,
and a **partner monitoring** watchlist - all as native tools an autonomous **dispatch
agent** can call without building an integration.

---

## Design boundary

The **protocol and tool surface are open** (this repository). The **verification engine,
scoring model, and infrastructure are proprietary** (see [`../NOTICE`](../NOTICE)). You
integrate against a stable, documented interface; the trust computation stays a managed
service that improves without breaking your agent.

---

Setup + live examples: <https://www.cipherandrow.com/mcp/setup> ·
Security model: [PROMPT-INJECTION-DEFENSE.md](./PROMPT-INJECTION-DEFENSE.md) ·
Contact: **partnerships@cipherandrow.com**
