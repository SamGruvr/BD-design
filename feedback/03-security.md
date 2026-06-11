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

**File:** `backend/src/config/database.js`
**Issue:** `ssl: { rejectUnauthorized: false }` disables certificate verification on the Supabase connection. An attacker on the network path can intercept all database traffic via MITM.
**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `C1`.

### C3 — No Content-Security-Policy Header

**File:** `backend/src/middleware/securityHeaders.js`
**Issue:** CSP is absent. The app stores JWTs in `localStorage`. Without CSP, injected scripts can read `localStorage` and exfiltrate tokens. XSS + no CSP + localStorage token storage = full session hijack.
**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `C3`.

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
