# Tool notes for documents

Document types: ADHOC, ANNUAL_MEETING, BUILDING, BYLAWS, COMMUNICATION_COMMENT, CONVERSATION,
ECONOMIC_REPORT, ECONOMY, EXPENSE, FDV, HOMEPAGE, HOUSE_RULES, INSURANCE, INTERNAL_MESSAGE, INVOICE, ISSUE,
LOAN, MEETING, MESSAGE, NEIGHBORHOOD_GROUP, NEIGHBORHOOD_POST, NEWSLETTER, OTHER, POST, PRACTICAL_INFO,
PRIVATE, PROJECT, ROUTINE, SETTLEMENT, SETTLEMENT_CUSTOM_COST, SUPPLIER_FOR_COMPANY, TRASH.

| Tool | Kind | Required | Optional | Result fields to use |
|---|---|---|---|---|
| `search_documents` | read | none | `companyIds`, `search`, `type[]`, `createdAtFrom`, `createdAtTo`, `count` (100), `pageToken` | `items[].{id,sourceDocumentId,companyId,displayName,fileName,type,isDirectory,parentId,belongsToId,createdAt,childCount,searchMatch.{matchedIn,snippets},directLinkTool}`, `paging.next` |
| `browse_document_folder` | read | `companyId`, `documentType` | `parentId`, `belongsToId`, `isDirectory`, `count`, `pageToken` | same item fields |
| `list_document_types` | read | none | none | `items[].type` |
| `get_document_metadata` | read | `companyId`, `documentId` | `fileName`, `displayName` | `url`, `markdownLink`, `mediaType`, `textExtractionStatus` |
| `get_document_link` | read | `companyId`, `documentId` | `fileName`, `displayName` | `downloadUrl` (temporary), `url` (review page), `markdownLink`, `linkLabel` |
| `get_document_archive` | read | `companyId` | `documentType`, `directoryId` (needs `documentType`) | `downloadUrl`, `markdownLink`, `fileName`, `byteLength`, `expiresAt` |
| `reveal_document_content` | audited read | `companyId`, `documentId` | `fileName`, `displayName`, `maxBytes`, `maxTextChars` (120,000) | `contentText`, `contentTextChars`, `contentTextTruncated`, `textExtractionStatus` |
| `reveal_document_creators` | audited read | as `search_documents` | `count` (20) | items plus `createdByName` |
| `upload_document` | preview, write confirm | `companyId`, `documentType`, `file` (file_id, download_url, optional file_name and mime_type) | `fileName`, `belongsToId`, `parentId` | after confirmation `documentId`, `fileName`, `displayName`, `parentId`, `readTool` |
| `rename_document` | preview, write confirm | `companyId`, `documentId`, `fileName` | `currentFileName` | `fileName`, `displayName` |
| `move_document` | preview, write confirm | `companyId`, `documentId`, `parentId` | `fileName` | `parentId` |
| `create_document_folder` | preview, write confirm | `companyId`, `documentType`, `folderName` (max 255, no slashes) | `parentId` | `documentId`, `isDirectory` |
| `trash_document`, `restore_document` | preview, write confirm | `companyId`, `documentId` | `fileName`; restore also `documentType`, `parentId` | `success` |
| `delete_document` | preview, write confirm, destructive | `companyId`, `documentId` | `fileName`, `displayName` | `deleted` |
| `batch_update_documents` | preview, write confirm | `companyId`, `operation` MOVE, TRASH or RESTORE, `documentIds` (max 100) | `parentId` (MOVE), `documentType` (RESTORE) | `count`, `success` |
| `confirm_pending_write` | confirm | `previewId`, `confirmed` true | none | `pendingAction.{status,executed,dryRun}`, `result` |

File links must be HTTPS with a DNS hostname (no IP literals), at most 20 MB per file. Reuse
`sourceDocumentId` from search results when attaching an existing document elsewhere.
