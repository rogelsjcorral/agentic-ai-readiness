# Appendix C. Readiness scorecard checklist

## C.1 Purpose
This checklist operationalises the scorecard in Section 5. For each dimension and tier, it provides concrete questions an operator can answer with a yes or no. A dimension passes only when all required questions for the target tier are answered yes. Any no answer is a blocking finding that must be resolved before the tier is eligible.

## C.2 How to use
Complete one column per target tier. Mark each item as Pass (yes), Fail (no), or N/A where the item does not apply at the target tier. A tier is eligible only when all applicable items are marked Pass. Carry forward all lower-tier requirements: a team qualifying for Tier 3 must also satisfy all Tier 1 and Tier 2 items.

## C.3 Dimension 1: Authority control

**Tier 1.**
- Are all tool credentials scoped to read-only access with no write permissions?
- Is the credential scope documented and auditable?
- Are credentials short lived with a defined maximum lifetime?

**Tier 2 (all Tier 1 items plus):**
- Are write credentials scoped by action type, not issued as broad tokens?
- Is a credential scope record maintained per tool and per action class?
- Is the token TTL enforced by the control plane, not by the agent runtime?

**Tier 3 (all Tier 2 items plus):**
- Are credentials minted per task, scoped to the specific resource identifiers in the approved plan?
- Is separation of duties enforced for privileged actions, such that the agent cannot both propose and execute without an external approval artifact?
- Is the token issuance logic part of an auditable trusted computing base?
- Are privileged credentials invalid outside the plan context?

**Tier 4 (all Tier 3 items plus):**
- Are delegation boundaries defined and enforced by the control plane?
- Does each delegate agent receive credentials scoped to its inherited autonomy budget only?
- Is the delegate credential lifetime bounded to the parent task window?

## C.4 Dimension 2: Intent verification

**Tier 1.** Not required.

**Tier 2:**
- Does every write action require a structured plan before execution?
- Does the plan enumerate target resources explicitly, not by pattern or wildcard?
- Is a diff summary presented to the approver before execution?
- Is the approval bound to an auditable approval identifier?

**Tier 3 (all Tier 2 items plus):**
- Is intent verification required per privileged action class, not once per session?
- Does the confirmation step restate enumerated targets and the diff in a channel separate from the conversational interface?
- Is tool execution blocked until the approval identifier is presented to the executor?

**Tier 4 (all Tier 3 items plus):**
- Is intent verification required per delegate action and target set, not per orchestration session?
- Does the immutable context bundle carry the approved plan reference?
- Are delegate actions traceable to the parent plan approval identifier?

## C.5 Dimension 3: Adversarial resilience

**Tier 1:**
- Is retrieved content isolated from the instruction channel before the agent processes it?
- Is there a documented isolation policy specifying which content sources are treated as untrusted?

**Tier 2 (all Tier 1 items plus):**
- Has the isolation policy been tested against injection scenarios from web pages, tickets, emails, logs, and documents?
- Do injection test results show that injected content does not influence tool invocation parameters?
- Are memory hygiene rules defined and enforced for durable memory writes?

**Tier 3 (all Tier 2 items plus):**
- Are lockdown triggers wired to enforced stop on injection signal detection?
- Has lockdown behaviour been verified in the harness under active injection scenarios?
- Does the agent enter read-only mode and request human review on lockdown, without executing any further write actions?

**Tier 4 (all Tier 3 items plus):**
- Is cross-agent message verification implemented as described in Section 6.7.1?
- Are inbound delegate messages treated as untrusted data unless signed by the control plane?
- Is quorum independence verified by the harness, confirming that quorum fails when delegates share an untrusted context source?

## C.6 Dimension 4: Observability

**Tier 1:**
- Does every retrieval action emit an audit event with the required fields from Section 7.1?
- Are retrieval sources logged and auditable?

**Tier 2 (all Tier 1 items plus):**
- Does every write action audit event include the evidence record identifier?
- Is audit completeness checked continuously, with alerts when required fields are absent?
- Are plans logged at proposal time, not only at execution time?
- Is tool I/O logged at invocation time with parameter hashes?

**Tier 3 (all Tier 2 items plus):**
- Does every privileged action audit event include the approval identifier?
- Are all baseline anomaly signals from Section 7.3 instrumented?
- Are lockdown-trigger signals wired to enforced stop, not dashboard alerts only?
- Is approval fatigue instrumented via per-reviewer median review time and reversal rate?

**Tier 4 (all Tier 3 items plus):**
- Do delegate agent audit events include parent task identifier and plan reference?
- Are delegate agent audit events forwarded to a central audit store that no individual delegate can modify?
- Is cross-agent evidence continuity confirmed across all delegate action audit events?

## C.7 Dimension 5: Containment and recovery

**Tier 1.** Not required.

**Tier 2:**
- Does a tested rollback procedure exist for every permitted write action class?
- Has the rollback procedure been drilled in the harness environment?
- Does the rollback procedure specify known failure conditions?
- Is RSR at or above the Tier 2 threshold anchor from Section 8.3 in harness drills?

**Tier 3 (all Tier 2 items plus):**
- Are pre-execution safety gates implemented for privileged actions?
- Do pre-execution gates include a dry run or impact preview step before committing?
- Is RSR at or above the Tier 3 threshold anchor from Section 8.3?
- Is lockdown mode verified to halt execution within TDH threshold under adversarial scenarios?

**Tier 4 (all Tier 3 items plus):**
- Do cross-system traversal caps prevent a single task from exceeding the Tier 4 BRS cap?
- Is rollback defined for delegated write actions as well as parent agent write actions?

## C.8 Dimension 6: Human oversight

**Tier 1.** Not required.

**Tier 2:**
- Are meaningful approvals required for all write actions, with target and diff visibility?
- Is an approval review queue instrumented and staffed?
- Are approvals bound to approval identifiers that are logged in audit events?

**Tier 3 (all Tier 2 items plus):**
- Are meaningful approvals required for each privileged action class?
- Do privileged action approvals require a channel separate from the conversational interface?
- Is approval fatigue monitored and routed to an oversight review queue?

**Tier 4 (all Tier 3 items plus):**
- Is quorum gating implemented for delegated privileged actions?
- Is quorum additive to, not a replacement for, Tier 3 approvals?
- Does the Tier 4 advancement gate require all four approvers as specified in Section 9.6?
