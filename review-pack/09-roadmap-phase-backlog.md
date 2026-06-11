# Roadmap + Next-Phase Backlog

## 0-2 Weeks (Stabilization)
1. Execute full browser E2E with stable test credentials. (Completed: 2026-05-01 `verify:full` pass)
2. Fix any defects from artifact-driven triage. (Completed: 2026-05-01 hosted context API drift fix + re-verify pass)
3. Finalize internal preview checklist and baseline tag. (Completed: 2026-05-01 baseline tag + checklist update)
4. Formalize Step 2/3 gate outcomes and evidence scoring model. (Completed: 2026-05-01 weighted scoring with section minimums + Market Evidence Option 2)

## 2-6 Weeks (Usability + Hardening)
1. UX polish sprint on top workflows.
2. Expand validation and save-flow guidance for key forms.
3. Add quality reporting cadence for trend tracking.
4. Remove runtime schema auto-create/alter patterns from request paths and move fully to migration-only DB lifecycle. (Completed)

## Phase 1.5 (Workflow Acceleration)
Definition gate for all items in this phase: each item must have approved scope, UX behavior, API/data contract, security considerations, and test acceptance criteria documented before implementation starts.
Deal search draft spec created for review: `14-phase-1.5-deal-search-spec-draft.md`.
1. Add deal search capability (global + deals list) with fast name/agency/owner matching.
2. Add saved filter presets for repeat review workflows.
3. Add quick-open from search results to deal detail/review context.

## 6-12 Weeks (Enterprise Readiness)
1. Introduce ORM layer and stricter data-contract governance.
2. Formalize API schema/version strategy.
3. Strengthen security/compliance operations and release controls.

## Priority Queue (Current)
- P1: Deal search capability (first Phase 1.5 backlog item)
- P1: UX consistency/usability improvements
- P1: Step 1 executive decision board refinement: align Opportunity Details to deck sections (Opportunity Information + Schedule and Key Dates), keep summary cards, include details-page anchor links, and add metric explainers (for example Win Probability calc method). Mockups: `docs/mockups/executive-gate-decision-mockup.html`, `docs/mockups/step-1-details-mockup.html`.
- P1: Deal Detail information architecture update: second-level left section navigation + Capture Context review-first redesign (decision pending from mockup review).
- P1: Program Details taxonomy UX refinement (field guidance, validation hints, and reporting visibility for schema-backed fields)
- P2: ORM + architecture formalization
- P2: Security/remediation operationalization

## Added From Internal Review (2026-04-30)
- P1: Partner legal status taxonomy update to `NDA Signed`, `No NDA`, `NDA Pending` across create/edit/reporting flows.
- P1: Add conditional Partner `TA status` field when partner is selected with options: `No TA`, `TA Pending`, `TA Signed`.
- P1: Add Stakeholder Profile field `profile` (structured summary of stakeholder background/context).
- P1: Add Partner Profile field `summary & details` for richer partner qualification notes.
- P2: Conduct targeted UX/content review of Capture Context tab (information architecture, ordering clarity, and guidance copy).

## Implemented Since Last Checkpoint
- Capture Content moved from placeholder to live deck-ordered workflow page with section readiness.
- Stakeholders moved from placeholder to live workflow page with editable `profile`.
- Reports moved from placeholder to live cross-deal decision-readiness baseline.
- Auth rate limiting added for login/register/password-reset endpoints.
- Assignment resolver service added and wired into deal/action assignment paths.
- API contract drift reduced by removing unsupported frontend delete endpoint references.
- Gate readiness backend service + endpoint implemented; readiness is now advisory guidance for an always-editable review process.
- Gate readiness enhanced with weighted section scoring + minimum section evidence checks.
- Market Evidence scoring upgraded to Option 2 (6 checks + 60% section minimum).
- Review decision vocabulary aligned to governance outcomes: `go`, `conditional_go`, `hold_recycle`, `no_go`.
- Admin action queue added for unresolved assignments so free-text names are allowed without blocking workflow, with admin follow-up path.
- Admin action queue hardened with filters (`status`, `source`, `dealId`) and resolution notes for remediation tracking.
- Program Details taxonomy now uses dedicated schema columns (`gwac`, `award_type`, `bureau_sub_agency`, `contract_type`) plus global suggestion options.
- Deal role assignments now store optional identity links in dedicated columns (`bd_lead_user_id`, `proposal_manager_user_id`, `director_user_id`) while preserving free-text workflow behavior.
