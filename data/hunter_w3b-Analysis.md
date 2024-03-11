# Analysis - Wise Lending Contest

![Wise_Lending-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2Fz8cwuKocv88.0&w=256&q=75)

## Description overview of `Wise Lending` Contest

**Wise Lending**

Wise Lending is a decentralized liquidity market that allows users to supply crypto assets and earn a variable APY from borrowers. It is a flagship dApp within the Wise Ecosystem, which aims to create a sustainable financial ecosystem in crypto.

**Key Features of Wise Lending:**

- **Best-in-Class APY for Lenders:** Wise Lending offers competitive APYs for lenders by interfacing with yield sources that borrowers can arbitrage against with a click of a button.
- **Robust Oracle Safety System:** To mitigate risks, Wise Lending uses a combination of Chainlink price feeds and Uniswap TWAPs to ensure accurate price data.
- **Lending Automated Scaling Algorithm (LASA AI):** An on-chain AI that determines the APY in borrowing pools based on market conditions.
- **Positions Collateralized as NFTs:** Each user's position is represented as an NFT, allowing for transferrable positions and the development of modular features.
- **Capped Liquidation Fees:** Liquidation fees are hard-capped at a profitable range to protect borrowers from excessive losses.
- **Solely Deposit Mode:** A feature that allows lenders to deposit funds into separate vaults that cannot be borrowed against, ensuring capital preservation.

**How Wise Lending Works:**

- **Deposit Crypto:** Lenders deposit crypto assets into lending pools to earn a variable APY.
- **Borrow Crypto:** Borrowers can access yield-farming strategies to multiply their earnings by borrowing against the deposited crypto.
- **Earn APY:** Lenders earn APY on their deposited assets, while borrowers pay interest on their borrowed funds.

**Benefits of Wise Lending:**

- **Passive Income:** Lenders can earn passive income by depositing crypto into lending pools.
- **Enhanced Yield Farming:** Borrowers can access yield-farming strategies to maximize their earnings.
- **Transparency and Security:** Wise Lending is a decentralized platform with proof of reserves and a robust oracle safety system.
- **No Locks on Funds:** Lenders can withdraw their funds whenever needed, and deposits can be borrowed against at will.

## System Overview

### Scope

#### Main Contracts

- **WiseLending.sol**: This contract defines various functions related to depositing, borrowing, paying back,collateralizing their deposits to secure the borrowed funds and liquidating loans on a decentralized lending platform. It focuses on managing positions represented by non-fungible tokens (NFTs) and allows users to interact with liquidity pools.

  - **Key Features**

    - **Lending and Borrowing:** Users can deposit and borrow various cryptocurrencies, including ETH and ERC-20 tokens.
    - **Collateralization:** Deposits can be collateralized to secure borrowed funds, reducing the risk for the lender.
    - **Variable Borrow Rates:** Borrow rates are variable and determined by the utilization of the pool, adjusting automatically through the LASA (Lending Automated Scaling Algorithm).
    - **Bounding Curve:** The protocol uses a family of bounding curves to calculate the share price of lending and borrow shares, adjusting over time based on pool utilization.
    - **Position NFTs:** Users can store their collateral and borrows in Position NFTs, enabling them to trade their positions or use them in other contracts.

  - **Functionalities**

    - **Deposit:** Users can deposit funds into the protocol, either publicly or privately (solely mode).
    - **Withdraw:** Users can withdraw deposited funds, both publicly and privately.
    - **Borrow:** Users can borrow funds against their collateralized deposits.
    - **Repay:** Users can repay borrowed funds, including the option to use lending shares of the same asset type.
    - **Payback:** Users can pay back ETH and ERC20 loans with an exact amount and exact number of shares.
    - **Liquidation:** Allows users to liquidate a position that exceeds a certain debt ratio. The liquidator can choose the tokens to use for payback and receive.
    - **Health Check:** The contract includes health checks to ensure the safety of user positions.
    - **Pool Sync:** The protocol runs the LASA algorithm to update pool data and adjust borrow rates.

  - **Security Measures**

  - **Collateralization Checks:** Collateralization ensures that users have sufficient collateral to cover their borrowed funds.
  - **Position NFT Management:** Position NFTs provide transparency and control over user positions.
  - **Health State Checks:** The contract checks the health of user positions after state changes to ensure they remain secure.
  - **Curve Security Checks:** The protocol monitors the bonding curves to prevent excessive share price fluctuations.

  - **Core Logic**

  - `_handlePayback`: Internal function for handling payback logic and emitting an event.
  - `_handleWithdrawAmount`: Internal function for preparing and executing token withdrawals.
  - `_handleSolelyWithdraw`: Internal function for handling withdrawals of the user's own funds.
  - `_handleWithdrawShares`: Internal function for handling withdrawals of specific share amounts.
  - `_handleBorrowExactAmount`: Internal function for handling exact amount borrowing.
  - `_reservePosition`: Internal function for reserving a position NFT ID.

  - **Additional Features**

  - **Aave Pools:** Users can earn Aave supply APY for deposited funds that are not borrowed.
  - **PowerFarm Pools:** Special curve pools can be used as collateral, expanding the range of supported assets.
  - **Solely Deposit and Withdraw:** Users can deposit and withdraw funds privately, keeping their balances hidden.

- **MainHelper.sol**: This contract abstractly assists in managing lending and borrowing operations within the Wise protocol, facilitating calculations for converting token amounts into lending or borrowing shares, ensuring security checks, and updating pool utilization and pseudo amounts while interacting with external protocols such as Aave and Curve.

  - **Key Features**

    - **Lending and borrowing:** Users can deposit assets into lending pools and borrow assets from borrowing pools.
    - **Isolation pools:** Users can create isolated pools to manage their assets separately from other users.
    - **Liquidation:** Users can liquidate positions that are underwater.
    - **Fee management:** The protocol collects fees on all transactions and distributes them to fee managers.

  - **Functionalities**

    - **`calculateLendingShares`:** Calculates the number of lending shares that a user will receive for depositing a certain amount of an asset.
    - **`calculateBorrowShares`:** Calculates the number of borrow shares that a user will receive for borrowing a certain amount of an asset.
    - **`cashoutAmount`:** Calculates the amount of an asset that a user will receive for cashing out a certain number of lending shares.
    - **`paybackAmount`:** Calculates the amount of an asset that a user needs to repay to pay back a certain number of borrow shares.
    - **`_preparationsWithdraw`:** Prepares the system for a user to withdraw assets from a lending pool.
    - **`_getValueUtilization`:** Calculates the utilization of a lending pool.
    - **`_updateUtilization`:** Updates the utilization of a lending pool.
    - **`_checkCleanUp`:** Checks if there are any falsely sent tokens in a lending pool and adds them to the pool if necessary.
    - **`_onlyIsolationPool`:** Checks if a user is trying to access an isolation pool that they are not authorized to access.
    - **`_validateIsolationPoolLiquidation`:** Validates the inputs for a liquidation transaction in an isolation pool.
    - **`_getBalance`:** Gets the balance of an asset in the protocol.
    - **`_cleanUp`:** Adds falsely sent tokens to a lending pool if necessary.
    - **`_preparePool`:** Prepares a lending pool for use.
    - **`_preparationTokens`:** Prepares the tokens that a user is trying to deposit or borrow.
    - **`_curveSecurityChecks`:** Checks the security of curve pools that are used in the protocol.
    - **`_updatePseudoTotalAmounts`:** Updates the pseudo total amounts of a lending pool.
    - **`_increasePositionLendingDeposit`:** Increases the number of lending shares that a user has in a lending pool.
    - **`_decreaseLendingShares`:** Decreases the number of lending shares that a user has in a lending pool.
    - **`_addPositionTokenData`:** Adds a token to a user's position data.
    - **`_getHash`:** Gets the hash of a user's position data.
    - **`_removePositionData`:** Removes a token from a user's position data.
    - **`_deleteLastPositionLendingData`:** Deletes the last entry in a user's lending position data.
    - **`_corePayback`:** Pays back a borrow position.
    - **`isUncollateralized`:** Checks if a user's position is uncollateralized.
    - **`_checkLendingDataEmpty`:** Checks if a user's lending position data is empty.
    - **`_calculateNewBorrowRate`:** Calculates the new borrow rate for a lending pool.
    - **`_newBorrowRate`:** Updates the borrow rate for a lending pool.
    - **`_aboveThreshold`:** Checks if the time interval for the next LASA call has passed.
    - **`_scalingAlgorithm`:** Adjusts the resonance factor of a lending pool to maximize total deposit shares.
    - **`_newMaxPoolShares`:** Sets the new max value in shares for a lending pool.
    - **`_saveUp`:** Sets the previous value and timestamp scaling for LASA of a lending pool.
    - **`_resonanceOutcome`:** Determines if the resonance factor of a lending pool needs to be reset.
    - **`_resetResonanceFactor`:** Resets the resonance factor of a lending pool to the last best value.
    - **`_revertDirectionState`:** Reverts the flag for stepping direction from LASA.
    - **`_updateResonanceFactor`:** Updates the resonance factor of a lending pool.
    - **`_reversedResonanceFactor`:** Does a revert stepping and swaps stepping state in opposite flag.
    - **`_changingResonanceFactor`:** Increasing or decresing resonance factor depending on flag value.
    - **`_increaseResonanceFactor`:** Stepping function increasing the resonance factor depending on the time past in the last time interval.
    - **`_decreaseResonanceFactor`:** Stepping function decresing the resonance factor depending on the time past in the last time interval.
    - **`_removeEmptyLendingData`:** Removes token address from lending data array if all shares are removed.

  - **Core Logic**

    The core logic of the protocol is to allow users to deposit assets into lending pools and borrow assets from borrowing pools. The protocol uses a unique algorithm called LASA (Liquidity Adjusted Supply Algorithm) to adjust the resonance factor of lending pools in order to maximize total deposit shares.

  - **Security Measures**

    - **Isolation pools:** Users can create isolated pools to manage their assets separately from other users. This helps to protect users from the risk of other users' positions being liquidated.
    - **Liquidation:** Users can liquidate positions that are underwater. This helps to protect the protocol from the risk of bad debt.
    - **Fee management:** The protocol collects fees on all transactions and distributes them to fee managers. This helps to ensure that the protocol is sustainable.

  - **Additional Features**

    - **Flash loans:** Users can borrow assets without having to provide collateral. This can be useful for arbitrage opportunities or other short-term trading strategies.
    - **Margin trading:** Users can borrow assets to increase their exposure to a particular asset. This can be useful for traders who want to amplify their profits.
    - **Yield farming:** Users can earn rewards by depositing assets into lending pools. This can be a good way to earn passive income.

- **WiseCore.sol**: This abstract contract is `WiseCore` includes functions for depositing, withdrawing, and borrowing tokens, as well as liquidating positions.

  - **Key Features**

    - Allows users to deposit and withdraw tokens from the protocol.
    - Allows users to borrow tokens from the protocol.
    - Allows users to liquidate positions that are underwater.
    - Includes security measures to protect users from fraud and loss.

  - **Functionalities:**

  - **Preparing Associated Tokens:** The `_prepareAssociatedTokens` function prepares tokens associated with lending and borrowing, used in deposit and withdrawal operations.

    - **Withdraw Token Core:** The `_coreWithdrawToken` function combines withdrawal logic with security checks.

    - **Handle Deposit:** The `_handleDeposit` function handles deposits, including validation and updating pool storage.

    - **Check Position Locked:** Ensures that a position is not locked for power farms unless certain conditions are met.

    - **Check Deposit:** Performs security checks before allowing a deposit, including checking the oracle status and position ownership.

    - **Check Allow Deposit:** Ensures that only the owner or specific authorized contracts can perform a deposit.

    - **Check Max Deposit Value:** Verifies whether the deposit amount for a pool token exceeds the maximum allowed value.

    - **Withdraw Pure Collateral Liquidation:** Calculates the amount to withdraw from pure collateral during liquidation.

    - **Withdraw or Allocate Shares Liquidation:** Determines whether to withdraw tokens or allocate shares to a liquidator during liquidation, based on available funds.

    - **Calculate Receive Amount:** Calculates the amount to receive during liquidation, considering various factors such as collateral percentage and user shares.

    - **Core Liquidation:** Performs the core functionality for liquidation, including payback, calculation of receive amount, and security checks.

  - **Core Logic:**

    The core logic of the WiseCore contract revolves around managing deposits, withdrawals, and liquidations of tokens within the WISE protocol. It includes functions for preparing tokens, handling deposits and withdrawals, performing security checks, and executing liquidations.

  - **Security Measures**

    - Checks to ensure that users have sufficient collateral before they can borrow tokens.
    - Checks to ensure that users are not liquidating positions that are not underwater.
    - Checks to ensure that users are not withdrawing more tokens than they have deposited.

  - **Additional Features**

    - Allows users to specify the maximum fee they are willing to pay for liquidation.
    - Allows users to specify the base reward they are willing to pay for liquidation.
    - The contract includes functions for checking position locks, deposit validation, and enforcing maximum deposit values.
    - It provides comprehensive functionality for liquidating positions, including calculations and security checks.
    - The contract architecture facilitates modularity and extensibility by combining helper contracts and abstracting core functionalities for reuse in other contracts within the WISE protocol.

- **WiseLendingDeclaration.sol**: The `WiseLendingDeclaration` contract implements a lending protocol with features including depositing, withdrawing, borrowing, and returning funds, using various data structures, events, modifiers, and state variables, while integrating interfaces for Aave, position NFTs, security, Oracle, and fee management, alongside constants for precision factors, time, thresholds, and APR restrictions.

- **WiseLowLevelHelper.sol**: The `WiseLowLevelHelper` contract provides low-level helper functions for managing lending pools, including validation of parameters, retrieval of pool data, setting internal values.

- **PositionNFTs.sol**: The contract `PositionNFTs` is an ERC721-compliant contract responsible for managing NFTs representing positions within the WiseLending protocol. It allows users to position and mint NFTs, which grant access to protocol functionalities, and provides additional features such as role-based access control, URI management, and NFT metadata generation.

  - **Key Features**

    - Allows users to reserve positions in a decentralized lending protocol.

    - Allows users to mint NFTs to represent those positions.

    - Allows users to manage the ownership of those NFTs.

    - The contract facilitates fee management by transferring a special NFT (`FEE_MANAGER_NFT`) to a designated fee manager contract.

    - It allows setting and updating the base URI and extension for NFT metadata URIs.

  - **Functionalities**

    - **Position Reservation**: Users can reserve positions within the WiseLending protocol by calling the `reservePosition` or `reservePositionForUser` functions.

    - **Position Minting**: Users can mint NFTs representing positions using the `mintPosition` or `mintPositionForUser` functions.

    - **Role Assignment**: The contract provides a function (`assignReserveRole`) for the master contract to assign or revoke the reserve role to specific addresses.

    - **Fee Manager Assignment**: The fee manager contract can be assigned by calling the `forwardFeeManagerNFT` function.

  - **Core Logic**

    - Users can reserve positions in the decentralized lending protocol by calling the `reservePosition` function.
    - Users can mint NFTs to represent those positions by calling the `mintPosition` function.
    - Users can transfer ownership of NFTs by calling the `transferFrom` function.
    - Users can approve others to transfer ownership of NFTs by calling the `approve` function.
    - Users can get the metadata associated with an NFT by calling the `tokenURI` function.

  - **Security Measures**

    - Checks to ensure that the user has sufficient permissions to perform an action before allowing them to perform that action.
    - Checks to ensure that the NFT exists before allowing the user to perform an action on it.
    - Checks to ensure that the user is the owner of the NFT before allowing them to perform an action on it.

  - **Additional Features**

    - Allows users to set a base URI for the NFT metadata.
    - Allows users to set a base extension for the NFT metadata.
    - Allows users to get the positions of a specific owner.

- **PoolManager.sol**: The `PoolManager` contract facilitates the creation and management of lending pools with adjustable parameters, such as borrowing capabilities, collateral factors, and maximum deposit amounts, ensuring the integrity and efficiency of the WiseCore protocol.

- **OwnableMaster.sol**: This contract defines an ownable master pattern with a proposal mechanism. It allows the current master to propose a new master, who can then claim ownership to become the new master. The master can also renounce ownership, effectively removing the master role.

#### WiseSecurity

- **WiseSecurity.sol**: The `WiseSecurity` contract plays a crucial role in ensuring the security and integrity of the WiseLending protocol by enforcing various checks, calculations, and safeguards related to lending and borrowing positions, as well as providing mechanisms for handling emergencies and mitigating risks associated with certain tokens.

  - **Key Features:**

    - **Security Checks**: The contract performs various security checks before allowing users to perform certain actions like withdrawals, borrowing, collateralization, and uncollateralized deposits.
    - **Debt Ratio Calculation**: It provides a function to calculate the live debt ratio of a position, which helps determine the health of the position.
    - **Liquidation Settings**: It allows the master address to set the liquidation settings, such as the base reward, maximum fees, and boundaries for liquidation.
    - **Curve Pool Integration**: It supports the integration of Curve pools and allows setting up various Curve-related data and approvals.
    - **APY Calculations**: It provides functions to calculate the overall lending APY, borrowing APY, and net APY for a given position.
    - **Position Limit Calculations**: It provides a function to calculate the safe limit for borrowing against a position.
    - **Blacklisting Tokens**: It allows the master address to blacklist tokens, preventing them from being used as collateral or borrowed.
    - **Security Shutdown**: It includes a security shutdown mechanism that can be triggered by authorized security workers to pause all active pools.

  - **Functionalities:**

    - `getLiveDebtRatio`: Calculates the live debt ratio of a position.
    - `setLiquidationSettings`: Allows the master address to set liquidation settings.
    - `checksLiquidation`: Performs checks for the liquidation logic.
    - `prepareCurvePools`: Prepares Curve pools for integration.
    - `curveSecurityCheck`: Performs a security check for Curve pools.
    - `checksWithdraw` and `checksSolelyWithdraw`: Check the conditions for allowing withdrawals.
    - `checksBorrow`: Checks the conditions for allowing borrowing.
    - `checksCollateralizeDeposit`: Checks the conditions for collateralized deposits.
    - `checkUncollateralizedDeposit`: Checks the conditions for uncollateralized deposits.
    - `checkHealthState`: Checks if a position is in a healthy state.
    - `checkBadDebtLiquidation`: Checks for bad debt liquidation scenarios.
    - `overallLendingAPY`, `overallBorrowAPY`, and `overallNetAPY`: Calculate the lending, borrowing, and net APYs for a position.
    - `safeLimitPosition`: Calculates the safe borrowing limit for a position.
    - `positionBlackListToken`: Checks if a position is locked due to a blacklisted token.
    - `maximumWithdrawToken` and `maximumWithdrawTokenSolely`: Calculate the maximum withdrawal amount for a position and a specific token.

    - `maximumBorrowToken`: Calculates the maximum borrowing amount for a position and a specific token.
    - `getExpectedPaybackAmount`: Calculates the expected payback amount for a position and a specific token.
    - `getExpectedLendingAmount`: Calculates the expected lending amount for a position and a specific token.
    - `setBlacklistToken`: Allows the master address to blacklist or unblacklist tokens.
    - `setSecurityWorker`: Allows the master address to add or remove security workers.
    - `securityShutdown`: Allows security workers to initiate a security shutdown of all active pools.
    - `revokeShutdown`: Allows the master address to revoke the security shutdown.
    - `checkPoolCondition` and `checkPoolWithMinDeposit`: Check the conditions for a pool and the minimum deposit value.
    - `changeMinDepositValue`: Allows the master address to change the minimum deposit value.

  - **Core Logic:**

    The core logic of the contract revolves around performing various security checks and calculations related to lending and borrowing positions. It enforces conditions such as verifying the caller's role (master, security worker, or WiseLending contract), checking the health of positions, ensuring proper collateralization, and preventing actions on blacklisted tokens or locked positions.

  - **Security Measures:**

    - **Access Control**: The contract employs various access control mechanisms, such as the `onlyMaster` and `onlyWiseLending` modifiers, to restrict sensitive operations to authorized addresses.
    - **Reentrancy Guard**: The `curveSecurityCheck` function includes a reentrancy guard by forcing a swap to update internal Curve values before proceeding.
    - **Security Shutdown**: The contract features a security shutdown mechanism that can be triggered by authorized security workers, allowing for a temporary pause of all active pools in case of emergencies.
    - **Input Validation**: Various functions include input validation checks to prevent invalid or malicious inputs.

  - **Additional Features:**

    - **Blacklisting Tokens**: The contract allows the master address to blacklist tokens, preventing them from being used as collateral or borrowed. This feature can be used to mitigate risks associated with certain tokens.
    - **Minimum Deposit Value**: The contract includes a mechanism to enforce a minimum deposit value in terms of ETH. This can help prevent users from making insignificant deposits, potentially saving on gas costs.
    - **Curve Pool Integration**: The contract supports the integration of Curve pools, allowing for swaps and interactions with Curve-related contracts.

- **WiseSecurityHelper.sol**: This `WiseSecurityHelper` provides various functions for calculating and managing collateral, borrow amounts, liquidation, and pool states in a lending protocol, utilizing weighted and unweighted collateral factors for positions with specified NFT IDs.

  - **Key Features**

    - **Collateral Calculations:**
    - Computes weighted and unweighted total collateral of a position.
    - Calculates collateral values updated to current time.
    - **Borrow Calculations:**
    - Determines the total borrow amount of a position, considering heartbeat and blacklisted checks.
    - Calculates updated borrow amounts based on interest accumulation.
    - **Liquidation Checks:**
    - Compares borrow and collateral amounts to determine if liquidation is possible.
    - Limits the number of shares a liquidator can receive to prevent excessive liquidation.
    - **Health Checks:**Verifies that debt ratio is not greater than 100% after withdraw.
    - **Registration Checks:**Ensures that positions are empty before registering as power farms.
    - **Pool Management:** Locks or unlocks all pools for borrow and deposit actions.

  - **Functionalities**

    - **`overallETHCollateralsBoth`:** Calculates weighted and unweighted total collateral of a position.
    - **`overallETHCollateralsWeighted`:** Calculates weighted total collateral of a position.
    - **`overallETHCollateralsBare`:** Calculates unweighted total collateral of a position.
    - **`_overallETHCollateralsWeighted`:** Calculates weighted total collateral of a position, updated to current time.
    - **`getFullCollateralETH`:** Computes the full collateral amount of a token from a position.
    - **`_isUncollateralized`:** Checks if a fund is uncollateralized.
    - **`_getCollateralOfTokenETHUpdated`:** Calculates the full collateral amount of a token from a position, updated to current time.
    - **`getETHCollateralUpdated`:** Calculates the public collateral amount of a token from a position, updated to current time.
    - **`getETHCollateral`:** Calculates the public collateral amount of a token from a position.
    - **`_getTokensInEth`:** Converts token amounts to ETH values.
    - **`overallETHBorrowBare`:** Calculates the total borrow amount of a position, excluding heartbeat and blacklisted checks.
    - **`overallETHBorrowHeartbeat`:** Calculates the total borrow amount of a position, considering heartbeat checks.
    - **`overallETHBorrow`:** Calculates the total borrow amount of a position, considering heartbeat and blacklisted checks.
    - **`_checkBlacklisted`:** Checks if a pool is blacklisted.
    - **`_overallETHBorrow`:** Calculates the total borrow amount of a position, updated to current time.
    - **`_getUpdatedPseudoBorrow`:** Calculates the updated pseudo borrow amount of a pool.
    - **`_getUpdatedPseudoPool`:** Calculates the updated pseudo lending amount of a pool.
    - **`_getInterest`:** Calculates the accumulated interest amount for a pool.
    - **`_getETHBorrowUpdated`:** Calculates the full borrow amount of a token from a position, updated to current time.
    - **`getETHBorrow`:** Calculates the borrow amount of a token from a position.
    - **`checkTokenAllowed`:** Checks if a token is allowed for borrowing.
    - **`checkHeartbeat`:** Checks if the ChainLink feed for a pool has been updated within an expected timeframe.
    - **`_checkPositionLocked`:** Checks if a position is locked for interactions.
    - **`_checkMaxFee`:** Calculates the maximum fee for liquidation.
    - **`calculateWishPercentage`:** Computes the percentage of receiving tokens a liquidator receives for liquidation.
    - **`_checkHealthState`:** Checks if debt ratio is not greater than 100% after withdraw.
    - **`_getState`:** Determines if a position has bad debt.
    - **`checksRegister`:** Checks if a position is empty before registering as a power farm.
    - **`canLiquidate`:** Compares borrow and collateral amounts to determine if liquidation is possible.
    - **`checkMaxShares`:** Checks if the number of shares a liquidator is requesting is within acceptable limits.
    - **`checkBadDebtThreshold`:** Determines if a position has bad debt.
    - **`getPositionLendingAmount`:** Computes the lending token amount for a pool.
    - **`getPositionBorrowAmount`:** Computes the borrow token amount for a pool.
    - **`checkOwnerPosition`:** Checks if an address is the owner of a position.
    - **`getBorrowRate`:** Returns the borrow rate for a pool.
    - **`getLendingRate`:** Returns the lending rate for a pool.
    - **`_getPossibleWithdrawAmount`:** Calculates the possible withdraw amount of a token from a position, updated to current time.
    - **`_setPoolState`:** Locks or unlocks all pools for borrow and deposit actions.
    - **`_checkPoolCondition`:** Checks if a pool is blacklisted.
    - **`_checkSuccess`:** Checks if a low-level byte call was successful.

  - **Core Logic**

    The core logic of this contract revolves around calculating collateral and borrow amounts, performing liquidation checks, and managing pool states. It ensures that positions are healthy, liquidation is fair, and pools are operating smoothly.

  - **Security Measures**

    - **Heartbeat Checks:** Ensures that ChainLink feeds are updated within expected timeframes.
    - **Blacklisted Checks:** Prevents interactions with pools that are flagged as blacklisted.
    - **Position Lock Checks:** Blocks interactions with positions that are locked.
    - **Max Fee Calculations:** Limits the fee a liquidator can receive to prevent excessive liquidation.
    - **Bad Debt Thresholds:** Identifies positions with high debt ratios to prevent liquidation when collateral is insufficient.

  - **Additional Features**

    - **Power Farm Registration Checks:** Ensures that positions are empty before registering as power farms.
    - **Withdraw Health Checks:** Verifies that debt ratio is not greater than 100% after withdraw.
    - **Pool Locking and Unlocking:** Allows for centralized control over pool operations.

- **WiseSecurityDeclarations.sol**: This contract is a collection of declarations and constants used by other contracts in the Wise Security system. It includes variables, mappings, and functions that are used to configure and manage the system.

  - **Key Features:**

    - **Fee Management**: It includes a fee manager contract (`FEE_MANAGER`) responsible for managing fees associated with protocol actions.

    - **Liquidation Management**: The contract handles liquidations within the WiseLending protocol, setting liquidation incentive thresholds, fees, and rewards.

    - **Chainlink Integration**: The contract integrates with Chainlink oracles for fetching real-time data, facilitating secure and reliable protocol operations.

  - **Functionalities:**

    - **Liquidation Settings**: It allows the adjustment of liquidation-related settings such as base rewards, maximum fees, and liquidation incentives.

    - **Blacklist Management**: The contract maintains a blacklist of pool tokens that are not allowed to be used within the protocol.

    - **Curve Swap Data**: It stores information related to Curve swaps, including basic swap data and swap info for reentrancy guard.

  - **Additional Features:**

    - **Chainlink Integration**: The contract utilizes Chainlink oracles for fetching real-time data, enhancing the accuracy and reliability of protocol operations.

    - **Liquidation Management**: Liquidation-related settings such as base rewards, maximum fees, and liquidation incentives can be adjusted to optimize protocol performance and security.

#### PowerFarms

- **PendlePowerFarm**

  - **PendlePowerFarm.sol**: This contract is the main implementation of the Pendle Power Farm functionalities and logic for managing positions within the Pendle Power Farm. It extends the `PendlePowerFarmLeverageLogic` contract and provides additional functionalities related to opening, closing, and managing leveraged positions within the Pendle protocol.

    - **Key Features**

      - **Leverage Calculation**: The contract includes functions for calculating the leverage amount based on the initial amount and leverage factor.
      - **Position Management**: The contract provides functions for opening and closing leveraged positions, including managing the borrowing and lending of assets.
      - **Pendle Integration**: The contract interacts with the Pendle protocol to access yield-generating opportunities and reward tokens.
      - **Liquidation**: The contract includes a liquidation function for positions that exceed the maximum debt ratio.
      - **Manual Payback and Withdraw**: The contract allows users to manually repay borrowed shares and withdraw collateral shares.

    - **Functionalities**

      - **`getApproxNetAPY`()**: Approximates the net APY for a given initial amount, leverage factor, and Pendle child APY.
      - **`getNewBorrowRate`()**: Approximates the new borrow rate for the pool when a given amount is borrowed.
      - **`getLiveDebtRatio`()**: Returns the current debt ratio for a given position.
      - **`liquidatePartiallyFromToken`()**: Liquidates a position by repaying a specified amount of borrowed shares.
      - **`_manuallyPaybackShares`()**: Allows users to manually repay borrowed shares.
      - **`_manuallyWithdrawShares`()**: Allows users to manually withdraw collateral shares.
      - **`_openPosition`()**: Combines the core logic for opening a leveraged position.
      - **`_closingPosition`()**: Combines the core logic for closing a leveraged position.
      - **`_registrationFarm`()**: Registers a position for specific farm use.

    - **Core Logic**

      The core logic of the contract revolves around managing leveraged positions and interacting with the Pendle protocol. It uses mathematical formulas to calculate leverage amounts, borrow rates, and net APYs. The contract also handles the borrowing and lending of assets through the Pendle protocol.

  - **PendlePowerFarmLeverageLogic.sol**: The contract provides a comprehensive solution for managing leveraged positions and flash loans in protocol, with several security measures in place to protect user funds. Allows users to open and close leveraged positions on the Pendle protocol.

    - **Key Features**:

      - **Flash Loan Functionality**: Uses Balancer flash loans to execute leveraged trades.
      - **Leverage Management**: The contract manages leverage for users by opening and closing positions using flash loans and other DeFi protocols.
      - **Liquidity Provision**: It interacts with liquidity pools, likely providing liquidity to earn yield or facilitate trading.
      - Supports both Aave and Wise lending protocols.

    - **Functionalities**:

      - **Execute Balancer Flash Loan**: Prepares and executes flash loans from the Balancer protocol, passing necessary data to the flash loan receiver.
      - **Receive Flash Loan**: Handles flash loan callbacks, determining whether to open or close a position based on certain conditions.
      - **Open Position**: Deposits collateral, borrows funds, and adds liquidity to DeFi protocols to open a leveraged position.
      - **Close Position**: Withdraws liquidity, repays borrowed funds, and redeems collateral to close a leveraged position.
      - **Core Liquidation**: Manages liquidation events, ensuring the safety of the protocol and its users by liquidating positions when necessary.

    - **Core Logic**:

      The core logic of this contract is to use Balancer flash loans to execute leveraged trades on the Pendle protocol. When a user opens a leveraged position, the contract borrows a certain amount of ETH from Balancer and uses it to buy the underlying asset on Pendle. The user then deposits the purchased asset as collateral on Pendle and borrows the desired amount of the leveraged asset. When the user closes a leveraged position, the contract sells the underlying asset on Pendle and uses the proceeds to repay the flash loan.

    - **Security Measures**:

      - **Debt Ratio Checks**: The contract checks debt ratios to ensure that leveraged positions are within acceptable risk levels.
      - **Isolation Pools**: For liquidation events, the contract uses isolation pools to separate funds and minimize the impact of liquidations on other users.
      - **Chain ID Validation**: Certain functions validate the chain ID to ensure that transactions are executed on the correct blockchain network.

    - **Additional Features**:

      - **Flash Loan Support**: The contract supports flash loans from the Balancer protocol, enabling users to execute complex trading strategies without upfront capital.
      - **Leverage Management**: Users can open leveraged positions by depositing collateral and borrowing funds, potentially increasing their exposure to market movements.
      - **Automated Position Management**: The contract automates various aspects of position management, including collateralization, borrowing, and liquidation, reducing the need for manual intervention.
      - **Compatibility with Multiple Protocols**: The contract interacts with multiple DeFi protocols, such as Balancer, Aave, Uniswap V3, and Curve Finance, providing users with access to diverse liquidity sources and trading opportunities.

  - **PendlePowerFarmDeclarations.sol**: The `PendlePowerFarmDeclarations` contract is a set of declarations and initialization logic for managing farming activities, interfacing with various protocols including Aave, Pendle, Balancer, Curve, Uniswap, and providing functionalities such as approval management, entry and exit event logging, and contract activation/deactivation controls.

  - **PendlePowerFarmMathLogic.sol**: This contract defines the mathematical logic for the Pendle Power Farm.

    - **Key Features**

      - **Leveraged Yield Farming**: The contract allows users to borrow assets to increase their exposure to yield-generating opportunities.
      - **Approximated Net APY Calculation**: It provides an approximation of the net APY that users can expect from their leveraged positions.
      - **Debt Ratio Check**: It checks if a position's debt ratio is below 100%, ensuring that users are not overleveraged.
      - **Minimum Deposit Amount Check**: It verifies that the leveraged amount is not below a specified minimum threshold.

    - **Functionalities**

      - **`_updatePools`()**: Updates the state of the lending pools used by the Pendle Power Farm.
      - **`_checkReentrancy`()**: Prevents reentrancy attacks by checking if other critical functions are currently executing.
      - **`_getPositionBorrowShares`()**: Gets the borrow shares for a given position and token.
      - **`_getPositionLendingShares`()**: Gets the lending shares for a given position and token.
      - **`_getPostionCollateralTokenAmount`()**: Converts lending shares into the corresponding token amount.
      - **`getPositionBorrowETH`()**: Calculates the total borrow amount in ETH for a given position.
      - **`getTotalWeightedCollateralETH`()**: Calculates the total collateral amount in ETH for a given position, weighted by the collateral factor.
      - **`_getTokensInETH`()**: Converts a token amount to its equivalent ETH value using an oracle.
      - **`_getEthInTokens`()**: Converts an ETH amount to its equivalent token amount using an oracle.
      - **`getLeverageAmount`()**: Calculates the leveraged amount based on the initial amount and leverage factor.
      - **`_getApproxNetAPY`()**: Approximates the net APY for a leveraged position.
      - **`_getNewBorrowRate`()**: Calculates the new borrow APY based on the borrowed amount and current pool utilization.
      - **`_checkDebtRatio`()**: Checks if a position's debt ratio is below 100%.
      - **`_notBelowMinDepositAmount`()**: Checks if the leveraged amount is not below a specified minimum threshold.

    - **Core Logic**

      The core logic of the contract revolves around calculating the net APY for leveraged positions and ensuring that positions are not overleveraged. It uses mathematical formulas to approximate the net APY and checks the debt ratio to prevent excessive risk.

  - **PendlePowerManager.sol**: This contract allows users to leverage their assets to maximize their yield. It inherits from the PendlePowerFarm contract, which defines the core mathematical logic and data structures, and the MinterReserver contract, which provides functionality for minting and reserving NFTs.

    - **Key Features:**

      - **Leveraged Yield Farming**: The contract enables users to borrow assets to increase their exposure to yield-generating opportunities.
      - **NFT-Based Key Management**: Each leveraged position is represented by a unique NFT, which serves as a key for managing the position.
      - **Flexible Leverage**: Users can choose their desired leverage factor to customize their risk and reward profile.
      - **Shutdown Mechanism**: The contract can be deactivated to prevent new positions from being opened, allowing users to manually close their existing positions.
      - **ETH and ERC-20 Support**: The contract supports both ETH and ERC-20 tokens as underlying assets for leveraged positions.

    - **Functionalities:**

      - **receive():** Standard receive function to accept ETH payments, forwarding them to the master address.

      - **enterFarm():** Allows users to enter the farming position with ERC20 tokens, specifying leverage and allowed spread.

      - **enterFarmETH():** Allows users to enter the farming position with ETH directly, specifying leverage and allowed spread.

      - **exitFarm():** Enables users to exit the farming position, releasing the associated NFT and returning shares.

      - **manuallyPaybackShares():** Allows users to manually pay back shares associated with a farming position.

      - **manuallyWithdrawShares():** Allows users to manually withdraw shares from a farming position, ensuring the debt ratio remains within acceptable limits.

    - **Core Logic:**

      The core logic of the contract revolves around managing farming positions, including opening, closing, and tracking positions within the Pendle protocol. It also handles the reservation and release of NFTs representing farming positions, along with facilitating entry and exit operations.

    - **Additional Features:**

      - **Dynamic Position Allocation:** The contract dynamically allocates and releases farming positions represented by NFTs, optimizing resource utilization within the Pendle protocol.

      - **Manual Position Management:** Users can manually manage their farming positions by paying back shares or withdrawing shares, providing greater control over their investment strategies within the Pendle protocol.

- **PendlePowerFarmController**

  - **PendlePowerFarmToken.sol**: This contract implements a Pendle Power Farm Token, a type of `yield-bearing` token that represents ownership of underlying LP assets in a Pendle market. It allows users to deposit and withdraw LP assets (from the underlying Pendle market) and mint and burn shares of the Power Farm Token. The contract also distributes rewards to token holders.

    - **Key Features**:

      - **Integration with Pendle Protocol:** The contract interacts with the Pendle Protocol, as evidenced by the imports of `IPendle.sol` and `IPendleController.sol`. This suggests that the contract is part of a larger ecosystem involving decentralized finance (DeFi) protocols.

      - **Yield-bearing:** Token holders earn rewards in the form of LP assets.

      - **Flexible:** Users can deposit and withdraw LP assets at any time.

      - **Transparent:** The contract's logic and state are publicly verifiable.

      - **Fee Mechanism:** The contract implements a fee mechanism for minting tokens. Users pay a fee when minting new tokens, with the fee amount configurable by the contract owner.

      - **Market Dynamics:** The contract tracks various market dynamics, such as the total balance of LPs (liquidity providers) backing the token, the LP assets available for distribution, and the share price of the token.

    - **Functionalities**:

      - **Minting and Burning:** Users can mint new tokens by depositing underlying LP assets into the contract, and they can burn tokens to withdraw underlying LP assets. These functionalities are implemented through the `depositExactAmount`, `withdrawExactShares`, and `withdrawExactAmount` functions.

      - **Fee Adjustment:** The contract owner can adjust the minting fee using the `changeMintFee` function.

      - **Deposit LP Assets:** Users can deposit LP assets into the contract to mint Power Farm Tokens.

      - **Withdraw LP Assets:** Users can withdraw LP assets from the contract by burning Power Farm Tokens.

      - **Mint Power Farm Tokens:** Users can mint Power Farm Tokens by depositing LP assets.

      - **Burn Power Farm Tokens:** Users can burn Power Farm Tokens to withdraw LP assets.

      - **Distribute Rewards:** The contract distributes rewards to token holders in the form of LP assets.

    - **Core Logic**:

      - **Minting Fee Calculation:** The contract calculates a minting fee based on a percentage of the number of tokens being minted.

      - **Market Synchronization:** The contract synchronizes with the Pendle market by updating indices, overwriting market parameters, and calculating rewards.

      - **Share Price Management:** The contract ensures that the share price does not decrease over time and enforces a maximum growth rate for the share price.

    - **Security Measures**:

      - **Modifier Restrictions:** The contract uses modifiers to restrict access to certain functions. For example, the `onlyController` modifier restricts access to functions that can only be called by the Pendle controller.

      - **Error Handling:** The contract uses custom error messages and reverts transactions in case of invalid inputs or unauthorized access.

    - **Additional Features**:

      - **Market Dynamics Tracking:** The contract tracks various market dynamics, such as total LP assets and share price, to ensure that the token's value is accurately reflected.

      - **Fee Mechanism:** The contract implements a fee mechanism for minting tokens, which provides an incentive for LPs to provide liquidity while generating revenue for the contract owner.

      - **Integration with Pendle Protocol:** The contract integrates with the Pendle Protocol, allowing users to participate in DeFi activities such as yield farming and liquidity provision.

      - **Add Compound Rewards:** Allows users to add compound rewards to the contract.

      - **Change Mint Fee:** Allows the contract owner to change the mint fee.

  - **PendlePowerFarmController.sol**: This contract that implements the logic for a Pendle Power Farm Controller, that allows users to earn rewards by staking their Pendle (PENDLE) tokens. The controller contract is responsible for managing the staking process and distributing rewards to users.Overall, the `PendlePowerFarmController` contract plays a crucial role in managing various aspects of the Pendle protocol, including reward distribution, market management, and governance, while prioritizing security and efficiency.

    - **Key Features**

      - Allows users to stake their PENDLE tokens to earn rewards.
      - Distributes rewards to users based on their stake and the length of time they have staked.
      - Allows users to withdraw their stake and rewards at any time.
      - Provides a secure and transparent way to stake PENDLE tokens.

    - **Functionalities**

      - **Withdraw LP Tokens:** Allows users to withdraw LP tokens from a specified Pendle market.
      - **Exchange Rewards for Compounding:** Enables users to exchange rewards for compounding with an incentive.
      - **Exchange LP Fees for Pendle:** Allows users to exchange LP fees for Pendle tokens with an incentive.
      - **Skimming:** Provides a function to skim excess assets from a Pendle market.
      - **Adding Pendle Markets:** Allows the addition of new Pendle markets to the protocol.
      - **Updating Reward Tokens:** Updates the reward tokens associated with a Pendle market.
      - **Locking Pendle Tokens:** Enables users to lock Pendle tokens for a specified period.
      - **Claiming Arbitrum Rewards:** Functionality to claim rewards on the Arbitrum network.
      - **Withdrawing Pendle Lock:** Allows users to withdraw locked Pendle tokens.
      - **Claiming Vote Rewards:** Functionality to claim vote rewards.
      - **Forwarding ETH:** Allows the contract to forward ETH to a specified address.
      - **Voting:** Provides functionality for voting on Pendle pools.

    - **Core Logic**

      The core logic of the contract revolves around managing various actions related to yield farming, reward distribution, LP token management, market addition, and voting. It interacts with other Pendle protocol contracts to execute these actions securely and efficiently.

    - **Additional Features**

      - **Reward Distribution:** The contract manages the distribution of rewards earned from yield farming activities, ensuring that users receive their entitled rewards securely.

      - **Dynamic Fee Management:** Users can change the minting fee associated with a Pendle market, providing flexibility in managing market fees.

      - **Locking Mechanism:** Users can lock Pendle tokens for a specified period, earning rewards based on the locking duration.

      - **Arbitrum Integration:** Functionality is provided to claim rewards on the Arbitrum network, enhancing cross-chain interoperability.

      - **Voting Mechanism:** Users can participate in voting on Pendle pools, allowing for community governance and decision-making within the protocol.

  - **PendlePowerFarmControllerBase.sol**: The `PendlePowerFarmControllerBase` contract is a base contract providing core functionalities for managing Pendle Power Farming, including handling compound structures, interacting with Pendle contracts and protocols, managing rewards, exchanging tokens, and facilitating Pendle token locking and claiming on the Arbitrum chain.

  - **PendlePowerFarmControllerHelper.sol**: This contract is a helper contract for the Pendle Power Farm Controller, It provides helper functions and data structures for managing leveraged positions and interacting with the Pendle protocol.

    - **Key Features:**

      - **Utility Functions:** The contract provides utility functions for calculating expiry times, finding indices in arrays, and converting token amounts to ETH and vice versa.

      - **Data Retrieval Methods:** Various external functions allow querying data related to locked positions, reward tokens, user rewards, and active Pendle markets.

      - **Validation Checks:** The contract includes validation checks to ensure correctness and integrity, such as verifying token feed addresses and comparing hashes of reward token arrays.

    - **Functionalities:**

      - **`_calcExpiry`():** Calculates the expiry time based on the number of weeks provided.

      - **`_getExpiry`():** Retrieves the expiry time of the locked position associated with the contract.

      - **`_getLockAmount`():** Retrieves the locked amount associated with the contract.

      - **`_getTokensInETH`():** Converts token amounts to ETH using the Pendle Oracle Hub.

      - **`_getTokensFromETH`():** Converts ETH amounts to tokens using the Pendle Oracle Hub.

      - **`getExpiry`():** External function to fetch the expiry time of the locked position.

      - **`getLockAmount`():** External function to fetch the locked amount associated with the contract.

    - **Core Logic:**

      The core logic of the contract revolves around providing essential helper functions and data retrieval methods required for managing locked positions within the Pendle protocol. It includes calculations, validations, and data manipulation functions necessary for proper contract operation.

    - **Security Measures:**

      - **Feed Validation:** The contract checks the validity of token feed addresses to ensure accurate price information.

      - **Hash Comparison:** Hash comparison is utilized to validate arrays of reward tokens, ensuring consistency and integrity.

    - **Additional Features:**

      - **Data Retrieval:** External functions provide convenient access to various data points, including expiry times, locked amounts, reward tokens, user rewards, and active Pendle markets, enhancing transparency and accessibility for users and developers.

      - **Token Conversion:** Utility functions for converting token amounts to ETH and vice versa using the Pendle Oracle Hub contribute to the interoperability and usability of the contract within the Pendle ecosystem.

  - **PendlePowerFarmTokenFactory.sol**: This contract is a factory for creating Pendle Power Farm Tokens. Pendle Power Farm Tokens are ERC20 tokens that represent a user's share of the rewards earned by a Pendle Power Farm.

    - **Key Features**

      - **Factory Pattern**: The contract uses the factory pattern to create new Pendle Power Farm Tokens. This allows for the creation of new tokens without having to deploy a new contract for each token.
      - **Centralized Control**: The contract is controlled by the Pendle Power Farm Controller, which is responsible for deploying new tokens and managing the rewards distribution.

    - **Functionalities**

      - **deploy()**: Deploys a new Pendle Power Farm Token.
      - **`_clone`()**: Clones the implementation contract to create a new Pendle Power Farm Token.

    - **Core Logic**

      The core logic of the contract is to clone the implementation contract to create a new Pendle Power Farm Token. The implementation contract contains the logic for the token, including the minting and burning of tokens, the distribution of rewards, and the management of the token's metadata.

    - **Security Measures**

      - **Access Control**: Only the Pendle Power Farm Controller can deploy new tokens.
      - **Salt**: A salt is used when cloning the implementation contract to ensure that each new token has a unique address.

- **PowerFarmNFTs**

  - **PowerFarmNFTs.sol**: This contract serves as an `ERC721-compliant` NFT contract responsible for minting, burning, and managing farming NFTs associated with positions within the Pendle protocol, with functionalities including setting the farm contract address, minting and burning NFTs, checking ownership, retrieving NFTs owned by an address, and setting base URI for metadata.

  - **MinterReserver.sol**: `MinterReserver` contract is a utility contract for the Pendle Power Farm that manages the minting and reservation of NFTs used to represent leveraged positions. It keeps track of minted and reserved NFTs, assigns keys to users, and allows users to mint reserved NFTs.

#### FeeManager

- **FeeManager.sol**: The FeeManager contract is responsible for managing the distribution of fees from the platform to various stakeholders, including beneficial owners, incentive owners, and the platform itself. The contract also includes mechanisms for paying back bad debt and managing pool tokens.

  - **Key Features**

    - **Fee Distribution**: The contract allows the fee manager to claim fees from the `wiseLending` contract for each pool and distribute them accordingly.

    - **Incentive Management**: It provides functionality for adjusting and distributing incentives to incentivize various roles within the ecosystem.

    - **Bad Debt Tracking**: The contract keeps track of bad debt associated with each position and provides mechanisms for paying it back using gathered fees.

    - **Pool Management**: It includes functions for adjusting pool fees, mapping pool tokens with their corresponding underlying tokens, and adding/removing pool tokens manually.

    - **Beneficial Address Management**: The contract allows certain addresses to be declared as beneficial, enabling them to claim gathered fees.

  - **Functionalities**

    - **setAaveFlag**: Maps underlying token with corresponding aToken. Sets bool to identify pool token as aToken.
    - **setPoolFee**: Adjusts the paid out incentive percentage for user to reduce bad debt.
    - **setPoolFeeBulk**: Adjusts pool fees in bulk. Fee for each pool can not be greater than 100% or lower than 1%. Can be adjusted for each pool individually.
    - **proposeIncentiveMaster**: Proposes new incentive master. This role can increase the incentive amount for both incentive mappings. These are two roles for incentivising external persons e.g. developers.
    - **claimOwnershipIncentiveMaster**: Claim proposed incentive master by proposed entity.
    - **increaseIncentiveA**: Increase function for increasing incentive amount for entity A. Only callable by incentive master.
    - **increaseIncentiveB**: Increase function for increasing incentive amount for entity B. Only callable by incentive master.
    - **claimIncentivesBulk**: Claims gathered incentives for a specific token.
    - **claimIncentives**: Claims gathered incentives for a specific token.
    - **changeIncentiveUSDA**: Function changing incentiveOwnerA! Only callable by incentiveOwnerA.
    - **changeIncentiveUSDB**: Function changing incentiveOwnerB! Only callable by incentiveOwnerB.
    - **addPoolTokenAddress**: Adds new pool token to pool token list. Called during pool creation and only callable by wiseLending contract.
    - **addPoolTokenAddressManual**: Function to add pool token manualy. Only callable by feeManager master.
    - **removePoolTokenManual**: Function to remove pool token manualy from pool token list. Only callable by feeManager master.
    - **increaseTotalBadDebtLiquidation**: Increase function for total bad debt of wiseLending. Only callable by wiseSecurity contract during liquidation.
    - **setBadDebtUserLiquidation**: Increase function for bad debt of a position. Only callable by wiseSecurity contract during liquidation.
    - **setBeneficial**: Set function to declare an address as beneficial for a fee token. Address can claim gathered fee token as long as it is declared as beneficial. Only setable by master.
    - **revokeBeneficial**: Set function to remove an address as beneficial for a fee token. Only setable by master.
    - **claimWiseFeesBulk**: Claim all fees from wiseLending and send them to feeManager.
    - **claimWiseFees**: Claim fees from wiseLending and send them to feeManager for a specific pool.
    - **claimFeesBeneficial**: Function for beneficial to claim gathered fees. Can only claim fees for which the beneficial is allowed. Can only claim token which are inside the feeManager.
    - **paybackBadDebtForToken**: Function for paying back bad debt of a position. Caller chooses postion, token and receive token. Only gathered fee token can be distributed as receive token. Caller gets 5% more in ETH value as incentive.
    - **paybackBadDebtNoReward**: Function for paying back bad debt of a position. Caller chooses postion, token and receive token. Caller gets no receive token!
    - **getPoolTokenAddressesLength**: Returning the number of pool token addresses saved inside the feeManager.
    - **getPoolTokenAdressesByIndex**: Returns the pool token address at the \_index postion of the array.
    - **syncAllPools**: Bulk function for updating pools - loops through all pools saved inside the poolTokenAddresses array.

  - **Core Logic**

    The core logic of the contract is to manage the distribution of fees from the Wise Lending platform. This is done by tracking the fees earned by each pool and distributing them to the appropriate stakeholders. The contract also includes mechanisms for paying back bad debt and managing pool tokens.

  - **Additional Features**

    - The includes bulk functions for adjusting pool fees, setting Aave flags, and claiming fees/incentives for multiple pools simultaneously.
    - The ability to adjust the pool fee for each pool
    - The ability to propose and claim a new incentive master
    - The ability to add and remove pool tokens

- **FeeManagerHelper.sol**: This `FeeManagerHelper` is a abstract contract with functions for managing fees, bad debts, incentives, and token transfers within the Wise protocol.

  - **Key Features:**

    - Management of borrow and collateral tokens for positions.
    - Adjustment of bad debt amounts for positions and globally.
    - Tracking and updating fee token balances.
    - Setting allowed tokens for users.
    - Distribution of incentives to incentive owners.
    - Calculation of receiving token amounts for callers.

  - **Functionalities:**

    - `_prepareBorrows(uint256 _nftId)`: Internal function to update borrow tokens for a given position by calling `WISE_LENDING.syncManually()` for each borrow token.
    - `_prepareCollaterals(uint256 _nftId)`: Internal function to update collateral tokens for a given position by calling `WISE_LENDING.syncManually()` for each collateral token.
    - `_setBadDebtPosition(uint256 _nftId, uint256 _amount)`: Internal function to set the bad debt amount for a specific position.
    - `_increaseTotalBadDebt(uint256 _amount)`: Internal function to increase the global bad debt amount.
    - `_decreaseTotalBadDebt(uint256 _amount)`: Internal function to decrease the global bad debt amount.
    - `_eraseBadDebtUser(uint256 _nftId)`: Internal function to delete the bad debt amount for a specific position.
    - `_updateUserBadDebt(uint256 _nftId)`: Internal function to update the bad debt amount for a specific position and globally.
    - `_increaseFeeTokens(address _feeToken, uint256 _amount)`: Internal function to increase the balance of a fee token.
    - `_decreaseFeeTokens(address _feeToken, uint256 _amount)`: Internal function to decrease the balance of a fee token.
    - `_setAllowedTokens(address _user, address _feeToken, bool _state)`: Internal function to set allowed tokens for a user.
    - `_setAaveFlag(address _poolToken, address _underlyingToken)`: Internal function to set a flag indicating Aave tokens.
    - `getReceivingToken(address _paybackToken, address _receivingToken, uint256 _paybackAmount)`: Public function to calculate the receiving token amount for a caller.
    - `updatePositionCurrentBadDebt(uint256 _nftId)`: Public function to update the bad debt of a position by preparing collaterals and borrows and then updating the bad debt amount.
    - `_distributeIncentives(uint256 _amount, address _poolToken, address _underlyingToken)`: Internal function to distribute incentives to incentive owners.
    - `_gatherIncentives(address _poolToken, address _underlyingToken, address _incentiveOwner, uint256 _amount)`: Internal function to compute the incentive amount for an incentive owner and reduce the open incentive amount for the owner.
    - `_checkValue(uint256 _value)`: Internal function to check if the passed value is within certain bounds.

  - **Core Logic:**

    The core logic revolves around updating borrow and collateral tokens for positions, adjusting bad debt amounts, tracking fee token balances, setting allowed tokens for users, distributing incentives, and calculating receiving token amounts.

  - **Security Measures:**

    - Use of internal functions for sensitive operations to prevent external interference.
    - Implementation of bounds checking in `_checkValue()` to prevent unexpected behavior due to extreme values.
    - Proper event emissions (`TotalBadDebtIncreased`, `TotalBadDebtDecreased`, `UpdateBadDebtPosition`) to provide transparency and traceability.

  - **Additional Features:**

    - Support for setting allowed tokens for users (`_setAllowedTokens()`).
    - Distribution of incentives to incentive owners (`_distributeIncentives()`).
    - Calculation of receiving token amounts for callers (`getReceivingToken()`).

- **DeclarationsFeeManager.sol**: This contract defines the core data structures and modifiers for the Fee Manager contract in the Wise protocol. It includes mappings for tracking bad debt, fee tokens, incentive amounts, and allowed tokens. It also defines modifiers to restrict access to certain functions based on the sender's identity.

- **FeeManagerEvents.sol**:

  - `ClaimedFeesWiseBulk`: Emitted when fees are claimed in bulk by the Wise contract.
  - `PoolTokenRemoved`: Emitted when a pool token is removed from the FeeManager.
  - `PoolTokenAdded`: Emitted when a pool token is added to the FeeManager.
  - `IncentiveOwnerBChanged`: Emitted when the incentive owner B is changed.
  - `IncentiveOwnerAChanged`: Emitted when the incentive owner A is changed.
  - `ClaimedOwnershipIncentiveMaster`: Emitted when the ownership of the incentive master is claimed.
  - `IncentiveMasterProposed`: Emitted when a new incentive master is proposed.
  - `PoolFeeChanged`: Emitted when the pool fee is changed for a pool token.
  - `ClaimedIncentives`: Emitted when incentives are claimed by a user.
  - `ClaimedIncentivesBulk`: Emitted when incentives are claimed in bulk by the Wise contract.
  - `IncentiveIncreasedB`: Emitted when the incentive B is increased.
  - `IncentiveIncreasedA`: Emitted when the incentive A is increased.
  - `BadDebtIncreasedLiquidation`: Emitted when the bad debt is increased due to a liquidation.
  - `TotalBadDebtIncreased`: Emitted when the total bad debt is increased.
  - `TotalBadDebtDecreased`: Emitted when the total bad debt is decreased.
  - `SetBadDebtPosition`: Emitted when the bad debt position is set for an NFT.
  - `UpdateBadDebtPosition`: Emitted when the bad debt position is updated for an NFT.
  - `SetBeneficial`: Emitted when a user is set as beneficial for a token.
  - `RevokeBeneficial`: Emitted when a user is revoked as beneficial for a token.
  - `ClaimedFeesWise`: Emitted when fees are claimed by the Wise contract.
  - `ClaimedFeesBeneficial`: Emitted when fees are claimed by a beneficial user.
  - `PayedBackBadDebt`: Emitted when bad debt is paid back for an NFT.
  - `PayedBackBadDebtFree`: Emitted when bad debt is paid back for an NFT without using any collateral.

#### WiseOracleHub

- **WiseOracleHub.sol**: WiseOracleHub contract is an implementation of a oracle hub that provides price feeds for tokens. It allows users to add new price feeds, and to query the latest price of a token.

  - **Key Features**

    - **Price Feeds Management**: Allows addition of new price feed contracts for tokens, preventing overwriting of existing mappings.
    - **ETH Value Conversion**: Provides functions to convert token amounts to their equivalent ETH value and vice versa.
    - **Heartbeat Checks**: Includes functionality to check if Chainlink price feeds are being updated within expected timeframes, with the ability to recalibrate expected heartbeat intervals.

  - **Functionalities:**

    - **Latest Resolver**: Retrieves the latest ETH value of a token by passing the underlying token address, leveraging the `latestRoundData` function for Chainlink oracles.
    - **Adding Oracles**: Supports addition of new price feed contracts for tokens, ensuring uniqueness of mappings.
    - **Heartbeat Checks**: Checks if Chainlink feeds are updated within expected timeframes, aiding in assessing the reliability of price data.
    - **ETH Value Conversion**: Facilitates conversion between token amounts and their corresponding ETH values, considering token decimals and price feed accuracy.

  - **Core Logic:**

    - Users can add new price feeds for tokens.
    - Users can query the latest price of a token by passing the token address.
    - The contract includes a number of security measures to protect against fraud and manipulation.

  - **Security Measures:**

    - Checks to ensure that the price feed is valid before adding it to the contract.
    - Checks to ensure that the price feed is not being manipulated before returning the latest price.

  - **Additional Features:**

    - **Recalibration**: Provides functions for recalibrating expected heartbeat intervals for pricing feeds, allowing adjustments based on changing network conditions or oracle performance.
    - **Bulk Operations**: Supports bulk operations for adding multiple price feed contracts or recalibrating heartbeat intervals for multiple tokens simultaneously, enhancing efficiency in managing oracle data.
    - Allows users to specify the maximum number of price feeds that can be added to the contract.
    - Allows users to specify the minimum number of price feeds that must be present before the contract can be used.

- **OracleHelper.sol**: The contract `OracleHelper` is an abstract contract designed to assist in managing price oracles for various tokens. It facilitates the addition of price feeds, management of TWAP oracles, and validation of price data obtained from external sources. The contract includes functions to add oracles, retrieve price data, validate oracle responses, and manage TWAP oracles.

  - **Key Features**:

    - **Price Feed Management**: Allows for the addition and management of price feeds for tokens.
    - **TWAP Oracle Support**: Supports the management and retrieval of TWAP price data for tokens.
    - **Chainlink Integration**: Integrates with Chainlink oracles to obtain price data.
    - **Validation Mechanisms**: Includes functions to validate price data obtained from oracles.
    - **Security Measures**: Implements checks to ensure the integrity of oracle data and prevent manipulation.
    - **Heartbeat Recalibration**: Implements a method to recalibrate the expected heartbeat interval for Chainlink oracles based on historical data.

  - **Functionalities**:

    - **`_addOracle`:** Adds a Chainlink oracle for a given token address.
    - **`_addAggregator`:** Adds a Uniswap TWAP oracle for a given token address.
    - **`_checkFunctionExistence`:** Checks if a function exists in a given contract.
    - **`_compareMinMax`:** Compares the latest answer from a Chainlink oracle to its minimum and maximum values.
    - **`latestResolverTwap`:** Returns the latest TWAP price for a given token address.
    - **`_validateAnswer`:** Validates the answer from a Chainlink oracle and compares it to the TWAP price.
    - **`_writeUniTwapPoolInfoStruct`:** Stores TWAP pool information for a given token address.
    - **`_writeUniTwapPoolInfoStructDerivative`:** Stores TWAP pool information for a given token address and its derivative.
    - **`_getRelativeDifference`:** Calculates the relative difference between two values.
    - **`_compareDifference`:** Compares the relative difference to an allowed threshold.
    - **`_getChainlinkAnswer`:** Retrieves the latest answer from a Chainlink oracle.
    - **`getETHPriceInUSD`:** Returns the ETH price in USD using a Chainlink oracle.
    - **`_getPool`:** Retrieves the pool address for a given pair of tokens and fee from Uniswap V3 Factory.
    - **`_validateTokenAddress`:** Validates that a given token address is not the zero address and matches the expected tokens.
    - **`_validatePoolAddress`:** Validates that a given pool address matches the expected pool.
    - **`_validatePriceFeed`:** Validates that a Chainlink price feed is set for a given token address.
    - **`_validateTwapOracle`:** Validates that a TWAP oracle is not already set for a given token address.
    - **`_getTwapPrice`:** Returns the TWAP price for a given token address using a Uniswap TWAP oracle.
    - **`_getOneUnit`:** Returns a unit value for a given token address based on its decimals.
    - **`_getAverageTick`:** Calculates the average tick for a given Uniswap TWAP oracle.
    - **`decimals`:** Returns the decimals of a given token address.
    - **`_getTwapDerivatePrice`:** Returns the TWAP price for a derivative token using a combination of TWAP oracles.
    - **`_recalibrate`:** Recalibrates the expected heartbeat value for a Chainlink oracle.
    - **`_recalibratePreview`:** Calculates the expected heartbeat value for a Chainlink oracle.
    - **`sequencerIsDead`:** Checks if the Arbitrum sequencer is not functioning correctly.
    - **`_chainLinkIsDead`:** Checks if a Chainlink oracle is not functioning correctly.
    - **`getLatestRoundId`:** Returns the latest round ID for a Chainlink oracle.
    - **`_getRoundTimestamp`:** Returns the timestamp of a specific round for a Chainlink oracle.

  - **Core Logic**:

    - Adding and managing oracles for tokens.
    - Retrieving price data from oracles and TWAP sources.
    - Validating the accuracy and integrity of price data.
    - Managing TWAP oracles and their associated information.
    - Recalibrating expected heartbeat values for pricing feeds.

  - **Security Measures**:

    - **Oracle Validation:** The contract validates oracles before using them to ensure they are functioning correctly.
    - **Answer Comparison:** It compares the answer from a Chainlink oracle to the TWAP price to detect potential discrepancies.
    - **Heartbeat Recalibration:** The heartbeat interval for Chainlink oracles is recalibrated based on historical data to ensure timely updates.

  - **Additional Features**:

    - **TWAP Oracle Support**: Provides support for TWAP oracles, allowing for the retrieval of time-weighted average prices.
    - **Chainlink Integration**: Integrates with Chainlink oracles to obtain accurate and reliable price data.
    - **ETH Price Retrieval:** It provides a function to retrieve the ETH price in USD using a Chainlink oracle.
    - **Recalibration Mechanism**: Includes functionality to recalibrate expected heartbeat values for pricing feeds, ensuring the accuracy of price data over time.
    - **Sequencer Health Check:** It includes a function to check if the Arbitrum sequencer is not functioning correctly.

- **Declarations.sol**: This contract defines the core data structures and constants used by the Pendle Oracle Hub contract. It includes mappings for token decimals, price feeds, heartbeats, token aggregators, underlying feed tokens for multi-token derivative oracles, Uniswap TWAP pool or derivative info, and derivative partner addresses for TWAPs. It also defines various constants and immutable variables related to precision, time periods, and error handling.

#### WrapperHub

- **AaveHub.sol**: The `AaveHub` contract is designed to optimize capital efficiency by utilizing Aave pools. It facilitates the deposit, withdrawal, borrowing, and repayment of assets in Aave pools, managing accounting through position NFTs. Additionally, it includes functionalities for setting Aave token addresses, handling ETH transactions, and managing borrowings and repayments.

  - **Key Features:**

    - **Aave Pool Integration:** Utilizes Aave pools to optimize capital efficiency by earning supply APY on deposited assets.
    - **Position NFT Accounting:** Manages accounting through position NFTs, allowing for efficient tracking of deposited assets.
    - **Bulk Operations:** Supports bulk operations for setting Aave token addresses, enhancing efficiency in managing Aave pool integrations.
    - **ETH Handling:** Handles ETH transactions, allowing for deposit, withdrawal, borrowing, and repayment of ETH directly from/to Aave pools.

  - **Functionalities:**

    - **Setting Aave Token Addresses:** Allows the master to set Aave token addresses, linking underlying assets with corresponding aTokens.
    - **Deposit:** Allows users to deposit ERC20 tokens or ETH into Aave pools, minting position NFTs to reduce transaction overhead.
    - **Withdrawal:** Enables users to withdraw deposited assets or ETH from Aave pools, with options for exact amount or exact shares.
    - **Borrowing:** Facilitates borrowing of ERC20 tokens or ETH from Aave pools, requiring supplied collateral within the same position.
    - **Repayment:** Allows users to repay borrowed assets or ETH to Aave pools, reducing borrowed shares and potentially earning supply APY.

  - **Core Logic:**

    - **Aave Pool Integration:** Manages Aave pool integrations by setting Aave token addresses and handling deposit, withdrawal, borrowing, and repayment operations.
    - **Position NFT Accounting:** Tracks deposited assets and borrowings through position NFTs, ensuring accurate accounting and efficient management of user funds.
    - **ETH Handling:** Handles ETH transactions by wrapping and unwrapping ETH to/from WETH, allowing seamless integration with Aave pools.

  - **Additional Features:**

    - **Bulk Operations:** Supports bulk operations for setting Aave token addresses, enabling efficient management of multiple token mappings simultaneously.
    - Allows users to specify the maximum amount of assets that they want to borrow.
    - Allows users to specify the minimum amount of assets that they want to deposit.
    - Allows users to set a stop-loss order to protect their assets from liquidation.

- **AaveHelper.sol**: The contract `AaveHelper` is an abstract contract that provides various helper functions for interacting with the Aave lending protocol within the broader context of the `Declarations` contract. It includes functions for managing reentrancy, validating Aave tokens, depositing, withdrawing, borrowing, and repaying assets on Aave, as well as additional functions for preparing collaterals and borrows, retrieving Aave pool APY, and handling value transfers.

  - **Key Features:**

    - **Helper Functions**: The contract provides helper functions to facilitate interactions with the Aave lending protocol, ensuring safer and more efficient operations.

    - Uses the Aave lending protocol to earn interest on deposited assets and to borrow assets at a low interest rate.

  - **Functionalities:**

    - **Token Validation**: The `validToken` modifier checks if a given pool token is valid by verifying if it has any deposit shares in the WISE_LENDING contract.

    - **Asset Interactions**: Functions such as `_wrapDepositExactAmount`, `_wrapWithdrawExactAmount`, `_wrapWithdrawExactShares`, and `_wrapBorrowExactAmount` facilitate actions like depositing, withdrawing, and borrowing assets on Aave.

    - **Value Transfer Handling**: The `_sendValue` function handles value transfers, ensuring that the contract has enough balance before transferring funds and guards against reentrancy.

    - **APY Retrieval**: The `getAavePoolAPY` function retrieves the current liquidity rate of an Aave pool, providing insight into the potential returns for supplying assets to that pool.

  - **Core Logic:**

    The core logic revolves around interacting with the Aave lending protocol to perform actions such as depositing, withdrawing, and borrowing assets. It ensures that these interactions are performed safely and efficiently, protecting against potential vulnerabilities such as reentrancy attacks and ensuring the validity of tokens being manipulated.

  - **Additional Features:**

    - **APY Retrieval**: The contract includes a function to retrieve the current liquidity rate of Aave pools, providing users with valuable information for making informed decisions about their asset allocations.

    - **Value Transfer Handling**: The contract handles value transfers securely, checking the contract's balance before initiating transfers and guarding against reentrancy to prevent potential exploits.

- **Declarations.sol**: This contract defines the core data structures and constants used by the Aave Hub contract. It includes mappings for Aave token addresses, as well as various constants and immutable variables related to precision, maximum amounts, and error handling. It also defines functions for checking ownership and position lock status, and a function for setting the WiseSecurity contract address.

- **AaveEvents.sol**: This contract defines the events emitted by the Aave Hub contract. These events include:

  - SetAaveTokenAddress: Emitted when the Aave token address is set for an underlying asset.

  - IsDepositAave: Emitted when a deposit is made to Aave for an NFT.

  - IsWithdrawAave: Emitted when a withdrawal is made from Aave for an NFT.

  - IsBorrowAave: Emitted when a borrow is made from Aave for an NFT.

  - IsPaybackAave: Emitted when a payback is made to Aave for an NFT.

  - IsSolelyDepositAave: Emitted when a solely deposit is made to Aave for an NFT.

  - IsSolelyWithdrawAave: Emitted when a solely withdrawal is made from Aave for an NFT.

#### TransferHub

- **SendValueHelper.sol**: This contract defines a helper function to send ETH to an address. It includes error handling for cases where the contract does not have enough ETH or the send operation fails.

- **WrapperHelper.sol**: This contract defines helper functions for wrapping and unwrapping ETH using the WETH contract.

- **CallOptionalReturn.sol**: This contract defines a helper function to make low-level calls to other contracts and check the return value. It is useful for interacting with contracts that do not follow the ERC20 standard or that have custom return values.

- **TransferHelper.sol**: This contract defines helper functions to safely transfer and transferFrom tokens. It uses the CallOptionalReturn contract to check the return value of the transfer and transferFrom calls and revert if the call was unsuccessful or the token does not follow the ERC20 standard.

**ApprovalHelper.sol**: This contract defines a helper function to safely approve tokens. It uses the `CallOptionalReturn` contract to check the return value of the approve call and revert if the call was unsuccessful or the token does not follow the ERC20 standard.

#### DerivativeOracles

- **PendleLpOracle.sol**: `PendleLpOracle` contract is a price feed oracle for Pendle LP tokens. It combines the ETH value of the underlying asset (e.g., USDC) from a Chainlink price feed with the LP token's exchange rate to ETH from a Pendle oracle to provide the latest ETH value for the LP token.

  - **Key Features**

    - **Pendle Integration**: The contract integrates with the Pendle protocol to access the LP token's exchange rate to ETH.
    - **Chainlink Integration**: The contract uses a Chainlink price feed to obtain the ETH value of the underlying asset.
    - **TWAP Calculation**: The contract uses a time-weighted average price (TWAP) to smooth out price fluctuations.

  - **Functionalities**

    - **latestAnswer()**: Returns the latest ETH value for the LP token.
    - **decimals()**: Returns the number of decimals for the price feed (18).
    - **latestRoundData()**: Returns the latest round data for the price feed, including the round ID, answer, timestamps, and answered-in-round.

  - **Core Logic**

    The core logic of the contract is to combine the ETH value of the underlying asset from the Chainlink price feed with the LP token's exchange rate to ETH from the Pendle oracle. This is done by multiplying the Chainlink answer by the LP token's exchange rate and dividing by the precision factor (1E18).

  - **Additional Features**

    The contract includes a function to retrieve the LP token's exchange rate to ETH using a TWAP calculation. This function is used internally by the contract but can also be called externally.

- **PtOracleDerivative.sol**: This contract is a price feed oracle for Pendle PtTokens. It combines the USD value of the underlying asset (e.g., USDC) from a Chainlink price feed with the PtToken's exchange rate to USD from a Pendle oracle to provide the latest USD value for the PtToken.

  - **Key Features:**

    - Integration with ChainLink price feeds: The contract interfaces with ChainLink price feeds for both USD and ETH, allowing for accurate price conversion.
    - TWAP calculation: Utilizes Time-Weighted Average Price (TWAP) to ensure a robust and reliable price calculation mechanism.
    - Oracle state verification: Checks for the required cardinality and oldest observation to ensure the accuracy and reliability of the oracle's data.

  - **Functionalities:**

    - **latestAnswer():** Returns the latest price of PtToken in USD by combining data from USD and ETH ChainLink price feeds and TWAP calculations.
    - **decimals():** Returns the number of decimals used for the price feeds.
    - **latestRoundData():** Provides detailed information about the latest round data, including the round ID, answer, timestamp, and more.

  - **Core Logic:**

    - Obtains the latest prices from the USD and ETH ChainLink price feeds.
    - Checks the oracle state to ensure that the required cardinality and oldest observation conditions are satisfied.
    - Calculates the PtToken to asset rate using TWAP.
    - Combines the obtained prices and TWAP-calculated rate to derive the latest price of PtToken in USD.

  - **Additional Features:**

    - **Flexibility:** The contract allows for easy integration with other smart contracts or systems requiring access to the latest PtToken price.
    - **Decimals control:** Provides control over the number of decimals used for the price feeds, ensuring compatibility with various systems and standards.
    - **Detailed round data:** Offers detailed information about the latest round data, allowing for transparency and traceability of the price calculation process.

- **PtOraclePure.sol**: This contract is a price feed oracle for Pendle `PtTokens`. It combines the ETH value of the underlying asset (e.g., USDC) from a Chainlink price feed with the PtToken's exchange rate to ETH from a Pendle oracle to provide the latest ETH value for the PtToken.

- **PendleChildLpOracle.sol**: PendleChildLpOracle contract is a custom oracle setup for Pendle child LP tokens. It combines the price feed of the Pendle LP oracle with the total LP assets and total supply of the Pendle child token to provide the latest USD value of the child LP token.

- **CustomOracleSetup.sol**: This contract defines a custom oracle setup that allows the owner to set the last update time, round data, and global aggregator round ID. It also includes a function to get the current timestamp.

### Roles

1.  **WiseLending.sol**:

    1. **User/Borrower/Lender**: The contract allows users to deposit and borrow tokens against their collateral. Users can interact with the contract by calling functions like `depositExactAmountETH`, `borrowExactAmountETH`, and others.
    2. **Liquidator**: In case a user's debt ratio exceeds 100%, a liquidator can liquidate a position partially or fully by calling the `liquidatePartiallyFromTokens` function.
    3. **WiseOracleHub**: This contract relies on the WiseOracleHub for price information and other security checks. The WiseOracleHub address is passed as a constructor argument.
    4. **NFT Contract**: An NFT contract is used to represent users' positions in the system. The NFT contract address is passed as a constructor argument, and functions like `_reservePosition` are used to interact with it.
    5. **Master Contract**: The master contract address is passed as a constructor argument and is used to forward Ether received by the contract.
    6. **aaveHub**: Some functions, like `withdrawOnBehalfExactAmount` and `borrowOnBehalfExactAmount`, are marked with the `onlyAaveHub` modifier, which means that only the aaveHub contract can call these functions.
    7. **Fee Manager**: The `corePaybackFeeManager` function is marked with the `onlyFeeManager` modifier, indicating that only the fee manager contract can call this function.
    8. **Isolation Pool**: The `setRegistrationIsolationPool` function allows registering a position for isolation pool functionality. This might represent a separate contract or module within the system.
    9. **LASA**: LASA is not a direct role in the contract but is an algorithm used to automatically adjust bonding curves for different asset types. It's mentioned in the contract's documentation.

2.  **WiseSecurity.sol**:

    1. **WiseLending**: This contract interacts with the WiseLending contract to perform various actions like borrowing, lending, and liquidation. The WiseLending contract address is passed as a constructor argument, and functions like `checksBorrow`, `checksWithdraw`, and others are used to interact with it.
    2. **Master**: The master is an address with special permissions to perform certain actions like setting liquidation incentives, blacklisting tokens, and revoking a shutdown. The master address is passed as a constructor argument, and the `onlyMaster` modifier is used to restrict certain functions to be called only by the master.
    3. **aaveHub**: The aaveHub address is passed as a constructor argument, and it is used in the contract's constructor. However, the contract does not seem to interact directly with the aaveHub contract.
    4. **Curve Pool**: The contract interacts with Curve pools to swap tokens and update internal values. Functions like `prepareCurvePools` and `curveSecurityCheck` are used for this purpose.
    5. **Security Worker**: A security worker is a special role that can be set by the master to perform a security shutdown of all active pools. The `setSecurityWorker` function is used to add or remove security workers, and the `securityShutdown` function can be called by a security worker to shut down the pools.
    6. **User/Borrower/Lender**: The contract allows users to borrow, lend, and liquidate positions. Users can interact with the contract by calling functions like `getLiveDebtRatio`, `maximumBorrowToken`, and others.
    7. **Oracle**: The contract uses an oracle (WISE_ORACLE) to fetch token prices and other data. The oracle is not directly defined as a role in the contract but is an essential component of the system.
    8. **ApprovalHelper**: The contract inherits from ApprovalHelper, which provides functionality for managing approvals. This role is not explicitly mentioned but is inherited from the parent contract.

3.  **FeeManager.sol**:

    1. `Master` - Has administrative permissions to set pool fees, add/remove pool tokens, set aave flags, etc. Usually the owner/governance of the protocol.

    2. `IncentiveMaster` - Can propose and increase incentive amounts for incentive mappings. Used to incentivize external entities.

    3. `WiseLending` - Only it can add new pool tokens when pools are created. Can call functions related to pool/position management.

    4. `WiseSecurity` - Only it can increase bad debt amounts during liquidations.

    5. `FeeManager itself` - Collects fees from lending pools and distributes them as incentives. Manages fee distributions, bad debt tracking, and incentive payments.

    6. `IncentiveOwnerA/B` - Addresses that receive incentives from the protocol. Can update these addresses.

    7. `Beneficial addresses` - Addresses that can claim fees for approved tokens. Set by the master.

4.  **WiseCore.sol**:

    1. `MainHelper`: This is a parent contract that provides common functionality for the `WiseCore` contract.
    2. `TransferHelper`: This is another parent contract that provides functionality for handling transfers of assets.
    3. `powerFarms`: This is a role that can bypass certain security checks and have special privileges when interacting with the contract.
    4. `aaveHub`: This is a role that can perform certain actions on behalf of other users, such as depositing or withdrawing funds.
    5. `wiseOracle`: This is a role that provides price data for various assets.
    6. `wiseSecurity`: This is a role that performs security checks and ensures that the contract is operating correctly.
    7. `positionLendTokenData`: This is a mapping that stores information about the tokens that are being lent out by the contract.
    8. `positionBorrowTokenData`: This is a mapping that stores information about the tokens that are being borrowed by the contract.
    9. `pureCollateralAmount`: This is a mapping that stores information about the amount of collateral that is available for each user.
    10. `userLendingData`: This is a mapping that stores information about the lending activity of each user.
    11. `userBorrowShares`: This is a mapping that stores information about the borrowing activity of each user.
    12. `globalPoolData`: This is a mapping that stores information about the overall state of the contract.
    13. `lendingPoolData`: This is a mapping that stores information about the lending pools.
    14. `borrowPoolData`: This is a mapping that stores information about the borrowing pools.
    15. `timestampsPoolData`: This is a mapping that stores information about the timestamp of the last action performed on a pool.
    16. `hashMapPositionLending`: This is a mapping that stores information about the positions of the lending pools.
    17. `hashMapPositionBorrow`: This is a mapping that stores information about the positions of the borrowing pools.

5.  **PositionNFTs.sol**:

    1. `Master`: The owner of the contract who can add new mappings between underlying assets and corresponding aTokens, and can also skim aave tokens from the contract.
    2. `ReserveRole`: A role that can reserve positions for users.
    3. `FeeManager`: A role that can manage fees for the contract.
    4. `User`: Any user who can mint a position NFT and use the WiseLending protocol.
    5. `Contract`: The PositionNFTs contract itself, which manages the minting and reserving of position NFTs.

6.  **PowerFarmNFTs.sol**:

    1. `onlyMaster`: This modifier restricts access to certain functions to only the contract's master, which is the address that deployed the contract.
    2. `onlyFarmContract`: This modifier restricts access to certain functions to only the farm contract, which is the address of the contract that manages the farming process.
    3. `IFarmContract`: This interface defines the functions that the farm contract must implement in order to work with the PowerFarmNFTs contract.
    4. `OwnableMaster`: This contract inherits from OpenZeppelin's `Ownable` contract and adds additional functionality for managing ownership and permissions.
    5. `ERC721Enumerable`: This contract inherits from OpenZeppelin's `ERC721` contract and adds enumeration functionality for the NFTs.

7.  **MinterReserver.sol**:

    1. `MinterReserver`: This is the main contract that contains all the logic for minting and reserving NFTs. It has the following roles:

       - `onlyKeyOwner`: This modifier checks if the caller is the owner of a specific NFT.
       - `_reserveKey`: This function reserves an NFT for a specific user.
       - `_mintKeyForUser`: This function mints an NFT for a specific user.
       - `mintReserved`: This function allows a user to mint an NFT that they have previously reserved.

    2. `IPowerFarmsNFTs`: This is an interface that defines the functions that the `MinterReserver` contract needs to interact with the `PowerFarmsNFTs` contract.
    3. `FARMS_NFTS`: This is a variable that holds the address of the `PowerFarmsNFTs` contract.
    4. `totalMinted`: This is a variable that keeps track of the total number of NFTs that have been minted.
    5. `totalReserved`: This is a variable that keeps track of the total number of NFTs that have been reserved.
    6. `availableNFTCount`: This is a variable that keeps track of the number of NFTs that are currently available for minting.
    7. `farmingKeys`: This is a mapping that maps each NFT to its corresponding farm NFT.
    8. `reservedKeys`: This is a mapping that maps each user to their reserved NFT.
    9. `availableNFTs`: This is a mapping that maps each NFT to its corresponding available NFT.

8.  **OwnableMaster.sol**:

    1. `master`: The current owner of the contract.
    2. `proposedMaster`: The proposed new owner of the contract.
    3. `ZERO_ADDRESS`: A constant representing the zero address.

### Invariants Generated

1.  **WiseLending.sol**:

    1. `Total lending shares equals total deposits`:

       - This ensures the lending pool is fully accounted for. The total deposits placed in the pool in terms of the underlying asset (e.g. ETH, DAI) must equal the total shares issued to depositors.

    2. `Total borrow shares equals total borrows`:

       - Similarly for the borrowing side, the total tokens borrowed from the pool must be fully represented by the shares issued to borrowers.

    3. `User collateral <= deposited collateral`:

       - A user's collateral amount stored on the smart contract can't exceed what they originally deposited. This prevents collateral from being created out of thin air.

    4. `User borrow <= collateral`:

       - The amount a user borrows can't exceed the value of collateral they have deposited. This ensures the lending protocol maintains over-collateralized loans.

    5. `Pool borrow <= total collateral`:

       - At the pool level, the total tokens borrowed from the pool must always be less than or equal to the total collateral in the pool. Prevents the pool from becoming under-collateralized.

    6. `Lending/Borrow share price bounds`: - The lending and borrowing share prices are used to calculate withdrawal/repayment amounts. Bounding them ensures amounts can always be fully redeemed.

    7. `Withdrawals/paybacks <= deposits/borrows`:

       - Withdrawal and repayment amounts can't exceed what's actually been deposited/borrowed to the smart contract.

    8. The `_reservePosition()` function always returns a unique and unused NFT ID. This invariant ensures that each user has a unique position in the system.

    9. The `_getSharePrice()` function always returns a valid share price for the given pool token. This invariant ensures that the share price is always within the acceptable range and can be used for calculations.

    10. The `_checkHealthState()` function always ensures that a user's position is healthy and has sufficient collateral to cover their debt. This invariant prevents under-collateralized positions and ensures the overall stability of the system.

    11. The `_syncPoolBeforeCodeExecution()` function always updates the pseudo amounts for the given pool token. This invariant ensures that the pool data is up-to-date and accurate.

    12. The `_syncPoolAfterCodeExecution()` function always updates the borrow pool rate and share price for the given pool token. This invariant ensures that the borrowing costs are accurately reflected in the pool data.

    13. The `_depositExactAmount()` function always mints the correct number of lending shares for the given deposit amount. This invariant ensures that users receive the correct number of shares for their deposits.

    14. The `_withdrawExactAmount()` function always burns the correct number of lending shares for the given withdrawal amount. This invariant ensures that users can only withdraw the amount they have deposited.

    15. The `_handleSolelyDeposit()` function always increases the pure collateral amount for the given NFT ID and pool token. This invariant ensures that solely deposited funds are tracked correctly.

    16. The `_handleSolelyWithdraw()` function always decreases the pure collateral amount for the given NFT ID and pool token. This invariant ensures that solely withdrawn funds are tracked correctly.

    17. The `_corePayback()` function always decreases the borrower's debt and increases the lending pool's balance for the given pool token. This invariant ensures that the payback function is working correctly and that the borrower's debt is accurately reflected in the pool data.

2.  **WiseSecurity.sol**:

    1. `msg.sender` should be equal to `WISE_LENDING` address when calling functions with the `onlyWiseLending` modifier.
    2. `_nftId` should be owned by `_caller` when calling `checksCollateralizeDeposit`.
    3. The sum of the weighted collateral and the borrowed amount should satisfy the borrowing percentage cap condition when calling `maximumBorrowToken`.
    4. The total borrowed amount should not exceed the total collateral amount when calling `checksLiquidation`.
    5. The `_shareAmountToPay` should not exceed the maximum allowed shares when calling `checksLiquidation`.
    6. The `_poolToken` should not be blacklisted when calling `checksBorrow`.
    7. The `_poolToken` should be an allowed token when calling `checksBorrow`.
    8. The `_poolToken` should have a valid Chainlink heartbeat when calling `checksCollateralizeDeposit`.
    9. The position should not have an open borrow position if the `_poolToken` is blacklisted when calling `checksWithdraw` or `checkUncollateralizedDeposit`.
    10. The position should not be in an unhealthy state when calling `checkUncollateralizedDeposit`.
    11. The total borrowed amount should not exceed the total collateral amount when calling `checkBadDebtLiquidation`.
    12. The `_poolToken` should not be blacklisted when calling `maximumWithdrawToken` or `maximumWithdrawTokenSolely` for collateralized positions.
    13. The expected lending amount should not exceed the total pool amount when calling `maximumWithdrawToken`.
    14. The expected payback amount should not exceed the total borrowed shares when calling `getExpectedPaybackAmount`.
    15. The expected lending amount should not exceed the total deposit shares when calling `getExpectedLendingAmount`.
    16. The `_entitiy` should not be a security worker when calling `setSecurityWorker` with `_state` as `false`.
    17. The pool state should be set to `true` when calling `securityShutdown`.
    18. The pool state should be set to `false` when calling `revokeShutdown`.
    19. The `_newMinDepositValue` should not be zero when calling `changeMinDepositValue`.
    20. The deposit amount should be greater than or equal to the minimum deposit value when calling `checkPoolWithMinDeposit` or `checkMinDepositValue`.

3.  **MainHelper.sol**:

    1. totalDepositShares can never be negative:
       `lendingPoolData[_poolToken].totalDepositShares >= 0`

    2. pseudoTotalPool can never be negative:  
       `lendingPoolData[_poolToken].pseudoTotalPool >= 0`

    3. totalBorrowShares can never be negative:
       `borrowPoolData[_poolToken].totalBorrowShares >= 0`

    4. pseudoTotalBorrowAmount can never be negative:
       `borrowPoolData[_poolToken].pseudoTotalBorrowAmount >= 0`

    5. user's lending shares for a pool token can never be negative:
       `userLendingData[_nftId][_poolToken].shares >= 0`

    6. user's borrow shares for a pool token can never be negative:
       `userBorrowShares[_nftId][_poolToken] >= 0`

    7. pool utilization is between 0-1:
       `globalPoolData[_poolToken].utilization >= 0 && globalPoolData[_poolToken].utilization <= PRECISION_FACTOR_E18`

    8. borrow rate is non-negative:
       `borrowPoolData[_poolToken].borrowRate >= 0`

    9. positionNFT owner matches external caller:
       `POSITION_NFT.ownerOf(_nftId) == msg.sender`

    10. nftIds are unique and sequential

4.  **WiseSecurityHelper.sol**:

    **Invariant 1:**

          ```
          overallETHBorrow(nftId) < overallETHCollateralsWeighted(nftId)
          ```

    **Description:**
    This invariant ensures that the total borrow amount for a position is always less than the total weighted collateral amount. This prevents the position from becoming undercollateralized and being liquidated.

    **Invariant 2:**

            ```
            overallETHBorrow(nftId) < overallETHCollateralsBare(nftId)
            ```

    **Description:**
    This invariant ensures that the total borrow amount for a position is always less than the total unweighted collateral amount. This prevents the position from becoming undercollateralized and being liquidated.

    **Invariant 3:**

            ```
            getPositionLendingAmount(nftId, poolToken) > 0
            ```

    **Description:**
    This invariant ensures that the lending amount for a position is always greater than 0. This prevents the position from being liquidated due to a lack of collateral.

    **Invariant 4:**

            ```
            getPositionBorrowAmount(nftId, poolToken) > 0
            ```

    **Description:**
    This invariant ensures that the borrow amount for a position is always greater than 0. This prevents the position from being liquidated due to a lack of debt.

    **Invariant 5:**

            ```
            checkOwnerPosition(nftId, caller)
            ```

    **Description:**
    This invariant ensures that the caller is the owner of the position. This prevents unauthorized access to the position.

    **Invariant 6:**

            ```
            checkBlacklisted(poolToken)
            ```

    **Description:**
    This invariant ensures that the pool token is not blacklisted. This prevents the position from being liquidated due to a blacklisted pool token.

    **Invariant 7:**

            ```
            _checkSuccess(success)
            ```

    **Description:**
    This invariant ensures that the low level byte call of a function with {.call()} was successful. This prevents the position from being liquidated due to a failed function call.

5.  **PendlePowerFarmToken.sol**:

    **Invariant 1:** `underlyingLpAssetsCurrent` is always less than or equal to the total amount of LP assets that have been distributed.

    **Invariant 2:** `totalLpAssetsToDistribute` is always less than or equal to the total amount of LP assets that have been distributed.

    **Invariant 3:** The share price is always greater than or equal to the initial share price.

    **Invariant 4:** The share price growth is always less than or equal to the maximum allowed share price growth.

    **Invariant 5:** The mint fee is always less than or equal to the maximum allowed mint fee.

    **Invariant 6:** The number of shares is always greater than or equal to the number of shares that have been minted.

    **Invariant 7:** The number of shares that have been burned is always less than or equal to the number of shares that have been minted.

    **Invariant 8:** The total amount of LP assets that have been distributed is always greater than or equal to the total amount of LP assets that have been withdrawn.

    **Invariant 9:** The total amount of LP assets that have been withdrawn is always less than or equal to the total amount of LP assets that have been distributed.

    **Invariant 10:** The total amount of LP assets that have been distributed is always greater than or equal to the total amount of LP assets that have been burned.

    **Invariant 11:** The total amount of LP assets that have been burned is always less than or equal to the total amount of LP assets that have been distributed.

6.  **FeeManager.sol**:

    **Invariant 1:**

            ```solidity
            forall i. 0 <= i < getPoolTokenAddressesLength() ==> poolTokenAddresses[i] != ZERO_ADDRESS
            ```

    **Invariant 2:**

            ```solidity
            totalBadDebtETH >= 0
            ```

    **Invariant 3:**

            ```solidity
            forall i. 0 <= i < getPoolTokenAddressesLength() ==> allowedTokens[FEE_MANAGER_NFT][poolTokenAddresses[i]] = true
            ```

    **Invariant 4:**

            ```solidity
            forall i. 0 <= i < getPoolTokenAddressesLength() ==> allowedTokens[INCENTIVE_OWNER_A][poolTokenAddresses[i]] = true
            ```

    **Invariant 5:**

            ```solidity
            forall i. 0 <= i < getPoolTokenAddressesLength() ==> allowedTokens[INCENTIVE_OWNER_B][poolTokenAddresses[i]] = true
            ```

    **Invariant 6:**

            ```
            forall i. 0 <= i < getPoolTokenAddressesLength() ==> poolTokenAdded[poolTokenAddresses[i]] = true
            ```

7.  **PendlePowerFarmLeverageLogic.sol**:

    1. `_flashAmount` should be greater than 0 when calling `_executeBalancerFlashLoan`.
    2. `_initialAmount` should be greater than or equal to 0 when calling `_executeBalancerFlashLoan`.
    3. `_lendingShares` should be greater than or equal to 0 when calling `_executeBalancerFlashLoan`.
    4. `_borrowShares` should be greater than or equal to 0 when calling `_executeBalancerFlashLoan`.
    5. `_allowedSpread` should be greater than or equal to 0 and less than or equal to `2 * PRECISION_FACTOR_E18` when calling `_executeBalancerFlashLoan`.
    6. The length of `_flashloanToken` and `_flashloanAmounts` should be the same in the `receiveFlashLoan` function.
    7. The length of `_feeAmounts` should be the same as the length of `_flashloanToken` and `_flashloanAmounts` in the `receiveFlashLoan` function.
    8. `msg.sender` should be equal to `BALANCER_ADDRESS` in the `receiveFlashLoan` function.
    9. `_totalDebtBalancer` should be greater than or equal to `_flashloanAmounts[0]` in the `receiveFlashLoan` function.
    10. If `_initialAmount` is greater than 0, the `_logicOpenPosition` function should be called in the `receiveFlashLoan` function.
    11. If `_initialAmount` is 0, the `_logicClosePosition` function should be called in the `receiveFlashLoan` function.
    12. In the `_logicClosePosition` function, `_paybackExactShares` should be called before `_withdrawPendleLPs`.
    13. In the `_logicClosePosition` function, `_withdrawPendleLPs` should be called before the Pendle router functions.
    14. In the `_logicClosePosition` function, the Pendle router functions should be called before the token swap functions.
    15. In the `_logicClosePosition` function, the token swap functions should be called before the closing route functions.
    16. In the `_logicOpenPosition` function, the Pendle SY deposit function should be called before the Pendle router functions.
    17. In the `_logicOpenPosition` function, the Pendle router functions should be called before the PendleChild deposit function.
    18. In the `_logicOpenPosition` function, the PendleChild deposit function should be called before the borrowing function.
    19. In the `_logicOpenPosition` function, the borrowing function should be called before the debt ratio check.
    20. In the `_coreLiquidation` function, the debt ratio check should be performed before the liquidation process.

8.  **PendlePowerFarmController.sol**:

    1. `pendleChildAddress` is a mapping that associates a `PendlePowerFarmToken` contract address to each `_pendleMarket` address. This mapping is initialized when a new `PendlePowerFarmToken` is deployed for a `_pendleMarket` using the `addPendleMarket` function.

    2. `pendleChildCompoundInfo` is a mapping that stores a `CompoundStruct` struct for each `_pendleMarket`. This struct contains information related to reward tokens, reserved amounts for compounding, and indexes for calculating reward amounts.

    3. `activePendleMarkets` is an array that stores the addresses of all active `_pendleMarket` contracts.

    4. `reservedPendleForLocking` is a variable that tracks the amount of PENDLE tokens reserved for locking in the `VE_PENDLE_CONTRACT_ADDRESS` contract.

    5. The `exchangeIncentive` variable represents the incentive factor for exchanging LP fees for PENDLE tokens.

    6. The `IS_ETH_MAIN` variable is a constant that indicates whether the contract is deployed on the Ethereum mainnet or not.

    7. The `PENDLE_LOCK`, `PENDLE_TOKEN_ADDRESS`, `PENDLE_VOTER`, `PENDLE_VOTE_REWARDS`, `VE_PENDLE_CONTRACT_ADDRESS`, and `ARB_REWARDS` variables represent references to other contracts or addresses used by the `PendlePowerFarmController` contract.

    8. The `_getRewardTokens` function returns an array of reward token addresses for a given `_pendleMarket`.

    9. The `_compareHashes` function is used to compare the current reward token addresses with a new set of reward token addresses for a given `_pendleMarket`.

    10. The `_setRewardTokens` function updates the reward token addresses for a given `_pendleMarket`.

    11. The `_getUserRewardIndex` function retrieves the current reward index for a given `_pendleMarket`, `_rewardToken`, and `_user` address.

    12. The `_checkFeed` function checks the price feed for a given token address, potentially reverting if the feed is not available.

    13. The `_getAmountToSend` function calculates the amount of tokens to send based on the current exchange rate between the token and ETH.

    14. The `_calcExpiry` function calculates the expiry timestamp for a given `_weeks` value.

    15. The `_getExpiry` function retrieves the current expiry timestamp for the PENDLE lock.

9.  **WiseCore.sol**:

    1. `positionLocked[_nftId]` should always be a boolean value (true or false) for any given `_nftId`.
    2. `userBorrowShares[_nftId][_poolToken]` should never be negative for any `_nftId` and `_poolToken`.
    3. `userLendingData[_nftId][_poolToken].shares` should never be negative for any `_nftId` and `_poolToken`.
    4. If `positionLendTokenData[_nftId]` contains `_poolToken`, then `userLendingData[_nftId][_poolToken].shares` should be greater than zero.
    5. If `positionBorrowTokenData[_nftId]` contains `_poolToken`, then `userBorrowShares[_nftId][_poolToken]` should be greater than zero.
    6. `globalPoolData[_poolToken].totalPool` should never be negative for any `_poolToken`.
    7. `lendingPoolData[_poolToken].pseudoTotalPool` should never be negative for any `_poolToken`.
    8. `lendingPoolData[_poolToken].totalDepositShares` should never be negative for any `_poolToken`.
    9. `lendingPoolData[_poolToken].totalBorrowShares` should never be negative for any `_poolToken`.
    10. `pureCollateralAmount[_nftId][_poolToken]` should never be negative for any `_nftId` and `_poolToken`.
    11. `globalPoolData[_poolToken].totalBareToken` should never be negative for any `_poolToken`.
    12. If `userLendingData[_nftId][_poolToken].unCollateralized` is true, then `pureCollateralAmount[_nftId][_poolToken]` should be zero.
    13. If `pureCollateralAmount[_nftId][_poolToken]` is greater than zero, then `userLendingData[_nftId][_poolToken].unCollateralized` should be false.

10. **AaveHub.sol**:

    1. **Token Approval Invariant**: For each underlying asset and its corresponding aToken, the contract should have approved the `WISE_LENDING` contract and the `AAVE_ADDRESS` for the maximum amount (`MAX_AMOUNT`). This ensures that the contract can interact with these contracts without needing additional approvals.

       ```solidity
       invariant tokenApprovalInvariant(address _underlyingAsset) {
          address aaveToken = aaveTokenAddress[_underlyingAsset];
          assert(IERC20(aaveToken).allowance(address(this), address(WISE_LENDING)) == MAX_AMOUNT);
          assert(IERC20(_underlyingAsset).allowance(address(this), AAVE_ADDRESS) == MAX_AMOUNT);
       }
       ```

    2. **Mapping Consistency Invariant**: The `aaveTokenAddress` mapping should only contain valid addresses, and for each underlying asset, there should be a corresponding aToken address.

       ```solidity
       invariant mappingConsistencyInvariant() {
          for (address _underlyingAsset : aaveTokenAddress.keys) {
             address aaveToken = aaveTokenAddress[_underlyingAsset];
             assert(aaveToken != address(0));
          }
       }
       ```

    3. **Owner Invariant**: Only the contract owner (identified by the `master` address) should be able to call the `setAaveTokenAddress` and `setAaveTokenAddressBulk` functions.

       ```solidity
       invariant ownerInvariant() {
          assert(msg.sender == master);
       }
       ```

    4. **Position Ownership Invariant**: For any position (identified by the `_nftId`), the caller of the deposit, withdraw, borrow, and payback functions should be the owner of that position.

       ```solidity
       invariant positionOwnershipInvariant(uint256 _nftId) {
          address owner = WISE_LENDING.getPositionOwner(_nftId);
          assert(owner == msg.sender);
       }
       ```

    5. **Valid Token Invariant**: The `_underlyingAsset` address passed to the deposit, withdraw, borrow, and payback functions should be a valid token address registered in the `aaveTokenAddress` mapping.

       ```solidity
       invariant validTokenInvariant(address _underlyingAsset) {
          assert(aaveTokenAddress[_underlyingAsset] != address(0));
       }
       ```

    6. **Locked Position Invariant**: For the `paybackExactAmount`, `paybackExactAmountETH`, and `paybackExactShares` functions, the position identified by `_nftId` should be in a locked state before executing the payback operations.

       ```solidity
       invariant lockedPositionInvariant(uint256 _nftId) {
          bool isLocked = WISE_LENDING.getPositionLocked(_nftId);
          assert(isLocked);
       }
       ```

11. **PositionNFTs.sol**:

    1. **Master Invariant**: Only the master address (identified by the `master` variable) should be able to call the `assignReserveRole`, `blockReservePublic`, and `forwardFeeManagerNFT` functions.

       ```solidity
       invariant masterInvariant() {
          assert(msg.sender == master);
       }
       ```

    2. **FeeManager Invariant**: The `feeManager` address should be either zero (not set) or a valid non-zero address.

       ```solidity
       invariant feeManagerInvariant() {
          assert(feeManager == address(0) || feeManager != address(0));
       }
       ```

    3. **FeeManager NFT Ownership Invariant**: If the `feeManager` address is set, the `FEE_MANAGER_NFT` token should be owned by the `feeManager` address.

       ```solidity
       invariant feeManagerNFTOwnershipInvariant() {
          if (feeManager != address(0)) {
             assert(ownerOf(FEE_MANAGER_NFT) == feeManager);
          }
       }
       ```

    4. **Reserved Position Invariant**: For each address in the `reserved` mapping, the associated value should be a valid token ID (greater than zero).

       ```solidity
       invariant reservedPositionInvariant() {
          for (address _user : reserved.keys) {
             uint256 reservedId = reserved[_user];
             assert(reservedId > 0);
          }
       }
       ```

    5. **Total Reserved Invariant**: The sum of all reserved positions (values in the `reserved` mapping) should be equal to the `totalReserved` variable.

       ```solidity
       invariant totalReservedInvariant() {
          uint256 sum = 0;
          for (address _user : reserved.keys) {
             if (reserved[_user] > 0) {
                   sum++;
             }
          }
          assert(sum == totalReserved);
       }
       ```

    6. **Token Existence Invariant**: For any token ID returned by the `walletOfOwner` function, the token should exist (i.e., the corresponding NFT has been minted).

       ```solidity
       invariant tokenExistenceInvariant(address _owner) {
          uint256[] memory tokenIds = walletOfOwner(_owner);
          for (uint256 i = 0; i < tokenIds.length; i++) {
             assert(_exists(tokenIds[i]));
          }
       }
       ```

    7. **Token Owner Invariant**: For any token ID returned by the `walletOfOwner` function, the owner of the token should be the specified `_owner` address.

       ```solidity
       invariant tokenOwnerInvariant(address _owner) {
          uint256[] memory tokenIds = walletOfOwner(_owner);
          for (uint256 i = 0; i < tokenIds.length; i++) {
             assert(ownerOf(tokenIds[i]) == _owner);
          }
       }
       ```

12. **PoolManager.sol**:

    1. **Pool Token Validity Invariant**: For each pool token address, the corresponding `timestampsPoolData` entry should have a non-zero timestamp value.

       ```solidity
       invariant poolTokenValidityInvariant(address _poolToken) {
          assert(timestampsPoolData[_poolToken].timeStamp != 0);
       }
       ```

    2. **Pool Creation Invariant**: If a pool token address is non-zero, it should have a valid entry in the `globalPoolData`, `borrowRatesData`, `borrowPoolData`, `algorithmData`, `lendingPoolData`, and `timestampsPoolData` mappings.

       ```solidity
       invariant poolCreationInvariant(address _poolToken) {
          if (_poolToken != address(0)) {
             assert(globalPoolData[_poolToken].totalPool >= 0);
             assert(borrowRatesData[_poolToken].startValuePole > 0);
             assert(borrowPoolData[_poolToken].allowBorrow == true || borrowPoolData[_poolToken].allowBorrow == false);
             assert(algorithmData[_poolToken].bestPole >= 0);
             assert(lendingPoolData[_poolToken].pseudoTotalPool >= 0);
             assert(timestampsPoolData[_poolToken].timeStamp > 0);
          }
       }
       ```

    3. **Borrow Rates Invariant**: For each pool token, the `borrowRatesData` entry should have valid values for `startValuePole`, `staticDeltaPole`, `staticMinPole`, and `staticMaxPole`.

       ```solidity
       invariant borrowRatesInvariant(address _poolToken) {
          BorrowRatesEntry memory rates = borrowRatesData[_poolToken];
          assert(rates.startValuePole > 0);
          assert(rates.staticDeltaPole > 0);
          assert(rates.staticMinPole > 0);
          assert(rates.staticMaxPole > rates.staticMinPole);
       }
       ```

    4. **Parameter Locking Invariant**: If the parameters for a pool token are locked (`parametersLocked[_poolToken] == true`), the corresponding `borrowRatesData` and `lendingPoolData` entries should not be modified.

       ```solidity
       invariant parameterLockingInvariant(address _poolToken) {
          if (parametersLocked[_poolToken]) {
             // Check that borrowRatesData and lendingPoolData remain unchanged
             // ...
          }
       }
       ```

    5. **Verified Isolation Pool Invariant**: For each verified isolation pool address, there should be a corresponding entry in the `verifiedIsolationPool` mapping with a value of `true`.

       ```solidity
       invariant verifiedIsolationPoolInvariant(address _isolationPool) {
          if (verifiedIsolationPool[_isolationPool]) {
             assert(verifiedIsolationPool[_isolationPool] == true);
          }
       }
       ```

    6. **Maximum Deposit Invariant**: For each pool token, the `maxDepositValueToken` entry should be greater than or equal to zero.

       ```solidity
       invariant maximumDepositInvariant(address _poolToken) {
          assert(maxDepositValueToken[_poolToken] >= 0);
       }
       ```

13. **PendlePowerFarmMathLogic.sol**:

    1. **Reentrancy Protection Invariant**: The `sendingProgress` flag should be set to `false` before entering any function that updates state variables.

       ```solidity
       invariant reentrancyProtectionInvariant() {
          assert(sendingProgress == false);
       }
       ```

    2. **Pool Update Invariant**: After executing the `_updatePools` function, the `sendingProgress` flag of the `WISE_LENDING`, `AAVE_HUB`, and the contract itself should be set to `false`.

       ```solidity
       invariant poolUpdateInvariant() {
          assert(sendingProgress == false);
          assert(WISE_LENDING.sendingProgress() == false);
          assert(AAVE_HUB.sendingProgressAaveHub() == false);
       }
       ```

    3. **Borrow Share Validity Invariant**: For any given NFT ID (`_nftId`), the borrow shares returned by `_getPositionBorrowShares` and `_getPositionBorrowSharesAave` should be non-negative.

       ```solidity
       invariant borrowShareValidityInvariant(uint256 _nftId) {
          assert(_getPositionBorrowShares(_nftId) >= 0);
          assert(_getPositionBorrowSharesAave(_nftId) >= 0);
       }
       ```

    4. **Lending Share Validity Invariant**: For any given NFT ID (`_nftId`), the lending shares returned by `_getPositionLendingShares` should be non-negative.

       ```solidity
       invariant lendingShareValidityInvariant(uint256 _nftId) {
          assert(_getPositionLendingShares(_nftId) >= 0);
       }
       ```

    5. **Collateral Token Amount Validity Invariant**: For any given NFT ID (`_nftId`), the collateral token amount returned by `_getPostionCollateralTokenAmount` should be non-negative.

       ```solidity
       invariant collateralTokenAmountValidityInvariant(uint256 _nftId) {
          assert(_getPostionCollateralTokenAmount(_nftId) >= 0);
       }
       ```

    6. **Debt Ratio Invariant**: For any given NFT ID (`_nftId`), the result of `_checkDebtRatio` should be `true`, indicating that the debt ratio is below 100%.

       ```solidity
       invariant debtRatioInvariant(uint256 _nftId) {
          assert(_checkDebtRatio(_nftId) == true);
       }
       ```

    7. **Minimum Deposit Amount Invariant**: For any given amount (`_amount`), if `_notBelowMinDepositAmount` returns `true`, then the equivalent ETH value of `_amount` should be greater than or equal to `minDepositEthAmount`.

       ```solidity
       invariant minimumDepositAmountInvariant(uint256 _amount) {
          if (_notBelowMinDepositAmount(_amount)) {
             assert(_getTokensInETH(ENTRY_ASSET, _amount) >= minDepositEthAmount);
          }
       }
       ```

## Approach Taken-in Evaluating Wise Lending Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Wise Lending.

    I start with the following contracts, which play crucial roles in the Wise Lending:

2.  **Documentation Review**:

    Then went to Review [this docs](https://wisesoft.gitbook.io/wise) for a more detailed and technical explanation of the Wise Lending.

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Codebase Quality

Overall, I consider the quality of the `Wise Lending` protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Architecture & Design**                | The protocol features a modular design, segregating functionality into distinct contracts (e.g., `WiseLending`, `PowerFarms`, `WrapperHub`, `TransferHub`) for clarity and ease of maintenance.                                                                                                                                                                                                                                                                                                                                                                                                                 |
| **Community Governance & Participation** | The protocol incorporates mechanisms for community governance, enabling token holders to influence decisions. This fosters a decentralized and participatory ecosystem, aligning with the broader ethos of blockchain development.                                                                                                                                                                                                                                                                                                                                                                              |
| **Error Handling & Input Validation**    | Functions check for conditions and validate inputs to prevent invalid operations, though the depth of validation (e.g., for edge cases transactions) would benefit from closer examination.                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **Code Maintainability and Reliability** | The contracts are written with emphasis on sustainability and simplicity. The functions are single-purpose with little branching and low cyclomatic complexity. The protocol includes a novel mechanism for collecting fees that offers an incentive for this work to be done by outside actors,thereby removing the associated complexity.                                                                                                                                                                                                                                                                     |
| **Code Comments**                        | The contracts are accompanied by comprehensive comments, facilitating an understanding of the functional logic and critical operations within the code. Functions are described purposefully, and complex sections are elucidated with comments to guide readers through the logic. Despite this, certain areas, particularly those involving intricate mechanics or tokenomics, could benefit from even more detailed commentary to ensure clarity and ease of understanding for developers new to the project or those auditing the code.                                                                     |
| **Testing**                              | The contracts exhibit a commendable level of test coverage, which is indicative of a robust testing regime. This coverage ensures that a wide array of functionalities and edge cases are tested, contributing to the reliability and security of the code. However, to further enhance the testing framework, the incorporation of fuzz testing and invariant testing is recommended. These testing methodologies can uncover deeper, systemic issues by simulating extreme conditions and verifying the invariants of the contract logic, thereby fortifying the codebase against unforeseen vulnerabilities. |
| **Code Structure and Formatting**        | The codebase benefits from a consistent structure and formatting, adhering to the stylistic conventions and best practices of Solidity programming. Logical grouping of functions and adherence to naming conventions contribute significantly to the readability and navigability of the code. While the current structure supports clarity, further modularization and separation of concerns could be achieved by breaking down complex contracts into smaller, more focused components. This approach would not only simplify individual contract logic but also facilitate easier updates and maintenance. |
| **Strengths**                            | The strength of for example WiseLending codebase lies in its comprehensive approach to decentralized lending. It features a unique Lending Automated Scaling Algorithm (LASA) to dynamically adjust borrow rates, supports various interaction modes including direct ETH deposit, Aave pools, special curve pools, and NFT positions, and implements robust security checks and balances. The code is well-structured with clear comments and modular functions, enhancing its readability and maintainability.                                                                                                |
| **Documentation**                        | The NatSpec is mostly complete for all external functions,and there are helpful inline comments throughout. However, there currently is no external documentation for users or integrators. Additionally, some user-facing documentation does not identify the risks and nuances of the staking contract, which is important. Technical developer documentation would also be helpful for integrators or MEV searchers interested in collecting fees.                                                                                                                                                           |

## Architecturee

### **System Workflow**

The System Workflow for Wise Lending protocol involves several contracts that work together to provide a decentralized lending platform. Here's a high-level overview of the workflow:

1. **User Interaction**: Users can interact with the platform by depositing, borrowing, repaying, and withdrawing funds. They can also liquidate loans that exceed a certain debt ratio.

2. **WiseLending.sol**: This is the main contract that manages these interactions. It calculates variable borrow rates based on the utilization of the pool and uses a family of bounding curves to adjust the share price of lending and borrow shares over time.

3. **MainHelper.sol**: This contract assists in managing lending and borrowing operations within the Wise protocol. It facilitates calculations for converting token amounts into lending or borrowing shares, ensures security checks, and updates pool utilization and pseudo amounts while interacting with external protocols such as Aave and Curve.

4. **PositionNFTs.sol**: This contract manages the creation and transfer of NFTs that represent user positions. These NFTs allow users to store their collateral and borrows, enabling them to trade their positions or use them in other contracts.

5. **WiseSecurity.sol**: This contract ensures the security and integrity of the WiseLending protocol by enforcing various checks, calculations, and safeguards related to lending and borrowing positions. It also provides mechanisms for handling emergencies and mitigating risks associated with certain tokens.

6. **PoolManager.sol**: This contract facilitates the creation and management of lending pools with adjustable parameters, such as borrowing capabilities, collateral factors, and maximum deposit amounts.

7. **OwnableMaster.sol**: This contract defines an ownable master pattern with a proposal mechanism. It allows the current master to propose a new master, who can then claim ownership to become the new master. The master can also renounce ownership, effectively removing the master role.

8. **Aave Pools**: The protocol integrates with Aave, a decentralized lending and borrowing platform, to allow users to earn supply APY on their deposited funds that are not borrowed. This increases the capital efficiency of the platform and provides additional rewards to users.

9. **PowerFarm Pools**: The protocol also supports the use of special curve pools as collateral, expanding the range of assets that users can deposit and borrow. PowerFarm pools are a type of automated market maker (AMM) that uses a unique bonding curve to determine asset prices.

10. **Solely Deposit and Withdraw**: Users can deposit and withdraw funds privately, keeping their balances hidden. This feature is enabled by the use of Position NFTs, which represent user positions and can be traded or used in other contracts.

11. **Lending and Borrowing**: The protocol allows users to deposit and borrow a variety of cryptocurrencies, including ETH and ERC-20 tokens. Deposits can be collateralized to secure borrowed funds, reducing the risk for the lender. Borrow rates are variable and determined by the utilization of the pool, adjusting automatically through the LASA algorithm.

12. **Liquidation**: The protocol includes a mechanism for liquidating loans that exceed a certain debt ratio. Liquidators can choose the tokens to use for payback and receive, and they are incentivized to liquidate undercollateralized loans by receiving a portion of the collateral.

13. **Health Checks**: The protocol includes health checks to ensure the safety of user positions. These checks are performed after state changes to ensure that users remain secure.

14. **Curve Security Checks**: The protocol monitors the bonding curves used to determine asset prices to prevent excessive share price fluctuations. This helps to maintain the stability of the platform and protect users from sudden changes in asset prices.

15. **Isolation Pools**: In addition to the main lending pool, the protocol allows users to create isolated pools to manage their assets separately from other users. Isolation pools are designed to protect users from the risk of other users' positions being liquidated.

16. **Fee Management**: The protocol collects fees on all transactions and distributes them to fee managers. Fees are used to incentivize liquidity providers and maintain the stability of the platform.

17. **Flash Loans**: The protocol supports flash loans, which allow users to borrow assets without providing collateral as long as the loan is repaid within the same transaction. Flash loans are useful for arbitrage opportunities and other short-term trading strategies.

18. **Margin Trading**: The protocol also supports margin trading, which allows users to borrow assets to increase their exposure to a particular asset. Margin trading can be used to amplify profits, but it also increases the risk of liquidation.

19. **Yield Farming**: Users can earn rewards by depositing assets into lending pools and participating in yield farming. Yield farming is a way to earn passive income by providing liquidity to the platform.

20. **Security Measures**: The protocol includes a variety of security measures to protect users from fraud and loss. These measures include access controls, reentrancy guards, input validation, and emergency shutdown mechanisms.

21. **Modular Architecture**: The protocol is designed to be modular and extensible, with helper contracts and abstract contracts that can be reused in other contracts within the WiseLending ecosystem. This allows for easy integration with other decentralized platforms and services.

22. **Position NFTs**: The protocol uses NFTs (Non-Fungible Tokens) to represent user positions within the lending pools. These NFTs grant access to protocol functionalities and provide additional features such as transparency, control, and role-based access control. Users can mint NFTs to represent their positions and manage the ownership of those NFTs.

23. **LASA Algorithm**: The protocol uses a unique algorithm called the Liquidity Adjusted Supply Algorithm (LASA) to adjust the resonance factor of lending pools in order to maximize total deposit shares. The algorithm is designed to balance the needs of borrowers and lenders and maintain the stability of the platform.

24. **Blacklisting Tokens**: The protocol allows the master address to blacklist tokens, which prevents them from being used as collateral or borrowed. This feature can be used to mitigate risks associated with certain tokens.

**Overall Workflow**

The WiseLending protocol, a comprehensive decentralized lending platform, incorporates a modular architecture for seamless integration with other platforms, employs security measures for user protection, and utilizes LASA (Liquidity Adjusted Supply Algorithm) for optimal lending pool resonance adjustment, supporting features like flash loans, margin trading, and yield farming.

| File Name                          | Core Functionality                                                                                                                                                                                                                                                                      | Technical Characteristics                                                                                                                                                                                                                                                                                                                                                                                                   | Importance and Management                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `WiseLending.sol`                  | The core functionality of the `WiseLending` contract is to enable users to collateralize and borrow assets with variable borrow rates determined by the LASA algorithm.                                                                                                                 | The technical characteristics of the contract include the use of `ERC20` tokens for asset representation and `NFTs` for position representation, allowing users to keep their funds private and enabling them to withdraw even when the pools are borrowed empty. The contract also supports interactions with other platforms such as `Aave` and `Beefy Farms` for maximal capital efficiency and new usage possibilities. | The `importance` of the WiseLending contract lies in providing a secure, efficient, and reliable lending platform for users to `collateralize` and `borrow` assets without intermediaries. This can help to increase financial inclusion, reduce transaction costs, and improve access to credit.The management of the `WiseLending` contract involves regular updates, `monitoring`, and auditing to ensure the security and reliability of the platform. This includes testing for potential vulnerabilities, optimizing the code for efficiency, and ensuring compliance with regulatory requirements. Proper contract management is essential to maintain user trust and confidence in the platform and to prevent potential losses or hacks. |
| `WiseSecurity.sol`                 | The core functionality of the `WiseSecurity` contract is to perform security checks for withdraws, borrows, paybacks, and liquidations in the wiseLending platform, as well as providing read-only functions for UI data                                                                | Technical characteristics include the use inheritance from `WiseSecurityHelper` and `ApprovalHelper` contracts, and the implementation of modifiers and error handling for function calls                                                                                                                                                                                                                                   | Importance lies in ensuring the security and stability of the `wiseLending` platform by restricting certain actions and enforcing conditions; and its management is done through the use of the `onlyWiseLending` modifier, which restricts function calls to only the `WISE_LENDING` address, as well as the `onlyMaster` modifier, which restricts certain function calls to only the contract owner.                                                                                                                                                                                                                                                                                                                                           |
| `MainHelper.sol`                   | Manages lending and borrowing operations for various tokens in isolation pools.                                                                                                                                                                                                         | Uses pseudo amounts to track token balances and share prices accurately.Implements a scaling algorithm (LASA) to optimize total deposit shares in lending pools.Employs smooth functions to calculate borrow rates based on pool utilization.                                                                                                                                                                               | Enables efficient and secure lending and borrowing of crypto assets.Provides a mechanism for managing `isolation pools`, where users can trade specific tokens without affecting other pools.Optimizes pool performance by adjusting borrow rates and scaling total `deposit` shares. Internal helper functions are used to perform various operations, such as `calculating shares`, `updating pseudo amounts`, and managing pool data.Security checks are implemented to prevent invalid actions and ensure the integrity of the contract.The contract includes functions for cleaning up and preparing pools for use.                                                                                                                          |
| `WiseSecurityHelper.sol`           | Provides read functions to calculate and return `weighted` and `unweighted` total collateral, overall borrow amount, and updated borrow and collateral amounts.                                                                                                                         | Uses internal helper functions and external contract calls to access data from the underlying lending protocol.Implements checks to ensure `valid pool conditions`, prevent bad debt, and limit liquidation share amounts.                                                                                                                                                                                                  | Provides the underlying calculations and checks necessary for secure and efficient lending and borrowing operations.Includes functions to lock or unlock all pools for borrow and deposit actions, and to check for blacklisted tokens.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `PendlePowerFarmToken.sol`         | Core Functionality: The `PendlePowerFarmToken` contract facilitates the creation and management of LP tokens for farming, synchronizing supplies, and managing minting and burning processes.                                                                                           | Technical Characteristics: It integrates with Pendle's ecosystem through various interfaces, manages LP assets, enforces minting fees, handles reward distributions, and ensures synchronization of supply.                                                                                                                                                                                                                 | Importance and Management: This contract is crucial for efficient yield farming operations, providing flexibility in LP asset management, controlling minting fees, and `maintaining` accurate supply synchronization, all managed by the PendlePowerFarmController.                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `FeeManager.sol`                   | This contract `orchestrates` fee distribution from `wiseLending`, implements incentive structures for ecosystem bootstrapping, tracks bad debt of each position, and facilitates fee claims and payments, pivotal for sustaining the WISE ecosystem.                                    | The `FeeManager.sol` offers functions for adjusting pool fees, proposing and claiming incentive masters, managing bad debt, and facilitating fee claims and payments, leveraging blockchain technology for transparency and automation.                                                                                                                                                                                     | Essential for the smooth operation of the `WISE` ecosystem, the contract efficiently manages fees, incentives, and bad debt, fostering ecosystem growth and sustainability. Managed by designated roles like the master, `incentive masters`, and `fee beneficiaries`, it allows controlled adjustments and operations.                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `PendlePowerFarmLeverageLogic.sol` | This contract provides leverage functionality for Pendle Power Farm users, allowing them to execute balancer flash loans, manage positions, and handle liquidations efficiently.                                                                                                        | It integrates with Pendle Power Farm Math Logic and inherits from `IFlashLoanRecipient`, featuring functions for executing flash loans, handling flash loan receipts, opening and closing positions, and managing liquidations.                                                                                                                                                                                             | Crucial for users engaging in leveraged trading within the `Pendle Power Farm` ecosystem, this contract ensures efficient handling of flash loans, position management, and liquidations, contributing to the overall stability and functionality of the platform. Managed through defined contract functions and logic, it maintains integrity and reliability in leveraging operations.                                                                                                                                                                                                                                                                                                                                                         |
| `OracleHelper.sol`                 | This contract provides functionality for `managing price feeds` and `oracles`, including adding new price feeds, validating oracle responses, retrieving `TWAP prices`, recalibrating heartbeat values, and checking chainlink feed status.                                             | It abstracts functionalities from Declarations and includes functions for adding oracles, aggregators, and TWAP pool information, as well as validating, comparing, and recalibrating price feed data.                                                                                                                                                                                                                      | Crucial for ensuring `accurate pricing data` within the protocol, this contract handles the integration and management of various oracles and price feeds, contributing to the reliability and integrity of the system. Managed through defined contract functions and logic, it maintains consistency and accuracy in price data management.                                                                                                                                                                                                                                                                                                                                                                                                     |
| `PendlePowerFarmController.sol`    | This contract manages `Pendle Power Farm` markets, facilitating LP withdrawals, exchange of LP fees for Pendle with incentives, `skimming`, `adding new Pendle markets`, `updating reward tokens`, `managing locking and claiming of Pendle tokens`, and facilitating voting.           | It inherits functionality from `PendlePowerFarmControllerHelper`, integrates with `PendlePowerFarmTokenFactory`, and includes functions for LP management, fee exchange, locking, claiming, voting, and interfacing with other contracts.                                                                                                                                                                                   | Crucial for the operation of the `Pendle Power Farm` protocol, this contract orchestrates LP interactions, fee exchanges, reward claiming, and governance functions, ensuring the smooth functioning and integrity of the protocol, managed through defined contract functions and access control mechanisms.                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `WiseCore.sol`                     | This `abstract` contract serves as the core logic for `handling deposits`, `withdrawals`, `borrowing`, and `liquidations` within the Wise protocol, integrating with helper contracts and providing essential functions for interacting with lending pools and managing user positions. | It combines `deposit`, `withdrawal`, `borrow`, and `liquidation` logic with security checks, utilizes helper functions for pool preparations, and interfaces with other contracts for token transfers and oracle queries.                                                                                                                                                                                                   | Essential for the operation of the `Wise protocol`, this contract facilitates the safe and efficient handling of user funds, `collateral`, and `borrowing` activities, ensuring protocol integrity and security through well-defined core functions and security measures.                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `WiseOracleHub.sol`                | This contract, `WiseOracleHub`, acts as an `on-chain` extension for price feeds, allowing users to obtain the current ETH value of a token by simply knowing its address, while also providing heartbeat checks for token update intervals.                                             | It utilizes inheritance from `OracleHelper`, interacts with `Chainlink` oracles, `Uniswap` pools, and price feeds, and enables addition of new token `address-to-price` feed mappings and TWAP oracles.                                                                                                                                                                                                                     | Crucial for the `Wise protocol`, it ensures accurate `token pricing`, facilitates oracle additions through a `time-locked` master contract, and performs heartbeat checks to maintain data reliability, contributing to the protocol's overall security and stability.                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `AaveHub.sol`                      | This contract, `AaveHub`, optimizes capital efficiency by utilizing Aave pools, depositing non-borrowed funds into corresponding Aave pools to earn supply APY, with accounting managed by position NFTs.                                                                               | It imports helper contracts for Aave interaction, transfer, and approval, enables setting `Aave` token addresses, facilitates `deposits`, `withdrawals`, `borrowing`, and `repayments` of ERC20 tokens and ETH, and provides functions for rate calculation.                                                                                                                                                                | Essential for efficient capital deployment within the `Wise protocol`, it enhances yield generation by leveraging Aave's `lending infrastructure`, managed by the master address with functions ensuring proper asset linkage and transaction execution, contributing to the protocol's financial robustness.                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `WiseLowLevelHelper.sol`           | This abstract contract, `WiseLowLevelHelper`, provides essential internal functions and views for managing `low-level` operations within the Wise protocol, including data manipulation, parameter validation, and permission control.                                                  | It defines `modifiers` and internal functions for managing parameters, accessing pool data, `setting values`, and performing various internal operations critical for protocol integrity.                                                                                                                                                                                                                                   | Integral to the Wise protocol's infrastructure, it ensures secure and efficient handling of core operations such as `pool management`, `parameter validation`, and permission control, primarily managed by the protocol's fee manager for maintaining protocol stability and security.                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `PositionNFTs.sol`                 | The `PositionNFTs` contract, built on `ERC721Enumerable` and `OwnableMaster`, manages the creation, reservation, and ownership of `NFTs` crucial for accessing and utilizing the WiseLending protocol.                                                                                  | It incorporates features such as NFT minting, ownership verification, `role-based` access control, metadata URI generation, and dynamic base URI configuration for metadata retrieval.                                                                                                                                                                                                                                      | Vital for the `WiseLending` protocol, it enables users to obtain NFTs required for protocol participation, with reserved and public minting functionalities managed securely through role-based access control and ownership validation, overseen by the contract owner.                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `PoolManager.sol`                  | The `PoolManager` contract, extending from WiseCore, facilitates the creation and management of lending pools, including setting parameters such as `interest rates`, `collateral factors`, and `maximum deposit amounts`.                                                              | It employs structs to organize pool creation parameters and curve pool settings, utilizes mathematical functions for algorithmic calculations, and integrates access control mechanisms for parameter adjustments.                                                                                                                                                                                                          | Crucial for the functioning of the Wise protocol, it enables the creation of lending pools with customizable parameters while ensuring secure and efficient management through access control and algorithmic rate adjustments.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `PendlePowerFarmMathLogic.sol`     | The `PendlePowerFarmMathLogic` contract, extending from `PendlePowerFarmDeclarations`, provides mathematical logic for managing leverage positions, calculating borrow and lending amounts, and approximating net APYs for positions.                                                   | It utilizes modifiers for pool updates, internal functions for mathematical computations, and interfaces to interact with the Pendle lending protocol and oracles.                                                                                                                                                                                                                                                          | Crucial for the `Pendle Power Farm` protocol, it ensures accurate calculation of borrow and lending amounts, facilitates leverage management, and contributes to maintaining position health and profitability while preventing reentrancy attacks.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

## Systemic Risks, Centralization Risks, Technical Risks & Integration Risks

1.  **WiseLending.sol**

    - **Systemic Risks:**

      1. `Liquidity Risks`: The contract allows users to borrow tokens against their collateralized assets. If the value of the collateralized assets falls significantly, it could lead to a situation where the value of the collateral is no longer sufficient to cover the borrowed amount, leading to a potential systemic risk.
      2. `Contract Risks`: The contract relies on the correct functioning of various other contracts, such as the `PoolManager` contract. If there are any bugs or vulnerabilities in these contracts, it could potentially impact the functioning of this contract.
      3. `Centralized oracle risk`: The contract relies on a centralized oracle hub for market data like prices. This poses a centralization risk if the oracle hub goes down or is compromised.

    - **Centralization Risks:**

      1. `Contract Owner`: The contract has a contract owner who has the ability to upgrade the contract. This centralization of control could pose a risk if the contract owner is compromised or acts maliciously.
      2. `Oracle Dependency`: The contract depends on an oracle (WiseOracleHubAddress) for price feeds. If the oracle is compromised or provides incorrect price feeds, it could lead to potential losses for users.

    - **Technical Risks:**

      1. `Complexity`: The contract is quite complex, with multiple functions and dependencies on other contracts. This complexity could increase the risk of bugs and vulnerabilities.
      2. `Security Checks`: The contract has multiple security checks and validations throughout the code. While this is a good practice, it also increases the complexity of the contract and the risk of errors.
      3. The contract uses the `syncPool` modifier for several functions, which could potentially lead to state inconsistencies if the pool data is not properly updated or if the `LASA` algorithm has any issues.

    - **Integration Risks:**

      1. The contract integrates with other contracts such as PoolManager and WiseOracleHubAddress. Any issues with these contracts could impact the functioning of this contract.

      2. The contract interacts with various tokens (e.g., WETH, ERC20 tokens) and other contracts (e.g., Aave pools, curve pools). If these tokens or contracts have any vulnerabilities or if they are not properly integrated with the WiseLending contract, it could lead to issues such as token loss or unintended behavior.

      3. The contract relies on the WISE Oracle Hub for price data. If the oracle has any issues or is compromised, it could lead to incorrect price data being used by the contract, potentially resulting in financial losses.

      4. The contract uses an NFT contract to represent user positions. If the NFT contract has any vulnerabilities or if it is not properly integrated with the WiseLending contract, it could lead to issues with user positions or token loss.

2.  **WiseSecurity.sol**

    - **Systemic Risks**:

      1.  The `securityShutdown` function allows a designated `securityWorker` address to shut down all active pools, which could potentially disrupt the entire system if misused or compromised.
      2.  The `setBlacklistToken` function allows the `master` address to blacklist tokens, preventing them from being borrowed or used as collateral. This could potentially cause disruptions if important tokens are blacklisted.

    - **Centralization Risks**:

      1.  The contract heavily relies on the `master` address, which has the authority to set critical parameters such as liquidation settings, blacklist tokens, and revoke security shutdowns. This centralization of power poses a risk if the `master` address is compromised or abused.
      2.  The `WISE_LENDING` contract has significant control over various aspects of the system, such as verifying isolation pools, locking positions, and managing lending and borrowing data. Centralization of these functions in a single contract could introduce risks.

    - **Technical Risks**:

      1. The use of external contracts, such as `WiseSecurityHelper`, `ApprovalHelper`, and various other interfaces (`ICurve`, `WISE_ORACLE`, `FEE_MANAGER`), introduces dependencies and potential vulnerabilities if these contracts are not properly audited or maintained.
      2. The presence of several functions that perform critical operations, such as `checksLiquidation`, `checksWithdraw`, `checksBorrow`, and `checksCollateralizeDeposit`, increases the attack surface and the risk of potential vulnerabilities.
      3. The use of inline assembly (e.g., `success` in the `curveSecurityCheck` function) could introduce potential vulnerabilities if not implemented correctly.

    - **Integration Risks**:

      1. The contract heavily relies on external data sources, such as the `WISE_ORACLE` for token price data and the `FEE_MANAGER` for fee management. If these external sources are compromised or provide incorrect data, it could lead to issues within the system.
      2. The integration with various lending and borrowing pools, as well as the `CurveSwap` protocol, increases the complexity of the system and the potential for integration issues or vulnerabilities.

3.  **MainHelper.sol**

    - **Systemic Risks**:

      1.  **Algorithmic Complexity**: The contract includes an algorithmic mechanism for adjusting parameters over time. Any flaws in this mechanism could lead to unintended consequences, affecting the stability and functionality of the system.

    - **Centralization Risks**:

      1.  **Control of Critical Functions**: The contract contains functions such as `_updateUtilization`, `_updatePseudoTotalAmounts`, `_scalingAlgorithm`, etc., which could potentially be controlled by a centralized entity or admin. This centralized control poses risks of manipulation or misuse.
      2.  **Data Centralization**: The contract maintains various data mappings and structures that could potentially centralize control over critical parameters and operations, leading to single points of failure or control.

    - **Technical Risks**:

      1.  **Efficiency and Gas Costs**: The contract's complex algorithms and calculations may lead to high gas costs, limiting usability and scalability, particularly for users interacting with the contract on the Ethereum blockchain.
      2.  **Complexity and Maintenance**: The contract's complexity could make maintenance and debugging challenging, potentially leading to errors or vulnerabilities over time.

    - **Integration Risks**:

      1.  **External Integrations**: The contract integrates with external tokens (`IERC20`), NFT contracts (`POSITION_NFT`), and potentially other contracts not explicitly mentioned. Changes in the behavior or vulnerabilities in these external contracts could affect the functionality of this contract.
      2.  **Interoperability**: The contract's compatibility with other smart contracts and protocols may pose risks if not thoroughly tested and integrated, potentially leading to unexpected behavior or conflicts.

4.  **WiseSecurityHelper.sol**

    - **Systemic Risks:**

      1.  The contract relies heavily on external data from `Chainlink oracles` for token prices. If the oracles become unavailable or unreliable, the contract could make incorrect calculations.
      2.  The contract relies on the accuracy of the collateral factors set by the `pool managers`. If these factors are incorrect, the contract could over- or under-collateralize positions, leading to losses for users.

    - **Centralization Risks:**

      1.  The contract gives significant power to the pool managers, who can set collateral factors and blacklist tokens. This could allow them to manipulate the system to their advantage or the detriment of users.
      2.  The contract relies on a single entity (`FEE_MANAGER`) to manage the pool token addresses. If this entity is compromised, it could add malicious tokens to the pool, which could be used to exploit users.

    - **Technical Risks:**

      1. The contract uses complex calculations to determine collateral and borrow amounts. If these calculations are incorrect, it could lead to losses for users.

      2. The contract relies on external contracts (`POSITION_NFTS`, `WISE_LENDING`, `FEE_MANAGER`) to function properly. If these contracts are compromised or malfunction, the contract could fail.

      3. Precision Errors: The contract performs arithmetic calculations involving division and multiplication (overallETHCollateralsBoth, overallETHCollateralsWeighted, etc.), which could lead to precision errors if not handled carefully, especially considering the use of fixed-point arithmetic (PRECISION_FACTOR_E18).

    - **Integration Risks:**

      1. The contract interacts with several external systems (`Chainlink oracles`, `POSITION_NFTS`, `WISE_LENDING`, `FEE_MANAGER`). If these systems are not properly integrated, it could lead to errors or security vulnerabilities.

      2. The contract assumes that the external contracts it interacts with are secure and reliable. If these contracts are compromised or malfunction, it could impact the security of the contract.

5.  **PendlePowerFarmToken.sol**

    - **Systemic Risks**:

      1. **MarketExpired Error**: The contract includes an error called `MarketExpired`, which suggests that there might be a risk associated with markets expiring. If a market expires unexpectedly or prematurely, it could affect the functioning of the contract and potentially lead to loss of funds or disruptions in operations.

    - **Centralization Risks**:

      1. **Controller Dependency**: The contract heavily relies on a single controller (`PENDLE_POWER_FARM_CONTROLLER`) for various critical operations. Depending on a centralized controller can introduce risks related to control, censorship, and single points of failure. If the controller is compromised or malfunctions, it could disrupt the entire system.
      2. **Exclusive Mint Fee Change**: The ability to change the mint fee is restricted to the controller (`onlyController` modifier), which implies centralized control over fee adjustments. This centralization could lead to concerns regarding fairness and transparency in fee management.

    - **Technical Risks**:

      1. **Arithmetic Operations**: The contract involves various arithmetic operations, including multiplication and division, which are susceptible to overflow, underflow, and precision errors if not implemented correctly. Such errors could result in unexpected behavior and financial losses.
      2. **Time-Dependent Logic**: The contract relies on block timestamps for time-dependent logic, such as calculating rewards and determining share prices. Depending on block timestamps for critical operations can introduce risks related to manipulation and inaccurate calculations due to network delays or irregularities.

    - **Integration Risks**:

      1. **Initialization Process**: The contract includes an initialization function (`initialize`) that sets up critical parameters and dependencies. The correctness and completeness of the initialization process are crucial for the proper functioning of the contract. Any errors or oversights during initialization could lead to system instability and vulnerabilities.

      2. The contract relies on several external contracts, including:Pendle Market,Pendle Controller,Pendle Sy.

6.  **FeeManager.sol**

    - **Systemic Risks:**

      1.  **Fee Accumulation and Distribution:** The contract is responsible for accumulating and distributing fees from multiple pools. If there are errors or vulnerabilities in the fee calculation or distribution mechanisms, it could lead to systemic risks affecting all users and pools.
      2.  **Incentive Structures:** The contract implements incentive structures (incentiveOwnerA, incentiveOwnerB) that involve increasing incentive amounts. Poorly designed or manipulated incentive structures could incentivize unwanted behavior or lead to economic imbalances within the ecosystem.

      3.  The master role has broad administrative privileges, including the ability to set fees, configure Aave pool mappings, and manage beneficial addresses. Compromising the master role could lead to systemic risks.

    - **Centralization Risks:**

      1.  The `master` role has a high degree of centralized control over critical functions, such as setting pool fees, managing Aave pool mappings, and managing beneficial addresses.
      2.  The `incentiveMaster` role has centralized control over the incentive structures, including proposing a new `incentiveMaster`, increasing incentives for incentive owners, and managing bad debt incentives.
      3.  The `incentiveOwnerA` and `incentiveOwnerB` roles have centralized control over their respective incentive mappings.

    - **Technical Risks:**

      1.  **Unchecked Loops:** Several functions use unchecked loops, which could pose a risk of exceeding gas limits or causing unexpected behavior if the loop conditions are not properly managed.
      2.  **Revert Conditions:** Revert conditions are used in various functions to handle exceptional cases, but the correctness and completeness of these conditions need to be carefully reviewed to ensure proper error handling.
      3.  **External Calls:** The contract interacts with external contracts (e.g., AAVE) to withdraw tokens. Malfunctioning or malicious behavior in these external contracts could lead to technical risks for `FeeManager`.

    - **Integration Risks:**

      1.  **Integration with External Systems:** The contract integrates with external systems such as AAVE and WISE lending. Integration risks may arise from discrepancies or inconsistencies between these external systems and the `FeeManager` contract.
      2.  **Pool Token Management:** The contract manages a list of pool tokens, adding and removing them manually. Integration risks may arise if the manual management process is prone to errors or inconsistencies.
      3.  **Claiming Fees:** The contract allows users to claim fees and incentives, which require proper integration with user interfaces or external systems. Integration risks may arise if the claiming process is not properly coordinated or integrated.

7.  **PendlePowerFarmLeverageLogic.sol**
    Here are the potential risks identified in the provided smart contract:

    - **Systemic Risks**:

      1.  `Flash loan risk`: The contract relies on flash loans from Balancer, which could fail or be subject to malicious attacks.

      2.  The contract interacts with external contracts, such as `BALANCER_VAULT`, `AAVE_HUB`, `WISE_LENDING`, `PENDLE_ROUTER`, and others. Depending on the reliability and security of these contracts, there may be systemic risks associated with their operation. Any vulnerability or exploit in these contracts could affect the functionality and security of this contract.

      3.  `Market risk`: The value of the underlying assets (ETH, ENTRY_ASSET) could fluctuate, potentially leading to losses for users.

      4.  `Liquidity risk`: The liquidity of the underlying assets could be limited, making it difficult to close positions or liquidate collateral.

    - **Centralization Risks**:

      1. `Dependence on Balancer`: The contract relies on Balancer for flash loans, which could centralize control over the system.

      2. `Dependence on Pendle`: The contract relies on Pendle for lending and borrowing, which could centralize control over the system.

      3. `Dependence on Uniswap V3`: The contract relies on Uniswap V3 for token swaps, which could centralize control over the system.

      4. `Dependence on Curve`: The contract relies on Curve for token swaps, which could centralize control over the system.

    - **Technical Risks**:

      1.  The contract involves complex financial logic, flash loans, token swaps, liquidity provisioning, and borrowing. Any error or oversight in the implementation of these functionalities could lead to technical risks, such as unexpected behavior, loss of funds, or vulnerabilities to exploits.

    - **Integration Risks**:

      1.  The contract integrates with multiple external protocols and services, such as Balancer, Aave, Uniswap V3, Pendle, and others. Integration risks arise from potential issues in communication, compatibility, or dependency on external systems. Changes in the external APIs or protocols may lead to integration issues and affect the contract's functionality.

      2.  Compatibility: The contract may not be compatible with future versions of Balancer, Pendle, Uniswap V3, or Curve.

8.  **OracleHelper.sol**

    - **Systemic Risks**:

      1.  The contract relies heavily on external contracts and data sources, such as Uniswap V3 Factory, Chainlink price feeds, and Uniswap V3 pools. Any issues or vulnerabilities in these external dependencies could impact the functionality and security of this contract.
      2.  The contract assumes that the Chainlink price feeds are reliable and up-to-date. If the price feeds become stale or manipulated, it could lead to incorrect pricing and potential financial losses.

    - **Centralization Risks**:

      1.  The contract relies on a centralized Chainlink oracle system for price data. While Chainlink aims to decentralize oracle services, there is still a degree of centralization in their architecture, which could pose a risk if the system is compromised or experiences downtime.

    - **Technical Risks**:

      1.  There are several potential sources of technical risks, such as:

          - Incorrect implementation of arithmetic operations or type conversions, which could lead to integer overflows or underflows.
          - Incorrect handling of edge cases, such as zero addresses or non-existent pools.
          - Potential reentrancy vulnerabilities if external contracts are called before updating state variables.
          - Potential gas limitations or out-of-gas situations due to the complexity of the contract and the external calls made.

    - **Integration Risks**:

      1.  The contract makes several external calls to other contracts, such as Uniswap V3 Factory, Chainlink price feeds, and Uniswap V3 pools. Any changes or upgrades to these external contracts could potentially break the integration and functionality of this contract.
      2.  The contract assumes specific interfaces and functions from the external contracts, which could change in future versions, leading to integration issues.

9.  **PendlePowerFarmController.sol**

    - **Systemic Risks:**

      1.  The contract relies heavily on the `vePendle` contract and other external contracts like `PendleVoter`, `PendleVoteRewards`, and `PendleArbitrumRewards`. If any of these external contracts have vulnerabilities or issues, it could impact the functionality and security of this contract.
      2.  The contract interacts with multiple external tokens and reward tokens. Any issues or vulnerabilities in these external contracts could potentially affect this contract.

    - **Centralization Risks:**

      1. The contract has an `onlyMaster` role, which gives a single address (the `master`) a significant amount of control over critical functions like adding new Pendle markets, changing fees, locking Pendle tokens, voting, and forwarding ETH.
      2. The contract is deployed by the `master` address, which creates and initializes the `PendlePowerFarmTokenFactory` contract.

    - **Technical Risks:**

      1.  The contract contains complex logic and multiple external interactions, increasing the attack surface and potential for bugs or vulnerabilities.
      2.  There are multiple external calls to transfer tokens and potentially send ETH, which could be susceptible to reentrancy attacks if not properly handled.
      3.  The contract uses the `transfer` function to send ETH, which can fail silently if the recipient contract's fallback function fails.
      4.  There are several instances of unchecked arithmetic operations, which could potentially lead to integer overflows or underflows if the inputs are not properly validated.

    - **Integration Risks:**

      1.  The contract integrates with multiple external contracts and tokens, increasing the complexity of the system and the potential for compatibility issues or breaking changes in the future.
      2.  The contract relies on external oracles (through the `wiseOracleHub`) for price data, which could be a potential point of failure or manipulation if the oracles are compromised or provide incorrect data.
      3.  The contract interacts with the Arbitrum network for claiming rewards, which introduces additional complexity and potential risks related to cross-chain interactions.

10. **WiseCore.sol**

    - **Systemic Risks:**

      1.  The contract interacts with external contracts like `WISE_ORACLE`, `WISE_SECURITY`, `POSITION_NFT`, and `AAVE_HUB_ADDRESS`. If any of these external contracts have vulnerabilities or undergo unexpected changes, it could impact the functionality of the contract.

      2.  The contract relies heavily on the `MainHelper` and `TransferHelper` contracts, which are not provided in the code snippet. Any vulnerabilities or issues in these external dependencies could introduce systemic risks.

    - **Centralization Risks:**

      1.  The contract has several access control mechanisms, such as the `_byPassCase` function and the `AAVE_HUB_ADDRESS` constant. If these access control mechanisms are compromised or misused, it could lead to centralization risks.

      2.  The contract interacts with external contracts like `WISE_ORACLE`, `WISE_SECURITY`, `POSITION_NFT`, and `AAVE_HUB_ADDRESS`. If the owners or developers of these contracts have malicious intentions or become compromised, it could introduce centralization risks.

    - **Technical Risks:**

      1.  The contract uses a complex set of mappings, structs, and functions, which increases the attack surface and the potential for vulnerabilities.

      2.  There are several arithmetic operations involving divisions and multiplications, which could be susceptible to rounding errors or integer overflows/underflows if not handled properly.

      3.  The contract has several external function calls, which could be susceptible to reentrancy attacks if not implemented correctly.

      4.  The contract uses multiple external contracts, which increases the complexity and the potential for compatibility issues or versioning conflicts.

    - **Integration Risks:**

      1. The contract heavily relies on external contracts like `WISE_ORACLE`, `WISE_SECURITY`, `POSITION_NFT`, and `AAVE_HUB_ADDRESS`. Any changes or upgrades to these external contracts could break the integration and functionality of the contract.

      2. The contract interacts with various tokens and pool tokens, which could introduce integration risks if the token standards or interfaces change or if there are compatibility issues with specific token implementations.

      3. The contract has complex logic for deposit, withdrawal, borrowing, and liquidation operations, which could increase the risk of integration issues with external systems or interfaces that interact with these operations.

11. **WiseOracleHub.sol**

    - **Systemic Risks:**

      1.  `Centralization of power`: The contract has a `master` address that can add new price feeds and oracles. This centralization of power could potentially lead to abuse or censorship if the `master` address is compromised or controlled by a malicious entity.

    - **Centralization Risks:**

      1.  The `master` address has significant control over the contract's functionality, as it can add new price feeds, oracles, and perform other critical operations. If this address is compromised or controlled by a malicious entity, the entire system could be at risk.

    - **Technical Risks:**

      1.  `Dependency on external contracts`: The contract relies on external price feeds and oracles, such as ChainLink and Uniswap. If these external systems fail or are compromised, the functionality of the contract could be affected.
      2.  `Potential rounding errors`: The contract performs various calculations involving token decimals, which could lead to rounding errors or precision issues if not handled carefully.
      3.  Inline Assembly Usage: The usage of inline assembly (`assembly { ... }`) in some functions, such as `_getPool`, `_validatePoolAddress`, and `_getRoundTimestamp`, poses technical risks. Incorrect implementation or vulnerabilities in assembly code could compromise the security and stability of the contract.

    - **Integration Risks:**

      1.  Compatibility with external systems: The contract integrates with various external systems, such as ChainLink and Uniswap. Any changes or updates to these external systems could potentially break the integration or require modifications to the contract.
      2.  Reliance on external data: The contract relies on external data sources for pricing information. If these data sources provide incorrect or manipulated data, the functionality of the contract could be compromised.

12. **AaveHub.sol**

    - **Systemic Risks:**

      1.  The contract allows the master address to set and update the mapping between underlying assets and aTokens, which could lead to potential misuse or manipulation if the master address is compromised.

    - **Centralization Risks**:

      1.  The `onlyMaster` modifier restricts certain functions to be callable only by the master address. Depending on the privileges and control granted to the master address, there could be centralization risks if the master address is compromised or misused.

    - **Technical Risks**:

      1.  The contract relies on external contracts (e.g., `WISE_LENDING`, `AAVE`) and their interfaces, which could lead to potential compatibility issues or breaking changes if these external contracts are updated or modified.
      2.  The contract uses multiple external libraries and dependencies, which could introduce potential security risks or compatibility issues if these dependencies are not maintained or updated properly.

    - **Integration Risks**:

      1.  The contract heavily integrates with other external protocols (e.g., `AAVE`), which could lead to potential issues or vulnerabilities related to the integration or the external protocols themselves.
      2.  The contract assumes certain behavior and functionality from the external contracts it integrates with (e.g., `WISE_LENDING`, `AAVE`), which could lead to potential issues or vulnerabilities if these external contracts do not function as expected or if their behavior changes unexpectedly.
      3.  The contract relies on the correct mapping between underlying assets and aTokens, which could lead to potential issues or vulnerabilities if this mapping is incorrectly set or updated.

13. **PositionNFTs.sol**

    - **Centralization Risks**:

      1.  The contract uses the `OwnableMaster` contract, which means there is a central authority (the owner) who has special privileges, such as the ability to assign reserve roles, block reserve public, forward fee manager NFT, and set base URI and extension. This centralization could lead to potential misuse of power or single point of failure.

    - **Technical Risks**:

      1.  The `_toString` function could potentially lead to a re-entrancy attack as it uses a while loop and performs divisions, which could be expensive in terms of gas and potentially create an opening for a re-entrancy attack.
      2.  The `approveMint` function calls `_mintPositionForUser` within an `approve` call, which could potentially lead to unexpected behavior or security vulnerabilities.

    - **Integration Risks**:

      1.  The contract integrates with other contracts through the `ERC721Enumerable` and `OwnableMaster` contracts. If these contracts have any vulnerabilities or if they are not properly integrated, it could lead to issues in this contract.
      2.  The `forwardFeeManagerNFT` function transfers the FEE_MANAGER_NFT to a new contract. If this new contract is not secure or has vulnerabilities, it could potentially affect the functionality and security of this contract.
      3.  The `reservePositionForUser` and `mintPositionForUser` functions allow the minting of new NFTs for a user. If these functions are not properly integrated with the rest of the system, it could lead to issues such as double minting or unauthorized minting.

14. **PoolManager.sol**

    - **Systemic Risks:**

      1.  The contract relies on external libraries such as `Babylonian` for mathematical operations. Any vulnerabilities or issues in these external dependencies could potentially affect the security and functionality of this contract.

    - **Centralization Risks:**

      1.  The contract utilizes the `onlyMaster` modifier, indicating that certain functions can only be called by the master address. Depending on the privileges and control granted to the master address, there could be centralization risks if the master address is compromised or misused.

    - **Technical Risks**:

      1.  The contract uses the `createPool` and createCurv`ePool functions to create new pools, but it does not check if the pool already exists. This could potentially lead to the creation of duplicate pools or unexpected behavior.

      2.  The contract uses the `_safeTransfer` function to transfer tokens, but it does not check if the transfer was successful. If the transfer fails for any reason, it could lead to loss of tokens or unexpected behavior.

      3.  The `_getPoleValue` function uses the sqrt function from the `Babylonian` library, which could potentially lead to a re-entrancy attack as it uses a while loop and performs divisions, which could be expensive in terms of gas and potentially create an opening for a re-entrancy attack.

      4.  The contract utilizes `mathematical` operations such as square root calculations. While these operations are common in smart contracts, there's a risk of arithmetic overflow or underflow if inputs are not properly validated or handled

    - **Integration Risks:**

      1. The contract integrates with other contracts through the `WISE_SECURITY` and `FEE_MANAGER` contracts. If these contracts have any vulnerabilities or if they are not properly integrated, it could lead to issues in this contract.

      2. The `createCurvePool` function calls the `prepareCurvePools` function from the `WISE_SECURITY` contract, but it does not check if the function call was successful. If the function call fails for any reason, it could lead to unexpected behavior or security vulnerabilities.

      3. The contract uses the `_getBalance` function to fetch the balance of a token, but it does not check if the fetched balance is valid. If the fetched balance is not valid, it could lead to unexpected behavior or security vulnerabilities.

      4. The contract allows parameters to be locked once set, preventing further modifications. While this may enhance security and stability, it also introduces inflexibility, especially if adjustments are needed in response to changing conditions or requirements. Careful consideration should be given to parameter locking to avoid unintended consequences or constraints.

## Suggestions

### What could they have done better?

1. If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;

[![test-cases.jpg](https://i.postimg.cc/1zgD5wCt/test-cases.jpg)](https://postimg.cc/v1s40gdF)

Ref:https://xin-xia.github.io/publication/icse194.pdf

[![nabeel-1.jpg](https://i.postimg.cc/6qtBdLQW/nabeel-1.jpg)](https://postimg.cc/bDVXPnbW)

2. It is recommended to increase the test coverage to 100% so make sure that each and every line is battle tested

### What ideas can be incorporated?

1. `Advanced liquidation mechanisms`: Implement automated liquidation bots, partial liquidations, liquidation auction houses, etc. to minimize losses.

2. `More collateral options`: Support additional assets as collateral beyond ERC20 tokens, like NFTs. This expands the types of assets users can leverage.

3. `Yield farming integrations`: Allow deposited funds to earn yields from DeFi protocols like Aave, Compound, Yearn, etc.

4. `Varied loan types`: Support different loan types beyond variable-rate, like fixed-rate, amortizing, etc. tailored to user needs.

5. `Customizable pools`: Allow creators to configure pool-specific parameters like collateral factors, borrow caps, etc.

6. `Expand Yield Source Options`: Integrate with additional yield aggregators to diversify revenue streams and increase borrowing demand.

7. `Liquidity Mining Incentives`: Introduce liquidity mining programs to incentivize liquidity providers and bootstrap liquidity for new markets. These programs can reward users with additional tokens for providing liquidity to specific pools or participating in strategic initiatives.

8. `Add more details on the tokenomics of the WISE token`: how it will be distributed, inflation/deflation mechanics, uses cases, etc. This helps analyze the sustainability and incentives.

9. `Using Pendle Power Farms`: There is a `1-3%` initiation fee for using Pendle Power Farms, but this fee is converted into staked PEDNLE tokens (vePENDLE), which can unlock higher tiers of APY for users.

10. `Using Pendle LPs instead of leveraging PT`: Pendle LPs can provide higher returns and lower liquidation risk than leveraging PT directly. This is because Pendle LPs can accrue more PT as the value of PT goes down, which can offset any temporary drops in value.

### What’s unique?

- **LASA (Liquidity Adjusted Supply Algorithm)**: A proprietary AI algorithm that dynamically adjusts bonding curves and resonance factors to optimize capital efficiency. This is a core part of how the protocol sets variable borrowing rates.

- **Solely deposit mode**: Allows lenders to opt-out of having their funds borrowed against while still earning yield, prioritizing capital preservation.

- **NFT-based positions**: Each user's collateralized position is represented as a non-fungible token (NFT). This provides transparency over lending/borrowing amounts and enables transferrable/composable DeFi positions.

- **Robust oracle integration**: Leverages Chainlink and Uniswap TWAPs for reliable price feeds, with mechanisms like heartbeat checks and recalibration to ensure accuracy over time.

- **Capped liquidation fees**: Protects borrowers by limiting the maximum fee liquidators can charge, balancing incentives.

- **Customizable APY algorithms**: Offers competitive yields through interfaces to protocols like AAVE, optimizing revenue through arbitrage across lending rates.

- **Composable modular design**: Abstracts core components as reusable helpers/declarations, allowing for extensibility and integration with other DeFi apps.

- **Decentralized governance**: The Wise ecosystem aims to implement a DAO for community-led development and decision making over time.

## Issues surfaced from Attack Ideas

1. **Oracle manipulation**: Since the protocol relies on oracles like Chainlink for key price data, an attacker could theoretically try to manipulate oracle prices in an attempt to liquidate positions or profit from trades. The protocol mitigates this risk through mechanisms like heartbeat checks and multiple oracle integrations.

2. **Flash loan attacks**: Like other DeFi protocols, Wise Lending is potentially vulnerable to flash loan attacks where tokens are borrowed to manipulate prices or force liquidations. The protocol incorporates caps on liquidation fees and collateral ratios to minimize loss from such attacks.

3. **Parameter manipulation**: Roles like the master address or incentive owners have control over some configurable parameters. An attack could try exploiting this centralization by modifying parameters maliciously, such as setting absurd collateral factors.

4. **LASA exploit**: Since the Liquidity Adjusted Scaling Algorithm (LASA) dynamically tunes key variables, an advanced attack may try reverse-engineering it to force dislocations in borrow rates or pool utilization. Strong validation of LASA inputs/outputs would be needed.

5. **Integration exploits**: Because the system comprises many discrete contracts, an attacker may analyze inter-contract function calls and external dependencies to identify integration flaws or race conditions between components. Proper abstraction and encapsulation of couplings would help address such exploits.

6. **Mathematical exploits**: Functions performing calculations are vulnerable if inputs are not validated properly. For example, an attacker could attempt Rounding errors in accounting functions related to shares, assets, rates etc. Strong input validation across computationally intensive areas would be important to audit.


### Time spent:
110 hours