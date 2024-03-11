## \[N-01\] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier.

```diff
- 37:    uint256 internal constant PRECISION_FACTOR_E9 = 1E9;
- 38:    uint256 internal constant PRECISION_FACTOR_E18 = 1E18;

+ 37:    uint256 internal constant PRECISION_FACTOR_E9 = 1E9; // 1_000_000_000 
+ 38:    uint256 internal constant PRECISION_FACTOR_E18 = 1E18; //  	1_000_000_000_000_000_000
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/Declarations.sol#L37C5-L38C59

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/Declarations.sol#L118

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOracleDerivative.sol#L68

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLendingDeclaration.sol#L188

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLendingDeclaration.sol#L295C5-L296C59

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L99C4-L100C59

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L49C1-L51C59

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L206C1-L218C59

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PendleLpOracle.sol#L70

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L56C1-L57C58

## \[N-02\] Use underscores for number literals

```diff
- 112:     uint256 internal ALLOWED_DIFFERENCE = 10250;
- 121:	   uint256 internal constant GRACE_PEROID = 3600;
- 134:    uint256 internal constant ARBITRUM_CHAIN_ID = 42161;

+ 112:     uint256 internal ALLOWED_DIFFERENCE = 10_250;
+ 121:	   uint256 internal constant GRACE_PEROID = 3_600;
+ 134:    uint256 internal constant ARBITRUM_CHAIN_ID = 42_161;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/Declarations.sol#L112

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLendingDeclaration.sol#L307

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L54

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L54

## \[N‑03\] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10\*\*18)

While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist.

```
uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PendleLpOracle.sol#L67

&nbsp;

## \[N-04\] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol  
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

&nbsp;

## \[N-05\] Generate perfect code headers every time

Description:  
I recommend using header for Solidity code layout and readability

```
/*´:°•.°+.*•´.*:˚.°*.˚•´.°:°•.°•.*•´.*:˚.°*.˚•´.°:°•.°+.*•´.*:*/  
/* CONSTANTS */  
/*.•°:°.´+˚.*°.˚:*.´•*.+°.•°:´*.´•*.•°.•°:°.´:•˚°.*°.˚:*.´+°.•*/
```

## \[N-06\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-07\] NatSpec comments should be increased in contracts

> all contest

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity Official documentation

in complext project such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.  
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[S-01\] You can explain the operation of critical functions in NatSpec with an infographic.