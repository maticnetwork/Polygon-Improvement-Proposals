| PIP              | Title                          | Description         | Author                       | Discussion | Status | Type                                    | Date                 |  
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|  
| 25 | Adjust POL Total Supply | Proposes burning POL tokens corresponding to previously burned MATIC | David Silverman, Paul Gebheim, Harry Rook, Mateusz Rzeszowski | [Forum](https://forum.polygon.technology/t/pip-25-adjust-pol-total-supply/13008) |  Last Call | Contracts |2023-10-04
 
### Abstract

In response to community feedback, this PIP proposes that the `burn()` function be called on the migrator contract, introduced in [PIP-17](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md), in order to burn an amount of POL equivalent to MATIC in the addresses currently used to permanently remove MATIC tokens from supply (ex: 0x000..dEaD) as well as other permanently inaccessible tokens, thus bringing the POL circulating supply inline with the MATIC supply.

### Motivation

[PIP-17](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md), which initiates the POL upgrade, proposes the minting of 10B POL to the migrator contract which corresponds to the entire original supply of MATIC. In order to accurately reflect the amount of previously burned MATIC in the POL supply, the authors propose the burning of corresponding POL.

### Specification

Call `burn()` on the migrator contract with an amount reflective of the MATIC balances of the following addresses:

-   0x000000000000000000000000000000000000dEaD \[MATIC burn address\] - 28,048,923.20944512682731682 MATIC 
-   0x7D1AfA7B718fb893dB30A3aBc0Cfc608AaCfeBB0 \[MATIC token address\] - 479,955.33752218388182171 MATIC 

*Exact values determined by the [Agra Hardfork](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-28.md)*

### Backward Compatibility

This change causes no identifiable backward incompatibilities. 

### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
