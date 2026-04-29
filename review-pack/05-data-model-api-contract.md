# Data Model + API Contract Overview

## Data Layer
- Postgres schema with migration-driven evolution.
- Core entities include users, deals, actions, risks, partners, audit, reviews, deal context sub-tables.
- Foreign keys enforce integrity (e.g., user references from deals).

## API Surface (Key)
- Auth: login, me, register, profile update, password reset flows.
- Deals: list/get/create/update + detail payloads.
- Reviews: per-deal review create/update/list.
- Deal Context: profile/milestones/incumbent/status endpoints.
- Admin/User: users/roles endpoints with role-based controls.

## Contract Quality
- Frontend uses centralized API client + endpoint modules.
- Error handling improved for auth DB outage exposure (sanitized service-unavailable messaging).
- Next phase: explicit API schema docs/versioning and contract test coverage expansion.
