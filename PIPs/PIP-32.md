| PIP | Title | Description | Author | Discussion | Status | Type | Date |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 31 | Ancient data pruning | Ability to prune ancient blockchain data for PoS | [Manav Darji](https://github.com/manav2401/) | TBD | Draft | Core | 2023-12-18 |

### Abstract

A full node of the primary client of Polygon PoS, [bor](https://github.com/maticnetwork/bor) bifurcates data storage to the following areas:

1.  LevelDB (A key-value store)
2.  Freezer Database

The most recent data (the past `90000` blocks (a configurable limit)) is stored in LevelDB for faster read/write on the most recent block data and state. The data beyond this limit is moved to the Freezer Database, which stores the historical data. For a full node, the Freezer Database stores block headers, body and receipts for fulfilling RPC requests over them in a flattened raw binary format. Currently, as bor is derived from geth, it supports state pruning, which prunes the old state present in the key-value store, accumulating as the chain progresses. This reduces disk space as we remove the state data that is not required.

### Motivation

Currently, the disk size of the bor database on mainnet and testnet are as follows:

1.  Mainnet:

* Total datadir: 5 TB
* Ancient dir: 1.7 TB

2.  Testnet:

* Total datadir: 461 GB
* Ancient dir: 191 GB

The ancient data accounts for 34% and 41% of the total size in mainnet and testnet respectively.

While this data is required for nodes to serve RPC requests on historical blocks, it is not a necessity for a node to store this data if its purpose is to produce/validate a block and participate in consensus (i.e. a validator node) or a regular sentry node. Currently, there's no way to prune the Freezer/Ancient database from a full node.

Having a way to prune this data helps in reducing the disk space for normal full nodes that don't serve historical RPC requests. Moreover, it would enable a step towards enabling [EIP 4444](https://eips.ethereum.org/EIPS/eip-4444) for Polygon PoS (although it's a bit different, it's a step in the same direction). This PIP proposes the bor client to have the ability to prune ancient data reducing the overall disk space consumption.




### Specification

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

This command should be used in an _offline_ mode.

### Rationale

The prune command keeps the last `N` blocks in the Freezer database and removes the rest. Value of `N` can vary from 0 to `K`, where `K` is the length of the chain excluding the blocks in LevelDB (90k default) and can be set using the flag `block-amount-reserved`. Basically, the pruning process will begin from genesis moving towards the most recent block. Due to the nature of implementation, pruning can be done multiple times.

For example: the Freezer database has these blocks: \[0, 1, ..., 999, 1000\]. If the value of `block-amount-reserved` is set to 100, then \[0, 1, ..., 900\] will be pruned and the database will be left with \[901, ..., 1000\]. On the second round of pruning with the value of `block-amount-reserved` set to 50, the remaining values will be \[951, ..., 1000\].

The pruner maintains an `offset` in the key value store, which depicts the start block number of the Freezer db containing ancient data. The pruner keeps updating this value after each pruning round and uses it for the next round to determine the starting point. The pruner opens a backup Freezer db and moves the blocks to be kept from the old db location. Upon completion, it performs the validation to make sure the Freezer db and kvDB (LevelDB) are in continuity and proceeds to delete the old db fully.

Although the implementation is straightforward, it opens up some security concerns which are addressed in the security considerations section below.

### Backwards Compatibility

Although this feature doesn't need a hard fork, the changes are backwards incompatible (unless a separate fork is maintained). If pruning is done (at least once), the `offset` value is updated and used to open the Freezer db. Moreover, it helps in maintaining context that the old historical block data isn't available. The versions of bor which don't support pruning will fail to open the Freezer db as it doesn't have the required ancient data and won't know the `offset` value.


### Security Considerations

This section mentions some concerns raised by this change and also some alternatives to achieve the same. 

1.  Impacts the data availability of the chain as nodes maintaining/serving that data will be reduced.
-   It will be difficult to find peers who will have (and hence serve) historical data in the p2p network (a variation of EIP 4444).   
-   A simple counterargument for this concern is that node operators can rely heavily on snapshots for initial synchronization instead of syncing from scratch. 
-   For querying historical data via RPC, people will move to RPC providers who are incentivized to serve and keep that data.
    
2.  Disk costs can be reduced/mitigated by moving Freezer db containing historical data to a cheaper storage medium like HDD with a simple configuration change, removing the need for this feature.

### Copyright

All copyrights and related rights in this work are waived under [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
