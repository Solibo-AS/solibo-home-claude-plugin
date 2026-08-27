# Solibo Home Claude Plugin

This Claude Code plugin connects Claude to the hosted Solibo Home MCP server.
It can search the property-management data your account can access and can
prepare permission-gated changes and resident communications for explicit
confirmation.

The plugin does not ship credentials. Claude Code discovers OAuth from
`https://mcp.home.solibo.no` and prompts each user to sign in with their own
Solibo Home account.

## Install

After approval in the Claude Community marketplace:

```text
/plugin marketplace add anthropics/claude-plugins-community
/plugin install solibo-home@claude-community
```

The plugin is disabled immediately after installation. Enable it in `/plugin`,
then open `/mcp` and authenticate the `solibo-home` server.

## Permissions and safety

- Solibo Home remains the source of truth for data and permissions.
- Read access requires the `solibo.mcp/read` OAuth scope.
- Changes require `solibo.mcp/write`; resident communications require
  `solibo.mcp/communicate`.
- Sensitive changes and communications use preview and confirmation flows
  before anything is saved, sent, or published.
- Production currently provides tools only; interactive MCP Apps are not
  enabled.

## Local Validation

From the plugin directory:

```sh
claude plugin validate . --strict
```

For a local smoke test:

```sh
claude --plugin-dir .
```

Then run `/mcp` in Claude Code and authenticate the `solibo-home` server.

## Published Behavior

- The plugin is disabled by default so users explicitly opt in before Claude
  connects to Solibo Home.
- The bundled MCP server uses the remote HTTP transport.
- Solibo Home OAuth scopes and user permissions continue to gate tool access.

## Support and policies

- Documentation: <https://mcp.solibo.no>
- Support: <https://www.solibo.no/kontakt>
- Privacy policy: <https://www.solibo.no/privacy-policy>
- Terms of service: <https://home.solibo.no/privacy/>
