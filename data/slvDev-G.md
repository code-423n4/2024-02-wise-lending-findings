## Gas Findings

|    | Issue | Instances | Total Gas Saved |
|----|-------|:---------:|:---------:|
| [G-01] | Optimize Storage with Byte Truncation for Time Related State Variables | 1 | 20000 |
| [G-02] | Optimize by Using Assembly for Low-Level Calls' Return Data | 5 | 795 |
| [G-03] | Unlimited gas consumption risk due to external call recipients | 4 | - |
| [G-04] | Use `assembly` for Efficient Event Emission | 73 | 27740 |
| [G-05] | Avoid Emitting Events Inside Loops | 1 | 400 |
| [G-06] | Use `immutable`s variables directly, instead of cache them in stack | 2 | 0 |
| [G-07] | Assigning `state` variables directly with struct constructors wastes gas | 6 | 497 |
| [G-08] | Assembly: Use scratch space when building emitted events with two data arguments | 2 | 30 |
| [G-09] | Stack variable is only used once | 156 | 468 |
| [G-10] | Consider Using `selfbalance()` Over `address(this).balance` | 2 | - |
| [G-11] | Use `revert()` to gain maximum gas savings | 174 | 8700 |
| [G-12] | Use local variables for emitting | 1 | 100 |
| [G-13] | `x.a = x.a + b` always cheaper than `x.a += b` for Structs | 14 | 70 |
| [G-14] | Initializers can be marked as payable | 1 | - |
| [G-15] | Unused `private` functions should be removed to save deployment gas | 8 | 0 |
| [G-16] | Create variable outside the loop | 13 | 260 |
| [G-17] | Use bytes32 in place of string | 4 | 0 |
| [G-18] | Using `delete` on a `bool` variable is cheaper than assigning it to `false` | 8 | 192 |
| [G-19] | Use `Array.unsafeAccess()` to avoid repeated array length checks | 1 | 2100 |
| [G-20] | Split `revert` checks to save gas | 1 | - |
| [G-21] | Reduce deployment costs by tweaking contracts' metadata | 44 | 466400 |
| [G-22] | Use `storage` instead of `memory` for state structs/arrays | 4 | 8400 |
| [G-23] | Unutilized Named Return Variables Waste Gas Without Optimizer | 23 | - |
| [G-24] | Using `private` rather than `public`, saves gas | 164 | 3608000 |
| [G-25] | Consider using ERC721A instead of ERC721 | 2 | - |
| [G-26] | Refactor duplicated require()/revert() checks to save gas | 13 | - |
| [G-27] | Use Assembly for Efficient Memory Management in Multiple External Calls | 11 | 30000 |
| [G-28] | Optimize `require/revert` Statements with Assembly | 6 | 1800 |
| [G-29] | Cached Global Variables | 3 | 36 |
| [G-30] | Use Cached Contracts for Multiple External Calls | 10 | 1680 |
| [G-31] | Avoid Zero to Non-Zero Storage Writes Where Possible | 23 | 508300 |
| [G-32] | Consider Using `calldata` Instead of `memory` for Function Parameters | 4 | 1200 |
| [G-33] | State Variables Should Be `immutable` Since They Are Only Set in the Constructor | 12 | 25200 |
| [G-34] | Use solady library where possible to save gas | 2 | 2000 |
| [G-35] | Keep Strings Smaller Than 32 Bytes for Storage Efficiency | 7 | 140000 |
| [G-36] | Delete Unused State Variables | 2 | 40000 |
| [G-37] | Mark Functions That Revert For Normal Users As `payable` | 80 | 1680 |
| [G-38] | Inefficient Use of String Constants | 2 | 756 |
| [G-39] | Consider using OZ EnumerateSet in place of nested mappings | 8 | 8000 |
| [G-40] | Declare `immutable` as `private` to save gas | 65 | 195000 |
| [G-41] | Optimize External Calls with Assembly for Memory Efficiency | 37 | 8140 |


## Gas Findings Details

### [G-01] Optimize Storage with Byte Truncation for Time Related State Variables

Certain state variables, particularly timestamps, can be safely stored using `uint32`. 
By optimizing these variables, contracts can utilize storage more efficiently.
This not only results in a reduction in the initial gas costs (due to fewer Gsset operations) but also provides savings in subsequent read and write operations.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit - State variables after packing use 10 storage slots, but can be packed to 9 storage slots.
//	uint256 public lastInteraction;
//	uint256 public mintFee;
//	IPendleController public PENDLE_CONTROLLER;
//	IPendleSy public PENDLE_SY;
//	IPendleMarket public PENDLE_MARKET;
//	uint256 public totalLpAssetsToDistribute;
//	uint256 public underlyingLpAssetsCurrent;
//	address public PENDLE_POWER_FARM_CONTROLLER;
//	uint32 private INITIAL_TIME_STAMP;
//	uint16 public MAX_CARDINALITY;
//	address public UNDERLYING_PENDLE_MARKET;

25: contract PendlePowerFarmToken is SimpleERC20, TransferHelper
```
[25](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L25)
</details>

### [G-02] Optimize by Using Assembly for Low-Level Calls' Return Data

Even second return value from a low-level call is not assigned, it is still copied to memory, leading to additional gas costs.
By employing assembly, you can bypass this memory copying, achieving a 159 gas saving.

<details>
<summary><i>5 issue instances in 4 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

205: (
            bool success
            ,
        ) = curvePool.call{value: 0} (
222: (
            success
            ,
        ) = curveMeta.call{value: 0} (
```
[205](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L205) | [222](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L222)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

78: (bool success, ) = _tokenAggregator.call(
```
[78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L78)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

208: (bool success, ) = payable(_recipient).call{
```
[208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L208)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

24: (
            bool success
            ,
        ) = payable(_recipient).call{
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L24)
</details>

### [G-03] Unlimited gas consumption risk due to external call recipients

When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted.
To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.

<details>
<summary><i>4 issue instances in 3 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

208: ) = curvePool.call{value: 0} (
225: ) = curveMeta.call{value: 0} (
```
[208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L208) | [225](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L225)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

208: (bool success, ) = payable(_recipient).call{
            value: _amount
        }("");
```
[208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L208)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

27: ) = payable(_recipient).call{
            value: _amount
        }("");
```
[27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L27)
</details>

### [G-04] Use `assembly` for Efficient Event Emission

To efficiently emit events, consider utilizing assembly by making use of scratch space and the free memory pointer.
This approach can potentially avoid the costs associated with memory expansion.

However, it's crucial to cache and restore the free memory pointer for safe optimization.
Good examples of such practices can be found in well-optimized [Solady's codebases](https://github.com/Vectorized/solady/blob/main/src/tokens/ERC1155.sol#L167).
Please review your code and consider the potential gas savings of this approach.

<details>
<summary><i>73 issue instances in 12 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

136: emit FundsSolelyWithdrawn(
            _caller,
            _nftId,
            _poolToken,
            _amount,
            block.timestamp
        )
153: emit FundsSolelyDeposited(
            _caller,
            _nftId,
            _poolToken,
            _amount,
            block.timestamp
        )
1450: emit FundsReturned(
            _caller,
            _poolToken,
            _nftId,
            _amount,
            _shares,
            block.timestamp
        )
```
[136](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L136) | [153](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L153) | [1450](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1450)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

1022: emit SecurityShutdown(
            msg.sender,
            block.timestamp
        )
```
[1022](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1022)

```solidity
File: contracts/FeeManager/FeeManager.sol

124: emit PoolFeeChanged(
            _poolToken,
            _newFee,
            block.timestamp
        )
156: emit PoolFeeChanged(
                _poolTokens[i],
                _newFees[i],
                block.timestamp
            )
185: emit IncentiveMasterProposed(
            _proposedIncentiveMaster,
            block.timestamp
        )
204: emit ClaimedOwnershipIncentiveMaster(
            incentiveMaster,
            block.timestamp
        )
222: emit IncentiveIncreasedA(
            _value,
            block.timestamp
        )
240: emit IncentiveIncreasedB(
            _value,
            block.timestamp
        )
276: emit ClaimedIncentivesBulk(
            block.timestamp
        )
297: emit ClaimedIncentives(
            msg.sender,
            _feeToken,
            amount,
            block.timestamp
        )
342: emit IncentiveOwnerAChanged(
            _newOwner,
            block.timestamp
        )
379: emit IncentiveOwnerBChanged(
            _newOwner,
            block.timestamp
        )
402: emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        )
428: emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        )
478: emit PoolTokenRemoved(
                _poolToken,
                block.timestamp
            )
504: emit BadDebtIncreasedLiquidation(
            _amount,
            block.timestamp
        )
526: emit SetBadDebtPosition(
            _nftId,
            _amount,
            block.timestamp
        )
560: emit SetBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        )
593: emit RevokeBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        )
619: emit ClaimedFeesWiseBulk(
            block.timestamp
        )
677: emit ClaimedFeesWise(
            underlyingTokenAddress,
            tokenAmount,
            block.timestamp
        )
716: emit ClaimedFeesBeneficial(
            caller,
            _feeToken,
            _amount,
            block.timestamp
        )
801: emit PayedBackBadDebt(
            _nftId,
            msg.sender,
            _paybackToken,
            _receivingToken,
            paybackAmount,
            block.timestamp
        )
852: emit PayedBackBadDebtFree(
            _nftId,
            msg.sender,
            _paybackToken,
            paybackAmount,
            block.timestamp
        )
```
[124](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L124) | [156](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L156) | [185](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L185) | [204](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L204) | [222](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L222) | [240](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L240) | [276](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L276) | [297](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L297) | [342](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L342) | [379](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L379) | [402](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L402) | [428](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L428) | [478](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L478) | [504](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L504) | [526](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L526) | [560](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L560) | [593](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L593) | [619](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L619) | [677](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L677) | [716](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L716) | [801](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L801) | [852](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L852)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

46: emit WithdrawLp(
            _pendleMarket,
            _to,
            _amount
        )
103: emit ExchangeRewardsForCompounding(
            _pendleMarket,
            _rewardToken,
            _rewardAmount,
            sendingAmount
        )
159: emit ExchangeLpFeesForPendle(
            _pendleMarket,
            _pendleChildShares,
            tokenAmountSend,
            withdrawnAmount
        )
287: emit AddPendleMarket(
            _pendleMarket,
            pendleChild
        )
313: emit UpdateRewardTokens(
            _pendleMarket,
            rewardTokens
        )
329: emit ChangeExchangeIncentive(
            _newExchangeIncentive
        )
355: emit ChangeMintFee(
            _pendleMarket,
            _newFee
        )
421: emit LockPendle(
            _amount,
            expiry,
            newVeBalance,
            _fromInside,
            _sameExpiry,
            block.timestamp
        )
444: emit ClaimArb(
            _accrued,
            _proof,
            block.timestamp
        )
467: emit WithdrawLock(
                amount,
                block.timestamp
            )
491: emit WithdrawLock(
            amount,
            block.timestamp
        )
520: emit IncreaseReservedForCompound(
            _pendleMarket,
            _amounts
        )
593: emit ClaimVoteRewards(
            _amount,
            _merkleProof,
            block.timestamp
        )
```
[46](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L46) | [103](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L103) | [159](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L159) | [287](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L287) | [313](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L313) | [329](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L329) | [355](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L355) | [421](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L421) | [444](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L444) | [467](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L467) | [491](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L491) | [520](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L520) | [593](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L593)

```solidity
File: contracts/WiseCore.sol

77: emit FundsWithdrawnOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            )
86: emit FundsWithdrawn(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            )
156: emit FundsDeposited(
            _caller,
            _nftId,
            _poolToken,
            _amount,
            shareAmount,
            block.timestamp
        )
398: emit FundsBorrowedOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            )
407: emit FundsBorrowed(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            )
```
[77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L77) | [86](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L86) | [156](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L156) | [398](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L398) | [407](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L407)

```solidity
File: contracts/WrapperHub/AaveHub.sol

145: emit IsDepositAave(
            _nftId,
            block.timestamp
        )
190: emit IsDepositAave(
            _nftId,
            block.timestamp
        )
226: emit IsWithdrawAave(
            _nftId,
            block.timestamp
        )
269: emit IsWithdrawAave(
            _nftId,
            block.timestamp
        )
302: emit IsWithdrawAave(
            _nftId,
            block.timestamp
        )
342: emit IsWithdrawAave(
            _nftId,
            block.timestamp
        )
378: emit IsBorrowAave(
            _nftId,
            block.timestamp
        )
421: emit IsBorrowAave(
            _nftId,
            block.timestamp
        )
470: emit IsPaybackAave(
            _nftId,
            block.timestamp
        )
544: emit IsPaybackAave(
            _nftId,
            block.timestamp
        )
603: emit IsPaybackAave(
            _nftId,
            block.timestamp
        )
688: emit SetAaveTokenAddress(
            _underlyingAsset,
            _aaveToken,
            block.timestamp
        )
```
[145](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L145) | [190](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L190) | [226](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L226) | [269](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L269) | [302](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L302) | [342](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L342) | [378](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L378) | [421](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L421) | [470](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L470) | [544](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L544) | [603](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L603) | [688](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L688)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

129: emit ETHReceived(
            msg.value,
            msg.sender
        )
```
[129](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L129)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

96: emit TotalBadDebtIncreased(
            _amount,
            block.timestamp
        )
112: emit TotalBadDebtDecreased(
            _amount,
            block.timestamp
        )
161: emit UpdateBadDebtPosition(
                _nftId,
                0,
                block.timestamp
            )
183: emit UpdateBadDebtPosition(
                _nftId,
                newBadDebt,
                block.timestamp
            )
```
[96](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L96) | [112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L112) | [161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L161) | [183](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L183)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

25: emit ETHReceived(
            msg.value,
            msg.sender
        )
71: emit MinDepositChange(
            _newMinDeposit,
            block.timestamp
        )
90: emit FarmStatus(
            _state,
            block.timestamp
        )
131: emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            _amount,
            block.timestamp
        )
174: emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            msg.value,
            block.timestamp
        )
246: emit FarmExit(
            _keyId,
            wiseLendingNFT,
            _allowedSpread,
            block.timestamp
        )
266: emit ManualPaybackShares(
            _keyId,
            farmingKeys[_keyId],
            _paybackShares,
            block.timestamp
        )
295: emit ManualWithdrawShares(
            _keyId,
            wiseLendingNFT,
            _withdrawShares,
            block.timestamp
        )
```
[25](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L25) | [71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L71) | [90](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L90) | [131](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L131) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L174) | [246](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L246) | [266](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L266) | [295](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L295)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

284: emit RegistrationFarm(
            _nftId,
            block.timestamp
        )
```
[284](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L284)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

169: emit ERC721Received(
            _operator,
            _from,
            _tokenId,
            _data
        )
```
[169](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L169)

```solidity
File: contracts/OwnableMaster.sol

82: emit MasterProposed(
            msg.sender,
            _proposedOwner
        )
110: emit RenouncedOwnership(
            msg.sender
        )
```
[82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L82) | [110](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L110)
</details>

### [G-05] Avoid Emitting Events Inside Loops

Emitting an event inside a loop performs a LOG operation N times, where N is the loop length.
Consider refactoring the code to emit the event only once at the end of the loop.
Gas savings should be multiplied by the average loop length.
This approach can improve the gas efficiency of your contract and make transaction costs more predictable for users.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/FeeManager/FeeManager.sol

156: emit PoolFeeChanged(
                _poolTokens[i],
                _newFees[i],
                block.timestamp
            )
```
[156](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L156)
</details>

### [G-06] Use `immutable`s variables directly, instead of cache them in stack

Caching `immutable`s variable in stack is not necessary, and it will cost more gas.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

30: address flashAsset = WETH_ADDRESS
```
[30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L30)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

125: address poolAddress = WETH_ADDRESS
```
[125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L125)
</details>

### [G-07] Assigning `state` variables directly with struct constructors wastes gas

Utilizing named struct constructors for `state`(only state) variable assignments can lead to inefficient gas usage.
When a struct is initialized with named arguments, the Solidity compiler must first organize these fields in memory, which consumes additional gas.
To optimize gas usage, it is recommended to assign struct fields directly in storage using dot notation.
```solidity
    // stateStruct = TestStruct(20, 20); // 4633 gas
    // stateStruct = TestStruct({ var1: 20, var2: 20 }); // 4633 gas
    // stateStruct.var1 = 20;
    // stateStruct.var2 = 20; // 4562 gas
```

<details>
<summary><i>6 issue instances in 1 files:</i></summary>

```solidity
File: contracts/PoolManager.sol

208: globalPoolData[_params.poolToken] = GlobalPoolEntry({
            totalPool: 0,
            utilization: 0,
            totalBareToken: 0,
            poolFee: 20 * PRECISION_FACTOR_E16
        })
222: borrowRatesData[_params.poolToken] = BorrowRatesEntry(
            startValuePole,
            staticDeltaPole,
            staticMinPole,
            staticMaxPole,
            _params.poolMulFactor
        )
231: borrowPoolData[_params.poolToken] = BorrowPoolEntry({
            allowBorrow: _params.allowBorrow,
            pseudoTotalBorrowAmount: GHOST_AMOUNT,
            totalBorrowShares: GHOST_AMOUNT,
            borrowRate: 0
        })
239: algorithmData[_params.poolToken] = AlgorithmEntry({
            bestPole: startValuePole,
            maxValue: 0,
            previousValue: 0,
            increasePole: false
        })
247: lendingPoolData[_params.poolToken] = LendingPoolEntry({
            pseudoTotalPool: GHOST_AMOUNT,
            totalDepositShares: GHOST_AMOUNT,
            collateralFactor: _params.poolCollFactor
        })
254: timestampsPoolData[_params.poolToken] = TimestampsPoolEntry({
            timeStamp: block.timestamp,
            timeStampScaling: block.timestamp,
            initialTimeStamp: block.timestamp
        })
```
[208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L208) | [222](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L222) | [231](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L231) | [239](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L239) | [247](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L247) | [254](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L254)
</details>

### [G-08] Assembly: Use scratch space when building emitted events with two data arguments

Using the scratch space for more than one, but at most two words worth of data (non-indexed arguments) will save gas over needing Solidity's abi memory expansion used for emitting normally.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

129: emit ETHReceived(
            msg.value,
            msg.sender
        )
```
[129](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L129)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

169: emit ERC721Received(
            _operator,
            _from,
            _tokenId,
            _data
        )
```
[169](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L169)
</details>

### [G-09] Stack variable is only used once

If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the 3 gas the extra stack assignment would spend

<details>
<summary><i>156 issue instances in 27 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

/// @audit - `timeDifference` variable
262: uint256 timeDifference = block.timestamp
            - timestampsPoolData[_poolToken].initialTimeStamp
/// @audit - `shareAmount` variable
407: uint256 shareAmount = _handleDeposit(
            msg.sender,
            _nftId,
            WETH_ADDRESS,
            msg.value
        )
/// @audit - `shareAmount` variable
469: uint256 shareAmount = _handleDeposit(
            msg.sender,
            _nftId,
            _poolToken,
            _amount
        )
/// @audit - `shares` variable
1066: uint256 shares = _handleBorrowExactAmount({
            _nftId: _nftId,
            _poolToken: _poolToken,
            _amount: _amount,
            _onBehalf: true
        })
```
[262](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L262) | [407](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L407) | [469](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L469) | [1066](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1066)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

/// @audit - `lenBorrow` variable
572: uint256 lenBorrow = WISE_LENDING.getPositionBorrowTokenLength(
            _nftId
        )
/// @audit - `lenDeposit` variable
576: uint256 lenDeposit = WISE_LENDING.getPositionLendingTokenLength(
            _nftId
        )
/// @audit - `lenDeposit` variable
702: uint256 lenDeposit = WISE_LENDING.getPositionLendingTokenLength(
            _nftId
        )
/// @audit - `lenBorrow` variable
728: uint256 lenBorrow = WISE_LENDING.getPositionBorrowTokenLength(
            _nftId
        )
/// @audit - `term` variable
877: uint256 term = _overallETHCollateralsWeighted(_nftId, _interval)
            * BORROW_PERCENTAGE_CAP
            / PRECISION_FACTOR_E18
/// @audit - `borrowETH` variable
881: uint256 borrowETH = term
            - _overallETHBorrow(
                _nftId,
                _interval
            )
/// @audit - `borrowShares` variable
915: uint256 borrowShares = WISE_LENDING.getPositionBorrowShares(
            _nftId,
            _poolToken
        )
/// @audit - `updatedPseudo` variable
928: uint256 updatedPseudo = _getUpdatedPseudoBorrow(
            _poolToken,
            _interval
        )
/// @audit - `lendingShares` variable
952: uint256 lendingShares = WISE_LENDING.getPositionLendingShares(
            _nftId,
            _poolToken
        )
/// @audit - `updatedPseudo` variable
965: uint256 updatedPseudo = _getUpdatedPseudoPool(
            _poolToken,
            _interval
        )
```
[572](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L572) | [576](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L576) | [702](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L702) | [728](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L728) | [877](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L877) | [881](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L881) | [915](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L915) | [928](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L928) | [952](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L952) | [965](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L965)

```solidity
File: contracts/MainHelper.sol

/// @audit - `product` variable
122: uint256 product = _shares
            * borrowPoolData[_poolToken].pseudoTotalBorrowAmount
/// @audit - `totalBorrowShares` variable
125: uint256 totalBorrowShares = borrowPoolData[_poolToken].totalBorrowShares
/// @audit - `timeDifference` variable
357: uint256 timeDifference = block.timestamp
            - timestampsPoolData[_poolToken].timeStamp
/// @audit - `l` variable
426: uint256 l = _tokens.length
/// @audit - `l` variable
479: uint256 l = _tokens.length
/// @audit - `baseDivider` variable
859: uint256 baseDivider = pole
            * (pole - utilization)
/// @audit - `delta` variable
1103: uint256 delta = borrowData.deltaPole
            * (block.timestamp - timestampsPoolData[_poolToken].timeStampScaling)
/// @audit - `setValue` variable
1109: uint256 setValue = sum > borrowData.maxPole
            ? borrowData.maxPole
            : sum
/// @audit - `setValue` variable
1139: uint256 setValue = sub < minValue
            ? minValue
            : sub
```
[122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L122) | [125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L125) | [357](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L357) | [426](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L426) | [479](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L479) | [859](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L859) | [1103](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1103) | [1109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1109) | [1139](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1139)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

/// @audit - `l` variable
28: uint256 l = WISE_LENDING.getPositionLendingTokenLength(
            _nftId
        )
/// @audit - `l` variable
75: uint256 l = WISE_LENDING.getPositionLendingTokenLength(
            _nftId
        )
/// @audit - `l` variable
117: uint256 l = WISE_LENDING.getPositionLendingTokenLength(
            _nftId
        )
/// @audit - `l` variable
156: uint256 l = WISE_LENDING.getPositionLendingTokenLength(
            _nftId
        )
/// @audit - `currentTotalLendingShares` variable
299: uint256 currentTotalLendingShares = WISE_LENDING.getTotalDepositShares(
            _poolToken
        )
/// @audit - `updatedPseudo` variable
303: uint256 updatedPseudo = _getUpdatedPseudoPool(
            _poolToken,
            _interval
        )
/// @audit - `updatedToken` variable
308: uint256 updatedToken = lendingShares
            * updatedPseudo
            / currentTotalLendingShares
/// @audit - `l` variable
368: uint256 l = WISE_LENDING.getPositionBorrowTokenLength(
            _nftId
        )
/// @audit - `l` variable
404: uint256 l = WISE_LENDING.getPositionBorrowTokenLength(
            _nftId
        )
/// @audit - `l` variable
440: uint256 l = WISE_LENDING.getPositionBorrowTokenLength(
            _nftId
        )
/// @audit - `l` variable
497: uint256 l = WISE_LENDING.getPositionBorrowTokenLength(
            _nftId
        )
/// @audit - `currentPseudo` variable
538: uint256 currentPseudo = WISE_LENDING.getPseudoTotalBorrowAmount(
            _poolToken
        )
/// @audit - `currentPseudo` variable
562: uint256 currentPseudo = WISE_LENDING.getPseudoTotalPool(
            _poolToken
        )
/// @audit - `borrowPoolData` variable
586: BorrowPoolEntry memory borrowPoolData = WISE_LENDING.borrowPoolData(
            _poolToken
        )
/// @audit - `timeInterval` variable
590: uint256 timeInterval = _interval
            + block.timestamp
            - WISE_LENDING.getTimeStamp(_poolToken)
/// @audit - `rate` variable
594: uint256 rate = timeInterval
            * borrowPoolData.borrowRate
            * WISE_LENDING.getPseudoTotalBorrowAmount(_poolToken)
            / PRECISION_FACTOR_E18
            / ONE_YEAR
/// @audit - `currentTotalBorrowShares` variable
628: uint256 currentTotalBorrowShares = WISE_LENDING.getTotalBorrowShares(
            _poolToken
        )
/// @audit - `updatesPseudo` variable
632: uint256 updatesPseudo = _getUpdatedPseudoBorrow(
            _poolToken,
            _intervall
        )
/// @audit - `updatedToken` variable
637: uint256 updatedToken = borrowShares
            * updatesPseudo
            / currentTotalBorrowShares
/// @audit - `feeETH` variable
771: uint256 feeETH = _checkMaxFee(
            _paybackETH,
            _baseRewardLiquidation,
            _maxFeeETH
        )
/// @audit - `numerator` variable
777: uint256 numerator = (feeETH + _paybackETH)
            * PRECISION_FACTOR_E18
/// @audit - `denominator` variable
780: uint256 denominator = getFullCollateralETH(
            _nftId,
            _receiveToken
        )
/// @audit - `overallCollateral` variable
822: uint256 overallCollateral = _powerFarm == true
            ? overallETHCollateralsBare(_nftId)
            : overallETHCollateralsWeighted(_nftId)
/// @audit - `maxShares` variable
891: uint256 maxShares = checkBadDebtThreshold(_borrowETHTotal, _unweightedCollateralETH)
            ? totalSharesUser
            : totalSharesUser * MAX_LIQUIDATION_50 / PRECISION_FACTOR_E18
/// @audit - `adjustedRate` variable
1014: uint256 adjustedRate = getBorrowRate(_poolToken)
            * (PRECISION_FACTOR_E18 - WISE_LENDING.globalPoolData(_poolToken).poolFee)
            / PRECISION_FACTOR_E18
/// @audit - `term` variable
1038: uint256 term = _overallETHBorrow(_nftId, _interval)
            * PRECISION_FACTOR_E18
            / BORROW_PERCENTAGE_CAP
/// @audit - `withdrawETH` variable
1042: uint256 withdrawETH = PRECISION_FACTOR_E18
            * (_overallETHCollateralsWeighted(_nftId, _interval) - term)
            / WISE_LENDING.lendingPoolData(_poolToken).collateralFactor
/// @audit - `len` variable
1063: uint256 len = FEE_MANAGER.getPoolTokenAddressesLength()
```
[28](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L28) | [75](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L75) | [117](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L117) | [156](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L156) | [299](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L299) | [303](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L303) | [308](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L308) | [368](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L368) | [404](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L404) | [440](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L440) | [497](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L497) | [538](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L538) | [562](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L562) | [586](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L586) | [590](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L590) | [594](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L594) | [628](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L628) | [632](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L632) | [637](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L637) | [771](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L771) | [777](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L777) | [780](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L780) | [822](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L822) | [891](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L891) | [1014](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1014) | [1038](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1038) | [1042](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1042) | [1063](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1063)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit - `sharePriceBefore` variable
89: uint256 sharePriceBefore = _getSharePrice()
/// @audit - `timeDifference` variable
120: uint256 timeDifference = block.timestamp
            - INITIAL_TIME_STAMP
/// @audit - `maximum` variable
123: uint256 maximum = timeDifference
            * RESTRICTION_FACTOR
            + PRECISION_FACTOR_E18
/// @audit - `l` variable
202: uint256 l = rewardsOutsideArray.length
/// @audit - `scaleNecessary` variable
281: bool scaleNecessary = totalLpAssetsCurrent < lpBalanceController
/// @audit - `currentRate` variable
403: uint256 currentRate = totalLpAssetsToDistribute
            / ONE_WEEK
```
[89](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L89) | [120](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L120) | [123](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L123) | [202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L202) | [281](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L281) | [403](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L403)

```solidity
File: contracts/FeeManager/FeeManager.sol

/// @audit - `l` variable
90: uint256 l = _poolTokens.length
/// @audit - `l` variable
143: uint256 l = _poolTokens.length
/// @audit - `l` variable
255: uint256 l = getPoolTokenAddressesLength()
/// @audit - `l` variable
546: uint256 l = _feeTokens.length
/// @audit - `l` variable
579: uint256 l = _feeTokens.length
/// @audit - `l` variable
607: uint256 l = getPoolTokenAddressesLength()
/// @audit - `l` variable
902: uint256 l = poolTokenAddresses.length
```
[90](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L90) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L143) | [255](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L255) | [546](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L546) | [579](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L579) | [607](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L607) | [902](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L902)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

/// @audit - `flashAsset` variable
30: address flashAsset = WETH_ADDRESS
/// @audit - `ethValueBefore` variable
157: uint256 ethValueBefore = _getTokensInETH(
            PENDLE_CHILD,
            withdrawnLpsAmount
        )
/// @audit - `tokenOut` variable
174: address tokenOut = block.chainid == 1
            ? ST_ETH_ADDRESS
            : ENTRY_ASSET
/// @audit - `reverseAllowedSpread` variable
188: uint256 reverseAllowedSpread = 2
            * PRECISION_FACTOR_E18
            - _allowedSpread
/// @audit - `ethValueAfter` variable
204: uint256 ethValueAfter = _getTokensInETH(
            WETH_ADDRESS,
            ethAmount
        )
            * _allowedSpread
            / PRECISION_FACTOR_E18
/// @audit - `reverseAllowedSpread` variable
423: uint256 reverseAllowedSpread = 2
            * PRECISION_FACTOR_E18
            - _allowedSpread
/// @audit - `ethValueBefore` variable
485: uint256 ethValueBefore = _getTokensInETH(
            ENTRY_ASSET,
            _depositAmount
        )
/// @audit - `ethValueAfter` variable
497: uint256 ethValueAfter = _getTokensInETH(
            PENDLE_CHILD,
            receivedShares
        )
            * _allowedSpread
            / PRECISION_FACTOR_E18
/// @audit - `cutoffShares` variable
584: uint256 cutoffShares = isAave[_nftId] == true
            ? _getPositionBorrowSharesAave(_nftId)
                * FIVTY_PERCENT
                / PRECISION_FACTOR_E18
            : _getPositionBorrowShares(_nftId)
                * FIVTY_PERCENT
                / PRECISION_FACTOR_E18
```
[30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L30) | [157](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L157) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L174) | [188](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L188) | [204](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L204) | [423](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L423) | [485](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L485) | [497](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L497) | [584](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L584)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

/// @audit - `uniTwapPoolInfoStruct` variable
49: UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[
            _tokenAddress
        ]
/// @audit - `maxAnswer` variable
94: int192 maxAnswer = _tokenAggregator.maxAnswer()
/// @audit - `minAnswer` variable
95: int192 minAnswer = _tokenAggregator.minAnswer()
/// @audit - `uniTwapPoolInfoStruct` variable
138: UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[
            _tokenAddress
        ]
/// @audit - `relativeDifference` variable
163: uint256 relativeDifference = _getRelativeDifference(
                answer,
                fetchTwapValue
            )
/// @audit - `firstQuote` variable
472: uint256 firstQuote = OracleLibrary.getQuoteAtTick(
            _getAverageTick(
                _uniTwapPoolInfo.oracle
            ),
            _getOneUnit(
                partnerInfo.partnerTokenAddress
            ),
            partnerInfo.partnerTokenAddress,
            WETH_ADDRESS
        )
/// @audit - `secondQuote` variable
483: uint256 secondQuote = OracleLibrary.getQuoteAtTick(
            _getAverageTick(
                partnerInfo.partnerOracleAddress
            ),
            _getOneUnit(
                _tokenAddress
            ),
            _tokenAddress,
            partnerInfo.partnerTokenAddress
        )
/// @audit - `timeSinceUp` variable
541: uint256 timeSinceUp = block.timestamp
            - startedAt
/// @audit - `latestRoundId` variable
570: uint80 latestRoundId = getLatestRoundId(
            _tokenAddress
        )
```
[49](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L49) | [94](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L94) | [95](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L95) | [138](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L138) | [163](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L163) | [472](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L472) | [483](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L483) | [541](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L541) | [570](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L570)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

/// @audit - `length` variable
509: uint256 length = childInfo.rewardTokens.length
/// @audit - `length` variable
551: uint256 length = pendleChildCompoundInfo[
            _pendleMarket
        ].rewardTokens.length
/// @audit - `len` variable
624: uint256 len = _weights.length
```
[509](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L509) | [551](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L551) | [624](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L624)

```solidity
File: contracts/WiseCore.sol

/// @audit - `state` variable
301: bool state = maxDepositValueToken[_poolToken]
            < globalPoolData[_poolToken].totalBareToken
            + lendingPoolData[_poolToken].pseudoTotalPool
            + _amount
/// @audit - `product` variable
436: uint256 product = _percentLiquidation
            * pureCollateralAmount[_nftId][_poolToken]
/// @audit - `product` variable
469: uint256 product = _percentWishCollateral
            * userLendingData[_nftId][_poolToken].shares
```
[301](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L301) | [436](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L436) | [469](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L469)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

/// @audit - `length` variable
293: uint256 length = _uniPools.length
/// @audit - `l` variable
388: uint256 l = _tokenAddresses.length
/// @audit - `l` variable
507: uint256 l = _tokenAddresses.length
```
[293](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L293) | [388](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L388) | [507](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L507)

```solidity
File: contracts/WrapperHub/AaveHub.sol

/// @audit - `lendingShares` variable
139: uint256 lendingShares = _wrapDepositExactAmount(
            _nftId,
            _underlyingAsset,
            _amount
        )
/// @audit - `lendingShares` variable
184: uint256 lendingShares = _wrapDepositExactAmount(
            _nftId,
            WETH_ADDRESS,
            msg.value
        )
/// @audit - `withdrawAmount` variable
295: uint256 withdrawAmount = _wrapWithdrawExactShares(
            _nftId,
            _underlyingAsset,
            msg.sender,
            _shareAmount
        )
/// @audit - `borrowShares` variable
371: uint256 borrowShares = _wrapBorrowExactAmount(
            _nftId,
            _underlyingAsset,
            msg.sender,
            _borrowAmount
        )
/// @audit - `borrowShares` variable
405: uint256 borrowShares = _wrapBorrowExactAmount(
            _nftId,
            WETH_ADDRESS,
            address(this),
            _borrowAmount
        )
/// @audit - `aaveToken` variable
447: address aaveToken = aaveTokenAddress[
            _underlyingAsset
        ]
/// @audit - `actualAmountDeposit` variable
458: uint256 actualAmountDeposit = _wrapAaveReturnValueDeposit(
            _underlyingAsset,
            _paybackAmount,
            address(this)
        )
/// @audit - `borrowSharesReduction` variable
464: uint256 borrowSharesReduction = WISE_LENDING.paybackExactAmount(
            _nftId,
            aaveToken,
            actualAmountDeposit
        )
/// @audit - `userBorrowShares` variable
498: uint256 userBorrowShares = WISE_LENDING.getPositionBorrowShares(
            _nftId,
            aaveWrappedETH
        )
/// @audit - `maxPaybackAmount` variable
507: uint256 maxPaybackAmount = WISE_LENDING.paybackAmount(
            aaveWrappedETH,
            userBorrowShares
        )
/// @audit - `actualAmountDeposit` variable
525: uint256 actualAmountDeposit = _wrapAaveReturnValueDeposit(
            WETH_ADDRESS,
            paybackAmount,
            address(this)
        )
/// @audit - `borrowSharesReduction` variable
531: uint256 borrowSharesReduction = WISE_LENDING.paybackExactAmount(
            _nftId,
            aaveWrappedETH,
            actualAmountDeposit
        )
/// @audit - `lendingRate` variable
646: uint256 lendingRate = WISE_SECURITY.getLendingRate(
            aToken
        )
/// @audit - `aaveRate` variable
650: uint256 aaveRate = getAavePoolAPY(
            _underlyingAsset
        )
/// @audit - `utilization` variable
654: uint256 utilization = WISE_LENDING.globalPoolData(
            aToken
        ).utilization
```
[139](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L139) | [184](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L184) | [295](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L295) | [371](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L371) | [405](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L405) | [447](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L447) | [458](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L458) | [464](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L464) | [498](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L498) | [507](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L507) | [525](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L525) | [531](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L531) | [646](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L646) | [650](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L650) | [654](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L654)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

/// @audit - `length` variable
276: uint256 length = activePendleMarkets.length
```
[276](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L276)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

/// @audit - `actualDepositAmount` variable
64: uint256 actualDepositAmount = _wrapAaveReturnValueDeposit(
            _underlyingAsset,
            _depositAmount,
            address(this)
        )
/// @audit - `lendingShares` variable
74: uint256 lendingShares = WISE_LENDING.depositExactAmount(
            _nftId,
            aaveTokenAddress[_underlyingAsset],
            actualDepositAmount
        )
/// @audit - `withdrawnShares` variable
95: uint256 withdrawnShares = WISE_LENDING.withdrawOnBehalfExactAmount(
            _nftId,
            aaveTokenAddress[_underlyingAsset],
            _withdrawAmount
        )
/// @audit - `actualAmount` variable
101: uint256 actualAmount = AAVE.withdraw(
            _underlyingAsset,
            _withdrawAmount,
            _underlyingAssetRecipient
        )
/// @audit - `aaveToken` variable
122: address aaveToken = aaveTokenAddress[
            _underlyingAsset
        ]
/// @audit - `borrowShares` variable
150: uint256 borrowShares = WISE_LENDING.borrowOnBehalfExactAmount(
            _nftId,
            aaveTokenAddress[_underlyingAsset],
            _borrowAmount
        )
/// @audit - `balanceBefore` variable
177: uint256 balanceBefore = token.balanceOf(
            address(this)
        )
/// @audit - `balanceAfter` variable
188: uint256 balanceAfter = token.balanceOf(
            address(this)
        )
/// @audit - `l` variable
250: uint256 l = WISE_LENDING.getPositionLendingTokenLength(
            _nftId
        )
/// @audit - `l` variable
286: uint256 l = WISE_LENDING.getPositionBorrowTokenLength(
            _nftId
        )
```
[64](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L64) | [74](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L74) | [95](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L95) | [101](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L101) | [122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L122) | [150](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L150) | [177](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L177) | [188](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L188) | [250](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L250) | [286](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L286)

```solidity
File: contracts/PositionNFTs.sol

/// @audit - `k` variable
394: uint256 k = length
```
[394](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L394)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

/// @audit - `lendingMaster` variable
77: address lendingMaster = WISE_LENDING.master()
/// @audit - `feeManagerContract` variable
81: FeeManager feeManagerContract = new FeeManager(
            lendingMaster,
            IAaveHubWiseSecurity(AAVE_HUB).AAVE_ADDRESS(),
            _wiseLendingAddress,
            oracleHubAddress,
            address(this),
            positionNFTAddress
        )
/// @audit - `maxFee` variable
146: uint256 maxFee = IS_ETH_MAINNET == true
            ? LIQUIDATION_FEE_MAX_ETH
            : LIQUIDATION_FEE_MAX_NON_ETH
/// @audit - `minFee` variable
150: uint256 minFee = IS_ETH_MAINNET == true
            ? LIQUIDATION_FEE_MIN_ETH
            : LIQUIDATION_FEE_MIN_NON_ETH
/// @audit - `maxFeeFarm` variable
164: uint256 maxFeeFarm = IS_ETH_MAINNET == true
            ? LIQUIDATION_FEE_MAX_ETH
            : LIQUIDATION_FEE_MAX_NON_ETH
/// @audit - `minFeeFarm` variable
168: uint256 minFeeFarm = IS_ETH_MAINNET == true
            ? LIQUIDATION_FEE_MIN_ETH
            : LIQUIDATION_FEE_MIN_NON_ETH
```
[77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L77) | [81](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L81) | [146](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L146) | [150](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L150) | [164](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L164) | [168](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L168)

```solidity
File: contracts/PoolManager.sol

/// @audit - `staticDeltaPole` variable
73: uint256 staticDeltaPole = _getDeltaPole(
            staticMaxPole,
            staticMinPole
        )
/// @audit - `staticDeltaPole` variable
201: uint256 staticDeltaPole = _getDeltaPole(
            staticMaxPole,
            staticMinPole
        )
```
[73](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L73) | [201](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L201)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

/// @audit - `l` variable
21: uint256 l = WISE_LENDING.getPositionBorrowTokenLength(
            _nftId
        )
/// @audit - `currentAddress` variable
27: address currentAddress = WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
/// @audit - `l` variable
53: uint256 l = WISE_LENDING.getPositionLendingTokenLength(
            _nftId
        )
/// @audit - `currentAddress` variable
59: address currentAddress = WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
/// @audit - `increasedAmount` variable
253: uint256 increasedAmount = _paybackAmount
            * (PRECISION_FACTOR_E18 + paybackIncentive)
            / PRECISION_FACTOR_E18
```
[21](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L21) | [27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L27) | [53](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L53) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L59) | [253](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L253)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

/// @audit - `borrowSharesAave` variable
192: uint256 borrowSharesAave = _getPositionBorrowSharesAave(
            _nftId
        )
/// @audit - `tokenValueAaveEth` variable
217: uint256 tokenValueAaveEth = _getTokensInETH(
            AAVE_WETH_ADDRESS,
            borrowTokenAmountAave
        )
/// @audit - `leveragedAmount` variable
308: uint256 leveragedAmount = getLeverageAmount(
            _initialAmount,
            _leverage
        )
/// @audit - `flashloanAmount` variable
313: uint256 flashloanAmount = leveragedAmount
            - _initialAmount
/// @audit - `newBorrowRate` variable
316: uint256 newBorrowRate = _getNewBorrowRate(
            flashloanAmount
        )
/// @audit - `netAPY` variable
330: uint256 netAPY = isPositive == true
            ? leveragedPositivAPY - leveragedNegativeAPY
            : leveragedNegativeAPY - leveragedPositivAPY
/// @audit - `mulFactor` variable
372: uint256 mulFactor = WISE_LENDING.borrowRatesData(
            ENTRY_ASSET
        ).multiplicativeFactor
/// @audit - `baseDivider` variable
376: uint256 baseDivider = pole
            * (pole - newUtilization)
/// @audit - `borrowShares` variable
396: uint256 borrowShares = isAave[_nftId]
            ? _getPositionBorrowSharesAave(
                _nftId
            )
            : _getPositionBorrowShares(
                _nftId
        )
/// @audit - `equivETH` variable
423: uint256 equivETH = _getTokensInETH(
            ENTRY_ASSET,
            _amount
        )
```
[192](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L192) | [217](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L217) | [308](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L308) | [313](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L313) | [316](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L316) | [330](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L330) | [372](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L372) | [376](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L376) | [396](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L396) | [423](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L423)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

/// @audit - `len` variable
18: uint256 len = _array.length
/// @audit - `startTime` variable
39: uint128 startTime = uint128((block.timestamp / WEEK)
            * WEEK)
```
[18](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L18) | [39](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L39)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

/// @audit - `borrowShares` variable
64: uint256 borrowShares = isAave[_nftId]
            ? _getPositionBorrowSharesAave(
                _nftId
            )
            : _getPositionBorrowShares(
                _nftId
            )
/// @audit - `paybackAmount` variable
131: uint256 paybackAmount = WISE_LENDING.paybackAmount(
            poolAddress,
            _paybackShares
        )
/// @audit - `borrowShares` variable
238: uint256 borrowShares = _isAave == false
            ? _getPositionBorrowShares(
                _nftId
            )
            : _getPositionBorrowSharesAave(
                _nftId
            )
/// @audit - `borrowTokenAmount` variable
246: uint256 borrowTokenAmount = _isAave == false
            ? _getPositionBorrowTokenAmount(
                _nftId
            )
            : _getPositionBorrowTokenAmountAave(
                _nftId
            )
```
[64](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L64) | [131](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L131) | [238](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L238) | [246](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L246)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

/// @audit - `farm` variable
116: IFarmContract farm = IFarmContract(
            farmContract
        )
/// @audit - `k` variable
236: uint256 k = length
```
[116](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L116) | [236](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L236)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

/// @audit - `lpRate` variable
112: uint256 lpRate = _getLpToAssetRateWrapper(
            IPMarket(PENDLE_MARKET_ADDRESS),
            TWAP_DURATION
        )
```
[112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L112)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

/// @audit - `ptToAssetRate` variable
117: uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
            PENDLE_MARKET_ADDRESS,
            TWAP_DURATION
        )
```
[117](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L117)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

/// @audit - `ptToAssetRate` variable
98: uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
            PENDLE_MARKET_ADDRESS,
            DURATION
        )
```
[98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L98)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

/// @audit - `implementation` variable
22: PendlePowerFarmToken implementation = new PendlePowerFarmToken{
            salt: keccak256(
                abi.encodePacked(
                    _pendlePowerFarmController
                )
            )
        }()
/// @audit - `salt` variable
65: bytes32 salt = keccak256(
            abi.encodePacked(
                _underlyingPendleMarket
            )
        )
/// @audit - `targetBytes` variable
71: bytes20 targetBytes = bytes20(
            IMPLEMENTATION_TARGET
        )
```
[22](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L22) | [65](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L65) | [71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L71)

```solidity
File: contracts/TransferHub/CallOptionalReturn.sol

/// @audit - `results` variable
26: bool results = returndata.length == 0 || abi.decode(
            returndata,
            (bool)
        )
```
[26](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L26)
</details>

### [G-10] Consider Using `selfbalance()` Over `address(this).balance`

The `selfbalance()` function from Yul can be more gas-efficient than using `address(this).balance` in certain scenarios.
Although the Solidity compiler is sometimes optimized enough to handle this, manually switching to `selfbalance()` could yield gas savings.

Note: Always thoroughly test both approaches to confirm the actual gas savings.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/WrapperHub/AaveHelper.sol

202: address(this).balance
```
[202](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L202)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

18: address(this).balance
```
[18](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L18)
</details>

### [G-11] Use `revert()` to gain maximum gas savings

If you dont need Error messages, or you want gain maximum gas savings - `revert()` is a cheapest way to revert transaction in terms of gas.
```solidity
    revert(); // 117 gas 
    require(false); // 132 gas
    revert CustomError(); // 157 gas
    assert(false); // 164 gas
    revert("Custom Error"); // 406 gas
    require(false, "Custom Error"); // 421 gas
```


<details>
<summary><i>174 issue instances in 35 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

39: revert NotWiseLendingSecurity()
249: revert OpenBorrowPosition()
287: revert OpenBorrowPosition()
344: revert ChainlinkDead()
370: revert OpenBorrowPosition()
1015: revert NotAllowedEntity()
1105: revert DepositAmountTooSmall()
1118: revert NoValue()
```
[39](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L39) | [249](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L249) | [287](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L287) | [344](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L344) | [370](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L370) | [1015](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1015) | [1105](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1105) | [1118](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1118)

```solidity
File: contracts/MainHelper.sol

224: revert InvalidAction()
245: revert NotPowerFarm()
254: revert InvalidCaller()
266: revert LiquidatorIsInPowerFarm()
270: revert InvalidLiquidator()
274: revert InvalidLiquidator()
638: revert TooManyTokens()
```
[224](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L224) | [245](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L245) | [254](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L254) | [266](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L266) | [270](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L270) | [274](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L274) | [638](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L638)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

679: revert NotAllowedToBorrow()
708: revert PositionLockedWiseSecurity()
802: revert ResultsInBadDebt()
850: revert NotAllowedWiseSecurity()
867: revert LiquidationDenied()
899: revert TooManyShares()
977: revert NotOwner()
1087: revert TokenBlackListed()
1103: revert SecuritySwapFailed()
```
[679](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L679) | [708](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L708) | [802](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L802) | [850](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L850) | [867](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L867) | [899](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L899) | [977](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L977) | [1087](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1087) | [1103](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1103)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

77: revert NotController()
108: revert InvalidSharePrice()
128: revert InvalidSharePriceGrowth()
509: revert ZeroAmount()
540: revert ZeroAmount()
549: revert NotEnoughLpAssetsTransferred()
560: revert ZeroFee()
564: revert TooMuchFee()
599: revert FeeTooHigh()
616: revert ZeroAmount()
620: revert InsufficientShares()
655: revert ZeroAmount()
664: revert NotEnoughShares()
692: revert AlreadyInitialized()
700: revert MarketExpired()
```
[77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L77) | [108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L108) | [128](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L128) | [509](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L509) | [540](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L540) | [549](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L549) | [560](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L560) | [564](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L564) | [599](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L599) | [616](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L616) | [620](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L620) | [655](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L655) | [664](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L664) | [692](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L692) | [700](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L700)

```solidity
File: contracts/FeeManager/FeeManager.sol

180: revert ZeroAddress()
198: revert NotAllowed()
292: revert NoIncentive()
321: revert NotAllowed()
325: revert NotAllowed()
329: revert NotAllowed()
358: revert NotAllowed()
362: revert NotAllowed()
366: revert NotAllowed()
419: revert PoolAlreadyAdded()
450: revert PoolNotPresent()
486: revert PoolNotPresent()
698: revert ExistingBadDebt()
702: revert NotAllowed()
754: revert PoolNotActive()
758: revert PoolNotActive()
833: revert PoolNotActive()
```
[180](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L180) | [198](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L198) | [292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L292) | [321](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L321) | [325](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L325) | [329](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L329) | [358](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L358) | [362](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L362) | [366](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L366) | [419](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L419) | [450](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L450) | [486](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L486) | [698](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L698) | [702](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L702) | [754](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L754) | [758](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L758) | [833](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L833)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

68: revert AccessDenied()
74: revert InvalidParam()
78: revert NotBalancerVault()
212: revert TooMuchValueLost()
261: revert WrongChainId()
505: revert TooMuchValueLost()
521: revert DebtRatioTooHigh()
572: revert DebtRatioTooLow()
593: revert TooMuchShares()
```
[68](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L68) | [74](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L74) | [78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L78) | [212](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L212) | [261](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L261) | [505](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L505) | [521](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L521) | [572](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L572) | [593](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L593)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

20: revert OracleAlreadySet()
42: revert AggregatorAlreadySet()
46: revert FunctionDoesntExist()
54: revert AggregatorNotNecessary()
98: revert OracleIsDead()
242: revert OraclesDeviate()
271: revert OracleIsDead()
312: revert ZeroAddressNotAllowed()
316: revert TokenAddressMismatch()
332: revert PoolDoesNotExist()
336: revert PoolAddressMismatch()
351: revert ChainLinkOracleNotSet()
366: revert TwapOracleAlreadySet()
567: revert HeartBeatNotSet()
```
[20](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L20) | [42](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L42) | [46](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L46) | [54](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L54) | [98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L98) | [242](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L242) | [271](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L271) | [312](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L312) | [316](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L316) | [332](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L332) | [336](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L336) | [351](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L351) | [366](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L366) | [567](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L567)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

72: revert NotEnoughCompound()
125: revert ZeroShares()
183: revert WrongAddress()
195: revert NothingToSkim()
221: revert AlreadySet()
346: revert WrongAddress()
385: revert LockTimeTooShort()
476: revert NotExpired()
620: revert InvalidLength()
636: revert InvalidWeightSum()
```
[72](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L72) | [125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L125) | [183](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L183) | [195](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L195) | [221](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L221) | [346](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L346) | [385](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L385) | [476](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L476) | [620](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L620) | [636](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L636)

```solidity
File: contracts/WiseCore.sol

208: revert PositionLocked()
247: revert DeadOracle()
286: revert InvalidAction()
307: revert InvalidAction()
```
[208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L208) | [247](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L247) | [286](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L286) | [307](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L307)

```solidity
File: contracts/WiseLendingDeclaration.sol

120: revert NoValue()
124: revert NoValue()
147: revert InvalidAction()
262: revert InvalidCaller()
```
[120](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L120) | [124](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L124) | [147](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L147) | [262](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L262)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

77: revert OracleIsDead()
```
[77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L77)

```solidity
File: contracts/WrapperHub/AaveHub.sol

671: revert AlreadySet()
```
[671](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L671)

```solidity
File: contracts/WiseLowLevelHelper.sol

22: revert InvalidCaller()
33: revert InvalidAction()
355: revert InvalidAction()
359: revert InvalidAction()
454: revert ValueIsZero()
465: revert ValueNotZero()
```
[22](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L22) | [33](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L33) | [355](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L355) | [359](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L359) | [454](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L454) | [465](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L465)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

244: revert CollateralFactorTooHigh()
366: revert Deactivated()
```
[244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L244) | [366](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L366)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

241: revert NotArbitrum()
252: revert NotAllowed()
266: revert WrongAddress()
```
[241](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L241) | [252](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L252) | [266](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L266)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

30: revert InvalidToken()
39: revert InvalidAction()
43: revert InvalidAction()
71: revert InvalidAction()
203: revert InvalidValue()
215: revert FailedInnerCall()
```
[30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L30) | [39](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L39) | [43](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L43) | [71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L71) | [203](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L203) | [215](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L215)

```solidity
File: contracts/PositionNFTs.sol

348: require(
            _exists(_tokenId) == true,
            "PositionNFTs: WRONG_TOKEN"
        )
48: revert NotPermitted()
78: revert NotPermitted()
185: revert NotPermitted()
```
[348](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L348) | [48](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L48) | [78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L78) | [185](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L185)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

62: revert NoValue()
66: revert NoValue()
127: revert BaseRewardTooHigh()
131: revert BaseRewardTooLow()
137: revert BaseRewardFarmTooHigh()
141: revert BaseRewardFarmTooLow()
155: revert MaxFeeEthTooHigh()
159: revert MaxFeeEthTooLow()
173: revert MaxFeeFarmEthTooHigh()
177: revert MaxFeeFarmEthTooLow()
```
[62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L62) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L66) | [127](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L127) | [131](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L131) | [137](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L137) | [141](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L141) | [155](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L155) | [159](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L159) | [173](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L173) | [177](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L177)

```solidity
File: contracts/PoolManager.sol

37: revert InvalidAction()
175: revert InvalidAddress()
```
[37](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L37) | [175](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L175)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

390: revert TooLowValue()
394: revert TooHighValue()
```
[390](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L390) | [394](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L394)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

40: revert AccessDenied()
44: revert AccessDenied()
48: revert AccessDenied()
```
[40](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L40) | [44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L44) | [48](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L48)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

46: revert NoValue()
50: revert NoValue()
54: revert NoValue()
58: revert NoValue()
62: revert NoValue()
199: revert NotIncentiveMaster()
210: revert NotWiseLiquidation()
221: revert NotWiseLending()
```
[46](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L46) | [50](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L50) | [54](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L54) | [58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L58) | [62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L62) | [199](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L199) | [210](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L210) | [221](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L221)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

292: revert DebtRatioTooHigh()
```
[292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L292)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

29: revert NotFound()
86: revert ValueTooSmall()
140: revert WrongAddress()
```
[29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L29) | [86](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L86) | [140](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L140)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

195: revert LeverageTooHigh()
204: revert AmountTooSmall()
```
[195](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L195) | [204](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L204)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

31: require(
            msg.sender == farmContract,
            "PowerFarmsNFTs: INVALID_FARM"
        )
190: require(
            _exists(_tokenId) == true,
            "PowerFarmsNFTs: WRONG_TOKEN"
        )
```
[31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L31) | [190](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L190)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

47: require(
            isOwner(
                _keyId,
                msg.sender
            ) == true
        )
85: revert AlreadyReserved()
123: revert InvalidKey()
```
[47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L47) | [85](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L85) | [123](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L123)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

45: revert InvalidDecimals()
105: revert CardinalityNotSatisfied()
109: revert OldestObservationNotSatisfied()
```
[45](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L45) | [105](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L105) | [109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L109)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

110: revert CardinalityNotSatisfied()
114: revert OldestObservationNotSatisfied()
```
[110](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L110) | [114](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L114)

```solidity
File: contracts/WrapperHub/Declarations.sol

58: revert NoValue()
62: revert NoValue()
112: revert AlreadySet()
```
[58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L58) | [62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L62) | [112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L112)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

91: revert CardinalityNotSatisfied()
95: revert OldestObservationNotSatisfied()
```
[91](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L91) | [95](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L95)

```solidity
File: contracts/OwnableMaster.sol

29: revert NotMaster()
45: revert NotProposed()
61: revert NoValue()
77: revert NoValue()
```
[29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L29) | [45](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L45) | [61](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L61) | [77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L77)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

45: revert DeployForbidden()
```
[45](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L45)

```solidity
File: contracts/DerivativeOracles/CustomOracleSetup.sol

15: require(
            msg.sender == master,
            "CustomOracleSetup: NOT_MASTER"
        )
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L15)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

19: revert AmountTooSmall()
34: revert SendValueFailed()
```
[19](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L19) | [34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L34)
</details>

### [G-12] Use local variables for emitting

Use the function/modifier's local copy of the state variable, rather than incurring an extra Gwarmaccess (100 gas).
In the unlikely event that the state variable hasn't already been used by the function/modifier, consider whether it is really necessary to include it in the event, given the fact that it incurs a Gcoldsload (2100 gas), or whether it can be passed in to or back out of the functions that do use it.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/FeeManager/FeeManager.sol

204: emit ClaimedOwnershipIncentiveMaster(
            incentiveMaster,
            block.timestamp
        )
```
[204](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L204)
</details>

### [G-13] `x.a = x.a + b` always cheaper than `x.a += b` for Structs

Using direct assignment operations (e.g., `x.a = x.a + b`) is more gas-efficient compared to compound assignment operations (e.g., `x.a += b`).
Examples:
```solidity
    // direct write to storage   -> 5337 gas
    // struct storage pointer    -> 5341 gas
    // memory struct             -> 4628 gas
    // memory function param     -> 1109 gas
    x.a += b;

    // direct write to storage   -> 5330 gas
    // struct storage pointer    -> 5334 gas
    // memory struct             -> 4625 gas
    // memory function param     -> 1106 gas
    x.a = struct.a + b;
```


<details>
<summary><i>14 issue instances in 2 files:</i></summary>

```solidity
File: contracts/MainHelper.sol

591: userLendingData[_nftId][_poolToken].shares += _shares
606: userLendingData[_nftId][_poolToken].shares -= _shares
```
[591](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L591) | [606](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L606)

```solidity
File: contracts/WiseLowLevelHelper.sol

230: globalPoolData[_poolToken].totalPool += _amount
239: globalPoolData[_poolToken].totalPool -= _amount
248: lendingPoolData[_poolToken].totalDepositShares += _amount
257: lendingPoolData[_poolToken].totalDepositShares -= _amount
266: borrowPoolData[_poolToken].pseudoTotalBorrowAmount += _amount
275: borrowPoolData[_poolToken].pseudoTotalBorrowAmount -= _amount
284: borrowPoolData[_poolToken].totalBorrowShares += _amount
293: borrowPoolData[_poolToken].totalBorrowShares -= _amount
302: lendingPoolData[_poolToken].pseudoTotalPool += _amount
311: lendingPoolData[_poolToken].pseudoTotalPool -= _amount
338: globalPoolData[_poolToken].totalBareToken += _amount
347: globalPoolData[_poolToken].totalBareToken -= _amount
```
[230](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L230) | [239](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L239) | [248](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L248) | [257](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L257) | [266](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L266) | [275](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L275) | [284](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L284) | [293](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L293) | [302](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L302) | [311](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L311) | [338](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L338) | [347](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L347)
</details>

### [G-14] Initializers can be marked as payable

Payable functions cost less gas to execute, because the compiler does not have to add extra checks to ensure that no payment is provided.
Initializers can be safely marked as payable, because only the deployer or the factory contract would call the function without carrying any funds.

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

### [G-15] Unused `private` functions should be removed to save deployment gas

All private functions that are never used can be safely removed or commented out to save gas.

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

### [G-16] Create variable outside the loop

Creating variables inside the loop consum more gas compared to declaring them outside and just reaffecting values to them inside the loop.
```solidity
    // uint256 outsideVar = anyValue;
    for (uint256 i = 0; i < 10; i++) {
        // outsideVar = value;     // 1984 gas
        uint256 insideVar = value; // 2012 gas 
    }
```

<details>
<summary><i>13 issue instances in 6 files:</i></summary>

```solidity
File: contracts/MainHelper.sol

711: address poolToken = _getPositionTokenByIndex(
                _nftId,
                endPosition
            )
```
[711](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L711)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

237: UserReward memory userReward = _getUserReward(
                rewardTokens[i],
                PENDLE_POWER_FARM_CONTROLLER
            )
274: uint256 indexDiff = index
                - lastIndex[i]
277: uint256 activeBalance = _getActiveBalance()
278: uint256 totalLpAssetsCurrent = totalLpAssets()
279: uint256 lpBalanceController = _getBalanceLpBalanceController()
281: bool scaleNecessary = totalLpAssetsCurrent < lpBalanceController
```
[237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L237) | [274](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L274) | [277](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L277) | [278](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L278) | [279](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L279) | [281](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L281)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

627: uint256 currentTimestamp = _getRoundTimestamp(
                _tokenAddress,
                latestRoundId - i
            )
```
[627](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L627)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

260: address token = childInfo.rewardTokens[i]
```
[260](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L260)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

256: address currentAddress = WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
292: address currentAddress = WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
```
[256](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L256) | [292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L292)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

27: address currentAddress = WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            )
59: address currentAddress = WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            )
```
[27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L27) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L59)
</details>

### [G-17] Use bytes32 in place of string

For strings of 32 char strings and below you can use bytes32 instead as it's more gas efficient

<details>
<summary><i>4 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PositionNFTs.sol

359: string(
            abi.encodePacked(
                currentBaseURI,
                _toString(_tokenId),
                baseExtension
            )
        )
406: string(
            bstr
        )
```
[359](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L359) | [406](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L406)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

201: string(
            abi.encodePacked(
                currentBaseURI,
                _toString(_tokenId),
                baseExtension
            )
        )
248: string(
            bstr
        )
```
[201](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L201) | [248](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L248)
</details>

### [G-18] Using `delete` on a `bool` variable is cheaper than assigning it to `false`

```solidity
function test() external {
    delete bar; // 5184 gas
    bar = false; // 5208 gas
}
```

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

### [G-19] Use `Array.unsafeAccess()` to avoid repeated array length checks

When using storage arrays, solidity adds an internal lookup of the array's length (a Gcoldsload 2100 gas) to ensure you don't read past the array's end.
You can avoid this lookup by using `Array.unsafeAccess()` in cases where the length has already been checked, as is the case with the instances below.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/WrapperHub/AaveHub.sol

71: for (uint256 i = 0; i < _underlyingAssets.length; i++) {
```
[71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L71)
</details>

### [G-20] Split `revert` checks to save gas

Using boolean operators in a single `if() revert` statement can consume more gas than necessary.
Consider splitting these statements to save gas.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

96: if (_answer > maxAnswer || _answer < minAnswer) {
            revert OracleIsDead();
```
[96](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L96)
</details>

### [G-21] Reduce deployment costs by tweaking contracts' metadata

The Solidity compiler appends 53 bytes of metadata to the smart contract code which translates to an extra 10,600 gas (200 per bytecode) + the calldata cost (16 gas per non-zero bytes, 4 gas per zero-byte).
This translates to up to 848 additional gas in calldata cost.
One way to reduce this cost is by optimizing the IPFS hash that gets appended to the smart contract code.

Why is this important?
- The metadata adds an extra 53 bytes, resulting in an additional 10,600 gas cost for deployment.
- It also incurs up to 848 additional gas in calldata cost.

Options to Reduce Gas:
1. Use the `--no-cbor-metadata` compiler option to exclude metadata, but this might affect contract verification.
2. Mine for code comments that lead to an IPFS hash with more zeros, reducing calldata costs.

<details>
<summary><i>44 issue instances in 44 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1)

```solidity
File: contracts/MainHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L1)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L1)

```solidity
File: contracts/FeeManager/FeeManager.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L1)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L1)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L1)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L1)

```solidity
File: contracts/WiseCore.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L1)

```solidity
File: contracts/WiseLendingDeclaration.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L1)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L1)

```solidity
File: contracts/WrapperHub/AaveHub.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L1)

```solidity
File: contracts/WiseLowLevelHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L1)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L1)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L1)

```solidity
File: contracts/WrapperHub/AaveHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L1)

```solidity
File: contracts/PositionNFTs.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L1)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L1)

```solidity
File: contracts/PoolManager.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L1)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L1)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L1)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L1)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L1)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L1)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L1)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L1)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L1)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L1)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L1)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L1)

```solidity
File: contracts/WrapperHub/Declarations.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L1)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L1)

```solidity
File: contracts/OwnableMaster.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L1)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L1)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L1)

```solidity
File: contracts/FeeManager/FeeManagerEvents.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerEvents.sol#L1)

```solidity
File: contracts/DerivativeOracles/CustomOracleSetup.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L1)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L1)

```solidity
File: contracts/TransferHub/WrapperHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol#L1)

```solidity
File: contracts/TransferHub/CallOptionalReturn.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L1)

```solidity
File: contracts/TransferHub/TransferHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol#L1)

```solidity
File: contracts/WrapperHub/AaveEvents.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveEvents.sol#L1)

```solidity
File: contracts/TransferHub/ApprovalHelper.sol

1: Consider optimizing the IPFS hash during deployment.
```
[1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/ApprovalHelper.sol#L1)
</details>

### [G-22] Use `storage` instead of `memory` for state structs/arrays

When fetching data from storage, assigning it to a memory variable causes all fields of the struct/array to be read from storage, incurring a Gcoldsload (2100 gas) for each field.
Reading fields from the new memory variable incurs an additional MLOAD rather than a cheap stack read.

Instead, declare variables with the storage keyword and cache any fields that need to be re-read in stack variables.
This approach is more cost-efficient, only incurring the Gcoldsload for fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable is if the full struct/array is being returned by the function, passed to a function that requires memory, or read from another memory array/struct.

<details>
<summary><i>4 issue instances in 2 files:</i></summary>

```solidity
File: contracts/MainHelper.sol

1099: BorrowRatesEntry memory borrowData = borrowRatesData[
            _poolToken
        ];
```
[1099](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1099)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

62: CompoundStruct memory childInfo = pendleChildCompoundInfo[
            _pendleMarket
        ];
300: address[] memory rewardTokens = _getRewardTokens(
            _pendleMarket
        );
504: CompoundStruct memory childInfo = pendleChildCompoundInfo[
            _pendleMarket
        ];
```
[62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L62) | [300](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L300) | [504](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L504)
</details>

### [G-23] Unutilized Named Return Variables Waste Gas Without Optimizer

Utilizing named return variables without assignment or explicit return in functions can lead to unnecessary gas costs without an active optimizer.
To enhance gas efficiency, you might consider opting for unnamed return variables, especially when they aren't explicitly used or returned.
Keeping the current configuration without an active optimizer can result in extra gas costs associated with superfluous stack variables.

<details>
<summary><i>23 issue instances in 17 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

/// @audit - Redundant `return true;` statement in functions with named return variables
/// @audit - Redundant `return true;` statement in functions with named return variables
/// @audit - Redundant `return true;` statement in functions with named return variables
/// @audit - Redundant `return true;` statement in functions with named return variables
/// @audit - Redundant `return true;` statement in functions with named return variables
237: function checksWithdraw(
        uint256 _nftId,
        address _caller,
        address _poolToken
    )
        external
        view
        returns (bool specialCase)
    {
/// @audit - Redundant `return true;` statement in functions with named return variables
/// @audit - Redundant `return true;` statement in functions with named return variables
/// @audit - Redundant `return true;` statement in functions with named return variables
275: function checksSolelyWithdraw(
        uint256 _nftId,
        address _caller,
        address _poolToken
    )
        external
        view
        returns (bool specialCase)
    {
/// @audit - Redundant `return true;` statement in functions with named return variables
/// @audit - Redundant `return true;` statement in functions with named return variables
306: function checksBorrow(
        uint256 _nftId,
        address _caller,
        address _poolToken
    )
        external
        view
        returns (bool specialCase)
    {
```
[237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L237) | [275](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L275) | [306](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L306)

```solidity
File: contracts/MainHelper.sol

/// @audit - Redundant `return;` statement in functions with named return variables
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
[667](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L667)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol

/// @audit - Redundant `return ethCollateral;` statement in functions with named return variables
/// @audit - Redundant `return ethCollateral;` statement in functions with named return variables
190: function getFullCollateralETH(
        uint256 _nftId,
        address _poolToken
    )
        public
        view
        returns (uint256 ethCollateral)
    {
/// @audit - Redundant `return ethCollateral;` statement in functions with named return variables
246: function _getCollateralOfTokenETHUpdated(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        internal
        view
        returns (uint256 ethCollateral)
    {
```
[190](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L190) | [246](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L246)

```solidity
File: contracts/FeeManager/FeeManager.sol

/// @audit - Redundant `return (
                0,
                0
            );` statement in functions with named return variables
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
/// @audit - Redundant `return 0;` statement in functions with named return variables
816: function paybackBadDebtNoReward(
        uint256 _nftId,
        address _paybackToken,
        uint256 _shares
    )
        external
        returns (uint256 paybackAmount)
    {
```
[730](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L730) | [816](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L816)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

/// @audit - Redundant `return IPendleChild(PENDLE_CHILD).withdrawExactShares(
            WISE_LENDING.withdrawExactShares(
                _nftId,
                PENDLE_CHILD,
                _lendingShares
            )
        );` statement in functions with named return variables
309: function _withdrawPendleLPs(
        uint256 _nftId,
        uint256 _lendingShares
    )
        private
        returns (uint256 withdrawnLpsAmount)
    {
```
[309](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L309)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

/// @audit - Redundant `return UNI_V3_FACTORY.getPool(
            _token0,
            _token1,
            _fee
        );` statement in functions with named return variables
283: function _getPool(
        address _token0,
        address _token1,
        uint24 _fee
    )
        internal
        view
        returns (address pool)
    {
```
[283](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L283)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

/// @audit - Redundant `return newVeBalance;` statement in functions with named return variables
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
/// @audit - Redundant `return amount;` statement in functions with named return variables
450: function withdrawLock()
        external
        onlyMaster
        returns (uint256 amount)
    {
```
[364](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L364) | [450](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L450)

```solidity
File: contracts/WiseCore.sol

/// @audit - Redundant `return receiveAmount + potentialPureExtraCashout;` statement in functions with named return variables
/// @audit - Redundant `return receiveAmount;` statement in functions with named return variables
/// @audit - Redundant `return receiveAmount;` statement in functions with named return variables
/// @audit - Redundant `return _withdrawOrAllocateSharesLiquidation(
            _nftId,
            _nftIdLiquidator,
            _receiveTokens,
            _removePercentage
        ) + receiveAmount;` statement in functions with named return variables
543: function _calculateReceiveAmount(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _receiveTokens,
        uint256 _removePercentage
    )
        private
        returns (uint256 receiveAmount)
    {
```
[543](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L543)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

/// @audit - Redundant `return true;` statement in functions with named return variables
/// @audit - Redundant `return _chainLinkIsDead(
                _tokenAddress
            );` statement in functions with named return variables
/// @audit - Redundant `return state;` statement in functions with named return variables
439: function chainLinkIsDead(
        address _tokenAddress
    )
        public
        view
        returns (bool state)
    {
```
[439](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L439)

```solidity
File: contracts/PositionNFTs.sol

/// @audit - Redundant `return "0";` statement in functions with named return variables
371: function _toString(
        uint256 _tokenId
    )
        internal
        pure
        returns (string memory str)
    {
```
[371](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L371)

```solidity
File: contracts/FeeManager/FeeManagerHelper.sol

/// @audit - Redundant `return ORACLE_HUB.getTokensFromETH(
            _receivingToken,
            ORACLE_HUB.getTokensInETH(
                _paybackToken,
                increasedAmount
            )
        );` statement in functions with named return variables
244: function getReceivingToken(
        address _paybackToken,
        address _receivingToken,
        uint256 _paybackAmount
    )
        public
        view
        returns (uint256 receivingAmount)
    {
```
[244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L244)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

/// @audit - Redundant `return _coreLiquidation(
            _nftId,
            _nftIdLiquidator,
            _shareAmountToPay
        );` statement in functions with named return variables
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
```
[93](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L93)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

/// @audit - Redundant `return "0";` statement in functions with named return variables
213: function _toString(
        uint256 _tokenId
    )
        internal
        pure
        returns (string memory str)
    {
```
[213](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L213)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

/// @audit - Redundant `return (
            roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answeredInRound
        );` statement in functions with named return variables
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
```
[151](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L151)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

/// @audit - Redundant `return (
            roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answeredInRound
        );` statement in functions with named return variables
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
```
[146](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L146)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

/// @audit - Redundant `return (
            roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answeredInRound
        );` statement in functions with named return variables
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
```
[125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L125)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

/// @audit - Redundant `return (
            roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answerdInRound
        );` statement in functions with named return variables
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
/// @audit - Redundant `return (
            _roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answeredInRound
        );` statement in functions with named return variables
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
[54](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L54) | [77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L77)
</details>

### [G-24] Using `private` rather than `public`, saves gas

Public storage variables increase the contract's size due to the implicit generation of public getter functions. 
This makes the contract larger and could increase deployment and interaction costs.

If you do not require other contracts to read these variables, consider making them `private`. 

Example:
```solidity
/// 145426 gas to deploy
contract PublicState {
    address public first;
    address public second;
}
/// 77126 gas to deploy
contract PrivateState {
    address private first;
    address private second;
}
```

<details>
<summary><i>164 issue instances in 20 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

28: address public UNDERLYING_PENDLE_MARKET;
29: address public PENDLE_POWER_FARM_CONTROLLER;
32: uint256 public underlyingLpAssetsCurrent;
35: uint256 public totalLpAssetsToDistribute;
38: IPendleMarket public PENDLE_MARKET;
41: IPendleSy public PENDLE_SY;
44: IPendleController public PENDLE_CONTROLLER;
47: uint16 public MAX_CARDINALITY;
49: uint256 public mintFee;
50: uint256 public lastInteraction;
```
[28](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L28) | [29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L29) | [32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L32) | [35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L35) | [38](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L38) | [41](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L41) | [44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L44) | [47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L47) | [49](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L49) | [50](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L50)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

10: PendlePowerFarmTokenFactory public immutable PENDLE_POWER_FARM_TOKEN_FACTORY;
```
[10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L10)

```solidity
File: contracts/WiseLendingDeclaration.sol

165: address public immutable WETH_ADDRESS;
174: IWiseSecurity public WISE_SECURITY;
180: IPositionNFTs public immutable POSITION_NFT;
183: IWiseOracleHub public immutable WISE_ORACLE;
268: mapping(address => uint256) public maxDepositValueToken;
270: mapping(uint256 => address[]) public positionLendTokenData;
271: mapping(uint256 => address[]) public positionBorrowTokenData;
273: mapping(uint256 => mapping(address => uint256)) public userBorrowShares;
274: mapping(uint256 => mapping(address => uint256)) public pureCollateralAmount;
275: mapping(uint256 => mapping(address => LendingEntry)) public userLendingData;
278: mapping(address => BorrowRatesEntry) public borrowRatesData;
279: mapping(address => AlgorithmEntry) public algorithmData;
280: mapping(address => GlobalPoolEntry) public globalPoolData;
281: mapping(address => LendingPoolEntry) public lendingPoolData;
282: mapping(address => BorrowPoolEntry) public borrowPoolData;
283: mapping(address => TimestampsPoolEntry) public timestampsPoolData;
286: mapping(uint256 => bool) public positionLocked;
288: mapping(address => bool) public verifiedIsolationPool;
```
[165](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L165) | [174](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L174) | [180](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L180) | [183](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L183) | [268](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L268) | [270](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L270) | [271](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L271) | [273](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L273) | [274](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L274) | [275](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L275) | [278](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L278) | [279](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L279) | [280](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L280) | [281](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L281) | [282](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L282) | [283](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L283) | [286](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L286) | [288](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L288)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

43: bool public isShutdown;
46: bool public allowEnter;
49: uint256 public collateralFactor;
52: uint256 public minDepositEthAmount;
54: address public immutable aaveTokenAddresses;
55: address public immutable borrowTokenAddresses;
57: address public immutable ENTRY_ASSET;
58: address public immutable PENDLE_CHILD;
66: IAave public immutable AAVE;
67: IAaveHub public immutable AAVE_HUB;
68: IWiseLending public immutable WISE_LENDING;
69: IWiseOracleHub public immutable ORACLE_HUB;
70: IWiseSecurity public immutable WISE_SECURITY;
71: IBalancerVault public immutable BALANCER_VAULT;
72: IPositionNFTs public immutable POSITION_NFT;
73: IStETH public immutable ST_ETH;
74: ICurve public immutable CURVE;
75: IUniswapV3 public immutable UNISWAP_V3_ROUTER;
77: IPendleSy public immutable PENDLE_SY;
78: IPendleRouter public immutable PENDLE_ROUTER;
79: IPendleMarket public immutable PENDLE_MARKET;
80: IPendleRouterStatic public immutable PENDLE_ROUTER_STATIC;
108: mapping(uint256 => bool) public isAave;
```
[43](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L43) | [46](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L46) | [49](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L49) | [52](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L52) | [54](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L54) | [55](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L55) | [57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L57) | [58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L58) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L66) | [67](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L67) | [68](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L68) | [69](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L69) | [70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L70) | [71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L71) | [72](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L72) | [73](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L73) | [74](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L74) | [75](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L75) | [77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L77) | [78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L78) | [79](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L79) | [80](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L80) | [108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L108)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

56: uint256 public reservedPendleForLocking;
58: mapping(address => address) public pendleChildAddress;
61: address[] public activePendleMarkets;
66: uint256 public exchangeIncentive;
68: IPendleLock immutable public PENDLE_LOCK;
69: IPendleVoter immutable public PENDLE_VOTER;
70: IPendleVoteRewards immutable public PENDLE_VOTE_REWARDS;
72: IArbRewardsClaimer public ARB_REWARDS;
```
[56](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L56) | [58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L58) | [61](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L61) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L66) | [68](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L68) | [69](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L69) | [70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L70) | [72](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L72)

```solidity
File: contracts/PositionNFTs.sol

13: string public baseURI;
14: string public baseExtension;
16: address public feeManager;
17: uint256 public totalReserved;
18: uint256 public immutable FEE_MANAGER_NFT;
20: mapping(address => uint256) public reserved;
21: mapping(address => bool) public reserveRole;
23: bool public reservePublicBlocked;
```
[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L13) | [14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L14) | [16](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L16) | [17](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L17) | [18](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L18) | [20](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L20) | [21](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L21) | [23](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L23)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

185: uint256 public constant BORROW_PERCENTAGE_CAP = 95 * PRECISION_FACTOR_E16;
186: address public immutable AAVE_HUB;
191: IFeeManager public immutable FEE_MANAGER;
194: IWiseLending public immutable WISE_LENDING;
197: IPositionNFTs public immutable POSITION_NFTS;
200: IWiseOracleHub public immutable WISE_ORACLE;
203: IWiseLiquidation public immutable WISE_LIQUIDATION;
214: uint256 public minDepositEthValue = 1; // in wei?
238: mapping(address => bool) public wasBlacklisted;
241: mapping(address => CurveSwapStructData) public curveSwapInfoData;
244: mapping(address => CurveSwapStructToken) public curveSwapInfoToken;
247: mapping(address => bool) public securityWorker;
252: uint256 public maxFeeETH;
255: uint256 public maxFeeFarmETH;
258: uint256 public baseRewardLiquidation;
261: uint256 public baseRewardLiquidationFarm;
```
[185](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L185) | [186](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L186) | [191](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L191) | [194](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L194) | [197](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L197) | [200](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L200) | [203](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L203) | [214](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L214) | [238](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L238) | [241](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L241) | [244](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L244) | [247](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L247) | [252](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L252) | [255](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L255) | [258](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L258) | [261](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L261)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

100: IAave public immutable AAVE;
103: IWiseLending public immutable WISE_LENDING;
106: IPositionNFTs public immutable POSITION_NFTS;
109: IWiseSecurity public immutable WISE_SECURITY;
112: IWiseOracleHub public immutable ORACLE_HUB;
117: uint256 public totalBadDebtETH;
120: uint256 public paybackIncentive;
123: address[] public poolTokenAddresses;
126: address public incentiveMaster;
129: address public proposedIncentiveMaster;
132: address public incentiveOwnerA;
135: address public incentiveOwnerB;
140: mapping(uint256 => uint256) public badDebtPosition;
143: mapping(address => uint256) public feeTokens;
146: mapping(address => uint256) public incentiveUSD;
149: mapping(address => bool) public poolTokenAdded;
152: mapping(address => bool) public isAaveToken;
155: mapping(address => address) public underlyingToken;
158: mapping(address => mapping(address => bool)) public allowedTokens;
161: mapping(address => mapping(address => uint256)) public gatheredIncentiveToken;
164: uint256 public immutable FEE_MANAGER_NFT;
172: uint256 public constant INCENTIVE_PORTION = 5 * PRECISION_FACTOR_E15;
```
[100](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L100) | [103](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L103) | [106](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L106) | [109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L109) | [112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L112) | [117](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L117) | [120](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L120) | [123](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L123) | [126](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L126) | [129](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L129) | [132](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L132) | [135](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L135) | [140](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L140) | [143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L143) | [146](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L146) | [149](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L149) | [152](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L152) | [155](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L155) | [158](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L158) | [161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L161) | [164](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L164) | [172](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L172)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

26: string public baseURI;
27: string public baseExtension;
28: address public farmContract;
```
[26](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L26) | [27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L27) | [28](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L28)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

15: uint256 public totalMinted;
18: uint256 public totalReserved;
21: uint256 public availableNFTCount;
24: mapping(uint256 => uint256) public farmingKeys;
27: mapping(address => uint256) public reservedKeys;
30: mapping(uint256 => uint256) public availableNFTs;
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L15) | [18](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L18) | [21](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L21) | [24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L24) | [27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L27) | [30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L30)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

58: address public immutable PENDLE_MARKET_ADDRESS;
59: IPendleMarket public immutable PENDLE_MARKET;
62: IPendleSy public immutable PENDLE_SY;
63: IPriceFeed public immutable FEED_ASSET;
64: IOraclePendle public immutable ORACLE_PENDLE_PT;
73: uint32 public immutable TWAP_DURATION;
76: string public name;
```
[58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L58) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L59) | [62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L62) | [63](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L63) | [64](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L64) | [73](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L73) | [76](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L76)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

76: address public immutable WETH_ADDRESS;
79: address public constant SEQUENCER_ADDRESS = 0xFdB631F5EE196F0ed6FAa767959853A9F217697D;
82: address public immutable ETH_USD_PLACEHOLDER;
88: IPriceFeed public immutable ETH_PRICE_FEED;
91: IPriceFeed public immutable SEQUENCER;
94: IUniswapV3Factory public immutable UNI_V3_FACTORY;
142: mapping(address => IPriceFeed) public priceFeed;
145: mapping(address => uint256) public heartBeat;
148: mapping(address => IAggregator) public tokenAggregatorFromTokenAddress;
151: mapping(address => address[]) public underlyingFeedTokens;
154: mapping(address => UniTwapPoolInfo) public uniTwapPoolInfo;
157: mapping(address => DerivativePartnerInfo) public derivativePartnerTwap;
```
[76](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L76) | [79](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L79) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L82) | [88](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L88) | [91](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L91) | [94](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L94) | [142](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L142) | [145](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L145) | [148](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L148) | [151](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L151) | [154](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L154) | [157](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L157)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

50: IPriceFeed public immutable USD_FEED_ASSET;
53: IPriceFeed public immutable ETH_FEED_ASSET;
56: IOraclePendle public immutable ORACLE_PENDLE_PT;
59: uint256 public immutable POW_USD_FEED;
62: uint256 public immutable POW_ETH_USD_FEED;
71: uint32 public immutable TWAP_DURATION;
74: string public name;
```
[50](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L50) | [53](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L53) | [56](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L56) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L59) | [62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L62) | [71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L71) | [74](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L74)

```solidity
File: contracts/WrapperHub/Declarations.sol

25: bool public sendingProgressAaveHub;
29: IWiseLending immutable public WISE_LENDING;
30: IPositionNFTs immutable public POSITION_NFT;
32: IWiseSecurity public WISE_SECURITY;
34: address immutable public WETH_ADDRESS;
35: address immutable public AAVE_ADDRESS;
41: mapping(address => address) public aaveTokenAddress;
```
[25](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L25) | [29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L29) | [30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L30) | [32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L32) | [34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L34) | [35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L35) | [41](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L41)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

44: address public immutable PENDLE_MARKET_ADDRESS;
47: IPriceFeed public immutable FEED_ASSET;
48: IOraclePendle public immutable ORACLE_PENDLE_PT;
57: uint32 public immutable DURATION;
60: string public name;
```
[44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L44) | [47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L47) | [48](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L48) | [57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L57) | [60](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L60)

```solidity
File: contracts/OwnableMaster.sol

11: address public master;
12: address public proposedMaster;
```
[11](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L11) | [12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L12)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

13: address public immutable IMPLEMENTATION_TARGET;
14: address public immutable PENDLE_POWER_FARM_CONTROLLER;
```
[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L13) | [14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L14)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

15: IPriceFeed public priceFeedPendleLpOracle;
16: IPendleChildToken public pendleChildToken;
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L15) | [16](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L16)

```solidity
File: contracts/DerivativeOracles/CustomOracleSetup.sol

7: address public master;
8: uint256 public lastUpdateGlobal;
10: uint80 public globalRoundId;
12: mapping(uint80 => uint256) public timeStampByRoundId;
```
[7](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L7) | [8](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L8) | [10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L10) | [12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L12)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

10: bool public sendingProgress;
```
[10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L10)
</details>

### [G-25] Consider using ERC721A instead of ERC721

The ERC721A standard, an evolution of the ERC721 standard, has been designed to significantly reduce gas costs associated with minting multiple non-fungible tokens (NFTs) in a single transaction.
Switching to ERC721A can offer substantial gas savings, thereby optimizing the efficiency of NFT transactions and potentially leading to considerable cost reductions.

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

### [G-26] Refactor duplicated require()/revert() checks to save gas

Duplicate require()/revert() checks can be refactored into a modifier or function, saving deployment costs.

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

### [G-27] Use Assembly for Efficient Memory Management in Multiple External Calls

When making multiple external calls in a Solidity contract, it may be more efficient to use inline assembly to reuse the memory space allocated for function arguments and return data.
This can prevent unnecessary memory expansion and reduce gas costs.

Example:
```solidity
contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}

contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

<details>
<summary><i>11 issue instances in 4 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

/// @audit function `prepareCurvePools()` has multiple external calls
157: ICurve(curvePool).coins(tokenIndexForApprove),
            curvePool,
            0
        );
163: ICurve(curvePool).coins(tokenIndexForApprove),
            curvePool,
            UINT256_MAX
        );
177: ICurve(curveMetaPool).coins(tokenIndexForApprove),
            curveMetaPool,
            0
        );
183: ICurve(curveMetaPool).coins(tokenIndexForApprove),
            curveMetaPool,
            UINT256_MAX
        );
```
[157](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L157) | [163](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L163) | [177](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L177) | [183](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L183)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit function `_calculateRewardsClaimedOutside()` has multiple external calls
222: address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(
            UNDERLYING_PENDLE_MARKET
        );
226: uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
            UNDERLYING_PENDLE_MARKET
        );
243: PENDLE_MARKET.redeemRewards(
                    PENDLE_POWER_FARM_CONTROLLER
                );
```
[222](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L222) | [226](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L226) | [243](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L243)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

/// @audit function `latestAnswer()` has multiple external calls
104: ) = ORACLE_PENDLE_PT.getOracleState(
            PENDLE_MARKET_ADDRESS,
            TWAP_DURATION
        );
117: uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
            PENDLE_MARKET_ADDRESS,
            TWAP_DURATION
        );
```
[104](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L104) | [117](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L117)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

/// @audit function `latestAnswer()` has multiple external calls
85: ) = ORACLE_PENDLE_PT.getOracleState(
            PENDLE_MARKET_ADDRESS,
            DURATION
        );
98: uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
            PENDLE_MARKET_ADDRESS,
            DURATION
        );
```
[85](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L85) | [98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L98)
</details>

### [G-28] Optimize `require/revert` Statements with Assembly

Using inline assembly for revert statements in Solidity can offer gas optimizations.
The typical `require` or `revert` statements in Solidity perform additional memory operations and type checks which can be avoided by using low-level assembly code.

In certain contracts, particularly those that might revert often or are otherwise sensitive to gas costs, using assembly to handle reversion can offer meaningful savings.
These savings primarily come from avoiding memory expansion costs and extra type checks that the Solidity compiler performs.

Example:
```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
function restrictedAction(uint256 num)  external {
    require(owner == msg.sender, "caller is not owner");
    specialNumber = num;
}

// calling restrictedAction(2) with a non-owner address: 23734
function restrictedAction(uint256 num)  external {
    assembly {
        if sub(caller(), sload(owner.slot)) {
            mstore(0x00, 0x20) // store offset to where length of revert message is stored
            mstore(0x20, 0x13) // store length (19)
            mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
            revert(0x00, 0x60) // revert with data
        }
    }
    specialNumber = num;
}
```

<details>
<summary><i>6 issue instances in 5 files:</i></summary>

```solidity
File: contracts/PositionNFTs.sol

348: require(
            _exists(_tokenId) == true,
            "PositionNFTs: WRONG_TOKEN"
        );
```
[348](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L348)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

31: require(
            msg.sender == farmContract,
            "PowerFarmsNFTs: INVALID_FARM"
        );
190: require(
            _exists(_tokenId) == true,
            "PowerFarmsNFTs: WRONG_TOKEN"
        );
```
[31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L31) | [190](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L190)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

47: require(
            isOwner(
                _keyId,
                msg.sender
            ) == true
        );
```
[47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L47)

```solidity
File: contracts/DerivativeOracles/CustomOracleSetup.sol

15: require(
            msg.sender == master,
            "CustomOracleSetup: NOT_MASTER"
        );
```
[15](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L15)

```solidity
File: contracts/TransferHub/CallOptionalReturn.sol

32: revert();
```
[32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L32)
</details>

### [G-29] Cached Global Variables

Storing global variables in local storage might appear as a useful caching mechanism. 
However, in Solidity, accessing global variables directly is often more gas-efficient than caching them. 
Consider removing these redundant assignments and use the global variables directly.

<details>
<summary><i>3 issue instances in 3 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

1120: uint256 requiredAmount = msg.value;
```
[1120](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1120)

```solidity
File: contracts/MainHelper.sol

505: uint256 currentTime = block.timestamp;
```
[505](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L505)

```solidity
File: contracts/FeeManager/FeeManager.sol

695: address caller = msg.sender;
```
[695](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L695)
</details>

### [G-30] Use Cached Contracts for Multiple External Calls

When function makes multiple calls to the same external contract, it is more gas-efficient to use a local copy of the contract.
This is because the EVM will cache the contract in memory, and subsequent calls will be cheaper.
It's especially true for contracts that are large and/or have many functions.
```solidity
    // local cache -> 6561 gas
    IToken localCache = storageContract;
    localCache.externalCall();
    localCache.externalCall();

    // direct call 6683 gas
    storageContract.externalCall();
    storageContract.externalCall();
```

<details>
<summary><i>10 issue instances in 3 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit function _calculateRewardsClaimedOutside() make external call of `PENDLE_CONTROLLER` - 2 times
222: address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(
226: uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
/// @audit function _increaseCardinalityNext() make external call of `PENDLE_MARKET` - 2 times
350: MarketStorage memory storageMarket = PENDLE_MARKET._storage();
353: PENDLE_MARKET.increaseObservationsCardinalityNext(
/// @audit function initialize() make external call of `PENDLE_MARKET` - 2 times
699: if (PENDLE_MARKET.isExpired() == true) {
718: ) = PENDLE_MARKET.readTokens();
```
[222](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L222) | [226](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L226) | [350](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L350) | [353](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L353) | [699](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L699) | [718](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L718)

```solidity
File: contracts/WiseLendingDeclaration.sol

/// @audit function setSecurity() make external call of `WISE_SECURITY` - 2 times
155: WISE_SECURITY.FEE_MANAGER()
158: AAVE_HUB_ADDRESS = WISE_SECURITY.AAVE_HUB();
```
[155](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L155) | [158](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L158)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

/// @audit function latestAnswer() make external call of `pendleChildToken` - 2 times
41: * pendleChildToken.totalLpAssets()
43: / pendleChildToken.totalSupply()
```
[41](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L41) | [43](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L43)
</details>

### [G-31] Avoid Zero to Non-Zero Storage Writes Where Possible

Changing a storage variable from zero to non-zero costs 22,100 gas in total. (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access)
Consider using non-zero architecture to avoid high gas costs for zero to non-zero storage writes.

Example:

```solidity
- uint256 public counter = 0;  // rewrite this costs more
+ uint256 public counter = 1;  // rewrite this costs less
```

<details>
<summary><i>23 issue instances in 7 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

344: totalLpAssetsToDistribute -= additonalAssets;
602: mintFee = _newFee;
628: underlyingLpAssetsCurrent -= tokenAmount;
672: underlyingLpAssetsCurrent -= _underlyingLpAssetAmount;
707: MAX_CARDINALITY = _maxCardinality;
726: lastInteraction = block.timestamp;
731: INITIAL_TIME_STAMP = block.timestamp;
```
[344](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L344) | [602](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L602) | [628](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L628) | [672](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L672) | [707](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L707) | [726](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L726) | [731](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L731)

```solidity
File: contracts/PositionNFTs.sol

68: reservePublicBlocked = true;
122: reserved[_user] = reservedId;
124: totalReserved =
        totalReserved + 1;
334: baseExtension = _newBaseExtension;
```
[68](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L68) | [122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L122) | [124](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L124) | [334](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L334)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

134: baseRewardLiquidation = _baseReward;
144: baseRewardLiquidationFarm = _baseRewardFarm;
162: maxFeeETH = _newMaxFeeETH;
180: maxFeeFarmETH = _newMaxFeeFarmETH;
```
[134](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L134) | [144](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L144) | [162](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L162) | [180](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L180)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

176: baseExtension = _newBaseExtension;
```
[176](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L176)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

90: reservedKeys[_userAddress] = keyId;
91: farmingKeys[keyId] = _wiseLendingNFT;
```
[90](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L90) | [91](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L91)

```solidity
File: contracts/DerivativeOracles/CustomOracleSetup.sol

39: lastUpdateGlobal = _time;
49: timeStampByRoundId[_roundId] = _updateTime;
58: globalRoundId = _aggregatorRoundId;
```
[39](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L39) | [49](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L49) | [58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L58)

```solidity
File: contracts/TransferHub/SendValueHelper.sol

22: sendingProgress = true;
31: sendingProgress = false;
```
[22](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L22) | [31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L31)
</details>

### [G-32] Consider Using `calldata` Instead of `memory` for Function Parameters

When function parameters are not being modified within the function, it can be more gas efficient 
to use `calldata` instead of `memory`. The Ethereum Virtual Machine (EVM) does not need to copy 
parameters marked with `calldata` from `calldata` to `memory`, saving gas. This is particularly effective 
for larger arrays and structs.

<details>
<summary><i>4 issue instances in 3 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

/// @audit `_tokenName, _symbolName` not modified within function
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

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

/// @audit `_flashloanToken, _flashloanAmounts, _feeAmounts` not modified within function
59: function receiveFlashLoan(
        IERC20[] memory _flashloanToken,
        uint256[] memory _flashloanAmounts,
        uint256[] memory _feeAmounts,
        bytes memory _userData
    )
        external
    {
```
[59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L59)

```solidity
File: contracts/PositionNFTs.sol

/// @audit `_newBaseURI` not modified within function
319: function setBaseURI(
        string memory _newBaseURI
    )
        external
        onlyMaster
    {
/// @audit `_newBaseExtension` not modified within function
327: function setBaseExtension(
        string memory _newBaseExtension
    )
        external
        onlyMaster
    {
```
[319](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L319) | [327](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L327)
</details>

### [G-33] State Variables Should Be `immutable` Since They Are Only Set in the Constructor

State variables that are only set in the constructor and are not modified elsewhere in the code should be declared as `immutable`.
This can optimize gas usage as `immutable` variables are read from code, not from storage.

<details>
<summary><i>12 issue instances in 7 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

247: collateralFactor = _collateralFactor;
```
[247](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L247)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

122: exchangeIncentive = 50000;
118: ARB_REWARDS = IArbRewardsClaimer(
```
[122](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L122) | [118](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L118)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

88: paybackIncentive = 5 * PRECISION_FACTOR_E16;
87: incentiveMaster = _master;
90: incentiveOwnerA = 0xf69A0e276664997357BF987df83f32a1a3F80944;
91: incentiveOwnerB = 0x8f741ea9C9ba34B5B8Afc08891bDf53faf4B3FE7;
```
[88](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L88) | [87](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L87) | [90](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L90) | [91](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L91)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

55: name = _oracleName;
```
[55](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L55)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

43: name = _oracleName;
```
[43](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L43)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

40: name = _oracleName;
```
[40](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L40)

```solidity
File: contracts/DerivativeOracles/PendleChildLpOracle.sol

27: priceFeedPendleLpOracle = IPriceFeed(
30: pendleChildToken = IPendleChildToken(
```
[27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L27) | [30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L30)
</details>

### [G-34] Use solady library where possible to save gas

The following OpenZeppelin imports have a Solady equivalent, as such they can be used to save GAS as Solady modules have been specifically designed to be as GAS efficient as possible.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PositionNFTs.sol

4: import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
```
[4](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L4)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

12: import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
```
[12](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L12)
</details>

### [G-35] Keep Strings Smaller Than 32 Bytes for Storage Efficiency

In Solidity, how strings are stored in the contract's storage can have implications for gas costs. Specifically, strings that are 32 bytes or longer are stored differently than shorter strings:

- If the length of the string is 32 bytes or longer, the storage slot for the string variable will hold the value of the length of the string multiplied by 2, plus 1. The actual data of the string is stored in a separate slot, the address of which is the keccak256 hash of the original storage slot.

- For strings shorter than 32 bytes, the length multiplied by 2 is stored in the least significant byte of its storage slot. The remaining bytes of the slot store the string data itself.

Keeping strings shorter than 32 bytes could therefore yield gas savings due to more efficient storage use.

<details>
<summary><i>7 issue instances in 5 files:</i></summary>

```solidity
File: contracts/PositionNFTs.sol

/// @audit variable initial value is not set, it should be set to a value shorter than 32 bytes
13: string public baseURI;
/// @audit variable initial value is not set, it should be set to a value shorter than 32 bytes
14: string public baseExtension;
```
[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L13) | [14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L14)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

/// @audit variable initial value is not set, it should be set to a value shorter than 32 bytes
26: string public baseURI;
/// @audit variable initial value is not set, it should be set to a value shorter than 32 bytes
27: string public baseExtension;
```
[26](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L26) | [27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L27)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

/// @audit variable initial value is not set, it should be set to a value shorter than 32 bytes
76: string public name;
```
[76](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L76)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

/// @audit variable initial value is not set, it should be set to a value shorter than 32 bytes
74: string public name;
```
[74](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L74)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

/// @audit variable initial value is not set, it should be set to a value shorter than 32 bytes
60: string public name;
```
[60](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L60)
</details>

### [G-36] Delete Unused State Variables

State variables that aren't used in the contract not only clutter the codebase but also consume unnecessary gas during deployment.
Specifically, setting non-zero initial values for state variables costs significant gas.
By removing these unused state variables, you can save on both deployment gas and potential future storage gas costs.
This optimization not only reduces gas expenditures but also enhances code clarity and maintainability.
Always ensure a thorough review to confirm that these variables are indeed redundant before removal.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

/// @audit Unused state variable: ARB_TOKEN_ADDRESS
75: address internal constant ARB_TOKEN_ADDRESS = 0x912CE59144191C1204E64559FE8253a0e49E6548;
```
[75](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L75)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

/// @audit Unused state variable: POW_FEED_DECIMALS
67: uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;
```
[67](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L67)
</details>

### [G-37] Mark Functions That Revert For Normal Users As `payable`

Functions guaranteed to revert when called by normal users can be marked `payable`.
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function.
Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

<details>
<summary><i>80 issue instances in 16 files:</i></summary>

```solidity
File: contracts/WiseLending.sol

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
```
[859](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L859) | [939](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L939) | [1055](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1055) | [1412](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1412)

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

85: function setLiquidationSettings(
        uint256 _baseReward,
        uint256 _baseRewardFarm,
        uint256 _newMaxFeeETH,
        uint256 _newMaxFeeFarmETH
    )
        external
        onlyMaster
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
405: function checkBadDebtLiquidation(
        uint256 _nftId
    )
        external
        onlyWiseLending
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
1032: function revokeShutdown()
        external
        onlyMaster
    {
1110: function changeMinDepositValue(
        uint256 _newMinDepositValue
    )
        external
        onlyMaster
    {
```
[85](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L85) | [142](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L142) | [193](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L193) | [405](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L405) | [980](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L980) | [995](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L995) | [1032](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1032) | [1110](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1110)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

591: function changeMintFee(
        uint256 _newFee
    )
        external
        onlyController
    {
```
[591](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L591)

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
```
[49](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L49) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L66) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L82) | [108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L108) | [135](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L135) | [173](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L173) | [214](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L214) | [232](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L232) | [390](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L390) | [412](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L412) | [438](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L438) | [494](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L494) | [514](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L514) | [538](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L538) | [571](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L571)

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
[31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L31) | [210](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L210) | [292](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L292) | [320](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L320) | [333](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L333) | [364](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L364) | [430](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L430) | [450](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L450) | [496](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L496) | [525](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L525) | [543](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L543) | [565](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L565) | [599](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L599) | [611](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L611)

```solidity
File: contracts/WiseLendingDeclaration.sol

139: function setSecurity(
        address _wiseSecurity
    )
        external
        onlyMaster
    {
```
[139](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L139)

```solidity
File: contracts/WiseOracleHub/WiseOracleHub.sol

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
```
[218](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L218) | [266](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L266) | [359](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L359) | [379](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L379) | [402](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L402)

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
610: function skimAave(
        address _underlyingAsset,
        bool _isAave
    )
        external
        validToken(_underlyingAsset)
        onlyMaster
    {
```
[44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L44) | [63](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L63) | [610](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L610)

```solidity
File: contracts/WiseLowLevelHelper.sol

423: function setPoolFee(
        address _poolToken,
        uint256 _newFee
    )
        external
        onlyFeeManager
    {
```
[423](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L423)

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
98: function reservePositionForUser(
        address _user
    )
        onlyReserveRole
        external
        returns (uint256)
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
```
[53](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L53) | [63](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L63) | [70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L70) | [98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L98) | [319](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L319) | [327](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L327)

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
```
[24](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L24) | [99](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L99) | [125](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L125) | [134](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L134) | [145](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L145)

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
209: function exitFarm(
        uint256 _keyId,
        uint256 _allowedSpread,
        bool _ethBack
    )
        external
        updatePools
        onlyKeyOwner(_keyId)
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
[62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L62) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L82) | [209](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L209) | [273](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L273)

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
```
[50](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L50) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L66) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L82) | [161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L161) | [169](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L169)

```solidity
File: contracts/WrapperHub/Declarations.sol

104: function setWiseSecurity(
        address _securityAddress
    )
        external
        onlyMaster
    {
```
[104](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L104)

```solidity
File: contracts/OwnableMaster.sol

70: function proposeOwner(
        address _proposedOwner
    )
        external
        onlyMaster
    {
92: function claimOwnership()
        external
        onlyProposed
    {
103: function renounceOwnership()
        external
        onlyMaster
    {
```
[70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L70) | [92](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L92) | [103](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L103)

```solidity
File: contracts/DerivativeOracles/CustomOracleSetup.sol

25: function renounceOwnership()
        external
        onlyOwner
    {
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
[25](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L25) | [32](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L32) | [41](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L41) | [51](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L51)
</details>

### [G-38] Inefficient Use of String Constants

In Solidity, if data can fit into 32 bytes, it's more efficient to use the bytes32 datatype instead of bytes or strings.

This is because the bytes32 datatype is cheaper in terms of gas usage.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/PositionNFTs.sol

14: string public baseExtension;
```
[14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L14)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

27: string public baseExtension;
```
[27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L27)
</details>

### [G-39] Consider using OZ EnumerateSet in place of nested mappings

Nested mappings and multi-dimensional arrays in Solidity operate through a process of double hashing, wherein the original storage slot and the first key are concatenated and hashed, and then this hash is again concatenated with the second key and hashed. This process can be quite gas expensive due to the double-hashing operation and subsequent storage operation (sstore).

A possible optimization involves manually concatenating the keys followed by a single hash operation and an sstore. However, this technique introduces the risk of storage collision, especially when there are other nested hash maps in the contract that use the same key types. Because Solidity is unaware of the number and structure of nested hash maps in a contract, it follows a conservative approach in computing the storage slot to avoid possible collisions.

OpenZeppelin's EnumerableSet provides a potential solution to this problem. It creates a data structure that combines the benefits of set operations with the ability to enumerate stored elements, which is not natively available in Solidity. EnumerableSet handles the element uniqueness internally and can therefore provide a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays in certain scenarios.

<details>
<summary><i>8 issue instances in 4 files:</i></summary>

```solidity
File: contracts/MainHelper.sol

617: mapping(bytes32 => bool) storage hashMap,
        mapping(uint256 => address[]) storage userTokenData
    )
        internal
    {
        bytes32 hashData = _getHash(
            _nftId,
            _poolToken
        );
```
[617](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L617)

```solidity
File: contracts/WiseLendingDeclaration.sol

272: mapping(uint256 => mapping(address => uint256)) public userBorrowShares;
274: mapping(uint256 => mapping(address => uint256)) public pureCollateralAmount;
275: mapping(uint256 => mapping(address => LendingEntry)) public userLendingData;
```
[272](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L272) | [274](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L274) | [275](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L275)

```solidity
File: contracts/WiseLowLevelHelper.sol

372: mapping(uint256 => mapping(address => uint256)) storage userMapping,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        internal
    {
        userMapping[_nftId][_poolToken] -= _amount;
383: mapping(uint256 => mapping(address => uint256)) storage userMapping,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        internal
    {
        userMapping[_nftId][_poolToken] += _amount;
```
[372](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L372) | [383](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L383)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

158: mapping(address => mapping(address => bool)) public allowedTokens;
161: mapping(address => mapping(address => uint256)) public gatheredIncentiveToken;
```
[158](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L158) | [161](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L161)
</details>

### [G-40] Declare `immutable` as `private` to save gas

Using `private` instead of `public` for immutables saves gas.

The compiler doesn't need to create non-payable getter functions for deployment calldata, store the bytes of the value outside of where it's used, or add another entry to the method ID table, saving 3406-3606 gas in deployment.

<details>
<summary><i>65 issue instances in 13 files:</i></summary>

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

10: PendlePowerFarmTokenFactory public immutable PENDLE_POWER_FARM_TOKEN_FACTORY;
```
[10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L10)

```solidity
File: contracts/WiseLendingDeclaration.sol

165: address public immutable WETH_ADDRESS;
180: IPositionNFTs public immutable POSITION_NFT;
183: IWiseOracleHub public immutable WISE_ORACLE;
```
[165](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L165) | [180](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L180) | [183](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L183)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

54: address public immutable aaveTokenAddresses;
55: address public immutable borrowTokenAddresses;
57: address public immutable ENTRY_ASSET;
58: address public immutable PENDLE_CHILD;
66: IAave public immutable AAVE;
67: IAaveHub public immutable AAVE_HUB;
68: IWiseLending public immutable WISE_LENDING;
69: IWiseOracleHub public immutable ORACLE_HUB;
70: IWiseSecurity public immutable WISE_SECURITY;
71: IBalancerVault public immutable BALANCER_VAULT;
72: IPositionNFTs public immutable POSITION_NFT;
73: IStETH public immutable ST_ETH;
74: ICurve public immutable CURVE;
75: IUniswapV3 public immutable UNISWAP_V3_ROUTER;
77: IPendleSy public immutable PENDLE_SY;
78: IPendleRouter public immutable PENDLE_ROUTER;
79: IPendleMarket public immutable PENDLE_MARKET;
80: IPendleRouterStatic public immutable PENDLE_ROUTER_STATIC;
```
[54](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L54) | [55](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L55) | [57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L57) | [58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L58) | [66](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L66) | [67](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L67) | [68](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L68) | [69](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L69) | [70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L70) | [71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L71) | [72](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L72) | [73](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L73) | [74](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L74) | [75](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L75) | [77](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L77) | [78](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L78) | [79](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L79) | [80](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L80)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

68: IPendleLock immutable public PENDLE_LOCK;
69: IPendleVoter immutable public PENDLE_VOTER;
70: IPendleVoteRewards immutable public PENDLE_VOTE_REWARDS;
```
[68](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L68) | [69](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L69) | [70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L70)

```solidity
File: contracts/PositionNFTs.sol

18: uint256 public immutable FEE_MANAGER_NFT;
```
[18](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L18)

```solidity
File: contracts/WiseSecurity/WiseSecurityDeclarations.sol

186: address public immutable AAVE_HUB;
191: IFeeManager public immutable FEE_MANAGER;
194: IWiseLending public immutable WISE_LENDING;
197: IPositionNFTs public immutable POSITION_NFTS;
200: IWiseOracleHub public immutable WISE_ORACLE;
203: IWiseLiquidation public immutable WISE_LIQUIDATION;
```
[186](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L186) | [191](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L191) | [194](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L194) | [197](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L197) | [200](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L200) | [203](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L203)

```solidity
File: contracts/FeeManager/DeclarationsFeeManager.sol

100: IAave public immutable AAVE;
103: IWiseLending public immutable WISE_LENDING;
106: IPositionNFTs public immutable POSITION_NFTS;
109: IWiseSecurity public immutable WISE_SECURITY;
112: IWiseOracleHub public immutable ORACLE_HUB;
164: uint256 public immutable FEE_MANAGER_NFT;
```
[100](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L100) | [103](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L103) | [106](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L106) | [109](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L109) | [112](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L112) | [164](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L164)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

58: address public immutable PENDLE_MARKET_ADDRESS;
59: IPendleMarket public immutable PENDLE_MARKET;
62: IPendleSy public immutable PENDLE_SY;
63: IPriceFeed public immutable FEED_ASSET;
64: IOraclePendle public immutable ORACLE_PENDLE_PT;
73: uint32 public immutable TWAP_DURATION;
```
[58](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L58) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L59) | [62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L62) | [63](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L63) | [64](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L64) | [73](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L73)

```solidity
File: contracts/WiseOracleHub/Declarations.sol

76: address public immutable WETH_ADDRESS;
82: address public immutable ETH_USD_PLACEHOLDER;
88: IPriceFeed public immutable ETH_PRICE_FEED;
91: IPriceFeed public immutable SEQUENCER;
94: IUniswapV3Factory public immutable UNI_V3_FACTORY;
```
[76](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L76) | [82](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L82) | [88](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L88) | [91](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L91) | [94](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L94)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

50: IPriceFeed public immutable USD_FEED_ASSET;
53: IPriceFeed public immutable ETH_FEED_ASSET;
56: IOraclePendle public immutable ORACLE_PENDLE_PT;
59: uint256 public immutable POW_USD_FEED;
62: uint256 public immutable POW_ETH_USD_FEED;
71: uint32 public immutable TWAP_DURATION;
```
[50](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L50) | [53](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L53) | [56](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L56) | [59](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L59) | [62](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L62) | [71](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L71)

```solidity
File: contracts/WrapperHub/Declarations.sol

29: IWiseLending immutable public WISE_LENDING;
30: IPositionNFTs immutable public POSITION_NFT;
34: address immutable public WETH_ADDRESS;
35: address immutable public AAVE_ADDRESS;
```
[29](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L29) | [30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L30) | [34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L34) | [35](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L35)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

44: address public immutable PENDLE_MARKET_ADDRESS;
47: IPriceFeed public immutable FEED_ASSET;
48: IOraclePendle public immutable ORACLE_PENDLE_PT;
57: uint32 public immutable DURATION;
```
[44](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L44) | [47](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L47) | [48](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L48) | [57](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L57)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

13: address public immutable IMPLEMENTATION_TARGET;
14: address public immutable PENDLE_POWER_FARM_CONTROLLER;
```
[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L13) | [14](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L14)
</details>

### [G-41] Optimize External Calls with Assembly for Memory Efficiency

Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization.
Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs. 

Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets.
This can result in notable gas savings, especially for contracts that make frequent external calls.

Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using `extcodesize(addr)` before making the call, mitigating risks associated with contract interactions.

<details>
<summary><i>37 issue instances in 15 files:</i></summary>

```solidity
File: contracts/WiseSecurity/WiseSecurity.sol

157: ICurve(curvePool).coins(tokenIndexForApprove),
            curvePool,
            0
        );
163: ICurve(curvePool).coins(tokenIndexForApprove),
            curvePool,
            UINT256_MAX
        );
177: ICurve(curveMetaPool).coins(tokenIndexForApprove),
            curveMetaPool,
            0
        );
183: ICurve(curveMetaPool).coins(tokenIndexForApprove),
            curveMetaPool,
            UINT256_MAX
        );
```
[157](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L157) | [163](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L163) | [177](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L177) | [183](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L183)

```solidity
File: contracts/MainHelper.sol

285: return IERC20(_tokenAddress).balanceOf(
            address(this)
        );
```
[285](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L285)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

164: return PENDLE_CONTROLLER.updateRewardTokens(
            UNDERLYING_PENDLE_MARKET
        );
172: PENDLE_CONTROLLER.overWriteIndexAll(
            UNDERLYING_PENDLE_MARKET
        );
182: PENDLE_CONTROLLER.overWriteIndex(
            UNDERLYING_PENDLE_MARKET,
            _index
        );
191: PENDLE_CONTROLLER.overWriteAmounts(
            UNDERLYING_PENDLE_MARKET
        );
206: PENDLE_CONTROLLER.increaseReservedForCompound(
                    UNDERLYING_PENDLE_MARKET,
                    rewardsOutsideArray
                );
222: address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(
            UNDERLYING_PENDLE_MARKET
        );
226: uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
            UNDERLYING_PENDLE_MARKET
        );
243: PENDLE_MARKET.redeemRewards(
                    PENDLE_POWER_FARM_CONTROLLER
                );
310: return PENDLE_MARKET.balanceOf(
            PENDLE_POWER_FARM_CONTROLLER
        );
320: return PENDLE_MARKET.activeBalance(
            PENDLE_POWER_FARM_CONTROLLER
        );
353: PENDLE_MARKET.increaseObservationsCardinalityNext(
                storageMarket.observationCardinalityNext + 1
            );
365: PENDLE_CONTROLLER.withdrawLp(
            UNDERLYING_PENDLE_MARKET,
            _to,
            _amount
        );
380: return PENDLE_MARKET.userReward(
            _rewardToken,
            _user
        );
```
[164](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L164) | [172](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L172) | [182](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L182) | [191](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L191) | [206](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L206) | [222](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L222) | [226](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L226) | [243](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L243) | [310](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L310) | [320](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L320) | [353](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L353) | [365](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L365) | [380](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L380)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

317: return IPendleChild(PENDLE_CHILD).withdrawExactShares(
            WISE_LENDING.withdrawExactShares(
                _nftId,
                PENDLE_CHILD,
                _lendingShares
            )
        );
493: ) = IPendleChild(PENDLE_CHILD).depositExactAmount(
            netLpOut
        );
```
[317](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L317) | [493](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L493)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol

423: ) = IUniswapV3Pool(_oracle).observe(
            secondsAgo
        );
```
[423](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L423)

```solidity
File: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

149: uint256 withdrawnAmount = IPendlePowerFarmToken(pendleChild).withdrawExactShares(
            _pendleChildShares
        );
186: uint256 balance = IPendleMarket(_pendleMarket).balanceOf(
            address(this)
        );
224: address pendleChild = PENDLE_POWER_FARM_TOKEN_FACTORY.deploy(
            _pendleMarket,
            _tokenName,
            _symbolName,
            _maxCardinality
        );
```
[149](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L149) | [186](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L186) | [224](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L224)

```solidity
File: contracts/WrapperHub/AaveHub.sol

626: IERC20(tokenToSend).balanceOf(address(this))
        );
```
[626](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L626)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

249: aaveTokenAddresses = AAVE_HUB.aaveTokenAddress(
            borrowTokenAddresses
        );
```
[249](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L249)

```solidity
File: contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

108: if (FARMS_NFTS.ownerOf(_keyId) == _owner) {
            return true;
130: FARMS_NFTS.mintKey(
            _userAddress,
            _keyId
        );
```
[108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L108) | [130](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L130)

```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol

99: ) = ORACLE_PENDLE_PT.getOracleState(
            PENDLE_MARKET_ADDRESS,
            TWAP_DURATION
        );
```
[99](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L99)

```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol

104: ) = ORACLE_PENDLE_PT.getOracleState(
            PENDLE_MARKET_ADDRESS,
            TWAP_DURATION
        );
117: uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
            PENDLE_MARKET_ADDRESS,
            TWAP_DURATION
        );
```
[104](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L104) | [117](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L117)

```solidity
File: contracts/WrapperHub/Declarations.sol

87: WISE_SECURITY.checkOwnerPosition(
            _nftId,
            msg.sender
        );
99: WISE_LENDING.checkPositionLocked(
            _nftId,
            msg.sender
        );
```
[87](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L87) | [99](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L99)

```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol

85: ) = ORACLE_PENDLE_PT.getOracleState(
            PENDLE_MARKET_ADDRESS,
            DURATION
        );
98: uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
            PENDLE_MARKET_ADDRESS,
            DURATION
        );
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
        );
```
[102](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L102)

```solidity
File: contracts/TransferHub/WrapperHelper.sol

43: WETH.withdraw(
            _value
        );
```
[43](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol#L43)
</details>