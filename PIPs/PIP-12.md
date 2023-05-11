| PIP               | Title                           | Description          | Author                        | Discussion | Status | Type                                     | Date                  |
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|
| 12 | Time Based StateSync Confirmations Delay  | Fix to the statesync confirmations issue in bor | Jerry Chen, Pratik Patil | [Forum]()  | Draft | Core | 2023-11-05
---

### Abstract 

[PIP-10](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-10.md) outlined a state sync bug that can arise when network partitions (reorg) have lengths  > `sprintLength` (16 blocks). In summary, the issue occurs when the two chains eventually merge, as the current value of  `to` (which is used to fetch the state sync events) is based on `sprintLength`. Block production will occur at different times on each fork, meaning nodes on the incoming chain may have a different number of state sync transactions.

PIP-10 sought to delay state sync intervals so that the delay in the value of `to` is comfortably larger than the time of block `n = (current block number) - (sprint length)`.  This solution still depended on the assumption that, although less likely, if a reorg of length greater than 128 takes place, the bug's resultant `BADBLOCK` error can still occur for nodes on the incoming chain. 


### Rational

When calculating the value of `to`, the block header is retrieved using the function `GetHeaderByNumber` which returns the header from Bor’s local database and not from the incoming fork. During a network partition (reorg), an ideal `to` value should be taken from the incoming fork. However, Bor chooses the value based on the existing chain written in the database, leading to different values of `to`.

This PIP proposes to calculate the value of `to` in a way that does not rely on querying Bor’s local database and instead uses the current block.


### Specification

This PIP proposes to introduce a new genesis parameter: `stateSyncConfirmationDelay`. This parameter defines the number of seconds subtracted from the current block’s timestamp to calculate the value of `to`, while fetching the statesync events from Heimdall.

Instead of taking the timestamp of a block in the past, the timestamp of the current block is taken, subtracting 128 seconds from it. This way, the value of `to` will remain consistent across the network, even in the case of a reorg. The current block will be the one that is imported, therefore returning the same value of `to`, resulting in the same statesync transactions returned from Heimdall.

#### Example: 

Assuming there are two different forks, “A” and “B”, with a different number of state sync transactions at the start of a particular sprint:

If there are 2 state syncs in fork A and 3 state syncs in fork B, when forks A and B merge back, (fork B merges back to fork A), then B will import A’s state.

When nodes on fork B import the state from fork A (at the first block of each sprint), Bor will attempt to fetch the state sync from Heimdall. 

Nodes from fork B will execute these blocks from fork A one block at a time, and if the current block is the first block of a sprint, then Bor will subtract 128 seconds from that block’s timestamp to get the value of `to`:
- ```
  to = (current block timestamp) - (128 seconds).
  ```

This value will then be used to query statesync transactions from Heimdall, returning 2 transactions (currently it would return 3, causing a `BADBLOCK` error).


#### Genesis File
A new genesis parameter is introduced `stateSyncConfirmationDelay` which  is stored as a `map[string]unit64` and used to calculate the time delay for state-sync confirmations.
```
"stateSyncConfirmationDelay": { 
	"HF_BLOCK": 128
},
```

#### Bor Consensus Rules

Current Implementation:
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

### Security Considerations

The proposed state sync confirmation delay would lead to state sync transactions being delayed by (128 - 16* 2.25) ~ 92 seconds as the interval is increased from 16 blocks to 128 seconds.

### Backward Compatibility

This PIP will not be backward compatible with the current implementation of Bor and will therefore require a Hard Fork.

### Appendix

* [PIP-10: Increase StateSync Confirmations](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-10.md)

### Copyright 
All copyrights and related rights in this work are waived under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).

