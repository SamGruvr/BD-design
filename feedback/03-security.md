# Security Review

---

## Current Posture

### What Is Implemented Correctly

- JWT auth uses `jwt.verify` (not `jwt.decode`) — token tampering is caught
- `bcryptjs` with cost factor 10 for password hashing
- Password reset: SHA-256 token hashed before storage, 30-minute expiry, single-use enforced
- Security headers: 6 of 7 critical headers set (`X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`, `X-Permitted-Cross-Domain-Policies`, `Cross-Origin-Opener-Policy`, HSTS in prod)
- Two-dimensional rate limiting on auth endpoints (per-IP + per-IP+credential)
- Bootstrap admin: `adminCount === 0` guard prevents post-setup privilege escalation
- `.env.example` comprehensive, all secrets use placeholder values
- `.gitignore` correctly excludes `.env` and `.env.local`

---

## Critical Vulnerabilities

### C1 — TLS Certificate Verification Disabled

**Status: Closed 2026-06-11** (commit `a7d33d4`). `DB_SSL_CA` env var (PEM content) + `DB_SSL_REJECT_UNAUTHORIZED=true` now set in production; certificate chain is verified.

**File:** `backend/src/config/database.js`
**Issue (historical):** `ssl: { rejectUnauthorized: false }` disabled certificate verification on the Supabase connection. An attacker on the network path could intercept all database traffic via MITM.

### C4 — Row Level Security Disabled on All 41 Public Tables (found 2026-06-13)

**Status: Closed 2026-06-13.** RLS enabled (default-deny, zero policies) on all 41 tables via the Supabase advisor's remediation SQL, applied directly by Sam through the Supabase SQL editor. Verified post-fix: advisor now reports `rls_enabled_no_policy` (informational) instead of `rls_disabled_in_public` (critical/external) on every table. App health check (`/health`) confirmed `db: ok` immediately after — the Express backend's direct `pg`/`DATABASE_URL` connection is unaffected, as expected, since it never went through PostgREST.
**Where:** Supabase project `srguiseeksrguemxdduo`, `public` schema, all 41 tables including `users` and `password_reset_tokens`.
**Issue (historical):** RLS was disabled on every table exposed to PostgREST. A live, non-disabled legacy `anon` API key existed for this project. Anyone holding that key could call the Supabase auto-generated REST API (`https://srguiseeksrguemxdduo.supabase.co/rest/v1/<table>`) directly and read or write every row in every table — including password reset tokens and the full `users` table — completely bypassing the app's own JWT auth and Express backend.
**Residual note:** No RLS policies exist (by design — default-deny). If any future integration needs legitimate PostgREST/anon-key access to specific tables, explicit policies will need to be added at that time; until then, that path is fully closed.

### C3 — No Content-Security-Policy Header

**Status: Closed.** CSP baseline implemented in `backend/src/middleware/securityHeaders.js`.

**File:** `backend/src/middleware/securityHeaders.js`
**Issue (historical):** CSP was absent. The app stores JWTs in `localStorage`. Without CSP, injected scripts could read `localStorage` and exfiltrate tokens.

---

## High Vulnerabilities

### H3 — Rate Limiter Bypassed in Serverless

**File:** `backend/src/middleware/rateLimit.js`
**Issue:** Rate limit state stored in `Map` in process memory. Vercel runs each function invocation in a separate process. State is not shared. Attacker triggers parallel invocations to bypass all rate limits entirely.
**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `H3`.

### H5 — Bootstrap Admin Promotion: No Audit Trail

**File:** `backend/src/routes/auth.js` — `promoteBootstrapAdminIfMatch()`
**Issue:** When a user is elevated to admin via the bootstrap mechanism, no audit log entry is created. Admin privilege escalation occurs silently with no forensic record.
**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `H5`.

### H2 — Audit Log: Silent Write Failures

**File:** `backend/src/middleware/auditLog.js`
**Issue:** `auditLog()` is fire-and-forget inside the `res.send` override. Errors are caught with `console.error` only — no retry, no alert, no dead-letter. A failed audit write is undetectable.
**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `H2`.

---

## Medium Vulnerabilities

### M6 — Reset Token Leaked in Non-Production Environments

**File:** `backend/src/routes/auth.js`
**Issue:** If `NODE_ENV !== 'production'` and `ENABLE_IN_APP_PASSWORD_RESET_TOKEN=true`, the raw reset token is returned in the API response body. If a staging environment uses `NODE_ENV=development`, tokens are leaked in API responses.
**Fix:** Audit all non-production environments to ensure `NODE_ENV=staging` or `NODE_ENV=test` (not `development`) is set.

### M9 — `localStorage` Token Storage

**File:** `frontend/src/api/client.js`, `frontend/src/context/AuthContext.jsx`
**Issue:** JWTs stored in `localStorage` are accessible to any JavaScript on the page. Standard SPA pattern but elevated risk without CSP. Mitigation is C3 (CSP) rather than moving to `httpOnly` cookies (which requires backend changes).
**Note:** C3 fix is the priority. Cookie-based auth is a Phase 2 consideration.

---

## Security Scanning — Current State: None

| Control | Status | Risk |
|---|---|---|
| SAST (static analysis) | Absent | Logic flaws, injection patterns ship undetected |
| `npm audit` | Present but non-blocking | Known CVEs ship to production |
| Secret scanning | Absent | Accidental key commits go undetected |
| DAST | Absent | Runtime auth bypass/IDOR only caught by pen test |
| Dependency license scan | Absent | GPL contamination risk |
| Dependabot / automated patches | Absent | Dependency drift accumulates |

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) items `S1` through `S5`.

### Required CI Security Controls (Priority Order)

1. `npm audit --audit-level=high` → **blocking** (S1)
2. `gitleaks` secret scanning → **blocking** (S2)
3. CodeQL SAST on every PR → advisory then blocking (S3)
4. Dependabot weekly patch PRs (S4)
5. `actions/dependency-review-action` on PRs (S5)

---

## Patching Policy — Current State: Manual Only

No mechanism exists to surface or apply dependency patches automatically. With `npm audit` non-blocking, CVEs accumulate silently between manual reviews.

**Minimum viable patching policy:**
- Dependabot weekly for `npm` (frontend + backend)
- `npm audit --audit-level=high` blocking in CI
- Monthly review of Snyk or GitHub Security tab
- Supabase Postgres version monitored via Supabase dashboard (managed — but verify major version)

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) items `S4` and `S5`.

---

## Environment Variable Security

| Finding | Status |
|---|---|
| `.env.example` complete and well-documented | Good |
| `.env` and `.env.local` in `.gitignore` | Good |
| `backend/.env.test` committed | Acceptable — test-only credentials, no prod secrets |
| Supabase project ID in public PRD doc | Low risk but unnecessary — remove |
| JWT secret falls back to `undefined` if `STRICT_ENV_VALIDATION=false` | Fix: remove fallback, always throw |

---

## Strengths

- Auth flow is correctly implemented end-to-end
- Password reset is one of the more carefully built flows in the codebase
- Security header middleware is clean and correct (minus CSP)
- Rate limiter design (two-key strategy) is above-average for a POC
- Env validation fails fast on startup with clear error messages
