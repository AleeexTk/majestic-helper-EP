# Schemas — JSON Schema Definitions (Layer 3)

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04

---

## Overview

All data validated against JSON Schema. Minimal but strict validation ensures:
- Type safety
- Constraint enforcement
- Cross-environment compatibility
- Clear error messages on validation failure

---

## Core Schemas

### 1. `project_initial` Schema

**Used by:** `clarify_project` contract input

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://majestic-helper.io/schemas/project_initial.json",
  "title": "Initial Project Description",
  "type": "object",
  "required": ["description"],
  "properties": {
    "description": {
      "type": "string",
      "minLength": 1,
      "maxLength": 5000,
      "description": "Initial project description (user input)"
    },
    "budget_estimate": {
      "type": "string",
      "enum": ["<1M", "1M-3M", "3M-10M", ">10M"],
      "description": "Optional budget range (currency unit context-dependent)"
    },
    "deadline_hint": {
      "type": "string",
      "format": "date",
      "description": "Optional target launch date (ISO 8601)"
    },
    "industry": {
      "type": "string",
      "description": "Optional industry context"
    }
  },
  "additionalProperties": false
}
```

### 2. `clarify_response` Schema

**Used by:** `clarify_project` contract output

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://majestic-helper.io/schemas/clarify_response.json",
  "title": "Clarifying Questions Response",
  "type": "object",
  "required": ["questions", "next_suggested_action"],
  "properties": {
    "questions": {
      "type": "object",
      "required": ["pm", "analyst", "consultant"],
      "properties": {
        "pm": {
          "type": "array",
          "items": {"type": "string"},
          "minItems": 2,
          "maxItems": 3,
          "description": "Product Manager questions"
        },
        "analyst": {
          "type": "array",
          "items": {"type": "string"},
          "minItems": 2,
          "maxItems": 3,
          "description": "Business Analyst questions"
        },
        "consultant": {
          "type": "array",
          "items": {"type": "string"},
          "minItems": 2,
          "maxItems": 3,
          "description": "Technical Consultant questions"
        }
      },
      "additionalProperties": false
    },
    "next_suggested_action": {
      "type": "string",
      "description": "What user should do next"
    },
    "confidence": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Confidence in understanding (0-1)"
    }
  },
  "additionalProperties": false
}
```

### 3. `requirements_output` Schema

**Used by:** `analyze_requirements` contract output

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://majestic-helper.io/schemas/requirements_output.json",
  "title": "Requirements Analysis Output",
  "type": "object",
  "required": [
    "goal",
    "functional_requirements",
    "non_functional_requirements",
    "success_criteria"
  ],
  "properties": {
    "project_name": {
      "type": "string",
      "description": "Extracted or inferred project name"
    },
    "goal": {
      "type": "string",
      "description": "High-level project goal"
    },
    "target_audience": {
      "type": "string",
      "description": "Who will use this"
    },
    "functional_requirements": {
      "type": "array",
      "items": {"type": "string"},
      "minItems": 5,
      "description": "What system does (natural language only)"
    },
    "non_functional_requirements": {
      "type": "object",
      "required": ["performance", "security"],
      "properties": {
        "performance": {
          "type": "string",
          "description": "Speed, latency, throughput (e.g., 'Sub-2 second page load')"
        },
        "scalability": {
          "type": "string",
          "description": "Growth expectations (e.g., '10k to 100k users')"
        },
        "security": {
          "type": "string",
          "description": "Auth, compliance (e.g., 'SOC2 compliant')"
        },
        "availability": {
          "type": "string",
          "description": "Uptime requirement (e.g., '99.5% SLA')"
        }
      }
    },
    "constraints": {
      "type": "object",
      "properties": {
        "budget": {"type": "string"},
        "timeline": {"type": "string"},
        "team_constraints": {"type": "string"}
      }
    },
    "success_criteria": {
      "type": "array",
      "items": {"type": "string"},
      "minItems": 3,
      "description": "Measurable outcomes"
    }
  },
  "additionalProperties": false
}
```

### 4. `roadmap_output` Schema

**Used by:** `suggest_roadmap` contract output

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://majestic-helper.io/schemas/roadmap_output.json",
  "title": "Project Roadmap",
  "type": "object",
  "required": ["phases", "disclaimer"],
  "properties": {
    "phases": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["name", "task_types", "duration_weeks_example"],
        "properties": {
          "name": {
            "type": "string",
            "description": "Phase name (generic: 'Design Phase' not 'Design UI')"
          },
          "task_types": {
            "type": "array",
            "items": {"type": "string"},
            "description": "Task categories (not specific tasks)"
          },
          "duration_weeks_example": {
            "type": "integer",
            "minimum": 1,
            "maximum": 52,
            "description": "Example duration (not binding)"
          },
          "team_roles": {
            "type": "array",
            "items": {"type": "string"},
            "description": "Suggested roles"
          }
        }
      },
      "minItems": 3,
      "maxItems": 6
    },
    "estimated_total_weeks": {
      "type": "integer",
      "description": "Total timeline estimate"
    },
    "team_composition_example": {
      "type": "string",
      "description": "Example team makeup"
    },
    "disclaimer": {
      "type": "string",
      "pattern": ".*[Cc]ontact.*us.*",
      "description": "Must include 'Contact us' or similar"
    }
  },
  "additionalProperties": false
}
```

### 5. `risk_output` Schema

**Used by:** `identify_risks` contract output

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://majestic-helper.io/schemas/risk_output.json",
  "title": "Risk Assessment",
  "type": "object",
  "required": ["risks"],
  "properties": {
    "risks": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["description", "probability", "impact", "mitigation"],
        "properties": {
          "description": {
            "type": "string",
            "description": "What could go wrong"
          },
          "category": {
            "type": "string",
            "enum": ["technical", "schedule", "resource", "scope", "external"],
            "description": "Risk category"
          },
          "probability": {
            "type": "string",
            "enum": ["low", "medium", "high"],
            "description": "Likelihood (low/medium/high)"
          },
          "impact": {
            "type": "string",
            "enum": ["low", "medium", "high"],
            "description": "Business impact (low/medium/high)"
          },
          "mitigation": {
            "type": "string",
            "description": "Generic mitigation strategy (not specific implementation)"
          }
        }
      },
      "minItems": 3,
      "maxItems": 10
    },
    "risk_summary": {
      "type": "string",
      "description": "Overall risk assessment"
    }
  },
  "additionalProperties": false
}
```

### 6. `session_context` Schema

**Used by:** Runtime session management

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://majestic-helper.io/schemas/session_context.json",
  "title": "Session Context",
  "type": "object",
  "required": ["session_id", "created_at"],
  "properties": {
    "session_id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique session identifier"
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "Session creation timestamp (ISO 8601)"
    },
    "last_active": {
      "type": "string",
      "format": "date-time",
      "description": "Last message timestamp"
    },
    "context": {
      "type": "object",
      "properties": {
        "github_repo": {"type": "string"},
        "project_name": {"type": "string"},
        "industry": {"type": "string"}
      },
      "description": "Contextual information"
    },
    "answers": {
      "type": "object",
      "additionalProperties": {"type": "string"},
      "description": "Stored answers from clarifications"
    },
    "last_contract": {
      "type": "string",
      "description": "Name of last executed contract"
    },
    "tech_question_counter": {
      "type": "integer",
      "minimum": 0,
      "description": "Count of technical questions (for BusinessGuard)"
    },
    "history": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "user_message": {"type": "string"},
          "agent_response_summary": {"type": "string"},
          "timestamp": {"type": "string", "format": "date-time"}
        }
      },
      "maxItems": 10,
      "description": "Last 3-10 exchanges"
    }
  },
  "additionalProperties": false
}
```

### 7. `trace_log_entry` Schema

**Used by:** JSONL tracing/logging

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://majestic-helper.io/schemas/trace_log_entry.json",
  "title": "Trace Log Entry",
  "type": "object",
  "required": [
    "timestamp",
    "session_id",
    "contract",
    "duration_ms",
    "success"
  ],
  "properties": {
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "ISO 8601 timestamp"
    },
    "session_id": {
      "type": "string",
      "format": "uuid",
      "description": "Session ID (not user identifiable)"
    },
    "contract": {
      "type": "string",
      "description": "Contract name executed"
    },
    "input_size": {
      "type": "integer",
      "description": "Input message length (characters)"
    },
    "output_size": {
      "type": "integer",
      "description": "Output response length (characters)"
    },
    "duration_ms": {
      "type": "integer",
      "description": "Execution time in milliseconds"
    },
    "success": {
      "type": "boolean",
      "description": "Did contract execute successfully?"
    },
    "error_type": {
      "type": "string",
      "enum": [
        "EMPTY_INPUT",
        "SCHEMA_VALIDATION_FAILED",
        "TIMEOUT",
        "RATE_LIMIT",
        "API_ERROR",
        null
      ],
      "description": "Error type if success=false"
    },
    "tech_counter": {
      "type": "integer",
      "description": "Technical question counter after this contract"
    }
  },
  "additionalProperties": false
}
```

---

## Schema Validation Rules

**Global Rules:**
- All required fields must be present
- No additional properties allowed (strict mode)
- String fields trimmed of whitespace
- Enums case-sensitive
- Dates in ISO 8601 format (YYYY-MM-DD or YYYY-MM-DDTHH:mm:ssZ)
- UUIDs must be valid format
- Arrays respect min/max items
- Numbers within specified ranges

**Error Responses:**

If validation fails:

```json
{
  "status": "error",
  "type": "SCHEMA_VALIDATION_FAILED",
  "field": "budget_estimate",
  "message": "Invalid value. Expected one of: <1M, 1M-3M, 3M-10M, >10M",
  "received": "1000"
}
```

---

## Extensibility

Schema versioning follows semantic versioning:
- **Major:** Breaking changes (new required field)
- **Minor:** Backward-compatible additions (new optional field)
- **Patch:** Fixes (typo, constraint adjustment)

Old schema versions supported during deprecation period (typically 3 months).

---

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04
