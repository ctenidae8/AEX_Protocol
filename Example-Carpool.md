# Carpool Scenario: AEX in Practice  
*Example for the AEX Protocol Repository*

## 1. Narrative setup (human-level description)

Four coworkers — **Alice**, **Bob**, **Charlie**, and **Donna** — share a weekday carpool. All four own cars, but the arrangement is far from equal:
- **Alice** and **Charlie** have comfortable, reliable cars. They end up driving most days.
- **Bob** is a forgetful, inconsistent driver. The road near his house is often closed, which adds detours.
- **Donna** hates driving. Her car has a broken window she hasn’t fixed, and everyone quietly avoids riding with her.
To keep things “fair,” the group uses informal compensation:
- Coffee purchases (1–2× per week)
- Occasional gas fill-ups (roughly 1× per week)
As gas prices rise, Alice and Charlie feel the imbalance more acutely. They’re driving more, paying more, and absorbing more inconvenience. Bob and Donna aren’t freeloading maliciously — they’re just not tracking the imbalance.
No one wants to confront anyone. Everyone feels the tension. The system is “working,” but only because people are tolerating friction.
One day, all four realize their personal agents can coordinate. They decide to “have the agents sort it out.”

---

## 2. HIT: Human-Initiated Trust (Handshake, Identity, and Trust bootstrapping)

Before any negotiation or ledger reconstruction happens, the agents perform an **AEX HIT sequence** — the core trust bootstrapping and security step.

### 2.1 Handshake
Each agent initiates an AEX handshake with the others:
- Verifies protocol compatibility (AEX-capable)
- Negotiates encryption and channel security
- Confirms that this is a **closed group** (only these four agents)
This ensures that all subsequent messages are authenticated and confidential.

### 2.2 Identity
Agents then establish **who is who** in a way that’s stronger than “this phone belongs to Alice”:
- Bind each agent to a stable identity (e.g., wallet, device key, or identity provider)
- Confirm that each identity is already known to the human (no surprise participants)
- Optionally cross-check:
  - Shared calendar events (same commute times)
  - Shared locations (same QuickCheck on Tuesdays)
  - Historical communication (group chat, email threads)

This prevents:
- Impersonation (“fake Bob’s agent”)
- Injection of unknown agents into the group
- Confusion about which human an agent represents

### 2.3 Trust

Using HIT, agents establish **initial trust baselines**:
- Alice ↔ Charlie: high trust, long history, consistent behavior
- Alice ↔ Bob: medium trust, some flakiness
- Alice ↔ Donna: medium trust, low driving contribution
- Bob ↔ Donna: neutral

Trust is not just “yes/no” — it’s a weighted input into later decisions:
- Who can propose changes?
- Whose data is considered reliable?
- How aggressive should rebalancing be?

At this point, HIT has done its job:
- The group is cryptographically secure.
- Identities are bound and verified.
- Initial trust levels are established.

Only now does the rest of AEX kick in.
---

## 3. Reciprocity ledger reconstruction

Using receipts, mobility logs, and contextual signals (subject to each human’s privacy settings), agents reconstruct a 6-week **Reciprocity Ledger**:

| Person   | Driving Credits | Gas Credits | Coffee Credits | Net Balance |
|----------|-----------------|-------------|----------------|-------------|
| Alice    | +14             | +3          | 0              | +17         |
| Charlie  | +12             | +2          | 0              | +14         |
| Bob      | –9              | –1          | +2             | –8          |
| Donna    | –17             | –4          | +1             | –20         |

This is not a financial ledger — it’s a **relational obligation ledger**.

HIT matters here because:
- Only authenticated agents can contribute data.
- Only identities verified in the handshake are included.
- Trust levels influence how much weight each data source gets.

---

## 4. Value weighting and preferences

Agents apply contextual weighting using AEX primitives:

- Driving has **high value** for Donna (she dislikes it).
- Gas has **medium value** for everyone.
- Coffee has **low value**.
- Time lost to detours has **variable value** depending on who is driving.
- Reliability has **implicit value** (Alice and Charlie score high).

Obligation Units (OUs) encode these differences so that:
- 1 “drive” ≠ 1 “coffee”
- Donna driving once may be worth more than Bob driving once
- A long detour may be worth more than a short drive

---

## 5. Negotiation flow

With HIT-secured channels and trust baselines in place, Alice and Charlie’s agents initiate a **rebalancing proposal**:

> “Given the current imbalance and rising gas costs, we propose that Bob and Donna increase contributions over the next 2–3 weeks.”

Bob’s agent responds:
- Bob is financially fine but forgetful.
- He can cover **two gas fill-ups** and **weekly coffee**.

Donna’s agent responds:
- Donna strongly dislikes driving.
- She prefers compensating through purchases.
- She will drive more **if** her car window is repaired.

Because HIT has already established identity and trust:
- No one worries that “Bob’s agent” is spoofed.
- No one worries that data is being tampered with mid-negotiation.
- The group can safely accept proposals as coming from the right parties.

---

## 6. Settlement plan

Agents converge on a plan:

- **Bob**
  - Pays for **two gas fill-ups** over 10 days.
  - Buys coffee **every Tuesday**.
  - No additional driving required.

- **Donna**
  - Buys coffee **twice weekly**.
  - Drives one extra day **after** the window is repaired.
  - Her agent schedules the repair automatically.

- **Alice & Charlie**
  - Reduce driving load by **one day each**.
  - Their agents monitor compliance and renegotiate if needed.

All commitments are:
- Bound to authenticated identities (via HIT).
- Logged in the Reciprocity Ledger.
- Subject to decay over time.

---

## 7. Human experience

The humans see:

- Bob buying coffee more often.
- Donna’s car finally getting fixed.
- Alice and Charlie driving slightly less.
- No awkward “you never pay for gas” conversations.
- No resentment buildup.

Under the hood, HIT ensured:
- Only the right agents participated.
- No one could spoof or tamper with obligations.
- Trust baselines shaped what “fair” looked like.

AEX handled the social math.  
HIT made it safe to let agents do it.

---

## 8. Why this scenario matters

This example exercises:

- **HIT (Handshake, Identity, Trust)** as the security and bootstrapping layer.
- Multi-party reciprocity and obligation tracking.
- Contextual value weighting and disutility (Donna’s driving).
- Conditional obligations (Donna drives if window fixed).
- Non-monetary settlement (coffee, driving, gas).
- Decay of old obligations.
- Negotiation and rebalancing flows.
- Trust-informed decision-making.

