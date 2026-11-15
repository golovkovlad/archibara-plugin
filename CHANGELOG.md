# Changelog

## 0.1.1

- Connect Claude Code to the fixed address `https://app.archibara.com/mcp`, the same server the
  Archibara connector uses, so the Claude directory can read the plugin's server. The **Archibara
  server** setting is gone: a self-hosted server is now added as its own MCP server, as the
  README describes.

## 0.1.0

- Add the Archibara MCP server with browser sign-in (OAuth with PKCE), asking for read and edit
  scopes so the consent page offers both levels of access.
- Add the `architecture` skill, which the agent uses by itself to answer structure and impact
  questions from the model and checks the answers against the code.
- Add the review command for a read-only comparison of the model with the code, and the sync
  command to update the model from the code after a confirmed, validated preview.
- Support Claude Code (`/archibara:review`, `/archibara:sync`) and Codex (`$archibara:review`,
  `$archibara:sync`) from one package.
- Add an **Archibara server** setting for self-hosted installations in Claude Code; in Codex, a
  server named `archibara` in `config.toml` replaces the plugin's server.
