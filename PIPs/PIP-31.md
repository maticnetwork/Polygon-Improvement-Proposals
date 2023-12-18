
| PIP               | Title                           | Description          | Author                        | Discussion | Status | Type                                     | Date                  |
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|
| 31 | Ancient data pruning | Ability to prune ancient blockchain data for PoS | [Manav Darji](https://github.com/manav2401/) | TBD  | Draft  | Core | 2023-12-18 |

## Abstract

A full node of the primary client of PoS, [bor](https://github.com/maticnetwork/bor) stores data in 2 different databases.
1. LevelDB (A key-value store)
2. AncientDB

The recent most data of past `90000` blocks (a configurable limit) is stored in leveldb for faster read/write on the recent most block data and state. While the data beyond this limit is moved to the ancient database which stores the historical data. For a full node, it stores block headers, body and receipts for fulfilling RPC requests over them. Currently as bor is derived from geth, it supports state pruning which basically prunes the old state present in the key-value store which keeps getting accumulated as the chain progresses. This doesn’t impacts the working of the chain in any way except reducing disk space as we’re just removing the state data which is not required.

## Motivation

Currently, the disk size of the bor database on mainnet and testet are as follows:
1. Mainnet (TODO)
2. Testnet (TODO)

As you see, the ancient data accounts for x% and y% (TODO) in mainnet and testet respectively.

While this data is required for nodes to serve RPC requests on historical blocks, it is not a necessity for a node to store this data if it's purpose is to produce/validate a block and participate in consensus (i.e. a validator node) or a regular sentry node. Currently, there's no way to prune the `ancient` database from a full node. The data keeps getting accumulated as the chain progresses and is bound to increase. This PIP proposes the node to have ability to prune ancient data reducing the overall disk space consumption.

## Specification

This PIP proposes to introduce a new cli command for pruning the ancient data as follows:

```
Usage: bor snapshot prune-block [options...]
  This command will prune ancient blockchain data at the given datadir location. Options:
  -datadir <value>                  Path of the data directory to store information
  -datadir.ancient <value>          Path of the old ancient data directory
  -block-amount-reserved <value>    Sets the expected reserved number of blocks for offline block prune (default: 1024)
  -cache <value>                    Megabytes of memory allocated to internal caching (default: 1024)
  -cache.triesinmemory              Number of block states (tries) to keep in memory (default: 128)
  -check-snapshot-with-mpt          Enable checking between snapshot and Merkle Patricia Tree (default: false)
```

This command should be used in an *offline* mode.

## Rationale

The prune command keeps the last `N` blocks in the ancient database and removes the rest. Value of N can vary from 0 to K, where K is length of chain exclusing the blocks in leveldb (90k default) and can be set using the flag `block-amount-reserved`. Basically, the pruning process will begin from genesis moving towards the recent most block. Due to the nature of implementation, pruning can be done multiple times.

For example: If the ancient database has these blocks: [0, 1, ..., 999, 1000]. If the value of `block-amount-reserved` is set to 100, then [0, 1, ..., 900] will be pruned and the ancient database will be left with [901, ..., 1000]. On second round of pruning with value of `block-amount-reserved` set to 50, the remaining values will be [951, ..., 1000].

The pruner maintains an `offset` in the key value store, which depicts the start block number of ancient db. The pruner keeps updating this value after each pruning round and uses it for next round to determine the starting point. The pruner opens a backup ancient db and moves the blocks to be kept from the old ancient db location. Upon completion, it performs the validation to make sure the ancient db and kvDB are in continuity or not and proceeds to delete the old db fully.

Although the implementation is straightforward, it opens up some security concerns which are addressed in the security considerations section below.

## Backwards Compatibility

Although, this feature doesn't need a hard fork, the changes are backwards incompatible (unless a separate fork is maintained). If pruning is done (at least once), the `offset` value is updated and used to open the ancient db. Moreover, it helps in maintaining context that the old historical block data isn't available. The versions of bor which don't support pruning will fail to open the ancient db as it doesn't have the required ancient data and won't know the `offset` value.

## Test Cases

Manual testing as well as end to end unit testing have been carried for this feature.

## Reference Implementation

The reference implementation is available as a part of [this pull request](https://github.com/maticnetwork/bor/pull/751). Huge shoutout and thanks to external contributor [jsvisa](https://github.com/maticnetwork/bor/pull/751) for implementing this feature.

## Security Considerations

Although, having this in PoS is very helpful, it does raise some concerns over data availability. This section will try to put forward some of them in form of pros and cons list.

*Pros*
1. Helps in reducing disk space for normal full node operators which doesn't serve historical rpc requests
2. Allows us to take a step towards enabling [EIP 4444](https://eips.ethereum.org/EIPS/eip-4444) for PoS (although it's a bit different, it's a step in the same direction).

*Cons*
1. Disk costs can be reduced / mitigated by moving ancient db to a cheaper storage medium like HDD with a simple configuration change.
2. Impacts the data availability of the chain as nodes maintaining / serving that data will be reduced. Below are some cases where it will impact.
    - It will be difficult to find peers who will have (and hence serve) historical data in the p2p network (a variation of EIP 4444). A simple counter argument would be that people rely heavily on snapshots for syncing and the number of nodes syncing from scratch will be very less.
    - People might move to RPC providers which anyways have incentives to historical RPC requests.

## Copyright

All copyrights and related rights in this work are waived under [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).