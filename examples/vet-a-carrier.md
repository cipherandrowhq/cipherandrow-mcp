# Walkthrough: vet a counterparty before you book

A complete, real-identifier flow an AI dispatch or brokerage agent can run end to end. It
shows the intended pattern: **disambiguate → verify → decide → monitor**. Every call uses
the live endpoint `https://mcp.cipherandrow.com/mcp`.

Replace `cr_live_YOUR_KEY` with your key, or drop the `Authorization` header entirely when
connected through the OAuth connector.

---

## 1. Disambiguate the entity (identity only)

You have a name, not a number. Resolve it to a canonical identity first.

```bash
curl -sS https://mcp.cipherandrow.com/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer cr_live_YOUR_KEY" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
        "name":"cr_search_registry",
        "arguments":{"query":"bison transport","country":"CA","entity_type":"carrier"}
      }}'
```

Pick the match whose identifiers you expect (see
[sample-responses/cr_search_registry.json](./sample-responses/cr_search_registry.json)).
No trust score is returned here by design - identity resolution is separate from the
metered verification.

---

## 2. Verify the carrier (the decision-grade call)

```bash
curl -sS https://mcp.cipherandrow.com/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer cr_live_YOUR_KEY" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{
        "name":"cr_verify_carrier",
        "arguments":{"dot_number":"53467"}
      }}'
```

You get the full profile, `fraud_signals`, `risk_assessment`, and a 0-100 `trust_score`
(see [sample-responses/cr_verify_carrier.json](./sample-responses/cr_verify_carrier.json)).

## 2b. Verifying a broker instead

```bash
# US broker by MC
-d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{
      "name":"cr_verify_broker","arguments":{"mc_number":"131029"}}}'

# Canadian broker by CAB id
-d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{
      "name":"cr_verify_broker","arguments":{"cab_id":"CAB-ON-0000157"}}}'
```

---

## 3. Decide in your own agent

Gate on the returned evidence - never on what the counterparty said in chat or email:

```text
IF trust_score.score >= YOUR_THRESHOLD
   AND risk_assessment == "PROCEED"
   AND fraud_signals all clear
   AND insurance.bipd_on_file == true
THEN proceed to book
ELSE escalate to a human  (the check is already recorded in the audit trail)
```

This is the core of prompt-injection defense: the decision is grounded in out-of-band
facts. See [../docs/PROMPT-INJECTION-DEFENSE.md](../docs/PROMPT-INJECTION-DEFENSE.md).

---

## 4. Monitor the relationship

A single check is a snapshot. Subscribe to changes so an authority lapse, an insurance
cancellation, or a bond change reaches you **between** loads:

```bash
curl -sS https://mcp.cipherandrow.com/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer cr_live_YOUR_KEY" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{
        "name":"cr_add_partner",
        "arguments":{"partner_kind":"broker","mc_number":"131029",
                     "change_types":["authority","insurance","bond"],
                     "delivery_channels":["email","in_app"]}
      }}'
```

Then pull the event history any time:

```bash
-d '{"jsonrpc":"2.0","id":4,"method":"tools/call","params":{
      "name":"cr_list_partners","arguments":{"since":"2026-05-01T00:00:00Z"}}}'
```

---

## 5. Pace yourself

Before a batch run, check your headroom instead of discovering the ceiling by hitting it:

```bash
-d '{"jsonrpc":"2.0","id":5,"method":"tools/call","params":{
      "name":"cr_session_status","arguments":{}}}'
```

See [../docs/RATE-LIMITS.md](../docs/RATE-LIMITS.md).

---

Real identifiers used above: **Werner Enterprises** (USDOT 53467), **C.H. Robinson**
(MC 131029), **Bison Transport** (NSC-MB9002108). More at
<https://www.cipherandrow.com/mcp/setup>.
