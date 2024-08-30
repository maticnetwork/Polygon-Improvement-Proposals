---
PIP: 14
Title: Checkpoint Buffer Time
Description: Increase in Checkpoint Buffer Time
Author: Vaibhav Jindal (@VAIBHAVJINDAL3012), Sandeep Sreenath (@ssandeep)
Discussion: https://forum.polygon.technology/t/pip-14-increase-checkpoint-buffer-time/12369
Status: Final
Type: Core
Date: 2023-07-13
---

### Abstract
This proposal describes a change to the value of the ‘Checkpoint Buffer Time’ in Heimdall, from 1000 seconds to 1500 seconds.

Currently the parameter ‘Checkpoint Buffer Time’ is used at the following two places:

In the handler of ‘Checkpoint Msg’, where it checks whether ‘Checkpoint Buffer Time’ has passed since the last buffered checkpoint (if the buffered checkpoint exists), and;

In the handler of ‘No Ack ‘, where it checks whether ‘Checkpoint Buffer Time’ has passed since the last stored checkpoint.

### Motivation
Following ‘the Merge’ on the checkpoint rootchain (Ethereum), transaction finality time increased from approx 750 seconds to 1150 seconds. This delayed the average time between each checkpoint on Polygon PoS, commensurate with this increased delay in finality on Ethereum.

The proposed change will bring the ‘Checkpoint Buffer Time’ in line with the current average time between checkpoints.

We propose to execute this change via the Heimdall Governance Module, which would avoid the coordination overhead associated with hard forking the chain.

### Specification
        {
            "subspace": "checkpoint",
            "key": "CheckpointBufferTime",
            "value": "1500000000000"
        }

### Backward Compatibility
This PIP will not be backward compatible with the current implementation of Bor and Heimdall. It would therefore require a hard fork, or, synchronous upgrade via the Heimdall Governance Module.

### Copyright
All copyrights and related rights in this work are waived under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/legalcode).
