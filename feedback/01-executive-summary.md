# Executive Summary

**Project:** BD Capture & Intelligence Platform (POC)
**Stack:** Node/Express · React/Vite · PostgreSQL (Supabase) · Vercel · Socket.io (disabled)
**Review date:** 2026-05-02

> Reconciliation note: This summary reflects the initial audit snapshot. See `09-current-state-reconciliation.md` for validated current status against latest code.

---

## Scorecard

| Area | Grade | Key Finding |
|---|---|---|
| Architecture | B+ | Sound layering; DDL-on-demand is structural flaw |
| Code Quality | B | Consistent conventions; raw body pass-through on risk/action routes |
| Security | C+ | Good auth implementation; TLS cert check disabled, CSP absent, serverless rate limiting bypassed |
| Testing | C | Tests verify shape not behavior; integration tests non-functional; critical logic untested |
| CI/CD | B- | Gate runs on PRs; `npm audit` non-blocking; no lint, no coverage threshold |
| Observability | C | Console logs only; audit log fire-and-forget; no request correlation IDs |
| Scalability | C+ | No pagination, no query timeouts, pool config wrong for serverless, no caching |
| Documentation | B- | PRD is strong; review-pack too thin; no ADRs, no OpenAPI, no ERD |
| Docs↔Code Alignment | C+ | Test count conflict, Socket.io status conflict, role/persona mismatch |

---

## Top Risks

### Critical — Fix First
1. **TLS cert verification disabled** on production database connection (`rejectUnauthorized: false`)
2. **DDL executes on every API call** — `ALTER TABLE` in model methods causes lock contention at load
3. **No Content-Security-Policy header** — XSS can steal JWT tokens from `localStorage`

### Already Closed Since Initial Snapshot
1. Full hosted verification now passes (`verify:full`)
2. Step 2/3 weighted scoring is implemented and documented
3. Hosted Deal Context API 500 from schema drift is remediated
4. Unmatched assignee workflow is remediated via admin action queue

### High — Fix Before Scale
4. Raw `req.body` passed to `Risk.update()` and `Action.update()` — no contract validation
5. Audit log is fire-and-forget — failed writes are silent and undetected
6. Rate limiter uses in-memory `Map` — bypassed entirely in Vercel serverless (each invocation = new process)
7. Integration tests pass on DB failure — false assurance in CI

### Scalability Floor
8. No pagination on `Deal.list()` — full table scan returns all rows
9. No DB connection pool sizing for serverless — risk of exhausting Supabase connection limit
10. `computeGateReadiness()` runs 10 parallel DB queries per request, every time — no caching

### Security Posture
11. No SAST, no secret scanning, no dependency review gate in CI
12. `npm audit` runs but failure is non-blocking — CVEs ship to production
13. No automated dependency update mechanism (no Dependabot)

---

## Recommended Fix Sequence

Work through [08-codex-action-plan.md](08-codex-action-plan.md) in order. Phase 1 items must complete before Phase 2.

| Phase | Focus | Files |
|---|---|---|
| 1 — Critical | TLS, DDL migration, CSP | `02-architecture.md`, `03-security.md` |
| 2 — High | Validation, audit log, rate limiter, tests | `04-code-quality.md`, `05-testing.md` |
| 3 — Scalability | Pool, pagination, cache, timeouts | `06-scalability.md` |
| 4 — CI Hardening | SAST, secret scan, Dependabot | `03-security.md` |
| 5 — Docs | OpenAPI, ADRs, reconcile conflicts | `07-documentation.md` |
