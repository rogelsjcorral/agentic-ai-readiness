Appendix A. Autonomy Budget Template

A.1 Purpose
This template defines enforceable caps for tool using agents. Budgets are tier specific and must be enforced by the control plane, not by prompts. Budgets exist to bound blast radius when ambiguity, injection, or tool misuse occurs.

A.2 Budget Record
Budget ID:
Owner:
Environment: sandbox / staging / production
Capability Tier: 0 / 1 / 2 / 3 / 4
Effective Date:
Review Cadence: weekly / monthly / quarterly
Emergency Contact:

A.3 Global Execution Caps
Max runtime per task (seconds):
Max tool calls per task:
Max concurrent tool calls:
Max unique systems touched per task:
Max unique target resources per task:

A.4 Write and Privilege Caps
Max write actions per task:
Max privileged actions per session:
Max irreversible actions per session: 0 (default)
Max delete operations per task:
Max permission changes per task:
Max spend per task (currency):
Max external data exports per task (records or bytes):

A.5 Credential Constraints
Credential lifetime (minutes):
Credential scope granularity: per tool / per action type / per resource
Privileged credentials allowed: yes / no
Separation of duties required for privileged actions: yes / no

A.6 Escalation Triggers
Approval required when:
1. Any write action is proposed above threshold.
2. Any privileged action is proposed.
3. Any cap in A.3 or A.4 would be exceeded.

Lockdown triggers when:
1. Untrusted content is detected in the instruction channel.
2. Plan execution deviates from the approved plan.
3. Novel targets exceed baseline.
4. Repeated tool failures exceed threshold.

Human takeover required when:
1. Rollback is unavailable for any proposed write.
2. Privileged actions are requested.
3. Injection indicators are present.

A.7 Evidence Requirements (Tier 2+)
Every write action must produce an evidence record containing intent reference, enumerated targets, approvals, tool calls, outcomes, and rollback identifier when available.
