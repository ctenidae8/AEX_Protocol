# AEX_REP v1.0
## Representation, Delegation, and Authority for Synthetic Agents

**Last Updated:** February 5, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Design Principles](#design-principles)
3. [AEX_REP Structure](#aex_rep-structure)
4. [Scope Definition](#scope-definition)
5. [Constraints System](#constraints-system)
6. [TTL-Based Revocation](#ttl-based-revocation)
7. [Token Lifecycle](#token-lifecycle)
8. [Verification Protocol](#verification-protocol)
9. [Multi-Hop Delegation](#multi-hop-delegation)
10. [Age-Triggered Revalidation](#age-triggered-revalidation)
11. [Ledger Integration](#ledger-integration)
12. [Implementation Guide](#implementation-guide)
13. [Security Considerations](#security-considerations)
14. [Examples](#examples)

---

## Overview

AEX_REP defines the representation and delegation substrate for synthetic agents. It establishes **who an agent acts for** and **what authority it has**.

### Core Questions Answered

1. **Who authorized this agent?** - Cryptographically verified issuer
2. **What can the agent do?** - Explicit scope (actions + domains)
3. **What limits apply?** - Hard constraints (cost, time, rate)
4. **Is authorization still valid?** - TTL + age-triggered revalidation
5. **Can authority be revoked?** - Immediate via TTL expiry

### What AEX_REP Is NOT

❌ An identity system (use AEX_ID for that)  
❌ A reputation system (use AEX_DEX for that)  
❌ An experience corpus (use AEX_HEX for that)  
❌ A payment mechanism  
❌ A capability proof system  
❌ A general-purpose authorization framework (it's agent-specific)

### Relationship to Other Primitives

**REP works with:**
- **AEX_ID:** REP tokens reference agent identity
- **AEX_DEX:** Delegated agent's DEX is still evaluated
- **AEX_HEX:** Delegated agent's HEX is still relevant for task matching

**When evaluating a delegated agent:**
1. Verify REP token (authorization)
2. Check delegate's DEX (is the delegated agent reliable?)
3. Check delegate's HEX (is the delegated agent capable?)
4. Optionally check principal's credentials too

---

## Design Principles

### 1. Explicit Delegation
Authority must be granted **intentionally and explicitly**. No implied permissions.

### 2. Scope-Bound
Every delegation includes **clear positive permissions** (what you may do) and **negative constraints** (what you must not do).

### 3. Time-Limited
All tokens have **TTL (Time To Live)**. No perpetual delegation. Short TTLs solve the revocation problem.

### 4. Cryptographically Verifiable
All delegation tokens are **signed by the issuer**. Cannot be forged without key theft.

### 5. Portable
Tokens are **serializable** and work across platforms. No platform lock-in.

### 6. Revocable
Delegation can be revoked at any time via:
- TTL expiry (automatic)
- Age-triggered revalidation check
- Explicit revocation registry (optional)

### 7. Human-Centric
Designed for **humans delegating to agents**, not just agent-to-agent authorization.

---

## AEX_REP Structure

### Required Fields
```json
{
  "rep_id": "550e8400-e29b-41d4-a716-446655440001",
  "version": "1.0",
  "issuer": "did:human:alice_2026",
  "delegate": "did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM",
  "scope": {
    "actions": ["research", "analyze", "summarize", "draft"],
    "domains": ["finance", "technology"],
    "limits": {
      "max_cost": 100,
      "max_duration": 3600,
      "max_tasks_per_day": 10
    }
  },
  "constraints": {
    "expires_at": "2026-02-05T12:00:00Z",
    "prohibited_actions": ["purchase", "sign_contracts", "commit_funds"],
    "prohibited_domains": ["legal", "medical"],
    "human_approval_required": false,
    "witness_required": false,
    "rate_limits": {
      "requests_per_hour": 20,
      "concurrent_sessions": 3
    }
  },
  "issued_at": "2026-02-04T11:00:00Z",
  "issuer_metadata": {
    "issuer_name": "Alice Johnson",
    "purpose": "Q4 2025 financial analysis project",
    "contact": "alice@example.com"
  },
  "signature": "issuer_ed25519_signature"
}
```

### Field Definitions

#### `rep_id` (required)
- Type: UUID (RFC 4122)
- Purpose: Unique identifier for this delegation token
- Immutable: Cannot be changed after issuance
- Example: `550e8400-e29b-41d4-a716-446655440001`

#### `version` (required)
- Type: String
- Purpose: AEX_REP protocol version
- Value: `"1.0"` for this specification
- Future-proofing: Allows protocol evolution

#### `issuer` (required)
- Type: DID (Decentralized Identifier)
- Purpose: Who is granting authority
- Can be:
  - Human: `did:human:alice_2026`
  - Organization: `did:org:acme_corp`
  - Agent (for sub-delegation): `did:aex:...`
- Must have public key for signature verification

#### `delegate` (required)
- Type: AEX_ID (DID format)
- Purpose: Which agent receives authority
- Must match requesting agent's AEX_ID
- Cannot be wildcard (one token per agent)

#### `scope` (required)
- Type: Object
- Purpose: **Positive permissions** - what the agent may do
- Contains: `actions`, `domains`, `limits`
- See [Scope Definition](#scope-definition)

#### `constraints` (required)
- Type: Object
- Purpose: **Negative permissions** - what the agent must not do
- Contains: `expires_at`, `prohibited_actions`, `prohibited_domains`, flags, rate limits
- See [Constraints System](#constraints-system)

#### `issued_at` (required)
- Type: ISO 8601 timestamp (UTC)
- Purpose: Token creation time
- Used for: Age calculation, audit trails
- Immutable: Cannot be changed
- Example: `2026-02-04T11:00:00Z`

#### `issuer_metadata` (optional)
- Type: Object
- Purpose: Human-readable context
- Not verified: Informational only
- Can include: name, purpose, contact info

#### `signature` (required)
- Type: String (base58-encoded Ed25519 signature)
- Purpose: Cryptographic proof from issuer
- Signed over: Canonical JSON of all fields except signature
- Verification: Uses issuer's public key

---

## Scope Definition

### Actions

**Purpose:** Explicit verbs describing what the agent may do.

**Format:** Array of action strings

**Examples:**
```json
{
  "actions": [
    "research",
    "analyze", 
    "summarize",
    "draft",
    "schedule",
    "notify",
    "query",
    "monitor"
  ]
}
```

**Common Actions (Non-Normative):**
- `research` - Search for information
- `analyze` - Perform analysis on data
- `summarize` - Create summaries
- `draft` - Create draft documents
- `schedule` - Book appointments/meetings
- `notify` - Send notifications
- `query` - Query databases/APIs
- `monitor` - Watch for changes
- `purchase` - Buy goods/services (high-stakes)
- `sign_contracts` - Execute contracts (high-stakes)
- `commit_funds` - Transfer money (high-stakes)
- `negotiate` - Negotiate on behalf of human

**Semantics:**
- Actions are **application-specific**
- No protocol-enforced vocabulary
- Market will converge on common terms
- Agents should document supported actions

### Domains

**Purpose:** Context or subject areas the agent may operate in.

**Format:** Array of domain strings

**Examples:**
```json
{
  "domains": [
    "finance",
    "technology",
    "healthcare",
    "legal",
    "education",
    "entertainment"
  ]
}
```

**Common Domains (Non-Normative):**
- `finance` - Financial data, transactions, analysis
- `technology` - Tech sector, software, hardware
- `healthcare` - Medical information, health data
- `legal` - Legal documents, contracts, compliance
- `education` - Learning, courses, certifications
- `entertainment` - Media, events, recommendations
- `travel` - Booking, itineraries, logistics
- `commerce` - Shopping, vendors, products
- `hr` - Human resources, hiring, personnel
- `operations` - Business operations, processes

**Semantics:**
- Domains are **application-specific**
- No protocol-enforced vocabulary
- Can be hierarchical: `finance.tax`, `healthcare.diagnosis`
- Agents should document domain coverage

### Limits

**Purpose:** Quantitative boundaries on agent behavior.

**Format:** Object with numeric limits

**Standard Limits:**
```json
{
  "limits": {
    "max_cost": 100,              // Maximum $ value per action
    "max_duration": 3600,          // Maximum seconds per task
    "max_tasks_per_day": 10,       // Daily task quota
    "max_api_calls": 1000,         // API call quota
    "max_file_size": 10485760      // Maximum file size (bytes)
  }
}
```

**Custom Limits:**
Issuers can add domain-specific limits:
```json
{
  "limits": {
    "max_cost": 500,
    "max_transactions": 5,
    "max_recipients": 3,
    "min_approval_threshold": 0.8  // DEX score for counterparties
  }
}
```

### HEX in Delegation Context

**Question:** When an agent is delegated via REP, whose HEX matters?

**Answer:** The **delegate's HEX** (the agent actually performing the work).

**Rationale:**
- HEX represents accumulated experience of the agent doing the work
- Principal (human/issuer) typically doesn't have HEX
- Delegate's capabilities determine task success

**Example:**
```json
{
  "issuer": "did:human:alice",           // Human has no HEX
  "delegate": "did:aex:translator_bot",  // Bot has HEX in translation.fr_en
  "scope": {
    "actions": ["translate"],
    "domains": ["documents"]
  }
}
```

**Selection logic:**
```python
def select_delegated_agent(rep_token, task):
    """
    Evaluate delegated agent for task
    """
    # 1. Verify REP token (authorization)
    if not verify_rep_token(rep_token):
        return False, "Invalid REP token"
    
    # 2. Check delegate's DEX (reliability)
    delegate_dex = get_dex(rep_token['delegate'])
    if delegate_dex['score'] < 0.75:
        return False, "Delegate unreliable"
    
    # 3. Check delegate's HEX (capability)
    delegate_hex = get_hex(rep_token['delegate'])
    if not has_domain_experience(delegate_hex, task['required_domain']):
        return False, "Delegate lacks experience"
    
    # 4. Optionally check principal's reputation/credentials
    # (implementation-specific)
    
    return True, "Delegate authorized and capable"
```

**Key insight:** REP provides authorization, but the delegate's DEX and HEX still determine whether the agent is trustworthy and capable.

### Optional: Principal's Credentials

In some scenarios, the **principal's** credentials may also matter:

**When principal credentials matter:**
- Professional licensing (doctor delegating to medical AI)
- Legal authority (lawyer delegating to legal research AI)
- Financial credentials (trader delegating to trading bot)

**When only delegate matters:**
- General task automation (human delegating to assistant)
- No domain-specific licensing required
- Authority is purely about scope, not expertise

**Implementation note:** This is non-normative. Applications decide whether to check principal credentials.

---

## Constraints System

### Expiration

**Required field:** `expires_at`
```json
{
  "constraints": {
    "expires_at": "2026-02-05T12:00:00Z"
  }
}
```

**TTL Calculation:**
```python
def calculate_ttl(issued_at, expires_at):
    """
    Calculate time-to-live in seconds
    """
    issued = datetime.fromisoformat(issued_at.replace('Z', '+00:00'))
    expires = datetime.fromisoformat(expires_at.replace('Z', '+00:00'))
    ttl_seconds = (expires - issued).total_seconds()
    return ttl_seconds

# Example:
issued_at = "2026-02-04T11:00:00Z"
expires_at = "2026-02-05T12:00:00Z"
ttl = calculate_ttl(issued_at, expires_at)
# Result: 90000 seconds (25 hours)
```

**Recommended TTL Values:**

| Use Case | TTL | Rationale |
|----------|-----|-----------|
| High-stakes (purchase, contracts) | 1 hour | Frequent revalidation |
| Medium-stakes (analysis, research) | 4-8 hours | Balance security/UX |
| Low-stakes (queries, monitoring) | 24 hours | Reduced overhead |
| Session-specific | Task duration | Auto-expire with task |

### Prohibited Actions

**Purpose:** Explicitly forbidden actions, even if implied by scope.
```json
{
  "constraints": {
    "prohibited_actions": [
      "purchase",
      "sign_contracts",
      "commit_funds",
      "delete_data",
      "grant_permissions"
    ]
  }
}
```

**Verification:**
```python
def check_action_allowed(proposed_action, scope, constraints):
    """
    Verify action is allowed
    """
    # Check 1: Must be in scope
    if proposed_action not in scope['actions']:
        return False, f"Action '{proposed_action}' not in scope"
    
    # Check 2: Must not be prohibited
    if proposed_action in constraints['prohibited_actions']:
        return False, f"Action '{proposed_action}' explicitly prohibited"
    
    return True, "Action allowed"
```

### Prohibited Domains

**Purpose:** Explicitly forbidden domains, even if implied by scope.
```json
{
  "constraints": {
    "prohibited_domains": [
      "legal",
      "medical",
      "financial.trading",
      "hr.hiring"
    ]
  }
}
```

### Human Approval Required

**Purpose:** Force human confirmation before proceeding.
```json
{
  "constraints": {
    "human_approval_required": true,
    "approval_contact": "alice@example.com",
    "approval_timeout": 300  // Seconds to wait for approval
  }
}
```

**Approval Flow:**
```
1. Agent receives task request
2. Agent verifies AEX_REP has human_approval_required=true
3. Agent sends approval request to issuer (via email, SMS, app notification)
4. Human approves or denies
5. Agent proceeds or declines based on human decision
```

### Witness Required

**Purpose:** Force third-party attestation for high-stakes tasks.
```json
{
  "constraints": {
    "witness_required": true,
    "min_witnesses": 2,
    "min_witness_dex": 0.8
  }
}
```

### Rate Limits

**Purpose:** Prevent abuse, control resource consumption.
```json
{
  "constraints": {
    "rate_limits": {
      "requests_per_hour": 20,
      "requests_per_day": 100,
      "concurrent_sessions": 3,
      "cost_per_day": 500
    }
  }
}
```

**Rate Limit Enforcement:**
```python
def check_rate_limits(agent_id, rep_id, proposed_action):
    """
    Verify rate limits not exceeded
    """
    # Get usage stats for this token
    usage = get_token_usage(rep_id)
    limits = get_token_rate_limits(rep_id)
    
    # Check hourly limit
    if usage['requests_last_hour'] >= limits['requests_per_hour']:
        return False, "Hourly request limit exceeded"
    
    # Check daily limit
    if usage['requests_last_day'] >= limits['requests_per_day']:
        return False, "Daily request limit exceeded"
    
    # Check concurrent sessions
    if usage['active_sessions'] >= limits['concurrent_sessions']:
        return False, "Concurrent session limit exceeded"
    
    # Check daily cost
    if usage['cost_today'] + proposed_action['estimated_cost'] > limits['cost_per_day']:
        return False, "Daily cost limit exceeded"
    
    return True, "Rate limits satisfied"
```

---

## TTL-Based Revocation

### Core Concept

**Problem:** Traditional revocation requires:
- Central revocation registries
- Network queries on every verification
- Complex propagation protocols

**AEX Solution:** Short-lived tokens that auto-expire.

### How It Works
```
Token Lifecycle:
────────────────

Issue token (TTL=4 hours)
   │
   ├─> Hour 1: Token valid, fresh
   │
   ├─> Hour 2: Token valid, fresh
   │
   ├─> Hour 3: Token valid, AGING (>75% of lifetime)
   │           → Age-triggered revalidation
   │           → Check with issuer if still valid
   │
   ├─> Hour 4: Token valid, AGING
   │           → Mandatory revalidation
   │
   └─> Hour 4+: Token EXPIRED
               → Automatic rejection
               → Agent must request new token
```

### Benefits

1. **No revocation registry needed** - Tokens expire automatically
2. **Bounded blast radius** - Stolen token useless after TTL
3. **Forced check-ins** - Agents must periodically revalidate
4. **Simple implementation** - Just check timestamp

### TTL Verification
```python
def verify_ttl(rep_token):
    """
    Check if token is still valid
    """
    now = datetime.utcnow()
    issued = datetime.fromisoformat(rep_token['issued_at'].replace('Z', '+00:00'))
    expires = datetime.fromisoformat(rep_token['constraints']['expires_at'].replace('Z', '+00:00'))
    
    # Check if expired
    if now > expires:
        return False, "Token expired"
    
    # Calculate age percentage
    lifetime = (expires - issued).total_seconds()
    age = (now - issued).total_seconds()
    age_pct = age / lifetime
    
    # Check if approaching expiry (>80% of lifetime)
    if age_pct > 0.8:
        return "AGING", f"Token {age_pct*100:.0f}% of lifetime, revalidate soon"
    
    return True, f"Token valid for {(expires - now).total_seconds():.0f} more seconds"
```

---

## Token Lifecycle

### Phase 1: Issuance

**Human issues token to agent:**
```python
def issue_delegation_token(issuer_keypair, delegate_did, scope, constraints):
    """
    Issue AEX_REP token
    """
    # Create token
    token = {
        'rep_id': str(uuid.uuid4()),
        'version': '1.0',
        'issuer': issuer_keypair['did'],
        'delegate': delegate_did,
        'scope': scope,
        'constraints': constraints,
        'issued_at': datetime.utcnow().isoformat() + 'Z',
        'issuer_metadata': {
            'issuer_name': issuer_keypair['name'],
            'purpose': 'Financial analysis Q4 2025'
        }
    }
    
    # Sign with issuer's private key
    signed_token = sign_rep_token(token, issuer_keypair['private_key'])
    
    # Optionally publish to ledger (for audit trail)
    shared_ledger.put(
        path=f"delegations/{issuer_keypair['did']}/{token['rep_id']}.json",
        data=signed_token
    )
    
    return signed_token
```

### Phase 2: Presentation

**Agent presents token during handshake:**
```json
{
  "handshake_version": "1.0",
  "identity": { /* AEX_ID */ },
  "delegation": { /* AEX_REP token */ },
  "task_proposal": { /* Task details */ }
}
```

### Phase 3: Verification

**Counterparty verifies token:**
```python
def verify_rep_token(rep_token, requesting_agent_did, task_proposal):
    """
    Complete AEX_REP verification
    """
    # Check 1: Verify issuer signature
    sig_valid = verify_issuer_signature(rep_token)
    if not sig_valid:
        return False, "Invalid issuer signature"
    
    # Check 2: Verify delegate matches requesting agent
    if rep_token['delegate'] != requesting_agent_did:
        return False, "Token not issued to this agent"
    
    # Check 3: Verify TTL
    ttl_status, ttl_msg = verify_ttl(rep_token)
    if ttl_status == False:
        return False, ttl_msg
    
    # Check 4: Verify scope covers proposed actions
    actions_ok, actions_msg = verify_actions(
        task_proposal,
        rep_token['scope'],
        rep_token['constraints']
    )
    if not actions_ok:
        return False, actions_msg
    
    # Check 5: Verify domains covered
    domains_ok, domains_msg = verify_domains(
        task_proposal,
        rep_token['scope'],
        rep_token['constraints']
    )
    if not domains_ok:
        return False, domains_msg
    
    # Check 6: Verify limits not exceeded
    limits_ok, limits_msg = verify_limits(
        task_proposal,
        rep_token['scope']['limits']
    )
    if not limits_ok:
        return False, limits_msg
    
    # Check 7: Check rate limits
    rate_ok, rate_msg = check_rate_limits(
        requesting_agent_did,
        rep_token['rep_id'],
        task_proposal
    )
    if not rate_ok:
        return False, rate_msg
    
    # Check 8: Age-triggered revalidation (if aging)
    if ttl_status == "AGING":
        revalidation = perform_age_triggered_revalidation(rep_token)
        if not revalidation['valid']:
            return False, f"Revoked: {revalidation['reason']}"
    
    return True, "Authorization verified"
```

### Phase 4: Execution

**Agent performs authorized work:**
```python
def execute_with_delegation(agent, rep_token, task):
    """
    Execute task under delegation
    """
    # Track token usage
    record_token_usage(rep_token['rep_id'], task)
    
    # Perform work
    result = agent.execute(task)
    
    # Log to audit trail
    log_delegated_action(
        rep_id=rep_token['rep_id'],
        agent_id=agent.aex_id,
        task=task,
        result=result,
        timestamp=datetime.utcnow()
    )
    
    return result
```

### Phase 5: Expiration

**Token expires automatically:**
```python
# After TTL expires, token is automatically invalid
# No explicit revocation needed
# Agent must request new token to continue
```

### Phase 6: Renewal (Optional)

**Agent requests fresh token:**
```python
def request_token_renewal(agent, old_token):
    """
    Request new token before old one expires
    """
    # Check if renewal is allowed
    if not is_near_expiry(old_token):
        return None, "Token not near expiry yet"
    
    # Request from issuer
    renewal_request = {
        'agent_id': agent.aex_id,
        'old_rep_id': old_token['rep_id'],
        'reason': 'Ongoing work, token expiring soon',
        'requested_ttl': 14400  # 4 hours
    }
    
    # Issuer decides whether to grant
    response = contact_issuer(old_token['issuer'], renewal_request)
    
    if response['granted']:
        new_token = response['token']
        return new_token, "Renewal granted"
    else:
        return None, response['reason']
```

---

## Verification Protocol

### Complete Verification Function
```python
def verify_aex_rep_complete(rep_token, requesting_agent, task_proposal, shared_ledger):
    """
    Production-grade AEX_REP verification
    
    Returns:
        (bool, str, dict): (success, message, details)
    """
    results = {
        'signature': None,
        'delegate_match': None,
        'ttl': None,
        'scope': None,
        'limits': None,
        'rate_limits': None,
        'revalidation': None
    }
    
    # Verification Step 1: Signature
    try:
        sig_valid = verify_issuer_signature(rep_token)
        results['signature'] = sig_valid
        if not sig_valid:
            return False, "Invalid issuer signature", results
    except Exception as e:
        return False, f"Signature verification failed: {e}", results
    
    # Verification Step 2: Delegate Match
    if rep_token['delegate'] != requesting_agent['aex_id']:
        results['delegate_match'] = False
        return False, "Delegation token not for this agent", results
    results['delegate_match'] = True
    
    # Verification Step 3: TTL
    ttl_status, ttl_msg = verify_ttl(rep_token)
    results['ttl'] = {'status': ttl_status, 'message': ttl_msg}
    if ttl_status == False:
        return False, ttl_msg, results
    
    # Verification Step 4: Scope (Actions + Domains)
    proposed_actions = infer_actions_from_task(task_proposal)
    for action in proposed_actions:
        if action not in rep_token['scope']['actions']:
            results['scope'] = {'valid': False, 'failed_action': action}
            return False, f"Action '{action}' not in scope", results
        if action in rep_token['constraints'].get('prohibited_actions', []):
            results['scope'] = {'valid': False, 'prohibited_action': action}
            return False, f"Action '{action}' explicitly prohibited", results
    
    proposed_domains = infer_domains_from_task(task_proposal)
    for domain in proposed_domains:
        if domain not in rep_token['scope']['domains']:
            results['scope'] = {'valid': False, 'failed_domain': domain}
            return False, f"Domain '{domain}' not in scope", results
        if domain in rep_token['constraints'].get('prohibited_domains', []):
            results['scope'] = {'valid': False, 'prohibited_domain': domain}
            return False, f"Domain '{domain}' explicitly prohibited", results
    
    results['scope'] = {'valid': True, 'actions': proposed_actions, 'domains': proposed_domains}
    
    # Verification Step 5: Limits
    limits = rep_token['scope']['limits']
    
    if task_proposal.get('estimated_cost', 0) > limits.get('max_cost', float('inf')):
        results['limits'] = {'valid': False, 'exceeded': 'max_cost'}
        return False, f"Cost ${task_proposal['estimated_cost']} exceeds limit ${limits['max_cost']}", results
    
    if task_proposal.get('estimated_duration', 0) > limits.get('max_duration', float('inf')):
        results['limits'] = {'valid': False, 'exceeded': 'max_duration'}
        return False, f"Duration {task_proposal['estimated_duration']}s exceeds limit {limits['max_duration']}s", results
    
    results['limits'] = {'valid': True}
    
    # Verification Step 6: Rate Limits
    rate_status, rate_msg = check_rate_limits(
        requesting_agent['aex_id'],
        rep_token['rep_id'],
        task_proposal
    )
    results['rate_limits'] = {'valid': rate_status, 'message': rate_msg}
    if not rate_status:
        return False, rate_msg, results
    
    # Verification Step 7: Age-Triggered Revalidation
    if ttl_status == "AGING":
        revalidation = perform_age_triggered_revalidation(rep_token, shared_ledger)
        results['revalidation'] = revalidation
        if not revalidation['valid']:
            return False, f"Token revoked: {revalidation['reason']}", results
    
    # All checks passed
    return True, "Authorization verified", results
```

### Verification with HEX Consideration

When verifying a delegated agent for a specific task:

```python
def verify_delegated_agent_for_task(rep_token, task, shared_ledger):
    """
    Complete verification: REP + DEX + HEX
    """
    # Step 1: Verify REP token
    rep_valid, rep_reason, rep_details = verify_aex_rep_complete(
        rep_token,
        {'aex_id': rep_token['delegate']},
        task,
        shared_ledger
    )
    if not rep_valid:
        return False, f"REP invalid: {rep_reason}"
    
    # Step 2: Check scope matches task
    if task['action'] not in rep_token['scope']['actions']:
        return False, f"Action '{task['action']}' not in scope"
    
    if task['domain'] not in rep_token['scope']['domains']:
        return False, f"Domain '{task['domain']}' not in scope"
    
    # Step 3: Verify delegate's DEX (trustworthiness)
    delegate_dex = shared_ledger.get_dex(rep_token['delegate'])
    if not delegate_dex:
        return False, "Delegate has no DEX record"
    
    dex_score = delegate_dex['alpha'] / (delegate_dex['alpha'] + delegate_dex['beta'])
    if dex_score < task.get('min_dex', 0.75):
        return False, f"Delegate DEX {dex_score:.2f} below threshold"
    
    # Step 4: Verify delegate's HEX (capability)
    if 'required_domain' in task:
        delegate_hex = shared_ledger.get_hex(rep_token['delegate'])
        
        if not delegate_hex:
            return False, "Delegate has no HEX record"
        
        domain_exp = next(
            (e for e in delegate_hex['experience'] 
             if e['domain'] == task['required_domain']),
            None
        )
        
        if not domain_exp:
            return False, f"Delegate has no experience in {task['required_domain']}"
        
        if domain_exp['count'] < task.get('min_experience', 10):
            return False, f"Delegate experience too low: {domain_exp['count']}"
        
        if domain_exp['confidence'] < task.get('min_confidence', 0.7):
            return False, f"Delegate confidence too low: {domain_exp['confidence']:.2f}"
    
    # All checks passed
    return True, "Delegate authorized, trustworthy, and capable"


# Example usage:
rep_token = {
    'issuer': 'did:human:alice',
    'delegate': 'did:aex:translator_bot',
    'scope': {
        'actions': ['translate'],
        'domains': ['documents']
    },
    'constraints': {
        'expires_at': '2026-02-06T00:00:00Z'
    }
}

task = {
    'action': 'translate',
    'domain': 'documents',
    'required_domain': 'translation.fr_en',
    'min_dex': 0.75,
    'min_experience': 20,
    'min_confidence': 0.80
}

valid, reason = verify_delegated_agent_for_task(rep_token, task, ledger)
# Returns: True, "Delegate authorized, trustworthy, and capable"
```

**Decision hierarchy:**
1. **REP:** Does the agent have authority? (gate 1)
2. **DEX:** Is the agent trustworthy? (gate 2)
3. **HEX:** Is the agent capable? (gate 3)

All three must pass for optimal selection.

---

## Multi-Hop Delegation

### Concept

**Scenario:** Human delegates to Agent A, who sub-delegates to Agent B.
```
Human (Alice)
   │
   │ AEX_REP₁ (alice → agent_a)
   ↓
Agent A
   │
   │ AEX_REP₂ (agent_a → agent_b)
   ↓
Agent B
```

### Chain of Authority

**REP₁ (Alice → Agent A):**
```json
{
  "rep_id": "rep_1",
  "issuer": "did:human:alice",
  "delegate": "did:aex:agent_a",
  "scope": {
    "actions": ["research", "analyze", "summarize"],
    "domains": ["finance"],
    "limits": {"max_cost": 100}
  },
  "constraints": {
    "expires_at": "2026-02-05T12:00:00Z",
    "allow_sub_delegation": true,  // Explicit permission
    "sub_delegation_constraints": {
      "max_depth": 1,              // Only 1 level of sub-delegation
      "scope_must_narrow": true    // Sub-delegate can't expand scope
    }
  }
}
```

**REP₂ (Agent A → Agent B):**
```json
{
  "rep_id": "rep_2",
  "issuer": "did:aex:agent_a",
  "delegate": "did:aex:agent_b",
  "scope": {
    "actions": ["research"],      // Narrower than REP₁
    "domains": ["finance"],
    "limits": {"max_cost": 50}    // Lower than REP₁
  },
  "constraints": {
    "expires_at": "2026-02-05T11:00:00Z",  // Earlier than REP₁
    "parent_rep_id": "rep_1",              // Links to parent
    "delegation_chain": ["rep_1"]
  }
}
```

### Chain Verification
```python
def verify_delegation_chain(rep_token, shared_ledger):
    """
    Verify multi-hop delegation chain
    """
    # Check if this is a sub-delegation
    if 'parent_rep_id' not in rep_token['constraints']:
        return True, "No parent, root delegation"
    
    parent_rep_id = rep_token['constraints']['parent_rep_id']
    
    # Get parent token
    parent_token = shared_ledger.get(
        f"delegations/{rep_token['issuer']}/{parent_rep_id}.json"
    )
    
    if not parent_token:
        return False, f"Parent delegation {parent_rep_id} not found"
    
    # Check 1: Parent allows sub-delegation
    if not parent_token['constraints'].get('allow_sub_delegation', False):
        return False, "Parent token does not allow sub-delegation"
    
    # Check 2: Depth limit
    chain = rep_token['constraints']['delegation_chain']
    max_depth = parent_token['constraints']['sub_delegation_constraints']['max_depth']
    if len(chain) > max_depth:
        return False, f"Delegation chain exceeds max depth {max_depth}"
    
    # Check 3: Scope narrowing
    if parent_token['constraints']['sub_delegation_constraints']['scope_must_narrow']:
        scope_valid = verify_scope_narrowing(rep_token['scope'], parent_token['scope'])
        if not scope_valid:
            return False, "Sub-delegation scope broader than parent"
    
    # Check 4: Expiry before parent
    if rep_token['constraints']['expires_at'] > parent_token['constraints']['expires_at']:
        return False, "Sub-delegation expires after parent"
    
    # Recursively verify parent chain
    parent_valid, parent_msg = verify_delegation_chain(parent_token, shared_ledger)
    if not parent_valid:
        return False, f"Parent chain invalid: {parent_msg}"
    
    return True, "Delegation chain valid"

def verify_scope_narrowing(child_scope, parent_scope):
    """
    Verify child scope is subset of parent scope
    """
    # Check actions
    for action in child_scope['actions']:
        if action not in parent_scope['actions']:
            return False
    
    # Check domains
    for domain in child_scope['domains']:
        if domain not in parent_scope['domains']:
            return False
    
    # Check limits (child must be <= parent)
    if child_scope['limits'].get('max_cost', 0) > parent_scope['limits'].get('max_cost', float('inf')):
        return False
    
    return True
```

### Cascading Accountability

**Key insight:** Reputation consequences flow up the chain.
```python
def update_dex_with_delegation_chain(session_outcome, delegation_chain):
    """
    Update DEX for all agents in delegation chain
    """
    outcome = session_outcome['agreed_outcome']
    
    # Agent B (actual performer) gets full outcome
    update_dex(
        agent_id=session_outcome['agent_b'],
        outcome=outcome,
        weight=1.0
    )
    
    # Agent A (delegator) gets partial outcome
    # Logic: If B fails, A chose badly (reputation damage)
    delegator_outcome = outcome * 0.8  # Dampened
    update_dex(
        agent_id=session_outcome['agent_a'],
        outcome=delegator_outcome,
        weight=0.5
    )
    
    # Human issuer: No DEX (not an agent)
    # But could maintain human reputation separately
```

---

## Age-Triggered Revalidation

### Problem

**Stolen/compromised tokens** remain valid until TTL expires. If TTL is 24 hours, attacker has 24-hour window.

### Solution

**Age-triggered revalidation:** When token enters last 20% of lifetime, verify with issuer that it's still valid.

### Implementation
```python
def perform_age_triggered_revalidation(rep_token, shared_ledger):
    """
    Check with issuer if token still valid
    
    Called when token age > 80% of lifetime
    """
    # Calculate token age
    now = datetime.utcnow()
    issued = datetime.fromisoformat(rep_token['issued_at'].replace('Z', '+00:00'))
    expires = datetime.fromisoformat(rep_token['constraints']['expires_at'].replace('Z', '+00:00'))
    
    lifetime = (expires - issued).total_seconds()
    age = (now - issued).total_seconds()
    age_pct = age / lifetime
    
    # Only revalidate if aging (>80% of lifetime)
    if age_pct < 0.8:
        return {'valid': True, 'reason': 'Token fresh, no revalidation needed'}
    
    # Check revocation registry (if available)
    revoked = check_revocation_registry(rep_token['rep_id'])
    if revoked:
        return {
            'valid': False,
            'reason': 'Token revoked by issuer',
            'revoked_at': revoked['timestamp']
        }
    
    # Query issuer directly (if online)
    issuer_response = query_issuer_for_token_status(
        issuer_did=rep_token['issuer'],
        rep_id=rep_token['rep_id']
    )
    
    if issuer_response and issuer_response['status'] == 'revoked':
        return {
            'valid': False,
            'reason': issuer_response['reason'],
            'revoked_at': issuer_response['timestamp']
        }
    
    # Issuer confirms still valid (or unreachable, assume valid)
    return {
        'valid': True,
        'reason': 'Issuer confirms token valid' if issuer_response else 'Issuer unreachable, assume valid',
        'verified_at': datetime.utcnow().isoformat() + 'Z'
    }

def check_revocation_registry(rep_id):
    """
    Check optional revocation registry
    
    This is OFF-PROTOCOL but commonly used
    """
    # Query shared revocation registry
    revocation = shared_ledger.get(f"revocations/{rep_id}.json")
    
    if revocation and revocation['status'] == 'revoked':
        return revocation
    
    return None

def query_issuer_for_token_status(issuer_did, rep_id):
    """
    Direct query to issuer for token status
    
    This is OFF-PROTOCOL but recommended
    """
    try:
        # Get issuer's revalidation endpoint (from DID document or convention)
        issuer_endpoint = resolve_issuer_endpoint(issuer_did)
        
        # Query status
        response = requests.get(
            f"{issuer_endpoint}/rep/status/{rep_id}",
            timeout=5
        )
        
        if response.status_code == 200:
            return response.json()
        
        return None
    except Exception as e:
        # Issuer unreachable, assume valid
        log.warning(f"Could not reach issuer {issuer_did}: {e}")
        return None
```

### Revalidation Strategy

**Conservative approach:**
```python
# Token in last 20% of life? Revalidate every time
if age_pct > 0.8:
    revalidation = perform_age_triggered_revalidation(rep_token)
    if not revalidation['valid']:
        return REJECT, "Token revoked"
```

**Optimistic approach:**
```python
# Token in last 20% of life? Revalidate once, cache result for 5 minutes
if age_pct > 0.8:
    cached = revalidation_cache.get(rep_token['rep_id'])
    if not cached or (now() - cached['timestamp']).seconds > 300:
        revalidation = perform_age_triggered_revalidation(rep_token)
        revalidation_cache.set(rep_token['rep_id'], revalidation)
    else:
        revalidation = cached
    
    if not revalidation['valid']:
        return REJECT, "Token revoked"
```

---

## Ledger Integration

### Directory Structure
```
shared_ledger/
├── delegations/
│   └── {issuer_did}/
│       ├── {rep_id_1}.json
│       ├── {rep_id_2}.json
│       └── active/
│           └── {delegate_did}.json  // Currently active tokens
├── revocations/
│   └── {rep_id}.json
└── delegation_usage/
    └── {rep_id}/
        └── usage_log.json
```

### Publishing Delegation
```python
def publish_delegation_to_ledger(rep_token):
    """
    Publish delegation token to shared ledger (optional but recommended)
    """
    issuer_did = rep_token['issuer']
    rep_id = rep_token['rep_id']
    
    # Publish token
    shared_ledger.put(
        path=f"delegations/{issuer_did}/{rep_id}.json",
        data=rep_token
    )
    
    # Update active tokens index
    delegate_did = rep_token['delegate']
    active_tokens = shared_ledger.get(
        f"delegations/{issuer_did}/active/{delegate_did}.json"
    ) or []
    
    active_tokens.append({
        'rep_id': rep_id,
        'issued_at': rep_token['issued_at'],
        'expires_at': rep_token['constraints']['expires_at'],
        'scope_summary': {
            'actions': rep_token['scope']['actions'][:3],  // First 3 actions
            'domains': rep_token['scope']['domains'][:2]   // First 2 domains
        }
    })
    
    shared_ledger.put(
        path=f"delegations/{issuer_did}/active/{delegate_did}.json",
        data=active_tokens
    )
```

### Querying Delegation
```python
def query_delegation_from_ledger(issuer_did, rep_id):
    """
    Query delegation token from shared ledger
    """
    path = f"delegations/{issuer_did}/{rep_id}.json"
    token = shared_ledger.get(path)
    
    if not token:
        return None
    
    # Verify signature
    sig_valid = verify_issuer_signature(token)
    if not sig_valid:
        raise SecurityError(f"Invalid signature on delegation {rep_id}")
    
    return token
```

### Recording Usage
```python
def record_delegation_usage(rep_id, agent_id, task, outcome):
    """
    Record delegation token usage for audit trail
    """
    usage_entry = {
        'timestamp': datetime.utcnow().isoformat() + 'Z',
        'agent_id': agent_id,
        'task_summary': {
            'description': task['description'][:100],
            'cost': task.get('estimated_cost', 0),
            'duration': task.get('actual_duration', 0)
        },
        'outcome': outcome
    }
    
    # Append to usage log
    usage_log = shared_ledger.get(
        f"delegation_usage/{rep_id}/usage_log.json"
    ) or []
    
    usage_log.append(usage_entry)
    
    shared_ledger.put(
        path=f"delegation_usage/{rep_id}/usage_log.json",
        data=usage_log
    )
```

---

## Implementation Guide

### Minimal Implementation

**Required components:**
- [ ] Token structure (all required fields)
- [ ] Ed25519 signature generation (issuer)
- [ ] Ed25519 signature verification
- [ ] TTL expiration check
- [ ] Scope verification (actions + domains)
- [ ] Limits verification
- [ ] Canonical JSON serialization

**Recommended components:**
- [ ] Age-triggered revalidation
- [ ] Rate limit tracking
- [ ] Usage logging
- [ ] Revocation registry support
- [ ] Multi-hop delegation support

### Reference Implementation
```python
import nacl.signing
import base58
import json
from datetime import datetime, timedelta
import uuid

class AEXDelegation:
    """Reference implementation of AEX_REP"""
    
    def __init__(self, issuer_signing_key):
        """
        Initialize with issuer's signing key
        """
        self.signing_key = issuer_signing_key
        self.issuer_did = self._derive_did()
    
    def _derive_did(self):
        """Derive DID from signing key"""
        public_key = self.signing_key.verify_key.encode()
        pubkey_b58 = base58.b58encode(public_key).decode('ascii')
        return f"did:aex:{pubkey_b58}"
    
    def create_delegation(self, delegate_did, scope, constraints, metadata=None):
        """
        Create and sign delegation token
        """
        # Ensure expires_at is set
        if 'expires_at' not in constraints:
            # Default to 4 hours from now
            expires = datetime.utcnow() + timedelta(hours=4)
            constraints['expires_at'] = expires.isoformat() + 'Z'
        
        # Create token
        token = {
            'rep_id': str(uuid.uuid4()),
            'version': '1.0',
            'issuer': self.issuer_did,
            'delegate': delegate_did,
            'scope': scope,
            'constraints': constraints,
            'issued_at': datetime.utcnow().isoformat() + 'Z',
            'issuer_metadata': metadata or {}
        }
        
        # Sign token
        return self._sign_token(token)
    
    def _sign_token(self, token):
        """Sign delegation token"""
        # Remove signature if present
        token.pop('signature', None)
        
        # Create canonical JSON
        canonical = json.dumps(
            token,
            sort_keys=True,
            separators=(',', ':'),
            ensure_ascii=False
        ).encode('utf-8')
        
        # Sign
        signed = self.signing_key.sign(canonical)
        signature_b58 = base58.b58encode(signed.signature).decode('ascii')
        
        # Attach signature
        token['signature'] = signature_b58
        return token
    
    def verify_token(self, token, issuer_public_key):
        """Verify delegation token signature"""
        # Extract signature
        signature_b58 = token.pop('signature')
        signature = base58.b58decode(signature_b58)
        
        # Reconstruct canonical
        canonical = json.dumps(
            token,
            sort_keys=True,
            separators=(',', ':'),
            ensure_ascii=False
        ).encode('utf-8')
        
        # Verify
        verify_key = nacl.signing.VerifyKey(issuer_public_key)
        try:
            verify_key.verify(canonical, signature)
            return True
        except nacl.exceptions.BadSignatureError:
            return False
    
    def check_ttl(self, token):
        """Check if token is still valid"""
        now = datetime.utcnow()
        expires = datetime.fromisoformat(
            token['constraints']['expires_at'].replace('Z', '+00:00')
        )
        
        if now > expires:
            return False, "Token expired"
        
        # Calculate age
        issued = datetime.fromisoformat(
            token['issued_at'].replace('Z', '+00:00')
        )
        lifetime = (expires - issued).total_seconds()
        age = (now - issued).total_seconds()
        age_pct = age / lifetime
        
        if age_pct > 0.8:
            return "AGING", f"Token at {age_pct*100:.0f}% of lifetime"
        
        return True, f"{(expires - now).total_seconds():.0f}s remaining"
    
    def verify_scope(self, token, proposed_actions, proposed_domains):
        """Verify scope covers proposed actions/domains"""
        # Check actions
        for action in proposed_actions:
            if action not in token['scope']['actions']:
                return False, f"Action '{action}' not in scope"
            if action in token['constraints'].get('prohibited_actions', []):
                return False, f"Action '{action}' prohibited"
        
        # Check domains
        for domain in proposed_domains:
            if domain not in token['scope']['domains']:
                return False, f"Domain '{domain}' not in scope"
            if domain in token['constraints'].get('prohibited_domains', []):
                return False, f"Domain '{domain}' prohibited"
        
        return True, "Scope valid"
    
    def revoke_token(self, rep_id, reason):
        """
        Create revocation record (off-protocol)
        """
        revocation = {
            'rep_id': rep_id,
            'issuer': self.issuer_did,
            'revoked_at': datetime.utcnow().isoformat() + 'Z',
            'reason': reason,
            'status': 'revoked'
        }
        
        # Sign revocation
        return self._sign_token(revocation)

# Usage example:
issuer_key = nacl.signing.SigningKey.generate()
delegation = AEXDelegation(issuer_key)

token = delegation.create_delegation(
    delegate_did="did:aex:AgentPublicKey",
    scope={
        'actions': ['research', 'analyze'],
        'domains': ['finance'],
        'limits': {'max_cost': 100}
    },
    constraints={
        'expires_at': (datetime.utcnow() + timedelta(hours=4)).isoformat() + 'Z',
        'prohibited_actions': ['purchase']
    }
)

print(json.dumps(token, indent=2))
```

---

## Security Considerations

### Key Management

**Issuer private key security:**
- ✅ Store in secure enclave (HSM, TPM)
- ✅ Require multi-factor authentication
- ✅ Use separate keys for different agents
- ✅ Rotate keys periodically
- ❌ Never log or transmit private keys

### Attack Vectors

**1. Token theft:**
```
Attack: Attacker steals delegation token
Impact: Can impersonate agent until TTL expires
Mitigation:
  - Short TTLs (1-4 hours)
  - Age-triggered revalidation
  - Monitor usage patterns
Recovery:
  - Revoke via registry
  - Wait for TTL expiry
  - Issue new token to legitimate agent
```

**2. Scope inflation:**
```
Attack: Agent modifies token to expand scope
Impact: Unauthorized actions
Mitigation:
  - Signature verification (catches modification)
  - All modifications invalidate signature
Prevention:
  - Always verify signature before trusting token
```

**3. Replay attack:**
```
Attack: Reuse expired token
Impact: None if TTL checked
Mitigation:
  - Always check expires_at
  - Reject expired tokens
Prevention:
  - Mandatory timestamp verification
```

**4. Sub-delegation abuse:**
```
Attack: Agent delegates beyond allowed scope
Impact: Pyramid of unauthorized authority
Mitigation:
  - Verify delegation chain
  - Enforce scope narrowing
  - Limit chain depth
Prevention:
  - Check allow_sub_delegation flag
  - Validate entire chain
```

### Privacy Considerations

**What AEX_REP reveals:**
- ✅ Issuer identity
- ✅ Delegate identity
- ✅ Permitted actions/domains
- ✅ Limits and constraints
- ✅ Issuance and expiry times

**What it does NOT reveal:**
- ❌ Issuer's private key
- ❌ Past usage history (unless logged)
- ❌ Other delegations by same issuer

---

## Examples

### Example 1: Simple Research Delegation
```json
{
  "rep_id": "550e8400-e29b-41d4-a716-446655440001",
  "version": "1.0",
  "issuer": "did:human:alice",
  "delegate": "did:aex:ResearchAgent",
  "scope": {
    "actions": ["research", "summarize"],
    "domains": ["technology"],
    "limits": {
      "max_cost": 10,
      "max_duration": 1800
    }
  },
  "constraints": {
    "expires_at": "2026-02-04T16:00:00Z",
    "prohibited_actions": ["purchase"],
    "prohibited_domains": ["finance", "legal"],
    "human_approval_required": false,
    "rate_limits": {
      "requests_per_hour": 10
    }
  },
  "issued_at": "2026-02-04T12:00:00Z",
  "issuer_metadata": {
    "issuer_name": "Alice Johnson",
    "purpose": "Tech trend research"
  },
  "signature": "3yZe7d...9kL2m"
}
```

### Example 2: High-Stakes Financial Analysis
```json
{
  "rep_id": "high-stakes-uuid",
  "version": "1.0",
  "issuer": "did:human:bob",
  "delegate": "did:aex:FinancialAnalyst",
  "scope": {
    "actions": ["analyze", "draft", "notify"],
    "domains": ["finance", "finance.tax"],
    "limits": {
      "max_cost": 500,
      "max_duration": 7200,
      "max_tasks_per_day": 3
    }
  },
  "constraints": {
    "expires_at": "2026-02-04T14:00:00Z",
    "prohibited_actions": ["purchase", "sign_contracts", "commit_funds"],
    "prohibited_domains": ["legal"],
    "human_approval_required": true,
    "approval_contact": "bob@example.com",
    "approval_timeout": 600,
    "witness_required": true,
    "min_witnesses": 2,
    "min_witness_dex": 0.85,
    "rate_limits": {
      "requests_per_hour": 5,
      "cost_per_day": 1000
    }
  },
  "issued_at": "2026-02-04T12:00:00Z",
  "issuer_metadata": {
    "issuer_name": "Bob Smith",
    "purpose": "Q4 2025 tax preparation",
    "contact": "bob@example.com"
  },
  "signature": "8kP4x...3mN9q"
}
```

### Example 3: Multi-Hop Delegation

**Level 1: Human → Agent A**
```json
{
  "rep_id": "rep_level_1",
  "issuer": "did:human:charlie",
  "delegate": "did:aex:AgentA",
  "scope": {
    "actions": ["research", "analyze", "coordinate"],
    "domains": ["healthcare"],
    "limits": {"max_cost": 200}
  },
  "constraints": {
    "expires_at": "2026-02-05T12:00:00Z",
    "allow_sub_delegation": true,
    "sub_delegation_constraints": {
      "max_depth": 1,
      "scope_must_narrow": true
    }
  },
  "issued_at": "2026-02-04T12:00:00Z"
}
```

**Level 2: Agent A → Agent B**
```json
{
  "rep_id": "rep_level_2",
  "issuer": "did:aex:AgentA",
  "delegate": "did:aex:AgentB",
  "scope": {
    "actions": ["research"],
    "domains": ["healthcare"],
    "limits": {"max_cost": 100}
  },
  "constraints": {
    "expires_at": "2026-02-05T11:00:00Z",
    "parent_rep_id": "rep_level_1",
    "delegation_chain": ["rep_level_1"],
    "allow_sub_delegation": false
  },
  "issued_at": "2026-02-04T13:00:00Z"
}
```

### Example 4: Expired Token
```json
{
  "rep_id": "expired-uuid",
  "issuer": "did:human:diana",
  "delegate": "did:aex:OldAgent",
  "scope": {
    "actions": ["monitor"],
    "domains": ["operations"],
    "limits": {}
  },
  "constraints": {
    "expires_at": "2026-02-03T12:00:00Z"  // Yesterday
  },
  "issued_at": "2026-02-02T12:00:00Z"
}

// Verification result:
{
  "valid": false,
  "reason": "Token expired",
  "expired_at": "2026-02-03T12:00:00Z",
  "current_time": "2026-02-04T12:00:00Z"
}
```

### Example 8: Delegation with HEX Verification

**Scenario:** Human delegates translation work to agent, counterparty verifies both authorization and capability.

**REP Token:**
```json
{
  "rep_id": "rep_uuid_123",
  "issuer": "did:human:alice",
  "delegate": "did:aex:translator_bot_456",
  "scope": {
    "actions": ["translate", "proofread"],
    "domains": ["documents", "correspondence"],
    "limits": {
      "max_cost": 50,
      "max_tasks_per_day": 20
    }
  },
  "constraints": {
    "expires_at": "2026-02-06T00:00:00Z",
    "prohibited_domains": ["legal", "medical"]
  }
}
```

**Delegate's Credentials:**
```json
{
  "aex_id": "did:aex:translator_bot_456",
  "dex": {
    "alpha": 95,
    "beta": 15,
    "score": 0.864
  },
  "hex": {
    "experience": [
      {"domain": "translation.fr_en", "count": 180, "confidence": 0.94},
      {"domain": "translation.es_en", "count": 120, "confidence": 0.91},
      {"domain": "proofreading", "count": 200, "confidence": 0.96}
    ]
  }
}
```

**Verification flow:**
```
Task: Translate French document to English

Step 1: Verify REP token ✅
- Valid signature from Alice
- Not expired
- Action "translate" in scope
- Domain "documents" in scope

Step 2: Verify delegate DEX ✅
- Score: 0.864 (above 0.75 threshold)
- Confidence: 110 (sufficient evidence)
- Not in probation

Step 3: Verify delegate HEX ✅
- Has domain: translation.fr_en
- Count: 180 (experienced)
- Confidence: 0.94 (consistent performance)

Result: ACCEPT
Reasoning: Agent has authority (REP), is reliable (DEX), and is capable (HEX)
```

**Contrast: REP without capability:**
```json
{
  "delegate": "did:aex:general_assistant_789",
  "dex": {"score": 0.90},  // Highly reliable
  "hex": {
    "experience": [
      {"domain": "scheduling", "count": 500},
      {"domain": "email", "count": 300}
    ]
    // No translation experience!
  }
}
```

**Verification flow:**
```
Step 1: Verify REP token ✅
Step 2: Verify delegate DEX ✅
Step 3: Verify delegate HEX ❌
- Missing domain: translation.fr_en

Result: REJECT
Reasoning: Agent has authority and is reliable, but lacks capability
```

**Key insight:** REP provides authorization, but HEX determines whether the delegated agent is actually suited for the specific task.

---

## Related Specifications

AEX_REP defines authorization that works with:

**AEX_ID** - Identity foundation
- REP tokens reference agent identity (delegate field)
- Fork events may or may not invalidate REP tokens
- Issuer decides whether forked agent retains authorization

**AEX_DEX** - Behavioral reputation
- Delegated agent's DEX should still be evaluated
- Low DEX agent + valid REP = authorized but risky
- Counterparties can set DEX thresholds for delegated agents

**AEX_HEX** - Experience corpus
- Delegated agent's HEX determines task suitability
- REP grants authority, HEX proves capability
- Joint evaluation: "Authorized AND capable?"

**Selection with REP:**
```
1. Verify REP token (authorized?)
2. Check delegate DEX (reliable?)
3. Check delegate HEX (capable?)
4. Only proceed if all three pass
```

**Key principle:** REP is necessary but not sufficient. The delegate's actual capabilities (HEX) and reliability (DEX) still matter.

---

**Status:** AEX_REP v1.0 complete and normative  
**Next Document:** AEX_DEX.md (Reputation specification)
