# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Repository Purpose

This is the **documentation and GTM strategy repository** for ShiftBuddy — a WhatsApp-based ops task manager and asset control system for field teams (construction, equipment rental, logistics, field services). This repo contains **no application code**; the product codebase lives in a separate repository (`shift-buddy-pro`).

## Repository Structure

```
CLAUDE.md                             # This file — AI assistant instructions
.gitignore
docs/
├── prd/                              # Product Requirements Document (4-part)
│   ├── 01-vision.md                  # Executive summary, problem statement, market context
│   ├── 02-personas.md                # User personas and jobs-to-be-done
│   ├── 03-requirements.md            # Functional requirements (FR-001 to FR-063) and user flows
│   └── 04-architecture-and-roadmap.md  # Tech architecture, data model, KPIs, risks, roadmap
├── sales/
│   ├── prompts/                      # AI researcher prompts for lead generation
│   │   ├── lead-researcher.md        # Master prompt: signal-led lead gen on LinkedIn/Reddit (Italian)
│   │   ├── firecrawl-pipeline.md     # Firecrawl research pipeline (5 phases: discovery → enrichment → signals → scoring → output)
│   │   ├── agent-team-session-1.md   # Agent team prompt for pipeline Session 1 (COMPLETED)
│   │   ├── agent-team-session-2.md   # Agent team prompt for pipeline Session 2 (READY)
│   │   └── csv-research/             # Industry-specific CSV schema prompts for Clay-style enrichment (1 per ICP)
│   │       ├── construction.md
│   │       ├── equipment-rental.md
│   │       ├── logistics.md
│   │       ├── field-services.md
│   │       └── facility-management.md
│   ├── analysis/                     # Research output & analysis artifacts
│   │   ├── company-research/
│   │   │   └── master-company-list.md  # Single source of truth: 81 qualified + 17 DQ companies
│   │   └── signals-session-1.md      # 22 classified signals from Session 1 (pain, hiring, growth)
│   └── strategy/
│       ├── icp-profiles.md           # 5 ICP profiles with pain points and buyer personas
│       ├── signal-led-strategy.md    # Full GTM playbook: scoring, signal taxonomy, outreach sequences, LinkedIn content
│       └── communities.md            # Target communities: Reddit, LinkedIn, Facebook, forums, Slack/Discord
└── ShiftBuddy_Cost_Model_v2.xlsx     # Financial/cost model
```

### Tooling (not committed)
```
.firecrawl/                           # Firecrawl CLI cache and pipeline outputs (gitignored)
.agents/skills/firecrawl/             # Firecrawl skill installation for Claude Code
```

## Key Context for Working in This Repo

### Product
- ShiftBuddy turns WhatsApp conversations into structured task management and asset tracking for field teams
- No new app install for crews — everything works via WhatsApp Business API
- Web dashboard for managers (task board, asset registry, custody chain, reports)
- Target verticals: Construction (primary), Equipment Rental (primary), Logistics, Field Services, Facility Management

### Target Markets (priority order)
1. Italy (home market — search in Italian)
2. UK (English)
3. UAE + Saudi Arabia (English)
4. Brazil (Portuguese)
5. Sub-Saharan Africa — South Africa, Kenya, Nigeria (English)

### PRD Conventions
- Functional requirements use IDs `FR-001` through `FR-063` with MoSCoW priorities (Must/Should/Could/Won't)
- User personas: Marco (Ops Manager), Jose (Field Worker), Serena (Fleet Manager), Antonio (Owner)
- Roadmap: Phase 1 (MVP, weeks 1-12), Phase 2 (Growth, weeks 13-30), Phase 3 (Scale, weeks 31-52)

### Sales Strategy Conventions
- **Two scoring rubrics exist** (different purposes):
  - `signal-led-strategy.md` — **Outreach scoring** (7 categories, 100 pts): Industry 25, Size 15, Geography 15, Tech Stack 15, Hiring 10, Pain/Growth 10, Engagement 10. Used for outreach prioritization.
  - `firecrawl-pipeline.md` — **Pipeline scoring** (5 categories, 100 pts): Signal Strength 30, Industry Fit 25, Company Size 20, Geography 15, Tech Stack 10. Used for research qualification.
- Tier A (75-100), Tier B (50-74), Tier C (25-49)
- Three outreach sequences: A (Pain signal), B (Growth signal), C (Cold ICP-fit)
- Lead output format uses structured template (see `prompts/lead-researcher.md`)
- Outreach sequence personalization uses `{{token}}` syntax (e.g., `{{first_name}}`, `{{company}}`, `{{signal_detail}}`)

### Disqualification Criteria (applied across all research)
- Fewer than 15 employees
- More than 500 employees
- Uses SAP, Oracle, Samsara, Verizon Connect, ServiceTitan, Jobber (enterprise tools)
- No field operations (office-only company)
- Government agency or public utility

## Working with Documents

- All docs are Markdown with tables and structured templates
- Sales docs are living documents — check "Last updated" dates
- When updating strategy docs, maintain the existing formatting patterns
- ICP profiles and signal taxonomy should stay synchronized across `icp-profiles.md` and `signal-led-strategy.md`
- The master company list is organized by **Geography → ICP** (e.g., "Italy > ICP 1 — Construction", "Italy > ICP 2 — Equipment Rental")

## Research Pipeline Status

- **Pipeline prompt:** `docs/sales/prompts/firecrawl-pipeline.md`
- **Session 1:** COMPLETED (2026-02-16)
  - 81 qualified companies (32 ICP 1 + 31 ICP 2 + 19 ICP 3) across 5 geographies
  - 22 signals detected (10 pain, 5 hiring, 7 growth)
  - 10 companies scored (top: CANTIERI S.p.A. 90/100, ALC Contracting 87/100)
  - Signals report: `docs/sales/analysis/signals-session-1.md`
- **Session 2 priorities:**
  1. Enrich ICP 2/3 companies (fill LinkedIn/domain/employee gaps → target 60%+ completeness)
  2. Deep-dive UAE company-specific news (contract wins for top-scored companies)
  3. Brazil labor shortage / technology adoption angle
  4. Italy infrastructure companies in Terna EUR17.7B plan
  5. Score remaining companies with full rubric
  6. Discover ICP 4 (Field Services) and ICP 5 (Facility Management) companies
- **Key gap:** Many ICP 2/3 companies are "Discovery Only" — need enrichment before outreach
- **Tools:** Firecrawl CLI installed and authenticated
