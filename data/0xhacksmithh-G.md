### [Gas-01] Less gas consuming `checks` should be moved to the top of function
So that it will revert eailer before state read.
```file: MainHelper.sol```
```diff
    function _checkLiquidatorNft(
        uint256 _nftId,
        uint256 _nftIdLiquidator
    )
        internal
        view
    {
+       if (_nftIdLiquidator == _nftId) { 
+           revert InvalidLiquidator();
+       }

        if (positionLocked[_nftIdLiquidator] == true) { 
            revert LiquidatorIsInPowerFarm();
        }

-       if (_nftIdLiquidator == _nftId) { 
-           revert InvalidLiquidator();
-       }

        if (_nftIdLiquidator >= POSITION_NFT.getNextExpectedId()) {
            revert InvalidLiquidator();
        }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L265-L275
```

### [Gas-02] `_compareMinMax()` could be re arranged to save gas
`Before`
- 2 external function call to find out `maxAnswer` & `minAnswer`
- then value checked against both max & min
     - if failed both external call gas wasted


`After`
- 1 external call to fetch maxAnswer
- then check against value
      - if failed only 1 external call gas cost wasted
      - if success
           - second external call

```file : OracleHelper.sol```

```diff
    function _compareMinMax(
        IAggregator _tokenAggregator,
        int192 _answer
    )
        internal
        view
    {
-       int192 maxAnswer = _tokenAggregator.maxAnswer();
-       int192 minAnswer = _tokenAggregator.minAnswer();

-       if (_answer > maxAnswer || _answer < minAnswer) { 
-           revert OracleIsDead();
-       }


+       int192 maxAnswer = _tokenAggregator.maxAnswer();
+       if (_answer > maxAnswer){
+                 revert OracleIsDead();
+       }

+       int192 minAnswer = _tokenAggregator.minAnswer();
+       if (_answer < minAnswer) { 
+           revert OracleIsDead();
+       }
    }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L94-L99
```

### [Gas-03] via Re-arranging state variables extra storage slots can be saved
```File : PendleLpOracle.sol```
One extra storage slot saved here
```diff
    IPendleSy public immutable PENDLE_SY;
    IPriceFeed public immutable FEED_ASSET;
    IOraclePendle public immutable ORACLE_PENDLE_PT;

    uint8 internal constant FEED_DECIMALS = 18;
+   uint32 public immutable TWAP_DURATION;

    uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;

    // Precision factor for computations.
    uint256 internal constant PRECISION_FACTOR_E18 = 1E18;

    // -- Twap duration in seconds ---
-   uint32 public immutable TWAP_DURATION;

    // -- Farm description --
    string public name;
```

### [Gas-04] State variable can be cached rather than reading that multile times inside a function
```file : WiseLending.sol```
```diff
    {
+      uint256 _powerFarmCheck = powerFarmCheck;
        _checkHealthState(
            _nftId,
-          powerFarmCheck 
+          _powerFarmCheck
        );

        if (powerFarmCheck == true) { 
-           powerFarmCheck = false;
+           _powerFarmCheck = false;
        }
    }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L82-L89
```


#### `borrowPoolData[_poolToken]` & `lendingPoolData[_poolToken]` could be cached in memory
```diff

-       uint256 borrowSharePrice = borrowPoolData[_poolToken].pseudoTotalBorrowAmount
            * PRECISION_FACTOR_E18
            / borrowPoolData[_poolToken].totalBorrowShares;  

        _validateParameter(
            MIN_BORROW_SHARE_PRICE,
            borrowSharePrice
        );

        return (
            lendingPoolData[_poolToken].pseudoTotalPool
                * PRECISION_FACTOR_E18
                / lendingPoolData[_poolToken].totalDepositShares, 
            borrowSharePrice
        );
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L175-L189
```


#### `WISE_SECURITY` could be cached rather than re-calling it 2 times
```diff
        data.maxFeeETH = WISE_SECURITY.maxFeeETH(); // @audit cache WISE_SECURITY
        data.baseRewardLiquidation = WISE_SECURITY.baseRewardLiquidation();
```
```diff
    function setRegistrationIsolationPool(
        uint256 _nftId,
        bool _registerState
    )
        external
    {
        _onlyIsolationPool(
            msg.sender
        );

        _validateZero(
            WISE_SECURITY.overallETHCollateralsBare(_nftId) // @audit WISE_SECURITY
        );

        _validateZero(
            WISE_SECURITY.overallETHBorrowBare(_nftId)
        );
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1273-L1274

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1397-L1401
```


#### `WISE_LENDING` should be cached
```file : WiseSecurity.sol```
```diff
        if (WISE_LENDING.verifiedIsolationPool(_caller) == true) { // @audit WISE_LENDING
            return true;
        }

        if (WISE_LENDING.positionLocked(_nftId) == true) {
            return true;
        }

        if (_isUncollateralized(_nftId, _poolToken) == true) {
            return true;
        }

        if (WISE_LENDING.getPositionBorrowTokenLength(_nftId) == 0) {
            return true;
        }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L255-L269
```

```diff
        if (WISE_LENDING.verifiedIsolationPool(_caller) == true) { 
            return true;
        }

        if (WISE_LENDING.positionLocked(_nftId) == true) {
            return true;
        }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L293-L297
```

#### `WISE_SECURITY` could be cached
```file : WiseLendingDeclaration.sol```
```diff
        if (address(WISE_SECURITY) > ZERO_ADDRESS) {
            revert InvalidAction();
        }

        WISE_SECURITY = IWiseSecurity(
            _wiseSecurity
        );

        FEE_MANAGER = IFeeManagerLight(
            WISE_SECURITY.FEE_MANAGER() // @audit G:: 2 state read can avoid here by using _wiseSecurity
        );

        AAVE_HUB_ADDRESS = WISE_SECURITY.AAVE_HUB();
    }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L146-L156
```

#### `POSITION_NFT` could be cached
```diff
uint256 nftId = POSITION_NFT.mintPosition(); 

            _registrationFarm(
                nftId
            );

            POSITION_NFT.approve(
                AAVE_HUB_ADDRESS,
                nftId
            );
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L191-L200
```

#### `incentiveOwnerA` & `incentiveOwnerB` could be cached
```file : FeeManager.sol```
One extra storage read saved.

```diff
        if (msg.sender != incentiveOwnerA) { // @audit G:: cache incentiveOwnerA
            revert NotAllowed();
        }

        if (_newOwner == incentiveOwnerA) { 
            revert NotAllowed();
        }

        if (_newOwner == incentiveOwnerB) {
            revert NotAllowed();
        }

        incentiveUSD[_newOwner] = incentiveUSD[
            incentiveOwnerA
        ];

        delete incentiveUSD[
            incentiveOwnerA
        ];

        incentiveOwnerA = _newOwner;
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L320-L340
```
```diff
if (msg.sender != incentiveOwnerB) {  // @audit chache incentiveOwnerB
            revert NotAllowed();
        }

        if (_newOwner == incentiveOwnerA) {
            revert NotAllowed();
        }

        if (_newOwner == incentiveOwnerB) {
            revert NotAllowed();
        }

        incentiveUSD[_newOwner] = incentiveUSD[
            incentiveOwnerB
        ];

        delete incentiveUSD[
            incentiveOwnerB
        ];

        incentiveOwnerB = _newOwner;

        emit IncentiveOwnerBChanged(
            _newOwner,
            block.timestamp
        );
    }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L357-L377
```

### [Gas-05] No need to craete a stackk variable, function result could directly used
```file : WiseLending.sol```
```diff
{
-       uint256 shareAmount = _handleDeposit( 
-           msg.sender,
-           _nftId,
-           _poolToken,
-           _amount
-       );

        _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        );

-       return shareAmount;
+       return  _handleDeposit( 
+           msg.sender,
+           _nftId,
+           _poolToken,
+           _amount
+       );
    }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L407-L418
```

#### no need to create `timeSinceUp` variable
```file: OracleHelper.sol```
```diff
-       uint256 timeSinceUp = block.timestamp - startedAt;

-       if (timeSinceUp <= GRACE_PEROID) {

+       if(block.timestamp - startedAt <= GRACE_PEROID)
            return true;
        }
```

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L541-L546
```

### [Gas-06] Some mapping results could be cached rather than recalling it

#### `curveSwapInfoData[_poolToken]` could be cached
```file : WiseSecurity.sol```
```diff
        address curvePool = curveSwapInfoData[_poolToken].curvePool; // @audit cache curveSwapInfoData[_poolToken]

        if (curvePool == ZERO_ADDRESS) {
            return;
        }

        (
            bool success
            ,
        ) = curvePool.call{value: 0} (
            curveSwapInfoData[_poolToken].swapBytesPool
        );

        _checkSuccess(
            success
        );

        address curveMeta = curveSwapInfoData[_poolToken].curveMetaPool;

        if (curveMeta == ZERO_ADDRESS) {
            return;
        }

        (
            success
            ,
        ) = curveMeta.call{value: 0} (
            curveSwapInfoData[_poolToken].swapBytesMeta
        );
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L199-L226
```

#### `incentiveUSD[_incentiveOwner]` could be cached
```file: FeeManagerHelper.sol```
```diff
 uint256 reduceUSD = usdEquivalent < incentiveUSD[_incentiveOwner]
            ? usdEquivalent
            : incentiveUSD[_incentiveOwner];
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L348-L350
```

#### `borrowPoolData[_poolToken]` & `lendingPoolData[_poolToken]` could be cached
```file : MainHelper.sol```
```diff
return _calculateShares(
            borrowPoolData[_poolToken].totalBorrowShares * _amount, 
            borrowPoolData[_poolToken].pseudoTotalBorrowAmount,
            _maxSharePrice
        );
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L65-L66

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L102-L103

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L123-L125

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L507-L510
```

### [Gas-07] Inside loop state variable called
#### `WISE_LENDING` & `WISE_ORACLE` should be cached outside of loop
```file : WiseSecurity.sol```
```diff
        while (i < len) {

            token = WISE_LENDING.getPositionLendingTokenByIndex(  //@audit  loop state variable call
                _nftId,
                i
            );

            amount = getPositionLendingAmount(
                _nftId,
                token
            );

            ethValue = WISE_ORACLE.getTokensInETH(
                token,
                amount
            );

            weightedRate += ethValue
                * getLendingRate(token);

            overallETH += ethValue;

            unchecked {
                ++i;
            }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L466

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L476

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L523

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L533

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L582

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L662

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L679

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L711

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L763
```

#### `WISE_LENDING` could cached outside the loop
```file : FeeManager.sol```
```diff
        while (i < l) {

            _checkValue(
                _newFees[i]
            );

            WISE_LENDING.setPoolFee(
                _poolTokens[i],
                _newFees[i]
            );
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L145-L154
```

#### `WISE_LENDING` could cached
```file : AaveHelper.sol```
```diff
for (i; i < l;) {

            address currentAddress = WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            );

            unchecked {
                ++i;
            }

            if (currentAddress == _poolToken) {
                continue;
            }

            WISE_LENDING.preparePool( // @audit
                currentAddress
            );

            WISE_LENDING.newBorrowRate(
                _poolToken
            );
        }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L254-L274
```

### [Gas-08] Some state variable could be `immutable`
#### `WISE_SECURITY`, `POSITION_NFT` & `AAVE_HUB_ADDRESS` could be immutable
```file : WiseLendingDeclaration.sol```
```diff

```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L162

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L174

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L180
```

### [Gas-09] Before starting loop both inputed arrays lengths same or not checked
```file : FeeManager.sol```
```diff
    function setAaveFlagBulk(
        address[] calldata _poolTokens,
        address[] calldata _underlyingTokens
    )
        external
        onlyMaster
    {
        uint256 i;
        uint256 l = _poolTokens.length; // @audit length same or not not checked

        while (i < l) {
            _setAaveFlag(
                _poolTokens[i],
                _underlyingTokens[i]
            );

            unchecked {
                ++i;
            }
        }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L90-L96

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L143-L154
```
### [Gas-10] Instead of reading state variable `proposedIncentiveMaster`, `msg.sender` could used
```file : FeeManager.sol```
One extra storage read saved.

```diff
    function claimOwnershipIncentiveMaster()
        external
    {
        if (msg.sender != proposedIncentiveMaster) {
            revert NotAllowed();
        }

-       incentiveMaster = proposedIncentiveMaster; 
+       incentiveMaster = msg.sender;

        proposedIncentiveMaster = ZERO_ADDRESS;
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L90-L96
```

### [Gas-11] No need to cache `msg.sender` into memory variable, rather using directly cost less gas
```file : FeeManager.sol```

```diff
-      address caller = msg.sender; 

        if (totalBadDebtETH > 0) {
            revert ExistingBadDebt();
        }

        if (allowedTokens[caller][_feeToken] == false) {
            revert NotAllowed();
        }

        _decreaseFeeTokens(
            _feeToken,
            _amount
        );

        _safeTransfer(
            _feeToken,
-           caller,
+           msg.sender,
            _amount
        );

        emit ClaimedFeesBeneficial(
-           caller,
+           msg.sender,
            _feeToken,
            _amount,
            block.timestamp
        );
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L695

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L712

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L717
```

### [Gas-12] Constant should be value not expression
#### `MAX_COLLATERAL_FACTOR` should be a value and following comment explains how this result calculated from an expression
``` File : PendlePowerFarmDeclaration.sol ```
```diff
 uint256 internal constant MAX_COLLATERAL_FACTOR = 95 * PRECISION_FACTOR_E16;
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L62
```


### [Gas-13] Corresponding variable checks should preform eairlier before writing to storage
#### `sendingProgressAaveHub` should writed after `success` variable check
If `success` condition true then whole Tx failed and gas wasted by writing to `sendingProgressAaveHub`
```file : AaveHelper.sol```
```diff
    function _sendValue(
        address _recipient,
        uint256 _amount
    )
        internal
    {
        if (address(this).balance < _amount) {
            revert InvalidValue();
        }

        sendingProgressAaveHub = true;

        (bool success, ) = payable(_recipient).call{
            value: _amount
        }("");

+       if (success == false) { 
+           revert FailedInnerCall();
+       }

        sendingProgressAaveHub = false;

-       if (success == false) { 
-           revert FailedInnerCall();
-       }
```
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L196-L216
```

