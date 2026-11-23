# Changelog

All notable changes to the Solibo Home Claude plugin are documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and plugin versions follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.5.1] - 2026-11-23

### Changed

- The changelog no longer spells out the path of the listing logo. The Claude directory check read the
  path as an image file that the plugin's code could reach.

## [1.5.0] - 2026-11-21

### Added

- `company-briefing`: `get_my_follow_ups` lists the user's own supplier invoices and expenses to approve and
  protocols to sign across all their companies.
- `conversation-triage`: `mark_conversations_spam` and `mark_conversations_not_spam` (spam is hidden from
  `search_conversations` unless `status` includes SPAM), and `delete_internal_note` / `restore_internal_note`
  for the author's own internal notes.
- `economy-review`: `get_bank_account_transactions` for one bank account's statement transactions (with the
  `bankAccounts[].id` that `get_economy_status` now returns), `get_customer_ledger` for one invoicing
  customer's ledger, and `delete_economic_report` for an ordered report.
- `portfolio-sweeps`: paused payment reminders (`get_invoicing_overdue_delays_for_companies`) and new tenants
  and sublets (`list_recent_tenant_registrations`) across companies.
- `resident-communication`: a section on homepage post comments and the neighbourhood (nabolaget): reading
  groups, posts and comments, and writing, editing and deleting the user's own comments and posts.
- `supplier-register`: `update_supplier_comment` and `delete_supplier_comment`.

### Changed

- `economy-review` and `meeting-agenda` no longer label tools by the role that may use them ("Solibo staff
  only", "Solibo superuser only"); they say the tool works when the user's access includes it, matching the
  tool descriptions since 2026-11-21. The server still withholds those tools from callers without access.

## [1.4.0] - 2026-11-11

### Changed

- All eight skills follow the tool behaviour after the November dev retests:
  previews refuse unknown or mismatched records before a confirmation, name
  the records and values they change, and confirm-time errors pass on Solibo
  Home's reason.
- `conversation-triage`: a draft needs a category to be published or
  restored; `create_conversation_inquiry` takes a contact category only;
  bulk delete reports `deletedIds` and `trashedIds`; deleting a folder also
  deletes its subfolders; `updatedSince` is the filter for "what changed".
- `document-filing` and `economy-review`: new files come from any public
  HTTPS download link; an existing document is reused with
  `sourceDocumentId`; expense receipts and approvals (two different
  approvers, `fullyApproved`) and the masked expense payout account with
  `reveal_expense_account_numbers`; only OTHER or BUILDING archive documents
  can be trashed.
- `company-briefing`: `get_company_activity_digest` for "what is new".
- `meeting-agenda`, `resident-communication`, `supplier-register` and
  `portfolio-sweeps`: tool notes match the current inputs and outputs.

### Removed

- References to the removed public conversation tools
  (`reply_to_public_conversation`, `create_public_conversation_inquiry`,
  `remove_public_conversation_document`): the plugin always signs in.

## [1.3.0] - 2026-10-28

### Added

- `resident-communication` covers one-off custom selections (egendefinert
  utvalg): find the sections with `search_sections` by entrance letter,
  apartment number or H-number, then pass `recipientSectionIds` with
  `recipientSectionResidents` on `draft_customer_message`. The tool notes
  list both fields. The recipe left the server instructions so they stay
  equal to the approved live text.

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
- The Claude manifest names the listing icon (the same logo file the ChatGPT
  manifest uses) and the website, support, privacy
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

