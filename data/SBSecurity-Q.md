### Low Risk

| Count | Title |
| --- | --- |
| [L-01] | Critical functions not called on child contract deployments |
| [L-02] | Liquidation bots will not be able to determine state of borrow, potentially delaying liquidations  |
| [L-03] | Code with no purpose only increases complexity  |
| [L-04] | Duplicate state variables  |
| [L-05] | Consider setting minLpOut on SY to LP Pendle tokens swaps |
| [L-06] | Hardcoded Uniswap pool fee, will render power farm contract useless |

| Total Low-Risk Issues | 6 |
| --- | --- |

### Non-Critical

| Count | Title |
| --- | --- |
| [NC-01] | Typos |

| Total Non-Critical Issues | 1 |
| --- | --- |

## Low Risks

## [L-01] Critical functions not called on child contract deployments

**Issue Description:**

Functions such as `setPoolParameters` and `setParamsLASA`, should be called immediately after `createPool` , without them there will be no way for a poolMarket to be properly configured and operate.

[PoolManager.sol#L100-L120](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L100-L120)

```solidity
function setPoolParameters(
  address _poolToken,
  uint256 _collateralFactor,
  uint256 _maximumDeposit
)
  external
  onlyMaster
{
  if (_maximumDeposit > 0) {
      maxDepositValueToken[_poolToken] = _maximumDeposit;
  }

  if (_collateralFactor > 0) {
      lendingPoolData[_poolToken].collateralFactor = _collateralFactor;
  }

  _validateParameter(
      _collateralFactor,
      PRECISION_FACTOR_E18
  );
}
```

[PoolManager.sol#L25-L98](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L25-L98)

```solidity
function setParamsLASA(
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
    if (parametersLocked[_poolToken] == true) {
        revert InvalidAction();
    }

    parametersLocked[_poolToken] = _isFinal;

    AlgorithmEntry storage algoData = algorithmData[
        _poolToken
    ];

    algoData.increasePole = _steppingDirection;

    _validateParameter(
        _upperBoundMaxRate,
        UPPER_BOUND_MAX_RATE
    );

    _validateParameter(
        _lowerBoundMaxRate,
        LOWER_BOUND_MAX_RATE
    );

    _validateParameter(
        _lowerBoundMaxRate,
        _upperBoundMaxRate
    );

    uint256 staticMinPole = _getPoleValue(
        _poolMulFactor,
        _upperBoundMaxRate
    );

    uint256 staticMaxPole = _getPoleValue(
        _poolMulFactor,
        _lowerBoundMaxRate
    );

    uint256 staticDeltaPole = _getDeltaPole(
        staticMaxPole,
        staticMinPole
    );

    uint256 startValuePole = _getStartValue(
        staticMaxPole,
        staticMinPole
    );

    _validateParameter(
        _poolMulFactor,
        PRECISION_FACTOR_E18
    );

    borrowRatesData[_poolToken] = BorrowRatesEntry(
        startValuePole,
        staticDeltaPole,
        staticMinPole,
        staticMaxPole,
        _poolMulFactor
    );

    algoData.bestPole = startValuePole;
    algoData.maxValue = lendingPoolData[_poolToken].totalDepositShares;
}
```

**Recommendation:**

Call both functions inside `createPool` as it will remove the need of additional function executions for each action.

## [L-02] Liquidation bots will not be able to determine state of borrow, potentially delaying liquidations because of reverting `getLiveDebtRatio` function

**Issue Description:**

Currently, this is the only function that can be used for liquidation bots to query if a given NFT is a liquidateable, but `overallETHCollateralsWeighted` will revert if one of the pool tokens contained in the `positionLendTokenData`  for the given NFT id is blacklisted. From the known issues of the protocol we can see that the intentions of the sponsors is to allow such positions to be liquidated. That way bots specially developed to serve WiseLending will have no other way to verify the health factor of a position, unless blindly liquidating it and risking to end up paying the gas.

[WiseSecurity.sol#L59-L77](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L59-L77)

```solidity
function getLiveDebtRatio(
  uint256 _nftId
)
  external
  view
  returns (uint256)
{
  uint256 overallCollateral = overallETHCollateralsWeighted(
      _nftId
  );

  if (overallCollateral == 0) {
      return 0;
  }

  return overallETHBorrow(_nftId)
      * PRECISION_FACTOR_E18
      / overallCollateral;
}
```

[WiseSecurityHelper.sol#L65-L100](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L65-L100)

```solidity
function overallETHCollateralsWeighted(
    uint256 _nftId
)
    public
    view
    returns (uint256 weightedTotal)
{
    address tokenAddress;

    uint256 i;
    uint256 l = WISE_LENDING.getPositionLendingTokenLength(
        _nftId
    );

    while (i < l) {

        tokenAddress = WISE_LENDING.getPositionLendingTokenByIndex(
            _nftId,
            i
        );

        _checkPoolCondition(tokenAddress);//@audit reverts on blacklisted tokens
        
        weightedTotal += WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor
            * getFullCollateralETH(
                _nftId,
                tokenAddress
            ) / PRECISION_FACTOR_E18;

        unchecked {
            ++i;
        }
    }
}
```

**Recommendation:** 

Consider using the same function which is used at liquidation to properly represent the state of a position and not cause eventual delays, risking the position to become insolvent.

## [L-03] Code with no purpose only increases complexity

**Issue Description:**

There are a lot of functions across the codebase with tags that says they are unused and should be deleted, but there are also functions without any contracts, such as 

[AaveHelper.sol#L243-L277](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHelper.sol#L243-L277)

```solidity
function _prepareCollaterals(
  uint256 _nftId,
  address _poolToken
)
  private
{
  uint256 i;
  uint256 l = WISE_LENDING.getPositionLendingTokenLength(
      _nftId
  );

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

      WISE_LENDING.preparePool( //@audit missing in WiseLending
          currentAddress
      );

      WISE_LENDING.newBorrowRate( //@audit missing in WiseLending
          _poolToken
      );
  }
}
```

[AaveHelper.sol#L279-L313](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHelper.sol#L278-L313)

```solidity
function _prepareBorrows(
  uint256 _nftId,
  address _poolToken
)
  private
{
  uint256 i;
  uint256 l = WISE_LENDING.getPositionBorrowTokenLength(
      _nftId
  );

  for (i; i < l;) {

      address currentAddress = WISE_LENDING.getPositionBorrowTokenByIndex(
          _nftId,
          i
      );

      unchecked {
          ++i;
      }

      if (currentAddress == _poolToken) {
          continue;
      }

      WISE_LENDING.preparePool( //@audit missing in WiseLending
          currentAddress
      );

      WISE_LENDING.newBorrowRate( //@audit missing in WiseLending
          _poolToken
      );
  }
}
```

These functions are not being used anymore but still are presented in the codebase, they only add additional complexity to the code, making it harder to read. They do not pose any security risk, since their modifier is `private`.

Such redundant functions can be found inside the Oracles contracts, such as ones calculating in `USD` asset.

**Recommendation:** 

Remove all the functions that are not being referenced in the contracts and not serving any particular purpose of `WiseLending` protocol.

## [L-04] Duplicate state variables

**Issue Description:**

In `PendlePowerFarmToken::initialize` we can see that there are variables that are set to same addresses, without any purpose:

[PendlePowerFarmToken.sol#L682-L732](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L682-L732)

```solidity
function initialize(
      address _underlyingPendleMarket,
      address _pendleController,
      string memory _tokenName,
      string memory _symbolName,
      uint16 _maxCardinality
  ) external {
      if (address(PENDLE_MARKET) != address(0)) {
          revert AlreadyInitialized();
      }

      PENDLE_MARKET = IPendleMarket(_underlyingPendleMarket);

      if (PENDLE_MARKET.isExpired() == true) {
          revert MarketExpired();
      }

      PENDLE_CONTROLLER = IPendleController(_pendleController);

      MAX_CARDINALITY = _maxCardinality;

      _name = _tokenName;
      _symbol = _symbolName;

      PENDLE_POWER_FARM_CONTROLLER = _pendleController;
      UNDERLYING_PENDLE_MARKET = _underlyingPendleMarket;

      (address pendleSyAddress,,) = PENDLE_MARKET.readTokens();

      PENDLE_SY = IPendleSy(pendleSyAddress);

      _decimals = PENDLE_SY.decimals();

      lastInteraction = block.timestamp;

      _totalSupply = 1;
      underlyingLpAssetsCurrent = 1;
      mintFee = 3000;
      INITIAL_TIME_STAMP = block.timestamp;
  }
```

As we can see `_underlyingPendleMarket` is assigned to both `PENDLE_MARKET` and `UNDERLYING_PENDLE_MARKET`, `_pendleController` to `PENDLE_CONTROLLER` and `PENDLE_POWER_FARM_CONTROLLER`. There is no need to have 2 variables, one holding the address and second holding the implementation, they can be derived from a single one and used where necessary.

**Recommendation:** 

Remove the redundant state variables 

```diff
function initialize(
      address _underlyingPendleMarket,
      address _pendleController,
      string memory _tokenName,
      string memory _symbolName,
      uint16 _maxCardinality
  ) external {
      if (address(PENDLE_MARKET) != address(0)) {
          revert AlreadyInitialized();
      }

      PENDLE_MARKET = IPendleMarket(_underlyingPendleMarket);

      if (PENDLE_MARKET.isExpired() == true) {
          revert MarketExpired();
      }

      PENDLE_CONTROLLER = IPendleController(_pendleController);

      MAX_CARDINALITY = _maxCardinality;

      _name = _tokenName;
      _symbol = _symbolName;

-     PENDLE_POWER_FARM_CONTROLLER = _pendleController;
-     UNDERLYING_PENDLE_MARKET = _underlyingPendleMarket;

      (address pendleSyAddress,,) = PENDLE_MARKET.readTokens();

      PENDLE_SY = IPendleSy(pendleSyAddress);

      _decimals = PENDLE_SY.decimals();

      lastInteraction = block.timestamp;

      _totalSupply = 1;
      underlyingLpAssetsCurrent = 1;
      mintFee = 3000;
      INITIAL_TIME_STAMP = block.timestamp;
  }
```

## [L-05] `minLpOut` on SY to LP Pendle tokens swaps is hardcoded at 0

**Issue Description:**

Hardcoding slippage parameters is not recommended, as this basically says that any amount back is expected.

[PendlePowerFarmLeverageLogic#L414-L529](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L414-L529)

```solidity
function _logicOpenPosition(
      bool _isAave,
      uint256 _nftId,
      uint256 _depositAmount,
      uint256 _totalDebtBalancer,
      uint256 _allowedSpread
  ) internal {
      uint256 reverseAllowedSpread = 2 * PRECISION_FACTOR_E18 - _allowedSpread;
...MORE CODE

      (uint256 netLpOut,) = PENDLE_ROUTER.addLiquiditySingleSy({
          _receiver: address(this),
          _market: address(PENDLE_MARKET),
          _netSyIn: syReceived,
          _minLpOut: 0,
          _guessPtReceivedFromSy: ApproxParams({
              guessMin: netPtFromSwap - 100,
              guessMax: netPtFromSwap + 100,
              guessOffchain: 0,
              maxIteration: 50,
              eps: 1e15
          })
      });

      uint256 ethValueBefore = _getTokensInETH(ENTRY_ASSET, _depositAmount);

      (uint256 receivedShares,) = IPendleChild(PENDLE_CHILD).depositExactAmount(netLpOut);

      uint256 ethValueAfter = _getTokensInETH(PENDLE_CHILD, receivedShares) * _allowedSpread / PRECISION_FACTOR_E18;

      if (ethValueAfter < ethValueBefore) {
          revert TooMuchValueLost();
      }

...MORE CODE
  }
```

From this code snippet we can see that basically any `netLpOut` is acceptable, which will result in the check after swap to revert. Just like all other swaps Pendle SY to LP is also a subject of slippage and proper parameter should be used.

**Recommendation:** 

Consider making possible user who enters the power farm to be able to specify any desired slippage tolerance for this types of swaps.

## [L-06] Hardcoded Uniswap pool fee will render power farm contract useless

**Issue Description:**

Using fee tier of 100 for the Uniswap pools will make most of the transactions fail because most of the WETH/ENTRY_ASSET pools have low liquidity, up to $50k and swaps will cause big price impact above the slippage specified and will make the whole functionality useless. 

That will require `PowerFarm` contracts to be redeployed with the right fee specified.

[PendlePowerFarmLeverageLogic#L414-L529](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L414-L529)

```solidity
  function _getTokensUniV3(uint256 _amountIn, uint256 _minOutAmount, address _tokenIn, address _tokenOut)
      internal
      returns (uint256)
  {
      return UNISWAP_V3_ROUTER.exactInputSingle(
          IUniswapV3.ExactInputSingleParams({
              tokenIn: _tokenIn,
              tokenOut: _tokenOut,
              fee: UNISWAP_V3_FEE,
              recipient: address(this),
              deadline: block.timestamp,
              amountIn: _amountIn,
              amountOutMinimum: _minOutAmount,
              sqrtPriceLimitX96: 0
          })
      );
  }
```

**Recommendation:**

Extract `UNISWAP_V3_FEE` as a constructor variable, making it possible for deployer to choose the desired pool where swaps will be performed.

## Non-Critical

## **[N‑01] Typos**

**Issue Description:** 

Various typos are presented in the codebase that should be fixed to improve the overall quality of the code and make the understanding easier

```solidity
 * - Aave pools  allow for maximal capital efficiency by earning aave supply APY for not
 *   borrowed funds. // NOTE - space after 'pools'
```

[WiseLending.sol#29](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L29)

```solidity
 * - Special curve pools nside beefy farms can be used as collateral, opening up new usage
 *   possibilities for these asset types. // NOTE - 'nside' -> 'inside'
```

[WiseLending.sol#32](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L32)

```solidity
 * @dev WiseSecurity is a core contract for wiseLending including most of
 * the performed security checks for withdraws, borrows, paybacks and liquidations.
 * It also has several read only functions providing UI data for a better user
 * experiencne. // NOTE - experiencne -> experience
```

[WiseSecurity.sol#18](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L18)

```solidity
 * @dev Set function for blacklisting token.
 * Those token can not be borrowed or used as // NOTE - token -> tokens
 * collateral anymore. Only callable by master.
```

[WiseSecurity.sol#977](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L977)

There are over 40 instances where spelling of the ‘position’ is wrong.

```solidity
 * @dev Internal increas function for
 * lending shares of a postion {_nftId} // NOTE - postion -> position
 * and {_poolToken}.
```

MainHelper.sol ([581](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L581), [596](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L596), [731](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L731), [792](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L792))

[WiseCore.sol#187](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L187)

[WiseLending.sol#1243](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L1243)

FeeManager.sol ([21](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L21), [726](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L726), [813](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L813), [882](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L882))

FeeManagerHelper.sol ([119](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManagerHelper.sol#L119), [131](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManagerHelper.sol#L131), [267](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManagerHelper.sol#L267))

PendlePowerFarm.sol ([55](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L55), [91](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L91))

PendlePowerFarmMathLogic.sol ([153](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L153), [172](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L172), [227](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L227), [287](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L287))

WiseSecurity.sol ([56](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L56), [440](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L440), [497](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L497), [554](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L554), [639](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L639), [692](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L692), [761](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L761), [824](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L824), [865](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L865))

WiseSecurityHelper.sol ([12](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L12), [63](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L63), [104](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L104), [142](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L142), [356](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L356), [390](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L390), [428](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L428), [483](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L483), [698](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L698), [835](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L835), [904](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L904))

AaveHub.sol ([431](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L431), [480](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L480), [554](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L554))