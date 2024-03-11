## Gas Optimization

## Summary

|No|Issue|Instance|
|--|-----|--------|
|[G-01]|+= costs more gas than = + for state variables|35|
|[G-02]|Use assembly for loops|4|
|[G-03]|Using mappings instead of arrays to avoid length checks save gas|50|
|[G-04]|Should use arguments instead of state variable |67|
|[G-05]|Unnecessary look up in if condition|1|
|[G-06]| Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)|67|
|[G-07]|Optimize Address Storage Value Management with assembly|28|
|[G-08]|Use assembly to emit events|67|
|[G-09]|Cache array length outside of loop|2|
|[G-10]|address(this) can be stored in state variable that will ultimately cost less|35|
|[G-11]| Call msg.sender directly instead of caching them|5|
|[G-12]|Use constants instead of type(uintx).max|4|
|[G-13]|Use function instead of modifiers |17|
|[G-14]|Modifiers are redundant if used only once or not used at all. |17|
|[G-15]|Default value initialization|10|
|[G-16]|Use calldata instead of memory for function arguments that do not get mutated|17|
|[G-17]|Use assembly for loops|4|
|[G-18]|Counting down in for statements is more gas efficient|1|
|[G-19]|Use assembly in place of abi.decode to extract calldata values more efficiently|2|
|[G-20]|Use selfbalance() instead of address(this).balance|2|
|[G-21]|Change public function visibility to external|52|
|[G-22]| Stack variable cost less while used in emiting event|67|
|[G-23]|Emitting constants wastes gas|67|

## [G-01] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: WiseSecurity/WiseSecurity.sol

481    weightedRate += ethValue
        * getLendingRate(token);

484    overallETH += ethValue;

538 weightedRate += ethValue
      * getBorrowRate(token);

541   overallETH += ethValue;

592 ethValueDebt += ethValue

614 totalETHSupply += ethValue;
615    ethValueGain += ethValue

679  buffer += WISE_LENDING.lendingPoolData(token).collateralFactor
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L481-L484
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L538-L541
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L592
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L614-L615
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L679

```solidity

file: MainHelper.sol

591  userLendingData[_nftId][_poolToken].shares += _shares;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L591

```solidity

file: /WiseSecurity/WiseSecurityHelper.sol

44    weightedTotal += amount

48  unweightedAmount += amount;

90  weightedTotal += WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor

128     amount += getFullCollateralETH

171  weightedTotal += WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor

214  ethCollateral += getETHCollateral(

267  ethCollateral += getETHCollateralUpdated(  

374      buffer += getETHBorrow(      

415  buffer += getETHBorrow(    

455 buffer += getETHBorrow(    

512  buffer += _getETHBorrowUpdated(    
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L44
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L48
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L90
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L128
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L171
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L214
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L267
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L374
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L415
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L455
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L512

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

343  underlyingLpAssetsCurrent += additonalAssets;

512 totalLpAssetsToDistribute += _amount;

577   underlyingLpAssetsCurrent += _underlyingLpAssetAmount;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L343
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L512
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L577

```solidity

file: FeeManager/FeeManager.sol

220  incentiveUSD[incentiveOwnerA] += _value;

238  incentiveUSD[incentiveOwnerB] += _value;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L220
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L238

```solidity

file: WiseLowLevelHelper.sol

230  globalPoolData[_poolToken].totalPool += _amount;

248  lendingPoolData[_poolToken].totalDepositShares += _amount;

266    borrowPoolData[_poolToken].pseudoTotalBorrowAmount += _amount;

284    borrowPoolData[_poolToken].totalBorrowShares += _amount;

302  lendingPoolData[_poolToken].pseudoTotalPool += _amount;

338  globalPoolData[_poolToken].totalBareToken += _amount;

390 userMapping[_nftId][_poolToken] += _amount;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L230
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L248
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L266
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L284
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L302
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L338
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L390

```solidity

file: FeeManager/FeeManagerHelper.sol

94   totalBadDebtETH += _amount;

201 feeTokens[_feeToken] += _amount;

304  reduceAmount += _gatherIncentives(

314   reduceAmount += _gatherIncentives(    

358  [_underlyingToken] += incentiveAmount;

374  [_underlyingToken] += incentiveAmount;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L94
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L201
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L304
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L314
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L358
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L374

## [G-02]  Use assembly for loops
In the following instances, assembly is used for more gas efficient loops.

```solidity

file: WrapperHub/AaveHub.sol

71  for (uint256 i = 0; i < _underlyingAssets.length; i++) 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L71

```solidity

file: WrapperHub/AaveHelper.sol

254   for (i; i < l;) 

290 for (i; i < l;)
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L254
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L290

```solidity

file: PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

140  for (i; i < ownerTokenCount;) 

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L140

## [G-03] Using mappings instead of arrays to avoid length checks save gas
Just by using a mapping, we get a gas saving of 2102 gas. When you read the value of an index of an array, solidity adds bytecode that checks that you are reading from a valid index 

```solidity

file: MainHelper.sol

397      returns (address[] memory)
    {
399      address[] memory tokens = _userTokenData[

419     address[] memory _tokens      

455 address[] memory _lendTokens,
456 address[] memory _borrowTokens
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L397-L-399 
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L419
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L455-L456

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

199   uint256[] memory rewardsOutsideArray = _calculateRewardsClaimedOutside();

220  returns (uint256[] memory)
    {
        address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(
            UNDERLYING_PENDLE_MARKET
        );

        uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
            UNDERLYING_PENDLE_MARKET
        );

        uint256 l = rewardTokens.length;
231        uint256[] memory rewardsOutsideArray = new uint256[](l);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L199
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L220-L231

```solidity

file: FeeManager/FeeManager.sol#

83   address[] calldata _poolTokens,
84  address[] calldata _underlyingTokens

136 address[] calldata _poolTokens,
137  uint256[] calldata _newFees

540    address[] calldata _feeTokens

573     address[] memory _feeTokens
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L83
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L136
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L540
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L573

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

27  IERC20[] memory tokens = new IERC20[](1);
28  uint256[] memory amount = new uint256[](1);

60   IERC20[] memory _flashloanToken,
        uint256[] memory _flashloanAmounts,
        uint256[] memory _feeAmounts,
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L27-L28
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L60-L63

```solidity

fil: WiseOracleHub/OracleHelper.sol

15   address[] calldata _underlyingFeedTokens

413   uint32[] memory secondsAgo = new uint32[](

421  int56[] memory tickCumulatives
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L15
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L413
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L421

```solidity

foile: owerFarms/PendlePowerFarmController/PendlePowerFarmController.sol


248  childInfo.lastIndex = new uint128[](

300   address[] memory rewardTokens = _getRewardTokens(

433   bytes32[] calldata _proof

499 uint256[] calldata _amounts
    
576   childInfo.reservedForCompound = new uint256[](    

583  bytes32[] calldata _merkleProof      

613       address[] calldata _pools,
614        uint64[] calldata _weights
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L248
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L252
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L252
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L300
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L433
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L499
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L576
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L583
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L613-L614

```solidity

file: WiseCore.sol

21 address[] memory,
23    address[] memory

55  address[] memory lendTokens,
56    address[] memory borrowTokens

360   address[] memory lendTokens,
361    address[] memory borrowTokens
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L21-L22
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L55-L56
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L360-L361

```solidity

file: WiseLendingDeclaration.sol

248  address[] lendTokens;
249   address[] borrowTokens;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L248-L249

```solidity

file: WiseOracleHub/WiseOracleHub.sol

58 underlyingFeedTokens[ETH_USD_PLACEHOLDER] = new address[](0);

362   address[] calldata _underlyingFeedTokens

380   address[] calldata _tokenAddresses,
        IPriceFeed[] calldata _priceFeedAddresses,
383      address[][] calldata _underlyingFeedTokens

502  address[] calldata _tokenAddresses
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L58
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L362
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L380-L383
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L502

```solidity

file: WrapperHub/AaveHub.sol

65 address[] calldata _underlyingAssets,
66   address[] calldata _aaveTokens

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L65

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

36 uint256[] reservedForCompound;
        uint128[] lastIndex;
38        address[] rewardTokens;

61  address[] public activePendleMarkets;

171   address[] _rewardTokens

190     bytes32[] _proof,

196   uint256[] _amounts

201   bytes32[] _merkleProof,
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L36-L38
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L61
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L171
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L190
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L196
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L201

```solidity

file: PositionNFTs.sol

276  returns (uint256[] memory)

292   uint256[] memory tokenIds = new uint256[](
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L276
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L292

```solidity

file: FeeManager/DeclarationsFeeManager.sol

123  address[] public poolTokenAddresses;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L123

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

10  address[] memory _array,

100  returns (uint256[] memory)

110   returns (uint128[] memory)

120   returns (address[] memory)

219   address[] memory rewardTokensToCompare

238    address[] memory _rewardTokens

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L10
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L100
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L110
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L120
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L219
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L238

```solidity

file: PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

134  uint256[] memory tokenIds = new uint256[](

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L134

```solidity

file: FeeManager/FeeManagerEvents.sol

97  address[] token,

103   address[] token,
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerEvents.sol#L97
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerEvents.sol#L103

## [G-04] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas   

```solidity

file: WiseSecurity/WiseSecurity.sol

1022 
        emit SecurityShutdown(
            msg.sender,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1022

```solidity

file: FeeManager/FeeManager.sol

124  emit PoolFeeChanged(
            _poolToken,
            _newFee,
            block.timestamp
        )

156  emit PoolFeeChanged(
                _poolTokens[i],
                _newFees[i],
                block.timestamp
            );

185  emit IncentiveMasterProposed(
            _proposedIncentiveMaster,
            block.timestamp
        );            

204         emit ClaimedOwnershipIncentiveMaster(
            incentiveMaster,
            block.timestamp
        );
        
222   emit IncentiveIncreasedA(
            _value,
            block.timestamp
        ); 

240  emit IncentiveIncreasedB(
            _value,
            block.timestamp
        );

276   emit ClaimedIncentivesBulk(
            block.timestamp
        );

297    emit ClaimedIncentives(
            msg.sender,
            _feeToken,
            amount,
            block.timestamp
        );

342   emit IncentiveOwnerAChanged(
            _newOwner,
            block.timestamp
        );

379  emit IncentiveOwnerBChanged(
            _newOwner,
            block.timestamp
        ); 

402  emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );

428  emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );     

478  emit PoolTokenRemoved(
                _poolToken,
                block.timestamp
            );

504    emit BadDebtIncreasedLiquidation(
            _amount,
            block.timestamp
        );     

526   emit SetBadDebtPosition(
            _nftId,
            _amount,
            block.timestamp
        );

560  emit SetBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );

593  emit RevokeBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );

619  emit ClaimedFeesWiseBulk(
            block.timestamp
        );

677  emit ClaimedFeesWise(
            underlyingTokenAddress,
            tokenAmount,
            block.timestamp
        );  

716   emit ClaimedFeesBeneficial(
            caller,
            _feeToken,
            _amount,
            block.timestamp
        );

801   emit PayedBackBadDebt(
            _nftId,
            msg.sender,
            _paybackToken,
            _receivingToken,
            paybackAmount,
            block.timestamp
        );

852   emit PayedBackBadDebtFree(
            _nftId,
            msg.sender,
            _paybackToken,
            paybackAmount,
            block.timestamp
        );        
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L124
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L156
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L185
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L204
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L222
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L240
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L276
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L297
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L342
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L379
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L402
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L428
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L478
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L504
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L526
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L560
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L593
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L619
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L677
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L716
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L801
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L852

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

46    emit WithdrawLp(
            _pendleMarket,
            _to,
            _amount
        );

103    emit ExchangeRewardsForCompounding(
            _pendleMarket,
            _rewardToken,
            _rewardAmount,
            sendingAmount
        );

159    emit ExchangeLpFeesForPendle(
            _pendleMarket,
            _pendleChildShares,
            tokenAmountSend,
            withdrawnAmount
        );

287    emit AddPendleMarket(
            _pendleMarket,
            pendleChild
        );  

313 
        emit UpdateRewardTokens(
            _pendleMarket,
            rewardTokens
        );  

329  emit ChangeExchangeIncentive(
            _newExchangeIncentive
        );

355   emit ChangeMintFee(
            _pendleMarket,
            _newFee
        );

421  emit LockPendle(
            _amount,
            expiry,
            newVeBalance,
            _fromInside,
            _sameExpiry,
            block.timestamp
        );

444    emit ClaimArb(
            _accrued,
            _proof,
            block.timestamp
        ); 

467 emit WithdrawLock(
                amount,
                block.timestamp
            );

491   emit WithdrawLock(
            amount,
            block.timestamp
        );

520  emit IncreaseReservedForCompound(
            _pendleMarket,
            _amounts
        ); 

593   emit ClaimVoteRewards(
            _amount,
            _merkleProof,
            block.timestamp
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L46
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L103
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L159
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L287
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L313
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L329
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L355
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L421
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L444
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L467
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L491
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L520
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L593

```solidity

file: contracts/WiseCore.sol

77  emit FundsWithdrawnOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

86   emit FundsWithdrawn(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

156  emit FundsDeposited(
            _caller,
            _nftId,
            _poolToken,
            _amount,
            shareAmount,
            block.timestamp
        );

398    emit FundsBorrowedOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

407      emit FundsBorrowed(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L77
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L86
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L156
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L398
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L407

```solidity

file: WrapperHub/AaveHub.sol

145   emit IsDepositAave(
            _nftId,
            block.timestamp
        );

190    emit IsDepositAave(
            _nftId,
            block.timestamp
        );

226   emit IsWithdrawAave(
            _nftId,
            block.timestamp
        ); 

269  emit IsWithdrawAave(
            _nftId,
            block.timestamp
        ); 

302   emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );

342  emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );

378  emit IsBorrowAave(
            _nftId,
            block.timestamp
        );

421   emit IsBorrowAave(
            _nftId,
            block.timestamp
        );

470  emit IsPaybackAave(
            _nftId,
            block.timestamp
        );

544  emit IsPaybackAave(
            _nftId,
            block.timestamp
        );

603  emit IsPaybackAave(
            _nftId,
            block.timestamp
        ); 

688  emit SetAaveTokenAddress(
            _underlyingAsset,
            _aaveToken,
            block.timestamp
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L145
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L190
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L226
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L269
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L302
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L342
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L378
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L421
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L470
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L544
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L603
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L688

```solidity

file: FeeManager/FeeManagerHelper.sol

96  emit TotalBadDebtIncreased(
            _amount,
            block.timestamp
        );

112   emit TotalBadDebtDecreased(
            _amount,
            block.timestamp
        );

161   emit UpdateBadDebtPosition(
                _nftId,
                0,
                block.timestamp
            );

183   emit UpdateBadDebtPosition(
                _nftId,
                newBadDebt,
                block.timestamp
            );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L96
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L112
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L161
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L183

```slolidity

file: PowerFarms/PendlePowerFarm/PendlePowerManager.sol

71  emit MinDepositChange(
            _newMinDeposit,
            block.timestamp
        );

90    emit FarmStatus(
            _state,
            block.timestamp
        );

131  emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            _amount,
            block.timestamp
        );

174  emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            msg.value,
            block.timestamp
        );

246  emit FarmExit(
            _keyId,
            wiseLendingNFT,
            _allowedSpread,
            block.timestamp
        );

266  emit ManualPaybackShares(
            _keyId,
            farmingKeys[_keyId],
         

295   emit ManualWithdrawShares(
            _keyId,
            wiseLendingNFT,
            _withdrawShares,
            block.timestamp
        );            
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L71
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L90
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L131
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L174
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L246
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L266
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L295

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

284    emit RegistrationFarm(
            _nftId,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L284

```solidity

file: PowerFarms/PowerFarmNFTs/MinterReserver.sol

169  emit ERC721Received(
            _operator,
            _from,
            _tokenId,
            _data
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L169

```solidity

file: OwnableMaster.sol

82  emit MasterProposed(
            msg.sender,
            _proposedOwner
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L82

## [G-05] Unnecessary look up in if condition

If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: WiseOracleHub/OracleHelper.sol

97 if (_answer > maxAnswer || _answer < minAnswer) 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L97

## [G‑06] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity

file: WiseSecurity/WiseSecurity.sol

1022 
        emit SecurityShutdown(
            msg.sender,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1022

```solidity

file: FeeManager/FeeManager.sol

124  emit PoolFeeChanged(
            _poolToken,
            _newFee,
            block.timestamp
        )

156  emit PoolFeeChanged(
                _poolTokens[i],
                _newFees[i],
                block.timestamp
            );

185  emit IncentiveMasterProposed(
            _proposedIncentiveMaster,
            block.timestamp
        );            

204         emit ClaimedOwnershipIncentiveMaster(
            incentiveMaster,
            block.timestamp
        );
        
222   emit IncentiveIncreasedA(
            _value,
            block.timestamp
        ); 

240  emit IncentiveIncreasedB(
            _value,
            block.timestamp
        );

276   emit ClaimedIncentivesBulk(
            block.timestamp
        );

297    emit ClaimedIncentives(
            msg.sender,
            _feeToken,
            amount,
            block.timestamp
        );

342   emit IncentiveOwnerAChanged(
            _newOwner,
            block.timestamp
        );

379  emit IncentiveOwnerBChanged(
            _newOwner,
            block.timestamp
        ); 

402  emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );

428  emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );     

478  emit PoolTokenRemoved(
                _poolToken,
                block.timestamp
            );

504    emit BadDebtIncreasedLiquidation(
            _amount,
            block.timestamp
        );     

526   emit SetBadDebtPosition(
            _nftId,
            _amount,
            block.timestamp
        );

560  emit SetBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );

593  emit RevokeBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );

619  emit ClaimedFeesWiseBulk(
            block.timestamp
        );

677  emit ClaimedFeesWise(
            underlyingTokenAddress,
            tokenAmount,
            block.timestamp
        );  

716   emit ClaimedFeesBeneficial(
            caller,
            _feeToken,
            _amount,
            block.timestamp
        );

801   emit PayedBackBadDebt(
            _nftId,
            msg.sender,
            _paybackToken,
            _receivingToken,
            paybackAmount,
            block.timestamp
        );

852   emit PayedBackBadDebtFree(
            _nftId,
            msg.sender,
            _paybackToken,
            paybackAmount,
            block.timestamp
        );        
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L124
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L156
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L185
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L204
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L222
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L240
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L276
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L297
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L342
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L379
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L402
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L428
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L478
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L504
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L526
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L560
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L593
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L619
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L677
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L716
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L801
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L852

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

46    emit WithdrawLp(
            _pendleMarket,
            _to,
            _amount
        );

103    emit ExchangeRewardsForCompounding(
            _pendleMarket,
            _rewardToken,
            _rewardAmount,
            sendingAmount
        );

159    emit ExchangeLpFeesForPendle(
            _pendleMarket,
            _pendleChildShares,
            tokenAmountSend,
            withdrawnAmount
        );

287    emit AddPendleMarket(
            _pendleMarket,
            pendleChild
        );  

313 
        emit UpdateRewardTokens(
            _pendleMarket,
            rewardTokens
        );  

329  emit ChangeExchangeIncentive(
            _newExchangeIncentive
        );

355   emit ChangeMintFee(
            _pendleMarket,
            _newFee
        );

421  emit LockPendle(
            _amount,
            expiry,
            newVeBalance,
            _fromInside,
            _sameExpiry,
            block.timestamp
        );

444    emit ClaimArb(
            _accrued,
            _proof,
            block.timestamp
        ); 

467 emit WithdrawLock(
                amount,
                block.timestamp
            );

491   emit WithdrawLock(
            amount,
            block.timestamp
        );

520  emit IncreaseReservedForCompound(
            _pendleMarket,
            _amounts
        ); 

593   emit ClaimVoteRewards(
            _amount,
            _merkleProof,
            block.timestamp
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L46
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L103
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L159
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L287
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L313
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L329
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L355
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L421
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L444
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L467
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L491
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L520
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L593

```solidity

file: contracts/WiseCore.sol

77  emit FundsWithdrawnOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

86   emit FundsWithdrawn(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

156  emit FundsDeposited(
            _caller,
            _nftId,
            _poolToken,
            _amount,
            shareAmount,
            block.timestamp
        );

398    emit FundsBorrowedOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

407      emit FundsBorrowed(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L77
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L86
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L156
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L398
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L407

```solidity

file: WrapperHub/AaveHub.sol

145   emit IsDepositAave(
            _nftId,
            block.timestamp
        );

190    emit IsDepositAave(
            _nftId,
            block.timestamp
        );

226   emit IsWithdrawAave(
            _nftId,
            block.timestamp
        ); 

269  emit IsWithdrawAave(
            _nftId,
            block.timestamp
        ); 

302   emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );

342  emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );

378  emit IsBorrowAave(
            _nftId,
            block.timestamp
        );

421   emit IsBorrowAave(
            _nftId,
            block.timestamp
        );

470  emit IsPaybackAave(
            _nftId,
            block.timestamp
        );

544  emit IsPaybackAave(
            _nftId,
            block.timestamp
        );

603  emit IsPaybackAave(
            _nftId,
            block.timestamp
        ); 

688  emit SetAaveTokenAddress(
            _underlyingAsset,
            _aaveToken,
            block.timestamp
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L145
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L190
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L226
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L269
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L302
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L342
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L378
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L421
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L470
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L544
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L603
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L688

```solidity

file: FeeManager/FeeManagerHelper.sol

96  emit TotalBadDebtIncreased(
            _amount,
            block.timestamp
        );

112   emit TotalBadDebtDecreased(
            _amount,
            block.timestamp
        );

161   emit UpdateBadDebtPosition(
                _nftId,
                0,
                block.timestamp
            );

183   emit UpdateBadDebtPosition(
                _nftId,
                newBadDebt,
                block.timestamp
            );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L96
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L112
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L161
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L183

```slolidity

file: PowerFarms/PendlePowerFarm/PendlePowerManager.sol

71  emit MinDepositChange(
            _newMinDeposit,
            block.timestamp
        );

90    emit FarmStatus(
            _state,
            block.timestamp
        );

131  emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            _amount,
            block.timestamp
        );

174  emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            msg.value,
            block.timestamp
        );

246  emit FarmExit(
            _keyId,
            wiseLendingNFT,
            _allowedSpread,
            block.timestamp
        );

266  emit ManualPaybackShares(
            _keyId,
            farmingKeys[_keyId],
         

295   emit ManualWithdrawShares(
            _keyId,
            wiseLendingNFT,
            _withdrawShares,
            block.timestamp
        );            
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L71
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L90
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L131
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L174
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L246
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L266
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L295

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

284    emit RegistrationFarm(
            _nftId,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L284

```solidity

file: PowerFarms/PowerFarmNFTs/MinterReserver.sol

169  emit ERC721Received(
            _operator,
            _from,
            _tokenId,
            _data
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L169

```solidity

file: OwnableMaster.sol

82  emit MasterProposed(
            msg.sender,
            _proposedOwner
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L82

## [G-07] Optimize Address Storage Value Management with assembly

```solidity

file: WiseSecurity/WiseSecurity.sol

35  if (msg.sender == address(WISE_LENDING)) 

143  address _poolToken,

194 address _poolToken,

239  address _caller,
240  address _poolToken

277   address _caller,
278   address _poolToken

308  address _caller,
309  address _poolToken

337  address _caller,
338  address _poolAddress

981  address _tokenAddress,
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L35
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L143
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L194
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L239-L240
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L277-L278 
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L308-L309
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L337-L338
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L981

```solidity

file: MainHelper.sol

218  address _poolAddress

279   address _tokenAddress

423   address currentAddress;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L218
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L279
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L423

```solidity

file: WiseSecurity/WiseSecurityHelper.sol

25  address tokenAddress;

72   address tokenAddress;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L25
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L72

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

683 address _underlyingPendleMarket,
684   address _pendleController,

716    address pendleSyAddress,
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L683-L684
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L716

```solidity

file: FeeManager/FeeManager.sol

28   address _master,
        address _aaveAddress,
        address _wiseLendingAddress,
        address _oracleHubAddress,
        address _wiseSecurityAddress,
        address _positionNFTAddress
    )
        DeclarationsFeeManager(
            _master,
            _aaveAddress,
            _wiseLendingAddress,
            _oracleHubAddress,
            _wiseSecurityAddress,
            _positionNFTAddress
42        )

174  address _proposedIncentiveMaster

259     tokenAddress = poolTokenAddresses[i];

467   poolTokenAddresses[i] = poolTokenAddresses[lastEntry];

633   address underlyingTokenAddress = _poolToken;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L28-42
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L174
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L259
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L467
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L633

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

30  address flashAsset = WETH_ADDRESS;


```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L30

```solidity

file: WiseOracleHub/OracleHelper.sol

23   priceFeed[_tokenAddress] = _priceFeedAddress;


```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L23

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

128  address pendleChild = pendleChildAddress[
            _pendleMarket

178 address childMarket = pendleChildAddress[
            _pendleMarket

341 address child = pendleChildAddress[
            _pendleMarket            
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L128
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L178
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L341

```solidity

file: WiseLendingDeclaration.sol

131 WETH_ADDRESS = WISE_ORACLE.WETH_ADDRESS();

162    address internal AAVE_HUB_ADDRESS;

165   address public immutable WETH_ADDRESS;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L131
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L162
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L165

### [G-08] Use assembly to emit event
We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

```solidity

file: WiseSecurity/WiseSecurity.sol

1022 
        emit SecurityShutdown(
            msg.sender,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1022

```solidity

file: FeeManager/FeeManager.sol

124  emit PoolFeeChanged(
            _poolToken,
            _newFee,
            block.timestamp
        )

156  emit PoolFeeChanged(
                _poolTokens[i],
                _newFees[i],
                block.timestamp
            );

185  emit IncentiveMasterProposed(
            _proposedIncentiveMaster,
            block.timestamp
        );            

204         emit ClaimedOwnershipIncentiveMaster(
            incentiveMaster,
            block.timestamp
        );
        
222   emit IncentiveIncreasedA(
            _value,
            block.timestamp
        ); 

240  emit IncentiveIncreasedB(
            _value,
            block.timestamp
        );

276   emit ClaimedIncentivesBulk(
            block.timestamp
        );

297    emit ClaimedIncentives(
            msg.sender,
            _feeToken,
            amount,
            block.timestamp
        );

342   emit IncentiveOwnerAChanged(
            _newOwner,
            block.timestamp
        );

379  emit IncentiveOwnerBChanged(
            _newOwner,
            block.timestamp
        ); 

402  emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );

428  emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );     

478  emit PoolTokenRemoved(
                _poolToken,
                block.timestamp
            );

504    emit BadDebtIncreasedLiquidation(
            _amount,
            block.timestamp
        );     

526   emit SetBadDebtPosition(
            _nftId,
            _amount,
            block.timestamp
        );

560  emit SetBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );

593  emit RevokeBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );

619  emit ClaimedFeesWiseBulk(
            block.timestamp
        );

677  emit ClaimedFeesWise(
            underlyingTokenAddress,
            tokenAmount,
            block.timestamp
        );  

716   emit ClaimedFeesBeneficial(
            caller,
            _feeToken,
            _amount,
            block.timestamp
        );

801   emit PayedBackBadDebt(
            _nftId,
            msg.sender,
            _paybackToken,
            _receivingToken,
            paybackAmount,
            block.timestamp
        );

852   emit PayedBackBadDebtFree(
            _nftId,
            msg.sender,
            _paybackToken,
            paybackAmount,
            block.timestamp
        );        
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L124
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L156
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L185
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L204
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L222
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L240
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L276
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L297
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L342
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L379
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L402
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L428
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L478
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L504
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L526
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L560
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L593
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L619
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L677
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L716
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L801
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L852

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

46    emit WithdrawLp(
            _pendleMarket,
            _to,
            _amount
        );

103    emit ExchangeRewardsForCompounding(
            _pendleMarket,
            _rewardToken,
            _rewardAmount,
            sendingAmount
        );

159    emit ExchangeLpFeesForPendle(
            _pendleMarket,
            _pendleChildShares,
            tokenAmountSend,
            withdrawnAmount
        );

287    emit AddPendleMarket(
            _pendleMarket,
            pendleChild
        );  

313 
        emit UpdateRewardTokens(
            _pendleMarket,
            rewardTokens
        );  

329  emit ChangeExchangeIncentive(
            _newExchangeIncentive
        );

355   emit ChangeMintFee(
            _pendleMarket,
            _newFee
        );

421  emit LockPendle(
            _amount,
            expiry,
            newVeBalance,
            _fromInside,
            _sameExpiry,
            block.timestamp
        );

444    emit ClaimArb(
            _accrued,
            _proof,
            block.timestamp
        ); 

467 emit WithdrawLock(
                amount,
                block.timestamp
            );

491   emit WithdrawLock(
            amount,
            block.timestamp
        );

520  emit IncreaseReservedForCompound(
            _pendleMarket,
            _amounts
        ); 

593   emit ClaimVoteRewards(
            _amount,
            _merkleProof,
            block.timestamp
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L46
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L103
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L159
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L287
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L313
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L329
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L355
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L421
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L444
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L467
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L491
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L520
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L593

```solidity

file: contracts/WiseCore.sol

77  emit FundsWithdrawnOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

86   emit FundsWithdrawn(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

156  emit FundsDeposited(
            _caller,
            _nftId,
            _poolToken,
            _amount,
            shareAmount,
            block.timestamp
        );

398    emit FundsBorrowedOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

407      emit FundsBorrowed(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L77
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L86
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L156
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L398
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L407

```solidity

file: WrapperHub/AaveHub.sol

145   emit IsDepositAave(
            _nftId,
            block.timestamp
        );

190    emit IsDepositAave(
            _nftId,
            block.timestamp
        );

226   emit IsWithdrawAave(
            _nftId,
            block.timestamp
        ); 

269  emit IsWithdrawAave(
            _nftId,
            block.timestamp
        ); 

302   emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );

342  emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );

378  emit IsBorrowAave(
            _nftId,
            block.timestamp
        );

421   emit IsBorrowAave(
            _nftId,
            block.timestamp
        );

470  emit IsPaybackAave(
            _nftId,
            block.timestamp
        );

544  emit IsPaybackAave(
            _nftId,
            block.timestamp
        );

603  emit IsPaybackAave(
            _nftId,
            block.timestamp
        ); 

688  emit SetAaveTokenAddress(
            _underlyingAsset,
            _aaveToken,
            block.timestamp
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L145
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L190
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L226
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L269
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L302
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L342
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L378
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L421
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L470
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L544
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L603
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L688

```solidity

file: FeeManager/FeeManagerHelper.sol

96  emit TotalBadDebtIncreased(
            _amount,
            block.timestamp
        );

112   emit TotalBadDebtDecreased(
            _amount,
            block.timestamp
        );

161   emit UpdateBadDebtPosition(
                _nftId,
                0,
                block.timestamp
            );

183   emit UpdateBadDebtPosition(
                _nftId,
                newBadDebt,
                block.timestamp
            );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L96
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L112
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L161
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L183

```slolidity

file: PowerFarms/PendlePowerFarm/PendlePowerManager.sol

71  emit MinDepositChange(
            _newMinDeposit,
            block.timestamp
        );

90    emit FarmStatus(
            _state,
            block.timestamp
        );

131  emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            _amount,
            block.timestamp
        );

174  emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            msg.value,
            block.timestamp
        );

246  emit FarmExit(
            _keyId,
            wiseLendingNFT,
            _allowedSpread,
            block.timestamp
        );

266  emit ManualPaybackShares(
            _keyId,
            farmingKeys[_keyId],
         

295   emit ManualWithdrawShares(
            _keyId,
            wiseLendingNFT,
            _withdrawShares,
            block.timestamp
        );            
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L71
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L90
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L131
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L174
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L246
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L266
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L295

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

284    emit RegistrationFarm(
            _nftId,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L284

```solidity

file: PowerFarms/PowerFarmNFTs/MinterReserver.sol

169  emit ERC721Received(
            _operator,
            _from,
            _tokenId,
            _data
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L169

```solidity

file: OwnableMaster.sol

82  emit MasterProposed(
            msg.sender,
            _proposedOwner
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L82

## [G-09] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

```solidity

file: WrapperHub/AaveHub.sol

71   for (uint256 i = 0; i < _underlyingAssets.length; i++) 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L71

```solidity

file: PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

140  for (i; i < ownerTokenCount;) 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L140


### [G-10] address(this) can be stored in state variable that will ultimately cost less
than every time calculating this specifically when it used multiple times in a contract

```solidity

file: MainHelper.sol

286  address(this)
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L286

```solidity

file: FeeManager/FeeManager.sol

659    address(this)
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L659

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.so

167  _receiver: address(this),

180  _receiver: address(this),

279   recipient: address(this),

444  _receiver: address(this),

469  _receiver: address(this),
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L167
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L180
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L279
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L444
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L469

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

28   address(this)

89   address(this),

145   address(this),

187   address(this)

265   address(this)

400  address(this)

540 address(this)

588  address(this),
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L28
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L89
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L145
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L187
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L265
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L400
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L540
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L588

```solidity

file: WiseCore.sol

677   address(this),
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L677

```solidity

file: WrapperHub/AaveHub.sol

134   address(this),

256   address(this),

329  address(this),

408 address(this),

454   address(this

461  address(this)

586    address(this),

593    address(this),

626  IERC20(tokenToSend).balanceOf(address(this))
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L134
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L256
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L329
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L408
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L454
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L461
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L586
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L593
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L626

```solidity

file: WrapperHub/AaveHelper.sol

67  address(this)

178   address(this)

189   address(this)
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L67
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L178
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L189

```solidity

file: PositionNFTs.sol

41   address(this)

84   address(this),
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L41
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L84

```solidity

file: WiseSecurity/WiseSecurityDeclarations.sol

86   address(this),
            positionNFTAddress
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L86

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerManager.sol

112   address(this),
            _amount
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L112

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

50 return _getPositionData(address(this)).expiry;

58  return _getPositionData(address(this)).amount;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L50
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L58

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

139  address(this),
    paybackAmount
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L139

## [G-11] Call msg.sender directly instead of caching them

In the instance below, instead of caching `msg.sender` and incurring unnecessary stack manipulation, we can call `msg.sender` directly.

```solidity

file: WiseSecurity/WiseSecurity.sol

1022 
        emit SecurityShutdown(
            msg.sender,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1022

```solidity

297    emit ClaimedIncentives(
            msg.sender,
            _feeToken,
            amount,
            block.timestamp
        );



801   emit PayedBackBadDebt(
            _nftId,
            msg.sender,
            _paybackToken,
            _receivingToken,
            paybackAmount,
            block.timestamp
        );

852   emit PayedBackBadDebtFree(
            _nftId,
            msg.sender,
            _paybackToken,
            paybackAmount,
            block.timestamp
        );        
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L297
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L801
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L852

```solidity

file: OwnableMaster.sol

82  emit MasterProposed(
            msg.sender,
            _proposedOwner
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L82

## [G-12] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity

file: MainHelper.sol

918 if (algorithmData[_poolToken].maxValue <= totalShares)
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L918

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

98  uint256 internal constant MAX_AMOUNT = type(uint256).max;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L98

```solidity

file: WiseSecurity/WiseSecurityDeclarations.sol

209  uint256 internal constant UINT256_MAX = type(uint256).max;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L209

```solidity

file: WrapperHub/Declarations.sol

39 uint256 internal constant MAX_AMOUNT = type(uint256).max;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L39

## [G-13] Use function instead of modifiers 

```solidity

file: WiseSecurity/WiseSecurity.sol

26   modifier onlyWiseLending() 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L26

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

67  modifier onlyController()

81   modifier syncSupply()
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L67
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L81

```solidity

file: WiseLowLevelHelper.sol

9   modifier onlyFeeManager()
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L9

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

356 modifier isActive()
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L356

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

210   modifier syncSupply(

220  modifier onlyChildContract(
        address _pendleMarket
    )

230  modifier onlyArbitrum()    
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L210
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L220
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L230

```solidity

file: WrapperHub/AaveHelper.sol

9   modifier nonReentrant() {
        _nonReentrantCheck();
        _;
    }

14    modifier validToken(
        address _poolToken
    ) 

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L9-L14

```solidity

file: PositionNFTs.sol

45  modifier onlyReserveRole() 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L45

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

9  modifier updatePools()
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L9

```solidity

file: FeeManager/DeclarationsFeeManager.sol

176  modifier onlyWiseSecurity() {
        _onlyWiseSecurity();
        _;
    }

    modifier onlyWiseLending() {
        _onlyWiseLending();
        _;
    }

186  modifier onlyIncentiveMaster() {
        _onlyIncentiveMaster();



```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L176-L186

```solidity

file: PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

30    modifier onlyFarmContract()
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L30

```solidity

file: PowerFarms/PowerFarmNFTs/MinterReserver.sol

32    modifier onlyKeyOwner(
        uint256 _keyId
    ) 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L32

```solidity

file: OwnableMaster.sol

16   modifier onlyProposed() 

32  modifier onlyMaster() 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L16
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L32

```solidity

file: DerivativeOracles/CustomOracleSetup.sol

14    modifier onlyOwner() 

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L14

## [G-14] Modifiers are redundant if used only once or not used at all. 

```solidity

file: WiseSecurity/WiseSecurity.sol

26   modifier onlyWiseLending() 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L26

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

67  modifier onlyController()

81   modifier syncSupply()
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L67
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L81

```solidity

file: WiseLowLevelHelper.sol

9   modifier onlyFeeManager()
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L9

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

356 modifier isActive()
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L356

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

210   modifier syncSupply(

220  modifier onlyChildContract(
        address _pendleMarket
    )

230  modifier onlyArbitrum()    
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L210
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L220
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L230

```solidity

file: WrapperHub/AaveHelper.sol

9   modifier nonReentrant() {
        _nonReentrantCheck();
        _;
    }

14    modifier validToken(
        address _poolToken
    ) 

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L9-L14

```solidity

file: PositionNFTs.sol

45  modifier onlyReserveRole() 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L45

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

9  modifier updatePools()
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L9

```solidity

file: FeeManager/DeclarationsFeeManager.sol

176  modifier onlyWiseSecurity() {
        _onlyWiseSecurity();
        _;
    }

    modifier onlyWiseLending() {
        _onlyWiseLending();
        _;
    }

186  modifier onlyIncentiveMaster() {
        _onlyIncentiveMaster();



```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L176-L186

```solidity

file: PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

30    modifier onlyFarmContract()
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L30

```solidity

file: PowerFarms/PowerFarmNFTs/MinterReserver.sol

32    modifier onlyKeyOwner(
        uint256 _keyId
    ) 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L32

```solidity

file: OwnableMaster.sol

16   modifier onlyProposed() 

32  modifier onlyMaster() 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L16
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L32

```solidity

file: DerivativeOracles/CustomOracleSetup.sol

14    modifier onlyOwner() 

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L14

## [G-15] Default value initialization
Problem
If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type).
Explicitly initializing it with its default value is an anti-pattern and wastes gas.
Instances include:

```solidity

file: WiseSecurity/WiseSecurity.sol

600    i = 0;

732         i = 0;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L600
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L732

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

256    rewardsOutsideArray[i] = 0;

267  rewardsOutsideArray[i] = 0;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L256
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L267

```solidity

file: WiseOracleHub/OracleHelper.sol

418    secondsAgo[1] = 0;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L418

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

268  childInfo.reservedForCompound[i] = 0;

459  reservedPendleForLocking = 0;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L268
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L459

```solidity

file: WrapperHub/AaveHub.sol

71 for (uint256 i = 0; i < _underlyingAssets.length; i++) 

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L71

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerManager.sol

228     reservedKeys[msg.sender] = 0;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L228

```solidity

file: WrapperHub/Declarations.sol

27   uint16 internal constant REF_CODE = 0;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L27

## [G-16] Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity

file: MainHelper.sol

417  function _prepareTokens(
        address _poolToken,
        address[] memory _tokens
    )

454    function _curveSecurityChecks(
        address[] memory _lendTokens,
        address[] memory _borrowTokens
    )

473   function _whileLoopCurveSecurity(
        address[] memory _tokens
    )    
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L417
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L454
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L473

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

682  function initialize(
        address _underlyingPendleMarket,
        address _pendleController,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L682

```solidity

file: FeeManager.sol

571   function revokeBeneficial(
        address _user,
        address[] memory _feeTokens
    )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L571

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.so

59 function receiveFlashLoan(
        IERC20[] memory _flashloanToken,
        uint256[] memory _flashloanAmounts,
        uint256[] memory _feeAmounts,
        bytes memory _userData
    )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L59

```solidity

file: WiseOracleHub/OracleHelper.sol

460   function _getTwapDerivatePrice(
        address _tokenAddress,
        UniTwapPoolInfo memory _uniTwapPoolInfo
    )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L460

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

211 
    function addPendleMarket(
        address _pendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )


```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L211

```solidity

file: WiseCore.sol

625  CoreLiquidationStruct memory _data

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L625

```solidity

file: PositionNFTs.sol

319   function setBaseURI(
        string memory _newBaseURI
    )

328   function setBaseExtension(
        string memory _newBaseExtension
    )
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L319
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L328

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

9 
    function _findIndex(
        address[] memory _array,
        address _value
    )

217   function _compareHashes(
        address _pendleMarket,
        address[] memory rewardTokensToCompare
    )    

236  function _setRewardTokens(
        address _pendleMarket,
        address[] memory _rewardTokens
    )
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L9
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L217
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L236

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

35  function deploy(
        address _underlyingPendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )

56  function _clone(
        address _underlyingPendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L35
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L56

```solidity

file: TransferHub/CallOptionalReturn.sol

12   function _callOptionalReturn(
        address token,
        bytes memory data
    )
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L12

## [G-17] Use assembly for loops
In the following instances, assembly is used for more gas efficient loops.

```solidity

file: WrapperHub/AaveHub.sol

71   for (uint256 i = 0; i < _underlyingAssets.length; i++)

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L71

```solidity

file: WrapperHub/AaveHelper.sol

254 for (i; i < l;)

290  for (i; i < l;) 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L254
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L290

```solidity

file: PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

140  for (i; i < ownerTokenCount;)

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L140

## [G‑18] Counting down in for statements is more gas efficient
Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

```solidity

file: WrapperHub/AaveHub.sol

71   for (uint256 i = 0; i < _underlyingAssets.length; i++)

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L71

## [G-19] Use assembly in place of abi.decode to extract calldata values more efficiently
Using inline assembly to extract calldata values can be more gas-efficient than using abi.decode in Solidity. Inline assembly gives more direct access to EVM operations, enabling optimized usage of calldata. However, assembly should be used judiciously as it's more prone to errors. Opt for this approach when performance is critical and the complexity it introduces is manageable.

```solidity


file: PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.so

93  abi.decode
            
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L93

```solidity


file: TransferHub/CallOptionalReturn.sol

26  bool results = returndata.length == 0 || abi.decode(
            returndata,
            (bool)
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L26

## [G-20] Use selfbalance() instead of address(this).balance

```solidity

file: WrapperHub/AaveHelper.sol

202  if (address(this).balance < _amount) 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L202

```solidity

file: TransferHub/SendValueHelper.sol

18   if (address(this).balance < _amount)
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L18

## [G-21] Change public function visibility to external
Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

```solidity

file: MainHelper.sol

17   function calculateLendingShares(
        address _poolToken,
        uint256 _amount,
        bool _maxSharePrice
    )
        public
        view

55   function calculateBorrowShares(
        address _poolToken,
        uint256 _amount,
        bool _maxSharePrice
    )
        public
        view

114   function paybackAmount(
        address _poolToken,
        uint256 _shares
    )
        public
        view       
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L17
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L55
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L114

```solidity

file: WiseSecurity/WiseSecurityHelper.sol

15 function overallETHCollateralsBoth(
        uint256 _nftId
    )
        public
        view

65  function overallETHCollateralsWeighted(
        uint256 _nftId
    )
        public
        view     

107  function overallETHCollateralsBare(
        uint256 _nftId
    )
        public
        view 

190  function getFullCollateralETH(
        uint256 _nftId,
        address _poolToken
    )
        public
        view        

281  function getETHCollateralUpdated(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        public
        view     

323   function getETHCollateral(
        uint256 _nftId,
        address _poolToken
    )
        public
        view  

360  function overallETHBorrowBare(
        uint256 _nftId
    )
        public
        view

394   function overallETHBorrowHeartbeat(
        uint256 _nftId
    )
        public
        view 

430    function overallETHBorrow(
        uint256 _nftId
    )
        public
        view

651   function getETHBorrow(
        uint256 _nftId,
        address _poolToken
    )
        public
        view 

672  function checkTokenAllowed(
        address _poolAddress
    )
        public
        view

687  function checkHeartbeat(
        address _poolToken
    )
        public
        view

837  function checksRegister(
        uint256 _nftId,
        address _caller
    )
        public
        view    

859   function canLiquidate(
        uint256 _borrowETHTotal,
        uint256 _weightedCollateralETH
    )
        public
        pure

876 function checkMaxShares(
        uint256 _nftId,
        address _tokenToPayback,
        uint256 _borrowETHTotal,
        uint256 _unweightedCollateralETH,
        uint256 _shareAmountToPay
    )
        public
        view

906  function checkBadDebtThreshold(
        uint256 _borrowETHTotal,
        uint256 _unweightedCollateral
    )
        public
        pure

922 function getPositionLendingAmount(
        uint256 _nftId,
        address _poolToken
    )
        public
        view

945  function getPositionBorrowAmount(
        uint256 _nftId,
        address _poolToken
    )
        public
        view

966   function checkOwnerPosition(
        uint256 _nftId,
        address _caller
    )
        public
        view

985  function getBorrowRate(
        address _poolToken
    )
        public
        view  

999  function getLendingRate(
        address _poolToken
    )
        public
        view
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L15
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L65
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L107
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L190
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L281
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L323
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L360
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L394
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L430
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L651
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L672
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L687
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L837
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L859
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L876
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L906
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L922
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L945
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L966
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L985
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L999

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

386 function previewDistribution()
        public
        view

434   function totalLpAssets()
        public
        view

443  function previewUnderlyingLpAssets()
        public
        view  

452  function previewMintShares(
        uint256 _underlyingAssetAmount,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view

465  function previewAmountWithdrawShares(
        uint256 _shares,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view

478     function previewBurnShares(
        uint256 _underlyingAssetAmount,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view                            
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L386
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L434
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L443
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L452
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L465
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L478

```solidity

file: FeeManager/FeeManager.sol

284   function claimIncentives(
        address _feeToken
    )
        public

628   function claimWiseFees(
        address _poolToken
    )
        public

872  function getPoolTokenAddressesLength()
        public
        view
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L284
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L628
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L872

```solidity

file: WiseOracleHub/OracleHelper.sol

106  function latestResolverTwap(
        address _tokenAddress
    )
        public
        view

265   function getETHPriceInUSD()
        public
        view

450  function decimals(
        address _tokenAddress
    )
        public
        view

521  function sequencerIsDead()
        public
        view

699   function getLatestRoundId(
        address _tokenAddress
    )
        public
        view
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L106
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L265
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L450
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L521
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L699

```solidity

file: WiseOracleHub/WiseOracleHub.sol

69   function latestResolver(
        address _tokenAddress
    )
        public
        view

143     function getTokensInETH(
        address _tokenAddress,
        uint256 _tokenAmount
    )
        public
        view

327  function getTokensFromETH(
        address _tokenAddress,
        uint256 _ethAmount
    )
        public
        view

439  function chainLinkIsDead(
        address _tokenAddress
    )
        public
        view
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L69
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L143
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L327
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L439

```solidity

file: WrapperHub/AaveHub.sol

122 function depositExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _amount
    )
        public

172   function depositExactAmountETH(
        uint256 _nftId
    )
        public
        payable
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L122
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L172

```solidity

file: WiseLowLevelHelper.sol

39  function getTotalPool(
        address _poolToken
    )
        public
        view

49  function getPseudoTotalPool(
        address _poolToken
    )
        public
        view

144    function getPositionLendingTokenByIndex(
        uint256 _nftId,
        uint256 _index
    )
        public
        view

155   function getPositionLendingTokenLength(
        uint256 _nftId
    )
        public
        view 

165  function getPositionBorrowTokenByIndex(
        uint256 _nftId,
        uint256 _index
    )
        public
        view 

176     function getPositionBorrowTokenLength(
        uint256 _nftId
    )
        public
        view

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L39
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L49
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L144
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L155
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L165
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L176

```solidity

file: WrapperHub/AaveHelper.sol

file: WrapperHub/AaveHelper.sol

315   function getAavePoolAPY(
        address _underlyingAsset
    )
        public
        view
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L315

```solidity

file: DerivativeOracles/PendleChildLpOracle.sol

35  function latestAnswer()
        public
        view
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L35

## [G-22] Stack variable cost less while used in emiting event
Even if the variable is going to be used only one time, caching a state variable and use its cache in an emit would help you reduce the cost by at least

```solidity

file: WiseSecurity/WiseSecurity.sol

1022 
        emit SecurityShutdown(
            msg.sender,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1022

```solidity

file: FeeManager/FeeManager.sol

124  emit PoolFeeChanged(
            _poolToken,
            _newFee,
            block.timestamp
        )

156  emit PoolFeeChanged(
                _poolTokens[i],
                _newFees[i],
                block.timestamp
            );

185  emit IncentiveMasterProposed(
            _proposedIncentiveMaster,
            block.timestamp
        );            

204         emit ClaimedOwnershipIncentiveMaster(
            incentiveMaster,
            block.timestamp
        );
        
222   emit IncentiveIncreasedA(
            _value,
            block.timestamp
        ); 

240  emit IncentiveIncreasedB(
            _value,
            block.timestamp
        );

276   emit ClaimedIncentivesBulk(
            block.timestamp
        );

297    emit ClaimedIncentives(
            msg.sender,
            _feeToken,
            amount,
            block.timestamp
        );

342   emit IncentiveOwnerAChanged(
            _newOwner,
            block.timestamp
        );

379  emit IncentiveOwnerBChanged(
            _newOwner,
            block.timestamp
        ); 

402  emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );

428  emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );     

478  emit PoolTokenRemoved(
                _poolToken,
                block.timestamp
            );

504    emit BadDebtIncreasedLiquidation(
            _amount,
            block.timestamp
        );     

526   emit SetBadDebtPosition(
            _nftId,
            _amount,
            block.timestamp
        );

560  emit SetBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );

593  emit RevokeBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );

619  emit ClaimedFeesWiseBulk(
            block.timestamp
        );

677  emit ClaimedFeesWise(
            underlyingTokenAddress,
            tokenAmount,
            block.timestamp
        );  

716   emit ClaimedFeesBeneficial(
            caller,
            _feeToken,
            _amount,
            block.timestamp
        );

801   emit PayedBackBadDebt(
            _nftId,
            msg.sender,
            _paybackToken,
            _receivingToken,
            paybackAmount,
            block.timestamp
        );

852   emit PayedBackBadDebtFree(
            _nftId,
            msg.sender,
            _paybackToken,
            paybackAmount,
            block.timestamp
        );        
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L124
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L156
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L185
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L204
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L222
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L240
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L276
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L297
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L342
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L379
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L402
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L428
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L478
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L504
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L526
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L560
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L593
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L619
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L677
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L716
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L801
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L852

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

46    emit WithdrawLp(
            _pendleMarket,
            _to,
            _amount
        );

103    emit ExchangeRewardsForCompounding(
            _pendleMarket,
            _rewardToken,
            _rewardAmount,
            sendingAmount
        );

159    emit ExchangeLpFeesForPendle(
            _pendleMarket,
            _pendleChildShares,
            tokenAmountSend,
            withdrawnAmount
        );

287    emit AddPendleMarket(
            _pendleMarket,
            pendleChild
        );  

313 
        emit UpdateRewardTokens(
            _pendleMarket,
            rewardTokens
        );  

329  emit ChangeExchangeIncentive(
            _newExchangeIncentive
        );

355   emit ChangeMintFee(
            _pendleMarket,
            _newFee
        );

421  emit LockPendle(
            _amount,
            expiry,
            newVeBalance,
            _fromInside,
            _sameExpiry,
            block.timestamp
        );

444    emit ClaimArb(
            _accrued,
            _proof,
            block.timestamp
        ); 

467 emit WithdrawLock(
                amount,
                block.timestamp
            );

491   emit WithdrawLock(
            amount,
            block.timestamp
        );

520  emit IncreaseReservedForCompound(
            _pendleMarket,
            _amounts
        ); 

593   emit ClaimVoteRewards(
            _amount,
            _merkleProof,
            block.timestamp
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L46
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L103
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L159
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L287
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L313
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L329
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L355
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L421
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L444
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L467
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L491
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L520
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L593

```solidity

file: contracts/WiseCore.sol

77  emit FundsWithdrawnOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

86   emit FundsWithdrawn(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

156  emit FundsDeposited(
            _caller,
            _nftId,
            _poolToken,
            _amount,
            shareAmount,
            block.timestamp
        );

398    emit FundsBorrowedOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

407      emit FundsBorrowed(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L77
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L86
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L156
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L398
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L407

```solidity

file: WrapperHub/AaveHub.sol

145   emit IsDepositAave(
            _nftId,
            block.timestamp
        );

190    emit IsDepositAave(
            _nftId,
            block.timestamp
        );

226   emit IsWithdrawAave(
            _nftId,
            block.timestamp
        ); 

269  emit IsWithdrawAave(
            _nftId,
            block.timestamp
        ); 

302   emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );

342  emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );

378  emit IsBorrowAave(
            _nftId,
            block.timestamp
        );

421   emit IsBorrowAave(
            _nftId,
            block.timestamp
        );

470  emit IsPaybackAave(
            _nftId,
            block.timestamp
        );

544  emit IsPaybackAave(
            _nftId,
            block.timestamp
        );

603  emit IsPaybackAave(
            _nftId,
            block.timestamp
        ); 

688  emit SetAaveTokenAddress(
            _underlyingAsset,
            _aaveToken,
            block.timestamp
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L145
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L190
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L226
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L269
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L302
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L342
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L378
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L421
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L470
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L544
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L603
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L688

```solidity

file: FeeManager/FeeManagerHelper.sol

96  emit TotalBadDebtIncreased(
            _amount,
            block.timestamp
        );

112   emit TotalBadDebtDecreased(
            _amount,
            block.timestamp
        );

161   emit UpdateBadDebtPosition(
                _nftId,
                0,
                block.timestamp
            );

183   emit UpdateBadDebtPosition(
                _nftId,
                newBadDebt,
                block.timestamp
            );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L96
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L112
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L161
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L183

```slolidity

file: PowerFarms/PendlePowerFarm/PendlePowerManager.sol

71  emit MinDepositChange(
            _newMinDeposit,
            block.timestamp
        );

90    emit FarmStatus(
            _state,
            block.timestamp
        );

131  emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            _amount,
            block.timestamp
        );

174  emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            msg.value,
            block.timestamp
        );

246  emit FarmExit(
            _keyId,
            wiseLendingNFT,
            _allowedSpread,
            block.timestamp
        );

266  emit ManualPaybackShares(
            _keyId,
            farmingKeys[_keyId],
         

295   emit ManualWithdrawShares(
            _keyId,
            wiseLendingNFT,
            _withdrawShares,
            block.timestamp
        );            
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L71
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L90
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L131
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L174
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L246
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L266
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L295

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

284    emit RegistrationFarm(
            _nftId,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L284

```solidity

file: PowerFarms/PowerFarmNFTs/MinterReserver.sol

169  emit ERC721Received(
            _operator,
            _from,
            _tokenId,
            _data
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L169

```solidity

file: OwnableMaster.sol

82  emit MasterProposed(
            msg.sender,
            _proposedOwner
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L82

## [G-23] Emitting constants wastes gas
Every event parameter costs Glogdata (8 gas) per byte. You can avoid this extra cost, in cases where you're emitting a constant, by creating a new version of the event which doesn't have the parameter (and have users look to the contract's variables for its value instead). Alternatively, in the case of boolean constants, two events can be created - one representing the true case and one representing the false case.

```solidity

file: WiseSecurity/WiseSecurity.sol

1022 
        emit SecurityShutdown(
            msg.sender,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1022

```solidity

file: FeeManager/FeeManager.sol

124  emit PoolFeeChanged(
            _poolToken,
            _newFee,
            block.timestamp
        )

156  emit PoolFeeChanged(
                _poolTokens[i],
                _newFees[i],
                block.timestamp
            );

185  emit IncentiveMasterProposed(
            _proposedIncentiveMaster,
            block.timestamp
        );            

204         emit ClaimedOwnershipIncentiveMaster(
            incentiveMaster,
            block.timestamp
        );
        
222   emit IncentiveIncreasedA(
            _value,
            block.timestamp
        ); 

240  emit IncentiveIncreasedB(
            _value,
            block.timestamp
        );

276   emit ClaimedIncentivesBulk(
            block.timestamp
        );

297    emit ClaimedIncentives(
            msg.sender,
            _feeToken,
            amount,
            block.timestamp
        );

342   emit IncentiveOwnerAChanged(
            _newOwner,
            block.timestamp
        );

379  emit IncentiveOwnerBChanged(
            _newOwner,
            block.timestamp
        ); 

402  emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );

428  emit PoolTokenAdded(
            _poolToken,
            block.timestamp
        );     

478  emit PoolTokenRemoved(
                _poolToken,
                block.timestamp
            );

504    emit BadDebtIncreasedLiquidation(
            _amount,
            block.timestamp
        );     

526   emit SetBadDebtPosition(
            _nftId,
            _amount,
            block.timestamp
        );

560  emit SetBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );

593  emit RevokeBeneficial(
            _user,
            _feeTokens,
            block.timestamp
        );

619  emit ClaimedFeesWiseBulk(
            block.timestamp
        );

677  emit ClaimedFeesWise(
            underlyingTokenAddress,
            tokenAmount,
            block.timestamp
        );  

716   emit ClaimedFeesBeneficial(
            caller,
            _feeToken,
            _amount,
            block.timestamp
        );

801   emit PayedBackBadDebt(
            _nftId,
            msg.sender,
            _paybackToken,
            _receivingToken,
            paybackAmount,
            block.timestamp
        );

852   emit PayedBackBadDebtFree(
            _nftId,
            msg.sender,
            _paybackToken,
            paybackAmount,
            block.timestamp
        );        
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L124
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L156
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L185
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L204
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L222
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L240
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L276
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L297
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L342
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L379
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L402
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L428
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L478
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L504
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L526
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L560
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L593
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L619
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L677
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L716
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L801
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L852

```solidity

file: PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

46    emit WithdrawLp(
            _pendleMarket,
            _to,
            _amount
        );

103    emit ExchangeRewardsForCompounding(
            _pendleMarket,
            _rewardToken,
            _rewardAmount,
            sendingAmount
        );

159    emit ExchangeLpFeesForPendle(
            _pendleMarket,
            _pendleChildShares,
            tokenAmountSend,
            withdrawnAmount
        );

287    emit AddPendleMarket(
            _pendleMarket,
            pendleChild
        );  

313 
        emit UpdateRewardTokens(
            _pendleMarket,
            rewardTokens
        );  

329  emit ChangeExchangeIncentive(
            _newExchangeIncentive
        );

355   emit ChangeMintFee(
            _pendleMarket,
            _newFee
        );

421  emit LockPendle(
            _amount,
            expiry,
            newVeBalance,
            _fromInside,
            _sameExpiry,
            block.timestamp
        );

444    emit ClaimArb(
            _accrued,
            _proof,
            block.timestamp
        ); 

467 emit WithdrawLock(
                amount,
                block.timestamp
            );

491   emit WithdrawLock(
            amount,
            block.timestamp
        );

520  emit IncreaseReservedForCompound(
            _pendleMarket,
            _amounts
        ); 

593   emit ClaimVoteRewards(
            _amount,
            _merkleProof,
            block.timestamp
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L46
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L103
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L159
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L287
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L313
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L329
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L355
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L421
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L444
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L467
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L491
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L520
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L593

```solidity

file: contracts/WiseCore.sol

77  emit FundsWithdrawnOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

86   emit FundsWithdrawn(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

156  emit FundsDeposited(
            _caller,
            _nftId,
            _poolToken,
            _amount,
            shareAmount,
            block.timestamp
        );

398    emit FundsBorrowedOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );

407      emit FundsBorrowed(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L77
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L86
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L156
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L398
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L407

```solidity

file: WrapperHub/AaveHub.sol

145   emit IsDepositAave(
            _nftId,
            block.timestamp
        );

190    emit IsDepositAave(
            _nftId,
            block.timestamp
        );

226   emit IsWithdrawAave(
            _nftId,
            block.timestamp
        ); 

269  emit IsWithdrawAave(
            _nftId,
            block.timestamp
        ); 

302   emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );

342  emit IsWithdrawAave(
            _nftId,
            block.timestamp
        );

378  emit IsBorrowAave(
            _nftId,
            block.timestamp
        );

421   emit IsBorrowAave(
            _nftId,
            block.timestamp
        );

470  emit IsPaybackAave(
            _nftId,
            block.timestamp
        );

544  emit IsPaybackAave(
            _nftId,
            block.timestamp
        );

603  emit IsPaybackAave(
            _nftId,
            block.timestamp
        ); 

688  emit SetAaveTokenAddress(
            _underlyingAsset,
            _aaveToken,
            block.timestamp
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L145
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L190
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L226
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L269
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L302
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L342
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L378
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L421
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L470
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L544
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L603
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L688

```solidity

file: FeeManager/FeeManagerHelper.sol

96  emit TotalBadDebtIncreased(
            _amount,
            block.timestamp
        );

112   emit TotalBadDebtDecreased(
            _amount,
            block.timestamp
        );

161   emit UpdateBadDebtPosition(
                _nftId,
                0,
                block.timestamp
            );

183   emit UpdateBadDebtPosition(
                _nftId,
                newBadDebt,
                block.timestamp
            );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L96
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L112
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L161
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L183

```slolidity

file: PowerFarms/PendlePowerFarm/PendlePowerManager.sol

71  emit MinDepositChange(
            _newMinDeposit,
            block.timestamp
        );

90    emit FarmStatus(
            _state,
            block.timestamp
        );

131  emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            _amount,
            block.timestamp
        );

174  emit FarmEntry(
            keyId,
            wiseLendingNFT,
            _leverage,
            msg.value,
            block.timestamp
        );

246  emit FarmExit(
            _keyId,
            wiseLendingNFT,
            _allowedSpread,
            block.timestamp
        );

266  emit ManualPaybackShares(
            _keyId,
            farmingKeys[_keyId],
         

295   emit ManualWithdrawShares(
            _keyId,
            wiseLendingNFT,
            _withdrawShares,
            block.timestamp
        );            
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L71
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L90
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L131
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L174
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L246
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L266
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L295

```solidity

file: PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

284    emit RegistrationFarm(
            _nftId,
            block.timestamp
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L284

```solidity

file: PowerFarms/PowerFarmNFTs/MinterReserver.sol

169  emit ERC721Received(
            _operator,
            _from,
            _tokenId,
            _data
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L169

```solidity

file: OwnableMaster.sol

82  emit MasterProposed(
            msg.sender,
            _proposedOwner
        );