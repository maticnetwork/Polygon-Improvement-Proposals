---
pip: 11
title: Deterministic finality via Milestones
description: Proposal to introduce Milestones in Polygon PoS to have deterministic finality
author: Vaibhav Jindal (@VAIBHAVJINDAL3012), Arpit Temani (@temaniarpit27), Evgeniy Danilenko (@JekaMas), Manav Darji (@manav2401), Sandeep Sreenath (@ssandeep)
discussion: https://forum.polygon.technology/t/pip-11-deterministic-finality-via-milestones/11918/
status: Last Call
type: Core
date: 2023-09-25
---

## Table of contents

- Abstract
- Motivation
- Specification
    - Milestone Length
    - Consensus Layer (Heimdall) 
    - Execution Layer (Bor)
    - Whitelist  
    - Finality Tag
- Glossary 
- Security Considerations 
- Backward Compatibility
- Copyright 

## Abstract

This proposal describes a mechanism called `milestones` that enable faster deterministic finality by leveraging Polygon PoS’s dual client architecture. This is done using a hybrid system that utilizes Tendermint layer consensus, along with an additional fork choice rule within the execution layer.

## Motivation

Currently, finality on Polygon PoS is probabilistic up until the last checkpointed block (~ 512 blocks) after which the chain state is finalised on layer 1 via checkpointing, meaning that the assurance a transaction cannot be changed is determined by the number of confirmation blocks (1 checkpoint in this case). Bor picks the canonical chain via a fork choice rule that selects the longest chain with the highest difficulty.

Probabilistic finality indicates that there is always a chance for blocks to reorg themselves. This prompts dApps and users to require a relatively high number of confirmation blocks, thus negatively impacting user experience. 

By introducing milestones, user experience could be enhanced as it can provide faster and more definite finality, in turn reducing the likelihood of chain reorganizations.

## Specification

Milestones follow a comparable format to checkpoints, but rather than utilizing the `rootHash` in Bor, they utilize the hash of the `endBlock`.

```
type Milestone struct {
	Proposer    HeimdallAddress // Address of the validator who propose it
	StartBlock  uint64          // Start block of milestone
	EndBlock    uint64          // End block of milestone
	Hash        HeimdallHash    // Hash of the end block
	BorChainID  string          // Chain ID of bor (child) chain
	MilestoneID string          // Unique ID for every milestone
	TimeStamp   uint64          // Time at which at got saved in Db
}
```

### Milestone Length

The `endBlock` number will be calculated by fetching the latest block and using the `16 block` Matic chain confirmation. 

The Matic Chain Confirmation process works by waiting for a certain number of blocks to be added to the Bor chain before the `endBlock` of the milestone is selected by the proposer. This mechanism is designed to ensure that a significant number of nodes have the same `endBlock` in their canonical chain, thereby increasing the level of consensus and reducing the probability of milestone failure.

```
EndBlockNum = LatestBlockNum - MaticChainConfirmation (= 16 Blocks)
```

### Consensus Layer (Heimdall)

Tendermint’s instant finality feature will provide the source of truth for Milestones. 

A new `MilestoneValidatorSet` would be defined in the Staking module, responsible for selecting the Milestone proposer based on weighted stake. A new processor inside the bridge - Milestone Processor - would be introduced, responsible for proposing the milestone by fetching required data (hash of the end block) from Bor and broadcasting it to the Heimdall nodes.

The proposed milestone would go through these specific checks in the handler, side handler, and post handler (more details on role of each of them is given in the glossary section below): 

- Handler
    - Heimdall block height should be greater than or equal to the specified fork height.
    - Milestone Length (end-start+1) should be greater than or equal to the minimum milestone length.
    - The start block number should be one more than the latest stored milestone’s end block.
    - The proposer of the milestone should match the proposer from the `MilestoneValidatorSet`. If the proposer is down for any reason, after a specific time period, a new proposer is selected through the `MilestoneTimeout` Msg similar to the way No-Ack msg’s do when the checkpoint proposer is down.
    - And in every Handler call, `MilestoneValidatorSet` is rotated to change the proposer.

- Side Handler
    - Validation of end block hash with the local Bor chain. Heimdall makes the RPC call to local Bor, `GetVoteOnHash(start, end, hash, milestoneID)` (more details in Bor section) and based on the result of this function, it votes `Yes` or `No` for milestone msg.

- Post Handler
    - If the milestone msg fails (doesn’t get 2/3+1 votes or the milestone is not in continuity), then the milestoneID of that milestone is stored; otherwise, if it passes, the complete milestone is stored in Db.

Only the latest 100 milestones will be stored to avoid over-usage of the database. When the 101st milestone is received, the 1st milestone is deleted.


### Execution Layer (Bor)

#### APIs Implemented:

`GetVoteOnHash(startBlock, endBlock, hash, milestoneID)`
  - It will be used by Heimdall for milestone validation.
  - Accepts the `startBlock` number, `endBlock` number, `hash` of the end block (all with respect to bor) and a `milestoneID`.
  - The `hash` is then verified against the local chain, and if it returns true, the local chain will be locked until the `endBlock` number and the milestoneID will be stored in the milestoneID list.
  - After voting `YES` for a particular hash, Bor would not be allowed to reorg beyond that `endBlock` until the confirmation is received from Heimdall, whether the particular milestone has passed or not (this forms the additional fork choice rule).
  - This will prevent Bor from importing fork “B” after voting `YES` for fork “A”.
  - The milestoneID of failed milestones would continue to be fetched from Heimdall, to get the confirmation for that milestone for which has been voted `YES` (by the local node) but has failed in Heimdall. 

#### Bor will continuously fetch the latest Milestone from the Heimdall

- If the received milestone matches the local chain, that milestone would be whitelisted and stored.
- If it doesn’t match with the local chain, then it’s clear that the local chain is on the wrong fork, so the chain would need to be rewound to the end block of the latest whitelisted milestone number and would need to store the received milestone in the future milestone list. (In the rewind feature, the hard limit of 255 blocks is included; it would not be possible to rewind back to more than 255 blocks)
- If it is ahead of the local chain, that milestone will be stored in the future milestone list. 

### Whitelist 

The *whitelisted* milestone and *future milestones* will be used to:
- Check peers while connecting to them. If they don’t match the whitelist milestone, they won’t be used for syncing. 
- Check for the imported chain. If the imported chain doesn’t match with the whitelisted or future milestone, we won’t accept that chain and will stop syncing from that particular peer from whom we have received the chain.

### Finality Tag

Based on the whitelisted milestone, the chain will be treated as finalized until that `endBlock` number and a `finalized` API would be implemented similarly to Ethereum. An example request/response from a testing devnet is shown below: 

Request:
```
curl localhost:8545 -X POST --data '{"jsonrpc": "2.0", "method": "eth_getBlockByNumber", "params": ["finalized", true], "id":1}' -H "Content-Type: application/json"
```

Response:
```
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "baseFeePerGas": "0x7",
    "difficulty": "0x3",
    "extraData": "0xd482030383626f7286676f312e3139856c696e757800000000000000000000004e3e4400e0176fc5d0a1145f15dbd94d23a79f0a16b22f3d7254799e44c053ab6a49c3c623a1a53317cf4bac87b256c4c2238ef89ba9c8cb982eda1b92783e3500",
    "gasLimit": "0x1312d00",
    "gasUsed": "0x0",
    "hash": "0x79942b73bee9a3d97e9999e0fe468570e612b3019776b6a3fb60b753b1286bd6",
    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "miner": "0x0000000000000000000000000000000000000000",
    "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "nonce": "0x0000000000000000",
    "number": "0xb293",
    "parentHash": "0xdb89f77e83ea86d032f695e69f5a719ea57b5787b8ebc2339c2967ba9ea95f3a",
    "receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
    "size": "0x262",
    "stateRoot": "0x9e933b3d429dd25dceeeebdb6693434bc2eac401bae48d31462b5ea9bf52ebb6",
    "timestamp": "0x63ca7a05",
    "totalDifficulty": "0x217b9",
    "transactions": [],
    "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
    "uncles": []
  }
}
```

Cases in which finality API will return nil:
- When the Bor node doesn’t have the stored milestone in its local memory or db.
- When the local Bor chain is behind the stored latest milestone in its local memory or database. (This happens when the chain got rewind after storing the milestone)
- When the local Bor chain doesn’t match with the stored latest milestone. (This would happen if the nodes local chain was rewinded after storing the milestone and then was disconnected from its peers and then started to mine  its own blocks)

## Glossary 

- Handler - In Heimdall, a handler is a function that is responsible for processing a transaction message. When a transaction message is received by a node in the Heimdall, the appropriate handler is invoked to process the message and update the state of the blockchain accordingly. 
- Side Handler - a type of handler that is used for processing and voting on messages related to the side modules of an application  that extend the functionality of an application beyond its core features.
- Post Handler - It is a type of handler that is used for processing messages after they have been validated and executed by the side handlers. It also updates the state of the blockchain accordingly.

## Backward Compatibility

This PIP will not be backward compatible with the current implementation of Bor and Heimdall and will therefore require a hard fork. 

## Security Considerations 

The addition of Milestones and deterministic finality would see a large change to the current finality consensus mechanism. Currently, there are ongoing simulations on devnets, as well as an audit taking place to ensure that the implementation is sound. 
Once the code audits are completed, this proposal will be updated with any new security considerations. 

## Copyright
All copyrights and related rights in this work are waived under [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
