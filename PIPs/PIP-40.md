---
PIP: 40
Title: Support for Community Treasury Contracts
Description: Enables support for the new Community Treasury contracts
Author: Simon Dosch, Mateusz Rzeszowski 
Discussion: https://forum.polygon.technology/t/pip-40-support-for-community-treasury-contracts/17641
Status: Last Call
Type: Contracts
Date: 2024-06-24
---

## Abstract

The POL [Default Emission Manager](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md#:~:text=%3B%0A%0A///%20%40title-,Default%20Emission%20Manager,-///%20%40author%20Polygon%20Labs) specifies that a 1% annual compounding emission is directed to the Community Treasury.

Upon the deployment of POL contracts introduced in [PIP-17](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md), a multi-signature wallet (`0x2ff25495d77f380d5F65B95F103181aE8b1cf898`) was used to safeguard the funds until Community Board supervision is enacted.

[PFP-2](https://github.com/0xPolygon/Polygon-Funding-Proposals/blob/main/PFPs/PFP-02.md) now specifies the proposed implementation for the Community Treasury Board, including the use of Governor Bravo contracts to govern the Community Treasury funding allocation process.

This proposal will require a migration away from the current setup and, therefore, an upgrade to the Default Emission Manager via the Protocol Council.

## Specification

We propose to upgrade the Default Emission Manager contract treasury address to the new Community Treasury Contract (`0x86380e136a3aad5677a210ad02713694c4e6a5b9`).

The particulars of the new Community Treasury Board governance can be found in [PFP-2](https://github.com/0xPolygon/Polygon-Funding-Proposals/blob/main/PFPs/PFP-02.md), with the key technical details shown below.


>A new governance contract (Aragon) is proposed that will govern the Community Treasury.

>A non-upgradeable OpenZeppelin ERC20Votes will be used as a voting token held by two SAFE contracts. Transfers, Approvals and Delegation will be disabled and tokens will be distributed upon contract deployment. Two roles are furthermore distinguished, Proposer and Executor.

>* 50.00000001 tokens will be distributed to the Proposer SAFE (`0x2ff25495d77f380d5F65B95F103181aE8b1cf898`) wallet operated by Polygon Labs,
>* 49.99999999 tokens will be distributed to the Executor ⅗ consensus multisig wallet (`0xb7b02DbC9D054A8BA90b2172B4f8d2D79aC7d3a0`) operated by the Community Treasury Board.

>The Proposer is responsible for proposal creation, with 50 votes being the requirement.

>Once a proposal has been created, both the PROPOSER and EXECUTOR's yes votes are necessary to execute a proposal, with 100 votes being the required consensus.

>The above setup ensures that while Polygon Labs may assist the Community Treasury Board with day-to-day administrative and other non-managerial operations, it may not execute any transactions, including the distribution of funds, without the CTB's on-chain approval.

## Backward Compatibility

This change is not backwards compatible with the current implementation of the Default Emissions Manger and will require a contract upgrade.

## Security Considerations

The updated governing contracts for the treasury address utilize Aragon, which has been extensively tested and audited.

## Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
