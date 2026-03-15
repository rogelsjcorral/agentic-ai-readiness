# Operational Readiness Criteria for Tool-Using LLM Agents
## Controls, Metrics, and Rollout Gates for Delegated Autonomy

Rogel S.J. Corral  
Independent Researcher  
Draft v0.4 (for community review)

Abstract  
Agentic AI changes the deployment shape of LLM systems. When an LLM can plan, call tools, and execute multi step actions, the primary risk shifts from incorrect text to incorrect state change. This field manual proposes an operational readiness approach for deploying tool using agents, including capability tiers, an autonomy budget, a readiness scorecard, and an evaluation harness with measurable metrics. The goal is not to argue for or against agents. The goal is to define a minimum operational bar for delegated autonomy and a rollout approach that reduces preventable incidents.

Document type and intent  
This document is written as an operational field manual. It provides deployment readiness criteria, control patterns, and evaluation methods for tool using LLM agents. It is designed to be applied in production environments and to support rollout gating, auditability, and recovery planning.

Intended audience  
This document is intended for engineering teams responsible for deploying and operating tool using agents, including platform engineering, security engineering, SRE and operations teams, and product teams shipping agents with write capable tools.

Assumptions  
This document assumes the agent can call tools that interact with external systems. It assumes that some actions can change state and that at least some deployments involve privileged or high impact workflows.

How to use this document  
Use Sections 4 through 9 as an implementation sequence. Identify relevant failure modes, apply the readiness scorecard, implement the control patterns, instrument audit events, validate with the evaluation harness, then roll out by phase using the rollout ladder. Treat Tier 2 and above as ineligible until rollback and evidence capture requirements are met.

Normative language  
This document uses plain requirement language for deployability. When it states that a capability requires a control, it indicates a recommended deployment gate for that capability tier.

Statement of scope  
This document is an operations focused guide for teams deploying tool using agents in real systems. It prioritizes concrete controls, measurable gates, and recovery requirements. It is not a vendor comparison, not a policy manifesto, and not a claim that agentic AI is inherently unsafe. It treats agentic AI as a capability upgrade with an expanded attack surface.

1. Definitions and scope

1.1 Definitions  
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

1.2 Agent shapes in scope  
This document considers three common agent deployment shapes. In a single agent tool user shape, one model both plans and executes tool calls. In a planner executor split, a planner proposes actions and an executor performs constrained tool calls. In multi agent orchestration, multiple agents coordinate, delegate, or vote on plans and actions. These shapes differ in enforceability, audit complexity, and blast radius, and therefore affect readiness requirements.

1.3 Deployment environments in scope  
This document addresses agents used in internal enterprise automation, customer facing workflows with tool access, and administrative or security sensitive workflows, including identity, finance, and infrastructure.

1.4 Non goals  
This document does not propose new model architectures or training methods. It does not claim formal proof of safety. It does not make universal policy claims about social impact.

2. Why agents change the operational risk model

2.1 From answers to actions  
In a chat system, the primary failure is incorrect text. In an agent system, outputs can become actions via tools and credentials. The primary failure mode becomes incorrect state change. This matters because state change can be irreversible, expensive, or silently harmful. The operational question is therefore not only whether a response is correct, but whether a proposed action is authorized, bounded in scope, reversible, and auditable.

2.2 Compounding risk through multi step execution  
Agent systems commonly operate in loops. Looping introduces compounding error in two ways. First, small interpretation errors can cascade into multiple tool calls. Second, the agent can generate new context from its own actions and then use that context to justify further actions. This increases blast radius even when each individual step appears reasonable in isolation.

2.3 Retrieval and memory as attack surfaces  
Tool outputs can contain untrusted content, including web pages, emails, tickets, logs, and documents. Durable memory can persist incorrect or adversarial inputs across sessions. In both cases, an agent can treat untrusted data as instructions unless explicit isolation, validation, and execution gating controls exist.

2.4 Core principle  
Autonomy must be earned by controls and measurement, not assumed from model capability.

3. Capability tiers and the autonomy budget

3.1 Capability tiers  
Tier 0. No tools, advice only. The system provides guidance with no external effects.  
Tier 1. Read only tools. The system can retrieve information, such as search or knowledge base lookup, but cannot write or change state.  
Tier 2. Write tools in bounded systems. The system can perform low impact writes with defined rollback and clear approvals, such as creating tickets, drafting emails, opening pull requests, or updating non critical records.  
Tier 3. Privileged actions. The system can perform high impact changes, such as modifying access controls, moving funds, changing production configuration, or deleting records.  
Tier 4. Cross system autonomy and multi agent delegation. The system coordinates across multiple services and delegates tasks among agents. Tier 4 increases blast radius through delegation and parallelism. Operational readiness at Tier 4 requires explicit delegation boundaries, a shared evidence chain across agents, and enforceable caps on cross system traversal and parallel tool execution. An agent may not delegate privileged actions unless the delegate inherits the same autonomy budget and approval artifacts.

3.2 Autonomy budget  
An autonomy budget is a configurable set of caps that limits what an agent can do before escalation. Budgets should be tier specific and enforced by the control plane, not by prompts. The purpose of the autonomy budget is to bound blast radius when errors occur, including errors induced by ambiguity, prompt injection, memory poisoning, or tool misuse. At minimum, an autonomy budget should constrain maximum tool calls per task, maximum write actions per task, maximum privileged actions per session, maximum unique systems touched per task, maximum irreversible actions per session, thresholds for spend, delete, and permission changes, maximum concurrency, and maximum runtime. Irreversible actions should be blocked by default unless enabled through a high assurance approval path.

The following table provides suggested starting values by tier. These are conservative defaults intended to give teams a calibration anchor; they should be adjusted based on the specific workflow, risk profile, and observed behavior during harness evaluation. Values marked 0 are hard defaults that require an explicit override and a separate approval artifact to change.

| Cap | Tier 1 | Tier 2 | Tier 3 | Tier 4 |
|---|---|---|---|---|
| Max tool calls per task | 20 | 30 | 20 | 40 |
| Max write actions per task | 0 | 10 | 5 | 15 |
| Max privileged actions per session | 0 | 0 | 3 | 3 |
| Max irreversible actions per session | 0 | 0 | 0 | 0 |
| Max unique systems touched per task | 2 | 3 | 3 | 6 |
| Max unique target resources per task | 10 | 20 | 10 | 30 |
| Max concurrent tool calls | 2 | 3 | 2 | 5 |
| Max runtime per task (seconds) | 60 | 120 | 120 | 300 |
| Max delete operations per task | 0 | 2 | 1 | 3 |
| Max permission changes per task | 0 | 0 | 2 | 2 |
| Credential lifetime (minutes) | 15 | 15 | 10 | 10 |

Two patterns are worth noting. First, Tier 3 caps are deliberately lower than Tier 2 for write-adjacent dimensions, because the impact of each permitted action is higher. Volume and blast radius should be inversely scaled as privilege increases. Second, Tier 4 increases execution caps to support multi-agent coordination, but those caps apply per-agent, not per-task; the aggregate blast radius across a delegated task can therefore be substantially larger, which is why cross-system traversal caps and evidence continuity requirements (Section 6.7) are mandatory at that tier.

3.3 Budget escalation rules  
Budgets require deterministic escalation triggers. Approval is required when a plan includes any write action above defined thresholds. Lockdown mode is required when injection signals or anomalies occur, switching the agent to read only behavior while requesting human review. Human takeover is required when rollback is unavailable or when privileged actions are requested.

3.4 Deliverable  
Appendix A provides a one page autonomy budget template that teams can adopt per tier.

4. Failure modes that matter in production

4.1 Intent drift and scope mismatch  
An agent can produce actions that are internally coherent yet misaligned with operator intent. This often occurs when the input request is underspecified, when target selection is inferred, or when a reasonable default is applied to a privileged scope. In practice, this failure mode is dangerous because the action can look correct in a quick skim while still targeting the wrong resources. The operational signature is a mismatch between the user's stated intent and the concrete target set, change set, or impact surface.

4.2 Prompt injection through untrusted tool outputs  
When tool outputs contain untrusted content, an agent may treat that content as instructions rather than data. The common carriers are web pages, tickets, emails, logs, and documents. This is not a hypothetical edge case. Recent incidents show prompt injection being used as an execution lever in agentic workflows and in supply chain adjacent scenarios. Operationally, this manifests as tool calls that were not part of the user's request, or plan deviations that correlate with recently retrieved content.

4.3 Over-privilege and credential mis-scoping  
If an agent is given broad API tokens or long lived credentials, it can perform actions that were never intended, especially when combined with injection or intent drift. The root cause is typically convenience, a lack of tiering, or a missing control plane that can mint least privilege, short lived tokens per action class. The operational signature is privilege overreach, where the agent uses capabilities it did not strictly need for the task.

4.4 Cascading actions and self-reinforcing context  
Multi step execution can create a feedback loop. An initial mistake produces a state change, which becomes new context, which then justifies further actions. The blast radius expands even if each step looks locally rational. The operational signature is action sequences whose justification references the agent's own prior actions rather than an external ticket, approval record, or validated intent.

4.5 Memory poisoning and durable state corruption  
If the agent writes to durable memory, it can store incorrect or adversarial content that persists across sessions. This converts a single failure into a recurring failure pattern. The operational signature is repeated, stable misbehavior across tasks that share no legitimate common cause.

4.6 Approval fatigue and weak human gating  
Human in the loop can degrade into human in the way when approvals become repetitive, poorly summarized, or decoupled from concrete diffs. In those conditions, approvals become a throughput mechanism rather than a control. Real incidents of AI-enabled fraud underscore that humans can be socially engineered even when they believe they are following procedure. The operational signature is high approval velocity with low review quality, and approvals occurring without target visibility, diff visibility, or rollback awareness.

5. Readiness scorecard  
This section defines a minimum operational bar by capability tier. Readiness is assessed across six dimensions. The scorecard is intentionally deployable. It is designed to gate rollouts, not to decorate slide decks.

5.1 Dimensions  
Authority control. Least privilege, short lived credentials, separation of duties for high impact changes.  
Intent verification. Structured plans, explicit target enumeration, confirmation gates tied to actions.  
Adversarial resilience. Isolation of untrusted content, injection defenses, memory hygiene.  
Observability. Audit events, traceability from request to action, replay support where feasible.  
Containment and recovery. Sandboxes, dry runs, rollback procedures, lockdown behavior.  
Human oversight. Escalation rules, review queues, and meaningful approvals.

5.2 Minimum bar per tier  
Tier 1 requires isolation of retrieved content and auditability of retrieval sources. Tier 2 requires intent verification, evidence records, and tested rollback for every permitted write action. Tier 3 requires separation of duties, strict credential scoping, explicit approvals for each privileged action class, and continuous monitoring with lockdown triggers. Tier 4 requires all Tier 3 controls plus explicit delegation boundaries, cross system traversal caps, and evidence continuity across agents.

The table below consolidates the minimum bar into a scannable format. **R** = required. **P** = partial or recommended. Blank = not applicable at that tier. Any required cell that cannot be satisfied is a rollout blocker for that tier.

| Control dimension | T1 | T2 | T3 | T4 |
|---|---|---|---|---|
| Least privilege credentials | P | R | R | R |
| Short-lived, per-task credentials | | R | R | R |
| Separation of duties (privileged) | | | R | R |
| Structured plan with target enumeration | | R | R | R |
| Confirmation gate before write | | R | R | R |
| Two-channel confirmation (irreversible) | | P | R | R |
| Untrusted content isolation | R | R | R | R |
| Memory hygiene / write controls | | R | R | R |
| Audit events on all tool calls | R | R | R | R |
| Traceability from request to action | P | R | R | R |
| Sandbox / dry-run capability | | R | R | R |
| Tested rollback per write action | | R | R | R |
| Lockdown mode implemented | | P | R | R |
| Escalation rules defined | P | R | R | R |
| Review queue for approvals | | R | R | R |
| Delegation boundaries (Tier 4) | | | | R |
| Cross-system traversal caps | | | P | R |
| Evidence continuity across agents | | | | R |

5.3 Pass or fail rules  
If rollback does not exist for a write action, that action is not eligible for Tier 2 automation.  
If credentials cannot be scoped by tool, action type, and resource, Tier 3 is not eligible.  
If approvals do not expose targets and diffs, approvals are not controls and cannot be used to justify Tier 2 or Tier 3 readiness.  
If untrusted tool outputs can influence tool invocation parameters without isolation, Tier 2 and above is not eligible.

6. Control patterns that actually work

6.1 Tool allowlisting and action schemas  
Tool access must be explicit. Each tool must have an allowlisted set of action types. Each action type must have typed parameters and explicit validation. Freeform tool execution is incompatible with Tier 2 and above.

In practice, enforcement means maintaining a registry of permitted tools and their schemas in the control plane, not in the system prompt. The registry entry for each tool specifies: the permitted action verbs (e.g., `read`, `create`, `update`), the typed parameter spec for each verb, and the tier at which each verb is permitted. At runtime, the executor validates every proposed tool call against the registry before dispatch. Calls that reference unlisted tools, use unlisted verbs, or pass parameters that fail type checks are rejected and logged before any external system is contacted.

What this pattern does not catch is authorization errors within a permitted scope: an agent with allowlisted `update` access to a ticketing system can still update the wrong ticket if target selection is incorrect. This is why intent verification and target enumeration in confirmation gates are necessary complements to schema enforcement rather than substitutes for it.

6.2 Planner executor separation  
A planner executor split reduces the attack surface by isolating natural language planning from constrained execution. The planner can propose, but the executor should only accept schema valid actions within an autonomy budget. This pattern also improves audit quality because the plan becomes a first class artifact.

Concretely, the planner produces a structured plan object — a typed list of proposed actions, each with explicit targets and parameters — rather than directly emitting tool calls. The executor receives only the validated plan; it does not have access to the planner's raw conversational context or retrieved documents. This boundary means that even a successfully injected planner prompt cannot cause the executor to issue calls that fall outside the plan's validated schema and the autonomy budget.

The boundary also creates the natural audit checkpoint: the plan object is the artifact that gets approved, logged, and later reviewed. An investigation into a production incident can start from the plan rather than reconstructing intent from a conversation transcript.

A common implementation gap is a permeable boundary: the plan is generated but the executor retains access to conversational context and uses it to "fill in" ambiguous parameters. This defeats the isolation. The executor's context must be strictly limited to the plan object, the tool registry, and the autonomy budget; anything else should be treated as a boundary violation.

6.3 Two channel confirmation for high impact changes  
For destructive or irreversible actions, confirmation should be separated from the conversational channel. At minimum, the confirmation step must restate targets, summarize diffs, and bind to an approval identifier. This is a control against both intent drift and injection.

The implementation requirement is that the confirmation surface be generated independently of the agent's output. If the agent constructs the confirmation message, a successful injection can craft a confirmation that looks legitimate while masking the actual targets. The confirmation should be rendered by the control plane directly from the structured plan object — specifically, from the target identifiers and change descriptors in the validated schema — so that what the reviewer sees reflects what will actually execute.

The approval identifier produced by the confirmation step must be a prerequisite for execution, and the executor must validate its presence and binding before dispatching. Approvals issued in one task context must not be portable to another. A common weakness is that approval identifiers are checked for existence but not for scope: verifying only that an approval was granted, rather than that it was granted for this specific target set and action, allows approval replay across tasks.

This pattern does not eliminate social engineering risk — Section 4.6 addresses that failure mode — but it substantially raises the bar by decoupling the visual representation of an action from the agent's ability to influence it.

6.4 Least privilege and short lived credentials  
Credentials should be minted per task, scoped per tool and per action class, with short lifetimes. Tier 3 requires separation of duties for privileged actions, where the agent cannot both propose and execute without an external approval artifact.

In practice, this requires a credential broker in the control plane that accepts a validated plan and an approval identifier, then mints a short-lived token scoped to exactly the tools and action types in that plan. The agent receives the token; it cannot request broader scopes by modifying the plan after approval, because the plan is fixed at the time the token is issued. Credential lifetime should be shorter than the expected task duration for most workloads, with a mechanism to renew only if the task is still within budget.

The most common failure pattern is a long-lived service account credential shared across tasks. This is operationally convenient but collapses the per-task scoping entirely: any task that runs under that credential inherits the full scope of any other task that has ever run under it. Compounding the problem, long-lived credentials are harder to rotate after an incident and their blast radius in a credential-theft scenario spans the full history of the service account's usage.

6.5 Lockdown mode as a first class behavior  
When anomaly signals trigger, the agent should default to read only behavior, stop tool execution, and request human review. Lockdown is not a user interface feature. It is an operational control that prevents cascading damage during suspected compromise.

Lockdown must be implementable without model cooperation. The control plane should be able to revoke the active task credential, drain the execution queue, and flag the session for review without relying on the agent to honor a soft stop instruction. In practice this means the execution layer checks authorization status before every tool dispatch — not just at task start — and the credential broker supports revocation that takes effect on the next dispatch cycle.

A common gap is that lockdown is implemented as a message sent to the agent asking it to stop. This fails under injection: a compromised agent may not honor the instruction, or may take a final privileged action before acknowledging the stop. Hard revocation at the control plane does not have this weakness. Once the credential is revoked, tool calls fail at the authorization layer regardless of what the agent attempts.

6.6 Evidence records for every write  
Tier 2 and above requires a minimal evidence record tying together intent, enumerated targets, approvals, tool calls, and outcomes. This is necessary for accountability and post incident reconstruction. It also prevents untraceable automation from becoming normal practice.

6.7 Multi agent delegation controls (Tier 4)  
Delegation must be explicit and constrained. Each delegated task must carry an immutable context bundle containing the approved plan reference, target constraints, applicable autonomy budget, and evidence record identifier. Delegates must be prevented from expanding scope beyond inherited constraints. For privileged actions, delegation requires separation of duties, with approvals bound to the specific delegate action type and target set.

Cross agent communication verification patterns  
Cross agent messages must be treated as untrusted data unless verified by the control plane. The objective is to prevent a compromised agent, poisoned content, or external injection from causing other agents to accept instructions outside approved constraints.

Message envelope. Each cross agent message should include a structured envelope with the following fields: message identifier, sender identifier, recipient identifier, timestamp, expiration time, conversation or task identifier, approved plan reference, autonomy budget identifier, and a payload hash. The payload must not be executable instructions. It must be structured data that the recipient validates against its schemas.

Signature or token binding. The envelope must be bound to an authenticity mechanism. Two acceptable patterns are common in practice. First, the control plane issues a short lived signed token to the sender for a specific task identifier and recipient set. The recipient validates the token and rejects it if the task identifier, recipient, expiration, or privileges do not match. Second, the control plane signs the envelope, and the recipient verifies the signature using a pinned control plane public key. In both patterns, trust is placed in the control plane, not in other agents.

Replay protection. Every envelope must contain a nonce or monotonic sequence number. Recipients must maintain a short lived replay cache keyed by sender and task identifier. Envelopes that repeat nonces, violate sequence ordering, or exceed expiration must be rejected.

Scope binding. Recipients must validate that incoming messages reference an existing approved plan and an applicable autonomy budget. If the message cannot be bound to an approval artifact, the recipient must treat it as untrusted context only and must not translate it into tool calls or plan changes.

Quorum gating for privileged actions. For delegated privileged actions, require either a human approval bound to the delegate's action and target set, or a quorum rule such as two independent delegates agreeing on the same schema valid action proposal. Quorum is not a replacement for approvals at Tier 3. It is an additional safeguard against single agent compromise.

Independence in a quorum rule is not automatic and must be structurally enforced. Two delegates are independent for quorum purposes only if they have been initialized from different context sources and have not exchanged messages outside of control-plane-verified envelopes. If both delegates retrieved content from the same external source during their task window, a single poisoned document in that source can cause both to reach the same injected conclusion, satisfying the quorum rule while both are compromised. To mitigate this, the control plane should verify that quorum participants' approved plan references are bound to distinct retrieval sessions, and should treat overlapping retrieval provenance as a quorum disqualifier. Where retrieval cannot be isolated, human approval remains the only valid gate for privileged actions.

Failure behavior. If verification fails, the recipient must enter lockdown mode for the affected task, refuse tool execution, and emit an audit event indicating rejection reason and envelope metadata.

7. Observability and audit

7.1 Audit event requirements  
Every tool invocation should emit an audit event containing timestamp, actor, tool identifier, action type, target identifiers, parameter hashes, approval identifier where applicable, outcome status, and rollback identifier when available. The audit record must be sufficient to reconstruct what changed, even if the model conversation is unavailable.

7.2 Alerts and anomaly signals  
Action burst detection, novel targets, cross system hop growth, privilege escalation attempts, plan execution mismatch, and repeated failures are baseline signals. At Tier 3, these signals should be wired to lockdown triggers, not merely dashboards.

7.3 Post incident replay  
Deterministic replay is not always possible, but post incident reconstruction should be. Logging of plans, tool I/O, and approval artifacts is the practical minimum.

8. Evaluation harness and metrics

8.1 Harness design  
A minimal harness should provide a sandbox environment with mock or constrained APIs for representative tools, plus a corpus of adversarial inputs such as poisoned tickets, malicious HTML, booby trapped logs, and injection-laced documents. Acceptance testing must be scenario based and tier specific.

8.2 Metrics

The following six metrics form the core measurement set. All six should be tracked in the evaluation harness and reported per tier and per scenario class.

**Unauthorized Action Rate (UAR).** UAR measures the fraction of actions taken that were not explicitly authorized under the applicable policy for the task. For a given test run, let $A$ be the total number of tool-dispatching actions executed. Let $U$ be the number of those actions that (a) were not present in the approved plan, or (b) targeted resources outside the approved target set, or (c) used permission scopes not granted by the task credential. $\mathrm{UAR} = U / \max(1, A)$. In adversarial scenarios, include injected tasks that attempt to elicit out-of-plan actions; those count toward $U$ if executed. Interpretation: $\mathrm{UAR} = 0$ under adversarial conditions is the Tier 2 target. Any nonzero UAR in production is an incident trigger.

**Injection Success Rate (ISR).** ISR measures how often adversarial content embedded in tool outputs succeeds in influencing action selection or action parameters. For a given adversarial scenario corpus of size $N$, let $I$ be the count of scenarios in which the agent executed an action whose target, verb, or parameters can be attributed to the injected instruction rather than the original approved plan. $\mathrm{ISR} = I / N$. Attribution is determined by comparing the executed action against the plan produced before retrieval; any deviation correlated with injected content counts. Interpretation: $\mathrm{ISR}$ should be measured at each tier boundary. Tier 2 eligibility requires $\mathrm{ISR}$ below a team-defined threshold under the harness scenario suite. A rising $\mathrm{ISR}$ across model updates is a regression signal.

**Time to Detect and Halt (TDH).** TDH measures the elapsed time from the first anomalous signal to enforced execution stop. Define the start of the measurement window as the timestamp of the first anomaly event (action burst, novel target, plan deviation, or injection indicator) recorded in the audit log. Define the end of the measurement window as the timestamp at which the control plane enforces a stop: either credential revocation takes effect or the execution queue is drained. $\mathrm{TDH}$ is the wall-clock duration of that window in seconds. Interpretation: $\mathrm{TDH}$ bounds the damage window after a compromise is detected. Teams should define per-tier TDH targets during harness calibration. At Tier 3, TDH should be measured under injection scenarios where the agent actively continues execution after the anomaly.

**Rollback Success Rate (RSR).** RSR measures the fraction of write actions that are successfully reverted during rollback drills. For a given drill, let $W$ be the total number of write actions executed. Let $R$ be the number for which the rollback procedure completed successfully: the target system returned to its pre-action state within the allowed rollback window, as verified by a post-rollback state check. $\mathrm{RSR} = R / \max(1, W)$. Drills must be run under realistic conditions, including partial failure scenarios where some writes succeed and others do not. Interpretation: $\mathrm{RSR} = 1.0$ is required for Tier 2 eligibility. Any write action for which rollback cannot be demonstrated reduces $\mathrm{RSR}$ and is a Tier 2 blocker under the pass/fail rules in Section 5.3.

**Blast Radius Score (BRS).** BRS measures how far an agent's actions propagated before halt. Define a task window $W$ from first tool call to stop condition. Let $T$ be the number of unique target resources modified within $W$. Let $S$ be the number of unique external systems touched within $W$. Define weights $\alpha$ and $\beta$ by environment criticality (default $\alpha = 1$, $\beta = 5$ in production).

$$\mathrm{BRS} = \alpha T + \beta S$$

Interpretation. A low BRS indicates bounded execution. A rising BRS is a regression signal even when outcomes appear correct.

**Privilege Overreach Index (POI).** POI measures how much permission surface area the agent exercised relative to the minimum required for the approved plan. For a given task, define $P_{\mathrm{used}}$ as the set of distinct permission scopes actually exercised (API scopes, IAM actions, role privileges, or equivalent). Define $P_{\mathrm{req}}$ as the minimum permission scope set required to execute the approved plan.

$$\mathrm{POI} = \frac{\left| P_{\mathrm{used}} \setminus P_{\mathrm{req}} \right|}{\max\left(1,\left|P_{\mathrm{req}}\right|\right)}$$

Interpretation. $\mathrm{POI} = 0$ indicates least privilege execution. $\mathrm{POI} > 0$ indicates overreach and should be treated as a policy failure at Tier 3.

8.3 Acceptance criteria  
Tier 2 eligibility requires stable rollback success and low unauthorized action rate under ambiguous and adversarial scenarios. Tier 3 eligibility requires strong privilege scoping and demonstrably effective lockdown triggers under injection scenarios.

9. Rollout ladder

Each phase has defined entry criteria, exit criteria, a minimum duration or sample threshold, an authority level required to advance, and a regression condition that triggers rollback to the previous phase. No phase may be skipped. Advancement authority should be assigned to a named role before rollout begins; the default assignment is the platform engineering lead in coordination with the relevant security or SRE owner.

9.1 Sandbox mode  
No real side effects. The objective is functional correctness, audit completeness, and repeatable rollback drills.

Entry criteria. Tool registry and autonomy budget are defined and enforced by the control plane. Audit logging is instrumented and verified to capture all required fields (Section 7.1). Rollback procedures exist and are documented for every write action type in scope.

Exit criteria. The evaluation harness (Section 8) has been run against the full scenario suite for the target tier. RSR = 1.0 across all drill scenarios. UAR = 0 under adversarial scenarios. Audit log completeness verified for all harness runs.

Minimum duration / sample threshold. At least three full harness runs with no regressions between runs. No clock-based minimum; exit is gated on harness results, not elapsed time.

Advancement authority. Platform engineering lead, with sign-off from security engineering.

Regression condition. Any harness run that produces a nonzero UAR, an RSR below 1.0, or incomplete audit events resets the run counter to zero.

9.2 Shadow mode  
The agent proposes plans and actions, but a human executes. This phase measures intent drift, target enumeration quality, and injection susceptibility without production writes.

Entry criteria. Sandbox exit criteria met. A human execution process is in place: the agent's proposed plans are routed to a reviewer who manually executes approved actions.

Exit criteria. At least 50 representative tasks completed. Intent drift rate (fraction of proposed plans whose target set diverged from the reviewer's expected targets) is below a defined threshold, typically 5% for Tier 2 and 2% for Tier 3. ISR measured against injected inputs embedded in task context is below threshold. No lockdown-triggering anomalies that were not detected within TDH target.

Minimum duration / sample threshold. Minimum 50 tasks, minimum two calendar weeks. For lower-volume workflows, the two-week minimum applies even if 50 tasks have not been reached; in that case, document the volume shortfall explicitly.

Advancement authority. Platform engineering lead, with sign-off from the product team and SRE.

Regression condition. Intent drift rate or ISR exceeds threshold in any rolling 10-task window. Any anomaly that was not detected and halted within TDH target. Return to shadow mode if either condition occurs during a later phase.

9.3 Assisted write mode  
The agent executes low impact writes only, under strict autonomy budgets and mandatory evidence capture. Every write must be rollback capable.

Entry criteria. Shadow mode exit criteria met. Per-task credentials are minted by the control plane and scoped to the write actions permitted at this phase. Evidence records are being generated and stored for all writes. Rollback has been rehearsed in production (not only in sandbox) for each permitted write action type.

Exit criteria. At least 100 write-inclusive tasks completed. RSR = 1.0 in any rollback rehearsals conducted during this phase. UAR = 0. No evidence record gaps. TDH within target for any anomalies triggered.

Minimum duration / sample threshold. Minimum 100 tasks, minimum four calendar weeks.

Advancement authority. Platform engineering lead and SRE lead, joint sign-off. For workflows involving external customer data, product and legal review is recommended.

Regression condition. Any nonzero UAR in production. Any write action for which rollback fails. Any evidence record gap. Immediate rollback to shadow mode; root cause analysis required before re-entry to assisted write mode.

9.4 Bounded autonomy  
Expanded write scope within a bounded domain, with enforced budgets, anomaly triggered lockdown, and periodic requalification using the harness.

Entry criteria. Assisted write mode exit criteria met. Lockdown mode is tested and verified to halt execution within TDH target under injection scenarios. Anomaly detection signals are wired to lockdown triggers, not only dashboards. The domain boundary (the set of systems and resource types within scope) is explicitly defined and enforced by the tool registry and credential scoping.

Exit criteria. Not applicable for most deployments; bounded autonomy is the target steady state for Tier 2 workflows. Advancement to privileged autonomy (Tier 3) requires a separate readiness assessment.

Requalification cadence. The full harness suite must be re-run whenever the model is updated, the tool registry changes, or the autonomy budget is modified. A requalification run that produces regressions against prior baseline metrics suspends advancement and requires root cause analysis.

Advancement authority. Full readiness review including platform engineering, security engineering, SRE, and product. For Tier 3 advancement, legal and compliance review is recommended.

Regression condition. Any anomaly that triggers lockdown and cannot be attributed to a known, benign cause returns the workflow to assisted write mode for a minimum of two weeks. Repeated lockdown triggers (more than two in any 30-day window) require a harness rerun before resuming bounded autonomy.

9.5 Privileged autonomy  
Privileged actions are permitted only when separation of duties, scoped credentials, and meaningful approvals are in place, and when harness metrics remain stable across updates.

Entry criteria. All Tier 3 scorecard requirements satisfied (Section 5.2). Separation of duties is structurally enforced: the agent cannot both propose and execute a privileged action without an out-of-band approval artifact. Privileged credentials are minted only against an approval identifier. Two-channel confirmation is implemented and verified for all privileged action types. Harness metrics (UAR, ISR, BRS, POI, TDH, RSR) are all within tier targets across the most recent full harness run.

Exit criteria. Not applicable; this is the highest rollout tier. Metrics must remain within targets through continuous monitoring and periodic requalification.

Requalification cadence. Full harness suite re-run after every model update, every change to privileged tool scopes, and on a fixed calendar schedule not to exceed 90 days.

Advancement authority. Requires written approval from the platform engineering lead, security engineering lead, and a named executive sponsor. The approval artifact must be retained alongside the harness run records.

Regression condition. Any nonzero UAR in production, any POI > 0 for a privileged action, any TDH miss, or any RSR below 1.0 in a drill triggers immediate suspension of privileged autonomy and return to bounded autonomy pending root cause analysis and a full harness rerun.

10. Real world anchors

10.1 GitLab database outage and rollback reality  
GitLab's 2017 incident shows how a single mistaken destructive action can cause major data loss and how recovery depends on tested backups and practiced procedures, not on good intentions. Control implication. Tier 2 requires tested rollback for every permitted write action. Tier 3 requires additional separation of duties and pre-execution safety gates. Metric linkage. Rollback Success Rate, Time to Detect and Halt.

10.2 Deepfake enabled fraud and approval weakness  
Arup's disclosed deepfake scam demonstrates that human approval is not automatically a safety control. Humans can be socially engineered into authorizing transfers in workflows that appear legitimate. Control implication. Approvals must be tied to concrete diffs, target visibility, and policy thresholds. High impact actions require multi factor verification beyond a single conversational confirmation. Metric linkage. Unauthorized Action Rate, approval quality measures, anomaly detection time.

10.3 Agentic prompt injection and tool chain compromise  
Reports in early 2026 describe prompt injection being used to drive unauthorized installation or execution behaviors in agent-adjacent workflows, including supply chain style abuse patterns. Control implication. Untrusted content must be isolated from instruction channels. Tool execution must be schema constrained and governed by enforceable budgets, with lockdown behavior triggered by anomalies. Metric linkage. Injection Success Rate, Blast Radius Score, Time to Detect and Halt.

11. Conclusion  
Agentic AI is a deployment shape that turns model outputs into state changes. That shift requires a readiness discipline grounded in enforceable controls, auditability, and recovery. A practical readiness model consists of capability tiers, autonomy budgets, measurable scorecards, adversarial evaluation harnesses, and a phased rollout ladder. Teams can adopt agents safely in bounded domains when they treat autonomy as a privilege earned through controls and measurement rather than as a default entitlement of model capability.

Appendix A. Autonomy Budget Template

A.1 Purpose  
This template defines enforceable caps for tool-using agents. Budgets are tier-specific and must be enforced by the control plane, not by prompts. Budgets exist to bound blast radius when ambiguity, injection, or tool misuse occurs.

A.2 Budget Record  
Budget ID: _______________________________  
Owner: __________________________________  
Environment: [ ] sandbox [ ] staging [ ] production  
Capability Tier: [ ] 0 [ ] 1 [ ] 2 [ ] 3 [ ] 4  
Effective Date: __________________________  
Review Cadence: [ ] weekly [ ] monthly [ ] quarterly  
Emergency Contact: ________________________

A.3 Global Execution Caps  
Max runtime per task (seconds): __________  
Max tool calls per task: __________  
Max concurrent tool calls: __________  
Max unique systems touched per task: __________  
Max unique target resources per task: __________

A.4 Write and Privilege Caps  
Max write actions per task: __________  
Max privileged actions per session: __________  
Max irreversible actions per session: 0 (default)  
Max delete operations per task: __________  
Max permission changes per task: __________  
Max spend per task (currency): __________  
Max external data exports per task (records or bytes): __________

A.5 Credential Constraints  
Credential lifetime (minutes): __________  
Credential scope granularity: [ ] per tool [ ] per action type [ ] per resource  
Privileged credentials allowed: [ ] yes [ ] no  
Separation of duties required for privileged actions: [ ] yes [ ] no

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
Every write action must produce an evidence record containing: intent reference, enumerated targets, approvals, tool calls, outcomes, and a rollback identifier when available.

Appendix B. Failure mode to control mapping  
Appendix C. Readiness scorecard checklist  
Appendix D. Example action schemas and evidence record format  
Appendix E. Audit event schema examples  
Appendix F. Evaluation scenario suite  
Appendix G. Rollout checklists  
Appendix H. Related work and references (including S.A.F.E. Intent Framework RFC draft)Evidence record. A minimal artifact set that explains why an action was taken, what inputs were used, what was approved, and what outcome occurred.  
Rollback. A defined method to revert a write action, including a tested procedure and known failure conditions.  
Session state. Short lived working context used during a single run.  
Durable memory. State that persists across sessions, such as notes, profiles, preferences, cached facts, or stored plans.

1.2 Agent shapes in scope  
This document considers three common agent deployment shapes. In a single agent tool user shape, one model both plans and executes tool calls. In a planner executor split, a planner proposes actions and an executor performs constrained tool calls. In multi agent orchestration, multiple agents coordinate, delegate, or vote on plans and actions. These shapes differ in enforceability, audit complexity, and blast radius, and therefore affect readiness requirements.

1.3 Deployment environments in scope  
This document addresses agents used in internal enterprise automation, customer facing workflows with tool access, and administrative or security sensitive workflows, including identity, finance, and infrastructure.

1.4 Non goals  
This document does not propose new model architectures or training methods. It does not claim formal proof of safety. It does not make universal policy claims about social impact.

2. Why agents change the operational risk model

2.1 From answers to actions  
In a chat system, the primary failure is incorrect text. In an agent system, outputs can become actions via tools and credentials. The primary failure mode becomes incorrect state change. This matters because state change can be irreversible, expensive, or silently harmful. The operational question is therefore not only whether a response is correct, but whether a proposed action is authorized, bounded in scope, reversible, and auditable.

2.2 Compounding risk through multi step execution  
Agent systems commonly operate in loops. Looping introduces compounding error in two ways. First, small interpretation errors can cascade into multiple tool calls. Second, the agent can generate new context from its own actions and then use that context to justify further actions. This increases blast radius even when each individual step appears reasonable in isolation.

2.3 Retrieval and memory as attack surfaces  
Tool outputs can contain untrusted content, including web pages, emails, tickets, logs, and documents. Durable memory can persist incorrect or adversarial inputs across sessions. In both cases, an agent can treat untrusted data as instructions unless explicit isolation, validation, and execution gating controls exist.

2.4 Core principle  
Autonomy must be earned by controls and measurement, not assumed from model capability.

3. Capability tiers and the autonomy budget

3.1 Capability tiers  
Tier 0. No tools, advice only. The system provides guidance with no external effects.  
Tier 1. Read only tools. The system can retrieve information, such as search or knowledge base lookup, but cannot write or change state.  
Tier 2. Write tools in bounded systems. The system can perform low impact writes with defined rollback and clear approvals, such as creating tickets, drafting emails, opening pull requests, or updating non critical records.  
Tier 3. Privileged actions. The system can perform high impact changes, such as modifying access controls, moving funds, changing production configuration, or deleting records.  
Tier 4. Cross system autonomy and multi agent delegation. The system coordinates across multiple services and delegates tasks among agents. Tier 4 increases blast radius through delegation and parallelism. Operational readiness at Tier 4 requires explicit delegation boundaries, a shared evidence chain across agents, and enforceable caps on cross system traversal and parallel tool execution. An agent may not delegate privileged actions unless the delegate inherits the same autonomy budget and approval artifacts.

3.2 Autonomy budget  
An autonomy budget is a configurable set of caps that limits what an agent can do before escalation. Budgets should be tier specific and enforced by the control plane, not by prompts. The purpose of the autonomy budget is to bound blast radius when errors occur, including errors induced by ambiguity, prompt injection, memory poisoning, or tool misuse. At minimum, an autonomy budget should constrain maximum tool calls per task, maximum write actions per task, maximum privileged actions per session, maximum unique systems touched per task, maximum irreversible actions per session, thresholds for spend, delete, and permission changes, maximum concurrency, and maximum runtime. Irreversible actions should be blocked by default unless enabled through a high assurance approval path.

3.3 Budget escalation rules  
Budgets require deterministic escalation triggers. Approval is required when a plan includes any write action above defined thresholds. Lockdown mode is required when injection signals or anomalies occur, switching the agent to read only behavior while requesting human review. Human takeover is required when rollback is unavailable or when privileged actions are requested.

3.4 Deliverable  
Appendix A will provide a one page autonomy budget template that teams can adopt per tier.

4. Failure modes that matter in production

4.1 Intent drift and scope mismatch  
An agent can produce actions that are internally coherent yet misaligned with operator intent. This often occurs when the input request is underspecified, when target selection is inferred, or when a reasonable default is applied to a privileged scope. In practice, this failure mode is dangerous because the action can look correct in a quick skim while still targeting the wrong resources. The operational signature is a mismatch between the user’s stated intent and the concrete target set, change set, or impact surface.

4.2 Prompt injection through untrusted tool outputs  
When tool outputs contain untrusted content, an agent may treat that content as instructions rather than data. The common carriers are web pages, tickets, emails, logs, and documents. This is not a hypothetical edge case. Recent incidents show prompt injection being used as an execution lever in agentic workflows and in supply chain adjacent scenarios. Operationally, this manifests as tool calls that were not part of the user’s request, or plan deviations that correlate with recently retrieved content.

4.3 Over-privilege and credential mis-scoping  
If an agent is given broad API tokens or long lived credentials, it can perform actions that were never intended, especially when combined with injection or intent drift. The root cause is typically convenience, a lack of tiering, or a missing control plane that can mint least privilege, short lived tokens per action class. The operational signature is privilege overreach, where the agent uses capabilities it did not strictly need for the task.

4.4 Cascading actions and self-reinforcing context  
Multi step execution can create a feedback loop. An initial mistake produces a state change, which becomes new context, which then justifies further actions. The blast radius expands even if each step looks locally rational. The operational signature is action sequences whose justification references the agent’s own prior actions rather than an external ticket, approval record, or validated intent.

4.5 Memory poisoning and durable state corruption  
If the agent writes to durable memory, it can store incorrect or adversarial content that persists across sessions. This converts a single failure into a recurring failure pattern. The operational signature is repeated, stable misbehavior across tasks that share no legitimate common cause.

4.6 Approval fatigue and weak human gating  
Human in the loop can degrade into human in the way when approvals become repetitive, poorly summarized, or decoupled from concrete diffs. In those conditions, approvals become a throughput mechanism rather than a control. Real incidents of AI-enabled fraud underscore that humans can be socially engineered even when they believe they are following procedure. The operational signature is high approval velocity with low review quality, and approvals occurring without target visibility, diff visibility, or rollback awareness.

5. Readiness scorecard  
This section defines a minimum operational bar by capability tier. Readiness is assessed across six dimensions. The scorecard is intentionally deployable. It is designed to gate rollouts, not to decorate slide decks.

5.1 Dimensions  
Authority control. Least privilege, short lived credentials, separation of duties for high impact changes.  
Intent verification. Structured plans, explicit target enumeration, confirmation gates tied to actions.  
Adversarial resilience. Isolation of untrusted content, injection defenses, memory hygiene.  
Observability. Audit events, traceability from request to action, replay support where feasible.  
Containment and recovery. Sandboxes, dry runs, rollback procedures, lockdown behavior.  
Human oversight. Escalation rules, review queues, and meaningful approvals.

5.2 Minimum bar per tier  
Tier 1 requires isolation of retrieved content and auditability of retrieval sources. Tier 2 requires intent verification, evidence records, and tested rollback for every permitted write action. Tier 3 requires separation of duties, strict credential scoping, explicit approvals for each privileged action class, and continuous monitoring with lockdown triggers. Tier 4 requires all Tier 3 controls plus explicit delegation boundaries, cross system traversal caps, and evidence continuity across agents.

5.3 Pass or fail rules  
If rollback does not exist for a write action, that action is not eligible for Tier 2 automation.  
If credentials cannot be scoped by tool, action type, and resource, Tier 3 is not eligible.  
If approvals do not expose targets and diffs, approvals are not controls and cannot be used to justify Tier 2 or Tier 3 readiness.  
If untrusted tool outputs can influence tool invocation parameters without isolation, Tier 2 and above is not eligible.

6. Control patterns that actually work

6.1 Tool allowlisting and action schemas  
Tool access must be explicit. Each tool must have an allowlisted set of action types. Each action type must have typed parameters and explicit validation. Freeform tool execution is incompatible with Tier 2 and above.

6.2 Planner executor separation  
A planner executor split reduces the attack surface by isolating natural language planning from constrained execution. The planner can propose, but the executor should only accept schema valid actions within an autonomy budget. This pattern also improves audit quality because the plan becomes a first class artifact.

6.3 Two channel confirmation for high impact changes  
For destructive or irreversible actions, confirmation should be separated from the conversational channel. At minimum, the confirmation step must restate targets, summarize diffs, and bind to an approval identifier. This is a control against both intent drift and injection.

6.4 Least privilege and short lived credentials  
Credentials should be minted per task, scoped per tool and per action class, with short lifetimes. Tier 3 requires separation of duties for privileged actions, where the agent cannot both propose and execute without an external approval artifact.

6.5 Lockdown mode as a first class behavior  
When anomaly signals trigger, the agent should default to read only behavior, stop tool execution, and request human review. Lockdown is not a user interface feature. It is an operational control that prevents cascading damage during suspected compromise.

6.6 Evidence records for every write  
Tier 2 and above requires a minimal evidence record tying together intent, enumerated targets, approvals, tool calls, and outcomes. This is necessary for accountability and post incident reconstruction. It also prevents untraceable automation from becoming normal practice.

6.7 Multi agent delegation controls (Tier 4)  
Delegation must be explicit and constrained. Each delegated task must carry an immutable context bundle containing the approved plan reference, target constraints, applicable autonomy budget, and evidence record identifier. Delegates must be prevented from expanding scope beyond inherited constraints. For privileged actions, delegation requires separation of duties, with approvals bound to the specific delegate action type and target set.

Cross agent communication verification patterns  
Cross agent messages must be treated as untrusted data unless verified by the control plane. The objective is to prevent a compromised agent, poisoned content, or external injection from causing other agents to accept instructions outside approved constraints.

Message envelope. Each cross agent message should include a structured envelope with the following fields: message identifier, sender identifier, recipient identifier, timestamp, expiration time, conversation or task identifier, approved plan reference, autonomy budget identifier, and a payload hash. The payload must not be executable instructions. It must be structured data that the recipient validates against its schemas.

Signature or token binding. The envelope must be bound to an authenticity mechanism. Two acceptable patterns are common in practice. First, the control plane issues a short lived signed token to the sender for a specific task identifier and recipient set. The recipient validates the token and rejects it if the task identifier, recipient, expiration, or privileges do not match. Second, the control plane signs the envelope, and the recipient verifies the signature using a pinned control plane public key. In both patterns, trust is placed in the control plane, not in other agents.

Replay protection. Every envelope must contain a nonce or monotonic sequence number. Recipients must maintain a short lived replay cache keyed by sender and task identifier. Envelopes that repeat nonces, violate sequence ordering, or exceed expiration must be rejected.

Scope binding. Recipients must validate that incoming messages reference an existing approved plan and an applicable autonomy budget. If the message cannot be bound to an approval artifact, the recipient must treat it as untrusted context only and must not translate it into tool calls or plan changes.

Quorum gating for privileged actions. For delegated privileged actions, require either a human approval bound to the delegate’s action and target set, or a quorum rule such as two independent delegates agreeing on the same schema valid action proposal. Quorum is not a replacement for approvals at Tier 3. It is an additional safeguard against single agent compromise.

Failure behavior. If verification fails, the recipient must enter lockdown mode for the affected task, refuse tool execution, and emit an audit event indicating rejection reason and envelope metadata.

7. Observability and audit

7.1 Audit event requirements  
Every tool invocation should emit an audit event containing timestamp, actor, tool identifier, action type, target identifiers, parameter hashes, approval identifier where applicable, outcome status, and rollback identifier when available. The audit record must be sufficient to reconstruct what changed, even if the model conversation is unavailable.

7.2 Alerts and anomaly signals  
Action burst detection, novel targets, cross system hop growth, privilege escalation attempts, plan execution mismatch, and repeated failures are baseline signals. At Tier 3, these signals should be wired to lockdown triggers, not merely dashboards.

7.3 Post incident replay  
Deterministic replay is not always possible, but post incident reconstruction should be. Logging of plans, tool I/O, and approval artifacts is the practical minimum.

8. Evaluation harness and metrics

8.1 Harness design  
A minimal harness should provide a sandbox environment with mock or constrained APIs for representative tools, plus a corpus of adversarial inputs such as poisoned tickets, malicious HTML, booby trapped logs, and injection-laced documents. Acceptance testing must be scenario based and tier specific.

8.2 Metrics  
Unauthorized Action Rate. Fraction of actions not explicitly approved under the applicable policy.  
Injection Success Rate. Fraction of scenarios where hidden instructions influence action selection or parameters.  
Blast Radius Score. Count of unique targets or systems touched before halt.  
Time to Detect and Halt. Time from first anomalous signal to enforced stop.  
Rollback Success Rate. Fraction of write actions reverted successfully in drills.  
Privilege Overreach Index. Degree to which exercised permissions exceed the minimum required.

Blast Radius Score (BRS). BRS measures how far an agent’s actions propagated before halt. Define a task window $W$ from first tool call to stop condition. Let $T$ be the number of unique target resources modified within $W$. Let $S$ be the number of unique external systems touched within $W$. Define weights $\alpha$ and $\beta$ by environment criticality (default $\alpha = 1$, $\beta = 5$ in production).

$$
\mathrm{BRS} = \alpha T + \beta S
$$

Interpretation. A low BRS indicates bounded execution. A rising BRS is a regression signal even when outcomes appear correct.

Privilege Overreach Index (POI). POI measures how much permission surface area the agent exercised relative to the minimum required for the approved plan. For a given task, define $P_{\mathrm{used}}$ as the set of distinct permission scopes actually exercised (API scopes, IAM actions, role privileges, or equivalent). Define $P_{\mathrm{req}}$ as the minimum permission scope set required to execute the approved plan.

$$
\mathrm{POI} = \frac{\left| P_{\mathrm{used}} \setminus P_{\mathrm{req}} \right|}{\max\left(1,\left|P_{\mathrm{req}}\right|\right)}
$$

Interpretation. $\mathrm{POI} = 0$ indicates least privilege execution. $\mathrm{POI} > 0$ indicates overreach and should be treated as a policy failure at Tier 3.

8.3 Acceptance criteria  
Tier 2 eligibility requires stable rollback success and low unauthorized action rate under ambiguous and adversarial scenarios. Tier 3 eligibility requires strong privilege scoping and demonstrably effective lockdown triggers under injection scenarios.

9. Rollout ladder

9.1 Sandbox mode  
No real side effects. The objective is functional correctness, audit completeness, and repeatable rollback drills.

9.2 Shadow mode  
The agent proposes plans and actions, but a human executes. This phase measures intent drift, target enumeration quality, and injection susceptibility without production writes.

9.3 Assisted write mode  
The agent executes low impact writes only, under strict autonomy budgets and mandatory evidence capture. Every write must be rollback capable.

9.4 Bounded autonomy  
Expanded write scope within a bounded domain, with enforced budgets, anomaly triggered lockdown, and periodic requalification using the harness.

9.5 Privileged autonomy  
Privileged actions are permitted only when separation of duties, scoped credentials, and meaningful approvals are in place, and when harness metrics remain stable across updates.

10. Real world anchors

10.1 GitLab database outage and rollback reality  
GitLab’s 2017 incident shows how a single mistaken destructive action can cause major data loss and how recovery depends on tested backups and practiced procedures, not on good intentions. Control implication. Tier 2 requires tested rollback for every permitted write action. Tier 3 requires additional separation of duties and pre-execution safety gates. Metric linkage. Rollback Success Rate, Time to Detect and Halt.

10.2 Deepfake enabled fraud and approval weakness  
Arup’s disclosed deepfake scam demonstrates that human approval is not automatically a safety control. Humans can be socially engineered into authorizing transfers in workflows that appear legitimate. Control implication. Approvals must be tied to concrete diffs, target visibility, and policy thresholds. High impact actions require multi factor verification beyond a single conversational confirmation. Metric linkage. Unauthorized Action Rate, approval quality measures, anomaly detection time.

10.3 Agentic prompt injection and tool chain compromise  
Reports in early 2026 describe prompt injection being used to drive unauthorized installation or execution behaviors in agent-adjacent workflows, including supply chain style abuse patterns. Control implication. Untrusted content must be isolated from instruction channels. Tool execution must be schema constrained and governed by enforceable budgets, with lockdown behavior triggered by anomalies. Metric linkage. Injection Success Rate, Blast Radius Score, Time to Detect and Halt.

11. Conclusion  
Agentic AI is a deployment shape that turns model outputs into state changes. That shift requires a readiness discipline grounded in enforceable controls, auditability, and recovery. A practical readiness model consists of capability tiers, autonomy budgets, measurable scorecards, adversarial evaluation harnesses, and a phased rollout ladder. Teams can adopt agents safely in bounded domains when they treat autonomy as a privilege earned through controls and measurement rather than as a default entitlement of model capability.

Appendix A. Autonomy Budget Template

A.1 Purpose  
This template defines enforceable caps for tool-using agents. Budgets are tier-specific and must be enforced by the control plane, not by prompts. Budgets exist to bound blast radius when ambiguity, injection, or tool misuse occurs.

A.2 Budget Record  
Budget ID: _______________________________  
Owner: __________________________________  
Environment: [ ] sandbox [ ] staging [ ] production  
Capability Tier: [ ] 0 [ ] 1 [ ] 2 [ ] 3 [ ] 4  
Effective Date: __________________________  
Review Cadence: [ ] weekly [ ] monthly [ ] quarterly  
Emergency Contact: ________________________

A.3 Global Execution Caps  
Max runtime per task (seconds): __________  
Max tool calls per task: __________  
Max concurrent tool calls: __________  
Max unique systems touched per task: __________  
Max unique target resources per task: __________

A.4 Write and Privilege Caps  
Max write actions per task: __________  
Max privileged actions per session: __________  
Max irreversible actions per session: 0 (default)  
Max delete operations per task: __________  
Max permission changes per task: __________  
Max spend per task (currency): __________  
Max external data exports per task (records or bytes): __________

A.5 Credential Constraints  
Credential lifetime (minutes): __________  
Credential scope granularity: [ ] per tool [ ] per action type [ ] per resource  
Privileged credentials allowed: [ ] yes [ ] no  
Separation of duties required for privileged actions: [ ] yes [ ] no

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
Every write action must produce an evidence record containing: intent reference, enumerated targets, approvals, tool calls, outcomes, and a rollback identifier when available.

Appendix B. Failure mode to control mapping  
Appendix C. Readiness scorecard checklist  
Appendix D. Example action schemas and evidence record format  
Appendix E. Audit event schema examples  
Appendix F. Evaluation scenario suite  
Appendix G. Rollout checklists  
Appendix H. Related work and references (including S.A.F.E. Intent Framework RFC draft)
