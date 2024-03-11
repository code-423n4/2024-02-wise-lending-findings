## G-01 Prevent no-ops in revokeBeneficial and setBeneficial functions to save gas

If these functions are called with an empty _feeTokens array, the contract will still execute the loop's logic even though it's a no-op. This would result in wasted gas for the caller. 


```solidity
File: FeeManager.sol

function setBeneficial(
        address _user,
        address[] calldata _feeTokens
    )
        external
        onlyMaster
    {
        uint256 i;
        uint256 l = _feeTokens.length;

        while (i < l) {
            _setAllowedTokens(
                _user,
                _feeTokens[i],
                true
            );

            unchecked {
                ++i;
            }
        }
```
### LOC
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L538
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L538

### Recommendation
Check if _feeTokens is a non-empty array in revokeBeneficial and setBeneficial to save some gas.

## G-02 Pagination Approach for Bulk settings to save gas

Instead of performing all operations within a single transaction, consider implementing a paginated approach or ensuring the arrays passed to these functions are kept to a manageable size.

### LOC

- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L82

## G-03 Function calls in loops can be gas intensive in claimIncentivesBulk function

```solidity
File: FeeManager

while (i < l) {

            tokenAddress = poolTokenAddresses[i];

            if (isAaveToken[tokenAddress] == true) {
                tokenAddress = underlyingToken[
                    tokenAddress
                ];
            }

            claimIncentives(
                tokenAddress
            );

            unchecked {
                ++i;
            }
        }

```

### LOC
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L257C8-L273C14

## G-04 Compact Data Types of Function Arguments to save gas in MainHelper.sol

Ensure that the data types for the arguments (_nftId, _nftIdLiquidator) are the smallest necessary to hold the values expected.

```solidity
File: MainHelper.sol

function _checkLiquidatorNft(
        uint256 _nftId,
        uint256 _nftIdLiquidator
    )
        internal
        view
```

### LOC

- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L258C6-L261C6

## G-05 Reorder Condition Checks in _validateIsolationPoolLiquidation function

Conditions that are cheaper to check and more likely to fail should be placed earlier. For example checking ownership via POSITION_NFT.ownerOf(_nftId) != _caller is less expensive and more likely to fail than _onlyIsolationPool(_caller) or _checkLiquidatorNft(_nftId, _nftIdLiquidator)

```solidity

File: MainHelper.sol

function _validateIsolationPoolLiquidation(
        address _caller,
        uint256 _nftId,
        uint256 _nftIdLiquidator
    )
        internal
        view
    {
        _onlyIsolationPool(
            _caller
        );

        if (positionLocked[_nftId] == false) {
            revert NotPowerFarm();
        }

        _checkLiquidatorNft(
            _nftId,
            _nftIdLiquidator
        );

        if (POSITION_NFT.ownerOf(_nftId) != _caller) {
            revert InvalidCaller();
        }
    }
```