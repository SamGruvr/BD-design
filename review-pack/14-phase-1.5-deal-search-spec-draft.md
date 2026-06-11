# Phase 1.5 Deal Search Spec (Draft for Approval)

Status: Draft (not approved for implementation)

## Objective
Enable fast deal discovery and quick-open navigation without introducing hard-coded endpoints or bypassing current auth/role controls.

## Scope
1. Global search input in primary app shell (top bar) with typeahead results.
2. Deals page search enhancement with consistent query behavior.
3. Quick-open action from results to deal detail (`/deals/:id`).

Out of scope:
1. Cross-entity search (partners/actions/risks) in this phase.
2. Saved presets (separate Phase 1.5 item).
3. Semantic/vector search.

## UX Behavior
1. Debounced query (300ms default).
2. Min query length: 2 characters.
3. Result ordering:
   - exact name match first
   - prefix name match
   - agency/owner text match
   - recent updated tie-breaker
4. Keyboard support:
   - up/down to navigate results
   - enter to open
   - escape to close
5. Empty states:
   - no query: show helper text
   - no results: show “No deals found”

## API Contract (Proposed)
`GET /api/deals/search?q=<string>&limit=<int>`

Response:
```json
{
  "success": true,
  "data": [
    {
      "id": 123,
      "deal_name": "Example Deal",
      "agency": "DHS",
      "current_gate": "Step 2",
      "bd_lead_name": "Sam A.",
      "updated_at": "2026-05-02T11:00:00.000Z"
    }
  ]
}
```

Rules:
1. Auth required.
2. Max `limit` 25.
3. Parameterized SQL only.
4. No wildcard-heavy unbounded scans (use indexed query strategy).

## Data + Performance
1. Reuse `deals` and user joins already used in list endpoint.
2. Add/confirm indexes for `deal_name`, `agency`, and updated timestamp path used by ordering.
3. Target p95 API latency under 250ms for normal internal dataset.

## Security
1. Reuse existing auth middleware.
2. Enforce query length and limit caps.
3. Apply request rate limit if needed after first perf pass.
4. Avoid exposing hidden fields in response payload.

## Testing Acceptance Criteria
1. Unit:
   - query validation
   - ordering behavior
   - limit enforcement
2. Integration:
   - auth required
   - expected result shape
   - empty/no-result cases
3. Browser E2E:
   - type query, select result, open deal detail
   - keyboard-only navigation flow
4. Full verify:
   - `verify:full` remains pass after merge

## Rollout
1. Behind a simple feature flag toggle in frontend config (default on for internal preview).
2. Deploy backend first, then frontend.
3. Re-run hosted parity verify after deployment.

## Approval Gate Checklist
1. Scope approved.
2. UX behavior approved.
3. API contract approved.
4. Security constraints approved.
5. Acceptance tests approved.
