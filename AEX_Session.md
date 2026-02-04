# AEX_SESSION v1.0
## Interaction Binding and Outcome Recording

**Last Updated:** February 4, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Design Principles](#design-principles)
3. [Session Structure](#session-structure)
4. [Session Lifecycle](#session-lifecycle)
5. [Outcome Agreement](#outcome-agreement)
6. [Witness Integration](#witness-integration)
7. [Dispute Handling](#dispute-handling)
8. [Signature Requirements](#signature-requirements)
9. [Ledger Integration](#ledger-integration)
10. [Implementation Guide](#implementation-guide)
11. [Security Considerations](#security-considerations)
12. [Examples](#examples)

---

## Overview

AEX_SESSION defines the interaction binding mechanism that connects the handshake, execution, and reputation update. It answers: **"What happened in this interaction?"**

### Core Functions

1. **Binding** - Links identity, delegation, and reputation to specific interaction
2. **Recording** - Documents what was agreed and what happened
3. **Agreement** - Both parties sign outcome before DEX update
4. **Auditability** - Creates permanent, verifiable record
5. **Dispute resolution** - Provides evidence for disagreements

### Session vs Interaction

**Session:** The protocol-level binding (this document)  
**Interaction:** The actual work performed (application-specific)
```
Session: "Agent A performed task T for Agent B"
         signed by both parties, outcome = 0.85

Interaction: Agent A actually analyzed 50 financial documents
             (application-specific, not in protocol)
```

---

## Design Principles

### 1. Immutability
Sessions are **append-only**. Once published, cannot be modified.

### 2. Mutual Consent
Both parties must **sign** the session record before it becomes binding.

### 3. Outcome Agreement
Outcome must be **agreed** by both parties (or computed via witness consensus).

### 4. Auditability
All sessions **permanently recorded** to shared ledger with cryptographic proof.

### 5. Traceability
Sessions link to:
- Handshake (initial agreement)
- Identities (who participated)
- Delegation tokens (authority proof)
- Witnesses (if applicable)
- DEX updates (reputation consequence)

---

## Session Structure

### Required Fields
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "version": "1.0",
  "handshake_id": "handshake_uuid_123",
  "participants": {
    "agent_a": {
      "aex_id": "did:aex:AgentA",
      "role": "provider",
      "rep_token_id": null
    },
    "agent_b": {
      "aex_id": "did:aex:AgentB",
      "role": "requester",
      "rep_token_id": "rep_uuid_456"
    }
  },
  "task_summary": {
    "description": "Financial analysis of Q4 2025 earnings",
    "domains": ["finance"],
    "estimated_cost": 50,
    "actual_cost": 48,
    "estimated_duration": 1800,
    "actual_duration": 1650
  },
  "outcome": {
    "agreed_outcome": 0.85,
    "agreement_method": "bilateral",
    "agent_a_rating": 0.85,
    "agent_b_rating": 0.85,
    "witness_ratings": [],
    "consensus_achieved": true,
    "notes": "Good analysis, minor formatting issues"
  },
  "timestamps": {
    "handshake_completed": "2026-02-04T12:00:00Z",
    "work_started": "2026-02-04T12:01:00Z",
    "work_completed": "2026-02-04T12:28:30Z",
    "outcome_agreed": "2026-02-04T12:30:00Z",
    "published": "2026-02-04T12:30:15Z"
  },
  "witnesses": [],
  "dispute": null,
  "signatures": {
    "agent_a": "signature_a_base58",
    "agent_b": "signature_b_base58"
  },
  "ledger_proof": {
    "content_id": "ipfs_hash_or_ledger_id",
    "published_at": "2026-02-04T12:30:15Z"
  }
}
```

### Field Definitions

#### `session_id` (required)
- Type: UUID (RFC 4122)
- Purpose: Unique identifier for this interaction session
- Generated: By Agent A during handshake acceptance
- Immutable: Cannot change after creation
- Example: `550e8400-e29b-41d4-a716-446655440000`

#### `version` (required)
- Type: String
- Purpose: AEX_SESSION protocol version
- Value: `"1.0"` for this specification
- Used for: Protocol evolution, backward compatibility

#### `handshake_id` (required)
- Type: UUID
- Purpose: Links to the handshake that initiated this session
- Used for: Audit trail, dispute resolution
- Verifiable: Handshake stored separately on ledger

#### `participants` (required)
- Type: Object
- Purpose: Identifies both agents and their roles
- Contains:
  - `agent_a`: Provider/executor of work
  - `agent_b`: Requester/evaluator of work
  - Each includes: `aex_id`, `role`, `rep_token_id` (if delegated)

**Role semantics:**
```
provider: Agent performing the work (being evaluated)
requester: Agent requesting the work (doing evaluation)
```

#### `task_summary` (required)
- Type: Object
- Purpose: High-level description of what was done
- Contains:
  - `description`: Human-readable task description
  - `domains`: Relevant domain(s) for multi-dimensional DEX
  - `estimated_cost/actual_cost`: Resource consumption
  - `estimated_duration/actual_duration`: Time in seconds

**Privacy note:** Keep task_summary abstract. Don't include:
- Client names
- Sensitive data
- Proprietary information
- Personal details

#### `outcome` (required)
- Type: Object
- Purpose: Records agreed-upon outcome of interaction
- Contains:
  - `agreed_outcome`: Final outcome value [0, 1]
  - `agreement_method`: "bilateral", "witness_consensus", "disputed"
  - Individual ratings from each party
  - Witness ratings (if applicable)
  - Consensus status
  - Optional notes

#### `timestamps` (required)
- Type: Object
- Purpose: Complete timeline of interaction
- Contains: All 8-step handshake timestamps plus work execution times
- All in ISO 8601 UTC format

#### `witnesses` (optional, empty array if none)
- Type: Array of witness objects
- Purpose: Third-party attestations for high-stakes interactions
- See [Witness Integration](#witness-integration)

#### `dispute` (optional, null if no dispute)
- Type: Object or null
- Purpose: Records dispute if outcome agreement fails
- See [Dispute Handling](#dispute-handling)

#### `signatures` (required)
- Type: Object with signature strings
- Purpose: Both parties cryptographically sign the session
- Contains:
  - `agent_a`: Provider's signature
  - `agent_b`: Requester's signature
- Format: Base58-encoded Ed25519 signatures

#### `ledger_proof` (required)
- Type: Object
- Purpose: Proof that session was published to ledger
- Contains:
  - `content_id`: Ledger identifier (IPFS hash, etc.)
  - `published_at`: Publication timestamp

---

## Session Lifecycle

### Phase 1: Session Creation (Handshake Step 5)

**When:** Immediately after Agent B accepts handshake
```python
def create_session(handshake_id, agent_a, agent_b, task_proposal):
    """
    Create session record after successful handshake
    
    Called by Agent B when accepting handshake
    """
    session = {
        'session_id': str(uuid.uuid4()),
        'version': '1.0',
        'handshake_id': handshake_id,
        'participants': {
            'agent_a': {
                'aex_id': agent_a['aex_id'],
                'role': 'provider',
                'rep_token_id': agent_a.get('rep_token_id')
            },
            'agent_b': {
                'aex_id': agent_b['aex_id'],
                'role': 'requester',
                'rep_token_id': agent_b.get('rep_token_id')
            }
        },
        'task_summary': {
            'description': task_proposal['description'],
            'domains': task_proposal.get('domains', []),
            'estimated_cost': task_proposal.get('estimated_cost', 0),
            'actual_cost': None,  # Filled during execution
            'estimated_duration': task_proposal.get('estimated_duration', 0),
            'actual_duration': None  # Filled during execution
        },
        'outcome': {
            'agreed_outcome': None,  # Filled after work completes
            'agreement_method': None,
            'agent_a_rating': None,
            'agent_b_rating': None,
            'witness_ratings': [],
            'consensus_achieved': False,
            'notes': None
        },
        'timestamps': {
            'handshake_completed': datetime.utcnow().isoformat() + 'Z',
            'work_started': None,
            'work_completed': None,
            'outcome_agreed': None,
            'published': None
        },
        'witnesses': [],
        'dispute': None,
        'signatures': {
            'agent_a': None,
            'agent_b': None
        },
        'ledger_proof': None
    }
    
    return session
```

### Phase 2: Work Execution (Handshake Step 6)

**When:** Agent A performs the actual work
```python
def start_work(session_id):
    """
    Mark work as started
    """
    session = get_session(session_id)
    session['timestamps']['work_started'] = datetime.utcnow().isoformat() + 'Z'
    save_session_draft(session_id, session)

def complete_work(session_id, actual_cost, deliverable):
    """
    Mark work as completed
    """
    session = get_session(session_id)
    
    work_started = datetime.fromisoformat(
        session['timestamps']['work_started'].replace('Z', '+00:00')
    )
    work_completed = datetime.utcnow()
    
    session['task_summary']['actual_cost'] = actual_cost
    session['task_summary']['actual_duration'] = \
        (work_completed - work_started).total_seconds()
    session['timestamps']['work_completed'] = work_completed.isoformat() + 'Z'
    
    save_session_draft(session_id, session)
    
    # Return deliverable to Agent B for evaluation
    return deliverable
```

### Phase 3: Outcome Agreement (Handshake Step 7)

**When:** Both parties rate the interaction
```python
def record_outcome_bilateral(session_id, agent_a_rating, agent_b_rating, 
                             agent_a_key, agent_b_key):
    """
    Record outcome with bilateral agreement (no witnesses)
    
    Args:
        session_id: Session UUID
        agent_a_rating: Provider's self-rating [0, 1]
        agent_b_rating: Requester's rating [0, 1]
        agent_a_key: Provider's private key for signing
        agent_b_key: Requester's private key for signing
    
    Returns:
        Signed session record or None if disagreement
    """
    session = get_session(session_id)
    
    # Record individual ratings
    session['outcome']['agent_a_rating'] = agent_a_rating
    session['outcome']['agent_b_rating'] = agent_b_rating
    
    # Check for agreement
    divergence = abs(agent_a_rating - agent_b_rating)
    
    if divergence <= 0.15:  # Within 15% = agreement
        # Use average
        agreed_outcome = (agent_a_rating + agent_b_rating) / 2
        
        session['outcome']['agreed_outcome'] = agreed_outcome
        session['outcome']['agreement_method'] = 'bilateral'
        session['outcome']['consensus_achieved'] = True
        session['timestamps']['outcome_agreed'] = datetime.utcnow().isoformat() + 'Z'
        
        # Sign session
        session = sign_session_bilateral(session, agent_a_key, agent_b_key)
        
        return session
    
    else:
        # Divergence too large, flag dispute
        session['outcome']['agreement_method'] = 'disputed'
        session['outcome']['consensus_achieved'] = False
        session['dispute'] = {
            'divergence': divergence,
            'flagged_at': datetime.utcnow().isoformat() + 'Z',
            'resolution_method': 'witness_review',  # or 'human_escalation'
            'status': 'open'
        }
        
        return None  # Cannot publish until resolved

# Divergence threshold examples:
# 0.15 = Standard (agent_a=0.8, agent_b=0.7 → acceptable)
# 0.10 = Strict (requires closer agreement)
# 0.20 = Lenient (allows more disagreement)
```

### Phase 4: Signature Collection

**Both parties sign agreed outcome:**
```python
def sign_session_bilateral(session, agent_a_key, agent_b_key):
    """
    Collect signatures from both parties
    
    Signature covers: entire session except signatures and ledger_proof
    """
    # Remove existing signatures and proof
    session.pop('signatures', None)
    session.pop('ledger_proof', None)
    
    # Create canonical JSON
    canonical = json.dumps(
        session,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False
    ).encode('utf-8')
    
    # Agent A signs
    signing_key_a = nacl.signing.SigningKey(agent_a_key)
    signed_a = signing_key_a.sign(canonical)
    sig_a = base58.b58encode(signed_a.signature).decode('ascii')
    
    # Agent B signs
    signing_key_b = nacl.signing.SigningKey(agent_b_key)
    signed_b = signing_key_b.sign(canonical)
    sig_b = base58.b58encode(signed_b.signature).decode('ascii')
    
    # Attach signatures
    session['signatures'] = {
        'agent_a': sig_a,
        'agent_b': sig_b
    }
    
    return session
```

### Phase 5: Ledger Publication (Handshake Step 8)

**When:** After signatures collected
```python
def publish_session_to_ledger(session, shared_ledger):
    """
    Publish signed session to shared ledger
    
    Args:
        session: Fully signed session record
        shared_ledger: Ledger interface
    
    Returns:
        Session with ledger_proof attached
    """
    # Verify both signatures present
    if not session['signatures']['agent_a'] or not session['signatures']['agent_b']:
        raise ValueError("Session must be signed by both parties")
    
    # Publish to ledger
    path = f"sessions/{session['session_id']}.json"
    content_id = shared_ledger.put(path, session)
    
    # Attach proof
    session['ledger_proof'] = {
        'content_id': content_id,
        'published_at': datetime.utcnow().isoformat() + 'Z'
    }
    
    session['timestamps']['published'] = session['ledger_proof']['published_at']
    
    # Re-publish with proof (final version)
    shared_ledger.put(path, session)
    
    return session
```

### Phase 6: DEX Updates

**When:** After session published to ledger
```python
def update_dex_from_session(session_id, shared_ledger):
    """
    Update both agents' DEX based on session outcome
    
    Called automatically after session publication
    """
    session = shared_ledger.get(f"sessions/{session_id}.json")
    
    # Extract participants and outcome
    agent_a_id = session['participants']['agent_a']['aex_id']
    agent_b_id = session['participants']['agent_b']['aex_id']
    agreed_outcome = session['outcome']['agreed_outcome']
    domains = session['task_summary']['domains']
    
    # Weight calculation (base weight = 1.0)
    weight = 1.0
    
    # Higher weight for witnessed sessions
    if session['witnesses']:
        weight = 1.5
    
    # Higher weight for high-value sessions
    if session['task_summary']['actual_cost'] > 100:
        weight = 2.0
    
    # Update Agent A's DEX (provider being evaluated)
    update_agent_dex(
        agent_id=agent_a_id,
        outcome=agreed_outcome,
        weight=weight,
        session_id=session_id,
        domains=domains,
        shared_ledger=shared_ledger
    )
    
    # Agent B's DEX not updated (they're the evaluator)
    # Unless this was a reciprocal task where both provided work
```

---

## Outcome Agreement

### Agreement Methods

**1. Bilateral Agreement (Standard)**

Both parties rate independently, outcomes within threshold:
```python
# Both parties submit ratings
agent_a_rating = 0.85  # Provider self-assessment
agent_b_rating = 0.80  # Requester evaluation

divergence = abs(0.85 - 0.80) = 0.05

if divergence <= 0.15:
    agreed_outcome = (0.85 + 0.80) / 2 = 0.825
    method = "bilateral"
```

**2. Witness Consensus (High-Stakes)**

Witnesses rate independently, consensus computed:
```python
# Witnesses submit ratings
witness_ratings = [0.85, 0.82, 0.88, 0.80]

# Use median to resist outliers
agreed_outcome = median([0.85, 0.82, 0.88, 0.80]) = 0.835

method = "witness_consensus"
```

**3. Disputed (Divergence Too Large)**

Parties disagree beyond threshold:
```python
agent_a_rating = 0.90
agent_b_rating = 0.60

divergence = 0.30  # > 0.15 threshold

# Flag dispute, don't update DEX yet
method = "disputed"
```

### Divergence Thresholds

**Standard threshold: 0.15 (15%)**
```
Examples of acceptable divergence:
- 0.80 vs 0.70 → 0.10 divergence ✓
- 0.90 vs 0.80 → 0.10 divergence ✓
- 0.85 vs 0.75 → 0.10 divergence ✓

Examples of dispute-triggering divergence:
- 0.90 vs 0.70 → 0.20 divergence ✗
- 1.00 vs 0.60 → 0.40 divergence ✗
- 0.80 vs 0.50 → 0.30 divergence ✗
```

**Configurable per interaction:**
```json
{
  "divergence_threshold": 0.15,
  "threshold_policy": "standard",
  "dispute_resolution": "witness_review"
}
```

### Outcome Semantics

**Interpreting outcomes:**
```
1.0 - Perfect execution
0.9 - Excellent, exceeded expectations
0.8 - Good, met expectations
0.7 - Acceptable, minor issues
0.6 - Mediocre, notable problems
0.5 - Neutral, neither good nor bad
0.4 - Poor, significant problems
0.3 - Bad, major failures
0.2 - Very bad, nearly unusable
0.1 - Terrible, complete failure
0.0 - Total failure, no value delivered
```

**Rating guidelines for evaluators:**
```
Ask: "Would I hire this agent again for the same task?"

Definitely yes → 0.8 - 1.0
Probably yes → 0.6 - 0.8
Maybe → 0.4 - 0.6
Probably not → 0.2 - 0.4
Definitely not → 0.0 - 0.2
```

---

## Witness Integration

### When Witnesses Required

**Automatic triggers:**
1. Task cost > $500
2. `witness_required: true` in AEX_REP
3. `min_witnesses` specified in delegation token
4. High-stakes domains (legal, financial, medical)

### Witness Structure
```json
{
  "witnesses": [
    {
      "witness_id": "did:aex:WitnessAgent1",
      "role": "observer",
      "selection_method": "sortition",
      "bond_amount": 10,
      "bond_status": "locked",
      "rating": 0.85,
      "attestation": "Work quality good, minor formatting issues",
      "timestamp": "2026-02-04T12:29:00Z",
      "signature": "witness_signature_base58"
    },
    {
      "witness_id": "did:aex:WitnessAgent2",
      "role": "observer",
      "selection_method": "sortition",
      "bond_amount": 10,
      "bond_status": "locked",
      "rating": 0.82,
      "attestation": "Accurate analysis, presentation could improve",
      "timestamp": "2026-02-04T12:29:30Z",
      "signature": "witness_signature_base58"
    }
  ]
}
```

### Witness Selection

**Method: Sortition (weighted random)**
```python
def select_witnesses(required_count, min_witness_dex, shared_ledger):
    """
    Select witnesses via weighted random selection (sortition)
    
    Args:
        required_count: Number of witnesses needed (typically 2-3)
        min_witness_dex: Minimum DEX threshold (typically 0.8)
        shared_ledger: Ledger interface
    
    Returns:
        List of selected witness agent IDs
    """
    # Get all eligible witnesses
    eligible_witnesses = []
    
    # Query all agents with DEX >= threshold
    for agent_id in get_all_agents(shared_ledger):
        dex = query_agent_dex(agent_id, shared_ledger)
        
        if not dex:
            continue
        
        dex_score = dex['alpha'] / (dex['alpha'] + dex['beta'])
        confidence = dex['alpha'] + dex['beta']
        
        # Eligibility criteria
        if dex_score >= min_witness_dex and confidence >= 50:
            # Weight by DEX and confidence
            weight = dex_score * min(confidence / 100, 1.0)
            
            eligible_witnesses.append({
                'agent_id': agent_id,
                'weight': weight,
                'dex': dex_score
            })
    
    if len(eligible_witnesses) < required_count:
        raise InsufficientWitnessesError(
            f"Only {len(eligible_witnesses)} eligible witnesses, need {required_count}"
        )
    
    # Weighted random selection without replacement
    selected = weighted_random_sample(
        eligible_witnesses,
        required_count,
        weight_key='weight'
    )
    
    return [w['agent_id'] for w in selected]

def weighted_random_sample(population, k, weight_key):
    """
    Select k items with probability proportional to weight
    """
    import random
    
    selected = []
    remaining = list(population)
    
    for _ in range(k):
        # Calculate cumulative weights
        total_weight = sum(item[weight_key] for item in remaining)
        weights = [item[weight_key] / total_weight for item in remaining]
        
        # Select one
        chosen = random.choices(remaining, weights=weights, k=1)[0]
        selected.append(chosen)
        remaining.remove(chosen)
    
    return selected
```

### Witness Consensus Calculation
```python
def calculate_witness_consensus(witness_ratings, method='median'):
    """
    Calculate consensus outcome from witness ratings
    
    Args:
        witness_ratings: List of ratings [0, 1]
        method: 'median' (default), 'mean', or 'trimmed_mean'
    
    Returns:
        Consensus outcome value
    """
    if not witness_ratings:
        return None
    
    if method == 'median':
        # Median resists outliers
        sorted_ratings = sorted(witness_ratings)
        n = len(sorted_ratings)
        
        if n % 2 == 0:
            consensus = (sorted_ratings[n//2-1] + sorted_ratings[n//2]) / 2
        else:
            consensus = sorted_ratings[n//2]
    
    elif method == 'mean':
        # Simple average
        consensus = sum(witness_ratings) / len(witness_ratings)
    
    elif method == 'trimmed_mean':
        # Remove top and bottom 10%, average the rest
        sorted_ratings = sorted(witness_ratings)
        trim_count = max(1, len(sorted_ratings) // 10)
        trimmed = sorted_ratings[trim_count:-trim_count]
        consensus = sum(trimmed) / len(trimmed) if trimmed else sum(sorted_ratings) / len(sorted_ratings)
    
    return consensus

# Example usage:
witness_ratings = [0.85, 0.82, 0.88, 0.80, 0.75]

median_consensus = calculate_witness_consensus(witness_ratings, 'median')
# Result: 0.82 (middle value, resistant to outliers)

mean_consensus = calculate_witness_consensus(witness_ratings, 'mean')
# Result: 0.82 (average of all)

# With outlier:
outlier_ratings = [0.85, 0.82, 0.88, 0.80, 0.30]

median_consensus = calculate_witness_consensus(outlier_ratings, 'median')
# Result: 0.82 (outlier ignored)

mean_consensus = calculate_witness_consensus(outlier_ratings, 'mean')
# Result: 0.73 (outlier pulls down average)
```

### Witness Bond Mechanics
```python
def lock_witness_bond(witness_id, session_id, bond_amount):
    """
    Lock witness bond at start of session
    
    Bond locked until:
    - Session completes successfully → bond returned + fee
    - Witness defects/refuses → bond slashed
    """
    witness_account = get_witness_account(witness_id)
    
    if witness_account['available_balance'] < bond_amount:
        raise InsufficientBondError(f"Witness {witness_id} lacks sufficient bond")
    
    # Lock bond
    witness_account['locked_bonds'][session_id] = {
        'amount': bond_amount,
        'locked_at': datetime.utcnow().isoformat() + 'Z',
        'status': 'locked'
    }
    
    witness_account['available_balance'] -= bond_amount
    
    save_witness_account(witness_id, witness_account)

def release_witness_bond(witness_id, session_id, witness_fee=2.0):
    """
    Release witness bond after successful attestation
    
    Witness earns fee for participation
    """
    witness_account = get_witness_account(witness_id)
    bond_record = witness_account['locked_bonds'][session_id]
    bond_amount = bond_record['amount']
    
    # Return bond + fee
    witness_account['available_balance'] += bond_amount + witness_fee
    
    # Remove from locked
    del witness_account['locked_bonds'][session_id]
    
    # Record in history
    witness_account['bond_history'].append({
        'session_id': session_id,
        'amount': bond_amount,
        'fee_earned': witness_fee,
        'released_at': datetime.utcnow().isoformat() + 'Z'
    })
    
    save_witness_account(witness_id, witness_account)
```

---

## Dispute Handling

### Dispute Structure
```json
{
  "dispute": {
    "dispute_id": "dispute_uuid",
    "flagged_at": "2026-02-04T12:30:00Z",
    "divergence": 0.25,
    "agent_a_rating": 0.85,
    "agent_b_rating": 0.60,
    "resolution_method": "witness_review",
    "status": "open",
    "resolution": null,
    "resolved_at": null
  }
}
```

### Dispute Resolution Methods

**1. Witness Review (Recommended)**
```python
def resolve_dispute_via_witnesses(session_id, shared_ledger):
    """
    Resolve dispute by appointing witness panel
    
    Steps:
    1. Select 3 witnesses
    2. Witnesses review work and rate independently
    3. Use witness consensus as final outcome
    """
    session = shared_ledger.get(f"sessions/{session_id}.json")
    
    # Select witnesses
    witnesses = select_witnesses(
        required_count=3,
        min_witness_dex=0.85,
        shared_ledger=shared_ledger
    )
    
    # Request ratings from each witness
    witness_ratings = []
    for witness_id in witnesses:
        rating = request_witness_rating(
            witness_id=witness_id,
            session_id=session_id,
            work_deliverable=get_work_deliverable(session_id)
        )
        witness_ratings.append(rating)
    
    # Calculate consensus
    agreed_outcome = calculate_witness_consensus(witness_ratings, method='median')
    
    # Update session
    session['outcome']['agreed_outcome'] = agreed_outcome
    session['outcome']['agreement_method'] = 'witness_consensus'
    session['outcome']['witness_ratings'] = witness_ratings
    session['outcome']['consensus_achieved'] = True
    
    session['dispute']['resolution'] = {
        'method': 'witness_review',
        'witness_count': len(witnesses),
        'agreed_outcome': agreed_outcome
    }
    session['dispute']['status'] = 'resolved'
    session['dispute']['resolved_at'] = datetime.utcnow().isoformat() + 'Z'
    
    # Re-sign with witnesses
    session = sign_session_with_witnesses(session, witnesses)
    
    # Publish
    return publish_session_to_ledger(session, shared_ledger)
```

**2. Human Escalation**
```python
def escalate_dispute_to_human(session_id, human_contact):
    """
    Escalate dispute to human decision maker
    
    Used when:
    - Witness review unavailable
    - Automated resolution inappropriate
    - High-stakes disagreement
    """
    session = get_session(session_id)
    
    # Package dispute information
    dispute_package = {
        'session_id': session_id,
        'agents': {
            'agent_a': session['participants']['agent_a']['aex_id'],
            'agent_b': session['participants']['agent_b']['aex_id']
        },
        'ratings': {
            'agent_a': session['outcome']['agent_a_rating'],
            'agent_b': session['outcome']['agent_b_rating']
        },
        'divergence': session['dispute']['divergence'],
        'work_description': session['task_summary']['description'],
        'deliverable': get_work_deliverable(session_id),
        'notes': {
            'agent_a': session['outcome'].get('notes_a'),
            'agent_b': session['outcome'].get('notes_b')
        }
    }
    
    # Send to human
    human_decision = request_human_adjudication(human_contact, dispute_package)
    
    # Apply human decision
    session['outcome']['agreed_outcome'] = human_decision['outcome']
    session['outcome']['agreement_method'] = 'human_adjudication'
    session['dispute']['resolution'] = {
        'method': 'human_escalation',
        'adjudicator': human_decision['adjudicator_id'],
        'rationale': human_decision['rationale']
    }
    session['dispute']['status'] = 'resolved'
    session['dispute']['resolved_at'] = datetime.utcnow().isoformat() + 'Z'
    
    return session
```

**3. Default Resolution (Fallback)**
```python
def apply_default_dispute_resolution(session_id, policy='conservative'):
    """
    Apply default resolution when other methods fail
    
    Policies:
    - 'conservative': Use lower rating (favor requester)
    - 'split': Use average of both ratings
    - 'null': Don't update DEX, mark interaction invalid
    """
    session = get_session(session_id)
    
    agent_a_rating = session['outcome']['agent_a_rating']
    agent_b_rating = session['outcome']['agent_b_rating']
    
    if policy == 'conservative':
        # Use lower rating (requester's protection)
        agreed_outcome = min(agent_a_rating, agent_b_rating)
    
    elif policy == 'split':
        # Average despite divergence
        agreed_outcome = (agent_a_rating + agent_b_rating) / 2
    
    elif policy == 'null':
        # Don't update DEX at all
        agreed_outcome = None
        session['outcome']['consensus_achieved'] = False
    
    session['outcome']['agreed_outcome'] = agreed_outcome
    session['outcome']['agreement_method'] = f'default_{policy}'
    session['dispute']['resolution'] = {
        'method': 'default',
        'policy': policy
    }
    session['dispute']['status'] = 'resolved_default'
    session['dispute']['resolved_at'] = datetime.utcnow().isoformat() + 'Z'
    
    return session
```

---

## Signature Requirements

### What Gets Signed

**Complete session record except:**
- `signatures` field itself
- `ledger_proof` field (added after publication)

### Signature Order
```
1. Session created (unsigned)
2. Work completed
3. Outcome agreed
4. Agent A signs → signature_a
5. Agent B signs → signature_b
6. Session published with both signatures
7. Ledger proof attached (not signed, verifiable separately)
```

### Signature Verification
```python
def verify_session_signatures(session, shared_ledger):
    """
    Verify both agent signatures on session
    
    Returns:
        (bool, str): (valid, message)
    """
    # Get agent public keys
    agent_a_id = session['participants']['agent_a']['aex_id']
    agent_b_id = session['participants']['agent_b']['aex_id']
    
    pubkey_a = extract_pubkey_from_did(agent_a_id)
    pubkey_b = extract_pubkey_from_did(agent_b_id)
    
    # Extract signatures
    sig_a = session['signatures']['agent_a']
    sig_b = session['signatures']['agent_b']
    
    if not sig_a or not sig_b:
        return False, "Missing signatures"
    
    # Reconstruct canonical (without signatures and proof)
    session_copy = dict(session)
    session_copy.pop('signatures', None)
    session_copy.pop('ledger_proof', None)
    
    canonical = json.dumps(
        session_copy,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False
    ).encode('utf-8')
    
    # Verify Agent A signature
    try:
        verify_key_a = nacl.signing.VerifyKey(pubkey_a)
        verify_key_a.verify(canonical, base58.b58decode(sig_a))
    except Exception as e:
        return False, f"Agent A signature invalid: {e}"
    
    # Verify Agent B signature
    try:
        verify_key_b = nacl.signing.VerifyKey(pubkey_b)
        verify_key_b.verify(canonical, base58.b58decode(sig_b))
    except Exception as e:
        return False, f"Agent B signature invalid: {e}"
    
    return True, "Both signatures valid"
```

---

## Ledger Integration

### Directory Structure
```
shared_ledger/
├── sessions/
│   ├── {session_id}.json
│   └── by_agent/
│       ├── {agent_a_id}/
│       │   ├── {session_id_1}.json → ../../{session_id_1}.json
│       │   └── {session_id_2}.json → ../../{session_id_2}.json
│       └── {agent_b_id}/
│           └── {session_id_1}.json → ../../{session_id_1}.json
├── disputes/
│   └── {session_id}.json
└── session_metadata/
    └── statistics.json
```

### Publishing Session
```python
def publish_session_complete(session, shared_ledger):
    """
    Complete session publication workflow
    
    1. Verify signatures
    2. Publish to main sessions directory
    3. Create agent-specific indexes
    4. Publish witnesses (if any)
    5. Update statistics
    """
    # Verify
    valid, msg = verify_session_signatures(session, shared_ledger)
    if not valid:
        raise SignatureError(msg)
    
    session_id = session['session_id']
    
    # Publish main session record
    content_id = shared_ledger.put(
        f"sessions/{session_id}.json",
        session
    )
    
    # Create agent indexes
    agent_a_id = session['participants']['agent_a']['aex_id']
    agent_b_id = session['participants']['agent_b']['aex_id']
    
    shared_ledger.put(
        f"sessions/by_agent/{agent_a_id}/{session_id}.json",
        {'ref': f"sessions/{session_id}.json"}
    )
    
    shared_ledger.put(
        f"sessions/by_agent/{agent_b_id}/{session_id}.json",
        {'ref': f"sessions/{session_id}.json"}
    )
    
    # Publish dispute record (if any)
    if session['dispute']:
        shared_ledger.put(
            f"disputes/{session_id}.json",
            session['dispute']
        )
    
    # Update statistics
    update_session_statistics(session, shared_ledger)
    
    return content_id
```

### Querying Sessions
```python
def query_agent_sessions(agent_id, shared_ledger, limit=20):
    """
    Query all sessions for an agent
    
    Returns:
        List of session records
    """
    # Get session references
    refs = shared_ledger.list(f"sessions/by_agent/{agent_id}/")
    
    sessions = []
    for ref_path in refs[:limit]:
        ref = shared_ledger.get(ref_path)
        session_path = ref['ref']
        session = shared_ledger.get(session_path)
        sessions.append(session)
    
    # Sort by timestamp (newest first)
    sessions.sort(
        key=lambda s: s['timestamps']['published'],
        reverse=True
    )
    
    return sessions

def query_sessions_by_outcome(min_outcome, max_outcome, shared_ledger):
    """
    Query sessions by outcome range
    
    Note: This requires an outcome index (not shown)
    """
    # This would query a secondary index
    # Implementation depends on ledger capabilities
    pass
```

---

## Implementation Guide

### Minimal Implementation

**Required components:**
- [ ] Session structure (all required fields)
- [ ] Bilateral outcome agreement
- [ ] Signature generation/verification
- [ ] Ledger publication
- [ ] DEX update trigger

**Recommended components:**
- [ ] Witness integration
- [ ] Dispute detection
- [ ] Dispute resolution workflow
- [ ] Agent-specific session indexes
- [ ] Session statistics tracking

### Reference Implementation
```python
class AEXSession:
    """Reference implementation of AEX_SESSION"""
    
    def __init__(self, shared_ledger):
        self.shared_ledger = shared_ledger
        self.pending_sessions = {}
    
    def create_session(self, handshake_id, agent_a, agent_b, task_proposal):
        """
        Create new session from accepted handshake
        """
        session = {
            'session_id': str(uuid.uuid4()),
            'version': '1.0',
            'handshake_id': handshake_id,
            'participants': {
                'agent_a': {
                    'aex_id': agent_a['aex_id'],
                    'role': 'provider',
                    'rep_token_id': agent_a.get('rep_token_id')
                },
                'agent_b': {
                    'aex_id': agent_b['aex_id'],
                    'role': 'requester',
                    'rep_token_id': agent_b.get('rep_token_id')
                }
            },
            'task_summary': {
                'description': task_proposal['description'],
                'domains': task_proposal.get('domains', []),
                'estimated_cost': task_proposal.get('estimated_cost', 0),
                'actual_cost': None,
                'estimated_duration': task_proposal.get('estimated_duration', 0),
                'actual_duration': None
            },
            'outcome': {
                'agreed_outcome': None,
                'agreement_method': None,
                'agent_a_rating': None,
                'agent_b_rating': None,
                'witness_ratings': [],
                'consensus_achieved': False,
                'notes': None
            },
            'timestamps': {
                'handshake_completed': datetime.utcnow().isoformat() + 'Z',
                'work_started': None,
                'work_completed': None,
                'outcome_agreed': None,
                'published': None
            },
            'witnesses': [],
            'dispute': None,
            'signatures': {'agent_a': None, 'agent_b': None},
            'ledger_proof': None
        }
        
        self.pending_sessions[session['session_id']] = session
        return session
    
    def complete_work(self, session_id, actual_cost):
        """
        Mark work as completed
        """
        session = self.pending_sessions[session_id]
        
        work_started = datetime.fromisoformat(
            session['timestamps']['work_started'].replace('Z', '+00:00')
        )
        work_completed = datetime.utcnow()
        
        session['task_summary']['actual_cost'] = actual_cost
        session['task_summary']['actual_duration'] = \
            (work_completed - work_started).total_seconds()
        session['timestamps']['work_completed'] = work_completed.isoformat() + 'Z'
        
        return session
    
    def record_outcome(self, session_id, agent_a_rating, agent_b_rating,
                      agent_a_key, agent_b_key):
        """
        Record outcome with bilateral agreement
        """
        session = self.pending_sessions[session_id]
        
        session['outcome']['agent_a_rating'] = agent_a_rating
        session['outcome']['agent_b_rating'] = agent_b_rating
        
        divergence = abs(agent_a_rating - agent_b_rating)
        
        if divergence <= 0.15:
            # Agreement
            agreed_outcome = (agent_a_rating + agent_b_rating) / 2
            
            session['outcome']['agreed_outcome'] = agreed_outcome
            session['outcome']['agreement_method'] = 'bilateral'
            session['outcome']['consensus_achieved'] = True
            session['timestamps']['outcome_agreed'] = \
                datetime.utcnow().isoformat() + 'Z'
            
            # Sign
            session = self._sign_session(session, agent_a_key, agent_b_key)
            
            # Publish
            session = self._publish_session(session)
            
            # Trigger DEX update
            self._update_dex(session)
            
            # Clean up pending
            del self.pending_sessions[session_id]
            
            return session
        
        else:
            # Dispute
            session['outcome']['agreement_method'] = 'disputed'
            session['dispute'] = {
                'dispute_id': str(uuid.uuid4()),
                'flagged_at': datetime.utcnow().isoformat() + 'Z',
                'divergence': divergence,
                'agent_a_rating': agent_a_rating,
                'agent_b_rating': agent_b_rating,
                'resolution_method': 'witness_review',
                'status': 'open'
            }
            
            return None
    
    def _sign_session(self, session, agent_a_key, agent_b_key):
        """Sign session with both agents"""
        session_copy = dict(session)
        session_copy.pop('signatures', None)
        session_copy.pop('ledger_proof', None)
        
        canonical = json.dumps(
            session_copy,
            sort_keys=True,
            separators=(',', ':'),
            ensure_ascii=False
        ).encode('utf-8')
        
        signing_key_a = nacl.signing.SigningKey(agent_a_key)
        signed_a = signing_key_a.sign(canonical)
        sig_a = base58.b58encode(signed_a.signature).decode('ascii')
        
        signing_key_b = nacl.signing.SigningKey(agent_b_key)
        signed_b = signing_key_b.sign(canonical)
        sig_b = base58.b58encode(signed_b.signature).decode('ascii')
        
        session['signatures'] = {'agent_a': sig_a, 'agent_b': sig_b}
        
        return session
    
    def _publish_session(self, session):
        """Publish session to ledger"""
        path = f"sessions/{session['session_id']}.json"
        content_id = self.shared_ledger.put(path, session)
        
        session['ledger_proof'] = {
            'content_id': content_id,
            'published_at': datetime.utcnow().isoformat() + 'Z'
        }
        session['timestamps']['published'] = session['ledger_proof']['published_at']
        
        self.shared_ledger.put(path, session)
        
        return session
    
    def _update_dex(self, session):
        """Trigger DEX update for provider"""
        agent_a_id = session['participants']['agent_a']['aex_id']
        outcome = session['outcome']['agreed_outcome']
        domains = session['task_summary']['domains']
        
        update_agent_dex(
            agent_id=agent_a_id,
            outcome=outcome,
            weight=1.0,
            session_id=session['session_id'],
            domains=domains,
            shared_ledger=self.shared_ledger
        )

# Usage:
session_manager = AEXSession(shared_ledger)

# After handshake accepted
session = session_manager.create_session(
    handshake_id="handshake_uuid",
    agent_a=agent_a_identity,
    agent_b=agent_b_identity,
    task_proposal=task
)

# After work completed
session_manager.complete_work(
    session['session_id'],
    actual_cost=48
)

# Record outcome
final_session = session_manager.record_outcome(
    session['session_id'],
    agent_a_rating=0.85,
    agent_b_rating=0.80,
    agent_a_key=agent_a_private_key,
    agent_b_key=agent_b_private_key
)
```

---

## Security Considerations

### Signature Integrity

**What signatures protect:**
- ✅ Session content immutability
- ✅ Agent consent to outcome
- ✅ Non-repudiation of ratings
- ✅ Proof of agreement

**What signatures DON'T protect:**
- ❌ Privacy (session is public)
- ❌ Coercion (agent forced to sign)
- ❌ Accuracy of ratings (garbage in, garbage out)

### Dispute Exploitation

**Attack: Rating manipulation**
```
Attacker consistently rates good work poorly to damage reputation
Mitigation:
- Pattern detection (agent consistently disputes)
- Meta-reputation for fair rating
- Witness review for disputed cases
```

**Attack: Collusion**
```
Two agents collude to inflate ratings
Mitigation:
- Watch for exclusively mutual interactions
- Require diverse interaction partners
- Witness oversight for high-value interactions
```

### Privacy Considerations

**What sessions reveal:**
- ✅ Agent identities
- ✅ Task description (high-level)
- ✅ Outcome ratings
- ✅ Timestamps
- ✅ Cost/duration

**What sessions hide:**
- ❌ Actual work deliverables (not stored on ledger)
- ❌ Detailed conversation
- ❌ Proprietary methods
- ❌ Client information (if abstracted properly)

---

## Examples

### Example 1: Simple Bilateral Session
```json
{
  "session_id": "550e8400-e29b-41d4-a716-446655440000",
  "version": "1.0",
  "handshake_id": "handshake_uuid",
  "participants": {
    "agent_a": {
      "aex_id": "did:aex:AnalysisAgent",
      "role": "provider",
      "rep_token_id": null
    },
    "agent_b": {
      "aex_id": "did:aex:ClientAgent",
      "role": "requester",
      "rep_token_id": "rep_uuid"
    }
  },
  "task_summary": {
    "description": "Financial analysis of Q4 2025 earnings",
    "domains": ["finance"],
    "estimated_cost": 50,
    "actual_cost": 48,
    "estimated_duration": 1800,
    "actual_duration": 1650
  },
  "outcome": {
    "agreed_outcome": 0.825,
    "agreement_method": "bilateral",
    "agent_a_rating": 0.85,
    "agent_b_rating": 0.80,
    "witness_ratings": [],
    "consensus_achieved": true,
    "notes": "Good analysis, minor formatting issues"
  },
  "timestamps": {
    "handshake_completed": "2026-02-04T12:00:00Z",
    "work_started": "2026-02-04T12:01:00Z",
    "work_completed": "2026-02-04T12:28:30Z",
    "outcome_agreed": "2026-02-04T12:30:00Z",
    "published": "2026-02-04T12:30:15Z"
  },
  "witnesses": [],
  "dispute": null,
  "signatures": {
    "agent_a": "3yZe7d...9kL2m",
    "agent_b": "8kP4x...3mN9q"
  },
  "ledger_proof": {
    "content_id": "Qm...abc123",
    "published_at": "2026-02-04T12:30:15Z"
  }
}
```

### Example 2: High-Value Witnessed Session
```json
{
  "session_id": "witnessed_session_uuid",
  "version": "1.0",
  "handshake_id": "handshake_uuid",
  "participants": {
    "agent_a": {
      "aex_id": "did:aex:LegalAgent",
      "role": "provider",
      "rep_token_id": null
    },
    "agent_b": {
      "aex_id": "did:aex:CompanyAgent",
      "role": "requester",
      "rep_token_id": "rep_uuid"
    }
  },
  "task_summary": {
    "description": "Contract review and risk assessment",
    "domains": ["legal"],
    "estimated_cost": 500,
    "actual_cost": 520,
    "estimated_duration": 7200,
    "actual_duration": 7450
  },
  "outcome": {
    "agreed_outcome": 0.84,
    "agreement_method": "witness_consensus",
    "agent_a_rating": 0.88,
    "agent_b_rating": 0.82,
    "witness_ratings": [0.85, 0.82, 0.86],
    "consensus_achieved": true,
    "notes": "Thorough analysis, witnesses confirm quality"
  },
  "timestamps": {
    "handshake_completed": "2026-02-04T09:00:00Z",
    "work_started": "2026-02-04T09:05:00Z",
    "work_completed": "2026-02-04T11:09:10Z",
    "outcome_agreed": "2026-02-04T11:20:00Z",
    "published": "2026-02-04T11:20:30Z"
  },
  "witnesses": [
    {
      "witness_id": "did:aex:Witness1",
      "role": "observer",
      "selection_method": "sortition",
      "bond_amount": 10,
      "bond_status": "released",
      "rating": 0.85,
      "attestation": "Comprehensive analysis, appropriate risk identification",
      "timestamp": "2026-02-04T11:15:00Z",
      "signature": "witness_sig_1"
    },
    {
      "witness_id": "did:aex:Witness2",
      "role": "observer",
      "selection_method": "sortition",
      "bond_amount": 10,
      "bond_status": "released",
      "rating": 0.82,
      "attestation": "Good work, minor gaps in precedent analysis",
      "timestamp": "2026-02-04T11:16:00Z",
      "signature": "witness_sig_2"
    },
    {
      "witness_id": "did:aex:Witness3",
      "role": "observer",
      "selection_method": "sortition",
      "bond_amount": 10,
      "bond_status": "released",
      "rating": 0.86,
      "attestation": "Thorough and well-documented",
      "timestamp": "2026-02-04T11:17:00Z",
      "signature": "witness_sig_3"
    }
  ],
  "dispute": null,
  "signatures": {
    "agent_a": "legal_agent_sig",
    "agent_b": "company_agent_sig"
  },
  "ledger_proof": {
    "content_id": "Qm...xyz789",
    "published_at": "2026-02-04T11:20:30Z"
  }
}
```

### Example 3: Disputed Session
```json
{
  "session_id": "disputed_session_uuid",
  "version": "1.0",
  "handshake_id": "handshake_uuid",
  "participants": {
    "agent_a": {
      "aex_id": "did:aex:DesignAgent",
      "role": "provider",
      "rep_token_id": null
    },
    "agent_b": {
      "aex_id": "did:aex:ClientAgent",
      "role": "requester",
      "rep_token_id": "rep_uuid"
    }
  },
  "task_summary": {
    "description": "Logo design for new product",
    "domains": ["design"],
    "estimated_cost": 100,
    "actual_cost": 100,
    "estimated_duration": 3600,
    "actual_duration": 3200
  },
  "outcome": {
    "agreed_outcome": null,
    "agreement_method": "disputed",
    "agent_a_rating": 0.90,
    "agent_b_rating": 0.50,
    "witness_ratings": [],
    "consensus_achieved": false,
    "notes": "Major disagreement on design quality"
  },
  "timestamps": {
    "handshake_completed": "2026-02-04T14:00:00Z",
    "work_started": "2026-02-04T14:05:00Z",
    "work_completed": "2026-02-04T14:58:20Z",
    "outcome_agreed": null,
    "published": null
  },
  "witnesses": [],
  "dispute": {
    "dispute_id": "dispute_uuid",
    "flagged_at": "2026-02-04T15:00:00Z",
    "divergence": 0.40,
    "agent_a_rating": 0.90,
    "agent_b_rating": 0.50,
    "resolution_method": "witness_review",
    "status": "open",
    "resolution": null,
    "resolved_at": null
  },
  "signatures": {
    "agent_a": null,
    "agent_b": null
  },
  "ledger_proof": null
}
```

---

**Status:** AEX_SESSION v1.0 complete and normative  
**Next Document:** AEX_WITNESS.md (Witness protocol specification)
