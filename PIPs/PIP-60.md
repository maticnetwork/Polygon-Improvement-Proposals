---
PIP: 60
Title: Increase gas limit to 45M
Author: Manav Darji, Sandeep Sreenath
Description: Proposes increasing block gas limit from 30M to 45M in the Polygon PoS network.
Discussion: https://forum.polygon.technology/t/pip-60-increase-block-gas-limit-to-36m/20610
status: Final
Type: Core
Date: 2025-02-11
---

## Abstract
Increase maximum gas from 30M to 45M to provide greater capacity, allowing for more transactions in a single block.

## Motivation
As the network continues to increase its usage, the demand for gas will increase, potentially creating a bottleneck for transaction volumes.

By increasing the gas limit by 50%, headroom is created for additional capacity to handle growing demand without significantly straining node hardware.

## Specification

Validators can implement this change at the client level without requiring consensus changes. This client-based change will allow the network to uniformly implement higher gas so that individual validators do not cause bottlenecks during a particular span. 

**Parameter Changes**:
- Current Block Gas Limit: 30,000,000 gas
- Proposed Block Gas Limit: 45,000,000 gas

To ensure uniformity, the newly proposed gas limit will be part of the execution layer client binary itself (instead of validators trying to explicitly set it via flag).

The default value in [miner config](https://github.com/maticnetwork/bor/blob/v2.0.1/miner/miner.go#L61) will be updated.

```diff
- GasCeil:  30_000_000,
+ GasCeil:  45_000_000,
```

## Backwards Compatibility
This change is non-breaking and backward-compatible.

## Security Considerations
The increased block gas limit may result in larger blocks requiring more time to propagate across the network. Testing will ensure that this change does not significantly increase uncle rates or network desynchronization.

Validators must be capable of handling the updated gas and ensure it does not exceed their CPU, memory, and disk resources.

## Copyright

All copyrights and related rights in this work are waived under [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
