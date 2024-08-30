---
PIP: 46
Title: MATIC to POL Migration 
Description: Proposes the MATIC to POL migration
Author: Simon Dosch, Harry Rook
Discussion: https://forum.polygon.technology/t/pip-46-matic-to-pol-migration/19672
Status: Last Call
Type: Contracts
Date: 2024-08-21
---

### Abstract

This proposal specifies the activation of a number of changes to layer 1 contracts for the Polygon PoS chain, enabling the MATIC to POL migration. 

### Motivation

As specified in PIP-19, the POL token represents the next-generation native token of the Polygon PoS chain. This upgrade will combine a series of PIPs that enable the MATIC to POL migration in one transaction payload, which will ensure backward compatibility. 

### Specification

The migration will include the following PIPs:

* [PIP-19: Update Polygon PoS Native Token to POL](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-19.md)
* [PIP-41: Enable Direct POL Emissions to StakeManager.sol](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-41.md)
* [PIP-42: Polygon 2.0 - Upgrade PoS Staking to Use POL](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-42.md)

### Backwards Compatibility

This PIP will be backward compatible with the current implementation of the existing layer 1 contracts.

### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.


