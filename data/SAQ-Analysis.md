## Summary

no | File |
|-|:-|
| [[File-1](#file-1)] | WiseLending.sol |
| [[File-2](#file-2)] | WiseSecurity.sol | 
| [[File-3](#file-3)] | MainHelper.sol | 
| [[File-4](#file-4)] | WiseSecurityHelper.sol | 
| [[File-5](#file-5)] | PendlePowerFarmToken.sol | 
| [[File-6](#file-6)] | FeeManager.sol | 
| [[File-7](#file-7)] | PendlePowerFarmLeverageLogic.sol | 
| [[File-8](#file-8)] | OracleHelper.sol | 
| [[File-9](#file-9)] | PendlePowerFarmController.sol | 
| [[File-10](#file-10)] | WiseCore.sol | 
| [[File-11](#file-11)] | WiseLendingDeclaration.sol | 
| [[File-12](#file-12)] | WiseOracleHub.sol | 
| [[File-13](#file-13)] | AaveHub.sol | 
| [[File-14](#file-14)] | WiseLowLevelHelper.sol | 
| [[File-15](#file-15)] | PendlePowerFarmDeclarations.sol | 
| [[File-16](#file-16)] | PendlePowerFarmControllerBase.sol | 
| [[File-17](#file-17)] | AaveHelper.sol | 
| [[File-18](#file-18)] | PositionNFTs.sol | 
| [[File-19](#file-19)] | WiseSecurityDeclarations.sol | 
| [[File-20](#file-20)] | PoolManager.sol | 
| [[File-21](#file-21)] | FeeManagerHelper.sol | 
| [[File-22](#file-22)] | PendlePowerFarmMathLogic.sol | 
| [[File-23](#file-23)] | DeclarationsFeeManager.sol | 
| [[File-24](#file-24)] | PendlePowerManager.sol | 
| [[File-25](#file-25)] | PendlePowerFarmControllerHelper.sol | 
| [[File-26](#file-26)] | PendlePowerFarm.sol | 
| [[File-27](#file-27)] | PowerFarmNFTs.sol | 
| [[File-28](#file-28)] | MinterReserver.sol | 
| [[File-29](#file-29)] | PendleLpOracle.sol | 
| [[File-30](#file-30)] | Declarations.sol | 
| [[File-31](#file-31)] | PtOracleDerivative.sol | 
| [[File-32](#file-32)] | Declarations.sol | 
| [[File-33](#file-33)] | PtOraclePure.sol | 
| [[File-34](#file-34)] | OwnableMaster.sol | 
| [[File-35](#file-35)] | PendlePowerFarmTokenFactory.sol | 
| [[File-36](#file-36)] | PendleChildLpOracle.sol | 
| [[File-37](#file-37)] | FeeManagerEvents.sol | 
| [[File-38](#file-38)] | CustomOracleSetup.sol | 
| [[File-39](#file-39)] | SendValueHelper.sol | 
| [[File-40](#file-40)] | WrapperHelper.sol | 
| [[File-41](#file-41)] | CallOptionalReturn.sol | 
| [[File-42](#file-42)] | TransferHelper.sol | 
| [[File-43](#file-43)] | AaveEvents.sol | 
| [[File-44](#file-44)] | ApprovalHelper.sol | 


## Analysis Issue Report 


### [File-1] WiseLending.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The code contains functions that are restricted to certain roles, such as `onlyAaveHub` and `onlyFeeManager`, which suggests some level of centralized control.

* The use of privileged roles like `onlyAaveHub` and `onlyFeeManager` introduces the risk of potential admin abuse if these roles are not managed securely.

* The `setRegistrationIsolationPool` function allows the owner to register positions for isolation pool functionality, potentially giving them control over isolation pool operations.


#### Systemic Risks:

* The platform's reliance on external contracts, such as ``POSITION_NFT`` and `WISE_SECURITY`, may introduce systemic risks if these contracts have vulnerabilities or if their behavior changes unexpectedly.
* The liquidation logic relies on external factors, such as `WISE_SECURITY.checksLiquidation`, which could introduce risks if the external checks are not robust.


#### Technical Risks:

* The code includes numerous mathematical operations, such as arithmetic calculations and unchecked arithmetic, which can introduce risks if not handled carefully.
* The usage of the `_validateNonZero` and `_validateZero` functions suggests checks for non-zero or zero values, reducing the risk of unintended behavior due to invalid inputs.
* There is a reliance on the `_unwrapETH` and `_wrapETH` functions for handling ETH deposits and withdrawals, which must be implemented securely to avoid vulnerabilities.



#### Integration Risks:

* The integration of external contracts and reliance on certain assumptions about their behavior introduces integration risks. Any changes to these external contracts could impact the functionality of the WISE lending platform.



#### Non-Standard Token Risks:

* The code interacts with ERC-20 tokens, especially in functions related to deposits, withdrawals, and borrowings. Risks related to non-standard behavior of ERC-20 tokens, such as reentrancy vulnerabilities or non-compliant implementations, should be considered.


#### Overall Summary:

The code appears to implement a lending platform with features for depositing, withdrawing, borrowing, and liquidating positions. While it includes measures to mitigate certain risks, such as role-based access control and input validations, careful consideration should be given to potential vulnerabilities related to external contract interactions, mathematical operations, and ETH handling.

</details>


### [File-2] WiseSecurity.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract has a `onlyMaster` modifier, allowing certain functions to be executed only by the master. While this can provide necessary access control, it also introduces the risk of admin abuse if the master is compromised or acts maliciously.



#### Systemic Risks:

* The contract relies on external contracts such as `WISE_LENDING`, `WISE_ORACLE`, and `FEE_MANAGER`. Systemic risks may arise if these contracts have vulnerabilities or if their behavior changes unexpectedly.


#### Technical Risks:

* The contract uses the `error` statement for custom error handling, which is a new feature introduced in Solidity 0.8. However, it's essential to ensure that error handling is robust and does not introduce vulnerabilities.
* The usage of unchecked arithmetic operations may pose risks if not handled carefully. Careful consideration is needed to avoid arithmetic overflow or underflow.


#### Integration Risks:

* The contract integrates with other contracts, such as `WiseSecurityHelper` and `ApprovalHelper`. Integration risks may arise if there are vulnerabilities in these external contracts.


#### Non-Standard Token Risks:

* The contract interacts with ERC-20 tokens, especially in functions related to borrowing, lending, and collateral. Risks related to non-standard behavior of ERC-20 tokens, such as reentrancy vulnerabilities or non-compliant implementations, should be considered.


#### Overall Summary:

The contract appears to be a core component of the WISE lending system, implementing various security checks for withdrawals, borrows, paybacks, and liquidations. It relies on external contracts and has role-based access controls, introducing potential risks associated with external contract interactions and admin privileges.

</details>

### [File-3] MainHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* **Ownership and Access Control**: The smart contract does not seem to have explicit access control mechanisms or ownership restrictions. This lack of access control might pose a risk if not managed properly, allowing potential admin abuse.


#### Systemic Risks:

* **Reentrancy Guard**: The contract has a reentrancy guard, which is a good practice to prevent reentrancy attacks. However, the effectiveness of this guard depends on its placement and implementation in the entire system.


#### Technical Risks:

* **Security Functions**: The contract uses various internal functions to perform calculations, update state variables, and manage positions. The complexity of these functions might introduce bugs if not thoroughly tested


#### Integration Risks:

* **External Contract Interactions**: The smart contract interacts with other contracts, such as WiseLowLevelHelper and external interfaces like `IERC20`. Risks may arise from vulnerabilities in these external contracts.


#### Non-Standard Token Risks:

* **Custom Token Handling**: The contract deals with custom tokens as identified by the `_poolToken` parameter. Risks may arise if non-standard tokens are not handled properly or if the contract is not compatible with certain token standards.

#### Overall Summary:

The provided smart contract is an abstract contract named `MainHelper`, utilizing functions from `WiseLowLevelHelper`. It includes functionality related to lending and borrowing with a focus on calculating _shares_, _amounts_, and _rates_. 


</details>

### [File-4] WiseSecurityHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* **Not Applicable**: The smart contract does not have explicit admin-related functions or access control mechanisms. Therefore, the risk of admin abuse is not applicable in this case.



#### Systemic Risks:

* **Reentrancy Guard**: The contract incorporates a reentrancy guard in the form of the `unchecked` keyword in several loops, which helps prevent reentrancy attacks. This mitigates potential systemic risks associated with reentrancy vulnerabilities.

#### Technical Risks:

* **Complexity of Internal Functions**: The contract contains various internal functions related to collateral, borrowing, lending, and liquidation calculations. The complexity of these functions may introduce technical risks, such as computational inefficiencies or unintended behaviors. Thorough testing and code review are essential to address these potential risks.


#### Integration Risks:

* **Dependency on External Contracts**: The smart contract interacts with external contracts, such as `WISE_LENDING`, `WISE_ORACLE`, and `FEE_MANAGER`. Dependency on external contracts introduces integration risks, including vulnerabilities in the external contracts, changes in their interfaces, or reliance on their correct behavior. It is crucial to ensure compatibility and security of these dependencies.


#### Non-Standard Token Risks:

* **Handling Non-standard Tokens**: The contract interacts with custom tokens identified by `_poolToken`. Handling non-standard tokens may introduce risks related to the behavior of these tokens or potential vulnerabilities specific to them. Proper validation and testing with various token types are essential to address non-standard token risks.


#### Overall:

While the contract includes a reentrancy guard and functionality related to *lending*, *borrowing*, and *collateral* management, a comprehensive audit by security experts is recommended to identify and mitigate potential vulnerabilities. Additionally, careful consideration of external dependencies and thorough testing with various scenarios are crucial before deploying the contract in a production environment.

</details>

### [File-5] PendlePowerFarmToken.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract employs an `onlyController` modifier to restrict certain functions to be callable only by `PENDLE_POWER_FARM_CONTROLLER`, mitigating admin abuse risks to some extent.
* The `initialize` function checks if the contract has already been initialized, preventing potential reinitialization and unauthorized changes.


#### Systemic Risks:

* The `syncSupply` modifier is implemented to synchronize various internal state variables and update rewards. It aims to maintain consistency and prevent potential systemic issues.
* The contract incorporates checks related to the share price growth, ensuring that it doesn't grow excessively over time.


#### Technical Risks:

* The contract uses various mathematical calculations, especially in functions like `previewMintShares`, `previewAmountWithdrawShares`, and `previewBurnShares`. Careful review and testing are needed to ensure these calculations are correct and don't introduce vulnerabilities.
* The `syncSupply` modifier and associated functions introduce complexity; rigorous testing is required to ensure proper functionality.


#### Integration Risks:

* The contract relies on external contracts/interfaces such as `IPendle`, `IPendleController`, `IPendleMarket`, and `IPendleSy`. Integration risks arise from the dependencies on these external contracts, and any changes in their interfaces or behavior may impact the functioning of this contract. Compatibility checks and validation of external interactions are essential.


#### Non-Standard Token Risks:

* The contract works with an underlying LP token (`UNDERLYING_PENDLE_MARKET`). Risks related to the behavior of this token, especially if it's a non-standard token, should be considered and thoroughly tested.


#### Recommendations:

* Conduct thorough testing, especially focusing on mathematical calculations, to ensure correctness.
* Perform extensive testing with various scenarios, including edge cases and extreme values.
* Review and validate external contract interactions for compatibility and security.
* Consider adding detailed comments to improve the code's readability and maintainability.
* If possible, conduct a security audit to identify and address potential vulnerabilities.

</details>

### [File-6] FeeManager.sol
 
#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* **Admin-only Functions**: The contract has several functions restricted to the `onlyMaster` modifier, such as adjusting pool fees, proposing and claiming ownership of the incentive master, setting Aave flags, and more. Admins have significant control over various aspects of the contract, which could pose a risk if not managed properly.


#### Systemic Risks:

* **Dependency on External Contracts**: The contract relies on external contracts, such as `FeeManagerHelper`, `WISE_LENDING`, `AAVE`, etc. Any vulnerabilities or issues in these external contracts may impact the overall functionality and security of this contract.


#### Technical Risks:

1. **Unchecked Arithmetic Operations**: The contract uses unchecked arithmetic operations, such as in the `setPoolFeeBulk` and other loop-based functions. This might lead to unexpected behavior or vulnerabilities if not handled carefully.
2. **Conditional and Unconditional Reverts**: The contract uses reverts with conditions and without conditions. Proper handling and informative error messages are crucial for security and user experience.


#### Integration Risks:

1. **External Contract Integration**: The contract interacts with external contracts, and if any of these contracts change or become incompatible, it may affect the functionality of this contract.
2. **Integration with Aave**: The contract interacts with Aave contracts for token withdrawals. Any issues with Aave contracts might impact the reliability of this contract.


#### Non-Standard Token Risks:

* **Use of Custom Tokens**: The contract seems to interact with custom tokens (WISE tokens) and could be exposed to risks associated with these tokens. A thorough audit of these tokens and their functionalities is recommended.


#### Overall Summary:

The `FeeManager` smart contract facilitates fee distribution from the *wiseLending* protocol, *managing* fees acquired in the form of shares from various pools. Admins can adjust pool fees, set Aave flags, and propose new incentive masters for ecosystem bootstrapping. The contract also addresses bad debts, incentivizing their repayment through gathered fees. However, potential risks include admin-centric control, dependency on external contracts, unchecked arithmetic operations, and integration risks with Aave.

</details>

### [File-7] PendlePowerFarmLeverageLogic.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* No explicit admin functions are visible in the provided code, which reduces the risk of admin abuse. However, the inherited contracts (`PendlePowerFarmMathLogic` and `IFlashLoanRecipient`) should be reviewed to ensure that no admin-sensitive functionalities are introduced there.


#### Systemic Risks:

* The contract leverages flash loans from Balancer (`BALANCER_VAULT`) and interacts with external protocols like Aave (`AAVE_HUB`), Wise Lending (`WISE_LENDING`), and PendleSwap (`PENDLE_ROUTER`, `PENDLE_MARKET`, `PENDLE_CHILD`). Systemic risks include vulnerabilities in these external protocols, and the contract's functionality may depend heavily on the correctness and security of these external components.

#### Technical Risks:

* **Arithmetic Operation**s: There are arithmetic operations throughout the code, and careful attention should be paid to potential overflow, underflow, or precision errors.
* **Reentrancy**: Although not explicitly visible in the provided code, any external calls should be reviewed to ensure that reentrancy vulnerabilities are mitigated.
* **Dependency on External Contracts**: The contract relies on various external contracts, and any changes in their interfaces or implementations may affect the functioning of this contract.


#### Integration Risks:

* **Dependency on External Contracts**: The contract interacts with external contracts like Balancer, Aave, Wise Lending, and PendleSwap. Integration risks include potential changes in these external contracts, including upgrades or modifications that might affect the expected behavior of the current contract.


#### Non-Standard Token Risks:

* The code does not appear to involve non-standard tokens explicitly. However, interactions with external contracts might introduce non-standard tokens, and the handling of these tokens should be carefully reviewed.


</details>

### [File-8] OracleHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract does not seem to have explicit admin functions that could be prone to abuse.
* No clear evidence of admin privileges or potential misuse of power.



#### Systemic Risks:

* The contract involves oracles and relies on external price feeds for various tokens.
* The `_recalibrate` function may introduce latency if it's called frequently, depending on the number of iterations.
* The `sequencerIsDead` function checks if a sequencer is working but does not seem to be directly related to the contract's core functionality.


#### Technical Risks:

* The contract interacts with various external contracts and oracles, introducing dependencies and potential vulnerabilities in those external systems.
* The usage of assembly in `_checkFunctionExistence` might pose risks if not handled carefully.


#### Integration Risks:

* The contract seems to integrate with external systems, particularly Chainlink oracles and Uniswap V3 pools.
* The use of oracles introduces the risk of inaccurate or manipulated data affecting the contract's functionality.
* Uniswap V3 integration may have risks related to pool interactions and price calculations.


#### Non-Standard Token Risks:

* The contract interacts with tokens through oracles, but it doesn't directly handle custom or non-standard tokens.


</details>

### [File-9] PendlePowerFarmController.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The `onlyMaster` and `onlyChildContract` modifiers restrict certain functions to be accessible only by the master or child contracts. These modifiers can be considered as access control mechanisms to prevent potential admin abuse.
* The `PendlePowerFarmController` contract inherits from `PendlePowerFarmControllerHelper`, and it initializes several contract addresses in the constructor. The access to these addresses could be a potential risk if not handled carefully.


#### Systemic Risks:

* The contract deals with locking Pendle tokens, claiming rewards, and interacting with various other contracts, which introduces systemic risks related to the proper functioning of these external components.
* The use of modifiers and checks in functions aims to ensure that certain conditions are met before executing critical operations.

#### Technical Risks:

* The contract interacts with several external contracts and relies on correct functioning of these contracts, including `PendlePowerFarmTokenFactory`, `PendlePowerFarmControllerHelper`, and others. Any issues in these external contracts could impact the functionality of this contract.
* The use of modifiers for access control relies on the correctness of the implementation and the security of modifier implementations.


#### Integration Risks:

* The contract integrates with other Pendle contracts, such as `PENDLE_LOCK`, `PENDLE_VOTE_REWARDS`, `PENDLE_VOTER`, and `PENDLE_TOKEN_ADDRESS`. The correctness and security of these integrations are crucial for the overall system.
* The contract manages Pendle market addresses and corresponding child contracts, introducing potential risks related to proper initialization and updates.


#### Non-Standard Token Risks:

* The contract interacts with the Pendle token (`PENDLE_TOKEN_ADDRESS`), but it doesn't seem to handle custom or non-standard tokens directly.


</details>

### [File-10] WiseCore.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol)

<details>
<summary>see instances</summary>




#### Admin Abuse Risks:

* The contract uses various internal functions, and some are marked as `internal`, limiting external access. Access control mechanisms might be enforced through the usage of the `onlyMaster` modifier in certain functions. However, a more detailed analysis is required to assess the potential for admin abuse comprehensively.



#### Systemic Risks:

* The contract involves complex logic for handling deposits, withdrawals, borrowing, and liquidations. Any issues in these processes could lead to systemic risks in the functioning of the contract.


#### Technical Risks:

* The contract inherits from `MainHelper` and `TransferHelper`, suggesting dependencies on external functionality. The correctness and security of these dependencies are crucial for the overall stability of the contract.
* The usage of various storage variables and mappings implies a need for careful state management to avoid vulnerabilities such as reentrancy attacks or state manipulation.


#### Integration Risks:

* The contract interacts with external contracts, including `WISE_SECURITY`, `WISE_ORACLE`, and `POSITION_NFT`. The correct integration and security of these external contracts are essential to prevent potential risks.


#### Non-Standard Token Risks:

* The contract handles various tokens, but it doesn't seem to deal explicitly with non-standard tokens. The interactions with tokens are based on the common ERC-20 standard.


</details>

### [File-11] WiseLendingDeclaration.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* Admin abuse risks are not clearly defined in the contract. The contract relies on access control through modifiers such as `onlyMaster`, but a comprehensive analysis would require a closer look at the roles and permissions assigned to the contract owner and other potential administrators.



#### Systemic Risks:

* The contract involves complex logic related to lending, borrowing, liquidation, and fee management. Any issues or vulnerabilities in these processes could pose systemic risks to the overall functionality and security of the contract.


#### Technical Risks:

* The contract has dependencies on various external contracts and interfaces, such as `AAVE_HUB_ADDRESS`, `WISE_SECURITY`, `FEE_MANAGER`, `POSITION_NFT`, and `WISE_ORACLE`. The correctness and security of these dependencies are critical for the proper functioning of the contract.
* The usage of various mappings and storage variables requires careful state management to avoid vulnerabilities such as reentrancy attacks or state manipulation.


#### Integration Risks:

* The contract integrates with external contracts and interfaces, including Aave Hub, Wise Security, Fee Manager, Position NFTs, and Wise Oracle Hub. The correct integration and security of these external contracts are essential to prevent potential risks.
* The contract relies on external interfaces for critical functions, and any changes or upgrades to these interfaces may introduce integration risks.


#### Non-Standard Token Risks:

* The contract primarily deals with standard ERC-20 tokens, and there is no explicit handling of non-standard tokens. However, external contracts such as Aave Hub might introduce non-standard tokens, and their correct integration and handling should be ensured.


</details>



### [File-12] WiseOracleHub.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract implements an access control mechanism with the `onlyMaster` modifier to restrict certain functions to the master address. However, the roles and permissions of the master address and potential admin abuse risks are not explicitly documented.
* It is essential to thoroughly review the roles and permissions of the master address and ensure that it cannot be manipulated for malicious purposes.



#### Systemic Risks:

* The contract deals with oracles and price feeds, which are critical components for decentralized applications relying on accurate and timely pricing information. Any vulnerabilities in the oracle system could have systemic risks for applications relying on this oracle hub.


#### Technical Risks:

* The contract has dependencies on external contracts and interfaces, such as the `OracleHelper` and `IPriceFeed`. The correctness and security of these dependencies need to be ensured.
* The contract contains several functions related to oracles, price feeds, and their integration, making it crucial to conduct thorough testing to identify and address any technical risks.


#### Integration Risks:

* The contract integrates with external contracts, including the `OracleHelper` and `IPriceFeed`. The correct integration and security of these external contracts are essential to prevent potential risks.
* The contract relies on accurate and timely information from external oracles, and any issues with the integration or functionality of these oracles could pose integration risks.


#### Non-Standard Token Risks:

* The contract primarily deals with standard ERC-20 tokens and does not explicitly handle non-standard tokens. However, it is essential to ensure that the oracles and price feeds used support a wide range of tokens, including potential non-standard tokens.

</details>



### [File-13] AaveHub.sol
 
#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract implements an access control mechanism with the `onlyMaster` modifier to restrict certain functions to the master address. However, the roles and permissions of the master address and potential admin abuse risks are not explicitly documented.
* It is crucial to thoroughly review the roles and permissions of the master address and ensure that it cannot be manipulated for malicious purposes.
* Functions like `setAaveTokenAddress` and `skimAave` are restricted to the master, and it's essential to assess potential risks associated with these functions.



#### Systemic Risks:

* The contract interacts with the Aave protocol, which involves handling borrowed and supplied funds. Systemic risks may arise if there are vulnerabilities in the Aave protocol, or if there are unexpected changes in the behavior of Aave's smart contracts.
* The reliance on external contracts, such as `WISE_LENDING`, `WISE_SECURITY`, and `Aave-related` contracts, introduces systemic risks that need careful consideration and potentially third-party audits.


#### Technical Risks:

* The contract has dependencies on external contracts, including `AaveHelper`, `TransferHelper`, and `ApprovalHelper`. The correctness and security of these dependencies need to be ensured.
* The interaction with Aave's smart contracts involves complex financial operations, and thorough testing is necessary to identify and address any technical risks.


#### Integration Risks:

* The contract integrates with the Aave protocol, and any issues with the integration or functionality of Aave's smart contracts could pose integration risks.
* The `setAaveTokenAddress` functions link underlying assets with corresponding Aave tokens, and any errors in these mappings could lead to unexpected behavior or financial loss.


#### Non-Standard Token Risks:

* The contract primarily deals with ERC-20 tokens and Ether. It's important to ensure that the Aave protocol and the contract's functions support a wide range of ERC-20 tokens and handle them correctly.
* Verify that Aave's smart contracts can handle potential non-standard tokens used as collateral or borrowed assets.


</details>




### [File-14] WiseLowLevelHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract has an `onlyFeeManager` modifier, restricting access to certain functions to the address defined as `FEE_MANAGER`. However, the roles and permissions of the fee manager and potential admin abuse risks are not explicitly documented.
* It is crucial to thoroughly review the roles and permissions of the fee manager address and ensure that it cannot be manipulated for malicious purposes.
* The `setPoolFee` function is restricted to the fee manager, and potential risks associated with this function should be assessed.



#### Systemic Risks:

* The contract interacts with various data structures, including lending pools, borrow pools, and user data. Any vulnerabilities in the data structures or unexpected changes in their behavior could pose systemic risks.
* The contract depends on external declarations and data from other contracts, such as `WiseLendingDeclaration`, and the correctness of these dependencies needs to be ensured.
* The reliance on external contracts and interfaces, such as `IAaveHubLite`, introduces systemic risks that need careful consideration and potentially third-party audits.


#### Technical Risks:

* The contract implements various internal set and get functions to manage data related to lending and borrowing pools. Thorough testing is necessary to identify and address any technical risks associated with these functions.
* The contract uses modifiers and internal functions for access control and validation, and it's essential to ensure that these mechanisms work as intended and cannot be exploited.


#### Integration Risks:

* The contract interacts with external contracts, such as `AAVE_HUB_ADDRESS`, and relies on their correct behavior. Any issues with the integration or functionality of these external contracts could pose integration risks.
* The `setPoolFee` function modifies global pool data, and the impact of such changes on the overall system should be carefully considered.


#### Non-Standard Token Risks:

* The contract does not explicitly handle ERC-20 tokens or Ether transactions, and it seems to be more focused on managing lending and borrowing pool data. Therefore, non-standard token risks may not be applicable in this context.


</details>




### [File-15] PendlePowerFarmDeclarations.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract defines several adjustable parameters such as `isShutdown`, `allowEnter`, `collateralFactor`, `minDepositEthAmount`, and various addresses. Administrators may have the ability to modify these parameters, and the potential misuse or unintended changes could lead to adverse effects.
* There's a need for clear documentation specifying the roles and responsibilities of administrators, as well as guidelines on how and when to adjust these parameters to avoid potential abuse.



#### Systemic Risks:

* The contract relies on various external contracts and interfaces, including Aave, Pendle, WiseLending, OracleHub, BalancerVault, and others. Any vulnerabilities or unexpected behavior in these external contracts may result in systemic risks.
* The contract interacts with external systems for flashloans, and the proper functioning and security of these systems are crucial to prevent systemic failures.
* The contract uses a modifier (`isActive`) to check if the contract is active. However, the behavior of this modifier and how it affects the contract's functionality should be thoroughly assessed.


#### Technical Risks:

* The contract contains complex logic, including flashloan functionality, leverage calculations, and interactions with external protocols. Thorough testing and auditing are essential to identify and mitigate potential technical risks.
* The reliance on external interfaces, such as Aave and Pendle, introduces technical risks related to the correctness of the implementations and the potential for unforeseen vulnerabilities.


#### Integration Risks:

* The contract integrates with multiple external protocols, including Aave, Pendle, and others. Changes in the behavior of these protocols or unforeseen issues in the integration process could result in integration risks.
* The contract relies on external addresses and interfaces, and any changes in these addresses or the introduction of new versions could lead to integration issues.


#### Non-Standard Token Risks:

* The contract handles various tokens, including ERC-20 tokens and Ether. Potential risks related to non-standard tokens include transfer and approval mechanisms, as well as interactions with external contracts that may introduce vulnerabilities.


</details>




### [File-16] PendlePowerFarmControllerBase.so

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract inherits from `OwnableMaster`, implying that the owner has significant control over the contract. Potential risks include unauthorized modifications, withdrawals, or adjustments to critical parameters.
* The `exchangeIncentive` can be changed by the owner, and if misused, it might impact the behavior of the contract.



#### Systemic Risks:

* The contract relies on various external contracts and interfaces, such as Pendle, WiseOracleHub, PendleLock, PendleVoter, and others. Any vulnerabilities or unexpected behavior in these external contracts may lead to systemic risks.
* The contract handles multiple events, and proper handling and logging are crucial for monitoring and auditing. Inadequate event logging may impact the ability to trace and investigate potential issues.


#### Technical Risks:

* The contract uses modifiers and functions related to specific conditions, like `_syncSupply`, `_onlyChildContract`, and `_onlyArbitrum`. Proper testing and understanding of these conditions are essential to ensure the correct behavior of the contract.
* The contract interacts with external systems, including Pendle, WiseOracleHub, and ArbRewardsClaimer. Potential issues related to these interactions, such as reentrancy or unexpected behavior, need careful consideration.


#### Integration Risks:

* The contract integrates with multiple external protocols, and changes in the behavior of these protocols or unforeseen issues in the integration process could result in integration risks.
* The contract relies on various external addresses, and any changes in these addresses or the introduction of new versions could lead to integration issues.


#### Non-Standard Token Risks:

* The contract involves the handling of tokens (e.g., Pendle tokens) and ETH. It is important to ensure that the transfer and approval mechanisms are secure and that interactions with these tokens are correctly implemented.

</details>




### [File-17] /AaveHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract does not directly expose explicit administrative functionalities, but the inherited contracts (`Declarations` and potential other inherited contracts) may contain owner-specific functions that could pose admin abuse risks. A more in-depth analysis of the inherited contracts is neede



#### Systemic Risks:

* The contract interacts with external systems such as Aave, WiseLending, and other protocols. Any vulnerabilities in these external systems or unexpected changes in their behavior may lead to systemic risks.
* The usage of modifiers and functions like `_nonReentrantCheck`, `_reservePosition`, `_wrapDepositExactAmount`, `_wrapWithdrawExactAmount`, `_wrapWithdrawExactShares`, `_wrapBorrowExactAmount`, and others introduces complexity that requires careful testing and understanding to avoid systemic issues.


#### Technical Risks:

* The `nonReentrant` modifier implies the contract is designed to prevent reentrancy attacks. Ensuring that reentrancy vulnerabilities are properly mitigated is essential.
* The reliance on external protocols, including Aave and WiseLending, implies potential risks related to changes in these protocols or unforeseen issues in their interaction.
* The function `_wrapAaveReturnValueDeposit` involves interacting with Aave and wrapping the deposit amount. Ensuring the accuracy of the wrapped amount and handling any potential discrepancies is critical.


#### Integration Risks:

* The contract integrates with external protocols, and any changes in the behavior of these protocols or potential issues in the integration process could result in integration risks.
* The usage of external addresses and mappings, such as `aaveTokenAddress`, introduces risks related to address changes and mapping consistency.


#### Non-Standard Token Risks:

* The contract interacts with tokens, particularly Aave tokens. Ensuring that token transfers and approvals are handled securely is crucial to avoid non-standard token risks.


</details>



### [File-18] PositionNFTs.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract has the `onlyMaster` modifier in several places, restricting certain functions to be accessible only by the contract owner. Admin abuse risks could arise if the owner misuses these privileged functions to the detriment of users or the contract's intended functionality.


#### Systemic Risks:

* The contract inherits from `ERC721Enumerable`, and any vulnerabilities or weaknesses in the ERC721 standard or its implementations may pose systemic risks.
* The contract relies on external calls to Aave, WiseLending, and other protocols, which introduces dependencies. Changes in the behavior of these external protocols or unforeseen issues in their interaction may lead to systemic risks.


#### Technical Risks:

* The contract utilizes modifiers like `onlyReserveRole` and `_mintPositionForUser` to manage access and minting positions. Ensuring the correct implementation of these modifiers is crucial to prevent technical vulnerabilities.
* The usage of mappings, such as `reserveRole` and `reserved`, needs careful scrutiny to avoid potential vulnerabilities like reentrancy attacks or unexpected state changes.
* The implementation of the `approveMint` function has a conditional check that could potentially lead to unexpected behaviors. A detailed review is needed to ensure the intended logic is correctly implemented.


#### Integration Risks:

* The contract interacts with external contracts, including Aave and WiseLending. Any changes in the behavior of these external contracts or potential issues in their integration may lead to integration risks.


#### Non-Standard Token Risks:

* The contract deals with ERC721 tokens, and any non-standard implementations or deviations from the ERC721 standard may pose risks.

</details>



### [File-19] WiseSecurityDeclarations.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract has privileged functions such as `_setLiquidationSettings` and various mappings that could be manipulated by the owner, posing admin abuse risks. Care should be taken to ensure these functions are used responsibly, or access is restricted appropriately.



#### Systemic Risks:

* The contract relies on external contracts/interfaces like AaveHub, WiseLending, OracleHub, FeeManager, PositionNFTs, and WiseLiquidation. Any changes in the behavior of these external contracts or potential issues in their integration may lead to systemic risks.


#### Technical Risks:

* The contract includes various error conditions, and the logic inside functions should be thoroughly reviewed to ensure proper error handling and avoid unexpected behaviors.
* The usage of mappings, such as `securityWorker` and `wasBlacklisted`, may need additional scrutiny to prevent potential vulnerabilities like reentrancy attacks or unexpected state changes.
* The contract utilizes modifiers like `onlyMaster` and `_setLiquidationSettings`, and their implementations should be reviewed to prevent any technical vulnerabilities.


#### Integration Risks:

* The contract interacts with external contracts/interfaces, including AaveHub, WiseLending, OracleHub, FeeManager, PositionNFTs, and WiseLiquidation. Any issues in these interactions may pose integration risks.


#### Non-Standard Token Risks:

* The contract deals with ERC20 tokens, and the usage of external interfaces such as CurveSwapStructData and CurveSwapStructToken indicates possible interactions with custom tokens. Any non-standard implementations or deviations from standards may pose risks.

</details>



### [File-20] PoolManager.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract includes privileged functions such as `setParamsLASA`, `setPoolParameters`, `setVerifiedIsolationPool`, `createPool`, and `createCurvePool`, which can only be called by the owner (onlyMaster). These functions can be abused by the owner, posing admin abuse risks. Care should be taken to ensure these functions are used responsibly, or access is restricted appropriately.



#### Systemic Risks:

* The contract relies on various external factors, such as the use of other smart contracts and interfaces, including WiseCore, Babylonian, and external data structures. Any changes in the behavior of these external components may lead to systemic risks.


#### Technical Risks:

* The contract includes complex mathematical calculations involving square roots and various factors. The accuracy and precision of these calculations should be carefully reviewed to prevent unintended consequences or vulnerabilities.
* The use of external libraries such as Babylonian for mathematical calculations should be validated to ensure their correctness and security.
* The contract maintains state variables that track pool parameters, lending data, algorithmic data, and more. Careful attention should be paid to the interaction and manipulation of these state variables to avoid potential bugs or vulnerabilities.



#### Integration Risks:

* The contract interacts with other contracts, such as WiseCore, Babylonian, and potentially external token contracts. Any issues in the integration with these contracts may pose risks.


#### Non-Standard Token Risks:

* The contract deals with ERC20 tokens, and it defines various data structures and functions related to creating and managing pools. The usage of external interfaces and functions suggests interactions with custom tokens. Any non-standard implementations or deviations from standards may pose risks.


</details>



### [File-21] FeeManagerHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract includes several privileged functions like `_prepareBorrows`, `_prepareCollaterals`, `_setBadDebtPosition`, `_increaseTotalBadDebt`, `_decreaseTotalBadDebt`, `_eraseBadDebtUser`, `_updateUserBadDebt`, `_increaseFeeTokens`, `_decreaseFeeTokens`, `_setAllowedTokens`, `_setAaveFlag`, and others, which are marked as **internal** and can only be called within the contract. These functions could potentially be misused by the owner (master) and pose admin abuse risks. It's crucial to ensure that these functions are used responsibly and access is appropriately controlled.



#### Systemic Risks:

* The contract depends on external components and interfaces, such as `WISE_LENDING`, `WISE_SECURITY`, `TransferHelper`, and the `ORACLE_HUB`. Any changes in the behavior of these external elements might introduce systemic risks to the contract.


#### Technical Risks:

* The contract performs various complex operations, such as updating bad debt, distributing incentives, preparing borrows and collaterals, etc. The correctness and efficiency of these operations need to be thoroughly reviewed to prevent potential vulnerabilities or bugs.
* The use of dynamic storage for tracking fee tokens (`feeTokens`), bad debt per position (`badDebtPosition`), and incentive amounts (`incentiveUSD`) requires careful consideration to manage gas costs and storage efficiently.


#### Integration Risks:

* The contract interacts with other contracts and interfaces, such as `WISE_LENDING`, `WISE_SECURITY`, and `ORACLE_HUB`. Any issues in the integration with these external components may pose risks.


#### Non-Standard Token Risks:

* The contract interacts with various tokens through interfaces like `WISE_LENDING` and `ORACLE_HUB`. It's essential to ensure that these tokens adhere to standard ERC-20 specifications to prevent non-standard token risks


#### Suggestions:

It's recommended to conduct thorough testing, code reviews, and potentially third-party audits to identify and mitigate these potential risks accurately. Additionally, clear documentation regarding the functionality and usage of the contract should be provided to enhance transparency and assist in risk assessment.

</details>



### [File-22] PendlePowerFarmMathLogic.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract includes a modifier called `updatePools` which checks for reentrancy using the function `_checkReentrancy`. While reentrancy protection is essential, it's crucial to ensure that it doesn't limit legitimate use cases or introduce unintended consequences. A thorough review of the reentrancy protection mechanism is required to address any potential admin abuse risks.



#### Systemic Risks:

* The contract heavily relies on external interfaces such as `WISE_LENDING`, `WISE_SECURITY`, and `ORACLE_HUB`. Any changes to these external components might introduce systemic risks to the contract. It's essential to consider potential changes in the behavior of these interfaces and evaluate the impact on the contract's functionality.


#### Technical Risks:

* The contract involves complex mathematical calculations for approximating net APY, new borrow rates, leverage amounts, and debt ratios. These calculations should be carefully reviewed to ensure accuracy and to prevent vulnerabilities or bugs that may lead to incorrect financial outcomes.


#### Integration Risks:

* The integration with external contracts (`WISE_LENDING`, `WISE_SECURITY`, and `ORACLE_HUB`) poses integration risks. Any issues or changes in these external contracts could impact the functionality of the current contract. It's important to consider potential changes in the external contracts and assess their impact on the overall system.


#### Non-Standard Token Risks:

* The contract interacts with various tokens, such as `WETH_ADDRESS`, `AAVE_WETH_ADDRESS`, and `PENDLE_CHILD`. It's crucial to ensure that these tokens adhere to standard ERC-20 specifications to prevent non-standard token risks.


</details>



### [File-23] DeclarationsFeeManager.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract owner, controlled by `OwnableMaster`, has significant control over critical functions. This includes the ability to change the incentive master, propose a new incentive master, and add or remove pool tokens. These actions may impact the operation of the contract and should be carefully managed to prevent abuse.



#### Systemic Risks:

* The contract interacts with external contracts, such as `AAVE`, `WISE_LENDING`, `WISE_SECURITY`, `POSITION_NFTS`, and `ORACLE_HUB`. Changes to these external contracts or their unexpected behaviors might introduce systemic risks to the fee manager contract. It is essential to consider the potential changes in the behavior of these interfaces and evaluate their impact on the contract's functionality.


#### Technical Risks:

* The contract involves various mathematical calculations and relies on external interfaces. It's important to ensure the accuracy and reliability of these calculations to prevent vulnerabilities or bugs that could lead to incorrect financial outcomes.


#### Integration Risks:

* The contract integrates with several external contracts, including `AAVE`, `WISE_LENDING`, `WISE_SECURITY`, `POSITION_NFTS`, and `ORACLE_HUB`. Integration risks arise if these external contracts change their interfaces or behaviors. A careful analysis of these interfaces and potential changes is necessary to mitigate integration risks.


#### Non-Standard Token Risks:

* The contract interacts with different tokens, such as pool tokens, aTokens from Aave, and other ERC-20 tokens. Ensure that these tokens adhere to standard ERC-20 specifications to avoid non-standard token risks.

</details>



### [File-24] PendlePowerManager.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol)


<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract owner, controlled by `OwnableMaster`, has significant control over critical functions. This includes the ability to change the minimum deposit, shut down the farm, and perform manual paybacks and withdrawals. The misuse of these functions by the owner may lead to unintended consequences or loss of funds.



#### Systemic Risks:

* The contract integrates with various external contracts, such as `PendlePowerFarm`, `OwnableMaster`, and `MinterReserver`. Changes in the behavior of these contracts or their unexpected actions might introduce systemic risks to the PendlePowerManager contract. Ensuring compatibility with external contracts and monitoring potential changes is crucial.


#### Technical Risks:

* The contract involves complex logic for managing power farms, opening/closing positions, and interacting with external protocols. The mathematical calculations and interactions with external contracts must be thoroughly tested to avoid vulnerabilities or bugs that could compromise the contract's functionality


#### Integration Risks:

* The contract interacts with external contracts, including `PendlePowerFarm`, `OwnableMaster`, and `MinterReserver`. Changes in the interfaces or behaviors of these contracts may impact the `PendlePowerManager` contract. A careful analysis of these interfaces and potential changes is necessary to mitigate integration risks.


#### Non-Standard Token Risks:

* The contract interacts with various tokens, including WETH, power farm tokens, and others. Ensure that these tokens adhere to standard ERC-20 specifications to avoid non-standard token risks.

</details>



### [File-25] PendlePowerFarmControllerHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract owner is not explicitly defined in the provided code. Admin abuse risks might arise if there are undisclosed or elevated privileges given to specific addresses, leading to potential manipulation of critical functions. Ensure that admin privileges are well-documented and limited to necessary functions.



#### Systemic Risks:

* The contract relies on various external contracts, such as `PendlePowerFarmControllerBase`, `IPendleMarket`, and `PendleLock`. Changes in these external contracts or their unexpected behaviors could introduce systemic risks to the `PendlePowerFarmControllerHelper` contract. Regularly checking and adapting to updates in external contracts is essential.


#### Technical Risks:

* The contract involves complex logic for managing locked positions, calculating rewards, and interacting with external protocols. Potential technical risks include bugs, vulnerabilities, or inconsistencies in mathematical calculations. Comprehensive testing and code reviews are crucial to identifying and addressing technical risks.


#### Integration Risks:

* The contract interacts with external contracts, including oracles (`ORACLE_HUB`) and `IPendleMarket`. Changes in the interfaces or behaviors of these contracts may impact the `PendlePowerFarmControllerHelper` contract. Ensuring compatibility with external contracts and monitoring potential changes is necessary to mitigate integration risks.


#### Non-Standard Token Risks:

* The contract interacts with various tokens, including Pendle tokens (`PENDLE_LOCK`) and reward tokens. Ensure that these tokens adhere to standard ERC-20 specifications to avoid non-standard token risks. Additionally, confirm that the oracles (`ORACLE_HUB`) support the tokens used in the contract.


</details>



### [File-26] PendlePowerFarm.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract does not explicitly define an owner or access control mechanism, which could expose it to admin abuse risks. It's crucial to implement access controls and clearly define ownership to prevent unauthorized access to critical functions and potential abuse.



#### Systemic Risks:

* The contract relies on external contracts, such as `PendlePowerFarmLeverageLogic`, `PENDLE_CHILD`, and `WISE_LENDING`. Changes in these external contracts or their unexpected behaviors could introduce systemic risks to the `PendlePowerFarm` contract. Regularly checking and adapting to updates in external contracts is essential


#### Technical Risks:

* The contract involves complex logic related to leverage, liquidation, and interaction with external protocols. Potential technical risks include bugs, vulnerabilities, or inconsistencies in mathematical calculations. Comprehensive testing and code reviews are crucial to identifying and addressing technical risks.


#### Integration Risks:

* The contract interacts with external protocols and contracts, such as `PENDLE_CHILD` and `WISE_LENDING`. Changes in the interfaces or behaviors of these external contracts may impact the `PendlePowerFarm` contract. Ensuring compatibility with external contracts and monitoring potential changes is necessary to mitigate integration risks.


#### Non-Standard Token Risks:

* The contract interacts with tokens but doesn't explicitly specify the token standards or check for compliance. Ensure that the tokens used in the contract adhere to standard ERC-20 specifications to avoid non-standard token risks.


#### Suggestions:

Implementing access controls and providing clear documentation on the contract's functionality and usage can enhance transparency and risk assessment.

</details>


 
### [File-27] PowerFarmNFTs.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract has an `onlyMaster` modifier in the `setFarmContract`, `setBaseURI`, and `setBaseExtension` functions, restricting access to the owner. However, the farm contract address can be set only once, and subsequent calls to `setFarmContract` will not have any effect. There's a risk if this contract is not owned by a secure and trusted entity, as the initial farm contract setting is critical.



#### Systemic Risks:

* The contract interacts with the external `farmContract`, and the validity of its behavior relies on the correct implementation and behavior of this external contract. Changes in the `farmContract` may impact the expected behavior of this contract. It's crucial to ensure compatibility and proper functioning of external contracts.


#### Technical Risks:

* The contract inherits from the `ERC721Enumerable` contract, which is a well-known standard, but potential risks may arise from bugs or vulnerabilities in the implementation. Additionally, the contract relies on string concatenation for generating token URIs, which could lead to gas inefficiencies or potential vulnerabilities if not handled carefully.


#### Integration Risks:

* The contract interacts with external contracts through the `farmContract` interface. Any changes in the external contract's interface or behavior may lead to integration risks. It's important to monitor and adapt to updates in external contracts to maintain proper functionality.


#### Non-Standard Token Risks:

* The contract deals with ERC-721 tokens, and it doesn't explicitly specify handling of non-standard tokens. Ensure that the external contracts, especially the `farmContract`, use standard interfaces and adhere to expected behaviors to avoid non-standard token risks.


#### Suggestions:

To mitigate these risks, thorough testing, code reviews, and potentially third-party audits are recommended. Additionally, the contract owner should be a trusted entity, and the external contracts' interfaces should be well-documented and stable. Regular monitoring for updates and changes in external contracts is crucial for maintaining the integrity of the system.

</details>



### [File-28] MinterReserver.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract has an `onlyKeyOwner` modifier to ensure that certain functions can only be called by the owner of a specific key. However, the owner check relies on the `isOwner` function, which includes a check against the owner of the corresponding NFT in the `FARMS_NFTS` contract. If the external contract behavior changes unexpectedly, it could lead to potential admin abuse risks.



#### Systemic Risks:

* The contract interacts with an external contract (`FARMS_NFTS`), and its functionality depends on the proper behavior of this external contract. Changes in the external contract's interface or behavior could lead to systemic risks.


#### Technical Risks:

* The use of the `availableNFTCount` variable and its related functions might introduce potential issues if not managed correctly. The contract relies on proper incrementing and decrementing of counters, and any discrepancies could lead to unexpected behavior.


#### Integration Risks:

* The contract relies on external contracts, especially the `FARMS_NFTS` contract. Any changes in the external contract's behavior, interfaces, or ownership structure may lead to integration risks. Proper coordination and monitoring are essential to avoid disruptions.


#### Non-Standard Token Risks:

* The contract interacts with ERC-721 tokens, specifically the `FARMS_NFTS` contract. It assumes standard behavior and interfaces for ERC-721 tokens. If the external contract deviates from standards, it may introduce non-standard token risks.


</details>



### [File-29] PendleLpOracle.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract appears to lack explicit access control mechanisms, and anyone can deploy this contract. Depending on its deployment context and intended usage, this might pose admin abuse risks. Implementing proper access controls, such as using the Ownable pattern or a more sophisticated role-based access control system, would enhance security.



#### Systemic Risks:

* The contract relies on external contracts, such as `IPendle`, `IPriceFeed`, and `IOraclePendle`, which introduces systemic risks. Changes or vulnerabilities in these external contracts could affect the functionality and security of the `PendleLpOracle` contract. Close coordination and monitoring of these dependencies are essential to mitigate systemic risks.


#### Technical Risks:

* The contract uses the `FEED_ASSET.latestRoundData()` function, which retrieves data from an external price feed (`IPriceFeed`). If the external price feed has vulnerabilities or provides incorrect data, it may introduce technical risks. Ensure proper error handling and validation mechanisms are in place to handle potential issues with external data sources.


#### Integration Risks:

* The contract integrates with multiple external contracts, such as `IPendle`, `IPriceFeed`, and `IOraclePendle`. Any changes to these external contracts' interfaces or behaviors may introduce integration risks. Regular monitoring and testing are crucial to addressing potential issues arising from changes in external contracts.


#### Non-Standard Token Risks:

* The contract interacts with tokens, such as `IPendleSy`. Ensure that these tokens follow standard ERC-20 or ERC-721 interfaces, depending on their nature. Non-standard behavior or deviations from standard token interfaces could introduce risks. Reviewing and testing token interactions are essential to identify and mitigate such risks.

</details>



### [File-30] Declarations.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract inherits from `OwnableMaster`, which may introduce admin abuse risks if not properly managed. It's crucial to ensure that the ownership functions, such as `transferOwnership`, are used responsibly and are well-protected against potential unauthorized access.



#### Systemic Risks:

* The contract relies on external contracts and interfaces, such as `IERC20`, `IPriceFeed`, `IAggregator`, and others. Systemic risks may arise if these external contracts have vulnerabilities or undergo changes in their behavior. Regular monitoring and potentially audits of these dependencies are necessary to mitigate systemic risks.


#### Technical Risks:

* The contract involves complex calculations and interactions with external systems, such as Uniswap V3 and Chainlink oracles. Technical risks may arise from vulnerabilities in the calculations, reliance on external oracles, and potential inconsistencies in data sources. Thorough testing and validation of the contract's logic are essential to identify and address potential technical risks.


#### Integration Risks:

* The contract integrates with various external systems, including Uniswap V3, Chainlink oracles, and others. Changes in the behavior of these external systems or contract interfaces may introduce integration risks. Continuous monitoring and adaptation to changes in external dependencies are crucial to address potential integration risks.


#### Non-Standard Token Risks:

* The contract interacts with ERC-20 tokens (`IERC20`). Ensuring that these tokens follow standard ERC-20 interfaces is important to avoid non-standard token risks. Any deviations from the standard ERC-20 interface may lead to unexpected behavior or vulnerabilities.


#### Additional Notes:

Careful consideration of the external dependencies and their potential impact on the contract's functionality is crucial. Additionally, ensure that access control mechanisms are appropriately implemented to mitigate admin abuse risks. Regularly update the contract based on changes in external dependencies or evolving best practices in the blockchain space.


</details>



### [File-31] PtOracleDerivative.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract does not appear to have explicit access control mechanisms or ownership restrictions. Without proper access controls, there is a risk of admin abuse if unauthorized parties gain access to critical functions or if there is no way to limit or transfer administrative privileges.



#### Systemic Risks:

* The contract relies on external contracts and interfaces, such as `IPriceFeed` and `IOraclePendle`. Systemic risks may arise if these external contracts have vulnerabilities, experience changes in behavior, or if their interfaces are updated. Regular monitoring and potentially audits of these dependencies are necessary to mitigate systemic risks.


#### Technical Risks:

* The contract involves complex calculations and interactions with external systems, such as Chainlink oracles. Technical risks may arise from vulnerabilities in the calculations, reliance on external oracles, and potential inconsistencies in data sources. Thorough testing and validation of the contract's logic are essential to identify and address potential technical risks.


#### Integration Risks:

* The contract integrates with external oracles (`IPriceFeed`, `IOraclePendle`) and relies on their data. Integration risks may arise if the external oracles behave unexpectedly, return inaccurate data, or undergo changes. Continuous monitoring and adaptation to changes in external dependencies are crucial to address potential integration risks.


#### Non-Standard Token Risks:

* The contract performs calculations with token values (`answerUsdFeed` and `answerEthUsdFeed`). While the contract seems to handle token values properly, ensuring that these tokens follow standard ERC-20 interfaces is important to avoid non-standard token risks. Any deviations from the standard ERC-20 interface may lead to unexpected behavior or vulnerabilities.


</details>



### [File-32] Declarations.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract inherits from `OwnableMaster`, indicating that the owner has significant control over the contract. Admin abuse risks could arise if the owner misuses their privileges to modify critical functionalities or if there are vulnerabilities in the owner's functions. It's crucial to clearly define and limit the owner's powers to mitigate admin abuse risks.


#### Systemic Risks:

* The contract interacts with external contracts, including the Aave protocol (`IAave`), Wise lending platform (`IWiseLending`), and a security contract (`IWiseSecurity`). Systemic risks may arise if these external contracts have vulnerabilities, if their interfaces are modified, or if they behave unexpectedly. Regular monitoring and potential audits of these dependencies can help mitigate systemic risks.

#### Technical Risks:

* The contract has several internal functions (`_checkOwner`, `_checkPositionLocked`, `setWiseSecurity`) that interact with external contracts. Technical risks may include vulnerabilities in the contract's logic, especially in these functions. Rigorous testing and code reviews are essential to identify and address potential technical risks.


#### Integration Risks:

* Integration risks may arise due to interactions with external contracts, particularly if there are changes in the behavior of these external systems or if their interfaces are updated. Continuous monitoring and adaptation to changes in external dependencies are vital to address potential integration risks.


#### Non-Standard Token Risks:

* The contract interacts with tokens, and it's crucial to ensure that the tokens involved adhere to standard ERC-20 interfaces. Non-standard token risks may arise if tokens with non-standard behaviors are used. Verifying the compliance of tokens with standard interfaces is important to prevent unexpected behavior or vulnerabilities.


#### Additional Notes:

The owner's powers should be carefully defined and limited to necessary functions to prevent admin abuse. Additionally, ensuring the security of external dependencies and monitoring potential changes in their behavior is crucial to mitigate systemic and integration risks.


</details>



### [File-33] PtOraclePure.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract doesn't seem to have explicit admin functions or roles. However, the risk may be present if the owner has the ability to modify critical parameters or if any updates to the contract are centralized. The contract should clearly define and limit the owner's powers to mitigate admin abuse risks.



#### Systemic Risks:

* The contract interacts with external contracts, including the Pendle market contract (`IOraclePendle`) and the Chainlink price feed (`IPriceFeed`). Systemic risks may arise if these external contracts have vulnerabilities, if their interfaces are modified, or if they behave unexpectedly. Regular monitoring and potential audits of these dependencies can help mitigate systemic risks.


#### Technical Risks:

* The contract has several internal functions (`latestAnswer`, `decimals`, `latestRoundData`) that interact with external contracts. Technical risks may include vulnerabilities in the contract's logic, especially in these functions. Rigorous testing and code reviews are essential to identify and address potential technical risks.


#### Integration Risks:

* Integration risks may arise due to interactions with external contracts, particularly if there are changes in the behavior of these external systems or if their interfaces are updated. Continuous monitoring and adaptation to changes in external dependencies are vital to address potential integration risks.


#### Non-Standard Token Risks:

* The contract interacts with tokens, and it's crucial to ensure that the tokens involved adhere to standard ERC-20 interfaces. Non-standard token risks may arise if tokens with non-standard behaviors are used. Verifying the compliance of tokens with standard interfaces is important to prevent unexpected behavior or vulnerabilities.


</details>



### [File-34] OwnableMaster.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract has a mechanism to propose a new master and allows the proposed owner to claim ownership. Admin abuse risks may arise if the proposed owner is not carefully selected or if the proposed owner maliciously claims ownership. Additionally, the renounceOwnership function can potentially lead to admin abuse if triggered without proper consideration. It's essential to ensure that the proposed owner is trustworthy and that the ownership transfer process is secure.



#### Systemic Risks:

* The contract relies on the proposedMaster variable to manage ownership transitions. Systemic risks may arise if there are vulnerabilities in the logic related to the proposedMaster variable. It's crucial to thoroughly test the proposedMaster-related functions to identify and mitigate potential systemic risks


#### Technical Risks:

* The contract's implementation involves conditional checks and state transitions based on the proposedMaster variable. Technical risks may arise from vulnerabilities in these checks or transitions. Rigorous testing and code reviews are necessary to identify and address potential technical risks.


#### Integration Risks:

* The contract does not have explicit dependencies on external contracts, so integration risks are limited. However, the integration risk may arise if this contract is part of a larger system, and its interactions with other components are not well-defined or tested. Proper integration testing and verification are essential to mitigate potential risks in a broader system.


#### Non-Standard Token Risks:

* The contract does not directly interact with tokens, and there are no non-standard token risks associated with its current implementation.


</details>



### [File-35] PendlePowerFarmTokenFactory.so

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* Admin abuse risks may arise if the `PENDLE_POWER_FARM_CONTROLLER` address is not carefully managed or if an unauthorized entity gains control over it. This controller address is crucial, as only the controller can deploy new instances of `PendlePowerFarmToken`. It is essential to ensure that the controller address is secure, well-protected, and only accessible by authorized entities to prevent potential abuse.


#### Systemic Risks:

* The contract relies on the `IMPLEMENTATION_TARGET` address to determine the target implementation for the `PendlePowerFarmToken`. Systemic risks may arise if there are vulnerabilities in the logic related to the implementation target address. Careful consideration and testing of the implementation target logic are necessary to prevent potential systemic risks.


#### Technical Risks:

* The contract uses low-level assembly to create a new contract instance using `create2`. Technical risks may arise from vulnerabilities in the assembly code or from improper usage of the `create2` function. Thorough testing and code reviews are essential to identify and address potential technical risks associated with the low-level assembly code.


#### Integration Risks:

* Integration risks may arise if the contract is part of a larger system, and its interactions with other components are not well-defined or tested. Proper integration testing and verification are crucial to mitigate potential risks in a broader system. Additionally, the contract relies on the `PENDLE_POWER_FARM_CONTROLLER` address, and integration risks may emerge if this controller address is not properly integrated with other components.


#### Non-Standard Token Risks:

* The contract is responsible for deploying instances of the `PendlePowerFarmToken`, and non-standard token risks may arise if there are vulnerabilities in the implementation of the `PendlePowerFarmToken` contract. It is essential to thoroughly audit and test the `PendlePowerFarmToken` contract to ensure that it adheres to standard token practices and does not introduce any security risks.


#### Additional Notes:

Careful consideration should be given to the management of the `PENDLE_POWER_FARM_CONTROLLER` address to prevent admin abuse. Thorough testing of the implementation target logic and the assembly code is essential to identify and address potential systemic and technical risks. Proper integration testing and verification are crucial, especially if this contract is part of a larger system. Finally, a thorough audit of the `PendlePowerFarmToken` contract is necessary to address any non-standard token risks.

</details>



### [File-36] PendleChildLpOracle.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* Admin abuse risks in this contract are relatively low. The only potential point of concern could be the initialization of the `priceFeedPendleLpOracle` and `pendleChildToken` addresses during deployment. However, assuming these addresses are correctly set during deployment and are not later modified, the risk of admin abuse is limited


#### Systemic Risks:

* The contract relies on external interfaces (`IPriceFeed` and `IPendleChildToken`) to fetch data and perform calculations. Systemic risks may arise if these external contracts are not properly audited or if there are vulnerabilities in their logic. Careful consideration and auditing of the external contracts are crucial to minimize systemic risks.


#### Technical Risks:

* The calculation in the `latestAnswer` function involves multiplying values retrieved from the `priceFeedPendleLpOracle` and `pendleChildToken`. Technical risks may arise if there are precision errors or arithmetic overflows during these calculations. It is essential to conduct thorough testing and validation to ensure the correctness of these calculations.


#### Integration Risks:

* Integration risks may arise if this contract is part of a larger system, and its interactions with other components are not well-defined or tested. Proper integration testing and verification are crucial to mitigate potential risks in a broader system. Additionally, the correct initialization of the `priceFeedPendleLpOracle` and `pendleChildToken` addresses during deployment is critical for seamless integration.


#### Non-Standard Token Risks:

* The contract interacts with the `IPendleChildToken` interface, and non-standard token risks may arise if the implementation of `IPendleChildToken` does not adhere to standard token practices. A thorough audit of the `IPendleChildToken` contract is necessary to identify and address any non-standard token risks.


#### Additional Notes:

Careful consideration should be given to the initialization of external contract addresses during deployment to prevent admin abuse. Additionally, auditing of the external contracts (`IPriceFeed` and `IPendleChildToken`) is essential to minimize systemic risks. Comprehensive testing and validation of calculations are crucial to address potential technical risks. Proper integration testing and verification are necessary, especially if this contract is part of a larger system. Finally, an audit of the `IPendleChildToken` contract is recommended to ensure it adheres to standard token practices.


</details>



### [File-37] FeeManagerEvents.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerEvents.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* There is no admin functionality or state-changing logic in this contract. Therefore, the risk of admin abuse is minimal. Admin abuse typically involves misuse of privileged roles, but this contract is focused on emitting events and does not have such functionality.



#### Systemic Risks:

* Systemic risks are generally associated with the core functionality of a contract, such as smart contract vulnerabilities, logic flaws, or integration issues. As this contract is events-only, there is no inherent systemic risk within the contract itself. However, if it is part of a larger system, the overall system should be carefully audited to identify and mitigate any systemic risks.

#### Technical Risks:

* There are no technical risks within this contract since it lacks executable logic. The risk associated with its use is dependent on the implementation and security of the contracts that interact with these events.


#### Integration Risks:

* Integration risks might arise if other contracts or systems expect certain events to be emitted by this contract, and there is a mismatch in event structure or semantics. Integration testing is crucial to ensure that the events emitted by this contract are properly consumed and handled by other components in the system.


#### Non-Standard Token Risks:

* The contract does not interact with tokens directly, and it primarily emits events related to fees, incentives, and bad debt. Therefore, non-standard token risks are not applicable in this context.


#### Summary:

In summary, while the `FeeManagerEvents` contract itself does not pose significant risks, its usage and integration within a broader system should be carefully reviewed and tested to ensure seamless interoperability and alignment with the overall system's goals.


</details>



### [File-38] CustomOracleSetup.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The `onlyOwner` modifier restricts access to certain functions to only the owner, minimizing the risk of admin abuse.
* The `renounceOwnership` function allows the owner to renounce ownership, which may pose a risk if ownership is renounced without appropriate consideration and control measures in place.



#### Systemic Risks:

* The contract is primarily designed to manage timestamps, round data, and the global aggregator round ID. Systemic risks are minimal as the contract doesn't perform complex operations or handle critical functionality.


#### Technical Risks:

* The `setRoundData`, `setLastUpdateGlobal`, and `setGlobalAggregatorRoundId` functions can be used to update critical parameters. If misused, they may introduce technical risks, such as incorrect timestamp assignments or inconsistent data.


#### Integration Risks:

* If other contracts or systems rely on the data managed by this contract, there could be integration risks. For example, incorrect round data or global aggregator round ID updates might impact dependent contracts.


#### Non-Standard Token Risks:

* The contract doesn't directly deal with tokens, so non-standard token risks are not applicable.


#### Recommendations:

1. **Access Control:** Ensure that only trusted entities have ownership rights. The renounceOwnership function should be used cautiously, and proper access control measures must be in place.

2. **Event Emitters**: Consider adding events to signal changes in state, enhancing transparency and enabling other contracts to react to changes.

3. **Documentation**: Provide clear documentation about the purpose of the contract, its functions, and any dependencies it may have.

4. **Testing**: Rigorously test the contract in various scenarios, especially if it is part of a larger system. Pay attention to boundary cases and edge conditions.

5. **Access Control Update**: Consider adding a mechanism to transfer ownership securely, possibly through a multisig process or other secure methods.

6. **Code Auditing**: Conduct a thorough code review or consider a professional audit to identify any potential vulnerabilities or issues.

7. **Integration Testing**: If other contracts depend on this contract, perform comprehensive integration testing to ensure seamless interoperability.

8. **Event Emission**: Consider adding events to provide a clearer history of state changes, which can aid in debugging and monitoring.

9. **Emergency Stop Mechanism**: Evaluate the need for an emergency stop or upgrade mechanism to pause contract functionality in case of unforeseen issues.

</details>



### [File-39] SendValueHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract doesn't have explicit admin control mechanisms, but it has a `sendingProgress` state variable, which could be misused if not properly managed. If access control is not appropriately implemented, there may be a risk of admin abuse.



#### Systemic Risks:

* The `sendingProgress` state variable might lead to systemic risks if not handled properly. It is crucial to ensure that the state is properly managed to avoid unintended behaviors.

#### Technical Risks:

* The `_sendValue` function has the potential for reentrancy vulnerabilities if it's called in a way that allows the recipient contract to call back into the `_sendValue` function before it completes execution. This can be mitigated with appropriate checks and the use of the `reentrancyGuard` pattern.


#### Integration Risks:

* If this contract is integrated with other contracts or systems, there may be integration risks related to the transfer of Ether. Ensuring proper handling of Ether transfers and potential callback mechanisms is crucial for safe integration.


#### Non-Standard Token Risks:

* The contract doesn't deal with tokens directly, so non-standard token risks are not applicable.


#### Recommendations:

1. **Access Control**: Consider implementing proper access control mechanisms to prevent unauthorized entities from interacting with the contract. This can help mitigate admin abuse risks.

2. **Reentrancy Protection**: Implement reentrancy protection using the reentrancyGuard pattern to prevent potential reentrancy attacks.

3. **Error Handling**: Enhance error handling and provide informative error messages to aid in debugging and understanding failures.

4. **Documentation**: Clearly document the purpose and behavior of the contract, especially regarding the `sendingProgress` state variable and the `_sendValue` function.

5. **Testing**: Rigorously test the contract under various conditions, including edge cases, to identify and address any potential vulnerabilities.

6. **Event Emission**: Consider emitting events to provide transparency on state changes and significant contract actions. Events can assist in monitoring and debugging.

7. **Access Control Updates**: Implement emergency stop mechanisms or access control updates to pause or modify contract behavior if necessary.

8. **Security Audits**: Consider engaging in a security audit to identify and address any potential vulnerabilities or issues in the contract code.

</details>



### [File-40] WrapperHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract appears to be a helper contract and doesn't directly involve admin functionality. However, the integration of this contract into a larger system may introduce admin-related risks. Ensure that proper access controls are implemented in the broader system.



#### Systemic Risks:

* The contract doesn't seem to introduce systemic risks on its own. However, when integrated into a larger system, the proper functioning of this contract becomes critical to the overall system.
  

#### Technical Risks:

* The contract has straightforward functions for wrapping and unwrapping ETH using the `IWETH` interface. However, potential risks could arise if the `IWETH` contract itself has vulnerabilities. Ensure that the `IWETH` contract is well-audited and trusted.


#### Integration Risks:

* When integrated into a larger system, there may be risks related to the proper coordination and interaction with other components. Ensure that the integration is well-tested and follows best practices for interoperability.


#### Non-Standard Token Risks:

* The contract interacts with the `IWETH` interface, which represents WETH (Wrapped Ether). As long as WETH adheres to ERC-20 standards, non-standard token risks are minimal.


#### Recommendations:

1. **Access Control**: If this contract is part of a larger system, ensure that access controls are appropriately implemented in the broader system to prevent unauthorized use or admin abuse.

2. **Interface Verification**: Confirm that the `IWETH` interface is correctly defined and points to a well-audited and trusted WETH implementation.

3. **Integration Testing**: Thoroughly test the integration of this contract within the larger system to identify and address any issues related to interoperability.

4. **Error Handling**: Enhance error handling and provide informative error messages to aid in debugging and understanding failures.

5. **Documentation**: Clearly document the purpose and behavior of the contract, especially regarding the interaction with the `IWETH` interface.

6. **Security Audits**: If the contract is part of a critical system, consider engaging in a security audit to identify and address any potential vulnerabilities.

</details>

### [File-41] CallOptionalReturn.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract does not seem to introduce direct admin functionality or permissions. However, the usage of this contract within a broader system might expose admin abuse risks depending on how it's integrated. Ensure that proper access controls are implemented at the system level.



#### Systemic Risks:

* The contract itself does not introduce systemic risks. Its primary function is to facilitate low-level calls. However, when integrated into a larger system, the way it is used can introduce systemic risks if not properly managed.


#### Technical Risks:

* The `_callOptionalReturn` function facilitates low-level calls and checks the success of the call, but it may not handle all possible edge cases. Care must be taken to ensure that the function is used correctly and that failure cases are appropriately handled in the calling code.


#### Integration Risks:

* The contract poses integration risks if not used correctly within a larger system. Incorrect use of low-level calls or improper handling of return data could lead to unexpected behavior or vulnerabilities in the broader system.


#### Non-Standard Token Risks:

* The contract does not directly interact with tokens, but it contains a parameter for an address representing a token. If this contract is used in a system where tokens are involved, ensure that the calling code interacts with standard ERC-20 tokens to avoid non-standard token risks.


#### Recommendations:

1. **Access Control**: Implement proper access controls and permissions at the system level if the contract is used within a broader system.

2. **Documentation**: Clearly document the purpose and usage of the _callOptionalReturn function. Provide guidelines on correct usage to avoid misuse.

3. **Error Handling**: Enhance error handling in the _callOptionalReturn function to handle various edge cases and provide informative error messages.

4. **Integration Testing**: Thoroughly test the integration of this contract within the larger system to identify and address any issues related to interoperability.

5. **Security Audits**: Consider engaging in a security audit, especially if this contract is used in a critical system, to identify and address potential vulnerabilities.

</details>



### [File-42] TransferHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract does not introduce direct admin functionality or permissions. It is focused on providing safe transfer functions. Admin abuse risks are not applicable to this specific contract.



#### Systemic Risks:

* The contract itself does not introduce systemic risks. Its purpose is to provide safe transfer functions using low-level calls. However, when integrated into a larger system, the way it is used can introduce systemic risks if not properly managed.

#### Technical Risks:

* The `_safeTransfer` and `_safeTransferFrom` functions use low-level calls to execute token transfers. The success of these calls depends on the correctness of the underlying token contract and may not handle all possible edge cases. Ensure that the calling code uses these functions correctly.


#### Integration Risks:

* The contract poses integration risks if not used correctly within a larger system. Incorrect use of the `_safeTransfer` and `_safeTransferFrom` functions or improper handling of return data could lead to unexpected behavior or vulnerabilities in the broader system.


#### Non-Standard Token Risks:

* The contract interacts with ERC-20 tokens, and the risks associated with the ERC-20 standard apply. It is important to ensure that the tokens used are standard ERC-20 compliant to avoid non-standard token risks.


#### Recommendations:

**Documentation**: Clearly document the purpose and usage of the `_safeTransfer` and `_safeTransferFrom` functions. Provide guidelines on correct usage to avoid misuse.

**Error Handling**: Enhance error handling in the low-level calls to handle various edge cases and provide informative error messages.

**Integration Testing**: Thoroughly test the integration of this contract within the larger system to identify and address any issues related to interoperability.

**Token Standard Compliance**: Ensure that the tokens used with this contract are standard ERC-20 compliant to avoid potential non-standard token risks.

**Security Audits**: Consider engaging in a security audit, especially if this contract is used in a critical system, to identify and address potential vulnerabilities.

</details>



### [File-43] AaveEvents.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveEvents.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract doesn't expose any administrative functions or permissions, and there are no mechanisms for admin abuse. Therefore, admin abuse risks are not applicable to this contract.



#### Systemic Risks:

* The contract introduces events related to Aave interactions, such as deposits, withdrawals, borrows, paybacks, solely deposits, and solely withdrawals. However, systemic risks would be more associated with the system where these events are utilized rather than this contract itself.

#### Technical Risks:

* The contract relies on emitting events to log various Aave-related actions. The accuracy and completeness of these logs depend on the correct implementation and usage of the contract. If the contract is not correctly integrated or utilized, it may lead to inaccurate logs.


#### Integration Risks:

* The integration risks are more related to how these events are used in the larger system. For example, if these events are relied upon for critical decisions, their accurate emission and interpretation are crucial. Incorrect usage or interpretation in the broader system could lead to integration issues.


#### Non-Standard Token Risks:

* The contract doesn't directly interact with tokens or enforce any token standards. If there are other parts of the system that use these events to handle token interactions, the risks associated with token standards and interactions would be dependent on the implementation of those components.


#### Recommendations:

**Documentation**: Clearly document the purpose and expected usage of each event emitted in the contract.

**Event Consistency**: Ensure that the events are emitted consistently based on the actions they represent. Consistency is important for the accurate interpretation of events in the broader system.

**Integration Testing**: Thoroughly test the integration of these events within the larger system to identify and address any issues related to interoperability.

**Security Audits**: If these events are critical for system functionality, consider engaging in a security audit to identify and address potential vulnerabilities or issues related to their usage.

**Event-driven Architecture**: If the larger system relies heavily on these events, consider following best practices for event-driven architectures, including handling event reliability and ensuring proper event listeners.

</details>



### [File-44] ApprovalHelper.sol

#### The bellow issues related to the File ( Link )
[Github-Link](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/ApprovalHelper.sol)

<details>
<summary>see instances</summary>



#### Admin Abuse Risks:

* The contract does not introduce any administrative functionalities or permissions. Therefore, admin abuse risks are not applicable to this contract.



#### Systemic Risks:

* The contract provides a function `_safeApprove` for executing a safe approve for a token. The systemic risks associated with this functionality are more related to how it is used in the broader system rather than within this contract itself.

#### Technical Risks:

* The contract relies on low-level calls to execute the `approve` function on a token. This technique can introduce risks if not used properly, as it bypasses the standard interface for token interactions.
* There might be situations where the `approve` function of a token returns a value, and this contract does not check or handle the return value, potentially leading to issues.


#### Integration Risks:

* The integration risks are more related to how the `_safeApprove` function is used within the larger system. If the function is incorrectly utilized or if the return value is not handled appropriately, it may lead to integration issues.


#### Non-Standard Token Risks:

* The contract relies on the `approve` function, which is a standard ERC-20 function. If the tokens involved follow the ERC-20 standard, there should not be non-standard token risks associated with this contract.


#### Recommendations:

**Documentation**: Clearly document the purpose and expected usage of the `_safeApprove` function in the contract.

**Error Handling**: Implement error handling and checks for the return value of the `approve` function to ensure that the approval was successful.

**Testing**: Thoroughly test the contract's integration with tokens to identify and address any issues related to the `_safeApprove` function.

**Audit**: If this contract is a critical component in a larger system, consider engaging in a security audit to identify and address any vulnerabilities or issues related to its usage.

**Token Standards**: Confirm that the tokens involved follow the ERC-20 standard, as this contract assumes standard token behavior.

</details>




### Time spent:
48 hours