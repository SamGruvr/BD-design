# Product Scope + UX Baseline

## Latest UX Checkpoint (2026-05-02)
- Deal Detail navigation concept validated: keep global app sidebar and add second-level left section navigation within Deal Detail.
- Capture Context redesign concept drafted in **review-first** layout (summary-first, evidence matrix, reviewer aids, mode toggle).
- Mockups saved locally for next-session decision:
  - `/Users/sama/.codex/generated_images/019d7de1-b110-7fb0-bd1e-47cf00aef623/ig_003b5c0a60f95fad0169f676a028808198949fb337056dfe25.png`
  - `/Users/sama/.codex/generated_images/019d7de1-b110-7fb0-bd1e-47cf00aef623/ig_003b5c0a60f95fad0169f677bbe1a88198a8e9f37310350455.png`
- Implementation status: **not yet coded**; awaiting final UX direction confirmation.

## Implemented User Workflows
- Login/Register/Account profile update.
- Deals list with filtering/sorting.
- Deal detail with multi-tab data views.
- Review decision save flow (Step 2/3).
- Capture context editing and display, now 12 sections including Step 2 Qualifying Checklist and Strengths/Weaknesses/Why-Us (SWB).
- Capture Content page with deck-ordered readiness view and direct context drilldown.
- Risk/action linkage and management.
- Stakeholders page with editable stakeholder profile capture.
- Reports page with cross-deal readiness baseline.
- Admin console + dedicated user administration.
- Audit log review.
- Executive Readout (`/deals/:id/readout`): read-only deck-ordered walkthrough per deal, step-through or scroll mode, ending at the recorded/pending decision.

## UX State
- Readability/contrast improved in multiple pages.
- Information density and clarity improved in admin/audit/deal workflows.
- Remaining opportunity: reduce form friction and tighten consistency across interaction patterns.

## Internal Preview Readiness
- Suitable for internal functional walkthroughs and process validation.
- Not yet final-polish for high-stakes external stakeholder demo.

## Highest UX Gaps
1. Consistency of labels/microcopy/action confirmations.
2. Formal gate-outcome UX (explicit gate decision states and rationale capture across Step 2/3) — partially addressed: Executive Readout now surfaces the recorded decision (or "No decision recorded") with outcome language at the end of each step's sections; the decision-entry UX itself is unchanged.
3. Streamlining top workflow path (login -> capture content -> step review -> decision submit).
4. Aligning visual hierarchy and spacing across high-traffic pages.
5. VOC stakeholder selection is still free-text entry, not a picker against existing deal stakeholders.
6. Partners tab does not yet expose teaming rationale or partner size/demography, though both are captured in the data model.
