PIP: 17
Title: Polygon Ecosystem Token (POL)
Description: Upgrade to MATIC in the form of the Polygon Ecosystem Token (POL)
Author: Mihailo Bjelic, Mudit Gupta, Will Schwab, Daniel Gretzke, Dhairya Sethi, Ankit Maity, Harry Rook (@hrook1), Mateusz Rzeszowski
Discussion: https://forum.polygon.technology/t/pip-17-polygon-ecosystem-token-pol/12912
Status: Final
Type: Contracts
Date: 2023-09-14
---
 
### Abstract

This proposal describes an upgrade to MATIC (0x7d1afa7b718fb893db30a3abc0cfc608aacfebb0) in the form of the Polygon Ecosystem Token (POL). POL is the upgraded native token of Polygon 2.0, along with its accompanying contracts and initial configurations to handle emission management and token migration. POL allows for a one-to-one migration with MATIC with an initial supply of 10 billion POL and yearly emission of 2% that will be equally distributed to stakers and a community treasury contract.


### Motivation

The token of the Polygon PoS chain, MATIC, powered this single chain that allowed Ethereum to scale during times of high network congestion. Polygon 2.0 is the next iteration in the Ethereum scaling journey, with zero-knowledge proofs (“zk”) facilitating the expansion of Ethereum block-space across a multitude of L2 chains whilst also inheriting its security.  

POL represents a next-generation token able to accommodate an ecosystem of zk-based Layer 2 chains by enabling the following utility:

* Staking,
* Community ownership, and
* Governance.

### Specification

#### POL Token Contract

A token contract, POL, is proposed, broadly based on the MIT-licensed OpenZeppelin ERC20 implementations which provide support for the default ERC20 standard, along with some non-standard functions for allowance modifications. The implementation also provides support for EIP-2612: Signature-Based Permit Approvals.

Upon genesis, an initial supply of 10 billion will be minted to a migration contract (see below for details). Further mints may be called by an emission manager contract (see below for details). Emission Managers can be managed by Governance. An additional check-in mint function requires the mint rate to be less than 10 POL per second.

The POL token contract is not upgradeable, however Permit2 default approvals can be enabled or disabled by Governance and is enabled by default.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.21;

import {ERC20, ERC20Permit, IERC20} from "openzeppelin-contracts/contracts/token/ERC20/extensions/ERC20Permit.sol";
import {AccessControlEnumerable} from "openzeppelin-contracts/contracts/access/AccessControlEnumerable.sol";
import {IPolygonEcosystemToken} from "./interfaces/IPolygonEcosystemToken.sol";

/// @title Polygon ERC20 token
/// @author Polygon Labs (@DhairyaSethi, @gretzke, @qedk, @simonDos)
/// @notice This is the Polygon ERC20 token contract on Ethereum L1
/// @dev The contract allows for a 1-to-1 representation between $POL and $MATIC and allows for additional emission based on hub and treasury requirements
/// @custom:security-contact security@polygon.technology
contract PolygonEcosystemToken is ERC20Permit, AccessControlEnumerable, IPolygonEcosystemToken {
    bytes32 public constant EMISSION_ROLE = keccak256("EMISSION_ROLE");
    bytes32 public constant CAP_MANAGER_ROLE = keccak256("CAP_MANAGER_ROLE");
    bytes32 public constant PERMIT2_REVOKER_ROLE = keccak256("PERMIT2_REVOKER_ROLE");
    address public constant PERMIT2 = 0x000000000022D473030F116dDEE9F6B43aC78BA3;
    uint256 public mintPerSecondCap = 13.37e18;
    uint256 public lastMint;
    bool public permit2Enabled;

    constructor(
        address migration,
        address emissionManager,
        address protocolCouncil,
        address emergencyCouncil
    ) ERC20("Polygon Ecosystem Token", "POL") ERC20Permit("Polygon Ecosystem Token") {
        if (
            migration == address(0) ||
            emissionManager == address(0) ||
            protocolCouncil == address(0) ||
            emergencyCouncil == address(0)
        ) revert InvalidAddress();
        _grantRole(DEFAULT_ADMIN_ROLE, protocolCouncil);
        _grantRole(EMISSION_ROLE, emissionManager);
        _grantRole(CAP_MANAGER_ROLE, protocolCouncil);
        _grantRole(PERMIT2_REVOKER_ROLE, protocolCouncil);
        _grantRole(PERMIT2_REVOKER_ROLE, emergencyCouncil);
        _mint(migration, 10_000_000_000e18);
        // we can safely set lastMint here since the emission manager is initialised after the token and won't hit the cap.
        lastMint = block.timestamp;
        _updatePermit2Allowance(true);
    }

    /// @inheritdoc IPolygonEcosystemToken
    function mint(address to, uint256 amount) external onlyRole(EMISSION_ROLE) {
        uint256 timeElapsedSinceLastMint = block.timestamp - lastMint;
        uint256 maxMint = timeElapsedSinceLastMint * mintPerSecondCap;
        if (amount > maxMint) revert MaxMintExceeded(maxMint, amount);

        lastMint = block.timestamp;
        _mint(to, amount);
    }

    /// @inheritdoc IPolygonEcosystemToken
    function updateMintCap(uint256 newCap) external onlyRole(CAP_MANAGER_ROLE) {
        emit MintCapUpdated(mintPerSecondCap, newCap);
        mintPerSecondCap = newCap;
    }

    /// @inheritdoc IPolygonEcosystemToken
    function updatePermit2Allowance(bool enabled) external onlyRole(PERMIT2_REVOKER_ROLE) {
        _updatePermit2Allowance(enabled);
    }

    /// @dev The permit2 contract has full approval by default. If the approval is revoked, it can still be manually approved.
    function allowance(address owner, address spender) public view override(ERC20, IERC20) returns (uint256) {
        if (spender == PERMIT2 && permit2Enabled) return type(uint256).max;
        return super.allowance(owner, spender);
    }

    /// @inheritdoc IPolygonEcosystemToken
    function version() external pure returns (string memory) {
        return "1.1.0";
    }

    function _updatePermit2Allowance(bool enabled) private {
        emit Permit2AllowanceUpdated(enabled);
        permit2Enabled = enabled;
    }
}
```

#### Migration Contract

The migration contract will accept two addresses, one for the MATIC token and one for the POL token respectively.

The contract shall receive the entire initial 10 billion POL supply in order to allow 1-to-1 swaps for the entire 10 billion MATIC supply.

Governance can lock and unlock the ability to unmigrate POL tokens back into an equivalent amount of MATIC.

This contract is upgradeable via Governance with POL tokens also being burnable via Governance. The initial implementation is described below.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.21;

import {IERC20} from "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
import {IERC20Permit} from "openzeppelin-contracts/contracts/token/ERC20/extensions/IERC20Permit.sol";
import {SafeERC20} from "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
import {Ownable2StepUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/Ownable2StepUpgradeable.sol";
import {IPolygonMigration} from "./interfaces/IPolygonMigration.sol";

/// @title Polygon Migration
/// @author Polygon Labs (@DhairyaSethi, @gretzke, @qedk)
/// @notice This is the migration contract for Matic <-> Polygon ERC20 token on Ethereum L1
/// @dev The contract allows for a 1-to-1 conversion from $MATIC into $POL and vice-versa
/// @custom:security-contact security@polygon.technology
contract PolygonMigration is Ownable2StepUpgradeable, IPolygonMigration {
    using SafeERC20 for IERC20;
    using SafeERC20 for IERC20Permit;

    IERC20 public immutable matic;
    IERC20 public polygon;
    bool public unmigrationLocked;

    modifier onlyUnmigrationUnlocked() {
        if (unmigrationLocked) revert UnmigrationLocked();
        _;
    }

    constructor(address matic_) {
        if (matic_ == address(0)) revert InvalidAddress();
        matic = IERC20(matic_);
        // so that the implementation contract cannot be initialized
        _disableInitializers();
    }

    function initialize() external initializer {
        __Ownable_init();
    }

    /// @notice This function allows owner/governance to set POL token address *only once*
    /// @param polygon_ Address of deployed POL token
    function setPolygonToken(address polygon_) external onlyOwner {
        if (polygon_ == address(0) || address(polygon) != address(0)) revert InvalidAddressOrAlreadySet();
        polygon = IERC20(polygon_);
    }

    /// @inheritdoc IPolygonMigration
    function migrate(uint256 amount) external {
        emit Migrated(msg.sender, amount);

        matic.safeTransferFrom(msg.sender, address(this), amount);
        polygon.safeTransfer(msg.sender, amount);
    }

    /// @inheritdoc IPolygonMigration
    function unmigrate(uint256 amount) external onlyUnmigrationUnlocked {
        emit Unmigrated(msg.sender, msg.sender, amount);

        polygon.safeTransferFrom(msg.sender, address(this), amount);
        matic.safeTransfer(msg.sender, amount);
    }

    /// @inheritdoc IPolygonMigration
    function unmigrateTo(address recipient, uint256 amount) external onlyUnmigrationUnlocked {
        emit Unmigrated(msg.sender, recipient, amount);

        polygon.safeTransferFrom(msg.sender, address(this), amount);
        matic.safeTransfer(recipient, amount);
    }

    /// @inheritdoc IPolygonMigration
    function unmigrateWithPermit(
        uint256 amount,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external onlyUnmigrationUnlocked {
        emit Unmigrated(msg.sender, msg.sender, amount);

        IERC20Permit(address(polygon)).safePermit(msg.sender, address(this), amount, deadline, v, r, s);
        polygon.safeTransferFrom(msg.sender, address(this), amount);
        matic.safeTransfer(msg.sender, amount);
    }

    /// @inheritdoc IPolygonMigration
    function updateUnmigrationLock(bool unmigrationLocked_) external onlyOwner {
        emit UnmigrationLockUpdated(unmigrationLocked_);
        unmigrationLocked = unmigrationLocked_;
    }

    /// @inheritdoc IPolygonMigration
    function version() external pure returns (string memory) {
        return "1.1.0";
    }

    /// @inheritdoc IPolygonMigration
    function burn(uint256 amount) external onlyOwner {
        polygon.safeTransfer(0x000000000000000000000000000000000000dEaD, amount);
    }

    uint256[49] private __gap;
}
```

#### Emission Manager Contract

The emission manager contract has the exclusive ability to mint new POL tokens and is implemented to distribute the tokens as follows:

#### Validator Rewards 

Emissions are minted to the PoS staking contract (0x5e3ef299fddf15eaa0432e6e66473ace8c13d908) for staking rewards. After minting the appropriate amount of POL, the migration contract will be used to ensure that staking rewards continue to be paid out in MATIC, maximizing backward compatibility.

Year 1-3:

For years 1-3, the POL emissions schedule will follow the schedule defined in [PIP-26](https://github.com/maticnetwork/Polygon-Improvement-Proposals/blob/main/PIPs/PIP-26.md).

After Year 3:

-   1% annual compounding. 

#### Community Treasury

-   1% annual compounding emission is minted to a community treasury. Upon the deployment of contracts introduced in this proposal, a multi-signature wallet (0x2ff25495d77f380d5F65B95F103181aE8b1cf898) will be used to safeguard the funds until Community Board supervision is enacted.
    
A publicly callable function that is callable by any address can trigger the immediate minting of all vested emission.

This contract is upgradeable via Governance. The initial implementation is described below.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.21;

import {IPolygonEcosystemToken} from "./interfaces/IPolygonEcosystemToken.sol";
import {IPolygonMigration} from "./interfaces/IPolygonMigration.sol";
import {IDefaultEmissionManager} from "./interfaces/IDefaultEmissionManager.sol";
import {Ownable2StepUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/Ownable2StepUpgradeable.sol";
import {SafeERC20} from "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
import {PowUtil} from "./lib/PowUtil.sol";

/// @title Default Emission Manager
/// @author Polygon Labs (@DhairyaSethi, @gretzke, @qedk, @simonDos)
/// @notice A default emission manager implementation for the Polygon ERC20 token contract on Ethereum L1
/// @dev The contract allows for a 3% mint per year (compounded). 2% staking layer and 1% treasury
/// @custom:security-contact security@polygon.technology
contract DefaultEmissionManager is Ownable2StepUpgradeable, IDefaultEmissionManager {
    using SafeERC20 for IPolygonEcosystemToken;

    uint256 public constant INTEREST_PER_YEAR_LOG2 = 0.04264433740849372e18;
    uint256 public constant START_SUPPLY = 10_000_000_000e18;
    address private immutable DEPLOYER;

    IPolygonMigration public immutable migration;
    address public immutable stakeManager;
    address public immutable treasury;

    IPolygonEcosystemToken public token;
    uint256 public startTimestamp;

    constructor(address migration_, address stakeManager_, address treasury_) {
        if (migration_ == address(0) || stakeManager_ == address(0) || treasury_ == address(0)) revert InvalidAddress();
        DEPLOYER = msg.sender;
        migration = IPolygonMigration(migration_);
        stakeManager = stakeManager_;
        treasury = treasury_;

        // so that the implementation contract cannot be initialized
        _disableInitializers();
    }

    function initialize(address token_, address owner_) external initializer {
        // prevent front-running since we can't initialize on proxy deployment
        if (DEPLOYER != msg.sender) revert();
        if (token_ == address(0) || owner_ == address(0)) revert InvalidAddress();

        token = IPolygonEcosystemToken(token_);
        startTimestamp = block.timestamp;

        assert(START_SUPPLY == token.totalSupply());

        token.safeApprove(address(migration), type(uint256).max);
        // initial ownership setup bypassing 2 step ownership transfer process
        _transferOwnership(owner_);
    }

    /// @inheritdoc IDefaultEmissionManager
    function mint() external {
        uint256 currentSupply = token.totalSupply(); // totalSupply after the last mint
        uint256 newSupply = inflatedSupplyAfter(
            block.timestamp - startTimestamp // time elapsed since deployment
        );
        uint256 amountToMint = newSupply - currentSupply;
        if (amountToMint == 0) return; // no minting required

        uint256 treasuryAmt = amountToMint / 3;
        uint256 stakeManagerAmt = amountToMint - treasuryAmt;

        emit TokenMint(amountToMint, msg.sender);

        IPolygonEcosystemToken _token = token;
        _token.mint(address(this), amountToMint);
        _token.safeTransfer(treasury, treasuryAmt);
        // backconvert POL to MATIC before sending to StakeManager
        migration.unmigrateTo(stakeManager, stakeManagerAmt);
    }

    /// @inheritdoc IDefaultEmissionManager
    function inflatedSupplyAfter(uint256 timeElapsed) public pure returns (uint256 supply) {
        uint256 supplyFactor = PowUtil.exp2((INTEREST_PER_YEAR_LOG2 * timeElapsed) / 365 days);
        supply = (supplyFactor * START_SUPPLY) / 1e18;
    }

    /// @inheritdoc IDefaultEmissionManager
    function version() external pure returns (string memory) {
        return "1.1.0";
    }

    uint256[48] private __gap;
}
```
### Backward Compatibility

This proposal does not change any active systems on either the Polygon PoS or Polygon zkEVM networks. All existing contracts will function as previously designed.

### Security Considerations

In the event of an exploit, to prevent arbitrary amounts of POL being minted, there is a variable hard cap on the maximum amount of tokens allowed to be minted per second. It is initialized at a maximum allowed number of 10 POL to be minted per second.

Due to the size, complexity, and importance of the proposed POL contracts, there will be several internal and external code audits to ensure the security of the implementation detailed above.

### References

* [POL Whitepaper](https://polygon.technology/papers/pol-whitepaper/)
* [ERC-20: Token Standard](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md/)
* [EIP-2612: Permit Extension for EIP-20 Signed Approvals](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-2612.md)

### Implementation 

* [https://github.com/0xPolygon/pol-token](https://github.com/0xPolygon/pol-token) 

### Copyright

All copyrights and related rights in this work are waived under CC0 1.0 Universal.
