
# Wise Landing Audit Analysis

![1500x500-_1_ (1)](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/23735fea-405c-4516-8366-50df3a4c09a8)

Decentralized liquidity market that allows users to supply crypto assets and start earning a variable APY from borrowers.
  
  

# Overview

Structuring the code is crucial for conducting a thorough analysis of the code base. This enables auditors to predict the level of contract difficulty, identify critical contracts, and determine security-critical features such as payment functions or assembly usage. Additionally, this approach accurately measures the time required to perform a detailed audit and helps determine the cost of a project audit. To achieve efficiency, clarity, and improvements, it is essential to have a comprehensive view of the entire architecture. This enables the identification of the basic structure, relationships between modules, and key design patterns. By understanding the big picture, we can analyze the complexity of the code and reveal its strengths and potential for improvement with confidence.

# Introduction

In the ever-evolving landscape of cryptocurrency, the Wise Ecosystem emerges as a beacon of innovation, offering a paradigm shift towards fairness, decentralization, and long-term sustainability. Unlike conventional crypto projects driven solely by short-term gains, Wise prioritizes the creation of a robust financial ecosystem that benefits the community at its core.

At the heart of the Wise Ecosystem lies the $WISE token, serving as the backbone of its operations. Unlike typical tokens subject to inflationary pressures, $WISE stands apart by being backed by a pool of ETH and appreciating based on revenue generated from Wise dApps. This unique approach ensures stability and value preservation for token holders.

Central to the ethos of Wise is the WISE DAO, a community-driven entity that fosters ownership and governance among its participants. Unlike traditional projects where revenue flows back to centralized entities, the Wise DAO ensures that profits are shared equitably among the community, aligning incentives for long-term sustainability and growth.

One of the flagship offerings of the Wise Ecosystem is WiseLending.com, a decentralized lending platform revolutionizing access to financial services in the crypto space. With Wise Lending, users can earn passive income by depositing crypto assets into decentralized lending pools, while borrowers gain access to competitive yield-farming strategies without the constraints of traditional financial systems.

Furthermore, the introduction of the WISEr token empowers users with a stake in the DAO, allowing them to claim a share of the ecosystem's funds and participate in governance decisions. This commitment to community ownership underscores Wise's dedication to inclusivity and transparency.

In collaboration with Pendle Strategies, Wise augments its offerings with innovative yield-enhancing solutions, ensuring optimal returns for users while upholding the principles of decentralization and sustainability.

As the crypto landscape continues to evolve, the Wise Ecosystem stands as a testament to the transformative power of decentralized finance, offering a vision of a future where financial empowerment is accessible to all. Join us in reshaping the future of finance with the Wise Ecosystem.


## $WISE Token - Backbone of the Ecosystem

$WISE token is the reserve token of the Wise Ecosystem.
Unlike other tokens, $WISE isn't used for APY rewards or subject to hyper-inflation.
It's backed by a pool of ETH and appreciates based on revenue from Wise dApps.

## The WISE DAO - Community-Driven Value

Wise operates differently, with revenue shared with the community, not VCs.
The DAO ensures community ownership and drives value back to the users.

## WiseLending.com - Empowering Financial Freedom

Wise Lending offers decentralized lending pools where users earn passive income.
Borrowers access competitive yield-farming strategies with ease and without lock-ins.

## How Can Projects Benefit?

Projects with tokens can pair them with $WISE for liquidity, enhancing value.
Token-less projects can join by holding WISE in their treasury.

## WISEr Token - Power to the Community

WISEr represents DAO ownership and is minted by adding ETH to the DAO.
It allows users to claim a share of the DAO funds, promoting community governance.

## Wise Lending: Redefining DeFi Lending

It's a decentralized lending platform with no lock-ins and robust safety measures.
Power Farms offer one-click yield-farming strategies for enhanced returns.

## Pendle Strategies: Augmenting Yield

Pendle enhances APY of yield tokens and ensures decentralized standards.
Users can boost APY using Pendle LPs, with fees converted into higher APY tiers.

# Codebase quality

Overall, I consider the quality of the Wise Landing codebase to be excellent. The code appears to be very mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

# Fee Manager

| Issue                     | Description                                                                                                                                                                                                                   | Suggestion                                                                                                                                                                                                                                                                                                           |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Potential performance issue | The `claimWiseFeesBulk` function iterates through all pool tokens and calls `claimWiseFees` for each one. If there are many pool tokens, this operation can be slow and impact performance.                    | Consider implementing a more efficient data structure, such as a priority queue, to store and process pool tokens based on their priority or frequency of usage.                                                                                                                                                  |
| Excessive memory usage     | The `setAaveFlagBulk` function takes two arrays of equal length and iterates through them, calling `_setAaveFlag` for each pair of elements. This operation can consume a significant amount of memory if the arrays are large. | Consider using a more memory-efficient data structure, such as a map, to store and process the pool tokens and their corresponding underlying tokens.                                                                                                                                                                |
| Resource leak              | The `claimWiseFees` function calls `_distributeIncentives` if `totalBadDebtETH` is equal to zero. However, there is no check to ensure that `totalBadDebtETH` is actually updated after the function call. If `totalBadDebtETH` is not updated, the function will continue to call `_distributeIncentives` indefinitely, leading to a resource leak. | Add a check to ensure that `totalBadDebtETH` is updated after the function call, or consider using a different mechanism to prevent the function from being called unnecessarily.                                                                                                                                  |
| Code pattern that may impact performance | The `claimWiseFees` function uses a low-level `.call` statement to transfer funds from the `WISE_LENDING` contract to the current contract. This operation can be slow and impact performance, especially if there are network congestion or other issues.               | Consider using a higher-level function, such as `transfer` or `transferFrom`, to transfer funds between contracts. These functions provide better error handling and are generally more efficient than low-level `.call` statements.                                                                                  |


# PendlePowerFarmToken

## Avoid expensive division operations

For example, in the _validateSharePriceGrowth() function, you can calculate the maximum value more efficiently

```solidity
uint256 maximum = (timeDifference * RESTRICTION_FACTOR) / PRECISION_FACTOR_E18 + 1;
```

## Optimize loops
a loop that can potentially be expensive. To optimize this loop, consider using a more efficient data structure, such as a mapping, to store and access user rewards.
```solidity
PendlePowerFarmToken.sol
218:     function _calculateRewardsClaimedOutside()
219:         internal
220:         returns (uint256[] memory)
221:     {
```
## Reduce gas costs for external functions
are marked as external, which means they can be called by other contracts. These functions perform multiple operations, including transferring tokens and updating the contract state. To reduce gas costs, consider splitting these functions into smaller, more focused functions.
```solidity
PendlePowerFarmToken.sol
529:    function depositExactAmount(
530:         uint256 _underlyingLpAssetAmount
531:     )
532:         external
533:         syncSupply
534:         returns (
535:             uint256,
536:             uint256
537:         )
538:     {

608:     function withdrawExactShares(
609:         uint256 _shares
610:     )
611:         external
612:         syncSupply
613:         returns (uint256)
614:     {

```

## Consider using a more efficient storage structure
uses arrays to store user rewards, which can be inefficient for large numbers of users. Consider using a more efficient data structure, such as a mapping, to store user rewards.

## Use calldata instead of memory for function parameters 
Functions like _calculateRewardsClaimedOutside() use memory for function parameters. Using calldata instead can save gas costs.


# Approach we followed when reviewing the code

## AaveHelper

### Internal Functions

- `_wrapDepositExactAmount()`
- `_wrapWithdrawExactAmount()`
- `_wrapWithdrawExactShares()`
- `_wrapBorrowExactAmount()`
- `_wrapAaveReturnValueDeposit()`
- `_prepareCollaterals()`
- `_prepareBorrows()`

### Additional Functions

- `getAavePoolAPY()`: Retrieves the current APR (Annual Percentage Rate) for a given asset in the Aave pool.

### Modifiers

- `nonReentrant`: Ensures that a function cannot be called again while it is still executing.
- `validToken`: Checks if a given token is valid before proceeding with the function.

## AaveHub

`AaveHub` is an implementation of several features from `AaveHelper`, `TransferHelper`, and `ApprovalHelper`. This contract aims to enhance capital efficiency by utilizing the Aave pool. In this contract, idle funds are deposited into the corresponding Aave pool to earn supply APY. `aAave` tokens are owned by the `wiseLending` contract, but the details of calculation are governed by NFT positions.


## FeeManager

FeeManager is responsible for managing and distributing fees obtained from transactions. These fees are divided into two categories: incentives and bad debt. Incentives are fees given to users who have participated in the WISE platform. Incentives are further divided into two categories: incentive A and incentive B. Users eligible for incentive A are those who have participated in transactions on the WISE platform. While users eligible for incentive B are those who have participated in transactions and have followed specific rules set by the WISE platform.

### Functions

- `setPoolAddresses()`: Sets the pool addresses that will receive incentive and bad debt fees.
- `setIncentiveA()` and `setIncentiveB()`: Sets the percentage of incentives A and B to be given to eligible users.
- `setBadDebtLiquidation()`: Sets the percentage of bad debt to be given to eligible users.
- `setAaveFlag()` and `setAaveFlagBulk()`: Sets the Aave flag for token and base token pools.
- `setPoolFee()` and `setPoolFeeBulk()`: Sets the fee to be given to token and base token pools.
- `proposeIncentiveMaster()`: Sets a new address for the incentive master.
- `claimOwnershipIncentiveMaster()`: Changes the incentive master address to the proposed new address.
- `increaseIncentiveA()` and `increaseIncentiveB()`: Increases the percentage of incentives A and B to be given to eligible users.
- `changeIncentiveUSDA()` and `changeIncentiveUSDB()`: Sets a new address for USDA and USDB incentives.
- `addPoolTokenAddress()` and `addPoolTokenAddressManual()`: Sets a new address for the pool token to receive incentive and bad debt fees.
- `removePoolTokenManual()`: Removes a previously set pool token address.
- `increaseTotalBadDebtLiquidation()` and `setBadDebtUserLiquidation()`: Increases the percentage of bad debt to be given to eligible users.
- `setBeneficial()` and `revokeBeneficial()`: Sets or removes addresses as beneficiaries of incentive and bad debt fees.
- `claimWiseFeesBulk()` and `claimWiseFees()`: Manages and distributes incentive and bad debt fees to eligible users.
- `claimFeesBeneficial()`: Manages and distributes incentive and bad debt fees to users designated as beneficial.
- `paybackBadDebtForToken()` and `paybackBadDebtNoReward()`: Manages and returns bad debt to users who have performed transactions but have not fulfilled their obligations.

### Access Control

Access to certain functions is controlled by:

- `onlyMaster()`: Accessible only by the contract owner.
- `onlyIncentiveMaster()`: Accessible only by the incentive master.
- `onlyWiseLending()`: Accessible only by the WISE Lending contract.
- `onlyWiseSecurity()`: Accessible only by the WISE Security contract.
- `onlyBeneficial()`: Accessible only by beneficiaries of incentive and bad debt fees.

### Input and Output

- `_checkValue()`: Checks if the input value is valid or not.
- `_safeTransfer()`: Safely transfers tokens to the destination address.
- `_safeTransferFrom()`: Safely transfers tokens from the sender address to the destination address.

### Contract State Control

- `_increaseTotalBadDebt()`: Increases the percentage of bad debt to be given to eligible users.
- `_setBadDebtPosition()`: Sets the position of bad debt to be given to eligible users.
- `_setAllowedTokens()`: Sets or removes addresses as beneficiaries of incentive and bad debt fees.
- `_distributeIncentives()`: Distributes incentive and bad debt fees to eligible users.
- `_updateUserBadDebt()`: Updates the bad debt value of users who have performed transactions but have not fulfilled their obligations.

### Information Retrieval

- `getPoolTokenAddressesLength()`: Retrieves the number of pool addresses that have been set.
- `getPoolTokenAdressesByIndex()`: Retrieves the pool address based on the given index.


## MainHelper

`MainHelper` imports and inherits from another contract called `WiseLowLevelHelper`. It contains several functions to help manage a lending and borrowing system.

- `calculateLendingShares`: Calculates the number of lending shares obtained by depositing a certain amount of a specific pool token. It considers total deposit shares and pseudo total pool of the specified token.
- `_calculateShares`: Private function to calculate shares based on product, pseudo, and max share price.
- `calculateBorrowShares`: Calculates the number of borrow shares obtained by borrowing a certain amount of a specific pool token. It considers total borrow shares and pseudo total borrow amount of the specified token.
- `cashoutAmount`: Calculates the amount of a specific pool token obtained by redeeming a certain number of lending shares. It considers pseudo total pool and total deposit shares of the specified token.
- `_cashoutAmount`: Internal function to calculate the cashout amount.
- `paybackAmount`: Calculates the number of borrow shares needed to repay a certain amount of a specific pool token. It considers pseudo total borrow amount and total borrow shares of the specified token.
- `_preparationsWithdraw`: Internal function to check if the caller is the owner of a specific NFT and calculates the number of lending shares that can be withdrawn from a specific pool token.
- `_getValueUtilization`: Private function to calculate the utilization of a specific pool token. It considers total pool and pseudo total pool of the specified token.
- `_updateUtilization`: Private function to update the utilization of a specific pool token.
- `_checkCleanUp`: Private function to check if the contract has received new tokens for a specific pool token and updates total pool and bare token variables accordingly.
- `_onlyIsolationPool`: Internal function to check if a specific pool address is an isolation pool.
- `_validateIsolationPoolLiquidation`: Internal function to check if input parameters for a liquidation are valid.
- `_checkLiquidatorNft`: Internal function to check if provided NFTs for a liquidation are valid.
- `_getBalance`: Internal function to return the balance of a specific token in the contract.
- `_cleanUp`: Internal function to update total pool and bare token variables for a specific pool token if the contract has received new tokens.
- `_getAllowedDifference`: Private function to calculate the allowed difference between total pool and pseudo total pool for a specific pool token.
- `_preparePool`: Internal function to update total pool and bare token variables for a specific pool token and update pseudo total amounts for all tokens in the pool.
- `_preparationTokens`: Internal function to update pseudo total amounts for all tokens in a specific pool and set new borrow rates for the tokens.
- `_curveSecurityChecks`: Internal function to perform security checks for all curve pools in the contract.
- `_updatePseudoTotalAmounts`: Private function to update pseudo total amounts for all tokens in a specific pool and update borrow rate for the pool.
- `_increasePositionLendingDeposit`: Internal function to increase the number of lending shares for a specific position and pool token.
- `_decreaseLendingShares`: Internal function to decrease the number of lending shares for a specific position and pool token.
- `_addPositionTokenData`: Internal function to add a new token to the list of tokens for a specific position if it does not already exist.
- `_getHash`: Private function to calculate a hash value for a specific NFT and pool token.
- `_removePositionData`: Private function to remove a specific token from the list of tokens for a specific position.
- `_deleteLastPositionLendingData`: Private function to remove the last token from the list of tokens for a specific position.


## OracleHelper

OracleHelper is responsible for adding and managing price feeds and aggregators for various tokens.

### Variable Declarations

- `ZERO_FEED`
- `ZERO_AGGREGATOR`
- `PRECISION_FACTOR_E4`
- `ALLOWED_DIFFERENCE`
- `ETH_USD_PLACEHOLDER`
- `UNI_V3_FACTORY`
- `WETH_ADDRESS`
- `TWAP_PERIOD`
- `GRACE_PEROID`
- `IS_ARBITRUM_CHAIN`
- `heartBeat`

These variables are used throughout the contract for various purposes, such as setting default values, storing addresses, and managing time intervals.

### Functions

- `_addOracle`: Adds a price feed for a given token with its underlying feed tokens.
- `_addAggregator`: Adds an aggregator for a given token.
- `_checkFunctionExistence`: Checks if a specific function exists in the token aggregator contract.
- `_compareMinMax`: Compares the answer from the aggregator with the minimum and maximum values.
- `latestResolverTwap`: Returns the latest Twaps USD value for a given token.
- `_validateAnswer`: Validates the answer from the price feed or Twap oracle.
- `_writeUniTwapPoolInfoStruct` and `_writeUniTwapPoolInfoStructDerivative`: Add uniTwapPoolInfo for a given token and its derivative.
- `_getRelativeDifference` and `_compareDifference`: Calculate and compare the relative difference between two values.
- `_getChainlinkAnswer`: Retrieves the Chainlink answer for a given token address.
- `getETHPriceInUSD`: Returns the ETH price in USD.
- `_getPool`, `_validateTokenAddress`, `_validatePoolAddress`, `_validatePriceFeed`, and `_validateTwapOracle`: Validate and retrieve pool-related information.
- `_getTwapPrice` and `_getTwapDerivatePrice`: Calculate the Twap price for a given token and its derivative.
- `decimals`: Returns the decimals of a token.
- `_recalibrate`, `sequencerIsDead`, `_chainLinkIsDead`, `_recalibratePreview`, `_getIterationCount`, `_getRoundTimestamp`, and `getLatestRoundId`: Various helper functions to manage heartbeats, timestamps, and round data.


## PendlePowerFarmController

PendlePowerFarmController manages and controls various aspects of the Pendle Power Farms.

### Variables

- `PENDLE_POWER_FARM_TOKEN_FACTORY`: An immutable variable holding the address of the PendlePowerFarmTokenFactory contract.

### Constructor

Initializes the PendlePowerFarmController `PENDLE_POWER_FARM_TOKEN_FACTORY`.

### Function Key

- `withdrawLp`: Transfers a specified amount of LP tokens from a Pendle market to a specified address.
- `exchangeRewardsForCompoundingWithIncentive`: Exchanges reward tokens for compounding with an incentive. Calculates the amount of reward tokens to send based on the reward amount and the exchange incentive.
- `exchangeLpFeesForPendleWithIncentive`: Exchanges LP fees for Pendle tokens with an incentive. Calculates the amount of Pendle tokens to send based on the LP shares and the exchange incentive.
- `skim`: Transfers a specified amount of assets from a Pendle market to the master address.
- `addPendleMarket`: Adds a new Pendle market with the given parameters. Deploys a new Pendle Power Farm token using the PendlePowerFarmTokenFactory.
- `updateRewardTokens`: Updates the reward tokens for a Pendle market.
- `changeExchangeIncentive`: Changes the exchange incentive for the Pendle Power Farms.
- `changeMintFee`: Changes the mint fee for a specific Pendle market.
- `lockPendle`: Increases the locked Pendle amount for a specified expiry time.
- `claimArb`: Claims rewards from the Arbitrum chain.
- `withdrawLock`: Withdraws locked Pendle tokens.
- `increaseReservedForCompound`: Increases the reserved amount for compounding.
- `overWriteIndex`: Overwrites the index for a specific reward token in a Pendle market.
- `overWriteIndexAll`: Overwrites the index for all reward tokens in a Pendle market.
- `overWriteAmounts`: Overwrites the reserved amounts for compounding in a Pendle market.
- `claimVoteRewards`: Claims voting rewards for the contract.
- `forwardETH`: Transfers Ether to a specified address.
- `vote`: Casts a vote for specified pools with specified weights.


## PendlePowerFarmLeverageLogic

PendlePowerFarmLeverageLogic manages the opening and closing of positions in a leveraged yield farming strategy using flash loans from Balancer.

### Key Functions

- `_executeBalancerFlashLoan`: Initiates a flash loan from Balancer with specified parameters. Sets the `allowEnter` variable to true, allowing the `receiveFlashLoan` function to be called.
- `receiveFlashLoan`: Called by Balancer after the flash loan initiation. Checks loan parameters and either opens or closes a position based on the `initialAmount` variable.
- `_logicClosePosition`: Called when closing a position. Repays borrowed shares, withdraws LP tokens, swaps them for the desired token, and sends the token or ETH back to the caller.
- `_getEthBack`: Swaps the desired token back to ETH.
- `_getTokensUniV3`: Swaps tokens using Uniswap V3.
- `_swapStETHintoETH`: Swaps stETH into ETH using Curve.
- `_withdrawPendleLPs`: Withdraws the LP tokens.
- `_paybackExactShares`: Pays back the borrowed shares.
- `_closingRouteToken`: Sends the desired token back to the caller.
- `_closingRouteETH`: Sends ETH back to the caller.
- `_logicOpenPosition`: Called when opening a position. Deposits ETH, swaps it for the desired token, adds liquidity, borrows the flash loan token, and deposits it as collateral.
- `_borrowExactAmount`: Borrows the specified amount of the flash loan token.
- `_coreLiquidation`: Performs liquidation checks and initiates the liquidation process.

### Variables

- `BALANCER_VAULT`
- `BALANCER_ADDRESS`
- `PENDLE_ROUTER`
- `PENDLE_MARKET`
- `PENDLE_CHILD`
- `WISE_LENDING`
- `AAVE_HUB`
- `WETH_ADDRESS`
- `ENTRY_ASSET`
- `ST_ETH_ADDRESS`
- `PRECISION_FACTOR_E18`
- `FIVTY_PERCENT`
- `UNISWAP_V3_ROUTER`

## PendlePowerFarmMathLogic

PendlePowerFarmMathLogic is an abstract contract that inherits from PendlePowerFarmDeclarations.

### Modifiers and Functions

- `updatePools()`: Modifier ensuring pools are updated before executing functions using them. Checks for reentrancy and updates pools.
- `_updatePools()`: Manually syncs WISE Lending pools for WETH, AAVE-WETH, and PENDLE-CHILD.
- `_checkReentrancy()`: Checks for reentrancy attacks by examining `sendingProgress` and `WISE_LENDING.sendingProgress()` variables. Reverts transaction with an `AccessDenied()` error if true.
- `_getPositionBorrowShares()`: Returns borrow shares of a position with a given NFT ID and borrow token.
- `_getPositionBorrowSharesAave()`: Returns borrow shares of a position with a given NFT ID and borrow token for AAVE.
- `_getPositionBorrowTokenAmount()`: Returns the borrow token amount for a given NFT ID.
- `_getPositionBorrowTokenAmountAave()`: Returns the borrow token amount for a given NFT ID for AAVE.
- `_getPositionLendingShares()`: Returns the lending shares of a position with a given NFT ID and lending token.
- `_getPositionCollateralTokenAmount()`: Returns the collateral token amount for a given NFT ID.
- `getPositionBorrowETH()`: Returns the total borrow amount in ETH for a given NFT ID.
- `getTotalWeightedCollateralETH()`: Returns the total weighted collateral amount in ETH for a given NFT ID.
- `_getTokensInETH()`: Returns the equivalent amount in ETH for a given token amount.
- `_getEthInTokens()`: Returns the equivalent amount in tokens for a given ETH amount.
- `getLeverageAmount()`: Returns the leveraged amount for a given initial amount and leverage.
- `_getApproxNetAPY()`: Returns the approximate net APY for a given initial amount, leverage, and WstETH APY.
- `_getNewBorrowRate()`: Returns the new borrow rate for a given borrow amount.
- `_checkDebtRatio()`: Checks if the debt ratio for a given NFT ID is under 100%.

## WiseCore

WiseCore is an abstract contract that inherits from `MainHelper` and `TransferHelper`.

### Internal Functions

- `_prepareAssociatedTokens`: Prepares associated tokens for lending and borrowing.
- `_coreWithdrawToken`: Withdraws tokens from a pool with security checks.
- `_handleDeposit`: Deposits tokens into a pool with security checks and event emission.
- `_checkPositionLocked`: Checks if a position is locked for powerFarms.
- `_checkDeposit`: Performs security checks for deposit logic.
- `_checkAllowDeposit`: Checks if a caller is allowed to deposit.
- `_checkMaxDepositValue`: Checks if the deposit amount for a pool token is reached.
- `_coreWithdrawBare`: Withdraws tokens from a pool without security checks.
- `_coreBorrowTokens`: Borrows tokens from a pool with security checks.
- `_withdrawPureCollateralLiquidation`: Withdraws pure collateral for liquidation.
- `_withdrawOrAllocateSharesLiquidation`: Withdraws or allocates shares for liquidation.
- `_calculateReceiveAmount`: Calculates the amount to receive during liquidation.
- `_calculatePotentialPureExtraCashout`: Calculates potential pure extra cashout during liquidation.

### External Functions

- `checkPositionLocked`: Checks if a position is locked for powerFarms.
- `checkDeposit`: Performs security checks for deposit logic.

### State Variables

- `positionLocked`: A mapping that stores whether a position is locked for powerFarms.
- `maxDepositValueToken`: A mapping that stores the maximum deposit value for a pool token.
- `globalPoolData`: A mapping that stores global pool data.
- `lendingPoolData`: A mapping that stores lending pool data.
- `userBorrowShares`: A mapping that stores user borrow shares.
- `pureCollateralAmount`: A mapping that stores pure collateral amounts.
- `userLendingData`: A mapping that stores user lending data.
- `positionLendTokenData`: A mapping that stores position lend token data.
- `positionBorrowTokenData`: A mapping that stores position borrow token data.
- `WISE_ORACLE`: An instance of the WISE oracle contract.
- `WISE_SECURITY`: An instance of the WISE security contract.
- `POSITION_NFT`: An instance of the position NFT contract.
- `AAVE_HUB_ADDRESS`: The address of the Aave hub contract.
- `PRECISION_FACTOR_E18`: A constant representing the precision factor in E18.


## PoolManager

PoolManager implements functions for deposit, withdraw, borrow, and payback. It uses a variable borrow rate determined through the utilization of the pool and a bonding curve adjusted by a Lending Automated Scaling Algorithm (LASA).

### Features

- Users can deposit and withdraw assets in private mode, keeping their funds private and allowing withdrawals even when pools are borrowed empty.
- Supports Aave pools for maximal capital efficiency, earning Aave supply APY for unused funds.
- Special curve pools can be used as collateral, opening up new usage possibilities for these asset types.
- Users can pay back their borrow with lending shares of the same asset type, making it easier to manage their positions.
- Users can save their collaterals and borrows inside a position NFT, enabling trading of entire positions or use in second-layer contracts.

### Modifiers

- `healthStateCheck`: Ensures correct execution of functions and access control.
- `syncPool`: Syncs pool data.
- `onlyAaveHub`: Restricts certain functions to be called only by the Aave Hub contract.
- `onlyFeeManager`: Restricts certain functions to be called only by the Fee Manager contract.
- `onlyIsolationPool`: Restricts certain functions to be called only by the Isolation Pool contract.

### State Variables

- `master`: The contract's master address, able to call specific functions like `syncManually`.
- `_wiseOracleHubAddress`: Address of the Wise Oracle Hub used for price feeds and oracle-related functionality.
- `_nftContract`: Address of the NFT contract used for position management.

### Checks

- Checks for owner position:
  Functions like `collateralizeDeposit`, `unCollateralizeDeposit`, `withdrawExactAmount`, `withdrawExactShares`, `borrowExactAmount`, `borrowExactAmountETH`, `paybackExactAmount`, `paybackExactAmountETH`, and `paybackExactShares` ensure the caller is the owner of the position NFT.
  
- Checks for liquidator NFT:
  `liquidatePartiallyFromTokens` function includes checks to ensure the liquidator provides a valid NFT.
  
- Checks for isolation pool registration:
  `setRegistrationIsolationPool` function includes checks to ensure the caller is an isolation pool.

## WiseSecurity

WiseSecurity is used to implement security checks for various operations called by WiseLending. It checks liquidation, withdrawal, borrowing, depositing, and other operations. It also includes view functions for returning various data related to a user's position such as debt ratio, lending APY, borrow APY, net APY, etc. Additionally, it includes functions for setting and checking blacklisted tokens, security workers, and performing security shutdowns.

### Modifiers and Functions

- `onlyWiseLending`: Modifier ensuring that a function can only be called by the WiseLending contract.
- `_onlyWiseLending`: Private view function used by the `onlyWiseLending` modifier to check if the contract is being called by the WiseLending contract.
- Constructor function sets the address of the master, WiseLending, and AaveHub contracts.
- `getLiveDebtRatio`: View function returning the current debt ratio of a position in normal mode.
- `setLiquidationSettings`: Function used by the master to set liquidation incentives and boundaries.
- `checksLiquidation`: View function checking for liquidation logic.
- `prepareCurvePools`: Function used to prepare curve pools.


## WiseSecurityHelper

WiseSecurityHelper is a contract that inherits from WiseSecurityDeclarations and provides functions for calculating overall collateral and borrow amounts for a given non-fungible token (NFT) ID.

### Collateral Calculation Functions

- `overallETHCollateralsBoth`: Calculates both weighted and unweighted total collateral amounts for a given NFT ID.
- `overallETHCollateralsWeighted`: Calculates the weighted total collateral amount for a given NFT ID.
- `overallETHCollateralsBare`: Calculates the unweighted total collateral amount for a given NFT ID.
- `_overallETHCollateralsWeighted`: Internal version of `overallETHCollateralsWeighted` with an additional parameter for interval calculation.
- `getFullCollateralETH`: Calculates the full collateral amount for a given NFT ID and pool token.
- `_isUncollateralized`: Internal function to check if a pool token is uncollateralized.
- `_getCollateralOfTokenETHUpdated`: Internal function to calculate updated Ether collateral amount for a given NFT ID, pool token, and interval.
- `getETHCollateralUpdated`: Calculates updated Ether collateral amount for a given NFT ID, pool token, and interval.
- `getETHCollateral`: Calculates Ether collateral amount for a given NFT ID and pool token.
- `_getTokensInEth`: Internal function to convert a given amount of pool tokens to Ether.

### Borrow Calculation Functions

- `overallETHBorrowBare`: Calculates total borrow amount for a given NFT ID without heartbeat or blacklisted checks.
- `overallETHBorrowHeartbeat`: Calculates total borrow amount for a given NFT ID without blacklisted checks.
- `overallETHBorrow`: Calculates total borrow amount for a given NFT ID.
- `_checkBlacklisted`: Internal function to check if a pool token is blacklisted.
- `_overallETHBorrow`: Internal function to calculate total borrow amount for a given NFT ID and interval.
- `_getETHBorrowUpdated`: Internal function to calculate updated Ether borrow amount for a given NFT ID, pool token, and interval.
- `getETHBorrow`: Calculates Ether borrow amount for a given NFT ID and pool token.

### Helper and Modifier Functions

- `checkTokenAllowed`: Checks if a given pool token is allowed to borrow.
- `checkHeartbeat`: Checks if the Chainlink feed for a given pool token was updated within the expected timeframe.
- `_checkPositionLocked`: Checks if the position with a given NFT ID is locked for interactions.
- `_checkMaxFee`: Internal pure function to return the possible fee for liquidation.
- `calculateWishPercentage`: Calculates the percentage of the receiving token that the liquidator receives for liquidation.
- `_checkHealthState`: Checks if the debt ratio is not greater than 100% after withdrawing a given pool token for a given amount.
- `_getState`: Internal function to check if the debt ratio is greater than 100% after withdrawing a given pool token for a given amount.
- `checksRegister`: Checks if a user can register a power farm for a given NFT ID.
- `canLiquidate`: Checks if a position can be liquidated.
- `checkMaxShares`: Checks if the share amount to pay is greater than the maximum shares that can be paid.
- `checkBadDebtThreshold`: Checks if a position has bad debt.
- `getPositionLendingAmount`: Calculates the lending token amount for a given NFT ID and pool token.
- `getPositionBorrowAmount`: Calculates the borrow token amount for a given NFT ID and pool token.
- `checkOwnerPosition`: Checks the owner of a given NFT ID.
- `getBorrowRate`: Returns the borrow rate from a pool with a given pool token.
- `getLendingRate`: Returns the lending rate from a pool with a given pool token.
- `_getPossibleWithdrawAmount`: Internal function to calculate the possible withdraw amount of a given pool token under current borrow and collateral amounts for a given NFT ID and interval.
- `_setPoolState`: Sets the state of all pools for borrow and deposit actions.
- `_checkPoolCondition`: Checks if a given pool token is blacklisted.
- `_checkSuccess`: Checks if a low-level byte call of a function with `.call()` was successful.



# Analysis of the code base and diagram architecture

# WiseLending
![wiselanding](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/fcf7a0f0-13b9-403d-916d-4161552602ac)

# Main Helper
![mainhelper](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/a442c83b-d2f1-4e52-9390-d3462d292f1a)

# PendlePowerFarmToken
![PendlePowerFarmToken](https://github.com/yongsxyz/Audit-Analysis/assets/39027215/154d6036-9d79-4bdb-87af-b5fd8958f976)

# Centralization risk

Concentration of Powers: The contract is an abstract contract that contains several critical functions, such as checking the health state of a position, liquidation checks, and managing the state of pools. If this contract is not properly inherited and its functions are not overridden by other contracts, it may result in a concentration of powers in a single contract, leading to centralization.

Single Point of Failure: The contract could potentially become a single point of failure. If this contract has a bug or is compromised, all the contracts that inherit from it could also be affected, leading to a centralized point of failure.

Immutable Contract: The contract does not seem to be upgradable. Once deployed, its code cannot be changed. This could potentially lead to centralization as any bugs or vulnerabilities in the contract cannot be fixed without deploying a new contract.

Centralized Control Over Fees: The contract has a function _setPoolState that can lock or unlock all pools for borrow and deposit actions. If this function is controlled by a centralized entity, it could lead to centralization.

# Conclusion

In general, the Wise Landing project exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.



### Time spent:
38 hours