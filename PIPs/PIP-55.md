---
PIP: 55
Title: Addition of a new span message type in heimdall
Author: Raneet Debnath (@Raneet10), Angel Valkov (@avalkov)
Description: Proposes a new span message transaction type in heimdall
Discussion: TODO
Status: Draft
Type: Core
Date: 2025-01-08
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

This proposal introduces a new field in span transaction message type:

```go
type MsgProposeSpanV2 struct {
ID         uint64                  `json:"span_id"`
Proposer   hmTypes.HeimdallAddress `json:"proposer"`
StartBlock uint64                  `json:"start_block"`
EndBlock   uint64                  `json:"end_block"`
ChainID    string                  `json:"bor_chain_id"`
Seed       common.Hash             `json:"seed"`
SeedAuthor common.Address          `json:"seed_author"`
}
```

where `SeedAuthor` is the producer of the bor block whose hash corresponds to `Seed`.

## Specification
For context, refer to [PIP-52](https://github.com/Raneet10/Polygon-Improvement-Proposals/blob/raneet10/pip-55/PIPs/PIP-52.md) and [PIP-53](https://github.com/Raneet10/Polygon-Improvement-Proposals/blob/raneet10/pip-55/PIPs/PIP-53.md).
After the Jorvik hardfork went live on Amoy, we saw several instances of heimdall nodes crashing. Upon investigation, it was revealed this happened due to a non-deterministic change introduced when a span is being committed in the database:

```go
_, producer, err := k.getBorBlockForSpanSeed(ctx, lastSpan, msg.ID)
if err != nil {
logger.Error("Unable to get seed producer", "Error", err)
return common.ErrUnableToGetSeed(k.Codespace()).Result()
}
```

`getBorBlockForSpanSeed` tries to fetch the producer for the bor block to be used as `Seed` in the span corresponding to the `msg`. If the bor node from which this info was being fetched from doesn't return deterministic results, i.e. if it lags behind or doesn't respond etc., it would cause the corresponding heimdall to incorrectly reject that span proposal and produce an inconsistent application DB commit hash (known as `app hash`) from the rest of the network.

To mitigate such occurrence, we add the seed producer as part of the span msg itself which will be proposed by a validator. The rest of the validators can then verify the correctness of the `SeedAuthor` stateless-ly, like other fields in the span msg.

## Backwards Compatibility
The upgrade is not backward compatible, hence will require a hard fork of the Heimdall network.

## Security Considerations
TBD.

## Copyright
All copyrights and related rights in this work are waived under CC0 1.0 Universal.
