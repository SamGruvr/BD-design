# Code Quality Review

---

## Naming Conventions

Generally consistent: `camelCase` JavaScript, `snake_case` database columns, domain-named route files.

### Issue — `directory_lead_id` Typo

**Files:** `database/schema.sql`, `backend/src/models/Deal.js`, `backend/src/routes/deals.js`, `backend/src/repositories/DealRepository.js`

Column name is `directory_lead_id` throughout. Should be `director_lead_id`. Semantic error propagated across the full stack and API contract. Requires a database migration to fix.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `M1`.

---

## Error Handling

### What Works

- Global error handler in `backend/src/middleware/errorHandler.js` — correctly catches all `next(err)` calls
- Custom error classes (`ValidationError`, `NotFoundError`, `UnauthorizedError`) — well-defined, consistently used in auth and deal routes
- JWT error types distinguished (expired vs. invalid)

### Issues

**H1 — No validation on risk/action routes**
`risks.js` and `actions.js` pass `req.body` directly to model update methods. No `validateBody` middleware. The only guard is `if (!deal_id || !risk_title)`. Inconsistent with `deals.js` and `users.js` which apply full contract validation.

**M — `isDatabaseUnavailableError` is fragile**
`backend/src/routes/auth.js` determines DB availability by checking string content of error messages. Will miss novel Postgres error codes and break if error message format changes.

**Fix:** Replace with explicit check against Postgres error codes (`error.code === '57P03'` for shutdown, `'08006'` for connection failure, etc.).

---

## Code Duplication

### H4 — `sanitizeUsername` Duplicated

Identical function defined in both `backend/src/routes/auth.js` and `backend/src/routes/users.js`. If username sanitization rules change, both must be updated in sync. Divergence risk creates username collision vulnerability.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `H4`.

### Medium — User Shape Normalization Inconsistent

`auth.js` uses a `mapAuthUser()` helper to normalize the user object shape. `users.js` constructs the shape inline as `{ id: user.id, username: ... }`. Two endpoints returning subtly different user shapes causes client-side schema drift.

**Fix:** Export `mapAuthUser` from a shared util and use it in both routes.

---

## Complexity

### `gateReadiness.js`

Most complex file in the codebase. ~200 lines, 10 parallel DB queries, multi-dimensional weighted scoring:
- Section weights: 35 / 20 / 20 / 15 / 10
- Minimum-percent thresholds: 70 / 80
- Recommendation tiers based on score bands

The weights and thresholds are magic numbers with no inline explanation of business origin. `computeGateReadiness` has high cyclomatic complexity. Zero test coverage.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) items `M4` and `M10`.

### `auth.js` Route File

320+ lines. 6 endpoints + 10 helper functions. Should be split into route file and service module.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `M8`.

---

## Comments and Documentation

### In-Code

Comments are present only as route labels (`// GET /api/deals`). No JSDoc anywhere. Magic numbers in `gateReadiness.js` have no explanation. The `.agents/` and `docs/` directories contain substantial external documentation but the code is sparsely commented where it matters most.

**Rule for Codex:** Only add comments when the WHY is non-obvious — a hidden constraint, a subtle invariant, or behavior that would surprise a reader. Business rule constants (weights, thresholds) always warrant a comment explaining their origin.

---

## Additional Low Issues

| Issue | Location | Fix |
|---|---|---|
| `app.js` body size limit is 8MB | `app.js` | Reduce to `1mb` — BD tracking app has no large payload use case |
| `console.log` in WebSocket hot paths | `backend/src/websocket/setup.js` | Replace with structured logger or remove in prod |
| `PUT` used for partial updates | `deals.js`, `risks.js` | Use `PATCH` — `PUT` semantics imply full replacement |
| No request correlation IDs | All routes | Add `x-request-id` header via middleware; include in all log output |

---

## Strengths

- Route files are thin — business logic correctly pushed to services and models
- Contract validation in `contractValidation.js` is lightweight and effective
- Active Record pattern in models is consistent and appropriate for this scale
- `verify-all.sh` produces structured JSON output — production-quality tooling
- Dependency count is lean — 8 production backend deps, no bloat
