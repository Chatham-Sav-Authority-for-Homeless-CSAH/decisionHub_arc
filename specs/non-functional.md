# Architecture Constraints

## Overview
Non-functional requirements and quality attributes that constrain design decisions across the CSAH partner app. These are not features — they are properties the system must have.

---

## NFR-1 — Cost Efficiency

**Priority:** Hard constraint

- Recurring infrastructure cost must be minimal — nonprofit budget
- Hosting platform TBD (Netlify, Vercel, Loveable, and others under consideration); no large managed cloud (AWS, GCP) deployments to start
- Kiosk software licensing target: ~$2k/device/year max (KioWare baseline); MDM alternatives significantly cheaper
- Total kiosk software budget: ~$6k/year for 3 devices (KioWare ceiling)
- App hosting should be well under $100/month at expected scale

---

## NFR-2 — Scale (User Volume)

- ~50 concurrent partner users max at MVP
- ~600 active cases in the system at any time (fluid population)
- ~250 SMS notifications/month
- No high-throughput requirements — this is not a consumer app

---

## NFR-3 — Availability & Reliability

- Field workers use this in real-time crisis situations — downtime during outreach hours has real consequences
- Target uptime: 99%+ during outreach hours (Mon–Sat 8:30am–9pm)
- Graceful degradation acceptable outside outreach hours
- SMS fallback (Twilio) covers push notification delivery gaps

---

## NFR-4 — Offline / Low-Connectivity Behavior

**Partner app:**
- Field workers operate in areas with unreliable connectivity
- PWA service worker: cache critical resources and UI for offline view
- Write operations (interaction logs, messages): queue locally, sync when connection restored
- Push notifications: SMS fallback handles delivery if device is offline

**Kiosk:**
- Offline caching via service worker under evaluation — resource directory should remain viewable if connectivity drops rather than displaying an error to a user in crisis
- Dependent on SPA vs. lightweight PWA decision (TBD)

---

## NFR-5 — Accessibility

- WCAG 2.1 AA compliance target for both surfaces
- No Section 508 requirement — CSAH is not federally funded; ADA physical compliance is a hardware procurement requirement, not an app requirement

**Partner app:**
- UI must be clear, low-friction, usable in field conditions (glare, one hand, urgency)
- Partner users may have varying levels of tech literacy

**Kiosk:**
- Large touch targets — users may have limited fine motor control or be unfamiliar with touchscreens
- Simplified navigation, minimal choices per screen, high-contrast text
- Designed for users with varying literacy levels, cognitive load, and physical challenges
- WCAG 2.1 AA delivered by the app layer — not by kiosk lockdown software

---

## NFR-6 — Security & Privacy

- Case records contain sensitive information about vulnerable individuals — treat as PII-adjacent even if not formally classified
- Auth required for all case/messaging functionality
- No anonymous access to case data
- Geolocation data is a single pin drop per interaction — not continuous; stored in case record, not shared externally
- No PII collected from homeless individuals on kiosk surface
- Data must not leave the US (no offshore processing)
- HMIS data is NOT in this system (MVP) — but design should not foreclose future integration

---

## NFR-7 — Maintainability

- Small team (Allyson + Alan) owns the system post-handoff; or CSAH self-maintains
- Prefer simple, well-understood stack over clever/novel choices
- Minimize vendor lock-in where practical
- No custom infra management — Supabase for backend (ADR-005, decided); managed hosting (Netlify or Vercel, TBD) over self-hosted
- Single codebase, two entry points (`/app/*` partner app, `/kiosk/*` kiosk) — shared Supabase hooks and resource directory components; see `docs/app-architecture.html`

---

## NFR-8 — Time to Launch

- Grant funds must be spent by December 2026
- MVP needs to be functional well before that (buffer for procurement, hardware setup, training)
- Prioritize speed of initial delivery; avoid gold-plating for MVP

---

## NFR-9 — Device & Platform Constraints

**Partner app:**
- Platform: PWA — confirmed (ADR-001, June 2026); not contingent on government device policy outcomes
- Device environment: mixed — issued government devices (SPD HOPE, CCPD), personal devices, tablets; PWA runs in any browser with no install required
- iOS push reliability is a known PWA weakness; SMS fallback (Twilio) is the mitigation
- Expo/React Native remains a possible future upgrade path but does not change the build target for this engagement

**Kiosk:**
- Dedicated outdoor hardware running kiosk lockdown software (Scalefusion, KioWare, or equivalent)
- Lockdown software restricts the device to the `/kiosk` URL; handles remote device management
- OS and browser environment to be confirmed against final hardware selection before build

---

## NFR-10 — Audit / Reporting

- All interactions, notifications, and case updates must be timestamped and attributed to a specific user
- Data must support grant reporting (volume of contacts, outcomes, resource referrals)
- "Truth telling tool" requirement: geolocation + timestamp proves contact before enforcement action
- No PII export from the system without deliberate admin action
