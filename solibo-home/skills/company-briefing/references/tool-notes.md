# Tool notes for company briefings

Ids are strings unless noted. Read tools accept `includeFields` (["email"], ["mobile"] or both) to disclose
personal contact fields for that call only.

| Tool | Required | Useful optional arguments | Result fields to use |
|---|---|---|---|
| `initialize_session_context` | none | none | `audience.{kind,isSoliboStaff,canActAs,recommendedMode}`, `companies[].{id,name,orgNr,status,roleTypes}`, `relevantCompanies[]`, `roleContexts[]`, `defaultCompanyScope`, `workingCompany.{companyId,companyName}`, `guidance[]` |
| `search` | `query` | `count`, `companyPageToken`, `personPageToken` | `companies[].{id,name,orgNr,status}`, `persons[].{id,name}`, `sources.companies.next`, `sources.persons.next` (persons time out after 6 s: `sources.persons.status` TIMEOUT) |
| `search_companies` | `search` | `count` (100), `pageToken` | `items[].{id,name,orgNr,classification,status}`, `paging.next` |
| `get_company` | `companyId` | `boardPageToken` | `id,name,orgNr,status,dedicatedEmail,invoiceReception`, `businessAgreement.{startAt,signedAt,annualMeetingIncluded}`, `addresses[]`, `administrators.{administration,accounting,collection,ownership}`, `boardMembers[]`, `boardCoverage.paging.next` |
| `get_company_snapshot` | `companyId` | none | `company`, `board[].{id,name,roles,newConversations,inProgressConversations}`, `conversations.{new,inProgress,total}`, `issues[]`, `tasks[]`, `upcomingMeetings[].{id,title,plannedStartAt}`, `economy`, `activity.{openIssueCount,minimumPendingTaskCount}`, `sources.*` |
| `get_board_report` | `companyId` | `responseLanguage` (nb, en), `limit` (5, max 20), `priorityLimit` (10, max 30) | `answerDraft`, `answerGuidance`, `counts.{newConversations,inProgressConversations,openIssues,openTasks,upcomingMeetings,importantDocuments}`, `priorities[].{priority,sourceType,sourceId,reason,recommendedAction,markdownLink,linkLabel}`, `documents[]`, `economy`, `sources.*` |
| `get_company_activity_digest` | `companyId` | none | `answerDraft`, `newConversations[]`, `openIssues[]`, `upcomingMeetings[]`, `pendingTasks[]`, `recentOwnershipChanges[]`, `sources.*` |
| `get_portfolio_overview` | none | `status` (ACTIVE), `sort` (active, name), `count` (1000), `pageToken`, `companyIds` | `answerDraft`, `companies[].{id,name,orgNr,status,sectionCount,activeConversations,administrators[]}`, `totalCount`, `paging.next` |
| `search_tasks` | none | `companyIds`, `search`, `completed`, `responsibleId` (`get_current_user` `personId`), `dueDateFrom`, `dueDateTo`, `count` (100), `pageToken` | `items[].{id,companyId,title,dueDate,completedAt,responsibleName}`, `paging.next` |
| `search_issues` | none | `companyIds`, `search`, `status` [OPEN, IN_MEETING, CLOSED], `responsibleId`, `createdAtFrom`, `createdAtTo`, `updatedSince`, `count` (20), `pageToken` | `items[].{id,companyId,title,status,responsibleName,meetingId,meetingTitle}`, `paging.next` |
| `search_meetings` | none | `companyId` or `companyIds`, `search`, `status` [PLANNING, NOTIFIED, IN_PROGRESS, TO_SIGN, SIGNED], `meetingDateFrom`, `meetingDateTo`, `count` (100), `pageToken` | `items[].{id,companyId,title,status,plannedStartAt,plannedEndAt,location,issueCount}`, `paging.next` |
| `get_conversation_meta` | `companyId` | `foldersParentId`, `count` | `counts.{new,inProgress,done,draft,total}` (total includes trashed), `categories[].{id,name}`, `folders[].{id,name,parentId,unreadCount}` |
| `get_my_conversations` | none | `unread` (omit for the whole queue), `count` (10 company groups), `pageToken` | `readState`, `groups[].{companyId,companyName,conversations[].{id,title,status,conversationStartedAt,readByCurrentUser}}`, `paging.next` |
| `get_company_supplier_invoice_approval_status` | `companyId` | none | `forApprovalCount`, `invoices[].{tripletexId,supplierName,amount,currency,dueDate}`, `invoiceRowsUnavailable` |
| `get_company_economy_overview` | `companyId` | none | `economyStatus`, `loans[]`, `insurances[]`, `economicReports[]`, `unpaidInvoices[]`, `sources.*` |
| `get_session_context` | none | none | `actingAs`, `defaultCompanyScope`, `workingCompany`, `permissions.{actions[],complete}` |
| `set_working_company` | `companyId` | none | `companyId`, `companyName`, `message` |
| `clear_working_company` | none | none | `message` (keeps acting-as and portfolio scope) |
| `list_user_actions` | none | `companyId`, `count`, `pageToken` | `actions[]`, `paging.next` |
| `get_current_user` | none | `includeFields` | `personId` (use this for `responsibleId`), `name`, `roles[].{type,companyId,sectionId}` |

Dates: ISO dates (YYYY-MM-DD) are inclusive Oslo days; timestamps need an offset. Page tokens are opaque:
pass `paging.next` back unchanged and never retype it. A missing `paging.next` means the list is complete.
