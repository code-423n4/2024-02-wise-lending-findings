## Low Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [L-01] | Potential Gas Griefing Due to Non-Handling of Return Data in External Calls | 1 |
| [L-02] | `.call` bypasses function existence check, type checking and argument packing | 3 |
| [L-03] | Low Level Calls to Custom Addresses | 2 |
| [L-04] | Usage of Deprecated `latestAnswer()` Function | 1 |
| [L-05] | Unchecked Return Values of the `approve()` Function | 1 |
| [L-06] | Unsafe Conversion from various type `int/uint` values | 1 |
| [L-07] | Unsafe downcast from larger to smaller integer types | 2 |
| [L-08] | Unbounded loop may run out of gas | 1 |
| [L-09] | Functions calling tokens with transfer hooks are missing reentrancy guards | 42 |
| [L-10] | Upgradable contracts not taken into account | 3 |
| [L-11] | `internal/private` Function calls within for loops | 57 |
| [L-12] | Prevent Division by Zero | 7 |
| [L-13] | Initializer missing `address(0)` Check before assignment | 1 |
| [L-14] | For loops in public or external functions should be avoided due to high gas costs and possible `revert` | 21 |
| [L-15] | Potential Precision Errors due to Division Before Multiplication | 2 |
| [L-16] | Potential Re-org Attack Vector | 1 |
| [L-17] | Consider Adding a Denylist modifier to Your NFT Contracts | 2 |
| [L-18] | Subtraction may underflow if multiplication is too large | 2 |
| [L-19] | Unauthorized `receive()/fallback()` Functions | 2 |


## NonCritical Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [N-01] | Consider splitting long calculations | 12 |
| [N-02] | Missing Event Emission After Critical Initialize() Function | 1 |
| [N-03] | Prefer Custom Errors Over Unnamed `revert()` | 1 |
| [N-04] | Consider using descriptive `constant`s when passing zero as a function argument | 3 |
| [N-05] | Unused `private` functions can be removed | 8 |
| [N-06] | Consider using `delete` rather than assigning `false` to clear values | 8 |
| [N-07] | Consider using named returns | 242 |
| [N-08] | Style guide: Initialisms should be capitalized | 284 |
| [N-09] | Consider using named function arguments | 129 |
| [N-10] | Risk of Permanently Locked Ether | 4 |
| [N-11] | Contract/Library Names Not in `CapWords` Style (CamelCase) | 2 |
| [N-12] | Redundant Implementation of Ownable | 9 |
| [N-13] | Events are missing sender information | 51 |
| [N-14] | Consider Using OpenZeppelin's ReentrancyGuard Instead of Custom Implementation | 1 |
| [N-15] | High cyclomatic complexity | 2 |
| [N-16] | Contract should expose an `interface` | 250 |
| [N-17] | Contract uses both `require()`/`revert()` as well as custom errors | 21 |
| [N-18] | Solidity Version Too Recent to be Trusted | 44 |
| [N-19] | Use `bytes.concat()` instead of `abi.encodePacked()` | 6 |
| [N-20] | Function/Constructor Argument Names Not in mixedCase | 492 |
| [N-21] | Unnecessary Initialization of `address` with `address(0)` value | 2 |
| [N-22] | Contract Doesn't Handle All NFT Types | 1 |
| [N-23] | Duplicated `require()`/`revert()` checks should be refactored to a modifier or function | 13 |
| [N-24] | `constant` variables not using all capital letters | 2 |
| [N-25] | Non-uppercase/Constant-case Naming for `immutable` Variables | 3 |
| [N-26] | Function Names Not in mixedCase | 60 |
| [N-27] | Insufficient Comments in Complex Functions | 46 |
| [N-28] | Common functions should be refactored to a common base contract | 14 |
| [N-29] | Variable Names Not in mixedCase | 13 |
| [N-30] | Inconsistent Modifier Order in Function Declaration | 4 |
| [N-31] | Unused Function Parameters | 1 |
| [N-32] | Consider bounding input array length | 1 |
| [N-33] | Named Imports of Parent Contracts Are Missing | 37 |
| [N-34] | Prefer Casting to `bytes` or `bytes32` Over `abi.encodePacked()` for Single Arguments | 3 |
| [N-35] | Unused `Ownable` Inheritance in Contract | 9 |
| [N-36] | Numeric Values Related to Time Lacking Units | 3 |
| [N-37] | Unused import | 16 |
| [N-38] | Use of `override` is unnecessary | 2 |
| [N-39] | Setters should prevent re-setting of the same value | 46 |
| [N-40] | Use Unchecked for Divisions on Constant or Immutable Values | 3 |
| [N-41] | Explicit Visibility Recommended in Variable/Function Definitions | 11 |


## Low Findings Details

### [L-01] Potential Gas Griefing Due to Non-Handling of Return Data in External Calls

Due to the EVM architecture, return data (bool success,) has to be stored.
However, when 'out' and 'outsize' values are given (0,0), this storage disappears.
This can lead to potential gas griefing/theft issues, especially when dealing with external contracts.
```solidity
assembly {
    success: = call(gas(), dest, amount, 0, 0)
}
require(success, "transfer failed");

```
Consider using a safe call pattern above to avoid these issues.
The following instances show the unsafe external call patterns found in the code.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/WrapperHub/AaveHelper.sol

208: (bool success, ) = payable(_recipient).call{
```
[208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L208)
</details>

### [L-02] `.call` bypasses function existence check, type checking and argument packing

Using the `.call` method in Solidity enables direct communication with an address, bypassing function existence checks, type checking, and argument packing.
While this can save gas and provide flexibility, it can also introduce security risks and potential errors. The absence of these checks can lead to unexpected behavior if the callee contract's interface changes or if the input parameters are not crafted with care.
The resolution to these issues is to use Solidity's high-level interface for calling functions when possible, as it automatically manages these aspects.
If using `.call` is necessary, ensure that the inputs are carefully validated and that awareness of the called contract's behavior is maintained.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

208: curvePool.call{value: 0} (
            curveSwapInfoData[_poolToken].swapBytesPool
        )
225: curveMeta.call{value: 0} (
            curveSwapInfoData[_poolToken].swapBytesMeta
        )
```
[208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L208) | [225](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L225)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

78: _tokenAggregator.call(
            abi.encodeWithSignature(
                "maxAnswer()"
            )
        )
```
[78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L78)
</details>

### [L-03] Low Level Calls to Custom Addresses

Contracts should avoid making low-level calls to custom addresses, especially if these calls are based on address parameters in the function. 
Such behavior can lead to unexpected execution of untrusted code. Instead, consider using Solidity's high-level function calls or contract interactions.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/WrapperHub/AaveHelper.sol

208: (bool success, ) = payable(_recipient).call{
```
[208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L208)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

27: ) = payable(_recipient).call{
```
[27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L27)
</details>

### [L-04] Usage of Deprecated `latestAnswer()` Function

The function `latestAnswer` has been deprecated and is now considered unsafe to use.
It is recommended to use `latestRoundData` function instead.
The deprecated `latestAnswer` function returns zero if it fails to fetch data.
This can occur if the API, such as ChainLink, which the function depends on, discontinues its support.
As the `latestAnswer` function and its deprecation message have even been removed from the ChainLink website, the continued usage of this function in the contract poses a significant risk.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

40: return priceFeedPendleLpOracle.latestAnswer()
```
[40](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L40)
</details>

### [L-05] Unchecked Return Values of the `approve()` Function

The unchecked return value of the `approve()` method can potentially cause transaction failures to go unnoticed in your contract. 

Some IERC20 token implementations utilize boolean return values to indicate transaction failures, instead of relying on the `revert()` function. If the return value of the `approve()` method isn't appropriately verified, transactions may seemingly proceed even when the necessary token approvals have not been appropriately executed.

As a safeguard, it is crucial to consistently verify the return values of these methods. This precaution helps to ensure the correct execution of token approvals, maintaining the integrity of transaction operations and reducing the risk of unnoticed transaction failures.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

197: POSITION_NFT.approve(
                AAVE_HUB_ADDRESS,
                nftId
            );
```
[197](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L197)
</details>

### [L-06] Unsafe Conversion from various type `int/uint` values

Solidity follows two's complement rules for its integers. It's always unsafe to cast from one type to another.
Therefore, casting an unsigned integer to a signed one may result in an unexpected change of the sign or magnitude of the value.
For instance, `int8(type(uint8).max)` doesn't equal `type(int8).max` but is -1. [More information about Two's complement](https://en.wikipedia.org/wiki/Two%27s_complement)

Additionally, if the value from the source type exceeds the maximum value of the target type, it gets truncated by the modulo operation, leading to unpredictable outcomes.
This might introduce unintended behaviors in the contract logic.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

/// @audit cast from `uint192` to `int192`
157: int192(uint192(answer))
```
[157](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L157)
</details>

### [L-07] Unsafe downcast from larger to smaller integer types

Casting from larger types to smaller ones can potentially lead to overflows and thus unexpected behavior.
For instance, when a `uint256` value gets cast to `uint8`, it could end up as an entirely different, much smaller number due to the reduction in bits. 

OpenZeppelin's `SafeCast` library provides functions for safe type conversions, throwing an error whenever an overflow would occur.
It is generally recommended to use `SafeCast` or similar protective measures when performing type conversions to ensure the accuracy of your computations and the security of your contracts.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

/// @audit cast from `uint256` to `uint128`
379: ? uint128(currentExpiry)
/// @audit cast from `uint256` to `uint128`
407: uint128(_amount),
```
[379](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L379) | [407](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L407)
</details>

### [L-08] Unbounded loop may run out of gas

Unbounded loops in smart contracts pose a risk because they iterate over an unknown number of elements, potentially consuming all available gas for a transaction.
This can result in unintended transaction failures. Gas consumption increases linearly with the number of iterations, and if it surpasses the gas limit, the transaction reverts, wasting the gas spent.
To mitigate this, developers should either set a maximum limit on loop iterations.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/WrapperHub/AaveHub.sol

71: for (uint256 i = 0; i < _underlyingAssets.length; i++)
```
[71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L71)
</details>

### [L-09] Functions calling tokens with transfer hooks are missing reentrancy guards

Functions that call contracts or addresses with transfer hooks should use reentrancy guards for protection. 
Even if these functions adhere to the check-effects-interaction best practice, absence of reentrancy guards can expose the protocol users to read-only reentrancies.
Without the guards, the only protective measure is to block-list the entire protocol, which isn't an optimal solution.

<details>
<summary><i>42 issue instances in 13 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

/// @audit function `depositExactAmount()` 
476: _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        );
/// @audit function `solelyDeposit()` 
622: _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        );
/// @audit function `withdrawExactAmount()` 
738: _safeTransfer(
            _poolToken,
            msg.sender,
            _withdrawAmount
        );
/// @audit function `solelyWithdraw()` 
796: _safeTransfer(
            _poolToken,
            msg.sender,
            _withdrawAmount
        );
/// @audit function `withdrawOnBehalfExactAmount()` 
889: _safeTransfer(
            _poolToken,
            msg.sender,
            _withdrawAmount
        );
/// @audit function `withdrawExactShares()` 
926: _safeTransfer(
            _poolToken,
            msg.sender,
            withdrawAmount
        );
/// @audit function `withdrawOnBehalfExactShares()` 
960: _safeTransfer(
            _poolToken,
            msg.sender,
            withdrawAmount
        );
/// @audit function `borrowExactAmount()` 
1042: _safeTransfer(
            _poolToken,
            msg.sender,
            _amount
        );
/// @audit function `borrowOnBehalfExactAmount()` 
1073: _safeTransfer(
            _poolToken,
            msg.sender,
            _amount
        );
/// @audit function `paybackExactAmount()` 
1190: _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        );
/// @audit function `paybackExactShares()` 
1230: _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            repaymentAmount
        );
```
[476](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L476) | [622](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L622) | [738](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L738) | [796](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L796) | [889](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L889) | [926](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L926) | [960](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L960) | [1042](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1042) | [1073](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1073) | [1190](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1190) | [1230](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1230)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit function `addCompoundRewards()` 
518: _safeTransferFrom(
            UNDERLYING_PENDLE_MARKET,
            msg.sender,
            PENDLE_POWER_FARM_CONTROLLER,
            _amount
        );
/// @audit function `depositExactAmount()` 
579: _safeTransferFrom(
            UNDERLYING_PENDLE_MARKET,
            msg.sender,
            PENDLE_POWER_FARM_CONTROLLER,
            _underlyingLpAssetAmount
        );
```
[518](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L518) | [579](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L579)

```solidity
File: contracts/FeeManager/FeeManager.sol

/// @audit function `claimIncentives()` 
304: _safeTransfer(
            _feeToken,
            msg.sender,
            amount
        );
/// @audit function `claimFeesBeneficial()` 
710: _safeTransfer(
            _feeToken,
            caller,
            _amount
        );
/// @audit function `paybackBadDebtForToken()` 
788: _safeTransferFrom(
            _paybackToken,
            msg.sender,
            address(WISE_LENDING),
            paybackAmount
        );
/// @audit function `paybackBadDebtForToken()` 
795: _safeTransfer(
            _receivingToken,
            msg.sender,
            receivingAmount
        );
/// @audit function `paybackBadDebtNoReward()` 
860: _safeTransferFrom(
            _paybackToken,
            msg.sender,
            address(WISE_LENDING),
            paybackAmount
        );
```
[304](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L304) | [710](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L710) | [788](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L788) | [795](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L795) | [860](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L860)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

/// @audit function `_closingRouteToken()` 
365: _safeTransfer(
            WETH_ADDRESS,
            msg.sender,
            _totalDebtBalancer
        );
/// @audit function `_closingRouteToken()` 
371: _safeTransfer(
            WETH_ADDRESS,
            _caller,
            _tokenAmount
                - _totalDebtBalancer
        );
/// @audit function `_closingRouteETH()` 
394: _safeTransfer(
            WETH_ADDRESS,
            msg.sender,
            _totalDebtBalancer
        );
/// @audit function `_logicOpenPosition()` 
524: _safeTransfer(
            WETH_ADDRESS,
            BALANCER_ADDRESS,
            _totalDebtBalancer
        );
```
[365](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L365) | [371](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L371) | [394](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L394) | [524](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L524)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

/// @audit function `_checkFunctionExistence()` 
78: (bool success, ) = _tokenAggregator.call(
            abi.encodeWithSignature(
                "maxAnswer()"
            )
        );
```
[78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L78)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

/// @audit function `withdrawLp()` 
40: _safeTransfer(
            _pendleMarket,
            _to,
            _amount
        );
/// @audit function `exchangeRewardsForCompoundingWithIncentive()` 
86: _safeTransferFrom(
            _pendleMarket,
            msg.sender,
            address(this),
            sendingAmount
        );
/// @audit function `exchangeRewardsForCompoundingWithIncentive()` 
97: _safeTransfer(
            childInfo.rewardTokens[index],
            msg.sender,
            _rewardAmount
        );
/// @audit function `exchangeLpFeesForPendleWithIncentive()` 
142: _safeTransferFrom(
            PENDLE_TOKEN_ADDRESS,
            msg.sender,
            address(this),
            tokenAmountSend
        );
/// @audit function `exchangeLpFeesForPendleWithIncentive()` 
153: _safeTransfer(
            _pendleMarket,
            msg.sender,
            withdrawnAmount
        );
/// @audit function `skim()` 
202: _safeTransfer(
            _pendleMarket,
            master,
            difference
        );
/// @audit function `lockPendle()` 
397: _safeTransferFrom(
                    PENDLE_TOKEN_ADDRESS,
                    msg.sender,
                    address(this),
                    _amount
                );
/// @audit function `withdrawLock()` 
461: _safeTransfer(
                PENDLE_TOKEN_ADDRESS,
                master,
                amount
            );
/// @audit function `withdrawLock()` 
483: _safeTransfer(
            PENDLE_TOKEN_ADDRESS,
            master,
            amount
        );
/// @audit function `forwardETH()` 
607: payable(_to).transfer(
            _amount
        );
```
[40](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L40) | [86](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L86) | [97](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L97) | [142](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L142) | [153](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L153) | [202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L202) | [397](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L397) | [461](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L461) | [483](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L483) | [607](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L607)

```solidity
File: contracts/WiseCore.sol

/// @audit function `_coreLiquidation()` 
674: _safeTransferFrom(
            _data.tokenToPayback,
            _data.caller,
            address(this),
            _data.paybackAmount
        );
/// @audit function `_coreLiquidation()` 
681: _safeTransfer(
            _data.tokenToRecieve,
            _data.caller,
            receiveAmount
        );
```
[674](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L674) | [681](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L681)

```solidity
File: contracts/WrapperHub/AaveHub.sol

/// @audit function `skimAave()` 
623: _safeTransfer(
            tokenToSend,
            master,
            IERC20(tokenToSend).balanceOf(address(this))
        );
```
[623](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L623)

```solidity
File: contracts/PositionNFTs.sol

/// @audit function `forwardFeeManagerNFT()` 
83: _transfer(
            address(this),
            _feeManagerContract,
            FEE_MANAGER_NFT
        );
```
[83](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L83)

```solidity
File: contracts/PoolManager.sol

/// @audit function `_createPool()` 
269: _safeTransfer(
                _params.poolToken,
                master,
                fetchBalance
            );
```
[269](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L269)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

/// @audit function `enterFarm()` 
109: _safeTransferFrom(
            WETH_ADDRESS,
            msg.sender,
            address(this),
            _amount
        );
```
[109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L109)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

/// @audit function `_manuallyPaybackShares()` 
136: _safeTransferFrom(
            poolAddress,
            msg.sender,
            address(this),
            paybackAmount
        );
/// @audit function `_manuallyWithdrawShares()` 
174: _safeTransfer(
            PENDLE_CHILD,
            msg.sender,
            withdrawAmount
        );
```
[136](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L136) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L174)

```solidity
File: contracts/TransferHub/CallOptionalReturn.sol

/// @audit function `_callOptionalReturn()` 
22: ) = token.call(
            data
        );
```
[22](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L22)
</details>

### [L-10] Upgradable contracts not taken into account

In the realm of blockchain development, it's crucial to consider the impact of upgradable contracts, especially when handling token addresses through interfaces like IERC20.
These contracts can evolve over time, potentially altering their behavior or interface. Such changes may lead to compatibility issues or security vulnerabilities in the protocol that relies on them.

<details>
<summary><i>3 issue instances in 3 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

32: tokens[0] = IERC20(flashAsset);
```
[32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L32)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

25: _tokenDecimals[_tokenAddress] = IERC20(
            _tokenAddress
        ).decimals();
```
[25](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L25)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

173: IERC20 token = IERC20(
            aaveTokenAddress[_underlyingAsset]
        );
```
[173](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L173)
</details>

### [L-11] `internal/private` Function calls within for loops

Making function calls or external calls within loops in Solidity can lead to inefficient gas usage, potential bottlenecks, and increased vulnerability to attacks.
Each function call or external call consumes gas, and when executed within a loop, the gas cost multiplies, potentially causing the transaction to run out of gas or exceed block gas limits.
This can result in transaction failure or unpredictable behavior.

<details>
<summary><i>57 issue instances in 12 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

471: getPositionLendingAmount(
                _nftId,
                token
            )
482: getLendingRate(token)
528: getPositionBorrowAmount(
                _nftId,
                token
            )
539: getBorrowRate(token)
587: getETHBorrow(
                _nftId,
                token
            )
593: getBorrowRate(token)
609: getETHCollateral(
                _nftId,
                token
            )
616: getLendingRate(token)
671: checkHeartbeat(token)
680: getFullCollateralETH(
                    _nftId,
                    token
                )
716: _checkBlacklisted(token)
741: _checkBlacklisted(token)
```
[471](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L471) | [482](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L482) | [528](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L528) | [539](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L539) | [587](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L587) | [593](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L593) | [609](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L609) | [616](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L616) | [671](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L671) | [680](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L680) | [716](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L716) | [741](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L741)

```solidity
File: contracts/MainHelper.sol

440: _preparePool(
                currentAddress
            )
444: _newBorrowRate(
                currentAddress
            )
696: _deleteLastPositionData(
                    _nftId,
                    _poolToken
                )
704: _getPositionTokenByIndex(_nftId, i)
711: _getPositionTokenByIndex(
                _nftId,
                endPosition
            )
720: _deleteLastPositionData(
                _nftId,
                _poolToken
            )
```
[440](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L440) | [444](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L444) | [696](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L696) | [704](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L704) | [711](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L711) | [720](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L720)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

39: getFullCollateralETH(
                _nftId,
                tokenAddress
            )
86: _checkPoolCondition(
                tokenAddress
            )
91: getFullCollateralETH(
                    _nftId,
                    tokenAddress
                )
128: getFullCollateralETH(
                _nftId,
                tokenAddress
            )
167: _checkPoolCondition(
                tokenAddress
            )
172: _getCollateralOfTokenETHUpdated(
                    _nftId,
                    tokenAddress,
                    _interval
                )
374: getETHBorrow(
                _nftId,
                WISE_LENDING.getPositionBorrowTokenByIndex(
                    _nftId,
                    i
                )
            )
415: getETHBorrow(
                _nftId,
                tokenAddress
            )
451: _checkPoolCondition(
                tokenAddress
            )
455: getETHBorrow(
                _nftId,
                tokenAddress
            )
508: _checkPoolCondition(
                tokenAddress
            )
512: _getETHBorrowUpdated(
                _nftId,
                tokenAddress,
                _interval
            )
```
[39](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L39) | [86](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L86) | [91](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L91) | [128](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L128) | [167](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L167) | [172](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L172) | [374](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L374) | [415](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L415) | [451](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L451) | [455](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L455) | [508](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L508) | [512](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L512)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

237: _getUserReward(
                rewardTokens[i],
                PENDLE_POWER_FARM_CONTROLLER
            )
247: _getUserReward(
                    rewardTokens[i],
                    PENDLE_POWER_FARM_CONTROLLER
                )
257: _overWriteIndex(
                    i
                )
277: _getActiveBalance()
278: totalLpAssets()
279: _getBalanceLpBalanceController()
293: _overWriteIndex(
                i
            )
```
[237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L237) | [247](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L247) | [257](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L257) | [277](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L277) | [278](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L278) | [279](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L279) | [293](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L293)

```solidity
File: contracts/FeeManager/FeeManager.sol

93: _setAaveFlag(
                _poolTokens[i],
                _underlyingTokens[i]
            )
147: _checkValue(
                _newFees[i]
            )
267: claimIncentives(
                tokenAddress
            )
549: _setAllowedTokens(
                _user,
                _feeTokens[i],
                true
            )
582: _setAllowedTokens(
                _user,
                _feeTokens[i],
                false
            )
610: claimWiseFees(
                poolTokenAddresses[i]
            )
```
[93](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L93) | [147](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L147) | [267](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L267) | [549](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L549) | [582](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L582) | [610](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L610)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

627: _getRoundTimestamp(
                _tokenAddress,
                latestRoundId - i
            )
```
[627](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L627)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

262: _getUserRewardIndex(
                _pendleMarket,
                token,
                address(this)
            )
270: _checkFeed(
                token
            )
556: overWriteIndex(
                _pendleMarket,
                i
            )
```
[262](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L262) | [270](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L270) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L556)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

296: _getPool(
                _token0Array[i],
                _token1Array[i],
                _feeArray[i]
            )
302: _validatePoolAddress(
                pool,
                _uniPools[i]
            )
391: _addOracle(
                _tokenAddresses[i],
                _priceFeedAddresses[i],
                _underlyingFeedTokens[i]
            )
463: _chainLinkIsDead(
                underlyingFeedTokens[_tokenAddress][i]
            )
467: _validateAnswer(
                underlyingFeedTokens[_tokenAddress][i]
            )
510: _recalibrate(
                _tokenAddresses[i]
            )
```
[296](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L296) | [302](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L302) | [391](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L391) | [463](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L463) | [467](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L467) | [510](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L510)

```solidity
File: contracts/WrapperHub/AaveHub.sol

72: _setAaveTokenAddress(
                _underlyingAssets[i],
                _aaveTokens[i]
            )
```
[72](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L72)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

279: _syncSupply(
                activePendleMarkets[i]
            )
```
[279](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L279)

```solidity
File: contracts/PositionNFTs.sol

299: tokenOfOwnerByIndex(
                _owner,
                i
            )
```
[299](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L299)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

141: tokenOfOwnerByIndex(
                _owner,
                i
            )
```
[141](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L141)
</details>

### [L-12] Prevent Division by Zero

On several locations in the code, precautions are not being taken for not dividing by 0. This will revert the code.
These functions can be called with a 0 value in the input, this value is not checked for being bigger than 0,
that means in some scenarios this can potentially trigger a division by zero.

<details>
<summary><i>7 issue instances in 3 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

/// @audit ovrallETH - can be zero
491: return weightedRate
            / overallETH;
/// @audit ovrallETH - can be zero
548: return weightedRate
            / overallETH;
/// @audit totalETHSupply - can be zero
625: netAPY = (ethValueGain - ethValueDebt)
                / totalETHSupply;
/// @audit totalETHSupply - can be zero
631: netAPY = (ethValueDebt - ethValueGain)
                / totalETHSupply;
```
[491](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L491) | [548](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L548) | [625](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L625) | [631](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L631)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit totalSupply - can be zero
330: return previewUnderlyingLpAssets() * PRECISION_FACTOR_E18
            / totalSupply();
/// @audit undrlyingLpAsstsCurrnt - can be zero
490: ? product / _underlyingLpAssetsCurrent
```
[330](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L330) | [490](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L490)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

/// @audit twapPriodInt56 - can be zero
435: tickCumulativesDelta
            / twapPeriodInt56
```
[435](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L435)
</details>

### [L-13] Initializer missing `address(0)` Check before assignment

The initializer does not include a check for `address(0)` when initializing state variables that hold addresses.
Initializing a state variable with `address(0)` can lead to unintended behavior and vulnerabilities in the contract, 
such as sending funds to an inaccessible address. 
It is recommended to include a validation step to ensure that address parameters are not set to `address(0)`.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit `_underlyingPendleMarket` has lack of `address(0)` check before use
/// @audit `_pendleController` has lack of `address(0)` check before use
681: function initialize(
        address _underlyingPendleMarket,
        address _pendleController,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
    {
```
[681](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L681)
</details>

### [L-14] For loops in public or external functions should be avoided due to high gas costs and possible revert

Using loops within `public` or `external` functions can pose risks and inefficiencies.
Unpredictable gas consumption due to loop iterations can hinder a function's usability and cost-effectiveness. 
Furthermore, if the loop's logic can be externally influenced or altered, it might be exploited to intentionally drain gas, making the function impractical or uneconomical to call.
To ensure consistent performance and avoid potential vulnerabilities, it's advisable to avoid or limit loops in public functions, especially if their logic can change.

<details>
<summary><i>21 issue instances in 7 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

464: while (i < len) {
521: while (i < len) {
580: while (i < lenBorrow) {
602: while (i < lenDeposit) {
660: while (i < len) {
709: while (i < lenDeposit) {
734: while (i < lenBorrow) {
```
[464](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L464) | [521](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L521) | [580](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L580) | [602](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L602) | [660](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L660) | [709](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L709) | [734](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L734)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

32: while (i < l) {
79: while (i < l) {
121: while (i < l) {
372: while (i < l) {
408: while (i < l) {
444: while (i < l) {
```
[32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L32) | [79](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L79) | [121](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L121) | [372](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L372) | [408](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L408) | [444](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L444)

```solidity
File: contracts/FeeManager/FeeManager.sol

257: while (i < l) {
609: while (i < l) {
904: while (i < l) {
```
[257](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L257) | [609](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L609) | [904](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L904)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

461: while (i < length) {
509: while (i < l) {
```
[461](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L461) | [509](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L509)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

278: while (i < length) {
```
[278](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L278)

```solidity
File: contracts/PositionNFTs.sol

298: while (i < ownerTokenCount) {
```
[298](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L298)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

140: for (i; i < ownerTokenCount;) {
```
[140](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L140)
</details>

### [L-15] Potential Precision Errors due to Division Before Multiplication

Performing division operations before multiplication can lead to precision errors, due to truncation in Solidity's 
integer division. It is generally advisable to perform multiplication before division to preserve precision, especially 
when dealing with fractional values or computations requiring high accuracy.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

124: / POW_USD_FEED
            * POW_ETH_USD_FEED
            / uint256(answerEthUsdFeed)
            * ptToAssetRate
            / PRECISION_FACTOR_E18;
```
[124](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L124)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

105: / POW_FEED
            * ptToAssetRate
            / PRECISION_FACTOR_E18;
```
[105](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L105)
</details>

### [L-16] Potential Re-org Attack Vector

The contract appears to deploy new contracts using the `new` keyword.
In a re-org attack scenario, such deployments can be exploited by a malicious actor who might rewrite the blockchain's history and deploy the contract at an expected address. 

Consider deploying the contract via `CREATE2` opcode with a specific salt that includes `msg.sender` and the existing contract address.
This will ensure a predictable contract address, reducing the chances of such an attack.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

81: FeeManager feeManagerContract = new FeeManager(
```
[81](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L81)
</details>

### [L-17] Consider Adding a Denylist modifier to Your NFT Contracts

Given the rise in NFT thefts, it's important to implement safeguards to prevent hacked or stolen NFTs from being converted into liquidity on your platform. One such safeguard is a denylist modifier.

A denylist modifier would prevent reported stolen NFTs from being listed or transacted. Many marketplaces such as Opensea and NFT projects like Manifold already implement this feature in their smart contracts.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PositionNFTs.sol

10: contract PositionNFTs is ERC721Enumerable, OwnableMaster {
```
[10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L10)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

23: contract PowerFarmNFTs is ERC721Enumerable, OwnableMaster {
```
[23](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L23)
</details>

### [L-18] Subtraction may underflow if multiplication is too large

In scenarios where a large multiplication precedes a subtraction, there is a heightened risk of underflow. This can occur if the multiplication results in a significantly large value, potentially leading to an underflow when it is subsequently subtracted.
Such underflows can cause critical miscalculations, affecting the contract's logic and integrity.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/MainHelper.sol

177: return PRECISION_FACTOR_E18 - (PRECISION_FACTOR_E18
            * totalPool
            / pseudoPool
        );
```
[177](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L177)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

363: uint256 newUtilization = PRECISION_FACTOR_E18 - (PRECISION_FACTOR_E18
            * (totalPool - _borrowAmount)
            / pseudoPool
        );
```
[363](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L363)
</details>

### [L-19] Unauthorized `receive()/fallback()` Functions

Empty receive() or payable fallback() functions without proper authorization can result in a loss of funds. If the intention is for the Ether to be used, the function should call another function, otherwise, it should revert (e.g. require(msg.sender == address(weth))).

Having no access control on the function means that someone may send Ether to the contract and have no way to get anything back out. Even if the concern is spending a small amount of gas to check the sender, the code should at least have a function to rescue unused Ether, ensuring funds are not trapped within the contract.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

125: receive()
        external
        payable
```
[125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L125)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

21: receive()
        external
        payable
```
[21](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L21)
</details>


## NonCritical Findings Details

### [N-01] Consider splitting long calculations

For improved readability and maintainability, it's suggested to limit arithmetic operations to 3 per expression.
Excessive operations can convolute the code, making it more prone to errors and more difficult to debug or review.
Splitting operations across multiple lines is often a good approach.
Special care should be taken with division operations. The number of arithmetic operations per line ideally shouldn't exceed three.

<details>
<summary><i>12 issue instances in 8 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

566: _getInterest(_poolToken, _interval)
            * (PRECISION_FACTOR_E18 - WISE_LENDING.globalPoolData(_poolToken).poolFee)
            / PRECISION_FACTOR_E18
            + currentPseudo
```
[566](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L566)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

283: scaleNecessary
                ? indexDiff
                    * activeBalance
                    * totalLpAssetsCurrent
                    / lpBalanceController
                    / PRECISION_FACTOR_E18
                : indexDiff
                    * activeBalance
                    / PRECISION_FACTOR_E18
489: product % _underlyingLpAssetsCurrent == 0
            ? product / _underlyingLpAssetsCurrent
            : product / _underlyingLpAssetsCurrent + 1
```
[283](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L283) | [489](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L489)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

108: _decimalsETH < tokenDecimals
            ? _tokenAmount
                * latestResolver(_tokenAddress)
                / 10 ** decimals(_tokenAddress)
                / 10 ** (tokenDecimals - _decimalsETH)
            : _tokenAmount
                * 10 ** (_decimalsETH - tokenDecimals)
                * latestResolver(_tokenAddress)
                / 10 ** decimals(_tokenAddress)
159: _decimalsETH < tokenDecimals
            ? _tokenAmount
                * latestResolver(_tokenAddress)
                / 10 ** decimals(_tokenAddress)
                / 10 ** (tokenDecimals - _decimalsETH)
            : _tokenAmount
                * 10 ** (_decimalsETH - tokenDecimals)
                * latestResolver(_tokenAddress)
                / 10 ** decimals(_tokenAddress)
183: _decimalsETH < tokenDecimals
            ? _usdValue
                * 10 ** (tokenDecimals - _decimalsETH)
                * 10 ** decimals(_tokenAddress)
                / latestResolver(_tokenAddress)
            : _usdValue
                * 10 ** decimals(_tokenAddress)
                / latestResolver(_tokenAddress)
                / 10 ** (_decimalsETH - tokenDecimals)
343: _decimalsETH < tokenDecimals
            ? _ethAmount
                * 10 ** (tokenDecimals - _decimalsETH)
                * 10 ** decimals(_tokenAddress)
                / latestResolver(_tokenAddress)
            : _ethAmount
                * 10 ** decimals(_tokenAddress)
                / latestResolver(_tokenAddress)
                / 10 ** (_decimalsETH - tokenDecimals)
```
[108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L108) | [159](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L159) | [183](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L183) | [343](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L343)

```solidity
File: contracts/WrapperHub/AaveHub.sol

658: aaveRate
            * (PRECISION_FACTOR_E18 - utilization)
            / PRECISION_FACTOR_E18
            + lendingRate
```
[658](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L658)

```solidity
File: contracts/PoolManager.sol

285: PRECISION_FACTOR_E18 / 2
            + (PRECISION_FACTOR_E36 / 4
                + _poolMulFactor
                    * PRECISION_FACTOR_E36
                    / _poleBoundRate
            ).sqrt()
```
[285](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L285)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

122: uint256(answerUsdFeed)
            * PRECISION_FACTOR_E18
            / POW_USD_FEED
            * POW_ETH_USD_FEED
            / uint256(answerEthUsdFeed)
            * ptToAssetRate
            / PRECISION_FACTOR_E18
```
[122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L122)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

103: uint256(answerFeed)
            * PRECISION_FACTOR_E18
            / POW_FEED
            * ptToAssetRate
            / PRECISION_FACTOR_E18
```
[103](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L103)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

40: priceFeedPendleLpOracle.latestAnswer()
            * pendleChildToken.totalLpAssets()
            * PRECISION_FACTOR_E18
            / pendleChildToken.totalSupply()
            / PRECISION_FACTOR_E18
```
[40](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L40)
</details>

### [N-02] Missing Event Emission After Critical Initialize() Function

Emitting an initialization event offers clear, on-chain evidence of the contract's initialization state, enhancing transparency and auditability.
This practice aids users and developers in accurately tracking the contract's lifecycle, pinpointing the precise moment of its initialization.
Moreover, it aligns with best practices for event logging in smart contracts, ensuring that significant state changes are both observable and verifiable through emitted events.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

682: function initialize(
        address _underlyingPendleMarket,
        address _pendleController,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
```
[682](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L682)
</details>

### [N-03] Prefer Custom Errors Over Unnamed `revert()`

Using custom error types with `revert()` in Solidity provides clearer, more informative error handling compared to unnamed `revert()` calls.
Custom errors, introduced in Solidity 0.8.4, allow you to define specific error conditions with descriptive names and optionally include additional data about the error.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/TransferHub/CallOptionalReturn.sol

32: revert()
```
[32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L32)
</details>

### [N-04] Consider using descriptive `constant`s when passing zero as a function argument

In instances where utilizing a zero parameter is essential, it is recommended to employ descriptive constants or an enum instead of directly integrating zero within function calls.
This strategy aids in clearly articulating the caller's intention and minimizes the risk of errors.
Emitting zero also not recomended, as it is not clear what the intention is.

<details>
<summary><i>3 issue instances in 3 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

143: _withdrawLp(
            UNDERLYING_PENDLE_MARKET,
            0
        )
```
[143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L143)

```solidity
File: contracts/PoolManager.sol

169: _validateParameter(
            timestampsPoolData[_params.poolToken].timeStamp,
            0
        )
```
[169](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L169)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

161: UpdateBadDebtPosition(
                _nftId,
                0,
                block.timestamp
            )
```
[161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L161)
</details>

### [N-05] Unused `private` functions can be removed

All unused code should be removed.

<details>
<summary><i>8 issue instances in 3 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

1460: function _handleWithdrawAmount(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        bool _onBehalf
    )
        private
        returns (uint256 withdrawShares)
1517: function _handleWithdrawShares(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _shares,
        bool _onBehalf
    )
        private
        returns (uint256)
1553: function _handleBorrowExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        bool _onBehalf
    )
        private
        returns (uint256)
```
[1460](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1460) | [1517](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1517) | [1553](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1553)

```solidity
File: contracts/MainHelper.sol

667: function _removePositionData(
        uint256 _nftId,
        address _poolToken,
        function(uint256) view returns (uint256) _getPositionTokenLength,
        function(uint256, uint256) view returns (address) _getPositionTokenByIndex,
        function(uint256, address) internal _deleteLastPositionData,
        bool isLending
    )
        private
733: function _deleteLastPositionLendingData(
        uint256 _nftId,
        address _poolToken
    )
        private
794: function _deleteLastPositionBorrowData(
        uint256 _nftId,
        address _poolToken
    )
        private
```
[667](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L667) | [733](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L733) | [794](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L794)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

243: function _prepareCollaterals(
        uint256 _nftId,
        address _poolToken
    )
        private
279: function _prepareBorrows(
        uint256 _nftId,
        address _poolToken
    )
        private
```
[243](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L243) | [279](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L279)
</details>

### [N-06] Consider using `delete` rather than assigning `false` to clear values

The use of the delete keyword is recommended over simply assigning values to false when you intend to reset the state of a variable.
The delete keyword more closely aligns with the semantic intent of clearing or resetting a variable.
This practice also makes the code more readable and highlights the change in state, which may encourage a more thorough audit of the surrounding logic.

<details>
<summary><i>8 issue instances in 6 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

88: powerFarmCheck = false
342: userLendingData[_nftId][_poolToken].unCollateralized = false
```
[88](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L88) | [342](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L342)

```solidity
File: contracts/MainHelper.sol

740: hashMapPositionLending[
            _getHash(
                _nftId,
                _poolToken
            )
        ] = false
801: hashMapPositionBorrow[
            _getHash(
                _nftId,
                _poolToken
            )
        ] = false
```
[740](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L740) | [801](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L801)

```solidity
File: contracts/FeeManager/FeeManager.sol

476: poolTokenAdded[_poolToken] = false
```
[476](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L476)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

71: allowEnter = false
```
[71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L71)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

212: sendingProgressAaveHub = false
```
[212](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L212)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

31: sendingProgress = false
```
[31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L31)
</details>

### [N-07] Consider using named returns

Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas.
The cases below are where there currently is at most one return statement, which is ideal for named returns.

<details>
<summary><i>242 issue instances in 29 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

165: function _getSharePrice(
        address _poolToken
    )
        private
        view
        returns (
            uint256,
            uint256
        )
255: function _getCurrentSharePriceMax(
        address _poolToken
    )
        private
        view
        returns (uint256)
388: function depositExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        syncPool(WETH_ADDRESS)
        returns (uint256)
401: function _depositExactAmountETH(
        uint256 _nftId
    )
        private
        returns (uint256)
426: function depositExactAmountETHMint()
        external
        payable
        syncPool(WETH_ADDRESS)
        returns (uint256)
442: function depositExactAmountMint(
        address _poolToken,
        uint256 _amount
    )
        external
        returns (uint256)
460: function depositExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        public
        syncPool(_poolToken)
        returns (uint256)
636: function withdrawExactAmountETH(
        uint256 _nftId,
        uint256 _amount
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
        returns (uint256)
675: function withdrawExactSharesETH(
        uint256 _nftId,
        uint256 _shares
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
        returns (uint256)
714: function withdrawExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _withdrawAmount
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
859: function withdrawOnBehalfExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _withdrawAmount
    )
        external
        onlyAaveHub
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
902: function withdrawExactShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
939: function withdrawOnBehalfExactShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        external
        onlyAaveHub
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
975: function borrowExactAmountETH(
        uint256 _nftId,
        uint256 _amount
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
        returns (uint256)
1016: function borrowExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
1055: function borrowOnBehalfExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        external
        onlyAaveHub
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
1088: function paybackExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        syncPool(WETH_ADDRESS)
        returns (uint256)
1161: function paybackExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        external
        syncPool(_poolToken)
        returns (uint256)
1204: function paybackExactShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        external
        syncPool(_poolToken)
        returns (uint256)
1250: function liquidatePartiallyFromTokens(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _paybackToken,
        address _receiveToken,
        uint256 _shareAmountToPay
    )
        external
        syncPool(_paybackToken)
        syncPool(_receiveToken)
        returns (uint256)
1314: function coreLiquidationIsolationPools(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _caller,
        address _paybackToken,
        address _receiveToken,
        uint256 _paybackAmount,
        uint256 _shareAmountToPay
    )
        external
        syncPool(_paybackToken)
        syncPool(_receiveToken)
        returns (uint256)
1517: function _handleWithdrawShares(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _shares,
        bool _onBehalf
    )
        private
        returns (uint256)
1553: function _handleBorrowExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        bool _onBehalf
    )
        private
        returns (uint256)
1588: function _reservePosition()
        private
        returns (uint256)
```
[165](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L165) | [255](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L255) | [388](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) | [401](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L401) | [426](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L426) | [442](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L442) | [460](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L460) | [636](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L636) | [675](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L675) | [714](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L714) | [859](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L859) | [902](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L902) | [939](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L939) | [975](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L975) | [1016](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1016) | [1055](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1055) | [1088](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1088) | [1161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1161) | [1204](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1204) | [1250](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1250) | [1314](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1314) | [1517](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1517) | [1553](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1553) | [1588](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1588)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

59: function getLiveDebtRatio(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
442: function overallLendingAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
499: function overallBorrowAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
556: function overallNetAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256, bool)
641: function safeLimitPosition(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
695: function positionBlackListToken(
        uint256 _nftId
    )
        external
        view
        returns (bool, address)
764: function maximumWithdrawToken(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval,
        uint256 _solelyWithdrawAmount
    )
        external
        view
        returns (uint256)
826: function maximumWithdrawTokenSolely(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval,
        uint256 _poolWithdrawAmount
    )
        external
        view
        returns (uint256)
906: function getExpectedPaybackAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        external
        view
        returns (uint256)
943: function getExpectedLendingAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        public
        view
        returns (uint256)
1060: function checkPoolWithMinDeposit(
        address _token,
        uint256 _amount
    )
        external
        view
        returns (bool)
1078: function checkMinDepositValue(
        address _token,
        uint256 _amount
    )
        external
        view
        returns (bool)
1092: function _checkMinDepositValue(
        address _token,
        uint256 _amount
    )
        private
        view
        returns (bool)
```
[59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L59) | [442](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L442) | [499](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L499) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L556) | [641](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L641) | [695](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L695) | [764](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L764) | [826](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L826) | [906](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L906) | [943](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L943) | [1060](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1060) | [1078](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1078) | [1092](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1092)

```solidity
File: contracts/MainHelper.sol

17: function calculateLendingShares(
        address _poolToken,
        uint256 _amount,
        bool _maxSharePrice
    )
        public
        view
        returns (uint256)
33: function _calculateShares(
        uint256 _product,
        uint256 _pseudo,
        bool _maxSharePrice
    )
        private
        pure
        returns (uint256)
55: function calculateBorrowShares(
        address _poolToken,
        uint256 _amount,
        bool _maxSharePrice
    )
        public
        view
        returns (uint256)
79: function cashoutAmount(
        address _poolToken,
        uint256 _shares
    )
        external
        view
        returns (uint256)
93: function _cashoutAmount(
        address _poolToken,
        uint256 _shares
    )
        internal
        view
        returns (uint256)
114: function paybackAmount(
        address _poolToken,
        uint256 _shares
    )
        public
        view
        returns (uint256)
135: function _preparationsWithdraw(
        uint256 _nftId,
        address _caller,
        address _poolToken,
        uint256 _amount
    )
        internal
        view
        returns (uint256)
163: function _getValueUtilization(
        address _poolToken
    )
        private
        view
        returns (uint256)
202: function _checkCleanUp(
        uint256 _amountContract,
        uint256 _totalPool,
        uint256 _bareAmount
    )
        private
        pure
        returns (bool)
278: function _getBalance(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
350: function _getAllowedDifference(
        address _poolToken
    )
        private
        view
        returns (uint256)
391: function _preparationTokens(
        mapping(uint256 => address[]) storage _userTokenData,
        uint256 _nftId,
        address _poolToken
    )
        internal
        returns (address[] memory)
647: function _getHash(
        uint256 _nftId,
        address _poolToken
    )
        private
        pure
        returns (bytes32)
814: function isUncollateralized(
        uint256 _nftId,
        address _poolToken
    )
        external
        view
        returns (bool)
830: function _checkLendingDataEmpty(
        uint256 _nftId,
        address _poolToken
    )
        private
        view
        returns (bool)
896: function _aboveThreshold(
        address _poolToken
    )
        internal
        view
        returns (bool)
987: function _resonanceOutcome(
        address _poolToken,
        uint256 _shareValue
    )
        private
        view
        returns (bool)
```
[17](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L17) | [33](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L33) | [55](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L55) | [79](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L79) | [93](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L93) | [114](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L114) | [135](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L135) | [163](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L163) | [202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L202) | [278](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L278) | [350](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L350) | [391](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L391) | [647](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L647) | [814](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L814) | [830](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L830) | [896](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L896) | [987](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L987)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

15: function overallETHCollateralsBoth(
        uint256 _nftId
    )
        public
        view
        returns (uint256, uint256)
224: function _isUncollateralized(
        uint256 _nftId,
        address _poolToken
    )
        internal
        view
        returns (bool)
281: function getETHCollateralUpdated(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        public
        view
        returns (uint256)
323: function getETHCollateral(
        uint256 _nftId,
        address _poolToken
    )
        public
        view
        returns (uint256)
340: function _getTokensInEth(
        address _poolToken,
        uint256 _amount
    )
        internal
        view
        returns (uint256)
470: function _checkBlacklisted(
        address _poolToken
    )
        internal
        view
        returns (bool)
530: function _getUpdatedPseudoBorrow(
        address _poolToken,
        uint256 _interval
    )
        internal
        view
        returns (uint256)
554: function _getUpdatedPseudoPool(
        address _poolToken,
        uint256 _interval
    )
        internal
        view
        returns (uint256)
578: function _getInterest(
        address _poolToken,
        uint256 _interval
    )
        internal
        view
        returns (uint256)
610: function _getETHBorrowUpdated(
        uint256 _nftId,
        address _poolToken,
        uint256 _intervall
    )
        internal
        view
        returns (uint256)
651: function getETHBorrow(
        uint256 _nftId,
        address _poolToken
    )
        public
        view
        returns (uint256)
687: function checkHeartbeat(
        address _poolToken
    )
        public
        view
        returns (bool)
716: function checkMaxFee(
        uint256 _paybackETH,
        uint256 _feeLiquidation,
        uint256 _maxFeeETH
    )
        external
        pure
        returns (uint256)
736: function _checkMaxFee(
        uint256 _paybackETH,
        uint256 _liquidationFee,
        uint256 _maxFeeETH
    )
        internal
        pure
        returns (uint256)
760: function calculateWishPercentage(
        uint256 _nftId,
        address _receiveToken,
        uint256 _paybackETH,
        uint256 _maxFeeETH,
        uint256 _baseRewardLiquidation
    )
        external
        view
        returns (uint256)
806: function _getState(
        uint256 _nftId,
        bool _powerFarm
    )
        internal
        view
        returns (bool)
906: function checkBadDebtThreshold(
        uint256 _borrowETHTotal,
        uint256 _unweightedCollateral
    )
        public
        pure
        returns (bool)
922: function getPositionLendingAmount(
        uint256 _nftId,
        address _poolToken
    )
        public
        view
        returns (uint256)
945: function getPositionBorrowAmount(
        uint256 _nftId,
        address _poolToken
    )
        public
        view
        returns (uint256)
985: function getBorrowRate(
        address _poolToken
    )
        public
        view
        returns (uint256)
999: function getLendingRate(
        address _poolToken
    )
        public
        view
        returns (uint256)
1029: function _getPossibleWithdrawAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        internal
        view
        returns (uint256)
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L15) | [224](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L224) | [281](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L281) | [323](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L323) | [340](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L340) | [470](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L470) | [530](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L530) | [554](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L554) | [578](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L578) | [610](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L610) | [651](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L651) | [687](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L687) | [716](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L716) | [736](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L736) | [760](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L760) | [806](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L806) | [906](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L906) | [922](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L922) | [945](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L945) | [985](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L985) | [999](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L999) | [1029](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1029)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

98: function _validateSharePrice(
        uint256 _sharePriceBefore
    )
        private
        view
        returns (uint256)
160: function _updateRewardTokens()
        private
        returns (bool)
218: function _calculateRewardsClaimedOutside()
        internal
        returns (uint256[] memory)
305: function _getBalanceLpBalanceController()
        private
        view
        returns (uint256)
315: function _getActiveBalance()
        private
        view
        returns (uint256)
325: function _getSharePrice()
        private
        view
        returns (uint256)
372: function _getUserReward(
        address _rewardToken,
        address _user
    )
        internal
        view
        returns (UserReward memory)
386: function previewDistribution()
        public
        view
        returns (uint256)
422: function _applyMintFee(
        uint256 _amount
    )
        internal
        view
        returns (uint256)
434: function totalLpAssets()
        public
        view
        returns (uint256)
443: function previewUnderlyingLpAssets()
        public
        view
        returns (uint256)
452: function previewMintShares(
        uint256 _underlyingAssetAmount,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view
        returns (uint256)
465: function previewAmountWithdrawShares(
        uint256 _shares,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view
        returns (uint256)
478: function previewBurnShares(
        uint256 _underlyingAssetAmount,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view
        returns (uint256)
494: function manualSync()
        external
        syncSupply
        returns (bool)
529: function depositExactAmount(
        uint256 _underlyingLpAssetAmount
    )
        external
        syncSupply
        returns (
            uint256,
            uint256
        )
608: function withdrawExactShares(
        uint256 _shares
    )
        external
        syncSupply
        returns (uint256)
647: function withdrawExactAmount(
        uint256 _underlyingLpAssetAmount
    )
        external
        syncSupply
        returns (uint256)
```
[98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L98) | [160](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L160) | [218](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L218) | [305](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L305) | [315](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L315) | [325](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L325) | [372](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L372) | [386](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L386) | [422](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L422) | [434](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L434) | [443](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L443) | [452](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L452) | [465](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L465) | [478](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L478) | [494](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L494) | [529](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L529) | [608](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L608) | [647](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L647)

```solidity
File: contracts/FeeManager/FeeManager.sol

872: function getPoolTokenAddressesLength()
        public
        view
        returns (uint256)
884: function getPoolTokenAdressesByIndex(
        uint256 _index
    )
        external
        view
        returns (address)
```
[872](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L872) | [884](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L884)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

232: function _getEthBack(
        uint256 _swapAmount,
        uint256 _minOutAmount
    )
        internal
        returns (uint256)
264: function _getTokensUniV3(
        uint256 _amountIn,
        uint256 _minOutAmount,
        address _tokenIn,
        address _tokenOut
    )
        internal
        returns (uint256)
293: function _swapStETHintoETH(
        uint256 _swapAmount,
        uint256 _minOutAmount
    )
        internal
        returns (uint256)
```
[232](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L232) | [264](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L264) | [293](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L293)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

60: function _checkFunctionExistence(
        address _tokenAggregator
    )
        internal
        returns (bool)
106: function latestResolverTwap(
        address _tokenAddress
    )
        public
        view
        returns (uint256)
131: function _validateAnswer(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
216: function _getRelativeDifference(
        uint256 _answerUint256,
        uint256 _fetchTwapValue
    )
        internal
        pure
        returns (uint256)
246: function _getChainlinkAnswer(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
265: function getETHPriceInUSD()
        public
        view
        returns (uint256)
374: function _getTwapPrice(
        address _tokenAddress,
        address _oracle
    )
        internal
        view
        returns (uint256)
394: function _getOneUnit(
        address _tokenAddress
    )
        internal
        view
        returns (uint128)
406: function _getAverageTick(
        address _oracle
    )
        internal
        view
        returns (int24)
450: function decimals(
        address _tokenAddress
    )
        public
        view
        returns (uint8)
460: function _getTwapDerivatePrice(
        address _tokenAddress,
        UniTwapPoolInfo memory _uniTwapPoolInfo
    )
        internal
        view
        returns (uint256)
521: function sequencerIsDead()
        public
        view
        returns (bool)
556: function _chainLinkIsDead(
        address _tokenAddress
    )
        internal
        view
        returns (bool)
592: function _recalibratePreview(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
674: function _getRoundTimestamp(
        address _tokenAddress,
        uint80 _roundId
    )
        internal
        view
        returns (uint256)
```
[60](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L60) | [106](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L106) | [131](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L131) | [216](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L216) | [246](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L246) | [265](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L265) | [374](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L374) | [394](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L394) | [406](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L406) | [450](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L450) | [460](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L460) | [521](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L521) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L556) | [592](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L592) | [674](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L674)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

53: function exchangeRewardsForCompoundingWithIncentive(
        address _pendleMarket,
        address _rewardToken,
        uint256 _rewardAmount
    )
        external
        syncSupply(_pendleMarket)
        returns (uint256)
113: function exchangeLpFeesForPendleWithIncentive(
        address _pendleMarket,
        uint256 _pendleChildShares
    )
        external
        syncSupply(_pendleMarket)
        returns (
            uint256,
            uint256
        )
172: function skim(
        address _pendleMarket
    )
        external
        returns (uint256)
293: function updateRewardTokens(
        address _pendleMarket
    )
        external
        onlyChildContract(_pendleMarket)
        returns (bool)
```
[53](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L53) | [113](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L113) | [172](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L172) | [293](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L293)

```solidity
File: contracts/WiseCore.sol

15: function _prepareAssociatedTokens(
        uint256 _nftId,
        address _poolTokenLend,
        address _poolTokenBorrow
    )
        internal
        returns (
            address[] memory,
            address[] memory
        )
106: function _handleDeposit(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        internal
        returns (uint256)
460: function _withdrawOrAllocateSharesLiquidation(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _poolToken,
        uint256 _percentWishCollateral
    )
        private
        returns (uint256)
604: function _calculatePotentialPureExtraCashout(
        uint256 _userShares,
        address _poolToken,
        uint256 _removePercentage
    )
        private
        view
        returns (uint256)
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L15) | [106](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L106) | [460](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L460) | [604](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L604)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

69: function latestResolver(
        address _tokenAddress
    )
        public
        view
        returns (uint256)
85: function getTokenDecimals(
        address _tokenAddress
    )
        external
        view
        returns (uint8)
96: function getTokensInUSD(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        external
        view
        returns (uint256)
123: function getTokensPriceInUSD(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        external
        view
        returns (uint256)
143: function getTokensInETH(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        public
        view
        returns (uint256)
171: function getTokensFromUSD(
        address _tokenAddress,
        uint256 _usdValue
    )
        external
        view
        returns (uint256)
198: function getTokensPriceFromUSD(
        address _tokenAddress,
        uint256 _usdValue
    )
        external
        view
        returns (uint256)
327: function getTokensFromETH(
        address _tokenAddress,
        uint256 _ethAmount
    )
        public
        view
        returns (uint256)
419: function recalibratePreview(
        address _tokenAddress
    )
        external
        view
        returns (uint256)
```
[69](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L69) | [85](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L85) | [96](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L96) | [123](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L123) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L143) | [171](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L171) | [198](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L198) | [327](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L327) | [419](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L419)

```solidity
File: contracts/WrapperHub/AaveHub.sol

103: function depositExactAmountMint(
        address _underlyingAsset,
        uint256 _amount
    )
        external
        returns (uint256)
122: function depositExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _amount
    )
        public
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
158: function depositExactAmountETHMint()
        external
        payable
        returns (uint256)
172: function depositExactAmountETH(
        uint256 _nftId
    )
        public
        payable
        nonReentrant
        returns (uint256)
202: function withdrawExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _withdrawAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
238: function withdrawExactAmountETH(
        uint256 _nftId,
        uint256 _withdrawAmount
    )
        external
        nonReentrant
        returns (uint256)
281: function withdrawExactShares(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _shareAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
314: function withdrawExactSharesETH(
        uint256 _nftId,
        uint256 _shareAmount
    )
        external
        nonReentrant
        returns (uint256)
357: function borrowExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _borrowAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
393: function borrowExactAmountETH(
        uint256 _nftId,
        uint256 _borrowAmount
    )
        external
        nonReentrant
        returns (uint256)
433: function paybackExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _paybackAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
482: function paybackExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        nonReentrant
        returns (uint256)
556: function paybackExactShares(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _shares
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
635: function getLendingRate(
        address _underlyingAsset
    )
        external
        view
        returns (uint256)
```
[103](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L103) | [122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L122) | [158](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L158) | [172](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L172) | [202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L202) | [238](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L238) | [281](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L281) | [314](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L314) | [357](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L357) | [393](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L393) | [433](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L433) | [482](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L482) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L556) | [635](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L635)

```solidity
File: contracts/WiseLowLevelHelper.sol

39: function getTotalPool(
        address _poolToken
    )
        public
        view
        returns (uint256)
49: function getPseudoTotalPool(
        address _poolToken
    )
        public
        view
        returns (uint256)
59: function getTotalBareToken(
        address _poolToken
    )
        external
        view
        returns (uint256)
69: function getPseudoTotalBorrowAmount(
        address _poolToken
    )
        external
        view
        returns (uint256)
79: function getTotalDepositShares(
        address _poolToken
    )
        external
        view
        returns (uint256)
89: function getTotalBorrowShares(
        address _poolToken
    )
        external
        view
        returns (uint256)
99: function getPositionLendingShares(
        uint256 _nftId,
        address _poolToken
    )
        external
        view
        returns (uint256)
110: function getPositionBorrowShares(
        uint256 _nftId,
        address _poolToken
    )
        external
        view
        returns (uint256)
121: function getPureCollateralAmount(
        uint256 _nftId,
        address _poolToken
    )
        external
        view
        returns (uint256)
134: function getTimeStamp(
        address _poolToken
    )
        external
        view
        returns (uint256)
144: function getPositionLendingTokenByIndex(
        uint256 _nftId,
        uint256 _index
    )
        public
        view
        returns (address)
155: function getPositionLendingTokenLength(
        uint256 _nftId
    )
        public
        view
        returns (uint256)
165: function getPositionBorrowTokenByIndex(
        uint256 _nftId,
        uint256 _index
    )
        public
        view
        returns (address)
176: function getPositionBorrowTokenLength(
        uint256 _nftId
    )
        public
        view
        returns (uint256)
363: function _sendingProgressAaveHub()
        private
        view
        returns (bool)
393: function _byPassCase(
        address _sender
    )
        internal
        view
        returns (bool)
```
[39](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L39) | [49](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L49) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L59) | [69](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L69) | [79](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L79) | [89](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L89) | [99](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L99) | [110](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L110) | [121](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L121) | [134](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L134) | [144](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L144) | [155](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L155) | [165](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L165) | [176](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L176) | [363](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L363) | [393](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L393)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

47: function _reservePosition()
        internal
        returns (uint256)
56: function _wrapDepositExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _depositAmount
    )
        internal
        returns (uint256)
83: function _wrapWithdrawExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        address _underlyingAssetRecipient,
        uint256 _withdrawAmount
    )
        internal
        returns (
            uint256,
            uint256
        )
113: function _wrapWithdrawExactShares(
        uint256 _nftId,
        address _underlyingAsset,
        address _underlyingAssetRecipient,
        uint256 _shareAmount
    )
        internal
        returns (uint256)
141: function _wrapBorrowExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        address _underlyingAssetRecipient,
        uint256 _borrowAmount
    )
        internal
        returns (uint256)
219: function _getInfoPayback(
        uint256 _ethSent,
        uint256 _maxPaybackAmount
    )
        internal
        pure
        returns (
            uint256,
            uint256
        )
315: function getAavePoolAPY(
        address _underlyingAsset
    )
        public
        view
        returns (uint256)
```
[47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L47) | [56](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L56) | [83](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L83) | [113](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L113) | [141](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L141) | [219](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L219) | [315](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L315)

```solidity
File: contracts/PositionNFTs.sol

90: function reservePosition()
        external
        returns (uint256)
99: function reservePositionForUser(
        address _user
    )
        onlyReserveRole
        external
        returns (uint256)
111: function _reservePositionForUser(
        address _user
    )
        internal
        returns (uint256)
130: function getNextExpectedId()
        public
        view
        returns (uint256)
142: function mintPosition()
        external
        returns (uint256)
154: function getApprovedSoft(
        uint256 tokenId
    )
        external
        view
        returns (address)
174: function mintPositionForUser(
        address _user
    )
        external
        returns (uint256)
193: function _mintPositionForUser(
        address _user
    )
        internal
        returns (uint256)
222: function isOwner(
        uint256 _nftId,
        address _owner
    )
        external
        view
        returns (bool)
271: function walletOfOwner(
        address _owner
    )
        external
        view
        returns (uint256[] memory)
340: function tokenURI(
        uint256 _tokenId
    )
        public
        view
        override
        returns (string memory)
```
[90](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L90) | [99](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L99) | [111](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L111) | [130](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L130) | [142](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L142) | [154](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L154) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L174) | [193](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L193) | [222](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L222) | [271](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L271) | [340](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L340)

```solidity
File: contracts/PoolManager.sol

277: function _getPoleValue(
        uint256 _poolMulFactor,
        uint256 _poleBoundRate
    )
        private
        pure
        returns (uint256)
293: function _getDeltaPole(
        uint256 _maxPole,
        uint256 _minPole
    )
        private
        pure
        returns (uint256)
304: function _getStartValue(
        uint256 _maxPole,
        uint256 _minPole
    )
        private
        pure
        returns (uint256)
```
[277](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L277) | [293](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L293) | [304](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L304)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

292: function _distributeIncentives(
        uint256 _amount,
        address _poolToken,
        address _underlyingToken
    )
        internal
        returns (uint256)
330: function _gatherIncentives(
        address _poolToken,
        address _underlyingToken,
        address _incentiveOwner,
        uint256 _amount
    )
        internal
        returns (uint256)
```
[292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L292) | [330](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L330)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

57: function _getPositionBorrowShares(
        uint256 _nftId
    )
        internal
        view
        returns (uint256)
75: function _getPositionBorrowSharesAave(
        uint256 _nftId
    )
        internal
        view
        returns (uint256)
136: function _getPositionLendingShares(
        uint256 _nftId
    )
        internal
        view
        returns (uint256)
153: function _getPostionCollateralTokenAmount(
        uint256 _nftId
    )
        internal
        view
        returns (uint256)
174: function getPositionBorrowETH(
        uint256 _nftId
    )
        public
        view
        returns (uint256)
229: function getTotalWeightedCollateralETH(
        uint256 _nftId
    )
        public
        view
        returns (uint256)
244: function _getTokensInETH(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        internal
        view
        returns (uint256)
258: function _getEthInTokens(
        address _tokenAddress,
        uint256 _ethAmount
    )
        internal
        view
        returns (uint256)
272: function getLeverageAmount(
        uint256 _initialAmount,
        uint256 _leverage
    )
        public
        pure
        returns (uint256)
289: function _getApproxNetAPY(
        uint256 _initialAmount,
        uint256 _leverage,
        uint256 _wstETHAPY
    )
        internal
        view
        returns (
            uint256,
            bool
        )
344: function _getNewBorrowRate(
        uint256 _borrowAmount
    )
        internal
        view
        returns (uint256)
389: function _checkDebtRatio(
        uint256 _nftId
    )
        internal
        view
        returns (bool)
416: function _notBelowMinDepositAmount(
        uint256 _amount
    )
        internal
        view
        returns (bool)
```
[57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L57) | [75](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L75) | [136](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L136) | [153](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L153) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L174) | [229](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L229) | [244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L244) | [258](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L258) | [272](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L272) | [289](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L289) | [344](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L344) | [389](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L389) | [416](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L416)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

96: function enterFarm(
        bool _isAave,
        uint256 _amount,
        uint256 _leverage,
        uint256 _allowedSpread
    )
        external
        isActive
        updatePools
        returns (uint256)
142: function enterFarmETH(
        bool _isAave,
        uint256 _leverage,
        uint256 _allowedSpread
    )
        external
        payable
        isActive
        updatePools
        returns (uint256)
185: function _getWiseLendingNFT()
        internal
        returns (uint256)
```
[96](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L96) | [142](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L142) | [185](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L185)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

9: function _findIndex(
        address[] memory _array,
        address _value
    )
        internal
        pure
        returns (uint256)
32: function _calcExpiry(
        uint128 _weeks
    )
        internal
        view
        returns (uint128)
45: function _getExpiry()
        internal
        view
        returns (uint256)
53: function _getLockAmount()
        internal
        view
        returns (uint256)
61: function _getPositionData(
        address _user
    )
        private
        view
        returns (LockedPosition memory)
73: function _getAmountToSend(
        address _tokenAddress,
        uint256 _rewardValue
    )
        internal
        view
        returns (uint256)
95: function pendleChildCompoundInfoReservedForCompound(
        address _pendleMarket
    )
        external
        view
        returns (uint256[] memory)
105: function pendleChildCompoundInfoLastIndex(
        address _pendleMarket
    )
        external
        view
        returns (uint128[] memory)
115: function pendleChildCompoundInfoRewardTokens(
        address _pendleMarket
    )
        external
        view
        returns (address[] memory)
125: function activePendleMarketsLength()
        external
        view
        returns (uint256)
144: function _getRewardTokens(
        address _pendleMarket
    )
        internal
        view
        returns (address[] memory)
156: function _getUserReward(
        address _pendleMarket,
        address _rewardToken,
        address _user
    )
        internal
        view
        returns (UserReward memory)
173: function _getUserRewardIndex(
        address _pendleMarket,
        address _rewardToken,
        address _user
    )
        internal
        view
        returns (uint128)
189: function _getTokensInETH(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        internal
        view
        returns (uint256)
203: function _getTokensFromETH(
        address _tokenAddress,
        uint256 _ethValue
    )
        internal
        view
        returns (uint256)
217: function _compareHashes(
        address _pendleMarket,
        address[] memory rewardTokensToCompare
    )
        internal
        view
        returns (bool)
245: function getExpiry()
        external
        view
        returns (uint256)
253: function getLockAmount()
        external
        view
        returns (uint256)
```
[9](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L9) | [32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L32) | [45](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L45) | [53](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L53) | [61](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L61) | [73](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L73) | [95](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L95) | [105](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L105) | [115](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L115) | [125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L125) | [144](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L144) | [156](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L156) | [173](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L173) | [189](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L189) | [203](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L203) | [217](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L217) | [245](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L245) | [253](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L253)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

15: function getApproxNetAPY(
        uint256 _initialAmount,
        uint256 _leverage,
        uint256 _pendleChildApy
    )
        external
        view
        returns (
            uint256,
            bool
        )
41: function getNewBorrowRate(
        uint256 _borrowAmount
    )
        external
        view
        returns (uint256)
57: function getLiveDebtRatio(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L15) | [41](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L41) | [57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L57)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

93: function checkOwner(
        uint256 _keyId,
        address _ownerAddress
    )
        external
        view
        returns (bool)
109: function walletOfOwner(
        address _owner
    )
        external
        view
        returns (uint256[] memory)
182: function tokenURI(
        uint256 _tokenId
    )
        public
        view
        override
        returns (string memory)
```
[93](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L93) | [109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L109) | [182](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L182)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

63: function _incrementReserved()
        internal
        returns (uint256)
70: function _getNextReserveKey()
        internal
        returns (uint256)
77: function _reserveKey(
        address _userAddress,
        uint256 _wiseLendingNFT
    )
        internal
        returns (uint256)
96: function isOwner(
        uint256 _keyId,
        address _owner
    )
        public
        view
        returns (bool)
115: function _mintKeyForUser(
        uint256 _keyId,
        address _userAddress
    )
        internal
        returns (uint256)
141: function mintReserved()
        external
        returns (uint256)
160: function onERC721Received(
        address _operator,
        address _from,
        uint256 _tokenId,
        bytes calldata _data
    )
        external
        returns (bytes4)
```
[63](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L63) | [70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L70) | [77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L77) | [96](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L96) | [115](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L115) | [141](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L141) | [160](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L160)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

83: function latestAnswer()
        public
        view
        returns (uint256)
122: function _getLpToAssetRateWrapper(
        IPMarket _market,
        uint32 _duration
    )
        internal
        view
        returns (uint256)
139: function decimals()
        external
        pure
        returns (uint8)
```
[83](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L83) | [122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L122) | [139](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L139)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

81: function latestAnswer()
        public
        view
        returns (uint256)
134: function decimals()
        external
        pure
        returns (uint8)
```
[81](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L81) | [134](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L134)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

68: function latestAnswer()
        public
        view
        returns (uint256)
113: function decimals()
        external
        pure
        returns (uint8)
```
[68](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L68) | [113](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L113)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

35: function deploy(
        address _underlyingPendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
        returns (address)
```
[35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L35)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

35: function latestAnswer()
        public
        view
        returns (uint256)
47: function decimals()
        external
        pure
        returns (uint8)
```
[35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L35) | [47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L47)

```solidity
File: contracts/DerivativeOracles/CustomOracleSetup.sol

61: function getTimeStamp()
        external
        view
        returns (uint256)
```
[61](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L61)
</details>


















### [N-08] Style guide: Initialisms should be capitalized

According to the Solidity [style guide](https://docs.soliditylang.org/en/latest/style-guide.html#naming-styles) initialisms such as "NFT", "ERC", etc should be capitalized.

<details>
<summary><i>284 issue instances in 29 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

68: uint256 _nftId
78: uint256 _nftId
119: address _nftContract
130: uint256 _nftId
147: uint256 _nftId
193: uint256 _nftId
308: function _syncPoolAfterCodeExecution(
        address _poolToken,
        uint256 _lendSharePriceBefore,
        uint256 _borrowSharePriceBefore
    )
        private
330: uint256 _nftId
349: uint256 _nftId
389: uint256 _nftId
402: uint256 _nftId
461: uint256 _nftId
509: uint256 _nftId
541: uint256 _nftId
601: uint256 _nftId
637: uint256 _nftId
676: uint256 _nftId
715: uint256 _nftId
752: uint256 _nftId
781: uint256 _nftId
810: uint256 _nftId
860: uint256 _nftId
903: uint256 _nftId
940: uint256 _nftId
976: uint256 _nftId
1017: uint256 _nftId
1056: uint256 _nftId
1089: uint256 _nftId
1162: uint256 _nftId
1205: uint256 _nftId
1251: uint256 _nftId
1252: uint256 _nftIdLiquidator
1315: uint256 _nftId
1316: uint256 _nftIdLiquidator
1387: uint256 _nftId
1414: uint256 _nftId
1436: uint256 _nftId
1462: uint256 _nftId
1491: uint256 _nftId
1519: uint256 _nftId
1554: uint256 _nftId
```
[68](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L68) | [78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L78) | [119](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L119) | [130](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L130) | [147](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L147) | [193](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L193) | [308](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L308) | [330](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L330) | [349](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L349) | [389](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L389) | [402](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L402) | [461](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L461) | [509](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L509) | [541](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L541) | [601](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L601) | [637](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L637) | [676](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L676) | [715](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L715) | [752](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L752) | [781](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L781) | [810](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L810) | [860](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L860) | [903](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L903) | [940](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L940) | [976](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L976) | [1017](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1017) | [1056](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1056) | [1089](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1089) | [1162](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1162) | [1205](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1205) | [1251](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1251) | [1252](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1252) | [1315](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1315) | [1316](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1316) | [1387](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1387) | [1414](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1414) | [1436](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1436) | [1462](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1462) | [1491](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1491) | [1519](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1519) | [1554](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1554)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

60: uint256 _nftId
106: uint256 _nftIdLiquidate
238: uint256 _nftId
276: uint256 _nftId
307: uint256 _nftId
336: uint256 _nftId
361: uint256 _nftId
389: uint256 _nftId
406: uint256 _nftId
443: uint256 _nftId
500: uint256 _nftId
557: uint256 _nftId
642: uint256 _nftId
696: uint256 _nftId
765: uint256 _nftId
827: uint256 _nftId
869: uint256 _nftId
907: uint256 _nftId
944: uint256 _nftId
460: uint256 ethValue
565: uint256 ethValue
566: uint256 ethValueDebt
567: uint256 ethValueGain
```
[60](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L60) | [106](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L106) | [238](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L238) | [276](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L276) | [307](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L307) | [336](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L336) | [361](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L361) | [389](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L389) | [406](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L406) | [443](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L443) | [500](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L500) | [557](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L557) | [642](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L642) | [696](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L696) | [765](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L765) | [827](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L827) | [869](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L869) | [907](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L907) | [944](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L944) | [460](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L460) | [565](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L565) | [566](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L566) | [567](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L567)

```solidity
File: contracts/MainHelper.sol

136: uint256 _nftId
234: uint256 _nftId
235: uint256 _nftIdLiquidator
258: function _checkLiquidatorNft(
        uint256 _nftId,
        uint256 _nftIdLiquidator
    )
        internal
        view
259: uint256 _nftId
260: uint256 _nftIdLiquidator
393: uint256 _nftId
585: uint256 _nftId
600: uint256 _nftId
615: uint256 _nftId
647: function _getHash(
        uint256 _nftId,
        address _poolToken
    )
        private
        pure
        returns (bytes32)
648: uint256 _nftId
668: uint256 _nftId
671: function(uint256, uint256) view returns (address) _getPositionTokenByIndex
734: uint256 _nftId
753: uint256 _nftId
795: uint256 _nftId
815: uint256 _nftId
831: uint256 _nftId
896: function _aboveThreshold(
        address _poolToken
    )
        internal
        view
        returns (bool)
1156: uint256 _nftId
```
[136](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L136) | [234](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L234) | [235](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L235) | [258](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L258) | [259](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L259) | [260](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L260) | [393](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L393) | [585](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L585) | [600](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L600) | [615](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L615) | [647](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L647) | [648](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L648) | [668](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L668) | [671](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L671) | [734](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L734) | [753](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L753) | [795](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L795) | [815](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L815) | [831](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L831) | [896](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L896) | [1156](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1156)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

16: uint256 _nftId
66: uint256 _nftId
108: uint256 _nftId
146: uint256 _nftId
191: uint256 _nftId
196: uint256 ethCollateral
225: uint256 _nftId
247: uint256 _nftId
253: uint256 ethCollateral
282: uint256 _nftId
324: uint256 _nftId
340: function _getTokensInEth(
        address _poolToken,
        uint256 _amount
    )
        internal
        view
        returns (uint256)
361: uint256 _nftId
395: uint256 _nftId
431: uint256 _nftId
487: uint256 _nftId
611: uint256 _nftId
652: uint256 _nftId
702: uint256 _nftId
760: function calculateWishPercentage(
        uint256 _nftId,
        address _receiveToken,
        uint256 _paybackETH,
        uint256 _maxFeeETH,
        uint256 _baseRewardLiquidation
    )
        external
        view
        returns (uint256)
761: uint256 _nftId
795: uint256 _nftId
807: uint256 _nftId
838: uint256 _nftId
877: uint256 _nftId
923: uint256 _nftId
946: uint256 _nftId
967: uint256 _nftId
1030: uint256 _nftId
```
[16](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L16) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L66) | [108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L108) | [146](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L146) | [191](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L191) | [196](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L196) | [225](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L225) | [247](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L247) | [253](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L253) | [282](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L282) | [324](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L324) | [340](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L340) | [361](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L361) | [395](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L395) | [431](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L431) | [487](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L487) | [611](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L611) | [652](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L652) | [702](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L702) | [760](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L760) | [761](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L761) | [795](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L795) | [807](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L807) | [838](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L838) | [877](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L877) | [923](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L923) | [946](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L946) | [967](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L967) | [1030](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1030)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

140: function _triggerIndexUpdate()
        internal
169: function _overWriteIndexAll()
        private
177: function _overWriteIndex(
        uint256 _index
    )
        private
178: uint256 _index
453: uint256 _underlyingAssetAmount
479: uint256 _underlyingAssetAmount
226: uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
            UNDERLYING_PENDLE_MARKET
        )
234: uint128 index
274: uint256 indexDiff = index
                - lastIndex[i]
```
[140](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L140) | [169](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L169) | [177](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L177) | [178](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L178) | [453](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L453) | [479](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L479) | [226](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L226) | [234](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L234) | [274](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L274)

```solidity
File: contracts/FeeManager/FeeManager.sol

50: uint256 _percent
515: uint256 _nftId
731: uint256 _nftId
817: uint256 _nftId
884: function getPoolTokenAdressesByIndex(
        uint256 _index
    )
        external
        view
        returns (address)
885: uint256 _index
```
[50](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L50) | [515](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L515) | [731](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L731) | [817](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L817) | [884](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L884) | [885](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L885)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

16: uint256 _nftId
22: bool _ethBack
135: uint256 _nftId
141: bool _ethBack
232: function _getEthBack(
        uint256 _swapAmount,
        uint256 _minOutAmount
    )
        internal
        returns (uint256)
311: uint256 _nftId
328: uint256 _nftId
384: uint256 _ethAmount
414: function _logicOpenPosition(
        bool _isAave,
        uint256 _nftId,
        uint256 _depositAmount,
        uint256 _totalDebtBalancer,
        uint256 _allowedSpread
    )
        internal
416: uint256 _nftId
533: uint256 _nftId
561: uint256 _nftId
562: uint256 _nftIdLiquidator
157: uint256 ethValueBefore = _getTokensInETH(
            PENDLE_CHILD,
            withdrawnLpsAmount
        )
192: uint256 ethAmount = _getEthBack(
            tokenOutAmount,
            _getTokensInETH(
                block.chainid == 1
                    ? WETH_ADDRESS
                    : ENTRY_ASSET,
                tokenOutAmount
            )
                * reverseAllowedSpread
                / PRECISION_FACTOR_E18
        )
204: uint256 ethValueAfter = _getTokensInETH(
            WETH_ADDRESS,
            ethAmount
        )
            * _allowedSpread
            / PRECISION_FACTOR_E18
247: uint256 wethAmount = _getTokensUniV3(
                _swapAmount,
                _minOutAmount,
                ENTRY_ASSET,
                WETH_ADDRESS
            )
485: uint256 ethValueBefore = _getTokensInETH(
            ENTRY_ASSET,
            _depositAmount
        )
497: uint256 ethValueAfter = _getTokensInETH(
            PENDLE_CHILD,
            receivedShares
        )
            * _allowedSpread
            / PRECISION_FACTOR_E18
```
[16](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L16) | [22](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L22) | [135](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L135) | [141](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L141) | [232](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L232) | [311](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L311) | [328](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L328) | [384](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L384) | [414](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L414) | [416](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L416) | [533](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L533) | [561](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L561) | [562](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L562) | [157](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L157) | [192](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L192) | [204](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L204) | [247](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L247) | [485](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L485) | [497](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L497)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

15: address _voterContract
526: function overWriteIndex(
        address _pendleMarket,
        uint256 _tokenIndex
    )
        public
        onlyChildContract(_pendleMarket)
528: uint256 _tokenIndex
544: function overWriteIndexAll(
        address _pendleMarket
    )
        external
        onlyChildContract(_pendleMarket)
66: uint256 index = _findIndex(
            childInfo.rewardTokens,
            _rewardToken
        )
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L15) | [526](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L526) | [528](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L528) | [544](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L544) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L66)

```solidity
File: contracts/WiseCore.sol

16: uint256 _nftId
46: uint256 _nftId
108: uint256 _nftId
174: uint256 _nftId
194: uint256 _nftId
216: uint256 _nftId
237: uint256 _nftId
272: uint256 _nftId
317: uint256 _nftId
351: uint256 _nftId
429: uint256 _nftId
431: uint256 _percentLiquidation
461: uint256 _nftId
462: uint256 _nftIdLiquidator
464: uint256 _percentWishCollateral
544: uint256 _nftId
545: uint256 _nftIdLiquidator
547: uint256 _removePercentage
607: uint256 _removePercentage
635: uint256 collateralPercentage = WISE_SECURITY.calculateWishPercentage(
            _data.nftId,
            _data.tokenToRecieve,
            WISE_ORACLE.getTokensInETH(
                _data.tokenToPayback,
                _data.paybackAmount
            ),
            _data.maxFeeETH,
            _data.baseRewardLiquidation
        )
```
[16](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L16) | [46](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L46) | [108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L108) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L174) | [194](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L194) | [216](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L216) | [237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L237) | [272](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L272) | [317](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L317) | [351](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L351) | [429](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L429) | [431](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L431) | [461](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L461) | [462](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L462) | [464](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L464) | [544](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L544) | [545](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L545) | [547](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L547) | [607](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L607) | [635](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L635)

```solidity
File: contracts/WiseLendingDeclaration.sol

108: address _nftContract
```
[108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L108)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

34: address _wethAddrss
35: address _ethPricingFeed
329: uint256 _ethAmount
```
[34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L34) | [35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L35) | [329](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L329)

```solidity
File: contracts/WrapperHub/AaveHub.sol

45: address _underlyingAsset
65: address[] calldata _underlyingAssets
104: address _underlyingAsset
123: uint256 _nftId
124: address _underlyingAsset
173: uint256 _nftId
203: uint256 _nftId
204: address _underlyingAsset
239: uint256 _nftId
282: uint256 _nftId
283: address _underlyingAsset
315: uint256 _nftId
358: uint256 _nftId
359: address _underlyingAsset
394: uint256 _nftId
434: uint256 _nftId
435: address _underlyingAsset
483: uint256 _nftId
557: uint256 _nftId
558: address _underlyingAsset
612: address _underlyingAsset
636: address _underlyingAsset
665: address _underlyingAsset
```
[45](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L45) | [65](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L65) | [104](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L104) | [123](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L123) | [124](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L124) | [173](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L173) | [203](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L203) | [204](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L204) | [239](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L239) | [282](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L282) | [283](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L283) | [315](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L315) | [358](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L358) | [359](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L359) | [394](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L394) | [434](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L434) | [435](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L435) | [483](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L483) | [557](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L557) | [558](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L558) | [612](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L612) | [636](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L636) | [665](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L665)

```solidity
File: contracts/WiseLowLevelHelper.sol

100: uint256 _nftId
111: uint256 _nftId
122: uint256 _nftId
144: function getPositionLendingTokenByIndex(
        uint256 _nftId,
        uint256 _index
    )
        public
        view
        returns (address)
145: uint256 _nftId
146: uint256 _index
156: uint256 _nftId
165: function getPositionBorrowTokenByIndex(
        uint256 _nftId,
        uint256 _index
    )
        public
        view
        returns (address)
166: uint256 _nftId
167: uint256 _index
177: uint256 _nftId
373: uint256 _nftId
384: uint256 _nftId
435: uint256 _nftId
```
[100](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L100) | [111](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L111) | [122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L122) | [144](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L144) | [145](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L145) | [146](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L146) | [156](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L156) | [165](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L165) | [166](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L166) | [167](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L167) | [177](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L177) | [373](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L373) | [384](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L384) | [435](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L435)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

52: uint256 public minDepositEthAmount
167: address _dexAddress
```
[52](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L52) | [167](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L167)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

82: address _voterContract
```
[82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L82)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

57: uint256 _nftId
58: address _underlyingAsset
84: uint256 _nftId
85: address _underlyingAsset
86: address _underlyingAssetRecipient
114: uint256 _nftId
115: address _underlyingAsset
116: address _underlyingAssetRecipient
142: uint256 _nftId
143: address _underlyingAsset
144: address _underlyingAssetRecipient
166: address _underlyingAsset
220: uint256 _ethSent
244: uint256 _nftId
280: uint256 _nftId
316: address _underlyingAsset
```
[57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L57) | [58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L58) | [84](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L84) | [85](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L85) | [86](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L86) | [114](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L114) | [115](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L115) | [116](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L116) | [142](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L142) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L143) | [144](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L144) | [166](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L166) | [220](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L220) | [244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L244) | [280](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L280) | [316](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L316)

```solidity
File: contracts/PositionNFTs.sol

72: address _feeManagerContract
223: uint256 _nftId
247: uint256 _nftId
199: uint256 nftId = reserved[
            _user
        ]
```
[72](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L72) | [223](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L223) | [247](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L247) | [199](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L199)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

214: uint256 public minDepositEthValue = 1
241: mapping(address => CurveSwapStructData) public curveSwapInfoData
244: mapping(address => CurveSwapStructToken) public curveSwapInfoToken
81: FeeManager feeManagerContract = new FeeManager(
            lendingMaster,
            IAaveHubWiseSecurity(AAVE_HUB).AAVE_ADDRESS(),
            _wiseLendingAddress,
            oracleHubAddress,
            address(this),
            positionNFTAddress
        )
```
[214](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L214) | [241](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L241) | [244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L244) | [81](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L81)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

16: uint256 _nftId
48: uint256 _nftId
78: uint256 _nftId
122: uint256 _nftId
135: uint256 _nftId
271: uint256 _nftId
```
[16](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L16) | [48](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L48) | [78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L78) | [122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L122) | [135](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L135) | [271](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L271)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

58: uint256 _nftId
76: uint256 _nftId
93: uint256 _nftId
112: uint256 _nftId
137: uint256 _nftId
154: uint256 _nftId
175: uint256 _nftId
230: uint256 _nftId
258: function _getEthInTokens(
        address _tokenAddress,
        uint256 _ethAmount
    )
        internal
        view
        returns (uint256)
260: uint256 _ethAmount
390: uint256 _nftId
204: uint256 tokenValueEth
217: uint256 tokenValueAaveEth = _getTokensInETH(
            AAVE_WETH_ADDRESS,
            borrowTokenAmountAave
        )
```
[58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L58) | [76](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L76) | [93](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L93) | [112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L112) | [137](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L137) | [154](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L154) | [175](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L175) | [230](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L230) | [258](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L258) | [260](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L260) | [390](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L390) | [204](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L204) | [217](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L217)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

39: address _dexAddress
213: bool _ethBack
191: uint256 nftId = POSITION_NFT.mintPosition()
```
[39](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L39) | [213](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L213) | [191](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L191)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

9: function _findIndex(
        address[] memory _array,
        address _value
    )
        internal
        pure
        returns (uint256)
105: function pendleChildCompoundInfoLastIndex(
        address _pendleMarket
    )
        external
        view
        returns (uint128[] memory)
173: function _getUserRewardIndex(
        address _pendleMarket,
        address _rewardToken,
        address _user
    )
        internal
        view
        returns (uint128)
205: uint256 _ethValue
```
[9](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L9) | [105](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L105) | [173](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L173) | [205](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L205)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

58: uint256 _nftId
94: uint256 _nftId
95: uint256 _nftIdLiquidator
120: uint256 _nftId
158: uint256 _nftId
187: uint256 _nftId
232: uint256 _nftId
234: bool _ethBack
275: uint256 _nftId
```
[58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L58) | [94](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L94) | [95](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L95) | [120](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L120) | [158](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L158) | [187](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L187) | [232](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L232) | [234](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L234) | [275](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L275)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

34: IPriceFeed _priceFeedChainLinkEth
```
[34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L34)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

46: address _wethAddress
47: address _ethPriceFeed
```
[46](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L46) | [47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L47)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

27: IPriceFeed _priceFeedChainLinkEthUsd
```
[27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L27)

```solidity
File: contracts/WrapperHub/Declarations.sol

82: uint256 _nftId
94: uint256 _nftId
```
[82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L82) | [94](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L94)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

26: IPriceFeed _priceFeedChainLinkEth
```
[26](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L26)

```solidity
File: contracts/TransferHub/WrapperHelper.sol

12: address _wethAddress
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol#L12)
</details>

### [N-09] Consider using named function arguments

When calling functions in external contracts with multiple arguments, consider using named function parameters, rather than positional ones.

<details>
<summary><i>129 issue instances in 22 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

199: WISE_SECURITY.checkHealthState(
            _nftId,
            _powerFarm
        )
336: WISE_SECURITY.checksCollateralizeDeposit(
            _nftId,
            msg.sender,
            _poolToken
        )
371: WISE_SECURITY.checkUncollateralizedDeposit(
            _nftId,
            _poolToken
        )
825: WISE_SECURITY.checksSolelyWithdraw(
            _nftId,
            _caller,
            _poolToken
        )
1300: WISE_SECURITY.checksLiquidation(
            _nftId,
            _paybackToken,
            _shareAmountToPay
        )
```
[199](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L199) | [336](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L336) | [371](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L371) | [825](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L825) | [1300](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1300)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

431: FEE_MANAGER.setBadDebtUserLiquidation(
                _nftId,
                diff
            )
466: WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
476: WISE_ORACLE.getTokensInETH(
                token,
                amount
            )
523: WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
533: WISE_ORACLE.getTokensInETH(
                token,
                amount
            )
582: WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
604: WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
662: WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
711: WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
736: WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
851: WISE_LENDING.getPureCollateralAmount(
            _nftId,
            _poolToken
        )
887: WISE_ORACLE.getTokensFromETH(
            _poolToken,
            borrowETH
        )
915: WISE_LENDING.getPositionBorrowShares(
            _nftId,
            _poolToken
        )
952: WISE_LENDING.getPositionLendingShares(
            _nftId,
            _poolToken
        )
```
[431](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L431) | [466](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L466) | [476](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L476) | [523](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L523) | [533](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L533) | [582](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L582) | [604](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L604) | [662](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L662) | [711](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L711) | [736](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L736) | [851](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L851) | [887](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L887) | [915](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L915) | [952](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L952)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

34: WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
81: WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
123: WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
162: WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
200: WISE_LENDING.getPureCollateralAmount(
                _nftId,
                _poolToken
            )
210: WISE_LENDING.getPositionLendingShares(_nftId, _poolToken)
232: WISE_LENDING.isUncollateralized(
            _nftId,
            _poolToken
        )
257: WISE_LENDING.getPureCollateralAmount(
                _nftId,
                _poolToken
            )
290: WISE_LENDING.getPositionLendingShares(
            _nftId,
            _poolToken
        )
348: WISE_ORACLE.getTokensInETH(
            _poolToken,
            _amount
        )
376: WISE_LENDING.getPositionBorrowTokenByIndex(
                    _nftId,
                    i
                )
410: WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
446: WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
503: WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
619: WISE_LENDING.getPositionBorrowShares(
            _nftId,
            _poolToken
        )
886: WISE_LENDING.getPositionBorrowShares(
            _nftId,
            _tokenToPayback
        )
933: WISE_LENDING.getPositionLendingShares(
                    _nftId,
                    _poolToken
                )
953: WISE_LENDING.paybackAmount(
            _poolToken,
            WISE_LENDING.getPositionBorrowShares(
                _nftId,
                _poolToken
            )
        )
955: WISE_LENDING.getPositionBorrowShares(
                _nftId,
                _poolToken
            )
973: POSITION_NFTS.isOwner(
            _nftId,
            _caller
        )
1046: WISE_ORACLE.getTokensFromETH(
            _poolToken,
            withdrawETH
        )
```
[34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L34) | [81](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L81) | [123](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L123) | [162](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L162) | [200](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L200) | [210](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L210) | [232](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L232) | [257](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L257) | [290](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L290) | [348](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L348) | [376](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L376) | [410](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L410) | [446](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L446) | [503](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L503) | [619](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L619) | [886](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L886) | [933](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L933) | [953](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L953) | [955](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L955) | [973](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L973) | [1046](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1046)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

182: PENDLE_CONTROLLER.overWriteIndex(
            UNDERLYING_PENDLE_MARKET,
            _index
        )
206: PENDLE_CONTROLLER.increaseReservedForCompound(
                    UNDERLYING_PENDLE_MARKET,
                    rewardsOutsideArray
                )
365: PENDLE_CONTROLLER.withdrawLp(
            UNDERLYING_PENDLE_MARKET,
            _to,
            _amount
        )
380: PENDLE_MARKET.userReward(
            _rewardToken,
            _user
        )
```
[182](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L182) | [206](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L206) | [365](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L365) | [380](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L380)

```solidity
File: contracts/FeeManager/FeeManager.sol

119: WISE_LENDING.setPoolFee(
            _poolToken,
            _newFee
        )
151: WISE_LENDING.setPoolFee(
                _poolTokens[i],
                _newFees[i]
            )
635: WISE_LENDING.getPositionLendingShares(
            FEE_MANAGER_NFT,
            _poolToken
        )
644: WISE_LENDING.withdrawExactShares(
            FEE_MANAGER_NFT,
            _poolToken,
            shares
        )
656: AAVE.withdraw(
                underlyingTokenAddress,
                tokenAmount,
                address(this)
            )
761: WISE_LENDING.paybackAmount(
            _paybackToken,
            _shares
        )
766: WISE_LENDING.corePaybackFeeManager(
            _paybackToken,
            _nftId,
            paybackAmount,
            _shares
        )
836: WISE_LENDING.paybackAmount(
            _paybackToken,
            _shares
        )
841: WISE_LENDING.corePaybackFeeManager(
            _paybackToken,
            _nftId,
            paybackAmount,
            _shares
        )
```
[119](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L119) | [151](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L151) | [635](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L635) | [644](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L644) | [656](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L656) | [761](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L761) | [766](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L766) | [836](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L836) | [841](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L841)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

37: BALANCER_VAULT.flashLoan(
            this,
            tokens,
            amount,
            abi.encode(
                _nftId,
                _initialAmount,
                _lendingShares,
                _borrowShares,
                _allowedSpread,
                msg.sender,
                _ethBack,
                _isAave
            )
        )
318: WISE_LENDING.withdrawExactShares(
                _nftId,
                PENDLE_CHILD,
                _lendingShares
            )
334: AAVE_HUB.paybackExactShares(
                _nftId,
                WETH_ADDRESS,
                _borrowShares
            )
343: WISE_LENDING.paybackExactShares(
            _nftId,
            WETH_ADDRESS,
            _borrowShares
        )
447: PENDLE_SY.previewDeposit(
                    ENTRY_ASSET,
                    _depositAmount
                )
459: PENDLE_ROUTER_STATIC.addLiquiditySingleSyStatic(
            address(PENDLE_MARKET),
            syReceived
        )
508: WISE_LENDING.depositExactAmount(
            _nftId,
            PENDLE_CHILD,
            receivedShares
        )
539: AAVE_HUB.borrowExactAmount(
                _nftId,
                WETH_ADDRESS,
                _totalDebtBalancer
            )
548: WISE_LENDING.borrowExactAmount(
            _nftId,
            WETH_ADDRESS,
            _totalDebtBalancer
        )
579: WISE_LENDING.paybackAmount(
            paybackToken,
            _shareAmountToPay
        )
596: WISE_LENDING.coreLiquidationIsolationPools(
            _nftId,
            _nftIdLiquidator,
            msg.sender,
            paybackToken,
            PENDLE_CHILD,
            paybackAmount,
            _shareAmountToPay
        )
```
[37](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L37) | [318](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L318) | [334](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L334) | [343](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L343) | [447](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L447) | [459](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L459) | [508](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L508) | [539](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L539) | [548](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L548) | [579](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L579) | [596](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L596)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

292: UNI_V3_FACTORY.getPool(
            _token0,
            _token1,
            _fee
        )
382: OracleLibrary.getQuoteAtTick(
            _getAverageTick(
                _oracle
            ),
            _getOneUnit(
                _tokenAddress
            ),
            _tokenAddress,
            WETH_ADDRESS
        )
472: OracleLibrary.getQuoteAtTick(
            _getAverageTick(
                _uniTwapPoolInfo.oracle
            ),
            _getOneUnit(
                partnerInfo.partnerTokenAddress
            ),
            partnerInfo.partnerTokenAddress,
            WETH_ADDRESS
        )
483: OracleLibrary.getQuoteAtTick(
            _getAverageTick(
                partnerInfo.partnerOracleAddress
            ),
            _getOneUnit(
                _tokenAddress
            ),
            _tokenAddress,
            partnerInfo.partnerTokenAddress
        )
```
[292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L292) | [382](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L382) | [472](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L472) | [483](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L483)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

224: PENDLE_POWER_FARM_TOKEN_FACTORY.deploy(
            _pendleMarket,
            _tokenName,
            _symbolName,
            _maxCardinality
        )
406: PENDLE_LOCK.increaseLockPosition(
            uint128(_amount),
            expiry
        )
438: ARB_REWARDS.claim(
            master,
            _accrued,
            _proof
        )
587: PENDLE_VOTE_REWARDS.claimRetail(
            address(this),
            _amount,
            _merkleProof
        )
639: PENDLE_VOTER.vote(
            _pools,
            _weights
        )
```
[224](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L224) | [406](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L406) | [438](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L438) | [587](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L587) | [639](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L639)

```solidity
File: contracts/WiseCore.sol

63: WISE_SECURITY.checksWithdraw(
            _nftId,
            _caller,
            _poolToken
        )
260: WISE_SECURITY.checkPoolWithMinDeposit(
            _poolToken,
            _amount
        )
282: POSITION_NFT.isOwner(_nftId, _caller)
368: WISE_SECURITY.checksBorrow(
            _nftId,
            _caller,
            _poolToken
        )
635: WISE_SECURITY.calculateWishPercentage(
            _data.nftId,
            _data.tokenToRecieve,
            WISE_ORACLE.getTokensInETH(
                _data.tokenToPayback,
                _data.paybackAmount
            ),
            _data.maxFeeETH,
            _data.baseRewardLiquidation
        )
638: WISE_ORACLE.getTokensInETH(
                _data.tokenToPayback,
                _data.paybackAmount
            )
```
[63](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L63) | [260](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L260) | [282](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L282) | [368](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L368) | [635](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L635) | [638](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L638)

```solidity
File: contracts/WrapperHub/AaveHub.sol

464: WISE_LENDING.paybackExactAmount(
            _nftId,
            aaveToken,
            actualAmountDeposit
        )
498: WISE_LENDING.getPositionBorrowShares(
            _nftId,
            aaveWrappedETH
        )
507: WISE_LENDING.paybackAmount(
            aaveWrappedETH,
            userBorrowShares
        )
531: WISE_LENDING.paybackExactAmount(
            _nftId,
            aaveWrappedETH,
            actualAmountDeposit
        )
578: WISE_LENDING.paybackAmount(
            aaveToken,
            _shares
        )
590: AAVE.supply(
            _underlyingAsset,
            paybackAmount,
            address(this),
            REF_CODE
        )
597: WISE_LENDING.paybackExactShares(
            _nftId,
            aaveToken,
            _shares
        )
```
[464](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L464) | [498](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L498) | [507](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L507) | [531](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L531) | [578](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L578) | [590](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L590) | [597](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L597)

```solidity
File: contracts/WiseLowLevelHelper.sol

441: WISE_SECURITY.checkOwnerPosition(
            _nftId,
            _msgSender
        )
```
[441](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L441)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

70: POSITION_NFT.isOwner(_nftId, msg.sender)
74: WISE_LENDING.depositExactAmount(
            _nftId,
            aaveTokenAddress[_underlyingAsset],
            actualDepositAmount
        )
95: WISE_LENDING.withdrawOnBehalfExactAmount(
            _nftId,
            aaveTokenAddress[_underlyingAsset],
            _withdrawAmount
        )
101: AAVE.withdraw(
            _underlyingAsset,
            _withdrawAmount,
            _underlyingAssetRecipient
        )
126: WISE_LENDING.withdrawOnBehalfExactShares(
            _nftId,
            aaveToken,
            _shareAmount
        )
132: AAVE.withdraw(
            _underlyingAsset,
            withdrawAmount,
            _underlyingAssetRecipient
        )
150: WISE_LENDING.borrowOnBehalfExactAmount(
            _nftId,
            aaveTokenAddress[_underlyingAsset],
            _borrowAmount
        )
156: AAVE.withdraw(
            _underlyingAsset,
            _borrowAmount,
            _underlyingAssetRecipient
        )
181: AAVE.supply(
            _underlyingAsset,
            _depositAmount,
            _targetAddress,
            REF_CODE
        )
256: WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
292: WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
```
[70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L70) | [74](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L74) | [95](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L95) | [101](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L101) | [126](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L126) | [132](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L132) | [150](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L150) | [156](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L156) | [181](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L181) | [256](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L256) | [292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L292)

```solidity
File: contracts/PoolManager.sol

157: WISE_SECURITY.prepareCurvePools(
            _params.poolToken,
            _settings.curveSecuritySwapsData,
            _settings.curveSecuritySwapsToken
        )
```
[157](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L157)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

27: WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
59: WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
257: ORACLE_HUB.getTokensFromETH(
            _receivingToken,
            ORACLE_HUB.getTokensInETH(
                _paybackToken,
                increasedAmount
            )
        )
259: ORACLE_HUB.getTokensInETH(
                _paybackToken,
                increasedAmount
            )
343: ORACLE_HUB.getTokensPriceInUSD(
            _poolToken,
            incentiveAmount
        )
363: ORACLE_HUB.getTokensPriceFromUSD(
            _poolToken,
            reduceUSD
        )
```
[27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L27) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L59) | [257](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L257) | [259](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L259) | [343](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L343) | [363](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L363)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

64: WISE_LENDING.getPositionBorrowShares(
            _nftId,
            WETH_ADDRESS
        )
82: WISE_LENDING.getPositionBorrowShares(
            _nftId,
            AAVE_WETH_ADDRESS
        )
104: WISE_LENDING.paybackAmount(
                WETH_ADDRESS,
                positionBorrowShares
            )
124: WISE_LENDING.paybackAmount(
                AAVE_WETH_ADDRESS,
                positionBorrowSharesAave
            )
143: WISE_LENDING.getPositionLendingShares(
            _nftId,
            PENDLE_CHILD
        )
252: ORACLE_HUB.getTokensInETH(
            _tokenAddress,
            _tokenAmount
        )
266: ORACLE_HUB.getTokensFromETH(
            _tokenAddress,
            _ethAmount
        )
```
[64](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L64) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L82) | [104](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L104) | [124](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L124) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L143) | [252](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L252) | [266](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L266)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

165: IPendleMarket(
            _pendleMarket
        ).userReward(
            _rewardToken,
            _user
        )
197: ORACLE_HUB.getTokensInETH(
            _tokenAddress,
            _tokenAmount
        )
211: ORACLE_HUB.getTokensFromETH(
            _tokenAddress,
            _ethValue
        )
```
[165](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L165) | [197](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L197) | [211](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L211)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

131: WISE_LENDING.paybackAmount(
            poolAddress,
            _paybackShares
        )
143: WISE_LENDING.paybackExactShares(
            _nftId,
            poolAddress,
            _paybackShares
        )
163: WISE_LENDING.cashoutAmount(
            PENDLE_CHILD,
            _withdrawShares
        )
168: WISE_LENDING.withdrawExactShares(
            _nftId,
            PENDLE_CHILD,
            _withdrawShares
        )
279: WISE_LENDING.setRegistrationIsolationPool(
            _nftId,
            true
        )
```
[131](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L131) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L143) | [163](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L163) | [168](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L168) | [279](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L279)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

99: ORACLE_PENDLE_PT.getOracleState(
            PENDLE_MARKET_ADDRESS,
            TWAP_DURATION
        )
130: PendleLpOracleLib.getLpToAssetRate(
            _market,
            _duration
        )
```
[99](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L99) | [130](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L130)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

104: ORACLE_PENDLE_PT.getOracleState(
            PENDLE_MARKET_ADDRESS,
            TWAP_DURATION
        )
117: ORACLE_PENDLE_PT.getPtToAssetRate(
            PENDLE_MARKET_ADDRESS,
            TWAP_DURATION
        )
```
[104](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L104) | [117](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L117)

```solidity
File: contracts/WrapperHub/Declarations.sol

87: WISE_SECURITY.checkOwnerPosition(
            _nftId,
            msg.sender
        )
99: WISE_LENDING.checkPositionLocked(
            _nftId,
            msg.sender
        )
```
[87](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L87) | [99](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L99)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

85: ORACLE_PENDLE_PT.getOracleState(
            PENDLE_MARKET_ADDRESS,
            DURATION
        )
98: ORACLE_PENDLE_PT.getPtToAssetRate(
            PENDLE_MARKET_ADDRESS,
            DURATION
        )
```
[85](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L85) | [98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L98)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

102: PendlePowerFarmToken(pendlePowerFarmTokenAddress).initialize(
            _underlyingPendleMarket,
            PENDLE_POWER_FARM_CONTROLLER,
            _tokenName,
            _symbolName,
            _maxCardinality
        )
```
[102](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L102)
</details>

### [N-10] Risk of Permanently Locked Ether

When Ether is mistakenly sent to a contract without a means of retrieval, it becomes irrevocably locked.
Incidents of accidental Ether transfers have been observed even in high-profile projects, potentially leading to significant financial setbacks.
To enhance contract resilience, it's recommended to incorporate an "Ether recovery" function to serve as a protective measure against unintended Ether lockups.

<details>
<summary><i>4 issue instances in 4 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

49: receive()
        external
        payable
    {
```
[49](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L49)

```solidity
File: contracts/WrapperHub/AaveHub.sol

83: receive()
        external
        payable
    {
```
[83](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L83)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

125: receive()
        external
        payable
    {
```
[125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L125)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

21: receive()
        external
        payable
    {
```
[21](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L21)
</details>

### [N-11] Contract/Library Names Not in `CapWords` Style (CamelCase)

Contracts and libraries should be named using the `CapWords` style for better readability and consistency with Solidity style guidelines.
Examples of good practice include: SimpleToken, SmartBank, CertificateHashRepository, Player, Congress, Owned.
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#contract-and-library-names)

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PositionNFTs.sol

10: contract PositionNFTs is ERC721Enumerable, OwnableMaster {
```
[10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L10)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

23: contract PowerFarmNFTs is ERC721Enumerable, OwnableMaster {
```
[23](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L23)
</details>

### [N-12] Redundant Implementation of Ownable

Detected custom implementation of `Ownable` pattern.
It is recommended to use standardized implementations from libraries like OpenZeppelin or Solady.
These versions have been optimized and usage-hardened, thus re-inventing this is unnecessary and could lead to potential security risks.

<details>
<summary><i>9 issue instances in 9 files:</i></summary>

```solidity
File: contracts/WiseLendingDeclaration.sol

5: import "./OwnableMaster.sol";
```
[5](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L5)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

5: import "../../OwnableMaster.sol";
```
[5](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L5)

```solidity
File: contracts/PositionNFTs.sol

7: import "./OwnableMaster.sol";
```
[7](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L7)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

17: import "../OwnableMaster.sol";
```
[17](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L17)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

13: import "../OwnableMaster.sol";
```
[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L13)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

12: import "../../OwnableMaster.sol";
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L12)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

11: import "../../OwnableMaster.sol";
```
[11](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L11)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

12: import "../OwnableMaster.sol";
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L12)

```solidity
File: contracts/WrapperHub/Declarations.sol

12: import "../OwnableMaster.sol";
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L12)
</details>

### [N-13] Events are missing sender information

Events should include the sender information when emitted in `public` or `external` functions for better traceability.

<details>
<summary><i>51 issue instances in 5 files:</i></summary>

```solidity
File: contracts/FeeManager/FeeManager.sol

/// @audit external function `setPoolFee()` emits an event without sender information
123: emit PoolFeeChanged(
            _poolToken,
            _newFee,
            block.timestamp
        );
/// @audit external function `setPoolFeeBulk()` emits an event without sender information
155: emit PoolFeeChanged(
                _poolTokens[i],
                _newFees[i],
                block.timestamp
            );
/// @audit external function `proposeIncentiveMaster()` emits an event without sender information
184: emit IncentiveMasterProposed(
            _proposedIncentiveMaster,
            block.timestamp
        );
/// @audit external function `claimOwnershipIncentiveMaster()` emits an event without sender information
203: emit ClaimedOwnershipIncentiveMaster(
            incentiveMaster,
            block.timestamp
        );
/// @audit external function `increaseIncentiveA()` emits an event without sender information
221: emit IncentiveIncreasedA(
            _value,
            block.timestamp
        );
/// @audit external function `increaseIncentiveB()` emits an event without sender information
239: emit IncentiveIncreasedB(
            _value,
            block.timestamp
        );
/// @audit external function `claimIncentivesBulk()` emits an event without sender information
275: emit ClaimedIncentivesBulk(
            block.timestamp
        );
/// @audit external function `changeIncentiveUSDA()` emits an event without sender information
341: emit IncentiveOwnerAChanged(
            _newOwner,
            block.timestamp
        );
/// @audit external function `changeIncentiveUSDB()` emits an event without sender information
378: emit IncentiveOwnerBChanged(
            _newOwner,
            block.timestamp
        );
/// @audit external function `addPoolTokenAddress()` emits an event without sender information
401: emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );
/// @audit external function `addPoolTokenAddressManual()` emits an event without sender information
427: emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );
/// @audit external function `removePoolTokenManual()` emits an event without sender information
477: emit PoolTokenRemoved(
                _poolToken,
                block.timestamp
            );
/// @audit external function `increaseTotalBadDebtLiquidation()` emits an event without sender information
503: emit BadDebtIncreasedLiquidation(
            _amount,
            block.timestamp
        );
/// @audit external function `setBadDebtUserLiquidation()` emits an event without sender information
525: emit SetBadDebtPosition(
            _nftId,
            _amount,
            block.timestamp
        );
/// @audit external function `setBeneficial()` emits an event without sender information
559: emit SetBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );
/// @audit external function `revokeBeneficial()` emits an event without sender information
592: emit RevokeBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );
/// @audit external function `claimWiseFeesBulk()` emits an event without sender information
618: emit ClaimedFeesWiseBulk(
            block.timestamp
        );
/// @audit public function `claimWiseFees()` emits an event without sender information
676: emit ClaimedFeesWise(
            underlyingTokenAddress,
            tokenAmount,
            block.timestamp
        );
/// @audit external function `claimFeesBeneficial()` emits an event without sender information
715: emit ClaimedFeesBeneficial(
            caller,
            _feeToken,
            _amount,
            block.timestamp
        );
```
[123](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L123) | [155](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L155) | [184](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L184) | [203](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L203) | [221](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L221) | [239](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L239) | [275](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L275) | [341](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L341) | [378](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L378) | [401](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L401) | [427](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L427) | [477](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L477) | [503](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L503) | [525](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L525) | [559](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L559) | [592](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L592) | [618](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L618) | [676](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L676) | [715](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L715)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

/// @audit external function `withdrawLp()` emits an event without sender information
45: emit WithdrawLp(
            _pendleMarket,
            _to,
            _amount
        );
/// @audit external function `exchangeRewardsForCompoundingWithIncentive()` emits an event without sender information
102: emit ExchangeRewardsForCompounding(
            _pendleMarket,
            _rewardToken,
            _rewardAmount,
            sendingAmount
        );
/// @audit external function `exchangeLpFeesForPendleWithIncentive()` emits an event without sender information
158: emit ExchangeLpFeesForPendle(
            _pendleMarket,
            _pendleChildShares,
            tokenAmountSend,
            withdrawnAmount
        );
/// @audit external function `addPendleMarket()` emits an event without sender information
286: emit AddPendleMarket(
            _pendleMarket,
            pendleChild
        );
/// @audit external function `updateRewardTokens()` emits an event without sender information
312: emit UpdateRewardTokens(
            _pendleMarket,
            rewardTokens
        );
/// @audit external function `changeExchangeIncentive()` emits an event without sender information
328: emit ChangeExchangeIncentive(
            _newExchangeIncentive
        );
/// @audit external function `changeMintFee()` emits an event without sender information
354: emit ChangeMintFee(
            _pendleMarket,
            _newFee
        );
/// @audit external function `lockPendle()` emits an event without sender information
420: emit LockPendle(
            _amount,
            expiry,
            newVeBalance,
            _fromInside,
            _sameExpiry,
            block.timestamp
        );
/// @audit external function `claimArb()` emits an event without sender information
443: emit ClaimArb(
            _accrued,
            _proof,
            block.timestamp
        );
/// @audit external function `withdrawLock()` emits an event without sender information
466: emit WithdrawLock(
                amount,
                block.timestamp
            );
/// @audit external function `withdrawLock()` emits an event without sender information
490: emit WithdrawLock(
            amount,
            block.timestamp
        );
/// @audit external function `increaseReservedForCompound()` emits an event without sender information
519: emit IncreaseReservedForCompound(
            _pendleMarket,
            _amounts
        );
/// @audit external function `claimVoteRewards()` emits an event without sender information
592: emit ClaimVoteRewards(
            _amount,
            _merkleProof,
            block.timestamp
        );
```
[45](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L45) | [102](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L102) | [158](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L158) | [286](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L286) | [312](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L312) | [328](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L328) | [354](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L354) | [420](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L420) | [443](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L443) | [466](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L466) | [490](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L490) | [519](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L519) | [592](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L592)

```solidity
File: contracts/WrapperHub/AaveHub.sol

/// @audit public function `depositExactAmount()` emits an event without sender information
144: emit IsDepositAave(
            _nftId,
            block.timestamp
        );
/// @audit public function `depositExactAmountETH()` emits an event without sender information
189: emit IsDepositAave(
            _nftId,
            block.timestamp
        );
/// @audit external function `withdrawExactAmount()` emits an event without sender information
225: emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );
/// @audit external function `withdrawExactAmountETH()` emits an event without sender information
268: emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );
/// @audit external function `withdrawExactShares()` emits an event without sender information
301: emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );
/// @audit external function `withdrawExactSharesETH()` emits an event without sender information
341: emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );
/// @audit external function `borrowExactAmount()` emits an event without sender information
377: emit IsBorrowAave(
            _nftId,
            block.timestamp
        );
/// @audit external function `borrowExactAmountETH()` emits an event without sender information
420: emit IsBorrowAave(
            _nftId,
            block.timestamp
        );
/// @audit external function `paybackExactAmount()` emits an event without sender information
469: emit IsPaybackAave(
            _nftId,
            block.timestamp
        );
/// @audit external function `paybackExactAmountETH()` emits an event without sender information
543: emit IsPaybackAave(
            _nftId,
            block.timestamp
        );
/// @audit external function `paybackExactShares()` emits an event without sender information
602: emit IsPaybackAave(
            _nftId,
            block.timestamp
        );
```
[144](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L144) | [189](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L189) | [225](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L225) | [268](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L268) | [301](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L301) | [341](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L341) | [377](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L377) | [420](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L420) | [469](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L469) | [543](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L543) | [602](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L602)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

/// @audit external function `changeMinDeposit()` emits an event without sender information
70: emit MinDepositChange(
            _newMinDeposit,
            block.timestamp
        );
/// @audit external function `shutDownFarm()` emits an event without sender information
89: emit FarmStatus(
            _state,
            block.timestamp
        );
/// @audit external function `enterFarm()` emits an event without sender information
130: emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            _amount,
            block.timestamp
        );
/// @audit external function `enterFarmETH()` emits an event without sender information
173: emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            msg.value,
            block.timestamp
        );
/// @audit external function `exitFarm()` emits an event without sender information
245: emit FarmExit(
            _keyId,
            wiseLendingNFT,
            _allowedSpread,
            block.timestamp
        );
/// @audit external function `manuallyPaybackShares()` emits an event without sender information
265: emit ManualPaybackShares(
            _keyId,
            farmingKeys[_keyId],
            _paybackShares,
            block.timestamp
        );
/// @audit external function `manuallyWithdrawShares()` emits an event without sender information
294: emit ManualWithdrawShares(
            _keyId,
            wiseLendingNFT,
            _withdrawShares,
            block.timestamp
        );
```
[70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L70) | [89](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L89) | [130](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L130) | [173](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L173) | [245](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L245) | [265](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L265) | [294](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L294)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

/// @audit external function `onERC721Received()` emits an event without sender information
169: emit ERC721Received(
            _operator,
            _from,
            _tokenId,
            _data
        );
```
[169](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L169)
</details>

### [N-14] Consider Using OpenZeppelin's ReentrancyGuard Instead of Custom Implementation

OpenZeppelin's ReentrancyGuard and ReentrancyGuardUpgradeable contracts have been thoroughly tested and gas-optimized. Consider using them instead of writing your own reentrancy guard logic.

Using a proven and optimized library not only ensures security but can also save gas, contributing to overall efficiency and safety.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/WrapperHub/AaveHelper.sol

8: modifier nonReentrant() {
```
[8](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L8)
</details>

### [N-15] High cyclomatic complexity

Functions with high cyclomatic complexity are harder to understand, test, and maintain.
Consider breaking down these blocks into more manageable units, by splitting things into utility functions,
by reducing nesting, and by using early returns.

[Learn More About Cyclomatic Complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity)

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

/// @audit function `checksWithdraw` has a cyclomatic complexity of 6
237: function checksWithdraw(
        uint256 _nftId,
        address _caller,
        address _poolToken
    )
        external
        view
        returns (bool specialCase)
    {
```
[237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L237)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

/// @audit function `_setLiquidationSettings` has a cyclomatic complexity of 8
117: function _setLiquidationSettings(
        uint256 _baseReward,
        uint256 _baseRewardFarm,
        uint256 _newMaxFeeETH,
        uint256 _newMaxFeeFarmETH
    )
        internal
    {
```
[117](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L117)
</details>

### [N-16] Contract should expose an `interface`

Exposing an interface in a smart contract allows for easier integration by other projects and developers.
Without an interface, there's a risk that other projects will have to develop non-standard variants to interact with your contract.
It's highly recommended to define an interface for clearer and more robust collaboration.

<details>
<summary><i>250 issue instances in 19 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

76: function _healthStateCheck(
        uint256 _nftId
    )
        private
    {
127: function _emitFundsSolelyWithdrawn(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
144: function _emitFundsSolelyDeposited(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
165: function _getSharePrice(
        address _poolToken
    )
        private
        view
        returns (
            uint256,
            uint256
        )
    {
191: function _checkHealthState(
        uint256 _nftId,
        bool _powerFarm
    )
        internal
        view
    {
210: function _compareSharePrices(
        address _poolToken,
        uint256 _lendSharePriceBefore,
        uint256 _borrowSharePriceBefore
    )
        private
        view
    {
255: function _getCurrentSharePriceMax(
        address _poolToken
    )
        private
        view
        returns (uint256)
    {
275: function _syncPoolBeforeCodeExecution(
        address _poolToken
    )
        private
        returns (
            uint256 lendSharePrice,
            uint256 borrowSharePrice
        )
    {
308: function _syncPoolAfterCodeExecution(
        address _poolToken,
        uint256 _lendSharePriceBefore,
        uint256 _borrowSharePriceBefore
    )
        private
    {
329: function collateralizeDeposit(
        uint256 _nftId,
        address _poolToken
    )
        external
        syncPool(_poolToken)
    {
348: function unCollateralizeDeposit(
        uint256 _nftId,
        address _poolToken
    )
        external
        syncPool(_poolToken)
    {
388: function depositExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        syncPool(WETH_ADDRESS)
        returns (uint256)
    {
400: function _depositExactAmountETH(
        uint256 _nftId
    )
        private
        returns (uint256)
    {
426: function depositExactAmountETHMint()
        external
        payable
        syncPool(WETH_ADDRESS)
        returns (uint256)
    {
442: function depositExactAmountMint(
        address _poolToken,
        uint256 _amount
    )
        external
        returns (uint256)
    {
460: function depositExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        public
        syncPool(_poolToken)
        returns (uint256)
    {
493: function solelyDepositETHMint()
        external
        payable
    {
508: function solelyDepositETH(
        uint256 _nftId
    )
        public
        payable
        syncPool(WETH_ADDRESS)
    {
539: function _handleSolelyDeposit(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
581: function solelyDepositMint(
        address _poolToken,
        uint256 _amount
    )
        external
    {
600: function solelyDeposit(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        public
        syncPool(_poolToken)
    {
636: function withdrawExactAmountETH(
        uint256 _nftId,
        uint256 _amount
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
        returns (uint256)
    {
675: function withdrawExactSharesETH(
        uint256 _nftId,
        uint256 _shares
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
        returns (uint256)
    {
714: function withdrawExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _withdrawAmount
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {
751: function solelyWithdrawETH(
        uint256 _nftId,
        uint256 _withdrawAmount
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
    {
780: function solelyWithdraw(
        uint256 _nftId,
        address _poolToken,
        uint256 _withdrawAmount
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
    {
808: function _coreSolelyWithdraw(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
859: function withdrawOnBehalfExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _withdrawAmount
    )
        external
        onlyAaveHub
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {
902: function withdrawExactShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {
939: function withdrawOnBehalfExactShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        external
        onlyAaveHub
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {
975: function borrowExactAmountETH(
        uint256 _nftId,
        uint256 _amount
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
        returns (uint256)
    {
1016: function borrowExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {
1055: function borrowOnBehalfExactAmount(
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
1088: function paybackExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        syncPool(WETH_ADDRESS)
        returns (uint256)
    {
1161: function paybackExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        external
        syncPool(_poolToken)
        returns (uint256)
    {
1204: function paybackExactShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        external
        syncPool(_poolToken)
        returns (uint256)
    {
1250: function liquidatePartiallyFromTokens(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _paybackToken,
        address _receiveToken,
        uint256 _shareAmountToPay
    )
        external
        syncPool(_paybackToken)
        syncPool(_receiveToken)
        returns (uint256)
    {
1314: function coreLiquidationIsolationPools(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _caller,
        address _paybackToken,
        address _receiveToken,
        uint256 _paybackAmount,
        uint256 _shareAmountToPay
    )
        external
        syncPool(_paybackToken)
        syncPool(_receiveToken)
        returns (uint256)
    {
1367: function syncManually(
        address _poolToken
    )
        external
        syncPool(_poolToken)
    {
1386: function setRegistrationIsolationPool(
        uint256 _nftId,
        bool _registerState
    )
        external
    {
1412: function corePaybackFeeManager(
        address _poolToken,
        uint256 _nftId,
        uint256 _amount,
        uint256 _shares
    )
        external
        onlyFeeManager
        syncPool(_poolToken)
    {
1434: function _handlePayback(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        uint256 _shares
    )
        private
    {
1459: function _handleWithdrawAmount(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        bool _onBehalf
    )
        private
        returns (uint256 withdrawShares)
    {
1488: function _handleSolelyWithdraw(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
1516: function _handleWithdrawShares(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _shares,
        bool _onBehalf
    )
        private
        returns (uint256)
    {
1552: function _handleBorrowExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        bool _onBehalf
    )
        private
        returns (uint256)
    {
1588: function _reservePosition()
        private
        returns (uint256)
    {
```
[76](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L76) | [127](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L127) | [144](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L144) | [165](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L165) | [191](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L191) | [210](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L210) | [255](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L255) | [275](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L275) | [308](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L308) | [329](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L329) | [348](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L348) | [388](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) | [400](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L400) | [426](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L426) | [442](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L442) | [460](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L460) | [493](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L493) | [508](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L508) | [539](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L539) | [581](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L581) | [600](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L600) | [636](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L636) | [675](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L675) | [714](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L714) | [751](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L751) | [780](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L780) | [808](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L808) | [859](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L859) | [902](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L902) | [939](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L939) | [975](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L975) | [1016](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1016) | [1055](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1055) | [1088](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1088) | [1161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1161) | [1204](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1204) | [1250](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1250) | [1314](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1314) | [1367](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1367) | [1386](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1386) | [1412](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1412) | [1434](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1434) | [1459](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1459) | [1488](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1488) | [1516](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1516) | [1552](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1552) | [1588](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1588)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

30: function _onlyWiseLending()
        private
        view
    {
59: function getLiveDebtRatio(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
85: function setLiquidationSettings(
        uint256 _baseReward,
        uint256 _baseRewardFarm,
        uint256 _newMaxFeeETH,
        uint256 _newMaxFeeFarmETH
    )
        external
        onlyMaster
    {
105: function checksLiquidation(
        uint256 _nftIdLiquidate,
        address _tokenToPayback,
        uint256 _shareAmountToPay
    )
        external
        view
    {
142: function prepareCurvePools(
        address _poolToken,
        CurveSwapStructData calldata _curveSwapStructData,
        CurveSwapStructToken calldata _curveSwapStructToken
    )
        external
        onlyWiseLending
    {
193: function curveSecurityCheck(
        address _poolToken
    )
        external
        onlyWiseLending
    {
237: function checksWithdraw(
        uint256 _nftId,
        address _caller,
        address _poolToken
    )
        external
        view
        returns (bool specialCase)
    {
275: function checksSolelyWithdraw(
        uint256 _nftId,
        address _caller,
        address _poolToken
    )
        external
        view
        returns (bool specialCase)
    {
306: function checksBorrow(
        uint256 _nftId,
        address _caller,
        address _poolToken
    )
        external
        view
        returns (bool specialCase)
    {
335: function checksCollateralizeDeposit(
        uint256 _nftId,
        address _caller,
        address _poolAddress
    )
        external
        view
    {
360: function checkUncollateralizedDeposit(
        uint256 _nftId,
        address _poolToken
    )
        external
        view
    {
387: function checkHealthState(
        uint256 _nftId,
        bool _isPowerFarm
    )
        external
        view
    {
405: function checkBadDebtLiquidation(
        uint256 _nftId
    )
        external
        onlyWiseLending
    {
442: function overallLendingAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
499: function overallBorrowAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
556: function overallNetAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256, bool)
    {
641: function safeLimitPosition(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
695: function positionBlackListToken(
        uint256 _nftId
    )
        external
        view
        returns (bool, address)
    {
764: function maximumWithdrawToken(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval,
        uint256 _solelyWithdrawAmount
    )
        external
        view
        returns (uint256)
    {
826: function maximumWithdrawTokenSolely(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval,
        uint256 _poolWithdrawAmount
    )
        external
        view
        returns (uint256)
    {
868: function maximumBorrowToken(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        external
        view
        returns (uint256 tokenAmount)
    {
906: function getExpectedPaybackAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        external
        view
        returns (uint256)
    {
943: function getExpectedLendingAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        public
        view
        returns (uint256)
    {
980: function setBlacklistToken(
        address _tokenAddress,
        bool _state
    )
        external
        onlyMaster()
    {
995: function setSecurityWorker(
        address _entitiy,
        bool _state
    )
        external
        onlyMaster
    {
1011: function securityShutdown()
        external
    {
1032: function revokeShutdown()
        external
        onlyMaster
    {
1045: function checkPoolCondition(
        address _token
    )
        external
        view
    {
1060: function checkPoolWithMinDeposit(
        address _token,
        uint256 _amount
    )
        external
        view
        returns (bool)
    {
1077: function checkMinDepositValue(
        address _token,
        uint256 _amount
    )
        external
        view
        returns (bool)
    {
1091: function _checkMinDepositValue(
        address _token,
        uint256 _amount
    )
        private
        view
        returns (bool)
    {
1110: function changeMinDepositValue(
        uint256 _newMinDepositValue
    )
        external
        onlyMaster
    {
```
[30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L30) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L59) | [85](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L85) | [105](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L105) | [142](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L142) | [193](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L193) | [237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L237) | [275](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L275) | [306](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L306) | [335](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L335) | [360](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L360) | [387](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L387) | [405](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L405) | [442](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L442) | [499](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L499) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L556) | [641](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L641) | [695](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L695) | [764](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L764) | [826](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L826) | [868](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L868) | [906](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L906) | [943](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L943) | [980](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L980) | [995](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L995) | [1011](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1011) | [1032](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1032) | [1045](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1045) | [1060](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1060) | [1077](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1077) | [1091](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1091) | [1110](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1110)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

71: function _onlyController()
        private
        view
    {
97: function _validateSharePrice(
        uint256 _sharePriceBefore
    )
        private
        view
        returns (uint256)
    {
113: function _validateSharePriceGrowth(
        uint256 _sharePriceNow
    )
        private
        view
    {
131: function _overWriteCheck()
        internal
    {
139: function _triggerIndexUpdate()
        internal
    {
148: function _wrapOverWrites(
        bool _overWritten
    )
        internal
    {
159: function _updateRewardTokens()
        private
        returns (bool)
    {
168: function _overWriteIndexAll()
        private
    {
176: function _overWriteIndex(
        uint256 _index
    )
        private
    {
187: function _overWriteAmounts()
        private
    {
195: function _updateRewards()
        private
    {
217: function _calculateRewardsClaimedOutside()
        internal
        returns (uint256[] memory)
    {
304: function _getBalanceLpBalanceController()
        private
        view
        returns (uint256)
    {
314: function _getActiveBalance()
        private
        view
        returns (uint256)
    {
324: function _getSharePrice()
        private
        view
        returns (uint256)
    {
333: function _syncSupply()
        private
    {
346: function _increaseCardinalityNext()
        internal
    {
358: function _withdrawLp(
        address _to,
        uint256 _amount
    )
        internal
    {
371: function _getUserReward(
        address _rewardToken,
        address _user
    )
        internal
        view
        returns (UserReward memory)
    {
385: function previewDistribution()
        public
        view
        returns (uint256)
    {
415: function _setLastInteraction()
        private
    {
421: function _applyMintFee(
        uint256 _amount
    )
        internal
        view
        returns (uint256)
    {
433: function totalLpAssets()
        public
        view
        returns (uint256)
    {
442: function previewUnderlyingLpAssets()
        public
        view
        returns (uint256)
    {
451: function previewMintShares(
        uint256 _underlyingAssetAmount,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view
        returns (uint256)
    {
464: function previewAmountWithdrawShares(
        uint256 _shares,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view
        returns (uint256)
    {
477: function previewBurnShares(
        uint256 _underlyingAssetAmount,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view
        returns (uint256)
    {
493: function manualSync()
        external
        syncSupply
        returns (bool)
    {
501: function addCompoundRewards(
        uint256 _amount
    )
        external
        syncSupply
    {
529: function depositExactAmount(
        uint256 _underlyingLpAssetAmount
    )
        external
        syncSupply
        returns (
            uint256,
            uint256
        )
    {
591: function changeMintFee(
        uint256 _newFee
    )
        external
        onlyController
    {
608: function withdrawExactShares(
        uint256 _shares
    )
        external
        syncSupply
        returns (uint256)
    {
646: function withdrawExactAmount(
        uint256 _underlyingLpAssetAmount
    )
        external
        syncSupply
        returns (uint256)
    {
681: function initialize(
        address _underlyingPendleMarket,
        address _pendleController,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
    {
```
[71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L71) | [97](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L97) | [113](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L113) | [131](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L131) | [139](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L139) | [148](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L148) | [159](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L159) | [168](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L168) | [176](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L176) | [187](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L187) | [195](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L195) | [217](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L217) | [304](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L304) | [314](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L314) | [324](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L324) | [333](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L333) | [346](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L346) | [358](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L358) | [371](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L371) | [385](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L385) | [415](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L415) | [421](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L421) | [433](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L433) | [442](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L442) | [451](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L451) | [464](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L464) | [477](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L477) | [493](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L493) | [501](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L501) | [529](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L529) | [591](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L591) | [608](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L608) | [646](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L646) | [681](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L681)

```solidity
File: contracts/FeeManager/FeeManager.sol

49: function setRepayBadDebtIncentive(
        uint256 _percent
    )
        external
        onlyMaster
    {
66: function setAaveFlag(
        address _poolToken,
        address _underlyingToken
    )
        external
        onlyMaster
    {
82: function setAaveFlagBulk(
        address[] calldata _poolTokens,
        address[] calldata _underlyingTokens
    )
        external
        onlyMaster
    {
108: function setPoolFee(
        address _poolToken,
        uint256 _newFee
    )
        external
        onlyMaster
    {
135: function setPoolFeeBulk(
        address[] calldata _poolTokens,
        uint256[] calldata _newFees
    )
        external
        onlyMaster
    {
173: function proposeIncentiveMaster(
        address _proposedIncentiveMaster
    )
        external
        onlyIncentiveMaster
    {
194: function claimOwnershipIncentiveMaster()
        external
    {
214: function increaseIncentiveA(
        uint256 _value
    )
        external
        onlyIncentiveMaster
    {
232: function increaseIncentiveB(
        uint256 _value
    )
        external
        onlyIncentiveMaster
    {
249: function claimIncentivesBulk()
        external
    {
284: function claimIncentives(
        address _feeToken
    )
        public
    {
315: function changeIncentiveUSDA(
        address _newOwner
    )
        external
    {
352: function changeIncentiveUSDB(
        address _newOwner
    )
        external
    {
390: function addPoolTokenAddress(
        address _poolToken
    )
        external
        onlyWiseLending
    {
412: function addPoolTokenAddressManual(
        address _poolToken
    )
        external
        onlyMaster
    {
438: function removePoolTokenManual(
        address _poolToken
    )
        external
        onlyMaster
    {
494: function increaseTotalBadDebtLiquidation(
        uint256 _amount
    )
        external
        onlyWiseSecurity
    {
514: function setBadDebtUserLiquidation(
        uint256 _nftId,
        uint256 _amount
    )
        external
        onlyWiseSecurity
    {
538: function setBeneficial(
        address _user,
        address[] calldata _feeTokens
    )
        external
        onlyMaster
    {
571: function revokeBeneficial(
        address _user,
        address[] memory _feeTokens
    )
        external
        onlyMaster
    {
603: function claimWiseFeesBulk()
        external
    {
628: function claimWiseFees(
        address _poolToken
    )
        public
    {
689: function claimFeesBeneficial(
        address _feeToken,
        uint256 _amount
    )
        external
    {
730: function paybackBadDebtForToken(
        uint256 _nftId,
        address _paybackToken,
        address _receivingToken,
        uint256 _shares
    )
        external
        returns (
            uint256 paybackAmount,
            uint256 receivingAmount
        )
    {
816: function paybackBadDebtNoReward(
        uint256 _nftId,
        address _paybackToken,
        uint256 _shares
    )
        external
        returns (uint256 paybackAmount)
    {
872: function getPoolTokenAddressesLength()
        public
        view
        returns (uint256)
    {
884: function getPoolTokenAdressesByIndex(
        uint256 _index
    )
        external
        view
        returns (address)
    {
898: function syncAllPools()
        external
    {
```
[49](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L49) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L66) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L82) | [108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L108) | [135](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L135) | [173](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L173) | [194](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L194) | [214](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L214) | [232](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L232) | [249](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L249) | [284](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L284) | [315](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L315) | [352](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L352) | [390](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L390) | [412](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L412) | [438](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L438) | [494](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L494) | [514](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L514) | [538](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L538) | [571](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L571) | [603](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L603) | [628](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L628) | [689](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L689) | [730](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L730) | [816](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L816) | [872](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L872) | [884](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L884) | [898](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L898)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

31: function withdrawLp(
        address _pendleMarket,
        address _to,
        uint256 _amount
    )
        external
        onlyChildContract(_pendleMarket)
    {
52: function exchangeRewardsForCompoundingWithIncentive(
        address _pendleMarket,
        address _rewardToken,
        uint256 _rewardAmount
    )
        external
        syncSupply(_pendleMarket)
        returns (uint256)
    {
112: function exchangeLpFeesForPendleWithIncentive(
        address _pendleMarket,
        uint256 _pendleChildShares
    )
        external
        syncSupply(_pendleMarket)
        returns (
            uint256,
            uint256
        )
    {
171: function skim(
        address _pendleMarket
    )
        external
        returns (uint256)
    {
210: function addPendleMarket(
        address _pendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
        onlyMaster
    {
292: function updateRewardTokens(
        address _pendleMarket
    )
        external
        onlyChildContract(_pendleMarket)
        returns (bool)
    {
320: function changeExchangeIncentive(
        uint256 _newExchangeIncentive
    )
        external
        onlyMaster
    {
333: function changeMintFee(
        address _pendleMarket,
        uint256 _newFee
    )
        external
        onlyMaster
    {
364: function lockPendle(
        uint256 _amount,
        uint128 _weeks,
        bool _fromInside,
        bool _sameExpiry
    )
        external
        onlyMaster
        returns (uint256 newVeBalance)
    {
430: function claimArb(
        uint256 _accrued,
        bytes32[] calldata _proof
    )
        external
        onlyArbitrum
    {
450: function withdrawLock()
        external
        onlyMaster
        returns (uint256 amount)
    {
496: function increaseReservedForCompound(
        address _pendleMarket,
        uint256[] calldata _amounts
    )
        external
        onlyChildContract(_pendleMarket)
    {
525: function overWriteIndex(
        address _pendleMarket,
        uint256 _tokenIndex
    )
        public
        onlyChildContract(_pendleMarket)
    {
543: function overWriteIndexAll(
        address _pendleMarket
    )
        external
        onlyChildContract(_pendleMarket)
    {
565: function overWriteAmounts(
        address _pendleMarket
    )
        external
        onlyChildContract(_pendleMarket)
    {
580: function claimVoteRewards(
        uint256 _amount,
        bytes32[] calldata _merkleProof
    )
        external
    {
599: function forwardETH(
        address _to,
        uint256 _amount
    )
        external
        onlyMaster
    {
611: function vote(
        address[] calldata _pools,
        uint64[] calldata _weights
    )
        external
        onlyMaster
    {
```
[31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L31) | [52](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L52) | [112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L112) | [171](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L171) | [210](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L210) | [292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L292) | [320](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L320) | [333](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L333) | [364](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L364) | [430](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L430) | [450](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L450) | [496](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L496) | [525](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L525) | [543](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L543) | [565](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L565) | [580](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L580) | [599](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L599) | [611](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L611)

```solidity
File: contracts/WiseLendingDeclaration.sol

139: function setSecurity(
        address _wiseSecurity
    )
        external
        onlyMaster
    {
256: function _onlyAaveHub()
        private
        view
    {
```
[139](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L139) | [256](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L256)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

69: function latestResolver(
        address _tokenAddress
    )
        public
        view
        returns (uint256)
    {
84: function getTokenDecimals(
        address _tokenAddress
    )
        external
        view
        returns (uint8)
    {
96: function getTokensInUSD(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        external
        view
        returns (uint256)
    {
123: function getTokensPriceInUSD(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        external
        view
        returns (uint256)
    {
143: function getTokensInETH(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        public
        view
        returns (uint256)
    {
171: function getTokensFromUSD(
        address _tokenAddress,
        uint256 _usdValue
    )
        external
        view
        returns (uint256)
    {
198: function getTokensPriceFromUSD(
        address _tokenAddress,
        uint256 _usdValue
    )
        external
        view
        returns (uint256)
    {
218: function addTwapOracle(
        address _tokenAddress,
        address _uniPoolAddress,
        address _token0,
        address _token1,
        uint24 _fee
    )
        external
        onlyMaster
    {
266: function addTwapOracleDerivative(
        address _tokenAddress,
        address _partnerTokenAddress,
        address[2] calldata _uniPools,
        address[2] calldata _token0Array,
        address[2] calldata _token1Array,
        uint24[2] calldata _feeArray
    )
        external
        onlyMaster
    {
327: function getTokensFromETH(
        address _tokenAddress,
        uint256 _ethAmount
    )
        public
        view
        returns (uint256)
    {
359: function addOracle(
        address _tokenAddress,
        IPriceFeed _priceFeedAddress,
        address[] calldata _underlyingFeedTokens
    )
        external
        onlyMaster
    {
379: function addOracleBulk(
        address[] calldata _tokenAddresses,
        IPriceFeed[] calldata _priceFeedAddresses,
        address[][] calldata _underlyingFeedTokens
    )
        external
        onlyMaster
    {
402: function addAggregator(
        address _tokenAddress
    )
        external
        onlyMaster
    {
419: function recalibratePreview(
        address _tokenAddress
    )
        external
        view
        returns (uint256)
    {
439: function chainLinkIsDead(
        address _tokenAddress
    )
        public
        view
        returns (bool state)
    {
487: function recalibrate(
        address _tokenAddress
    )
        external
    {
501: function recalibrateBulk(
        address[] calldata _tokenAddresses
    )
        external
    {
```
[69](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L69) | [84](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L84) | [96](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L96) | [123](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L123) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L143) | [171](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L171) | [198](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L198) | [218](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L218) | [266](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L266) | [327](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L327) | [359](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L359) | [379](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L379) | [402](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L402) | [419](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L419) | [439](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L439) | [487](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L487) | [501](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L501)

```solidity
File: contracts/WrapperHub/AaveHub.sol

44: function setAaveTokenAddress(
        address _underlyingAsset,
        address _aaveToken
    )
        external
        onlyMaster
    {
63: function setAaveTokenAddressBulk(
        address[] calldata _underlyingAssets,
        address[] calldata _aaveTokens
    )
        external
        onlyMaster
    {
103: function depositExactAmountMint(
        address _underlyingAsset,
        uint256 _amount
    )
        external
        returns (uint256)
    {
122: function depositExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _amount
    )
        public
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
158: function depositExactAmountETHMint()
        external
        payable
        returns (uint256)
    {
172: function depositExactAmountETH(
        uint256 _nftId
    )
        public
        payable
        nonReentrant
        returns (uint256)
    {
202: function withdrawExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _withdrawAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
238: function withdrawExactAmountETH(
        uint256 _nftId,
        uint256 _withdrawAmount
    )
        external
        nonReentrant
        returns (uint256)
    {
281: function withdrawExactShares(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _shareAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
314: function withdrawExactSharesETH(
        uint256 _nftId,
        uint256 _shareAmount
    )
        external
        nonReentrant
        returns (uint256)
    {
357: function borrowExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _borrowAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
393: function borrowExactAmountETH(
        uint256 _nftId,
        uint256 _borrowAmount
    )
        external
        nonReentrant
        returns (uint256)
    {
433: function paybackExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _paybackAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
482: function paybackExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        nonReentrant
        returns (uint256)
    {
556: function paybackExactShares(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _shares
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
610: function skimAave(
        address _underlyingAsset,
        bool _isAave
    )
        external
        validToken(_underlyingAsset)
        onlyMaster
    {
635: function getLendingRate(
        address _underlyingAsset
    )
        external
        view
        returns (uint256)
    {
663: function _setAaveTokenAddress(
        address _underlyingAsset,
        address _aaveToken
    )
        internal
    {
```
[44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L44) | [63](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L63) | [103](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L103) | [122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L122) | [158](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L158) | [172](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L172) | [202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L202) | [238](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L238) | [281](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L281) | [314](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L314) | [357](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L357) | [393](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L393) | [433](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L433) | [482](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L482) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L556) | [610](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L610) | [635](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L635) | [663](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L663)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

265: function doApprovals()
        external
    {
273: function _doApprovals(
        address _wiseLendingAddress
    )
        internal
    {
360: function _isActive()
        internal
        view
    {
```
[265](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L265) | [273](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L273) | [360](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L360)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

235: function _onlyArbitrum()
        private
        view
    {
244: function _onlyChildContract(
        address _pendleMarket
    )
        private
        view
    {
255: function _syncSupply(
        address _pendleMarket
    )
        internal
    {
271: function syncAllSupply()
        public
    {
```
[235](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L235) | [244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L244) | [255](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L255) | [271](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L271)

```solidity
File: contracts/PositionNFTs.sol

53: function assignReserveRole(
        address _reserverForOthers,
        bool _state
    )
        external
        onlyMaster
    {
63: function blockReservePublic()
        external
        onlyMaster
    {
70: function forwardFeeManagerNFT(
        address _feeManagerContract
    )
        external
        onlyMaster
    {
89: function reservePosition()
        external
        returns (uint256)
    {
98: function reservePositionForUser(
        address _user
    )
        onlyReserveRole
        external
        returns (uint256)
    {
110: function _reservePositionForUser(
        address _user
    )
        internal
        returns (uint256)
    {
129: function getNextExpectedId()
        public
        view
        returns (uint256)
    {
142: function mintPosition()
        external
        returns (uint256)
    {
154: function getApprovedSoft(
        uint256 tokenId
    )
        external
        view
        returns (address)
    {
174: function mintPositionForUser(
        address _user
    )
        external
        returns (uint256)
    {
192: function _mintPositionForUser(
        address _user
    )
        internal
        returns (uint256)
    {
221: function isOwner(
        uint256 _nftId,
        address _owner
    )
        external
        view
        returns (bool)
    {
244: function approveMint(
        address _spender,
        uint256 _nftId
    )
        external
    {
271: function walletOfOwner(
        address _owner
    )
        external
        view
        returns (uint256[] memory)
    {
319: function setBaseURI(
        string memory _newBaseURI
    )
        external
        onlyMaster
    {
327: function setBaseExtension(
        string memory _newBaseExtension
    )
        external
        onlyMaster
    {
371: function _toString(
        uint256 _tokenId
    )
        internal
        pure
        returns (string memory str)
    {
```
[53](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L53) | [63](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L63) | [70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L70) | [89](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L89) | [98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L98) | [110](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L110) | [129](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L129) | [142](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L142) | [154](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L154) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L174) | [192](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L192) | [221](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L221) | [244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L244) | [271](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L271) | [319](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L319) | [327](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L327) | [371](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L371)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

117: function _setLiquidationSettings(
        uint256 _baseReward,
        uint256 _baseRewardFarm,
        uint256 _newMaxFeeETH,
        uint256 _newMaxFeeFarmETH
    )
        internal
    {
```
[117](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L117)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

190: function _onlyIncentiveMaster()
        private
        view
    {
201: function _onlyWiseSecurity()
        private
        view
    {
212: function _onlyWiseLending()
        private
        view
    {
```
[190](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L190) | [201](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L201) | [212](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L212)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

62: function changeMinDeposit(
        uint256 _newMinDeposit
    )
        external
        onlyMaster
    {
82: function shutDownFarm(
        bool _state
    )
        external
        onlyMaster
    {
95: function enterFarm(
        bool _isAave,
        uint256 _amount,
        uint256 _leverage,
        uint256 _allowedSpread
    )
        external
        isActive
        updatePools
        returns (uint256)
    {
141: function enterFarmETH(
        bool _isAave,
        uint256 _leverage,
        uint256 _allowedSpread
    )
        external
        payable
        isActive
        updatePools
        returns (uint256)
    {
184: function _getWiseLendingNFT()
        internal
        returns (uint256)
    {
209: function exitFarm(
        uint256 _keyId,
        uint256 _allowedSpread,
        bool _ethBack
    )
        external
        updatePools
        onlyKeyOwner(_keyId)
    {
253: function manuallyPaybackShares(
        uint256 _keyId,
        uint256 _paybackShares
    )
        external
        updatePools
    {
273: function manuallyWithdrawShares(
        uint256 _keyId,
        uint256 _withdrawShares
    )
        external
        updatePools
        onlyKeyOwner(_keyId)
    {
```
[62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L62) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L82) | [95](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L95) | [141](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L141) | [184](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L184) | [209](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L209) | [253](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L253) | [273](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L273)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

50: function setFarmContract(
        address _farmContract
    )
        external
        onlyMaster
    {
66: function mintKey(
        address _keyOwner,
        uint256 _keyId
    )
        external
        onlyFarmContract
    {
82: function burnKey(
        uint256 _keyId
    )
        external
        onlyFarmContract
    {
92: function checkOwner(
        uint256 _keyId,
        address _ownerAddress
    )
        external
        view
        returns (bool)
    {
109: function walletOfOwner(
        address _owner
    )
        external
        view
        returns (uint256[] memory)
    {
161: function setBaseURI(
        string calldata _newBaseURI
    )
        external
        onlyMaster
    {
169: function setBaseExtension(
        string calldata _newBaseExtension
    )
        external
        onlyMaster
    {
213: function _toString(
        uint256 _tokenId
    )
        internal
        pure
        returns (string memory str)
    {
```
[50](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L50) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L66) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L82) | [92](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L92) | [109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L109) | [161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L161) | [169](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L169) | [213](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L213)

```solidity
File: contracts/WrapperHub/Declarations.sol

80: function _checkOwner(
        uint256 _nftId
    )
        internal
        view
    {
92: function _checkPositionLocked(
        uint256 _nftId
    )
        internal
        view
    {
104: function setWiseSecurity(
        address _securityAddress
    )
        external
        onlyMaster
    {
```
[80](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L80) | [92](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L92) | [104](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L104)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

34: function latestAnswer()
        public
        view
        returns (uint256)
    {
46: function decimals()
        external
        pure
        returns (uint8)
    {
54: function latestRoundData()
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answerdInRound
        )
    {
77: function getRoundData(
        uint80 _roundId
    )
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
```
[34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L34) | [46](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L46) | [54](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L54) | [77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L77)

```solidity
File: contracts/TransferHub/TransferHelper.sol

13: function _safeTransfer(
        address _token,
        address _to,
        uint256 _value
    )
        internal
    {
34: function _safeTransferFrom(
        address _token,
        address _from,
        address _to,
        uint256 _value
    )
        internal
    {
```
[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol#L13) | [34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol#L34)

```solidity
File: contracts/TransferHub/ApprovalHelper.sol

13: function _safeApprove(
        address _token,
        address _spender,
        uint256 _value
    )
        internal
    {
```
[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/ApprovalHelper.sol#L13)
</details>

### [N-17] Contract uses both `require()`/`revert()` as well as custom errors

The contract uses both `require()/revert()` and `custom errors` for error handling.

It's recommended to use a consistent approach for error handling in a single file.

<details>
<summary><i>21 issue instances in 21 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

23: contract WiseSecurity is WiseSecurityHelper, ApprovalHelper {
```
[23](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L23)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

24: contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L24)

```solidity
File: contracts/FeeManager/FeeManager.sol

24: contract FeeManager is FeeManagerHelper {
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L24)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

7: contract PendlePowerFarmController is PendlePowerFarmControllerHelper {
```
[7](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L7)

```solidity
File: contracts/WiseLendingDeclaration.sol

29: contract WiseLendingDeclaration is
    OwnableMaster,
    WrapperHelper,
    SendValueHelper
{
```
[29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L29)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

30: contract WiseOracleHub is OracleHelper {
```
[30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L30)

```solidity
File: contracts/WrapperHub/AaveHub.sol

24: contract AaveHub is AaveHelper, TransferHelper, ApprovalHelper {
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L24)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

35: contract PendlePowerFarmDeclarations is
    WrapperHelper,
    TransferHelper,
    ApprovalHelper,
    SendValueHelper
{
```
[35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L35)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

29: contract PendlePowerFarmControllerBase is
    OwnableMaster,
    TransferHelper,
    ApprovalHelper
{
```
[29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L29)

```solidity
File: contracts/PositionNFTs.sol

10: contract PositionNFTs is ERC721Enumerable, OwnableMaster {
```
[10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L10)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

44: contract WiseSecurityDeclarations is OwnableMaster {
```
[44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L44)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

30: contract DeclarationsFeeManager is FeeManagerEvents, OwnableMaster {
```
[30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L30)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

14: contract PendlePowerManager is OwnableMaster, PendlePowerFarm, MinterReserver {
```
[14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L14)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

9: contract MinterReserver {
```
[9](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L9)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

27: contract PendleLpOracle {
```
[27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L27)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

21: contract PtOracleDerivative {
```
[21](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L21)

```solidity
File: contracts/WrapperHub/Declarations.sol

20: contract Declarations is OwnableMaster, AaveEvents, WrapperHelper {
```
[20](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L20)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

21: contract PtOraclePure {
```
[21](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L21)

```solidity
File: contracts/OwnableMaster.sol

8: contract OwnableMaster {
```
[8](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L8)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

8: contract PendlePowerFarmTokenFactory {
```
[8](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L8)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

7: contract SendValueHelper {
```
[7](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L7)
</details>

### [N-18] Solidity Version Too Recent to be Trusted

The current Solidity version used in the code is too recent to be trusted.
Using a newer version might introduce unrecovered bugs. It is recommended to use version 0.8.21.

<details>
<summary><i>44 issue instances in 44 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L3)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L3)

```solidity
File: contracts/MainHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L3)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L3)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L3)

```solidity
File: contracts/FeeManager/FeeManager.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L3)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L3)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L3)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L3)

```solidity
File: contracts/WiseCore.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L3)

```solidity
File: contracts/WiseLendingDeclaration.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L3)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L3)

```solidity
File: contracts/WrapperHub/AaveHub.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L3)

```solidity
File: contracts/WiseLowLevelHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L3)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L3)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L3)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L3)

```solidity
File: contracts/PositionNFTs.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L3)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L3)

```solidity
File: contracts/PoolManager.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L3)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L3)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L3)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L3)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L3)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L3)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L3)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L3)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L3)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L3)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L3)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L3)

```solidity
File: contracts/WrapperHub/Declarations.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L3)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L3)

```solidity
File: contracts/OwnableMaster.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L3)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L3)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

8: pragma solidity =0.8.24;
```
[8](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L8)

```solidity
File: contracts/FeeManager/FeeManagerEvents.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerEvents.sol#L3)

```solidity
File: contracts/DerivativeOracles/CustomOracleSetup.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L3)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L3)

```solidity
File: contracts/TransferHub/WrapperHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol#L3)

```solidity
File: contracts/TransferHub/CallOptionalReturn.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L3)

```solidity
File: contracts/TransferHub/TransferHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol#L3)

```solidity
File: contracts/WrapperHub/AaveEvents.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveEvents.sol#L3)

```solidity
File: contracts/TransferHub/ApprovalHelper.sol

3: pragma solidity =0.8.24;
```
[3](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/ApprovalHelper.sol#L3)
</details>

### [N-19] Use `bytes.concat()` instead of `abi.encodePacked()`

Solidity version 0.8.4 introduces `bytes.concat()` for concatenating byte arrays, which is more efficient than using `abi.encodePacked()`.
It is recommended to use `bytes.concat()` instead of `abi.encodePacked()` and upgrade to at least Solidity version 0.8.4 if required.

<details>
<summary><i>6 issue instances in 5 files:</i></summary>

```solidity
File: contracts/MainHelper.sol

656: abi.encodePacked(
```
[656](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L656)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

48: abi.encodePacked(
```
[48](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L48)

```solidity
File: contracts/PositionNFTs.sol

360: abi.encodePacked(
```
[360](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L360)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

202: abi.encodePacked(
```
[202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L202)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

24: abi.encodePacked(
66: abi.encodePacked(
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L24) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L66)
</details>

### [N-20] Function/Constructor Argument Names Not in mixedCase

Underscore before of after function argument names is a common convention in Solidity NOT a documentation requirement.

Function arguments should use mixedCase for better readability and consistency with Solidity style guidelines. 
Examples of good practice include: initialSupply, account, recipientAddress, senderAddress, newOwner. 
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#function-argument-names)

Rule exceptions
- Allow constant variable name/symbol/decimals to be lowercase (ERC20).
- Allow `_` at the beginning of the mixedCase match for `private variables` and `unused parameters`.

<details>
<summary><i>492 issue instances in 41 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

76: function _healthStateCheck(
        uint256 _nftId
    )
        private
    {
127: function _emitFundsSolelyWithdrawn(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
144: function _emitFundsSolelyDeposited(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
165: function _getSharePrice(
        address _poolToken
    )
        private
        view
        returns (
            uint256,
            uint256
        )
    {
191: function _checkHealthState(
        uint256 _nftId,
        bool _powerFarm
    )
        internal
        view
    {
210: function _compareSharePrices(
        address _poolToken,
        uint256 _lendSharePriceBefore,
        uint256 _borrowSharePriceBefore
    )
        private
        view
    {
255: function _getCurrentSharePriceMax(
        address _poolToken
    )
        private
        view
        returns (uint256)
    {
275: function _syncPoolBeforeCodeExecution(
        address _poolToken
    )
        private
        returns (
            uint256 lendSharePrice,
            uint256 borrowSharePrice
        )
    {
308: function _syncPoolAfterCodeExecution(
        address _poolToken,
        uint256 _lendSharePriceBefore,
        uint256 _borrowSharePriceBefore
    )
        private
    {
329: function collateralizeDeposit(
        uint256 _nftId,
        address _poolToken
    )
        external
        syncPool(_poolToken)
    {
348: function unCollateralizeDeposit(
        uint256 _nftId,
        address _poolToken
    )
        external
        syncPool(_poolToken)
    {
388: function depositExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        syncPool(WETH_ADDRESS)
        returns (uint256)
    {
400: function _depositExactAmountETH(
        uint256 _nftId
    )
        private
        returns (uint256)
    {
442: function depositExactAmountMint(
        address _poolToken,
        uint256 _amount
    )
        external
        returns (uint256)
    {
460: function depositExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        public
        syncPool(_poolToken)
        returns (uint256)
    {
508: function solelyDepositETH(
        uint256 _nftId
    )
        public
        payable
        syncPool(WETH_ADDRESS)
    {
539: function _handleSolelyDeposit(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
581: function solelyDepositMint(
        address _poolToken,
        uint256 _amount
    )
        external
    {
600: function solelyDeposit(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        public
        syncPool(_poolToken)
    {
636: function withdrawExactAmountETH(
        uint256 _nftId,
        uint256 _amount
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
        returns (uint256)
    {
675: function withdrawExactSharesETH(
        uint256 _nftId,
        uint256 _shares
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
        returns (uint256)
    {
714: function withdrawExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _withdrawAmount
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {
751: function solelyWithdrawETH(
        uint256 _nftId,
        uint256 _withdrawAmount
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
    {
780: function solelyWithdraw(
        uint256 _nftId,
        address _poolToken,
        uint256 _withdrawAmount
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
    {
808: function _coreSolelyWithdraw(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
859: function withdrawOnBehalfExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _withdrawAmount
    )
        external
        onlyAaveHub
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {
902: function withdrawExactShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {
939: function withdrawOnBehalfExactShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        external
        onlyAaveHub
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {
975: function borrowExactAmountETH(
        uint256 _nftId,
        uint256 _amount
    )
        external
        syncPool(WETH_ADDRESS)
        healthStateCheck(_nftId)
        returns (uint256)
    {
1016: function borrowExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        external
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {
1055: function borrowOnBehalfExactAmount(
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
1088: function paybackExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        syncPool(WETH_ADDRESS)
        returns (uint256)
    {
1161: function paybackExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        external
        syncPool(_poolToken)
        returns (uint256)
    {
1204: function paybackExactShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        external
        syncPool(_poolToken)
        returns (uint256)
    {
1250: function liquidatePartiallyFromTokens(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _paybackToken,
        address _receiveToken,
        uint256 _shareAmountToPay
    )
        external
        syncPool(_paybackToken)
        syncPool(_receiveToken)
        returns (uint256)
    {
1314: function coreLiquidationIsolationPools(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _caller,
        address _paybackToken,
        address _receiveToken,
        uint256 _paybackAmount,
        uint256 _shareAmountToPay
    )
        external
        syncPool(_paybackToken)
        syncPool(_receiveToken)
        returns (uint256)
    {
1367: function syncManually(
        address _poolToken
    )
        external
        syncPool(_poolToken)
    {
1386: function setRegistrationIsolationPool(
        uint256 _nftId,
        bool _registerState
    )
        external
    {
1412: function corePaybackFeeManager(
        address _poolToken,
        uint256 _nftId,
        uint256 _amount,
        uint256 _shares
    )
        external
        onlyFeeManager
        syncPool(_poolToken)
    {
1434: function _handlePayback(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        uint256 _shares
    )
        private
    {
1459: function _handleWithdrawAmount(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        bool _onBehalf
    )
        private
        returns (uint256 withdrawShares)
    {
1488: function _handleSolelyWithdraw(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
1516: function _handleWithdrawShares(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _shares,
        bool _onBehalf
    )
        private
        returns (uint256)
    {
1552: function _handleBorrowExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        bool _onBehalf
    )
        private
        returns (uint256)
    {
```
[76](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L76) | [127](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L127) | [144](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L144) | [165](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L165) | [191](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L191) | [210](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L210) | [255](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L255) | [275](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L275) | [308](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L308) | [329](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L329) | [348](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L348) | [388](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) | [400](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L400) | [442](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L442) | [460](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L460) | [508](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L508) | [539](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L539) | [581](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L581) | [600](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L600) | [636](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L636) | [675](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L675) | [714](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L714) | [751](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L751) | [780](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L780) | [808](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L808) | [859](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L859) | [902](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L902) | [939](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L939) | [975](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L975) | [1016](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1016) | [1055](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1055) | [1088](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1088) | [1161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1161) | [1204](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1204) | [1250](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1250) | [1314](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1314) | [1367](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1367) | [1386](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1386) | [1412](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1412) | [1434](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1434) | [1459](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1459) | [1488](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1488) | [1516](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1516) | [1552](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1552)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

59: function getLiveDebtRatio(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
85: function setLiquidationSettings(
        uint256 _baseReward,
        uint256 _baseRewardFarm,
        uint256 _newMaxFeeETH,
        uint256 _newMaxFeeFarmETH
    )
        external
        onlyMaster
    {
105: function checksLiquidation(
        uint256 _nftIdLiquidate,
        address _tokenToPayback,
        uint256 _shareAmountToPay
    )
        external
        view
    {
142: function prepareCurvePools(
        address _poolToken,
        CurveSwapStructData calldata _curveSwapStructData,
        CurveSwapStructToken calldata _curveSwapStructToken
    )
        external
        onlyWiseLending
    {
193: function curveSecurityCheck(
        address _poolToken
    )
        external
        onlyWiseLending
    {
237: function checksWithdraw(
        uint256 _nftId,
        address _caller,
        address _poolToken
    )
        external
        view
        returns (bool specialCase)
    {
275: function checksSolelyWithdraw(
        uint256 _nftId,
        address _caller,
        address _poolToken
    )
        external
        view
        returns (bool specialCase)
    {
306: function checksBorrow(
        uint256 _nftId,
        address _caller,
        address _poolToken
    )
        external
        view
        returns (bool specialCase)
    {
335: function checksCollateralizeDeposit(
        uint256 _nftId,
        address _caller,
        address _poolAddress
    )
        external
        view
    {
360: function checkUncollateralizedDeposit(
        uint256 _nftId,
        address _poolToken
    )
        external
        view
    {
387: function checkHealthState(
        uint256 _nftId,
        bool _isPowerFarm
    )
        external
        view
    {
405: function checkBadDebtLiquidation(
        uint256 _nftId
    )
        external
        onlyWiseLending
    {
442: function overallLendingAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
499: function overallBorrowAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
556: function overallNetAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256, bool)
    {
641: function safeLimitPosition(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
695: function positionBlackListToken(
        uint256 _nftId
    )
        external
        view
        returns (bool, address)
    {
764: function maximumWithdrawToken(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval,
        uint256 _solelyWithdrawAmount
    )
        external
        view
        returns (uint256)
    {
826: function maximumWithdrawTokenSolely(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval,
        uint256 _poolWithdrawAmount
    )
        external
        view
        returns (uint256)
    {
868: function maximumBorrowToken(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        external
        view
        returns (uint256 tokenAmount)
    {
906: function getExpectedPaybackAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        external
        view
        returns (uint256)
    {
943: function getExpectedLendingAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        public
        view
        returns (uint256)
    {
980: function setBlacklistToken(
        address _tokenAddress,
        bool _state
    )
        external
        onlyMaster()
    {
995: function setSecurityWorker(
        address _entitiy,
        bool _state
    )
        external
        onlyMaster
    {
1045: function checkPoolCondition(
        address _token
    )
        external
        view
    {
1060: function checkPoolWithMinDeposit(
        address _token,
        uint256 _amount
    )
        external
        view
        returns (bool)
    {
1077: function checkMinDepositValue(
        address _token,
        uint256 _amount
    )
        external
        view
        returns (bool)
    {
1091: function _checkMinDepositValue(
        address _token,
        uint256 _amount
    )
        private
        view
        returns (bool)
    {
1110: function changeMinDepositValue(
        uint256 _newMinDepositValue
    )
        external
        onlyMaster
    {
```
[59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L59) | [85](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L85) | [105](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L105) | [142](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L142) | [193](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L193) | [237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L237) | [275](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L275) | [306](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L306) | [335](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L335) | [360](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L360) | [387](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L387) | [405](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L405) | [442](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L442) | [499](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L499) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L556) | [641](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L641) | [695](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L695) | [764](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L764) | [826](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L826) | [868](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L868) | [906](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L906) | [943](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L943) | [980](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L980) | [995](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L995) | [1045](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1045) | [1060](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1060) | [1077](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1077) | [1091](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1091) | [1110](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1110)

```solidity
File: contracts/MainHelper.sol

17: function calculateLendingShares(
        address _poolToken,
        uint256 _amount,
        bool _maxSharePrice
    )
        public
        view
        returns (uint256)
    {
32: function _calculateShares(
        uint256 _product,
        uint256 _pseudo,
        bool _maxSharePrice
    )
        private
        pure
        returns (uint256)
    {
55: function calculateBorrowShares(
        address _poolToken,
        uint256 _amount,
        bool _maxSharePrice
    )
        public
        view
        returns (uint256)
    {
79: function cashoutAmount(
        address _poolToken,
        uint256 _shares
    )
        external
        view
        returns (uint256)
    {
92: function _cashoutAmount(
        address _poolToken,
        uint256 _shares
    )
        internal
        view
        returns (uint256)
    {
114: function paybackAmount(
        address _poolToken,
        uint256 _shares
    )
        public
        view
        returns (uint256)
    {
135: function _preparationsWithdraw(
        uint256 _nftId,
        address _caller,
        address _poolToken,
        uint256 _amount
    )
        internal
        view
        returns (uint256)
    {
163: function _getValueUtilization(
        address _poolToken
    )
        private
        view
        returns (uint256)
    {
187: function _updateUtilization(
        address _poolToken
    )
        private
    {
202: function _checkCleanUp(
        uint256 _amountContract,
        uint256 _totalPool,
        uint256 _bareAmount
    )
        private
        pure
        returns (bool)
    {
217: function _onlyIsolationPool(
        address _poolAddress
    )
        internal
        view
    {
232: function _validateIsolationPoolLiquidation(
        address _caller,
        uint256 _nftId,
        uint256 _nftIdLiquidator
    )
        internal
        view
    {
257: function _checkLiquidatorNft(
        uint256 _nftId,
        uint256 _nftIdLiquidator
    )
        internal
        view
    {
277: function _getBalance(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
    {
298: function _cleanUp(
        address _poolToken
    )
        internal
    {
350: function _getAllowedDifference(
        address _poolToken
    )
        private
        view
        returns (uint256)
    {
372: function _preparePool(
        address _poolToken
    )
        internal
    {
417: function _prepareTokens(
        address _poolToken,
        address[] memory _tokens
    )
        private
    {
454: function _curveSecurityChecks(
        address[] memory _lendTokens,
        address[] memory _borrowTokens
    )
        internal
    {
473: function _whileLoopCurveSecurity(
        address[] memory _tokens
    )
        private
    {
500: function _updatePseudoTotalAmounts(
        address _poolToken
    )
        private
    {
584: function _increasePositionLendingDeposit(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        internal
    {
599: function _decreaseLendingShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        internal
    {
614: function _addPositionTokenData(
        uint256 _nftId,
        address _poolToken,
        mapping(bytes32 => bool) storage hashMap,
        mapping(uint256 => address[]) storage userTokenData
    )
        internal
    {
647: function _getHash(
        uint256 _nftId,
        address _poolToken
    )
        private
        pure
        returns (bytes32)
    {
667: function _removePositionData(
        uint256 _nftId,
        address _poolToken,
        function(uint256) view returns (uint256) _getPositionTokenLength,
        function(uint256, uint256) view returns (address) _getPositionTokenByIndex,
        function(uint256, address) internal _deleteLastPositionData,
        bool isLending
    )
        private
    {
733: function _deleteLastPositionLendingData(
        uint256 _nftId,
        address _poolToken
    )
        private
    {
752: function _corePayback(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        uint256 _shares
    )
        internal
    {
794: function _deleteLastPositionBorrowData(
        uint256 _nftId,
        address _poolToken
    )
        private
    {
814: function isUncollateralized(
        uint256 _nftId,
        address _poolToken
    )
        external
        view
        returns (bool)
    {
830: function _checkLendingDataEmpty(
        uint256 _nftId,
        address _poolToken
    )
        private
        view
        returns (bool)
    {
851: function _calculateNewBorrowRate(
        address _poolToken
    )
        internal
    {
877: function _newBorrowRate(
        address _poolToken
    )
        internal
    {
896: function _aboveThreshold(
        address _poolToken
    )
        internal
        view
        returns (bool)
    {
911: function _scalingAlgorithm(
        address _poolToken
    )
        internal
    {
947: function _newMaxPoolShares(
        address _poolToken,
        uint256 _shareValue
    )
        private
    {
969: function _saveUp(
        address _poolToken,
        uint256 _shareValue
    )
        private
    {
987: function _resonanceOutcome(
        address _poolToken,
        uint256 _shareValue
    )
        private
        view
        returns (bool)
    {
1006: function _resetResonanceFactor(
        address _poolToken,
        uint256 _shareValue
    )
        private
    {
1030: function _revertDirectionState(
        address _poolToken
    )
        private
    {
1045: function _updateResonanceFactor(
        address _poolToken,
        uint256 _shareValues
    )
        private
    {
1061: function _reversedResonanceFactor(
        address _poolToken
    )
        private
    {
1078: function _changingResonanceFactor(
        address _poolToken
    )
        private
    {
1094: function _increaseResonanceFactor(
        address _poolToken
    )
        private
    {
1125: function _decreaseResonanceFactor(
        address _poolToken
    )
        private
    {
1155: function _removeEmptyLendingData(
        uint256 _nftId,
        address _poolToken
    )
        internal
    {
1184: function _updatePoolStorage(
        address _poolToken,
        uint256 _amount,
        uint256 _shares,
        function(address, uint256) functionAmountA,
        function(address, uint256) functionAmountB,
        function(address, uint256) functionSharesA
    )
        internal
    {
```
[17](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L17) | [32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L32) | [55](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L55) | [79](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L79) | [92](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L92) | [114](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L114) | [135](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L135) | [163](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L163) | [187](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L187) | [202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L202) | [217](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L217) | [232](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L232) | [257](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L257) | [277](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L277) | [298](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L298) | [350](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L350) | [372](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L372) | [417](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L417) | [454](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L454) | [473](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L473) | [500](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L500) | [584](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L584) | [599](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L599) | [614](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L614) | [647](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L647) | [667](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L667) | [733](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L733) | [752](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L752) | [794](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L794) | [814](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L814) | [830](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L830) | [851](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L851) | [877](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L877) | [896](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L896) | [911](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L911) | [947](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L947) | [969](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L969) | [987](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L987) | [1006](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1006) | [1030](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1030) | [1045](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1045) | [1061](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1061) | [1078](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1078) | [1094](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1094) | [1125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1125) | [1155](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1155) | [1184](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1184)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

15: function overallETHCollateralsBoth(
        uint256 _nftId
    )
        public
        view
        returns (uint256, uint256)
    {
65: function overallETHCollateralsWeighted(
        uint256 _nftId
    )
        public
        view
        returns (uint256 weightedTotal)
    {
107: function overallETHCollateralsBare(
        uint256 _nftId
    )
        public
        view
        returns (uint256 amount)
    {
145: function _overallETHCollateralsWeighted(
        uint256 _nftId,
        uint256 _interval
    )
        internal
        view
        returns (uint256 weightedTotal)
    {
190: function getFullCollateralETH(
        uint256 _nftId,
        address _poolToken
    )
        public
        view
        returns (uint256 ethCollateral)
    {
224: function _isUncollateralized(
        uint256 _nftId,
        address _poolToken
    )
        internal
        view
        returns (bool)
    {
246: function _getCollateralOfTokenETHUpdated(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        internal
        view
        returns (uint256 ethCollateral)
    {
281: function getETHCollateralUpdated(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        public
        view
        returns (uint256)
    {
323: function getETHCollateral(
        uint256 _nftId,
        address _poolToken
    )
        public
        view
        returns (uint256)
    {
339: function _getTokensInEth(
        address _poolToken,
        uint256 _amount
    )
        internal
        view
        returns (uint256)
    {
360: function overallETHBorrowBare(
        uint256 _nftId
    )
        public
        view
        returns (uint256 buffer)
    {
394: function overallETHBorrowHeartbeat(
        uint256 _nftId
    )
        public
        view
        returns (uint256 buffer)
    {
430: function overallETHBorrow(
        uint256 _nftId
    )
        public
        view
        returns (uint256 buffer)
    {
470: function _checkBlacklisted(
        address _poolToken
    )
        internal
        view
        returns (bool)
    {
486: function _overallETHBorrow(
        uint256 _nftId,
        uint256 _interval
    )
        internal
        view
        returns (uint256 buffer)
    {
530: function _getUpdatedPseudoBorrow(
        address _poolToken,
        uint256 _interval
    )
        internal
        view
        returns (uint256)
    {
554: function _getUpdatedPseudoPool(
        address _poolToken,
        uint256 _interval
    )
        internal
        view
        returns (uint256)
    {
578: function _getInterest(
        address _poolToken,
        uint256 _interval
    )
        internal
        view
        returns (uint256)
    {
610: function _getETHBorrowUpdated(
        uint256 _nftId,
        address _poolToken,
        uint256 _intervall
    )
        internal
        view
        returns (uint256)
    {
651: function getETHBorrow(
        uint256 _nftId,
        address _poolToken
    )
        public
        view
        returns (uint256)
    {
672: function checkTokenAllowed(
        address _poolAddress
    )
        public
        view
    {
687: function checkHeartbeat(
        address _poolToken
    )
        public
        view
        returns (bool)
    {
701: function _checkPositionLocked(
        uint256 _nftId
    )
        internal
        view
    {
716: function checkMaxFee(
        uint256 _paybackETH,
        uint256 _feeLiquidation,
        uint256 _maxFeeETH
    )
        external
        pure
        returns (uint256)
    {
736: function _checkMaxFee(
        uint256 _paybackETH,
        uint256 _liquidationFee,
        uint256 _maxFeeETH
    )
        internal
        pure
        returns (uint256)
    {
760: function calculateWishPercentage(
        uint256 _nftId,
        address _receiveToken,
        uint256 _paybackETH,
        uint256 _maxFeeETH,
        uint256 _baseRewardLiquidation
    )
        external
        view
        returns (uint256)
    {
794: function _checkHealthState(
        uint256 _nftId,
        bool _powerFarm
    )
        internal
        view
    {
805: function _getState(
        uint256 _nftId,
        bool _powerFarm
    )
        internal
        view
        returns (bool)
    {
837: function checksRegister(
        uint256 _nftId,
        address _caller
    )
        public
        view
    {
859: function canLiquidate(
        uint256 _borrowETHTotal,
        uint256 _weightedCollateralETH
    )
        public
        pure
    {
876: function checkMaxShares(
        uint256 _nftId,
        address _tokenToPayback,
        uint256 _borrowETHTotal,
        uint256 _unweightedCollateralETH,
        uint256 _shareAmountToPay
    )
        public
        view
    {
906: function checkBadDebtThreshold(
        uint256 _borrowETHTotal,
        uint256 _unweightedCollateral
    )
        public
        pure
        returns (bool)
    {
922: function getPositionLendingAmount(
        uint256 _nftId,
        address _poolToken
    )
        public
        view
        returns (uint256)
    {
945: function getPositionBorrowAmount(
        uint256 _nftId,
        address _poolToken
    )
        public
        view
        returns (uint256)
    {
966: function checkOwnerPosition(
        uint256 _nftId,
        address _caller
    )
        public
        view
    {
985: function getBorrowRate(
        address _poolToken
    )
        public
        view
        returns (uint256)
    {
999: function getLendingRate(
        address _poolToken
    )
        public
        view
        returns (uint256)
    {
1029: function _getPossibleWithdrawAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        internal
        view
        returns (uint256)
    {
1057: function _setPoolState(
        bool _state
    )
        internal
    {
1080: function _checkPoolCondition(
        address _poolToken
    )
        internal
        view
    {
1096: function _checkSuccess(
        bool _success
    )
        internal
        pure
    {
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L15) | [65](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L65) | [107](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L107) | [145](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L145) | [190](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L190) | [224](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L224) | [246](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L246) | [281](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L281) | [323](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L323) | [339](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L339) | [360](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L360) | [394](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L394) | [430](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L430) | [470](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L470) | [486](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L486) | [530](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L530) | [554](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L554) | [578](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L578) | [610](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L610) | [651](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L651) | [672](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L672) | [687](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L687) | [701](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L701) | [716](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L716) | [736](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L736) | [760](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L760) | [794](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L794) | [805](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L805) | [837](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L837) | [859](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L859) | [876](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L876) | [906](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L906) | [922](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L922) | [945](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L945) | [966](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L966) | [985](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L985) | [999](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L999) | [1029](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1029) | [1057](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1057) | [1080](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1080) | [1096](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1096)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

97: function _validateSharePrice(
        uint256 _sharePriceBefore
    )
        private
        view
        returns (uint256)
    {
113: function _validateSharePriceGrowth(
        uint256 _sharePriceNow
    )
        private
        view
    {
148: function _wrapOverWrites(
        bool _overWritten
    )
        internal
    {
176: function _overWriteIndex(
        uint256 _index
    )
        private
    {
358: function _withdrawLp(
        address _to,
        uint256 _amount
    )
        internal
    {
371: function _getUserReward(
        address _rewardToken,
        address _user
    )
        internal
        view
        returns (UserReward memory)
    {
421: function _applyMintFee(
        uint256 _amount
    )
        internal
        view
        returns (uint256)
    {
451: function previewMintShares(
        uint256 _underlyingAssetAmount,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view
        returns (uint256)
    {
464: function previewAmountWithdrawShares(
        uint256 _shares,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view
        returns (uint256)
    {
477: function previewBurnShares(
        uint256 _underlyingAssetAmount,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view
        returns (uint256)
    {
501: function addCompoundRewards(
        uint256 _amount
    )
        external
        syncSupply
    {
529: function depositExactAmount(
        uint256 _underlyingLpAssetAmount
    )
        external
        syncSupply
        returns (
            uint256,
            uint256
        )
    {
591: function changeMintFee(
        uint256 _newFee
    )
        external
        onlyController
    {
608: function withdrawExactShares(
        uint256 _shares
    )
        external
        syncSupply
        returns (uint256)
    {
646: function withdrawExactAmount(
        uint256 _underlyingLpAssetAmount
    )
        external
        syncSupply
        returns (uint256)
    {
681: function initialize(
        address _underlyingPendleMarket,
        address _pendleController,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
    {
```
[97](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L97) | [113](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L113) | [148](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L148) | [176](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L176) | [358](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L358) | [371](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L371) | [421](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L421) | [451](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L451) | [464](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L464) | [477](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L477) | [501](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L501) | [529](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L529) | [591](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L591) | [608](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L608) | [646](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L646) | [681](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L681)

```solidity
File: contracts/FeeManager/FeeManager.sol

49: function setRepayBadDebtIncentive(
        uint256 _percent
    )
        external
        onlyMaster
    {
66: function setAaveFlag(
        address _poolToken,
        address _underlyingToken
    )
        external
        onlyMaster
    {
82: function setAaveFlagBulk(
        address[] calldata _poolTokens,
        address[] calldata _underlyingTokens
    )
        external
        onlyMaster
    {
108: function setPoolFee(
        address _poolToken,
        uint256 _newFee
    )
        external
        onlyMaster
    {
135: function setPoolFeeBulk(
        address[] calldata _poolTokens,
        uint256[] calldata _newFees
    )
        external
        onlyMaster
    {
173: function proposeIncentiveMaster(
        address _proposedIncentiveMaster
    )
        external
        onlyIncentiveMaster
    {
214: function increaseIncentiveA(
        uint256 _value
    )
        external
        onlyIncentiveMaster
    {
232: function increaseIncentiveB(
        uint256 _value
    )
        external
        onlyIncentiveMaster
    {
284: function claimIncentives(
        address _feeToken
    )
        public
    {
315: function changeIncentiveUSDA(
        address _newOwner
    )
        external
    {
352: function changeIncentiveUSDB(
        address _newOwner
    )
        external
    {
390: function addPoolTokenAddress(
        address _poolToken
    )
        external
        onlyWiseLending
    {
412: function addPoolTokenAddressManual(
        address _poolToken
    )
        external
        onlyMaster
    {
438: function removePoolTokenManual(
        address _poolToken
    )
        external
        onlyMaster
    {
494: function increaseTotalBadDebtLiquidation(
        uint256 _amount
    )
        external
        onlyWiseSecurity
    {
514: function setBadDebtUserLiquidation(
        uint256 _nftId,
        uint256 _amount
    )
        external
        onlyWiseSecurity
    {
538: function setBeneficial(
        address _user,
        address[] calldata _feeTokens
    )
        external
        onlyMaster
    {
571: function revokeBeneficial(
        address _user,
        address[] memory _feeTokens
    )
        external
        onlyMaster
    {
628: function claimWiseFees(
        address _poolToken
    )
        public
    {
689: function claimFeesBeneficial(
        address _feeToken,
        uint256 _amount
    )
        external
    {
730: function paybackBadDebtForToken(
        uint256 _nftId,
        address _paybackToken,
        address _receivingToken,
        uint256 _shares
    )
        external
        returns (
            uint256 paybackAmount,
            uint256 receivingAmount
        )
    {
816: function paybackBadDebtNoReward(
        uint256 _nftId,
        address _paybackToken,
        uint256 _shares
    )
        external
        returns (uint256 paybackAmount)
    {
884: function getPoolTokenAdressesByIndex(
        uint256 _index
    )
        external
        view
        returns (address)
    {
```
[49](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L49) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L66) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L82) | [108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L108) | [135](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L135) | [173](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L173) | [214](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L214) | [232](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L232) | [284](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L284) | [315](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L315) | [352](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L352) | [390](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L390) | [412](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L412) | [438](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L438) | [494](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L494) | [514](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L514) | [538](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L538) | [571](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L571) | [628](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L628) | [689](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L689) | [730](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L730) | [816](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L816) | [884](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L884)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

15: function _executeBalancerFlashLoan(
        uint256 _nftId,
        uint256 _flashAmount,
        uint256 _initialAmount,
        uint256 _lendingShares,
        uint256 _borrowShares,
        uint256 _allowedSpread,
        bool _ethBack,
        bool _isAave
    )
        internal
    {
59: function receiveFlashLoan(
        IERC20[] memory _flashloanToken,
        uint256[] memory _flashloanAmounts,
        uint256[] memory _feeAmounts,
        bytes memory _userData
    )
        external
    {
134: function _logicClosePosition(
        uint256 _nftId,
        uint256 _borrowShares,
        uint256 _lendingShares,
        uint256 _totalDebtBalancer,
        uint256 _allowedSpread,
        address _caller,
        bool _ethBack,
        bool _isAave
    )
        private
    {
231: function _getEthBack(
        uint256 _swapAmount,
        uint256 _minOutAmount
    )
        internal
        returns (uint256)
    {
263: function _getTokensUniV3(
        uint256 _amountIn,
        uint256 _minOutAmount,
        address _tokenIn,
        address _tokenOut
    )
        internal
        returns (uint256)
    {
293: function _swapStETHintoETH(
        uint256 _swapAmount,
        uint256 _minOutAmount
    )
        internal
        returns (uint256)
    {
309: function _withdrawPendleLPs(
        uint256 _nftId,
        uint256 _lendingShares
    )
        private
        returns (uint256 withdrawnLpsAmount)
    {
325: function _paybackExactShares(
        bool _isAave,
        uint256 _nftId,
        uint256 _borrowShares
    )
        internal
    {
354: function _closingRouteToken(
        uint256 _tokenAmount,
        uint256 _totalDebtBalancer,
        address _caller
    )
        private
    {
383: function _closingRouteETH(
        uint256 _ethAmount,
        uint256 _totalDebtBalancer,
        address _caller
    )
        internal
    {
414: function _logicOpenPosition(
        bool _isAave,
        uint256 _nftId,
        uint256 _depositAmount,
        uint256 _totalDebtBalancer,
        uint256 _allowedSpread
    )
        internal
    {
530: function _borrowExactAmount(
        bool _isAave,
        uint256 _nftId,
        uint256 _totalDebtBalancer
    )
        internal
    {
560: function _coreLiquidation(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        uint256 _shareAmountToPay
    )
        internal
        returns (
            uint256 paybackAmount,
            uint256 receivingAmount
        )
    {
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L15) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L59) | [134](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L134) | [231](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L231) | [263](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L263) | [293](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L293) | [309](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L309) | [325](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L325) | [354](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L354) | [383](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L383) | [414](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L414) | [530](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L530) | [560](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L560)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

12: function _addOracle(
        address _tokenAddress,
        IPriceFeed _priceFeedAddress,
        address[] calldata _underlyingFeedTokens
    )
        internal
    {
31: function _addAggregator(
        address _tokenAddress
    )
        internal
    {
59: function _checkFunctionExistence(
        address _tokenAggregator
    )
        internal
        returns (bool)
    {
86: function _compareMinMax(
        IAggregator _tokenAggregator,
        int192 _answer
    )
        internal
        view
    {
106: function latestResolverTwap(
        address _tokenAddress
    )
        public
        view
        returns (uint256)
    {
130: function _validateAnswer(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
    {
179: function _writeUniTwapPoolInfoStruct(
        address _tokenAddress,
        address _oracle,
        bool _isUniPool
    )
        internal
    {
195: function _writeUniTwapPoolInfoStructDerivative(
        address _tokenAddress,
        address _partnerTokenAddress,
        address _oracleAddress,
        address _partnerOracleAddress,
        bool _isUniPool
    )
        internal
    {
215: function _getRelativeDifference(
        uint256 _answerUint256,
        uint256 _fetchTwapValue
    )
        internal
        pure
        returns (uint256)
    {
234: function _compareDifference(
        uint256 _relativeDifference
    )
        internal
        view
    {
245: function _getChainlinkAnswer(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
    {
283: function _getPool(
        address _token0,
        address _token1,
        uint24 _fee
    )
        internal
        view
        returns (address pool)
    {
303: function _validateTokenAddress(
        address _tokenAddress,
        address _token0,
        address _token1
    )
        internal
        pure
    {
324: function _validatePoolAddress(
        address _pool,
        address _expectedPool
    )
        internal
        pure
    {
344: function _validatePriceFeed(
        address _tokenAddress
    )
        internal
        view
    {
359: function _validateTwapOracle(
        address _tokenAddress
    )
        internal
        view
    {
374: function _getTwapPrice(
        address _tokenAddress,
        address _oracle
    )
        internal
        view
        returns (uint256)
    {
393: function _getOneUnit(
        address _tokenAddress
    )
        internal
        view
        returns (uint128)
    {
405: function _getAverageTick(
        address _oracle
    )
        internal
        view
        returns (int24)
    {
450: function decimals(
        address _tokenAddress
    )
        public
        view
        returns (uint8)
    {
459: function _getTwapDerivatePrice(
        address _tokenAddress,
        UniTwapPoolInfo memory _uniTwapPoolInfo
    )
        internal
        view
        returns (uint256)
    {
507: function _recalibrate(
        address _tokenAddress
    )
        internal
    {
556: function _chainLinkIsDead(
        address _tokenAddress
    )
        internal
        view
        returns (bool)
    {
592: function _recalibratePreview(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
    {
658: function _getIterationCount(
        uint80 _latestAggregatorRoundId
    )
        internal
        pure
        returns (uint80 res)
    {
674: function _getRoundTimestamp(
        address _tokenAddress,
        uint80 _roundId
    )
        internal
        view
        returns (uint256)
    {
699: function getLatestRoundId(
        address _tokenAddress
    )
        public
        view
        returns (
            uint80 roundId
        )
    {
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L12) | [31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L31) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L59) | [86](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L86) | [106](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L106) | [130](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L130) | [179](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L179) | [195](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L195) | [215](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L215) | [234](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L234) | [245](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L245) | [283](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L283) | [303](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L303) | [324](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L324) | [344](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L344) | [359](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L359) | [374](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L374) | [393](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L393) | [405](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L405) | [450](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L450) | [459](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L459) | [507](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L507) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L556) | [592](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L592) | [658](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L658) | [674](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L674) | [699](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L699)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

31: function withdrawLp(
        address _pendleMarket,
        address _to,
        uint256 _amount
    )
        external
        onlyChildContract(_pendleMarket)
    {
52: function exchangeRewardsForCompoundingWithIncentive(
        address _pendleMarket,
        address _rewardToken,
        uint256 _rewardAmount
    )
        external
        syncSupply(_pendleMarket)
        returns (uint256)
    {
112: function exchangeLpFeesForPendleWithIncentive(
        address _pendleMarket,
        uint256 _pendleChildShares
    )
        external
        syncSupply(_pendleMarket)
        returns (
            uint256,
            uint256
        )
    {
171: function skim(
        address _pendleMarket
    )
        external
        returns (uint256)
    {
210: function addPendleMarket(
        address _pendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
        onlyMaster
    {
292: function updateRewardTokens(
        address _pendleMarket
    )
        external
        onlyChildContract(_pendleMarket)
        returns (bool)
    {
320: function changeExchangeIncentive(
        uint256 _newExchangeIncentive
    )
        external
        onlyMaster
    {
333: function changeMintFee(
        address _pendleMarket,
        uint256 _newFee
    )
        external
        onlyMaster
    {
364: function lockPendle(
        uint256 _amount,
        uint128 _weeks,
        bool _fromInside,
        bool _sameExpiry
    )
        external
        onlyMaster
        returns (uint256 newVeBalance)
    {
430: function claimArb(
        uint256 _accrued,
        bytes32[] calldata _proof
    )
        external
        onlyArbitrum
    {
496: function increaseReservedForCompound(
        address _pendleMarket,
        uint256[] calldata _amounts
    )
        external
        onlyChildContract(_pendleMarket)
    {
525: function overWriteIndex(
        address _pendleMarket,
        uint256 _tokenIndex
    )
        public
        onlyChildContract(_pendleMarket)
    {
543: function overWriteIndexAll(
        address _pendleMarket
    )
        external
        onlyChildContract(_pendleMarket)
    {
565: function overWriteAmounts(
        address _pendleMarket
    )
        external
        onlyChildContract(_pendleMarket)
    {
580: function claimVoteRewards(
        uint256 _amount,
        bytes32[] calldata _merkleProof
    )
        external
    {
599: function forwardETH(
        address _to,
        uint256 _amount
    )
        external
        onlyMaster
    {
611: function vote(
        address[] calldata _pools,
        uint64[] calldata _weights
    )
        external
        onlyMaster
    {
```
[31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L31) | [52](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L52) | [112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L112) | [171](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L171) | [210](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L210) | [292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L292) | [320](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L320) | [333](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L333) | [364](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L364) | [430](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L430) | [496](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L496) | [525](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L525) | [543](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L543) | [565](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L565) | [580](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L580) | [599](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L599) | [611](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L611)

```solidity
File: contracts/WiseCore.sol

15: function _prepareAssociatedTokens(
        uint256 _nftId,
        address _poolTokenLend,
        address _poolTokenBorrow
    )
        internal
        returns (
            address[] memory,
            address[] memory
        )
    {
44: function _coreWithdrawToken(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        uint256 _shares,
        bool _onBehalf
    )
        internal
    {
106: function _handleDeposit(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        internal
        returns (uint256)
    {
172: function checkPositionLocked(
        uint256 _nftId,
        address _caller
    )
        external
        view
    {
192: function _checkPositionLocked(
        uint256 _nftId,
        address _caller
    )
        internal
        view
    {
215: function checkDeposit(
        uint256 _nftId,
        address _caller,
        address _poolToken,
        uint256 _amount
    )
        external
        view
    {
236: function _checkDeposit(
        uint256 _nftId,
        address _caller,
        address _poolToken,
        uint256 _amount
    )
        internal
        view
    {
270: function _checkAllowDeposit(
        uint256 _nftId,
        address _caller
    )
        internal
        view
    {
294: function _checkMaxDepositValue(
        address _poolToken,
        uint256 _amount
    )
        private
        view
    {
316: function _coreWithdrawBare(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        uint256 _shares
    )
        private
    {
349: function _coreBorrowTokens(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        uint256 _shares,
        bool _onBehalf
    )
        internal
    {
428: function _withdrawPureCollateralLiquidation(
        uint256 _nftId,
        address _poolToken,
        uint256 _percentLiquidation
    )
        private
        returns (uint256 transferAmount)
    {
460: function _withdrawOrAllocateSharesLiquidation(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _poolToken,
        uint256 _percentWishCollateral
    )
        private
        returns (uint256)
    {
543: function _calculateReceiveAmount(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _receiveTokens,
        uint256 _removePercentage
    )
        private
        returns (uint256 receiveAmount)
    {
603: function _calculatePotentialPureExtraCashout(
        uint256 _userShares,
        address _poolToken,
        uint256 _removePercentage
    )
        private
        view
        returns (uint256)
    {
625: function _coreLiquidation(
        CoreLiquidationStruct memory _data
    )
        internal
        returns (uint256 receiveAmount)
    {
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L15) | [44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L44) | [106](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L106) | [172](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L172) | [192](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L192) | [215](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L215) | [236](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L236) | [270](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L270) | [294](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L294) | [316](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L316) | [349](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L349) | [428](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L428) | [460](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L460) | [543](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L543) | [603](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L603) | [625](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L625)

```solidity
File: contracts/WiseLendingDeclaration.sol

139: function setSecurity(
        address _wiseSecurity
    )
        external
        onlyMaster
    {
104: constructor(
        address _master,
        address _wiseOracleHub,
        address _nftContract
    )
        OwnableMaster(
            _master
        )
        WrapperHelper(
            IWiseOracleHub(
                _wiseOracleHub
            ).WETH_ADDRESS()
        )
    {
```
[139](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L139) | [104](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L104)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

69: function latestResolver(
        address _tokenAddress
    )
        public
        view
        returns (uint256)
    {
84: function getTokenDecimals(
        address _tokenAddress
    )
        external
        view
        returns (uint8)
    {
96: function getTokensInUSD(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        external
        view
        returns (uint256)
    {
123: function getTokensPriceInUSD(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        external
        view
        returns (uint256)
    {
143: function getTokensInETH(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        public
        view
        returns (uint256)
    {
171: function getTokensFromUSD(
        address _tokenAddress,
        uint256 _usdValue
    )
        external
        view
        returns (uint256)
    {
198: function getTokensPriceFromUSD(
        address _tokenAddress,
        uint256 _usdValue
    )
        external
        view
        returns (uint256)
    {
218: function addTwapOracle(
        address _tokenAddress,
        address _uniPoolAddress,
        address _token0,
        address _token1,
        uint24 _fee
    )
        external
        onlyMaster
    {
266: function addTwapOracleDerivative(
        address _tokenAddress,
        address _partnerTokenAddress,
        address[2] calldata _uniPools,
        address[2] calldata _token0Array,
        address[2] calldata _token1Array,
        uint24[2] calldata _feeArray
    )
        external
        onlyMaster
    {
327: function getTokensFromETH(
        address _tokenAddress,
        uint256 _ethAmount
    )
        public
        view
        returns (uint256)
    {
359: function addOracle(
        address _tokenAddress,
        IPriceFeed _priceFeedAddress,
        address[] calldata _underlyingFeedTokens
    )
        external
        onlyMaster
    {
379: function addOracleBulk(
        address[] calldata _tokenAddresses,
        IPriceFeed[] calldata _priceFeedAddresses,
        address[][] calldata _underlyingFeedTokens
    )
        external
        onlyMaster
    {
402: function addAggregator(
        address _tokenAddress
    )
        external
        onlyMaster
    {
419: function recalibratePreview(
        address _tokenAddress
    )
        external
        view
        returns (uint256)
    {
439: function chainLinkIsDead(
        address _tokenAddress
    )
        public
        view
        returns (bool state)
    {
487: function recalibrate(
        address _tokenAddress
    )
        external
    {
501: function recalibrateBulk(
        address[] calldata _tokenAddresses
    )
        external
    {
```
[69](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L69) | [84](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L84) | [96](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L96) | [123](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L123) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L143) | [171](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L171) | [198](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L198) | [218](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L218) | [266](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L266) | [327](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L327) | [359](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L359) | [379](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L379) | [402](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L402) | [419](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L419) | [439](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L439) | [487](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L487) | [501](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L501)

```solidity
File: contracts/WrapperHub/AaveHub.sol

44: function setAaveTokenAddress(
        address _underlyingAsset,
        address _aaveToken
    )
        external
        onlyMaster
    {
63: function setAaveTokenAddressBulk(
        address[] calldata _underlyingAssets,
        address[] calldata _aaveTokens
    )
        external
        onlyMaster
    {
103: function depositExactAmountMint(
        address _underlyingAsset,
        uint256 _amount
    )
        external
        returns (uint256)
    {
122: function depositExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _amount
    )
        public
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
172: function depositExactAmountETH(
        uint256 _nftId
    )
        public
        payable
        nonReentrant
        returns (uint256)
    {
202: function withdrawExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _withdrawAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
238: function withdrawExactAmountETH(
        uint256 _nftId,
        uint256 _withdrawAmount
    )
        external
        nonReentrant
        returns (uint256)
    {
281: function withdrawExactShares(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _shareAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
314: function withdrawExactSharesETH(
        uint256 _nftId,
        uint256 _shareAmount
    )
        external
        nonReentrant
        returns (uint256)
    {
357: function borrowExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _borrowAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
393: function borrowExactAmountETH(
        uint256 _nftId,
        uint256 _borrowAmount
    )
        external
        nonReentrant
        returns (uint256)
    {
433: function paybackExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _paybackAmount
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
482: function paybackExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        nonReentrant
        returns (uint256)
    {
556: function paybackExactShares(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _shares
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
610: function skimAave(
        address _underlyingAsset,
        bool _isAave
    )
        external
        validToken(_underlyingAsset)
        onlyMaster
    {
635: function getLendingRate(
        address _underlyingAsset
    )
        external
        view
        returns (uint256)
    {
663: function _setAaveTokenAddress(
        address _underlyingAsset,
        address _aaveToken
    )
        internal
    {
```
[44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L44) | [63](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L63) | [103](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L103) | [122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L122) | [172](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L172) | [202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L202) | [238](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L238) | [281](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L281) | [314](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L314) | [357](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L357) | [393](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L393) | [433](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L433) | [482](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L482) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L556) | [610](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L610) | [635](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L635) | [663](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L663)

```solidity
File: contracts/WiseLowLevelHelper.sol

24: function _validateParameter(
        uint256 _parameterValue,
        uint256 _parameterLimit
    )
        internal
        pure
    {
38: function getTotalPool(
        address _poolToken
    )
        public
        view
        returns (uint256)
    {
48: function getPseudoTotalPool(
        address _poolToken
    )
        public
        view
        returns (uint256)
    {
58: function getTotalBareToken(
        address _poolToken
    )
        external
        view
        returns (uint256)
    {
68: function getPseudoTotalBorrowAmount(
        address _poolToken
    )
        external
        view
        returns (uint256)
    {
78: function getTotalDepositShares(
        address _poolToken
    )
        external
        view
        returns (uint256)
    {
88: function getTotalBorrowShares(
        address _poolToken
    )
        external
        view
        returns (uint256)
    {
98: function getPositionLendingShares(
        uint256 _nftId,
        address _poolToken
    )
        external
        view
        returns (uint256)
    {
109: function getPositionBorrowShares(
        uint256 _nftId,
        address _poolToken
    )
        external
        view
        returns (uint256)
    {
120: function getPureCollateralAmount(
        uint256 _nftId,
        address _poolToken
    )
        external
        view
        returns (uint256)
    {
133: function getTimeStamp(
        address _poolToken
    )
        external
        view
        returns (uint256)
    {
143: function getPositionLendingTokenByIndex(
        uint256 _nftId,
        uint256 _index
    )
        public
        view
        returns (address)
    {
154: function getPositionLendingTokenLength(
        uint256 _nftId
    )
        public
        view
        returns (uint256)
    {
164: function getPositionBorrowTokenByIndex(
        uint256 _nftId,
        uint256 _index
    )
        public
        view
        returns (address)
    {
175: function getPositionBorrowTokenLength(
        uint256 _nftId
    )
        public
        view
        returns (uint256)
    {
187: function _setMaxValue(
        address _poolToken,
        uint256 _value
    )
        internal
    {
196: function _setBestPole(
        address _poolToken,
        uint256 _value
    )
        internal
    {
205: function _setIncreasePole(
        address _poolToken,
        bool _state
    )
        internal
    {
214: function _setPole(
        address _poolToken,
        uint256 _value
    )
        internal
    {
223: function _increaseTotalPool(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
232: function _decreaseTotalPool(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
241: function _increaseTotalDepositShares(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
250: function _decreaseTotalDepositShares(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
259: function _increasePseudoTotalBorrowAmount(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
268: function _decreasePseudoTotalBorrowAmount(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
277: function _increaseTotalBorrowShares(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
286: function _decreaseTotalBorrowShares(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
295: function _increasePseudoTotalPool(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
304: function _decreasePseudoTotalPool(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
313: function _setTimeStamp(
        address _poolToken,
        uint256 _time
    )
        internal
    {
322: function _setTimeStampScaling(
        address _poolToken,
        uint256 _time
    )
        internal
    {
331: function _increaseTotalBareToken(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
340: function _decreaseTotalBareToken(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
392: function _byPassCase(
        address _sender
    )
        internal
        view
        returns (bool)
    {
406: function _increaseTotalAndPseudoTotalPool(
        address _poolToken,
        uint256 _amount
    )
        internal
    {
423: function setPoolFee(
        address _poolToken,
        uint256 _newFee
    )
        external
        onlyFeeManager
    {
433: function _checkOwnerPosition(
        uint256 _nftId,
        address _msgSender
    )
        internal
        view
    {
446: function _validateNonZero(
        uint256 _value
    )
        internal
        pure
    {
457: function _validateZero(
        uint256 _value
    )
        internal
        pure
    {
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L24) | [38](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L38) | [48](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L48) | [58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L58) | [68](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L68) | [78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L78) | [88](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L88) | [98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L98) | [109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L109) | [120](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L120) | [133](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L133) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L143) | [154](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L154) | [164](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L164) | [175](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L175) | [187](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L187) | [196](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L196) | [205](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L205) | [214](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L214) | [223](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L223) | [232](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L232) | [241](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L241) | [250](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L250) | [259](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L259) | [268](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L268) | [277](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L277) | [286](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L286) | [295](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L295) | [304](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L304) | [313](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L313) | [322](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L322) | [331](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L331) | [340](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L340) | [392](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L392) | [406](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L406) | [423](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L423) | [433](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L433) | [446](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L446) | [457](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L457)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

273: function _doApprovals(
        address _wiseLendingAddress
    )
        internal
    {
158: constructor(
        address _wiseLendingAddress,
        address _pendleChilTokenAddress,
        address _pendleRouter,
        address _entryAsset,
        address _pendleSy,
        address _underlyingMarket,
        address _routerStatic,
        address _dexAddress,
        uint256 _collateralFactor
    )
        WrapperHelper(
            IWiseLending(
                _wiseLendingAddress
            ).WETH_ADDRESS()
        )
    {
```
[273](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L273) | [158](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L158)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

244: function _onlyChildContract(
        address _pendleMarket
    )
        private
        view
    {
255: function _syncSupply(
        address _pendleMarket
    )
        internal
    {
78: constructor(
        address _vePendle,
        address _pendleToken,
        address _voterContract,
        address _voterRewardsClaimerAddress,
        address _wiseOracleHub
    )
        OwnableMaster(
            msg.sender
        )
    {
```
[244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L244) | [255](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L255) | [78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L78)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

22: function _validToken(
        address _poolToken
    )
        internal
        view
    {
55: function _wrapDepositExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _depositAmount
    )
        internal
        returns (uint256)
    {
82: function _wrapWithdrawExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        address _underlyingAssetRecipient,
        uint256 _withdrawAmount
    )
        internal
        returns (
            uint256,
            uint256
        )
    {
112: function _wrapWithdrawExactShares(
        uint256 _nftId,
        address _underlyingAsset,
        address _underlyingAssetRecipient,
        uint256 _shareAmount
    )
        internal
        returns (uint256)
    {
140: function _wrapBorrowExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        address _underlyingAssetRecipient,
        uint256 _borrowAmount
    )
        internal
        returns (uint256)
    {
164: function _wrapAaveReturnValueDeposit(
        address _underlyingAsset,
        uint256 _depositAmount,
        address _targetAddress
    )
        internal
        returns (uint256 res)
    {
195: function _sendValue(
        address _recipient,
        uint256 _amount
    )
        internal
    {
218: function _getInfoPayback(
        uint256 _ethSent,
        uint256 _maxPaybackAmount
    )
        internal
        pure
        returns (
            uint256,
            uint256
        )
    {
242: function _prepareCollaterals(
        uint256 _nftId,
        address _poolToken
    )
        private
    {
278: function _prepareBorrows(
        uint256 _nftId,
        address _poolToken
    )
        private
    {
314: function getAavePoolAPY(
        address _underlyingAsset
    )
        public
        view
        returns (uint256)
    {
```
[22](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L22) | [55](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L55) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L82) | [112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L112) | [140](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L140) | [164](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L164) | [195](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L195) | [218](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L218) | [242](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L242) | [278](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L278) | [314](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L314)

```solidity
File: contracts/PositionNFTs.sol

53: function assignReserveRole(
        address _reserverForOthers,
        bool _state
    )
        external
        onlyMaster
    {
70: function forwardFeeManagerNFT(
        address _feeManagerContract
    )
        external
        onlyMaster
    {
98: function reservePositionForUser(
        address _user
    )
        onlyReserveRole
        external
        returns (uint256)
    {
110: function _reservePositionForUser(
        address _user
    )
        internal
        returns (uint256)
    {
174: function mintPositionForUser(
        address _user
    )
        external
        returns (uint256)
    {
192: function _mintPositionForUser(
        address _user
    )
        internal
        returns (uint256)
    {
221: function isOwner(
        uint256 _nftId,
        address _owner
    )
        external
        view
        returns (bool)
    {
244: function approveMint(
        address _spender,
        uint256 _nftId
    )
        external
    {
271: function walletOfOwner(
        address _owner
    )
        external
        view
        returns (uint256[] memory)
    {
319: function setBaseURI(
        string memory _newBaseURI
    )
        external
        onlyMaster
    {
327: function setBaseExtension(
        string memory _newBaseExtension
    )
        external
        onlyMaster
    {
340: function tokenURI(
        uint256 _tokenId
    )
        public
        view
        override
        returns (string memory)
    {
371: function _toString(
        uint256 _tokenId
    )
        internal
        pure
        returns (string memory str)
    {
24: constructor(
        string memory _name,
        string memory _symbol,
        string memory _baseURI
    )
        ERC721(
            _name,
            _symbol
        )
        OwnableMaster(
            msg.sender
        )
    {
```
[53](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L53) | [70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L70) | [98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L98) | [110](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L110) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L174) | [192](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L192) | [221](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L221) | [244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L244) | [271](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L271) | [319](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L319) | [327](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L327) | [340](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L340) | [371](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L371) | [24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L24)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

117: function _setLiquidationSettings(
        uint256 _baseReward,
        uint256 _baseRewardFarm,
        uint256 _newMaxFeeETH,
        uint256 _newMaxFeeFarmETH
    )
        internal
    {
51: constructor(
        address _master,
        address _wiseLendingAddress,
        address _aaveHubAddress
    )
        OwnableMaster(
            _master
        )
    {
```
[117](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L117) | [51](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L51)

```solidity
File: contracts/PoolManager.sol

24: function setParamsLASA(
        address _poolToken,
        uint256 _poolMulFactor,
        uint256 _upperBoundMaxRate,
        uint256 _lowerBoundMaxRate,
        bool _steppingDirection,
        bool _isFinal
    )
        external
        onlyMaster
    {
99: function setPoolParameters(
        address _poolToken,
        uint256 _collateralFactor,
        uint256 _maximumDeposit
    )
        external
        onlyMaster
    {
125: function setVerifiedIsolationPool(
        address _isolationPool,
        bool _state
    )
        external
        onlyMaster
    {
134: function createPool(
        CreatePool calldata _params
    )
        external
        onlyMaster
    {
145: function createCurvePool(
        CreatePool calldata _params,
        CurvePoolSettings calldata _settings
    )
        external
        onlyMaster
    {
163: function _createPool(
        CreatePool calldata _params
    )
        private
    {
276: function _getPoleValue(
        uint256 _poolMulFactor,
        uint256 _poleBoundRate
    )
        private
        pure
        returns (uint256)
    {
292: function _getDeltaPole(
        uint256 _maxPole,
        uint256 _minPole
    )
        private
        pure
        returns (uint256)
    {
303: function _getStartValue(
        uint256 _maxPole,
        uint256 _minPole
    )
        private
        pure
        returns (uint256)
    {
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L24) | [99](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L99) | [125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L125) | [134](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L134) | [145](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L145) | [163](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L163) | [276](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L276) | [292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L292) | [303](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L303)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

15: function _prepareBorrows(
        uint256 _nftId
    )
        internal
    {
47: function _prepareCollaterals(
        uint256 _nftId
    )
        internal
    {
77: function _setBadDebtPosition(
        uint256 _nftId,
        uint256 _amount
    )
        internal
    {
89: function _increaseTotalBadDebt(
        uint256 _amount
    )
        internal
    {
105: function _decreaseTotalBadDebt(
        uint256 _amount
    )
        internal
    {
121: function _eraseBadDebtUser(
        uint256 _nftId
    )
        internal
    {
134: function _updateUserBadDebt(
        uint256 _nftId
    )
        internal
    {
195: function _increaseFeeTokens(
        address _feeToken,
        uint256 _amount
    )
        internal
    {
208: function _decreaseFeeTokens(
        address _feeToken,
        uint256 _amount
    )
        internal
    {
220: function _setAllowedTokens(
        address _user,
        address _feeToken,
        bool _state
    )
        internal
    {
229: function _setAaveFlag(
        address _poolToken,
        address _underlyingToken
    )
        internal
    {
244: function getReceivingToken(
        address _paybackToken,
        address _receivingToken,
        uint256 _paybackAmount
    )
        public
        view
        returns (uint256 receivingAmount)
    {
270: function updatePositionCurrentBadDebt(
        uint256 _nftId
    )
        public
    {
292: function _distributeIncentives(
        uint256 _amount,
        address _poolToken,
        address _underlyingToken
    )
        internal
        returns (uint256)
    {
330: function _gatherIncentives(
        address _poolToken,
        address _underlyingToken,
        address _incentiveOwner,
        uint256 _amount
    )
        internal
        returns (uint256)
    {
383: function _checkValue(
        uint256 _value
    )
        internal
        pure
    {
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L15) | [47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L47) | [77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L77) | [89](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L89) | [105](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L105) | [121](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L121) | [134](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L134) | [195](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L195) | [208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L208) | [220](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L220) | [229](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L229) | [244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L244) | [270](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L270) | [292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L292) | [330](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L330) | [383](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L383)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

57: function _getPositionBorrowShares(
        uint256 _nftId
    )
        internal
        view
        returns (uint256)
    {
75: function _getPositionBorrowSharesAave(
        uint256 _nftId
    )
        internal
        view
        returns (uint256)
    {
92: function _getPositionBorrowTokenAmount(
        uint256 _nftId
    )
        internal
        view
        returns (uint256 tokenAmount)
    {
110: function _getPositionBorrowTokenAmountAave(
        uint256 _nftId
    )
        internal
        view
        returns (uint256 tokenAmountAave)
    {
136: function _getPositionLendingShares(
        uint256 _nftId
    )
        internal
        view
        returns (uint256)
    {
153: function _getPostionCollateralTokenAmount(
        uint256 _nftId
    )
        internal
        view
        returns (uint256)
    {
174: function getPositionBorrowETH(
        uint256 _nftId
    )
        public
        view
        returns (uint256)
    {
229: function getTotalWeightedCollateralETH(
        uint256 _nftId
    )
        public
        view
        returns (uint256)
    {
243: function _getTokensInETH(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        internal
        view
        returns (uint256)
    {
257: function _getEthInTokens(
        address _tokenAddress,
        uint256 _ethAmount
    )
        internal
        view
        returns (uint256)
    {
271: function getLeverageAmount(
        uint256 _initialAmount,
        uint256 _leverage
    )
        public
        pure
        returns (uint256)
    {
289: function _getApproxNetAPY(
        uint256 _initialAmount,
        uint256 _leverage,
        uint256 _wstETHAPY
    )
        internal
        view
        returns (
            uint256,
            bool
        )
    {
344: function _getNewBorrowRate(
        uint256 _borrowAmount
    )
        internal
        view
        returns (uint256)
    {
389: function _checkDebtRatio(
        uint256 _nftId
    )
        internal
        view
        returns (bool)
    {
416: function _notBelowMinDepositAmount(
        uint256 _amount
    )
        internal
        view
        returns (bool)
    {
```
[57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L57) | [75](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L75) | [92](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L92) | [110](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L110) | [136](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L136) | [153](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L153) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L174) | [229](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L229) | [243](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L243) | [257](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L257) | [271](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L271) | [289](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L289) | [344](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L344) | [389](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L389) | [416](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L416)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

32: constructor(
        address _master,
        address _aaveAddress,
        address _wiseLendingAddress,
        address _oracleHubAddress,
        address _wiseSecurityAddress,
        address _positionNFTAddress
    )
        OwnableMaster(
            _master
        )
    {
```
[32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L32)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

62: function changeMinDeposit(
        uint256 _newMinDeposit
    )
        external
        onlyMaster
    {
82: function shutDownFarm(
        bool _state
    )
        external
        onlyMaster
    {
95: function enterFarm(
        bool _isAave,
        uint256 _amount,
        uint256 _leverage,
        uint256 _allowedSpread
    )
        external
        isActive
        updatePools
        returns (uint256)
    {
141: function enterFarmETH(
        bool _isAave,
        uint256 _leverage,
        uint256 _allowedSpread
    )
        external
        payable
        isActive
        updatePools
        returns (uint256)
    {
209: function exitFarm(
        uint256 _keyId,
        uint256 _allowedSpread,
        bool _ethBack
    )
        external
        updatePools
        onlyKeyOwner(_keyId)
    {
253: function manuallyPaybackShares(
        uint256 _keyId,
        uint256 _paybackShares
    )
        external
        updatePools
    {
273: function manuallyWithdrawShares(
        uint256 _keyId,
        uint256 _withdrawShares
    )
        external
        updatePools
        onlyKeyOwner(_keyId)
    {
```
[62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L62) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L82) | [95](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L95) | [141](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L141) | [209](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L209) | [253](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L253) | [273](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L273)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

8: function _findIndex(
        address[] memory _array,
        address _value
    )
        internal
        pure
        returns (uint256)
    {
31: function _calcExpiry(
        uint128 _weeks
    )
        internal
        view
        returns (uint128)
    {
60: function _getPositionData(
        address _user
    )
        private
        view
        returns (LockedPosition memory)
    {
72: function _getAmountToSend(
        address _tokenAddress,
        uint256 _rewardValue
    )
        internal
        view
        returns (uint256)
    {
94: function pendleChildCompoundInfoReservedForCompound(
        address _pendleMarket
    )
        external
        view
        returns (uint256[] memory)
    {
104: function pendleChildCompoundInfoLastIndex(
        address _pendleMarket
    )
        external
        view
        returns (uint128[] memory)
    {
114: function pendleChildCompoundInfoRewardTokens(
        address _pendleMarket
    )
        external
        view
        returns (address[] memory)
    {
143: function _getRewardTokens(
        address _pendleMarket
    )
        internal
        view
        returns (address[] memory)
    {
155: function _getUserReward(
        address _pendleMarket,
        address _rewardToken,
        address _user
    )
        internal
        view
        returns (UserReward memory)
    {
172: function _getUserRewardIndex(
        address _pendleMarket,
        address _rewardToken,
        address _user
    )
        internal
        view
        returns (uint128)
    {
188: function _getTokensInETH(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        internal
        view
        returns (uint256)
    {
202: function _getTokensFromETH(
        address _tokenAddress,
        uint256 _ethValue
    )
        internal
        view
        returns (uint256)
    {
216: function _compareHashes(
        address _pendleMarket,
        address[] memory rewardTokensToCompare
    )
        internal
        view
        returns (bool)
    {
235: function _setRewardTokens(
        address _pendleMarket,
        address[] memory _rewardTokens
    )
        internal
    {
```
[8](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L8) | [31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L31) | [60](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L60) | [72](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L72) | [94](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L94) | [104](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L104) | [114](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L114) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L143) | [155](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L155) | [172](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L172) | [188](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L188) | [202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L202) | [216](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L216) | [235](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L235)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

15: function getApproxNetAPY(
        uint256 _initialAmount,
        uint256 _leverage,
        uint256 _pendleChildApy
    )
        external
        view
        returns (
            uint256,
            bool
        )
    {
41: function getNewBorrowRate(
        uint256 _borrowAmount
    )
        external
        view
        returns (uint256)
    {
57: function getLiveDebtRatio(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
93: function liquidatePartiallyFromToken(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        uint256 _shareAmountToPay
    )
        external
        updatePools
        returns (
            uint256 paybackAmount,
            uint256 receivingAmount
        )
    {
119: function _manuallyPaybackShares(
        uint256 _nftId,
        uint256 _paybackShares
    )
        internal
    {
157: function _manuallyWithdrawShares(
        uint256 _nftId,
        uint256 _withdrawShares
    )
        internal
    {
185: function _openPosition(
        bool _isAave,
        uint256 _nftId,
        uint256 _initialAmount,
        uint256 _leverage,
        uint256 _allowedSpread
    )
        internal
    {
230: function _closingPosition(
        bool _isAave,
        uint256 _nftId,
        uint256 _allowedSpread,
        bool _ethBack
    )
        internal
    {
274: function _registrationFarm(
        uint256 _nftId
    )
        internal
    {
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L15) | [41](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L41) | [57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L57) | [93](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L93) | [119](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L119) | [157](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L157) | [185](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L185) | [230](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L230) | [274](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L274)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

50: function setFarmContract(
        address _farmContract
    )
        external
        onlyMaster
    {
66: function mintKey(
        address _keyOwner,
        uint256 _keyId
    )
        external
        onlyFarmContract
    {
82: function burnKey(
        uint256 _keyId
    )
        external
        onlyFarmContract
    {
92: function checkOwner(
        uint256 _keyId,
        address _ownerAddress
    )
        external
        view
        returns (bool)
    {
109: function walletOfOwner(
        address _owner
    )
        external
        view
        returns (uint256[] memory)
    {
161: function setBaseURI(
        string calldata _newBaseURI
    )
        external
        onlyMaster
    {
169: function setBaseExtension(
        string calldata _newBaseExtension
    )
        external
        onlyMaster
    {
182: function tokenURI(
        uint256 _tokenId
    )
        public
        view
        override
        returns (string memory)
    {
213: function _toString(
        uint256 _tokenId
    )
        internal
        pure
        returns (string memory str)
    {
```
[50](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L50) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L66) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L82) | [92](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L92) | [109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L109) | [161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L161) | [169](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L169) | [182](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L182) | [213](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L213)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

40: function _onlyKeyOwner(
        uint256 _keyId
    )
        private
        view
    {
76: function _reserveKey(
        address _userAddress,
        uint256 _wiseLendingNFT
    )
        internal
        returns (uint256)
    {
95: function isOwner(
        uint256 _keyId,
        address _owner
    )
        public
        view
        returns (bool)
    {
114: function _mintKeyForUser(
        uint256 _keyId,
        address _userAddress
    )
        internal
        returns (uint256)
    {
159: function onERC721Received(
        address _operator,
        address _from,
        uint256 _tokenId,
        bytes calldata _data
    )
        external
        returns (bytes4)
    {
54: constructor(
        address _powerFarmNFTs
    ) {
```
[40](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L40) | [76](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L76) | [95](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L95) | [114](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L114) | [159](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L159) | [54](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L54)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

121: function _getLpToAssetRateWrapper(
        IPMarket _market,
        uint32 _duration
    )
        internal
        view
        returns (uint256)
    {
31: constructor(
        address _pendleMarketAddress,
        IPriceFeed _priceFeedChainLinkEth,
        IOraclePendle _oraclePendlePt,
        string memory _oracleName,
        uint32 _twapDuration
    )
    {
```
[121](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L121) | [31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L31)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

44: constructor(
        address _wethAddress,
        address _ethPriceFeed,
        address _uniswapV3Factory
    )
        OwnableMaster(
            msg.sender
        )
    {
```
[44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L44)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

23: constructor(
        address _pendleMarketAddress,
        IPriceFeed _priceFeedChainLinkUsd,
        IPriceFeed _priceFeedChainLinkEthUsd,
        IOraclePendle _oraclePendlePt,
        string memory _oracleName,
        uint32 _twapDuration
    )
    {
```
[23](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L23)

```solidity
File: contracts/WrapperHub/Declarations.sol

80: function _checkOwner(
        uint256 _nftId
    )
        internal
        view
    {
92: function _checkPositionLocked(
        uint256 _nftId
    )
        internal
        view
    {
104: function setWiseSecurity(
        address _securityAddress
    )
        external
        onlyMaster
    {
42: constructor(
        address _master,
        address _aaveAddress,
        address _lendingAddress
    )
        OwnableMaster(
            _master
        )
        WrapperHelper(
            IWiseLending(
                _lendingAddress
            ).WETH_ADDRESS()
        )
    {
```
[80](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L80) | [92](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L92) | [104](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L104) | [42](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L42)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

23: constructor(
        address _pendleMarketAddress,
        IPriceFeed _priceFeedChainLinkEth,
        IOraclePendle _oraclePendlePt,
        string memory _oracleName,
        uint32 _twapDuration
    )
    {
```
[23](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L23)

```solidity
File: contracts/OwnableMaster.sol

70: function proposeOwner(
        address _proposedOwner
    )
        external
        onlyMaster
    {
56: constructor(
        address _master
    ) {
```
[70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L70) | [56](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L56)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

34: function deploy(
        address _underlyingPendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
        returns (address)
    {
55: function _clone(
        address _underlyingPendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        private
        returns (address pendlePowerFarmTokenAddress)
    {
15: constructor(
        address _pendlePowerFarmController
    )
    {
```
[34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L34) | [55](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L55) | [15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L15)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

77: function getRoundData(
        uint80 _roundId
    )
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
20: constructor(
        address _pendleLpOracle,
        address _pendleChild
    )
        CustomOracleSetup()
    {
```
[77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L77) | [20](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L20)

```solidity
File: contracts/DerivativeOracles/CustomOracleSetup.sol

32: function setLastUpdateGlobal(
        uint256 _time
    )
        external
        onlyOwner
    {
41: function setRoundData(
        uint80 _roundId,
        uint256 _updateTime
    )
        external
        onlyOwner
    {
51: function setGlobalAggregatorRoundId(
        uint80 _aggregatorRoundId
    )
        external
        onlyOwner
    {
```
[32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L32) | [41](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L41) | [51](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L51)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

11: function _sendValue(
        address _recipient,
        uint256 _amount
    )
        internal
    {
```
[11](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L11)

```solidity
File: contracts/TransferHub/WrapperHelper.sol

24: function _wrapETH(
        uint256 _value
    )
        internal
    {
38: function _unwrapETH(
        uint256 _value
    )
        internal
    {
10: constructor(
        address _wethAddress
    )
    {
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol#L24) | [38](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol#L38) | [10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol#L10)

```solidity
File: contracts/TransferHub/TransferHelper.sol

13: function _safeTransfer(
        address _token,
        address _to,
        uint256 _value
    )
        internal
    {
34: function _safeTransferFrom(
        address _token,
        address _from,
        address _to,
        uint256 _value
    )
        internal
    {
```
[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol#L13) | [34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol#L34)

```solidity
File: contracts/TransferHub/ApprovalHelper.sol

13: function _safeApprove(
        address _token,
        address _spender,
        uint256 _value
    )
        internal
    {
```
[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/ApprovalHelper.sol#L13)
</details>

### [N-21] Unnecessary Initialization of `address` with `address(0)` value

By default, `address` variables in Solidity are initialized to `address(0)`.
Explicitly setting variables to address(0) during their declaration is redundant and might cause confusion.
Removing the explicit zero initialization can improve code readability and understanding.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/OwnableMaster.sol

14: address internal constant ZERO_ADDRESS = address(0x0);
```
[14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L14)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

11: address internal constant ZERO_ADDRESS = address(0x0);
```
[11](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L11)
</details>

### [N-22] Contract Doesn't Handle All NFT Types

Supporting both ERC-721 and ERC-1155 standards increases the potential market for your NFT contract.
Consider implementing both standards to reach a broader audience.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

/// @audit - noERC1155Received() not implemented
160: function onERC721Received(
```
[160](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L160)
</details>

### [N-23] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function

Duplicated require() or revert() checks should be refactored to a modifier or function.
This helps in maintaining a clean and organized codebase and saves deployment costs.

<details>
<summary><i>13 issue instances in 4 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

246: if (_checkBlacklisted(_poolToken) == true) {

            if (overallETHBorrowBare(_nftId) > 0) {
                revert OpenBorrowPosition();
284: if (_checkBlacklisted(_poolToken) == true) {

            if (overallETHBorrowBare(_nftId) > 0) {
                revert OpenBorrowPosition();
367: if (_checkBlacklisted(_poolToken) == true) {

            if (overallETHBorrowBare(_nftId) > 0) {
                revert OpenBorrowPosition();
```
[246](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L246) | [284](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L284) | [367](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L367)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

539: if (_underlyingLpAssetAmount == 0) {
            revert ZeroAmount();
654: if (_underlyingLpAssetAmount == 0) {
            revert ZeroAmount();
```
[539](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L539) | [654](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L654)

```solidity
File: contracts/FeeManager/FeeManager.sol

324: if (_newOwner == incentiveOwnerA) {
            revert NotAllowed();
361: if (_newOwner == incentiveOwnerA) {
            revert NotAllowed();
328: if (_newOwner == incentiveOwnerB) {
            revert NotAllowed();
365: if (_newOwner == incentiveOwnerB) {
            revert NotAllowed();
757: if (WISE_LENDING.getTotalDepositShares(_paybackToken) == 0) {
            revert PoolNotActive();
832: if (WISE_LENDING.getTotalDepositShares(_paybackToken) == 0) {
            revert PoolNotActive();
```
[324](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L324) | [361](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L361) | [328](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L328) | [365](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L365) | [757](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L757) | [832](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L832)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

211: if (ethValueAfter < ethValueBefore) {
            revert TooMuchValueLost();
504: if (ethValueAfter < ethValueBefore) {
            revert TooMuchValueLost();
```
[211](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L211) | [504](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L504)
</details>

### [N-24] `constant` variables not using all capital letters

Variable names for `constant` variables should consist of all capital letters.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: contracts/WiseOracleHub/Declarations.sol

97: uint8 internal constant _decimalsUSD = 8;
100: uint8 internal constant _decimalsETH = 18;
```
[97](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L97) | [100](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L100)
</details>

### [N-25] Non-uppercase/Constant-case Naming for `immutable` Variables

For better readability and adherence to common naming conventions, variable names declared as immutable should be written in all uppercase letters.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

54: address public immutable aaveTokenAddresses;
55: address public immutable borrowTokenAddresses;
```
[54](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L54) | [55](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L55)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

85: uint8 internal immutable _decimalsWETH;
```
[85](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L85)
</details>

### [N-26] Function Names Not in mixedCase

Function names should use mixedCase for better readability and consistency with Solidity style guidelines. 
Examples of good practice include: getBalance, transfer, verifyOwner, addMember, changeOwner. 
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#function-names)

<details>
<summary><i>60 issue instances in 17 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

388: function depositExactAmountETH(
400: function _depositExactAmountETH(
426: function depositExactAmountETHMint()
493: function solelyDepositETHMint()
508: function solelyDepositETH(
636: function withdrawExactAmountETH(
675: function withdrawExactSharesETH(
751: function solelyWithdrawETH(
975: function borrowExactAmountETH(
1088: function paybackExactAmountETH(
```
[388](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) | [400](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L400) | [426](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L426) | [493](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L493) | [508](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L508) | [636](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L636) | [675](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L675) | [751](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L751) | [975](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L975) | [1088](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1088)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

442: function overallLendingAPY(
499: function overallBorrowAPY(
556: function overallNetAPY(
```
[442](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L442) | [499](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L499) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L556)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

15: function overallETHCollateralsBoth(
65: function overallETHCollateralsWeighted(
107: function overallETHCollateralsBare(
145: function _overallETHCollateralsWeighted(
190: function getFullCollateralETH(
246: function _getCollateralOfTokenETHUpdated(
281: function getETHCollateralUpdated(
323: function getETHCollateral(
360: function overallETHBorrowBare(
394: function overallETHBorrowHeartbeat(
430: function overallETHBorrow(
486: function _overallETHBorrow(
610: function _getETHBorrowUpdated(
651: function getETHBorrow(
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L15) | [65](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L65) | [107](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L107) | [145](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L145) | [190](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L190) | [246](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L246) | [281](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L281) | [323](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L323) | [360](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L360) | [394](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L394) | [430](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L430) | [486](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L486) | [610](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L610) | [651](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L651)

```solidity
File: contracts/FeeManager/FeeManager.sol

315: function changeIncentiveUSDA(
352: function changeIncentiveUSDB(
```
[315](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L315) | [352](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L352)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

293: function _swapStETHintoETH(
309: function _withdrawPendleLPs(
383: function _closingRouteETH(
```
[293](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L293) | [309](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L309) | [383](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L383)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

264: function getETHPriceInUSD()
```
[264](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L264)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

599: function forwardETH(
```
[599](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L599)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

96: function getTokensInUSD(
123: function getTokensPriceInUSD(
143: function getTokensInETH(
171: function getTokensFromUSD(
198: function getTokensPriceFromUSD(
327: function getTokensFromETH(
```
[96](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L96) | [123](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L123) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L143) | [171](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L171) | [198](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L198) | [327](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L327)

```solidity
File: contracts/WrapperHub/AaveHub.sol

158: function depositExactAmountETHMint()
172: function depositExactAmountETH(
238: function withdrawExactAmountETH(
314: function withdrawExactSharesETH(
393: function borrowExactAmountETH(
482: function paybackExactAmountETH(
```
[158](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L158) | [172](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L172) | [238](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L238) | [314](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L314) | [393](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L393) | [482](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L482)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

314: function getAavePoolAPY(
```
[314](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L314)

```solidity
File: contracts/PositionNFTs.sol

70: function forwardFeeManagerNFT(
```
[70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L70)

```solidity
File: contracts/PoolManager.sol

24: function setParamsLASA(
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L24)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

174: function getPositionBorrowETH(
229: function getTotalWeightedCollateralETH(
243: function _getTokensInETH(
289: function _getApproxNetAPY(
```
[174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L174) | [229](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L229) | [243](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L243) | [289](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L289)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

141: function enterFarmETH(
184: function _getWiseLendingNFT()
```
[141](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L141) | [184](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L184)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

188: function _getTokensInETH(
202: function _getTokensFromETH(
```
[188](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L188) | [202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L202)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

15: function getApproxNetAPY(
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L15)

```solidity
File: contracts/TransferHub/WrapperHelper.sol

24: function _wrapETH(
38: function _unwrapETH(
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol#L24) | [38](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol#L38)
</details>

### [N-27] Insufficient Comments in Complex Functions

Complex functions spanning multiple lines should have more comments to better explain the purpose of each logic step.
Proper commenting can greatly improve the readability and maintainability of the codebase. Consider adding explicit comments
to provide insights into the function's operation.

<details>
<summary><i>46 issue instances in 19 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

/// @audit function with 31 lines has only 0 comment lines
808: function _coreSolelyWithdraw(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        private
    {
/// @audit function with 46 lines has only 0 comment lines
1088: function paybackExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        syncPool(WETH_ADDRESS)
        returns (uint256)
    {
/// @audit function with 37 lines has only 0 comment lines
1250: function liquidatePartiallyFromTokens(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _paybackToken,
        address _receiveToken,
        uint256 _shareAmountToPay
    )
        external
        syncPool(_paybackToken)
        syncPool(_receiveToken)
        returns (uint256)
    {
```
[808](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L808) | [1088](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1088) | [1250](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1250)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

/// @audit function with 34 lines has only 0 comment lines
442: function overallLendingAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
/// @audit function with 34 lines has only 0 comment lines
499: function overallBorrowAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
/// @audit function with 53 lines has only 0 comment lines
556: function overallNetAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256, bool)
    {
/// @audit function with 32 lines has only 0 comment lines
641: function safeLimitPosition(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
/// @audit function with 43 lines has only 0 comment lines
695: function positionBlackListToken(
        uint256 _nftId
    )
        external
        view
        returns (bool, address)
    {
/// @audit function with 33 lines has only 0 comment lines
764: function maximumWithdrawToken(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval,
        uint256 _solelyWithdrawAmount
    )
        external
        view
        returns (uint256)
    {
```
[442](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L442) | [499](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L499) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L556) | [641](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L641) | [695](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L695) | [764](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L764)

```solidity
File: contracts/MainHelper.sol

/// @audit function with 30 lines has only 0 comment lines
298: function _cleanUp(
        address _poolToken
    )
        internal
    {
/// @audit function with 57 lines has only 0 comment lines
500: function _updatePseudoTotalAmounts(
        address _poolToken
    )
        private
    {
/// @audit function with 39 lines has only 0 comment lines
667: function _removePositionData(
        uint256 _nftId,
        address _poolToken,
        function(uint256) view returns (uint256) _getPositionTokenLength,
        function(uint256, uint256) view returns (address) _getPositionTokenByIndex,
        function(uint256, address) internal _deleteLastPositionData,
        bool isLending
    )
        private
    {
```
[298](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L298) | [500](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L500) | [667](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L667)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit function with 65 lines has only 0 comment lines
217: function _calculateRewardsClaimedOutside()
        internal
        returns (uint256[] memory)
    {
/// @audit function with 40 lines has only 0 comment lines
529: function depositExactAmount(
        uint256 _underlyingLpAssetAmount
    )
        external
        syncSupply
        returns (
            uint256,
            uint256
        )
    {
/// @audit function with 30 lines has only 0 comment lines
681: function initialize(
        address _underlyingPendleMarket,
        address _pendleController,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
    {
```
[217](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L217) | [529](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L529) | [681](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L681)

```solidity
File: contracts/FeeManager/FeeManager.sol

/// @audit function with 30 lines has only 0 comment lines
438: function removePoolTokenManual(
        address _poolToken
    )
        external
        onlyMaster
    {
/// @audit function with 39 lines has only 0 comment lines
628: function claimWiseFees(
        address _poolToken
    )
        public
    {
/// @audit function with 56 lines has only 0 comment lines
730: function paybackBadDebtForToken(
        uint256 _nftId,
        address _paybackToken,
        address _receivingToken,
        uint256 _shares
    )
        external
        returns (
            uint256 paybackAmount,
            uint256 receivingAmount
        )
    {
/// @audit function with 35 lines has only 0 comment lines
816: function paybackBadDebtNoReward(
        uint256 _nftId,
        address _paybackToken,
        uint256 _shares
    )
        external
        returns (uint256 paybackAmount)
    {
```
[438](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L438) | [628](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L628) | [730](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L730) | [816](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L816)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

/// @audit function with 54 lines has only 0 comment lines
59: function receiveFlashLoan(
        IERC20[] memory _flashloanToken,
        uint256[] memory _flashloanAmounts,
        uint256[] memory _feeAmounts,
        bytes memory _userData
    )
        external
    {
/// @audit function with 72 lines has only 0 comment lines
134: function _logicClosePosition(
        uint256 _nftId,
        uint256 _borrowShares,
        uint256 _lendingShares,
        uint256 _totalDebtBalancer,
        uint256 _allowedSpread,
        address _caller,
        bool _ethBack,
        bool _isAave
    )
        private
    {
/// @audit function with 93 lines has only 0 comment lines
414: function _logicOpenPosition(
        bool _isAave,
        uint256 _nftId,
        uint256 _depositAmount,
        uint256 _totalDebtBalancer,
        uint256 _allowedSpread
    )
        internal
    {
```
[59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L59) | [134](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L134) | [414](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L414)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

/// @audit function with 30 lines has only 0 comment lines
459: function _getTwapDerivatePrice(
        address _tokenAddress,
        UniTwapPoolInfo memory _uniTwapPoolInfo
    )
        internal
        view
        returns (uint256)
    {
/// @audit function with 40 lines has only 0 comment lines
592: function _recalibratePreview(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
    {
```
[459](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L459) | [592](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L592)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

/// @audit function with 40 lines has only 0 comment lines
52: function exchangeRewardsForCompoundingWithIncentive(
        address _pendleMarket,
        address _rewardToken,
        uint256 _rewardAmount
    )
        external
        syncSupply(_pendleMarket)
        returns (uint256)
    {
/// @audit function with 38 lines has only 0 comment lines
112: function exchangeLpFeesForPendleWithIncentive(
        address _pendleMarket,
        uint256 _pendleChildShares
    )
        external
        syncSupply(_pendleMarket)
        returns (
            uint256,
            uint256
        )
    {
/// @audit function with 54 lines has only 0 comment lines
210: function addPendleMarket(
        address _pendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
        onlyMaster
    {
/// @audit function with 44 lines has only 0 comment lines
364: function lockPendle(
        uint256 _amount,
        uint128 _weeks,
        bool _fromInside,
        bool _sameExpiry
    )
        external
        onlyMaster
        returns (uint256 newVeBalance)
    {
```
[52](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L52) | [112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L112) | [210](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L210) | [364](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L364)

```solidity
File: contracts/WiseCore.sol

/// @audit function with 42 lines has only 0 comment lines
44: function _coreWithdrawToken(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        uint256 _shares,
        bool _onBehalf
    )
        internal
    {
/// @audit function with 44 lines has only 0 comment lines
106: function _handleDeposit(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        internal
        returns (uint256)
    {
/// @audit function with 56 lines has only 0 comment lines
349: function _coreBorrowTokens(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        uint256 _shares,
        bool _onBehalf
    )
        internal
    {
/// @audit function with 53 lines has only 0 comment lines
460: function _withdrawOrAllocateSharesLiquidation(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _poolToken,
        uint256 _percentWishCollateral
    )
        private
        returns (uint256)
    {
/// @audit function with 42 lines has only 0 comment lines
543: function _calculateReceiveAmount(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _receiveTokens,
        uint256 _removePercentage
    )
        private
        returns (uint256 receiveAmount)
    {
/// @audit function with 47 lines has only 0 comment lines
625: function _coreLiquidation(
        CoreLiquidationStruct memory _data
    )
        internal
        returns (uint256 receiveAmount)
    {
```
[44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L44) | [106](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L106) | [349](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L349) | [460](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L460) | [543](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L543) | [625](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L625)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

/// @audit function with 37 lines has only 0 comment lines
266: function addTwapOracleDerivative(
        address _tokenAddress,
        address _partnerTokenAddress,
        address[2] calldata _uniPools,
        address[2] calldata _token0Array,
        address[2] calldata _token1Array,
        uint24[2] calldata _feeArray
    )
        external
        onlyMaster
    {
```
[266](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L266)

```solidity
File: contracts/WrapperHub/AaveHub.sol

/// @audit function with 48 lines has only 0 comment lines
482: function paybackExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        nonReentrant
        returns (uint256)
    {
/// @audit function with 35 lines has only 0 comment lines
556: function paybackExactShares(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _shares
    )
        external
        nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)
    {
```
[482](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L482) | [556](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L556)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

/// @audit function with 64 lines has only 0 comment lines
273: function _doApprovals(
        address _wiseLendingAddress
    )
        internal
    {
```
[273](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L273)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

/// @audit function with 40 lines has only 0 comment lines
117: function _setLiquidationSettings(
        uint256 _baseReward,
        uint256 _baseRewardFarm,
        uint256 _newMaxFeeETH,
        uint256 _newMaxFeeFarmETH
    )
        internal
    {
```
[117](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L117)

```solidity
File: contracts/PoolManager.sol

/// @audit function with 49 lines has only 0 comment lines
24: function setParamsLASA(
        address _poolToken,
        uint256 _poolMulFactor,
        uint256 _upperBoundMaxRate,
        uint256 _lowerBoundMaxRate,
        bool _steppingDirection,
        bool _isFinal
    )
        external
        onlyMaster
    {
/// @audit function with 80 lines has only 9 comment lines
163: function _createPool(
        CreatePool calldata _params
    )
        private
    {
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L24) | [163](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L163)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

/// @audit function with 39 lines has only 0 comment lines
134: function _updateUserBadDebt(
        uint256 _nftId
    )
        internal
    {
```
[134](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L134)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

/// @audit function with 33 lines has only 0 comment lines
174: function getPositionBorrowETH(
        uint256 _nftId
    )
        public
        view
        returns (uint256)
    {
```
[174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L174)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

/// @audit function with 30 lines has only 0 comment lines
109: function walletOfOwner(
        address _owner
    )
        external
        view
        returns (uint256[] memory)
    {
```
[109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L109)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

/// @audit function with 37 lines has only 0 comment lines
81: function latestAnswer()
        public
        view
        returns (uint256)
    {
```
[81](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L81)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

/// @audit function with 36 lines has only 0 comment lines
55: function _clone(
        address _underlyingPendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        private
        returns (address pendlePowerFarmTokenAddress)
    {
```
[55](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L55)
</details>

### [N-28] Common functions should be refactored to a common base contract

The codebase contains several functions with the same implementations across multiple files.
To enhance maintainability and reduce potential errors, consider refactoring these common functions into a shared base contract.
This practice promotes code reuse and easier future upgrades.

<details>
<summary><i>14 issue instances in 9 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

1588: function _reservePosition()
        private
        returns (uint256)
    {
        return POSITION_NFT.reservePositionForUser(
            msg.sender
        );
    }
```
[1588](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1588)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

46: function _reservePosition()
        internal
        returns (uint256)
    {
        return POSITION_NFT.reservePositionForUser(
            msg.sender
        );
    }
```
[46](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L46)

```solidity
File: contracts/PositionNFTs.sol

319: function setBaseURI(
        string memory _newBaseURI
    )
        external
        onlyMaster
    {
        baseURI = _newBaseURI;
    }
327: function setBaseExtension(
        string memory _newBaseExtension
    )
        external
        onlyMaster
    {
        baseExtension = _newBaseExtension;
    }
```
[319](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L319) | [327](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L327)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

161: function setBaseURI(
        string calldata _newBaseURI
    )
        external
        onlyMaster
    {
        baseURI = _newBaseURI;
    }
169: function setBaseExtension(
        string calldata _newBaseExtension
    )
        external
        onlyMaster
    {
        baseExtension = _newBaseExtension;
    }
```
[161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L161) | [169](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L169)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

243: function _getTokensInETH(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        internal
        view
        returns (uint256)
    {
        return ORACLE_HUB.getTokensInETH(
            _tokenAddress,
            _tokenAmount
        );
    }
```
[243](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L243)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

188: function _getTokensInETH(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        internal
        view
        returns (uint256)
    {
        return ORACLE_HUB.getTokensInETH(
            _tokenAddress,
            _tokenAmount
        );
    }
```
[188](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L188)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

139: function decimals()
        external
        pure
        returns (uint8)
    {
        return FEED_DECIMALS;
    }
151: function latestRoundData()
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
        return (
            roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answeredInRound
        );
    }
```
[139](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L139) | [151](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L151)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

134: function decimals()
        external
        pure
        returns (uint8)
    {
        return FEED_DECIMALS;
    }
146: function latestRoundData()
        public
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
        return (
            roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answeredInRound
        );
    }
```
[134](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L134) | [146](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L146)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

113: function decimals()
        external
        pure
        returns (uint8)
    {
        return FEED_DECIMALS;
    }
125: function latestRoundData()
        public
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
        return (
            roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answeredInRound
        );
    }
```
[113](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L113) | [125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L125)
</details>

### [N-29] Variable Names Not in mixedCase

State or Local Variable names in your contract don't align with the Solidity naming convention.
For clarity and code consistency, it's recommended to use mixedCase for local and state variables that are not constants, and add a trailing underscore for internal variables.
Adhering to this convention helps in improving code readability and maintenance.
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#function-names)

<details>
<summary><i>13 issue instances in 5 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit - Variable name `UNDERLYING_PENDLE_MARKET` should be in mixedCase
28: address public UNDERLYING_PENDLE_MARKET;
/// @audit - Variable name `PENDLE_POWER_FARM_CONTROLLER` should be in mixedCase
29: address public PENDLE_POWER_FARM_CONTROLLER;
/// @audit - Variable name `PENDLE_MARKET` should be in mixedCase
38: IPendleMarket public PENDLE_MARKET;
/// @audit - Variable name `PENDLE_SY` should be in mixedCase
41: IPendleSy public PENDLE_SY;
/// @audit - Variable name `PENDLE_CONTROLLER` should be in mixedCase
44: IPendleController public PENDLE_CONTROLLER;
/// @audit - Variable name `MAX_CARDINALITY` should be in mixedCase
47: uint16 public MAX_CARDINALITY;
/// @audit - Variable name `INITIAL_TIME_STAMP` should be in mixedCase
61: uint256 private INITIAL_TIME_STAMP;
```
[28](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L28) | [29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L29) | [38](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L38) | [41](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L41) | [44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L44) | [47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L47) | [61](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L61)

```solidity
File: contracts/WiseLendingDeclaration.sol

/// @audit - Variable name `AAVE_HUB_ADDRESS` should be in mixedCase
162: address internal AAVE_HUB_ADDRESS;
/// @audit - Variable name `WISE_SECURITY` should be in mixedCase
174: IWiseSecurity public WISE_SECURITY;
/// @audit - Variable name `FEE_MANAGER` should be in mixedCase
177: IFeeManagerLight internal FEE_MANAGER;
```
[162](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L162) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L174) | [177](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L177)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

/// @audit - Variable name `ARB_REWARDS` should be in mixedCase
72: IArbRewardsClaimer public ARB_REWARDS;
```
[72](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L72)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

/// @audit - Variable name `ALLOWED_DIFFERENCE` should be in mixedCase
112: uint256 internal ALLOWED_DIFFERENCE = 10250;
```
[112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L112)

```solidity
File: contracts/WrapperHub/Declarations.sol

/// @audit - Variable name `WISE_SECURITY` should be in mixedCase
32: IWiseSecurity public WISE_SECURITY;
```
[32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L32)
</details>

### [N-30] Inconsistent Modifier Order in Function Declaration

The modifier order for a function should be: Visibility -> Mutability -> Virtual -> Override -> Custom modifiers.
The following instances violate this recommended order.

<details>
<summary><i>4 issue instances in 2 files:</i></summary>

```solidity
File: contracts/MainHelper.sol

391: function _preparationTokens(
        mapping(uint256 => address[]) storage _userTokenData,
        uint256 _nftId,
        address _poolToken
    )
        internal
        returns (address[] memory)
    {
614: function _addPositionTokenData(
        uint256 _nftId,
        address _poolToken,
        mapping(bytes32 => bool) storage hashMap,
        mapping(uint256 => address[]) storage userTokenData
    )
        internal
    {
1184: function _updatePoolStorage(
        address _poolToken,
        uint256 _amount,
        uint256 _shares,
        function(address, uint256) functionAmountA,
        function(address, uint256) functionAmountB,
        function(address, uint256) functionSharesA
    )
        internal
    {
```
[391](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L391) | [614](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L614) | [1184](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1184)

```solidity
File: contracts/PositionNFTs.sol

98: function reservePositionForUser(
        address _user
    )
        onlyReserveRole
        external
        returns (uint256)
    {
```
[98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L98)
</details>

### [N-31] Unused Function Parameters

Function parameters that are not used within the function body can introduce confusion and reduce code clarity.
It's recommended to comment out the variable name or remove it if not needed. This will not only improve the readability of the code but also suppress any compiler warnings associated with unused parameters.
Ensure that removing or commenting out the parameter doesn't affect any function calls or overrides in derived contracts.

If function is `virtual`, it's recommended to remove the parameter name from the function declaration.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/MainHelper.sol

/// @audit `bool` never used in function body or passed to modifiers
614: function _addPositionTokenData(
        uint256 _nftId,
        address _poolToken,
        mapping(bytes32 => bool) storage hashMap,
        mapping(uint256 => address[]) storage userTokenData
    )
        internal
    {
```
[614](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L614)
</details>

### [N-32] Consider bounding input array length

The functions in question operate on arrays without established boundaries, executing function calls for each of their entries.
If these arrays become excessively long, a function might revert due to gas constraints.
To enhance user experience, consider incorporating a `require()` statement that enforces a sensible maximum array length.
This approach can avoid unnecessary computational work and ensure smoother transactions.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/WrapperHub/AaveHub.sol

/// @audit `_underlyingAssets` is a function array parameter and `_underlyingAssets.length` is not bounded
71: for (uint256 i = 0; i < _underlyingAssets.length; i++) {
```
[71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L71)
</details>

### [N-33] Named Imports of Parent Contracts Are Missing

It's important to have named imports for parent contracts to ensure code readability and maintainability.
Missing named imports can make it difficult to understand the code hierarchy and can lead to issues in the future.

<details>
<summary><i>37 issue instances in 19 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

/// @audit Missing named import for parent contract: `PoolManager`
42: contract WiseLending is PoolManager {
```
[42](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L42)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

/// @audit Missing named import for parent contract: `WiseSecurityHelper`
23: contract WiseSecurity is WiseSecurityHelper, ApprovalHelper {
/// @audit Missing named import for parent contract: `ApprovalHelper`
23: contract WiseSecurity is WiseSecurityHelper, ApprovalHelper {
```
[23](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L23) | [23](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L23)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit Missing named import for parent contract: `SimpleERC20`
24: contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
/// @audit Missing named import for parent contract: `TransferHelper`
24: contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L24) | [24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L24)

```solidity
File: contracts/FeeManager/FeeManager.sol

/// @audit Missing named import for parent contract: `FeeManagerHelper`
24: contract FeeManager is FeeManagerHelper {
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L24)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

/// @audit Missing named import for parent contract: `PendlePowerFarmControllerHelper`
7: contract PendlePowerFarmController is PendlePowerFarmControllerHelper {
```
[7](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L7)

```solidity
File: contracts/WiseLendingDeclaration.sol

/// @audit Missing named import for parent contract: `OwnableMaster`
29: contract WiseLendingDeclaration is
    OwnableMaster,
    WrapperHelper,
    SendValueHelper
{
/// @audit Missing named import for parent contract: `WrapperHelper`
29: contract WiseLendingDeclaration is
    OwnableMaster,
    WrapperHelper,
    SendValueHelper
{
/// @audit Missing named import for parent contract: `SendValueHelper`
29: contract WiseLendingDeclaration is
    OwnableMaster,
    WrapperHelper,
    SendValueHelper
{
```
[29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L29) | [29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L29) | [29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L29)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

/// @audit Missing named import for parent contract: `OracleHelper`
30: contract WiseOracleHub is OracleHelper {
```
[30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L30)

```solidity
File: contracts/WrapperHub/AaveHub.sol

/// @audit Missing named import for parent contract: `AaveHelper`
24: contract AaveHub is AaveHelper, TransferHelper, ApprovalHelper {
/// @audit Missing named import for parent contract: `TransferHelper`
24: contract AaveHub is AaveHelper, TransferHelper, ApprovalHelper {
/// @audit Missing named import for parent contract: `ApprovalHelper`
24: contract AaveHub is AaveHelper, TransferHelper, ApprovalHelper {
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L24) | [24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L24) | [24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L24)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

/// @audit Missing named import for parent contract: `WrapperHelper`
35: contract PendlePowerFarmDeclarations is
    WrapperHelper,
    TransferHelper,
    ApprovalHelper,
    SendValueHelper
{
/// @audit Missing named import for parent contract: `TransferHelper`
35: contract PendlePowerFarmDeclarations is
    WrapperHelper,
    TransferHelper,
    ApprovalHelper,
    SendValueHelper
{
/// @audit Missing named import for parent contract: `ApprovalHelper`
35: contract PendlePowerFarmDeclarations is
    WrapperHelper,
    TransferHelper,
    ApprovalHelper,
    SendValueHelper
{
/// @audit Missing named import for parent contract: `SendValueHelper`
35: contract PendlePowerFarmDeclarations is
    WrapperHelper,
    TransferHelper,
    ApprovalHelper,
    SendValueHelper
{
```
[35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L35) | [35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L35) | [35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L35) | [35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L35)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

/// @audit Missing named import for parent contract: `OwnableMaster`
29: contract PendlePowerFarmControllerBase is
    OwnableMaster,
    TransferHelper,
    ApprovalHelper
{
/// @audit Missing named import for parent contract: `TransferHelper`
29: contract PendlePowerFarmControllerBase is
    OwnableMaster,
    TransferHelper,
    ApprovalHelper
{
/// @audit Missing named import for parent contract: `ApprovalHelper`
29: contract PendlePowerFarmControllerBase is
    OwnableMaster,
    TransferHelper,
    ApprovalHelper
{
```
[29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L29) | [29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L29) | [29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L29)

```solidity
File: contracts/PositionNFTs.sol

/// @audit Missing named import for parent contract: `ERC721Enumerable`
10: contract PositionNFTs is ERC721Enumerable, OwnableMaster {
/// @audit Missing named import for parent contract: `OwnableMaster`
10: contract PositionNFTs is ERC721Enumerable, OwnableMaster {
```
[10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L10) | [10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L10)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

/// @audit Missing named import for parent contract: `OwnableMaster`
44: contract WiseSecurityDeclarations is OwnableMaster {
```
[44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L44)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

/// @audit Missing named import for parent contract: `FeeManagerEvents`
30: contract DeclarationsFeeManager is FeeManagerEvents, OwnableMaster {
/// @audit Missing named import for parent contract: `OwnableMaster`
30: contract DeclarationsFeeManager is FeeManagerEvents, OwnableMaster {
```
[30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L30) | [30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L30)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

/// @audit Missing named import for parent contract: `OwnableMaster`
14: contract PendlePowerManager is OwnableMaster, PendlePowerFarm, MinterReserver {
/// @audit Missing named import for parent contract: `PendlePowerFarm`
14: contract PendlePowerManager is OwnableMaster, PendlePowerFarm, MinterReserver {
/// @audit Missing named import for parent contract: `MinterReserver`
14: contract PendlePowerManager is OwnableMaster, PendlePowerFarm, MinterReserver {
```
[14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L14) | [14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L14) | [14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L14)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

/// @audit Missing named import for parent contract: `ERC721Enumerable`
23: contract PowerFarmNFTs is ERC721Enumerable, OwnableMaster {
/// @audit Missing named import for parent contract: `OwnableMaster`
23: contract PowerFarmNFTs is ERC721Enumerable, OwnableMaster {
```
[23](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L23) | [23](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L23)

```solidity
File: contracts/WrapperHub/Declarations.sol

/// @audit Missing named import for parent contract: `OwnableMaster`
20: contract Declarations is OwnableMaster, AaveEvents, WrapperHelper {
/// @audit Missing named import for parent contract: `AaveEvents`
20: contract Declarations is OwnableMaster, AaveEvents, WrapperHelper {
/// @audit Missing named import for parent contract: `WrapperHelper`
20: contract Declarations is OwnableMaster, AaveEvents, WrapperHelper {
```
[20](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L20) | [20](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L20) | [20](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L20)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

/// @audit Missing named import for parent contract: `CustomOracleSetup`
12: contract PendleChildLpOracle is CustomOracleSetup  {
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L12)

```solidity
File: contracts/TransferHub/TransferHelper.sol

/// @audit Missing named import for parent contract: `CallOptionalReturn`
6: contract TransferHelper is CallOptionalReturn {
```
[6](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol#L6)

```solidity
File: contracts/TransferHub/ApprovalHelper.sol

/// @audit Missing named import for parent contract: `CallOptionalReturn`
6: contract ApprovalHelper is CallOptionalReturn {
```
[6](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/ApprovalHelper.sol#L6)
</details>

### [N-34] Prefer Casting to `bytes` or `bytes32` Over `abi.encodePacked()` for Single Arguments

When using `abi.encodePacked()` on a single argument, it is often clearer to use a cast to `bytes` or `bytes32`.
This improves the semantic clarity of the code, making it easier for reviewers to understand the developer's intentions.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

48: abi.encodePacked(
                            "ETH_USD_PLACEHOLDER"
                        )
```
[48](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L48)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

24: abi.encodePacked(
                    _pendlePowerFarmController
                )
66: abi.encodePacked(
                _underlyingPendleMarket
            )
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L24) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L66)
</details>

### [N-35] Unused `Ownable` Inheritance in Contract

The contract inherits from the `Ownable` contract but does not utilize its `onlyOwner` modifier.
This might indicate an oversight or a lack of access control on some methods.
Review the contract design and either apply the `onlyOwner` modifier where appropriate, or remove the `Ownable` inheritance if it is not required.

<details>
<summary><i>9 issue instances in 9 files:</i></summary>

```solidity
File: contracts/WiseLendingDeclaration.sol

5: import "./OwnableMaster.sol";
```
[5](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L5)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

5: import "../../OwnableMaster.sol";
```
[5](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L5)

```solidity
File: contracts/PositionNFTs.sol

7: import "./OwnableMaster.sol";
```
[7](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L7)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

17: import "../OwnableMaster.sol";
```
[17](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L17)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

13: import "../OwnableMaster.sol";
```
[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L13)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

12: import "../../OwnableMaster.sol";
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L12)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

11: import "../../OwnableMaster.sol";
```
[11](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L11)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

12: import "../OwnableMaster.sol";
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L12)

```solidity
File: contracts/WrapperHub/Declarations.sol

12: import "../OwnableMaster.sol";
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L12)
</details>

### [N-36] Numeric Values Related to Time Lacking Units

For enhanced readability and comprehensibility, numeric values related to time in your Solidity code should employ time units like seconds, minutes, hours, days, or weeks.

Please note that:
- 1 == 1 seconds
- 1 minutes == 60 seconds
- 1 hours == 60 minutes
- 1 days == 24 hours
- 1 weeks == 7 days

More about Time Uints in [Solidity Documentation](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#time-units).

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: contracts/WiseLendingDeclaration.sol

311: uint256 internal constant UPPER_BOUND_MAX_RATE = 300 * PRECISION_FACTOR_E16;
```
[311](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L311)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

106: uint32 internal constant SECONDS_IN_MINUTE = 60;
121: uint256 internal constant GRACE_PEROID = 3600;
```
[106](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L106) | [121](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L121)
</details>

### [N-37] Unused import

The contract contains import statements for libraries or other contracts that are not utilized within the code.
Excessive or unused imports can clutter the codebase, leading to inefficiency and potential confusion.
Consider removing any imports that are not essential to the contract's functionality.

<details>
<summary><i>16 issue instances in 10 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit - SimpleERC20Clone imported but not used
5: import "./SimpleERC20Clone.sol";
/// @audit - IPendle imported but not used
7: import "../../InterfaceHub/IPendle.sol";
```
[5](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L5) | [7](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L7)

```solidity
File: contracts/WiseLendingDeclaration.sol

/// @audit - IAaveHubLite imported but not used
7: import "./InterfaceHub/IAaveHubLite.sol";
```
[7](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L7)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

/// @audit - IERC20 imported but not used
5: import "../../InterfaceHub/IERC20.sol";
/// @audit - IPendle imported but not used
7: import "../../InterfaceHub/IPendle.sol";
/// @audit - IBalancerFlashloan imported but not used
14: import "../../InterfaceHub/IBalancerFlashloan.sol";
```
[5](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L5) | [7](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L7) | [14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L14)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

/// @audit - IPendle imported but not used
10: import "../../InterfaceHub/IPendle.sol";
/// @audit - IPendlePowerFarmTokenFactory imported but not used
14: import "../../InterfaceHub/IPendlePowerFarmTokenFactory.sol";
```
[10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L10) | [14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L14)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

/// @audit - IERC20 imported but not used
5: import "../InterfaceHub/IERC20.sol";
/// @audit - ICurve imported but not used
6: import "../InterfaceHub/ICurve.sol";
```
[5](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L5) | [6](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L6)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

/// @audit - IERC20 imported but not used
6: import "../InterfaceHub/IERC20.sol";
/// @audit - IFeeManager imported but not used
8: import "../InterfaceHub/IFeeManager.sol";
```
[6](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L6) | [8](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L8)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

/// @audit - IPendle imported but not used
16: import "../InterfaceHub/IPendle.sol";
```
[16](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L16)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

/// @audit - OracleLibrary imported but not used
10: import "./Libraries/OracleLibrary.sol";
```
[10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L10)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

/// @audit - IPendle imported but not used
5: import "../InterfaceHub/IPendle.sol";
```
[5](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L5)

```solidity
File: contracts/TransferHub/CallOptionalReturn.sol

/// @audit - IERC20 imported but not used
5: import "../InterfaceHub/IERC20.sol";
```
[5](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L5)
</details>

### [N-38] Use of `override` is unnecessary

In Solidity version 0.8.8 and later, the use of the `override` keyword becomes superfluous when a function is overriding solely from an interface and the function isn't present in multiple base contracts.
Previously, the `override` keyword was required as an explicit indication to the compiler. However, this is no longer the case, and the extraneous use of the keyword can make the code less clean and more verbose.

Solidity documentation on [Function Overriding](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding).

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PositionNFTs.sol

340: function tokenURI(
        uint256 _tokenId
    )
        public
        view
        override
        returns (string memory)
    {
```
[340](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L340)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

182: function tokenURI(
        uint256 _tokenId
    )
        public
        view
        override
        returns (string memory)
    {
```
[182](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L182)
</details>

### [N-39] Setters should prevent re-setting of the same value

Setter functions should include a condition to check if the new value being assigned is different from the current value. This practice prevents redundant state changes and the emission of unnecessary events, ensuring that changes only occur when actual updates to the values are made.

This not only helps to maintain the integrity of the contract's state but also keeps the event logs cleaner and more meaningful by avoiding the recording of identical consecutive values. Such an approach can improve the clarity and efficiency of debugging and data tracking processes.

<details>
<summary><i>46 issue instances in 15 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

/// @audit Missing `_registerState` check before state change
1403: positionLocked[_nftId] = _registerState;
```
[1403](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1403)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

/// @audit Missing `_newMaxFeeFarmETH` check before state change
94: _setLiquidationSettings(
            _baseReward,
            _baseRewardFarm,
            _newMaxFeeETH,
            _newMaxFeeFarmETH
        );
/// @audit Missing `_state` check before state change
987: wasBlacklisted[_tokenAddress] = _state;
/// @audit Missing `_state` check before state change
1002: securityWorker[_entitiy] = _state;
/// @audit Missing `_newMinDepositValue` check before state change
1120: minDepositEthValue = _newMinDepositValue;
```
[94](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L94) | [987](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L987) | [1002](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1002) | [1120](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1120)

```solidity
File: contracts/MainHelper.sol

/// @audit Missing `_poolToken` check before state change
515: _setTimeStamp(
                _poolToken,
                currentTime
            );
/// @audit Missing `_poolToken` check before state change
543: _setTimeStamp(
                _poolToken,
                currentTime
            );
/// @audit Missing `_poolToken` check before state change
555: _setTimeStamp(
                _poolToken,
                currentTime
            );
/// @audit Missing `_poolToken` check before state change
573: _setTimeStamp(
            _poolToken,
            currentTime
        );
```
[515](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L515) | [543](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L543) | [555](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L555) | [573](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L573)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

/// @audit Missing `_state` check before state change
1069: ] = _state;
```
[1069](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1069)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit Missing `_newFee` check before state change
601: mintFee = _newFee;
```
[601](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L601)

```solidity
File: contracts/FeeManager/FeeManager.sol

/// @audit Missing `_percent` check before state change
58: paybackIncentive = _percent;
/// @audit Missing `_underlyingToken` check before state change
73: _setAaveFlag(
            _poolToken,
            _underlyingToken
        );
/// @audit Missing `_underlyingTokens` check before state change
93: _setAaveFlag(
                _poolTokens[i],
                _underlyingTokens[i]
            );
/// @audit Missing `_newFee` check before state change
119: WISE_LENDING.setPoolFee(
            _poolToken,
            _newFee
        );
/// @audit Missing `_newFees` check before state change
151: WISE_LENDING.setPoolFee(
                _poolTokens[i],
                _newFees[i]
            );
/// @audit Missing `_amount` check before state change
521: _setBadDebtPosition(
            _nftId,
            _amount
        );
/// @audit Missing `_feeTokens` check before state change
549: _setAllowedTokens(
                _user,
                _feeTokens[i],
                true
            );
```
[58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L58) | [73](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L73) | [93](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L93) | [119](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L119) | [151](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L151) | [521](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L521) | [549](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L549)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

/// @audit Missing `_pendleMarket` check before state change
308: _setRewardTokens(
            _pendleMarket,
            rewardTokens
        );
/// @audit Missing `_newExchangeIncentive` check before state change
327: exchangeIncentive = _newExchangeIncentive;
/// @audit Missing `_newExchangeIncentive` check before state change
329: emit ChangeExchangeIncentive(
            _newExchangeIncentive
        );
/// @audit Missing `_newFee` check before state change
351: ).changeMintFee(
            _newFee
        );
```
[308](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L308) | [327](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L327) | [329](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L329) | [351](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L351)

```solidity
File: contracts/WrapperHub/AaveHub.sol

/// @audit Missing `_aaveToken` check before state change
51: _setAaveTokenAddress(
            _underlyingAsset,
            _aaveToken
        );
/// @audit Missing `_aaveTokens` check before state change
72: _setAaveTokenAddress(
                _underlyingAssets[i],
                _aaveTokens[i]
            );
/// @audit Missing `_aaveToken` check before state change
673: aaveTokenAddress[_underlyingAsset] = _aaveToken;
```
[51](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L51) | [72](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L72) | [673](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L673)

```solidity
File: contracts/PositionNFTs.sol

/// @audit Missing `_newBaseURI` check before state change
325: baseURI = _newBaseURI;
/// @audit Missing `_newBaseExtension` check before state change
334: baseExtension = _newBaseExtension;
```
[325](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L325) | [334](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L334)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

/// @audit Missing `_baseReward` check before state change
133: baseRewardLiquidation = _baseReward;
/// @audit Missing `_baseReward` check before state change
143: baseRewardLiquidationFarm = _baseRewardFarm;
/// @audit Missing `_newMaxFeeETH` check before state change
161: maxFeeETH = _newMaxFeeETH;
/// @audit Missing `_newMaxFeeFarmETH` check before state change
179: maxFeeFarmETH = _newMaxFeeFarmETH;
```
[133](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L133) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L143) | [161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L161) | [179](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L179)

```solidity
File: contracts/PoolManager.sol

/// @audit Missing `_isFinal` check before state change
39: parametersLocked[_poolToken] = _isFinal;
/// @audit Missing `_maximumDeposit` check before state change
109: maxDepositValueToken[_poolToken] = _maximumDeposit;
/// @audit Missing `_state` check before state change
132: verifiedIsolationPool[_isolationPool] = _state;
```
[39](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L39) | [109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L109) | [132](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L132)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

/// @audit Missing `_amount` check before state change
83: badDebtPosition[_nftId] = _amount;
/// @audit Missing `_nftId` check before state change
174: _setBadDebtPosition(
                _nftId,
                newBadDebt
            );
/// @audit Missing `_state` check before state change
227: allowedTokens[_user][_feeToken] = _state;
/// @audit Missing `_underlyingToken` check before state change
237: underlyingToken[_poolToken] = _underlyingToken;
/// @audit Missing `_nftId` check before state change
283: _updateUserBadDebt(
            _nftId
        );
```
[83](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L83) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L174) | [227](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L227) | [237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L237) | [283](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L283)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

/// @audit Missing `_newMinDeposit` check before state change
69: minDepositEthAmount = _newMinDeposit;
```
[69](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L69)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

/// @audit Missing `_farmContract` check before state change
58: farmContract = _farmContract;
/// @audit Missing `_newBaseURI` check before state change
167: baseURI = _newBaseURI;
/// @audit Missing `_newBaseExtension` check before state change
176: baseExtension = _newBaseExtension;
```
[58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L58) | [167](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L167) | [176](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L176)

```solidity
File: contracts/DerivativeOracles/CustomOracleSetup.sol

/// @audit Missing `_time` check before state change
39: lastUpdateGlobal = _time;
/// @audit Missing `_updateTime` check before state change
49: timeStampByRoundId[_roundId] = _updateTime;
/// @audit Missing `_aggregatorRoundId` check before state change
58: globalRoundId = _aggregatorRoundId;
```
[39](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L39) | [49](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L49) | [58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L58)
</details>

### [N-40] Use Unchecked for Divisions on Constant or Immutable Values

Unsigned divisions on constant or immutable values do not result in overflow.
Therefore, these operations can be marked as unchecked, optimizing gas usage without compromising safety.

For instance, if `a` is an unsigned integer and `b` is a constant or immutable, a / b can be safely rewritten as: unchecked { a / b }

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

287: / lpBalanceController
                    / PRECISION_FACTOR_E18
403: uint256 currentRate = totalLpAssetsToDistribute
            / ONE_WEEK;
```
[287](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L287) | [403](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L403)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

43: / pendleChildToken.totalSupply()
            / PRECISION_FACTOR_E18;
```
[43](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L43)
</details>

### [N-41] Explicit Visibility Recommended in Variable/Function Definitions

In Solidity, variable/function visibility is crucial for controlling access and protecting against unwanted modifications. 
While Solidity functions default to `internal` visibility, it is best practice to explicitly state the visibility for better code readability and avoiding confusion.

The missing visibility could lead to a false sense of security in contract design and potential vulnerabilities.

<details>
<summary><i>11 issue instances in 8 files:</i></summary>

```solidity
File: contracts/WiseLendingDeclaration.sol

168: uint256 immutable FEE_MANAGER_NFT;
```
[168](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L168)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

84: address immutable AAVE_ADDRESS;
88: address immutable AAVE_HUB_ADDRESS;
90: address immutable AAVE_WETH_ADDRESS;
```
[84](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L84) | [88](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L88) | [90](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L90)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

59: mapping(address => CompoundStruct) pendleChildCompoundInfo;
63: bool immutable IS_ETH_MAIN;
```
[59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L59) | [63](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L63)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

222: bool immutable IS_ETH_MAINNET;
```
[222](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L222)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

12: IPowerFarmsNFTs immutable FARMS_NFTS;
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L12)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

139: mapping(address => uint8) _tokenDecimals;
```
[139](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L139)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

47: address immutable PENDLE_MARKET_ADDRESS;
```
[47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L47)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

18: uint8 constant DECIMALS_PRECISION = 18;
```
[18](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L18)
</details>

