## Unauthorized Withdrawal of Private Deposited ERC20 Funds by FeeManager

* The vulnerability arises from the solelyWithdraw function in WiseLending.sol at line 781. This function utilizes the POSITION_NFTS.isOwner(_nftId, _caller) == false call to check the ownership of _nftId.

* However, a privileged condition in the function grants exclusive withdrawal rights to the FeeManager for a specific _nftId. The code snippet responsible for this behavior is:

```
# PositionNFTs.sol

if (_nftId == FEE_MANAGER_NFT) {
    return feeManager == _owner;
}
```

* This centralized behavior effectively allows the FeeManager to obtain owner permissions for any _nftId and subsequently withdraw private ERC20 funds. This poses a critical security risk, enabling unauthorized access to sensitive funds.

### Time spent:
3 hours