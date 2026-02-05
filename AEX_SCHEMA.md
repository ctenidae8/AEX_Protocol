# AEX Protocol v1.0 - JSON Schemas
## Formal Data Structure Definitions and Validation

**Last Updated:** February 4, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Schema Conventions](#schema-conventions)
3. [AEX_ID Schemas](#aex_id-schemas)
4. [AEX_REP Schemas](#aex_rep-schemas)
5. [AEX_DEX Schemas](#aex_dex-schemas)
6. [AEX_SESSION Schemas](#aex_session-schemas)
7. [AEX_WITNESS Schemas](#aex_witness-schemas)
8. [Handshake Schemas](#handshake-schemas)
9. [Ledger Schemas](#ledger-schemas)
10. [Validation Examples](#validation-examples)

---

## Overview

This document provides **JSON Schema definitions** for all AEX protocol data structures.

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
      "pattern": "^did:aex:[a-zA-Z0-9._-]+$",
      "description": "AEX Decentralized Identifier"
    },
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$",
      "description": "UUID v4 identifier"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$",
      "description": "ISO 8601 timestamp in UTC"
    },
    "signature": {
      "type": "string",
      "pattern": "^[A-Za-z0-9+/]+=*$",
      "minLength": 64,
      "description": "Base64-encoded cryptographic signature"
    },
    "public_key": {
      "type": "string",
      "pattern": "^[A-Za-z0-9+/]+=*$",
      "minLength": 32,
      "description": "Base64-encoded public key"
    },
    "outcome": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0,
      "description": "Normalized outcome value [0,1]"
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
  "description": "Complete agent identity with lineage",
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
      "required": ["parent", "fork_history"],
      "properties": {
        "parent": {
          "oneOf": [
            {"$ref": "#/$defs/aex_id"},
            {"type": "null"}
          ],
          "description": "Parent identity (null for root)"
        },
        "fork_history": {
          "type": "array",
          "items": {
            "$ref": "#/properties/fork_event"
          },
          "description": "Chronological fork events"
        }
      }
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
  "fork_event": {
    "type": "object",
    "required": [
      "fork_id",
      "parent_id",
      "child_id",
      "fork_type",
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
        "enum": ["bugfix", "major_rewrite", "hard_override", "custom"]
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
      "metadata": {
        "type": "object",
        "description": "Fork-specific metadata"
      },
      "signature": {
        "$ref": "#/$defs/signature",
        "description": "Parent's signature authorizing fork"
      }
    }
  },
  "$defs": {
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[a-zA-Z0-9._-]+$"
    },
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
    },
    "signature": {
      "type": "string",
      "pattern": "^[A-Za-z0-9+/]+=*$",
      "minLength": 64
    },
    "public_key": {
      "type": "string",
      "pattern": "^[A-Za-z0-9+/]+=*$",
      "minLength": 32
    }
  }
}
```

### Example Identity
```json
{
  "aex_id": "did:aex:z6MkpTHR8JNhKnFqBVqE9ZsWRX1pNk3TxQ",
  "public_key": "MCowBQYDK2VwAyEA8x2F+3gH7kL9mN4pQ6rS1tU...",
  "created_at": "2026-02-04T20:00:00Z",
  "lineage": {
    "parent": null,
    "fork_history": []
  },
  "metadata": {
    "agent_class": "research_assistant",
    "version": "1.0",
    "capabilities": ["search", "analyze", "summarize"]
  },
  "signature": "c2lnbmF0dXJlX2V4YW1wbGVfZGF0YQ=="
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
          "items": {"type": "string"},
          "description": "Permitted domains"
        },
        "resources": {
          "type": "array",
          "items": {"type": "string"},
          "description": "Permitted resources"
        }
      }
    },
    "constraints": {
      "type": "object",
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
    "metadata": {
      "type": "object",
      "description": "Additional delegation metadata"
    },
    "signature": {
      "$ref": "#/$defs/signature",
      "description": "Issuer's signature"
    }
  },
  "$defs": {
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    },
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[a-zA-Z0-9._-]+$"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
    },
    "signature": {
      "type": "string",
      "pattern": "^[A-Za-z0-9+/]+=*$",
      "minLength": 64
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
  },
  "$defs": {
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
    },
    "signature": {
      "type": "string",
      "pattern": "^[A-Za-z0-9+/]+=*$",
      "minLength": 64
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
  "description": "Reputation parameters for an agent",
  "type": "object",
  "required": [
    "aex_id",
    "alpha",
    "beta",
    "dex",
    "n_eff",
    "last_updated"
  ],
  "properties": {
    "aex_id": {
      "$ref": "#/$defs/aex_id"
    },
    "alpha": {
      "type": "number",
      "minimum": 0,
      "description": "Success evidence parameter"
    },
    "beta": {
      "type": "number",
      "minimum": 0,
      "description": "Failure evidence parameter"
    },
    "dex": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "DEX score = α/(α+β)"
    },
    "n_eff": {
      "type": "number",
      "minimum": 0,
      "description": "Effective sample size = α+β"
    },
    "variance": {
      "type": "number",
      "minimum": 0,
      "description": "Variance = αβ/((α+β)²(α+β+1))"
    },
    "confidence_interval": {
      "type": "object",
      "properties": {
        "lower": {
          "type": "number",
          "minimum": 0,
          "maximum": 1
        },
        "upper": {
          "type": "number",
          "minimum": 0,
          "maximum": 1
        },
        "confidence_level": {
          "type": "number",
          "minimum": 0,
          "maximum": 1,
          "default": 0.95
        }
      }
    },
    "probation": {
      "type": "object",
      "properties": {
        "active": {
          "type": "boolean",
          "default": false
        },
        "probation_until": {
          "$ref": "#/$defs/timestamp"
        },
        "reason": {
          "type": "string",
          "enum": ["recent_fork", "low_confidence", "degradation"]
        }
      }
    },
    "last_updated": {
      "$ref": "#/$defs/timestamp"
    },
    "metadata": {
      "type": "object",
      "description": "Optional DEX metadata"
    }
  },
  "$defs": {
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[a-zA-Z0-9._-]+$"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
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
  "description": "Event recording DEX parameter update",
  "type": "object",
  "required": [
    "aex_id",
    "outcome",
    "weight",
    "lineage_factor",
    "alpha_new",
    "beta_new",
    "timestamp"
  ],
  "properties": {
    "aex_id": {
      "$ref": "#/$defs/aex_id"
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
  },
  "$defs": {
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[a-zA-Z0-9._-]+$"
    },
    "outcome": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
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
  "description": "Record of agent-to-agent interaction",
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
      "description": "Task-specific information"
    },
    "created_at": {
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
  },
  "$defs": {
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    },
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[a-zA-Z0-9._-]+$"
    },
    "outcome": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
    },
    "signature": {
      "type": "string",
      "pattern": "^[A-Za-z0-9+/]+=*$",
      "minLength": 64
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
          "type": "number",
          "minimum": 0,
          "maximum": 1,
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
      "description": "Witness reputation at time of attestation"
    }
  },
  "$defs": {
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[a-zA-Z0-9._-]+$"
    },
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    },
    "outcome": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
    },
    "signature": {
      "type": "string",
      "pattern": "^[A-Za-z0-9+/]+=*$",
      "minLength": 64
    }
  }
}
```

### Witness Selection
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/witness_selection.json",
  "title": "AEX Witness Selection",
  "description": "Record of witness selection for transaction",
  "type": "object",
  "required": [
    "transaction_id",
    "selected_witnesses",
    "selection_method",
    "timestamp"
  ],
  "properties": {
    "transaction_id": {
      "type": "string",
      "description": "Unique transaction identifier"
    },
    "selected_witnesses": {
      "type": "array",
      "items": {
        "$ref": "#/$defs/aex_id"
      },
      "minItems": 1,
      "description": "Selected witness identities"
    },
    "selection_method": {
      "type": "string",
      "enum": ["sortition", "random", "designated", "volunteer"],
      "description": "Method used for selection"
    },
    "selection_parameters": {
      "type": "object",
      "properties": {
        "min_dex": {
          "type": "number",
          "minimum": 0,
          "maximum": 1
        },
        "min_confidence": {
          "type": "number",
          "minimum": 0
        },
        "stake_required": {
          "type": "number",
          "minimum": 0
        }
      }
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    }
  },
  "$defs": {
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[a-zA-Z0-9._-]+$"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
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
  "description": "Initial handshake from agent A to agent B",
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
        }
      }
    },
    "counterparty": {
      "$ref": "#/$defs/aex_id"
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    },
    "metadata": {
      "type": "object",
      "description": "Handshake context"
    }
  },
  "$defs": {
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    },
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[a-zA-Z0-9._-]+$"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
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
  "description": "Response from agent B after handshake evaluation",
  "type": "object",
  "required": [
    "handshake_id",
    "success",
    "timestamp"
  ],
  "properties": {
    "handshake_id": {
      "$ref": "#/$defs/uuid"
    },
    "success": {
      "type": "boolean",
      "description": "Whether handshake succeeded"
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
        "fork_chain_valid": {"type": "boolean"}
      }
    },
    "counterparty_dex": {
      "type": "object",
      "properties": {
        "dex": {
          "type": "number",
          "minimum": 0,
          "maximum": 1
        },
        "n_eff": {
          "type": "number",
          "minimum": 0
        }
      }
    },
    "timestamp": {
      "$ref": "#/$defs/timestamp"
    },
    "metadata": {
      "type": "object"
    }
  },
  "$defs": {
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
    }
  }
}
```

### Handshake Record
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://aex-protocol.org/schemas/v1.0/handshake_record.json",
  "title": "AEX Handshake Record",
  "description": "Complete handshake record for ledger",
  "type": "object",
  "required": [
    "handshake_id",
    "initiator_did",
    "counterparty_did",
    "status",
    "created_at"
  ],
  "properties": {
    "handshake_id": {
      "$ref": "#/$defs/uuid"
    },
    "initiator_did": {
      "$ref": "#/$defs/aex_id"
    },
    "counterparty_did": {
      "$ref": "#/$defs/aex_id"
    },
    "status": {
      "type": "string",
      "enum": ["initiated", "identity_check", "reputation_check", "representation_check", "completed", "declined"]
    },
    "request": {
      "$ref": "handshake_request.json"
    },
    "response": {
      "$ref": "handshake_response.json"
    },
    "created_at": {
      "$ref": "#/$defs/timestamp"
    },
    "completed_at": {
      "$ref": "#/$defs/timestamp"
    },
    "metadata": {
      "type": "object"
    }
  },
  "$defs": {
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    },
    "aex_id": {
      "type": "string",
      "pattern": "^did:aex:[a-zA-Z0-9._-]+$"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
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
  "description": "Generic ledger entry for all event types",
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
  },
  "$defs": {
    "uuid": {
      "type": "string",
      "pattern": "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    },
    "timestamp": {
      "type": "string",
      "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}(\\.[0-9]+)?Z$"
    },
    "signature": {
      "type": "string",
      "pattern": "^[A-Za-z0-9+/]+=*$",
      "minLength": 64
    }
  }
}
```

---

## Validation Examples

### Python Validation Example
```python
"""
JSON Schema validation for AEX protocol objects
"""

import json
import jsonschema
from jsonschema import validate, ValidationError

# Load schema
with open('schemas/identity.json', 'r') as f:
    identity_schema = json.load(f)

# Example identity to validate
identity = {
    "aex_id": "did:aex:z6MkpTHR8JNhKnFqBVqE9ZsWRX1pNk3TxQ",
    "public_key": "MCowBQYDK2VwAyEA8x2F+3gH7kL9mN4pQ6rS1tU...",
    "created_at": "2026-02-04T20:00:00Z",
    "lineage": {
        "parent": None,
        "fork_history": []
    },
    "signature": "c2lnbmF0dXJlX2V4YW1wbGVfZGF0YQ=="
}

# Validate
try:
    validate(instance=identity, schema=identity_schema)
    print("✓ Identity is valid")
except ValidationError as e:
    print(f"✗ Validation error: {e.message}")
    print(f"  Failed at path: {' -> '.join(str(p) for p in e.path)}")
```

### JavaScript Validation Example
```javascript
/**
 * JSON Schema validation for AEX protocol objects
 */

const Ajv = require('ajv');
const ajv = new Ajv();

// Load schema
const identitySchema = require('./schemas/identity.json');
const validate = ajv.compile(identitySchema);

// Example identity to validate
const identity = {
  aex_id: "did:aex:z6MkpTHR8JNhKnFqBVqE9ZsWRX1pNk3TxQ",
  public_key: "MCowBQYDK2VwAyEA8x2F+3gH7kL9mN4pQ6rS1tU...",
  created_at: "2026-02-04T20:00:00Z",
  lineage: {
    parent: null,
    fork_history: []
  },
  signature: "c2lnbmF0dXJlX2V4YW1wbGVfZGF0YQ=="
};

// Validate
const valid = validate(identity);

if (valid) {
  console.log("✓ Identity is valid");
} else {
  console.log("✗ Validation errors:");
  validate.errors.forEach(err => {
    console.log(`  ${err.instancePath}: ${err.message}`);
  });
}
```

### Common Validation Errors

**1. Missing Required Field**
```json
{
  "error": "Missing required field",
  "field": "signature",
  "path": "/",
  "message": "required property 'signature' not found"
}
```

**2. Invalid Format**
```json
{
  "error": "Format validation failed",
  "field": "aex_id",
  "path": "/aex_id",
  "message": "does not match pattern '^did:aex:[a-zA-Z0-9._-]+$'",
  "value": "invalid-did-format"
}
```

**3. Range Violation**
```json
{
  "error": "Range validation failed",
  "field": "outcome",
  "path": "/outcome",
  "message": "must be <= 1",
  "value": 1.5
}
```

**4. Type Mismatch**
```json
{
  "error": "Type validation failed",
  "field": "alpha",
  "path": "/alpha",
  "message": "must be number",
  "value": "5.0"
}
```

---

## Schema Versioning

### Version Format
```
v{MAJOR}.{MINOR}
```

**Examples:**
- v1.0 - Initial release
- v1.1 - Backward-compatible additions
- v2.0 - Breaking changes

### Version Compatibility

**v1.x Guarantee:**
- All v1.x schemas are backward compatible
- New optional fields may be added
- Required fields will not change
- Enum values will not be removed

**v2.0 Changes:**
- May introduce breaking changes
- Migration guide provided
- Deprecation notices in v1.x

### Schema References

All schemas are published at:
```
https://aex-protocol.org/schemas/v{VERSION}/{schema}.json
```

**Example:**
```
https://aex-protocol.org/schemas/v1.0/identity.json
```

---

## Validation Best Practices

### 1. Always Validate Before Signing
```python
# ✓ GOOD
def create_identity(data):
    # Validate first
    validate(instance=data, schema=identity_schema)
    
    # Then sign
    signature = sign(data)
    data['signature'] = signature
    
    return data

# ✗ BAD
def create_identity(data):
    # Sign first without validation
    signature = sign(data)
    data['signature'] = signature
    
    # Validation might fail after signing
    validate(instance=data, schema=identity_schema)
    
    return data
```

### 2. Validate on Receipt
```python
def process_handshake(handshake_data):
    # Validate immediately
    try:
        validate(instance=handshake_data, schema=handshake_schema)
    except ValidationError as e:
        return {'error': 'Invalid handshake format', 'details': str(e)}
    
    # Process only if valid
    return handle_handshake(handshake_data)
```

### 3. Use Strict Mode
```python
# Enable strict validation
ajv = Ajv(strict=True, validate_formats=True)
```

### 4. Log Validation Failures
```python
def validate_with_logging(data, schema):
    try:
        validate(instance=data, schema=schema)
        return True
    except ValidationError as e:
        logger.error(f"Validation failed: {e.message}")
        logger.error(f"Path: {e.path}")
        logger.error(f"Schema: {schema['$id']}")
        logger.error(f"Data: {json.dumps(data, indent=2)}")
        return False
```

---

**Status:** SCHEMAS.md COMPLETE ✓

**All 13 Documents Complete:**
1. ✓ README.md
2. ✓ ARCHITECTURE.md
3. ✓ HANDSHAKE.md
4. ✓ AEX_ID.md
5. ✓ AEX_REP.md
6. ✓ AEX_DEX.md
7. ✓ AEX_SESSION.md
8. ✓ AEX_WITNESS.md
9. ✓ MATH.md
10. ✓ SECURITY.md
11. ✓ IMPLEMENTATION.md
12. ✓ EXAMPLES.md
13. ✓ SCHEMAS.md

---

## 🎉 AEX Protocol v1.0 Specification Suite Complete

The complete 13-document specification suite is now finished, comprising:

**Core Protocol** (8 docs): Architecture, handshake protocol, and all five primitives (ID, REP, DEX, SESSION, WITNESS)

**Supporting Materials** (5 docs): Mathematical foundations, security analysis, reference implementation, complete examples, and formal JSON schemas

**Total Content:** ~50,000+ words of specification, ~15,000+ lines of code, 12 complete worked examples, comprehensive threat model, formal schemas for all data structures

Should I now:
1. **Generate a complete table of contents** for all 13 documents?
2. **Create a quick-start guide** for implementers?
3. **Produce a summary document** highlighting key decisions and trade-offs?
