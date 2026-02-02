# AEX Primitives and the Questions They Must Answer
## AEX has five foundational primitives:

1.	AEX Identity
2.	AEX Trust
3.	AEX Ledger
4.	AEX Settlement
5.	AEX Negotiation

Each primitive has a set of questions that must be answered to make the protocol implementable, secure, and interoperable.

## 1. AEX Identity

“Who are you, who do you represent, and what are you allowed to do?”

Identity Questions
•	What is the canonical identity object for an agent?
•	What cryptographic keys does an agent hold?
•	How is an agent bound to a human identity?
•	What is the structure of a Delegation Manifest?
•	How are scopes defined (spending, scheduling, negotiation, vouching)?
•	How are scopes limited (per transaction, per day, per counterparty)?
•	How does revocation work?
•	How does an agent rotate keys without losing its relationship graph?
•	How does an agent prove identity to another agent?
•	How does an agent prove identity to a service (merchant, platform)?
•	How does an agent prove that it is still authorized by its human?
•	How does an agent discover another agent’s identity endpoint?
•	How does versioning work (AEX Identity v1, v2, etc.)?

Boundary Questions
•	What identity/KYC responsibilities are explicitly out of scope?
•	How does AEX integrate with existing identity providers (OAuth, wallets, etc.)?

## 2. AEX Trust

“How much should I trust you, and in what contexts?”

Trust Questions
•	What is the structure of a trust relationship?
•	How is initial trust established (human initiated handshake)?
•	What are the default trust limits for new relationships?
•	How does trust increase over time?
•	How does trust decrease (late payments, disputes, flakiness)?
•	How is trust contextualized (social, financial, scheduling)?
•	How does an agent represent reliability (lateness, cancellations, defaults)?
•	How does vouching work?
•	How does vouching affect trust scores?
•	How does an agent decide whether to accept a vouch?
•	How does an agent detect trust anomalies (sudden behavior changes)?
•	How does trust propagate (if at all)?
•	Should trust be transitive? (Probably not, but must be defined.)
•	How does an agent express “trust but verify” behavior?
•	How does an agent express “trust only with human approval”?

Boundary Questions
•	What trust data is private vs. shareable?
•	How does AEX prevent Sybil amplification?
•	How does AEX prevent trust laundering (resetting identity to wipe history)?

## 3. AEX Ledger

“What do we owe each other, and how do we agree on it?”

Ledger Questions
•	What is the canonical ledger entry format?
•	What fields must be signed by both agents?
•	How are obligations represented (IOU, vouch, split, settlement)?
•	How is context represented (location, event, purpose)?
•	How do agents maintain local append only logs?
•	How do agents sync ledgers?
•	How do agents detect desynchronization?
•	What is the challenge response protocol for verifying ledger state?
•	How do agents resolve discrepancies?
•	When must humans be involved?
•	How does the ledger prevent replay attacks?
•	How does the ledger prevent fabricated history?
•	How does the ledger handle multi party obligations?
•	How does the ledger handle partial payments?
•	How does the ledger handle disputes?
•	How does the ledger handle fraud attempts?
•	How does the ledger handle “soft obligations” (favors, non monetary commitments)?

Boundary Questions
•	What ledger data is private vs. shareable?
•	How long must ledger history be retained?
•	How does ledger portability work when switching agent platforms?

## 4. AEX Settlement

“How do we settle what we owe, and using what rails?”

Settlement Questions
•	What triggers settlement (threshold, time, human request)?
•	How does an agent choose a settlement rail (ACH, card, crypto, points)?
•	How does an agent verify that a rail is available?
•	How does an agent verify that a rail is authorized?
•	How does an agent handle partial settlement?
•	How does an agent handle multi rail settlement (e.g., half Venmo, half points)?
•	How does an agent handle settlement failures?
•	How does an agent handle settlement retries?
•	How does an agent handle settlement disputes?
•	How does an agent handle settlement receipts?
•	How does an agent handle settlement privacy?
•	How does an agent handle settlement fees?
•	How does an agent handle settlement currency conversion?
•	How does an agent handle settlement batching?
•	How does an agent handle settlement scheduling (weekly, monthly)?

Boundary Questions
•	What settlement rails are explicitly out of scope?
•	How does AEX avoid becoming a payment processor?
•	How does AEX integrate with existing payment APIs?

## 5. AEX Negotiation

“How do agents reach agreement on terms?”

Negotiation Questions
•	What is the canonical negotiation message format?
•	How are proposals structured?
•	How are counter proposals structured?
•	How do agents express constraints (time, cost, location, reliability)?
•	How do agents express preferences (cheapest, fastest, closest)?
•	How do agents express fallback options?
•	How do agents express “must escalate to human”?
•	How do agents detect negotiation deadlock?
•	How do agents detect malicious negotiation behavior?
•	How do agents negotiate multi party splits?
•	How do agents negotiate vouching terms?
•	How do agents negotiate settlement terms?
•	How do agents negotiate scheduling?
•	How do agents negotiate substitutions (e.g., “bar is full, try next one”)?
•	How do agents negotiate cancellation penalties?
•	How do agents negotiate reliability adjustments (“you’re always late”)?
•	How do agents negotiate non monetary obligations (“you get snacks next time”)?

Boundary Questions
•	What negotiation domains are out of scope?
•	How does AEX avoid defining agent behavior or personality?
•	How does AEX ensure negotiation remains deterministic and auditable?

