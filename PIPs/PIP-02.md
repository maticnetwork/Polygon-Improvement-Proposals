---
PIP: 2
Title: Proposal to distribute and decentralize staking power across the Polygon chain
Description: Proposal to increase the minimun commission.
Author: Delroy Bosco
Discussion: https://forum.polygon.technology/t/pip-2-proposal-to-distribute-and-decentralize-staking-power-across-the-polygon-chain/219/11
Status: Stagnant
Type: Contracts
Date: 2021-11-01
---

### Abstract:

Polygon has seen an explosion in staking in the over one year mainnet has been active. All 100 validator slots are now filled, but the distribution of delegated MATIC is clustered around large validators with low or 0% commission. Unfortunately, this has caused centralization and puts the network at risk. By increasing the minimum commission, the validators, delegators, and the network can benefit:

### Motivation:

The top 10 validators have received 82% of the total delegation. Out of those validators, only the Binance node has a commission greater than 4%, with 7 out of 10 having a 0% commission. These low commissions incentivize additional delegations, which further centralizes the network. 0% commission also removes incentive for validators to keep a high uptime, which further threatens network security.
Increasing the minimum commission charged will give validators more incentive to utilize professional hardware and maintain the highest possible uptime. Increasing the minimum commission will also give delegators more nodes to choose from at the same reward level. This will certainly help to decentralize the network. A decentralized network is more secure, leading to safety of funds for both delegators and the Polygon network as a whole. Properly funding validators will allow nodes to invest in performant hardware, which will allow a greater uptime percentage. With greater uptime percentage, fewer checkpoints will be missed and that will translate into more block rewards for delegators.

### Specification:

Currently, there is only a MAX_COMMISSION_RATE set to 100 in the Matic Stake Manager 7 contract. A new contract would be required to include a MIN_COMMISSION_RATE, possibly as a configurable variable for easy update in the future. In the below code, this MIN_COMMISSION_RATE is created, along with the code enforcing the new minimum.

```
uint256 constant MIN_COMMISSION_RATE = 5;

uint256 constant MAX_COMMISSION_RATE = 100;

...

function updateCommissionRate(uint256 validatorId, uint256 newCommissionRate) external {

...

require(newCommissionRate >= MIN_COMMISSION_RATE, "Incorrect value");

require(newCommissionRate <= MAX_COMMISSION_RATE, "Incorrect value");

validators[validatorId].commissionRate = newCommissionRate;

}

```

A minimum of a 5% commission was selected as enough room for the average validator to maintain their servers, but not too much that it starts to cut into delegators rewards. This is also in line with many other PoS blockchains, where 0% commission is not as common. Tezos’ bakers average 8-15%, Harmony One validators average 2.5-10%, Persistence One validators average 5-10%, and Solana validators average 10%, to name a few.

### Rationale:

**A common misconception about the Polygon network is that staking rewards are distributed when a validator creates a block or sends a checkpoint, and delegators will get more rewards by staking with larger validators. This isn’t completely accurate, as any validator that is online to validate a checkpoint will receive rewards. Rewards are sent to validators and delegators based on their total stake in the network. Therefore, outside of commission, the only variables in rewards are how much a delegator stake, and the uptime of the validator. Not only does staking with smaller validators provide the same rewards, it encourages better decentralization of the network.**

Other options explored:

- Putting a max cap on each validator’s total stake
- Decreasing rewards if a validator’s total stake surpasses a threshold
- Requiring a validator to stake a percentage of the total stake on their node, which incentivizes good behavior among larger validators

### Backwards Compatibility:

There are no backwards compatibility issues in terms of functionality, as having a validator change their commission is already built in. However, the current contract would not be usable as there is no way to enforce a minimum commission, so a contract would need to be deployed.

### **Security Considerations:**

Although there is not much risk in implementing a minimum commission, any modification or new deployment of a contract inherits risks of bugs. Issues with the updateCommissionRate should only be limited to allowing validators to choose any number for their commission, and should not extend into other parts of the contract or Polygon ecosystem.

---

### References

[Polygon Forum Discussion](https://forum.polygon.technology/t/pip-2-proposal-to-distribute-and-decentralize-staking-power-across-the-polygon-chain/219/1)
