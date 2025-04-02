---
PIP: 62
Title: Heimdall v2 Migration
Author: Marcello Ardizzone (@marcello33), Harry Rook (@hrook1)
Description: Proposes Heimdall v.1--> v.2 Migration
Discussion: 
Status: Draft
Type: Core
Date: 2025-04-02
---

### Abstract

This proposal specifies deprecating Heimdall v.1 and activating Heimdall v.2 features on the Polygon PoS network.

### Motivation

This proposal specifies the migration to Heimdall v.2, which refactors Heimdall to target Cosmos-sdk version `v0.50.x` and replaces Tendermint with CometBFT `v0.38.x`.

### Specification

The migration will enable the following upgrades:

 * [PIP-43: Replace Tendermint with CometBFT in Heimdall](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-43.md)
 * [PIP-44: Upgrade cosmos-sdk in Heimdall](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-44.md)

### Activation

- Activation block (`halt_height`):
  * For Amoy - `TBC`
  * For Mainnet - `TBC`

### Backward Compatibility

This upgrade is not backward compatible with the current implementation of Heimdall and will require migration to a completely new chain. Due to the upgrade from Tendermint to CometBFT, the block history will not be preserved. Therefore, in case RPC providers are willing to keep that data, node operators would need to keep running Heimdall v1 service, which would only serve as a source of data for all the blocks previous to the `halt_height` before the migration takes place (in a read-only mode). When it comes to application data, it will be preserved by performing the migration itself, during which data will be exported from v1, and injected at genesis into Heimdall v.2, after proper modifications. This means that information such as checkpoints, milestones and state-syncs are preserved within Heimdall-v2.

### Security Considerations

Downtime is expected while migrating from v1 to v2. Execution clients will be made independent of Heimdall v.1, so the side chains will continue to work as expected, but bridge operations (and everything concerning cross-chain data between L1 and L2) will be paused. The migration expected time will be driven by the time taken from Heimdall v.1 to export its genesis.

### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.


