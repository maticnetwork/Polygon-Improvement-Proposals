| PIP               | Title                           | Description          | Author                        | Discussion | Status | Type                                     | Date                  |
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|
| 41 | Enable Direct POL Emissions to StakeManager.sol | Upgrade to Emissions Manager contract for Direct POL Emissions | Simon Dosch, Harry Rook | [Forum]() | Peer Review | Contracts | 2024-06-24 |

## Abstract

Currently, to support validator rewards on the Polygon PoS network, a StakeManager contract is used to facilitate distribution in accordance with the MATIC issuance schedule.

In order to maintain compatibility in the short term, the POL token [EmissionManager.sol](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md#emission-manager-contract) contract was set up in such a way that POL would be un-migrated to MATIC before being sent to the StakeManager.

## Motivation

This proposed change will remove the un-migration of POL to MATIC in the EmissionManager.sol contract, allowing for the direct transfer of newly minted POL to the StakeManager contract.

Specification

The proposed upgrade does not require a large change in the current codebase and can be achieved via two lines of code in the EmissionManager.sol contract.

Current:

```
// backconvert POL to MATIC before sending to StakeManager

migration.unmigrateTo(stakeManager, stakeManagerAmt);
```

Would be replaced with:

```
_token.safeTransfer(stakeManager, stakeManagerAmt);
```

## Backward Compatibility

The change will require a timely upgrade to the StakeManager.sol contract to fully support POL, so that stakers can also withdraw POL.

## Security Considerations

The change in code is not complex, however, due to the importance of the POL contracts within the network, both an internal, as well as an external audit were done to ensure the security of the implementation.

## Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
