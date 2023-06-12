| PIP               | Title                           | Description          | Author                        | Discussion | Status | Type                                     | Date                  |
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|
| 12 | Time Based StateSync Confirmations Delay  | Fix to the statesync confirmations issue in bor | [Jerry Chen](https://github.com/cffls), [Pratik Patil](https://github.com/pratikspatil024), [Manav Darji](https://github.com/manav2401) | [Forum](https://forum.polygon.technology/t/pip-12-time-based-statesync-confirmations-delay/11950)  | Review | Core | 2023-05-11
---

### Abstract 

[PIP-10](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-10.md) outlined a state sync bug that can arise when network partitions (reorg) have lengths  > `sprintLength` (16 blocks). In summary, the issue occurs when the two chains eventually merge, as the current value of  `to` (which is used to fetch the state sync events) is based on `sprintLength`. Block production will occur at different times on each fork, meaning nodes on the incoming chain may have a different number of state sync transactions.

PIP-10 sought to delay state sync intervals so that the delay in the value of `to` is comfortably larger than the time of block `n = (current block number) - (sprint length)`.  This solution still depended on the assumption that, although less likely, if a reorg of length greater than 128 takes place, the bug's resultant `BADBLOCK` error can still occur for nodes on the incoming chain.

Moreover, another issue (which was discovered later on) can occur causing `BADBLOCK` error in bor due to the `from` field as well. The `from` ID denotes the state sync ID to start fetching data from. This value is derived from the public variable `lastStateID` of the validator set genesis contracts on child chain (i.e. bor) from the local state instead of incoming one leading to wrong values for reorgs with lengths > 2 * `sprintLength` (16 blocks).


### Rationale

When calculating the value of `to`, the block header is retrieved using the function `GetHeaderByNumber` which returns the header from Bor’s local database and not from the incoming fork. During a network partition (reorg), an ideal `to` value should be taken from the incoming fork. However, Bor chooses the value based on the existing chain written in the database, leading to different values of `to`.

When calculating the value of `from` ID which is `lastStateId+1`, the value of last state ID is fetched rom the genesis contracts using a normal `eth_call`. This call will perform this EVM call on the local state. Ideally, the query should be performed on the incoming chain's state instead. This leads to wrong calculation of the `from` field.

This PIP proposes 2 modifications.
1. Calculating the value of `to` in a way that does not rely on querying Bor’s local database and instead uses the current block.
2. Calculating the value of `from` in a way that does not rely on local state but instead uses incoming state.


### Specification

This PIP proposes to introduce a new genesis parameter: `stateSyncConfirmationDelay`. This parameter defines the number of seconds subtracted from the current block’s timestamp to calculate the value of `to`, while fetching the statesync events from Heimdall.

Instead of taking the timestamp of a block in the past, the timestamp of the current block is taken, subtracting 128 seconds from it. This way, the value of `to` will remain consistent across the network (as it's now self contained in the chain itself), even in the case of a reorg. The current block will be the one that is imported, therefore returning the same value of `to`, resulting in the same statesync transactions returned from Heimdall.

Moreover, a new method called `eth_callWithState` has been implemented which will take a pointer to the `stateDB` instance as an input and perform an EVM call on top of it. As we have access to the incoming state in the `CommitStates` function of bor consensus, we can leverage it to make this call.


#### Example: 

Assuming there are two different forks, “A” and “B”, with a different number of state sync transactions at the start of a particular sprint:

If there are 2 state syncs in fork A and 3 state syncs in fork B, when forks A and B merge back, (fork B merges back to fork A), then B will import A’s state.

When nodes on fork B import the state from fork A (at the first block of each sprint), Bor will attempt to fetch the state sync from Heimdall. 

Nodes from fork B will execute these blocks from fork A one block at a time. If the current block is the first block of a sprint, then Bor will perform an `eth_callWithState` to fetch the `lastStateID` to calculate `from` and will subtract 128 seconds from that block’s timestamp to get the value of `to`:
- ```
  lastStateId = eth_callWithState(ctx, state, data, ...)
  from = lastStateId + 1
  to = (current block timestamp) - (128 seconds).
  ```

This values will then be used to query statesync transactions from Heimdall, returning 2 transactions (currently it would return 3, causing a `BADBLOCK` error).


#### Genesis File
A new genesis parameter is introduced `stateSyncConfirmationDelay` which  is stored as a `map[string]unit64` and used to calculate the time delay for state-sync confirmations.
```
"stateSyncConfirmationDelay": { 
	"HF_BLOCK": 128
},
```

#### Bor Consensus Rules

The existing implementation and proposed changes for calculating `from` and `to` are mentioned below (with few added abstractions from actual implementation for easy understanding):

Current implementation for fetching from:
```
lastStateID := bor.GenesisContractClient.LastStateId(parentBlock.Number)
from := lastStateID + 1

func (gc *GenesisContractClient) LastStateId(number *types.Number) {
  // build transaction args
  result := gc.ethAPI.Call(ctx, args, number)
  return result
}
```

Proposed changes:
```
lastStateID := bor.GenesisContractClient.LastStateId(state.Copy(), parentBlock.Number, parentBlock.Hash)
from := lastStateID + 1

func (gc *GenesisContractClient) LastStateId(state *state.StateDB, number *types.Number, hash common.Hash) {
  // build transaction args
  result := gc.ethAPI.CallWithState(ctx, args, state, number, hash)
  return result
}
```

Current implementation for fetching to:
```
// Currently sprintLength is 16
to = GetHeader(currentBlockNumber - sprintLength).Time
```

Proposed Change:
```
// after hard fork, fetch stateSyncConfirmationDelay from genesis file
stateSyncDelay:=FetchStateSyncDelay(currentBlockNumber)
to := header.Time - stateSyncDelay

// before hard fork, (same as earlier)
to = GetHeader(currentBlockNumber - sprintLength).Time
```

Summary:<br>
- Current sprint length = 16
- ***State sync confirmation delay = 128 seconds (Proposed)***
- The range to calculate the `to` value will increase from 16 blocks to 128 seconds.
- Use incoming chain's state for fetching the last state ID used for calculating the `from` value.

### Security Considerations

The proposed state sync confirmation delay would lead to state sync transactions being delayed by (128 - 16* 2.25) ~ 92 seconds as the interval is increased from 16 blocks to 128 seconds.

### Backward Compatibility

This PIP will not be backward compatible with the current implementation of Bor and will therefore require a Hard Fork.

### Appendix

* [PIP-10: Increase StateSync Confirmations](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-10.md)
* [Implementation PR](https://github.com/maticnetwork/bor/pull/864)

### Copyright 
All copyrights and related rights in this work are waived under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).

