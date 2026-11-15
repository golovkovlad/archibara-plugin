---
name: review
description: Review the Archibara architecture model against this codebase and report drift, model gaps and risks without changing anything. Optional focus - drift, security, scalability, risks or unknowns.
argument-hint: "[drift | security | scalability | risks | unknowns]"
disable-model-invocation: true
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Review the architecture against the code

Focus: $ARGUMENTS

If the focus line above is empty or shows only a placeholder, take the focus from the user's
message. Without a focus, review drift and model quality. This review only reads: do not call
tools that change the model.

If the Archibara tools are missing or need authentication, ask the user to sign in in the
browser: in Claude Code with `/mcp`, selecting `plugin:archibara:archibara`; in Codex with
`codex mcp login archibara` in a terminal, or **Authenticate** in the Codex app. Never ask for a
key or token in the chat.

1. Call `workspace_list` and choose the project as the `architecture` skill does. Read it with
   `workspace_inspect` and run `model_validate`.
2. Collect architecture-level evidence from the code, each fact with a file path: deployable
   units, data stores and messaging, external systems, and the interactions between them. Never
   print secret values.
3. Report findings ordered by impact. Give each one the model object, the evidence and a concrete
   fix.
   - **Drift**: what the code has and the model lacks; what the model has and the code no longer
     has (skip lifecycle `future` elements, which are planned); wrong technology, direction or
     parent.
   - **Model quality**: validator findings that matter, wrong C4 levels, relationships without
     technology or interaction style, elements without a description, the same relationship
     modelled at several levels.
   - **security**: trust boundaries, authentication between services, stores of personal or
     payment data, secrets handling and what is exposed to the internet, as the code shows them.
   - **scalability**: long synchronous call chains, single points of failure, shared databases,
     hot paths without a queue or cache.
   - **risks**: architectural risks, each with a severity from `modeling_guide`.
   - **unknowns**: questions for the team that neither the model nor the code answers.
4. End with next steps: the sync command to fix the drift (`/archibara:sync` in Claude Code,
   `$archibara:sync` in Codex). If the user then asks to store risks
   or unknowns as Archibara records, list the records first and create them only after they
   confirm, in one `model_apply_operations` batch with `linkedElementIds` set. That needs **Read
   and edit architecture** access.
