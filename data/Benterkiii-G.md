# Using immutable variable can save gas

variables that's been set in the constructor and remain unchanged throughout their lifecycle should be immutable.
###### 1 instance:

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLendingDeclaration.sol#L162


# Checking earlier could save gas in case the function reverts 

###### 4 instances in "setParamsLASA()" function:

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
