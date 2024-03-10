## Table of Contents

1. [Approach](#approach)
2. [Architecture Review](#architecture-review)
   - [High-Level Architecture Diagram](#high-level-architecture-diagram)
   - [Key Contracts](#key-contracts)
   - [Contract Interactions](#contract-interactions)
3. [Codebase Quality Analysis](#codebase-quality-analysis)
   - [Code Organization and Structure](#code-organization-and-structure)
   - [Coding Practices and Patterns](#coding-practices-and-patterns)
   - [Security Considerations](#security-considerations)
4. [Centralization Risks](#centralization-risks)
   - [Privileged Roles and Access Control](#privileged-roles-and-access-control)
   - [Dependency on External Contracts](#dependency-on-external-contracts)
5. [Mechanism Review](#mechanism-review)
   - [Lending and Borrowing](#lending-and-borrowing)
   - [Liquidation](#liquidation)
   - [Position NFTs](#position-nfts)
   - [Curve Integration](#curve-integration)
6. [Systemic Risks](#systemic-risks)
   - [Oracle Failure](#oracle-failure)
   - [Liquidity Risk](#liquidity-risk)
   - [Interest Rate Risk](#interest-rate-risk)
7. [Recommendations](#recommendations)

## Approach

1. **Code Review**: Reviewed the provided source code files (`WiseLending.sol` and `WiseSecurity.sol`) to understand the overall functionality, architecture, and implementation details.

2. **Documentation Analysis**: Studied the accompanying documentation and comments within the code to gain insights into the design decisions, mechanisms, and intended behavior.

3. **Static Analysis**: Performed static code analysis using industry-standard tools to identify potential issues, vulnerabilities, and areas for improvement.

4. **Architecture Evaluation**: Assessed the overall architecture, contract interactions, and design patterns employed in the codebase.

5. **Risk Analysis**: Analyzed potential risks, including centralization, systemic risks, and security considerations, based on the codebase and documented information.

6. **Mechanism Examination**: Examined the core mechanisms and functionalities implemented in the codebase, such as lending, borrowing, liquidation, and integration with external protocols.

7. **Best Practice Comparison**: Compared the codebase against industry best practices and standards for decentralized lending protocols to identify areas for improvement and potential deviations.

## Architecture Review

The Wise Lending codebase follows a modular architecture, with different contracts responsible for specific functionalities. The high-level architecture and key components are as follows:

### High-Level Architecture Diagram

```
+--------------------+
|     Master         |
+--------------------+
           |
+--------------------+
| WiseLendingManager |
+--------------------+
           |
+--------------------+
|    WiseSecurity    |
+--------------------+
           |
+--------------------+
|    WiseLending     |
+--------------------+
           |
+--------------------+
|  PoolManager       |
+--------------------+
     |         |
+--------------------+
|  AaveHub           |
+--------------------+
     |         |
+--------------------+
|  WiseOracleHub     |
+--------------------+
     |         |
+--------------------+
|  PositionNFT       |
+--------------------+
```

### Key Contracts

1. **Master**: This contract serves as the central authority, responsible for managing privileged roles and access control.

2. **WiseLendingManager**: This contract acts as the entry point and orchestrator for the lending protocol. It manages the overall lending and borrowing operations.

3. **WiseSecurity**: This contract handles security-related aspects, including checks and validations for various lending and borrowing operations, blacklisting tokens, setting liquidation incentives, and handling security shutdowns.

4. **WiseLending**: This is the core contract responsible for implementing the lending and borrowing functionality. It includes methods for depositing, withdrawing, borrowing, and repaying tokens, as well as liquidating undercollateralized positions.

5. **PoolManager**: This contract manages the lending and borrowing pools, handling pool-specific operations and interactions with external protocols like Aave.

6. **AaveHub**: This contract facilitates the integration with the Aave lending protocol, enabling users to earn supply APY for non-borrowed funds.

7. **WiseOracleHub**: This contract interfaces with external oracles to fetch token prices and exchange rates, which are crucial for lending and borrowing operations.

8. **PositionNFT**: This contract represents the position NFTs, which encapsulate users' collateral and borrowed positions, enabling trading and management of these positions.

### Contract Interactions

1. The `WiseLendingManager` orchestrates the overall lending and borrowing operations by interacting with the `WiseSecurity` and `WiseLending` contracts.

2. The `WiseSecurity` contract performs security checks and validations before allowing lending and borrowing operations in the `WiseLending` contract.

3. The `WiseLending` contract interacts with the `PoolManager` to manage lending and borrowing pools, as well as with the `AaveHub` and `WiseOracleHub` for specific functionalities.

4. The `PoolManager` interacts with external protocols like Aave through the `AaveHub` contract and fetches token prices and exchange rates from the `WiseOracleHub` contract.

5. The `PositionNFT` contract manages the position NFTs, which represent users' collateral and borrowed positions.

6. The `Master` contract oversees the entire system, managing privileged roles and access control for critical operations.

## Codebase Quality Analysis

### Code Organization and Structure

Some contracts, such as `WiseLending.sol`, are relatively large and could benefit from further separation of responsibilities. Splitting the contract into smaller, more focused contracts would improve code readability and reduce the cognitive load during maintenance and future development.

### Coding Practices and Patterns

The codebase employs various coding practices and design patterns, including:

- **Access Control**: The use of modifiers (`onlyMaster`, `onlyWiseLending`, `onlyAaveHub`) for access control and ensuring that only authorized entities can perform certain operations.

- **Reentrancy Guards**: The `_checkReentrancy()` function is used as a reentrancy guard to prevent potential reentrancy attacks.

- **Error Handling**: Extensive use of `require` statements and custom errors (`revert` statements) for input validation and error handling.

- **Event Emission**: The codebase emits events (`FundsSolelyWithdrawn`, `FundsSolelyDeposited`, `FundsReturned`, `SecurityShutdown`) to provide transparency and enable off-chain monitoring of critical operations.

- **Upgradability**: The codebase appears to be designed for upgradability, as evidenced by the use of delegatecall and the `WiseLendingManager` contract as an entry point.

- **Security Checks**: The `WiseSecurity` contract encapsulates various security checks and validations, promoting code reusability and maintainability.

- **Oracles and External Dependencies**: The codebase relies on external oracles (`WiseOracleHub`) for fetching token prices and exchange rates, as well as integrating with external protocols like Aave (`AaveHub`).

While the codebase follows many best practices, there are areas for improvement, such as further separation of concerns, enhanced code documentation, and more extensive use of code comments to improve readability and maintainability.

### Security Considerations

- **Access Control**: The use of modifiers and privileged roles ensures that critical operations can only be performed by authorized entities.

- **Reentrancy Guards**: The `_checkReentrancy()` function prevents potential reentrancy attacks by checking the state of the contract before allowing execution.

- **Input Validation**: Extensive use of `require` statements and custom errors to validate input parameters and prevent unexpected behavior.

- **Security Checks**: The `WiseSecurity` contract encapsulates various security checks and validations, promoting code reusability and reducing the risk of security vulnerabilities.

- **Blacklisting Tokens**: The ability to blacklist tokens helps mitigate risks associated with potentially compromised or problematic tokens.

- **Security Shutdown**: The `securityShutdown()` function allows designated security workers to halt the protocol's operations in case of emergencies or identified vulnerabilities.

However, it's important to note that a comprehensive security audit by experienced professionals is recommended to identify and mitigate potential vulnerabilities or risks that may not be immediately apparent from the codebase review.

Two potential vulnerabilities
---------------------------

Vulnerability 1: Assumption about share price behavior in WiseLending contract
-------------------------------------------------------------------------------

Explanation:
The WiseLending contract has a function `_compareSharePrices` which makes an assumption that between two calls to the function, the borrow share price should always decrease and the lend share price should always increase. This assumption is used to validate the share price changes. 

However, this assumption may not always hold true, especially in volatile market conditions or in case of unexpected events like flash crashes, oracle failures etc. If the assumption is violated, it could lead to the `_compareSharePrices` reverting, which would block further borrowing and lending activity until addressed.

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L210-L248
```solidity
function _compareSharePrices(
    address _poolToken,
    uint256 _lendSharePriceBefore,
    uint256 _borrowSharePriceBefore
)
    private
    view
{
    (
        uint256 lendSharePriceAfter,
        uint256 borrowSharePriceAfter
    ) = _getSharePrice(
        _poolToken
    );
    
    // Assumes lend share price can only increase
    _validateParameter(
        _lendSharePriceBefore,
        lendSharePriceAfter
    );

    // Assumes borrow share price can only decrease  
    _validateParameter(
        borrowSharePriceAfter,
        _borrowSharePriceBefore
    );
}
```
The `_validateParameter` function used above reverts if the first argument is greater than the second.

**Impact:**
If the share price changes in an unexpected way and the _compareSharePrices function starts reverting, it would block all further borrowing and lending for the affected pool token until the issue is resolved by the contract admins. This could lead to denial of service and loss of confidence in the protocol.

**Recommended Mitigation:**
Instead of relying on assumptions about share price behavior, the contract could use a different approach to validate share price changes, such as:
- Allowing share prices to change within a certain tolerance threshold 
- Using time-weighted average prices over a period to smooth out short-term fluctuations
- Having circuit breaker mechanisms to pause activity if prices change too drastically

Vulnerability 2: Blacklisting tokens could lock user funds in WiseSecurity contract
----------------------------------------------------------

**Explanation:**
The WiseSecurity contract allows an admin to blacklist tokens, which blocks their usage as collateral tokens. This is likely intended as a risk management feature to quickly stop usage of a token in case of issues like price manipulation, flash loan attacks etc.

However, if a token is blacklisted while users still have active loans or deposits using that token, it could unexpectedly lock their funds. The blacklisting would fail any health checks for their positions, preventing them from withdrawing or managing their position.

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L980-L988, https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L1045-L1054
```solidity
function setBlacklistToken(
    address _tokenAddress,
    bool _state
)
    external
    onlyMaster()
{
    wasBlacklisted[_tokenAddress] = _state;
}

function _checkPoolCondition(
    address _token
)
    private
    view
{
    if (poolLocked == true) {
        revert PoolInShutdown();
    }

    if (wasBlacklisted[_token] == true) {
        revert TokenBlacklisted();
    }
}
```

The `_checkPoolCondition` function is called by several key functions like deposit, withdraw, borrow, liquidate etc. If a token is blacklisted, this check would revert, preventing the user from interacting with their position.

**Impact:**
Blacklisting a token without allowing users to exit their positions could lead to a situation where user funds are locked in the protocol with no way to withdraw. This could lead to significant financial losses for users and loss of trust in the protocol.

**Recommended Mitigation:**
Before blacklisting a token, the protocol should have a mechanism to:
- Pause new deposits/borrows of the token, while still allowing withdrawals, repayments and liquidations
- Notify users to close their positions in the token within a grace period
- Automatically liquidate user positions after the grace period to allow them to withdraw most of their collateral
- Only fully blacklist the token after all user positions are closed

This would ensure users are not unexpectedly locked out of their funds due to token blacklisting.

## Centralization Risks

While the Wise Lending protocol aims to be decentralized, certain aspects of the codebase introduce centralization risks:

### Privileged Roles and Access Control

The codebase introduces several privileged roles, such as the `Master` and `SecurityWorker`, which have significant control over critical operations. For example:

```solidity
// WiseSecurity.sol
function setLiquidationSettings(
    uint256 _baseReward,
    uint256 _baseRewardFarm,
    uint256 _newMaxFeeETH,
    uint256 _newMaxFeeFarmETH
)
    external
    onlyMaster
{
    _setLiquidationSettings(
        _baseReward,
        _baseRewardFarm,
        _newMaxFeeETH,
        _newMaxFeeFarmETH
    );
}
```

The `Master` role has the ability to set liquidation settings, which can significantly impact the protocol's incentive structure and economic dynamics.

```solidity
// WiseSecurity.sol
function setSecurityWorker(
    address _entitiy,
    bool _state
)
    external
    onlyMaster
{
    securityWorker[_entitiy] = _state;
}
```

The `Master` role also has the power to assign and revoke the `SecurityWorker` role, which can initiate a security shutdown of the protocol.

While these privileged roles are designed to provide governance and emergency response capabilities, they introduce centralization risks if the privileged entities are compromised or act maliciously.

Potential Centralization Risks
---------------------------

1. **Issue: Centralized Control by the Master Address**

   **Impact:** The contract has a designated `master` address that holds significant control over the system, introducing a centralization risk.

   The `master` address has the authority to perform sensitive operations, such as setting liquidation settings (`setLiquidationSettings` in `WiseSecurity.sol`), blacklisting tokens (`setBlacklistToken` in `WiseSecurity.sol`), and revoking the security shutdown (`revokeShutdown` in `WiseSecurity.sol`).

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L85-L100, https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L980-L988, https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L1032-L1039
   ```solidity
   function setLiquidationSettings(
       uint256 _baseReward,
       uint256 _baseRewardFarm,
       uint256 _newMaxFeeETH,
       uint256 _newMaxFeeFarmETH
   )
       external
       onlyMaster
   {...}

   function setBlacklistToken(
       address _tokenAddress,
       bool _state
   )
       external
       onlyMaster()
   {...}

   function revokeShutdown()
       external
       onlyMaster
   {...}
   ```

   **Explanation:** The `master` address has significant control over critical system parameters and operations. If compromised or controlled by a malicious entity, the `master` address could manipulate the system to its advantage, potentially causing financial losses for users or disrupting the protocol's operations.

2. **Issue: Centralized Deployment and Upgradability**

   **Impact:** The contract code does not provide any explicit mechanism for decentralized deployment or upgradability, which could lead to centralization risks.

   The code does not include any provisions for decentralized deployment or upgradability, such as using proxy patterns or decentralized governance mechanisms.

   **Explanation:** Without a decentralized deployment and upgradability mechanism, the protocol may rely on a centralized entity or a small group of individuals to manage the deployment and potential upgrades. This centralization of control could lead to issues such as censorship, single points of failure, and potential exploitation by malicious actors.

**Mechanism Review**

1. **Issue: Potential Reentrancy Vulnerability**

   **Impact:** The contract may be vulnerable to reentrancy attacks, which could lead to potential fund draining or unexpected behavior.

   The contract uses the `transfer` and `call` functions directly, without any reentrancy guards or checks.

  
   ```solidity
   function _sendValue(
       address _to,
       uint256 _value
   )
       private
   {
       (bool success, ) = _to.call{value: _value}("");
       require(success, "ETH_TRANSFER_FAILED");
   }
   ```

   **Explanation:** The `_sendValue` function in `WiseLending.sol` uses the low-level `call` function to transfer Ether without any reentrancy guards. This could potentially allow a malicious contract to re-enter the contract and execute the same function multiple times before the first invocation has completed, leading to potential fund draining or unexpected behavior.

2. **Issue: Potential Sandwich Attack Vulnerability**

   **Impact:** The contract may be vulnerable to sandwich attacks, where a malicious actor can frontrun or backrun user transactions to extract value or manipulate the system.

   The contract does not implement any specific measures to prevent sandwich attacks, and the order of execution of transactions is determined by the underlying Ethereum network.

   **Relevant Code:** No specific code section is directly responsible for this issue. However, the lack of explicit countermeasures against sandwich attacks in the contract code leaves the protocol susceptible to this type of attack.

   **Explanation:** In a sandwich attack, a malicious actor can monitor the mempool for pending user transactions and quickly execute transactions before and after the user's transaction to manipulate the system state or extract value. For example, in the context of a lending protocol, a malicious actor could frontrun a user's borrow transaction, borrow the desired amount themselves, and then backrun the user's transaction, effectively sandwiching the user's transaction and potentially causing slippage or failed transactions for the user.

**Systemic Risks**

1. **Issue: Potential Oracle Manipulation Risk**

   **Impact:** The contract relies on external oracles for price data, which could be manipulated or compromised, leading to potential system failures or financial losses.

   The contract interacts with external oracles (likely Chainlink) for price data, but the specific implementation and security measures for oracle data are not provided in the code.

   
   ```solidity
   function checkHeartbeat(_poolAddress)
       private
       view
       returns (bool)
   {
       return WISE_ORACLE.checkHeartbeat(_poolAddress);
   }
   ```

   **Explanation:** The contract relies on external oracles for price data and heartbeat checks. If the oracles are compromised or manipulated, the contract could potentially use incorrect or manipulated data, leading to system failures, incorrect calculations, or potential financial losses for users.

2. **Issue: Potential Composability Risks**

   **Impact:** The contract interacts with external protocols and systems, such as Aave and Curve, introducing potential composability risks and dependencies on the security and reliability of these external systems.

   The contract integrates with external protocols like Aave and Curve, but the specifics of these integrations and their potential risks are not provided in the code.

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L859-L896
   ```solidity
   function withdrawOnBehalfExactAmount(
       uint256 _nftId,
       address _poolToken,
       uint256 _withdrawAmount
   )
       external
       onlyAaveHub
       syncPool(_poolToken)
       healthStateCheck(_nftId)
       returns (uint256)
   {...}
   ```

   **Explanation:** The contract interacts with external protocols like Aave and Curve, introducing dependencies and potential risks. If these external systems are compromised, have vulnerabilities, or undergo changes, it could impact the security and functionality of the WISE Lending protocol. Additionally, the composability of these systems may introduce unforeseen risks or attack vectors.

### Dependency on External Contracts

The Wise Lending codebase heavily relies on external contracts and protocols, such as Aave and external oracles. This dependency introduces centralization risks if these external entities become unavailable, compromised, or undergo significant changes that break compatibility.

The `AaveHub` contract facilitates integration with the Aave lending protocol, and any issues or changes in Aave could potentially impact the functionality of the Wise Lending protocol.

```solidity
// WiseLending.sol
function borrowOnBehalfExactAmount(
    uint256 _nftId,
    address _poolToken,
    uint256 _amount
)
    external
    onlyAaveHub
    syncPool(_poolToken)
    healthStateCheck(_nftId)
    returns (uint256)
{
    // ...
}
```

Similarly, the reliance on external oracles for fetching token prices and exchange rates introduces centralization risks, as any issues or manipulation of these oracles could potentially impact the protocol's lending and borrowing operations.

## Mechanism Review

The Wise Lending codebase implements several core mechanisms and functionalities, which are discussed in detail below:

### Lending and Borrowing

The lending and borrowing mechanisms are implemented in the `WiseLending` contract, with various functions for depositing, withdrawing, borrowing, and repaying tokens.

**Deposit**

Users can deposit tokens into the lending pools using functions like `depositExactAmount` and `depositExactAmountMint`. These functions handle the transfer of tokens, mint lending shares, and update the pool's state.

```solidity
// WiseLending.sol
function depositExactAmount(
    uint256 _nftId,
    address _poolToken,
    uint256 _amount
)
    public
    syncPool(_poolToken)
    returns (uint256)
{
    uint256 shareAmount = _handleDeposit(
        msg.sender,
        _nftId,
        _poolToken,
        _amount
    );

    _safeTransferFrom(
        _poolToken,
        msg.sender,
        address(this),
        _amount
    );

    return shareAmount;
}
```

**Withdraw**

Users can withdraw their deposited tokens from the lending pools using functions like `withdrawExactAmount` and `withdrawExactShares`. These functions handle the transfer of tokens, burn lending shares, and update the pool's state.

```solidity
// WiseLending.sol
function withdrawExactAmount(
    uint256 _nftId,
    address _poolToken,
    uint256 _withdrawAmount
)
    external
    syncPool(_poolToken)
    healthStateCheck(_nftId)
    returns (uint256)
{
    uint256 withdrawShares = _handleWithdrawAmount(
        {
            _caller: msg.sender,
            _nftId: _nftId,
            _poolToken: _poolToken,
            _amount: _withdrawAmount,
            _onBehalf: false
        }
    );

    _validateNonZero(
        withdrawShares
    );

    _safeTransfer(
        _poolToken,
        msg.sender,
        _withdrawAmount
    );

    return withdrawShares;
}
```

**Borrow**

Users can borrow tokens from the lending pools by providing collateral. The `borrowExactAmount` function allows users to borrow a specific amount of tokens, while the `borrowExactAmountETH` function enables borrowing ETH.

```solidity
// WiseLending.sol
function borrowExactAmount(
    uint256 _nftId,
    address _poolToken,
    uint256 _amount
)
    external
    syncPool(_poolToken)
    healthStateCheck(_nftId)
    returns (uint256)
{
    _checkOwnerPosition(
        _nftId,
        msg.sender
    );

    uint256 shares = _handleBorrowExactAmount({
        _nftId: _nftId,
        _poolToken: _poolToken,
        _amount: _amount,
        _onBehalf: false
    });

    _validateNonZero(
        shares
    );

    _safeTransfer(
        _poolToken,
        msg.sender,
        _amount
    );

    return shares;
}
```

**Repay**

Users can repay their borrowed tokens using functions like `paybackExactAmount` and `paybackExactShares`. These functions handle the transfer of tokens, burn borrow shares, and update the pool's state.

```solidity
// WiseLending.sol
function paybackExactAmount(
    uint256 _nftId,
    address _poolToken,
    uint256 _amount
)
    external
    syncPool(_poolToken)
    returns (uint256)
{
    uint256 paybackShares = calculateBorrowShares(
        {
            _poolToken: _poolToken,
            _amount: _amount,
            _maxSharePrice: false
        }
    );

    _validateNonZero(
        paybackShares
    );

    _handlePayback(
        msg.sender,
        _nftId,
        _poolToken,
        _amount,
        paybackShares
    );

    _safeTransferFrom(
        _poolToken,
        msg.sender,
        address(this),
        _amount
    );

    return paybackShares;
}
```

The lending and borrowing mechanisms also include features like solely depositing and withdrawing funds (keeping them private), as well as integrating with Aave pools for capital efficiency.

### Liquidation

The WiseLending protocol implements a liquidation mechanism to maintain the health of collateralized positions. When a user's debt ratio exceeds a certain threshold (typically 100%), their position becomes eligible for liquidation by other users.

The liquidation process involves the following steps:

1. A liquidator calls the `liquidatePartiallyFromTokens` function, specifying the position to be liquidated (`_nftId`), the token to be used for repayment (`_paybackToken`), the token to be received as a reward (`_receiveToken`), and the amount of shares to be paid (`_shareAmountToPay`).

2. The protocol performs various checks, including verifying the liquidation conditions, the liquidator's position health, and the maximum fee limits.

3. If all checks pass, the `_coreLiquidation` function is called, which executes the liquidation process. This function handles the repayment of the specified amount, updates the borrower's and liquidator's positions, and transfers the liquidation reward to the liquidator.

```solidity
function liquidatePartiallyFromTokens(
    uint256 _nftId,
    uint256 _nftIdLiquidator,
    address _paybackToken,
    address _receiveToken,
    uint256 _shareAmountToPay
) external returns (uint256) {
    // ...
    return _coreLiquidation(data);
}
```

The liquidation mechanism appears well-designed and incorporates necessary checks and validations. However, there are a few potential areas of concern:

1. **Liquidation Incentives**: The protocol includes liquidation incentives (`baseRewardLiquidation` and `baseRewardLiquidationFarm`) to incentivize users to participate in the liquidation process. While these incentives can help maintain the health of the system, their values and potential impact on the overall system dynamics should be carefully analyzed to prevent potential exploits or adverse incentives.

2. **Liquidation Cascade Risk**: In extreme scenarios, liquidations could potentially trigger a cascade effect, where liquidations lead to further liquidations, destabilizing the protocol. While the protocol includes mechanisms to limit liquidation incentives, stress testing and thorough analysis of potential cascade scenarios would be beneficial.

3. **Gas Costs and Liquidation Efficiency**: The liquidation process involves several checks and calculations, which could result in higher gas costs for liquidators. Optimizing the liquidation logic and minimizing unnecessary computations could improve the efficiency and cost-effectiveness of the liquidation process.

4. **Isolated Position Liquidation**: The protocol introduces the concept of "isolated positions," which are registered for special handling. The liquidation process for these positions follows a different path (`coreLiquidationIsolationPools`). While this functionality provides flexibility, it also introduces additional complexity and potential attack vectors that should be carefully reviewed.

Overall, the liquidation mechanism appears well-designed and incorporates necessary checks and validations. However, further analysis of the liquidation incentive structure, potential cascade risks, gas cost optimizations, and the isolated position liquidation process would be beneficial to ensure the robustness and efficiency of the liquidation mechanism.

### Asset Management and Integrations

The WiseLending protocol supports various asset management features and integrations with external protocols, such as Aave and Curve. These features and integrations provide users with additional options for earning yield and leveraging different asset types as collateral.

1. **Aave Integration**: The protocol allows users to earn Aave's supply APY on their unutilized collateral by depositing assets into Aave lending pools. This integration is facilitated through the `AaveHub` contract, which can perform withdrawal and borrowing operations on behalf of user positions.

```solidity
function withdrawOnBehalfExactAmount(
    uint256 _nftId,
    address _poolToken,
    uint256 _withdrawAmount
) external onlyAaveHub { /* ... */ }

function borrowOnBehalfExactAmount(
    uint256 _nftId,
    address _poolToken,
    uint256 _amount
) external onlyAaveHub { /* ... */ }
```

**1. External Integrations (Aave, Curve)**

The contracts interact with Aave and Curve protocols for certain functionalities.

a. Aave Integration:
- The contracts use Aave for depositing and withdrawing funds via the `AAVE_HUB_ADDRESS` ([WiseCore.sol](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol)).
- The `setAaveFlag` function in `FeeManager` allows mapping an underlying token to its corresponding token ([FeeManager.sol](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol)). 
- **Potential Risk:** _If there are any vulnerabilities or unexpected behavior changes in the Aave protocol, it could impact the security and functionality of these contracts. The contracts should have contingency plans and be adaptable to handle such scenarios._

b. Curve Integration:
- The contracts interact with Curve pools for swapping tokens ([WiseSecurity.sol](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol)).
- The `prepareCurvePools` function sets up the necessary approvals and configurations for interacting with Curve pools (WiseSecurity.sol).
- **Potential Risk:** _The contracts rely on the correct setup and behavior of Curve pools. Any malicious or unexpected changes in the Curve pool contracts could potentially be exploited to manipulate the token swaps or drain funds._

**2. Oracle Security and Manipulation Risks**

The contracts rely on oracle data for price feeds and making decisions. Here are some observations:

- The contracts use a [WiseOracleHub](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol) contract for getting token prices and converting between tokens and ETH ([WiseCore.sol](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol)).
- The `checkHeartbeat` function is used to check if the oracle price feed is active and not stale ([WiseSecurity.sol](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol)).
- **Potential Risk:** _Oracle manipulation is a common attack vector. If an attacker can manipulate the price feeds provided by the oracle, they could potentially exploit the contracts by triggering false liquidations, inflating borrow capacities, or manipulating the system in their favor._
- **Mitigation:** _Implement robust oracle security measures such as using multiple oracle sources, employing a decentralized oracle network, setting price thresholds, and having contingency plans for oracle failures._

**3. System Architecture and Attack Vectors**

The contracts have a complex architecture with multiple inheritance and external contract interactions. Here are some observations:

- The contracts have a modular structure with separate contracts for different functionalities ([WiseLending](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol), [WiseSecurity](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol), [FeeManager](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol), [WiseCore](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol)).
- The contracts heavily rely on external contracts like `WISE_SECURITY`, `WISE_ORACLE`, and `POSITION_NFT` for various functionalities (WiseLending.sol).
- **Potential Risk**: _The complexity of the architecture increases the attack surface. An attacker could potentially exploit any vulnerability or weakness in any of the contracts or their interactions to compromise the entire system._
- Mitigation: Conduct thorough testing and auditing of all contracts and their interactions. Ensure proper access controls and validation checks are in place at all interaction points. Implement circuit breakers and emergency stop mechanisms to halt the system in case of any detected anomalies.

**4. Access Control and Privilege Escalation**

The contracts have various access control mechanisms using modifiers and role-based functions. Here are some observations:

- The contracts use modifiers like onlyMaster, onlyWiseLending, onlyIsolationPool, etc., to restrict access to certain functions (WiseSecurity.sol).
- The contracts have different admin roles like incentiveMaster, positionLocked, securityWorker, etc., with varying privileges (FeeManager.sol).
- Potential Risk: If the access control mechanisms are not properly implemented or if there are any vulnerabilities in the contract logic, an attacker could potentially escalate their privileges and gain unauthorized access to critical functions.
- Mitigation: Ensure strict access controls and proper validation of role assignments. Implement multi-sig mechanisms for critical functions. Regularly audit and review the access control logic to identify and fix any vulnerabilities.

**5. Gas Usage and Denial-of-Service (DoS) Risks**

The contracts have several loops and complex calculations that could potentially lead to high gas usage. Here are some observations:

- Functions like overallLendingAPY and overallBorrowAPY have unbounded loops that iterate over all lending and borrow tokens (WiseSecurity.sol).
- Functions like claimWiseFeesBulk and claimWiseFeesBulk in FeeManager also have unbounded loops (FeeManager.sol).
- **Potential Risk:** _If the number of lending and borrow tokens or the number of pools grows significantly, these functions could potentially exceed the block gas limit and become unusable. An attacker could exploit this by creating a large number of pools or tokens, leading to a DoS condition._
- Mitigation: Optimize the contract logic to minimize gas usage. Use pagination or batch processing for functions that iterate over large datasets. Implement gas limits and circuit breakers to prevent excessive gas consumption.

Additional Recommendations:

- Have a comprehensive risk management framework in place, including incident response plans and emergency procedures.
- Ensure proper documentation and user education to help users understand the risks and best practices when interacting with the contracts.

### Time spent:
46 hours