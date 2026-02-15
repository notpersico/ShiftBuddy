# 06 — MVP Personas

> These are the only people who matter for the MVP. Everyone else is phase 2.

---

## Primary: Marco — The Buyer and Only User

**Operations Manager, 42. Construction company, 85 employees, 6 sites.**

Marco is the product. If Marco doesn't use it, nobody does. There is no second user in the MVP — no worker app, no owner dashboard, no team lead view. Marco opens it, Marco enters data, Marco gets value. Alone.

### What he does today

Wakes at 5:45. Scrolls WhatsApp by 6:10 — three groups, 40+ unread. Somewhere in there, a machine broke down at 17:30 yesterday. He finds out now. Calls Luigi. No answer. Calls Davide. Gets a maybe. Drives to the wrong site. Loses 90 minutes before the day starts.

He keeps a whiteboard, a spreadsheet, and three WhatsApp groups. None of them agree. When equipment moves between sites, it enters a black hole until someone physically sees it. He is the human router between chaos and action, and he's burning out.

### What he needs from ShiftBuddy

One thing: **open the app and know where his stuff is.**

Not tasks. Not scheduling. Not communication. He needs to search "CAT 320" and get: Site Roma Nord, held by Giovanni since Tuesday, photo of condition at handoff. Done. The 45-minute phone chain replaced by a 5-second search.

### Why he'll do the data entry himself

Because he's already doing it — badly. His spreadsheet has columns for equipment location. His WhatsApp messages are ad-hoc status updates. He's already the data entry person. ShiftBuddy isn't adding work. It's replacing a worse version of the same work with something that actually retains information.

The WhatsApp import eliminates the cold start. He doesn't manually enter 50 assets on day one — he drops his group chat export and confirms what the AI found. From there, logging a handoff takes 15 seconds: tap asset, tap person, snap photo, done.

### Why he'll pay

He lost a €2,000 generator last month. Nobody knows who had it. His boss asked for a list of equipment by site — he spent 3 hours building it from memory and WhatsApp scrolling. He knows this is insane. He's been looking for something simple enough that he'll actually use it. Everything else is either a $500/mo enterprise platform or a generic project management tool that his crew ignores.

€49/mo to stop the bleeding is a rounding error on his equipment budget.

### His objections

- "I tried [tool X], nobody used it." → Nobody needs to use this except you. Your crew doesn't install anything.
- "I don't have time to set this up." → Drop your WhatsApp export, review what we found, add photos. 20 minutes.
- "My boss won't approve another subscription." → Show him the €40K write-off line and the €600/year price tag.
- "What if I stop updating it?" → That's the argument for the bot. But for now, 15 seconds per handoff is less than the WhatsApp message you're already sending.

### His aha moment

Day 5. Someone calls asking where the plate compactor is. He opens ShiftBuddy, searches, finds it in 3 seconds with a photo showing it at Cantiere Tivoli, held by Ahmed since Monday. He answers the call in 10 seconds flat. Previously this was 20 minutes and 4 phone calls.

He'll never go back to not knowing.

---

## Shadow Persona: Antonio — The Check-Signer

**Owner, 54. Field services company, 60 employees.**

Antonio doesn't use the product. He doesn't log in. He doesn't know it exists until Marco shows him the CSV export.

### Why he matters

Antonio approves the €49/mo. That's it. His decision criteria are simple:

1. Does it save more than it costs? (€600/year vs €40K in losses — trivial)
2. Does it require me to do anything? (No.)
3. Can I see proof? (CSV export with custody log. Photo evidence of handoffs.)

Antonio doesn't need a demo. He needs Marco to say "boss, I found a thing that tracks our equipment for €49/mo and I've already been using it for two weeks."

### What converts him

The first insurance claim or client dispute where Marco pulls up a timestamped photo trail. That's when Antonio stops questioning the subscription and starts asking "can we put this on ALL the sites?"

---

## Anti-Persona: José — The Worker Who Doesn't Exist Yet

**Equipment operator, 29. Won't download an app. Period.**

José is not a user. José is not a consideration. José does not touch the MVP.

### Why he's documented anyway

Because the temptation to build for José is the single biggest scope risk. Every feature request that starts with "but what if the workers could..." is a trap. The bot, the crew interface, the mobile-first worker experience — all of that is phase 2, triggered only when paying Marcos independently ask for it.

José's existence in the PRD is a warning label: **build for Marco first, prove Marco will pay, then earn the right to build for José.**

### When José enters the picture

When 3+ paying customers say: "This is great but I'm tired of being the only one entering data. Can my workers do handoffs from WhatsApp?"

That's the signal. Not before. Let demand pull the feature.

---

## Anti-Persona: Serena — The Power User Who Isn't Ready

**Fleet & Asset Manager, 37. 120 employees, 800+ tracked assets, 47-tab spreadsheet.**

Serena needs ShiftBuddy more than anyone. She also needs features the MVP doesn't have: bulk operations, advanced filtering, utilization reports, maintenance scheduling, multi-site views with dozens of assets per site.

### Why she's out of scope

Serena's needs are real but premium. Building for Serena means 3x the surface area: advanced tables, batch operations, reporting engine, role-based views. That's a Pro/Business tier feature set, not an MVP.

More importantly: Serena is rare. There are 100 Marcos for every Serena. Marcos run 30-80 person companies with 20-50 pieces of equipment. That's the volume market. Serena runs a fleet operation that needs a fleet management system. ShiftBuddy becomes that eventually, not on day one.

### When Serena enters the picture

When the Pro tier (200 assets, 30 team members) starts feeling tight for 5+ customers. When managers ask for reports that CSV export can't satisfy. When someone says "I need my fleet manager to have her own login with different permissions." That's Serena's cue.

---

## Persona Map

```
MVP (now)
━━━━━━━━━━━━━━━━━━━━━
  Marco ← sole user, sole buyer
    │
    └→ Antonio ← approves budget (sees CSV/export only)


Phase 2 (demand-triggered)
━━━━━━━━━━━━━━━━━━━━━
  Marco ← dashboard user
    │
    ├→ José ← WhatsApp bot user (handoffs, check-ins)
    │
    ├→ Antonio ← dashboard viewer (reports, KPIs)
    │
    └→ Serena ← power user (fleet ops, analytics)
```

Every phase transition is triggered by paying customer demand, not roadmap ambition.
