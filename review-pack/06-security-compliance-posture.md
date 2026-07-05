# Security & Compliance Posture

## Implemented Controls
- Role-based access control for admin-sensitive actions.
- Auth token validation and guarded routes.
- Configurable auth rate limiting on high-risk auth endpoints.
- Security headers middleware (XFO, XCTO, COOP, referrer policy, etc.).
- Security headers expanded with Content-Security-Policy baseline.
- Audit log functionality for critical actions.
- Environment validation and reduced hardcoded endpoint behavior.
- CI security workflows added: CodeQL, dependency review, and secret scanning (gitleaks).

## Secure Operations Notes
- Backend env correctness is critical for auth/register reliability.
- Secrets should remain backend-only; no service credentials in frontend.
- Password reset and user lifecycle functions exist with admin restrictions.

## Residual Risks
- Ongoing dependency vulnerability monitoring/remediation needed.
- Security audit data may be unavailable in constrained network environments; reporting now explicitly flags when audit data could not be fetched.
- Need steady cadence for security test scenarios and config drift checks.
- **Critical, open (found 2026-06-13): Row Level Security is disabled on all 41 public tables in Supabase project `srguiseeksrguemxdduo`**, and a live, non-disabled legacy `anon` API key exists. This means Supabase's auto-generated REST API (PostgREST) can be used by anyone holding that key to read/write every row in every table — including `users` and `password_reset_tokens` — bypassing the app's own JWT auth entirely. The Express backend uses a direct `pg`/`DATABASE_URL` connection and does not rely on PostgREST, so enabling RLS with no policies (default-deny) closes this with no expected impact to the running app. Not yet remediated — flagged for immediate review. See `feedback/03-security.md` item `C4`.

## Next Security Enhancements
1. Enable RLS (default-deny) on all Supabase tables to close the PostgREST exposure above — highest priority, zero known app dependency on that path.
2. Add dependency audit gate output into regular status reporting and enforce remediation SLO.
3. Add documented remediation SLA/severity policy.
4. Expand auth abuse/rate-limiting hardening if user volume increases.
