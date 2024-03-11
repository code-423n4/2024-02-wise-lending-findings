## [L-01] The implementation of _getCurrentSharePriceMax method is not consistent with the comment.
The limit written in the comment is 500%, while the actual implementation we can see that it can reach 1100% in a year. It is recommended to keep it consistent
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L250-L268
```solidity
    /**
    * @dev Since pool inception share price
    * increase for both lending and borrow shares
    * is capped at 500% apr max in between a transaction.
    */
    function _getCurrentSharePriceMax(
        address _poolToken
    )
        private
        view
        returns (uint256)
    {
        uint256 timeDifference = block.timestamp
            - timestampsPoolData[_poolToken].initialTimeStamp;

        return timeDifference
            * RESTRICTION_FACTOR
            + PRECISION_FACTOR_E18;
    }
```