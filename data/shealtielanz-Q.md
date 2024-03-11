# L1 - Using `block.timestamp` as `deadline` can be problematic.
# L2 - `PendlePowerFarmLeverageLogi`c doesn't adhere to `EIP-3156` specs completely
# L3 - Sealth donation attack is possible in `PendlePowerFarmToken`.
# L4 - Allow users to specify a `min amount out` when depositing and withdrawing from PendlePowerFarmToken.
# Info1 - `totalSupply()` is depreciated for curve pools.
# Info2 - `extcodesize()` check against zero isn't safe.
# Info3 - contracts that have been destroyed will pass the `token.code.length > 0` check.
# Info4 - Use a maths library when dealing with calculations.
# Info5 - Calculations using `PRECISION_FACTOR_YEAR` will not always be accurate.
# Info6 - Delete functions that you don't intend to use.
# R1 - Rounding issues arise during shares and amount calculation in `PendlePowerFarmToken`.