# Solibo Home plugin

Use Solibo Home in Claude and ChatGPT with your own account. Board members and
property managers can review issues, meetings, documents and invoices for their
housing communities; residents can find practical information, their invoices
and conversations. Changes are saved only after you confirm, and the assistant
sees only what your account may see.

The plugin bundles the hosted Solibo Home MCP server with the skills that teach
the assistant how to use it. One folder serves both hosts; it is published whole
to the public repository `Solibo-AS/solibo-home-claude-plugin`, which the Claude
directory lists, and zipped for the ChatGPT plugin directory:

| File | Host |
|------|------|
| `.claude-plugin/plugin.json`, `.mcp.json` | Claude (claude.ai, desktop, Cowork, Claude Code) |
| `plugin.json`, `mcp.json`, `assets/` | ChatGPT and Codex (Agent Plugins manifest) |
| `skills/<name>/SKILL.md` | Both |

The plugin does not ship credentials. Each host discovers OAuth from
`https://mcp.home.solibo.no` and prompts each user to sign in with their own
Solibo Home account.

## Skills

Each skill is a workflow the assistant loads when a request matches it. The
skills only call the Solibo Home tools; they add the order of calls, the checks
before a change, and the shape of the answer.

| Skill | Use it for |
|-------|-----------|
| `company-briefing` | Status of one housing community or a portfolio: board report, snapshot, open work, economy, morning brief |
| `conversation-triage` | Open conversations: counts, what to read, reply, move, close, with the bulk tools |
| `resident-communication` | Newsletters, SMS, feed posts and practical information: audience, notification groups, preview, draft, send or schedule |
| `meeting-agenda` | Board meetings: find, prepare, add agenda items, follow-ups, summons and signing |
| `economy-review` | Expenses, supplier invoices, accounting, loans and insurance for one community |
| `portfolio-sweeps` | Work across many communities with the portfolio tools and correct paging: the user's own companies, or for Solibo staff every assigned company |
| `document-filing` | Find, read, download, upload, rename and move documents |
| `supplier-register` | Suppliers and supplier contacts, including imports from accounting entries |

## Install in Claude

After approval in the Claude directory or the community marketplace:

```text
/plugin marketplace add anthropics/claude-plugins-community
/plugin install solibo-home@claude-community
```

The plugin is disabled immediately after installation. Enable it in `/plugin`,
then open `/mcp` and authenticate the `solibo-home` server. In claude.ai, add
the plugin from **Customize > Plugins** and connect the server on its
**Connectors** tab.

## Use in ChatGPT

The same folder is submitted to the ChatGPT plugin directory as a ZIP of the
Agent Plugins files (`plugin.json`, `mcp.json`, `README.md`, `skills/`,
`assets/`). Skills are part of the uploaded package; ChatGPT does not fetch them
from the server at runtime, so every skill change ships as a new plugin version.
The build script, the catalogue test and the publishing workflow live in the
Solibo Home MCP service repository, which is the source of truth for this
package.

## Permissions and safety

- Solibo Home remains the source of truth for data and permissions.
- Read access requires the `solibo.mcp/read` OAuth scope.
- Changes require `solibo.mcp/write`; resident communications require
  `solibo.mcp/communicate`.
- Sensitive changes and communications use preview and confirmation flows
  before anything is saved, sent, or published.
- The plugin sends your requests and the assistant's tool calls to
  `https://mcp.home.solibo.no` under your own account. It stores nothing itself.
- Production currently provides tools only; interactive MCP Apps are not
  enabled.

## Local validation

From this folder, with Claude Code 2.1.281 or newer (older versions warn about
the directory listing fields such as `icon`, and `--strict` turns that into a
failure):

```sh
claude plugin validate . --strict
```

For a local smoke test in Claude Code:

```sh
claude --plugin-dir .
```

Then run `/mcp` in Claude Code and authenticate the `solibo-home` server. The
service repository also runs a test that checks every tool a skill names against
the tool catalogue and that the two manifests describe the same plugin and server.

## Licence and trademarks

The files in this plugin (manifests, skills and this README) are released under
the MIT License; see `LICENSE`. The licence covers these files only, not the
Solibo Home service or the MCP server, which stay behind each user's own account.
"Solibo" and "Solibo Home" are trademarks of Solibo AS. The MIT License grants no
rights to those names: do not use them to name or brand a derived plugin, and do
not present a derived plugin as published by Solibo AS.

## Published behavior

- The plugin is disabled by default in Claude Code so users explicitly opt in
  before Claude connects to Solibo Home.
- The bundled MCP server uses the remote HTTP transport.
- Solibo Home OAuth scopes and user permissions continue to gate tool access.
