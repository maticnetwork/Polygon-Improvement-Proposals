| PIP | Title          | Description                | Author                        | Discussion | Status      | Type                                     | Date                  |
|-----|----------------|----------------------------|-------------------------------|------------|-------------|------------------------------------------|-----------------------|
| 44 |Upgrade cosmos-sdk in heimdall| Proposes an upgrade for heimdall by bumping cosmos-sdk dependency | [Marcello Ardizzone](https://github.com/marcello33) | [Forum](https://forum.polygon.technology/t/pip-upgrade-cosmos-sdk-in-heimdall/17732)  | Draft | Core | 2024-06-27

### Abstract

Currently, Heimdall uses a fork of `cosmos-sdk`, which has been modified to support the specific needs of Polygon PoS, and is based on version `v0.37.4`. We propose a refactoring that will upgrade such dependency to target version  `v0.50.x`.

### Motivation

The primary motivation for such an upgrade is to solve a tech debt that anchors Heimdall to a `cosmos-sdk` version released around four years ago, at the time of the launch of the Polygon PoS network. Despite the continuous upgrades made throughout these years, the upgrade to `cosmos-sdk` will be beneficial for the PoS network for many technical factors.

### Specification

The new `cosmos-sdk` version will come with a set of new features that Heimdall can leverage. Amongst them:
1. A new application layer with a brand new set of functionalities, which can be leveraged by the Polygon PoS network.
2. New modules —considered the core of the `cosmos-sdk` applications— that will enhance the user experience of PoS users.
3. The architecture is very simplified and more robust, and the composition of modules has been enhanced to provide different services (e.g. `protobuf` services, `gRPC` query services, `autocli`, new REST routes).
4. The new `cosmos-sdk` fully leverages the capabilities of `cometBFT`'s ABCI++, enhancing transactions and blocks executions.
5. The upgrade to a more recent version will allow more frequent upstream merges, which means more functionalities to leverage and a higher level of security.

The upgrade process will be possible via a `cosmos-sdk` based hard fork. This is not a synonym of a PoS HF (or, generally speaking, a EVM-based HF). Specifically, the whole network (validators, non-validating full nodes, seed nodes, light nodes, etc) is stopped in a coordinated fashion at a particular height that has previously been agreed upon.

Since the whole network stops at the same height, the application's database has the same content across all nodes. Once the network has halted on the predetermined height, the application's database (an IAVL tree) is exported into a file in JSON format. The resulting JSON file, which should be exactly the same at all nodes in the network (the file's checksum can be used to verify this), is used as the initial application's state in a new genesis file. This genesis file is then used to spawn the generation of a new chain (different chain ID). Before the new chain is started, the JSON file has to undergo some sanity checks and needs to be adapted to be compatible with the new version of the `cosmos-sdk`.

The advantages of a hard fork (compared to a mere DB migration) are the following:
- Less complex and error-prone, since there's no need to:
    - migrate DBs (except for the adaptations that may need to be done on the exported JSON file).
    - translate blocks, or solve block format compatibility.
    - deal with any database at `CometBFT` level: the new chain starts with a "clean slate" `CometBFT`.
- Not going off the beaten path:
    - not a purely theoretical approach, but one that has been proven to work on the field, as this has been executed by `cosmos-sdk` chains several times, going from cosmos hub version 1 (used at time of PoS creation) to version 4 (current version).
- Less effort to adapt the data, since:
    - only the application's database may need to be adapted
    - `CometBFT` databases are not migrated

These are drawbacks of a hard fork which are summarised below and discussed in more detail under security considerations:
- The system needs to halt for several hours.
    - From the moment the old chain is halted until the new one starts, the Heimdall application is effectively down.
    - The duration of the downtime depends on the actions that need to be performed before the new chain can be started.
    - Such duration heavily depends on the size of the application's database, and the sanity checks to run (e.g. in case of the `cosmos-sdk` Stargate upgrade, the outage lasted around 6 hours)
- Other applications (and possibly smart contracts) that depend on the old chain (e.g. bor) need to make provisions
    - To support or absorb the downtime, and
    - To change to the new chain when the new chain is up.
- If the application needs to access data both before and after the migration, it will be more complex.
    - Both chains need to be up; the old one halted, acting as archive.
    - The application needs to use the migration height to route queries to the right chain.
- The history previous to the migration point is lost.
    - Block data, validators' data, old versions of key-value pairs stored in the application's DB.
    - Inability to query a node (`CometBFT` RPC, or `cosmos-sdk` gRPC) for data prior to the hard fork.
        - As a workaround, the community may agree to keep a few nodes, called Heimdall-archive-nodes, serving RPC requests for the old chain's data.
        - These are full nodes running the old software, using the old databases, but no validator is started, so the (old) chain stays halted.
        - Notice that those archive nodes represent a distinct chain (using a different chain ID) and therefore don't communicate with nodes of the new chain.

The proposed solution - which relies on other upgrades, involves several steps:
1. Completion of migration from `peppermint` to `cometBFT`
2. Embedding `cometBFT` into the new Heimdall application
3. Rewriting of Heimdall modules based on `cosmos-sdk v0.50.x`
4. Test the standalone application locally
5. Audit the code
6. Adapt Bor to work with the new Heimdall
7. Modify the PoS testing tools
8. Release of a `beta` version
9. Simulate the migration on devnets
10. Execute the coordinated upgrade on testnets (Mumbai and Amoy)
11. Execute a coordinated upgrade for mainnet


### Backwards Compatibility

The upgrade is incompatible with the current Heimdall chain. This means that launching a newer version of `cosmos-sdk` within Polygon PoS will only be possible by migrating Heimdall to a brand new chain.

### Security considerations

The upgrade involves a complete refactoring of the Heimdall codebase. Hence, Heimdall will undergo a meticulous phase of tests, audits, and performance reviews. Despite the many new functionalities offered by the new library, this migration will be—at first—a 1:1 migration, keeping intact the Heimdall capabilities and utilities.

During the migration process, the generation of the new genesis file is a delicate moment. The file is an unencrypted JSON file, so considerations are to be made to mitigate any risks, such as:

* Create migration-time estimates on a backup of the live data
* Rehearse in advance the whole process in a testnet to ensure validators and operators are prepared
* Publish in advance and audit all the tools (scripts, steps to follow, commands to run, etc.) that node operators need to use in order to generate the genesis file by themselves
* Encourage all node operators to generate the genesis file on their own
* Publish a "ceremonial" checksum of the genesis file so that those operators that produce the genesis file by themselves can check whether their file contents match the "official" hash
* In case of a large number of operators obtaining conflicting hashes, a mechanism should be in place to investigate what is causing the hash mismatch (whether a tampering attempt or just a bug in the tools for generating the genesis file)
* These last two points are crucial, as all `CometBFT` full nodes (validators included) must start a new chain from an exact same copy of the genesis file. Otherwise, the first block of the chain will run into an `app_hash` mismatch at proposal time.


### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
