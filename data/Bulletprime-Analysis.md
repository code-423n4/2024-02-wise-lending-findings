Brief Overview
The Wise Ecosystem is a decentralized financial platform that aims to create a sustainable financial ecosystem in the crypto space. It is built around the `$WISE` token, which is backed by a pool of ETH and appreciates based on the revenue earned from the platform's dApps. Unlike most other tokens, `$WISE` is not used to pay out APY rewards to dApp users, but instead is used to create deflation. The platform's flagship dApp is WiseLending, a no-strings-attached lending and borrowing protocol that allows users to earn passively by depositing crypto into lending pools and access competitive yield-farming strategies. The Wise DAO, which drives value to the community rather than VCs, is also a key component of the ecosystem. The platform is currently live and has been working as intended since December 2020. Projects can benefit from the Wise Ecosystem by pairing their token with `$WISE` for liquidity, which allows them to piggyback on the value created by the ecosystem. Grants and developer support are available for those who wish to join the ecosystem.

My analysis focuses on the additional risk that comes from the composability within the in-scope systems.
The goal of the analysis is to show my adversarial thought process and to suggest a robust security process to ship the codebase in a state that is maintainable and safer for people to use. Please note that untill explicitly stated otherwise, any architectural risks or implementation issues mentioned in this analysis are not to be considered vulnerabilities for altering the architecture or code solely based on this analysis. As an auditor, I acknowledge the vitality of a thorough evaluation for design decisions in a complex project, taking associated risks into consideration as one single part of an overarching process. 

Codebase Approach 
Understanding the Contract -  Thoroghly examining `audit documentation and scope`, understanding the purpose, functionality, and architecture of the contract. This includes reviewing the codebase, documentation, and any related materials.

Static Analysis - Performed  thorough static analysis of the contract's source code. This involves reviewing the code for potential vulnerabilities, such as input validation issues, access control problems, reentrancy vulnerabilities, overflow and underflow issues, gas limit attack susceptibilities, timestamp dependency risks, front running attack vulnerabilities, and denial of service (DoS) attack susceptibilities.

Dynamic Analysis - I also performed dynamic analysis of the protocol by executing the contract in a controlled environment, such as a local development node and a testing framework. This helped identify any runtime issues, such as unexpected behavior, crashes, or other anomalies.

Substrate Testing - Setup and Tests - Setting up to execute forge test was remarkably effortless, greatly enhancing the efficiency of the auditing process. With a fully functional test harness at our disposal, we not only accelerate the testing of intricate concepts and potential vulnerabilities but also gain insights into the developer’s expectations regarding the implementation. Moreover, we can deliver
more value to the project by incorporating our proofs of concept into its tests wherever feasible.

Manual Code Review - I then manually reviewed  the contract's source code, focusing on critical areas of the codebase, such as functions that handle user input, functions that manage access control, and functions that perform sensitive operations.

Reverse Engineering - In some cases, I may needed to perform reverse engineering on the contract's compiled binary code to gain a deeper understanding of the contract's inner workings and to identify any hidden vulnerabilities or backdoors.

Exploit Development - Moving on, as I identified any potential vulnerabilities in the contract, I would attempt to develop proof-of-concept (PoC) exploits to demonstrate the feasibility of exploiting the identified vulnerabilities bearing in mind, invariants and critial parts of functions that can lead to users lossing funds.

Reporting and Remediation - Finally, I prepared a detailed report outlining the identified vulnerabilities, the associated risks, and recommendations for remediation. I also provide guidance on how to fix the identified vulnerabilities and how to prevent similar issues from occurring in the future.

Architecture
The architecture of the WiseLending contract, incorporates several key components and patterns that suggests a comprehensive approach to building a DeFi lending platform that aims to leverage the benefits of decentralized finance while addressing its inherent challenges.
The `constructor` function initializes the contract with addresses for the master, Wise Lending, and Aave Hub. This setup is crucial for the contract's operation, as it defines the roles and addresses involved in the contract's logic.
The `getLiveDebtRatio` function calculates the debt ratio of a position in normal mode, which is a critical metric for understanding the financial health of a position. It uses a precision factor to ensure the ratio is represented accurately.
The `setLiquidationSettings` function allows the master to configure liquidation settings, including base rewards and maximum fees. This is important for managing the liquidation process and ensuring it aligns with the contract's risk management strategy.
The `prepareCurvePools` and curveSecurityCheck functions are related to interacting with Curve pools, a decentralized exchange (DEX) aggregator. These functions ensure that the contract can safely interact with Curve pools, including approving tokens for swapping and performing security checks.
The `checksLiquidation`, `checksWithdraw`, `checksSolelyWithdraw`, `checksBorrow`, `checksCollateralizeDeposit`, `checkUncollateralizedDeposit`, `checkHealthState`, `checkBadDebtLiquidation`, `overallLendingAPY`, `overallBorrowAPY`, `overallNetAPY`, and `safeLimitPosition` functions are various checks and calculations related to the financial operations of the contract, such as liquidation, withdrawal, borrowing, and deposit operations. These functions are crucial for ensuring the contract's logic operates correctly and securely.

Strengths
Modularity and Reusability - The contract is structured to be modular, with clear separation of concerns. For example, the use of modifiers like `healthStateCheck` and `syncPool` for specific operations enhances code readability and reusability.
Security Checks - The contract includes comprehensive security checks, such as `_checkHealthState`, `_compareSharePrices`, and `_getCurrentSharePriceMax`. These checks ensure that operations are only executed under safe conditions, reducing the risk of exploits.
Automated Scaling Algorithm (LASA) - The LASA algorithm is designed to dynamically adjust pool data based on token prices, which can help maintain liquidity and efficiency in the lending pool.
Event Emitting - The contract emits events for significant actions like deposits and withdrawals. This is crucial for transparency and tracking of contract activities.
Reentrancy Guard - The use of `_checkReentrancy` in `_syncPoolBeforeCodeExecution` suggests an attempt to mitigate reentrancy attacks, which is a common vulnerability in smart contracts.

Downsides
Complexity - The contract is quite complex, with multiple modifiers, internal functions, and external functions. This complexity can increase the risk of bugs and make the contract harder to audit and maintain.
Gas Costs - The use of modifiers and the LASA algorithm, which likely involves complex calculations, could lead to higher gas costs for transactions. This might discourage users from interacting with the contract.
Dependency on External Contracts: The contract relies heavily on external contracts (e.g., WISE_SECURITY, WISE_ORACLE, ICurve). This introduces a dependency risk. If any of these external contracts are compromised or behave unexpectedly, it could affect the entire system.
Lack of Detailed Documentation - While the contract includes comments explaining the purpose of functions and modifiers, more detailed documentation on the overall architecture, security considerations, and potential edge cases would be beneficial for developers and auditors.
Potential for Reentrancy Attacks - While the contract includes a reentrancy guard, it's crucial to ensure that all external calls are made after state changes. Failure to do so could still expose the contract to reentrancy attacks.

Leverages
Decentralization and Accessibility
Decentralization - The use of smart contracts on the Ethereum blockchain indicates a commitment to decentralization, removing intermediaries and enabling direct peer-to-peer transactions. This aligns with the core principles of DeFi, offering a transparent and permissionless financial system.
 
Accessibility - The platform's design suggests an effort to make lending accessible to anyone with an internet connection, regardless of geography or credit history. This is a significant advantage over traditional lending systems, which often impose barriers to entry.

Security and Risk Management
Security - The architecture includes security measures such as reentrancy guards and comprehensive security checks before executing transactions. This reflects an understanding of the security challenges in DeFi and a proactive approach to mitigating risks.

Risk Management - The platform's design suggests an emphasis on risk management, with features like liquidation settings and checks for healthy positions. This is crucial for maintaining the stability and integrity of the platform.

User Experience and Interface
User Experience - The focus on creating a simple and easy-to-use interface indicates a commitment to user experience. This is essential for attracting and retaining users in the competitive DeFi space.
Interactive Components - The mention of interactive components and displays for user flows suggests an effort to make the platform engaging and user-friendly, which can be critical for adoption and retention.

Community and Support
Community Involvement - The architecture suggests a strategy for involving the community, including platforms for user support and fostering an active community. This can help in the continuous development and improvement of the platform.
Documentation and Tutorials - The emphasis on providing comprehensive documentation, tutorials, and FAQs indicates a commitment to supporting users and ensuring they can effectively use the platform.

Legal and Regulatory Compliance
Compliance: The architecture suggests an awareness of the legal and regulatory landscape, including the need for licenses and adherence to anti-money laundering (AML) and know-your-customer (KYC) rules. This is crucial for operating within the legal framework and ensuring the platform's sustainability.

Qualitative Analysis
Code quality and documentation is very mature, Assessed through several key dimensions -
Security Best Practices
Principle of Least Privilege - The contract uses modifiers like onlyWiseLending and onlyMaster to restrict access to sensitive functions, adhering to the principle of least privilege. This is crucial for preventing unauthorized access and potential exploits.
Secure Data Handling - The contract employs safe arithmetic operations and data types, which helps mitigate common vulnerabilities such as integer overflow/underflow. This is evident in the use of unchecked blocks for arithmetic operations where overflow/underflow is expected and acceptable.
Gas Limit Considerations - While not explicitly mentioned, optimizing for gas efficiency is a critical aspect of smart contract development. The contract's architecture suggests an awareness of gas costs, especially in functions like `syncPool` and `healthStateCheck`, which are likely to be called frequently.
Continuous Monitoring and Maintenance - The contract's design suggests a focus on security and monitoring, although specific mechanisms for continuous monitoring and maintenance are not detailed. Implementing such mechanisms is essential for identifying and addressing security vulnerabilities post-deployment.

Testing Strategies
Unit Testing - The contract's architecture suggests the implementation of unit tests to verify the correctness of individual functions and contracts. This is crucial for ensuring that each component of the contract behaves as expected under various conditions.
Integration Testing: The contract likely involves integration testing to ensure that different components and contracts interact correctly. This is important for maintaining the overall functionality and security of the system .
Fuzz Testing -Incorporating fuzz testing could help uncover unexpected behavior or vulnerabilities by generating random inputs and interactions with the smart contract.
Property-Based Testing - The contract's design suggests the use of property-based tests to verify desired properties or invariants, ensuring the correctness and security of the contract's logic.

Auditing Solidity Code
Reviewing Code Logic - The contract's architecture focuses on logical correctness and security. This includes the use of modifiers for access control and the implementation of functions that perform critical operations like liquidation and deposit/withdrawal.
Identifying Security Patterns - The contract demonstrates the use of recognized security patterns, such as access control through modifiers and the use of revert statements for error handling. This suggests an awareness of common security patterns and anti-patterns.
Analyzing External Dependencies - The contract's reliance on external contracts (e.g., WISE_SECURITY, WISE_ORACLE) suggests the importance of evaluating these dependencies for potential security risks. This is a critical aspect of smart contract development, ensuring that external interactions do not introduce vulnerabilities.
Conducting Automated Analysis - The use of automated analysis tools and scanners is recommended to identify common vulnerabilities. This includes checking for reentrancy, integer overflow/underflow, and uninitialized variables.

The WiseLending code generally demonstrates a proactive approach to securing smart contracts, adhering to security best practices, and employing effective testing strategies. However, a comprehensive audit and continuous monitoring are essential for identifying and mitigating potential vulnerabilities post-deployment. 

Centralization Risk
Areas of Potential Failure
Smart Contract Vulnerabilities - The exploit in Wise Lending highlighted a vulnerability due to precision loss in smart contract operations. This indicates a critical area where the contract's logic could be exploited if not properly safeguarded.
External Dependencies - The contract's reliance on external contracts (e.g., WISE_SECURITY, WISE_ORACLE) introduces a risk of centralization. If these external contracts are compromised or behave unexpectedly, it could affect the entire system.
Regulatory Compliance - The lack of detailed information on how the contract addresses regulatory compliance, such as KYC/AML, could pose risks. Non-compliance with regulations could lead to legal and financial consequences.
Liquidity and Market Balance - The exploit in Wise Lending involved markets with zero balance, indicating a potential vulnerability in handling markets with insufficient liquidity.

Threat of Centralization Risks
Centralization Through External Dependencies - The contract's reliance on external contracts and services can introduce centralization risks. If these dependencies are centralized, they could become single points of failure or be targeted by attackers.
Regulatory Compliance - The need to comply with regulations, especially those related to KYC/AML, could lead to centralization if compliance is managed by a centralized authority or if the contract relies on centralized services for compliance checks.
Suggestions to Mitigate
Robust Mathematical Functions - Implement robust mathematical functions within smart contracts to handle high-precision calculations safely. Use libraries or frameworks designed for safe arithmetic operations in smart contract environments.
Checks and Balances for Markets - Implement checks and balances for markets with zero balance, especially when they are newly deployed. This could involve temporarily disabling certain functionalities until a minimum liquidity threshold is met.
Formal Verification - Apply formal verification techniques to mathematically prove that the code behaves as intended under all possible conditions. This can help identify and rectify potential vulnerabilities and logic errors.
Decentralized Solutions for Compliance - Develop native decentralized solutions for compliance needs, such as KYC/AML, to avoid relying on centralized third parties. This can help mitigate the risk of centralization and ensure compliance without sacrificing privacy and flexibility.
Continuous Monitoring and Security Audits - Implement continuous monitoring tools to flag any unusual activity and conduct regular security audits to identify and address vulnerabilities. This can help in promptly mitigating risks and ensuring the security of the contract.
Education and Transparency - Educate users about the risks and ensure transparency in the contract's operations. This can help users make informed decisions and reduce the risk of exploitation.
By addressing these areas and implementing the suggested mitigations, the WiseLending contract can reduce the risk of points of failure and centralization risks, enhancing its security and reliability in the DeFi ecosystem.

 Mechanism Review
My Mechanism review for the WiseLending contract, focuses on governance, transparency, dispute resolution, and future considerations.

Governance Mechanism
On-Chain Governance - The WiseLending contract, being a decentralized lending and borrowing protocol, likely employs on-chain governance where rules and decision-making processes are embedded in the smart contract code itself. This ensures that the rules are enforced by the blockchain network, and any changes to the rules need to be approved by the network participants.
Off-Chain Governance - While not explicitly mentioned, off-chain governance could also be a part of the WiseLending protocol's governance model, involving stakeholders outside the blockchain network for decision-making processes. This model allows for more complex decision-making processes but can be more prone to centralization.

Transparency and Auditability
Transparency and Auditability - The contract should ensure transparency and auditability, making all transactions visible to all parties and having mechanisms in place to detect and prevent fraud. This is crucial for building trust among users and stakeholders.

Dispute Resolution Mechanism
Dispute Resolution - The WiseLending contract should include mechanisms for resolving disputes that may arise during the contract's execution. This could involve the use of arbitration or other dispute resolution mechanisms that are agreed upon by all parties involved in the contract.
Upgradability
Upgradability - The contract should be designed to allow for upgrades and changes to the system, ensuring that the contract remains relevant and can adapt to changing market conditions. However, any changes to the contract should be agreed upon by all parties, and there should be mechanisms in place to prevent unauthorized changes.

Future Considerations
Standards and DAOs - As the use of smart contracts continues to grow, there will be an increased focus on developing industry-wide standards for smart contract governance. The emergence of Decentralized Autonomous Organizations (DAOs) could revolutionize traditional governance models by removing the need for central authorities and intermediaries.
Oracles - The role of oracles in providing data to smart contracts and triggering events based on external data sources will become increasingly important. However, there is a risk that oracles could be compromised or manipulated, which could undermine the integrity of the entire smart contract system.

Systemic Risks
Smart Contract Vulnerabilities - Vulnerabilities in smart contracts can lead to exploits that could disrupt the entire DeFi ecosystem. This includes vulnerabilities in the WiseLending contract itself or in any external contracts it interacts with.
Over-Collateralization - Over-collateralization, where users lock up more assets than necessary to secure a loan, can lead to liquidity issues if the value of the collateral falls significantly. This risk is particularly relevant in the context of lending and borrowing protocols like WiseLending 5.
Concentration of Assets - The concentration of assets within a few DeFi protocols increases systemic risk. If a large portion of the DeFi ecosystem's assets are concentrated in a few protocols, a failure in one of these protocols could have a significant impact on the entire ecosystem.

Mitigation
Smart Contract Audits and Security - Conduct thorough audits of smart contracts by reputable security firms to identify vulnerabilities and bugs in the code. Continuous monitoring of smart contracts is also crucial to promptly address any identified vulnerabilities.
Diversification and Asset Collateralization - Encourage diversification of assets among users and avoid over-collateralization in protocols. Protocols that accept multiple types of collateral and implement dynamic collateralization ratios based on market conditions can enhance stability and resilience.
Stress Testing and Scenario Simulations - Perform stress testing and simulated extreme scenarios to identify weaknesses in DeFi protocols. This helps improve risk management and assess the overall health of the ecosystem.
Decentralization and User Education - Promote decentralization and educate users about DeFi risks. This is essential for reducing centralization risks and enhancing the overall stability of the DeFi ecosystem.
Insurance and Stability Mechanisms - Implement insurance mechanisms and stability protocols to protect users and the ecosystem from large-scale losses. This includes mechanisms for liquidation, short-selling, and other forms of risk management.
By addressing these systemic risks through the suggested mitigation strategies, the WiseLending contract and the broader DeFi ecosystem can fortify themselves against potential crises, establishing a solid foundation for sustained growth and resilience in the face of unforeseen challenges.

I spent 6 days on this audit, each day I put in around 5-8 hours of work. In total would be around 40 hours.

3 days were to analyze and fully understand the codebase.
2 days were dedicated only to bug hunting. I didn’t follow the recognition pattern; I was mostly focused on identifying critical issues caused by incorrect assumptions and bugs in the code.
The last day was dedicated to filling out the reports and writing this analysis.


### Time spent:
40 hours