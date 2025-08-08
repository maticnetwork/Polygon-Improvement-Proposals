---
PIP: 47
Title: Protocol Council Multi-Threshold Smart Contract Upgrades
Description: Proposes a multi-threshold upgradability mechanism for the Protocol Council
Author: Carlos Gonzalez Juarez, Javier Gonzalez de Chaves Garcia, Fabrice François, Harry Rook, Tanisha Katara, Mateusz Rzeszowski
Discussion: https://forum.polygon.technology/t/pip-47-protocol-council-multi-threshold-smart-contract-upgrades/19795
Status: Stagnant
Type: Contracts
Date: 2024-09-24
---

## Abstract

This proposal introduces a multi-threshold multisig system to the Protocol Council governing body that builds upon Aragon's OSx protocol, enhancing access control while ensuring extensibility, in line with [PIP-29: Polygon Protocol Council](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-29.md) specification.

The proposed contracts integrate the flexibility of multiple approval thresholds, improving upon traditional dual multisig setups by allowing fine-grained permissions and proposal execution control.

## Motivation

Current dual-contract multisig setups often introduce inefficiencies in access control by requiring additional roles and permissions for contract-based governance. This can lead to complexity in managing contracts and reduce flexibility in decision-making processes.

The proposed multi-threshold multisig offers a streamlined solution for the Protocol Council by eliminating the need for new roles and instead leveraging the existing permission structures within the system. This significantly reduces friction in access control management, making the system more efficient and easier to use.

Multi-threshold upgradeability also follows the specification laid out for the Protocol Council in [PIP-29](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-29.md):

>Regular, i.e., non-emergency, changes to the contracts are facilitated by a 7/13 (55%) consensus to maximize efficiency. These types of changes require a timelock delay of 10 days to ensure the ability for the community to exit the system before any change takes place.

>At the same time, an emergency route can facilitate immediate changes to system smart contracts in case of a critical issue. A timelock mechanism proves inefficient in such scenarios, as it’s unable to ensure any potential issue remains contained and is addressed immediately without opportunities for outside interference, due to the public nature of timelocked transactions. Consequently, an emergency route for changing system smart contracts is introduced, requiring a very high 10/13 consensus of the Protocol Council to execute any change without a timelock.

## Definitions

**Multi-Threshold Multisig**: A governance system that requires varying approval thresholds depending on the context of the proposal.

**Timelock**: A delay mechanism that ensures a waiting period before executing certain actions or proposals. This is primarily used for non-emergency proposals to provide time for community review.

**Aragon OSx**: A modular contract suite for decentralized decision governance and permission management written in Solidity. It allows for composable governance structures using plugins to easily upgrade functionalities without disrupting operations.

**Executor (DAO.sol)**: The contract responsible for holding permissions and executing actions. It verifies whether an address has the required permissions to perform a certain operation.

**Governor**: The multisig component of the system is responsible for creating proposals, gathering approvals, and executing actions once the required number of approvals is met. It works alongside the Executor to ensure the correct functioning of the multisig system.

**Plugin Setup Contract**: A contract that handles the installation and configuration of the multisig plugin within the executor contract (DAO.sol). It assigns permissions and ensures that all components of the multisig system are correctly connected.

**PermissionManager**: A part of the system that manages permissions for different roles and actions. It controls who can perform certain actions within the system, ensuring secure and decentralized access control.

**allowFailureMap**: A feature that allows certain actions within a proposal to fail without reverting the entire transaction. This provides more flexibility in the execution of complex proposals by permitting non-critical failures.

**Emergency Proposal**: A type of proposal that requires immediate execution due to a critical situation. It is subject to higher approval thresholds and may bypass the usual timelock or approval process to address urgent issues quickly.

## Specification

The multi-threshold multisig is built on top of Aragon’s OSx protocol, following the typical Solidity topology of Executor and Governor contracts with a Plugin Setup contract to ensure correct functioning of the plugin in the different stages of its lifecycle.

![|452x281](https://lh7-qw.googleusercontent.com/docsz/AD_4nXenjlFcNUEh3Wr0WlFDNjIOicJVLjLXn6NG7y0e6CH0DWQqPEE-HUEYFm4fot6mX8n0fGQM1Bp7rpw4uOozblBmFEaF7O2I6UbhgM8KHD-4y859Ivy8tR9e-ILVeo-KOzqokHOl6hDWIFkmTvrOPiSIqFw?key=b8h055Jyd32Zp6xNQ9BT6g)

The `DAO.sol` contract serves two main purposes:

* Hold the list of the permissions assigned
* Execute on behalf of the contracts that own that permission

The function `hasPermission()` checks if an address has permission on a contract via a permission identifier and considers if `ANY_ADDRESS` was used in the granting process.

```solidity
function hasPermission(

    address _where,

    address _who,

    bytes32 _permissionId,

    bytes memory _data

) external view returns (bool);
```

The input parameters descriptions are the following:

* `_where`: The address of the contract.
* `_who`: The address of an EOA or contract to give the permissions.
* `_permissionId`: The permission identifier.
* `_data`: The optional data passed to the `PermissionCondition` registered.

The second most important functionality in this contract is the `execute()` function doing so for a list of Actions. If a zero allow-failure map is provided, a failing action reverts the entire execution. If a non-zero allow-failure map is provided, allowed actions can fail without the entire call being reverted.

``` solidity
function execute(

    bytes32 _callId,

    Action[] memory _actions,

    uint256 _allowFailureMap

) external returns (bytes[] memory, uint256);
```

The input parameters descriptions are the following:

* `_callId`: The ID of the call. The definition of the value of `callId` is up to the calling contract and can be used, e.g., as a nonce.
* `_actions`: The array of actions.
* `_allowFailureMap`: A bitmap allowing execution to succeed, even if individual actions might revert. If the bit at index `i` is `1`, the execution succeeds even if `i` action reverts. A failure map value of `0` requires every action not to revert.

As stated before, the `DAO.sol` (also referred to as the Executor) is connected to the Multi-threshold Multisig (Governor), and it does so by the permissions stored in the first contract. These permissions are set at “installation” time by the `PluginSetup`, which tells the `DAO.sol` which permissions should be granted from whom and to who for the correct functioning of the system. It does so by deploying a specific instance of the Multisig contract for the `DAO.sol` and then using the `DAO.sol` Permission Manager to assign the required Operations.

``` solidity
// Prepare permissions

PermissionLib.MultiTargetPermission[]

memory permissions = new PermissionLib.MultiTargetPermission[](3);

// Set permissions to be granted.

// Grant the list of permissions of the plugin to the DAO.

permissions[0] = PermissionLib.MultiTargetPermission(

PermissionLib.Operation.Grant,

plugin,

_dao,

PermissionLib.NO_CONDITION,

multisigBase.UPDATE_MULTISIG_SETTINGS_PERMISSION_ID()

);

permissions[1] = PermissionLib.MultiTargetPermission(

PermissionLib.Operation.Grant,

plugin,

_dao,

PermissionLib.NO_CONDITION,

multisigBase.UPGRADE_PLUGIN_PERMISSION_ID()

);

// Grant `EXECUTE_PERMISSION` of the DAO to the plugin.

permissions[2] = PermissionLib.MultiTargetPermission(

PermissionLib.Operation.Grant,

_dao,

plugin,

PermissionLib.NO_CONDITION,

DAO(payable(_dao)).EXECUTE_PERMISSION_ID()

);
```

With that, we have the Executor connected to the Governor, that is, the `DAO.sol` contract with the Multisig Plugin.

Now, since `DAO.sol` doesn’t hold any logic by itself to tell it when to execute something, that’s delegated to the Multisig, which has the functionality to create proposals, approve them, wait, and confirm their execution.

``` solidity
function createProposal(

    bytes calldata _metadata,

    IDAO.Action[] calldata _actions,

    uint256 _allowFailureMap,

    bool _approveProposal,

    uint64 _startDate,

    uint64 _endDate,

    bool _emergency

) external returns (uint256 proposalId);
```

When creating a proposal, the multisig members are required to pass certain parameters in. The first, the metadata is an IPFS cid, in which members can pass any important offchain data that helps users understand the proposal being discussed. The file stored in IPFS should be in json format, and while it’s open for anyone to input whatever is desired, it is recommended having at least the following keys:

``` solidity
metadata: {

title: string;

summary: string;

type: string;

description?: string;

resources: Array<{

name: string;

url: string;

}>;

};
```

For the rest of the parameters of the proposal creation function:

* `_actions`: The actions that will be executed after the proposal passes.
* `_allowFailureMap`: A bitmap allowing the proposal to succeed, even if individual actions might revert. If the bit at index `i` is `1`, the proposal succeeds even if the `i` action reverts. A failure map value of `0` requires every action to not revert.
* `_approveProposal`: If `true`, the sender will approve the proposal.
* `_startDate`: The start date of the proposal.
* `_endDate`: The end date of the proposal.
* `_emergency`: Whether the proposal is an emergency proposal or not.

At this point, when the data is being stored, the plugin parameters get saved into the proposal itself, freezing any changes from misconfiguring the proposal. These plugin-level parameters are the following:

``` solidity 
struct MultisigSettings {

bool onlyListed = True;

uint16 minApprovals = 7;

uint16 emergencyMinApprovals = 10;

uint64 delayDuration = 10 days;

bool memberOnlyProposalExecution = True;

uint256 minExtraDuration = 7 days;

}
```

The plugin parameters are set by governance, and can be changed at the plugin level by a proposal passed by the council.

The parameters function as follows:

* `onlyListed`: Whether only listed members can create proposals.
* `minApprovals`: The minimal number of approvals required for a proposal to pass
* `emergencyMinApprovals`: The minimal number of approvals required for an emergency proposal to pass.
* `delayDuration`: The duration of the timelock between proposals has been approved and then confirmed at a later date.
* `memberOnlyProposalExecution`: Boolean to set if only multi-sig members should be allowed to execute once the proposal is finished and confirmed
* `minExtraDuration`: The minimal extra duration for members to approve and confirm the proposal. This prevents proposals from being created with a timespan that won’t allow the council to participate effectively.

Once a proposal has been created, the first stage starts, and it’s the same whether the proposal is an emergency one or not: enough approvals need to be gathered in time for the proposal to proceed.

``` solidity
address approver = _msgSender();

if (!_canApprove(_proposalId, approver)) {

revert ApprovalCastForbidden(_proposalId, approver);

}

Proposal storage proposal_ = proposals[_proposalId];

unchecked {

proposal_.approvals += 1;

}

proposal_.approvers[approver] = true;
```

Once the required number of approvals is met, there are two possible execution routes:

### Emergency

Can be executed by calling the `execute()` function:

```solidity
return

proposal_.parameters.emergency

? proposal_.approvals >= proposal_.parameters.emergencyMinApprovals

: (proposal_.approvals >= proposal_.parameters.minApprovals &&

proposal_.confirmations >= proposal_.parameters.minApprovals);
```

The proposal is not an emergency, and therefore, it requires following the normal flow in which members upload the secondary metadata and start the timelock specified by the plugin settings.

After the timelock has expired, a second confirmation is required from the multisig members.

```solidity
address approver = _msgSender();

if (!_canConfirm(_proposalId, approver)) {

revert ConfirmationCastForbidden(_proposalId, approver);

}

Proposal storage proposal_ = proposals[_proposalId];

unchecked {

proposal_.confirmations += 1;

}

proposal_.confirmation_approvers[approver] = true;
```

After that, the process reaches its final point, in which can be executed if the required number of members have approved and confirmed the proposal in time:

```solidity
Proposal storage proposal_ = proposals[_proposalId];
proposal_.executed = true;

_executeProposal(

dao(),

_proposalId,

proposals[_proposalId].actions,

proposals[_proposalId].allowFailureMap

);
```

This execution ties back to the initial step, where the `DAO.sol` contract receives a set of actions to execute, along with a failure map. The failure map allows for greater granularity in handling the atomicity of these executions.

## Backward Compatibility

The proposed multi-threshold multisig system introduces new functionalities that require upgrades to existing contracts.

Specifically, the new multisig system requires updates to the roles and permissions managed by existing [Protocol Council](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-29.md) multisig contracts.

## Security Considerations

The primary security challenge is ensuring that the system can handle varying approval thresholds and partial action failures without introducing vulnerabilities that could be exploited.

Additionally, the proposal system's staged approval mechanism, requiring both an initial and a final confirmation for non-emergency proposals, adds an additional layer of security by preventing premature execution of unauthorized actions.

To mitigate edge cases, the system includes expiry mechanisms for approvals and confirmations, ensuring that proposals cannot linger indefinitely in an approved state and become vulnerable to future exploits.

Further real-world testing will occur during phased deployments to ensure all potential issues are identified and mitigated.

## Copyright

All copyrights and related rights in this work are waived under [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
