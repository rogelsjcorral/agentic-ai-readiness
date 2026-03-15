# Revision History

This document tracks substantive changes to the paper and its operational artifacts.
Minor copy edits and formatting changes are not listed.

## Rev v0.4 (2026-03-15)
- Added tier-differentiated autonomy budget starting values (T1–T4) with a reference table for 11 cap dimensions.
- Expanded readiness scorecard with an inline control matrix mapping all six dimensions to tiers, formatted as a rollout blocker checklist.
- Operationalized all six evaluation metrics (UAR, ISR, TDH, RSR) with formal definitions, denominator formulas, measurement windows, and interpretation guidance to match the existing BRS and POI specifications.
- Expanded rollout ladder phases with entry criteria, exit criteria, minimum duration and sample thresholds, advancement authority, and regression conditions.
- Added implementation detail to control patterns 6.1–6.5 (tool allowlisting, planner-executor separation, two-channel confirmation, least privilege credentials, lockdown mode), including common failure patterns and scope limitations for each.
- Strengthened quorum gating guidance in Section 6.7 with structural independence requirements and a description of the shared-context poisoning attack on quorum rules.

## Rev v0.3 (2026-03-15)
- Added Appendix A autonomy budget template.
- Defined calculation guidance for Blast Radius Score and Privilege Overreach Index.
- Expanded Tier 4 guidance with multi-agent delegation constraints.
- Clarified Tier 2 vs Tier 3 eligibility gates.

## Rev v0.2 (2026-03-14)
- Expanded production failure modes (scope mismatch, injection, over-privilege, cascading actions, memory poisoning, approval fatigue).
- Added readiness scorecard minimum bars by capability tier.
- Added evaluation harness concept and baseline metrics.

## Rev v0.1 (2026-03-13)
- Established baseline scope, terminology, capability tiers, and autonomy budget concept.
- Drafted initial operational risk model for tool-using agents.
