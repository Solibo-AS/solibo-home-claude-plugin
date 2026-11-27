---
name: other-invoicing
description: "Prepare or maintain customer invoices outside regular common costs in Solibo Home. Use for Annen fakturering, Massefakturering, an engangsfaktura, a recurring extra charge, a charge on the next felleskostnader invoice, or PDF attachments to a scheduled invoice. Applies where the signed-in user has the relevant invoicing access."
---

# Other invoicing

Resolve the housing community with `search_companies` and pin it with `set_working_company`.
Use `search_sections` for sections or `search_invoicing_customers` for direct customer billing.
A section charge goes to its configured invoice recipients. Use `get_section_invoice_recipients`
when the payer needs checking. Use the invoicing customer register skill to change a recipient
or add a customer. Get the income account's `number` from `get_invoicing_accounts`.
Confirm the audience, line text, NOK amount per recipient, income account and timing before preparing
the invoice if any of those details are ambiguous.

## Pick the invoice

| Request | Tool and inputs |
|---|---|
| One charge to one section or customer | `create_single_invoice` with `sectionId` or `customerType` and `customerId`, plus lines, `invoiceDate` and `dueDate`. Solibo Home produces and sends it on the invoice date. |
| Repeating charge to one section or customer | `create_recurring_invoice` with the same recipient choice, lines, `interval`, `startMonth`, and optional `endMonth`. Review the day and whether it is the invoice day (`INVOICE_DATE`) or due day (`DUE_DATE`). The schedule is fixed once created. |
| Same charge to every section or selected sections and customers | `create_ad_hoc_invoice_group` with `allSections` for every section or explicit `sectionIds` and/or `customers`. Choose separate invoice and due dates or set `addToNextCommonCostInvoice` without those dates. Review the named recipients and the common-cost run or separate invoice dates returned by the preview. |

Show the preview's recipients, amounts, total, schedule, attachment names and warnings. A due date
less than six days after the invoice date prevents eFaktura. If the user asked only for a preview,
stop there. After explicit approval, call `confirm_pending_write` with only `previewId` and
`confirmed` true. Report what the result says was scheduled or changed.

## Change an existing invoice

- For a single or recurring invoice, find its OTHER plan with `search_invoicing_plans` and
  read it with `get_invoice_plan`. Use `update_scheduled_invoice_lines` for future line amounts or
  text, and `stop_scheduled_invoice` to stop future production.
- For mass invoices, use `search_ad_hoc_invoice_groups` and `get_ad_hoc_invoice_lines`. Use
  `update_ad_hoc_invoice_lines` to change or remove lines and `delete_ad_hoc_invoice_group` to
  remove a whole group before it is invoiced. Review the exact group and lines in the preview.
- Attach PDF files when creating an invoice or use `update_invoice_attachments` on a running plan
  or uninvoiced group. Send file references through `files`, up to five PDFs, 5 MB each and 10 MB
  total. If the client cannot provide a supported file reference, direct the user to attach the
  PDF in Solibo Home. Review the file names, sizes and any removals before confirming.
- For an invoice that has already been produced, use receivables follow-up for crediting. For
  changes to regular felleskostnader, use the common-costs skill.
