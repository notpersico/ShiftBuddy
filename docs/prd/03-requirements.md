# 3. Functional Requirements & User Flows

## 3.1 WhatsApp Bot — Field Interface

### 3.1.1 Task Assignment

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-001 | The bot SHALL send task notifications to assigned crew members via WhatsApp, including: task type (deliver, move, service, return, inspect), asset identifier, origin site, destination site (if applicable), and deadline. | Must |
| FR-002 | Each task message SHALL include quick-reply buttons for **Accept**, **Reject**, and **Details** where the WhatsApp client supports interactive messages. | Must |
| FR-003 | On clients that do not support interactive buttons, the bot SHALL fall back to numbered text options (e.g., "Reply 1 to Accept, 2 to Reject, 3 for Details"). | Must |
| FR-004 | The bot SHALL support batch assignment — a dispatcher sends one command, and the system fans out individual task messages to each assigned crew member. | Should |
| FR-005 | Task messages SHALL display a human-readable reference code (e.g., `TSK-20260210-0042`) for voice/radio cross-reference. | Must |

### 3.1.2 Task Confirmation & Rejection

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-006 | When a crew member accepts a task, the bot SHALL confirm acceptance, echo the task summary, and update the task state to **Active**. | Must |
| FR-007 | When a crew member rejects a task, the bot SHALL prompt for a rejection reason (free-text or quick-reply: "Unavailable", "Wrong person", "Equipment issue", "Other"). | Must |
| FR-008 | Rejected tasks SHALL be returned to the dispatcher's queue on the web dashboard within 30 seconds, flagged with the rejection reason. | Must |
| FR-009 | If a crew member neither accepts nor rejects within a configurable timeout (default: 15 minutes), the bot SHALL send a reminder. After a second timeout (default: 15 minutes), the task SHALL auto-escalate to the dispatcher. | Must |
| FR-010 | A crew member SHALL be able to mark a task as **Completed** by replying "done", tapping a completion button, or sending a photo. | Must |
| FR-011 | The bot SHALL ask for an optional completion note upon task completion (free-text, skippable). | Should |

### 3.1.3 Asset Handoff Logging

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-012 | The bot SHALL support an asset handoff command where the outgoing custodian initiates a transfer by specifying the asset ID and the receiving person or site. | Must |
| FR-013 | Upon handoff initiation, the bot SHALL send a confirmation request to the receiving crew member: "Do you confirm receipt of [Asset X] from [Person Y]?" with Accept/Reject buttons. | Must |
| FR-014 | Both parties' confirmations (or the initiator's declaration + receiver's confirmation) SHALL be timestamped and recorded as an immutable custody-chain entry. | Must |
| FR-015 | If the receiver does not confirm within a configurable timeout (default: 30 minutes), the handoff SHALL be flagged as **Unconfirmed** and an alert sent to the dispatcher. | Must |
| FR-016 | The bot SHALL support site-to-site handoffs where no receiving person is specified (e.g., asset dropped at a depot). In this case, only the outgoing custodian's declaration is required. | Should |
| FR-017 | The bot SHALL allow an optional photo attachment during handoff to document asset condition. | Should |

### 3.1.4 Shift Open / Close

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-018 | Crew members SHALL be able to open a shift by sending a "start shift" message (or tapping a quick-reply button). The bot SHALL record the timestamp and confirm. | Must |
| FR-019 | The bot SHALL send a configurable shift-start prompt at scheduled times (e.g., 06:00) asking crew to check in. | Should |
| FR-020 | Crew members SHALL be able to close a shift by sending "end shift". The bot SHALL present a shift summary: tasks completed, tasks pending, assets currently in custody. | Must |
| FR-021 | If a crew member attempts to close a shift with open tasks or unresolved asset custody, the bot SHALL warn them and list the outstanding items. Shift close is still permitted after acknowledgment. | Must |
| FR-022 | Shift open/close events SHALL be recorded with timestamps for payroll and audit purposes. | Must |

### 3.1.5 Photo Capture

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-023 | The bot SHALL accept photos sent by crew at any time during an active task and associate them with the current task context. | Must |
| FR-024 | Dispatchers SHALL be able to request a photo from a specific crew member via the dashboard; the bot delivers the request as a WhatsApp message. | Should |
| FR-025 | Photos SHALL NOT be mandatory for task completion unless the task type is explicitly configured to require them (e.g., inspection tasks). | Must |
| FR-026 | All received photos SHALL be stored with metadata: sender, timestamp, associated task ID, and associated asset ID (if applicable). | Must |

### 3.1.6 Natural Language Understanding

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-027 | The bot SHALL understand intent from free-text messages in Italian, English, Spanish, and Arabic. | Must |
| FR-028 | Supported intents SHALL include at minimum: accept task, reject task, complete task, start shift, end shift, initiate handoff, ask asset location, report issue, request help. | Must |
| FR-029 | The bot SHALL auto-detect the language of incoming messages and respond in the same language. | Must |
| FR-030 | When the bot cannot determine intent with sufficient confidence (below configurable threshold, default 70%), it SHALL reply with a clarification prompt offering the most likely interpretations as quick-reply options. | Must |
| FR-031 | The bot SHALL support French and Portuguese as additional languages. | Could |
| FR-032 | The bot SHALL handle common misspellings, voice-to-text artifacts, and abbreviated field jargon. | Should |

### 3.1.7 Quick-Reply Buttons

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-033 | All decision-point messages (accept/reject, confirm handoff, shift open/close, completion) SHALL use WhatsApp interactive buttons when the client supports them. | Must |
| FR-034 | List messages (e.g., "select which asset") SHALL use WhatsApp list pickers for selections of 3–10 items. | Should |
| FR-035 | Button labels SHALL be localized to the crew member's detected or configured language. | Must |

---

## 3.2 Web Dashboard — Management Interface

### 3.2.1 Task Board

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-036 | The dashboard SHALL display a task board with columns/filters for: **Pending**, **Active**, **Overdue**, and **Done**. | Must |
| FR-037 | The dashboard SHALL support both Kanban (drag-and-drop) and list (table) views, togglable by the user. | Should |
| FR-038 | Each task card/row SHALL show: reference code, task type, assigned crew, asset ID, origin, destination, deadline, and current status. | Must |
| FR-039 | Dispatchers SHALL be able to create, edit, reassign, and cancel tasks from the dashboard. | Must |
| FR-040 | Task creation SHALL support bulk import via CSV upload. | Could |
| FR-041 | The task board SHALL update in real time (WebSocket or polling ≤5 seconds) as field crew interact with the bot. | Must |
| FR-042 | The dashboard SHALL support filtering tasks by: date range, crew member, team, task type, asset, site, and status. | Must |

### 3.2.2 Asset Registry & Last-Known Location

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-043 | The dashboard SHALL maintain an asset registry listing all tracked assets with: asset ID, description, category, current custodian, and last-known site. | Must |
| FR-044 | Last-known location SHALL be derived from the most recent task completion, handoff, or shift event involving the asset — not from GPS. | Must |
| FR-045 | Each asset SHALL have a detail page showing: current custodian, current site, full custody chain history, associated task history, and attached photos. | Must |
| FR-046 | Managers SHALL be able to add, edit, deactivate, and bulk-import assets. | Must |
| FR-047 | The asset registry SHALL support search by asset ID, description, custodian name, or site. | Must |

### 3.2.3 Assignment History & Custody Chain

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-048 | The dashboard SHALL provide a full audit log for each asset showing every custody transfer: from-person, to-person, from-site, to-site, timestamp, confirmation status, and any attached photos. | Must |
| FR-049 | Custody chain entries SHALL be immutable — edits create correction entries, not overwrites. | Must |
| FR-050 | The dashboard SHALL provide a full task history per crew member: assigned, accepted, rejected, completed, and overdue tasks with timestamps. | Must |

### 3.2.4 Overdue Alerts & Escalation

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-051 | The dashboard SHALL visually highlight overdue tasks (e.g., red badge, sorted to top). | Must |
| FR-052 | The system SHALL send overdue notifications to dispatchers via the dashboard (in-app notification + optional email). | Must |
| FR-053 | Escalation rules SHALL be configurable: define time thresholds and escalation targets (e.g., after 30 min → team lead; after 60 min → ops manager). | Should |
| FR-054 | The system SHALL support webhook-based escalation to external systems (e.g., PagerDuty, Slack). | Could |

### 3.2.5 Daily / Weekly Reports

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-055 | The dashboard SHALL generate a daily operations summary: tasks created, completed, overdue, rejected; average completion time; active crew count. | Must |
| FR-056 | The dashboard SHALL generate a weekly report with trend data: week-over-week task volume, completion rate, overdue rate, and crew utilization. | Should |
| FR-057 | Reports SHALL be exportable as PDF and CSV. | Must |
| FR-058 | Reports SHALL be schedulable for automatic email delivery to configured recipients. | Should |

### 3.2.6 User & Team Management

| ID | Requirement | Priority |
|--------|-------------|----------|
| FR-059 | Administrators SHALL be able to create, edit, and deactivate user accounts. Each user record includes: name, phone number (WhatsApp), role, team assignment, and preferred language. | Must |
| FR-060 | The system SHALL support the following roles: **Admin**, **Dispatcher**, **Team Lead**, **Crew Member**. | Must |
| FR-061 | Users SHALL be organizable into teams. A user may belong to one or more teams. | Must |
| FR-062 | Role-based access control SHALL restrict dashboard features: Crew Members have no dashboard access; Team Leads see only their team; Dispatchers see all operational data; Admins see everything including configuration. | Must |
| FR-063 | Onboarding a new crew member SHALL require only adding their WhatsApp number to the system. The bot initiates contact and confirms enrollment. | Must |

---

## 3.3 Core User Flows

### 3.3.1 Flow 1 — Morning Dispatch

**Actors:** Dispatcher (dashboard), Crew Members (WhatsApp)

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | Dispatcher | Opens dashboard, navigates to task board. Reviews any carry-over tasks from previous day. | Dashboard loads with pending/overdue tasks visible. |
| 2 | Dispatcher | Creates new tasks for the day: specifies task type, asset, origin/destination, deadline, and assigns crew members. | System validates inputs, creates tasks in **Pending** state. |
| 3 | System | — | Sends WhatsApp task notification to each assigned crew member with Accept/Reject buttons. |
| 4 | Crew Member | Receives WhatsApp message. Taps **Accept**. | Bot confirms: "✅ Task TSK-20260210-0042 accepted. Deliver Asset A-1234 from Depot North to Site 7 by 10:00." Dashboard updates task to **Active**. |
| 4a | Crew Member | (Alternative) Taps **Reject** and selects reason. | Bot acknowledges. Task returns to dispatcher queue as **Pending — Rejected** with reason. Dispatcher is notified. |
| 5 | Crew Member | Completes the task. Sends "done" or taps **Complete**. | Bot asks for optional note/photo. Records completion timestamp. Dashboard updates task to **Done**. |
| 6 | System | — | If crew member does not respond within timeout (FR-009), bot sends reminder. After second timeout, task escalates to dispatcher. |

### 3.3.2 Flow 2 — Asset Handoff

**Actors:** Outgoing Custodian (WhatsApp), Receiving Custodian (WhatsApp), System

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | Outgoing Custodian | Sends message: "Handoff A-1234 to Marco at Site 3" (or uses structured command). | Bot parses intent, identifies asset and recipient. |
| 2 | System | — | Bot replies to Outgoing Custodian: "Handing off Asset A-1234 to Marco Rossi at Site 3. Confirm? [Yes / Cancel]". |
| 3 | Outgoing Custodian | Taps **Yes**. Optionally attaches a photo of asset condition. | Bot records outgoing declaration with timestamp. |
| 4 | System | — | Bot sends message to Receiving Custodian (Marco): "Paolo has handed you Asset A-1234 at Site 3. Do you confirm receipt? [Confirm / Dispute]". |
| 5 | Receiving Custodian | Taps **Confirm**. | Bot confirms to both parties. Custody chain updated: custodian changed from Paolo → Marco, location updated to Site 3. Dashboard reflects new state. |
| 5a | Receiving Custodian | (Alternative) Taps **Dispute**. | Bot asks for reason. Handoff flagged as **Disputed**. Dispatcher alerted on dashboard. Both custodians notified of dispute status. |
| 6 | System | — | If no response from Receiving Custodian within 30 min (FR-015), handoff marked **Unconfirmed**. Dispatcher alerted. |

### 3.3.3 Flow 3 — End-of-Shift Reporting

**Actors:** Crew Member (WhatsApp), System, Dispatcher (dashboard)

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | Crew Member | Sends "end shift" or taps shift-close button. | Bot retrieves shift data. |
| 2 | System | — | Bot presents shift summary: "Your shift summary: ✅ 5 tasks completed, ⏳ 1 task still open (TSK-…-0087), 📦 You hold Asset A-5678. Close shift anyway? [Yes / Keep working]". |
| 3a | Crew Member | Has no open items. Taps **Yes**. | Bot confirms: "Shift closed at 18:32. Good work! 👋". Shift record saved. Dashboard updated. |
| 3b | Crew Member | Has open items. Taps **Yes** to close anyway. | Bot warns: "⚠️ Task TSK-…-0087 will be returned to the dispatcher queue. Asset A-5678 remains in your custody. Confirm close? [Confirm / Cancel]". |
| 4 | Crew Member | Confirms. | Shift closed. Open task returned to pending queue. Dispatcher notified of incomplete handover. |
| 5 | System | — | Dashboard shift log updated. Daily report data aggregated. |

### 3.3.4 Flow 4 — "Where Is X?" Query

**Actors:** Any authorized user (WhatsApp or Dashboard)

**Via WhatsApp:**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | User | Sends: "Where is A-1234?" or "dov'è il generatore?" | Bot parses intent and identifies asset via ID, name, or description match. |
| 2 | System | — | Bot replies: "📦 Asset A-1234 (Generator 20kW): Last seen at **Site 3**, held by **Marco Rossi** since 10 Feb 14:22. Status: Active task TSK-…-0055 (deliver to Site 7, due by 17:00)." |
| 2a | System | — | If multiple matches: "I found 3 assets matching 'generator': [list]. Which one?" with list picker. |
| 2b | System | — | If no match: "No asset found matching 'generatore'. Check the asset ID or ask your dispatcher." |

**Via Dashboard:**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | User | Searches asset in registry search bar. | Dashboard shows matching assets with current custodian, site, and active task info. |
| 2 | User | Clicks asset for detail view. | Full custody chain, task history, and photos displayed. |

### 3.3.5 Flow 5 — Overdue Task Escalation

**Actors:** System, Crew Member (WhatsApp), Dispatcher (dashboard), Team Lead / Ops Manager (dashboard + optional WhatsApp/email)

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | System | Task deadline passes. | Task status changes to **Overdue**. Dashboard highlights task in red. |
| 2 | System | — | Bot sends reminder to assigned crew member: "⏰ Task TSK-…-0042 is overdue (was due 10:00). Please complete or update status. [Done / Need more time / Can't complete]". |
| 3a | Crew Member | Taps **Done**. | Normal completion flow (FR-010). Overdue flag retained in history for reporting. |
| 3b | Crew Member | Taps **Need more time**. | Bot asks for new ETA. Updates deadline. Dispatcher notified of revised ETA. |
| 3c | Crew Member | Taps **Can't complete** or does not respond. | Proceeds to escalation. |
| 4 | System | Escalation Level 1 triggered (configurable, default: 30 min past deadline). | In-app notification + optional email to dispatcher. Dashboard overdue counter incremented. |
| 5 | System | Escalation Level 2 triggered (configurable, default: 60 min past deadline). | Notification to Team Lead and/or Ops Manager. If configured, WhatsApp message sent to escalation contacts. |
| 6 | Dispatcher / Manager | Reassigns task from dashboard, or contacts crew member directly. | Task updated. New assignee notified via bot. Original assignee notified of reassignment. |

---

## 3.4 Requirements Traceability Summary

| Priority | Count | Range |
|----------|-------|-------|
| **Must** | 46 | FR-001 – FR-063 |
| **Should** | 13 | FR-004, FR-011, FR-016, FR-017, FR-019, FR-032, FR-034, FR-037, FR-053, FR-056, FR-058, FR-032, FR-034 |
| **Could** | 3 | FR-031, FR-040, FR-054 |
| **Won't (this release)** | 0 | — |
