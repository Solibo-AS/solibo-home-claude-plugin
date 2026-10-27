# Tool notes for conversation triage

Ids are strings unless noted. `conversationIds` accepts an array or a comma-separated string (max 100).
Confirmation: `confirm_pending_write` or `confirm_pending_communication` with only `previewId` and
`confirmed` true; never pass `confirmed` to the target tool.

| Tool | Kind | Required | Optional | Result fields to use |
|---|---|---|---|---|
| `get_conversation_meta` | read | `companyId` | `foldersParentId`, `count`, `categoriesPageToken`, `foldersPageToken` | `counts.{new,inProgress,done,draft,total}`, `categories[].{id,name,color}`, `folders[].{id,name,parentId,unreadCount}` |
| `search_conversations` | read | none | `companyId`, `companyIds`, `search`, `status` (one of NEW, IN_PROGRESS, COMPLETE, DONE, DRAFT, TRASHED; one value per call), `categoryIds`, `byParticipantIds`, `folderId`, `onlyRoot`, `assignedToUserId`, `createdSince`, `count` (25), `pageToken`, `includeFields` | `items[].{id,companyId,title,status,conversationStartedAt,type,readByCurrentUser,participants[]}`, `paging.next` |
| `get_my_conversations` | read | none | `unread`, `count` (10 company groups), `pageToken` | `readState`, `groups[].{companyId,companyName,conversations[]}`, `paging.next` |
| `get_conversation` | read | `companyId`, `conversationId` | `messageLimit` (20, max 1000), `includeFields` | `title`, `status`, `folderId`, `categoryId`, `flaggedByCurrentUser`, `relatedIssueIds`, `latestActivityAt`, `participants`, `messages[].{id,content,sentAt,sentBy,type,documents,inlineDocuments}`, `messageListTruncated`, `messageTextTruncated` |
| `reveal_conversation_full_content` | audited read | as `get_conversation` | as `get_conversation` | same, full text |
| `reply_to_conversation` | preview, communication confirm | `companyId`, `conversationId`, `content` | `attachments`, `inlineDocuments`, `files` | `messageId`, `sentAt`, `documentIds` |
| `reply_to_public_conversation` | preview, communication confirm | `companySlug`, `conversationId`, `content` | `attachmentIds` | `action` |
| `create_internal_note` | preview, write confirm | `companyId`, `conversationId`, `content` | `documentIds` (existing, max 10) | `messageId`, `sentAt` |
| `create_conversation` | preview, communication confirm | `companyId`, `title`, `participants[].{participantId (int), participantType PERSON or ORGANIZATION}` | `content`, `attachments`, `inlineDocuments`, `files` | `conversationId`, `status`, `participantCount` |
| `create_conversation_inquiry` | preview, communication confirm | `companyId`, `categoryId`, `content` | `title`, `name`, `email`, `mobile` | `conversationId`, `status` |
| `publish_conversation` | preview, communication confirm | `companyId`, `conversationId` | none | `action` published |
| `update_conversation_details` | preview, write confirm | `companyId`, `conversationId` | `title`, `categoryId`, `content` (DRAFT only), `attachments`, `inlineDocuments`, `removeDocumentIds`, `files` | `title`, `status`, `categoryId` |
| `update_conversation` | preview, write confirm | `companyId`, `conversationId`, `status` DONE or IN_PROGRESS | none | `status` |
| `mark_conversation_read` | preview, write confirm | `companyId`, `conversationId` | none | `readByCurrentUser` |
| `mark_conversations_read` | preview, write confirm | `companyId`, `conversationIds`, `read` | none | `count`, `action` |
| `flag_conversations` | preview, write confirm | `companyId`, `conversationIds`, `flagged` | none | `count`, `action` |
| `close_conversations`, `reopen_conversations`, `trash_conversations`, `restore_conversations`, `delete_conversations` | preview, write confirm | `companyId`, `conversationIds` | none | `count`, `action` |
| `move_conversations_to_folder` | preview, write confirm | `companyId`, `conversationIds`, and `folderId` or `topLevel` true | none | `count`, `folderId` |
| `move_conversation_to_folder` | preview, write confirm | `companyId`, `conversationId`, and `folderId` or `topLevel` true | none | `folderId` |
| `add_conversation_participant`, `remove_conversation_participant` | preview, write confirm | `companyId`, `conversationId`, `participantType`, `participantId` | none | `action` |
| `create_conversation_folder` | preview, write confirm | `companyId`, `name` | `parentId` | `folderId` |
| `list_conversation_categories` | read | `companyId` | `count`, `pageToken` | `items[].{id,name,color,responsibleId,contactCategory,inUse}` |
| `update_conversation_categories` | preview, write confirm | `companyId`, `categories[].{id, responsibleId?, coResponsibleIds?}` | none | `verification`, `categories[]` |

Attachments on replies and new threads: `files[]` items hold file_id and download_url (a ChatGPT file or
a public HTTPS link); `attachments[]` items hold either `sourceDocumentId` (from `search_documents`) or
`fileId`; `inlineDocuments[]` items hold `slot`, `kind` image or link, a source, and `alt` or `label`,
with a `{{inline:slot}}` placeholder in `content`. Never resend files, bytes or text on the confirmation.
