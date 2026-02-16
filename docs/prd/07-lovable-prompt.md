# 07 — Lovable Prompt: ShiftBuddy MVP Dashboard

> **Last updated:** 2025-06-18
> **Purpose:** Initial prompt for Lovable to scaffold the ShiftBuddy MVP

---

## Prompt

Build a B2B SaaS dashboard called **ShiftBuddy** — an accountability system for tracking physical assets (equipment, tools, vehicles) that move between people and job sites. The target user is an operations manager at a 20-200 person construction or field services company. He is the only user — no crew-facing interface, no multi-user roles.

The app answers one question: **"Who has what, where?"**

### Authentication

- Email/password signup and login via Supabase Auth
- Each user belongs to one organization (created on signup)
- Single role: admin. No RBAC, no permissions system
- Protected routes: redirect unauthenticated users to login
- 14-day free trial, no credit card required

### Pages and Navigation

Use a collapsible sidebar layout with these pages:

1. **Dashboard** (home/default route)
2. **Assets** (registry)
3. **Tasks** (simple task tracker)
4. **Sites**
5. **Team Members**
6. **Import** (WhatsApp chat import)
7. **Settings** (organization, account, export, language)

Include a persistent global search bar in the top navigation.

### Page 1: Dashboard

Single-screen operational overview. This is where the manager starts every morning.

- **KPI row (4 cards):** Total assets tracked, Total estimated value (EUR), Assets currently assigned, Assets unaccounted (no custody event in >7 days)
- **Map section:** Display all sites as pins on a map with asset count badges. Clicking a pin navigates to that site's detail view.
- **Attention list:** A card listing assets flagged as damaged, flagged as lost, or stale (no custody event in >14 days). Each item is clickable and navigates to the asset timeline.
- **Recent activity feed:** The last 20 custody events shown as a reverse-chronological list. Each entry shows: event type badge, asset name, from person, to person, site, timestamp.

### Page 2: Assets (Registry)

A searchable, filterable table of all assets in the organization.

Each asset has these fields:
- Name (required)
- Category: power tool / vehicle / heavy equipment / scaffolding / other (select)
- Serial number or internal ID (optional text)
- Estimated value in EUR (optional number — prompt the user to fill this)
- Status: Available / Assigned / In Transit / Maintenance / Lost (select with colored badges)
- Condition: Good / Fair / Damaged (select with colored badges)
- Current custodian (link to team member, nullable)
- Current site (link to site, nullable)

Table features:
- Search by name, serial number, category, custodian, site
- Filter by status, category, site (using filter chips or dropdowns)
- Sortable columns
- Click a row to open the **Asset Detail** view

**Asset Detail** view (slide-over panel or dedicated page):
- All asset fields, editable
- A "Deactivate" button (never delete — audit trail matters)
- **Custody Timeline** below the fields: complete custody history in reverse chronological order. Each entry shows: event type badge, from → to, site, condition note, timestamp. This is the dispute-resolution screen.
- A prominent **"Record Handoff"** button that opens a dialog/drawer to create a new custody event

**Add Asset** dialog:
- Minimal form: name, category, site, custodian, estimated value, condition
- Must complete in under 30 seconds

**Record Handoff** dialog (custody event creation):
- Select event type: Assign / Transfer / Return / Flag Damaged / Flag Lost
- From whom (auto-filled with current custodian if applicable)
- To whom (select team member, or "Return to pool" for returns)
- Site (select)
- Condition note (optional textarea)
- Timestamp auto-set to now
- On submit: create an immutable custody event (append-only, no edits, no deletes) and update the asset's current custodian, site, status, and condition accordingly

### Page 3: Tasks (Simple Task Tracker)

A lightweight task manager inspired by Todoist. The manager uses this to assign and track operational tasks for the day.

Each task has:
- Title (required)
- Description (optional text)
- Assigned to (select team member, nullable)
- Related asset (select asset, nullable — link a task to an asset, e.g. "Bring generator to Cantiere Nord")
- Related site (select site, nullable)
- Due date (optional date picker)
- Priority: Low / Medium / High / Urgent (select with colored indicators)
- Status: To Do / In Progress / Done (select)

Views:
- **List view (default):** Tasks grouped by status (To Do, In Progress, Done) — collapsible sections. Each task shows title, assignee, due date, priority badge. Click to expand/edit inline.
- **Filter/sort:** Filter by assignee, site, priority, status. Sort by due date, priority, creation date.

Features:
- Quick-add: A persistent input at the top to create a task fast (just title + enter, fill details after)
- Inline editing: Click any task to expand and edit fields without navigating away
- Drag-and-drop between status columns (optional — nice to have)
- Mark as done with a single click/checkbox
- Bulk actions: select multiple tasks to mark done or reassign

Data model addition — **tasks** table:
- id, org_id, title, description, assigned_to (FK to team_members, nullable), asset_id (FK to assets, nullable), site_id (FK to sites, nullable), due_date (nullable), priority (low/medium/high/urgent), status (todo/in_progress/done), created_at, updated_at, created_by (FK to profiles)

### Page 4: Sites

A list/grid of job sites with a map view.

Each site has:
- Name (required)
- Address (required, with geocoding to get lat/lng on entry)
- Coordinates (auto from geocoding)

Views:
- **Map view:** All sites as pins with asset count badges
- **List view:** Table with name, address, asset count
- Click a site to see a detail view listing all assets currently at that site

### Page 5: Team Members

A list of people who can be custodians. These are NOT app users — they are data rows.

Each team member has:
- Display name (required)
- Phone number (optional — store for future WhatsApp integration)
- Role label (optional free text, e.g. "operaio", "caposquadra", "autista")

Features:
- Add / edit team members
- Click a member to see all assets currently in their custody

### Page 6: Import (WhatsApp Chat Import)

This is the onboarding hook. The "magic trick" that eliminates cold-start.

Flow:
1. Upload area: drag-and-drop or click to upload a .txt file (WhatsApp chat export)
2. Processing state: show a progress indicator while parsing
3. Results screen: "We found X assets and Y team members in your chat."
   - Display extracted items in two tabs: Assets and Team Members
   - Each item shows: extracted name, type, confidence indicator, source message preview
   - Each item has three actions: Confirm, Edit, Dismiss
   - Confirmed items get added to the asset registry / team members list
4. Summary: "X assets added, Y team members added. Your registry is ready."

Note: The actual AI extraction logic is handled server-side. The UI just needs the upload flow, the review/confirm interface, and success state.

### Page 7: Settings

- **Organization:** Name, plan info, trial status with days remaining
- **Account:** Email, password change
- **CSV Export:** Two export buttons:
  - "Export Asset List" — current asset registry with custody state
  - "Export Custody Log" — full event history
- **Bulk Import:** CSV upload for assets (for managers migrating from spreadsheets)
- **Language:** Toggle between Italian and English. Save preference per user.

### Global Search

A search bar in the top navigation. The manager types "CAT 320" or "generatore" or "Marco" and gets instant results showing matching assets, team members, or sites in a dropdown. Results must feel instant. Clicking a result navigates to that entity's detail view.

### Data Model (Supabase)
These are my tables in supabase so create components adaptable

- **organizations**: id, name, plan, trial_ends_at, created_at
- **profiles** (linked to Supabase auth): id, org_id, auth_user_id, email, name, role
- **sites**: id, org_id, name, address, lat, lng, created_at
- **team_members**: id, org_id, display_name, aliases (text array), phone, created_at
- **assets**: id, org_id, name, category, serial_number, estimated_value, status, condition, current_custodian_id (FK to team_members), current_site_id (FK to sites), created_at, updated_at
- **custody_events** (append-only, immutable): id, org_id, asset_id, event_type (register/assign/transfer/return/flag_damaged/flag_lost), from_member_id (FK to team_members, nullable), to_member_id (FK to team_members, nullable), site_id (FK to sites, nullable), condition_note, created_at, created_by (FK to profiles)
- **tasks**: id, org_id, title, description, assigned_to (FK to team_members, nullable), asset_id (FK to assets, nullable), site_id (FK to sites, nullable), due_date (nullable), priority (low/medium/high/urgent), status (todo/in_progress/done), created_at, updated_at, created_by (FK to profiles)
- **imports**: id, org_id, uploaded_by (FK to profiles), file_name, file_hash, status, message_count, created_at
- **raw_messages**: id, import_id, org_id, timestamp, sender_raw, content, has_media, content_hash (unique per org for dedup)
- **extracted_items**: id, import_id, org_id, item_type (asset/team_member), extracted_data (jsonb), confidence, status (pending/confirmed/dismissed), source_message_ids (text array), created_at

### UI/UX Design

- **Style:** Clean, minimal, professional. Not playful — this is a tool for a busy ops manager at 6am.
- **Layout:** Collapsible sidebar navigation. Content area with consistent padding and max-width.
- **Colors:** Neutral base (slate/gray). Use color only for status badges (green = available/good, yellow = in transit/fair, red = damaged/lost, blue = assigned/maintenance).
- **Typography:** Clear hierarchy. Large KPI numbers on dashboard. Readable table text.
- **Responsive:** Mobile-friendly. The manager may check on his phone from a job site.
- **Components:** Use shadcn/ui components throughout — Card, Table, Dialog, Sheet, Badge, Select, Input, Button, Command (for search), Tabs, Toast.
- **Loading states:** Skeleton loaders for data-fetching components.
- **Empty states:** Helpful messages with CTAs when no data exists (e.g. "No assets yet. Add your first asset or import from WhatsApp.").
- **Language:** Support Italian (default) and English. All UI labels, buttons, placeholder text, empty states, and toast messages must be translatable. Use a simple key-based translation system (e.g. a translations object per language). The user switches language in Settings. Keep copy simple and direct in both languages.

### Code Quality Principles

Enforce these principles across the entire codebase:

- **SOLID Principles:**
  - **Single Responsibility:** Each component, hook, and service does one thing. A form component handles rendering, not data fetching. A service handles queries, not UI state.
  - **Open/Closed:** Components accept props and composition for extension. Use variants (via CVA) instead of conditionals for visual changes. Add new event types without modifying existing custody chain logic.
  - **Liskov Substitution:** Shared component interfaces are consistent — a Button is a Button everywhere, with the same prop contract.
  - **Interface Segregation:** Props interfaces are minimal. Don't pass entire objects when a component only needs a name and id. Use pick/omit to keep prop surfaces small.
  - **Dependency Inversion:** Components depend on hooks, not services directly. Hooks depend on abstractions (query keys, service interfaces), not Supabase calls. The data layer is swappable.

- **DRY (Don't Repeat Yourself):**
  - Shared UI patterns extracted into reusable components (e.g. a StatusBadge used across assets, tasks, and custody events)
  - Common form fields (site selector, team member selector, asset selector) built as reusable controlled components
  - Query key factory — never hardcode query key strings
  - Shared types and Zod schemas for entities used across pages
  - Translation keys organized by domain, not duplicated per page

- **React Best Practices:**
  - Never use useEffect for data fetching — always TanStack Query
  - Derive state during render instead of syncing with useEffect
  - Colocate state as close as possible to where it's used
  - Lift state up only when multiple siblings need it
  - Composition over inheritance — use children, render props, and compound components
  - Always handle loading, error, and empty states for every data-fetching component
  - Use controlled components for forms with proper validation (Zod schemas)
  - Memoize expensive computations with useMemo, not everything
  - Name components descriptively (AssetDetailPanel, CustodyTimeline, not Panel1, List2)
  - Keep components small — if a file exceeds 150-200 lines, split it into subcomponents

### What is NOT in this MVP

Do not build any of these:
- Photo/image uploads (V2 feature)
- Scheduling or calendar
- WhatsApp bot or any messaging integration
- Crew-facing interface (workers don't log in)
- Push notifications or reminders
- Reports or analytics beyond CSV export
- Multi-organization support
- Roles or permissions beyond single admin
- Offline mode
