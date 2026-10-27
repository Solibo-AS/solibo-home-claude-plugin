# Tool notes for economy review

Ids are strings except in the three accounting tools, where `companyId`, `companyIds` and `projectId` are
integers. Dates are ISO dates (YYYY-MM-DD); timestamps need an offset.

| Tool | Required | Useful optional arguments | Result fields to use |
|---|---|---|---|
| `get_economy_readiness` | `companyId` | none | `accountingAvailable`, `invoicingActive`, `unattestedExpenseCount`, `unattestedSupplierInvoiceCount`, `unattestedSupplierInvoiceCountUnverified` |
| `get_company_economy_overview` | `companyId` | none | `economyStatus`, `loans[].{id,supplierName,loanType,remainingSum}`, `insurances[].{id,supplierName,premium}`, `economicReports[].{id,status,orderedAt}`, `unpaidInvoices[].{tripletexId,supplierName,amount,currency,dueDate,status,remainder}`, `sources.*` |
| `get_economy_status` | `companyId` | none | `availableFunds`, `availability`, `currency`, `source`, `dataNotice`, `bankAccounts[].{name,bank,balance,latestStatement}`, `loanAccounts[]` |
| `get_accounting_overview` | `companyId` (int), `periodStart`, `periodEnd` | none | `company.{dataAsOf,coverageFrom,coverageTo,complete,dataStatus,budgetAvailable,budgetStatus}`, `totals[].{currency,actualIncome,actualExpenses,actualNet,budgetIncome,budgetExpenses,budgetNet,expenseVariance,netVariance}`, `dimensions[].{dimension,id,name,totals,budgetAvailable}`, `liquidity[]` |
| `search_accounting_entries` | `periodStart`, `periodEnd`, `companyId` (int) or `companyIds` (ints, max 500) | `accountNumbers`, `projectIds`, `departmentIds`, `supplierIds`, `search`, `count` (100), `pageToken` | `items[].{companyId,id,postingDate,accountNumber,accountName,amount,currency,voucherId,projectName,departmentName,supplierName}`, `paging.next`, `meta.totalCount` |
| `get_accounting_project` | `companyId` (int), `projectId` (int), `periodStart`, `periodEnd` | none | `projectName`, `totals[]` |
| `search_expenses` | none | `companyIds`, `search`, `status` [FOR_APPROVAL, APPROVED, REJECTED], `dateOfPurchaseFrom`, `dateOfPurchaseTo`, `count` (20), `pageToken` | `items[].{id,companyId,companyScopeResolved,amount,expenseType,dateOfPurchase,description,status,payableToName}`, `paging.next` |
| `get_expense` | `companyId`, `expenseId` | none | `amount`, `status`, `expenseProcessing[].{result,performedAt}` |
| `accept_expense` | `companyId`, `expenseId` | `title` | preview, then `confirm_pending_write` |
| `reject_expense` | `companyId`, `expenseId` | `reason` | preview, then `confirm_pending_write` |
| `create_expense` | `companyId`, `account`, `amount`, `dateOfPurchase`, `expenseTypeId` (from `list_expense_types`), `payableTo` (person id from `search_persons` or `lookup_person`) | `description` | preview, then `confirm_pending_write` |
| `get_invoices` | `companyId` | `archive`, `year` | `invoices[].{id,tripletexId,supplierName,amount,currency,invoiceDate,dueDate,status,outstanding,remainder}` (unpaged) |
| `get_company_supplier_invoice_approval_status` | `companyId` | none | `forApprovalCount`, `forApprovalInvoiceIds[]`, `unattestedCount`, `invoices[].{tripletexId,supplierName,amount,currency,dueDate}`, `invoicesTruncated`, `invoiceRowsUnavailable` |
| `approve_supplier_invoice` | `companyId`, `invoiceId` (the `tripletexId`) | `reason` | preview, then `confirm_pending_write` |
| `reject_supplier_invoice` | `companyId`, `invoiceId` (the `tripletexId`) | `reason`, `description` | preview, then `confirm_pending_write` |
| `get_unapproved_invoices_overview` | none | `companyId`, `companyIds` | `companies[].{companyId,companyName,invoiceCount,earliestDueDate,invoices[]}`, `totalInvoices`, `unverifiedCompanyIds` |
| `get_supplier_invoice_approval_counts` | none | none | `counts[].{companyId,count}`, `totalCount` (cached) |
| `get_company_invoice_history` | `companyId`, `periodStart`, `periodEnd` | `count` (100), `pageToken` | `items[].{id,number,description,dueDate,invoiceDate,sum,open,type,sectionId,payerName,payerType,delivery}`, `paging.next` |
| `get_invoice_history_for_companies` | `periodStart`, `periodEnd` | `companyIds`, `search` (max 25 companies), `payerType`, `open`, `count`, `pageToken` | same item fields plus `companyId`, `paging.next` |
| `get_invoice_payments` | `companyId`, `invoiceId` | `count`, `pageToken` | `items[].{id,amount,currency,paymentDate,payerName,paymentHandling}` |
| `get_customer_invoices` | `companyId`, `customerType` (PERSON, ORGANIZATION), `customerId` (from `search_invoicing_customers`) | `count`, `pageToken` | `items[].{id,number,dueDate,sum,open}` |
| `get_loans` | none | `companyId`, `loanId`, `companyIds`, `search`, `active`, date bounds, `count`, `pageToken` | `items[].{id,supplierName,loanNumberMasked,loanType,initialSum,remainingSum,interest,startDate,endDate,inAgreement}` |
| `get_insurances` | none | `companyId`, `insuranceId`, `companyIds`, `search`, `count`, `pageToken` | `items[].{id,supplierName,policyNumber,premium,startDate,endDate}` (masked) |
| `get_economic_reports` | none | `companyId`, `reportId`, `companyIds`, `year`, `status` (ORDERED, DELIVERED) | `items[].{id,companyId,status,orderedAt}` or `reports[]` |
| `reveal_loan_numbers`, `reveal_insurance_policy_numbers` | as the masked tool | `count` (5) | full numbers; audited, explicit permission only |
| `confirm_pending_write` | `previewId`, `confirmed` (true) | none | `pendingAction.{status,executed,dryRun,message}`, `result` |
