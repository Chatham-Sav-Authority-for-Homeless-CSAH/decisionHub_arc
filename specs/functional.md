# Feature Specifications

## Overview
Partner-facing coordination app for CSAH staff, SPD HOPE Unit officers, CCPD Behavioral Health units, city/county code enforcement, and partner agency case workers (~50 users total). Homeless individuals do **not** have app access.

---

## F1 — Push Notification System (Crisis Coordination)

**Priority:** P0 — Core

### Description
Real-time push alerts for cross-agency coordination when a homeless individual is encountered in the field.

### Actors
- Sender: SPD HOPE Unit officers, CCPD Behavioral Health, code enforcement, community paramedicine
- Receiver: CSAH street outreach team, partner case workers

### Behavior
- Officer creates a notification tied to an individual or location
- Notification delivers to relevant partner(s) in real time
- Includes geolocation pin (one-time drop at moment of contact — not continuous tracking)
- Notification links to the associated case record
- SMS fallback if push not delivered (Twilio)

### Notes
- Geolocation is a single pin drop, not continuous tracking — minimal consent/union exposure
- "Truth telling tool": geolocation data proves contact was made before enforcement action
- Acts as a data counter to negative narratives about the homeless population

---

## F2 — Case Tracking

**Priority:** P0 — Core

### Description
Each homeless individual is a "case." Field interactions are logged with who made contact, what was offered, and the outcome.

### Actors
- Any credentialed partner user (create, update, view cases within their agency scope)

### Behavior
- Create a case tied to an individual (name or pseudonym — no HMIS link for MVP)
- Log interactions: date, officer/worker, contact type, services offered, outcome
- Active communication thread stays open until manually closed
- Close a case via dropdown — outcome options: housed, declined services, referred, lost contact, etc.
- Case history is visible to authorized partners across agencies

### Data Captured Per Interaction
- Date/time
- Location (optional pin drop)
- Worker/officer ID
- Services offered (multi-select from resource list)
- Outcome
- Free-text notes

### Notes
- No HMIS integration for MVP (2-year approval process); custom DB
- Identity schema: request HMIS data dictionary from Keisha for reference, but build independently
- Dataset goal: demonstrate intervention volume, outcomes, resource utilization over time

---

## F3 — Resource Directory

**Priority:** P1

### Description
Shared resource guide surfaced on two distinct surfaces: embedded in the partner app for quick worker reference, and as the primary experience on the kiosk for unhoused individuals. Same Supabase data, different UI treatment per surface.

### Actors
- Partner app: any credentialed partner user (read-only)
- Kiosk: unhoused individuals (public, no login)

### Content
- Day centers, meals, showers/laundry
- Emergency shelters (adults, families, youth, women, men, DV survivors)
- Transportation (CAT bus, CSAH emergency transport)
- CSAH services and contact info
- Healthcare and behavioral health

### Behavior — Partner App
- Browse and search resources by category
- View hours, location, eligibility notes
- Tap to call or navigate (maps)
- Resources shared to an individual via SMS (see F5)

### Behavior — Kiosk
- Touch-optimized, large-target UI designed for users with limited tech experience
- Browse by category; no login or account required
- Tap to call where phone access is available
- Session resets on idle (timeout duration TBD)

### Notes
- Resource data is maintained by partner orgs directly (see F9)
- Kiosk-specific content scope (whether all partner-app resources appear publicly) is an open content decision

---

## F4 — Partner-to-Partner Messaging (Case-Linked)

**Priority:** P1

### Description
Shared messaging tied to case records, not personal phones. Multi-agency communication in context of a specific individual.

### Actors
- Any credentialed partner user with access to the relevant case

### Behavior
- Message thread lives inside the case record
- All partners with case access can read and post
- Messages are persistent (not ephemeral)
- No anonymous messaging

### Notes
- Replaces informal coordination over personal phones
- Audit trail supports data reporting and grant documentation

---

## F5 — Partner → Individual Outbound SMS (Future)

**Priority:** P2 — Post-MVP

### Description
Worker sends a text with resource links/info to an individual's phone number. Inbound replies thread back to the case record.

### Behavior
- Worker selects resources from directory, composes message, enters individual's phone number
- Message sent via Twilio
- If individual replies, message is captured and attached to the case thread
- Individuals do not have app logins or accounts

### Notes
- ~90% of the homeless population has smartphones
- Volume estimate: ~250 notifications/month (low)
- 2-way threading required — Twilio selected for this reason (ADR-002)

---

## F6 — User Authentication & Access Control

**Priority:** P0 — Required for launch

### Behavior
- Credentialed logins for ~50 partner users
- Role or agency-based access scoping (TBD — e.g., HOPE officers vs. CSAH staff vs. case workers)
- No self-registration — admin-provisioned accounts
- No login surface for homeless individuals

---

## F7 — Management Dashboard

**Priority:** P1

### Description
Data reporting and storytelling tool for CSAH leadership. Surfaces intervention volume, outcomes, and response patterns in a format designed to secure budget, attract donations, and counter negative narratives about the unhoused population with data.

### Actors
- Primary: Jennifer Dulong (Executive Director) and CSAH leadership
- Secondary: any credentialed partner user with reporting access

### Behavior
- Counters for key metrics: contacts made, services offered, outcomes (housed, referred, declined, lost contact)
- Activity trends over time (daily, weekly, monthly)
- Agency-level breakdown (which teams are active, response rates)
- Visual, shareable format suitable for city/county presentations and grant documentation

### Notes
- Data is generated by F1 (push notifications) and F2 (case tracking) — no separate data entry needed; clean field data is a prerequisite
- "Truth telling tool": primary purpose is demonstrating compassionate diversion from arrest and countering narratives that enforcement happens without prior outreach attempts
- Exportable format (PDF or similar) needed for grant reporting use cases

---

## F8 — Emergency / Weather Alert System

**Priority:** P1

### Description
CSAH staff publish emergency alerts (inclement weather, safety notices, resource activations) that surface immediately on both the kiosk and partner app. One alert record in Supabase; both surfaces subscribe via Supabase Realtime.

### Actors
- Author: CSAH staff (admin-level users)
- Recipients: kiosk users (unhoused individuals), partner app users (field workers)

### Behavior
- CSAH admin creates an alert: type, message body, active status
- Alert activates immediately on publish; both surfaces react in real time via Supabase Realtime subscription
- CSAH admin deactivates the alert when conditions resolve
- Kiosk presentation: TBD — a prominent, hard-to-miss experience directing users to emergency resources
- Partner app presentation: TBD — workers need to know an alert is active to coordinate the response; exact UI treatment (banner, push, overlay) not yet decided

### Notes
- Alert presentation per surface is an open design question — see `docs/app-architecture.html`
- Alert data schema (type enum, message, active boolean, timestamps) needs to be defined before either surface can be built

---

## F9 — Partner Org Content Management

**Priority:** P1

### Description
Partner organizations maintain their own resource directory listings — hours, availability, services offered, contact info — directly in the system. Changes publish immediately to both the partner app resource directory and the kiosk.

### Actors
- Partner org staff: at least one authorized user per org (edit their org's records only)
- CSAH admin: full access to all org records; can create/deactivate partner accounts

### Behavior
- Authorized partner org users log in and update their org's listing (hours, services, availability, contact info)
- Changes are live immediately — no approval workflow
- CSAH admin manages partner org accounts and can edit any record directly
- CSAH staff maintain listings for orgs that don't self-manage

### Notes
- Data accuracy is critical — both field workers and kiosk users depend on current information
- Row-Level Security in Supabase enforces that partner org users can only edit their own org's records (see ADR-005)

---

## Application Routing Structure

Single codebase, two URL namespaces — see `docs/app-architecture.html` for full discussion.

| Route | Surface | Auth | Notes |
|---|---|---|---|
| `/app/*` | Partner App (PWA) | Required — admin-provisioned logins | Full feature set: cases, messaging, push, dashboard, resource directory |
| `/kiosk/*` | Kiosk (SPA/PWA TBD) | None | Resource directory only; lockdown software restricts device to this URL |

Route guards at the kiosk shell prevent navigation to `/app`. Auth middleware on `/app` routes provides a second layer — the partner app requires login, which the kiosk never provides. Deployment platform (Netlify/Vercel) must have SPA redirect rules configured so deep links to both namespaces resolve correctly (`/* → index.html`).

---

## Out of Scope (MVP)
- HMIS integration
- Kiosk surface features beyond resource directory (under separate discussion)
- Inbound SMS from individuals (F5 is outbound only for MVP)
