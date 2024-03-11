## [L-1]: Return value of call validation is not checked.
## [L-2]: open external function which accepts calldata may open paths for manipulations.


## [L-1]: Return value of call in various contracts is not checked.

### Impact
`CallOptionalReturn._callOptionalReturn` is an important function used by various other contracts including `ApprovalHelper` and `TransferHelper` where it takes in a token and performs a function call on that token. This `_callOptionalReturn` function returns a boolean value. However, this value is not checked by the contracts which use it. 

```
 function _callOptionalReturn(
        address token,
        bytes memory data
    )
        internal
        returns (bool call)
    { ............. 
}
```

```
src: 2024-02-wise-lending/contracts/TransferHub/ApprovalHelper.sol

function _safeApprove(
        address _token,
        address _spender,
        uint256 _value
    )
        internal
    {
        // @audit-low: Should check return value
        _callOptionalReturn(
            _token,
            abi.encodeWithSelector(
                IERC20.approve.selector,
                _spender,
                _value
            )
        );
    }
```
This is the case in all other contracts where this function is used. The call may potentially return false or even fail and this is not checked.

### Proof of Concept
This is as given above in the impact. The return value of the function call is not checked.

### Tools Used
Manual review 

### Recommended Mitigation
1. Check the the return value of `_callOptionalReturn` whereever they are used.

```diff
```
src: 2024-02-wise-lending/contracts/TransferHub/ApprovalHelper.sol

function _safeApprove(
        address _token,
        address _spender,
        uint256 _value
    )
        internal
    {
       
+           bool success = _callOptionalReturn(
            _token,
            abi.encodeWithSelector(
                IERC20.approve.selector,
                _spender,
                _value
            )
        );
    }
```

Additional actions can be implemented based on result of this call.

## [L-2]: Open external function which accepts calldata may open paths for manipulations.



### Impact
The `WiseOracleHub.recalibrateBulk` is an open function which accepts token addresses in the form of calldata and performs a recalibration process on these tokens.
```
function recalibrateBulk(
        address[] calldata _tokenAddresses
    )
        external
    {
        uint256 i;
        uint256 l = _tokenAddresses.length;

        while (i < l) {
            _recalibrate(
                _tokenAddresses[i]
            );

            unchecked {
                ++i;
            }
        }
    }
```
 The issue is that an open external function which accepts calldata such as this may open up attack paths for DOS and other scenarios. Although, the potential impact of any attack using this function is not clear, this may lead to complex attacks nontheless.


### Proof of Concept

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/WiseOracleHub.sol#L501C14-L501C29

An attacker may call this function with enough data to cause the function to become unavailable to other users of the system. Additionally, calldata can be very arbitrary, therefore, it is possible for more complex scenarios to happen starting from this function.

### Tools Used
Manual review 

### Recommended Mitigation
1. Allow this function to be called by only trusted admins or users. 
If this function is to be left open, the calldata should pass through necessary checks to ensure the right data has been passed in.






