# Architecture Decision Records (ADRs)

## Format
Each ADR captures: the decision, the context that drove it, the options considered, and consequences/risks. Status: **Open** | **Accepted** | **Superseded** | **Deprecated**.

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

## ADR-002 — SMS Provider: Twilio

**Date:** 2026-05-08
**Confirmed:** 2026-08-04
**Status:** Accepted — Final

**Context**
PWA push notifications are unreliable on iOS. Field workers on iPhones may miss critical coordination alerts. The system must reliably deliver notifications to all users regardless of device. Twilio is also the frontrunner for the partner → individual outbound SMS feature (F5) and the kiosk request nudge (ADR-012).

**Decision**
Use Twilio as the SMS provider, for both the push-notification fallback and every other SMS use case in the project (F5, ADR-012).

**Options Considered**
1. **Twilio** (chosen) — best API, 2-way threading, only provider with a documented A2P 10DLC nonprofit registration path
2. **Telnyx** — comparable API and pricing; submits to the same TCR (The Campaign Registry) as Twilio for US A2P 10DLC registration, so no meaningful deliverability difference at this scale. No documented nonprofit registration path.
3. Textbelt — dead simple, outbound only, no 2-way support
4. Vonage/Infobip — comparable to Twilio, no meaningful cost difference at this volume

**Why Twilio over Telnyx specifically**
The dominant SMS failure mode for a project this size isn't a delivery-percentage difference between providers — it's a rejected or suspended A2P 10DLC campaign registration, which stops messages from sending at all. Government and nonprofit agencies are eligible for Twilio's A2P 10DLC Special Use Cases (increased throughput and/or discounted pricing); this is the only *documented* nonprofit registration path found across the providers evaluated. Since US A2P deliverability is governed by 10DLC registration and trust score in TCR — not by which provider submitted it — Twilio and Telnyx perform equivalently once registered. The deciding factor is registration risk, not runtime deliverability. Twilio's longer history and deeper relationships with US carriers is a secondary, marginal factor on top of that.

**Consequences**
- Estimated ~250 notifications/month = ~$20–50/mo; negligible for a nonprofit
- 2-way threading (partner → individual → case record) supported by Twilio
- SMS adds a communication layer outside the app — must ensure replies are captured in case records
- One provider across all SMS use cases (push fallback, F5 outbound, ADR-012 kiosk nudge) — one A2P 10DLC campaign registration to maintain, not several
- Every send from this provider gets delivery-status tracking — see ADR-013

**Risks**
- Cost scales with volume; current estimate is low but should be monitored
- A2P 10DLC campaign registration itself is a real setup step with CSAH's nonprofit documentation (501(c)(3) charter, etc.) — not instant; needs lead time before launch, not a same-day integration
- Open question: is push + SMS duplication too invasive? User preference settings may be needed (see `open.md`)

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
2. **Custom backend** (Node/Express or Python/FastAPI on Netlify) — maximum control, zero lock-in, but requires building auth, permissions, and realtime from scratch; more upfront work; better ceiling if HMIS integration eventually demands complex custom logic
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
- ~~Netlify vs. Vercel~~ — Resolved by ADR-011 (2026-08-03): Netlify selected.

---

## ADR-007 — Repository Structure: Monorepo (Single Codebase, Two Entry Points)

**Date:** 2026-07-29
**Status:** Accepted

**Context**
The kiosk app (public-facing, `/kiosk/*`) and the partner app (staff-facing, `/app/*`) are two different UI surfaces with different audiences, but they share a substantial amount of underlying code: resource directory components, Supabase data hooks (see ADR-004), and — the main driver for formalizing this now — language translation strings. Splitting these into separate repositories would mean maintaining two independent copies of every shared string, with real risk of the same phrase being translated inconsistently (or updated on one surface and forgotten on the other).

**Decision**
Build the kiosk and partner app as a single codebase (monorepo) with two route-level entry points (`/kiosk/*`, `/app/*`) as separate UI shells over shared components, Supabase hooks, and a shared translation/i18n layer. Route guards prevent kiosk users from reaching partner-only views and vice versa.

**Options Considered**
1. **Monorepo, two entry points** (chosen) — one codebase; shared components, hooks, and translations; route-based separation
2. **Two separate repos** — fully independent kiosk and partner app codebases, each maintaining its own copy of shared UI and translation strings
3. **Monorepo with separate internal packages** (e.g., npm workspaces / Turborepo) — shared code extracted into installable internal packages consumed by two app packages — more structure than this project's scale currently justifies

**Consequences**
- Shared resource directory components and Supabase data hooks are written once — extends ADR-004's single-data-source principle to the code layer, not just the data layer
- **Language translation strings are maintained in one place.** A string translated once is available on both the kiosk and partner surfaces — no risk of the same phrase being translated differently, or translated on one surface and missed on the other
- Single build/deploy pipeline covers both surfaces (see ADR-006)
- Route guards are the only mechanism keeping kiosk users out of partner-only views — there's no repo-level isolation forcing that separation, so this needs solid test coverage

**Risks**
- A bug in shared code can affect both surfaces simultaneously; mitigated by route-level isolation for UI, but shared components need solid test coverage
- No package-level boundaries yet between "shared" and "kiosk-only"/"partner-only" code — fine at this scale, but revisit workspace tooling (option 3) if the shared surface grows large enough that this becomes hard to reason about

---

## ADR-008 — Mapping / Maps API for Kiosk

**Date:** 2026-07-30
**Status:** Open — Not Decided

**Context**
Kiosk wayfinding requires an in-app maps/directions integration. Three vendor candidates are under evaluation.

**Candidates**
1. MapLibre + OpenStreetMap-derived routing
2. Google Maps Platform (JS API + Directions) — in play pending Text To Speech (TTS) answer
3. Mapbox GL JS + Mapbox Directions

**Decided**
- Desireable to cache in db directions from Kiosks to resources since both are static. Saves on API calls (latency or service outtage issues) and possible costs. A typical model would be Lazy caching: A user at the Union Mission kiosk asks for directions to Emmaus House → you call the API, serve it, and store the result. The next person who requests that same pair within 30 days gets the stored copy. At 30 days it expires and the next request re-fetches.
- Maps render in-app. No external redirect, no whitelisting google.com. Applies to all candidates.
- Offline turn-by-turn is not a requirement. Directory data and tiles cover the offline case; only the live route line is lost if cell connectivity has issue.
- Southwest Chatham Library kiosk gets no walking directions. ~8 miles from all downtown resources. Needs transit/pickup UX — CAT bus, 912-547-7877, Monday Southside→Union Mission pickup. Separate screen design.
- Union Mission and Bull St Library are the two walking-directions sites.
- Build the maps integration behind an abstraction (e.g., MapLibre) that can point at Google tiles today and self-hosted OSM tiles — or another provider — later, without a rebuild, rather than wiring the app directly to one vendor's SDK.

**Verified Constraints**
- Mapbox: no caching/storage of Directions results on any level (Product Terms §2.10.1). Tiles cacheable on device 30 days (§2.8.1). No automated/bulk queries (§1.9). Mandatory attribution + "Improve this map" link on-screen.
- Google: 30-day cache allowed for intermittent connectivity issues only — not to build a permanent cache. No prefetching.
- Neither vendor has a wheelchair routing profile. Accessibility routing is out of scope; replaced by accessibility metadata on resource records plus worker-pinned hazards.

**Blocking the Decision**
- Google TTS clause clarification & lazy cache clarification — determines whether Web Speech narration of directions is permitted. Also, whether a lazy read-through cache with 30-day expiry is acceptable. See ADR-010, "Where the map vendor touches this," for why this clause matters only for directions text and not CSAH's own content.
- Route quality test results — script written, not run. Pass/fail on sidewalk-vs-centerline routing determines helps to know if vendor option 1 is robust

**Not a Factor**
- Field geolocation capture — browser Geolocation API, vendor-independent.
- Geocoding storage — decoupled from map vendor; Census Geocoder or Mapbox Permanent, decide separately.
- Cost — all candidates effectively free at projected volume, expectially if lazy caching.

**Note**
Cache strategy (see Decided, above) is a preference, not a hard requirement. Mapbox's Product Terms §2.10.1 prohibits it outright; Google's 30-day allowance is scoped to connectivity resilience, not cost/latency savings, so the stated rationale above may not qualify under Google's terms as written. Only MapLibre + OSM is unambiguously compatible with the caching approach as described. Mapbox and Google hand you a finished-looking map. MapLibre gives you a functional but plain one — you start from a free pre-made style and adjust colors and fonts to match CSAH branding in a visual editor. MapLibre also has a Tile hosting dependency.

R.E Google Maps: Most teams ship and accept the risk. Enforcement is account suspension, not litigation, and it's rare against small compliant-looking users. The clause is aimed at scrapers.

Teams with counsel document a risk determination. The org's attorney reads it, writes a one-paragraph memo on the reasonable interpretation, and that memo is the artifact that protects everyone. That's the real deliverable, not a Google reply.

Everyone else routes around it — picks the vendor without the ambiguous clause and stops thinking about it.

For you: CSAH almost certainly has counsel or a pro bono relationship. That's a 30-minute ask and a documented answer. Cheaper than waiting on Google, and it's the record you'd actually want if it ever mattered.

---

## ADR-009 — Geolocation Capture: Browser API + Human-Confirmed Pin

**Date:** 2026-07-30
**Status:** Accepted

**Context**
ADR-003 established *what* gets captured — a single pin drop at the moment of contact, not continuous tracking. This ADR covers *how* that pin drop is technically captured with usable accuracy. Raw device GPS alone isn't reliable enough for this use case (a worker may be standing behind a building, not inside it, and the coordinate needs to reflect reality). The mechanism also needs to stay independent of whichever mapping vendor ADR-008 eventually lands on.

**Decision**
Use the browser's native `navigator.geolocation` API — no library, no vendor, no cost, unrelated to whichever map provider is chosen. `getCurrentPosition()` is called and returns coordinates asynchronously.

Production pattern:
- Request the location fix when the form opens, not when the worker hits save — GPS needs a few seconds to acquire, so it settles in the background while they're filling out the rest of the form.
- Show a small map with a pin at the returned coordinates.
- Let the worker drag the pin to correct it. A human standing there beats any algorithm — they know they're behind the Kroger, not inside it.
- Store what actually got submitted, plus the reported accuracy, plus a flag for whether it was auto-captured or hand-adjusted.

Two hard dependencies:
- **HTTPS is mandatory** — geolocation is blocked on insecure origins. Netlify's hosting already covers this.
- **Browser/device location permission must be enabled** on the worker's phone. This is on the user, not the app — include it in staff training/device rollout checklist.

**Options Considered**
1. Browser Geolocation API + human-confirmed pin (chosen) — free, vendor-independent, no library to maintain
2. Continuous GPS tracking — already ruled out under ADR-003 on consent/union-exposure grounds, not reconsidered here
3. Third-party geolocation/IP-lookup service — unnecessary; device-reported GPS via the browser API is more accurate than IP-based lookup and adds no cost, so there's no reason to introduce a vendor for this

**Consequences**
- No mapping/geo vendor dependency for the capture step itself — works regardless of which vendor ADR-008 selects
- No cost, no third-party library to maintain or update
- Requesting the fix on form-open rather than form-save avoids a save-time delay waiting on GPS acquisition
- The draggable-pin confirm step handles the real-world gap between raw device GPS and where the worker actually is (which side of a building, which storefront)
- The auto-vs-hand-adjusted flag creates an audit trail distinguishing algorithmic capture from human correction — useful for later data-quality review

**Risks**
- Depends on the worker's phone having location permission granted to the browser/PWA — not guaranteed by default; if training doesn't cover this, the fix silently fails or prompts mid-task
- A worker who submits before GPS has settled may get a low-accuracy fix; the drag-to-correct step mitigates but doesn't eliminate this — relies on the worker actually checking pin placement, not just trusting it
- Insecure origin (non-HTTPS) breaks geolocation entirely — not a risk in production given Netlify, but would silently fail in any local/preview setup that isn't served over HTTPS

**Open Questions**
- What's the fallback UX if the browser denies or the user rejects the location permission prompt entirely — does the map still render (centered on a default location) so the worker can manually place the pin with no GPS fix at all? Not addressed yet; worth confirming before this ships.

---

## ADR-010 — Localization & Narration: Database-Native MT (No TMS, No Overlay Vendor)

**Date:** 2026-07-30
**Status:** Accepted

**Acronyms**

| Term | Definition |
|---|---|
| **MT** | Machine Translation — an API that converts text from one language to another (e.g. Google Cloud Translation, Azure Translator, DeepL) |
| **TMS** | Translation Management System — a hosted workspace where human translators review and edit strings, with version history, approval workflow, and MT pre-fill built in (e.g. Crowdin, Lokalise, Phrase, Tolgee) |

**Context**
CSAH's kiosk and partner app need to serve Spanish-speaking users with translated text and spoken narration. **Language scope confirmed as Spanish only by Jen Dulong on 2026-07-29** — the schema below keeps additional languages an additive change (new rows, not a migration) should that change. Prior vendor research (see `open.md`) compared machine-translation providers on cost, free tier, and language coverage; this ADR decides the surrounding architecture — where translated content lives, how it gets there, and what tooling touches it — independent of which MT vendor is ultimately selected.

**Decision**
Build localization and narration into the existing Supabase schema and custom admin app. No CMS, no TMS, no accessibility overlay vendor.

- **Locales:** `en-US` and `es-US`. Not generic `es` — the region subtag drives both MT target selection and Web Speech API voice selection, and generic `es` risks a Castilian voice reading to a Latin American audience.
- **MT runs on every write.** Machine-generated Spanish is produced at save time and stored. The kiosk reads stored text; no translation API is ever called at render time.
- **Human review is a trailing pass**, not a gate. A native speaker reviews the full baseline corpus before launch. Ongoing edits publish immediately with MT output and are surfaced for later review.
- **Narration** via Web Speech API, all locales including English.

**Options Considered**

1. **Accessibility overlay vendor (accessiBe, UserWay, AudioEye) — rejected.** This is the only product category that bundles translation and narration into a single script, and it is the category to avoid. The Federal Trade Commission fined accessiBe $1 million in 2025 over claims its AI tool could make any website WCAG-compliant; the complaint alleged the plug-in failed on navigation menus, form fields, and image descriptions — precisely the components assistive technology users depend on. Over 800 businesses using overlays were sued in 2023–2024, and courts have consistently declined to accept overlay installation as evidence of ADA compliance.

2. **TMS (Crowdin, Lokalise, Phrase, Tolgee) — rejected.** A TMS is built for shipping locale files across many languages using external translation vendors. It assumes translations live in version-controlled files that sync into the build. Our strings live in the database precisely so CSAH can edit them without a deploy — the two models work against each other. Beyond the architectural mismatch: it is another vendor, another subscription (~$30–120/month at small scale), another login for an 11-person team, and content sitting in a third-party system. The features it provides that we actually need — a side-by-side editor and a review queue — amount to a small slice of our admin app.

3. **Headless CMS for the kiosk — rejected.** Considered and rejected because the resource directory is shared between the kiosk and the partner app. A CMS for the kiosk would mean either maintaining resources in two systems or building sync between them. The roadmap makes this decisive: PIT surveys, ~50 partner agency users, referral close-out, and audit logging are application concerns, not content concerns. A CMS would be a second system outgrown almost immediately, with the kiosk already coupled to it. (Consistent with ADR-004.)

4. **MT with human post-editing, stored in Supabase — chosen.** Machine translation pre-fill with human review is the standard industry pattern. Our variant stores the output rather than translating at render time, which yields bounded cost, no third-party dependency in the request path, and no vendor uptime exposure on the kiosk. Volume makes cost a non-factor: roughly 80,000 characters for the full directory, incremental edits thereafter. This sits permanently inside the free tier of every major provider (Azure 2M characters/month, Google 500K/month). Provider selection is deferred; with Spanish as the only target, DeepL is the likely pick on quality grounds, and Latin American Spanish support as a distinct target must be confirmed with whichever vendor is chosen.

**Content Editing Model: Typed Entries**

Four CMS content patterns were evaluated to model after:

| Pattern | Description | Verdict |
|---|---|---|
| **Typed entries** | Content is documents with defined fields; admin renders a form from that shape | **Chosen** — already how `resources` works |
| **Block composition** | Pages are arrays of rearrangeable typed blocks (Storyblok, Gutenberg) | Rejected — kiosk screen layouts are fixed by design; rearrangeable page structure is capability we would build and never use, and it would let staff break a carefully designed accessible layout |
| **Flat string catalog** | Every string has a key, edited in one flat list | Rejected as-is — an editor seeing `nav.findShelter` has no idea which of nine screens it appears on. Adopted only in modified form, with `screen` and `sort_order` grouping |
| **In-context visual editing** | Live app embedded in an iframe; click text to edit in place (Tina, Storyblok, Sanity Presentation) | Rejected — overkill at this scale. Requires preview-mode rendering, `data-i18n-key` tagging on every string, and postMessage plumbing between parent and iframe. Meaningful build cost to save an 11-person team a browser tab switch |

**Chosen: typed entries, with `ui_strings` grouped by `screen` and `sort_order`.**

Because UI strings are database-backed rather than compiled into the bundle, there is no build or deploy step. An editor saves a change, opens the live app in another tab, toggles to Spanish, and sees it. This is why no preview tooling is needed — the "preview" is the app itself.

**User Flow**

1. Editor logs into the CSAH app (authenticated, so identity and timestamp are captured)
2. Navigates to a resource, or to UI strings grouped by screen
3. Edits the English field. English and Spanish appear side by side, source left, target right — the layout every translation tool uses, because reviewers need the source permanently visible
4. On save, a single atomic transaction writes: the English text with the editor's user ID, and the MT-generated Spanish with the reserved system actor ID. Either both land or neither does
5. If the editor also edits the Spanish, their user ID is written to that row instead of the system ID
6. Content is live immediately — no build, no deploy, no approval gate
7. A report lists Spanish rows written by the system actor since the last review, for a native speaker to work through as time allows

**No approve button.** An explicit approve action was considered and rejected as added confusion for an 11-person team where the same person edits and vouches for the content. Machine-authored rows are identified by `last_edited_by = SYSTEM_ACTOR`; the digest is scoped by date rather than by state, so reviewed rows fall off the list naturally and nothing lingers.

**Database Schema**

Translatable text is separated from the entities that own it. Base tables hold only non-linguistic fields; all human-readable text lives in dedicated translation tables, one row per locale.

English is a locale row, not a privileged column on the base table. Narration applies to English as much as Spanish, so treating English as just another locale means one code path for rendering and one for narration, rather than a branch on "is this the source language."

Three tables hold text:

| Table | Holds | Keyed on |
|---|---|---|
| `resource_translations` | Resource name, description, notes, plus their optional spoken counterparts | resource + locale |
| `alert_translations` | Alert title and body, plus spoken counterparts | alert + locale |
| `ui_strings` | Buttons, navigation, headings, error messages — all app chrome | string key + locale |

Base tables (`resources`, `alerts`) hold everything that isn't language-dependent: category, phone, address, coordinates, and structured hours stored as time values and day-of-week arrays rather than pre-formatted strings. Keeping hours structured is what allows the narration formatter to generate a correct spoken form per locale.

Every translation row carries the display text, an optional spoken variant (null means fall back to display), the user ID of whoever last wrote it, and a timestamp. `ui_strings` adds three fields the entity tables don't need: `screen` and `sort_order` to group the admin editor list by screen in render order, and an optional `max_length` to warn editors when a Spanish string will overflow constrained kiosk UI.

String keys are code-controlled. The admin edits values only. A renamed key silently blanks a screen with no trace, so key management stays in the repo.

Why per-entity translation tables rather than one global `translations` table with an entity-type discriminator: a global table cannot carry a real foreign key, because its ID column points at different tables depending on the row. That costs cascade-on-delete, invites orphan rows, forces RLS policies that can't express simple ownership, types every field as generic text, and makes every query join on a string column. That design earns its keep in a CMS with forty content types. We have three or four.

Why not wide columns (`name_es`, `description_es` on the base table): adding a language becomes a migration touching every table and every query. Per-locale rows make it an insert.

Constraint required: a trigger or check guaranteeing an English row exists for every base row. This is the one guarantee given up by not putting English on the base table.

System actor: a reserved user ID representing machine-translation writes, so the editor field is never null and the review digest is a simple predicate — Spanish rows last written by the system actor since a given date.

**Staleness Detection**

`updated_at` per translation row. A regular sweep flags any entity where the `es-US` row's `updated_at` is older than the `en-US` row's, or where the `es-US` row is missing entirely.

Under atomic writes this job should find nothing. Its purpose is catching writes that bypass the application pipeline — bulk imports, direct SQL, migrations, backup restores. Nobody should be doing this on the CSAH team.

**Narration (Spoken Fields)**

Every translatable field has an optional spoken counterpart. It is null in the large majority of rows, falling back to the display value. Three cases:

- **Default** — spoken is null. Prose reads aloud correctly as written.
- **Structured fields** — generated, never authored. Hours and phone numbers are stored as structured data, not strings. One formatter per locale produces both forms from the same value: "8am–9pm" for display, "eight in the morning to nine at night" / "de ocho de la mañana a nueve de la noche" for speech. Phone numbers are read digit by digit, in the digits of the active locale. Nobody types these.
- **Override** — hand-authored, rare. For cases the formatter cannot reach: an organization name needing phonetic help, an acronym that should be spelled out. English overrides are typed by the editor; Spanish overrides run through the same MT path as any other field and appear in the same review digest.

This is the second payoff from the content-tier split below — structured data can generate a correct spoken form, while a pre-formatted display string cannot.

**Narration Implementation**

`speechSynthesis` — built into the browser, free, no vendor, no licensing. Hand it text and a language code; it speaks. The voices come from the kiosk's **operating system**, not the browser itself — this is why voice availability (see Risks, below) is a hardware/OS question, not a browser-support question. Kiosk hardware selection must confirm Spanish voices are installed at the OS level, not just that the Web Speech API is exposed.

What gets built:

1. **A narration service.** One module wrapping `speechSynthesis` — speak, stop, pause, queue, current language. Every screen calls into this module rather than touching the Web Speech API directly.
2. **Display/spoken field pairs.** Covered above — screen text and spoken text differ (abbreviations, phone numbers, addresses read digit-by-digit), and the pairing flows through the existing translate-on-write pipeline so Spanish spoken text is authored alongside English, not bolted on afterward.
3. **A persistent narration control.** Speaker icon + stop button on every screen, large touch target, state survives navigation — once a user turns narration on, it stays on until session reset.
4. **Language wiring.** The app's active language sets the utterance's `lang` (`en-US` / `es-US`) — the same toggle already driving UI string selection.
5. **Session reset behavior.** Narration off, language back to English when the kiosk resets between users. Without this, the next person at the kiosk inherits the previous user's settings.

**Discovering narration exists, and audio routing**

On touch, before any narration mode is active, the kiosk speaker announces itself once: "Compass. Audio guidance available. Headphone jack and keypad on the lower right. For audio guidance, press any button on the keypad or insert headphones." Pressing any keypad button, or inserting headphones, puts the app into narration mode from that point on — this is how a user who can't see the screen discovers the feature exists at all.

Once active, narration routes to headphones when present, and the speaker is suppressed entirely. This isn't just UX polish: narration through an open speaker means the kiosk announces "Domestic violence shelter, Safe Shelter Savannah" out loud in a public lobby — a real safety concern for that population, not merely an annoyance.

Worth testing before relying on it: whether `navigator.mediaDevices.ondevicechange` actually fires in the kiosk's locked-down browser shell on headphone insertion. If it does, narration can auto-start on plug-in — the same behavior JAWS for Kiosk provides natively. If it doesn't fire reliably in that environment, the spoken attract-loop announcement above is what carries discoverability instead.

**The real scope of "narration"**

ARIA labels don't produce speech — they're metadata, not an execution path. The actual work is a focus-tracking narrator: a layer that tracks what's focused or selected in the UI, reads the corresponding accessibility-tree content, and calls `speechSynthesis` to speak it. This — not the speak/stop/pause wrapper in item 1 above — is the largest single piece of implementation work in the narration feature, and none of the schema or pipeline work described elsewhere in this ADR gets it for free. It's entirely ahead of the team.

**Where the map vendor touches this**

Only one place: directions text.

CSAH's own content — shelter names, hours, addresses — comes from the database and flows entirely through the display/spoken pipeline above; no vendor terms apply to any of it. Directions text is different: it arrives from the map API at request time, already phrased for turn-by-turn narration in the requested language, and bypasses the display/spoken pipeline entirely. Two consequences:

- **Quality is unreviewed.** Abbreviations like "St" or "N" may not expand cleanly in speech. Needs testing against real routes, in both languages, before relying on it.
- **This is exactly where ADR-008's Google TTS clause lands.** Passing Google's directions text into a speech synthesizer is the ambiguous case flagged there — see ADR-008, "Blocking the Decision." Mapbox and OpenRouteService carry no equivalent restriction, so narrating their directions text is unambiguous regardless of which vendor ADR-008 ultimately selects for maps overall.

**Content Tiers**

Content is classified by translation risk, not by field type:

| Tier | Examples | Handling |
|---|---|---|
| Structured / enumerated | service type, eligibility category, population served, days & hours, accessibility features | Controlled vocabulary (~150 terms). Human-translated once, reused across every resource. Never touches MT. |
| Free prose | descriptions, notes, what to bring | MT draft, human review as a trailing pass |
| Never translated | organization names, addresses, phone numbers | Excluded from the MT call or glossary-pinned. "Union Mission" must remain "Union Mission" or a Spanish-speaking user cannot ask for it by name on arrival. |

The purpose of the split is safety. Content where a mistranslation causes real harm — "men only," "arrive before 7pm," "no intoxicated guests" — never passes through machine translation at all. It also shrinks the free-prose surface enough that human review remains realistic over time.

**Consequences**
- Cost of translation is effectively zero and structurally bounded — MT is called on write, not on read
- Kiosk has no runtime dependency on any translation or accessibility vendor; if the provider is down, stored content still renders and narrates
- CSAH edits content, including button labels, without a developer or a deploy
- Adding a third language is rows plus a formatter, not a schema migration
- One code path for rendering and one for narration across all locales
- Attribution and timestamp are captured per locale row without a separate audit mechanism
- Safety-critical content is structurally excluded from machine translation
- No third-party subscription, no vendor lock-in, no additional tool for CSAH to learn

**Risks**

*Accepted:* Spanish text expansion may break kiosk layouts. Spanish typically runs 15–30% longer than equivalent English. On a kiosk with fixed-width buttons and large touch targets, a longer label can wrap, clip, or break alignment. A side-by-side text editor makes the translation look correct while hiding the layout consequence entirely.

This risk is accepted rather than eliminated. Mitigations, in order of cost:
- `max_length` per `ui_strings` key for constrained UI, with a character counter and overflow warning in the editor
- Layout tested at longest-plausible-string during design, rather than per-edit
- Editors can view the live app in Spanish in a second tab at any time

The residual risk is a temporarily awkward-looking button, caught on the next visual check. Given an 11-person team editing infrequently, this does not justify building in-context preview tooling.

- **Unreviewed machine translation reaches the public.** Content publishes immediately; review trails. Mitigated by the tier split keeping safety-critical fields out of MT entirely, and by the reviewed baseline at launch. The alternative — holding edits until reviewed — means stale English-correct information stays live, which is worse.
- **Review backlog may go unworked.** The digest makes it visible but nothing enforces it. This is explicitly CSAH's responsibility, accepted as such.
- **Web Speech API voice availability is device-dependent.** Spanish voices ship on essentially every platform, so this is low risk at current scope. It must still be confirmed with kiosk hardware vendors — the criterion is not "does your browser expose the Web Speech API" but "list the installed TTS voices." Fallback if a voice is missing is server-side pre-generated audio into Supabase Storage, using the same write-time pipeline.
- **MT provider not yet selected.** Deferred, not blocking — the schema is provider-agnostic.
- **Deafblind users are out of scope.** No braille output path exists or is planned. Accepted limitation, not deferred — flagging it here so it's a documented decision rather than a silent gap.

**Open Questions**
- Narration scope — full resource record, or essentials only? Determines how many fields need a distinct spoken value and how much formatter work is required. Jen / Kishia.
- MT provider — DeepL, Google Cloud Translation, or Azure Translator. Confirm Latin American Spanish (es-419 / es-US) is supported as a distinct target from generic es.
- Nonprofit credits — Google for Nonprofits and Microsoft's Azure nonprofit grant would both cover this volume many times over. Confirm eligibility before attaching a payment method. Kay.
- Kiosk vendor TTS voice inventory — add to hardware evaluation criteria. Low risk for Spanish, still worth confirming in writing.
- Per-field review status — currently row-level (`last_edited_by` on the whole translation row). If reviewers need to vet a description without touching the notes, this needs to move to field level. Decide when speccing the admin UI.
- Speech rate — default `speechSynthesis` rate reads as too slow; needs a user-adjustable rate control, not just a fixed setting.
- Orientation announcements — e.g. "Resources. 7 options." — announcing where the user is and what's available on a given screen.
- Position state — e.g. "Shelters, 3 of 7" — announcing position within a list as the user navigates through it.
- Turn-by-turn narration text — directions are visual-only right now; narrating them requires generating spoken turn-by-turn text, not just reading the visual route (see "Where the map vendor touches this," above).
- Alert interrupts — an incoming alert needs to `cancel()` the current speech queue rather than queue behind whatever's already speaking.
- PIT survey translation (roadmap, not MVP) — the survey lives in the partner app but staff administer it to clients. If an outreach worker walks a Spanish-speaking person through the questions, question text must render in Spanish on the worker's screen, which pulls translation into the partner app surface. Kishia, when PIT is scoped.

---

## ADR-011 — Hosting Platform: Netlify

**Date:** 2026-08-03
**Status:** Accepted

**Context**
ADR-006 established a Git-based CI/CD pipeline on "Netlify or Vercel" without picking between them, and ADR-007 established a single codebase serving both the kiosk (`/kiosk/*`) and partner app (`/app/*`) from one deployment. The project needed a production host for that deployment: one that serves a React PWA at low/no recurring cost for a nonprofit budget, supports serverless functions for Twilio SMS (ADR-002) without standing up a separate backend, and gives Alan/Jen a shareable preview URL for review before production release.

**Decision**
Use Netlify as the production hosting platform for both the kiosk and partner app surfaces.

**Options Considered**
Eight platforms were scored across cost, uptime, security, deploy ease, scale, support, monitoring, demo capability, Twilio integration, and PWA support — full breakdown in `docs/hosting-comparison.html`.
1. **Netlify** (chosen) — free tier covers CSAH's expected traffic with no commercial-use restriction (~100GB/mo bandwidth), native PWA/service-worker support, Netlify Functions handle Twilio server-side with no separate backend, branch preview URLs for demos
2. **Cloudflare Pages** — closest runner-up; unlimited free bandwidth beats Netlify outright, but Functions run on Workers' V8 isolates rather than full Node.js, so Twilio requires calling the REST API directly instead of using the Node SDK (~1hr extra setup, not a blocker)
3. **Vercel** — technically at parity with Netlify on every other dimension, but its free tier prohibits commercial use (nonprofits included), forcing a $20/mo minimum
4. **Render** — kept as the fallback if two-way inbound Twilio SMS ever requires a persistent Node process instead of serverless functions ($7+/mo)
5. **Railway, Replit** — ruled out (Railway: two platform-wide outages in the prior 8 months; Replit: highest cost of any option evaluated for the least capability)

**Consequences**
- $0/mo at current expected traffic; a Netlify usage alert will be set to flag approaching the free-tier bandwidth/build-minute limits before they're hit
- $19/mo Pro is the defined upgrade path if priority support or headroom is needed later — not a default cost
- Resolves ADR-006's open Netlify-vs-Vercel question; the staging/production branch-deploy strategy in ADR-006 now has a confirmed platform
- Netlify Functions is the execution environment for Twilio SMS (ADR-002) — full Node.js, no separate backend service required for the MVP
- HTTPS is auto-provisioned, satisfying the hard HTTPS dependency for geolocation capture (ADR-009)
- Same Netlify deployment serves both `/kiosk/*` and `/app/*` per the monorepo structure (ADR-007) — no per-surface hosting decision needed

**Risks**
- No contractual SLA or service credits below Netlify's Enterprise tier — accepted given Netlify's independently measured >99.9% uptime and the low cost of brief downtime relative to Enterprise pricing
- Free-tier bandwidth (~100GB/mo) is finite; mitigated by the planned usage alert, with Pro ($19/mo) or a swap to Cloudflare Pages (unlimited bandwidth, same architecture) both available without a rebuild if traffic grows past projections
- If two-way inbound SMS becomes a hard requirement, Netlify Functions (serverless, stateless) are not the ideal execution model for that — Render is the identified fallback path, not yet built

---

## ADR-012 — Kiosk Request Flow: Database-First, SMS as Notification Only

**Date:** 2026-08-04
**Status:** Accepted

**Context**
Kiosk Compass lets someone at a kiosk request help (e.g., transportation, shelter) — per the architecture diagram, this triggers an SMS to the CSAH outreach team. Not to be confused with ADR-002, which is Twilio-as-fallback for the *partner app's own* push notifications (staff-to-staff coordination) — this ADR covers a different flow: a kiosk-initiated request reaching staff in the first place. An SMS-only design for that flow has a serious failure mode: if the text doesn't arrive — Twilio outage, staff phone off, carrier spam filter — the request simply doesn't exist anywhere. No row, no trace, no way to know it happened or recover it after the fact. That's a person who asked for help and got nothing. Compounding this: CSAH outreach staff work Mon–Sat, 8:30 a.m.–9 p.m., but the kiosk itself is reachable 24/7 — a request made at 10 p.m. Sunday has no one on shift to see it for roughly 10.5 hours.

**Decision**
Write the request to Postgres first. Send the SMS second, as a notification only — not the system of record.

- The kiosk request creates a row in the database the moment it's submitted. That row is the durable, queryable fact of "someone asked for X, at kiosk Y, at time Z" — it exists independent of whether the SMS ever arrives.
- The SMS is sent after the write succeeds, as a best-effort nudge to staff — e.g. "New transportation request, Bull St kiosk, 2:14pm" — with a link into the request queue in the partner/admin PWA.
- Staff acknowledge/resolve the request in the app; the row closes on that action, not on message delivery. This is the only way to know whether anyone responded — an SMS has no such state.
- The kiosk's confirmation screen checks the time **server-side** against outreach hours (Mon–Sat, 8:30 a.m.–9 p.m.) and renders different copy outside that window. Never "someone will contact you shortly" at 2 a.m. — the copy has to be honest about when a person will actually see the request.

**Options Considered**
1. **Database-first, SMS as notification** (chosen) — durable record regardless of SMS delivery outcome; enables acknowledgment state and honest, hours-aware kiosk copy
2. **SMS-only (fire-and-forget)** — rejected. A failed send means the request exists nowhere. No queue to check, no way to detect the failure, no way to follow up.

**Consequences**
- Every kiosk request is visible and auditable in the partner/admin PWA queue, independent of SMS deliverability — this is the same durability principle ADR-004 established for the resource directory, applied to a write path instead of a read path
- Enables an acknowledgment/resolution workflow (staff marks handled → row closes) that an SMS-only design structurally cannot support
- Feeds the same truth-telling/reporting story as F1 and the Mgmt Dashboard (F6) — request volume and response outcomes become real, queryable data instead of an untracked side channel
- Requires the kiosk confirmation screen to carry real logic (server-side clock check against outreach hours), not just static "thank you" copy
- Lowers the stakes on Twilio deliverability — since the database row is the actual source of truth, an SMS delivery failure degrades the experience (staff finds out late, from checking the queue) rather than losing the request entirely
- Pairs with ADR-013: this ADR keeps the *request* from being lost if the SMS fails; ADR-013 is how you'd actually detect that the SMS failed in the first place

**Risks**
- A request made outside outreach hours can still sit unacknowledged for up to ~10.5 hours — the database-first pattern guarantees the request isn't *lost*, not that it's answered faster. That's a staffing constraint, not something this pattern solves.
- A bug or timezone error in the server-side hour-check could show incorrect "we'll respond soon" copy at the wrong time — needs testing against real clock boundaries (including the Saturday 9pm→Sunday all-day→Monday 8:30am gap), not just the happy path.
- Acknowledgment state adds a small amount of UI complexity to the partner app (a queue view, a "mark handled" action) beyond what a pure notification-only design would need.

**Open Questions**
- Where does this queue live in the partner app UI — a dedicated "Kiosk Requests" view, or merged into the existing F1 notification/case queue?
- Does an unacknowledged request need an escalation path (e.g., re-notify after N minutes), or is staff checking the queue at shift start sufficient for MVP?
- Exact copy for the outside-hours confirmation screen — needs Jen/Kishia input, same as other user-facing kiosk copy decisions.

---

## ADR-013 — SMS Delivery Observability: Log Twilio Status Webhooks

**Date:** 2026-08-04
**Status:** Accepted

**Context**
SMS can fail to reach a recipient without any error surfacing to the sending application — silent carrier-side filtering, an under-trust-scored or misconfigured A2P 10DLC campaign (see ADR-002), or a bad number. This project has three SMS use cases: F1's push-notification fallback, F5's partner → individual outbound messages, and ADR-012's kiosk request nudge. ADR-012 in particular treats the SMS as "just a nudge" on top of a database row that's the real source of truth — but that framing only holds up if there's a way to know, after the fact, whether the nudge actually landed. Without delivery tracking, a consistently failing SMS channel is invisible: CSAH would have no real delivery rate to look at, and a pattern of failures (a carrier block, a registration problem) could run for months undetected.

**Decision**
Every outbound Twilio SMS is logged with its delivery status, using Twilio's status callback webhook. Each message row carries, at minimum: message SID, recipient, timestamp sent, and a status field the webhook updates as Twilio reports it (`queued` → `sent` → `delivered` / `undelivered` / `failed`), including Twilio's error code when one is returned.

**Options Considered**
1. **Status webhook logged per send** (chosen) — Twilio POSTs delivery status updates to a callback URL; store status against the message row. Standard Twilio feature, no added cost.
2. **No delivery tracking (fire-and-forget)** — rejected. This is exactly the blind spot ADR-012 was written to eliminate for the kiosk flow; leaving F1 and F5 without it would just relocate the same problem.
3. **Poll Twilio's Message resource API periodically** — rejected as unnecessary complexity. Webhooks are push-based and lower-latency for data Twilio already provides for free.

**Consequences**
- CSAH gets a real, measured delivery rate instead of an assumed one — feeds the Mgmt Dashboard (F6) truth-telling story, and surfaces a failing campaign registration or carrier block as data instead of silence
- Requires a webhook endpoint (a Netlify Function) that's publicly routable and validates Twilio's request signature — this is a real build item, not a config toggle
- Complements ADR-012 rather than duplicating it: ADR-012's acknowledgment state tells you a human saw the request; this ADR's delivery status tells you whether the SMS itself arrived. Different failure modes, both need visibility.
- One webhook handler serves all three SMS callers (F1, F5, ADR-012) — a single piece of infrastructure, not three

**Risks**
- The status webhook itself can fail to arrive (network blip, endpoint downtime) — the message row would just stay at `sent` indefinitely. A degraded state, not a regression from having no tracking at all.
- Twilio's status callback must be signature-validated, or the endpoint becomes a spoofable write path into message status. Twilio's SDK provides the helper for this — it has to actually be used, not skipped for expediency.
- Adds a small amount of schema and infrastructure (status field, webhook route) that needs to exist before the first SMS ships, not retrofitted after CSAH asks why nobody knows the real delivery rate.

**Open Questions**
- Does an `undelivered`/`failed` status trigger any automated action (retry via a second channel), or is it visibility/reporting only for MVP? Leaning visibility-only, since ADR-012 already makes the database row the durable source of truth regardless of SMS outcome.
- Where does aggregate delivery-rate reporting surface — a Supabase view feeding the Mgmt Dashboard (F6), alongside the referral/encounter views from ADR-005?

---

## ADR-014 — Push Notification Architecture: Web Push API Direct

**Date:** 2026-08-04
**Status:** Accepted

**Context**
The partner app needs push notifications for crisis coordination (F1) — alerts must reach a phone even when the app isn't open on screen (locked, backgrounded, in a pocket). This is architecturally distinct from in-app queue updates like ADR-012's kiosk request queue, which use Supabase Realtime — already in the stack, no new dependency, but only works while the app is actually open on screen. Out-of-app push is a different mechanism entirely: the Web Push API, a service worker, and VAPID — genuinely new infrastructure, not an extension of Realtime. Three vendor approaches were evaluated: (A) direct Web Push using a self-generated VAPID keypair and the `web-push` library in a Supabase Edge Function; (B) Firebase Cloud Messaging (FCM) for web; (C) a managed push vendor (e.g., OneSignal-style).

**Decision**
Build push notifications as direct Web Push (Option A): CSAH/ARC generates its own VAPID keypair, `web-push` runs in a Supabase Edge Function, and subscriptions are stored in Postgres (RLS-scoped). No third-party push vendor. Delivery path: browser → Apple/Google/Mozilla's own push service → device.

**The stack, and why only one layer actually varies**
Push notifications break into ten layers. Nine are identical no matter which vendor option is chosen:

| # | Layer | What it is | Varies by option? |
|---|---|---|---|
| 1 | Service worker registration | Client registers `sw.js`; required before any push subscription exists | No — universal |
| 2 | Subscription | `PushManager.subscribe()` with the VAPID public key; returns endpoint + auth keys | No |
| 3 | Subscription storage | `push_subscriptions` table, RLS-scoped | No |
| 4 | Trigger | Postgres trigger → database webhook or `pg_net` → Edge Function | No |
| 5 | Recipient resolution | Who gets this? Hand-written logic | No |
| 6 | Encryption + VAPID signing | RFC 8291 payload encryption, RFC 8292 JWT signed with the VAPID private key | Partially |
| 7 | Delivery | Push service hands off to the device | **Yes — the only layer the options differ on** |
| 8 | Receipt | Service worker `push` event fires | No |
| 9 | Display | `showNotification()` in the service worker | No |
| 10 | Click handling | `notificationclick` → focus or open the app at the queue | No |

Layers 1–5 and 8–10 are the same build regardless of vendor choice — that's most of the work. Vendor choice (A/B/C) only really changes layer 7, and partially layer 6.

**Service worker is universal, not option-specific**
A service worker is required for every option, not just Option A — FCM's web SDK requires shipping `firebase-messaging-sw.js`; OneSignal injects its own. There's no push-without-service-worker path, because the service worker is what stays alive when the app is closed and receives the push event. One exception: Safari 18.4 introduced Declarative Web Push, which doesn't require a service worker — but it's Safari-only, so the service worker path is still needed for Android and desktop regardless. Not a plan, just worth knowing it exists if iOS install friction becomes a real problem later.

This creates a real sequencing dependency: push requires a registered service worker, and service worker / offline / PWA update strategy is explicitly parked for its own dedicated session. Push scope is downstream of that session — layer 1 shouldn't be built in isolation. That session needs to happen before push gets committed to a sprint.

**VAPID is identity, not encryption — and it's where the options actually differ**
VAPID is how a push service knows who sent the message: one keypair, public half goes to the client at subscribe time, private half signs a JWT on every send.

| Option | VAPID keypair |
|---|---|
| A — Direct | Generated by CSAH/ARC, private key stored in Supabase secrets. CSAH owns it. |
| B — FCM | Still generated/configured for web push, but held inside Google's Firebase project. |
| C — Managed vendor | Vendor generates and holds it; CSAH may never see the private key. |

Rotating or losing the VAPID keypair invalidates every subscription and forces all ~50 users back through "Add to Home Screen" and re-permission. On Option C, the vendor controls whether that ever happens. On Option A, CSAH controls it — the right property for a system CSAH inherits and maintains long after this engagement ends.

**Options Considered**
1. **Option A — Web Push direct** (chosen) — self-generated VAPID keys, `web-push` in a Supabase Edge Function, subscriptions in Postgres. No vendor, no cost, no account to hand off.
2. **Option B — Firebase Cloud Messaging (FCM)** — Google holds the VAPID key inside a Firebase project; adds a vendor account CSAH doesn't otherwise need.
3. **Option C — Managed push vendor** (e.g., OneSignal) — vendor generates and holds the VAPID key entirely; CSAH has no direct control over the one credential that can invalidate every subscription.

**Consequences**
- No new vendor account, subscription, or dependency to hand off to CSAH at the end of the engagement
- The credential capable of invalidating all ~50 users' subscriptions (the VAPID private key) lives in CSAH's own Supabase secrets — not a vendor's infrastructure
- Push work is explicitly sequenced behind the (already-parked) service worker/offline/PWA update strategy session — can't be scheduled independently of it
- Nine of ten architectural layers are unaffected by this decision and would need to be built identically under Option B or C — this decision has a narrower blast radius than "push notifications" as a whole might suggest

**Risks**
- CSAH (or ARC on CSAH's behalf) is now responsible for VAPID key custody and rotation discipline — there's no vendor safety net if the key is lost or mishandled, though that's also exactly the point of this decision
- `web-push` running in a Supabase Edge Function means ARC owns the encryption/signing implementation (RFC 8291/8292) rather than delegating it to an SDK — more surface area to get right, though these are well-documented, stable specs
- Safari's Declarative Web Push (no service worker required) is not used in this design; if iOS-specific install/permission friction becomes a real adoption blocker later, that's a possible future mitigation, not something this ADR builds toward now

**Open Questions**
- Resolves and replaces the earlier open item "Push notification provider: Expo Push vs FCM direct" (`open.md`) — that framing predates this analysis and doesn't reflect the options actually considered here.
- Exact sequencing: which session addresses service worker / offline / PWA update strategy, and when, relative to when push work gets scheduled?
- Recipient resolution logic (layer 5) — who receives a given alert — still needs to be specced out; this ADR fixes the delivery mechanism, not the routing rules.

---


