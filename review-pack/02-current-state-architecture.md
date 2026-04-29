# Current-State Architecture

## System Topology
- Frontend: React + Vite SPA.
- Backend: Node.js + Express REST API.
- Database: Postgres (Supabase-hosted).
- Deployment: Vercel (separate frontend and backend projects).

## Runtime Flow
1. User authenticates via `/api/auth/login`.
2. Backend signs JWT with role claim.
3. Frontend stores token and enforces route-level auth.
4. Sidebar/admin routes render based on role (`admin` required for admin console/user admin).
5. API calls include bearer token and are authorized server-side.

## Auth/Admin Controls
- Admin gating in frontend navigation and pages.
- Bootstrap admin email parsing supported via backend env variables.
- User admin supports create/edit/deactivate/delete/reset-password.

## Data/Domain Modules
- Deals + details UI mapping.
- Review steps (Step 2/3) persisted and editable.
- Capture context subresources (profile/milestones/incumbent/status updates).
- Risks/actions/partners/audit integrated.

## Deployment/Config
- Backend requires correct `DATABASE_URL` (pooler user, encoded password, SSL mode).
- Frontend API and socket config are env-driven (localhost fallbacks removed from production pathing).

## Architecture Notes
- Current stack is intentionally simple and scalable enough for near-term growth.
- Next structural step for enterprise scale: introduce formal ORM layer and stricter contract/version governance.
