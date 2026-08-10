# Tech Stack Decisions

## Status Key
- ✅ Decided
- 🔶 Leaning / likely
- ❓ Open

---

## Frontend

| Layer | Choice | Status | Notes |
|---|---|---|---|
| App platform | PWA | ✅ | Avoids gov't MDM distribution problem; SMS fallback covers iOS push weakness; Expo is upgrade path only if device policies allow |
| UI framework | TBD | ❓ | React is likely given Allyson's background; no decision recorded |
| Offline support | Service worker (PWA) | ✅ | Cache critical UI; queue writes for sync on reconnect |

---

## Backend

| Layer | Choice | Status | Notes |
|---|---|---|---|
| Hosting | TBD | ❓ | Netlify, Vercel, Loveable, and others under consideration; lean/affordable required; no large managed cloud |
| API style | REST | 🔶 | No decision recorded; REST is default unless real-time requirements push toward WebSockets/GraphQL subscriptions |
| Language/Runtime | TBD | ❓ | Node.js likely given Allyson's stack; not formally decided |

---

## Database

| Layer | Choice | Status | Notes |
|---|---|---|---|
| DB platform | TBD (custom DB) | 🔶 | No HMIS integration; building custom schema from scratch; Supabase (Postgres) is a strong candidate |
| HMIS integration | Not now | ✅ | 2-year HUD approval process; revisit post-MVP |
| Schema reference | HMIS data dictionary | ❓ | Request from Keisha (HMIS compliance) to inform custom schema design — not to implement |

---

## SMS

| Layer | Choice | Status | Notes |
|---|---|---|---|
| SMS provider | Twilio | 🔶 | Industry standard; best API; handles 2-way threading; ~$20–50/mo at estimated volume |
| Volume estimate | ~250 notifications/mo | 🔶 | Low; cost is negligible at this scale |
| 2-way threading | Required | ✅ | Partner → individual outbound; inbound replies thread back to case record |

---

## Push Notifications

| Layer | Choice | Status | Notes |
|---|---|---|---|
| Push mechanism | PWA push (Web Push API) | 🔶 | Native to PWA; works on Android; iOS reliability issues |
| iOS fallback | SMS via Twilio | ✅ | Covers push delivery gaps; ~250/mo volume |
| Push + SMS duplication | TBD | ❓ | Open question: too invasive? User preference (push only / SMS only / both)? |

---

## Kiosk

| Layer | Choice | Status | Notes |
|---|---|---|---|
| Kiosk software | TBD (KioWare vs. alternatives) | ❓ | SCAD recommends KioWare (~$2k/device/yr); Scalefusion is compelling alternative (~$25–50/device/yr); decision pending vendor confirmation |
| Kiosk data source | Same backend API as partner app | ✅ | Single data source; kiosk lockdown software just points to a URL; updates propagate automatically |
| Kiosk hardware | TBD | ❓ | $10–25k per unit; 3 units planned; procurement not started |
| Priority locations | Goodwill Opportunity Campus, Union Mission Resource Center, Public Library | ✅ | Private nonprofit property avoids permitting delays |

---

## Auth

| Layer | Choice | Status | Notes |
|---|---|---|---|
| Auth provider | TBD | ❓ | Admin-provisioned accounts; no self-registration; ~50 users |
| Access control | Role/agency scoping | 🔶 | Exact permission model TBD |

---

## Hosting & Infrastructure

| Layer | Choice | Status | Notes |
|---|---|---|---|
| App hosting | Netlify (Pro) | ✅ | See ADR-011; Pro required from day one, not a growth-triggered upgrade — see note below |
| Doc hosting (internal) | Netlify (docs/ folder) | ✅ | Auto-deploys from GitHub on push to main; serves decision docs to Alan |
| Repo | GitHub (private) | ✅ | Netlify free tier handles a private repo on a personal account, but **not** one owned by a GitHub Organization — CSAH's app repo will be exactly that. Pro required as a result. See ADR-011. |

---

## Not Using (and Why)

| Technology | Reason |
|---|---|
| HMIS / Caseworthy API | 2-year HUD approval process; not viable for MVP |
| Native mobile app (iOS/Android) | Gov't device MDM policies may block installation; PWA avoids this |
| AWS (as primary) | Not the preferred forward path; managed services preferred for this scale and budget |
| Java | Non-starter |
