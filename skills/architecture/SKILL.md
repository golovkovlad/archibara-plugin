---
name: architecture
description: Answer questions about this system's architecture from the team's Archibara C4 model, checked against the code. Use when the user asks how the system is structured, which service or container owns something, what depends on a component or what a change would affect; before planning a change that crosses services, data stores or external integrations; and whenever the prompt contains an ArchibaraRef line.
argument-hint: "[question or ArchibaraRef line]"
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Architecture context from Archibara

Archibara holds the team's C4 model of this system: people, software systems, containers and
components, the relationships between them, views (diagrams) and records (decisions, risks,
assumptions, unknowns, requirements and notes). Treat the model as the documented design and the
code as what actually runs. Neither wins automatically.

Request: $ARGUMENTS

If the request line above is empty or shows only a placeholder, take the request from the user's
message.

## Connect

The tools below come from the Archibara MCP server that this plugin adds. If they are missing,
or a call reports that authentication is required, ask the user to sign in in the browser:

- Claude Code: run `/mcp`, select `plugin:archibara:archibara` and authenticate.
- Codex: run `codex mcp login archibara` in a terminal, or select **Authenticate** for Archibara
  in the Codex app.

Never ask for a key or token in the chat.

## Find the project

1. If the request contains an `ArchibaraRef` line, or an older `StructSmithRef` line, call
   `reference_resolve` with its `workspaceId`, `type` and `targetId` and start from the object it
   returns.
2. Otherwise call `workspace_list`. Browser sign-in grants one project, so usually exactly one is
   listed. With several, choose the one whose name matches this repository or the project named
   in CLAUDE.md or AGENTS.md. If it is still unclear, ask once, and suggest noting the project
   name there.
3. Call `workspace_inspect` once and reuse the packet for the rest of the task. Read it again
   only after the model has changed.

## Answer

- Map the question to model elements by name, technology, description and tags, then find the
  matching code: deployable units, package manifests, Dockerfiles, deployment manifests, API
  clients and data access code.
- For "what depends on X" or "what does changing X affect", call `explore_dependencies` instead
  of following relationships by hand. When it reports truncation, say so.
- Check every claim about the code in the code itself. When the model and the code disagree, say
  so, name both sides with the element and the file, and suggest the sync command to update the
  model: `/archibara:sync` in Claude Code, `$archibara:sync` in Codex. An element with lifecycle `future` is planned, so its absence from the code is expected;
  `deprecated` and `removed` elements can still appear in the code.
- Mention records linked to the elements involved (`linkedElementIds`), especially open risks and
  decisions that constrain the change.
- Name elements with their C4 kind, for example "the Billing API container".

## Limits

This skill only reads. Do not change the model here. If the user wants the model updated, point
them to the sync command: `/archibara:sync` in Claude Code, `$archibara:sync` in Codex.
