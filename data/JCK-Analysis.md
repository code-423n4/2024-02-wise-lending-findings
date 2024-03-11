
# Analysis - Wise Lending Contest

##  System Overview

The Wise Ecosystem is designed to create a sustainable, decentralized, and competitive financial environment within the crypto space. It aims to offer a platform where users can invest their crypto assets to earn long-term returns, focusing on sustainability and community value over traditional venture capital (VC) funding models. The system is built around several key components:
WISE is deflationary, with revenue from dApps boosting its value. It is not used for paying out APY rewards to dApp users, does not have whale wallets, and is not speculative. Its value is backed by a pool of ETH and correlates with the price of ETH.
The WISE DAO: This decentralized autonomous organization (DAO) is unique in that it drives value to the community rather than to VCs or founders. The founder and development team funded the project themselves, without VCs, ensuring that revenue from dApps is owned by the community of token holders.
WiseLending.com: The flagship dApp of the Wise Ecosystem, offering a platform for lending, borrowing, and earning. It allows users to deposit crypto into decentralized lending pools to earn passive income and provides competitive yield-farming strategies for borrowers. The APY is paid out in the same asset deposited, enhancing user experience and trust.
Ecosystem Status: As of the last update, the Wise Ecosystem is live and functioning as intended since December 2020. The MVP is live on the testnet and undergoing a third-party audit. The Wise DAO is complete with a full mint capacity of 1152 ETH, and no WISE tokens have been burnt yet, with total revenue below $100k/year.
Benefits for Other Projects: Projects with a token and a Uniswap liquidity pool can benefit from the Wise Ecosystem by pairing their token with WISE for liquidity instead of ETH. This allows their token to benefit from the value created by the Wise Ecosystem. Token-less projects can also join by maintaining a portion of their treasury in WISE, with grants and developer support available on a case-by-case basis.

##  Admin Rules

The system admin rules are centered around the master address, which is a timelock contract secured by a multisig. This setup ensures that only the master can add new price feed and address pairs to the system, providing a layer of security and control over the price feed data. The system also includes modifiers like onlyMaster to restrict certain functions to the master address.
in the system `PtOracleDerivative.sol` contract constructor function initializes several key components, including the Pendle market address, the Chainlink price feeds for USD and ETH/USD, the Pendle oracle for PtToken, the oracle name, and the TWAP duration. This suggests that the contract's initial setup and configuration are controlled by the deployer, who would have the necessary permissions to set these parameters.
and also in the `FeeManager.sol` contract's constructor function initializes several key components, including addresses for various services and contracts such as the master, Aave, wiseLending, oracleHub, wiseSecurity, and positionNFT. The contract also includes modifiers like onlyMaster and onlyIncentiveMaster to restrict access to certain functions, indicating that the contract's initial setup and configuration are controlled by the deployer and specific roles within the system.

## key contracts of the system 

| Number | contracts | work |
|--------|-----------|------|
| 1 | FeeManager | Purpose of this contract is to organize fee distribution from wiseLending. The feeManager aquires fee token in form of shares from each pool and can call them with "claimWiseFees()" for each pool. Furthermore, this contracts has two different incentive structures which can be used to bootstrap the WISE ecosystem (beneficial and incnetiveOwner roles). Additionally, this contract keeps track of the bad debt of each postion and has a simple mechanism to pay them back via incentives. The incentive amount is funded by the gathered fees. |
| 2 | PendleLpOracle | PriceFeed contract for Lp-Token with USD chainLink feed to get a feed measured in ETH. Takes chainLink oracle value and multiplies it with the corresponding TWAP and other chainLink oracles. |
| 3 | WiseOracleHub | WiseOracleHub is an onchain extension for price feeds (chainLink or others).The master address is owned by a timelock contract which itself is secured by amultisig. Only the master can add new price feed <-> address pairs to the contract.One advantage is the linking of price feeds to their underlying token address.Therefore, users can get the current ETH value of a token by just knowing the tokenaddress when calling {latestResolver}. It takes the answer from {latestRoundData}for chainLink oracles as recommended from chainLink. If you want to propose adding an own developed price feed it ismandatory to wrap its answer into a function mimicking {latestRoundData}(See {latestResolver} implementation).Additionally, the oracleHub provides so called heartbeat checks if a token getsstill updated in expected time interval.|
| 4 | ApprovalHelper |  Allows to execute safe approve for a token |
| 5 | WiseSecurity | WiseSecurity is a core contract for wiseLending including most ofthe performed security checks for withdraws, borrows, paybacks and liquidations.It also has several read only functions providing UI data for a better userexperiencne.  |
| 6 | AaveHub   | Purpose of this contract is to optimize capital efficency by using aave pools. Not borrowed funds are deposited into correspoding aave pools to earn supply APY. The aToken are holded by the wiseLending contract but the accounting is managed by the position NFTs. This is possible due to the included onBehlaf functionallity inside wiseLending. |
| 7 | WiseLending   | WISE lending is an automated lending platform on which users can collateralize their assets and borrow tokens against them. Users need to pay borrow rates for debt tokens, which are reflected in a borrow APY for each asset type (pool). This borrow rate is variable over time and determined through the utilization of the pool. The bounding curve is a family of different bonding curves adjusted automatically by LASA (Lending Automated Scaling Algorithm). For more information, see: [https://wisesoft.gitbook.io/wise/wise-lending-protocol/lasa-ai] In addition to normal deposit, withdraw, borrow, and payback functions, there are other interacting modes: Solely deposit and withdraw allows the user to keep their funds private, enabling  them to withdraw even when the pools are borrowed empty. Aave pools  allow for maximal capital efficiency by earning aave supply APY for not borrowed funds. Special curve pools nside beefy farms can be used as collateral, opening up new usage possibilities for these asset types Users can pay back their borrow with lending shares of the same asset type, making it easier to manage their positions.Users save their collaterals and borrows inside a position NFT, making it possible to trade their whole positions or use them in second-layer contracts (e.g., spot trading with PTP NFT trading |
| 8 | PtOracleDerivative |  Pricefeed contract for PtToken with USD chainLink feed to get a feed measured in ETH. Takes chainLink oracle value and multiplies it with the corresponding TWAP and other chainLink oracles |


## Approach Taken to the Code Base

The system follows a modular approach. but the  `ApprovalHelper.sol` and `TransferHelper.sol` contract use adopts a straightforward approach to executing a safe approval operation. It uses the _callOptionalReturn function from the CallOptionalReturn contract it inherits from, which is designed to handle calls to external contracts that may not return a value. This approach ensures that the contract can safely execute the approval operation without causing the transaction to revert if the token contract does not return a value.



| Codebase Quality | Comments | 
|------------------|----------|
| Code Maintainability and Reliability | The system is well-structured, with clear separation of concerns and use of modular components |
| Code Comments and Documentation |  The sytem includes detailed comments and documentation, particularly in the constructor and functions, which aids in understanding the purpose and functionality of the system. |
| Code Structure and Formatting | The system follows a clean and organized structure, with clear separation of state variables, constructor, and functions. It uses Solidity's immutable keyword for variables that do not change after contracts deployment, which is a good practice for gas optimization. |

## Centralization risk and systemic Risk

### Centralization Risk

the system have centralization risks through the use of a single master account (onlyMaster modifier) in the `FeeManager.sol` contract  and an incentive master (onlyIncentiveMaster modifier). These roles have significant control over the contract's operations, including setting fees, adjusting incentives, and managing bad debt. If these roles were to be compromised or misused, it could lead to centralization of control and potential exploitation of the system.
also in the `AaveHub.sol` contract introduces centralization risks through its reliance on the wiseLending contract for accounting and the Aave protocol for lending activities. If the wiseLending contract or the Aave protocol were to be compromised or behave unexpectedly, this could have significant implications for the contract's functionality and the broader DeFi ecosystem. Additionally, the contract's dependency on a specific set of external contracts could introduce risks if these contracts were to become outdated or ineffective.

### Systemic Risk

Systemic risks in the Wise Lending ecosystem are deeply intertwined with the reliance on external dependencies, such as price feeds from Chainlink, and the integration of external contracts and services like Aave, WiseLending, OracleHub, WiseSecurity, and PositionNFT. These dependencies are crucial for the functionality of smart contracts and the broader Wise Lending ecosystem. However, they also introduce vulnerabilities. If any of these external dependencies were to become compromised or if there were a significant error in their operation, it could have far-reaching implications for the contract's functionality and the stability of the Wise Lending ecosystem.

Furthermore, the contract's reliance on specific incentive structures could introduce additional risks. If these structures were to become outdated or ineffective, it could lead to unintended consequences for the contract's operation and the overall Wise Lending ecosystem. This highlights the importance of maintaining high-quality, secure, and reliable external data sources to mitigate systemic risks in Wise Lending applications. Chainlink, for instance, provides a secure decentralized middleware for connecting on-chain and off-chain environments, offering high data quality, security, and reliability guarantees for smart contract execution. This has helped reduce systemic risks in the Wise Lending ecosystem and enabled users to trust smart contracts with significant value across various Wise Lending applications 3410.

Chainlink's approach to securing the Wise Lending ecosystem includes a multi-layered defense-in-depth strategy, robust blockchain-agnostic architecture, and backup oracle networks with client diversity. These measures ensure that Chainlink Price Feeds are securely updated and delivered, mitigating potential issues and providing a reliable source of data for Wise Lending applications

## Gas Optimizations

| Number | Issue | Instences |
|--------|-------|-----------|
|[G-01]| Using bytes32 is cheaper than using string | 28 |
|[G-02]| Store Data in calldata Instead of Memory for Certain Function Parameters  | 19 |
|[G-03]| Use assembly to emit events | 75 |
|[G-04]| Usethe external Visibility Modifier | 1 |
|[G-05]| Avoid Unnecessary Use of Storage | 7 |
|[G-06]| Precompute Off-Chain When Possible | 2 |
|[G-07]| State variables only set in the constructor should be declared immutable | 10 |
|[G-08]| Do not calculate constants  | 23 |
|[G-09]| Redundant state variable getters | 1 |
|[G-10]| Check if amount is greater than zero before performing safeTransfer | 10 |
|[G-11]| State variables should be cached in stack variables rather than re-reading them from storage | 2 |
|[G-12]| Calculation(addition, subtruction) which won’t underflow can be unchecked in | 13 |
|[G-13]| Using Storage Instead of Memory for structs/arrays Saves Gas | 13 |
|[G-14]| Use named returns for local variables of pure functions where it is possible | 6 |
|[G-15]| Using assembly to revert with an error message | 4 |
|[G-16]| Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4 | 6 |
|[G-17]| Using > 0 costs more gas than != 0 when used on a uint in a if() statement | 21 |
|[G-18]| Functions guaranteed to revert when called by normal users can be marked payable | 4 |
|[G-19]| require()/revert() strings longer than 32 bytes cost extra gas | 4 |


## Recommendations

Regular Audits: Consider having the system audited by a third-party security firm to identify any potential vulnerabilities or issues.
Diversify Roles: To mitigate the risk associated with centralization, consider implementing a more decentralized governance model for critical operations, such as setting fees and adjusting incentives.
Monitor and Update Dependencies: Keep the contract's dependencies up to date to benefit from the latest security patches and improvements.
Testing and Validation: Ensure thorough testing, including edge cases and potential failure modes, to identify and address any issues before deployment.
Documentation and Maintenance: Maintain comprehensive documentation and ensure that the contract is regularly reviewed and updated to address any new security concerns or changes in the DeFi ecosystem.

## Time spent

55 hours

### Time spent:
55 hours