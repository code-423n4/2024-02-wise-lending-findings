## G-1. Use unchecked for arithmetic operations that cannot underflow or overflow.

This can save gas by avoiding unnecessary checks. For example, in the `WiseLending::_getCurrentSharePriceMax` function, the `timeDifference` and return statements can be wrapped in an unchecked block:

```solidity
function _getCurrentSharePriceMax(
    address _poolToken
)
    private
    view
    returns (uint256)
{
    uint256 timeDifference = block.timestamp - timestampsPoolData[_poolToken].initialTimeStamp;

    unchecked {
        return timeDifference * RESTRICTION_FACTOR + PRECISION_FACTOR_E18;
    }
}
```

## G-2. Use immutable instead of constant for variables that are set only once in the constructor.

This can save gas by avoiding storage reads. For example, in the WiseLending contract, the `master`, `wiseOracleHubAddress`, and `nftContract` variables can be changed to immutable:

## G-3 Use storage instead of memory for variables that are modified in a loop.

This can save gas by avoiding multiple memory copies. For example, in the `WiseLending::_syncPoolBeforeCodeExecution` function, the `lendSharePrice` and `borrowSharePrice` variables can be changed to storage:

```solidity
function _syncPoolBeforeCodeExecution(
    address _poolToken
)
    private
    returns (
        uint256 lendSharePrice,
        uint256 borrowSharePrice
    )
{
    // rest of function

    (lendSharePrice, borrowSharePrice) = _getSharePrice(_poolToken);
}
```

## G-4 Use delete to clear storage variables that are no longer needed.

This can save gas by freeing up storage space. For example, in the `_handleWithdrawShares` function, the `withdrawAmount` variable can be deleted after it is used:

```solidity
function _handleWithdrawShares(
    address _caller,
    uint256 _nftId,
    address _poolToken,
    uint256 _shares,
    bool _onBehalf
)
    private
    returns (uint256)
{
    // rest of function

    uint256 withdrawAmount = _cashoutAmount(_poolToken, _shares);

    // use withdrawAmount

    delete withdrawAmount;
}
```

## G-5 Use `.` instead of `[]` for structs:

You can use `_curveSwapStructData.curvePool` instead of `_curveSwapStructData[0]` in prepareCurvePools function.

```solidity
        curveSwapInfoData[_poolToken] = _curveSwapStructData;
        curveSwapInfoToken[_poolToken] = _curveSwapStructToken;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L150-L151

## G-6 Reduce the size of `MainHelper::userLendingData`:

Instead of storing all user lending data for each NFT in a separate map, consider consolidating data into a single array. This will reduce the storage space required for userLendingData.

## G-7 Combine events to reduce gas cost:

combine events `WiseCore::_coreWithdrawToken`

```solidity

 if (_onBehalf == true) {
            emit FundsWithdrawnOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
        } else {
            emit FundsWithdrawn(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
        }


  if (_onBehalf == true) {
            emit FundsBorrowedOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
        } else {
            emit FundsBorrowed(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
        }

```

## G-8 Admin function should payable:

```solidity
72  function setWiseSecurity(address _securityAddress) external onlyMaster {
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L72

```solidity
  function renounceOwnership() external onlyOwner {
        master = address(0x0);
    }

    function setLastUpdateGlobal(uint256 _time) external onlyOwner {
        lastUpdateGlobal = _time;
    }

    function setRoundData(uint80 _roundId, uint256 _updateTime) external onlyOwner {
        timeStampByRoundId[_roundId] = _updateTime;
    }

    function setGlobalAggregatorRoundId(uint80 _aggregatorRoundId) external onlyOwner {
        globalRoundId = _aggregatorRoundId;
  }
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L22-L36

```solidity
  function setRepayBadDebtIncentive(uint256 _percent) external onlyMaster {
  function setAaveFlag(address _poolToken, address _underlyingToken) external onlyMaster {
  function setAaveFlagBulk(address[] calldata _poolTokens, address[] calldata _underlyingTokens)
        external
        onlyMaster
    {
   function setPoolFee(address _poolToken, uint256 _newFee) external onlyMaster {
   function setPoolFeeBulk(address[] calldata _poolTokens, uint256[] calldata _newFees) external onlyMaster {
   function proposeIncentiveMaster(address _proposedIncentiveMaster) external onlyIncentiveMaster {
   function increaseIncentiveA(uint256 _value) external onlyIncentiveMaster {
   function increaseIncentiveB(uint256 _value) external onlyIncentiveMaster {
   function addPoolTokenAddress(address _poolToken) external onlyWiseLending {
   function addPoolTokenAddressManual(address _poolToken) external onlyMaster {
   function removePoolTokenManual(address _poolToken) external onlyMaster {
   function increaseTotalBadDebtLiquidation(uint256 _amount) external onlyWiseSecurity {
   function setBadDebtUserLiquidation(uint256 _nftId, uint256 _amount) external onlyWiseSecurity {
   function setBeneficial(address _user, address[] calldata _feeTokens) external onlyMaster {
   function revokeBeneficial(address _user, address[] memory _feeTokens) external onlyMaster{
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L46

```solidity
function changeMinDeposit(uint256 _newMinDeposit) external onlyMaster {
function shutDownFarm(bool _state) external onlyMaster {
function exitFarm(uint256 _keyId, uint256 _allowedSpread, bool _ethBack)
        external
        updatePools
        onlyKeyOwner(_keyId)
    {
function manuallyWithdrawShares(uint256 _keyId, uint256 _withdrawShares)
        external
        updatePools
        onlyKeyOwner(_keyId)
    {
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L50

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

function withdrawLp(address _pendleMarket, address _to, uint256 _amount)
    external
    onlyChildContract(_pendleMarket)
{
function addPendleMarket(
    address _pendleMarket,
    string memory _tokenName,
    string memory _symbolName,
    uint16 _maxCardinality
) external onlyMaster {

function updateRewardTokens(address _pendleMarket) external onlyChildContract(_pendleMarket) returns (bool) {
function changeExchangeIncentive(uint256 _newExchangeIncentive) external onlyMaster {
function changeMintFee(address _pendleMarket, uint256 _newFee) external onlyMaster {
  function lockPendle(uint256 _amount, uint128 _weeks, bool _fromInside, bool _sameExpiry)
      external
      onlyMaster
      returns (uint256 newVeBalance)
  {
function claimArb(uint256 _accrued, bytes32[] calldata _proof) external onlyArbitrum {
function withdrawLock() external onlyMaster returns (uint256 amount) {
function increaseReservedForCompound(address _pendleMarket, uint256[] calldata _amounts)
       external
       onlyChildContract(_pendleMarket)
   {
function overWriteIndex(address _pendleMarket, uint256 _tokenIndex) public onlyChildContract(_pendleMarket) {
function overWriteIndexAll(address _pendleMarket) external onlyChildContract(_pendleMarket) {
function overWriteAmounts(address _pendleMarket) external onlyChildContract(_pendleMarket) {
function forwardETH(address _to, uint256 _amount) external onlyMaster {
function vote(address[] calldata _pools, uint64[] calldata _weights) external onlyMaster {
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L23

```solidity
function changeMintFee(uint256 _newFee) external onlyController {
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L378

```solidity

    function setFarmContract(address _farmContract) external onlyMaster {
    function mintKey(address _keyOwner, uint256 _keyId) external onlyFarmContract {
    function burnKey(uint256 _keyId) external onlyFarmContract {
    function setBaseURI(string calldata _newBaseURI) external onlyMaster {
    function setBaseExtension(string calldata _newBaseExtension) external onlyMaster {
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L29

```solidity

function curveSecurityCheck(address _poolToken) external onlyWiseLending {
function checkBadDebtLiquidation(uint256 _nftId) external onlyWiseLending {
function setBlacklistToken(address _tokenAddress, bool _state) external onlyMaster {
function setSecurityWorker(address _entitiy, bool _state) external onlyMaster {
function revokeShutdown() external onlyMaster {
function changeMinDepositValue(uint256 _newMinDepositValue) external onlyMaster {
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L142

## G-9 Calculation(addition, subtruction) which won’t underflow can be unchecked in

```solidity

storageMarket.observationCardinalityNext + 1
return underlyingLpAssetsCurrent
            + totalLpAssetsToDistribute;
   return previewDistribution()
            + underlyingLpAssetsCurrent;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L385

```solidity
ownerTokenCount + reservedCount
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L135

```solidity
totalPool + bareToken
  uint256 sum = delta
            + borrowData.pole;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L321

```solidity
totalReserved + 1;
ownerTokenCount + reservedCount
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L125

```solidity
withdrawAmount = withdrawAmount - _solelyWithdrawAmount;
tokenAmount = tokenAmount - _poolWithdrawAmount;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L812

```solidity
return receiveAmount + potentialPureExtraCashout;
uint256 shareDifference = cashoutShares
            - totalPoolInShares;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L585
