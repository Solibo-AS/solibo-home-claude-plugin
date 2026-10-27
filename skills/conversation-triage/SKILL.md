---
name: conversation-triage
description: "Work through the conversations (henvendelser, meldinger) of a housing community in Solibo Home: how many are open, which need an answer first, what a thread says, replying, internal notes, moving threads to folders, closing, reopening or flagging many at once. Use when the user asks about unanswered or open conversations, their inbox for a company, what residents are writing about, or wants to answer, file or close conversations."
---

# Conversation triage

Count first, read a sample second, open full threads last, and change many conversations with one bulk
tool and one confirmation.

## Counts and sample

1. Resolve the company (`search_companies`) and, for a session about one company, `set_working_company`.
2. `get_conversation_meta` gives exact counts: `counts.new`, `counts.inProgress`, `counts.done`,
   `counts.draft` (`counts.total` includes trashed threads). It also lists `folders[]` and `categories[]`,
   the source of every `folderId` and `categoryId`. "Open" means NEW plus IN_PROGRESS; read or unread is a
   separate signal (`readByCurrentUser`).
3. `search_conversations` for the list. `status` takes one value per call, so open threads are two calls,
   NEW and then IN_PROGRESS (DONE is an alias of COMPLETE; omitting `status` includes TRASHED). Add
   `search` for words, `createdSince` (a bare date is the
   start of that Oslo day), `folderId`, `categoryIds`, `count` (default 25). Follow `paging.next` only as
   far as the question needs; do not page through thousands of threads to answer "what is going on".
4. Across all of the user's companies, `get_my_conversations` groups by company; pass `unread` true for
   new mail and keep the same value while paging.

## Reading a thread

- `get_conversation` with `companyId` and `conversationId` returns the newest messages (oldest first, up to
  `messageLimit`, default 20) with each message cut to 100 characters, plus `status`, `folderId`,
  `categoryId`, `relatedIssueIds` and `participants`. This is enough to say what a thread is about and
  who wrote last.
- Open a few threads, not all of them. For "summarise every conversation this month", summarise from titles
  and the truncated previews and offer to read selected threads in full.
- `reveal_conversation_full_content` returns full message text and is logged as personal-data access. Use
  it only when the user asks for the content of a named thread, or asks you to answer it; one thread per
  call, never as a loop over a list.
- Contact fields appear only when asked for (`includeFields`).

## Answering

- `reply_to_conversation` writes a message the resident sees; `content` is required, documents go in
  `attachments`, `inlineDocuments` or `files`. It is a preview; after the user approves the text, call
  `confirm_pending_communication` with `previewId` and `confirmed` true. There is no draft or
  scheduling for replies: confirming sends the message to the resident at once, and an unconfirmed preview is
  temporary and saved nowhere, so make clear which of the two the user is choosing.
- `create_internal_note` is visible to staff and board only; `documentIds` may name existing documents.
  Confirm with `confirm_pending_write`.
- New threads: `create_conversation` with `title` and `participants` (each `participantId` from
  `search_persons`, `search_residents` or `search_organizations`, with `participantType`); or
  `create_conversation_inquiry` with a `categoryId` from the meta call. Both confirm through
  `confirm_pending_communication`. A DRAFT thread becomes visible with `publish_conversation`.
- `update_conversation_details` changes `title` or `categoryId`; its `content` only while the thread is
  DRAFT. `update_conversation` sets `status` DONE or IN_PROGRESS for one thread.

## Changing many threads at once

Use the bulk tools with `conversationIds` (up to 100 per call) and one confirmation, never a loop of
single-thread calls:

| Intent | Tool | Extra argument |
|---|---|---|
| File threads in a folder | `move_conversations_to_folder` | `folderId`, or `topLevel` true |
| Mark read or unread | `mark_conversations_read` | `read` true or false |
| Flag or unflag | `flag_conversations` | `flagged` |
| Close, reopen | `close_conversations`, `reopen_conversations` | none |
| Trash, restore | `trash_conversations`, `restore_conversations` | none |

`move_conversation_to_folder` and `mark_conversation_read` exist for a single thread; with two or more
ids use the bulk tool. `delete_conversations` is a soft delete; prefer trash when the user may want the
threads back. A new folder comes from `create_conversation_folder` (`name`, optional `parentId`).

Each bulk call returns a preview listing the ids and the action. Show the count and the folder or status,
get the user's approval, then `confirm_pending_write` with `previewId` and `confirmed` true. A result whose
`pendingAction.status` is DRY_RUN changed nothing.

## Answer shape

Counts first (new, in progress, done), then the threads that need a reply with title, started date and who
wrote last, then the suggested action per thread. Say when the list is a sample and when a thread was
summarised from truncated previews.

`references/tool-notes.md` lists the arguments and result fields of every tool named here.
