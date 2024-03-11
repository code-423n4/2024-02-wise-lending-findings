  1. Missing zero value check in constructor
The following functions are missing a zero valuecheck, initial total supply of tokens to be 0.1e18 and override decimals to be 0

**Occurences in:
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/Token.sol#L41
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/Token.sol#L43
```
constructor(
        uint8 _dec,
        address _user
    )
    {   decimals = _dec;
        if (_user > address(0)) {
            totalSupply = 100000000000000000 * 10 ** _dec;
            balanceOf[_user] = totalSupply;
        }
```
A zero-value check should be included to ensure the decimal value parameter is not 0. 


2. Missing input validation on array lengths
The functions below fail to perform input validation on arrays to verify the lengths match.
A mismatch could lead to an exception or undefined behavior.
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L64-L75