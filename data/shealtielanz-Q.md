# L1 - Using `block.timestamp` as `deadline` can be problematic.
## Summary
On the call to `_getTokensUniV3()` the swap performed in the call uses `block.stamp` as the deadline for the transaction, the issue here is that due to MEV, miners can simply hold such a transaction on the TX queue and execute it whenever it might be profitable to them, which might also in loss to the contract due to slippage issues.
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L274C1-L287C6
```solidity
    function _getTokensUniV3(uint256 _amountIn, uint256 _minOutAmount, address _tokenIn, address _tokenOut)
        internal
        returns (uint256)
    {

       // @audit deadline params.
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
## Suggestion
Allow the caller to specify a preferred deadline for the TX to be executed.

# L2 - `PendlePowerFarmLeverageLogic` doesn't adhere to `EIP-3156` specs completely
## Summary 
As per the EIP-3156 Specs
> If successful, onFlashLoan MUST return the keccak256 hash of “ERC3156FlashBorrower.onFlashLoan”.

- 
> To trust that the value of data is genuine, in addition to the check in point 1, it is recommended to verify that the initiator belongs to a group of trusted addresses. Trusting the lender and the initiator is enough to trust that the contents of data are genuine.


In contracts function, it neither returns the hash nor checks that the initiator was from a verified source though it doesn't safety checks against the consequences it must adhere to EIP-3156, if it intends to use a flash loan.
- https://eips.ethereum.org/EIPS/eip-3156


https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L59C1-L75C10
```solidity
    function receiveFlashLoan(
        IERC20[] memory _flashloanToken,
        uint256[] memory _flashloanAmounts,
        uint256[] memory _feeAmounts,
        bytes memory _userData
    )
        external
    {
        if (allowEnter == false) {
            revert AccessDenied();
        }

        allowEnter = false;

        if (_flashloanToken.length == 0) {
            revert InvalidParam();
        }
```
## Suggestion
Adhere to EIP-3156 Specs as in the link I provided above.

# L3 - Stealth donation attack is possible in `PendlePowerFarmToken`.
on the call to `withdrawExactAmount()` the shares to be burned from a user using the `previewBurnShares()` function.
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L478C2-L492C6
```solidity
    function previewBurnShares(
        uint256 _underlyingAssetAmount,
        uint256 _underlyingLpAssetsCurrent
    )
        public
        view
        returns (uint256)
    {
        uint256 product = _underlyingAssetAmount
            * totalSupply();


        return product % _underlyingLpAssetsCurrent == 0
            ? product / _underlyingLpAssetsCurrent
            : product / _underlyingLpAssetsCurrent + 1;
    }
```
It ensures that the shares to be burned are rounded up to safeguard the protocol, but this safety check can be used against it, via stealth donation attack, where the attacker can purposefully lose funds to the protocol(as it rounds up the shares and burns from the caller) to drive the price of the LP tokens up and use it as collateral in another pool.
Such an example has happened live to Wiselending before - Read up -> https://twitter.com/danielvf/status/1746303616778981402?s=46

## Suggestion.
Use a similar calculation as used in WiseLending MainHelper.sol to calculate the shares here:
```solidity
   function _calculateShares(uint256 _product, uint256 _pseudo, bool _maxSharePrice) private pure returns (uint256) {
        return _maxSharePrice == true ? _product / _pseudo + 1 : _product / _pseudo - 1;
    }
```
# L4 - Allow users to specify a `min amount out` when depositing and withdrawing from `PendlePowerFarmToken.sol`.
The functions :
- `depositExactAmount()` ~ https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L529
- `withdrawExactShares()` ~ https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L608
- `withdrawExactAmount()` ~ https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L647
Take the generic ERC-4626 style of shares minted/Burned and the amount transferred or taken.
It is best to protect the users from local attacks like Inflation/1wei/Slippage attacks where an attacker can front-run to steal the depositor's funds.

## Suggestion
Allow users to specify a minimum amount to receive from the specified functions.

# L5 - values gotten from `uniSwap::slot0()` are easily manipulated.
## Summary 
In `PendleTokenCustomOracle.sol` it uses the value gotten from `uniswap.slot0()` to derive the `latestAnswer()` the issue here is that the values gotten from the uni swap function are highly manipulatable by an attack and can be used to wreck the protocol if such a function is used to determine the price of an asset.
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PendleTokenCustomOracle.sol#L98C1-L104C45
```solidity
//@audit values gotten from uniswap slot zero are easily manipulated
    function latestAnswer() public view returns (uint256 amountOutForOneTokenIn) {
        (, int24 tick,,,,,) = IUniswapV3Pool(uniPool).slot0(); //@audit-isse

        address tokenIn = targetToken;
        uint128 amountIn = uint128(globalAmountIn);

        address tokenOut = targetToken == token1 ? token0 : token1;

        amountOutForOneTokenIn = OracleLibrary.getQuoteAtTick(tick, amountIn, tokenIn, tokenOut);
    }
```

## Suggestion
Use TWAP rather than `uniSwap.solt0()`.


# Info1 - `totalSupply()` is depreciated for curve pools.
# Info2 - `extcodesize()` check against zero isn't safe.
# Info3 - contracts that have been destroyed will pass the `token.code.length > 0` check.
# Info4 - Use a maths library when dealing with calculations.
# Info5 - Calculations using `PRECISION_FACTOR_YEAR` will not always be accurate.
# Info6 - Delete functions that you don't intend to use.
# R1 - Rounding issues arise during shares and amount calculation in `PendlePowerFarmToken`.