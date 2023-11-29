| PIP               | Title                           | Description          | Author                        | Discussion | Status | Type                                     | Date                  |
|-------------------|---------------------------------|----------------------|-------------------------------|------------|--------|------------------------------------------|-----------------------|
| 30 | Increase Max Code Size Limit to 32KB | Proposal to increase Max Code Size Limit from 24KB to 32KB | Mudit Gupta, David Silverman (@oneski22) | [Forum](https://forum.polygon.technology/t/pip-30-increase-max-code-size-limit-to-32kb/13266) | Draft | Core | 2023-11-29 |

### Abstract

This PIP proposes increasing the `MAX_CODE_SIZE` Limit set in [EIP-170](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-170.md) from `0x6000 (2**14+2**13)` to `0x8000 (2**15)`.

### Motivation

The author proposes raising the limit to `0x8000` to allow for the deployment of more complex contracts without needing to resort to alternative development patterns.
  
### Specification

At block `FORK_BLKNUM`, the `MAX_CODE_SIZE` limit changes from `0x6000` bytes to `0x8000` bytes. If contract creation initialization returns data with length of more than `0x8000` bytes, contract creation fails with an out of gas error.

### Backward Compatibility

This change causes no identifiable backward incompatibilities.  

### Security Considerations 

Added in the Spurious Dragon Ethereum Hard Fork, `MAX_CODE_SIZE` was introduced to cap an attack vector where a quadratic action, O(n) of reading code from disk, was charged a constant gas. At the time the limit `0x6000` was set as there were no contracts of that size. Since that time, contracts have gotten more complex and frequently brush up against that limit, leading to the need to use various different methods of reducing contract size, some of which can introduce vulnerabilities.

Storage mediums have gotten faster since this limit was introduced so fetching large data is easier. Furthermore, 32KB fits perfectly in two 16 KB I/O blocks or a single 32 KB I/O block.

### References

-   [EIP-170: Contract code size limit](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-170.md)
  
### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.  
