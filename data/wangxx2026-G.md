##[G-01] In the getValueUtilization method, you can add the unchecked code to save gas when you return the calculation.
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L163-L181
```solidity
    function _getValueUtilization(
        address _poolToken
    )
        private
        view
        returns (uint256)
    {
        uint256 totalPool = globalPoolData[_poolToken].totalPool;
        uint256 pseudoPool = lendingPoolData[_poolToken].pseudoTotalPool;

        if (totalPool >= pseudoPool) {
            return 0;
        }
        // @audit The above has already judged totalPoolis less than pseudoPool, here the operation will not overflow, you can add the unchecked
        return PRECISION_FACTOR_E18 - (PRECISION_FACTOR_E18
            * totalPool
            / pseudoPool
        );
    }
```
