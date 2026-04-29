# Quality & Test Evidence

## Current Signal
- Backend unit/integration suite repeatedly passing (`100/100`).
- Frontend production build repeatedly passing.
- Unified verification script available with structured summary JSON.

## Diagnostic Tooling
- API diagnostic runner: connectivity/auth/deals/context checks + fault mapping.
- Browser diagnostic runner: login -> deals -> deal detail -> capture context checks + artifacts.
- Verify summary includes step statuses, durations, runtime context, and report paths.

## Artifact Locations
- `test-reports/verify-summary.json`
- `test-reports/e2e-diagnostic-*.md|.json`
- `test-reports/e2e-browser-diagnostic-*.md|.json`
- `test-reports/browser-e2e-*/` (trace/screenshots on failures)

## Quality Gaps
- Full browser E2E pass in non-dry-run should be part of regular pre-demo gate.
