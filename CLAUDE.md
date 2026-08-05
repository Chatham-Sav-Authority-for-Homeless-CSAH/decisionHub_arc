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

## Street Outreach Programs

Source: homelessauthority.org/street-outreach/

**Approach:** Trauma-informed, evidence-based. Meets people where they are to build trust with service-resistant individuals and connect them to safe shelter, services, and housing via case management.

### Programs

**Streets to Stability**
Street outreach and case management. Funded by the City of Savannah. Operates Mon–Sat 8:30am–9pm.

**Project Hope Water**
Mobile shower unit with multiple showers including ADA-accessible facilities. "Radical hospitality" model — connects unhoused residents to basic services. Funded by Chatham County and City of Savannah via American Rescue Plan Act.

**Inclement Weather Response**
Coordinated emergency services during extreme weather — temporary shelter coordination and transportation for individuals with complex medical/mental health needs.

**Encampment Engagement**
"Compassionate encampment management and closing plan" — designed to preserve resident dignity while addressing community concerns.

### 2023–2025 Impact
- 42 days of inclement weather support (2024)
- 55% encampment reduction (87 → 39 locations)
- Leads the annual Point-in-Time Count
- Transportation services connecting residents to medical care, shelter, and food

### Street Outreach Partners
HUD, Chatham County, City of Savannah, Chatham Emergency Management Agency, emergency shelters, behavioral health providers, code enforcement, local government agencies.

---

## Existing System — The Compass Project

A prior SCAD Serve team built and delivered a working web app for CSAH. We wil not keep this work, this is just a baseline for us to start real software development.

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

### Team Roles at ARC Consulting
- **Alan Robinson** (alan@alanrobinson.co) — client-facing lead / consultant. Manages PMO functions: client relationship, proposal, grant process, scheduling.
- **Allyson Short** — technical lead / developer. Owns architecture, build, kiosk implementation, and technical decisions.

### Sharing Docs with Alan during development
Alan is not a software developer. He is a visual/UI person. Decision documents are written as **styled HTML files** stored in the `docs/` folder and served via **Netlify** (connected to the private GitHub repo) so Alan can open them in a browser from a URL. Do not share raw markdown, code files, or anything requiring technical context to read.

- All decision/research docs live in `docs/` as `.html` files
- When creating a new decision doc, follow the visual style established in `docs/pwa-vs-mobile.html` (dark header, card-based layout, color-coded columns, CSAH Impact callout rows)
- Netlify publish directory is set to `docs/` — every push to main auto-deploys
- Netlify URL: `csahcompassapp.netlify.app` — e.g. `csahcompassapp.netlify.app/kiosk_software_comparison.html`
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
Keep it as lean as possible but with an eye for expansion. CSAH is a nonprofit with a small budget. No overengineering. Low recurring costs are important. See `docs/hosting-comparison.html` for the hosting platform comparison (Netlify recommended). Not clear on what the CI/CD platform should look like as the end result will be very non technical folks needing GUIs to update content only. If they need any feature adds or changes, ARC consulting will be doing the work. Will need to produce a prototype with Alan.

### Grant Funding
- **$150,000 SCAD grant** covers both app development AND kiosk hardware/software
- **Must spend al 
l funds by December 2026**
- **SCAD grant structure:** Reimbursement model — requires purchase orders and invoices. No detailed line-item scrutiny; treats kiosk + app as a connected ecosystem.
- SCAD has a new AI department and wants a "flashy" innovation showcase — lean into AI features in the presentation.
- **Sustainability:** 3–5 year maintenance funding plan needed beyond initial deployment.

### Budget Breakdown (Rough)
- Kiosk hardware: $10k–25k per unit
- Kiosk software licensing: ~$2k/year per unit
- Three priority kiosk locations identified (see Kiosk section)
- Remaining budget available for app development

### Two Distinct User Surfaces — One Supabase Data Source

Both surfaces pull from the same Supabase backend. Resource directory data, weather/emergency alerts, and service availability are authored once and consumed by both. The surfaces differ fundamentally in audience, interaction model, and feature set.

**1. CSAH Staff App (PWA) — CSAH outreach team (MVP / test run)**
- Audience (MVP): CSAH outreach staff and case managers only. Partner agencies (SPD HOPE, CCPD behavioral health, code enforcement, paramedicine) are a future expansion — CSAH is the test pilot and needs to prove the tool internally first.
- Full-featured PWA: case tracking, CSAH-internal messaging, push notifications, management dashboard, geolocation, resource directory, embedded survey tool
- Homeless individuals do NOT have app access
- Resource directory is embedded here for quick in-person reference — same Supabase data as kiosk, different UI context (compact, search-first, designed for quick lookup during field contact)
- Weather/emergency alerts: presentation TBD — workers need to know an alert is active and coordinate the response

**2. Kiosk App — unhoused individuals**
- Audience: people experiencing homelessness at public outdoor touchscreen stations
- Simplified, touch-optimized interface; primary function is the resource directory
- SPA vs. lightweight PWA: still under evaluation. The kiosk does not need to be "installable," but a service worker for offline caching may be worth including (resource directory loads from cache if connectivity drops). Not yet decided.
- Kiosk lockdown software (Scalefusion/KioWare/etc.) is a container only — it locks the browser to the kiosk URL and handles remote device management; it does not touch the data layer
- Weather/emergency alerts: presentation TBD — a prominent alert experience for kiosk users is needed; the specific format (full-screen overlay, persistent banner, etc.) is still being evaluated

**Resource Directory — Shared Across Both Surfaces**
- Lives in Supabase; maintained by resource providers via login access to their data
- Kiosk: the primary (and nearly only) user experience
- Partner app: embedded tool for workers during field contact with individuals
- Same data, different presentation — no sync required, no duplication

**Architecture Lean (as of June 2026)**
- Approach: Pattern 1 — custom web app running on kiosk hardware via lockdown browser. Not a kiosk content platform (Pattern 2/CMS approach). CSAH needs custom UI, complex data model, and shared code between surfaces.
- Structure: single app, two entry points — one codebase, kiosk routes (`/kiosk/*`) and partner routes (`/app/*`) as separate UI shells over shared Supabase hooks and resource directory components. Avoids duplicating resource directory code; route guards prevent kiosk users from accessing partner views. See `docs/app-architecture.html`.

---

## Core Partner App Functions

1. **Push notification system for crisis coordination**
   - Within CSAH outreach team — internal coordination among CSAH staff for MVP
   - Partner agency integration (SPD HOPE, CCPD, code enforcement, paramedicine) is a future phase; CSAH proves the tool internally first
   - Includes geolocation — maps outreach locations, proves contact was made before enforcement

2. **Case tracking**
   - Each homeless individual is a "case." Interactions (who made contact, what was offered, outcome) get logged.
   - Communication thread stays active until case is manually closed unless automated way can be found
   - Closure via dropdown — outcome options (e.g., housed, declined, referred)
   - Builds dataset showing intervention volume, outcomes, resource utilization
   - Primary goal: demonstrating compassionate diversion from arrest

3. **Embedded resource directory**
   - For CSAH outreach workers in the field — quick lookup during contact with individuals
   - Faster than QR codes and paper handouts currently in use

4. **Team coordination (CSAH-internal for MVP)**
   - Shared messaging tied to case records, not personal phones — CSAH team only for initial deployment
   - Multi-agency expansion (HOPE, partner orgs) is a future roadmap phase

5. **Point-in-Time count survey tool**
   - Embedded survey for the annual 10-day Point-in-Time count period
   - Survey questions to be scoped with Kishia; administered via the app rather than paper
   - Once-per-year feature but critical for CSAH data collection and grant reporting

6. **Mgmt Dashboard**
   - The ED Jen, wants to use good data in a persuasive way to   secure more budget and donations. She wants to tell the story of  how their team is responding with compassion and care. 
   - "Truth telling tool": data counters negative narratives about homeless population to city/county gov

7. **Staff → homeless individual (outbound SMS) — future**
   - Worker sends a text with resource links/info to an individual's phone number
   - Inbound replies should thread back to the case record
   - Individuals do not have app logins
   - 90% of homeless population has smartphones for charging/communication

---

## Kiosks —

**Three confirmed priority locations:**
1. Union Mission Resource Center — 120 Fahm St, outdoor placement (accessible 24/7 even when resource center is closed; clients banned from the property can still access the kiosk from outside)
2. Goodwill Opportunity Campus — 761 Wheaton Street (has already said yes)
3. Public library — Bull Street branch

**Potential additions (Jen pursuing):**
- Greyhound bus station — High PR/proof-of-concept value; entry point for transient unhoused population coming into Savannah. Jen will go through city manager to get approval. If approved, may become Kiosk 1 for the launch PR event; Union Mission is Plan B.
- Memorial hospital — Jen's future priority: high volume of unhoused individuals discharged with no housing plan.

**Staged rollout under consideration:**
Alan proposed launching 1 kiosk as a high-visibility PR proof-of-concept (possibly Greyhound), then paying for and scheduling kiosks 2 and 3 before December 2026 but installing in 2027. Jen reviewing SCAD grant requirements before committing.

**Installation notes:**
- Private nonprofit property avoids city permitting delays
- City manager relationship enables fast-track for public property if needed
- No property use fees expected — organizations see value for their populations

**Future kiosk → app integration:**
- Kiosk user requests shelter → push notification to CSAH outreach team

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

## Analytics & Reporting Architecture

Two distinct reporting surfaces, two different tools. The line between them: **Plausible handles behavioral signals that don't exist in the schema; Supabase views handle operational data that already does.**

### Kiosk → Plausible Analytics
- **Tool:** Plausible Analytics ($9/month cloud)
- **Why:** Cookieless, privacy-first, no PII leaves CSAH control, GDPR-compliant, no cookie banner required, dashboard legible without training
- **What Plausible gives out of the box:** Sessions, unique visitors, session duration, page-level views, date range filtering, CSV export
- **Kiosk location segmentation is free:** Path-based location IDs (`/kiosk/union-mission/*`, `/kiosk/goodwill/*`) appear as distinct paths in Plausible automatically — no custom event wiring needed for location filtering
- **Custom events to instrument:**
  - `Resource Clicked` → `{props: {category: 'shelter', name: 'Union Mission'}}`
  - `SMS Sent from Kiosk`
  - `QR Code Displayed`
  - `Search Performed` → `{props: {term: '...', result_count: N}}`
  - `Search Returned No Results`
  - `Alert Viewed / Acknowledged`
  - `Accessibility Feature Toggled` (large text, audio — only if built as explicit in-app toggles, not OS-level)

### CSAH Staff App → Postgres Views (Supabase)
- **Tool:** Supabase Postgres views and functions — no third-party analytics
- **Why:** Everything Jen wants (referrals by staff, encounters by location, open cases, resource frequency) already exists as structured records in the schema. Adding Mixpanel/Amplitude would mean paying to duplicate data you own, and shipping case-adjacent sensitive data to a third party.
- **Pattern:** Create Supabase views (`referral_summary_by_staff`, `encounter_density_by_region`, `open_referrals_by_agency`). React dashboard fetches from these views via the Supabase client like any other data call.
- **Map visualizations:** Mapbox GL JS or Leaflet with react-leaflet. Encounter coordinates are already stored from one-time pin drops — a heat map is just feeding those coordinates to a rendering library.
- **Exports:** CSV is a one-function utility from JSON. Excel via SheetJS. PDF via Playwright/Chromium. No third-party service needed.

---

## Data Notes

- **HMIS integration: NOT now, but desired long-term.** All 76 partner agencies feed into Caseworthy platform. 2-year API approval process due to strict HUD regulations. Building custom DB for now is likely needed
- **Not integrating with HUD HMIS** for MVP
- Homeless individual population is fluid — ~600 in Savannah currently, rotating
- No PII collection from homeless individuals on the kiosk

---

## Key Stakeholders

| Name | Role | Contact |
|---|---|---|
| Jennifer Dulong | CSAH (executive director) | jdulong@homelessauthority.org |
| Kishia Young | HMIS compliance director | kishia@homelessauthority.org |
| McKaylin Zukowski ("Kay") | CSAH grant admin | kay@homelessauthority.org |
| Brianna Magoon | CSAH | brianna@homelessauthority.org |
| Aaron | CSAH marketing team | TBD |
---

## SMS — Decided: Twilio

**Decision: Finalized (2026-08-04).** Twilio.

### Use Cases
- Partner ↔ partner messaging tied to case records (not personal phones)
- Partner → homeless individual outbound (resource info + links)
- Kiosk request nudge to CSAH outreach team (see ADR-012, `specs/tech-decision-log.md`)
- Potential inbound reply threading back to case record

### Options Evaluated

| Provider | Est. Cost | Notes |
|---|---|---|
| **Twilio** (chosen) | ~$20–50/mo at this volume | Industry standard. Best API. Handles 2-way threading. Works with hosting. |
| **Telnyx** | Comparable to Twilio | Same underlying US A2P 10DLC registration system as Twilio (both submit to TCR — The Campaign Registry). No meaningful deliverability difference at this scale. |
| **Textbelt** | ~$25/500 texts (credit) | Dead simple, great for low-frequency outbound only. No real 2-way support. |
| **Vonage (Infobip)** | Comparable to Twilio | Solid alternative, slightly more nonprofit-friendly. No meaningful cost difference at this scale. |

**Estimated volume:** ~250 notifications/month (small). At this scale, cost is negligible.

### Why Twilio
- **Twilio has the only documented nonprofit path for A2P 10DLC campaign registration.** Government and nonprofit agencies are eligible for A2P 10DLC Special Use Cases — increased throughput and/or discounted pricing. The dominant SMS failure mode at this scale isn't delivery percentage, it's a rejected or suspended campaign registration; reducing that risk beats optimizing marginal deliverability.
- For US A2P messaging generally, deliverability is determined by 10DLC registration and trust score, not the provider — Twilio and Telnyx both submit to the same TCR (The Campaign Registry) system. This is why Telnyx isn't meaningfully worse, just not better enough to justify not using the provider with the clearer nonprofit registration path.
- Twilio's deeper carrier relationships and longer history with US carriers can translate to marginally better delivery on edge cases.
- 2-way threading needed for partner↔individual replies.
- Cost at this volume is negligible for a nonprofit.
- Works with lightweight hosting like Netlify.
- All messages logged against case records, not personal phones.

---

## Decisions Log

| Date | Decision | Rationale |
|---|---|---|
| 2026-05-08 | No HMIS integration for now | 2-year approval process; build custom DB instead; revisit later |
| 2026-05-08 | Hosting philosophy: keep it lean | Nonprofit budget; low recurring costs; Netlify candidate |
| 2026-05-08 | SMS provider TBD | Leaning Twilio; ~250/mo estimated volume |
| 2026-05-08 | Budget: $150k SCAD grant | Must spend by Dec 2026; kiosk hardware $10–25k/unit |
| 2026-05-08 | Push notifications are core | Not just SMS — real-time push coordination is the primary mechanism; initially CSAH-internal, partner agencies in future phase |
| 2026-05-08 | No kiosks purchased yet | Allyson + Alan own procurement and implementation by Dec 2026 |
| 2026-05-08 | No budget split scrutiny | SCAD treats kiosk + app as a connected ecosystem; no line-item breakdown required |
| 2026-05-08 | Geolocation = one-time pin drop only | Not continuous tracking; officer logs location at moment of contact; minimal consent/union exposure |
| 2026-05-08 | App platform leaning PWA + SMS | SMS fallback neutralizes main PWA weakness (iOS push reliability); avoids gov't MDM distribution problem; Expo is upgrade path only if device policies allow |
| 2026-05-08 | Decision docs shared via GitHub Pages | Alan is non-technical/visual; styled HTML in docs/ folder served via GitHub Pages |
| 2026-06-24 | Scope narrowed to CSAH-only for MVP test run | Partner agencies (HOPE, CCPD, etc.) are a future expansion; CSAH must prove the tool works internally before expanding to other agencies |
| 2026-06-24 | Survey tool added to app scope | Point-in-Time count survey embedded in CSAH staff app; administered during the annual 10-day count window; Kishia driving question scope |
| 2026-06-24 | Greyhound station added as potential kiosk location | Jen pursuing city manager approval; high PR value as proof-of-concept launch site; Union Mission is Plan B |
| 2026-08-03 | Netlify selected for hosting | Free tier covers CSAH's expected traffic with no commercial-use restriction; $19/mo Pro upgrade path if support/bandwidth needs grow; see docs/hosting-comparison.html for full vendor comparison |
| 2026-08-04 | Twilio selected for SMS | Only provider with a documented A2P 10DLC nonprofit registration path; campaign rejection/suspension is the dominant failure mode at this scale, not raw deliverability — see ADR-002, specs/tech-decision-log.md |

---

## Open Questions

See [`open.md`](open.md) for the full list of open questions, leans, and to-dos, organized by: Tech Architecture / Product & UX / Working with AI / Grant & Admin.

---

*Last updated: 2026-06-29 — Analytics & Reporting architecture added; open questions moved to open.md*
