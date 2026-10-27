---
name: portfolio-sweeps
description: "Work across many housing communities at once in Solibo Home: which companies an administrator or accountant is assigned, portfolio rankings, company detail exports (invoice reception, business agreements) as CSV, invoicing audits across plans and transactions, first-invoice follow-up for newly managed companies, accounting comparisons, approval backlogs, team workload, and the same record created in many companies. Use when the user asks about \"all my companies\", their portfolio, every borettslag or sameie they manage, or a list that spans more than a handful of companies."
---

# Portfolio sweeps

Use the tools that take `companyIds` and page, never a loop of single-company calls. One call per page
beats one call per company; a sweep over 800 companies with a per-company tool is the pattern to avoid.

## Scope first

1. `initialize_session_context` once. Staff see `audience.isSoliboStaff` true and every company as
   relevant. If it is false, the portfolio is the user's own companies from the session context (a board
   member or accountant with several communities): run the sweeps below over those `companyIds` and skip
   the staff-only tools, which are not listed for them (`get_staff_portfolios`, `set_acting_as`,
   `clear_acting_as`, `get_team_performance`, `get_invoice_approval_backlog`). Asked to act as another
   administrator, say that is a Solibo staff workflow and offer the user's own companies instead.
2. Staff only, "my assigned companies": `get_staff_portfolios` lists administrators and accountants with
   their companies; pass those explicit `companyIds` onward. Assignment is not the same as readable
   access, and do not impersonate anyone to answer. `set_acting_as` with an `adminId` from that list
   scopes later portfolio calls to that person's companies (needs `audience.canActAs`); `clear_acting_as`
   ends it.
3. A pinned working company narrows the `companyIds` tools to that company when `companyIds` is omitted.
   For a sweep, pass `companyIds` explicitly or `clear_working_company` first.

## Sweeps and their tools

| Ask | Call |
|---|---|
| Rank the portfolio by workload or activity | `get_portfolio_overview` with `status` ACTIVE, `count` 1000 (`answerDraft`, `companies[]`, `totalCount`) |
| Company facts as a list or CSV: org number, status, invoice reception, business agreement dates | `get_portfolio_overview` for the ids, then `search_company_details` with `companyIds`, `count` at least the number of ids, and `fields` such as [id, name, orgNr, status, invoiceReception, businessAgreement.startAt, businessAgreement.signedAt]; a long id list returns in one response |
| Board members across companies | `get_portfolio_board_members` with `companyIds`, follow `paging.next` |
| Accounting actuals, budgets and liquidity side by side | `get_accounting_portfolio_overview` with `periodStart`, `periodEnd` (exclusive, 1 to 1096 days), `companyIds` (max 500), `count` up to 100; follow `paging.next` before claiming completeness; compare currencies separately; a missing budget is "unavailable", never zero |
| Cross-company cost lines | `search_accounting_entries` with integer `companyIds`, `accountNumbers` or `search`; supplier, project and department ids need one company |
| Invoicing plans and their activity | `search_invoicing_plans_for_companies` (`type` FELLESKOSTNADER, KAPITALINNKREVING or OTHER, `active`), `get_invoice_plan_transactions_for_companies` (`invoicePlanIds`, `createdAtFrom`, `createdAtTo`); only then `get_invoice_plan`, `get_invoice_plan_distributions`, `get_invoice_plan_lines` or `preview_invoicing_plan` for the few plans that need detail |
| Customers, queued invoices, settings across companies | `search_invoicing_customers_for_companies`, `get_queued_invoices_for_companies`, `get_invoicing_settings_for_companies` |
| Companies without invoicing accounts | `get_invoicing_settings_for_companies` first; call `get_invoicing_accounts` only for companies known to invoice (it returns "not found" for the rest, and 900 such calls is the sweep to avoid) |
| First invoice for newly managed companies | `get_invoice_history_for_companies` for the target window, then the same call for the earlier comparison window with `companyIds` limited to the companies that appeared in the first result; `open` true for unpaid rows; `count` is capped at 200 without a short company list |
| Supplier-invoice approval backlog | `get_invoice_approval_backlog` (per accountant; `tripletexId` is the approve or reject id) and `get_unapproved_invoices_overview` with `companyIds` (narrow it if it reports too many rows); companies listed as unverified are unknown, not zero |
| Team workload and stale inboxes | `get_team_performance` (`workloadScore`, `staleCompanies[]`) |
| The same case, task, conversation, meeting, post or practical information in many companies | `create_issues_for_companies`, `create_tasks_for_companies`, `create_conversations_for_companies`, `create_meetings_for_companies`, `create_feed_posts_for_companies`, `create_practical_info_for_companies` with `companyIds` and templates such as {{company.name}}; one preview, one `confirm_pending_write` |
| Company master data in bulk (agreement dates, status) | `update_multiple_company_details` with `items[]` and `dryRun` true to validate; after approval the same `items` without `dryRun`, then `confirm_pending_write` |

Board report or snapshot for every company in a list: these are single-company tools
(`get_board_report`, `get_company_snapshot`). Offer the portfolio overview first; if the user still wants
per-company briefings, agree on the company list and say how many calls it takes before starting.

## Paging rules

- Follow `paging.next` until it is absent before saying a list is complete; pass the token back exactly
  as returned, never retyped. A missing `paging.next` means the end.
- Do not ask the user to narrow a request only because it has many rows; page instead. Do narrow when a
  tool reports a row or page limit.
- Repeat `includeFields` on every page when contact fields were requested.

## Output

CSV or a table with one row per company, the columns the user asked for, blank cells where a value is
absent, and a closing line with the row count, the pages read and the companies that could not be read.
Keep KID, account numbers, loan numbers and policy numbers masked unless the user approved a scoped
reveal. A record type with no multi-company tool goes to `send_feedback_to_solibo` as a request, not to
an ad hoc loop.

`references/tool-notes.md` lists the arguments and result fields of every tool named here.
