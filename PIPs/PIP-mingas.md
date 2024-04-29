| PIP | Title          | Description                | Author                        | Discussion | Status      | Type                                     | Date                  |
|-----|----------------|----------------------------|------------------------------|------------|-------------|------------------------------------------|-----------------------|
|   | MinGas Increase | Increase minimum gas to 30gwei| [Marcello Ardizzone](https://https://github.com/marcello33)| | Draft | Core | 2024-04-29

### Abstract

A minimum fee for transaction gas price is required in both the p2p and execution layer of the network. In details, the miner gas price is the minimum tip given to miners to validate transactions, whilst the transactions pool price limit is the lower bound to accept a tx in the pool.

These parameters also prevent spam transactions from overburdening the mempool, which could create DDOS attack vectors. Moreover, a third parameter allows to define a gas price threshold below which the gas price oracle will ignore the transactions.

###  Motivation

Currently, heterogeneous implementations for such parameters mean that when a validator with a lower gas price is primary block producer, the required priority fee starts decreasing, as reflected in gas trackers.

Users can submit txs with lower gas price and such txs could get stuck in the txpool for a long time (if these txs are not immediately executed by the current validator), as the next validator/primary producer might require a much higher gas price.

This feature will help improve the UX and make it more uniform by doing away with client level config for such parameters, and instead adding them as part of the protocol hardcoded params, with a default value of `30gwei`.


### Specification

`txpool`, `eth` and `miner` modules are affected:

-   `txpool.pricelimit`: Minimum gas price limit to enforce the acceptance of txs into the pool (default: `30000000000`).

-    `miner.gasprice`: Minimum gas price for mining a transaction (default: `30000000000`).

-    `gpo.ignoreprice`: Gas price below which gpo will ignore transactions (default: `30000000000`).


When a new backend is initialized at startup in he client, some utility functions will check for such values, and enforce them to be `30gwei`.


### Backward Compatibility

This change causes no identifiable backward incompatibilities.

### Copyright

All copyrights and related rights in this work are waived under [CCO 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
