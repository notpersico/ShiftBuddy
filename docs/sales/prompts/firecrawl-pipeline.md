# ShiftBuddy — Firecrawl Research Pipeline Prompt

> **Version:** 2.0
> **Last updated:** 2026-02-16
> **Purpose:** Systematic web research pipeline using Firecrawl CLI to discover, enrich, and qualify ICP companies across all 5 verticals and 5 geographies. Designed to be executed step-by-step in a Claude Code session with Firecrawl installed.

---

## SESSION LOG

| Session | Date | Result | Key Metrics |
|---------|------|--------|-------------|
| **Session 1** | 2026-02-16 | **COMPLETED** | 81 qualified companies (32 ICP1 + 31 ICP2 + 19 ICP3), 22 signals, 10 companies scored |
| Session 2 | TBD | Planned | Target: Enrich ICP 2/3, discover ICP 4/5, score all companies |
| Session 3 | TBD | Planned | Target: 150+ companies, 80%+ enrichment, 30+ signals, 20+ Tier A |

### Session 1 Results
- **Pipeline:** 35 → 81 qualified companies (+17 DQ)
- **ICP coverage:** ICP 1 (32), ICP 2 (31), ICP 3 (19), ICP 4 (0), ICP 5 (0)
- **Signals:** 22 total (10 pain, 5 hiring, 7 growth), 15 matched to companies
- **Top scored:** CANTIERI S.p.A. (90), ALC Contracting (87), Safwan General (87)
- **Enrichment gap:** ~46% have both domain + LinkedIn (target was 60%+)
- **Master list:** `docs/sales/analysis/company-research/master-company-list.md`
- **Signals report:** `docs/sales/analysis/signals-session-1.md`
- **Session prompt:** `docs/sales/prompts/agent-team-session-1.md`

### Session 2 Priorities
1. Enrich ICP 2/3 Discovery Only companies (fill LinkedIn/domain/employee gaps → target 60%+)
2. Deep-dive UAE company-specific news (contract wins for ALC, Safwan, Rukin)
3. Brazil labor shortage / technology adoption angle for outreach hooks
4. Italy infrastructure companies in Terna EUR17.7B plan
5. Discover ICP 4 (Field Services) and ICP 5 (Facility Management) companies
6. Score all 81 companies with full pipeline rubric
7. Competitor monitoring: AlterSquare, Symterra (publishing content in our space)

---

## MISSION

You are executing a **signal-led B2B research pipeline** for ShiftBuddy, a WhatsApp-native field operations platform. Your job is to systematically discover and enrich target companies using Firecrawl web search and scraping.

**Pipeline status:** Session 1 complete. 81 companies across 3 ICPs and 5 geos. ICP 4/5 have zero companies. ~46% enrichment completeness. Next: enrich ICP 2/3, discover ICP 4/5, score all companies.

---

## SETUP

Before starting, verify Firecrawl is ready:

```bash
firecrawl --status
```

Create output directories:

```bash
mkdir -p .firecrawl/pipeline/{discovery,enrichment,signals}
mkdir -p .firecrawl/pipeline/discovery/{construction,equipment-rental,logistics,field-services,facility-management}
mkdir -p .firecrawl/pipeline/enrichment/{domains,linkedin,contacts}
mkdir -p .firecrawl/pipeline/signals/{pain,hiring,growth}
```

---

## PHASE 1: COMPANY DISCOVERY (Expand from 47 → 150+ companies)

### Priority Order
1. **Italy** (home market) — all 5 ICPs
2. **UK** — Construction + Equipment Rental + Field Services
3. **UAE/GCC** — Construction + Equipment Rental
4. **Brazil** — Construction + Logistics
5. **Sub-Saharan Africa** — Construction only

### Search Queries by ICP

Execute these searches IN PARALLEL (use `&` and `wait`). Adapt queries per geography and language.

#### ICP 1: Construction (30-150 employees, multi-site)

```bash
# Italy
firecrawl search "imprese edili Italia 50 100 dipendenti cantieri" --limit 20 --country IT -o .firecrawl/pipeline/discovery/construction/italy-search-1.json --json &
firecrawl search "costruzioni generali Italia PMI impresa edile media dimensione" --limit 20 --country IT -o .firecrawl/pipeline/discovery/construction/italy-search-2.json --json &
firecrawl search "site:linkedin.com/company impresa costruzioni Italia edilizia 51-200" --limit 20 -o .firecrawl/pipeline/discovery/construction/italy-linkedin-1.json --json &

# UK
firecrawl search "UK construction company 50 150 employees regional contractor" --limit 20 --country GB -o .firecrawl/pipeline/discovery/construction/uk-search-1.json --json &
firecrawl search "site:linkedin.com/company UK construction contractor 51-200 employees" --limit 20 -o .firecrawl/pipeline/discovery/construction/uk-linkedin-1.json --json &

# UAE/GCC
firecrawl search "Dubai construction company contractor 50 200 employees building" --limit 20 --country AE -o .firecrawl/pipeline/discovery/construction/gcc-search-1.json --json &
firecrawl search "Saudi Arabia construction contractor SME building company" --limit 20 --country SA -o .firecrawl/pipeline/discovery/construction/gcc-search-2.json --json &

# Brazil
firecrawl search "construtora Brasil 50 150 funcionários obras" --limit 20 --country BR -o .firecrawl/pipeline/discovery/construction/brazil-search-1.json --json &
firecrawl search "empresa construção civil Brasil médio porte" --limit 20 --country BR -o .firecrawl/pipeline/discovery/construction/brazil-search-2.json --json &

# Africa
firecrawl search "construction company Kenya Nigeria South Africa 50 employees contractor" --limit 20 -o .firecrawl/pipeline/discovery/construction/africa-search-1.json --json &
wait
```

#### ICP 2: Equipment Rental (50-200 employees, 200+ tracked assets)

```bash
# Italy
firecrawl search "noleggio attrezzature edili Italia azienda flotta" --limit 20 --country IT -o .firecrawl/pipeline/discovery/equipment-rental/italy-search-1.json --json &
firecrawl search "noleggio mezzi da cantiere Italia PMI piattaforme aeree escavatori" --limit 20 --country IT -o .firecrawl/pipeline/discovery/equipment-rental/italy-search-2.json --json &

# UK
firecrawl search "UK plant hire equipment rental company regional 50 200 employees" --limit 20 --country GB -o .firecrawl/pipeline/discovery/equipment-rental/uk-search-1.json --json &
firecrawl search "UK tool hire construction equipment rental fleet" --limit 20 --country GB -o .firecrawl/pipeline/discovery/equipment-rental/uk-search-2.json --json &

# UAE/GCC
firecrawl search "Dubai equipment rental construction machinery hire company" --limit 20 --country AE -o .firecrawl/pipeline/discovery/equipment-rental/gcc-search-1.json --json &

# Brazil
firecrawl search "locação equipamentos construção Brasil empresa aluguel máquinas" --limit 20 --country BR -o .firecrawl/pipeline/discovery/equipment-rental/brazil-search-1.json --json &
wait
```

#### ICP 3: Logistics & Last-Mile (30-150 employees, 50+ daily dispatches)

```bash
# Italy
firecrawl search "azienda logistica Italia corriere ultimo miglio PMI 50 dipendenti" --limit 20 --country IT -o .firecrawl/pipeline/discovery/logistics/italy-search-1.json --json &
firecrawl search "trasporti e logistica Italia flotta furgoni consegne" --limit 20 --country IT -o .firecrawl/pipeline/discovery/logistics/italy-search-2.json --json &

# UK
firecrawl search "UK last mile delivery company courier logistics 50 150 employees" --limit 20 --country GB -o .firecrawl/pipeline/discovery/logistics/uk-search-1.json --json &

# Brazil
firecrawl search "empresa logística Brasil entrega última milha frota" --limit 20 --country BR -o .firecrawl/pipeline/discovery/logistics/brazil-search-1.json --json &
wait
```

#### ICP 4: Field Services — HVAC, Plumbing, Electrical (20-100 employees)

```bash
# Italy
firecrawl search "impiantista Italia HVAC idraulico elettricista azienda 30 100 dipendenti" --limit 20 --country IT -o .firecrawl/pipeline/discovery/field-services/italy-search-1.json --json &
firecrawl search "manutenzione impianti Italia azienda PMI tecnici sul campo" --limit 20 --country IT -o .firecrawl/pipeline/discovery/field-services/italy-search-2.json --json &

# UK
firecrawl search "UK HVAC plumbing electrical company 30 100 employees field service" --limit 20 --country GB -o .firecrawl/pipeline/discovery/field-services/uk-search-1.json --json &
firecrawl search "UK mechanical electrical contractor regional M&E" --limit 20 --country GB -o .firecrawl/pipeline/discovery/field-services/uk-search-2.json --json &

# UAE
firecrawl search "Dubai HVAC maintenance company facilities field service 50 employees" --limit 20 --country AE -o .firecrawl/pipeline/discovery/field-services/gcc-search-1.json --json &
wait
```

#### ICP 5: Facility Management (40-200 employees, multi-site)

```bash
# Italy
firecrawl search "facility management Italia azienda pulizie manutenzione immobili PMI" --limit 20 --country IT -o .firecrawl/pipeline/discovery/facility-management/italy-search-1.json --json &

# UK
firecrawl search "UK facility management company cleaning maintenance 50 200 employees" --limit 20 --country GB -o .firecrawl/pipeline/discovery/facility-management/uk-search-1.json --json &

# UAE
firecrawl search "Dubai facility management company building maintenance services" --limit 20 --country AE -o .firecrawl/pipeline/discovery/facility-management/gcc-search-1.json --json &
wait
```

### Processing Discovery Results

After each batch, extract company names and URLs:

```bash
# Extract company names and URLs from search results
jq -r '.data.web[] | "\(.title) | \(.url)"' .firecrawl/pipeline/discovery/construction/italy-search-1.json
```

For each promising company found, record in a structured format:
- company_name
- country
- icp_vertical
- source_url
- linkedin_url (if found)
- website_domain
- employee_estimate
- discovery_date

---

## PHASE 2: DOMAIN & WEBSITE ENRICHMENT

### Priority: Fill gaps for existing 47 companies FIRST

**Italy — 15 companies missing websites:**
```bash
firecrawl search "Costruzioni Iannini S.r.l. L'Aquila sito web" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/iannini.json --json &
firecrawl search "CANTIERI S.p.A. Poggio Renatico sito web" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/cantieri.json --json &
firecrawl search "Impresa Percassi costruzioni sito web" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/percassi.json --json &
firecrawl search "Preve Costruzioni SpA Roccavione sito web" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/preve.json --json &
firecrawl search "FAST SRL Engineering Construction Italy website" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/fast-srl.json --json &
firecrawl search "ECOEDILE SRL costruzioni sito web" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/ecoedile.json --json &
wait
```

**UK — 8 companies missing LinkedIn URLs:**
```bash
firecrawl search "site:linkedin.com/company Clark Contracts construction" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/clark-contracts.json --json &
firecrawl search "site:linkedin.com/company Princebuild construction Peterborough" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/princebuild.json --json &
firecrawl search "site:linkedin.com/company C3 Construction Ltd Leicester" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/c3-construction.json --json &
wait
```

### For each newly discovered company

```bash
# Find domain
firecrawl search "[Company Name] [City] [Country] official website" --limit 5 -o .firecrawl/pipeline/enrichment/domains/[company-slug].json --json

# Find LinkedIn
firecrawl search "site:linkedin.com/company [Company Name] [Country]" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/[company-slug].json --json
```

### Scrape company websites for enrichment data

Once domains are found, scrape them for employee count, services, fleet size, locations:

```bash
firecrawl scrape "https://[company-domain.com]/about" --only-main-content -o .firecrawl/pipeline/enrichment/domains/[company-slug]-about.md &
firecrawl scrape "https://[company-domain.com]/chi-siamo" --only-main-content -o .firecrawl/pipeline/enrichment/domains/[company-slug]-chi-siamo.md &
firecrawl scrape "https://[company-domain.com]/services" --only-main-content -o .firecrawl/pipeline/enrichment/domains/[company-slug]-services.md &
wait
```

---

## PHASE 3: SIGNAL DETECTION

This is the **most valuable** and **completely untouched** part of the pipeline.

### Pain Signals (Critical — outreach triggers)

```bash
# Reddit — field operations pain
firecrawl search "site:reddit.com construction equipment tracking lost stolen tools WhatsApp chaos" --limit 20 --tbs qdr:m -o .firecrawl/pipeline/signals/pain/reddit-construction-1.json --json &
firecrawl search "site:reddit.com HVAC plumbing scheduling nightmare dispatch WhatsApp" --limit 20 --tbs qdr:m -o .firecrawl/pipeline/signals/pain/reddit-field-services-1.json --json &
firecrawl search "site:reddit.com fleet management spreadsheet nightmare equipment rental tracking" --limit 20 --tbs qdr:m -o .firecrawl/pipeline/signals/pain/reddit-rental-1.json --json &
firecrawl search "site:reddit.com logistics delivery dispatch coordination WhatsApp group chaos" --limit 20 --tbs qdr:m -o .firecrawl/pipeline/signals/pain/reddit-logistics-1.json --json &

# LinkedIn — pain signals
firecrawl search "site:linkedin.com construction equipment lost stolen tracking frustrating" --limit 20 --tbs qdr:m -o .firecrawl/pipeline/signals/pain/linkedin-pain-1.json --json &
firecrawl search "site:linkedin.com field operations WhatsApp coordination challenge inefficient" --limit 20 --tbs qdr:m -o .firecrawl/pipeline/signals/pain/linkedin-pain-2.json --json &
wait
```

### Hiring Signals (High — indicates scaling pain)

```bash
# Operations Manager / Fleet Coordinator / Logistics Supervisor hiring
firecrawl search "site:linkedin.com/jobs operations manager construction Italy 2026" --limit 20 -o .firecrawl/pipeline/signals/hiring/linkedin-ops-italy.json --json &
firecrawl search "site:linkedin.com/jobs fleet coordinator equipment rental UK" --limit 20 -o .firecrawl/pipeline/signals/hiring/linkedin-fleet-uk.json --json &
firecrawl search "site:linkedin.com/jobs dispatch manager logistics Dubai" --limit 20 -o .firecrawl/pipeline/signals/hiring/linkedin-dispatch-gcc.json --json &
firecrawl search "site:linkedin.com/jobs field service manager HVAC plumbing UK" --limit 20 -o .firecrawl/pipeline/signals/hiring/linkedin-field-uk.json --json &
wait
```

### Growth Signals (High — indicates imminent scaling)

```bash
# Company growth, new contracts, fleet expansion
firecrawl search "construction company expansion new contract 2026 Italy" --limit 20 --sources news --tbs qdr:m -o .firecrawl/pipeline/signals/growth/news-italy-1.json --json &
firecrawl search "construction company expansion new site UK 2026" --limit 20 --sources news --tbs qdr:m -o .firecrawl/pipeline/signals/growth/news-uk-1.json --json &
firecrawl search "construction company Dubai new project awarded 2026" --limit 20 --sources news --tbs qdr:m -o .firecrawl/pipeline/signals/growth/news-gcc-1.json --json &
wait
```

---

## PHASE 4: PROCESSING & SCORING

After collecting data, process results into the master company list.

### For each company, compute ICP Score (max 100):

| Category | Signal | Points |
|----------|--------|--------|
| **Signal Strength** (max 30) | Critical pain signal found | 30 |
| | Hiring/growth signal found | 20 |
| | Warm/contextual signal | 10 |
| **Industry Fit** (max 25) | Construction / Equipment Rental | 25 |
| | Logistics / Field Services | 20 |
| | Other with field teams | 10 |
| **Company Size** (max 20) | 30-150 employees | 20 |
| | 20-29 or 151-200 employees | 15 |
| | Other (but field team >10) | 5 |
| **Geography** (max 15) | Tier 1 (IT, UK, UAE, SA) | 15 |
| | Tier 2 (BR, ZA, KE, NG) | 10 |
| | Other | 3 |
| **Tech Stack** (max 10) | No tools / WhatsApp-dependent | 10 |
| | Spreadsheets / basic tools | 7 |
| | Lightweight competitor | 3 |
| | Enterprise solution (DISQUALIFY) | 0 |

### Disqualification Criteria (REMOVE from pipeline)

- Fewer than 15 employees
- More than 500 employees
- Uses SAP, Oracle, Samsara, Verizon Connect, ServiceTitan, Jobber
- No field operations (office-only)
- Primary comms on Slack/Teams (not WhatsApp)

---

## PHASE 5: OUTPUT FORMAT

For each qualified company, produce this structured record:

```
COMPANY: [Name]
━━━━━━━━━━━━━━━━━━
ICP: [1-Construction / 2-Rental / 3-Logistics / 4-Field / 5-FM]
GEOGRAPHY: [Country] (Tier [1/2])
EMPLOYEES: [count or range]
DOMAIN: [website.com]
LINKEDIN: [URL]

SIGNAL:
  Type: [Pain / Hiring / Growth / Warm]
  Strength: [Critical 🔴 / High 🟠 / Warm 🟡]
  Source: [URL]
  Detail: "[exact quote or description]"

ICP SCORE: [0-100]
TIER: [A (75-100) / B (50-74) / C (25-49)]

BUYER PERSONAS:
  Primary: [Title — Name if found]
  Secondary: [Title — Name if found]

OUTREACH:
  Sequence: [A (Pain) / B (Growth) / C (Cold ICP)]
  Hook: "[2-3 sentence personalization based on signal]"
  Language: [EN / IT / PT / AR]

STATUS: [Discovery / Enriched / Signal-Qualified / Outreach-Ready]
━━━━━━━━━━━━━━━━━━
```

---

## EXECUTION PRIORITIES

### Session 1 — COMPLETED (2026-02-16)
- ~~Run Phase 2 enrichment for existing 35 companies~~ Done: 25 enriched, 3 DQ'd
- ~~Run Phase 1 discovery for ICP 2 (Equipment Rental) + ICP 3 (Logistics)~~ Done: 49 new companies
- ~~Run Phase 3 signal detection on Reddit + LinkedIn + news~~ Done: 22 signals detected

### Session 2 (NEXT — highest ROI)
1. Run Phase 2 enrichment for 49 new ICP 2/3 companies (fill domain + LinkedIn + employee gaps)
2. Run Phase 1 discovery for ICP 4 (Field Services) + ICP 5 (Facility Management)
3. Run Phase 3 deep-dive signals: UAE company-specific news, Brazil tech adoption, Italy Terna plan
4. Run Phase 4 scoring for all 81 companies
5. Cross-reference new signals with existing companies

### Session 3
1. Complete Phase 4 scoring + disqualification for all companies
2. Run Phase 3 company-specific signals for top 20 Tier A prospects
3. Begin Phase 5 output generation for all Tier A companies (outreach-ready records)
4. Competitor intelligence: AlterSquare, Symterra content monitoring

### Target: After 3 sessions
- 150+ companies discovered (across all 5 ICPs)
- 80%+ enrichment completeness (domains + LinkedIn)
- 30+ companies with active signals detected
- 20+ companies scored Tier A (75-100)
- 10+ companies fully outreach-ready with personalized hooks

---

## RULES

1. **Always save to files** — never dump Firecrawl output into context. Use `-o` flag.
2. **Run parallel** — use `&` and `wait` for all independent searches.
3. **Adapt language** — use Italian for Italy, Portuguese for Brazil, English for UK/GCC/Africa.
4. **Source everything** — every signal must have a URL. No signal, no lead.
5. **Disqualify fast** — check size + industry before enriching. Don't waste credits on bad fits.
6. **Update master list** — after each session, consolidate findings into `docs/sales/analysis/company-research/master-company-list.md`.
7. **Track progress** — log which queries ran, which companies were found, which need enrichment.
