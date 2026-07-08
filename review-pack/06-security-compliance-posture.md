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
- TLS certificate verification to Supabase (`DB_SSL_CA` + `DB_SSL_REJECT_UNAUTHORIZED=true`).
- Row Level Security enabled (default-deny) on all 41 Supabase tables, closing direct PostgREST/anon-key access to the database.
- Rate limiting is now serverless-safe: rewritten from an in-memory `Map` to a shared Postgres counter table (`rate_limit_counters`), so counts persist across concurrent Vercel function instances instead of resetting per-instance. See `feedback/03-security.md` item `H3`.

## Secure Operations Notes
- Backend env correctness is critical for auth/register reliability.
- Secrets should remain backend-only; no service credentials in frontend.
- Password reset and user lifecycle functions exist with admin restrictions.

## Residual Risks
- Ongoing dependency vulnerability monitoring/remediation needed.
- Security audit data may be unavailable in constrained network environments; reporting now explicitly flags when audit data could not be fetched.
- Need steady cadence for security test scenarios and config drift checks.
- ~~Row Level Security disabled on all 41 public tables~~ — **Closed 2026-06-13.** RLS enabled (default-deny) on all tables; verified via advisor re-scan and a live `/health` check post-fix. See `feedback/03-security.md` item `C4`.

## Decision Log: Rate Limiter Store (2026-07-07)
Considered Upstash Redis (the standard recommendation for serverless rate limiting — REST-based client, no persistent TCP connections, atomic INCR) vs. Postgres against the existing Supabase connection. Chose Postgres: no new vendor/account/credentials to provision, rotate, or monitor; the app already has a TLS-verified connection; and at auth-endpoint request volumes (not page-view scale), Postgres's higher per-check latency (~10-50ms vs Redis's ~1-5ms) and row-lock contention under concurrent hits are not expected to matter. Revisit if login/register/reset traffic ever grows enough that this becomes a measurable bottleneck.

## Next Security Enhancements
1. Add dependency audit gate output into regular status reporting and enforce remediation SLO.
2. Add documented remediation SLA/severity policy.
3. Expand auth abuse/rate-limiting hardening if user volume increases.
4. If any future integration needs PostgREST/anon-key access, add explicit RLS policies scoped to that need rather than reopening default access.
