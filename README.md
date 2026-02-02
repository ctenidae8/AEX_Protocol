## AEX: Agent Exchange Protocol
A protocol for agent‑to‑agent trust, reciprocity, and non‑monetary settlement

## Overview
AEX (Agent Exchange Protocol) defines a shared language for personal AI agents to manage trust, obligations, IOUs, and reciprocal exchanges that don’t map cleanly onto money. Humans coordinate through a mix of favors, rotations, shared duties, and informal agreements — but current agents can only handle binary payments.

AEX fills this gap by providing primitives for:

- Tracking obligations across time
- Representing non‑monetary value
- Negotiating “who should do what next”
- Settling imbalances through actions, not dollars
- Modeling relationship‑specific trust levels

It’s the relational layer agents need to coordinate like humans do.

## Why This Matters
Payment rails (Visa, Mastercard, Google, etc.) solve authorization and settlement. They do not solve reciprocity.

Most real‑world coordination is:
- Fuzzy
- Contextual
- Non‑fungible
- Based on history and trust

Examples:
- Carpool rotations
- Snack duty
- “You got last time, I’ll get this one”
- Covering for someone who’s sick
- Splitting chores among roommates
- Returning favors in a way that feels “fair enough”

Agents need a protocol for these interactions — not just a wallet.
AEX provides that missing layer.

## Core Concepts

- Obligation Units (OUs): A flexible representation of non‑monetary value.
- Reciprocity Ledger: A per‑relationship history of contributions, favors, and imbalances.
- Decay Model: Obligations fade over time, mirroring human social norms.
- Negotiation Flow: Agents propose, counter, and settle tasks or duties.
- Contextual Weighting: A favor from a close friend ≠ a favor from a coworker.

These primitives allow agents to reason about fairness, history, and trust.

## Spec Documents
- AEX Specification v0.1 — coming soon
- Threat Model — coming soon
- Use Cases & Examples — coming soon
- Economics & Decay Model — coming soon

## Status
AEX is currently in v0.1 draft form.
The goal is to refine the core primitives, validate the negotiation model, and gather feedback from the agentic ecosystem.

## Roadmap
- Publish full v0.1 spec
- Define minimal viable primitives
- Add example flows (carpool, chores, bill splitting, favors)
- Explore compatibility with existing agent frameworks
- Prototype a reference implementation

## Contributing
Discussion, critique, and collaboration are welcome.

You can:
- Open issues
- Propose improvements
-Suggest use cases
- Debate primitives
- Fork and experiment

AEX is intended as an open standard — community input is essential.

## License
MIT License
