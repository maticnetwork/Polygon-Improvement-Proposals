---
pip: 20
title: State-Sync Verbosity
description: Proposal to introduce more visibility of state-sync transactions
author: Shivam Sharma (@0xsharma), Sandeep Sreenath (@ssandeep), William Schwab (@wschwab)
discussion: https://forum.polygon.technology/t/pip-20-state-sync-verbosity/13050
status: Peer Review
type: Core
date: 2023-09-14
---

## Abstract

This proposal specifies the addition of event emission in the state-receiver genesis contract on PoS network which can give information on the success of the execution of the stateIDs in a state-sync transaction. It will also enable easier mappings with the `Ethereum TxHash <-> Bor-TxHash` using heimdall-api as a medium.

## Motivation

Presently, in the event that a stateID encounters failure within the child chain at the `execution` level owing to inadequate implementation of the recipient contract by a Dapp Developer, no logging or alerts are generated to signal such occurrences. This situation has the potential to impede developers' progress and lead to the oversight of bugs. The proposed PIP (Protocol Improvement Proposal) addresses this issue by enhancing the observability of state-sync transactions, thereby rectifying the aforementioned challenges.

## Specification

#### The event is defined as :
```
event StateCommitted(uint256 indexed stateId, bool success)
```

#### The event is emitted in :
```
if (isContract(receiver)) {
	uint256 txGas = 5000000;
	bytes memory data = abi.encodeWithSignature("onStateReceive(uint256,bytes)", stateId, stateData);

	assembly {
		success := call(txGas, receiver, 0, add(data, 0x20), mload(data), 0, 0)
	}

	emit StateCommitted(stateId, success);
}
```
#### Genesis changes to upgrade contract on target blockNum :
```
{
	config{
	....
		bor{
		....
			"blockAlloc": {
				"<BLOCK_NUMBER>": {
					"<STATE_RECEIVER_CONTRACT_ADDRESS>": {
						"balance": "0x0",
						"code": "<NEW_BYTECODE>"
					}
				}
			},
		....
		}
	....
	}
}
```

## Backwards Compatibility

This PIP will not be backward compatible with the current implementation of Bor and will therefore require a Hard Fork.

## Reference Implementation

- https://github.com/maticnetwork/genesis-contracts/pull/71
- https://github.com/maticnetwork/bor/pull/947


## Copyright

All copyrights and related rights in this work are waived under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
