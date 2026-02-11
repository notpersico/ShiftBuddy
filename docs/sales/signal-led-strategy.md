# ShiftBuddy — Signal-Led Sales Strategy

> **Last updated:** 2026-02-11
> **Owner:** Alessio Persichetti, Founder
> **Stage:** Pre-revenue / First 50 customers
> **Target:** B2B SaaS for field teams (20-200 employees) using WhatsApp as ops backbone

---

## 1. ICP Definition & Scoring Model

### Target Buyer Persona

| Role | Title | Why they buy |
|------|-------|-------------|
| **Primary** | Operations Manager / COO | Owns the chaos. Feels equipment loss, missed tasks, coordination failures daily. |
| **Secondary** | Fleet/Logistics Manager | Manages assets + field crews. Lives in WhatsApp groups already. |
| **Economic** | Owner / GM | Signs the check. Cares about cost of lost equipment, downtime, liability. |

### Tier Definitions

#### Tier A — "Hot Fit" (Score 75-100)
- 30-150 employees, 10+ field workers
- Construction, equipment rental, or logistics
- Based in Italy, UK, UAE, Saudi Arabia, Brazil, Nigeria, South Africa, Kenya
- Already using WhatsApp groups for ops coordination
- No visible telematics/asset tracking vendor
- Active hiring for ops/fleet/logistics roles OR recent pain signal

#### Tier B — "Warm Fit" (Score 50-74)
- 20-200 employees, 5+ field workers
- Field services, facility management, waste management, agriculture
- WhatsApp-heavy geography (any of the above + India, Indonesia, Philippines, Colombia, Mexico)
- May have basic tools (Google Sheets, Trello) but no field-specific solution
- Growth signals present but no explicit pain signal

#### Tier C — "Cold Fit" (Score 25-49)
- 20-200 employees in a relevant industry
- Geography fits but no visible signals
- Might use competitor tools (lightweight ones)
- Worth nurturing, not worth cold outreach at scale

### Scoring Rubric (0-100)

| Category | Signal | Points |
|----------|--------|--------|
| **Industry** (max 25) | Construction / Equipment Rental | 25 |
| | Logistics / Field Services | 20 |
| | Facility Mgmt / Waste / Agriculture | 15 |
| | Other with field teams | 10 |
| **Company Size** (max 15) | 30-150 employees | 15 |
| | 20-29 or 151-200 | 10 |
| | <20 or >200 | 5 |
| **Geography** (max 15) | Italy | 15 |
| | UK, UAE, Saudi, Brazil, South Africa | 12 |
| | Other WhatsApp-heavy market | 8 |
| | Low WhatsApp market | 3 |
| **Tech Stack** (max 15) | No asset tracking tool visible + uses WhatsApp groups | 15 |
| | Uses spreadsheets / basic tools | 10 |
| | Has a lightweight competitor | 5 |
| | Has enterprise solution (SAP, Oracle) | 0 |
| **Hiring Signals** (max 10) | Hiring Ops Manager / Fleet Coordinator / Logistics Supervisor | 10 |
| | Hiring general field workers (scaling) | 5 |
| | No hiring activity | 0 |
| **Pain/Growth Signals** (max 10) | Posted about equipment loss, coordination problems, disputes | 10 |
| | New project win / office opening / fleet expansion | 7 |
| | Generic growth indicators | 3 |
| **Engagement** (max 10) | Visited website + opened email | 10 |
| | Engaged with LinkedIn content | 7 |
| | Viewed profile | 3 |
| | No engagement | 0 |

**Total: 100 points**

### Priority Markets (Launch Order)

1. **Italy** — Home market. Network advantage. Italian-language support = moat.
2. **UK** — English-speaking, high WhatsApp adoption in trades/construction, SMB-dense.
3. **UAE + Saudi Arabia** — Construction boom, WhatsApp is default comms, high willingness to pay.
4. **Brazil** — Largest WhatsApp market globally, massive construction + logistics sector.
5. **South Africa + Kenya + Nigeria** — WhatsApp-first economies, growing construction sectors.

---

## 2. Signal Taxonomy

### 2.1 Hiring Signals
**What it means:** They're scaling operations = growing pains = coordination breaks down.

| Signal | Detection Method | Tool | Priority Weight |
|--------|-----------------|------|----------------|
| Job post: "Operations Manager" | LinkedIn Jobs search, Indeed scraping | LinkedIn Sales Nav, Apollo.io, Google Alerts | **HIGH** (10 pts) |
| Job post: "Fleet Coordinator" / "Fleet Manager" | Same | Same | **HIGH** (10 pts) |
| Job post: "Logistics Supervisor" / "Site Supervisor" | Same | Same | **HIGH** (10 pts) |
| Hiring 5+ field workers simultaneously | Job board monitoring | Indeed, Apollo job change alerts | **MEDIUM** (5 pts) |
| New Head of Ops / COO hired (job change alert) | LinkedIn notifications | Sales Nav alerts, Apollo | **HIGH** — new leader = new tools budget |

**Detection cadence:** Check weekly. Set up saved searches in LinkedIn + Apollo.

### 2.2 Tech Signals
**What it means:** They have the problem but don't have a solution.

| Signal | Detection Method | Tool | Priority Weight |
|--------|-----------------|------|----------------|
| No GPS/telematics vendor on website or LinkedIn | Manual check of company page, tech stack tools | BuiltWith, Wappalyzer, manual review | **HIGH** (15 pts) |
| Uses WhatsApp groups for coordination | Mentioned in job posts ("must be comfortable with WhatsApp"), visible in reviews | Glassdoor, Indeed job descriptions | **HIGH** (15 pts) |
| Uses Google Sheets / Excel for tracking | Job posts mention "proficient in Excel", no SaaS tools listed | Job descriptions, company reviews | **MEDIUM** (10 pts) |
| Website has no "technology" or "platform" section | Manual review | Browse company website | **LOW** (5 pts) |

**Key negative signal:** If they list Verizon Connect, Samsara, Teletrac, or any enterprise ERP with field module → deprioritize.

### 2.3 Pain Signals
**What it means:** They're actively experiencing the problem ShiftBuddy solves.

| Signal | Detection Method | Tool | Priority Weight |
|--------|-----------------|------|----------------|
| LinkedIn post about equipment theft/loss | Boolean search: industry + "equipment stolen" / "missing tools" / "theft on site" | LinkedIn search, Google Alerts, Mention.com | **CRITICAL** (10 pts) — immediate outreach |
| Comment on post about coordination chaos | Monitor industry LinkedIn groups + influencer posts | Manual + Phantombuster for scraping | **HIGH** (10 pts) |
| Negative Google/Trustpilot review mentioning ops failures | Review monitoring | Google Alerts for company name + reviews | **MEDIUM** (7 pts) |
| Forum post (Reddit, industry forums) about "how do you track equipment" | Reddit search, industry forum monitoring | Reddit search, Google Alerts | **HIGH** (10 pts) |
| Rental dispute mentioned publicly | Google search, court records, LinkedIn | Google Alerts | **MEDIUM** (7 pts) |

**Detection cadence:** Daily for LinkedIn searches (5 min). Weekly for broader monitoring.

### 2.4 Growth Signals
**What it means:** They're expanding = more assets, more people, more chaos.

| Signal | Detection Method | Tool | Priority Weight |
|--------|-----------------|------|----------------|
| Won a major new contract/project | LinkedIn company posts, PR, local news | Google Alerts, LinkedIn company follow | **HIGH** (7 pts) |
| Opening new office/warehouse/site | Company announcements, LinkedIn, local news | Google Alerts | **HIGH** (7 pts) |
| Fleet expansion (buying equipment) | LinkedIn posts, industry marketplace activity | Manual monitoring | **MEDIUM** (5 pts) |
| Funding round announced | Crunchbase, TechCrunch, local press | Crunchbase alerts (free tier) | **MEDIUM** (5 pts) |
| Revenue growth mentioned | Press releases, interviews | Google Alerts | **LOW** (3 pts) |

### 2.5 Engagement Signals
**What it means:** They already know about ShiftBuddy or related content.

| Signal | Detection Method | Tool | Priority Weight |
|--------|-----------------|------|----------------|
| Visited ShiftBuddy website | Website analytics + visitor identification | Clearbit Reveal / RB2B (free) / Leadfeeder | **CRITICAL** (10 pts) |
| Opened email 3+ times | Email tracking | Instantly.ai / Mailtrack | **HIGH** (7 pts) |
| Clicked link in email | Email tracking | Instantly.ai | **HIGH** (10 pts) |
| Liked/commented on Alessio's LinkedIn post | LinkedIn notifications | Manual + Sales Nav | **MEDIUM** (7 pts) |
| Viewed Alessio's LinkedIn profile | LinkedIn notifications | Sales Nav | **MEDIUM** (3 pts) |

### 2.6 Industry Signals
**What it means:** External forces making the problem worse or more urgent.

| Signal | Detection Method | Tool | Priority Weight |
|--------|-----------------|------|----------------|
| Construction spring ramp-up (Feb-Apr) | Calendar-based | Internal calendar | **Context** — boost all construction outreach |
| New safety/compliance regulation | Government sites, industry news | Google Alerts for "[country] construction regulation" | **HIGH** — angle for outreach |
| Insurance premium increases for equipment | Industry press | Trade publication monitoring | **MEDIUM** — cost justification angle |
| Major equipment theft news story | News | Google Alerts "equipment theft [industry]" | **HIGH** — ride the wave with content |
| Industry conference season | Event calendars | Manual tracking | **Context** — time content around events |

---

## 3. Outreach Sequences

### Personalization Tokens Reference
- `{{first_name}}` — Contact first name
- `{{company}}` — Company name
- `{{industry}}` — Their specific industry
- `{{signal}}` — The specific signal that triggered outreach
- `{{signal_detail}}` — Specifics (job title posted, project won, etc.)
- `{{city}}` — Their city/region
- `{{team_size}}` — Estimated field team size
- `{{pain_point}}` — Specific pain inferred from signal

---

### Sequence A: High-Intent Signal Detected
**Trigger:** Pain signal — they posted about equipment loss, coordination failure, or rental disputes.
**Goal:** Book a 15-min call within 7 days.
**Tone:** Empathetic peer, not salesy.

#### Touch 1 — LinkedIn Connection Request (Day 0)
> **Note:** (300 char limit)
>
> Hi {{first_name}}, saw your post about {{signal_detail}}. I've been building something specifically for this — WhatsApp-based asset tracking for field teams. Would love to share what we're learning. No pitch, just a conversation between operators.

#### Touch 2 — LinkedIn DM (Day 1, after accepted)
> Hey {{first_name}}, thanks for connecting.
>
> Your post about {{signal_detail}} hit home — I spent 2 years in the field watching the same thing happen. Tools disappear, nobody knows who had what, and the WhatsApp group becomes a blame game.
>
> That's actually why I built ShiftBuddy. It turns WhatsApp into an ops task manager + asset tracker. Your crew doesn't need a new app — they just use WhatsApp like they already do, but now every task and every piece of equipment is logged automatically.
>
> Would a 15-min call be useful? I can show you how 2-3 companies similar to {{company}} are handling this. No commitment.
>
> Either way, happy to connect with someone who gets the field ops reality.

#### Touch 3 — Email (Day 3)
> **Subject:** {{first_name}}, re: your {{signal_detail}} problem
>
> Hi {{first_name}},
>
> I noticed your LinkedIn post about {{signal_detail}} at {{company}}. I've been hearing this from a lot of {{industry}} companies with {{team_size}}+ people in the field.
>
> Quick question: how are you currently tracking which equipment is where and who's responsible?
>
> We built ShiftBuddy because every field team we talked to was using WhatsApp groups + spreadsheets and losing €10-50K/year in equipment alone. Our approach: your crew keeps using WhatsApp (no new app to learn), but every task assignment and asset handoff gets logged automatically.
>
> Worth a 15-min chat? I can share what's working for similar teams.
>
> Cheers,
> Alessio

#### Touch 4 — LinkedIn Comment/Engage (Day 5)
> Don't DM. Instead, engage with their content. Like a recent post. Leave a thoughtful comment on something they shared. Stay visible. If they posted again about ops, comment with a useful insight (NOT a pitch).

#### Touch 5 — Email Breakup (Day 8)
> **Subject:** closing the loop
>
> Hi {{first_name}},
>
> I reached out last week about the {{pain_point}} challenge at {{company}}. Totally understand if the timing isn't right.
>
> I'll drop a link to a quick guide we put together: "5 Ways Field Teams Lose €30K/Year Without Knowing It" — might be useful regardless.
>
> [Link]
>
> If things get painful enough, my calendar's always open: [Calendly link]
>
> Rooting for you either way.
>
> Alessio

---

### Sequence B: Growth Signal Detected
**Trigger:** Hiring fleet/ops roles, won new project, opening new site.
**Goal:** Plant the seed before the pain hits.
**Tone:** Congratulatory, forward-looking.

#### Touch 1 — LinkedIn Connection Request (Day 0)
> Hi {{first_name}}, congrats on {{signal_detail}} at {{company}}! I work with {{industry}} teams scaling their field operations — would love to connect and share some patterns I'm seeing.

#### Touch 2 — Email (Day 2)
> **Subject:** Scaling {{company}}'s field ops without the chaos
>
> Hi {{first_name}},
>
> Congrats on {{signal_detail}} — that's a big move for {{company}}.
>
> I've been talking to a lot of {{industry}} companies going through similar growth, and there's a pattern: what works with 15 field workers breaks completely at 30+. The WhatsApp group becomes unmanageable, equipment starts "disappearing," and the ops manager becomes a full-time firefighter.
>
> We built ShiftBuddy to prevent that. It plugs into WhatsApp (no new app for the crew) and gives you automatic task tracking + asset control. Think of it as the ops layer your field team didn't know they needed.
>
> Since you're scaling now, this might be worth a 15-min look before the pain hits. Open to a quick chat this week?
>
> Best,
> Alessio

#### Touch 3 — LinkedIn DM (Day 5)
> Hey {{first_name}}, not sure if you saw my email — totally get it if you're buried in the growth.
>
> One quick thought: we put together a checklist called "The Field Ops Scaling Checklist — What Breaks at 30, 50, and 100+ Workers." Want me to send it over? Might save you some headaches.

#### Touch 4 — Email Value-Add (Day 8)
> **Subject:** what breaks at 50 field workers
>
> Hi {{first_name}},
>
> Short one. I compiled data from 20+ field ops companies on what breaks as you scale:
>
> - At 30 workers: WhatsApp groups become noise. Tasks get missed daily.
> - At 50 workers: Equipment loss hits 5-8% of fleet annually.
> - At 100 workers: You need a dedicated person just to manage coordination.
>
> Doesn't have to be this way. Most of this is solvable with better systems — not more people.
>
> Here's the full breakdown: [Link to content piece]
>
> Worth a chat?
>
> Alessio

#### Touch 5 — LinkedIn Engage + Soft CTA (Day 12)
> Engage with their content for the next week. Then:
>
> Hey {{first_name}}, I keep seeing great things happening at {{company}}. Whenever you want to chat about keeping field ops tight during the growth, I'm here. No rush — [Calendly link]

---

### Sequence C: Cold but ICP-Fit
**Trigger:** No signal, but they match Tier A/B on firmographics.
**Goal:** Generate a signal. Get them to engage with content.
**Tone:** Peer-to-peer, industry insider.

#### Touch 1 — Email (Day 0)
> **Subject:** quick question about {{company}}'s field ops
>
> Hi {{first_name}},
>
> I'm researching how {{industry}} companies in {{city}} manage field team coordination. {{company}} came up as an interesting example given your size and scope.
>
> Curious: are your field crews using WhatsApp groups to coordinate daily tasks and equipment handoffs? (Most companies your size are, and most are frustrated with it.)
>
> I'm building a tool that turns WhatsApp into an actual ops system — task tracking, asset control, accountability — without making the team learn a new app.
>
> Would love to hear how you're handling it today, even if just a quick reply.
>
> Cheers,
> Alessio

#### Touch 2 — LinkedIn Connection Request (Day 2)
> Hi {{first_name}}, I'm working with {{industry}} companies on field ops efficiency. Thought we might have some mutual interests — happy to connect.

#### Touch 3 — Email (Day 5)
> **Subject:** re: field ops at {{company}}
>
> Hi {{first_name}},
>
> Forgot to mention — I put together a quick breakdown of the hidden costs most {{industry}} companies don't track:
>
> 1. **Equipment loss:** Average 3-7% of movable assets lost/stolen annually (€15-75K for a company your size)
> 2. **Task slippage:** 12-18% of daily tasks fall through the cracks without a system
> 3. **Coordination time:** Ops managers spend 2-3 hours/day just chasing updates via WhatsApp
>
> Any of this sound familiar?
>
> Full article here: [Link]
>
> Alessio

#### Touch 4 — LinkedIn DM or Comment (Day 8)
> If connected: share a relevant content piece via DM.
> If not connected: comment on a company post or the contact's post with genuine insight about their industry.

#### Touch 5 — Email Breakup (Day 12)
> **Subject:** not for everyone
>
> Hi {{first_name}},
>
> I've reached out a couple of times about field ops at {{company}}. Totally possible this isn't a priority right now — or you've already solved it.
>
> If you ever want to compare notes on how {{industry}} companies are managing field coordination, I'm easy to find: [Calendly]
>
> One resource to bookmark: [Link to best content piece]
>
> All the best with {{company}}.
>
> Alessio

---

## 4. LinkedIn Content Strategy

### Profile Optimization for Alessio

**Headline:**
`Building ShiftBuddy — turning WhatsApp into an ops system for field teams | Construction · Equipment Rental · Logistics`

**About section:**
> Field teams run on WhatsApp. But WhatsApp wasn't built to track tasks, manage equipment, or create accountability.
>
> I'm building ShiftBuddy to fix that. No new app for your crew to learn. Just WhatsApp — but with task management, asset tracking, and real operational control built in.
>
> If you manage 20-200 field workers and you're tired of equipment disappearing, tasks falling through cracks, and WhatsApp groups full of noise — let's talk.
>
> 📩 alessio@shiftbuddy.io | 📅 [Calendly link]

**Banner image:** Simple graphic: "WhatsApp + Field Ops = ShiftBuddy" with construction/logistics imagery.

**Featured section:** Pin the best-performing post + a link to a lead magnet (checklist or guide).

### Content Pillars

1. **Field Ops Pain** — Real stories about what goes wrong without systems
2. **WhatsApp in Business** — Why teams use it, why it breaks, how to fix it
3. **Asset Tracking Reality** — Equipment loss stats, theft stories, cost breakdowns
4. **Industry Insights** — Trends, data, regulatory changes affecting field teams

### 12 Post Ideas (Month 1: 3x/week)

**Week 1**

**Post 1 — Pain (Mon)**
> Hook: "A construction company in Milan lost €47,000 in equipment last year. They didn't even know until the audit."
>
> Angle: Walk through how "small" losses compound. Each site loses a few tools. Nobody reports it because there's no system. End with: "If your tracking system is a WhatsApp message that says 'who has the generator?' — you don't have a tracking system."
> CTA: "What's the most expensive thing that 'disappeared' from your site? 👇"

**Post 2 — WhatsApp (Wed)**
> Hook: "Your field team already has the best ops tool installed on their phone. You're just not using it right."
>
> Angle: WhatsApp is already where coordination happens. The problem isn't the channel — it's that nothing is tracked, nothing is assigned, nothing is accountable. What if WhatsApp messages became tasks automatically?
> CTA: "How many WhatsApp groups does your ops manager monitor daily?"

**Post 3 — Industry Data (Fri)**
> Hook: "73% of construction companies under 200 employees have no formal asset tracking system. Zero."
>
> Angle: Share real stats (cite sources). Compare cost of losses vs. cost of a simple system. Frame it as "this isn't a tech problem, it's a visibility problem."
> CTA: "Is your company in the 73% or the 27%?"

**Week 2**

**Post 4 — Story (Mon)**
> Hook: "I watched a site manager spend 45 minutes on the phone trying to find a concrete saw. It was in the truck parked 50 meters away."
>
> Angle: Personal story or anonymized customer story. The absurdity of not knowing where your own equipment is. The hidden cost of "searching time."
> CTA: "What's the dumbest thing your team spent time looking for?"

**Post 5 — Contrarian (Wed)**
> Hook: "Stop trying to get your field crew off WhatsApp. You'll lose."
>
> Angle: Every ops manager tries to introduce a new app. Crew ignores it within 2 weeks. The answer isn't replacing WhatsApp — it's making WhatsApp work harder. Meet people where they are.
> CTA: "Have you ever introduced a field app that actually stuck? Genuinely curious."

**Post 6 — Data/Insight (Fri)**
> Hook: "The average equipment rental company spends 14% of revenue replacing lost or damaged assets. Not repairing. Replacing."
>
> Angle: Break down the math. Show how even reducing this by half pays for an ops system 10x over. Keep it simple, visual if possible.
> CTA: "What percentage would you guess for your company?"

**Week 3**

**Post 7 — Behind the Scenes (Mon)**
> Hook: "I'm building ShiftBuddy in public. Here's what I learned talking to 30 field ops managers this month."
>
> Angle: Share 3-5 key insights from customer discovery. What surprised you. What everyone said. Make it feel like inside access.
> CTA: "If you run field operations, I'd love to add your perspective. DM me."

**Post 8 — How-To (Wed)**
> Hook: "A free way to reduce equipment loss by 30% this month (no software needed)"
>
> Angle: Give away a genuine process improvement. Daily check-in/check-out photo system via WhatsApp. Show the manual process. Then hint: "We automated this entirely — but the manual version still works."
> CTA: "Would you try this on your next project?"

**Post 9 — Pain Carousel (Fri)**
> Hook: "5 signs your field ops are running on chaos (slide 1 of 5)"
>
> Angle: Carousel/document post. Each slide = one sign. 1) More than 5 WhatsApp groups for coordination. 2) Ops manager spends 2+ hrs/day chasing updates. 3) Equipment "just disappears" regularly. 4) New hires take 2+ weeks to get productive. 5) You've said "who had the drill last?" this week.
> CTA: "How many of these hit? Be honest 👇"

**Week 4**

**Post 10 — Social Proof (Mon)**
> Hook: "Before/After: How one logistics company went from 3-hour morning coordination to 15 minutes."
>
> Angle: Case study format (anonymized if needed, or use a beta user). Show the old process vs. new process. Quantify time saved.
> CTA: "What does your morning coordination look like?"

**Post 11 — Industry Trend (Wed)**
> Hook: "Construction companies that digitize field ops win 23% more repeat contracts. Here's why."
>
> Angle: Tie operational excellence to business outcomes. Clients want to work with organized companies. Insurance costs drop. Disputes disappear.
> CTA: "Has a client ever asked about your ops systems before awarding a contract?"

**Post 12 — Personal/Mission (Fri)**
> Hook: "I quit my job to solve a problem most people don't even know they have."
>
> Angle: Founder story. Why ShiftBuddy exists. The moment you realized this problem was worth solving. Authentic, human, slightly vulnerable.
> CTA: "If you've ever felt the pain of managing field chaos, I'd love to hear your story."

### Comment Strategy

**Daily (15 min/day):**
1. Search LinkedIn for keywords: "equipment theft", "construction management", "field operations", "fleet management", "WhatsApp business"
2. Find posts from target ICPs or industry influencers
3. Leave value-add comments (NOT "great post!"):
   - Share a relevant data point
   - Ask a thoughtful follow-up question
   - Share a related personal experience
4. Engage with commenters on your own posts within 1 hour of posting

**Target communities/hashtags:**
- #construction #equipmentrental #logistics #fieldservice #fleetmanagement
- Groups: Construction Management, Logistics Professionals, Equipment Rental Network

**Rule:** 5 genuine comments on others' posts for every 1 post you publish. Give before you ask.

---

## 5. Metrics & Feedback Loop

### Weekly Metrics Dashboard

| Metric | Target (Month 1) | Target (Month 3) |
|--------|-------------------|-------------------|
| Signals detected | 20/week | 50/week |
| Outreach sent (total touches) | 50/week | 150/week |
| Connection requests sent | 30/week | 60/week |
| Connection accept rate | >30% | >40% |
| Email open rate | >50% | >55% |
| Email reply rate | >8% | >12% |
| Calls booked | 2/week | 5/week |
| LinkedIn post impressions | 1,000/week | 5,000/week |
| LinkedIn post engagement rate | >3% | >5% |
| Content pieces published | 3/week | 3/week |
| Comments on others' posts | 25/week | 25/week |

### Signal Catalogue — Spreadsheet Structure

**Sheet 1: Signal Log**

| Date | Company | Contact | Signal Type | Signal Detail | Source | Score | Sequence Assigned | Status |
|------|---------|---------|-------------|---------------|--------|-------|-------------------|--------|
| 2026-02-11 | Rossi Costruzioni | Marco Rossi | Pain | LinkedIn post about stolen generator | LinkedIn search | 85 | Sequence A | Touch 1 sent |

**Sheet 2: Account Scores**

| Company | Industry | Size | Geography | Tech Score | Hiring Score | Pain Score | Engagement Score | Total Score | Tier | Last Signal Date | Notes |
|---------|----------|------|-----------|------------|-------------|------------|-----------------|-------------|------|-----------------|-------|

**Sheet 3: Sequence Tracker**

| Company | Contact | Sequence | Touch # | Channel | Date Sent | Opened | Replied | Call Booked | Notes |
|---------|---------|----------|---------|---------|-----------|--------|---------|-------------|-------|

**Sheet 4: Weekly Review**

| Week | Signals Found | Outreach Sent | Replies | Calls Booked | Key Learning | Adjustment Made |
|------|--------------|---------------|---------|-------------|-------------|-----------------|

### Feedback Loop Process

```
DETECT → SCORE → SEQUENCE → TRACK → LEARN → ADJUST
  │                                        │
  └────────────────────────────────────────┘
```

**Every Friday (30 min):**
1. Review all outreach sent this week
2. Log which signals led to replies/calls (Sheet 4)
3. Update scoring weights if certain signals convert better
4. Adjust sequence copy based on what got replies
5. Add new signal sources discovered
6. Archive dead accounts, promote warm ones

**Monthly (1 hour):**
1. Analyze signal-to-call conversion by signal type
2. Identify top-performing email subject lines and LinkedIn hooks
3. Update ICP definition if patterns emerge
4. Adjust geographic focus based on response rates
5. Decide whether to expand to new signal sources or tools

---

## 6. Tools Stack

### Bootstrapped Stack (<€200/month)

| Tool | Purpose | Cost | Free Alternative |
|------|---------|------|-----------------|
| **LinkedIn Sales Navigator Core** | Find ICPs, saved searches, lead alerts, InMail | €80/mo | LinkedIn free search (limited) |
| **Apollo.io** (Free tier) | Email finding, job change alerts, company data | €0 (free: 10K credits/mo) | Hunter.io free tier |
| **Instantly.ai** (Growth) | Email outreach, sequences, warmup | €30/mo | Gmail + Sheets manual tracking |
| **RB2B** (Free tier) | Website visitor identification | €0 (free: 100 IDs/mo) | Google Analytics (no company ID) |
| **Phantombuster** | LinkedIn scraping, auto-connect | €56/mo | Manual LinkedIn work |
| **Google Sheets** | CRM, signal tracking, scoring | €0 | — |
| **Calendly** (Free) | Meeting scheduling | €0 | Cal.com (free, open source) |
| **Google Alerts** | Monitor news, keywords, companies | €0 | — |
| **Canva** (Free) | LinkedIn carousels, graphics | €0 | — |

**Total: ~€166/month**

### Phase 1 Budget (First 2 months — €100/mo max)

Start with only:
- **Apollo.io Free** — €0 (email finding + basic signals)
- **Instantly.ai Growth** — €30/mo (email sequences + warmup)
- **Google Sheets** — €0 (CRM + tracking)
- **Google Alerts** — €0 (signal detection)
- **Calendly Free** — €0
- **LinkedIn Free** — €0 (manual search, 100 profile views/week)
- **Canva Free** — €0

**Phase 1 Total: €30/month**

### Phase 2 Budget (Month 3+, after first revenue — €200/mo)

Add:
- **LinkedIn Sales Navigator** — €80/mo (unlocks saved searches, lead alerts, InMail)
- **Phantombuster** — €56/mo (automate LinkedIn outreach)

**Phase 2 Total: €166/month**

### Phase 3 Budget (After 10 paying customers)

Consider:
- **Clay** — €149/mo (enrichment + signal automation) — replaces manual signal detection
- **Lemlist** — alternative to Instantly with LinkedIn integration
- **HubSpot Free CRM** — replaces Google Sheets when pipeline grows

### Tool Setup Priorities (Do This Week)

1. ☐ Create Apollo.io free account, import first 50 ICP companies
2. ☐ Set up Instantly.ai, connect email, start warmup (takes 2 weeks)
3. ☐ Create Google Sheets CRM from template above
4. ☐ Set up 10 Google Alerts (key industry terms + competitor names)
5. ☐ Optimize LinkedIn profile per Section 4
6. ☐ Set up Calendly with 15-min "Quick Chat" meeting type
7. ☐ Write first 4 LinkedIn posts (Week 1) and schedule
8. ☐ Build first target list: 50 Tier A companies in Italy + UK

---

## Appendix: First 2 Weeks Execution Plan

### Week 1
- **Mon:** Set up all free tools. Optimize LinkedIn. Write 4 posts.
- **Tue:** Build target list of 50 companies (25 Italy, 25 UK). Score them.
- **Wed:** Publish first LinkedIn post. Start signal detection routine.
- **Thu:** Send first 10 outreach emails (Sequence C — cold ICP fit, while warmup runs).
- **Fri:** Send 10 LinkedIn connection requests. Review and adjust.

### Week 2
- **Mon:** Publish post #2. Follow up on Week 1 connections.
- **Tue:** Detect new signals. Score and assign sequences.
- **Wed:** Publish post #3. Send 15 outreach touches.
- **Thu:** Engage in 10 LinkedIn conversations (comments).
- **Fri:** Weekly review. Update sheets. Adjust copy if needed.

**Target end of Week 2:** 50 touches sent, 3 posts published, 30 connections requested, 1-2 calls booked.

---

*This is a living document. Update weekly based on what's working.*
