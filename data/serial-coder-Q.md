# [L-01] Incorrect `collateralFactor` upper limit

## Impact

In the `PendlePowerFarmDeclarations::constructor()`, the `_collateralFactor` parameter is validated against the incorrect upper limit constant. The `PRECISION_FACTOR_E18`, which indicates `1e18` or `100%`, is used instead of the `MAX_COLLATERAL_FACTOR`, which indicates `95e16` or `95%`.

Therefore, the `_collateralFactor` parameter can be greater than the expected upper limit value, leading to the incorrect configuration of the protocol's parameter.

## Proof of Concept

The [`MAX_COLLATERAL_FACTOR` constant](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L62) is defined as `95%` maximum (`95e16`).

However, in the `PendlePowerFarmDeclarations::constructor()`, the [`PRECISION_FACTOR_E18`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L243) (at `100%`) is used as the upper limit instead.

```solidity
    contract PendlePowerFarmDeclarations is
        WrapperHelper,
        TransferHelper,
        ApprovalHelper,
        SendValueHelper
    {
        ...

        // RESTRICTION VALUES

@1      uint256 internal constant MAX_COLLATERAL_FACTOR = 95 * PRECISION_FACTOR_E16; //@audit -- The MAX_COLLATERAL_FACTOR is defined at 95% maximum

        ...

        constructor(
            address _wiseLendingAddress,
            address _pendleChilTokenAddress,
            address _pendleRouter,
            address _entryAsset,
            address _pendleSy,
            address _underlyingMarket,
            address _routerStatic,
            address _dexAddress,
            uint256 _collateralFactor
        )
            WrapperHelper(
                IWiseLending(
                    _wiseLendingAddress
                ).WETH_ADDRESS()
            )
        {
            ...

@2          if (_collateralFactor > PRECISION_FACTOR_E18) { //@audit -- In the constructor(), the PRECISION_FACTOR_E18 (at 100%) is used as the maximum value instead of the MAX_COLLATERAL_FACTOR
                revert CollateralFactorTooHigh();
            }

            collateralFactor = _collateralFactor;

            ...
        }

        ...
    }
```

- `@1 -- The MAX_COLLATERAL_FACTOR is defined at 95% maximum`: https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L62

- `@2 -- In the constructor(), the PRECISION_FACTOR_E18 (at 100%) is used as the maximum value instead of the MAX_COLLATERAL_FACTOR`: https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L243

## Tools Used

Manual Review

## Recommended Mitigation Steps

Use the `MAX_COLLATERAL_FACTOR` instead of the `PRECISION_FACTOR_E18`.

```diff
    constructor(
        address _wiseLendingAddress,
        address _pendleChilTokenAddress,
        address _pendleRouter,
        address _entryAsset,
        address _pendleSy,
        address _underlyingMarket,
        address _routerStatic,
        address _dexAddress,
        uint256 _collateralFactor
    )
        WrapperHelper(
            IWiseLending(
                _wiseLendingAddress
            ).WETH_ADDRESS()
        )
    {
        ...

-       if (_collateralFactor > PRECISION_FACTOR_E18) {
+       if (_collateralFactor > MAX_COLLATERAL_FACTOR) {
            revert CollateralFactorTooHigh();
        }

        collateralFactor = _collateralFactor;

        ...
    }
```

---

# [L-02] Inconsistent `MAX_COLLATERAL_FACTOR` constants

## Impact

There are different `MAX_COLLATERAL_FACTOR` constants defined in the `PendlePowerFarmDeclarations` and `WiseLendingDeclaration` contracts that can lead to an inconsistency issue.

## Proof of Concept

- The `MAX_COLLATERAL_FACTOR` constant of the `PendlePowerFarmDeclarations` contract is [`95e16` (`95%`)](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L62).

- The `MAX_COLLATERAL_FACTOR` constant of the `WiseLendingDeclaration` contract is [`85e16` (`85%`)](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLendingDeclaration.sol#L319).

## Tools Used

Manual Review

## Recommended Mitigation Steps

Make both constants to be consistent.

---

# [L-03] Use of unchangeable oracles

## Impact

In the `Wise Lending` protocol, Chainlink's `price feed,` Chainlink's `aggregator`, and `TWAP oracle` for each specific token cannot be changed or updated. If one is down or offline, the protocol will be dos'ed. 

Another risk is that if an admin somehow misconfigures oracle data, there will be no functionality to update them.

## Proof of Concept

- The [`OracleHelper::_addOracle()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L19-L21) will revert the transaction if the `price feed` is already set.
- The [`OracleHelper::_addAggregator()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L41-L43) will revert the transaction if the `aggregator` is already set.
- The [`OracleHelper::_validateTwapOracle()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L365-L367) will revert the transaction if the `TWAP oracle` (for a specific token) is already set.

```solidity
    function _addOracle(
        address _tokenAddress,
        IPriceFeed _priceFeedAddress,
        address[] calldata _underlyingFeedTokens
    )
        internal
    {
@1      if (priceFeed[_tokenAddress] > ZERO_FEED) { //@audit -- Chainlink's price feed cannot be changed/updated
@1          revert OracleAlreadySet();
@1      }

        priceFeed[_tokenAddress] = _priceFeedAddress;

        _tokenDecimals[_tokenAddress] = IERC20(
            _tokenAddress
        ).decimals();

        underlyingFeedTokens[_tokenAddress] = _underlyingFeedTokens;
    }

    ...

    function _addAggregator(
        address _tokenAddress
    )
        internal
    {
        IAggregator tokenAggregator = IAggregator(
            priceFeed[_tokenAddress].aggregator()
        );

@2      if (tokenAggregatorFromTokenAddress[_tokenAddress] > ZERO_AGGREGATOR) { //@audit -- Chainlink's aggregator cannot be changed/updated
@2          revert AggregatorAlreadySet();
@2      }

        if (_checkFunctionExistence(address(tokenAggregator)) == false) {
            revert FunctionDoesntExist();
        }

        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[
            _tokenAddress
        ];

        if (uniTwapPoolInfoStruct.oracle > ZERO_ADDRESS) {
            revert AggregatorNotNecessary();
        }

        tokenAggregatorFromTokenAddress[_tokenAddress] = tokenAggregator;
    }

    ...

    function _validateTwapOracle(
        address _tokenAddress
    )
        internal
        view
    {
@3      if (uniTwapPoolInfo[_tokenAddress].oracle > ZERO_ADDRESS) { //@audit -- TWAP oracle for each specific token cannot be changed/updated
@3          revert TwapOracleAlreadySet();
@3      }
    }
```

- `@1 -- Chainlink's price feed cannot be changed/updated`: https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L19-L21

- `@2 -- Chainlink's aggregator cannot be changed/updated`: https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L41-L43

- `@3 -- TWAP oracle for each specific token cannot be changed/updated`: https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L365-L367

## Tools Used

Manual Review

## Recommended Mitigation Steps

Make the oracles to be updatable.

---

# [L-04] Unhandled Chainlink revert can deny access to oracle prices

## Impact

The protocol lacks a preventive approach to handling the `Chainlink`'s `latestRoundData()` reverts, resulting in a denial of service to accessing oracle prices.

The following is a complete list of the functions that execute the `latestRoundData()`.
- [`OracleHelper::_getChainlinkAnswer()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L253-L258)
- [`OracleHelper::sequencerIsDead()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L532-L535)
- [`OracleHelper::getLatestRoundId()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L708-L714)
- [`PendleLpOracle::latestAnswer()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PendleLpOracle.sol#L88-L93)
- [`PtOracleDerivative::latestAnswer()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOracleDerivative.sol#L86-L91) #1
- [`PtOracleDerivative::latestAnswer()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOracleDerivative.sol#L93-L98) #2
- [`PtOraclePure::latestAnswer()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOraclePure.sol#L73-L79)

## Proof of Concept

For example, the `OracleHelper::_getChainlinkAnswer()` uses `Chainlink`'s `latestRoundData()`. The calls to the `latestRoundData()` can be reverted for several reasons, such as [`Chainlink`'s multisigs block access to price feeds](https://blog.openzeppelin.com/secure-smart-contract-guidelines-the-dangers-of-price-oracles), etc.

However, there is no preventive approach to handling the case of the `latestRoundData()` reverts, resulting in a denial of service when accessing oracle prices.

```solidity
    function _getChainlinkAnswer(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
    {
@>      (
@>          ,
@>          int256 answer,
@>          ,
@>          ,
@>      ) = priceFeed[_tokenAddress].latestRoundData();

        return uint256(
            answer
        );
    }
```

- `OracleHelper::_getChainlinkAnswer()`: https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L253-L258

## Tools Used

Manual Review

## Recommended Mitigation Steps

Wrap the call to the `latestRoundData()` in a try/catch block and handle any errors appropriately—for instance, fallback to `Uniswap`'s TWAP oracle or other off-chain oracle services such as `Tellor` or `Band`. The fallback solution will also ensure the price data is available for the protocol.

```diff
    function _getChainlinkAnswer(
        address _tokenAddress
    )
        internal
        view
        returns (uint256)
    {
-       (
-           ,
-           int256 answer,
-           ,
-           ,
-       ) = priceFeed[_tokenAddress].latestRoundData();
-
-       return uint256(
-           answer
-       );
+       try priceFeed[_tokenAddress].latestRoundData() returns (
+           ,
+           int256 answer,
+           ,
+           ,
+       ) {
+           // Handle the successful execution here
+           ...
+
+           return answer;
+
+       } catch Error(string memory) {
+           // Handle errors here: (e.g., fallback to Uniswap's TWAP oracle or other off-chain oracle services such as Tellor or Band)
+           ...
+       }
    }
```