---
PIP: 45
Title: Token Symbol change
Description: Proposes token symbol be changed from MATIC to POL
Author: Simon Dosch, Dhairya Sethi
Discussion: https://forum.polygon.technology/t/pip-45-pos-mrc-20-token-symbol-change/19437
Status: Peer Review
Type: Core
Date: 2024-25-07
---

## Abstract

This proposal suggests changing the token symbol for the native token of the Polygon PoS chain from "MATIC" to "POL."  
The proposed modification will be implemented in the  
MATIC Token contract at the address: "0x0000000000000000000000000000000000001010",  
as well as WMATIC": "0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270.

## Motivation

As the Polygon PoS network upgrades more broadly to utilize POL, an update to the MATIC token symbol is required to ensure this upgrade is operationalized across all key infrastructure, allowing the upgrade to be effected efficiently for users and service providers.

## Specification

Changes to "[MRC20.sol](https://polygonscan.com/address/0x0000000000000000000000000000000000001010)" (1010)


```diff
function name() public pure returns (string memory) {

return "Polygon Ecosystem Token";

}

function symbol() public pure returns (string memory) {

return "POL";

}
```

"MRC20.sol" inherits from "EIP712.sol", where the following changes have been made:


```diff
string internal constant EIP712_DOMAIN_NAME = "Polygon Ecosystem Token";


string internal constant EIP712_DOMAIN_VERSION = "2";

```

**This change will have as a consequence that calls to `transferWithSig` will need to adhere to the new EIP712 domain name and version from now on.**

Changes to "[WMATIC.sol](https://polygonscan.com/address/0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270)" (1270):


```diff
string public name = "Wrapped Polygon Ecosystem Token";


string public symbol = "WPOL";

```

Both of these changes will be made by replacing the bytecode at the contract address via a hard fork.

## Backward Compatibility

This PIP is not backward compatible with the current implementation of Bor and will, therefore, require a hard fork.

## Security Considerations

Changing the token symbol involves updates at the smart contract level, necessitating a thorough review and testing phase. This process will include comprehensive testing and possibly third-party audits to ensure that the change does not introduce vulnerabilities or disrupt the network's stability.

Given the reliance of exchanges, wallets, and other infrastructure on the MATIC symbol, there is potential for significant disruptions if the change is not handled carefully. To mitigate this, broad network buy-in and coordination are essential.

## Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
