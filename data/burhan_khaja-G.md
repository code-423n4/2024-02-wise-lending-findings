| ID     | Title                                                                                                            
|--------|-------------------------------------------------------------------------
| G-01   |Using bytes32 instead of string for strings with expected length < 32 bytes saves gas                                                                          
| G-02   |Inlining onlyproposed modifier will save gas as it is used only once, (not in 4nalyser or bot-report)
| 3      |Making Admin Functions payable saves both execution and deployment gas costs ,without risking user experience
| 4      |                                         
| 5      |           
| 6      |                                                                               
| 7      |                                                                                    
| 8      |   


## G-01 - Using bytes32 instead of string for strings with expected length < 32 bytes saves gas
If the length of a string is less than 32 bytes, it’s always efficient to store it in a bytes32 variable and use assembly to use it when needed.

Because In Solidity, strings are variable length dynamic data types, meaning their length can change and grow as needed.

If the length is 32 bytes or longer, the slot in which they are defined stores the length of the string * 2 + 1, while their actual data is stored elsewhere (the keccak hash of that slot).

However, if a string is less than 32 bytes, the length * 2 is stored at the least significant byte of it’s storage slot and the actual data of the string is stored starting from the most significant byte in the slot in which it is defined.

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOraclePure.sol#L60
```diff
- string public name;
+ bytes32 public name;
```
similarly, [1](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOracleDerivative.sol#L74) [2](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L14) [3](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L27)

## G-02 - Inlining onlyproposed modifier will save gas as it is used only once, (not in 4nalyser or bot-report)

please note that no public known issues contain this, the funny part is [bot-report](https://github.com/code-423n4/2024-02-wise-lending/blob/main/bot-report.md#g-51-a-modifier-used-only-once-and-not-being-inherited-should-be-inlined-to-save-gas) just mentions the heading "A modifier used only once and not being inherited should be inlined to save gas" but shows no valid instance in codebase instead it confuses constructor for modifier

Modifier [onlyProposed()](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/OwnableMaster.sol#L16-L19) across the codebase is used only in [claimOwnership()](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/OwnableMaster.sol#L92-L97) function
```
    function claimOwnership()
        external
        onlyProposed
    {
        master = proposedMaster;
    }
```

The Best Approach would be to remove the definition and implementation of onlyProposed() modifier & _onlyProposed() function and use the inline logic to minimize the contract code-size and deployment gas costs.

`Here is the fix: `
```
// Remove lines 16-19

 -   modifier onlyProposed() {
 -      _onlyProposed();
 -      _;
    }

// Remove lines 37-46

 -   function _onlyProposed() private view {
 -       if (msg.sender == proposedMaster) {
 -          return;
 -       }
 -      revert NotProposed();
 -  }


// Update lines 92 - 97

 +   function claimOwnership()
 +       external
 +  {
 +      if (msg.sender != proposedMaster) revert("unauth");
 +      master = proposedMaster;
 +  }

```

## G-03 - Making Admin Functions payable saves both execution and deployment gas costs ,without risking user experience

Non-payable functions have an implicit require(msg.value == 0) inserted in them.
We can make admin specific functions payable to save gas, because the compiler won’t be checking the callvalue of the function.
We could also define every function as payable but since inexperienced users can accidentally loose ethers.

This Optimization technique will also make the contract smaller and cheaper to deploy as there will be fewer opcodes in the creation and runtime code.

[EXAMPLE](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L125-L133)
`Before fix`

```
    function setVerifiedIsolationPool(
        address _isolationPool,
        bool _state
    )
        external
        onlyMaster
    {
        verifiedIsolationPool[_isolationPool] = _state;
    }
```
`After Fix`
```
    function setVerifiedIsolationPool(
        address _isolationPool,
        bool _state
    )
        external
        payable
        onlyMaster
    {
        verifiedIsolationPool[_isolationPool] = _state;
    }

```
Similarly, [1](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L34) [2](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L106) [3](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L139) [4](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L151)

               
