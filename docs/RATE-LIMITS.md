# Rate limits

Limits are enforced **server-side, per calling key**, and are **persistent** across
server restarts. They exist to keep the platform fair and to protect the upstream FMCSA
quota that every caller shares.

## Categories

Each tool counts against one of four per-minute buckets:

| Category | Tools |
| --- | --- |
| `read` | `cr_verify_carrier`, `cr_verify_broker`, `cr_search_registry`, `cr_list_partners`, `cr_session_status` |
| `setup` | `cr_add_partner`, `cr_remove_partner`, `cr_set_notifications` |
| `deal` | (Phase 2 - not on the public surface) |
| `sensitive` | (Phase 2 - not on the public surface) |

Separate buckets mean a burst of partner-management calls (`setup`) never crowds out your
FMCSA verifications (`read`).

## Per-minute ceilings by subscription tier

| Tier | setup / min | read / min | deal / min | sensitive / min |
| --- | ---: | ---: | ---: | ---: |
| Free | 5 | 30 | 10 | 3 |
| Carrier Verified+ | 10 | 60 | 30 | 10 |
| Dispatcher Solo | 10 | 100 | 50 | 15 |
| Dispatcher Pro | 25 | 300 | 150 | 50 |
| Broker Compliance | 10 | 100 | 50 | 15 |
| Broker Pro | 25 | 300 | 150 | 50 |
| Broker Scale | 100 | 1500 | 750 | 200 |
| Enterprise | 500 | 5000 | 2500 | 1000 |

A per-company **monthly** call cap also applies on lower tiers; it scales with tier and is
uncapped on Enterprise.

## When you hit a limit

A throttled call returns a JSON-RPC error with the category, your limit, and the reset
time, alongside HTTP `429` semantics (`Retry-After`, `reset_at`). Back off and retry after
the reset.

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rate limit exceeded. Tier \"free\" allows 30 read ops per 60s; you used 30.",
    "data": { "tier": "free", "category": "read", "limit": 30, "used": 30, "remaining": 0, "reset_at": "..." }
  }
}
```

## Check your headroom

`cr_session_status` returns your current per-category headroom and tier, so an agent can
pace itself instead of discovering the ceiling by hitting it:

```json
{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"cr_session_status","arguments":{}}}
```

Need higher limits? **partnerships@cipherandrow.com** ·
Live table: <https://www.cipherandrow.com/mcp/setup>
