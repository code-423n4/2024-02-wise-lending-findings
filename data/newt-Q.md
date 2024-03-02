# Summary

The function _getDeltaPole is a pure punction. A pure function does not read or modify a state variable. In the function below, NORMALISATION_FACTOR is a state variable.    

    function _getDeltaPole(
        uint256 _maxPole,
        uint256 _minPole
    )
        private
        pure
        returns (uint256)
    {
        return (_maxPole - _minPole) / NORMALISATION_FACTOR;
    }

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L293-L302