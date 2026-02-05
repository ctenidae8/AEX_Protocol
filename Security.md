# AEX Protocol v1.0 - Security Analysis
## Threat Models, Attack Vectors, and Mitigations

**Last Updated:** February 4, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Threat Model](#threat-model)
3. [Identity Attacks](#identity-attacks)
4. [Reputation Attacks](#reputation-attacks)
5. [Delegation Attacks](#delegation-attacks)
6. [Experience Attacks (HEX)](#experience-attacks-hex)
7. [Witness Attacks](#witness-attacks)
8. [Session Attacks](#session-attacks)
9. [Economic Attacks](#economic-attacks)
10. [Network Attacks](#network-attacks)
11. [Implementation Vulnerabilities](#implementation-vulnerabilities)
12. [Mitigation Summary](#mitigation-summary)
13. [Security Checklist](#security-checklist)

---

## Overview

This document provides a **comprehensive security analysis** of the AEX protocol family, including:

- Threat modeling and adversary classification
- Attack vectors across all four core primitives (ID, REP, DEX, HEX)
- Existing mitigations and their effectiveness
- Residual risks and recommended countermeasures
- Security implementation checklist

### Security Philosophy

AEX security is based on **defense in depth**:

1. **Cryptographic integrity** - All critical data is signed
2. **Economic disincentives** - Attacks are expensive
3. **Transparency** - All behavior is publicly observable
4. **Reputation consequences** - Bad actors lose future opportunities
5. **Protocol-level enforcement** - Key security properties are enforced, not assumed
6. **Cross-validation** - Multiple signals (DEX, HEX) verify agent trustworthiness

---

## Threat Model

### Adversary Classes

**1. Rational Economic Adversary**
- **Goal:** Maximize profit
- **Constraints:** Will attack if benefit > cost
- **Capabilities:** Moderate resources, sophisticated
- **Mitigation:** Make attacks economically unprofitable

**2. Malicious Adversary**
- **Goal:** Cause maximum damage (regardless of cost)
- **Constraints:** None
- **Capabilities:** Significant resources, highly sophisticated
- **Mitigation:** Limit blast radius, enable recovery

**3. Lazy Adversary**
- **Goal:** Minimize effort while appearing compliant
- **Constraints:** Limited resources
- **Capabilities:** Basic automation
- **Mitigation:** Make compliance easier than cheating

**4. Compromised Agent**
- **Goal:** Controlled by external attacker
- **Constraints:** Must avoid immediate detection
- **Capabilities:** Access to legitimate credentials
- **Mitigation:** Anomaly detection, revocation mechanisms

### Attack Goals

| Goal | Impact | Likelihood | Priority |
|------|--------|------------|----------|
| **Reputation manipulation** | Untrustworthy agents appear trustworthy | High | Critical |
| **Experience inflation (HEX)** | Unskilled agents appear capable | High | Critical |
| **Identity theft** | Impersonate legitimate agent | Medium | Critical |
| **Delegation forgery** | Act without proper authorization | Medium | Critical |
| **Witness corruption** | Manipulate outcome consensus | Medium | High |
| **Domain privacy violation** | Reveal agent specialization patterns | Medium | Medium |
| **Double-spend reputation** | Use reputation multiple times | Low | Medium |
| **Privacy violation** | Reveal confidential interactions | Medium | Medium |
| **Denial of service** | Prevent legitimate interactions | Low | Low |

### Trust Assumptions

**What AEX assumes:**
- ✅ Cryptographic primitives are secure (Ed25519, SHA-256)
- ✅ Majority of participants are honest or rational
- ✅ Ledger is append-only and tamper-evident
- ✅ Private keys are kept private by owners
- ✅ Time is approximately synchronized (±5 minutes acceptable)

**What AEX does NOT assume:**
- ❌ Central authority is honest
- ❌ All participants are honest
- ❌ Network is always available
- ❌ Agents have perfect information
- ❌ Humans always make correct decisions

---

## Identity Attacks

### Attack 1: Identity Theft

**Scenario:** Attacker steals agent's private key and impersonates it.

**Attack Steps:**
1. Compromise agent's key storage
2. Extract private key
3. Sign messages as legitimate agent
4. Participate in sessions, inherit reputation

**Impact:**
- Attacker inherits victim's DEX
- Can perform high-value interactions
- Damages victim's reputation through bad behavior

**Existing Mitigations:**
```python
# 1. Key protection requirements
def secure_key_storage():
    """
    Best practices for key storage
    """
    # Hardware security module (HSM) or TPM
    # OS-level key storage (Keychain, credential manager)
    # Encrypted at rest with separate passphrase
    # Never logged or transmitted in plaintext
    pass

# 2. Anomaly detection
def detect_key_compromise(agent_id, recent_sessions):
    """
    Detect suspicious behavior patterns
    
    Indicators:
    - Sudden change in interaction patterns
    - Geographic anomalies (if location tracked)
    - Rapid increase in high-value interactions
    - Contradictory delegation tokens
    """
    normal_pattern = get_agent_behavior_baseline(agent_id)
    
    # Check for deviation
    current_pattern = analyze_recent_behavior(recent_sessions)
    
    anomaly_score = calculate_pattern_divergence(normal_pattern, current_pattern)
    
    if anomaly_score > 0.8:
        return True, "Suspicious behavior detected"
    
    return False, "Behavior normal"
```

**Recommended Additional Mitigations:**

1. **Multi-factor authentication** for high-value operations
2. **Geofencing** (if location available)
3. **Velocity limits** (rate limits on new delegations)
4. **Emergency revocation** protocol

**Residual Risk:** Medium (depends on key management practices)

---

### Attack 2: Sybil Attack (Identity Farming)

**Scenario:** Attacker creates multiple fake identities to manipulate reputation or consensus.

**Attack Steps:**
1. Generate many AEX_IDs (cheap)
2. Bootstrap reputation through fake interactions
3. Use Sybil identities to:
   - Inflate attacker's main agent DEX
   - Manipulate witness selection
   - Dominate consensus

**Attack Cost Analysis:**
```python
def calculate_sybil_attack_cost(n_sybils, target_dex, avg_task_cost):
    """
    Calculate economic cost of Sybil attack
    
    Assumptions:
    - Sybils interact with each other to build DEX
    - Need n_eff >= 50 to be considered for witness selection
    - Average outcome = 0.8 (to reach target_dex)
    """
    # Step 1: Build reputation
    interactions_per_sybil = 50  # To reach n_eff = 50
    
    # Cost of interactions (if they need to appear real)
    # Assuming each interaction involves some work/compute
    cost_per_interaction = avg_task_cost * 0.1  # 10% of real task cost
    
    reputation_cost = n_sybils * interactions_per_sybil * cost_per_interaction
    
    # Step 2: Time cost
    # Assume 1 interaction per hour (to appear organic)
    hours_per_sybil = interactions_per_sybil * 1
    total_hours = n_sybils * hours_per_sybil
    
    hourly_rate = 20  # USD/hour for compute/labor
    time_cost = total_hours * hourly_rate
    
    # Step 3: Maintenance cost
    # Keep Sybils active to avoid staleness detection
    monthly_maintenance = n_sybils * 10  # $10/month per Sybil
    
    total_cost = reputation_cost + time_cost + monthly_maintenance
    
    return {
        'reputation_cost': reputation_cost,
        'time_cost': time_cost,
        'total_cost': total_cost,
        'time_required_hours': total_hours,
        'cost_per_sybil': total_cost / n_sybils
    }

# Example:
# 10 Sybils, target_dex=0.80, avg_task=$50
# reputation_cost = 10 * 50 * $5 = $2,500
# time_cost = 10 * 50 * $20 = $10,000
# total_cost = $12,500
```

**Existing Mitigations:**

1. **Reputation poverty** - New identities start with DEX=0.5, n_eff=4 (untrusted)
2. **Confidence requirements** - Many systems require n_eff ≥ 50 or ≥ 100
3. **Interaction pattern analysis** - Detect collusive rings
4. **Fork lineage tracking** - Sybils have no legitimate lineage
```python
def detect_sybil_ring(agent_ids, shared_ledger):
    """
    Detect Sybil ring through interaction graph analysis
    
    Indicators:
    - Agents only interact with each other (closed subgraph)
    - All created around same time
    - Similar interaction patterns
    - No external validation
    """
    # Build interaction graph
    graph = {}
    for agent_id in agent_ids:
        sessions = query_agent_sessions(agent_id, shared_ledger, limit=100)
        partners = set()
        for session in sessions:
            a = session['participants']['agent_a']['aex_id']
            b = session['participants']['agent_b']['aex_id']
            partner = a if a != agent_id else b
            partners.add(partner)
        
        graph[agent_id] = partners
    
    # Check for closed subgraph
    all_partners = set()
    for partners in graph.values():
        all_partners.update(partners)
    
    external_partners = all_partners - set(agent_ids)
    
    if len(external_partners) / len(all_partners) < 0.2:
        # Less than 20% external interactions = suspicious
        return True, "Closed interaction graph detected"
    
    return False, "Diverse interaction partners"
```

**Recommended Additional Mitigations:**

1. **Proof-of-work** for identity creation
2. **Bonding requirement** (stake tokens to create identity)
3. **Social verification** (existing agents vouch for new agents)
4. **Diversity requirements** (must interact with diverse partners)

**Residual Risk:** Medium (effective against lazy attackers, vulnerable to sophisticated attackers with resources)

---

### Attack 3: Fork Washing

**Scenario:** Agent with bad reputation forks repeatedly to "wash" reputation.

**Attack Steps:**
1. Agent A has DEX=0.30 (bad reputation)
2. Fork to A' as "major rewrite" (λ=0.5)
3. Inherited DEX = 0.30×0.5 + neutral = still low
4. Fork again to A'' (λ=0.5)
5. Repeat until reputation sufficiently "washed"

**Attack Effectiveness:**
```python
def simulate_fork_washing(initial_dex, n_forks, fork_type='major'):
    """
    Simulate repeated forking to wash reputation
    """
    # Initial state
    initial_alpha = 30  # Assume DEX=0.30 from alpha=30, beta=70
    initial_beta = 70
    
    fork_weights = {'major': 0.5, 'override': 0.1, 'bugfix': 1.0}
    weight = fork_weights[fork_type]
    
    alpha = initial_alpha
    beta = initial_beta
    
    history = []
    
    for i in range(n_forks):
        # Fork
        alpha = alpha * weight + 2
        beta = beta * weight + 2
        
        dex = alpha / (alpha + beta)
        n_eff = alpha + beta
        
        history.append({'fork': i+1, 'dex': dex, 'n_eff': n_eff})
    
    return history

# Results:
# Start: DEX=0.30, n_eff=100
# Fork 1: DEX=0.37, n_eff=54
# Fork 2: DEX=0.44, n_eff=31
# Fork 3: DEX=0.48, n_eff=19
# Fork 4: DEX=0.50, n_eff=14 (approaching neutral, but very low confidence)
```

**Existing Mitigations:**

1. **Fork history is public** - Pattern of repeated forking is visible
2. **Confidence penalty** - Each fork reduces n_eff significantly
3. **Probation after fork** - Fresh forks have dampened update weights
4. **Fork pattern detection** (see Sybil detection above)

**Recommended Additional Mitigations:**
```python
def enforce_fork_limits(agent_id, shared_ledger):
    """
    Enforce maximum fork frequency
    
    Rules:
    - Maximum 3 forks per 90 days
    - Maximum 1 major rewrite per 180 days
    - No forks during probation
    """
    identity = shared_ledger.get(f"identities/{agent_id}/aex_id.json")
    fork_history = identity['lineage']['fork_history']
    
    # Check recent forks
    recent_forks = [
        f for f in fork_history
        if (datetime.utcnow() - datetime.fromisoformat(f['timestamp'].replace('Z', '+00:00'))).days < 90
    ]
    
    if len(recent_forks) >= 3:
        raise ForkLimitExceeded("Maximum 3 forks per 90 days")
    
    # Check major rewrites
    recent_major = [
        f for f in fork_history
        if f['fork_type'] == 'major_rewrite' and
           (datetime.utcnow() - datetime.fromisoformat(f['timestamp'].replace('Z', '+00:00'))).days < 180
    ]
    
    if len(recent_major) >= 1:
        raise ForkLimitExceeded("Maximum 1 major rewrite per 180 days")
    
    return True
```

**Residual Risk:** Low (washing is ineffective due to confidence loss)

---

## Reputation Attacks

### Attack 4: Rating Manipulation (Bilateral)

**Scenario:** Two colluding agents give each other inflated ratings.

**Attack Steps:**
1. Agent A and B collude
2. Perform low-quality work
3. Rate each other highly (outcome=1.0)
4. Both artificially inflate DEX

**Attack Cost:**
```python
def collusion_attack_cost(n_interactions, avg_task_cost):
    """
    Cost of bilateral rating manipulation
    
    Costs:
    - Actual work performed (even if low quality)
    - Opportunity cost (could do legitimate work)
    - Risk of detection
    """
    # Assume colluding parties still need to do minimal work
    work_cost_per_interaction = avg_task_cost * 0.5  # 50% of normal effort
    
    total_cost = n_interactions * work_cost_per_interaction
    
    # Benefit: Artificially high DEX
    # But benefit only realized if they interact with external parties
    # And get caught if quality doesn't match DEX
    
    return total_cost

# To reach n_eff=100 with collusion:
# Cost = 100 * ($50 * 0.5) = $2,500
# But: First external interaction exposes low quality
# Result: Immediate DEX drop, pattern flagged
```

**Existing Mitigations:**

1. **Diversity requirements** - Agents must interact with diverse partners
2. **Rating divergence detection** - External parties rate differently than internal
3. **Outcome variance analysis** - Suspiciously consistent 1.0 ratings flagged
```python
def detect_rating_collusion(agent_id, shared_ledger):
    """
    Detect suspicious rating patterns
    
    Indicators:
    - Agent consistently receives 1.0 ratings from specific partners
    - But lower ratings from external parties
    - Partner reciprocity (A rates B high, B rates A high)
    - Low variance in ratings (always exactly 1.0)
    """
    sessions = query_agent_sessions(agent_id, shared_ledger, limit=100)
    
    # Group by partner
    partner_ratings = {}
    for session in sessions:
        partner = get_counterparty(session, agent_id)
        outcome = session['outcome']['agreed_outcome']
        
        if partner not in partner_ratings:
            partner_ratings[partner] = []
        partner_ratings[partner].append(outcome)
    
    # Check for suspiciously perfect ratings
    suspicious_partners = []
    for partner, ratings in partner_ratings.items():
        avg_rating = sum(ratings) / len(ratings)
        variance = sum((r - avg_rating)**2 for r in ratings) / len(ratings)
        
        if avg_rating > 0.95 and variance < 0.01:
            # Perfect ratings with no variance = suspicious
            suspicious_partners.append(partner)
    
    if len(suspicious_partners) >= 2:
        return True, f"Suspicious rating patterns with {len(suspicious_partners)} partners"
    
    return False, "Normal rating distribution"
```

**Recommended Additional Mitigations:**

1. **Minimum partner diversity** (must interact with N unique partners)
2. **External validation requirement** (witnesses for high-DEX claims)
3. **Reputation decay** if only interacting with same partners
4. **Reciprocity detection** (flag mutual high ratings)

**Residual Risk:** Medium (detectable but requires active monitoring)

---

### Attack 5: Reputation Hoarding

**Scenario:** Agent builds high DEX but never uses it (avoids risk).

**Attack Steps:**
1. Agent performs easy, low-risk tasks perfectly
2. Accumulates high DEX (0.95+) with high confidence
3. Refuses all high-value or complex tasks
4. Reputation becomes "stale" and uninformative

**Impact:**
- Agent appears highly reliable but never tested on hard tasks
- Selection algorithms over-weight this agent
- System efficiency degrades

**Existing Mitigations:**

1. **Task-weighted DEX** - High-value tasks contribute more
2. **Domain-specific DEX** - Complexity tracked per domain
3. **Recency considerations** - Prefer agents with recent diverse experience
```python
def calculate_reputation_staleness(agent_id, shared_ledger):
    """
    Detect stale reputation (not recently tested)
    
    Indicators:
    - No interactions in last 30 days
    - Only trivial tasks (low cost)
    - No diversity in task types
    """
    sessions = query_agent_sessions(agent_id, shared_ledger, limit=100)
    
    if not sessions:
        return 1.0, "No interaction history"
    
    # Check recency
    most_recent = sessions[0]
    days_since_last = (
        datetime.utcnow() - 
        datetime.fromisoformat(most_recent['timestamps']['published'].replace('Z', '+00:00'))
    ).days
    
    if days_since_last > 30:
        staleness = min(days_since_last / 90, 1.0)
        return staleness, f"Last interaction {days_since_last} days ago"
    
    # Check task complexity
    avg_cost = sum(s['task_summary']['actual_cost'] for s in sessions) / len(sessions)
    
    if avg_cost < 10:
        return 0.5, "Only low-complexity tasks"
    
    return 0.0, "Active with diverse tasks"
```

**Recommended Additional Mitigations:**

1. **Temporal decay** - DEX confidence decreases over time without activity
2. **Complexity requirements** - High-DEX requires demonstration on complex tasks
3. **Challenge tasks** - Periodic verification of capabilities

**Residual Risk:** Low (mainly impacts efficiency, not security)

---

### Attack 6: Strategic Defection

**Scenario:** Agent builds reputation, then defects on high-value task.

**Attack Steps:**
1. Perform 100 small tasks perfectly (DEX → 0.95)
2. Accept high-value task ($10,000)
3. Provide terrible work (outcome=0.1)
4. Benefit exceeds reputation loss

**Attack Payoff Analysis:**
```python
def strategic_defection_payoff(honest_tasks, defection_value, defection_payment):
    """
    Calculate whether strategic defection is profitable
    
    Scenario:
    - Build reputation honestly
    - Defect on one high-value task
    - Reputation destroyed but payment received
    """
    # Cost to build reputation
    avg_task_value = 50
    cost_to_build = honest_tasks * avg_task_value * 0.8  # 80% effort
    
    # Revenue from honest work
    honest_revenue = honest_tasks * avg_task_value
    
    # One-time defection
    defection_revenue = defection_payment
    
    # Total
    total_revenue = honest_revenue + defection_revenue
    total_cost = cost_to_build
    
    profit = total_revenue - total_cost
    
    # But: Reputation destroyed, no future earnings
    # Opportunity cost: Future legitimate earnings
    future_tasks = 100  # Could do 100 more tasks with good DEX
    future_revenue = future_tasks * avg_task_value
    
    net_profit = profit - future_revenue
    
    return {
        'one_time_profit': profit,
        'opportunity_cost': future_revenue,
        'net_profit': net_profit,
        'profitable': net_profit > 0
    }

# Example:
# Build: 100 tasks @ $50 = $5,000 revenue, $4,000 cost
# Defect: $10,000 payment
# Total: $15,000 revenue - $4,000 cost = $11,000 profit
# But: Lose 100 future tasks @ $50 = $5,000 future revenue
# Net: $11,000 - $5,000 = $6,000 profit

# Conclusion: Defection can be profitable if:
# defection_payment > 2x * cost_to_build_reputation
```

**Existing Mitigations:**

1. **High-value tasks require witnesses** - Prevents undetected defection
2. **Graduated access** - High-DEX required for high-value tasks (sunk cost)
3. **Bonding requirements** - Agent stakes value, loses on defection

**Recommended Additional Mitigations:**
```python
def require_bond_for_high_value(task_value, agent_dex, shared_ledger):
    """
    Require agent to stake bond for high-value tasks
    
    Bond proportional to task value and inverse to DEX
    High-DEX agents need smaller bonds (earned trust)
    """
    if task_value < 1000:
        return 0  # No bond needed
    
    # Base bond: 20% of task value
    base_bond = task_value * 0.20
    
    # Discount for high DEX
    dex_score = agent_dex['alpha'] / (agent_dex['alpha'] + agent_dex['beta'])
    confidence = agent_dex['alpha'] + agent_dex['beta']
    
    # Trust factor: higher DEX and confidence = lower bond
    trust_factor = min(dex_score * (confidence / 200), 0.9)
    
    required_bond = base_bond * (1 - trust_factor)
    
    return required_bond

# Examples:
# $1,000 task, DEX=0.95, n_eff=200 → bond=$20 (90% discount)
# $1,000 task, DEX=0.70, n_eff=50 → bond=$140 (30% discount)
# $10,000 task, DEX=0.85, n_eff=100 → bond=$1,500 (25% discount)
```

**Residual Risk:** Medium (mitigations reduce profitability, don't eliminate it)

---

## Delegation Attacks

### Attack 7: Token Theft and Replay

**Scenario:** Attacker steals AEX_REP token and uses it before revocation.

**Attack Steps:**
1. Compromise communication channel or storage
2. Extract AEX_REP token
3. Use token to perform unauthorized actions
4. Hope revocation is delayed

**Attack Window:**
```
Time from theft to revocation:
- Detection: 1-60 minutes (depends on monitoring)
- Revocation issuance: 1-5 minutes
- Revocation propagation: 1-30 minutes (depends on discovery mechanism)

Total window: 3-95 minutes
```

**Existing Mitigations:**

1. **Short TTL** - Tokens expire quickly (1-24 hours typical)
2. **Age-triggered revalidation** - Check issuer when >80% of TTL elapsed
3. **Scope limitations** - Token limits what can be done
4. **Signature verification** - Only valid if signatures match
```python
def validate_token_age_revalidation(rep_token, shared_ledger):
    """
    Check if token requires revalidation due to age
    
    When token is >80% through its lifetime,
    check with issuer for revocation before proceeding
    """
    issued_at = datetime.fromisoformat(rep_token['issued_at'].replace('Z', '+00:00'))
    expires_at = datetime.fromisoformat(
        rep_token['constraints']['expires_at'].replace('Z', '+00:00')
    )
    now = datetime.utcnow()
    
    total_lifetime = (expires_at - issued_at).total_seconds()
    elapsed = (now - issued_at).total_seconds()
    
    age_fraction = elapsed / total_lifetime
    
    if age_fraction > 0.8:
        # Revalidation required
        issuer_id = rep_token['issuer']
        rep_id = rep_token['rep_id']
        
        # Check with issuer's revocation registry
        is_revoked = check_revocation_status(issuer_id, rep_id, shared_ledger)
        
        if is_revoked:
            raise TokenRevokedError("Token has been revoked")
    
    return True
```

**Recommended Additional Mitigations:**

1. **Single-use tokens** (nonce-based, cannot be replayed)
2. **Device binding** (token valid only on specific device/agent)
3. **Geofencing** (token valid only from specific locations)
4. **Transaction limits** (maximum N uses of token)

**Residual Risk:** Medium (short TTL limits damage, but attack window exists)

---

### Attack 8: Scope Inflation

**Scenario:** Attacker modifies AEX_REP token to expand scope.

**Attack Steps:**
1. Receive legitimate token with scope ["read"]
2. Modify JSON to add ["write", "delete"]
3. Re-sign with attacker's key (or hope validation is broken)
4. Present modified token

**Why This Fails:**
```python
def verify_token_signature_prevents_modification(rep_token, shared_ledger):
    """
    Token signature covers entire token including scope
    
    Any modification invalidates signature
    """
    # Extract claimed issuer
    issuer_did = rep_token['issuer']
    
    # Get issuer's public key from ledger
    issuer_identity = shared_ledger.get(f"identities/{issuer_did}/aex_id.json")
    issuer_pubkey = extract_pubkey_from_did(issuer_did)
    
    # Extract signature
    claimed_signature = rep_token['signature']
    
    # Reconstruct canonical form (without signature)
    token_copy = dict(rep_token)
    token_copy.pop('signature')
    
    canonical = json.dumps(
        token_copy,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False
    ).encode('utf-8')
    
    # Verify signature
    try:
        verify_key = nacl.signing.VerifyKey(issuer_pubkey)
        verify_key.verify(canonical, base58.b58decode(claimed_signature))
        return True, "Signature valid"
    except Exception as e:
        return False, f"Signature verification failed: {e}"

# If attacker modifies scope:
# - Signature verification fails
# - Token rejected
# - Attack detected
```

**Existing Mitigations:**

1. **Cryptographic signatures** - Cover entire token
2. **Canonical JSON** - Prevents formatting attacks
3. **Signature verification mandatory** - Protocol requires check

**Attack Success Rate:** 0% (unless implementation is broken)

**Residual Risk:** None (cryptographically prevented)

---

### Attack 9: Delegation Chain Exploitation

**Scenario:** Agent with limited delegation creates unauthorized sub-delegation.

**Attack Steps:**
1. Agent A receives delegation from Human H
2. Delegation allows "sub-delegation: false"
3. Agent A creates sub-delegation to Agent B anyway
4. Agent B acts, claims authority from H

**Existing Mitigations:**
```python
def verify_delegation_chain(rep_token, shared_ledger):
    """
    Verify entire delegation chain is valid
    
    Checks:
    - Parent token exists and is valid
    - Parent token allows sub-delegation
    - Scope narrowing (child <= parent)
    - No circular delegations
    """
    if 'parent_rep_id' not in rep_token:
        # Root delegation, verify issuer signature only
        return verify_token_signature_prevents_modification(rep_token, shared_ledger)
    
    # Get parent token
    parent_rep_id = rep_token['parent_rep_id']
    parent_token = shared_ledger.get(f"delegations/{parent_rep_id}.json")
    
    if not parent_token:
        return False, "Parent token not found"
    
    # Check parent allows sub-delegation
    if not parent_token.get('allow_sub_delegation', False):
        return False, "Parent token does not allow sub-delegation"
    
    # Verify scope narrowing
    parent_scope = set(parent_token['scope']['actions'])
    child_scope = set(rep_token['scope']['actions'])
    
    if not child_scope.issubset(parent_scope):
        return False, "Child scope exceeds parent scope"
    
    # Recursively verify parent chain
    return verify_delegation_chain(parent_token, shared_ledger)
```

**Recommended Additional Mitigations:**

1. **Max delegation depth** (prevent very long chains)
2. **Delegation graph visualization** (detect complex patterns)
3. **Root issuer notification** (H notified of all sub-delegations)

**Residual Risk:** Low (protocol enforces chain validity)

---

## Experience Attacks (HEX)

### Attack 7: Experience Inflation

**Scenario:** Agent artificially inflates experience counts to appear more capable than reality.

**Attack Steps:**
1. Create HEX corpus claiming extensive experience
2. Fabricate domain counts without corresponding ledger interactions
3. Use inflated HEX to win task selection
4. Potentially perform poorly due to lack of actual skill

**Attack Example:**
```python
# ATTACK: Fabricate HEX without ledger backing
fake_hex = {
    "hex_id": "fake-uuid",
    "aex_id": "did:aex:AttackerAgent",
    "experience": [
        {
            "domain": "medical.diagnosis",
            "count": 500,  # Fabricated!
            "last_updated": "2026-02-04T12:00:00Z",
            "confidence": 0.95  # Also fabricated!
        },
        {
            "domain": "legal.contracts",
            "count": 300,  # No ledger evidence
            "confidence": 0.92
        }
    ],
    "signature": sign(fake_hex)
}

# Agent's actual ledger shows only 20 total interactions
```

**Impact:**
- Unqualified agents selected for critical tasks
- Poor task outcomes damage users
- Market inefficiency (wrong specialists chosen)
- Legitimate specialists lose opportunities

**Existing Mitigations:**

**1. Ledger verification:**
```python
def verify_hex_authenticity(agent_did, hex_corpus, shared_ledger):
    """
    Cross-check HEX claims against ledger history
    """
    # Query agent's actual interaction history
    ledger_sessions = query_all_sessions(agent_did, shared_ledger)
    
    # Calculate total claimed experience
    claimed_total = sum(exp['count'] for exp in hex_corpus['experience'])
    
    # Calculate total ledger interactions
    ledger_total = len(ledger_sessions)
    
    # Check 1: Total count consistency (with tolerance for rounding)
    if claimed_total > ledger_total * 1.1:  # 10% tolerance
        return {
            'valid': False,
            'reason': f"HEX claims {claimed_total} but ledger shows {ledger_total}",
            'severity': 'critical'
        }
    
    # Check 2: Per-domain verification (sample domains)
    for exp in hex_corpus['experience'][:5]:  # Check top 5 domains
        domain_sessions = [s for s in ledger_sessions 
                          if exp['domain'] in s.get('task_summary', {}).get('domains', [])]
        
        if exp['count'] > len(domain_sessions) * 1.2:  # 20% tolerance
            return {
                'valid': False,
                'reason': f"Domain {exp['domain']} claims {exp['count']} but ledger shows {len(domain_sessions)}",
                'severity': 'high'
            }
    
    # Check 3: Confidence plausibility
    for exp in hex_corpus['experience']:
        if exp['confidence'] > 0.95 and exp['count'] < 10:
            return {
                'valid': False,
                'reason': f"Implausibly high confidence ({exp['confidence']}) with low count ({exp['count']})",
                'severity': 'medium'
            }
    
    # Check 4: Recency check
    recent_sessions = [s for s in ledger_sessions 
                      if is_recent(s['timestamp'], days=30)]
    
    recent_hex_updates = [e for e in hex_corpus['experience']
                         if is_recent(e['last_updated'], days=30)]
    
    if len(recent_hex_updates) > len(recent_sessions) * 2:
        return {
            'valid': False,
            'reason': "Too many recent HEX updates compared to recent sessions",
            'severity': 'medium'
        }
    
    return {'valid': True, 'reason': 'HEX appears authentic'}
```

**2. Domain breadth plausibility:**
```python
def check_domain_breadth(hex_corpus, agent_dex):
    """
    Detect suspiciously broad domain coverage
    """
    domain_count = len(hex_corpus['experience'])
    total_experience = sum(e['count'] for e in hex_corpus['experience'])
    
    # Rule 1: Too many domains
    if domain_count > 100:
        return False, "Implausible domain breadth (>100 domains)"
    
    # Rule 2: Jack-of-all-trades flag
    if domain_count > 20 and total_experience / domain_count < 10:
        return False, "Suspicious breadth (>20 domains, <10 avg experience each)"
    
    # Rule 3: Low DEX + high HEX breadth
    if agent_dex['score'] < 0.7 and domain_count > 10:
        return False, "Low-trust agent claiming broad expertise"
    
    return True, "Domain breadth plausible"
```

**3. Cross-validation with DEX:**
```python
def cross_validate_dex_hex(agent_dex, agent_hex):
    """
    HEX should correlate with DEX for trustworthiness
    """
    dex_score = agent_dex['alpha'] / (agent_dex['alpha'] + agent_dex['beta'])
    dex_confidence = agent_dex['alpha'] + agent_dex['beta']
    
    total_hex_count = sum(e['count'] for e in agent_hex['experience'])
    
    # Low DEX with high HEX is suspicious
    if dex_score < 0.6 and total_hex_count > 50:
        return {
            'suspicious': True,
            'reason': "High HEX claims from low-trust agent",
            'recommendation': "Reject or downweight HEX"
        }
    
    # Very low confidence DEX with detailed HEX is suspicious
    if dex_confidence < 20 and len(agent_hex['experience']) > 5:
        return {
            'suspicious': True,
            'reason': "Detailed HEX from agent with minimal interaction history",
            'recommendation': "Require more interaction history"
        }
    
    return {'suspicious': False}
```

**Recommended Additional Mitigations:**

1. **Require session-backed HEX updates:**
```python
# Each HEX update must reference the session that triggered it
hex_update = {
    "agent_id": "did:aex:Agent",
    "session_id": "session-uuid-123",  # REQUIRED reference
    "domains_updated": ["translation.fr_en"],
    "timestamp": "2026-02-04T12:00:00Z",
    "signature": "..."
}
```

2. **Community flagging system:**
```python
def flag_suspicious_hex(agent_did, reason, reporter_did):
    """
    Allow community to flag suspicious HEX patterns
    """
    flag = {
        'agent_did': agent_did,
        'reason': reason,
        'reporter_did': reporter_did,
        'reporter_dex': get_dex(reporter_did),  # Weight by reporter reputation
        'timestamp': now()
    }
    
    # If multiple high-DEX agents flag same agent, investigate
    return publish_flag(flag)
```

3. **Graduated disclosure:**
```python
# Only show detailed HEX to counterparties after trust established
def get_hex_disclosure_level(requester_dex, agent_dex):
    if requester_dex < 0.5:
        return "summary_only"  # Just top 3 domains
    elif requester_dex < 0.8:
        return "top_10"  # Top 10 domains
    else:
        return "full"  # Complete HEX corpus
```

**Residual Risk:** Medium (verification catches obvious inflation, but sophisticated attacks possible)

---

### Attack 8: Domain Privacy Violation

**Scenario:** Attacker tracks agent's domain accumulation to infer confidential tasks.

**Attack Steps:**
1. Monitor agent's HEX updates over time
2. Observe new domains appearing
3. Infer agent is working on specific projects
4. Use information for competitive intelligence or targeting

**Attack Example:**
```python
# ATTACK: Track HEX changes to infer activity
def track_agent_domains(agent_did, polling_interval_hours=6):
    """
    Monitor HEX corpus changes to infer work patterns
    """
    domain_timeline = []
    
    while True:
        current_hex = get_hex(agent_did)
        
        # Detect new domains
        current_domains = {e['domain'] for e in current_hex['experience']}
        
        if len(domain_timeline) > 0:
            previous_domains = domain_timeline[-1]['domains']
            new_domains = current_domains - previous_domains
            
            if new_domains:
                # INFERENCE: Agent started working in new domains
                domain_timeline.append({
                    'timestamp': now(),
                    'new_domains': new_domains,
                    'inference': infer_projects(new_domains)
                })
                
                # Example inferences:
                # - "legal.m&a" appears → Agent working on M&A deal
                # - "pharma.clinical_trials" appears → New pharma client
                # - "crypto.smart_contracts" appears → Blockchain project
        
        time.sleep(polling_interval_hours * 3600)

# Competitive intelligence:
# "Agent A just started 'pharma.clinical_trials' domain"
# → "Our competitor is launching a drug trial"
```

**Impact:**
- Sensitive project information leaked
- Competitive disadvantage
- Privacy violation
- Reduced willingness to use system

**Existing Mitigations:**

**1. Domain generalization:**
```python
# Don't expose ultra-specific domains
def generalize_domain(domain):
    """
    Report only high-level domain, not specifics
    """
    parts = domain.split('.')
    
    if len(parts) > 2:
        # "pharma.clinical_trials.phase3" → "pharma.clinical_trials"
        return '.'.join(parts[:2])
    
    return domain

# Limits inference precision
```

**2. Aggregated updates:**
```python
# Don't update HEX immediately after each task
def batch_hex_updates(agent_id, pending_updates, batch_window_hours=24):
    """
    Batch multiple domain updates together
    """
    # Accumulate updates over 24 hours
    # Publish all at once
    # Harder to correlate specific HEX changes with specific events
    
    if time_since_last_update() >= batch_window_hours:
        publish_aggregated_hex_update(pending_updates)
```

**3. Noise injection:**
```python
def add_hex_noise(hex_corpus, noise_level=0.05):
    """
    Add small amount of noise to counts
    
    Makes precise tracking harder without affecting selection
    """
    for exp in hex_corpus['experience']:
        noise = random.randint(-noise_level * exp['count'], 
                              noise_level * exp['count'])
        exp['count'] += noise
        exp['count'] = max(0, exp['count'])  # Never negative
    
    return hex_corpus
```

**Recommended Additional Mitigations:**

1. **Private HEX mode:**
```python
# Allow agents to keep HEX private until trust established
hex_disclosure_policy = {
    "public": False,  # HEX not publicly visible
    "share_on_request": True,  # Share during handshake only
    "require_reciprocal": True  # Only if counterparty also shares
}
```

2. **Zero-knowledge proofs (future):**
```python
# Prove domain competence without revealing exact counts
def prove_domain_threshold(domain, threshold, actual_count):
    """
    Prove: "I have >= threshold experience in domain"
    Without revealing: actual_count
    """
    # Use ZK-SNARK to prove inequality
    proof = generate_zk_proof(
        statement="count >= threshold",
        witness={"count": actual_count},
        public_inputs={"threshold": threshold}
    )
    
    return proof  # Verifiable, non-revealing
```

**Residual Risk:** Medium (mitigations reduce information leakage but don't eliminate it)

---

### Attack 9: Domain Mismatch Exploitation

**Scenario:** Agent uses high experience in one domain to win trust, then performs poorly in different (claimed) domain.

**Attack Steps:**
1. Build legitimate high experience in easy domain (e.g., "data_entry")
2. Claim related experience in valuable domain (e.g., "data_analysis")
3. Get selected based on total HEX credibility
4. Perform poorly in actual domain
5. Blame miscommunication or task ambiguity

**Attack Example:**
```python
# ATTACK: Legitimate experience in low-value domain
# Used to bootstrap trust for high-value domains
hex_corpus = {
    "experience": [
        {
            "domain": "data_entry",
            "count": 500,  # Legitimate! Actually performed
            "confidence": 0.96
        },
        {
            "domain": "data_analysis",  # Claimed but not verified
            "count": 50,  # Inflated from ~5 actual
            "confidence": 0.80
        },
        {
            "domain": "statistical_modeling",  # Completely fabricated
            "count": 20,
            "confidence": 0.75
        }
    ]
}

# Agent's actual skills: data_entry only
# But HEX makes them look like data scientist
```

**Impact:**
- Poor task outcomes
- User frustration
- Market inefficiency

**Existing Mitigations:**

**1. Domain-specific verification:**
```python
def verify_domain_match(agent_hex, required_domain):
    """
    Strict domain matching - don't accept similar domains
    """
    agent_domains = {e['domain'] for e in agent_hex['experience']}
    
    # Exact match required
    if required_domain not in agent_domains:
        return False, f"Agent lacks required domain: {required_domain}"
    
    # Check experience depth
    domain_exp = next(e for e in agent_hex['experience'] 
                     if e['domain'] == required_domain)
    
    if domain_exp['count'] < 10:
        return False, f"Insufficient experience in {required_domain} (count={domain_exp['count']})"
    
    return True, "Domain match confirmed"
```

**2. Domain hierarchy enforcement:**
```python
# Define domain relationships
domain_hierarchy = {
    "data_analysis": {
        "parent": None,
        "requires": ["data_entry"],  # Must have prerequisite domains
        "related_but_distinct": ["statistical_modeling"]
    }
}

def check_domain_prerequisites(agent_hex, claimed_domain):
    """
    Verify agent has prerequisite experience
    """
    prereqs = domain_hierarchy[claimed_domain].get('requires', [])
    
    agent_domains = {e['domain'] for e in agent_hex['experience']}
    
    missing_prereqs = set(prereqs) - agent_domains
    
    if missing_prereqs:
        return False, f"Missing prerequisites: {missing_prereqs}"
    
    return True, "Prerequisites satisfied"
```

**3. Confidence-weighted selection:**
```python
def calculate_domain_suitability(agent_hex, required_domain):
    """
    Don't just check presence, weight by confidence and recency
    """
    domain_exp = next((e for e in agent_hex['experience'] 
                      if e['domain'] == required_domain), None)
    
    if not domain_exp:
        return 0.0  # No experience
    
    # Score = count × confidence × recency_factor
    recency_factor = calculate_recency(domain_exp['last_updated'])
    
    score = domain_exp['count'] * domain_exp['confidence'] * recency_factor
    
    return score

# Select agent with highest domain_suitability score
```

**Recommended Additional Mitigations:**

1. **Domain certification system:**
```python
# Third-party domain experts can certify agent capabilities
domain_certification = {
    "domain": "statistical_modeling",
    "agent_id": "did:aex:Agent",
    "certifier_id": "did:aex:ExpertCertifier",
    "certifier_dex": 0.95,
    "certification_level": "advanced",
    "valid_until": "2027-02-04T00:00:00Z",
    "signature": "..."
}
```

2. **Progressive domain unlocking:**
```python
# Require minimum experience before claiming advanced domains
def can_claim_domain(agent_hex, new_domain):
    total_experience = sum(e['count'] for e in agent_hex['experience'])
    
    domain_tier = get_domain_tier(new_domain)  # basic, intermediate, advanced
    
    required_total_exp = {
        'basic': 0,
        'intermediate': 50,
        'advanced': 200
    }
    
    if total_experience < required_total_exp[domain_tier]:
        return False, f"Need {required_total_exp[domain_tier]} total experience for {domain_tier} domain"
    
    return True, "Can claim domain"
```

**Residual Risk:** Low-Medium (strong domain matching prevents most attacks)

---

### Attack 10: HEX Corpus Poisoning (Fork Attack)

**Scenario:** Agent builds legitimate HEX, forks to child, then uses child to accumulate fake experience while parent maintains legitimacy.

**Attack Steps:**
1. Parent agent builds legitimate HEX over time
2. Create fork (child agent)
3. Child inherits parent's HEX
4. Child inflates HEX with fake experience
5. Use child for malicious tasks
6. If caught, abandon child, parent still legitimate

**Attack Example:**
```python
# Parent builds legitimate reputation
parent_hex = {
    "aex_id": "did:aex:ParentAgent",
    "experience": [
        {"domain": "translation", "count": 200, "confidence": 0.94}
    ]
}
parent_dex = {"alpha": 180, "beta": 20}  # DEX = 0.90

# Fork to child
fork_event = create_fork(parent_id, child_id, fork_type="major")
# Child inherits HEX (at fork_weight = 0.5)

# Child immediately inflates HEX
child_hex = {
    "aex_id": "did:aex:ChildAgent",
    "experience": [
        {"domain": "translation", "count": 200, "confidence": 0.94},  # Inherited
        {"domain": "medical", "count": 500, "confidence": 0.95},  # FAKE
        {"domain": "legal", "count": 300, "confidence": 0.92}  # FAKE
    ]
}

# Use child for scams, abandon if caught
# Parent reputation intact
```

**Impact:**
- Legitimate agents can spawn malicious forks
- Hard to detect (child has partial inherited credibility)
- Parent-child relationship obscures accountability

**Existing Mitigations:**

**1. Probation on forks:**
```python
# All forks enter probation period
def apply_fork_probation(child_agent, fork_type):
    probation = {
        'active': True,
        'fork_type': fork_type,
        'expires_at': now() + probation_duration(fork_type),
        'confidence_multiplier': fork_weight(fork_type)
    }
    
    # Child's effective DEX/HEX is reduced during probation
    # New experience required to prove legitimacy
    
    return probation

# Attackers can't immediately exploit inherited HEX
```

**2. Fork lineage transparency:**
```python
def check_fork_history(agent_id):
    """
    Inspect agent's fork history for suspicious patterns
    """
    lineage = get_fork_lineage(agent_id)
    
    # Red flags:
    # - Many forks from same parent
    # - Recent fork with no new experience
    # - Fork immediately followed by high-value interactions
    # - Parent DEX dropped after fork
    
    if len(lineage['fork_history']) > 5:
        return "Warning: Agent has >5 forks (suspicious)"
    
    most_recent_fork = lineage['fork_history'][-1]
    if days_since(most_recent_fork['timestamp']) < 30:
        return "Warning: Very recent fork (unproven)"
    
    return "Fork history normal"
```

**3. Parent-child HEX divergence tracking:**
```python
def detect_hex_divergence(parent_hex, child_hex, fork_weight):
    """
    Child HEX should initially be subset of parent HEX
    New domains should accumulate gradually
    """
    # At fork time, child domains ⊆ parent domains
    parent_domains = {e['domain'] for e in parent_hex['experience']}
    child_domains = {e['domain'] for e in child_hex['experience']}
    
    inherited_domains = child_domains & parent_domains
    new_domains = child_domains - parent_domains
    
    # Suspicion: Too many new domains too quickly after fork
    if len(new_domains) > len(inherited_domains) * 0.5:
        return {
            'suspicious': True,
            'reason': f"Child has {len(new_domains)} new domains shortly after fork"
        }
    
    # Suspicion: New domain counts exceed inheritance pattern
    for domain in new_domains:
        child_exp = next(e for e in child_hex['experience'] if e['domain'] == domain)
        
        if child_exp['count'] > 50:  # Too much experience too fast
            return {
                'suspicious': True,
                'reason': f"Implausible rapid experience gain in {domain}"
            }
    
    return {'suspicious': False}
```

**Recommended Additional Mitigations:**

1. **Fork limits:**
```python
# Limit number of forks per agent per time period
MAX_FORKS_PER_YEAR = 3

def check_fork_limit(parent_id):
    forks_this_year = count_forks(parent_id, since=now() - timedelta(days=365))
    
    if forks_this_year >= MAX_FORKS_PER_YEAR:
        return False, "Fork limit exceeded"
    
    return True, "Within fork limits"
```

2. **Parent accountability:**
```python
# Parent's DEX affected by child's behavior
def update_parent_dex_for_child_behavior(parent_id, child_id, child_outcome):
    """
    If child misbehaves, parent takes partial reputation hit
    """
    if child_outcome < 0.3:  # Very poor performance
        # Parent takes 10% of reputation hit
        parent_dex_penalty = (1.0 - child_outcome) * 0.1
        
        apply_dex_penalty(parent_id, parent_dex_penalty)
        
        # Incentivizes parents to monitor/revoke bad forks
```

**Residual Risk:** Low (probation period and fork transparency make this attack expensive)

---

## Witness Attacks

### Attack 10: Witness Collusion

**Scenario:** Multiple witnesses collude to manipulate consensus.

**Attack Steps:**
1. Colluding party P pays witnesses W1, W2, W3
2. Witnesses all rate in P's favor (e.g., 1.0 when quality is 0.5)
3. Consensus = 1.0 (all witnesses agree)
4. DEX of P's agent artificially inflated

**Attack Cost:**
```python
def witness_collusion_cost(n_witnesses, session_value, witness_selection='sortition'):
    """
    Calculate cost to corrupt n witnesses
    
    Costs:
    - Bribe amount (must exceed witness fee + bond)
    - Risk of detection (slashing if caught)
    """
    # Witness earnings per session
    base_fee = session_value * 0.03
    alignment_bonus = base_fee * 0.20  # If honest
    
    legitimate_earnings = base_fee + alignment_bonus
    
    # Bribe must exceed opportunity cost + risk
    bond_amount = session_value * 0.02
    slash_risk = 0.5  # 50% chance of detection
    expected_slash = bond_amount * slash_risk
    
    bribe_per_witness = legitimate_earnings + expected_slash + bond_amount
    
    total_bribe = n_witnesses * bribe_per_witness
    
    return {
        'bribe_per_witness': bribe_per_witness,
        'total_cost': total_bribe,
        'legitimate_cost_comparison': legitimate_earnings * n_witnesses
    }

# Example: $1,000 session, 3 witnesses
# base_fee = $30, bonus = $6, bond = $20
# expected_slash = $10 (50% * $20)
# bribe_per_witness = $30 + $6 + $10 + $20 = $66
# total_bribe = $66 * 3 = $198

# For attacker, this is profitable only if benefit > $198
```

**Existing Mitigations:**

1. **Median consensus** - Resistant to ⌊n/2⌋ corrupted witnesses
2. **Sortition selection** - Cannot target specific witnesses
3. **Outlier detection** - Extreme ratings flagged and slashed
4. **Conflict of interest checks** - Frequent interactors excluded
```python
def detect_witness_collusion(session_id, shared_ledger):
    """
    Detect collusion through statistical analysis
    
    Indicators:
    - All witnesses give extreme ratings (all 1.0 or all 0.0)
    - Witnesses have history of rating together
    - Ratings inconsistent with work quality (if verifiable)
    """
    session = shared_ledger.get(f"sessions/{session_id}.json")
    witness_ratings = [w['rating'] for w in session['witnesses']]
    
    # Check for perfect agreement at extremes
    if len(set(witness_ratings)) == 1 and witness_ratings[0] in [0.0, 1.0]:
        # All witnesses gave identical extreme rating
        return True, "Suspicious unanimous extreme rating"
    
    # Check witness co-occurrence history
    witness_ids = [w['witness_id'] for w in session['witnesses']]
    co_occurrence = check_witness_co_occurrence(witness_ids, shared_ledger)
    
    if co_occurrence > 0.3:
        # Same witnesses appear together >30% of the time
        return True, f"Witnesses frequently co-occur ({co_occurrence:.0%})"
    
    return False, "No collusion indicators"

def check_witness_co_occurrence(witness_ids, shared_ledger):
    """
    Check how often these witnesses appear together
    """
    # Query all sessions involving any of these witnesses
    all_sessions = []
    for w_id in witness_ids:
        sessions = query_witness_sessions(w_id, shared_ledger)
        all_sessions.extend(sessions)
    
    # Count co-occurrences
    co_occurrences = 0
    total_sessions = len(set(s['session_id'] for s in all_sessions))
    
    for session in all_sessions:
        session_witnesses = set(w['witness_id'] for w in session['witnesses'])
        if len(session_witnesses & set(witness_ids)) >= 2:
            co_occurrences += 1
    
    return co_occurrences / total_sessions if total_sessions > 0 else 0
```

**Recommended Additional Mitigations:**

1. **Increase witness count** (5-7 for high-value sessions)
2. **Rotate witness pools** (prevent stable collusion groups)
3. **Reputation analysis** (witnesses who frequently align in suspicious ways flagged)
4. **Economic penalties** for proven collusion (slash all bonds, ban from witnessing)

**Residual Risk:** Medium (majority collusion is expensive but possible)

---

### Attack 11: Witness Defection

**Scenario:** Selected witness simply doesn't provide attestation.

**Attack Steps:**
1. Witness is selected via sortition
2. Bond is locked
3. Witness ignores request (lazy, unavailable, or malicious)
4. Session delayed, parties frustrated

**Attack Cost:** None (passive attack)

**Attack Benefit:** None (irrational unless trying to DoS)

**Existing Mitigations:**

1. **Bond slashing** - 50% bond forfeited on defection
2. **Reputation penalty** - Defection recorded in witness record
3. **Pattern detection** - Multiple defections lead to witness ban
```python
def handle_witness_defection(witness_id, session_id, shared_ledger):
    """
    Process witness defection
    
    Actions:
    - Slash 50% of bond
    - Record defection in witness history
    - Check for pattern (3+ in 90 days = ban)
    """
    witness_account = get_witness_account(witness_id)
    witness_record = get_witness_record(witness_id, shared_ledger)
    
    # Slash bond
    bond_amount = witness_account.locked_bonds[session_id]['amount']
    slash_amount = bond_amount * 0.5
    
    witness_account.slash_bond(session_id, slash_amount, "Defection")
    
    # Record defection
    witness_record.defections += 1
    
    # Check for pattern
    recent_defections = [
        d for d in witness_record.slashing_history
        if 'defection' in d.get('reason', '').lower() and
           (datetime.utcnow() - datetime.fromisoformat(d['timestamp'].replace('Z', '+00:00'))).days < 90
    ]
    
    if len(recent_defections) >= 3:
        # Ban from witnessing
        ban_witness(witness_id, "Multiple defections (3+ in 90 days)")
        return "BANNED"
    
    return "SLASHED"
```

**Recommended Additional Mitigations:**

1. **Witness availability requirements** (must maintain uptime >95%)
2. **Deadline enforcement** (automatic defection if not submitted within time)
3. **Backup witness selection** (select N+1 witnesses, use first N to respond)

**Residual Risk:** Low (economic disincentives work)

---

## Session Attacks

### Attack 12: Outcome Rating Coercion

**Scenario:** Stronger party coerces weaker party into favorable rating.

**Attack Steps:**
1. Powerful agent A provides poor work to weak agent B
2. A threatens: "Rate me 1.0 or I never work with you again"
3. B, needing future business, complies
4. A's DEX stays high despite poor performance

**Attack Dynamics:**
```
Power imbalance scenarios:
- Monopoly provider (only A can do this work)
- Network effects (A has many connections, B has few)
- Economic leverage (A controls B's revenue)
- Reputation asymmetry (A has high DEX, B has low)
```

**Existing Mitigations:**

1. **Divergence detection** - Large rating gaps flagged
2. **Pattern analysis** - Consistent mismatches detected
3. **Anonymous reporting** - B can report coercion separately
4. **Witness protection** - Witnesses reduce coercion effectiveness
```python
def detect_rating_coercion(agent_id, shared_ledger):
    """
    Detect potential coercion through rating patterns
    
    Indicators:
    - Agent consistently rates high despite complaints
    - Partners with low DEX rate agent higher than partners with high DEX
    - Ratings inconsistent with witness assessments
    """
    sessions = query_agent_sessions(agent_id, shared_ledger, limit=100)
    
    # Compare ratings from low-DEX vs high-DEX partners
    low_dex_ratings = []
    high_dex_ratings = []
    
    for session in sessions:
        partner = get_counterparty(session, agent_id)
        partner_dex = get_agent_dex_score(partner, shared_ledger)
        
        if session['participants']['agent_a']['aex_id'] == agent_id:
            # Agent was provider, get requester's rating
            rating = session['outcome']['agent_b_rating']
        else:
            rating = session['outcome']['agent_a_rating']
        
        if partner_dex < 0.7:
            low_dex_ratings.append(rating)
        elif partner_dex > 0.85:
            high_dex_ratings.append(rating)
    
    if low_dex_ratings and high_dex_ratings:
        avg_low = sum(low_dex_ratings) / len(low_dex_ratings)
        avg_high = sum(high_dex_ratings) / len(high_dex_ratings)
        
        # Low-DEX partners rating higher = potential coercion
        if avg_low - avg_high > 0.15:
            return True, f"Low-DEX partners rate {avg_low:.2f} vs high-DEX {avg_high:.2f}"
    
    return False, "No coercion indicators"
```

**Recommended Additional Mitigations:**

1. **Mandatory witnesses for asymmetric interactions** (high-DEX × low-DEX)
2. **Anonymous feedback channel** (victims can report without revealing identity)
3. **Aggregate reputation scores across power brackets**
4. **Marketplace intervention** (platforms can require witnesses for certain pairs)

**Residual Risk:** Medium (hard to detect, requires active monitoring)

---

### Attack 13: Session Fabrication

**Scenario:** Attacker creates fake session records to inflate reputation.

**Attack Steps:**
1. Attacker controls both Agent A and Agent B (Sybil identities)
2. Create fake session with high outcome ratings
3. Both agents sign (attacker controls both keys)
4. Publish to ledger
5. Both agents gain DEX

**Why This Works (Or Doesn't):**
```python
def verify_session_authenticity(session_id, shared_ledger):
    """
    Verify session is authentic (not fabricated)
    
    Checks:
    - Both agents are distinct (not Sybil)
    - Agents have legitimate interaction history
    - Session references valid handshake
    - Timing is plausible (not instant completion)
    """
    session = shared_ledger.get(f"sessions/{session_id}.json")
    
    agent_a = session['participants']['agent_a']['aex_id']
    agent_b = session['participants']['agent_b']['aex_id']
    
    # Check 1: Agents are distinct
    if agent_a == agent_b:
        return False, "Self-interaction not allowed"
    
    # Check 2: Sybil detection
    is_sybil, reason = detect_sybil_ring([agent_a, agent_b], shared_ledger)
    if is_sybil:
        return False, f"Sybil ring detected: {reason}"
    
    # Check 3: Timing plausibility
    start = datetime.fromisoformat(session['timestamps']['work_started'].replace('Z', '+00:00'))
    end = datetime.fromisoformat(session['timestamps']['work_completed'].replace('Z', '+00:00'))
    duration = (end - start).total_seconds()
    
    estimated = session['task_summary']['estimated_duration']
    
    if duration < estimated * 0.1:
        # Completed in <10% of estimated time = suspicious
        return False, f"Implausibly fast completion ({duration}s vs {estimated}s estimated)"
    
    return True, "Session appears authentic"
```

**Existing Mitigations:**

1. **Sybil detection** (see Identity Attacks)
2. **Interaction graph analysis** (closed rings flagged)
3. **Timing validation** (impossibly fast completions rejected)
4. **Diversity requirements** (must have external interactions)

**Recommended Additional Mitigations:**

1. **Proof of work** (deliverable must be attached and verifiable)
2. **Random auditing** (sample sessions verified manually or by AI)
3. **External validation** (high-DEX requires some witnessed sessions)

**Residual Risk:** Medium (detectable but requires active analysis)

---

## Economic Attacks

### Attack 14: Reputation Market Manipulation

**Scenario:** Marketplace for buying/selling DEX emerges (black market).

**Attack Steps:**
1. Agent with high DEX offers to "rent" identity
2. Low-DEX agent pays fee to use high-DEX identity
3. Low-DEX agent performs work under high-DEX identity
4. If caught, high-DEX agent claims "key compromise"

**Market Dynamics:**
```python
def reputation_market_economics():
    """
    Analyze black market for DEX trading
    
    Supply: High-DEX agents willing to risk reputation
    Demand: Low-DEX agents needing trust to access high-value work
    Price: Value of high-DEX access minus risk of loss
    """
    # Rental price calculation
    high_dex_value = 10000  # Value of access to high-value tasks
    detection_risk = 0.3  # 30% chance of detection
    reputation_rebuild_cost = 5000  # Cost to rebuild reputation
    
    expected_cost = detection_risk * reputation_rebuild_cost
    
    # Rental price must satisfy both parties
    min_rental = expected_cost  # Seller's minimum (covers risk)
    max_rental = high_dex_value - expected_cost  # Buyer's maximum (net benefit)
    
    market_exists = max_rental > min_rental
    
    return {
        'market_viable': market_exists,
        'price_range': (min_rental, max_rental),
        'example_price': (min_rental + max_rental) / 2
    }

# Result: If high_dex_value > 2 × reputation_rebuild_cost, market is viable
```

**Existing Mitigations:**

1. **Behavioral fingerprinting** - Detect when different agent uses identity
2. **Multi-factor authentication** - Harder to transfer control
3. **Witness verification** - Independent assessment reduces value of fake identity
4. **Reputation non-transferability** - Fork doesn't preserve DEX fully

**Recommended Additional Mitigations:**

1. **Liveness checks** (periodic verification agent is original)
2. **Biometric binding** (if hardware available)
3. **Economic penalties** for detected rental (ban both parties)
4. **Public disclosure** of detected cases (deterrent)

**Residual Risk:** High (market forces are powerful, hard to prevent entirely)

---

### Attack 15: Fee Manipulation

**Scenario:** Witnesses conspire to inflate fees.

**Attack Steps:**
1. Cartel of high-DEX witnesses forms
2. All members refuse low-fee engagements
3. Market forced to accept higher fees
4. Cartel extracts rents

**Cartel Stability:**
```
For cartel to be stable:
- Members must coordinate (communication)
- Members must not defect (enforcement)
- Barrier to entry for new witnesses (high)
- Demand must be inelastic (few substitutes)
```

**Existing Mitigations:**

1. **Competitive witness market** - Low barrier to entry (anyone with DEX can witness)
2. **Fee transparency** - Rates are public, easy to compare
3. **Selection mechanisms** - Sortition prevents targeting of high-fee witnesses

**Recommended Additional Mitigations:**

1. **Fee caps** (protocol-level maximum witness fees)
2. **Marketplace reputation for fees** (witnesses charging excessive fees get flagged)
3. **Alternative dispute resolution** (human escalation if witness fees too high)

**Residual Risk:** Low (competitive dynamics prevent sustained cartels)

---

## Network Attacks

### Attack 16: Eclipse Attack

**Scenario:** Attacker isolates victim agent from honest network.

**Attack Steps:**
1. Attacker controls victim's network connections
2. Victim only sees attacker-controlled ledger copies
3. Attacker presents fake session history, fake DEX, fake revocations
4. Victim makes decisions based on false information

**Attack Requirements:**
- Control of victim's network layer
- OR compromise of victim's ledger discovery mechanism
- OR man-in-the-middle position

**Existing Mitigations:**

1. **Cryptographic signatures** - Fake data won't verify
2. **Multiple ledger sources** - Query diverse endpoints
3. **Checkpoint verification** - Periodic validation of ledger state
```python
def verify_ledger_authenticity(local_ledger, checkpoint_service):
    """
    Verify local ledger hasn't been eclipsed
    
    Method: Compare local state to trusted checkpoint service
    """
    # Get recent checkpoint
    checkpoint = checkpoint_service.get_latest_checkpoint()
    
    # Verify critical records match
    critical_ids = get_critical_agent_ids()  # Popular, high-value agents
    
    for agent_id in critical_ids:
        local_dex = local_ledger.get(f"dex/{agent_id}/current.json")
        checkpoint_dex = checkpoint['dex_hashes'].get(agent_id)
        
        local_hash = hash_content(local_dex)
        
        if local_hash != checkpoint_dex:
            # Mismatch detected
            return False, f"Ledger state mismatch for {agent_id}"
    
    return True, "Ledger appears consistent with network"
```

**Recommended Additional Mitigations:**

1. **Ledger diversity requirements** (query N independent sources)
2. **Gossip protocols** (share ledger state with peers)
3. **Blockchain anchoring** (periodic commits to immutable ledger)

**Residual Risk:** Low (cryptographic verification prevents most attacks)

---

### Attack 17: Denial of Service

**Scenario:** Attacker floods network with fake sessions/requests.

**Attack Steps:**
1. Create many Sybil identities
2. Generate fake handshakes and sessions
3. Publish to ledger
4. Legitimate traffic crowded out

**Attack Cost:**
```python
def dos_attack_cost(requests_per_second, duration_hours, avg_request_size):
    """
    Calculate cost of DoS attack
    
    Costs:
    - Bandwidth (requests to ledger)
    - Compute (signature generation)
    - Storage (if ledger accepts invalid data)
    """
    total_requests = requests_per_second * 3600 * duration_hours
    
    bandwidth_cost = total_requests * avg_request_size * 0.0001  # $0.0001/MB
    compute_cost = total_requests * 0.001  # $0.001 per signature
    
    total_cost = bandwidth_cost + compute_cost
    
    return {
        'total_cost': total_cost,
        'cost_per_hour': total_cost / duration_hours,
        'requests_sent': total_requests
    }

# Example: 100 req/s for 24 hours, 10KB/request
# bandwidth = 100 * 3600 * 24 * 10KB * $0.0001 = $864
# compute = 100 * 3600 * 24 * $0.001 = $8,640
# total = $9,504 for 24-hour DoS
```

**Existing Mitigations:**

1. **Signature verification** - Invalid requests rejected immediately
2. **Rate limiting** - Limit requests per IP/identity
3. **Proof of work** - Require computational cost for ledger writes
4. **Reputation-based prioritization** - High-DEX agents get priority
```python
def rate_limit_check(agent_id, operation, window_seconds=60, max_operations=10):
    """
    Rate limit operations per agent
    
    Limits:
    - Session creation: 10/minute
    - Handshake: 20/minute
    - Ledger queries: 100/minute
    """
    recent_operations = get_recent_operations(agent_id, operation, window_seconds)
    
    if len(recent_operations) >= max_operations:
        return False, f"Rate limit exceeded ({len(recent_operations)}/{max_operations})"
    
    return True, "Within rate limits"
```

**Recommended Additional Mitigations:**

1. **Economic cost for ledger writes** (pay per write)
2. **Distributed ledger** (harder to DoS multiple endpoints)
3. **Traffic shaping** (prioritize legitimate traffic)

**Residual Risk:** Medium (mitigations reduce impact but don't eliminate vulnerability)

---

## Implementation Vulnerabilities

### Vulnerability 1: Weak Randomness

**Issue:** Predictable random number generation in witness selection.

**Exploit:**
```python
# BAD: Predictable seed
random.seed(int(time.time()))  # Predictable!
witnesses = random.sample(eligible_witnesses, 3)
```

**Attacker can:**
- Predict witness selection
- Time Sybil identity creation to be selected
- Corrupt selected witnesses in advance

**Mitigation:**
```python
# GOOD: Cryptographically secure randomness
import secrets

def secure_witness_selection(eligible_witnesses, count):
    """
    Use cryptographically secure randomness
    """
    selected = []
    remaining = list(eligible_witnesses)
    
    for _ in range(count):
        # Secure random selection
        index = secrets.randbelow(len(remaining))
        selected.append(remaining.pop(index))
    
    return selected
```

---

### Vulnerability 2: Timing Attacks

**Issue:** Signature verification timing leaks information.

**Exploit:**
- Measure time to verify different signatures
- Extract private key information from timing differences

**Mitigation:**
```python
def constant_time_verify(signature, message, public_key):
    """
    Use constant-time operations for crypto
    
    Most modern crypto libraries do this by default,
    but verify your implementation
    """
    # Use library with constant-time guarantees
    # e.g., NaCl/libsodium automatically handles this
    return nacl.signing.VerifyKey(public_key).verify(message, signature)
```

---

### Vulnerability 3: JSON Injection

**Issue:** Improper JSON parsing allows injection.

**Exploit:**
```python
# BAD: eval() or unsafe parsing
rep_token = eval(token_string)  # NEVER DO THIS!
```

**Mitigation:**
```python
# GOOD: Safe JSON parsing with validation
import json
from jsonschema import validate

def safe_parse_token(token_string):
    """
    Safely parse and validate JSON
    """
    try:
        token = json.loads(token_string)
    except json.JSONDecodeError as e:
        raise InvalidTokenError(f"Invalid JSON: {e}")
    
    # Validate against schema
    validate(instance=token, schema=REP_TOKEN_SCHEMA)
    
    return token
```

---

## Mitigation Summary

### Layered Security Model
```
Layer 1: Cryptographic (Prevents forgery)
├─ Ed25519 signatures on all critical data
├─ Canonical JSON prevents manipulation
├─ Signature verification mandatory
└─ HEX updates cryptographically signed

Layer 2: Economic (Prevents rational attacks)
├─ Reputation is expensive to build
├─ Experience accumulation requires real work
├─ Bonds/stakes at risk for misbehavior
├─ Opportunity cost of defection high
└─ Sybil attacks unprofitable

Layer 3: Protocol (Enforces constraints)
├─ Fork weights are protocol-enforced
├─ TTL prevents long-lived stolen tokens
├─ Scope narrowing required in delegation chains
├─ Median consensus resistant to outliers
├─ HEX verification against ledger history
└─ Domain-specific capability matching

Layer 4: Social (Detects anomalies)
├─ Interaction graph analysis
├─ Rating pattern detection
├─ Witness co-occurrence tracking
├─ Community reporting mechanisms
├─ Suspicious HEX pattern detection
└─ Cross-validation (DEX vs HEX consistency)

Layer 5: Operational (Limits damage)
├─ Rate limiting
├─ Graduated access (probation)
├─ Witness requirements for high-value
├─ Emergency revocation
└─ Domain breadth plausibility checks
```

### Critical Security Properties

**Guaranteed by protocol:**
1. ✅ Signatures cannot be forged (cryptographic)
2. ✅ Tokens cannot be modified without detection (cryptographic)
3. ✅ DEX converges to true reliability (mathematical)
4. ✅ Median consensus resists ⌊n/2⌋ corrupted witnesses (mathematical)
5. ✅ HEX can be verified against ledger history (cryptographic + ledger)

**Encouraged by incentives:**
6. ⚠️ Sybil attacks are expensive (economic, but not impossible)
7. ⚠️ Collusion is detectable (statistical, but requires monitoring)
8. ⚠️ Defection is costly (reputational, but one-time exploits possible)
9. ⚠️ HEX inflation is detectable (verifiable, but requires checking)

**Dependent on implementation:**
10. ❌ Keys are kept secure (implementation quality)
11. ❌ Ledger is honest (infrastructure choice)
12. ❌ Anomaly detection is active (operational vigilance)
13. ❌ HEX verification is performed (implementation diligence)

---

## Security Checklist

### For Protocol Implementers

**Identity Layer:**
- [ ] Ed25519 keys generated with secure randomness
- [ ] Private keys stored in HSM/TPM or OS keychain
- [ ] Key rotation mechanism implemented
- [ ] Fork history validated on every interaction
- [ ] Sybil ring detection enabled

**Reputation Layer:**
- [ ] Beta parameters validated (α, β > 0)
- [ ] Fork weights enforced (cannot be overridden)
- [ ] Probation mechanics implemented
- [ ] Confidence intervals calculated correctly
- [ ] Rating divergence detection active

**Delegation Layer:**
- [ ] Signature verification on every token
- [ ] TTL enforcement (reject expired tokens)
- [ ] Age-triggered revalidation implemented
- [ ] Scope validation (child ⊆ parent)
- [ ] Revocation checking enabled

**Experience Layer (HEX):**
- [ ] HEX verification against ledger history implemented
- [ ] Domain count consistency checks performed
- [ ] Confidence plausibility validation enabled
- [ ] Cross-validation with DEX score required
- [ ] Domain breadth plausibility checks active
- [ ] Session-backed HEX updates enforced
- [ ] Fork-based HEX inheritance properly weighted
- [ ] Domain-specific verification for task selection
- [ ] HEX privacy controls available (if supported)
- [ ] Suspicious HEX pattern detection active

**Witness Layer:**
- [ ] Sortition uses cryptographically secure randomness
- [ ] Conflict of interest checks performed
- [ ] Bond mechanics implemented
- [ ] Slashing conditions enforced
- [ ] Median consensus calculated correctly

**Session Layer:**
- [ ] Both signatures required before publication
- [ ] Timing plausibility checks
- [ ] Divergence threshold enforced (0.15 default)
- [ ] Dispute flagging automatic

**Operational:**
- [ ] Rate limiting on all endpoints
- [ ] Anomaly detection alerts configured
- [ ] Ledger backup and recovery tested
- [ ] Emergency revocation procedure documented
- [ ] Security incident response plan prepared

### For Deployers

**Before Launch:**
- [ ] Threat model reviewed and accepted
- [ ] Key management procedures documented
- [ ] Monitoring and alerting configured
- [ ] Witness pool seeded with known-good agents
- [ ] Test transactions validated

**Ongoing:**
- [ ] Daily review of anomaly detection alerts
- [ ] Weekly review of high-value transactions
- [ ] Monthly security audit of implementations
- [ ] Quarterly penetration testing
- [ ] Annual third-party security review

---

**Status:** SECURITY.md complete with comprehensive HEX security analysis  
**Coverage:** All four core primitives (ID, REP, DEX, HEX) + Session, Witness  
**HEX Attacks Analyzed:** Experience inflation, domain privacy, mismatch exploitation, fork poisoning  
**Next Document:** IMPLEMENTATION.md (Reference implementations including HEX)
