---
name: document-filing
description: "Documents and files of a housing community in Solibo Home: find a document, open its review page or a temporary download link, read a PDF's text on request, download a folder as a ZIP, upload one or many files into the right folder, rename, move, trash, restore or delete documents, and create folders. Use when the user mentions documents, files, PDFs, reports, certificates, bylaws, house rules, folders, uploads, downloads or ZIPs (dokumenter, vedlegg, attest, nedlasting)."
---

# Document filing

## Find

1. Resolve the company; `set_working_company` for a session about one company.
2. `search_documents` with `search` (words, "quoted phrases", or, -word; matches names, folders and the
   text of documents), `type` for the filing area, `createdAtFrom` and `createdAtTo`, `count` (default
   100). Results carry `displayName` (use it as the label), `fileName`, `type`, `isDirectory`, `parentId`,
   `sourceDocumentId` (the id to reuse for attachments) and `searchMatch.matchedIn`. The document type is
   the filing area (BYLAWS, HOUSE_RULES, MEETING, ANNUAL_MEETING, INSURANCE, LOAN, FDV, ECONOMY, ...);
   content words such as "minutes" belong in `search`, not in `type`. `list_document_types` lists the
   areas.
3. `browse_document_folder` walks one area: `documentType`, then `parentId` for a subfolder (keep the same
   `parentId` while paging). Use it when the user knows where a file lives, not to crawl every folder.
4. Several matches: show them with type, folder and date and let the user choose. Creator names appear
   only through `reveal_document_creators`.

## Open, download, read

- `get_document_metadata` returns the authenticated review link (`markdownLink`) and no content.
- `get_document_link` returns a temporary signed `downloadUrl` and a `markdownLink`; present the markdown
  link directly. Call again if it expired. It is a link, not an attachment, so never claim a file is
  attached in chat.
- `get_document_archive` returns one ZIP for the company, a `documentType` or a `directoryId` (link
  expires after about ten minutes; exports above 50 MiB fail without a partial file). Use it for "all
  documents in", never a loop of single downloads.
- `reveal_document_content` extracts the text of a PDF or text file (`maxTextChars`, default 120,000) and
  is logged as content access. A scoped request such as "summarise this report" is the permission; a
  download request is not. Say when `contentTextTruncated` is true and distinguish what the document states
  from your suggestions.

## Upload

- A file exists for an upload only when the conversation holds an attached file reference (a file id with its
  download URL) or the user gave a public HTTPS link. When the user says "this file" and neither is there,
  ask for the file, the document type and the folder before any preview; never invent a file reference.
- `upload_document` with `companyId`, `documentType`, and `file` (a ChatGPT-attached file, or any public
  HTTPS link with a file_id you choose and the download_url); `fileName` only for a rename the user asked
  for; `parentId` for the folder; `belongsToId` when the file belongs to a case, meeting or section. The
  preview stages the bytes; after approval, `confirm_pending_write` with `previewId` and `confirmed` true
  saves it. Never resend the file, its bytes or its name on the confirmation.
- Several files: one preview and one confirmation per file, in order; say how many confirmations to expect.
- Files meant for a post, newsletter, conversation reply or case go on that tool's `attachments`,
  `inlineDocuments` or `files`; a separate upload is not needed and a CONVERSATION upload is visible to
  every participant, residents included.

## Organise

- `create_document_folder` (`documentType`, `folderName`, `parentId`), `rename_document` (`fileName` is the
  new name), `move_document` (`parentId` is the target folder).
- `trash_document` is reversible with `restore_document`; `delete_document` only when the user explicitly
  asks for deletion.
- Many documents at once: `batch_update_documents` with `operation` MOVE, TRASH or RESTORE and
  `documentIds` (up to 100; root folder ids only), one preview and one confirmation.

## Guardrails

- Downloading grants no permission to extract text; reading text grants no permission to send it on.
- A failed fetch of a link by the assistant does not invalidate the link; present it again.
- Report a file as saved, moved or deleted only from an executed confirmation.

`references/tool-notes.md` lists the arguments and result fields of every tool named here.
