# POC Completion Plan — PRD v1.3 Gap Closure

**Created:** 2026-06-12
**Verified against:** live Supabase schema (39 tables) + `BD_ITA` routes/migrations
**Format:** follows `08-codex-action-plan.md` — each item: what exists, the gap, a paste-ready agent prompt, size.
**Sequencing:** Group A (schema deltas) → Group B (UI) → Group C (cross-cutting) → Group D (debt, interleave anytime).

Key finding: the schema is ahead of the PRD. Four of six deck-parity items are partially or fully present in the database; most work is UI exposure and small column additions, not new architecture.

---

## Group A — Schema Deltas

### [ ] A1 — Step 2 Qualifying Checklist (PRD §5.3.2)

**Exists:** `deal_step3_readiness_checklist` (Step 3 only). Nothing for Step 2.
**Gap:** new sub-resource, four yes/no + notes questions.

> Create migration `0NN_step2_qualifying_checklist.sql`: table `deal_step2_checklist` (id, deal_id FK→deals, question_key text, answer boolean, notes text, created_at, updated_at). Seed four question keys per deal on first access (follow the seeding pattern in `006_step3_checklist_seed_defaults.sql`): `old_solicitation` ("Do we have the old solicitation?"), `public_info_reviewed` ("Have we reviewed publicly available information — press releases, G2X, incumbent site?"), `agency_support` ("Do we support this agency, or can we pull in a team member who does?"), `customer_access` ("Do we have a way to get in front of the customer?"). Add GET/PUT endpoints in `backend/src/routes/dealContext.js` following the existing step3 checklist pattern, with `validateBody` contract. Unit test the route with supertest.

**Size:** M

### [ ] A2 — Key Meetings: attendees field (PRD §5.3.3)

**Exists:** `deal_key_meetings` (meeting_type, meeting_date, notes).
**Gap:** deck and PRD require attendees.

> Create migration adding `attendees text` to `deal_key_meetings`. Add the field to the meetings contract in `dealContext.js` and to the meetings form/list UI. Test update.

**Size:** S

### [ ] A3 — Teaming Strategy: rationale per sub (PRD §5.3.3)

**Exists:** `deal_partners` (role, work_percentage, legal_status, selection_status, agreement_type, ta_status) + partner taxonomy (migration 007).
**Gap:** deck slide 16 wants size and rationale per teaming partner.

> Create migration adding `teaming_rationale text` to `deal_partners`. Confirm partner size/demography lives on `partners` via migration 007 taxonomy; if absent, add `business_size text` to `partners`. Expose both in the partners contract and the Partners tab UI (rationale as textarea, size as select using existing taxonomy options). Tests.

**Size:** S-M

### [ ] A4 — Strengths/Weaknesses/Benefits — "Why Us?" (PRD §5.3.3)

**Exists:** `swot` (category, description, impact) — but SWOT tab is a different artifact than deck slide 18.
**Gap:** Step 3 needs three free-text blocks: strengths, weaknesses, benefits-to-customer/why-us.

> Create migration `deal_swb` (id, deal_id FK, strengths text, weaknesses text, customer_benefits text, updated_at) — one row per deal. Add GET/PUT in `dealContext.js` following the `deal_customer_knowledge` single-row pattern. Add a "Strengths / Weaknesses / Why Us" section to the Step 3 capture screen. Do NOT merge with the SWOT tab — they serve different stages. Tests.

**Size:** M

### [ ] A5 — Voice of the Customer: stakeholder link + fields (PRD §5.3.3)

**Exists:** `deal_voice_of_customer` (concern_need, associated_customer text, response_strategy).
**Gap:** PRD wants stakeholder picked from previously entered stakeholders or added new, plus role/title, other notes, [Company] access.

> Create migration adding to `deal_voice_of_customer`: `stakeholder_id int NULL FK→stakeholders`, `stakeholder_role text`, `other_notes text`, `company_access text`. Keep `associated_customer` for free-text fallback. Update VOC contract and UI: stakeholder dropdown sourced from the deal's stakeholders with an "Add new" path that creates the stakeholder record first. Tests.

**Size:** M

---

## Group B — UI Exposure (schema already done)

### [ ] B1 — Capabilities Alignment scoring legend + concerns row (PRD §5.3.2/§5.3.3)

**Exists:** `deal_capability_alignment` with `referenceability_score`, `concern_weaknesses`.
**Gap:** verify/complete UI.

> In the capability alignment component (bound via `dealContext.js`), ensure: (1) each SOW/capability pair row has a 1–3 score selector bound to `referenceability_score`; (2) legend rendered under the table: "1 = 1 referenceable PP or excellent CPAR · 2 = 2 · 3 = 3+"; (3) a persistent "Areas of Concern or Weakness" row bound to `concern_weaknesses`; (4) entries display as a table after capture per PRD display note. Component test.

**Size:** S

### [ ] B2 — Customer Knowledge section (PRD §5.3.3)

**Exists:** `deal_customer_knowledge` (customer_familiarity, familiarity_details, positioning_step_1..4) — complete.
**Gap:** verify UI exposes familiarity yes/no + capacity description + the four numbered positioning steps on the Step 3 screen.

**Size:** S (verification + polish)

### [ ] B3 — Decision Point presentation (PRD §5.3.2/§5.3.3)

**Exists:** `deal_reviews` (step, decision, decision_notes, status) + decision outcomes (migration 008).
**Gap:** verify both Step 2 and Step 3 screens end with the explicit three-outcome decision control: Approved to proceed | No-bid | Complete actions and re-present; confirm outcome labels match PRD wording.

**Size:** S

### [ ] B4 — Deal color badge descriptions (PRD §5.2, status Partial)

> On dashboard deal cards, add a tooltip or sublabel to the color badge: Blue = new business, Green = recompete, White = IDIQ/task order. Component test.

**Size:** S

### [ ] B5 — Competitor demography (PRD §5.3.2)

**Exists:** `deal_competitor_hypotheses` (competitor_name, rationale, details, confidence).
**Gap:** PRD wants demography per competitor (SB, WOSB, DVSB, ANC, Native American, Large).

> Migration: add `demography text` to `deal_competitor_hypotheses`. Add a select with the six options to the competitor entry form. Include in the displayed competitor table. Tests.

**Size:** S

---

## Group C — Cross-Cutting

### [ ] C1 — Executive Readout View (PRD §5.2, P1)

**Exists:** deck-ordered traceability (review-pack 11); deal detail sections.
**Gap:** a dedicated view that steps or scrolls through Step 2/3 sections in deck order, ending at the Decision Point.

> Create an Executive Readout route/page for a deal: read-only, deck-ordered sections (Opportunity Info → Schedule → Step 2 items → Step 2 Decision → Step 3 items → Step 3 Decision), one section per screen with next/previous navigation AND continuous-scroll mode. Pull data from the existing `/api/deals/:id/details` and context endpoints — no new backend. Final screen shows the decision control (or recorded decision). Add a "Readout" button on the deal card and deal detail header. Component tests for navigation order.

**Size:** M

### [ ] C2 — Role rename: analyst → standard_user (PRD §4.6)

> Migration: `UPDATE roles SET name='standard_user' WHERE name='analyst';` plus same on `users.role` values. Global find/replace `analyst` → `standard_user` in backend role checks and frontend role displays. Update seed files and `.env.example` references. Keep a one-release compatibility alias in the auth middleware (accept both, log deprecation) to avoid breaking existing JWTs until expiry. Run full test suite.

**Size:** S-M (alias is the careful part)

### [ ] C3 — Save / Save-and-Exit / Save-and-Create-New (PRD §5.3 repeating entries)

> Audit repeating-entry forms (competitors, stakeholders, VOC, meetings, capability rows): each should offer Save, Save & Exit, Save & Create New per PRD. Implement missing actions using the existing save-flow pattern. Component tests.

**Size:** S-M

---

## Group D — Engineering Debt (from 09-reconciliation, interleave anytime)

| Item | Source | Size |
|---|---|---|
| [ ] D1 — gateReadiness unit tests (M4) | `08-codex-action-plan.md` M4 | S |
| [ ] D2 — DDL-on-demand → startup migrations (C2 in 08) | `08` C2 — `ensure*()` still in 5 files | M |
| [ ] D3 — Redis-backed rate limiter (H3) | `08` H3 | S-M |
| [ ] D4 — CI: blocking npm audit, ESLint, coverage threshold (S1/M7/T3) | `08` | S |
| [ ] D5 — auth test supertest rewrite (M5), auth.js split (M8) | `08` | S |
| [ ] D6 — `/health` endpoint performs `SELECT 1` against DB | new (found 2026-06-11: health returns ok with DB down) | S |

---

## Suggested Sprint Sequence

1. **Sprint 1 (demo value):** A1, A2, B1, B3, B4 + D6 — Step 2 capture complete and presentable.
2. **Sprint 2 (Step 3 complete):** A3, A4, A5, B2, B5, C3.
3. **Sprint 3 (exec demo):** C1 exec readout + C2 role rename + D1.
4. **Ongoing:** D2-D5 interleaved, one per sprint.

Total: roughly 2-3 focused weeks of agent-driven work for Groups A-C; Group D adds ~1.

## Doc Maintenance

When an item closes, mark `[x]` here and reflect status in `09-current-state-reconciliation.md`. PRD v1.3 feature tables should flip Partial→Built as B1/B3/B4/C1 land.
