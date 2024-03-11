# Gas Optimizations

**Note : _G-03_, _G-05_ and _G-17_ contains only those instances which were missed by bot. Since they are major gas savings so I included those missed instances**

## Table of Contents

- [G-01] [Cache function calls outside of the loop avoid 3 function call in loop saves significant gas](#g-01-cache-function-calls-outside-of-the-loop-avoid-3-function-calls-in-loop)

- [G-02] [State variables can be cached outside of the loop instead of re-reading them on each iteration](#g-02-state-variables-can-be-cached-outside-of-the-loop-instead-of-re-reading-them-on-each-iteration)

- [G-03] [Reduce the size of struct variable and pack them together to save storage slots(_Instances Missed by bot_)(**Gas Saved ~2000Gas**)](#g-03-reduce-the-size-of-struct-variables-and-pack-them-together-to-save-storage-slotsinstances-missed-by-botgas-saved--2000-gas)

- [G-04] [Re-arrange state variable order to save storage slots (**Saves ~2000 Gas**)](#g-04-re-arrange-state-variable-order-to-save-storage-slots-saves-2000-gas)

- [G-05] [State variables can be packed by truncating timestamp(_Instance Missed by bot_)(**Gas Saved ~4000 GAS**)](#g-05-state-variables-can-be-packed-by-truncating-timestampinstance-missed-by-botgas-saved-4000-gas)
- [G-06] [State variables only set in the constructor should be declared immutable(**Gas Saved ~4200 Gas**)](#g-06-state-variables-only-set-in-the-constructor-should-be-declared-immutable)
- [G-07] [Remove unused immutable/constant variables](#g-07-remove-unused-immutableconstant-variables)
- [G-08] [Refactor `PtOracleDerivative::latestAnswer` function to fail early saves 2 external call](#g-08-refactor-ptoraclederivativelatestanswer-function-to-fail-early-saves-2-external-call)
- [G-09] [Refactor `PtOraclePure::latestAnswer` function to fail early saves 1 external call](#g-09-refactor-ptoraclepurelatestanswer-function-to-fail-early-saves-1-external-call)
- [G-10] [Use above assigned to state variable values directly instead of reading them from storage saves SLOADs](#g-10-use-above-assigned-to-state-variable-values-directly-instead-of-reading-them-from-storage-saves-sloads)
- [G-11] [State variables can be packed into fewer storage slot by reducing their size (saves ~8000 Gas)](#g-11-state-variables-can-be-packed-into-fewer-storage-slot-by-reducing-their-size-saves-8000-gas)
- [G-12] [Use directly `msg.sender` instead of reading address from `storage` if both are equal (Saves: ~900 Gas)](#g-12-use-directly-msgsender-instead-of-reading-address-from-storage-if-both-are-equal-saves-900-gas)
- [G-13] [Switch the order of if statements to save gas](#g-13-switch-the-order-of-if-statements-to-save-gas)
- [G-14] [Refactor `CallOptionalReturn::_callOptionalReturn` function to save gas](#g-14-refactor-calloptionalreturn_calloptionalreturn-function-to-save-gas)
- [G-15] [Don't cache state variable if it is used only once](#g-15-dont-cache-state-variable-if-it-is-used-only-once)
- [G-16] [Remove redundant if statements](#g-16-remove-redundant-if-statements)
- [G-17] [State variables should be cached in stack variables rather than re-reading them from storage(Instances Missed by Bot) (Gas Saved ~400 Gas)](#g-17-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storageinstances-missed-by-bot-gas-saved-400-gas)

## Auditor's Disclaimer :

All these findings are good findings and 100% safe to implement at no security/logic risk. They all are found by thorough manual review.

## [G-01] Cache function calls outside of the loop avoid 3 function calls in loop

Since these functions return same value in loop's all iterations and their return value not depending on loop's operations or index . So their return value can be cached outside instead of re-calling them on every iterations saves very significant gas.

### Cache `_getActiveBalance()`, `totalLpAssets()` and `_getBalanceLpBalanceController()` outside of the loop

```solidity
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

236: while (i < l) {
...
277:  uint256 activeBalance = _getActiveBalance();
278:  uint256 totalLpAssetsCurrent = totalLpAssets();
279:  uint256 lpBalanceController = _getBalanceLpBalanceController();
280:
281:       bool scaleNecessary = totalLpAssetsCurrent < lpBalanceController;
282:
283:        rewardsOutsideArray[i] = scaleNecessary
284:             ? indexDiff
285:              * activeBalance
286:              * totalLpAssetsCurrent
287:              / lpBalanceController
288:              / PRECISION_FACTOR_E18
289:                : indexDiff
290:                    * activeBalance
291:                    / PRECISION_FACTOR_E18;
```

[PendlePowerFarmToken.sol#L236-L291](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L236-L291)

**Recommended Mitigation Steps:**

```diff
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

+277:  uint256 activeBalance = _getActiveBalance();
+278:  uint256 totalLpAssetsCurrent = totalLpAssets();
+279:  uint256 lpBalanceController = _getBalanceLpBalanceController();

236: while (i < l) {
...
-277:  uint256 activeBalance = _getActiveBalance();
-278:  uint256 totalLpAssetsCurrent = totalLpAssets();
-279:  uint256 lpBalanceController = _getBalanceLpBalanceController();
280:
281:       bool scaleNecessary = totalLpAssetsCurrent < lpBalanceController;
282:
283:        rewardsOutsideArray[i] = scaleNecessary
284:             ? indexDiff
285:              * activeBalance
286:              * totalLpAssetsCurrent
287:              / lpBalanceController
288:              / PRECISION_FACTOR_E18
289:                : indexDiff
290:                    * activeBalance
291:                    / PRECISION_FACTOR_E18;
```

## [G-02] State variables can be cached outside of the loop instead of re-reading them on each iteration

Caching storage var. outside loop is very efficient when their value doesn't depends on index/loop operations and fixed for every iteration. Saves Gwarmsload on each iteration.

### Cache `PENDLE_POWER_FARM_CONTROLLER` and `PENDLE_MARKET` to save `4 SLOADs per iteration`

```solidity
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

236: while (i < l) {
237:     UserReward memory userReward = _getUserReward(
238:         rewardTokens[i],
239:         PENDLE_POWER_FARM_CONTROLLER
240:     );
241:
242:     if (userReward.accrued > 0) {
243:         PENDLE_MARKET.redeemRewards(
244:             PENDLE_POWER_FARM_CONTROLLER
245:         );
246:
247:         userReward = _getUserReward(
248:             rewardTokens[i],
249:             PENDLE_POWER_FARM_CONTROLLER
250:         );
251:     }


```

[PendlePowerFarmToken.sol#L236-L251](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L236C9-L251C14)

**Recommended Mitigation Steps:**

```diff
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

+    address _PENDLE_POWER_FARM_CONTROLLER = PENDLE_POWER_FARM_CONTROLLER;
+    IPendleMarket  _PENDLE_MARKET = PENDLE_MARKET;
236: while (i < l) {
237:     UserReward memory userReward = _getUserReward(
238:         rewardTokens[i],
-239:         PENDLE_POWER_FARM_CONTROLLER
+239:         _PENDLE_POWER_FARM_CONTROLLER
240:     );
241:
242:     if (userReward.accrued > 0) {
-243:         PENDLE_MARKET.redeemRewards(
-244:             PENDLE_POWER_FARM_CONTROLLER
+243:         _PENDLE_MARKET.redeemRewards(
+244:             _PENDLE_POWER_FARM_CONTROLLER
245:         );
246:
247:         userReward = _getUserReward(
248:             rewardTokens[i],
-249:             PENDLE_POWER_FARM_CONTROLLER
+249:             _PENDLE_POWER_FARM_CONTROLLER
250:         );
251:     }

```

## [G-03] Reduce the size of struct variables and pack them together to save storage slots(_Instances Missed by bot_)(Gas Saved ~ 2000 Gas)

`utilization` is only set in `MainHelper::_updateUtilization` function where it's value come from it's `_getValueUtilization` function which will return the value `PRECISION_FACTOR_E18 - (PRECISION_FACTOR_E18 * totalPool / pseudoPool)` which will never be more than **1e18** and it will be assigned to `utilization`. So `utilization` value will be lesser than 1e18 always. So it is completely safe to reduce the size of it to `uint64` to hold it's value.

`poolFee` is set in `WiseLending::setPoolFee` function which is called by only `FeeManager::setPoolFee` function where newFee is passed through a check that newFee can not be more than **1e18**. So it is completely safe to reduce the size of it to `uint64` to hold it's value.

### So `utilization` and `poolFee` can be packed together into 1 slot saves 1 Storage Slot.(~2000 Gas)

At every key's value in `globalPoolData` mapping in WiseLendingDeclaration.sol.

All these contracts with declaration inherited into WiseLending.sol finally through MainHelper.sol and WiseLendingDeclaration and will be deployed as one WiseLending.sol.

```solidity
File : WiseLendingDeclaration.sol

212:    struct GlobalPoolEntry {
213:        uint256 totalPool;
214:        uint256 utilization;
215:        uint256 totalBareToken;
216:        uint256 poolFee;
217:    }

```

[WiseLendingDeclaration.sol#L212C1-L217C6](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L212C1-L217C6)

**Recommended Mitigation Steps:**

```diff
File : WiseLendingDeclaration.sol

212:    struct GlobalPoolEntry {
213:        uint256 totalPool;
- 214:        uint256 utilization;
215:        uint256 totalBareToken;
- 216:        uint256 poolFee;
+ 216:        uint64 poolFee;
+             uint64 utilization;
217:    }

```

## [G-04] Re-arrange state variable order to save storage slots (Saves ~2000 Gas)

### We can re-arrange the order of `powerFarmCheck` to save 1 storage slot (~2000 Gas)

```solidity
File : WiseLendingDeclaration.sol

162: address internal AAVE_HUB_ADDRESS;
...
186:  bool internal powerFarmCheck;

```

[WiseLendingDeclaration.sol#L186](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L186), [WiseLendingDeclaration.sol#L162](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L162)

**Recommended Mitigation Steps:**

```diff
File : WiseLendingDeclaration.sol

162: address internal AAVE_HUB_ADDRESS;
+186:  bool internal powerFarmCheck;
...
-186:  bool internal powerFarmCheck;

```

## [G-05] State variables can be packed by truncating timestamp(Instance Missed by bot)(Gas Saved ~4000 GAS)

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs), we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

**NOTE:-------------------**

### `lastUpdateGlobal` and `address master` can be packed in a single slot `SAVES: ~2000 Gas, 1 SLOT`

Since `lastUpdateGlobal` holds time. So `uint48` is more than sufficient to hold any realistic time.

```solidity
File : DerivativeOracles/CustomOracleSetup.sol

7:  address public master;
8:  uint256 public lastUpdateGlobal;

```

[CustomOracleSetup.sol#L7-L8](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L7-L8)

**Recommended Mitigation Steps:**

```diff
File : DerivativeOracles/CustomOracleSetup.sol

7:  address public master;
+8:  uint48 public lastUpdateGlobal;
-8:  uint256 public lastUpdateGlobal;

```

### `lastInteraction` and `MAX_CARDINALITY` can be packed in single slot SAVES: ~2000 GAS, 1 Slot

Since `lastInteraction` hold block.timestamp. So uint64 is more than sufficient to hold time. uint64 can valid to `584942417355.07202148` years.

```solidity
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
47:  uint16 public MAX_CARDINALITY;
...
50:  uint256 public lastInteraction;

```

[PendlePowerFarmToken.sol#L47C1-L50C36](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L47C1-L50C36)

**Recommended Mitigation Steps:**

```diff
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
47:  uint16 public MAX_CARDINALITY;
+50:  uint64 public lastInteraction;
...
-50:  uint256 public lastInteraction;

```

### `INITIAL_TIME_STAMP` and `UNDERLYING_PENDLE_MARKET` can be packed in a single slot Saves: ~2000 Gas, 1 Slot

Since `INITIAL_TIME_STAMP` hold block.timestamp. So uint64 is more than sufficient to hold time. uint64 can valid to `584942417355.07202148` years.

```solidity
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

28:   address public UNDERLYING_PENDLE_MARKET;
...
61:  uint256 private INITIAL_TIME_STAMP;

```

[PendlePowerFarmToken.sol#L61](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L61), [PendlePowerFarmToken.sol#L28](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L28)

**Recommended Mitigation Steps:**

```diff
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

28:   address public UNDERLYING_PENDLE_MARKET;
+61:  uint64 private INITIAL_TIME_STAMP
...
-61:  uint256 private INITIAL_TIME_STAMP;

```

## [G-06] State variables only set in the constructor should be declared immutable

State variables only set in the constructor should be declared immutable as it avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

### `priceFeedPendleLpOracle` and `pendleChildToken` can be marked immutable SAVES:

```solidity
File : DerivativeOracles/PendleChildLpOracle.sol

15: IPriceFeed public priceFeedPendleLpOracle;
16: IPendleChildToken public pendleChildToken;

```

[PendleChildLpOracle.sol#L15-L16](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L15-L16)

**Recommended Mitigation Steps:**

```diff
File : DerivativeOracles/PendleChildLpOracle.sol

-15: IPriceFeed public priceFeedPendleLpOracle;
-16: IPendleChildToken public pendleChildToken;

+15: IPriceFeed public immutable priceFeedPendleLpOracle;
+16: IPendleChildToken public immutable pendleChildToken;

```

## [G-07] Remove unused immutable/constant variables

### Remove `PENDLE_MARKET`, `PENDLE_SY` and `POW_FEED_DECIMALS` since these variables are never used.

```solidity
File : DerivativeOracles/PendleLpOracle.sol

32: constructor(
...
51:        PENDLE_MARKET = IPendleMarket(
52:            _pendleMarketAddress
53:        );
...
...
59: IPendleMarket public immutable PENDLE_MARKET;
...
62: IPendleSy public immutable PENDLE_SY;
...
67: uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;

```

[PendleLpOracle.sol#L51-L67](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L51-L67)

**Recommended Mitigation Steps:**

```diff
File : DerivativeOracles/PendleLpOracle.sol

32: constructor(
...
-51:        PENDLE_MARKET = IPendleMarket(
-52:            _pendleMarketAddress
-53:        );
...
...
-59: IPendleMarket public immutable PENDLE_MARKET;
...
-62: IPendleSy public immutable PENDLE_SY;
...
-67: uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;

```

## [G-08] Refactor `PtOracleDerivative::latestAnswer` function to fail early saves 2 external call

`answerUsdFeed` and `answerEthUsdFeed` are used after the if statements. So make external call after if statement.

```solidity
File : DerivativeOracles/PtOracleDerivative.sol

81: function latestAnswer()
82:     public
83:     view
84:     returns (uint256)
85: {
86:     (
87:         ,
88:         int256 answerUsdFeed,
89:         ,
90:         ,
91:     ) = USD_FEED_ASSET.latestRoundData();
92:
93:     (
94:         ,
95:         int256 answerEthUsdFeed,
96:         ,
97:         ,
98:     ) = ETH_FEED_ASSET.latestRoundData();
99:
100:     (
101:         bool increaseCardinalityRequired,
102:         ,
103:         bool oldestObservationSatisfied
104:     ) = ORACLE_PENDLE_PT.getOracleState(
105:         PENDLE_MARKET_ADDRESS,
106:         TWAP_DURATION
107:     );
108:
109:     if (increaseCardinalityRequired == true) {
110:         revert CardinalityNotSatisfied();
111:     }
112:
113:     if (oldestObservationSatisfied == false) {
114:         revert OldestObservationNotSatisfied();
115:     }
116:
117:     uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
118:         PENDLE_MARKET_ADDRESS,
119:         TWAP_DURATION
120:     );
121:
122:     return uint256(answerUsdFeed)
123:         * PRECISION_FACTOR_E18
124:         / POW_USD_FEED
125:         * POW_ETH_USD_FEED
126:         / uint256(answerEthUsdFeed)
127:         * ptToAssetRate
128:         / PRECISION_FACTOR_E18;
129: }

```

[PtOracleDerivative.sol#L81-L129](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L81C1-L129C6)

**Recommended Mitigation Steps:**

```diff
File : DerivativeOracles/PtOracleDerivative.sol

81: function latestAnswer()
82:     public
83:     view
84:     returns (uint256)
85: {
-86:     (
-87:         ,
-88:         int256 answerUsdFeed,
-89:         ,
-90:         ,
-91:     ) = USD_FEED_ASSET.latestRoundData();
-92:
-93:     (
-94:         ,
-95:         int256 answerEthUsdFeed,
-96:         ,
-97:         ,
-98:     ) = ETH_FEED_ASSET.latestRoundData();
99:
100:     (
101:         bool increaseCardinalityRequired,
102:         ,
103:         bool oldestObservationSatisfied
104:     ) = ORACLE_PENDLE_PT.getOracleState(
105:         PENDLE_MARKET_ADDRESS,
106:         TWAP_DURATION
107:     );
108:
109:     if (increaseCardinalityRequired == true) {
110:         revert CardinalityNotSatisfied();
111:     }
112:
113:     if (oldestObservationSatisfied == false) {
114:         revert OldestObservationNotSatisfied();
115:     }
+86:     (
+87:         ,
+88:         int256 answerUsdFeed,
+89:         ,
+90:         ,
+91:     ) = USD_FEED_ASSET.latestRoundData();
+92:
+93:     (
+94:         ,
+95:         int256 answerEthUsdFeed,
+96:         ,
+97:         ,
+98:     ) = ETH_FEED_ASSET.latestRoundData();
116:
117:     uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
118:         PENDLE_MARKET_ADDRESS,
119:         TWAP_DURATION
120:     );
121:
122:     return uint256(answerUsdFeed)
123:         * PRECISION_FACTOR_E18
124:         / POW_USD_FEED
125:         * POW_ETH_USD_FEED
126:         / uint256(answerEthUsdFeed)
127:         * ptToAssetRate
128:         / PRECISION_FACTOR_E18;
129: }

```

## [G-09] Refactor `PtOraclePure::latestAnswer` function to fail early saves 1 external call

`answerFeed` is used after the if statements so it si better to make external call after these checks.

```solidity
File : DerivativeOracles/PtOraclePure.sol

68: function latestAnswer()
69:     public
70:     view
71:     returns (uint256)
72: {
73:     (
74:         ,
75:         int256 answerFeed,
76:         ,
77:         ,
78:
79:     ) = FEED_ASSET.latestRoundData();
80:
81:     (
82:         bool increaseCardinalityRequired,
83:         ,
84:         bool oldestObservationSatisfied
85:     ) = ORACLE_PENDLE_PT.getOracleState(
86:         PENDLE_MARKET_ADDRESS,
87:         DURATION
88:     );
89:
90:     if (increaseCardinalityRequired == true) {
91:         revert CardinalityNotSatisfied();
92:     }
93:
94:     if (oldestObservationSatisfied == false) {
95:         revert OldestObservationNotSatisfied();
96:     }
97:
98:     uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
99:         PENDLE_MARKET_ADDRESS,
100:         DURATION
101:     );
102:
103:     return uint256(answerFeed)
104:         * PRECISION_FACTOR_E18
105:         / POW_FEED
106:         * ptToAssetRate
107:         / PRECISION_FACTOR_E18;
108: }

```

[PtOraclePure.sol#L68-L108](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L68C1-L108C6)

**Recommended Mitigation Steps:**

```diff
File : DerivativeOracles/PtOraclePure.sol

68: function latestAnswer()
69:     public
70:     view
71:     returns (uint256)
72: {
-73:     (
-74:         ,
-75:         int256 answerFeed,
-76:         ,
-77:         ,
-78:
-79:     ) = FEED_ASSET.latestRoundData();
80:
81:     (
82:         bool increaseCardinalityRequired,
83:         ,
84:         bool oldestObservationSatisfied
85:     ) = ORACLE_PENDLE_PT.getOracleState(
86:         PENDLE_MARKET_ADDRESS,
87:         DURATION
88:     );
89:
90:     if (increaseCardinalityRequired == true) {
91:         revert CardinalityNotSatisfied();
92:     }
93:
94:     if (oldestObservationSatisfied == false) {
95:         revert OldestObservationNotSatisfied();
96:     }
+73:     (
+74:         ,
+75:         int256 answerFeed,
+76:         ,
+77:         ,
+78:
+79:     ) = FEED_ASSET.latestRoundData();
97:
98:     uint256 ptToAssetRate = ORACLE_PENDLE_PT.getPtToAssetRate(
99:         PENDLE_MARKET_ADDRESS,
100:         DURATION
101:     );
102:
103:     return uint256(answerFeed)
104:         * PRECISION_FACTOR_E18
105:         / POW_FEED
106:         * ptToAssetRate
107:         / PRECISION_FACTOR_E18;
108: }
```

## [G-10] Use above assigned to state variable values directly instead of reading them from storage saves SLOADs

```solidity
File : FeeManager/DeclarationsFeeManager.sol

90:   incentiveOwnerA = 0xf69A0e276664997357BF987df83f32a1a3F80944;
91:   incentiveOwnerB = 0x8f741ea9C9ba34B5B8Afc08891bDf53faf4B3FE7;
92:
93:   incentiveUSD[incentiveOwnerA] = 98000 * PRECISION_FACTOR_E18;
94:   incentiveUSD[incentiveOwnerB] = 106500 * PRECISION_FACTOR_E18;

```

[DeclarationsFeeManager.sol#L90-L94](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L90-L94)

**Recommended Mitigation Steps:**

```diff
File : FeeManager/DeclarationsFeeManager.sol

90:   incentiveOwnerA = 0xf69A0e276664997357BF987df83f32a1a3F80944;
91:   incentiveOwnerB = 0x8f741ea9C9ba34B5B8Afc08891bDf53faf4B3FE7;
92:
-93:   incentiveUSD[incentiveOwnerA] = 98000 * PRECISION_FACTOR_E18;
-94:   incentiveUSD[incentiveOwnerB] = 106500 * PRECISION_FACTOR_E18;
+93:   incentiveUSD[0xf69A0e276664997357BF987df83f32a1a3F80944] = 98000 * PRECISION_FACTOR_E18;
+94:   incentiveUSD[0x8f741ea9C9ba34B5B8Afc08891bDf53faf4B3FE7] = 106500 * PRECISION_FACTOR_E18;

```

## [G-11] State variables can be packed into fewer storage slot by reducing their size (saves ~8000 Gas)

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs), we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

**NOTE:-----------------**

### `SAVE: ~8000 GAS, 4 SLOT`

### `paybackIncentive` and `incentiveMaster` can be packed in single slot `SAVES: ~2000 GAS, 1 SLOT`

Since `paybackIncentive` can max up to 1e18 because of:

```solidity
File : FeeManager/DeclarationsFeeManager.sol

88: paybackIncentive = 5 * PRECISION_FACTOR_E16;
```

[DeclarationsFeeManager.sol#L88](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L88)

So we can easily reduce `paybackIncentive` to uint64 and pack with address `incentiveMaster`. Since uint64 can easily hold up to `1.8446744073709551615 × 10^19`

```solidity
File : FeeManager/DeclarationsFeeManager.sol

120: uint256 public paybackIncentive;
...
126: address public incentiveMaster;

```

[DeclarationsFeeManager.sol#L120-L126](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L120C1-L126C36)

**Recommended Mitigation Steps:**

```diff
File : FeeManager/DeclarationsFeeManager.sol

-120: uint256 public paybackIncentive;
...
126: address public incentiveMaster;
+120: uint64 public paybackIncentive;

```

### `collateralFactor` and `allowEnter` can be packed in single slot `SAVE:~2000 Gas, 1 Slot`

`collateralFactor` can't more than 1e18 deu to this check:

```solidity

  if (_collateralFactor > PRECISION_FACTOR_E18) {
      revert CollateralFactorTooHigh();

```

[PendlePowerFarmDeclarations.sol#L243C1-L245C9](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L243C1-L245C9)

So we can easily reduce `collateralFactor` to uint64 and pack with bool `allowEnter`. Since uint64 can easily hold up to `1.8446744073709551615 × 10^19`

```solidity
File : PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

46:  bool public allowEnter;
47:
48:    // Ratio between borrow and lend
49:    uint256 public collateralFactor;

```

[PendlePowerFarmDeclarations.sol#L46C1-L49C37](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L46C1-L49C37)

**Recommended Mitigation Steps:**

```diff
File : PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

46:  bool public allowEnter;
+49:    uint64 public collateralFactor;
47:
48:    // Ratio between borrow and lend
-49:    uint256 public collateralFactor;
```

### `mintFee`, `MAX_CARDINALITY` and `PENDLE_POWER_FARM_CONTROLLER` can be packed in a single slot Saves: ~4000 Gas, 2 Slot

Since `mintFee` can't more than 10000 because of this check:

```solidity
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

if (_newFee > MAX_MINT_FEE) {
            revert FeeTooHigh();
        }

        mintFee = _newFee;

```

[PendlePowerFarmToken.sol#L598C9-L602C27](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L598C9-L602C27)

So we easily reduce `mintFee` to uint16 and pack both `mintFee` and `MAX_CARDINALITY` with address `PENDLE_POWER_FARM_CONTROLLER`. uint16 can hold up to 65535.

```solidity
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

29:   address public PENDLE_POWER_FARM_CONTROLLER;
...
47:    uint16 public MAX_CARDINALITY;
48:
49:    uint256 public mintFee;

```

[PendlePowerFarmToken.sol#L47C1-L49C28](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L47C1-L49C28)

**Recommended Mitigation Steps:**

```diff
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

29: address public PENDLE_POWER_FARM_CONTROLLER;
+47:    uint16 public MAX_CARDINALITY;
+49:    uint16 public mintFee;
...
-47:    uint16 public MAX_CARDINALITY;
48:
-49:    uint256 public mintFee;

```

### `maxFeeETH`, `maxFeeFarmETH`, `baseRewardLiquidation` and `baseRewardLiquidationFarm` can be packed in two slots instead of four `SAVES: ~4000 Gas, 2 SLot`

`maxFeeETH` has max value `100e18`.
`maxFeeFarmETH` has max value `100e18`.
`baseRewardLiquidation` has max value `11e16`.
`baseRewardLiquidationFarm` has max value `4e16`.

```solidity
File : WiseSecurity/WiseSecurityDeclarations.sol

 // Max reward ETH for liquidator power farm liquidation
252:    uint256 public maxFeeETH;

    // Max reward ETH for liquidator normal liquidation
255:    uint256 public maxFeeFarmETH;

    // Base reward for liquidator normal liquidation
258:    uint256 public baseRewardLiquidation;

    // Base reward for liquidator power farm liquidation
261:    uint256 public baseRewardLiquidationFarm;

```

[WiseSecurityDeclarations.sol#L251C4-L261C46](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L251C4-L261C46)

**Recommended Mitigation Steps:**

```diff
File : WiseSecurity/WiseSecurityDeclarations.sol

 // Max reward ETH for liquidator power farm liquidation
-252:    uint256 public maxFeeETH;

    // Max reward ETH for liquidator normal liquidation
-255:    uint256 public maxFeeFarmETH;

    // Base reward for liquidator normal liquidation
-258:    uint256 public baseRewardLiquidation;

    // Base reward for liquidator power farm liquidation
-261:    uint256 public baseRewardLiquidationFarm;

+252:    uint128 public maxFeeETH;

    // Max reward ETH for liquidator normal liquidation
+255:    uint128 public maxFeeFarmETH;

    // Base reward for liquidator normal liquidation
+258:    uint128 public baseRewardLiquidation;

    // Base reward for liquidator power farm liquidation
+261:    uint128 public baseRewardLiquidationFarm;

```

## [G-12] Use directly `msg.sender` instead of reading address from `storage` if both are equal (Saves: ~900 Gas)

### 4 Instances

1. ### SAVES: 2 SLOADs (~200 Gas)

After the if statement `msg.sender` and `proposedIncentiveMaster` are equal so we can use directly `msg.sender` instead of `proposedIncentiveMaster` also we can emit `msg.sender` instead of `incentiveMaster` because both are same.

```solidity
File : FeeManager/FeeManager.sol

194:  function claimOwnershipIncentiveMaster()
195:     external
196:  {
197:     if (msg.sender != proposedIncentiveMaster) {
198:         revert NotAllowed();
199:     }
200:
201:    incentiveMaster = proposedIncentiveMaster;
202:    proposedIncentiveMaster = ZERO_ADDRESS;
203:
204:    emit ClaimedOwnershipIncentiveMaster(
205:        incentiveMaster,
206:        block.timestamp
207:     );
208:  }

```

[FeeManager.sol#L194-L208](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L194C1-L208C6)

**Recommended Mitigation Steps:**

```diff
File : FeeManager/FeeManager.sol

194:  function claimOwnershipIncentiveMaster()
195:     external
196:  {
197:     if (msg.sender != proposedIncentiveMaster) {
198:         revert NotAllowed();
199:     }
200:
-201:    incentiveMaster = proposedIncentiveMaster;
+201:    incentiveMaster = msg.sender;
202:    proposedIncentiveMaster = ZERO_ADDRESS;
203:
204:    emit ClaimedOwnershipIncentiveMaster(
-205:        incentiveMaster,
+205:        msg.sender,
206:        block.timestamp
207:     );
208:  }

```

2. ### SAVES: 3 SLOADs (~300 Gas)

After the first if statement it is proved that `msg.sender` and `incentiveOwnerA` are equal so we can use `msg.sender` instead of `incentiveOwnerA`. It can save `3 sloads`.

```solidity
File : FeeManager/FeeManager.sol

315: function changeIncentiveUSDA(
316:     address _newOwner
317: )
318:     external
319: {
320:     if (msg.sender != incentiveOwnerA) {
321:         revert NotAllowed();
322:     }
323:
324:     if (_newOwner == incentiveOwnerA) {
325:         revert NotAllowed();
326:     }
327:
...
331:
332:     incentiveUSD[_newOwner] = incentiveUSD[
333:         incentiveOwnerA
334:     ];
335:
336:     delete incentiveUSD[
337:         incentiveOwnerA
338:     ];
```

[FeeManager.sol#L315-L338](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L315C1-L338C11)

**Recommended Mitigation Steps:**

```diff
File : FeeManager/FeeManager.sol

315: function changeIncentiveUSDA(
316:     address _newOwner
317: )
318:     external
319: {
320:     if (msg.sender != incentiveOwnerA) {
321:         revert NotAllowed();
322:     }
323:
-324:     if (_newOwner == incentiveOwnerA) {
+324:     if (_newOwner == msg.sender) {
325:         revert NotAllowed();
326:     }
327:
...
331:
332:     incentiveUSD[_newOwner] = incentiveUSD[
-333:         incentiveOwnerA
+333:         msg.sender
334:     ];
335:
336:     delete incentiveUSD[
-337:         incentiveOwnerA
+337:         msg.sender
338:     ];
```

3. ### SAVES: 3 SLOADs (~300 Gas)

After the first if statement it is proved that `msg.sender` and `incentiveOwnerB` are equal so we can use `msg.sender` instead of `incentiveOwnerB`. It can save `3 sloads`.

```solidity
File : FeeManager/FeeManager.sol

352: function changeIncentiveUSDB(
353:     address _newOwner
354: )
355:     external
356: {
357:     if (msg.sender != incentiveOwnerB) {
358:         revert NotAllowed();
359:     }
360:
361:     if (_newOwner == incentiveOwnerA) {
362:         revert NotAllowed();
363:     }
364:
365:     if (_newOwner == incentiveOwnerB) {
366:         revert NotAllowed();
367:     }
368:
369:     incentiveUSD[_newOwner] = incentiveUSD[
370:         incentiveOwnerB
371:     ];
372:
373:     delete incentiveUSD[
374:         incentiveOwnerB
375:     ];

```

[FeeManager.sol#L352-L375](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L352C1-L375C11)

**Recommended Mitigation Steps:**

```diff
File : FeeManager/FeeManager.sol

352: function changeIncentiveUSDB(
353:     address _newOwner
354: )
355:     external
356: {
357:     if (msg.sender != incentiveOwnerB) {
358:         revert NotAllowed();
359:     }
360:
361:     if (_newOwner == incentiveOwnerA) {
362:         revert NotAllowed();
363:     }
364:
-365:     if (_newOwner == incentiveOwnerB) {
+365:     if (_newOwner == msg.sender) {
366:         revert NotAllowed();
367:     }
368:
369:     incentiveUSD[_newOwner] = incentiveUSD[
-370:         incentiveOwnerB
+370:         msg.sender
371:     ];
372:
373:     delete incentiveUSD[
-374:         incentiveOwnerB
+374:         msg.sender
375:     ];

```

4. ### SAVES: 1 SLOAD(~100 Gas)

Since `onlyProposed` modifier check insure that `proposedMaster == msg.sender`:

```solidity
File : OwnableMaster.sol

   modifier onlyProposed() {
        _onlyProposed();
        _;
    }

    ...

    function _onlyProposed()
        private
        view
    {
        if (msg.sender == proposedMaster) {
            return;
        }

        revert NotProposed();

```

[OwnableMaster.sol#L37-L46](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L37C1-L46C6)

So we can use msg.sender instead of `proposedMaster`. Saves 1 SLOAD(~100 Gas)

```solidity
File : OwnableMaster.sol

92:  function claimOwnership()
93:    external
94:     onlyProposed
95:  {
96:     master = proposedMaster;
97:  }

```

[OwnableMaster.sol#L92-L97](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L92C1-L97C6)

**Recommended Mitigation Steps:**

```diff
File : OwnableMaster.sol

92:  function claimOwnership()
93:    external
94:     onlyProposed
95:  {
-96:     master = proposedMaster;
+96:     master = msg.sender;
97:  }

```

## [G-13] Switch the order of if statements to save gas

Rearranging the order of the if statements to reduce gas costs associated with storage loads (SLOAD) when the conditions fail. The rationale is that by placing the condition with a lower gas cost before the condition with a higher gas cost, the likelihood of reaching the higher-cost operation is minimized, resulting in potential gas savings.

```solidity
File : FeeManager/FeeManager.sol

352: function changeIncentiveUSDB(
353:     address _newOwner
354: )
355:     external
356: {
357:     if (msg.sender != incentiveOwnerB) {
358:         revert NotAllowed();
359:     }
360:
361:     if (_newOwner == incentiveOwnerA) {
362:         revert NotAllowed();
363:     }
364:
365:     if (_newOwner == incentiveOwnerB) {
366:         revert NotAllowed();
367:     }

```

[FeeManager.sol#L352-L367](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L352-L367)

**Recommended Mitigation Steps:**

```diff
File : FeeManager/FeeManager.sol

352: function changeIncentiveUSDB(
353:     address _newOwner
354: )
355:     external
356: {
357:     if (msg.sender != incentiveOwnerB) {
358:         revert NotAllowed();
359:     }
+365:     if (_newOwner == incentiveOwnerB) {
+366:         revert NotAllowed();
+367:     }
360:
361:     if (_newOwner == incentiveOwnerA) {
362:         revert NotAllowed();
363:     }
364:
-365:     if (_newOwner == incentiveOwnerB) {
-366:         revert NotAllowed();
-367:     }

```

## [G-14] Refactor `CallOptionalReturn::_callOptionalReturn` function to save gas

The `results` variable is removed and the condition is directly integrated into the assignment of the call variable.
The condition `token.code.length > 0` is checked first in the assignment of call.
The `results` condition is combined with the overall condition simplifying the logic.
This refactor aims to improve gas efficiency by eliminating unnecessary variables and streamlining the code.

```solidity
File : TransferHub/CallOptionalReturn.sol

12: function _callOptionalReturn(
13:     address token,
14:     bytes memory data
15: )
16:     internal
17:     returns (bool call)
18: {
19:     (
20:         bool success,
21:         bytes memory returndata
22:     ) = token.call(
23:         data
24:     );
25:
26:     bool results = returndata.length == 0 || abi.decode(
27:         returndata,
28:         (bool)
29:     );
30:
31:     if (success == false) {
32:         revert();
33:     }
34:
35:     call = success
36:         && results
37:         && token.code.length > 0;
38: }


```

[CallOptionalReturn.sol#L12C1-L38C6](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L12C1-L38C6)

**Recommended Mitigation Steps:**

```diff
File : TransferHub/CallOptionalReturn.sol

12: function _callOptionalReturn(
13:     address token,
14:     bytes memory data
15: )
16:     internal
17:     returns (bool call)
18: {
19:     (
20:         bool success,
21:         bytes memory returndata
22:     ) = token.call(
23:         data
24:     );
25:
-26:     bool results = returndata.length == 0 || abi.decode(
-27:         returndata,
-28:         (bool)
-29:     );
30:
31:     if (success == false) {
32:         revert();
33:     }
34:
-35:     call = success
-36:         && results
-37:         && token.code.length > 0;

+35:     call = token.code.length > 0
+36:         && returndata.length == 0 || abi.decode(
+37:    returndata,(bool)
38: }

```

## [G-15] Don't cache state variable if it is used only once

### No need to cache `aaveTokenAddress[_underlyingAsset]`

#### Instance 1

```solidity
File : WrapperHub/AaveHelper.sol

122:   address aaveToken = aaveTokenAddress[
123:     _underlyingAsset
124:    ];
125:
126:   uint256 withdrawAmount = WISE_LENDING.withdrawOnBehalfExactShares(
127:        _nftId,
128:        aaveToken,
129:       _shareAmount
130:    )

```

[AaveHelper.sol#L122-L130](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L122C1-L130C10)

**Recommended Mitigation Steps:**

```diff
File : WrapperHub/AaveHelper.sol

-122:   address aaveToken = aaveTokenAddress[
-123:     _underlyingAsset
-124:    ];
125:
126:   uint256 withdrawAmount = WISE_LENDING.withdrawOnBehalfExactShares(
127:        _nftId,
-128:        aaveToken,
+128:        aaveTokenAddress[_underlyingAsset],
129:       _shareAmount
130:    )

```

#### Instance 2

```solidity
File :WrapperHub/AaveHub.sol

447:   address aaveToken = aaveTokenAddress[
448:      _underlyingAsset
449:    ];
...
464:    uint256 borrowSharesReduction = WISE_LENDING.paybackExactAmount(
465:        _nftId,
466:       aaveToken,
467:      actualAmountDeposit
468:      );

```

[AaveHub.sol#L447-L468](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L447C1-L468C11)

**Recommended Mitigation Steps:**

```diff
File :WrapperHub/AaveHub.sol

-447:   address aaveToken = aaveTokenAddress[
-448:      _underlyingAsset
-449:    ];
...
464:    uint256 borrowSharesReduction = WISE_LENDING.paybackExactAmount(
465:        _nftId,
-466:       aaveToken,
+466:       aaveTokenAddress[_underlyingAsset],
467:      actualAmountDeposit
468:      );

```

## [G-16] Remove redundant if statements

This `if (_lendingAddress == ZERO_ADDRESS)` check is redundant because if it is address(0) it is not going to pass from `IWiseLending(lendingAddress).WETH_ADDRESS()`.

```solidity
File : WrapperHub/Declarations.sol

51:  WrapperHelper(
52:       IWiseLending(
53:           _lendingAddress
54:       ).WETH_ADDRESS()
55:     )
56:  {
57:     if (_aaveAddress == ZERO_ADDRESS) {
58:         revert NoValue();
59:      }
60:
61:     if (_lendingAddress == ZERO_ADDRESS) {
62:          revert NoValue();
63:        }

```

[Declarations.sol#L51C9-L63C10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L51C9-L63C10)

**Recommended Mitigation Steps:**

```diff
File : WrapperHub/Declarations.sol

51:  WrapperHelper(
52:       IWiseLending(
53:           _lendingAddress
54:       ).WETH_ADDRESS()
55:     )
56:  {
57:     if (_aaveAddress == ZERO_ADDRESS) {
58:         revert NoValue();
59:      }
60:
-61:     if (_lendingAddress == ZERO_ADDRESS) {
-62:          revert NoValue();
-63:        }

```

## [G-17] State variables should be cached in stack variables rather than re-reading them from storage(Instances Missed by Bot) (Gas Saved ~400 Gas)

### `SAVE: `~400 GAS, 4 SLOAD`

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### `PENDLE_CONTROLLER` and `UNDERLYING_PENDLE_MARKET` can be cached to save 2 SLOAD (~200 Gas)

```solidity
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

222:    address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(
223:        UNDERLYING_PENDLE_MARKET
224:    );
225:
226:        uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
227:            UNDERLYING_PENDLE_MARKET
228:        );

```

[PendlePowerFarmToken.sol#L222-L228C](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L222C1-L228C11)

**Recommended Mitigation Steps:**

```diff
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

+       IPendleController _PENDLE_CONTROLLER = PENDLE_CONTROLLER;
+        address _UNDERLYING_PENDLE_MARKET = UNDERLYING_PENDLE_MARKET;

-222:    address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(
-223:        UNDERLYING_PENDLE_MARKET
+222:    address[] memory rewardTokens = _PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(
+223:        _UNDERLYING_PENDLE_MARKET
224:    );
225:
-226:        uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
-227:            UNDERLYING_PENDLE_MARKET
+226:        uint128[] memory lastIndex = _PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
+227:            _UNDERLYING_PENDLE_MARKET
228:        );

```

### Cache `lastInteraction` to save 1 SLOAD (~100 Gas)

```solidity
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

395:        if (block.timestamp == lastInteraction) {
396:            return 0;
397:        }
...
...
406:        uint256 additonalAssets = currentRate
407:            * (block.timestamp - lastInteraction)
```

[PendlePowerFarmToken.sol#L395-L407](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L395C1-L407C50)

**Recommended Mitigation Steps:**

```diff
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

+       uint256 _lastInteraction = lastInteraction;

-395:        if (block.timestamp == lastInteraction) {
+395:        if (block.timestamp == _lastInteraction) {
396:            return 0;
397:        }
...
...
406:        uint256 additonalAssets = currentRate
-407:            * (block.timestamp - lastInteraction)
+407:            * (block.timestamp - _lastInteraction)
```

### `underlyingLpAssetsCurrent` can be cached to save 1 SLOAD (~100 Gas)

```solidity
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

529: function depositExactAmount(
...

543:        uint256 shares = previewMintShares(
544:            _underlyingLpAssetAmount,
545:            underlyingLpAssetsCurrent
546:        );
...
577:     underlyingLpAssetsCurrent += _underlyingLpAssetAmount;

```

[PendlePowerFarmToken.sol#L529C5-L578C1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L529C5-L578C1)

**Recommended Mitigation Steps:**

```diff
File : PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

529: function depositExactAmount(
...
+        uint256 cached_underlyingLpAssetsCurrent = underlyingLpAssetsCurrent;
543:        uint256 shares = previewMintShares(
544:            _underlyingLpAssetAmount,
-545:            underlyingLpAssetsCurrent
+545:            cached_underlyingLpAssetsCurrent
546:        );
...
-577:     underlyingLpAssetsCurrent += _underlyingLpAssetAmount;
+577:     underlyingLpAssetsCurrent = cached_underlyingLpAssetsCurrent + _underlyingLpAssetAmount;

```
