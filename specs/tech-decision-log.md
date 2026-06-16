# Architecture Decision Records (ADRs)

## Format
Each ADR captures: the decision, the context that drove it, the options considered, and consequences/risks. Status: **Accepted** | **Superseded** | **Deprecated**.

---

## ADR-001 — App Platform: PWA over Native

**Date:** 2026-06-10
**Confirmed:** 2026-06-16
**Status:** Accepted — Final

**Context**
Partner users include SPD HOPE Unit officers and CCPD Behavioral Health staff on government-issued devices. Government device MDM policies may prohibit installation of third-party apps. Reaching these users via the App Store or Play Store may not be possible.

**Decision**
Build as a Progressive Web App (PWA). This decision is confirmed and not contingent on government device policy outcomes. Expo/React Native remains a possible future upgrade path only — it does not change the build target for this engagement.

**Options Considered**
1. PWA (chosen)
2. Expo / React Native
3. Native iOS + Android

**Consequences**
- No distribution barrier — runs in the browser, no MDM approval needed
- iOS push notification reliability is weaker than native; mitigated by SMS fallback (ADR-002)
- Service worker enables offline caching and write queuing for low-connectivity field use

**Risks**
- If iOS push proves too unreliable in practice, SMS fallback adds cost and latency
- If device policies do allow native apps, Expo remains an upgrade path — but the PWA investment is not wasted (same codebase)

---

## ADR-002 — SMS Fallback via Twilio

**Date:** 2026-05-08
**Status:** Accepted (provider pending final confirmation)

**Context**
PWA push notifications are unreliable on iOS. Field workers on iPhones may miss critical coordination alerts. The system must reliably deliver notifications to all users regardless of device.

**Decision**
Use Twilio as an SMS fallback for push notifications. Twilio is also the frontrunner for the partner → individual outbound SMS feature (F5).

**Options Considered**
1. Twilio (chosen/leaning) — best API, 2-way threading
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

## ADR-003 — Geolocation: One-Time Pin Drop Only

**Date:** 2026-05-08
**Status:** Accepted

**Context**
Push notifications for crisis coordination should include location context so outreach workers can find the individual. However, continuous GPS tracking of government officers raises concerns and consent complications.

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

## ADR-004 — Single Data Source for Kiosk and Partner App

**Date:** 2026-05-25
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

## ADR-005 — Backend: Supabase (BaaS)

**Date:** 2026-06-09
**Status:** Accepted

**Context**
The system requires a database, authentication, row-level data access control per partner agency, real-time updates for coordination features, and file storage — all with low recurring cost and a path to HMIS integration later. CSAH has no dedicated infrastructure team; ongoing maintenance falls to Allyson and Alan (ARC Consulting) or CSAH staff with non-technical backgrounds.

**Decision**
Use Supabase as the primary backend. Postgres for data, Supabase Auth for authentication, Row-Level Security for agency-scoped access control, Realtime for push/coordination features, and Edge Functions for lightweight server-side logic.

**Options Considered**
1. **Supabase** (chosen) — Postgres + auth + realtime + storage + edge functions in one platform; free/cheap tier is real at this scale (~$0–25/mo); built-in admin UI (Supabase Studio); data layer is portable (plain Postgres)
2. **Custom backend** (Node/Express or Python/FastAPI on Replit or Vercel) — maximum control, zero lock-in, but requires building auth, permissions, and realtime from scratch; more upfront work; better ceiling if HMIS integration eventually demands complex custom logic
3. **Low-code** (Bubble, FlutterFlow, etc.) — fastest to MVP, but ceiling is real and expansion path is murky for a 3–5 year horizon

**Consequences**
- Row-Level Security maps cleanly to "Agency X only sees Agency X's cases" — the core multi-agency access control requirement is handled at the DB layer
- Supabase Studio gives a developer-friendly admin UI; not Kay/Aaron-friendly as-is, so custom admin views will be needed for content management
- Realtime subscriptions enable live coordination updates without building WebSocket infrastructure
- Postgres is portable — if Supabase ever becomes untenable, the data layer exports cleanly
- Lock-in is real on auth and edge functions, but mitigated by Postgres portability

**Risks**
- Vendor lock-in on auth and edge functions; if Supabase pricing changes or the product direction shifts, migration is non-trivial for those layers
- Supabase Studio is not a substitute for a content management UI for CSAH staff; requires building a custom admin surface for non-technical users (Kay, Aaron)
- Edge Functions (Deno-based) have a different runtime than Node.js; minor friction for Node-heavy patterns

**Open Questions**
- Does the HMIS integration path (Caseworthy API, 2-year approval) introduce any constraints that would favor a fully custom backend over Supabase? Revisit when that process starts.

---

## ADR-006a — Data Validation: Pydantic (Python service layer)

**Date:** 2026-06-16
**Status:** Accepted

**Context**
Any Python service layer in the project — most likely an AI/LLM integration component for the "flashy innovation" SCAD grant requirement — needs a consistent approach to data validation, serialization, and schema enforcement at API boundaries. Without it, malformed payloads can propagate silently into case records, which the app uses as a truth-telling tool for external stakeholders.

**Decision**
Use Pydantic for data validation and schema definition in all Python service components.

**Options Considered**
1. **Pydantic** (chosen) — de facto standard in Python; native integration with FastAPI; LangChain and AI/ML frameworks use Pydantic models natively for structured outputs and tool definitions; type hints provide IDE and linter support
2. **marshmallow** — older, more verbose, no native FastAPI integration; no advantage over Pydantic for this stack
3. **No validation library** — ruled out; manual validation at Python API boundaries is error-prone

**Consequences**
- Type-safe data contracts at Python service boundaries — malformed payloads caught before they reach Supabase
- FastAPI integration is automatic; no additional validation code needed for endpoints
- LangChain structured outputs and tool call schemas use Pydantic models natively

**Risks**
- Pydantic v1 vs. v2 API differences are significant; ensure all Python dependencies (LangChain, etc.) pin to the same major version
- Pydantic is Python-only; the TypeScript/React layers need a separate approach (Zod is the equivalent)

**Open Questions**
- If the AI feature layer remains lightweight (no dedicated Python service), Pydantic may be unused until that work begins. Confirm whether a Python microservice is in scope for MVP or a later phase.

---

## ADR-006 — CI/CD Pipeline & Staging Environment Strategy

**Date:** 2026-06-11
**Status:** Accepted

**Context**
The partner app is used by frontline field workers in real-time crisis situations — downtime or broken features have direct real-world consequences. Pushing code directly to production introduces unacceptable risk. At the same time, CSAH's nonprofit budget rules out enterprise-grade cloud infrastructure. The project already uses Netlify's free tier to auto-deploy the `docs/` folder from a private GitHub repo, establishing a baseline for Git-driven deployment.

**Decision**
Implement a Git-based CI/CD pipeline (Netlify or Vercel) with a strict `main`/`staging` branching strategy and two isolated Supabase projects (staging DB and production DB).

- **Staging:** Pushes to the `staging` branch trigger an automatic build pointed at the staging Supabase project via environment variables, generating a Preview URL backed by test data only.
- **Production:** Merges into `main` trigger a production build pointed at the production Supabase project, updating the live partner app and kiosks.
- **Schema sync:** DB migrations are managed via CLI migration files to keep staging and production schemas structurally identical while keeping data strictly isolated.

**Options Considered**
1. **Git-based pipeline with isolated staging DB** (chosen)
2. **Direct-to-production deployments** — ruled out; no safety net for a tool handling sensitive field interactions in live crisis situations
3. **Large managed cloud (AWS/GCP)** — ruled out; violates the core project constraint to keep infrastructure lean and costs low

**Consequences**
- Catches bugs, broken UI, and integration regressions before they reach the ~50 partner users
- Provides a live Preview URL for Alan and Jen to approve features before production release
- Protects production data integrity — the app is a "truth telling tool" and test data polluting real case records would undermine its credibility with city/county stakeholders
- Staging environment runs indefinitely within Supabase and Netlify/Vercel free tiers; production costs stay well under $100/month

**Risks**
- Requires a strict branching discipline and PR workflow rather than direct pushes — adds process overhead
- Two databases require ongoing schema synchronization via migration files; drift between staging and production schemas would introduce hard-to-diagnose bugs

**Open Questions**
- Netlify vs. Vercel: current Netlify usage for `docs/` makes Netlify the path-of-least-resistance, but Vercel's preview deployment UX may be preferable for Alan/Jen review flows. Confirm before first app deploy.

---


