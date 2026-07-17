# Cursor, Windsurf, and other MCP clients

The Cipher & Row MCP server is a standards-compliant **remote HTTP (Streamable HTTP)**
server at:

```
https://mcp.cipherandrow.com/mcp
```

Any MCP client that supports remote servers uses that one URL. Authentication is
OAuth 2.1 + PKCE (discovered automatically) or an `Authorization: Bearer cr_live_...`
API key.

## Cursor

Add to `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (per-project):

```json
{
  "mcpServers": {
    "cipher-and-row": {
      "url": "https://mcp.cipherandrow.com/mcp"
    }
  }
}
```

If your Cursor build only supports stdio servers, bridge with `mcp-remote`:

```json
{
  "mcpServers": {
    "cipher-and-row": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.cipherandrow.com/mcp"]
    }
  }
}
```

## Windsurf / Cline / other clients

Use the same URL as a remote/HTTP MCP server, or the `mcp-remote` bridge shown above
for stdio-only clients.

## OAuth metadata (for client implementers)

The server publishes standard discovery documents; a compliant client needs no manual
configuration beyond the URL:

- `https://mcp.cipherandrow.com/.well-known/oauth-authorization-server` (RFC 8414)
- `https://mcp.cipherandrow.com/.well-known/oauth-protected-resource` (RFC 9728)

Dynamic Client Registration (RFC 7591) is supported, so the client registers itself
on first connect.

---

Full setup guide and per-tool examples: <https://www.cipherandrow.com/mcp/setup>
