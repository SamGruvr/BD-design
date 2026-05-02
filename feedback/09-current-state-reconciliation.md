# Current-State Reconciliation (BD_ITA)

**Validated against code repo:** `BD_ITA`
**Validation date:** 2026-05-02
**Reference commit:** `ef43a04`
**Latest Phase 1 baseline tag:** `internal-preview-2026-05-01-r2`

This file reconciles the original feedback package with the current implementation state.

---

## Status Legend
- `Closed` = issue addressed in current build
- `Open` = still valid gap
- `Partial` = partially addressed; follow-up still needed
- `Superseded` = statement is outdated by later implementation

---

## Key Reconciliation Results

| Topic | Original Feedback Position | Current State | Status |
|---|---|---|---|
| Full hosted verification | Non-dry-run validation incomplete | `verify:full` passes (diagnostic + browser + build + tests + security artifact) | `Closed` |
| Step 2/3 weighted scoring | Not fully formalized | Implemented with section weights + minimum checks; Market Evidence Option 2 added | `Closed` |
| Deal context hosted 500 risk | Runtime schema drift causes failures | Runtime schema guard added for context routes; hosted parity pass recorded | `Closed` |
| Admin unresolved assignments | Workflow blocks on unmatched users | Unmatched names allowed + admin action queue + filters/notes | `Closed` |
| CSP header | Missing CSP | Still not implemented in backend header middleware | `Open` |
| TLS `rejectUnauthorized` | Disabled in DB config | Still `rejectUnauthorized: false` in `backend/src/config/database.js` | `Open` |
| Distributed rate limiting | In-memory map not serverless-safe | Still in-memory (`Map`) | `Open` |
| Risk/action route validation | Raw body update pass-through | Still no `validateBody` middleware on update routes | `Open` |
| Integration DB test validity | test passes on DB failure | Still present in `backend/tests/integration/setup.test.js` | `Open` |
| Auth tests fragile handler extraction | Express stack index based | Still used in `backend/tests/unit/auth.route.test.js` | `Open` |
| CI security gates (CodeQL/gitleaks/dependabot) | Missing | Still missing; only `quality-gate.yml` safe verify job present | `Open` |

---

## Reconciled Priority Order (Now)

1. `C1` TLS verification hardening
2. `C3` CSP header policy
3. `H3` distributed/serverless-safe rate limiting
4. `H1` risk/action contract validation
5. `H6` integration DB test correctness
6. `S1-S5` CI security hardening (audit blocking, secret scan, SAST, dependency review, Dependabot)
7. `Phase 1.5` define-first deal-search execution (spec drafted, not implemented)

---

## Notes

- The original review package remains valuable as an architectural/security backlog.
- This reconciliation should be treated as the source-of-truth overlay for current execution.
- Before opening new implementation work from `08-codex-action-plan.md`, check this file first to avoid repeating already completed items.
