# Testing Review

> Reconciliation note: `verify:full` now passes in current build. This file's original gaps remain relevant for depth/coverage quality, but no longer imply complete absence of hosted browser/API validation.

---

## Current Coverage Summary

### Backend

| Test File | Type | Quality |
|---|---|---|
| `tests/unit/api.test.js` | Unit | Weak — verifies endpoint path strings and field array lengths, no HTTP calls |
| `tests/unit/auth.route.test.js` | Unit | Moderate — mocks User model, exercises login handler directly |
| `tests/unit/dealReviews.route.test.js` | Unit | Unknown depth |
| `tests/unit/endpoints.test.js` | Unit | Structural — verifies route registration, not behavior |
| `tests/unit/middleware.test.js` | Unit | Moderate — tests middleware functions in isolation |
| `tests/unit/securityHeaders.test.js` | Unit | Good — verifies each header is set correctly |
| `tests/unit/validation.test.js` | Unit | Good — tests `contractValidation.js` rules |
| `tests/integration/setup.test.js` | Integration | **Broken** — see H6 below |

### Frontend

| Test File | Type | Quality |
|---|---|---|
| `ActionItem.test.jsx` | Component | Present |
| `DealCard.test.jsx` | Component | Present |
| `RiskCard.test.jsx` | Component | Present |
| `api.test.js` | Unit | Present |

**No tests exist for:** `AuthContext`, `useApi`, `useSocket`, any page-level flow, auth flow end-to-end, `gateReadiness.js` service.

---

## Critical Issues

### H6 — Integration Test Passes on Database Failure

**File:** `backend/tests/integration/setup.test.js`

```js
// Current (broken):
try {
  await connectToDatabase();
  expect(connected).toBe(true);
} catch (error) {
  expect(error).toBeDefined(); // passes on ANY error — test is meaningless
}
```

The test passes whether the database is available or not. CI shows green integration tests with zero integration confidence.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `H6`.

---

## High Issues

### M4 — `gateReadiness.js` Has Zero Test Coverage

`computeGateReadiness()` is the most complex domain logic in the codebase:
- Weighted scoring across 5 sections (weights: 35/20/20/15/10)
- Minimum-percent thresholds per section (70/80)
- Recommendation tiers based on score bands
- 10 parallel database queries

Pure function design makes it trivially testable. Zero tests exist. A scoring bug silently produces wrong gate readiness scores, which is the core business value of the platform.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `M4`.

---

## Medium Issues

### M5 — `auth.route.test.js` Accesses Express Internal Stack

```js
// Fragile — breaks if any middleware added to /login route
const loginHandler = authRoutes.stack
  .find(layer => layer.route && layer.route.path === '/login')
  .route.stack[0].handle;
```

Adding rate-limit middleware to the `/login` route changes `stack[0]` from the handler to the middleware. Test silently breaks and tests wrong function.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `M5`.

---

## CI/CD Gate

**Workflow:** `.github/workflows/quality-gate.yml`

| Step | Status | Issue |
|---|---|---|
| Checkout | Pass | — |
| Node 20 setup + cache | Pass | — |
| Install deps | Pass | — |
| `verify-all.sh --safe` | Pass | Runs dry-run, not live |
| Frontend build | Pass | — |
| Backend unit tests | Pass | Tests pass but coverage is shallow |
| `npm audit` | Non-blocking | CVEs don't fail the build |
| ESLint | **Absent** | Linting never runs in CI |
| Coverage threshold | **Absent** | No minimum coverage enforced |
| Branch protection | Unknown | No required status checks visible |

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) items `M7` and `S1`.

---

## Missing Test Coverage (Priority Order)

1. `gateReadiness.js` — weighted scoring, threshold logic, recommendation tiers
2. `Risk` and `Action` model update allowlisting
3. Auth flow end-to-end (login → token → protected route → 401 on expiry)
4. `AuthContext` — login, logout, token persistence
5. Rate limiter — verify limits are enforced, verify reset behavior
6. Audit log — verify entries are created on CREATE/UPDATE/DELETE
7. Bootstrap admin — verify promotion only happens when `adminCount === 0`
8. Pagination — verify `Deal.list()` respects limit/offset parameters

---

## Recommended Testing Strategy

### Backend
- **Unit tests:** Pure functions (gateReadiness, contractValidation, sanitizeUsername)
- **Route tests:** Use `supertest` to make real HTTP requests through the app layer — not handler extraction
- **Integration tests:** Real test database (`backend/.env.test` exists), run migrations before suite, tear down after
- **Coverage threshold:** Set Jest `coverageThreshold` to 70% lines for `services/` and `models/`

### Frontend
- **Component tests:** Already started — continue with React Testing Library
- **Context tests:** `AuthContext` — mock API responses, verify state transitions
- **Page tests:** At minimum: Login page, Deal list page, Deal detail page

### CI Additions
- Add `--coverage` flag to Jest run in CI
- Add `coverageThreshold` in `jest.config.js`: `{ global: { lines: 70 } }`
- Report coverage to GitHub PR as a comment

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) items `T1` through `T4`.
