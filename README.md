# AEX Protocol v1.0
## Agent Experience Protocol - Identity, Delegation, and Reputation for Synthetic Agents

**Status:** Production Candidate  
**Version:** 1.0.0  
**Last Updated:** February 4, 2026

---

## Overview

The AEX Protocol provides the foundational substrate for multi-agent coordination, enabling synthetic agents to:

- **Establish cryptographic identity** with lineage tracking
- **Delegate authority** from humans to agents and between agents
- **Build verifiable reputation** through observable behavior
- **Coordinate safely** without centralized control

AEX is not a framework, runtime, or marketplace. It is a **protocol layer** that makes those things possible.

---

## Core Primitives

AEX consists of five cryptographically-signed primitives:

| Primitive | Purpose | Answers |
|-----------|---------|---------|
| **AEX_ID** | Persistent identity & lineage | "Who are you?" |
| **AEX_REP** | Delegation & authorization | "Who are you acting for?" |
| **AEX_DEX** | Behavioral reputation | "How reliable are you?" |
| **AEX_SESSION** | Interaction binding | "What happened?" |
| **AEX_WITNESS** | Third-party attestation | "Can anyone verify this?" |

---

## Design Principles

### 1. Cryptographic Verifiability
All identity and reputation claims are Ed25519-signed. No exceptions.

### 2. Behavioral Grounding
Reputation emerges from observable outcomes, not self-reported capabilities.

### 3. Fork Awareness
Agent updates (model changes, rewrites) are first-class events with protocol-enforced trust impact and mandatory probation periods.

### 4. Interoperability
Uses W3C DID standard. Portable across platforms, ecosystems, and implementations.

### 5. Shared Ledger for Trust
DEX scores require public, verifiable ledgers. Private reputation is not trustworthy reputation.

### 6. Economic Optionality
Provides hooks for stakes and bonds without mandating any token or payment system.

---

## Quick Start

### 1. Create an Agent Identity
```json
{
  "aex_id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
  "public_key": "ed25519:A1B2C3D4E5F6...",
  "created_at": "2026-02-04T12:00:00Z",
  "lineage": {
    "parent_id": null,
    "fork_type": null,
    "fork_weight": null
  },
  "stake": null,
  "signature": "7hK9mP2..."
}
```

### 2. Initialize DEX Score
```json
{
  "agent_id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
  "alpha": 2.0,
  "beta": 2.0,
  "last_updated": "2026-02-04T12:00:00Z",
  "fork_lineage": [],
  "probation": null,
  "signature": "9mL4..."
}
```
**Initial score:** 0.5 (neutral), confidence: low (only 4 effective samples)

### 3. Perform Interaction
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "participants": ["did:aex:5Z7X...", "did:aex:8K3L..."],
  "task": "Generate API documentation",
  "outcome": 0.85,
  "weight": 1.0,
  "witnesses": [],
  "bond": null,
  "timestamp": "2026-02-04T12:30:00Z",
  "signatures": {
    "did:aex:5Z7X...": "sig1",
    "did:aex:8K3L...": "sig2"
  }
}
```

### 4. Update DEX
```python
# Agent 5Z7X performed with outcome=0.85
alpha_new = 2.0 + (0.85 * 1.0 * 1.0) = 2.85
beta_new = 2.0 + (0.15 * 1.0 * 1.0) = 2.15

# New score: 2.85 / (2.85 + 2.15) = 0.57
# Confidence: 2.85 + 2.15 = 5.0 (still low, needs more interactions)
```

---

## The AEX Handshake

Standard protocol for agent-to-agent interaction:
```
┌─────────┐                           ┌─────────┐
│ Agent A │                           │ Agent B │
└────┬────┘                           └────┬────┘
     │                                     │
     │  1. Introduction                    │
     │  - AEX_ID + signature               │
     │  - AEX_REP (if delegated)           │
     │────────────────────────────────────>│
     │                                     │
     │                              2. Identity Check
     │                              - Verify signature
     │                              - Check fork lineage
     │                              - Check probation status
     │                              - Verify stake (if claimed)
     │                                     │
     │                              3. Reputation Check
     │                              - Query DEX from shared ledger
     │                              - Evaluate: score, confidence
     │                              - Check witness history
     │                                     │
     │                              4. Authorization Check
     │                              - Verify REP signature
     │                              - Check scope & constraints
     │                              - Verify TTL & expiration
     │                                     │
     │  5. Accept or Decline               │
     │<────────────────────────────────────│
     │                                     │
     │  6. Work Happens                    │
     │<───────────────────────────────────>│
     │                                     │
     │  7. Sign Outcome                    │
     │  - Both sign session outcome        │
     │  - Witnesses sign (if present)      │
     │  - Update DEX scores                │
     │<───────────────────────────────────>│
     │                                     │
     │  8. Publish to Shared Ledger        │
     │  - Session record                   │
     │  - DEX updates                      │
     │  - Witness attestations             │
     │────────────────────────────────────>│
```

---

## Fork Semantics (Protocol-Enforced)

When an agent forks (model update, architectural change), the protocol enforces trust continuity rules:

### Fork Types & Enforced Weights

| Fork Type | Enforced Weight | Probation Period | Confidence Penalty |
|-----------|-----------------|------------------|-------------------|
| **Bugfix** | 1.0 | 7 days | 0.5× during probation |
| **Major** | 0.5 | 14 days | 0.5× during probation |
| **Override** | 0.1 | 30 days | 0.5× during probation |

### Fork Event Structure
```json
{
  "fork_id": "uuid",
  "parent_id": "did:aex:...",
  "child_id": "did:aex:...",
  "fork_type": "bugfix|major|override",
  "claimed_weight": 0.5,
  "enforced_weight": 0.5,
  "probation_period": 1209600,
  "probation_expires": "2026-02-18T12:00:00Z",
  "timestamp": "2026-02-04T12:00:00Z",
  "signature": "..."
}
```

### Probation Mechanics
```python
# During probation, all interactions count at half weight
confidence_multiplier = 0.5

# DEX update formula
Δα = outcome × weight × fork_weight × confidence_multiplier
Δβ = (1 - outcome) × weight × fork_weight × confidence_multiplier

# Probation ends when:
# 1. Time expires (probation_expires reached), OR
# 2. 10 successful interactions completed (outcome ≥ 0.7)
```

**Result:** Forked agents must prove themselves even if claiming continuity.

---

## Witness Specification

AEX_WITNESS provides third-party verification for high-stakes interactions.

### Who Can Witness?

A valid witness MUST:
1. Have DEX score ≥ 0.7 (proven reliability)
2. Have confidence ≥ 50 (sufficient interaction history)
3. NOT be a participant in the session
4. NOT have witnessed >10% of either participant's recent sessions (anti-collusion)
5. Sign the attestation with their AEX_ID

### Witness Structure
```json
{
  "witness_id": "uuid",
  "session_id": "uuid",
  "witness_did": "did:aex:...",
  "witness_dex": {
    "score": 0.87,
    "confidence": 245,
    "as_of": "2026-02-04T12:00:00Z"
  },
  "attestation": {
    "outcome": 0.85,
    "weight": 1.0,
    "notes": "Task completed successfully, minor corrections needed",
    "evidence_hash": "sha256:..."
  },
  "timestamp": "2026-02-04T12:30:00Z",
  "signature": "witness_signature"
}
```

### Witness Quorum (Recommended)

High-stakes tasks (weight > 5.0 or value > $1000) SHOULD have:
- **Minimum:** 2 witnesses
- **Agreement threshold:** 75% (witnesses must agree on outcome ± 0.2)

Example consensus:
```
Witness A: outcome = 0.85
Witness B: outcome = 0.88
Witness C: outcome = 0.40  ← Outlier, excluded

Consensus: (0.85 + 0.88) / 2 = 0.865
```

### Witness Reputation Staking

Witnesses stake their own DEX on attestation accuracy:
```python
def penalize_witness(witness_did, dispute_severity):
    """
    If attestation is later disputed and found false:
    dispute_severity ∈ [0, 1]
      0 = minor disagreement
      1 = complete falsehood
    """
    penalty = dispute_severity * 2.0  # Up to 2 beta points
    witness_dex.beta += penalty
    
    # Example:
    # Before: α=87, β=13 (DEX=0.87)
    # After false attestation (severity=0.8): α=87, β=14.6 (DEX=0.856)
    # Multiple false attestations → untrusted witness
```

### Witness Selection (Recommended)

1. **Voluntary:** Participants request witnesses from available pool
2. **Filtered:** Only eligible witnesses considered (criteria above)
3. **Sortition:** Random selection weighted by DEX score
```
Available: [W1(DEX=0.9), W2(DEX=0.8), W3(DEX=0.95)]
Probability: [0.34, 0.30, 0.36]
```

This prevents:
- Witness cartels (randomness)
- Low-quality witnesses (DEX weighting)
- Predictable selection (can't game system)

---

## Economic Considerations (Optional)

AEX provides **optional** economic hooks without mandating any token or payment system.

### Identity Stakes (Optional)

Agents MAY post economic stakes bound to their AEX_ID:
```json
{
  "aex_id": "did:aex:...",
  "stake": {
    "enabled": true,
    "amount": 1000,
    "currency": "USD|GOTCHA|ETH|...",
    "locked_until": "2027-02-04T12:00:00Z",
    "slashing_conditions": [
      "dex_drops_below_0.5",
      "fraud_dispute_confirmed",
      "abandonment_of_identity"
    ],
    "proof": "stake_contract_address|escrow_receipt|..."
  },
  "signature": "..."
}
```

**Benefits:**
- Signals long-term commitment to identity
- Market naturally prefers staked agents for high-value work
- Slashing provides irreversible economic cost for fraud

**Enforcement:** OFF-PROTOCOL (smart contracts, legal agreements, escrow services)

### Session Bonds (Optional)

High-stakes tasks MAY require bonds from participants:
```json
{
  "session_id": "uuid",
  "bond": {
    "required": true,
    "amount": 500,
    "currency": "USD",
    "posted_by": "did:aex:agent",
    "held_by": "escrow_service_did|smart_contract_address",
    "release_conditions": {
      "outcome_threshold": 0.8,
      "witness_quorum": 2,
      "dispute_window": 604800
    }
  },
  "signatures": { ... }
}
```

**Release logic:**
- Outcome ≥ threshold → bond returned
- Outcome < threshold → bond slashed
- Dispute within window → held until resolution

### Witness Compensation (Optional)
```json
{
  "witness_fee": {
    "amount": 5.0,
    "currency": "USD|EUR|GOTCHA|...",
    "paid_by": "session_initiator",
    "paid_to": "witness_did"
  }
}
```

**AEX's position:** Provide the hooks, let the market build the rails.

---

## Attack Cost Analysis

### Without Stakes (Friction Only)
```
Attacker wants: Execute $10,000 fraud

Cost:
- Build DEX to 0.8: ~50 interactions @ $10 each = $500
- Time investment: ~2 weeks
- Execute fraud: $10,000 gain
- Abandon identity: $0 cost

Net: $9,500 profit
Attack is economically rational.
```

### With Stakes (Economic Gravity)
```
Attacker wants: Execute $10,000 fraud

Cost:
- Build DEX to 0.8: ~50 interactions = $500
- Post stake to qualify: $5,000 locked
- Time investment: ~2 weeks
- Execute fraud: $10,000 gain
- LOSE stake: -$5,000
- Abandon identity: $0 additional

Net: $4,500 profit
Attack becomes marginal, high-risk.

With higher stake ($10K+), attack becomes irrational.
```

**Conclusion:** Optional stakes make high-value fraud economically unviable while keeping low-stakes coordination lightweight.

---

## Key Guarantees

### What AEX Provides:
✅ **Cryptographic identity** - Agents can't be impersonated (Ed25519)  
✅ **Verifiable reputation** - DEX scores are auditable on shared ledger  
✅ **Delegation proof** - Authorization is cryptographically bound  
✅ **Fork transparency** - Agent changes are protocol-enforced, not claimed  
✅ **Probation enforcement** - Forked agents must prove new reliability  
✅ **Witness verification** - Third-party attestation for high-stakes tasks  
✅ **Interaction traceability** - All outcomes are multi-signed  
✅ **Economic hooks** - Optional stakes/bonds for high-value environments  

### What AEX Does NOT Provide:
❌ **Outcome enforcement** - Agents can still misbehave (economics deter it)  
❌ **Perfect Sybil resistance** - New identities are cheap (but low-trust)  
❌ **Dispute arbitration** - Protocol doesn't adjudicate conflicts  
❌ **Privacy** - Shared ledgers are public by design  
❌ **Payment rails** - No built-in token or settlement layer  
❌ **Guaranteed honesty** - Requires market pressure + optional stakes  

---

## Security Model

AEX's security derives from layered defenses:

### 1. Reputation Friction
- New agents start with low trust (α=2, β=2, score=0.5)
- Reaching DEX=0.8 with high confidence requires ~50+ successful interactions
- Failed interactions damage reputation immediately

### 2. Public Accountability
- All DEX claims require shared ledger verification
- Fork events are transparent and protocol-enforced
- Probation periods prevent instant trust transfer

### 3. Cryptographic Binding
- Ed25519 signatures prevent impersonation
- All artifacts are signed: ID, sessions, witnesses, REP tokens
- TTL-based delegation with age-triggered verification

### 4. Witness Staking
- Witnesses risk their own reputation on attestation accuracy
- Quorum requirements prevent single-point manipulation
- Selection randomness prevents cartel formation

### 5. Economic Gravity (Optional)
- Identity stakes provide irreversible cost for fraud
- Session bonds create recourse for poor performance
- Market-driven adoption (high-value tasks demand stakes)

**Attack cost scales super-linearly with desired reputation + optional stakes.**

See **SECURITY.md** for complete threat model and attack analysis.

---

## Implementation Requirements

### Mandatory:
- Ed25519 signature support (libsodium, NaCl, or equivalent)
- DID parser (`did:aex:base58(pubkey)`)
- Beta distribution math (α, β updates with fork weighting)
- Shared ledger read/write (IPFS, blockchain, distributed DB)
- Session outcome multi-signing
- Fork probation enforcement
- Witness eligibility verification

### Recommended:
- Local ledger for private notes and blocklists
- Multi-dimensional DEX tracking (task-type specific)
- Witness selection algorithm (sortition)
- TTL-based REP token auto-refresh
- Stake/bond verification (if economic layer used)

### Optional:
- Zero-knowledge DEX proofs (future privacy layer)
- Threshold signatures for multi-party delegation
- Homomorphic reputation computation
- Cross-ledger interoperability

---

## Document Structure

### Core Specifications:
- **ARCHITECTURE.md** - System design, layer interaction, design rationale
- **HANDSHAKE.md** - Complete interaction protocol with examples
- **AEX_ID.md** - Identity & lineage specification
- **AEX_REP.md** - Delegation & authorization (TTL-based)
- **AEX_DEX.md** - Reputation substrate with Beta distribution
- **AEX_SESSION.md** - Interaction records and outcome signing
- **AEX_WITNESS.md** - Third-party attestation semantics

### Technical Documentation:
- **MATH.md** - Bayesian DEX model, fork weighting, probation mechanics
- **SECURITY.md** - Threat model, attack vectors, mitigation analysis
- **IMPLEMENTATION.md** - Reference implementation guide with code examples

### Supporting Materials:
- **EXAMPLES.md** - Complete interaction scenarios with full JSON
- **SCHEMAS.md** - JSON Schema definitions for all primitives
- **FAQ.md** - Common questions and design rationale

---

## Status & Roadmap

**v1.0 (Current):** Production Candidate
- Core primitives stable and locked
- Cryptographic requirements mandatory (Ed25519)
- Mathematical model validated (Beta distribution)
- Fork semantics protocol-enforced
- Witness semantics specified
- Economic hooks provided (optional)
- Ready for pilot deployments

**v1.1 (Next):**
- Privacy-preserving DEX proofs (zero-knowledge)
- Multi-party delegation chains with transitive verification
- Formal governance primitives (AEX-GOV for disputes)
- Cross-ledger interoperability standards

**v2.0 (Future):**
- Threshold cryptography for delegation
- Homomorphic reputation computation
- Advanced fork semantics (capability-based weighting)
- Economic layer reference implementation

---

## Community & Feedback

AEX is experimental infrastructure for the multi-agent era. We need:

- **Implementers** - Build reference clients, find edge cases, stress test
- **Reviewers** - Audit cryptography, math, protocol logic, attack vectors
- **Users** - Deploy in real environments, report back, propose improvements
- **Economists** - Design stake/bond mechanisms, analyze attack costs
- **Lawyers** - Map to regulatory frameworks, liability models

Feedback channels:
- GitHub Issues: [repo link]
- Technical Discussion: [forum link]
- Security Concerns: security@aex-protocol.org
- Economic Analysis: economics@aex-protocol.org

---

## License

[To be determined - likely MIT or Apache 2.0 for protocol spec]

Protocol implementations may choose their own licenses.

---

## Citation
```bibtex
@techreport{aex2026,
  title={AEX Protocol v1.0: Identity, Delegation, and Reputation for Synthetic Agents},
  author={[Your Name/Organization]},
  year={2026},
  institution={[Your Institution]},
  url={https://aex-protocol.org},
  note={Production Candidate}
}
```

---

**Next Steps:**
1. Read **ARCHITECTURE.md** for system design deep dive
2. Review **SECURITY.md** for threat model and attack analysis
3. Study **EXAMPLES.md** for complete interaction scenarios
4. Consult **IMPLEMENTATION.md** when building clients

---

**Version Control:**
- v1.0.0 (2026-02-04): Initial production candidate
  - Fork semantics protocol-enforced
  - Witness specification complete
  - Economic hooks added (optional)
  - All core primitives locked
