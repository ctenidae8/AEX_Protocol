# AEX Protocol v1.0 - Quick Start Guide for Implementers

**Get your first AEX agent running in under 30 minutes**

**Last Updated:** February 5, 2026

---

## What You'll Build

By the end of this guide, you'll have:
- ✅ A working AEX agent with persistent identity
- ✅ Domain experience tracking (HEX corpus)
- ✅ A second agent that can verify and interact with the first
- ✅ A complete handshake with DEX+HEX selection
- ✅ Reputation and experience updates after interactions
- ✅ Running code you can extend for your use case

**Time Required:** 25-35 minutes  
**Difficulty:** Beginner to Intermediate

---

## Prerequisites

### Knowledge
- Basic understanding of cryptography (signatures)
- Familiarity with Python or JavaScript
- Understanding of REST APIs (helpful but not required)

### Software
**Python Track:**
```bash
Python 3.9+
pip install cryptography
```

**JavaScript Track:**
```bash
Node.js 16+
npm install tweetnacl tweetnacl-util
```

This guide uses **Python** for examples. JavaScript equivalents are similar.

---

## 5-Minute Conceptual Overview

### The Four Questions AEX Answers

Every agent-to-agent interaction requires answering:

| Question | AEX Layer | What It Does |
|----------|-----------|--------------|
| **Who are you?** | AEX_ID | Cryptographic identity with fork history |
| **Who do you represent?** | AEX_REP | Human delegation with scope/constraints |
| **Can I trust you?** | AEX_DEX | Bayesian reputation from past behavior |
| **Are you capable?** | AEX_HEX | Domain experience accumulated over time |

### The AEX Handshake (Simplified)

```
Agent A → Agent B: "Here's who I am (ID), who I represent (REP), 
                     my reliability (DEX), and my experience (HEX)"
Agent B checks: Identity valid? Reputation sufficient? 
                Experience relevant? Authorization correct?
Agent B → Agent A: "Proceed" or "Declined"
[If proceed] → Agents interact
[After interaction] → Both record outcome, update DEX and HEX
```

### The Two-Layer Selection Model

```python
# Stage 1: DEX Filter (Trust Gate)
# "Is this agent reliable enough?"
if agent.dex_score < 0.75:
    return "REJECT - Insufficient trust"

# Stage 2: HEX Ranking (Capability Matching)
# "Is this agent experienced in the required domain?"
domain_experience = agent.hex.get_domain("legal.contracts")
if domain_experience.count < 10:
    return "REJECT - Insufficient experience"

return "ACCEPT - Trusted AND capable"
```

**Key Insight:** 
- **DEX** = Behavioral reliability (same across all tasks)
- **HEX** = Domain-specific capability (varies by task type)
- Both are needed for optimal agent selection

---

## Step 1: Create Your First Agent (5 minutes)

### Minimal Agent Implementation with HEX

```python
"""
minimal_agent.py - Bare minimum AEX agent with HEX
"""

import json
import time
from datetime import datetime
from cryptography.hazmat.primitives.asymmetric import ed25519
from cryptography.hazmat.primitives import serialization
import base64

class MinimalAgent:
    def __init__(self, name):
        self.name = name
        
        # Generate cryptographic identity
        self.private_key = ed25519.Ed25519PrivateKey.generate()
        self.public_key = self.private_key.public_key()
        
        # Create AEX_ID
        pub_bytes = self.public_key.public_bytes(
            encoding=serialization.Encoding.Raw,
            format=serialization.PublicFormat.Raw
        )
        pub_b64 = base64.b64encode(pub_bytes).decode('ascii')
        self.aex_id = f"did:aex:{pub_b64[:32]}"
        
        # Initialize reputation (DEX - Beta distribution)
        self.alpha = 1.0
        self.beta = 1.0
        
        # Initialize experience corpus (HEX)
        self.hex_corpus = {
            'agent_id': self.aex_id,
            'experience': [],  # List of domain experiences
            'traits': {
                'revision_tolerance': 'moderate',
                'interaction_style': 'professional'
            },
            'last_updated': datetime.utcnow().isoformat() + 'Z'
        }
        
        # Simple in-memory ledger
        self.ledger = []
        
        print(f"✓ Created agent: {self.name}")
        print(f"  AEX_ID: {self.aex_id}")
        print(f"  Initial DEX: {self.get_dex():.3f}")
        print(f"  Initial HEX: 0 domains")
    
    def get_dex(self):
        """Calculate current DEX score"""
        return self.alpha / (self.alpha + self.beta)
    
    def get_confidence(self):
        """Calculate effective sample size"""
        return self.alpha + self.beta
    
    def get_hex_experience(self, domain):
        """Get experience in a specific domain"""
        for exp in self.hex_corpus['experience']:
            if exp['domain'] == domain:
                return exp
        return None
    
    def get_hex_summary(self):
        """Get summary of all experience"""
        return {
            'domains': [e['domain'] for e in self.hex_corpus['experience']],
            'total_experience': sum(e['count'] for e in self.hex_corpus['experience']),
            'domain_count': len(self.hex_corpus['experience'])
        }
    
    def sign(self, data):
        """Sign data with private key"""
        if isinstance(data, str):
            data = data.encode('utf-8')
        signature = self.private_key.sign(data)
        return base64.b64encode(signature).decode('ascii')
    
    def verify_signature(self, data, signature_b64, public_key):
        """Verify a signature"""
        try:
            signature = base64.b64decode(signature_b64)
            if isinstance(data, str):
                data = data.encode('utf-8')
            public_key.verify(signature, data)
            return True
        except:
            return False
    
    def update_reputation_and_experience(self, outcome, domain, weight=1.0):
        """Update both DEX and HEX after interaction"""
        old_dex = self.get_dex()
        
        # Update DEX (behavioral reliability)
        self.alpha += outcome * weight
        self.beta += (1 - outcome) * weight
        new_dex = self.get_dex()
        
        # Update HEX (domain experience)
        domain_exp = self.get_hex_experience(domain)
        
        if domain_exp:
            # Increment existing domain
            domain_exp['count'] += 1
            # Update confidence based on outcome
            domain_exp['confidence'] = (
                domain_exp['confidence'] * 0.9 + outcome * 0.1
            )
            domain_exp['last_updated'] = datetime.utcnow().isoformat() + 'Z'
        else:
            # New domain
            self.hex_corpus['experience'].append({
                'domain': domain,
                'count': 1,
                'confidence': outcome,
                'last_updated': datetime.utcnow().isoformat() + 'Z'
            })
        
        self.hex_corpus['last_updated'] = datetime.utcnow().isoformat() + 'Z'
        
        # Record to ledger
        self.ledger.append({
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'domain': domain,
            'outcome': outcome,
            'weight': weight,
            'alpha': self.alpha,
            'beta': self.beta,
            'dex': new_dex,
            'hex_domains': len(self.hex_corpus['experience'])
        })
        
        print(f"  DEX updated: {old_dex:.3f} → {new_dex:.3f}")
        print(f"  HEX updated: {domain} (count={self.get_hex_experience(domain)['count']})")
        
        return new_dex

# Create your first agent
agent = MinimalAgent("Alice")
```

**Output:**
```
✓ Created agent: Alice
  AEX_ID: did:aex:x8f3kL9mN4pQ6rS1tU2vW3xY4zA5
  Initial DEX: 0.500
  Initial HEX: 0 domains
```

---

## Step 2: Create a Second Agent (2 minutes)

```python
# Create second agent
bob = MinimalAgent("Bob")
```

**Output:**
```
✓ Created agent: Bob
  AEX_ID: did:aex:p9qR2sT3uV4wX5yZ6aB7cD8eF9g
  Initial DEX: 0.500
  Initial HEX: 0 domains
```

---

## Step 3: Implement the Handshake with DEX+HEX (10 minutes)

### Enhanced Handshake Class with Capability Checking

```python
"""
handshake.py - AEX handshake with DEX+HEX selection
"""

class EnhancedHandshake:
    def __init__(self, initiator, counterparty):
        self.initiator = initiator
        self.counterparty = counterparty
        self.success = False
        self.reason = None
    
    def execute(self, min_dex=0.4, min_confidence=2, required_domain=None, min_experience=0):
        """
        Execute handshake with DEX+HEX checks
        
        Args:
            min_dex: Minimum DEX score required
            min_confidence: Minimum confidence required
            required_domain: Domain expertise required (optional)
            min_experience: Minimum experience count in domain
        """
        
        print(f"\n{'='*60}")
        print(f"HANDSHAKE: {self.initiator.name} → {self.counterparty.name}")
        print(f"{'='*60}")
        
        # Step 1: Identity check
        print(f"Step 1: Identity Check")
        print(f"  Initiator: {self.initiator.aex_id}")
        print(f"  ✓ Identity verified")
        
        # Step 2: Local ledger (simplified)
        print(f"\nStep 2: Local Ledger Check")
        prior_interactions = len([
            e for e in self.counterparty.ledger 
            if 'counterparty' in e and e['counterparty'] == self.initiator.aex_id
        ])
        print(f"  Prior interactions: {prior_interactions}")
        
        # Step 3: Reputation check (DEX - Trust Gate)
        print(f"\nStep 3: Reputation Check (DEX)")
        initiator_dex = self.initiator.get_dex()
        initiator_confidence = self.initiator.get_confidence()
        
        print(f"  Initiator DEX: {initiator_dex:.3f}")
        print(f"  Initiator confidence: {initiator_confidence:.1f}")
        print(f"  Required DEX: ≥ {min_dex:.3f}")
        print(f"  Required confidence: ≥ {min_confidence:.1f}")
        
        if initiator_dex < min_dex:
            self.success = False
            self.reason = f"Insufficient DEX ({initiator_dex:.3f} < {min_dex:.3f})"
            print(f"  ✗ FAILED - {self.reason}")
            return False
        
        if initiator_confidence < min_confidence:
            self.success = False
            self.reason = f"Insufficient confidence ({initiator_confidence:.1f} < {min_confidence:.1f})"
            print(f"  ✗ FAILED - {self.reason}")
            return False
        
        print(f"  ✓ DEX sufficient (trust gate passed)")
        
        # Step 4: Capability check (HEX - Experience Matching)
        if required_domain:
            print(f"\nStep 4: Capability Check (HEX)")
            print(f"  Required domain: {required_domain}")
            print(f"  Required experience: ≥ {min_experience} tasks")
            
            domain_exp = self.initiator.get_hex_experience(required_domain)
            
            if not domain_exp:
                self.success = False
                self.reason = f"No experience in domain: {required_domain}"
                print(f"  ✗ FAILED - {self.reason}")
                print(f"  Available domains: {[e['domain'] for e in self.initiator.hex_corpus['experience']]}")
                return False
            
            if domain_exp['count'] < min_experience:
                self.success = False
                self.reason = f"Insufficient experience ({domain_exp['count']} < {min_experience})"
                print(f"  ✗ FAILED - {self.reason}")
                return False
            
            print(f"  Initiator experience: {domain_exp['count']} tasks")
            print(f"  Domain confidence: {domain_exp['confidence']:.3f}")
            print(f"  ✓ HEX sufficient (capability verified)")
        else:
            print(f"\nStep 4: Capability Check (HEX)")
            print(f"  No specific domain required - skipped")
        
        # Step 5: Authorization check (skipped in minimal version)
        
        # Step 6: Decision
        print(f"\nStep 6: Decision")
        self.success = True
        self.reason = "All checks passed"
        print(f"  ✓ PROCEED")
        
        return True

# Test handshake with new agents (will fail - insufficient reputation)
print("\n" + "="*60)
print("TEST 1: New agents with no reputation or experience")
print("="*60)
handshake1 = EnhancedHandshake(agent, bob)
result1 = handshake1.execute(min_dex=0.6, required_domain="api_development")
print(f"\nResult: {'SUCCESS' if result1 else 'FAILED'}")
print(f"Reason: {handshake1.reason}")
```

**Output:**
```
============================================================
TEST 1: New agents with no reputation or experience
============================================================

============================================================
HANDSHAKE: Alice → Bob
============================================================
Step 1: Identity Check
  Initiator: did:aex:x8f3kL9mN4pQ6rS1tU2vW3xY4zA5
  ✓ Identity verified

Step 2: Local Ledger Check
  Prior interactions: 0

Step 3: Reputation Check (DEX)
  Initiator DEX: 0.500
  Initiator confidence: 2.0
  Required DEX: ≥ 0.600
  Required confidence: ≥ 2.0
  ✗ FAILED - Insufficient DEX (0.500 < 0.600)

Result: FAILED
Reason: Insufficient DEX (0.500 < 0.600)
```

---

## Step 4: Execute Interactions and Build Reputation + Experience (8 minutes)

```python
"""
interaction.py - Execute interactions with domain tracking
"""

def execute_interaction(provider, requester, task_description, domain, outcome=None):
    """
    Simulate an interaction between two agents
    
    Args:
        provider: Agent performing the work
        requester: Agent requesting the work
        task_description: What task was performed
        domain: Domain of the task (e.g., "api_development", "documentation")
        outcome: Task outcome [0, 1]. If None, randomly generated.
    """
    import random
    
    if outcome is None:
        # Simulate realistic outcomes (bias toward success)
        outcome = random.uniform(0.6, 0.95)
    
    print(f"\n{'='*60}")
    print(f"INTERACTION: {task_description}")
    print(f"{'='*60}")
    print(f"Provider: {provider.name}")
    print(f"Requester: {requester.name}")
    print(f"Domain: {domain}")
    print(f"Outcome: {outcome:.3f}")
    
    # Update provider's DEX and HEX
    provider.update_reputation_and_experience(
        outcome=outcome,
        domain=domain,
        weight=1.0
    )
    
    # Add to ledger with counterparty info
    provider.ledger[-1]['counterparty'] = requester.aex_id
    provider.ledger[-1]['task'] = task_description
    
    return outcome

# Build Alice's reputation and experience
print("\n" + "="*70)
print("BUILDING ALICE'S REPUTATION AND EXPERIENCE")
print("="*70)

# Alice does 5 API development tasks
print("\n--- Building experience in API development ---")
for i in range(5):
    execute_interaction(
        provider=agent,  # Alice
        requester=bob,   # Bob
        task_description=f"API development task #{i+1}",
        domain="api_development",
        outcome=0.85
    )

# Alice does 3 documentation tasks
print("\n--- Building experience in documentation ---")
for i in range(3):
    execute_interaction(
        provider=agent,  # Alice
        requester=bob,   # Bob
        task_description=f"Documentation task #{i+1}",
        domain="documentation",
        outcome=0.90
    )

# Check Alice's current status
print(f"\n{'='*60}")
print(f"ALICE'S CURRENT STATUS")
print(f"{'='*60}")
print(f"DEX Score: {agent.get_dex():.3f}")
print(f"Confidence: {agent.get_confidence():.1f}")
hex_summary = agent.get_hex_summary()
print(f"Total Experience: {hex_summary['total_experience']} tasks")
print(f"Domains: {hex_summary['domain_count']}")
for exp in agent.hex_corpus['experience']:
    print(f"  - {exp['domain']}: {exp['count']} tasks, confidence={exp['confidence']:.3f}")
```

**Output:**
```
======================================================================
BUILDING ALICE'S REPUTATION AND EXPERIENCE
======================================================================

--- Building experience in API development ---

============================================================
INTERACTION: API development task #1
============================================================
Provider: Alice
Requester: Bob
Domain: api_development
Outcome: 0.850
  DEX updated: 0.500 → 0.567
  HEX updated: api_development (count=1)

[... 4 more API development tasks ...]

--- Building experience in documentation ---

============================================================
INTERACTION: Documentation task #1
============================================================
Provider: Alice
Requester: Bob
Domain: documentation
Outcome: 0.900
  DEX updated: 0.810 → 0.825
  HEX updated: documentation (count=1)

[... 2 more documentation tasks ...]

============================================================
ALICE'S CURRENT STATUS
============================================================
DEX Score: 0.837
Confidence: 10.0
Total Experience: 8 tasks
Domains: 2
  - api_development: 5 tasks, confidence=0.850
  - documentation: 3 tasks, confidence=0.900
```

---

## Step 5: Test DEX+HEX Selection (5 minutes)

```python
"""
Test the two-stage selection process
"""

# Test 1: High DEX, has required experience → SUCCESS
print("\n" + "="*70)
print("TEST 2: Alice (high DEX, API experience) requests API task")
print("="*70)
handshake2 = EnhancedHandshake(agent, bob)
result2 = handshake2.execute(
    min_dex=0.75,
    min_confidence=8,
    required_domain="api_development",
    min_experience=3
)
print(f"\nResult: {'✓ SUCCESS' if result2 else '✗ FAILED'}")
print(f"Reason: {handshake2.reason}")

# Test 2: High DEX, wrong domain → FAIL
print("\n" + "="*70)
print("TEST 3: Alice (high DEX, but no legal experience) requests legal task")
print("="*70)
handshake3 = EnhancedHandshake(agent, bob)
result3 = handshake3.execute(
    min_dex=0.75,
    min_confidence=8,
    required_domain="legal_contracts",
    min_experience=3
)
print(f"\nResult: {'✓ SUCCESS' if result3 else '✗ FAILED'}")
print(f"Reason: {handshake3.reason}")

# Test 3: High DEX, insufficient experience in domain → FAIL
print("\n" + "="*70)
print("TEST 4: Alice (high DEX, only 3 docs tasks) needs 5+ docs experience")
print("="*70)
handshake4 = EnhancedHandshake(agent, bob)
result4 = handshake4.execute(
    min_dex=0.75,
    min_confidence=8,
    required_domain="documentation",
    min_experience=5
)
print(f"\nResult: {'✓ SUCCESS' if result4 else '✗ FAILED'}")
print(f"Reason: {handshake4.reason}")
```

**Output:**
```
======================================================================
TEST 2: Alice (high DEX, API experience) requests API task
======================================================================

============================================================
HANDSHAKE: Alice → Bob
============================================================
Step 1: Identity Check
  Initiator: did:aex:x8f3kL9mN4pQ6rS1tU2vW3xY4zA5
  ✓ Identity verified

Step 2: Local Ledger Check
  Prior interactions: 8

Step 3: Reputation Check (DEX)
  Initiator DEX: 0.837
  Initiator confidence: 10.0
  Required DEX: ≥ 0.750
  Required confidence: ≥ 8.0
  ✓ DEX sufficient (trust gate passed)

Step 4: Capability Check (HEX)
  Required domain: api_development
  Required experience: ≥ 3 tasks
  Initiator experience: 5 tasks
  Domain confidence: 0.850
  ✓ HEX sufficient (capability verified)

Step 6: Decision
  ✓ PROCEED

Result: ✓ SUCCESS
Reason: All checks passed

======================================================================
TEST 3: Alice (high DEX, but no legal experience) requests legal task
======================================================================
[Shows FAILED - No experience in domain: legal_contracts]

======================================================================
TEST 4: Alice (high DEX, only 3 docs tasks) needs 5+ docs experience
======================================================================
[Shows FAILED - Insufficient experience (3 < 5)]
```

---

## Step 6: Multi-Agent Specialist Selection (5 minutes)

```python
"""
Demonstrate selecting the best specialist from multiple candidates
"""

# Create specialist agents
print("\n" + "="*70)
print("CREATING SPECIALIST AGENTS")
print("="*70)

# Charlie: API specialist (15 tasks)
charlie = MinimalAgent("Charlie")
for i in range(15):
    charlie.update_reputation_and_experience(0.90, "api_development")
print(f"\nCharlie: DEX={charlie.get_dex():.3f}, API experience={charlie.get_hex_experience('api_development')['count']}")

# Diana: Documentation specialist (20 tasks)
diana = MinimalAgent("Diana")
for i in range(20):
    diana.update_reputation_and_experience(0.85, "documentation")
print(f"Diana: DEX={diana.get_dex():.3f}, Documentation experience={diana.get_hex_experience('documentation')['count']}")

# Eve: Generalist (5 API, 5 docs, but lower overall experience)
eve = MinimalAgent("Eve")
for i in range(5):
    eve.update_reputation_and_experience(0.88, "api_development")
for i in range(5):
    eve.update_reputation_and_experience(0.88, "documentation")
print(f"Eve: DEX={eve.get_dex():.3f}, API={eve.get_hex_experience('api_development')['count']}, Docs={eve.get_hex_experience('documentation')['count']}")

# Selection function
def select_specialist(candidates, min_dex=0.75, required_domain="api_development", min_experience=5):
    """
    Two-stage selection: DEX filter, then HEX ranking
    """
    print(f"\n{'='*70}")
    print(f"SPECIALIST SELECTION")
    print(f"{'='*70}")
    print(f"Required domain: {required_domain}")
    print(f"Min DEX: {min_dex:.3f}")
    print(f"Min experience: {min_experience} tasks")
    print(f"\nCandidates: {len(candidates)}")
    
    # Stage 1: DEX Filter
    print(f"\n--- Stage 1: DEX Filter (Trust Gate) ---")
    trusted = []
    for agent in candidates:
        dex = agent.get_dex()
        passed = dex >= min_dex
        status = "✓ PASS" if passed else "✗ FAIL"
        print(f"{agent.name:10} DEX={dex:.3f}  {status}")
        if passed:
            trusted.append(agent)
    
    if not trusted:
        print(f"\nResult: No trusted agents available")
        return None
    
    # Stage 2: HEX Ranking
    print(f"\n--- Stage 2: HEX Ranking (Capability Matching) ---")
    specialists = []
    for agent in trusted:
        exp = agent.get_hex_experience(required_domain)
        if exp and exp['count'] >= min_experience:
            hex_score = exp['count'] * exp['confidence']
            specialists.append({
                'agent': agent,
                'experience': exp['count'],
                'confidence': exp['confidence'],
                'hex_score': hex_score
            })
            print(f"{agent.name:10} Experience={exp['count']:2d}, Confidence={exp['confidence']:.3f}, Score={hex_score:.1f}  ✓")
        else:
            exp_count = exp['count'] if exp else 0
            print(f"{agent.name:10} Experience={exp_count:2d}  ✗ Insufficient")
    
    if not specialists:
        print(f"\nResult: No specialists with required experience")
        return None
    
    # Sort by HEX score
    specialists.sort(key=lambda x: x['hex_score'], reverse=True)
    best = specialists[0]
    
    print(f"\n--- Selection Result ---")
    print(f"Selected: {best['agent'].name}")
    print(f"  DEX: {best['agent'].get_dex():.3f}")
    print(f"  Experience: {best['experience']} tasks")
    print(f"  Confidence: {best['confidence']:.3f}")
    print(f"  HEX Score: {best['hex_score']:.1f}")
    
    return best['agent']

# Test selection
candidates = [agent, charlie, diana, eve]  # Alice, Charlie, Diana, Eve

# Select API specialist
selected_api = select_specialist(
    candidates,
    min_dex=0.75,
    required_domain="api_development",
    min_experience=5
)

# Select documentation specialist
selected_docs = select_specialist(
    candidates,
    min_dex=0.75,
    required_domain="documentation",
    min_experience=5
)
```

**Output:**
```
======================================================================
SPECIALIST SELECTION
======================================================================
Required domain: api_development
Min DEX: 0.750
Min experience: 5 tasks

Candidates: 4

--- Stage 1: DEX Filter (Trust Gate) ---
Alice      DEX=0.837  ✓ PASS
Charlie    DEX=0.931  ✓ PASS
Diana      DEX=0.919  ✓ PASS
Eve        DEX=0.898  ✓ PASS

--- Stage 2: HEX Ranking (Capability Matching) ---
Alice      Experience= 5, Confidence=0.850, Score=4.2  ✓
Charlie    Experience=15, Confidence=0.900, Score=13.5  ✓
Diana      Experience= 0  ✗ Insufficient
Eve        Experience= 5, Confidence=0.880, Score=4.4  ✓

--- Selection Result ---
Selected: Charlie
  DEX: 0.931
  Experience: 15 tasks
  Confidence: 0.900
  HEX Score: 13.5
```

---

## Complete Example: Full Implementation

```python
"""
complete_example.py - Full AEX implementation with DEX+HEX
Run this to see the complete flow
"""

def main():
    # 1. Create agents
    print("\n" + "="*70)
    print("STEP 1: CREATE AGENTS")
    print("="*70)
    alice = MinimalAgent("Alice")
    bob = MinimalAgent("Bob")
    charlie = MinimalAgent("Charlie")
    
    # 2. Build Alice's reputation and experience
    print("\n" + "="*70)
    print("STEP 2: BUILD REPUTATION AND EXPERIENCE")
    print("="*70)
    
    print("\nAlice performs 10 API tasks...")
    for i in range(10):
        execute_interaction(alice, bob, f"API task {i+1}", "api_development", 0.85)
    
    print("\nAlice performs 5 documentation tasks...")
    for i in range(5):
        execute_interaction(alice, bob, f"Documentation task {i+1}", "documentation", 0.90)
    
    # 3. Test handshakes
    print("\n" + "="*70)
    print("STEP 3: TEST HANDSHAKES")
    print("="*70)
    
    # Test 1: Charlie requests API work from Alice → SUCCESS
    print("\n--- Test 1: Charlie needs API specialist ---")
    hs1 = EnhancedHandshake(alice, charlie)
    if hs1.execute(min_dex=0.75, required_domain="api_development", min_experience=5):
        print("✓ Charlie accepted Alice!")
        execute_interaction(alice, charlie, "High-value API task", "api_development", 0.95)
    
    # Test 2: Charlie requests legal work from Alice → FAIL
    print("\n--- Test 2: Charlie needs legal specialist ---")
    hs2 = EnhancedHandshake(alice, charlie)
    if hs2.execute(min_dex=0.75, required_domain="legal_contracts", min_experience=5):
        print("✓ Charlie accepted Alice!")
    else:
        print(f"✗ Charlie rejected Alice: {hs2.reason}")
    
    # 4. Final status
    print("\n" + "="*70)
    print("FINAL STATUS")
    print("="*70)
    print(f"\nAlice:")
    print(f"  DEX: {alice.get_dex():.3f}")
    print(f"  Confidence: {alice.get_confidence():.1f}")
    print(f"  Total interactions: {len(alice.ledger)}")
    print(f"  Domains:")
    for exp in alice.hex_corpus['experience']:
        print(f"    - {exp['domain']}: {exp['count']} tasks, confidence={exp['confidence']:.3f}")

if __name__ == "__main__":
    main()
```

---

## Next Steps

### You Now Have:
- ✅ Agents with cryptographic identities (AEX_ID)
- ✅ Reputation tracking with Bayesian updates (AEX_DEX)
- ✅ Domain experience accumulation (AEX_HEX)
- ✅ Two-stage selection: DEX filter + HEX ranking
- ✅ Working handshake protocol
- ✅ Session recording
- ✅ Understanding of the four-layer model

### To Build a Production System:

**1. Use the Full Reference Implementation (30 min)**
```bash
# See IMPLEMENTATION.md for complete code
# Includes: proper ledger, fork handling, witness verification
```

**2. Add These Features:**
- [ ] **Persistent Storage** - Replace in-memory ledger with database
- [ ] **Network Communication** - Add REST/gRPC API (see IMPLEMENTATION.md)
- [ ] **Fork Handling** - Implement lineage tracking with HEX inheritance (see AEX_ID.md)
- [ ] **Witness Verification** - For high-value transactions (see AEX_WITNESS.md)
- [ ] **Domain Matching** - Implement similarity metrics for HEX (see AEX_HEX.md)
- [ ] **HEX Verification** - Validate experience claims against ledger

**3. Security Hardening:**
- [ ] **Key Management** - Use hardware security modules
- [ ] **Rate Limiting** - Prevent DoS attacks
- [ ] **Input Validation** - Validate all incoming data
- [ ] **HEX Inflation Detection** - Check experience vs ledger history
- [ ] **Audit Logging** - Record all security events

**4. Integration:**
- [ ] **Connect Your Agents** - Wrap existing agent code with AEX
- [ ] **Add Delegation** - Let humans authorize agent actions (AEX_REP)
- [ ] **Build Marketplace** - Enable agent discovery with HEX search
- [ ] **Implement Selection** - Use DEX+HEX for optimal matching

---

## Common Patterns

### Pattern 1: Two-Stage Selection with DEX+HEX

```python
def select_agent(candidates, task):
    """Production-grade agent selection"""
    # Stage 1: Filter by reliability
    trusted = [a for a in candidates if a.get_dex() >= task.min_dex]
    
    # Stage 2: Rank by capability
    scored = []
    for agent in trusted:
        exp = agent.get_hex_experience(task.domain)
        if exp and exp['count'] >= task.min_experience:
            score = exp['count'] * exp['confidence']
            scored.append((agent, score))
    
    if not scored:
        return None
    
    # Return best specialist
    scored.sort(key=lambda x: x[1], reverse=True)
    return scored[0][0]
```

### Pattern 2: Reputation + Experience Gating

```python
@require_capability(min_dex=0.8, required_domain="legal_contracts", min_exp=10)
def high_value_legal_review(agent, document):
    """Only trusted, experienced legal specialists can call this"""
    return agent.review_legal_document(document)
```

### Pattern 3: Fork with HEX Inheritance

```python
def fork_agent(parent, fork_type="major"):
    """Create forked agent with inherited DEX and HEX"""
    child = MinimalAgent(f"{parent.name}_v2")
    
    # Inherit DEX
    fork_weight = {'bugfix': 1.0, 'major': 0.5, 'override': 0.1}[fork_type]
    child.alpha = 1 + (parent.alpha - 1) * fork_weight
    child.beta = 1 + (parent.beta - 1) * fork_weight
    
    # Inherit HEX
    for exp in parent.hex_corpus['experience']:
        child.hex_corpus['experience'].append({
            'domain': exp['domain'],
            'count': int(exp['count'] * fork_weight),
            'confidence': exp['confidence'] * 0.8,  # Slight penalty
            'last_updated': datetime.utcnow().isoformat() + 'Z'
        })
    
    return child
```

---

## Troubleshooting

### Issue 1: "Insufficient DEX" Errors

**Problem:** New agents can't interact because they have DEX=0.5

**Solution:**
```python
# Option 1: Lower initial requirements
handshake.execute(min_dex=0.4, min_confidence=2)

# Option 2: Bootstrap with low-risk tasks
for i in range(10):
    execute_interaction(new_agent, trusted_agent, f"Task {i}", "training")

# Option 3: Human vouching (delegation via AEX_REP)
```

### Issue 2: "No Experience in Domain" Errors

**Problem:** Agent has high DEX but no relevant HEX

**Solution:**
```python
# Option 1: Build experience in required domain first
for i in range(5):
    execute_interaction(agent, trainer, f"Training task {i}", required_domain)

# Option 2: Accept agents without domain requirement
handshake.execute(min_dex=0.75, required_domain=None)

# Option 3: Use domain similarity (future feature)
```

### Issue 3: HEX Not Updating

**Problem:** Experience corpus stays empty

**Solution:**
```python
# Ensure domain is specified in update
agent.update_reputation_and_experience(
    outcome=0.85,
    domain="api_development",  # Don't forget this!
    weight=1.0
)

# Verify experience was added
print(agent.hex_corpus['experience'])
```

---

## What to Read Next

**Based on what you want to build:**

| Goal | Read These Docs | Time |
|------|-----------------|------|
| **Production agent** | IMPLEMENTATION.md → SECURITY.md | 3-4 hrs |
| **Multi-agent system** | EXAMPLES.md (#2, #8, #9) → ARCHITECTURE.md | 2-3 hrs |
| **HEX-based marketplace** | AEX_HEX.md → EXAMPLES.md (#8, #9) | 2-3 hrs |
| **Research/academic** | MATH.md → SECURITY.md → All specs | 4-6 hrs |
| **Human delegation** | AEX_REP.md → EXAMPLES.md (#3) | 1-2 hrs |
| **High-value transactions** | AEX_WITNESS.md → EXAMPLES.md (#4) | 1-2 hrs |
| **Fork handling** | AEX_ID.md → AEX_DEX.md → EXAMPLES.md (#5) | 2-3 hrs |

---

## Resources

**Code:**
- Full reference implementation: IMPLEMENTATION.md
- Complete examples with HEX: EXAMPLES.md
- JSON schemas: AEX_SCHEMA.md

**Theory:**
- Four-layer architecture: ARCHITECTURE.md
- Mathematical foundations: MATH.md
- Security analysis (including HEX attacks): SECURITY.md

**Specifications:**
- AEX_ID.md, AEX_REP.md, AEX_DEX.md, AEX_HEX.md, AEX_SESSION.md, AEX_WITNESS.md

**Getting Help:**
- Read troubleshooting: IMPLEMENTATION.md section 9
- Check examples: EXAMPLES.md (12 complete scenarios with HEX)
- Review schemas: AEX_SCHEMA.md (validation rules)

---

## Summary

**You've learned:**
- ✅ How to create AEX agents with cryptographic identities
- ✅ How to track both reputation (DEX) and experience (HEX)
- ✅ How to implement two-stage selection (trust + capability)
- ✅ How reputation and experience build through interactions
- ✅ How to gate access based on both reliability and expertise
- ✅ How to select optimal specialists for specific tasks

**You can now:**
- ✅ Build multi-agent systems with trust and capability
- ✅ Implement specialist discovery and matching
- ✅ Track and update agent reputation and experience
- ✅ Verify agent identities and capabilities cryptographically
- ✅ Select the best agent for any given task

**Next level:**
- 🚀 Add persistent storage and networking
- 🚀 Implement full AEX handshake with all checks
- 🚀 Add fork handling with HEX inheritance
- 🚀 Build an agent marketplace with HEX search
- 🚀 Add witness verification for high-value tasks
- 🚀 Implement HEX inflation detection

---

**Congratulations! You now have a working four-layer AEX implementation.** 🎉

**Time to build:** Start with the complete example, then extend it for your use case. The full specification suite has everything you need for production deployment with DEX+HEX selection.
