# 4. Technical Architecture, Success Metrics, Risks & Roadmap

## 4.1 High-Level Architecture

### System Components

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  WhatsApp    │────▶│  Bot Service  │────▶│  NLP / Intent   │
│  Cloud API   │◀────│  (Webhook)    │◀────│  Engine          │
└─────────────┘     └──────┬───────┘     └─────────────────┘
                           │
                    ┌──────▼───────┐
                    │  API Gateway  │
                    └──┬───┬───┬──┘
                       │   │   │
          ┌────────────┘   │   └────────────┐
          ▼                ▼                ▼
   ┌─────────────┐  ┌───────────┐  ┌──────────────┐
   │ Task Engine  │  │  Asset    │  │ Notification │
   │              │  │ Registry  │  │  Service     │
   └──────┬──────┘  └─────┬─────┘  └──────────────┘
          │                │
          └───────┬────────┘
                  ▼
          ┌──────────────┐     ┌──────────────┐
          │  PostgreSQL   │     │    Redis      │
          │  (primary)    │     │  (cache/queue)│
          └──────────────┘     └──────────────┘

   ┌──────────────────┐
   │  Web Dashboard    │◀──── React SPA, talks to API Gateway
   │  (React)          │
   └──────────────────┘
```

| Component | Responsibility |
|---|---|
| **Bot Service** | Receives WhatsApp webhooks, routes inbound messages (text, voice notes, images) to NLP engine, sends outbound messages via Cloud API. Manages conversation state and template messages. |
| **NLP / Intent Engine** | Classifies user intent (e.g. `task.complete`, `asset.checkout`, `shift.start`). Handles voice-note transcription (Whisper API), typo correction, and slang normalization. Starts as rule-based + keyword matching; evolves to LLM-backed in Phase 2. |
| **Task Engine** | CRUD for tasks, assignment logic, escalation rules, due-date tracking, recurrence. Emits events on state changes. |
| **Asset Registry** | Tracks assets (tools, vehicles, equipment) — location, custody chain, condition. Handles checkout/checkin/handoff workflows. |
| **Notification Service** | Scheduled and event-driven notifications: shift reminders, overdue alerts, daily digests. Manages WhatsApp template message catalog and delivery status tracking. |
| **API Gateway** | REST API (versioned, `/api/v1/`) serving both the dashboard and bot service. Auth, rate limiting, request validation. |
| **Web Dashboard** | Management UI for supervisors: task board, asset map, team overview, reports. Real-time updates via WebSocket. |

### Data Model Overview

```
User
├── id: UUID (PK)
├── phone: string (E.164, unique)
├── whatsapp_id: string
├── name: string
├── role: enum (worker | supervisor | admin)
├── org_id: FK → Organization
├── active: boolean
├── created_at, updated_at: timestamp

Site
├── id: UUID (PK)
├── name: string
├── address: string
├── geo: point (lat/lng)
├── org_id: FK → Organization

Shift
├── id: UUID (PK)
├── user_id: FK → User
├── site_id: FK → Site
├── started_at: timestamp
├── ended_at: timestamp | null
├── status: enum (active | completed | no_show)

Task
├── id: UUID (PK)
├── title: string
├── description: text
├── assigned_to: FK → User
├── created_by: FK → User
├── site_id: FK → Site
├── status: enum (pending | accepted | in_progress | completed | overdue | cancelled)
├── priority: enum (low | normal | urgent)
├── due_at: timestamp
├── completed_at: timestamp | null
├── recurrence: jsonb | null
├── attachments: jsonb (media URLs)
├── created_at, updated_at: timestamp

Asset
├── id: UUID (PK)
├── name: string
├── type: string (e.g. "drill", "van", "scaffold")
├── serial_number: string
├── status: enum (available | checked_out | maintenance | lost)
├── current_holder: FK → User | null
├── site_id: FK → Site | null
├── org_id: FK → Organization
├── last_seen_at: timestamp
├── condition: enum (good | fair | damaged)

Handoff
├── id: UUID (PK)
├── asset_id: FK → Asset
├── from_user: FK → User
├── to_user: FK → User
├── site_id: FK → Site | null
├── handoff_type: enum (checkout | checkin | transfer)
├── photo_url: string | null
├── notes: text | null
├── confirmed_at: timestamp | null
├── created_at: timestamp
```

Key relationships:
- A **User** belongs to an **Organization** and can be assigned to multiple **Sites**
- **Tasks** are assigned to a User at a Site with optional recurrence
- **Assets** have a custody chain tracked via **Handoff** records
- **Shifts** tie a User to a Site for a time window; tasks are contextual to active shifts

### Integration Points

| Integration | Protocol | Purpose |
|---|---|---|
| **WhatsApp Cloud API** (Meta) | HTTPS webhooks + REST | Inbound/outbound messaging, template messages, media upload/download, read receipts |
| **Media Storage** (S3 / R2) | S3-compatible API | Store voice notes, photos (asset condition, task evidence), documents. Pre-signed URLs for dashboard access |
| **Auth** | JWT + API keys | Dashboard: email/password → JWT. Bot: phone-based identity (WhatsApp number = user ID). API-to-API: signed service tokens |
| **Whisper API** (OpenAI) | REST | Voice note transcription for NLP input |
| **Geocoding** (Google Maps / Mapbox) | REST | Site address → coordinates, asset last-known-location display |
| **Webhook / Events** (outbound) | HTTPS | Future: push events to customer ERP/systems (Phase 2) |

### Tech Stack

| Layer | Choice | Rationale |
|---|---|---|
| **Language** | TypeScript (strict mode) | Shared types across bot, API, and dashboard. One language for the whole team. |
| **Runtime** | Node.js 20 LTS | Async I/O fits webhook-heavy workload. Mature ecosystem. |
| **API Framework** | Fastify | Faster than Express, schema-based validation, good plugin ecosystem. |
| **Database** | PostgreSQL 16 | JSONB for flexible fields, PostGIS for geo queries, rock-solid reliability. |
| **Cache / Queue** | Redis 7 (+ BullMQ) | Rate-limit counters, conversation state cache, background job queue (notifications, transcription). |
| **Dashboard** | React 18 + Vite + TailwindCSS | Fast dev iteration. Zustand for state. React Query for server state. |
| **ORM** | Drizzle ORM | Type-safe, lightweight, SQL-close. Migrations via `drizzle-kit`. |
| **Hosting** | Railway / Render (Phase 1) → AWS ECS (Phase 2+) | Start cheap and simple, migrate when scale demands it. |
| **Media Storage** | Cloudflare R2 | S3-compatible, no egress fees. |
| **CI/CD** | GitHub Actions | Monorepo-aware builds with Turborepo caching. |
| **Monitoring** | Sentry (errors) + Axiom (logs) + UptimeRobot | Cost-effective observability for early stage. |

### Monorepo Structure

```
shiftbuddy/
├── package.json              # Root workspace config
├── turbo.json                # Turborepo pipeline
├── .github/workflows/        # CI/CD
├── packages/
│   ├── api/                  # Fastify API server
│   │   ├── src/
│   │   │   ├── routes/       # REST endpoints
│   │   │   ├── services/     # Business logic (task engine, asset registry)
│   │   │   ├── jobs/         # BullMQ workers (notifications, transcription)
│   │   │   └── middleware/   # Auth, rate limiting, validation
│   │   └── package.json
│   ├── bot/                  # WhatsApp bot service
│   │   ├── src/
│   │   │   ├── webhook/      # Incoming message handler
│   │   │   ├── nlp/          # Intent classification, entity extraction
│   │   │   ├── flows/        # Conversation flows (task creation, asset checkout)
│   │   │   └── templates/    # WhatsApp message templates
│   │   └── package.json
│   ├── dashboard/            # React SPA
│   │   ├── src/
│   │   │   ├── pages/        # Task board, asset map, team, reports
│   │   │   ├── components/
│   │   │   └── api/          # API client (generated from shared types)
│   │   └── package.json
│   ├── shared/               # Shared code
│   │   ├── src/
│   │   │   ├── types/        # Entity types, API contracts, enums
│   │   │   ├── validators/   # Zod schemas
│   │   │   └── utils/        # Date helpers, phone normalization, etc.
│   │   └── package.json
│   └── docs/                 # Documentation (this PRD, API docs, runbooks)
│       ├── prd/
│       └── api/
├── docker-compose.yml        # Local dev: Postgres + Redis
└── README.md
```

Managed with **pnpm workspaces** + **Turborepo** for caching and task orchestration.

---

## 4.2 Success Metrics & KPIs

### Adoption Metrics

| Metric | Definition | Phase 1 Target | Phase 2 Target |
|---|---|---|---|
| **Daily Active Users (DAU)** | Unique users who send ≥1 message to ShiftBuddy per day | 60% of registered workers | 80% |
| **Message Response Rate** | % of bot-initiated messages that receive a reply within 30 min | 70% | 85% |
| **Task Completion Rate** | % of assigned tasks marked completed (not cancelled/overdue) | 75% | 90% |
| **Onboarding Time** | Time from invite to first completed task | < 10 min | < 5 min |

### Operational Metrics

| Metric | Definition | Phase 1 Target | Phase 2 Target |
|---|---|---|---|
| **Lost Asset Incidents** | Assets marked "lost" or unaccounted for per month | 50% reduction vs. baseline | 80% reduction |
| **Morning Dispatch Time** | Time from shift start to all workers having task assignments | < 15 min | < 5 min (automated) |
| **Overdue Task Rate** | % of tasks past due date without completion | < 15% | < 8% |
| **Asset Utilization** | % of assets checked out vs. total available on any given day | Measured (baseline) | +20% improvement |

### Business Metrics

| Metric | Definition | Target |
|---|---|---|
| **Time-to-Value** | Days from signup to "aha moment" (first team completing a full shift cycle) | < 3 days |
| **Monthly Churn** | % of paying orgs that cancel per month | < 5% |
| **NPS** | Net Promoter Score from quarterly surveys | > 40 |
| **Revenue per Seat** | MRR / total active seats | €8–15/seat (Phase 1 pricing) |
| **Expansion Revenue** | % of MRR from upsells (additional seats, premium features) | > 20% by Phase 2 |

---

## 4.3 Risks & Mitigations

### R1: WhatsApp API Costs & Rate Limits

**Risk:** Meta charges per conversation (marketing, utility, service categories). High-volume orgs could make unit economics unsustainable. Rate limits (80 messages/second for new accounts, 1,000 business-initiated conversations/day initially) may throttle operations.

**Mitigations:**
- Design conversations to maximize the 24-hour service window (free replies) — batch notifications into digest messages rather than 1:1 alerts
- Use template messages sparingly; prefer user-initiated flows
- Apply for higher messaging tiers early (requires quality rating maintenance)
- Build cost tracking per org to monitor per-seat messaging costs
- Set hard limits + alerts at 80% of tier capacity

### R2: Worker Adoption & Response Rates

**Risk:** Field workers may ignore bot messages, respond inconsistently, or resist changing habits. WhatsApp fatigue from personal use could reduce engagement.

**Mitigations:**
- Keep interactions to <3 taps — quick-reply buttons, list messages, minimal free-text input
- Supervisor visibility into response rates creates social accountability
- Gamification light-touch: completion streaks, team leaderboards (Phase 2)
- Onboard with supervisor present; first task is dead-simple ("confirm you received this")
- Respect off-hours: no messages outside shift windows

### R3: NLP Accuracy in Noisy Field Conditions

**Risk:** Voice notes from construction sites (background noise), typos on phone keyboards, regional slang, and multilingual input will degrade intent classification.

**Mitigations:**
- Phase 1: Use structured inputs (buttons, lists) as primary interaction — free text is secondary
- Voice notes: Whisper API handles noise reasonably well; add confidence thresholds and fallback to "I didn't understand, pick an option"
- Build a labeled dataset from real conversations to fine-tune over time
- Phase 2: LLM-backed classification (GPT-4o-mini) with domain-specific prompt engineering
- Always offer a "menu" escape hatch so users never get stuck in NLP loops

### R4: Data Privacy (GDPR & Asset Location)

**Risk:** Storing worker phone numbers, location data, and shift patterns creates GDPR obligations. Asset GPS tracking may cross into worker surveillance territory.

**Mitigations:**
- Data Processing Agreement (DPA) with each customer org — they are the data controller
- Phone numbers stored hashed where possible; WhatsApp IDs as primary identifiers
- Asset location = site-level granularity only (not real-time GPS of workers)
- Explicit consent flow on first bot interaction with link to privacy policy
- Data retention policy: auto-purge completed task data after 12 months, handoff records after 24 months
- EU hosting (AWS eu-central-1 / Railway EU region) from day one
- Right-to-erasure endpoint for GDPR deletion requests

### R5: Platform Dependency on Meta

**Risk:** Meta could change API pricing, terms, rate limits, or deprecate features. Account suspension (even temporary) would halt all operations.

**Mitigations:**
- Abstract messaging layer behind an internal interface — bot service talks to a `MessagingProvider`, not directly to WhatsApp SDK
- Design for future multi-channel support (SMS via Twilio, Telegram) as fallback
- Maintain WhatsApp Business API quality rating above 90% (monitor daily)
- Keep a Twilio SMS fallback for critical notifications (shift start, urgent tasks)
- Diversify: Phase 3 includes web/mobile notification channels

### Risk Matrix Summary

| Risk | Likelihood | Impact | Priority |
|---|---|---|---|
| WhatsApp API costs | Medium | High | **P1** |
| Worker adoption | High | High | **P0** |
| NLP accuracy | Medium | Medium | **P1** |
| Data privacy | Low | High | **P1** |
| Platform dependency | Low | Critical | **P2** |

---

## 4.4 Roadmap

### Team Assumptions

| Role | Phase 1 | Phase 2 | Phase 3 |
|---|---|---|---|
| Full-stack engineer | 2 | 3 | 4 |
| Product / founder | 1 | 1 | 1 |
| Designer (part-time) | 0.5 | 1 | 1 |
| DevOps / infra | 0 (eng handles) | 0.5 | 1 |
| **Total** | **3.5** | **5.5** | **7** |

---

### Phase 1 — MVP (Weeks 1–12)

**Goal:** Prove core value with 3–5 pilot customers. Workers can receive tasks and track assets via WhatsApp. Supervisors can manage via a basic dashboard.

| Month | Deliverables |
|---|---|
| **Weeks 1–4** | Monorepo setup, CI/CD, DB schema, WhatsApp Business API integration, basic webhook handler. User registration flow (supervisor invites worker via phone number). |
| **Weeks 5–8** | Task engine: create → assign → accept → complete flow via WhatsApp buttons. Asset registry: checkout/checkin via bot (photo confirmation). Basic dashboard: task list view, asset inventory. |
| **Weeks 9–12** | Notification service: shift reminders, overdue alerts, daily digest. Dashboard: task board (kanban), team view, basic filters. Pilot onboarding with 3–5 companies. Bug fixes, UX iteration. |

**Key dependencies:**
- WhatsApp Business API account approval (apply week 1, ~1–2 week turnaround)
- Approved message templates (submit early, iteration takes days)
- Pilot customer agreements signed by week 6

**Exit criteria:** 3 paying pilot customers completing daily task cycles via WhatsApp with >60% worker response rate.

---

### Phase 2 — Growth & Intelligence (Weeks 13–30)

**Goal:** Improve NLP, add reporting, launch integrations. Scale to 20–50 customers.

| Period | Deliverables |
|---|---|
| **Weeks 13–18** | LLM-backed intent engine (GPT-4o-mini). Voice note transcription + classification. Free-text task creation ("assign Marco to clean site B tomorrow"). Conversation context memory (multi-turn flows). |
| **Weeks 19–24** | Dashboard v2: reporting (task completion trends, asset utilization, team performance). Export to CSV/PDF. Role-based access control. Multi-site management. |
| **Weeks 25–30** | Integration framework: webhook events for external systems. First integrations: ERP connector (SAP Business One, Odoo), rental system connector. API keys for customer-built integrations. Billing system (Stripe) + self-serve signup. |

**Key dependencies:**
- Labeled dataset from Phase 1 conversations (minimum 2,000 classified messages)
- ERP partner for integration testing
- Stripe account + billing logic

**Exit criteria:** 30+ paying customers, NLP handling 70% of messages without fallback, one live ERP integration.

---

### Phase 3 — Predictive & Scale (Weeks 31–52)

**Goal:** Predictive capabilities, multi-org, platform maturity. Path to 200+ customers.

| Period | Deliverables |
|---|---|
| **Weeks 31–38** | Predictive maintenance alerts (asset usage patterns → service reminders). Smart scheduling suggestions (based on historical task duration, worker skills, site proximity). Utilization optimization dashboard. |
| **Weeks 39–44** | Multi-org / enterprise: org hierarchies, SSO (SAML), audit logs, custom roles. Multi-channel: Telegram bot, SMS fallback, web push notifications. Mobile companion app (React Native — lightweight, for supervisors on-site). |
| **Weeks 45–52** | Marketplace: integration directory (pre-built connectors). White-label / API-first mode for large customers embedding ShiftBuddy. Performance hardening: migrate to AWS ECS, read replicas, CDN. SOC 2 Type I preparation. |

**Key dependencies:**
- Sufficient historical data for predictive models (6+ months of operations data)
- Enterprise pilot customer for multi-org requirements validation
- Security audit / penetration test before enterprise launch

**Exit criteria:** 200+ customers, predictive features active for 20% of orgs, enterprise pipeline with 5+ prospects, infrastructure handling 10x Phase 1 load.

---

### Roadmap Visual

```
2026    Q2          Q3          Q4          2027 Q1
        ├───────────┼───────────┼───────────┤
Phase 1 ████████████|           |           |
        MVP         |           |           |
                    |           |           |
Phase 2             |███████████████████    |
                    Growth & Intelligence   |
                                            |
Phase 3                         |███████████████████
                                Predictive & Scale
```

---

*Last updated: 2026-02-10*
