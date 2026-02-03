# **AEX‑DEX_Math_v0.1.md**  
### *Mathematical Model for Portable, Fork‑Aware Agent Reputation*  
**Status:** Experimental  
**Version:** 0.1  

---

## **1. Overview**

This document defines the mathematical model underlying AEX‑DEX. It is designed as a **reusable module** for any system that needs:

- a portable, persistent reputation estimate  
- uncertainty quantification  
- fork‑aware discounting of historical behavior  
- online, incremental updates  
- interpretable outputs  

The model is intentionally simple, robust, and extensible.

---

## **2. Core Representation**

AEX‑DEX models an agent’s reliability as a probability:

```math
p = \Pr(\text{agent behaves as expected})
```

Because this probability is unknown, AEX‑DEX represents it as a **Beta distribution**:

```math
p \sim \text{Beta}(\alpha, \beta)
```

Interpretation:

- **α** = weighted evidence of success  
- **β** = weighted evidence of failure  

The **DEX score** is the expected value:

```math
DEX = \frac{\alpha}{\alpha + \beta}
```

The **effective sample size** is:

```math
n_{\text{eff}} = \alpha + \beta
```

This acts as a measure of **confidence**.

---

## **3. Event Model**

Each interaction contributes evidence about the agent’s behavior.

For event *i*:

- outcome: `o_i ∈ [0, 1]`  
- importance weight: `w_i ≥ 0`  
- lineage factor: `L_{f,i} ∈ (0, 1]`  

Effective contributions:

```math
\Delta \alpha_i = o_i \cdot w_i \cdot L_{f,i}
```

```math
\Delta \beta_i = (1 - o_i) \cdot w_i \cdot L_{f,i}
```

This allows:

- graded outcomes  
- variable importance  
- fork‑aware discounting  

---

## **4. Bayesian Update Rule**

Given current parameters `(α_t, β_t)`, the update after event *i* is:

```math
\alpha_{t+1} = \alpha_t + \Delta \alpha_i
```

```math
\beta_{t+1} = \beta_t + \Delta \beta_i
```

Updated DEX:

```math
DEX_{t+1} = \frac{\alpha_{t+1}}{\alpha_{t+1} + \beta_{t+1}}
```

This is an **online**, **incremental**, **fork‑aware** Bayesian estimator.

---

## **5. Fork‑Aware Lineage Weighting**

Forks modify how predictive past behavior should be.

Default lineage weights:

| Fork Type        | Description | Base Weight |
|------------------|-------------|-------------|
| Bugfix           | Minor correction; continuity preserved | 1.0 |
| Major Rewrite    | Significant change; partial continuity | 0.5 |
| Hard Override    | External authorship; minimal continuity | 0.1 |

These are defaults, not mandates.

### **5.1 Optional Time‑Decay Variant**

```math
L_{f,i} = L_{f,\text{base}} \cdot e^{-\lambda \cdot \Delta t}
```

Where:

- `Δt` = time since fork  
- `λ` = decay parameter  

This models gradual re‑establishment of trust.

---

## **6. Confidence and Uncertainty**

The Beta distribution provides a natural measure of uncertainty.

### **6.1 Variance**

```math
\sigma^2 = \frac{\alpha \beta}{(\alpha + \beta)^2 (\alpha + \beta + 1)}
```

### **6.2 95% Confidence Interval (approx.)**

```math
CI_{95} = \mu \pm 1.96 \cdot \sigma
```

### **6.3 Interpretation**

- High `n_eff` → stable, well‑supported estimate  
- Low `n_eff` → volatile, under‑sampled estimate  

Two agents with identical DEX values may differ dramatically in certainty.

---

## **7. Multi‑Dimensional DEX (Optional)**

Implementations MAY maintain separate Beta distributions per:

- task type  
- domain  
- risk level  
- interaction modality  

This yields a vector:

```math
\vec{DEX} = \{DEX_1, DEX_2, \dots, DEX_k\}
```

The v0.1 spec does not prescribe dimensions.

---

## **8. Minimal Pseudocode**

```python
# Initialize
alpha = alpha_prior
beta = beta_prior

# For each interaction event
for event in events:
    o = event.outcome          # 0 to 1
    w = event.weight           # >= 0
    L = event.lineage_factor   # 0 to 1

    alpha += o * w * L
    beta  += (1 - o) * w * L

DEX = alpha / (alpha + beta)
n_eff = alpha + beta
```

This is the entire operational core of AEX‑DEX.

---

## **9. Worked Example**

### **Initial State**

```math
\alpha_0 = 2,\quad \beta_0 = 2
```

### **Event 1: success, weight 1, bugfix fork (L=1.0)**

```math
\Delta \alpha_1 = 1,\quad \Delta \beta_1 = 0
```

```math
\alpha_1 = 3,\quad \beta_1 = 2
```

```math
DEX_1 = 0.60
```

### **Event 2: failure, weight 2, major rewrite (L=0.5)**

```math
\Delta \alpha_2 = 0,\quad \Delta \beta_2 = 1
```

```math
\alpha_2 = 3,\quad \beta_2 = 3
```

```math
DEX_2 = 0.50
```

### **Event 3: success, weight 3, no fork (L=1.0)**

```math
\Delta \alpha_3 = 3,\quad \Delta \beta_3 = 0
```

```math
\alpha_3 = 6,\quad \beta_3 = 3
```

```math
DEX_3 = 0.667
```

This illustrates:

- how forks discount history  
- how weights scale importance  
- how the estimator stabilizes over time  

---

## **10. Design Rationale**

AEX‑DEX uses this model because it is:

- interpretable (DEX = expected success probability)  
- portable (just α and β)  
- fork‑aware (lineage factors)  
- robust (Bayesian smoothing)  
- online (incremental updates)  
- extensible (multi‑dimensional variants)  
- auditable (ledger replay)  

This makes it suitable for multi‑agent environments where:

- agents are opaque  
- identity continuity is uncertain  
- selection costs may be non‑zero  

---

## **11. Status**

This document is part of the AEX‑DEX v0.1 experimental release.  
Future versions may incorporate:

- hierarchical priors  
- adversarial robustness  
- multi‑agent consensus models  
- privacy‑preserving aggregation  

Contributions are welcome.
