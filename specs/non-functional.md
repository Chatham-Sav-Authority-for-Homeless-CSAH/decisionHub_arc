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

- Field workers operate in areas with unreliable connectivity
- App should handle degraded connectivity gracefully
- PWA service worker: cache critical resources and UI for offline view
- Write operations (interaction logs, messages): queue locally, sync when connection restored
- Push notifications: SMS fallback handles delivery if device is offline

---

## NFR-5 — Accessibility

- Partner users may have varying levels of tech literacy
- UI must be clear, low-friction, usable in field conditions (glare, one hand, urgency)
- WCAG 2.1 AA compliance target
- No accessibility requirements specific to partner app (kiosk has JAWS/screen reader requirements — separate)

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
- No custom infra management — managed services (Supabase, or similar) over self-hosted
- Clear separation between app logic and data layer so kiosk surface can share the same API

---

## NFR-8 — Time to Launch

- Grant funds must be spent by December 2026
- MVP needs to be functional well before that (buffer for procurement, hardware setup, training)
- Prioritize speed of initial delivery; avoid gold-plating for MVP

---

## NFR-9 — Device & Platform Constraints

- Partner users: mixed device environment — issued government devices (SPD HOPE, CCPD) with unknown MDM policies; personal devices; tablets
- Key open question: can government officers install third-party apps on issued devices? This gates the PWA vs. native app decision
- PWA is the safer bet until device policy is confirmed — no distribution barrier
- iOS push reliability is a known PWA weakness; SMS fallback is the mitigation

---

## NFR-10 — Audit / Reporting

- All interactions, notifications, and case updates must be timestamped and attributed to a specific user
- Data must support grant reporting (volume of contacts, outcomes, resource referrals)
- "Truth telling tool" requirement: geolocation + timestamp proves contact before enforcement action
- No PII export from the system without deliberate admin action
