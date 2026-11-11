# Tool notes for conversation triage

Ids are strings unless noted. `conversationIds` accepts an array or a comma-separated string (max 100).
Confirmation: `confirm_pending_write` or `confirm_pending_communication` with only `previewId` and
`confirmed` true; never pass `confirmed` to the target tool.

| Tool | Kind | Required | Optional | Result fields to use |
|---|---|---|---|---|
| `get_conversation_meta` | read | `companyId` | `foldersParentId`, `count`, `categoriesPageToken`, `foldersPageToken` | `counts.{new,inProgress,done,draft,total}`, `categories[].{id,name,color}`, `folders[].{id,name,parentId,unreadCount}`, `notice` (names a part that could not be read; the call fails when none could) |
| `search_conversations` | read | none | `companyId`, `companyIds`, `search`, `status` (one of NEW, IN_PROGRESS, COMPLETE, DONE, DRAFT, TRASHED; one value per call), `categoryIds`, `byParticipantIds` (person ids unless `byParticipantType` says ORGANIZATION, EMPLOYEE, SUPPLIER_CONTACT or INQUIRY), `folderId`, `folderIds`, `onlyRoot`, `assignedToUserId`, `assignedToUserIds`, `createdSince`, `updatedSince` (activity at or after), `readByCurrentUser`, `flaggedByCurrentUser`, `hasAttachments`, `latestPublicMessageSide` (BOARD, NON_BOARD, NONE), `orderBy` (LATEST_ACTIVITY_AT default, CONVERSATION_STARTED_AT, LATEST_MESSAGE_AT, LATEST_EVENT_AT), `orderDirection`, `count` (25), `pageToken`, `includeFields` | `items[].{id,companyId,title,status,conversationStartedAt,latestActivityAt,type,readByCurrentUser,flaggedByCurrentUser,folderId,category,relatedIssues,participants[].{personId,organizationId,participantType,active,participantRowId,name}}`, `paging.next` |
| `get_my_conversations` | read | none | `unread`, `search`, `orderBy`, `orderDirection`, `includeTrashed` (false), `count` (10 company groups), `pageToken` | `readState`, `groups[].{companyId,companyName,conversations[].{id,title,status,latestActivityAt,readByCurrentUser,flaggedByCurrentUser}}`, `paging.next` |
| `get_conversation` | read | `companyId`, `conversationId` | `messageLimit` (20, max 1000), `translateToCountryId`, `includeFields` | `title`, `status`, `folderId`, `categoryId`, `flaggedByCurrentUser`, `relatedIssueIds`, `latestActivityAt`, `firstMessageAt`, `participants[].{personId,organizationId,participantType,active,participantRowId,name}`, `messages[].{id,content,sentAt,sentBy,type,documents,inlineDocuments}`, `internalNotes[]` (same shape, staff and board only), `messageListTruncated`, `messageTextTruncated` |
| `reveal_conversation_full_content` | audited read | as `get_conversation` | as `get_conversation` | same, full text |
| `reply_to_conversation` | preview, communication confirm | `companyId`, `conversationId`, `content` | `attachments`, `inlineDocuments`, `files` | `messageId`, `sentAt`, `documentIds` |
| `create_internal_note` | preview, write confirm | `companyId`, `conversationId`, `content` | `documentIds` (existing, max 10) | `messageId`, `sentAt` |
| `create_conversation` | preview, communication confirm | `companyId`, `title`, `participants[].{participantId (int), participantType PERSON or ORGANIZATION}` | `categoryId` (recommended), `folderId`, `draft`, `content`, `attachments`, `inlineDocuments`, `files` | `conversationId`, `status`, `participantCount`, `categoryId`, `folderId` |
| `create_conversation_inquiry` | preview, communication confirm | `companyId`, `categoryId` (a contact category; others are refused at the preview), `content` | `title`, `name`, `email`, `mobile` | `conversationId`, `status` |
| `publish_conversation` | preview, communication confirm | `companyId`, `conversationId` (a DRAFT with a category) | none | `action` published |
| `update_conversation_details` | preview, write confirm | `companyId`, `conversationId` | `title`, `categoryId`, `content` (DRAFT only), `attachments`, `inlineDocuments`, `removeDocumentIds`, `files` | `title`, `status`, `categoryId` |
| `update_conversation` | preview, write confirm | `companyId`, `conversationId`, `status` DONE or IN_PROGRESS | none | `status` |
| `mark_conversation_read` | preview, write confirm | `companyId`, `conversationId` | none | `readByCurrentUser` |
| `mark_conversations_read` | preview, write confirm | `companyId`, `conversationIds`, `read` | none | `count`, `action` |
| `flag_conversations` | preview, write confirm | `companyId`, `conversationIds`, `flagged` | none | `count`, `action` |
| `close_conversations`, `reopen_conversations`, `trash_conversations`, `restore_conversations`, `delete_conversations` | preview, write confirm | `companyId`, `conversationIds` | none | `count`, `action`. `restore_conversations` refuses, before the preview, ids that are not in the trash or have no category, and names the ids that can be restored |
| `move_conversations_to_folder` | preview, write confirm | `companyId`, `conversationIds`, and `folderId` or `topLevel` true | none | `count`, `folderId` |
| `move_conversation_to_folder` | preview, write confirm | `companyId`, `conversationId`, and `folderId` or `topLevel` true | none | `folderId` |
| `add_conversation_participant`, `remove_conversation_participant` | preview, write confirm | `companyId`, `conversationId`, `participantType`, `participantId` (the participant's `personId` or `organizationId`, never `id` or `participantRowId`) | none | `action`, `personId`, `organizationId`; remove also `verification` (CONFIRMED, MISMATCH, UNVERIFIED) |
| `create_conversation_folder` | preview, write confirm | `companyId`, `name` | `parentId` | `folderId` |
| `list_conversation_categories` | read | `companyId` | `count`, `pageToken` | `items[].{id,name,color,responsibleId,contactCategory,inUse}` |
| `update_conversation_categories` | preview, write confirm | `companyId`, `categories[].{id, responsibleId?, coResponsibleIds?}` | none | `verification`, `categories[]`; a confirm is refused when a kept responsible changed after the preview: preview again |

Attachments on replies and new threads: `files[]` items hold a caller-chosen file_id and downloadable
HTTPS download_url on a public host (signed links accepted without extra authorization headers); `attachments[]` items hold either `sourceDocumentId` (from `search_documents`) or
`fileId`; `inlineDocuments[]` items hold `slot`, `kind` image or link, a source, and `alt` or `label`,
with a `{{inline:slot}}` placeholder in `content`. Never resend files, bytes or text on the confirmation.

`get_document_upload_options` (read-only): required `companyId`, optional `documentType` and
`belongsToId`; returns `homeUrl`, `uploadInstruction`, `reuseInstruction`, `fileTransferGuidance`.
The link opens a Home company area, not an upload modal. Upload manually and then discover the source ID.
