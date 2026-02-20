# Agent Team Prompt — Pipeline Session 2

> **Purpose:** Launch a 3-agent team for the second research session. Focus: enrich ICP 2/3 companies, discover ICP 4/5, and deep-dive signals on top-scored companies.
> **Prerequisite:** Session 1 completed. Firecrawl CLI authenticated.
> **Inputs:** `docs/sales/analysis/company-research/master-company-list.md` (81 qualified), `docs/sales/analysis/signals-session-1.md` (22 signals)
> **Outputs:** Updated master list, `docs/sales/analysis/signals-session-2.md`, all companies scored.

---

## How to Run

Paste the following into a Claude Code session with Firecrawl installed:

```
use firecrawl skill, Create a team called "pipeline-session-2" with 3 agents running in parallel.
Read docs/sales/prompts/agent-team-session-2.md for full instructions.

Session 2 success criteria:

- 110+ total companies (up from 81)
- 60%+ enrichment completeness (both domain + LinkedIn URL)
- ICP 4 (Field Services) populated for the first time (15+ companies)
- ICP 5 (Facility Management) populated for the first time (10+ companies)
- All companies scored with pipeline rubric
- 15+ company-specific signals detected
- 15+ Tier A companies identified
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
- Use the `firecrawl` Skill tool for ALL web searches and scraping
- NEVER use WebFetch or WebSearch — ONLY use the firecrawl skill
- Save outputs to `.firecrawl/pipeline/session-2/` subdirectories
- Run searches in parallel when possible

### Pipeline Scoring Rubric (apply to all companies)

| Category | Signal | Points |
|----------|--------|--------|
| **Signal Strength** (max 30) | Critical pain signal found | 30 |
| | Hiring/growth signal found | 20 |
| | Warm/contextual signal | 10 |
| | No signal | 0 |
| **Industry Fit** (max 25) | Construction / Equipment Rental | 25 |
| | Logistics / Field Services | 20 |
| | Facility Management / Other with field teams | 10 |
| **Company Size** (max 20) | 30-150 employees | 20 |
| | 20-29 or 151-200 employees | 15 |
| | 201-500 (borderline) | 5 |
| | TBD / Unknown | 0 |
| **Geography** (max 15) | Tier 1 (IT, UK, UAE, SA) | 15 |
| | Tier 2 (BR, ZA, KE, NG) | 10 |
| **Tech Stack** (max 10) | No tools / WhatsApp-dependent | 10 |
| | Spreadsheets / basic tools | 7 |
| | Lightweight competitor | 3 |
| | Unknown | 5 |

**Tier thresholds:** A (75-100), B (50-74), C (25-49)

---

## Agent 1: Enrichment Agent

**Name:** `enrichment-agent`
**Mission:** Fill data gaps for the 49 ICP 2/3 "Discovery Only" companies + remaining TBD fields from Session 1. Target: 60%+ of all companies have both domain AND LinkedIn URL.

### Current gap analysis

**Companies missing LinkedIn URL (20):**
- Italy: Impresa Percassi, Nolotecnica, CMI Noleggio, Edimservizi
- UK: Ardent Hire Solutions, Eagle Plant, Stennetts, Stevens Equipment Rental
- GCC: QER Plant Hire, AMHEC, Arabian Equipment & Machinery Rental, Easy Equipment Rental
- Brazil: Satel Locacao, Murici Logistica, Kingdom Transportes
- Africa: Star General Contractors (Kenya), Times Construction (Nigeria), 313 Plant Hire, East Africa Plant Hire, Transeast Limited

**Companies missing Website (24):**
- Italy: CANTIERI S.p.A., Costruzioni Iannini S.r.l., Liccardi Express, BLL Express
- UK: Smiths Hire, Jarvie Plant, Multi-Hire Power Tools, Rabbits Vehicle Hire, Stevens Equipment Rental, Absolutely Couriers, Speed Couriers, Danzen Group
- GCC: Ten World Contracting, Emirates Star, Access Rental Gulf, One Click Delivery
- Brazil: Connor Engenharia, AD Construtora, NSG Engineering, Concreto Locadora, Grupo ML Locacao, Estaf Equipamentos, Smolka Transportes
- Africa: Bauhaus International, Estim Construction

**Companies with TBD employee count (30):**
- Italy: Baraldini Quirino, FAST SRL, ECOEDILE, Nolotecnica, CMI, Edimservizi, Liccardi, BLL, NTL
- UK: Eagle Plant, Stennetts, Ascendex, Speed Couriers, Danzen
- GCC: Biston, Gamma, Etlad, QER, AMHEC, Arabian Equipment, Easy Equipment, Al Nasheet, One Click, Jeebly
- Brazil: Concreto, Grupo ML, Estaf, Kingdom, Smolka
- Africa: Bauhaus, SANA GROUP, CaratConcept, Integrum, Star General, Times, SABC, 313 Plant, East Africa Plant, Transeast

### Step 1: Read the master list
```bash
Read docs/sales/analysis/company-research/master-company-list.md
```

### Step 2: Search for missing LinkedIn URLs (priority: ICP 2/3 companies)

```bash
# Italy
firecrawl search "site:linkedin.com/company Impresa Percassi costruzioni" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/percassi.json --json &
firecrawl search "site:linkedin.com/company Nolotecnica noleggio Italia" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/nolotecnica.json --json &
firecrawl search "site:linkedin.com/company CMI Noleggio" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/cmi.json --json &
firecrawl search "site:linkedin.com/company Edimservizi" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/edimservizi.json --json &

# UK
firecrawl search "site:linkedin.com/company Ardent Hire Solutions plant" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/ardent.json --json &
firecrawl search "site:linkedin.com/company Eagle Plant hire UK" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/eagle-plant.json --json &
firecrawl search "site:linkedin.com/company Stennetts plant hire Surrey" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/stennetts.json --json &
firecrawl search "site:linkedin.com/company Stevens Equipment Rental" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/stevens.json --json &
wait

# GCC
firecrawl search "site:linkedin.com/company QER Plant Hire UAE" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/qer.json --json &
firecrawl search "site:linkedin.com/company AMHEC Arabian Machinery Saudi" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/amhec.json --json &
firecrawl search "site:linkedin.com/company Arabian Equipment Machinery Rental UAE" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/arabian-equipment.json --json &
firecrawl search "site:linkedin.com/company Easy Equipment Rental Dubai" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/easy-equipment.json --json &

# Brazil
firecrawl search "site:linkedin.com/company Satel Locacao Belo Horizonte" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/satel.json --json &
firecrawl search "site:linkedin.com/company Murici Logistica Brasil" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/murici.json --json &
firecrawl search "site:linkedin.com/company Kingdom Transportes Brasil" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/kingdom.json --json &

# Africa
firecrawl search "site:linkedin.com/company 313 Plant Hire South Africa" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/313-plant.json --json &
firecrawl search "site:linkedin.com/company East Africa Plant Hire" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/ea-plant.json --json &
firecrawl search "site:linkedin.com/company Transeast Limited logistics" --limit 3 -o .firecrawl/pipeline/session-2/enrichment/linkedin/transeast.json --json &
wait
```

### Step 3: Search for missing websites

```bash
# UK companies
firecrawl search "Smiths Equipment Hire official website UK" --limit 5 --country GB -o .firecrawl/pipeline/session-2/enrichment/domains/smiths.json --json &
firecrawl search "Jarvie Plant Ltd Scotland official website" --limit 5 --country GB -o .firecrawl/pipeline/session-2/enrichment/domains/jarvie.json --json &
firecrawl search "Multi-Hire Power Tools Limited website" --limit 5 --country GB -o .firecrawl/pipeline/session-2/enrichment/domains/multi-hire.json --json &
firecrawl search "Rabbits Vehicle Hire Reading website" --limit 5 --country GB -o .firecrawl/pipeline/session-2/enrichment/domains/rabbits.json --json &
firecrawl search "Stevens Equipment Rental Ltd website UK" --limit 5 --country GB -o .firecrawl/pipeline/session-2/enrichment/domains/stevens.json --json &
firecrawl search "Absolutely Couriers London official website" --limit 5 --country GB -o .firecrawl/pipeline/session-2/enrichment/domains/absolutely.json --json &

# Italy (CANTIERI and Iannini still missing websites)
firecrawl search "CANTIERI S.p.A. Poggio Renatico sito ufficiale" --limit 5 --country IT -o .firecrawl/pipeline/session-2/enrichment/domains/cantieri.json --json &
firecrawl search "Costruzioni Iannini S.r.l. L'Aquila sito web" --limit 5 --country IT -o .firecrawl/pipeline/session-2/enrichment/domains/iannini.json --json &
firecrawl search "Liccardi Express Courier sito web Italia" --limit 5 --country IT -o .firecrawl/pipeline/session-2/enrichment/domains/liccardi.json --json &

# Brazil
firecrawl search "Connor Engenharia Montes Claros site oficial" --limit 5 --country BR -o .firecrawl/pipeline/session-2/enrichment/domains/connor.json --json &
firecrawl search "AD Construtora Salvador Bahia site oficial" --limit 5 --country BR -o .firecrawl/pipeline/session-2/enrichment/domains/ad-construtora.json --json &
firecrawl search "Concreto Locadora equipamentos site" --limit 5 --country BR -o .firecrawl/pipeline/session-2/enrichment/domains/concreto.json --json &
wait
```

### Step 4: Scrape discovered websites for employee count + enrichment data

For each company where you find or already have a domain, scrape their About/Chi-Siamo/Quem-Somos page to extract: employee count, services, fleet size, locations, years in business.

```bash
# Template — repeat for all discovered domains
firecrawl scrape "https://[domain]/about" --only-main-content -o .firecrawl/pipeline/session-2/enrichment/about/[slug].md &
```

### Step 5: Verify employee counts for TBD companies

For companies where website scrape doesn't reveal employee count, search directly:

```bash
firecrawl search "[Company Name] employees number of staff team size" --limit 5 -o .firecrawl/pipeline/session-2/enrichment/employees/[slug].json --json
```

### Step 6: Update master company list

Edit `docs/sales/analysis/company-research/master-company-list.md`:
- Fill in discovered domains, LinkedIn URLs, employee counts
- Update Status: "Discovery Only" → "Partial" or "Enriched"
- Re-evaluate ICP Tier if employee count changes the fit (e.g., Gamma Contracting may be >500 → DQ)
- DQ any companies confirmed outside ICP range

### Output

Send a message to team-lead with:
- How many companies enriched (LinkedIn found, domains found, employee counts confirmed)
- Updated enrichment completeness percentage (companies with both domain + LinkedIn / total)
- Companies that couldn't be enriched (no web presence found)
- Any tier changes or DQs

---

## Agent 2: Discovery Agent

**Name:** `discovery-agent`
**Mission:** Discover 30+ new companies for ICP 4 (Field Services) and ICP 5 (Facility Management) across all priority geographies.

### ICP 4: Field Services / HVAC / Plumbing / Electrical
- **Profile:** 20-100 employees, 10-50 daily job dispatches, van-based operations
- **Pain:** tool/parts shrinkage, job visibility gap, customer disputes, owner scaling from 30 to 100 without back-office collapse
- **Buyer:** Owner/GM, Operations Manager

### ICP 5: Facility Management
- **Profile:** 40-200 employees, managing multiple client sites (offices, malls, hospitals)
- **Pain:** SLA compliance risk, asset rotation chaos across client sites, onboarding friction for new hires
- **Buyer:** Operations Director, Contract/Client Manager

### Step 1: Read the current master company list
```bash
Read docs/sales/analysis/company-research/master-company-list.md
```

### Step 2: ICP 4 — Field Services Discovery

```bash
# Italy
firecrawl search "impiantista Italia HVAC idraulico azienda 30 100 dipendenti manutenzione" --limit 20 --country IT -o .firecrawl/pipeline/session-2/discovery/field-services/italy-1.json --json &
firecrawl search "manutenzione impianti Italia azienda PMI tecnici sul campo furgoni" --limit 20 --country IT -o .firecrawl/pipeline/session-2/discovery/field-services/italy-2.json --json &
firecrawl search "site:linkedin.com/company manutenzione impianti Italia HVAC elettricista 11-50 51-200" --limit 15 -o .firecrawl/pipeline/session-2/discovery/field-services/italy-linkedin.json --json &

# UK
firecrawl search "UK HVAC plumbing company 30 100 employees field service maintenance" --limit 20 --country GB -o .firecrawl/pipeline/session-2/discovery/field-services/uk-1.json --json &
firecrawl search "UK mechanical electrical contractor regional M&E maintenance 50 employees" --limit 20 --country GB -o .firecrawl/pipeline/session-2/discovery/field-services/uk-2.json --json &
firecrawl search "UK electrical contractor field engineers van-based 20 100 employees" --limit 15 --country GB -o .firecrawl/pipeline/session-2/discovery/field-services/uk-3.json --json &
firecrawl search "site:linkedin.com/company HVAC plumbing contractor UK 51-200 employees" --limit 15 -o .firecrawl/pipeline/session-2/discovery/field-services/uk-linkedin.json --json &

# UAE
firecrawl search "Dubai HVAC maintenance company facilities field service 50 employees" --limit 15 --country AE -o .firecrawl/pipeline/session-2/discovery/field-services/gcc-1.json --json &
firecrawl search "UAE MEP maintenance contractor AC plumbing electrical building" --limit 15 --country AE -o .firecrawl/pipeline/session-2/discovery/field-services/gcc-2.json --json &

# Brazil
firecrawl search "empresa manutenção predial Brasil HVAC ar condicionado elétrica hidráulica PME" --limit 15 --country BR -o .firecrawl/pipeline/session-2/discovery/field-services/brazil-1.json --json &
firecrawl search "empresa instalação manutenção equipamentos Brasil técnicos campo" --limit 15 --country BR -o .firecrawl/pipeline/session-2/discovery/field-services/brazil-2.json --json &

# Africa
firecrawl search "HVAC maintenance company South Africa plumbing electrical field service" --limit 15 -o .firecrawl/pipeline/session-2/discovery/field-services/africa-1.json --json &
wait
```

### Step 3: ICP 5 — Facility Management Discovery

```bash
# Italy
firecrawl search "facility management Italia azienda pulizie manutenzione immobili PMI 50 200 dipendenti" --limit 20 --country IT -o .firecrawl/pipeline/session-2/discovery/facility-mgmt/italy-1.json --json &
firecrawl search "gestione strutture Italia servizi integrati pulizia manutenzione multi-sito" --limit 15 --country IT -o .firecrawl/pipeline/session-2/discovery/facility-mgmt/italy-2.json --json &
firecrawl search "site:linkedin.com/company facility management Italia 51-200" --limit 15 -o .firecrawl/pipeline/session-2/discovery/facility-mgmt/italy-linkedin.json --json &

# UK
firecrawl search "UK facility management company cleaning maintenance 50 200 employees regional" --limit 20 --country GB -o .firecrawl/pipeline/session-2/discovery/facility-mgmt/uk-1.json --json &
firecrawl search "UK building maintenance company multi-site property services 50 employees" --limit 15 --country GB -o .firecrawl/pipeline/session-2/discovery/facility-mgmt/uk-2.json --json &
firecrawl search "site:linkedin.com/company facilities management UK 51-200 employees" --limit 15 -o .firecrawl/pipeline/session-2/discovery/facility-mgmt/uk-linkedin.json --json &

# UAE
firecrawl search "Dubai facility management company building maintenance services 50 200" --limit 15 --country AE -o .firecrawl/pipeline/session-2/discovery/facility-mgmt/gcc-1.json --json &
firecrawl search "UAE property management company facilities cleaning maintenance" --limit 15 --country AE -o .firecrawl/pipeline/session-2/discovery/facility-mgmt/gcc-2.json --json &

# Brazil
firecrawl search "empresa facilities management Brasil gestão predial limpeza manutenção PME" --limit 15 --country BR -o .firecrawl/pipeline/session-2/discovery/facility-mgmt/brazil-1.json --json &

# Africa
firecrawl search "facility management company South Africa Kenya building maintenance" --limit 15 -o .firecrawl/pipeline/session-2/discovery/facility-mgmt/africa-1.json --json &
wait
```

### Step 4: Process Results

For each company found:
1. Check it's not already in the master list (avoid duplicates)
2. Check against disqualification criteria
3. Record: name, country, ICP, source URL, website, LinkedIn (if found), employee estimate

### Step 5: Quick LinkedIn verification for top finds

```bash
firecrawl search "site:linkedin.com/company [Company Name]" --limit 3 -o .firecrawl/pipeline/session-2/discovery/[icp]/[slug]-linkedin.json --json
```

### Step 6: Add to master company list

Edit `docs/sales/analysis/company-research/master-company-list.md`:
- Add new ICP 4 and ICP 5 sections under each geography
- Add to the ICP Legend table: ICP 4 (Field Services) and ICP 5 (Facility Management)
- Use the same table schema: Company | LinkedIn URL | Website | Employees | HQ | Founded | ICP Fit | Status | Notes
- Update the Pipeline Summary table with new ICP rows

### Output

Send a message to team-lead with:
- Total new companies per ICP and geography
- Top 10 highest-confidence finds
- Updated pipeline summary counts

---

## Agent 3: Deep Signals Agent

**Name:** `signals-agent`
**Mission:** Find company-specific signals for the top-scored companies from Session 1, plus competitor intelligence. This is about upgrading generic market signals to actionable, company-specific outreach triggers.

### Step 1: Read Session 1 outputs
```bash
Read docs/sales/analysis/company-research/master-company-list.md
Read docs/sales/analysis/signals-session-1.md
```

### Priority companies from Session 1 (focus here):

| Company | Score | Geo | Why |
|---------|-------|-----|-----|
| CANTIERI S.p.A. | 90 | Italy | Highest scored. 200+ vehicles. Infrastructure focus |
| ALC Contracting LLC | 87 | UAE | Dubai boom. 70% repeat clients |
| Safwan General Contracting | 87 | UAE | Small-medium contractor in growing market |
| Rukin Al Basmah | 82 | UAE | Dubai growth signal |
| Connor Engenharia | 82 | Brazil | Growth + labor shortage |
| Construtora Saba | 82 | Brazil | Growth + labor shortage |
| Aka Brasil Construcoes | 82 | Brazil | Growth + labor shortage |
| Princebuild | 77 | UK | WhatsApp pain signal. 101-250 employees |
| Giffi Noleggi | unscored | Italy | ICP 2 Tier A. Actively hiring |
| Murici Logistica | unscored | Brazil | 450+ Ford Transit fleet. Rapid growth |

### Step 2: UAE company-specific signals

```bash
# Contract wins, project announcements for top UAE companies
firecrawl search "ALC Contracting Dubai new project contract awarded 2025 2026" --limit 10 --tbs qdr:y -o .firecrawl/pipeline/session-2/signals/company/alc-news.json --json &
firecrawl search "Safwan General Contracting Dubai project awarded expansion" --limit 10 --tbs qdr:y -o .firecrawl/pipeline/session-2/signals/company/safwan-news.json --json &
firecrawl search "Rukin Al Basmah building contracting Dubai project" --limit 10 --tbs qdr:y -o .firecrawl/pipeline/session-2/signals/company/rukin-news.json --json &

# Scrape their LinkedIn for recent posts
firecrawl search "site:linkedin.com ALC Contracting LLC Dubai 2026" --limit 5 -o .firecrawl/pipeline/session-2/signals/company/alc-linkedin.json --json &
firecrawl search "site:linkedin.com Safwan General Contracting Dubai 2026" --limit 5 -o .firecrawl/pipeline/session-2/signals/company/safwan-linkedin.json --json &
wait
```

### Step 3: Brazil company-specific signals + labor shortage angle

```bash
# Connor, Saba, Aka specific news
firecrawl search "Connor Engenharia Montes Claros projeto obra contrato 2025 2026" --limit 10 --country BR -o .firecrawl/pipeline/session-2/signals/company/connor-news.json --json &
firecrawl search "Construtora Saba Belo Horizonte obra projeto 2025 2026" --limit 10 --country BR -o .firecrawl/pipeline/session-2/signals/company/saba-news.json --json &

# Brazil construction tech adoption / WhatsApp in construction
firecrawl search "construção civil Brasil tecnologia WhatsApp digitalização gestão obra" --limit 15 --country BR -o .firecrawl/pipeline/session-2/signals/pain/brazil-whatsapp-construction.json --json &
firecrawl search "falta mão de obra construção Brasil 2025 2026 soluções tecnológicas" --limit 15 --country BR -o .firecrawl/pipeline/session-2/signals/pain/brazil-labor-shortage-tech.json --json &

# Murici Logistica growth
firecrawl search "Murici Logística Ford Transit frota expansão 2025 2026" --limit 10 --country BR -o .firecrawl/pipeline/session-2/signals/company/murici-news.json --json &
wait
```

### Step 4: Italy company-specific signals + Terna infrastructure

```bash
# CANTIERI specific news (highest scored company)
firecrawl search "CANTIERI SpA Poggio Renatico appalto progetto infrastruttura 2025 2026" --limit 10 --country IT -o .firecrawl/pipeline/session-2/signals/company/cantieri-news.json --json &

# Terna EUR17.7B infrastructure plan — which companies benefit?
firecrawl search "Terna piano industriale 17 miliardi appaltatori imprese edili coinvolte" --limit 15 --country IT -o .firecrawl/pipeline/session-2/signals/growth/italy-terna-plan.json --json &

# Italian construction digitalization pain
firecrawl search "cantiere digitale Italia WhatsApp coordinamento squadre problema gestione" --limit 15 --country IT -o .firecrawl/pipeline/session-2/signals/pain/italy-digital-construction.json --json &

# Giffi Noleggi hiring/growth signals
firecrawl search "Giffi Noleggi espansione crescita assunzioni nuova sede 2025 2026" --limit 10 --country IT -o .firecrawl/pipeline/session-2/signals/company/giffi-news.json --json &
wait
```

### Step 5: UK company-specific signals

```bash
# Princebuild specific news
firecrawl search "Princebuild Peterborough new contract project awarded 2025 2026" --limit 10 --country GB -o .firecrawl/pipeline/session-2/signals/company/princebuild-news.json --json &

# UK equipment rental ICP 2 signals — more pain + hiring
firecrawl search "UK plant hire equipment rental companies growing fleet expansion 2026" --limit 15 --sources news --tbs qdr:m -o .firecrawl/pipeline/session-2/signals/growth/uk-rental-expansion.json --json &

# UK field services pain (for ICP 4 context)
firecrawl search "site:reddit.com UK HVAC plumbing electrician crew scheduling WhatsApp job management" --limit 15 --tbs qdr:y -o .firecrawl/pipeline/session-2/signals/pain/uk-field-services-reddit.json --json &
wait
```

### Step 6: Africa WhatsApp penetration data

```bash
firecrawl search "WhatsApp adoption Africa construction field workers business communication statistics 2025" --limit 15 -o .firecrawl/pipeline/session-2/signals/growth/africa-whatsapp-adoption.json --json &
firecrawl search "Africa construction technology adoption digital tools field operations" --limit 15 -o .firecrawl/pipeline/session-2/signals/growth/africa-construction-tech.json --json &
wait
```

### Step 7: Competitor intelligence

```bash
# AlterSquare — publishes Excel/WhatsApp migration content
firecrawl search "AlterSquare construction platform Excel WhatsApp migration" --limit 10 -o .firecrawl/pipeline/session-2/signals/competitors/altersquare.json --json &
firecrawl scrape "https://altersquare.medium.com/" --only-main-content -o .firecrawl/pipeline/session-2/signals/competitors/altersquare-blog.md &

# Symterra — UK-focused, wrote about WhatsApp hidden costs
firecrawl search "Symterra construction communication platform WhatsApp alternative" --limit 10 -o .firecrawl/pipeline/session-2/signals/competitors/symterra.json --json &
firecrawl scrape "https://www.symterra.co.uk/blog" --only-main-content -o .firecrawl/pipeline/session-2/signals/competitors/symterra-blog.md &
wait
```

### Step 8: Scrape top signal sources

For the top 10 most relevant findings (Reddit threads, LinkedIn posts, news articles), scrape full content:

```bash
firecrawl scrape "[url]" --only-main-content -o .firecrawl/pipeline/session-2/signals/deep/[slug].md &
```

### Step 9: Classify signals and write report

For each signal found, classify using this format:

```
SIGNAL:
  ID: S2-[P/H/G/C]-[number]  (P=Pain, H=Hiring, G=Growth, C=Company-specific)
  Type: [Pain / Hiring / Growth / Company-Specific]
  Strength: [Critical / High / Warm]
  Source: [URL]
  Platform: [Reddit / LinkedIn / News / Company Website]
  ICP Match: [1-5]
  Geography: [Country]
  Company Match: [Company name from master list, or "No direct match"]
  Quote: "[Key excerpt]"
  Outreach Hook: "[2-3 sentence hook for outreach based on this signal]"
```

Write the signals report to `docs/sales/analysis/signals-session-2.md` containing:
- All classified signals organized by company (company-specific first), then by type
- Combined signal map: merge Session 1 + Session 2 signals per company
- Competitor intelligence summary
- Top 5 strongest new signals with full context and outreach hooks

### Output

Send a message to team-lead with:
- Total new signals found by type
- Company-specific signals matched to top-scored companies
- Competitor intelligence summary (AlterSquare, Symterra positioning + gaps)
- Recommendations for outreach priority order

---

## Team Lead: Consolidation (After All Agents Complete)

Once all 3 agents finish:

### 1. Verify master company list
- Check enrichment-agent and discovery-agent updates don't conflict
- Ensure new ICP 4/5 sections are properly formatted
- Verify Pipeline Summary counts match actual table rows

### 2. Score ALL companies
Apply the pipeline scoring rubric to every company in the master list. Add `**Score: XX/100**` to the Notes column. Use available data:
- Signal Strength: check `signals-session-1.md` + `signals-session-2.md` for matches
- Industry Fit: use ICP number
- Company Size: use employee count (0 if TBD)
- Geography: use HQ location
- Tech Stack: use enrichment notes (default 5 if unknown)

### 3. Re-tier companies based on scores
- Score 75-100 → Tier A
- Score 50-74 → Tier B
- Score 25-49 → Tier C
- Update the ICP Fit column if score disagrees with current tier

### 4. Update pipeline summary
Refresh all counts in the Pipeline Summary table including new ICP 4/5 rows.

### 5. Produce outreach priority list
Create a ranked list of the top 20 companies by score. For each, include:
- Company name, geography, ICP, score
- Signal summary (all signals from both sessions)
- Recommended outreach sequence (A=Pain, B=Growth, C=Cold ICP)
- Outreach language (IT/EN/PT)
- Best outreach hook (from signals report)

Save to `docs/sales/analysis/outreach-priority-session-2.md`

### Session 2 Success Criteria
- [ ] 110+ total companies in master list (up from 81)
- [ ] 60%+ companies have both domain and LinkedIn URL
- [ ] ICP 4 (Field Services) has 15+ companies
- [ ] ICP 5 (Facility Management) has 10+ companies
- [ ] All companies scored with pipeline rubric
- [ ] 15+ company-specific signals detected
- [ ] 15+ Tier A companies (score 75-100)
- [ ] Top 20 outreach priority list produced
