# ShiftBuddy — CSV Research: Facility Management Companies

> **Last updated:** 2026-02-16
> **Purpose:** Prompt for an AI research specialist to analyze the Facility Management ICP and produce an optimal outbound CSV schema for Clay-style enrichment workflows

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

### ICP 5: Facility Management Companies

**Profile:** 40–200 employees, managing multiple client sites (offices, malls, hospitals).

**Why them:**
- SLA-driven contracts where missed maintenance = financial penalties.
- Equipment (cleaning rigs, HVAC units, maintenance tools) rotates across client sites.
- High staff turnover means tribal knowledge disappears constantly.

**Pain we target:**
- *SLA compliance risk* — Scheduled maintenance gets missed because task assignment is verbal. Client finds out before the FM company does.
- *Asset rotation chaos* — Equipment moves between 10+ client sites with no log. "Where's the floor scrubber?" becomes a daily question.
- *Onboarding friction* — New hires take 2+ weeks to get productive because processes live in people's heads, not systems.

**How we solve it:**
- Scheduled tasks pushed via WhatsApp with confirmation required → missed tasks surface immediately on dashboard.
- Asset check-in/check-out at each client site via WhatsApp → rotation history is automatic.
- Standardized task flows in WhatsApp = new hires follow the same process from day one.

**Buyer:** Operations Director · Contract/Client Manager

**Priority:** Tertiary — Good fit but longer sales cycle, SLA angle needs case studies first

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
