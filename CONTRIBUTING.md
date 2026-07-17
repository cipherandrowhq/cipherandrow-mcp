# Contributing

Thanks for your interest. This repository is the **public interface** for the Cipher & Row
MCP server - schemas, client configs, and docs. The server itself is not in this repo.

## What we welcome

- **Corrections and clarifications** to the docs, schemas, or examples.
- **Additional client configs** (new MCP clients, editor integrations).
- **Better examples** for real integration patterns.
- **Reports of schema drift** - if the live server returns something the schemas here
  don't describe, that's a bug worth an issue.

## What lives elsewhere

- Changes to the **tools themselves**, the **trust model**, or **server behavior** are
  made on the server (proprietary) - open an issue describing the need and we'll route it.
- **Security issues**: do not open a public issue. See [`SECURITY.md`](./SECURITY.md).

## Ground rule: the live server is the source of truth

The schemas and manifest here mirror what the live server advertises via `tools/list`. When
in doubt, connect to `https://mcp.cipherandrow.com/mcp` and compare. PRs that bring this
repo closer to the live surface are especially appreciated.

## How to propose a change

1. Open an issue describing the change and why.
2. For docs/examples, a PR is welcome directly.
3. Keep it accurate - no speculative fields or endpoints that the server doesn't actually
   expose.

Questions: **partnerships@cipherandrow.com**.
