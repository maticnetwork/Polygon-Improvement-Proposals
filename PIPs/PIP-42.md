| PIP               | Title                           | Description          | Author                        | Discussion | Status | Type                                     | Date                  |
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|
| 42 | PIP-42: Polygon 2.0 - Upgrade PoS Staking to Use POL | Upgrade to Emissions Manager contract for Direct POL Emissions | Simon Dosch, Zero Ekkusu, David Silverman, Paul Gebheim, Paul O'Leary | [Forum](https://forum.polygon.technology/t/pip-41-enable-direct-pol-emissions-to-stakemanager-sol/17642) | Peer Review | Contracts | 2024-06-25 |

## Abstract

This proposal calls for upgrading the staking contract for the Polygon PoS network to change from using MATIC (0x7d1afa7b718fb893db30a3abc0cfc608aacfebb0) as the primary staking token to POL (address) in a method that ensures maximum backward compatibility. The authors recommend this by upgrading the PoS Stake Manager Contract (0x5e3Ef299fDDf15eAa0432E6e66473ace8c13D908) to a new implementation at (address to be determined) that:

* Converts all the MATIC in the Stake Manager to POL
* Uses POL for all new staking and unstaking requests
* Has legacy functions that allow for staking and unstaking using MATIC by leveraging the POL Migration Contract (0x29e7DF7b6A1B2b07b731457f499E1696c60E2C4e) to preserve maximum backward compatibility.

This proposed upgrade will not change any contracts on the Polygon PoS Network. Likewise, all other properties about staking (reward rate, unbonding period, slashing etc.) will remain unchanged.

## Motivation

The staking token is the token which stakers and validators of the Polygon PoS network use to secure the network, and come to consensus on the state. The present staking token is MATIC which was set as the staking token upon genesis of the Polygon PoS network. Now that POL (0x455e53CBB86018Ac2B8092FdCd39d8444aFFC3F6) is live, the authors propose setting it as the staking token of the network via an upgrade of the Stake Manager in a maximally backward compatible manner.

## Specification

Upgrade the Stake Manager contract to a new implementation at < address to be determined >.

## Backward Compatibility

Systems that expect to stake and unstake MATIC from the PoS Staking system will be affected unless they move to using the new legacy functions.

## Security Considerations

This upgrade adjusts a core contract of the Polygon PoS network. All proper security procedures including necessary audits will be taken, and should be carefully reviewed prior to activation.

## References

* [PIP-17: Polygon Ecosystem Token (POL)](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md)
* [PIP-18: Polygon 2.0 Phase 0 - Frontier](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-18.md)

## Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
