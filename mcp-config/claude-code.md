# Add Cipher & Row to Claude Code

The Cipher & Row MCP is a **remote HTTP server**. Add it with one command:

```bash
claude mcp add --transport http cipher-row https://mcp.cipherandrow.com/mcp
```

Then, inside a Claude Code session:

1. Run `/mcp`
2. Select **cipher-row → Authenticate**
3. Approve the consent screen in your browser (OAuth 2.1 + PKCE)

The `cr_*` tools now appear in the session. There is **no API key to paste** - the
server resolves your identity from the OAuth token.

## Prefer a pasteable key?

If you would rather use a Cipher & Row API key (for CI, scripts, or a `curl` smoke
test), create one at **Dashboard → MCP → Create key** and send it as a Bearer token:

```bash
curl -sS https://mcp.cipherandrow.com/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer cr_live_YOUR_KEY" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Verify the connection

```bash
# Lists all 8 Phase 1 tools with their JSON schemas
claude mcp list
```

Or call the introspection tool once connected:

```
cr_session_status
```

It returns your agent identity, subscription tier, per-category rate-limit headroom,
and the exact set of tools your account type can call.

---

Full setup guide, working examples for every tool, and rate limits:
<https://www.cipherandrow.com/mcp/setup>
