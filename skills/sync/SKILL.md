---
name: sync
description: Update the Archibara architecture model to match this codebase. Maps the repository into a first model, or brings an existing model up to date after code changes. Shows a plan and changes the model only after the user confirms.
argument-hint: "[full | <git range> | <path>]"
disable-model-invocation: true
allowed-tools:
  - mcp__plugin_archibara_archibara__workspace_list
  - mcp__plugin_archibara_archibara__workspace_inspect
  - mcp__plugin_archibara_archibara__modeling_guide
  - mcp__plugin_archibara_archibara__model_get
  - mcp__plugin_archibara_archibara__model_validate
  - mcp__plugin_archibara_archibara__model_preview_operations
  - mcp__plugin_archibara_archibara__view_list
  - mcp__plugin_archibara_archibara__view_get
  - mcp__plugin_archibara_archibara__record_list
  - Bash(git status:*)
  - Bash(git diff:*)
  - Bash(git log:*)
  - Bash(git merge-base:*)
  - Bash(git rev-parse:*)
---

# Sync the Archibara model with the code

Scope: $ARGUMENTS

If the scope line above is empty or shows only a placeholder, take the scope from the user's
message.

- No scope: the changes on the current branch since it left the default branch (find the base
  with `git merge-base`), plus uncommitted changes. If the model is empty, map the whole
  repository instead.
- `full`: the whole repository.
- A git range such as `main..HEAD` or `HEAD~5..HEAD`: the changes in that range.
- A path: only that part of the repository.

If the Archibara tools are missing or need authentication, ask the user to sign in in the
browser and choose **Read and edit architecture**: in Claude Code with `/mcp`, selecting
`plugin:archibara:archibara`; in Codex with `codex mcp login archibara` in a terminal, or
**Authenticate** in the Codex app. Never ask for a key or token in the chat.

## 1. Read the model

Call `workspace_list` and choose the project as the `architecture` skill does: the only one
listed, the one matching this repository or the one named in CLAUDE.md or AGENTS.md, otherwise
ask. Read it
with `workspace_inspect` and keep its `revision`. Call `modeling_guide` for element kinds, enum
values and the operation format, and follow its rules.

## 2. Collect evidence from the code

Look only for architecture-level facts, and note a file path for each one:

- Deployable units: apps, services, workers, functions and scheduled jobs (package manifests,
  Dockerfiles, compose files, Kubernetes or Terraform resources, CI deploy steps).
- Data stores and messaging: databases, caches, queues and topics (connection settings, ORM
  configuration, migrations).
- External systems: third-party APIs, SaaS products and identity providers (SDK clients, base
  URLs, webhooks).
- Interactions: HTTP or gRPC clients, published and consumed messages, shared databases, with
  protocol and direction.
- People: user roles, only where the code shows them, such as separate admin and customer apps.

Record environment variable names, never their values. Do not copy secrets, credentials or
personal data into the model. Model components only inside containers that already have
components in the model, or when the user asks for that level.

## 3. Plan

Compare the evidence with the model and present a short table: change, model object, evidence.

- Reuse existing elements. Match by name, technology and description before creating anything;
  never create duplicates.
- Update what the code proves: technology, a missing relationship, a wrong parent. Keep accurate
  human-written names and descriptions as they are.
- Never delete on your own. For something gone from the code, propose lifecycle `deprecated` or
  `removed`. Delete only when the user asks.
- Elements with lifecycle `future` are planned; their absence from the code is not drift.
- Model each relationship once, at the most specific level where both ends exist in the model,
  with `interactionStyle` and `technology` (such as "HTTPS/JSON" or "AMQP"). Views lift
  relationships to higher levels automatically.
- Do not invent what the code cannot show, such as owning teams, purposes or SLAs. List them as
  open questions, and offer to store them as `unknown` records.
- For an empty model, create the focal software system, its containers, and the people and
  external systems the code proves, plus a `systemContext` view and a `container` view, with an
  `autoLayoutView` operation for each view in the same batch.
- Add new elements to the existing views where they belong (`setViewElements`). Leave existing
  layouts alone.

## 4. Preview, confirm, apply

1. Put the whole change into one batch with `ref` aliases and run `model_preview_operations`.
   Fix the errors it reports and preview again.
2. Show the plan and the preview result, and ask the user to confirm. Change nothing before they
   do.
3. Apply the confirmed batch with one `model_apply_operations` call, passing the revision you read
   as `expectedRevision` and a short `label` such as "Sync with code". Archibara takes a snapshot
   first, so the change can be undone in the app. If the revision has moved, read the model again
   and re-plan instead of forcing the batch.
4. Run `model_validate`, then report what changed, which views it touched and which questions are
   still open.

If `model_apply_operations` is not available, the connection is read-only. Show the plan, and tell
the user to sign in again with **Read and edit architecture**, which needs the Editor or
Administrator role in the project. In Claude Code, they clear the server's authentication in
`/mcp` and authenticate again; in Codex, they run `codex mcp logout archibara` and then
`codex mcp login archibara`.
