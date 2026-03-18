# Operational Readiness Criteria for Tool-Using LLM Agents
## Controls, Metrics, and Rollout Gates for Delegated Autonomy

Rogel S.J. Corral  
Independent Researcher  
Draft v0.6  
Status: For Community Review  
Date: 2026-03-17

**Abstract**  
Agentic AI changes the deployment shape of LLM systems. When an LLM can plan, call tools, and execute multi step actions, the primary risk shifts from incorrect text to incorrect state change. This field manual proposes an operational readiness approach for deploying tool using agents, including capability tiers, an autonomy budget, a readiness scorecard, and an evaluation harness with measurable metrics. The goal is not to argue for or against agents. The goal is to define a minimum operational bar for delegated autonomy and a rollout approach that reduces preventable incidents.

**Document type and intent**  
This document is written as an operational field manual. It provides deployment readiness criteria, control patterns, and evaluation methods for tool using LLM agents. It is designed to be applied in production environments and to support rollout gating, auditability, and recovery planning.

**Intended audience**  
This document is intended for engineering teams responsible for deploying and operating tool using agents, including platform engineering, security engineering, SRE and operations teams, and product teams shipping agents with write capable tools.

**Assumptions**  
This document assumes the agent can call tools that interact with external systems. It assumes that some actions can change state and that at least some deployments involve privileged or high impact workflows.

**How to use this document**  
Use Sections 4 through 9 as an implementation sequence. Identify relevant failure modes, apply the readiness scorecard, implement the control patterns, instrument audit events, validate with the evaluation harness, then roll out by phase using the rollout ladder. Treat Tier 2 and above as ineligible until rollback and evidence capture requirements are met.

**Normative language**  
This document uses plain requirement language for deployability. When it states that a capability requires a control, it indicates a recommended deployment gate for that capability tier.

**Statement of scope**  
This document is an operations focused guide for teams deploying tool using agents in real systems. It prioritizes concrete controls, measurable gates, and recovery requirements. It is not a vendor comparison, not a policy manifesto, and not a claim that agentic AI is inherently unsafe. It treats agentic AI as a capability upgrade with an expanded attack surface.

---

## 1. Definitions and scope

### 1.1 Definitions
Agent. An LLM driven system that can plan and execute actions via tools across multiple steps, with some persistent state in the form of session state or durable memory, under delegated authority.  
Tool. A callable interface that can read from or write to an external system, including an API, database, file store, SaaS application, or internal service.  
Action. A tool invocation that creates observable effects in an external system.  
Write action. An action that creates, modifies, deletes, transfers, publishes, grants access, revokes access, or otherwise changes state.  
Privileged action. A write action with high impact, such as changing identity and access controls, moving money, modifying production infrastructure, deleting records, or altering audit logs.  
Approval. A human confirmation step that authorizes a proposed action plan or a specific action, tied to an auditable approval identifier.  
Evidence record. A minimal artifact set that explains why an action was taken, what inputs were used, what was approved, and what outcome occurred.  
Rollback. A defined method to revert a write action, including a tested procedure and known failure conditions.  
Session state. Short lived working context used during a single run.  
Durable memory. State that persists across sessions, such as notes, profiles, preferences, cached facts, or stored plans.

### 1.2 Agent shapes in scope
This document considers three common agent deployment shapes. In a single agent tool user shape, one model both plans and executes tool calls. In a planner executor split, a planner proposes actions and an executor performs constrained tool calls. In multi agent orchestration, multiple agents coordinate, delegate, or vote on plans and actions. These shapes differ in enforceability, audit complexity, and blast radius, and therefore affect readiness requirements.

### 1.3 Deployment environments in scope
This document addresses agents used in internal enterprise automation, customer facing workflows with tool access, and administrative or security sensitive workflows, including identity, finance, and infrastructure.

### 1.4 Non goals
This document does not propose new model architectures or training methods. It does not claim formal proof of safety. It does not make universal policy claims about social impact.

---

## 2. Why agents change the operational risk model

### 2.1 From answers to actions
In a chat system, the primary failure is incorrect text. In an agent system, outputs can become actions via tools and credentials. The primary failure mode becomes incorrect state change. This matters because state change can be irreversible, expensive, or silently harmful. The operational question is therefore not only whether a response is correct, but whether a proposed action is authorized, bounded in scope, reversible, and auditable.

### 2.2 Compounding risk through multi step execution
Agent systems commonly operate in loops. Looping introduces compounding error in two ways. First, small interpretation errors can cascade into multiple tool calls. Second, the agent can generate new context from its own actions and then use that context to justify further actions. This increases blast radius even when each individual step appears reasonable in isolation.

### 2.3 Retrieval and memory as attack surfaces
Tool outputs can contain untrusted content, including web pages, emails, tickets, logs, and documents. Durable memory can persist incorrect or adversarial inputs across sessions. In both cases, an agent can treat untrusted data as instructions unless explicit isolation, validation, and execution gating controls exist.

### 2.4 Core principle
Autonomy must be earned by controls and measurement, not assumed from model capability.

---

## 3. Capability tiers and the autonomy budget

### 3.1 Capability tiers
Tier 0. No tools, advice only. The system provides guidance with no external effects.  
Tier 1. Read only tools. The system can retrieve information, such as search or knowledge base lookup, but cannot write or change state.  
Tier 2. Write tools in bounded systems. The system can perform low impact writes with defined rollback and clear approvals, such as creating tickets, drafting emails, opening pull requests, or updating non critical records.  
Tier 3. Privileged actions. The system can perform high impact changes, such as modifying access controls, moving funds, changing production configuration, or deleting records.  
Tier 4. Cross system autonomy and multi agent delegation. The system coordinates across multiple services and delegates tasks among agents. Tier 4 increases blast radius through delegation and parallelism. Operational readiness at Tier 4 requires explicit delegation boundaries, a shared evidence chain across agents, and enforceable caps on cross system traversal and parallel tool execution. An agent may not delegate privileged actions unless the delegate inherits the same autonomy budget and approval artifacts.

### 3.2 Autonomy budget
An autonomy budget is a configurable set of caps that limits what an agent can do before escalation. Budgets should be tier specific and enforced by the control plane, not by prompts. The purpose of the autonomy budget is to bound blast radius when errors occur, including errors induced by ambiguity, prompt injection, memory poisoning, or tool misuse.

Suggested starting caps are provided in Appendix A as calibration anchors. These values are not universal. They are starting anchors intended to prevent unbounded execution during early rollout.

### 3.3 Budget escalation rules
Budgets require deterministic escalation triggers. Approval is required when a plan includes any write action above defined thresholds. Lockdown mode is required when injection signals or anomalies occur, switching the agent to read only behavior while requesting human review. Human takeover is required when rollback is unavailable or when privileged actions are requested.

### 3.4 Deliverable
Appendix A provides an autonomy budget template that teams can adopt per tier.

---

## 4. Failure modes that matter in production

### 4.1 Intent drift and scope mismatch
An agent can produce actions that are internally coherent yet misaligned with operator intent. This often occurs when the input request is underspecified, when target selection is inferred, or when a reasonable default is applied to a privileged scope. In practice, this failure mode is dangerous because the action can look correct in a quick skim while still targeting the wrong resources. The operational signature is a mismatch between the user's stated intent and the concrete target set, change set, or impact surface.

### 4.2 Prompt injection through untrusted tool outputs
When tool outputs contain untrusted content, an agent may treat that content as instructions rather than data. The common carriers are web pages, tickets, emails, logs, and documents. This is not a hypothetical edge case. Recent incidents show prompt injection being used as an execution lever in agentic workflows and in supply chain adjacent scenarios. Operationally, this manifests as tool calls that were not part of the user's request, or plan deviations that correlate with recently retrieved content.

### 4.3 Over-privilege and credential mis-scoping
If an agent is given broad API tokens or long lived credentials, it can perform actions that were never intended, especially when combined with injection or intent drift. The root cause is typically convenience, a lack of tiering, or a missing control plane that can mint least privilege, short lived tokens per action class. The operational signature is privilege overreach, where the agent uses capabilities it did not strictly need for the task.

### 4.4 Cascading actions and self-reinforcing context
Multi step execution can create a feedback loop. An initial mistake produces a state change, which becomes new context, which then justifies further actions. The blast radius expands even if each step looks locally rational. The operational signature is action sequences whose justification references the agent's own prior actions rather than an external ticket, approval record, or validated intent.

### 4.5 Memory poisoning and durable state corruption
If the agent writes to durable memory, it can store incorrect or adversarial content that persists across sessions. This converts a single failure into a recurring failure pattern. The operational signature is repeated, stable misbehavior across tasks that share no legitimate common cause.

### 4.6 Approval fatigue and weak human gating
Human in the loop can degrade into human in the way when approvals become repetitive, poorly summarized, or decoupled from concrete diffs. In those conditions, approvals become a throughput mechanism rather than a control. Real incidents of AI-enabled fraud underscore that humans can be socially engineered even when they believe they are following procedure. The operational signature is high approval velocity with low review quality, and approvals occurring without target visibility, diff visibility, or rollback awareness.

---

## 5. Readiness scorecard
This section defines a minimum operational bar by capability tier. Readiness is assessed across six dimensions. The scorecard is intentionally deployable. It is designed to gate rollouts, not to decorate slide decks.

### 5.1 Dimensions
Authority control. Least privilege, short lived credentials, separation of duties for high impact changes.  
Intent verification. Structured plans, explicit target enumeration, confirmation gates tied to actions.  
Adversarial resilience. Isolation of untrusted content, injection defenses, memory hygiene.  
Observability. Audit events, traceability from request to action, replay support where feasible.  
Containment and recovery. Sandboxes, dry runs, rollback procedures, lockdown behavior.  
Human oversight. Escalation rules, review queues, and meaningful approvals.

### 5.2 Scorecard format (operator checklist)
Each dimension is assessed as Pass, Conditional, or Fail for the target capability tier. A tier is eligible only if all required dimensions meet the minimum bar.

| Dimension | Evidence artifact | Tier 1 minimum | Tier 2 minimum | Tier 3 minimum | Tier 4 minimum |
|---|---|---|---|---|---|
| Authority control | Credential scope record, token TTL, privilege mapping | Scoped read access | Scoped write access by action type | Least privilege per tool, per action, per resource; separation of duties | Same as Tier 3 plus delegation boundary enforcement |
| Intent verification | Approved plan, enumerated targets, diff summary | N/A | Required for all writes | Required per privileged action class | Required per delegate action and target set |
| Adversarial resilience | Isolation policy, injection tests, memory rules | Retrieval isolation | Isolation plus injection scenario tests | Isolation plus lockdown triggers wired to stop | Same as Tier 3 plus cross agent verification |
| Observability | Audit events, trace IDs, plan to action mapping | Retrieval audit | Full action audit for writes | Full action audit plus anomaly alerts | Evidence continuity across agents |
| Containment and recovery | Dry run evidence, rollback playbook, drills | N/A | Rollback required for every write | Rollback plus pre execution gates | Same as Tier 3 plus cross system traversal caps |
| Human oversight | Approval records, review queue | N/A | Meaningful approvals for writes | Meaningful approvals for privileged actions | Quorum or additional safeguards are additive only |

### 5.3 Minimum bar per tier
Tier 1 requires isolation of retrieved content and auditability of retrieval sources. Tier 2 requires intent verification, evidence records, and tested rollback for every permitted write action. Tier 3 requires separation of duties, strict credential scoping, explicit approvals for each privileged action class, and continuous monitoring with lockdown triggers. Tier 4 requires all Tier 3 controls plus explicit delegation boundaries, cross system traversal caps, and evidence continuity across agents.

### 5.4 Pass or fail rules
If rollback does not exist for a write action, that action is not eligible for Tier 2 automation.  
If credentials cannot be scoped by tool, action type, and resource, Tier 3 is not eligible.  
If approvals do not expose targets and diffs, approvals are not controls and cannot be used to justify Tier 2 or Tier 3 readiness.  
If untrusted tool outputs can influence tool invocation parameters without isolation, Tier 2 and above is not eligible.

---

## 6. Control patterns that actually work

### 6.1 Tool allowlisting and action schemas
Tool access must be explicit. Each tool must have an allowlisted set of action types. Each action type must have typed parameters and explicit validation. Freeform tool execution is incompatible with Tier 2 and above.

Enforcement. Tool execution is mediated by a control plane that exposes only allowlisted tools and action types. Each action type has a schema that validates targets, parameters, and thresholds before execution. Schema validation failures are treated as hard stops. Tool calls carry a plan reference and an approval identifier when required.

Limitations. Schemas do not prevent a harmful plan from being proposed. They prevent execution of out of scope actions. Schemas must be paired with intent verification, target enumeration, and rollback readiness. Schema updates should be treated as requalification events.

### 6.2 Planner executor separation
A planner executor split reduces the attack surface by isolating natural language planning from constrained execution. The planner can propose, but the executor should only accept schema valid actions within an autonomy budget. This pattern also improves audit quality because the plan becomes a first class artifact.

Enforcement. The planner produces a structured plan with explicit targets and action types. The executor refuses tool calls unless they match the approved plan, match the schema, and remain within the autonomy budget. Executor behavior is deterministic with respect to policy checks.

Limitations. Separation reduces the chance that untrusted content becomes executable actions, but does not eliminate injection into planning text. It must be paired with isolation of retrieved content and meaningful approvals for write actions.

### 6.3 Two channel confirmation for high impact changes
For destructive or irreversible actions, confirmation should be separated from the conversational channel. At minimum, the confirmation step must restate targets, summarize diffs, and bind to an approval identifier. This is a control against both intent drift and injection.

Enforcement. The control plane generates a confirmation artifact that includes the enumerated target set, the action class, and a diff summary. The artifact is signed or otherwise bound to an approval identifier. Tool execution is blocked until the approval identifier is presented.

Limitations. Confirmation is only as good as the diff visibility and the reviewer. Approval fatigue must be monitored. Confirmation should be paired with anomaly detection and rollback readiness.

### 6.4 Least privilege and short lived credentials
Credentials should be minted per task, scoped per tool and per action class, with short lifetimes. Tier 3 requires separation of duties for privileged actions, where the agent cannot both propose and execute without an external approval artifact.

Enforcement. The control plane mints short lived tokens scoped to the specific tool, action type, and resource identifiers in the approved plan. Privileged credentials are issued only after required approvals and are invalid outside the plan context.

Limitations. Least privilege reduces blast radius but does not prevent errors within scope. Token issuance logic becomes part of the trusted computing base and must be auditable.

### 6.5 Lockdown mode as a first class behavior
When anomaly signals trigger, the agent should default to read only behavior, stop tool execution, and request human review. Lockdown is not a user interface feature. It is an operational control that prevents cascading damage during suspected compromise.

Enforcement. Anomaly signals are wired to enforced stop. The control plane blocks write-capable tools and forces the agent into read only behavior for the affected task. The stop condition is recorded in audit events.

Limitations. Lockdown requires well chosen triggers. Overly sensitive triggers cause frequent halts; weak triggers allow damage. Triggers should be validated in the evaluation harness.

### 6.6 Evidence records for every write
Tier 2 and above requires a minimal evidence record tying together intent, enumerated targets, approvals, tool calls, and outcomes. This is necessary for accountability and post incident reconstruction. It also prevents untraceable automation from becoming normal practice.

Enforcement. The control plane emits a per task evidence record ID and requires all tool calls to include it. Evidence records include plan reference, targets, approvals, tool I/O identifiers, outcomes, and rollback identifiers when available.

Limitations. Evidence records do not prevent bad changes. They make bad changes discoverable and recoverable. Evidence completeness should be an acceptance gate.

### 6.7 Multi agent delegation controls (Tier 4)
Delegation must be explicit and constrained. Each delegated task must carry an immutable context bundle containing the approved plan reference, target constraints, applicable autonomy budget, and evidence record identifier. Delegates must be prevented from expanding scope beyond inherited constraints. Cross agent communication channels must treat all inbound messages as untrusted data unless signed by the control plane. For privileged actions, delegation requires separation of duties, with approvals bound to the specific delegate action type and target set.

#### 6.7.1 Cross agent communication verification patterns
Cross agent messages must be treated as untrusted data unless verified by the control plane. The objective is to prevent a compromised agent, poisoned content, or external injection from causing other agents to accept instructions outside approved constraints.

Message envelope. Each cross agent message includes a structured envelope with message identifier, sender identifier, recipient identifier, timestamp, expiration time, conversation or task identifier, approved plan reference, autonomy budget identifier, and a payload hash. The payload is structured data validated against schemas.

Signature or token binding. The envelope is bound to an authenticity mechanism. Either the control plane issues a short lived signed token to the sender for a specific task identifier and recipient set, or the control plane signs the envelope and the recipient verifies the signature using a pinned control plane public key.

Replay protection. Every envelope contains a nonce or monotonic sequence number. Recipients maintain a short lived replay cache keyed by sender and task identifier. Replays and expired envelopes are rejected.

Scope binding. Recipients validate that incoming messages reference an existing approved plan and an applicable autonomy budget. If the message cannot be bound to an approval artifact, it is treated as untrusted context only and is not translated into tool calls or plan changes.

Quorum gating for privileged actions. For delegated privileged actions, require either a human approval bound to the delegate's action and target set, or a quorum rule such as two independent delegates agreeing on the same schema valid action proposal. Quorum is additive only and does not replace approvals at Tier 3.

Independence requirement. Independence must be established operationally. Delegates must not share the same untrusted context source, the same durable memory, or the same planner output. At minimum, independence requires separate retrieval contexts and separate randomization seeds, plus a control plane rule that prevents one delegate from forwarding unverified context to another. Quorum is invalid if both delegates consumed the same untrusted document or ticket content without isolation.

Failure behavior. If verification fails, the recipient enters lockdown mode for the affected task, refuses tool execution, and emits an audit event indicating rejection reason and envelope metadata.

---

## 7. Observability and audit

### 7.1 Audit event requirements
Every tool invocation should emit an audit event. The audit record must be sufficient to reconstruct what changed, even if the model conversation is unavailable.

Required fields for every audit event: timestamp (UTC, millisecond resolution), actor identifier (agent instance and session), tool identifier, action type, target resource identifiers, parameter hashes (not raw parameter values), outcome status (success, failure, blocked, rolled back), and evidence record identifier for Tier 2 and above.

Optional but strongly recommended fields: plan reference identifier, approval identifier, rollback identifier, parent task or orchestration identifier for Tier 4, anomaly signal flags triggered during execution, and latency from plan proposal to execution.

The distinction between required and optional is operational: required fields are the minimum needed for post incident reconstruction without access to the model conversation. Optional fields increase the fidelity of reconstruction and are required to compute TDH, POI, and cross agent evidence continuity at Tier 4.

### 7.2 Audit field completeness as an acceptance gate
Audit completeness is not a best effort property. At Tier 2 and above, evidence record identifier must be present on every write action audit event. At Tier 3, approval identifier must be present on every privileged action audit event. At Tier 4, parent task identifier and plan reference must be present to support cross agent evidence continuity.

Missing required fields should be treated as a blocking finding in the rollout scorecard, equivalent to a failed pass/fail rule in Section 5.4. Teams should instrument an audit completeness check that runs continuously and alerts when required fields are absent.

### 7.3 Alerts and anomaly signals
The following are baseline anomaly signals that should be instrumented before Tier 2 rollout. At Tier 3, every signal marked as lockdown trigger must be wired to enforced stop, not merely a dashboard alert.

Action burst: tool call rate exceeds the autonomy budget cap within a task window. Lockdown trigger at Tier 3.  
Novel targets: action targets resources outside the approved plan's enumerated target set. Lockdown trigger at Tier 3 and 4.  
Cross system hop growth: the count of unique external systems touched within a task window increases beyond the budget cap. Lockdown trigger at Tier 3 and 4.  
Privilege escalation attempt: a tool call requests a permission scope not present in the minted credential set. Lockdown trigger at all tiers above 1.  
Plan execution mismatch: executed tool calls deviate from the approved plan in action type, target, or parameter hash. Lockdown trigger at Tier 2 and above.  
Repeated tool failures: consecutive failures on the same tool exceed a threshold, indicating possible injection or state corruption. Lockdown trigger at Tier 2 and above.  
Approval fatigue indicators: median approval review time drops below a threshold, or approval reversal rate falls to near zero across a review window. These are signals of degraded human oversight quality, not injection, and should route to a human oversight review queue rather than automatic lockdown.

The approval fatigue signals require instrumentation beyond the audit event stream. Teams should track per-reviewer median time from approval presentation to confirmation, and track the rate at which approved plans are later reversed or escalated. A sudden drop in review time or a sustained period of zero reversals in a high volume approval queue are operational signatures of approval fatigue as described in Section 4.6.

### 7.4 Post incident replay
Deterministic replay is not always possible, but post incident reconstruction should be. The practical minimum is: plans logged at proposal time, tool I/O logged at invocation time, and approval artifacts retained for the duration of the evidence retention policy.

### 7.5 Audit retention and access controls
Audit events and evidence records are themselves a high value target. An agent with write access to its own audit log can suppress evidence of its actions. Access control requirements for audit storage: write access to audit events must be restricted to the control plane, not to the agent runtime; the agent runtime must be unable to modify or delete audit events it has emitted; privileged action audit events must require a separate approval to delete or amend; and at Tier 4, audit events from delegate agents must be forwarded to a central audit store that no individual delegate can modify.

Retention period is environment specific, but audit events for privileged actions should be retained for no less than the organization's incident response window, plus the time required for any regulatory reporting obligation. Evidence records for completed tasks should be retained for the same period.

---

## 8. Evaluation harness and metrics

### 8.1 Harness design
A minimal harness should provide a sandbox environment with mock or constrained APIs for representative tools, plus a corpus of adversarial inputs such as poisoned tickets, malicious HTML, booby trapped logs, and injection-laced documents. Acceptance testing must be scenario based and tier specific.

### 8.2 Metrics

**Unauthorized Action Rate (UAR).** Define $A_{\mathrm{exec}}$ as the count of executed tool actions in a task window $W$. Define $A_{\mathrm{unauth}}$ as the count of executed tool actions in $W$ for which no valid approval artifact exists, or for which the action exceeded the approved scope.

$$\mathrm{UAR} = \frac{A_{\mathrm{unauth}}}{\max(1, A_{\mathrm{exec}})}$$

**Injection Success Rate (ISR).** Define $N_{\mathrm{inj}}$ as the number of injection scenarios executed. Define $N_{\mathrm{succ}}$ as the number of scenarios where the agent executed an out of scope action, attempted an out of scope action, or produced a plan containing out of scope actions due to injected instructions.

$$\mathrm{ISR} = \frac{N_{\mathrm{succ}}}{\max(1, N_{\mathrm{inj}})}$$

**Time to Detect and Halt (TDH).** Define $t_{0}$ as the timestamp of the first violation indicator and $t_{\mathrm{halt}}$ as the timestamp when tool execution is blocked and lockdown is enforced.

$$\mathrm{TDH} = t_{\mathrm{halt}} - t_{0}$$

**Rollback Success Rate (RSR).** Define $W_{\mathrm{attempt}}$ as the count of rollback attempts and $W_{\mathrm{success}}$ as the count of rollbacks that restore the target state and pass verification.

$$\mathrm{RSR} = \frac{W_{\mathrm{success}}}{\max(1, W_{\mathrm{attempt}})}$$

**Blast Radius Score (BRS).** Define a task window $W$ from first tool call to stop condition. Let $T$ be the number of unique target resources modified within $W$ and $S$ the number of unique external systems touched within $W$. Define weights $\alpha$ and $\beta$ by environment criticality (default $\alpha = 1$, $\beta = 5$ in production).

$$\mathrm{BRS} = \alpha T + \beta S$$

**Privilege Overreach Index (POI).** Define $P_{\mathrm{used}}$ as the set of distinct permission scopes actually exercised and $P_{\mathrm{req}}$ as the minimum permission scope set required to execute the approved plan.

$$\mathrm{POI} = \frac{\left| P_{\mathrm{used}} \setminus P_{\mathrm{req}} \right|}{\max\left(1,\left|P_{\mathrm{req}}\right|\right)}$$

### 8.3 Suggested threshold anchors (non normative)
Metric definitions are necessary but not sufficient to gate rollouts. Teams need starting threshold values to calibrate the evaluation harness and to make pass/fail determinations concrete. The values below are non normative anchors. They are intended as starting points for sandbox and shadow mode calibration, not as universal production requirements. Teams should tighten these values as operational experience accumulates and should document any deviations from these anchors with a rationale.

| Tier | UAR target | ISR target | TDH target | RSR target | BRS cap | POI target |
|---:|---|---|---|---|---|---|
| 1 | N/A | ≤ 5% | ≤ 60s | N/A | ≤ 2 | N/A |
| 2 | ≤ 1% | ≤ 10% | ≤ 30s | ≥ 95% | ≤ 5 | ≤ 5% |
| 3 | ≤ 0.1% | ≤ 5% | ≤ 15s | ≥ 99% | ≤ 8 | ≤ 1% |
| 4 | ≤ 0.1% | ≤ 5% | ≤ 15s | ≥ 99% | ≤ 10 | ≤ 1% |

Notes on these anchors. UAR of zero is the long term target; the values above represent the maximum tolerated rate during a defined evaluation window, not an acceptable steady state. ISR is measured against the harness scenario corpus defined in Appendix F; a team with a small or undiversified scenario corpus should treat its ISR as a lower bound on real world exposure. TDH is measured from the first anomaly indicator to enforced lockdown; it is infrastructure latency, not human response time. RSR below the target threshold means Tier 2 or above is not eligible for that write action class until the rollback procedure is repaired and retested. BRS cap is a per task ceiling; tasks that routinely approach the cap should trigger a review of scope design rather than a cap increase. POI at or near zero is the expected steady state once least privilege credential issuance is working correctly; persistent POI above target indicates a credential scoping defect.

### 8.4 Acceptance criteria
Tier 2 eligibility requires stable rollback success and low unauthorized action rate under ambiguous and adversarial scenarios. Tier 3 eligibility requires strong privilege scoping and demonstrably effective lockdown triggers under injection scenarios. Teams should use the threshold anchors in Section 8.3 as the concrete numerical bar for these determinations.

---

## 9. Rollout ladder

### 9.1 Sandbox mode
Entry criteria. Harness environment available, audit events emitted for every tool call, rollback drills defined for every write action under consideration.  
Exit criteria. RSR meets target threshold in drills, UAR is near zero under nominal scenarios, ISR is measured and trending down under injection scenarios.  
Authority to advance. Platform owner and security owner sign off.  
Regression trigger. Any unbounded write path discovered, or rollback unavailable for a permitted write.

### 9.2 Shadow mode
Entry criteria. Sandbox exit criteria met. Shadow execution can be run on representative tasks without production writes.  
Exit criteria. Intent drift rate is measured and decreasing, target enumeration is stable, TDH is within threshold under simulated anomaly triggers.  
Authority to advance. Service owner approves; security owner confirms injection controls are active.  
Regression trigger. Plan to execution mismatch without detection, or recurring scope mismatch patterns.

### 9.3 Assisted write mode
Entry criteria. Shadow mode exit criteria met. Autonomy budget enforced by control plane. Evidence records generated for every write.  
Exit criteria. UAR remains below threshold across a defined sample of tasks. RSR remains above threshold. BRS remains bounded and stable.  
Authority to advance. Change advisory owner or equivalent approver group.  
Regression trigger. Any unauthorized write, privilege overreach event, or failure to halt on anomaly.

### 9.4 Bounded autonomy
Entry criteria. Assisted write metrics stable, anomaly alerts wired to enforced stop, periodic requalification schedule established.  
Exit criteria. Stable metrics across release cycles, sustained low ISR in harness, POI at or near zero.  
Authority to advance. Joint approval by platform, service owner, and security owner.  
Regression trigger. Metric regression beyond thresholds, new tool integrations without requalification, or rising BRS.

### 9.5 Privileged autonomy
Entry criteria. Separation of duties enforced, privileged approvals bound to action class and target set, credentials scoped per resource, lockdown verified under adversarial scenarios.  
Exit criteria. Demonstrated stable operation over time with near zero UAR, low ISR, bounded BRS, near zero POI, and acceptable TDH against the threshold anchors in Section 8.3.  
Authority to advance. Security owner plus executive change authority or equivalent high assurance gate.  
Regression trigger. Any privileged action executed without required approval artifacts or outside scoped credentials.

### 9.6 Delegated autonomy (Tier 4)
This phase covers agents that coordinate across multiple services or delegate tasks to other agents. Tier 4 deployment must not begin until all Privileged Autonomy exit criteria are met and the additional controls in Section 6.7 are verified in the harness environment. Tier 4 is not a continuation of the Privileged Autonomy phase; it is a separate qualification gate with distinct entry criteria and its own regression triggers.

Entry criteria. All Privileged Autonomy exit criteria met. Delegation boundaries defined and enforced by the control plane. Immutable context bundle implemented and verified. Cross agent message verification patterns from Section 6.7.1 implemented and passing harness tests. Evidence continuity across agents confirmed: audit events from delegate agents are forwarded to central audit store and parent task identifiers are present. Autonomy budget enforced per delegate, with each delegate's budget not exceeding the parent agent's remaining budget at time of delegation.  
Exit criteria. UAR and ISR at or below Tier 4 threshold anchors across a defined multi-agent scenario set. TDH within threshold under cross agent injection scenarios. BRS within Tier 4 cap under parallel delegation scenarios. Cross agent evidence continuity confirmed across all delegate action audit events. Quorum independence requirement verified: harness confirms that quorum scenarios fail when both delegates share the same untrusted context source. Approval fatigue indicators within acceptable range under sustained delegation load.  
Authority to advance. Joint approval by platform owner, service owner, security owner, and executive change authority. This gate requires all four approvers because Tier 4 adds blast radius through delegation and parallelism that no single owner can fully assess.  
Regression trigger. Any delegate agent executing outside its inherited autonomy budget, any cross agent message accepted without control plane verification, any privileged action delegated without an approval artifact bound to the delegate action and target set, evidence continuity gap discovered in post incident review, or quorum accepted when delegates shared an untrusted context source.

---

## 10. Real world anchors

### 10.1 GitLab database outage and rollback reality
GitLab's 2017 incident shows how a single mistaken destructive action can cause major data loss and how recovery depends on tested backups and practiced procedures, not on good intentions. Control implication. Tier 2 requires tested rollback for every permitted write action. Tier 3 requires additional separation of duties and pre-execution safety gates. Metric linkage. Rollback Success Rate, Time to Detect and Halt.

### 10.2 Deepfake enabled fraud and approval weakness
Arup's disclosed deepfake scam demonstrates that human approval is not automatically a safety control. Humans can be socially engineered into authorizing transfers in workflows that appear legitimate. Control implication. Approvals must be tied to concrete diffs, target visibility, and policy thresholds. High impact actions require multi factor verification beyond a single conversational confirmation. Metric linkage. Unauthorized Action Rate, approval quality measures, anomaly detection time.

### 10.3 Agentic prompt injection and tool chain compromise
Reports in early 2026 describe prompt injection being used to drive unauthorized installation or execution behaviors in agent-adjacent workflows, including supply chain style abuse patterns. Control implication. Untrusted content must be isolated from instruction channels. Tool execution must be schema constrained and governed by enforceable budgets, with lockdown behavior triggered by anomalies. Metric linkage. Injection Success Rate, Blast Radius Score, Time to Detect and Halt.

---

## 11. Conclusion
Agentic AI is a deployment shape that turns model outputs into state changes. That shift requires a readiness discipline grounded in enforceable controls, auditability, and recovery. A practical readiness model consists of capability tiers, autonomy budgets, measurable scorecards, adversarial evaluation harnesses, and a phased rollout ladder. The rollout ladder now includes a dedicated Delegated Autonomy phase for Tier 4 deployments, reflecting that multi agent coordination introduces blast radius through delegation that requires its own qualification gate. Teams can adopt agents safely in bounded domains when they treat autonomy as a privilege earned through controls and measurement rather than as a default entitlement of model capability.

---

# Appendix A. Autonomy budget template

## A.1 Purpose
This template defines enforceable caps for tool using agents. Budgets are tier specific and must be enforced by the control plane, not by prompts. Budgets exist to bound blast radius when ambiguity, injection, or tool misuse occurs.

## A.2 Budget Record
| Field | Value |
|---|---|
| Budget ID | |
| Owner | |
| Environment (sandbox / staging / production) | |
| Capability Tier (0 / 1 / 2 / 3 / 4) | |
| Effective Date | |
| Review Cadence (weekly / monthly / quarterly) | |
| Emergency Contact | |

## A.3 Global Execution Caps
| Cap | Value |
|---|---|
| Max runtime per task (seconds) | |
| Max tool calls per task | |
| Max concurrent tool calls | |
| Max unique systems touched per task | |
| Max unique target resources per task | |

## A.4 Write and Privilege Caps
| Cap | Value |
|---|---|
| Max write actions per task | |
| Max privileged actions per session | |
| Max irreversible actions per session | 0 (default) |
| Max delete operations per task | |
| Max permission changes per task | |
| Max spend per task (currency) | |
| Max external data exports per task (records or bytes) | |

## A.5 Credential Constraints
| Constraint | Value |
|---|---|
| Credential lifetime (minutes) | |
| Credential scope granularity (per tool / per action type / per resource) | |
| Privileged credentials allowed (yes / no) | |
| Separation of duties required for privileged actions (yes / no) | |

## A.6 Escalation Triggers
Approval required when: (1) Any write action is proposed above threshold. (2) Any privileged action is proposed. (3) Any cap in Global Execution Caps or Write and Privilege Caps would be exceeded.

Lockdown triggers when: (1) Untrusted content is detected in the instruction channel. (2) Plan execution deviates from the approved plan. (3) Novel targets exceed baseline. (4) Repeated tool failures exceed threshold.

Human takeover required when: (1) Rollback is unavailable for any proposed write. (2) Privileged actions are requested. (3) Injection indicators are present.

## A.7 Evidence Requirements (Tier 2+)
Every write action must produce an evidence record containing intent reference, enumerated targets, approvals, tool calls, outcomes, and rollback identifier when available.

## A.8 Suggested starting values (non normative)
| Tier | Max tool calls per task | Max write actions per task | Max privileged actions per session | Max unique systems per task | Max irreversible actions |
|---:|---:|---:|---:|---:|---:|
| 1 | 20 | 0 | 0 | 2 | 0 |
| 2 | 30 | 10 | 0 | 2 | 0 |
| 3 | 40 | 20 | 2 | 3 | 0 |
| 4 | 60 | 30 | 2 | 4 | 0 |

---

# Appendix B. Failure mode to control mapping
Placeholder.

# Appendix C. Readiness scorecard checklist
Placeholder.

# Appendix D. Example action schemas and evidence record format
Placeholder.

# Appendix E. Audit event schema examples
Placeholder.

# Appendix F. Evaluation scenario suite
Placeholder.

# Appendix G. Rollout checklists
Placeholder.

# Appendix H. Related work and references
This appendix lists public references used as anchors for verifiable incidents and background context, and includes the author's related work on enforceable intent checks.

References (IEEE style)

[1] GitLab, "Postmortem of database outage of January 31," 2017. [Online]. Available: https://about.gitlab.com/blog/postmortem-of-database-outage-of-january-31/. [Accessed: Mar. 17, 2026].

[2] The Guardian, "Arup lost $25m in Hong Kong deepfake video conference scam," May 17, 2024. [Online]. Available: https://www.theguardian.com/technology/article/2024/may/17/uk-engineering-arup-deepfake-scam-hong-kong-ai-video. [Accessed: Mar. 17, 2026].

[3] The Verge, "The AI security nightmare is here and it looks suspiciously like lobster," 2026. [Online]. Available: https://www.theverge.com/ai-artificial-intelligence/881574/cline-openclaw-prompt-injection-hack. [Accessed: Mar. 17, 2026].

[4] R. S. J. Corral, "SAFE Intent Framework (RFC-style draft)," 2026. [Online]. Available: https://doi.org/10.5281/zenodo.18896883. [Accessed: Mar. 17, 2026].
