---
PIP: 18
Title: Polygon 2.0 Phase 0 - Frontier
Description: Specification of Polygon 2.0 - Phase 0
Author: David Schwartz, Mihailo Bjelic, Mudit Gupta, Grace Torrellas, Mateusz Rzeszowski (@matrzeszowski), Daniel Gretzke, David Silverman, Paul Gebheim
Discussion: https://forum.polygon.technology/t/pip-18-polygon-2-0-phase-0-frontier/12913
Status: Peer Review
Type: Informational
Date: 2023-09-14
---

### Abstract

This proposal specifies Phase 0 of Polygon 2.0, a multi-phased upgrade to the Polygon Ecosystem. In Phase 0, the functioning of Polygon PoS and Polygon zkEVM are entirely unaffected for end users. Multiple changes are proposed to be implemented in the system smart contracts of these chains on Ethereum. In an homage to the original Ethereum release, we propose calling this initial phase of changes, “Frontier”. 

The core of the proposed changes are:

1.  The initiation of the POL upgrade
2.  The upgrade from MATIC to POL as the native (gas) token for PoS
3.  The adoption of POL as the staking token for Polygon PoS
4.  The launch of the Staking Layer and migration of Polygon public chains to consequently leverage it
    
We propose a high-level outline for these changes, which should be deployed incrementally, with further detail on each coming through separate proposals to be published for community review.

### Motivation

Polygon 2.0 envisions a network of interconnected ZK-powered L2 chains that, on aggregate, expand Ethereum blockspace and create the Value Layer of the Internet. This environment is seamlessly interoperable, offering **access to unified blockspace** across all Polygon chains as well as infinite scalability. 

**The end goal is to extend Ethereum blockspace in a fashion that more resembles a mesh network topology like the internet**. With ZK technology, Ethereum blockspace can scale to the size of the Internet for the first time in blockchain history.

From the start, the Polygon ecosystem has been committed to sustainably scaling Ethereum, bringing the idea of credibly neutral, decentralized blockchains to developers of all skill levels, while also ensuring that these blockchains remain cheap and accessible for all users. 

This proposal represents the next step in this journey, defining the initial tasks to bring the Polygon 2.0 vision to life.

### Specification

Phase 0 is specifically designed to ensure that there will be no required changes for end users and developers already on the PoS or zkEVM chains. All proposed adjustments are targeted toward the various contracts and systems controlling the Polygon 1.0 ecosystem on Ethereum. 

The following milestones will make up Phase 0:

#### POL Upgrade

-   Initiate upgrade of POL, a technical upgrade to MATIC, designed to power the Polygon 2.0 ecosystem.
-   Adopt POL as the native (gas) token for Polygon PoS. This will be accomplished by an upgrade to the Plasma Bridge, changing the MATIC withdrawal logic to only output POL.
-   Adopt POL as the staking token for Polygon PoS.
-   Upgrade the Polygon PoS validator rewards from MATIC to POL.
  
#### Staking Layer

-   Launch the Staking Layer, a series of smart contracts that enable validators in the Polygon ecosystem to use their stake to participate in multiple services across the Polygon 2.0 ecosystem such as Validation, Sequencing, Proving, and Data Availability services for a variety of chains. 
-   Migrate Polygon PoS staking and validation to a new service on the Staking Layer.
-   Migrate validator rewards from the legacy Polygon PoS staking contract to the new PoS Staking Layer service.

![](https://lh4.googleusercontent.com/U1IpTIuLkykLmYkvot6IBNdO1NY6h3Kj_mQA1rh4dLfhKbmYI_Ml-aeeEk2kcFsJUpXrgRrs114OY6s_-6v7O15u71qin6g2juF_EFLPuZZtFj9dzo-bheH_k4zmq6kwSqhtBhLmzKBM-b5PlPVwEb4)

### Backward Compatibility

This proposal seeks to lay out a roadmap that includes backwards incompatibility through the introduction, deprecation, and modification of Polygon system smart contracts on the Ethereum mainnet. As a result, this proposal covers a proposed order of operations and directly informs the creation of future PIPs to implement the functionality laid out in the “Specification” section. Relevant parties should carefully consider the changes laid out in this post, and monitor the eventual PIPs detailing them further to be prepared for any incompatibilities that will be introduced.

### Security Considerations

Polygon 2.0 is a significant technical upgrade of the Polygon system with broadscale changes to the PoS network, as well as the introduction of several novel systems. There are inherent risks in this migration, however, each component will undergo extensive testing, auditing, and public scrutiny prior to activation.  The staging of the PIPs and the methodical, piecemeal upgrade of the system is intended to ensure there is sufficient time for testing, auditing and scrutiny. 

### References

-   [PIP-17: Polygon Ecosystem Token (POL)](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md)
-   [PIP-19: Update Polygon PoS Native Token to POL](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-19.md)

### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.

