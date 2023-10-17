| PIP| Title| Description| Author| Discussion | Status | Type | Date|
|-|-|-|-|-|-|-|-|
|17| Polygon Ecosystem Token (POL) | Upgrade to MATIC in the form of the Polygon Ecosystem Token (POL) |Mihailo Bjelic, Mudit Gupta, Will Schwab, Daniel Gretzke, Dhairya Sethi, Ankit Maity, Harry Rook, Mateusz Rzeszowski| [Forum](https://forum.polygon.technology/t/pip-17-polygon-ecosystem-token-pol/12912) | Peer Review | Contracts | 2023-09-14 |
 
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

Upon genesis, an initial supply of 10 billion will be minted to a migration contract (see below for details). Further mints may be called by an emission manager contract (see below for details). An additional check-in mint function requires the mint rate to be less than 10 POL per second.

The POL token contract is not upgradeable.

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.21;

import {ERC20, ERC20Permit} from "openzeppelin-contracts/contracts/token/ERC20/extensions/ERC20Permit.sol";
import {AccessControlEnumerable} from "openzeppelin-contracts/contracts/access/AccessControlEnumerable.sol";
import {IPolygonEcosystemToken} from "./interfaces/IPolygonEcosystemToken.sol";

/// @title Polygon ERC20 token
/// @author Polygon Labs (@DhairyaSethi, @gretzke, @qedk)
/// @notice This is the Polygon ERC20 token contract on Ethereum L1
/// @dev The contract allows for a 1-to-1 representation between $POL and $MATIC and allows for additional emission based on hub and treasury requirements
/// @custom:security-contact security@polygon.technology
contract PolygonEcosystemToken is ERC20Permit, AccessControlEnumerable, IPolygonEcosystemToken {
    bytes32 public constant EMISSION_ROLE = keccak256("EMISSION_ROLE");
    bytes32 public constant CAP_MANAGER_ROLE = keccak256("CAP_MANAGER_ROLE");
    uint256 public mintPerSecondCap = 10e18; // 10 POL tokens per second
    uint256 public lastMint;

    constructor(
        address migration,
        address emissionManager,
        address governance
    ) ERC20("Polygon Ecosystem Token", "POL") ERC20Permit("Polygon Ecosystem Token") {
        if (migration == address(0) || emissionManager == address(0) || governance == address(0))
            revert InvalidAddress();
        _grantRole(DEFAULT_ADMIN_ROLE, governance);
        _grantRole(EMISSION_ROLE, emissionManager);
        _grantRole(CAP_MANAGER_ROLE, governance);
        _mint(migration, 10_000_000_000e18);
        // we can safely set lastMint here since the emission manager is initialised after the token and won't hit the cap.
        lastMint = block.timestamp;
    }

    /// @notice Mint token entrypoint for the emission manager contract
    /// @dev The function only validates the sender, the emission manager is responsible for correctness
    /// @param to Address to mint to
    /// @param amount Amount to mint
    function mint(address to, uint256 amount) external onlyRole(EMISSION_ROLE) {
        uint256 timeElapsedSinceLastMint = block.timestamp - lastMint;
        uint256 maxMint = timeElapsedSinceLastMint * mintPerSecondCap;
        if (amount > maxMint) revert MaxMintExceeded(maxMint, amount);

        lastMint = block.timestamp;
        _mint(to, amount);
    }

    /// @notice Update the limit of tokens that can be minted per second
    /// @param newCap the amount of tokens in 18 decimals as an absolute value
    function updateMintCap(uint256 newCap) external onlyRole(CAP_MANAGER_ROLE) {
        emit MintCapUpdated(mintPerSecondCap, newCap);
        mintPerSecondCap = newCap;
    }
}
```
#### Migration Contract

The migration contract will accept two addresses, one for the MATIC token and one for the POL token respectively.

The contract shall receive the entire initial 10 billion POL supply in order to allow 1-to-1 swaps for the entire 10 billion MATIC supply.

Governance can lock and unlock the ability to unmigrate POL tokens back into an equivalent amount of MATIC. 

This contract is upgradeable via Governance. The initial implementation is described below.

```
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

    IERC20 public polygon;
    IERC20 public matic;
    bool public unmigrationLocked;

    modifier onlyUnmigrationUnlocked() {
        if (unmigrationLocked) revert UnmigrationLocked();
        _;
    }

    constructor() {
        // so that the implementation contract cannot be initialized
        _disableInitializers();
    }

    function initialize(address matic_) external initializer {
        __Ownable_init();
        if (matic_ == address(0)) revert InvalidAddress();
        matic = IERC20(matic_);
    }

    /// @notice This function allows owner/governance to set POL token address *only once*
    /// @param polygon_ Address of deployed POL token
    function setPolygonToken(address polygon_) external onlyOwner {
        if (polygon_ == address(0) || address(polygon) != address(0)) revert InvalidAddressOrAlreadySet();
        polygon = IERC20(polygon_);
    }

    /// @notice This function allows for migrating MATIC tokens to POL tokens
    /// @dev The function does not do any validation since the migration is a one-way process
    /// @param amount Amount of MATIC to migrate
    function migrate(uint256 amount) external {
        emit Migrated(msg.sender, amount);

        matic.safeTransferFrom(msg.sender, address(this), amount);
        polygon.safeTransfer(msg.sender, amount);
    }

    /// @notice This function allows for unmigrating from POL tokens to MATIC tokens
    /// @param amount Amount of POL to migrate
    function unmigrate(uint256 amount) external onlyUnmigrationUnlocked {
        emit Unmigrated(msg.sender, amount);

        polygon.safeTransferFrom(msg.sender, address(this), amount);
        matic.safeTransfer(msg.sender, amount);
    }

    /// @notice This function allows for unmigrating POL tokens (from msg.sender) to MATIC tokens (to account)
    /// @param amount Amount of POL to migrate
    /// @param account Address to receive MATIC tokens
    function unmigrateTo(address account, uint256 amount) external onlyUnmigrationUnlocked {
        emit Unmigrated(msg.sender, amount);

        polygon.safeTransferFrom(msg.sender, address(this), amount);
        matic.safeTransfer(account, amount);
    }

    /// @notice This function allows for unmigrating from POL tokens to MATIC tokens using an EIP-2612 permit
    /// @param amount Amount of POL to migrate
    function unmigrateWithPermit(
        uint256 amount,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external onlyUnmigrationUnlocked {
        emit Unmigrated(msg.sender, amount);

        IERC20Permit(address(polygon)).safePermit(msg.sender, address(this), amount, deadline, v, r, s);
        polygon.safeTransferFrom(msg.sender, address(this), amount);
        matic.safeTransfer(msg.sender, amount);
    }

    /// @notice Allows governance to lock or unlock the unmigration process
    /// @dev The function does not do any validation since governance can update the unmigration process if required
    /// @param unmigrationLocked_ New unmigration lock status
    function updateUnmigrationLock(bool unmigrationLocked_) external onlyOwner {
        unmigrationLocked = unmigrationLocked_;
        emit UnmigrationLockUpdated(unmigrationLocked_);
    }

    /**
     * @dev This empty reserved space is put in place to allow future versions to add new
     * variables without shifting down storage in the inheritance chain.
     * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
     */
    uint256[50] private __gap;
}
```

#### Emission Manager Contract

The emission manager contract has the exclusive ability to mint new POL tokens and is implemented to distribute the tokens as follows: 

-   1% annual compounding emission is minted to the PoS staking contract (0x5e3ef299fddf15eaa0432e6e66473ace8c13d908) for staking rewards. After minting the appropriate amount of POL, the migration contract will be used to ensure that staking rewards continue to be paid out in MATIC, maximizing backward compatibility.
-   1% annual compounding emission is minted to a community treasury. Upon the deployment of contracts introduced in this proposal, a multi-signature wallet (0x2ff25495d77f380d5F65B95F103181aE8b1cf898) will be used to safeguard the funds until Community Board supervision is enacted.
    

A publicly callable function that is callable by any address can trigger the immediate minting of all vested emission. 

This contract is upgradeable via Governance. The initial implementation is described below.

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.21;

import {IPolygonEcosystemToken} from "./interfaces/IPolygonEcosystemToken.sol";
import {IPolygonMigration} from "./interfaces/IPolygonMigration.sol";
import {IDefaultEmissionManager} from "./interfaces/IDefaultEmissionManager.sol";
import {Ownable2StepUpgradeable} from "openzeppelin-contracts-upgradeable/contracts/access/Ownable2StepUpgradeable.sol";
import {Initializable} from "openzeppelin-contracts-upgradeable/contracts/proxy/utils/Initializable.sol";
import {SafeERC20} from "openzeppelin-contracts/contracts/token/ERC20/utils/SafeERC20.sol";
import {PowUtil} from "./lib/PowUtil.sol";

/// @title Default Emission Manager
/// @author Polygon Labs (@DhairyaSethi, @gretzke, @qedk)
/// @notice A default emission manager implementation for the Polygon ERC20 token contract on Ethereum L1
/// @dev The contract allows for a 1% mint *each* per year (compounded every year) to the stakeManager and treasury contracts
/// @custom:security-contact security@polygon.technology
contract DefaultEmissionManager is Initializable, Ownable2StepUpgradeable, IDefaultEmissionManager {
    using SafeERC20 for IPolygonEcosystemToken;

    // log2(2%pa continuously compounded emission per year) in 18 decimals, see _inflatedSupplyAfter
    uint256 public constant INTEREST_PER_YEAR_LOG2 = 0.028569152196770894e18;
    uint256 public constant START_SUPPLY = 10_000_000_000e18;
    address private immutable DEPLOYER;

    IPolygonEcosystemToken public token;
    IPolygonMigration public migration;
    address public stakeManager;
    address public treasury;

    uint256 public startTimestamp;

    constructor() {
        DEPLOYER = msg.sender;
        // so that the implementation contract cannot be initialized
        _disableInitializers();
    }

    function initialize(
        address token_,
        address migration_,
        address stakeManager_,
        address treasury_,
        address owner_
    ) external initializer {
        // prevent front-running since we can't initialize on proxy deployment
        if (DEPLOYER != msg.sender) revert();
        if (
            token_ == address(0) ||
            migration_ == address(0) ||
            stakeManager_ == address(0) ||
            treasury_ == address(0) ||
            owner_ == address(0)
        ) revert InvalidAddress();

        token = IPolygonEcosystemToken(token_);
        migration = IPolygonMigration(migration_);
        stakeManager = stakeManager_;
        treasury = treasury_;
        startTimestamp = block.timestamp;

        assert(START_SUPPLY == token.totalSupply());

        token.safeApprove(migration_, type(uint256).max);
        // initial ownership setup bypassing 2 step ownership transfer process
        _transferOwnership(owner_);
    }

    /// @notice Allows anyone to mint tokens to the stakeManager and treasury contracts based on current emission rates
    /// @dev Minting is done based on totalSupply diffs between the currentTotalSupply (maintained on POL, which includes any
    /// previous mints) and the newSupply (calculated based on the time elapsed since deployment)
    function mint() external {
        uint256 currentSupply = token.totalSupply(); // totalSupply after the last mint
        uint256 newSupply = _inflatedSupplyAfter(
            block.timestamp - startTimestamp // time elapsed since deployment
        );
        uint256 amountToMint = newSupply - currentSupply;
        if (amountToMint == 0) return; // no minting required

        uint256 treasuryAmt = amountToMint / 2;
        uint256 stakeManagerAmt = amountToMint - treasuryAmt;

        emit TokenMint(amountToMint, msg.sender);

        token.mint(address(this), amountToMint);
        token.safeTransfer(treasury, treasuryAmt);
        // backconvert POL to MATIC before sending to StakeManager
        migration.unmigrateTo(stakeManager, stakeManagerAmt);
    }

    /// @notice Returns total supply from compounded emission after timeElapsed from startTimestamp (deployment)
    /// @param timeElapsed The time elapsed since startTimestamp
    /// @dev interestRatePerYear = 1.02; 2% per year
    /// approximate the compounded interest rate using x^y = 2^(log2(x)*y)
    /// where x is the interest rate per year and y is the number of seconds elapsed since deployment divided by 365 days in seconds
    /// log2(interestRatePerYear) = 0.028569152196770894 with 18 decimals, as the interest rate does not change, hard code the value
    /// @return supply total supply from compounded emission after timeElapsed
    function _inflatedSupplyAfter(uint256 timeElapsed) private pure returns (uint256 supply) {
        uint256 supplyFactor = PowUtil.exp2((INTEREST_PER_YEAR_LOG2 * timeElapsed) / 365 days);
        supply = (supplyFactor * START_SUPPLY) / 1e18;
    }

    /**
     * @dev This empty reserved space is put in place to allow future versions to add new
     * variables without shifting down storage in the inheritance chain.
     * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
     */
    uint256[50] private __gap;
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
