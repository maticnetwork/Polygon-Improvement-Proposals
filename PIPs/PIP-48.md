---
PIP: 48
Title: Path-Based Storage Scheme
Description: Proposes a path-based storage scheme for the Polygon PoS chain
Author: Jerry Chen (@cffls)
Discussion: https://forum.polygon.technology/t/pip-48-path-based-storage-scheme/19917
Status: Draft
Type: Core
Date: 2024-10-17
---


## Abstract

This proposal introduces a Path-Based Storage Model for the Polygon PoS chain, aimed at improving trie access speed, reducing data redundancy and disk I/O. 
This model changes the way trie nodes are persisted on disk and manages multiple versions of the state, addressing challenges with the current storage mechanism.

## Motivation

As Polygon PoS's user base and on-chain activity continue to grow, the current hash based state storage model faces limitations, particularly around data redundancy and performance degradation over time. 
Every update to the state trie persists new nodes into LevelDB based on their hash, causing redundancy and substantial data growth which increases I/O and memory hardware requirements. 
The Path-Based Storage Model seeks to address these issues by switching the trie-node storage format to a path-based key system. This model allows for online data pruning, faster trie access, and optimized memory usage.

## Specification

### Path-Based Trie Persistence
In the proposed model, trie nodes are persisted to disk using the encoded path as the key. The key encoding scheme varies based on the trie type:
Account Trie: The key is generated from the account address hash.
Storage Trie: The key is generated from the storage address hash and account address hash.
Using the path-based approach, both the account and storage tries update in place, ensuring that only a single version of the trie node exists for any given path, providing a natural state pruning mechanism.

### Diff and Disk Layer Management
In this structure, each state represents a layer of changes applied to the trie, with the bottom-most diff layer eventually being flushed to disk. The disk layer stores only the latest trie state for each path, while the diff layers handle recent changes in memory.

```
+-----------------+-----------------+
| State Y         | State Y'        |
+-----------------+-----------------+
| State X         | State X'        |
+-----------------------------------+
|              .......              |
+-----------------------------------+
| Bottom-most diff layer            |
+-----------------------------------+
| Disk layer (singleton trie)       |
+-----------------------------------+
```

The new model organizes trie nodes into a diff layer and a disk layer. These layers represent different versions of the state trie as they evolve over time. The diff layer is kept in memory, and changes are only flushed to disk when necessary. A maximum of 128 diff layers are maintained, ensuring recent state changes are efficiently stored and retrievable.
When a large number of trie nodes accumulate, the bottom-most diff layer is flushed into the disk layer to prevent memory exhaustion. The structure also supports multiple version state access, allowing Polygon PoS to handle reorgs efficiently by applying reverse diffs from memory or disk.

### Workflow Comparison

Current Implementation:
Upon receiving a new block, transactions are executed, updating the account and storage tries.
Modified tries are committed to an in-memory database (TrieDB), then persisted to disk (LevelDB) using the node’s hash as the key.
This results in redundant data being stored, as new versions of nodes with different hashes are written to disk without deleting old nodes.
Proposed Path-Based Model:
Upon state changes, trie nodes are stored in memory and then persisted to disk using their path as the key.
Stale nodes are automatically pruned since the path for a node remains constant across updates. The latest version overwrites the old data.
Trie access becomes faster as only a single version of each node is retained on disk.

## Backward Compatibility

The path-based model introduces changes to the storage mechanism but remains backward compatible. It does not require any changes to existing smart contracts or account structures, as it is an implementation-level improvement.
However, an existing database with hash-based scheme won't be compatible with the new path-based scheme. A node will need to either re-sync from genesis with path-based scheme from genesis block or use a snapshot that is path-based scheme.
Currently, path-based scheme only supports full node. In the future, we can consider supporting path-based scheme for archive node as well.

## Security Considerations
Fundamentally, the path-based model is simply a better implementation of persisting tries in memory and database. The path calculation of accounts and storage slots stays the same, so it won’t have any impact on security.

## Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
