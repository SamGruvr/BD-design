# Architecture Review

---

## Current Structure

### Backend Layer Separation

| Layer | Path | Role |
|---|---|---|
| Routes | `backend/src/routes/` | HTTP surface, input parsing, response shaping |
| Models | `backend/src/models/` | Active Record against Postgres via `pg` |
| Repositories | `backend/src/repositories/` | Complex query aggregation |
| Services | `backend/src/services/` | Business logic (assignment, gate scoring, mailer, admin queue) |
| Middleware | `backend/src/middleware/` | Auth, rate limiting, error handling, audit, contract validation, security headers |

Layer separation is sound. Routes are thin. Services contain meaningful domain logic. Middleware stack order in `app.js` is correct: security headers → contract validation → CORS → audit → routes → 404 → error handler.

### Frontend

React 18 SPA with React Router v6. Context-based auth (`AuthContext`). Axios for API calls. Socket.io-client installed but non-functional in serverless deployment.

---

## Structural Flaws

### CRITICAL — DDL on Every API Call

**Files affected:** `backend/src/models/User.js`, `backend/src/models/Deal.js`, `backend/src/models/AdminActionQueue.js`, `backend/src/services/gateReadiness.js`, `backend/src/repositories/DealRepository.js`

**Pattern:**
```js
// Called at the top of every static model method
await this.ensureProfileColumns();       // User.js — 4x ALTER TABLE per call
await this.ensureBidPhaseColumn();       // Deal.js
await this.ensureTable();                // AdminActionQueue.js
await this.ensureReadinessSchema();      // gateReadiness.js
await this.ensureDealUiDataTable();      // DealRepository.js
```

Each call issues `ALTER TABLE ... ADD COLUMN IF NOT EXISTS` DDL. Under concurrent load this serializes DDL through the connection pool and causes PostgreSQL lock contention. Schema should be stable at runtime — DDL belongs in migrations, not hot paths.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `C2`.

---

### HIGH — Dual Abstraction on Deal Entity

`backend/src/models/Deal.js` and `backend/src/repositories/DealRepository.js` operate on the same entity without a documented contract between them. `Deal` model handles CRUD; `DealRepository` adds `getAggregatedDetails` and `saveDealUiData`. Justified but undocumented — future contributors will not know which to extend.

**Fix:** Add a comment block to `DealRepository.js` top explaining the split. No code change needed.

---

### MEDIUM — Raw Body Pass-Through on Risk and Action Routes

`backend/src/routes/risks.js`:
```js
const updated = await Risk.update(parseInt(id), req.body); // no validation
```
`backend/src/routes/actions.js`:
```js
const updated = await Action.update(parseInt(id), req.body); // no validation
```

`deals.js` and `users.js` apply `validateBody` middleware before model calls. Risks and actions do not. Inconsistent and relies on model-level allowlisting that is not enforced at the route boundary.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `H1`.

---

### MEDIUM — `auth.js` Route File Too Large

`backend/src/routes/auth.js` is 320+ lines handling 6 endpoints plus 10+ helper functions: `sanitizeUsername`, `mapAuthUser`, `isDatabaseUnavailableError`, `promoteBootstrapAdminIfMatch`, password reset helpers. Should be split into route file + controller/service module.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `M8`.

---

### LOW — HTTP Verb Inconsistency

Deals and risks use `PUT` for partial updates. `PATCH` is already used for `/api/users/:id`. `PUT` semantics imply full replacement; these updates are partial. Minor but inconsistent with the existing pattern in the same codebase.

---

## File Structure vs CLAUDE.md

CLAUDE.md specifies `execution/` (Python scripts), `directives/` (Markdown SOPs), `.tmp/` (intermediates). None exist in `BD_ITA`. The `.agents/` directory serves a similar purpose but uses a different convention. This is acceptable — CLAUDE.md is a generic template, and the `.agents/` + `docs/review-pack/` structure is a reasonable alternative.

### Committed Artifacts That Should Not Be in Source Control

| File/Dir | Issue |
|---|---|
| `frontend/dist/` | Build output — Vercel builds from source |
| `README copy.md` | Dev artifact |
| `TEST_RESULTS.txt` | Dev artifact |
| `CHECKLIST.md`, `INDEX.md`, `SUMMARY.md` | AI session artifacts |
| ~~`BD_Platform_PRD_v1.0.docx`~~ | Resolved — deleted from repo (commit `f8ba28e`) |
| `BD_Platform_Wireframe_v1.0.html` | Product artifact — belongs in `docs/` |

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `M3`.

---

## Strengths

- Middleware stack order is correct and follows security-first principle
- 20 route files map cleanly to domain areas with correct REST nesting
- `gateReadiness.js` is a pure function — no side effects, takes data, returns result; now has unit test coverage (7 tests, added 2026-06-13) confirming weighted scoring and threshold behavior
- Docker and Vercel deployment configs coexist correctly without modification
- `api/index.js` serverless entry point is clean: `module.exports = require('../src/app')`
