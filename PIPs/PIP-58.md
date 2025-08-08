---
PIP: 58
Title: Increase BaseFeeChangeDenominator to 64
Author: Jerry Chen
Description: Proposes a change in the BaseFeeChangeDenominator from 16 to 64
Discussion: https://forum.polygon.technology/t/pip-58-increase-basefeechangedenominator-to-64/20596
status: Final
Type: Core
Date: 2025-02-08
---

## Abstract 
This proposal seeks to increase the BaseFeeChangeDenominator (the maximum rate at which the base fee can be adjusted between blocks) on the Polygon PoS network from 16 to 64. Increasing this value will reduce the volatility of gas prices during periods of high demand.

## Motivation 
Polygon PoS chain introduced EIP-1559 as part of the London Hardfork, although this upgrade has significantly reduced the occurrence of gas spikes, certain network conditions can still lead to significant increases in gas in a fairly short timeframe. 

The initial parameters of EIP-1559 were changed in the Delhi hardfork with the BaseFeeChangeDenominator increased from 8 to 16. 

Polygon PoS has 2-second block times, so currently with the BaseFeeChangeDenominator set to 16, the base fee can increase by up to 6.25% per block, leading to a 1000% increase in 80 seconds. 


## Specification
We propose increasing the BaseFeeChangeDenominator from its current value of 16 to 64. This change will halve the maximum base fee change per block, smoothing out gas price volatility.

### Pre and Post-Change Values

- `BaseFeeChangeDenominatorPre = 16`
- `BaseFeeChangeDenominatorPost = 64`

### Updated Logic

```go
func BaseFeeChangeDenominator(borConfig *BorConfig, number *big.Int) uint64 {
    if borConfig.IsUpgradeBlock(number) {
        return BaseFeeChangeDenominatorPostUpgrade
    } else {
        return BaseFeeChangeDenominatorPreUpgrade
    }
}
```

## Backwards Compatibility

This change is not compatible with the protocol's existing consensus checks and will require a hardfork. On the infrastructure level, this change is fully backward-compatible with existing wallet and applications. The core functionality of EIP-1559 will remain intact, and no modifications are required on the part of users or applications.

## Security Considerations

This change does not introduce new security risks. The adjustment only alters the rate of base fee changes, ensuring smoother fee dynamics without affecting consensus or transaction validity.

In the long term, this could lead to increased strain on node hardware as this change in effect reduces a countermeasure (gas fees) to short-term capacity spikes. 

## Copyright

All copyrights and related rights in this work are waived under [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
