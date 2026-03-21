# Start Here

This repository is an operational field manual for deploying tool-using LLM agents with enforceable controls, measurable gates, and phased rollout.

## What you do first
1. Pick the target capability tier (Tier 1 through Tier 4).
2. Define an autonomy budget for that tier and environment.
3. Implement the control patterns required by the readiness scorecard for that tier.
4. Emit audit events for every tool call and generate an evidence record for every write.
5. Run the evaluation scenario suite and compute metrics.
6. Roll out using the ladder with regression triggers enabled.

## Minimal adoption path (Tier 2)
1. Fill an autonomy budget.
   - Use `appendices/appendix-a-autonomy-budget-template.md`
   - See examples under `examples/autonomy-budgets/`
2. Enforce tool allowlisting and action schemas.
3. Require approvals for every write action.
4. Require tested rollback for every permitted write.
5. Emit audit events and generate evidence records.
6. Run the scenario suite.
   - Use `appendices/appendix-f-evaluation-scenarios.md`
7. Roll out using the ladder.
   - Use `appendices/appendix-g-rollout-checklist.md`

## Where to find things
- Paper and release artifact: `paper/`
- Appendices and operational kit: `appendices/`
- Examples (filled artifacts): `examples/`
- License overview: `LICENSE`
- Full license texts: `LICENSES/`
