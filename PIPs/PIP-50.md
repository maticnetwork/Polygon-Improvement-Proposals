---
PIP: 50
Title: Staked Tokenholder Signalling
Description: Tokenholder signalling for system smart contracts governance
Author: Carlos Gonzalez Juarez, Tanisha Katara, Mateusz Rzeszowski
Discussion: https://forum.polygon.technology/t/pip-50-staked-tokenholder-signalling/19974
Status: Continuous
Type:  Informational
Date: 2024-11-8
---

## Abstract

This proposal introduces an initial decision-making framework for PoS Stakers to participate in signaling on PIPs related to system smart contract upgrades in Polygon networks. The framework enables a three-stage process for all regular proposals: Firstly, following the regular PIP flow which includes the PPGC, a majority consensus from the [Protocol Council](https://polygon.technology/blog/meet-the-polygon-protocol-council) is reached. Secondly, the community of PoS stakers and their delegates review the proposal and may signal vote approval or veto. Finally, the Protocol Council must confirm the proposal for on-chain execution or decide not to execute it. This three-stage governance framework, powered by the Governance Hub, lays the foundation for a transparent decision-making process over Polygon networks.

## Motivation

The primary motivation behind this proposal is to enhance the decentralization of community governance over L1 smart contract upgrades by enabling staked tokenholders to participate in governance through an off-chain signaling mechanism.

## Definitions

- **Delegates**: Individuals or entities to whom staked tokenholders can delegate their voting power.

- **Delegation**: The ability for staked tokenholders to delegate a part or the entirety of their voting power to any number of selected delegates.

- **Adaptive Quorum**: A voting mechanism where the required veto majority adjusts dynamically based on participation levels.

- **Snapshot**: A mechanism that captures the state of staked token holdings at a specific block for calculation of voting power.

## Specification

### Delegations and Partial Delegations:

The delegation system is built using a combination of on-chain contracts and a Snapshot strategy to ensure flexibility and accuracy in voting power (VP) calculations. PoS stakers can choose to delegate part or all of their voting power to multiple delegates. The key components are outlined below:

* **On-Chain Contracts**:

  * Polygon Staking Manager Contract: This contract aggregates the voting power (VP) from all the validators, with whom tokenholders have staked their POL tokens, and returns it so it can be queried over in Snapshot. There’s no execution available in this contract and can only be queried externally. Additionally, even though it’s not upgradeable, since it’s only used for reading off-chain, the Snapshot strategy can be upgraded.
  * Split Delegation Contract: This contract allows staked tokenholders to delegate their voting power to multiple delegates in varying proportions, supporting partial delegations. Delegation events are emitted on-chain and indexed off-chain via Subgraph.
  * Polygon Delegate Announcer Contract: This contract allows for users to explicitly mark themselves as community delegates. The reasoning behind this is to avoid users receiving delegations without actually wanting them, as has happened before in other organizations. All of the metadata for this contract is stored in IPFS, available to anyone.

* **Subgraph**:

  * The Graph plays a critical role in managing and tracking delegation events. It captures all delegation actions from the Split Delegation Contract. It then processes these events via Subgraph, and provides the data necessary for calculating the final voting power of each delegate.

* **Snapshot Strategy**:

  * The Snapshot strategy wraps around the existing polygon-self-staked-pol strategy with the added support of the split-delegation strategy. This ensures that delegation ratios and self-staked voting power are both considered when determining the final voting power for each delegate. The Snapshot system takes a point-in-time record of the delegation state, allowing accurate off-chain calculations and signaling.

* **GovHub API and Delegates Webpage**:

  * The Polygon GovHub API communicates with the Subgraph to retrieve data about delegators and their respective delegation ratios. This information is then fed into the Delegates Webpage, where delegators can view and manage their delegations. The webpage displays the voting power (VP) for each delegate and ensures transparency throughout the governance process.

### Adaptive Quorum:

An adaptive quorum mechanism plays a crucial role in the community signaling stage of the governance process. During this stage, the broader community of PoS stakers and delegates can signal veto or approval on proposals. A dynamic quorum ensures that the veto power can still be exercised even when the number of actively voting tokens is lower than expected, providing an extra layer of security and decentralization, without compromising on efficient decision-making.

For the proposal to be vetoed, **the ratio of nay votes to PoS-staked tokens** must be **greater than the ratio of yay votes to actively voting tokens**. This adaptive approach ensures that the level of participation—whether high or low—can still lead to a valid decision, as long as the condition is met.

In the adaptive quorum mechanism, a multiplier will be applied to the "Yay votes”, based on rigorous testing and continuous voting participation data.

Importantly, while Snapshot strategies do not support this mechanism, the Governance Hub allows for this information to be displayed to the users transparently in the UI. To do so, the voting terminal provides not only messages, but also a graph of the growing number of votes, giving users clarity on how the ongoing vote is going.

## Backward Compatibility

The Governance Hub is built and will be expanded based on careful iteration. For that reason, all of the existing and proven processes previously built for PIPs still apply and have been included in the Governance Hub. For instance, every proposal created in the PIP repository will appear in the Governance Hub published under the name of the person who wrote it. As a result, all of the contributors Polygon has had in the past, are highlighted. Similar backward-compatible mechanisms have been adopted to cover all parameters of the PIP framework, e.g., PIP flows and statuses.

## Security Considerations

Iteration is one of the key characteristics of this project, taking into account security considerations, as well as steadily iterating on previous standards. For that reason, on-chain components have been reduced to a minimum, and even the existing ones are being used solely for off-chain actions, e.g., Snapshot voting. If an issue were to be identified, all off-chain components can be updated and also may point to a new contract fixing the issue. The goal with this approach is to slowly adapt towards future versions based on past learnings.

## Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
