---
PIP: 52
Title: Execution Derived Randomness for Producer Selection
Author: Marcello Ardizzone (@marcello33), Raneet Debnath (@Raneet10), Angel Valkov (@avalkov)
Description: Proposes an upgrade for Heimdall by improving the bor blocks producers’ selection algorithm
Discussion: TODO
Status: Draft
Type: Core
Date: 2024-11-20
---

## Abstract
In Heimdall, the producers of a range of Bor blocks are calculated ahead of time. This range of blocks is called a span, and it consists of a start and end block, along with the subset of validators producing those blocks.
A validator proposes details of the next span using the structure below:

```go
type MsgProposeSpan struct {
ID         uint64                  json:"span_id"
Proposer   hmTypes.HeimdallAddress json:"proposer"
StartBlock uint64                  json:"start_block"
EndBlock   uint64                  json:"end_block"
ChainID    string                  json:"bor_chain_id"
Seed       common.Hash             json:"seed"
}
```

Here, the Seed is the Ethereum block hash used as the pseudo-random number generator for the producer selection process. Heimdall locally stores the block hash and increments the block once we have successfully decided upon a span.

## Specification
To make the algorithm more secure and fair, when selecting block producers for span N, the voting power of the validators is rolled back from span N-2. If a new validator joins in the interval of spans [N-2, N], they won’t participate in this run. The same goes for validators exiting during that timeframe, as they won’t be considered. We don’t consider span N-1 in such a scenario, because when proposing span N, the previous span is not yet completed.
This proposal seeks to amend the source of randomness used in Seed from an Ethereum hash to a bor block hash. The selection process includes identifying the authors of the last 100 span seeds. Then, it iterates across the blocks from the end block to start within the span N-2 and finds a block that was not produced by an identified author. If such a block is not found, the algorithm returns the last block that presents authors who are different from the last one.
The selection algorithm can be shown as follows:

```pseudo
// FetchNextSpanSeed generates the seed for the next span using Bor chain data
FUNCTION FetchNextSpanSeed(context, spanId):
seedSpanID = spanId - 2


    // Fetch the span corresponding to the seedSpanID
    seedSpan = GetSpan(context, seedSpanID)


    // Fetch the Bor block and author for the seed span
    borBlock = GetBlockForSeed(context, seedSpan, spanId)


    // Fetch the block header from the Bor chain for the specified block
    header = GetBlockHeader(borBlock)


    RETURN header.Hash()


// FreezeValSet stores the validator set for the next span
FUNCTION FreezeValSet(context, id, startBlock, endBlock, borChainID):
// Get the seed for the next span
seed = FetchNextSpanSeed(context, id)

    lastSpanId = id - 2
    lastSpan = GetSpan(context, lastSpanId)
    prevVals = [validator FOR validator IN lastSpan.ValidatorSet.Validators]


    // Select producers based on previous validators
    newProducers = SelectNextProducers(context, seed, prevVals)
    
    // Create and store the new span
    newSpan = NewSpan(id, startBlock, endBlock, GetValidatorSet(context), newProducers, borChainID)
    
    RETURN AddNewSpan(context, newSpan)


// SelectNextSpanProducers selects producers for next span
FUNCTION SelectNextSpanProducers(context, seed, prevVals):
// Get validators eligible for the next span
spanEligibleVals = GetSpanEligibleValidators(context)
producerCount = Params.ProducerCount


    // Adjust voting powers if previous validators exist
    IF prevVals IS NOT NULL:
        spanEligibleVals = RollbackVotingPowers(spanEligibleVals, prevVals)


    // Select new producers based on seed and eligibility
    newProducersIds = SelectionAlgorithm(seed, spanEligibleVals, producerCount)


    // Assign voting power to selected producers
    IDToPower = {}
    FOR EACH ID IN newProducersIds:
        IDToPower[ID] = IDToPower.get(ID, 0) + 1


    // Create new producer list with updated voting powers
    vals = []
    FOR EACH (key, value) IN IDToPower:
        validator = GetValidatorFromValID(context, NewValidatorID(key))
        IF validator EXISTS:
            validator.VotingPower = value
            APPEND validator TO vals


    RETURN SortValidatorByAddress(vals)


// RollbackVotingPowers restores voting powers of eligible validators
FUNCTION RollbackVotingPowers(valsNew, valsOld):
idToVP = {validator.ID: validator.VotingPower FOR validator IN valsOld}


    FOR EACH validator IN valsNew:
        validator.VotingPower = idToVP.get(validator.ID, 0)


    RETURN valsNew
```

The code determines the validators eligible for the next span, ensuring that those marked for deactivation are excluded. If a set of previous validators exists, the voting power of eligible validators is adjusted based on historical data to maintain continuity.
A deterministic selection algorithm, driven by a seed, is used to choose block producers from the eligible set. Voting power is aggregated and assigned to each selected validator. Finally, the producer set is sorted by address to standardize the order. The seed span ID is calculated as N-2, and the seed is derived from the block hash retrieved from the Bor chain.

## Backwards Compatibility
The upgrade is not backward compatible, hence will require a hard fork of the Heimdall network.
Security Considerations

In order to prevent liveness failures in Bor due to the seed not being received in Hiemdall in sufficient time, the seed from span N-2 is used instead of span N-1. This is because we start looking for the seed from the end block of the span, which would be too late when we want to commit span N. Similarly, retrospectively looking at 100 spans to uniquely decide upon the next seed author was meant to impartially choose a seed for the algorithm.

## Copyright
All copyrights and related rights in this work are waived under CC0 1.0 Universal.


