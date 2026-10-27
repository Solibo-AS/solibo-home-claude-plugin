---
name: economy-review
description: "Review the economy of one housing community in Solibo Home: expenses waiting for approval, supplier invoices to approve or reject, accounting actuals against budget, liquidity and bank balances, the accounting entries behind a number, produced customer invoices and their payments, loans and insurance. Use when the user asks about money, bills, invoices, budget, økonomi, regnskap, fakturaer, lån or forsikring for a company, or wants to approve or reject an expense or a supplier invoice."
---

# Economy review

Work on one company at a time: resolve it and call `set_working_company`, then omit `companyId`. Money
questions across many companies belong to the portfolio-sweeps skill; the one cross-company read that fits
here is `get_unapproved_invoices_overview`.

## Start with what exists

1. `get_economy_readiness`: `accountingAvailable`, `invoicingActive`, `unattestedExpenseCount`,
   `unattestedSupplierInvoiceCount`. A missing flag means unknown, and `accountingAvailable` false means
   "could not be confirmed", not "no accounting".
2. `get_company_economy_overview` for the picture: `economyStatus`, `loans[]` and `insurances[]` (samples of
   up to 20), `economicReports[]`, `unpaidInvoices[]` (supplier invoices).
3. `get_economy_status` for available funds and bank balances. `availability` is AVAILABLE, UNAVAILABLE,
   NOT_CONFIGURED, FORBIDDEN or UNKNOWN and describes bank statements only; an omitted amount is not zero;
   `source` SYNTHETIC means fixture data, say so.

## Periodic review

The usual monthly set is `search_expenses`, `get_accounting_overview`, `get_invoices`,
`get_company_economy_overview` and, if the user wants the board view, `get_board_report`.

- `get_accounting_overview`: `companyId` as an integer, `periodStart` inclusive and `periodEnd` exclusive
  (YYYY-MM-DD, 1 to 1096 days). `totals[]` per currency holds actual and budget income, expenses and net
  with variances; `dimensions[]` are projects and departments; `liquidity[]`. Amounts are decimal strings.
  Compare currencies separately. `budgetAvailable` false means "no budget", never zero. Report
  `company.dataAsOf` and `company.dataStatus` with the numbers.
- `search_accounting_entries` for the lines behind a figure: same period arguments, `companyId` (integer),
  `accountNumbers` (1000 to 9999), `projectIds`, `departmentIds` or `supplierIds` (one company only),
  `search` of 3 to 100 characters matching account, project, department or supplier names, not voucher
  text. Pages hold at most 100 rows: follow `paging.next`. Do not page through everything to compute a total
  that the overview already gives.
- `get_accounting_project` for one project; `projectId` comes from `dimensions[].id` in the overview.
- `search_expenses` with `status` FOR_APPROVAL, APPROVED or REJECTED and `dateOfPurchaseFrom` or
  `dateOfPurchaseTo`. `companyId` on an item is filled only when `companyIds` named one company; when
  `companyScopeResolved` is false, search again with one company before `get_expense` or any decision.
- `get_invoices` lists supplier invoices: `archive` false is the approval queue, a `year` implies the
  archive (0 means every year). `tripletexId` is the id that approve and reject take.
- Produced customer invoices: `get_company_invoice_history` with `periodStart` and `periodEnd`, or
  `get_invoice_history_for_companies` with `companyIds`; `open` true for unpaid. Cursors may start with
  "portfolio:", pass them back unchanged. `get_invoice_payments` lists the payments of one customer invoice
  (`invoiceId` from the history). Keep supplier invoices and customer invoices apart in the answer.
- `get_loans`, `get_insurances` and `get_economic_reports` with `companyId`; loan and policy numbers are
  masked to the last four digits.

## Approvals

Every approval or rejection is a preview followed by a confirmation. The user names the exact item
(supplier or submitter, amount, date) before you preview. One item per preview; never "approve all".

- Expenses: find the item with `search_expenses` scoped to the company, then `accept_expense` or
  `reject_expense` (`reason` is shown to the submitter; rejecting needs board-member access). Show the
  preview, and after the user approves call `confirm_pending_write` with `previewId` and `confirmed` true.
- Supplier invoices: `get_company_supplier_invoice_approval_status` gives `forApprovalInvoiceIds` and
  `invoices[].tripletexId`. If `invoiceRowsUnavailable` is true there are no ids and approval must happen in
  Solibo Home; say so. `approve_supplier_invoice` or `reject_supplier_invoice` take `invoiceId` set to the
  `tripletexId`; an id outside the queue is refused before anything changes.
- Across companies: `get_unapproved_invoices_overview` with `companyIds` (narrow it if it reports too many
  rows), `get_supplier_invoice_approval_counts` for cached counts. Companies listed as unverified are
  unknown, not zero.

A confirmation result with `pendingAction.status` DRY_RUN changed nothing. An outcome-unknown error means
check Solibo Home before repeating.

## Identifiers

KID numbers, bank account numbers, loan numbers and policy numbers are masked by default. Use
`reveal_loan_numbers` or `reveal_insurance_policy_numbers` only for one narrow purpose the user has stated
and approved; they are audited and default to five rows. Decline "all identifiers" exports and offer a masked
report instead.

## Answer shape

A table with period, currency, actual, budget and variance where available; the data freshness; a list of
decisions waiting with ids and amounts; then what could not be read. Never present unverified or
unavailable data as zero.

`references/tool-notes.md` lists the arguments and result fields of every tool named here.
