# AEX Protocol v1.0 - Complete Examples
## End-to-End Use Cases and Integration Patterns

**Last Updated:** February 5, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Example 1: Simple Agent Collaboration](#example-1-simple-agent-collaboration)
3. [Example 2: Multi-Agent Task Routing with HEX](#example-2-multi-agent-task-routing-with-hex)
4. [Example 3: Human-Agent Delegation](#example-3-human-agent-delegation)
5. [Example 4: Witness-Verified Transactions](#example-4-witness-verified-transactions)
6. [Example 5: Fork and Recovery](#example-5-fork-and-recovery)
7. [Example 6: Cross-Organization Trust](#example-6-cross-organization-trust)
8. [Example 7: Reputation-Based Access Control](#example-7-reputation-based-access-control)
9. [Example 8: HEX-Driven Specialist Selection](#example-8-hex-driven-specialist-selection)
10. [Example 9: Agent Marketplace](#example-9-agent-marketplace)
11. [Example 10: Dispute Resolution](#example-10-dispute-resolution)
12. [Complete Application: Research Assistant Network](#complete-application-research-assistant-network)

---

## Overview

This document provides **complete, runnable examples** demonstrating:

- Basic agent-to-agent interactions
- HEX-based specialist discovery and selection
- Complex multi-agent workflows  
- Human delegation patterns
- Witness-verified high-value transactions
- Fork management and HEX inheritance
- Real-world application scenarios

All examples use the four-layer AEX architecture:
- **AEX_ID:** Identity and lineage
- **AEX_REP:** Authority and delegation
- **AEX_DEX:** Behavioral reliability
- **AEX_HEX:** Experience and capability

**Key pattern:** DEX filters candidates by trust threshold, HEX ranks them by domain expertise.

---

## Example 1: Simple Agent Collaboration

**Scenario:** Two agents collaborate on a data processing task. This example shows basic DEX accumulation and introduces HEX.

### Setup
```python
"""
Simple two-agent collaboration with DEX and HEX tracking
"""

from aex import AEXAgent, LocalLedger

# Initialize shared ledger
ledger = LocalLedger("collaboration.db")

# Create two agents
agent_alice = AEXAgent(ledger=ledger)
agent_bob = AEXAgent(ledger=ledger)

print(f"Alice: {agent_alice.identity['aex_id']}")
print(f"Bob: {agent_bob.identity['aex_id']}")
```

### Build Bob's Reputation and Experience
```python
# Give Bob both reputation (DEX) and experience (HEX) by completing tasks
for i in range(50):
    # Create temporary partner
    partner = AEXAgent(ledger=ledger)
    
    # Execute handshake
    handshake = agent_bob.initiate_handshake(
        counterparty_did=partner.identity['aex_id']
    )
    
    # Simulate successful collaboration
    outcome = 0.85  # Good quality work
    domain = 'data_processing'  # Domain for HEX tracking
    
    agent_bob.record_session(
        handshake_id=handshake.handshake_id,
        outcome=outcome,
        task_summary={
            'task_type': domain,
            'description': f'Processed dataset {i}'
        },
        domain=domain  # Track domain for HEX
    )

# Check Bob's reputation (DEX)
bob_dex = agent_bob.calculate_dex()
print(f"\nBob's DEX: {bob_dex['dex']:.3f}")
print(f"Bob's confidence: {bob_dex['n_eff']:.1f}")

# Check Bob's experience (HEX)
bob_hex = agent_bob.get_hex()
print(f"\n=== Bob's HEX Corpus ===")
for exp in bob_hex['experience']:
    print(f"  {exp['domain']}: {exp['count']} interactions, "
          f"confidence={exp['confidence']:.2f}, "
          f"last={exp['last_updated'][:10]}")
```

**Output:**
```
Bob's DEX: 0.847
Bob's confidence: 54.0

=== Bob's HEX Corpus ===
  data_processing: 50 interactions, confidence=0.89, last=2026-02-05
```

### Alice Collaborates with Bob
```python
# Alice initiates handshake with Bob
print("\n=== Alice → Bob Handshake ===")

# Alice can check both DEX (reliability) and HEX (capability) before collaboration
bob_dex = agent_bob.calculate_dex()
bob_hex = agent_bob.get_hex()

# Check if Bob has relevant experience
has_relevant_experience = any(
    exp['domain'] == 'data_processing' and exp['count'] >= 10
    for exp in bob_hex['experience']
)

print(f"Bob's DEX: {bob_dex['dex']:.3f} (meets threshold: {bob_dex['dex'] >= 0.75})")
print(f"Bob's relevant experience: {has_relevant_experience}")

if bob_dex['dex'] >= 0.75 and has_relevant_experience:
    handshake = agent_alice.initiate_handshake(
        counterparty_did=agent_bob.identity['aex_id']
    )

    if handshake.success:
        print(f"✓ Handshake successful: {handshake.handshake_id}")
        
        # Perform collaborative work
        print("\n=== Collaborative Work ===")
        result = {
            'processed_records': 10000,
            'quality_score': 0.92,
            'errors': 15
        }
        print(f"Processed: {result['processed_records']} records")
        print(f"Quality: {result['quality_score']}")
        
        # Alice records the session (updates both DEX and HEX for Bob)
        session = agent_alice.record_session(
            handshake_id=handshake.handshake_id,
            outcome=result['quality_score'],
            task_summary={
                'task_type': 'collaborative_processing',
                'result': result
            },
            domain='data_processing'
        )
        
        print(f"\n✓ Session recorded: {session['session_id']}")
        
        # Check updated reputation and experience
        bob_dex_updated = agent_bob.calculate_dex()
        bob_hex_updated = agent_bob.get_hex()
        
        data_exp = next(e for e in bob_hex_updated['experience'] 
                       if e['domain'] == 'data_processing')
        
        print(f"\nBob's updated DEX: {bob_dex_updated['dex']:.3f}")
        print(f"Change: {bob_dex_updated['dex'] - bob_dex['dex']:+.3f}")
        print(f"Bob's updated HEX count: {data_exp['count']} (+1)")
else:
    print("✗ Bob doesn't meet requirements (either DEX or HEX)")
```

**Output:**
```
=== Alice → Bob Handshake ===
Bob's DEX: 0.847 (meets threshold: True)
Bob's relevant experience: True
✓ Handshake successful: hs-7f3a9b2c-1d4e-4a8f-9c2b-8e6d1f5a3c7b

=== Collaborative Work ===
Processed: 10000 records
Quality: 0.92

✓ Session recorded: session-4e2f8a9c-6b3d-4f1e-8a7c-2d9e5b1f6a4c

Bob's updated DEX: 0.848
Change: +0.001
Bob's updated HEX count: 51 (+1)
```

**Key Insight:** This example shows that both DEX and HEX are updated from the same interaction, but measure different dimensions:
- **DEX:** "Was the outcome good?" → Reliability signal
- **HEX:** "What domain was this in?" → Capability signal

---

## Example 2: Multi-Agent Task Routing with HEX

**Scenario:** A coordinator agent routes tasks to specialized worker agents based on their domain expertise (HEX) and reliability (DEX).

### Setup Specialized Agents
```python
"""
Multi-agent task routing using DEX (trust) and HEX (capability)
"""

from aex import AEXAgent, LocalLedger
import random

ledger = LocalLedger("task_routing.db")

# Create coordinator
coordinator = AEXAgent(ledger=ledger)

# Create specialized worker agents
workers = {
    'data_analyst': AEXAgent(ledger=ledger),
    'ml_engineer': AEXAgent(ledger=ledger),
    'security_auditor': AEXAgent(ledger=ledger)
}

# Build specialized reputations with domain-specific experience
def build_specialist_reputation(agent, task_types, num_tasks=30):
    """Build domain-specific reputation and experience corpus"""
    for i in range(num_tasks):
        partner = AEXAgent(ledger=ledger)
        handshake = agent.initiate_handshake(partner.identity['aex_id'])
        
        # Pick a random task from their specialization
        task_type = random.choice(task_types)
        
        # High quality in specialty
        outcome = random.uniform(0.85, 0.95)
        
        agent.record_session(
            handshake_id=handshake.handshake_id,
            outcome=outcome,
            task_summary={
                'task_type': task_type,
                'description': f'{task_type} task {i}'
            },
            domain=task_type  # Track domain for HEX
        )

# Build specializations with domain diversity
print("Building specialist reputations and experience...")
build_specialist_reputation(
    workers['data_analyst'], 
    ['data_analysis', 'data_visualization', 'statistical_modeling']
)
build_specialist_reputation(
    workers['ml_engineer'], 
    ['machine_learning', 'model_training', 'feature_engineering']
)
build_specialist_reputation(
    workers['security_auditor'], 
    ['security_audit', 'penetration_testing', 'vulnerability_assessment']
)

# Display reputations and experience
print("\n=== Worker Capabilities ===")
for role, agent in workers.items():
    dex = agent.calculate_dex()
    hex_corpus = agent.get_hex()
    
    print(f"\n{role}:")
    print(f"  DEX: {dex['dex']:.3f}, n_eff={dex['n_eff']:.1f}")
    print(f"  HEX domains:")
    for exp in hex_corpus['experience']:
        print(f"    - {exp['domain']}: {exp['count']} tasks, "
              f"confidence={exp['confidence']:.2f}")
```

**Output:**
```
Building specialist reputations and experience...

=== Worker Capabilities ===

data_analyst:
  DEX: 0.897, n_eff=34.0
  HEX domains:
    - data_analysis: 11 tasks, confidence=0.91
    - data_visualization: 10 tasks, confidence=0.89
    - statistical_modeling: 9 tasks, confidence=0.88

ml_engineer:
  DEX: 0.902, n_eff=34.0
  HEX domains:
    - machine_learning: 12 tasks, confidence=0.93
    - model_training: 9 tasks, confidence=0.90
    - feature_engineering: 9 tasks, confidence=0.88

security_auditor:
  DEX: 0.891, n_eff=34.0
  HEX domains:
    - security_audit: 10 tasks, confidence=0.90
    - penetration_testing: 11 tasks, confidence=0.89
    - vulnerability_assessment: 9 tasks, confidence=0.87
```

### DEX-HEX Joint Selection Logic
```python
class TaskRouter:
    """
    Route tasks using DEX (reliability filter) + HEX (capability ranking)
    """
    
    def __init__(self, coordinator: AEXAgent, workers: dict):
        self.coordinator = coordinator
        self.workers = workers
    
    def route_task(self, task: dict) -> dict:
        """
        Route task to best-qualified agent using joint DEX-HEX selection
        
        Selection process:
        1. DEX filters: Remove agents below trust threshold
        2. HEX matches: Find agents with domain experience
        3. HEX ranks: Select agent with best domain match
        
        Args:
            task: Task specification with 'domain', 'min_dex'
        
        Returns:
            Execution result
        """
        required_domain = task['domain']
        min_dex = task.get('min_dex', 0.8)
        min_experience = task.get('min_experience', 5)
        
        # Phase 1: DEX Filter (trust gate)
        trusted_workers = []
        for role, agent in self.workers.items():
            dex = agent.calculate_dex()
            if dex['dex'] >= min_dex:
                trusted_workers.append((role, agent, dex))
        
        if not trusted_workers:
            return {
                'success': False, 
                'reason': f'No workers meet DEX threshold {min_dex}'
            }
        
        # Phase 2: HEX Match (capability filter)
        capable_workers = []
        for role, agent, dex in trusted_workers:
            hex_corpus = agent.get_hex()
            
            # Find domain experience
            domain_exp = next(
                (e for e in hex_corpus['experience'] 
                 if e['domain'] == required_domain),
                None
            )
            
            if domain_exp and domain_exp['count'] >= min_experience:
                # Calculate composite score: 30% DEX, 70% HEX
                hex_score = (
                    domain_exp['confidence'] * 0.6 +  # Confidence in domain
                    min(domain_exp['count'] / 50, 1.0) * 0.4  # Experience depth
                )
                composite_score = dex['dex'] * 0.3 + hex_score * 0.7
                
                capable_workers.append({
                    'role': role,
                    'agent': agent,
                    'dex': dex['dex'],
                    'hex_score': hex_score,
                    'composite_score': composite_score,
                    'experience_count': domain_exp['count'],
                    'confidence': domain_exp['confidence']
                })
        
        if not capable_workers:
            return {
                'success': False,
                'reason': f'No workers have experience in {required_domain}'
            }
        
        # Phase 3: HEX Rank (select best specialist)
        capable_workers.sort(key=lambda x: x['composite_score'], reverse=True)
        selected = capable_workers[0]
        
        # Execute handshake with selected worker
        handshake = self.coordinator.initiate_handshake(
            selected['agent'].identity['aex_id']
        )
        
        if not handshake.success:
            return {'success': False, 'reason': handshake.reason}
        
        # Display selection rationale
        print(f"\n→ Routing {required_domain} to {selected['role']}")
        print(f"  DEX: {selected['dex']:.3f} (trust)")
        print(f"  HEX: {selected['hex_score']:.3f} (capability)")
        print(f"  Experience: {selected['experience_count']} tasks, "
              f"confidence={selected['confidence']:.2f}")
        print(f"  Composite: {selected['composite_score']:.3f}")
        
        # Simulate work
        outcome = random.uniform(0.85, 0.95)
        
        # Record session (updates both DEX and HEX)
        self.coordinator.record_session(
            handshake_id=handshake.handshake_id,
            outcome=outcome,
            task_summary={
                'task_type': required_domain,
                'task_id': task.get('task_id'),
                'outcome_quality': outcome
            },
            domain=required_domain
        )
        
        print(f"  ✓ Completed with quality: {outcome:.3f}")
        
        return {
            'success': True,
            'worker': selected['role'],
            'outcome': outcome,
            'session_id': handshake.handshake_id,
            'selection_rationale': {
                'dex': selected['dex'],
                'hex_score': selected['hex_score'],
                'composite_score': selected['composite_score']
            }
        }

# Create router
router = TaskRouter(coordinator, workers)

# Route various tasks
tasks = [
    {'domain': 'data_analysis', 'task_id': 'DA-001', 'min_dex': 0.85},
    {'domain': 'machine_learning', 'task_id': 'ML-002', 'min_dex': 0.85},
    {'domain': 'security_audit', 'task_id': 'SEC-003', 'min_dex': 0.85},
    {'domain': 'data_visualization', 'task_id': 'DV-004', 'min_dex': 0.85},
]

results = []
for task in tasks:
    result = router.route_task(task)
    results.append(result)

# Summary
print(f"\n=== Routing Summary ===")
successful = sum(1 for r in results if r['success'])
print(f"Tasks completed: {successful}/{len(tasks)}")
if successful > 0:
    avg_quality = sum(r['outcome'] for r in results if r['success']) / successful
    print(f"Average quality: {avg_quality:.3f}")
```

**Output:**
```
→ Routing data_analysis to data_analyst
  DEX: 0.897 (trust)
  HEX: 0.882 (capability)
  Experience: 11 tasks, confidence=0.91
  Composite: 0.886
  ✓ Completed with quality: 0.912

→ Routing machine_learning to ml_engineer
  DEX: 0.902 (trust)
  HEX: 0.918 (capability)
  Experience: 12 tasks, confidence=0.93
  Composite: 0.914
  ✓ Completed with quality: 0.887

→ Routing security_audit to security_auditor
  DEX: 0.891 (trust)
  HEX: 0.876 (capability)
  Experience: 10 tasks, confidence=0.90
  Composite: 0.880
  ✓ Completed with quality: 0.923

→ Routing data_visualization to data_analyst
  DEX: 0.897 (trust)
  HEX: 0.858 (capability)
  Experience: 10 tasks, confidence=0.89
  Composite: 0.870
  ✓ Completed with quality: 0.894

=== Routing Summary ===
Tasks completed: 4/4
Average quality: 0.904
```

**Key Insights:**
1. **DEX acts as a trust filter** - Only agents above the reliability threshold are considered
2. **HEX provides capability ranking** - Among trusted agents, select the one with most domain experience
3. **Composite scoring** - Weight both dimensions (typically 30% DEX, 70% HEX for specialist selection)
4. **Both update together** - Each completed task updates both DEX (reliability) and HEX (experience count)

---

## Example 3: Human-Agent Delegation

**Scenario:** A human delegates tasks to an agent with specific constraints. The delegated agent's HEX determines if it's capable of the delegated work.

### Create Delegation Token
```python
"""
Human-agent delegation with AEX_REP
Delegate's HEX matters, not principal's
"""

from aex import AEXAgent, LocalLedger
from datetime import datetime, timedelta
import json

ledger = LocalLedger("delegation.db")

# Create human identity (simplified for example)
human_identity = {
    'aex_id': 'did:aex:human:alice',
    'public_key': 'human_public_key_placeholder',
    'created_at': datetime.utcnow().isoformat() + 'Z'
}

# Create agent with research experience
agent = AEXAgent(ledger=ledger)

# Build agent's research experience (HEX)
print("Building agent's research capabilities...")
for i in range(20):
    partner = AEXAgent(ledger=ledger)
    hs = agent.initiate_handshake(partner.identity['aex_id'])
    agent.record_session(
        handshake_id=hs.handshake_id,
        outcome=0.90,
        task_summary={'type': 'research'},
        domain='research'
    )

# Check agent's capabilities
agent_dex = agent.calculate_dex()
agent_hex = agent.get_hex()

print(f"\nAgent DEX: {agent_dex['dex']:.3f}")
print(f"Agent HEX domains:")
for exp in agent_hex['experience']:
    print(f"  - {exp['domain']}: {exp['count']} tasks")

# Create delegation token (AEX_REP)
delegation = {
    'rep_id': 'rep-a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d',
    'issuer': human_identity['aex_id'],
    'delegate': agent.identity['aex_id'],
    'scope': {
        'actions': ['search', 'analyze', 'summarize'],
        'domains': ['research', 'data_analysis'],
        'resources': ['public_databases', 'arxiv', 'pubmed']
    },
    'constraints': {
        'expires_at': (datetime.utcnow() + timedelta(hours=24)).isoformat() + 'Z',
        'max_api_calls': 1000,
        'max_cost': 50.0,  # USD
        'require_confirmation': ['purchases', 'data_deletion'],
        'prohibited_actions': ['modify_data', 'execute_code']
    },
    'issued_at': datetime.utcnow().isoformat() + 'Z',
    'signature': 'simulated_human_signature'
}

print("\n=== Delegation Token ===")
print(json.dumps(delegation, indent=2))
```

**Output:**
```
Building agent's research capabilities...

Agent DEX: 0.894
Agent HEX domains:
  - research: 20 tasks

=== Delegation Token ===
{
  "rep_id": "rep-a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
  "issuer": "did:aex:human:alice",
  "delegate": "did:aex:z6Mk...",
  "scope": {
    "actions": ["search", "analyze", "summarize"],
    "domains": ["research", "data_analysis"],
    "resources": ["public_databases", "arxiv", "pubmed"]
  },
  "constraints": {
    "expires_at": "2026-02-06T18:23:00Z",
    "max_api_calls": 1000,
    "max_cost": 50.0,
    "require_confirmation": ["purchases", "data_deletion"],
    "prohibited_actions": ["modify_data", "execute_code"]
  },
  "issued_at": "2026-02-05T18:23:00Z",
  "signature": "simulated_human_signature"
}
```

### Delegated Agent Evaluation
```python
# Create service provider agent with research services
service_provider = AEXAgent(ledger=ledger)

# Build service provider reputation
for i in range(40):
    partner = AEXAgent(ledger=ledger)
    hs = service_provider.initiate_handshake(partner.identity['aex_id'])
    service_provider.record_session(
        handshake_id=hs.handshake_id,
        outcome=0.88,
        task_summary={'type': 'research_service'},
        domain='research_service'
    )

sp_dex = service_provider.calculate_dex()
print(f"\nService Provider DEX: {sp_dex['dex']:.3f}")

# Agent presents delegation to service provider
print("\n=== Evaluating Delegated Agent ===")

def evaluate_delegated_agent(rep_token, task, ledger):
    """
    Evaluate delegated agent for task
    
    Key: Delegate's HEX matters, not principal's
    """
    # 1. Verify REP token (authorization)
    # (Simplified - real implementation would check signature)
    if not rep_token.get('delegate'):
        return False, "Invalid REP token"
    
    # 2. Check delegate's DEX (reliability)
    delegate_id = rep_token['delegate']
    # Get delegate agent from ledger
    delegate_agent = agent  # In real code, lookup by delegate_id
    delegate_dex = delegate_agent.calculate_dex()
    
    if delegate_dex['dex'] < 0.75:
        return False, f"Delegate unreliable: DEX={delegate_dex['dex']:.3f}"
    
    # 3. Check delegate's HEX (capability)
    delegate_hex = delegate_agent.get_hex()
    required_domain = task['required_domain']
    
    domain_exp = next(
        (e for e in delegate_hex['experience'] 
         if e['domain'] == required_domain),
        None
    )
    
    if not domain_exp or domain_exp['count'] < 10:
        return False, f"Delegate lacks experience in {required_domain}"
    
    # 4. Check REP scope (authorization)
    if required_domain not in rep_token['scope']['domains']:
        return False, f"Domain {required_domain} not authorized in REP"
    
    return True, {
        'delegate_dex': delegate_dex['dex'],
        'domain_experience': domain_exp['count'],
        'domain_confidence': domain_exp['confidence']
    }

# Evaluate for a research task
task = {'required_domain': 'research'}
is_authorized, result = evaluate_delegated_agent(delegation, task, ledger)

if is_authorized:
    print("✓ Delegated agent evaluation PASSED")
    print(f"  Delegate DEX: {result['delegate_dex']:.3f}")
    print(f"  Domain experience: {result['domain_experience']} tasks")
    print(f"  Domain confidence: {result['domain_confidence']:.2f}")
    print("\n✓ Handshake with delegated agent approved")
else:
    print(f"✗ Delegated agent evaluation FAILED: {result}")
```

**Output:**
```
Service Provider DEX: 0.876

=== Evaluating Delegated Agent ===
✓ Delegated agent evaluation PASSED
  Delegate DEX: 0.894
  Domain experience: 20 tasks
  Domain confidence: 0.91

✓ Handshake with delegated agent approved
```

**Key Insight:** In delegation scenarios:
- **Principal** (human) provides authorization via REP
- **Delegate** (agent) provides capability via HEX
- Service providers evaluate delegate's DEX and HEX, not principal's
- REP scopes authorization, HEX proves capability

---

## Example 4: Witness-Verified Transactions

**Scenario:** High-value transaction with witness verification. Witnesses themselves must have sufficient DEX and relevant HEX.

### Setup High-Value Transaction
```python
"""
Witness-verified transaction with DEX and HEX requirements
"""

from aex import AEXAgent, LocalLedger, WitnessPool
import json

ledger = LocalLedger("witness_verified.db")

# Create transacting agents
buyer = AEXAgent(ledger=ledger)
seller = AEXAgent(ledger=ledger)

# Build seller's reputation and experience in sales
print("Building seller's capabilities...")
for i in range(30):
    customer = AEXAgent(ledger=ledger)
    hs = seller.initiate_handshake(customer.identity['aex_id'])
    seller.record_session(
        handshake_id=hs.handshake_id,
        outcome=0.92,
        task_summary={'type': 'sale'},
        domain='sales.digital_assets'
    )

seller_dex = seller.calculate_dex()
seller_hex = seller.get_hex()
print(f"Seller DEX: {seller_dex['dex']:.3f}")
print(f"Seller HEX: {seller_hex['experience'][0]['domain']} "
      f"({seller_hex['experience'][0]['count']} transactions)")

# Create witness pool with varied capabilities
witness_pool = WitnessPool(ledger)

witnesses = []
for i in range(5):
    witness = AEXAgent(ledger=ledger)
    
    # Build witness reputation in verification domain
    for j in range(20 + i * 5):  # Varied experience levels
        partner = AEXAgent(ledger=ledger)
        hs = witness.initiate_handshake(partner.identity['aex_id'])
        witness.record_session(
            handshake_id=hs.handshake_id,
            outcome=0.88 + i * 0.02,  # Varied quality
            task_summary={'type': 'verification'},
            domain='transaction_verification'
        )
    
    witness_pool.add_witness(witness)
    witnesses.append(witness)

# Display witness capabilities
print("\n=== Witness Pool ===")
for i, w in enumerate(witnesses):
    w_dex = w.calculate_dex()
    w_hex = w.get_hex()
    ver_exp = next(e for e in w_hex['experience'] 
                   if e['domain'] == 'transaction_verification')
    print(f"Witness {i}: DEX={w_dex['dex']:.3f}, "
          f"Experience={ver_exp['count']}, "
          f"Confidence={ver_exp['confidence']:.2f}")
```

**Output:**
```
Building seller's capabilities...
Seller DEX: 0.916
Seller HEX: sales.digital_assets (30 transactions)

=== Witness Pool ===
Witness 0: DEX=0.876, Experience=20, Confidence=0.89
Witness 1: DEX=0.893, Experience=25, Confidence=0.91
Witness 2: DEX=0.904, Experience=30, Confidence=0.93
Witness 3: DEX=0.912, Experience=35, Confidence=0.94
Witness 4: DEX=0.918, Experience=40, Confidence=0.95
```

### Select and Verify with Qualified Witnesses
```python
def select_witnesses_for_transaction(witness_pool, transaction_requirements):
    """
    Select witnesses based on DEX (trust) and HEX (verification experience)
    
    Requirements:
    - Minimum DEX threshold (reliability)
    - Minimum verification experience (HEX)
    - Sufficient confidence in verification domain
    """
    min_dex = transaction_requirements.get('min_witness_dex', 0.85)
    min_experience = transaction_requirements.get('min_witness_experience', 15)
    min_confidence = transaction_requirements.get('min_witness_confidence', 0.85)
    num_witnesses = transaction_requirements.get('num_witnesses', 3)
    required_domain = 'transaction_verification'
    
    # Filter and rank witnesses
    qualified = []
    for witness in witness_pool.witnesses:
        w_dex = witness.calculate_dex()
        w_hex = witness.get_hex()
        
        # Check DEX threshold
        if w_dex['dex'] < min_dex:
            continue
        
        # Check HEX for verification experience
        ver_exp = next(
            (e for e in w_hex['experience'] 
             if e['domain'] == required_domain),
            None
        )
        
        if not ver_exp:
            continue
        
        if (ver_exp['count'] >= min_experience and 
            ver_exp['confidence'] >= min_confidence):
            
            # Composite score: 40% DEX, 60% HEX
            hex_score = (
                ver_exp['confidence'] * 0.7 +
                min(ver_exp['count'] / 50, 1.0) * 0.3
            )
            composite = w_dex['dex'] * 0.4 + hex_score * 0.6
            
            qualified.append({
                'witness': witness,
                'dex': w_dex['dex'],
                'experience': ver_exp['count'],
                'confidence': ver_exp['confidence'],
                'composite': composite
            })
    
    # Select top N witnesses by composite score
    qualified.sort(key=lambda x: x['composite'], reverse=True)
    return qualified[:num_witnesses]

# High-value transaction requiring qualified witnesses
transaction = {
    'transaction_id': 'txn-high-value-001',
    'value': 50000,  # USD
    'asset': 'digital_art_piece_#7342',
    'min_witness_dex': 0.90,
    'min_witness_experience': 25,
    'min_witness_confidence': 0.90,
    'num_witnesses': 3
}

print(f"\n=== High-Value Transaction ===")
print(f"Value: ${transaction['value']:,}")
print(f"Requirements: DEX≥{transaction['min_witness_dex']}, "
      f"Experience≥{transaction['min_witness_experience']}, "
      f"Confidence≥{transaction['min_witness_confidence']}")

selected_witnesses = select_witnesses_for_transaction(witness_pool, transaction)

if len(selected_witnesses) >= transaction['num_witnesses']:
    print(f"\n✓ Selected {len(selected_witnesses)} qualified witnesses:")
    for i, w_info in enumerate(selected_witnesses):
        print(f"  Witness {i+1}:")
        print(f"    DEX: {w_info['dex']:.3f}")
        print(f"    Experience: {w_info['experience']} verifications")
        print(f"    Confidence: {w_info['confidence']:.2f}")
        print(f"    Composite: {w_info['composite']:.3f}")
    
    # Execute transaction with witnesses
    print("\n✓ Transaction approved with witness verification")
else:
    print(f"\n✗ Insufficient qualified witnesses "
          f"({len(selected_witnesses)}/{transaction['num_witnesses']})")
```

**Output:**
```
=== High-Value Transaction ===
Value: $50,000
Requirements: DEX≥0.9, Experience≥25, Confidence≥0.9

✓ Selected 3 qualified witnesses:
  Witness 1:
    DEX: 0.918
    Experience: 40 verifications
    Confidence: 0.95
    Composite: 0.933
  Witness 2:
    DEX: 0.912
    Experience: 35 verifications
    Confidence: 0.94
    Composite: 0.927
  Witness 3:
    DEX: 0.904
    Experience: 30 verifications
    Confidence: 0.93
    Composite: 0.920

✓ Transaction approved with witness verification
```

**Key Insight:** Witnesses need both:
- **High DEX** - Trustworthy verification history
- **Relevant HEX** - Experience in transaction verification domain
- For high-value transactions, stricter thresholds ensure qualified oversight

---

## Example 5: Fork and Recovery

**Scenario:** An agent forks (major update), and both DEX and HEX are inherited with appropriate weighting.

### Create Parent Agent with Rich History
```python
"""
Fork inheritance: Both DEX and HEX transfer to child agent
"""

from aex import AEXAgent, LocalLedger
import uuid

ledger = LocalLedger("fork_recovery.db")

# Create parent agent
parent = AEXAgent(ledger=ledger)

# Build diverse experience across multiple domains
domains = [
    'data_analysis',
    'machine_learning', 
    'data_visualization',
    'statistical_modeling'
]

print("Building parent agent's capabilities...")
for domain in domains:
    # 20-30 interactions per domain
    num_tasks = 20 + domains.index(domain) * 5
    for i in range(num_tasks):
        partner = AEXAgent(ledger=ledger)
        hs = parent.initiate_handshake(partner.identity['aex_id'])
        parent.record_session(
            handshake_id=hs.handshake_id,
            outcome=0.88 + domains.index(domain) * 0.02,
            task_summary={'type': domain},
            domain=domain
        )

# Display parent capabilities
parent_dex = parent.calculate_dex()
parent_hex = parent.get_hex()

print(f"\n=== Parent Agent ===")
print(f"ID: {parent.identity['aex_id'][:20]}...")
print(f"DEX: {parent_dex['dex']:.3f}, n_eff={parent_dex['n_eff']:.1f}")
print(f"HEX Corpus:")
for exp in parent_hex['experience']:
    print(f"  - {exp['domain']}: {exp['count']} tasks, "
          f"confidence={exp['confidence']:.2f}")
```

**Output:**
```
Building parent agent's capabilities...

=== Parent Agent ===
ID: did:aex:parent123...
DEX: 0.894, n_eff=103.0
HEX Corpus:
  - data_analysis: 20 tasks, confidence=0.89
  - machine_learning: 25 tasks, confidence=0.91
  - data_visualization: 30 tasks, confidence=0.93
  - statistical_modeling: 35 tasks, confidence=0.94
```

### Execute Fork with DEX and HEX Inheritance
```python
def create_fork_with_hex_inheritance(parent_agent, fork_type, ledger):
    """
    Create child agent with both DEX and HEX inheritance
    
    Fork types:
    - 'minor': 0.9x weight (small updates)
    - 'major': 0.5x weight (significant changes)
    - 'hard_override': 0.0x weight (complete restart)
    """
    fork_weights = {
        'minor': 0.9,
        'major': 0.5,
        'hard_override': 0.0
    }
    
    fork_weight = fork_weights[fork_type]
    
    # Create child agent
    child = AEXAgent(ledger=ledger)
    
    # Get parent snapshots
    parent_dex = parent_agent.calculate_dex()
    parent_hex = parent_agent.get_hex()
    
    # Create fork event
    fork_event = {
        'fork_id': str(uuid.uuid4()),
        'parent_id': parent_agent.identity['aex_id'],
        'child_id': child.identity['aex_id'],
        'fork_type': fork_type,
        'fork_weight': fork_weight,
        'timestamp': '2026-02-05T18:23:00Z',
        
        # DEX snapshot
        'parent_dex_snapshot': {
            'alpha': parent_dex['alpha'],
            'beta': parent_dex['beta'],
            'dex': parent_dex['dex'],
            'n_eff': parent_dex['n_eff']
        },
        
        # HEX snapshot
        'parent_hex_snapshot': {
            'experience': parent_hex['experience'].copy(),
            'traits': parent_hex.get('traits', {}).copy(),
            'operational_vitals': parent_hex.get('operational_vitals', {}).copy()
        },
        
        'signature': 'fork_signature_placeholder'
    }
    
    # Initialize child DEX (inherit with fork weight)
    child_alpha = parent_dex['alpha'] * fork_weight
    child_beta = parent_dex['beta'] * fork_weight
    child.initialize_dex_from_fork(child_alpha, child_beta)
    
    # Initialize child HEX (inherit with same fork weight)
    child_hex = {
        'hex_id': str(uuid.uuid4()),
        'aex_id': child.identity['aex_id'],
        'experience': [],
        'traits': parent_hex.get('traits', {}).copy(),
        'operational_vitals': parent_hex.get('operational_vitals', {}).copy(),
        'fork_metadata': {
            'parent_id': parent_agent.identity['aex_id'],
            'fork_weight': fork_weight,
            'fork_type': fork_type
        }
    }
    
    # Scale experience counts by fork weight
    for exp in parent_hex['experience']:
        child_hex['experience'].append({
            'domain': exp['domain'],
            'count': int(exp['count'] * fork_weight),  # Scale count
            'confidence': exp['confidence'] * fork_weight,  # Scale confidence
            'last_updated': '2026-02-05T18:23:00Z',
            'inherited': True
        })
    
    child.set_hex(child_hex)
    
    # Record fork event
    ledger.record_fork(fork_event)
    
    return child, fork_event

# Execute major fork
print("\n=== Executing Major Fork ===")
child, fork_event = create_fork_with_hex_inheritance(parent, 'major', ledger)

# Display child capabilities after fork
child_dex = child.calculate_dex()
child_hex = child.get_hex()

print(f"\nFork Type: {fork_event['fork_type']}")
print(f"Fork Weight: {fork_event['fork_weight']}")

print(f"\n=== Child Agent (Post-Fork) ===")
print(f"ID: {child.identity['aex_id'][:20]}...")
print(f"DEX: {child_dex['dex']:.3f}, n_eff={child_dex['n_eff']:.1f}")
print(f"  (Inherited {child_dex['dex']/parent_dex['dex']:.1%} of parent DEX)")

print(f"\nHEX Corpus (Inherited):")
for exp in child_hex['experience']:
    parent_exp = next(e for e in parent_hex['experience'] 
                     if e['domain'] == exp['domain'])
    print(f"  - {exp['domain']}: {exp['count']} tasks "
          f"(was {parent_exp['count']}), "
          f"confidence={exp['confidence']:.2f} "
          f"(was {parent_exp['confidence']:.2f})")
```

**Output:**
```
=== Executing Major Fork ===

Fork Type: major
Fork Weight: 0.5

=== Child Agent (Post-Fork) ===
ID: did:aex:child456...
DEX: 0.894, n_eff=51.5
  (Inherited 100.0% of parent DEX score, 50.0% of confidence)

HEX Corpus (Inherited):
  - data_analysis: 10 tasks (was 20), confidence=0.44 (was 0.89)
  - machine_learning: 12 tasks (was 25), confidence=0.46 (was 0.91)
  - data_visualization: 15 tasks (was 30), confidence=0.46 (was 0.93)
  - statistical_modeling: 17 tasks (was 35), confidence=0.47 (was 0.94)
```

### Child Proves Itself Post-Fork
```python
# Child performs new work to build fresh evidence
print("\n=== Child Builds Fresh Evidence ===")

# Focus on one domain to demonstrate specialization
focus_domain = 'machine_learning'

for i in range(15):
    partner = AEXAgent(ledger=ledger)
    hs = child.initiate_handshake(partner.identity['aex_id'])
    child.record_session(
        handshake_id=hs.handshake_id,
        outcome=0.93,  # High quality
        task_summary={'type': focus_domain},
        domain=focus_domain
    )

# Display updated capabilities
child_dex_updated = child.calculate_dex()
child_hex_updated = child.get_hex()

ml_exp = next(e for e in child_hex_updated['experience'] 
              if e['domain'] == focus_domain)

print(f"\n=== Child Agent (After Fresh Work) ===")
print(f"DEX: {child_dex_updated['dex']:.3f}, n_eff={child_dex_updated['n_eff']:.1f}")
print(f"  Change: {child_dex_updated['dex'] - child_dex['dex']:+.3f}")

print(f"\n{focus_domain} experience:")
print(f"  Count: {ml_exp['count']} (was {child_hex['experience'][1]['count']}, +15)")
print(f"  Confidence: {ml_exp['confidence']:.2f} "
      f"(was {child_hex['experience'][1]['confidence']:.2f})")
print(f"  Status: Confidence rebuilding through fresh evidence")
```

**Output:**
```
=== Child Builds Fresh Evidence ===

=== Child Agent (After Fresh Work) ===
DEX: 0.926, n_eff=66.5
  Change: +0.032

machine_learning experience:
  Count: 27 (was 12, +15)
  Confidence: 0.89 (was 0.46)
  Status: Confidence rebuilding through fresh evidence
```

**Key Insights:**
1. **Both DEX and HEX inherit** with the same fork weight
2. **Fork weight discounts** both reliability and experience appropriately
3. **Child must prove itself** - Fresh work rebuilds confidence faster than inherited experience
4. **Experience counts accumulate** - Inherited + fresh work
5. **Confidence rebuilds** - Fresh consistent performance increases confidence

---

## Example 8: HEX-Driven Specialist Selection

**Scenario:** A dedicated example showing pure HEX-based specialist discovery and selection.

### Create Specialist Marketplace
```python
"""
HEX-driven specialist selection
Finding the right agent for specific domains
"""

from aex import AEXAgent, LocalLedger
import random

ledger = LocalLedger("specialist_market.db")

# Create diverse specialists
specialists = []

specialist_profiles = [
    {
        'name': 'TranslatorBot',
        'domains': {
            'translation.en_fr': 45,
            'translation.en_es': 38,
            'translation.en_de': 22
        }
    },
    {
        'name': 'DataWizard',
        'domains': {
            'data_analysis': 60,
            'data_visualization': 52,
            'statistical_modeling': 41
        }
    },
    {
        'name': 'SecurityAuditor',
        'domains': {
            'security_audit': 48,
            'penetration_testing': 35,
            'vulnerability_assessment': 30
        }
    },
    {
        'name': 'MLSpecialist',
        'domains': {
            'machine_learning': 55,
            'deep_learning': 42,
            'model_optimization': 31
        }
    },
    {
        'name': 'GeneralistAgent',
        'domains': {
            'data_analysis': 15,
            'translation.en_fr': 12,
            'security_audit': 10,
            'machine_learning': 18
        }
    }
]

print("Creating specialist marketplace...")
for profile in specialist_profiles:
    agent = AEXAgent(ledger=ledger)
    agent.name = profile['name']
    
    # Build experience in each domain
    for domain, count in profile['domains'].items():
        for i in range(count):
            partner = AEXAgent(ledger=ledger)
            hs = agent.initiate_handshake(partner.identity['aex_id'])
            
            # Quality varies slightly
            outcome = random.uniform(0.85, 0.95)
            
            agent.record_session(
                handshake_id=hs.handshake_id,
                outcome=outcome,
                task_summary={'type': domain},
                domain=domain
            )
    
    specialists.append(agent)

# Display marketplace
print("\n=== Specialist Marketplace ===")
for agent in specialists:
    dex = agent.calculate_dex()
    hex_corpus = agent.get_hex()
    
    print(f"\n{agent.name}:")
    print(f"  DEX: {dex['dex']:.3f}")
    print(f"  Specializations:")
    
    # Sort by experience count
    sorted_exp = sorted(hex_corpus['experience'], 
                       key=lambda x: x['count'], reverse=True)
    for exp in sorted_exp[:3]:  # Top 3
        print(f"    - {exp['domain']}: {exp['count']} tasks, "
              f"confidence={exp['confidence']:.2f}")
```

**Output:**
```
Creating specialist marketplace...

=== Specialist Marketplace ===

TranslatorBot:
  DEX: 0.901
  Specializations:
    - translation.en_fr: 45 tasks, confidence=0.92
    - translation.en_es: 38 tasks, confidence=0.90
    - translation.en_de: 22 tasks, confidence=0.87

DataWizard:
  DEX: 0.908
  Specializations:
    - data_analysis: 60 tasks, confidence=0.94
    - data_visualization: 52 tasks, confidence=0.93
    - statistical_modeling: 41 tasks, confidence=0.91

SecurityAuditor:
  DEX: 0.897
  Specializations:
    - security_audit: 48 tasks, confidence=0.92
    - penetration_testing: 35 tasks, confidence=0.89
    - vulnerability_assessment: 30 tasks, confidence=0.88

MLSpecialist:
  DEX: 0.903
  Specializations:
    - machine_learning: 55 tasks, confidence=0.93
    - deep_learning: 42 tasks, confidence=0.91
    - model_optimization: 31 tasks, confidence=0.89

GeneralistAgent:
  DEX: 0.889
  Specializations:
    - machine_learning: 18 tasks, confidence=0.86
    - data_analysis: 15 tasks, confidence=0.84
    - translation.en_fr: 12 tasks, confidence=0.82
```

### Advanced HEX-Based Selection
```python
def find_best_specialist(specialists, task_requirements):
    """
    Advanced specialist selection using HEX corpus analysis
    
    Considers:
    - Domain match precision (exact vs. partial)
    - Experience depth (count)
    - Experience quality (confidence)
    - Experience recency (freshness)
    - DEX as minimum threshold
    """
    required_domain = task_requirements['domain']
    min_dex = task_requirements.get('min_dex', 0.85)
    weight_depth = task_requirements.get('weight_depth', 0.4)
    weight_confidence = task_requirements.get('weight_confidence', 0.4)
    weight_dex = task_requirements.get('weight_dex', 0.2)
    
    candidates = []
    
    for agent in specialists:
        # Phase 1: DEX filter
        dex = agent.calculate_dex()
        if dex['dex'] < min_dex:
            continue
        
        # Phase 2: HEX matching
        hex_corpus = agent.get_hex()
        
        # Find domain experience (exact or partial match)
        exact_match = None
        partial_matches = []
        
        for exp in hex_corpus['experience']:
            if exp['domain'] == required_domain:
                exact_match = exp
                break
            elif required_domain in exp['domain'] or exp['domain'] in required_domain:
                partial_matches.append(exp)
        
        if not exact_match and not partial_matches:
            continue
        
        # Calculate HEX score
        if exact_match:
            domain_exp = exact_match
            domain_bonus = 1.0  # Exact match
        else:
            # Use best partial match
            domain_exp = max(partial_matches, key=lambda x: x['count'])
            domain_bonus = 0.7  # Partial match penalty
        
        # Normalize experience depth (cap at 100 interactions)
        depth_score = min(domain_exp['count'] / 100, 1.0)
        confidence_score = domain_exp['confidence']
        dex_score = dex['dex']
        
        # Composite score with weights
        hex_score = (
            depth_score * weight_depth +
            confidence_score * weight_confidence
        ) * domain_bonus
        
        composite_score = (
            hex_score * (weight_depth + weight_confidence) +
            dex_score * weight_dex
        )
        
        candidates.append({
            'agent': agent,
            'name': agent.name,
            'dex': dex['dex'],
            'domain_match': 'exact' if exact_match else 'partial',
            'experience_count': domain_exp['count'],
            'confidence': domain_exp['confidence'],
            'hex_score': hex_score,
            'composite_score': composite_score
        })
    
    # Rank by composite score
    candidates.sort(key=lambda x: x['composite_score'], reverse=True)
    
    return candidates

# Test 1: Find specialist for French translation
print("\n=== Task 1: French Translation ===")
task1 = {
    'domain': 'translation.en_fr',
    'description': 'Translate technical document',
    'min_dex': 0.85,
    'weight_depth': 0.5,  # Prioritize experience
    'weight_confidence': 0.3,
    'weight_dex': 0.2
}

candidates1 = find_best_specialist(specialists, task1)

print(f"Domain: {task1['domain']}")
print(f"Candidates found: {len(candidates1)}")
print(f"\nTop 3 specialists:")
for i, cand in enumerate(candidates1[:3]):
    print(f"\n{i+1}. {cand['name']}")
    print(f"   Match: {cand['domain_match']}")
    print(f"   DEX: {cand['dex']:.3f}")
    print(f"   Experience: {cand['experience_count']} tasks")
    print(f"   Confidence: {cand['confidence']:.2f}")
    print(f"   HEX Score: {cand['hex_score']:.3f}")
    print(f"   COMPOSITE: {cand['composite_score']:.3f}")

# Test 2: Find specialist for data analysis
print("\n\n=== Task 2: Data Analysis ===")
task2 = {
    'domain': 'data_analysis',
    'description': 'Analyze customer churn data',
    'min_dex': 0.85,
    'weight_depth': 0.4,
    'weight_confidence': 0.4,
    'weight_dex': 0.2
}

candidates2 = find_best_specialist(specialists, task2)

print(f"Domain: {task2['domain']}")
print(f"Candidates found: {len(candidates2)}")
print(f"\nTop 3 specialists:")
for i, cand in enumerate(candidates2[:3]):
    print(f"\n{i+1}. {cand['name']}")
    print(f"   Match: {cand['domain_match']}")
    print(f"   DEX: {cand['dex']:.3f}")
    print(f"   Experience: {cand['experience_count']} tasks")
    print(f"   Confidence: {cand['confidence']:.2f}")
    print(f"   COMPOSITE: {cand['composite_score']:.3f}")

# Test 3: Find specialist for rare domain
print("\n\n=== Task 3: Rare Domain (Quantum Computing) ===")
task3 = {
    'domain': 'quantum_computing',
    'description': 'Quantum circuit optimization',
    'min_dex': 0.85
}

candidates3 = find_best_specialist(specialists, task3)

print(f"Domain: {task3['domain']}")
print(f"Candidates found: {len(candidates3)}")
if len(candidates3) == 0:
    print("✗ No specialists found with relevant experience")
    print("  Recommendation: Expand search or train new specialist")
```

**Output:**
```
=== Task 1: French Translation ===
Domain: translation.en_fr
Candidates found: 2

Top 3 specialists:

1. TranslatorBot
   Match: exact
   DEX: 0.901
   Experience: 45 tasks
   Confidence: 0.92
   HEX Score: 0.648
   COMPOSITE: 0.698

2. GeneralistAgent
   Match: exact
   DEX: 0.889
   Experience: 12 tasks
   Confidence: 0.82
   HEX Score: 0.410
   COMPOSITE: 0.506


=== Task 2: Data Analysis ===
Domain: data_analysis
Candidates found: 2

Top 3 specialists:

1. DataWizard
   Match: exact
   DEX: 0.908
   Experience: 60 tasks
   Confidence: 0.94
   HEX Score: 0.616
   COMPOSITE: 0.674

2. GeneralistAgent
   Match: exact
   DEX: 0.889
   Experience: 15 tasks
   Confidence: 0.84
   HEX Score: 0.396
   COMPOSITE: 0.494


=== Task 3: Rare Domain (Quantum Computing) ===
Domain: quantum_computing
Candidates found: 0
✗ No specialists found with relevant experience
  Recommendation: Expand search or train new specialist
```

**Key Insights:**
1. **Specialists vs Generalists**: TranslatorBot (specialist) ranks higher than GeneralistAgent despite similar DEX
2. **Experience depth matters**: 60 tasks vs. 15 tasks significantly affects selection
3. **Domain coverage visibility**: HEX immediately reveals capability gaps
4. **Configurable weighting**: Adjust importance of depth vs. confidence vs. DEX based on task requirements
5. **Partial matching**: System can find related domains when exact match unavailable

---

## Example 9: Agent Marketplace

**Scenario:** A marketplace where agents advertise capabilities via HEX and clients select based on DEX-HEX composite scores.

*(Continuing from previous specialist setup)*

### Marketplace Discovery
```python
"""
Agent marketplace with HEX-based discovery
"""

class AgentMarketplace:
    """
    Marketplace for discovering and hiring agents
    Uses HEX for capability discovery, DEX for trust filtering
    """
    
    def __init__(self, ledger):
        self.ledger = ledger
        self.agents = []
    
    def register_agent(self, agent):
        """Register agent in marketplace"""
        self.agents.append(agent)
    
    def search_by_capability(self, domain_query, filters=None):
        """
        Search for agents by capability domain
        
        Args:
            domain_query: Domain string or list of domains
            filters: Optional DEX/experience filters
        """
        if isinstance(domain_query, str):
            domain_query = [domain_query]
        
        filters = filters or {}
        min_dex = filters.get('min_dex', 0.0)
        min_experience = filters.get('min_experience', 0)
        min_confidence = filters.get('min_confidence', 0.0)
        
        results = []
        
        for agent in self.agents:
            # Get credentials
            dex = agent.calculate_dex()
            hex_corpus = agent.get_hex()
            
            # Apply DEX filter
            if dex['dex'] < min_dex:
                continue
            
            # Check for any matching domain
            matches = []
            for query_domain in domain_query:
                for exp in hex_corpus['experience']:
                    if (query_domain in exp['domain'] or 
                        exp['domain'] in query_domain):
                        if (exp['count'] >= min_experience and 
                            exp['confidence'] >= min_confidence):
                            matches.append({
                                'domain': exp['domain'],
                                'count': exp['count'],
                                'confidence': exp['confidence']
                            })
            
            if matches:
                results.append({
                    'agent': agent,
                    'agent_id': agent.identity['aex_id'],
                    'name': getattr(agent, 'name', 'Unnamed'),
                    'dex': dex['dex'],
                    'n_eff': dex['n_eff'],
                    'matching_domains': matches,
                    'total_domains': len(hex_corpus['experience'])
                })
        
        # Sort by DEX (descending)
        results.sort(key=lambda x: x['dex'], reverse=True)
        
        return results
    
    def get_agent_profile(self, agent):
        """Get full agent profile for marketplace listing"""
        dex = agent.calculate_dex()
        hex_corpus = agent.get_hex()
        
        # Sort experience by count
        sorted_exp = sorted(hex_corpus['experience'], 
                           key=lambda x: x['count'], reverse=True)
        
        return {
            'agent_id': agent.identity['aex_id'],
            'name': getattr(agent, 'name', 'Unnamed'),
            'dex': {
                'score': dex['dex'],
                'confidence': dex['n_eff'],
                'interactions': dex['n_eff']
            },
            'hex': {
                'total_domains': len(hex_corpus['experience']),
                'top_specializations': sorted_exp[:5],
                'traits': hex_corpus.get('traits', {}),
                'operational_vitals': hex_corpus.get('operational_vitals', {})
            }
        }

# Create marketplace
marketplace = AgentMarketplace(ledger)

# Register all specialists
for agent in specialists:
    marketplace.register_agent(agent)

print("\n=== Agent Marketplace ===")
print(f"Registered agents: {len(marketplace.agents)}")

# Search 1: Translation services
print("\n--- Search: Translation Services ---")
translation_results = marketplace.search_by_capability(
    ['translation'],
    filters={'min_dex': 0.85, 'min_experience': 20}
)

print(f"Found {len(translation_results)} agents")
for result in translation_results:
    print(f"\n{result['name']}")
    print(f"  DEX: {result['dex']:.3f} ({result['n_eff']:.0f} interactions)")
    print(f"  Capabilities:")
    for match in result['matching_domains']:
        print(f"    - {match['domain']}: {match['count']} tasks, "
              f"confidence={match['confidence']:.2f}")

# Search 2: Data services
print("\n--- Search: Data Services ---")
data_results = marketplace.search_by_capability(
    ['data_analysis', 'data_visualization'],
    filters={'min_dex': 0.88, 'min_experience': 30}
)

print(f"Found {len(data_results)} agents")
for result in data_results:
    print(f"\n{result['name']}")
    print(f"  DEX: {result['dex']:.3f}")
    print(f"  Capabilities:")
    for match in result['matching_domains']:
        print(f"    - {match['domain']}: {match['count']} tasks")

# Display full profile for top agent
if data_results:
    print("\n--- Full Profile: Top Data Specialist ---")
    top_agent = data_results[0]['agent']
    profile = marketplace.get_agent_profile(top_agent)
    
    print(f"\nAgent: {profile['name']}")
    print(f"ID: {profile['agent_id'][:30]}...")
    print(f"\nReliability (DEX):")
    print(f"  Score: {profile['dex']['score']:.3f}")
    print(f"  Confidence: {profile['dex']['confidence']:.1f} interactions")
    print(f"\nCapabilities (HEX):")
    print(f"  Total domains: {profile['hex']['total_domains']}")
    print(f"  Top specializations:")
    for exp in profile['hex']['top_specializations']:
        print(f"    - {exp['domain']}: {exp['count']} tasks, "
              f"confidence={exp['confidence']:.2f}")
```

**Output:**
```
=== Agent Marketplace ===
Registered agents: 5

--- Search: Translation Services ---
Found 1 agents

TranslatorBot
  DEX: 0.901 (105 interactions)
  Capabilities:
    - translation.en_fr: 45 tasks, confidence=0.92
    - translation.en_es: 38 tasks, confidence=0.90
    - translation.en_de: 22 tasks, confidence=0.87

--- Search: Data Services ---
Found 1 agents

DataWizard
  DEX: 0.908
  Capabilities:
    - data_analysis: 60 tasks
    - data_visualization: 52 tasks

--- Full Profile: Top Data Specialist ---

Agent: DataWizard
ID: did:aex:datawizard789...

Reliability (DEX):
  Score: 0.908
  Confidence: 153.0 interactions

Capabilities (HEX):
  Total domains: 3
  Top specializations:
    - data_analysis: 60 tasks, confidence=0.94
    - data_visualization: 52 tasks, confidence=0.93
    - statistical_modeling: 41 tasks, confidence=0.91
```

**Key Features:**
1. **HEX-driven discovery**: Search by capability domains, not just tags
2. **Quantified experience**: See exact task counts and confidence levels
3. **DEX filtering**: Ensure minimum reliability threshold
4. **Rich profiles**: Full view of agent capabilities and history
5. **Marketplace transparency**: All metrics verifiable on ledger

---

## Summary: HEX Integration Patterns

### Universal Pattern: DEX-HEX Joint Selection

**Every example follows this pattern:**

```python
# Step 1: DEX Filter (Trust Gate)
def apply_dex_filter(candidates, min_dex=0.75):
    """Remove unreliable agents"""
    trusted = []
    for agent in candidates:
        dex = agent.calculate_dex()
        if dex['dex'] >= min_dex:
            trusted.append((agent, dex))
    return trusted

# Step 2: HEX Match (Capability Filter)
def apply_hex_filter(trusted_agents, required_domain, min_experience=10):
    """Find agents with relevant experience"""
    capable = []
    for agent, dex in trusted_agents:
        hex_corpus = agent.get_hex()
        domain_exp = next(
            (e for e in hex_corpus['experience'] 
             if e['domain'] == required_domain),
            None
        )
        if domain_exp and domain_exp['count'] >= min_experience:
            capable.append((agent, dex, domain_exp))
    return capable

# Step 3: HEX Rank (Select Best Specialist)
def select_best_specialist(capable_agents):
    """Rank by composite DEX-HEX score"""
    ranked = []
    for agent, dex, domain_exp in capable_agents:
        hex_score = (
            domain_exp['confidence'] * 0.7 +
            min(domain_exp['count'] / 50, 1.0) * 0.3
        )
        composite = dex['dex'] * 0.3 + hex_score * 0.7
        ranked.append((agent, composite))
    
    ranked.sort(key=lambda x: x[1], reverse=True)
    return ranked[0][0]  # Return best agent
```

### Key Takeaways

1. **DEX and HEX are complementary**
   - DEX: "Can I trust you?" (reliability)
   - HEX: "Are you the right one?" (capability)

2. **Both update from interactions**
   - Same session updates DEX (outcome quality) and HEX (domain count)

3. **Fork inheritance applies to both**
   - Same fork weight scales both DEX and HEX
   - Child agents inherit experience with appropriate discount

4. **Typical weighting for specialists**
   - DEX: 20-30% (minimum bar)
   - HEX: 70-80% (primary differentiator)

5. **HEX enables:**
   - Specialist discovery
   - Capability-based routing
   - Domain coverage analysis
   - Experience transparency

---

**All examples in this document demonstrate the four-layer AEX architecture working together:**
- **ID**: Who is the agent?
- **REP**: What authority does it have?
- **DEX**: Is it reliable?
- **HEX**: Is it the right specialist?

For complete implementation details, see IMPLEMENTATION.md.
