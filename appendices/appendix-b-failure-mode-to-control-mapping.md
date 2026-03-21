# Appendix B. Failure mode to control mapping

## B.1 Purpose
This appendix maps each production failure mode from Section 4 to the primary controls in Section 6 and the metrics in Section 8 that detect it. Teams can use this table to verify that every failure mode is covered before advancing a rollout phase, and to prioritise which controls to implement first for a given deployment shape.

## B.2 Reading the table
Primary control is the control pattern most directly preventing or limiting the failure mode. Secondary controls provide defence in depth. Detection metric is the metric in Section 8.2 most sensitive to this failure mode occurring. A failure mode with no instrumented detection metric is a blind spot; teams should treat it as a rollout blocker.

| Failure mode | Primary control | Secondary controls | Detection metric | Minimum tier gate |
|---|---|---|---|---|
| Intent drift and scope mismatch (4.1) | Planner executor separation (6.2); two channel confirmation (6.3) | Tool allowlisting and action schemas (6.1); evidence records (6.6) | UAR; BRS | Tier 2 |
| Prompt injection through untrusted tool outputs (4.2) | Retrieval isolation; lockdown mode (6.5) | Tool allowlisting (6.1); planner executor separation (6.2) | ISR; TDH | Tier 1 |
| Over-privilege and credential mis-scoping (4.3) | Least privilege and short lived credentials (6.4) | Tool allowlisting (6.1); evidence records (6.6) | POI | Tier 2 |
| Cascading actions and self-reinforcing context (4.4) | Autonomy budget enforcement (Section 3.2); lockdown mode (6.5) | Planner executor separation (6.2); evidence records (6.6) | BRS; TDH | Tier 2 |
| Memory poisoning and durable state corruption (4.5) | Retrieval isolation; memory hygiene rules (see 5.1 adversarial resilience) | Lockdown mode (6.5); evidence records (6.6) | ISR (cross-session variant) | Tier 2 |
| Approval fatigue and weak human gating (4.6) | Two channel confirmation with diff visibility (6.3) | Approval fatigue instrumentation (Section 7.3); anomaly alerts | Approval fatigue indicators (Section 7.3) | Tier 2 |
| Shared context quorum failure (Section 6.7.1) | Independence requirement enforcement (6.7.1) | Multi agent delegation controls (6.7); lockdown on verification failure | ISR (cross-agent variant); BRS | Tier 4 |
| Audit trail suppression (Section 7.5) | Audit storage access controls (7.5) | Control plane write restriction; central audit store at Tier 4 | Audit completeness check (7.2) | Tier 2 |

## B.3 Coverage gaps to check before advancing
Before advancing past Sandbox mode, verify that every failure mode applicable to the deployment's capability tier has an instrumented detection metric and a primary control implemented and tested. A failure mode with a control but no detection metric means an incident could occur without generating a signal. A failure mode with a detection metric but no control means the metric will fire but nothing will stop or contain the failure.
