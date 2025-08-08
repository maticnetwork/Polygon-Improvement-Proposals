---
PIP: 43
Title: Replace Tendermint with CometBFT in Heimdall
Description: Proposes an upgrade for heimdall by replacing tendermint with cometBFT
Author: Marcello Ardizzone (@marcello33), Sergio Mena (@sergio-mena), Greg Szabo (@greg-szabo)
Discussion: https://forum.polygon.technology/t/pip-43-replace-tendermint-with-cometbft-in-heimdall/17731
Status: Final
Type: Core
Date: 2023-10-17
---

### Abstract

Heimdall, a crucial component of Polygon’s Proof-of-Stake (PoS) architecture, currently relies on `Peppermint`, a custom fork of Tendermint `v0.32.7`, to handle consensus. However, this implementation is now outdated and creates technical debt. This proposal recommends upgrading Heimdall’s consensus layer by replacing Tendermint with CometBFT `v0.38.x`, which brings modernized consensus features, including ABCI 2.0 (Application Blockchain Interface `v2.0`, codenamed ABCI++). This upgrade will streamline transaction processing, improve state synchronization, and allow more efficient handling of external data -- such as side transactions -- using new mechanisms like vote extensions.

### Motivation

The key motivation behind this upgrade is Heimdall's reliance on an outdated version of Tendermint, which limits scalability and requires ongoing maintenance of the `Peppermint` fork. Although Peppermint has served the network well, keeping it up to date with the latest Tendermint features requires substantial development resources. CometBFT offers a natural evolution by introducing better modularity, scalability, and flexibility, allowing Heimdall to maintain high performance while reducing tech debt.

Moreover, with CometBFT, Heimdall will benefit from new consensus features such as two-phase block processing, application-driven proposal handling and validation, and more advanced state sync capabilities. These features will allow Polygon PoS to handle higher throughput, improve node synchronization times, and more effectively manage data consistency across the network.

The fact that Peppermint's logic for side transactions can be fully mapped onto vote extensions provided by ABCI 2.0 will produce a much thinner fork of CometBFT, with a plan to shed the fork entirely in the next version.

### Rationale

The transition from Tendermint to CometBFT is not just a simple software upgrade; it involves leveraging the next generation of consensus architecture provided by CometBFT’s ABCI 2.0.

#### Architectural Overview:

**Current Architecture (Tendermint-based Peppermint):**

1. **Consensus Layer:** The current consensus mechanism follows the standard Tendermint design. It uses a linear process where blocks are proposed, validated, and finalized by validators in a single step, from the application's point of view. Each block contains transactions, which are processed by the application layer only after the block is committed. In Heimdall, external transactions (like state sync and checkpointing) are processed as side transactions, which have a special handling mechanism.
2. **Application Layer:** Heimdall handles Polygon-specific needs, such as checkpointing to Ethereum and validator management. These functionalities rely on side transactions, which embed data from external chains (like Bor or Ethereum) into the consensus mechanism.

**Upgraded Architecture (CometBFT with ABCI 2.0):**

CometBFT redefines the interaction between the consensus and application layers using ABCI 2.0. The new features allow more dynamic control over block creation, validation, and transaction processing.

1. **ABCI 2.0 (Application Blockchain Interface v2.0, a.k.a ABCI++):**
* **PrepareProposal:** The application can intervene before a block is proposed, optimizing the block for specific needs such as transaction order or removing unnecessary data. This enables a more efficient proposal process, reducing the overall block processing time, and detecting and excluding garbage transactions from the proposal.
* **ProcessProposal:** Validators can now pre-validate proposed blocks, ensuring that invalid blocks are rejected earlier in the consensus process. This provides an additional layer of security and helps prevent unnecessary delays caused by processing invalid data.
* **ExtendVote / VerifyVoteExtension:** Vote extensions allow validators to include additional, application-specific data (such as checkpoint hashes) in the block precommit phase. This replaces the side transaction system in Heimdall and allows the application layer to directly handle external data through consensus, providing a much tighter integration with the rest of the protocol. As a result, no bespoke code for dealing with side transactions is needed at consensus level.
2. **State Synchronization:** CometBFT enhances state sync mechanisms at the consensus level, enabling new validators or nodes to catch up with the current blockchain state more rapidly and efficiently. This is particularly useful for increasing the overall network resilience by enabling faster recovery times and reducing the time required to onboard new validators.
3. **Two-Phase Block Commit:** With ABCI 2.0, block creation and validation can happen in two distinct steps. First, blocks are proposed and partially validated (with PrepareProposal and ProcessProposal). Later, a final commit phase locks the state changes, making the system more resilient to issues like network latency or bad block proposals. This two-step process introduces flexibility for implementing optimizations like batch transaction processing or delayed execution.

### Specification

**ABCI 2.0 Overview:** CometBFT's ABCI 2.0 introduces an enhanced application-consensus interaction model, enabling more efficient block processing and better state synchronization. Here's how it breaks down:

1. **PrepareProposal and ProcessProposal:**
* **PrepareProposal**: This allows block proposers to optimize the block at the application level before it is proposed, e.g., by reordering transactions or removing unnecessary ones. This is a crucial performance optimization that wasn’t possible with Tendermint.
* **ProcessProposal**: This allows validators to pre-validate proposed blocks before committing to them. It adds an extra layer of security and efficiency in block validation.
2. **Vote Extensions:** A powerful new feature replacing Heimdall’s side transaction system, allowing validators to append arbitrary data to their precommit votes. This feature is particularly important for managing cross-chain data, such as Polygon’s checkpoints to Ethereum or state sync updates from the Bor chain. It offers a flexible mechanism for integrating external data without requiring customized side transaction handling at consensus level.
3. **State Sync Enhancements:** CometBFT provides an improved state sync protocol, allowing nodes to catch up with the network faster by using snapshots. This reduces the time and resources required to synchronize new nodes or validators, ultimately improving the resilience and scalability of the network.
4. **Backward Compatibility Considerations:** This upgrade requires migrating Heimdall to a new chain due to dependencies on a more recent version of the Cosmos SDK. This migration will involve coordinated updates across the Polygon ecosystem to ensure a smooth transition.

### Backwards Compatibility

The upgrade depends on a newer version of `cosmos-sdk`,. Additionally, recent versions of CometBFT (e.g. `v0.38.x`) contain changes to the block format, when compared to the current version of Peppermint. As a result, code is not backward compatible with the current Heimdall chain. This means the launch of `cometBFT` within Polygon PoS will only be possible by migrating Heimdall to a brand new chain.

### Security Considerations

This upgrade fundamentally alters the structure of how Heimdall interacts with its consensus engine, making security a top priority. Key areas of concern include the handling of cryptographic keys, the correct implementation of vote extensions, and the introduction of the two-phase commit mechanism. Thorough testing, security audits, and performance benchmarking will be required before deployment. The upgrade will initially follow a 1:1 migration path, maintaining existing functionality while gradually rolling out new features.

### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
