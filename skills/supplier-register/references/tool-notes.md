# Tool notes for the supplier register

`supplierId` is always the catalogue id: `search_suppliers` `items[].id` or `search_company_suppliers`
`items[].supplierId`. Write tools return a preview; confirm with `confirm_pending_write` and only
`previewId` and `confirmed` true.

| Tool | Kind | Required | Optional | Result fields to use |
|---|---|---|---|---|
| `search_suppliers` | read | none | `search`, `count` (100), `pageToken` | `items[].{id,name,orgNr,serviceCategories}` |
| `search_suppliers_by_orgnr` | read | `orgNrs` (array or comma list) or `search` | `count`, `pageToken` | `items[].{id,name,orgNr}` |
| `search_company_suppliers` | read | `companyId` | `search`, `count` (100), `pageToken` | `items[].{id (relationship),supplierId,name,orgNr,serviceCategories,globalServiceCategories,createdAt,createdByName}` |
| `get_company_supplier` | read | `companyId`, `supplierId` | none | same plus `comments[].{id,content,createdAt,emoticon,createdByName}` |
| `create_company_supplier` | preview, write confirm | `companyId`, `name`, `orgNr` | none | `supplierId`, `companySupplierId`, `action` created |
| `delete_company_supplier` | preview, write confirm, destructive | `companyId`, `supplierId` | `name` | `deleted` |
| `update_company_supplier_categories` | preview, write confirm | `companyId`, `supplierId`, and `serviceCategories` or `addServiceCategories`/`removeServiceCategories` | none | `serviceCategories` |
| `add_supplier_comment` | preview, write confirm | `companyId`, `supplierId`, `content` | `emoticon` | `commentId` |
| `set_supplier_comment_emoticon` | preview, write confirm | `companyId`, `supplierId`, `commentId` | `emoticon` | `action` |
| `list_company_supplier_contacts` | read | `companyId` | `supplierId`, `count` (100), `pageToken`, `includeFields` | `items[].{contactId,supplierId,name,title,email,mobile}`, `returnedCount` |
| `create_supplier_contact` | preview, write confirm | `companyId`, `supplierId`, `name`, `email` or `mobile` | `title` | `contactId` |
| `update_supplier_contact` | preview, write confirm | `companyId`, `supplierId`, `contactId`, one of `name`, `email`, `mobile`, `title` | none | `action` updated |
| `delete_supplier_contact` | preview, write confirm, destructive | `companyId`, `supplierId`, `contactId` | `name` | `deleted` |
| `get_supplier_invoice_notification_settings` | read | `companyId` | none | `futureInvoice`, `notifyIds[]` |
| `update_supplier_invoice_notification_settings` | preview, write confirm | `companyId`, `notifyIds` or `futureInvoice` | none | `updatedNotifyIds`, `updatedFutureInvoice` |
| `list_available_company_administrators` | read | `companyId` | none | person ids for `notifyIds` |
| `search_organizations` | read | none | `search` (at least 2 characters), `count`, `pageToken` | `items[].{id,name,orgNr,classification,roles[]}` |
| `get_organization` | read | `organizationId` | `includeFields` | `name`, `orgNr`, `businessAddress`, `employees[].{id,name}` |
| `search_accounting_entries` | read | `periodStart`, `periodEnd`, `companyId` (integer) | `supplierIds`, `search`, `count` (100), `pageToken` | `items[].{postingDate,accountNumber,amount,currency,supplierId (accounting id),supplierName}` |
