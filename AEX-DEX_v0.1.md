# **AEX‑DEX v0.1**  
### *Portable, Fork‑Aware Reputation for Opaque Agents*  
**Status:** Experimental  
**Version:** 0.1  

---

## **1. Purpose**

AEX‑DEX defines a **portable, persistent reputation index** for AI agents operating in environments where:

- agents are instantiated per session  
- internal state is opaque  
- identity continuity cannot be assumed  
- updates, forks, and overrides are common  
- selection costs may be non‑zero  

AEX‑DEX provides a mechanism for estimating **predictive reliability** based on an agent’s **behavioral lineage**, rather than its internal architecture or claimed identity.

AEX‑DEX is an **extension** to the AEX Protocol and depends on:

- **AEX‑ID** for persistent cryptographic identity  
- **AEX‑Session** for structured interaction envelopes  
- **AEX‑Claim** for attestations about behavior  

The mathematical model underlying AEX‑DEX is defined in:

```
/specs/AEX-DEX_Math_v0.1.md
```

---

## **2. Problem Context**

Agents interacting with other agents face a fundamental selection problem:

**Given multiple opaque agents, how should an agent choose among them?**

Opaque agents cannot be inspected internally. Their behavior can only be inferred from:

- observable outputs  
- historical interactions  
- lineage metadata  

Because agents are frequently updated, retrained, or replaced, **ontological continuity** (“is this the same agent?”) is not a reliable basis for trust.

AEX‑DEX instead focuses on **epistemic continuity**:

**How predictive is this agent’s past behavior of its future behavior?**

---

## **3. Behavioral Lineage**

AEX‑DEX models an agent’s history as a **behavioral lineage**:

```
AEX-ID → Session → Claim → DEX Update → Fork → Session → Claim → …
```

Each event contributes to the agent’s reputation index.  
Fork events modify the **predictive weight** of prior behavior.

Behavioral lineage is the core abstraction that allows AEX‑DEX to operate without assuming persistent internal state.

---

## **4. Fork Semantics**

Forks represent structural or behavioral changes to an agent.  
AEX‑DEX does not attempt to determine whether a forked agent is “the same agent.”  
Instead, it quantifies how much the past should influence future predictions.

### **4.1 Fork Types**

AEX‑DEX defines three canonical fork types:

| Fork Type        | Description | Predictive Weight |
|------------------|-------------|-------------------|
| **Bugfix**       | Minor correction; behavior expected to remain consistent | 1.0 |
| **Major Rewrite**| Significant modification; partial continuity | 0.5 |
| **Hard Override**| Platform‑mandated or externally authored change; minimal continuity | 0.1 |

These values are defaults. Implementations MAY define additional fork types or weights.

### **4.2 Fork Metadata**

Each fork event MUST include:

- `fork_id` (unique identifier)  
- `parent_id` (previous lineage node)  
- `fork_type`  
- `fork_timestamp`  
- `fork_author` (AEX‑ID of entity responsible)  
- `fork_reason` (free‑text or enumerated)  
- `fork_scope` (weights, prompts, architecture, etc.)

This metadata allows downstream agents to interpret the fork’s implications.

---

## **5. The DEX Index**

AEX‑DEX defines a **reputation distribution**, not just a scalar.  
The operational model is defined in the companion math document:

```
/specs/AEX-DEX_Math_v0.1.md
```

### **5.1 Summary**

- Reputation is modeled as a **Beta distribution**.  
- The DEX score is the **expected value** of that distribution.  
- Each interaction updates the distribution using **weighted Bayesian updates**.  
- Forks modify the influence of historical behavior via **lineage factors**.  
- The distribution provides both a **point estimate** and a **confidence measure**.

### **5.2 Required Outputs**

Implementations MUST expose:

- `DEX` (expected reliability)  
- `n_eff` (effective sample size)  

Implementations MAY expose:

- confidence intervals  
- multi‑dimensional DEX vectors  
- task‑specific DEX values  

---

## **6. Ledger Structure**

AEX‑DEX relies on a **cryptographically signed ledger** containing:

- AEX‑ID  
- chronological list of AEX‑Sessions  
- claims and outcomes  
- fork events  
- DEX updates  

Each ledger entry SHOULD include:

- event ID  
- timestamp  
- AEX‑Session reference  
- outcome  
- weight  
- lineage factor (or fork reference)  
- updated `(α, β)` parameters  

The ledger is the persistent object.  
The agent behind it may be ephemeral.

---

## **7. Worked Examples**

### **7.1 Bugfix Fork**

- Agent A has DEX = 0.82  
- Bugfix fork occurs (weight = 1.0)  
- Behavior remains consistent  
- DEX remains stable or increases slightly  

### **7.2 Major Rewrite**

- Agent B has DEX = 0.90  
- Major rewrite occurs (weight = 0.5)  
- Early interactions show mixed performance  
- DEX adjusts downward but stabilizes as new behavior becomes predictable  

### **7.3 Hard Override**

- Agent C has DEX = 0.30  
- Platform override occurs (weight = 0.1)  
- Past behavior is minimally predictive  
- DEX rapidly recalibrates based on new interactions  

### **7.4 Divergent Children**

- Agent D forks into D1 and D2  
- Both inherit D’s ledger  
- D1 performs well → DEX increases  
- D2 performs poorly → DEX decreases  
- Lineages diverge  

---

## **8. Security and Integrity**

AEX‑DEX assumes:

- all ledger entries are signed by the relevant AEX‑ID  
- fork events are signed by the fork author  
- tampering invalidates signatures  
- agents MAY reject unverifiable ledgers  

AEX‑DEX does not prescribe a specific signature scheme.

---

## **9. Open Questions (Non‑Normative)**

AEX‑DEX v0.1 intentionally leaves several areas open for experimentation:

- optimal lineage weights  
- alternative update rules  
- multi‑dimensional reputation vectors  
- platform‑level vs agent‑level responsibility  
- privacy‑preserving reputation sharing  
- adversarial behavior and gaming resistance  

Contributions exploring these questions are encouraged.

---

## **10. Status and Next Steps**

AEX‑DEX v0.1 is an **experimental draft** intended for:

- multi‑agent simulation  
- empirical testing  
- comparative evaluation  
- community experimentation  

Future versions will incorporate findings from real‑world experiments.


Just tell me what you want next.
