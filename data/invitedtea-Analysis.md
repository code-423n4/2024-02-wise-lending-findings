- [Project Description](#project-description)
  - [**Wise Lending Protocol**](#wise-lending-protocol)
- [Key Modular in Wise Lending:](#key-modular-in-wise-lending)
  - [**Power Farms**](#power-farms)
    - [Overview](#overview)
    - [Key Function](#key-function)
    - [Mechanisms](#mechanisms)
    - [Operational Requirements](#operational-requirements)
    - [Risk Analysis and Security Considerations](#risk-analysis-and-security-considerations)
    - [System Design Security](#system-design-security)
  - [**Pendle Strategies Module**](#pendle-strategies-module)
    - [Overview](#overview-1)
    - [Key Functions](#key-functions)
    - [Mechanisms](#mechanisms-1)
    - [Operational Requirements](#operational-requirements-1)
    - [Risk Analysis and Security Considerations](#risk-analysis-and-security-considerations-1)
    - [System Design Security](#system-design-security-1)
  - [**Oracle Module**](#oracle-module)
    - [Overview](#overview-2)
    - [Key Functions](#key-functions-1)
    - [Mechanisms](#mechanisms-2)
    - [Operational Requirements](#operational-requirements-2)
    - [Risk Analysis and Security Considerations](#risk-analysis-and-security-considerations-2)
    - [System Design Security](#system-design-security-2)
- [Approach](#approach)
- [Audit Practices for this Project](#audit-practices-for-this-project)
    - [Codebase Quality](#codebase-quality)
- [Systemic \& Centralization Risks](#systemic--centralization-risks)
- [Recommendations](#recommendations)
- [Final Thoughts](#final-thoughts)
  - [Time spent:](#time-spent)



## Project Description
### **Wise Lending Protocol**
The Wise Lending Protocol, part of the Wise Ecosystem, is a decentralized finance (DeFi) platform focused on sustainability, allowing users to earn passive income by depositing crypto into lending pools or borrow using competitive yield-farming strategies, with APY paid in the same asset deposited, not relying on inflationary tokens for rewards.
This diagram comes from Opus Official Doc: ![image](https://wisesoft.gitbook.io/~gitbook/image?url=https:%2F%2F682390371-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FCGOqkoMdDdslYTrv5Ltr%252Fuploads%252FWNyBoEG4Tv4Z2nGSTD16%252FThe%2520Wise%2520Ecosystem.png%3Falt=media%26token=a83fc0c9-53b6-4532-9802-4ce74739aa0b&width=768&dpr=1&quality=100&sign=3986c57f3d5d6436559fa14d422da154a42594fee427e45a91bd0671207311dc)

Key features and highlights:
- **Decentralized and Sustainable**: Wise Lending emphasizes a sustainable financial ecosystem within the crypto space, focusing on long-term viability over short-term trends.
- **No Inflationary Rewards**: Unlike many DeFi platforms, Wise Lending does not use its $WISE token to pay out APY rewards, avoiding hyper-inflation and ensuring the token's value is backed by actual assets (ETH) and dApp revenue.
- **Community-Centric**: The Wise DAO ensures that revenue generated from dApps benefits the community of token holders rather than VCs or insiders, promoting a fair and decentralized governance structure.
- **Passive Earning**: Users can deposit cryptocurrencies into decentralized lending pools to earn passive income, with APY paid in the same asset they deposit.
- **Competitive Yield-Farming**: Offers borrowers access to competitive yield-farming strategies, leveraging trusted and proven sources in the crypto yield space.
- **WISE Token Backed by ETH**: The $WISE token is backed by a pool of ETH, correlating its price to ETH and allowing for appreciation based on dApp revenue, which creates a deflationary effect.
- **Open Ecosystem Participation**: Projects with a token can join the Wise Ecosystem by pairing their token with WISE for liquidity, benefiting from the ecosystem's value creation.


## Key Modular in Wise Lending:


### **Power Farms**

#### Overview

Power Farms within the Wise Lending ecosystem offer a one-click yield-farming strategy that utilizes the liquidity from Wise Lending's platform to amplify yields on selected Pendle zero-impermanent-loss LPs. Their primary role is to generate a consistent and significant demand for borrowed funds from Wise Lending, ensuring high returns for lenders by addressing the common issue of low borrowing demand found in traditional lending platforms. Power Farms are designed to be safe, utilizing decentralized and battle-tested yield sources, protecting leveraged positions against liquidations with mechanisms in place for managing risks associated with black swan events or negative APY scenarios. They offer yields in the same type of asset as deposited, avoiding speculative tokens, ensuring scalability and sustainability without compromising on security or decentralization.

#### Key Function
- **PendlePowerFarmController**: Manages the creation and administration of yield farming strategies.
- **PendlePowerFarmToken**: Represents the yield farming positions, likely allowing users to stake and earn rewards.
- **SimpleERC20Clone**: A simplified ERC20 token contract, possibly used for creating custom tokens within the ecosystem.
- **DeployLibrary & ContractLibrary**: Provide utility functions and constants for contract deployment and interaction.

#### Mechanisms
- **Yield Farming**: The contracts facilitate one-click yield farming strategies, leveraging liquidity from the lending platform to enhance yields.
- **Token Management**: Through `PendlePowerFarmToken` and `SimpleERC20Clone`, the system manages token creation, staking, and reward distribution.
- **Decentralized Control**: The use of a controller pattern (`PendlePowerFarmController`) and possibly governance mechanisms to manage the yield farming strategies.

#### Operational Requirements
- **Solidity 0.8.24**: Contracts are written in Solidity version 0.8.24, requiring a compatible Ethereum virtual machine for deployment.
- **External Dependencies**: The contracts interact with external protocols or contracts, such as Pendle and possibly others like Aave, for yield sources and liquidity management.
- **Administrative Functions**: The presence of controller and factory patterns suggests that certain administrative privileges are required for contract deployment, strategy creation, and parameter adjustments.

#### Risk Analysis and Security Considerations
- **Smart Contract Risks**: As with any DeFi protocol, risks include smart contract vulnerabilities, such as reentrancy attacks, improper access controls, and issues related to upgradeability (if applicable).
- **Yield Source Risks**: Dependence on external protocols like Pendle for yield generation introduces risks related to those protocols, such as smart contract failures, liquidity issues, or changes in protocol rules.
- **Impermanent Loss**: While the documentation suggests strategies to mitigate impermanent loss, any strategy involving liquidity provision is subject to this risk, especially in volatile market conditions.

#### System Design Security
- **Modular Design**: The separation of concerns, as seen in different contracts for tokens, controllers, and libraries, helps in isolating functionalities and reducing the attack surface.
- **Error Handling**: The use of custom error messages (e.g., in `SimpleERC20Clone`) improves the clarity of failure modes and aids in debugging and risk mitigation.
- **Upgradeability and Ownership**: The use of a controller and factory pattern may indicate an upgradeable design, which requires careful management of ownership and administrative functions to prevent unauthorized changes.


### **Pendle Strategies Module**

#### Overview

Pendle Strategies focus on enhancing the returns of yield-bearing tokens by adding a unique layer of yield on top of the existing one. Pendle achieves this by separating the yield component from the principal of yield-bearing tokens, allowing users to engage in yield speculation by trading either component. This process generates transaction fees, serving as an additional reward layer for holders. Despite using their own token for rewards, which is generally not favored, Pendle's model remains valuable due to the fees generated from yield speculation. Pendle adheres to true DeFi principles with fully decentralized, non-upgradable contracts and uses factory contracts for pool creation, ensuring security and integrity. However, the safety of Pendle pools relies on the security of the underlying yield sources. Pendle Power Farms, utilized by Wise Lending, prefer LPs over leveraging PT to mitigate liquidation risks associated with fluctuations in PT value. While Pendle Power Farms incur a 1-3% initiation fee, these fees contribute to higher APYs by accumulating staked Pendle tokens (vePENDLE), benefiting all users in the ecosystem.

#### Key Functions
- **getApproxNetAPY**: Estimates the net APY for a given position setup in `PendlePowerFarm.sol`, without performing a syncPool.
- **_executeBalancerFlashLoan**: Encapsulates the logic for preparing and executing a Balancer flash loan in `PendlePowerFarmLeverageLogic.sol`.
- **_updatePools**: Updates the pool logic through Wise Lending interfaces in `PendlePowerFarmMathLogic.sol`.
- **Constructor in `PendlePowerFarmTester.sol`**: Initializes the contract with necessary addresses and parameters for testing purposes.
- **Inheritance in `PendlePowerManager.sol`**: Demonstrates reliance on multiple inherited functionalities (`OwnableMaster`, `PendlePowerFarm`, `MinterReserver`) for managing the Pendle Power Farms.

#### Mechanisms
- **Yield Augmentation**: Enhances the APY of yield-bearing tokens by creating an additional layer of yield through yield speculation and transaction fees.
- **Leverage and Flash Loans**: Utilizes leverage and flash loans to optimize yield farming strategies and manage risks associated with impermanent loss.
- **Decentralized Contract Management**: Leverages fully decentralized contracts, factory contracts for pool generation, and non-upgradable proxies to ensure security and trust in the system.

#### Operational Requirements
- **Solidity Version**: Contracts are written in Solidity version 0.8.24, indicating the need for a compatible execution environment.
- **Decentralized Finance Standards**: Adheres to true DeFi principles with fully decentralized contracts, emphasizing the importance of non-upgradable proxies and factory contract mechanisms for pool creation.
- **External Interfaces**: Integrates with external protocols and interfaces, such as Aave and Pendle, to access yield sources and liquidity management tools.

#### Risk Analysis and Security Considerations
- **Underlying Yield Source Security**: The security of Pendle pools depends on the security of the underlying yield sources, with a careful selection of blue-chip assets to minimize risk.
- **Leverage and Liquidation Risks**: Mitigates risks associated with leveraging positions, especially in scenarios where yield token values fluctuate significantly.

#### System Design Security
- **Non-Upgradable Proxies**: Enhances security by using non-upgradable contracts to prevent unauthorized changes post-deployment.
- **Factory Contracts for Pool Generation**: Mimics Uniswap's approach to pool creation, contributing to the ecosystem's robustness and trustworthiness.
- **Comprehensive Error Handling and Modularity**: Likely incorporates detailed error handling and a modular design to isolate functionalities, reduce the attack surface, and facilitate debugging and risk mitigation.

### **Oracle Module**

#### Overview
Wise Lending use the Uniswap TWAP as a sanity check. If either price feed deviates by more than 2.5% of each other, it triggers a freeze on that asset. During a freeze, lenders may unwind leverage, but may not borrow. This state continues for 30 minutes and repeats until the price feeds are once again within 2.5% of each other.

#### Key Functions
- **UsdcEthOracle & UsdtEthOracle**: These contracts provide price feeds for USDC and USDT in terms of ETH by taking Chainlink oracle values and adjusting them based on the ETH/USD rate.
- **WstETHOracle**: Delivers a price feed for WstETH by using Chainlink oracles to adjust the stETH amount for 1E18 WstETH, ensuring accurate ETH valuation.
- **PendleTokenCustomOracle & PendleLpOracle**: These contracts likely provide customized oracle services for Pendle tokens and LP tokens, integrating with Chainlink oracles and possibly incorporating TWAPs (Time-Weighted Average Prices) and other metrics for valuation.
- **PtOraclePure & PtOracleDerivative**: Designed to supply price feeds for pure and derivative PT tokens, these contracts might utilize Chainlink oracles and TWAPs to ensure accurate pricing in ETH.

#### Mechanisms
- **Chainlink Integration**: The extensive use of Chainlink oracles across these contracts indicates reliance on external, reliable data sources for accurate price feeds.
- **TWAP Utilization**: The incorporation of TWAPs suggests a mechanism to smooth out price feeds over time, reducing the impact of short-term volatility.

#### Operational Requirements
- **Solidity Version 0.8.24**: Consistent use of this Solidity version across all contracts indicates the need for a compatible execution environment.
- **Chainlink Oracles**: Dependence on Chainlink's decentralized oracle network for real-time price data.
- **External Interface Compliance**: These contracts must adhere to the interfaces of Chainlink oracles and potentially other DeFi protocols like Pendle.

#### Risk Analysis and Security Considerations
- **Oracle Manipulation Risk**: Reliance on external oracles introduces the risk of price manipulation, which could impact the contracts' operations.
- **Smart Contract Dependencies**: Dependencies on external contracts like Chainlink oracles and Pendle increase the risk profile, as vulnerabilities in these contracts could affect the oracle contracts.

#### System Design Security
- **Decentralized Oracles**: The use of Chainlink, a decentralized oracle network, enhances the security and reliability of the price feeds.
- **Immutable Logic**: The apparent absence of upgradeable proxy patterns in these contracts suggests an immutable design, reducing the risk of unauthorized changes.
- **Error Handling**: The use of custom errors in contracts like `PtOraclePure.sol` and `PtOracleDerivative.sol` enhances clarity and security by providing explicit failure modes.



## Approach
The review process focused on examining the functionality and security features of the Wise Protocol's smart contracts. The audit emphasized the following areas:
1. **Contract Logic and State Management**
2. **Interest Rate Adjustment Mechanism**
3. **Liquidation System and Stability Pool**


## Audit Practices for this Project
- **Automated Scanning**: Utilized automated tools to pinpoint standard vulnerabilities.
- **Manual Review**: Performed an in-depth, line-by-line examination of the smart contracts to grasp the intricacies of the system and identify potential edge cases.

#### Codebase Quality
The codebase is well-structured, showcasing logical modularization. It's evident that each module is designed to encapsulate distinct functionalities of the protocol, enhancing readability and maintainability.

## Systemic & Centralization Risks
The Wise Protocol design indicates a dependency on a trusted admin role for crucial access control functions, introducing a central point of trust and thereby a risk of centralization.

## Recommendations
1. **Fallback Oracle**: A secondary oracle should be considered to maintain consistent reliability of the price feeds.
2. **Interest Accrual Optimization**: The process of accruing interest could be refined to ensure timeliness, particularly during protocol shutdown phases.

## Final Thoughts
The Wise Protocol's smart contract ecosystem is commendably organized, with well-defined components dedicated to specific protocol operations. The dynamic nature of the interest rate adjustments and market-responsive liquidation mechanisms are indicative of a robust DeFi offering. Advancing towards decentralized governance and incorporating a fallback oracle mechanism would significantly augment the protocol's resilience and user trust.



### Time spent:
32 hours

### Time spent:
32 hours