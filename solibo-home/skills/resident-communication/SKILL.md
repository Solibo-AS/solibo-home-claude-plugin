---
name: resident-communication
description: "Send or prepare communication to the residents or board of a housing community in Solibo Home: email newsletters, SMS, feed posts, practical information, letters, and the same message to several communities. Covers choosing the audience (board, owners, tenants, custom notification groups built from section tags), previewing, saving a draft, sending now or scheduling, attachments, and delivery reports. Use when the user wants to inform, notify, write to, or publish something for residents, the board, or everyone in a company (beboere, styret, nyhetsbrev, SMS, oppslag)."
---

# Resident communication

Every message goes preview, then approval, then one confirmation. Preview-only requests end at the
preview. Saving a draft, sending now and scheduling are different actions; when the user has not said which,
ask before the preview.

## Resolve company and audience

1. Resolve the company (`search_companies`); with one company in play, `set_working_company`.
2. Audiences are `recipientGroups`: ALL, BOARD_CHAIRS, BOARD_MEMBERS, BOARD_ALTERNATES, OWNERS, TENANTS,
   CO_HABITANTS, EXTERNALS. "The board" (styret) is BOARD_CHAIRS, BOARD_MEMBERS and BOARD_ALTERNATES unless
   the user narrows it. `get_communication_notification_groups` with `channel` EMAIL or SMS lists the
   groups and the section tags available as custom groups; `get_sms_readiness` says whether enough SMS
   credits exist.
3. Custom groups (one per street, building or entrance) are section tags. To build one: `search_sections`
   with `companyId` and the address words, then tag them all with one `add_section_tags` call (`sectionIds`
   from the search, up to 300, and `tagNames`), one preview and one confirmation; `sectionId` still works
   for a single section, and a new name creates the tag. Check `failedCount` and `unknownCount` in the result:
   name any section that was not tagged, and for an UNKNOWN row read `get_section_tags_by_section` before
   retrying, because the tag may already be set. `get_company_section_tags` lists the
   company's tags, `get_section_tags_by_section` shows which sections carry them, `remove_section_tag`
   removes one. To send to a group, pass the tag name in `recipientTags`. A tag reaches the current owners and
   section contact persons of the tagged sections plus the company's contact persons, resolved at send
   time, never tenants or co-habitants; say all three groups in the approval summary. To include tenants or
   co-habitants of the tagged
   sections, pass `recipientTagResidents` ["TENANTS"] or ["CO_HABITANTS"] and say that they are fixed
   recipients that do not follow later tenancy changes. The TENANTS and CO_HABITANTS groups reach the whole
   company.
4. People outside Solibo Home go in `directRecipients` (addresses or numbers the user supplied); registered
   people in `directRecipientPersonIds` (integers from `search_persons`), organisations in
   `directRecipientOrganizationIds`. Unknown ids are dropped silently and reported after confirmation in
   `droppedDirectRecipientPersonIds` and `droppedDirectRecipientOrganizationIds`; tell the user about every
   dropped id in both fields, since those recipients will not receive the message.

## Email or SMS to one company

- `draft_customer_message` with `messageType` email or sms, `subject` (email), `content`, the audience
  fields, and for email `htmlText`, `attachments`, `inlineDocuments` and `files`. The preview saves
  nothing. After approval, `confirm_pending_communication` saves a draft and returns `newsletterId` or
  `broadcastId`.
- To schedule, set `scheduledAt` to a full timestamp with an offset ("2026-11-04T09:00:00+01:00") on the
  draft; the saved draft has status scheduled and nothing more is needed. A past value is treated as
  "send now" intent, so ask instead of guessing.
- To send a saved draft now: `send_customer_message` with `messageType` and `messageId` (the id from the
  save result), preview, then `confirm_pending_communication`. Irreversible; never reuse the save
  `previewId` to send.
- To change a draft: `update_email_newsletter_draft` or `update_sms_broadcast_draft` (omitted fields are
  kept; `scheduledAt` or `clearSchedule`, not both). `get_email_newsletter`, `get_sms_broadcast` and
  `search_newsletters_and_sms_broadcasts` (`type` EMAIL or SMS, `published`) find earlier messages.
- Delivery: `get_communication_report` for counts, `get_communication_delivery_report` with `filter`
  PROBLEM for bounces and failures. A sent status does not mean every resident has read the message.

## The same email to several companies

`prepare_batch_newsletter` with `companyIds` (1 to 100), `subjectTemplate` and `contentTemplate` (placeholders
such as {{company.name}}) renders every company's message without saving; when `readyForDrafting` is true,
its `previewId` is the frozen company set. After approval, `draft_batch_newsletter` with that `previewId`
(and `scheduledAt` for a future send) creates the drafts after `confirm_pending_communication`; its result
holds `batchDraftId`. `publish_batch_newsletter` with the `batchDraftId` sends them all after a second
confirmation. Never rebuild the company list after a complete preview, and never loop
`send_customer_message` over the drafts.

## Feed posts and practical information

- `create_feed_post` (`title`, `content`; `notifyEmailGroups`, `notifySmsGroups`; `pinned`; `image`,
  `attachments`, `inlineDocuments`, `files`) and `create_practical_info` for one company; both confirm with
  `confirm_pending_communication`. A feed post without `publishedAt` publishes on confirmation; a future
  `publishedAt` schedules it; `draft` true saves it unpublished. Practical information publishes on
  confirmation unless `draft` is true. `validate_homepage_sms` checks SMS credits for the chosen groups.
- `get_homepage` and `list_homepage_posts` (`postKind` FEED or PRACTICAL_INFORMATION) give `postId` and
  `practicalInfoId` for `update_feed_post` and `update_practical_info`; editing a published post notifies
  nobody unless `notifyAgain` is true.
- For several companies, `create_feed_posts_for_companies` or `create_practical_info_for_companies` with
  `companyIds`, `titleTemplate` and `contentTemplate`, one preview and one `confirm_pending_write`. Do not
  loop the single-company tools.

## Letters and attachments

- `send_company_letter` posts a physical letter (`fullName`, `addressLineOne`, `postalArea`, `city`,
  `content`; `personId` for a registered person); real delivery, communication confirm. Never include a
  national identity number.
- Attach files on the message tool itself: `files[]` items carry file_id and download_url (a ChatGPT
  attachment or a public HTTPS link), `attachments[]` items refer to a `fileId` or a `sourceDocumentId`
  from `search_documents`, `inlineDocuments[]` items name a `slot` placed as {{inline:slot}} in `htmlText`
  or `content`. SMS takes no documents; newsletters take no `image`. A separate `upload_document` is not
  needed to attach a file. Never resend files, bytes or text on the confirmation hop.

## Guardrails

- Do not put personal email addresses, phone numbers, national identity numbers or registered addresses
  into message text unless the user wrote them.
- Ask for the company, audience, content and timing that are missing before the preview; do not invent
  dates, times or sender names.
- Report "saved" or "sent" only from an executed confirmation; a dry run or a denial changed nothing.

`references/tool-notes.md` lists the arguments and result fields of every tool named here.
