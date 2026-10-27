# Changelog

All notable changes to the Solibo Home Claude plugin are documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and plugin versions follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.2.0] - 2026-10-27

### Changed

- Skill descriptions: the company briefing no longer claims the economy
  (money, budget and invoices point to `economy-review`), document filing
  names certificates, downloads and ZIPs, and portfolio sweeps apply to
  anyone who manages several companies, not only Solibo staff: the scope
  section now runs the sweeps over a non-staff user's own companies and keeps
  assignment lookups and acting-as to Solibo staff. A low-effort
  model routed "actuals against budget" to the briefing and skipped the
  document skill on "give me a download link" (local eval, 2026-10-26).

## [1.1.0] - 2026-10-26

### Added

- Eight skills under `skills/`: company briefing, conversation triage, resident
  communication, meeting agenda, economy review, portfolio sweeps (staff),
  document filing and supplier register. They encode the tool sequences,
  checks and paging rules that the server instructions used to carry.
- An Agent Plugins manifest (`plugin.json`, `mcp.json`, `assets/`) so the same
  folder packages the ChatGPT plugin; `scripts/package-chatgpt-plugin.sh`
  builds the submission ZIP.
- `PluginSkillsContractTest` keeps the skills in step with the tool catalog.
- `LICENSE` (MIT) and a `license` field in both manifests, as the Claude
  directory requires, with a trademark note in the README.
- The Claude manifest names the listing icon (`assets/solibo-home-logo.png`,
  the same file the ChatGPT manifest uses) and the website, support, privacy
  and terms URLs, so the directory listing no longer depends on an older
  upload.

### Changed

- The plugin description and README now describe Solibo Home by role and
  housing community, for Claude and ChatGPT.

## [1.0.0] - 2026-08-27

### Added

- Initial Claude Code plugin for the hosted Solibo Home MCP server.
- OAuth discovery through the production MCP endpoint.
- Disabled-by-default installation with permission-gated read, write, and
  resident-communication capabilities.

