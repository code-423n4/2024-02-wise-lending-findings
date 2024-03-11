
# Low Risk

| Number | Issue | Instences |
|--------|-------|-----------|
|[L-01]| Loss of precision | 15 |
|[L-02]| Use forceApprove in place of approve | 1 |
|[L-03]| ERC-20: Large transfers may revertf | 6 |
|[L-04]| approve()/safeApprove() may revert if the current approval is not zero | 2 |
|[L-05]| File allows a version of solidity that is susceptible to .selector-related optimizer bug | 1 |
|[L-06]| onlyOwner functions not accessible if owner renounces ownership | 4 |
|[L-07]| Consider disallowing minting/transfers to address(this) | 18 |

## [L-01] Loss of precision

Dividing by large numbers in Solidity can cause a loss of precision due to the language's inherent integer division behavior. Solidity does not support floating-point arithmetic, and as a result, division between integers yields an integer result, truncating any fractional part. When dividing by a large number, the resulting value may become significantly smaller, leading to a loss of precision, as the fractional part is discarded.

```solidity
file: blob/main/contracts/MainHelper.sol

177   return PRECISION_FACTOR_E18 - (PRECISION_FACTOR_E18
            * totalPool
            / pseudoPool
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L177-L180

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

84   return getPositionBorrowETH(_nftId)
            * PRECISION_FACTOR_E18
            / totalCollateral;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L84-L86

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

363  uint256 newUtilization = PRECISION_FACTOR_E18 - (PRECISION_FACTOR_E18
            * (totalPool - _borrowAmount)
            / pseudoPool
        );

379   return mulFactor
            * PRECISION_FACTOR_E18
            * newUtilization
            / baseDivider;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L363-L366

```solidity
file: blob/main/contracts/WiseOracleHub/OracleHelper.sol
 
434   int24 tick = int24(
            tickCumulativesDelta
            / twapPeriodInt56
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L434-L437


```solidity
file: blob/main/contracts/WiseSecurity/WiseSecurity.sol

74    return overallETHBorrow(_nftId)
            * PRECISION_FACTOR_E18
            / overallCollateral;

491   return weightedRate
            / overallETH

548    return weightedRate
            / overallETH;

625   netAPY = (ethValueGain - ethValueDebt)
                / totalETHSupply;

631    netAPY = (ethValueDebt - ethValueGain)
                / totalETHSupply;

933   return borrowShares
            * updatedPseudo
            / currentTotalBorrowShares;

970   return lendingShares
            * updatedPseudo
            / currentTotalLendingShares;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L74-L76

```solidity
file: blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol

308    uint256 updatedToken = lendingShares
            * updatedPseudo
            / currentTotalLendingShares;

637    uint256 updatedToken = borrowShares
            * updatesPseudo
            / currentTotalBorrowShares;

1018   return adjustedRate
            * WISE_LENDING.getPseudoTotalBorrowAmount(_poolToken)
            / pseudoTotalPool;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L308-L310

## [L-02] Use forceApprove in place of approve

The forceApprove function is a specialized solution addressing the issues that arise with non-standard ERC-20 tokens, such as USDT, which exhibit non-standard behavior in their approve function. These tokens may necessitate setting the allowance to 0 before changing it, may not return a boolean value from approve calls, or may return false instead of reverting on failure. OpenZeppelin's SafeERC20 library includes a forceApprove method to safely handle these peculiarities, ensuring that smart contracts can interact with such tokens without transaction failures. It rectifies inconsistencies by ensuring that all token interactions, including approvals, adhere to expected standards, even when the underlying tokens do not.

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

197  POSITION_NFT.approve(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L197

## [L-03] ERC-20: Large transfers may revertf

Some IERC20 implementations (e.g UNI, COMP) may fail if the valued transferred is larger than uint96 source(https://github.com/d-xo/weird-erc20/blob/main/src/Uint96.sol)


```solidity
file: blob/main/contracts/WiseLending.sol

476  _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        );

622    _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        );

1190    _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        )

1230  _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            repaymentAmount
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L476-L481

```solidity
file: blob/main/contracts/PositionNFTs.sol

83  _transfer(
            address(this),
            _feeManagerContract,
            FEE_MANAGER_NFT
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L83-L87

```solidity
file: blob/main/contracts/WiseCore.sol
 
674    _safeTransferFrom(
            _data.tokenToPayback,
            _data.caller,
            address(this),
            _data.paybackAmount
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L674-L680

## [L-04] approve()/safeApprove() may revert if the current approval is not zero

Calling approve() without first calling approve(0) if the current approval is non-zero will revert with some tokens, such as Tether (USDT). While Tether is known to do this, it applies to other tokens as well, which are trying to protect against this attack vector(https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit?pli=1#heading=h.m9fhqynw2xvt). safeApprove() itself also implements this protection.

Always reset the approval to zero before changing it to a new value (OpenZeppelin's SafeERC20.forceApprove() does this for you), or use safeIncreaseAllowance()/safeDecreaseAllowance()

```solidity
file: blob/main/contracts/WiseSecurity/WiseSecurity.sol

156   _safeApprove(
            ICurve(curvePool).coins(tokenIndexForApprove),
            curvePool,
            0
        );

176   _safeApprove(
            ICurve(curveMetaPool).coins(tokenIndexForApprove),
            curveMetaPool,
            0
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L156-L110


## [L-05] File allows a version of solidity that is susceptible to .selector-related optimizer bug

In solidity versions prior to 0.8.21, there is a legacy code generation bug where if foo().selector is called, foo() doesn't actually get evaluated. It is listed as low-severity, because projects usually use the contract name rather than a function call to look up the selector.

```solidity
file:  blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

176   return this.onERC721Received.selector;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L176

## [L-06] onlyOwner functions not accessible if owner renounces ownership

The owner is able to perform certain privileged activities, but it's possible to set the owner to address(0). This can represent a certain risk if the ownership is renounced for any other reason than by design.

Renouncing ownership will leave the contract without an owner, therefore limiting any functionality that needs authority.



```solidity
file: blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol

26   function renounceOwnership()
        external
        onlyOwner
    {

33  function setLastUpdateGlobal(
        uint256 _time
    )
        external
        onlyOwner
    {

42  function setRoundData(
        uint80 _roundId,
        uint256 _updateTime
    )
        external
        onlyOwner
    {

52  function setGlobalAggregatorRoundId(
        uint80 _aggregatorRoundId
    )
        external
        onlyOwner
    {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L26-L29

## [L-07] Consider disallowing minting/transfers to address(this)

A tranfer to the token contract itself is unlikely to be correct and more likely to be a common user error due to a copy & paste mistake. Proceeding with such a transfer will result in the permanent loss of user tokens.

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

136   _safeTransferFrom(
            poolAddress,
            msg.sender,
            address(this),
            paybackAmount
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L136-L141

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

109   _safeTransferFrom(
            WETH_ADDRESS,
            msg.sender,
            address(this),
            _amount
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L109-L114


```solidity
file: blob/main/contracts/WrapperHub/AaveHub.sol

132  _safeTransferFrom(
            _underlyingAsset,
            msg.sender,
            address(this),
            _amount
        );

451  _safeTransferFrom(
            _underlyingAsset,
            msg.sender,
            address(this),
            _paybackAmount
        );

583  _safeTransferFrom(
            _underlyingAsset,
            msg.sender,
            address(this),
            paybackAmount
        );


```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L132-L137

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

86  _safeTransferFrom(
            _pendleMarket,
            msg.sender,
            address(this),
            sendingAmount
        );

397   _safeTransferFrom(
                    PENDLE_TOKEN_ADDRESS,
                    msg.sender,
                    address(this),
                    _amount
                );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L86-L91

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

86  _safeTransferFrom(
            _pendleMarket,
            msg.sender,
            address(this),
            sendingAmount
        );

142  _safeTransferFrom(
            PENDLE_TOKEN_ADDRESS,
            msg.sender,
            address(this),
            tokenAmountSend
        );

397  _safeTransferFrom(
                    PENDLE_TOKEN_ADDRESS,
                    msg.sender,
                    address(this),
                    _amount
                );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L86-L91


```solidity
file: blob/main/contracts/PositionNFTs.sol

83   _transfer(
            address(this),
            _feeManagerContract,
            FEE_MANAGER_NFT
        );
    }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L83-L88

```solidity
file: blob/main/contracts/WiseLending.sol

476   _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        );

622  _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        );

738  _safeTransfer(
            _poolToken,
            msg.sender,
            _withdrawAmount
        );
    
889   _safeTransfer(
            _poolToken,
            msg.sender,
            _withdrawAmount
        );

926    _safeTransfer(
            _poolToken,
            msg.sender,
            withdrawAmount
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L476-L481


```solidity
file: blob/main/contracts/FeeManager/FeeManager.sol

788  _safeTransferFrom(
            _paybackToken,
            msg.sender,
            address(WISE_LENDING),
            paybackAmount
        );

860    _safeTransferFrom(
            _paybackToken,
            msg.sender,
            address(WISE_LENDING),
            paybackAmount
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L788-L793