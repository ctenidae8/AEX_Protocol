# AEX Protocol v1.0 - Implementation Guide
## Reference Implementations, Best Practices, and Integration Patterns

**Last Updated:** February 4, 2026

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture Patterns](#architecture-patterns)
3. [Reference Implementation](#reference-implementation)
4. [Ledger Storage](#ledger-storage)
5. [Cryptography](#cryptography)
6. [Network Communication](#network-communication)
7. [Integration Patterns](#integration-patterns)
8. [Performance Optimization](#performance-optimization)
9. [Testing Strategy](#testing-strategy)
10. [Deployment Guide](#deployment-guide)
11. [Troubleshooting](#troubleshooting)
12. [Migration and Versioning](#migration-and-versioning)

---

## Overview

This document provides **practical implementation guidance** for the AEX protocol family, including:

- Recommended architecture patterns
- Complete reference implementation in Python
- Storage and persistence strategies
- Cryptographic best practices
- Network protocol options
- Integration with existing systems
- Performance optimization techniques
- Testing and deployment procedures

### Design Philosophy

AEX implementations should be:

1. **Minimal** - Only implement what the protocol requires
2. **Portable** - Work across platforms and environments
3. **Secure by default** - Safe configurations out of the box
4. **Observable** - Extensive logging and monitoring
5. **Testable** - Easy to verify correctness

### Technology Choices

**Recommended:**
- **Language:** Python 3.10+ (reference), Rust, Go, TypeScript
- **Crypto:** PyNaCl/libsodium (Ed25519), cryptography.io
- **Storage:** SQLite (local), PostgreSQL (shared), S3 (archives)
- **Serialization:** JSON (human-readable), CBOR (efficient)
- **Transport:** HTTPS/REST, gRPC, WebSocket

**Avoid:**
- Custom crypto implementations
- Unencrypted network protocols
- Mutable ledger backends
- Blocking I/O in critical paths

---

## Architecture Patterns

### Pattern 1: Embedded Agent

**Use Case:** Single agent embedded in application
```
┌─────────────────────────────────┐
│      Host Application           │
│  ┌──────────────────────────┐  │
│  │   AEX Agent Library      │  │
│  │  ┌────────┐ ┌─────────┐ │  │
│  │  │Identity│ │Handshake│ │  │
│  │  └────────┘ └─────────┘ │  │
│  │  ┌────────┐ ┌─────────┐ │  │
│  │  │DEX Calc│ │Sessions │ │  │
│  │  └────────┘ └─────────┘ │  │
│  └──────────────────────────┘  │
│  ┌──────────────────────────┐  │
│  │   Local Ledger (SQLite)  │  │
│  └──────────────────────────┘  │
└─────────────────────────────────┘
```

**Pros:**
- Simple deployment
- Low latency
- No network dependencies
- Easy debugging

**Cons:**
- Limited to single agent
- No shared infrastructure
- Harder to upgrade

**Example:**
```python
# Embedded agent in application
from aex import Agent, LocalLedger

# Initialize agent
agent = Agent(
    identity_path="~/.aex/identity.json",
    ledger=LocalLedger("~/.aex/ledger.db")
)

# Use in application
async def perform_task(counterparty_did):
    # Execute AEX handshake
    handshake = await agent.initiate_handshake(counterparty_did)
    
    if not handshake.success:
        raise ValueError(f"Handshake failed: {handshake.reason}")
    
    # Do work...
    result = await do_work()
    
    # Record session
    await agent.record_session(
        handshake_id=handshake.id,
        outcome=0.9,
        task_summary="Data processing"
    )
```

---

### Pattern 2: Agent Service

**Use Case:** Multiple agents, shared infrastructure
```
┌──────────────────────────────────────┐
│         Agent Service                │
│  ┌────────────────────────────────┐ │
│  │      Agent Manager             │ │
│  │  ┌─────────┐  ┌─────────┐     │ │
│  │  │Agent A  │  │Agent B  │ ... │ │
│  │  └─────────┘  └─────────┘     │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │   Shared Ledger (PostgreSQL)   │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │   REST API / gRPC Interface    │ │
│  └────────────────────────────────┘ │
└──────────────────────────────────────┘
         ▲
         │ HTTP/gRPC
         │
┌────────┴───────┐
│  Client Apps   │
└────────────────┘
```

**Pros:**
- Centralized management
- Shared storage and caching
- Easy monitoring
- Horizontal scaling

**Cons:**
- Network overhead
- Single point of failure
- More complex deployment

**Example:**
```python
# Agent service with REST API
from fastapi import FastAPI, HTTPException
from aex import AgentManager, SharedLedger

app = FastAPI()
manager = AgentManager(ledger=SharedLedger("postgresql://..."))

@app.post("/agents/{agent_id}/handshake")
async def initiate_handshake(agent_id: str, counterparty_did: str):
    agent = manager.get_agent(agent_id)
    handshake = await agent.initiate_handshake(counterparty_did)
    return handshake.to_dict()

@app.post("/agents/{agent_id}/sessions")
async def record_session(agent_id: str, session: SessionCreate):
    agent = manager.get_agent(agent_id)
    result = await agent.record_session(**session.dict())
    return result
```

---

### Pattern 3: Federated Network

**Use Case:** Multiple organizations, shared reputation
```
┌─────────────────┐       ┌─────────────────┐
│  Organization A │       │  Organization B │
│  ┌───────────┐  │       │  ┌───────────┐  │
│  │Agent Svc  │  │◄─────►│  │Agent Svc  │  │
│  └───────────┘  │       │  └───────────┘  │
│  ┌───────────┐  │       │  ┌───────────┐  │
│  │Local Ledg │  │       │  │Local Ledg │  │
│  └───────────┘  │       │  └───────────┘  │
└─────────────────┘       └─────────────────┘
         │                         │
         └────────┬────────────────┘
                  │
         ┌────────▼────────┐
         │ Shared Ledger   │
         │  (Blockchain/   │
         │   Distributed)  │
         └─────────────────┘
```

**Pros:**
- Cross-organization trust
- No single authority
- High availability
- Geographic distribution

**Cons:**
- Complex consensus
- Network latency
- Harder to debug
- Requires coordination

---

## Reference Implementation

### Core Classes

#### 1. AEXAgent
```python
"""
Core agent implementation
"""

import json
import hashlib
from typing import Optional, Dict, List
from datetime import datetime, timedelta
import nacl.signing
import nacl.encoding
import base58

class AEXAgent:
    """
    Complete AEX protocol implementation
    
    Handles:
    - Identity management
    - Handshake protocol
    - DEX calculation
    - Session recording
    - Delegation verification
    """
    
    def __init__(
        self,
        identity: Optional[Dict] = None,
        ledger: 'Ledger' = None,
        shared_ledger: Optional['SharedLedger'] = None
    ):
        """
        Initialize agent
        
        Args:
            identity: Existing identity dict, or None to generate new
            ledger: Local ledger for storage
            shared_ledger: Optional shared ledger for network queries
        """
        self.ledger = ledger or LocalLedger()
        self.shared_ledger = shared_ledger
        
        if identity:
            self.identity = identity
            self.signing_key = self._load_signing_key(identity)
        else:
            self.identity, self.signing_key = self._generate_identity()
            self._save_identity()
    
    def _generate_identity(self) -> tuple:
        """
        Generate new AEX_ID with Ed25519 keypair
        
        Returns:
            (identity_dict, signing_key)
        """
        # Generate keypair
        signing_key = nacl.signing.SigningKey.generate()
        verify_key = signing_key.verify_key
        
        # Create DID
        pubkey_bytes = bytes(verify_key)
        pubkey_b58 = base58.b58encode(pubkey_bytes).decode('ascii')
        did = f"did:aex:{pubkey_b58}"
        
        # Build identity object
        identity = {
            "aex_id": did,
            "public_key": pubkey_b58,
            "created_at": datetime.utcnow().isoformat() + 'Z',
            "lineage": {
                "parent": None,
                "fork_history": []
            },
            "stake": {
                "amount": 0,
                "locked_at": None
            },
            "metadata": {
                "version": "1.0",
                "created_by": "aex-python-reference"
            }
        }
        
        # Self-sign
        canonical = self._canonicalize(identity)
        signature = signing_key.sign(canonical)
        
        identity["signature"] = base58.b58encode(signature.signature).decode('ascii')
        
        return identity, signing_key
    
    def _canonicalize(self, obj: Dict) -> bytes:
        """
        Create canonical JSON representation for signing
        
        Args:
            obj: Dictionary to canonicalize
        
        Returns:
            UTF-8 encoded canonical JSON
        """
        # Remove signature if present
        obj_copy = {k: v for k, v in obj.items() if k != 'signature'}
        
        # Canonical JSON: sorted keys, no whitespace
        canonical = json.dumps(
            obj_copy,
            sort_keys=True,
            separators=(',', ':'),
            ensure_ascii=False
        )
        
        return canonical.encode('utf-8')
    
    def initiate_handshake(
        self,
        counterparty_did: str,
        rep_token: Optional[Dict] = None
    ) -> 'HandshakeResult':
        """
        Execute AEX handshake protocol
        
        Args:
            counterparty_did: DID of agent to interact with
            rep_token: Optional delegation token if acting on behalf of human
        
        Returns:
            HandshakeResult with success/failure and details
        """
        handshake = AEXHandshake(
            initiator=self,
            counterparty_did=counterparty_did,
            rep_token=rep_token,
            ledger=self.ledger,
            shared_ledger=self.shared_ledger
        )
        
        return handshake.execute()
    
    def calculate_dex(self) -> Dict:
        """
        Calculate current DEX parameters
        
        Returns:
            Dict with alpha, beta, dex, n_eff, confidence_interval
        """
        sessions = self.ledger.get_agent_sessions(self.identity['aex_id'])
        
        # Start with prior
        alpha = 2.0
        beta = 2.0
        
        # Process each session
        for session in sessions:
            outcome = session['outcome']['agreed_outcome']
            weight = session.get('weight', 1.0)
            lineage_factor = session.get('lineage_factor', 1.0)
            
            # Bayesian update
            alpha += outcome * weight * lineage_factor
            beta += (1 - outcome) * weight * lineage_factor
        
        # Calculate derived values
        dex = alpha / (alpha + beta)
        n_eff = alpha + beta
        variance = (alpha * beta) / ((alpha + beta)**2 * (alpha + beta + 1))
        
        # Confidence interval (95%)
        from scipy.stats import beta as beta_dist
        ci_lower = beta_dist.ppf(0.025, alpha, beta)
        ci_upper = beta_dist.ppf(0.975, alpha, beta)
        
        return {
            'alpha': alpha,
            'beta': beta,
            'dex': dex,
            'n_eff': n_eff,
            'variance': variance,
            'confidence_interval': {
                'lower': ci_lower,
                'upper': ci_upper,
                'width': ci_upper - ci_lower
            },
            'last_updated': datetime.utcnow().isoformat() + 'Z'
        }
    
    def record_session(
        self,
        handshake_id: str,
        outcome: float,
        task_summary: Dict,
        witnesses: Optional[List[Dict]] = None
    ) -> Dict:
        """
        Record completed session
        
        Args:
            handshake_id: ID of completed handshake
            outcome: Outcome rating [0,1]
            task_summary: Task details
            witnesses: Optional witness attestations
        
        Returns:
            Published session dict
        """
        # Retrieve handshake
        handshake = self.ledger.get_handshake(handshake_id)
        if not handshake:
            raise ValueError(f"Handshake {handshake_id} not found")
        
        # Create session
        session = AEXSession.create(
            handshake=handshake,
            task_summary=task_summary,
            agent_rating=outcome,
            witnesses=witnesses
        )
        
        # Sign and publish
        session.sign(self.signing_key)
        self.ledger.publish_session(session.to_dict())
        
        # Update DEX
        dex = self.calculate_dex()
        self.ledger.update_dex(self.identity['aex_id'], dex)
        
        return session.to_dict()
    
    def verify_delegation(self, rep_token: Dict) -> tuple[bool, str]:
        """
        Verify AEX_REP delegation token
        
        Args:
            rep_token: Delegation token to verify
        
        Returns:
            (valid, reason)
        """
        # Check required fields
        required = ['rep_id', 'issuer', 'delegate', 'scope', 'constraints', 'issued_at', 'signature']
        if not all(k in rep_token for k in required):
            return False, "Missing required fields"
        
        # Verify delegate matches this agent
        if rep_token['delegate'] != self.identity['aex_id']:
            return False, "Delegate mismatch"
        
        # Check expiration
        expires_at = datetime.fromisoformat(
            rep_token['constraints']['expires_at'].replace('Z', '+00:00')
        )
        if datetime.utcnow() > expires_at.replace(tzinfo=None):
            return False, "Token expired"
        
        # Check TTL-based revalidation
        issued_at = datetime.fromisoformat(rep_token['issued_at'].replace('Z', '+00:00'))
        age = (datetime.utcnow() - issued_at.replace(tzinfo=None)).total_seconds()
        lifetime = (expires_at - issued_at).total_seconds()
        
        if age / lifetime > 0.8:
            # Token is >80% through lifetime, revalidate
            is_revoked = self._check_revocation(rep_token)
            if is_revoked:
                return False, "Token revoked"
        
        # Verify issuer signature
        issuer_did = rep_token['issuer']
        issuer_identity = self._get_identity(issuer_did)
        
        if not issuer_identity:
            return False, "Issuer identity not found"
        
        try:
            issuer_pubkey = base58.b58decode(issuer_identity['public_key'])
            verify_key = nacl.signing.VerifyKey(issuer_pubkey)
            
            canonical = self._canonicalize(rep_token)
            signature = base58.b58decode(rep_token['signature'])
            
            verify_key.verify(canonical, signature)
            return True, "Token valid"
        
        except Exception as e:
            return False, f"Signature verification failed: {e}"
    
    def fork(
        self,
        fork_type: str,
        reason: str,
        metadata: Optional[Dict] = None
    ) -> 'AEXAgent':
        """
        Fork this agent to create child with new identity
        
        Args:
            fork_type: 'bugfix', 'major_rewrite', or 'hard_override'
            reason: Human-readable reason for fork
            metadata: Optional additional metadata
        
        Returns:
            New AEXAgent instance (child)
        """
        # Generate new identity
        child_agent = AEXAgent(ledger=self.ledger, shared_ledger=self.shared_ledger)
        
        # Set parent lineage
        child_agent.identity['lineage']['parent'] = self.identity['aex_id']
        
        # Record fork event
        fork_event = {
            'fork_id': self._generate_uuid(),
            'parent_id': self.identity['aex_id'],
            'child_id': child_agent.identity['aex_id'],
            'fork_type': fork_type,
            'reason': reason,
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'metadata': metadata or {}
        }
        
        # Sign fork event
        canonical = self._canonicalize(fork_event)
        signature = self.signing_key.sign(canonical)
        fork_event['signature'] = base58.b58encode(signature.signature).decode('ascii')
        
        # Add to child's fork history
        child_agent.identity['lineage']['fork_history'].append(fork_event)
        
        # Initialize child DEX with inherited parameters
        fork_weights = {
            'bugfix': 1.0,
            'major_rewrite': 0.5,
            'hard_override': 0.1
        }
        
        weight = fork_weights.get(fork_type, 0.5)
        parent_dex = self.calculate_dex()
        
        child_alpha = parent_dex['alpha'] * weight + 2
        child_beta = parent_dex['beta'] * weight + 2
        
        child_dex = {
            'alpha': child_alpha,
            'beta': child_beta,
            'dex': child_alpha / (child_alpha + child_beta),
            'n_eff': child_alpha + child_beta,
            'inherited_from': self.identity['aex_id'],
            'fork_weight': weight,
            'last_updated': datetime.utcnow().isoformat() + 'Z'
        }
        
        # Save child identity and DEX
        child_agent._save_identity()
        child_agent.ledger.update_dex(child_agent.identity['aex_id'], child_dex)
        
        return child_agent
    
    def _generate_uuid(self) -> str:
        """Generate UUID for IDs"""
        import uuid
        return str(uuid.uuid4())
    
    def _save_identity(self):
        """Save identity to ledger"""
        self.ledger.save_identity(self.identity)
    
    def _load_signing_key(self, identity: Dict) -> nacl.signing.SigningKey:
        """Load signing key from secure storage"""
        # In production, load from HSM/TPM/OS keychain
        # This is simplified for reference
        seed = hashlib.sha256(identity['aex_id'].encode()).digest()
        return nacl.signing.SigningKey(seed)
    
    def _get_identity(self, did: str) -> Optional[Dict]:
        """Get identity from local or shared ledger"""
        # Try local first
        identity = self.ledger.get_identity(did)
        if identity:
            return identity
        
        # Try shared if available
        if self.shared_ledger:
            return self.shared_ledger.get_identity(did)
        
        return None
    
    def _check_revocation(self, rep_token: Dict) -> bool:
        """Check if token has been revoked"""
        # Query issuer's revocation registry
        issuer_did = rep_token['issuer']
        rep_id = rep_token['rep_id']
        
        # Try shared ledger
        if self.shared_ledger:
            revocations = self.shared_ledger.get_revocations(issuer_did)
            return any(r['rep_id'] == rep_id for r in revocations)
        
        return False
```

---

#### 2. AEXHandshake
```python
"""
Handshake protocol implementation
"""

from enum import Enum
from typing import Optional, Dict

class HandshakeStep(Enum):
    """Handshake protocol steps"""
    INTRODUCTION = 1
    IDENTITY_CHECK = 2
    REPUTATION_CHECK = 3
    REPRESENTATION_CHECK = 4
    DECISION = 5
    FINALIZE = 6

class HandshakeResult:
    """Result of handshake execution"""
    
    def __init__(
        self,
        success: bool,
        handshake_id: str,
        reason: str = "",
        details: Optional[Dict] = None
    ):
        self.success = success
        self.handshake_id = handshake_id
        self.reason = reason
        self.details = details or {}
    
    def to_dict(self) -> Dict:
        return {
            'success': self.success,
            'handshake_id': self.handshake_id,
            'reason': self.reason,
            'details': self.details
        }

class AEXHandshake:
    """
    AEX handshake protocol execution
    
    Implements 8-step handshake with verification at each stage
    """
    
    def __init__(
        self,
        initiator: 'AEXAgent',
        counterparty_did: str,
        rep_token: Optional[Dict],
        ledger: 'Ledger',
        shared_ledger: Optional['SharedLedger']
    ):
        self.initiator = initiator
        self.counterparty_did = counterparty_did
        self.rep_token = rep_token
        self.ledger = ledger
        self.shared_ledger = shared_ledger
        
        self.handshake_id = self._generate_uuid()
        self.current_step = HandshakeStep.INTRODUCTION
    
    def execute(self) -> HandshakeResult:
        """
        Execute complete handshake protocol
        
        Returns:
            HandshakeResult with outcome
        """
        try:
            # Step 1: Introduction
            self._step_introduction()
            
            # Step 2: Identity Check
            identity_valid, reason = self._step_identity_check()
            if not identity_valid:
                return self._fail(reason)
            
            # Step 3: Reputation Check
            reputation_acceptable, reason = self._step_reputation_check()
            if not reputation_acceptable:
                return self._fail(reason)
            
            # Step 4: Representation Check
            if self.rep_token:
                rep_valid, reason = self._step_representation_check()
                if not rep_valid:
                    return self._fail(reason)
            
            # Step 5: Decision
            proceed, reason = self._step_decision()
            if not proceed:
                return self._fail(reason)
            
            # Step 6: Finalize
            self._step_finalize()
            
            return HandshakeResult(
                success=True,
                handshake_id=self.handshake_id,
                reason="Handshake successful"
            )
        
        except Exception as e:
            return self._fail(f"Handshake error: {e}")
    
    def _step_introduction(self):
        """Step 1: Exchange identities and tokens"""
        self.current_step = HandshakeStep.INTRODUCTION
        
        # Record introduction
        intro = {
            'handshake_id': self.handshake_id,
            'initiator_did': self.initiator.identity['aex_id'],
            'counterparty_did': self.counterparty_did,
            'rep_token': self.rep_token,
            'timestamp': datetime.utcnow().isoformat() + 'Z'
        }
        
        self.ledger.save_handshake(self.handshake_id, intro)
    
    def _step_identity_check(self) -> tuple[bool, str]:
        """Step 2: Verify counterparty identity"""
        self.current_step = HandshakeStep.IDENTITY_CHECK
        
        # Get counterparty identity
        counterparty = self._get_identity(self.counterparty_did)
        if not counterparty:
            return False, "Counterparty identity not found"
        
        # Verify signature
        try:
            pubkey = base58.b58decode(counterparty['public_key'])
            verify_key = nacl.signing.VerifyKey(pubkey)
            
            canonical = self.initiator._canonicalize(counterparty)
            signature = base58.b58decode(counterparty['signature'])
            
            verify_key.verify(canonical, signature)
        except Exception as e:
            return False, f"Identity signature verification failed: {e}"
        
        # Check fork history if exists
        if counterparty['lineage']['fork_history']:
            # Validate fork chain
            valid, reason = self._validate_fork_chain(counterparty)
            if not valid:
                return False, reason
        
        return True, "Identity verified"
    
    def _step_reputation_check(self) -> tuple[bool, str]:
        """Step 3: Evaluate counterparty reputation"""
        self.current_step = HandshakeStep.REPUTATION_CHECK
        
        # Get DEX parameters
        dex = self._get_counterparty_dex()
        if not dex:
            return False, "No reputation data available"
        
        # Check minimum thresholds
        min_dex = 0.70  # Configurable
        min_confidence = 20  # Configurable
        
        if dex['dex'] < min_dex:
            return False, f"DEX {dex['dex']:.3f} below threshold {min_dex}"
        
        if dex['n_eff'] < min_confidence:
            return False, f"Confidence {dex['n_eff']} below threshold {min_confidence}"
        
        # Check if in probation
        if dex.get('probation', {}).get('active', False):
            return False, "Agent currently in probation"
        
        return True, "Reputation acceptable"
    
    def _step_representation_check(self) -> tuple[bool, str]:
        """Step 4: Verify delegation if present"""
        self.current_step = HandshakeStep.REPRESENTATION_CHECK
        
        if not self.rep_token:
            return True, "No representation to verify"
        
        # Verify delegation token
        valid, reason = self.initiator.verify_delegation(self.rep_token)
        
        return valid, reason
    
    def _step_decision(self) -> tuple[bool, str]:
        """Step 5: Make final proceed/decline decision"""
        self.current_step = HandshakeStep.DECISION
        
        # All checks passed, proceed
        return True, "Proceeding with interaction"
    
    def _step_finalize(self):
        """Step 6: Finalize handshake, record outcome"""
        self.current_step = HandshakeStep.FINALIZE
        
        # Update handshake record
        handshake = self.ledger.get_handshake(self.handshake_id)
        handshake['status'] = 'completed'
        handshake['completed_at'] = datetime.utcnow().isoformat() + 'Z'
        
        self.ledger.update_handshake(self.handshake_id, handshake)
    
    def _fail(self, reason: str) -> HandshakeResult:
        """Record handshake failure"""
        handshake = self.ledger.get_handshake(self.handshake_id)
        handshake['status'] = 'failed'
        handshake['failed_at'] = datetime.utcnow().isoformat() + 'Z'
        handshake['failure_reason'] = reason
        
        self.ledger.update_handshake(self.handshake_id, handshake)
        
        return HandshakeResult(
            success=False,
            handshake_id=self.handshake_id,
            reason=reason
        )
    
    def _get_identity(self, did: str) -> Optional[Dict]:
        """Get identity from ledgers"""
        identity = self.ledger.get_identity(did)
        if identity:
            return identity
        
        if self.shared_ledger:
            return self.shared_ledger.get_identity(did)
        
        return None
    
    def _get_counterparty_dex(self) -> Optional[Dict]:
        """Get counterparty DEX"""
        dex = self.ledger.get_dex(self.counterparty_did)
        if dex:
            return dex
        
        if self.shared_ledger:
            return self.shared_ledger.get_dex(self.counterparty_did)
        
        return None
    
    def _validate_fork_chain(self, identity: Dict) -> tuple[bool, str]:
        """Validate fork lineage chain"""
        fork_history = identity['lineage']['fork_history']
        
        for fork_event in fork_history:
            # Verify fork signature
            parent_did = fork_event['parent_id']
            parent = self._get_identity(parent_did)
            
            if not parent:
                return False, f"Fork parent {parent_did} not found"
            
            # Verify fork event signature
            try:
                parent_pubkey = base58.b58decode(parent['public_key'])
                verify_key = nacl.signing.VerifyKey(parent_pubkey)
                
                canonical = self.initiator._canonicalize(fork_event)
                signature = base58.b58decode(fork_event['signature'])
                
                verify_key.verify(canonical, signature)
            except Exception as e:
                return False, f"Fork signature verification failed: {e}"
        
        return True, "Fork chain valid"
    
    def _generate_uuid(self) -> str:
        import uuid
        return str(uuid.uuid4())
```

---

### Ledger Implementation

#### LocalLedger (SQLite)
```python
"""
Local ledger implementation using SQLite
"""

import sqlite3
import json
from typing import Optional, List, Dict
from pathlib import Path

class LocalLedger:
    """
    Local ledger backed by SQLite
    
    Provides:
    - Identity storage
    - DEX parameters
    - Session records
    - Handshake history
    """
    
    def __init__(self, db_path: str = "~/.aex/ledger.db"):
        self.db_path = Path(db_path).expanduser()
        self.db_path.parent.mkdir(parents=True, exist_ok=True)
        
        self.conn = sqlite3.connect(str(self.db_path))
        self.conn.row_factory = sqlite3.Row
        
        self._init_schema()
    
    def _init_schema(self):
        """Initialize database schema"""
        cursor = self.conn.cursor()
        
        # Identities table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS identities (
                did TEXT PRIMARY KEY,
                identity_json TEXT NOT NULL,
                created_at TEXT NOT NULL,
                updated_at TEXT NOT NULL
            )
        """)
        
        # DEX parameters table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS dex_parameters (
                did TEXT PRIMARY KEY,
                alpha REAL NOT NULL,
                beta REAL NOT NULL,
                dex REAL NOT NULL,
                n_eff REAL NOT NULL,
                last_updated TEXT NOT NULL,
                FOREIGN KEY (did) REFERENCES identities(did)
            )
        """)
        
        # Sessions table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS sessions (
                session_id TEXT PRIMARY KEY,
                handshake_id TEXT NOT NULL,
                agent_a TEXT NOT NULL,
                agent_b TEXT NOT NULL,
                session_json TEXT NOT NULL,
                created_at TEXT NOT NULL,
                published_at TEXT,
                FOREIGN KEY (agent_a) REFERENCES identities(did),
                FOREIGN KEY (agent_b) REFERENCES identities(did)
            )
        """)
        
        # Handshakes table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS handshakes (
                handshake_id TEXT PRIMARY KEY,
                initiator_did TEXT NOT NULL,
                counterparty_did TEXT NOT NULL,
                status TEXT NOT NULL,
                handshake_json TEXT NOT NULL,
                created_at TEXT NOT NULL,
                completed_at TEXT,
                FOREIGN KEY (initiator_did) REFERENCES identities(did),
                FOREIGN KEY (counterparty_did) REFERENCES identities(did)
            )
        """)
        
        # Delegations table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS delegations (
                rep_id TEXT PRIMARY KEY,
                issuer TEXT NOT NULL,
                delegate TEXT NOT NULL,
                rep_json TEXT NOT NULL,
                issued_at TEXT NOT NULL,
                expires_at TEXT NOT NULL,
                revoked_at TEXT,
                FOREIGN KEY (delegate) REFERENCES identities(did)
            )
        """)
        
        # Revocations table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS revocations (
                revocation_id TEXT PRIMARY KEY,
                rep_id TEXT NOT NULL,
                issuer TEXT NOT NULL,
                revocation_json TEXT NOT NULL,
                timestamp TEXT NOT NULL,
                FOREIGN KEY (rep_id) REFERENCES delegations(rep_id)
            )
        """)
        
        # Create indexes
        cursor.execute("CREATE INDEX IF NOT EXISTS idx_sessions_agent_a ON sessions(agent_a)")
        cursor.execute("CREATE INDEX IF NOT EXISTS idx_sessions_agent_b ON sessions(agent_b)")
        cursor.execute("CREATE INDEX IF NOT EXISTS idx_handshakes_initiator ON handshakes(initiator_did)")
        cursor.execute("CREATE INDEX IF NOT EXISTS idx_delegations_delegate ON delegations(delegate)")
        
        self.conn.commit()
    
    def save_identity(self, identity: Dict):
        """Save identity to ledger"""
        cursor = self.conn.cursor()
        
        now = datetime.utcnow().isoformat() + 'Z'
        
        cursor.execute("""
            INSERT OR REPLACE INTO identities (did, identity_json, created_at, updated_at)
            VALUES (?, ?, ?, ?)
        """, (
            identity['aex_id'],
            json.dumps(identity),
            identity.get('created_at', now),
            now
        ))
        
        self.conn.commit()
    
    def get_identity(self, did: str) -> Optional[Dict]:
        """Get identity from ledger"""
        cursor = self.conn.cursor()
        
        cursor.execute("SELECT identity_json FROM identities WHERE did = ?", (did,))
        row = cursor.fetchone()
        
        if row:
            return json.loads(row['identity_json'])
        
        return None
    
    def update_dex(self, did: str, dex: Dict):
        """Update DEX parameters"""
        cursor = self.conn.cursor()
        
        cursor.execute("""
            INSERT OR REPLACE INTO dex_parameters (did, alpha, beta, dex, n_eff, last_updated)
            VALUES (?, ?, ?, ?, ?, ?)
        """, (
            did,
            dex['alpha'],
            dex['beta'],
            dex['dex'],
            dex['n_eff'],
            dex['last_updated']
        ))
        
        self.conn.commit()
    
    def get_dex(self, did: str) -> Optional[Dict]:
        """Get DEX parameters"""
        cursor = self.conn.cursor()
        
        cursor.execute("""
            SELECT alpha, beta, dex, n_eff, last_updated
            FROM dex_parameters
            WHERE did = ?
        """, (did,))
        
        row = cursor.fetchone()
        
        if row:
            return {
                'alpha': row['alpha'],
                'beta': row['beta'],
                'dex': row['dex'],
                'n_eff': row['n_eff'],
                'last_updated': row['last_updated']
            }
        
        return None
    
    def publish_session(self, session: Dict):
        """Publish session to ledger"""
        cursor = self.conn.cursor()
        
        cursor.execute("""
            INSERT INTO sessions (
                session_id, handshake_id, agent_a, agent_b,
                session_json, created_at, published_at
            )
            VALUES (?, ?, ?, ?, ?, ?, ?)
        """, (
            session['session_id'],
            session['handshake_id'],
            session['participants']['agent_a']['aex_id'],
            session['participants']['agent_b']['aex_id'],
            json.dumps(session),
            session['timestamps']['created'],
            session['timestamps']['published']
        ))
        
        self.conn.commit()
    
    def get_agent_sessions(
        self,
        did: str,
        limit: int = 100
    ) -> List[Dict]:
        """Get sessions for agent"""
        cursor = self.conn.cursor()
        
        cursor.execute("""
            SELECT session_json
            FROM sessions
            WHERE agent_a = ? OR agent_b = ?
            ORDER BY published_at DESC
            LIMIT ?
        """, (did, did, limit))
        
        return [json.loads(row['session_json']) for row in cursor.fetchall()]
    
    def save_handshake(self, handshake_id: str, handshake: Dict):
        """Save handshake record"""
        cursor = self.conn.cursor()
        
        cursor.execute("""
            INSERT INTO handshakes (
                handshake_id, initiator_did, counterparty_did,
                status, handshake_json, created_at
            )
            VALUES (?, ?, ?, ?, ?, ?)
        """, (
            handshake_id,
            handshake['initiator_did'],
            handshake['counterparty_did'],
            'pending',
            json.dumps(handshake),
            handshake['timestamp']
        ))
        
        self.conn.commit()
    
    def get_handshake(self, handshake_id: str) -> Optional[Dict]:
        """Get handshake record"""
        cursor = self.conn.cursor()
        
        cursor.execute("""
            SELECT handshake_json
            FROM handshakes
            WHERE handshake_id = ?
        """, (handshake_id,))
        
        row = cursor.fetchone()
        
        if row:
            return json.loads(row['handshake_json'])
        
        return None
    
    def update_handshake(self, handshake_id: str, handshake: Dict):
        """Update handshake record"""
        cursor = self.conn.cursor()
        
        cursor.execute("""
            UPDATE handshakes
            SET status = ?, handshake_json = ?, completed_at = ?
            WHERE handshake_id = ?
        """, (
            handshake.get('status', 'pending'),
            json.dumps(handshake),
            handshake.get('completed_at'),
            handshake_id
        ))
        
        self.conn.commit()
    
    def close(self):
        """Close database connection"""
        self.conn.close()
```
