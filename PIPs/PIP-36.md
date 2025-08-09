---
PIP: 36
Title: Replay Failed State Syncs
Description: Enable replay of failed state sync transactions
Author: Dhairya Sethi @DhairyaSethi, Simon Dosch (@simonDos)
Discussion: https://forum.polygon.technology/t/pip-36-replay-failed-state-syncs/13864
Status: Final
Type: Core
Date: 2024-4-30 
---

## Abstract

Since the gas repricing of the [Berlin Hardfork](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-2070.md), a bug exists on the [PoS Matic Contract](https://github.com/maticnetwork/contracts/blob/main/contracts/child/MRC20.sol#L44) which reverts the deposit state sync txn on L2 and irreversibly locks MATIC on the L1 Plasma bridge if the receiver on L2 is not an EOA, but a contract with a receive fallback containing any amount of code.

It is important to note that this issue solely affects the MATIC token. To address this issue, we propose patching the bug & introducing the ability to replay failed state syncs.

## Rationale

> TLDR: Plasma MATIC deposits revert with out of gas when the receiver is a GnosisSafe contract because it uses [transfer instead of call](https://github.com/maticnetwork/contracts/blob/main/contracts/child/MRC20.sol#L44) when sending MATIC (native on child) and the fallback required more than 2300 gas (and more recently [2600](https://help.safe.global/en/articles/40813-why-can-t-i-transfer-eth-from-a-contract-into-a-safe)).

MATIC deposits occur via the plasma bridge, wherein a user holding MATIC on L1 calls **`depositERC20()`** on the Plasma bridge **`DepositManager`**. This action emits a State Sync, which is picked up by validators and executed on L2. This state sync triggers the **`StateReceiver`** on L2, which, after the necessary validations, calls the **`deposit`** method on the childToken (MATIC(0x00..1010) in this instance).

```solidity
    function deposit(address user, uint256 amount) public onlyOwner {
	... // perform necessary checks and log effects

        // interaction: transfer amount to user
        _user.transfer(amount); // <-- the bug

		...
    }
```

This contract utilizes the built-in Solidity transfer method, which limits the available transferred gas to 2300. Before the Berlin Hardfork, this amount was generally sufficient for most contracts. However, post-Berlin Hardfork, accessing a contract for the first time (which occurs when MATIC is moved to an account that is a contract) incurs gas costs of 2600, resulting in a failure with an `out of gas` revert.

- Gas costs before Berlin hardfork

  Access account: 700 gas

  Read storage slot: 800 gas

- Gas costs after Berlin hardfork

  Access account:

  First time per address in tx: 2600 gas

  Any additional time in tx: 100 gas

  Read storage slot:

  First time per storage slot in tx: 2100 gas

  Any additional time in tx: 100 gas

This issue also exists in Ethereum, leading to the inclusion of [EIP-2930](https://github.com/ethereum/ercs/blob/master/ERCS/erc-2390.md) alongside [EIP-2929](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-2929.md) to mitigate such occurrences. EIP-2930 introduces an access list, prepaying gas for any address or storage slot on the list, thus reducing runtime costs to 100 gas. This solution addresses the problem of insufficient gas availability when transferring ETH from one contract to another using default Solidity methods.

**Note** that this is a special case for the MATIC token only and not any other [ERC-20](https://github.com/ethereum/ercs/blob/master/ERCS/erc-20.md) token as this token has the unique property of being a native token on L2.

## Specification

This PIP proposes to patch the transfer vs send bug mentioned above by updating the bytecode of MRC-20 MATIC token on L2 through a hardfork. We specify a **`txGasLimit`** while doing the call such that the receiver doesn’t consume all available gas in the state sync block limit to prevent DoS (state sync’s are batched together in one txn).

```solidity
      (bool success, ) = _user.call(value: amount, gas: txGasLimit);
      require(success, "!transfer");
```

**Replay Failed State Syncs**

Along with the above, this PIP proposes two more additions for the replayability of failed state syncs.

The State Receiver currently emits `StateCommitted(uint stateId, bool success)`.

```solidity
  function commitState(uint256 syncTime, bytes calldata recordBytes) external onlySystem returns(bool success) {
    ...

    if (isContract(receiver)) { // notify state receiver contract, in a non-revert manner
      uint256 txGas = 5000000;
      bytes memory data = abi.encodeWithSignature("onStateReceive(uint256,bytes)", stateId, stateData);
      assembly {
        success := call(txGas, receiver, 0, add(data, 0x20), mload(data), 0, 0)
      }
      emit StateCommitted(stateId, success);
    }
  }

```

We propose the StateReceiver also stores the failed state syncs.

mapping failedStateSyncs (uint stateId ⇒ bytes encodedData (abi.encode(receiver, data)).

which can be re-executed once by anyone through an external method `replayFailedStateSync(uint stateId)`

This would delete the entry in failedStateSyncs mapping only on a successful call.

This enabled users with failed state syncs to replay their transaction with

- higher gas, and
- at a different time (where the state has changed & possibly the control flow if receiver succeeds).

**Replay Failed State Syncs**

The State Receiver currently emits `StateCommitted(uint stateId, bool success)`.

> TLDR: make a merkle tree of previous failed state syncs, post immutable root on StateReceiver, such that anyone can prove their failed state sync and replay them on chain trustlessly.

A merkle tree of previous failed state syncs from the Shanghai Hardfork (50563600) until the present (an ever-expanding list) is created.

A package to fetch and filter historic events will be made public such that this tree can be built, and root cross-checked by anyone. This will be a merkle tree of depth 16, where the leaf’s will be the keccak(abi.encode(stateId, receiver, calldata)) \[stateId provides sufficient collision resistance].

An external method on StateReceiver will be exposed: `replayHistoricFailedStateSync(bytes32[16] calldata proof, uint256 leafIndex, uint256 stateId, address receiver, bytes calldata data)` which nullifies the leaf on successful state syncs.

## Observability

The overall observability of the plasma bridge would increase with this update.
The main improvement is that failed state syncs would be recorded.

## Backward Compatibility

As the contract impacted by this PIP is not upgradeable, the changes would require a hardfork.

The L2 contracts being upgraded would be the following:

- StateReceiver: 0x0000000000000000000000000000000000001001
- MRC-20(Matic Token): 0x0000000000000000000000000000000000001010

## Security Considerations

Internal as well as external security reviews will have to be done to ensure maximum safety. Limited gas will be provided during the MATIC token internal call while depositing native tokens, and `Check Effects` pattern must be followed during on `replayStateSync`.

## Copyright

All copyrights and related rights in this work are waived under [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
