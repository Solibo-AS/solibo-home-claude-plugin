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
   `search` for words, `createdSince` or `updatedSince` (latest message or uploaded document at or after a
   time; a bare date is the
   start of that Oslo day; `updatedSince` is the one for "what changed since"), `readByCurrentUser`,
   `latestPublicMessageSide` (NON_BOARD means a board reply is due), `folderId`, `categoryIds`,
   `orderBy` (default LATEST_ACTIVITY_AT, newest first), `count` (default 25). Follow `paging.next` only as
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
  Confirm with `confirm_pending_write`. `get_conversation` returns the notes in `internalNotes`; never quote
  them in a reply the resident sees.
- New threads: `create_conversation` with `title` and `participants` (each `participantId` from
  `search_persons`, `search_residents` or `search_organizations`, with `participantType`); or
  `create_conversation_inquiry` with a contact `categoryId` from the meta call. Both confirm through
  `confirm_pending_communication`. Give `create_conversation` a `categoryId`: a draft without a category
  cannot be published or restored from the trash. `draft` true saves a DRAFT that nobody is notified about; it becomes
  visible with `publish_conversation`, which needs the category.
- Participants: `participants[].personId` (or `organizationId`) is the id every tool takes. `id` and
  `participantRowId` are the row in that one thread. `active` false is a former participant.
- `update_conversation_details` changes `title` or `categoryId`; its `content` only while the thread is
  DRAFT. `update_conversation` sets `status` DONE (also given as Complete) or IN_PROGRESS for one thread.

## Attachment transport

Use `files[]` with a caller-chosen `file.file_id` and downloadable HTTPS `file.download_url` on a public host;
`attachments[]` sources use exactly one matching `fileId` or discovered `sourceDocumentId`.
Signed links work without extra authorization headers. A client attachment is usable only when the
client provides that URL; a Claude local attachment ID alone is insufficient. Never invent URLs,
pass base64 or ask the user to publish a private file. If transfer is unavailable,
`get_document_upload_options` returns an authenticated Home area and instructions for manual upload;
it opens no upload modal. After the user uploads, discover `sourceDocumentId` and reuse it on the
reply/thread tool. Conversation files are visible to its participants, including residents.
The configured per-file maximum defaults to 20 MB; integrated limits are ten files, fifty document
sources and 50 MB of total transfers, counting repeated uses. Confirm using only the preview ID and
`confirmed` true.

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
ids use the bulk tool. `delete_conversations` is a soft delete that only works on drafts (unless the user
is a super user, whose published threads go to the trash instead); Solibo Home refuses the whole call if
any selected thread is published. The result says which went: `deletedIds`, `trashedIds`. Use trash for
published threads, or when the user may want the threads back. `delete_conversation_folder` deletes the
folder's subfolders too; its preview names them. A new folder comes from `create_conversation_folder` (`name`, optional `parentId`).

Each bulk call returns a preview listing the ids and the action. Show the count and the folder or status,
get the user's approval, then `confirm_pending_write` with `previewId` and `confirmed` true. A result whose
`pendingAction.status` is DRY_RUN changed nothing.

## Answer shape

Counts first (new, in progress, done), then the threads that need a reply with title, started date and who
wrote last, then the suggested action per thread. Say when the list is a sample and when a thread was
summarised from truncated previews.

`references/tool-notes.md` lists the arguments and result fields of every tool named here.
