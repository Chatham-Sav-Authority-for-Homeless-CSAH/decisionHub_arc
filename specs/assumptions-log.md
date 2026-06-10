# Architecture Decision Records (ADRs)

## Format
Each ADR captures: the decision, the context that drove it, the options considered, and consequences/risks. Status: **Accepted** | **Superseded** | **Deprecated**.

---

## ADR-001 — No HMIS Integration at MVP

**Date:** 2026-05-08
**Status:** Accepted

**Context**
All 76 partner agencies in Chatham County feed case data into the Caseworthy HMIS platform. Integration would give the app access to existing case histories. However, HMIS data access requires a formal HUD approval process.

**Decision**
Build a custom database for MVP. Do not attempt HMIS integration now.

**Options Considered**
1. Custom DB (chosen)
2. HMIS API integration

**Consequences**
- No access to existing case histories at launch — workers start fresh records
- Custom schema must be designed deliberately so future HMIS sync is feasible (request Keisha's HMIS data dictionary as reference)
- Revisit after MVP ships; 2-year approval clock can start running in parallel

**Risks**
- Data duplication: same individuals may be tracked in both systems for a time
- Schema divergence if we don't reference HMIS data dictionary

---

## ADR-002 — App Platform: PWA over Native

**Date:** 2026-05-08
**Status:** Accepted

**Context**
Partner users include SPD HOPE Unit officers and CCPD Behavioral Health staff on government-issued devices. Government device MDM policies may prohibit installation of third-party apps. Reaching these users via the App Store or Play Store may not be possible.

**Decision**
Build as a Progressive Web App (PWA). Expo/React Native is an upgrade path only if device policies are confirmed to allow it.

**Options Considered**
1. PWA (chosen)
2. Expo / React Native
3. Native iOS + Android

**Consequences**
- No distribution barrier — runs in the browser, no MDM approval needed
- iOS push notification reliability is weaker than native; mitigated by SMS fallback (ADR-003)
- Service worker enables offline caching and write queuing for low-connectivity field use

**Risks**
- If iOS push proves too unreliable in practice, SMS fallback adds cost and latency
- If device policies do allow native apps, we may revisit Expo — but the PWA investment is not wasted (same codebase)

**Open Question**
Confirm with SPD HOPE and CCPD whether officers can install third-party apps on issued devices. This is the primary trigger for revisiting this decision.

---

## ADR-003 — SMS Fallback via Twilio

**Date:** 2026-05-08
**Status:** Accepted (provider pending final confirmation)

**Context**
PWA push notifications are unreliable on iOS. Field workers on iPhones may miss critical coordination alerts. The system must reliably deliver notifications to all users regardless of device.

**Decision**
Use Twilio as an SMS fallback for push notifications. Twilio is also the frontrunner for the partner → individual outbound SMS feature (F5).

**Options Considered**
1. Twilio (chosen/leaning) — best API, 2-way threading, works with Replit
2. Textbelt — dead simple, outbound only, no 2-way support
3. Vonage/Infobip — comparable to Twilio, no meaningful cost difference at this volume

**Consequences**
- Estimated ~250 notifications/month = ~$20–50/mo; negligible for a nonprofit
- 2-way threading (partner → individual → case record) supported by Twilio
- SMS adds a communication layer outside the app — must ensure replies are captured in case records

**Risks**
- Cost scales with volume; current estimate is low but should be monitored
- Open question: is push + SMS duplication too invasive? User preference settings may be needed

---

## ADR-004 — Geolocation: One-Time Pin Drop Only

**Date:** 2026-05-08
**Status:** Accepted

**Context**
Push notifications for crisis coordination should include location context so outreach workers can find the individual. However, continuous GPS tracking of government officers raises union concerns and consent complications.

**Decision**
Geolocation is a single pin drop at the moment of contact — not continuous tracking. Location is stored in the case interaction record.

**Options Considered**
1. One-time pin drop at moment of contact (chosen)
2. Continuous GPS tracking

**Consequences**
- Minimal consent/union exposure
- Location data is timestamped and tied to the case record — serves as proof of contact before any enforcement action
- "Truth telling tool": geolocation data counters narratives that enforcement happens without outreach attempts

**Risks**
- Single pin drop does not show where an individual moved after contact
- Workers must remember to drop the pin; no automatic capture

---

## ADR-005 — Single Data Source for Kiosk and Partner App

**Date:** 2026-05-08
**Status:** Accepted

**Context**
The kiosk (public-facing) and partner app share overlapping content — resource directory, service availability, alerts. Keeping these in sync across two separate data sources would create maintenance burden.

**Decision**
The kiosk points to the same backend API as the partner app. Kiosk lockdown software (KioWare or alternative) just restricts the device to a URL. Any update to the data source propagates to both surfaces automatically.

**Options Considered**
1. Shared backend API (chosen)
2. Kiosk platform with built-in CMS (e.g., SiteKiosk)

**Consequences**
- Single point of truth for all content
- No manual sync required between kiosk and app
- Kiosk software choice does not gate this — it's an architecture decision, not a vendor decision

**Risks**
- If kiosk-specific content (e.g., attract screen, idle messaging) diverges from partner app content, the shared layer needs to handle routing/access control cleanly

---

## ADR-006 — Decision Docs Served as HTML via Netlify

**Date:** 2026-05-08
**Status:** Accepted

**Context**
Alan Robinson (client-facing lead) is non-technical and visual. He needs to review architecture and product decisions but cannot read raw markdown or code files.

**Decision**
Decision documents are authored as styled HTML files in the `docs/` folder and served via Netlify (auto-deployed from the private GitHub repo). Alan opens them in a browser from a URL.

**Options Considered**
1. Styled HTML via Netlify (chosen)
2. Notion or Google Docs
3. Raw markdown in GitHub

**Consequences**
- Netlify publish directory: `docs/`
- Base URL: `csahcompassapp.netlify.app`
- Every push to main auto-deploys
- `docs/index.html` is the navigation surface for Alan
- Visual style established in `docs/pwa-vs-mobile.html` — dark header, card layout, color-coded columns

**Risks**
- Netlify free tier has bandwidth limits (unlikely to be hit at this scale)
- HTML files require more authoring effort than markdown — offset by the visual consistency they provide

---

## ADR-007 — Lean Hosting Philosophy

**Date:** 2026-05-08
**Status:** Accepted

**Context**
CSAH is a nonprofit with constrained budget. The grant covers initial development but long-term sustainability requires low recurring costs. The system serves ~50 concurrent users — not high-throughput.

**Decision**
Replit is the candidate hosting platform. No large managed cloud deployments (AWS, GCP) to start. Prefer managed services over self-hosted infrastructure.

**Options Considered**
1. Replit (chosen/leaning)
2. AWS
3. GCP
4. Heroku / Railway / Render

**Consequences**
- Low monthly cost at expected scale
- Managed services (DB, auth) reduce operational burden for a small team
- No custom infra management

**Risks**
- Replit reliability and cold-start behavior needs validation before committing
- If scale grows significantly, migration path to a more robust host will be needed

---

## ADR-008 — No Kiosk Hardware Purchased Yet

**Date:** 2026-05-08
**Status:** Accepted

**Context**
The $150k SCAD grant must be fully spent by December 2026. Kiosk hardware is $10–25k per unit. Three units are planned. Procurement has not started.

**Decision**
Allyson and Alan own kiosk procurement and implementation. No purchase orders placed yet. Kiosk software decision (KioWare vs. Scalefusion vs. alternatives) must be finalized before procurement.

**Consequences**
- Buffer time needed for hardware delivery, setup, and testing before December 2026 deadline
- Software licensing decision gates hardware order (compatibility must be confirmed)

**Risks**
- Supply chain delays on kiosk hardware
- December 2026 spend deadline is hard — procurement should start by Q3 2026
