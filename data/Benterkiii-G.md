# Using immutable variable can save gas

variables that's been set in the constructor and remain unchanged throughout their lifecycle should be immutable.
###### 1 instance:

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLendingDeclaration.sol#L162


# Avoid Making any state reads if we can revert early (Save 2100 Gas) 

##### 8 instances of this issue.
###### 4 instances in "PoolManager.setParamsLASA()" function:

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L48
```solidity
        _validateParameter(
            _upperBoundMaxRate,
            UPPER_BOUND_MAX_RATE
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L53
```solidity
        _validateParameter(
            _lowerBoundMaxRate,
            LOWER_BOUND_MAX_RATE
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L58
```solidity
        _validateParameter(
            _lowerBoundMaxRate,
            _upperBoundMaxRate
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L83
```solidity
        _validateParameter(
            _poolMulFactor,
            PRECISION_FACTOR_E18
        );
```
###### 4 instances in "WiseSecurityDeclarations._setLiquidationSettings()" function:

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L154
```solidity
        if (_newMaxFeeETH > maxFee) {
            revert MaxFeeEthTooHigh();
        }
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L158
```solidity
        if (_newMaxFeeETH < minFee) {
            revert MaxFeeEthTooLow();
        }
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L172
```solidity
        if (_newMaxFeeFarmETH > maxFeeFarm) {
            revert MaxFeeFarmEthTooHigh();
        }
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L176
```solidity
        if (_newMaxFeeFarmETH < minFeeFarm) {
            revert MaxFeeFarmEthTooLow();
        }
```