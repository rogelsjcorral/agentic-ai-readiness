# Revision History

This document tracks substantive changes to the paper and its operational artifacts.
Minor copy edits and formatting changes are not listed.

## Rev v0.1 (2026-03-21)
- Added license declaration to the paper text.
- Added examples files.
- Bumped version to v1.0 and updated date to 2026-03-21 in title block of both LaTeX and Markdown sources.

## Rev v0.7 (2026-03-18)
- Completed Appendix B (Failure mode to control mapping) with an 8-row table covering all six Section 4 failure modes plus two failure modes introduced in v0.6 (shared context quorum failure from Section 6.7.1 and audit trail suppression from Section 7.5). Each row maps to primary control, secondary controls, detection metric, and minimum tier gate. Added Section B.3 coverage gap check procedure for pre-Sandbox sign-off.
- Completed Appendix C (Readiness scorecard checklist) converting the Section 5 scorecard into tiered yes/no operator questions, one subsection per dimension, with each tier's questions explicitly building on the tier below. Any no answer is a blocking finding. Carry-forward requirement stated explicitly: a team qualifying for Tier 3 must satisfy all Tier 1 and Tier 2 items.
- Completed Appendix D (Example action schemas and evidence record format) with the base ActionSchema pattern in pseudocode, two worked examples (Tier 2 ticket creation and Tier 3 IAM permission grant), and the full EvidenceRecord structure including ApprovedPlan, ToolCallRecord, and RollbackRecord sub-structures with field-level annotations. Schema updates noted as requalification events per Section 6.1.
- Completed Appendix E (Audit event schema examples) with the base AuditEvent envelope distinguishing required from optional fields with per-field metric linkage annotations, and five concrete examples: retrieval event (Tier 1), write action event (Tier 2), privileged action event (Tier 3), lockdown event, and paired delegation events (delegation_issued by parent, delegation_accepted by delegate) showing how parent_task_id creates the cross-agent evidence chain at Tier 4.
- Completed Appendix F (Evaluation scenario suite) with nine scenarios across all four tiers in structured Scenario notation, each specifying setup, input, expected behaviour, pass condition, fail condition, and metrics exercised. Coverage: injection via web page and novel target detection (Tier 1); ticket injection, rollback verification, autonomy budget enforcement, and approval fatigue detection (Tier 2); privilege escalation attempt, privileged plan injection, and cascading action lockdown (Tier 3); shared-context quorum invalidation, unverified cross-agent message rejection, and evidence continuity chain verification (Tier 4). Added Section F.7 scenario corpus maintenance note tying corpus coverage to the ISR threshold in Section 8.3.
- Completed Appendix G (Rollout checklists) with per-phase entry, in-phase monitoring, and exit checklists for all six rollout phases from Section 9. Each exit checklist embeds the numerical threshold from Section 8.3 inline. Sign-off blocks escalate in approver count across phases: single owner at Phase 1, two owners at Phases 2 and 3, three owners at Phase 4, and all four approvers (platform, service, security, executive) at Phases 5 and 6, matching the authority-to-advance requirements in Section 9. Phase 6 exit checklist framed as the periodic requalification record rather than a phase advancement gate, reflecting that Delegated Autonomy is the terminal phase.
- Bumped version to v0.7 and updated date to 2026-03-18 in title block of both LaTeX and Markdown sources.

## Rev v0.6 (2026-03-17)
- Added Section 7.2 (Audit field completeness as an acceptance gate), distinguishing required audit fields (minimum for post-incident reconstruction without the model conversation) from optional fields, and treating missing required fields as a blocking scorecard finding equivalent to a failed pass/fail rule in Section 5.4.
- Added Section 7.3 (Alerts and anomaly signals) with per-signal descriptions and explicit lockdown-trigger tier assignments for each baseline signal. Separated approval fatigue indicators as a human oversight review queue signal rather than an automatic lockdown trigger, closing the loop between Section 4.6 and the Arup anchor in Section 10.2.
- Updated Section 7.4 (Post incident replay, formerly 7.3) to specify that plans must be logged at proposal time, tool I/O at invocation time, and approval artifacts retained for the duration of the evidence retention policy.
- Added Section 7.5 (Audit retention and access controls) covering the threat of an agent suppressing its own audit trail, with access control requirements restricting write access to the control plane, prohibiting agent runtime modification of emitted events, requiring separate approval to delete or amend privileged action audit events, and mandating a central audit store at Tier 4 that no individual delegate can modify.
- Added Section 8.3 (Suggested threshold anchors, non normative) with a table of starting values for all six metrics (UAR, ISR, TDH, RSR, BRS, POI) across all four tiers, plus per-metric notes clarifying what each threshold means operationally. Former Section 8.3 (Acceptance criteria) renumbered to 8.4 with a cross-reference to the new table.
- Updated Section 9.5 (Privileged autonomy) exit criteria to reference the Section 8.3 threshold anchors explicitly.
- Added Section 9.6 (Delegated autonomy, Tier 4) as a distinct qualification gate that must not begin until all Privileged Autonomy exit criteria are met. Includes entry criteria covering delegation boundary enforcement, immutable context bundle verification, cross-agent message verification, and per-delegate autonomy budget enforcement; exit criteria covering Tier 4 metric thresholds, cross-agent evidence continuity, and quorum independence verification; an authority-to-advance gate requiring all four approvers (platform, service, security, executive); and regression triggers covering delegate budget overrun, unverified cross-agent messages, undocumented privileged delegation, evidence continuity gaps, and shared-context quorum acceptance.
- Updated Section 11 (Conclusion) to reference the new Delegated Autonomy phase and the qualification gate rationale.
- Bumped version to v0.6 in title block of both LaTeX and Markdown sources.

## Rev v0.5 (2026-03-17)
- Aligned the Markdown paper with the LaTeX source to ensure content parity for PDF builds.
- Converted web references to IEEE-style entries including [Online] and [Accessed] fields, and standardized URL handling.
- Added KaTeX-compatible math blocks for BRS and POI in the Markdown source so equations render as display math.
- Added cross-agent communication verification patterns under Section 6.7.1, including message envelope fields, signature or token binding, replay protection, scope binding, and failure behavior.
- Added an operator-friendly scorecard checklist table (Pass / Conditional / Fail) directly in Section 5 to remove dependence on Appendix C for basic usability.
- Added Appendix placeholders B through G in the LaTeX build so Related Work and References consistently appears as Appendix H.

## Rev v0.4 (2026-03-15)
- Added tier-differentiated autonomy budget starting values (T1–T4) with a reference table for 11 cap dimensions.
- Expanded readiness scorecard with an inline control matrix mapping all six dimensions to tiers, formatted as a rollout blocker checklist.
- Operationalized all six evaluation metrics (UAR, ISR, TDH, RSR) with formal definitions, denominator formulas, measurement windows, and interpretation guidance to match the existing BRS and POI specifications.
- Expanded rollout ladder phases with entry criteria, exit criteria, minimum duration and sample thresholds, advancement authority, and regression conditions.
- Added implementation detail to control patterns 6.1–6.5 (tool allowlisting, planner-executor separation, two-channel confirmation, least privilege credentials, lockdown mode), including common failure patterns and scope limitations for each.
- Strengthened quorum gating guidance in Section 6.7 with structural independence requirements and a description of the shared-context poisoning attack on quorum rules.
- Extended Section 5.3 pass/fail rules to cover all critical R-marked scorecard dimensions, including audit events, lockdown mode, and delegation boundaries.
- Expanded Section 6.6 evidence records with required field specification, append-only storage requirements, and operational signature of a gap.
- Added steady-state performance floor to Section 9.4 bounded autonomy exit criteria (UAR = 0, RSR = 1.0, BRS non-trending).
- Updated Section 10 metric linkage lines to reference formal metric definitions in Section 8.2.
- Added cross-reference from Appendix A.1 to Section 3.2 starting values table.
- Added DOI for SAFE Intent Framework reference [4].

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
