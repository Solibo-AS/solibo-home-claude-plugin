---
name: payment-allocation
description: "Handle incoming payments in Solibo Home (Innbetalinger). Use to investigate unmatched or unknown-KID payments, register a manual payment, allocate a payment to invoices or customer credit, prepare a refund voucher, or retry, reverse or reallocate a payment."
---

# Payment allocation

## Find the payment and its current state

- Use `search_payment_sources` across accessible companies or with the requested company scope. Pass a
  date range, payer, KID or state when known. The default lists payments that need handling. Set
  `unprocessedOnly` false when looking for a settled or reversed payment without a state filter. Follow
  `paging.next` for complete results. Resolve an ambiguous match by company, amount, date and payer.
- Pass the row's `companyId` and payment source ID to `get_payment_source`. Inspect the payment's
  workflow state, allocation history and KID match. Use `candidates` true for eligible invoices and
  their outstanding amounts. If `moreCandidates` is true, narrow by invoice number or by the payer's
  type and ID. Candidate filters use PERSON or COMPANY for the payer type.
- Keep the returned KID and sender account masked. When `writesPaused` is true or unavailable, report
  the limitation and stay with reads. The returned `actions` are state hints. Solibo Home checks access
  and the current state when preparing the action.

## Choose the action

| User intent | Workflow |
|---|---|
| Register a payment received outside the bank feed | Resolve the customer with `search_invoicing_customers`. Use `register_manual_payment` with the payer, NOK amount, payment date and description. Review the bookkeeping account, which defaults to 1920. Registration creates the payment source for a later allocation. |
| Apply a received payment | Use `allocate_payment`. Omit allocation lines only when the user wants the invoice identified by the KID and the returned match is clear. Otherwise use the chosen candidate invoice IDs and amounts. The preview must account for the full payment, including any remainder. |
| Apply credit, leave a remainder unallocated, or prepare a refund voucher | Use the allocation targets CUSTOMER_OVERPAYMENT, UNAPPLIED or REFUND_PENDING as requested. To close a received payment without an invoice, use `close_payment_source` with UNAPPLIED or REFUND_PENDING and a reason. A refund voucher draft still needs processing. |
| Retry a stopped allocation | Use `retry_payment_allocation` when offered for the source. It retries a FAILED_RETRYABLE allocation or re-arms an eligible Tripletex booking with a reason. If the provider outcome is uncertain, read the source and report its state. |
| Undo or replace a settled allocation | Preview `reverse_payment_allocation` with the reason. Reversal reopens the invoices. If required, set `confirmResidualUnused` only after the user verifies that the residual has not been used, refunded or reallocated. Reread the source and verify REVERSED before preparing `reallocate_payment` with current invoice targets, amounts and a reason. Confirm the new allocation separately. |

For a split invoice payment and refund, put both amounts in explicit allocation lines. For example,
allocate NOK 1,000 to the matched INVOICE and NOK 250 to REFUND_PENDING. The refund line prepares
a voucher draft. `addResidual` supports credit or unallocated amounts, and closing a source without
an invoice applies the disposition to the full payment.

For allocation, show the company, payment amount, selected invoice numbers and amounts, customer
credit or other remainder, and any voucher or reversal effects from the preview. Get the user's
reason for an allocation to another customer, a payment without a payer, or an unallocated or refund
line. A rejection requires a voucher description. For an imported plan, tell the user that its
Tripletex and Kravia entries require manual reversal.

Omit `confirmed` when preparing the action. After explicit approval of the specific preview, call
`confirm_pending_write` with only `previewId` and `confirmed` true. Report the command status and read `get_payment_source` for the actual outcome
if it is pending or unclear. During an unsettled outcome, wait for the source state instead of
preparing a replacement action.
