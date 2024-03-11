WiseLending GAS OPTIMIZATIONS
--
INTRODUCTION
--
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime.

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include `@audit` tags in comments to facilitate issue explanations."



**Note: The issues addressed here were not reported by the bot, for packing variables, notes explaining the how and why are included**

## Gas report

### Table Of Contents
- [State variables can be packed into fewer storage slot (saves 5 SLOTS: 10.2k Gas)](#g-01-state-variables-can-be-packed-into-fewer-storage-slot-saves-5-slots-102k-gas)
    - [Pack the `globalRoundId` with address `master` by changing their order (saves 2.1k Gas)](#pack-the-globalroundid-with-address-master-by-changing-their-order-saves-21k-gas)
    - [Pack the `FEED_DECIMALS` with address `TWAP_DURATION` by changing their order (saves 2.1k Gas)](#pack-the-feed_decimals-with-address-twap_duration-by-changing-their-order-saves-21k-gas)
    - [Pack the `FEED_DECIMALS` with address `TWAP_DURATION` by changing their order (saves 2.1k Gas)](#pack-the-feed_decimals-with-address-twap_duration-by-changing-their-order-saves-21k-gas)
    - [Pack the `FEED_DECIMALS` with address `DURATION` by changing their order (saves 2.1k Gas)](#pack-the-feed_decimals-with-address-duration-by-changing-their-order-saves-21k-gas)
    - [Pack structs by reducing variable sizes to save storage slots (saves 2 SLOTS: 4.2k Gas)](#g-02-pack-structs-by-reducing-variable-sizes-to-save-storage-slots-saves-2-slots-42k-gas)
- [Cache calculations in loop to avoid re-calculating on each iteration](#g-03-cache-calculations-in-loop-to-avoid-re-calculating-on-each-iteration)
- [Minimize the external calls in while loop by storing the result call in a local variable before using in loop](#g-04-minimize-the-external-calls-in-while-loop-by-storing-the-result-call-in-a-local-variable-before-using-in-loop)
   -  [The `getPositionBorrowTokenByIndex` call within the loop is an external call](#the-getpositionborrowtokenbyindex-call-within-the-loop-is-an-external-call)
   -  [The `lendingPoolData(tokenAddress).collateralFactor` call within the loop is an external call](#the-lendingpooldatatokenaddresscollateralfactor-call-within-the-loop-is-an-external-call)

- [Move lesser gas costing require checks to the top](#g-05-move-lesser-gas-costing-require-checks-to-the-top)
- [Stack variables should be used in place of state variables rather than re-reading them from storage](#g-06-stack-variables-should-be-used-in-place-of-state-variables-rather-than-re-reading-them-from-storage)
- [Directly modifies the value in the storage location without reading the current value first](#g-07-directly-modifies-the-value-in-the-storage-location-without-reading-the-current-value-first)
- [State variables should be cached in stack variables rather than re-reading them from storage](#g-08-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage)
- [Do not cache global variable in local variable](#g-09-do-not-cache-global-variable-in-local-variable)
- [Refactor `_preparationToken` to avoid redundant storage read](#g-10-refactor-_preparationtoken-to-avoid-redundant-storage-read)
- [Storing similar computations to reduce the number of operations](#g-11-storing-similar-computations-to-reduce-the-number-of-operations)
- [Do not cache `uniTwapPoolInfo[_tokenAddress]` in memory if used only once](#g-12-do-not-cache-unitwappoolinfo_tokenaddress-in-memory-if-used-only-once)
- [Check amount for zero before send ETH (Missed by bot)](#g-13-check-amount-for-zero-before-send-eth-missed-by-bot)



**Note: The issues addressed here were not reported by the bot, for packing variables, notes explaining the how and why are included**

## [G-01] State variables can be packed into fewer storage slot (saves 5 SLOTS: 10.2k Gas)
Note: The bot report did not cover these instances.

### Pack the following by reducing their size (saves 2.1k Gas)

Here we can pack `isShutDown` and `allowEnter` with address.By doing this we can save one slot which helps to save 2.1k gas 

### Proof of Code
- [PendlePowerFarmDeclarations.sol#L43C4-L47C1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L43C4-L47C1)

```solidity
File: contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol
43:    bool public isShutdown; //@audit pack
44:
45:    // Protects from callbacks (flashloan balancer)
46:    bool public allowEnter;
```
### Optimized code:
```diff
diff --git a/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol b/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol
index 6a5ed05..704e68c 100644
--- a/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol
+++ b/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol
@@ -39,18 +39,18 @@ contract PendlePowerFarmDeclarations is
     ApprovalHelper,
     SendValueHelper
 {
-    // Allows to pause some functions
-    bool public isShutdown;
-
-    // Protects from callbacks (flashloan balancer)
-    bool public allowEnter;
-
     // Ratio between borrow and lend
     uint256 public collateralFactor;
 
     // Adjustable by admin of the contract
     uint256 public minDepositEthAmount;
 
+    // Allows to pause some functions
+    bool public isShutdown;
+
+    // Protects from callbacks (flashloan balancer)
+    bool public allowEnter;
+    
     address public immutable aaveTokenAddresses;
     address public immutable borrowTokenAddresses;
```
### Pack the `globalRoundId` with address `master` by changing their order (saves 2.1k Gas)

In `CustomOracleSetup.sol` we can pack globalRoundId with address master. By doing this we can avoid one slot. which helps to save 2.1k gas.

### Proof of Code
- [CustomOracleSetup.sol#L6C1-L10C33](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L6C1-L10C33)

```solidity
FIle: contracts/DerivativeOracles/CustomOracleSetup.sol
7: address public master;
    uint256 public lastUpdateGlobal;

    uint80 public globalRoundId;
```
### Optimized code:

```diff
diff --git a/contracts/DerivativeOracles/CustomOracleSetup.sol b/contracts/DerivativeOracles/CustomOracleSetup.sol
index 9767fa1..1713ca3 100644
--- a/contracts/DerivativeOracles/CustomOracleSetup.sol
+++ b/contracts/DerivativeOracles/CustomOracleSetup.sol
@@ -5,9 +5,9 @@ pragma solidity =0.8.24;
 contract CustomOracleSetup {
 
     address public master;
+    uint80 public globalRoundId;
     uint256 public lastUpdateGlobal;
 
-    uint80 public globalRoundId;
 
     mapping(uint80 => uint256) public timeStampByRoundId;
```
### Pack the `FEED_DECIMALS` with address `TWAP_DURATION` by changing their order (saves 2.1k Gas)

In `PendleLpOracle.sol` we can pack FEED_DECIMALS with TWAP_DURATION. By doing this we can avoid one slot. which helps to save 2.1k gas.
### Proof of Code
- [PendleLpOracle.sol#L66C4-L73C43](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L66C4-L73C43)
  
```solidity
File: contracts/DerivativeOracles/PendleLpOracle.sol
66:  uint8 internal constant FEED_DECIMALS = 18; //@audit tight pack
@>    uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;

    // Precision factor for computations.
    uint256 internal constant PRECISION_FACTOR_E18 = 1E18;

    // -- Twap duration in seconds --
@>     uint32 public immutable TWAP_DURATION; //@with this
```
### Optimized code:

```diff
diff --git a/contracts/DerivativeOracles/PendleLpOracle.sol b/contracts/DerivativeOracles/PendleLpOracle.sol
index 3819097..38facb7 100644
--- a/contracts/DerivativeOracles/PendleLpOracle.sol
+++ b/contracts/DerivativeOracles/PendleLpOracle.sol
@@ -63,14 +63,13 @@ contract PendleLpOracle {
     IPriceFeed public immutable FEED_ASSET;
     IOraclePendle public immutable ORACLE_PENDLE_PT;
 
-    uint8 internal constant FEED_DECIMALS = 18;
     uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;
 
     // Precision factor for computations.
     uint256 internal constant PRECISION_FACTOR_E18 = 1E18;
-
+    uint8 internal constant FEED_DECIMALS = 18;
     // -- Twap duration in seconds --
```

### Pack the `FEED_DECIMALS` with address `TWAP_DURATION` by changing their order (saves 2.1k Gas)
In `PtOracleDerivative.sol` we can pack FEED_DECIMALS with TWAP_DURATION. By doing this we can avoid one slot. which helps to save 2.1k gas.

### Proof of Code
- [PtOracleDerivative.sol#L65C1-L72C1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L65C1-L72C1)
```solidity
File: contracts/DerivativeOracles/PtOracleDerivative.sol
65:   uint8 internal constant FEED_DECIMALS = 18; //@audit tight pack state

    // Precision factor for computations.
    uint256 internal constant PRECISION_FACTOR_E18 = 1E18;

    // -- Twap Duration --
@>     uint32 public immutable TWAP_DURATION;

```
### Optimized code:

```diff
diff --git a/contracts/DerivativeOracles/PtOracleDerivative.sol b/contracts/DerivativeOracles/PtOracleDerivative.sol
index 5958c7d..3ccb667 100644
--- a/contracts/DerivativeOracles/PtOracleDerivative.sol
+++ b/contracts/DerivativeOracles/PtOracleDerivative.sol
@@ -61,11 +61,10 @@ contract PtOracleDerivative {
     // 10 ** Decimals of the feeds for EthUSD.
     uint256 public immutable POW_ETH_USD_FEED;
 
-    // Default decimals for the Feed.
-    uint8 internal constant FEED_DECIMALS = 18;
-
+    // Default decimals for the Feed.   
     // Precision factor for computations.
     uint256 internal constant PRECISION_FACTOR_E18 = 1E18;
+    uint8 internal constant FEED_DECIMALS = 18;
```
### Pack the `FEED_DECIMALS` with address `DURATION` by changing their order (saves 2.1k Gas)

In `PtOraclePure.sol` we can pack FEED_DECIMALS with DURATION. By doing this we can avoid one slot. which helps to save 2.1k gas.
### Proof of Code
- [PtOraclePure.sol#L49C1-L57C38](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L49C1-L57C38)
```solidity
File: contracts/DerivativeOracles/PtOraclePure.sol
51: uint8 internal constant FEED_DECIMALS = 18; //@audit pack state variable

    // Precision factor for computations.
    uint256 internal constant PRECISION_FACTOR_E18 = 1E18;

    // -- Twap Duration --
@>    uint32 public immutable DURATION;
```
### Optimized code:

```diff
diff --git a/contracts/DerivativeOracles/PtOraclePure.sol b/contracts/DerivativeOracles/PtOraclePure.sol
index dafae12..603e990 100644
--- a/contracts/DerivativeOracles/PtOraclePure.sol
+++ b/contracts/DerivativeOracles/PtOraclePure.sol
@@ -48,11 +48,10 @@ contract PtOraclePure {
     IOraclePendle public immutable ORACLE_PENDLE_PT;
 
     uint256 internal immutable POW_FEED;
-    uint8 internal constant FEED_DECIMALS = 18;
 
     // Precision factor for computations.
     uint256 internal constant PRECISION_FACTOR_E18 = 1E18;
-
+    uint8 internal constant FEED_DECIMALS = 18;
     // -- Twap Duration --
     uint32 public immutable DURATION;
```
## [G-02] Pack structs by reducing variable sizes to save storage slots (saves 2 SLOTS: 4.2k Gas)

Note: The bot report did not cover these instances.

###  struct can be packed by truncating timestamp(Instance Missed by bot)(Saves 2 SLOT: 4.2K Gas)

In `wiseLendingDeclaration:` we can tightly pack TimeStamPoolEntry struct.Since this are timestamps, we can reduce the size to something like uint64 and should still be big enough. We can see the evidence of them being timestamps in how they are assigned

By considering all these conditions we can say that uint64 is more than enough to time. Which can save 2 SLOTS in storage for every Entry.
### Proof of Code
- [WiseLendingDeclaration.sol#L231C1-L236C6](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L231C1-L236C6)
```solidity
File: contracts/WiseLendingDeclaration.sol
232: struct TimestampsPoolEntry {
        uint256 timeStamp; //@audit struct pack
        uint256 timeStampScaling;
        uint256 initialTimeStamp;
    }
```
### Optimized code:

```diff
diff --git a/contracts/WiseLendingDeclaration.sol b/contracts/WiseLendingDeclaration.sol
index 05a0e01..608c798 100644
--- a/contracts/WiseLendingDeclaration.sol
+++ b/contracts/WiseLendingDeclaration.sol
@@ -230,13 +230,13 @@ contract WiseLendingDeclaration is
     }
 
     struct TimestampsPoolEntry {
-        uint256 timeStamp;
-        uint256 timeStampScaling;
-        uint256 initialTimeStamp;
+        uint64 timeStamp; 
+        uint64 timeStampScaling;
+        uint64 initialTimeStamp;
     }
 
     struct CoreLiquidationStruct {
```
## [G-03] Cache calculations in loop to avoid re-calculating on each iteration

### Details
In `FeeManager.sol:setPoolFeeBuli()` : `_newFees[i]` we can cache once and use multiple times. Instead of repeatedly calling _newFees[i] in the loop, you can calculate it once and use the result.
### Proof of Code
- [FeeManager.sol#L135C3-L165C10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L135C3-L165C10)

```solidity
File: /contracts/FeeManager/FeeManager.sol
135:  function setPoolFeeBulk(
        address[] calldata _poolTokens,
        uint256[] calldata _newFees
    )
        external
        onlyMaster
    {
        uint256 i;
        uint256 l = _poolTokens.length;

        while (i < l) {

            _checkValue(
@>                 _newFees[i] //@audit cache _newFees[i]
            );

            WISE_LENDING.setPoolFee(
                _poolTokens[i],
@>                 _newFees[i]
            );

            emit PoolFeeChanged(
                _poolTokens[i],
@>                 _newFees[i],
                block.timestamp
            );

            unchecked {
                ++i;
            }
        }
    }
```
### Optimized code:

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..9dbc7fc 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -143,19 +143,19 @@ contract FeeManager is FeeManagerHelper {
         uint256 l = _poolTokens.length;
 
         while (i < l) {
-
+           uint256 newFees = _newFees[i]; 
             _checkValue(
-                _newFees[i]
+                newFees
             );
 
             WISE_LENDING.setPoolFee(
                 _poolTokens[i],
-                _newFees[i]
+                newFees
             );
 
             emit PoolFeeChanged(
                 _poolTokens[i],
-                _newFees[i],
+                newFees,
                 block.timestamp
             );
```
## [G-04] Minimize the external calls in while loop by storing the result call in a local variable before using in loop.

### The `getPositionBorrowTokenByIndex` call within the loop is an external call

The `getPositionBorrowTokenByIndex` call within the loop is an external call, and making it repeatedly can be expensive. To address this, we can introduce a local variable to store the result of the external call in the loop:
### Proof of Code
- [WiseSecurityHelper.sol#L360C4-L386C6](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L360C4-L386C6)

```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol
360:  function overallETHBorrowBare(
        uint256 _nftId
    )
        public
        view
        returns (uint256 buffer)
    {
        uint256 i;
        uint256 l = WISE_LENDING.getPositionBorrowTokenLength(
            _nftId
        );

        while (i < l) {

            buffer += getETHBorrow(
                _nftId,
@>                 WISE_LENDING.getPositionBorrowTokenByIndex( //@audit cache external call in loop like l 410
                    _nftId,
                    i
                )
            );

            unchecked {
                ++i;
            }
        }
    }
```
### Optimized code:

```diff
diff --git a/contracts/WiseSecurity/WiseSecurityHelper.sol b/contracts/WiseSecurity/WiseSecurityHelper.sol
index a8bb146..5a3dce2 100644
--- a/contracts/WiseSecurity/WiseSecurityHelper.sol
+++ b/contracts/WiseSecurity/WiseSecurityHelper.sol
@@ -364,19 +364,21 @@ abstract contract WiseSecurityHelper is WiseSecurityDeclarations {
         view
         returns (uint256 buffer)
     {
+        address tokenAddress;
         uint256 i;
         uint256 l = WISE_LENDING.getPositionBorrowTokenLength(
             _nftId
         );
 
         while (i < l) {
+             tokenAddress = WISE_LENDING.getPositionBorrowTokenByIndex(
+                _nftId,
+                i
+            ); 
 
             buffer += getETHBorrow(
                 _nftId,
-                WISE_LENDING.getPositionBorrowTokenByIndex(
-                    _nftId,
-                    i
-                )
+                tokenAddress
             );
```
### The `lendingPoolData(tokenAddress).collateralFactor` call within the loop is an external call

The `lendingPoolData(tokenAddress).collateralFactor` call within the loop is an external call, and making it repeatedly can be expensive. To address this, we can introduce a local variable to store the result of the external call in the loop:
### Proof of Code
- [WiseSecurityHelper.sol#L90](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L90)
```solidity
File: contracts/WiseSecurity/WiseSecurityHelper.sol
79:  while (i < l) {

            tokenAddress = WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            );

            _checkPoolCondition(
                tokenAddress
            );

@>             weightedTotal += WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor
                * getFullCollateralETH(
                    _nftId,
                    tokenAddress
                ) / PRECISION_FACTOR_E18;

            unchecked {
                ++i;
            }
        }
```
### Optimized code:

```diff
diff --git a/contracts/WiseSecurity/WiseSecurityHelper.sol b/contracts/WiseSecurity/WiseSecurityHelper.sol
index a8bb146..1e93ad2 100644
--- a/contracts/WiseSecurity/WiseSecurityHelper.sol
+++ b/contracts/WiseSecurity/WiseSecurityHelper.sol
@@ -70,7 +70,7 @@ abstract contract WiseSecurityHelper is WiseSecurityDeclarations {
         returns (uint256 weightedTotal)
     {
         address tokenAddress;
-
+        uint256 lendingPoolData;  
         uint256 i;
         uint256 l = WISE_LENDING.getPositionLendingTokenLength(
             _nftId
@@ -82,12 +82,12 @@ abstract contract WiseSecurityHelper is WiseSecurityDeclarations {
                 _nftId,
                 i
             );
-
+            lendingPoolData = WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor;
             _checkPoolCondition(
                 tokenAddress
             );
 
-            weightedTotal += WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor
+            weightedTotal += lendingPoolData
                 * getFullCollateralETH(
                     _nftId,
                     tokenAddress
```
### Instance 2
The `lendingPoolData(tokenAddress).collateralFactor` call within the loop is an external call, and making it repeatedly can be expensive. To address this, we can introduce a local variable to store the result of the external call in the loop:
### Proof of Code
- [WiseSecurityHelper.sol#L171](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L171)

```diff
@@ -152,7 +155,7 @@ abstract contract WiseSecurityHelper is WiseSecurityDeclarations {
     {
         uint256 i;
         address tokenAddress;
-
+        uint256 lendingPoolData;
         uint256 l = WISE_LENDING.getPositionLendingTokenLength(
             _nftId
         );
@@ -164,11 +167,13 @@ abstract contract WiseSecurityHelper is WiseSecurityDeclarations {
                 i
             );
 
+            lendingPoolData = WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor;
+
             _checkPoolCondition(
                 tokenAddress
             );
 
-            weightedTotal += WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor
+            weightedTotal += lendingPoolData
                 * _getCollateralOfTokenETHUpdated(
                     _nftId,
                     tokenAddress,
```
### Instance 3
The `lendingPoolData(tokenAddress).collateralFactor` call within the loop is an external call, and making it repeatedly can be expensive. To address this, we can introduce a local variable to store the result of the external call in the loop:

- [WiseSecurityHelper.sol#L376](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L376)
- 
### Proof of Code
```diff
diff --git a/contracts/WiseSecurity/WiseSecurityHelper.sol b/contracts/WiseSecurity/WiseSecurityHelper.sol
index a8bb146..c4cd132 100644
--- a/contracts/WiseSecurity/WiseSecurityHelper.sol
+++ b/contracts/WiseSecurity/WiseSecurityHelper.sol
@@ -23,6 +23,7 @@ abstract contract WiseSecurityHelper is WiseSecurityDeclarations {
         uint256 weightedTotal;
         uint256 unweightedAmount;
         address tokenAddress;
+        uint256 lendingPoolData;  
 
         uint256 i;
         uint256 l = WISE_LENDING.getPositionLendingTokenLength(
@@ -36,13 +37,15 @@ abstract contract WiseSecurityHelper is WiseSecurityDeclarations {
                 i
             );
 
+            lendingPoolData = WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor;
+
             amount = getFullCollateralETH(
                 _nftId,
                 tokenAddress
             );
 
             weightedTotal += amount
-                * WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor
+                * lendingPoolData
                 / PRECISION_FACTOR_E18;
 
             unweightedAmount += amount;
```
## [G-05] Move lesser gas costing require checks to the top
Require() or revert() statements that check input arguments or cost lesser gas should be at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### Details
In `MainHelper.sol._checkLiquidatorNft`: This function check liquidator which check against input argument which is cheaper to check. we revert this condition early.In a result we can save gas through avoiding critical check which involve external call and state read.
### Proof of Code
- [MainHelper.sol#L258C5-L278C1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L258C5-L278C1)
```solidity
File: contracts/MainHelper.sol
258: function _checkLiquidatorNft(curveSecurityCheck
        uint256 _nftId,
        uint256 _nftIdLiquidator
    )
        internal
        view
    {
        if (positionLocked[_nftIdLiquidator] == true) {
            revert LiquidatorIsInPowerFarm();
        }

@>         if (_nftIdLiquidator == _nftId) { //@audit check first
            revert InvalidLiquidator();
        }

        if (_nftIdLiquidator >= POSITION_NFT.getNextExpectedId()) {
            revert InvalidLiquidator();
        }
    }
```
### Optimized code:

```diff
diff --git a/contracts/MainHelper.sol b/contracts/MainHelper.sol
index 46854bc..5e7afd9 100644
--- a/contracts/MainHelper.sol
+++ b/contracts/MainHelper.sol
@@ -262,12 +262,12 @@ abstract contract MainHelper is WiseLowLevelHelper {
         internal
         view
     {
-        if (positionLocked[_nftIdLiquidator] == true) {
-            revert LiquidatorIsInPowerFarm();
+         if (_nftIdLiquidator == _nftId) { 
+            revert InvalidLiquidator();
         }
 
-        if (_nftIdLiquidator == _nftId) {
-            revert InvalidLiquidator();
+        if (positionLocked[_nftIdLiquidator] == true) {
+            revert LiquidatorIsInPowerFarm();
         }
 
         if (_nftIdLiquidator >= POSITION_NFT.getNextExpectedId()) {
```
### Instances 2
In `AaveHelper.sol._wrapDepositExactAmount`: This function check if nft id and msg.sender then revert.Before these check there is varuiable assignment operation which cakculate `_wrapAaveReturnValueDeposit`.If we revert this condition early.In a result we can save gas through avoiding critical check which involve external call and state read.
### Proof of Code
- [AaveHelper.sol#L56C5-L72C10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L56C5-L72C10)

```solidity
File: contracts/WrapperHub/AaveHelper.sol
56: function _wrapDepositExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _depositAmount
    )
        internal
        returns (uint256)
    {
        uint256 actualDepositAmount = _wrapAaveReturnValueDeposit(
            _underlyingAsset,
            _depositAmount,
            address(this)
        );

@>         if (POSITION_NFT.isOwner(_nftId, msg.sender) == false) {
            revert InvalidAction();
        } //@audit check first

        uint256 lendingShares = WISE_LENDING.depositExactAmount(
            _nftId,
            aaveTokenAddress[_underlyingAsset],
            actualDepositAmount
        );

        return lendingShares;
    }

```
### Optimized code:

```diff
diff --git a/contracts/WrapperHub/AaveHelper.sol b/contracts/WrapperHub/AaveHelper.sol
index eca55d1..f2a7812 100644
--- a/contracts/WrapperHub/AaveHelper.sol
+++ b/contracts/WrapperHub/AaveHelper.sol
@@ -61,16 +61,16 @@ abstract contract AaveHelper is Declarations {
         internal
         returns (uint256)
     {
+        if (POSITION_NFT.isOwner(_nftId, msg.sender) == false) {
+            revert InvalidAction();
+        }
+
         uint256 actualDepositAmount = _wrapAaveReturnValueDeposit(
             _underlyingAsset,
             _depositAmount,
             address(this)
         );
```
## [G-06] Stack variables should be used in place of state variables rather than re-reading them from storage

### Details
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
### Proof of Code
- [PositionNFTs.sol#L76C4-L79C10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L76C4-L79C10)

```solidity
File: contracts/PositionNFTs.sol
71:  function forwardFeeManagerNFT(
        address _feeManagerContract
    )
        external
        onlyMaster
    {
@>         if (feeManager > ZERO_ADDRESS) { //@audit use memory value 
            revert NotPermitted();
        }

@>         feeManager = _feeManagerContract;

        _transfer(
            address(this),
            _feeManagerContract,
            FEE_MANAGER_NFT
        );
    }

```
### Optimized code:

```diff
diff --git a/contracts/PositionNFTs.sol b/contracts/PositionNFTs.sol
index fb680ec..23b2e46 100644
--- a/contracts/PositionNFTs.sol
+++ b/contracts/PositionNFTs.sol
@@ -74,7 +74,7 @@ contract PositionNFTs is ERC721Enumerable, OwnableMaster {
         external
         onlyMaster
     {
-        if (feeManager > ZERO_ADDRESS) {
+        if (_feeManagerContract > ZERO_ADDRESS) { 
             revert NotPermitted();
         }
```
### Details
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
### Proof of Code
- [WiseCore.sol#L543C4-L558C10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L543C4-L558C10)

```solidity
543:  function _calculateReceiveAmount(
        uint256 _nftId,
        uint256 _nftIdLiquidator,
        address _receiveTokens,
        uint256 _removePercentage
    )
        private
        returns (uint256 receiveAmount)
    {
@>         if (pureCollateralAmount[_nftId][_receiveTokens] > 0) { //@audit use cache 
            receiveAmount = _withdrawPureCollateralLiquidation(
                _nftId,
                _receiveTokens,
                _removePercentage
            );
        }

```
### Optimized code:

```diff
diff --git a/contracts/WiseCore.sol b/contracts/WiseCore.sol
index e50ef82..9d1a983 100644
--- a/contracts/WiseCore.sol
+++ b/contracts/WiseCore.sol
@@ -549,7 +549,7 @@ abstract contract WiseCore is MainHelper, TransferHelper {
         private
         returns (uint256 receiveAmount)
     {
-        if (pureCollateralAmount[_nftId][_receiveTokens] > 0) {
+        if (pureCollateral > 0) { //@audit use cache 
             receiveAmount = _withdrawPureCollateralLiquidation(
                 _nftId,
                 _receiveTokens,
```
## [G-07] Directly modifies the value in the storage location without reading the current value first

### Details
using pre-increment (++i) instead of the addition assignment (i += 1) can sometimes result in more efficient code. By doing this we can avoid state variable read as a result we can save gas.
### Proof of Code
- [PositionNFTs.sol#L111C4-L128C6](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L111C4-L128C6)

```solidity
File: contracts/PositionNFTs.sol
111: function _reservePositionForUser(
        address _user
    )
        internal
        returns (uint256)
    {
        if (reserved[_user] > 0) {
            return reserved[_user];
        }

        uint256 reservedId = getNextExpectedId();
        reserved[_user] = reservedId;

        totalReserved =
@>         totalReserved + 1; //@audit use preincrement

        return reservedId;
    }
```
### Optimized code:

```diff
diff --git a/contracts/PositionNFTs.sol b/contracts/PositionNFTs.sol
index fb680ec..946a418 100644
--- a/contracts/PositionNFTs.sol
+++ b/contracts/PositionNFTs.sol
@@ -122,7 +122,7 @@ contract PositionNFTs is ERC721Enumerable, OwnableMaster {
         reserved[_user] = reservedId;
 
-        totalReserved =
-        totalReserved + 1;
+        ++totalReserved;
 
         return reservedId;
     }
```
## [G-08] State variables should be cached in stack variables rather than re-reading them from storage
Note: Please note these instances were not included in the bots reports Proof of Code:

### Details
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
### Proof of Code
- [PendlePowerFarmToken.sol#L386C4-L412C1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L386C4-L412C1)

```solidity
File: /contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
386:  function previewDistribution()
        public
        view
        returns (uint256)
    {
        if (totalLpAssetsToDistribute == 0) {
            return 0;
        }

@>        if (block.timestamp == lastInteraction) { //@audit cache lastINteraction
            return 0;
        }

        if (totalLpAssetsToDistribute < ONE_WEEK) {
            return totalLpAssetsToDistribute;
        }

        uint256 currentRate = totalLpAssetsToDistribute
            / ONE_WEEK;

@>        uint256 additonalAssets = currentRate
            * (block.timestamp - lastInteraction);
```
### Optimized code:

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..04c15de 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -391,8 +391,8 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
         if (totalLpAssetsToDistribute == 0) {
             return 0;
         }
-
-        if (block.timestamp == lastInteraction) {
+        _lastInteraction = lastInteraction;
+        if (block.timestamp == _lastInteraction) {
             return 0;
         }
 
@@ -404,7 +404,7 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
             / ONE_WEEK;
 
         uint256 additonalAssets = currentRate
-            * (block.timestamp - lastInteraction);
+            * (block.timestamp - _lastInteraction);
 
         if (additonalAssets > totalLpAssetsToDistribute) {
             return totalLpAssetsToDistribute;
```

### Instance 2
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
### Proof of Code
- [PendlePowerFarmToken.sol#L529C3-L554C11](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L529C3-L554C11)
  
```solidity
File: /contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
529:    function depositExactAmount(
        uint256 _underlyingLpAssetAmount
    )
        uint256 shares = previewMintShares(
            _underlyingLpAssetAmount,
@>            underlyingLpAssetsCurrent //@audit cache state variable
        );
....

        _mint(
            PENDLE_POWER_FARM_CONTROLLER,
            feeShares
        );

@>        underlyingLpAssetsCurrent += _underlyingLpAssetAmount;
```
### Optimized code:

```diff
@@ -539,10 +539,10 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
         if (_underlyingLpAssetAmount == 0) {
             revert ZeroAmount();
         }
-
+        _underlyingLpAssetsCurrent = underlyingLpAssetsCurrent;
         uint256 shares = previewMintShares(
             _underlyingLpAssetAmount,
-            underlyingLpAssetsCurrent
+            _underlyingLpAssetsCurrent 
         );
 
         if (shares == 0) {
@@ -574,7 +574,7 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
             feeShares
         );
 
-        underlyingLpAssetsCurrent += _underlyingLpAssetAmount;
+        underlyingLpAssetsCurrent =_underlyingLpAssetsCurrent + _underlyingLpAssetAmount;
 
         _safeTransferFrom(
             UNDERLYING_PENDLE_MARKET,
```
## [G-09] Do not cache global variable in local variable

### Details
Use global value instead of caching in local variable.Caching global variable is more likely consume more gas.
### Proof of Code
- [FeeManager.sol#L695](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L695)

```solidity 
File: contracts/FeeManager/FeeManager.sol
695:        address caller = msg.sender; //@audit do not cache
```
### Optimized code:

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..e09ab30 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -692,13 +705,13 @@ contract FeeManager is FeeManagerHelper {
     )
         external
     {
-        address caller = msg.sender;
+
 
         if (totalBadDebtETH > 0) {
             revert ExistingBadDebt();
         }
 
-        if (allowedTokens[caller][_feeToken] == false) {
+        if (allowedTokens[msg.sender][_feeToken] == false) {
             revert NotAllowed();
         }
         _safeTransfer(
             _feeToken,
-            caller,
+            msg.sender,
             _amount
         );
 
         emit ClaimedFeesBeneficial(
-            caller,
+            msg.sender,
             _feeToken,
             _amount,
             block.timestamp
```
## [G-10] Refactor `_preparationToken` to avoid redundant storage read

### Details
when we use address[] memory tokens = _userTokenData[_nftId];, we are creating a new memory array and copying the values from the storage array referenced by _userTokenData[_nftId]. This operation incurs some gas cost, especially if the array is large.

By returning address[] storage, you are returning a reference to the storage array _userTokenData[_nftId]. This can save gas because you avoid the overhead of copying the array to memory.

### Proof of Code
- [MainHelper.sol#L391C4-L410C1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L391C4-L410C1)
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
@>         address[] memory tokens = _userTokenData[ //@audit redundant check
            _nftId
        ];

        _prepareTokens(
            _poolToken,
            tokens 
        );

        return tokens;
    }

```
### Optimized code:

```diff
function _preparationTokens(
    mapping(uint256 => address[]) storage _userTokenData,
    uint256 _nftId,
    address _poolToken
)
    internal
-    returns (address[] memory)
+    returns (address[] storage)
{
    // Consider removing this redundant storage read
-    address[] memory tokens = _userTokenData[_nftId];

    _prepareTokens(_poolToken, _userTokenData[_nftId]);

    return _userTokenData[_nftId];
}
```
## [G-11] Storing similar computations to reduce the number of operations.

### Details
Combine similar computations to reduce the number of operations.The calculation involving currentTime - timestampsPoolData[_poolToken].timeStamp is repeated. Consider storing this value in a variable.

### Proof of Code
- [MainHelper.sol#L507C1-L510C42](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L507C1-L510C42)

```solidity
File: contracts/MainHelper.sol
508: function _updatePseudoTotalAmounts( 
        address _poolToken
    )
        private
    {
        uint256 currentTime = block.timestamp;

@>         uint256 bareIncrease = borrowPoolData[_poolToken].borrowRate
            * (currentTime - timestampsPoolData[_poolToken].timeStamp)
            * borrowPoolData[_poolToken].pseudoTotalBorrowAmount
            + bufferIncrease[_poolToken];

        if (bareIncrease < PRECISION_FACTOR_YEAR) {
            bufferIncrease[_poolToken] = bareIncrease;

            _setTimeStamp(
                _poolToken,
                currentTime
            );

            return;
        }

```
### Optimized code:

```diff
+uint256 timeDifference = currentTime - timestampsPoolData[_poolToken].timeStamp;

+uint256 bareIncrease = borrowPoolData[_poolToken].borrowRate * timeDifference;
```
## [G-12] Do not cache `uniTwapPoolInfo[_tokenAddress]` in memory if used only once
### Details
In `OracleHepler.sol` cache uniTwapPoolInfoStruct in memory which is not needed.If variable used once in fucntion then we can avoid caching variable in memory.

### Proof of Code
- [OracleHelper.sol#L45C8-L47C10](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L45C8-L47C10)

```solidity
File: contracts/WiseOracleHub/OracleHelper.sol
46:   if (_checkFunctionExistence(address(tokenAggregator)) == false) {
            revert FunctionDoesntExist();
        }

@>        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[ //@audit do not cache used only once
            _tokenAddress
        ];

```
### Optimized code:

```diff
diff --git a/contracts/WiseOracleHub/OracleHelper.sol b/contracts/WiseOracleHub/OracleHelper.sol
index 687b6f1..d3fb824 100644
--- a/contracts/WiseOracleHub/OracleHelper.sol
+++ b/contracts/WiseOracleHub/OracleHelper.sol
@@ -46,11 +46,7 @@ abstract contract OracleHelper is Declarations {
             revert FunctionDoesntExist();
         }
 
-        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[
-            _tokenAddress
-        ];
-
-        if (uniTwapPoolInfoStruct.oracle > ZERO_ADDRESS) {
+        if (uniTwapPoolInfo[_tokenAddress].oracle > ZERO_ADDRESS) {
             revert AggregatorNotNecessary();
         }
```
## [G-13] Check amount for zero before send ETH (Missed by bot)
Note: Bot only covers check 0 value transfers in tranfser and transferFrom but not covered when _send ETH 0 amounts.

### Details
In the withdrawExactAmountETH function, when calling _sendValue, there is no explicit check to ensure that _amount is greater than zero before attempting to send ETH. Implementing a zero amount check avoids these unnecessary function calls, saving gas and improving efficiency.
### Proof of Code
- [WiseLending.sol#L1004C2-L1008C1](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1004C2-L1008C1)
```solidity
File: contracts/WiseLending.sol
1004:    _sendValue(
            msg.sender,
            _amount //@audit validate zero value   if (refundAmount > 0) {}
        );
```
```solidity
File:
663: _sendValue(
            msg.sender,
            _amount //@audit no check for zero address   if (refundAmount > 0) {}
        );
```
### Optimized code:
This prevents unnecessary gas consumption if _amount is zero and ensures that the transfer is only attempted when there is a non-zero withdrawal amount.
```diff
+ if (_amount > 0) {
    _sendValue(msg.sender, _amount);
+  }
```