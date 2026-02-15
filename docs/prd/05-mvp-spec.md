# 05 — MVP Spec: Custody Chain

> The smallest product that makes someone pay.

---

## The Bet

**"Who has what, with photo proof"** is worth €49/mo to any company that's ever lost a piece of equipment.

We're not building a task manager. We're not building a scheduling tool. We're building an **accountability system for physical assets that move between people and places.**

The insight: task management is crowded. Asset tracking software assumes barcodes and warehouses. Nobody is solving custody-chain-for-dumb-equipment used by people who live on WhatsApp and refuse to install apps. That's the gap.

---

## Problem Worth Solving

Construction and field service companies lose 3-8% of movable assets annually. For a company with €500K in equipment, that's €15-40K/year evaporating. Not because of theft — because of **no accountability chain.** Nobody knows who had it last. Nobody can prove condition at handoff. Disputes are unresolvable. Insurance claims fail.

The ops manager spends 30-60 minutes every morning calling around asking "where is the generator?" The owner writes off €40K at year-end and shrugs.

This isn't a technology problem. It's a visibility problem. The solution doesn't need GPS, barcodes, or IoT. It needs: **a log of who touched what, when, with a photo.**

---

## Target User

One buyer. One user. Same person for MVP.

**The operations manager at a 20-200 person field company who is personally tired of not knowing where their equipment is.**

They will do the data entry themselves. They will take the photos themselves. They will be the only person using the product for the first 2 weeks. That's fine. If the product doesn't survive the "single user doing all the work" test, it doesn't survive at all.

The crew doesn't touch this product. Not yet. The manager is the system of record. If that creates enough value to pay for, we've validated. If it doesn't, no WhatsApp bot saves us.

---

## What's In

### 1. Onboarding — WhatsApp Import (The Magic Trick)

The cold start problem kills asset trackers. Nobody will manually enter 50 equipment items on day one.

Solution: **import their existing WhatsApp group chat.** Manager exports their ops group (built-in WhatsApp feature, produces .txt file), uploads it, AI extracts every asset mention into a pre-populated registry.

This is the demo. This is the hook. This is why they don't bounce on day one.

- Upload .txt file from WhatsApp export
- Deterministic parser splits messages (handle Android/iOS/locale variants)
- LLM pass extracts: asset names, types, last mentioned location, condition notes, people associated
- Manager sees: "We found 23 assets in your chat. Confirm, edit, or dismiss each one."
- Confirmed assets land in the registry, pre-populated. Manager fills gaps (photo, value, serial number).
- Also extracts team member names from chat participants → pre-populates team list.

This replaces 30 minutes of manual data entry with 3 minutes of review. It uses their real data. It's the "before/after" moment: your chaos, now structured.

### 2. Asset Registry

The canonical list of everything worth tracking.

Each asset has:
- Name, category (power tool / vehicle / heavy equipment / scaffolding / other)
- Photo (required — snap or upload)
- Serial number or internal ID (optional)
- Estimated value (optional but prompted — drives ROI messaging)
- Status: Available / Assigned / In Transit / Maintenance / Lost
- Current custodian (who has it right now)
- Current site (where it is right now)
- Condition: Good / Fair / Damaged

Operations:
- Add single asset (photo + basic fields, <30 seconds)
- CSV bulk import (for managers migrating from spreadsheets)
- Search by name, serial number, category, custodian, site
- Filter by status, category, site
- Edit, deactivate (never delete — audit trail matters)

### 3. Custody Chain (The Core)

Every time an asset changes hands, a custody event is recorded. This is the product.

A custody event captures:
- Asset
- From whom (or "initial registration" / "returned to pool")
- To whom
- At which site
- Photo of asset condition at moment of transfer (**mandatory** — this is non-negotiable)
- Condition note (optional free text)
- Timestamp (auto)
- GPS coordinates from photo metadata (stored if available, not required)

Event types:
- **Register** — asset enters the system (first photo)
- **Assign** — asset given to a person
- **Transfer** — asset moves from person A to person B
- **Return** — asset returned to pool / depot (no custodian)
- **Flag: Damaged** — condition change logged with photo
- **Flag: Lost** — custodian reports asset missing

Custody events are **append-only and immutable.** No edits. No deletes. Corrections create new events with notes. This is the legal-grade audit trail that makes the product worth paying for.

### 4. Asset Timeline

For any asset, show the complete custody history in reverse chronological order.

Each entry shows: who → who, when, where, condition photo, notes. Click any entry to see the photo full-screen.

This is the screen the manager opens when someone says "the generator was already damaged when I got it." They scroll, find the handoff photo, and the argument is over in 10 seconds.

### 5. Sites

Simple list of physical locations where work happens.

- Name, address
- Geocoded coordinates (free geocoding API on address entry)
- Map view showing all sites with asset count badges

Clicking a site shows all assets currently at that site. That's it.

### 6. Team Members

People who can be custodians of assets.

- Name, phone number (for future WhatsApp integration — store now, use later)
- Role label (free text: "operaio", "caposquadra", "autista" — no permissions system)
- Auto-populated from WhatsApp import (chat participants)
- Manager can add/edit manually

Team members do NOT have accounts. They do NOT log in. They exist only as custody chain participants. The manager records everything on their behalf.

### 7. Dashboard

Single-screen operational overview:

- **KPI bar:** Total assets tracked, total estimated value, assets assigned, assets unaccounted (no check-in in >7 days)
- **Map:** Sites with pins, asset counts as badges. Click to drill into site.
- **Attention list:** Assets flagged damaged, assets flagged lost, assets with no custody event in >14 days (stale)
- **Recent activity:** Last 20 custody events (live feed of what moved)

### 8. Search

Global search bar. Manager types "CAT 320" or "generatore" or "Marco" — results show matching assets, team members, or sites. The "where is it?" feature. Must return results in <1 second.

### 9. CSV Export

Export asset list with current custody state. Export full custody log. Managers need this for insurance, audits, and proving value to their boss during the trial.

---

## What's Out

| Feature | Why |
|---|---|
| Task management | Different problem, different product surface. Layer it after custody is validated. |
| WhatsApp bot | The riskiest and most expensive piece. Validate demand with dashboard first. Bot is phase 2. |
| Crew-facing interface | Crew doesn't log in. Manager is the sole user. Crew interface requires the bot. |
| Notifications / reminders | No push yet. Manager pulls info when they need it. |
| Reports / analytics | CSV export is sufficient. Automated reports are a paid tier upsell later. |
| Multi-org / team hierarchy | Single org, flat team. One manager per account. |
| Roles / permissions / RBAC | One role: admin. Everyone else is a data row, not a user. |
| Scheduling / calendar | Not this product. |
| Offline mode | Not for web MVP. |
| Mobile native app | Responsive web is sufficient. |
| Integrations | Nothing to integrate with yet. |
| i18n | Italian first. English second. No framework — just write both. |

---

## Data Model

```
organizations
  id, name, plan, trial_ends_at, created_at

profiles (Supabase auth-linked)
  id, org_id, auth_user_id, email, name, role

sites
  id, org_id, name, address, lat, lng, created_at

team_members (NOT auth users — just data rows)
  id, org_id, display_name, aliases[], phone, created_at

assets
  id, org_id, name, category, serial_number,
  photo_url, estimated_value, status, condition,
  current_custodian_id → team_members,
  current_site_id → sites,
  created_at, updated_at

custody_events (append-only, immutable)
  id, org_id, asset_id,
  event_type, -- register | assign | transfer | return | flag_damaged | flag_lost
  from_member_id → team_members (nullable),
  to_member_id → team_members (nullable),
  site_id → sites (nullable),
  photo_url (required),
  photo_lat, photo_lng (nullable),
  condition_note,
  created_at, created_by → profiles

imports
  id, org_id, uploaded_by → profiles,
  file_name, file_hash, status, message_count,
  created_at

raw_messages
  id, import_id, org_id,
  timestamp, sender_raw, content, has_media,
  content_hash (unique per org — dedup on re-import)

extracted_items
  id, import_id, org_id,
  item_type, -- asset | team_member
  extracted_data (jsonb),
  confidence, status, -- pending | confirmed | dismissed
  source_message_ids[],
  created_at
```

RLS: everything scoped by org_id. No cross-org data access.

---

## Pricing

```
TRIAL: 14 days, full access, no card required
  - Unlimited assets, unlimited events, all features

STARTER: €49/mo (or €490/year)
  - Up to 50 assets, 10 team members
  - Full custody chain, photo proof, CSV export

PRO: €99/mo (or €990/year)
  - Up to 200 assets, 30 team members
  - Everything in Starter + priority support

BUSINESS: €199/mo (or €1990/year)
  - Unlimited assets and members
  - Everything in Pro + API access + custom fields
```

Flat-rate, not per-seat. Asset count is the scaling axis. More assets = more value = natural upgrade path.

**Trial expiry behavior:** Read-only mode. All data visible, no new custody events or assets. The data becomes hostage. This is deliberate.

---

## The Demo

On a live call with a prospect:

1. "Open WhatsApp. Pick your busiest work group. Export chat, no media. Send me the file."
2. Upload it. 30-second wait.
3. "Here. 23 assets your team mentioned this month. 4 were flagged as damaged in conversations. This one — the 'generatore' — was last mentioned by Luca at Cantiere Nord, 6 days ago. Do you know where it is right now?"
4. They don't.
5. "That's the problem. Let me show you what happens when you start tracking."
6. Add one asset with photo. Assign to a person. Show the timeline. Show the map.
7. "Next time someone asks 'who has the generator?' you answer in 5 seconds with a photo. €49/month. You lost €40K last year."

---

## Conversion Mechanics

**What triggers payment:**
- Manager has entered 10+ assets with real values
- Manager has recorded 3+ handoffs with photos
- Manager has searched "where is X?" and gotten an instant answer
- Trial expires and they try to record a handoff → paywall
- The data they've built is now valuable enough to pay to keep

**Email sequence during trial:**
- Day 1: "You're tracking €__K in assets. Here's your first custody report." (PDF)
- Day 7: "__ handoffs recorded. That's __ fewer disputes waiting to happen."
- Day 11: "Trial ends in 3 days. Here's what your team moved this week."
- Day 14: "Trial ended. Your data is safe — upgrade to keep recording."
- Day 17: "You tried to record 3 handoffs this week. Upgrade to continue."

---

## Success Criteria

This MVP is validated if:

1. **>8% trial→paid conversion rate** — industry average for B2B SaaS is 5-7%. We need to beat it.
2. **>70% of signups add ≥5 assets** — proves onboarding works.
3. **>30% of signups record ≥3 handoffs** — proves core loop activates.
4. **Average tracked asset value >€50K per account** — proves they're tracking real stuff, not testing.
5. **≥3 paying customers within 60 days of launch** — proves willingness to pay.

If any of these fail, diagnose before building more.

---

## What This Proves (and What It Doesn't)

**Proves:**
- Ops managers will pay for asset visibility
- Photo-proof custody chain has tangible ROI
- The WhatsApp import hook drives activation
- There's a market willing to pay €49-199/mo for this

**Doesn't prove:**
- Field workers will interact with a WhatsApp bot (that's phase 2's bet)
- Task management is needed alongside asset tracking
- The product works at scale (>50 assets, >30 team members)
- Multi-site operations need different UX

**Phase 2 trigger:** When 3+ paying customers independently ask "can my workers update this from WhatsApp?" — that's the signal to build the bot. Not before. Let demand pull the feature out of you.

---

## Risks

| Risk | Severity | Mitigation |
|---|---|---|
| **Manager won't do data entry** | High | WhatsApp import eliminates cold start. Keep handoff flow under 15 seconds. Position data entry fatigue as the argument FOR the bot (phase 2). |
| **"This is just a spreadsheet"** | Medium | Spreadsheets don't have: immutable audit trail, mandatory photos, map view, instant search, custody chain. The simplicity is the feature. |
| **WhatsApp export parsing breaks** | Medium | Format varies by OS/locale. Build parser with test suite covering Android IT, iOS IT, Android EN, iOS EN. Handle gracefully — worst case, manual entry still works. |
| **AI extraction inaccurate** | Medium | Confidence scores + manual confirm/dismiss. Manager reviews everything. Bad extraction is annoying, not fatal — they can add assets manually. |
| **No virality / only 1 user per org** | Low (for now) | Acceptable for MVP. Virality comes with the bot (workers become users). Dashboard-only is a single-player game by design. |
| **Pricing too low** | Low | €49/mo is deliberately low to reduce purchase friction. Raise prices with confidence after proving value. Easier to go up than down. |
