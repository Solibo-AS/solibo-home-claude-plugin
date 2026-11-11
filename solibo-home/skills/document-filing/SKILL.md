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

- New uploads require a downloadable HTTPS `file.download_url` on a public host and a caller-chosen
  `file.file_id`, with optional `file.file_name` and `file.mime_type`. Signed links and links from other connectors
  work when they download without extra authorization headers. A client attachment works only when
  the client provides that URL; a Claude local attachment ID alone does not. Never invent a URL,
  send base64 or ask the user to publish a private file.
- If the client cannot transfer the file, call `get_document_upload_options` with `companyId` and
  optionally `documentType` and `belongsToId`. Present its authenticated Home link and instructions.
  The link opens the company documents/expenses area, not an upload modal. The user signs in, selects
  the intended category or existing record and uploads manually. Then discover its `sourceDocumentId`
  with `search_documents` or `browse_document_folder` and reuse it.
- `upload_document` takes `companyId`, `documentType`, and exactly one `file` reference or existing
  `sourceDocumentId`. Set `belongsToId` when the file belongs to an issue, meeting, expense or section.
  For a new file, `parentId` selects the folder and `fileName` is only for a requested rename or missing
  filename. Existing-source reuse requires a positive `belongsToId` for an existing record, preserves
  the source filename, accepts neither `parentId` nor `fileName`, and downloads no bytes. It cannot
  copy a file into a generic folder or category root.
  Show the preview, then after approval call `confirm_pending_write` with only `previewId` and
  `confirmed` true. Never resend file inputs on confirmation.
- Several standalone files: one preview and one confirmation per file, in order; say how many to expect.
- Use integrated `files` and `attachments` on homepage posts, practical information, email newsletters,
  conversation creation/replies or draft conversation updates, and expense creation. Inline documents
  are supported on those communication tools, but expenses take attachments only. For cases/issues,
  meetings and other tools without integrated inputs, use `upload_document` with the correct category
  and owner after that record exists. A CONVERSATION upload is visible to every participant, residents
  included.
- New-file limit: configured per-file maximum, default 20 MB. Integrated tools allow ten files,
  fifty document sources and 50 MB transferred in total, counting repeated uses of the same file.
- If expense receipt attachment fails after the claim was created, never recreate the claim. Inspect
  documents with `browse_document_folder`, `documentType` EXPENSE and `belongsToId` the existing expense
  ID before retrying an unknown outcome; attach only missing receipts to that expense.

## Organise

- `create_document_folder` (`documentType`, `folderName`, `parentId`), `rename_document` (`fileName` is the
  new name), `move_document` (`parentId` is the target folder).
- `trash_document` is reversible with `restore_document`; `delete_document` only when the user explicitly
  asks for deletion. Only archive documents of type OTHER or BUILDING can be trashed: a document that
  belongs to a record (a loan or insurance agreement, an expense receipt, a meeting attachment) is refused
  whatever the user's role, so tell the user that instead of blaming their access.
- Many documents at once: `batch_update_documents` with `operation` MOVE, TRASH or RESTORE and
  `documentIds` (up to 100; root folder ids only), one preview and one confirmation.

## Guardrails

- Downloading grants no permission to extract text; reading text grants no permission to send it on.
- A failed fetch of a link by the assistant does not invalidate the link; present it again.
- Report a file as saved, moved or deleted only from an executed confirmation.

`references/tool-notes.md` lists the arguments and result fields of every tool named here.
