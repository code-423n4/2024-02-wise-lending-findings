## Unrestricted Access to NFTs via Pool Manager or Isolation Pool Address

* The WiseLending.sol contract in the setRegistrationIsolationPool() external function in  ```L-1407``` allows the Isolation Pool owner to lock any NFT using the _nftId. Subsequently, the victim is unable to access the depositExactAmountETH() function at ```L-389``` due to the check in the _handleDeposit() function at ```L- 408```. This is because the PositionLocked status, validated at ```L-204``` in WiseCore.sol, ensures that positionLocked[_nftId] must be false for successful execution.

```
#WiseLending.sol 

L-1389 function setRegistrationIsolationPool(
        uint256 _nftId,
        bool _registerState
    ) external {
...
...

@audit-issue  Pool Manager or Isolation Pool Address locked any NFT
L-1407 positionLocked[_nftId] = _registerState; 

}
```

* victim can't access depositExactAmountETH function

```
#WiseLending.sol 

 L-389   function depositExactAmountETH(
        uint256 _nftId
    )
        external
        payable
        syncPool(WETH_ADDRESS)
        returns (uint256)
    {
        return _depositExactAmountETH(
            _nftId
        );  @audit-issue this function will check _checkPositionLocked status 
    }

#WiseCore.sol

 L-193    function _checkPositionLocked(
        uint256 _nftId,
        address _caller
    )
        internal
        view
    {
 ...
        if (positionLocked[_nftId] == false) {
            return;
        }
        revert PositionLocked(); @audit-issue revert the  process 
    }
...
...
```



