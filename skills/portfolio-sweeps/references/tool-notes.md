# Tool notes for portfolio sweeps

Staff-only tools are hidden from non-staff lists: `get_staff_portfolios`, `set_acting_as`,
`clear_acting_as`, `get_team_performance`, `get_invoice_approval_backlog`. Paged reads return `items[]`,
`meta.totalCount` and `paging.next`. The accounting tools take integer ids; the rest take strings.

| Tool | Required | Useful optional arguments | Result fields to use |
|---|---|---|---|
| `initialize_session_context` | none | none | `audience.{kind,isSoliboStaff,canActAs,recommendedMode}`, `relevantCompaniesAreAllCompanies`, `workingCompany` |
| `get_staff_portfolios` | none | `includeFields` | `administrators[]` and `accountants[]` with `{id,name,roleType,companyCount,companies[].{id,name}}` (`id` is the `adminId`) |
| `set_acting_as` | `adminId` | none | `adminName`, `companyCount`, `scopeLabel` |
| `clear_acting_as`, `clear_company_scope`, `clear_working_company` | none | none | `message` |
| `get_portfolio_overview` | none | `status`, `sort` (active, name), `count` (1000), `pageToken`, `companyIds` | `answerDraft`, `companies[].{id,name,orgNr,status,classification,sectionCount,activeConversations,administrators[]}`, `totalCount`, `paging.next` |
| `search_company_details` | none | `companyIds` (integers), `search`, `count` (20, max 1000), `fields` (id, name, slug, orgNr, classification, status, managed, dedicatedEmail, invoiceReception, businessAgreement.startAt, businessAgreement.signedAt, address, sections, appEnabled, relations), `pageToken` | `items[]` with the chosen fields; a long `companyIds` list returns complete with no cursor |
| `update_multiple_company_details` | `items[].{companyId, at least one field}` (max 100) | `dryRun`; item fields `name`, `orgNr`, `status`, `managed`, `businessAgreement{startAt,signedAt,...}`, `relations[]`, `note`, `clientReference` | `summary.{dryRun,total,updated,validated}`, `items[].{clientReference,status,requestedFields}` |
| `get_portfolio_board_members` | none | `companyIds`, `search`, `count` (100), `pageToken`, `includeFields` | `items[].{id,name,title,roles[].{type,companyId}}` |
| `get_accounting_portfolio_overview` | `periodStart`, `periodEnd` | `companyIds` (integers, max 500), `count` (25, max 100), `pageToken` | `items[].{company.{companyId,companyName,dataAsOf,coverageFrom,coverageTo,complete,dataStatus,budgetAvailable},totals[],dimensions[],liquidity[]}`, `paging.next` |
| `search_accounting_entries` | `periodStart`, `periodEnd`, `companyId` or `companyIds` (integers) | `accountNumbers`, `search`, `count` (100), `pageToken` | `items[].{companyId,postingDate,accountNumber,accountName,amount,currency,supplierName}` |
| `get_invoice_history_for_companies` | `periodStart`, `periodEnd` | `companyIds`, `search` (max 25 companies), `payerType`, `open`, `count`, `pageToken` | `items[].{id,number,invoiceDate,dueDate,sum,open,payerName,payerType,companyId,delivery}` |
| `search_invoicing_plans_for_companies` | none | `companyIds`, `search`, `active`, `type`, `count` (50), `pageToken` | `items[].{id,name,active,type,companyId,nextPreview}` (`id` is the `planId`) |
| `get_invoice_plan_transactions_for_companies` | none | `companyIds`, `invoicePlanIds`, `active`, `type`, `overdueEnabled`, `createdAtFrom`, `createdAtTo`, `count` (100), `pageToken` | `items[].{companyId,id,createdAt,invoicePlanId,invoicePlanName,invoicePlanType,overdueEnabled}` |
| `search_invoicing_customers_for_companies` | none | `companyIds`, `search`, `active`, `customerType`, `count` (100), `pageToken`, `includeFields` | `items[].{id,companyId,active,type,name,sectionIds}` |
| `get_queued_invoices_for_companies` | none | `companyIds`, `count` (50), `pageToken` | `items[].{companyId,invoice.{id,payerName,invoiceDate,dueDate,claim,lineCount}}` |
| `get_invoicing_settings_for_companies` | none | `companyIds`, `count` (100), `pageToken` | `items[].{id,companyId,paperTax,paperTaxAccount}` |
| `get_invoicing_accounts` | `companyId` | `count`, `pageToken` | `items[].{id,name,accountNumber (masked),bank}`; not found when the company does not invoice |
| `search_invoicing_plans` | `companyId` | `active`, `type`, `count` (5), `pageToken` | `items[].{id,name,active,type}` |
| `get_invoice_plan` | `companyId`, `planId` | `at` | `name`, `active`, `type`, `nextPreview`, `settings`, `distributions[]`, `transactions[]`, `lines[]` |
| `get_invoice_plan_distributions`, `get_invoice_plan_lines`, `get_invoice_plan_settings` | `companyId`, `planId` | `count`, `pageToken`, `active` (lines), `at` (settings) | `items[]` or `revisions[]` |
| `preview_invoicing_plan` | `companyId`, `planId` | `count` (100; prefer one large page) | `items[].{boligNr,dueDate,invoiceDate,payer,lines[]}`, `blocked`, `issues[]` |
| `get_invoice_approval_backlog` | none | `includeFields` | `accountants[].{accountantName,totalInvoices,totalNok,totalOverdue,companies[].{companyName,invoices[].{tripletexId,supplierName,amount,dueDate,ageDays}}}`, `unverifiedCompanyIds` |
| `get_unapproved_invoices_overview` | none | `companyIds` | `companies[].{companyId,companyName,invoiceCount,earliestDueDate,invoices[].{tripletexId,supplierName,amount,dueDate}}`, `unverifiedCompanyIds` |
| `get_team_performance` | none | `includeFields` | `administrators[].{adminName,portfolioSize,totalNew,totalInProgress,totalOverdueTasks,workloadScore}`, `staleCompanies[].{companyName,newConversationCount,daysSinceOldest}` |
| `get_homepage` | `companyId` | `feedCount` | `feed[]`, `practicalInformation[]` |
| `get_board_report`, `get_company_snapshot` | `companyId` | see the company-briefing skill | single-company aggregates |
| `create_issues_for_companies`, `create_tasks_for_companies`, `create_conversations_for_companies`, `create_meetings_for_companies`, `create_feed_posts_for_companies`, `create_practical_info_for_companies` | `companyIds` (max 500), `titleTemplate` (+ `contentTemplate`, times or participants per tool) | `responsibleId`, `responsibleByCompany[]`, `dueDate`, `participants[]`, `participantsByCompany[]`, `location`, notify groups, `publishedAt`, `draft` | preview `companies[].{companyId,companyName,warnings}`, `unresolvedPlaceholders`; after `confirm_pending_write` `items[].{companyId,entityId,status,warnings}` |
| `send_feedback_to_solibo` | `category` (missing_feature, unexpected_behaviour, bug, data_issue, other), `message` | `contextSummary` | `submitted` after `confirm_pending_communication` |
