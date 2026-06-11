# Quality & Test Evidence

## Current Signal
- Backend unit suite passing (`101/101`) in verify path.
- Frontend production build repeatedly passing.
- Unified verification script available with structured summary JSON.
- Full verify now runs browser/API diagnostics + build + unit tests by default; DB integration tests are opt-in (`RUN_DB_INTEGRATION_TESTS=true`).
- Review workflow status is validated as non-blocking (`In Progress` / `Shared`) with readiness guidance preserved.
- Runtime baseline standardized to Node `20.x` in CI and local shell baseline (`.nvmrc`).

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
- Integration DB test lane should be enabled in CI with explicit DB SSL config and secrets.
