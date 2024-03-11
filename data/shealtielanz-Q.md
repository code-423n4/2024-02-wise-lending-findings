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


In the contracts function, it neither returns the hash nor checks that the initiator was from a verified source though it doesn't safety checks against the consequences it must adhere to EIP-3156, if it intends to use a flash loan.
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


# Info1 - contracts that have been destroyed will pass the `token.code.length > 0` check.
In `CallOptionalReturn.sol` the function used by the transfer helpers `_callOptionalReturn()` check that the token actual exist with the check `token.code.length > 0`
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/CallOptionalReturn.sol#L35C1-L38C6
```solidity
    function _callOptionalReturn(address token, bytes memory data) internal returns (bool call) {
        (bool success, bytes memory returndata) = token.call(data);

        bool results = returndata.length == 0 || abi.decode(returndata, (bool));

        if (success == false) {
            revert();
        }

        call = success && results && token.code.length > 0;//@audit
    }
```

The issue here is that contracts that have been self-destroyed will pass that check, for example, Upgradeable tokens when the logic is transferred to another address, if the owner destroys the logic of the past contract, the check will still pass for such a call thereby and allowing a caller to perform malicious actions in the protocol.

## POC 
```solidity
pragma solidity ^0.8.17;

import "forge-std/Test.sol";
import "forge-std/console.sol";

contract ForloopTest is Test {

    SelfDestructExample public self;
    address payable me = payable(address(this));

    function setUp() public {
        self = new SelfDestructExample(me);
    }

    function testLoop(uint256 x) public {
        assertEq(loop._toString(x), loop.toString(x));
    }

    function testContract() public {
        bool check = activate();
        console.log("here boolie", check);
        self.destroyAndSend();
        console.log("here boolie", check);
    }

    function activate() public returns (bool) {
        address _self = address(self);
        return _self.code.length > 0;
    }
}

contract SelfDestructExample {
    address payable public beneficiary;

    constructor(address payable _beneficiary) {
        beneficiary = _beneficiary;
    }

    function destroyAndSend() public {
        require(msg.sender == beneficiary, "You are not the beneficiary");
        selfdestruct(beneficiary);
    }
}
```
# Info2 - `extcodesize()` check against zero isn't always safe.
In OracleHelper.sol the function `_checkFunctionExistence()` checks contract actual exists.
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L68C1-L76C10
```
       assembly {
            size := extcodesize(_tokenAggregator)
        }

        if (size == 0) {//@audit checks before the call
            return false;
        }

```
However, this  check is insufficient to determine if an address has existing code. According to EIP-1052 https://eips.ethereum.org/EIPS/eip-1052.
> The EXTCODEHASH of an precompiled contract is either c5d246... or 0.

# Info3 - Use a maths library when dealing with calculations.
Lots of raw maths calculations are used all over wise lending contracts, and maths in solidity can lead to certain behaviors like rounding and truncation.
Sample :
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L119C1-L129C6
```solidity
    function latestResolverTwap(address _tokenAddress) public view returns (uint256) {
        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[_tokenAddress];

        if (uniTwapPoolInfoStruct.isUniPool == true) {
            return _getTwapPrice(_tokenAddress, uniTwapPoolInfoStruct.oracle)
                / 10 ** (_decimalsWETH - decimals(_tokenAddress));
        }

        return _getTwapDerivatePrice(_tokenAddress, uniTwapPoolInfoStruct)
            / 10 ** (_decimalsWETH - decimals(_tokenAddress));
    }
```
It is best to make use of a safeMaths or decimal library to ensure against rounding and truncation errors.
# Info4 - Calculations using `PRECISION_FACTOR_YEAR` will not always be accurate.
The `PRECISION_FACTOR_YEAR` is a multiple of 1e18 and 365 days:
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L59C1-L59C87
```solidity
uint256 internal constant PRECISION_FACTOR_YEAR = PRECISION_FACTOR_E18 * ONE_YEAR;
```
The issue here is that it is used in multiple calculations in the different contracts, however, it doesn't put to context Leap years and during such a period it could affect the calculations on the contracts as leap years come and go from time to time.
# Info5 - Delete functions that you don't intend to use.
There are functions that are specified to be deleted by the protocol however such functions haven't been deleted and might lead to issues in the future.
Sample:
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/WiseOracleHub.sol#L95C1-L117C6
```solidity

    // @TODO: Delete later, keep for backward compatibility //@audit
    function getTokensInUSD(address _tokenAddress, uint256 _tokenAmount) external view returns (uint256) {
        uint8 tokenDecimals = _tokenDecimals[_tokenAddress];

        return _decimalsETH < tokenDecimals
            ? _tokenAmount * latestResolver(_tokenAddress) / 10 ** decimals(_tokenAddress)
                / 10 ** (tokenDecimals - _decimalsETH)
            : _tokenAmount * 10 ** (_decimalsETH - tokenDecimals) * latestResolver(_tokenAddress)
                / 10 ** decimals(_tokenAddress);
    }
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/WiseOracleHub.sol#L170C1-L192C6
```solidity
   // @TODO: Delete later, keep for backward compatibility//@audit
    function getTokensFromUSD(address _tokenAddress, uint256 _usdValue) external view returns (uint256) {
        uint8 tokenDecimals = _tokenDecimals[_tokenAddress];

        return _decimalsETH < tokenDecimals
            ? _usdValue * 10 ** (tokenDecimals - _decimalsETH) * 10 ** decimals(_tokenAddress)
                / latestResolver(_tokenAddress)
            : _usdValue * 10 ** decimals(_tokenAddress) / latestResolver(_tokenAddress)
                / 10 ** (_decimalsETH - tokenDecimals);
    }
```
# R1 - Rounding issues arise during shares and amount calculation in `PendlePowerFarmToken`.
refactor the `previewAmountWithdrawShares()` to ensure tightly against precision loss, so the user doesn't get a lesser amount transferred to them if the denominator is slightly bigger than the individual numerators.
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L465C1-L477C1
```solidity
   function previewAmountWithdrawShares(uint256 _shares, uint256 _underlyingLpAssetsCurrent)
        public
        view
        returns (uint256)
    {
        return (_shares * ((_underlyingLpAssetsCurrent * 1e18) / totalSupply())) / 1e18;
    }
```

