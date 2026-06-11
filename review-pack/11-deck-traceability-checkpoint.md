# Deck Traceability Checkpoint (Updated 2026-05-01)

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

4. Slides 6-7 - Step 2 Items + Incumbent: Partial
- Incumbent profile now maps to canonical structure and persists.
- Need stricter required fields and URL validation gates for FPDS/press links.

5. Slides 8-11 - Capability, Competitors, Customer, Other Intel: Partial
- Competitor/stakeholder/workflow exists.
- Data remains mixed between structured tables and `uiData` blobs; needs normalization for decision-grade reporting.

6. Slide 13 - Step 2 Decision: Pass
- Decision model supports governance outcomes: `go`, `conditional_go`, `hold_recycle`, `no_go`.
- Readiness guidance is shown in-line; save/share remains non-blocking to support an evolving workflow.

7. Slides 14-15 - Step 3 Status + Contract Changes: Partial
- Contract status updates persist (`review_step`, `changes_since_last_step`, `funding_modifications`, `captured_at`).
- Need stronger UI sequencing tied to Step 3 flow and decision readiness cues.

8. Slide 16 - Teaming Strategy: Partial
- Partner/workshare exists.
- Rationales/size coverage still needs strict deck-field parity checks.

9. Slides 17-21 - Updated Capability, SWOT/Benefits, Customer Knowledge, VOC, Meetings: Partial
- Core sections exist in UI.
- Some areas still persist as freeform arrays/`uiData`; requires structured models for auditability and scoring.

10. Slide 22 - Pricing Intel: Partial
- Basic pricing fields exist.
- Needs explicit deck-aligned capture model and validation.

11. Slide 23 - Capture Roles: Partial
- Role-like fields exist across overview/program data.
- Needs consolidated, canonical capture-role section.

12. Slide 24 - Step 3 Preparation Checklist: Partial
- Structured checklist capture exists and appears in context readiness signals.
- Baseline threshold enforcement and weighted/section-level scoring are implemented; remaining gap is deeper section-by-section evidence normalization.

13. Slide 25 - Step 3 Decision: Pass
- Decision model exists with weighted readiness guidance and advisory thresholds.

## Current Decision Readiness
Overall: Partial / Improving, not yet governance-ready.

Blocking reasons:
- Incomplete structured normalization across several Step 3 data domains.
- Remaining evidence normalization gaps in a subset of decision-critical sections.
- Need stronger UX guidance and reporting surfacing for readiness trends across deals.

## New Context From Gate-Review Research
- Step review decks should be treated as governance decision artifacts, not presentation-only summaries.
- Expected operating model:
  - explicit outcomes (`go`, `conditional go`, `hold/recycle`, `no-go`);
  - predefined scoring criteria and evidence traceability;
  - auditable rationale for every gate decision.
- This context confirms current trajectory (deck-ordered capture + readiness reporting) and defines next required build step: gate scoring + decision outcomes.

## Immediate Next Build Sequence
1. Extend current gate thresholds into weighted Gate Scoring + Decision Outcome model (`go`, `conditional go`, `hold/recycle`, `no-go`) for Step 2/3.
2. Add required-evidence threshold matrix aligned to deck-critical sections.
3. Normalize remaining `uiData` decision-critical sections into structured entities.
4. Add end-to-end "step-decision certification" test for evidence thresholds and outcome transitions.
