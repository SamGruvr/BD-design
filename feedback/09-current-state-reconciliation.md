# Current-State Reconciliation (BD_ITA)

**Validated against code repo:** `BD_ITA`
**Validation date:** 2026-06-11
**Reference commit:** `7b4479f`
**Latest Phase 1 baseline tag:** `internal-preview-2026-05-01-r2`
**Prior validation:** 2026-05-02 @ `ef43a04`

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
| C1 | TLS `rejectUnauthorized` | Open | Env-driven: `DB_SSL_REJECT_UNAUTHORIZED === 'true'`, CA path supported via `DB_SSL_CA_PATH`. **Defaults to `false` when env var unset.** Secure only if production env sets it. Vercel prod env not verified | `Partial` |
| C2 | DDL-on-demand | Open (unlisted May 2) | `ensure*()` schema functions still present in `User.js`, `Role.js`, `AdminActionQueue.js`, `gateReadiness.js`, `UserRepository.js` | `Open` |
| H3 | Distributed rate limiting | Open | Still in-memory `Map` in `rateLimit.js`; no Redis | `Open` |
| M4 | `gateReadiness.js` test coverage | Open | Zero tests on core scoring logic | `Open` |
| M5 | Auth tests fragile handler extraction | Open | Still Express stack traversal; now selects last handler (less fragile, still internal-API dependent) | `Open` |
| S1 | `npm audit` blocking in CI | Open | No audit step in `quality-gate.yml` at all | `Open` |
| M7 | ESLint in CI | Open | Absent from `quality-gate.yml` | `Open` |
| T3 | Coverage threshold | Open | Absent | `Open` |

---

## Reconciled Priority Order (Now)

1. `C1` Confirm `DB_SSL_REJECT_UNAUTHORIZED=true` set in Vercel backend production env (config check, not code change)
2. `M4` gateReadiness unit tests — core business logic, silent scoring bugs possible until covered
3. `C2` Replace DDL-on-demand with startup migrations
4. `H3` Serverless-safe rate limiting (Redis-backed)
5. `S1`/`M7`/`T3` CI hardening: blocking `npm audit`, ESLint, coverage threshold
6. `M5` Rewrite auth route tests with `supertest`
7. `Phase 1.5` define-first deal-search execution (spec drafted, not implemented)

---

## Notes

- The original review package remains valuable as an architectural/security backlog.
- This reconciliation is the source-of-truth overlay for current execution.
- Before opening new implementation work from `08-codex-action-plan.md`, check this file first to avoid repeating already completed items.
- Known doc-set divergence: `BD_ITA/docs/review-pack/` has 14 files; `BD-design/review-pack/` has 10. Files 11–14 exist only in the code repo. Decide one home and sync.
