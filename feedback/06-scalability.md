# Scalability Review

---

## Architecture Context

The platform is deployed on Vercel Serverless Functions (Node/Express). Database is Supabase (managed Postgres). This architecture has specific scalability constraints that differ from traditional long-running servers.

**Key constraint:** Each Vercel function invocation can be a fresh process with its own connection pool. Total active DB connections = concurrent Vercel invocations × pool `max`. Supabase free tier supports ~60 connections. Pro tier supports ~200.

---

## Critical Issues

### DB Connection Pool Not Sized for Serverless

**File:** `backend/src/config/database.js`

Default `pg.Pool` `max` is 10. If Vercel runs 10 concurrent function invocations (common under moderate load), that is 100 connections — exceeding Supabase free tier (60) and approaching Pro tier limits. Connection failures surface as 500 errors with no clear message.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `SC1`.

---

### No Pagination on `Deal.list()`

**File:** `backend/src/models/Deal.js` — `list()` method

No `LIMIT` or `OFFSET`. Returns all rows on every call. At 500 deals this causes slow queries. At 5,000 deals this causes memory pressure and timeouts. The API surface has no `?page=` or `?limit=` parameter.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `SC2`.

---

### No Query Timeouts

**Files:** All model and repository files

No `statement_timeout` set on queries. A slow query (table scan, lock wait, index miss) holds the Vercel function slot open for the full 30-second Vercel timeout. Under load, this exhausts the function concurrency budget and causes cascading timeouts.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `SC3`.

---

### `computeGateReadiness` — 10 DB Queries Per Request, No Cache

**File:** `backend/src/services/gateReadiness.js`

Every request to the gate readiness endpoint runs 10 parallel Postgres queries. No caching. For a deal that hasn't changed, this is 10 wasted queries per page view. Under concurrent access to the same deal, this multiplies linearly.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `SC4`.

---

### Rate Limiter Not Distributed

**File:** `backend/src/middleware/rateLimit.js`

In-memory `Map` keyed by IP. Not shared across Vercel invocations. Rate limit state resets on cold start. An attacker can bypass by triggering parallel invocations.

**Fix (security + scalability):** See [08-codex-action-plan.md](08-codex-action-plan.md) item `H3`.

---

## Medium Issues

### Socket.io Incompatible With Serverless

Socket.io requires persistent connections. Vercel serverless functions are stateless and short-lived. Socket.io is currently disabled — the PRD notes it as "future phase." No documented migration path exists.

**For production real-time:** Use Ably, Pusher, or Supabase Realtime (already in stack) as a managed WebSocket service. This eliminates the infrastructure problem entirely.

**Fix:** See [08-codex-action-plan.md](08-codex-action-plan.md) item `SC5` (documentation only — architecture decision).

---

### No Index Strategy Documented

The `database/schema.sql` file exists but index coverage is unknown from the review. At scale, unindexed foreign keys (`deal_id` on risks, actions, review stages) cause full table scans.

**Recommendation:** Verify indexes on: `risks.deal_id`, `actions.deal_id`, `review_stages.deal_id`, `audit_log.user_id`, `audit_log.created_at`. Add a `database/migrations/` entry for any missing indexes.

---

### No Horizontal Scaling Strategy

Single Express app exported as a Vercel function. Works for POC. At team scale (multi-region, high concurrency), the following become constraints:
- In-memory session state (rate limiter — already flagged)
- File system assumptions (none currently, but risky to add)
- Single Supabase project (no read replicas)

**This is not a current problem** — document in `scaling-strategy.md` as a Phase 3 concern.

---

## Scalability Roadmap

| Phase | Action | When |
|---|---|---|
| Now | Fix pool max to 3, add query timeout | Before any load |
| Now | Add pagination to `Deal.list()` | Before >100 deals |
| Now | Cache `computeGateReadiness` (60s TTL) | Before team use |
| Phase 2 | Redis-backed rate limiter (Upstash) | Before public access |
| Phase 2 | Supabase Realtime replacing Socket.io | When real-time required |
| Phase 3 | Read replicas / connection pooler (PgBouncer) | At sustained team scale |
| Phase 3 | Supabase Pro tier (200 connections) | At 5+ concurrent users |

---

## Connection Budget Math

```
Supabase Free Tier: 60 connections max
Vercel max concurrency: ~100 (default, uncapped)
Current pool max: 10

Worst case: 100 invocations × 10 pool max = 1,000 connection attempts → all fail

Recommended pool max: 3
Safe concurrency at pool max 3: 60 ÷ 3 = 20 concurrent invocations
→ Sufficient for POC and early team use
→ Upgrade to Supabase Pro when sustained concurrency exceeds 20
```
