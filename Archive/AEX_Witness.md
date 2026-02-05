# AEX_WITNESS v1.0
## Third-Party Attestation and Consensus Protocol

**Last Updated:** February 4, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Design Principles](#design-principles)
3. [Witness Roles and Types](#witness-roles-and-types)
4. [Eligibility Criteria](#eligibility-criteria)
5. [Selection Mechanisms](#selection-mechanisms)
6. [Bond and Stake System](#bond-and-stake-system)
7. [Attestation Protocol](#attestation-protocol)
8. [Consensus Calculation](#consensus-calculation)
9. [Witness Reputation](#witness-reputation)
10. [Economic Model](#economic-model)
11. [Implementation Guide](#implementation-guide)
12. [Security Considerations](#security-considerations)
13. [Examples](#examples)

---

## Overview

AEX_WITNESS defines the third-party attestation layer for high-stakes agent interactions. It provides **independent verification** when bilateral agreement is insufficient or disputed.

### Core Functions

1. **Independent evaluation** - Third-party agents rate work quality
2. **Dispute resolution** - Break deadlock when parties disagree
3. **Quality assurance** - Add confidence to high-value interactions
4. **Fraud deterrence** - Economic consequences for bad witnessing
5. **Decentralized trust** - No central authority needed

### When Witnesses Used

**Automatic triggers:**
- Task value > $500
- `witness_required: true` in AEX_REP delegation token
- `min_witnesses` specified in delegation constraints
- Bilateral rating divergence > threshold (dispute)
- High-stakes domains (legal, medical, financial)

**Optional use:**
- Either party requests witnesses for added confidence
- Marketplace requires witnessed transactions
- Human issuer mandates witnessing in delegation

---

## Design Principles

### 1. Independence
Witnesses must be **unaffiliated** with transaction parties. No conflict of interest.

### 2. Competence
Witnesses must have **domain expertise** and **high DEX** in relevant areas.

### 3. Economic Accountability
Witnesses **stake bonds** that are slashed for misbehavior (collusion, defection, dishonesty).

### 4. Fair Selection
Witnesses chosen via **weighted random selection (sortition)** to prevent gaming.

### 5. Consensus-Driven
Multiple witnesses provide ratings; **consensus algorithm** determines final outcome.

### 6. Transparent Incentives
Witnesses earn **fees** for honest participation; lose **bonds** for misbehavior.

---

## Witness Roles and Types

### Observer Witness (Standard)

**Purpose:** Evaluate completed work and provide rating

**Responsibilities:**
- Review work deliverable
- Assess quality against task requirements
- Provide rating [0, 1] with brief attestation
- Sign attestation with witness identity

**Requirements:**
- DEX ≥ 0.80 (configurable)
- n_eff ≥ 50 (sufficient confidence)
- Domain experience (if multi-dimensional DEX)
- Available bond ≥ required amount

**Compensation:**
- Base fee: 2-5% of task value
- Bonus for consensus alignment
- Bond returned after completion

### Active Monitor (Advanced)

**Purpose:** Observe work in real-time during execution

**Responsibilities:**
- Monitor work as it progresses
- Detect issues early
- Provide interim feedback
- Final rating and attestation

**Requirements:**
- DEX ≥ 0.85
- n_eff ≥ 100
- Available for duration of task
- Higher bond requirement (2x standard)

**Compensation:**
- Base fee: 5-10% of task value
- Time-based component (hourly rate)
- Premium for real-time monitoring

### Arbitrator (Expert)

**Purpose:** Resolve complex disputes requiring deep expertise

**Responsibilities:**
- Review all evidence (work, ratings, context)
- Interview parties if necessary
- Provide detailed written decision
- Final binding ruling

**Requirements:**
- DEX ≥ 0.90
- n_eff ≥ 200
- Domain expertise verified
- Arbitration experience
- Higher bond (5x standard)

**Compensation:**
- Fee: 10-20% of task value
- Based on complexity and time
- Reputation bonus for fair arbitration

---

## Eligibility Criteria

### Global Eligibility

**Minimum requirements for any witness role:**
```python
def check_global_eligibility(agent_id, shared_ledger):
    """
    Check if agent meets basic witness eligibility
    
    Returns:
        (bool, str): (eligible, reason)
    """
    # Get agent DEX
    dex = query_agent_dex(agent_id, shared_ledger)
    
    if not dex:
        return False, "No DEX record found"
    
    dex_score = dex['alpha'] / (dex['alpha'] + dex['beta'])
    confidence = dex['alpha'] + dex['beta']
    
    # Check 1: Minimum DEX threshold
    if dex_score < 0.80:
        return False, f"DEX {dex_score:.2f} below minimum 0.80"
    
    # Check 2: Minimum confidence
    if confidence < 50:
        return False, f"Confidence {confidence:.0f} below minimum 50"
    
    # Check 3: Not in probation
    probation = check_probation_status(agent_id, shared_ledger)
    if probation['active']:
        return False, "Agent in probation period"
    
    # Check 4: No recent slashing
    witness_record = get_witness_record(agent_id, shared_ledger)
    if witness_record:
        recent_slashings = [
            s for s in witness_record.get('slashing_history', [])
            if (datetime.utcnow() - datetime.fromisoformat(s['timestamp'].replace('Z', '+00:00'))).days < 90
        ]
        if recent_slashings:
            return False, f"Slashed {len(recent_slashings)} times in last 90 days"
    
    # Check 5: Sufficient bond available
    witness_account = get_witness_account(agent_id)
    if not witness_account or witness_account['available_balance'] < 10:
        return False, "Insufficient bond available"
    
    return True, "Eligible"
```

### Domain Expertise

**For domain-specific witnessing:**
```python
def check_domain_expertise(agent_id, required_domains, min_domain_dex, shared_ledger):
    """
    Check if witness has expertise in required domains
    
    Returns:
        (bool, str, dict): (qualified, reason, domain_scores)
    """
    dex = query_agent_dex(agent_id, shared_ledger)
    
    if 'dimensions' not in dex:
        return False, "No domain-specific DEX available", {}
    
    domain_scores = {}
    
    for domain in required_domains:
        if domain not in dex['dimensions']:
            return False, f"No experience in domain '{domain}'", domain_scores
        
        domain_dex = dex['dimensions'][domain]
        score = domain_dex['alpha'] / (domain_dex['alpha'] + domain_dex['beta'])
        confidence = domain_dex['alpha'] + domain_dex['beta']
        
        domain_scores[domain] = {
            'score': score,
            'confidence': confidence
        }
        
        # Check score threshold
        if score < min_domain_dex:
            return False, f"Domain '{domain}' DEX {score:.2f} below threshold {min_domain_dex}", domain_scores
        
        # Check confidence
        if confidence < 20:
            return False, f"Domain '{domain}' confidence {confidence:.0f} too low", domain_scores
    
    return True, "Qualified in all required domains", domain_scores
```

### Conflict of Interest Check

**Ensure witness independence:**
```python
def check_conflicts_of_interest(witness_id, agent_a_id, agent_b_id, shared_ledger):
    """
    Check for conflicts of interest
    
    Conflicts:
    - Witness is one of the transaction parties
    - Witness has frequent interactions with either party
    - Witness is in same organization/delegation chain
    - Witness has financial relationship with parties
    
    Returns:
        (bool, list): (has_conflict, conflicts_found)
    """
    conflicts = []
    
    # Check 1: Not a transaction party
    if witness_id == agent_a_id or witness_id == agent_b_id:
        conflicts.append("Witness is a transaction party")
        return True, conflicts
    
    # Check 2: Interaction frequency
    witness_sessions = query_agent_sessions(witness_id, shared_ledger, limit=100)
    
    interactions_with_a = sum(
        1 for s in witness_sessions
        if agent_a_id in [s['participants']['agent_a']['aex_id'], 
                         s['participants']['agent_b']['aex_id']]
    )
    
    interactions_with_b = sum(
        1 for s in witness_sessions
        if agent_b_id in [s['participants']['agent_a']['aex_id'], 
                         s['participants']['agent_b']['aex_id']]
    )
    
    # If >20% of recent interactions with either party, flag conflict
    if interactions_with_a / len(witness_sessions) > 0.2:
        conflicts.append(f"Frequent interactions with Agent A ({interactions_with_a}/100)")
    
    if interactions_with_b / len(witness_sessions) > 0.2:
        conflicts.append(f"Frequent interactions with Agent B ({interactions_with_b}/100)")
    
    # Check 3: Delegation chain relationship
    # (Check if witness is delegated by same human as either party)
    witness_identity = shared_ledger.get(f"identities/{witness_id}/aex_id.json")
    agent_a_identity = shared_ledger.get(f"identities/{agent_a_id}/aex_id.json")
    agent_b_identity = shared_ledger.get(f"identities/{agent_b_id}/aex_id.json")
    
    # Check fork lineage for common ancestors
    if has_common_ancestor(witness_identity, agent_a_identity, depth=3):
        conflicts.append("Common lineage with Agent A")
    
    if has_common_ancestor(witness_identity, agent_b_identity, depth=3):
        conflicts.append("Common lineage with Agent B")
    
    return len(conflicts) > 0, conflicts

def has_common_ancestor(identity_1, identity_2, depth=3):
    """
    Check if two identities share common ancestor within depth
    """
    ancestors_1 = get_ancestors(identity_1, depth)
    ancestors_2 = get_ancestors(identity_2, depth)
    
    return bool(set(ancestors_1) & set(ancestors_2))
```

---

## Selection Mechanisms

### Sortition (Weighted Random Selection)

**Default mechanism:** Fair selection weighted by reputation
```python
def select_witnesses_sortition(session_id, required_count, min_dex, domains, 
                               agent_a_id, agent_b_id, shared_ledger):
    """
    Select witnesses via weighted random selection (sortition)
    
    Process:
    1. Identify eligible candidates
    2. Filter by conflict of interest
    3. Weight by DEX and domain expertise
    4. Random selection proportional to weight
    
    Args:
        session_id: Session requiring witnesses
        required_count: Number of witnesses needed (typically 2-3)
        min_dex: Minimum DEX threshold
        domains: Required domain expertise
        agent_a_id, agent_b_id: Transaction parties (for CoI check)
        shared_ledger: Ledger interface
    
    Returns:
        List of selected witness agent IDs
    """
    # Stage 1: Global eligibility
    all_agents = get_all_agents(shared_ledger)
    eligible = []
    
    for agent_id in all_agents:
        # Check global eligibility
        is_eligible, reason = check_global_eligibility(agent_id, shared_ledger)
        if not is_eligible:
            continue
        
        # Check domain expertise
        if domains:
            is_qualified, reason, domain_scores = check_domain_expertise(
                agent_id, domains, min_dex, shared_ledger
            )
            if not is_qualified:
                continue
        
        # Check conflicts of interest
        has_conflict, conflicts = check_conflicts_of_interest(
            agent_id, agent_a_id, agent_b_id, shared_ledger
        )
        if has_conflict:
            continue
        
        # Agent is eligible
        eligible.append(agent_id)
    
    if len(eligible) < required_count:
        raise InsufficientWitnessesError(
            f"Only {len(eligible)} eligible witnesses, need {required_count}"
        )
    
    # Stage 2: Calculate weights
    weighted_candidates = []
    
    for agent_id in eligible:
        dex = query_agent_dex(agent_id, shared_ledger)
        
        # Base weight from global DEX
        dex_score = dex['alpha'] / (dex['alpha'] + dex['beta'])
        confidence = dex['alpha'] + dex['beta']
        
        # Confidence factor (cap at 100)
        confidence_factor = min(confidence / 100, 1.0)
        
        # Domain expertise bonus
        domain_bonus = 1.0
        if domains and 'dimensions' in dex:
            domain_scores = []
            for domain in domains:
                if domain in dex['dimensions']:
                    dim = dex['dimensions'][domain]
                    dim_score = dim['alpha'] / (dim['alpha'] + dim['beta'])
                    domain_scores.append(dim_score)
            
            if domain_scores:
                avg_domain_dex = sum(domain_scores) / len(domain_scores)
                domain_bonus = 1.0 + (avg_domain_dex - min_dex)  # Bonus for exceeding threshold
        
        # Witness experience bonus
        witness_record = get_witness_record(agent_id, shared_ledger)
        witness_experience = witness_record.get('total_attestations', 0) if witness_record else 0
        experience_bonus = 1.0 + min(witness_experience / 100, 0.5)  # Up to +50% for 100+ attestations
        
        # Final weight
        weight = dex_score * confidence_factor * domain_bonus * experience_bonus
        
        weighted_candidates.append({
            'agent_id': agent_id,
            'weight': weight,
            'dex': dex_score,
            'confidence': confidence
        })
    
    # Stage 3: Weighted random selection
    selected = []
    remaining = list(weighted_candidates)
    
    for _ in range(required_count):
        if not remaining:
            break
        
        # Calculate cumulative weights
        total_weight = sum(c['weight'] for c in remaining)
        
        # Weighted random choice
        rand = random.random() * total_weight
        cumulative = 0
        
        for candidate in remaining:
            cumulative += candidate['weight']
            if rand <= cumulative:
                selected.append(candidate['agent_id'])
                remaining.remove(candidate)
                break
    
    return selected
```

### Direct Appointment (Optional)

**Parties agree on specific witnesses:**
```python
def appoint_witnesses_direct(witness_ids, agent_a_id, agent_b_id, domains, 
                             min_dex, shared_ledger):
    """
    Directly appoint specific witnesses (requires both parties' consent)
    
    Validation:
    - All witnesses meet eligibility criteria
    - No conflicts of interest
    - Both parties sign appointment
    
    Returns:
        (bool, list): (approved, witness_ids or reasons)
    """
    reasons = []
    
    for witness_id in witness_ids:
        # Check eligibility
        eligible, reason = check_global_eligibility(witness_id, shared_ledger)
        if not eligible:
            reasons.append(f"{witness_id}: {reason}")
            continue
        
        # Check domain expertise
        if domains:
            qualified, reason, scores = check_domain_expertise(
                witness_id, domains, min_dex, shared_ledger
            )
            if not qualified:
                reasons.append(f"{witness_id}: {reason}")
                continue
        
        # Check conflicts
        has_conflict, conflicts = check_conflicts_of_interest(
            witness_id, agent_a_id, agent_b_id, shared_ledger
        )
        if has_conflict:
            reasons.append(f"{witness_id}: {', '.join(conflicts)}")
            continue
    
    if reasons:
        return False, reasons
    
    return True, witness_ids
```

### Marketplace Selection

**Witnesses advertise availability and rates:**
```python
def select_witnesses_marketplace(required_count, domains, max_fee, shared_ledger):
    """
    Select witnesses from marketplace based on availability and price
    
    Process:
    1. Query available witnesses in marketplace
    2. Filter by domains and eligibility
    3. Sort by fee (ascending) then DEX (descending)
    4. Select top N within budget
    """
    # Query marketplace
    available_witnesses = query_witness_marketplace(
        domains=domains,
        max_fee=max_fee,
        shared_ledger=shared_ledger
    )
    
    # Filter by eligibility
    eligible = []
    for listing in available_witnesses:
        witness_id = listing['witness_id']
        
        is_eligible, reason = check_global_eligibility(witness_id, shared_ledger)
        if not is_eligible:
            continue
        
        if domains:
            is_qualified, reason, scores = check_domain_expertise(
                witness_id, domains, 0.80, shared_ledger
            )
            if not is_qualified:
                continue
        
        eligible.append(listing)
    
    # Sort by fee (primary) then DEX (secondary)
    eligible.sort(key=lambda x: (x['fee'], -x['dex_score']))
    
    # Select top N
    selected = [listing['witness_id'] for listing in eligible[:required_count]]
    
    if len(selected) < required_count:
        raise InsufficientWitnessesError(
            f"Only {len(selected)} available within budget, need {required_count}"
        )
    
    return selected
```

---

## Bond and Stake System

### Bond Mechanics

**Purpose:** Economic security - witnesses stake value they can lose
```python
class WitnessAccount:
    """
    Witness financial account for bonds and earnings
    """
    
    def __init__(self, witness_id):
        self.witness_id = witness_id
        self.available_balance = 0.0
        self.locked_bonds = {}  # session_id -> bond_record
        self.total_earned = 0.0
        self.total_slashed = 0.0
        self.bond_history = []
    
    def deposit(self, amount):
        """Add funds to account"""
        self.available_balance += amount
    
    def lock_bond(self, session_id, amount):
        """
        Lock bond for witness session
        
        Raises:
            InsufficientBalanceError if balance < amount
        """
        if self.available_balance < amount:
            raise InsufficientBalanceError(
                f"Balance {self.available_balance} < required {amount}"
            )
        
        self.locked_bonds[session_id] = {
            'amount': amount,
            'locked_at': datetime.utcnow().isoformat() + 'Z',
            'status': 'locked'
        }
        
        self.available_balance -= amount
    
    def release_bond(self, session_id, fee):
        """
        Release bond after successful attestation
        
        Returns bond + fee to available balance
        """
        if session_id not in self.locked_bonds:
            raise ValueError(f"No locked bond for session {session_id}")
        
        bond_record = self.locked_bonds[session_id]
        bond_amount = bond_record['amount']
        
        # Return bond + fee
        self.available_balance += bond_amount + fee
        self.total_earned += fee
        
        # Move to history
        self.bond_history.append({
            'session_id': session_id,
            'amount': bond_amount,
            'fee_earned': fee,
            'outcome': 'released',
            'timestamp': datetime.utcnow().isoformat() + 'Z'
        })
        
        del self.locked_bonds[session_id]
    
    def slash_bond(self, session_id, slash_amount, reason):
        """
        Slash bond for misbehavior
        
        Bond is forfeit, not returned to witness
        """
        if session_id not in self.locked_bonds:
            raise ValueError(f"No locked bond for session {session_id}")
        
        bond_record = self.locked_bonds[session_id]
        bond_amount = bond_record['amount']
        
        # Determine slash amount (partial or full)
        actual_slash = min(slash_amount, bond_amount)
        remainder = bond_amount - actual_slash
        
        # Return remainder if partial slash
        if remainder > 0:
            self.available_balance += remainder
        
        self.total_slashed += actual_slash
        
        # Record slashing
        self.bond_history.append({
            'session_id': session_id,
            'amount': bond_amount,
            'slashed': actual_slash,
            'reason': reason,
            'outcome': 'slashed',
            'timestamp': datetime.utcnow().isoformat() + 'Z'
        })
        
        del self.locked_bonds[session_id]
```

### Bond Requirements by Session Value
```python
def calculate_required_bond(session_value, witness_role='observer'):
    """
    Calculate required bond based on session value and witness role
    
    Bond scales with session value to align incentives
    """
    if witness_role == 'observer':
        # Standard: 2% of session value, min $10
        bond = max(session_value * 0.02, 10.0)
    
    elif witness_role == 'monitor':
        # Active monitor: 4% of session value, min $20
        bond = max(session_value * 0.04, 20.0)
    
    elif witness_role == 'arbitrator':
        # Arbitrator: 10% of session value, min $50
        bond = max(session_value * 0.10, 50.0)
    
    return bond

# Examples:
# Session value $100 → Observer bond $10 (capped at minimum)
# Session value $1000 → Observer bond $20
# Session value $5000 → Observer bond $100
# Session value $10000 → Arbitrator bond $1000
```

### Slashing Conditions

**When bonds are slashed:**

| Violation | Slash Amount | Rationale |
|-----------|--------------|-----------|
| **Defection** (fails to provide attestation) | 50% of bond | Wasted selection, delayed resolution |
| **Extreme outlier** (rating >0.3 from consensus) | 25% of bond | Likely dishonest or incompetent |
| **Collusion** (proven coordination with party) | 100% of bond + DEX penalty | Fraud |
| **Multiple defections** (3+ in 90 days) | 100% of bond + witness ban | Pattern of unreliability |
| **Deadline miss** (attestation >2x expected time) | 10% of bond | Delay penalty |
```python
def evaluate_slashing(witness_id, session_id, witness_rating, consensus_outcome, 
                     attestation_provided, deadline_met, shared_ledger):
    """
    Evaluate if witness should be slashed
    
    Returns:
        (bool, float, str): (should_slash, slash_percentage, reason)
    """
    witness_account = get_witness_account(witness_id)
    bond_amount = witness_account.locked_bonds[session_id]['amount']
    
    # Check 1: Defection (no attestation provided)
    if not attestation_provided:
        # Check history for pattern
        recent_defections = count_recent_defections(witness_id, days=90)
        
        if recent_defections >= 3:
            return True, 1.0, "Multiple defections (3+ in 90 days) - witness ban"
        else:
            return True, 0.5, "Failed to provide attestation"
    
    # Check 2: Extreme outlier
    if consensus_outcome is not None and witness_rating is not None:
        divergence = abs(witness_rating - consensus_outcome)
        
        if divergence > 0.3:
            return True, 0.25, f"Extreme outlier (divergence {divergence:.2f})"
    
    # Check 3: Deadline miss
    if not deadline_met:
        return True, 0.1, "Missed attestation deadline"
    
    # Check 4: Collusion (requires separate investigation, not automated)
    # This would be flagged manually and processed separately
    
    # No slashing
    return False, 0.0, "No violations"
```

---

## Attestation Protocol

### Attestation Structure
```json
{
  "attestation_id": "att_uuid",
  "session_id": "session_uuid",
  "witness_id": "did:aex:WitnessAgent",
  "role": "observer",
  "selection_method": "sortition",
  "bond": {
    "amount": 10,
    "locked_at": "2026-02-04T12:30:00Z",
    "status": "locked"
  },
  "evaluation": {
    "rating": 0.85,
    "confidence": 0.9,
    "dimensions": {
      "accuracy": 0.90,
      "completeness": 0.85,
      "presentation": 0.80
    }
  },
  "attestation_text": "Thorough analysis with minor formatting issues. Conclusions well-supported by data.",
  "evidence_reviewed": {
    "deliverable_hash": "sha256_hash",
    "task_description": true,
    "agent_ratings": true,
    "context_documents": false
  },
  "timestamps": {
    "appointed": "2026-02-04T12:30:00Z",
    "started_review": "2026-02-04T12:35:00Z",
    "completed_review": "2026-02-04T12:50:00Z",
    "submitted": "2026-02-04T12:51:00Z"
  },
  "signature": "witness_signature_base58",
  "metadata": {
    "review_duration": 900,
    "witness_dex_at_time": 0.88,
    "witness_domain_dex": {
      "finance": 0.92
    }
  }
}
```

### Attestation Workflow
```python
def perform_witness_attestation(witness_id, session_id, witness_private_key, shared_ledger):
    """
    Complete witness attestation workflow
    
    Steps:
    1. Lock bond
    2. Retrieve work deliverable
    3. Evaluate quality
    4. Generate rating and attestation
    5. Sign attestation
    6. Submit to session
    
    Returns:
        Signed attestation record
    """
    # Step 1: Lock bond
    session = shared_ledger.get(f"sessions/{session_id}.json")
    task_value = session['task_summary']['actual_cost']
    bond_amount = calculate_required_bond(task_value, 'observer')
    
    witness_account = get_witness_account(witness_id)
    witness_account.lock_bond(session_id, bond_amount)
    
    # Step 2: Retrieve deliverable
    appointed_time = datetime.utcnow()
    
    deliverable = get_work_deliverable(session_id)
    task_description = session['task_summary']['description']
    agent_a_rating = session['outcome']['agent_a_rating']
    agent_b_rating = session['outcome']['agent_b_rating']
    
    # Step 3: Evaluate (this is witness-specific logic, not protocol-defined)
    review_start = datetime.utcnow()
    
    evaluation_result = witness_evaluate_work(
        deliverable=deliverable,
        task_description=task_description,
        task_requirements=session['task_summary']
    )
    
    review_end = datetime.utcnow()
    
    # Step 4: Generate attestation
    attestation = {
        'attestation_id': str(uuid.uuid4()),
        'session_id': session_id,
        'witness_id': witness_id,
        'role': 'observer',
        'selection_method': 'sortition',
        'bond': {
            'amount': bond_amount,
            'locked_at': appointed_time.isoformat() + 'Z',
            'status': 'locked'
        },
        'evaluation': {
            'rating': evaluation_result['rating'],
            'confidence': evaluation_result.get('confidence', 0.9),
            'dimensions': evaluation_result.get('dimensions', {})
        },
        'attestation_text': evaluation_result['attestation_text'],
        'evidence_reviewed': {
            'deliverable_hash': hash_content(deliverable),
            'task_description': True,
            'agent_ratings': True,
            'context_documents': False
        },
        'timestamps': {
            'appointed': appointed_time.isoformat() + 'Z',
            'started_review': review_start.isoformat() + 'Z',
            'completed_review': review_end.isoformat() + 'Z',
            'submitted': None  # Filled after signing
        },
        'metadata': {
            'review_duration': (review_end - review_start).total_seconds(),
            'witness_dex_at_time': get_agent_dex_score(witness_id, shared_ledger),
            'witness_domain_dex': get_agent_domain_dex(witness_id, session['task_summary']['domains'], shared_ledger)
        }
    }
    
    # Step 5: Sign attestation
    signed_attestation = sign_attestation(attestation, witness_private_key)
    signed_attestation['timestamps']['submitted'] = datetime.utcnow().isoformat() + 'Z'
    
    # Step 6: Submit to session (this updates the session record)
    return signed_attestation

def sign_attestation(attestation, witness_private_key):
    """
    Witness signs attestation
    """
    attestation.pop('signature', None)
    
    canonical = json.dumps(
        attestation,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False
    ).encode('utf-8')
    
    signing_key = nacl.signing.SigningKey(witness_private_key)
    signed = signing_key.sign(canonical)
    signature_b58 = base58.b58encode(signed.signature).decode('ascii')
    
    attestation['signature'] = signature_b58
    
    return attestation
```

### Attestation Submission and Integration
```python
def submit_witness_attestation(attestation, session_id, shared_ledger):
    """
    Submit witness attestation to session
    
    Updates session record with witness attestation
    """
    session = shared_ledger.get(f"sessions/{session_id}.json")
    
    # Verify witness signature
    valid = verify_attestation_signature(attestation, shared_ledger)
    if not valid:
        raise SignatureError("Invalid witness attestation signature")
    
    # Add to session witnesses
    session['witnesses'].append({
        'witness_id': attestation['witness_id'],
        'role': attestation['role'],
        'selection_method': attestation['selection_method'],
        'bond_amount': attestation['bond']['amount'],
        'bond_status': 'locked',
        'rating': attestation['evaluation']['rating'],
        'attestation': attestation['attestation_text'],
        'timestamp': attestation['timestamps']['submitted'],
        'signature': attestation['signature']
    })
    
    # Store full attestation separately
    shared_ledger.put(
        f"attestations/{attestation['attestation_id']}.json",
        attestation
    )
    
    # Update session
    shared_ledger.put(f"sessions/{session_id}.json", session)
    
    return session
```

---

## Consensus Calculation

### Median Consensus (Default)

**Resistant to outliers:**
```python
def calculate_witness_consensus_median(witness_ratings):
    """
    Calculate consensus using median (default method)
    
    Median is robust to outliers and strategic manipulation
    
    Args:
        witness_ratings: List of ratings [0, 1]
    
    Returns:
        Consensus outcome value
    """
    if not witness_ratings:
        return None
    
    sorted_ratings = sorted(witness_ratings)
    n = len(sorted_ratings)
    
    if n % 2 == 0:
        # Even number: average of two middle values
        consensus = (sorted_ratings[n//2 - 1] + sorted_ratings[n//2]) / 2
    else:
        # Odd number: middle value
        consensus = sorted_ratings[n//2]
    
    return consensus

# Example: 3 witnesses
ratings = [0.85, 0.82, 0.88]
consensus = calculate_witness_consensus_median(ratings)
# Result: 0.85 (middle value)

# Example with outlier: 3 witnesses, 1 dishonest
ratings = [0.85, 0.82, 0.30]
consensus = calculate_witness_consensus_median(ratings)
# Result: 0.82 (outlier ignored)

# Example: 2 witnesses
ratings = [0.85, 0.80]
consensus = calculate_witness_consensus_median(ratings)
# Result: 0.825 (average)
```

### Trimmed Mean (Alternative)

**Remove extremes, average the rest:**
```python
def calculate_witness_consensus_trimmed_mean(witness_ratings, trim_percent=0.1):
    """
    Calculate consensus using trimmed mean
    
    Remove top and bottom percentiles, average the rest
    
    Args:
        witness_ratings: List of ratings
        trim_percent: Percentage to trim from each end (default 10%)
    
    Returns:
        Consensus outcome value
    """
    if not witness_ratings:
        return None
    
    if len(witness_ratings) < 5:
        # Too few ratings to trim effectively, use median
        return calculate_witness_consensus_median(witness_ratings)
    
    sorted_ratings = sorted(witness_ratings)
    n = len(sorted_ratings)
    
    # Calculate trim count
    trim_count = max(1, int(n * trim_percent))
    
    # Trim from both ends
    trimmed = sorted_ratings[trim_count:-trim_count]
    
    if not trimmed:
        # All trimmed, fall back to median
        return calculate_witness_consensus_median(witness_ratings)
    
    # Average remaining
    consensus = sum(trimmed) / len(trimmed)
    
    return consensus

# Example: 5 witnesses with one outlier on each end
ratings = [0.30, 0.80, 0.85, 0.82, 0.95]
consensus = calculate_witness_consensus_trimmed_mean(ratings)
# Trim 1 from each end: [0.80, 0.85, 0.82]
# Result: 0.823 (average of middle three)
```

### Weighted Average by Witness DEX (Advanced)

**Weight ratings by witness credibility:**
```python
def calculate_witness_consensus_weighted(attestations, shared_ledger):
    """
    Calculate consensus with weights based on witness DEX
    
    More credible witnesses have more influence
    
    Args:
        attestations: List of attestation objects (include witness_id and rating)
        shared_ledger: Ledger interface
    
    Returns:
        Weighted consensus outcome
    """
    if not attestations:
        return None
    
    weighted_sum = 0.0
    total_weight = 0.0
    
    for att in attestations:
        witness_id = att['witness_id']
        rating = att['evaluation']['rating']
        
        # Get witness DEX
        dex = query_agent_dex(witness_id, shared_ledger)
        dex_score = dex['alpha'] / (dex['alpha'] + dex['beta'])
        confidence = dex['alpha'] + dex['beta']
        
        # Weight calculation
        # Higher DEX and confidence = more weight
        weight = dex_score * min(confidence / 100, 1.0)
        
        weighted_sum += rating * weight
        total_weight += weight
    
    consensus = weighted_sum / total_weight if total_weight > 0 else None
    
    return consensus

# Example: 3 witnesses with varying credibility
attestations = [
    {'witness_id': 'W1', 'evaluation': {'rating': 0.85}},  # DEX=0.90, conf=100 → weight=0.90
    {'witness_id': 'W2', 'evaluation': {'rating': 0.80}},  # DEX=0.85, conf=80 → weight=0.68
    {'witness_id': 'W3', 'evaluation': {'rating': 0.88}}   # DEX=0.92, conf=120 → weight=0.92
]

# Weighted average: (0.85×0.90 + 0.80×0.68 + 0.88×0.92) / (0.90 + 0.68 + 0.92)
# = 0.8475 (high-DEX witnesses pull average toward their ratings)
```

### Consensus Resolution

**Complete workflow:**
```python
def resolve_session_with_witnesses(session_id, method='median', shared_ledger):
    """
    Resolve session outcome using witness consensus
    
    Args:
        session_id: Session to resolve
        method: 'median', 'trimmed_mean', or 'weighted'
        shared_ledger: Ledger interface
    
    Returns:
        Updated session with consensus outcome
    """
    session = shared_ledger.get(f"sessions/{session_id}.json")
    
    # Extract witness ratings
    witness_ratings = [w['rating'] for w in session['witnesses']]
    
    if not witness_ratings:
        raise ValueError("No witness ratings available")
    
    # Calculate consensus
    if method == 'median':
        consensus = calculate_witness_consensus_median(witness_ratings)
    elif method == 'trimmed_mean':
        consensus = calculate_witness_consensus_trimmed_mean(witness_ratings)
    elif method == 'weighted':
        consensus = calculate_witness_consensus_weighted(session['witnesses'], shared_ledger)
    else:
        raise ValueError(f"Unknown consensus method: {method}")
    
    # Update session outcome
    session['outcome']['agreed_outcome'] = consensus
    session['outcome']['agreement_method'] = f'witness_consensus_{method}'
    session['outcome']['witness_ratings'] = witness_ratings
    session['outcome']['consensus_achieved'] = True
    session['timestamps']['outcome_agreed'] = datetime.utcnow().isoformat() + 'Z'
    
    # Resolve dispute if present
    if session['dispute']:
        session['dispute']['status'] = 'resolved'
        session['dispute']['resolution'] = {
            'method': 'witness_consensus',
            'consensus_algorithm': method,
            'witness_count': len(witness_ratings),
            'agreed_outcome': consensus
        }
        session['dispute']['resolved_at'] = datetime.utcnow().isoformat() + 'Z'
    
    # Release witness bonds and pay fees
    for witness_record in session['witnesses']:
        witness_id = witness_record['witness_id']
        
        # Evaluate slashing
        should_slash, slash_pct, reason = evaluate_slashing(
            witness_id=witness_id,
            session_id=session_id,
            witness_rating=witness_record['rating'],
            consensus_outcome=consensus,
            attestation_provided=True,
            deadline_met=True,  # Simplified, would check actual timing
            shared_ledger=shared_ledger
        )
        
        witness_account = get_witness_account(witness_id)
        
        if should_slash:
            # Slash bond
            slash_amount = witness_account.locked_bonds[session_id]['amount'] * slash_pct
            witness_account.slash_bond(session_id, slash_amount, reason)
            witness_record['bond_status'] = f'slashed_{slash_pct*100:.0f}%'
        else:
            # Release bond + fee
            task_value = session['task_summary']['actual_cost']
            fee = task_value * 0.03  # 3% of task value
            witness_account.release_bond(session_id, fee)
            witness_record['bond_status'] = 'released'
    
    # Publish updated session
    shared_ledger.put(f"sessions/{session_id}.json", session)
    
    return session
```

---

## Witness Reputation

### Witness-Specific Metrics

**Beyond agent DEX, track witness performance:**
```python
class WitnessRecord:
    """
    Track witness-specific performance metrics
    """
    
    def __init__(self, witness_id):
        self.witness_id = witness_id
        self.total_attestations = 0
        self.successful_attestations = 0  # Not slashed
        self.defections = 0
        self.slashing_history = []
        self.average_alignment = 0.0  # How close to consensus
        self.average_review_time = 0.0  # Seconds
        self.domains_witnessed = {}  # domain -> count
        self.earnings_history = []
    
    def record_attestation(self, session_id, rating, consensus, slashed, 
                          review_time, fee_earned, domains):
        """
        Record completed attestation
        """
        self.total_attestations += 1
        
        if not slashed:
            self.successful_attestations += 1
        else:
            self.slashing_history.append({
                'session_id': session_id,
                'timestamp': datetime.utcnow().isoformat() + 'Z',
                'reason': 'See session dispute record'
            })
        
        # Update alignment metric
        if consensus is not None:
            alignment = 1.0 - abs(rating - consensus)
            self.average_alignment = (
                (self.average_alignment * (self.total_attestations - 1) + alignment) /
                self.total_attestations
            )
        
        # Update review time
        self.average_review_time = (
            (self.average_review_time * (self.total_attestations - 1) + review_time) /
            self.total_attestations
        )
        
        # Update domains
        for domain in domains:
            self.domains_witnessed[domain] = self.domains_witnessed.get(domain, 0) + 1
        
        # Record earnings
        if fee_earned > 0:
            self.earnings_history.append({
                'session_id': session_id,
                'fee': fee_earned,
                'timestamp': datetime.utcnow().isoformat() + 'Z'
            })
    
    def get_witness_quality_score(self):
        """
        Calculate overall witness quality score
        
        Factors:
        - Success rate (not slashed)
        - Alignment with consensus
        - Reliability (few defections)
        - Experience (total attestations)
        """
        if self.total_attestations == 0:
            return 0.0
        
        # Success rate (0-1)
        success_rate = self.successful_attestations / self.total_attestations
        
        # Alignment (0-1)
        alignment = self.average_alignment
        
        # Reliability (0-1, penalize defections)
        defection_rate = self.defections / max(self.total_attestations, 1)
        reliability = max(0, 1.0 - (defection_rate * 5))  # Heavy penalty for defections
        
        # Experience factor (0-1, asymptotic to 1)
        experience = min(self.total_attestations / 100, 1.0)
        
        # Weighted combination
        quality_score = (
            success_rate * 0.4 +
            alignment * 0.3 +
            reliability * 0.2 +
            experience * 0.1
        )
        
        return quality_score
```

### Witness Selection Priority

**Prefer high-quality witnesses:**
```python
def calculate_witness_priority(witness_id, shared_ledger):
    """
    Calculate priority score for witness selection
    
    Higher priority = more likely to be selected
    
    Returns:
        Priority score [0, 1]
    """
    # Get agent DEX
    dex = query_agent_dex(witness_id, shared_ledger)
    dex_score = dex['alpha'] / (dex['alpha'] + dex['beta'])
    
    # Get witness record
    witness_record = get_witness_record(witness_id, shared_ledger)
    
    if not witness_record:
        # New witness, use DEX only
        return dex_score * 0.7  # Discount for lack of witness experience
    
    # Get witness quality score
    quality_score = witness_record.get_witness_quality_score()
    
    # Combine DEX and witness quality
    priority = (dex_score * 0.5) + (quality_score * 0.5)
    
    return priority
```

---

## Economic Model

### Fee Structure

**Witness compensation:**

| Session Value | Observer Fee | Monitor Fee | Arbitrator Fee |
|---------------|-------------|-------------|----------------|
| $0 - $100 | 3% (min $3) | 5% (min $5) | 10% (min $10) |
| $100 - $1000 | 2.5% | 4% | 8% |
| $1000 - $5000 | 2% | 3% | 6% |
| $5000+ | 1.5% | 2.5% | 5% |
```python
def calculate_witness_fee(session_value, witness_role='observer'):
    """
    Calculate witness fee based on session value and role
    
    Fee decreases as percentage for higher-value sessions
    """
    if witness_role == 'observer':
        if session_value < 100:
            fee = max(session_value * 0.03, 3.0)
        elif session_value < 1000:
            fee = session_value * 0.025
        elif session_value < 5000:
            fee = session_value * 0.02
        else:
            fee = session_value * 0.015
    
    elif witness_role == 'monitor':
        if session_value < 100:
            fee = max(session_value * 0.05, 5.0)
        elif session_value < 1000:
            fee = session_value * 0.04
        elif session_value < 5000:
            fee = session_value * 0.03
        else:
            fee = session_value * 0.025
    
    elif witness_role == 'arbitrator':
        if session_value < 100:
            fee = max(session_value * 0.10, 10.0)
        elif session_value < 1000:
            fee = session_value * 0.08
        elif session_value < 5000:
            fee = session_value * 0.06
        else:
            fee = session_value * 0.05
    
    return fee

# Examples:
# $50 task → Observer: $3 (minimum)
# $500 task → Observer: $12.50 (2.5%)
# $5000 task → Observer: $100 (2%)
# $20000 task → Arbitrator: $1000 (5%)
```

### Bonus for Consensus Alignment

**Reward witnesses who align with consensus:**
```python
def calculate_alignment_bonus(witness_rating, consensus, base_fee):
    """
    Calculate bonus for aligning with consensus
    
    Incentivizes honest, accurate attestation
    """
    alignment = 1.0 - abs(witness_rating - consensus)
    
    if alignment >= 0.95:  # Within 5% of consensus
        bonus_pct = 0.20  # +20% bonus
    elif alignment >= 0.90:  # Within 10%
        bonus_pct = 0.10  # +10% bonus
    elif alignment >= 0.85:  # Within 15%
        bonus_pct = 0.05  # +5% bonus
    else:
        bonus_pct = 0.0  # No bonus
    
    bonus = base_fee * bonus_pct
    
    return bonus

# Example:
# Base fee: $10
# Witness rating: 0.84, Consensus: 0.85
# Alignment: 0.99 (within 1%)
# Bonus: $10 × 0.20 = $2
# Total: $12
```

### Slashed Bond Distribution

**Where slashed bonds go:**
```python
def distribute_slashed_bond(session_id, slashed_amount, shared_ledger):
    """
    Distribute slashed bond among parties
    
    Distribution:
    - 50% to transaction parties (split equally)
    - 30% to other witnesses (if any)
    - 20% to protocol treasury
    """
    session = shared_ledger.get(f"sessions/{session_id}.json")
    
    agent_a_id = session['participants']['agent_a']['aex_id']
    agent_b_id = session['participants']['agent_b']['aex_id']
    
    # 50% to transaction parties
    party_share = slashed_amount * 0.50 / 2
    credit_agent_account(agent_a_id, party_share)
    credit_agent_account(agent_b_id, party_share)
    
    # 30% to other witnesses
    other_witnesses = [
        w for w in session['witnesses']
        if w['bond_status'] == 'released'  # Only honest witnesses
    ]
    
    if other_witnesses:
        witness_share = slashed_amount * 0.30 / len(other_witnesses)
        for witness in other_witnesses:
            credit_agent_account(witness['witness_id'], witness_share)
    else:
        # No other witnesses, add to treasury
        slashed_amount += slashed_amount * 0.30
    
    # 20% to protocol treasury
    treasury_share = slashed_amount * 0.20
    credit_protocol_treasury(treasury_share)
```

---

## Implementation Guide

### Minimal Implementation

**Required components:**
- [ ] Witness eligibility checking
- [ ] Sortition selection
- [ ] Bond lock/release
- [ ] Attestation signing
- [ ] Median consensus
- [ ] Slashing evaluation

**Recommended components:**
- [ ] Conflict of interest detection
- [ ] Witness quality scoring
- [ ] Fee calculation
- [ ] Alignment bonuses
- [ ] Marketplace support

### Reference Implementation
```python
class AEXWitnessSystem:
    """Reference implementation of AEX_WITNESS protocol"""
    
    def __init__(self, shared_ledger):
        self.shared_ledger = shared_ledger
        self.witness_accounts = {}
    
    def select_witnesses(self, session_id, required_count, min_dex, domains):
        """
        Select witnesses for session via sortition
        """
        session = self.shared_ledger.get(f"sessions/{session_id}.json")
        agent_a_id = session['participants']['agent_a']['aex_id']
        agent_b_id = session['participants']['agent_b']['aex_id']
        
        selected = select_witnesses_sortition(
            session_id=session_id,
            required_count=required_count,
            min_dex=min_dex,
            domains=domains,
            agent_a_id=agent_a_id,
            agent_b_id=agent_b_id,
            shared_ledger=self.shared_ledger
        )
        
        return selected
    
    def appoint_witness(self, witness_id, session_id):
        """
        Appoint witness to session and lock bond
        """
        session = self.shared_ledger.get(f"sessions/{session_id}.json")
        task_value = session['task_summary']['actual_cost']
        
        bond_amount = calculate_required_bond(task_value, 'observer')
        
        witness_account = self.get_or_create_witness_account(witness_id)
        witness_account.lock_bond(session_id, bond_amount)
        
        return bond_amount
    
    def submit_attestation(self, attestation, session_id):
        """
        Submit witness attestation
        """
        return submit_witness_attestation(attestation, session_id, self.shared_ledger)
    
    def resolve_with_consensus(self, session_id, method='median'):
        """
        Calculate consensus and resolve session
        """
        session = resolve_session_with_witnesses(
            session_id, method, self.shared_ledger
        )
        
        # Process witness payments
        self.process_witness_payments(session_id)
        
        return session
    
    def process_witness_payments(self, session_id):
        """
        Release bonds and pay fees to witnesses
        """
        session = self.shared_ledger.get(f"sessions/{session_id}.json")
        task_value = session['task_summary']['actual_cost']
        consensus = session['outcome']['agreed_outcome']
        
        for witness_record in session['witnesses']:
            witness_id = witness_record['witness_id']
            rating = witness_record['rating']
            
            witness_account = self.get_or_create_witness_account(witness_id)
            
            # Calculate fee
            base_fee = calculate_witness_fee(task_value, 'observer')
            alignment_bonus = calculate_alignment_bonus(rating, consensus, base_fee)
            total_fee = base_fee + alignment_bonus
            
            # Release bond + fee
            witness_account.release_bond(session_id, total_fee)
    
    def get_or_create_witness_account(self, witness_id):
        """
        Get or create witness account
        """
        if witness_id not in self.witness_accounts:
            self.witness_accounts[witness_id] = WitnessAccount(witness_id)
        return self.witness_accounts[witness_id]

# Usage:
witness_system = AEXWitnessSystem(shared_ledger)

# Select witnesses
witnesses = witness_system.select_witnesses(
    session_id="session_uuid",
    required_count=3,
    min_dex=0.85,
    domains=['finance']
)

# Appoint each witness
for witness_id in witnesses:
    witness_system.appoint_witness(witness_id, "session_uuid")

# Each witness performs attestation
# (witness-specific logic, not shown)

# After all attestations submitted
final_session = witness_system.resolve_with_consensus(
    "session_uuid",
    method='median'
)
```

---

## Security Considerations

### Sybil Resistance

**Attack:** Create multiple witness identities to dominate selection

**Mitigation:**
- Sortition weighted by DEX (Sybil agents have low DEX)
- Bond requirements (expensive to operate multiple witnesses)
- Conflict of interest detection (related identities flagged)

### Collusion

**Attack:** Witnesses coordinate ratings with transaction parties

**Mitigation:**
- Median consensus (resistant to minority collusion)
- Alignment divergence detection (outliers flagged)
- Bond slashing for proven collusion
- Pattern analysis across sessions

### Witness Capture

**Attack:** High-value party "captures" high-DEX witnesses over time

**Mitigation:**
- Conflict of interest checks (frequent interaction detection)
- Randomized selection (sortition prevents targeting)
- Witness rotation enforcement

### Defection

**Attack:** Witness accepts appointment but doesn't deliver

**Mitigation:**
- Bond slashing (50% for first defection)
- Pattern tracking (multiple defections = ban)
- Reputation impact (defection recorded permanently)

---

## Examples

### Example 1: Standard Witnessed Session
```json
{
  "witnesses": [
    {
      "witness_id": "did:aex:Witness1",
      "role": "observer",
      "selection_method": "sortition",
      "bond_amount": 10,
      "bond_status": "released",
      "rating": 0.85,
      "attestation": "Thorough analysis, minor formatting issues",
      "timestamp": "2026-02-04T12:29:00Z",
      "signature": "witness_1_sig"
    },
    {
      "witness_id": "did:aex:Witness2",
      "role": "observer",
      "selection_method": "sortition",
      "bond_amount": 10,
      "bond_status": "released",
      "rating": 0.82,
      "attestation": "Good work, presentation could improve",
      "timestamp": "2026-02-04T12:29:30Z",
      "signature": "witness_2_sig"
    },
    {
      "witness_id": "did:aex:Witness3",
      "role": "observer",
      "selection_method": "sortition",
      "bond_amount": 10,
      "bond_status": "released",
      "rating": 0.86,
      "attestation": "Comprehensive and well-documented",
      "timestamp": "2026-02-04T12:30:00Z",
      "signature": "witness_3_sig"
    }
  ],
  "outcome": {
    "agreed_outcome": 0.85,
    "agreement_method": "witness_consensus_median",
    "witness_ratings": [0.85, 0.82, 0.86],
    "consensus_achieved": true
  }
}
```

### Example 2: Witness with Slashed Bond
```json
{
  "witnesses": [
    {
      "witness_id": "did:aex:HonestWitness",
      "rating": 0.85,
      "bond_status": "released"
    },
    {
      "witness_id": "did:aex:OutlierWitness",
      "rating": 0.30,
      "bond_status": "slashed_25%",
      "slashing_reason": "Extreme outlier (divergence 0.55 from consensus 0.85)",
      "bond_amount": 10,
      "slashed_amount": 2.5
    },
    {
      "witness_id": "did:aex:ReliableWitness",
      "rating": 0.88,
      "bond_status": "released"
    }
  ],
  "outcome": {
    "agreed_outcome": 0.865,
    "agreement_method": "witness_consensus_median",
    "consensus_achieved": true
  }
}
```

---

**Status:** AEX_WITNESS v1.0 complete and normative  
**Next Document:** MATH.md (Mathematical foundations and proofs)
