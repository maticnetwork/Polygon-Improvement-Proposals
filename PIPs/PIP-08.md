---
PIP: 8
Title: PIP Classification, Workflow and Implementation
Description: Describes a proposed classification scheme for PIPs and their respective work flows.
Author: Harry Rook (@hrook1), Mateusz Rzeszowski
Discussion: https://forum.polygon.technology/t/pip-8-pip-classification-workflow-and-implementation/11365/1
Status: Continuous
Type: Informational
Date: 2023-02-21
---

## Abstract

This document describes a proposed classification scheme for PIPs and their respective work flows.

The specification defines the layers and sets forth specific criteria for deciding to which layer a particular standard PIP belongs and how changes to each layer should be implemented. 

PIPs in all tracks follow in the spirit of rough consensus in which they are signaling mechanisms that are used to facilitate the molding of ideas into implemented changes.
 
## Motivation

Each PIP should be categorized and generally follow a defined workflow in an effort  to standardize the implementation framework.

## Specification

As detailed in PIP-1, PIPs are placed in one of four categories:

   1. Core
   2. Contracts 
   3. Interface
   4. Informational 
 
 #### 1. Core Layer
 
The core layer consists of two separate modules:

  1. Heimdall layer — a set of proof-of-stake Heimdall nodes running in parallel to the Ethereum mainnet, monitoring the set of staking contracts deployed on the Ethereum mainnet, and committing the Polygon network checkpoints to the Ethereum mainnet. Heimdall is based on Tendermint.
  2. Bor layer — a set of block-producing Bor nodes shuffled by Heimdall nodes. Bor is based on Go Ethereum.


## Implementation Flow

The Polygon core layer has two separate mechanisms to implement changes to the protocol. The Heimdall Governance Module can implement synchronous Heimdall parameter changes across the network. All remaining protocol changes are done via rough consensus, where validator nodes enforce a change with staked $MATIC by updating their client.  

## Governance Module

The Heimdall Governance Module can execute `parameterchangeproposal`s. Using this type of proposal, validators can change any parameters in any module of Heimdall. Example: change minimum tx_fees for the transaction in the auth module. When the proposal gets accepted, it automatically changes the params in Heimdall state. No extra transaction is needed.

A list of the changeable parameters by the Heimdall governance module are available [here](https://github.com/maticnetwork/heimdall/blob/develop/auth/types/params.go).

CLI commands:

Submit a proposal

`heimdallcli tx gov submit-proposal \
	--validator-id 1 param-change proposal.json \
	--chain-id <heimdall-chain-id>`

`proposal.json` is a file that includes a proposal in JSON format.

`{
  "title": "Auth Param Change",
  "description": "Update max tx gas",
  "changes": [
    {
      "subspace": "auth",
      "key": "MaxTxGas",
      "value": "2000000"
    }
  ],
  "deposit": [
    {
      "denom": "matic",
      "amount": "1000000000000000000"
    }
  ]
}`


## Rough Consensus 

‘Rough consensus’ is defined as the 'dominant view’ of a group as determined by the current consensus framework. In the absence of a vote that can carry out a synchronous update across the network, the ‘dominant view’ is defined by the node software used by each validator, weighted by its total stake. 

For all protocol change proposals that fall outside the scope of the Heimdall governance module, the use of rough consensus is used to implement changes on a node by node basis. 

In the case of a hardfork, consensus is programmatically defined by ⅔+1 of total staked $MATIC validating the chain. 

When a change requires a hardfork, these updates can be coordinated by anyone but absent any single party coordinating the change, the Polygon Labs Development Team will do so on the ‘Polygon Builder Sessions’ meetings, in which all Polygon community members are invited to join the conversation. The role of Polygon Labs will be limited to coordination.

#### 2. Contract Layer

To enable the Proof of Stake (PoS) mechanism on Polygon, the system employs a set of staking management contracts on the Ethereum mainnet.

The staking contracts are responsible for the following:

1. Staking $MATIC tokens on the Ethereum mainnet and joining the system as a validator/delegator.
2. Earn staking rewards for validating state transitions on the Polygon network.
3. Save checkpoints on the Ethereum mainnet.

The Polygon Layer 1 contracts are controlled by the 5/9 multi-sig contract 0xFa7D2a996aC6350f4b56C043112Da0366a59b74c. 

There is currently no prescribed consensus frameworks, processes, or voting parameters for contract layer PIPs. A voting outcome can be presented to all ecosystem participants, including all multi-sig key holders, to show preference and signal potential development on an issue.

For this reason, PIPs that aim to alter any element of the Polygon Layer 1 staking contracts are only advisory in nature and are non-binding. The only purpose of any vote is to signal community sentiment toward a certain issue. 

The multi-sig signatories are therefore not obligated to follow the result of a vote of this nature and do so at their own discretion. 

## Implementation Flow

The Polygon Layer 1 contracts signaling and multi-sig activity outlined above are implemented in accordance with the flow set forth below.

## PIP Creation

A PIP intending to make a change to the Polygon Layer 1 contracts should follow the standards and generalized flow as detailed in PIP-1.

## Signaling

The PIP-8 framework doesn’t prescribe any signaling process, but the community may use the various polling features on the community forum, e.g., a one validator, one vote poll.

## Technical Implementation 

A favorable signal may result in a participant in the Polygon community, which may include developers from the Polygon Labs Development Team, producing the transaction payload which is then proposed and voted on by the multi-sig signatories by way of transaction approval.
 
#### 3. Interface Layer

The API/RPC layer specifies higher level calls accessible to applications. Support for these PIPs is not required for basic network interoperability but might be expected by some client applications.

There’s room at this layer to allow for competing standards without breaking basic network interoperability.
PIPs of this nature should be implemented via rough consensus. 

#### 4. Informational 

These types of proposals contain recommendations, information, or general thinking on an issue to the community. An informational PIP doesn’t require validator consensus, but may be adopted by the wider ecosystem. Hence, no particular implementation flow is prescribed.

## Copyright
All copyrights and related rights in this work are waived under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).


