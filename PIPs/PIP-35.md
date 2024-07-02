| PIP | Title          | Description                    | Author                        | Discussion | Status      | Type                                     | Date                  |
|-----|----------------|--------------------------------|------------------------------|------------|-------------|------------------------------------------|-----------------------|
| 35  | MinGas Increase | Increase minimum gas to 25gwei | [Marcello Ardizzone](https://https://github.com/marcello33)| [Forum](https://forum.polygon.technology/t/pip-for-min-gas-increase-in-polygon-pos/13856)  | Last Call | Core | 2024-04-30

### Abstract

A minimum fee for transaction gas price is required in both the p2p and execution layers of the network. The miner gas price is the minimum tip given to miners to validate transactions, while the transaction pool price limit is the lower bound to accept a tx in the pool. 

These parameters also prevent spam transactions from overburdening the mempool, which could create DDOS attack vectors. Moreover, a third parameter allows the network to define a gas price threshold, below which the gas price oracle will ignore transactions.

###  Motivation

Currently, heterogeneous implementations for such parameters mean that when a validator with a lower gas price is the primary block producer, the required priority fee starts decreasing, as reflected in gas trackers.

Users can submit txs with a lower gas price and such txs could get stuck in the txpool for a long time (if these txs are not immediately executed by the current validator), as the next validator/primary producer might require a much higher gas price.

This feature will help improve the UX and make it more uniform by eliminating client-level config for such parameters and instead adding them as part of the protocol hardcoded params, with a default value of `25gwei`.


### Specification

`txpool`, `eth` and `miner` modules are affected:

-   `txpool.pricelimit`: Minimum gas price limit to enforce the acceptance of txs into the pool (default: `25000000000`).

-   `miner.gasprice`: Minimum gas price for mining a transaction (default: `25000000000`).

-   `gpo.ignoreprice`: Gas price below which gpo will ignore transactions (default: `25000000000`).


When a new backend is initialized at startup in he client, some utility functions will check for such values, and enforce them to be `25gwei`.


### Backward Compatibility

This change causes no backward incompatibilities.

### Copyright

All copyrights and related rights in this work are waived under [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
