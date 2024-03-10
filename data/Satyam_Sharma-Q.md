## L-01
`badDebtPosition[ _nftId] == 0` should be checked under `FeeManagerHelper._updateUserBadDebt` internal function, because as _updateUserBadDebt called under `FeeManager.paybackBadDebtForToken` calls 
 -> `updatePositionCurrentBadDebt` further calls -> `_updateUserBadDebt` internal function and this function has a mapping for `badDebtPosition` position as
```solidity
FeeManagerHelper.sol#L147
 uint256 currentBadDebt = badDebtPosition[_nftId];
```
the issue here is that after saving `badDebtPosition` mapping for particular _nftId it donot check whether the `badDebtPosition[_nftId]` is zero or not as its checked in,
```solidity
if (badDebtPosition[_nftId] == 0) {
            return (
                0,
                0
            );
        }
```
by checking badDebtPosition under `_updateUserBadDebt` for the particular nftId it should make revert than n their for `badDebtPosition[_nftId] == 0` it will make the function skip the next steps and will save gas.

## L-02
`_removeEmptyLendingData` called twice in the same function.
**Description**
Under `WiseCore._withdrawOrAllocateSharesLiquidation` an internal function  `MainHelper._removeEmptyLendingData`` called twice,
during an internal function called to `_coreWithdrawBare`
```solidity
WiseCore.sol#L488
_coreWithdrawBare(
            _nftId,
            _poolToken,
            totalPoolToken,
            totalPoolInShares
        );
```
```solidity
WiseCore.sol#L319
_removeEmptyLendingData(
            _nftId,
            _poolToken
        );
```
and in the function itself

```solidity
WiseCore.sol#L514
_removeEmptyLendingData(
            _nftId,
            _poolToken
        );
```
Remove the last call to `_removeEmptyLendingData`.

## L-03
In case of weightedtotal collateral factor should also checked for equal to `PRECISION_FACTOR_E18`
under `PoolManager.setPoolParameters` function the collateral factor for both unweightedtotal and weightedtotal checked for same condition, which can make the weightedtotal to be act as unweightedtotal collateral, (I have described this in my another finding how it can act as same),
```solidity
PoolManager.sol#L112-L119
if (_collateralFactor > 0) {
            lendingPoolData[_poolToken].collateralFactor = _collateralFactor;
        }

        _validateParameter(
            _collateralFactor,
            PRECISION_FACTOR_E18
        );
```
`_validateParameter` for both weighted and unweighted checked as,
```solidity
function _validateParameter(
        uint256 _parameterValue,
        uint256 _parameterLimit
    )
        internal
        pure
    {
        if (_parameterValue > _parameterLimit) {
            revert InvalidAction();
        }
    }
```
instead if statement for weighted collateral factor should be checked as,
```solidity
if (_parameterValue >= _parameterLimit) {
            revert InvalidAction();
        }
```

## L-04
`_data.tokenToRecieve` should be checked against blacklisted token under `WiseCore._coreLiquidation` and `WiseLending.liquidatePartiallyFromTokens` function,
```solidity
WiseCore.sol#591
receiveAmount = _calculateReceiveAmount(
            _data.nftId,
            _data.nftIdLiquidator,
            _data.tokenToRecieve,//@audit shouldn't be blacklisted token
            collateralPercentage
        );

WiseLending.sol#1265
data.tokenToRecieve = _receiveToken
```

