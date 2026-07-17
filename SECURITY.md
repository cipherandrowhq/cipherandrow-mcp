# Security Policy

## Reporting a vulnerability

If you believe you have found a security issue in the Cipher & Row MCP server or in this
repository, please report it privately:

**security@cipherandrow.com** (or **partnerships@cipherandrow.com**)

Please include enough detail to reproduce - affected endpoint or tool, request/response,
and impact. We aim to acknowledge within one business day and to keep you updated through
remediation. Please do **not** open a public issue for a suspected vulnerability, and
please give us reasonable time to fix before any public disclosure.

We appreciate coordinated disclosure and will credit reporters who wish to be named.

### In scope
- The MCP server at `https://mcp.cipherandrow.com/mcp` and its OAuth endpoints.
- The schemas and configs in this repository (e.g. an inaccuracy that could cause an
  integration to mishandle a security-relevant field).

### Out of scope
- Volumetric / denial-of-service testing against the live endpoint.
- Findings that require a compromised client, host, or operating system.
- Social engineering of Cipher & Row staff or customers.

## Security posture (summary)

The server is designed as an **air-gapped verification layer** for AI agents. Highlights:

- **Trust decisions are grounded in out-of-band authoritative data** (FMCSA, surety-bond,
  sanctions, registry), not in the model's persuadable context. See
  [`docs/PROMPT-INJECTION-DEFENSE.md`](./docs/PROMPT-INJECTION-DEFENSE.md).
- **Typed tool output is structured data.** Third-party free-text fields (e.g. a
  carrier's own FMCSA name) are reported as data to reason over, not endorsed as
  instructions - build your policy on the typed fields.
- **Bounded surface:** verification + watchlist only - no tool posts a load, moves money,
  or messages a counterparty.
- **Server-enforced role gating** and **persistent, per-key rate limits.**
- **Sensitive data minimization:** tax identifiers (EIN) are excluded from the agent door;
  string inputs are sanitized; request payloads are size-capped.
- **Audit log** of every authenticated verification call (tool, account, timestamp,
  outcome) for due-diligence records.
- **Standard OAuth 2.1 + PKCE** with Dynamic Client Registration for client auth.

## Handling credentials

API keys are prefixed `cr_live_` and are secrets. Never commit them, embed them in
client-side code, or paste them into shared chats. Rotate from the dashboard if a key is
exposed. The OAuth flow avoids static credentials entirely and is preferred for MCP
clients.
