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
- SMS fallback if push not delivered (Twilio, pending final decision)

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

## F3 — Embedded Resource Directory

**Priority:** P1

### Description
In-app resource guide for officers and workers who are not specialists in homeless services. Replaces QR codes and paper handouts.

### Actors
- Any credentialed partner user (read-only)

### Content
- Day centers, meals, showers/laundry
- Emergency shelters (adults, families, youth, women, men, DV survivors)
- Transportation (CAT bus, CSAH emergency transport)
- CSAH services and contact info
- Healthcare and behavioral health

### Behavior
- Browse and search resources by category
- View hours, location, eligibility notes
- Tap to call or navigate (maps)
- Resources shared to an individual via SMS (see F5)

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
- Message sent via Twilio (pending)
- If individual replies, message is captured and attached to the case thread
- Individuals do not have app logins or accounts

### Notes
- ~90% of the homeless population has smartphones
- Volume estimate: ~250 notifications/month (low)
- 2-way threading required — Twilio frontrunner for this reason

---

## F6 — User Authentication & Access Control

**Priority:** P0 — Required for launch

### Behavior
- Credentialed logins for ~50 partner users
- Role or agency-based access scoping (TBD — e.g., HOPE officers vs. CSAH staff vs. case workers)
- No self-registration — admin-provisioned accounts
- No login surface for homeless individuals

---

## Out of Scope (MVP)
- HMIS integration
- Kiosk surface (separate discussion)
- Inbound SMS from individuals (F5 is outbound only for MVP)
- Automated reporting/dashboards (logged data supports this later)
