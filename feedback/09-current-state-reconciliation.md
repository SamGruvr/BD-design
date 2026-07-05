# Current-State Reconciliation (BD_ITA)

**Validated against code repo:** `BD_ITA`
**Validation date:** 2026-06-13
**Reference commit:** `7833339` (POC completion plan A1-D6, live-verified on production)
**Latest Phase 1 baseline tag:** `internal-preview-2026-05-01-r2`
**Prior validation:** 2026-06-11 @ `7b4479f`

This file reconciles the original feedback package with the current implementation state. Verified directly against source files, not self-reported status.

---

## Status Legend
- `Closed` = issue addressed in current build
- `Open` = still valid gap
- `Partial` = partially addressed; follow-up still needed
- `Superseded` = statement is outdated by later implementation

---

## Key Reconciliation Results

| Item | Topic | May 2 Status | Current State (verified 2026-06-11) | Status |
|---|---|---|---|---|
| — | Full hosted verification | Closed | Unchanged | `Closed` |
| — | Step 2/3 weighted scoring | Closed | Unchanged | `Closed` |
| — | Deal context hosted 500 risk | Closed | Unchanged | `Closed` |
| — | Admin unresolved assignments | Closed | Unchanged | `Closed` |
| C3 | CSP header | Open | Implemented in `backend/src/middleware/securityHeaders.js` | `Closed` |
| H1 | Risk/action route validation | Open | `validateBody` contracts on both `risks.js` and `actions.js` update routes | `Closed` |
| H6 | Integration DB test validity | Open | Rewritten: real `SELECT 1` query, fails on DB error. `catch/toBeDefined` pattern removed | `Closed` |
| S2–S5 | CI security gates | Open | `codeql.yml`, `gitleaks.yml`, `dependency-review.yml`, `dependabot.yml` all present in `.github/` | `Closed` |
| SC2 | Pagination on `Deal.list()` | Not tracked | `list(filters, limit = 50, offset = 0)` with `LIMIT/OFFSET` in query | `Closed` |
| C1 | TLS `rejectUnauthorized` | Open | **Closed 2026-06-11.** `DB_SSL_CA` env var support added (commit `a7d33d4`), Supabase CA cert + `DB_SSL_REJECT_UNAUTHORIZED=true` set in Vercel production, deployed and login-verified with strict TLS | `Closed` |
| C2 | DDL-on-demand | Open (unlisted May 2) | `ensure*()` schema functions still present in `User.js`, `Role.js`, `AdminActionQueue.js`, `gateReadiness.js`, `UserRepository.js` | `Open` |
| H3 | Distributed rate limiting | Open | Still in-memory `Map` in `rateLimit.js`; no Redis | `Open` |
| M4 | `gateReadiness.js` test coverage | Open | **Closed 2026-06-13.** 7 unit tests added (`backend/tests/unit/gateReadiness.test.js`) covering weighted scoring, minimum-section blocking, and tier boundaries. Tests documented rather than fixed two pre-existing quirks: empty sections score 100% (vacuous), and Step 3 weights sum to 110, not 100 | `Closed` |
| M5 | Auth tests fragile handler extraction | Open | Still Express stack traversal; now selects last handler (less fragile, still internal-API dependent) | `Open` |
| S1 | `npm audit` blocking in CI | Open | No audit step in `quality-gate.yml` at all | `Open` |
| M7 | ESLint in CI | Open | Absent from `quality-gate.yml` | `Open` |
| T3 | Coverage threshold | Open | Absent | `Open` |
| D6 | `/health` DB connectivity check | Not tracked (found 2026-06-11) | **Closed 2026-06-13.** Endpoint now runs `SELECT 1` and returns `503`/`degraded` on failure instead of a no-op `ok` | `Closed` |

---

## Reconciled Priority Order (Now)

1. `C2` Replace DDL-on-demand with startup migrations
2. `H3` Serverless-safe rate limiting (Redis-backed)
3. `S1`/`M7`/`T3` CI hardening: blocking `npm audit`, ESLint, coverage threshold
4. `M5` Rewrite auth route tests with `supertest`
5. `M10` Normalize `gateReadiness.js` section weights (Step 3 sums to 110) and empty-section scoring semantics — new finding from `M4` test coverage, not yet fixed
6. VOC stakeholder dropdown (sourced from deal stakeholders, "add new" path) — PRD §5.3.3 partial
7. Surface `teaming_rationale` and partner size/demography in the Partners tab UI — data model/API ready, UI not updated
8. `Phase 1.5` define-first deal-search execution (spec drafted, not implemented)

---

## Notes

- The original review package remains valuable as an architectural/security backlog.
- This reconciliation is the source-of-truth overlay for current execution.
- Before opening new implementation work from `08-codex-action-plan.md`, check this file first to avoid repeating already completed items.
- `10-poc-completion-plan.md` is the source-of-truth for PRD v1.3 / deck-parity closure (Groups A-D); check it before starting new deck-parity work.
- Known doc-set divergence: `BD_ITA/docs/review-pack/` has 14 files; `BD-design/review-pack/` has 10. Files 11–14 exist only in the code repo. Decide one home and sync.
