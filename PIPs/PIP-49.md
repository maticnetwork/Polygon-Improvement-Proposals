---
PIP: 49
Title: ZK Checkpoint
Description: Proposes usage of zero knowledge proofs for checkpointing for the Polygon PoS chain
Author: Manav Darji (@manav2401)
Discussion: <TODO>
Status: Draft
Type: Core
Date: 2024-11-04
---

## Abstract

This proposal introduces a new mechanism to use zero knowledge proofs replacing the existing checkpointing mechanism. The primary aim of this is to reduce gas costs for settlement on Ethereum, while still achieving the same behaviour. The design moves the signature verification and validation of a checkpoint to an off-chain component reducing the on-chain costs with just sending a simple proof.

## Background

Polygon PoS contains 2 layers — bor (execution layer based on geth/erigon) and heimdall (consensus layer based on tendermint). In order to do settlement on Ethereum and bridging, heimdall submits checkpoint on regular intervals. It contains a merkle root hash of a tree generated from range of bor blocks and the message is signed by all validators and is later submitted to Ethereum. Basically, a checkpoint defines agreement of the majority (>2/3) of the validator set on a specific execution layer state.

## Motivation

Currently, roughly every 30 mins, a new checkpoint of avg. length of 512 bor blocks is submitted on-chain. A smart contract call is made, which validates the checkpoint and rewards the proposer and the validators who signed that checkpoint. The validation of checkpoint includes mainly these things:
- Checking the sequence of checkpoints
- Validating the signatures against the active validator set
- Checking if all of them together form a majority (>2/3) or not

While the logic seems trivial, the cost of verifying the signatures is very high.

### Status Quo

Below is the flame graph of a submit checkpoint call on ethereum mainnet.

![submit-checkpoint-flame-graph](./../assets/PIP-49/submit-checkpoint-flame-graph.png)

Some stats from the above graph
- Total gas used: 2.2M
- Check Signatures function: 1.9M (~86%)
- Gas Price: 14.489 gwei
- Total cost: 0.032 ETH ~ 84$
- Check Signatures cost: 0.028 ETH ~ 73$

A decent amount of time goes into SLOAD (as we read data for all validator) and ecrecover (i.e. signature verification). This is because of the large validator set (~105) on mainnet.

![submit-checkpoint-pie-chart](./../assets/PIP-49/submit-checkpoint-pie-chart.png)

As checkpointing is a regular and frequent operation, validators pay a lot of money in just submitting the checkpoint. Although there’s scope of optimisations to save gas, it's not significant.

## Specification

Using ZKPs for Checkpoint: With constructions of ZKPs (Zero Knowledge Proofs) becoming easier, it seems a viable option to leverage it to save the on-chain gas costs. Basically, use a zero knowledge proof (here we don’t really care about it being zero knowledge) which asserts that majority of the validator set voted upon a checkpoint in heimdall and offload the signature verification cost replacing it with a simple proof verification. As of now, we intend to use [SP1](https://github.com/succinctlabs/sp1) for generating ZK proofs.

***Prover / Circuit Design***

The goal is to design a circuit which reduces the possibility to generate false/fake proof i.e. the proof should be generate only and only for correct set of inputs and nothing else. Hence the inputs are chosen in a way which makes sure that they are validated against something and are provable. The circuit contains nothing but series of assertions. Here are the inputs to the circuit:

| Input | Description | Source | Operations / Validation |
| --- | --- | --- | --- |
| tx_data (String) | Amino marshalled bytes of the checkpoint transaction in heimdall.  | Tendermint endpoint of a heimdall node. | The message is decoded to get the checkpoint message. The contents are hashed to get the transaction hash.  |
| tx_hash (B256 / 32 bytes) | Transaction hash of the checkpoint transaction | Tendermint endpoint of a heimdall node. | Verify if this hash matches with the one calculated above.  |
| sigs (Vec<String>) | Array of signatures from each validator for the checkpoint transaction | Tendermint endpoint of a heimdall node. | N/A |
| signers (Vec<Address>) | Array of validator address which were used to sign | Tendermint endpoint of a heimdall node. | Verify if the signer actually exists on the staking contracts on L1. Verify if the validator address derived from the checkpoint message and signature above is the same or not.  |
| state_sketch_bytes (Vec<u8>) | EVM State Sketch Bytes (More details on SP1 Contract Call section below) for calls to fetch staking data + last checkpoint. | RPC Endpoint of corresponding L1 (ethereum mainnet or sepolia) | Used to fetch the staking + validator info along with last checkpoint. |
| root_chain_info_address: Address | Address of the root chain info contract (which gives us staking info based on active validator set and last checkpoint info) | RPC Endpoint of corresponding L1 (ethereum mainnet or sepolia) | Use this to fetch data from L1. |
| l1_block_hash (B256 / 32 bytes) | L1 block hash to be used for fetching data | Chosen by prover (currently any valid block can be used, later restricted by timestamp) | Used to make the call to L1 based on this block. |
| bor_block_hash (B256 / 32 bytes) | Block hash (of the end block of checkpoint) | RPC Endpoint of corresponding bor node | N/A |

***Commits / Public Inputs***

Public inputs are nothing but the set of inputs which the verifier needs to provide. This is to ensure that the prover doesn’t generate fake proofs and the proof has context. These are also called commits because the prover commits to these values. Hence, if the verifier provides the exact set of commit values, then and only then the proof verification passes (it fails in any other case). Here are the commits for checkpoint proof:

| Commit | Description |
| --- | --- |
| bor_block_number | New bor block number (which is being proved) (i.e. block number of the milestone end block) |
| bor_block_hash | New bor block hash (which is being proved) (i.e. block hash of the checkpoint end block) |
| l1_block_hash | The L1 block hash used to fetch the staking info.  |

***Pseudocode***

```python3
# Inputs
#  - tx_data
#  - tx_hash
#  - sigs
#  - signers
#  - state_sketch_bytes
#  - root_chain_info_address
#  - l1_block_hash
#  - bor_block_hash

# Construct checkpoint message
checkpoint = amino_unmarshal(tx_data)

# Verify the transaction hash
assert(hash(tx_data) == tx_hash)

# Fetch the last checkpoint info from l1
last_checkpoint_end_block = getLastCheckpointEndBlock(l1_block_hash, root_chain_info_address)

# Assert if we're generating proof for next checkpoint and nothing else
assert(checkpoint.start_block == last_checkpoint_end_block + 1)

# Assert if the number of signatures and signers are same (sanity check)
assert(sigs.len() == signers.len())

# Fetch the staking data from l1
(signers, powers, total_power) = getEncodedValidatorInfo(l1_block_hash, root_chain_info_address)

# Generate a map out of it for easy validation
validators_stakes = {signers[i]: powers[i] for i in range(len(signers)}

# Construct a message which was signed by validators
message = vec(1)
message.append(encode(checkpoint))

# Verify precommits
majority_power = 0
for i in range len(precommits):
		# Ensure that the signer is part of active validator set
		assert(signers[i] in validator_stakes.keys())

		# Verify if the signer address matches
		signer = extract_signer(keccak256(message, sigs[i]))
		assert(signer == signers[i])
		
		# Increment majority vote
		majority_power += power[i]
		
# Validate if we have reached 2/3+1 consensus
assert(majority_power > total_power / 3 * 2)

CheckpointProofCommit {
        bor_block_hash: input.bor_block_hash,
        l1_block_hash: input.l1_block_hash,
        bor_block_number: checkpoint.end_block,
    }

# Commit the values
return {
  bor_block_number, # bor block number
	bor_block_hash, # bor block hash
  l1_block_hash, # l1 block hash
}
```

***On-chain verification***

We generate a PLONK proof which can be verified on-chain using SP1’s verifier contract. If the proof was generated and verified correctly on-chain, we can be rest assured that majority of validators voted on this checkpoint and we can straight up skip the signature verification on the contract. We replace the ‘check signatures’ function with a simple PLONK verify call. While the PoC is still basic, we can tweak it to commit values which the contract will be able to provide (e.g. the next checkpoint’s start number). This simplifies the design even more.

## Initial Results

We deployed this PoC on sepolia to capture some initial costs and if we just compare the gas costs, they’re way better compared to the current ones. 

- PLONK Verify Call: 0.37M gas (~80% better compared to 1.9M signature verification earlier)

Here’s a sepolia transaction verifying checkpoint for amoy testnet: [0x3fb1....5a17](https://sepolia.etherscan.io/tx/0x3fb166bc84e4c2861a781bfb928042a8bf56a3411157328dc3b2b2d960e75a17)

Even though this is sepolia, PLONK verification is mostly stateless and hence we expect similar consumption on mainnet as well.

While we’re saving a good amount by reducing the on-chain costs, we can’t ignore the proving costs. These proofs were generated on Succinct’s Prover Network. One of the proof generation request for amoy can be found [here](https://explorer.succinct.xyz/proof/01jangnysme6ebmqr961qmwsyb). As seen, it takes about ~4 mins to generate a PLONK proof.

It takes ~16M cycles (which is a good metric to look at to analyse the costs and complexity of the circuit). While amoy has only 20 validators, this number may increase for mainnet it's seen before that while the costs are proportional to the computation, it’s not a linear graph. Using the proving costs on prover network, even the cost very high number of cycles is reasonable.

## Open Questions

The major open quesion is where to generate these proofs? Generating a PLONK proof requires good amount of resources (e.g. >128 gb of ram). There are multiple approaches to this.
- Validators prove locally (no external dependency but not practical for every validator)
- Validators use SP1's prover network (seems reasonable, will have to onboard all of them to the prover network)
- Prover run by Polygon or any trusted 3rd party (easiest to achieve, ensure we can submit checkpoint on someone else's behalf)

While these are not open questions, we need some though on these aspects of the desgin.
1. Proving time. Currently, proof generation takes ~4 mins. Changes in heimdall will be required to accomodate this waiting time.
2. Backup options. While this approach is great, we can't get rid of the old one. We'll have to make sure we design a good way to use the original submit checkpoint method if something goes wrong.

## Backwards Compatibility

This approach/design will be implemented in parallel to the existing checkpoint flow and hence it won't fully replace the existing one. Basically, to begin with, there will be 2 ways to submit checkpoints -- one will be the traditional route where every validation will happen on-chain and another will be via submitting a proof. In case of issues in submitting a proof, one will still have the original method as backup.

## Test Cases

As it's still in design and ideation phase, we have been able to carry manual tests only. In future, we intent to automate the testing process and test the approach on different cases. As of now, there's no concrete plans. This section will be updated once the design is concrete.

## Reference Implementation

Here's the [link to the PoC](https://github.com/manav2401/zk-checkpoint) which was used to derive these results

## FAQs

***Why not use the tendermint light client verifier?***

Succinct has a PoC for verifying tendermint blocks. It just takes 2 blocks as input and calls a verify function taken from the official tendermint light client. We can’t use this directly to prove consensus in our case because of side transactions in heimdall. Side transactions are included in block N and are finalised in block N+2. It is wrapped in a tendermint transaction and hence the outer transaction can be valid and the inner side transaction can still fail. Hence, just proving the block doesn’t prove consensus on a checkpoint/milestone message.

***How are we able to use L1 data inside prover?***

SP1 has developed a very simple to use library sp1-contract-call to generate proofs for any `eth_call`. What this essentially means is that if you want to derive something from a blockchain, let’s say L1, you can use that info inside your circuit along with a proof that the info being used is correct. The proof here simply is a storage proof. The operator of the service fetches data (any data from a contract at any block) using this library and constructs an EVM State Sketch for this call. We pass this sketch as an input and the prover also makes a similar call. The only difference is that it doesn’t make an actual call but reads required data from the sketch provided along with it’s proofs.

## Security Considerations

We need to make sure that the circuit is sound i.e. the prover cannot generate proof with fake/false inputs. The circuit needs to be audited. Once the design is concrete, more details can be shared.

## Copyright

All copyrights and related rights in this work are waived under [CC0 1.0 Universal]([CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode)).