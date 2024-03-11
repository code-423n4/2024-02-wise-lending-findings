# 🛠️ Wise Lending
**Decentralized liquidity market that allows users to supply crypto assets and start earning a variable APY from borrowers.**

## Overview of the protocol
The wiseLending protocol presents itself as a sophisticated construct within the Ethereum blockchain ecosystem, designed to intricately blend traditional DeFi lending models with an advanced leveraged farming mechanism. This melding of functionalities aims to redefine the user experience in the DeFi space by offering a more granular control over asset management and yield optimization.

Initiating the user journey with the fundamental act of depositing assets into the protocol, wiseLending leverages these deposits either as collateral for loans or as stakes in yield farming activities. This is where the project distinguishes itself, particularly through the integration with Aave for lending services and the novel implementation of the PowerFarm feature for enhanced yield farming operations. The PowerFarm, specifically, allows users to employ leveraged positions in farming activities, thereby amplifying the potential returns on their staked assets.

Central to orchestrating this complex interaction of depositing, borrowing, and farming is the suite of smart contracts developed for the protocol. Notably, contracts such as AaveHelper and PendlePowerFarm play crucial roles. AaveHelper facilitates seamless interactions with the Aave lending platform, enabling users to borrow against their deposits or earn interest. On the other hand, the PendlePowerFarm contract manages the leveraged yield farming activities, optimizing strategies to maximize user returns from staked assets.

Moreover, the protocol's design encompasses mechanisms for tokenizing yield through its partnership with Pendle, adding another layer of strategic asset management. This feature allows users to capitalize on future yield prospects, further enhancing the utility and flexibility of their investments within the platform.

The intricate web of functionalities, from asset deposits to strategy optimization for leveraged farming, is underpinned by robust governance structures ensured through contracts like OwnableMaster. This governance framework secures the protocol's integrity, operational

<br/>

[![Screenshot-from-2024-03-11-19-04-35.png](https://i.postimg.cc/9FSnCMKk/Screenshot-from-2024-03-11-19-04-35.png)](https://postimg.cc/GHFQjccP)

## The approach I would follow when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://code4rena.com/audits/2024-02-wise-lending#top

My approach to ensure a thorough and comprehensive audit would encompass several steps, combining theoretical understanding, practical testing, and security assessments. Here’s how I would proceed:

### 1. Documentation Review
First, I started by reading the project documentation in detail. Understanding the project's goals, architecture, and functionalities as outlined in the docs helps me grasp the intended use cases and the design rationale behind the smart contracts. This initial step is crucial for aligning my understanding with the project's objectives and preparing for a more technical dive.

### 2. Codebase Familiarization
Next, I cloned the project repository and familiarize myself with the codebase structure. I take note of how the contracts are organized, the naming conventions, and how the interfaces are defined and implemented. This step helps me understand the project’s modularity and the relationships between different contracts.

### 3. Dependency and Configuration Analysis
Before diving into the contract logic, I reviewed the project’s dependencies, such as OpenZeppelin contracts, and configuration settings, including compiler versions. This is to ensure that the project uses secure and up-to-date libraries and follows best practices for deployment.

### 4. Manual Code Review
With a solid understanding of the project structure and goals, I would begin a line-by-line review of the smart contracts. My focus would be on identifying common smart contract vulnerabilities such as reentrancy, overflow/underflow, improper access control, and unchecked external calls. Additionally, I assessed the contracts for logic errors, inefficiencies, and deviations from the documented functionalities.

### 5. Testing and Coverage Analysis
After the manual review, I run the project’s test suite to ensure all tests pass and to identify any areas of the code not covered by tests. I would write additional tests if necessary to achieve comprehensive coverage, particularly focusing on edge cases and failure modes.

### 9. Interaction with External Systems
Since the project interacts with aave, I review the integration points and external calls to ensure they are handled securely, respecting the trust boundaries and mitigating risks associated with external dependencies.

## Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;



-  **File name:** This field indicates the number of Contracts involves


-  **Lines:** This field represents the total number of lines in the source file, including code lines, comments, and blank lines.


-  **nSLOC:** nSLOC stands for "normalized source lines of code," and it further refines nLines by excluding both blank lines and comments. It gives a more accurate measure of the code's complexity.

-  **Comment Lines:** This field specifies the number of lines in the source file that contain comments.

-  **nLines:** nLines typically stands for "normalized lines" and represents the total number of lines in the source file excluding blank lines. 





## Languages
| language | files | code | comment | blank | total |
| :--- | ---: | ---: | ---: | ---: | ---: |
| Solidity | 44 | 6326 | 644 | 2,167 | 9137 |

#### Comment-to-Source Ratio: On average there are `4.25 code` lines per comment (lower=better).





## Codebase Quality

Overall, I consider the quality of the wise lending protocol codebase to be Good. The code appears to be mature and well-developed. I have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Documentation**                        | The codebase is accompanied by comprehensive documentation that clearly explains the purpose and functionality of each contract and function. Comments within the code are helpful for understanding the logic and flow, although there are areas where additional details could further clarify complex processes.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| **Consistency**                          | The naming conventions, file structure, and coding style are consistent throughout the project, facilitating readability and maintainability. This consistency extends to the use of modifiers, error handling, and event logging, which are uniformly applied across the contracts.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **Security Practices**                   | Security considerations are evident in the implementation of reentrancy guards, access controls, and checks-effects-interactions patterns. The use of well-established standards and libraries, such as OpenZeppelin for token standards and security utilities, adds to the robustness. However, the complexity of interactions between contracts, especially in the lending and borrowing mechanisms, necessitates rigorous security audits to ensure all edge cases and potential attack vectors are covered.                                                                                                                                                                                                                                                                           |
| **Testing and Coverage**                 | The protocol includes a suite of tests covering critical functionalities such as lending, borrowing, token transfers, and interaction with external protocols like AAVE. While the coverage is comprehensive for major flows, the addition of more tests for edge cases and stress conditions could further solidify confidence in the codebase's reliability.                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **Optimization and Gas Efficiency**      | The code demonstrates awareness of gas optimization, with practices like minimizing state changes, using efficient data storage types, and optimizing loops and calculations. However, given the protocol's complexity and interactions with external contracts, there are inherent challenges in achieving optimal gas efficiency. Continuous efforts to identify and implement optimizations would be beneficial, especially as the protocol scales.                                                                                                                                                                                                                                                                                                                                  |
| **Modularity and Upgradability**         | The protocol is designed with modularity in mind, allowing for easier maintenance and future expansion. Contracts are well-separated by functionality, which helps isolate changes and reduces the risk of unintended side effects. The approach to upgradability is cautious, with a clear governance process for making changes to critical components. This careful balance between modularity and upgradability is crucial for maintaining security while allowing the protocol to evolve.                                                                                                                                                                                                                                                                                 |
| **Dependency Management**                | Dependencies on external contracts and libraries are carefully managed, with explicit versioning for solidity and imported libraries. This reduces the risk of breaking changes and vulnerabilities introduced through dependencies. The protocol's reliance on external oracles and protocols like AAVE is a necessary aspect of its functionality, but it also introduces dependencies that need to be monitored for updates and potential security issues.                                                                                                                                                                                                                                                                                                                               |
| **Community and Ecosystem Integration**  | The protocol integrates well with the broader DeFi ecosystem, leveraging established protocols for lending, borrowing, and price feeds. This not only enriches the protocol's offerings but also encourages interoperability within the DeFi space. Active engagement with the community through documentation, open-source code, and channels for feedback contributes positively to the protocol's development and adoption.                                                                                                                                                                                                                                                                                                                             |


### Core Contracts and Functions:


In the wiseLending protocol, the liquidation and interest rate mechanisms are integral to maintaining the protocol's health and ensuring a stable equilibrium between lenders and borrowers. These mechanisms are meticulously designed, governed by a set of mathematical models to dynamically adjust interest rates and execute liquidations based on predefined conditions.

### Liquidation Mechanism

The liquidation process is initiated when a borrower's position becomes under-collateralized, defined by the Health Factor formula:

$$\text{Health Factor} = \frac{\text{Total Collateral Value} \times \text{Collateral Factor}}{\text{Total Borrow Value}}$$

This calculation is pivotal, ensuring that borrowed positions are sufficiently over-collateralized to cover potential losses. The `LiquidationManager.sol` contract encapsulates the logic for triggering liquidations. It assesses positions against the health factor threshold; positions falling below this threshold are marked for liquidation. The contract’s `executeLiquidation` function is then called, facilitating the transfer of collateral from the borrower to the liquidator, often at a discount to incentivize liquidation and quickly secure the protocol's assets.

### Interest Rate Model

The interest rate model is critical for balancing supply and demand within the lending pool. It adjusts borrowing costs and lending yields in response to the utilization rate of the pool, defined as the ratio of total borrows to total liquidity. The formula used to calculate interest rates based on utilization is:

\text{Interest Rate} = \text{Base Rate} + \left(\frac{\text{Utilization Rate}}{\text{Optimal Utilization Rate}}\right) \times \text{Slope Rate}

The `BorrowRateModel.sol` contract implements this model, specifically through the `getInterestRate` function, which calculates the current interest rate based on the pool's state. As utilization increases, borrowing costs rise to discourage further borrowing and encourage repayment, thereby replenishing liquidity. Conversely, lower utilization leads to lower borrowing costs, incentivizing borrowing activities.

This dynamic interest rate adjustment is essential for maintaining liquidity within the protocol and ensuring that lenders receive competitive yields on their deposits. The mathematical model behind this mechanism allows the protocol to respond to market conditions efficiently, ensuring stability and encouraging healthy lending and borrowing activities.




### Lending Mechanism

The lending process allows users to deposit assets into the protocol's liquidity pools, receiving pool tokens in return. These tokens represent the lender's share of the pool and entitle the holder to a portion of the interest payments collected from borrowers. The interest rate model adjusts the yield on deposited assets based on pool utilization, governed by the formula:

$$\text{Yield} = \text{f(Interest Rate, Pool Utilization)}$$

This formula ensures that lenders are compensated according to the current demand for borrowing. The `deposit` function in `WiseLending.sol` handles the asset transfer, minting of pool tokens, and recording of the deposit in the lender's account. Interest accrual is dynamically calculated based on the changing state of the pool, ensuring fair and transparent compensation for lenders.

### Borrowing Mechanism

Borrowing entails users locking collateral in exchange for liquidity from the pool. The amount one can borrow is determined by the collateral value and the pool's collateral factor, a safeguard to prevent under-collateralization. The borrowable amount and interest rates are recalculated using:

$$\text{Borrowable Amount} = \text{Collateral Value} \times \text{Collateral Factor}$$

$$\text{Interest Payment} = \text{Borrowed Amount} \times \text{Interest Rate}$$

The `borrow` function within `WiseLending.sol` manages these calculations, ensuring that borrowers receive liquidity according to their collateral's value while maintaining a buffer against market volatility. Interest rates are adjusted in real-time to reflect the pool's utilization rate, balancing supply and demand dynamics.



### Collateral Management and Tokenization of Positions

At the core of wiseLending's collateral management lies the intricate process of tokenizing user positions. This is achieved through the interaction of several smart contracts, notably the `PositionNFTs` contract, which mints non-fungible tokens (NFTs) representing the user's deposited collateral and borrowing positions. This NFT acts as a bearer token, encapsulating the rights and obligations of the underlying position, including the collateral value, borrow amount, and associated interest rates.

The tokenization of positions into NFTs introduces a novel layer of liquidity and transferability to otherwise static collateral. Users can transfer, sell, or use their NFTs in other decentralized finance (DeFi) protocols, unlocking new strategies for yield optimization and risk management. The `PositionNFTs` contract interacts closely with the main `WiseLending` contract, ensuring that all operations related to lending, borrowing, and collateral adjustment are accurately reflected in the tokenized positions.

### Aave Integration and Flash Loan Functionality

WiseLending's integration with Aave brings enhanced liquidity options and strategic financial tools to its users. The protocol leverages Aave's flash loans to facilitate certain operations like collateral swaps and debt restructuring without requiring upfront capital. This is meticulously handled through the `AaveHelper` contract, which serves as an intermediary to access Aave's flash loan service.

The flash loan process begins when a user or a smart contract requests liquidity for a specific operation, such as liquidating an undercollateralized position or rebalancing a portfolio. The `AaveHelper` contract initiates the flash loan, receives the requested assets, executes the predetermined operations (e.g., paying off debt, swapping collateral types), and returns the loan amount plus fees within a single transaction block. This sequence of actions requires precise coordination between the `WiseLending`, `AaveHelper`, and `PositionNFTs` contracts, ensuring that all changes are atomically executed and accurately reflected in the user's tokenized position.




## A Comprehensive Flowchart of the wiseLending Protocol Architecture
<br/>

[![Screenshot-from-2024-03-11-23-40-28.png](https://i.postimg.cc/cH8B8K7g/Screenshot-from-2024-03-11-23-40-28.png)](https://postimg.cc/R64HkZD4)


## Security Approach of the Project

### Successful current security understanding of the project;

1- The project has already underwent previous [audit](https://app.wiselending.com/omni-audit-v1.pdf), this innovative assessments on Code4rena is the second, where multiple auditors are scrutinizing the code.

### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

3- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

4- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

5- I also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

6 - As the project team, you can consider applying the multi-stage audit model.

[![sla.png](https://i.postimg.cc/nhR0kN3w/sla.png)](https://postimg.cc/Sn96Q1FW)

Read more about the MPA model;
https://mpa.solodit.xyz/

7 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19



## Systemic Risks, Centralization Risks, Technical Risks & Integration Risks

### Systemic Risks
Systemic risks in wiseLending mainly revolve around the complex interactions between multiple smart contracts, such as `WiseSecurityDeclarations`, `PendlePowerFarm`, and various `FeeManager` contracts. The interdependency on external protocols like AAVE for lending and borrowing functionalities adds to systemic complexities. Should any of these integrated protocols experience issues, it could potentially affect wiseLending’s operations. Additionally, the reliance on accurate price feeds from `IWiseOracleHub` for collateral valuation and liquidation thresholds introduces systemic exposure to oracle failure or manipulation.

### Centralization Risks
Centralization risks stem from the roles of `OwnableMaster` and administrative functions within contracts like `FeeManagerHelper` and `PendlePowerFarmControllerHelper`, which have the power to adjust critical parameters such as fee rates, incentive mechanisms, and contract integrations. While these functions are essential for protocol maintenance and upgrades, they also pose centralization risks if not decentralized or governed by a broad community or DAO structure.

### Technical Risks
The technical risks are primarily associated with the smart contract code's complexity and the potential for undiscovered vulnerabilities despite thorough audits. Contracts like `PendlePowerFarmMathLogic` and `WiseSecurityDeclarations` incorporate intricate mathematical models and financial algorithms to manage interest rates, leverage, and liquidations. Any inaccuracies in these algorithms could lead to unintended financial outcomes. Furthermore, the protocol's reliance on flash loans for leveraging positions introduces technical risk if the flash loan mechanisms are exploited.
Smart Contract Vulnerabilities: Bugs or logical errors in the smart contracts can lead to loss of funds, unauthorized access, or unintended behavior. Given the complexity of contracts like the Shrine, interest rate models, and oracle interactions, the attack surface is significant.

Smart Contract Vulnerabilities: Bugs or logical errors in the smart contracts can lead to loss of funds, unauthorized access, or unintended behavior. Given the complexity of contracts like the Shrine, interest rate models, and oracle interactions, the attack surface is significant.

Scalability Concerns: As transaction volumes grow, the platform must scale without compromising performance or security.




### Integration Risks
Integration risks are evident in the protocol’s heavy reliance on external DeFi platforms and standards, including AAVE for lending services and ERC-721 for NFT representations of positions. The `PowerFarmNFTs` contract, which tokenizes farming positions, and integration with AAVE through contracts like `PendlePowerFarmLeverageLogic` highlight these risks. Should there be any disruptions or changes in the integrated platforms' operations or APIs, wiseLending’s functionality could be directly impacted. Moreover, the protocol's functionality around flash loans and its integration with DEXs for liquidation processes are critical points of integration risk.



## Analysis of test cases

 **Foundry Testing:**
   
   Foundry, a modern smart contract testing framework, was utilized to test the unistaker contracts. This involved several key steps:
   
   a. **Installation and Setup:**
      - Foundry was installed using the command `curl -L https://foundry.paradigm.xyz | bash`, followed by `foundryup` to ensure the latest version was in use.
      - Dependencies were installed using `forge install`, ensuring all necessary components were available for the testing process.
      - Then to run the tests, I simply added the relevant files to the .env, referencing .env.example.
   
   b. **Execution of Tests:**
      - Tests were run using `forge install` followed by `forge build` and then `forge test`, executing a suite of predefined test cases that covered various functionalities and scenarios.
   
   c. **Test Coverage and Documentation:**
      - The overview of the testing suite, as referred to in the provided documentation, likely details the scope, scenarios, and objectives of each test, ensuring a comprehensive assessment of the contracts.
   

### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content. In particular, tests have been written successfully.

-   2) Overall line coverage percentage provided by your tests : 87

### What could they have done better?

-  1) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;

-  2) The coverage of test cases are good (i.e: 87%) but still near to 100 is required for best results to make sure that all the edge cases are tested
 
  **Test cases coverage wasn't mentioned on audit page so other resources were used**

[![test-cases.jpg](https://i.postimg.cc/1zgD5wCt/test-cases.jpg)](https://postimg.cc/v1s40gdF)

Ref:https://xin-xia.github.io/publication/icse194.pdf

[![nabeel-1.jpg](https://i.postimg.cc/6qtBdLQW/nabeel-1.jpg)](https://postimg.cc/bDVXPnbW)

<br/>
<br/>























## New insights and learning of project from this audit:


Throughout the audit of the wiseLending protocol, several insights and learnings emerged, shedding light on the intricacies of DeFi lending and borrowing platforms and the innovative approaches undertaken by this project. Here are some notable observations:

1. **Complexity of DeFi Integrations:** The project adeptly navigates the complexity of integrating with existing DeFi protocols such as Aave, showcasing a sophisticated understanding of external contracts and their interactions. This highlights the importance of deep domain knowledge in DeFi for creating interoperable and efficient systems.

2. **Innovative Use of NFTs:** The protocol's use of NFTs to represent positions is a novel approach, extending the utility of NFTs beyond the art and collectibles market into financial instruments. This could pave the way for more widespread tokenization of financial positions in DeFi.

3. **Advanced Financial Mechanisms:** The protocol employs complex financial mechanisms, including liquidation processes and interest rate models. This demonstrates an advanced level of financial engineering within the blockchain space, contributing to more sophisticated financial products available on-chain.

4. **Mathematical Rigor:** The project employs intricate mathematical formulas to calculate interest rates, liquidation thresholds, and rewards. This underscores the necessity of mathematical rigor and precision in the design of financial protocols, ensuring they operate predictably and fairly.

5. **Collateral Management:** The handling of collateral within the protocol, especially the tokenization of positions and integration with flash loans for liquidation, showcases the complexity of managing risk and liquidity in DeFi. It highlights the need for innovative solutions to these perennial financial challenges.


This audit not only provided a deep dive into the wiseLending protocol's workings but also offered broader insights into the state of DeFi development, security practices, and the potential for NFTs in financial contexts.


**NOTE: I do not track time when auditing or writing report, so the time I selected is just a number**

### Time spent:
5 hours