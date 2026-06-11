# Data Model + API Contract Overview

## Data Layer
- Postgres schema with migration-driven evolution.
- Core entities include users, deals, actions, risks, partners, audit, reviews, deal context sub-tables.
- Foreign keys enforce integrity (e.g., user references from deals).
- Assignment identity pattern is now centralized through a backend resolver service to reduce route-level duplication and prepare for future external IdP mapping.
- Program taxonomy is now stored in dedicated `deals` columns: `gwac`, `award_type`, `bureau_sub_agency`, `contract_type`.
- Global suggestion values are managed in `program_field_options` (normalized unique options per field).
- Optional identity-link columns were added for deal role assignments: `bd_lead_user_id`, `proposal_manager_user_id`, `director_user_id` (free-text display fields remain supported).

## API Surface (Key)
- Auth: login, me, register, profile update, password reset flows.
- Deals: list/get/create/update + detail payloads.
- Deals: global program taxonomy suggestions endpoint (`GET /api/deals/program-options`).
- Reviews: per-deal review create/update/list.
- Reviews: per-deal review create/update/list + readiness endpoint with non-blocking guidance policy.
- Deal Context: profile/milestones/incumbent/status endpoints.
- Admin/User: users/roles endpoints with role-based controls.
- Assignment resolution for people fields (e.g., deal leads, action assignees) now uses a shared service (`backend/src/services/assignmentResolver.js`) with explicit outcomes (`matched`, `unmatched`, `blocked`, `empty`); current primary flows permit unmatched and queue admin follow-up.
- Admin follow-up queue endpoint exists for unresolved names: `/api/admin/action-queue` (admin-only list/resolve).

## Contract Quality
- Frontend uses centralized API client + endpoint modules.
- Error handling improved for auth DB outage exposure (sanitized service-unavailable messaging).
- Next phase: explicit API schema docs/versioning and contract test coverage expansion.
- Current prep for Entra integration: assignment resolution is centralized so provider changes occur in one layer rather than across each route.
