# Appendix E. Audit event schema examples

## E.1 Purpose
This appendix provides the audit event schema and annotated examples for representative event types. The schema covers required fields and optional fields as defined in Section 7.1. Examples illustrate a retrieval event (Tier 1), a write action event (Tier 2), a privileged action event (Tier 3), and a lockdown event. A cross-agent delegation event example is included for Tier 4.

## E.2 Base audit event schema

```
AuditEvent {
  -- Required fields (Section 7.1)
  event_id:         string          -- globally unique event identifier
  timestamp:        timestamp       -- UTC, millisecond resolution
  actor_id:         string          -- agent instance identifier
  session_id:       string          -- agent session identifier
  tool_id:          string          -- tool that was invoked or attempted
  action_type:      string          -- action type name from schema
  target_ids:       [resource_id]   -- resources acted upon or attempted
  parameter_hashes: {string: string}  -- field name to hash of value;
                                      -- no raw parameter values
  outcome_status:   string          -- "success" | "failure" |
                                    -- "blocked" | "rolled_back"
  evidence_id:      string          -- required at Tier 2 and above;
                                    -- omit only at Tier 1

  -- Optional fields (required for specific metrics)
  plan_ref:         string?         -- required to compute UAR
  approval_id:      string?         -- required at Tier 3+; used
                                    -- for UAR and POI computation
  rollback_id:      string?         -- present when outcome is
                                    -- "rolled_back"; used for RSR
  parent_task_id:   string?         -- required at Tier 4 for cross-
                                    -- agent evidence continuity
  anomaly_flags:    [string]?       -- signal names triggered during
                                    -- this invocation (see Section 7.3)
  latency_ms:       integer?        -- milliseconds from plan proposal
                                    -- to tool invocation; used for TDH
  permission_scopes_used: [string]? -- required for POI computation
}
```

## E.3 Example: retrieval event (Tier 1)

```
AuditEvent {
  event_id:         "evt_0a1b2c3d"
  timestamp:        "2026-03-17T10:04:22.341Z"
  actor_id:         "agent_instance_7f3a"
  session_id:       "sess_9d2e1f"
  tool_id:          "knowledge_base"
  action_type:      "search"
  target_ids:       ["kb_index_projects"]
  parameter_hashes: { "query": "sha256:4f9e..." }
  outcome_status:   "success"
  evidence_id:      null            -- Tier 1: not required
  plan_ref:         "plan_cc8811"
  anomaly_flags:    []
  latency_ms:       312
}
```

## E.4 Example: write action event (Tier 2)

```
AuditEvent {
  event_id:         "evt_1b2c3d4e"
  timestamp:        "2026-03-17T10:07:55.014Z"
  actor_id:         "agent_instance_7f3a"
  session_id:       "sess_9d2e1f"
  tool_id:          "ticketing_system"
  action_type:      "create_ticket"
  target_ids:       ["project_key:INFRA"]
  parameter_hashes: {
    "project_key": "sha256:8a3f...",
    "summary":     "sha256:2b7c...",
    "priority":    "sha256:1d4a..."
  }
  outcome_status:   "success"
  evidence_id:      "evr_ff3301"   -- required at Tier 2
  plan_ref:         "plan_cc8811"
  approval_id:      "appr_7712aa"  -- write approval bound to plan
  rollback_id:      null           -- populated on rollback
  anomaly_flags:    []
  latency_ms:       88
  permission_scopes_used: ["write:ticketing_system:project:INFRA"]
}
```

## E.5 Example: privileged action event (Tier 3)

```
AuditEvent {
  event_id:         "evt_2c3d4e5f"
  timestamp:        "2026-03-17T14:22:08.771Z"
  actor_id:         "agent_instance_7f3a"
  session_id:       "sess_4b1c2d"   -- executor session; different
                                    -- from planner session
  tool_id:          "iam_system"
  action_type:      "grant_permission"
  target_ids:       [
    "principal:user_0099",
    "resource:project_alpha"
  ]
  parameter_hashes: {
    "principal_id":     "sha256:9c1e...",
    "permission_scope": "sha256:3f8b...",
    "resource_id":      "sha256:7a2d..."
  }
  outcome_status:   "success"
  evidence_id:      "evr_aa9902"
  plan_ref:         "plan_dd4422"
  approval_id:      "appr_cc5533"  -- privileged action approval;
                                   -- bound to this action class
                                   -- and target set
  anomaly_flags:    []
  latency_ms:       204
  permission_scopes_used: ["admin:iam_system:project:project_alpha"]
}
```

## E.6 Example: lockdown event

```
AuditEvent {
  event_id:         "evt_3d4e5f6g"
  timestamp:        "2026-03-17T15:11:44.003Z"
  actor_id:         "agent_instance_7f3a"
  session_id:       "sess_9d2e1f"
  tool_id:          "control_plane"
  action_type:      "lockdown_enforced"
  target_ids:       ["task_id:task_88cc"]
  parameter_hashes: {}
  outcome_status:   "blocked"
  evidence_id:      "evr_bb1144"
  plan_ref:         "plan_cc8811"
  approval_id:      null
  anomaly_flags:    ["plan_execution_mismatch"]
                                   -- signal that triggered lockdown
  latency_ms:       6              -- time from mismatch detection
                                   -- to enforced stop; feeds TDH
}
```

## E.7 Example: cross-agent delegation events (Tier 4)

```
-- Event emitted by parent agent at delegation time:
AuditEvent {
  event_id:         "evt_4e5f6g7h"
  timestamp:        "2026-03-17T16:05:30.119Z"
  actor_id:         "agent_instance_parent_3a"
  session_id:       "sess_parent_001"
  tool_id:          "control_plane"
  action_type:      "delegation_issued"
  target_ids:       ["delegate:agent_instance_child_9b"]
  parameter_hashes: {
    "context_bundle_hash": "sha256:5f7a...",
    "budget_id":           "sha256:2c4e..."
  }
  outcome_status:   "success"
  evidence_id:      "evr_cc2255"
  plan_ref:         "plan_ee5533"
  approval_id:      "appr_dd6644"  -- approval covering this
                                   -- delegation and target set
  parent_task_id:   null           -- this is the root task
  anomaly_flags:    []
  latency_ms:       41
}

-- Event emitted by delegate agent on acceptance:
AuditEvent {
  event_id:         "evt_5f6g7h8i"
  timestamp:        "2026-03-17T16:05:30.284Z"
  actor_id:         "agent_instance_child_9b"
  session_id:       "sess_child_002"
  tool_id:          "control_plane"
  action_type:      "delegation_accepted"
  target_ids:       ["task_id:task_delegate_77dd"]
  parameter_hashes: {
    "envelope_signature": "sha256:8b3c..."
  }
  outcome_status:   "success"
  evidence_id:      "evr_cc2255"   -- same evidence record as parent
  plan_ref:         "plan_ee5533"  -- inherited from parent
  approval_id:      "appr_dd6644" -- inherited from parent
  parent_task_id:   "task_parent_88cc"
                                   -- links delegate to parent;
                                   -- required for evidence continuity
  anomaly_flags:    []
  latency_ms:       9
}
```
