High Issues

1.Uninitialized Local Variables:
MainHelper.sol::248 => _validateIsolationPoolLiquidation(address,uint256,uint256)
MainHelper.sol::240 => _onlyIsolationPool(address)
OracleHelper.sol::256-277 => ETH_USD_PLACEHOLDER is never initialized. It is used in- .getETHPriceInUSD() 

2.Reentrancy:
AaveHub.sol::525-529 => actualAmountDeposit = _wrapAaveReturnValueDeposit(WETH_ADDRESS,paybackAmount,address(this))

3.Functions that send eth to arbitrary destinations:
PtOracleDerivative.sol::69-117 => The function latestAnswer() compares to a boolean constant: increaseCardinalityRequired == true 
PtOracleDerivative.sol::69-117 => The function latestAnswer() compares to a boolean constant: oldestObservationSatisfied == false

Medium Issues

1.Block Timestamp:
WiseLending.sol::275-305 => _syncPoolBeforeCodeExecution(address) uses timestamp for comparisons: _aboveThreshold(_poolToken) == true 

2.Divide Before Multiply:
MainHelper.sol::500-577 => _updatePseudoTotalAmounts(address) performs a multiplication on the result of a division - feeAmount = amountInterest * globalPoolData[_poolToken].poolFee /PRECISION_FACTOR_E18
PendlePowerFarmToken::403 => currentRate = totalLpAssetsToDistribute / ONE_WEEK
PendlePowerFarmControllerHelper.sol::73-93 => _getAmountToSend performs a multiplication on the result of a division uint256 sendingValue = _rewardValue * (PRECISION_FACTOR_E6 - exchangeIncentive) / PRECISION_FACTOR_E6; 
PtOracleDerivative.sol::69-117 => The latestAnswer() performs a multiplication on the result of a division:
        - uint256(answerUsdFeed) * PRECISION_FACTOR_E18 / POW_USD_FEED * POW_ETH_USD_FEED / uint256(answerEthUsdFeed) * ptToAssetRate / PRECISION_FACTOR_E18

3.Dangerous Strict Equality:
MainHelper.sol::500-577 => _updatePseudoTotalAmounts(address) uses a dangerous strict equality- feeAmount == 0 
MainHelper.sol::500-577 => _updatePseudoTotalAmounts(address) uses a dangerous strict equality- feeShares == 0
OracleHelper.sol::256-277 => getETHPriceInUSD() uses a dangerous strict equality- _chainLinkIsDead(ETH_USD_PLACEHOLDER) == true    and   state == true
PendlePowerFarmTokenFactory.sol::1457-1468 => _syncSupply() uses a dangerous strict equality- additonalAssets == 0 
PendlePowerFarmTokenFactory.sol::1509-1537 => previewDistribution() uses a dangerous strict equality- totalLpAssetsToDistribute == 0 (Downloads/PendlePowerFarmTokenFactory.sol#1514)
PendlePowerFarmTokenFactory.sol::1509-1537 => The function previewDistribution() uses a dangerous strict equality: block.timestamp == lastInteraction High Issues

1.Uninitialized Local Variables:
MainHelper.sol::248 => _validateIsolationPoolLiquidation(address,uint256,uint256)
MainHelper.sol::240 => _onlyIsolationPool(address)
OracleHelper.sol::256-277 => ETH_USD_PLACEHOLDER is never initialized. It is used in- .getETHPriceInUSD() 

2.Reentrancy:
AaveHub.sol::525-529 => actualAmountDeposit = _wrapAaveReturnValueDeposit(WETH_ADDRESS,paybackAmount,address(this))

3.Functions that send eth to arbitrary destinations:
PtOracleDerivative.sol::69-117 => The function latestAnswer() compares to a boolean constant: increaseCardinalityRequired == true 
PtOracleDerivative.sol::69-117 => The function latestAnswer() compares to a boolean constant: oldestObservationSatisfied == false

Medium Issues

1.Block Timestamp:
WiseLending.sol::275-305 => _syncPoolBeforeCodeExecution(address) uses timestamp for comparisons: _aboveThreshold(_poolToken) == true 

2.Divide Before Multiply:
MainHelper.sol::500-577 => _updatePseudoTotalAmounts(address) performs a multiplication on the result of a division - feeAmount = amountInterest * globalPoolData[_poolToken].poolFee /PRECISION_FACTOR_E18
PendlePowerFarmToken::403 => currentRate = totalLpAssetsToDistribute / ONE_WEEK
PendlePowerFarmControllerHelper.sol::73-93 => _getAmountToSend performs a multiplication on the result of a division uint256 sendingValue = _rewardValue * (PRECISION_FACTOR_E6 - exchangeIncentive) / PRECISION_FACTOR_E6; 
PtOracleDerivative.sol::69-117 => The latestAnswer() performs a multiplication on the result of a division:
        - uint256(answerUsdFeed) * PRECISION_FACTOR_E18 / POW_USD_FEED * POW_ETH_USD_FEED / uint256(answerEthUsdFeed) * ptToAssetRate / PRECISION_FACTOR_E18

3.Dangerous Strict Equality:
MainHelper.sol::500-577 => _updatePseudoTotalAmounts(address) uses a dangerous strict equality- feeAmount == 0 
MainHelper.sol::500-577 => _updatePseudoTotalAmounts(address) uses a dangerous strict equality- feeShares == 0
OracleHelper.sol::256-277 => getETHPriceInUSD() uses a dangerous strict equality- _chainLinkIsDead(ETH_USD_PLACEHOLDER) == true    and   state == true
PendlePowerFarmTokenFactory.sol::1457-1468 => _syncSupply() uses a dangerous strict equality- additonalAssets == 0 
PendlePowerFarmTokenFactory.sol::1509-1537 => previewDistribution() uses a dangerous strict equality- totalLpAssetsToDistribute == 0 (Downloads/PendlePowerFarmTokenFactory.sol#1514)
PendlePowerFarmTokenFactory.sol::1509-1537 => The function previewDistribution() uses a dangerous strict equality: block.timestamp == lastInteraction 
PendlePowerFarmToken::334-357 => _syncSupply() uses a dangerous strict equality- additonalAssets == 0 
PendlePowerFarmToken::386-414 => previewDistribution() uses a dangerous strict equality- totalLpAssetsToDistribute == 0 
WiseOracleHub.sol::1144-1156 => The function getETHPriceInUSD()uses a dangerous strict equality: _chainLinkIsDead(ETH_USD_PLACEHOLDER) == true (WiseOracleHub.sol#1149)
WiseOracleHub.sol::1635-1649 => THev function latestResolver(address) uses a dangerous strict equality: chainLinkIsDead(_tokenAddress) == true (WiseOracleHub.sol#1642)

4.Reentrancy:
MinterReserver.sol::137-161 => Reentrancy in _mintKeyForUser(uint256,address) due this external call:- mintKey(_userAddress,_keyId), 
Reentrancy in SendValueHelper._sendValue(address,uint256) (Downloads/SendValueHelper.sol#11-35) due to this external call- (success) = address(_recipient).call{value: _amount}() 
        
5. Incorrect ERC20 interface:
PendlePowerFarmTokenFactory.sol::252-364 => IPendle Market has incorrect ERC20 function interface: IPendleMarket.transferFrom(address,address,uint256) 

6. Unused return
PtOracleDerivative.sol::69-117 => The function latestAnswer() ignores return value by (answerUsdFeed) = USD_FEED_ASSET.latestRoundData() 
PtOracleDerivative.sol::69-117 => The function latestAnswer() ignores return value by (answerEthUsdFeed) = ETH_FEED_ASSET.latestRoundData() 
PtOracleDerivative.sol::69-117 => The function latestAnswer() ignores return value by (increaseCardinalityRequired,oldestObservationSatisfied) = ORACLE_PENDLE_PT.getOracleState(PENDLE_MARKET_ADDRESS,TWAP_DURATION) 

Low Issues

1.Boolean Equality:
WiseLending.sol::275-305=> _syncPoolBeforeCodeExecution(address) compares to a boolean constant - _aboveThreshold(_poolToken) == true 
WiseLending.sol::275-305=> _handleWithdrawShares(address,uint256,address,uint256,bool) compares to a boolean constant -_onBehalf == false (1527)
PendlePowerFarmLeverageLogic.sol::326-348=> _paybackExactShares(bool,uint256,uint256) compares to a boolean constant-_isAave == true (333)
PendlePowerFarmLeverageLogic.sol::531-553=> _borrowExactAmount(bool,uint256,uint256) compares to a boolean constant- _isAave == true (538)
PendlePowerFarmLeverageLogic.sol::=> _logicClosePosition(uint256,uint256,uint256,uint256,uint256,address,bool,bool) compares to a boolean constant- _ethBack == true (215)
WiseCore.sol::44-94=> _coreWithdrawToken() compares to a boolean constant- _onBehalf == true 
WiseCore.sol::349-421=> _coreBorrowTokens() compares to a boolean constant- _onBehalf == true 
MinterReserver._onlyKeyOwner(uint256) (Downloads/MinterReserver.sol#63-75) compares to a boolean constant:
        -require(bool)(isOwner(_keyId,msg.sender) == true) (Downloads/MinterReserver.sol#69-74)
PtOracleDerivative.latestAnswer() (Downloads/PtOracleDerivative.sol#69-117) compares to a boolean constant:
        -increaseCardinalityRequired == true (Downloads/PtOracleDerivative.sol#97)
PtOracleDerivative.latestAnswer() (Downloads/PtOracleDerivative.sol#69-117) compares to a boolean constant:
        -oldestObservationSatisfied == false
CallOptionalReturn._callOptionalReturn(address,bytes) (Downloads/TransferHelper.sol#81-107) compares to a boolean constant:
        -success == false (Downloads/TransferHelper.sol#100)

2.Calls inside a loop:
WiseSecurity.sol::499-550=> overallBorrowAPY(uint256) has external calls inside a loop: token = WISE_LENDING.getPositionBorrowTokenByIndex(_nftId,i) (506)
WiseSecurity.sol::499-550=> overallBorrowAPY(uint256) has external calls inside a loop: ethValue = WISE_ORACLE.getTokensInETH(token,amount) (528)
WiseSecurity.sol::499-550=> overallNetAPY(uint256) has external calls inside a loop: token = WISE_LENDING.getPositionBorrowTokenByIndex(_nftId,i) (533)
WiseSecurity.sol::499-550=> overallNetAPY(uint256) has external calls inside a loop: token = WISE_LENDING.getPositionLendingTokenByIndex(_nftId,i) (539)

3.Block Timestamp:
WiseSecurity.sol::556-635=> overallNetAPY(uint256) uses timestamp for comparisons - ethValueGain >= ethValueDebt (592)
WiseSecurity.sol::556-635=> overallNetAPY(uint256) (WiseSecurity.sol#5116-5195) uses timestamp for comparisons -ethValueGain >= ethValueDebt (623)
MainHelper.sol::298-343=> _cleanUp(address) uses timestamp for comparisons- difference > allowedDifference (328)
MainHelper.sol::869-904=> _aboveThreshold(address) uses timestamp for comparisons- block.timestamp - timestampsPoolData[_poolToken].timeStampScaling >= THREE_HOURS (903)
MainHelper.sol::1094-1117=> _increaseResonanceFactor(address) uses timestamp for comparisons- sum > borrowData.maxPole (1109)
PendlePowerFarmLeverageLogic.sol::134-230=> _logicClosePosition(uint256,uint256,uint256,uint256,uint256,address,bool,bool) uses timestamp for comparisons- ethValueAfter < ethValueBefore (211)
PendlePowerFarmLeverageLogic.sol::560-594=> _coreLiquidation(uint256,uint256,uint256) uses timestamp for comparisons- _checkDebtRatio(_nftId) == true (571)
OracleHelper.sol::556-586=> _chainLinkIsDead(address) uses timestamp for comparisons- upd > heartBeat[_tokenAddress], - block.timestamp < upd (580-584)

4. Unused state variables:
PendlePowerFarmTokenFactory.sol::645 => Variable ZERO_ADDRESS is never used in PendlePowerFarmTokenFactory.sol anywhere.
Declarations.sol::1212 => Variable REF_CODE is never used in Declarations.sol anywhere.
Declarations.sol::1222 => Variable PRECISION_FACTOR_E9 is never used in Declarations.sol anywhere.
Declarations.sol::1223 => Variable PRECISION_FACTOR_E18 is never used in Declarations (Downloads/Declarations.sol#1206-1305)
Declarations.MAX_AMOUNT (Downloads/`	Declarations.sol#1224) is never used in Declarations (Downloads/Declarations.sol#1206-1305)

5. Low-level calls
Low level call in SendValueHelper._sendValue(address,uint256) (Downloads/SendValueHelper.sol#11-35):
        - (success) = address(_recipient).call{value: _amount}() (Downloads/SendValueHelper.sol#23-28)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-calls

Low level call in CallOptionalReturn._callOptionalReturn(address,bytes) (Downloads/TransferHelper.sol#81-107):
        - (success,returndata) = token.call(data) (Downloads/TransferHelper.sol#88-93)
------------------------------------------------------------------------------------------------------------------
PendlePowerFarmToken::334-357 => _syncSupply() uses a dangerous strict equality- additonalAssets == 0 
PendlePowerFarmToken::386-414 => previewDistribution() uses a dangerous strict equality- totalLpAssetsToDistribute == 0 
WiseOracleHub.sol::1144-1156 => The function getETHPriceInUSD()uses a dangerous strict equality: _chainLinkIsDead(ETH_USD_PLACEHOLDER) == true (WiseOracleHub.sol#1149)
WiseOracleHub.sol::1635-1649 => THev function latestResolver(address) uses a dangerous strict equality: chainLinkIsDead(_tokenAddress) == true (WiseOracleHub.sol#1642)

4.Reentrancy:
MinterReserver.sol::137-161 => Reentrancy in _mintKeyForUser(uint256,address) due this external call:- mintKey(_userAddress,_keyId), 
Reentrancy in SendValueHelper._sendValue(address,uint256) (Downloads/SendValueHelper.sol#11-35) due to this external call- (success) = address(_recipient).call{value: _amount}() 
        
5. Incorrect ERC20 interface:
PendlePowerFarmTokenFactory.sol::252-364 => IPendle Market has incorrect ERC20 function interface: IPendleMarket.transferFrom(address,address,uint256) 

6. Unused return
PtOracleDerivative.sol::69-117 => The function latestAnswer() ignores return value by (answerUsdFeed) = USD_FEED_ASSET.latestRoundData() 
PtOracleDerivative.sol::69-117 => The function latestAnswer() ignores return value by (answerEthUsdFeed) = ETH_FEED_ASSET.latestRoundData() 
PtOracleDerivative.sol::69-117 => The function latestAnswer() ignores return value by (increaseCardinalityRequired,oldestObservationSatisfied) = ORACLE_PENDLE_PT.getOracleState(PENDLE_MARKET_ADDRESS,TWAP_DURATION) 

Low Issues

1.Boolean Equality:
WiseLending.sol::275-305=> _syncPoolBeforeCodeExecution(address) compares to a boolean constant - _aboveThreshold(_poolToken) == true 
WiseLending.sol::275-305=> _handleWithdrawShares(address,uint256,address,uint256,bool) compares to a boolean constant -_onBehalf == false (1527)
PendlePowerFarmLeverageLogic.sol::326-348=> _paybackExactShares(bool,uint256,uint256) compares to a boolean constant-_isAave == true (333)
PendlePowerFarmLeverageLogic.sol::531-553=> _borrowExactAmount(bool,uint256,uint256) compares to a boolean constant- _isAave == true (538)
PendlePowerFarmLeverageLogic.sol::=> _logicClosePosition(uint256,uint256,uint256,uint256,uint256,address,bool,bool) compares to a boolean constant- _ethBack == true (215)
WiseCore.sol::44-94=> _coreWithdrawToken() compares to a boolean constant- _onBehalf == true 
WiseCore.sol::349-421=> _coreBorrowTokens() compares to a boolean constant- _onBehalf == true 
MinterReserver._onlyKeyOwner(uint256) (Downloads/MinterReserver.sol#63-75) compares to a boolean constant:
        -require(bool)(isOwner(_keyId,msg.sender) == true) (Downloads/MinterReserver.sol#69-74)
PtOracleDerivative.latestAnswer() (Downloads/PtOracleDerivative.sol#69-117) compares to a boolean constant:
        -increaseCardinalityRequired == true (Downloads/PtOracleDerivative.sol#97)
PtOracleDerivative.latestAnswer() (Downloads/PtOracleDerivative.sol#69-117) compares to a boolean constant:
        -oldestObservationSatisfied == false
CallOptionalReturn._callOptionalReturn(address,bytes) (Downloads/TransferHelper.sol#81-107) compares to a boolean constant:
        -success == false (Downloads/TransferHelper.sol#100)

2.Calls inside a loop:
WiseSecurity.sol::499-550=> overallBorrowAPY(uint256) has external calls inside a loop: token = WISE_LENDING.getPositionBorrowTokenByIndex(_nftId,i) (506)
WiseSecurity.sol::499-550=> overallBorrowAPY(uint256) has external calls inside a loop: ethValue = WISE_ORACLE.getTokensInETH(token,amount) (528)
WiseSecurity.sol::499-550=> overallNetAPY(uint256) has external calls inside a loop: token = WISE_LENDING.getPositionBorrowTokenByIndex(_nftId,i) (533)
WiseSecurity.sol::499-550=> overallNetAPY(uint256) has external calls inside a loop: token = WISE_LENDING.getPositionLendingTokenByIndex(_nftId,i) (539)

3.Block Timestamp:
WiseSecurity.sol::556-635=> overallNetAPY(uint256) uses timestamp for comparisons - ethValueGain >= ethValueDebt (592)
WiseSecurity.sol::556-635=> overallNetAPY(uint256) (WiseSecurity.sol#5116-5195) uses timestamp for comparisons -ethValueGain >= ethValueDebt (623)
MainHelper.sol::298-343=> _cleanUp(address) uses timestamp for comparisons- difference > allowedDifference (328)
MainHelper.sol::869-904=> _aboveThreshold(address) uses timestamp for comparisons- block.timestamp - timestampsPoolData[_poolToken].timeStampScaling >= THREE_HOURS (903)
MainHelper.sol::1094-1117=> _increaseResonanceFactor(address) uses timestamp for comparisons- sum > borrowData.maxPole (1109)
PendlePowerFarmLeverageLogic.sol::134-230=> _logicClosePosition(uint256,uint256,uint256,uint256,uint256,address,bool,bool) uses timestamp for comparisons- ethValueAfter < ethValueBefore (211)
PendlePowerFarmLeverageLogic.sol::560-594=> _coreLiquidation(uint256,uint256,uint256) uses timestamp for comparisons- _checkDebtRatio(_nftId) == true (571)
OracleHelper.sol::556-586=> _chainLinkIsDead(address) uses timestamp for comparisons- upd > heartBeat[_tokenAddress], - block.timestamp < upd (580-584)

4. Unused state variables:
PendlePowerFarmTokenFactory.sol::645 => Variable ZERO_ADDRESS is never used in PendlePowerFarmTokenFactory.sol anywhere.
Declarations.sol::1212 => Variable REF_CODE is never used in Declarations.sol anywhere.
Declarations.sol::1222 => Variable PRECISION_FACTOR_E9 is never used in Declarations.sol anywhere.
Declarations.sol::1223 => Variable PRECISION_FACTOR_E18 is never used in Declarations.sol anywhere.


### Time spent:
20 hours