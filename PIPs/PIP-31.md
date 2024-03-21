| PIP | Title          | Description                | Author                                                                    | Discussion                                                               | Status      | Type                                     | Date                  |
|-----|----------------|----------------------------|---------------------------------------------------------------------------|--------------------------------------------------------------------------|-------------|------------------------------------------|-----------------------|
| 31 | Cancun EIPs | Proposes the Polygon Cancun EIPs changes | [Arpit Temani](https://github.com/temaniarpit27) | [Forum]() | Final | Core | 2024-01-11
---

### Abstract

This proposal specifies the activation of Cancun features (introduced by Ethereum - L1) on Polygon PoS network (L2). This will enable a set of functionalities which are scheduled to go live in Q1 '24 on Ethereum.

### Motivation
As [bor](https://github.com/maticnetwork/bor) is a fork of [geth](https://github.com/ethereum/go-ethereum), the practice of merging upstream changes gives Polygon the opportunity to enable new features defined in the L1.
With the this PIP, we plan to upgrade our network with a subset of EIPs included in Ethereum.

### Specification

The Cancun PIP will include the following EIPs:
1. [EIP-1153: Transient Storage Opcodes](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1153.md)
2. [EIP-5656: MCOPY - Memory copying instruction](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-5656.md)
3. [EIP-6780: SELFDESTRUCT only in same transaction](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-6780.md)

Compared to Ethereum Cancun upgrade, the following EIPs will not be included in Polygon specs:
1. [EIP-4788: Beacon block root in the EVM](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-4788.md)
2. [EIP-4844: Shard Blob Transactions](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-4844.md)
3. [EIP-7516: BLOBBASEFEE opcode](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7516.md)

The aforementioned EIPs has been specifically excluded because Polygon PoS has no concept of beacon chain and blobs.

### Backwards Compatibility

This PIP will not be backward compatible with the current implementation of bor and will therefore require a hard fork.
