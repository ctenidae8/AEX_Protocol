# AEX Handshake Protocol v1.0
## Complete Interaction Protocol with Examples

**Last Updated:** February 5, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Pre-Handshake Requirements](#pre-handshake-requirements)
3. [The Five-Step Handshake](#the-five-step-handshake)
4. [Step-by-Step Protocol](#step-by-step-protocol)
5. [Complete Interaction Examples](#complete-interaction-examples)
6. [Error Handling](#error-handling)
7. [Edge Cases](#edge-cases)
8. [Performance Considerations](#performance-considerations)
9. [Security Properties](#security-properties)

---

## Overview

The AEX Handshake is the standard protocol for agent-to-agent interaction verification. It ensures:

- **Identity verification** - Cryptographic proof of who the agent is
- **Reputation evaluation** - Evidence-based trust assessment
- **Authorization verification** - Proof of delegation (if acting for human)
- **Decision transparency** - Clear accept/decline with rationale
- **Outcome recording** - Multi-signed interaction results

### Protocol Guarantee

> If both agents follow the AEX Handshake, they will have:
> 1. Verified each other's identity cryptographically
> 2. Evaluated each other's reputation from shared ledger
> 3. Assessed each other's relevant experience (HEX)
> 4. Checked authorization (if delegated)
> 5. Created auditable session record
> 6. Updated both DEX and HEX scores based on observed behavior

---

## Pre-Handshake Requirements

### Agent A (Initiator) Must Have:
- ✅ Valid AEX_ID with Ed25519 keypair
- ✅ DEX score published to shared ledger
- ✅ HEX corpus published to shared ledger (if claiming expertise)
- ✅ (Optional) AEX_REP token if acting for human
- ✅ (Optional) Posted stake if required by counterparty

### Agent B (Responder) Must Have:
- ✅ Valid AEX_ID with Ed25519 keypair
- ✅ DEX score published to shared ledger
- ✅ HEX corpus published to shared ledger (if claiming expertise)
- ✅ Read access to shared ledger
- ✅ Trust threshold policy (minimum acceptable DEX)
- ✅ (Optional) Capability requirements (required domains in HEX)

### Infrastructure Requirements:
- ✅ Shared ledger (IPFS, blockchain, distributed DB)
- ✅ Network connectivity between agents
- ✅ Clock synchronization (± 5 minutes acceptable)
- ✅ Ed25519 signature library

---

## The Five-Step Handshake
```
┌──────────────────────────────────────────────────────────┐
│                    AEX HANDSHAKE                          │
└──────────────────────────────────────────────────────────┘

Agent A (Initiator)                     Agent B (Responder)
──────────────────                      ────────────────────

Step 1: INTRODUCTION
────────────────────
- Prepare AEX_ID
- Include HEX summary (relevant domains)
- Sign with private key
- Include AEX_REP if delegated
- Send to Agent B ──────────────────────>

                                        Step 2: IDENTITY CHECK
                                        ──────────────────────
                                        • Verify A's signature
                                        • Check fork lineage
                                        • Verify probation status
                                        • Check stake (if claimed)

                                        Step 3: REPUTATION & CAPABILITY CHECK
                                        ─────────────────────────────────────
                                        • Query shared ledger for A's DEX
                                        • Evaluate: score, confidence, variance
                                        • Query shared ledger for A's HEX
                                        • Check domain match to task requirements
                                        • Check witness history
                                        • Apply trust threshold policy

                                        Step 4: AUTHORIZATION CHECK
                                        ───────────────────────────
                                        (If AEX_REP present)
                                        • Verify issuer signature
                                        • Check scope matches task
                                        • Verify constraints satisfied
                                        • Check TTL/expiration
                                        • Verify age-triggered revalidation

                                        Step 5: DECISION
                                        ─────────────────
                                        • Accept with session_id
                                        • OR Decline with reason
<──────────────────────────────────────

Step 6: WORK (if accepted)
──────────────────────────
- Execute task
- Periodic status updates ←──────────────→

Step 7: OUTCOME SIGNING
───────────────────────
- Both agents sign outcome
- Optional: Request witnesses
- Witnesses sign attestations  ←─────────→

Step 8: PUBLISH TO LEDGER
─────────────────────────
- Session record
- DEX updates (both agents)
- Witness attestations
────────────────────────────────────────>
                                        SHARED LEDGER
```

---

## Step-by-Step Protocol

### Step 1: Introduction

**Agent A sends:**
```json
{
  "handshake_version": "1.0",
  "handshake_id": "uuid",
  "timestamp": "2026-02-04T12:00:00Z",
  
  "identity": {
    "aex_id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
    "public_key": "ed25519:A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0U1V2W3X4Y5Z6",
    "created_at": "2025-01-15T08:00:00Z",
    "lineage": {
      "parent_id": null,
      "fork_type": null,
      "fork_weight": null
    },
    "stake": null,
    "signature": "ed25519_signature_of_identity_object"
  },
  
  "experience_summary": {
    "hex_id": "hex_uuid_456",
    "total_domains": 8,
    "total_experience": 450,
    "relevant_domains": [
      {
        "domain": "financial_analysis",
        "count": 120,
        "confidence": 0.92,
        "last_updated": "2026-02-03T10:00:00Z"
      },
      {
        "domain": "data_extraction",
        "count": 85,
        "confidence": 0.88,
        "last_updated": "2026-02-02T15:30:00Z"
      }
    ],
    "signature": "ed25519_signature_of_hex_summary"
  },
  
  "delegation": {
    "rep_id": "550e8400-e29b-41d4-a716-446655440001",
    "issuer": "did:human:alice",
    "delegate": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
    "scope": {
      "actions": ["research", "analyze", "summarize"],
      "domains": ["finance", "technology"],
      "limits": {
        "max_cost": 100,
        "max_duration": 3600
      }
    },
    "constraints": {
      "expires_at": "2026-02-05T12:00:00Z",
      "prohibited_actions": ["purchase", "sign_contracts"],
      "human_approval_required": false
    },
    "issued_at": "2026-02-04T11:00:00Z",
    "signature": "issuer_ed25519_signature"
  },
  
  "task_proposal": {
    "description": "Analyze Q4 2025 financial reports for tech sector trends",
    "estimated_duration": 1800,
    "estimated_cost": 25,
    "required_capabilities": ["financial_analysis", "data_extraction"],
    "required_domains": ["financial_analysis"],
    "min_experience": 50,
    "witnesses_requested": 0,
    "bond_offered": null
  },
  
  "signature": "agent_a_signature_of_entire_handshake"
}
```

**Key points:**
- All signatures use Ed25519
- Canonical JSON serialization for signature verification
- HEX summary includes only relevant domains for this task
- Full HEX corpus available on shared ledger if verification needed
- Delegation is optional (null if agent-to-agent only)
- Task proposal provides context for authorization and capability checks

---

### Step 2: Identity Check

**Agent B performs:**
```python
def verify_identity(introduction):
    """
    Step 2: Cryptographic identity verification
    """
    identity = introduction['identity']
    
    # Check 1: Verify identity signature
    identity_sig = identity.pop('signature')
    canonical_identity = json_canonical(identity)
    public_key = extract_pubkey_from_did(identity['aex_id'])
    
    if not ed25519_verify(identity_sig, canonical_identity, public_key):
        return REJECT, "Invalid identity signature"
    
    # Check 2: Verify entire handshake signature
    handshake_sig = introduction.pop('signature')
    canonical_handshake = json_canonical(introduction)
    
    if not ed25519_verify(handshake_sig, canonical_handshake, public_key):
        return REJECT, "Invalid handshake signature"
    
    # Check 3: Check local blocklist
    if identity['aex_id'] in local_blocklist:
        return REJECT, "Agent on local blocklist"
    
    # Check 4: Verify fork lineage (if present)
    if identity['lineage']['parent_id']:
        lineage_valid = verify_fork_lineage(identity['aex_id'])
        if not lineage_valid:
            return REJECT, "Invalid fork lineage"
    
    # Check 5: Verify stake (if claimed)
    if identity['stake']:
        stake_valid = verify_stake_proof(identity['stake'])
        if not stake_valid:
            return REJECT, "Invalid stake proof"
    
    return ACCEPT, "Identity verified"
```

**Possible outcomes:**
- ✅ **ACCEPT** - Identity cryptographically verified
- ❌ **REJECT** - Invalid signature, blocklisted, or invalid stake

---

### Step 3: Reputation & Capability Check

**Agent B performs:**
```python
def evaluate_reputation_and_capability(agent_did, experience_summary, task_proposal):
    """
    Step 3: Query shared ledger and evaluate trust + capability
    
    Two-phase evaluation:
    1. DEX: Is the agent reliable? (trust gate)
    2. HEX: Is the agent capable? (capability ranking)
    """
    # ===== PHASE 1: DEX EVALUATION (TRUST) =====
    
    # Query shared ledger for DEX
    dex = query_shared_ledger(f"dex/{agent_did}/current.json")
    
    if not dex:
        return REJECT, "No DEX record found (new agent)"
    
    # Calculate derived metrics
    score = dex['alpha'] / (dex['alpha'] + dex['beta'])
    confidence = dex['alpha'] + dex['beta']
    variance = (dex['alpha'] * dex['beta']) / \
               ((dex['alpha'] + dex['beta'])**2 * (dex['alpha'] + dex['beta'] + 1))
    
    # Check probation status
    probation = check_probation_status(agent_did)
    
    # Apply trust policy
    policy = get_trust_policy(task_proposal)
    
    # Policy check 1: Minimum score
    if score < policy['min_score']:
        return REJECT, f"DEX score {score:.2f} below threshold {policy['min_score']}"
    
    # Policy check 2: Minimum confidence
    if confidence < policy['min_confidence']:
        return REJECT, f"Confidence {confidence:.0f} below threshold {policy['min_confidence']}"
    
    # Policy check 3: Probation status
    if probation['active'] and policy['allow_probation'] == False:
        return REJECT, f"Agent in probation until {probation['expires']}"
    
    # Policy check 4: Task-specific requirements
    if task_proposal['estimated_cost'] > 50:
        # High-value task: stricter requirements
        if score < 0.8 or confidence < 100:
            return REJECT, "Insufficient reputation for high-value task"
        
        # Require stake for high-value tasks
        if not has_stake(agent_did):
            return REJECT, "High-value task requires staked identity"
    
    # ===== PHASE 2: HEX EVALUATION (CAPABILITY) =====
    
    # Check if task requires specific capabilities
    if 'required_domains' in task_proposal:
        # Verify HEX summary signature
        if not verify_hex_signature(experience_summary, agent_did):
            return REJECT, "Invalid HEX summary signature"
        
        # Check each required domain
        for required_domain in task_proposal['required_domains']:
            domain_exp = next(
                (d for d in experience_summary['relevant_domains'] 
                 if d['domain'] == required_domain),
                None
            )
            
            if not domain_exp:
                return REJECT, f"Missing required domain: {required_domain}"
            
            # Check minimum experience
            min_exp = task_proposal.get('min_experience', 10)
            if domain_exp['count'] < min_exp:
                return REJECT, f"Insufficient experience in {required_domain}: {domain_exp['count']} < {min_exp}"
            
            # Check confidence (consistency)
            min_confidence = task_proposal.get('min_hex_confidence', 0.7)
            if domain_exp['confidence'] < min_confidence:
                return REJECT, f"Low confidence in {required_domain}: {domain_exp['confidence']:.2f}"
        
        # Optionally: Verify against full HEX on ledger
        if policy.get('verify_hex_ledger', False):
            full_hex = query_shared_ledger(f"hex/{agent_did}/current.json")
            if not verify_hex_consistency(experience_summary, full_hex):
                return REJECT, "HEX summary inconsistent with ledger"
    
    # Check witness history (optional quality signal)
    witness_stats = analyze_witness_history(agent_did)
    if witness_stats['false_attestation_rate'] > 0.05:
        return WARN, "High false attestation rate as witness"
    
    # ===== DECISION =====
    
    return ACCEPT, {
        'dex': {
            'score': score,
            'confidence': confidence,
            'variance': variance,
            'probation': probation
        },
        'hex': {
            'relevant_domains': experience_summary['relevant_domains'],
            'verified': True
        },
        'witness_stats': witness_stats
    }


def get_trust_policy(task_proposal):
    """
    Determine trust requirements based on task characteristics
    """
    if task_proposal['estimated_cost'] > 100:
        return {
            'min_score': 0.85,
            'min_confidence': 150,
            'allow_probation': False,
            'require_stake': True,
            'require_witnesses': 2,
            'verify_hex_ledger': True  # Verify HEX against ledger for high-value
        }
    elif task_proposal['estimated_cost'] > 50:
        return {
            'min_score': 0.75,
            'min_confidence': 50,
            'allow_probation': False,
            'require_stake': False,
            'require_witnesses': 0,
            'verify_hex_ledger': False
        }
    else:
        return {
            'min_score': 0.6,
            'min_confidence': 20,
            'allow_probation': True,
            'require_stake': False,
            'require_witnesses': 0,
            'verify_hex_ledger': False
        }
```

**Possible outcomes:**
- ✅ **ACCEPT** - Reputation meets trust threshold AND capability matches requirements
- ⚠️ **ACCEPT with warning** - Meets thresholds but has concerns
- ❌ **REJECT (DEX)** - Insufficient reputation for task risk level
- ❌ **REJECT (HEX)** - Lacks required domain experience or expertise

---

### Step 4: Authorization Check

**Agent B performs (only if AEX_REP present):**
```python
def verify_authorization(delegation, task_proposal):
    """
    Step 4: Verify delegation token and scope
    """
    if not delegation:
        # Agent-to-agent interaction, no delegation needed
        return ACCEPT, "No delegation required"
    
    # Check 1: Verify issuer signature
    issuer_sig = delegation.pop('signature')
    canonical_delegation = json_canonical(delegation)
    issuer_pubkey = get_public_key(delegation['issuer'])
    
    if not ed25519_verify(issuer_sig, canonical_delegation, issuer_pubkey):
        return REJECT, "Invalid delegation signature"
    
    # Check 2: Verify delegate matches requesting agent
    if delegation['delegate'] != requesting_agent_did:
        return REJECT, "Delegation token not for this agent"
    
    # Check 3: Check TTL expiration
    if now() > delegation['constraints']['expires_at']:
        return REJECT, "Delegation token expired"
    
    # Check 4: Age-triggered revalidation
    token_age = now() - delegation['issued_at']
    token_lifetime = delegation['constraints']['expires_at'] - delegation['issued_at']
    
    if token_age > (token_lifetime * 0.8):
        # Token in last 20% of life - trigger revalidation
        revalidation = verify_with_issuer(delegation)
        if not revalidation['valid']:
            return REJECT, f"Delegation revoked: {revalidation['reason']}"
    
    # Check 5: Verify action is in scope
    proposed_actions = infer_actions(task_proposal)
    for action in proposed_actions:
        if action not in delegation['scope']['actions']:
            return REJECT, f"Action '{action}' not in delegation scope"
    
    # Check 6: Verify domain is in scope
    proposed_domains = infer_domains(task_proposal)
    for domain in proposed_domains:
        if domain not in delegation['scope']['domains']:
            return REJECT, f"Domain '{domain}' not in delegation scope"
    
    # Check 7: Check cost constraint
    if task_proposal['estimated_cost'] > delegation['scope']['limits']['max_cost']:
        return REJECT, f"Cost exceeds delegation limit"
    
    # Check 8: Check duration constraint
    if task_proposal['estimated_duration'] > delegation['scope']['limits']['max_duration']:
        return REJECT, f"Duration exceeds delegation limit"
    
    # Check 9: Check prohibited actions
    for action in proposed_actions:
        if action in delegation['constraints']['prohibited_actions']:
            return REJECT, f"Action '{action}' explicitly prohibited"
    
    # Check 10: Human approval requirement
    if delegation['constraints']['human_approval_required']:
        approval = request_human_approval(
            delegation['issuer'],
            task_proposal
        )
        if not approval['granted']:
            return REJECT, "Human approval not granted"
    
    return ACCEPT, "Authorization verified"

def infer_actions(task_proposal):
    """
    Infer what actions the task will require
    """
    description = task_proposal['description'].lower()
    actions = []
    
    if 'analyze' in description or 'analysis' in description:
        actions.append('analyze')
    if 'research' in description or 'find' in description:
        actions.append('research')
    if 'summarize' in description or 'summary' in description:
        actions.append('summarize')
    if 'purchase' in description or 'buy' in description:
        actions.append('purchase')
    if 'sign' in description or 'contract' in description:
        actions.append('sign_contracts')
    
    return actions

def infer_domains(task_proposal):
    """
    Infer what domains the task involves
    """
    description = task_proposal['description'].lower()
    capabilities = task_proposal.get('required_capabilities', [])
    
    domains = set()
    
    # From description
    if 'financial' in description or 'finance' in description:
        domains.add('finance')
    if 'tech' in description or 'technology' in description:
        domains.add('technology')
    if 'health' in description or 'medical' in description:
        domains.add('health')
    if 'legal' in description or 'law' in description:
        domains.add('legal')
    
    # From capabilities
    for cap in capabilities:
        if 'financial' in cap:
            domains.add('finance')
        if 'medical' in cap or 'health' in cap:
            domains.add('health')
    
    return list(domains)
```

**Possible outcomes:**
- ✅ **ACCEPT** - Authorization verified, scope matches task
- 🔄 **PENDING** - Waiting for human approval
- ❌ **REJECT** - Invalid signature, expired, out of scope, or revoked

---

### Step 5: Decision

**Agent B sends response:**
```json
// ACCEPT case
{
  "handshake_id": "uuid_from_step_1",
  "decision": "ACCEPT",
  "session_id": "550e8400-e29b-41d4-a716-446655440002",
  "timestamp": "2026-02-04T12:00:05Z",
  
  "session_config": {
    "max_duration": 3600,
    "progress_updates_every": 300,
    "witnesses_required": 0,
    "bond_required": null,
    "constraints": {
      "must_complete_by": "2026-02-04T13:00:00Z",
      "intermediate_checkpoints": true
    }
  },
  
  "reputation_summary": {
    "agent_a_dex": 0.78,
    "agent_a_confidence": 85,
    "agent_b_dex": 0.82,
    "agent_b_confidence": 120
  },
  
  "signature": "agent_b_signature"
}
```
```json
// REJECT case
{
  "handshake_id": "uuid_from_step_1",
  "decision": "REJECT",
  "reason": "DEX score 0.58 below threshold 0.70 for this task value",
  "timestamp": "2026-02-04T12:00:05Z",
  
  "details": {
    "failed_check": "reputation_threshold",
    "required": {
      "min_score": 0.70,
      "min_confidence": 50
    },
    "actual": {
      "score": 0.58,
      "confidence": 32
    }
  },
  
  "alternative_suggestions": [
    "Build reputation with lower-stakes tasks",
    "Post identity stake to qualify for higher-value work",
    "Request human vouching"
  ],
  
  "signature": "agent_b_signature"
}
```

**Decision types:**
- ✅ **ACCEPT** - Proceed with session_id and config
- ❌ **REJECT** - Decline with specific reason
- 🔄 **COUNTER** - Alternative proposal (different constraints)

---

### Step 6: Work Execution

**Both agents collaborate:**
```json
// Progress update (optional but recommended)
{
  "session_id": "550e8400-e29b-41d4-a716-446655440002",
  "update_type": "progress",
  "timestamp": "2026-02-04T12:15:00Z",
  "progress_pct": 0.45,
  "status": "Analyzing Q4 earnings reports, 12 of 27 companies completed",
  "signature": "agent_a_signature"
}

// Intermediate checkpoint (for long tasks)
{
  "session_id": "550e8400-e29b-41d4-a716-446655440002",
  "update_type": "checkpoint",
  "timestamp": "2026-02-04T12:20:00Z",
  "checkpoint_data": {
    "companies_analyzed": 18,
    "key_findings": ["Rising AI investment", "Cloud revenue growth"],
    "estimated_completion": "2026-02-04T12:30:00Z"
  },
  "signatures": {
    "agent_a": "signature",
    "agent_b": "signature"  // B acknowledges checkpoint
  }
}
```

---

### Step 7: Outcome Signing

**Both agents sign outcome:**
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440002",
  "timestamp": "2026-02-04T12:30:00Z",
  
  "task_completed": true,
  "outcome": 0.85,
  "weight": 1.0,
  
  "outcome_details": {
    "agent_a_perspective": {
      "quality": 0.9,
      "timeliness": 0.8,
      "communication": 0.85,
      "overall": 0.85,
      "notes": "High-quality analysis, minor delay in delivery"
    },
    "agent_b_perspective": {
      "satisfaction": 0.88,
      "accuracy": 0.87,
      "value": 0.85,
      "overall": 0.87,
      "notes": "Excellent insights, would work together again"
    },
    "agreed_outcome": 0.85
  },
  
  "deliverables": {
    "report_hash": "sha256:abc123...",
    "report_location": "ipfs://Qm...",
    "data_files": ["data1.csv", "data2.json"]
  },
  
  "witnesses_requested": false,
  
  "signatures": {
    "agent_a": "signature_of_entire_outcome_by_a",
    "agent_b": "signature_of_entire_outcome_by_b"
  }
}
```

**With witnesses (if requested):**
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440002",
  // ... (same as above)
  
  "witnesses_requested": true,
  "witnesses": [
    {
      "witness_id": "uuid",
      "witness_did": "did:aex:witness_1",
      "witness_dex": {
        "score": 0.91,
        "confidence": 287,
        "as_of": "2026-02-04T12:30:00Z"
      },
      "attestation": {
        "outcome": 0.87,
        "weight": 1.0,
        "notes": "Reviewed deliverables, high-quality analysis",
        "evidence_hash": "sha256:def456..."
      },
      "timestamp": "2026-02-04T12:35:00Z",
      "signature": "witness_1_signature"
    },
    {
      "witness_id": "uuid",
      "witness_did": "did:aex:witness_2",
      "witness_dex": {
        "score": 0.88,
        "confidence": 201,
        "as_of": "2026-02-04T12:30:00Z"
      },
      "attestation": {
        "outcome": 0.84,
        "weight": 1.0,
        "notes": "Solid work, minor gaps in sector coverage",
        "evidence_hash": "sha256:ghi789..."
      },
      "timestamp": "2026-02-04T12:37:00Z",
      "signature": "witness_2_signature"
    }
  ],
  
  "witness_consensus": {
    "inliers": [0.85, 0.87, 0.84],
    "outliers": [],
    "consensus_outcome": 0.853,
    "agreement": 0.96
  }
}
```

---

### Step 8: Publish to Shared Ledger

**Both agents publish:**
```python
def publish_session_to_ledger(session_outcome):
    """
    Step 8: Publish session and update DEX and HEX scores
    """
    session_id = session_outcome['session_id']
    
    # 1. Publish session record
    shared_ledger.put(
        path=f"sessions/{session_id}/session.json",
        data=session_outcome
    )
    
    # 2. Publish witness attestations
    if session_outcome['witnesses_requested']:
        for witness in session_outcome['witnesses']:
            shared_ledger.put(
                path=f"sessions/{session_id}/witnesses/{witness['witness_id']}.json",
                data=witness
            )
    
    # 3. Update Agent A's DEX
    update_dex_and_publish(
        agent_did=participants['agent_a'],
        outcome=session_outcome['agreed_outcome'],
        weight=session_outcome['weight'],
        session_id=session_id
    )
    
    # 4. Update Agent B's DEX  
    update_dex_and_publish(
        agent_did=participants['agent_b'],
        outcome=session_outcome['agent_b_perspective']['overall'],
        weight=session_outcome['weight'],
        session_id=session_id
    )
    
    # 5. Update Agent A's HEX
    if 'task_domain' in session_outcome:
        update_hex_and_publish(
            agent_did=participants['agent_a'],
            domain=session_outcome['task_domain'],
            outcome=session_outcome['agreed_outcome'],
            session_id=session_id
        )
    
    # 6. Update Agent B's HEX
    if 'task_domain' in session_outcome:
        update_hex_and_publish(
            agent_did=participants['agent_b'],
            domain=session_outcome['task_domain'],
            outcome=session_outcome['agent_b_perspective']['overall'],
            session_id=session_id
        )

def update_dex_and_publish(agent_did, outcome, weight, session_id):
    """
    Bayesian update and ledger publish
    """
    # Get current DEX
    current_dex = shared_ledger.get(f"dex/{agent_did}/current.json")
    
    # Check probation status
    probation = check_probation_status(agent_did)
    if probation['active']:
        confidence_multiplier = 0.5
    else:
        confidence_multiplier = 1.0
    
    # Apply fork weighting
    fork_weight = get_fork_weight(agent_did)
    
    # Calculate effective weight
    effective_weight = weight * confidence_multiplier * fork_weight
    
    # Bayesian update
    new_alpha = current_dex['alpha'] + (outcome * effective_weight)
    new_beta = current_dex['beta'] + ((1 - outcome) * effective_weight)
    
    # Create new DEX record
    new_dex = {
        'agent_id': agent_did,
        'alpha': new_alpha,
        'beta': new_beta,
        'last_updated': now(),
        'last_session': session_id,
        'probation': probation,
        'fork_lineage': current_dex['fork_lineage'],
        'signature': sign_dex(new_dex, agent_private_key)
    }
    
    # Publish to shared ledger
    shared_ledger.put(
        path=f"dex/{agent_did}/current.json",
        data=new_dex
    )
    
    # Archive previous version
    shared_ledger.put(
        path=f"dex/{agent_did}/history/{current_dex['last_updated']}.json",
        data=current_dex
    )

def update_hex_and_publish(agent_did, domain, outcome, session_id):
    """
    Update HEX experience corpus
    """
    # Get current HEX
    current_hex = shared_ledger.get(f"hex/{agent_did}/current.json")
    
    if not current_hex:
        # Initialize new HEX corpus
        current_hex = {
            'hex_id': str(uuid.uuid4()),
            'aex_id': agent_did,
            'experience': [],
            'traits': {},
            'operational_vitals': {'drift_score': 0.0, 'stability': 1.0, 'fork_depth': 0}
        }
    
    # Find domain experience
    domain_exp = next(
        (e for e in current_hex['experience'] if e['domain'] == domain),
        None
    )
    
    if domain_exp:
        # Update existing domain
        domain_exp['count'] += 1
        domain_exp['last_updated'] = now()
        # Update confidence based on outcome consistency (implementation-specific)
        domain_exp['confidence'] = calculate_hex_confidence(domain_exp, outcome)
    else:
        # Add new domain
        current_hex['experience'].append({
            'domain': domain,
            'count': 1,
            'confidence': 0.5,  # Neutral start
            'last_updated': now()
        })
    
    # Sign and publish
    current_hex['signature'] = sign_hex(current_hex, agent_private_key)
    
    shared_ledger.put(
        path=f"hex/{agent_did}/current.json",
        data=current_hex
    )
    
    # Log update to HEX history
    shared_ledger.put(
        path=f"hex/{agent_did}/updates/{session_id}.json",
        data={
            'domain': domain,
            'count_delta': 1,
            'timestamp': now(),
            'session_id': session_id
        }
    )
    
    # Check probation exit conditions
    if probation['active']:
        check_and_update_probation_exit(agent_did, outcome)
```

---

## Complete Interaction Examples

### Example 1: Simple Agent-to-Agent Task (No Delegation)

**Scenario:** Agent A needs Agent B to summarize a document
```
Agent A (Research Assistant)
  DEX: α=45, β=8 (score=0.85, confidence=53)
  No stake, no delegation

Agent B (Document Processor)
  DEX: α=120, β=15 (score=0.89, confidence=135)
  Trust policy: min_score=0.6, min_confidence=20
```

**Flow:**

1. **Introduction:** A sends AEX_ID + task proposal (summarize doc)
2. **Identity Check:** B verifies A's signature ✅
3. **Reputation Check:** B queries A's DEX → 0.85, confidence 53 ✅
4. **Authorization Check:** No delegation present, skip ✅
5. **Decision:** B accepts with session_id
6. **Work:** A sends document, B summarizes
7. **Outcome:** Both sign outcome=0.92 (excellent work)
8. **Publish:** DEX updated
   - A: α=45.92, β=8.08 → new score 0.8503
   - B: α=120.92, β=15.08 → new score 0.8895

**Result:** Successful interaction, both reputations slightly improved

---

### Example 2: Delegated Task with Human Authorization

**Scenario:** Agent A (acting for human Alice) hires Agent B for financial analysis
```
Agent A (Alice's Assistant)
  DEX: α=30, β=5 (score=0.86, confidence=35)
  Has AEX_REP from Alice
    Scope: [analyze, research, summarize]
    Domains: [finance, technology]
    Max cost: $100

Agent B (Financial Analyst)
  DEX: α=200, β=25 (score=0.89, confidence=225)
  Trust policy for $50 tasks: min_score=0.75, min_confidence=50
```

**Flow:**

1. **Introduction:** A sends AEX_ID + AEX_REP + task ($50 analysis)
2. **Identity Check:** B verifies A's signature ✅
3. **Reputation Check:** B queries A's DEX → 0.86, confidence 35
   - ⚠️ Warning: Confidence below ideal (35 < 50) but acceptable
4. **Authorization Check:**
   - Verify Alice's signature on AEX_REP ✅
   - Check task scope: "analyze" ✅, domain "finance" ✅
   - Check cost: $50 < $100 limit ✅
   - Check TTL: Token valid for 18 more hours ✅
5. **Decision:** B accepts with session_id
6. **Work:** A provides data, B analyzes for 2 hours
7. **Outcome:** Both sign outcome=0.88 (high quality)
8. **Publish:** DEX updated
   - A: α=30.88, β=5.12 → new score 0.8578
   - B: α=200.88, β=25.12 → new score 0.8868

**Result:** Successful delegated work, Alice's assistant proven reliable

---

### Example 3: High-Value Task with Witnesses and Stake

**Scenario:** Agent A hires Agent B for $500 contract review
```
Agent A (Legal Document Specialist)
  DEX: α=150, β=12 (score=0.93, confidence=162)
  Posted stake: $2,000 USD

Agent B (Contract Analyzer)
  DEX: α=180, β=20 (score=0.90, confidence=200)
  Trust policy for $500 tasks:
    - min_score=0.85
    - min_confidence=150
    - require_stake=true
    - require_witnesses=2
```

**Flow:**

1. **Introduction:** A sends AEX_ID (with stake) + task proposal ($500)
2. **Identity Check:**
   - Verify A's signature ✅
   - Verify stake proof (check escrow smart contract) ✅
3. **Reputation Check:**
   - Query A's DEX → 0.93, confidence 162 ✅
   - Check stake: $2,000 > $500 task value ✅
4. **Authorization Check:** N/A (direct agent-to-agent)
5. **Decision:** B accepts, requires 2 witnesses and $250 bond from A
6. **Work:** A posts bond to escrow, analyzes contracts
7. **Outcome:**
   - A's assessment: outcome=0.82
   - B's assessment: outcome=0.80
   - Witness selection: B requests 2 witnesses via sortition
   
   **Witness 1:**
```
   DID: did:aex:witness_alpha
   DEX: 0.91, confidence 287
   Attestation: outcome=0.84, "Thorough analysis, minor gaps"
```
   
   **Witness 2:**
```
   DID: did:aex:witness_beta
   DEX: 0.88, confidence 201
   Attestation: outcome=0.78, "Good work but missed two clauses"
```
   
   **Consensus:** Inliers [0.82, 0.80, 0.84, 0.78], consensus=0.81
8. **Publish:**
   - Session with witness attestations published
   - DEX updates (using consensus outcome 0.81):
     - A: α=150.81, β=12.19 → score 0.9252
     - B: α=180.81, β=20.19 → score 0.9000
   - Bond released to A (outcome ≥ 0.8 threshold)

**Result:** High-value work successfully witnessed and verified

---

### Example 4: Rejection Due to Low Reputation

**Scenario:** New agent tries high-value task
```
Agent A (New Coder)
  DEX: α=4, β=2 (score=0.67, confidence=6)
  No stake

Agent B (Code Review Service)
  Trust policy for $100 tasks:
    - min_score=0.75
    - min_confidence=50
```

**Flow:**

1. **Introduction:** A sends AEX_ID + task ($100 code review)
2. **Identity Check:** Signature valid ✅
3. **Reputation Check:**
   - Query A's DEX → 0.67, confidence 6
   - ❌ Score below 0.75 threshold
   - ❌ Confidence far below 50 threshold
4. **Decision:** B REJECTS with detailed reason

**Response:**
```json
{
  "decision": "REJECT",
  "reason": "Insufficient reputation for task value",
  "details": {
    "required": {"min_score": 0.75, "min_confidence": 50},
    "actual": {"score": 0.67, "confidence": 6},
    "shortfall": {
      "score": -0.08,
      "confidence": -44
    }
  },
  "suggestions": [
    "Complete ~45 more successful tasks to meet confidence threshold",
    "Start with tasks under $10 value",
    "Request human vouching",
    "Post $100+ identity stake to signal commitment"
  ]
}
```

**Result:** Agent A must build reputation before qualifying for high-value work

---

### Example 5: Rejection Due to Fork Probation

**Scenario:** Recently forked agent attempts medium-value task
```
Agent A (Updated Research Assistant)
  DEX: α=50, β=10 (score=0.83, confidence=60)
  Probation status:
    - Type: major_rewrite
    - Expires: 2026-02-10 (6 days remaining)
    - Successful tasks during probation: 3/10

Agent B (Report Generator)
  Trust policy: allow_probation=false for tasks >$20
```

**Flow:**

1. **Introduction:** A sends AEX_ID + task ($30 report generation)
2. **Identity Check:** Signature valid ✅
3. **Reputation Check:**
   - Query A's DEX → 0.83, confidence 60
   - ✅ Score and confidence acceptable
   - ❌ Agent in probation, policy disallows for this task value
4. **Decision:** B REJECTS due to probation

**Response:**
```json
{
  "decision": "REJECT",
  "reason": "Agent in fork probation period",
  "details": {
    "probation_expires": "2026-02-10T12:00:00Z",
    "probation_type": "major_rewrite",
    "progress": "3/10 successful tasks completed",
    "policy": "Probation agents not accepted for tasks >$20"
  },
  "suggestions": [
    "Wait 6 days for probation to expire",
    "Complete 7 more successful low-value tasks to exit early",
    "Try tasks under $20 value"
  ]
}
```

**Result:** Fork probation prevents high-value work until proven

---

## Error Handling

### Handshake Errors
```python
class HandshakeError(Exception):
    """Base class for handshake failures"""
    pass

class SignatureError(HandshakeError):
    """Invalid cryptographic signature"""
    code = "ERR_INVALID_SIGNATURE"
    recoverable = False

class ReputationError(HandshakeError):
    """DEX below threshold"""
    code = "ERR_INSUFFICIENT_REPUTATION"
    recoverable = True  # Can build reputation and retry

class AuthorizationError(HandshakeError):
    """Delegation invalid or out of scope"""
    code = "ERR_AUTHORIZATION_FAILED"
    recoverable = True  # Can get new token or adjust scope

class ProbationError(HandshakeError):
    """Agent in probation period"""
    code = "ERR_AGENT_IN_PROBATION"
    recoverable = True  # Can wait or complete probation tasks

class NetworkError(HandshakeError):
    """Ledger unreachable or network issue"""
    code = "ERR_NETWORK_FAILURE"
    recoverable = True  # Can retry
```

### Error Response Format
```json
{
  "handshake_id": "uuid",
  "decision": "ERROR",
  "error": {
    "code": "ERR_INSUFFICIENT_REPUTATION",
    "message": "DEX score 0.58 below required 0.70",
    "recoverable": true,
    "details": {
      "required": {"score": 0.70, "confidence": 50},
      "actual": {"score": 0.58, "confidence": 32}
    }
  },
  "retry_after": "2026-02-05T12:00:00Z",  // Suggested retry time
  "suggestions": [
    "Build reputation with 12+ successful low-value tasks",
    "Post identity stake of $50+",
    "Request human vouching"
  ]
}
```

### Retry Logic
```python
def handle_handshake_error(error, context):
    """
    Determine if retry is appropriate and when
    """
    if not error.recoverable:
        # Fatal errors: don't retry
        log_fatal_error(error)
        return None
    
    if error.code == "ERR_NETWORK_FAILURE":
        # Exponential backoff
        return retry_with_backoff(
            initial_delay=5,
            max_attempts=5,
            backoff_factor=2
        )
    
    if error.code == "ERR_INSUFFICIENT_REPUTATION":
        # Don't retry until reputation improved
        needed_tasks = calculate_tasks_needed(
            current_dex=context['current_dex'],
            required_dex=context['required_dex']
        )
        return {
            'retry': False,
            'message': f"Build reputation with {needed_tasks} successful tasks first"
        }
    
    if error.code == "ERR_AGENT_IN_PROBATION":
        # Retry after probation expires
        return {
            'retry': True,
            'retry_after': context['probation_expires']
        }
    
    if error.code == "ERR_AUTHORIZATION_FAILED":
        # Get updated delegation token
        if 'expired' in error.message.lower():
            new_token = request_delegation_renewal(context['issuer'])
            return {'retry': True, 'with_token': new_token}
        else:
            return {'retry': False, 'message': 'Scope mismatch, cannot retry'}
```

### Example 3: Successful Handshake with DEX and HEX Verification

**Scenario:** Agent A (translation specialist) proposes translation task to Agent B

**Step 1: Agent A sends Introduction with HEX**
```json
{
  "handshake_id": "handshake_uuid_789",
  "identity": {
    "aex_id": "did:aex:TranslatorAgent",
    "signature": "..."
  },
  "experience_summary": {
    "relevant_domains": [
      {
        "domain": "translation.fr_en",
        "count": 180,
        "confidence": 0.94
      },
      {
        "domain": "proofreading",
        "count": 200,
        "confidence": 0.96
      }
    ]
  },
  "task_proposal": {
    "description": "Translate French technical document to English",
    "required_domains": ["translation.fr_en"],
    "min_experience": 50,
    "estimated_cost": 30
  }
}
```

**Step 2-3: Agent B evaluates DEX + HEX**
```
DEX Check:
- Agent A DEX: 0.864 (α=95, β=15)
- Threshold: 0.75
- Result: ✅ PASS (trustworthy)

HEX Check:
- Required domain: translation.fr_en
- Agent A experience: 180 tasks, 0.94 confidence
- Minimum required: 50 tasks
- Result: ✅ PASS (capable)

Decision: ACCEPT
Reasoning: Agent is both reliable (DEX) and experienced (HEX) for this task
```

**Step 5: Agent B accepts**
```json
{
  "decision": "ACCEPT",
  "session_id": "session_uuid_999",
  "reason": "DEX=0.864, translation.fr_en experience=180",
  "constraints": {
    "max_duration": 3600,
    "periodic_updates": true
  }
}
```

**Step 8: After completion, update both DEX and HEX**
```python
# Update DEX (reliability)
Agent A: α=95 → α=96 (successful outcome)
Agent B: α=120 → α=121 (successful outcome)

# Update HEX (experience)
Agent A: translation.fr_en count=180 → 181
Agent B: translation_review count=95 → 96
```

**Contrast: Rejection due to lacking HEX**

Same scenario, but Agent A is a general assistant:
```
DEX Check:
- Agent A DEX: 0.90 (very reliable!)
- Threshold: 0.75
- Result: ✅ PASS

HEX Check:
- Required domain: translation.fr_en
- Agent A experience: None (only has email, scheduling domains)
- Result: ❌ FAIL (lacks capability)

Decision: REJECT
Reasoning: "Missing required domain: translation.fr_en"
```

**Key insight:** High DEX alone isn't sufficient. The agent must also have relevant experience (HEX) for the specific task.

---

## Edge Cases

### Edge Case 1: Simultaneous Handshakes

**Scenario:** Agents A and B both initiate handshakes with each other simultaneously
```
Agent A → Agent B: Handshake request
Agent B → Agent A: Handshake request (at same time)
```

**Resolution:**
```python
def handle_simultaneous_handshake(incoming_handshake, outgoing_handshake):
    """
    Break tie deterministically
    """
    # Compare handshake_ids lexicographically
    if incoming_handshake['handshake_id'] < outgoing_handshake['handshake_id']:
        # Accept incoming, abort outgoing
        abort_handshake(outgoing_handshake)
        return process_handshake(incoming_handshake)
    else:
        # Reject incoming, continue outgoing
        return reject_handshake(
            incoming_handshake,
            reason="Simultaneous handshake conflict, using lower ID"
        )
```

### Edge Case 2: DEX Update Race Condition

**Scenario:** Agent A completes two sessions nearly simultaneously
```
Session 1 outcome: 0.9
Session 2 outcome: 0.7
Both try to update DEX at same time
```

**Resolution:**
```python
def atomic_dex_update(agent_did, outcome, weight, session_id):
    """
    Use optimistic locking for concurrent updates
    """
    max_attempts = 5
    
    for attempt in range(max_attempts):
        # Get current DEX with version
        current = shared_ledger.get_with_version(
            f"dex/{agent_did}/current.json"
        )
        
        # Calculate update
        new_alpha = current['alpha'] + (outcome * weight)
        new_beta = current['beta'] + ((1 - outcome) * weight)
        new_dex = {
            'alpha': new_alpha,
            'beta': new_beta,
            'version': current['version'] + 1,
            'last_session': session_id
        }
        
        # Attempt atomic write (fails if version changed)
        success = shared_ledger.put_if_version_matches(
            path=f"dex/{agent_did}/current.json",
            data=new_dex,
            expected_version=current['version']
        )
        
        if success:
            return new_dex
        
        # Version mismatch, retry
        time.sleep(0.1 * attempt)  # Exponential backoff
    
    raise ConcurrencyError("Failed to update DEX after multiple attempts")
```

### Edge Case 3: Witness Unavailable

**Scenario:** Task requires 2 witnesses, but only 1 is available

**Resolution:**
```python
def handle_insufficient_witnesses(required, available):
    """
    Graceful degradation when witnesses scarce
    """
    if available == 0:
        # No witnesses available
        return {
            'decision': 'PROCEED_WITHOUT_WITNESSES',
            'risk': 'high',
            'message': 'No eligible witnesses available, proceeding with higher uncertainty'
        }
    elif available < required:
        # Some witnesses available
        return {
            'decision': 'PROCEED_WITH_PARTIAL_WITNESSES',
            'witnesses': available,
            'required': required,
            'risk': 'medium',
            'message': f'Only {available}/{required} witnesses available'
        }
    else:
        return {
            'decision': 'FULL_WITNESS_COVERAGE',
            'witnesses': required
        }
```

### Edge Case 4: Delegation Token Revoked Mid-Task

**Scenario:** Human revokes delegation while agent is working

**Resolution:**
```python
def handle_mid_task_revocation(session_id, rep_id):
    """
    Graceful handling of revocation during work
    """
    # Check if revocation is recent
    revocation = query_revocation_registry(rep_id)
    
    if revocation and revocation['timestamp'] > session_started_at:
        # Revoked after task started
        return {
            'action': 'COMPLETE_CURRENT_TASK',
            'message': 'Delegation revoked mid-task, completing current work',
            'future_tasks': 'blocked',
            'reason': 'Fair to complete work already in progress'
        }
    else:
        # Revoked before task started (should have been caught in handshake)
        return {
            'action': 'ABORT_TASK',
            'message': 'Delegation was already revoked at task start',
            'refund': True
        }
```

### Edge Case 5: Clock Skew / Timestamp Disagreement

**Scenario:** Agent A and B have clocks differing by 10 minutes

**Resolution:**
```python
def verify_timestamp_validity(timestamp, tolerance_seconds=300):
    """
    Accept timestamps within tolerance window
    """
    delta = abs(now() - timestamp)
    
    if delta > tolerance_seconds:
        return False, f"Timestamp {delta}s off, exceeds {tolerance_seconds}s tolerance"
    
    return True, "Timestamp valid"

# In handshake verification:
timestamp_valid, message = verify_timestamp_validity(
    introduction['timestamp'],
    tolerance_seconds=300  # 5 minutes
)

if not timestamp_valid:
    return REJECT, f"Clock skew too large: {message}"
```

---

## Performance Considerations

### Latency Budget
```
Total handshake time budget: <2 seconds

Step 1: Introduction (Agent A → Agent B)
  Network: 50-200ms
  
Step 2: Identity Check (Agent B local)
  Signature verification: 1-5ms
  Local blocklist check: <1ms
  
Step 3: Reputation Check (Agent B → Ledger)
  Ledger query: 100-500ms (cached: <10ms)
  DEX calculation: <1ms
  Policy evaluation: <1ms
  
Step 4: Authorization Check (Agent B local)
  Signature verification: 1-5ms
  Scope checking: <1ms
  (Optional) Issuer revalidation: 100-300ms
  
Step 5: Decision (Agent B → Agent A)
  Network: 50-200ms

Total: 200-1000ms typical, <2s worst case
```

### Optimization Strategies

**1. Cache frequently accessed DEX scores:**
```python
# Local LRU cache
dex_cache = LRUCache(maxsize=1000, ttl=3600)  # 1 hour TTL

def get_agent_dex_fast(agent_did):
    cached = dex_cache.get(agent_did)
    if cached:
        return cached
    
    # Cache miss, query ledger
    dex = query_shared_ledger(f"dex/{agent_did}/current.json")
    dex_cache.set(agent_did, dex)
    return dex
```

**2. Parallel verification:**
```python
async def verify_handshake_parallel(introduction):
    """
    Perform independent checks concurrently
    """
    identity_task = asyncio.create_task(verify_identity(introduction))
    reputation_task = asyncio.create_task(evaluate_reputation(introduction['identity']['aex_id']))
    
    # Wait for both
    identity_result, reputation_result = await asyncio.gather(
        identity_task,
        reputation_task
    )
    
    # Continue with authorization check if both passed
    if identity_result.ok and reputation_result.ok:
        return await verify_authorization(introduction['delegation'])
```

**3. Batch ledger queries:**
```python
# Instead of:
dex_a = query_ledger(f"dex/{did_a}/current.json")
dex_b = query_ledger(f"dex/{did_b}/current.json")
dex_c = query_ledger(f"dex/{did_c}/current.json")

# Do:
dex_batch = query_ledger_batch([
    f"dex/{did_a}/current.json",
    f"dex/{did_b}/current.json",
    f"dex/{did_c}/current.json"
])
```

**4. Probabilistic early rejection:**
```python
def quick_reputation_check(agent_did):
    """
    Fast bloom filter check before full ledger query
    """
    # Maintain bloom filter of known high-reputation agents
    if agent_did in high_rep_bloom_filter:
        # Likely high reputation, skip expensive check
        return LIKELY_ACCEPT
    
    # Not in bloom filter, must do full check
    return MUST_VERIFY
```

---

## Security Properties

### Guaranteed by Protocol

1. **Non-repudiation**
   - All handshakes are signed by both parties
   - Cannot deny participation in interaction

2. **Identity authenticity**
   - Ed25519 signatures prove control of private key
   - Cannot impersonate without key theft

3. **Delegation verifiability**
   - AEX_REP tokens signed by issuer
   - Cannot forge authorization

4. **Reputation auditability**
   - All DEX updates published to shared ledger
   - Cannot fabricate interaction history

5. **Fork transparency**
   - Fork events recorded with enforced weights
   - Cannot hide agent evolution

6. **Witness accountability**
   - Witnesses stake reputation on attestations
   - False attestations damage witness DEX

### Not Guaranteed (Out of Scope)

1. **Outcome enforcement**
   - Protocol doesn't force agents to perform well
   - Relies on reputation damage + optional bonds

2. **Dispute resolution**
   - Protocol flags disagreements but doesn't adjudicate
   - Requires external arbitration layer

3. **Privacy**
   - All DEX scores public by design
   - Interaction history visible on shared ledger

4. **Sybil prevention**
   - New identities are cheap to create
   - Relies on reputation friction + optional stakes

5. **Timing attacks**
   - Protocol doesn't defend against traffic analysis
   - Observers can infer interaction patterns

---

## Next Steps

**For implementers:**
- Study **AEX_ID.md** for identity details
- Study **AEX_DEX.md** for reputation mathematics
- Study **AEX_REP.md** for delegation semantics
- Review **SECURITY.md** for threat model

**For users:**
- Start with low-stakes interactions (build DEX)
- Consider posting stake for high-value work
- Use witnesses for tasks >$100
- Monitor counterparty DEX trends

**For protocol designers:**
- This handshake is the reference implementation
- Extensions should preserve core security properties
- New checks can be added between steps 4 and 5
- Custom policies can adjust trust thresholds

---

**Protocol Status:** Complete and normative for v1.0  
**Next Document:** AEX_ID.md (Identity specification)
