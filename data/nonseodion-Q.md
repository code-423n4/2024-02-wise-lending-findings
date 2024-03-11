## L-1: No check for Negative Chainlink price
The price returned by Chainlink is of type int256. There's no check to see if the price is negative before being used in the code.

**[OracleHelper.sol#L253-L258](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L253-L258)**
```solidity
        (
            ,
            int256 answer,
            ,
            ,
        ) = priceFeed[_tokenAddress].latestRoundData();
```

## L-2: No check if Liquidators own the nft they want to send received shares to.
When liquidators liquidate a position, they can send any received shares to an NFT they don't know. There's no check in `_checkLiquidatorNft()` if the liquidator owns the NFT. This allows a malicious liquidator to send a tiny amount of tokens to an NFT and make actions like withdrawing and borrowing using that NFT more costly.

**[MainHelper.sol#L258-L276](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L258-L276)**
```solidity
    function _checkLiquidatorNft(
        uint256 _nftId,
        uint256 _nftIdLiquidator
    )
        internal
        view
    {
        if (positionLocked[_nftIdLiquidator] == true) {
            revert LiquidatorIsInPowerFarm();
        }

        if (_nftIdLiquidator == _nftId) {
            revert InvalidLiquidator();
        }

        if (_nftIdLiquidator >= POSITION_NFT.getNextExpectedId()) {
            revert InvalidLiquidator();
        }
    }
```

## L-3: If the largest time difference occurs more than once during recalibration it is returned instead of the second biggest price difference.
If the time difference returned from the historical prices are: 

10, 11, 23, 23, 65, 76, 89, 89

89 is returned as the second biggest time instead of 76.

**[OracleHelper.sol#L637C1-L645C1](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L637C1-L645C1)**
```solidity
            if (currentDiff >= currentBiggest) {

                currentSecondBiggest = currentBiggest;
                currentBiggest = currentDiff;

            } else if (currentDiff > currentSecondBiggest) {
                currentSecondBiggest = currentDiff;
            }

```

## L-4:  claimOwnership() function does not clear proposedMaster in OwnnableMaster.

**[OwnableMaster.sol#L92-L97](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/OwnableMaster.sol#L92-L97)**
```solidity
    function claimOwnership()
        external
        onlyProposed
    {
        master = proposedMaster;
    }
```

## L-5: Rewards from Pendle Reward tokens are lost when a new reward token is added.
**Note:** This is different from the known issue.
When a user tries to deposit with `depositExactAmount()` in the PendlePowerFarmToken, the `syncSupply()` modifier is called.

If a new reward token is added the indexes of the reward tokens are overwritten to the current index. The old rewards earned before are lost here which is what the known issue talks about.


**(PendlePowerFarmToken.sol#L81-L96)[https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L81-L96]**
```solidity
    modifier syncSupply()
    {
        _triggerIndexUpdate();
❌       _overWriteCheck();
        _syncSupply();
❌       _updateRewards();
        _setLastInteraction();
        _increaseCardinalityNext();
        uint256 sharePriceBefore = _getSharePrice();
        _;
        _validateSharePriceGrowth(
            _validateSharePrice(
                sharePriceBefore
            )
        );
    }
```

When `_updateRewards()` is called, it redeems the rewards from the Pendle market but since the indexes have already been overwritten these rewards aren't recorded in the PendlePowerFarmController. It returns when it gets to the portion of `_calculateRewardsClaimedOutside()` in the snippet below

**[PendlePowerFarmToken.sol#L266-L272](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L266-L272)**
```
            if (index == lastIndex[i]) {
                rewardsOutsideArray[i] = 0;
                unchecked {
                    ++i;
                }
                continue;
            }
```

## L-6: The function `maximumBorrowToken()` does not check `allowBorrow` to know if the token can be borrowed.

**[WiseSecurity.sol#L868-L899](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L868-L899)**
```solidity
function maximumBorrowToken(
        uint256 _nftId,
        address _poolToken,
        uint256 _interval
    )
        external
        view
        returns (uint256 tokenAmount)
    {
        uint256 term = _overallETHCollateralsWeighted(_nftId, _interval)
            * BORROW_PERCENTAGE_CAP
            / PRECISION_FACTOR_E18;

        uint256 borrowETH = term
            - _overallETHBorrow(
                _nftId,
                _interval
            );

        tokenAmount = WISE_ORACLE.getTokensFromETH(
            _poolToken,
            borrowETH
        );

        uint256 maxPoolAmount = WISE_LENDING.getTotalPool(
            _poolToken
        );

        if (tokenAmount > maxPoolAmount) {
            tokenAmount = maxPoolAmount;
        }
    }
```