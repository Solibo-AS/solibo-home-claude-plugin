---
name: meeting-agenda
description: "Board meetings (styremøter, årsmøter) in a housing community in Solibo Home: find the next meeting, prepare a briefing or packet, add cases to the agenda, reorder or follow up items, create a meeting, update participants, send summons, record attendance and signing. Use when the user mentions a meeting, agenda, saksliste, innkalling, protocol, or wants to put a case (sak) on a meeting."
---

# Meeting agenda

## Find and prepare

1. Resolve the company; `set_working_company` for a session about one company.
2. `search_meetings` with `companyId`; results come newest planned start first, so the nearest meeting is
   last. For "the next meeting" pass `meetingDateFrom` today and `meetingDateTo` a few months ahead, follow
   `paging.next` if present, and pick the row with the earliest future `plannedStartAt`. `status` filters
   take raw values (PLANNING, NOTIFIED, IN_PROGRESS, TO_SIGN, SIGNED). The user often knows the date, not
   the id.
3. `get_meeting_prep` with `meetingId` is the briefing: `meeting`, `agendaIssues[]` (issue id, title,
   status, responsible, description, and what this meeting holds for it: `report`, `meetingResolution`,
   `treated`, `followUp`; `lastResolution` only once the meeting is closed), `recentConversations[]`,
   `boardSummary`, `answerGuidance`. Drill into
   `get_issue` for proposed resolutions and comments, `search_documents` for board papers (type MEETING or
   ANNUAL_MEETING, or the subject words), and `get_conversation` for a referenced thread.
4. `get_meeting` only when participant details, RSVP, attendance or the agenda order (`issues[].sequence`)
   are needed; contact fields only with `includeFields`.

A briefing is read-only: it changes nothing and notifies nobody. Say so, and say which parts are samples.

## Put a case on the agenda

An agenda item is an issue (sak) attached to a meeting. A new one is a single preview and confirmation:

1. `create_issue` with `title`, `description`, `responsibleId` (a person id from `search_persons`,
   `lookup_person` or the `personId` of `get_current_user`) and `meetingId` in the same call; `beforeIndex`
   (zero-based) places it, omitting it appends. Do not create a bare title and update it afterwards. Confirm
   with `confirm_pending_write`; the result holds the issue `id` and `agendaOutcome`. FAILED means the issue
   exists but is not on the agenda: follow `agendaNote`, which says whether the refusal is final (no access,
   or Solibo Home rejected the change) or whether `add_issue_to_meeting` with that `issueId` is the next step;
   never create the issue again. UNKNOWN means Solibo Home did not answer the agenda step: read the agenda with `get_meeting`
   first, and attach only if the issue is missing, because attaching twice duplicates the item.
2. Existing cases come from `search_issues` (`status` ["OPEN"]) and go on the agenda with
   `add_issue_to_meeting` (`meetingId`, `issueId`, optional `beforeIndex`).

Several cases for one meeting: one preview and one confirmation per case, in order, and tell the user how
many confirmations to expect. For the same case in many companies use `create_issues_for_companies` with
`companyIds` and `titleTemplate`; it does not attach to meetings.

Later changes: `update_issue` (title, description, `proposedResolution`), `update_issue_responsible`,
`create_issue_comment`, `add_issue_conversation` and `add_issue_section` to link a thread or a section,
`reorder_meeting_issue` (`beforeIndex`), `remove_issue_from_meeting` (the case survives), `close_issue`
and `reopen_issue`.

`update_issue_report` (protocol text) and `update_issue_conclusion` (`resolution`) only work while the
case is on the agenda of a meeting that is not closed: Solibo Home stores both on that meeting's agenda
row. A closed case, or one on no agenda, is refused with the reason. Read them back with `get_issue`
(`currentMeeting.report`, `currentMeeting.resolution`) or `get_meeting` (`issues[]`); `lastResolution`
is filled only when the meeting has been closed.

## Create or change a meeting

- `create_meeting` with `title`, `plannedStartAt` and `plannedEndAt` as full timestamps with offset
  ("2026-11-12T18:00:00+01:00"), `location`. It sends no summons. For many companies,
  `create_meetings_for_companies` with `companyIds`, `titleTemplate` and the two times.
- `update_meeting` changes times, `location` or `comment`; `sendUpdatedNotification` true emails
  participants and needs the user's explicit wish.
- `update_meeting_participants` replaces the bucket you pass (`participantIds`, `guestParticipantIds`,
  `externalParticipants`); omitted buckets are kept, an empty list clears one. Added or removed people
  may be emailed if the meeting was already notified.
- `send_meeting_summons` (optional `comment`) emails every participant of the meeting: board members,
  guests and external participants alike. Its preview names the recipients and their number; show that
  list so the user approves the whole audience. It is a communication preview confirmed with
  `confirm_pending_communication`, only after the user approves the meeting, the audience and the text.
  A closed meeting or one without participants is refused.

## During and after the meeting

`get_meeting` gives each participant two ids. `participantId` (the same value as `id`) is the row in
this meeting; `personId` is the person. `mark_meeting_attendance` and `rsvp_meeting` (`participantId`)
and `mark_meeting_participant_signed` (`attendeeId`) take the participant id; a participant's person id
there is translated to it, and any other id is refused before the preview with the valid ids.
`send_meeting_signature_reminder` `idsToSign` takes person ids and translates a participant id to the
person; `update_meeting_participants` takes person ids.

`start_meeting`, `mark_meeting_attendance` (`participantId`, `attended`), `mark_meeting_issue_treated`
(one-way), `set_meeting_issue_follow_up` (`shouldFollowUp`); `get_meeting` shows both flags on
`issues[]` as `treated` and `followUp`. `close_meeting` needs at least two participants marked as
attended; the meeting then has status TO_SIGN (closed, awaiting signatures; there is no CLOSED status)
and Solibo Home emails a signature request to the attending board members. It becomes SIGNED when they
have all signed or been marked signature-unavailable.

`reopen_meeting` only applies to a closed meeting and discards the signing: all signatures and
signature-unavailable marks are cleared and the protocol document is deleted. `delete_meeting` on a
closed meeting destroys the same. Both previews list exactly what would be lost; show that to the user
and get an explicit yes before confirming.

Signing: `send_meeting_signature_reminder` (optional `idsToSign` with person ids, communication
confirm; the preview names who is reminded and counts only those `get_meeting` shows with a board role,
since Solibo Home emails current board members only; a participant marked signature-unavailable is left
out), `mark_meeting_participant_signed` (`attendeeId` is the
participant id from `get_meeting`), `finalize_meeting_signing` (forces signed; Solibo superuser only).
`rsvp_meeting` records YES, NO or MAYBE for a participant (`participantId` from `get_meeting`).
Follow-up to-dos that are not cases go in `create_task` (`title`, `dueDate` as a timestamp,
`responsibleId`).

## Guardrails

- Every write is a preview followed by `confirm_pending_write` (or `confirm_pending_communication` for
  summons, reminders and RSVP) with only `previewId` and `confirmed` true.
- Ask for the missing meeting, time, case text or responsible person before the preview; never invent them.
- Report the item as added only after an executed confirmation; a DRY_RUN result changed nothing.

`references/tool-notes.md` lists the arguments and result fields of every tool named here.
