---
name: supplier-register
description: "Suppliers (leverandører) of a housing community in Solibo Home: find a supplier in the shared catalogue or by organisation number, see which suppliers a company uses with their categories and comments, add or remove a supplier for a company, maintain supplier contacts, import suppliers that appear in the accounting entries, and the supplier-invoice notification settings. Use when the user mentions suppliers, vendors, leverandører, contractors, a company's supplier list, or supplier contacts."
---

# Supplier register

Two ids exist: the catalogue supplier id, which every company-supplier tool takes as `supplierId`, and the
relationship id (`items[].id` of `search_company_suppliers`), which none of them takes. The `supplierId` in
accounting entries is a third, company-local id and does not work here either.

## Find

1. Resolve the company; `set_working_company` for a session about one company.
2. `search_company_suppliers` with `companyId` and `search` lists the company's suppliers:
   `items[].supplierId` (pass this onward), `name`, `orgNr`, `serviceCategories`.
3. `get_company_supplier` with `supplierId` adds `comments[]` (each with `id`, `content`, `emoticon`).
4. The shared catalogue: `search_suppliers` by name, `search_suppliers_by_orgnr` with `orgNrs`; both
   return `items[].id` (the catalogue id), `name`, `orgNr`. A permission error means the account may not
   browse the catalogue; say so.
5. `search_organizations` and `get_organization` find organisations that are not suppliers yet.

## Add, change, remove

- `create_company_supplier` needs `companyId`, `name` and `orgNr`, all three. Find the organisation
  number first with `search_suppliers_by_orgnr`, `search_suppliers` or `search_organizations`; never guess
  it. With a working company set, omit `companyId`. Preview, user approval, then `confirm_pending_write`
  with `previewId` and `confirmed` true.
- `update_company_supplier_categories` with either `serviceCategories` (the final list; an empty list
  clears) or `addServiceCategories` and `removeServiceCategories`, not both.
- `add_supplier_comment` (`content`, optional `emoticon` such as THUMBS_UP) and
  `set_supplier_comment_emoticon` (`commentId` from `get_company_supplier`; omit `emoticon` to clear).
- `delete_company_supplier` removes the company's link to the supplier (destructive; confirm).

## Contacts

`list_company_supplier_contacts` with `companyId` and `supplierId` (keep paging even when a page returns
zero rows for that supplier); contact details appear only with `includeFields`. `create_supplier_contact`
needs `name` and `email` or `mobile`, from values the user supplied; `update_supplier_contact` keeps
omitted fields and refuses to clear both email and mobile; `delete_supplier_contact` is destructive.

## Import from the accounting entries

1. `search_accounting_entries` with the period and the company (integer id) gives `supplierName` per line;
   collect the distinct names, and the amounts if the user wants a ranking.
2. For each name, find the organisation number: `search_suppliers` or `search_suppliers_by_orgnr`.
   Names without a match go to the user as a list to resolve; do not create a supplier without an
   organisation number.
3. Compare with `search_company_suppliers` to skip suppliers already linked.
4. Create the rest with `create_company_supplier`, one preview and one confirmation each, and tell the user
   how many confirmations to expect before starting. Stop and ask if a preview is refused.

## Notifications

`get_supplier_invoice_notification_settings` (`futureInvoice`, `notifyIds`) and
`update_supplier_invoice_notification_settings` with `notifyIds` (person ids from
`list_available_company_administrators`; an empty list clears) or `futureInvoice`.

## Answer shape

A table of suppliers with organisation number and categories; for an import, three lists: linked already,
to create, and unresolved names. Report a supplier as added only from an executed confirmation.

`references/tool-notes.md` lists the arguments and result fields of every tool named here.
