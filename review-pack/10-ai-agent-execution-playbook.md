# AI Agent Execution Playbook

## Objective
Use AI agents to implement safely, verify rigorously, and hand off clearly without overcomplicating code.

## Guardrails
- Keep implementations simple/effective/scalable.
- Security-first defaults.
- No hidden environment assumptions.
- Commit small, validated increments.

## Standard Agent Loop
1. Plan smallest meaningful increment.
2. Implement change.
3. Run verification (`verify-all` safe/full/browser as appropriate).
4. Produce/inspect diagnostic artifacts on failure.
5. Apply focused fix using fault-map file targets.
6. Re-verify and commit.

## Required Outputs Per Iteration
- Commit message with intent.
- Verification result summary.
- If failed: likely fault location + suggested fix path.

## Escalation Conditions
- Major UX/functional direction change: pause for option review.
- Security-sensitive change: require explicit risk note in handoff.
- Repeated failure loop: switch strategy and document why.

## Session Handoff
- Update status file and working memory.
- Leave clear next 1-3 executable steps.
