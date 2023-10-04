| PIP               | Title                           | Description          | Author                        | Discussion | Status | Type                                     | Date                  |  
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|  
| 24 | Change EIP-1559 Policy | Changes the EIP-1559 implementation to facilitate changes as introduced in PIP-18 | David Silverman, Paul Gebheim, Harry Rook, Mateusz Rzeszowski  | [Forum](https://forum.polygon.technology/t/pip-24-change-eip-1559-policy/13007)  | Draft | Core |2023-10-04

### Abstract

This proposal calls for a set of changes to the [EIP-1559](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md) burn system and constitutes a prerequisite for the implementation of Phase 0 - Frontier of Polygon 2.0, introduced in [PIP-18](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-18.md).  This document proposes changing the recipient of the EIP-1559 burn on the Polygon PoS network from 0x70bca57f4579f58670ab2d18ef16e02c17553c38 to (contract to be deployed) in order to prepare for updates to the Plasma bridges as part of upgrades proposed in [PIP-19](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-19.md).

### Definitions 

EIP-1559: A transaction pricing mechanism that includes fixed-per-block network fee that is burned and dynamically expands/contracts block sizes to deal with transient congestion.

Plasma Bridge: Polygon Plasma is the canonical bridge used to bridge MATIC tokens from Ethereum to Polygon chain. Withdrawals have a required 7-day delay. This delay is designed to allow users to challenge a malicious withdrawal.

LxLy Bridge: LxLy is the bridge that powers the Polygon zkEVM. It leverages zero knowledge validity proofs to secure all cross chain messages and asset transfers, including L2 to L2 transactions.

### Motivation

EIP-1559 is the mechanism by which the Polygon PoS network burns MATIC collected from the base fee paid by users transacting on the network. The present EIP-1559 implementation sends the baseFee   to an immutable contract at 0x70bca57f4579f58670ab2d18ef16e02c17553c38 which, using [EIP-1014: Skinny CREATE2](https://eips.ethereum.org/EIPS/eip-1014), allows users to send MATIC from Polygon PoS to the Plasma Bridge using the same contract address 0x70bca57f4579f58670ab2d18ef16e02c17553c38 on Ethereum. This contract in turn sends MATIC to a burn address (0x000000000000000000000000000000000000dEaD), therefore removing it permanently from the supply.

  
As part of the upcoming Polygon 2.0 series of proposed upgrades, changes to the PoS system will be required, which include updates to the bridge used by the current EIP-1559 contracts, the Plasma Bridge. These changes are specified in [PIP-19](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-19.md) and cause breaking changes for the EIP-1559 contracts. In advance of [PIP-19](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-19.md)’s implementation, the authors propose migrating the recipient of the EIP-1559 mechanism to (contract to be deployed). This new recipient is a temporary holder contract that will collect MATIC (or POL, should PIP-19 be implemented) to be burned. A future proposal can upgrade this contract to use a new transfer mechanism once the LxLy Bridge is deployed to PoS as part of the proposed Polygon 2.0 zkPoS conversion process.

### Specification

Change the recipient of EIP-1559 Burn by adding a new entry in burntContract with the hardfork block number and the new address in Bor’s Genesis file to (contract to be deployed).

### Backward Compatibility

This change is not backward compatible and will require a hard fork of Bor to redirect burned MATIC. The old EIP-1559 contract will still function and be able to burn its remaining MATIC, however, all new MATIC destined for burning will be sent to the new recipient.  

The return of tokens burned due to EIP-1559 to Ethereum, in order to facilitate an L1 burn, will require another future protocol change.

### Security Considerations

The temporary recipient contract will be assigned an owner, allowing it to be upgraded post-deployment of LxLy. This owner will be the Polygon Commitchain Multisig (0x355b8E02e7F5301E6fac9b7cAc1D6D9c86C0343f) that already secures large parts of the PoS system.

### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
