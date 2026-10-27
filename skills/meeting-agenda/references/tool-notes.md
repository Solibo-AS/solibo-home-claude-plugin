# Tool notes for meetings and agendas

Timestamps are ISO instants with an offset or Z. `responsibleId`, `participantIds` and `guestParticipantIds`
are person ids. Write tools return a preview; confirm with `confirm_pending_write`, or with
`confirm_pending_communication` for `send_meeting_summons`, `send_meeting_signature_reminder` and
`rsvp_meeting`.

| Tool | Required | Optional | Result fields to use |
|---|---|---|---|
| `search_meetings` | none | `companyId`, `companyIds`, `search`, `status` [PLANNING, NOTIFIED, IN_PROGRESS, TO_SIGN, SIGNED], `meetingDateFrom`, `meetingDateTo`, `count` (100), `pageToken` | `items[].{id,companyId,title,status,plannedStartAt,plannedEndAt,location,issueCount}` newest planned start first, `paging.next` |
| `get_meeting_prep` | `companyId`, `meetingId` | none | `meeting.{id,title,status,plannedStartAt,participantSummary}`, `agendaIssues[].{issueId,title,status,responsibleName,description}`, `recentConversations[]`, `boardSummary`, `answerGuidance` |
| `get_meeting` | `companyId`, `meetingId` | `includeFields` | `participants[].{id,personId,name,role,isGuest,rsvp,attended,signedAt}`, `externalParticipants[]`, `issues[].{id,title,status,sequence}`, `reportId` |
| `create_meeting` | `title`, `plannedStartAt`, `plannedEndAt` | `companyId`, `location` | `id`, `status` |
| `update_meeting` | `companyId`, `meetingId` | `title`, `plannedStartAt`, `plannedEndAt`, `actualStartAt`, `actualEndAt`, `location`, `comment`, `sendUpdatedNotification`, `updatedNotificationComment` | `meetingId`, `status` |
| `update_meeting_participants` | `companyId`, `meetingId`, one bucket | `participantIds`, `guestParticipantIds`, `externalParticipants[].{name,email,mobile?,comment?}`, `notificationComment` | `participantCount`, `externalParticipantCount` |
| `delete_meeting` | `companyId`, `meetingId` | `title` | `deleted` |
| `add_issue_to_meeting` | `companyId`, `meetingId`, `issueId` | `beforeIndex` (0-based) | `action` ADDED, `issueTitle` |
| `remove_issue_from_meeting` | `companyId`, `meetingId`, `issueId` | none | `action` REMOVED |
| `reorder_meeting_issue` | `companyId`, `meetingId`, `issueId`, `beforeIndex` | none | `action` REORDERED |
| `mark_meeting_issue_treated` | `companyId`, `meetingId`, `issueId` | none | `treated` |
| `set_meeting_issue_follow_up` | `companyId`, `meetingId`, `issueId`, `shouldFollowUp` | none | `shouldFollowUp` |
| `create_issue` | `title` | `companyId`, `description`, `responsibleId`, `meetingId`, `beforeIndex` | `id`, `status`, `meetingId`, `agendaOutcome` (ATTACHED, FAILED, UNKNOWN), `agendaAttached`, `agendaNote` |
| `update_issue` | `companyId`, `issueId` | `title`, `description`, `proposedResolution` | `id`, `status` |
| `update_issue_report` | `companyId`, `issueId`, `report` | none | `report` |
| `update_issue_conclusion` | `companyId`, `issueId`, `resolution` | none | `status` |
| `update_issue_responsible` | `companyId`, `issueId`, `responsibleId` | none | `status` |
| `create_issue_comment` | `companyId`, `issueId`, `content` | none | `commentCount` |
| `get_issue` | `companyId`, `issueId` | `includeFields` | `title`, `status`, `description`, `proposedResolution`, `lastResolution`, `responsible.{id,name}`, `relatedSections[]`, `comments[].{id,content,createdAt,createdBy}` |
| `search_issues` | none | `companyIds`, `search`, `status` [OPEN, IN_MEETING, CLOSED], `responsibleId`, `createdAtFrom`, `createdAtTo`, `updatedSince`, `count` (20), `pageToken` | `items[].{id,companyId,title,status,responsibleName,meetingId,meetingTitle}` |
| `close_issue`, `reopen_issue` | `companyId`, `issueId` | none | `status` |
| `add_issue_conversation` | `companyId`, `issueId`, `conversationId` | none | `action` |
| `add_issue_section` | `companyId`, `issueId`, `sectionId` (from `search_sections`) | none | `action` |
| `send_meeting_summons` | `companyId`, `meetingId` | `comment` | `sent` (notifies all participants, guests and externals included) |
| `rsvp_meeting` | `companyId`, `meetingId`, `participantId`, `rsvp` YES, NO or MAYBE | `comment` | `rsvp` |
| `start_meeting`, `close_meeting`, `reopen_meeting` | `companyId`, `meetingId` | none | `status` |
| `mark_meeting_attendance` | `companyId`, `meetingId`, `participantId`, `attended` | none | `attended` |
| `finalize_meeting_signing` | `companyId`, `meetingId` | none | `finalized` |
| `mark_meeting_participant_signed` | `companyId`, `meetingId`, `attendeeId` | none | `signatureState` |
| `send_meeting_signature_reminder` | `companyId`, `meetingId` | `idsToSign` | `sent` |
| `create_meetings_for_companies` | `companyIds` (max 500), `titleTemplate`, `plannedStartAt`, `plannedEndAt` | `location` | `items[].{companyId,entityId,status,warnings}` |
| `create_issues_for_companies` | `companyIds`, `titleTemplate` | `contentTemplate`, `responsibleId`, `responsibleByCompany[]` | `items[].{companyId,entityId,status,warnings}` |
| `create_task` | `companyId`, `title` | `description`, `dueDate` (timestamp), `responsibleId` | `id` |
| `update_task`, `complete_task` | `companyId`, `taskId` (from `search_tasks`) | `title`, `description`, `dueDate`, `responsibleId` | `id`, `completed` |
| `search_documents` | none | `companyIds`, `search`, `type` [MEETING, ANNUAL_MEETING, ...], `createdAtFrom`, `createdAtTo`, `count`, `pageToken` | `items[].{id,sourceDocumentId,displayName,type,parentId,searchMatch}` |
