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
- Auth routes now include configurable rate limiting for abuse resistance (`/login`, `/register`, `/forgot-password`, `/reset-password`).

## Data/Domain Modules
- Deals + details UI mapping.
- Review steps (Step 2/3) persisted and editable.
- Review readiness now has a backend gate-evaluation endpoint (`/api/deals/:dealId/reviews/readiness?step=2|3`) used for decision guidance in a non-blocking workflow.
- Review readiness uses weighted section scoring with section minimum rules, including expanded Market Evidence checks.
- Capture context subresources (profile/milestones/incumbent/status updates).
- Risks/actions/partners/audit integrated.
- Dedicated workflow pages now live for:
  - Capture Content (`/capture-content`) with deck-ordered section-readiness summary.
  - Stakeholders (`/stakeholders`) with editable stakeholder `profile` capture.
  - Reports (`/reports`) with cross-deal decision-readiness baseline.

## Deployment/Config
- Backend requires correct `DATABASE_URL` (pooler user, encoded password, SSL mode).
- Frontend API and socket config are env-driven (localhost fallbacks removed from production pathing).
- API contract baseline now exposes explicit version metadata (`X-API-Version` header + `/api/meta` endpoint) for controlled evolution.
- Core write endpoints now enforce lightweight request contract validation (`deals`, `partners`, `users`) via centralized middleware.

## Architecture Notes
- Current stack is intentionally simple and scalable enough for near-term growth.
- Phase 2 started: partner domain now uses a repository layer to separate route orchestration from persistence logic.
- User admin domain now also uses repository-backed data access for cleaner route/service boundaries.
- Deal detail aggregation/UI-data persistence now also runs through a repository layer for cleaner maintainability.
- API versioning strategy has started with lightweight contract signaling; next step is schema-driven contract validation per route group.
- API versioning strategy is now paired with request contract checks on core routes; next step is broader schema coverage + response contract checks.
- Unmatched free-text assignments are now captured in an admin action queue (`/api/admin/action-queue`) so non-registered names can still be used while preserving governance follow-up.
- Admin action queue now supports filtering (`status`, `source`, `dealId`) and resolution notes to document remediation actions.
- Runtime schema mutation has been removed from request paths; schema evolution is migration-driven.
- Assignment resolution for people fields is now centralized in a service layer (`backend/src/services/assignmentResolver.js`) and reused by deal/action routes.
- Program Details data model now includes dedicated schema columns on `deals`: `gwac`, `award_type`, `bureau_sub_agency`, `contract_type`.
- Global reusable options for those fields are stored in `program_field_options` and exposed via `GET /api/deals/program-options`.
- Gate-review architecture direction clarified:
  - Step review artifacts should be treated as governance-grade decision gates (`go`, `conditional go`, `hold/recycle`, `no-go`) rather than presentation-only decks.
  - Decision evidence should remain structured and traceable to deck sections, with explicit readiness scoring and audit trail.
- Next structural step for enterprise scale: extend repository/data-access pattern across deals/users, then formalize ORM and stricter contract/version governance.
