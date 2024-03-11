# WISE LENDING GAS OPTIMIZATIONS


## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."






## [G-01] Refactor `PendlePowerFarmController.increaseReservedForCompound()` function to avoid unnecessary coping from storage to memory and vice-versa
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L497-#L524

The `PendlePowerFarmController.increaseReservedForCompound()` function can be refactored to be better gas efficient by avoiding unnecessary coping of values from storage to memory updating the values then coping back to storage. Having to copy from the storage variable `pendleChildCompoundInfo[_pendleMarket]` into a memory variable `childInfo` would mean that every storage slot of `pendleChildCompoundInfo[_pendleMarket]` would be read (even those not needed in the function) which would cost 2100 gas units for every slot read, then having to update the memory variable in the while loop before coping the memory variable into the storage is absolutely gas inefficient. We can rectify this by making the `childInfo` variable a storage variable, doing this would avoid having copy values from storage to memory since the it is passed by reference also there would be absolutely no need to copy from memory back to storage. The diff below shows how the code should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

497:    function increaseReservedForCompound(
498:        address _pendleMarket,
499:        uint256[] calldata _amounts
500:    )
501:        external
502:        onlyChildContract(_pendleMarket)
503:    {
504:        CompoundStruct memory childInfo = pendleChildCompoundInfo[
505:            _pendleMarket
506:        ];
507:
508:        uint256 i;
509:        uint256 length = childInfo.rewardTokens.length;
510:
511:        while (i < length) {
512:            childInfo.reservedForCompound[i] += _amounts[i];
513:            unchecked {
514:                ++i;
515:            }
516:        }
517:
518:        pendleChildCompoundInfo[_pendleMarket] = childInfo;
519:
520:        emit IncreaseReservedForCompound(
521:            _pendleMarket,
522:            _amounts
523:        );
524:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.solindex 15cb863..842b94e 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
@@ -501,7 +501,7 @@ contract PendlePowerFarmController is PendlePowerFarmControllerHelper {
         external
         onlyChildContract(_pendleMarket)
     {
-        CompoundStruct memory childInfo = pendleChildCompoundInfo[
+        CompoundStruct storage childInfo = pendleChildCompoundInfo[
             _pendleMarket
         ];

@@ -515,8 +515,6 @@ contract PendlePowerFarmController is PendlePowerFarmControllerHelper {
             }
         }

-        pendleChildCompoundInfo[_pendleMarket] = childInfo;
-
         emit IncreaseReservedForCompound(
             _pendleMarket,
             _amounts
```
```
Estimated gas saved: 13672 gas units
```




## [G-02] Avoid Reading and writing to state if the value is zero

### 2 Instances

1. #### Refactor `FeeManager.increaseIncentiveA()` to avoid reading or writing to state if `_value` is 0.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L220

In the `FeeManager.increaseIncentiveA()` function as shown below checks should be implemented to avoid  reading and writing to state if the `_value` argument is zero this is because in scenarios where `_value` is 0 the statement `incentiveUSD[incentiveOwnerA] += _value` would not in any way change the value of `incentiveUSD[incentiveOwnerA]` since it is being incremented by 0.  This then means that in scenarios where `_value_` is 0 the statement `incentiveUSD[incentiveOwnerA] += _value` is re-assigning the same value to state i.e there is no state change.

```solidity
file: contracts/FeeManager/FeeManager.sol

214:    function increaseIncentiveA(
215:        uint256 _value
216:    )
217:        external
218:        onlyIncentiveMaster
219:    {
220:        incentiveUSD[incentiveOwnerA] += _value;   // @audit implement zero checks
221:
222:        emit IncentiveIncreasedA(
223:            _value,
224:            block.timestamp
225:        );
226:    }
```

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..4515083 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -24,6 +24,7 @@ import "./FeeManagerHelper.sol";

 contract FeeManager is FeeManagerHelper {

+    error ZeroValue();
     constructor(
         address _master,
         address _aaveAddress,
@@ -217,6 +218,9 @@ contract FeeManager is FeeManagerHelper {
         external
         onlyIncentiveMaster
     {
+        if(_value == 0) {
+            revert ZeroValue();
+        }
         incentiveUSD[incentiveOwnerA] += _value;
```
```
Estimated gas saved: 50000 gas units.
```

2. #### Refactor `FeeManager.increaseIncentiveB()` to avoid reading or writing to state if `_value` is 0.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L238

In the `FeeManager.increaseIncentiveB()` function as shown below checks should be implemented to avoid  reading and writing to state if the `_value` argument is zero this is because in scenarios where `_value` is 0 the statement `incentiveUSD[incentiveOwnerB] += _value` would not in any way change the value of `incentiveUSD[incentiveOwnerB]` since it is being incremented by 0.  This then means that in scenarios where `_value` is 0 the statement `incentiveUSD[incentiveOwnerB] += _value` is re-assigning the same value to state i.e there is no state change.

```solidity
file: contracts/FeeManager/FeeManager.sol

232:    function increaseIncentiveB(
233:        uint256 _value
234:    )
235:        external
236:        onlyIncentiveMaster
237:    {
238:        incentiveUSD[incentiveOwnerB] += _value;   // @audit implement zero checks
239:
240:        emit IncentiveIncreasedB(
241:            _value,
242:            block.timestamp
243:        );
244:    }
```

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..79f4dc6 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -24,6 +24,7 @@ import "./FeeManagerHelper.sol";

 contract FeeManager is FeeManagerHelper {

+    error ZeroValue();
     constructor(
         address _master,
         address _aaveAddress,
@@ -234,7 +235,10 @@ contract FeeManager is FeeManagerHelper {
     )
         external
         onlyIncentiveMaster
-    {
+    {
+        if(_value == 0) {
+            revert ZeroValue();
+        }
         incentiveUSD[incentiveOwnerB] += _value;

         emit IncentiveIncreasedB(
```

```
Estimated gas saved: 5000 gas units
```





## [G-03] Refactor the following function to reduce number of `SLOAD`

### Instances
1. #### Refactor the `WiseLending._healthStateCheck()` function such that the number of storage reads is reduced
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L77-#L90

We can make the `WiseLending._healthStateCheck()` function better gas efficient if we reduce the number of state reads in the function . We can do this by caching state variables that are read more than once into stack variables. The `powerFarmCheck` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the statck variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored.

```solidity
file: contracts/WiseLending.sol

77:    function _healthStateCheck(
78:        uint256 _nftId
79:    )
80:        private
81:    {
82:        _checkHealthState(
83:            _nftId,
84:            powerFarmCheck   //@audit 1st powerFarmCheck SLOAD
85:        );
86:
87:        if (powerFarmCheck == true) {       //@audit 2nd powerFarmCheck SLOAD
88:            powerFarmCheck = false;
89:        }
90:    }
```

```diff
diff --git a/contracts/WiseLending.sol b/contracts/WiseLending.sol
index 045678d..a35c653 100644
--- a/contracts/WiseLending.sol
+++ b/contracts/WiseLending.sol
@@ -79,12 +79,13 @@ contract WiseLending is PoolManager {
     )
         private
     {
+        bool _powerFarmCheck = powerFarmCheck;
         _checkHealthState(
             _nftId,
-            powerFarmCheck
+            _powerFarmCheck
         );

-        if (powerFarmCheck == true) {
+        if (_powerFarmCheck == true) {
             powerFarmCheck = false;
         }
     }
```
```
Estimated gas saved: 97 gas units
```

2. #### Refactor the `PendlePowerFarmToken.addCompoundRewards()` function such that the number of storage reads is reduced
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L502-#L524

We can make the `PendlePowerFarmToken.addCompoundRewards()` function better gas efficient if we reduce the number of state reads in the function . We can do this by caching state variables that are read more than once into stack variables. The `PENDLE_POWER_FARM_CONTROLLER` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

502:    function addCompoundRewards(
503:        uint256 _amount
504:    )
505:        external
506:        syncSupply
507:    {
508:        if (_amount == 0) {
509:            revert ZeroAmount();
510:        }
511:
512:        totalLpAssetsToDistribute += _amount;
513:
514:        if (msg.sender == PENDLE_POWER_FARM_CONTROLLER) {   // @audit 1st PENDLE_POWER_FARM_CONTROLLER SLOAD
515:            return;
516:        }
517:
518:        _safeTransferFrom(
519:            UNDERLYING_PENDLE_MARKET,
520:            msg.sender,
521:            PENDLE_POWER_FARM_CONTROLLER,      // @audit 1st PENDLE_POWER_FARM_CONTROLLER SLOAD
522:            _amount
523:        );
524:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..2f13a79 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -510,15 +510,15 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
         }

         totalLpAssetsToDistribute += _amount;
-
-        if (msg.sender == PENDLE_POWER_FARM_CONTROLLER) {
+        address _PENDLE_POWER_FARM_CONTROLLER = PENDLE_POWER_FARM_CONTROLLER;
+        if (msg.sender == _PENDLE_POWER_FARM_CONTROLLER) {
             return;
         }

         _safeTransferFrom(
             UNDERLYING_PENDLE_MARKET,
             msg.sender,
-            PENDLE_POWER_FARM_CONTROLLER,
+            _PENDLE_POWER_FARM_CONTROLLER,
             _amount
         );
     }
```
```
Estimated gas saved: 97 gas units
```

3. #### Refactor the `PendlePowerFarmToken.withdrawExactShares()` function such that the number of storage reads is reduced
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L608-#L645

We can make the `PendlePowerFarmToken.withdrawExactShares()` function better gas efficient if we reduce the number of state reads in the function . We can do this by caching state variables that are read more than once into stack variables. The `underlyingLpAssetsCurrent` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

608:    function withdrawExactShares(
609:        uint256 _shares
610:    )
611:        external
612:        syncSupply
613:        returns (uint256)
614:    {
615:        if (_shares == 0) {
616:            revert ZeroAmount();
617:        }
618:
619:        if (_shares > balanceOf(msg.sender)) {
620:            revert InsufficientShares();
621:        }
622:
623:        uint256 tokenAmount = previewAmountWithdrawShares(
624:            _shares,
625:            underlyingLpAssetsCurrent   // @audit underlyingLpAssetsCurrent 1st SLOAD
626:        );
627:
628:        underlyingLpAssetsCurrent -= tokenAmount;   // @audit underlyingLpAssetsCurrent 2nd SLOAD
629:
630:        _burn(
631:            msg.sender,
632:            _shares
633:        );
634:
635:        if (msg.sender == PENDLE_POWER_FARM_CONTROLLER) {
636:            return tokenAmount;
637:        }
638:
639:        _withdrawLp(
640:            msg.sender,
641:            tokenAmount
642:        );
643:
644:        return tokenAmount;
645:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..436d8be 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -620,12 +620,14 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
             revert InsufficientShares();
         }

+        uint256 _underlyingLpAssetsCurrent = underlyingLpAssetsCurrent;
+
         uint256 tokenAmount = previewAmountWithdrawShares(
             _shares,
-            underlyingLpAssetsCurrent
+            _underlyingLpAssetsCurrent
         );

-        underlyingLpAssetsCurrent -= tokenAmount;
+        underlyingLpAssetsCurrent = _underlyingLpAssetsCurrent - tokenAmount;

         _burn(
             msg.sender,
```
```
Estimated gas saved: 97 gas units
```


4. #### Refactor the `PendlePowerFarmToken.withdrawExactAmount()` function such that the number of storage reads is reduced
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L647-#L680

We can make the `PendlePowerFarmToken.withdrawExactAmount()` function better gas efficient if we reduce the number of state reads in the function . We can do this by caching state variables that are read more than once into stack variables. The `underlyingLpAssetsCurrent` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

647:    function withdrawExactAmount(
648:        uint256 _underlyingLpAssetAmount
649:    )
650:        external
651:        syncSupply
652:        returns (uint256)
653:    {
654:        if (_underlyingLpAssetAmount == 0) {
655:            revert ZeroAmount();
656:        }
657:
658:        uint256 shares = previewBurnShares(
659:            _underlyingLpAssetAmount,
660:            underlyingLpAssetsCurrent   // @audit underlyingLpAssetsCurrent 1st SLOAD
661:        );
662:
663:        if (shares > balanceOf(msg.sender)) {
664:            revert NotEnoughShares();
665:        }
666:
667:        _burn(
668:            msg.sender,
669:            shares
670:        );
671:
672:        underlyingLpAssetsCurrent -= _underlyingLpAssetAmount; // @audit underlyingLpAssetsCurrent 2nd SLOAD
673:
674:        _withdrawLp(
675:            msg.sender,
676:            _underlyingLpAssetAmount
677:        );
678:
679:        return shares;
680:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..5912c50 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -655,9 +655,10 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
             revert ZeroAmount();
         }

+        uint256 _underlyingLpAssetsCurrent = underlyingLpAssetsCurrent;
         uint256 shares = previewBurnShares(
             _underlyingLpAssetAmount,
-            underlyingLpAssetsCurrent
+            _underlyingLpAssetsCurrent
         );

         if (shares > balanceOf(msg.sender)) {
@@ -669,7 +670,7 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
             shares
         );

-        underlyingLpAssetsCurrent -= _underlyingLpAssetAmount;
+        underlyingLpAssetsCurrent = _underlyingLpAssetsCurrent - _underlyingLpAssetAmount;

         _withdrawLp(
             msg.sender,
```
```
Estimated gas saved: 97 gas units
```

5. #### Refactor the `FeeManager.claimOwnershipIncentiveMaster()` function such that the number of storage reads is reduced
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L194-#L208

We can make the `FeeManager.claimOwnershipIncentiveMaster()` function better gas efficient if we reduce the number of state reads in the function . We can do this by caching state variables that are read more than once into stack variables. The `proposedIncentiveMaster` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable also the `proposedIncentiveMaster` was assigned to `incentiveMaster` which is also used in the event `ClaimedOwnershipIncentiveMaster` rather than using `incentiveMaster` in the event we should use the cached stack variable  . Implementing this would avoid 2 `SLOAD`(warmaccess) `200` gas units and replace it with cheaper stack reads. The diff below shows how the function should be refactored:

```solidity
file: contracts/FeeManager/FeeManager.sol

194:    function claimOwnershipIncentiveMaster()
195:        external
196:    {
197:        if (msg.sender != proposedIncentiveMaster) { // @audit proposedIncentiveMaster 1st SLOAD
198:            revert NotAllowed();
199:        }
200:
201:        incentiveMaster = proposedIncentiveMaster; // @audit proposedIncentiveMaster 2nd SLOAD
202:        proposedIncentiveMaster = ZERO_ADDRESS;
203:
204:        emit ClaimedOwnershipIncentiveMaster(
205:            incentiveMaster,
206:            block.timestamp
207:        );
208:    }
```

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..1ef0ae7 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -194,15 +194,16 @@ contract FeeManager is FeeManagerHelper {
     function claimOwnershipIncentiveMaster()
         external
     {
-        if (msg.sender != proposedIncentiveMaster) {
+        address _proposedIncentiveMaster = proposedIncentiveMaster;
+        if (msg.sender != _proposedIncentiveMaster) {
             revert NotAllowed();
         }

-        incentiveMaster = proposedIncentiveMaster;
+        incentiveMaster = _proposedIncentiveMaster;
         proposedIncentiveMaster = ZERO_ADDRESS;

         emit ClaimedOwnershipIncentiveMaster(
-            incentiveMaster,
+            _proposedIncentiveMaster,
             block.timestamp
         );
     }
```
```
Estimated gas saved: 194 gas units
```

6. #### Refactor the `WiseOracleHub.getTokensFromUSD()` function such that the number of storage reads is reduced
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L171-#L192

We can make the `WiseOracleHub.getTokensFromUSD()` function better gas efficient if we reduce the number of state reads in the function . We can do this by caching state variables that are read more than once into stack variables. The `_decimalsETH` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored:

```solidity
file: contracts/WiseOracleHub/WiseOracleHub.sol

171:    function getTokensFromUSD(
172:        address _tokenAddress,
173:        uint256 _usdValue
174:    )
175:        external
176:        view
177:        returns (uint256)
178:    {
179:        uint8 tokenDecimals = _tokenDecimals[
180:            _tokenAddress
181:        ];
182:
183:        return _decimalsETH < tokenDecimals  // @audit _decimalsETH 1st SLOAD
184:            ? _usdValue
185:                * 10 ** (tokenDecimals - _decimalsETH)    // @audit _decimalsETH 2nd SLOAD
186:                * 10 ** decimals(_tokenAddress)
187:                / latestResolver(_tokenAddress)
188:            : _usdValue
189:                * 10 ** decimals(_tokenAddress)
190:                / latestResolver(_tokenAddress)
191:                / 10 ** (_decimalsETH - tokenDecimals);  // @audit _decimalsETH 2nd SLOAD
192:    }
```

```diff
diff --git a/contracts/WiseOracleHub/WiseOracleHub.sol b/contracts/WiseOracleHub/WiseOracleHub.sol
index 12fe0c8..8d08417 100644
--- a/contracts/WiseOracleHub/WiseOracleHub.sol
+++ b/contracts/WiseOracleHub/WiseOracleHub.sol
@@ -179,16 +179,16 @@ contract WiseOracleHub is OracleHelper {
         uint8 tokenDecimals = _tokenDecimals[
             _tokenAddress
         ];
-
-        return _decimalsETH < tokenDecimals
+        uint8 decimalsEth = _decimalsETH;
+        return decimalsEth < tokenDecimals
             ? _usdValue
-                * 10 ** (tokenDecimals - _decimalsETH)
+                * 10 ** (tokenDecimals - decimalsEth)
                 * 10 ** decimals(_tokenAddress)
                 / latestResolver(_tokenAddress)
             : _usdValue
                 * 10 ** decimals(_tokenAddress)
                 / latestResolver(_tokenAddress)
-                / 10 ** (_decimalsETH - tokenDecimals);
+                / 10 ** (decimalsEth - tokenDecimals);
     }

     /**
```
```
Estimated gas saved: 97 gas units
```

7. #### Refactor the `WiseOracleHub.getTokensFromETH()` function such that the number of storage reads is reduced
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L327-#L352

We can make the `WiseOracleHub.getTokensFromETH()` function better gas efficient if we reduce the number of state reads in the function . We can do this by caching state variables that are read more than once into stack variables. The `_decimalsETH` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored:

```solidity
file: contracts/WiseOracleHub/WiseOracleHub.sol

327:    function getTokensFromETH(
328:        address _tokenAddress,
329:        uint256 _ethAmount
330:    )
331:        public
332:        view
333:        returns (uint256)
334:    {
335:        if (_tokenAddress == WETH_ADDRESS) {
336:            return _ethAmount;
337:        }
338:
339:        uint8 tokenDecimals = _tokenDecimals[
340:            _tokenAddress
341:        ];
342:
343:        return _decimalsETH < tokenDecimals       // @audit _decimalsETH 1st SLOAD
344:            ? _ethAmount
345:                * 10 ** (tokenDecimals - _decimalsETH)        // @audit _decimalsETH 2nd SLOAD
346:                * 10 ** decimals(_tokenAddress)
347:                / latestResolver(_tokenAddress)
348:            : _ethAmount
349:                * 10 ** decimals(_tokenAddress)
350:                / latestResolver(_tokenAddress)
351:                / 10 ** (_decimalsETH - tokenDecimals);   // @audit _decimalsETH 3rd SLOAD
352:    }
```

```diff
diff --git a/contracts/WiseOracleHub/WiseOracleHub.sol b/contracts/WiseOracleHub/WiseOracleHub.sol
index 12fe0c8..78004fe 100644
--- a/contracts/WiseOracleHub/WiseOracleHub.sol
+++ b/contracts/WiseOracleHub/WiseOracleHub.sol
@@ -339,16 +339,16 @@ contract WiseOracleHub is OracleHelper {
         uint8 tokenDecimals = _tokenDecimals[
             _tokenAddress
         ];
-
-        return _decimalsETH < tokenDecimals
+        uint8 decimalsEth = _decimalsETH;
+        return decimalsEth < tokenDecimals
             ? _ethAmount
-                * 10 ** (tokenDecimals - _decimalsETH)
+                * 10 ** (tokenDecimals - decimalsEth)
                 * 10 ** decimals(_tokenAddress)
                 / latestResolver(_tokenAddress)
             : _ethAmount
                 * 10 ** decimals(_tokenAddress)
                 / latestResolver(_tokenAddress)
-                / 10 ** (_decimalsETH - tokenDecimals);
+                / 10 ** (decimalsEth - tokenDecimals);
     }

     /**
```
```
Estimated gas saved: 97 gas units
```




## [G-04] Unnecessary copy of storage struct to memory in the `OracleHelper` contract
The OracleHelper contract has multiple instances where a complete struct is being copied to memory to end up using only one of its attributes. The contract should be refactored such that only the required attribute is read from storage and cached into stack variable.

### Instances
1. #### Refactor `OracleHelper._addAggregator()` function to avoid coping the storage struct `uniTwapPoolInfo[  _tokenAddress]` into memory struct variable `uniTwapPoolInfoStruct`.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L49-#L51

We could save up to 2.1k gas units if we read the `oracle` attribute of the storage struct ``uniTwapPoolInfo[  _tokenAddress]` directly rather than having to copy the storage struct in to a memory struct variable before accessing the member.

```solidity
file: contracts/WiseOracleHub/OracleHelper.sol

32:    function _addAggregator(
33:        address _tokenAddress
34:    )
35:        internal
36:    {
37:        IAggregator tokenAggregator = IAggregator(
38:            priceFeed[_tokenAddress].aggregator()
39:        );
40:
41:        if (tokenAggregatorFromTokenAddress[_tokenAddress] > ZERO_AGGREGATOR) {
42:            revert AggregatorAlreadySet();
43:        }
44:
45:        if (_checkFunctionExistence(address(tokenAggregator)) == false) {
46:            revert FunctionDoesntExist();
47:        }
48:
49:        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[  
50:            _tokenAddress
51:        ];
52:
53:        if (uniTwapPoolInfoStruct.oracle > ZERO_ADDRESS) {
54:            revert AggregatorNotNecessary();
55:        }
56:
57:        tokenAggregatorFromTokenAddress[_tokenAddress] = tokenAggregator;
58:    }
```
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
```
Estimated gas saved: 2100 gas units
```

2. #### Refactor `OracleHelper._validateAnswer()` function to avoid coping the storage struct `uniTwapPoolInfo[  _tokenAddress]` into memory struct variable `uniTwapPoolInfoStruct`.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L138-#L140

We could save up to 2.1k gas units if we read the `oracle` attribute of the storage struct ``uniTwapPoolInfo[  _tokenAddress]` directly rather than having to copy the storage struct in to a memory struct variable before accessing the member.

```solidity
file: contracts/WiseOracleHub/OracleHelper.sol

131:    function _validateAnswer(
132:        address _tokenAddress
133:    )
134:        internal
135:        view
136:        returns (uint256)
137:    {
138:        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[
139:            _tokenAddress
140:        ];
141:
142:        uint256 fetchTwapValue;
143:
144:        if (uniTwapPoolInfoStruct.oracle > ZERO_ADDRESS) {
145:            fetchTwapValue = latestResolverTwap(
146:                _tokenAddress
147:            );
148:        }
.
.
.
174:    }
```
```diff
diff --git a/contracts/WiseOracleHub/OracleHelper.sol b/contracts/WiseOracleHub/OracleHelper.sol
index 687b6f1..9200896 100644
--- a/contracts/WiseOracleHub/OracleHelper.sol
+++ b/contracts/WiseOracleHub/OracleHelper.sol
@@ -135,13 +135,10 @@ abstract contract OracleHelper is Declarations {
         view
         returns (uint256)
     {
-        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[
-            _tokenAddress
-        ];

         uint256 fetchTwapValue;

-        if (uniTwapPoolInfoStruct.oracle > ZERO_ADDRESS) {
+        if (uniTwapPoolInfo[_tokenAddress].oracle > ZERO_ADDRESS) {
             fetchTwapValue = latestResolverTwap(
                 _tokenAddress
             );
```
```
Estimated gas saved: 2100 gas units
```




## [G-05] Redundant state variable getters
Getters for public state variables are automatically generated by the solidity compiler so there is no need to code them manually as this increases deployment cost.
#### Please note the this instance was not included in the bots report.

### Instances
1. #### Make `positionLendTokenData` mapping variable `private` or `internal` since a getter function was defined for it.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L270
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L144-#L153

The solidity compiler would automatically create a getter function for the `positionLendTokenData` mapping since it is declared as a `public` variable but a getter function `getPositionLendingTokenByIndex()` was also declared in the `WiseLowLevelHelper` contract for the same variable thereby making it two getter functions for the same variable in the contract. We could rectify this issue by making the `positionLendTokenData` variable private or internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

```solidity
file: contracts/WiseLendingDeclaration.sol

270:    mapping(uint256 => address[]) public positionLendTokenData;


file: contracts/WiseLowLevelHelper.sol

144:    function getPositionLendingTokenByIndex(
145:        uint256 _nftId,
146:        uint256 _index
147:    )
148:        public
149:        view
150:        returns (address)
151:    {
152:        return positionLendTokenData[_nftId][_index];
153:    }
```

```diff
diff --git a/contracts/WiseLendingDeclaration.sol b/contracts/WiseLendingDeclaration.sol
index 05a0e01..d1ba624 100644
--- a/contracts/WiseLendingDeclaration.sol
+++ b/contracts/WiseLendingDeclaration.sol
@@ -267,7 +267,7 @@ contract WiseLendingDeclaration is
     mapping(address => uint256) internal bufferIncrease;
     mapping(address => uint256) public maxDepositValueToken;

-    mapping(uint256 => address[]) public positionLendTokenData;
+    mapping(uint256 => address[]) internal positionLendTokenData;
     mapping(uint256 => address[]) public positionBorrowTokenData;

     mapping(uint256 => mapping(address => uint256)) public userBorrowShares;
```


2. #### Make `poolTokenAddresses` array variable `private` or `internal` since a getter function was defined for it.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L123
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L884-#L892

The solidity compiler would automatically create a getter function for the `poolTokenAddresses` array since it is declared as a `public` variable but a getter function `getPoolTokenAdressesByIndex()` was also declared in the `FeeManager` contract for the same variable thereby making it two getter functions for the same variable in the contract. We could rectify this issue by making the `poolTokenAddresses` variable private or internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

```solidity
file: contracts/FeeManager/DeclarationsFeeManager.sol

123:    address[] public poolTokenAddresses;


file: contracts/FeeManager/FeeManager.sol

884:    function getPoolTokenAdressesByIndex(
885:        uint256 _index
886:    )
887:        external
888:        view
889:        returns (address)
890:    {
891:        return poolTokenAddresses[_index];
892:    }
```

```diff
diff --git a/contracts/FeeManager/DeclarationsFeeManager.sol b/contracts/FeeManager/DeclarationsFeeManager.sol
index ba7eed7..e10f9be 100644
--- a/contracts/FeeManager/DeclarationsFeeManager.sol
+++ b/contracts/FeeManager/DeclarationsFeeManager.sol
@@ -120,7 +120,7 @@ contract DeclarationsFeeManager is FeeManagerEvents, OwnableMaster {
     uint256 public paybackIncentive;

     // Array of pool tokens in wiseLending
-    address[] public poolTokenAddresses;
+    address[] internal poolTokenAddresses;

     // Address of incentive master
     address public incentiveMaster;
```




### [G-06]  Using `storage` instead of `memory` for struct saves gas

### Instances
1. #### Make `borrowData` a storage variable
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1099

We could make the `_increaseResonanceFactor()` function more gas efficient if we declare the `borrowData` struct variable as  `storage` rather than `memory` . In implementing this we would avoid having to copy all the members of `BorrowRatesEntry` struct from storage to memory which would incur a Gcoldsload (2100 gas) for each storage slot read which totals to 10500 gas units. Declaring the `borrowData` struct variable as a `storage` variable would only cost 100 gas units (Gwarmsload) which totals to 300 gas for the members read in the function (deltaPole, pole and maxPole) and members read more than once in the function are cached in stack variable and the stack variable used subsequently. The diff below shows how the code should be refactored: 

```solidity
file: contracts/MainHelper.sol

1094:    function _increaseResonanceFactor(
1095:        address _poolToken
1096:    )
1097:        private
1098:    {
1099:        BorrowRatesEntry memory borrowData = borrowRatesData[
1100:            _poolToken
1101:        ];
1102:
1103:        uint256 delta = borrowData.deltaPole
1104:            * (block.timestamp - timestampsPoolData[_poolToken].timeStampScaling);
1105:
1106:        uint256 sum = delta
1107:            + borrowData.pole;
1108:
1109:        uint256 setValue = sum > borrowData.maxPole
1110:            ? borrowData.maxPole
1111:            : sum;
1112:
1113:        _setPole(
1114:            _poolToken,
1115:            setValue
1116:        );
1117:    }
```

```diff
diff --git a/contracts/MainHelper.sol b/contracts/MainHelper.sol
index 46854bc..a1b4d0f 100644
--- a/contracts/MainHelper.sol
+++ b/contracts/MainHelper.sol
@@ -1096,7 +1096,7 @@ abstract contract MainHelper is WiseLowLevelHelper {
     )
         private
     {
-        BorrowRatesEntry memory borrowData = borrowRatesData[
+        BorrowRatesEntry storage borrowData = borrowRatesData[
             _poolToken
         ];

@@ -1106,8 +1106,9 @@ abstract contract MainHelper is WiseLowLevelHelper {
         uint256 sum = delta
             + borrowData.pole;

-        uint256 setValue = sum > borrowData.maxPole
-            ? borrowData.maxPole
+        unit256 _maxPole = borrowData.maxPole;
+        uint256 setValue = sum > _maxPole
+            ? _maxPole
             : sum;
```
```
Estimated gas saved: 10200 gas units.
```




## [G-07] Unchecked Divisions
Divisions which do not divide by -X cannot overflow or overflow so such operations can be unchecked to save gas.

#### Please note this instance was not included in the bots report

### 5 Instances
1. #### unchecked uint divisions in `WiseSecurity.overallLendingAPY()` function to reduce its gas cost.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L491-#L492

```solidity
file: contracts/WiseSecurity/WiseSecurity.sol

442:    function overallLendingAPY(
443:        uint256 _nftId
444:    )
445:        external
446:        view
447:        returns (uint256)
448:    {
449:        uint256 len = WISE_LENDING.getPositionLendingTokenLength(
450:            _nftId
451:        );
.
.
.
491:        return weightedRate     //@audit can be unchecked
492:            / overallETH;
493:    }
```

```diff
diff --git a/contracts/WiseSecurity/WiseSecurity.sol b/contracts/WiseSecurity/WiseSecurity.sol
index d2cfb24..a101048 100644
--- a/contracts/WiseSecurity/WiseSecurity.sol
+++ b/contracts/WiseSecurity/WiseSecurity.sol
@@ -487,9 +487,11 @@ contract WiseSecurity is WiseSecurityHelper, ApprovalHelper {
                 ++i;
             }
         }
-
-        return weightedRate
+        unchecked {
+            return weightedRate
             / overallETH;
+        }
+
     }
```
```
Estimated gas saved: 45 gas units
```

2. #### unchecked uint divisions in `WiseSecurity.overallBorrowAPY()` function to reduce its gas cost.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L548-#L549

```solidity
file: WiseSecurity/WiseSecurity.sol

499:    function overallBorrowAPY(
500:        uint256 _nftId
501:    )
502:        external
503:        view
504:        returns (uint256)
505:    {
506:        uint256 len = WISE_LENDING.getPositionBorrowTokenLength(
507:            _nftId
508:        );
.
.
.
548:        return weightedRate     // @audit can be unchecked
549:            / overallETH;
550:    }
```

```diff
diff --git a/contracts/WiseSecurity/WiseSecurity.sol b/contracts/WiseSecurity/WiseSecurity.sol
index d2cfb24..568e44c 100644
--- a/contracts/WiseSecurity/WiseSecurity.sol
+++ b/contracts/WiseSecurity/WiseSecurity.sol
@@ -545,8 +545,10 @@ contract WiseSecurity is WiseSecurityHelper, ApprovalHelper {
             }
         }

-        return weightedRate
+        unchecked {
+            return weightedRate
             / overallETH;
+        }
     }
```
```
Estimated gas saved: 45 gas units
```

3. #### unchecked uint divisions in `MainHelper._updatePseudoTotalAmounts()` function to reduce its gas cost.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L525-#L526
```solidity
file: contracts/MainHelper.sol

500:    function _updatePseudoTotalAmounts(
501:        address _poolToken
502:    )
503:        private
504:    {
505:        uint256 currentTime = block.timestamp;
.
.
.
525:        uint256 amountInterest = bareIncrease   //@audit can be unchecked
526:            / PRECISION_FACTOR_YEAR;
527:
528:        uint256 feeAmount = amountInterest
529:            * globalPoolData[_poolToken].poolFee
530:            / PRECISION_FACTOR_E18;
.
.
.
577:    }
```

```diff
diff --git a/contracts/MainHelper.sol b/contracts/MainHelper.sol
index 46854bc..de070c3 100644
--- a/contracts/MainHelper.sol
+++ b/contracts/MainHelper.sol
@@ -521,9 +521,11 @@ abstract contract MainHelper is WiseLowLevelHelper {
         }

         delete bufferIncrease[_poolToken];
-
-        uint256 amountInterest = bareIncrease
-            / PRECISION_FACTOR_YEAR;
+        uint256 amountInterest;
+        unchecked {
+            amountInterest = bareIncrease / PRECISION_FACTOR_YEAR;
+        }
+
```
```
Estimated gas saved: 45 gas units
```

4. #### unchecked uint divisions in `MainHelper._updatePseudoTotalAmounts()` function to reduce its gas cost.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L403-#L404

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

386:    function previewDistribution()
387:        public
388:        view
389:        returns (uint256)
390:    {
391:        if (totalLpAssetsToDistribute == 0) {
392:            return 0;
393:        }
394:
395:        if (block.timestamp == lastInteraction) {
396:            return 0;
397:        }
398:
399:        if (totalLpAssetsToDistribute < ONE_WEEK) {
400:            return totalLpAssetsToDistribute;
401:        }
402:
403:        uint256 currentRate = totalLpAssetsToDistribute     // @audit can be unchecked
404:            / ONE_WEEK;
405:
406:        uint256 additonalAssets = currentRate
407:            * (block.timestamp - lastInteraction);
408:
409:        if (additonalAssets > totalLpAssetsToDistribute) {
410:            return totalLpAssetsToDistribute;
411:        }
412:
413:        return additonalAssets;
414:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..c39ed9c 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -400,8 +400,11 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
             return totalLpAssetsToDistribute;
         }

-        uint256 currentRate = totalLpAssetsToDistribute
-            / ONE_WEEK;
+        uint256 currentRate;
+        unchecked {
+            currentRate = totalLpAssetsToDistribute / ONE_WEEK;
+        }
+

         uint256 additonalAssets = currentRate
             * (block.timestamp - lastInteraction);
```
```
Estimated gas saved: 45 gas units
```

4. #### unchecked uint divisions in `AaveHelper._underlyingAsset()` function to reduce its gas cost.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L322-#L323

```solidity
file: contracts/WrapperHub/AaveHelper.sol

315:    function getAavePoolAPY(
316:        address _underlyingAsset
317:    )
318:        public
319:        view
320:        returns (uint256)
321:    {
322:        return AAVE.getReserveData(_underlyingAsset).currentLiquidityRate   //@audit can be unchecked
323:            / PRECISION_FACTOR_E9;
324    }
```

```diff
diff --git a/contracts/WrapperHub/AaveHelper.sol b/contracts/WrapperHub/AaveHelper.sol
index eca55d1..a4633bc 100644
--- a/contracts/WrapperHub/AaveHelper.sol
+++ b/contracts/WrapperHub/AaveHelper.sol
@@ -319,7 +319,9 @@ abstract contract AaveHelper is Declarations {
         view
         returns (uint256)
     {
-        return AAVE.getReserveData(_underlyingAsset).currentLiquidityRate
+        unchecked {
+            return AAVE.getReserveData(_underlyingAsset).currentLiquidityRate
             / PRECISION_FACTOR_E9;
+        }
     }
 }
```
```
Estimated gas saved: 45 gas units
```



## [G-08] State variables read in loop in `PendlePowerFarmToken._calculateRewardsClaimedOutside()` function should be avoided.

In the `PendlePowerFarmToken._calculateRewardsClaimedOutside()` function as shown below the state variables `PENDLE_MARKET` and `PENDLE_POWER_FARM_CONTROLLER` (`PENDLE_POWER_FARM_CONTROLLER` was read multiple times in the loop) were read in a loop. Doing it this way is gas in-efficient rather we should cache these state variables before the loop then subsitute the cached variables for the state variables in the loop implementing this would help avoid `SLOAD` Gwarmaccess (100 gas units) and replace them with cheaper stack reads. The diff below sgows how the function should be refactored:

- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L239
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L243
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L244
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L249

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

218:    function _calculateRewardsClaimedOutside()
219:        internal
220:        returns (uint256[] memory)
221:    {
222:        address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(
223:            UNDERLYING_PENDLE_MARKET
224:        );
225:
226:        uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
227:            UNDERLYING_PENDLE_MARKET
228:        );
229:
230:        uint256 l = rewardTokens.length;
231:        uint256[] memory rewardsOutsideArray = new uint256[](l);
232:
233:        uint256 i;
234:        uint128 index;
235:
236:        while (i < l) {
237:            UserReward memory userReward = _getUserReward(
238:                rewardTokens[i],
239:                PENDLE_POWER_FARM_CONTROLLER    //@audit state variable read in loop
240:            );
241:
242:            if (userReward.accrued > 0) {
243:                PENDLE_MARKET.redeemRewards(        //@audit state variable read in loop
244:                    PENDLE_POWER_FARM_CONTROLLER    //@audit state variable read in loop
245:                );
246:
247:                userReward = _getUserReward(
248:                    rewardTokens[i],
249:                    PENDLE_POWER_FARM_CONTROLLER    //@audit state variable read in loop
250:                );
251:            }
252:
253:            index = userReward.index;
254:
255:            if (lastIndex[i] == 0 && index > 0) {
256:                rewardsOutsideArray[i] = 0;
257:                _overWriteIndex(
258:                    i
259:                );
260:                unchecked {
261:                    ++i;
262:                }
263:                continue;
264:            }
265:
266:            if (index == lastIndex[i]) {
267:                rewardsOutsideArray[i] = 0;
268:                unchecked {
269:                    ++i;
270:                }
271:                continue;
272:            }
273:
274:            uint256 indexDiff = index
275:                - lastIndex[i];
276:
277:            uint256 activeBalance = _getActiveBalance();
278:            uint256 totalLpAssetsCurrent = totalLpAssets();
279:            uint256 lpBalanceController = _getBalanceLpBalanceController();
280:
281:            bool scaleNecessary = totalLpAssetsCurrent < lpBalanceController;
282:
283:            rewardsOutsideArray[i] = scaleNecessary
284:                ? indexDiff
285:                    * activeBalance
286:                    * totalLpAssetsCurrent
287:                    / lpBalanceController
288:                    / PRECISION_FACTOR_E18
289:                : indexDiff
290:                    * activeBalance
291:                    / PRECISION_FACTOR_E18;
292:
293:            _overWriteIndex(
294:                i
295:            );
296:
297:            unchecked {
298:                ++i;
299:            }
300:        }
301:
302:        return rewardsOutsideArray;
303:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..6cb42de 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -233,20 +233,22 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
         uint256 i;
         uint128 index;

+        IPendleMarket _PENDLE_MARKET = PENDLE_MARKET;
+        address _PENDLE_POWER_FARM_CONTROLLER = PENDLE_POWER_FARM_CONTROLLER;
         while (i < l) {
             UserReward memory userReward = _getUserReward(
                 rewardTokens[i],
-                PENDLE_POWER_FARM_CONTROLLER
+                _PENDLE_POWER_FARM_CONTROLLER
             );

             if (userReward.accrued > 0) {
-                PENDLE_MARKET.redeemRewards(
-                    PENDLE_POWER_FARM_CONTROLLER
+                _PENDLE_MARKET.redeemRewards(
+                    _PENDLE_POWER_FARM_CONTROLLER
                 );

                 userReward = _getUserReward(
                     rewardTokens[i],
-                    PENDLE_POWER_FARM_CONTROLLER
+                    _PENDLE_POWER_FARM_CONTROLLER
                 );
             }
```
```
Estimated gas saved: 388 gas units per iteration
```



## [G-09]  Move lesser gas costing  checks to the top
Revert() statements that check input arguments or cost lesser gas should be at the top of the function.
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### 1 Instance
1. #### Move `if (_nftIdLiquidator == _nftId) {revert InvalidLiquidator()}` to the top of the function.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L258-#L276

In the `_checkLiquidatorNft()` function as shown below the if-revert statement `if (positionLocked[_nftIdLiquidator] == true) {revert LiquidatorIsInPowerFarm()}` is more gas consumimg than the if-revert statement `if (_nftIdLiquidator == _nftId) {revert InvalidLiquidator()}` as the former reads from state which cost `2100` gas units. We can make the `_checkLiquidatorNft()` function more gas efficient by moving the cheaper if-revert statement `if (_nftIdLiquidator == _nftId) {revert InvalidLiquidator()}` to the top of the function so that in scenarios where the cheaper revert statement fails the function would revert without having to read from state which is expensive. The diff below shows how the code should be refactored:

```solidity
file: contracts/MainHelper.sol

258:    function _checkLiquidatorNft(
259:        uint256 _nftId,
260:        uint256 _nftIdLiquidator
261:    )
262:        internal
263:        view
264:    {
265:        if (positionLocked[_nftIdLiquidator] == true) {
266:            revert LiquidatorIsInPowerFarm();
267:        }
268:
269:        if (_nftIdLiquidator == _nftId) {
270:            revert InvalidLiquidator();
271:        }
272:
273:        if (_nftIdLiquidator >= POSITION_NFT.getNextExpectedId()) {
274:            revert InvalidLiquidator();
275:        }
276:    }
```

```diff
diff --git a/contracts/MainHelper.sol b/contracts/MainHelper.sol
index 46854bc..d84e200 100644
--- a/contracts/MainHelper.sol
+++ b/contracts/MainHelper.sol
@@ -262,14 +262,14 @@ abstract contract MainHelper is WiseLowLevelHelper {
         internal
         view
     {
-        if (positionLocked[_nftIdLiquidator] == true) {
-            revert LiquidatorIsInPowerFarm();
-        }
-
         if (_nftIdLiquidator == _nftId) {
             revert InvalidLiquidator();
         }

+        if (positionLocked[_nftIdLiquidator] == true) {
+            revert LiquidatorIsInPowerFarm();
+        }
+
         if (_nftIdLiquidator >= POSITION_NFT.getNextExpectedId()) {
             revert InvalidLiquidator();
         }
```
```
Estimated gas saved: 2100 gas units
```





## [G-10] Using calldata instead of memory for read-only arguments in external functions saves gas

### 13 Instances
1. #### Refactor the `PendlePowerFarmToken.initialize()` function to use `calldata` in place of `memory` for the `_tokenName` amd `_symbolName` parameters.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L685-#L686

Since the string parameters `_tokenName` amd `_symbolName` were not modified, we can reduce the gas cost of the `PendlePowerFarmToken.initialize()` function by using `calldata` in place of `memory` in the defination of the `_tokenName` amd `_symbolName` parameters. In implementing this we would avoid having to copy the strings from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

682:    function initialize(
683:        address _underlyingPendleMarket,
684:        address _pendleController,
685:        string memory _tokenName,   //@audit use calldata in-place of memory
686:        string memory _symbolName,   //@audit use calldata in-place of memory
687:        uint16 _maxCardinality
688:    )
689:        external
690:    {
.
.
.
732:    }
```
```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..2f53c1d 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -682,8 +682,8 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
     function initialize(
         address _underlyingPendleMarket,
         address _pendleController,
-        string memory _tokenName,
-        string memory _symbolName,
+        string calldata _tokenName,
+        string calldata _symbolName,
         uint16 _maxCardinality
     )
```

2. #### Refactor the `FeeManager.revokeBeneficial()` function to use `calldata` in place of `memory` for the `_feeTokens` parameter.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L573

Since the `_feeTokens` array is not modified, we can reduce the gas cost of the `FeeManager.revokeBeneficial()` function by using `calldata` in place of `memory` in the defination of the `_feeTokens` parameter. In implementing this we would avoid having to copy the array from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/FeeManager/FeeManager.sol

571:    function revokeBeneficial(
572:        address _user,
573:        address[] memory _feeTokens    //@audit use calldata in-place of memory
574:    )
575:        external
576:        onlyMaster
577:    {
.
.
.
598:    }
```

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..d32aa04 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -570,7 +570,7 @@ contract FeeManager is FeeManagerHelper {
      */
     function revokeBeneficial(
         address _user,
-        address[] memory _feeTokens
+        address[] calldata _feeTokens
     )
         external
         onlyMaster
```


3. #### Refactor the `PendlePowerFarmLeverageLogic.receiveFlashLoan()` function to use `calldata` in place of `memory` for the `_flashloanToken`, `_flashloanAmounts`, `_feeAmounts` and `_userData` parameters.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L60-#L63

Since the `_flashloanToken`, `_flashloanAmounts`, `_feeAmounts` arrays and the `_userData` were not modified, we can reduce the gas cost of the `PendlePowerFarmLeverageLogic.receiveFlashLoan()` function by using `calldata` in place of `memory` in the defination of these parameters. In implementing this we would avoid having to copy the arrays from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: /contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

59:     function receiveFlashLoan(
60:         IERC20[] memory _flashloanToken,    //@audit use calldata in-place of memory
61:         uint256[] memory _flashloanAmounts,    //@audit use calldata in-place of memory
62:         uint256[] memory _feeAmounts,    //@audit use calldata in-place of memory
63:         bytes memory _userData    //@audit use calldata in-place of memory
64:     )
65:         external
66:     {
.
.
.
129:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol b/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol
index 70644f2..855189c 100644
--- a/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol
+++ b/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol
@@ -57,10 +57,10 @@ abstract contract PendlePowerFarmLeverageLogic is
      * logic. Overwritten with opening flows.
      */
     function receiveFlashLoan(
-        IERC20[] memory _flashloanToken,
-        uint256[] memory _flashloanAmounts,
-        uint256[] memory _feeAmounts,
-        bytes memory _userData
+        IERC20[] calldata _flashloanToken,
+        uint256[] calldata _flashloanAmounts,
+        uint256[] calldata _feeAmounts,
+        bytes calldata _userData
     )
         external
```


4. #### Refactor the `PendlePowerFarmController.addPendleMarket()` function to use `calldata` in place of `memory` for the `_tokenName` and `_symbolName` parameters.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L213-#L214

Since the strings parameters `_tokenName` and `_symbolName` were not modified, we can reduce the gas cost of the `PendlePowerFarmController.addPendleMarket()` function by using `calldata` in place of `memory` in the defination of these parameters. In implementing this we would avoid having to copy the strings from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

211:    function addPendleMarket(
212:        address _pendleMarket,
213:        string memory _tokenName,    //@audit use calldata in-place of memory
214:        string memory _symbolName,    //@audit use calldata in-place of memory
215:        uint16 _maxCardinality
216:    )
217:        external
218:        onlyMaster
219:    {
.
.
.
291    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.solindex 15cb863..813cab4 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
@@ -210,8 +210,8 @@ contract PendlePowerFarmController is PendlePowerFarmControllerHelper {

     function addPendleMarket(
         address _pendleMarket,
-        string memory _tokenName,
-        string memory _symbolName,
+        string calldata _tokenName,
+        string calldata _symbolName,
         uint16 _maxCardinality
     )
```


5. #### Refactor the `PositionNFTs.setBaseURI()` function to use `calldata` in place of `memory` for the `_newBaseURI` parameter.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L320

Since the string parameter `_newBaseURI` was not modified, we can reduce the gas cost of the `PositionNFTs.setBaseURI()` function by using `calldata` in place of `memory` in the defination of the `_newBaseURI` parameter. In implementing this we would avoid having to copy the string from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/PositionNFTs.sol

319:    function setBaseURI(
320:        string memory _newBaseURI    //@audit use calldata in-place of memory
321:    )
322:        external
323:        onlyMaster
324:    {
325:        baseURI = _newBaseURI;
326:    }
```

```diff
diff --git a/contracts/PositionNFTs.sol b/contracts/PositionNFTs.sol
index fb680ec..b43253d 100644
--- a/contracts/PositionNFTs.sol
+++ b/contracts/PositionNFTs.sol
@@ -317,7 +317,7 @@ contract PositionNFTs is ERC721Enumerable, OwnableMaster {
      * @dev Allows to update base target for MetaData.
      */
     function setBaseURI(
-        string memory _newBaseURI
+        string calldata _newBaseURI
     )
         external
         onlyMaster
```

6. #### Refactor the `PositionNFTs.setBaseExtension()` function to use `calldata` in place of `memory` for the `_newBaseExtension` parameter.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L329

Since the string parameter `_newBaseExtension` was not modified, we can reduce the gas cost of the `PositionNFTs.setBaseExtension()` function by using `calldata` in place of `memory` in the defination of the `_newBaseExtension` parameter. In implementing this we would avoid having to copy the string from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/PositionNFTs.sol

328:    function setBaseExtension(
329:        string memory _newBaseExtension    //@audit use calldata in-place of memory
330:    )
331:        external
332:        onlyMaster
333:    {
334:        baseExtension = _newBaseExtension;
335:    }
```

```diff
diff --git a/contracts/PositionNFTs.sol b/contracts/PositionNFTs.sol
index fb680ec..ed078dd 100644
--- a/contracts/PositionNFTs.sol
+++ b/contracts/PositionNFTs.sol
@@ -326,7 +326,7 @@ contract PositionNFTs is ERC721Enumerable, OwnableMaster {
     }

     function setBaseExtension(
-        string memory _newBaseExtension
+        string calldata _newBaseExtension
     )
         external
         onlyMaster
```

7. #### Refactor the `PendlePowerFarmTokenFactory.deploy()` function to use `calldata` in place of `memory` for the `_tokenName` and `_symbolName` parameters.
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L37-#L38

Since the strings parameters `_tokenName` and `_symbolName` were not modified, we can reduce the gas cost of the `PendlePowerFarmTokenFactory.deploy()` function by using `calldata` in place of `memory` in the defination of these parameters. In implementing this we would avoid having to copy the strings from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

35:    function deploy(
36:        address _underlyingPendleMarket,
37:        string memory _tokenName,     //@audit use calldata in-place of memory
38:        string memory _symbolName,    //@audit use calldata in-place of memory
39:        uint16 _maxCardinality
40:    )
41:        external
42:        returns (address)
43:    {
44:        if (msg.sender != PENDLE_POWER_FARM_CONTROLLER) {
45:            revert DeployForbidden();
46:        }
47:
48:        return _clone(
49:            _underlyingPendleMarket,
50:            _tokenName,
51:            _symbolName,
52:            _maxCardinality
53:        );
54:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol
index 0936d72..836a38b 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol
@@ -34,8 +34,8 @@ contract PendlePowerFarmTokenFactory {

     function deploy(
         address _underlyingPendleMarket,
-        string memory _tokenName,
-        string memory _symbolName,
+        string calldata _tokenName,
+        string calldata _symbolName,
         uint16 _maxCardinality
     )
```




## [G-11] Caching the length of calldata increases gas cost instead of reducing it

### Proof of Concept
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CacheCallDataLength {
    

    function calculateTotal(uint256[5] memory numbers) public pure returns(uint256 total) {
        uint256 len = numbers.length;

        for(uint i; i < len; ++i) {
            total += numbers[i];
        }

    }
}

```
```
test for test/CacheCallDataLength.t.sol:CacheCallDataLengthTest
[PASS] test_calculateTotal() (gas: 1633)
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;
contract NoCacheCallDataLength {
    

    function calculateTotal(uint256[5] memory numbers) public pure returns(uint256 total) {

        for(uint i; i < numbers.length; ++i) {
            total += numbers[i];
        }
        
    }
}
```
```
test for test/NoCacheCallDataLength.t.sol:NoCacheCallDataLengthTest
[PASS] test_calculateTotal() (gas: 1628)
```

### 4 Instances
1. #### Reduce gas cost of `FeeManager.setBeneficial()` function by not caching the length of calldata variable `_feeTokens`
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L546


```solidity
file: contracts/FeeManager/FeeManager.sol

538:    function setBeneficial(
539:        address _user,
540:        address[] calldata _feeTokens
541:    )
542:        external
543:        onlyMaster
544:    {
545:        uint256 i;
546:        uint256 l = _feeTokens.length;  //@audit don't cache calldata length
547:
548:        while (i < l) {
549:            _setAllowedTokens(
550:                _user,
551:                _feeTokens[i],
552:                true
553:            );
554:
555:            unchecked {
556:                ++i;
557:            }
.
.
.
565:    }
```

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..9e78788 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -543,9 +543,8 @@ contract FeeManager is FeeManagerHelper {
         onlyMaster
     {
         uint256 i;
-        uint256 l = _feeTokens.length;

-        while (i < l) {
+        while (i < _feeTokens.length) {
             _setAllowedTokens(
                 _user,
                 _feeTokens[i],
```

2. #### Reduce gas cost of `FeeManager.setBeneficial()` function by not caching the length of calldata variable `_feeTokens`
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L293

```solidity
file: contracts/WiseOracleHub/WiseOracleHub.sol

266:    function addTwapOracleDerivative(
267:        address _tokenAddress,
268:        address _partnerTokenAddress,
269:        address[2] calldata _uniPools,
270:        address[2] calldata _token0Array,
271:        address[2] calldata _token1Array,
272:        uint24[2] calldata _feeArray
273:    )
274:        external
275:        onlyMaster
276:    {
.
.
.
291:        uint256 i;
292:        address pool;
293:        uint256 length = _uniPools.length;  //@audit don't cache calldata length
294:
295:        while (i < length) {
296:            pool = _getPool(
297:                _token0Array[i],
298:                _token1Array[i],
299:                _feeArray[i]
300:            );
301:
302:            _validatePoolAddress(
303:                pool,
304:                _uniPools[i]
305:            );
306:
307:            unchecked {
308:                ++i;
309:            }
310:        }
.
.
.
321:    }
```

```diff
diff --git a/contracts/WiseOracleHub/WiseOracleHub.sol b/contracts/WiseOracleHub/WiseOracleHub.sol
index 12fe0c8..f105337 100644
--- a/contracts/WiseOracleHub/WiseOracleHub.sol
+++ b/contracts/WiseOracleHub/WiseOracleHub.sol
@@ -290,9 +290,8 @@ contract WiseOracleHub is OracleHelper {

         uint256 i;
         address pool;
-        uint256 length = _uniPools.length;

-        while (i < length) {
+        while (i < _uniPools.length) {
             pool = _getPool(
                 _token0Array[i],
                 _token1Array[i],

```

3. #### Reduce gas cost of `WiseOracleHub.addOracleBulk()` function by not caching the length of calldata variable `_tokenAddresses`
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L388


```solidity
file: contracts/WiseOracleHub/WiseOracleHub.sol

379:    function addOracleBulk(
380:        address[] calldata _tokenAddresses,
381:        IPriceFeed[] calldata _priceFeedAddresses,
382:        address[][] calldata _underlyingFeedTokens
383:    )
384:        external
385:        onlyMaster
386:    {
387:        uint256 i;
388:        uint256 l = _tokenAddresses.length;
389:
390:        while (i < l) {
391:            _addOracle(
392:                _tokenAddresses[i],
393:                _priceFeedAddresses[i],
394:                _underlyingFeedTokens[i]
395:            );
396:
397:            unchecked {
398:                ++i;
399:            }
400:        }
401:    }
```

```diff
diff --git a/contracts/WiseOracleHub/WiseOracleHub.sol b/contracts/WiseOracleHub/WiseOracleHub.sol
index 12fe0c8..ecc8e55 100644
--- a/contracts/WiseOracleHub/WiseOracleHub.sol
+++ b/contracts/WiseOracleHub/WiseOracleHub.sol
@@ -385,9 +385,8 @@ contract WiseOracleHub is OracleHelper {
         onlyMaster
     {
         uint256 i;
-        uint256 l = _tokenAddresses.length;

-        while (i < l) {
+        while (i < _tokenAddresses.length) {
             _addOracle(
                 _tokenAddresses[i],
                 _priceFeedAddresses[i],
```


4. #### Reduce gas cost of `WiseOracleHub.recalibrateBulk()` function by not caching the length of calldata variable `_tokenAddresses`
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L507

```solidity
file: contracts/WiseOracleHub/WiseOracleHub.sol

501:    function recalibrateBulk(
502:        address[] calldata _tokenAddresses
503:    )
504:        external
505:    {
506:        uint256 i;
507:        uint256 l = _tokenAddresses.length;
508:
509:        while (i < l) {
510:            _recalibrate(
511:                _tokenAddresses[i]
512:            );
513:
514:            unchecked {
515:                ++i;
516:            }
517:        }
518:    }
```

```diff
diff --git a/contracts/WiseOracleHub/WiseOracleHub.sol b/contracts/WiseOracleHub/WiseOracleHub.sol
index 12fe0c8..169243e 100644
--- a/contracts/WiseOracleHub/WiseOracleHub.sol
+++ b/contracts/WiseOracleHub/WiseOracleHub.sol
@@ -504,9 +504,8 @@ contract WiseOracleHub is OracleHelper {
         external
     {
         uint256 i;
-        uint256 l = _tokenAddresses.length;

-        while (i < l) {
+        while (i < _tokenAddresses.length) {
             _recalibrate(
                 _tokenAddresses[i]
             );
```




## [G-12] Avoid emitting event on every iteration
Expensive operations should always try to be avoided within loops. Such operations include: reading/writing to storage, heavy calculations, external calls, and emitting events. In this instance, an event is being emitted every iteration. Events have a base cost of Glog (375 gas) per emit and Glogdata (8 gas) * number of bytes in event. We can avoid incurring those costs each iteration by emitting the event outside of the loop.

### 1 Instance
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L156-#L160



```solidity
file: contracts/FeeManager/FeeManager.sol

135:    function setPoolFeeBulk(
136:        address[] calldata _poolTokens,
137:        uint256[] calldata _newFees
138:    )
139:        external
140:        onlyMaster
141:    {
142:        uint256 i;
143:        uint256 l = _poolTokens.length;
144:
145:        while (i < l) {
146:
147:            _checkValue(
148:                _newFees[i]
149:            );
150:
151:            WISE_LENDING.setPoolFee(
152:                _poolTokens[i],
153:                _newFees[i]
154:            );
155:
156:            emit PoolFeeChanged(    //@audit event emmited in loop
157:                _poolTokens[i],
158:                _newFees[i],
159:                block.timestamp
160:            );
161:
162:            unchecked {
163:                ++i;
164:            }
165:        }
166:    }
```



## [G-13] memory variable should created outside of loop
memory variable should created outside of loop, and get overriden with each iteration of loop, By doing so we save gas cost for memory variable creation in each iteration.

### Proof of Concept
```solidity
contract VarInLoop {

    uint256[] numbers = [1,2,3,4,5];
    uint256 total;

    function double(uint256 num) public pure returns(uint256 ans) {
        ans = num * 2;
    }

    function doubleNumbers() public {
        uint256 len = numbers.length;

        for(uint256 i; i < len; ++i) {
            uint256 doubleNum =  double(numbers[i]);
            total = total + doubleNum;
        }
    }
}
```
```
test for test/VarInLoop.t.sol:VarInLoopTest
[PASS] test_doubleNumbers() (gas: 38185)
```


```
contract VarOutLoop {

    uint256[] numbers = [1,2,3,4,5];
    uint256 total;

    function double(uint256 num) public pure returns(uint256 ans) {
        ans = num * 2;
    }

    function doubleNumbers() public {
        uint256 len = numbers.length;

        uint256 doubleNum;
        for(uint256 i; i < len; ++i) {
            doubleNum =  double(numbers[i]);
            total = total + doubleNum;
        }
    }
}
```
```
test for test/VarOutLoop.t.sol:VarOutLoopTest
[PASS] test_doubleNumbers() (gas: 38170)
```


### 13 Instances
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L711
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L237
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L274
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L277
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L278
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L279
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L281
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L627
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L260
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L256
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L292
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L27
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L59






## [G-14] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.
the recommended Mitigation Steps is that Functions guaranteed to revert when called by normal users can be marked payable.
https://gist.github.com/itsmetechjay/09ae039958715d881a47209f74af1b7c#g-8-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable

### Instances
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L859-#L896
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L939-#L967
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1055-#L1080
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1412-#L1428
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L85-#L100
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L142-#L187
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L193-#L232
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L405-#L436
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L980-#L988
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L995-#L1003
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1032-#L1039
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1111-#L1122
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L592-#L603
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L49-#L60
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L66-#L77
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L82-#L102
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L108-#L129
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L135-#L166
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L173-#L189
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L214-#L226
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L232-#L244
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L390-#L406
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L412-#L432
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L438-#L487
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L494-#L508
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L514-#L531
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L538-#L565
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L571-#L598
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L32-#L51
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L211-#L291
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L293-#L319
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L321-#L332
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L334-#L359
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L364-#L429
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L431-#L449
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L451-#L495
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L497-#L524
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L526-#L542
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L544-#L564
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L566-#L579
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L600-#L610
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L612-#L644
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L140-#L159
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L218-#L260
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L266-#L321
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L359-#L372
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L379-#L401
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L403-#L412
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L44-#L56
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L64-#L77
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L611-#L628






## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.
