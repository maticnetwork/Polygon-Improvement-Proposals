| PIP               | Title                           | Description          | Author                        | Discussion | Status | Type                                     | Date                  |
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|
| 26 | Transition from MATIC to POL Validator Rewards | Proposes aligning the initial POL Validator Rewards to current rewards schedule | [Derek Meyer](https://twitter.com/data_nexus) [Dimitri Nikolaros](https://twitter.com/dnikolaros)  | [Forum Post](https://forum.polygon.technology/t/pip-26-transition-from-matic-to-pol-validator-rewards/13046) | Peer Review  | Contracts | 2023-10-12

### Abstract

This PIP proposes aligning the initial POL Validator Rewards with the original [MATIC Validator Rewards Schedule](https://wiki.polygon.technology/docs/pos/design/validator/rewards/#what-is-the-incentive), introduced during the launch of MATIC in 2020, and ending in June 2025. The proposal is to continue, i.e. adhere to the current MATIC reward schedule, and, once it ends (in June 2025), switch to the proposed Polygon 2.0 reward schedule. This would honor the social contract, i.e. public commitment, with regards to Polygon validator rewards, and enable smooth transition to Polygon 2.0.
  
### Motivation

[PIP-17](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-17.md), which initiates the POL upgrade, introduces a 1% annual POL emission for validator rewards.  The authors propose to complete the original commitments of the genesis MATIC Validator Rewards Schedule, before commencing the aforementioned Polygon 2.0 validator reward schedule.
  
The original and proposed outline for emissions can be seen in the table below. 

*Proposed Emissions Schedule With Comparison:*

| Year    | Dates                         | Original Reward Pool | Original POL Proposed Reward Pool | PIP-26 Proposed Reward Pool |
| ------- | ----------------------------- | -------------------- | --------------------------------- | --------------------------- |
| First   | June 2020 - June  2021        | 312,917,639          | -                                 | -                           |
| Second  | July 2021 - July 2022         | 275,625,675          | -                                 | -                           |
| Third   | July 2022 - July 2023         | 246,933,140 | - | - |
| Fourth | July 2023 - July 2024 | 204,303,976 |\*134,700,000|\*201,400,000|
| Fifth  | June  2024 - June 2025 | 160,219,840          | 101,000,000                       | 150,300,000                 |
| Sixth | June 2025 -  June 2026 | 0 | 101,000,000 | 103,530,000 |

\*estimated using 4 months of original schedule, 8 months of newly proposed schedule.
  
### Specification

Adjust the implementation of DefaultEmissionManager.sol to follow the current MATIC validator reward schedule, gradually reducing the reward emission via subsequent PIPs until June 2025. In July 2025,  commence the Polygon 2.0 schedule of 1% annual POL emission for validator rewards.

### Backward Compatibility

This change causes no identifiable backward incompatibilities.
  
### References 

-   [Original Distribution of Staking Token Rewards](https:forum.polygon.technology/t/an-update-on-distribution-of-staking-token-rewards/9654/)
    
### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.


