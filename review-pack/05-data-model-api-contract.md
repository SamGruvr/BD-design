# Data Model + API Contract Overview

## Data Layer
- Postgres schema with migration-driven evolution.
- Core entities include users, deals, actions, risks, partners, audit, reviews, deal context sub-tables.
- Foreign keys enforce integrity (e.g., user references from deals).
- Assignment identity pattern is now centralized through a backend resolver service to reduce route-level duplication and prepare for future external IdP mapping.
- Program taxonomy is now stored in dedicated `deals` columns: `gwac`, `award_type`, `bureau_sub_agency`, `contract_type`.
- Global suggestion values are managed in `program_field_options` (normalized unique options per field).
- Optional identity-link columns were added for deal role assignments: `bd_lead_user_id`, `proposal_manager_user_id`, `director_user_id` (free-text display fields remain supported).
- New tables (migration `011_poc_completion_v13.sql`): `deal_step2_checklist` (deal_id, question_key, answer boolean, notes) seeded with 4 canonical Step 2 qualifying questions; `deal_swb` (deal_id, strengths, weaknesses, customer_benefits — one row per deal).
- New columns (same migration): `deal_key_meetings.attendees`; `deal_partners.teaming_rationale`; `deal_voice_of_customer.stakeholder_id` (FK, nullable), `.stakeholder_role`, `.other_notes`, `.company_access`; `deal_competitor_hypotheses.demography` (validated against SB/WOSB/DVSB/ANC/Native American/Large).
- Role taxonomy: `roles.name` and `users.role` values migrated `analyst` → `standard_user` (migration `012_role_rename_standard_user.sql`).

## API Surface (Key)
- Auth: login, me, register, profile update, password reset flows.
- Deals: list/get/create/update + detail payloads.
- Deals: global program taxonomy suggestions endpoint (`GET /api/deals/program-options`).
- Reviews: per-deal review create/update/list.
- Reviews: per-deal review create/update/list + readiness endpoint with non-blocking guidance policy.
- Deal Context: profile/milestones/incumbent/status endpoints, plus `PUT /api/deals/:dealId/context/step2-checklist` and `PUT /api/deals/:dealId/context/swb` (new, upsert-per-deal patterns matching the existing readiness-checklist and customer-knowledge endpoints).
- Deal Context aggregation (`GET /api/deals/:dealId/context`) now also returns `step2Checklist` (merged with question-key defaults) and `swb`.
- Admin/User: users/roles endpoints with role-based controls.
- Assignment resolution for people fields (e.g., deal leads, action assignees) now uses a shared service (`backend/src/services/assignmentResolver.js`) with explicit outcomes (`matched`, `unmatched`, `blocked`, `empty`); current primary flows permit unmatched and queue admin follow-up.
- Admin follow-up queue endpoint exists for unresolved names: `/api/admin/action-queue` (admin-only list/resolve).

## Contract Quality
- Frontend uses centralized API client + endpoint modules.
- Error handling improved for auth DB outage exposure (sanitized service-unavailable messaging).
- `gateReadiness.js` (weighted Step 2/3 scoring) now has unit test coverage (7 tests). Tests confirmed and documented two pre-existing scoring quirks rather than changing behavior: empty sections score as 100% (vacuous), and Step 3 section weights sum to 110, not 100 (35+20+20+15+5+5+10). Neither is fixed yet — flagged for a normalization pass.
- Next phase: explicit API schema docs/versioning and contract test coverage expansion.
- Current prep for Entra integration: assignment resolution is centralized so provider changes occur in one layer rather than across each route.
