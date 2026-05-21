# CSAH — Chatham Savannah Authority for the Homeless
## Project Context for Claude Code

This file provides persistent context for AI-assisted development.
Place at the root of your project repository.

---

## Client Overview

**Client:** Chatham Savannah Authority for the Homeless (CSAH)
**Website:** homelessauthority.org
**Type:** 501(c)(3) nonprofit, founded 1989 by the Georgia Legislature
**Mission:** Lead the end of homelessness in Chatham County, GA
**Contact:** 912-813-4614

**Scale:** As of FY'25 Point-in-Time count — **628 people experiencing homelessness** in Chatham County; 73% unsheltered. Chronic homelessness rate is 11% (well below the 30% national average).

**What CSAH does:**
- Runs the Interagency Council on Homelessness
- Coordinates the city's Coordinated Entry system
- Operates street outreach teams (Streets to Stability, Project Hope Water)
- Develops housing (The Cove at Dundee, Dundee Cottages for veterans and families)
- Runs needs assessment/triage at Union Mission, 120 Fahm St (M–T 8:30am–4pm, F 8:30am–12pm)
- Outreach services Mon–Sat 8:30am–9pm

---

## Existing System — The Compass Project

A prior SCAD Serve team built and delivered a working web app for CSAH.

**Live URL:** `thecompassproject.framer.website`

**Pages:** `/welcome`, `/about`, `/feedback`, `/getid`, `/getcaseworker`, and a full `/resources` section with sub-pages:
- `/resources/directory`
- `/resources/meals`
- `/resources/day-centers`
- `/resources/showers`
- `/resources/healthcare`
- `/resources/shelters`
- `/resources/transportation`

**Special alert system:** Emergency pop-up overlays (weather, safety alerts) managed in Framer.

### Current Tech Stack

| Layer | Tool | Notes |
|---|---|---|
| UI/Website | **Framer** | All page content edited here |
| Maps & Widgets | **Common Ninja** | Resource maps, weather, translation, feedback form |
| Kiosk Lockdown | **KioWare** | Restricts device to Compass Project + Google Maps |
| Kiosk Remote Mgmt | **KioCloud** | Remote monitoring, updates, analytics across kiosk network |
| Screen Reader | **JAWS** (Freedom Scientific) | Accessibility on kiosks |

**Analytics:** Page visits via Framer; engagement clicks via Common Ninja. No PII collected from users currently.

**Staff handoff:** A SCAD-authored instruction booklet exists covering how to update Framer content, manage Common Ninja widgets, configure KioWare, and use KioCloud. CSAH staff are expected to self-maintain.

**Existing CSAH website:** WordPress, maintained manually by Aaron (marketing team). Separate from Compass Project / Framer site.

### Resource Content (digitizes the printed CSAH trifold)
- Day Centers (Union Mission, Salvation Army, The Dive, Come As You Are, Family Promise)
- Meals, Showers, Laundry (Old Savannah City Mission, Emmaus House, Savannah Baptist Center, etc.)
- Emergency Shelters (8 locations — adults, families, youth, women, men, DV survivors)
- Transportation (Free CAT bus + CSAH emergency transport)
- CSAH's own services

---

## Ways of Working

### Team Roles
- **Alan Robinson** (alan@alanrobinson.co) — client-facing lead / consultant. Manages PMO functions: client relationship, proposal, grant process, scheduling.
- **Allyson Short** — technical lead / developer. Owns architecture, build, kiosk implementation, and technical decisions.

### Sharing Docs with Alan
Alan is not a software developer. He is a visual/UI person. Decision documents are written as **styled HTML files** stored in the `docs/` folder and served via **Netlify** (connected to the private GitHub repo) so Alan can open them in a browser from a URL. Do not share raw markdown, code files, or anything requiring technical context to read.

- All decision/research docs live in `docs/` as `.html` files
- When creating a new decision doc, follow the visual style established in `docs/pwa-vs-mobile.html` (dark header, card-based layout, color-coded columns, CSAH Impact callout rows)
- Netlify publish directory is set to `docs/` — every push to main auto-deploys
- Netlify URL: `csahcompassapp.netlify.app` — e.g. `csahcompassapp.netlify.app/kiosk_comparison.html`
- A `docs/index.html` landing page listing all docs is the intended navigation surface for Alan
- Repo is **private** on GitHub — Netlify free tier handles deployment from private repos

### Honesty & Confidence Levels
- **Never hallucinate.** If the answer isn't known, say so directly.
- **Flag partial answers.** If a response includes guesses, assumptions, or things that should be verified, call them out explicitly — don't bury them in otherwise confident-sounding text.
- This applies especially to: vendor pricing, API capabilities, government policy details, and any claims about third-party tools.

### Documentation Pattern
When Allyson and Claude work through a technical decision, the output is a styled HTML decision doc. Structure:
1. Context box (what problem are we solving, who are the users)
2. Key context callout if there's a framing caveat (e.g. "SMS is already in the plan")
3. Factor-by-factor comparison with 3 columns (option A / option B / option C) + CSAH Impact row per factor
4. Summary cards — one per option, clearly labeled with recommendation signal
5. Recommendation section (dark background) — direct lean with reasoning
6. Open questions list — numbered, specific, actionable

---

## Project Scope (This Engagement)

### Philosophy
Keep it lean. CSAH is a nonprofit with a small budget. No overengineering. Low recurring costs are a priority. Replit is a candidate hosting platform. Not a big hosted AWS deployment to start.

### Grant Funding
- **$150,000 SCAD grant** covers both app development AND kiosk hardware/software
- **Must spend all funds by December 2026**
- **SCAD grant structure:** Reimbursement model — requires purchase orders and invoices. No detailed line-item scrutiny; treats kiosk + app as a connected ecosystem.
- SCAD has a new AI department and wants a "flashy" innovation showcase — lean into AI features in the presentation.
- **Sustainability:** 3–5 year maintenance funding plan needed beyond initial deployment.

### Budget Breakdown (Rough)
- Kiosk hardware: $10k–25k per unit
- Kiosk software licensing: ~$2k/year per unit
- Three priority kiosk locations identified (see Kiosk section)
- Remaining budget available for app development

### Two Distinct User Surfaces

**1. Partner App (~50 users)**
- Users: CSAH staff, HOPE unit officers, partner agency case workers
- Boots-on-the-ground teams across multiple agencies
- Track cases, coordinate across agencies, push resources to individuals
- Homeless individuals do NOT have app access

**2. Kiosk (future — separate discussion)**
- Public-facing touchscreen stations
- Where homeless individuals will interface with the system
- Architecture TBD pending more context

---

## Core App Functions

1. **Push notification system for crisis coordination**
   - SPD HOPE unit → Homeless Authority street outreach team
   - CCPD behavioral health units, city/county code enforcement, community paramedicine
   - Includes geolocation — maps outreach locations, proves contact was made before enforcement
   - "Truth telling tool": data counters negative narratives about homeless population to city/county gov

2. **Case tracking**
   - Each homeless individual is a "case." Interactions (who made contact, what was offered, outcome) get logged.
   - Communication thread stays active until case is manually closed
   - Closure via dropdown — outcome options (e.g., housed, declined, referred)
   - Builds dataset showing intervention volume, outcomes, resource utilization
   - Primary goal: demonstrating compassionate diversion from arrest

3. **Embedded resource directory**
   - For officers and workers not specialized in homeless services
   - Faster than QR codes and paper handouts currently in use

4. **Partner ↔ partner coordination**
   - Shared messaging tied to case records, not personal phones
   - Multi-agency (HOPE, CSAH, partner orgs) communication in context of a specific individual

5. **Partner → homeless individual (outbound SMS) — future**
   - Worker sends a text with resource links/info to an individual's phone number
   - Inbound replies should thread back to the case record
   - Individuals do not have app logins
   - 90% of homeless population has smartphones for charging/communication

---

## Kiosk — Planned (Separate Discussion)

**Three priority locations ("easy wins"):**
1. Goodwill Opportunity Campus — 761 Wheaton Street (already wants one)
2. Union Mission Resource Center — across from Greyhound, emergency shelter area
3. Public library — Bull Street or Southside branch

**Installation notes:**
- Private nonprofit property avoids city permitting delays
- City manager relationship enables fast-track for public property if needed
- No property use fees expected — organizations see value for their populations

**Future kiosk → app integration:**
- Kiosk user requests shelter → push notification to appropriate partner team

---

## Kiosk Software — Options Under Consideration

**SCAD recommendation:** KioWare (already in use on Compass Project kiosks).

### Key Requirement: Single Data Source for Kiosk + App
This is primarily an **architecture concern, not a kiosk software concern.** Kiosk lockdown software restricts the device to a URL — if the kiosk runs the same PWA as the partner app (pointing to the same backend API), any update to the data source propagates to both surfaces automatically, on demand. No special kiosk software feature is required for this. The choice of kiosk platform does not gate the single-data-source goal.

The one exception: platforms with a built-in CMS. If the kiosk software manages content (rather than just locking down a URL), you'd need to evaluate whether that CMS can also serve the app layer. For this project, the cleaner path is to own the data layer in the app backend and let the kiosk software just do lockdown + remote management.

---

### Comparison: KioWare vs. Alternatives

**Pricing note:** Figures below are estimates based on publicly available information as of mid-2026. Verify directly with vendors before committing — nonprofit pricing and bundling can significantly change the numbers.

---

#### KioWare *(SCAD-recommended, currently deployed)*
- **Est. cost:** ~$2k/year per device (from existing budget estimate; includes KioCloud remote management)
- **Pros:** Already in use; SCAD staff familiar with it; proven on this hardware; JAWS/accessibility compatible; KioCloud handles remote monitoring + updates across kiosk network
- **Cons:** Higher per-device cost than MDM alternatives; two-product stack (KioWare + KioCloud); licensing model can feel dated compared to modern MDMs
- **CSAH fit:** Known quantity, lowest switching risk. But at ~$2k/device/year × 3 kiosks = ~$6k/year ongoing.

---

#### Scalefusion *(Modern MDM + kiosk, all-in-one)*
- **Est. cost:** ~$25–50/device/year *(significantly cheaper than KioWare; verify nonprofit pricing)*
- **Pros:** Bundles kiosk lockdown + remote management in one platform (replaces both KioWare + KioCloud); modern web console; strong Android + Windows support; good for web app/PWA kiosks; on-demand remote push updates built in
- **Cons:** Less specialized for kiosk use cases than KioWare; may require more configuration for touchscreen/public-facing hardening; less known quantity for this hardware setup
- **CSAH fit:** Best cost-efficiency of the alternatives. If hardware runs Android or Windows and is PWA-compatible, this could cut ongoing software costs by 90%+. Worth a trial.

---

#### Hexnode MDM *(Lean MDM with kiosk mode)*
- **Est. cost:** ~$15–30/device/year *(among the cheapest options; verify)*
- **Pros:** Solid kiosk mode + MDM in one; remote device management included; good web app / browser kiosk support; reportedly strong nonprofit pricing; simple setup
- **Cons:** Less specialized than KioWare for public-facing kiosk hardening; fewer kiosk-specific features (e.g., attract screen, session reset timers) out of the box; smaller ecosystem
- **CSAH fit:** Fine if the kiosk is just a locked-down browser pointing to a URL. Not the right call if you need advanced kiosk UX behaviors (idle reset, attract loop, etc.) without custom config work.

---

#### SiteKiosk Online *(by Provisio — direct KioWare competitor)*
- **Est. cost:** ~$150–300/device/year *(perpetual + maintenance model; verify current SaaS pricing)*
- **Pros:** Purpose-built for public-facing web kiosks; direct KioWare competitor with comparable feature set; includes content scheduling + remote management; strong browser lockdown features; good accessibility track record
- **Cons:** Less widely adopted than KioWare in nonprofit/government contexts; pricing model less transparent; less familiar to SCAD staff who may support CSAH
- **CSAH fit:** Closest feature-for-feature swap for KioWare at lower cost. Worth evaluating if SCAD support isn't a dependency — but staff familiarity advantage of KioWare disappears if SCAD isn't maintaining it long-term.

---

### Summary

| Option | Est. Annual Cost (3 kiosks) | Remote Mgmt Included | Single Data Source | Notes |
|---|---|---|---|---|
| KioWare + KioCloud | ~$6k/yr | Yes (KioCloud) | Via app architecture | SCAD-recommended; known quantity |
| Scalefusion | ~$75–150/yr | Yes (built-in) | Via app architecture | Best value; modern MDM |
| Hexnode | ~$45–90/yr | Yes (built-in) | Via app architecture | Leanest option; less kiosk-specialized |
| SiteKiosk Online | ~$450–900/yr | Yes (built-in) | Via app architecture | Closest KioWare alternative |

**Open question:** If CSAH staff will self-maintain post-handoff, KioWare's familiarity advantage matters. If Allyson + Alan own ongoing maintenance, switching cost is lower and Scalefusion's price point is compelling.

---

## Data Notes

- **HMIS integration: NOT now, but desired long-term.** All 76 partner agencies feed into Caseworthy platform. 2-year API approval process due to strict HUD regulations. Building custom DB for now.
- **Not integrating with HUD HMIS** for MVP
- Building a **custom DB** for the app
- Homeless individual population is fluid — ~600 in Savannah currently, rotating
- No PII collection from homeless individuals on the kiosk

---

## Key Stakeholders

| Name | Role | Contact |
|---|---|---|
| Jennifer Dulong | CSAH (primary contact) | jdulong@homelessauthority.org |
| Kishia Young | CSAH | kishia@homelessauthority.org |
| McKaylin Zukowski ("Kay") | CSAH | kay@homelessauthority.org |
| Brianna Magoon | CSAH | brianna@homelessauthority.org |
| Keisha | HMIS compliance director | TBD |
| Aaron | CSAH marketing team | TBD |
| Jen (Renegade) | SCAD grant process | TBD — 3-way call planned |

---

## SMS — Decision Pending

**Decision: NOT finalized.** Leaning Twilio.

### Use Cases
- Partner ↔ partner messaging tied to case records (not personal phones)
- Partner → homeless individual outbound (resource info + links)
- Potential inbound reply threading back to case record

### Options Evaluated

| Provider | Est. Cost | Notes |
|---|---|---|
| **Twilio** | ~$20–50/mo at this volume | Industry standard. Best API. Handles 2-way threading. Works with Replit. Frontrunner. |
| **Textbelt** | ~$25/500 texts (credit) | Dead simple, great for low-frequency outbound only. No real 2-way support. |
| **Vonage (Infobip)** | Comparable to Twilio | Solid alternative, slightly more nonprofit-friendly. No meaningful cost difference at this scale. |

**Estimated volume:** ~250 notifications/month (small). At this scale, cost is negligible.

### Why Twilio is the likely call
- 2-way threading needed for partner↔individual replies
- Cost at this volume is negligible for a nonprofit
- Works with lightweight hosting like Replit
- All messages logged against case records, not personal phones

---

## Decisions Log

| Date | Decision | Rationale |
|---|---|---|
| 2026-05-08 | No HMIS integration for now | 2-year approval process; build custom DB instead; revisit later |
| 2026-05-08 | Hosting philosophy: keep it lean | Nonprofit budget; low recurring costs; Replit candidate |
| 2026-05-08 | SMS provider TBD | Leaning Twilio; ~250/mo estimated volume |
| 2026-05-08 | Budget: $150k SCAD grant | Must spend by Dec 2026; kiosk hardware $10–25k/unit |
| 2026-05-08 | Push notifications are core | Not just SMS — real-time push between partner agencies is the primary coordination mechanism |
| 2026-05-08 | No kiosks purchased yet | Allyson + Alan own procurement and implementation by Dec 2026 |
| 2026-05-08 | No budget split scrutiny | SCAD treats kiosk + app as a connected ecosystem; no line-item breakdown required |
| 2026-05-08 | Geolocation = one-time pin drop only | Not continuous tracking; officer logs location at moment of contact; minimal consent/union exposure |
| 2026-05-08 | App platform leaning PWA + SMS | SMS fallback neutralizes main PWA weakness (iOS push reliability); avoids gov't MDM distribution problem; Expo is upgrade path only if device policies allow |
| 2026-05-08 | Decision docs shared via GitHub Pages | Alan is non-technical/visual; styled HTML in docs/ folder served via GitHub Pages |

---

## Open Questions

- [ ] Gov't device policies: can SPD HOPE and CCPD officers install third-party apps on issued devices? (Gates PWA vs Expo decision)
- [ ] SMS provider final decision — leaning Twilio
- [ ] Is push + SMS duplication too invasive? Should notification channel be a user preference (push only / SMS only / both)?
- [ ] Confirm Twilio cost at combined volume (resource SMS + push fallback)
- [ ] Kiosk architecture — separate discussion, TBD
- [ ] Does CSAH have working logins to Framer and Common Ninja?
- [ ] Contact info for Keisha (HMIS compliance) and Aaron (marketing)
- [ ] Three-way call with Jen (Renegade) re: SCAD grant process — schedule
- [ ] Case identity: request HMIS data dictionary from Keisha for custom DB schema reference

---

*Last updated: 2026-05-08 — Architecture decisions, SMS fallback strategy, and ways of working documented*
