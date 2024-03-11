### Advanced Analysis Report for [Wise Lending Protocol](https://github.com/code-423n4/2024-02-wise-lending) by K42

#### Overview

The [Wise Lending Protocol](https://github.com/code-423n4/2024-02-wise-lending) is a decentralized lending platform that allows users to collateralize their assets and borrow tokens against them. It utilizes a unique Lending Automated Scaling Algorithm (LASA) to adjust the bonding curves and determine the variable borrow rates over time based on pool utilization. It incorporates various features such as solely deposit and withdraw, Aave pools integration, special curve pools inside beefy farms, and position NFTs for collateral and borrow management.

#### Understanding the Ecosystem:

- The core contracts of the protocol are WiseLending, WiseCore, and WiseSecurity, which handle the main lending, borrowing, and security functionalities respectively.

- The protocol interacts with various external contracts and systems, including Aave, Curve, Uniswap V3, Balancer, Pendle, and Chainlink oracles, to enable additional features and integrations.

- The WiseOracleHub contract acts as an on-chain extension for price feeds, allowing the protocol to retrieve accurate token prices and handle oracle-related functionalities.

#### Codebase Quality Analysis:

- The codebase follows a modular and hierarchical structure, with contracts inheriting from abstract base contracts and utilizing libraries for specific functionalities.

**1. Structures and Libraries:**

- The protocol defines several structs to represent pool data, such as `LendingEntry`, `BorrowRatesEntry`, `AlgorithmEntry`, `GlobalPoolEntry`, `LendingPoolEntry`, `BorrowPoolEntry`, and `TimestampsPoolEntry`. These structs help in organizing and managing the pool-related data efficiently.

- The codebase utilizes libraries like `Babylonian` for mathematical computations, which enhances code reusability and readability.

**2. Modularity and Inheritance:**

- The contracts are structured in a modular way, with each contract focusing on specific functionalities. For example, `WiseLending` handles the main lending and borrowing operations, while `WiseCore` and `WiseSecurity` handle core functionalities and security checks respectively.

- The contracts inherit from abstract base contracts like `PoolManager`, `WiseCore`, and `WiseSecurity`, which provide common functionalities and help in maintaining a clean and organized codebase.

**3. Access Control and Permissions:**

- The protocol uses modifiers like `onlyMaster`, `onlyWiseLending`, `onlyAaveHub`, `onlyFeeManager`, and `onlyWiseSecurity` to restrict access to certain functions based on the caller's role or permissions. This helps in enforcing proper access control and preventing unauthorized actions.

- The `OwnableMaster` contract is used as a base contract for implementing ownership and master-level control over the protocol.

**4. Error Handling and Validation:**

- The codebase defines custom error types using the `error` keyword, such as `NotWiseLendingSecurity`, `NotAllowedEntity`, `InvalidAction`, `ValueIsZero`, `ValueNotZero`, and others. These error types provide clear and descriptive error messages when certain conditions are not met.

- The contracts also utilize custom errors defined in `PendlePowerFarmDeclarations` and `WiseLendingDeclaration`, which provide more gas-efficient and informative error handling compared to traditional `require` statements.

- The contracts use `require` statements and custom error types to validate input parameters, check preconditions, and handle error cases. This helps in preventing invalid or unexpected behaviour and provides informative error messages to the users.

**5. Reentrancy Protection:**

- The protocol implements reentrancy protection using the `_checkReentrancy` function in the `WiseCore` contract. This function checks if there are any ongoing transactions or if the `sendingProgress` flag is set, and reverts if a reentrancy attempt is detected.

- The `nonReentrant` modifier is used in the `AaveHub` contract to prevent reentrancy attacks on specific functions.

**6. Upgrade Mechanism:**

- The protocol does not seem to have an explicit upgrade mechanism or use of proxy contracts for upgradability. The contracts are deployed as standalone contracts without any apparent upgradeability features.

**7. Helper Contracts and Libraries:**

- The protocol makes extensive use of helper contracts and libraries, such as `TransferHelper`, `ApprovalHelper`, and `SendValueHelper`, which enhance code reusability and modularity.

- These helper contracts provide safe and gas-efficient implementations of common functionalities like token transfers, approvals, and ETH sending.

Overall, the codebase follows good practices in terms of modularity, inheritance, access control, error handling, and the use of helper contracts and libraries. The use of structs, custom errors, and modifiers enhances code reusability, maintainability, and gas efficiency.

#### Architecture Recommendations:

- Evaluate the potential benefits of introducing an upgrade mechanism, such as using proxy contracts or a plugin-based architecture, to enable future upgrades and enhancements to the protocol without requiring a full redeployment.

- Consider implementing additional security measures, such as rate limiting or more circuit breakers, to mitigate the impact of potential attacks or abnormal market conditions.

#### Centralization Risks:

- **Possible Risk**: The protocol relies on the `master` address, which is controlled by a single entity or a multisig, for critical operations and decision-making across various contracts, including the core contracts, `FeeManager`, and `PendlePowerFarmController`. This introduces a centralization risk, as the `master` address holds significant control over the protocol.

- **Potential Impact**: If the `master` address is compromised or if there is a malicious action by the controlling entity, it could lead to unauthorized changes, fund manipulation, or disruption of the protocol's functionalities.

- **Mitigation Strategy**: Implement a decentralized governance mechanism, such as a DAO (Decentralized Autonomous Organization), to distribute control and decision-making power among stakeholders. Establish clear guidelines and processes for proposing, voting, and executing changes to the protocol.

#### Mechanism Review:

- The Lending Automated Scaling Algorithm (LASA) dynamically adjusts the bonding curves and determines the variable borrow rates based on pool utilization. This mechanism helps in maintaining a balanced and sustainable lending ecosystem.

- The integration with Aave and Curve pools allows for enhanced capital efficiency and additional yield generation opportunities for users.

- The liquidation mechanism is primarily handled by the `WiseCore` contract, which interacts with the `WiseSecurity` contract for price calculations and health checks. It enables the liquidation of undercollateralized positions, helping to maintain the stability and solvency of the protocol.

#### Systemic Risks:

- The protocol's dependence on external oracles, such as Chainlink and Uniswap V3 TWAPs, introduces risks associated with oracle failures, manipulations, or attacks. Any issues with the oracle data feeds could impact the accuracy of price calculations and lead to unexpected behaviour.

- The integration with external protocols like Aave, Curve, Uniswap V3, Balancer, and Pendle exposes the protocol to the risks and vulnerabilities present in those systems. A failure or exploit in any of the integrated protocols could have a cascading effect on the Wise Lending Protocol.

#### Areas of Concern:

1. Reentrancy vulnerability in the `paybackBadDebtForToken` function of the `FeeManager` contract:
   - The function first updates the user's bad debt using `updatePositionCurrentBadDebt(_nftId)` and then transfers the receiving token to the caller using `_safeTransfer(_receivingToken, msg.sender, receivingAmount)`.
   - If the `msg.sender` is a malicious contract, it can potentially perform a reentrancy attack by calling back into the `FeeManager` contract before the state updates are completed, leading to unexpected behaviour or fund manipulation.

2. Improper access control in the `PendlePowerFarmController` contract:
   - The `addPendleMarket` function allows the contract owner (master) to add a new Pendle market and set the reward tokens.
   - However, there is no validation on the input parameters, such as checking if the provided `_pendleMarket` address is a valid and trusted contract.
   - An attacker with ownership access could potentially add a malicious Pendle market, leading to the manipulation of reward tokens and user funds.

3. Unsafe ERC20 token transfers in the `PendlePowerFarmLeverageLogic` contract:
   - The `_logicOpenPosition` function performs a series of token transfers and interactions with external contracts, such as Pendle Router, Pendle SY, and Uniswap V3 Router.
   - However, there is no proper error handling or checks for the return values of these external calls.
   - If any of the external contracts fail or return unexpected values, it could lead to loss of funds or stuck transactions.

4. Incorrect calculation of collateral and debt in the `WiseCore` contract:
   - The `_coreLiquidation` function calculates the collateral percentage and debt ratio based on the liquidator's input.
   - However, there is no validation or sanity check on the input parameters, such as `_shareAmountToPay` and `_percentWishCollateral`.
   - An attacker could potentially provide manipulated input values to inflate the collateral percentage or debt ratio, leading to improper liquidations and loss of funds.

5. Unhandled edge cases in the `PendleLpOracle` contract:
   - The `latestAnswer` function relies on the Pendle market's oracle state and the chainlink price feed to calculate the LP token price.
   - However, there are no checks for extreme cases, such as when the `lpRate` or `answerFeed` values are zero or abnormally high.
   - These edge cases could lead to unexpected behaviour or incorrect price calculations, affecting the integrity of the oracle and dependent contracts.

6. Potential risks with low-level assembly code in `PendlePowerFarmTokenFactory`:
   - The `_clone` function in the `PendlePowerFarmTokenFactory` contract uses low-level assembly code to deploy new instances of the `PendlePowerFarmToken` contract.
   - While the use of assembly can be justified for certain optimizations, it requires extra scrutiny to ensure its correctness and safety.
   - Any errors or vulnerabilities in the assembly code could lead to unexpected behaviour or potential exploits.

#### Codebase Analysis:

- The codebase demonstrates a modular and well-organized structure, with contracts separated based on their functionalities and responsibilities. This promotes code reusability, maintainability, and readability.

- The use of abstract contracts and libraries enhances code modularity and allows for efficient code sharing and reuse.

- The protocol incorporates various security measures, such as access control modifiers, reentrancy guards, and input validation, to mitigate common security risks.

#### Contract Details, with Recommendations:

### 1. WiseLending Contract
The WiseLending contract is the main entry point for user interactions, handling deposits, withdrawals, borrows, and paybacks. It relies on WiseCore for core lending logic and WiseSecurity for security checks.

Recommendations:
- Implement input validation in functions like `depositExactAmount` and `withdrawExactAmount` to prevent potential edge cases or exploits.
- Add a reentrancy guard to the `receive` function to prevent potential reentrancy attacks.
- Implement granular access control for functions like `collateralizeDeposit` and `unCollateralizeDeposit`.

### 2. WiseCore Contract
The WiseCore contract implements core lending and borrowing logic, including token transfers, liquidations, and pool management.

Recommendations:
- Thoroughly review the `_coreLiquidation` function to ensure it handles edge cases correctly and is not vulnerable to reentrancy attacks.
- Add input validation in functions like `_withdrawOrAllocateSharesLiquidation` to prevent division-by-zero errors or unexpected behaviour.
- Implement strict access controls for functions like `syncManually` to prevent unauthorized manipulation of pool states.

### 3. WiseSecurity Contract
The WiseSecurity contract performs security checks and validations, such as health factor calculation, token allowance, and price feed staleness.

Recommendations:
- Test the `checkHealthState` function thoroughly to ensure accurate calculation of position health states and prevent incorrect liquidations.
- Implement additional checks in the `checkTokenAllowed` function to handle potential edge cases or changes in the token allowance mechanism.
- Consider adding a fallback mechanism for the `checkHeartbeat` function to handle scenarios where the price feed becomes stale or unavailable.

### 4. PoolManager Contract
The PoolManager contract handles the creation and configuration of lending pools, including setting pool parameters and updating the LASA algorithm.

Recommendations:
- Implement strict access controls for functions like `setParamsLASA` and `setPoolParameters` to ensure only authorized entities can modify pool parameters.
- Add input validation in the `createPool` function to prevent the creation of invalid or malicious pools.
- Regularly monitor and assess the effectiveness of the LASA algorithm and make necessary adjustments.

### 5. FeeManager Contract
The FeeManager contract handles fee distribution, bad debt management, and incentives for the protocol.

Recommendations:
- Implement robust access control for functions like `setRepayBadDebtIncentive` and `setAaveFlag` to ensure only authorized entities can modify critical parameters.
- Add input validation and error handling in functions like `paybackBadDebtForToken` and `claimWiseFees` to prevent potential exploits or unexpected behaviour.
- Regularly review and update the fee distribution and incentive mechanisms to align with the protocol's goals and market conditions.

### 6. AaveHub Contract
The AaveHub contract integrates with the Aave lending protocol, enabling users to deposit and borrow assets through Aave.

Recommendations:
- Thoroughly test the integration with Aave to ensure it handles all possible scenarios and edge cases correctly.
- Implement error handling and input validation in functions like `_wrapAaveReturnValueDeposit` to handle potential failures or unexpected return values from Aave.
- Consider adding a mechanism to pause or restrict interactions with Aave in case of security incidents or vulnerabilities discovered in the Aave protocol.

### 7. PendlePowerFarmController Contract
The PendlePowerFarmController contract manages the Pendle Power Farm mechanism for liquidity provision and yield optimization.

Recommendations:
- Implement strict access controls and input validation in functions like `addPendleMarket` and `changeMintFee` to prevent unauthorized additions of markets or modifications of critical parameters.
- Add checks and error handling in functions like `exchangeRewardsForCompoundingWithIncentive` and `lockPendle` to handle potential edge cases or failures in the underlying Pendle contracts.
- Regularly monitor and assess the performance and security of the integrated Pendle contracts.

### 8. PendlePowerFarm Contract
The PendlePowerFarm contract handles the entry and exit of users into the Pendle Power Farm.

Recommendations:
- Thoroughly test the `enterFarm` and `exitFarm` functions to ensure they handle all possible scenarios and edge cases correctly, including potential failures in the underlying DEX interactions.
- Implement additional checks and error handling in the `_logicOpenPosition` and `_logicClosePosition` functions to prevent potential exploits or unexpected behaviour.
- Consider adding a mechanism to pause or restrict farm entries and exits in case of security incidents or vulnerabilities discovered in the integrated contracts or DEXes.

### 9. PendleLpOracle Contract
The PendleLpOracle contract provides price oracle functionality for Pendle LP tokens.

Recommendations:
- Implement robust error handling and fallback mechanisms in the `latestAnswer` function to handle scenarios where the underlying price feeds or oracles fail or provide invalid data.
- Add checks and input validation in the `_getLpToAssetRateWrapper` function to prevent potential manipulation or unexpected behaviour.
- Regularly monitor and assess the accuracy and reliability of the integrated price feeds and oracles.

### 10. PositionNFTs Contract
The PositionNFTs contract represents user positions as non-fungible tokens (NFTs) and provides functionality for managing these positions.

Recommendations:
- Implement strict access controls for functions like `assignReserveRole` and `forwardFeeManagerNFT` to ensure only authorized entities can modify critical parameters or transfer the fee manager NFT.
- Add checks and error handling in functions like `mintPosition` and `mintPositionForUser` to prevent potential misuse or unauthorized minting of position NFTs.
- Regularly review and update the NFT metadata and rendering logic to ensure compatibility with various NFT marketplaces and wallets.

### 11. WiseOracleHub Contract
The WiseOracleHub contract acts as an on-chain price oracle, providing token prices and heartbeat checks.

Recommendations:
- Implement robust error handling and fallback mechanisms in functions like `latestResolver` and `latestResolverTwap` to handle scenarios where the underlying price feeds or oracles fail or provide invalid data.
- Add checks and input validation in functions like `addOracle` and `addTwapOracle` to prevent potential manipulation or unexpected behaviour.
- Regularly monitor and assess the accuracy and reliability of the integrated price feeds and oracles.

### 12. MainHelper Contract
The MainHelper contract provides helper functions for the WiseLending contract, such as calculating shares, managing position data, and updating pool state.

Recommendations:
- Thoroughly test the share calculation functions like `calculateLendingShares` and `calculateBorrowShares` to ensure accurate results and prevent potential exploits.
- Implement input validation and error handling in functions like `_updateUtilization` and `_scalingAlgorithm` to handle potential edge cases or unexpected behaviour.
- Regularly review and optimize the helper functions to improve gas efficiency and readability.

### 13. WiseSecurityHelper Contract
The WiseSecurityHelper contract provides additional security-related functions and calculations used by the WiseSecurity contract.

Recommendations:
- Thoroughly test the calculation functions like `overallETHCollateralsBoth` and `overallETHBorrow` to ensure accurate results and prevent potential exploits.
- Implement input validation and error handling in functions like `_getPossibleWithdrawAmount` and `_distributeIncentives` to handle potential edge cases or unexpected behaviour.
- Regularly review and update the security helper functions to align with the evolving security requirements of the protocol.

### 14. OracleHelper Contract
The OracleHelper contract provides helper functions for interacting with price oracles and performing price-related calculations.

Recommendations:
- Implement robust error handling and fallback mechanisms in functions like `latestResolverTwap` and `_validateAnswer` to handle scenarios where the underlying price feeds or oracles fail or provide invalid data.
- Add checks and input validation in functions like `_addOracle` and `_addAggregator` to prevent potential manipulation or unexpected behaviour.
- Regularly monitor and assess the accuracy and reliability of the integrated price feeds and oracles.

### 15. FeeManagerHelper Contract
The FeeManagerHelper contract provides additional functions and calculations used by the FeeManager contract.

Recommendations:
- Implement input validation and error handling in functions like `_setBadDebtPosition` and `_updateUserBadDebt` to prevent potential exploits or unexpected behaviour.
- Thoroughly test the fee distribution and incentive calculation functions to ensure accurate results and prevent potential exploits.
- Regularly review and update the fee manager helper functions to align with the evolving requirements of the protocol.

### 16. AaveHelper Contract
The AaveHelper contract provides helper functions for interacting with the Aave lending protocol.

Recommendations:
- Thoroughly test the integration with Aave to ensure it handles all possible scenarios and edge cases correctly.
- Implement error handling and input validation in functions like `_wrapAaveReturnValueDeposit` and `_getInfoPayback` to handle potential failures or unexpected return values from Aave.
- Regularly monitor and assess the performance and security of the Aave integration to identify and mitigate potential risks.

### 17. PendlePowerFarmControllerHelper Contract
The PendlePowerFarmControllerHelper contract provides additional functions and calculations used by the PendlePowerFarmController contract.

Recommendations:
- Implement strict access controls and input validation in functions like `_findIndex` and `_setRewardTokens` to prevent unauthorized access or manipulation of critical data.
- Thoroughly test the reward token and index management functions to ensure accurate results and prevent potential exploits.
- Regularly review and update the controller helper functions to align with the evolving requirements of the Pendle Power Farm mechanism.

### 18. PendlePowerFarmMathLogic Contract
The PendlePowerFarmMathLogic contract provides mathematical functions and calculations used by the PendlePowerFarm contract.

Recommendations:
- Thoroughly test the mathematical functions like `getLeverageAmount` and `_getApproxNetAPY` to ensure accurate results and prevent potential exploits.
- Implement input validation and error handling in functions like `_getNewBorrowRate` and `_checkDebtRatio` to handle potential edge cases or unexpected behaviour.
- Regularly review and optimize the math logic functions to improve gas efficiency and readability.

### 19. PendlePowerFarmLeverageLogic Contract
The PendlePowerFarmLeverageLogic contract provides leverage-related functions and calculations used by the PendlePowerFarm contract.

Recommendations:
- Thoroughly test the leverage logic functions like `_executeBalancerFlashLoan` and `_logicOpenPosition` to ensure accurate results and prevent potential exploits.
- Implement input validation and error handling in functions like `_logicClosePosition` and `_getEthBack` to handle potential edge cases or unexpected behaviour.
- Regularly review and update the leverage logic functions to align with the evolving requirements of the Pendle Power Farm mechanism.

### 20. PendlePowerFarmDeclarations Contract
The PendlePowerFarmDeclarations contract provides declarations and constants used by the PendlePowerFarm contract.

Recommendations:
- Ensure that the declared constants and variables are appropriately set and aligned with the intended behaviour of the PendlePowerFarm contract.
- Regularly review and update the declarations to reflect any changes in the protocol's requirements or external dependencies.

### 21. PendlePowerManager Contract
The PendlePowerManager contract serves as the main entry point for managing the Pendle Power Farm mechanism.

Recommendations:
- Implement strict access controls for functions like `shutDownFarm` and `changeMinDeposit` to ensure only authorized entities can modify critical parameters.
- Thoroughly test the farm entry and exit functions to ensure they handle all possible scenarios and edge cases correctly, including potential failures in the underlying contracts.
- Regularly monitor and assess the performance and security of the Pendle Power Farm mechanism to identify and mitigate potential risks.

### 22. PowerFarmNFTs Contract
The PowerFarmNFTs contract represents the NFTs used in the Pendle Power Farm mechanism.

Recommendations:
- Implement strict access controls for functions like `setFarmContract` and `burnKey` to ensure only authorized entities can modify critical parameters or burn NFTs.
- Add checks and error handling in functions like `mintKey` and `walletOfOwner` to prevent potential misuse or unauthorized minting of NFTs.
- Regularly review and update the NFT metadata and rendering logic to ensure compatibility with various NFT marketplaces and wallets.

### 23. MinterReserver Contract
The MinterReserver contract handles the reservation and minting of Pendle Power Farm NFTs.

Recommendations:
- Implement strict access controls for functions like `_reserveKey` and `_mintKeyForUser` to ensure only authorized entities can reserve or mint NFTs.
- Add checks and error handling in functions like `mintReserved` and `onERC721Received` to prevent potential misuse or unauthorized minting of NFTs.
- Regularly review and update the minter reserver logic to align with the evolving requirements of the Pendle Power Farm mechanism.

### 24. PendleLpOracleLib Contract
The PendleLpOracleLib contract provides library functions for calculating LP token prices in the PendleLpOracle contract.

Recommendations:
- Thoroughly test the price calculation functions to ensure accurate results and prevent potential exploits.
- Implement input validation and error handling in functions like `getLpToAssetRate` to handle potential edge cases or unexpected behaviour.
- Regularly review and update the oracle library functions to align with the evolving requirements of the PendleLpOracle contract.

### 25. OwnableMaster Contract
The OwnableMaster contract provides ownership and access control functionality for the Wise Lending Protocol contracts.

Recommendations:
- Ensure that the ownership transfer process is secure and follows best practices to prevent unauthorized access.
- Consider implementing a multi-signature or time-locked ownership transfer mechanism to enhance security and prevent single points of failure.
- Regularly review and audit the access control logic to ensure it aligns with the protocol's security requirements.

### 26. TransferHelper Contract
The TransferHelper contract provides safe token transfer functionality used by various contracts in the Wise Lending Protocol.

Recommendations:
- Thoroughly test the token transfer functions to ensure they handle all possible scenarios and edge cases correctly, including potential failures or unexpected behaviour.
- Implement input validation and error handling in functions like `_safeTransfer` and `_safeTransferFrom` to prevent potential exploits or unexpected behaviour.
- Regularly review and update the transfer helper functions to align with the latest token standards and best practices.

### 27. ApprovalHelper Contract
The ApprovalHelper contract provides safe token approval functionality used by various contracts in the Wise Lending Protocol.

Recommendations:
- Thoroughly test the token approval functions to ensure they handle all possible scenarios and edge cases correctly, including potential failures or unexpected behaviour.
- Implement input validation and error handling in functions like `_safeApprove` to prevent potential exploits or unexpected behaviour.
- Regularly review and update the approval helper functions to align with the latest token standards and best practices.

### 28. SendValueHelper Contract
The SendValueHelper contract provides functionality for safely sending ETH used by various contracts in the Wise Lending Protocol.

Recommendations:
- Thoroughly test the ETH sending functions to ensure they handle all possible scenarios and edge cases correctly, including potential failures or unexpected behaviour.
- Implement input validation and error handling in functions like `_sendValue` to prevent potential exploits or unexpected behaviour.
- Regularly review and update the send value helper functions to align with the latest Ethereum best practices and security guidelines.

### 29. WiseCore Abstract Contract
The WiseCore abstract contract provides core functionality for the Wise Lending Protocol, including token transfers, liquidations, and pool management.

Recommendations:
- Thoroughly review and test the core functions to ensure they handle all possible scenarios and edge cases correctly, including potential edge cases and unexpected behaviour.
- Implement strict access controls and input validation in functions like `_preparationTokens` and `_coreLiquidation` to prevent unauthorized access or manipulation of critical data.
- Regularly review and update the core functions to align with the evolving requirements and security best practices of the Wise Lending Protocol.

### 30. PendlePowerFarm Abstract Contract
The PendlePowerFarm abstract contract provides core functionality for the Pendle Power Farm mechanism, including farm entry, exit, and leveraged position management.

Recommendations:
- Thoroughly review and test the farm entry and exit functions to ensure they handle all possible scenarios and edge cases correctly, including potential failures in the underlying contracts.
- Implement strict access controls and input validation in functions like `_manuallyPaybackShares` and `_manuallyWithdrawShares` to prevent unauthorized access or manipulation of user positions.
- Regularly review and update the Pendle Power Farm functions to align with the evolving requirements and security best practices of the mechanism.

### 31. PendlePowerFarmControllerBase Abstract Contract
The PendlePowerFarmControllerBase abstract contract provides base functionality for the PendlePowerFarmController contract, including Pendle token management and vote rewards claiming.

Recommendations:
- Implement strict access controls and input validation in functions like `proposeIncentiveMaster` and `claimOwnershipIncentiveMaster` to prevent unauthorized access or manipulation of critical parameters.
- Thoroughly test the token locking and vote reward claiming functions to ensure they handle all possible scenarios and edge cases correctly, including potential failures in the underlying contracts.
- Regularly review and update the controller base functions to align with the evolving requirements and security best practices of the Pendle Power Farm mechanism.

### 32. PendlePowerFarmDeclarations Abstract Contract
The PendlePowerFarmDeclarations abstract contract provides declarations and constants used by the PendlePowerFarm contract.

Recommendations:
- Ensure that the declared constants and variables are appropriately set and aligned with the intended behaviour of the PendlePowerFarm contract.
- Regularly review and update the declarations to reflect any changes in the protocol's requirements or external dependencies.

### 33. PendlePowerFarmMathLogic Abstract Contract
The PendlePowerFarmMathLogic abstract contract provides mathematical functions and calculations used by the PendlePowerFarm contract.

Recommendations:
- Thoroughly test the mathematical functions to ensure accurate results and prevent potential exploits.
- Implement input validation and error handling in functions like `_getNewBorrowRate` and `_checkDebtRatio` to handle potential edge cases or unexpected behaviour.
- Regularly review and optimize the math logic functions to improve gas efficiency and readability.

### 34. WiseLendingDeclaration Abstract Contract
The WiseLendingDeclaration abstract contract provides declarations and constants used by the WiseLending contract.

Recommendations:
- Ensure that the declared constants and variables are appropriately set and aligned with the intended behaviour of the WiseLending contract.
- Regularly review and update the declarations to reflect any changes in the protocol's requirements or external dependencies.

### 35. WiseSecurityHelper Abstract Contract
The WiseSecurityHelper abstract contract provides security-related functions and calculations used by the WiseSecurity contract.

Recommendations:
- Thoroughly test the security functions to ensure accurate results and prevent potential exploits.
- Implement input validation and error handling in functions like `_checkHealthState` and `_getState` to handle potential edge cases or unexpected behaviour.
- Regularly review and update the security helper functions to align with the evolving security requirements of the protocol.

### 36. WiseLowLevelHelper Abstract Contract
The WiseLowLevelHelper abstract contract provides low-level helper functions used by various contracts in the Wise Lending Protocol.

Recommendations:
- Thoroughly test the low-level helper functions to ensure they handle all possible scenarios and edge cases correctly, including potential failures or unexpected behaviour.
- Implement input validation and error handling in functions like `_validateParameter` and `_checkReentrancy` to prevent potential exploits or unexpected behaviour.
- Regularly review and update the low-level helper functions to align with the latest Ethereum best practices and security guidelines.

### 37. PtOracleDerivative Contract
The PtOracleDerivative contract is a price feed contract for PtToken with USD and ETH Chainlink feeds to get a feed measured in ETH. It combines the Chainlink oracle values with the corresponding TWAP and other Chainlink oracles.

Recommendations:
- Thoroughly test the `latestAnswer` function to ensure accurate price calculations and handle potential edge cases or failures in the underlying price feeds.
- Consider adding additional checks and error handling in the `latestRoundData` function to handle scenarios where the price feeds fail or provide invalid data.
- Regularly monitor and assess the accuracy and reliability of the integrated price feeds and oracles to ensure the integrity of the PtToken pricing.

### 38. PtOraclePure Contract
The PtOraclePure contract is a price feed contract for PtToken with a USD Chainlink feed to get a feed measured in ETH. It takes the Chainlink oracle value and multiplies it with the corresponding TWAP and other Chainlink oracles.

Recommendations:
- Thoroughly test the `latestAnswer` function to ensure accurate price calculations and handle potential edge cases or failures in the underlying price feeds.
- Consider adding additional checks and error handling in the `latestRoundData` function to handle scenarios where the price feeds fail or provide invalid data.
- Regularly monitor and assess the accuracy and reliability of the integrated price feeds and oracles to ensure the integrity of the PtToken pricing.

### 39. PendlePowerFarmTokenFactory Contract
The PendlePowerFarmTokenFactory contract is responsible for deploying new instances of the PendlePowerFarmToken contract. It ensures that only the authorized Pendle Power Farm Controller can deploy new tokens.

Recommendations:
- Ensure that the `PENDLE_POWER_FARM_CONTROLLER` address is set correctly and securely to prevent unauthorized deployment of tokens.
- Consider implementing additional access controls and input validation in the `deploy` function to prevent potential misuse or unauthorized token deployments.
- Regularly review and update the factory contract to align with the evolving requirements and security best practices of the Pendle Power Farm mechanism.

### 40. PendleChildLpOracle Contract
The PendleChildLpOracle contract provides price oracle functionality for Pendle Child LP tokens. It calculates the price based on the underlying Pendle LP oracle and the total assets and supply of the Pendle Child token.

Recommendations:
- Thoroughly test the `latestAnswer` function to ensure accurate price calculations and handle potential edge cases or failures in the underlying price feeds and token balances.
- Consider adding additional checks and error handling in the `latestRoundData` and `getRoundData` functions to handle scenarios where the price feeds fail or provide invalid data.
- Regularly monitor and assess the accuracy and reliability of the integrated price feeds and token balances to ensure the integrity of the Pendle Child LP token pricing.

### 41. FeeManagerEvents Contract
The FeeManagerEvents contract defines various events related to fee management, incentives, and bad debt tracking in the Wise Lending Protocol.

Recommendations:
- Ensure that all relevant actions and state changes in the FeeManager contract emit the appropriate events defined in this contract.
- Consider adding more granular events to provide better transparency and auditability of fee-related actions and changes.
- Regularly review and update the event definitions to align with the evolving requirements and functionalities of the FeeManager contract.

### 42. CustomOracleSetup Contract
The CustomOracleSetup contract provides a customizable setup for oracle-related functionalities, allowing the contract owner to set and update various oracle parameters.

Recommendations:
- Implement strict access controls for the `onlyOwner` functions to ensure that only authorized entities can modify critical oracle parameters.
- Consider adding input validation and error handling in functions like `setLastUpdateGlobal`, `setRoundData`, and `setGlobalAggregatorRoundId` to prevent potential misuse or invalid data settings.
- Regularly review and update the custom oracle setup to align with the evolving requirements and security best practices of the oracle integration.

### 43. SendValueHelper Contract
The SendValueHelper contract provides a helper function for safely sending ETH to a recipient address, handling potential errors and reverting on failures.

Recommendations:
- Thoroughly test the `_sendValue` function to ensure it handles all possible scenarios and edge cases correctly, including potential failures or unexpected behaviour.
- Consider implementing additional checks and error handling to prevent potential exploits or unexpected behaviour.
- Regularly review and update the send value helper function to align with the latest Ethereum best practices and security guidelines.

### 44. WrapperHelper Contract
The WrapperHelper contract provides helper functions for wrapping and unwrapping ETH using the WETH (Wrapped ETH) contract.

Recommendations:
- Ensure that the `_wethAddress` passed to the constructor is a valid and trusted WETH contract address to prevent potential vulnerabilities or incompatibilities.
- Thoroughly test the `_wrapETH` and `_unwrapETH` functions to ensure they handle all possible scenarios and edge cases correctly, including potential failures or unexpected behaviour.
- Regularly review and update the wrapper helper functions to align with the latest WETH standards and best practices.

### 45. CallOptionalReturn Contract
The CallOptionalReturn contract provides a helper function for performing low-level calls to external contracts and handling the return data.

Recommendations:
- Thoroughly test the `_callOptionalReturn` function to ensure it handles all possible scenarios and edge cases correctly, including potential failures or unexpected behaviour from the external contract.
- Consider implementing additional checks and error handling to prevent potential exploits or unexpected behaviour.
- Regularly review and update the call optional return function to align with the latest Ethereum best practices and security guidelines.

### 46. TransferHelper Contract
The TransferHelper contract provides helper functions for safely transferring ERC20 tokens, handling potential errors and reverting on failures.

Recommendations:
- Thoroughly test the `_safeTransfer` and `_safeTransferFrom` functions to ensure they handle all possible scenarios and edge cases correctly, including potential failures or unexpected behaviour from the token contracts.
- Consider implementing additional checks and error handling to prevent potential exploits or unexpected behaviour.
- Regularly review and update the transfer helper functions to align with the latest ERC20 token standards and best practices.

### 47. AaveEvents Contract
The AaveEvents contract defines various events related to Aave interactions in the Wise Lending Protocol, such as deposits, withdrawals, borrows, and paybacks.

Recommendations:
- Ensure that all relevant Aave-related actions and state changes in the AaveHub contract emit the appropriate events defined in this contract.
- Consider adding more granular events to provide better transparency and auditability of Aave-related actions and changes.
- Regularly review and update the event definitions to align with the evolving requirements and functionalities of the AaveHub contract.

### 48. ApprovalHelper Contract
The ApprovalHelper contract provides a helper function for safely approving ERC20 token allowances, handling potential errors and reverting on failures.

Recommendations:
- Thoroughly test the `_safeApprove` function to ensure it handles all possible scenarios and edge cases correctly, including potential failures or unexpected behaviour from the token contracts.
- Consider implementing additional checks and error handling to prevent potential exploits or unexpected behaviour.
- Regularly review and update the approval helper function to align with the latest ERC20 token standards and best practices.

#### Conclusion:

The [Wise Lending Protocol](https://github.com/code-423n4/2024-02-wise-lending) is a sophisticated DeFi ecosystem that offers collateralized lending and borrowing functionalities with dynamic interest rates and automated scaling mechanisms. The protocol's codebase demonstrates good practices in terms of modularity, access control, and error handling. However, the complexity of the system and its reliance on external dependencies introduce potential risks and challenges.

To enhance the security and reliability of the protocol, it is crucial to prioritize the identified recommendations based on their criticality and potential impact. The protocol should focus on addressing the most critical vulnerabilities and areas of concern, such as reentrancy risks, improper access controls, unsafe token transfers, and incorrect calculations.

Additionally, the protocol should establish a robust governance mechanism to manage critical protocol parameters, respond swiftly to security incidents, and align with the evolving requirements of the ecosystem. Regular audits, security assessments, and ongoing monitoring of the integrated external contracts and dependencies are essential to identify and mitigate potential risks.

By addressing the specific recommendations provided for each contract, prioritizing critical vulnerabilities, and continuously improving the protocol's security posture, the [Wise Lending Protocol](https://github.com/code-423n4/2024-02-wise-lending) can establish itself as a secure and trustworthy decentralized lending and borrowing platform.

Use [Tenderly](https://dashboard.tenderly.co/) and [Defender](defender.openzeppelin.com) for continued monitoring to prevent unforeseen risks or actions, and reference Security Alliance's [SEAL-911](https://securityalliance.org/) to help with trust and incident response.

With a focus on security, transparency, and continuous improvement, the [Wise Lending Protocol](https://github.com/code-423n4/2024-02-wise-lending) can position itself as a leading decentralized lending and borrowing platform in the DeFi ecosystem.

### Time spent:
42 hours