## Conceptual Overview

By looking into the wiseLending protocol, it's clear we're looking at an intricate web of functionalities designed to optimize and secure the lending and borrowing landscape within DeFi. Starting off, users engage with the protocol by depositing their digital assets. This isn't just a deposit; it's the entryway into a realm where those assets can work harder for the user, be it through earning interest, serving as collateral for loans, or embarking on leveraged yield farming ventures. This initial step is vital—it's where the journey begins.

A standout aspect of this protocol is its smart approach to managing these deposits and the financial activities they empower. Through tokenization, encapsulated within contracts like `PowerFarmNFTs`, the protocol doesn't just handle assets; it transforms positions into NFTs. This move adds a layer of flexibility and liquidity unseen in traditional finance—positions can now be easily managed, transferred, or even used in other DeFi applications.

As users navigate this system, they may opt to borrow, leveraging their deposits to possibly amplify their investment outcomes. Here's where the protocol's brain, its interest rate models and liquidation mechanisms, comes into play. Borrowing isn't a static affair; it's governed by dynamic rates that adjust based on the market's heartbeat, ensuring fairness and efficiency.

But it's not all about making gains without guardrails. The protocol's liquidation strategies and collateral management are like the safety nets at a trapeze show—they're there to catch you if market volatility tries to throw you off balance. The integration of flash loans adds another layer, offering users sophisticated strategies for leverage or liquidation that were once out of reach.

All these features, from depositing to borrowing, from tokenization to strategic liquidations, are underpinned by robust contracts such as `WiseSecurity` and `FeeManager`. These aren't just background processes; they're the guardians of the protocol's integrity and the fair distribution of fees, ensuring that the system remains secure and equitable for all users.



[![Conceptual.png](https://i.postimg.cc/Gpj7rnzj/Conceptual.png)](https://postimg.cc/v1cLLjJ1)

## Technical Overview

In auditing the wiseLending protocol, the Technical Overview emphasizes a deep dive into its codebase, illustrating how each component interacts within the ecosystem to facilitate lending, borrowing, and leveraged yield farming. At the core, the protocol hinges on smart contracts meticulously crafted to manage the lifecycle of deposits, loans, and leveraged positions with an intricate linkage to external DeFi platforms for enhanced functionality such as Aave for flash loans.

Upon a user initiating a deposit, the interaction begins with the `wiseLending.sol` contract, where assets are tokenized and pooled. This process intricately utilizes ERC-20 tokens, interfacing with the `PowerFarmNFTs.sol` for the tokenization of positions into NFTs, enabling a unique representation of each user's stake. This NFT mechanism, managed through `PowerFarmNFTs.sol`, is crucial for delineating ownership and state of individual lending and borrowing positions within the protocol.

For borrowing, the `WiseSecurity.sol` contract takes center stage, implementing sophisticated logic to dynamically adjust interest rates based on prevailing market conditions and protocol-specific risk parameters. This is further bolstered by a comprehensive liquidation mechanism detailed within `FeeManager.sol`, designed to protect the protocol from undercollateralization. This system is sensitive to real-time price feeds, fetched from `WiseOracleHub.sol`, ensuring that loans remain over-collateralized and positions are liquidated promptly should the collateral value fall below a predetermined threshold.

The leveraged yield farming feature, a standout functionality, is meticulously managed through the `PendlePowerFarmToken.sol` and its auxiliary contracts, allowing users to leverage their positions to maximize yield. This component intricately interfaces with Aave's flash loans, as per the logic in `WiseSecurityHelper.sol`, to provide the capital necessary for establishing leveraged positions, all while managing the risks associated with such strategies.

Each interaction within the ecosystem is engineered for gas efficiency and security, with an emphasis on minimizing the attack surfaces through reentrancy guards and ensuring the integrity of financial transactions via secure smart contract practices. The integration with external DeFi protocols like Aave for flash loans is handled with a high degree of caution, with contracts such as `WiseSecurity.sol` ensuring that these interactions do not expose the protocol to undue risk.

[![Technical.png](https://i.postimg.cc/rp865JPQ/Technical.png)](https://postimg.cc/rDYYkScr)


## Core Features of this Project

## Yield Farming

Diving deeper into the Leveraged Yield Farming feature within the wiseLending protocol, let's dissect the execution flow, especially focusing on the calculations and the underlying logic that governs the leveraging mechanism.

At the heart of this feature lies the `PendlePowerFarm` contract, which orchestrates the leveraged yield farming operations. The execution begins with the initiation of a leveraged farming position through the `enterFarm` function, which takes parameters such as the leverage ratio (`_leverage`), the amount of collateral deposited (`_initialAmount`), and the desired yield farming strategy.

### Leveraged Position Calculation

The leverage mechanism is intricately designed to calculate the optimal amount of funds to be borrowed against the user's collateral to achieve the desired leverage ratio. This calculation takes place in the `_openPosition` function and leverages the following formula:

```
leveragedAmount = _initialAmount * _leverage
```

The `leveragedAmount` represents the total investment in the yield farming position, combining the user's own capital and borrowed funds. The `_leverage` parameter, expressed as a multiplier (e.g., a leverage of 2x corresponds to `_leverage = 2 * 1e18` considering solidity's lack of floating numbers), dictates how much larger the total position is compared to the initial collateral.

### Flash Loans for Leverage

To facilitate the borrowing of additional funds, the protocol employs a flash loan mechanism through the `executeBalancerFlashLoan` function. This allows for the temporary acquisition of liquidity needed to reach the target leveraged amount without requiring additional capital input from the user. The key to this strategy is the instantaneous nature of flash loans, which must be repaid within the same transaction, thus mitigating counterparty risk.

### Repaying the Flash Loan

Upon receiving the flash loan, the borrowed funds, alongside the user's initial collateral, are deployed into the selected yield farming strategy. The interest generated from these farming activities contributes to covering the flash loan's interest and potentially yields a higher return due to the increased capital exposure. The repayment of the flash loan is automatically handled within the same transaction, ensuring the atomicity of the operation.

### Interest Rate and Liquidation Logic

The interest rate applied to the borrowed funds is a critical component, directly influencing the profitability of leveraged positions. It is dynamically calculated based on the current utilization rate of the lending pool (sourced from AAVE), adhering to predefined interest rate models that account for factors like liquidity and market volatility. The `getNewBorrowRate` function illustrates this dynamic calculation, adjusting the borrow rates in real-time to reflect changing market conditions.

Liquidation safeguards are integral to maintaining protocol stability and protecting against market downturns. Leveraged positions are continuously monitored, and if the collateral value dips below a certain threshold relative to the borrowed amount (a condition assessed through the `getLiveDebtRatio` function), the position becomes eligible for liquidation. This process entails selling off collateral to repay the borrowed funds, mitigated through the protocol's strategic interaction with decentralized exchange platforms for asset liquidation.

[![1.png](https://i.postimg.cc/SQf71bDS/1.png)](https://postimg.cc/G99y9NWV)


## Interest Rate Models

Interest Rate Models (IRMs) play a pivotal role within the wiseLending protocol, as they determine the borrowing and lending rates users encounter when interacting with the platform. The protocol utilizes sophisticated mathematical models to dynamically adjust these rates based on various factors such as market conditions, asset utilization, and risk parameters.

One of the fundamental IRMs employed is the utilization-based interest rate model, which calculates borrowing rates based on the utilization ratio of assets within the lending pools. The formula to determine the utilization ratio (UR) is as follows:

$$UR = \frac{Total Borrowed Funds}{Total Available Funds}$$

This equation quantifies the extent to which assets in the lending pools are utilized, providing insights into the platform's overall liquidity and demand for borrowing. The borrowing rate (BR) is then derived from the utilization ratio using a mathematical function that adjusts rates according to market dynamics and risk considerations. For instance, a common formula for calculating borrowing rates based on utilization ratio might resemble:

$BR = \text{Base Rate + Spread } { *  UR}$

Here, the base rate represents the minimum borrowing rate set by the protocol, while the spread parameter determines the sensitivity of borrowing rates to changes in the utilization ratio. Higher spreads indicate a more significant impact of utilization on borrowing costs, reflecting a more aggressive pricing strategy to incentivize or disincentivize borrowing activity.

Additionally, the wiseLending protocol may incorporate risk-based adjustments to borrowing rates, accounting for factors such as asset volatility, counterparty credit risk, and market conditions. These adjustments can be implemented through dynamic risk premiums or tiered rate structures, where borrowers with higher risk profiles face elevated borrowing costs to compensate for increased default probabilities.

From a technical perspective, the implementation of IRMs within the wiseLending protocol involves extensive calculations and parameter tuning to ensure optimal rate dynamics and risk management. Smart contracts responsible for interest rate determination leverage on-chain data feeds, oracle integrations, and mathematical functions to compute rates dynamically in real-time, adapting to changing market conditions and user behaviors.

Furthermore, the protocol's architecture facilitates seamless integration of multiple IRMs, allowing for experimentation and optimization to achieve desired outcomes such as liquidity provisioning, risk mitigation, and user incentives. This modular approach enhances the protocol's flexibility and resilience, enabling it to adapt to evolving market trends and user preferences while maintaining robust risk management practices.

[![interest.png](https://i.postimg.cc/2yNxgzBG/interest.png)](https://postimg.cc/sQKShCmG)

## Mathematical Rigor

The Mathematical Rigor within the Wise Lending protocol is foundational to its operations, ensuring precise and equitable financial transactions. At its core, the protocol employs complex mathematical models and algorithms to determine crucial parameters such as interest rates, loan-to-value ratios, and risk premiums. These calculations are not only essential for maintaining the stability and efficiency of the lending platform but also for providing users with transparent and reliable financial services.

One fundamental aspect of the protocol's mathematical framework is the calculation of the Borrowing Rate (BR) for borrowers. The Borrowing Rate is determined based on several factors, including the Utilization Ratio (UR), Interest Base Score (IBS), Asset Risk Premium (ARP), and Relative Coefficient (RC). The UR reflects the proportion of the total borrowing capacity currently utilized by borrowers, while the IBS assesses the creditworthiness of borrowers based on factors such as credit history and collateralization. The ARP accounts for the risk associated with the underlying assets used as collateral, and the RC adjusts the base interest rate to reflect market conditions and risk factors.

The calculation of the Borrowing Rate involves a series of intricate steps, beginning with the determination of the Base Calculation (BC). The BC is calculated using the IBS, ARP, and RC, providing a baseline interest rate that serves as the foundation for determining the final Borrowing Rate. Additionally, the Borrowing Rate may be further adjusted based on factors such as the Compounding Benchmark (CB), Spread Coefficient (SC), and Upper Bound (UB), ensuring that the Borrowing Rate remains competitive and within acceptable limits.

To illustrate, let's consider an example calculation of the Borrowing Rate:

1. Calculate the Base Calculation (BC):
   - IBS = 750 (based on borrower's creditworthiness)
   - ARP = 1.5% (based on risk associated with collateral)
   - RC = 0.8 (adjustment factor based on market conditions)
   - BC = IBS + ARP + RC = 750 + 1.5 + 0.8 = 751.3

2. Adjust the BC based on the Compounding Benchmark (CB) and Spread Coefficient (SC):
   - CB = 2% (predefined benchmark rate)
   - SC = 0.5 (adjustment factor based on market conditions)
   - Adjusted BC = BC + (CB * SC) = 751.3 + (2 * 0.5) = 752.3

3. Determine the final Borrowing Rate (BR) within the Upper Bound (UB):
   - UB = 10% (maximum allowable Borrowing Rate)
   - BR = min(Adjusted BC, UB) = min(752.3, 10) = 752.3

In this example, the Borrowing Rate is calculated as 7.523%, ensuring that it remains competitive and within the upper limit set by the protocol. This meticulous calculation process exemplifies the Mathematical Rigor embedded within the Wise Lending protocol, providing users with accurate and fair interest rates tailored to their individual circumstances.

[![math.png](https://i.postimg.cc/T1Pnzp3r/math.png)](https://postimg.cc/ZB193Y3R)

## Financial Mechanism

Advanced financial mechanisms in Wise Lending encompass sophisticated algorithms and strategies designed to optimize lending, borrowing, and yield farming activities while managing risk and maximizing returns for users. These mechanisms leverage complex mathematical models and formulas to determine interest rates, collateral requirements, leverage ratios, and other key parameters.

One such mechanism is the calculation of interest rates for lending and borrowing activities. In Wise Lending, interest rates are dynamically adjusted based on various factors such as market conditions, asset volatility, supply and demand dynamics, and user preferences. The calculation of interest rates involves intricate formulas that take into account variables like base rates, liquidity premiums, risk factors, and market spreads.

For example, the formula for calculating the interest rate on a loan might incorporate the following variables:

- Base Rate: The baseline interest rate determined by market conditions and protocol settings.
- Liquidity Premium: Additional interest charged to borrowers to compensate lenders for providing liquidity to the market.
- Risk Premium: Extra interest charged to account for the credit risk associated with borrowers.
- Market Spread: The difference between the borrowing and lending rates, reflecting the profit margin for the protocol.

These variables are combined using mathematical equations to compute the final interest rate, which is then applied to loans and deposits within the platform. The calculation of interest rates is dynamic and may be adjusted periodically to reflect changing market conditions and risk factors.

In addition to interest rate calculations, advanced financial mechanisms in Wise Lending also include risk management strategies such as collateralization, leverage management, and liquidation mechanisms. These mechanisms aim to mitigate the risk of default, protect user funds, and maintain the stability of the platform.

For instance, the protocol may employ algorithms to determine optimal collateral ratios based on asset volatility, historical performance, and market conditions. Users seeking to borrow funds must provide sufficient collateral to cover their loans, with the collateral ratio adjusted dynamically to reflect changes in asset values and risk levels.


## Innovative Use of NFTs

In Wise Lending, NFTs (Non-Fungible Tokens) are used innovatively to represent users' positions and participation in the lending and borrowing ecosystem. These NFTs serve as unique digital assets that grant holders specific rights and privileges within the protocol.

One innovative use of NFTs in Wise Lending is to tokenize users' positions in the lending and borrowing pools. When users deposit assets into the protocol or borrow funds, they receive corresponding NFTs representing their positions. These NFTs contain metadata that encode important information such as the deposited assets, loan amounts, interest rates, and expiration dates.

By tokenizing positions as NFTs, Wise Lending introduces several benefits and features:

1. Ownership and Control: NFT holders have full ownership and control over their positions, allowing them to transfer, trade, or redeem their NFTs as desired. This provides users with flexibility and liquidity, enabling them to manage their assets more efficiently.

2. Programmable Rights: NFTs in Wise Lending can be programmed with specific rights and functionalities, allowing for dynamic and customizable features. For example, NFTs may grant holders voting rights in governance decisions, access to exclusive features or rewards, or participation in protocol upgrades and improvements.

3. Transparency and Accountability: By representing positions as NFTs on the blockchain, Wise Lending ensures transparency and accountability in the lending and borrowing process. Users can easily verify the status and details of their positions by inspecting the metadata encoded in their NFTs, promoting trust and confidence in the protocol.

4. Integration with DeFi Ecosystem: NFTs in Wise Lending can be seamlessly integrated with other DeFi protocols and applications, enabling interoperability and synergy across the decentralized finance ecosystem. For example, users may use their NFTs as collateral in other lending protocols, participate in decentralized exchanges, or engage in yield farming strategies.



## Potential Risk

One potential serious risk in the Wise Lending project is the reliance on centralized price oracles and external protocols such as Aave. Dependency on centralized components introduces the risk of censorship, manipulation, or downtime. If these centralized components were to experience issues such as data manipulation or service outages, it could adversely affect the accuracy of pricing information or the availability of lending and borrowing services on Wise Lending. This risk highlights the importance of diversifying data sources and exploring decentralized alternatives to mitigate reliance on centralized entities. Additionally, rigorous monitoring and contingency plans should be in place to respond promptly to any disruptions in external services.

### Time spent:
32 hours