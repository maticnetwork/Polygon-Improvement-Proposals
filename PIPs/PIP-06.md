---
pip: 6
title: Change in BaseFeeChangeDenominator
description: Proposes a change in the BaseFeeChangeDenominator
author: Shivam Sharma, Sandeep Sreenath (@ssandeep), Paul O’Leary
discussion: https://forum.polygon.technology/t/pip-6-change-in-basefeechangedenominator/10875/4
status: Final
type: Core
date: 2023-01-10
---

### Motivation

Polygon PoS chain saw a major update in the beginning of 2022 when EIP-1559 was rolled out on Mainnet as part of the London Hardfork [(EIP1559 (London Fork) on Mainnet](https://forum.polygon.technology/t/eip1559-london-fork-on-mainnet/549)) which changed the on-chain gas price dynamics. Although it works well majority of the time, during high demand, we have seen huge gas spikes due to rapid increase in the base fee. We propose to smoothen the change in base fee by changing the BaseFeeChangeDenominator.

To know more about the implementation of EIP-1559 and its effects on Polygon, you can refer to [Impact of EIP1559 and Future Possibilities](https://forum.polygon.technology/t/impact-of-eip1559-and-future-possibilities/1749)

To summarize, the main reasons for gas spikes happening during high demand are:

Exponential Gas pricing due to EIP-1559 and the current basefee: On Ethereum, a series of constant full blocks will increase the gas price by a factor of 10 every ~20 blocks (~4.3 min on average). But on the PoS chain, since the block time is 2 secs, the baseFee goes 10x in just 40 seconds. A user’s transaction becomes ineligible due to high gasPrice right after the particular block it was fired in. As the network utilization goes high, the baseFee starts climbing and the transaction is stuck in pending till the baseFee comes back down and the transaction becomes eligible. Also, gas SPIKES faster than applications (wallets, etc) can keep up / make predictions. This results in poor user experience during peak times.

Bad contracts: Occasionally poor design choices by dapp developers will unintentionally create a DDoS type effect on the network and litter the transaction mempool with tons of transactions driving up demand for blockspace resulting in higher gas prices for everyone.

### Specification

We propose increasing the BaseFeeChangeDenominator from the current value of 8 to 16. This will smoothen the increase(/decrease) in baseFee when the gas used in blocks is higher(/lower) than the target gas limit. After this change, the rate of change will decrease to 6.25% (100/16) as compared to the current 12.5% (100/8).

BaseFeeChangeDenominatorPreDelhi = 8 // Bounds the amount the base fee can
change between blocks before Delhi Hard Fork.

BaseFeeChangeDenominatorPostDelhi = 16 // Bounds the amount the base fee can
change between blocks after Delhi Hard Fork.

```
func BaseFeeChangeDenominator(borConfig *BorConfig, number *big.Int) uint64 {
	if borConfig.IsDelhi(number) {
		return BaseFeeChangeDenominatorPostDelhi
	} else {
		return BaseFeeChangeDenominatorPreDelhi
	}
}
```
### Test Cases

Example 1: (when network utilisation is full):

```
block0 basefeePerGas : x
block0 utilisation = 100 percent
```

For block1 :
	
    With baseFeeChangeDenominator = 8 ( default )
	block1 baseFeePerGas : 112.5% of x

	With baseFeeChangeDenominator = 16 ( proposed )
	block1 baseFeePerGas : 106.25% of x
    
For block10 :
	
    With baseFeeChangeDenominator = 8 ( default )
	block1 baseFeePerGas : 324.73% of x

	With baseFeeChangeDenominator = 16 ( proposed )
	block1 baseFeePerGas : 183.35% of x
For block20 :
	
    With baseFeeChangeDenominator = 8 ( default )
	block1 baseFeePerGas : 1054.51% of x

	With baseFeeChangeDenominator = 16 ( proposed )
	block1 baseFeePerGas : 336.19% of x

For block50 :
	
    With baseFeeChangeDenominator = 8 ( default )
	block1 baseFeePerGas : 36109% of x

	With baseFeeChangeDenominator = 16 ( proposed )
	block1 baseFeePerGas : 2072.27% of x
 
Example 2 : (when network utilisation is 0):

```
block0 basefeePerGas : x
block0 utilisation = 0 percent
```

For block1 :
	
    With baseFeeChangeDenominator = 8 ( default )
	block1 baseFeePerGas : 87.5% of x

	With baseFeeChangeDenominator = 16 ( proposed )
	block1 baseFeePerGas : 93.75% of x
For block10 :
	
    With baseFeeChangeDenominator = 8 ( default )
	block1 baseFeePerGas : 26.31% of x

	With baseFeeChangeDenominator = 16 ( proposed )
	block1 baseFeePerGas : 52.45% of x
For block20 :
	
    With baseFeeChangeDenominator = 8 ( default )
	block1 baseFeePerGas : 6.92% of x

	With baseFeeChangeDenominator = 16 ( proposed )
	block1 baseFeePerGas : 27.51% of x
For block50 :
	
    With baseFeeChangeDenominator = 8 ( default )
	block1 baseFeePerGas : 0.13% of x

	With baseFeeChangeDenominator = 16 ( proposed )
	block1 baseFeePerGas : 3.97% of x

---

### References

- [Pre-PIP Discussion: Addressing Reorgs and Gas Spikes 2](https://forum.polygon.technology/t/pre-pip-discussion-addressing-reorgs-and-gas-spikes/10623/25) 
- [Polygon Forum Discussion](https://forum.polygon.technology/t/pip-6-change-in-basefeechangedenominator/10875/4) 
