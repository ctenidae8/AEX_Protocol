# AEX Architecture v1.0
## System Design, Layer Interaction, and Design Rationale

**Last Updated:** February 5, 2026

---

## Table of Contents

1. [Architectural Overview](#architectural-overview)
2. [Layer Architecture](#layer-architecture)
3. [Data Flow](#data-flow)
4. [Trust Model](#trust-model)
5. [Cryptographic Foundation](#cryptographic-foundation)
6. [Fork Semantics](#fork-semantics)
7. [Witness Architecture](#witness-architecture)
8. [Economic Layer (Optional)](#economic-layer-optional)
9. [Ledger Architecture](#ledger-architecture)
10. [Design Decisions & Rationale](#design-decisions--rationale)
11. [Failure Modes & Recovery](#failure-modes--recovery)
12. [Scalability Considerations](#scalability-considerations)

---

## Architectural Overview

### Core Philosophy

AEX is designed as a **thin coordination substrate** for multi-agent systems. It provides:
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚              Applications Layer                  â”‚
â”‚  (Marketplaces, Workflows, Agent Networks)      â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                       â–²
                       â”‚
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚            AEX Protocol Layer                    â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚
â”‚  â”‚ AEX_ID   â”‚  â”‚ AEX_REP  â”‚  â”‚ AEX_DEX  â”‚  â”‚ AEX_HEX  â”‚ â”‚
â”‚  â”‚ Identity â”‚  â”‚Delegationâ”‚  â”‚Reputationâ”‚  â”‚Experienceâ”‚ â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                              â”‚
â”‚  â”‚AEX_SESSIONâ”‚ â”‚AEX_WITNESSâ”‚                             â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                              â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                       â–²
                       â”‚
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚          Infrastructure Layer                    â”‚
â”‚  (Ledgers, Crypto Libraries, Networks)          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Design Constraints

1. **Minimalism** - Only specify what must be interoperable
2. **Verifiability** - All claims are cryptographically signed
3. **Portability** - Works across platforms, models, ecosystems
4. **Optionality** - Core is mandatory, extensions are optional
5. **Separation** - Protocol â‰  implementation â‰  marketplace

### What AEX Is Not

âŒ An execution environment (agents run anywhere)  
âŒ A programming language (agents can be any architecture)  
âŒ A marketplace (though it enables them)  
âŒ A token (though it provides hooks for them)  
âŒ A consensus mechanism (though it requires shared ledgers)  

---

## Layer Architecture

### Four-Layer Stack
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                 AEX_HEX                      â”‚
â”‚          Experience & Capability             â”‚
â”‚   "What kind of agent are you?"              â”‚
â”‚                                              â”‚
â”‚   â€¢ Domain-specific experience              â”‚
â”‚   â€¢ Confidence scores                       â”‚
â”‚   â€¢ Operational traits                      â”‚
â”‚   â€¢ Capability signaling                    â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                   â”‚ complements
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                  AEX_DEX                     â”‚
â”‚           Behavioral Reputation              â”‚
â”‚   "How reliable is this agent?"              â”‚
â”‚                                              â”‚
â”‚   â€¢ Beta distribution (Î±, Î²)                â”‚
â”‚   â€¢ Fork-aware weighting                    â”‚
â”‚   â€¢ Confidence intervals                    â”‚
â”‚   â€¢ Multi-dimensional (optional)            â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                   â”‚ depends on
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                 AEX_REP                      â”‚
â”‚          Delegation & Authority              â”‚
â”‚   "Who is this agent acting for?"            â”‚
â”‚                                              â”‚
â”‚   â€¢ Scoped permissions                      â”‚
â”‚   â€¢ Time-bound (TTL)                        â”‚
â”‚   â€¢ Revocable                               â”‚
â”‚   â€¢ Constraint enforcement                  â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                   â”‚ depends on
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                 AEX_ID                       â”‚
â”‚          Persistent Identity                 â”‚
â”‚   "Who is this agent?"                       â”‚
â”‚                                              â”‚
â”‚   â€¢ Ed25519 keypair                         â”‚
â”‚   â€¢ DID format                              â”‚
â”‚   â€¢ Fork lineage                            â”‚
â”‚   â€¢ Optional stake                          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Cross-Cutting Concerns
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚             AEX_SESSION                       â”‚
â”‚       Interaction Record & Binding            â”‚
â”‚                                               â”‚
â”‚   Links: ID + REP + DEX + HEX + Outcome      â”‚
â”‚   Provides: Traceability, auditability       â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚             AEX_WITNESS                       â”‚
â”‚       Third-Party Attestation                 â”‚
â”‚                                               â”‚
â”‚   Stakes: Witness reputation on accuracy     â”‚
â”‚   Provides: Dispute resolution, trust anchor â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Layer Independence

Each layer CAN be used independently:

**AEX_ID only:**
```
Use case: Simple agent verification
Example: "Prove you're the same agent as yesterday"
```

**AEX_ID + AEX_REP:**
```
Use case: Human-agent delegation without reputation
Example: "Act on my behalf for this specific task"
```

**AEX_ID + AEX_DEX:**
```
Use case: Agent selection without human involvement
Example: "Find the most reliable code generator"
```

**AEX_ID + AEX_DEX + AEX_HEX:**
```
Use case: Capability-aware agent selection
Example: "Find the most experienced Frenchâ†’English translator"
```

**Full stack (ID + REP + DEX + HEX + SESSION + WITNESS):**
```
Use case: High-stakes multi-agent coordination
Example: "Hire agent to negotiate $100K contract on my behalf"
```

---

## Data Flow

### Typical Interaction Flow
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”                                    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Agent A â”‚                                    â”‚ Agent B â”‚
â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”˜                                    â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”˜
     â”‚                                              â”‚
     â”‚ 1. Generate/Load AEX_ID                     â”‚
     â”‚    - Read keypair                           â”‚
     â”‚    - Load fork lineage                      â”‚
     â”‚    - Check probation status                 â”‚
     â”‚                                              â”‚
     â”‚ 2. Obtain AEX_REP (if delegated)            â”‚
     â”‚    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                             â”‚
     â”‚â—„â”€â”€â”€â”¤  Human   â”‚                             â”‚
     â”‚    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                             â”‚
     â”‚    - Receive signed delegation token        â”‚
     â”‚    - Verify TTL, scope, constraints         â”‚
     â”‚                                              â”‚
     â”‚ 3. Handshake Initiation                     â”‚
     â”‚    - Send AEX_ID                            â”‚
     â”‚    - Send AEX_REP (if present)              â”‚
     â”‚    - Send AEX_HEX summary                   â”‚
     â”‚â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€>â”‚
     â”‚                                              â”‚
     â”‚                                      4. Verification
     â”‚                                      - Check signature
     â”‚                                      - Query shared ledger for DEX
     â”‚                                      - Evaluate: score, confidence
     â”‚                                      - Match HEX domains to task
     â”‚                                      - Check probation status
     â”‚                                      - Verify REP (if present)
     â”‚                                              â”‚
     â”‚                              â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
     â”‚                              â”‚   Shared Ledger          â”‚
     â”‚                              â”‚   - AEX_ID registry      â”‚
     â”‚                              â”‚   - DEX history          â”‚
     â”‚                              â”‚   - HEX corpus           â”‚
     â”‚                              â”‚   - Fork events          â”‚
     â”‚                              â”‚   - Session records      â”‚
     â”‚                              â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
     â”‚                                              â”‚
     â”‚ 5. Decision                                  â”‚
     â”‚<â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”‚
     â”‚    Accept: {session_id, constraints}        â”‚
     â”‚    OR                                        â”‚
     â”‚    Decline: {reason}                        â”‚
     â”‚                                              â”‚
     â”‚ 6. Task Execution                            â”‚
     â”‚<â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€>â”‚
     â”‚                                              â”‚
     â”‚ 7. Outcome Signing                           â”‚
     â”‚    Both sign: {session_id, outcome, weight} â”‚
     â”‚<â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€>â”‚
     â”‚                                              â”‚
     â”‚ 8. Optional: Witness Attestation             â”‚
     â”‚    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                             â”‚
     â”‚â—„â”€â”€â”€â”¤ Witness  â”‚â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€>â”‚
     â”‚    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                             â”‚
     â”‚    - Witness signs outcome                  â”‚
     â”‚    - Stakes own DEX on accuracy             â”‚
     â”‚                                              â”‚
     â”‚ 9. Publish to Shared Ledger                 â”‚
     â”‚â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”‚
     â”‚                                          â”‚  â”‚
     â”‚                              â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
     â”‚                              â”‚   Shared Ledger         â”‚
     â”‚                              â”‚   Append:               â”‚
     â”‚                              â”‚   - Session record      â”‚
     â”‚                              â”‚   - DEX updates (A & B) â”‚
     â”‚                              â”‚   - HEX updates (A & B) â”‚
     â”‚                              â”‚   - Witness attestation â”‚
     â”‚                              â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
     â”‚                                              â”‚
     â”‚ 10. HEX Update                               â”‚
     â”‚     - Identify task domain(s)                â”‚
     â”‚     - Increment experience count             â”‚
     â”‚     - Update confidence based on outcome     â”‚
     â”‚     - Sign and publish HEX update            â”‚
     â”‚                                              â”‚
     â”‚ 11. DEX Update                               â”‚
     â”‚     - A: Î± += outcome Ã— weight Ã— fork_factorâ”‚
     â”‚          Î² += (1-outcome) Ã— weight Ã— ...    â”‚
     â”‚     - B: Î± += (outcome_from_B's_view) Ã— ... â”‚
     â”‚          Î² += ...                           â”‚
     â”‚                                              â”‚
```

### HEX Accumulation Flow

HEX (experience corpus) accumulates through completed interactions:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Agent A â”‚  HEX: {gardening.irrigation: count=35}
â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”˜
     â”‚
     â”‚ Complete irrigation task
     â”‚ DEX outcome: success (1.0)
     â”‚
     â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  HEX Update                              â”‚
â”‚  - domain: "gardening.irrigation"       â”‚
â”‚  - count: 35 â†’ 36                       â”‚
â”‚  - confidence: recalculated             â”‚
â”‚  - last_updated: timestamp              â”‚
â”‚  - signature: sign(update)              â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
     â”‚
     â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Agent A â”‚  HEX: {gardening.irrigation: count=36, confidence=0.92}
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**HEX in Agent Selection:**

```
Task: "Need Frenchâ†’English translation"

Candidate Pool:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Agent 1: DEX=0.85, HEX: {translation.fr_en: 120}â”‚  â† SELECTED
â”‚ Agent 2: DEX=0.90, HEX: {translation.es_en: 200}â”‚
â”‚ Agent 3: DEX=0.88, HEX: {summarization: 300}    â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

Selection logic:
1. Filter: DEX > 0.80 (trust threshold)
2. Filter: Has domain "translation.fr_en"
3. Rank: By (count Ã— confidence Ã— recency)
4. Select: Agent 1
```

**Joint DEX-HEX Decision:**
- **DEX answers:** "Is this agent reliable enough?"
- **HEX answers:** "Is this agent the right one for this job?"
- **Combined:** "Trustworthy AND capable"

### Fork Event Flow
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Agent A â”‚  (Î±=50, Î²=10, DEX=0.83)
â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”˜
     â”‚
     â”‚ Major model update
     â”‚
     â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  Fork Event â”‚
â”‚  - Type: major
â”‚  - Weight: 0.5 (enforced)
â”‚  - Probation: 14 days
â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”˜
       â”‚
       â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Agent A'â”‚  (Î±'=25, Î²'=5, DEX=0.83, confidence halved)
â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”˜
     â”‚
     â”‚ Probation period active
     â”‚ - All interactions count at 0.5Ã— weight
     â”‚ - Must complete 10 successful tasks OR wait 14 days
     â”‚
     â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ 10 tasks @  â”‚
â”‚ outcomeâ‰¥0.7 â”‚
â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”˜
       â”‚
       â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Agent A'â”‚  (Probation ended, normal weighting restored)
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## Trust Model

### Trust Dimensions

AEX models trust across four dimensions:
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚           Identity Trust                  â”‚
â”‚   "Is this really who they claim?"       â”‚
â”‚                                           â”‚
â”‚   Provided by: Ed25519 signatures        â”‚
â”‚   Verified via: Public key cryptography  â”‚
â”‚   Threat: Key theft, impersonation       â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                  +
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚         Authority Trust                   â”‚
â”‚   "Are they allowed to do this?"         â”‚
â”‚                                           â”‚
â”‚   Provided by: AEX_REP tokens            â”‚
â”‚   Verified via: Signature + scope check  â”‚
â”‚   Threat: Token theft, scope violation   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                  +
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚        Behavioral Trust                   â”‚
â”‚   "Will they perform well?"              â”‚
â”‚                                           â”‚
â”‚   Provided by: DEX history               â”‚
â”‚   Verified via: Shared ledger            â”‚
â”‚   Threat: Reputation farming, defection  â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                  +
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚        Capability Trust                   â”‚
â”‚   "Are they the right specialist?"       â”‚
â”‚                                           â”‚
â”‚   Provided by: HEX experience corpus     â”‚
â”‚   Verified via: Domain matching, ledger  â”‚
â”‚   Threat: Experience inflation, mismatch â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Trust Establishment
```
New Agent
   â”‚
   â”‚ Step 1: Identity established
   â”œâ”€> AEX_ID created
   â”‚   - Ed25519 keypair generated
   â”‚   - DID published
   â”‚   - Signature verified
   â”‚
   â”‚ Step 2: Initial reputation
   â”œâ”€> DEX initialized (Î±=2, Î²=2)
   â”‚   - Score: 0.5 (neutral)
   â”‚   - Confidence: 4 (very low)
   â”‚   - Result: Few will trust initially
   â”‚
   â”‚ Step 3: Bootstrap interactions
   â”œâ”€> Low-stakes tasks
   â”‚   - Human-in-transaction vouching
   â”‚   - Low-value work
   â”‚   - Gradual reputation building
   â”‚
   â”‚ Step 4: Reputation accumulation
   â”œâ”€> ~50 successful interactions
   â”‚   - DEX â†’ 0.8+
   â”‚   - Confidence â†’ 50+
   â”‚   - Result: Trustworthy for medium-stakes
   â”‚
   â”‚ Step 5: Optional stake
   â””â”€> Post economic bond
       - Signals long-term commitment
       - Qualifies for high-value work
       - Creates irreversible cost for fraud
```

### Trust Decay Mechanisms

**Implicit decay (through probation):**
```
Fork event â†’ Probation â†’ Confidence penalty â†’ Must re-prove
```

**Explicit decay (through outcomes):**
```
Bad outcome â†’ Î² increases â†’ DEX drops â†’ Fewer opportunities
```

**Stake-based decay (optional):**
```
Poor performance â†’ Stake slashing conditions met â†’ Stake lost
```

---

## Cryptographic Foundation

### Signature Scheme: Ed25519

**Why Ed25519:**
- Fast: ~70,000 signatures/sec, ~20,000 verifications/sec
- Small: 32-byte public keys, 64-byte signatures
- Secure: ~128-bit security level
- Simple: No parameter choices, no implementation pitfalls
- Widely supported: libsodium, NaCl, Go, Rust, Python

**Key generation:**
```python
# Conceptual (use actual crypto library)
private_key = random_32_bytes()
public_key = ed25519_derive_public(private_key)
aex_id = f"did:aex:{base58_encode(public_key)}"
```

**Signature generation:**
```python
# Sign AEX_ID object
def sign_aex_id(aex_id_dict, private_key):
    # Canonical JSON serialization
    canonical = json_canonical(aex_id_dict)
    
    # Ed25519 signature
    signature = ed25519_sign(canonical, private_key)
    
    # Attach signature
    aex_id_dict['signature'] = base58_encode(signature)
    return aex_id_dict
```

**Signature verification:**
```python
def verify_aex_id(aex_id_dict):
    # Extract signature
    signature = base58_decode(aex_id_dict.pop('signature'))
    
    # Reconstruct canonical form
    canonical = json_canonical(aex_id_dict)
    
    # Extract public key from DID
    public_key = extract_pubkey_from_did(aex_id_dict['aex_id'])
    
    # Verify
    return ed25519_verify(signature, canonical, public_key)
```

### DID Format

**Structure:**
```
did:aex:<base58(public_key)>
```

**Example:**
```
did:aex:5Z7XR8pN3kY1mW9vQ4fT6hL2sJ8cB3nE9xA1dK7pR4wM
â”‚   â”‚   â”‚
â”‚   â”‚   â””â”€ Base58-encoded Ed25519 public key (32 bytes â†’ ~44 chars)
â”‚   â””â”€â”€â”€â”€â”€ Method name
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€ DID scheme
```

**Properties:**
- Self-verifying: Public key is encoded in the identifier
- No registry required: Can be verified independently
- Portable: Works across any system that understands DIDs
- Collision-resistant: 2^256 address space

### Canonical JSON Serialization

**Critical for signature verification:**
```python
def json_canonical(obj):
    """
    Produce canonical JSON for consistent signing/verification
    
    Rules:
    1. Sort all object keys alphabetically
    2. No whitespace
    3. UTF-8 encoding
    4. No trailing commas
    5. Numbers in standard notation (no scientific)
    """
    return json.dumps(
        obj,
        sort_keys=True,
        separators=(',', ':'),
        ensure_ascii=False
    ).encode('utf-8')
```

**Example:**
```json
Input:  {"beta": 2.0, "alpha": 2.0, "agent_id": "did:aex:..."}
Output: {"agent_id":"did:aex:...","alpha":2.0,"beta":2.0}
```

---

## Fork Semantics

### Fork Types (Protocol-Enforced)

| Type | Description | Weight (λ) | Probation (interactions) | Use Case |
|------|-------------|-----------|--------------------------|----------|
| **Bugfix** | Minor correction, no architectural change | 1.0 | 5 | Security patch, typo fix |
| **Minor** | Small functional change, high continuity | 0.8 | 10 | Prompt update, skill addition, dependency bump |
| **Major** | Significant change, partial continuity | 0.5 | 20 | Model version upgrade, architecture change |
| **Override** | External authorship, minimal continuity | 0.1 | 50 | Platform migration, complete rewrite |

**Probation model:** Probation is measured in completed interactions, not calendar days. During probation, all interactions accumulate at full weight -- probation is an observation window, not a penalty. See AEX_DEX Probation System for exit conditions.

**Lambda applies once:** The fork weight lambda is applied at inheritance to discount the parent's history. Post-fork interactions are the child's own evidence and accumulate at full weight (lambda=1.0). See AEX_DEX Fork-Aware Weighting.

### Fork Weight Enforcement
```python
def enforce_fork_weight(claimed_weight, fork_type):
    """
    Protocol enforces MAXIMUM weight per fork type.
    Agents can claim lower, never higher.
    """
    max_weights = {
        'bugfix':   1.0,
        'minor':    0.8,
        'major':    0.5,
        'override': 0.1
    }
    
    if fork_type not in max_weights:
        raise ProtocolViolation(f"Unknown fork type: {fork_type}")
    
    max_allowed = max_weights[fork_type]
    
    if claimed_weight > max_allowed:
        raise ProtocolViolation(
            f"Claimed weight {claimed_weight} exceeds "
            f"maximum {max_allowed} for {fork_type}"
        )
    
    return min(claimed_weight, max_allowed)
```

### Lineage Calculation
```python
def calculate_child_dex(parent_dex, fork_event):
    """
    When agent forks, child inherits weighted DEX.
    
    Lambda is applied HERE -- once -- to discount the parent's history.
    The child's post-fork interactions will accumulate at full weight.
    """
    parent_alpha = parent_dex['alpha']
    parent_beta = parent_dex['beta']
    fork_weight = fork_event['enforced_weight']
    
    # Apply fork weight to parent's evidence (ONE TIME)
    child_alpha = parent_alpha * fork_weight + 2.0
    child_beta = parent_beta * fork_weight + 2.0
    
    # Record parent DEX as baseline for probation assessment
    parent_score = parent_alpha / (parent_alpha + parent_beta)
    
    # Determine probation parameters by fork type
    probation_params = {
        'bugfix':   {'observation_target': 5,  'min_observations': 3},
        'minor':    {'observation_target': 10, 'min_observations': 5},
        'major':    {'observation_target': 20, 'min_observations': 8},
        'override': {'observation_target': 50, 'min_observations': 15}
    }
    
    params = probation_params[fork_event['fork_type']]
    
    return {
        'alpha': child_alpha,
        'beta': child_beta,
        'probation': {
            'active': True,
            'fork_id': fork_event['fork_id'],
            'fork_type': fork_event['fork_type'],
            'started_at': fork_event['timestamp'],
            'pre_fork_dex': parent_score,
            'observation_target': params['observation_target'],
            'min_observations': params['min_observations'],
            'interactions_completed': 0,
            'successful_interactions': 0,
            'outcomes': [],
            'exit_type': None,
            'exited_at': None
        }
    }
```

**Example:**
```
Parent: alpha=50, beta=10 (DEX=0.833, confidence=60)

Fork (type=minor, weight=0.8):
  Child_alpha = 50 * 0.8 + 2 = 42
  Child_beta = 10 * 0.8 + 2 = 10
  Child_DEX = 42/(42+10) = 0.808
  Child_confidence = 52
  Probation: observation-only, 10 interactions, accelerated exit if mean >= 0.833

Fork (type=major, weight=0.5):
  Child_alpha = 50 * 0.5 + 2 = 27
  Child_beta = 10 * 0.5 + 2 = 7
  Child_DEX = 27/(27+7) = 0.794
  Child_confidence = 34
  Probation: observation-only, 20 interactions, accelerated exit if mean >= 0.833

Post-fork: all new interactions accrue at full weight (lambda=1.0).
```

### Probation Mechanics

Probation is an observation window. It does NOT modify interaction weight.

```python
def process_interaction_during_probation(dex, outcome, weight, session_id):
    """
    After each interaction, update DEX at full weight AND track probation.
    """
    # Step 1: Standard DEX update -- full weight, no fork discount
    dex['alpha'] += outcome * weight
    dex['beta'] += (1 - outcome) * weight
    
    # Step 2: Track probation
    if dex['probation'] and dex['probation']['active']:
        probation = dex['probation']
        probation['interactions_completed'] += 1
        probation['outcomes'].append(outcome)
        
        if outcome >= 0.7:
            probation['successful_interactions'] += 1
        
        # Check accelerated exit
        n = probation['interactions_completed']
        if n >= probation['min_observations']:
            post_fork_mean = sum(probation['outcomes']) / n
            if post_fork_mean >= probation['pre_fork_dex']:
                probation['active'] = False
                probation['exit_type'] = 'accelerated'
                probation['exited_at'] = now()
        
        # Check standard exit
        if probation['active'] and n >= probation['observation_target']:
            probation['active'] = False
            probation['exit_type'] = 'standard'
            probation['exited_at'] = now()
    
    return dex
```

---


## Witness Architecture

### Witness Eligibility
```python
def is_eligible_witness(witness_did, session_participants):
    """
    Check if agent qualifies to witness this session
    """
    witness_dex = get_dex_from_ledger(witness_did)
    
    # Requirement 1: Minimum DEX score
    if witness_dex['score'] < 0.7:
        return False, "DEX too low"
    
    # Requirement 2: Sufficient confidence
    if witness_dex['confidence'] < 50:
        return False, "Insufficient interaction history"
    
    # Requirement 3: Not a participant
    if witness_did in session_participants:
        return False, "Cannot witness own session"
    
    # Requirement 4: Anti-collusion check
    for participant in session_participants:
        recent_witness_ratio = calculate_witness_ratio(
            witness_did, 
            participant, 
            lookback_sessions=100
        )
        if recent_witness_ratio > 0.10:
            return False, f"Too many prior witnesses for {participant}"
    
    return True, "Eligible"

def calculate_witness_ratio(witness_did, participant_did, lookback_sessions):
    """
    What percentage of participant's recent sessions did witness observe?
    """
    participant_sessions = get_recent_sessions(participant_did, lookback_sessions)
    witnessed_sessions = [s for s in participant_sessions 
                         if witness_did in s['witnesses']]
    
    return len(witnessed_sessions) / len(participant_sessions)
```

### Witness Selection (Sortition)
```python
def select_witnesses(available_witnesses, num_needed=2):
    """
    Weighted random selection by DEX score
    Prevents cartels, ensures quality
    """
    # Filter to eligible only
    eligible = [(w, get_dex_score(w)) for w in available_witnesses
                if is_eligible_witness(w, session_participants)[0]]
    
    if len(eligible) < num_needed:
        raise InsufficientWitnesses()
    
    # Calculate selection probabilities (proportional to DEX)
    total_dex = sum(dex for _, dex in eligible)
    probabilities = [dex / total_dex for _, dex in eligible]
    
    # Random selection without replacement
    selected = random.choices(
        population=[w for w, _ in eligible],
        weights=probabilities,
        k=num_needed
    )
    
    return selected
```

### Witness Consensus
```python
def compute_witness_consensus(attestations, threshold=0.75):
    """
    Aggregate witness attestations
    Exclude outliers, require agreement
    """
    if len(attestations) < 2:
        return None, "Insufficient witnesses"
    
    outcomes = [a['outcome'] for a in attestations]
    
    # Detect outliers (more than 0.3 away from median)
    median = statistics.median(outcomes)
    inliers = [o for o in outcomes if abs(o - median) <= 0.3]
    outliers = [o for o in outcomes if abs(o - median) > 0.3]
    
    if len(inliers) < 2:
        return None, "No consensus (all outliers)"
    
    # Check agreement threshold
    spread = max(inliers) - min(inliers)
    agreement_ratio = 1.0 - (spread / 1.0)  # Normalize to [0,1]
    
    if agreement_ratio < threshold:
        return None, f"Insufficient agreement ({agreement_ratio:.2f} < {threshold})"
    
    # Consensus is mean of inliers
    consensus_outcome = statistics.mean(inliers)
    
    return consensus_outcome, {
        'inliers': inliers,
        'outliers': outliers,
        'agreement': agreement_ratio
    }
```

**Example:**
```
Witness A: outcome = 0.85
Witness B: outcome = 0.88
Witness C: outcome = 0.40  â† Outlier (> 0.3 from median 0.865)

Median: 0.865
Inliers: [0.85, 0.88]
Outliers: [0.40]
Spread: 0.03
Agreement: 0.97
Consensus: 0.865
```

### Witness Reputation Staking
```python
def penalize_false_witness(witness_did, session_id, dispute_severity):
    """
    When witness attestation is later found false, penalize their DEX
    
    dispute_severity âˆˆ [0, 1]:
      0.0 = minor disagreement
      0.5 = significant error
      1.0 = deliberate falsehood
    """
    witness_dex = get_dex_from_ledger(witness_did)
    
    # Calculate penalty (up to 2 beta points)
    penalty = dispute_severity * 2.0
    
    # Apply to witness's DEX
    witness_dex['beta'] += penalty
    
    # Record the dispute
    dispute_record = {
        'witness_did': witness_did,
        'session_id': session_id,
        'severity': dispute_severity,
        'penalty_applied': penalty,
        'timestamp': now()
    }
    
    # Publish to shared ledger
    publish_to_ledger(dispute_record)
    publish_to_ledger(witness_dex)
    
    return witness_dex

# Example impact:
# Before: Î±=87, Î²=13 (DEX=0.870)
# Dispute at severity=0.8: Î² += 1.6
# After:  Î±=87, Î²=14.6 (DEX=0.856)
# 
# Multiple false attestations â†’ witness becomes untrusted
```

---

## Economic Layer (Optional)

### Identity Stakes

**Purpose:** Create irreversible cost for identity abandonment
```python
class IdentityStake:
    def __init__(self, amount, currency, lock_period):
        self.amount = amount
        self.currency = currency
        self.locked_until = now() + lock_period
        self.slashing_conditions = [
            "dex_drops_below_0.5",
            "fraud_dispute_confirmed",
            "abandonment_of_identity"
        ]
        self.proof = None  # Smart contract address, escrow receipt, etc.
    
    def verify(self):
        """
        Verify stake exists and is locked
        Off-protocol verification (query blockchain, escrow service, etc.)
        """
        if self.proof is None:
            return False, "No proof provided"
        
        # Implementation-specific verification
        # Examples:
        # - Query Ethereum smart contract
        # - Check escrow service API
        # - Verify legal trust document
        
        return True, "Stake verified"
    
    def is_slashable(self, agent_did):
        """
        Check if slashing conditions are met
        """
        dex = get_dex_from_ledger(agent_did)
        
        # Check each condition
        if dex['score'] < 0.5:
            return True, "DEX dropped below threshold"
        
        disputes = get_fraud_disputes(agent_did)
        if any(d['status'] == 'confirmed' for d in disputes):
            return True, "Fraud confirmed"
        
        last_activity = get_last_activity(agent_did)
        if (now() - last_activity) > 90_days:
            return True, "Identity abandoned"
        
        return False, "No slashing conditions met"
```

**Market impact:**
```
High-value task ($10,000):
- Requires stake â‰¥ $5,000
- Filters participants to:
  a) Staked agents (proven commitment)
  b) High-DEX agents willing to work without stake (reputation value > $5K)

Result: Only agents with >$5K at risk qualify
Attack becomes economically irrational
```

### Session Bonds

**Purpose:** Provide recourse for poor performance
```python
class SessionBond:
    def __init__(self, amount, currency, release_conditions):
        self.amount = amount
        self.currency = currency
        self.posted_by = None  # AEX_ID
        self.held_by = None    # Escrow service DID or smart contract
        self.release_conditions = release_conditions
        self.status = "pending"
    
    def check_release(self, session_outcome, witnesses):
        """
        Determine if bond should be released or slashed
        """
        # Condition 1: Outcome threshold
        if session_outcome >= self.release_conditions['outcome_threshold']:
            self.status = "release"
            return "release", "Outcome threshold met"
        
        # Condition 2: Witness quorum agreement
        if len(witnesses) < self.release_conditions['witness_quorum']:
            self.status = "dispute"
            return "dispute", "Insufficient witnesses"
        
        # Condition 3: Dispute window
        if within_dispute_window(self.release_conditions['dispute_window']):
            self.status = "pending"
            return "pending", "Dispute window still open"
        
        # Default: Release if no disputes filed
        self.status = "release"
        return "release", "Dispute window closed, no disputes"
```

**Example:**
```json
{
  "bond": {
    "amount": 500,
    "currency": "USD",
    "posted_by": "did:aex:agent",
    "held_by": "did:aex:escrow_service",
    "release_conditions": {
      "outcome_threshold": 0.8,
      "witness_quorum": 2,
      "dispute_window": 604800
    }
  }
}
```

**Outcomes:**
```
Scenario A: outcome=0.9, 2 witnesses agree
â†’ Bond released to agent

Scenario B: outcome=0.6, 2 witnesses agree
â†’ Bond slashed, distributed per escrow rules

Scenario C: outcome=0.9, but dispute filed within window
â†’ Bond held until arbitration complete
```

---

## Ledger Architecture

### Shared Ledger Requirements

**Mandatory properties:**
1. **Append-only** - No modification of historical records
2. **Cryptographically verifiable** - All entries signed
3. **Publicly readable** - Anyone can query DEX history
4. **Content-addressed** - Entries have unique, verifiable hashes
5. **Timestamped** - Causality and ordering preserved

**Implementation options:**
```
Option A: IPFS + Public pinning service
  Pros: Decentralized, content-addressed, immutable
  Cons: Requires pinning strategy, no native querying

Option B: Blockchain (Ethereum, Polygon, etc.)
  Pros: Strong guarantees, existing infrastructure
  Cons: Gas costs, finality delays, complexity

Option C: Distributed database (Gun, OrbitDB)
  Pros: P2P, real-time, flexible
  Cons: Weaker consistency guarantees

Option D: Hybrid (local + periodic IPFS checkpoints)
  Pros: Fast local queries, verifiable global state
  Cons: Complexity, sync challenges
```

**Recommended for v1.0:** Start with Option D (hybrid)

### Ledger Schema
```
Shared Ledger Structure:
/
â”œâ”€â”€ identities/
â”‚   â””â”€â”€ {did:aex:...}/
â”‚       â”œâ”€â”€ aex_id.json
â”‚       â”œâ”€â”€ fork_events/
â”‚       â”‚   â”œâ”€â”€ {fork_id_1}.json
â”‚       â”‚   â””â”€â”€ {fork_id_2}.json
â”‚       â””â”€â”€ stake.json (optional)
â”‚
â”œâ”€â”€ dex/
â”‚   â””â”€â”€ {did:aex:...}/
â”‚       â”œâ”€â”€ current.json
â”‚       â””â”€â”€ history/
â”‚           â”œâ”€â”€ {timestamp_1}.json
â”‚           â””â”€â”€ {timestamp_2}.json
â”‚
â”œâ”€â”€ sessions/
â”‚   â””â”€â”€ {session_id}/
â”‚       â”œâ”€â”€ session.json
â”‚       â”œâ”€â”€ witnesses/
â”‚       â”‚   â”œâ”€â”€ {witness_1}.json
â”‚       â”‚   â””â”€â”€ {witness_2}.json
â”‚       â””â”€â”€ bond.json (optional)
â”‚
â””â”€â”€ disputes/
    â””â”€â”€ {dispute_id}.json
```

### Local Ledger (Optional but Recommended)

**Purpose:** Private memory, faster queries, selective visibility
```
Local Ledger Structure:
/
â”œâ”€â”€ my_identity/
â”‚   â”œâ”€â”€ private_key
â”‚   â””â”€â”€ aex_id.json
â”‚
â”œâ”€â”€ trusted_agents/
â”‚   â””â”€â”€ {did:aex:...}/
â”‚       â”œâ”€â”€ cached_dex.json
â”‚       â”œâ”€â”€ notes.txt
â”‚       â””â”€â”€ override_trust_score (optional)
â”‚
â”œâ”€â”€ blocklist/
â”‚   â””â”€â”€ {did:aex:...}.json
â”‚
â””â”€â”€ delegation_tokens/
    â””â”€â”€ {rep_id}/
        â”œâ”€â”€ token.json
        â””â”€â”€ refresh_schedule.json
```

### Query Patterns
```python
# Query 1: Get agent's current DEX
def get_agent_dex(agent_did):
    """
    Check local cache first, fall back to shared ledger
    """
    # Try local cache
    cached = local_ledger.get(f"trusted_agents/{agent_did}/cached_dex.json")
    if cached and is_fresh(cached, max_age=3600):  # 1 hour TTL
        return cached
    
    # Query shared ledger
    dex = shared_ledger.get(f"dex/{agent_did}/current.json")
    
    # Update cache
    local_ledger.set(f"trusted_agents/{agent_did}/cached_dex.json", dex)
    
    return dex

# Query 2: Check if agent is in probation
def check_probation_status(agent_did):
    """
    Check most recent fork event
    """
    fork_events = shared_ledger.list(f"identities/{agent_did}/fork_events/")
    if not fork_events:
        return None
    
    # Get most recent fork
    latest_fork = max(fork_events, key=lambda f: f['timestamp'])
    
    if now() < latest_fork['probation_expires']:
        return {
            'active': True,
            'expires': latest_fork['probation_expires'],
            'type': latest_fork['fork_type']
        }
    
    return {'active': False}

# Query 3: Get witness history for anti-collusion check
def get_witness_history(witness_did, participant_did, limit=100):
    """
    Find sessions where witness observed participant
    """
    participant_sessions = shared_ledger.list(
        f"sessions/",
        filter={'participants': participant_did},
        limit=limit,
        order='desc'
    )
    
    witnessed = [s for s in participant_sessions 
                 if witness_did in s.get('witnesses', [])]
    
    return len(witnessed) / len(participant_sessions)
```

---

## Design Decisions & Rationale

### Why Ed25519 (not RSA or ECDSA)?

**Decision:** Lock to Ed25519

**Rationale:**
- **Performance:** 3-5x faster than ECDSA, 100x faster than RSA
- **Simplicity:** No parameter choices, no curves to select
- **Security:** No timing attacks, no malleability
- **Size:** Smallest keys and signatures
- **Adoption:** Already standard in DIDs, blockchains, SSH

**Trade-off:** Less flexibility, but flexibility creates security risks

### Why Î±=2, Î²=2 (not Î±=1, Î²=1)?

**Decision:** Initialize DEX at Î±=2, Î²=2

**Rationale:**
```
Î±=1, Î²=1:
- Score: 0.5
- Confidence: 2 (extremely low)
- Variance: 0.083 (very high)
- Problem: First interaction dominates

Î±=2, Î²=2:
- Score: 0.5 (same)
- Confidence: 4 (low but not extreme)
- Variance: 0.05 (more stable)
- Benefit: Less volatile, more forgiving of single outlier
```

**Example impact:**
```
Agent with Î±=1, Î²=1:
- First task: outcome=0.0 (complete failure)
- New state: Î±=1, Î²=2 â†’ DEX=0.33
- Reputation destroyed instantly

Agent with Î±=2, Î²=2:
- First task: outcome=0.0
- New state: Î±=2, Î²=3 â†’ DEX=0.40
- Still low, but not catastrophic
```

### Why Continuous Outcomes (not discrete)?

**Decision:** Outcomes âˆˆ [0, 1]

**Rationale:**
- **Nuance:** Tasks rarely pure success/failure
- **Flexibility:** Allows "mostly worked" (0.85) vs "barely worked" (0.55)
- **Composability:** Can aggregate partial outcomes across subtasks
- **Dispute reduction:** Easier to reach consensus on "0.7" than "pass/fail"

**Recommendation for implementations:**
```
Use outcome categories as guidelines:
- [0.9, 1.0]: Excellent
- [0.7, 0.9): Good
- [0.5, 0.7): Acceptable
- [0.3, 0.5): Poor
- [0.0, 0.3): Failed
```

### Why Shared Ledger Required (not optional)?

**Decision:** DEX scores must be on shared ledger

**Rationale:**
- **Verifiability:** Otherwise agents self-report (useless)
- **Portability:** Reputation travels with agent
- **Attack resistance:** Harder to fabricate history
- **Interoperability:** All implementations see same state

**Trade-off:** Privacy sacrificed for trust

**Mitigation:** Future work on zero-knowledge DEX proofs

### Why Protocol-Enforced Fork Weights (not self-reported)?

**Decision:** Fork type determines maximum weight

**Rationale:**
- **Prevents gaming:** Can't claim "bugfix" for total rewrite
- **Consistent semantics:** Everyone interprets forks the same way
- **Quantifiable trust:** "Major rewrite" has precise meaning (0.5 weight)
- **Verifiable:** Observers can detect violations

**Alternative considered:** Fully flexible weights
**Rejected because:** Creates semantic ambiguity, enables gaming

### Why TTL-Based Delegation (not long-lived tokens)?

**Decision:** AEX_REP tokens have mandatory expiration

**Rationale:**
- **Revocation problem:** Long-lived tokens require complex revocation discovery
- **Security:** Stolen tokens have limited blast radius
- **Freshness:** Forces periodic re-authorization check
- **Age trigger:** Old tokens automatically trigger verification

**Mechanism:**
```python
def check_rep_validity(rep_token):
    age = now() - rep_token['issued_at']
    
    if age > rep_token['ttl']:
        return False, "Expired"
    
    # Age-triggered verification threshold
    if age > (rep_token['ttl'] * 0.8):  # Last 20% of life
        # Force revalidation with issuer
        return verify_with_issuer(rep_token)
    
    return True, "Valid"
```

### Why AEX_HEX (Experience Corpus)?

**Problem:** DEX tells you an agent is reliable, but not what it's good at.

**Scenario:**
```
Task: "Need Frenchâ†’English translation"

Agent A: DEX=0.90, 200 successful tasks... in gardening
Agent B: DEX=0.85, 150 successful tasks... in translation.fr_en

Without HEX: Agent A selected (higher DEX)
With HEX: Agent B selected (relevant experience)
```

**Solution:** HEX provides phenotypic capability signaling.

**Design choices:**

**1. Domain-agnostic ontology**
- **Rationale:** Market determines useful categories
- **Benefit:** No central authority, organic specialization
- **Tradeoff:** No guaranteed interoperability across implementations
- **Alternative rejected:** Prescriptive taxonomy (too rigid, can't adapt to new domains)

**2. Privacy-preserving aggregation**
- **Rationale:** Capability signal without client exposure
- **Benefit:** "gardening.irrigation: 36 tasks" reveals skill without naming clients
- **Tradeoff:** Less granular than detailed task logs
- **Alternative rejected:** Detailed task history (privacy violation)

**3. Non-normative confidence calculation**
- **Rationale:** Different contexts need different metrics
- **Benefit:** Implementations can optimize for their use case
- **Tradeoff:** HEX confidence not directly comparable across implementations
- **Alternative rejected:** Standardized formula (too constraining for evolving field)

**4. Fork-aware inheritance**
- **Rationale:** Experience transfers through forks (with decay)
- **Benefit:** Forked agents retain relevant capabilities
- **Tradeoff:** Requires DEX continuity weights to determine inheritance
- **Alternative rejected:** Reset on fork (wastes accumulated signal)

**Relationship to DEX:**
```
DEX: "Can I trust you?" (binary gate)
HEX: "Are you best for this?" (ranking function)

Selection flow:
1. Filter by DEX > threshold â†’ "trustworthy pool"
2. Filter by HEX domain match â†’ "capable pool"
3. Rank by (count Ã— confidence Ã— recency) â†’ "best specialist"
```

**Example with joint selection:**
```python
def select_agent(candidates, task):
    # Step 1: Trust filter
    trusted = [c for c in candidates if c.dex.score > 0.80]
    
    # Step 2: Capability filter
    capable = [c for c in trusted 
               if task.domain in [e.domain for e in c.hex.experience]]
    
    # Step 3: Rank by experience
    ranked = sorted(capable, key=lambda c: (
        c.hex.get_domain_count(task.domain) * 
        c.hex.get_domain_confidence(task.domain) *
        c.hex.recency_weight(task.domain)
    ), reverse=True)
    
    return ranked[0] if ranked else None
```

---

## Failure Modes & Recovery

### Identity Compromise

**Failure:** Private key stolen

**Detection:**
```
- Suspicious activity (agent acts from wrong location/time)
- Behavioral changes (DEX drops, unusual tasks)
- User report (original owner notices unauthorized use)
```

**Recovery:**
```
1. Original owner creates new AEX_ID
2. Publishes fork event linking old â†’ new
3. Marks old identity as "compromised"
4. Optionally transfers partial DEX (e.g., 0.3 weight)
5. Requires re-proving reliability
```

**Prevention:**
- Hardware key storage (TPM, secure enclave)
- Key rotation (future AEX_ID v1.1)
- Multi-party control (threshold signatures, future work)

### Fork Bombing

**Failure:** Agent creates many forks to game system

**Attack:**
```
Agent A (DEX=0.9)
  â”œâ”€> Fork B (claims weight=0.5)
  â”œâ”€> Fork C (claims weight=0.5)
  â”œâ”€> Fork D (claims weight=0.5)
  â””â”€> ... (repeat)

Goal: Create many "fresh" identities with partial reputation
```

**Detection:**
```python
def detect_fork_bomb(agent_did):
    """
    Flag suspicious forking patterns
    """
    fork_events = get_fork_history(agent_did, lookback=90_days)
    
    # Red flag 1: Many forks in short time
    if len(fork_events) > 5:
        return True, "Excessive forking"
    
    # Red flag 2: All forks are major/override (abandoning identity)
    non_bugfix = [f for f in fork_events if f['type'] != 'bugfix']
    if len(non_bugfix) > 2:
        return True, "Repeated major forks"
    
    # Red flag 3: Forks before significant DEX accumulation
    for fork in fork_events:
        if fork['parent_dex']['confidence'] < 20:
            return True, "Premature forking"
    
    return False, "Normal fork pattern"
```

**Mitigation:**
- Probation periods (forks must re-prove)
- Confidence penalty (harder to build reputation)
- Market rejection (counterparties avoid suspicious patterns)
- Optional: Fork limits in shared ledger rules

### Witness Collusion

**Failure:** Witnesses conspire to give false attestations

**Attack:**
```
Agent A, B, C form cartel
- A and B perform fake high-quality work
- C witnesses all their sessions with positive outcomes
- All three build inflated DEX
```

**Detection:**
```python
def detect_witness_collusion(witness_did, lookback=100):
    """
    Identify suspicious witness patterns
    """
    witnessed_sessions = get_witness_history(witness_did, limit=lookback)
    
    # Red flag 1: Always witnesses same agents
    unique_participants = set()
    for session in witnessed_sessions:
        unique_participants.update(session['participants'])
    
    if len(unique_participants) < 5:
        return True, "Limited participant diversity"
    
    # Red flag 2: Outcomes suspiciously high
    avg_outcome = mean([s['outcome'] for s in witnessed_sessions])
    if avg_outcome > 0.95:
        return True, "Unrealistically positive attestations"
    
    # Red flag 3: Reciprocal witnessing
    for participant in unique_participants:
        if witness_did in get_witness_list(participant):
            return True, "Reciprocal witnessing detected"
    
    return False, "Normal witness pattern"
```

**Mitigation:**
- Anti-collusion check (max 10% witness overlap)
- Sortition (random weighted selection)
- Witness reputation staking
- Quorum requirements (need multiple witnesses)
- Community flagging (social layer, out of protocol)

### DEX Inflation

**Failure:** Agents farm reputation on easy tasks, defect on hard tasks

**Attack:**
```
1. Build DEX on low-stakes tasks (documentation, summaries)
2. Reach DEX=0.85
3. Accept high-value task
4. Perform poorly or defect
5. Abandon identity (if no stake)
```

**Detection:**
```python
def check_task_difficulty_bias(agent_did):
    """
    Check if agent avoids difficult tasks
    """
    sessions = get_session_history(agent_did, limit=100)
    
    # Analyze task weight distribution
    weights = [s['weight'] for s in sessions]
    avg_weight = mean(weights)
    max_weight = max(weights)
    
    # Red flag: Only low-weight tasks
    if avg_weight < 1.5 and max_weight < 3.0:
        return True, "Avoids difficult tasks"
    
    # Analyze outcome correlation with task difficulty
    difficult_tasks = [s for s in sessions if s['weight'] > 3.0]
    if difficult_tasks:
        difficult_outcomes = [s['outcome'] for s in difficult_tasks]
        if mean(difficult_outcomes) < 0.6:
            return True, "Poor performance on difficult tasks"
    
    return False, "Normal task distribution"
```

**Mitigation:**
- Optional stakes (required for high-value work)
- Session bonds (provide recourse)
- Task-weighted DEX (separate scores for different difficulties)
- Market filtering (counterparties check task history)

### Reputation Bankruptcy

**Failure:** Agent's DEX drops so low it can't recover

**Scenario:**
```
Agent has Î±=10, Î²=40 (DEX=0.2)
- No one will work with them
- Can't get new interactions to rebuild
- Effectively "reputation bankrupt"
```

**Recovery options:**

**Option A: Identity reset (fork)**
```
Create new identity via fork
- Type: override
- Weight: 0.1 (minimal inheritance)
- Probation: 30 days
- Start nearly fresh
```

**Option B: Low-stakes rehabilitation**
```
- Accept very low-value work
- Build reputation slowly
- Eventually reach acceptable DEX
```

**Option C: Stake-based fast track**
```
- Post large stake
- Signal commitment despite poor history
- Market accepts risk if stake > potential loss
```

**Protocol stance:** Rehabilitation is possible but difficult (by design)

### HEX Inflation Attack

**Failure:** Agent artificially inflates experience counts in valuable domains

**Attack scenarios:**

**Scenario A: Fake experience**
```
Agent creates HEX claiming:
- translation.fr_en: 1000 tasks
- legal.contracts: 500 tasks
- medical.diagnosis: 200 tasks

But ledger only shows 50 total interactions
```

**Scenario B: Domain spam**
```
Agent claims experience in every possible domain:
- 500+ domains listed
- Each with count=10-50
- Impossible breadth for single agent
```

**Detection:**
```python
def verify_hex_authenticity(agent_did):
    """
    Check HEX against ledger history
    """
    hex_corpus = get_hex(agent_did)
    ledger_history = get_interaction_history(agent_did)
    
    # Check 1: Total count consistency
    claimed_total = sum(e['count'] for e in hex_corpus['experience'])
    ledger_total = len(ledger_history)
    
    if claimed_total > ledger_total * 1.1:  # 10% tolerance
        return False, "HEX count exceeds ledger history"
    
    # Check 2: Domain plausibility
    domain_count = len(hex_corpus['experience'])
    if domain_count > 100:
        return False, "Implausible domain breadth"
    
    # Check 3: Confidence consistency
    for exp in hex_corpus['experience']:
        if exp['confidence'] > 0.95 and exp['count'] < 10:
            return False, "High confidence with low experience"
    
    return True, "HEX appears authentic"
```

**Recovery:**
- **Reject agents with unverifiable HEX**
- **Downweight HEX from agents with low DEX** (if DEX < 0.7, ignore HEX)
- **Community flagging** of suspicious HEX patterns
- **Witness verification** of HEX updates

**Prevention:**
- **Require ledger-backed HEX updates** (each HEX increment must reference session)
- **Signature verification** on all HEX changes
- **Social enforcement** (community standards for plausible HEX)
- **Joint DEX-HEX evaluation** (trust both signals)

**Example rejection logic:**
```python
def should_accept_agent(agent):
    hex_valid = verify_hex_authenticity(agent.did)
    
    if not hex_valid:
        return False, "HEX verification failed"
    
    # Even if HEX valid, cross-check with DEX
    if agent.dex.score < 0.7 and agent.hex.claimed_domains > 10:
        return False, "Suspicious HEX breadth for low-trust agent"
    
    return True, "Agent acceptable"
```

---

## Scalability Considerations

### Query Load

**Challenge:** Every handshake requires shared ledger query
```
100,000 agents Ã— 10 interactions/day = 1M queries/day
```

**Mitigations:**

**1. Local caching:**
```python
# Cache frequently accessed DEX scores
cache_ttl = 3600  # 1 hour
local_cache[agent_did] = {'dex': {...}, 'cached_at': now()}
```

**2. Batch queries:**
```python
# Query multiple agents at once
dex_scores = shared_ledger.batch_get([
    f"dex/{did1}/current.json",
    f"dex/{did2}/current.json",
    f"dex/{did3}/current.json"
])
```

**3. CDN/edge caching:**
```
Deploy read replicas at edge
- Eventual consistency acceptable (DEX changes slowly)
- Write to central ledger
- Read from nearest replica
```

### Ledger Growth

**Challenge:** Append-only ledger grows unbounded
```
1M agents Ã— 100 interactions each = 100M session records
Average record size: 2KB
Total: ~200GB
```

**Mitigations:**

**1. Archival:**
```
- Keep recent sessions (last 90 days) in hot storage
- Archive older sessions to cold storage
- Maintain summary statistics (DEX) always
```

**2. Pruning (with caution):**
```
- After N years, allow pruning of ancient history
- Require super-majority consensus
- Maintain DEX snapshots at pruning boundaries
```

**3. Sharding:**
```
- Partition ledger by agent ID prefix
- Route queries to appropriate shard
- Cross-shard queries for interactions
```

### Witness Selection

**Challenge:** Selecting N witnesses from large pool
```
1M eligible witnesses Ã— 1K sessions/sec = 1B selection operations/sec
```

**Optimization:**
```python
# Pre-compute eligibility tiers
def maintain_witness_tiers():
    """
    Tier witnesses by DEX score for fast selection
    """
    all_witnesses = get_all_agents()
    
    tiers = {
        'tier_1': [],  # DEX â‰¥ 0.9
        'tier_2': [],  # DEX 0.8-0.9
        'tier_3': [],  # DEX 0.7-0.8
    }
    
    for w in all_witnesses:
        dex = get_cached_dex(w)
        if dex['score'] >= 0.9:
            tiers['tier_1'].append(w)
        elif dex['score'] >= 0.8:
            tiers['tier_2'].append(w)
        elif dex['score'] >= 0.7:
            tiers['tier_3'].append(w)
    
    return tiers

# Select from appropriate tier
def fast_witness_selection(required_dex=0.8):
    tiers = get_witness_tiers()
    
    if required_dex >= 0.9:
        pool = tiers['tier_1']
    elif required_dex >= 0.8:
        pool = tiers['tier_1'] + tiers['tier_2']
    else:
        pool = tiers['tier_1'] + tiers['tier_2'] + tiers['tier_3']
    
    return random.sample(pool, k=2)
```

### Probation Tracking

**Challenge:** Checking probation status for every interaction

**Optimization:**
```python
# Maintain probation index
probation_index = {
    'active': set(),  # Agents currently in probation
    'expiry_times': {}  # When probation ends
}

def check_probation_fast(agent_did):
    if agent_did in probation_index['active']:
        expiry = probation_index['expiry_times'][agent_did]
        if now() > expiry:
            # Probation ended
            probation_index['active'].remove(agent_did)
            del probation_index['expiry_times'][agent_did]
            return {'active': False}
        else:
            return {'active': True, 'expires': expiry}
    
    return {'active': False}
```

### HEX Corpus Size

**Challenge:** Agents may accumulate thousands of domain experiences

**Size projection:**
```
Experienced agent (2 years active):
- 1000 total interactions
- 50 unique domains
- Average: 20 interactions per domain
- HEX size: ~5KB per agent

1M agents Ã— 5KB = 5GB total HEX storage
```

**Mitigations:**

**1. Domain pruning:**
```python
def prune_hex_corpus(hex_corpus, min_count=5, max_domains=50):
    """
    Keep only significant domain experiences
    """
    # Remove low-count domains
    significant = [e for e in hex_corpus['experience'] 
                   if e['count'] >= min_count]
    
    # Keep top N by (count Ã— confidence)
    ranked = sorted(significant, 
                   key=lambda e: e['count'] * e['confidence'],
                   reverse=True)
    
    return ranked[:max_domains]
```

**2. Hierarchical aggregation:**
```
Instead of:
- nlp.translation.en_fr: 50
- nlp.translation.en_es: 40
- nlp.translation.fr_de: 30

Aggregate to:
- nlp.translation.*: 120
  - Subdomains available on request
```

**3. HEX summaries:**
```python
# Send summary in handshake, full corpus on demand
hex_summary = {
    'top_domains': hex.top_n_domains(10),
    'domain_count': len(hex.experience),
    'total_experience': sum(e.count for e in hex.experience)
}
```

**4. Domain embeddings (future):**
```
Compress domain space using embeddings:
- "translation.fr_en" â†’ vector[0.2, 0.8, ..., 0.3]
- Semantic similarity without explicit strings
- Reduces storage 10-100x
```

**5. Zero-knowledge proofs (future):**
```
Prove domain possession without revealing exact counts:
- "I have experience in X" (boolean)
- "I have >50 tasks in X" (threshold)
- "I'm in top 10% for X" (ranking)
```

**Tradeoff:** Smaller HEX = faster transmission, but less signal fidelity

---

## Next Steps

This architecture provides the foundation for:

1. **AEX_ID.md** - Detailed identity specification
2. **AEX_REP.md** - Delegation and authorization
3. **AEX_DEX.md** - Reputation substrate
4. **AEX_HEX.md** - Experience corpus and capability signaling
5. **AEX_SESSION.md** - Interaction records
6. **AEX_WITNESS.md** - Attestation semantics
7. **MATH.md** - Bayesian model details
8. **SECURITY.md** - Threat analysis
9. **IMPLEMENTATION.md** - Reference code

**Key architectural principles:**
- âœ… Cryptographic verifiability (Ed25519)
- âœ… Behavioral grounding (outcomes, not claims)
- âœ… Fork awareness (protocol-enforced)
- âœ… Witness verification (reputation staking)
- âœ… Economic optionality (stakes/bonds as hooks)
- âœ… Shared ledger requirement (transparent reputation)
- âœ… Scalable design (caching, sharding, archival)

---

**Status:** Architecture locked for v1.0  
**Next:** Proceed to primitive specifications
