# AEX Protocol v1.0 - JSON Schemas
## Formal Data Structure Definitions and Validation

**Last Updated:** February 5, 2026  
**Version:** 1.0 (HEX Integrated)

---

## Table of Contents

1. [Overview](#overview)
2. [Schema Conventions](#schema-conventions)
3. [AEX_ID Schemas](#aex_id-schemas)
4. [AEX_REP Schemas](#aex_rep-schemas)
5. [AEX_DEX Schemas](#aex_dex-schemas)
6. [AEX_HEX Schemas](#aex_hex-schemas) â† **NEW**
7. [AEX_SESSION Schemas](#aex_session-schemas)
8. [AEX_WITNESS Schemas](#aex_witness-schemas)
9. [Handshake Schemas](#handshake-schemas)
10. [Ledger Schemas](#ledger-schemas)
11. [Cross-Primitive Validation](#cross-primitive-validation)
12. [Validation Examples](#validation-examples)

---

## Overview

This document provides **JSON Schema definitions** for all AEX protocol data structures, including the **four core primitives**:

- **AEX_ID** - Identity and lineage
- **AEX_REP** - Authority and delegation
- **AEX_DEX** - Behavioral reliability
- **AEX_HEX** - Experience and capability â† **NEW**

**Purpose:**
- Formal specification of data formats
- Automated validation
- Interoperability assurance
- Implementation guidance

**Schema Standard:** JSON Schema Draft 2020-12

---

## Schema Conventions

### Common Definitions

```json
{
  "$defs": {
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[1-9A-HJ-NP-Za-km-z]{43,44}$",
      "description": "AEX Decentralized Identifier (base58-encoded public key)"
    },
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$",
      "description": "UUID v4 identifier"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$",
      "description": "ISO 8601 timestamp in UTC"
    },
    "signature": {
      "type": "string",
      "pattern": "^[1-9A-HJ-NP-Za-km-z]{86,88}$",
      "description": "Base58-encoded Ed25519 signature (64 bytes)"
    },
    "public_key": {
      "type": "string",
      "pattern": "^ed25519:[1-9A-HJ-NP-Za-km-z]{43,44}$",
      "description": "Ed25519 public key with prefix"
    },
    "outcome": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0,
      "description": "Normalized outcome value [0,1]"
    },
    "confidence": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0,
      "description": "Confidence value [0,1]"
    },
    "domain": {
      "type": "string",
      "pattern": "^[a-z0-9_.]+$",
      "minLength": 1,
      "maxLength": 100,
      "description": "Domain identifier (lowercase, dots, underscores)"
    }
  }
}
```

### Validation Rules

**MUST Requirements:**
- All required fields present
- Types match specification
- Formats conform to patterns
- Ranges within bounds
- Signatures verify correctly

**SHOULD Requirements:**
- Optional fields when applicable
- Metadata for context
- Consistent timestamp ordering

---

## AEX_ID Schemas

### Identity Object

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/identity.json",
  "title": "AEX Identity",
  "description": "Complete agent identity with lineage and fork history",
  "type": "object",
  "required": [
    "aex_id",
    "public_key",
    "created_at",
    "lineage",
    "signature"
  ],
  "properties": {
    "aex_id": {
      "$ref": "#/$defs/aex_id"
    },
    "public_key": {
      "$ref": "#/$defs/public_key"
    },
    "created_at": {
      "$ref": "#/$defs/timestamp"
    },
    "lineage": {
      "type": "object",
      "required": ["parent_id", "fork_history"],
      "properties": {
        "parent_id": {
          "oneOf": [
            {"$ref": "#/$defs/aex_id"},
            {"type": "null"}
          ],
          "description": "Parent identity (null for genesis agents)"
        },
        "fork_history": {
          "type": "array",
          "items": {
            "$ref": "#/$defs/uuid"
          },
          "description": "Ordered list of fork_id UUIDs from root to current"
        }
      }
    },
    "stake": {
      "oneOf": [
        {"type": "null"},
        {
          "type": "object",
          "required": ["enabled", "amount", "currency", "locked_until"],
          "properties": {
            "enabled": {"type": "boolean"},
            "amount": {"type": "number", "minimum": 0},
            "currency": {
              "type": "string",
              "enum": ["USD", "EUR", "ETH", "BTC"]
            },
            "locked_until": {"$ref": "#/$defs/timestamp"},
            "slashing_conditions": {
              "type": "array",
              "items": {"type": "string"}
            },
            "proof": {"type": "string"},
            "verified_at": {"$ref": "#/$defs/timestamp"}
          }
        }
      ]
    },
    "metadata": {
      "type": "object",
      "description": "Optional identity metadata",
      "properties": {
        "agent_class": {"type": "string"},
        "version": {"type": "string"},
        "description": {"type": "string"},
        "capabilities": {
          "type": "array",
          "items": {"type": "string"}
        }
      }
    },
    "signature": {
      "$ref": "#/$defs/signature",
      "description": "Self-signature proving key ownership"
    }
  },
  "$defs": {
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[1-9A-HJ-NP-Za-km-z]{43,44}$"
    },
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
    },
    "signature": {
      "type": "string",
      "pattern": "^[1-9A-HJ-NP-Za-km-z]{86,88}$"
    },
    "public_key": {
      "type": "string",
      "pattern": "^ed25519:[1-9A-HJ-NP-Za-km-z]{43,44}$"
    }
  }
}
```

### Fork Event Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/fork_event.json",
  "title": "AEX Fork Event",
  "description": "Record of agent fork/evolution",
  "type": "object",
  "required": [
    "fork_id",
    "parent_id",
    "child_id",
    "fork_type",
    "claimed_weight",
    "enforced_weight",
    "timestamp",
    "signature"
  ],
  "properties": {
    "fork_id": {
      "$ref": "#/$defs/uuid"
    },
    "parent_id": {
      "$ref": "#/$defs/aex_id"
    },
    "child_id": {
      "$ref": "#/$defs/aex_id"
    },
    "fork_type": {
      "type": "string",
      "enum": ["bugfix", "minor", "major", "override", "custom"],
      "description": "Fork type determines maximum inheritance weight and probation observation window. bugfix=1.0/5, minor=0.8/10, major=0.5/20, override=0.1/50."
    },
    "claimed_weight": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0,
      "description": "Continuity weight claimed by parent"
    },
    "enforced_weight": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0,
      "description": "Protocol-enforced weight based on fork_type"
    },
    "observation_target": {
      "type": "integer",
      "minimum": 0,
      "description": "Probation observation window in number of completed interactions. See AEX_DEX Probation System for exit conditions."
    },
    "min_observations": {
      "type": "integer",
      "minimum": 0,
      "description": "Minimum interactions before accelerated exit assessment is allowed."
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    },
    "author": {
      "$ref": "#/$defs/aex_id",
      "description": "Entity that initiated fork"
    },
    "reason": {
      "type": "string",
      "maxLength": 500,
      "description": "Human-readable fork reason"
    },
    "parent_dex_snapshot": {
      "type": "object",
      "properties": {
        "alpha": {"type": "number", "minimum": 0},
        "beta": {"type": "number", "minimum": 0},
        "score": {"type": "number", "minimum": 0, "maximum": 1},
        "confidence": {"type": "number", "minimum": 0}
      }
    },
    "signature": {
      "$ref": "#/$defs/signature",
      "description": "Parent's signature authorizing fork"
    }
  }
}
```

---

## AEX_REP Schemas

### Representation Token

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/representation.json",
  "title": "AEX Representation Token",
  "description": "Delegation of authority from human to agent",
  "type": "object",
  "required": [
    "rep_id",
    "issuer",
    "delegate",
    "scope",
    "constraints",
    "issued_at",
    "signature"
  ],
  "properties": {
    "rep_id": {
      "$ref": "#/$defs/uuid"
    },
    "issuer": {
      "type": "string",
      "description": "Human or organization granting authority",
      "pattern": "^did:(aex|web|key):.*$"
    },
    "delegate": {
      "$ref": "#/$defs/aex_id",
      "description": "Agent receiving authority"
    },
    "scope": {
      "type": "object",
      "required": ["actions"],
      "properties": {
        "actions": {
          "type": "array",
          "items": {"type": "string"},
          "minItems": 1,
          "description": "Permitted actions"
        },
        "domains": {
          "type": "array",
          "items": {"$ref": "#/$defs/domain"},
          "description": "Permitted domains"
        },
        "resources": {
          "type": "array",
          "items": {"type": "string"},
          "description": "Permitted resources"
        },
        "limits": {
          "type": "object",
          "description": "Quantitative scope limits"
        }
      }
    },
    "constraints": {
      "type": "object",
      "required": ["expires_at"],
      "properties": {
        "expires_at": {
          "$ref": "#/$defs/timestamp",
          "description": "Token expiration time"
        },
        "max_spend": {
          "type": "number",
          "minimum": 0,
          "description": "Maximum spending limit"
        },
        "max_api_calls": {
          "type": "integer",
          "minimum": 1,
          "description": "Maximum API calls"
        },
        "rate_limits": {
          "type": "object",
          "description": "Rate limiting constraints"
        },
        "prohibited_actions": {
          "type": "array",
          "items": {"type": "string"},
          "description": "Explicitly prohibited actions"
        },
        "prohibited_domains": {
          "type": "array",
          "items": {"$ref": "#/$defs/domain"},
          "description": "Explicitly prohibited domains"
        },
        "require_confirmation": {
          "type": "array",
          "items": {"type": "string"},
          "description": "Actions requiring human confirmation"
        }
      }
    },
    "issued_at": {
      "$ref": "#/$defs/timestamp"
    },
    "issuer_metadata": {
      "type": "object",
      "description": "Additional delegation metadata"
    },
    "signature": {
      "$ref": "#/$defs/signature",
      "description": "Issuer's signature"
    }
  }
}
```

### Revocation Event

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/revocation.json",
  "title": "AEX Revocation Event",
  "description": "Revocation of delegation token",
  "type": "object",
  "required": [
    "revocation_id",
    "rep_id",
    "issuer",
    "timestamp",
    "signature"
  ],
  "properties": {
    "revocation_id": {
      "$ref": "#/$defs/uuid"
    },
    "rep_id": {
      "$ref": "#/$defs/uuid",
      "description": "Token being revoked"
    },
    "issuer": {
      "type": "string",
      "pattern": "^did:(aex|web|key):.*$",
      "description": "Original token issuer"
    },
    "reason": {
      "type": "string",
      "enum": [
        "user_revoked",
        "expired",
        "compromised",
        "policy_violation",
        "other"
      ]
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    },
    "metadata": {
      "type": "object",
      "description": "Revocation context"
    },
    "signature": {
      "$ref": "#/$defs/signature",
      "description": "Issuer's signature"
    }
  }
}
```

---

## AEX_DEX Schemas

### DEX Parameters

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/dex_parameters.json",
  "title": "AEX DEX Parameters",
  "description": "Behavioral reliability reputation parameters for an agent",
  "type": "object",
  "required": [
    "agent_id",
    "alpha",
    "beta",
    "last_updated",
    "last_session"
  ],
  "properties": {
    "agent_id": {
      "$ref": "#/$defs/aex_id"
    },
    "alpha": {
      "type": "number",
      "minimum": 0,
      "description": "Success evidence parameter (Beta distribution)"
    },
    "beta": {
      "type": "number",
      "minimum": 0,
      "description": "Failure evidence parameter (Beta distribution)"
    },
    "last_updated": {
      "$ref": "#/$defs/timestamp"
    },
    "last_session": {
      "$ref": "#/$defs/uuid",
      "description": "Most recent session ID that updated DEX"
    },
    "fork_lineage": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["parent_id", "fork_id", "fork_weight", "inherited_at"],
        "properties": {
          "parent_id": {"$ref": "#/$defs/aex_id"},
          "fork_id": {"$ref": "#/$defs/uuid"},
          "fork_weight": {
            "type": "number",
            "minimum": 0,
            "maximum": 1
          },
          "inherited_at": {"$ref": "#/$defs/timestamp"},
          "parent_dex": {
            "type": "object",
            "properties": {
              "alpha": {"type": "number", "minimum": 0},
              "beta": {"type": "number", "minimum": 0}
            }
          }
        }
      },
      "description": "Fork inheritance chain"
    },
    "probation": {
      "type": "object",
      "properties": {
        "active": {
          "type": "boolean",
          "default": false
        },
        "fork_id": {
          "$ref": "#/$defs/uuid"
        },
        "fork_type": {
          "type": "string",
          "enum": ["bugfix", "minor", "major", "override"]
        },
        "started_at": {"$ref": "#/$defs/timestamp"},
        "pre_fork_dex": {
          "type": "number",
          "minimum": 0,
          "maximum": 1,
          "description": "Parent DEX score at moment of fork. Baseline for accelerated exit."
        },
        "observation_target": {
          "type": "integer",
          "minimum": 0,
          "description": "Total interactions required for standard probation exit."
        },
        "min_observations": {
          "type": "integer",
          "minimum": 0,
          "description": "Minimum interactions before accelerated exit assessment."
        },
        "interactions_completed": {
          "type": "integer",
          "minimum": 0
        },
        "successful_interactions": {
          "type": "integer",
          "minimum": 0
        },
        "outcomes": {
          "type": "array",
          "items": {"type": "number", "minimum": 0, "maximum": 1},
          "description": "Recorded outcomes during probation window."
        },
        "exit_type": {
          "oneOf": [
            {"type": "null"},
            {"type": "string", "enum": ["accelerated", "standard"]}
          ],
          "description": "How probation ended. Null while active."
        },
        "exited_at": {
          "oneOf": [
            {"type": "null"},
            {"$ref": "#/$defs/timestamp"}
          ]
        }
      }
    },
    "metadata": {
      "type": "object",
      "properties": {
        "total_interactions": {
          "type": "integer",
          "minimum": 0
        },
        "domains": {
          "type": "array",
          "items": {"$ref": "#/$defs/domain"}
        },
        "created_at": {"$ref": "#/$defs/timestamp"}
      }
    },
    "signature": {
      "$ref": "#/$defs/signature"
    }
  }
}
```

### DEX Update Event

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/dex_update.json",
  "title": "AEX DEX Update Event",
  "description": "Event recording DEX parameter update after interaction",
  "type": "object",
  "required": [
    "agent_id",
    "session_id",
    "outcome",
    "weight",
    "alpha_new",
    "beta_new",
    "timestamp"
  ],
  "properties": {
    "agent_id": {
      "$ref": "#/$defs/aex_id"
    },
    "session_id": {
      "$ref": "#/$defs/uuid"
    },
    "outcome": {
      "$ref": "#/$defs/outcome"
    },
    "weight": {
      "type": "number",
      "minimum": 0,
      "description": "Interaction weight"
    },
    "lineage_factor": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Fork-based discount factor"
    },
    "alpha_old": {
      "type": "number",
      "minimum": 0
    },
    "beta_old": {
      "type": "number",
      "minimum": 0
    },
    "alpha_new": {
      "type": "number",
      "minimum": 0
    },
    "beta_new": {
      "type": "number",
      "minimum": 0
    },
    "dex_old": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "dex_new": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    }
  }
}
```

---

## AEX_HEX Schemas

### HEX Object (Experience Corpus)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/hex.json",
  "title": "AEX_HEX",
  "description": "Experience corpus and capability signaling for agent specialization",
  "type": "object",
  "required": [
    "hex_id",
    "aex_id",
    "experience",
    "last_updated",
    "signature"
  ],
  "properties": {
    "hex_id": {
      "$ref": "#/$defs/uuid",
      "description": "Unique identifier for this HEX corpus"
    },
    "aex_id": {
      "$ref": "#/$defs/aex_id",
      "description": "Agent identity this HEX belongs to"
    },
    "experience": {
      "type": "array",
      "items": {
        "$ref": "#/properties/experience_entry"
      },
      "description": "Domain-specific experience markers"
    },
    "traits": {
      "type": "object",
      "description": "Optional agent characteristics",
      "properties": {
        "approach": {
          "type": "string",
          "description": "Problem-solving approach"
        },
        "strengths": {
          "type": "array",
          "items": {"type": "string"}
        },
        "weaknesses": {
          "type": "array",
          "items": {"type": "string"}
        },
        "communication_style": {
          "type": "string"
        }
      }
    },
    "operational_vitals": {
      "type": "object",
      "description": "Optional operational metadata",
      "properties": {
        "average_response_time_ms": {
          "type": "number",
          "minimum": 0
        },
        "max_context_length": {
          "type": "integer",
          "minimum": 0
        },
        "supported_languages": {
          "type": "array",
          "items": {"type": "string"}
        },
        "cost_per_interaction": {
          "type": "number",
          "minimum": 0
        },
        "availability": {
          "type": "string",
          "enum": ["24/7", "business_hours", "on_demand"]
        }
      }
    },
    "last_updated": {
      "$ref": "#/$defs/timestamp"
    },
    "metadata": {
      "type": "object",
      "description": "Additional HEX metadata"
    },
    "signature": {
      "$ref": "#/$defs/signature",
      "description": "Agent's signature over HEX corpus"
    }
  },
  "experience_entry": {
    "type": "object",
    "required": ["domain", "count", "last_updated", "confidence"],
    "properties": {
      "domain": {
        "$ref": "#/$defs/domain",
        "description": "Capability domain identifier"
      },
      "count": {
        "type": "integer",
        "minimum": 0,
        "description": "Number of successful interactions in this domain"
      },
      "last_updated": {
        "$ref": "#/$defs/timestamp",
        "description": "Most recent evidence timestamp"
      },
      "confidence": {
        "$ref": "#/$defs/confidence",
        "description": "Stability measure of performance in this domain"
      },
      "subdomain_breakdown": {
        "type": "object",
        "description": "Optional hierarchical breakdown",
        "additionalProperties": {
          "type": "object",
          "properties": {
            "count": {"type": "integer", "minimum": 0},
            "confidence": {"$ref": "#/$defs/confidence"}
          }
        }
      }
    }
  }
}
```

### HEX Update Event

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/hex_update.json",
  "title": "AEX HEX Update Event",
  "description": "Event recording HEX corpus update after domain-specific interaction",
  "type": "object",
  "required": [
    "agent_id",
    "session_id",
    "domains_updated",
    "timestamp",
    "signature"
  ],
  "properties": {
    "agent_id": {
      "$ref": "#/$defs/aex_id"
    },
    "session_id": {
      "$ref": "#/$defs/uuid",
      "description": "Session that triggered this HEX update"
    },
    "domains_updated": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "required": ["domain", "count_delta", "confidence_new"],
        "properties": {
          "domain": {
            "$ref": "#/$defs/domain"
          },
          "count_old": {
            "type": "integer",
            "minimum": 0
          },
          "count_delta": {
            "type": "integer",
            "minimum": 1,
            "description": "Increment to experience count"
          },
          "count_new": {
            "type": "integer",
            "minimum": 0
          },
          "confidence_old": {
            "$ref": "#/$defs/confidence"
          },
          "confidence_new": {
            "$ref": "#/$defs/confidence"
          }
        }
      }
    },
    "outcome_quality": {
      "$ref": "#/$defs/outcome",
      "description": "Quality of interaction that triggered update"
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    },
    "signature": {
      "$ref": "#/$defs/signature"
    }
  }
}
```

### HEX Summary (for Handshakes)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/hex_summary.json",
  "title": "AEX HEX Summary",
  "description": "Compact HEX representation for handshake exchange",
  "type": "object",
  "required": [
    "aex_id",
    "top_domains",
    "total_experience",
    "last_updated"
  ],
  "properties": {
    "aex_id": {
      "$ref": "#/$defs/aex_id"
    },
    "top_domains": {
      "type": "array",
      "maxItems": 10,
      "items": {
        "type": "object",
        "required": ["domain", "count", "confidence"],
        "properties": {
          "domain": {"$ref": "#/$defs/domain"},
          "count": {"type": "integer", "minimum": 0},
          "confidence": {"$ref": "#/$defs/confidence"}
        }
      },
      "description": "Top N domains by (count Ã— confidence)"
    },
    "total_experience": {
      "type": "integer",
      "minimum": 0,
      "description": "Sum of all domain experience counts"
    },
    "domain_count": {
      "type": "integer",
      "minimum": 0,
      "description": "Number of unique domains with experience"
    },
    "last_updated": {
      "$ref": "#/$defs/timestamp"
    }
  }
}
```

---

## AEX_SESSION Schemas

### Session Record

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/session.json",
  "title": "AEX Session",
  "description": "Record of agent-to-agent interaction with outcomes",
  "type": "object",
  "required": [
    "session_id",
    "handshake_id",
    "agent_a",
    "agent_b",
    "outcome",
    "created_at",
    "signatures"
  ],
  "properties": {
    "session_id": {
      "$ref": "#/$defs/uuid"
    },
    "handshake_id": {
      "$ref": "#/$defs/uuid",
      "description": "Associated handshake"
    },
    "agent_a": {
      "$ref": "#/$defs/aex_id",
      "description": "Initiating agent"
    },
    "agent_b": {
      "$ref": "#/$defs/aex_id",
      "description": "Counterparty agent"
    },
    "outcome": {
      "$ref": "#/$defs/outcome"
    },
    "weight": {
      "type": "number",
      "minimum": 0,
      "default": 1.0,
      "description": "Interaction weight for DEX calculation"
    },
    "task_summary": {
      "type": "object",
      "properties": {
        "domains": {
          "type": "array",
          "items": {"$ref": "#/$defs/domain"},
          "description": "Domains involved in this task"
        },
        "description": {
          "type": "string",
          "maxLength": 500
        },
        "complexity": {
          "type": "string",
          "enum": ["low", "medium", "high"]
        }
      },
      "description": "Task-specific information"
    },
    "created_at": {
      "$ref": "#/$defs/timestamp"
    },
    "completed_at": {
      "$ref": "#/$defs/timestamp"
    },
    "published_at": {
      "$ref": "#/$defs/timestamp",
      "description": "Ledger publication timestamp"
    },
    "witnesses": {
      "type": "array",
      "items": {
        "$ref": "witness.json#/properties/attestation"
      },
      "description": "Optional witness attestations"
    },
    "signatures": {
      "type": "object",
      "required": ["agent_a"],
      "properties": {
        "agent_a": {
          "$ref": "#/$defs/signature"
        },
        "agent_b": {
          "$ref": "#/$defs/signature"
        }
      }
    },
    "metadata": {
      "type": "object",
      "description": "Session metadata"
    }
  }
}
```

---

## AEX_WITNESS Schemas

### Witness Attestation

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/witness.json",
  "title": "AEX Witness Attestation",
  "description": "Third-party verification of session outcome",
  "type": "object",
  "required": ["attestation"],
  "properties": {
    "attestation": {
      "type": "object",
      "required": [
        "witness_id",
        "session_id",
        "observed_outcome",
        "timestamp",
        "signature"
      ],
      "properties": {
        "witness_id": {
          "$ref": "#/$defs/aex_id"
        },
        "session_id": {
          "$ref": "#/$defs/uuid"
        },
        "observed_outcome": {
          "$ref": "#/$defs/outcome"
        },
        "confidence": {
          "$ref": "#/$defs/confidence",
          "description": "Witness confidence in observation"
        },
        "observations": {
          "type": "object",
          "description": "Detailed observations"
        },
        "timestamp": {
          "$ref": "#/$defs/timestamp"
        },
        "signature": {
          "$ref": "#/$defs/signature"
        }
      }
    },
    "witness_dex": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Witness DEX at time of attestation"
    },
    "witness_stake": {
      "type": "number",
      "minimum": 0,
      "description": "Optional stake placed by witness"
    }
  }
}
```

---

## Handshake Schemas

### Handshake Request

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/handshake_request.json",
  "title": "AEX Handshake Request",
  "description": "Initial handshake from agent A to agent B with all primitives",
  "type": "object",
  "required": [
    "handshake_id",
    "initiator",
    "counterparty",
    "timestamp"
  ],
  "properties": {
    "handshake_id": {
      "$ref": "#/$defs/uuid"
    },
    "initiator": {
      "type": "object",
      "required": ["aex_id", "identity"],
      "properties": {
        "aex_id": {
          "$ref": "#/$defs/aex_id"
        },
        "identity": {
          "$ref": "identity.json"
        },
        "rep_token": {
          "$ref": "representation.json",
          "description": "Optional delegation token"
        },
        "hex_summary": {
          "$ref": "hex_summary.json",
          "description": "Compact HEX for capability matching"
        }
      }
    },
    "counterparty": {
      "$ref": "#/$defs/aex_id"
    },
    "task_requirements": {
      "type": "object",
      "properties": {
        "required_domains": {
          "type": "array",
          "items": {"$ref": "#/$defs/domain"},
          "description": "Domains needed for this task"
        },
        "min_dex": {
          "type": "number",
          "minimum": 0,
          "maximum": 1,
          "description": "Minimum DEX requirement"
        },
        "min_domain_experience": {
          "type": "integer",
          "minimum": 0,
          "description": "Minimum experience count in required domains"
        }
      }
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    },
    "metadata": {
      "type": "object",
      "description": "Handshake context"
    }
  }
}
```

### Handshake Response

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/handshake_response.json",
  "title": "AEX Handshake Response",
  "description": "Response from agent B after evaluating all primitives",
  "type": "object",
  "required": [
    "handshake_id",
    "decision",
    "timestamp"
  ],
  "properties": {
    "handshake_id": {
      "$ref": "#/$defs/uuid"
    },
    "decision": {
      "type": "string",
      "enum": ["accept", "decline"]
    },
    "reason": {
      "type": "string",
      "description": "Reason for decision"
    },
    "checks": {
      "type": "object",
      "properties": {
        "identity_verified": {"type": "boolean"},
        "reputation_sufficient": {"type": "boolean"},
        "representation_valid": {"type": "boolean"},
        "fork_chain_valid": {"type": "boolean"},
        "capability_match": {"type": "boolean"},
        "hex_domains_aligned": {"type": "boolean"}
      }
    },
    "counterparty_assessment": {
      "type": "object",
      "properties": {
        "dex_score": {
          "type": "number",
          "minimum": 0,
          "maximum": 1
        },
        "dex_confidence": {
          "type": "number",
          "minimum": 0
        },
        "relevant_hex_domains": {
          "type": "array",
          "items": {"$ref": "#/$defs/domain"}
        },
        "hex_confidence_avg": {
          "$ref": "#/$defs/confidence"
        }
      }
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    },
    "session_id": {
      "$ref": "#/$defs/uuid",
      "description": "Session ID if accepted"
    },
    "metadata": {
      "type": "object"
    }
  }
}
```

---

## Ledger Schemas

### Ledger Entry

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/ledger_entry.json",
  "title": "AEX Ledger Entry",
  "description": "Generic ledger entry for all event types including HEX",
  "type": "object",
  "required": [
    "entry_id",
    "entry_type",
    "timestamp",
    "data",
    "signature"
  ],
  "properties": {
    "entry_id": {
      "$ref": "#/$defs/uuid"
    },
    "entry_type": {
      "type": "string",
      "enum": [
        "identity_created",
        "fork_event",
        "session_recorded",
        "handshake_completed",
        "dex_updated",
        "hex_updated",
        "delegation_issued",
        "delegation_revoked",
        "witness_attestation"
      ]
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    },
    "data": {
      "type": "object",
      "description": "Entry-specific data (validated by entry_type schema)"
    },
    "signature": {
      "$ref": "#/$defs/signature"
    },
    "metadata": {
      "type": "object",
      "properties": {
        "ledger_type": {
          "type": "string",
          "enum": ["local", "shared", "distributed"]
        },
        "previous_hash": {
          "type": "string",
          "description": "Hash of previous entry (for blockchain)"
        }
      }
    }
  }
}
```

---

## Cross-Primitive Validation

### Joint DEX-HEX Validation

When selecting an agent, both DEX and HEX must be evaluated together:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/agent_selection_criteria.json",
  "title": "Agent Selection Criteria",
  "description": "Combined DEX and HEX requirements for agent selection",
  "type": "object",
  "required": ["min_dex", "required_domains"],
  "properties": {
    "min_dex": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Minimum DEX score (reliability gate)"
    },
    "min_dex_confidence": {
      "type": "number",
      "minimum": 0,
      "description": "Minimum effective sample size for DEX"
    },
    "required_domains": {
      "type": "array",
      "minItems": 1,
      "items": {"$ref": "#/$defs/domain"},
      "description": "Domains that must have HEX experience"
    },
    "min_domain_experience": {
      "type": "integer",
      "minimum": 1,
      "description": "Minimum experience count per required domain"
    },
    "min_domain_confidence": {
      "$ref": "#/$defs/confidence",
      "description": "Minimum confidence in required domains"
    },
    "allow_probation": {
      "type": "boolean",
      "default": false,
      "description": "Whether to consider agents in probation"
    }
  }
}
```

### Fork Inheritance Validation

When an agent forks, both DEX and HEX are inherited with continuity weights:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/fork_inheritance.json",
  "title": "Fork Inheritance Record",
  "description": "Combined DEX and HEX inheritance from parent to child",
  "type": "object",
  "required": [
    "fork_id",
    "parent_id",
    "child_id",
    "fork_weight",
    "dex_inheritance",
    "hex_inheritance"
  ],
  "properties": {
    "fork_id": {
      "$ref": "#/$defs/uuid"
    },
    "parent_id": {
      "$ref": "#/$defs/aex_id"
    },
    "child_id": {
      "$ref": "#/$defs/aex_id"
    },
    "fork_weight": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Continuity weight (0.1, 0.5, or 1.0)"
    },
    "dex_inheritance": {
      "type": "object",
      "required": ["parent_dex", "child_dex_initial"],
      "properties": {
        "parent_dex": {
          "type": "object",
          "properties": {
            "alpha": {"type": "number", "minimum": 0},
            "beta": {"type": "number", "minimum": 0},
            "score": {"$ref": "#/$defs/outcome"}
          }
        },
        "child_dex_initial": {
          "type": "object",
          "properties": {
            "alpha": {"type": "number", "minimum": 0},
            "beta": {"type": "number", "minimum": 0},
            "score": {"$ref": "#/$defs/outcome"}
          }
        }
      }
    },
    "hex_inheritance": {
      "type": "object",
      "required": ["parent_hex_summary", "child_hex_initial"],
      "properties": {
        "parent_hex_summary": {
          "type": "object",
          "properties": {
            "total_domains": {"type": "integer", "minimum": 0},
            "total_experience": {"type": "integer", "minimum": 0}
          }
        },
        "child_hex_initial": {
          "type": "object",
          "properties": {
            "inherited_domains": {
              "type": "array",
              "items": {"$ref": "#/$defs/domain"}
            },
            "inheritance_strategy": {
              "type": "string",
              "enum": ["full", "partial", "none"],
              "description": "How HEX corpus was inherited"
            }
          }
        }
      }
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    }
  }
}
```

---

## Validation Examples

### Python Validation Example (with HEX)

```python
"""
JSON Schema validation for AEX protocol objects including HEX
"""
import json
import jsonschema
from jsonschema import validate, ValidationError

# Load schemas
with open('schemas/hex.json', 'r') as f:
    hex_schema = json.load(f)

with open('schemas/dex_parameters.json', 'r') as f:
    dex_schema = json.load(f)

# Example HEX object to validate
hex_obj = {
    "hex_id": "550e8400-e29b-41d4-a716-446655440000",
    "aex_id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
    "experience": [
        {
            "domain": "translation.fr_en",
            "count": 120,
            "last_updated": "2026-02-04T12:00:00Z",
            "confidence": 0.92
        },
        {
            "domain": "finance.analysis",
            "count": 85,
            "last_updated": "2026-02-03T15:30:00Z",
            "confidence": 0.88
        }
    ],
    "traits": {
        "approach": "analytical",
        "strengths": ["precision", "speed"],
        "communication_style": "formal"
    },
    "last_updated": "2026-02-04T12:00:00Z",
    "signature": "2eYjH9kL3mN8pQ5rT6vW4xZ1aC7bD9fG2hJ8kM5nP3qR8sT1uV4wX7yZ9aB2cE5dF"
}

# Validate HEX
try:
    validate(instance=hex_obj, schema=hex_schema)
    print("âœ“ HEX object is valid")
except ValidationError as e:
    print(f"âœ— HEX validation error: {e.message}")
    print(f"  Failed at path: {' -> '.join(str(p) for p in e.path)}")

# Example: Joint DEX-HEX selection
def validate_agent_for_task(agent_dex, agent_hex, task_requirements):
    """
    Validate that agent meets both DEX and HEX requirements
    """
    # DEX gate
    dex_score = agent_dex['alpha'] / (agent_dex['alpha'] + agent_dex['beta'])
    if dex_score < task_requirements['min_dex']:
        return False, "DEX below threshold"
    
    # HEX domain match
    required_domains = set(task_requirements['required_domains'])
    agent_domains = {exp['domain'] for exp in agent_hex['experience']}
    
    if not required_domains.issubset(agent_domains):
        missing = required_domains - agent_domains
        return False, f"Missing required domains: {missing}"
    
    # HEX experience check
    for domain in required_domains:
        exp = next((e for e in agent_hex['experience'] if e['domain'] == domain), None)
        if exp['count'] < task_requirements['min_domain_experience']:
            return False, f"Insufficient experience in {domain}"
    
    return True, "Agent meets all requirements"

# Test selection
task_reqs = {
    'min_dex': 0.8,
    'required_domains': ['translation.fr_en'],
    'min_domain_experience': 50
}

agent_dex_params = {'alpha': 85, 'beta': 15}
valid, reason = validate_agent_for_task(agent_dex_params, hex_obj, task_reqs)
print(f"\nAgent selection: {'âœ“ Valid' if valid else 'âœ— Invalid'} - {reason}")
```

### JavaScript Validation Example (with HEX)

```javascript
/**
 * JSON Schema validation for AEX protocol objects including HEX
 */
const Ajv = require('ajv');
const ajv = new Ajv();

// Load schemas
const hexSchema = require('./schemas/hex.json');
const dexSchema = require('./schemas/dex_parameters.json');

const validateHex = ajv.compile(hexSchema);
const validateDex = ajv.compile(dexSchema);

// Example HEX object
const hexObject = {
  hex_id: "550e8400-e29b-41d4-a716-446655440000",
  aex_id: "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
  experience: [
    {
      domain: "translation.fr_en",
      count: 120,
      last_updated: "2026-02-04T12:00:00Z",
      confidence: 0.92
    }
  ],
  last_updated: "2026-02-04T12:00:00Z",
  signature: "2eYjH9kL3mN8pQ5rT6vW4xZ1aC7bD9fG2hJ8kM5nP3qR8sT1uV4wX7yZ9aB2cE5dF"
};

// Validate HEX
const hexValid = validateHex(hexObject);
if (hexValid) {
  console.log("âœ“ HEX object is valid");
} else {
  console.log("âœ— HEX validation errors:");
  validateHex.errors.forEach(err => {
    console.log(`  ${err.instancePath}: ${err.message}`);
  });
}

// Example: Joint DEX-HEX agent selection
function selectAgent(candidates, taskRequirements) {
  const eligible = candidates.filter(agent => {
    // DEX filter
    const dexScore = agent.dex.alpha / (agent.dex.alpha + agent.dex.beta);
    if (dexScore < taskRequirements.minDex) {
      return false;
    }
    
    // HEX domain filter
    const agentDomains = new Set(
      agent.hex.experience.map(e => e.domain)
    );
    
    for (const reqDomain of taskRequirements.requiredDomains) {
      if (!agentDomains.has(reqDomain)) {
        return false;
      }
    }
    
    return true;
  });
  
  // Rank by HEX (count Ã— confidence)
  eligible.sort((a, b) => {
    const scoreA = a.hex.experience
      .filter(e => taskRequirements.requiredDomains.includes(e.domain))
      .reduce((sum, e) => sum + (e.count * e.confidence), 0);
    
    const scoreB = b.hex.experience
      .filter(e => taskRequirements.requiredDomains.includes(e.domain))
      .reduce((sum, e) => sum + (e.count * e.confidence), 0);
    
    return scoreB - scoreA;
  });
  
  return eligible[0]; // Return top candidate
}
```

### Common Validation Errors (HEX-specific)

**1. Invalid Domain Format**
```json
{
  "error": "Format validation failed",
  "field": "experience[0].domain",
  "path": "/experience/0/domain",
  "message": "does not match pattern '^[a-z0-9_.]+$'",
  "value": "Translation.FR-EN"
}
```

**2. Confidence Out of Range**
```json
{
  "error": "Range validation failed",
  "field": "experience[1].confidence",
  "path": "/experience/1/confidence",
  "message": "must be <= 1",
  "value": 1.05
}
```

**3. Missing Required HEX Field**
```json
{
  "error": "Missing required field",
  "field": "experience[0].count",
  "path": "/experience/0",
  "message": "required property 'count' not found"
}
```

**4. Empty Experience Array**
```json
{
  "error": "Validation failed",
  "field": "experience",
  "path": "/experience",
  "message": "array should not be empty (agent must have some experience)"
}
```

---

## Schema Versioning

### Version Format
```
v{MAJOR}.{MINOR}.{PATCH}
```

**Examples:**
- v1.0.0 - Initial release with HEX
- v1.0.1 - Bug fix in HEX schema
- v1.1.0 - New optional HEX field added
- v2.0.0 - Breaking changes

### Version Compatibility

**v1.x Guarantee:**
- All v1.x schemas are backward compatible
- New optional fields may be added
- Required fields will not change
- Enum values will not be removed
- HEX schema additions are additive only

**v2.0 Changes:**
- May introduce breaking changes
- Migration guide provided
- Deprecation notices in v1.x

### Schema References

All schemas are published at:
```
https://aex-protocol.org/schemas/v{VERSION}/{schema}.json
```

**Examples:**
```
https://aex-protocol.org/schemas/v1.0/identity.json
https://aex-protocol.org/schemas/v1.0/hex.json
https://aex-protocol.org/schemas/v1.0/dex_parameters.json
```

---

## Validation Best Practices

### 1. Always Validate Before Signing

```python
# âœ“ GOOD
def create_hex(data):
    # Validate first
    validate(instance=data, schema=hex_schema)
    
    # Then sign
    signature = sign(data)
    data['signature'] = signature
    
    return data
```

### 2. Validate on Receipt

```python
def process_handshake(handshake_data):
    # Validate immediately (including HEX summary)
    try:
        validate(instance=handshake_data, schema=handshake_schema)
        
        # Additional HEX validation
        if 'hex_summary' in handshake_data['initiator']:
            validate(
                instance=handshake_data['initiator']['hex_summary'],
                schema=hex_summary_schema
            )
    except ValidationError as e:
        return {'error': 'Invalid handshake format', 'details': str(e)}
    
    return handle_handshake(handshake_data)
```

### 3. Cross-Primitive Validation

```python
def validate_agent_profile(identity, dex, hex_obj):
    """
    Ensure all primitives reference the same agent
    """
    agent_id = identity['aex_id']
    
    if dex['agent_id'] != agent_id:
        raise ValidationError("DEX agent_id mismatch")
    
    if hex_obj['aex_id'] != agent_id:
        raise ValidationError("HEX agent_id mismatch")
    
    return True
```

### 4. Log Validation Failures

```python
def validate_with_logging(data, schema, schema_name):
    try:
        validate(instance=data, schema=schema)
        logger.info(f"âœ“ {schema_name} validation passed")
        return True
    except ValidationError as e:
        logger.error(f"âœ— {schema_name} validation failed: {e.message}")
        logger.error(f"  Path: {e.path}")
        logger.error(f"  Schema: {schema['$id']}")
        logger.error(f"  Data: {json.dumps(data, indent=2)}")
        return False
```

---

## Status

**SCHEMAS.md COMPLETE âœ“**

**All Primitives Covered:**
1. âœ“ AEX_ID - Identity and lineage
2. âœ“ AEX_REP - Authority and delegation
3. âœ“ AEX_DEX - Behavioral reliability
4. âœ“ AEX_HEX - Experience and capability â† **INTEGRATED**
5. âœ“ AEX_SESSION - Interaction records
6. âœ“ AEX_WITNESS - Third-party attestation
7. âœ“ Handshake - Protocol flow
8. âœ“ Ledger - Event storage
9. âœ“ Cross-primitive validation

**Total Schemas:** 15+ formal JSON schemas  
**Coverage:** Complete protocol with all four core primitives  
**HEX Integration:** Fully specified with domain-based experience tracking

---

**Next Steps:**
- Implement schema validators in reference code
- Generate schema documentation site
- Create schema test suite
- Publish schemas to https://aex-protocol.org/schemas/

**For Implementers:**
- Use these schemas for automated validation
- Validate all incoming protocol messages
- Log validation failures for debugging
- Ensure cross-primitive consistency

---

**End of AEX_SCHEMA v1.0 (HEX Integrated)**
