# Lines of code

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L71


# Vulnerability details

## Impact
The incorrect logic leaves one token unattended while setting token 
## Proof of Concept
The loop below
   ![](C:\Users\USER\Pictures\setAaveTokensBulk.jpg)

 only picks i < _underlyingAssets.length and this would leave an unattended token on [line 71](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L71)

## Tools Used
Manual Analysis
## Recommended Mitigation Steps
Use i <= _underlyingAssets.length for this logic instead of i < _underlyingAssets.length 


## Assessed type

Context