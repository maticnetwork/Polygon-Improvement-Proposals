---
pip: 16
title: Transaction Dependency Data
description: Improving the efficiency of block validation by adding transaction dependency data to block header
author: Jerry Chen (@cffls), Pratik Patil (@pratikspatil024)
discussion: https://forum.polygon.technology/t/proposal-transaction-dependency-data/12705
status: Draft
type: Core
date: 2023-08-04
---

## Abstract

Version 0.4.0 of Bor introduced the capability to execute transactions in parallel during the block importing process, thereby reducing the time taken for block execution and propagation. The current mechanism for parallel execution identifies dependencies between transactions by optimistically executing them simultaneously, then re-executing those dependent on others. However, this trial-and-error method could lead to wasted time and computational resources. To address this problem, there is a solution that enables a validator to add transaction dependency data in the block header, which can then be used by the rest of the nodes in the network to execute transactions more efficiently during parallel block execution.

## Motivation

This enhancement is intended to optimize the efficiency and performance of the Bor network. As the volume of transactions continues to grow, it's crucial to ensure that the system can handle this increase without compromising block execution and propagation time. The current parallel execution approach, while effective, can sometimes lead to unnecessary computational work. By incorporating transaction dependencies directly into the block header, validators can efficiently schedule the parallel execution without needing to re-execute transactions.

## Definitions
**TxN**: A transaction whose position in a block is `N`, where `N ≥ 0`.
Transaction dependency: `TxB` is claimed to have a dependency on transaction `TxA`, when `TxB` reads a value that has been written or modified by `TxA`, where `A < B`. For example, if `Tx0` increases the balance of address `0x0001`** by 1 MATIC, and `Tx1` reads the balance of `0x0001`, then `Tx1` depends on `Tx0`.
**Key**: The identifier of a read/write operation. For example, if a transaction reads slot `0x1234`** from account address `0xabcd`, the key of this read operation will be `0xabcd1234`.

** Account address and slot key in the examples are shortened for readability.


## Specification

Dependency data of a transaction, `TxN`, is stored in an array, where each element in the array is an integer, representing the index of the transaction `TxN` depends on. When a transaction doesn’t have any dependency, the array will be empty.
The dependency data of a block is simply a two-dimensional array, where each element in the first dimension corresponds to a transaction within the block, and the second dimension corresponds to the dependencies of the transaction. This two-dimension array is encoded along with the validator set by recursive length prefix (“RLP”) and added to the extra data field in the block header.

Transaction Dependency data structure:

```
type TxDependency [][]uint64
```

Current extra bytes format:

```
Extra Vanity |    Validator Set    | Extra Seal 
     
   32 bytes      N * 2 * 20 bytes     65 bytes
```

New extra bytes format:

```
Extra Vanity | rlp.encode(Validator Set, TxDependency) | Extra Seal 

   32 bytes              Any length                       65 bytes
```

**Dependency Data Example**

Consider a block with 6 transactions, where `Tx2` depends on `Tx0`, `Tx3` depends on `[Tx1, Tx2]`, and Tx5 depends on `Tx4`. The dependency will be represented in the two-dimension array below:

```
[
  [],
  [], 
  [0], 
  [1,2], 
  [], 
  [4]
]
```

## Rationale

In the existing model of parallel execution, the dependencies between transactions are calculated during block execution. This is done by recording the keys that each transaction reads and writes during execution and finding the common keys between the read set of a transaction and the write set of another. When a dependency between two transactions is identified, it will be added to a dependency map, which is used by an execution scheduler that ensures a transaction doesn't get executed ahead of time. If validators were to calculate the dependency map, and the execution scheduler had access to this information, there would be a significant reduction in time and computational resources, as dependency wouldn't have to be discovered in real-time during block execution.
This PIP proposes two major modifications:
As a part of the block production process, validators will calculate and append dependency data to the block headers.
When validating blocks, full nodes will make use of this dependency data in parallel execution.
Dependency representation
Dependency data can be perceived as a form of mapping, which provides guidance to a node, indicating a series of transactions that must be fulfilled prior to initiating a given transaction.
The example below demonstrates how a scheduler will use the dependency data to schedule transaction execution. Assuming there are 6 transactions in the block (`Tx0, Tx1, … Tx5`). `Tx2` depends on `Tx0`, `Tx3` depends on `Tx1` and `Tx2`, and `Tx5` depends on `Tx4`.

```
{
  2: [0]
  3: [1, 2]
  5: [4]
}
```

The scheduler manages task distribution based on this dependency data. It maintains a local queue of transactions whose dependencies have been fulfilled, thus ready for execution. These queued tasks are then dispatched to parallel workers, promoting efficiency and concurrent processing.

At the beginning, `Tx0`, `Tx1`, and `Tx4` will be pushed to execution queue, because they don’t have any dependency and ready to execute. A transaction will be taken out of the queue whenever a worker is ready for picking up a new execution task. For simplicity, let’s assume it takes the same to execute any transaction.

```
Queue: [0, 1, 4]
```

Given that `Tx0`, `Tx1`, and `Tx4` complete simultaneously, the execution queue and the state of the dependencies would be updated accordingly.

First, we update the dependency data. Since `Tx0`, `Tx1`, and `Tx4` are dependencies for `Tx2`, `Tx3`, and `Tx5` respectively, their dependencies counts are decremented:

```
{
  2: [0] ->  2: []
  3: [1, 2] ->  3: [2]
  5: [4] ->  5: []
}
```

Because `Tx2` and `Tx5` now have no remaining dependencies, they're added to the execution queue:

```
Queue: [2, 5]
```

At this point, Workers 1, 2, and 3 have finished their transactions and are ready to pick up new ones. Workers 1 and 2 pick up `Tx2` and `Tx5` respectively from the queue, leaving the queue empty:

```
Queue: []
```

Worker 3 is now idle as there are no more transactions in the queue.

After `Tx2` finishes executing, its completion triggers a decrement of the dependencies for `Tx3`:

```
{
  3: [2] ->  3: []
}
```

Now that all dependencies for `Tx3` have been fulfilled, it is added to the execution queue:

```
Queue: [3]
```

Worker 3, which is free, can now pick up `Tx3` from the queue for execution, leaving the queue empty again:

```
Queue: []
```

At this point, all transactions have either been completed or are in the process of being executed, and the queue management continues until all tasks are completed.

**Alternative approach of dependency representation**

An alternative approach of expressing the dependency is to linearize the dependency into multiple independent execution paths. Using the example above, the independent execution path will look like this:

```
Path 1: [0,1,2,3]
Path 2: [4,5]
```

Here, `Tx0`, `Tx1`, `Tx2`, and `Tx3` fall into one execution path, while `Tx4` and `Tx5` fall into another. Each path can then be assigned to a different worker

Worker 1 executes the transactions in Path 1 in the order they are listed, beginning with `Tx0`, followed by `Tx1`, `Tx2`, and finally `Tx3`. Worker 2 does the same for Path 2, starting with `Tx4` and then executing `Tx5`.

This process carries on until all transactions within the assigned paths have been executed.

However, there are trade-offs with this approach. By serializing transactions into paths, the opportunity for parallel execution between `Tx2` and `Tx4` is lost. In the previous dependency-driven approach, `Tx2` could execute concurrently with `Tx4` (assuming there were available workers), but in this path-based approach, `Tx4` wouldn't begin execution until all transactions in Path 1 (i.e., `Tx0, Tx1, Tx2, Tx3`) have completed.

This might lead to sub-optimal usage of resources if the number of paths is less than the number of available workers. For example, if there were three workers, one of them would remain idle during this computation because there are only two paths.

### Storage of dependencies

Dependencies of a transaction remain undefined at the time of the transaction's creation. Therefore, these dependencies should be calculated and subsequently recorded in the block headers by the miner who mines the block.

There are two plausible methods for storing dependency data: 1) the addition of a new field within the block header or 2) an expansion of an already existing field in the block header, termed **`extra`**.

The creation of a new field within the block header offers a simplistic and easily implementable solution. However, this approach presents an issue as it diverges Bor's header specification from that of Ethereum. This deviation could pose complexities for node clients that concurrently support both Bor and Ethereum, such as Erigon.

The **`extra`** field, currently existing within the block header, is a byte array that serves two distinct functions in bor:

1. It holds the miner's signature.
2. It stores public keys of validators alongside their corresponding voting powers.

The incorporation of a third component, transaction dependency, within the byte array is feasible. The primary advantage of this approach lies in its avoidance of necessitating a new field, thereby ensuring forward compatibility with future modifications to the header. In comparison to the addition of a new field, modifying the **`extra`** data field may not be as straightforward, but it would require just a single implementation effort.


## Backwards Compatibility

Since this PIP will add dependency data to the block header, it will not be backward compatible with the current implementation of Bor and will therefore require a hard fork.

## Security Considerations

There could be scenarios where a validator injects wrong dependencies in the block header, and this could make the validation slower, or in the worst case, cause the node to be stuck on a particular block.

#### Incorrect Dependency
In this case, the validator could wrongly say that `Tx2` is dependent on `Tx1` even when `Tx1` and `Tx2` are not dependent on each other. This will cause `Tx2` to wait till `Tx1` is executed, hence resulting in a less ideal execution/validation time of the block. In the worst case scenario, a miner can claim that every single transaction depends on the result from its previous one. In this case, the execution time will be roughly the same as serial execution.

#### Out-of-range problem
In this case, the miner could say that `Tx3` is dependent on `Tx10`. But, the block only contains five transactions. This will make the executor wait for `Tx10` to be executed to execute `Tx3`, but as `Tx10` is not in the block, it will keep on waiting forever. This could be mitigated by adding a preliminary check prior to parallel execution.

#### Circular Dependency
This is a case where the miner could say that `Tx1` is dependent on `Tx2`, and `Tx2` is dependent on `Tx1`. Hence the executor will not execute any of `Tx1` or `Tx2` as both of them are waiting for each other to finish the execution. To prevent this from happening, we can assert that for every transaction, `TxA`, in the dependencies of `TxB`, the condition `A < B` must hold true.


## Copyright

All copyrights and related rights in this work are waived under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
