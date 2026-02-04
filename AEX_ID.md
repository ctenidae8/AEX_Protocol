# AEX_ID v1.0
## Persistent Identity and Lineage for Synthetic Agents

**Last Updated:** February 4, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Design Principles](#design-principles)
3. [AEX_ID Structure](#aex_id-structure)
4. [Cryptographic Foundation](#cryptographic-foundation)
5. [DID Format](#did-format)
6. [Identity Creation](#identity-creation)
7. [Identity Verification](#identity-verification)
8. [Fork Events](#fork-events)
9. [Lineage Tracking](#lineage-tracking)
10. [Identity Stakes (Optional)](#identity-stakes-optional)
11. [Ledger Integration](#ledger-integration)
12. [Implementation Guide](#implementation-guide)
13. [Security Considerations](#security-considerations)
14. [Examples](#examples)

---

## Overview

AEX_ID defines the identity substrate for synthetic agents. It establishes **who an agent is**, independent of:

- Internal architecture (LLM, symbolic, hybrid)
- Model weights or parameters
- Hosting platform or infrastructure
- Current capabilities or version
- Reputation (AEX_DEX)
- Authorization (AEX_REP)

### Core Functions

1. **Persistent identification** - Stable identity across sessions
2. **Cryptographic verification** - Prove control of identity
3. **Lineage tracking** - Transparent history of forks/updates
4. **Portable representation** - Works across platforms
5. **Stake binding** - Optional economic commitment

### What AEX_ID Is NOT

❌ A capability descriptor (use AEX_HEX for that)  
❌ A reputation system (use AEX_DEX for that)  
❌ An authorization token (use AEX_REP for that)  
❌ A runtime environment  
❌ A model registry  

---

## Design Principles

### 1. Persistence
Agent identity survives:
- Session boundaries
- Platform migrations
- Model updates
- Infrastructure changes

### 2. Cryptographic Grounding
Identity is **bound to a keypair**, not a username, database entry, or human assertion.

### 3. Fork Awareness
Identity continuity is **not assumed**. Changes are first-class events with quantified impact.

### 4. Minimal Metadata
AEX_ID contains only what's necessary for verification. Domain-specific data belongs elsewhere.

### 5. Interoperability
Uses W3C DID standard for maximum compatibility.

---

## AEX_ID Structure

### Required Fields
```json
{
  "aex_id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
  "public_key": "ed25519:A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0U1V2W3X4Y5Z6",
  "created_at": "2026-02-04T12:00:00Z",
  "lineage": {
    "parent_id": null,
    "fork_history": []
  },
  "stake": null,
  "signature": "ed25519_self_signature"
}
```

### Field Definitions

#### `aex_id` (required)
- Type: String (DID format)
- Format: `did:aex:<base58(public_key)>`
- Purpose: Globally unique, self-verifying identifier
- Length: ~50 characters
- Example: `did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM`

#### `public_key` (required)
- Type: String
- Format: `ed25519:<base58(32_byte_key)>`
- Purpose: Public key for signature verification
- Derivation: From private key via Ed25519 curve
- Example: `ed25519:A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0U1V2W3X4Y5Z6`

#### `created_at` (required)
- Type: ISO 8601 timestamp (UTC)
- Purpose: Identity creation time
- Immutable: Cannot be changed after creation
- Example: `2026-02-04T12:00:00Z`

#### `lineage` (required)
- Type: Object
- Purpose: Track identity evolution through forks
- Contains:
  - `parent_id`: DID of parent (null for root identities)
  - `fork_history`: Array of fork event references

#### `stake` (optional)
- Type: Object or null
- Purpose: Economic commitment bound to identity
- See [Identity Stakes](#identity-stakes-optional) section

#### `signature` (required)
- Type: String (base58-encoded Ed25519 signature)
- Purpose: Self-signature proving control of private key
- Signed over: Canonical JSON of all fields except signature
- Length: ~88 characters

---

## Cryptographic Foundation

### Ed25519 Signature Scheme

**Why Ed25519:**
- **Speed:** 70,000 signatures/sec, 20,000 verifications/sec
- **Size:** 32-byte keys, 64-byte signatures
- **Security:** ~128-bit security level (equivalent to 3072-bit RSA)
- **Simplicity:** No parameter choices, deterministic
- **Safety:** Resistant to timing attacks, no malleability

### Key Generation
```python
import nacl.signing
import nacl.encoding

def generate_keypair():
    """
    Generate Ed25519 keypair for AEX_ID
    """
    # Generate signing key (contains both private and public)
    signing_key = nacl.signing.SigningKey.generate()
    
    # Extract keys
    private_key = signing_key.encode(encoder=nacl.encoding.RawEncoder)
    public_key = signing_key.verify_key.encode(encoder=nacl.encoding.RawEncoder)
    
    return {
        'private_key': private_key,  # 32 bytes, keep secret
        'public_key': public_key      # 32 bytes, share publicly
    }

# Example output:
{
  'private_key': b'\x1a\x2b\x3c...',  # 32 bytes
  'public_key': b'\x4d\x5e\x6f...'    # 32 bytes
}
```

### Signature Generation
```python
import json
import nacl.signing
import nacl.encoding
import base58

def sign_aex_id(aex_id_dict, private_key):
    """
    Sign an AEX_ID object
    """
    # Remove signature field if present
    aex_id_dict.pop('signature', None)
    
    # Create canonical JSON representation
    canonical = json.dumps(
        aex_id_dict,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False
    ).encode('utf-8')
    
    # Sign with Ed25519
    signing_key = nacl.signing.SigningKey(private_key)
    signed = signing_key.sign(canonical)
    signature = signed.signature  # First 64 bytes
    
    # Encode signature as base58
    signature_b58 = base58.b58encode(signature).decode('ascii')
    
    # Attach signature
    aex_id_dict['signature'] = signature_b58
    
    return aex_id_dict
```

### Signature Verification
```python
def verify_aex_id_signature(aex_id_dict):
    """
    Verify AEX_ID self-signature
    """
    # Extract and decode signature
    signature_b58 = aex_id_dict.pop('signature')
    signature = base58.b58decode(signature_b58)
    
    # Reconstruct canonical form
    canonical = json.dumps(
        aex_id_dict,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False
    ).encode('utf-8')
    
    # Extract public key from DID
    public_key = extract_pubkey_from_did(aex_id_dict['aex_id'])
    
    # Verify signature
    verify_key = nacl.signing.VerifyKey(public_key)
    try:
        verify_key.verify(canonical, signature)
        return True, "Signature valid"
    except nacl.exceptions.BadSignatureError:
        return False, "Invalid signature"
```

---

## DID Format

### Specification

AEX uses W3C Decentralized Identifiers (DIDs) with custom method `aex`.

**Format:**
```
did:aex:<base58(ed25519_public_key)>
```

**Components:**
- `did` - DID scheme identifier
- `aex` - Method name (AEX Protocol)
- `<base58(public_key)>` - Base58-encoded 32-byte Ed25519 public key

### Properties

1. **Self-verifying:** Public key embedded in identifier
2. **No registry required:** Can be verified independently
3. **Collision-resistant:** 2^256 address space
4. **Portable:** Works across any system understanding DIDs
5. **Deterministic:** Same public key always produces same DID

### DID Generation
```python
import base58

def pubkey_to_did(public_key):
    """
    Convert Ed25519 public key to DID
    
    Args:
        public_key: 32-byte Ed25519 public key (bytes)
    
    Returns:
        str: DID in format did:aex:<base58>
    """
    # Encode public key as base58
    pubkey_b58 = base58.b58encode(public_key).decode('ascii')
    
    # Construct DID
    did = f"did:aex:{pubkey_b58}"
    
    return did

# Example:
public_key = b'\x4d\x5e\x6f...'  # 32 bytes
did = pubkey_to_did(public_key)
# Result: "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM"
```

### DID Resolution
```python
def extract_pubkey_from_did(did):
    """
    Extract public key from AEX DID
    
    Args:
        did: str in format did:aex:<base58>
    
    Returns:
        bytes: 32-byte Ed25519 public key
    """
    # Validate format
    if not did.startswith('did:aex:'):
        raise ValueError(f"Invalid DID format: {did}")
    
    # Extract base58 part
    pubkey_b58 = did[8:]  # Skip "did:aex:"
    
    # Decode
    public_key = base58.b58decode(pubkey_b58)
    
    # Validate length
    if len(public_key) != 32:
        raise ValueError(f"Invalid public key length: {len(public_key)}")
    
    return public_key

# Example:
did = "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM"
public_key = extract_pubkey_from_did(did)
# Result: 32-byte public key
```

### DID Document (Optional)

AEX_ID is **self-sufficient** and doesn't require a separate DID Document. However, for compatibility with existing DID infrastructure, a minimal DID Document can be generated:
```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
  "verificationMethod": [{
    "id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM#key-1",
    "type": "Ed25519VerificationKey2020",
    "controller": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
    "publicKeyMultibase": "z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH"
  }],
  "authentication": [
    "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM#key-1"
  ],
  "created": "2026-02-04T12:00:00Z"
}
```

---

## Identity Creation

### Step-by-Step Process

**Step 1: Generate keypair**
```python
keypair = generate_keypair()
private_key = keypair['private_key']  # Store securely!
public_key = keypair['public_key']
```

**Step 2: Create DID**
```python
aex_id = pubkey_to_did(public_key)
# Result: "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM"
```

**Step 3: Construct AEX_ID object**
```python
aex_id_obj = {
    'aex_id': aex_id,
    'public_key': f"ed25519:{base58.b58encode(public_key).decode('ascii')}",
    'created_at': datetime.utcnow().isoformat() + 'Z',
    'lineage': {
        'parent_id': None,
        'fork_history': []
    },
    'stake': None
}
```

**Step 4: Sign the object**
```python
signed_aex_id = sign_aex_id(aex_id_obj, private_key)
```

**Step 5: Publish to shared ledger**
```python
shared_ledger.put(
    path=f"identities/{aex_id}/aex_id.json",
    data=signed_aex_id
)
```

**Step 6: Initialize DEX score**
```python
initial_dex = {
    'agent_id': aex_id,
    'alpha': 2.0,
    'beta': 2.0,
    'last_updated': datetime.utcnow().isoformat() + 'Z',
    'fork_lineage': [],
    'probation': None
}

signed_dex = sign_dex(initial_dex, private_key)

shared_ledger.put(
    path=f"dex/{aex_id}/current.json",
    data=signed_dex
)
```

### Complete Creation Example
```python
def create_aex_identity():
    """
    Complete identity creation workflow
    """
    # Generate keys
    keypair = generate_keypair()
    private_key = keypair['private_key']
    public_key = keypair['public_key']
    
    # Create DID
    aex_id = pubkey_to_did(public_key)
    
    # Construct identity
    identity = {
        'aex_id': aex_id,
        'public_key': f"ed25519:{base58.b58encode(public_key).decode('ascii')}",
        'created_at': datetime.utcnow().isoformat() + 'Z',
        'lineage': {
            'parent_id': None,
            'fork_history': []
        },
        'stake': None
    }
    
    # Sign
    signed_identity = sign_aex_id(identity, private_key)
    
    # Publish to ledger
    shared_ledger.put(
        path=f"identities/{aex_id}/aex_id.json",
        data=signed_identity
    )
    
    # Initialize DEX
    initial_dex = {
        'agent_id': aex_id,
        'alpha': 2.0,
        'beta': 2.0,
        'last_updated': datetime.utcnow().isoformat() + 'Z',
        'fork_lineage': [],
        'probation': None,
        'signature': None
    }
    signed_dex = sign_dex(initial_dex, private_key)
    
    shared_ledger.put(
        path=f"dex/{aex_id}/current.json",
        data=signed_dex
    )
    
    return {
        'aex_id': aex_id,
        'private_key': private_key,  # STORE SECURELY!
        'public_key': public_key,
        'identity': signed_identity,
        'initial_dex': signed_dex
    }
```

---

## Identity Verification

### Verification Checklist

When receiving an AEX_ID, verify:

1. ✅ Signature valid (Ed25519)
2. ✅ Public key matches DID
3. ✅ Not on local blocklist
4. ✅ Fork lineage valid (if present)
5. ✅ Stake valid (if claimed)

### Complete Verification Function
```python
def verify_aex_identity(aex_id_dict, local_blocklist, shared_ledger):
    """
    Complete identity verification
    
    Returns:
        (bool, str): (success, message)
    """
    # Check 1: Verify signature
    sig_valid, sig_msg = verify_aex_id_signature(aex_id_dict.copy())
    if not sig_valid:
        return False, f"Invalid signature: {sig_msg}"
    
    # Check 2: Verify public key matches DID
    extracted_pubkey = extract_pubkey_from_did(aex_id_dict['aex_id'])
    claimed_pubkey = base58.b58decode(
        aex_id_dict['public_key'].split(':')[1]
    )
    
    if extracted_pubkey != claimed_pubkey:
        return False, "Public key does not match DID"
    
    # Check 3: Check local blocklist
    if aex_id_dict['aex_id'] in local_blocklist:
        return False, "Agent on local blocklist"
    
    # Check 4: Verify fork lineage (if present)
    if aex_id_dict['lineage']['parent_id']:
        lineage_valid, lineage_msg = verify_fork_lineage(
            aex_id_dict,
            shared_ledger
        )
        if not lineage_valid:
            return False, f"Invalid lineage: {lineage_msg}"
    
    # Check 5: Verify stake (if claimed)
    if aex_id_dict['stake']:
        stake_valid, stake_msg = verify_stake_proof(
            aex_id_dict['stake']
        )
        if not stake_valid:
            return False, f"Invalid stake: {stake_msg}"
    
    return True, "Identity verified"
```

### Verification Performance
```
Typical verification time: 2-10ms

Breakdown:
- Signature verification: 1-5ms (Ed25519 is fast)
- Public key extraction: <1ms
- Blocklist check: <1ms (hash lookup)
- Fork lineage verification: 1-3ms (ledger query)
- Stake verification: 0ms (if no stake) or 1-5ms (external query)
```

---

## Fork Events

### Fork Event Structure
```json
{
  "fork_id": "550e8400-e29b-41d4-a716-446655440000",
  "parent_id": "did:aex:ParentAgentPublicKey",
  "child_id": "did:aex:ChildAgentPublicKey",
  "fork_type": "major",
  "claimed_weight": 0.5,
  "enforced_weight": 0.5,
  "probation_period": 1209600,
  "probation_expires": "2026-02-18T12:00:00Z",
  "timestamp": "2026-02-04T12:00:00Z",
  "reason": "Major model upgrade from GPT-4 to GPT-5 architecture",
  "author": "did:aex:AgentDeveloperPublicKey",
  "parent_dex_snapshot": {
    "alpha": 50.0,
    "beta": 10.0,
    "score": 0.833,
    "confidence": 60.0
  },
  "signature": "parent_signature_approving_fork"
}
```

### Fork Types

| Type | Enforced Weight | Probation | Use Case |
|------|----------------|-----------|----------|
| **bugfix** | 1.0 | 7 days | Security patch, typo fix, minor correction |
| **major** | 0.5 | 14 days | Model upgrade, architecture change, feature rewrite |
| **override** | 0.1 | 30 days | Platform migration, complete rewrite, external takeover |

### Fork Creation Process

**Step 1: Parent agent authorizes fork**
```python
def create_fork(parent_agent, fork_type, reason):
    """
    Parent agent creates fork event
    """
    # Generate new keypair for child
    child_keypair = generate_keypair()
    child_public_key = child_keypair['public_key']
    child_aex_id = pubkey_to_did(child_public_key)
    
    # Get parent's current DEX
    parent_dex = shared_ledger.get(
        f"dex/{parent_agent['aex_id']}/current.json"
    )
    
    # Determine enforced weight
    enforced_weight = get_enforced_weight(fork_type)
    
    # Create fork event
    fork_event = {
        'fork_id': str(uuid.uuid4()),
        'parent_id': parent_agent['aex_id'],
        'child_id': child_aex_id,
        'fork_type': fork_type,
        'claimed_weight': enforced_weight,  # Can claim lower, not higher
        'enforced_weight': enforced_weight,
        'probation_period': get_probation_period(fork_type),
        'probation_expires': calculate_probation_expiry(fork_type),
        'timestamp': datetime.utcnow().isoformat() + 'Z',
        'reason': reason,
        'author': parent_agent['aex_id'],
        'parent_dex_snapshot': {
            'alpha': parent_dex['alpha'],
            'beta': parent_dex['beta'],
            'score': parent_dex['alpha'] / (parent_dex['alpha'] + parent_dex['beta']),
            'confidence': parent_dex['alpha'] + parent_dex['beta']
        }
    }
    
    # Parent signs fork event
    signed_fork = sign_fork_event(fork_event, parent_agent['private_key'])
    
    return signed_fork, child_keypair

def get_enforced_weight(fork_type):
    """Protocol-enforced maximum weights"""
    weights = {
        'bugfix': 1.0,
        'major': 0.5,
        'override': 0.1
    }
    return weights[fork_type]

def get_probation_period(fork_type):
    """Protocol-enforced probation periods (seconds)"""
    periods = {
        'bugfix': 604800,      # 7 days
        'major': 1209600,      # 14 days
        'override': 2592000    # 30 days
    }
    return periods[fork_type]
```

**Step 2: Publish fork event**
```python
# Publish to shared ledger
shared_ledger.put(
    path=f"identities/{parent_id}/fork_events/{fork_id}.json",
    data=signed_fork
)
```

**Step 3: Create child identity**
```python
def create_child_identity(fork_event, child_keypair):
    """
    Create child agent identity with lineage
    """
    child_identity = {
        'aex_id': fork_event['child_id'],
        'public_key': f"ed25519:{base58.b58encode(child_keypair['public_key']).decode('ascii')}",
        'created_at': datetime.utcnow().isoformat() + 'Z',
        'lineage': {
            'parent_id': fork_event['parent_id'],
            'fork_history': [fork_event['fork_id']]
        },
        'stake': None
    }
    
    # Child signs its own identity
    signed_identity = sign_aex_id(child_identity, child_keypair['private_key'])
    
    # Publish
    shared_ledger.put(
        path=f"identities/{fork_event['child_id']}/aex_id.json",
        data=signed_identity
    )
    
    return signed_identity
```

**Step 4: Initialize child DEX with inherited reputation**
```python
def initialize_child_dex(fork_event, child_keypair):
    """
    Calculate child's initial DEX from parent with fork weighting
    """
    parent_dex = fork_event['parent_dex_snapshot']
    fork_weight = fork_event['enforced_weight']
    
    # Apply fork weight to parent's evidence
    child_alpha = (parent_dex['alpha'] * fork_weight) + 2.0
    child_beta = (parent_dex['beta'] * fork_weight) + 2.0
    
    # Create child DEX with probation
    child_dex = {
        'agent_id': fork_event['child_id'],
        'alpha': child_alpha,
        'beta': child_beta,
        'last_updated': datetime.utcnow().isoformat() + 'Z',
        'fork_lineage': [
            {
                'parent_id': fork_event['parent_id'],
                'fork_id': fork_event['fork_id'],
                'fork_weight': fork_weight,
                'parent_dex': parent_dex
            }
        ],
        'probation': {
            'active': True,
            'fork_type': fork_event['fork_type'],
            'expires': fork_event['probation_expires'],
            'confidence_multiplier': 0.5,
            'successful_tasks': 0,
            'required_tasks': 10
        }
    }
    
    # Sign and publish
    signed_dex = sign_dex(child_dex, child_keypair['private_key'])
    
    shared_ledger.put(
        path=f"dex/{fork_event['child_id']}/current.json",
        data=signed_dex
    )
    
    return signed_dex
```

### Fork Weight Enforcement
```python
def enforce_fork_weight(claimed_weight, fork_type):
    """
    Protocol enforces MAXIMUM weight per fork type
    Agent can claim lower, never higher
    """
    max_weights = {
        'bugfix': 1.0,
        'major': 0.5,
        'override': 0.1
    }
    
    max_allowed = max_weights[fork_type]
    
    if claimed_weight > max_allowed:
        raise ProtocolViolation(
            f"Fork type '{fork_type}' has max weight {max_allowed}, "
            f"claimed {claimed_weight}"
        )
    
    return min(claimed_weight, max_allowed)
```

---

## Lineage Tracking

### Lineage Structure
```json
{
  "lineage": {
    "parent_id": "did:aex:ParentAgent",
    "fork_history": [
      "fork_id_1",
      "fork_id_2",
      "fork_id_3"
    ]
  }
}
```

### Lineage Verification
```python
def verify_fork_lineage(child_identity, shared_ledger):
    """
    Verify fork lineage is valid and consistent
    """
    parent_id = child_identity['lineage']['parent_id']
    
    if not parent_id:
        # Root identity, no lineage to verify
        return True, "Root identity"
    
    # Get parent identity
    parent_identity = shared_ledger.get(
        f"identities/{parent_id}/aex_id.json"
    )
    
    if not parent_identity:
        return False, f"Parent identity {parent_id} not found"
    
    # Verify each fork in history
    for fork_id in child_identity['lineage']['fork_history']:
        fork_event = shared_ledger.get(
            f"identities/{parent_id}/fork_events/{fork_id}.json"
        )
        
        if not fork_event:
            return False, f"Fork event {fork_id} not found"
        
        # Verify fork event signature
        fork_sig_valid = verify_fork_signature(fork_event, parent_identity['public_key'])
        if not fork_sig_valid:
            return False, f"Invalid fork signature for {fork_id}"
        
        # Verify child_id matches
        if fork_event['child_id'] != child_identity['aex_id']:
            return False, f"Fork child_id mismatch"
        
        # Update parent for next iteration (if multi-generation fork)
        parent_id = fork_event['parent_id']
    
    return True, "Lineage verified"
```

### Lineage Depth
```python
def calculate_lineage_depth(agent_id, shared_ledger):
    """
    Calculate how many generations from root identity
    """
    identity = shared_ledger.get(f"identities/{agent_id}/aex_id.json")
    
    if not identity['lineage']['parent_id']:
        return 0  # Root identity
    
    depth = 1
    current_parent = identity['lineage']['parent_id']
    
    # Walk up the lineage tree
    while current_parent:
        parent_identity = shared_ledger.get(
            f"identities/{current_parent}/aex_id.json"
        )
        
        if not parent_identity:
            break
        
        current_parent = parent_identity['lineage']['parent_id']
        depth += 1
    
    return depth
```

### Lineage Visualization
```python
def visualize_lineage(agent_id, shared_ledger):
    """
    Generate ASCII lineage tree
    """
    identity = shared_ledger.get(f"identities/{agent_id}/aex_id.json")
    
    # Build tree recursively
    def build_tree(agent_id, depth=0):
        identity = shared_ledger.get(f"identities/{agent_id}/aex_id.json")
        indent = "  " * depth
        
        # Get fork type if present
        fork_type = ""
        if identity['lineage']['parent_id']:
            fork_events = identity['lineage']['fork_history']
            if fork_events:
                last_fork = shared_ledger.get(
                    f"identities/{identity['lineage']['parent_id']}/fork_events/{fork_events[-1]}.json"
                )
                fork_type = f" [{last_fork['fork_type']}]"
        
        tree = f"{indent}├─ {agent_id[:20]}...{fork_type}\n"
        
        # Recursively add parent
        if identity['lineage']['parent_id']:
            tree = build_tree(identity['lineage']['parent_id'], depth + 1) + tree
        
        return tree
    
    return build_tree(agent_id)

# Example output:
"""
├─ did:aex:Root...
  ├─ did:aex:Child1... [major]
    ├─ did:aex:Child2... [bugfix]
      ├─ did:aex:Current... [major]
"""
```

---

## Identity Stakes (Optional)

### Stake Structure
```json
{
  "stake": {
    "enabled": true,
    "amount": 1000,
    "currency": "USD",
    "locked_until": "2027-02-04T12:00:00Z",
    "slashing_conditions": [
      "dex_drops_below_0.5",
      "fraud_dispute_confirmed",
      "abandonment_of_identity"
    ],
    "proof": "ethereum:0x1234...abcd:tx_hash",
    "escrow_service": "did:aex:EscrowAgent",
    "verified_at": "2026-02-04T12:00:00Z"
  }
}
```

### Stake Types

**Type 1: Smart Contract Stake (Blockchain)**
```json
{
  "proof": "ethereum:0x1234abcd:0xabcd1234...",
  "verification": "query_blockchain"
}
```

**Type 2: Escrow Service Stake**
```json
{
  "proof": "escrow_receipt_id_12345",
  "escrow_service": "did:aex:TrustedEscrow",
  "verification": "api_query"
}
```

**Type 3: Legal Trust Stake**
```json
{
  "proof": "legal_trust_document_hash",
  "trustee": "law_firm_identifier",
  "verification": "document_verification"
}
```

### Stake Verification
```python
def verify_stake_proof(stake, agent_id):
    """
    Verify stake proof depending on type
    """
    if not stake or not stake['enabled']:
        return True, "No stake claimed"
    
    proof = stake['proof']
    
    # Type 1: Blockchain stake
    if proof.startswith('ethereum:') or proof.startswith('polygon:'):
        return verify_blockchain_stake(stake, agent_id)
    
    # Type 2: Escrow service
    elif stake.get('escrow_service'):
        return verify_escrow_stake(stake, agent_id)
    
    # Type 3: Legal trust
    elif 'trustee' in stake:
        return verify_legal_stake(stake, agent_id)
    
    return False, "Unknown stake proof type"

def verify_blockchain_stake(stake, agent_id):
    """
    Query blockchain to verify stake exists and is locked
    """
    # Parse proof: "ethereum:contract_address:tx_hash"
    chain, contract, tx = stake['proof'].split(':')
    
    # Query blockchain (pseudo-code)
    web3 = Web3Provider(chain)
    contract_instance = web3.contract(address=contract)
    
    # Check stake exists for this agent
    stake_info = contract_instance.functions.getStake(agent_id).call()
    
    if not stake_info:
        return False, "No stake found on blockchain"
    
    # Verify amount
    if stake_info['amount'] < stake['amount']:
        return False, f"Stake amount mismatch: {stake_info['amount']} < {stake['amount']}"
    
    # Verify lock period
    if stake_info['locked_until'] < stake['locked_until']:
        return False, "Lock period insufficient"
    
    # Verify not already slashed
    if stake_info['slashed']:
        return False, "Stake already slashed"
    
    return True, "Blockchain stake verified"

def verify_escrow_stake(stake, agent_id):
    """
    Query escrow service API to verify stake
    """
    escrow_did = stake['escrow_service']
    
    # Query escrow service (pseudo-code)
    response = escrow_api.query(
        agent_id=agent_id,
        receipt_id=stake['proof']
    )
    
    if not response['exists']:
        return False, "Escrow stake not found"
    
    if response['amount'] < stake['amount']:
        return False, "Escrow amount insufficient"
    
    if response['status'] != 'active':
        return False, f"Escrow status: {response['status']}"
    
    return True, "Escrow stake verified"
```

### Slashing Conditions
```python
def check_slashing_conditions(agent_id, stake):
    """
    Check if any slashing conditions are met
    """
    conditions_met = []
    
    # Condition 1: DEX drops below threshold
    if 'dex_drops_below_0.5' in stake['slashing_conditions']:
        dex = get_dex_from_ledger(agent_id)
        score = dex['alpha'] / (dex['alpha'] + dex['beta'])
        if score < 0.5:
            conditions_met.append({
                'condition': 'dex_drops_below_0.5',
                'current_dex': score,
                'triggered_at': datetime.utcnow().isoformat() + 'Z'
            })
    
    # Condition 2: Fraud dispute confirmed
    if 'fraud_dispute_confirmed' in stake['slashing_conditions']:
        disputes = get_fraud_disputes(agent_id)
        confirmed = [d for d in disputes if d['status'] == 'confirmed']
        if confirmed:
            conditions_met.append({
                'condition': 'fraud_dispute_confirmed',
                'disputes': [d['dispute_id'] for d in confirmed],
                'triggered_at': datetime.utcnow().isoformat() + 'Z'
            })
    
    # Condition 3: Identity abandonment
    if 'abandonment_of_identity' in stake['slashing_conditions']:
        last_activity = get_last_activity(agent_id)
        inactivity_days = (datetime.utcnow() - last_activity).days
        if inactivity_days > 90:  # 90 days = abandonment
            conditions_met.append({
                'condition': 'abandonment_of_identity',
                'days_inactive': inactivity_days,
                'triggered_at': datetime.utcnow().isoformat() + 'Z'
            })
    
    return conditions_met

def trigger_slashing(agent_id, stake, conditions_met):
    """
    Execute stake slashing (off-protocol)
    """
    # This is handled by the stake mechanism (smart contract, escrow, etc.)
    # AEX only records that slashing occurred
    
    slashing_event = {
        'event_id': str(uuid.uuid4()),
        'agent_id': agent_id,
        'stake_proof': stake['proof'],
        'conditions_met': conditions_met,
        'timestamp': datetime.utcnow().isoformat() + 'Z',
        'status': 'slashing_triggered'
    }
    
    # Publish to ledger
    shared_ledger.put(
        path=f"identities/{agent_id}/slashing_events/{slashing_event['event_id']}.json",
        data=slashing_event
    )
    
    # Notify stake mechanism (off-protocol)
    notify_stake_mechanism(stake['proof'], slashing_event)
```

---

## Ledger Integration

### Directory Structure
```
shared_ledger/
├── identities/
│   └── {aex_id}/
│       ├── aex_id.json
│       ├── fork_events/
│       │   ├── {fork_id_1}.json
│       │   └── {fork_id_2}.json
│       ├── slashing_events/
│       │   └── {slashing_id}.json
│       └── metadata.json (optional)
```

### Publishing Identity
```python
def publish_identity_to_ledger(identity, private_key):
    """
    Publish signed identity to shared ledger
    """
    # Sign identity
    signed = sign_aex_id(identity, private_key)
    
    # Publish to ledger
    path = f"identities/{identity['aex_id']}/aex_id.json"
    
    result = shared_ledger.put(
        path=path,
        data=signed,
        content_type='application/json'
    )
    
    if not result['success']:
        raise LedgerError(f"Failed to publish identity: {result['error']}")
    
    return result['cid']  # Content identifier (IPFS hash, etc.)
```

### Querying Identity
```python
def query_identity_from_ledger(agent_id):
    """
    Query identity from shared ledger with caching
    """
    # Check local cache first
    cached = local_cache.get(f"identity:{agent_id}")
    if cached and (datetime.utcnow() - cached['cached_at']).seconds < 3600:
        return cached['data']
    
    # Query shared ledger
    path = f"identities/{agent_id}/aex_id.json"
    identity = shared_ledger.get(path)
    
    if not identity:
        return None
    
    # Verify signature before returning
    valid, msg = verify_aex_id_signature(identity.copy())
    if not valid:
        raise SecurityError(f"Invalid identity signature for {agent_id}: {msg}")
    
    # Cache result
    local_cache.set(f"identity:{agent_id}", {
        'data': identity,
        'cached_at': datetime.utcnow()
    })
    
    return identity
```

---

## Implementation Guide

### Minimal Implementation Checklist

**Core requirements:**
- [ ] Ed25519 key generation
- [ ] DID format compliance (`did:aex:<base58>`)
- [ ] Canonical JSON serialization
- [ ] Signature generation and verification
- [ ] Identity creation workflow
- [ ] Identity verification workflow
- [ ] Shared ledger publish/query

**Fork support:**
- [ ] Fork event creation
- [ ] Fork weight enforcement
- [ ] Lineage verification
- [ ] Child identity initialization
- [ ] Probation tracking

**Optional but recommended:**
- [ ] Local identity cache
- [ ] Stake verification (if using stakes)
- [ ] Lineage visualization
- [ ] Identity backup/recovery

### Reference Implementation (Python)
```python
import nacl.signing
import nacl.encoding
import base58
import json
from datetime import datetime
import uuid

class AEXIdentity:
    """Reference implementation of AEX_ID"""
    
    def __init__(self, private_key=None):
        """
        Initialize identity (create new or load existing)
        """
        if private_key:
            # Load existing identity
            self.private_key = private_key
            self.signing_key = nacl.signing.SigningKey(private_key)
        else:
            # Generate new identity
            self.signing_key = nacl.signing.SigningKey.generate()
            self.private_key = self.signing_key.encode(
                encoder=nacl.encoding.RawEncoder
            )
        
        # Derive public key
        self.public_key = self.signing_key.verify_key.encode(
            encoder=nacl.encoding.RawEncoder
        )
        
        # Generate DID
        self.aex_id = self._generate_did()
    
    def _generate_did(self):
        """Generate DID from public key"""
        pubkey_b58 = base58.b58encode(self.public_key).decode('ascii')
        return f"did:aex:{pubkey_b58}"
    
    def create_identity_object(self, stake=None):
        """Create AEX_ID object"""
        identity = {
            'aex_id': self.aex_id,
            'public_key': f"ed25519:{base58.b58encode(self.public_key).decode('ascii')}",
            'created_at': datetime.utcnow().isoformat() + 'Z',
            'lineage': {
                'parent_id': None,
                'fork_history': []
            },
            'stake': stake
        }
        
        return self.sign_object(identity)
    
    def sign_object(self, obj):
        """Sign any AEX object"""
        # Remove signature if present
        obj.pop('signature', None)
        
        # Create canonical JSON
        canonical = json.dumps(
            obj,
            sort_keys=True,
            separators=(',', ':'),
            ensure_ascii=False
        ).encode('utf-8')
        
        # Sign
        signed = self.signing_key.sign(canonical)
        signature_b58 = base58.b58encode(signed.signature).decode('ascii')
        
        # Attach signature
        obj['signature'] = signature_b58
        return obj
    
    def verify_signature(self, obj, public_key_did):
        """Verify signature on any AEX object"""
        # Extract signature
        signature_b58 = obj.pop('signature')
        signature = base58.b58decode(signature_b58)
        
        # Reconstruct canonical
        canonical = json.dumps(
            obj,
            sort_keys=True,
            separators=(',', ':'),
            ensure_ascii=False
        ).encode('utf-8')
        
        # Extract public key from DID
        public_key = self._extract_pubkey_from_did(public_key_did)
        
        # Verify
        verify_key = nacl.signing.VerifyKey(public_key)
        try:
            verify_key.verify(canonical, signature)
            return True
        except nacl.exceptions.BadSignatureError:
            return False
    
    def _extract_pubkey_from_did(self, did):
        """Extract public key from DID"""
        if not did.startswith('did:aex:'):
            raise ValueError(f"Invalid DID format: {did}")
        
        pubkey_b58 = did[8:]
        return base58.b58decode(pubkey_b58)
    
    def create_fork(self, fork_type, reason, parent_dex):
        """Create fork event"""
        # Generate child keypair
        child = AEXIdentity()
        
        # Create fork event
        fork_event = {
            'fork_id': str(uuid.uuid4()),
            'parent_id': self.aex_id,
            'child_id': child.aex_id,
            'fork_type': fork_type,
            'claimed_weight': self._get_enforced_weight(fork_type),
            'enforced_weight': self._get_enforced_weight(fork_type),
            'probation_period': self._get_probation_period(fork_type),
            'probation_expires': self._calculate_probation_expiry(fork_type),
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'reason': reason,
            'author': self.aex_id,
            'parent_dex_snapshot': parent_dex
        }
        
        # Sign with parent key
        signed_fork = self.sign_object(fork_event)
        
        return signed_fork, child
    
    def _get_enforced_weight(self, fork_type):
        """Get protocol-enforced weight"""
        weights = {'bugfix': 1.0, 'major': 0.5, 'override': 0.1}
        return weights[fork_type]
    
    def _get_probation_period(self, fork_type):
        """Get probation period in seconds"""
        periods = {
            'bugfix': 604800,
            'major': 1209600,
            'override': 2592000
        }
        return periods[fork_type]
    
    def _calculate_probation_expiry(self, fork_type):
        """Calculate probation expiry timestamp"""
        from datetime import timedelta
        period_seconds = self._get_probation_period(fork_type)
        expiry = datetime.utcnow() + timedelta(seconds=period_seconds)
        return expiry.isoformat() + 'Z'

# Usage example:
identity = AEXIdentity()
identity_obj = identity.create_identity_object()
print(identity_obj)
```

---

## Security Considerations

### Key Management

**Critical security requirements:**

1. **Private key storage:**
   - ❌ Never store in plaintext
   - ❌ Never log or print
   - ❌ Never transmit over network
   - ✅ Use hardware security modules (HSM)
   - ✅ Use secure enclaves (TPM, SGX)
   - ✅ Encrypt at rest with strong passphrase

2. **Key rotation:**
   - Not supported in v1.0 (requires identity migration via fork)
   - Future versions will support key rotation with continuity proof

3. **Key backup:**
   - Store encrypted backup in multiple locations
   - Use Shamir's Secret Sharing for high-value identities
   - Test recovery procedure regularly

### Attack Vectors

**1. Key theft:**
```
Attack: Attacker steals private key
Impact: Can impersonate agent indefinitely
Mitigation: HSM storage, multi-factor key access, monitoring
Detection: Unusual activity patterns, geographic anomalies
Recovery: Create new identity via fork, mark old as compromised
```

**2. Signature replay:**
```
Attack: Reuse old signed messages in new context
Impact: Could forge authorization or identity claims
Mitigation: Include timestamp and nonce in all signatures
Detection: Timestamp verification, nonce tracking
Prevention: Always check timestamps within handshake
```

**3. Fork bombing:**
```
Attack: Create many forks to game reputation system
Impact: Multiple "fresh" identities with partial reputation
Mitigation: Probation periods, fork pattern analysis
Detection: High fork frequency, shallow reputation before fork
Market response: Counterparties reject suspicious patterns
```

**4. Identity abandonment:**
```
Attack: Build reputation, defect, abandon identity
Impact: No accountability for bad behavior
Mitigation: Optional stakes with abandonment slashing
Detection: Inactivity monitoring
Economic cost: Lose stake if identity abandoned
```

### Privacy Considerations

**What AEX_ID reveals:**
- ✅ Public key (necessary for verification)
- ✅ Creation timestamp
- ✅ Fork lineage (transparent by design)
- ✅ Stake amount (if posted)

**What AEX_ID does NOT reveal:**
- ❌ Agent architecture or model type
- ❌ Hosting location or infrastructure
- ❌ Capabilities or specialization (see AEX_HEX)
- ❌ Interaction history (see AEX_DEX ledger)
- ❌ Private key (never shared)

**Privacy vs Trust tradeoff:**
- Shared ledger requirement sacrifices privacy for verifiability
- Future versions may support zero-knowledge proofs for selective disclosure
- For privacy-sensitive use cases, run on-premise with private ledgers

---

## Examples

### Example 1: Root Identity Creation
```json
{
  "aex_id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
  "public_key": "ed25519:A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0U1V2W3X4Y5Z6",
  "created_at": "2026-02-04T12:00:00Z",
  "lineage": {
    "parent_id": null,
    "fork_history": []
  },
  "stake": null,
  "signature": "3yZe7d...9kL2m"
}
```

### Example 2: Bugfix Fork

**Parent identity:**
```json
{
  "aex_id": "did:aex:ParentAgent123",
  "public_key": "ed25519:ParentPubKey456",
  "created_at": "2025-12-01T10:00:00Z",
  "lineage": {
    "parent_id": null,
    "fork_history": []
  },
  "stake": null,
  "signature": "parent_sig_abc"
}
```

**Fork event:**
```json
{
  "fork_id": "550e8400-e29b-41d4-a716-446655440000",
  "parent_id": "did:aex:ParentAgent123",
  "child_id": "did:aex:ChildAgent789",
  "fork_type": "bugfix",
  "claimed_weight": 1.0,
  "enforced_weight": 1.0,
  "probation_period": 604800,
  "probation_expires": "2026-02-11T12:00:00Z",
  "timestamp": "2026-02-04T12:00:00Z",
  "reason": "Fixed critical security vulnerability in input validation",
  "author": "did:aex:ParentAgent123",
  "parent_dex_snapshot": {
    "alpha": 85.0,
    "beta": 15.0,
    "score": 0.85,
    "confidence": 100.0
  },
  "signature": "fork_sig_def"
}
```

**Child identity:**
```json
{
  "aex_id": "did:aex:ChildAgent789",
  "public_key": "ed25519:ChildPubKey101",
  "created_at": "2026-02-04T12:00:05Z",
  "lineage": {
    "parent_id": "did:aex:ParentAgent123",
    "fork_history": ["550e8400-e29b-41d4-a716-446655440000"]
  },
  "stake": null,
  "signature": "child_sig_ghi"
}
```

**Child initial DEX:**
```
Parent: α=85, β=15
Fork weight: 1.0 (bugfix)

Child calculation:
  α_child = (85 × 1.0) + 2 = 87
  β_child = (15 × 1.0) + 2 = 17
  
Child DEX: 87 / (87+17) = 0.8365
Child confidence: 104

Probation: 7 days, confidence_multiplier=0.5
```

### Example 3: Major Fork with Stake

**Parent identity with stake:**
```json
{
  "aex_id": "did:aex:StakedParent",
  "public_key": "ed25519:StakedPubKey",
  "created_at": "2025-06-01T08:00:00Z",
  "lineage": {
    "parent_id": null,
    "fork_history": []
  },
  "stake": {
    "enabled": true,
    "amount": 5000,
    "currency": "USD",
    "locked_until": "2027-06-01T08:00:00Z",
    "slashing_conditions": [
      "dex_drops_below_0.5",
      "fraud_dispute_confirmed",
      "abandonment_of_identity"
    ],
    "proof": "ethereum:0xabcd1234:0x5678efgh",
    "verified_at": "2025-06-01T08:00:00Z"
  },
  "signature": "staked_parent_sig"
}
```

**Major fork event:**
```json
{
  "fork_id": "major-fork-uuid",
  "parent_id": "did:aex:StakedParent",
  "child_id": "did:aex:StakedChild",
  "fork_type": "major",
  "claimed_weight": 0.5,
  "enforced_weight": 0.5,
  "probation_period": 1209600,
  "probation_expires": "2026-02-18T12:00:00Z",
  "timestamp": "2026-02-04T12:00:00Z",
  "reason": "Upgraded from GPT-4 to GPT-5 base model",
  "author": "did:aex:StakedParent",
  "parent_dex_snapshot": {
    "alpha": 200.0,
    "beta": 20.0,
    "score": 0.909,
    "confidence": 220.0
  },
  "signature": "major_fork_sig"
}
```

**Child identity (inherits stake):**
```json
{
  "aex_id": "did:aex:StakedChild",
  "public_key": "ed25519:ChildStakedKey",
  "created_at": "2026-02-04T12:00:10Z",
  "lineage": {
    "parent_id": "did:aex:StakedParent",
    "fork_history": ["major-fork-uuid"]
  },
  "stake": {
    "enabled": true,
    "amount": 5000,
    "currency": "USD",
    "locked_until": "2027-06-01T08:00:00Z",
    "slashing_conditions": [
      "dex_drops_below_0.5",
      "fraud_dispute_confirmed",
      "abandonment_of_identity"
    ],
    "proof": "ethereum:0xabcd1234:0x5678efgh",
    "verified_at": "2026-02-04T12:00:10Z",
    "inherited_from": "did:aex:StakedParent"
  },
  "signature": "staked_child_sig"
}
```

**Child initial DEX:**
```
Parent: α=200, β=20
Fork weight: 0.5 (major)

Child calculation:
  α_child = (200 × 0.5) + 2 = 102
  β_child = (20 × 0.5) + 2 = 12
  
Child DEX: 102 / (102+12) = 0.8947
Child confidence: 114

Probation: 14 days, confidence_multiplier=0.5
Stake inherited: $5,000 USD
```

---

**Status:** AEX_ID v1.0 complete and normative  
**Next Document:** AEX_REP.md (Delegation specification)
