## Low Risk
| ID   | TITLE 
|------|------
| L-01 |Incorrect ownership status returned by isOwner() function for FEE_MANAGER_NFT
| L-02 |Usage of `_exists(uint256)` is depreciated in Openzeppelin ERC721 implementations Leading to compilation issues

## L-01 - Incorrect ownership status returned by isOwner() function for FEE_MANAGER_NFT

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L222-L232

when `FEE_MANAGER_NFT` isn't forwarded to `feeManager`, it is owned by `POSITON_NFT_CONTRACT`, calling `isOwner(FEE_MANAGER_NFT, POSITON_NFT_CONTRACT)` returns false while as underlying ERC721 `ownerof(FEE_MANAGER_NFT)` returns POSITON_NFT_CONTRACT because it is owned by POSITON_NFT_CONTRACT
```c
constructor(...) ...{
...
    FEE_MANAGER_NFT = _mintPositionForUser(address(this)); // here it is given to POSITON_NFT_ADDRESS
}

```

The issue arises because when the _nftId is FEE_MANAGER_NFT (ie, nftid == 0), isOwner() checks whether `feeManager == _owner`, 
```c
    function isOwner(uint256 _nftId, address _owner) external view returns (bool) {
        if (_nftId == FEE_MANAGER_NFT) {
            return feeManager == _owner;
        }
        
        ...
    }
```

Since FEE_MANAGER_NFT isn't forwarded yet by calling forwardFeeManagerNFT() which is the only function that updates feeManager address state. 

Therefore when FEE_MANAGER_NFT isn't forwarded to feeManager, calling isOwner with the POSITON_NFT_CONTRACT return false, as it compares whether `feeManager == _owner`

which translates to `address(0) == address(POSITON_NFT_CONTRACT)`

```c
     if (_nftId == FEE_MANAGER_NFT) {  
            return feeManager == _owner; // address(0)  == address(address(this)) / address(POSITION_NFT)
        }

```

`Recommended Fix`
- add a boolean that tracks whether the FEE_MANAGER_NFT is forwarded to manager or is still owned by POSITION_NFT_CONTRACT
- change that boolean to true in  forwardFeeManagerNFT()
- then use if/else in IsOwner function to handle both ownership cases
```diff
+ bool public managerNftTransferred;

   function forwardFeeManagerNFT(address _feeManagerContract) external      onlyMaster {
 ...
 ...
+ managerNftTransferred = true;
}

    function isOwner(uint256 _nftId,address _owner) external view returns (bool){
        if (_nftId == FEE_MANAGER_NFT) {
+           if (managerNftTransferred) {
            return feeManager == _owner;
+           } else {
+           return address(this) == _owner;
+        }
        }
      ...
      ...
}

```

## L-02 - Usage of `_exists(uint256)` is depreciated in Openzeppelin ERC721 implementations Leading

[url-1](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L161-L163)
[url-2](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L349)

Openzeppelin has depreciated _exists(uint256)  due to concerns that it was not part of the standard ERC-721 specification. 

This function was initially proposed as a proprietary extension to the ERC-721 standard, providing a way to check if a specific token exists within a contract. 

However, it was decided that this function should not be part of the standard interface for ERC-721 tokens.

You can read more about here in openzeppelin [github-issue](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1192)

Since The PositionNFTs.sol is importing openzeppelin contracts, on top of that pragma is 0.8.24, The compiler will raise `Undeclared identifier` compilation issue



## Non-Criticals
| ID   | TITLE
|------|------
| N-01 | Reserving specific nftId of POSITON_NFT can be frontRunned

## N-01 -  Reserving specific nftId of POSITON_NFT can be frontRunned
[url-1](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L90-L97)
[url-2](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L99-L109)

```
   function reservePositionForUser(
        address _user
    )
        onlyReserveRole
        external
        returns (uint256)
    {
        return _reservePositionForUser(
            _user
        );
    }
```
Initially when contracts  public reserving isn't blocked, i.e, `bool public reservePublicBlocked;` is not set to true, anyone can reserve a specific NFT id for them
But Anybody can spend extra gas and frontrun other's transaction for reserving specific nftId which kinda hurts some user's sentiment if they believe in lucky numbers, lol