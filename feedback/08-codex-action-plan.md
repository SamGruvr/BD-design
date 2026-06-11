# Codex Action Plan

All items are paste-ready Codex prompts. Work top to bottom — Phase 1 items unblock Phase 2.
Mark each item `[x]` when complete.

**Source repos:**
- Code: `SamGruvr/BD_ITA` (`backend/`, `frontend/`, `database/`, `.github/`)
- Docs: `SamGruvr/BD-design` (`design-docs/`, `review-pack/`)

---

## Phase 1 — Critical (Do First)

### [ ] C1 — Fix TLS Certificate Verification

> In `backend/src/config/database.js`, find the SSL configuration block that sets `rejectUnauthorized: false`. Replace it with `rejectUnauthorized: true`. If the Supabase connection requires a CA certificate, add the CA bundle path via `ca: fs.readFileSync(process.env.SUPABASE_CA_CERT_PATH)` and add `SUPABASE_CA_CERT_PATH` to `backend/.env.example` with a placeholder. Run existing backend tests to confirm the connection still works.

---

### [ ] C2 — Replace DDL-on-Demand with Startup Migration

> In `backend/src/`, remove the following functions entirely: `User.ensureProfileColumns()` (in `models/User.js`), `Deal.ensureBidPhaseColumn()` (in `models/Deal.js`), `AdminActionQueue.ensureTable()` (in `models/AdminActionQueue.js`), `gateReadiness.ensureReadinessSchema()` (in `services/gateReadiness.js`), `DealRepository.ensureDealUiDataTable()` (in `repositories/DealRepository.js`). Move all their DDL (`CREATE TABLE IF NOT EXISTS`, `ALTER TABLE ... ADD COLUMN IF NOT EXISTS`) into a new migration file at `database/migrations/YYYYMMDD_consolidate_schema.sql`. Install `node-pg-migrate` as a backend dev dependency. Add a `migrate` script to `backend/package.json` that runs all pending migrations. In `backend/src/app.js`, add a startup hook that runs `node-pg-migrate up` before the server begins accepting connections. Do not change any other logic. Run existing tests after.

---

### [ ] C3 — Add Content-Security-Policy Header

> In `backend/src/middleware/securityHeaders.js`, add a `Content-Security-Policy` response header with the following policy: `default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; connect-src 'self' wss:; img-src 'self' data:; frame-ancestors 'none'; base-uri 'self'; form-action 'self'`. Apply it in the same location as the other security headers. Do not modify any other headers or files. Run `backend/tests/unit/securityHeaders.test.js` and add a test case for the CSP header.

---

## Phase 2 — High Priority

### [ ] H1 — Add Contract Validation to Risk and Action Routes

> In `backend/src/routes/risks.js`, apply the `validateBody` middleware (imported from `../middleware/contractValidation`) to the PUT/PATCH route handler for updating a risk. Define allowed fields for risk updates: `risk_title`, `risk_description`, `likelihood`, `impact`, `mitigation_plan`, `status`, `owner`. In `backend/src/routes/actions.js`, do the same for action updates with fields: `action_title`, `action_description`, `due_date`, `status`, `owner`, `priority`. Use the same contract pattern already used in `deals.js`. Do not change model logic.

---

### [ ] H2 — Make Audit Log Awaited and Error-Safe

> In `backend/src/middleware/auditLog.js`, find the `auditLog()` call inside the `res.send` override. Make it `await`ed (convert the surrounding function to async if needed). Wrap the call in a `try/catch` block. In the catch block, log a structured error object: `{ event: 'audit_log_failure', error: err.message, path: req.path, method: req.method, userId: req.user?.id }` using `console.error`. Do not throw in the catch block — audit failure must not break the response. Run existing tests after.

---

### [ ] H3 — Replace In-Memory Rate Limiter with Redis-Backed

> Install `rate-limiter-flexible` and `ioredis` as backend production dependencies. Install `@upstash/redis` if using Upstash (serverless-compatible Redis). Rewrite `backend/src/middleware/rateLimit.js` to use `RateLimiterRedis` from `rate-limiter-flexible` with a Redis client pointed at `process.env.UPSTASH_REDIS_URL` using `process.env.UPSTASH_REDIS_TOKEN`. Preserve the existing two-key strategy: one limiter keyed by IP, one keyed by IP+credential. Preserve existing thresholds (make them configurable via env vars). Add `UPSTASH_REDIS_URL` and `UPSTASH_REDIS_TOKEN` to `backend/.env.example` with placeholder values and comments. Add a fallback: if Redis is unavailable (connection error on startup), log a warning and fall back to in-memory — do not crash the server. Update `backend/.env.test` with a local Redis URL if testing locally.

---

### [ ] H4 — Deduplicate `sanitizeUsername`

> Create `backend/src/utils/sanitize.js`. Move the `sanitizeUsername` function from `backend/src/routes/auth.js` into it and export it as a named export. Update `backend/src/routes/auth.js` to import `sanitizeUsername` from `../utils/sanitize`. Update `backend/src/routes/users.js` to do the same and remove its local definition. Do not change the function logic. Run existing tests after.

---

### [ ] H5 — Audit Log Bootstrap Admin Promotion

> In `backend/src/routes/auth.js`, in the `promoteBootstrapAdminIfMatch()` function, immediately after the database `UPDATE` that sets the user's role to `admin`, call `auditLog()` with: `{ action: 'ADMIN_BOOTSTRAP_PROMOTION', resourceType: 'user', resourceId: user.id, userId: user.id, metadata: { email: user.email, promotedAt: new Date().toISOString() } }`. Use the existing audit log import already present in the file. Do not change any other logic.

---

### [ ] H6 — Fix Non-Functional Integration Test

> In `backend/tests/integration/setup.test.js`, replace the entire test body with: (1) attempt to connect to the test database using the existing DB client, (2) if the connection fails, throw the error — the test must fail if the DB is unavailable, (3) if the connection succeeds, run `SELECT 1 as result` and assert the returned value equals `1`, (4) close the connection in `afterAll`. Remove the `catch (error) { expect(error).toBeDefined() }` pattern entirely. Update `backend/.env.test` if the test DB URL needs correction.

---

## Phase 3 — Scalability

### [ ] SC1 — Fix DB Connection Pool for Serverless

> In `backend/src/config/database.js`, set the connection pool `max` to `3`. Add `statement_timeout: 10000` (10,000ms) and `idle_in_transaction_session_timeout: 10000` to the pool configuration object. Add a comment above these settings explaining: "Vercel runs N concurrent function invocations each with their own pool. Total connections = N × pool_max. Supabase free tier = 60 connections. Pool max 3 = safe for up to 20 concurrent invocations. Increase pool_max only when upgrading to Supabase Pro (200 connections)." Do not change any other database config.

---

### [ ] SC2 — Add Pagination to `Deal.list()`

> In `backend/src/models/Deal.js`, update the `list()` method to accept `{ offset = 0, limit = 50 }` parameters. Add `LIMIT $n OFFSET $m` to the query. Cap `limit` at 100 server-side regardless of input. In `backend/src/routes/deals.js`, update the GET `/api/deals` handler to extract `?page=` and `?limit=` query parameters. Compute `offset = (page - 1) * limit`. Pass `{ offset, limit }` to `Deal.list()`. Return a response envelope: `{ data: deals, pagination: { page, limit, total } }` where `total` comes from `Deal.count()`. Do not change any other model methods.

---

### [ ] SC3 — Add Global Query Timeout Middleware

> Create `backend/src/middleware/queryTimeout.js`. The middleware should, on each request, acquire a DB client from the pool and run `SET LOCAL statement_timeout = '8000'` before releasing it. Alternatively, add `options: { statement_timeout: 8000 }` to the pool query defaults if `pg` supports it at pool level. Apply this middleware globally in `backend/src/app.js` after the DB pool is initialized, before routes. This ensures no query holds a function slot open longer than 8 seconds. Do not change any route or model files.

---

### [ ] SC4 — Cache `computeGateReadiness` Results

> In `backend/src/services/gateReadiness.js`, add an in-memory cache above the `computeGateReadiness` function: a `Map` keyed by `dealId` storing `{ result, expiresAt }`. At the top of `computeGateReadiness`, check if a non-expired cache entry exists for the `dealId` — if so, return it immediately. Cache TTL: 60,000ms (60 seconds). Add a `invalidateGateReadinessCache(dealId)` exported function that deletes the cache entry for a given deal. Call `invalidateGateReadinessCache(dealId)` from any route that writes to review stages (in `backend/src/routes/`). No external dependency — use the `Map` only.

---

### [ ] SC5 — Document Scaling Strategy (BD-design repo)

> In the `SamGruvr/BD-design` repo, create `docs/scaling-strategy.md`. Include: (1) current serverless architecture limits and the DB connection budget math (Vercel concurrency × pool_max vs Supabase tier limits), (2) the recommended pool_max values per Supabase tier, (3) Socket.io migration path — recommend Supabase Realtime (already in stack) or Ably as the production real-time solution; explain why Socket.io is incompatible with serverless, (4) horizontal scaling thresholds — when to upgrade to Supabase Pro, when to add a PgBouncer connection pooler, (5) Phase 3 considerations: read replicas, multi-region, CDN for static assets.

---

## Phase 4 — CI Security Hardening

### [ ] S1 — Make `npm audit` Blocking at High Severity

> In `.github/workflows/quality-gate.yml`, find the existing `npm audit` step (or add one if absent). Change it to: `cd backend && npm audit --audit-level=high` and `cd frontend && npm audit --audit-level=high`. Ensure both commands are in the same CI step or separate steps that both fail the workflow on non-zero exit code. Remove any `|| true` or `continue-on-error: true` flags. Run audit locally first and fix any existing high/critical CVEs before adding the blocking step.

---

### [ ] S2 — Add Secret Scanning to CI

> Add a new step to `.github/workflows/quality-gate.yml` that runs `gitleaks/gitleaks-action@v2` using the action's default configuration. Place this step immediately after checkout, before any install steps. The step must fail the workflow if any secret is detected. Add a `.gitleaks.toml` config file at the repo root to allowlist any intentional test credentials in `backend/.env.test` (use the `allowlist` → `regexes` pattern). Do not change any source files.

---

### [ ] S3 — Add CodeQL SAST

> Create `.github/workflows/codeql.yml`. Configure it to run on `pull_request` targeting `main` and on `push` to `main`. Use `github/codeql-action/init@v3` with `languages: ['javascript']` and the `security-extended` query suite. Run `github/codeql-action/analyze@v3` after. The workflow should run on the default GitHub-hosted runner. Do not modify `quality-gate.yml`.

---

### [ ] S4 — Add Dependabot

> Create `.github/dependabot.yml` at the repo root. Configure two `npm` package ecosystems: one for `frontend/` directory, one for `backend/` directory. Set `schedule: interval: weekly` for both. Set `open-pull-requests-limit: 5`. Group patch-level updates together (label: `dependencies`). Group minor-level updates separately (label: `dependencies-minor`). Target branch: `main`.

---

### [ ] S5 — Add Dependency Review on PRs

> Create `.github/workflows/dependency-review.yml`. Configure it to run on `pull_request` targeting `main`. Use `actions/dependency-review-action@v4` with `fail-on-severity: high`. This blocks merging any PR that introduces a new dependency with a known high or critical CVE. Do not modify other workflow files.

---

## Phase 5 — Code Quality

### [ ] M1 — Fix `directory_lead_id` Typo

> Write a database migration at `database/migrations/YYYYMMDD_rename_directory_lead_id.sql` that renames column `directory_lead_id` to `director_lead_id` on the `deals` table: `ALTER TABLE deals RENAME COLUMN directory_lead_id TO director_lead_id;`. Then do a global find-and-replace of `directory_lead_id` → `director_lead_id` in: `backend/src/models/Deal.js`, `backend/src/routes/deals.js`, `backend/src/repositories/DealRepository.js`. Do not change any other files. Run existing tests after.

---

### [ ] M3 — Clean Repo Root

> Delete these files from the `BD_ITA` repo root: `README copy.md`, `TEST_RESULTS.txt`, `CHECKLIST.md`, `INDEX.md`, `SUMMARY.md`. Move `BD_Platform_PRD_v1.0.docx` and `BD_Platform_Wireframe_v1.0.html` into a new `docs/` directory at the repo root. Add the following entries to `.gitignore`: `TEST_RESULTS.txt`, `*.tmp`, `CHECKLIST.md`. Remove `frontend/dist/` from version control: delete the directory and add `frontend/dist/` to `.gitignore`. Do not change any source files.

---

### [ ] M4 — Write Tests for `gateReadiness.js`

> Create `backend/tests/unit/gateReadiness.test.js`. Write unit tests for `computeGateReadiness()` covering these scenarios: (1) all input data empty → overall score is 0, recommendation tier is lowest, (2) all sections fully complete → overall score is 100, recommendation tier is highest, (3) one section score below minimum threshold (e.g., section with 70% minimum receives 60%) → that section is flagged, overall score is penalized, (4) mixed section completeness → verify the weighted average is computed correctly using the documented weights (35/20/20/15/10), (5) score at exactly each recommendation tier boundary → verify tier assignment is correct. Mock the database calls — `computeGateReadiness` should accept pre-fetched data as input, not make DB calls directly. If the current function signature makes DB calls internally, refactor it to accept data as parameters first, then add tests.

---

### [ ] M5 — Fix Fragile Route Handler Extraction in Tests

> In `backend/tests/unit/auth.route.test.js`, replace the Express internal stack traversal (`authRoutes.stack.find(...)`.route.stack[0].handle`) with `supertest`. Import `supertest` and the Express `app` (or create a minimal test app that mounts the auth router). Make actual HTTP POST requests to `/api/auth/login` in the test. Mock the `User` model at the module level. This approach is not fragile to middleware additions. Rewrite all existing test cases in this file to use `supertest` instead of direct handler invocation.

---

### [ ] M7 — Add ESLint to CI

> Install `eslint-plugin-security` as a dev dependency in `backend/`. Add it to `backend/.eslintrc.js` (create if absent): extend `plugin:security/recommended`. In `.github/workflows/quality-gate.yml`, add two steps: (1) `cd frontend && npm run lint` — fail on lint error, (2) `cd backend && npx eslint src/ --ext .js` — fail on lint error. Fix any pre-existing lint errors in `backend/src/` before adding the CI step. Do not change frontend source files unless ESLint auto-fix is safe.

---

### [ ] M8 — Split `auth.js` Route File

> In `backend/src/routes/auth.js`, extract all helper functions (`sanitizeUsername` — already done in H4, `mapAuthUser`, `isDatabaseUnavailableError`, `promoteBootstrapAdminIfMatch`, password reset helpers) into `backend/src/services/authService.js`. Export each function. Update `auth.js` to import them from `../services/authService`. The route file should contain only route definitions and handlers — no business logic inline. Do not change any handler behavior. Run existing tests after.

---

### [ ] M10 — Document `gateReadiness.js` Scoring Constants

> In `backend/src/services/gateReadiness.js`, add inline comments above the scoring weight constants and threshold values explaining their business origin. Example: `// Section weights determined by BD leadership (April 2026): deal strategy is weighted highest at 35% because ...`. Add a comment above each recommendation tier boundary explaining what it means operationally. Do not change any logic — comments only.

---

## Phase 6 — Documentation (BD-design repo)

### [ ] D1 — Generate OpenAPI Specification

> Read all route files in `backend/src/routes/` of `BD_ITA`. Generate a complete OpenAPI 3.0 YAML specification saved as `docs/openapi.yaml` in the `BD-design` repo. Cover all endpoints: path, HTTP method, authentication requirement (Bearer JWT), request body schema (use `contractValidation.js` field definitions as source of truth), response schema for 200 success and standard error codes (400, 401, 403, 404, 500). Include a `components/schemas` section for reusable types (Deal, Risk, Action, User, Partner). Include a `components/securitySchemes` section defining Bearer JWT auth.

---

### [ ] D2 — Write Architecture Decision Records

> Create `docs/adr/` directory in `BD-design` repo. Write one ADR markdown file for each of these decisions using Michael Nygard format (Title, Status, Context, Decision, Consequences, Alternatives Considered): (1) `001-vercel-serverless.md` — why Vercel Functions over containers/VMs, (2) `002-supabase-postgres.md` — why Supabase over self-hosted Postgres or other managed DB, (3) `003-jwt-authentication.md` — why JWT over server-side sessions, trade-offs including no-refresh-token gap, (4) `004-no-orm.md` — why raw `pg` over Prisma/TypeORM/Sequelize, (5) `005-socket-io-deferred.md` — why Socket.io is present but disabled, and what the migration path to production real-time looks like (Supabase Realtime or Ably).

---

### [ ] D3 — Reconcile Test Count and Socket.io Status

> In `BD-design` repo: (1) in `review-pack/07-quality-test-evidence.md`, update the test count to match the actual count from `BD_ITA` package.json test output — remove the "100/100" claim and replace with the actual number and a note that integration tests are currently being fixed, (2) in `review-pack/02-current-state-architecture.md`, add a section "Real-Time (Socket.io)" stating explicitly: "Socket.io client and server are installed but disabled in the current serverless deployment. Real-time features are not operational in the current version. Production real-time will require migration to a managed WebSocket service (Supabase Realtime or Ably).", (3) in `review-pack/01-executive-readout.md`, add to Key Risks: "JWT tokens expire after 24 hours with no refresh mechanism — users are hard-logged out mid-session daily. Token refresh is a Phase 1 gap."

---

### [ ] D4 — Fix Persona-to-Role Mapping in PRD

> In `BD_Platform_PRD_v1.0.docx` in `BD-design/design-docs/`: (1) update §4 (Personas) to show which system role each persona maps to — add a "System Role" column to the persona table, (2) update §5.1 (Roles) to use consistent role names: `bd_lead`, `analyst`, `admin`. Map: BD Lead → `bd_lead`, Proposal Manager → `analyst`, BD Executive → `admin` (read-heavy), System Admin → `admin`, (3) add a Role-to-Capability matrix table showing which endpoints and features each role can access, (4) clarify §10 Constraints: replace "POC is single-user (Sam Gruverman)" with "POC is single-tenant, deployed for a single organization. Multi-user access is supported within that organization via role-based access control."

---

### [ ] D5 — Generate ERD

> Using the `database/schema.sql` file from `BD_ITA`, generate an Entity Relationship Diagram. Use dbdiagram.io DBML format or Mermaid `erDiagram` syntax. Save the output as `docs/erd.md` (Mermaid) or `docs/erd.dbml` (DBML) in `BD-design`. Include all 23 tables with their primary keys, foreign key relationships, and key non-null fields. Add the ERD file path to the `review-pack/05-data-model-api-contract.md` doc as a reference.

---

### [ ] D6 — Expand Ops Runbook

> In `BD-design/review-pack/08-ops-deployment-runbook.md`, expand from the current 5-bullet stub to cover: (1) deployment procedure — step-by-step Vercel deployment from `main`, including env var checklist, (2) database migration procedure — how to run `node-pg-migrate up` against production Supabase, (3) rollback procedure — how to revert a bad deployment on Vercel (previous deployment promotion), (4) database restore procedure — how to restore from Supabase backup, (5) incident response — who to contact, how to check Vercel function logs, how to check Supabase connection count, (6) monitoring checklist — Vercel analytics, Supabase dashboard, error rate indicators.

---

### [ ] D7 — Create Scaling Strategy Document

> In `BD-design/docs/`, create `scaling-strategy.md`. Content should mirror what is produced in `SC5` (Codex action plan item above) — copy or link between repos as appropriate. Include DB connection budget math, Supabase tier upgrade triggers, Socket.io migration path, and Phase 3 horizontal scaling considerations.

---

## Testing Additions (Phase 2–3)

### [ ] T1 — Fix Integration Test Suite

> See item `H6` above — already covers this.

### [ ] T2 — Add Auth Flow End-to-End Test

> In `backend/tests/`, create `integration/auth.integration.test.js`. Using `supertest` against the full Express app: (1) register a new user, (2) log in and receive a JWT, (3) call a protected route with the JWT — expect 200, (4) call a protected route without a JWT — expect 401, (5) call a protected route with an invalid JWT — expect 401. Use the test database (`backend/.env.test`). Clean up created user in `afterAll`.

### [ ] T3 — Add Coverage Threshold to Jest

> In `backend/jest.config.js` (create if absent), add: `collectCoverageFrom: ['src/**/*.js']` and `coverageThreshold: { global: { lines: 70, functions: 70 } }`. In `.github/workflows/quality-gate.yml`, add `--coverage` flag to the Jest run command. Add a step that fails if coverage drops below threshold. Report coverage summary to the CI log.

### [ ] T4 — Add Frontend `AuthContext` Tests

> In `frontend/src/`, create `context/AuthContext.test.jsx`. Using React Testing Library and `vitest`: (1) render a component wrapped in `AuthContext.Provider`, (2) mock the API client (`frontend/src/api/client.js`), (3) test login — call login function, verify token is stored in localStorage, verify user state is set, (4) test logout — verify token is removed, user state is null, (5) test initial load — verify token from localStorage is used to restore session on mount.
