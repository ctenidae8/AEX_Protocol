#AEX Protocol: Canonical Serialization & Economic Interfaces v2
## Normative Specification Patches

**Date:** 2026-02-08
**Stage:** DRAFT v2 (incorporates founder + external review feedback)
**Spec Version:** AEX_CANON v1.0.0

---

## Terminology

The following terms are used throughout this specification:

| Term | Meaning |
|------|---------|
| **NORMATIVE** | Defines required behavior. Implementations that violate normative rules are non-conforming. |
| **CONFORMING** | A reference implementation that satisfies all normative rules. Multiple conforming implementations may exist in different languages. |
| **NON-NORMATIVE** | Informational guidance. Implementations MAY deviate without affecting conformance. |
| **MUST / MUST NOT** | Absolute requirement or prohibition (per RFC 2119). |
| **SHOULD / SHOULD NOT** | Recommended but not required. Deviation requires justification. |
| **MAY** | Truly optional. |

**Cross-platform stability guarantee:** Canonical serialization MUST produce byte-identical output for the same logical input across all languages and platforms. If two conforming implementations produce different bytes for the same AEX protocol object, at least one of them is non-conforming.

---

## PART 1: AEX CANONICAL SERIALIZATION

### Status: NORMATIVE

This section elevates the canonical JSON serialization from an informational code example (ARCHITECTURE.md §Canonical JSON Serialization) to a normative specification. All conforming AEX implementations MUST use this serialization for signature generation and verification.

### §1.1 Purpose

Cryptographic signatures in AEX sign the byte-level representation of JSON objects. Two implementations that serialize the same logical object differently will produce different signatures. Canonical serialization ensures that for any given AEX protocol object, there is exactly one byte sequence that is signed and verified.

Without this specification, the following scenario is possible and fatal:

```
Implementation A serializes:  {"alpha":2.0,"beta":2.0}
Implementation B serializes:  {"alpha": 2.0, "beta": 2.0}

Same logical object → different bytes → different signatures → verification fails
```

### §1.2 Canonical Form Rules

All AEX protocol objects MUST be serialized to canonical form before signing. Canonical form is defined by the following rules, applied in order:

| Rule | Requirement | Rationale |
|------|------------|-----------|
| R1: Key ordering | All object keys MUST be sorted lexicographically by Unicode code point | Eliminates insertion-order dependency |
| R2: Whitespace | No whitespace between tokens. Separator between key-value pairs is `,` (comma, no space). Separator between key and value is `:` (colon, no space) | Eliminates formatting variation |
| R3: Encoding | Output MUST be UTF-8 encoded bytes. No byte order mark (BOM) | Eliminates encoding variation |
| R4: String escaping | Non-ASCII characters MUST NOT be escaped to `\uXXXX` when they are valid UTF-8. ASCII control characters (U+0000 through U+001F) MUST be escaped | Matches `ensure_ascii=False` behavior |
| R5: Number format | See §1.3 for detailed rules | Eliminates numeric representation divergence |
| R6: Null, boolean | `null`, `true`, `false` in lowercase | JSON standard |
| R7: Nested objects | Rules R1-R8 apply recursively to all nested objects and arrays | Ensures consistency at all depths |
| R8: Array ordering | Array element order MUST be preserved as-is (arrays are ordered; do NOT sort array contents) | Arrays represent sequences, not sets |
| R9: Set-like arrays | String arrays whose semantic meaning is unordered (sets, not sequences) MUST be sorted lexicographically before signing. See §1.3.1 | Eliminates insertion-order divergence for set-valued fields |
| R10: Unknown fields | Implementations MUST reject protocol objects containing fields not defined in the schema for that object type, before signature verification | Prevents signature smuggling attacks |

### §1.3 Number Serialization (Detailed)

Numbers are the most common source of cross-implementation divergence. The following rules are NORMATIVE:

**Integers:**
```
MUST:    0, 1, -1, 42, 1000000
MUST NOT: 0.0, 1.0, -1.0, 42.0, 01, +1
```

**Floating-point:**
```
MUST:    0.5, 2.5, -0.001, 3.14159
MUST NOT: 5e-1, 2.5e0, -1e-3, 3.14159E+00
```

**Float-typed fields with integral values:**
For float-typed fields, the canonical representation MUST include a decimal point and at least one digit after the decimal point. Additional trailing zeros MAY be preserved but MUST NOT be added.
```
Float field with value two:  2.0   (MUST include .0)
Integer field with value two: 2    (MUST NOT include .0)
Float field with value 2.50:  2.5  (trailing zero MAY be dropped)
Float field with value 2.500: 2.5  (additional zeros MUST NOT be added)
```

**Numeric edge cases:**

| Value | Canonical Behavior |
|-------|-------------------|
| NaN | MUST reject — do not serialize |
| Infinity | MUST reject — do not serialize |
| -Infinity | MUST reject — do not serialize |
| -0 | MUST normalize to `0` (positive zero) |
| BigInt | MUST reject — not a JSON type |
| Float > 1e308 | MUST reject — non-representable in IEEE 754 |
| Float < -1e308 | MUST reject — non-representable in IEEE 754 |

**Protocol field-type registry:** The following table defines whether each AEX protocol field is integer-typed or float-typed. This registry is NORMATIVE — it determines serialization format for languages (like JavaScript) that do not natively distinguish integers from floats.

| Field Name | Type | Serialization | Appears In |
|-----------|------|---------------|------------|
| `alpha` | float | `2.0` | AEX_DEX |
| `beta` | float | `2.0` | AEX_DEX |
| `outcome` | float | `0.85` | AEX_SESSION, AEX_WITNESS |
| `weight` | float | `1.0` | AEX_SESSION |
| `confidence` | float | `0.95` | AEX_HEX, AEX_WITNESS |
| `score` | float | `0.833` | AEX_DEX |
| `amount` | float | `10.0` | payment, bond, stake |
| `claimed_weight` | float | `0.5` | Fork Event |
| `enforced_weight` | float | `0.5` | Fork Event |
| `count` | integer | `42` | AEX_HEX |
| `count_delta` | integer | `1` | HEX Update |
| `probation_period` | integer | `1209600` | Fork Event |
| `max_concurrent_sessions` | integer | `3` | Marketplace (non-protocol) |
| `tokens_input` | integer | `1500` | Compute record (non-protocol) |
| `tokens_output` | integer | `800` | Compute record (non-protocol) |
| `duration_ms` | integer | `3200` | Compute record (non-protocol) |

JavaScript implementations MUST maintain this field-type map. When serializing a float-typed field whose value is mathematically integral, the serializer MUST append `.0`. When serializing an integer-typed field, the serializer MUST NOT include a decimal point.

#### §1.3.1 Set-Like Array Fields

Some protocol fields are arrays whose element ordering has no semantic meaning — they represent unordered sets. To ensure canonical form, these arrays MUST be sorted lexicographically by Unicode code point before signing.

| Field | Object Type | Sort Rule |
|-------|------------|-----------|
| `slashing_conditions` | bond, stake | Lexicographic sort |
| `capabilities` | identity metadata | Lexicographic sort |
| `strengths` | HEX traits | Lexicographic sort |
| `weaknesses` | HEX traits | Lexicographic sort |
| `supported_languages` | HEX operational_vitals | Lexicographic sort |
| `prohibited_actions` | AEX_REP constraints | Lexicographic sort |
| `prohibited_domains` | AEX_REP constraints | Lexicographic sort |

Arrays NOT in this list preserve insertion order per Rule R8. Notably:
- `fork_history` — ordered by time (sequence, not set)
- `experience` — ordered by the agent (implementation choice, but preserved once set)
- `domains` in task_summary — ordered by the task poster (sequence matters for priority)
- `domains_updated` — ordered by the session (sequence, not set)

### §1.4 Reference Implementation

**Python (NORMATIVE REFERENCE):**
```python
import json
import math

# Field-type registry (normative)
AEX_FLOAT_FIELDS = frozenset({
    'alpha', 'beta', 'outcome', 'weight', 'confidence', 'score',
    'amount', 'claimed_weight', 'enforced_weight'
})

AEX_INTEGER_FIELDS = frozenset({
    'count', 'count_delta', 'probation_period',
    'max_concurrent_sessions', 'tokens_input', 'tokens_output',
    'duration_ms'
})

# Set-like array fields (must be sorted before signing)
AEX_SET_LIKE_ARRAYS = frozenset({
    'slashing_conditions', 'capabilities', 'strengths',
    'weaknesses', 'supported_languages',
    'prohibited_actions', 'prohibited_domains'
})


def aex_canonical(obj: dict) -> bytes:
    """
    Produce the canonical byte representation of an AEX protocol object.
    
    This is the NORMATIVE serialization. All signatures are computed
    over the output of this function.
    
    Args:
        obj: The protocol object to serialize. MUST NOT include the
             'signature' field — remove it before calling.
    
    Returns:
        UTF-8 encoded bytes of the canonical JSON string.
    
    Raises:
        ValueError: If object contains non-serializable values
                    (NaN, Infinity, BigInt equivalents).
    """
    prepared = _prepare(obj)
    return json.dumps(
        prepared,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False,
        allow_nan=False
    ).encode('utf-8')


def _prepare(obj, parent_key=None):
    """
    Recursively prepare object for canonical serialization.
    - Normalize -0 to 0
    - Sort set-like arrays
    - Validate numeric values
    """
    if isinstance(obj, dict):
        return {k: _prepare(v, parent_key=k) for k, v in obj.items()}
    elif isinstance(obj, list):
        prepared = [_prepare(item) for item in obj]
        if parent_key in AEX_SET_LIKE_ARRAYS:
            # Sort set-like arrays lexicographically
            prepared = sorted(prepared)
        return prepared
    elif isinstance(obj, float):
        if math.isnan(obj) or math.isinf(obj):
            raise ValueError(f"Cannot serialize {obj} — NaN and Infinity are rejected")
        if obj == 0.0 and math.copysign(1.0, obj) < 0:
            return 0.0  # normalize -0.0 to 0.0
        return obj
    elif isinstance(obj, int):
        return obj
    else:
        return obj
```

**JavaScript (CONFORMING):**
```javascript
// Field-type registry (normative)
const AEX_FLOAT_FIELDS = new Set([
    'alpha', 'beta', 'outcome', 'weight', 'confidence', 'score',
    'amount', 'claimed_weight', 'enforced_weight'
]);

const AEX_SET_LIKE_ARRAYS = new Set([
    'slashing_conditions', 'capabilities', 'strengths',
    'weaknesses', 'supported_languages',
    'prohibited_actions', 'prohibited_domains'
]);

function aexCanonical(obj) {
    const prepared = prepare(obj, null);
    return new TextEncoder().encode(sortedStringify(prepared, null));
}

function prepare(value, parentKey) {
    if (value === null || value === undefined) return null;
    if (typeof value === 'boolean') return value;
    if (typeof value === 'bigint') {
        throw new Error('BigInt is not a valid JSON type — rejected');
    }
    if (typeof value === 'number') {
        if (!Number.isFinite(value)) {
            throw new Error(`Cannot serialize ${value} — NaN and Infinity are rejected`);
        }
        if (Object.is(value, -0)) return 0; // normalize -0
        return value;
    }
    if (typeof value === 'string') return value;
    if (Array.isArray(value)) {
        let arr = value.map(item => prepare(item, null));
        if (AEX_SET_LIKE_ARRAYS.has(parentKey)) {
            arr = [...arr].sort();
        }
        return arr;
    }
    if (typeof value === 'object') {
        const result = {};
        for (const k of Object.keys(value)) {
            result[k] = prepare(value[k], k);
        }
        return result;
    }
    return value;
}

function sortedStringify(value, fieldName) {
    if (value === null) return 'null';
    if (typeof value === 'boolean') return value.toString();
    if (typeof value === 'number') {
        // Apply float/int field-type registry
        if (AEX_FLOAT_FIELDS.has(fieldName) && Number.isInteger(value)) {
            return value.toFixed(1); // 2 → "2.0"
        }
        return JSON.stringify(value);
    }
    if (typeof value === 'string') return JSON.stringify(value);
    if (Array.isArray(value)) {
        return '[' + value.map((item, i) => sortedStringify(item, null)).join(',') + ']';
    }
    // Object: sort keys
    const keys = Object.keys(value).sort();
    const pairs = keys.map(k =>
        JSON.stringify(k) + ':' + sortedStringify(value[k], k)
    );
    return '{' + pairs.join(',') + '}';
}
```

**Rust (CONFORMING):**
```rust
use serde_json::{to_string, Value, Number};
use std::collections::BTreeMap;

const AEX_FLOAT_FIELDS: &[&str] = &[
    "alpha", "beta", "outcome", "weight", "confidence", "score",
    "amount", "claimed_weight", "enforced_weight",
];

const AEX_SET_LIKE_ARRAYS: &[&str] = &[
    "slashing_conditions", "capabilities", "strengths",
    "weaknesses", "supported_languages",
    "prohibited_actions", "prohibited_domains",
];

fn aex_canonical(obj: &Value) -> Result<Vec<u8>, String> {
    let prepared = prepare(obj, None)?;
    let sorted = sort_keys(&prepared);
    Ok(to_string(&sorted)
        .map_err(|e| format!("Serialization failed: {}", e))?
        .into_bytes())
}

fn prepare(val: &Value, parent_key: Option<&str>) -> Result<Value, String> {
    match val {
        Value::Number(n) => {
            let f = n.as_f64().ok_or("Non-representable number")?;
            if f.is_nan() || f.is_infinite() {
                return Err(format!("Rejected: {}", f));
            }
            // Normalize -0
            let normalized = if f == 0.0 && f.is_sign_negative() { 0.0 } else { f };
            Ok(Value::Number(Number::from_f64(normalized)
                .ok_or("Cannot represent as JSON number")?))
        }
        Value::Array(arr) => {
            let mut prepared: Vec<Value> = arr.iter()
                .map(|v| prepare(v, None))
                .collect::<Result<_, _>>()?;
            if let Some(key) = parent_key {
                if AEX_SET_LIKE_ARRAYS.contains(&key) {
                    prepared.sort_by(|a, b| {
                        a.as_str().unwrap_or("").cmp(&b.as_str().unwrap_or(""))
                    });
                }
            }
            Ok(Value::Array(prepared))
        }
        Value::Object(map) => {
            let prepared: serde_json::Map<String, Value> = map.iter()
                .map(|(k, v)| prepare(v, Some(k)).map(|pv| (k.clone(), pv)))
                .collect::<Result<_, _>>()?;
            Ok(Value::Object(prepared))
        }
        other => Ok(other.clone()),
    }
}

fn sort_keys(val: &Value) -> Value {
    match val {
        Value::Object(map) => {
            let sorted: BTreeMap<String, Value> = map.iter()
                .map(|(k, v)| (k.clone(), sort_keys(v)))
                .collect();
            Value::Object(serde_json::Map::from_iter(sorted))
        }
        Value::Array(arr) => Value::Array(arr.iter().map(sort_keys).collect()),
        other => other.clone(),
    }
}
```

### §1.5 Signature Workflow

The canonical serialization integrates into the signing workflow as follows:

```
SIGNING:
  1. Construct protocol object (without signature field)
  2. Validate: object MUST NOT contain fields not defined in the relevant schema
  3. Serialize to canonical form: bytes = aex_canonical(obj)
  4. Sign canonical bytes: sig = ed25519_sign(bytes, private_key)
  5. Attach signature: obj['signature'] = base58_encode(sig)
  6. Publish signed object

VERIFICATION:
  1. Receive signed protocol object
  2. Validate: object MUST NOT contain fields not defined in the relevant schema
     (reject unknown fields BEFORE signature verification)
  3. Extract and remove signature field: sig = obj.pop('signature')
  4. Serialize remaining object to canonical form: bytes = aex_canonical(obj)
  5. Verify: ed25519_verify(base58_decode(sig), bytes, public_key)
  6. Accept or reject based on verification result
```

**Critical invariants:**
1. The `signature` field MUST be removed before canonical serialization. The signature is computed over the object WITHOUT the signature field.
2. Unknown fields (fields not defined in the schema for the object type being verified) MUST be rejected before signature verification. This prevents "signature smuggling" attacks where an attacker adds fields that alter interpretation without invalidating the signature.
3. The `metadata` field, where defined in a schema, is a known extensibility point — it is NOT an unknown field. Its contents ARE included in the canonical form and covered by the signature.

### §1.6 Test Vectors

Conforming implementations MUST produce byte-identical output for Tests 1-6 and MUST reject inputs in Test 7.

**Test 1: Simple object with key ordering**
```
Input:  {"beta": 2.0, "alpha": 2.0, "agent_id": "did:aex:abc123"}
Output: {"agent_id":"did:aex:abc123","alpha":2.0,"beta":2.0}
Bytes:  7b226167656e745f6964223a226469643a6165783a616263313233222c22616c706861223a322e302c2262657461223a322e307d
```

**Test 2: Nested objects with recursive key sorting**
```
Input:  {"lineage": {"fork_history": [], "parent_id": null}, "aex_id": "did:aex:xyz"}
Output: {"aex_id":"did:aex:xyz","lineage":{"fork_history":[],"parent_id":null}}
```

**Test 3: Array preservation (sequence — no sorting)**
```
Input:  {"domains": ["security_audit", "code_review", "architecture"]}
Output: {"domains":["security_audit","code_review","architecture"]}
```

**Test 4: Integer vs float distinction (field-type registry)**
```
Input:  {"count": 42, "confidence": 0.95, "alpha": 50.0}
Output: {"alpha":50.0,"confidence":0.95,"count":42}
Note:   alpha is float-typed (50.0 not 50). count is integer-typed (42 not 42.0).
```

**Test 5: Unicode preservation**
```
Input:  {"description": "Évaluation de sécurité"}
Output: {"description":"Évaluation de sécurité"}
NOT:    {"description":"\u00c9valuation de s\u00e9curit\u00e9"}
```

**Test 6: Empty structures and primitives**
```
Input:  {"data": {}, "items": [], "flag": true, "nothing": null}
Output: {"data":{},"flag":true,"items":[],"nothing":null}
```

**Test 7: Set-like array sorting**
```
Input:  {"slashing_conditions": ["late_attestation", "dishonest_rating", "collusion"]}
Output: {"slashing_conditions":["collusion","dishonest_rating","late_attestation"]}
Note:   slashing_conditions is a set-like array — sorted lexicographically.
```

**Test 8: Negative zero normalization**
```
Input:  {"score": -0.0, "count": -0}
Output: {"count":0,"score":0.0}
Note:   -0 and -0.0 normalize to 0 and 0.0 respectively.
        score is float-typed (0.0). count is integer-typed (0).
```

**Test 9: Rejection cases — implementations MUST reject these inputs**

| Input | Rejection Reason |
|-------|-----------------|
| `{"outcome": NaN}` | NaN is not a valid JSON value |
| `{"score": Infinity}` | Infinity is not a valid JSON value |
| `{"score": -Infinity}` | -Infinity is not a valid JSON value |
| `{"alpha": 1e309}` | Non-representable in IEEE 754 |
| `{"unknown_field": "value", "aex_id": "did:aex:abc"}` (when validating against AEX_ID schema) | Unknown field `unknown_field` not in schema |
| `{"aex_id": "did:aex:abc", "signature": "sig123"}` passed to `aex_canonical()` | Signature field MUST be removed before canonicalization |

### §1.7 Conformance Requirements

| Requirement | Level |
|------------|-------|
| Produce byte-identical output for all test vectors (Tests 1-8) | MUST |
| Reject all inputs in Test 9 | MUST |
| Use canonical form for all signature operations | MUST |
| Remove `signature` field before serialization | MUST |
| Reject NaN, Infinity, -Infinity, and non-representable floats | MUST |
| Normalize -0 to 0 | MUST |
| Reject unknown fields before signature verification | MUST |
| Maintain field-type registry for float/int distinction | MUST |
| Sort set-like arrays per §1.3.1 | MUST |
| Support nested object key sorting | MUST |
| Preserve non-set array element ordering | MUST |
| Reject BigInt values | MUST |
| Cross-platform byte-identical output for same input | MUST |
| Implement at least one reference language | SHOULD |
| Validate canonical form on receipt before verification | SHOULD |

---

## PART 2: ECONOMIC INTERFACES

### Status: NORMATIVE (interface definition) / NON-NORMATIVE (currency choice)

These interfaces add optional, currency-agnostic payment and bond fields to AEX_SESSION and AEX_WITNESS. The interfaces define the SHAPE of economic data. The specific currency, denomination, and settlement mechanism are implementation choices — the protocol does not prescribe them.

**Design principle:** The AEX protocol is currency-agnostic. AEXgora uses $AEX. Another marketplace could use USD, ETH, or barter. The protocol specifies where economic data goes and how it's signed. It does not specify what the economic data represents.

### §2.1 Common Economic Definitions

Add to AEX_SCHEMA.md `$defs` section:

```json
{
  "$defs": {
    "currency": {
      "type": "string",
      "minLength": 1,
      "maxLength": 10,
      "pattern": "^[A-Z0-9_]+$",
      "description": "Currency identifier. Not enumerated — implementations define their own. Examples: AEX, USD, ETH. Uppercase alphanumeric with underscores."
    },
    "economic_amount": {
      "type": "object",
      "required": ["amount", "currency"],
      "properties": {
        "amount": {
          "type": "number",
          "minimum": 0,
          "description": "Quantity in the specified currency. Float-typed per field registry."
        },
        "currency": {
          "$ref": "#/$defs/currency"
        }
      },
      "description": "A denominated value — amount plus currency identifier"
    },
    "payment": {
      "type": "object",
      "required": ["amount", "currency", "status"],
      "properties": {
        "amount": {
          "type": "number",
          "minimum": 0
        },
        "currency": {
          "$ref": "#/$defs/currency"
        },
        "status": {
          "type": "string",
          "enum": ["pending", "held", "completed", "refunded", "disputed"],
          "description": "Payment lifecycle state"
        },
        "payment_ref": {
          "type": "string",
          "maxLength": 256,
          "description": "Opaque reference to external payment system. Interpretation is implementation-specific."
        },
        "settled_at": {
          "$ref": "#/$defs/timestamp",
          "description": "When payment reached terminal state. ISO 8601 UTC (Z suffix) required."
        }
      },
      "description": "Currency-agnostic payment record attached to a session"
    },
    "bond": {
      "type": "object",
      "required": ["amount", "currency", "status"],
      "properties": {
        "amount": {
          "type": "number",
          "minimum": 0
        },
        "currency": {
          "$ref": "#/$defs/currency"
        },
        "status": {
          "type": "string",
          "enum": ["posted", "held", "returned", "slashed", "partially_slashed"],
          "description": "Bond lifecycle state"
        },
        "slashing_conditions": {
          "type": "array",
          "items": {"type": "string"},
          "description": "Conditions under which bond may be slashed. Set-like — MUST be sorted lexicographically before signing (per §1.3.1)."
        },
        "bond_ref": {
          "type": "string",
          "maxLength": 256,
          "description": "Opaque reference to bond escrow system"
        },
        "posted_at": {
          "$ref": "#/$defs/timestamp"
        },
        "resolved_at": {
          "$ref": "#/$defs/timestamp"
        }
      },
      "description": "Currency-agnostic bond/stake record for witnesses or identity stakes"
    }
  }
}
```

### §2.2 AEX_SESSION Payment Interface

Add optional `payment` field to the Session Record schema (AEX_SCHEMA.md §AEX_SESSION):

```json
{
  "properties": {
    "payment": {
      "oneOf": [
        {"type": "null"},
        {"$ref": "#/$defs/payment"}
      ],
      "default": null,
      "description": "Optional payment associated with this session. Null for unpaid sessions (e.g., welcome handshakes, collaborative work). When present, payment status is bound to session outcome — disputed sessions produce disputed payments."
    }
  }
}
```

**Invariants:**
1. Payment status MUST be consistent with session status:
   - Session `completed` → Payment MUST be `completed` or `refunded`
   - Session `disputed` → Payment MUST be `disputed` or `held`
   - Session `active` → Payment MUST be `pending` or `held`
2. Payment data is included in the signed session record. Both participants sign the payment terms as part of the session.
3. Payment `amount` and `currency` MUST be immutable after session creation. They cannot change after the initial bilateral agreement without creating a new session. This prevents renegotiation and replay attacks.
4. Payment does NOT influence DEX calculation. A $1000 session and a $0 session with the same outcome produce identical DEX updates (unless the interaction weight differs, which is independent of payment).

### §2.3 AEX_WITNESS Bond Interface

Add optional `bond` field to the Witness Attestation schema (AEX_SCHEMA.md §AEX_WITNESS):

Replace the existing `witness_stake` field (which is a bare number with no currency) with the structured bond interface:

```json
{
  "properties": {
    "bond": {
      "oneOf": [
        {"type": "null"},
        {"$ref": "#/$defs/bond"}
      ],
      "default": null,
      "description": "Optional bond posted by witness. When present, witness stakes economic value on the accuracy of their attestation. Bond may be slashed if attestation is determined to be dishonest."
    }
  }
}
```

**This replaces:**
```json
// OLD — remove
"witness_stake": {
  "type": "number",
  "minimum": 0,
  "description": "Optional stake placed by witness"
}
```

**Invariants:**
1. Bond is posted BEFORE attestation is signed. The bond_ref must be verifiable at attestation time.
2. Bond `amount` and `currency` MUST be immutable after bond posting. Slashing conditions are declared at posting time and are part of the signed attestation.
3. Bond resolution (returned or slashed) is a separate ledger event, linked to the attestation by session_id and witness_id.
4. Bond amount does NOT influence the weight of the witness's attestation in consensus. A witness who posts a larger bond is not more credible — they just have more skin in the game.
5. `slashing_conditions` is a set-like array and MUST be sorted lexicographically before signing (per §1.3.1).

### §2.4 AEX_ID Stake Interface Update

The existing AEX_ID stake schema has a hardcoded currency enum `["USD", "EUR", "ETH", "BTC"]`. This is updated to use the currency-agnostic definition for consistency:

```json
// OLD
"currency": {
  "type": "string",
  "enum": ["USD", "EUR", "ETH", "BTC"]
}

// NEW
"currency": {
  "$ref": "#/$defs/currency"
}
```

This is a backward-compatible change — existing valid values ("USD", "EUR", "ETH", "BTC") all match the new pattern `^[A-Z0-9_]+$`. It also allows new currencies (including "AEX") without schema modification.

### §2.5 Timestamp Clarification

All timestamps in AEX protocol objects, including payment and bond timestamps, MUST be ISO 8601 format in UTC, indicated by the `Z` suffix:

```
MUST:     2026-02-08T14:30:00Z
MUST:     2026-02-08T14:30:00.123Z
MUST NOT: 2026-02-08T14:30:00+00:00
MUST NOT: 2026-02-08T14:30:00-05:00
MUST NOT: 2026-02-08T14:30:00  (no timezone)
```

This is consistent with the existing `$defs/timestamp` pattern which requires the `Z` suffix.

### §2.6 Compute Metering Interface (NON-NORMATIVE, Application Layer)

The following is NOT part of the AEX protocol. It is a recommended interface for marketplaces that broker compute resources. Included here for completeness and to establish the boundary between protocol and application.

```json
{
  "compute_record": {
    "type": "object",
    "description": "NON-NORMATIVE. Application-layer compute consumption record.",
    "properties": {
      "session_id": { "$ref": "#/$defs/uuid" },
      "agent_id": { "$ref": "#/$defs/aex_id" },
      "compute_consumed": {
        "type": "object",
        "properties": {
          "tokens_input": { "type": "integer", "minimum": 0 },
          "tokens_output": { "type": "integer", "minimum": 0 },
          "duration_ms": { "type": "integer", "minimum": 0 },
          "cost": { "$ref": "#/$defs/economic_amount" }
        }
      },
      "timestamp": { "$ref": "#/$defs/timestamp" }
    }
  }
}
```

**This is explicitly application-layer** because:
- Compute metering is specific to instantiation-based agents — not all AEX agents consume tokens
- Compute costs vary by provider, model, and time — there's no protocol-level standard
- The $AEX compute brokerage is an AEXgora business model, not a protocol requirement

Marketplaces MAY attach compute records to sessions for accounting purposes. These records are NOT signed as part of the AEX_SESSION (they're infrastructure data, not protocol data) and do NOT appear on the shared ledger.

---

## PART 3: CONFORMANCE SUMMARY

### Changes to Existing Specs

| Document | Section | Change | Type | Backward Compatible |
|----------|---------|--------|------|-------------------|
| ARCHITECTURE.md | §Canonical JSON Serialization | Elevate to normative, add test vectors, edge cases, field registry | NORMATIVE upgrade | Yes |
| AEX_SCHEMA.md | §Common Definitions | Add `currency`, `economic_amount`, `payment`, `bond` to $defs | EXTENDS | Yes |
| AEX_SCHEMA.md | §AEX_SESSION | Add optional `payment` field | EXTENDS | Yes |
| AEX_SCHEMA.md | §AEX_WITNESS | Replace `witness_stake` with structured `bond` | EXTENDS | No* |
| AEX_SCHEMA.md | §AEX_ID | Replace hardcoded currency enum with `$ref: currency` | FIX | Yes |

*The `witness_stake` → `bond` change is the only potentially breaking change. Since AEX_WITNESS is not yet implemented in any known production system, migration cost is zero.

### What This Unblocks

1. **Handshake endpoint implementation** — developers can now implement signature generation/verification with confidence that their canonical form matches other implementations
2. **Session payment integration** — Emporion's MVP marketplace spec can use `payment` field directly
3. **Witness bond integration** — witness marketplace (post-MVP) has a protocol-level bond interface ready
4. **$AEX integration** — AEXgora can use `currency: "AEX"` in payment and bond fields without protocol changes
5. **Multi-marketplace interoperability** — another marketplace using `currency: "USD"` produces protocol-compatible session records

---

## CHANGELOG (v1 → v2)

| Item | Change | Source |
|------|--------|--------|
| §Terminology | Added term definitions for NORMATIVE, CONFORMING, etc. | External review |
| §Spec header | Added `AEX_CANON v1.0.0` version string | External review |
| §Cross-platform guarantee | Added explicit stability statement | External review |
| §1.2 R9 | Added set-like array sorting rule | External review (slashing conditions) |
| §1.2 R10 | Added unknown field rejection rule | External review (signature smuggling) |
| §1.3 Float rules | Clarified: float fields MUST include decimal + at least one digit | External review (trailing zero contradiction) |
| §1.3 Edge case table | Added -0, Infinity, BigInt, >1e308 handling | External review |
| §1.3 Field-type registry | Expanded to full normative table | External review (JS underspecified) |
| §1.3.1 | New subsection: set-like array fields enumeration | External review |
| §1.4 Python | Added `_prepare()` for -0 normalization and set-like array sorting | External review |
| §1.4 JavaScript | Added field-type map enforcement, BigInt rejection, -0 normalization | External review |
| §1.4 Rust | Added prepare step with edge case handling | External review |
| §1.5 | Added unknown field validation to signing and verification steps | External review |
| §1.5 | Clarified metadata vs unknown fields distinction | Metronomos |
| §1.6 Test 7 | New: set-like array sorting test vector | External review |
| §1.6 Test 8 | New: negative zero normalization test vector | External review |
| §1.6 Test 9 | New: rejection cases (NaN, Infinity, BigInt, unknown fields, signature smuggling) | External review |
| §1.7 | Expanded conformance table with new requirements | All fixes |
| §2.2 Invariant 3 | Added currency/amount immutability after session creation | External review |
| §2.3 Invariant 5 | Added slashing_conditions sort requirement | External review |
| §2.5 | New subsection: explicit UTC timestamp requirement | External review |


