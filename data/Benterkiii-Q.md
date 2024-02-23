# Critical Vulnerability in "WiseLendingDeclaration.sol" Contract

The "WiseLendingDeclaration.sol" contract contains a critical security flaw within the "setSecurity()" function. This vulnerability poses a significant risk to the integrity and functionality of the contract.
## Issue Description

When a public state variable of type interface is declared, its purpose is to hold an address adhering to the methods defined in that interface. This mechanism ensures strict adherence to a specific set of function signatures, providing a level of security and standardization.

However, a critical oversight has been identified in the "setSecurity()" function, leading to an inevitable revert condition every time the function get called.

```solidity
if (address(WISE_SECURITY) > ZERO_ADDRESS) {
    revert InvalidAction();
}
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLendingDeclaration.sol#L146

