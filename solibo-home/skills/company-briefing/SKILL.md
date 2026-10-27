---
name: company-briefing
description: "Status briefing for one housing community (borettslag, sameie, boligselskap) or a portfolio in Solibo Home. Use when the user asks how a company is doing, wants a board briefing, a morning or weekly brief, priorities, what needs attention, or an overview of open tasks, cases, conversations and meetings. Covers resolving the company by name, choosing between board report, snapshot and activity digest, and drilling down only where the briefing shows a need. Money, budget, invoices and bills belong to economy-review."
---

# Company briefing

This skill is guidance, not data. Every count, item, amount and link in a briefing comes from a tool result in
this conversation: after loading the skill, call the aggregate tool, then answer. Never answer a status question
from the skill text or from memory, and when the user rules out a source (for example the economy tools), say
that part is unavailable instead of describing it.

Answer status questions with one aggregate call, then drill down only where the aggregate shows a need or
the user asks. Do not rebuild the picture by looping `search_tasks`, `search_issues`, `search_meetings`,
`search_documents` and `search_conversations` over a company: `get_company_snapshot` and `get_board_report`
already read those sources together.

## Before you start

1. If no session context is loaded in this conversation, call `initialize_session_context` once. Read
   `audience.recommendedMode` and `relevantCompanies`. With exactly one company the working company is set
   for you. Never call it for a request that involves passwords, codes or tokens.
2. Resolve the company from what the user wrote: `search_companies` with the name or organisation number
   (the id is `items[].id`); use `search` when the words could also name a person. Do not ask for an id the
   user already gave as a name. Several matches: ask which one.
3. For a series of questions about one company, call `set_working_company` once. Later calls may omit
   `companyId`; passing a different `companyId` while a working company is set is an error.

## Pick the aggregate

| The user wants | Call |
|---|---|
| Board briefing, "prepare me", morning brief, priorities, what needs attention | `get_board_report`: open with `answerDraft`, then `counts`, then `priorities[]` (each has `reason`, `recommendedAction`, `markdownLink`). `limit` and `priorityLimit` size the lists; `responseLanguage` sets the language of the draft. |
| Broad status: profile, board, conversation counts, cases, tasks, meetings, economy summary | `get_company_snapshot` |
| What is new or recent | `get_company_activity_digest` (samples of 10 conversations, cases and tasks, 5 meetings and ownership changes; `pendingTasks` is never complete) |
| Several companies ranked by workload | `get_portfolio_overview` with `status` ACTIVE and `count` up to 1000; `answerDraft` opens the answer |
| Profile only: organisation number, administrators, business agreement, addresses | `get_company` |

## Drill down only when needed

- Open tasks: `search_tasks` with `companyIds` [id] and `completed` false; `dueDateFrom` and `dueDateTo`
  (ISO dates, inclusive Oslo days) for "this week" or "overdue". Follow `paging.next` until it is absent
  before calling a list complete or empty.
- Cases: `search_issues` with `status` ["OPEN", "IN_MEETING"]; `get_issue` for description and comments.
- Meetings: `search_meetings` (newest planned start first); the next meeting and its agenda belong to the
  meeting-agenda skill.
- Conversations: `get_conversation_meta` gives exact NEW, IN_PROGRESS and DONE counts; reading and
  answering threads belongs to the conversation-triage skill.
- Economy: `get_company_economy_overview` for status, loans, insurances and unpaid supplier invoices;
  deeper reading belongs to the economy-review skill.

## Morning brief for "my companies"

Run `initialize_session_context`, then `get_my_conversations` with `unread` true (keep the same `unread`
value when paging; the whole queue has no total count), then `search_tasks` with `companyIds` set to the
user's companies, `completed` false and `dueDateTo` yesterday for overdue tasks (a date bound covers that whole
Oslo day, so tasks due today are a separate list; follow `paging.next`), then for
each company where the user approves invoices `get_company_supplier_invoice_approval_status`, and only then
`get_conversation` for the two or three threads worth summarising. Do not open every conversation: summarise from titles, counts and
`readByCurrentUser`, and name the threads that need a reply.

## Reading the numbers

- A source that could not be read is not zero. Check `sources.<name>.status` and `complete`; say "could
  not be read" instead of reporting 0.
- `activity.openIssueCount` is exact; `activity.minimumPendingTaskCount` is a lower bound.
- Rankings in `get_portfolio_overview` cover the returned page only. Do not ask the user to narrow a
  portfolio only because it is large; continue with `pageToken`.
- Personal email and mobile numbers appear only when the user asked for them (`includeFields`).

## Permissions

A permission error on one source means the account cannot see it: say so and continue with the rest. Before
proposing any change, call `list_user_actions` with the `companyId` unless the permissions are already known
in this conversation. Staff-only tools (acting as another administrator, staff portfolios) are available only
when `audience.isSoliboStaff` is true.

## Answer shape

1. Three to five priorities, each with why it matters and the next action.
2. A short table of counts: new and in-progress conversations, open cases, open tasks, upcoming meetings.
3. Links as returned (`markdownLink`, `linkLabel`), never rewritten.
4. One line on what was sampled, truncated or unavailable.

`references/tool-notes.md` lists the arguments and result fields of every tool named here.
