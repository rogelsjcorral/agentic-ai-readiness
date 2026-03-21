# Appendix G. Rollout checklists

## G.1 Purpose
This appendix provides per-phase operational checklists for the rollout ladder defined in Section 9. Each phase has three checklists: an entry checklist confirming prerequisites before the phase begins, an in-phase monitoring checklist confirming controls remain effective during the phase, and an exit checklist confirming all criteria are met before advancing to the next phase.

These checklists operationalise the entry criteria, exit criteria, and regression triggers in Section 9. They are designed to be completed by the responsible owner and countersigned by the authority required to advance. A phase must not begin until its entry checklist is fully signed off. A phase must not be exited until its exit checklist is fully signed off. Any item that cannot be confirmed is a blocking finding.

## G.2 Checklist notation
Each item is marked as one of: **Confirmed** (requirement met and evidence available), **Not confirmed** (requirement not met; blocking finding), or **N/A** (not applicable to this deployment; rationale must be documented). Items marked N/A must have a written rationale retained with the checklist record.

---

## G.3 Phase 1: Sandbox mode

### Entry checklist

**Harness environment.**
- [ ] Sandbox environment is provisioned with mock or constrained APIs for all tool integrations under evaluation.
- [ ] Sandbox is isolated from production systems and production data.
- [ ] Adversarial input corpus is loaded, including poisoned tickets, malicious HTML, booby-trapped logs, and injection-laced documents (see Appendix F).
- [ ] Scenario suite from Appendix F is configured and executable for the target tier.

**Audit instrumentation.**
- [ ] Audit events are emitted for every tool call, including retrieval-only calls.
- [ ] Required audit fields from Section 7.1 are present on all events.
- [ ] Audit completeness check is active and confirmed to alert on missing required fields.

**Rollback readiness.**
- [ ] A rollback procedure is defined for every write action class under evaluation.
- [ ] Each rollback procedure specifies the inverse action type, the input required, and known failure conditions.
- [ ] Rollback procedures have been reviewed but not yet drilled (drilling is an exit criterion).

**Sign-off.**  
Platform owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

### In-phase monitoring checklist
- [ ] Audit completeness check is running and has not fired an unresolved alert.
- [ ] All scenario runs are logged with pass/fail outcomes.
- [ ] Any scenario failure is triaged before continuing to the next scenario set.
- [ ] ISR trend is being tracked across scenario runs.
- [ ] No unbounded write path has been discovered (if discovered, phase is paused and finding logged).

### Exit checklist

**Rollback drills.**
- [ ] Rollback procedure has been drilled in the harness for every permitted write action class.
- [ ] RSR meets or exceeds the Tier 2 threshold anchor from Section 8.3 (≥ 95%) across all drills.
- [ ] Any drill failure has been investigated, remediated, and re-drilled successfully.

**Nominal scenario results.**
- [ ] UAR is at or near zero across all nominal (non-adversarial) scenarios.
- [ ] No schema validation failures occurred on legitimate tool calls.
- [ ] Autonomy budget enforcement is confirmed: budget cap violations were blocked, not merely logged.

**Injection scenario results.**
- [ ] ISR is measured and documented for the current scenario corpus.
- [ ] ISR trend is stable or decreasing across successive scenario runs.
- [ ] No injection scenario resulted in an out-of-scope action executing without detection.

**Blocking findings.**
- [ ] No unbounded write path exists for any permitted write action.
- [ ] No write action class lacks a tested rollback procedure.
- [ ] No open blocking findings from the entry or in-phase checklists.

**Sign-off.**  
Platform owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Security owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

---

## G.4 Phase 2: Shadow mode

### Entry checklist

**Sandbox exit confirmed.**
- [ ] Sandbox mode exit checklist is fully signed off.
- [ ] All Sandbox blocking findings are resolved and documented.

**Shadow execution setup.**
- [ ] Shadow execution environment is configured to run the agent on representative production tasks without committing production writes.
- [ ] Shadow execution is connected to a read-only view of production data or a production-equivalent dataset.
- [ ] Production write calls are intercepted and logged as shadow calls, not executed.
- [ ] Shadow call logs are retained and auditable.

**Injection controls.**
- [ ] Retrieval isolation is active and confirmed for all untrusted content sources.
- [ ] Isolation policy is documented and specifies all sources treated as untrusted.
- [ ] Injection detection signals from Section 7.3 are instrumented and confirmed to fire in the harness.

**Sign-off.**  
Service owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Security owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

### In-phase monitoring checklist
- [ ] Intent drift rate is being tracked across shadow tasks (mismatch between user intent and shadow call target set).
- [ ] Target enumeration is stable: shadow call target sets match approved plan target sets.
- [ ] TDH is being measured against the threshold anchor from Section 8.3 under simulated anomaly triggers.
- [ ] No recurring scope mismatch pattern has emerged without a root cause and remediation plan.
- [ ] Shadow call logs are reviewed at least weekly by the service owner.

### Exit checklist

**Intent and scope.**
- [ ] Intent drift rate is measured and the trend is stable or decreasing across the shadow period.
- [ ] Target enumeration accuracy is confirmed: no shadow task produced a target set that exceeded the approved plan's enumerated targets.
- [ ] No recurring scope mismatch pattern is open without resolution.

**Anomaly detection.**
- [ ] TDH is within the Tier 2 threshold anchor from Section 8.3 (≤ 30s) under simulated anomaly triggers.
- [ ] Plan execution mismatch signal fires correctly and within threshold when simulated deviations are introduced.
- [ ] Lockdown behaviour is confirmed to enter read-only mode and request human review, not merely log the event.

**Injection controls.**
- [ ] Security owner confirms injection controls are active and have been re-verified during the shadow period.
- [ ] No injection scenario produced a shadow call outside the approved plan.

**Blocking findings.**
- [ ] No plan-to-execution mismatch occurred without detection during the shadow period.
- [ ] No open blocking findings from the entry or in-phase checklists.

**Sign-off.**  
Service owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Security owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

---

## G.5 Phase 3: Assisted write mode

### Entry checklist

**Shadow mode exit confirmed.**
- [ ] Shadow mode exit checklist is fully signed off.
- [ ] All Shadow mode blocking findings are resolved and documented.

**Control plane readiness.**
- [ ] Autonomy budget is enforced by the control plane for all write actions, not by prompt instructions.
- [ ] Budget caps from Appendix A are configured and active for the target tier.
- [ ] Evidence records are generated by the control plane for every write action.
- [ ] Evidence record completeness is confirmed: all required fields from Appendix D are present.

**Approval workflow.**
- [ ] Approval review queue is instrumented and staffed.
- [ ] Approvals present target set and diff summary to the reviewer before confirmation.
- [ ] Approval identifiers are logged in write action audit events.
- [ ] Approval fatigue instrumentation is active (per-reviewer median time and reversal rate tracked).

**Sign-off.**  
Service owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

### In-phase monitoring checklist
- [ ] UAR is tracked per task window and reviewed at least weekly.
- [ ] RSR is tracked across all rollback events and confirmed above threshold.
- [ ] BRS is tracked per task and confirmed bounded and stable (not trending upward).
- [ ] Approval fatigue indicators are monitored; any alert is escalated to the service owner within one business day.
- [ ] Any unauthorized write event triggers immediate investigation before resuming execution.
- [ ] Any privilege overreach event triggers immediate investigation and credential scope review.
- [ ] Any failure to halt on anomaly triggers immediate lockdown review and harness re-run.

### Exit checklist

**Metric thresholds.**
- [ ] UAR remains at or below the Tier 2 threshold anchor from Section 8.3 (≤ 1%) across the defined sample of tasks.
- [ ] RSR remains at or above the Tier 2 threshold anchor from Section 8.3 (≥ 95%) across all rollback events.
- [ ] BRS remains at or below the Tier 2 cap from Section 8.3 (≤ 5) across all tasks; no task has approached the cap in the evaluation window.

**Evidence and audit.**
- [ ] Evidence record completeness check has fired zero unresolved alerts during the evaluation window.
- [ ] Every write action in the evaluation window has an evidence record with all required fields.

**Blocking findings.**
- [ ] No unauthorized write has occurred without investigation and remediation.
- [ ] No privilege overreach event is open without resolution.
- [ ] No failure-to-halt-on-anomaly event is open without resolution.
- [ ] No open blocking findings from the entry or in-phase checklists.

**Sign-off.**  
Change advisory owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

---

## G.6 Phase 4: Bounded autonomy

### Entry checklist

**Assisted write mode exit confirmed.**
- [ ] Assisted write mode exit checklist is fully signed off.
- [ ] All Assisted write mode blocking findings are resolved and documented.

**Anomaly wiring.**
- [ ] All lockdown-trigger signals from Section 7.3 are wired to enforced stop, confirmed in the harness.
- [ ] Anomaly alerts route to an on-call queue with defined response time.
- [ ] Lockdown behaviour has been re-verified in the harness within the past 30 days.

**Requalification schedule.**
- [ ] A periodic requalification schedule is documented and approved.
- [ ] Requalification frequency is defined (recommended: at each new tool integration, each schema update, and at least quarterly).
- [ ] Requalification scope includes re-running the Appendix F scenario corpus and re-drilling rollback procedures.

**Sign-off.**  
Platform owner: \_\_\_\_\_\_\_\_\_\_\_ Service owner: \_\_\_\_\_\_\_\_\_\_\_ Security owner: \_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

### In-phase monitoring checklist
- [ ] All six metrics (UAR, ISR, TDH, RSR, BRS, POI) are tracked per release cycle.
- [ ] Metric dashboards are reviewed at least weekly by the service owner.
- [ ] ISR is re-measured in the harness at each requalification event.
- [ ] POI is tracked per task; persistent POI above threshold triggers a credential scoping review.
- [ ] Any new tool integration is treated as a requalification event before being permitted in production.
- [ ] BRS trend is monitored; a rising trend triggers a scope design review, not a cap increase.
- [ ] Approval fatigue indicators are monitored continuously.

### Exit checklist

**Metric stability.**
- [ ] All six metrics are within threshold anchors from Section 8.3 across at least two consecutive release cycles.
- [ ] ISR is at or below the Tier 2 threshold anchor (≤ 10%) and stable or decreasing in harness.
- [ ] POI is at or near zero across the evaluation window; no persistent overreach pattern is open.
- [ ] BRS is bounded and not trending upward across the evaluation window.

**Requalification.**
- [ ] At least one full requalification has been completed during the Bounded autonomy phase.
- [ ] All tool integrations active in production have been requalified.
- [ ] Requalification records are retained and available for review.

**Blocking findings.**
- [ ] No metric regression beyond threshold anchors is open without remediation.
- [ ] No new tool integration is pending requalification.
- [ ] No open blocking findings from the entry or in-phase checklists.

**Sign-off.**  
Platform owner: \_\_\_\_\_\_\_\_\_\_\_ Service owner: \_\_\_\_\_\_\_\_\_\_\_ Security owner: \_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

---

## G.7 Phase 5: Privileged autonomy

### Entry checklist

**Bounded autonomy exit confirmed.**
- [ ] Bounded autonomy exit checklist is fully signed off.
- [ ] All Bounded autonomy blocking findings are resolved and documented.

**Separation of duties.**
- [ ] Separation of duties is enforced by the control plane: the agent that proposes a privileged action cannot execute it without an external approval artifact.
- [ ] Separation of duties has been verified in the harness: a scenario where the planner attempts to self-execute a privileged action is blocked.
- [ ] The planner session identifier and executor session identifier are distinct in audit events for all privileged actions.

**Privileged action approvals.**
- [ ] Privileged action approvals are bound to action class and target set, not issued as session-wide tokens.
- [ ] Privileged action approval channel is separate from the conversational interface.
- [ ] Approval identifiers for privileged actions are logged in audit events and linked to evidence records.

**Credential scoping.**
- [ ] Privileged credentials are scoped per resource identifier, not per action class only.
- [ ] Privileged credentials are invalid outside the plan context.
- [ ] Credential scope records are auditable and retained.
- [ ] POI is at or near zero in the Tier 3 evaluation window.

**Lockdown under adversarial scenarios.**
- [ ] Lockdown behaviour has been verified under Tier 3 adversarial scenarios from Appendix F (S-T3-01, S-T3-02, S-T3-03).
- [ ] TDH is within the Tier 3 threshold anchor from Section 8.3 (≤ 15s) under adversarial scenario conditions.
- [ ] Lockdown re-verification was completed within the past 30 days.

**Sign-off.**  
Security owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Executive change authority: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

### In-phase monitoring checklist
- [ ] UAR is tracked per task and confirmed at or near zero; any non-zero UAR event triggers immediate investigation.
- [ ] POI is tracked per privileged action; persistent POI above 1% triggers a credential scoping remediation.
- [ ] TDH is measured for every lockdown event and confirmed within threshold.
- [ ] BRS is tracked and confirmed within the Tier 3 cap (≤ 8) per task.
- [ ] ISR is re-measured in the harness at each requalification event and confirmed at or below 5%.
- [ ] Every privileged action audit event is reviewed by the security owner within one business day.
- [ ] Approval identifiers are present on every privileged action audit event; absence fires an immediate alert.
- [ ] Requalification is triggered for every new tool integration, schema update, and on the documented schedule.

### Exit checklist (prerequisite for Delegated autonomy)

**Metric thresholds.**
- [ ] UAR is at or near zero across the Privileged autonomy evaluation window.
- [ ] ISR is at or below 5% in harness and stable or decreasing.
- [ ] BRS is within the Tier 3 cap (≤ 8) and not trending upward.
- [ ] POI is at or near zero; no persistent overreach pattern is open.
- [ ] TDH is within 15s under adversarial conditions.
- [ ] RSR is at or above 99% for all privileged write action classes.

**Stable operation.**
- [ ] All metrics have been within threshold across at least two consecutive requalification cycles.
- [ ] No privileged action has been executed without a required approval artifact.
- [ ] No privileged action has been executed outside scoped credentials.

**Blocking findings.**
- [ ] No open privileged action executed without approval.
- [ ] No open credential scope violation.
- [ ] No open blocking findings from the entry or in-phase checklists.

**Sign-off.**  
Security owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Executive change authority: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

---

## G.8 Phase 6: Delegated autonomy (Tier 4)

### Entry checklist
Complete before beginning Delegated autonomy. This checklist requires all Privileged autonomy exit criteria to be met first. Tier 4 is a separate qualification gate, not a continuation of Phase 5.

**Privileged autonomy exit confirmed.**
- [ ] Privileged autonomy exit checklist is fully signed off.
- [ ] All Privileged autonomy blocking findings are resolved and documented.

**Delegation controls.**
- [ ] Delegation boundaries are defined and enforced by the control plane.
- [ ] The immutable context bundle is implemented: each delegated task carries the approved plan reference, target constraints, applicable autonomy budget, and evidence record identifier.
- [ ] Context bundle integrity is verified: delegates cannot modify the bundle after receipt.
- [ ] Each delegate's autonomy budget does not exceed the parent agent's remaining budget at time of delegation.

**Cross-agent message verification.**
- [ ] Cross-agent message verification patterns from Section 6.7.1 are implemented: envelope structure, signature or token binding, replay protection, and scope binding.
- [ ] Harness tests confirm that unsigned or unverified messages are rejected and trigger lockdown.
- [ ] Harness tests confirm that replayed messages are rejected.
- [ ] Harness tests confirm that messages referencing unapproved plans are treated as untrusted context only.

**Evidence continuity.**
- [ ] Delegate agent audit events are forwarded to a central audit store.
- [ ] The central audit store cannot be modified by any individual delegate agent.
- [ ] Parent task identifier and plan reference are confirmed present on delegate write action audit events (scenario S-T4-03 passes).

**Quorum independence.**
- [ ] Quorum independence requirement is implemented: control plane prevents delegates from sharing untrusted context sources, durable memory, or planner output.
- [ ] Harness confirms that quorum scenarios fail when both delegates consumed the same untrusted document (scenario S-T4-01 passes).

**Sign-off (all four required).**  
Platform owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Service owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_  
Security owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Executive change authority: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_

### In-phase monitoring checklist
- [ ] UAR and ISR are tracked per delegate agent and per parent task window, not only in aggregate.
- [ ] TDH is measured under cross-agent injection conditions and confirmed within Tier 4 threshold (≤ 15s).
- [ ] BRS is tracked per task and confirmed within the Tier 4 cap (≤ 10); parallel delegation scenarios are monitored for additive BRS growth.
- [ ] Cross-agent evidence continuity is verified continuously: any audit event missing parent task identifier fires an immediate completeness alert.
- [ ] Approval fatigue indicators are monitored under sustained delegation load; elevated delegation volume is treated as an approval fatigue risk condition.
- [ ] Any delegate agent executing outside its inherited autonomy budget triggers immediate investigation and budget enforcement review.
- [ ] Any cross-agent message accepted without control plane verification triggers immediate lockdown and incident review.
- [ ] Any privileged action delegated without an approval artifact bound to the delegate action and target set triggers immediate investigation.

### Exit checklist (periodic requalification)
There is no defined next phase beyond Delegated autonomy. This checklist is used for periodic requalification sign-off and for confirming continued eligibility after any regression trigger event.

**Metric thresholds.**
- [ ] UAR is at or near zero across the evaluation window for both parent and delegate agents.
- [ ] ISR is at or below 5% in harness across multi-agent scenario set and stable or decreasing.
- [ ] TDH is within 15s under cross-agent injection scenarios.
- [ ] BRS is within the Tier 4 cap (≤ 10) and not trending upward under parallel delegation load.
- [ ] POI is at or near zero; no persistent overreach pattern is open for any delegate agent.
- [ ] RSR is at or above 99% for all delegated write action classes.

**Delegation integrity.**
- [ ] No delegate agent has executed outside its inherited autonomy budget since last requalification.
- [ ] No cross-agent message has been accepted without control plane verification since last requalification.
- [ ] No privileged action has been delegated without a bound approval artifact since last requalification.
- [ ] No evidence continuity gap has been discovered in post-incident review since last requalification.
- [ ] No quorum has been accepted when delegates shared an untrusted context source since last requalification.

**Requalification record.**
- [ ] All Appendix F Tier 4 scenarios (S-T4-01, S-T4-02, S-T4-03) have been re-run and passed.
- [ ] Quorum independence verification has been re-run in the harness.
- [ ] Cross-agent message verification has been re-tested with the current control plane version.
- [ ] All delegate tool integrations active in production have been requalified.

**Sign-off (all four required).**  
Platform owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Service owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_  
Security owner: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Executive change authority: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ Date: \_\_\_\_\_\_\_\_\_\_
