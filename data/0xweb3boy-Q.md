# [L-01] `WiseLending.sol::liquidatePartiallyFromTokens()` Powefarm positions will no be able to liquidate normal position

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1295C9-L1298C11

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L265-L267

`liquidatePartiallyFromTokens()` is calling internally `_checkLiquidatorNft()` function to check if the `_nftIdLiquidator` is a powerfarm position or not and if it is the whole function just reverts, which shouldn't be the case. Anyone should be able to liquidate normal position.

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

If you look at the above function there is a check ` if (positionLocked[_nftIdLiquidator] == true) {
            revert LiquidatorIsInPowerFarm();
        }` 
`positionLocked[_nftIdLiquidator] == true` this means that this function will revert since the liquidator is in `powerfarm`, and they can't liquidate normal users.



# [NC-01] `FeeManager.sol::claimIncentives()` Even should be emitted at last

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L297-L302

```solidity
    function claimIncentives(
        address _feeToken
    )
        public
    {
        uint256 amount = gatheredIncentiveToken[msg.sender][_feeToken];

        if (amount == 0) {
            revert NoIncentive();
        }

        delete gatheredIncentiveToken[msg.sender][_feeToken];

        emit ClaimedIncentives(//@audit QA event should be emitted in the last
            msg.sender,
            _feeToken,
            amount,
            block.timestamp
        );

        _safeTransfer(
            _feeToken,
            msg.sender,
            amount
        );
    }
```

# [NC-02] `MainHelper.sol::_cleanUp()` The function has redundant code

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L320-L322
```solidity
 unchecked {

            uint256 difference = amountContract - ( 
                totalPool + bareToken
            );

            uint256 allowedDifference = _getAllowedDifference(
                _poolToken
            );

            if (difference > allowedDifference) { //ok

                _increaseTotalAndPseudoTotalPool(
                    _poolToken,
                    allowedDifference
                );

                return;
            }
```

The unchecked block above has 

```
uint256 difference = amountContract - ( 
                totalPool + bareToken
            );
```
This is reddundant code since no need to put this into unchecked block since above _checkCleanUp() will always return if bareToken + totalPool >= amountContract and so `difference`  will always be positive and never overflow

# [NC-03] `WiseLendingDeclaration.sol::NORMALISATION_FACTOR` Natspec says `Two months in seconds`

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L307

` uint256 internal constant NORMALISATION_FACTOR = 4838400;`

The natspec for NORMALISATION_FACTOR says two months in seconds while the value is actually 56 days in total.

[NC-03] `Declaration.sol::SEQUENCER_ADDRESS` Avoid hardcoded addresses

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L79

 address public constant SEQUENCER_ADDRESS = 0xFdB631F5EE196F0ed6FAa767959853A9F217697D;