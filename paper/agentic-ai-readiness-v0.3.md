# Operational Readiness Criteria for Tool-Using LLM Agents
## Controls, Metrics, and Rollout Gates for Delegated Autonomy

Rogel S.J. Corral  
Independent Researcher  
Draft v0.3 (for community review)

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
Delegation must be explicit and constrained. Each delegated task must carry an immutable context bundle containing the approved plan reference, target constraints, applicable autonomy budget, and evidence record identifier. Delegates must be prevented from expanding scope beyond inherited constraints. Cross agent communication channels must treat all inbound messages as untrusted data unless signed by the control plane. For privileged actions, delegation requires separation of duties, with approvals bound to the specific delegate action type and target set.

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
