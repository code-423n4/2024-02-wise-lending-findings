## Summary

### Gas Optimization

no |Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |Add unchecked{} for subtractions where the operands cannot underflow because of a previous require() or if statement |  |11|--| 
| [G-02] | Functions guaranteed to when called by normal users can be marked Payable |  |24|--| 
| [G-03] |<X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= ) |  |30|--| 
| [G-04] |Using fixed bytes is cheaper than using string |  |12|--| 
| [G-05] | Do not calculate constants |  |24|--| 
| [G-06] |State variables should be cached in stack variables rather than re-reading them from storage |  |8|--| 
| [G-07] |Not using the named return variable when a function returns, wastes deployment gas |  |1|--| |
| [G-08] |Don't initialize variables with default value | |1|--| 
| [G-09] | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | | 2 |
| [G-10] | Functions guaranteed to when called by normal users can be marked Payable | | 12 |
| [G-11] | Using fixed bytes is cheaper than using string |  | 6 |
| [G-12] |  Not using the named return variable when a function returns, wastes deployment gas | | 5 |
| [G-13] | Can make the variable outside the loop to save gas |  | 2 |
| [G-14] | Using `calldata` instead of `memory` for read-only arguments in external functions saves gas  | | 10 |
| [G-15] | Public function not called by the contract should be declared external instead  |  | 1 |
| [G-16] | Using storage instead of memory for structs/arrays saves gas | | 8 |
| [G-17] | With assembly, .call (bool success)  transfer can be done gas-optimized |  | 5 |
| [G-18] | Duplicated require()/if() checks should be refactored to a modifier or function |  | 1 |
| [G-19] | Use constants instead of type(uintx).max |  | 3 |
| [G-20] | Use nested if and, avoid multiple check combinations | | 4 |
| [G-21] | Use assembly to emit events | | 2 |
| [G-22] | Using ERC721A instead of ERC721 for more gas-efficient  |  | 2|



## Gas Optimizations  

## [G-1] Add unchecked{} for subtractions where the operands cannot underflow because of a previous require() or if statement  

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn't possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.



```solidity
file:/contracts/WiseSecurity/WiseSecurity.so
625           netAPY = (ethValueGain - ethValueDebt)
626              / totalETHSupply;

631        netAPY = (ethValueDebt - ethValueGain)
532               / totalETHSupply;
848            tokenAmount = tokenAmount - _poolWithdrawAmount;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L625-L626

```solidity
File: /contracts/MainHelper.sol
 357      uint256 timeDifference = block.timestamp
 358          - timestampsPoolData[_poolToken].timeStamp;

 507        uint256 bareIncrease = borrowPoolData[_poolToken].borrowRate
 508          * (currentTime - timestampsPoolData[_poolToken].timeStamp)
 509          * borrowPoolData[_poolToken].pseudoTotalBorrowAmount
 510          + bufferIncrease[_poolToken];


```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L357-L358


## [G-2] Functions guaranteed to when called by normal users can be marked Payable

The onlyOwner modifier makes a function revert if not called by the address registered as the owner


```solidity
file: /contracts/DerivativeOracles/CustomOracleSetup.sol
26   function renounceOwnership()
27        external
27        onlyOwner
29    {
30        master = address(0x0);
31    }

42        function setRoundData(
43                uint80 _roundId,
44                uint256 _updateTime
45            )
46                external
47                onlyOwner
48           {
49                timeStampByRoundId[_roundId] = _updateTime;
50            }

52            function setGlobalAggregatorRoundId(
53                uint80 _aggregatorRoundId
54            )
55                external
56                onlyOwner
57            {
58                globalRoundId = _aggregatorRoundId;
59            }


```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/CustomOracleSetup.sol#L26-L31

## [G-3] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )


 AVOID COMPOUND ASSIGNMENT OPERATOR IN STATE VARIABLES

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

```solidity
file:/contracts/WiseSecurity/WiseSecurity.sol
481            weightedRate += ethValue

484            overallETH += ethValue;

538            weightedRate += ethValue

614            totalETHSupply += ethValue;

615           ethValueGain += ethValue
616             * getLendingRate(token);

679            buffer += WISE_LENDING.lendingPoolData(token).collateralFactor

```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L481

```solidity
file: /contracts/MainHelper.sol
591       userLendingData[_nftId][_poolToken].shares += _shares;
606        userLendingData[_nftId][_poolToken].shares -= _shares;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L591


```solidity
file:/contracts/WiseSecurity/WiseSecurityHelper.sol
 44       weightedTotal += amount
 48            unweightedAmount += amount;

99            weightedTotal += WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor

128            amount += getFullCollateralETH(

177            weightedTotal += WISE_LENDING.lendingPoolData(tokenAddress).collateralFactor

214        ethCollateral += getETHCollateral(

267        ethCollateral += getETHCollateralUpdated(
```
            
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L44-L44
```solidity
file:/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

343       underlyingLpAssetsCurrent += additonalAssets;
344        totalLpAssetsToDistribute -= additonalAssets;
512        totalLpAssetsToDistribute += _amount;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L343-L344C

```solidity
file:/contracts/FeeManager/FeeManager.sol
 220       incentiveUSD[incentiveOwnerA] += _value;

 238        incentiveUSD[incentiveOwnerB] += _value;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L220


```solidity
file:/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
 140       reservedPendleForLocking += tokenAmountSend;
 512           childInfo.reservedForCompound[i] += _amounts[i];
 630                weightSum += _weights[i];
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L140

```solidity
file:/contracts/WiseLowLevelHelper.sol
230        globalPoolData[_poolToken].totalPool += _amount;
248        lendingPoolData[_poolToken].totalDepositShares += _amount;
266        borrowPoolData[_poolToken].pseudoTotalBorrowAmount += _amount;
347        globalPoolData[_poolToken].totalBareToken -= _amount;
379        userMapping[_nftId][_poolToken] -= _amount;

note : it has more instances about this report but i mentioned some of thems
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLowLevelHelper.sol#L230
## [G-4] Using fixed bytes is cheaper than using string


As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.




```solidity
file:/contracts/PositionNFTs.sol#L13-L14

13   string public baseURI;
14    string public baseExtension;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L13-L14

```solidity
file:/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.so

26         string public baseURI;
27         string public baseExtension;
162        string calldata _newBaseURI
171        string calldata _newBaseExtension
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L26-L27

```solidity
file:/contracts/DerivativeOracles/PendleLpOracle.sol
36        string memory _oracleName,
76        string public name;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PendleLpOracle.sol#L36

```solidity
file:/contracts/DerivativeOracles/PtOracleDerivative.sol
29        string memory _oracleName,
74        string public name;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOracleDerivative.sol#L29

```solidity
file:
28        string memory _oracleName,
60        string public name;


```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PtOraclePure.sol#L28
``

## [G-5] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.


```solidity
file:/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
58    uint256 internal constant PRECISION_FACTOR_E36 = PRECISION_FACTOR_E18 * PRECISION_FACTOR_E18;
59    uint256 internal constant PRECISION_FACTOR_YEAR = PRECISION_FACTOR_E18 * ONE_YEAR;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L58-L59
```solidity
file:/contracts/WiseLendingDeclaration.sol
170   uint256 internal constant MIN_BORROW_SHARE_PRICE = 5 * PRECISION_FACTOR_E18   / 10;
297    uint256 internal constant PRECISION_FACTOR_E36 = PRECISION_FACTOR_E18 * PRECISION_FACTOR_E18;
303    uint256 internal constant PRECISION_FACTOR_YEAR = PRECISION_FACTOR_E18 * ONE_YEAR;
310   uint256 internal constant LOWER_BOUND_MAX_RATE = 100 * PRECISION_FACTOR_E16;
311    uint256 internal constant UPPER_BOUND_MAX_RATE = 300 * PRECISION_FACTOR_E16;

315    uint256 internal constant THRESHOLD_SWITCH_DIRECTION = 90 * PRECISION_FACTOR_E16;
316    uint256 internal constant THRESHOLD_RESET_RESONANCE_FACTOR = 75 * PRECISION_FACTOR_E16;

319     uint256 internal constant MAX_COLLATERAL_FACTOR = 85 * PRECISION_FACTOR_E16;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLendingDeclaration.sol#L170-L171
```solidity
file:/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol
62      uint256 internal constant MAX_COLLATERAL_FACTOR = 95 * PRECISION_FACTOR_E16;
101    uint256 internal constant MAX_LEVERAGE = 15 * PRECISION_FACTOR_E18;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L62
```solidity
file:/contracts/WiseSecurity/WiseSecurityDeclarations.sol
185    uint256 public constant BORROW_PERCENTAGE_CAP = 95 * PRECISION_FACTOR_E16;

225       uint256 internal constant LIQUIDATION_INCENTIVE_MAX = 11 * PRECISION_FACTOR_E16;
226   uint256 internal constant LIQUIDATION_INCENTIVE_MIN = 2 * PRECISION_FACTOR_E16;
227    uint256 internal constant LIQUIDATION_INCENTIVE_POWERFARM_MAX = 4 * PRECISION_FACTOR_E16;


    // Liquidation Fee threshholds
230  uint256 internal constant LIQUIDATION_FEE_MIN_ETH = 3 * PRECISION_FACTOR_E18;
231    uint256 internal constant LIQUIDATION_FEE_MAX_ETH = 100 * PRECISION_FACTOR_E18;
232    uint256 internal constant LIQUIDATION_FEE_MAX_NON_ETH = 10 * PRECISION_FACTOR_E18;
233    uint256 internal constant LIQUIDATION_FEE_MIN_NON_ETH = 30 * PRECISION_FACTOR_E16;


```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L185

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L225-L234


```solidity
file:/contracts/FeeManager/DeclarationsFeeManager.sol
```solidity
172     uint256 public constant INCENTIVE_PORTION = 5 * PRECISION_FACTOR_E15;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/DeclarationsFeeManager.sol#L172
```solidity
file:
67    uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/PendleLpOracle.sol#L67
```solidity
file:/contracts/WiseOracleHub/Declarations.sol
 109   uint32 internal constant TWAP_PERIOD = 30 * SECONDS_IN_MINUTE;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/Declarations.sol#L109


## [G-6] State variables should be cached in stack variables rather than re-reading them from storage

/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol
The instances below point to the second+ access of a state variable within a function.
Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read.
Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
Most of the times this if statement will be true and we will save 100 gas at a small possibility of 3 gas loss ,

```solidity
file:

343        underlyingLpAssetsCurrent += additonalAssets;
344        totalLpAssetsToDistribute -= additonalAssets;
      //uint256 private INITIAL_TIME_STAMP;
120    uint256 timeDifference = block.timestamp
121            - INITIAL_TIME_STAMP;
731        INITIAL_TIME_STAMP = block.timestamp;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L120-L121
```solidity
file:/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol
 365       if (isShutdown == true) {
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L365-L365

```solidity
file:/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol
135        totalMinted++;
136        totalReserved--;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L135-L136

## [G-7]Not using the named return variable when a function returns, wastes deployment gas

When you execute a function that returns values in Solidity, the EVM still performs the necessary operations to execute and return those values. This includes the cost of allocating memory and packing the return values. If the returned values are not utilized, it can be seen as wasteful since you are incurring gas costs for operations that have no effect.
```solidity
file:/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
378        returns (UserReward memory)

```
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L378



## [G-8]Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```solidity

27    uint16 internal constant REF_CODE = 0;
   
  
```
 https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/Declarations.sol#L27-L27


## [G-9]  Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
Saves 5 gas per iteration


```solidity
file: /PowerFarms/PowerFarmNFTs/MinterReserver.sol

135         totalMinted++;

136        totalReserved--;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L135C1-L136C25
 

## [G-10] Functions guaranteed to when called by normal users can be marked Payable

The onlyOwner modifier makes a function revert if not called by the address registered as the owner


```solidity
file: /DerivativeOracles/CustomOracleSetup.sol

   26    function renounceOwnership()
        external
        onlyOwner
    {
        master = address(0x0);
    }


    42    function setRoundData(
        uint80 _roundId,
        uint256 _updateTime
    )
        external
        onlyOwner
    {


    52    function setGlobalAggregatorRoundId(
        uint80 _aggregatorRoundId
    )
        external
        onlyOwner
    {
        globalRoundId = _aggregatorRoundId;
    } 

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L26C1-L31C6


```solidity
    file: /WiseLending.sol

    859    function withdrawOnBehalfExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _withdrawAmount
    )
        external
        onlyAaveHub
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {


    939    function withdrawOnBehalfExactShares(
        uint256 _nftId,
        address _poolToken,
        uint256 _shares
    )
        external
        onlyAaveHub
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {        


    1055    function borrowOnBehalfExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        external
        onlyAaveHub
        syncPool(_poolToken)
        healthStateCheck(_nftId)
        returns (uint256)
    {


    1412    function corePaybackFeeManager(
        address _poolToken,
        uint256 _nftId,
        uint256 _amount,
        uint256 _shares
    )
        external
        onlyFeeManager
        syncPool(_poolToken)
    {        


```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L859C1-L869C6


```solidity
file: /WiseSecurity/WiseSecurity.sol

    85     function setLiquidationSettings(
        uint256 _baseReward,
        uint256 _baseRewardFarm,
        uint256 _newMaxFeeETH,
        uint256 _newMaxFeeFarmETH
    )
        external
        onlyMaster
    {

    142    function prepareCurvePools(
        address _poolToken,
        CurveSwapStructData calldata _curveSwapStructData,
        CurveSwapStructToken calldata _curveSwapStructToken
    )
        external
        onlyWiseLending
    {       

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L85C1-L94C5


```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

    592    function changeMintFee(
        uint256 _newFee
    )
        external
        onlyController
    {
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L592C1-L597C6



```solidity
file: /contracts/FeeManager/FeeManager.sol

 66    function setAaveFlag(
        address _poolToken,
        address _underlyingToken
    )
        external
        onlyMaster
    {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L66C1-L72C6

```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

    32    function withdrawLp(
        address _pendleMarket,
        address _to,
        uint256 _amount
    )
        external
        onlyChildContract(_pendleMarket)
    {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L32C1-L39C6


## [G-11] Using fixed bytes is cheaper than using string

```solidity
file: /PositionNFTs.sol

376         returns (string memory str)


```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L376C1-L376C36

    
```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

685        string memory _tokenName,
686    string memory _symbolName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L685C1-L686C35


```solidity
file: /PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

39        string memory _name,
40        string memory _symbol

```
 https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L39C1-L40C30


```solidity
file: /DerivativeOracles/PendleLpOracle.sol

76    string public name;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L76C1-L76C24


## [G-12] Not using the named return variable when a function returns, wastes deployment gas

```solidity
 file: /WiseSecurity/WiseSecurityHelper.sol
 
207            return ethCollateral;

211            return ethCollateral;
   

264            return ethCollateral;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L207C6-L207C34

```solidity
 file: /WiseCore.sol

589            return receiveAmount;

593            return receiveAmount;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L589C1-L589C34


## [G-13] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas


```solidity
file: /WrapperHub/AaveHelper.sol

256            address currentAddress = WISE_LENDING.getPositionLendingTokenByIndex(
                _nftId,
                i
            );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L256C1-L259C82


```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

260            address token = childInfo.rewardTokens[i];

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L260C1-L260C55


## [G-14] Using `calldata` instead of `memory` for read-only arguments in external functions saves gas 


```solidity
file: /PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

59    function receiveFlashLoan(
        IERC20[] memory _flashloanToken,
        uint256[] memory _flashloanAmounts,
        uint256[] memory _feeAmounts,
        bytes memory _userData
    )
        external
    {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L59C1-L66C6

```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

685        string memory _tokenName,
686        string memory _symbolName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L685C1-L686C35


```solidity
file: FeeManager/FeeManager.sol

573        address[] memory _feeTokens

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L573C1-L573C36


```solidity
file: /PositionNFTs.sol

276        returns (uint256[] memory)

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L276C1-L276C35


```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

37        string memory _tokenName,
38        string memory _symbolName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L37C1-L38C35


## [G-15]  Public function not called by the contract should be declared external instead 

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.

```solidity
file: /contracts/MainHelper.sol

55    function calculateBorrowShares(
        address _poolToken,
        uint256 _amount,
        bool _maxSharePrice
    )
        public
        view
        returns (uint256)
    {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L55C1-L63C6




## [G-16] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
file: /contracts/MainHelper.sol

399        address[] memory tokens = _userTokenData[
            _nftId
        ];

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L399C1-L401C11


```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

199        uint256[] memory rewardsOutsideArray = _calculateRewardsClaimedOutside();

222        address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(
            UNDERLYING_PENDLE_MARKET
        );

226        uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
            UNDERLYING_PENDLE_MARKET
        );

231        uint256[] memory rewardsOutsideArray = new uint256[](l);

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L199C1-L199C82


```solidity
file: PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

27        IERC20[] memory tokens = new IERC20[](1);
28        uint256[] memory amount = new uint256[](1);

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L27C1-L28C52


```solidity
file: /PositionNFTs.sol

292        uint256[] memory tokenIds = new uint256[](

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L292C1-L292C51

 
## [G-17] With assembly, .call (bool success)  transfer can be done gas-optimized
 
```solidity
file: /WiseSecurity/WiseSecurity.sol

205           (
            bool success
            ,
        ) = curvePool.call{value: 0} (
            curveSwapInfoData[_poolToken].swapBytesPool
        );


```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L206C1-L210C11


```solidity
file: /WiseOracleHub/OracleHelper.sol

78        (bool success, ) = _tokenAggregator.call(
            abi.encodeWithSignature(
                "maxAnswer()"
            )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L78C1-L81C14


```solidity
file: /WrapperHub/AaveHelper.sol

208        (bool success, ) = payable(_recipient).call{
            value: _amount
        }("");

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L208C1-L210C15


```solidity
file: /TransferHub/SendValueHelper.sol

24        (
            bool success
            ,
        ) = payable(_recipient).call{
            value: _amount
        }("");

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L24C1-L29C15


```solidity
file: /TransferHub/CallOptionalReturn.sol

19        (
            bool success,
            bytes memory returndata
        ) = token.call(
            data
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L19C1-L24C11


## [G-18] Duplicated require()/if() checks should be refactored to a modifier or function   

```solidity
file: /WiseSecurity/WiseSecurity.sol
/// @audit this if state come duplicate on line: 284, 367

246         if (_checkBlacklisted(_poolToken) == true) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L246C1-L246C53


## [G-19] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

98    uint256 internal constant MAX_AMOUNT = type(uint256).max;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L98C1-L98C62


```solidity
file: /WiseSecurity/WiseSecurityDeclarations.sol

209    uint256 internal constant UINT256_MAX = type(uint256).max;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L209C1-L209C63


```solidity
file: /WrapperHub/Declarations.sol

39    uint256 internal constant MAX_AMOUNT = type(uint256).max;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L39C1-L39C62



## [G-20] Use nested if and, avoid multiple check combinations

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.


```solidity
file: /WiseOracleHub/OracleHelper.sol

315        if (_tokenAddress != _token0 && _tokenAddress != _token1) {

439        if (tickCumulativesDelta < 0 && (tickCumulativesDelta % twapPeriodInt56 != 0)) { 

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L315C1-L315C68


```solidity
file: /WiseCore.sol

572        if (potentialPureExtraCashout > 0 && potentialPureExtraCashout <= pureCollateral) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L572C1-L572C92


```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

255            if (lastIndex[i] == 0 && index > 0) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L255C1-L255C50



## [G-21] Use assembly to emit events

We can use assembly to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.
 
```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

129        emit ETHReceived(
            msg.value,
            msg.sender
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L129C1-L132C11


```solidity
file: /PowerFarms/PowerFarmNFTs/MinterReserver.sol

169        emit ERC721Received(
            _operator,
            _from,
            _tokenId,
            _data
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L169C1-L174C11


## [G-22] Using ERC721A instead of ERC721 for more gas-efficient 

ERC721A is a proposed extension to the ERC721 standard that aims to improve gas efficiency and reduce the cost of deploying and using non-fungible tokens (NFTs) on the Ethereum blockchain. 

```solidity
file: /PositionNFTs.sol

11 contract PositionNFTs is ERC721Enumerable, OwnableMaster {

30        ERC721(
            _name,
            _symbol
        )
        OwnableMaster(
            msg.sender
        )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L11


```solidity
file: /PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

42        ERC721(
            _name,
            _symbol
        )
        OwnableMaster(
            msg.sender
        )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L42C1-L48C10