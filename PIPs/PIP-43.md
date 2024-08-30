---
PIP: 43
Title: Replace Tendermint with CometBFT in Heimdall
Description: Proposes an upgrade for heimdall by replacing tendermint with cometBFT
Author: Marcello Ardizzone (@marcello33)
Discussion: https://forum.polygon.technology/t/pip-43-replace-tendermint-with-cometbft-in-heimdall/17731
Status: Draft
Type: Core
Date: 2023-10-17
---

### Abstract

Currently, Heimdall uses a fork of `tendermint` (called [`peppermint`](https://github.com/maticnetwork/tendermint)). The fork, which has been modified to support Polygon PoS specific needs, is based off version `v0.32.7`.
The refactoring required replaces `peppermint` with the newer library called `cometBFT`, which will target version `v0.38.x`.

### Motivation

The primary motivation for this proposed upgrade is to remove tech debt that anchors `Heimdall` to a `Tendermint` version that was released around four years ago and is outdated. Despite the continuous upgrades made throughout these years to `Peppermint`, the upgrade to `CometBFT` will be beneficial for the PoS network for many technical factors.

### Rationale

`cometBFT` will come with several new features. The main amongst them include:
1. A new application blockchain interface, called ABCI++, that introduces new methods and types, more flexibility in transactions execution, and a two-phase commit process. Moreover, the blocks execution process is more flexible compared to the linear process of ABCI. Amongst the others, it also introduces advanced functionalities in term of state sync capabilities, allowing for faster and more efficient synchronization.
2. ABCI++ also comes with a brand new feature, called Vote Extensions. Such functionality will replace the concept of Side Transactions in Heimdall, which was the main driver for forking `tendermint`. Side transactions are Heimdall transactions whose data comes from external sources (eg: `checkpoint` txs contain root hash of Bor chain blocks, `stateSync` txs contain event data from L1 Ethereum). Hence, the data in side transactions needs to be verified by the validators on Heimdall. A `VoteExtension` is just an arbitrary sequence of bytes that gets added to the block precommit. This means that it is the responsibility of the ABCI++ application (Heimdall in this case) to define and interpret it. In short, vote extensions are basically a device with which validators can vote on data that is extrinsic and agnostic to the consensus protocol.
3. The upgrade will target a fork of `cometBFT` which contains the `peppermint` changes on top of `cometBFT v0.38.x`. This will allow more frequent upstream merges, which means more functionalities to leverage and a higher level of security.
   
*A comprehensive overview of the new features included within CometBFT can be found under references.*

### Specification

The proposed solution - which is dependent on other upgrades (such as upgrading `cosmos-sdk` dependency in Heimdall) - involves several steps to implement:
1. Port the Polygon PoS changes from `peppermint` to `cometBFT`
2. Upgrade of Heimdall application to a newer version of `cosmos-sdk` (`v0.50.x`)
3. Release of a `beta` version
4. Embed the `cometBFT` into the newer Heimdall
5. Test the standalone application locally
6. Audit the code
7. Possibly rework the code, based on audit results
8. Release of a `stable` tag for the Polygon PoS fork of `cometBFT`


### Backwards Compatibility

The upgrade depends on a newer version of `cosmos-sdk`, which makes the code non-backward compatible with the current Heimdall chain. This means the launch of `cometBFT` within Polygon PoS will only be possible by migrating Heimdall to a brand new chain.

### Security considerations

The upgrade involves a complete refactoring of the Heimdall codebase, including changes that handle cryptographic keys. For these reasons, Heimdall will undergo a meticulous phase of tests, audits, and performance reviews. Despite the many new functionalities offered by the new library, this migration will be —at first— a 1:1 migration, keeping intact the Heimdall capabilities and utilities.


### References

- [CometBFT Repository](https://github.com/cometbft/cometbft)

### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
