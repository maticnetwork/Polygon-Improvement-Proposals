| PIP               | Title                           | Description          | Author                        | Discussion | Status | Type                                     | Date                  |
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|
| 10 | Increase StateSync Confirmations  | Proposal to increase the statesync confirmations in bor | Krishna Upadhyaya, Manav Darji, Pratik Patil | [Forum](https://forum.polygon.technology/t/proposal-increase-statesync-confirmations/11779)  | WIP | Core | 2023-18-04
---

### Abstract 

At block #39599624, The Polygon PoS chain experienced a reorg with a depth of 157 blocks. This caused temporary instability in the chain, leading to issues for users.

Nodes were experiencing a bad block error. As a result of this, reorgs were occurring at a greater frequency than normal and block propagation was being affected as peers would reject blocks and fail to broadcast them further.

It was observed there were two forks running with more than 16 (sprint length) blocks, and as a result of this, the `to` value, which is used to fetch the state sync events, was different in both forks, causing bad blocks to occur.

```
to := time.Unix(int64(chain.Chain.GetHeaderByNumber(number-c.config.CalculateSprint(number)).Time), 0)
```

### Motivation

In order to query the state sync events, 2 arguments are needed, `fromID` and `to`. `fromID` is the ID of the state sync (which is an incremental integer), and `to` is the time of the block which is the sprint length behind the current block. Now, if the sprint length is decreased, then the value of to would decrease, as there would be less number of blocks between the current block(`n`) and the block which is a sprint length behind (`n - sprint length`).

The probability of this scenario happening has increased after the Delhi Hard Fork where the sprint length was decreased from 64 to 16 as a decrease in the sprint length would result in a decreased value of `to`. The lower the value of `to`, increases the probability of having different values of `to` for different transient forks.

### Specification

This PIP proposes to:
1. Remove depending on the Sprint Length (`c.config.CalculateSprint(number)`).
2. Calculate the value of the `to` variable by multiplying `c.config.CalculateSprint(number)` so that the interval is 128 blocks. Meaning that from the genesis block, the multiplier will be at `1` and after the implementation of increased state sync confirmations, it will be `8`. The value of the multiplier will be stored in the genesis file as `map[string]uint64`, a mapping from the block number to the multiplier’s value. 

### Implementation

This implementation aims to be as simple as possible and can be accomplished by updating the genesis file ‘indoreBlock’ hard fork block number, as well as adding the semantics for the mapping of the block number to the multiplier:
```
"indoreBlock": 42500032,

"stateSyncConfirmationMultiplyer": {
        "0":        1,
        "42500032": 8
},
```

Current Implementation of ‘to’ value:

```
to := time.Unix(int64(chain.Chain.GetHeaderByNumber(number-c.config.CalculateSprint(number)).Time), 0)
```

Proposed change for ‘to’ value:
```
stateSyncMultiplyer := c.config.FetchStateSyncMultiplyer(number)
to := time.Unix(int64(chain.Chain.GetHeaderByNumber(number-(stateSyncMultiplyer*c.config.CalculateSprint(number))).Time), 0)
```

### Security Considerations

The only consequence of this change would be that state sync transactions is delayed by `((128-16) * 2.25) ~ 252 seconds` as the interval would be increased from 16 to 128 blocks.

### Backward Compatibility

This PIP will not be backwards compatible with the current implementation of Bor and Heimdall and will therefore require a hard fork. 


### Appendix

* [PIP-7: Delhi Hardfork](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-7.md)


