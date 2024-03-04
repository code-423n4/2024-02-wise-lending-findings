## [G] Gas Optimization in the _calculateRewardsClaimedOutside function

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L218-L303

function _calculateRewardsClaimedOutside()internal returns (uint256[] memory)

//code...

while (i < l) {
            UserReward memory userReward = _getUserReward(
                rewardTokens[i],
                PENDLE_POWER_FARM_CONTROLLER
            );

            if (userReward.accrued > 0) {
                PENDLE_MARKET.redeemRewards(
                    PENDLE_POWER_FARM_CONTROLLER
                );

                userReward = _getUserReward(
                    rewardTokens[i],
                    PENDLE_POWER_FARM_CONTROLLER
                );
            }

            index = userReward.index;

            if (lastIndex[i] == 0 && index > 0) {
                rewardsOutsideArray[i] = 0;
                _overWriteIndex(
                    i
                );
                unchecked {
                    ++i;
                }
                continue;
            }

            if (index == lastIndex[i]) {
                rewardsOutsideArray[i] = 0;
                unchecked {
                    ++i;
                }
                continue;
            }

            uint256 indexDiff = index
                - lastIndex[i];

            uint256 activeBalance = _getActiveBalance();
            uint256 totalLpAssetsCurrent = totalLpAssets();
            uint256 lpBalanceController = _getBalanceLpBalanceController();

            bool scaleNecessary = totalLpAssetsCurrent < lpBalanceController;

            rewardsOutsideArray[i] = scaleNecessary
                ? indexDiff
                    * activeBalance
                    * totalLpAssetsCurrent
                    / lpBalanceController
                    / PRECISION_FACTOR_E18
                : indexDiff
                    * activeBalance
                    / PRECISION_FACTOR_E18;

                   _overWriteIndex(
                i
            );

            unchecked {
                ++i;
            }
        }

        return rewardsOutsideArray;
    }

My recommendation for gas optimization involves caching the values of `activeBalance`, `totalLpAssetsCurrent`, and `lpBalanceController` outside of the while loop within the `_calculateRewardsClaimedOutside` function. This optimization is based on the fact that these values do not change during the execution of the loop, and therefore, there is no need to repeatedly call the functions that compute them for each iteration.

###### Here's a detailed explanation of the optimization:

1 **Initial Calls**:

- **activeBalance**: Obtained by calling `_getActiveBalance()`, which likely makes an external call to the `PENDLE_MARKET` contract to retrieve the active balance of the `PENDLE_POWER_FARM_CONTROLLER`.

- **totalLpAssetsCurrent**: Obtained by calling `totalLpAssets()`, which computes the sum of `underlyingLpAssetsCurrent` and `totalLpAssetsToDistribute`.

- **lpBalanceController**: Obtained by calling `_getBalanceLpBalanceController()`, which likely makes an external call to the `PENDLE_MARKET` contract to retrieve the LP token balance of the `PENDLE_POWER_FARM_CONTROLLER`.

2 **Loop Execution**:

 - In the original code, these values are retrieved within the loop for each reward token. Since external calls (such as contract calls) are expensive in terms of gas, and since the values do not change during the loop, this results in unnecessary gas consumption.

3 **Optimization**:

- By moving the retrieval of these values outside the loop, you ensure that each function is called only once, regardless of the number of reward tokens. This reduces the number of external calls and thus saves gas.
The optimized code performs these calls before entering the loop and uses the cached values within the loop.

4 **Gas Savings**:

 - External calls can be one of the most significant contributors to gas costs in Ethereum smart contracts. By reducing the number of external calls, you save a significant amount of gas, especially when the loop has many iterations.
Additionally, by caching the values, you also save the computational overhead of repeatedly executing the logic within the called functions.

5 **Code Maintainability**:

This optimization not only saves gas but also improves the readability and maintainability of the code. It becomes clearer that these values are constants within the scope of the function execution, and it avoids potential bugs that could arise if the values were mistakenly thought to change within the loop.

## Note:

The `PENDLE_POWER_FARM_CONTROLLER` can also be cached outside the while loop. However, in this case, `PENDLE_POWER_FARM_CONTROLLER` is not a variable that is being computed or retrieved through a function call; it is a state variable that holds the address of the controller. Accessing a state variable is relatively cheap in terms of gas costs compared to external function calls.

Nonetheless, even though the cost savings might be minimal, it is still a good practice to reduce the number of times you access state variables within a loop if they do not change during the loop's execution.