# ShiftBuddy — CSV Research: Mid-Size Construction Companies

> **Last updated:** 2026-02-16
> **Purpose:** Prompt for an AI research specialist to analyze the Construction ICP and produce an optimal outbound CSV schema for Clay-style enrichment workflows

---

## ROLE

You are a senior B2B market intelligence analyst and outbound systems architect.

Your task is to perform a deep-dive research analysis on the provided ICP profile for a WhatsApp-native field operations platform (ShiftBuddy).

---

## INSTRUCTIONS

You must:

1. Analyze the ICP below in depth:

   * Industry structure
   * Company characteristics
   * Asset profile
   * Buyer titles
   * Organizational signals
   * Operational complexity indicators
   * Likelihood of WhatsApp-native coordination
   * Revenue leakage signals
   * Field team size indicators

2. Identify:

   * The strongest disambiguation signals for accurate company matching
   * The highest-converting buying triggers
   * The most predictive firmographic filters
   * The cleanest enrichment-ready identifiers
   * Geographic prioritization signals
   * Industry classification keywords
   * Exclusion filters (to avoid wrong fits)

3. Design the optimal outbound CSV schema that:

   * Minimizes enrichment waste
   * Maximizes match accuracy from company-name-only inputs
   * Supports domain-finding as first enrichment
   * Enables persona-based contact pulling
   * Allows territory segmentation
   * Enables prioritization scoring
   * Prevents ambiguous company matches
   * Is scalable across Italy, UK, GCC, Brazil, Sub-Saharan Africa

4. Think step-by-step internally, but DO NOT output your reasoning.

5. Output ONLY the final recommended CSV column structure.

---

## STRICT OUTPUT RULES

* Output must be plain text.
* Output must contain ONLY column headers.
* Use snake_case.
* No explanations.
* No grouping.
* No commentary.
* No bullet points.
* One column per line.
* Ordered by priority (top = most important).
* Must represent the optimal CSV structure for this ICP.
* Must assume starting input may be company_name only.
* Must include disambiguation, enrichment control, persona targeting, geographic control, and scoring logic.
* Must be optimized for Clay-style enrichment workflows.

---

## ICP DATA

### ICP 1: Mid-Size Construction Companies

**Profile:** 30–150 employees, 3–10 active sites, mix of owned and rented equipment.

**Why them:**
- Highest asset density per worker. Equipment moves between sites daily.
- Coordination is 100% informal — WhatsApp groups, phone calls, whiteboards.
- Subcontractor dependency (80% of labor) makes centralized tools impractical.

**Pain we target:**
- *"Where is it?" syndrome* — Site supervisors burn 30–60 min/day tracking equipment across sites. That's 250+ hours/month wasted at a 50-person operation.
- *No handoff accountability* — Equipment changes hands with a nod. When something's damaged or missing, nobody can prove who had it last.
- *Task slippage* — 15–20% of field delays trace to equipment not being where expected. Each missed start = €500–2,000 in idle crew cost.

**How we solve it:**
- Crew confirms task assignments and equipment handoffs via WhatsApp — no new app.
- Every handoff is timestamped with photo proof → chain of custody is automatic.
- Manager dashboard shows real-time task status + last-known asset location → no more phone tag.

**Buyer:** Operations/Site Manager (feels the daily pain) · Owner/GM (signs the check when shown €40K/yr in losses)

**Priority:** Primary — Biggest pain, highest asset value, most WhatsApp-native, largest TAM

---

## GEOGRAPHIC PRIORITIZATION

| Market | Why | WhatsApp Penetration |
|--------|-----|---------------------|
| Italy | Home market, network advantage, Italian-language moat | High |
| UK | English-speaking, SMB-dense trades sector | High in trades |
| UAE + Saudi | Construction boom, high willingness to pay | Very high |
| Brazil | Largest WhatsApp market, massive construction/logistics | Dominant |
| Sub-Saharan Africa | WhatsApp-first economies, growing construction | Dominant |

---

## COMMON THREAD

**The universal insight:** Field teams already coordinate on WhatsApp. The problem isn't communication — it's that nothing is structured, tracked, or accountable. ShiftBuddy doesn't replace WhatsApp. It makes WhatsApp *work* as an ops system.

**The adoption unlock:** Zero app installs. Zero training. If your crew can send a WhatsApp message, they can use ShiftBuddy. That's why we win where Jobber, ServiceTitan, Samsara, and every enterprise EAM/CMMS fails — adoption.

---

Now analyze the ICP data above and produce the final CSV structure.
