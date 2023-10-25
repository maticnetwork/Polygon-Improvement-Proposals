 | PIP | Title          | Description                | Author                        | Discussion | Status | Type                                     | Date                  |
|-----|----------------|----------------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|
| 29  | Polygon Protocol Council | Proposes a Protocol Council for Polygon 2.0 | Mihailo Bjelic, Mateusz Rzeszowski | [Forum](https://forum.polygon.technology/t/pip-29-polygon-protocol-council/13075) | Last Call  | Contracts | 2023-10-18

## Abstract

This proposal introduces the Protocol Council governance body responsible for performing regular and emergency upgrades to system smart contracts, i.e. components of Polygon protocols implemented in the form of smart contracts on Ethereum.

## Motivation

A need for security, as well as for the ability to seamlessly move to an improved version under specific circumstances, dictates the requirement for upgradeability as it relates to core pieces of Polygon 2.0 architecture implemented as L1 smart contracts. 

The Protocol Council – 13 publicly-named members jointly acting to execute changes on Polygon infrastructure – answers the need for security and upgradeability. 

As representatives of the community, the Protocol Council's main objective is to perform regular and emergency changes with varying timelocks and internal consensus requirements to the Polygon 2.0 components. 

The Protocol Council is an initial step towards the final result of Polygon 2.0 governance – an on-chain, trust-minimized, and community-based framework for efficient and decentralized decision-making, which will be formalized in future proposals. 

## Specification

### Multisig Contract 
 
It is proposed for the contract to be a [Gnosis Safe](https://github.com/safe-global/safe-contracts). 

### Ownership

It is proposed that the Protocol Council will represent the community in governing all future Polygon 2.0 system smart contracts. 

Initially, the Council will have the ability to make emergency and regular track changes to the following contracts:

-   [POL Migration Contract](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md#migration-contract) 
-   [Emission Manager Contract](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md#emission-manager-contract) 

### Timelock 

The above contracts will be changeable by use of one of the two routes:  
  
-   a regular change route requiring 7/13 consensus of the Protocol Council with a 10-day timelock, which will be used for all configuration changes in the contract;
-   an emergency change route requiring 10/13 consensus of the Protocol Council with no timelock, which will be used to upgrade contract implementations in the proxy contracts.

### Proposed Signers: 

-   Jordi Baylina (Polygon Labs)
-   Viktor Bunin (Coinbase)
-   Mariano Conti (independent)
-   Justin Drake (Ethereum Foundation)
-   Gauntlet
-   Mudit Gupta (Polygon Labs)
-   L2Beat
-   Zaki Manian (Sommelier Finance)
-   Anthony Sassano (The Daily Gwei)
-   Liz Steininger (Least Authority)
-   Jerome de Tychey (EthCC)
-   Mehdi Zerouali (Sigma Prime)
-   ZachXBT (independent)

## Rationale

### Principles for Member Selection

The proposed Protocol Council list of members comprises thirteen community representatives of which the selection process has been guided by factors listed below. 

#### Ecosystem Reputation

Members of the Protocol Council should be value-aligned with Ethereum, the L2 ecosystem, and the wider Web3 ethos, having historically proven themselves as engaged and committed members of the relevant communities. 
  
#### Resilient Nature of the Protocol Council 

The composition of the Protocol Council should ensure (operational) resilience, including by means of jurisdictional diversity, organizational diversity, and identification diversity, e.g., including public persons, anonymous individuals, and companies.

#### Self-limitation of Influence by Polygon Labs 

No entity should hold wide sway over the decision-making of the Protocol Council, and this is mainly expressed in the list of proposed decision-makers, including limiting Polygon Labs’ representation on the Council.

#### Technical and Governance Ability

Protocol Council members should demonstrate a level of technical, security, and governance competence, necessary for performing their duties and relevant to ensuring they’re acting in the community's best interest. This competence can be quantified by looking at member’s past participation in Web3 governance frameworks, security Councils, and technical involvement in the L2 space, among other experiences.

The Protocol Council will be able to add or remove members and otherwise determine its affairs among the members, governed by the PIP framework.

### Principles for Protocol Council Architecture

In order to optimize for both security and efficiency, a dual-route approach is introduced.

Regular, i.e., non-emergency, changes to the contracts are facilitated by a 7/13 (55%) consensus to maximize efficiency. These types of changes require a timelock delay of 10 days to ensure the ability for the community to exit the system before any change takes place. 

At the same time, an emergency route can facilitate immediate changes to system smart contracts in case of a critical issue. A timelock mechanism proves inefficient in such scenarios,  as it’s unable to ensure any potential issue remains contained and is addressed immediately without opportunities for outside interference, due to the public nature of timelocked transactions. Consequently, an emergency route for changing system smart contracts is introduced, requiring a very high 10/13 consensus of the Protocol Council to execute any change without a timelock. The security of this approach comes from a two-fold consideration:

-   What’s the flat number of compromised actors necessary to perform a malicious change? (10)
-   Of the overall actor set, what’s the percentage of compromised actors necessary to perform a malicious change? (77%)

As a result, any immediately-executable malicious change would require a large number (10) of  hypothetically-compromised actors, who at the same time make up the overwhelming majority (77%) of the Protocol Council set, leading to the conclusion that a successful attack is improbable.

While in the initial implementation POL contract changes will be governed by either one of the execution routes (regular or emergency), a future PIP will introduce dual route execution allowing the regular change route to be the main execution route, with the emergency route being enabled for a select number of system contract changes. 

## Backward Compatibility

This change causes no identifiable backward incompatibilities. 

## Security Considerations

Polygon 2.0 is a significant technical upgrade of the Polygon system with broadscale changes to the PoS network, as well as the introduction of several novel systems. There are inherent risks in this migration, however, each component will undergo extensive testing, auditing, and public scrutiny prior to activation. The staging of the PIPs and the methodical, piecemeal upgrade of the system are intended to ensure there is sufficient time for testing, auditing, and scrutiny. 

## Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
