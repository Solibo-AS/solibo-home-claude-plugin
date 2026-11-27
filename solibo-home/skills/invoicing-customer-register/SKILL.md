---
name: invoicing-customer-register
description: "Maintain billing customers and invoice recipients in Solibo Home (Kunderegister, Fakturamottakere). Use to add an external customer, change who gets a section's invoices after an ownership change, or update invoice addresses, references, EHF details, delivery methods, internal customer comments, or paper invoice fees."
---

# Invoicing customer register

Resolve the housing community with `search_companies` and pin it with `set_working_company`.
Find the billing customer with `search_invoicing_customers`. Use the returned ID and
`customerType` (PERSON or ORGANIZATION) together. When names match, check the section,
external status and active status before preparing a change. Use `get_invoicing_customer`
for the customer record and `get_invoicing_customer_metadata` for the reference, grouping
and reminder settings. A metadata ID of -1 represents defaults. Request optional email or
mobile fields only when the user explicitly needs them.

## Add or update a customer

- Use `add_external_invoicing_customer` with exactly one of an existing `personId` from
  `search_persons`, a `newPerson`, or an `organization`. A new person needs a name and either an
  email or a complete postal address. An organization needs its nine-digit organization number,
  name, organization form and complete postal address. Confirm the billing identity before adding
  it. Enter national identity numbers in the portal if needed.
- Use `update_external_customer_address` for the external customer's postal address. Owners'
  registered addresses follow the resident register. For a separate billing address or email, use
  `update_customer_invoicing_details`, which also handles invoice grouping, the invoice reference
  (up to 40 characters), and an organization's alternative EHF organization number. Send only the
  requested fields. `clearAlternativeAddress` removes both the alternative postal address and email.
- Use `add_invoicing_customer_comment` for an internal register note. The comment creates no
  customer notification.

## Change who receives a section's invoices

Find the section with `search_sections`, then read its current groups and recipients with
`get_section_invoice_recipients`. For an ownership change, confirm which owner or designated
contact person should receive the invoices. Prepare one `change_section_invoice_recipients` call
with the new recipient in `add` and the outgoing recipient in `remove`, each with the matched
customer type and ID. A section must retain at least one recipient. Show the current and proposed
recipients and explain that the change applies to coming invoices.

The tool checks that the addition took effect before removing the old recipient. If the result is partial or
the recipient readback fails, reread the section and prepare only the remaining changes.

## Delivery and paper fees

Use `set_invoice_delivery_method` for AUTO, EMAIL, DIGIPOST, LETTER (paper), or NONE. The choice
applies to all of that customer's invoices in the company. AUTO tries AvtaleGiro, eFaktura,
Digipost, email and paper in that order. Explain that an explicit selection overrides AvtaleGiro
and eFaktura. Use the alternative EHF organization number in the customer details when asked
to change EHF addressing.

Use `get_invoicing_settings` and `get_invoicing_accounts` before preparing a company paper fee
change with `update_invoice_settings`. The fee is 0 to 500 NOK, where 0 removes the fee. Use the
income account's `number` when changing the account. Show which future paper invoices it affects.

## Confirm the change

Show the company, customer or section, and the exact changes and current settings shown in the preview.
After explicit approval, call `confirm_pending_write` with only `previewId` and `confirmed` true.
Use other invoicing or common costs when the request is to schedule a charge.
