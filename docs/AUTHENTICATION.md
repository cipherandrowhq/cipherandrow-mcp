# Authentication

The Cipher & Row MCP server supports two authentication modes. Pick whichever your client
speaks natively - both resolve to the same account, role, and rate-limit tier.

Endpoint: `https://mcp.cipherandrow.com/mcp`

---

## Mode 1 - OAuth 2.1 + PKCE (recommended for MCP clients)

Standard **Authorization Code flow with PKCE** (RFC 6749 + RFC 7636), plus **Dynamic
Client Registration** (RFC 7591), so a compliant MCP client registers itself and signs in
with **no key to paste**. This is how Claude (web, desktop, Code), Cursor, and other MCP
clients connect.

The server publishes the standard discovery documents; point your client at the base URL
and it finds everything it needs:

| Document | Path |
| --- | --- |
| Authorization Server Metadata (RFC 8414) | `/.well-known/oauth-authorization-server` |
| Protected Resource Metadata (RFC 9728) | `/.well-known/oauth-protected-resource` |
| Authorization endpoint | `/oauth/authorize` |
| Token endpoint | `/oauth/token` |
| Revocation endpoint | `/oauth/revoke` |

**Flow, end to end:**

1. Client requests a protected method and receives `401` with a
   `WWW-Authenticate` header pointing at the protected-resource metadata.
2. Client reads the metadata, discovers the authorization server, and (if needed)
   dynamically registers.
3. User approves the consent screen in a browser (PKCE protects the exchange).
4. Client receives a token and calls tools as `Authorization: Bearer <token>`.

When authenticated this way, you **omit `agent_key`** from tool arguments - the server
resolves your identity from the token.

---

## Mode 2 - API key (Bearer token)

For CI, scripts, servers, or a quick `curl` smoke test, use a Cipher & Row API key.

1. [Sign up](https://www.cipherandrow.com/signup) - free, no credit card.
2. **Dashboard → MCP → Create key.** Keys are prefixed `cr_live_`.
3. Send it on every request:

```bash
curl -sS https://mcp.cipherandrow.com/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer cr_live_YOUR_KEY" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{
        "name":"cr_verify_carrier",
        "arguments":{"dot_number":"53467"}
      }}'
```

> Treat `cr_live_` keys as secrets. Do not commit them, embed them in client-side code,
> or paste them into shared chats. Rotate from the dashboard if one is exposed.

---

## Identity, roles, and what you can call

Your account carries a **logistics role** (dispatcher, broker, 3PL, freight forwarder,
carrier, ...) and a **subscription tier**. Both are enforced **server-side**:

- **Role gating** decides *which tools* you can call. For example, carrier-role accounts
  cannot call `cr_verify_carrier` (this prevents account-type arbitrage). A carrier can
  still watch its broker counterparties via `cr_add_partner`.
- **Tier** decides *how often* - see [RATE-LIMITS.md](./RATE-LIMITS.md).

Call **`cr_session_status`** any time to see your resolved identity, role, tier,
per-category rate-limit headroom, and the exact tool set available to your account.

---

## Anthropic marketplace reviewers / integration engineers

The server speaks standard OAuth 2.1 + PKCE with DCR, so the Claude connector
authenticates automatically - just add `https://mcp.cipherandrow.com/mcp` as a custom
connector and approve the consent screen. If you need a single pasteable, scoped,
read-only review key for a `curl` smoke test instead, email
**partnerships@cipherandrow.com** with your Anthropic project ID; same-business-day
turnaround.

---

Setup guide with per-tool examples: <https://www.cipherandrow.com/mcp/setup>
