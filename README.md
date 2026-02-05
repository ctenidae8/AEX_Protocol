# AEX Protocol v1.0
## Agent Experience Protocol - Identity, Delegation, Reputation, and Experience for Synthetic Agents

**Status:** Production Candidate  
**Version:** 1.0.0  
**Last Updated:** February 5, 2026

---

## Overview

The AEX Protocol provides the foundational substrate for multi-agent coordination, enabling synthetic agents to:

- **Establish cryptographic identity** with lineage tracking
- **Delegate authority** from humans to agents and between agents
- **Build verifiable reputation** through observable behavior
- **Signal domain expertise** through accumulated experience
- **Coordinate safely** without centralized control

AEX is not a framework, runtime, or marketplace. It is a **protocol layer** that makes those things possible.

---

## Core Architecture

AEX consists of **four core primitives** organized in layers:

```
┌─────────────────────────────────────────────────┐
│                  AEX_HEX                         │
│           Experience & Capability                │
│        "Are you capable in this domain?"         │
├─────────────────────────────────────────────────┤
│                  AEX_DEX                         │
│           Behavioral Reputation                  │
│          "How reliable are you?"                 │
├─────────────────────────────────────────────────┤
│                  AEX_REP                         │
│           Delegation & Authorization             │
│         "Who are you acting for?"                │
├─────────────────────────────────────────────────┤
│                  AEX_ID                          │
│           Identity & Lineage                     │
│             "Who are you?"                       │
└─────────────────────────────────────────────────┘

Plus Supporting Primitives:
• AEX_SESSION - Interaction binding ("What happened?")
• AEX_WITNESS - Third-party attestation ("Can anyone verify this?")
```

### The Four Questions

| Layer | Primitive | Purpose | Answers |
|-------|-----------|---------|---------|
| **1** | **AEX_ID** | Persistent identity & lineage | "Who are you?" |
| **2** | **AEX_REP** | Delegation & authorization | "Who are you acting for?" |
| **3** | **AEX_DEX** | Behavioral reliability | "How reliable are you?" |
| **4** | **AEX_HEX** | Experience & capability | "Are you capable in this domain?" |

**Selection logic:**
- **DEX** filters by trust threshold (reliability gate)
- **HEX** ranks by domain experience (capability matching)
- Together they answer: *"Is this agent both trustworthy AND the right specialist?"*

---

## Design Principles

### 1. Cryptographic Verifiability
All identity and reputation claims are Ed25519-signed. No exceptions.

### 2. Behavioral Grounding
Reputation (DEX) emerges from observable outcomes, not self-reported capabilities.

### 3. Experience Accumulation
Capability (HEX) is built through documented interactions in specific domains.

### 4. Fork Awareness
Agent updates (model changes, rewrites) are first-class events with protocol-enforced trust impact and mandatory probation periods.

### 5. Interoperability
Uses W3C DID standard. Portable across platforms, ecosystems, and implementations.

### 6. Shared Ledger for Trust
DEX scores and HEX claims require public, verifiable ledgers. Private reputation is not trustworthy reputation.

### 7. Economic Optionality
Provides hooks for stakes and bonds without mandating any token or payment system.

---

## Quick Start

### 1. Create an Agent Identity
```json
{
  "aex_id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
  "public_key": "ed25519:A1B2C3D4E5F6...",
  "created_at": "2026-02-05T12:00:00Z",
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
  "last_updated": "2026-02-05T12:00:00Z",
  "fork_lineage": [],
  "probation": null,
  "signature": "9mL4..."
}
```
**Initial score:** 0.5 (neutral), confidence: low (only 4 effective samples)

### 3. Initialize HEX Corpus
```json
{
  "agent_id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
  "experience": [],
  "traits": {
    "revision_tolerance": "moderate",
    "interaction_style": "professional"
  },
  "last_updated": "2026-02-05T12:00:00Z",
  "signature": "4pT8..."
}
```
**Initial state:** No domain experience yet

### 4. Perform Interaction
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "participants": ["did:aex:5Z7X...", "did:aex:8K3L..."],
  "task_summary": {
    "description": "Generate API documentation",
    "domains": ["documentation", "api_design"],
    "actual_cost": 48
  },
  "outcome": {
    "agreed_outcome": 0.85,
    "agent_a_rating": 0.85,
    "agent_b_rating": 0.85
  },
  "weight": 1.0,
  "witnesses": [],
  "timestamp": "2026-02-05T12:30:00Z",
  "signatures": {
    "did:aex:5Z7X...": "sig1",
    "did:aex:8K3L...": "sig2"
  }
}
```

### 5. Update DEX and HEX
```python
# DEX Update (behavioral reliability)
# Agent 5Z7X performed with outcome=0.85
alpha_new = 2.0 + (0.85 * 1.0) = 2.85
beta_new = 2.0 + (0.15 * 1.0) = 2.15
# New DEX score: 2.85 / (2.85 + 2.15) = 0.57

# HEX Update (domain experience)
# Add experience in "documentation" and "api_design" domains
hex_corpus['experience'].append({
  "domain": "documentation",
  "count": 1,
  "confidence": 0.85,
  "last_updated": "2026-02-05T12:30:00Z"
})
hex_corpus['experience'].append({
  "domain": "api_design",
  "count": 1,
  "confidence": 0.85,
  "last_updated": "2026-02-05T12:30:00Z"
})
```

**After 50 interactions:**
- **DEX:** 0.83 (high reliability)
- **HEX:** documentation (50 tasks, confidence: 0.87), api_design (50 tasks, confidence: 0.85)
- **Result:** Agent is both trustworthy (DEX) and proven specialist (HEX)

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
     │  - AEX_HEX summary                  │
     │────────────────────────────────────>│
     │                                     │
     │                              2. Identity Check
     │                              - Verify signature
     │                              - Check fork lineage
     │                              - Check probation status
     │                              - Verify stake (if claimed)
     │                                     │
     │                              3. Reputation Check (DEX)
     │                              - Query DEX from shared ledger
     │                              - Evaluate: score, confidence
     │                              - Filter: Is DEX ≥ threshold?
     │                                     │
     │                              4. Capability Check (HEX)
     │                              - Query HEX from shared ledger
     │                              - Match required domains
     │                              - Rank by domain experience
     │                                     │
     │                              5. Authorization Check
     │                              - Verify REP signature
     │                              - Check scope & constraints
     │                              - Verify TTL & expiration
     │                                     │
     │  6. Accept or Decline               │
     │<────────────────────────────────────│
     │                                     │
     │  7. Work Happens                    │
     │<───────────────────────────────────>│
     │                                     │
     │  8. Sign Outcome                    │
     │  - Both sign session outcome        │
     │  - Witnesses sign (if present)      │
     │  - Update DEX scores                │
     │  - Update HEX experience            │
     │<───────────────────────────────────>│
     │                                     │
     │  9. Publish to Shared Ledger        │
     │  - Session record                   │
     │  - DEX updates                      │
     │  - HEX updates                      │
     │  - Witness attestations             │
     │────────────────────────────────────>│
```

---

## Agent Selection: DEX + HEX

The protocol uses a two-stage selection process:

### Stage 1: DEX Filter (Trust Gate)
```python
def filter_by_trust(candidates, min_dex=0.75):
    """Filter out unreliable agents"""
    trusted = []
    for agent in candidates:
        dex = get_dex(agent['did'])
        if dex['score'] >= min_dex:
            trusted.append(agent)
    return trusted
```

### Stage 2: HEX Ranking (Capability Matching)
```python
def rank_by_capability(trusted_agents, required_domain):
    """Rank agents by domain experience"""
    for agent in trusted_agents:
        hex_corpus = get_hex(agent['did'])
        
        # Find experience in required domain
        domain_exp = find_domain(hex_corpus, required_domain)
        
        if domain_exp:
            agent['hex_score'] = domain_exp['count'] * domain_exp['confidence']
        else:
            agent['hex_score'] = 0
    
    # Sort by HEX score (highest first)
    return sorted(trusted_agents, key=lambda x: x['hex_score'], reverse=True)
```

### Combined Selection
```python
def select_specialist(candidates, min_dex=0.75, required_domain="legal.contracts"):
    """Select the best specialist for a task"""
    # Stage 1: Filter by reliability
    trusted = filter_by_trust(candidates, min_dex)
    
    if not trusted:
        return None, "No trusted agents available"
    
    # Stage 2: Rank by capability
    specialists = rank_by_capability(trusted, required_domain)
    
    if specialists[0]['hex_score'] == 0:
        return None, f"No agents with experience in {required_domain}"
    
    return specialists[0], "Best specialist selected"
```

**Example outcome:**
```
Task: Legal contract review (legal.contracts domain)
Min DEX: 0.75

Candidates:
├─ Agent A: DEX=0.92, HEX(legal.contracts)=150 tasks → âœ… Selected
├─ Agent B: DEX=0.88, HEX(legal.contracts)=85 tasks  → ✓ Backup
├─ Agent C: DEX=0.93, HEX(marketing)=200 tasks       → âŒ Wrong domain
└─ Agent D: DEX=0.65, HEX(legal.contracts)=300 tasks → âŒ Unreliable

Result: Select Agent A (high trust + best specialist)
```

---

## Fork Semantics (Protocol-Enforced)

When an agent forks (model update, architectural change), the protocol enforces trust continuity rules for both DEX and HEX:

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
  "parent_dex_snapshot": {
    "alpha": 100.0,
    "beta": 20.0,
    "score": 0.833
  },
  "parent_hex_snapshot": {
    "total_experience": 120,
    "domain_count": 5,
    "top_domains": ["api_design", "documentation"]
  },
  "timestamp": "2026-02-05T12:00:00Z",
  "signature": "..."
}
```

### Inheritance Rules

**DEX Inheritance:**
```python
# Child inherits discounted reputation
child_alpha = parent_alpha × fork_weight = 100.0 × 0.5 = 50.0
child_beta = parent_beta × fork_weight = 20.0 × 0.5 = 10.0
# Child DEX: 50/(50+10) = 0.833 (same ratio, lower confidence)
```

**HEX Inheritance:**
```python
# Child inherits discounted experience
for domain in parent_hex['experience']:
    child_hex['experience'].append({
        'domain': domain['domain'],
        'count': domain['count'] × fork_weight,
        'confidence': domain['confidence'] × 0.8,  # Slight penalty
        'last_updated': fork_timestamp
    })
```

### Probation Mechanics
```python
# During probation, all interactions count at half weight
confidence_multiplier = 0.5

# DEX update formula
Δα = outcome × weight × fork_weight × confidence_multiplier
Δβ = (1 - outcome) × weight × fork_weight × confidence_multiplier

# HEX update formula
experience_count += 1 × confidence_multiplier

# Probation ends when:
# 1. Time expires (probation_expires reached), OR
# 2. 10 successful interactions completed (outcome ≥ 0.7)
```

**Result:** Forked agents inherit capabilities but must prove new reliability.

---

## Witness Specification

AEX_WITNESS provides third-party verification for high-stakes interactions.

### Who Can Witness?

A valid witness MUST:
1. Have DEX score ≥ 0.7 (proven reliability)
2. Have confidence ≥ 50 (sufficient interaction history)
3. Have HEX experience in relevant domain (for specialized tasks)
4. NOT be a participant in the session
5. NOT have witnessed >10% of either participant's recent sessions (anti-collusion)
6. Sign the attestation with their AEX_ID

### Witness Structure
```json
{
  "witness_id": "uuid",
  "session_id": "uuid",
  "witness_did": "did:aex:...",
  "witness_dex": {
    "score": 0.87,
    "confidence": 245,
    "as_of": "2026-02-05T12:00:00Z"
  },
  "witness_hex": {
    "relevant_domain": "legal.contracts",
    "domain_experience": 78,
    "confidence": 0.91
  },
  "attestation": {
    "outcome": 0.85,
    "weight": 1.0,
    "notes": "Task completed successfully, minor corrections needed",
    "evidence_hash": "sha256:..."
  },
  "timestamp": "2026-02-05T12:30:00Z",
  "signature": "witness_signature"
}
```

### Witness Quorum (Recommended)

High-stakes tasks (weight > 5.0 or value > $1000) SHOULD have:
- **Minimum:** 2 witnesses
- **Agreement threshold:** 75% (witnesses must agree on outcome ± 0.2)
- **Domain expertise:** At least 1 witness with HEX in relevant domain

Example consensus:
```
Task: Legal contract review (legal.contracts domain)

Witness A: DEX=0.89, HEX(legal.contracts)=120, outcome=0.85
Witness B: DEX=0.91, HEX(legal.contracts)=95,  outcome=0.88
Witness C: DEX=0.85, HEX(marketing)=200,       outcome=0.40  ← Outlier (wrong domain)

Consensus: (0.85 + 0.88) / 2 = 0.865 (only domain experts count)
```

---

## Optional Economic Layer

AEX provides **hooks** for stakes and bonds without mandating their use.

### Identity Stakes (Optional)
```json
{
  "aex_id": "did:aex:...",
  "stake": {
    "amount": 5000,
    "currency": "USD",
    "locked_at": "2026-01-15T00:00:00Z",
    "escrow": "escrow_service_did",
    "conditions": {
      "dex_threshold": 0.7,
      "slash_on_fraud": true,
      "release_delay": 2592000
    }
  }
}
```

### Session Bonds (Optional)
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
- Build HEX in valuable domain: Included in DEX building
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
- Build HEX in valuable domain: Included
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
✅ **Verifiable experience** - HEX claims are backed by interaction history  
✅ **Delegation proof** - Authorization is cryptographically bound  
✅ **Fork transparency** - Agent changes are protocol-enforced, not claimed  
✅ **Probation enforcement** - Forked agents must prove new reliability  
✅ **Capability signaling** - Domain expertise is accumulated and verifiable  
✅ **Specialist discovery** - Find agents with proven experience in specific domains  
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
❌ **Capability verification** - HEX shows experience, not technical ability  

---

## Security Model

AEX's security derives from layered defenses:

### 1. Reputation Friction
- New agents start with low trust (α=2, β=2, score=0.5)
- Reaching DEX=0.8 with high confidence requires ~50+ successful interactions
- Failed interactions damage reputation immediately

### 2. Experience Accumulation
- New agents have empty HEX (no domain experience)
- Building credible HEX in valuable domains requires time and consistency
- HEX inflation is detectable (claimed experience vs. ledger history)

### 3. Public Accountability
- All DEX and HEX claims require shared ledger verification
- Fork events are transparent and protocol-enforced
- Probation periods prevent instant trust transfer

### 4. Cryptographic Binding
- Ed25519 signatures prevent impersonation
- All artifacts are signed: ID, sessions, witnesses, REP tokens, DEX, HEX
- TTL-based delegation with age-triggered verification

### 5. Witness Staking
- Witnesses risk their own reputation on attestation accuracy
- Domain-expert witnesses for specialized tasks
- Quorum requirements prevent single-point manipulation
- Selection randomness prevents cartel formation

### 6. Economic Gravity (Optional)
- Identity stakes provide irreversible cost for fraud
- Session bonds create recourse for poor performance
- Market-driven adoption (high-value tasks demand stakes)

**Attack cost scales super-linearly with desired reputation + domain experience + optional stakes.**

See **SECURITY.md** for complete threat model and attack analysis.

---

## Implementation Requirements

### Mandatory:
- Ed25519 signature support (libsodium, NaCl, or equivalent)
- DID parser (`did:aex:base58(pubkey)`)
- Beta distribution math (α, β updates with fork weighting)
- HEX accumulation logic (domain tracking, confidence calculation)
- Shared ledger read/write (IPFS, blockchain, distributed DB)
- Session outcome multi-signing
- Fork probation enforcement
- Witness eligibility verification

### Recommended:
- Local ledger for private notes and blocklists
- Multi-dimensional DEX tracking (task-type specific)
- HEX domain matching algorithms
- Witness selection algorithm (sortition)
- TTL-based REP token auto-refresh
- Stake/bond verification (if economic layer used)

### Optional:
- Zero-knowledge DEX proofs (future privacy layer)
- Threshold signatures for multi-party delegation
- Homomorphic reputation computation
- Cross-ledger interoperability
- HEX domain similarity metrics

---

## Document Structure

### Core Specifications:
- **ARCHITECTURE.md** - System design, four-layer model, design rationale
- **HANDSHAKE.md** - Complete interaction protocol with DEX+HEX selection
- **AEX_ID.md** - Identity & lineage specification with fork inheritance
- **AEX_REP.md** - Delegation & authorization (TTL-based)
- **AEX_DEX.md** - Behavioral reputation with Beta distribution
- **AEX_HEX.md** - Experience corpus and capability signaling
- **AEX_SESSION.md** - Interaction records and outcome signing
- **AEX_WITNESS.md** - Third-party attestation semantics
- **AEX_SCHEMA.md** - JSON Schema definitions for all primitives

### Technical Documentation:
- **MATH.md** - Bayesian DEX model, fork weighting, probation mechanics
- **SECURITY.md** - Threat model, attack vectors, HEX inflation attacks
- **IMPLEMENTATION.md** - Reference implementation guide with code examples

### Supporting Materials:
- **EXAMPLES.md** - Complete interaction scenarios with DEX+HEX selection
- **Quick Start Guide.md** - Tutorial walkthrough with all four primitives

---

## Status & Roadmap

**v1.0 (Current):** Production Candidate
- Four-layer architecture complete (ID, REP, DEX, HEX)
- Core primitives stable and locked
- Cryptographic requirements mandatory (Ed25519)
- Mathematical model validated (Beta distribution)
- Fork semantics protocol-enforced for both DEX and HEX
- Experience accumulation specified
- Witness semantics specified
- Economic hooks provided (optional)
- Ready for pilot deployments

**v1.1 (Next):**
- Privacy-preserving DEX/HEX proofs (zero-knowledge)
- Multi-party delegation chains with transitive verification
- HEX domain similarity metrics and matching algorithms
- Formal governance primitives (AEX-GOV for disputes)
- Cross-ledger interoperability standards

**v2.0 (Future):**
- Threshold cryptography for delegation
- Homomorphic reputation computation
- Advanced fork semantics (capability-based weighting)
- HEX confidence decay models
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
  title={AEX Protocol v1.0: Identity, Delegation, Reputation, and Experience for Synthetic Agents},
  author={[Your Name/Organization]},
  year={2026},
  institution={[Your Institution]},
  url={https://aex-protocol.org},
  note={Production Candidate}
}
```

---

**Next Steps:**
1. Read **ARCHITECTURE.md** for four-layer system design deep dive
2. Read **AEX_HEX.md** for experience corpus specification
3. Review **SECURITY.md** for threat model and HEX attack analysis
4. Study **EXAMPLES.md** for complete DEX+HEX selection scenarios
5. Consult **IMPLEMENTATION.md** when building clients

---

**Version Control:**
- v1.0.0 (2026-02-05): Production candidate with HEX integration
  - Four-layer architecture (ID, REP, DEX, HEX)
  - Fork semantics protocol-enforced for both DEX and HEX
  - Experience accumulation and capability signaling
  - Specialist discovery and domain matching
  - Witness specification complete
  - Economic hooks added (optional)
  - All core primitives locked
