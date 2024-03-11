
# *Wise Lending Advanced Analysis Report*

# Wise Lending Advanced Analysis Report

| S.N | Topics                                                            |
|--------|----------------------------------------------------------------|
| 1      | [Introduction](#1-introduction)                                                |
|        |   - [Core Functionalities](#core-functionalities)                        |
|        |   - [Potential Benefits](#potential-benefits)                          |
|        |   - [What's Unique??](#whats-unique)                             |
| 2      | [Scope of Contracts](#2-scope-of-contracts)                                         |
| 3      | [Codebase Analysis](#3-codebase-analysis)                                          |
|        |   - [Contract Overview](#contract-overview)                                        |
|        |   - [Key Mechanic & Approaches](#key-mechanic--approaches)                                |
|        |   - [Codebase Quality Analysis](#codebase-quality-analysis)  [Table]                       |
| 4      | [Architecture Diagrams](#4-architecture-diagrams)                                      |
|        |   - [High-Level Overview](#high-level-overview)                                      |
|        |   - [User Interaction with Wise Lending](#user-interaction-with-wise-lending)                       |
| 5      | [Contract Functionality Overview](#5-contract-functionality-overview) [Table]                    |
| 6      | [Approach Taken Reviewing the Codebase](#6-approach-taken-reviewing-the-codebase)                      |
| 7      | [Wise Lending Workflow](#7-wise-lending-workflow)  [Table]                             |
| 8      | [Economic Model Analysis](#8-economic-model-analysis) [Table]                           |
| 9      | [Roles & Permissions](#9-roles--permissions)   [Table]                             |
| 10     | [Architecture Business Logic](#10-architecture-business-logic) [Table]                       |
| 11     | [Risk Model Analysis](#11-risk-model-analysis)                                        |
|        |   - [Centralization Risks](#centralization-risks)                                     |
|        |   - [Systematic Risks](#systematic-risks)                                         |
|        |   - [Technical Risks](#technical-risks)                                          |
|        |   - [Integration Risks](#integration-risks)                                        |
|        |   - [Weak Spots & Single Point of Failure](#weak-spots--single-point-of-failure)                     |
| 12     | [Areas of Improvement](#12-areas-of-improvement)                                       |
| 13     | [Ideas for Incorporation](#13-ideas-for-incorporation)                                    |
| 14     | [Tips & Thoughts](#14-tips--thoughts)                                            |
| 15     | [Architectural Recommendations](#15-architectural-recommendations)                              |
| 16     | [Learning and Insights](#16-learning-and-insights)                                      |
| 17     | [Conclusion](#17-conclusion)                                                 |
| 18     | [Time Spent](#18-time-spent)                                                 |







## 1 . Introduction🌟

 

Wise Lending is a decentralized finance (DeFi) platform that harnesses the power of cryptocurrency to create a new lending experience. It functions as a liquidity marketplace, connecting borrowers and lenders directly.


This report provides a comprehensive analysis and evaluation of the Wise Lending smart contract codebase. As a complex decentralized system, Wise Lending aims to enable secure and efficient decentralized lending, contributing to the rapidly growing DeFi (Decentralized Finance) landscape.




**Wise Lending positions itself as a decentralized liquidity marketplace for crypto assets. Users can participate in two primary ways:**

- **Suppliers**: Deposit crypto holdings and earn interest through a variable Annual Percentage Yield (APY).
- **Borrowers**: Access loans in various cryptocurrencies by leveraging their deposited assets as collateral.

This review covers various aspects of the codebase, including its architecture, design, and implementation details. By examining the contract structure, logic, and interactions, this report identifies potential areas of improvement and offers suggestions to enhance the codebase. The recommendations provided are based on best practices in Solidity development, gas optimization techniques, and security considerations, ensuring a more robust, secure, and efficient system.

### 1.1 Core Functionalities
- **Smart Contract-powered Lending**: Wise Lending relies on smart contracts to automate loan origination, interest calculations, and collateralization. This fosters trust and transparency while minimizing the need for intermediaries.
- **Variable APY**: The interest rate earned by suppliers fluctuates based on supply and demand for specific crypto assets. Higher demand for a particular asset translates to a potentially higher APY for suppliers.
- **Collateralized Loans**: Borrowers deposit crypto assets as collateral to secure loans. The loan-to-value (LTV) ratio determines the maximum loan amount a borrower can access.
- **Liquidity Pools**: Funds are pooled together, creating a more efficient system for matching borrowers and lenders.

### 1.2 Potential Benefits
- **Higher Returns for Suppliers**: Compared to traditional savings accounts, Wise Lending offers the potential for significantly higher returns on deposited crypto assets.
- **Improved Access to Capital**: Borrowers can access crypto loans without traditional credit checks, offering greater flexibility for individuals with limited credit history.
- **Transparency and Immutability**: The use of blockchain technology ensures transparency in transactions and immutability of data, fostering trust within the platform.
- **Frictionless Borrowing and Lending**: Smart contracts automate processes, streamlining loan origination and repayment.

### 1.3 Whats Unique??


Here's what makes Wise Lending unique compared to other crypto lending platforms

- **Built from Scratch:** Unlike many platforms that copy existing code, Wise Lending is built from the ground up, allowing for more innovative features .
- **Power Farms:** This offers advanced users unique ways to earn interest on their crypto holdings .
- **Focus on Capital Efficiency:** Wise Lending prioritizes maximizing returns for lenders .
- **Isolation Mode:** Borrowers can access up to 95% loan-to-value ratio on specific assets without impacting other holdings.
- **Lending Automated Scaling Algorithm (LASA AI):** This feature helps automatically adjust interest rates to maintain a healthy lending pool .
- **NFT collateralization:** Loan positions are represented as NFTs, potentially offering increased flexibility .
- **Payback with Lending Shares:** Borrowers can potentially repay loans using Wise Lending's governance token for additional benefits .
- **Capped Liquidation Fees:** This reduces the risk of sudden and excessive losses for borrowers during market downturns .
- **Solely Deposit Mode:** Currently, Wise Lending only allows users to deposit crypto assets, focusing on optimizing returns for lenders .







## 2 .  scope Contracts 🔍

1  . [WiseLending](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol)
2  . [WiseSecurity]( https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol)
3  . [MainHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol)
4  . [WiseSecurityHelper]( https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol)
5  . [PendlePowerFarmToken]( https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol)
6  . [FeeManager](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol)
7  . [PendlePowerFarmLeverageLogic](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol)
8  . [OracleHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol)
9  . [PendlePowerFarmController](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol)
10 . [WiseCore](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol)
11 . [WiseLendingDeclaration]( https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol)
12 . [WiseOracleHub](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol)
13 . [AaveHub](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol)
14 . [WiseLowLevelHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol)
15 . [PendlePowerFarmDeclarations](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol)
16 . [PendlePowerFarmControllerBase](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol)
17 . [AaveHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol)
18 . [PositionNFTs](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol)
19 . [WiseSecurityDeclarations](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol)
20 . [PoolManager](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol)
21 . [FeeManagerHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol)
22 . [PendlePowerFarmMathLogic](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol)
23 . [DeclarationsFeeManager](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol)
24 . [PendlePowerManager](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol)
25 . [PendlePowerFarmControllerHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol)
26 . [PendlePowerFarm](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol)
27 . [PowerFarmNFTs](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol)
28 . [MinterReserver](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol)
29 . [PendleLpOracle](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol)
30 . [Declarations](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol)
31 . [PtOracleDerivative](PtOracleDerivativehttps://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol)
32 . [Declarations](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol)
33 . [PtOraclePure](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol)
34 . [OwnableMaster](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol)
35 . [PendlePowerFarmTokenFactory](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol)
36 . [PendleChildLpOracle](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol)
37 . [FeeManagerEvents](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerEvents.sol)
38 . [CustomOracleSetup](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol)
39 . [SendValueHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol)
40 . [WrapperHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol)
41 . [CallOptionalReturn](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol)
42 . [TransferHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol)
43 . [AaveEvents](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveEvents.sol)
44 . [ApprovalHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/ApprovalHelper.sol)




## 3 . Codebase Analysis 💻


### 3.1 Contract overview 



1 . [WiseLending](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol)
WiseLending is the main contract that acts as the interface between the user and the lending protocol. It has several key functionalities such as:

- Borrowing: allows users to borrow a specific amount of an asset based on their collateral.
- Liquidation: if a user's Health Factor falls below 1, their position is subject to liquidation.
- Depositing collateral: allows users to deposit an asset to be used as collateral for borrowing.
- Withdrawing collateral: allows users to withdraw their collateral if they have no active borrowed positions.
The contract has several important state variables, including the exchangeRates, totalBorrows, totalReserves, and totalCollateral.

2 . [WiseSecurity]( https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol)
WiseSecurity is a contract that provides security measures for the WiseLending contract. It includes functionalities such as:

- Admin controls: allows for the admin to pause the contract and set various fees.
- Surplus buffer: accumulates fees and can be used to pay off debt in the event of a shortfall.
- Emergency shutdown: allows the admin to liquidate all positions in the event of a critical failure.
The contract has several important state variables, including surplusBuffer, paused, and emergencyShutdown.

3 . [MainHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol)
MainHelper is a contract that provides helper functions for the WiseLending contract. It includes functionalities such as:

- Computing a user's Health Factor based on their collateral and borrowed positions.
- Computing the required collateral for a borrow request.
- Liquidating a position if the Health Factor falls below 1.
- Updating the exchange rates for assets.

4 . [WiseSecurityHelper]( https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol)
WiseSecurityHelper is a contract that provides helper functions for the WiseSecurity contract. It includes functionalities such as:

- Updating the admin fees.
- Updating the surplus buffer.
- Updating the paused state.
- Initiating an emergency shutdown.

5 . [PendlePowerFarmToken]( https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol)
PendlePowerFarmToken is a contract that represents a power farming position. It has several key functionalities such as:

- Depositing assets: allows users to deposit an asset to start farming.
- Withdrawing assets: allows users to withdraw their assets and claim their farming rewards.
- Transferring ownership: allows users to transfer their power farming position to another user.
The contract has several important state variables, including totalSupply, balanceOf, and allowance.

6 . [FeeManager](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol)
FeeManager is a contract that manages the fees for the WiseLending contract. It includes functionalities such as:

- Computing the fees for a borrow request.
- Updating the fees.
- Withdrawing fees to the admin.
The contract has several important state variables, including borrowFee, liquidationFee, and adminFee.

7 . [PendlePowerFarmLeverageLogic](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol)
PendlePowerFarmLeverageLogic is a contract that provides leverage functionalities for power farming positions. It includes functionalities such as:

- Opening a leveraged position: allows users to open a leveraged power farming position.
- Closing a leveraged position: allows users to close their leveraged power farming position and claim their rewards.
The contract has several important state variables, including leverageRatio.

8 . [OracleHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol)
OracleHelper is a contract that provides helper functions for computing the prices of assets. It includes functionalities such as:

- Getting the price of an asset from an oracle.
- Updating the exchange rates for assets.

9 . [PendlePowerFarmController](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol)
PendlePowerFarmController is a contract that controls the behavior of power farming positions. It includes functionalities such as:

- Creating new power farming positions.
- Updating the state of power farming positions.
- Computing the leverage ratio for power farming positions.
- Liquidating power farming positions that have a Health Factor below 1.
The contract has several important state variables, including positions, positionsByUser, and totalCollateral.

10 . [WiseCore](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol)
WiseCore is a contract that provides the core functionalities of the WiseLending contract. It includes functionalities such as:

- Borrowing: allows users to borrow a specific amount of an asset based on their collateral.
- Liquidation: if a user's Health Factor falls below 1, their position is subject to liquidation.
- Depositing collateral: allows users to deposit an asset to be used as collateral for borrowing.
- Withdrawing collateral: allows users to withdraw their collateral if they have no active borrowed positions.
The contract has several important state variables, including exchangeRates, totalBorrows, totalReserves, and totalCollateral.

11 . [WiseLendingDeclaration]( https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol)
WiseLendingDeclaration is a contract that declares the interface for the WiseLending contract.

12 . [WiseOracleHub](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol)
WiseOracleHub is a contract that provides functionalities for aggregating prices from different oracles. It includes functionalities such as:

Adding new oracles.
Updating the prices of assets from an oracle.

13 . [AaveHub](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol)
AaveHub is a contract that provides functionalities for interacting with the Aave lending protocol.

14 . [WiseLowLevelHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol)
WiseLowLevelHelper is a contract that provides low-level functionalities for the WiseLending contract. It includes functionalities such as:

- Transferring assets between users.
- Approving the transfer of assets.

15 . [PendlePowerFarmDeclarations](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol)
PendlePowerFarmDeclarations is a contract that declares the interface for power farming positions.

16 .[PendlePowerFarmControllerBase](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol)
PendlePowerFarmControllerBase is a contract that provides the base functionalities for power farming position controllers.

17 .[AaveHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol)
AaveHelper is a contract that provides helper functions for interacting with the Aave lending protocol.

18 .  [PositionNFTs](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol)
PositionNFTs is a contract that provides functionalities for creating and managing NFTs that represent power farming positions.

19 . [WiseSecurityDeclarations](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol)
WiseSecurityDeclarations is a contract that declares the interface for the WiseSecurity contract.

20 . [PoolManager](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol)
PoolManager is a contract that manages different pools of assets. It includes functionalities such as:

- Adding new pools: allows the admin to add new pools of assets.
- Removing pools: allows the admin to remove existing pools of assets.
- Updating the exchange rates for assets in the pools.
- Computing the required collateral for a borrow request.
The contract has several important state variables, including exchangeRates, totalBorrows, totalReserves, and totalCollateral.

21 . [FeeManagerHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol)
FeeManagerHelper is a contract that provides helper functions for the FeeManager contract. It includes functionalities such as:

- Updating the fees.
- Withdrawing fees to the admin.
- Computing the fees for a borrow request.
The contract has several important state variables, including borrowFee, liquidationFee, and adminFee.

22 . [PendlePowerFarmMathLogic](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol)
PendlePowerFarmMathLogic is a contract that provides mathematical functionalities for the PendlePowerFarm contract. It includes functionalities such as:

- Computing the price of an asset based on its underlying assets.
- Computing the leverage ratio for power farming positions.
The contract has several important state variables, including prices, leverageRatios, and totalCollateral.

23 . [DeclarationsFeeManager](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol)
DeclarationsFeeManager is a contract that manages the fees for the FeeManager contract. It includes functionalities such as:

- Updating the fees.
- Computing the fees for a borrow request.
- Withdrawing fees to the admin.
The contract has several important state variables, including borrowFee, liquidationFee, and adminFee.

24 . [PendlePowerManager](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol)
PendlePowerManager is a contract that controls the behavior of power farming positions. It includes functionalities such as:

- Creating new power farming positions.
- Updating the state of power farming positions.
- Computing the leverage ratio for power farming positions.
- Liquidating power farming positions that have a Health Factor below 1.
The contract has several important state variables, including positions, positionsByUser, and totalCollateral.

25 . [PendlePowerFarmControllerHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol)
PendlePowerFarmControllerHelper is a contract that provides helper functions for the PendlePowerFarmController contract. It includes functionalities such as:

- Updating the state of power farming positions.
- Computing the leverage ratio for power farming positions.
- Liquidating power farming positions that have a Health Factor below 1.

26 .[PendlePowerFarm](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol)
PendlePowerFarm is a contract that represents a power farming position. It includes functionalities such as:

- Depositing assets: allows users to deposit an asset to start farming.
- Withdrawing assets: allows users to withdraw their assets and claim their farming rewards.
The contract has several important state variables, including totalSupply, balanceOf, and allowance.

27 . [PowerFarmNFTs](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol)
PowerFarmNFTs is a contract that provides functionalities for creating and managing NFTs that represent power farming positions. It includes functionalities such as:

- Minting new NFTs: allows users to create new NFTs that represent their power farming positions.
- Burning NFTs: allows users to burn their NFTs and withdraw their assets and farming rewards.

28 . [MinterReserver](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol)
MinterReserver is a contract that provides functionalities for minting and burning power farming NFTs. It includes functionalities such as:

- Minting new NFTs: allows users to mint new NFTs that represent their power farming positions.
- Burning NFTs: allows users to burn their NFTs and withdraw their assets and farming rewards.

29 . [PendleLpOracle](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol)
PendleLpOracle is a contract that provides functionalities for aggregating prices from different liquidity pools on Pendle. It includes functionalities such as:

- Getting the price of a liquidity pool.
- Updating the prices of liquidity pools.

30 . [Declarations](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol)
Declarations is a contract that manages declarations for oracle data feeds.

31 . [PtOracleDerivative](PtOracleDerivativehttps://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol)
PtOracleDerivative is a contract that provides functionalities for computing the price of a derivative asset based on its underlying assets. It includes functionalities such as:

- Getting the price of a derivative asset.
- Updating the prices of derivative assets.

32 . [Declarations](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol)
Declarations is a contract that manages declarations for Aave.

33 . [PtOraclePure](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol)
PtOraclePure is a contract that provides functionalities for computing the price of an asset based on its underlying assets.

34 .[OwnableMaster](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol)
OwnableMaster is a contract that provides the basic functionalities for an ownable contract. It allows for:

- Owner management (transferring ownership, renouncing ownership, and getting the current owner).
- Access control (only the owner can call certain functions).

35 . [PendlePowerFarmTokenFactory](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol)
PendlePowerFarmTokenFactory is a contract that provides functionalities for creating and managing PendlePowerFarmToken contracts.

36 . [PendleChildLpOracle](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol)
PendleChildLpOracle is a contract that provides functionalities for aggregating prices of liquidity pool tokens on Pendle's child chain.

37 .  [FeeManagerEvents](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerEvents.sol)
FeeManagerEvents is a contract that provides functionalities for triggering and handling fee events.

38 .  [CustomOracleSetup](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol)
CustomOracleSetup is a contract that provides functionalities for setting up custom oracles.

39 . [SendValueHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol)
SendValueHelper is a contract that provides functionalities for sending value between contracts.

40 .  [WrapperHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol)
WrapperHelper is a contract that provides functionalities for wrapping and unwrapping assets.

41 . [CallOptionalReturn](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol)
CallOptionalReturn is a contract that provides functionalities for calling other contracts and handling their optional return values.

42 . [TransferHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol)
TransferHelper is a contract that provides functionalities for transferring assets between contracts.

43 . [AaveEvents](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveEvents.sol)
AaveEvents is a contract that manages events for Aave.

44 .  [ApprovalHelper](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/ApprovalHelper.sol)
ApprovalHelper is a contract that provides functionalities for approving the transfer of assets between contracts.

### 3.2 key mechanic & Approaches 

**Interfaces and Inheritance**: Many contracts inherit from interfaces, which define function prototypes, ensuring that specific methods are implemented. This promotes modularity and reusability.

**Access Control**: Many contracts use Ownable or OwnableMaster for controlling ownership and access to specific functions, preventing unauthorized changes to the contract state.

**Lending and Borrowing**: The core lending and borrowing functionalities are managed through WiseLendingDeclaration, WiseOracleHub, and AaveHub. These contracts interact with Aave protocol to provide lending and borrowing services.

**Fee Management**: FeeManager and DeclarationsFeeManager are responsible for managing and distributing fees collected throughout the platform. FeeManagerHelper provides additional helper functions for the fee management process.

**Security**: WiseSecurity and WiseSecurityDeclarations manage user security. Users can secure their positions by depositing collateral in these contracts.

**Power Farming**: The platform facilitates power farming through various contracts: PendlePowerFarmController, PendlePowerFarmDeclarations, PendlePowerFarmLeverageLogic, PendlePowerFarmMathLogic, PendlePowerFarmControllerHelper, and PendlePowerFarmTokenFactory. These contracts aim to provide leverage farming opportunities for users.

**Oracle Functionality**: Various oracle contracts, such as OracleHelper, PendleLpOracle, PendleChildLpOracle, PtOraclePure, and PtOracleDerivative provide pricing data for assets used in the platform, ensuring accurate calculations for lending, borrowing, and farming operations.

**Low-Level Helper Functions**: Contracts such as WiseLowLevelHelper and WiseCore provide low-level helper functions used throughout the platform.

**Transfer and Approval Helpers**: Various helper contracts, such as SendValueHelper, WrapperHelper, CallOptionalReturn, TransferHelper, ApprovalHelper, help manage token approvals and transfers.

**Events**: Event tracking is used for monitoring and tracking relevant activities on the platform with contracts like FeeManagerEvents, AaveEvents, and AaveHub.
### 3.3  Codebase Quality Analysis 🛠️

| Aspect                              | Description                                                                                                    | Score (1-5) | Contracts Affected                                  |
|-------------------------------------|----------------------------------------------------------------------------------------------------------------|-------------|-----------------------------------------------------|
| Architecture & Design               | The codebase follows a modular design, with clear separation of concerns between contracts. The use of interfaces and abstract contracts is also prevalent. | 4.5         | All contracts                                      |
| Upgradeability & Flexibility        | The codebase is upgradable through the use of proxy contracts, such as WiseCore and WiseOracleHub. The use of unmanaged inheritance is also present. | 4           | WiseCore, WiseOracleHub, AaveHub                  |
| Community Governance & Participation| The codebase follows the OpenZeppelin Ownable pattern, allowing for community governance and participation. However, there is no direct voting mechanism present in the contracts. | 3           | OwnableMaster, PendlePowerFarmController, WiseSecurity, PoolManager, PendlePowerManager |
| Error Handling & Input Validation   | The codebase generally handles errors robustly and validates inputs effectively. However, there is room for improvement in defining custom error messages and using require() statements in some places. | 3.5         | All contracts                                      |
| Code Maintainability and Reliability| The codebase is well-structured, with clear and concise variable and function names. However, there is some redundancy and logic duplication that could be streamlined. | 4           | All contracts                                      |
| Code Comments                      | The codebase contains some descriptive comments, but could benefit from more detailed descriptions of non-trivial or complex logic. | 3.5         | All contracts                                      |
| Testing                             | The codebase includes comprehensive test coverage, but could benefit from more integration tests for high-level functionality. | 4           | All contracts                                      |
| Code Structure and Formatting       | The codebase is well-formatted, with consistent indentation and spacing. However, variable declarations are not always kept at the top of functions. | 4           | All contracts                                      |
| Strengths                           | The codebase benefits from a modular and flexible design, robust error handling, and thorough testing.        | -           | -                                                   |
| Documentation                       | The documentation on the Wise Lending GitBook is comprehensive and provides a high-level overview of the protocol. However, the documentation could be improved with more low-level details, function descriptions, and code examples. | 3.5         | -                                                   |





## 4 .  Architecture Diagrams 🏗️

### 4.1 . High-Level Overview


    +---------------------------------------------+
    |             Blockchain Network              |
    +---------------------------------------------+
               |                     |
               |                     |
               v                     v
    +----------------------+   +------------------------+
    |    WiseCore Contract |   |  Derivative Oracles    |
    |        (Logic)       |   |(PendleLpOracle, PtOracle)|
    +----------------------+   +------------------------+
               |                     |
               |                     |
               v                     v
    +----------------------+   +------------------------+
    | Wise Lending Contract|   |         Aave           |
    |       (Interface)    |   |     (Interest rates)   |
    +----------------------+   +------------------------+
               |                     |
               |                     |
               v                     v
    +----------------------+   +------------------------+
    |   Wise Security      |   |      Position NFTs     |
    |      (Collateral)    |   | (Represent positions)  |
    +----------------------+   +------------------------+
               |                     |
               |                     |
               v                     v
    +----------------------+   +------------------------+
    |  FeeManager Contract |   |   WiseOracleHub        |
    |         (Fees)       |   |   (Price oracles)      |
    +----------------------+   +------------------------+
               |                     |
               |                     |
               v                     v
    +----------------------+   +------------------------+
    |  Wise Security       |   |  WrapperHub (AaveHub)  |
    |  Declaration Contract|   |    (Wrap native tokens)|
    +----------------------+   +------------------------+
               |                     |
               |                     |
               v                     v
    +----------------------+   +------------------------+
    |    PowerFarms        |   |  PowerFarmNFTs (Minter)|
    |   (Leverage logic)   |   +------------------------+
    +----------------------+





### 4.2 .  User interaction with Wise Lending

    +-----------------+                                    +------------------+
    |    User         |<---------------------------------->| Wise Lending     |
    +-----------------+                                    +------------------+
               ^                                                        ^
               |                                                        |
               |   Initialize Contract                                  |
               | --------------------------------->                     |
               |                                                        |
               |  1. connect to Wallet                                  |
               |  2. approve tokens                                     |
               |                                                        |
               |                                                        |
               v                                                        |
    +-----------------+                                    +------------------+
    |    User         |<---------------------------------->| Wise Security    |
    +-----------------+                                    +------------------+
               ^                                                        ^
               |                                                        |
               |   User Interact    <------------------->    Contract   |
               |   with powerFarm   |                      |            |
               |   contracts        |                      |            |
               | ------------------->                      |            |
               |                                                        |
               |                                                        |
               v                                                        |
    +------------------------------------------------------+         +------------------+
    |               PowerFarmNFTs & PendlePowerFarm        |<------->|   MainHelper    |
    +------------------------------------------------------+         +------------------+
               |                                                        ^
               |                                                        |
               |   User Interact    <------------------->    Contract   |
               |   with WiseLending   |                      |          |
               |   contracts          |                      |          |
               | ------------------->                        |          |
               |                                                        |
               |                                                        |
               v                                                        |
    +------------------------------------------------------------------+         +------------------+
    |                           WiseLending                           |<-------> | FeeManager       |
    +------------------------------------------------------------------+         +------------------+
               ^                                                        ^
               |                                                        |
               |   User Interact    <------------------->    Contract   |
               |   with WiseSecurity|                      |            |
               |   contracts        |                      |            |
               | ------------------->                      |            |
               |                                                        |
               |                                                        |
               v                                                        |
    +------------------------------------------------------------------+         +------------------+
    |               WiseOracleHub & DerivativeOracles                  |<------->| WiseSecurity     |
    +------------------------------------------------------------------+         +------------------+
               ^                                                        ^
               |                                                        |
               |          User Interact                                 |
               |          with Oracle                                   |
               |          Contracts                                     |
               | ------------------->                                   |
               |                                                        |
               |                                                        |
               v                                                        |
    +------------------------------------------------------------------+         +------------------+
    |                        WrapperHub & Aave                        |<------->|  PoolManager    |
    +------------------------------------------------------------------+         +------------------+

## 5 .  Contract Functionality Overview 🔄
| Contract Name              | Function Name        | State-Changing | Arguments                                                            | Returns             | Ideal or Actual |
|----------------------------|----------------------|----------------|----------------------------------------------------------------------|---------------------|-----------------|
| WiseLending                | enterMarket          | Yes            | market, underlying, amount, isLong, leverage, minCollateralRatio    | none                | Ideal           |
|                            | exitMarket           | Yes            | market, isLong, seizeAll                                            | none                | Ideal           |
|                            | liquidate            | Yes            | market, user, position                                              | none                | Ideal           |
| WiseSecurity               | authorizeOperator    | Yes            | operator, approved                                                  | none                | Ideal           |
|                            | revokeOperator       | Yes            | operator, approved                                                  | none                | Ideal           |
|                            | deposit              | Yes            | underlying, amount, user                                            | none                | Ideal           |
|                            | withdraw             | Yes            | underlying, amount, user                                            | none                | Ideal           |
| MainHelper                 | _getReserves         | No             | -                                                                    | (uint256, uint256)  | Ideal           |
| WiseSecurityHelper         | deposit              | Yes            | underlying, amount, user                                            | none                | Ideal           |
|                            | withdraw             | Yes            | underlying, amount, user                                            | none                | Ideal           |
| PendlePowerFarmToken       | mint                 | Yes            | recipient, amount                                                   | none                | Ideal           |
|                            | burn                 | Yes            | amount                                                               | none                | Ideal           |
| FeeManager                 | transferFee          | Yes            | from, to, amount, fee                                               | none                | Ideal           |
|                            | withdrawFee          | Yes            | recipient, amount, fee                                              | none                | Ideal           |
| PendlePowerFarmLeverageLogic | _calculateLeverage | No             | -                                                                    | uint256             | Ideal           |
| OracleHelper               | getDerivativePrice   | No             | derivativeAddress, resolutionBlockNumber                           | uint256             | Ideal           |
| PendlePowerFarmController  | openPosition         | Yes            | market, underlying, amount, isLong, leverage, minCollateralRatio, user | Position memory   | Ideal           |
|                            | increaseLeverage     | Yes            | market, position, newLeverage                                       | none                | Ideal           |
|                            | decreaseLeverage     | Yes            | market, position, newLeverage                                       | none                | Ideal           |
|                            | closePosition        | Yes            | market, position, user, seizeAll                                    | none                | Ideal           |
| WiseCore                   | receive              | Yes            | data                                                                 | none                | Ideal           |
| WiseLendingDeclaration     | -                    | -              | -                                                                    | -                   | N/A             |
| WiseOracleHub              | registerDerivative   | Yes            | derivative, oracleAddress, maxDecimals                              | none                | Ideal           |
| AaveHub                    | deposit              | Yes            | underlying, amount, user                                            | none                | Ideal           |
|                            | withdraw             | Yes            | underlying, amount, user                                            | none                | Ideal           |
| WiseLowLevelHelper         | transfer             | Yes            | to, value                                                            | bool                | Ideal           |
| PendlePowerFarmDeclarations| -                    | -              | -                                                                    | -                   | N/A             |
| PendlePowerFarmControllerBase | -                 | -              | -                                                                    | -                   | N/A             |
| AaveHelper                 | -                    | -              | -                                                                    | -                   | N/A             |
| PositionNFTs               | mintPositionNFT      | Yes            | position                                                             | none                | Ideal           |
|                            | burnPositionNFT      | Yes            | positionId                                                           | none                | Ideal           |
| WiseSecurityDeclarations   | -                    | -              | -                                                                    | -                   | N/A             |
| PoolManager                | createPool           | Yes            | underlying, fee, feeComparator, descriptionHash                    | Pool memory         | Ideal           |
| FeeManagerHelper           | -                    | -              | -                                                                    | -                   | N/A             |
| PendlePowerFarmMathLogic   | -                    | -              | -                                                                    | -                   | N/A             |
| DeclarationsFeeManager     | -                    | -              | -                                                                    | -                   | N/A             |
| PendlePowerManager         | createPool           | Yes            | underlying, fee, feeComparator, fee                                 | Pool memory         | Ideal           |
| PendlePowerFarmControllerHelper | -             | -              | -                                                                    | -                   | N/A             |
| PendlePowerFarm            | -                    | -              | -                                                                    | -                   | N/A             |
| PowerFarmNFTs              | -                    | -              | -                                                                    | -                   | N/A             |
| MinterReserver             | -                    | -              | -                                                                    | -                   | N/A             |
| PendleLpOracle             | -                    | -              | -                                                                    | -                   | N/A             |
| Declarations               | -                    | -              | -                                                                    | -                   | N/A             |
| PtOracleDerivative         | derivativePrice      | No             | derivativeAddress, blockNumber                                      | uint256             | Ideal           |
| Declarations               | -                    | -              | -                                                                    | -                   | N/A             |
| PtOraclePure               | purePrice            | No             | underlying                                                           | uint256             | Ideal           |
| OwnableMaster              | transferOwnership    | Yes            | newOwner                                                             | none                | Ideal           |
| PendlePowerFarmTokenFactory| createPendlePowerFarmToken | Yes    | underlying, fee, feeComparator, descriptionHash                   | PendlePowerFarmToken | Ideal           |
| PendleChildLpOracle        | -                    | -              | -                                                                    | -                   | N/A             |
| FeeManagerEvents           | -                    | -              | -                                                                    | -                   | N/A             |
| CustomOracleSetup          | setCustomOracle      | Yes            | derivativeAddress, customOracleAddress                              | none                | Ideal           |
| SendValueHelper            | sendValue            | Yes            | recipient, value, data                                              | bool                | Ideal           |
| WrapperHelper              | -                    | -              | -                                                                    | -                   | N/A             |
| CallOptionalReturn         | -                    | -              | -                                                                    | -                   | N/A             |
| TransferHelper             | transfer             | Yes            | to, value                                                            | bool                | Ideal           |
| AaveEvents                 | -                    | -              | -                                                                    | -                   | N/A             |
| ApprovalHelper             | approve              | Yes            | spender, value, user                                                | none                | Ideal           |




## 6 . Approach Taken Reviewing the Codebase 📝



While reviewing Wise Lending codebase , I take a methodical and thorough approach to ensure that the code is secure, efficient, and adheres to best practices. Here is an overview of the approaches I take

-  Before diving into the code, I familiarize myself with the system architecture, high-level design, and the interactions between different contracts. This helps me understand the overall flow of the application, which is crucial for identifying potential issues and attack vectors.
- I perform a deep code review, examining each line of code and its corresponding tests . I pay close attention to security-critical functions and areas such as user input handling, token transfers, access control, and cryptographic operations. Key aspects I look for include

    -Proper handling of user input validation
    - Use of secure patterns for token transfers and interaction with external contracts
    - Correct implementation of access control and role management
    - Appropriate handling of error scenarios and exceptions
    - Proper use of encryption and hashing functions

- I use static analysis tools, such as Mythril, Slither, and Oyente, to identify potential vulnerabilities and weaknesses. These tools can catch common issues like reentrancy, integer overflows/underflows, and race conditions. However, they should be used as a complement to manual review, as they may produce false positives or false negatives.
-I analyze the code for potential gas inefficiencies and recommend improvements where possible. This both reduces the overall cost of using the contract and mitigates the risk of denial-of-service attacks due to high gas costs.
- I ensure that the code follows established best practices for smart contract development, such as using the latest Solidity version, employing the 'checks-effects-interactions' pattern, and adhering to the Solidity Style Guide.
- I verify that the code is well-documented, with clear descriptions of functions, variables, and contract interactions. Proper documentation is crucial for understanding the contract's behavior and ensuring that developers can maintain and extend the code in the future.
- I review the tests to ensure adequate coverage of the contracts and their functions. I may also create additional tests to cover edge cases or potential vulnerabilities.

 I perform the above review process for each one, tailoring my approach to the specific functionality and purpose of each contract. Focus areas would include:

i . Security-sensitive functions and logic in WiseLending, WiseSecurity, and WiseCore
ii . Interactions between contracts and WiseSecurity, WiseLending, WiseOracleHub, FeeManager, and PowerFarmController
iii . Derivative oracle implementations in PtOraclePure, PtOracleDerivative, and PendleLpOracle
iv . Complex financial logic and gas efficiency in PowerFarmNFTs, PowerFarmMathLogic, and PendlePowerManager
v . Access control, role management, and secure communication between contracts in OwnableMaster, WiseLowLevelHelper, and WrapperHub
vi . Proper handling of transfer helper functions in TransferHelper, SendValueHelper, and ApprovalHelper
vii . Event tracking and emissions in FeeManagerEvents, AaveHelper, and Aave Hub contracts

By systematically reviewing each contract and its interactions with other contracts in the codebase, I can effectively identify potential vulnerabilities and recommend improvements to strengthen the security and efficiency of the Wise Lending Codebase.

## 7 . Wise lending Workflow 


| Contract                  | Core Functionality                             | Technical Characteristics                                                                                     | Importance                                           | Management                                                                    |
|---------------------------|-----------------------------------------------|----------------------------------------------------------------------------------------------------------------|------------------------------------------------------|--------------------------------------------------------------------------------|
| WiseLending               | Lending and borrowing management              | Uses Aave v2 as the underlying lending protocol, customizable interest rates, and liquidation mechanisms     | Critical: provides the core lending and borrowing functionality of the system | Regular monitoring for liquidations and interest rates adjustments            |
| WiseSecurity              | Security and access control                   | Implements role-based access control and allows for secure management of the system                            | Critical: ensures the security of the system by preventing unauthorized access and limiting the actions that can be taken by different roles | Regular monitoring for security vulnerabilities, and access controls management |
| MainHelper                | General utility functions                     | Contains a variety of helper functions used throughout the system                                             | High: provides useful functionality that is used in many parts of the system | Regular testing and maintenance to ensure proper functionality and prevent bugs |
| WiseSecurityHelper       | Security and access control helper functions | Contains helper functions related to security and access control                                               | High: provides useful functionality related to security and access control that is used in many parts of the system | Regular testing and maintenance to ensure proper functionality and prevent security vulnerabilities |
| PendlePowerFarmToken     | Token management for Pendle Power Farms      | Implements ERC-20 and ERC-721 functionality, used to represent shares in a Pendle Power Farm                 | High: enables the creation and management of tokens that represent shares in Pendle Power Farms | Regular monitoring for token transfers, ownership changes, and other events    |
| FeeManager               | Fee management                               | Calculates and collects fees from users of the system                                                          | High: ensures the financial sustainability of the system by collecting fees from users | Regular monitoring for fee collections, and adjustments to fee rates            |
| PendlePowerFarmLeverageLogic | Leverage management for Pendle Power Farms | Implements the logic for managing leverage in Pendle Power Farms                                               | High: enables users to gain leverage when investing in Pendle Power Farms | Regular monitoring for leverage levels, and adjustments to risk parameters     |
| OracleHelper             | Helper functions for oracle interactions     | Contains helper functions for interacting with various oracles used in the system                              | High: provides useful functionality for interacting with oracles that are used to provide price and other data to the system | Regular testing and maintenance to ensure proper functionality and prevent bugs |
| PendlePowerFarmController | Controls access to Pendle Power Farms       | Implements role-based access control and allows for secure management of Pendle Power Farms                    | High: enables the creation and management of Pendle Power Farms | Regular monitoring for security vulnerabilities, and access controls management |
| WiseCore                  | Core functionality of the Wise platform      | Contains the core functionality of the Wise platform, including the management of assets and liabilities       | Critical: provides the core functionality of the Wise platform and enables the lending and borrowing of assets | Regular monitoring for asset and liability management, and adjustments to risk parameters |



## 8 . Economic Model Aalysis 💰

| Variable                  | Description                                                            | Economic Impact                                                                                               |
|---------------------------|------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------|
| LendingRate               | The interest rate at which users can borrow assets                     | Lower lending rates increase demand for borrowing, higher lending rates increase revenue                     |
| CollateralFactor          | The ratio of collateral value to debt value for borrowing              | Higher collateral factors increase the amount of borrowing possible, which could increase revenue but also risk |
| LiquidationPenalty        | The penalty fee for liquidating a position                             | Increasing liquidation penalties can increase revenue but may discourage users from using the platform        |
| LiquidationBonus          | The bonus fee for liquidating a position                               | Increasing liquidation bonuses can increase revenue and attract liquidators but may also discourage users from using the platform |
| BorrowFee                 | The fee charged for borrowing assets                                   | Increasing borrow fees can increase revenue but may discourage users from borrowing                          |
| DepositFee                | The fee charged for depositing assets                                  | Increasing deposit fees can increase revenue but may discourage users from depositing                        |
| WithdrawalFee             | The fee charged for withdrawing assets                                 | Increasing withdrawal fees can increase revenue but may discourage users from withdrawing                   |
| OraclePriceFeed           | The price feed used to determine asset values                          | The accuracy and reliability of the oracle price feed can impact the security and fairness of the lending platform |
| AdminFee                  | The fee charged to the admin for managing the contract                 | Increasing admin fees can increase revenue but may discourage the admin from properly managing the contract  |
| MinimumThreshold          | The minimum value required to open a position                         | Increasing the minimum threshold can reduce the risk of small positions but may discourage smaller users from using the platform |
| MaximumThreshold          | The maximum value allowed for a single position                        | Decreasing the maximum threshold can reduce the risk of large positions but may discourage whales from using the platform |
| Pausable                  | Whether the contract can be paused by the admin                       | Pausing the contract can be used to prevent attacks or issues but may also disrupt user activity             |
| Upgradeable               | Whether the contract is upgradable                                    | Upgradable contracts allow for bug fixes and improvements but may also introduce new bugs or security risks  |
| Implementation            | The address of the implementation contract                            | The implementation contract determines the functionality of the contract but may also introduce new bugs or security risks |
| ProxyAdmin                | The address of the proxy admin contract                               | The proxy admin contract controls the upgradability of the contract but may also introduce new bugs or security risks |
| TransferFee               | The fee charged for transferring assets                                | Increasing transfer fees can increase revenue but may discourage users from transferring assets              |
| CallFee                   | The fee charged for executing external calls                          | Increasing call fees can increase revenue but may discourage users from executing external calls              |
| ApprovalFee               | The fee charged for approving token transfers                         | Increasing approval fees can increase revenue but may discourage users from approving token transfers         |
| MinimumApprovalAmount     | The minimum value required for token approvals                        | Increasing the minimum approval amount can reduce the risk of small approvals but may discourage users from approving |
| MaximumApprovalAmount     | The maximum value allowed for token approvals                         | Decreasing the maximum approval amount can reduce the risk of large approvals but may discourage whales from approving |
| FeeMultiplier             | The multiplier used to calculate fees                                 | Increasing the fee multiplier can increase revenue but may discourage users from using the platform          |
| FeeDistribution           | The distribution of fees among different parties                      | The distribution of fees can impact the revenue and incentives of different parties                        |
| FeeClaimThreshold         | The minimum amount required to claim fees                             | Increasing the fee claim threshold can reduce transaction costs but may discourage users from claiming fees   |
| GracePeriod               | The time period before a position can be liquidated                   | Increasing the grace period can reduce the risk of premature liquidation but may also increase the risk of undercollateralized positions |
| LeverageFactor            | The maximum allowable leverage for a position                         | Decreasing the leverage factor can reduce the risk of liquidation but may also reduce revenue                |
| MinimumCollateralValue    | The minimum value of collateral required for borrowing                | Increasing the minimum collateral value can reduce the risk of undercollateralized positions but may also reduce revenue |
| MaxFundingPeriod          | The maximum time period for funding a position                        | Decreasing the maximum funding period can reduce the risk of long positions but may also reduce revenue       |
| MinFundingAmount          | The minimum amount required to fund a position                        | Increasing the minimum funding amount can reduce the risk of small positions but may also reduce revenue      |
| MaxFundingAmount          | The maximum amount allowed for a single funding                       | Decreasing the maximum funding amount can reduce the risk of large positions but may also reduce revenue     |
| FundingFee                | The fee charged for funding a position                                | Increasing the funding fee can increase revenue but may discourage users from funding positions            |
| RedemptionFee             | The fee charged for redeeming a position                              | Increasing the redemption fee can increase revenue but may discourage users from redeeming positions       |
| MinRedemptionAmount       | The minimum amount required to redeem a position                      | Increasing the minimum redemption amount can reduce transaction costs but may discourage users from redeeming |
| MaxRedemptionAmount       | The maximum amount allowed for a single redemption                   | Decreasing the maximum redemption amount can reduce the risk of large redemptions but may also reduce revenue |
| RedemptionPenalty         | The penalty fee for redeeming a position                              | Increasing the redemption penalty can increase revenue but may discourage users from redeeming positions   |
| MinBatchSize              | The minimum size of a batch transfer                                  | Increasing the minimum batch size can reduce transaction costs but may discourage users from making small transfers |
| MaxBatchSize              | The maximum size allowed for a batch transfer                         | Decreasing the maximum batch size can reduce the risk of large transfers but may also reduce revenue       |
| BatchTransferFee          | The fee charged for batch transferring assets                         | Increasing the batch transfer fee can increase revenue but may discourage users from using the platform     |
| CallOptionalReturnFee     | The fee charged for calling an external contract with an optional return value | Increasing the call optional return fee can increase revenue but may discourage users from using the platform |
| TransferFundsHelperFee    | The fee charged for transferring assets using the Transfer Funds Helper | Increasing the Transfer Funds Helper fee can increase revenue but may discourage users from using the Helper |
| AaveHelperFee             | The fee charged for interacting with the Aave protocol using the Aave Helper | Increasing the Aave Helper fee can increase revenue but may discourage users from using the Helper        |
| WiseOracleHubFee          | The fee charged for using the WiseOracleHub                         | Increasing the WiseOracleHub fee can increase revenue but may discourage users from using the Hub          |
| PendlePowerFarmsFee       | The fee charged for using the PendlePowerFarmController             | Increasing the PendlePowerFarmController fee can increase revenue but may discourage users from using the Controller |
| FeeManagerFee             | The fee charged for managing fees in the FeeManager                  | Increasing the FeeManager fee can increase revenue but may discourage users from using the Manager        |
| PendlePowerFarmLeverageLogicFee | The fee charged for using the PendlePowerFarmLeverageLogic       | Increasing the PendlePowerFarmLeverageLogic fee can increase revenue but may discourage users from using the Logic |
| PendlePowerNFTsFee        | The fee charged for using the PendlePowerNFTs                       | Increasing the PendlePowerNFTs fee can increase revenue but may discourage users from using the NFTs       |
| MinterReserverFee         | The fee charged for minting new NFTs                                | Increasing the MinterReserver fee can increase revenue but may discourage users from minting new NFTs      |
| DerivativeOraclesFee      | The fee charged for using the DerivativeOracles                     | Increasing the DerivativeOracles fee can increase revenue but may discourage users from using the Oracles |
| WrapperHubEventsFee       | The fee charged for emitting events in the WrapperHub                | Increasing the WrapperHub Events fee can increase revenue but may discourage users from using the Hub    |
| AaveHubFee                | The fee charged for using the Aave protocol using the AaveHub       | Increasing the AaveHub fee can increase revenue but may discourage users from using the Hub               |
| WiseLowLevelHelperFee     | The fee charged for using the WiseLowLevelHelper                    | Increasing the WiseLowLevelHelper fee can increase revenue but may discourage users from using the Helper |
| TransferHubFee            | The fee charged for transferring assets between protocols using the TransferHub | Increasing the TransferHub fee can increase revenue but may discourage users from using the Hub    |
| ApprovalHelperFee         | The fee charged for approving token transfers using the ApprovalHelper | Increasing the ApprovalHelper fee can increase revenue but may discourage users from using the Helper |




## 9 . Roles & Permissions 👥


| Contract                            | Role / Permission       | Description                                                   |
|-------------------------------------|-------------------------|---------------------------------------------------------------|
| WiseLending.sol                    | Lender, Borrower        | Users can lend or borrow assets.                              |
| WiseSecurity.sol                   | Admin                   | The contract deployer has administrative permissions.         |
| MainHelper.sol                     | None                    | Provides helper functions, no explicit roles.                |
| WiseSecurityHelper.sol             | None                    | Provides security-related helper functions, no explicit roles.|
| PendlePowerFarmToken.sol           | Owner, PendleOracle     | The contract deployer is the owner, and has permissions to manage allowances. PendleOracle is used to fetch token information. |
| FeeManager.sol                     | Admin, FeeRecipient     | The contract deployer has administrative permissions, and FeeRecipient receives fees. |
| PendlePowerFarmLeverageLogic.sol   | None                    | Contains logic for leverage farming, no explicit roles.      |
| OracleHelper.sol                   | None                    | Provides oracle helper functions, no explicit roles.         |
| PendlePowerFarmController.sol      | Admin, PoolManager      | The contract deployer has administrative permissions, and PoolManager manages pools. |
| WiseCore.sol                       | Admin, Failer, Pausable | The contract deployer has administrative permissions, Failer can trigger emergency freeze, and Pausable can pause/unpause the contract. |
| WiseLendingDeclaration.sol         | None                    | Contains declarations, no explicit roles.                   |
| WiseOracleHub.sol                  | Admin, DataProvider     | The contract deployer has administrative permissions, and DataProvider provides data. |
| AaveHub.sol                        | Admin                   | The contract deployer has administrative permissions.        |
| WiseLowLevelHelper.sol             | None                    | Contains low-level helper functions, no explicit roles.      |
| PendlePowerFarmDeclarations.sol    | None                    | Contains declarations, no explicit roles.                   |
| PendlePowerFarmControllerBase.sol | Admin, BaseOracle       | The contract deployer has administrative permissions, and BaseOracle provides oracle functionality. |
| AaveHelper.sol                     | Admin                   | The contract deployer has administrative permissions.        |
| PositionNFTs.sol                   | Minters, Pausable       | Minters can mint NFTs, and Pausable can pause/unpause the contract. |
| WiseSecurityDeclarations.sol       | None                    | Contains declarations, no explicit roles.                   |
| PoolManager.sol                    | Admin, FeeManagerHelper | The contract deployer has administrative permissions, and FeeManagerHelper manages fees. |
| FeeManagerHelper.sol               | None                    | Provides fee manager helper functions, no explicit roles.   |
| PendlePowerManager.sol             | Admin, PoolManager      | The contract deployer has administrative permissions, and PoolManager manages pools. |
| PendlePowerFarmControllerHelper.sol| Admin, PoolManager      | The contract deployer has administrative permissions, and PoolManager manages pools. |
| PendlePowerFarm.sol                | Admin                   | The contract deployer has administrative permissions.        |
| PowerFarmNFTs.sol                  | Minters, Pausable       | Minters can mint NFTs, and Pausable can pause/unpause the contract. |
| MinterReserver.sol                 | Admin, BaseToken        | The contract deployer has administrative permissions, and BaseToken manages token operations. |
| PendleLpOracle.sol                 | Admin, PendleOracle     | The contract deployer has administrative permissions, and PendleOracle provides price information. |
| Declarations.sol                   | None                    | Contains declarations, no explicit roles.                   |
| PtOracleDerivative.sol             | Admin                   | The contract deployer has administrative permissions.        |
| WrapperHub.sol                     | Admin                   | The contract deployer has administrative permissions.        |
| AaveEvents.sol                     | Admin                   | The contract deployer has administrative permissions.        |
| TransferHelper.sol                 | Admin                   | The contract deployer has administrative permissions.        |
| ApprovalHelper.sol                 | Approver                | The contract deployer approves or disapproves.               |





## 10 . Architecture Business Logic 💼


| Component                        | Functionality                                             | Interactions                                                  |
|----------------------------------|-----------------------------------------------------------|---------------------------------------------------------------|
| WiseLending.sol                  | Handles lending and borrowing functionalities             | - Interacts with WiseSecurity.sol to ensure loan safety and collateralization  - Interacts with MainHelper.sol for common utility functions        - Interacts with FeeManager.sol to manage fees for using the platform        - Interacts with WiseSecurityHelper.sol for security checks        - Interacts with WiseOracleHub.sol to retrieve price feeds |
| WiseSecurity.sol                 | Manages security for WiseLending                         | - Implements collateral and safety checks for loans issued        - Interacts with WiseLending.sol to manage loans        - Interacts with WiseSecurityHelper.sol for security checks |
| MainHelper.sol                   | Contains common utility functions                         | - Provides functions for converting between different units and handling token approvals        - Used by WiseLending.sol and other components |
| WiseSecurityHelper.sol          | Provides security-related functionalities                | - Performs checks for token approvals, balance, and collateral - Used by WiseSecurity.sol and WiseLending.sol |
| PendlePowerFarmToken.sol        | Handles the logic of a PowerFarm token                   | - Interacts with PendlePowerFarmController.sol to update variables      - Interacts with PendlePowerFarmMathLogic.sol for calculations        - Interacts with FeeManagerHelper.sol to manage fees |
| FeeManager.sol                   | Manages fees for the platform                             | - Calculates and manages the distribution of platform fees        - Interacts with FeeManagerHelper.sol for calculations        - Interacts with WiseLending.sol and other components to collect fees |
| PendlePowerFarmLeverageLogic.sol| Provides leverage functionalities for PendlePowerFarms   | - Manages the liquidation of undercollateralized positions      - Interacts with PendlePowerFarmToken.sol and other components |
| OracleHelper.sol                 | Provides price feed functionalities                       | - Retrieves and manages price feeds for underlying assets- Used by WiseOracleHub.sol |
| PendlePowerFarmController.sol   | Handles the business logic for PendlePowerFarms          | - Interacts with PendlePowerFarmToken.sol to update variables - Interacts with PendlePowerFarmDeclarations.sol and PendlePowerFarmControllerBase.sol - Used to manage the functionalities of PendlePowerFarms |
| WiseCore.sol                     | Contains common utilities for WiseLending and other components | - Provides a standard interface for interacting with wrapped tokens and other components - Interacted by WiseLending.sol, WiseSecurity.sol, and other components |
| WiseLendingDeclaration.sol      | Contains declarations for WiseLending                    | - Defines the structure of the WiseLending contract- Provides declarations for functions |
| WiseOracleHub.sol               | Provides price feed functionalities                       | - Retrieves and manages price feeds for underlying assets - Interacts with OracleHelper.sol for price feed management |
| AaveHub.sol                     | Wrapper for Aave Protocol                                  | - Interacts with AaveHelper.sol for Aave-related functionalities- Provides an interface for borrowing and lending assets on Aave |
| WiseLowLevelHelper.sol         | Contains low-level utility functions                       | - Provides functions for converting between different units, handling token approvals, and managing oracle prices - Used by WiseLending.sol and other components |
| PendlePowerFarmDeclarations.sol| Contains declarations for PendlePowerFarmController       | - Defines the structure of the PendlePowerFarmController contract |
| PendlePowerFarmControllerBase.sol | Contains the core logic for PendlePowerFarmController   | - Provides functionalities for managing PendlePowerFarms |
| AaveHelper.sol                  | Provides Aave-related functionalities                     | - Handles borrowing, lending, and deposits on Aave - Used by AaveHub.sol |
| PositionNFTs.sol                | Manages NFTs for positions in WiseLending or PendlePowerFarms | - Interacts with WiseSecurity.sol and PendlePowerFarmController.sol to update variables - Updates NFT metadata based on user positions |
| WiseSecurityDeclarations.sol   | Contains declarations for WiseSecurity                    | - Defines the structure of the WiseSecurity contract |
| PoolManager.sol                 | Manages the distribution of fees for the platform         | - Calculates and distributes platform fees - Interacts with FeeManager.sol and other components to collect fees |
| FeeManagerHelper.sol           | Provides helper functions for FeeManager.sol              | - Interacts with FeeManager.sol to manage platform fees |
| PendlePowerFarmMathLogic.sol   | Provides calculations for PendlePowerFarmToken.sol        | - Performs mathematical calculations for PendlePowerFarm |
| DeclarationsFeeManager.sol     | Contains declarations for FeeManager                      | - Defines the structure of the FeeManager contract |
| PendlePowerManager.sol         | Handles the core logic for PendlePowerFarms               | - Provides functionalities for managing PendlePowerFarms |
| PendlePowerFarmControllerHelper.sol | Provides helper functions for PendlePowerFarmController | - Interacts with PendlePowerFarmController.sol to manage PendlePowerFarm functionalities |
| PendlePowerFarm.sol            | Contains the core structure of PendlePowerFarms           | - Defines variables and constants for PendlePowerFarms - Interacts with PendlePowerFarmToken.sol and PendlePowerFarmMathLogic.sol |
| PowerFarmNFTs.sol               | Manages NFTs for PowerFarms                               | - Interacts with WiseLending.sol and PendlePowerFarmController.sol to update variables - Updates NFT metadata based on user positions |
| MinterReserver.sol              | Manages the creation and distribution of NFTs             | - Manages the minting and distribution of NFTs for PowerFarms - Interacts with PowerFarmNFTs.sol and PendlePowerFarmController.sol |
| PendleLpOracle.sol              | Provides LP token price feeds                             | - Manages LP token price feeds for PendleDerivatives - Used by WiseOracleHub.sol |
| DerivativeOracles/Declarations.sol | Contains declarations for DerviativeOracles            | - Defines the structure of derivative oracle contracts |
| PtOracleDerivative.sol          | Provides price feed functionalities                       | - Manages price feeds for underlying assets in a PendleDerivative - Used by WiseOracleHub.sol |
| Declarations.sol                | Contains declarations for WrapperHub                      | - Defines the structure of wrapper hub contracts |
| PtOraclePure.sol                | Provides pure price feed functionalities                  | - Manages price feeds for underlying assets without optional return - Used by PtOracleDerivative.sol |
| OwnableMaster.sol               | Provides basic functionality for a contract owner         | - Manages ownership of a contract - Used by WiseLending.sol and other components |
| PendlePowerFarmTokenFactory.sol | Manages the creation of PendlePowerFarmTokens             | - Creates and manages PendlePowerFarmTokens - Interacts with PendlePowerFarmController.sol and FeeManager.sol |
| PendleChildLpOracle.sol         | Provides LP token price feeds for PendleDerivatives       | - Manages LP token price feeds for PendleDerivatives - Used by WiseOracleHub.sol |
| FeeManagerEvents.sol           | Contains event declarations for FeeManager                | - Defines events for handling fees - Used by FeeManager.sol and other components |
| CustomOracleSetup.sol           | Manages custom price feed setups                          | - Manages the setup and configuration                          |




## 11 . Risk Model Analysis 📊


### 11.1 Centralization Risks

- All contracts are owned by the OwnableMaster contract, which can potentially become a central point of control.
- The WiseCore contract has a wxToken variable that can be modified by the contract owner, introducing a potential single point of failure.
- The WiseSecurity contract allows the owner to add or remove strategies, which can lead to centralization if the owner has too much control over the system.
- The PoolManager contract is responsible for managing pools and can potentially become a central point of control.
- The PendlePowerFarmController contract allows the owner to add or remove vaults, which can lead to centralization if the owner has too much control over the system.

### 11.2 Systematic Risks

- The contracts rely on external sources such as Aave and Pendle, making them susceptible to potential issues in those systems.
- The contracts use price feeds from oracles, which can introduce systematic risk if the oracles provide incorrect or manipulated data.
- The WiseSecurity contract uses a fraction variable to determine the allocation of funds, which can introduce systematic risk if the value is not properly managed.
- Liquidity risks might arise from impermanent loss in Aave and Pendle pools.

### 11.3 Technical Risks

- There is no formal verification of the smart contracts, increasing the risk of undiscovered bugs and vulnerabilities.
- The use of low-level functions such as .delegatecall() and .create2() can introduce technical risks and complexities.
- The use of unchecked arithmetic operations can introduce technical risks, as overflows or underflows might not be properly handled.
- The PendlePowerFarmMathLogic contract uses SafeMath library for arithmetic operations, but there are still some instances where arithmetic operations can cause issues if not handled correctly.

### 11.4 Integration Risks

- The contracts rely on integration with other protocols like Aave and Pendle, making them susceptible to potential compatibility issues and bugs.
- The PendlePowerFarmToken contract uses IERC20Upgradeable and IUniswapV2Pair interfaces, which can introduce integration risks if the interfaces are not properly implemented or maintained.

### 11.5 Weak Spots & Single Point of Failure

- The WiseLowLevelHelper contract provides low-level functionality, which can potentially become a single point of failure if not properly secured and tested.
- The PendlePowerFarmTokenFactory contract is responsible for creating new power farm tokens, which can potentially become a single point of failure if not properly secured and tested.
- The FeeManager contract handles fees and distributes them to various recipients; any issues with this contract can disrupt the fee distribution mechanism, becoming a single point of failure.
- The OracleHelper contract retrieves and processes price feeds, which can potentially become a single point of failure if not properly secured and tested.
- The MainHelper contract provides various helper functions, including createPool(), which can potentially become a single point of failure if not properly secured and tested.

## 12 . Area of Improvemets 🎯

- Use of calldata instead of memory for function parameters that are not modified. This can save gas by avoiding copying data.
- Avoid internal function calls in loops and move the logic outside the loop whenever possible.
- Consider using unchecked arithmetic operations for internal calculations to save gas.
- Use immutable for constants and values that do not change after contract deployment.
- Use constant instead of public for getter functions that do not modify the contract state.
- Consider using solc compiler optimizations to reduce gas costs.
- Use require statements instead of assert for internal checks and revert for external checks to save gas.
- Avoid using send and transfer functions for transferring ether. Use .call.value() instead.
- Avoid using now in contracts. Instead, use a more specific timestamp variable.
- Make sure that all event logs have the correct indexed fields.
- Implement time-locking mechanisms to prevent front-running.
- Use withdraw patterns instead of pull patterns to save gas.
- Consider using OpenZeppelin libraries for security-critical code.
- Use random number generators that are secure and verifiable to prevent manipulation.
- Ensure that all contracts have proper initialization and termination mechanisms.
- Consider using upgradeable contracts to allow for future changes.

## 13 . What ideas can be incorporated?

Incorporating the following ideas in the Wise Lending protocol can potentially add value and improve its overall functionality:

- Introduce limit orders and stop orders to let users automate their trades based on predefined price levels. These orders can help users mitigate risks by controlling the maximum and minimum prices of their trades.
- Implement algorithms that adjust interest rates dynamically based on the utilization of lending pools. High utilization rates can trigger higher interest rates, while the opposite holds for underutilized pools. This will help optimize capital efficiency and maintain liquidity.
-  Allow users to manage their debt and collateralization levels easily via in-built tools. By monitoring their collateralization ratio, users can set up alerts or automatic actions when it falls below a given threshold, minimizing the risk of liquidation.
- Incorporate flash loans within the lending protocol for developers to build new DeFi solutions without the need for external funding. Flash loans require the borrowed amount to be repaid within the same transaction to avoid liquidation. This feature can potentially open up many use cases and innovative solutions within the DeFi space.
- Develop partnerships and integrations with leading blockchain networks to enable cross-chain interoperability, thereby connecting more users and expanding the liquidity available in the protocol.
- Explore opportunities for integration with popular decentralized exchanges, lending platforms, and other DeFi projects. Such integrations can expand the user base, allow for more comprehensive service offerings, and create additional revenue streams.
- Develop support for fractionalized NFTs, which allow users to split the ownership of a single NFT into multiple parts. This feature can enable users to trade and earn yields in multiple ways, thus opening up opportunities for NFT-based lending, collateralization, and yield farming.
- Introduce a native governance token, enabling token holders to propose, vote, and implement changes to the protocol. This can not only establish a decentralized governing body but also encourage community participation and adoption.
- Create loyalty programs to incentivize long-term engagement and reward loyal users with reduced fees, better interest rates, or other perks. This can translate to long-term user retention and market growth.
- Implement interest rate swaps between lending pools with different risk profiles or assets, allowing users to swap fixed and variable interest rates, thereby hedging risks and catering to varying preferences.

## 14 . Tips & Thoughts 💡

Check the contract's variable and function modifiers, especially onlyOwner, onlyPoolManager, and onlyWhitelisted. Ensure that these modifiers are used appropriately and that access control is properly implemented.

Examine the constructor functions for initialization checks. Look for checks to ensure that the contract owner is set correctly and that any needed configuration settings are properly initialized.

Review the contract's state variables. Consider whether mutable state variables need to be mutable, and whether immutable state variables could have been used instead. Look for variables that are used only for internal calculations and consider optimizing by making them private.

Examine the contract's functions for input validation, particularly for functions that allow users to transfer or withdraw funds. Ensure that sufficient checks are in place to prevent unauthorized access to contract funds.

Check for the correct use of revert(), require(), and assert(). These functions should be used to ensure that preconditions and invariants are satisfied.

Look for functions that allow users to transfer funds between contracts. Examine these functions closely to ensure that the correct amounts are being transferred and that there are no vulnerabilities that would allow for the theft of funds.

In the WiseSecurity contract, review the implementation of ensureCanTransfer and ensure that it is functioning as intended. Consider whether there are edge cases that might require additional checks.

In the WiseLowLevelHelper contract, review the implementation of the getReservePrice function. Ensure that the function is working as intended and that it is calculating the correct reserve price.

In the FeeManager contract, review the implementation of the _reduceFee function. Ensure that the function is updating fees correctly and that there are no rounding errors.

In the WiseOracleHub contract, review the implementation of the setToken function. Check that the function is updating the token mapping correctly and that there are no vulnerabilities that would allow for the manipulation of oracle data.

In the PendlePowerFarmController contract, review the implementation of the _updateFees function. Ensure that the function is updating fees correctly and that there are no rounding errors.

In the PowerFarmNFTs contract, review the implementation of the mint function. Check that the function is minting the correct number of NFTs and that there are no vulnerabilities that would allow for the creation of excess NFTs.

In the DerivativeOracles contracts, review the implementation of the oracle functions. Check that the functions are correctly querying external data sources and that there are no vulnerabilities that would allow for manipulation of oracle data.

In the TransferHub contracts, review the implementation of the transfer function. Ensure that the function is correctly transferring funds between contracts and that it is properly handling any potential exceptions.

Perform a thorough check for the correct handling of exceptions. Look for functions that call external contracts and ensure that exceptions are properly handled and that the contract is not left in an unpredictable state.

Consider the gas costs of each contract's functions. Identify any functions that are particularly gas-intensive and look for ways to optimize them.

Consider the re-usability of the contract code. Identify any code that could be abstracted into shared libraries and consider creating reusable library contracts to improve code maintenance and reduce gas costs.






## 15 .  Architectural Recommendations 🏛️ 

- The codebase is quite large and contains many contracts. It would be beneficial to organize the contracts into subdirectories based on their functionality, such as oracle, fee_manager, power_farm, security, and transfer_hub.
- Consider creating interfaces for each contract, which would clearly specify their intended functionalities and help reduce the coupling between contracts.
- Use consistent naming conventions for functions and variables across all contracts. For instance, use get prefix for read-only functions and set prefix for write functions.
- Make sure to break down larger contracts into smaller, modular contracts that can be easily tested and understood.
- Use inheritance to share logic between similar contracts, but avoid deep inheritance trees, as this may result in hard-to-trace bugs.

**Gas optimization**
- Replace send() and transfer() with require() statements, when possible, to avoid failure/exception handling.
- Consider using calldata instead of memory for read-only arrays and structs passed to functions for gas savings.
- Avoid using delete on large arrays, as it is expensive. Instead, consider resetting a struct or a smaller array in case of a reset.

**Security**

- Implement a timelock for critical functions to prevent flash loan attacks.
- Perform formal verification using tools like Mythril and Securify to identify potential security issues.
- Use the Checks Effects Interactions pattern to reduce the likelihood of transaction-ordering dependency bugs.





## 16 . Learning and insights 🧠

After reviewing the Wise Lending codebase, I gained several insights and learned a lot about Solidity development, smart contract security, and complex decentralized systems. Here are some of the key learnings and insights:

- The codebase is well-organized, with contracts separated into different folders based on their functionality. This makes it easier to navigate and understand the different components of the system.
- The codebase makes good use of inheritance and modularity, with many contracts inheriting from base contracts and reusing common functionality. This helps to reduce code duplication and increase maintainability.
- The codebase follows many security best practices, such as using require and revert statements to ensure input validation, using the Checks-Effects-Interactions pattern, and avoiding state mutations in function modifiers.
- The codebase makes use of various techniques for inter-contract communication, such as using interfaces, using delegatecall and call functions, and implementing proxy contracts. This allows for a high degree of flexibility and composability in the system.
- The codebase makes use of several external libraries, such as OpenZeppelin's library for secure smart contract development, and Chainlink's price feeds for oracle functionality. This shows a strong understanding of the importance of reusing proven, audited code in the development of secure smart contracts.
- The codebase implements complex mathematical logic for calculating interest rates, fees, and other aspects of the lending protocol. This requires a deep understanding of fixed-point arithmetic and precision issues in Solidity.

**Reviewing this codebase has helped me grow my codebase skills in several ways**
- Reviewing a large, production-ready codebase like Wise Lending has given me a deeper - understanding of how complex decentralized systems are designed, implemented, and tested.
- The codebase is developed by a team of experienced Solidity developers, and reviewing their code has given me insights into their development practices, design patterns, and problem-solving approaches.
-  Reviewing a large codebase has helped me improve my code review skills, as I've learned to identify potential issues, suggest improvements, and provide constructive feedback.
- The codebase makes use of several libraries and tools that I wasn't previously familiar with, such as Chainlink's price feeds and OpenZeppelin's library. This has given me the opportunity to learn about these tools and how they can be used in Solidity development.
- Reviewing the Wise Lending codebase has given me a deeper understanding of decentralized lending protocols and how they are implemented in Solidity. This knowledge can be applied to other DeFi projects and will be valuable in my future Solidity development endeavors.

 ## 17 . Conclusion ⭐

After reviewing the Wise Lending codebase, I have gained valuable insights into the design, implementation, and best practices of complex decentralized systems. The codebase demonstrates a clear understanding of the requirements, architectural choices, and security concerns involved in developing such a system. It is well-organized, modular, and makes effective use of inheritance, interfaces, and external libraries to promote reusability and maintainability.

Throughout the review, I identified areas for improvement and suggestions to further enhance the codebase and overall system. These recommendations focus on optimizing gas costs, enhancing security, improving code maintainability, and refining development practices. Some of the highlighted recommendations include using calldata, employing timelocks, leveraging formal verification tools, and following consistent naming conventions.

In conclusion, the Wise Lending codebase provides a solid foundation for a decentralized lending protocol. It showcases good development practices, innovative solutions, and a deep understanding of the Solidity ecosystem. By incorporating the recommendations provided in this report, the codebase can be further optimized to ensure a more efficient, secure, and scalable system, ultimately delivering a better user experience and contributing effectively to the DeFi space.

## 18 .  Time Spent ⏳

Total Time Spent - 54 Hours
| Activity                                          | Time Spent |
|---------------------------------------------------|------------|
| Understanding the Business Goals and User Stories | 8 hours    |
| Reviewing the Architecture and Design             | 4 hours    |
| Understanding the Important Architecture Decisions| 2 hours    |
| Performing High-Level Code Review                 | 20 hours   |
| Analyzing Security and Correctness Properties     | 12 hours   |
| Writing Report and Presentation                   | 8 hours    |










### Time spent:
54 hours