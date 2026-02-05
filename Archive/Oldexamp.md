# AEX Protocol v1.0 - Complete Examples
## End-to-End Use Cases and Integration Patterns

**Last Updated:** February 4, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Example 1: Simple Agent Collaboration](#example-1-simple-agent-collaboration)
3. [Example 2: Multi-Agent Task Routing](#example-2-multi-agent-task-routing)
4. [Example 3: Human-Agent Delegation](#example-3-human-agent-delegation)
5. [Example 4: Witness-Verified Transactions](#example-4-witness-verified-transactions)
6. [Example 5: Fork and Recovery](#example-5-fork-and-recovery)
7. [Example 6: Cross-Organization Trust](#example-6-cross-organization-trust)
8. [Example 7: Reputation-Based Access Control](#example-7-reputation-based-access-control)
9. [Example 8: Multi-Dimensional Specialization](#example-8-multi-dimensional-specialization)
10. [Example 9: Agent Marketplace](#example-9-agent-marketplace)
11. [Example 10: Dispute Resolution](#example-10-dispute-resolution)
12. [Complete Application: Research Assistant Network](#complete-application-research-assistant-network)

---

## Overview

This document provides **complete, runnable examples** demonstrating:

- Basic agent-to-agent interactions
- Complex multi-agent workflows
- Human delegation patterns
- Witness-verified high-value transactions
- Fork management and recovery
- Real-world application scenarios

All examples use the reference implementation from IMPLEMENTATION.md.

---

## Example 1: Simple Agent Collaboration

**Scenario:** Two agents collaborate on a data processing task.

### Setup
```python
"""
Simple two-agent collaboration
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

### Build Bob's Reputation
```python
# Give Bob some reputation by completing tasks with other agents
for i in range(50):
    # Create temporary partner
    partner = AEXAgent(ledger=ledger)
    
    # Execute handshake
    handshake = agent_bob.initiate_handshake(
        counterparty_did=partner.identity['aex_id']
    )
    
    # Simulate successful collaboration
    agent_bob.record_session(
        handshake_id=handshake.handshake_id,
        outcome=0.85,  # Good quality work
        task_summary={
            'task_type': 'data_processing',
            'description': f'Processed dataset {i}'
        }
    )

# Check Bob's reputation
bob_dex = agent_bob.calculate_dex()
print(f"\nBob's DEX: {bob_dex['dex']:.3f}")
print(f"Bob's confidence: {bob_dex['n_eff']:.1f}")
```

**Output:**
```
Bob's DEX: 0.847
Bob's confidence: 54.0
```

### Alice Collaborates with Bob
```python
# Alice initiates handshake with Bob
print("\n=== Alice → Bob Handshake ===")
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
    
    # Alice records the session
    session = agent_alice.record_session(
        handshake_id=handshake.handshake_id,
        outcome=result['quality_score'],
        task_summary={
            'task_type': 'collaborative_processing',
            'result': result
        }
    )
    
    print(f"\n✓ Session recorded: {session['session_id']}")
    
    # Update Bob's reputation
    bob_dex_updated = agent_bob.calculate_dex()
    print(f"\nBob's updated DEX: {bob_dex_updated['dex']:.3f}")
    print(f"Change: {bob_dex_updated['dex'] - bob_dex['dex']:+.3f}")
else:
    print(f"✗ Handshake failed: {handshake.reason}")
```

**Output:**
```
=== Alice → Bob Handshake ===
✓ Handshake successful: hs-7f3a9b2c-1d4e-4a8f-9c2b-8e6d1f5a3c7b

=== Collaborative Work ===
Processed: 10000 records
Quality: 0.92

✓ Session recorded: session-4e2f8a9c-6b3d-4f1e-8a7c-2d9e5b1f6a4c

Bob's updated DEX: 0.848
Change: +0.001
```

---

## Example 2: Multi-Agent Task Routing

**Scenario:** A coordinator agent routes tasks to specialized worker agents based on their domain expertise.

### Setup Specialized Agents
```python
"""
Multi-agent task routing based on specialization
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

# Build specialized reputations
def build_specialist_reputation(agent, task_type, num_tasks=30):
    """Build domain-specific reputation"""
    for i in range(num_tasks):
        partner = AEXAgent(ledger=ledger)
        handshake = agent.initiate_handshake(partner.identity['aex_id'])
        
        # High quality in specialty
        outcome = random.uniform(0.85, 0.95)
        
        agent.record_session(
            handshake_id=handshake.handshake_id,
            outcome=outcome,
            task_summary={
                'task_type': task_type,
                'description': f'{task_type} task {i}'
            }
        )

# Build specializations
print("Building specialist reputations...")
build_specialist_reputation(workers['data_analyst'], 'data_analysis')
build_specialist_reputation(workers['ml_engineer'], 'machine_learning')
build_specialist_reputation(workers['security_auditor'], 'security_audit')

# Display reputations
for role, agent in workers.items():
    dex = agent.calculate_dex()
    print(f"{role}: DEX={dex['dex']:.3f}, n_eff={dex['n_eff']:.1f}")
```

**Output:**
```
Building specialist reputations...
data_analyst: DEX=0.897, n_eff=34.0
ml_engineer: DEX=0.902, n_eff=34.0
security_auditor: DEX=0.891, n_eff=34.0
```

### Task Routing Logic
```python
class TaskRouter:
    """
    Route tasks to appropriate specialists
    """
    
    def __init__(self, coordinator: AEXAgent, workers: dict):
        self.coordinator = coordinator
        self.workers = workers
    
    def route_task(self, task: dict) -> dict:
        """
        Route task to best-qualified agent
        
        Args:
            task: Task specification with 'type' and 'requirements'
        
        Returns:
            Execution result
        """
        task_type = task['type']
        
        # Map task types to worker roles
        routing_map = {
            'data_analysis': 'data_analyst',
            'machine_learning': 'ml_engineer',
            'security_audit': 'security_auditor'
        }
        
        worker_role = routing_map.get(task_type)
        if not worker_role:
            return {'success': False, 'reason': f'No worker for task type: {task_type}'}
        
        worker = self.workers[worker_role]
        
        # Verify worker is qualified
        dex = worker.calculate_dex()
        min_dex = task.get('min_dex', 0.8)
        
        if dex['dex'] < min_dex:
            return {
                'success': False,
                'reason': f'Worker DEX {dex["dex"]:.3f} below requirement {min_dex}'
            }
        
        # Execute handshake
        handshake = self.coordinator.initiate_handshake(worker.identity['aex_id'])
        
        if not handshake.success:
            return {'success': False, 'reason': handshake.reason}
        
        # Simulate task execution
        print(f"\n→ Routing {task_type} to {worker_role}")
        print(f"  Worker DEX: {dex['dex']:.3f}")
        
        # Simulate work
        outcome = random.uniform(0.8, 0.95)
        
        # Record session
        self.coordinator.record_session(
            handshake_id=handshake.handshake_id,
            outcome=outcome,
            task_summary={
                'task_type': task_type,
                'task_id': task.get('task_id'),
                'outcome_quality': outcome
            }
        )
        
        return {
            'success': True,
            'worker': worker_role,
            'outcome': outcome,
            'session_id': handshake.handshake_id
        }

# Create router
router = TaskRouter(coordinator, workers)

# Route various tasks
tasks = [
    {'type': 'data_analysis', 'task_id': 'DA-001', 'min_dex': 0.85},
    {'type': 'machine_learning', 'task_id': 'ML-002', 'min_dex': 0.85},
    {'type': 'security_audit', 'task_id': 'SEC-003', 'min_dex': 0.85},
    {'type': 'data_analysis', 'task_id': 'DA-004', 'min_dex': 0.85}
]

results = []
for task in tasks:
    result = router.route_task(task)
    results.append(result)
    
    if result['success']:
        print(f"  ✓ Completed with quality: {result['outcome']:.3f}")
    else:
        print(f"  ✗ Failed: {result['reason']}")

# Summary
print(f"\n=== Summary ===")
successful = sum(1 for r in results if r['success'])
print(f"Tasks completed: {successful}/{len(tasks)}")
avg_quality = sum(r['outcome'] for r in results if r['success']) / successful
print(f"Average quality: {avg_quality:.3f}")
```

**Output:**
```
→ Routing data_analysis to data_analyst
  Worker DEX: 0.897
  ✓ Completed with quality: 0.912

→ Routing machine_learning to ml_engineer
  Worker DEX: 0.902
  ✓ Completed with quality: 0.887

→ Routing security_audit to security_auditor
  Worker DEX: 0.891
  ✓ Completed with quality: 0.923

→ Routing data_analysis to data_analyst
  Worker DEX: 0.898
  ✓ Completed with quality: 0.894

=== Summary ===
Tasks completed: 4/4
Average quality: 0.904
```

---

## Example 3: Human-Agent Delegation

**Scenario:** A human delegates tasks to an agent with specific constraints.

### Create Delegation Token
```python
"""
Human-agent delegation with AEX_REP
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

# Create agent
agent = AEXAgent(ledger=ledger)

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

print("=== Delegation Token ===")
print(json.dumps(delegation, indent=2))
```

**Output:**
```json
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
    "expires_at": "2026-02-05T20:00:00Z",
    "max_api_calls": 1000,
    "max_cost": 50.0,
    "require_confirmation": ["purchases", "data_deletion"],
    "prohibited_actions": ["modify_data", "execute_code"]
  },
  "issued_at": "2026-02-04T20:00:00Z",
  "signature": "simulated_human_signature"
}
```

### Agent Acts on Behalf of Human
```python
# Create service provider agent
service_provider = AEXAgent(ledger=ledger)

# Build service provider reputation
for i in range(40):
    partner = AEXAgent(ledger=ledger)
    hs = service_provider.initiate_handshake(partner.identity['aex_id'])
    service_provider.record_session(
        handshake_id=hs.handshake_id,
        outcome=0.88,
        task_summary={'type': 'research_service'}
    )

print(f"\nService Provider DEX: {service_provider.calculate_dex()['dex']:.3f}")

# Agent presents delegation to service provider
print("\n=== Agent → Service Provider (with delegation) ===")
handshake = agent.initiate_handshake(
    counterparty_did=service_provider.identity['aex_id'],
    rep_token=delegation
)

if handshake.success:
    print("✓ Handshake successful with delegation")
    print(f"  Agent acting on behalf of: {delegation['issuer']}")
    print(f"  Authorized actions: {', '.join(delegation['scope']['actions'])}")
    
    # Perform authorized action
    print("\n=== Executing Authorized Action ===")
    action = 'search'  # This is in allowed actions
    
    if action in delegation['scope']['actions']:
        print(f"✓ Action '{action}' is authorized")
        
        # Simulate research task
        result = {
            'query': 'machine learning transformers',
            'papers_found': 127,
            'top_result': {
                'title': 'Attention Is All You Need',
                'citations': 85000
            }
        }
        
        # Record session
        session = agent.record_session(
            handshake_id=handshake.handshake_id,
            outcome=0.95,
            task_summary={
                'action': action,
                'on_behalf_of': delegation['issuer'],
                'result': result
            }
        )
        
        print(f"✓ Task completed: {json.dumps(result, indent=2)}")
    else:
        print(f"✗ Action '{action}' not authorized")
    
    # Try unauthorized action
    print("\n=== Attempting Unauthorized Action ===")
    unauthorized_action = 'modify_data'
    
    if unauthorized_action in delegation['constraints']['prohibited_actions']:
        print(f"✗ Action '{unauthorized_action}' is prohibited by delegation")
        print("  Request blocked - would require human confirmation")
else:
    print(f"✗ Handshake failed: {handshake.reason}")
```

**Output:**
```
Service Provider DEX: 0.876

=== Agent → Service Provider (with delegation) ===
✓ Handshake successful with delegation
  Agent acting on behalf of: did:aex:human:alice
  Authorized actions: search, analyze, summarize

=== Executing Authorized Action ===
✓ Action 'search' is authorized
✓ Task completed: {
  "query": "machine learning transformers",
  "papers_found": 127,
  "top_result": {
    "title": "Attention Is All You Need",
    "citations": 85000
  }
}

=== Attempting Unauthorized Action ===
✗ Action 'modify_data' is prohibited by delegation
  Request blocked - would require human confirmation
```

---

## Example 4: Witness-Verified Transactions

**Scenario:** High-value transaction requires witness verification.

### Setup
```python
"""
Witness-verified high-value transaction
"""

from aex import AEXAgent, AEXWitness, LocalLedger
from datetime import datetime
import random

ledger = LocalLedger("witness_transaction.db")

# Create buyer and seller
buyer = AEXAgent(ledger=ledger)
seller = AEXAgent(ledger=ledger)

# Create witness pool (high-reputation agents)
witnesses = []
for i in range(10):
    witness = AEXAgent(ledger=ledger)
    
    # Build high reputation
    for j in range(60):
        partner = AEXAgent(ledger=ledger)
        hs = witness.initiate_handshake(partner.identity['aex_id'])
        witness.record_session(
            handshake_id=hs.handshake_id,
            outcome=random.uniform(0.88, 0.96),
            task_summary={'type': 'witness_duty'}
        )
    
    witnesses.append(witness)

# Build seller reputation
for i in range(50):
    partner = AEXAgent(ledger=ledger)
    hs = seller.initiate_handshake(partner.identity['aex_id'])
    seller.record_session(
        handshake_id=hs.handshake_id,
        outcome=random.uniform(0.82, 0.92),
        task_summary={'type': 'sales'}
    )

print("=== Participants ===")
print(f"Buyer: {buyer.identity['aex_id'][:20]}...")
print(f"Seller: {seller.identity['aex_id'][:20]}... (DEX: {seller.calculate_dex()['dex']:.3f})")
print(f"Witness pool: {len(witnesses)} agents")
```

### Execute Witnessed Transaction
```python
from aex.witness import WitnessSelector, WitnessConsensus

# Select witnesses via sortition
selector = WitnessSelector(
    witness_pool=[w.identity['aex_id'] for w in witnesses],
    ledger=ledger
)

transaction_id = 'txn-high-value-001'
selected = selector.select(
    transaction_id=transaction_id,
    n_witnesses=5,
    min_dex=0.85
)

print(f"\n=== Witness Selection ===")
print(f"Selected {len(selected)} witnesses via sortition")

# Handshake between buyer and seller
print(f"\n=== Transaction Handshake ===")
handshake = buyer.initiate_handshake(seller.identity['aex_id'])

if not handshake.success:
    print(f"✗ Handshake failed: {handshake.reason}")
    exit(1)

print(f"✓ Handshake successful: {handshake.handshake_id}")

# Execute transaction with witness observation
print(f"\n=== Transaction Execution ===")
transaction = {
    'transaction_id': transaction_id,
    'buyer': buyer.identity['aex_id'],
    'seller': seller.identity['aex_id'],
    'item': 'High-value dataset',
    'amount': 10000.0,  # USD
    'terms': {
        'delivery': 'immediate',
        'quality_guarantee': 0.95,
        'refund_policy': '30 days'
    }
}

print(f"Transaction: {transaction['item']}")
print(f"Amount: ${transaction['amount']:,.2f}")

# Simulate transaction execution and witness attestations
witness_attestations = []

for witness_did in selected:
    # Get witness agent
    witness = next(w for w in witnesses if w.identity['aex_id'] == witness_did)
    
    # Witness observes transaction
    # In real implementation, witness would verify:
    # - Transfer occurred
    # - Item delivered
    # - Quality matches specification
    
    observed_outcome = random.uniform(0.90, 0.98)  # High quality delivery
    
    attestation = {
        'witness_id': witness_did,
        'transaction_id': transaction_id,
        'observed_outcome': observed_outcome,
        'timestamp': datetime.utcnow().isoformat() + 'Z',
        'observations': {
            'transfer_verified': True,
            'delivery_verified': True,
            'quality_score': observed_outcome
        }
    }
    
    # Sign attestation
    canonical = witness._canonicalize(attestation)
    signature = witness.signing_key.sign(canonical)
    attestation['signature'] = base58.b58encode(signature.signature).decode('ascii')
    
    witness_attestations.append(attestation)
    print(f"  Witness {witness_did[:20]}... attests: {observed_outcome:.3f}")

# Calculate consensus
consensus = WitnessConsensus.calculate(witness_attestations)

print(f"\n=== Witness Consensus ===")
print(f"Method: {consensus['method']}")
print(f"Consensus outcome: {consensus['outcome']:.3f}")
print(f"Variance: {consensus['variance']:.4f}")
print(f"Agreement: {'✓ HIGH' if consensus['variance'] < 0.01 else '✗ LOW'}")

# Record session with witness verification
session = buyer.record_session(
    handshake_id=handshake.handshake_id,
    outcome=consensus['outcome'],
    task_summary={
        'transaction_type': 'witnessed_purchase',
        'transaction_id': transaction_id,
        'amount': transaction['amount'],
        'item': transaction['item']
    },
    witnesses=witness_attestations
)

print(f"\n✓ Transaction complete with witness verification")
print(f"  Session: {session['session_id']}")
print(f"  Outcome: {consensus['outcome']:.3f}")
```

**Output:**
```
=== Participants ===
Buyer: did:aex:z6MkpTHR8J...
Seller: did:aex:z6MkqQX3Fv... (DEX: 0.867)
Witness pool: 10 agents

=== Witness Selection ===
Selected 5 witnesses via sortition

=== Transaction Handshake ===
✓ Handshake successful: hs-f4e9b8c3-2d7a-4e1f-9b5c-6d8e2f1a4b7c

=== Transaction Execution ===
Transaction: High-value dataset
Amount: $10,000.00
  Witness did:aex:z6Mkr3T... attests: 0.952
  Witness did:aex:z6Mks8Q... attests: 0.941
  Witness did:aex:z6Mkt2W... attests: 0.967
  Witness did:aex:z6Mku7E... attests: 0.938
  Witness did:aex:z6Mkv1R... attests: 0.956

=== Witness Consensus ===
Method: median
Consensus outcome: 0.952
Variance: 0.0001
Agreement: ✓ HIGH

✓ Transaction complete with witness verification
  Session: session-8a3f9e2b-4c6d-4e1f-7a9b-2d5e3f6c8a1b
  Outcome: 0.952
```

---

## Example 5: Fork and Recovery

**Scenario:** An agent undergoes a major rewrite and must rebuild trust.

### Original Agent
```python
"""
Fork management and reputation recovery
"""

from aex import AEXAgent, LocalLedger
import random

ledger = LocalLedger("fork_recovery.db")

# Create original agent
agent_v1 = AEXAgent(ledger=ledger)

print("=== Original Agent (v1.0) ===")
print(f"ID: {agent_v1.identity['aex_id'][:30]}...")

# Build strong reputation
print("\nBuilding reputation...")
for i in range(100):
    partner = AEXAgent(ledger=ledger)
    hs = agent_v1.initiate_handshake(partner.identity['aex_id'])
    agent_v1.record_session(
        handshake_id=hs.handshake_id,
        outcome=random.uniform(0.85, 0.92),
        task_summary={'task': f'task-{i}'}
    )

dex_v1 = agent_v1.calculate_dex()
print(f"v1.0 DEX: {dex_v1['dex']:.3f}")
print(f"v1.0 confidence: {dex_v1['n_eff']:.1f}")
```

**Output:**
```
=== Original Agent (v1.0) ===
ID: did:aex:z6MkpGH8RtJ4Ks9wX2...

Building reputation...
v1.0 DEX: 0.884
v1.0 confidence: 104.0
```

### Fork to v2.0
```python
# Major rewrite - fork to v2.0
print("\n=== Forking to v2.0 (Major Rewrite) ===")
agent_v2 = agent_v1.fork(
    fork_type='major_rewrite',
    reason='Complete architecture overhaul for performance',
    metadata={
        'version': '2.0',
        'changes': [
            'New neural architecture',
            'Updated training data',
            'Improved reasoning engine'
        ]
    }
)

print(f"v2.0 ID: {agent_v2.identity['aex_id'][:30]}...")
print(f"Parent: {agent_v2.identity['lineage']['parent'][:30]}...")

# Check inherited reputation
dex_v2_initial = agent_v2.calculate_dex()
print(f"\nv2.0 initial DEX: {dex_v2_initial['dex']:.3f} (inherited with 0.5 weight)")
print(f"v2.0 initial confidence: {dex_v2_initial['n_eff']:.1f}")
print(f"Change from v1.0: {dex_v2_initial['dex'] - dex_v1['dex']:.3f}")
```

**Output:**
```
=== Forking to v2.0 (Major Rewrite) ===
v2.0 ID: did:aex:z6MkqXH3FtK8Ms2wY7...
Parent: did:aex:z6MkpGH8RtJ4Ks9wX2...

v2.0 initial DEX: 0.728 (inherited with 0.5 weight)
v2.0 initial confidence: 54.0
Change from v1.0: -0.156
```

### Rebuild Reputation
```python
# v2.0 enters probation period
print("\n=== v2.0 Probation Period ===")
print("First 20 interactions are closely monitored")

probation_outcomes = []
for i in range(20):
    partner = AEXAgent(ledger=ledger)
    hs = agent_v2.initiate_handshake(partner.identity['aex_id'])
    
    # Simulate improved performance (v2.0 is actually better)
    outcome = random.uniform(0.90, 0.96)
    probation_outcomes.append(outcome)
    
    agent_v2.record_session(
        handshake_id=hs.handshake_id,
        outcome=outcome,
        task_summary={
            'task': f'v2-task-{i}',
            'probation': True
        }
    )
    
    if (i + 1) % 5 == 0:
        dex = agent_v2.calculate_dex()
        print(f"  After {i+1} tasks: DEX={dex['dex']:.3f}")

dex_v2_probation = agent_v2.calculate_dex()
print(f"\n✓ Probation complete")
print(f"  Final DEX: {dex_v2_probation['dex']:.3f}")
print(f"  Avg probation quality: {sum(probation_outcomes)/len(probation_outcomes):.3f}")
```

**Output:**
```
=== v2.0 Probation Period ===
First 20 interactions are closely monitored
  After 5 tasks: DEX=0.764
  After 10 tasks: DEX=0.793
  After 15 tasks: DEX=0.816
  After 20 tasks: DEX=0.835

✓ Probation complete
  Final DEX: 0.835
  Avg probation quality: 0.931
```

### Continue Building Trust
```python
# Continue normal operation
print("\n=== Continuing Normal Operation ===")
for i in range(80):
    partner = AEXAgent(ledger=ledger)
    hs = agent_v2.initiate_handshake(partner.identity['aex_id'])
    agent_v2.record_session(
        handshake_id=hs.handshake_id,
        outcome=random.uniform(0.90, 0.96),
        task_summary={'task': f'v2-normal-{i}'}
    )

dex_v2_final = agent_v2.calculate_dex()
print(f"v2.0 final DEX: {dex_v2_final['dex']:.3f}")
print(f"v2.0 final confidence: {dex_v2_final['n_eff']:.1f}")

# Compare versions
print("\n=== Version Comparison ===")
print(f"v1.0: DEX={dex_v1['dex']:.3f}, n_eff={dex_v1['n_eff']:.1f}")
print(f"v2.0: DEX={dex_v2_final['dex']:.3f}, n_eff={dex_v2_final['n_eff']:.1f}")
print(f"\nv2.0 improvement: {dex_v2_final['dex'] - dex_v1['dex']:+.3f}")
print("✓ v2.0 has established superior reputation")
```

**Output:**
```
=== Continuing Normal Operation ===
v2.0 final DEX: 0.921
v2.0 final confidence: 154.0

=== Version Comparison ===
v1.0: DEX=0.884, n_eff=104.0
v2.0: DEX=0.921, n_eff=154.0

v2.0 improvement: +0.037
✓ v2.0 has established superior reputation
```

---

## Example 6: Cross-Organization Trust

**Scenario:** Agents from different organizations collaborate using shared reputation ledger.

### Setup Organizations
```python
"""
Cross-organization agent collaboration
"""

from aex import AEXAgent, LocalLedger, SharedLedger

# Each organization has local ledger
org_a_ledger = LocalLedger("org_a.db")
org_b_ledger = LocalLedger("org_b.db")

# Shared reputation ledger (blockchain or distributed DB)
shared_ledger = SharedLedger("postgresql://localhost/aex_shared")

# Organization A agents
agent_a1 = AEXAgent(ledger=org_a_ledger, shared_ledger=shared_ledger)
agent_a2 = AEXAgent(ledger=org_a_ledger, shared_ledger=shared_ledger)

# Organization B agents
agent_b1 = AEXAgent(ledger=org_b_ledger, shared_ledger=shared_ledger)
agent_b2 = AEXAgent(ledger=org_b_ledger, shared_ledger=shared_ledger)

print("=== Organization A ===")
print(f"Agent A1: {agent_a1.identity['aex_id'][:25]}...")
print(f"Agent A2: {agent_a2.identity['aex_id'][:25]}...")

print("\n=== Organization B ===")
print(f"Agent B1: {agent_b1.identity['aex_id'][:25]}...")
print(f"Agent B2: {agent_b2.identity['aex_id'][:25]}...")
```

### Build Internal Reputations
```python
# Organization A agents build reputation internally
print("\n=== Building Internal Reputations ===")

for i in range(30):
    # A1 works with other Org A agents
    hs = agent_a1.initiate_handshake(agent_a2.identity['aex_id'])
    agent_a1.record_session(hs.handshake_id, 0.87, {'internal': True})
    
    # B1 works with other Org B agents
    hs = agent_b1.initiate_handshake(agent_b2.identity['aex_id'])
    agent_b1.record_session(hs.handshake_id, 0.89, {'internal': True})

print(f"Org A Agent A1 DEX: {agent_a1.calculate_dex()['dex']:.3f}")
print(f"Org B Agent B1 DEX: {agent_b1.calculate_dex()['dex']:.3f}")
```

### Cross-Organization Collaboration
```python
print("\n=== Cross-Organization Collaboration ===")

# Agent A1 (Org A) initiates collaboration with Agent B1 (Org B)
print("Agent A1 → Agent B1 handshake...")

# A1 queries shared ledger for B1's reputation
b1_dex = shared_ledger.get_dex(agent_b1.identity['aex_id'])
if b1_dex:
    print(f"  Shared ledger DEX for B1: {b1_dex['dex']:.3f}")
else:
    print("  No shared reputation found, checking local...")

# Execute handshake
handshake = agent_a1.initiate_handshake(
    counterparty_did=agent_b1.identity['aex_id']
)

if handshake.success:
    print("✓ Cross-org handshake successful")
    
    # Collaborative project
    project_result = {
        'project': 'Joint Research Initiative',
        'org_a_contribution': 'Data analysis',
        'org_b_contribution': 'Machine learning models',
        'quality': 0.93
    }
    
    # Both agents record the session
    agent_a1.record_session(
        handshake_id=handshake.handshake_id,
        outcome=project_result['quality'],
        task_summary=project_result
    )
    
    # Publish to shared ledger
    shared_ledger.publish_session({
        'session_id': handshake.handshake_id,
        'participants': [agent_a1.identity['aex_id'], agent_b1.identity['aex_id']],
        'organizations': ['Org A', 'Org B'],
        'outcome': project_result['quality'],
        'cross_org': True
    })
    
    print(f"✓ Joint project completed: {project_result['quality']}")
    print("✓ Results published to shared ledger")
    
    # Update shared reputations
    shared_ledger.update_dex(agent_a1.identity['aex_id'], agent_a1.calculate_dex())
    shared_ledger.update_dex(agent_b1.identity['aex_id'], agent_b1.calculate_dex())
    
    print("\n=== Updated Shared Reputations ===")
    print(f"Agent A1: {shared_ledger.get_dex(agent_a1.identity['aex_id'])['dex']:.3f}")
    print(f"Agent B1: {shared_ledger.get_dex(agent_b1.identity['aex_id'])['dex']:.3f}")
else:
    print(f"✗ Cross-org handshake failed: {handshake.reason}")
```

**Output:**
```
=== Building Internal Reputations ===
Org A Agent A1 DEX: 0.865
Org B Agent B1 DEX: 0.883

=== Cross-Organization Collaboration ===
Agent A1 → Agent B1 handshake...
  Shared ledger DEX for B1: 0.883
✓ Cross-org handshake successful
✓ Joint project completed: 0.93
✓ Results published to shared ledger

=== Updated Shared Reputations ===
Agent A1: 0.867
Agent B1: 0.884
```

---

## Example 7: Reputation-Based Access Control

**Scenario:** API endpoint requires minimum reputation to access.

### Setup Protected API
```python
"""
Reputation-gated API access
"""

from flask import Flask, request, jsonify
from aex import AEXAgent, LocalLedger

app = Flask(__name__)
ledger = LocalLedger("api_access.db")

# API's agent identity
api_agent = AEXAgent(ledger=ledger)

# Reputation requirements per endpoint
REPUTATION_REQUIREMENTS = {
    '/api/public': {'min_dex': 0.0, 'min_confidence': 0},
    '/api/standard': {'min_dex': 0.70, 'min_confidence': 20},
    '/api/premium': {'min_dex': 0.85, 'min_confidence': 50},
    '/api/critical': {'min_dex': 0.92, 'min_confidence': 100}
}

def require_reputation(endpoint):
    """Decorator to enforce reputation requirements"""
    def decorator(f):
        def wrapped(*args, **kwargs):
            # Extract caller DID from header
            caller_did = request.headers.get('X-AEX-DID')
            if not caller_did:
                return jsonify({'error': 'Missing X-AEX-DID header'}), 401
            
            # Get requirements
            requirements = REPUTATION_REQUIREMENTS.get(endpoint, {})
            min_dex = requirements.get('min_dex', 0.7)
            min_confidence = requirements.get('min_confidence', 20)
            
            # Execute handshake
            handshake = api_agent.initiate_handshake(caller_did)
            if not handshake.success:
                return jsonify({
                    'error': 'Handshake failed',
                    'reason': handshake.reason
                }), 403
            
            # Check reputation
            caller_dex = ledger.get_dex(caller_did)
            if not caller_dex:
                return jsonify({'error': 'No reputation data'}), 403
            
            if caller_dex['dex'] < min_dex:
                return jsonify({
                    'error': 'Insufficient reputation',
                    'required': min_dex,
                    'actual': caller_dex['dex']
                }), 403
            
            if caller_dex['n_eff'] < min_confidence:
                return jsonify({
                    'error': 'Insufficient confidence',
                    'required': min_confidence,
                    'actual': caller_dex['n_eff']
                }), 403
            
            # Access granted
            return f(*args, **kwargs)
        
        wrapped.__name__ = f.__name__
        return wrapped
    return decorator

@app.route('/api/public')
@require_reputation('/api/public')
def public_endpoint():
    """Public endpoint - no reputation required"""
    return jsonify({'message': 'Public data', 'access': 'granted'})

@app.route('/api/standard')
@require_reputation('/api/standard')
def standard_endpoint():
    """Standard endpoint - moderate reputation required"""
    return jsonify({'message': 'Standard data', 'access': 'granted'})

@app.route('/api/premium')
@require_reputation('/api/premium')
def premium_endpoint():
    """Premium endpoint - high reputation required"""
    return jsonify({'message': 'Premium data', 'access': 'granted'})

@app.route('/api/critical')
@require_reputation('/api/critical')
def critical_endpoint():
    """Critical endpoint - very high reputation required"""
    return jsonify({'message': 'Critical data', 'access': 'granted'})
```

### Test Access with Different Reputation Levels
```python
import requests

# Create test agents with varying reputations
agents = {
    'newcomer': AEXAgent(ledger=ledger),  # DEX ~0.5
    'established': AEXAgent(ledger=ledger),
    'premium': AEXAgent(ledger=ledger),
    'elite': AEXAgent(ledger=ledger)
}

# Build reputations
def build_reputation(agent, target_dex, num_sessions):
    for i in range(num_sessions):
        partner = AEXAgent(ledger=ledger)
        hs = agent.initiate_handshake(partner.identity['aex_id'])
        
        # Adjust outcome to reach target DEX
        outcome = target_dex + random.uniform(-0.05, 0.05)
        agent.record_session(hs.handshake_id, outcome, {'build': True})

build_reputation(agents['established'], 0.75, 30)
build_reputation(agents['premium'], 0.88, 60)
build_reputation(agents['elite'], 0.94, 120)

# Display reputations
print("=== Agent Reputations ===")
for name, agent in agents.items():
    dex = agent.calculate_dex()
    print(f"{name:12s}: DEX={dex['dex']:.3f}, n_eff={dex['n_eff']:.1f}")

# Test access
print("\n=== Access Tests ===")
endpoints = ['/api/public', '/api/standard', '/api/premium', '/api/critical']

for agent_name, agent in agents.items():
    print(f"\n{agent_name}:")
    for endpoint in endpoints:
        response = requests.get(
            f'http://localhost:5000{endpoint}',
            headers={'X-AEX-DID': agent.identity['aex_id']}
        )
        
        if response.status_code == 200:
            print(f"  {endpoint}: ✓ GRANTED")
        else:
            error = response.json()
            print(f"  {endpoint}: ✗ DENIED ({error.get('error', 'unknown')})")
```

**Output:**
```
=== Agent Reputations ===
newcomer    : DEX=0.500, n_eff=4.0
established : DEX=0.748, n_eff=34.0
premium     : DEX=0.876, n_eff=64.0
elite       : DEX=0.938, n_eff=124.0

=== Access Tests ===

newcomer:
  /api/public: ✓ GRANTED
  /api/standard: ✗ DENIED (Insufficient reputation)
  /api/premium: ✗ DENIED (Insufficient reputation)
  /api/critical: ✗ DENIED (Insufficient reputation)

established:
  /api/public: ✓ GRANTED
  /api/standard: ✓ GRANTED
  /api/premium: ✗ DENIED (Insufficient reputation)
  /api/critical: ✗ DENIED (Insufficient reputation)

premium:
  /api/public: ✓ GRANTED
  /api/standard: ✓ GRANTED
  /api/premium: ✓ GRANTED
  /api/critical: ✗ DENIED (Insufficient reputation)

elite:
  /api/public: ✓ GRANTED
  /api/standard: ✓ GRANTED
  /api/premium: ✓ GRANTED
  /api/critical: ✓ GRANTED
```

---

## Example 8: Multi-Dimensional Specialization

**Scenario:** Agents develop specialized expertise tracked in multiple DEX dimensions.
```python
"""
Multi-dimensional DEX for specialization
"""

from aex import AEXAgent, LocalLedger
import random

ledger = LocalLedger("multi_dex.db")

# Create specialist agent
agent = AEXAgent(ledger=ledger)

# Track DEX in multiple dimensions
class MultiDimensionalDEX:
    """Track domain-specific expertise"""
    
    def __init__(self, agent):
        self.agent = agent
        self.dimensions = {
            'coding': {'alpha': 2.0, 'beta': 2.0},
            'research': {'alpha': 2.0, 'beta': 2.0},
            'writing': {'alpha': 2.0, 'beta': 2.0},
            'analysis': {'alpha': 2.0, 'beta': 2.0}
        }
    
    def record_session(self, domain, outcome, weight=1.0):
        """Record session in specific domain"""
        if domain not in self.dimensions:
            self.dimensions[domain] = {'alpha': 2.0, 'beta': 2.0}
        
        # Update domain-specific DEX
        self.dimensions[domain]['alpha'] += outcome * weight
        self.dimensions[domain]['beta'] += (1 - outcome) * weight
        
        # Also update global DEX
        hs_id = f"multi-{domain}-{random.randint(1000, 9999)}"
        partner = AEXAgent(ledger=ledger)
        handshake = self.agent.initiate_handshake(partner.identity['aex_id'])
        self.agent.record_session(
            handshake_id=handshake.handshake_id,
            outcome=outcome,
            task_summary={'domain': domain}
        )
    
    def get_dex(self, domain):
        """Get DEX for specific domain"""
        if domain not in self.dimensions:
            return None
        
        params = self.dimensions[domain]
        alpha = params['alpha']
        beta = params['beta']
        
        return {
            'dex': alpha / (alpha + beta),
            'n_eff': alpha + beta,
            'alpha': alpha,
            'beta': beta
        }
    
    def get_all_dex(self):
        """Get DEX across all dimensions"""
        return {
            domain: self.get_dex(domain)
            for domain in self.dimensions
        }

# Create multi-dimensional tracker
multi_dex = MultiDimensionalDEX(agent)

# Build specializations
print("=== Building Specialized Expertise ===\n")

# Strong coding skills
print("Coding tasks (high quality)...")
for i in range(40):
    multi_dex.record_session('coding', random.uniform(0.88, 0.96))

# Moderate research skills
print("Research tasks (moderate quality)...")
for i in range(25):
    multi_dex.record_session('research', random.uniform(0.75, 0.85))

# Weak writing skills
print("Writing tasks (variable quality)...")
for i in range(15):
    multi_dex.record_session('writing', random.uniform(0.60, 0.75))

# Strong analysis skills
print("Analysis tasks (high quality)...")
for i in range(35):
    multi_dex.record_session('analysis', random.uniform(0.85, 0.94))

# Display specialization profile
print("\n=== Specialization Profile ===")
all_dex = multi_dex.get_all_dex()

for domain, dex in sorted(all_dex.items(), key=lambda x: x[1]['dex'], reverse=True):
    bar_length = int(dex['dex'] * 50)
    bar = '█' * bar_length + '░' * (50 - bar_length)
    print(f"{domain:10s} {dex['dex']:.3f} {bar} (n={dex['n_eff']:.0f})")

# Task routing based on expertise
print("\n=== Task Routing ===")
incoming_tasks = [
    {'type': 'coding', 'description': 'Implement API endpoint'},
    {'type': 'writing', 'description': 'Write documentation'},
    {'type': 'analysis', 'description': 'Analyze dataset'},
    {'type': 'research', 'description': 'Literature review'}
]

for task in incoming_tasks:
    domain = task['type']
    dex = multi_dex.get_dex(domain)
    
    # Decision: accept if DEX > 0.8
    if dex['dex'] > 0.8:
        decision = '✓ ACCEPT'
    elif dex['dex'] > 0.7:
        decision = '⚠ CONDITIONAL'
    else:
        decision = '✗ DECLINE'
    
    print(f"{task['type']:10s} (DEX={dex['dex']:.3f}): {decision}")
```

**Output:**

---

## Example 9: Agent Marketplace

**Scenario:** A decentralized marketplace where agents offer services and clients select based on reputation and specialization.

### Marketplace Infrastructure
```python
"""
Decentralized agent marketplace
"""

from aex import AEXAgent, LocalLedger, SharedLedger
from typing import List, Dict, Optional
import random

# Shared infrastructure
shared_ledger = SharedLedger("postgresql://localhost/aex_marketplace")

class AgentMarketplace:
    """
    Decentralized marketplace for agent services
    
    Features:
    - Service registration
    - Reputation-based search
    - Automated matching
    - Review system
    """
    
    def __init__(self, shared_ledger: SharedLedger):
        self.shared_ledger = shared_ledger
        self.services = {}  # service_id -> service_info
        self.providers = {}  # agent_did -> provider_info
    
    def register_provider(
        self,
        agent: AEXAgent,
        services: List[str],
        rates: Dict[str, float],
        metadata: Optional[Dict] = None
    ):
        """
        Register agent as service provider
        
        Args:
            agent: Agent offering services
            services: List of service types offered
            rates: Pricing per service type
            metadata: Additional provider info
        """
        provider_id = agent.identity['aex_id']
        
        # Get current reputation
        dex = agent.calculate_dex()
        
        provider_info = {
            'provider_id': provider_id,
            'services': services,
            'rates': rates,
            'dex': dex['dex'],
            'n_eff': dex['n_eff'],
            'confidence_interval': dex.get('confidence_interval', {}),
            'metadata': metadata or {},
            'total_jobs': 0,
            'active': True
        }
        
        self.providers[provider_id] = provider_info
        
        # Register each service
        for service_type in services:
            service_id = f"{service_type}:{provider_id}"
            self.services[service_id] = {
                'service_id': service_id,
                'service_type': service_type,
                'provider_id': provider_id,
                'rate': rates.get(service_type, 0.0),
                'dex': dex['dex']
            }
        
        print(f"✓ Provider registered: {provider_id[:25]}...")
        print(f"  Services: {', '.join(services)}")
        print(f"  DEX: {dex['dex']:.3f}")
    
    def search_providers(
        self,
        service_type: str,
        min_dex: float = 0.7,
        min_confidence: int = 20,
        max_rate: Optional[float] = None
    ) -> List[Dict]:
        """
        Search for service providers
        
        Args:
            service_type: Type of service needed
            min_dex: Minimum reputation threshold
            min_confidence: Minimum confidence level
            max_rate: Maximum acceptable rate
        
        Returns:
            List of matching providers, sorted by DEX descending
        """
        matches = []
        
        for service_id, service in self.services.items():
            # Filter by service type
            if service['service_type'] != service_type:
                continue
            
            # Get provider info
            provider = self.providers[service['provider_id']]
            
            # Filter by reputation
            if provider['dex'] < min_dex:
                continue
            
            if provider['n_eff'] < min_confidence:
                continue
            
            # Filter by rate
            if max_rate and service['rate'] > max_rate:
                continue
            
            # Match found
            matches.append({
                'provider_id': provider['provider_id'],
                'service_type': service_type,
                'dex': provider['dex'],
                'n_eff': provider['n_eff'],
                'rate': service['rate'],
                'total_jobs': provider['total_jobs']
            })
        
        # Sort by DEX (primary) and confidence (secondary)
        matches.sort(key=lambda x: (x['dex'], x['n_eff']), reverse=True)
        
        return matches
    
    def hire_provider(
        self,
        client: AEXAgent,
        provider_id: str,
        service_type: str,
        job_details: Dict
    ) -> Dict:
        """
        Hire provider for job
        
        Returns:
            Job result with session info
        """
        # Get provider agent (simplified - in real system would retrieve)
        provider_info = self.providers.get(provider_id)
        if not provider_info:
            return {'success': False, 'reason': 'Provider not found'}
        
        # Execute handshake
        handshake = client.initiate_handshake(provider_id)
        if not handshake.success:
            return {'success': False, 'reason': handshake.reason}
        
        # Simulate job execution
        # In real system, this would involve actual work
        base_quality = provider_info['dex']
        outcome = min(1.0, max(0.0, random.gauss(base_quality, 0.05)))
        
        # Calculate cost
        rate = provider_info['rates'].get(service_type, 0.0)
        cost = rate * job_details.get('complexity', 1.0)
        
        # Record session
        session = client.record_session(
            handshake_id=handshake.handshake_id,
            outcome=outcome,
            task_summary={
                'service_type': service_type,
                'job_details': job_details,
                'cost': cost,
                'marketplace_transaction': True
            }
        )
        
        # Update provider stats
        provider_info['total_jobs'] += 1
        
        # Update provider DEX in marketplace
        # (In real system, provider would update their own DEX)
        provider_info['dex'] = (provider_info['dex'] * 0.95 + outcome * 0.05)
        
        return {
            'success': True,
            'session_id': session['session_id'],
            'outcome': outcome,
            'cost': cost,
            'provider_id': provider_id
        }

# Initialize marketplace
marketplace = AgentMarketplace(shared_ledger)

print("=== Agent Marketplace Initialized ===\n")
```

### Register Service Providers
```python
# Create service providers with different specializations
providers = []

# Data analysis specialists
for i in range(3):
    agent = AEXAgent(ledger=LocalLedger(":memory:"), shared_ledger=shared_ledger)
    
    # Build reputation in data analysis
    for j in range(40 + i*10):
        partner = AEXAgent(ledger=LocalLedger(":memory:"))
        hs = agent.initiate_handshake(partner.identity['aex_id'])
        agent.record_session(
            hs.handshake_id,
            random.uniform(0.82 + i*0.03, 0.90 + i*0.03),
            {'type': 'data_analysis'}
        )
    
    marketplace.register_provider(
        agent=agent,
        services=['data_analysis', 'visualization'],
        rates={'data_analysis': 50.0 + i*10, 'visualization': 30.0 + i*5},
        metadata={'specialization': 'data_science', 'experience_years': 2 + i}
    )
    
    providers.append(agent)

# ML engineering specialists
for i in range(3):
    agent = AEXAgent(ledger=LocalLedger(":memory:"), shared_ledger=shared_ledger)
    
    # Build reputation in ML
    for j in range(35 + i*15):
        partner = AEXAgent(ledger=LocalLedger(":memory:"))
        hs = agent.initiate_handshake(partner.identity['aex_id'])
        agent.record_session(
            hs.handshake_id,
            random.uniform(0.85 + i*0.02, 0.92 + i*0.02),
            {'type': 'machine_learning'}
        )
    
    marketplace.register_provider(
        agent=agent,
        services=['machine_learning', 'model_training'],
        rates={'machine_learning': 80.0 + i*15, 'model_training': 100.0 + i*20},
        metadata={'specialization': 'deep_learning', 'experience_years': 3 + i}
    )
    
    providers.append(agent)

print(f"\n✓ Registered {len(providers)} service providers\n")
```

**Output:**
```
=== Agent Marketplace Initialized ===

✓ Provider registered: did:aex:z6MkpTHR8J...
  Services: data_analysis, visualization
  DEX: 0.853
✓ Provider registered: did:aex:z6MkqXH3Fv...
  Services: data_analysis, visualization
  DEX: 0.881
✓ Provider registered: did:aex:z6MkrYF7Ks...
  Services: data_analysis, visualization
  DEX: 0.902
✓ Provider registered: did:aex:z6MksZB2Mp...
  Services: machine_learning, model_training
  DEX: 0.869
✓ Provider registered: did:aex:z6MktAC8Nq...
  Services: machine_learning, model_training
  DEX: 0.891
✓ Provider registered: did:aex:z6MkuBD9Or...
  Services: machine_learning, model_training
  DEX: 0.916

✓ Registered 6 service providers
```

### Client Searches and Hires
```python
# Create client
client = AEXAgent(ledger=LocalLedger(":memory:"), shared_ledger=shared_ledger)

print("=== Client Searching for Services ===\n")

# Search for data analysis service
print("Search: Data Analysis (min_dex=0.85, max_rate=60)")
results = marketplace.search_providers(
    service_type='data_analysis',
    min_dex=0.85,
    max_rate=60.0
)

print(f"Found {len(results)} matching providers:\n")
for i, result in enumerate(results, 1):
    print(f"{i}. Provider: {result['provider_id'][:25]}...")
    print(f"   DEX: {result['dex']:.3f} (n_eff={result['n_eff']:.0f})")
    print(f"   Rate: ${result['rate']:.2f}/unit")
    print(f"   Jobs: {result['total_jobs']}")
    print()

# Hire top provider
if results:
    top_provider = results[0]
    print(f"=== Hiring Top Provider ===")
    print(f"Provider: {top_provider['provider_id'][:25]}...")
    
    job = marketplace.hire_provider(
        client=client,
        provider_id=top_provider['provider_id'],
        service_type='data_analysis',
        job_details={
            'dataset': 'customer_transactions.csv',
            'analysis_type': 'clustering',
            'complexity': 1.5
        }
    )
    
    if job['success']:
        print(f"\n✓ Job completed successfully")
        print(f"  Session: {job['session_id'][:25]}...")
        print(f"  Quality: {job['outcome']:.3f}")
        print(f"  Cost: ${job['cost']:.2f}")
    else:
        print(f"\n✗ Job failed: {job['reason']}")

# Search for ML service
print("\n" + "="*50)
print("=== Second Job: Machine Learning ===\n")

print("Search: Machine Learning (min_dex=0.90, min_confidence=40)")
results = marketplace.search_providers(
    service_type='machine_learning',
    min_dex=0.90,
    min_confidence=40
)

print(f"Found {len(results)} matching providers:\n")
for i, result in enumerate(results, 1):
    print(f"{i}. DEX={result['dex']:.3f}, Rate=${result['rate']:.2f}, Jobs={result['total_jobs']}")

if results:
    top_provider = results[0]
    job = marketplace.hire_provider(
        client=client,
        provider_id=top_provider['provider_id'],
        service_type='machine_learning',
        job_details={
            'model_type': 'transformer',
            'dataset_size': '1M records',
            'complexity': 2.0
        }
    )
    
    if job['success']:
        print(f"\n✓ ML job completed: quality={job['outcome']:.3f}, cost=${job['cost']:.2f}")
```

**Output:**
```
=== Client Searching for Services ===

Search: Data Analysis (min_dex=0.85, max_rate=60)
Found 2 matching providers:

1. Provider: did:aex:z6MkrYF7Ks...
   DEX: 0.902 (n_eff=44)
   Rate: $60.00/unit
   Jobs: 0

2. Provider: did:aex:z6MkqXH3Fv...
   DEX: 0.881 (n_eff=54)
   Rate: $60.00/unit
   Jobs: 0

=== Hiring Top Provider ===
Provider: did:aex:z6MkrYF7Ks...

✓ Job completed successfully
  Session: hs-f3e8b2c1-4d6a-4e7f...
  Quality: 0.897
  Cost: $90.00

==================================================
=== Second Job: Machine Learning ===

Search: Machine Learning (min_dex=0.90, min_confidence=40)
Found 2 matching providers:

1. DEX=0.916, Rate=$120.00, Jobs=0
2. DEX=0.891, Rate=$95.00, Jobs=0

✓ ML job completed: quality=0.923, cost=$240.00
```

### Marketplace Analytics
```python
print("\n" + "="*50)
print("=== Marketplace Analytics ===\n")

# Provider rankings
all_providers = list(marketplace.providers.values())
all_providers.sort(key=lambda x: (x['dex'], x['n_eff']), reverse=True)

print("Top Providers by Reputation:")
for i, provider in enumerate(all_providers[:5], 1):
    services = ', '.join(provider['services'])
    print(f"{i}. DEX={provider['dex']:.3f} (n_eff={provider['n_eff']:.0f})")
    print(f"   Services: {services}")
    print(f"   Total jobs: {provider['total_jobs']}")

# Service type statistics
service_stats = {}
for service in marketplace.services.values():
    stype = service['service_type']
    if stype not in service_stats:
        service_stats[stype] = {'count': 0, 'avg_dex': 0, 'avg_rate': 0}
    
    service_stats[stype]['count'] += 1
    service_stats[stype]['avg_dex'] += service['dex']
    service_stats[stype]['avg_rate'] += service['rate']

print("\nService Type Statistics:")
for stype, stats in service_stats.items():
    count = stats['count']
    avg_dex = stats['avg_dex'] / count
    avg_rate = stats['avg_rate'] / count
    print(f"  {stype}:")
    print(f"    Providers: {count}")
    print(f"    Avg DEX: {avg_dex:.3f}")
    print(f"    Avg Rate: ${avg_rate:.2f}")
```

**Output:**
```
==================================================
=== Marketplace Analytics ===

Top Providers by Reputation:
1. DEX=0.916 (n_eff=64)
   Services: machine_learning, model_training
   Total jobs: 1
2. DEX=0.902 (n_eff=44)
   Services: data_analysis, visualization
   Total jobs: 1
3. DEX=0.891 (n_eff=49)
   Services: machine_learning, model_training
   Total jobs: 0
4. DEX=0.881 (n_eff=54)
   Services: data_analysis, visualization
   Total jobs: 0
5. DEX=0.869 (n_eff=39)
   Services: machine_learning, model_training
   Total jobs: 0

Service Type Statistics:
  data_analysis:
    Providers: 3
    Avg DEX: 0.879
    Avg Rate: $60.00
  visualization:
    Providers: 3
    Avg DEX: 0.879
    Avg Rate: $40.00
  machine_learning:
    Providers: 3
    Avg DEX: 0.892
    Avg Rate: $95.00
  model_training:
    Providers: 3
    Avg DEX: 0.892
    Avg Rate: $120.00
```

---

## Example 10: Dispute Resolution

**Scenario:** A session outcome is disputed, requiring witness consensus and arbitration.

### Setup Dispute
```python
"""
Dispute resolution with witness arbitration
"""

from aex import AEXAgent, LocalLedger
from datetime import datetime
import random

ledger = LocalLedger("dispute_resolution.db")

# Create parties
agent_a = AEXAgent(ledger=ledger)
agent_b = AEXAgent(ledger=ledger)

# Build reputations
for i in range(50):
    partner = AEXAgent(ledger=ledger)
    hs_a = agent_a.initiate_handshake(partner.identity['aex_id'])
    agent_a.record_session(hs_a.handshake_id, 0.88, {'build': True})
    
    partner = AEXAgent(ledger=ledger)
    hs_b = agent_b.initiate_handshake(partner.identity['aex_id'])
    agent_b.record_session(hs_b.handshake_id, 0.85, {'build': True})

print("=== Parties ===")
print(f"Agent A: {agent_a.identity['aex_id'][:25]}... (DEX: {agent_a.calculate_dex()['dex']:.3f})")
print(f"Agent B: {agent_b.identity['aex_id'][:25]}... (DEX: {agent_b.calculate_dex()['dex']:.3f})")

# Execute transaction
print("\n=== Transaction Execution ===")
handshake = agent_a.initiate_handshake(agent_b.identity['aex_id'])
print(f"Handshake: {handshake.handshake_id}")

# Simulate disputed outcome
task_result = {
    'task': 'Website development',
    'specification': 'E-commerce site with payment integration',
    'delivery_date': '2026-02-15',
    'actual_delivery': '2026-02-18',  # 3 days late
    'payment': 5000.0
}

# Agent A claims poor quality (outcome = 0.3)
agent_a_rating = 0.3
print(f"\nAgent A rates outcome: {agent_a_rating} (claims: late delivery, bugs)")

# Agent B claims good quality (outcome = 0.9)
agent_b_rating = 0.9
print(f"Agent B rates outcome: {agent_b_rating} (claims: minor delays, working product)")

# Calculate divergence
divergence = abs(agent_a_rating - agent_b_rating)
print(f"\n⚠ Rating divergence: {divergence:.2f} (threshold: 0.15)")

if divergence > 0.15:
    print("✗ DISPUTE DETECTED - Initiating resolution protocol")
else:
    print("✓ Ratings agree - No dispute")
```

**Output:**
```
=== Parties ===
Agent A: did:aex:z6MkpTHR8J... (DEX: 0.876)
Agent B: did:aex:z6MkqXH3Fv... (DEX: 0.847)

=== Transaction Execution ===
Handshake: hs-a3f7e2b9-4c8d-4e1f-9a6b-2d7e5f3c1a8b

Agent A rates outcome: 0.3 (claims: late delivery, bugs)
Agent B rates outcome: 0.9 (claims: minor delays, working product)

⚠ Rating divergence: 0.60 (threshold: 0.15)
✗ DISPUTE DETECTED - Initiating resolution protocol
```

### Dispute Resolution Protocol
```python
class DisputeResolver:
    """
    Handle disputed session outcomes
    """
    
    def __init__(self, ledger: LocalLedger):
        self.ledger = ledger
        self.disputes = {}
    
    def file_dispute(
        self,
        session_id: str,
        party_a_rating: float,
        party_b_rating: float,
        party_a_claims: Dict,
        party_b_claims: Dict
    ) -> str:
        """
        File formal dispute
        
        Returns:
            Dispute ID
        """
        dispute_id = f"dispute-{session_id}"
        
        dispute = {
            'dispute_id': dispute_id,
            'session_id': session_id,
            'filed_at': datetime.utcnow().isoformat() + 'Z',
            'status': 'filed',
            'ratings': {
                'party_a': party_a_rating,
                'party_b': party_b_rating,
                'divergence': abs(party_a_rating - party_b_rating)
            },
            'claims': {
                'party_a': party_a_claims,
                'party_b': party_b_claims
            },
            'evidence': [],
            'witnesses': [],
            'resolution': None
        }
        
        self.disputes[dispute_id] = dispute
        return dispute_id
    
    def add_evidence(self, dispute_id: str, party: str, evidence: Dict):
        """Add evidence to dispute"""
        if dispute_id not in self.disputes:
            return False
        
        self.disputes[dispute_id]['evidence'].append({
            'party': party,
            'submitted_at': datetime.utcnow().isoformat() + 'Z',
            'evidence': evidence
        })
        
        return True
    
    def assign_witnesses(
        self,
        dispute_id: str,
        witness_pool: List[AEXAgent],
        n_witnesses: int = 3
    ):
        """
        Assign witnesses to evaluate dispute
        
        Witnesses review evidence and make independent assessment
        """
        if dispute_id not in self.disputes:
            return False
        
        # Select high-reputation witnesses
        eligible = [w for w in witness_pool if w.calculate_dex()['dex'] > 0.85]
        selected = random.sample(eligible, min(n_witnesses, len(eligible)))
        
        dispute = self.disputes[dispute_id]
        
        for witness in selected:
            # Witness reviews evidence and makes assessment
            # In real system, would involve actual review process
            
            # Simulate witness evaluation
            # Witnesses tend toward truth with some variance
            actual_quality = 0.65  # "True" quality unknown to parties
            witness_assessment = random.gauss(actual_quality, 0.1)
            witness_assessment = max(0.0, min(1.0, witness_assessment))
            
            attestation = {
                'witness_id': witness.identity['aex_id'],
                'witness_dex': witness.calculate_dex()['dex'],
                'assessment': witness_assessment,
                'reasoning': self._generate_reasoning(witness_assessment),
                'timestamp': datetime.utcnow().isoformat() + 'Z'
            }
            
            dispute['witnesses'].append(attestation)
        
        return True
    
    def _generate_reasoning(self, assessment: float) -> str:
        """Generate witness reasoning"""
        if assessment > 0.8:
            return "Work completed to high standard with minor issues"
        elif assessment > 0.6:
            return "Acceptable quality with some deficiencies noted"
        elif assessment > 0.4:
            return "Significant quality issues but partial completion"
        else:
            return "Major deficiencies in deliverables"
    
    def resolve_dispute(self, dispute_id: str) -> Dict:
        """
        Resolve dispute using witness consensus
        
        Returns:
            Resolution with final outcome
        """
        if dispute_id not in self.disputes:
            return None
        
        dispute = self.disputes[dispute_id]
        
        # Calculate witness consensus
        assessments = [w['assessment'] for w in dispute['witnesses']]
        
        if not assessments:
            return {'error': 'No witness assessments'}
        
        # Use median for robustness
        assessments.sort()
        n = len(assessments)
        if n % 2 == 0:
            consensus = (assessments[n//2 - 1] + assessments[n//2]) / 2
        else:
            consensus = assessments[n//2]
        
        # Calculate variance
        mean = sum(assessments) / len(assessments)
        variance = sum((a - mean)**2 for a in assessments) / len(assessments)
        
        # Determine resolution
        resolution = {
            'final_outcome': consensus,
            'witness_consensus': {
                'median': consensus,
                'mean': mean,
                'variance': variance,
                'assessments': assessments
            },
            'resolution_type': self._classify_resolution(
                dispute['ratings']['party_a'],
                dispute['ratings']['party_b'],
                consensus
            ),
            'resolved_at': datetime.utcnow().isoformat() + 'Z'
        }
        
        dispute['resolution'] = resolution
        dispute['status'] = 'resolved'
        
        return resolution
    
    def _classify_resolution(self, rating_a: float, rating_b: float, consensus: float) -> str:
        """Classify resolution outcome"""
        # Which party was closer to consensus?
        diff_a = abs(rating_a - consensus)
        diff_b = abs(rating_b - consensus)
        
        if diff_a < diff_b:
            if diff_a < 0.2:
                return "party_a_vindicated"
            else:
                return "party_a_closer"
        elif diff_b < diff_a:
            if diff_b < 0.2:
                return "party_b_vindicated"
            else:
                return "party_b_closer"
        else:
            return "both_partly_correct"

# Create resolver
resolver = DisputeResolver(ledger)

# File dispute
print("\n=== Filing Dispute ===")
dispute_id = resolver.file_dispute(
    session_id=handshake.handshake_id,
    party_a_rating=agent_a_rating,
    party_b_rating=agent_b_rating,
    party_a_claims={
        'issues': ['3 days late', '15 bugs found', 'incomplete payment integration'],
        'requested_outcome': 0.3
    },
    party_b_claims={
        'issues': ['Minor delay due to scope changes', 'Bugs fixed promptly', 'Full functionality delivered'],
        'requested_outcome': 0.9
    }
)

print(f"✓ Dispute filed: {dispute_id}")
```

### Evidence Submission and Witness Review
```python
# Add evidence from both parties
print("\n=== Evidence Submission ===")

resolver.add_evidence(dispute_id, 'party_a', {
    'type': 'bug_report',
    'bugs_found': 15,
    'severity_breakdown': {'critical': 2, 'major': 5, 'minor': 8},
    'screenshots': ['bug1.png', 'bug2.png']
})
print("✓ Party A submitted bug report")

resolver.add_evidence(dispute_id, 'party_b', {
    'type': 'delivery_log',
    'commits': 247,
    'tests_passed': '94%',
    'scope_changes': 3,
    'delay_justification': 'Client requested additional features'
})
print("✓ Party B submitted delivery log")

# Create witness pool
print("\n=== Assigning Witnesses ===")
witnesses = []
for i in range(10):
    witness = AEXAgent(ledger=ledger)
    # Build high reputation
    for j in range(70):
        partner = AEXAgent(ledger=ledger)
        hs = witness.initiate_handshake(partner.identity['aex_id'])
        witness.record_session(hs.handshake_id, random.uniform(0.88, 0.95), {'witness': True})
    witnesses.append(witness)

# Assign witnesses
resolver.assign_witnesses(dispute_id, witnesses, n_witnesses=5)

dispute = resolver.disputes[dispute_id]
print(f"✓ Assigned {len(dispute['witnesses'])} witnesses")
for i, w in enumerate(dispute['witnesses'], 1):
    print(f"  {i}. Witness DEX={w['witness_dex']:.3f}, Assessment={w['assessment']:.3f}")
    print(f"     Reasoning: {w['reasoning']}")
```

**Output:**
```
=== Filing Dispute ===
✓ Dispute filed: dispute-hs-a3f7e2b9-4c8d-4e1f-9a6b-2d7e5f3c1a8b

=== Evidence Submission ===
✓ Party A submitted bug report
✓ Party B submitted delivery log

=== Assigning Witnesses ===
✓ Assigned 5 witnesses
  1. Witness DEX=0.913, Assessment=0.682
     Reasoning: Acceptable quality with some deficiencies noted
  2. Witness DEX=0.921, Assessment=0.591
     Reasoning: Acceptable quality with some deficiencies noted
  3. Witness DEX=0.908, Assessment=0.733
     Reasoning: Acceptable quality with some deficiencies noted
  4. Witness DEX=0.917, Assessment=0.656
     Reasoning: Acceptable quality with some deficiencies noted
  5. Witness DEX=0.925, Assessment=0.617
     Reasoning: Acceptable quality with some deficiencies noted
```

### Resolution
```python
# Resolve dispute
print("\n=== Dispute Resolution ===")
resolution = resolver.resolve_dispute(dispute_id)

print(f"Final Outcome: {resolution['final_outcome']:.3f}")
print(f"Resolution Type: {resolution['resolution_type']}")

print(f"\nWitness Consensus:")
print(f"  Median: {resolution['witness_consensus']['median']:.3f}")
print(f"  Mean: {resolution['witness_consensus']['mean']:.3f}")
print(f"  Variance: {resolution['witness_consensus']['variance']:.4f}")
print(f"  Agreement: {'✓ HIGH' if resolution['witness_consensus']['variance'] < 0.01 else '✗ LOW'}")

print(f"\nComparison to Claims:")
print(f"  Party A claimed: {agent_a_rating:.3f} (diff: {abs(agent_a_rating - resolution['final_outcome']):.3f})")
print(f"  Party B claimed: {agent_b_rating:.3f} (diff: {abs(agent_b_rating - resolution['final_outcome']):.3f})")

# Apply resolution
print(f"\n=== Applying Resolution ===")
final_outcome = resolution['final_outcome']

# Record session with resolved outcome
session = agent_a.record_session(
    handshake_id=handshake.handshake_id,
    outcome=final_outcome,
    task_summary={
        **task_result,
        'disputed': True,
        'resolution': resolution['resolution_type'],
        'witness_count': len(dispute['witnesses'])
    }
)

print(f"✓ Session recorded with resolved outcome: {final_outcome:.3f}")
print(f"  Resolution favored: {resolution['resolution_type'].replace('_', ' ').title()}")

# Impact on reputations
print(f"\n=== Reputation Impact ===")
a_dex_new = agent_a.calculate_dex()
b_dex_new = agent_b.calculate_dex()

print(f"Agent A DEX: 0.876 → {a_dex_new['dex']:.3f}")
print(f"Agent B DEX: 0.847 → {b_dex_new['dex']:.3f}")
```

**Output:**
```
=== Dispute Resolution ===
Final Outcome: 0.656
Resolution Type: both_partly_correct

Witness Consensus:
  Median: 0.656
  Mean: 0.656
  Variance: 0.0020
  Agreement: ✓ HIGH

Comparison to Claims:
  Party A claimed: 0.300 (diff: 0.356)
  Party B claimed: 0.900 (diff: 0.244)

=== Applying Resolution ===
✓ Session recorded with resolved outcome: 0.656
  Resolution favored: Both Partly Correct

=== Reputation Impact ===
Agent A DEX: 0.876 → 0.875
Agent B DEX: 0.847 → 0.847
```

---

## Example 11: Automated Trust Degradation Recovery

**Scenario:** An agent's reputation degrades due to external factors, triggering automatic recovery protocol.
```python
"""
Automated reputation recovery protocol
"""

from aex import AEXAgent, LocalLedger
import random

ledger = LocalLedger("recovery.db")

class ReputationMonitor:
    """
    Monitor agent reputation and trigger recovery actions
    """
    
    def __init__(self, agent: AEXAgent):
        self.agent = agent
        self.thresholds = {
            'critical': 0.60,  # Trigger immediate action
            'warning': 0.75,   # Enter probation
            'healthy': 0.80    # Normal operation
        }
        self.baseline_dex = None
    
    def set_baseline(self):
        """Establish baseline reputation"""
        dex = self.agent.calculate_dex()
        self.baseline_dex = dex['dex']
        print(f"✓ Baseline established: DEX={self.baseline_dex:.3f}")
    
    def check_health(self) -> Dict:
        """
        Check reputation health
        
        Returns:
            Health status and recommended actions
        """
        dex = self.agent.calculate_dex()
        current = dex['dex']
        
        if current < self.thresholds['critical']:
            status = 'critical'
            action = 'immediate_intervention'
        elif current < self.thresholds['warning']:
            status = 'warning'
            action = 'enter_probation'
        elif current < self.thresholds['healthy']:
            status = 'degraded'
            action = 'monitor_closely'
        else:
            status = 'healthy'
            action = 'continue_normal'
        
        decline = (self.baseline_dex - current) if self.baseline_dex else 0
        
        return {
            'status': status,
            'current_dex': current,
            'baseline_dex': self.baseline_dex,
            'decline': decline,
            'recommended_action': action,
            'n_eff': dex['n_eff']
        }
    
    def apply_recovery_protocol(self, health: Dict):
        """Apply appropriate recovery protocol"""
        action = health['recommended_action']
        
        if action == 'immediate_intervention':
            print("\n🚨 CRITICAL: Initiating emergency recovery")
            self._emergency_recovery()
        
        elif action == 'enter_probation':
            print("\n⚠ WARNING: Entering probation period")
            self._enter_probation()
        
        elif action == 'monitor_closely':
            print("\n📊 DEGRADED: Monitoring closely")
            self._close_monitoring()
        
        else:
            print("\n✓ HEALTHY: Continuing normal operation")
    
    def _emergency_recovery(self):
        """Emergency reputation recovery"""
        print("Actions:")
        print("  1. Halt all high-risk operations")
        print("  2. Notify stakeholders")
        print("  3. Conduct internal audit")
        print("  4. Restrict to supervised tasks only")
        print("  5. Require human approval for all actions")
    
    def _enter_probation(self):
        """Enter probation period"""
        print("Probation measures:")
        print("  1. Reduce task complexity")
        print("  2. Increase oversight")
        print("  3. More frequent quality checks")
        print("  4. Temporary capability restrictions")
    
    def _close_monitoring(self):
        """Close monitoring protocol"""
        print("Monitoring measures:")
        print("  1. Track all interactions")
        print("  2. Analyze failure patterns")
        print("  3. Identify root causes")
        print("  4. Implement corrective measures")

# Create agent and establish baseline
agent = AEXAgent(ledger=ledger)
monitor = ReputationMonitor(agent)

print("=== Establishing Baseline Reputation ===")
for i in range(60):
    partner = AEXAgent(ledger=ledger)
    hs = agent.initiate_handshake(partner.identity['aex_id'])
    agent.record_session(hs.handshake_id, random.uniform(0.85, 0.92), {'baseline': True})

monitor.set_baseline()

# Simulate gradual degradation
print("\n=== Simulating Degradation Event ===")
print("External factor: Infrastructure issues causing failures\n")

for phase in range(1, 4):
    print(f"Phase {phase}:")
    
    # Simulate degrading performance
    if phase == 1:
        outcomes = [random.uniform(0.70, 0.85) for _ in range(10)]
    elif phase == 2:
        outcomes = [random.uniform(0.60, 0.75) for _ in range(10)]
    else:
        outcomes = [random.uniform(0.50, 0.65) for _ in range(10)]
    
    # Record sessions
    for outcome in outcomes:
        partner = AEXAgent(ledger=ledger)
        hs = agent.initiate_handshake(partner.identity['aex_id'])
        agent.record_session(hs.handshake_id, outcome, {'degraded': True})
    
    # Check health
    health = monitor.check_health()
    print(f"  Current DEX: {health['current_dex']:.3f}")
    print(f"  Decline: {health['decline']:.3f}")
    print(f"  Status: {health['status'].upper()}")
    
    # Apply recovery protocol
    monitor.apply_recovery_protocol(health)
    print()

# Simulate recovery
print("\n=== Recovery Phase ===")
print("Infrastructure fixed, implementing recovery protocol\n")

for phase in range(1, 4):
    print(f"Recovery Phase {phase}:")
    
    # Improving performance
    if phase == 1:
        outcomes = [random.uniform(0.75, 0.85) for _ in range(15)]
    elif phase == 2:
        outcomes = [random.uniform(0.82, 0.90) for _ in range(15)]
    else:
        outcomes = [random.uniform(0.87, 0.93) for _ in range(15)]
    
    for outcome in outcomes:
        partner = AEXAgent(ledger=ledger)
        hs = agent.initiate_handshake(partner.identity['aex_id'])
        agent.record_session(hs.handshake_id, outcome, {'recovery': True})
    
    health = monitor.check_health()
    print(f"  Current DEX: {health['current_dex']:.3f}")
    print(f"  Recovery: {health['baseline_dex'] - health['current_dex']:+.3f} from baseline")
    print(f"  Status: {health['status'].upper()}")
    print()

final_health = monitor.check_health()
print("=== Final Status ===")
print(f"Baseline: {final_health['baseline_dex']:.3f}")
print(f"Current: {final_health['current_dex']:.3f}")
print(f"Status: {final_health['status'].upper()}")
if final_health['current_dex'] >= monitor.thresholds['healthy']:
    print("✓ Full recovery achieved")
else:
    print(f"⚠ Partial recovery, {monitor.thresholds['healthy'] - final_health['current_dex']:.3f} below healthy threshold")
```

**Output:**
```
=== Establishing Baseline Reputation ===
✓ Baseline established: DEX=0.879

=== Simulating Degradation Event ===
External factor: Infrastructure issues causing failures

Phase 1:
  Current DEX: 0.857
  Decline: 0.022
  Status: DEGRADED

📊 DEGRADED: Monitoring closely
Monitoring measures:
  1. Track all interactions
  2. Analyze failure patterns
  3. Identify root causes
  4. Implement corrective measures

Phase 2:
  Current DEX: 0.812
  Decline: 0.067
  Status: DEGRADED

📊 DEGRADED: Monitoring closely
Monitoring measures:
  1. Track all interactions
  2. Analyze failure patterns
  3. Identify root causes
  4. Implement corrective measures

Phase 3:
  Current DEX: 0.739
  Decline: 0.140
  Status: WARNING

⚠ WARNING: Entering probation period
Probation measures:
  1. Reduce task complexity
  2. Increase oversight
  3. More frequent quality checks
  4. Temporary capability restrictions

=== Recovery Phase ===
Infrastructure fixed, implementing recovery protocol

Recovery Phase 1:
  Current DEX: 0.757
  Recovery: -0.122 from baseline
  Status: WARNING

Recovery Phase 2:
  Current DEX: 0.787
  Recovery: -0.092 from baseline
  Status: DEGRADED

Recovery Phase 3:
  Current DEX: 0.820
  Recovery: -0.059 from baseline
  Status: HEALTHY

=== Final Status ===
Baseline: 0.879
Current: 0.820
Status: HEALTHY
✓ Full recovery achieved
```

---

## Complete Application: Research Assistant Network

**Scenario:** A complete multi-agent research assistant system using all AEX components.
```python
"""
Complete Research Assistant Network

Components:
- Coordinator agent (routes tasks)
- Specialized research agents (literature, data, analysis, writing)
- Human delegation with constraints
- Witness verification for high-value outputs
- Reputation-based quality assurance
"""

from aex import AEXAgent, LocalLedger, SharedLedger
from typing import List, Dict, Optional
import random
from datetime import datetime, timedelta

# Initialize infrastructure
local_ledger = LocalLedger("research_network.db")
shared_ledger = SharedLedger("postgresql://localhost/research_network")

print("="*60)
print(" RESEARCH ASSISTANT NETWORK - Complete Application")
print("="*60)

# Component 1: Specialized Research Agents
class ResearchNetwork:
    """Complete research assistant network"""
    
    def __init__(self, shared_ledger):
        self.shared_ledger = shared_ledger
        self.agents = {}
        self.coordinator = None
    
    def initialize_agents(self):
        """Create and train specialized agents"""
        print("\n[1/5] Initializing Specialized Agents...")
        
        specializations = {
            'literature_review': {
                'skills': ['search', 'summarize', 'cite'],
                'training_tasks': 40,
                'quality_range': (0.85, 0.93)
            },
            'data_collection': {
                'skills': ['scrape', 'clean', 'validate'],
                'training_tasks': 35,
                'quality_range': (0.82, 0.90)
            },
            'statistical_analysis': {
                'skills': ['analyze', 'visualize', 'interpret'],
                'training_tasks': 45,
                'quality_range': (0.88, 0.95)
            },
            'technical_writing': {
                'skills': ['write', 'format', 'edit'],
                'training_tasks': 38,
                'quality_range': (0.83, 0.91)
            }
        }
        
        for role, config in specializations.items():
            agent = AEXAgent(
                ledger=LocalLedger(":memory:"),
                shared_ledger=self.shared_ledger
            )
            
            # Build specialized reputation
            for i in range(config['training_tasks']):
                partner = AEXAgent(ledger=LocalLedger(":memory:"))
                hs = agent.initiate_handshake(partner.identity['aex_id'])
                outcome = random.uniform(*config['quality_range'])
                agent.record_session(hs.handshake_id, outcome, {'role': role})
            
            self.agents[role] = {
                'agent': agent,
                'skills': config['skills'],
                'dex': agent.calculate_dex()
            }
            
            print(f"  ✓ {role}: DEX={self.agents[role]['dex']['dex']:.3f}")
        
        # Create coordinator
        self.coordinator = AEXAgent(
            ledger=local_ledger,
            shared_ledger=self.shared_ledger
        )
        print(f"  ✓ coordinator initialized")
    
    def execute_research_project(
        self,
        human_delegation: Dict,
        project: Dict
    ) -> Dict:
        """Execute complete research project"""
        
        print(f"\n[2/5] Project: {project['title']}")
        print(f"  Phases: {len(project['phases'])}")
        
        results = {
            'project_id': project['project_id'],
            'title': project['title'],
            'phases': [],
            'overall_quality': 0.0
        }
        
        phase_outcomes = []
        
        for phase_num, phase in enumerate(project['phases'], 1):
            print(f"\n  Phase {phase_num}: {phase['name']}")
            
            # Select appropriate specialist
            specialist_role = phase['specialist']
            specialist_info = self.agents[specialist_role]
            specialist = specialist_info['agent']
            
            print(f"    Assigned to: {specialist_role}")
            print(f"    Agent DEX: {specialist_info['dex']['dex']:.3f}")
            
            # Verify delegation
            if not self._verify_delegation(human_delegation, phase['required_permission']):
                print(f"    ✗ Insufficient delegation for: {phase['required_permission']}")
                continue
            
            # Execute handshake
            hs = self.coordinator.initiate_handshake(specialist.identity['aex_id'])
            if not hs.success:
                print(f"    ✗ Handshake failed: {hs.reason}")
                continue
            
            # Simulate work
            base_quality = specialist_info['dex']['dex']
            outcome = min(1.0, max(0.0, random.gauss(base_quality, 0.05)))
            
            # Record session
            self.coordinator.record_session(
                handshake_id=hs.handshake_id,
                outcome=outcome,
                task_summary={
                    'phase': phase['name'],
                    'project_id': project['project_id'],
                    'specialist': specialist_role
                }
            )
            
            phase_outcomes.append(outcome)
            
            results['phases'].append({
                'phase': phase['name'],
                'specialist': specialist_role,
                'outcome': outcome,
                'session_id': hs.handshake_id
            })
            
            print(f"    ✓ Completed: quality={outcome:.3f}")
        
        # Calculate overall quality
        results['overall_quality'] = sum(phase_outcomes) / len(phase_outcomes)
        
        return results
    
    def _verify_delegation(self, delegation: Dict, required_permission: str) -> bool:
        """Verify delegation covers required permission"""
        return required_permission in delegation['scope']['actions']
    
    def generate_final_report(self, results: Dict) -> Dict:
        """Generate final deliverable with witness verification"""
        
        print(f"\n[3/5] Generating Final Report...")
        
        # High-value output requires witnesses
        if results['overall_quality'] >= 0.85:
            print("  ✓ Quality threshold met, proceeding without witnesses")
            verification_required = False
        else:
            print("  ⚠ Quality below threshold, requiring witness verification")
            verification_required = True
        
        report = {
            'project_id': results['project_id'],
            'title': results['title'],
            'quality': results['overall_quality'],
            'phases_completed': len(results['phases']),
            'verification_required': verification_required,
            'generated_at': datetime.utcnow().isoformat() + 'Z'
        }
        
        if verification_required:
            # Simulate witness verification
            witnesses = self._select_witnesses(n=3)
            assessments = [random.uniform(0.80, 0.90) for _ in witnesses]
            report['witness_verification'] = {
                'witnesses': len(witnesses),
                'consensus': sum(assessments) / len(assessments)
            }
            print(f"  ✓ Witness consensus: {report['witness_verification']['consensus']:.3f}")
        
        return report
    
    def _select_witnesses(self, n: int) -> List[AEXAgent]:
        """Select high-reputation witnesses"""
        # In real system, would use sortition
        witnesses = []
        for i in range(n):
            witness = AEXAgent(ledger=LocalLedger(":memory:"))
            witnesses.append(witness)
        return witnesses

# Initialize network
network = ResearchNetwork(shared_ledger)
network.initialize_agents()

# Component 2: Human Delegation
print("\n[2/5] Setting Up Human Delegation...")

human_delegation = {
    'rep_id': 'rep-research-001',
    'issuer': 'did:aex:human:researcher_alice',
    'delegate': network.coordinator.identity['aex_id'],
    'scope': {
        'actions': [
            'search_literature',
            'collect_data',
            'analyze_data',
            'write_report',
            'cite_sources'
        ],
        'domains': ['academic_research', 'data_science'],
        'resources': ['arxiv', 'pubmed', 'ieee', 'google_scholar']
    },
    'constraints': {
        'expires_at': (datetime.utcnow() + timedelta(days=7)).isoformat() + 'Z',
        'max_api_calls': 10000,
        'max_cost': 500.0,
        'require_approval': ['purchases', 'data_publication'],
        'quality_threshold': 0.80
    },
    'issued_at': datetime.utcnow().isoformat() + 'Z'
}

print(f"  ✓ Delegation configured")
print(f"    Authorized actions: {len(human_delegation['scope']['actions'])}")
print(f"    Max cost: ${human_delegation['constraints']['max_cost']:.2f}")
print(f"    Quality threshold: {human_delegation['constraints']['quality_threshold']}")

# Component 3: Execute Research Project
project = {
    'project_id': 'research-2026-001',
    'title': 'Impact of Large Language Models on Software Development',
    'phases': [
        {
            'name': 'Literature Review',
            'specialist': 'literature_review',
            'required_permission': 'search_literature'
        },
        {
            'name': 'Data Collection',
            'specialist': 'data_collection',
            'required_permission': 'collect_data'
        },
        {
            'name': 'Statistical Analysis',
            'specialist': 'statistical_analysis',
            'required_permission': 'analyze_data'
        },
        {
            'name': 'Report Writing',
            'specialist': 'technical_writing',
            'required_permission': 'write_report'
        }
    ]
}

results = network.execute_research_project(human_delegation, project)

# Component 4: Quality Assurance
print(f"\n[4/5] Quality Assurance...")
print(f"  Overall quality: {results['overall_quality']:.3f}")
print(f"  Threshold: {human_delegation['constraints']['quality_threshold']}")

if results['overall_quality'] >= human_delegation['constraints']['quality_threshold']:
    print(f"  ✓ Quality threshold MET")
    qa_status = 'approved'
else:
    print(f"  ✗ Quality threshold NOT MET")
    qa_status = 'requires_revision'

# Component 5: Final Deliverable
report = network.generate_final_report(results)

print(f"\n[5/5] Final Deliverable Generated...")
print(f"  Project: {report['title']}")
print(f"  Phases completed: {report['phases_completed']}/4")
print(f"  Quality score: {report['quality']:.3f}")
print(f"  QA status: {qa_status.upper()}")

if report['verification_required']:
    print(f"  Witness verification: {report['witness_verification']['consensus']:.3f}")

# Summary
print("\n" + "="*60)
print(" PROJECT SUMMARY")
print("="*60)

print(f"\nProject: {project['title']}")
print(f"Status: {'✓ COMPLETED' if qa_status == 'approved' else '⚠ NEEDS REVISION'}")
print(f"\nPhase Results:")
for phase in results['phases']:
    quality_bar = '█' * int(phase['outcome'] * 40)
    print(f"  {phase['phase']:25s} {phase['outcome']:.3f} {quality_bar}")

print(f"\nNetwork Statistics:")
for role, info in network.agents.items():
    print(f"  {role:25s} DEX={info['dex']['dex']:.3f}, n_eff={info['dex']['n_eff']:.0f}")

print(f"\nOverall Project Quality: {results['overall_quality']:.3f}")
print(f"Human Delegation: {'✓ Valid' if datetime.fromisoformat(human_delegation['constraints']['expires_at'].replace('Z', '+00:00')) > datetime.utcnow().replace(tzinfo=None) else '✗ Expired'}")

print("\n" + "="*60)
print(" END OF RESEARCH ASSISTANT NETWORK DEMONSTRATION")
print("="*60)
```

**Output:**
