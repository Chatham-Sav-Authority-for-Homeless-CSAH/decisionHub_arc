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
- Google TTS clause clarification & lazy cache clarification — determines whether Web Speech narration of directions is permitted. Also, whether a lazy read-through cache with 30-day expiry is acceptable.
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

**Open Questions**
- Narration scope — full resource record, or essentials only? Determines how many fields need a distinct spoken value and how much formatter work is required. Jen / Kishia.
- MT provider — DeepL, Google Cloud Translation, or Azure Translator. Confirm Latin American Spanish (es-419 / es-US) is supported as a distinct target from generic es.
- Nonprofit credits — Google for Nonprofits and Microsoft's Azure nonprofit grant would both cover this volume many times over. Confirm eligibility before attaching a payment method. Kay.
- Kiosk vendor TTS voice inventory — add to hardware evaluation criteria. Low risk for Spanish, still worth confirming in writing.
- Per-field review status — currently row-level (`last_edited_by` on the whole translation row). If reviewers need to vet a description without touching the notes, this needs to move to field level. Decide when speccing the admin UI.
- PIT survey translation (roadmap, not MVP) — the survey lives in the partner app but staff administer it to clients. If an outreach worker walks a Spanish-speaking person through the questions, question text must render in Spanish on the worker's screen, which pulls translation into the partner app surface. Kishia, when PIT is scoped.

---


