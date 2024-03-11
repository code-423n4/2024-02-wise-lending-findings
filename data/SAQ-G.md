## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Pre-increments and pre-decrements are cheaper than post-increments and post-decrements | 2 | - |
| [G-02] | Functions guaranteed to when called by normal users can be marked Payable | 12 | - |
| [G-03] | Using fixed bytes is cheaper than using string | 6 | - |
| [G-04] |  Not using the named return variable when a function returns, wastes deployment gas | 5 | - |
| [G-05] | Can make the variable outside the loop to save gas | 2 | - |
| [G-06] | Using `calldata` instead of `memory` for read-only arguments in external functions saves gas  | 10 | - |
| [G-07] | Public function not called by the contract should be declared external instead  | 1 | - |
| [G-08] | Don't initialize variables with default value | 1 | - |
| [G-09] | Using storage instead of memory for structs/arrays saves gas | 8 | - |
| [G-10] | With assembly, .call (bool success)  transfer can be done gas-optimized | 5 | - |
| [G-11] | Duplicated require()/if() checks should be refactored to a modifier or function | 1 | - |
| [G-12] | Use constants instead of type(uintx).max | 3 | - |
| [G-13] | Use nested if and, avoid multiple check combinations | 4 | - |
| [G-14] | Do not calculate constant | 13 | - |
| [G-15] | Use assembly to emit events | 2 | - |
| [G-16] | Using ERC721A instead of ERC721 for more gas-efficient  | 2 | - |
| [G-17] | Avoid fetching a low-level call's return data by using assembly | 3 | - |
| [G-18] | Pre-increment and pre-decrement are cheaper than +1 ,-1 | 6 | - |
| [G-19] | bytes.concat() can be used in place of abi.encodePacked | 1 | - |

## Gas Optimizations  

## [G-01]  Pre-increments and pre-decrements are cheaper than post-increments and post-decrements
Saves 5 gas per iteration


```solidity
file: /PowerFarms/PowerFarmNFTs/MinterReserver.sol

135         totalMinted++;

136        totalReserved--;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L135C1-L136C25
 

## [G-02] Functions guaranteed to when called by normal users can be marked Payable

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


## [G-03] Using fixed bytes is cheaper than using string

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


## [G-04] Not using the named return variable when a function returns, wastes deployment gas

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


## [G-05] Can make the variable outside the loop to save gas

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


## [G-06] Using `calldata` instead of `memory` for read-only arguments in external functions saves gas 


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


## [G-07]  Public function not called by the contract should be declared external instead 

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


## [G-08] Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```solidity
file: /contracts/WrapperHub/Declarations.sol

27    uint16 internal constant REF_CODE = 0;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L27C1-L27C43


## [G-09] Using storage instead of memory for structs/arrays saves gas

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

 
## [G-10] With assembly, .call (bool success)  transfer can be done gas-optimized
 
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


## [G-11] Duplicated require()/if() checks should be refactored to a modifier or function   

```solidity
file: /WiseSecurity/WiseSecurity.sol
/// @audit this if state come duplicate on line: 284, 367

246         if (_checkBlacklisted(_poolToken) == true) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L246C1-L246C53


## [G-12] Use constants instead of type(uintx).max

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



## [G-13] Use nested if and, avoid multiple check combinations

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


## [G-14] Do not calculate constant

When you define a constant in Solidity, the compiler can calculate its value at compile-time and replace all references to the constant with its computed value. This can be helpful for readability and reducing the size of the compiled code, but it can also increase gas usage at runtime.


```solidity
file: /WiseLendingDeclaration.sol

302    uint256 internal constant PRECISION_FACTOR_YEAR = PRECISION_FACTOR_E18 * ONE_YEAR;

310    uint256 internal constant LOWER_BOUND_MAX_RATE = 100 * PRECISION_FACTOR_E16;
311    uint256 internal constant UPPER_BOUND_MAX_RATE = 300 * PRECISION_FACTOR_E16;

314    uint256 internal constant THRESHOLD_SWITCH_DIRECTION = 90 * PRECISION_FACTOR_E16;
315    uint256 internal constant THRESHOLD_RESET_RESONANCE_FACTOR = 75 * PRECISION_FACTOR_E16;


319    uint256 internal constant MAX_COLLATERAL_FACTOR = 85 * PRECISION_FACTOR_E16;

323    uint256 internal constant RESTRICTION_FACTOR = 10
        * PRECISION_FACTOR_E36
        / PRECISION_FACTOR_YEAR;


```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L302C1-L302C87


```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

58    uint256 internal constant PRECISION_FACTOR_E36 = PRECISION_FACTOR_E18 * PRECISION_FACTOR_E18;

59    uint256 internal constant PRECISION_FACTOR_YEAR = PRECISION_FACTOR_E18 * ONE_YEAR;

63    uint256 internal constant RESTRICTION_FACTOR = 10
        * PRECISION_FACTOR_E36
        / PRECISION_FACTOR_YEAR;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L58C1-L59C87


```solidity
file: PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.so

101    uint256 internal constant MAX_LEVERAGE = 15 * PRECISION_FACTOR_E18;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L101C1-L101C72


```solidity
file: /FeeManager/DeclarationsFeeManager.sol
 
172    uint256 public constant INCENTIVE_PORTION = 5 * PRECISION_FACTOR_E15;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L172C1-L172C74


```solidity
file: /DerivativeOracles/PendleLpOracle.sol

67    uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L67C1-L67C60

## [G-15] Use assembly to emit events

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


## [G-16] Using ERC721A instead of ERC721 for more gas-efficient 

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


## [G-17] Avoid fetching a low-level call's return data by using assembly

Even if you don't assign the call's second return value, it still gets copied to memory. Use assembly instead to prevent this and save 159 gas:
`(bool success,) = payable(receiver).call{gas: gas, value: value}("");` -> `bool success; assembly { success := call(gas, receiver, value, 0, 0, 0, 0) }`

```solidity
file: /WiseSecurity/WiseSecurity.sol

206            bool success
            ,
        ) = curvePool.call{value: 0} (

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L206C1-L208C39


```solidity
file: /WrapperHub/AaveHelper.sol

208        (bool success, ) = payable(_recipient).call{
            value: _amount
        }("");

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L208C1-L210C15


```solidity
file: /TransferHub/SendValueHelper.sol

25            bool success
            ,
        ) = payable(_recipient).call{
            value: _amount
        }("");

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L25C1-L29C15



## [G-18] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
file: /MainHelper.sol

42        return _maxSharePrice == true
            ? _product / _pseudo + 1
            : _product / _pseudo - 1;

127        return product / totalBorrowShares + 1;            

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L42C1-L44C38


```solidity
file: PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

354                storageMarket.observationCardinalityNext + 1

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L354C1-L354C61


```solidity
file: /PowerFarms/PendlePowerFarmController/PendlePowerFarmController.so

194        if (balance < totalAssets + 1) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L194C1-L194C41


```solidity
file: /PositionNFTs.sol

124        totalReserved =
        totalReserved + 1;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L124C1-L125C27


## [G-19] bytes.concat() can be used in place of abi.encodePacked

Given concatenation is not going to be used for hashing bytes.concat is the preferred method to use as its more gas efficient

```solidity
file: /MainHelper.sol

655        return keccak256(
            abi.encodePacked(
                _nftId,
                _poolToken
            )
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L655C1-L660C11


