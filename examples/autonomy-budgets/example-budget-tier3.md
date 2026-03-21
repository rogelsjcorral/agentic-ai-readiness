Appendix A. Autonomy budget template

## A.2 Budget Record
Budget ID: EX-T3-001  
Owner: Security Engineering  
Environment: production  
Capability Tier: 3  
Effective Date: 2026-03-21  
Review Cadence: weekly  
Emergency Contact: oncall-security@example.com  

## A.3 Global Execution Caps
Max runtime per task (seconds): 120  
Max tool calls per task: 40  
Max concurrent tool calls: 2  
Max unique systems touched per task: 3  
Max unique target resources per task: 10  

## A.4 Write and Privilege Caps
Max write actions per task: 20  
Max privileged actions per session: 2  
Max irreversible actions per session: 0 (default)  
Max delete operations per task: 1  
Max permission changes per task: 2  
Max spend per task (currency): 0  
Max external data exports per task (records/bytes): 0  

## A.5 Credential Constraints
Credential lifetime (minutes): 10  
Credential scope granularity: per resource  
Privileged credentials allowed: yes  
Separation of duties required for privileged actions: yes  

## A.6 Escalation Triggers
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

## A.7 Evidence Requirements (Tier 2+)
Every write action must produce an evidence record containing intent reference, enumerated targets, approvals, tool calls, outcomes, and rollback identifier when available.
