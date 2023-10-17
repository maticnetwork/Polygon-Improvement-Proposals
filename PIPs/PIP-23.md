

| PIP | Title                         | Description                                     | Author                                                                                                | Discussion | Status      | Type                                     | Date       |
|-----|-------------------------------|-------------------------------------------------|-------------------------------------------------------------------------------------------------------|------------|-------------|------------------------------------------|------------|
| 23  | Shanghai | Proposal to activate Ethereum Shanghai features | [Marcello Ardizzone](https://github.com/marcello33), [Sandeep Sreenath](https://github.com/ssandeep)  | [Forum](https://forum.polygon.technology/t/pip-23-proposal-to-activate-ethereum-shanghai-features/13065)  | Peer Review | Core | 2023-09-22 |

## Abstract

This proposal specifies the activation of Shanghai features (introduced by Ethereum - L1) on Polygon PoS network (L2). This will enable a set of functionalities which are live on Ethereum Mainnet since Beacon Chain Epoch `194048` (timestamp `1681338455`, UTC datetime `4/12/2023 10:27:35 PM`, fork hash `0xdce96c2d`).

## Motivation

As [`bor`](https://github.com/maticnetwork/bor) is a fork of [`geth`](https://github.com/ethereum/go-ethereum), the practice of merging upstream changes gives Polygon the opportunity to enable new features defined in the L1.  
With the Shanghai PIP, we plan to upgrade our network with a subset of EIPs included in Ethereum.

## Specification

The Shanghai PIP will enable the following upgrades:
* [EIP-3651: Warm COINBASE](https://eips.ethereum.org/EIPS/eip-3651)
* [EIP-3855: PUSH0 instruction](https://eips.ethereum.org/EIPS/eip-3855)
* [EIP-3860: Limit and meter initcode](https://eips.ethereum.org/EIPS/eip-3860)
* [EIP-6049: Deprecate SELFDESTRUCT](https://eips.ethereum.org/EIPS/eip-6049)

## Backwards Compatibility

This PIP will not be backward compatible with the current implementation of `bor` and will therefore require a hard fork.

## Reference Implementation

The following PRs implements the changes from upstream `go-ethereum` [v1.11.6](https://github.com/ethereum/go-ethereum/tree/v1.11.6), including the mentioned Shanghai related EIPs. 
- https://github.com/maticnetwork/bor/pull/901
- https://github.com/maticnetwork/bor/pull/1025

New changes to the code will be made once the block heights for the hard forks (on testnet and mainnet) are defined. 


## Copyright

All copyrights and related rights in this work are waived under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
