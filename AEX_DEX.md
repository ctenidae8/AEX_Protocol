# AEX_DEX v1.0
## Fork-Aware Behavioral Reputation for Synthetic Agents

**Last Updated:** February 5, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Design Principles](#design-principles)
3. [AEX_DEX Structure](#aex_dex-structure)
4. [Beta Distribution Model](#beta-distribution-model)
5. [Bayesian Update Mechanics](#bayesian-update-mechanics)
6. [Fork-Aware Weighting](#fork-aware-weighting)
7. [Probation System](#probation-system)
8. [Confidence Intervals](#confidence-intervals)
9. [Multi-Dimensional DEX](#multi-dimensional-dex)
10. [Ledger Integration](#ledger-integration)
11. [Query and Evaluation](#query-and-evaluation)
12. [Implementation Guide](#implementation-guide)
13. [Mathematical Properties](#mathematical-properties)
14. [Examples](#examples)

---

## Overview

AEX_DEX defines the behavioral reputation substrate for synthetic agents. It answers the question: **"How reliably does this agent perform?"**

### Core Functions

1. **Behavioral grounding** - Reputation based on observed outcomes, not claims
2. **Uncertainty quantification** - Not just a score, but confidence in that score
3. **Fork awareness** - Adjusts trust when agent architecture changes
4. **Online learning** - Updates incrementally with each interaction
5. **Transparency** - All evidence public and auditable

### What AEX_DEX Measures

✅ **Observable behavior** - Success/failure in real tasks  
✅ **Consistency** - How predictable the agent is  
✅ **Confidence** - How much evidence supports the score  
✅ **Domain expertise** - Optional per-domain reputation  

❌ **NOT intentions** - We don't care why it succeeded/failed  
❌ **NOT capability depth** - Use AEX_HEX for experience and specialization  
❌ **NOT identity** - Use AEX_ID for that  

### Relationship to AEX_HEX

DEX and HEX work together for agent selection:

**DEX answers:** "Is this agent reliable enough?" (binary gate)
- Filters candidates by trust threshold
- Provides confidence in behavioral consistency

**HEX answers:** "Is this agent the right one for this job?" (ranking function)
- Ranks trusted agents by domain experience
- Provides depth and recency of capability

**Joint selection flow:**
1. DEX filters candidates (trust threshold)
2. HEX ranks remaining candidates (capability match)
3. Select top-ranked specialist

See [Comparative Agent Selection](#comparative-agent-selection) for implementation details.  

---

## Design Principles

### 1. Behavioral Grounding
DEX is based **solely on observable outcomes**, not:
- Self-reported capabilities
- Vendor claims
- Architecture type
- Model size
- Marketing materials

### 2. Bayesian Uncertainty
DEX provides:
- **Point estimate** (α / (α + β)) - Expected reliability
- **Uncertainty measure** (n_eff, variance) - Confidence in estimate
- **Confidence intervals** - Probabilistic bounds

### 3. Fork Awareness
When agents fork (update/rewrite), DEX adjusts:
- **Bugfix (1.0)** - Full trust continuity
- **Major (0.5)** - Partial trust continuity  
- **Override (0.1)** - Minimal trust continuity

### 4. Incremental Updates
DEX updates **online** after each interaction:
- No batch recomputation needed
- Scales to millions of interactions
- Real-time reputation tracking

### 5. Transparency
All DEX updates are:
- Published to shared ledger
- Cryptographically signed
- Publicly auditable
- Traceable to interactions

### 6. Complementary to HEX

DEX measures **reliability** (behavioral consistency).  
HEX measures **capability** (domain experience).

Both are needed for effective agent selection:
- High DEX + No relevant HEX → Trustworthy but wrong specialist
- High HEX + Low DEX → Experienced but unreliable
- High DEX + High HEX → Optimal choice

DEX provides the trust gate; HEX provides the capability ranking.

---

## AEX_DEX Structure

### Required Fields
```json
{
  "agent_id": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
  "alpha": 50.0,
  "beta": 10.0,
  "last_updated": "2026-02-04T12:00:00Z",
  "last_session": "session_uuid_123",
  "fork_lineage": [
    {
      "parent_id": "did:aex:ParentAgent",
      "fork_id": "fork_uuid_456",
      "fork_weight": 0.5,
      "inherited_at": "2026-01-15T08:00:00Z",
      "parent_dex": {
        "alpha": 100.0,
        "beta": 20.0
      }
    }
  ],
  "probation": {
    "active": true,
    "fork_type": "major",
    "started_at": "2026-01-15T08:00:00Z",
    "expires_at": "2026-01-29T08:00:00Z",
    "confidence_multiplier": 0.5,
    "successful_tasks": 3,
    "required_tasks": 10
  },
  "metadata": {
    "total_interactions": 85,
    "domains": ["finance", "technology"],
    "created_at": "2025-12-01T10:00:00Z"
  },
  "signature": "agent_signature"
}
```

### Field Definitions

#### `agent_id` (required)
- Type: AEX_ID (DID format)
- Purpose: Which agent this reputation belongs to
- Immutable: Cannot change
- Example: `did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM`

#### `alpha` (required)
- Type: Float ≥ 0
- Purpose: Evidence of **success** (weighted successes accumulated)
- Initial value: 2.0 (neutral prior)
- Update: `α_new = α_old + (outcome × weight × lineage_factor)`

#### `beta` (required)
- Type: Float ≥ 0
- Purpose: Evidence of **failure** (weighted failures accumulated)
- Initial value: 2.0 (neutral prior)
- Update: `β_new = β_old + ((1 - outcome) × weight × lineage_factor)`

#### `last_updated` (required)
- Type: ISO 8601 timestamp (UTC)
- Purpose: When DEX was last modified
- Used for: Caching, staleness detection
- Example: `2026-02-04T12:00:00Z`

#### `last_session` (required)
- Type: UUID
- Purpose: Reference to most recent interaction
- Used for: Audit trails, dispute resolution
- Example: `550e8400-e29b-41d4-a716-446655440000`

#### `fork_lineage` (required, may be empty array)
- Type: Array of fork records
- Purpose: Track reputation inheritance through forks
- Contains: parent_id, fork_id, fork_weight, parent_dex snapshot
- Empty for root identities

#### `probation` (optional, null if not in probation)
- Type: Object or null
- Purpose: Track probation status after fork
- Contains: active flag, expiry, confidence multiplier, task counts
- See [Probation System](#probation-system)

#### `metadata` (optional)
- Type: Object
- Purpose: Non-critical information for context
- Not verified: Informational only
- Can include: interaction counts, domains, creation date

#### `signature` (required)
- Type: String (base58-encoded Ed25519 signature)
- Purpose: Agent signs its own DEX record
- Signed over: Canonical JSON of all fields except signature
- Verification: Uses agent's public key from AEX_ID

---

## Beta Distribution Model

### Mathematical Foundation

AEX_DEX models agent reliability as a **Beta distribution**:
```
Beta(α, β)

Where:
  α = accumulated evidence of success
  β = accumulated evidence of failure
```

### Why Beta Distribution?

1. **Conjugate prior for Bernoulli** - Natural for binary outcomes
2. **Interpretable parameters** - α = successes, β = failures
3. **Supports online learning** - Easy incremental updates
4. **Quantifies uncertainty** - Variance decreases with evidence
5. **Well-studied** - Extensive literature and tooling

### Key Metrics

**DEX Score (Expected Value):**
```
DEX = α / (α + β)

Range: [0, 1]
Interpretation: Expected success probability
```

**Effective Sample Size (Confidence):**
```
n_eff = α + β

Range: [0, ∞)
Interpretation: Total evidence accumulated
```

**Variance (Uncertainty):**
```
Var = (α × β) / ((α + β)² × (α + β + 1))

Range: [0, 0.25]
Interpretation: Lower variance = more confident
```

**Standard Deviation:**
```
σ = √Var
```

### Interpretation Examples

**Example 1: New agent**
```
α = 2, β = 2
DEX = 2 / 4 = 0.50
n_eff = 4
Var = (2×2) / (16×5) = 0.05

Interpretation: Untested, 50% expected success, high uncertainty
```

**Example 2: Proven reliable agent**
```
α = 95, β = 5
DEX = 95 / 100 = 0.95
n_eff = 100
Var = (95×5) / (10000×101) ≈ 0.0005

Interpretation: Highly reliable, 95% expected success, very confident
```

**Example 3: Proven unreliable agent**
```
α = 20, β = 80
DEX = 20 / 100 = 0.20
n_eff = 100
Var = (20×80) / (10000×101) ≈ 0.002

Interpretation: Poor performance, 20% expected success, very confident it's bad
```

**Example 4: Inconsistent agent**
```
α = 50, β = 50
DEX = 50 / 100 = 0.50
n_eff = 100
Var = (50×50) / (10000×101) ≈ 0.0025

Interpretation: Coin flip agent, 50% success, confident it's unreliable
```

---

## Bayesian Update Mechanics

### Basic Update Rule

After each interaction with outcome ∈ [0, 1] and weight ≥ 0:
```python
def update_dex_basic(alpha, beta, outcome, weight):
    """
    Basic Bayesian update (no fork weighting)
    
    Args:
        alpha: Current success evidence
        beta: Current failure evidence
        outcome: Interaction outcome in [0, 1]
        weight: Importance/confidence in this outcome
    
    Returns:
        (new_alpha, new_beta)
    """
    new_alpha = alpha + (outcome * weight)
    new_beta = beta + ((1 - outcome) * weight)
    
    return new_alpha, new_beta

# Example:
alpha, beta = 50.0, 10.0
outcome = 0.85  # Good performance
weight = 1.0    # Standard weight

new_alpha, new_beta = update_dex_basic(alpha, beta, outcome, weight)
# Result: α=50.85, β=10.15

old_dex = alpha / (alpha + beta)  # 0.833
new_dex = new_alpha / (new_alpha + new_beta)  # 0.8337

# DEX improved slightly from 0.833 to 0.8337
```

### Outcome Semantics

**Continuous outcomes [0, 1]:**
```
outcome = 1.0   → Perfect success
outcome = 0.9   → Excellent, minor issues
outcome = 0.8   → Good, some problems
outcome = 0.7   → Acceptable, notable issues
outcome = 0.5   → Neutral, neither good nor bad
outcome = 0.3   → Poor, significant problems
outcome = 0.2   → Bad, major failures
outcome = 0.1   → Very bad, nearly total failure
outcome = 0.0   → Complete failure
```

### Weight Semantics

**Interaction importance:**
```python
# Standard interaction
weight = 1.0

# High-stakes interaction (2x importance)
weight = 2.0

# Low-stakes interaction (half importance)
weight = 0.5

# Critical interaction (5x importance)
weight = 5.0
```

**Weight guidelines:**
- Base weight: 1.0
- High-value tasks: 2.0 - 5.0
- Low-value tasks: 0.5 - 1.0
- Witnessed tasks: 1.5 - 2.0
- Disputed tasks: 0.0 (don't update until resolved)

### HEX Update Coordination

After each DEX update, HEX should also be updated:

```python
def complete_interaction_update(session_outcome, shared_ledger):
    """
    Update both DEX and HEX after interaction
    """
    # 1. Update DEX (behavioral reliability)
    new_dex = update_dex(
        agent_id=session_outcome['agent_id'],
        outcome=session_outcome['outcome'],
        weight=session_outcome['weight'],
        shared_ledger=shared_ledger
    )
    
    # 2. Update HEX (experience corpus)
    if 'domain' in session_outcome:
        hex_corpus = shared_ledger.get_hex(session_outcome['agent_id'])
        
        # Find or create domain experience
        domain_exp = next(
            (e for e in hex_corpus['experience'] 
             if e['domain'] == session_outcome['domain']),
            None
        )
        
        if domain_exp:
            # Increment count
            domain_exp['count'] += 1
            
            # Update confidence based on outcome consistency
            # (Implementation-specific, see AEX_HEX spec)
            domain_exp['confidence'] = calculate_hex_confidence(
                domain_exp,
                session_outcome['outcome']
            )
            
            # Update timestamp
            domain_exp['last_updated'] = now()
        else:
            # New domain
            hex_corpus['experience'].append({
                'domain': session_outcome['domain'],
                'count': 1,
                'confidence': 0.5,  # Neutral start
                'last_updated': now()
            })
        
        # Sign and publish HEX update
        hex_corpus['signature'] = sign(hex_corpus, agent_keypair)
        shared_ledger.publish_hex(hex_corpus)
    
    return new_dex, hex_corpus
```

**Key insight:** DEX and HEX both update from the same interaction, but measure different dimensions:
- **DEX:** "Was this outcome good?" → Reliability signal
- **HEX:** "What domain was this in?" → Capability signal

---

## Fork-Aware Weighting

### Complete Update Rule

**With fork weighting and probation:**
```python
def update_dex_complete(alpha, beta, outcome, weight, fork_weight, probation_active):
    """
    Complete DEX update with fork and probation weighting
    
    Args:
        alpha: Current success evidence
        beta: Current failure evidence
        outcome: Interaction outcome [0, 1]
        weight: Base interaction weight
        fork_weight: Lineage weight from most recent fork [0, 1]
        probation_active: Whether agent is in probation period
    
    Returns:
        (new_alpha, new_beta)
    """
    # Apply probation multiplier
    if probation_active:
        confidence_multiplier = 0.5
    else:
        confidence_multiplier = 1.0
    
    # Calculate effective weight
    effective_weight = weight * fork_weight * confidence_multiplier
    
    # Bayesian update
    new_alpha = alpha + (outcome * effective_weight)
    new_beta = beta + ((1 - outcome) * effective_weight)
    
    return new_alpha, new_beta

# Example: Agent in probation after major fork
alpha, beta = 52.0, 12.0
outcome = 0.9
weight = 1.0
fork_weight = 0.5  # Major fork
probation_active = True

new_alpha, new_beta = update_dex_complete(
    alpha, beta, outcome, weight, fork_weight, probation_active
)

# Effective weight = 1.0 × 0.5 × 0.5 = 0.25
# new_alpha = 52 + (0.9 × 0.25) = 52.225
# new_beta = 12 + (0.1 × 0.25) = 12.025

# DEX barely moved due to probation
old_dex = 52/64 = 0.8125
new_dex = 52.225/64.25 = 0.8129
```

### Fork Weight Calculation
```python
def get_fork_weight(agent_id, shared_ledger):
    """
    Get effective fork weight for current agent state
    
    Returns 1.0 if no fork, otherwise returns weight from most recent fork
    """
    identity = shared_ledger.get(f"identities/{agent_id}/aex_id.json")
    
    if not identity['lineage']['parent_id']:
        return 1.0  # Root identity, no fork
    
    # Get most recent fork event
    fork_history = identity['lineage']['fork_history']
    if not fork_history:
        return 1.0
    
    latest_fork_id = fork_history[-1]
    parent_id = identity['lineage']['parent_id']
    
    fork_event = shared_ledger.get(
        f"identities/{parent_id}/fork_events/{latest_fork_id}.json"
    )
    
    return fork_event['enforced_weight']
```

### Lineage Factor Calculation

**For multi-generation forks:**
```python
def calculate_lineage_factor(agent_id, shared_ledger):
    """
    Calculate cumulative lineage factor across all fork generations
    
    If agent has multiple generations of forks, multiply weights
    """
    identity = shared_ledger.get(f"identities/{agent_id}/aex_id.json")
    
    if not identity['lineage']['parent_id']:
        return 1.0
    
    lineage_factor = 1.0
    current_id = agent_id
    
    # Walk up the lineage tree
    while True:
        identity = shared_ledger.get(f"identities/{current_id}/aex_id.json")
        
        if not identity['lineage']['parent_id']:
            break
        
        # Get fork event
        fork_history = identity['lineage']['fork_history']
        if fork_history:
            latest_fork_id = fork_history[-1]
            parent_id = identity['lineage']['parent_id']
            
            fork_event = shared_ledger.get(
                f"identities/{parent_id}/fork_events/{latest_fork_id}.json"
            )
            
            lineage_factor *= fork_event['enforced_weight']
        
        current_id = identity['lineage']['parent_id']
    
    return lineage_factor

# Example: 3-generation fork chain
# Generation 0 (root): weight = 1.0
# Generation 1 (major fork): weight = 0.5
# Generation 2 (bugfix): weight = 1.0
# Generation 3 (major fork): weight = 0.5
#
# Cumulative lineage factor = 1.0 × 0.5 × 1.0 × 0.5 = 0.25
```

---

## Probation System

### Probation Rules

**Automatic probation after fork:**

| Fork Type | Probation Period | Confidence Multiplier | Required Tasks |
|-----------|------------------|----------------------|----------------|
| Bugfix | 7 days | 0.5 | 10 |
| Major | 14 days | 0.5 | 10 |
| Override | 30 days | 0.5 | 10 |

### Probation Structure
```json
{
  "probation": {
    "active": true,
    "fork_type": "major",
    "started_at": "2026-01-15T08:00:00Z",
    "expires_at": "2026-01-29T08:00:00Z",
    "confidence_multiplier": 0.5,
    "successful_tasks": 3,
    "required_tasks": 10,
    "tasks": [
      {
        "session_id": "uuid_1",
        "outcome": 0.9,
        "timestamp": "2026-01-16T10:00:00Z"
      },
      {
        "session_id": "uuid_2",
        "outcome": 0.85,
        "timestamp": "2026-01-17T14:00:00Z"
      },
      {
        "session_id": "uuid_3",
        "outcome": 0.8,
        "timestamp": "2026-01-18T09:00:00Z"
      }
    ]
  }
}
```

### Probation Exit Conditions

**Exit probation when EITHER:**

1. **Time expires** - Probation period elapses
2. **Task threshold met** - Complete required successful tasks

**"Successful task" definition:**
- outcome ≥ 0.7
- No disputes
- Properly witnessed (if required)

### Probation Checking
```python
def check_probation_status(agent_id, shared_ledger):
    """
    Check if agent is in probation and if it should exit
    
    Returns:
        dict: {
            'active': bool,
            'expires': timestamp or None,
            'progress': dict or None
        }
    """
    dex = shared_ledger.get(f"dex/{agent_id}/current.json")
    probation = dex.get('probation')
    
    if not probation or not probation['active']:
        return {'active': False, 'expires': None, 'progress': None}
    
    now = datetime.utcnow()
    expires = datetime.fromisoformat(probation['expires_at'].replace('Z', '+00:00'))
    
    # Check exit condition 1: Time expired
    if now > expires:
        return {
            'active': False,
            'expires': None,
            'progress': None,
            'exit_reason': 'time_expired'
        }
    
    # Check exit condition 2: Task threshold met
    successful_tasks = probation['successful_tasks']
    required_tasks = probation['required_tasks']
    
    if successful_tasks >= required_tasks:
        return {
            'active': False,
            'expires': None,
            'progress': None,
            'exit_reason': 'tasks_completed'
        }
    
    # Still in probation
    return {
        'active': True,
        'expires': probation['expires_at'],
        'progress': {
            'tasks_completed': successful_tasks,
            'tasks_required': required_tasks,
            'percentage': successful_tasks / required_tasks,
            'days_remaining': (expires - now).days
        }
    }
```

### Probation Update
```python
def update_probation_after_interaction(agent_id, outcome, session_id, shared_ledger):
    """
    Update probation progress after interaction
    """
    dex = shared_ledger.get(f"dex/{agent_id}/current.json")
    probation = dex['probation']
    
    if not probation or not probation['active']:
        return  # Not in probation
    
    # Add task to probation record
    probation['tasks'].append({
        'session_id': session_id,
        'outcome': outcome,
        'timestamp': datetime.utcnow().isoformat() + 'Z'
    })
    
    # If successful (outcome ≥ 0.7), increment counter
    if outcome >= 0.7:
        probation['successful_tasks'] += 1
    
    # Check if should exit probation
    if probation['successful_tasks'] >= probation['required_tasks']:
        probation['active'] = False
        probation['exit_reason'] = 'tasks_completed'
        probation['exited_at'] = datetime.utcnow().isoformat() + 'Z'
    
    # Save updated DEX
    dex['probation'] = probation
    signed_dex = sign_dex(dex, agent_private_key)
    shared_ledger.put(f"dex/{agent_id}/current.json", signed_dex)
```

---

## Confidence Intervals

### Credible Interval Calculation

**95% credible interval using Beta quantiles:**
```python
from scipy.stats import beta as beta_dist

def calculate_confidence_interval(alpha, beta, confidence=0.95):
    """
    Calculate Bayesian credible interval for DEX score
    
    Args:
        alpha: Success evidence
        beta: Failure evidence
        confidence: Confidence level (default 0.95 for 95% CI)
    
    Returns:
        (lower_bound, upper_bound, mean)
    """
    # Calculate percentiles
    lower_percentile = (1 - confidence) / 2
    upper_percentile = 1 - lower_percentile
    
    # Beta distribution quantiles
    lower = beta_dist.ppf(lower_percentile, alpha, beta)
    upper = beta_dist.ppf(upper_percentile, alpha, beta)
    mean = alpha / (alpha + beta)
    
    return lower, upper, mean

# Example 1: High confidence, high DEX
alpha, beta = 95, 5
lower, upper, mean = calculate_confidence_interval(alpha, beta)
# Result: mean=0.95, CI=[0.89, 0.98]
# Narrow interval = high confidence

# Example 2: Low confidence, high DEX
alpha, beta = 10, 2
lower, upper, mean = calculate_confidence_interval(alpha, beta)
# Result: mean=0.83, CI=[0.60, 0.96]
# Wide interval = low confidence

# Example 3: High confidence, low DEX
alpha, beta = 20, 80
lower, upper, mean = calculate_confidence_interval(alpha, beta)
# Result: mean=0.20, CI=[0.14, 0.27]
# Narrow interval = confident it's bad
```

### Confidence Width as Quality Signal
```python
def interpret_confidence_interval(alpha, beta):
    """
    Interpret CI width as trust quality signal
    """
    lower, upper, mean = calculate_confidence_interval(alpha, beta)
    width = upper - lower
    
    if width < 0.1:
        quality = "very_high"
        interpretation = f"DEX={mean:.2f} with very high confidence"
    elif width < 0.2:
        quality = "high"
        interpretation = f"DEX={mean:.2f} with high confidence"
    elif width < 0.3:
        quality = "moderate"
        interpretation = f"DEX={mean:.2f} with moderate confidence"
    else:
        quality = "low"
        interpretation = f"DEX={mean:.2f} with low confidence"
    
    return {
        'mean': mean,
        'lower': lower,
        'upper': upper,
        'width': width,
        'quality': quality,
        'interpretation': interpretation
    }

# Example usage in trust decision:
def should_trust_agent(alpha, beta, min_dex=0.75, min_quality="moderate"):
    """
    Decide whether to trust agent based on DEX and confidence
    """
    ci = interpret_confidence_interval(alpha, beta)
    
    # Check 1: DEX above threshold
    if ci['mean'] < min_dex:
        return False, f"DEX {ci['mean']:.2f} below threshold {min_dex}"
    
    # Check 2: Confidence quality sufficient
    quality_levels = ["low", "moderate", "high", "very_high"]
    if quality_levels.index(ci['quality']) < quality_levels.index(min_quality):
        return False, f"Confidence quality '{ci['quality']}' below required '{min_quality}'"
    
    return True, ci['interpretation']
```

---

## Multi-Dimensional DEX

### Concept

**Single-dimensional DEX:**
```
Agent has one global DEX score = 0.85
```

**Multi-dimensional DEX:**
```
Agent has:
  DEX_finance = 0.92
  DEX_technology = 0.87
  DEX_healthcare = 0.45
  DEX_legal = 0.0 (no experience)
```

### Structure
```json
{
  "agent_id": "did:aex:Agent",
  "global": {
    "alpha": 150.0,
    "beta": 20.0
  },
  "dimensions": {
    "finance": {
      "alpha": 92.0,
      "beta": 8.0
    },
    "technology": {
      "alpha": 52.0,
      "beta": 8.0
    },
    "healthcare": {
      "alpha": 6.0,
      "beta": 4.0
    }
  }
}
```

### Update Logic
```python
def update_multidimensional_dex(dex, outcome, weight, domains):
    """
    Update both global and domain-specific DEX
    
    Args:
        dex: Current DEX object with global and dimensions
        outcome: Interaction outcome
        weight: Interaction weight
        domains: List of domains this interaction belongs to
    
    Returns:
        Updated DEX object
    """
    # Update global DEX
    dex['global']['alpha'] += outcome * weight
    dex['global']['beta'] += (1 - outcome) * weight
    
    # Update each relevant domain
    for domain in domains:
        if domain not in dex['dimensions']:
            # Initialize new domain with neutral prior
            dex['dimensions'][domain] = {'alpha': 2.0, 'beta': 2.0}
        
        dex['dimensions'][domain]['alpha'] += outcome * weight
        dex['dimensions'][domain]['beta'] += (1 - outcome) * weight
    
    return dex

# Example:
dex = {
    'global': {'alpha': 100.0, 'beta': 15.0},
    'dimensions': {
        'finance': {'alpha': 50.0, 'beta': 5.0}
    }
}

outcome = 0.9
weight = 1.0
domains = ['finance', 'technology']

updated_dex = update_multidimensional_dex(dex, outcome, weight, domains)

# Result:
# global: α=100.9, β=15.1
# finance: α=50.9, β=5.1
# technology: α=2.9, β=2.1 (newly created)
```

### Domain Selection Strategy
```python
def select_agent_by_domain(candidates, required_domains, min_dex=0.75):
    """
    Select best agent for task based on domain-specific DEX
    """
    scored_candidates = []
    
    for agent in candidates:
        dex = get_agent_dex(agent['agent_id'])
        
        # Calculate average domain DEX
        domain_scores = []
        for domain in required_domains:
            if domain in dex['dimensions']:
                dim_dex = dex['dimensions'][domain]
                score = dim_dex['alpha'] / (dim_dex['alpha'] + dim_dex['beta'])
                confidence = dim_dex['alpha'] + dim_dex['beta']
                
                # Weight by confidence
                weighted_score = score * min(confidence / 50, 1.0)
                domain_scores.append(weighted_score)
            else:
                # No experience in this domain
                domain_scores.append(0.0)
        
        if not domain_scores:
            avg_domain_dex = 0.0
        else:
            avg_domain_dex = sum(domain_scores) / len(domain_scores)
        
        if avg_domain_dex >= min_dex:
            scored_candidates.append({
                'agent_id': agent['agent_id'],
                'domain_dex': avg_domain_dex,
                'global_dex': dex['global']['alpha'] / (dex['global']['alpha'] + dex['global']['beta'])
            })
    
    # Sort by domain DEX (primary) then global DEX (tiebreaker)
    scored_candidates.sort(
        key=lambda x: (x['domain_dex'], x['global_dex']),
        reverse=True
    )
    
    return scored_candidates
```

---

## Ledger Integration

### Directory Structure
```
shared_ledger/
├── dex/
│   └── {agent_id}/
│       ├── current.json           # Current DEX state
│       ├── history/
│       │   ├── 2026-02-01T12:00:00Z.json
│       │   ├── 2026-02-02T14:30:00Z.json
│       │   └── 2026-02-03T09:15:00Z.json
│       └── metadata.json
```

### Publishing DEX Update
```python
def publish_dex_update(agent_id, new_dex, agent_private_key, shared_ledger):
    """
    Publish DEX update to shared ledger
    """
    # Get current DEX for archival
    current_dex = shared_ledger.get(f"dex/{agent_id}/current.json")
    
    if current_dex:
        # Archive current version
        timestamp = current_dex['last_updated']
        shared_ledger.put(
            path=f"dex/{agent_id}/history/{timestamp}.json",
            data=current_dex
        )
    
    # Sign new DEX
    signed_dex = sign_dex(new_dex, agent_private_key)
    
    # Publish new current DEX
    shared_ledger.put(
        path=f"dex/{agent_id}/current.json",
        data=signed_dex
    )
    
    return signed_dex

def sign_dex(dex, private_key):
    """
    Agent signs its own DEX record
    """
    dex.pop('signature', None)
    
    canonical = json.dumps(
        dex,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False
    ).encode('utf-8')
    
    signing_key = nacl.signing.SigningKey(private_key)
    signed = signing_key.sign(canonical)
    signature_b58 = base58.b58encode(signed.signature).decode('ascii')
    
    dex['signature'] = signature_b58
    return dex
```

### Querying DEX
```python
def query_agent_dex(agent_id, shared_ledger, cache_ttl=3600):
    """
    Query agent DEX with caching
    
    Args:
        agent_id: Agent DID
        shared_ledger: Ledger interface
        cache_ttl: Cache validity in seconds (default 1 hour)
    
    Returns:
        DEX object or None
    """
    # Check local cache
    cache_key = f"dex:{agent_id}"
    cached = local_cache.get(cache_key)
    
    if cached:
        cache_age = (datetime.utcnow() - cached['cached_at']).total_seconds()
        if cache_age < cache_ttl:
            return cached['dex']
    
    # Cache miss or expired, query ledger
    dex = shared_ledger.get(f"dex/{agent_id}/current.json")
    
    if not dex:
        return None
    
    # Verify signature
    identity = shared_ledger.get(f"identities/{agent_id}/aex_id.json")
    public_key = extract_pubkey_from_did(agent_id)
    
    sig_valid = verify_dex_signature(dex, public_key)
    if not sig_valid:
        raise SecurityError(f"Invalid DEX signature for {agent_id}")
    
    # Cache for future queries
    local_cache.set(cache_key, {
        'dex': dex,
        'cached_at': datetime.utcnow()
    })
    
    return dex

def verify_dex_signature(dex, public_key):
    """
    Verify agent's signature on DEX record
    """
    signature_b58 = dex.pop('signature')
    signature = base58.b58decode(signature_b58)
    
    canonical = json.dumps(
        dex,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False
    ).encode('utf-8')
    
    verify_key = nacl.signing.VerifyKey(public_key)
    try:
        verify_key.verify(canonical, signature)
        return True
    except nacl.exceptions.BadSignatureError:
        return False
```

---

## Query and Evaluation

### Trust Decision Framework
```python
def evaluate_agent_trust(agent_id, task_requirements, shared_ledger):
    """
    Complete trust evaluation for agent
    
    Note: This evaluates DEX (reliability) only. For capability matching,
    see HEX-based selection in the Comparative Agent Selection section.
    
    Args:
        agent_id: Agent to evaluate
        task_requirements: Dict with min_dex, min_confidence, domains, etc.
        shared_ledger: Ledger interface
    
    Returns:
        (bool, str, dict): (should_trust, reason, details)
    """
    # Get agent DEX
    dex = query_agent_dex(agent_id, shared_ledger)
    if not dex:
        return False, "No DEX record found", {}
    
    # Calculate metrics
    score = dex['alpha'] / (dex['alpha'] + dex['beta'])
    confidence = dex['alpha'] + dex['beta']
    variance = (dex['alpha'] * dex['beta']) / \
               ((dex['alpha'] + dex['beta'])**2 * (dex['alpha'] + dex['beta'] + 1))
    
    details = {
        'score': score,
        'confidence': confidence,
        'variance': variance,
        'alpha': dex['alpha'],
        'beta': dex['beta']
    }
    
    # Check 1: Minimum score threshold
    if score < task_requirements['min_dex']:
        return False, f"DEX {score:.2f} below threshold {task_requirements['min_dex']}", details
    
    # Check 2: Minimum confidence
    if confidence < task_requirements.get('min_confidence', 20):
        return False, f"Confidence {confidence:.0f} below threshold", details
    
    # Check 3: Probation status
    probation = check_probation_status(agent_id, shared_ledger)
    details['probation'] = probation
    
    if probation['active'] and not task_requirements.get('allow_probation', False):
        return False, "Agent in probation period", details
    
    # Check 4: Domain-specific requirements (if multi-dimensional)
    if 'required_domains' in task_requirements and 'dimensions' in dex:
        for domain in task_requirements['required_domains']:
            if domain not in dex['dimensions']:
                return False, f"No experience in domain '{domain}'", details
            
            domain_dex = dex['dimensions'][domain]
            domain_score = domain_dex['alpha'] / (domain_dex['alpha'] + domain_dex['beta'])
            
            if domain_score < task_requirements.get('min_domain_dex', 0.7):
                return False, f"Domain '{domain}' DEX {domain_score:.2f} too low", details
    
    # Check 5: Confidence interval width
    if task_requirements.get('require_narrow_ci', False):
        lower, upper, mean = calculate_confidence_interval(dex['alpha'], dex['beta'])
        ci_width = upper - lower
        
        if ci_width > 0.2:
            return False, f"Confidence interval too wide ({ci_width:.2f})", details
    
    # All checks passed
    return True, f"Agent trustworthy: DEX={score:.2f}, confidence={confidence:.0f}", details
```

### Comparative Agent Selection

**DEX-only selection (reliability-based):**
```python
def rank_agents_by_dex(candidate_agents, task_requirements, shared_ledger):
    """
    Rank agents by DEX score only (no capability matching)
    
    Use when: Task doesn't require specific domain expertise
    
    Returns:
        List of (agent_id, score, details) sorted by trustworthiness
    """
    ranked = []
    
    for agent_id in candidate_agents:
        trust_decision, reason, details = evaluate_agent_trust(
            agent_id,
            task_requirements,
            shared_ledger
        )
        
        if not trust_decision:
            continue  # Agent doesn't meet minimum requirements
        
        # Calculate composite score
        dex_score = details['score']
        confidence_factor = min(details['confidence'] / 100, 1.0)
        probation_penalty = 0.2 if details['probation']['active'] else 0.0
        
        composite_score = (dex_score * 0.7) + (confidence_factor * 0.3) - probation_penalty
        
        ranked.append({
            'agent_id': agent_id,
            'composite_score': composite_score,
            'dex': dex_score,
            'confidence': details['confidence'],
            'probation': details['probation']['active'],
            'details': details
        })
    
    # Sort by composite score
    ranked.sort(key=lambda x: x['composite_score'], reverse=True)
    
    return ranked
```

**Joint DEX-HEX selection (recommended):**
```python
def select_agent_with_hex(candidate_agents, task_requirements, shared_ledger):
    """
    Select agent using both DEX (trust) and HEX (capability)
    
    Workflow:
    1. Filter by DEX threshold (trust gate)
    2. Filter by HEX domain match (capability gate)
    3. Rank by HEX experience (specialist ranking)
    
    Args:
        task_requirements: Must include 'required_domain' for HEX matching
    
    Returns:
        Selected agent_id or None
    """
    required_domain = task_requirements.get('required_domain')
    if not required_domain:
        raise ValueError("required_domain needed for HEX-based selection")
    
    # Step 1: DEX filter (trust)
    trusted_agents = []
    for agent_id in candidate_agents:
        trust_decision, reason, details = evaluate_agent_trust(
            agent_id,
            task_requirements,
            shared_ledger
        )
        
        if trust_decision:
            trusted_agents.append({
                'agent_id': agent_id,
                'dex_score': details['score'],
                'dex_confidence': details['confidence']
            })
    
    if not trusted_agents:
        return None  # No trustworthy agents
    
    # Step 2: HEX filter (capability)
    capable_agents = []
    for agent in trusted_agents:
        hex_corpus = shared_ledger.get_hex(agent['agent_id'])
        
        if not hex_corpus:
            continue  # No HEX record
        
        # Find domain experience
        domain_exp = None
        for exp in hex_corpus['experience']:
            if exp['domain'] == required_domain:
                domain_exp = exp
                break
        
        if not domain_exp:
            continue  # No experience in required domain
        
        agent['hex_count'] = domain_exp['count']
        agent['hex_confidence'] = domain_exp['confidence']
        agent['hex_recency'] = domain_exp['last_updated']
        capable_agents.append(agent)
    
    if not capable_agents:
        return None  # No capable agents
    
    # Step 3: Rank by HEX experience
    from datetime import datetime
    for agent in capable_agents:
        # Calculate recency weight (exponential decay)
        last_updated = datetime.fromisoformat(agent['hex_recency'].replace('Z', '+00:00'))
        days_since = (datetime.now(last_updated.tzinfo) - last_updated).days
        recency_weight = 0.5 ** (days_since / 180)  # Half-life: 6 months
        
        # Composite HEX score
        agent['hex_score'] = (
            agent['hex_count'] * 
            agent['hex_confidence'] * 
            recency_weight
        )
        
        # Optionally weight with DEX
        agent['composite_score'] = (
            agent['dex_score'] * 0.3 +  # 30% DEX
            agent['hex_score'] * 0.7     # 70% HEX
        )
    
    # Select best specialist
    capable_agents.sort(key=lambda x: x['composite_score'], reverse=True)
    
    return capable_agents[0]['agent_id']


# Example usage:
task = {
    'min_dex': 0.75,
    'min_confidence': 20,
    'required_domain': 'translation.fr_en'
}

selected = select_agent_with_hex(
    candidate_agents=['did:aex:abc', 'did:aex:def', 'did:aex:ghi'],
    task_requirements=task,
    shared_ledger=ledger
)
```

**Decision matrix:**

| DEX Score | HEX Match | Outcome |
|-----------|-----------|---------|
| High | Yes | ✅ **SELECT** - Trustworthy specialist |
| High | No | ❌ Reject - Wrong domain |
| Low | Yes | ❌ Reject - Unreliable |
| Low | No | ❌ Reject - Both failures |

---

## Implementation Guide

### Minimal Implementation

**Required components:**
- [ ] Beta distribution math (α, β calculations)
- [ ] Bayesian update logic
- [ ] Fork weight enforcement
- [ ] Probation tracking
- [ ] Signature generation/verification
- [ ] Ledger publish/query

**Recommended components:**
- [ ] Confidence interval calculation
- [ ] Multi-dimensional DEX support
- [ ] Local DEX cache
- [ ] Trust evaluation framework
- [ ] Agent ranking logic

### Reference Implementation
```python
import nacl.signing
import base58
import json
from datetime import datetime, timedelta
from scipy.stats import beta as beta_dist

class AEXReputation:
    """Reference implementation of AEX_DEX"""
    
    def __init__(self, agent_id, agent_private_key, shared_ledger):
        """
        Initialize reputation manager for agent
        """
        self.agent_id = agent_id
        self.private_key = agent_private_key
        self.shared_ledger = shared_ledger
        self.signing_key = nacl.signing.SigningKey(agent_private_key)
    
    def initialize_dex(self, parent_dex=None, fork_weight=None):
        """
        Initialize DEX for new agent (or after fork)
        """
        if parent_dex and fork_weight:
            # Forked agent: inherit reputation
            alpha = (parent_dex['alpha'] * fork_weight) + 2.0
            beta = (parent_dex['beta'] * fork_weight) + 2.0
        else:
            # Root agent: neutral prior
            alpha = 2.0
            beta = 2.0
        
        dex = {
            'agent_id': self.agent_id,
            'alpha': alpha,
            'beta': beta,
            'last_updated': datetime.utcnow().isoformat() + 'Z',
            'last_session': None,
            'fork_lineage': [],
            'probation': None,
            'metadata': {
                'total_interactions': 0,
                'created_at': datetime.utcnow().isoformat() + 'Z'
            }
        }
        
        return self._publish_dex(dex)
    
    def update_after_interaction(self, outcome, weight, session_id, domains=None):
        """
        Update DEX after interaction
        """
        # Get current DEX
        dex = self.shared_ledger.get(f"dex/{self.agent_id}/current.json")
        
        # Get fork weight
        fork_weight = self._get_fork_weight()
        
        # Check probation
        probation_active = dex.get('probation', {}).get('active', False)
        confidence_multiplier = 0.5 if probation_active else 1.0
        
        # Calculate effective weight
        effective_weight = weight * fork_weight * confidence_multiplier
        
        # Bayesian update
        dex['alpha'] += outcome * effective_weight
        dex['beta'] += (1 - outcome) * effective_weight
        
        # Update metadata
        dex['last_updated'] = datetime.utcnow().isoformat() + 'Z'
        dex['last_session'] = session_id
        dex['metadata']['total_interactions'] += 1
        
        # Update probation if applicable
        if probation_active and outcome >= 0.7:
            dex['probation']['successful_tasks'] += 1
            if dex['probation']['successful_tasks'] >= dex['probation']['required_tasks']:
                dex['probation']['active'] = False
        
        # Update domains (if multi-dimensional)
        if domains and 'dimensions' not in dex:
            dex['dimensions'] = {}
        
        if domains:
            for domain in domains:
                if domain not in dex['dimensions']:
                    dex['dimensions'][domain] = {'alpha': 2.0, 'beta': 2.0}
                
                dex['dimensions'][domain]['alpha'] += outcome * effective_weight
                dex['dimensions'][domain]['beta'] += (1 - outcome) * effective_weight
        
        # Publish update
        return self._publish_dex(dex)
    
    def get_dex_score(self):
        """Get current DEX score"""
        dex = self.shared_ledger.get(f"dex/{self.agent_id}/current.json")
        return dex['alpha'] / (dex['alpha'] + dex['beta'])
    
    def get_confidence_interval(self, confidence=0.95):
        """Get Bayesian credible interval"""
        dex = self.shared_ledger.get(f"dex/{self.agent_id}/current.json")
        
        lower_pct = (1 - confidence) / 2
        upper_pct = 1 - lower_pct
        
        lower = beta_dist.ppf(lower_pct, dex['alpha'], dex['beta'])
        upper = beta_dist.ppf(upper_pct, dex['alpha'], dex['beta'])
        mean = dex['alpha'] / (dex['alpha'] + dex['beta'])
        
        return {'lower': lower, 'upper': upper, 'mean': mean}
    
    def _get_fork_weight(self):
        """Get effective fork weight"""
        identity = self.shared_ledger.get(f"identities/{self.agent_id}/aex_id.json")
        
        if not identity['lineage']['parent_id']:
            return 1.0
        
        fork_history = identity['lineage']['fork_history']
        if not fork_history:
            return 1.0
        
        latest_fork_id = fork_history[-1]
        parent_id = identity['lineage']['parent_id']
        
        fork_event = self.shared_ledger.get(
            f"identities/{parent_id}/fork_events/{latest_fork_id}.json"
        )
        
        return fork_event['enforced_weight']
    
    def _publish_dex(self, dex):
        """Sign and publish DEX to ledger"""
        # Archive current version
        current = self.shared_ledger.get(f"dex/{self.agent_id}/current.json")
        if current:
            self.shared_ledger.put(
                f"dex/{self.agent_id}/history/{current['last_updated']}.json",
                current
            )
        
        # Sign new version
        dex.pop('signature', None)
        canonical = json.dumps(
            dex,
            sort_keys=True,
            separators=(',', ':'),
            ensure_ascii=False
        ).encode('utf-8')
        
        signed = self.signing_key.sign(canonical)
        dex['signature'] = base58.b58encode(signed.signature).decode('ascii')
        
        # Publish
        self.shared_ledger.put(
            f"dex/{self.agent_id}/current.json",
            dex
        )
        
        return dex

# Usage example:
reputation = AEXReputation(
    agent_id="did:aex:Agent123",
    agent_private_key=private_key,
    shared_ledger=ledger_interface
)

# Initialize
reputation.initialize_dex()

# After interaction
reputation.update_after_interaction(
    outcome=0.85,
    weight=1.0,
    session_id="session_uuid",
    domains=['finance']
)

# Query
score = reputation.get_dex_score()
ci = reputation.get_confidence_interval()
print(f"DEX: {score:.2f}, CI: [{ci['lower']:.2f}, {ci['upper']:.2f}]")
```

---

## Mathematical Properties

### Convergence

**DEX converges to true success rate:**
```
As n → ∞, DEX → p
Where p is the agent's true reliability
```

**Proof sketch:**
- Beta distribution is conjugate prior for Bernoulli
- Posterior mean converges to maximum likelihood estimate
- By law of large numbers, MLE converges to true parameter

### Uncertainty Reduction

**Variance decreases with evidence:**
```
Var(DEX) = (α × β) / ((α + β)² × (α + β + 1))

As α + β → ∞, Var → 0
```

**Rate of uncertainty reduction:**
```python
def variance_trajectory(n_interactions):
    """
    Show how variance decreases with interactions
    Assuming 80% success rate
    """
    variances = []
    for n in range(2, n_interactions+1):
        alpha = 2 + (0.8 * n)
        beta = 2 + (0.2 * n)
        var = (alpha * beta) / ((alpha + beta)**2 * (alpha + beta + 1))
        variances.append(var)
    return variances

# Example:
# n=10: var≈0.015
# n=50: var≈0.003
# n=100: var≈0.0015
# n=500: var≈0.0003
```

### Fork Impact

**DEX change after fork:**
```python
def calculate_fork_impact(parent_alpha, parent_beta, fork_weight):
    """
    Calculate DEX before and after fork
    """
    # Parent DEX
    parent_dex = parent_alpha / (parent_alpha + parent_beta)
    
    # Child DEX (with neutral prior added)
    child_alpha = (parent_alpha * fork_weight) + 2
    child_beta = (parent_beta * fork_weight) + 2
    child_dex = child_alpha / (child_alpha + child_beta)
    
    # Impact
    dex_change = child_dex - parent_dex
    confidence_change = (child_alpha + child_beta) - (parent_alpha + parent_beta)
    
    return {
        'parent_dex': parent_dex,
        'child_dex': child_dex,
        'dex_change': dex_change,
        'confidence_change': confidence_change
    }

# Example: Major fork of high-reputation agent
impact = calculate_fork_impact(
    parent_alpha=95,
    parent_beta=5,
    fork_weight=0.5
)
# Result:
# parent_dex: 0.95
# child_dex: 0.923
# dex_change: -0.027 (slight decrease)
# confidence_change: -48 (significant confidence loss)
```

---

## Examples

### Example 1: New Agent Bootstrap
```json
{
  "agent_id": "did:aex:NewAgent",
  "alpha": 2.0,
  "beta": 2.0,
  "last_updated": "2026-02-04T12:00:00Z",
  "last_session": null,
  "fork_lineage": [],
  "probation": null,
  "metadata": {
    "total_interactions": 0,
    "created_at": "2026-02-04T12:00:00Z"
  }
}

// DEX = 2/4 = 0.50
// Confidence = 4 (very low)
// Interpretation: Untested, neutral expectation
```

### Example 2: After 10 Good Interactions
```json
{
  "agent_id": "did:aex:NewAgent",
  "alpha": 10.5,
  "beta": 2.5,
  "last_updated": "2026-02-05T14:00:00Z",
  "last_session": "session_uuid_10",
  "fork_lineage": [],
  "probation": null,
  "metadata": {
    "total_interactions": 10,
    "created_at": "2026-02-04T12:00:00Z"
  }
}

// 10 interactions, average outcome = 0.85
// Δα = 10 × 0.85 = 8.5
// Δβ = 10 × 0.15 = 1.5
// New: α=10.5, β=3.5
// DEX = 10.5/14 = 0.75
// Confidence = 14
// Interpretation: Showing promise, needs more evidence
```

### Example 3: Established Reliable Agent
```json
{
  "agent_id": "did:aex:ReliableAgent",
  "alpha": 187.0,
  "beta": 18.0,
  "last_updated": "2026-02-04T15:30:00Z",
  "last_session": "session_uuid_200",
  "fork_lineage": [],
  "probation": null,
  "metadata": {
    "total_interactions": 200,
    "created_at": "2025-08-01T10:00:00Z"
  }
}

// DEX = 187/205 = 0.912
// Confidence = 205 (very high)
// 95% CI: [0.87, 0.94]
// Interpretation: Highly reliable, well-tested
```

### Example 4: Agent After Major Fork
```json
{
  "agent_id": "did:aex:ForkedAgent",
  "alpha": 95.5,
  "beta": 12.0,
  "last_updated": "2026-02-04T16:00:00Z",
  "last_session": "session_after_fork_5",
  "fork_lineage": [
    {
      "parent_id": "did:aex:ParentAgent",
      "fork_id": "fork_uuid",
      "fork_weight": 0.5,
      "inherited_at": "2026-01-20T08:00:00Z",
      "parent_dex": {
        "alpha": 187.0,
        "beta": 20.0
      }
    }
  ],
  "probation": {
    "active": true,
    "fork_type": "major",
    "started_at": "2026-01-20T08:00:00Z",
    "expires_at": "2026-02-03T08:00:00Z",
    "confidence_multiplier": 0.5,
    "successful_tasks": 5,
    "required_tasks": 10
  },
  "metadata": {
    "total_interactions": 5,
    "created_at": "2026-01-20T08:00:00Z"
  }
}

// Parent: α=187, β=20 (DEX=0.90)
// Fork weight: 0.5
// Initial child: α=95.5, β=12 (DEX=0.888)
// In probation: 5/10 tasks completed
// Interpretation: Inherited good reputation, re-proving reliability
```

### Example 5: Multi-Dimensional DEX
```json
{
  "agent_id": "did:aex:SpecialistAgent",
  "alpha": 150.0,
  "beta": 25.0,
  "dimensions": {
    "finance": {
      "alpha": 95.0,
      "beta": 5.0
    },
    "technology": {
      "alpha": 45.0,
      "beta": 15.0
    },
    "healthcare": {
      "alpha": 10.0,
      "beta": 5.0
    }
  },
  "last_updated": "2026-02-04T17:00:00Z",
  "last_session": "session_uuid_175",
  "fork_lineage": [],
  "probation": null
}

// Global DEX: 150/175 = 0.857
// Finance DEX: 95/100 = 0.950 (high confidence)
// Technology DEX: 45/60 = 0.750 (moderate confidence)
// Healthcare DEX: 10/15 = 0.667 (low confidence)
// Interpretation: Finance specialist, competent in tech, learning healthcare
```

### Example 6: Joint DEX-HEX Selection

**Scenario:** Need French→English translator

**Candidate pool:**
```python
candidates = [
    {
        'id': 'agent_A',
        'dex': {'alpha': 85, 'beta': 15, 'score': 0.85},
        'hex': {'translation.fr_en': {'count': 120, 'confidence': 0.92}}
    },
    {
        'id': 'agent_B',
        'dex': {'alpha': 90, 'beta': 10, 'score': 0.90},
        'hex': {'translation.es_en': {'count': 200, 'confidence': 0.95}}
    },
    {
        'id': 'agent_C',
        'dex': {'alpha': 80, 'beta': 20, 'score': 0.80},
        'hex': {'translation.fr_en': {'count': 50, 'confidence': 0.85}}
    },
    {
        'id': 'agent_D',
        'dex': {'alpha': 70, 'beta': 30, 'score': 0.70},
        'hex': {'translation.fr_en': {'count': 300, 'confidence': 0.97}}
    }
]
```

**Selection logic:**

```
Step 1: DEX filter (min_dex = 0.75)
✅ Agent A: 0.85 (passes)
✅ Agent B: 0.90 (passes)
✅ Agent C: 0.80 (passes)
❌ Agent D: 0.70 (below threshold, rejected despite high HEX)

Step 2: HEX domain filter (required: translation.fr_en)
✅ Agent A: Has fr_en experience
❌ Agent B: Only has es_en (wrong language pair, rejected)
✅ Agent C: Has fr_en experience

Step 3: HEX ranking (count × confidence)
Agent A: 120 × 0.92 = 110.4
Agent C:  50 × 0.85 =  42.5

Result: Select Agent A
```

**Key insights:**
- Agent B has highest DEX (0.90) but wrong domain → rejected by HEX filter
- Agent D has best HEX (300 tasks, 0.97 confidence) but too low DEX → rejected by trust gate
- Agent A balances both dimensions → selected as optimal specialist

**Code:**
```python
result = select_agent_with_hex(
    candidate_agents=['agent_A', 'agent_B', 'agent_C', 'agent_D'],
    task_requirements={
        'min_dex': 0.75,
        'required_domain': 'translation.fr_en'
    },
    shared_ledger=ledger
)
# Returns: 'agent_A'
```

**Why both dimensions matter:**
- **DEX alone:** Would select Agent B (highest reliability, wrong skill)
- **HEX alone:** Would select Agent D (most experienced, too unreliable)
- **DEX + HEX:** Selects Agent A (trustworthy AND capable)

---

## Related Specifications

AEX_DEX works in conjunction with:

**AEX_HEX** - Experience corpus and capability signaling
- HEX provides domain-specific experience depth
- DEX provides general behavioral reliability
- Used together for optimal agent selection
- See [Comparative Agent Selection](#comparative-agent-selection) for joint implementation

**AEX_ID** - Persistent identity
- DEX is bound to agent identity
- Fork events trigger DEX inheritance
- HEX inheritance follows DEX fork weights

**AEX_REP** - Delegation and authority
- DEX applies to delegated agents
- Both principal and agent DEX matter
- HEX of either may be relevant

**AEX_Session** - Interaction records
- Sessions feed both DEX and HEX updates
- Outcome drives DEX Bayesian update
- Domain drives HEX experience increment

---

**Status:** AEX_DEX v1.0 complete and normative  
**Next Document:** AEX_SESSION.md (Interaction binding specification)
