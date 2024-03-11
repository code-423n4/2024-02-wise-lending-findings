# \[G-01\] Use constants instead of type(uintx).max

it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:

```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```

In the above example, we have a contract with a constant MAX\_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX\_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```
39:     uint256 internal constant MAX_AMOUNT = type(uint256).max;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/Declarations.sol#L39

```
uint256 internal constant MAX_AMOUNT = type(uint256).max;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L98

```
uint256 internal constant UINT256_MAX = type(uint256).max;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L209

## G-02\] uint8 is not always cheaper than uint256

The EVM only operates on 32 bytes/ 256 bits at a time. This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.

```
85: uint8 internal immutable _decimalsWETH;

97: uint8 internal constant _decimalsUSD = 8;  

100: uint8 internal constant _decimalsETH = 18;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/Declarations.sol#L85

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOracleDerivative.sol#L65

## \[G-03\]  `10 ** 18` can be changed to `1e18` and save some gas

`10 ** 18` can be changed to `1e18` to avoid unnecessary arithmetic operation and save gas.

```
uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PendleLpOracle.sol#L67

## \[G-04\] Duplicated require()/if() checks should be refactored to a modifier or function

to reduce code duplication and improve readability. • Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check. • A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```
453: if (len == 0) {
            return 0;
        }
        
510: if (len == 0) {
            return 0;
        }

652: if (len == 0) {
            return 0;
        }
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L453

## \[G-05\] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```
uint32 internal constant SECONDS_IN_MINUTE = 60;

    // Define TWAP period in seconds.
uint32 internal constant TWAP_PERIOD = 30 * SECONDS_IN_MINUTE;
```

&nbsp;https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/Declarations.sol#L109

```
uint256 internal constant PRECISION_FACTOR_E16 = 1E16;
    uint256 internal constant PRECISION_FACTOR_E18 = 1E18;
    uint256 internal constant PRECISION_FACTOR_E36 = PRECISION_FACTOR_E18 * PRECISION_FACTOR_E18;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLendingDeclaration.sol#L295C1-L297C98

## \[G-06\] Amounts should be checked for 0 before calling a transfer

It is considered a best practice to verify for zero values before executing any transfers within smart contract functions. This approach helps prevent unnecessary external calls and can effectively reduce gas costs.

This practice is particularly crucial when dealing with the transfer of tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can determine whether a value is zero by utilizing the == operator. Here's an example demonstrating how to check for a zero value before performing a transfer:

```
//example 
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

&nbsp;

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L626

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L463

## \[G-07\] Use bytes32 rather than string/bytes.

If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size.

```
74:     string public name;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOracleDerivative.sol#L74

```
76:   string public name;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PendleLpOracle.sol#L76

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOraclePure.sol#L60

## <ins>\[G-08\] Events are not indexed</ins>

The emitted events are not indexed, making off-chain scripts such as front-ends of dApps difficult to filter the events efficiently.

Recommend adding the `indexed` keyword in each event,

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveEvents.sol