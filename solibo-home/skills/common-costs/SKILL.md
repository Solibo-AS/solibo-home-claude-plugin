---
name: common-costs
description: "Review or change regular common-cost billing (felleskostnader) in Solibo Home. Use for a common-cost run preview, fee increases, plan lines, distribution keys (fordelingsnøkler), invoice timing, or turning a common-cost plan on or off."
---

# Common costs

Resolve the housing community with `search_companies` and set it as the working company with
`set_working_company`. Find its FELLESKOSTNADER plan with `search_invoicing_plans`.
Use `get_invoice_plan` for the plan, `get_invoice_plan_lines` for line revisions,
`get_invoice_plan_settings` for schedule revisions, and `get_invoice_plan_distributions`
for distribution keys. Check which revision is in force and whether a future one already exists.
A charge added to the next common-cost invoice without changing the regular plan belongs to other invoicing.

## Review a run

Use `preview_invoicing_plan` to see who will be billed, when, and what each invoice will contain.
By default it checks whether a run is due now. If none is due, use the plan's `nextPreview` or
the requested date as `at`, or use `forced` true for a hypothetical run. It reads the saved plan
and any saved revisions in force on that date. Label estimates for proposed edits that have yet to
be saved. Check `blocked`, `issues`, and `notice` before reporting the result. A payer can be
billed for multiple sections. Use `lineGroups` for the section breakdown, and use the payer
row's `total` once when adding up the run. Follow `paging.next` with the same date and forced
setting when a complete list is needed.

## Prepare a change

- **Fee lines.** Use `add_invoice_plan_line` with the whole company's yearly NOK amount, the
  income account, a distribution key, and the start month. Find the income account's `number`
  in `get_invoicing_accounts`. Use `change_invoice_plan_line` for a new yearly amount or percentage
  increase (rounded down to whole kroner), income account, distribution key or final month.
  Additions and changes start from a coming month. For a due-date plan, the change takes effect
  before that month's due day, so show the effective date returned by Solibo Home. A line can
  have one planned change at a time. Use `cancel_invoice_plan_line_change` if the user wants to
  remove that revision, then prepare the replacement after cancellation is confirmed. Use
  `rename_invoice_plan_line` for text alone and `set_invoice_plan_line_active` to pause one line.
- **Distribution keys.** Use `create_invoice_plan_distribution` for a custom key, either equal
  shares with optional exclusions or whole-number shares with a denominator equal to their sum.
  To change a custom key, use `update_invoice_plan_distribution` with the sections whose numerators
  change. Other numerators stay the same, but their fraction can change when the denominator changes.
  Review the denominator and the lines using the key. The change applies immediately to coming
  invoices for every line using the key. For a change to one line or from a future month, create
  a different key and set it on that line's revision. Use `delete_invoice_plan_distribution`
  only for a custom key with no references in any line revision.
- **Schedule and plan.** Use `update_invoice_plan_settings` for interval, invoice day (due day in
  due-date mode), days to due date or final month. The preview gives the effective date based on
  the next interval boundary. When changing intervals, Solibo Home uses the longer interval.
  The edit replaces any pending settings revision and keeps its values for omitted fields.
  Fewer than six days to due date prevents eFaktura. Use `set_invoice_plan_active` only when the
  user wants to switch the whole plan on or off. Switching it off stops production immediately.

Show the returned change preview with the affected plan, lines or sections, the old and new
values, and when they apply. After explicit approval, send only `previewId` and `confirmed` true
to `confirm_pending_write`. A request for a preview ends after showing it.
