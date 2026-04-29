# Operations & Deployment Runbook (Concise)

## Environments
- Frontend project: `bd-platform-app` (Vercel)
- Backend project: `bd-platform-api` (Vercel)
- Database: Supabase Postgres

## Critical Config Rules
- Set DB credentials on backend project only.
- Use correct pooler username format and URL-encoded password.
- Redeploy backend after env changes.

## Health/Smoke Checks
1. Backend health endpoint returns 200.
2. Auth login/register endpoints respond correctly.
3. Frontend loads latest deployed bundle and can complete login.

## Verification Commands
- Safe: `bash scripts/verify-all.sh`
- Browser mode: `bash scripts/verify-all.sh --browser`
- Full mode: `bash scripts/verify-all.sh --full`

## Recovery Basics
- If auth/register fail with DB auth messages, validate backend `DATABASE_URL` first.
- If pushes fail with ref anomalies, inspect and clean local remote refs before retry.
