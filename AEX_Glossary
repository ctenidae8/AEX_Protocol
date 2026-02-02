# AEX Protocol Glossary
Version: 0.1
Last Updated: January 2026
This glossary defines key terms, concepts, and innovations introduced in the AEX (Agent Exchange) Protocol specification.
________________________________________
## Core Protocol Terms

AEX (Agent Exchange)
A trust protocol for personal AI agent interactions that enables coordination, financial exchange, and authentication between agents representing individual humans.

Agent
An AI-powered software entity that acts on behalf of a human user to coordinate activities, manage relationships, negotiate terms, and execute transactions.

Personal Agent
An AI agent that represents an individual human (as opposed to a commercial or enterprise agent). Personal agents interact with other personal agents and commercial agents to accomplish tasks for their humans.
________________________________________
## Trust and Identity

HIT (Human-Initiated Trust)
The process by which two humans explicitly authorize their agents to establish a relationship. To "HIT up" an agent means to initiate a formal agent-to-agent relationship that mirrors the human relationship.
Example: "I HITup Bob's agent so we can coordinate carpools."

HIT Request
The formal message sent when one agent initiates a relationship with another agent, requiring human authorization on both sides.

Trust Level
A quantified measure of reliability between two agents, based on interaction history, successful coordination, payment patterns, and vouching behavior. Trust levels determine what permissions and obligations agents can negotiate autonomously.

Trust Progression
The automatic evolution of trust from low-stakes coordination (scheduling) to relationship hierarchy (priorities) to financial obligations (IOUs) based on successful interaction history.

Context-Bound Trust
Temporary trust granted for a specific transaction or situation, independent of overall relationship trust. Enables coordination between agents without deep relationship history.
Example: Three colleagues splitting lunch can establish context-bound trust for that single transaction.

Witness Agent
An anonymous agent in a user's trusted network that maintains encrypted backup copies of ledgers for redundancy and verification. Witnesses are automatically selected based on trust levels.

Anonymous Witness Network
The distributed set of highly-trusted agents that collectively maintain encrypted copies of transaction ledgers without knowing whose ledgers they're witnessing.
________________________________________

## Financial Concepts

$GOTCHA Economy
The relationship-backed system of IOUs that operates largely outside traditional financial infrastructure. Named for the social dynamic "I gotcha this time" / "I'll getya next time."

$GOTCHA
Conceptual unit of account representing one dollar worth of IOU within the AEX ecosystem (1 $GOTCHA = $1 USD IOU). May or may not become an actual token in future versions.

IOU (I Owe You)
A recorded obligation from one agent to another, tracked in both agents' ledgers. IOUs can accumulate over time and settle periodically rather than immediately. Consider changing to Obligation Unit or "OU" to avoid tax issues.

IOU Limit
The maximum balance one agent can owe another before settlement is required. Limits scale with trust level and relationship depth.

Relationship Capital
The accumulated trust, history, and mutual obligation between two agents that enables higher-velocity, lower-friction transactions.

Settlement Threshold
The IOU balance or time period that triggers automatic settlement between agents.

Off-Chain Settlement
IOU tracking and accumulation that occurs without touching traditional payment rails or blockchain. Value circulates as ledger entries rather than currency transfers.

On-Chain Settlement
Final settlement of IOU balances via cryptocurrency or stablecoin transactions on a blockchain.
________________________________________

## Transaction Types

Direct IOU
A simple obligation where one agent owes another a specific amount, recorded in both ledgers and subject to trust limits.

Immediate Settlement
A transaction that settles instantly via traditional payment rail (Venmo, Zelle) or crypto, with no ongoing IOU balance.

Vouched Transaction
A transaction where a third party agent guarantees payment on behalf of the debtor, assuming counterparty risk.
Example: Alice's agent vouches for Charlie's $30 debt to Bob.

Vouching
The act of one agent guaranteeing another agent's obligation, creating a trust chain. Vouching history affects an agent's reputation.

Multi-Party Split
A transaction involving three or more agents dividing a shared expense, with settlement terms negotiated based on pairwise trust relationships.

Credit Guarantor Transaction
A transaction where a commercial credit service (Klarna, Affirm, etc.) guarantees payment to enable transaction completion when peer vouching or immediate payment isn't available.
________________________________________

## Technical Components

AEX-Identity
The primitive handling agent authentication, human attestation, key management, and delegation.

AEX-Trust
The primitive handling relationship scoring, trust evolution, contextualization, and reputation.

AEX-Ledger
The primitive handling transaction recording, synchronization, verification, and conflict resolution.

AEX-Settlement
The primitive handling payment rail selection, batching, scheduling, and dispute resolution.

AEX-Negotiation
The primitive handling multi-party coordination, proposal/counter-proposal, deadlock detection, and escalation.

Agent Ledger
An append-only log maintained by each agent for each relationship, recording all transactions, attestations, and state changes.

Ledger Synchronization
The periodic process by which two agents compare and reconcile their shared transaction history.

Ledger Challenge-Response
An authentication mechanism where an agent proves its identity by correctly answering questions about shared transaction history.
Example: "What was the last transaction we recorded?" / "Who picked whom up from the airport last Tuesday?"

Distributed Ledger (Lightweight)
The AEX approach to ledger redundancy where witness agents maintain encrypted copies without requiring full blockchain consensus.
________________________________________

## Coordination Concepts

Scheduling Flexibility
A trust parameter defining how easily an agent can reschedule commitments on behalf of its human. Higher trust enables more flexibility.

Relationship Hierarchy
The prioritization of commitments (family > work > social) that agents negotiate and respect when coordinating schedules.

Escalation
The process by which agents defer to their humans when they cannot reach autonomous agreement or encounter novel situations.

Graceful Degradation
The principle that agents should escalate to humans rather than fail or proceed with uncertainty when facing complex decisions.
________________________________________

## Network and Infrastructure

Agent Network
The distributed set of all personal agents using the AEX protocol, forming a social graph that mirrors human relationships.

Trust Graph
The network topology formed by trust relationships between agents, determining witness selection and reputation propagation.

Second-Degree Trust
Trust relationships between agents that don't directly know each other but share mutual trusted connections.
Example: Bob's agent doesn't know Charlie's agent, but both trust Alice's agent.

Settlement Rail
The underlying payment infrastructure used to transfer value (ACH, card networks, Venmo, crypto, stablecoins).
________________________________________

## Social and Economic

Relationship Velocity
The rate at which value and coordination flow through an agent relationship. Higher trust enables higher velocity.

Trust Decay
The gradual reduction in trust level due to payment delays, flakiness, disputes, or inactivity.

Trust Amplification
The increase in trust limits and permissions resulting from consistent positive interaction history.

Social Balance Sheet
The hypothetical visibility of who owes what to whom across a social network. AEX defaults to privacy but the concept raises important social and political questions.

Disintermediation
The removal of traditional financial intermediaries (banks, payment processors, credit card companies) from everyday transactions as value flows through agent IOUs.
________________________________________

## Security and Privacy

Impersonation Attack
An attempt by a malicious actor to create a fake agent claiming to be someone's legitimate agent.

Sybil Attack
An attempt to create multiple fake agent identities to manipulate trust networks or accumulate fraudulent trust.

Replay Attack
An attempt to resubmit or duplicate a legitimate transaction to fraudulently increase balances or obligations.

Trust Laundering
The practice of abandoning a compromised or low-reputation agent identity and creating a new one to escape negative history.

Privacy by Default
The design principle that transaction details, relationship graphs, and reputation data should only be shared when explicitly necessary and consented to.
________________________________________
## Use Case Terminology

Family Agent Group
A set of agents representing household members (parents, children, nanny, extended family) that coordinate schedules, transportation, and obligations.

Carpool Coordination
Agent-to-agent negotiation of transportation duties, schedules, and cost-splitting for recurring shared trips.

Class Agent
An agent representing a group context (classroom, sports team, club) that coordinates activities, tracks shared expenses, and manages collective obligations.

Commercial Agent
An agent representing a business entity (retailer, service provider) that interacts with personal agents for transactions, authentication, and service delivery.
________________________________________

## Protocol Evolution

v1.0 Features
The initial AEX protocol version assuming single agent identity per human, human-initiated trust, and basic IOU tracking.

Multi-Agent Delegation (Potential v2.0)
Future capability allowing one human to operate multiple specialized agents (Social Agent, Financial Agent, etc.) with context-specific permissions.

Capability-Bound Sub-Keys
Cryptographic keys with limited permissions granted to specialized agents within a multi-agent hierarchy.
________________________________________
## Related Protocols and Standards

A2A (Agent2Agent Protocol)
Google's protocol for enterprise agent communication, focused on business workflows.

AP2 (Agent Payments Protocol)
Google's protocol for agent payments to commercial entities, supporting both fiat and crypto.

MCP (Model Context Protocol)
Anthropic's protocol for agents accessing tools and external resources.

ERC-8004
Ethereum standard for agent discovery, verification, and transaction on blockchain networks.
________________________________________
## Slang and Informal Usage

"I gotcha" / "I'll getya"
The social exchange underlying the $GOTCHA economy. "I gotcha this time" = I'll pay now. "I'll getya next time" = You owe me one.

"Hit up"
To initiate a relationship between agents via HIT request.
Example: "Hit up Sarah's agent to coordinate the camping trip."

"Agent good for it"
Informal expression meaning an agent's human has sufficient trust history or balance to honor an IOU.
Example: "Bob's agent is good for it, we go to lunch every week."

"Escalate"
When agents defer a decision to their humans.
Example: "The agents couldn't agree on a restaurant so they escalated."
________________________________________

## Common Abbreviations

•	AEX - Agent Exchange
•	HIT - Human-Initiated Trust
•	IOU - I Owe You
•	A2A - Agent-to-Agent (or Agent2Agent Protocol)
•	AP2 - Agent Payments Protocol
•	MCP - Model Context Protocol
•	KYC - Know Your Customer
•	AML - Anti-Money Laundering

________________________________________
Note: This glossary represents concepts and terminology from AEX Protocol v0.1 (January 2026). As the protocol evolves, new terms may be added and definitions may be refined based on implementation experience and community feedback.
