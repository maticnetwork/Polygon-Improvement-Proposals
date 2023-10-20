| PIP| Title| Description| Author| Discussion | Status | Type | Date|
|-|-|-|-|-|-|-|-|
| 19 | Update Polygon PoS Native Token to POL| Upgrades the native token of the Polygon PoS network from MATIC to POL |Will Schwab, Daniel Gretzke, Dhairya Sethi, Zero Ekkusu| [Forum](https://forum.polygon.technology/t/pip-19-update-polygon-pos-native-token-to-pol/12914)| Last Call | Contracts | 2023-09-14 |**

    

### Abstract

This proposal calls for upgrading the native token of the Polygon PoS network from MATIC (`0x7d1afa7b718fb893db30a3abc0cfc608aacfebb0`) to POL (address) in a method that ensures maximum backward compatibility. The authors recommend this by upgrading the PoS Plasma Bridge Contract (`0x401f6c983ea34274ec46f84d70b31c151321188b`) to a new implementation that:

-   Converts all the MATIC in the bridge to POL 
-   Fulfills all withdrawal requests for the native asset of Polygon PoS Chain with POL 
-   Credits all deposits in POL or MATIC with the native asset of the Polygon PoS chain.

This proposed upgrade will not change any contracts on the Polygon PoS Network. Likewise, the native token’s properties will remain unchanged. All contracts will function as designed. The native asset of the Polygon PoS chain will change from being a claim on the Plasma Bridge’s MATIC, to a claim on the Plasma Bridge’s POL.

  

MATIC will still be an operational ERC20 token on Ethereum throughout this phase. Similarly, staking rewards for validators will remain denominated in MATIC until further technical upgrades are proposed and, if welcomed by the community, then implemented.

### Motivation

The native token is the token used by users of the Polygon PoS network to pay gas fees in order to transact. The present native token is redeemable for MATIC which was set as the native token upon genesis of the Polygon PoS network. Now that POL (address), the updated successor of MATIC, has been proposed, the authors propose setting it as the native token of the network via an upgrade of the Plasma Bridge in a maximally backward compatible manner.

### Definitions

Plasma Bridge: Polygon Plasma is the official bridge used to bridge MATIC tokens from Ethereum to Polygon chain. Withdrawals have a required 7-day delay. This delay is designed to allow users to challenge a malicious withdrawal.

### Specification

Upgrade the Plasma Bridge contract to a new implementation at < address to be determined >.

### Backward Compatibility

Contracts that expect to receive MATIC from the Plasma bridge on Ethereum may be affected. 

### Security Considerations

This upgrade adjusts a core contract of the Polygon Ecosystem, all proper security procedures including necessary audits will be taken, and should be carefully reviewed prior to activation.
 
### References

-   [PIP-17: Polygon Ecosystem Token (POL)](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md)
-   [PIP-18: Polygon 2.0 Phase 0 - Frontier](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-18.md)
    
### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
