---
name: receivables-follow-up
description: "Work through overdue customer invoices and customer debt in Solibo Home. Use for Videre behandling, restanser, restansebrev, purring, postponing collection, crediting or closing a produced customer invoice, or creating or cancelling a claim in kundereskontro. Applies where the signed-in user has the relevant invoicing access."
---

# Receivables follow-up

## Review arrears

1. Use `get_overdue_customer_invoices_overview` for company totals. When the user explicitly asks
   who owes in one company, pass its `companyId` and `includeCustomers` true. Report the amounts
   by reminder stage, amounts older than six months and omissions from row limits. Restansenivå is the amount past the second reminder as a share of yearly
   common costs.
2. For one customer, pass the company ID and the customer's `customerType` and `customerId`
   to `get_customer_overdue_invoice_details`. Use `invoiceIds` only when selecting the invoices
   for the restansebrev. Summarize the returned letter lines and total. The letter is a read-only
   preview, and its PDF opens in the portal.
3. Use `get_customer_invoices` for the customer's produced invoices or `get_invoice_payments`
   to check payments on a specific invoice. Keep amounts invoiced, paid and still outstanding
   distinct. Use `get_customer_ledger` to find open claims, claimable debts and the reasons
   other debts are ineligible.
4. For the company's account 1500 postings, use `get_company_customer_ledger` and report the
   last Tripletex sync time. If the user requests a fresh sync, prepare
   `refresh_company_customer_ledger` and confirm it before requesting the sync. If it is running,
   report that status and reread the ledger when needed.

## Prepare the requested action

| User intent | Workflow |
|---|---|
| Postpone reminders or collection (utsett innkreving) | Use `set_invoice_overdue_delay` with the open invoice references and the agreed start and optional end. Use `end_invoice_overdue_delay` to change an active delay's end, or `remove_invoice_overdue_delay` to remove one before it starts. Show the affected invoices and the dates. |
| Credit a produced invoice | Check `get_invoice_credits` and use `credit_customer_invoice` for the exact invoice. Omit lines for a full credit when nothing was credited earlier, or specify line numbers and positive NOK amounts for partial credit. The credit note is booked to accounting and cannot be withdrawn. |
| Close for manual handling (lukk faktura) | Use `close_customer_invoice` with a reason. This permanently stops reminders, interest and collection. Closing creates no credit note, loss posting or customer notice. |
| Create a claim (opprett krav) | Use `create_customer_ledger_claim` with chosen debts from the ledger's `claimableDebts`. Show principal moved, interest, fee, extra lines and any old fees or interest being written off. The preview is Solibo Home's own calculation. |
| Cancel a claim or handle a posting | Use `cancel_customer_ledger_claim` for an unpaid claim with a reason. The original debt remains. Use `approve_customer_ledger_posting` only when the user confirms the posting is genuine debt, or `revoke_customer_ledger_posting_approval` when undoing an approval that no claim covers. |

For a write, pin the working company with `set_working_company` and create the action preview.
Show the exact customer, invoices or postings and the financial or collection effects. After
explicit approval, call `confirm_pending_write` with only `previewId` and `confirmed` true.
If Solibo Home reports the ledger changed, prepare a fresh preview for approval.
