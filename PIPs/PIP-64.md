---
PIP: 64
Title: Validator-Elected Block Producer
Author: Jerry Chen (fchen@polygon.technology)
Description: Introduces Validator-Elected Block Producers (VEBloP) for Enhanced Throughput on the Polygon PoS network.
Discussion: https://forum.polygon.technology/t/pip-64-validator-elected-block-producer/20918
Status: Draft
Type: Core
Date: 2025-04-29
---


_Disclaimer: This document presents a high-level design for Validator-Elected Block Producers. While the architecture and motivations are well-scoped, certain implementation details, parameters, and operational procedures remain under active discussion and subject to refinement._

## Abstract

This proposal outlines a new block production architecture for the Polygon PoS chain, introducing a Validator-Elected Block Producer (VEBloP). By designating a single elected producer per span and implementing stateless block validation, this architecture aims to substantially increase network throughput (targeting 10,000 TPS and potentially higher), shorten block confirmation times, and eliminate reorgs, all while maintaining decentralized block verification.

## Motivation

The current Polygon PoS network has a theoretical throughput ceiling of around 714 transactions per second (TPS), limited by the 30M block gas limit and 2-second block time. Achieving significantly higher throughput, like 10,000 TPS or more, faces several architectural hurdles within the existing system:

1. **Block Propagation Latency:** With potentially over 100 validators producing blocks, rapid propagation across the network is essential for consensus. Attempting to decrease block times below 1 second to boost TPS significantly strains the network, leading to latency issues and mini-reorgs due to inconsistent block views.
2. **Execution Bottlenecks:** Executing a full 30M gas block requires considerable time (roughly 125ms on the [recommended hardware](https://docs.polygon.technology/pos/how-to/prerequisites/#mainnet-specs), 16 cores and 64GB RAM). When combined with validation and propagation overhead, this makes sub-second block times impractical under the current model.
3. **Lack of Single Slot Finality:** While [PIP-11](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-11.md) introduced faster finality, the current system doesn't guarantee single-slot finality. Small block ranges can still be reorganized after propagation, negatively impacting user experience and system predictability, especially with faster block intervals.
4. **Economic Unsustainability:** Scaling throughput increases hardware demands. For smaller validators, the rising costs may outweigh their diminishing share of transaction fees, potentially reducing validator participation and harming decentralization.
5. **Consensus Overhead:** Coordinating ~20 producers per span introduces contention and increases the likelihood of conflicting blocks when aiming for higher transaction inclusion rates.

Simply adjusting parameters like block time or gas limits is insufficient to reach 10k–100k TPS within the current decentralized production model due to the amount of state Polygon chain has; it's both technically challenging and economically unsustainable. This proposal tackles these challenges by separating block production from validation. A small group of high-capacity producers can build blocks quickly, while the broader validator set focuses on efficient, stateless verification. This allows for significant performance scaling without compromising the core trust assumptions of the consensus mechanism.

## Specification

### Overview of Architectural Changes

The proposed design fundamentally alters block production through several key changes:

- **Validator-elected Block Producer:** A single validator is elected as the exclusive block producer for a given span.
- **Backup Producers:** Designated backup producers stand ready to take over if the primary producer experiences downtime or misbehaves.
- **Stateless Validation:** Validators use witness proofs for lightweight, stateless block verification when settling milestones, reducing their hardware burden.
- **Decoupling Production and Validation:** Separating these roles enhances scalability while preserving trust assumptions.

These modifications aim to simplify block propagation, strengthen finality guarantees, and enable significantly higher throughput.

### Validator-elected Block Producer (VEBloP)

To address propagation delays, eliminate contention among producers, and achieve deterministic finality, this proposal introduces the concept of a single, validator-elected block producer (VEBloP) per span. Assigning block creation to one entity ensures blocks are produced predictably and rapidly, drastically reducing the potential for network-wide reorgs and enhancing overall liveness and reliability.

Under this model, spans become indefinite in length, and Heimdall is responsible for selecting the primary block producer for each span. Heimdall also designates backup producers who can step in if the primary producer goes offline, ensuring continuous block production. This contrasts with the current system where around 20 producers operate within a fixed-length span, any of whom can produce a block, leading to potential ordering conflicts and no instant finality. The new approach provides instant finality on transaction ordering and inclusion, though not yet on the block state itself, which will be later finalized by milestones.

| Feature                                   | Current System                 | Proposed VEBloP System             |
| ----------------------------------------- | ------------------------------ | ---------------------------------- |
| Producer Set Size per Span                | ~20                            | 1                                  |
| Block Producer Determination              | Any node from the producer set | Single node determined by Heimdall |
| Instant Txn Ordering & Inclusion Finality | No                             | Yes                                |
| Instant Block State Finality              | No                             | No                                 |

### Validator Rotation

A crucial aspect of the VEBloP model is ensuring seamless transitions between block producers to maintain network liveness, even if the primary producer fails or acts maliciously. This rotation mechanism prevents extended downtime or conflicting block proposals.

Heimdall monitors the network for missing blocks. If it detects that the primary producer is offline, it proposes a new span, designating a backup producer to take over. To prevent overlaps or reorgs during transitions, producer changes are carefully coordinated using time-based spans. Instead of defining spans by block numbers, new spans specify a future Unix timestamp after which the newly selected producer can begin creating blocks. This ensures all validators have sufficient time to recognize the span change before it takes effect, preventing inconsistencies. The block timestamps produced by the new producer must be strictly greater than the `startTimestamp` defined in the new span proposal. Backup producers can then take over smoothly, maintaining chain continuity without causing reorgs. Mechanisms for producers to signal their readiness to Heimdall are also part of this design.

### Forced Transactions

To guarantee censorship resistance, even with a single block producer who could potentially misbehave, a mechanism for forcing the inclusion of critical transactions is necessary. This ensures a deterministic pathway for both user and system-level transactions to be executed, upholding protocol guarantees.

This proposal leverages Polygon PoS's existing state sync mechanism in Bor. Users can submit transaction calldata and a target address to a designated L1 contract. This contract then triggers a state sync message directed to Bor. At the end of each sprint (currently every 16 blocks), the active block producer queries Heimdall for any pending state syncs. The producer must then include a transaction that calls a special L2 contract, which interprets the state sync data and executes the user's original transaction. Validators verify the correct inclusion and execution of these forced transactions as part of their standard consensus process. If a block producer fails to include a required forced transaction, validators will reject the block as invalid, which in turn triggers Heimdall to initiate a producer rotation.

### Stateless Verification

To make block validation efficient and scalable, particularly as throughput increases, this proposal incorporates stateless verification. This eliminates the need for validators to maintain and constantly update the full chain state, significantly lowering their hardware requirements and encouraging broader participation.

Stateless verification relies on cryptographic witnesses generated by the block producer (or any full node) during block construction, based on established techniques (e.g., [Execution layer cross-validation](https://gist.github.com/karalabe/47c906f0ab4fdc5b8b791b74f084e5f9)). These witnesses contain the necessary state information for a block to be verified without access to a full state database. Witnesses are computed and propagated across the network, potentially via a dedicated peer-to-peer protocol like the [Ethereum Witness Protocol (wit)](https://github.com/ethereum/devp2p/blob/master/caps/wit.md). Validators receive a block and its corresponding witness, allowing them to re-execute the block's transactions and confirm its validity statelessly. Validators only need to store recent verified blocks necessary for settling milestones and checkpoints. Strategies like witness caching (for both code and state) are being explored to minimize the data propagated per block.

### Economic Incentives

Given that this change will impact validator economics, in particular their receiving transaction fees and any out-of-band MEV fees, the team will continue analysis on this economic impact and adopting additional changes to maintain economic incentives to provide validation services. For example, an option would be to redistribute all transaction fees and MEV fees to validators based on their proportional stake, reserving a portion as commission to the selected block producer.

Due to its size and complexity, the detailed economic model is considered out of scope for this architectural specification and will be addressed in a separate proposal. Crucially, this proposal does not alter the existing staking rewards validators receive for participation in checkpoint proposing and signing on Heimdall and L1.

## Rationale

This design strikes a balance between performance and decentralization. Centralizing block production allows for lower latency and higher throughput by removing the bottleneck of coordinating many producers for each block. Simultaneously, maintaining decentralized verification through stateless validation ensures that trust assumptions are upheld, as a broad set of validators confirms the chain's integrity. This separation of concerns enables significant scaling while preserving the core security properties of the network.

## Backwards Compatibility

- Both Heimdall and Bor are required to implement the new time-based span structure, producer election/rotation logic, and stateless witness verification. A hardfork will be required when the new architecture is rolled out.
- Existing validators can choose to run stateless Bor clients, reducing their hardware needs without needing to operate full nodes.
- The processes for [checkpoints](https://docs.polygon.technology/pos/architecture/heimdall/checkpoints/) and milestones remain unchanged.

## Security Considerations

- **Network Trust:** The model assumes the block producer connects to at least some honest RPC nodes to prevent issues like block withholding.
- **Witness Integrity:** The security of stateless validation hinges on the integrity of witnesses. Validators must be able to reliably detect and reject invalid blocks based on incorrect or manipulated witnesses. Robust witness generation and propagation are critical.

## References

- [PIP-11: Milestone](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-11.md)
- [Ethereum Witness Protocol (wit)](https://github.com/ethereum/devp2p/blob/master/caps/wit.md)
- [Execution layer cross-validation (Gist by karalabe)](https://gist.github.com/karalabe/47c906f0ab4fdc5b8b791b74f084e5f9)
- [Polygon PoS State Sync](https://docs.polygon.technology/pos/architecture/bor/state-sync/)
- [Polygon PoS Checkpoints](https://docs.polygon.technology/pos/architecture/heimdall/checkpoints/)
