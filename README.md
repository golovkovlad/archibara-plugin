# Archibara for Claude Code and Codex

Bring your team's [Archibara](https://archibara.com) architecture into Claude Code or Codex. The
agent reads the C4 model while it works on your code, checks the model against the code and,
when you ask, brings the model up to date.

You need an Archibara account. [Sign up free](https://app.archibara.com).

## Install

**Claude Code.** Add the Archibara marketplace and install the plugin:

```text
/plugin marketplace add golovkovlad/archibara-plugin
/plugin install archibara@archibara
```

Restart Claude Code if the plugin's commands don't appear.

**Codex.** In a terminal, add the same marketplace and install the plugin:

```bash
codex plugin marketplace add golovkovlad/archibara-plugin
codex plugin add archibara@archibara
```

Start a new Codex session, or restart the Codex app, to load the plugin.

## Sign in

**Claude Code.** Run `/mcp`, select **plugin:archibara:archibara** and choose to authenticate.

**Codex.** Run `codex mcp login archibara` in a terminal. In the Codex app, select
**Authenticate** when Archibara asks you to sign in.

Your browser opens the Archibara sign-in page:

1. Sign in or register.
2. Choose one project. Under **Access**, keep **Read architecture**, or choose **Read and edit
   architecture** if the agent should update the model with the sync command. Editing needs the
   Editor or Administrator role in that project.
3. Select **Allow and return to the app**. The agent finishes connecting; there is no key to
   copy.

A connection lasts up to 7 days, then you sign in again. To switch to another project or change
access, sign out and sign in again: in Claude Code, clear the server's authentication in `/mcp`;
in Codex, run `codex mcp logout archibara`. You can revoke a connection at any time on the
Archibara **MCP** page under **My MCP keys**.

## Use

| | Claude Code | Codex |
| --- | --- | --- |
| Ask about the architecture | Just ask, or `/archibara:architecture` | Just ask, or `$archibara:architecture` |
| Review the model against the code | `/archibara:review [focus]` | `$archibara:review [focus]` |
| Update the model from the code | `/archibara:sync [scope]` | `$archibara:sync [scope]` |

- **Ask about the architecture.** For example: "Which containers call the billing API?" or "What
  does changing the orders database affect?" The agent uses the plugin's architecture skill by
  itself when a question or a planned change concerns the system's structure, and checks what
  the model says against the code. Paste a reference copied from Archibara (an `ArchibaraRef`
  line) to start from that element.
- **Review** compares the model with the code and reports drift, gaps in the model and risks,
  ordered by impact. The focus can be `drift`, `security`, `scalability`, `risks` or
  `unknowns`. The review changes nothing.
- **Sync** updates the model to match the code. The scope can be `full` for the whole
  repository, a git range such as `main..HEAD`, or a path; without one, the agent looks at the
  changes on the current branch, or maps the whole repository into an empty model. It shows a
  plan and a validated preview, and changes the model only after you confirm. Archibara takes a
  snapshot before every change, so you can undo it in the app.

Review and sync start only when you run them; the agent never starts them by itself.

If you work with several Archibara projects, name the project for this repository in your
`CLAUDE.md` or `AGENTS.md`, for example "Archibara project: Payments platform".

## Access and security

- Browser sign-in uses OAuth with PKCE and grants access to **one project**. Your current
  Archibara role and plan limits apply to every call.
- Claude Code and Codex keep the tokens in their own credential stores. Access tokens last 15
  minutes and refresh tokens rotate.
- The agent reads freely but asks before any call that changes the model, and sync waits for
  your confirmation as well.
- The skills tell the agent to record environment variable names only, never secret values, and
  not to copy credentials or personal data into the model.
- The plugin adds no hooks, background tasks or telemetry, and uploads no source code. The agent
  sends Archibara only the model changes you confirm and the queries it makes.

## Self-hosted servers

**Claude Code.** Set **Archibara server** to your server's address, without a trailing slash,
for example `https://archibara.example.com`, in `/config`; Claude Code may also ask for it when
you enable the plugin.

**Codex.** The plugin connects to app.archibara.com. To use your own server, add a server with
the same name to `~/.codex/config.toml`; it replaces the plugin's server, and the skills keep
working:

```toml
[mcp_servers.archibara]
url = "https://archibara.example.com/mcp"
scopes = ["mcp:read", "mcp:write"]
default_tools_approval_mode = "writes"
```

Then run `codex mcp login archibara`.

Team installations use the same browser sign-in. A local single-user installation without
sign-in, such as `http://localhost:8090`, connects directly. Installations protected by a single
shared token (`AUTH_MODE=token`) are not supported by the plugin; connect them as described in
the Archibara documentation.

## Support

Questions and problems: [archibara.com/contact](https://archibara.com/contact/) or
[support@archibara.com](mailto:support@archibara.com).

Archibara is proprietary software. See the bundled license.
