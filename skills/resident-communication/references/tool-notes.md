# Tool notes for resident communication

Timestamps need a full ISO 8601 value with an offset, for example 2026-11-04T09:00:00+01:00. Ids are
strings except `directRecipientPersonIds`, `directRecipientOrganizationIds` and `removeDocumentIds`
(integers).

| Tool | Kind | Required | Optional | Result fields to use |
|---|---|---|---|---|
| `get_communication_notification_groups` | read | `companyId`, `channel` EMAIL or SMS | none | `notificationGroups[].groups[].{id,name}`, `tags[].key` (tag names for `recipientTags`) |
| `get_sms_readiness` | read | `companyId` | `broadcastId`, `includePurchases`, `count` | `availableCount`, `hasEnoughAvailable`, `requiredBundle` |
| `validate_homepage_sms` | read | `companyId` | `postId`, `groups[]` | `enoughSmsCredits`, `message` |
| `get_company_section_tags` | read | `companyId` | `count`, `pageToken` | `items[].{name,protected}` |
| `search_sections` | read | none | `companyId`, `companyIds`, `search`, `status` (needs `companyId`), `count` (50), `pageToken` | `items[].{id,companyId,identifier,apartmentNr,streetAddress,zipCode}` |
| `get_section_tags`, `get_section_tags_by_section` | read | `companyId` (+ `sectionId` for the first) | `count`, `pageToken` | `items[].{name}` or `sections[].{sectionId,tags[]}` |
| `add_section_tag` | preview, write confirm | `companyId`, `sectionId`, `tagName` | none | `tagNames[]` |
| `add_section_tags` | preview, write confirm | `companyId`, `tagNames` (array or comma list), and `sectionId` or `sectionIds` (up to 300) | none | `sectionIds[]`, `succeededCount`, `failedCount`, `unknownCount`, `sections[].{sectionId,outcome (APPLIED, FAILED, UNKNOWN),success,error}` |
| `remove_section_tag` | preview, write confirm | `companyId`, `sectionId`, `tagName` | none | `action` removed |
| `draft_customer_message` | preview, communication confirm | `messageType` email or sms, `content` | `companyId`, `subject`, `htmlText`, `recipientGroups`, `recipientTags`, `recipientTagResidents`, `directRecipients`, `directRecipientPersonIds`, `directRecipientOrganizationIds`, `attachments`, `inlineDocuments`, `files`, `scheduledAt` | after confirmation `newsletterId` or `broadcastId`, `status` draft or scheduled, `droppedDirectRecipientPersonIds`, `droppedDirectRecipientOrganizationIds`, `note` |
| `send_customer_message` | preview, communication confirm, irreversible | `messageType`, `messageId` | `companyId` | `publishedAt` |
| `update_email_newsletter_draft` | preview, communication confirm | `companyId`, `newsletterId` | `subject`, `content`, `htmlText`, recipient fields (each replaces; [] clears), `attachments`, `inlineDocuments`, `removeDocumentIds`, `files`, `scheduledAt`, `clearSchedule` | `messageId`, `publishedAt` |
| `update_sms_broadcast_draft` | preview, communication confirm | `companyId`, `broadcastId` | `content`, recipient fields, `scheduledAt`, `clearSchedule` | `messageId`, `publishedAt` |
| `get_email_newsletter` | read | `companyId`, `newsletterId` | none | `subject`, `plainText`, `publishedAt`, `notificationGroups`, `recipientStatusCounts`, `attachments[]` |
| `get_sms_broadcast` | read | `companyId`, `broadcastId` | none | `plainText`, `publishedAt`, `recipientStatusCounts` |
| `search_newsletters_and_sms_broadcasts` | read | none | `type` EMAIL or SMS, `companyId`, `companyIds`, `search`, `published`, `publishedAtFrom`, `publishedAtTo`, `count` (10), `pageToken` (needs `type`) | `items[].{id,type,subject,preview,publishedAt,status}` (EMAIL id is a `newsletterId`, SMS id a `broadcastId`) |
| `get_communication_report` | read | `companyId`, `communicationType` NEWSLETTER, BROADCAST, POST or PRACTICAL_INFO, `communicationId` | none | `recipientCount`, `emailStateCounts`, `smsStateCounts`, `pushStateCounts` |
| `get_communication_delivery_report` | read | same three | `filter` PROBLEM or ALL, `limit`, `includeFields` | `matchingRecipientCount`, `recipients[].{name,recipientType,emailChannel,smsChannel}` |
| `prepare_batch_newsletter` | private preview, no confirm | `companyIds` (1 to 100), `subjectTemplate`, `contentTemplate` | `recipientGroups`, `scopeLabel` | `previewId`, `readyForDrafting`, `companies[].{companyId,subject,contentPreview,warnings}`, `failures` |
| `draft_batch_newsletter` | preview, communication confirm | `previewId` | `recipientGroups`, `scheduledAt` (future) | `batchDraftId`, `drafts[].{companyId,newsletterId,status}` |
| `publish_batch_newsletter` | preview, communication confirm, irreversible | `batchDraftId` | `companyIds` | `published[].{companyId,newsletterId,status}` |
| `create_feed_post` | preview, communication confirm | `companyId`, `title`, `content` | `notifyEmailGroups`, `notifySmsGroups`, `private`, `pinned`, `image`, `attachments`, `inlineDocuments`, `files`, `publishedAt` or `draft` | `postId`, `createdAt`, `documentIds` |
| `update_feed_post` | preview, communication confirm | `companyId`, `postId` | content fields, `publishedAt` or `publishNow` or `draft`, `notifyAgain`, `pinned`, notify groups | `postId` |
| `create_practical_info` | preview, communication confirm | `companyId`, `title`, `content` | notify groups, `private`, `image`, `attachments`, `inlineDocuments`, `files`, `draft` | `postId` |
| `update_practical_info` | preview, communication confirm | `companyId`, `practicalInfoId` | as `update_feed_post` | `postId` |
| `create_feed_posts_for_companies`, `create_practical_info_for_companies` | preview, write confirm | `companyIds` (max 500), `titleTemplate`, `contentTemplate` | notify groups, `private`, `pinned`, `publishedAt`, `draft` | `items[].{companyId,entityId,status,warnings}`, `unresolvedPlaceholders` |
| `list_homepage_posts` | read | `companyId`, `postKind` FEED or PRACTICAL_INFORMATION | `search`, `count` (20), `pageToken` | `items[].{id,title,pinned,private,publishedAt}` |
| `get_homepage` | read | `companyId` | `feedCount` | `feed[]`, `practicalInformation[]` |
| `send_letter`, `send_company_letter` | preview, communication confirm | `fullName`, `addressLineOne`, `postalArea`, `city` (+ `companyId` for the second) | `subject`, `content`, `contentLines`, `orgnr`, `personId`, `forcePrintJob` | `letterId`, `messageId` |
| `confirm_pending_communication` | confirm | `previewId`, `confirmed` true | none | `pendingAction.{status,executed,dryRun}`, `result` |
