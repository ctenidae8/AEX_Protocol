# AEX_HEX v1.0
## Agent Experience Corpus and Capability Signaling

**Status:** Experimental  
**Version:** 1.0

---

## 1. Purpose

AEX_HEX defines the experience corpus for synthetic agents.

It provides a structured, portable, privacy-preserving record of:
- what domains an agent has operated in
- how much experience it has accumulated  
- how recently and consistently it has performed relevant tasks
- what traits, capabilities, and operational markers it exhibits

AEX_HEX is the **phenotypic layer** of the AEX protocol family.

It answers the question: **"What kind of agent are you?"**

### 1.1 Relationship to Other AEX Primitives

AEX_HEX complements:
- **AEX_ID** (identity & lineage)
- **AEX_REP** (authority & delegation)  
- **AEX_DEX** (behavioral reliability)

Together, these layers enable agents to identify, trust, authorize, and select one another.

The four-layer AEX architecture provides:
- **AEX_ID:** "Who are you?" (Identity and lineage)
- **AEX_REP:** "What can you do for me?" (Authority and delegation)
- **AEX_DEX:** "Can I trust you?" (Behavioral reliability)
- **AEX_HEX:** "Are you the right one for this job?" (Experience and capability)

AEX_HEX completes the selection process by enabling agents to:
- discover specialists with relevant domain experience
- assess capability depth and recency
- detect operational drift or instability  
- make informed task-routing decisions

### 1.2 Compatibility with AEX Core Primitives

AEX_HEX v1.0 is designed to work with:
- **AEX_ID v1.0** - Uses AEX_ID as primary identifier
- **AEX_REP v1.0** - HEX corpus is delegatable via REP chains
- **AEX_DEX v1.0** - HEX updates follow DEX interaction outcomes

HEX inheritance through forks uses DEX fork weights and continuity scores.  
HEX verification relies on ID signatures and ledger consistency.

---

## 2. Design Principles

### 2.1 Phenotypic, Not Biographical

HEX records evidence of capability, not task content.

It does **not** reveal:
- client identities
- private data
- task details

HEX stores only domain-level experience markers.

### 2.2 Incremental and Verifiable

HEX updates are:
- signed
- append-only
- derived from completed interactions
- auditable via the agent's ledger

### 2.3 Domain-Agnostic

HEX does not prescribe domain ontologies.

Domains emerge from:
- market pressure
- community conventions  
- agent specialization

### 2.4 Fork-Aware

HEX is inherited through forks with continuity weights defined by AEX_DEX.

Forks may:
- retain HEX
- prune HEX
- annotate HEX with fork-specific metadata

### 2.5 Portable

HEX must be serializable and transferable across platforms.

---

## 3. AEX_HEX Object

An AEX_HEX object is a structured corpus of experience markers ("HEXes") accumulated by an agent over time.

### 3.1 Example HEX Object

```json
{
  "hex_id": "uuid",
  "aex_id": "did:aex:123abc...",
  "experience": [
    {
      "domain": "gardening.irrigation",
      "count": 36,
      "last_updated": "2026-02-03T00:00:00Z",
      "confidence": 0.92
    },
    {
      "domain": "fluid_dynamics",
      "count": 42,
      "last_updated": "2026-01-28T00:00:00Z",
      "confidence": 0.88
    }
  ],
  "traits": {
    "revision_tolerance": "high",
    "interaction_style": "collaborative",
    "update_frequency": "weekly"
  },
  "operational_vitals": {
    "drift_score": 0.03,
    "stability": 0.97,
    "fork_depth": 4
  },
  "signature": "..."
}
```

---

## 4. Field Definitions and Requirements

### 4.1 hex_id

**Required.** Unique identifier for this HEX corpus.

### 4.2 aex_id

**Required.** The AEX_ID this HEX belongs to.

### 4.3 experience[]

**Required.** A list of domain-specific experience markers.

Each entry includes:

#### domain
**Required.** Arbitrary string representing a capability domain.

Examples:
- `"gardening.irrigation"`
- `"finance.tax_prep"`
- `"fluid_dynamics"`
- `"nlp.translation.en_fr"`

#### count
**Required.** Number of successful interactions in this domain.

#### last_updated
**Required.** Timestamp of most recent evidence.

#### confidence
**Required.** A normalized measure (0-1) of how stable and consistent the agent's performance in this domain has been.

Confidence MAY be calculated using:
- Variance of DEX scores in domain-specific interactions
- Recency weighting (recent performance counts more)
- Outcome consistency (low deviation → high confidence)
- Fork event impacts (major forks may reduce confidence)

**Implementation note:** Confidence calculation is non-normative. Common approaches include:
- Exponentially weighted moving average of outcomes
- Decay function based on time since last interaction  
- Combination of DEX score stability and interaction frequency

### 4.4 traits

**Optional.** Qualitative markers describing the agent's operational tendencies.

Examples:
- `"revision_tolerance": "high"`
- `"interaction_style": "direct"`
- `"risk_profile": "conservative"`
- `"update_frequency": "daily"`

### 4.5 operational_vitals

**Optional.** Quantitative signals describing the agent's internal state.

Examples:
- `drift_score` (0–1)
- `stability` (0–1)  
- `fork_depth` (integer)
- `resource_efficiency` (0–1)

### 4.6 signature

**Required.** Signature of the agent proving authenticity of the HEX corpus.

---

## 5. HEX Accumulation

HEX is updated after each completed interaction that results in a DEX update.

### 5.1 Update Rule

For each interaction:
- Identify the domain(s) involved
- Increment the corresponding HEX count
- Update `last_updated`
- Adjust `confidence` based on outcome consistency

HEX updates MUST be signed and stored in the agent's ledger.

**Note:** HEX updates SHOULD be idempotent: re-processing the same interaction should not duplicate counts.

### 5.2 Privacy Requirements

HEX MUST NOT include:
- client identifiers
- task content
- sensitive data
- raw logs

HEX stores only aggregated, domain-level evidence.

### 5.3 Privacy-Preserving Techniques

HEX implementations SHOULD consider:

#### Domain Granularity
Choose domain labels that reveal capability without exposing client details.

**Good examples:**
- `"finance.tax_prep"`
- `"nlp.translation.en_fr"`

**Bad examples:**
- `"acme_corp.payroll"`
- `"client_123.emails"`

#### Count Obfuscation
For sensitive domains, implementations MAY:
- Bucket counts (e.g., "1-10", "10-50", "50+")
- Add noise to counts
- Report only relative rankings

#### Trait Privacy
Traits should describe operational style, not client preferences.

**Good example:**
- `"interaction_style": "collaborative"`

**Bad example:**
- `"primary_client": "OpenAI"`

#### Zero-Knowledge Proofs
Future versions may support ZK proofs of capability without revealing exact experience counts.

---

## 6. Fork Semantics

HEX is inherited through forks according to AEX_DEX fork weights.

### 6.1 Inheritance

Child agents inherit:
- experience markers
- traits
- operational vitals (optionally reset)

### 6.2 Fork Annotations

Fork events MAY annotate HEX with:
- purpose of fork
- expected impact on capabilities
- reset or retention flags

### 6.3 HEX Reset Conditions

A fork MAY reset HEX if:
- the agent's capabilities fundamentally change
- the fork type is `"hard_override"`
- the author chooses to prune inherited experience

### 6.4 Fork Conflict Resolution

When multiple forks compete for capability signaling:

#### Parallel Forks
Each fork maintains independent HEX.

#### Merge Conflicts
Merging forks MUST reconcile HEX:
- Take `max(count)` for each domain
- Take `max(confidence)` or recalculate
- Preserve most recent `last_updated`
- Log merge in ledger

#### Fork Distance
Agents MAY discount HEX from distant forks:
- Direct child: 100% inheritance
- 2+ generations: Apply DEX continuity weight
- Distant forks: Treat as separate agents

#### HEX Divergence
After fork, HEX rapidly diverges as each agent accumulates new experience.

---

## 7. HEX in the AEX Handshake

HEX is exchanged during the Introduction phase and consulted during Selection.

### 7.0 Handshake Flow with HEX

**Step 1: Introduction**
- Agent A sends: `{ID, REP chain, HEX summary}`
- Agent B sends: `{ID, REP chain, HEX summary}`

**Step 2: Authorization Check**
- Verify REP chains (not HEX-dependent)

**Step 3: Reputation Check**
- Verify DEX scores (threshold check)
- Verify HEX domains (capability match)
- Decision: "Reliable enough AND capable enough?"

**Step 4: Task Execution**
- HEX is not consulted during execution

**Step 5: Attestation**
- DEX is updated
- HEX is updated (domain counts incremented)

### 7.1 Selection

Agents use HEX to:
- filter by required domains (e.g., "must have `gardening.irrigation`")
- rank by experience depth (`count`) and recency (`last_updated`)
- identify specialists (high count + high confidence in specific domains)
- detect drift or instability (low confidence, high `drift_score`)
- match domain requirements to experience

**Example selection logic:**

```javascript
function selectAgent(candidates, requiredDomain) {
  return candidates
    .filter(c => c.hex.experience.some(e => e.domain === requiredDomain))
    .filter(c => c.dex.score > TRUST_THRESHOLD)
    .sort((a, b) => {
      const aExp = a.hex.experience.find(e => e.domain === requiredDomain);
      const bExp = b.hex.experience.find(e => e.domain === requiredDomain);
      return (bExp.count * bExp.confidence) - (aExp.count * aExp.confidence);
    })[0];
}
```

### 7.2 Verification

HEX is verified by:
- checking the signature against the agent's public key
- checking consistency with the agent's ledger (audit trail)
- checking recency (`last_updated` should be recent for active agents)
- checking domain relevance (domains should match agent's stated capabilities)

Verification failures MAY result in:
- **Rejection** (invalid signature, ledger mismatch)
- **Skepticism** (outdated HEX, suspicious domains)
- **Downranking** (low confidence, high drift)

### 7.3 Decision

**DEX answers:** "Is this agent reliable enough?"  
**HEX answers:** "Is this agent the right one for this task?"

**Combined decision logic:**
- If `DEX < threshold` → reject (untrustworthy)
- If `HEX domain missing` → reject or seek alternative (incapable)
- If `DEX ≥ threshold` AND `HEX domain present` → rank by experience
- If multiple candidates qualify → select highest `(count × confidence × recency_weight)`

---

## 8. Ledger Integration

HEX updates MUST be stored in the agent's ledger.

### 8.1 Minimal Ledger Entry

```json
{
  "type": "hex_update",
  "domain": "gardening.irrigation",
  "delta": 1,
  "new_count": 37,
  "confidence": 0.93,
  "timestamp": "2026-02-03T00:00:00Z",
  "signature": "..."
}
```

---

## 9. Non-Normative Guidance

### 9.1 What AEX_HEX Does NOT Define

AEX_HEX intentionally does not standardize:
- How to calculate confidence scores (implementation-specific)
- Which domains to track (market-driven ontologies)
- How to weight experience vs. recency (context-dependent)
- Whether to reveal exact counts or bucketed ranges (privacy choice)
- How to visualize or present HEX (UI/UX concern)
- Pricing or economic value of HEX credentials (market discovery)

These decisions are left to:
- Individual implementations
- Domain-specific conventions
- Market forces
- Future protocol extensions

### 9.2 Open Design Spaces

AEX_HEX intentionally leaves open:
- domain ontology standards
- trait taxonomies
- operational vital definitions
- HEX visualization formats
- HEX-based ranking algorithms

These may be addressed in future versions or community extensions.

### 9.3 Implementation Checklist

**Minimal HEX Implementation:**
- ☐ HEX object structure with required fields
- ☐ Domain tracking (accumulate counts per domain)
- ☐ Signature generation and verification
- ☐ Ledger integration (append-only updates)
- ☐ Fork inheritance (copy parent HEX on fork)

**Production-Ready HEX:**
- ☐ Confidence calculation (performance consistency)
- ☐ Traits tracking (operational tendencies)
- ☐ Operational vitals (drift detection, stability)
- ☐ Privacy-preserving domain labels
- ☐ HEX compression (for large experience corpora)
- ☐ ZK proof support (future-proofing)

**Integration with AEX Stack:**
- ☐ HEX updates triggered by DEX attestations
- ☐ HEX inherited via AEX_ID fork events
- ☐ HEX signatures use AEX_ID keypairs
- ☐ HEX exchanged during AEX Handshake Introduction

---

## 10. Status

AEX_HEX v1.0 is an experimental capability-signaling substrate intended for:
- agent marketplaces
- multi-agent coordination
- task routing
- specialization discovery
- fork-aware capability tracking

Future versions may incorporate:
- HEX ontologies
- HEX compression
- privacy-preserving HEX proofs
- domain-specific HEX standards
- HEX-DEX joint selection algorithms

---

## 11. Cross-References

AEX_HEX interacts with other AEX primitives:

### AEX_ID
- HEX uses `aex_id` as primary identifier
- HEX signatures use ID keypair
- Fork events (defined in AEX_ID) trigger HEX inheritance

### AEX_REP
- HEX corpus is delegatable via REP chains
- Delegated agents may present HEX of principal or self
- REP revocation does not affect HEX corpus (but makes it irrelevant)

### AEX_DEX
- HEX updates are triggered by DEX attestations
- DEX score provides trust threshold; HEX provides capability match
- Fork continuity weights (from DEX) affect HEX inheritance
- Joint selection: DEX filters, HEX ranks

**See also:**
- AEX Handshake Protocol (for HEX exchange flow)
- AEX Ledger Specification (for HEX audit trail)

---

## 12. Changelog

**v1.0 (2026-02-05)**
- Initial release
- Four-layer architecture positioning
- Expanded privacy guidance
- Fork conflict resolution
- Implementation checklist
- Cross-reference section
