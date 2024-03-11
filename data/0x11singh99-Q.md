# Low-Severity

## [L-01] Missing address(0) check in `constructor` (**Note: Instances missed by bot-report**)

Not checking for address(0) in the constructor could lead to scenarios where the contract is deployed with invalid or uninitialized parameters, which might cause undesired behavior or even security vulnerabilities

### Check `_pendleLpOracle` and `_pendleChild` for address(0)

```solidity
File : contracts/DerivativeOracles/PendleChildLpOracle.sol

21:   constructor(
        address _pendleLpOracle,
        address _pendleChild
    )
        CustomOracleSetup()
    {
        priceFeedPendleLpOracle = IPriceFeed(
            _pendleLpOracle
        );
        pendleChildToken = IPendleChildToken(
            _pendleChild
        );
    }

```

[21-34](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L21C1-L34C1)

### Check `_priceFeedChainLinkEth` and `_oraclePendlePt` for address(0)

```solidity
File : contracts/DerivativeOracles/PendleLpOracle.sol

32:  constructor(
        address _pendleMarketAddress,
        IPriceFeed _priceFeedChainLinkEth,
        IOraclePendle _oraclePendlePt,
        string memory _oracleName,
        uint32 _twapDuration
    )

```

[32-38](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L32C4-L38C6)

### Check `_priceFeedChainLinkUsd`, `_priceFeedChainLinkEthUsd` and `_oraclePendlePt` for address(0)

```solidity
File : contracts/DerivativeOracles/PtOracleDerivative.sol

24:  constructor(
        address _pendleMarketAddress,
        IPriceFeed _priceFeedChainLinkUsd,
        IPriceFeed _priceFeedChainLinkEthUsd,
        IOraclePendle _oraclePendlePt,
        string memory _oracleName,
        uint32 _twapDuration
    )

```

[24-31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L24C5-L31C6)

### Check `_priceFeedChainLinkEth`, and `_oraclePendlePt` for address(0)

```solidity
File : contracts/DerivativeOracles/PtOraclePure.sol

24:   constructor(
        address _pendleMarketAddress,
        IPriceFeed _priceFeedChainLinkEth,
        IOraclePendle _oraclePendlePt,
        string memory _oracleName,
        uint32 _twapDuration
    )

```

[24-30](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L24C5-L30C6)

## [L-02] Check value before dividing by zero (**Note: Instances missed by analyzer-report**)

When division by zero occurs, Solidity will throw an exception, causing the transaction to revert. It's essential to include proper checks to ensure that denominators are never zero before performing division operations to prevent errors.

### Check `POW_USD_FEED * POW_ETH_USD_FEED` for `zero`

```solidity
File : contracts/DerivativeOracles/PtOracleDerivative.sol

81:   function latestAnswer()
...

122:    return uint256(answerUsdFeed)
            * PRECISION_FACTOR_E18
            / POW_USD_FEED
            * POW_ETH_USD_FEED
            / uint256(answerEthUsdFeed)
            * ptToAssetRate
            / PRECISION_FACTOR_E18;

```

[122-128](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L122C1-L128C36)

## [L-03] Unsafe casting from int256 to uint256 (**Note: Instances missed by bot-report**)

Unsafe casting from int256 to uint256 involves converting a signed integer (int256) to an unsigned integer (uint256) without proper consideration of the potential consequences. This operation can lead to unexpected behavior and errors.
If the signed integer's value is negative or larger than the maximum representable value for an unsigned integer, converting it to an unsigned integer will result in overflow.

### `answerUsdFeed` and `answerEthUsdFeed` are type of `int256`

```solidity
File : contracts/DerivativeOracles/PtOracleDerivative.sol

122:     return uint256(answerUsdFeed)
            * PRECISION_FACTOR_E18
            / POW_USD_FEED
            * POW_ETH_USD_FEED
            / uint256(answerEthUsdFeed)
            * ptToAssetRate
            / PRECISION_FACTOR_E18;

```

[122-129](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L122C1-L129C6)

## [L-04] It will return zero in everything expect answer

Since chainlink returns every param in latestRoundData but here only answer is returning correct other things will be 0 . So weak validation can be exploited or break the logic.

```solidity
File : contracts/DerivativeOracles/PendleLpOracle.sol

151:    function latestRoundData()
            external
            view
            returns (
               uint80 roundId,
               int256 answer,
               uint256 startedAt,
               uint256 updatedAt,
               uint80 answeredInRound
            )
       {
        return (
            roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answeredInRound
        );
    }

```

[151-169](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L151C5-L169C6)

```solidity
File : contracts/DerivativeOracles/PtOracleDerivative.sol

146:     function latestRoundData()
            public
            view
            returns (
               uint80 roundId,
               int256 answer,
               uint256 startedAt,
               uint256 updatedAt,
               uint80 answeredInRound
            )
       {
        return (
            roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answeredInRound
        );
    }

```

[146-164](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L146C5-L164C6)

```solidity
File : contracts/DerivativeOracles/PtOraclePure.sol

125:    function latestRoundData()
            public
            view
            returns (
                uint80 roundId,
                int256 answer,
                uint256 startedAt,
                uint256 updatedAt,
                uint80 answeredInRound
            )
        {
        return (
            roundId,
            int256(latestAnswer()),
            startedAt,
            updatedAt,
            answeredInRound
        );
    }

```

[125-143](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L125C5-L143C6)

## [L-05] Dividing before multiplying can cause loss of precisions

Dividing before multiplying (/ POW_FEED before \* ptToAssetRate) can potentially introduce loss of precision because you're dividing by a potentially large number (POW_FEED) before multiplying by another potentially large number (ptToAssetRate). To minimize the loss of precision, you might consider rearranging the operations to multiply before dividing.

```solidity
File : contracts/DerivativeOracles/PtOraclePure.sol

68:    function latestAnswer()
...
103:    return uint256(answerFeed)
            * PRECISION_FACTOR_E18
            / POW_FEED
            * ptToAssetRate
            / PRECISION_FACTOR_E18;

```

[103-107](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L103C8-L107C36)

## [L-06] Return value not checked, It can proceeds the unwanted transaction when call returns even false

The return value of the external call is not checked for its content before proceeding. This means that even if the external call fails (success == false), but the returned data is non-empty and does not represent a boolean value, the function will not revert, potentially allowing unwanted behavior to proceed. You should explicitly check the content of the returned data to ensure it represents the expected result of the external call. If the call fails or the returned data is unexpected, the function should revert to prevent unintended behavior or vulnerabilities.

```solidity
File : contracts/TransferHub/ApprovalHelper.sol

13:    function _safeApprove(
           address _token,
           address _spender,
           uint256 _value
        )
         internal
    {
20:     _callOptionalReturn(
            _token,
            abi.encodeWithSelector(
                IERC20.approve.selector,
                _spender,
                _value
            )
        );
    }

```

[13-28](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/ApprovalHelper.sol#L13C4-L28C6)

```solidity
File : contracts/TransferHub/TransferHelper.sol

13:     function _safeTransfer(
            address _token,
            address _to,
            uint256 _value
        )
           internal
      {
20:       _callOptionalReturn(
              _token,
              abi.encodeWithSelector(
                 IERC20.transfer.selector,
                 _to,
                 _value
               )
            );
       }

```

[13-28](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol#L13C5-L28C6)

```solidity
File : contracts/TransferHub/TransferHelper.sol

34:     function _safeTransferFrom(
            address _token,
            address _from,
            address _to,
            uint256 _value
        )
           internal
     {
42:        _callOptionalReturn(
                _token,
                abi.encodeWithSelector(
                  IERC20.transferFrom.selector,
                  _from,
                  _to,
                  _value
                )
            );
    }

```

[34-51](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol#L34C3-L51C6)

## [L-07] `nonReentrant` will not work in `depositExactAmount` function

How nonReentrant is implemented in this protocol. It can only work when sending ether using their \_sendValue function. And on other function this modifier will not work as normally mutex lock works. So use proper Openzeppelin ReentrancyGuard or implement like that it's nonReentrant with extra protocol functionality if need to be added

````solidity
File : contracts/WrapperHub/AaveHelper.sol

   modifier nonReentrant() {
        _nonReentrantCheck();
        _;
  //@audit no mutex lock
        ...


      function _nonReentrantCheck() internal view {
        if (sendingProgressAaveHub == true) {
            revert InvalidAction();
        }

        if (WISE_LENDING.sendingProgress() == true) {
            revert InvalidAction();
        }
    }
    }
    ```

```solidity
File : contracts/WrapperHub/AaveHub.sol

122:  function depositExactAmount(
        uint256 _nftId,
        address _underlyingAsset,
        uint256 _amount
    )
        public
128:    nonReentrant
        validToken(_underlyingAsset)
        returns (uint256)

````

[122-130](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L122C1-L130C26)

```solidity
File : contracts/WrapperHub/AaveHub.sol

172:  function depositExactAmountETH(
         uint256 _nftId
     )
        public
        payable
177:    nonReentrant
        returns (uint256)

```

[172-178](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L172C1-L178C26)

## [L-08] Using deadline `block.timestamp` can be executed whenever miner wants so use custom deadline

Using block.timestamp as the deadline in a transaction can indeed introduce vulnerabilities because miners have some control over the timestamp included in the block they are mining. This allows miners to manipulate the timestamp to some extent, potentially allowing them to include transactions with expired deadlines. Miners can manipulate the timestamp to include transactions with expired deadlines. By adjusting the timestamp slightly.

To mitigate these risks, it's generally recommended to use a custom deadline based on a timestamp that is set by the contract or the user. This ensures that the deadline is not reliant on the current block's timestamp, which can be manipulated by miners. The custom deadline should be set to a future timestamp that allows sufficient time for the transaction to be processed.

```solidity
File : contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

264:    function _getTokensUniV3(
...

273:     return UNISWAP_V3_ROUTER.exactInputSingle(
            IUniswapV3.ExactInputSingleParams(
                {
                    tokenIn: _tokenIn,
                    tokenOut: _tokenOut,
                    fee: UNISWAP_V3_FEE,
                    recipient: address(this),
280:                deadline: block.timestamp,
                    amountIn: _amountIn,
                    amountOutMinimum: _minOutAmount,
                    sqrtPriceLimitX96: 0
                }
            )
        );

```

[273-286](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L273C8-L286C11)

# Non-Critical

## [N-01] `public` functions not called by the contract should be declared `external` instead (**Note: Instances missed by bot-report**)

```solidity
File : contracts/FeeManager/FeeManagerHelper.sol

270:     function updatePositionCurrentBadDebt(

```

[270-274](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L270C1-L274C6)

```solidity
File : contracts/WiseOracleHub/OracleHelper.sol

265:    function getETHPriceInUSD()

521:    function sequencerIsDead()

```

[265-269](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L265C1-L269C6), [521-525](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L521C1-L525C6)

```solidity
File : contracts/WiseSecurity/WiseSecurityHelper.sol

15:     function overallETHCollateralsBoth(

394:    function overallETHBorrowHeartbeat(

672:    function checkTokenAllowed(

687:    function checkHeartbeat(

837:    function checksRegister(

859:    function canLiquidate(

876:    function checkMaxShares(

999:    function getLendingRate(


```

[15-21](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L15C1-L21C6), [394-400](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L394C1-L400C6), [672-677](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L672C1-L677C6), [687-693](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L687C1-L693C6), [837-843](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L837C1-L843C6), [859-865](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L859C1-L865C6), [876-885](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L876C1-L885C6), [999-1005](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L999C3-L1005C6)

## [N-02] Writing `CustomOracleSetup` is not necessary

The `PendleChildLpOracle` contract inherits from `CustomOracleSetup`. If `CustomOracleSetup` does not have any constructor parameters, you don't need to write a constructor in `PendleChildLpOracle` to explicitly call the constructor of `CustomOracleSetup`. Solidity will handle this automatically for you.

```solidity
File : contracts/DerivativeOracles/PendleChildLpOracle.sol

13:   contract PendleChildLpOracle is CustomOracleSetup  {

```

[13](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L13)

## [N-03] Do not implicit return if explicitly returned below (**Note: Instances missed by bot report**)

```solidity
File : contracts/DerivativeOracles/PendleChildLpOracle.sol

78:     function getRoundData(

83:     returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )

```

[83-89](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L83C8-L89C10)

## [N-04] Use one variable instead of using two variables for holding same value

```solidity
File : contracts/DerivativeOracles/PendleLpOracle.sol

67:    uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;

       // Precision factor for computations.
70:    uint256 internal constant PRECISION_FACTOR_E18 = 1E18;

```

[67-70](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L67C5-L70C59)

## [N-05] Make contract `abstract` if not deployed separately

```solidity
File : contracts/FeeManager/DeclarationsFeeManager.sol

contract DeclarationsFeeManager is FeeManagerEvents, OwnableMaster {

```

[31](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L31)

## [N-06] Typos

`addresses` spelling wrong can confuse or create errors

```diff
File : contracts/FeeManager/FeeManager.sol

-884:    function getPoolTokenAdressesByIndex(
+884:    function getPoolTokenAddressesByIndex(

```

[884](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L884)

## [N-07] Comment saying function `internal` but it is `public`

```solidity
File : contracts/FeeManager/FeeManagerHelper.sol

240:    /**
241:    * @dev Internal function calculating receive amount for the caller.
        * paybackIncentive is set to 5E16 => 5% incentive for paying back bad debt.
        */
        function getReceivingToken(
           address _paybackToken,
           address _receivingToken,
           uint256 _paybackAmount
        )
249:    public
        view
        returns (uint256 receivingAmount)
    {

```

[240-252](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L240C5-L252C6)

## [N-08] The error message `AmountTooSmall()` is updated to `AmountTooBig()` to indicate that the amount requested for sending is larger than the available balance

If the balance of the contract is less than the amount to be sent (\_amount), the transaction will revert with the error `AmountTooBig()`.
This provides clarity in the function's logic and helps developers understand the reason for the revert in case the condition is not met.

```diff
File : contracts/TransferHub/SendValueHelper.sol

12: function _sendValue(
        address _recipient,
        uint256 _amount
    )
        internal
    {
        if (address(this).balance < _amount) {
-            revert AmountTooSmall();
+            revert AmountTooBig();
        }

```

[12-20](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L12C1-L20C10)

## [N-09] Change the function name from `proposeOwner` to `proposeMaster` (**Note: Instances missed by bot report**)

Changing the function name from `proposeOwner` to `proposeMaster` could be done to better reflect the role or responsibility associated with the action performed by the function.

```solidity
File : contracts/OwnableMaster.sol

70:  function proposeOwner(
        address _proposedOwner
    )
        external
        onlyMaster

```

[70-74](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L70C1-L74C19)

## [N-10] `Increment` should happen at the end of the `loop`

Moving the increment operation (++i) to the end of the loop instead of at the beginning ensures that the loop control variable (i) is updated after each iteration of the loop completes. This change affects the behavior of the loop and ensures that the increment operation happens after the loop body executes.

```solidity
File : contracts/WrapperHub/AaveHelper.sol

254:    for (i; i < l;) {

            address currentAddress = WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            );

261:        unchecked {
262:               ++i;
263:         }

            if (currentAddress == _poolToken) {
                continue;
            }

            WISE_LENDING.preparePool(
                currentAddress
            );

            WISE_LENDING.newBorrowRate(
                _poolToken
            );
        }

```

[254-276](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L254C9-L276C10)

```solidity
File : contracts/WrapperHub/AaveHelper.sol

290:    for (i; i < l;) {

            address currentAddress = WISE_LENDING.getPositionBorrowTokenByIndex(
                _nftId,
                i
            );

297:        unchecked {
298:            ++i;
299:        }

            if (currentAddress == _poolToken) {
                continue;
            }

            WISE_LENDING.preparePool(
                currentAddress
            );

            WISE_LENDING.newBorrowRate(
                _poolToken
            );
        }

```

[290-312](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L290C9-L312C10)
