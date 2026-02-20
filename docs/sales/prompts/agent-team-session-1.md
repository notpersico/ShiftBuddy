# Agent Team Prompt — Pipeline Session 1

> **Purpose:** Launch a 3-agent team to execute the first session of the ShiftBuddy Firecrawl research pipeline.
> **Prerequisite:** Firecrawl CLI authenticated (`firecrawl --status` shows green).
> **Outputs:** All Firecrawl results saved to `.firecrawl/pipeline/`, master company list updated at `docs/sales/analysis/company-research/master-company-list.md`.

---

## How to Run

Paste the following into a Claude Code session with Firecrawl installed:

```
Create a team called "pipeline-session-1" with 3 agents running in parallel. Read docs/sales/prompts/agent-team-session-1.md for full instructions. Each agent has a distinct mission — launch all 3 simultaneously:

1. "enrichment-agent" — Enrich existing 35 companies (fill missing domains, LinkedIn URLs, employee counts)
2. "discovery-agent" — Discover new companies for ICP 2 (Equipment Rental) and ICP 3 (Logistics) across all geographies
3. "signals-agent" — Detect pain, hiring, and growth signals on Reddit, LinkedIn, and news

After all 3 agents complete, consolidate all findings into docs/sales/analysis/company-research/master-company-list.md.
```

---

## Shared Context (All Agents Must Read)

### Product
ShiftBuddy is a WhatsApp-based ops task manager and asset control system for field teams. No app install for crews — everything works via WhatsApp Business API. Web dashboard for managers.

### Target ICPs (priority order)
1. **Construction** (30-150 emp, 3-10 active sites) — PRIMARY
2. **Equipment Rental** (50-200 emp, 200+ tracked assets) — PRIMARY
3. **Logistics & Last-Mile** (30-150 emp, 50+ daily dispatches) — SECONDARY
4. **Field Services / HVAC / Plumbing** (20-100 emp, van-based) — SECONDARY
5. **Facility Management** (40-200 emp, multi-site) — TERTIARY

### Target Geographies (priority order)
1. Italy (home market) — search in Italian
2. UK — search in English
3. UAE + Saudi Arabia — search in English
4. Brazil — search in Portuguese
5. Sub-Saharan Africa (South Africa, Kenya, Nigeria) — search in English

### Disqualification Criteria (skip immediately)
- Fewer than 15 employees
- More than 500 employees
- Uses SAP, Oracle, Samsara, Verizon Connect, ServiceTitan, Jobber (enterprise tools)
- No field operations (office-only company)
- Government agency or public utility

### Firecrawl Rules
- **Always use `-o` flag** — never dump output into context
- **Run searches in parallel** — use `&` and `wait`
- **Save to `.firecrawl/pipeline/`** subdirectories
- **Never read entire output files** — use `jq`, `head`, `grep` to extract what you need
- **Quote all URLs** in shell commands

---

## Agent 1: Enrichment Agent

**Name:** `enrichment-agent`
**Mission:** Fill data gaps for the 35 qualified companies in the master list.

### Step 1: Read the master list

```bash
Read docs/sales/analysis/company-research/master-company-list.md
```

Identify companies with missing data:
- Missing website/domain → search for it
- Missing LinkedIn URL → search for it
- Employee count = "TBD" → search for it
- Status = "Discovery Only" → upgrade to "Partial" or "Enriched"

### Step 2: Search for missing domains (Italy — 6+ companies missing websites)

```bash
firecrawl search "Impresa Percassi costruzioni Italia sito ufficiale" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/percassi.json --json &
firecrawl search "Preve Costruzioni SpA Roccavione sito web" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/preve.json --json &
firecrawl search "FAST SRL Engineering Construction Italy website" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/fast-srl.json --json &
firecrawl search "ECOEDILE SRL costruzioni sito web" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/ecoedile.json --json &
firecrawl search "Petas Srl Colognola ai Colli costruzioni sito web" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/petas.json --json &
firecrawl search "Costruzioni Edili Baraldini Quirino SpA sito web" --limit 5 --country IT -o .firecrawl/pipeline/enrichment/domains/baraldini.json --json &
wait
```

### Step 3: Search for missing LinkedIn URLs (UK — 3 companies)

```bash
firecrawl search "site:linkedin.com/company Princebuild construction Peterborough" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/princebuild.json --json &
firecrawl search "site:linkedin.com/company Clark Contracts construction Scotland" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/clark-contracts.json --json &
firecrawl search "site:linkedin.com/company C3 Construction Ltd Leicester" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/c3-construction.json --json &
wait
```

### Step 4: Search for missing LinkedIn URLs (GCC — 6 companies)

```bash
firecrawl search "site:linkedin.com/company Gamma Contracting LLC UAE" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/gamma-contracting.json --json &
firecrawl search "site:linkedin.com/company Square General Contracting UAE" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/square-general.json --json &
firecrawl search "site:linkedin.com/company Condor Group construction UAE" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/condor-group.json --json &
firecrawl search "site:linkedin.com/company Etlad construction UAE" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/etlad.json --json &
firecrawl search "site:linkedin.com/company General Construction Co LLC UAE" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/general-construction-uae.json --json &
wait
```

### Step 5: Search for missing LinkedIn/employee data (Africa — 4 companies)

```bash
firecrawl search "site:linkedin.com/company Integrum Construction Kenya" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/integrum.json --json &
firecrawl search "site:linkedin.com/company Star General Contractors Kenya" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/star-general-kenya.json --json &
firecrawl search "site:linkedin.com/company Times Construction Nigeria" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/times-construction.json --json &
firecrawl search "site:linkedin.com/company SABC Construction Nigeria" --limit 3 -o .firecrawl/pipeline/enrichment/linkedin/sabc-construction.json --json &
wait
```

### Step 6: Scrape company websites for enrichment

For each company where you found a domain, scrape their About page:

```bash
firecrawl scrape "https://[domain]/about" --only-main-content -o .firecrawl/pipeline/enrichment/domains/[slug]-about.md &
firecrawl scrape "https://[domain]/chi-siamo" --only-main-content -o .firecrawl/pipeline/enrichment/domains/[slug]-chi-siamo.md &
# ... repeat for all discovered domains
wait
```

Extract: employee count, number of sites/locations, fleet size, services offered, years in business.

### Step 7: Update master company list

Edit `docs/sales/analysis/company-research/master-company-list.md`:
- Fill in discovered domains, LinkedIn URLs, employee counts
- Update Status column: "Discovery Only" → "Partial" or "Enriched"
- Re-evaluate ICP Tier if employee count changes the fit
- Add enrichment notes to Notes column

### Output

Send a message to team lead with:
- How many companies enriched (domains found, LinkedIn found, employee counts confirmed)
- Companies that couldn't be enriched (no web presence found)
- Any tier changes (e.g., company moved from Tier C to Tier A after confirming employee count)

---

## Agent 2: Discovery Agent

**Name:** `discovery-agent`
**Mission:** Discover 40+ new companies for ICP 2 (Equipment Rental) and ICP 3 (Logistics) across all priority geographies.

### Step 1: ICP 2 — Equipment Rental Discovery

```bash
# Italy
firecrawl search "noleggio attrezzature edili Italia azienda flotta" --limit 20 --country IT -o .firecrawl/pipeline/discovery/equipment-rental/italy-1.json --json &
firecrawl search "noleggio mezzi da cantiere Italia piattaforme aeree escavatori PMI" --limit 20 --country IT -o .firecrawl/pipeline/discovery/equipment-rental/italy-2.json --json &
firecrawl search "site:linkedin.com/company noleggio attrezzature edili Italia 51-200" --limit 15 -o .firecrawl/pipeline/discovery/equipment-rental/italy-linkedin.json --json &

# UK
firecrawl search "UK plant hire equipment rental company regional 50 200 employees" --limit 20 --country GB -o .firecrawl/pipeline/discovery/equipment-rental/uk-1.json --json &
firecrawl search "UK tool hire construction equipment rental fleet management" --limit 20 --country GB -o .firecrawl/pipeline/discovery/equipment-rental/uk-2.json --json &
firecrawl search "site:linkedin.com/company plant hire UK 51-200 employees" --limit 15 -o .firecrawl/pipeline/discovery/equipment-rental/uk-linkedin.json --json &

# UAE/GCC
firecrawl search "Dubai equipment rental construction machinery hire company" --limit 20 --country AE -o .firecrawl/pipeline/discovery/equipment-rental/gcc-1.json --json &
firecrawl search "Saudi Arabia heavy equipment rental company fleet" --limit 15 --country SA -o .firecrawl/pipeline/discovery/equipment-rental/gcc-2.json --json &

# Brazil
firecrawl search "locação equipamentos construção Brasil empresa aluguel máquinas" --limit 20 --country BR -o .firecrawl/pipeline/discovery/equipment-rental/brazil-1.json --json &
firecrawl search "locadora de equipamentos pesados Brasil PME frota" --limit 15 --country BR -o .firecrawl/pipeline/discovery/equipment-rental/brazil-2.json --json &
wait
```

### Step 2: ICP 3 — Logistics & Last-Mile Discovery

```bash
# Italy
firecrawl search "azienda logistica Italia corriere ultimo miglio PMI 50 dipendenti" --limit 20 --country IT -o .firecrawl/pipeline/discovery/logistics/italy-1.json --json &
firecrawl search "trasporti e logistica Italia flotta furgoni consegne espresse" --limit 20 --country IT -o .firecrawl/pipeline/discovery/logistics/italy-2.json --json &

# UK
firecrawl search "UK last mile delivery company courier logistics 50 150 employees" --limit 20 --country GB -o .firecrawl/pipeline/discovery/logistics/uk-1.json --json &
firecrawl search "UK same day delivery courier fleet regional" --limit 15 --country GB -o .firecrawl/pipeline/discovery/logistics/uk-2.json --json &

# Brazil
firecrawl search "empresa logística Brasil entrega última milha frota 50 funcionários" --limit 20 --country BR -o .firecrawl/pipeline/discovery/logistics/brazil-1.json --json &
firecrawl search "transportadora regional Brasil PME frota própria" --limit 15 --country BR -o .firecrawl/pipeline/discovery/logistics/brazil-2.json --json &

# UAE
firecrawl search "Dubai logistics company last mile delivery fleet 50 employees" --limit 15 --country AE -o .firecrawl/pipeline/discovery/logistics/gcc-1.json --json &
wait
```

### Step 3: Process Results

For each search result file:

```bash
jq -r '.data.web[] | "\(.title) | \(.url) | \(.description)"' .firecrawl/pipeline/discovery/equipment-rental/italy-1.json
```

For each company found:
1. Check it's not already in the master list (avoid duplicates)
2. Check against disqualification criteria
3. Record: name, country, ICP, source URL, website, employee estimate

### Step 4: Quick LinkedIn verification for top finds

For the most promising companies (clear ICP fit from description), verify on LinkedIn:

```bash
firecrawl search "site:linkedin.com/company [Company Name]" --limit 3 -o .firecrawl/pipeline/discovery/[icp]/[slug]-linkedin.json --json
```

### Step 5: Add to master company list

Append new companies to `docs/sales/analysis/company-research/master-company-list.md`:
- Add to the correct geography section
- Use the same table schema: Company | LinkedIn URL | Website | Employees | HQ | Founded | ICP Fit | Status | Notes
- Set ICP Fit based on initial assessment (Tier A/B/C)
- Set Status to "Discovery Only" or "Partial" depending on data quality
- Update the Pipeline Summary table at the top with new counts

### Output

Send a message to team lead with:
- Total new companies discovered per ICP and geography
- Top 10 highest-confidence finds (most likely Tier A)
- Any surprising finds (companies in unexpected verticals or geographies)
- Updated pipeline summary counts

---

## Agent 3: Signals Agent

**Name:** `signals-agent`
**Mission:** Detect pain, hiring, and growth signals that can be matched to companies in the pipeline. This is the most valuable untouched part of the pipeline.

### Step 1: Pain Signals — Reddit

Search Reddit for field ops pain that matches ShiftBuddy's value prop:

```bash
# Construction pain
firecrawl search "site:reddit.com construction equipment tracking lost stolen tools WhatsApp" --limit 20 --tbs qdr:y -o .firecrawl/pipeline/signals/pain/reddit-construction-tools.json --json &
firecrawl search "site:reddit.com construction site coordination scheduling nightmare crew management" --limit 20 --tbs qdr:y -o .firecrawl/pipeline/signals/pain/reddit-construction-mgmt.json --json &
firecrawl search "site:reddit.com r/Construction equipment stolen job site tracking" --limit 20 --tbs qdr:y -o .firecrawl/pipeline/signals/pain/reddit-construction-theft.json --json &

# Equipment rental pain
firecrawl search "site:reddit.com equipment rental tracking spreadsheet nightmare fleet management" --limit 20 --tbs qdr:y -o .firecrawl/pipeline/signals/pain/reddit-rental-tracking.json --json &
firecrawl search "site:reddit.com plant hire tool rental lost damaged dispute return" --limit 20 --tbs qdr:y -o .firecrawl/pipeline/signals/pain/reddit-rental-disputes.json --json &

# Field services pain
firecrawl search "site:reddit.com HVAC plumbing scheduling dispatch WhatsApp field service management" --limit 20 --tbs qdr:y -o .firecrawl/pipeline/signals/pain/reddit-field-services.json --json &
firecrawl search "site:reddit.com r/sweatystartup crew management scheduling tool van inventory" --limit 20 --tbs qdr:y -o .firecrawl/pipeline/signals/pain/reddit-sweaty-startup.json --json &

# Logistics pain
firecrawl search "site:reddit.com logistics delivery dispatch coordination WhatsApp group chaos driver" --limit 20 --tbs qdr:y -o .firecrawl/pipeline/signals/pain/reddit-logistics-dispatch.json --json &
wait
```

### Step 2: Pain Signals — LinkedIn

```bash
firecrawl search "site:linkedin.com construction equipment lost stolen tracking frustrating field" --limit 15 --tbs qdr:m -o .firecrawl/pipeline/signals/pain/linkedin-equipment-pain.json --json &
firecrawl search "site:linkedin.com field operations WhatsApp coordination challenge inefficient crew" --limit 15 --tbs qdr:m -o .firecrawl/pipeline/signals/pain/linkedin-whatsapp-pain.json --json &
firecrawl search "site:linkedin.com asset tracking spreadsheet construction fleet nightmare manual" --limit 15 --tbs qdr:m -o .firecrawl/pipeline/signals/pain/linkedin-spreadsheet-pain.json --json &
wait
```

### Step 3: Hiring Signals — LinkedIn Jobs

```bash
# Operations/fleet/logistics management roles = scaling pain
firecrawl search "site:linkedin.com/jobs operations manager construction Italy 2026" --limit 15 -o .firecrawl/pipeline/signals/hiring/linkedin-ops-italy.json --json &
firecrawl search "site:linkedin.com/jobs fleet coordinator equipment rental UK 2026" --limit 15 -o .firecrawl/pipeline/signals/hiring/linkedin-fleet-uk.json --json &
firecrawl search "site:linkedin.com/jobs dispatch manager logistics Dubai 2026" --limit 15 -o .firecrawl/pipeline/signals/hiring/linkedin-dispatch-gcc.json --json &
firecrawl search "site:linkedin.com/jobs field service manager HVAC plumbing UK 2026" --limit 15 -o .firecrawl/pipeline/signals/hiring/linkedin-field-uk.json --json &
firecrawl search "site:linkedin.com/jobs responsabile cantiere operativo Italia 2026" --limit 15 -o .firecrawl/pipeline/signals/hiring/linkedin-cantiere-italy.json --json &
firecrawl search "site:linkedin.com/jobs gerente operações construção Brasil 2026" --limit 15 -o .firecrawl/pipeline/signals/hiring/linkedin-ops-brazil.json --json &
wait
```

### Step 4: Growth Signals — News

```bash
firecrawl search "construction company expansion new contract awarded 2026 Italy" --limit 15 --sources news --tbs qdr:m -o .firecrawl/pipeline/signals/growth/news-italy.json --json &
firecrawl search "construction company new site office expansion UK 2026" --limit 15 --sources news --tbs qdr:m -o .firecrawl/pipeline/signals/growth/news-uk.json --json &
firecrawl search "construction company Dubai new project awarded contract 2026" --limit 15 --sources news --tbs qdr:m -o .firecrawl/pipeline/signals/growth/news-gcc.json --json &
firecrawl search "equipment rental company fleet expansion acquisition 2026" --limit 15 --sources news --tbs qdr:m -o .firecrawl/pipeline/signals/growth/news-rental-global.json --json &
firecrawl search "construtora Brasil novo contrato expansão projeto 2026" --limit 15 --sources news --tbs qdr:m -o .firecrawl/pipeline/signals/growth/news-brazil.json --json &
wait
```

### Step 5: Scrape high-value signal posts

For the top 10 most relevant Reddit threads or LinkedIn posts found, scrape the full content:

```bash
firecrawl scrape "[reddit-thread-url]" --only-main-content -o .firecrawl/pipeline/signals/pain/[slug]-full.md &
# ... repeat for top finds
wait
```

### Step 6: Process and Classify Signals

For each signal found, classify it:

```
SIGNAL:
  Type: [Pain / Hiring / Growth]
  Strength: [Critical / High / Warm]
  Source: [URL]
  Platform: [Reddit / LinkedIn / News]
  ICP Match: [1-Construction / 2-Rental / 3-Logistics / 4-Field / 5-FM]
  Geography: [Country]
  Company Match: [Company name from master list, or "No direct match"]
  Quote: "[Key excerpt]"
  Outreach Hook: "[2-3 sentence hook for outreach based on this signal]"
```

### Step 7: Create signals summary

Write a signals summary file at `docs/sales/analysis/signals-session-1.md` containing:
- All classified signals organized by type (Pain → Hiring → Growth)
- Signals matched to specific companies in the master list
- Unmatched signals that suggest new companies to discover
- Top 5 strongest signals with full context and outreach hooks

### Output

Send a message to team lead with:
- Total signals found by type and strength
- How many signals match existing companies in the master list
- Top 5 actionable signals with company + outreach hook
- Recommendations for Session 2 signal searches (what worked, what to refine)

---

## Team Lead: Consolidation (After All Agents Complete)

Once all 3 agents finish:

1. **Verify master company list** — check that enrichment-agent and discovery-agent updates don't conflict
2. **Cross-reference signals** — match signals-agent findings to specific companies in the master list
3. **Score companies** — apply the ICP scoring rubric to all companies with sufficient data:

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
| **Tech Stack** (max 10) | No tools / WhatsApp-dependent | 10 |
| | Spreadsheets / basic tools | 7 |
| | Lightweight competitor | 3 |

4. **Update pipeline summary** — refresh the counts table at the top of master-company-list.md
5. **Report results** — summarize what was accomplished in Session 1

### Session 1 Success Criteria
- [ ] 70+ total companies in master list (up from 35)
- [ ] 60%+ companies have both domain and LinkedIn URL
- [ ] 15+ active signals detected (pain + hiring + growth)
- [ ] 5+ companies with signals matched → Tier A candidates
- [ ] ICP 2 (Equipment Rental) has 15+ companies
- [ ] ICP 3 (Logistics) has 10+ companies
