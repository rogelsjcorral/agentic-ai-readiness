# Appendix D. Example action schemas and evidence record format

## D.1 Purpose
This appendix provides schema notation for tool action definitions and evidence record structures. These are not implementation files. They are reference patterns that teams can adapt to their control plane's schema language. All field names and types are illustrative. Teams should treat schema updates as requalification events as noted in Section 6.1.

## D.2 Action schema pattern

```
ActionSchema {
  tool_id:         string           -- unique identifier for the tool
  tool_version:    string           -- version; schema changes require
                                    -- requalification
  tier_minimum:    integer          -- minimum capability tier required
                                    -- to invoke this tool (1-4)
  action_types: [
    ActionType {
      name:        string           -- e.g. "create_ticket", "delete_record"
      is_write:    boolean          -- true if action changes external state
      is_privileged: boolean        -- true if Tier 3 controls required
      is_reversible: boolean        -- false means irreversible; blocks
                                    -- execution unless explicitly approved
      parameters: [
        Parameter {
          name:      string
          type:      string         -- e.g. "string", "integer", "resource_id"
          required:  boolean
          validation: ValidationRule {
            min_length:  integer?   -- for string types
            max_length:  integer?
            pattern:     string?    -- regex, applied before execution
            enum_values: [string]?  -- if parameter is an enumeration
            resource_scope: string? -- e.g. "project", "org"; limits
                                    -- target to declared scope
          }
        }
      ]
      requires_approval_id:  boolean  -- true for all write actions
                                      -- at Tier 2 and above
      requires_plan_ref:     boolean  -- true for Tier 2 and above
      requires_evidence_ref: boolean  -- true for Tier 2 and above
      rollback_available:    boolean  -- false blocks Tier 2 eligibility
      rollback_action_type:  string?  -- name of the inverse action type
    }
  ]
}
```

## D.3 Example: ticket creation action schema

```
ActionSchema {
  tool_id:       "ticketing_system"
  tool_version:  "2.1"
  tier_minimum:  2
  action_types: [
    ActionType {
      name:           "create_ticket"
      is_write:       true
      is_privileged:  false
      is_reversible:  true
      parameters: [
        Parameter {
          name:     "project_key"
          type:     "string"
          required: true
          validation: ValidationRule {
            pattern:        "^[A-Z]{2,10}$"
            resource_scope: "project"
          }
        }
        Parameter {
          name:      "summary"
          type:      "string"
          required:  true
          validation: ValidationRule {
            min_length: 1
            max_length: 255
          }
        }
        Parameter {
          name:      "priority"
          type:      "string"
          required:  false
          validation: ValidationRule {
            enum_values: ["low", "medium", "high"]
          }
        }
      ]
      requires_approval_id:  true
      requires_plan_ref:     true
      requires_evidence_ref: true
      rollback_available:    true
      rollback_action_type:  "delete_ticket"
    }
  ]
}
```

## D.4 Example: permission grant action schema

```
ActionSchema {
  tool_id:       "iam_system"
  tool_version:  "1.0"
  tier_minimum:  3
  action_types: [
    ActionType {
      name:           "grant_permission"
      is_write:       true
      is_privileged:  true
      is_reversible:  true
      parameters: [
        Parameter {
          name:     "principal_id"
          type:     "resource_id"
          required: true
          validation: ValidationRule {
            resource_scope: "org"
          }
        }
        Parameter {
          name:     "permission_scope"
          type:     "string"
          required: true
          validation: ValidationRule {
            enum_values: [
              "read:project",
              "write:project",
              "admin:project"
            ]
          }
        }
        Parameter {
          name:     "resource_id"
          type:     "resource_id"
          required: true
          validation: ValidationRule {
            resource_scope: "project"
          }
        }
      ]
      requires_approval_id:  true
      requires_plan_ref:     true
      requires_evidence_ref: true
      rollback_available:    true
      rollback_action_type:  "revoke_permission"
    }
  ]
}
```

## D.5 Evidence record format

```
EvidenceRecord {
  evidence_id:      string          -- unique; referenced in all
                                    -- write action audit events
  task_id:          string          -- parent task identifier
  session_id:       string          -- agent session identifier
  created_at:       timestamp       -- UTC, millisecond resolution
  intent_ref:       string          -- reference to the original
                                    -- user request or ticket
  plan_ref:         string          -- identifier of the approved plan
  approved_plan: ApprovedPlan {
    plan_id:        string
    proposed_at:    timestamp
    approved_at:    timestamp
    approved_by:    string          -- human approver identifier
    approval_id:    string          -- auditable approval identifier
    target_set:     [resource_id]   -- enumerated targets; no wildcards
    action_classes: [string]        -- list of action type names in plan
    diff_summary:   string          -- human-readable change summary
                                    -- shown to approver at approval time
  }
  tool_calls: [
    ToolCallRecord {
      call_id:          string
      tool_id:          string
      action_type:      string
      invoked_at:       timestamp
      parameter_hashes: {string: string}  -- field name to hash;
                                          -- not raw values
      target_ids:       [resource_id]
      outcome_status:   string      -- "success" | "failure" |
                                    -- "blocked" | "rolled_back"
      rollback_id:      string?     -- present if outcome_status
                                    -- is "rolled_back"
    }
  ]
  rollback_refs: [
    RollbackRecord {
      rollback_id:      string
      original_call_id: string
      rolled_back_at:   timestamp
      rollback_status:  string      -- "success" | "partial" | "failed"
      verification:     string      -- result of post-rollback
                                    -- state verification
    }
  ]
  closed_at:        timestamp?      -- set when task completes
                                    -- or is terminated
}
```
