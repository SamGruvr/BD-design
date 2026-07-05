# Deck Traceability Checkpoint (Updated 2026-06-13)

## Purpose
Validate current app capture against the Imaginam Step Review deck order and identify blockers to stakeholder decision readiness.

## Status Key
- Pass: Implemented and persisted with canonical backend fields.
- Partial: Some fields/workflow exist, but gaps remain in structure/order/validation.
- Gap: Missing or not reliably captured for decision use.

## Deck-Ordered Coverage
1. Slide 1 - Opportunity Name / Prepared By / Step 2 Date / Step 3 Date: Partial
- Opportunity name and review step/date exist.
- Prepared-by is derived from logged-in user.
- Step header exists in deal detail; Step 2/3 dates still need stronger dedicated header presentation.

2. Slides 3-4 - Opportunity Information and Role/Rationale: Pass
- Captured in `deal_opportunity_profile` via context editor.
- Fields aligned: `program_name`, `brief_description`, `procurement_method`, `place_of_performance`, `contract_type`, `imaginam_role`, `pursuit_rationale`.

3. Slide 5 - Schedule and Key Dates: Partial
- Structured milestones captured via `deal_milestones` with canonical types.
- Need explicit ordering/validation to ensure all core milestone types are completed for readiness.

4. Slides 6-7 - Step 2 Items + Incumbent: Pass (Step 2 Qualifying Checklist) / Partial (Incumbent)
- Incumbent profile maps to canonical structure and persists; still needs stricter required fields and URL validation gates for FPDS/press links.
- Step 2 Qualifying Checklist (old solicitation, public info reviewed, agency support, customer access) now captured in `deal_step2_checklist`, exposed via dedicated PUT endpoint, and surfaced in both the capture form and Executive Readout.

5. Slides 8-11 - Capability, Competitors, Customer, Other Intel: Partial, several sub-items closed
- Capability Alignment: scoring legend ("1 = one referenceable PP or excellent CPAR · 2 = two · 3 = three or more") and persistent concerns/weaknesses row confirmed live in both the capture form and readout — Pass.
- Competitor hypotheses: demography field (SB/WOSB/DVSB/ANC/Native American/Large) added and displayed — Pass.
- Remaining gap: data still mixed between structured tables and `uiData` blobs elsewhere in this slide range; needs further normalization for decision-grade reporting.

6. Slide 13 - Step 2 Decision: Pass
- Decision model supports governance outcomes: `go`, `conditional_go`, `hold_recycle`, `no_go`.
- Readiness guidance is shown in-line; save/share remains non-blocking to support an evolving workflow.

7. Slides 14-15 - Step 3 Status + Contract Changes: Partial
- Contract status updates persist (`review_step`, `changes_since_last_step`, `funding_modifications`, `captured_at`).
- Need stronger UI sequencing tied to Step 3 flow and decision readiness cues.

8. Slide 16 - Teaming Strategy: Partial, rationale closed
- Partner `teaming_rationale` (free-text) added to `deal_partners`, exposed in the Partners contract and backend; frontend Partners tab UI (`DealPartners.jsx`) has not yet been updated to display/edit it — UI exposure remains open.
- Partner size/demography (via existing partner taxonomy, migration 007) still needs strict deck-field parity checks.

9. Slides 17-21 - Updated Capability, SWOT/Benefits, Customer Knowledge, VOC, Meetings: Partial, several sub-items closed
- Strengths / Weaknesses / Benefits-to-Customer ("Why Us?") now captured in dedicated `deal_swb` table, distinct from the SWOT tab per PRD intent — Pass.
- Key Meetings: `attendees` field added, captured and displayed — Pass.
- Voice of Customer: `stakeholder_role`, `other_notes`, `company_access` fields added and captured — Pass on fields; stakeholder linkage is still free-text (`associated_customer`), not a dropdown sourced from the deal's stakeholder list with an "add new" path — that part of PRD §5.3.3 remains open.
- Customer Knowledge: verified by inspection — familiarity yes/no/partial, capacity details, and four numbered positioning steps already exposed on the Step 3 screen — Pass.
- Remaining gap: some areas still persist as freeform arrays/`uiData`; requires structured models for auditability and scoring.

10. Slide 22 - Pricing Intel: Partial
- Basic pricing fields exist.
- Needs explicit deck-aligned capture model and validation.

11. Slide 23 - Capture Roles: Partial
- Role-like fields exist across overview/program data.
- Needs consolidated, canonical capture-role section.

12. Slide 24 - Step 3 Preparation Checklist: Partial
- Structured checklist capture exists and appears in context readiness signals.
- Baseline threshold enforcement and weighted/section-level scoring are implemented and now unit-tested (7 tests on `gateReadiness.js`); remaining gap is deeper section-by-section evidence normalization.
- Scoring anomaly documented, not yet fixed: Step 3 section weights (35+20+20+15+5+5+10) sum to 110, not 100 — a fully complete Step 3 scores 110. Empty sections also score as 100% (vacuous), giving Step 2 a 10-point floor from the empty Reserve section alone.

13. Slide 25 - Step 3 Decision: Pass
- Decision model exists with weighted readiness guidance and advisory thresholds.

## Executive Readout (New)
A dedicated deck-ordered readout view now exists at `/deals/:id/readout` (deal detail header link), covering all 13 sections above in deck order — step-through (one section, Previous/Next) and continuous-scroll modes — and ending at each recorded Step 2/Step 3 decision or "No decision recorded" if pending. Reads entirely from existing deal/context/review endpoints; no new backend surface. Live-verified 2026-06-13 in both modes against a real deal.

## Current Decision Readiness
Overall: Partial / Improving, not yet governance-ready. Most deck-parity data-capture gaps identified at the 2026-05-01 checkpoint are now closed (Step 2 Checklist, SWB, teaming rationale, meetings attendees, VOC fields, competitor demography, capability scoring legend, Executive Readout). Remaining blockers are narrower and listed below.

Blocking reasons:
- Partner size/demography and teaming rationale not yet surfaced in the Partners tab UI (backend/data model ready).
- VOC stakeholder linkage still free-text; no dropdown-from-stakeholders or "add new" path.
- Incomplete structured normalization across several Step 3 data domains (uiData blobs vs. structured tables).
- Gate readiness scoring anomaly (Step 3 weights sum to 110; empty sections score 100%) not yet normalized.
- Need stronger UX guidance and reporting surfacing for readiness trends across deals.

## New Context From Gate-Review Research
- Step review decks should be treated as governance decision artifacts, not presentation-only summaries.
- Expected operating model:
  - explicit outcomes (`go`, `conditional go`, `hold/recycle`, `no-go`);
  - predefined scoring criteria and evidence traceability;
  - auditable rationale for every gate decision.
- This context confirms current trajectory (deck-ordered capture + readiness reporting) and defines next required build step: gate scoring + decision outcomes.

## Immediate Next Build Sequence
1. Normalize `gateReadiness.js` section weights to sum to 100 and decide on empty-section scoring semantics (currently vacuous 100%) — both are unit-tested and documented, not yet fixed.
2. Build VOC stakeholder dropdown (sourced from deal stakeholders, with "add new" path) to replace the current free-text fallback.
3. Surface `teaming_rationale` and partner size/demography in the Partners tab UI (`DealPartners.jsx`) — data model and API are ready.
4. Normalize remaining `uiData` decision-critical sections into structured entities.
5. Add end-to-end "step-decision certification" test for evidence thresholds and outcome transitions.
