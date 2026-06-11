# Internal Preview Checklist

Date: 2026-05-01
Baseline Commit: cce3dd7
Baseline Tag: internal-preview-2026-05-01-r2

## Verification
- [x] Frontend production build passes.
- [x] Backend unit/integration test suite passes.
- [x] Browser verification flow passes (`verify:browser`).
- [x] Full hosted verification flow passes (`verify:full`) including diagnostic + browser path.

## Workflow Readiness
- [x] Auth and role-gated navigation working.
- [x] User admin and role admin operational.
- [x] Deals list filtering/sorting operational.
- [x] Deal detail review flow operational.
- [x] Capture Context tab deck-ordered workflow and quick-nav implemented.
- [x] Actions edit + complete path stable.
- [x] Risks edit/details path stable.
- [x] Partners legal taxonomy + TA status + summary/details captured.
- [x] Stakeholder `profile` field captured.
- [x] Assignment resolution centralized via shared service for key deal/action flows.
- [x] Unmatched assignment handling no longer blocks workflow; unresolved names appear in Admin Action Queue.
- [x] Admin Action Queue supports `status/source/dealId` filtering and resolution notes.
- [x] Weighted scoring policy finalized with Market Evidence Option 2 checks.

## UX Readability
- [x] Dark-mode contrast pass completed.
- [x] Light-mode option with off-white base completed.
- [x] Capture Context section sequencing clarified.

## Known Constraints
- Hosted preview still depends on stable demo credentials and environment consistency over time.
- ORM/data-contract formalization remains planned for enterprise-readiness phase.
- Dependency audit results depend on npm audit endpoint availability from runtime environment.
