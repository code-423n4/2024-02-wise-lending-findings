# GAS OPTIMIZATION

# SUMMARY

|        | issue | instance |
| ------ | ----- | -------- |
| [G‑01] |Use calldata instead of memory for function arguments that do not get mutated|5|
| [G‑02] | Gas Optimization: Declare Variables Outside the Loop|2|
| [G‑03] |Amounts should be checked for 0 before calling a transfer|1|
| [G‑04] |Do not calculate constants|15|
| [G‑05] |Change public state variable visibility to private|11|
| [G‑06] | With assembly, .call (bool success) transfer can be done gas-optimized|5|
| [G‑07] |Use ERC721A instead ERC721|1|
| [G‑08] |Use of emit inside a loop|1|
| [G‑09] |multiplications of powers of 2 can be replaced by a left shift operation to save gas|3|
| [G‑10] |Gas Optimization Issue - Gas-Efficient Alternatives for Common Math Operations|9|
| [G‑11] |Use assembly to perform efficient back-to-back calls|7|
| [G‑12] |Use assembly to perform efficient back-to-back calls|14|
| [G‑13] |Functions guaranteed to revert when called by normal users can be marked payable|58|
| [G‑14] |State variables only set in the constructor should be declared immutable|7|
| [G‑15] |Use Modifiers Instead of Functions To Save Gas|12|
| [G‑16] |Use selfbalance() instead of address(this).balance|2|
| [G‑17] |Replace state variable reads and writes within loops with local variable reads and writes.|3|
| [G‑18] |Avoid contract existence checks by using low level calls|8|
| [G‑19] | Refactor external/internal function to avoid unnecessary External Calls|4|
| [G‑20] |Optimize External Calls with Assembly for Memory Efficiency|16|
| [G‑21] | Pre-increment and pre-decrement are cheaper thanĀ +1 ,-1|6|
| [G‑22] |Consider merging sequential loops|1|
| [G‑23] |Move storage pointer to top of function to avoid offset calculation|1|
| [G‑24] |Consider using OZ EnumerateSet in place of nested mappings|2|
| [G‑25] |Shorten the array rather than copying to a new one|4|

## [G-01] Use calldata instead of memory for function arguments that do not get mutated

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.


```solidity
File:  contracts/FeeManager/FeeManager.sol
571 function revokeBeneficial(
        address _user,
        address[] memory _feeTokens
    )
        external
        onlyMaster
    {
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L571-L577

```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol
59  function receiveFlashLoan(
        IERC20[] memory _flashloanToken,
        uint256[] memory _flashloanAmounts,
        uint256[] memory _feeAmounts,
        bytes memory _userData
    )
        external
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L59-L65

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
211 function addPendleMarket(
        address _pendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
        onlyMaster
    {
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L211-L219

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
682 function initialize(
        address _underlyingPendleMarket,
        address _pendleController,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
    {
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L682-L690

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol
35  function deploy(
        address _underlyingPendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L35-L40

## [G-02] Gas Optimization: Declare Variables Outside the Loop

**Explanation of Issue:**
When variables are declared inside a loop in Solidity, a new instance of the variable is created for each iteration of the loop. This can result in unnecessary gas costs, especially when the loop is executed frequently or iterates over a large number of elements. To optimize gas usage, it is advisable to declare the variable outside the loop and initialize it to zero to avoid the creation of multiple instances.



There are 2 instances of this issue:

```solidity
File:  contracts/WrapperHub/AaveHelper.sol
246  address currentAddress = WISE_LENDING.getPositionLendingTokenByIndex(

282  address currentAddress = WISE_LENDING.getPositionBorrowTokenByIndex(
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L246

## [G-03] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.

Note:  missed from bots

There are 1 instances of this issue:

```solidity
File:  contracts/TransferHub/TransferHelper.sol
20       _callOptionalReturn(
            _token,
            abi.encodeWithSelector(
                IERC20.transfer.selector,
                _to,
                _value  // found _value
            )
        );
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/TransferHelper.sol#L20-L27


```diff
    function _safeTransfer(
        address _token,
        address _to,
        uint256 _value
    )
        internal
    {
        if(_value > 0) {
           _callOptionalReturn(
                _token,
                abi.encodeWithSelector(
                IERC20.transfer.selector,
                _to,
                 _value
            )
        );
        }

    }

```

## [G-04] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas
[Reffrence](https://code4rena.com/reports/2022-07-golom#g-24-do-not-calculate-constants)


```solidity
File:  contracts/DerivativeOracles/PendleLpOracle.sol
59   uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L59

```solidity
File:  contracts/FeeManager/DeclarationsFeeManager.sol
157   uint256 public constant INCENTIVE_PORTION = 5 * PRECISION_FACTOR_E15;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L157

```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol
95  uint256 internal constant MAX_LEVERAGE = 15 * PRECISION_FACTOR_E18;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L95

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
57    uint256 internal constant PRECISION_FACTOR_E36 = PRECISION_FACTOR_E18 * PRECISION_FACTOR_E18;
58    uint256 internal constant PRECISION_FACTOR_YEAR = PRECISION_FACTOR_E18 * ONE_YEAR;

62    uint256 internal constant RESTRICTION_FACTOR = 10 * PRECISION_FACTOR_E36 / PRECISION_FACTOR_YEAR;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L57-L62

```solidity
File:  contracts/WiseOracleHub/Declarations.sol
83  uint32 internal constant TWAP_PERIOD = 30 * SECONDS_IN_MINUTE;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol#L83


```solidity
File:  contracts/WiseSecurity/WiseSecurityDeclarations.sol
149    uint256 public constant BORROW_PERCENTAGE_CAP = 95 * PRECISION_FACTOR_E16;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L149

```solidity
File:  contracts/WiseSecurity/WiseSecurityDeclarations.sol
189 uint256 internal constant LIQUIDATION_INCENTIVE_MAX = 11 * PRECISION_FACTOR_E16;
    uint256 internal constant LIQUIDATION_INCENTIVE_MIN = 2 * PRECISION_FACTOR_E16;
    uint256 internal constant LIQUIDATION_INCENTIVE_POWERFARM_MAX = 4 * PRECISION_FACTOR_E16;

    // Liquidation Fee threshholds
    uint256 internal constant LIQUIDATION_FEE_MIN_ETH = 3 * PRECISION_FACTOR_E18;
    uint256 internal constant LIQUIDATION_FEE_MAX_ETH = 100 * PRECISION_FACTOR_E18;
    uint256 internal constant LIQUIDATION_FEE_MAX_NON_ETH = 10 * PRECISION_FACTOR_E18;
    uint256 internal constant LIQUIDATION_FEE_MIN_NON_ETH = 30 * PRECISION_FACTOR_E16;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L189-L197


## [G-05] Change public state variable visibility to private

it's generally a good practice to limit the visibility of state variables to the minimum necessary level. This means that you should use the private visibility modifier for state variables whenever possible, and avoid using the public modifier unless it is absolutely necessary.

1.Security: Public state variables can be read and modified by anyone on the blockchain, which can make your contract vulnerable to attacks. By using the private modifier, you can limit access to your state variables and reduce the risk of malicious actors exploiting them.

2.Encapsulation: Using private state variables can help to encapsulate the internal workings of your contract and make it easier to reason about and maintain. By only exposing the necessary interfaces to the outside world, you can reduce the complexity and potential for errors in your contract.

3.Gas costs: Public state variables can be more expensive to read and write than private state variables, since Solidity generates additional getter and setter functions for public variables. By using private state variables, you can reduce the gas cost of your contract and improve its efficiency.


```solidity
File:  contracts/DerivativeOracles/CustomOracleSetup.sol
    address public master;
    uint256 public lastUpdateGlobal;

    uint80 public globalRoundId;

    mapping(uint80 => uint256) public timeStampByRoundId;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L6-L11

```solidity
File:  contracts/FeeManager/DeclarationsFeeManager.sol
    uint256 public totalBadDebtETH;

    // Incentive percentage for paying back bad debt
    uint256 public paybackIncentive;

    // Array of pool tokens in wiseLending
    address[] public poolTokenAddresses;

    // Address of incentive master
    address public incentiveMaster;

    // Proposed incentive master (for changing)
    address public proposedIncentiveMaster;

    // Address of incentive owner A
    address public incentiveOwnerA;

    // Address of incentive owner B
    address public incentiveOwnerB;

    // ---- Mappings ----

    // Bad debt of a specific position
    mapping(uint256 => uint256) public badDebtPosition;

    // Amount of fee token inside feeManager
    mapping(address => uint256) public feeTokens;

    // Open incetive amount for incentiveOwner in ETH
    mapping(address => uint256) public incentiveUSD;

    // Flag that specific token is already added
    mapping(address => bool) public poolTokenAdded;

    // Flag for token being aToken
    mapping(address => bool) public isAaveToken;

    // Getting underlying token of aave aToken
    mapping(address => address) public underlyingToken;

    // Showing which token are allowed to claim for beneficial address
    mapping(address => mapping(address => bool)) public allowedTokens;

    // Gives claimable token amount for incentiveOwner per token
    mapping(address => mapping(address => uint256)) public gatheredIncentiveToken;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L102-L146


## [G-06] With assembly, .call (bool success) transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
https://code4rena.com/reports/2023-01-biconomy#g-01-with-assembly-call-bool-success-transfer-can-be-done-gas-optimized


```solidity
File:  contracts/TransferHub/CallOptionalReturn.sol
12  (bool success, bytes memory returndata) = token.call(data);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L12

```solidity
File:  contracts/TransferHub/SendValueHelper.sol
18  (bool success,) = payable(_recipient).call{value: _amount}("");
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L18

```solidity
File:  contracts/WiseSecurity/WiseSecurity.sol
128  (bool success,) = curvePool.call{value: 0}(curveSwapInfoData[_poolToken].swapBytesPool);

138  (success,) = curveMeta.call{value: 0}(curveSwapInfoData[_poolToken].swapBytesMeta);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L128

```solidity
File:  contracts/WrapperHub/AaveHelper.sol
115   (bool success,) = payable(_recipient).call{value: _amount}("");
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L115

## [G-07] Use ERC721A instead ERC721
ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum's sky-rocketing gas fee.
[Reffrence](https://nextrope.com/erc721-vs-erc721a-2/)


```solidity
File:  contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol
27  constructor(string memory _name, string memory _symbol) ERC721(_name, _symbol) OwnableMaster(msg.sender) {}
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L27

## [G-08] Use of emit inside a loop
Emitting an event inside a loop performs a LOG op N times, where N is the loop length. Consider refactoring the code to emit the event only once at the end of loop. Gas savings should be multiplied by the average loop length.


```solidity
File:  contracts/FeeManager/FeeManager.sol
104  emit PoolFeeChanged(_poolTokens[i], _newFees[i], block.timestamp);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L104






## [G-09] multiplications of powers of 2 can be replaced by a left shift operation to save gas

Replace such found multiplications with left shift operations when ensured it is safe to do so. NOTE: This only applies to uint variables!

Note: this instances missed from bots

```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol
127  uint256 reverseAllowedSpread = 2 * PRECISION_FACTOR_E18 - _allowedSpread;

246  uint256 reverseAllowedSpread = 2 * PRECISION_FACTOR_E18 - _allowedSpread;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L127

```solidity
File:  contracts/WiseSecurity/WiseSecurityDeclarations.sol
190  uint256 internal constant LIQUIDATION_INCENTIVE_MIN = 2 * PRECISION_FACTOR_E16;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L190



## [G-10] Gas Optimization Issue - Gas-Efficient Alternatives for Common Math Operations

Common math operations, such as min and max, may have more gas-efficient alternatives. In scenarios where unoptimized code uses conditional operators, like the ternary operator, the resulting conditional jumps in opcodes can incur higher gas costs. Exploring gas-efficient alternatives for these operations is crucial for optimizing smart contract performance.





```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol
177  isPositive == true ? leveragedPositivAPY - leveragedNegativeAPY : leveragedNegativeAPY - leveragedPositivAPY;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L177

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
203                ? indexDiff * activeBalance * totalLpAssetsCurrent / lpBalanceController / PRECISION_FACTOR_E18
                : indexDiff * activeBalance / PRECISION_FACTOR_E18;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L203-L204

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
318        return product % _underlyingLpAssetsCurrent == 0
            ? product / _underlyingLpAssetsCurrent
            : product / _underlyingLpAssetsCurrent + 1;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L318-L320

```solidity
File:  contracts/WiseOracleHub/WiseOracleHub.sol
86        return _decimalsETH < tokenDecimals
            ? _tokenAmount * latestResolver(_tokenAddress) / 10 ** decimals(_tokenAddress)
                / 10 ** (tokenDecimals - _decimalsETH)
            : _tokenAmount * 10 ** (_decimalsETH - tokenDecimals) * latestResolver(_tokenAddress)
                / 10 ** decimals(_tokenAddress);


112       return _decimalsETH < tokenDecimals
            ? _tokenAmount * latestResolver(_tokenAddress) / 10 ** decimals(_tokenAddress)
                / 10 ** (tokenDecimals - _decimalsETH)
            : _tokenAmount * 10 ** (_decimalsETH - tokenDecimals) * latestResolver(_tokenAddress)
                / 10 ** decimals(_tokenAddress);    


123      return _decimalsETH < tokenDecimals
            ? _usdValue * 10 ** (tokenDecimals - _decimalsETH) * 10 ** decimals(_tokenAddress)
                / latestResolver(_tokenAddress)
            : _usdValue * 10 ** decimals(_tokenAddress) / latestResolver(_tokenAddress)
                / 10 ** (_decimalsETH - tokenDecimals);     


214      return _decimalsETH < tokenDecimals
            ? _ethAmount * 10 ** (tokenDecimals - _decimalsETH) * 10 ** decimals(_tokenAddress)
                / latestResolver(_tokenAddress)
            : _ethAmount * 10 ** decimals(_tokenAddress) / latestResolver(_tokenAddress)
                / 10 ** (_decimalsETH - tokenDecimals);                                      
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L86-L90

```solidity
File:  contracts/WiseSecurity/WiseSecurityHelper.sol
451  return numerator % denominator == 0 ? numerator / denominator : numerator / denominator + 1;

517   uint256 maxShares = checkBadDebtThreshold(_borrowETHTotal, _unweightedCollateralETH)
            ? totalSharesUser
            : totalSharesUser * MAX_LIQUIDATION_50 / PRECISION_FACTOR_E18;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L451



## [G-11] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.



```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol
88      uint256 withdrawAmount = WISE_LENDING.cashoutAmount(PENDLE_CHILD, _withdrawShares);

90      withdrawAmount = WISE_LENDING.withdrawExactShares(_nftId, PENDLE_CHILD, _withdrawShares);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L88-L90


```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol
327        paybackAmount = WISE_LENDING.paybackAmount(paybackToken, _shareAmountToPay);

329        receivingAmount = WISE_LENDING.coreLiquidationIsolationPools(
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L327-L329


```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol
187        uint256 totalPool = WISE_LENDING.getTotalPool(ENTRY_ASSET);

189        uint256 pseudoPool = WISE_LENDING.getPseudoTotalPool(ENTRY_ASSET);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L187

```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol
198        uint256 pole = WISE_LENDING.borrowRatesData(ENTRY_ASSET).pole;

200        uint256 mulFactor = WISE_LENDING.borrowRatesData(ENTRY_ASSET).multiplicativeFactor;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L198-L200

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
156        address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(UNDERLYING_PENDLE_MARKET);

158        uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(UNDERLYING_PENDLE_MARKET);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L156-L158

```solidity
File:  contracts/WiseSecurity/WiseSecurity.sol
618          uint256 lendingShares = WISE_LENDING.getPositionLendingShares(_nftId, _poolToken);

620          uint256 currentTotalLendingShares = WISE_LENDING.getTotalDepositShares(_poolToken);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L618-L620


```solidity
File:  contracts/WrapperHub/AaveHelper.sol
168            WISE_LENDING.preparePool(currentAddress);

170            WISE_LENDING.newBorrowRate(_poolToken);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L168-L170


## [G-12] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.


```solidity
File:  contracts/DerivativeOracles/CustomOracleSetup.sol
22  function renounceOwnership() external onlyOwner {
        master = address(0x0);
    }

    function setLastUpdateGlobal(uint256 _time) external onlyOwner {
        lastUpdateGlobal = _time;
    }

    function setRoundData(uint80 _roundId, uint256 _updateTime) external onlyOwner {
        timeStampByRoundId[_roundId] = _updateTime;
    }

    function setGlobalAggregatorRoundId(uint80 _aggregatorRoundId) external onlyOwner {
        globalRoundId = _aggregatorRoundId;
36  }
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L22-L36

```solidity
File:  contracts/FeeManager/FeeManager.sol
46  function setRepayBadDebtIncentive(uint256 _percent) external onlyMaster {

56  function setAaveFlag(address _poolToken, address _underlyingToken) external onlyMaster {

63  function setAaveFlagBulk(address[] calldata _poolTokens, address[] calldata _underlyingTokens)
        external
        onlyMaster
    {   

83  function setPoolFee(address _poolToken, uint256 _newFee) external onlyMaster {

95  function setPoolFeeBulk(address[] calldata _poolTokens, uint256[] calldata _newFees) external onlyMaster {

117 function proposeIncentiveMaster(address _proposedIncentiveMaster) external onlyIncentiveMaster {

145 function increaseIncentiveA(uint256 _value) external onlyIncentiveMaster {

155 function increaseIncentiveB(uint256 _value) external onlyIncentiveMaster {

265 function addPoolTokenAddress(address _poolToken) external onlyWiseLending {   

277 function addPoolTokenAddressManual(address _poolToken) external onlyMaster {

293 function removePoolTokenManual(address _poolToken) external onlyMaster {

338 function increaseTotalBadDebtLiquidation(uint256 _amount) external onlyWiseSecurity {

348 function setBadDebtUserLiquidation(uint256 _nftId, uint256 _amount) external onlyWiseSecurity {

359 function setBeneficial(address _user, address[] calldata _feeTokens) external onlyMaster {

378 function revokeBeneficial(address _user, address[] memory _feeTokens) external onlyMaster{                                                          
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L46

```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol
50   function changeMinDeposit(uint256 _newMinDeposit) external onlyMaster {

61   function shutDownFarm(bool _state) external onlyMaster {

124 function exitFarm(uint256 _keyId, uint256 _allowedSpread, bool _ethBack)
        external
        updatePools
        onlyKeyOwner(_keyId)
    {

154 function manuallyWithdrawShares(uint256 _keyId, uint256 _withdrawShares)
        external
        updatePools
        onlyKeyOwner(_keyId)
    {                
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L50

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
23    function withdrawLp(address _pendleMarket, address _to, uint256 _amount)
        external
        onlyChildContract(_pendleMarket)
    {

110    function addPendleMarket(
        address _pendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    ) external onlyMaster {

158  function updateRewardTokens(address _pendleMarket) external onlyChildContract(_pendleMarket) returns (bool) {  

172  function changeExchangeIncentive(uint256 _newExchangeIncentive) external onlyMaster {  

178  function changeMintFee(address _pendleMarket, uint256 _newFee) external onlyMaster {     

193    function lockPendle(uint256 _amount, uint128 _weeks, bool _fromInside, bool _sameExpiry)
        external
        onlyMaster
        returns (uint256 newVeBalance)
    {

231  function claimArb(uint256 _accrued, bytes32[] calldata _proof) external onlyArbitrum {  

237  function withdrawLock() external onlyMaster returns (uint256 amount) {  

264    function increaseReservedForCompound(address _pendleMarket, uint256[] calldata _amounts)
        external
        onlyChildContract(_pendleMarket)
    {

285 function overWriteIndex(address _pendleMarket, uint256 _tokenIndex) public onlyChildContract(_pendleMarket) {   

292  function overWriteIndexAll(address _pendleMarket) external onlyChildContract(_pendleMarket) {

304 function overWriteAmounts(address _pendleMarket) external onlyChildContract(_pendleMarket) {

316 function forwardETH(address _to, uint256 _amount) external onlyMaster {

320 function vote(address[] calldata _pools, uint64[] calldata _weights) external onlyMaster {                                                   
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L23

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
378 function changeMintFee(uint256 _newFee) external onlyController {
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L378

```solidity
File:  contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol
29  function setFarmContract(address _farmContract) external onlyMaster {

39  function mintKey(address _keyOwner, uint256 _keyId) external onlyFarmContract {

46  function burnKey(uint256 _keyId) external onlyFarmContract {

92  function setBaseURI(string calldata _newBaseURI) external onlyMaster {

96  function setBaseExtension(string calldata _newBaseExtension) external onlyMaster {                
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L29

```solidity
File:  contracts/WiseOracleHub/WiseOracleHub.sol
142 function addTwapOracle(
        address _tokenAddress,
        address _uniPoolAddress,
        address _token0,
        address _token1,
        uint24 _fee
    ) external onlyMaster {

166 function addTwapOracleDerivative(
        address _tokenAddress,
        address _partnerTokenAddress,
        address[2] calldata _uniPools,
        address[2] calldata _token0Array,
        address[2] calldata _token1Array,
        uint24[2] calldata _feeArray
    ) external onlyMaster {     

226 function addOracle(address _tokenAddress, IPriceFeed _priceFeedAddress, address[] calldata _underlyingFeedTokens)
        external
        onlyMaster
    {  

238 function addOracleBulk(
        address[] calldata _tokenAddresses,
        IPriceFeed[] calldata _priceFeedAddresses,
        address[][] calldata _underlyingFeedTokens
    ) external onlyMaster {                 
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L142

```solidity
File:  contracts/WiseSecurity/WiseSecurity.sol
61  function setLiquidationSettings(
        uint256 _baseReward,
        uint256 _baseRewardFarm,
        uint256 _newMaxFeeETH,
        uint256 _newMaxFeeFarmETH
    ) external onlyMaster {

89  function prepareCurvePools(
        address _poolToken,
        CurveSwapStructData calldata _curveSwapStructData,
        CurveSwapStructToken calldata _curveSwapStructToken
    ) external onlyWiseLending {        

121 function curveSecurityCheck(address _poolToken) external onlyWiseLending {

259 function checkBadDebtLiquidation(uint256 _nftId) external onlyWiseLending {

636 function setBlacklistToken(address _tokenAddress, bool _state) external onlyMaster {

645 function setSecurityWorker(address _entitiy, bool _state) external onlyMaster {

669 function revokeShutdown() external onlyMaster {

697  function changeMinDepositValue(uint256 _newMinDepositValue) external onlyMaster {                            
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L142

```solidity
File:  contracts/WrapperHub/AaveHub.sol
33  function setAaveTokenAddress(address _underlyingAsset, address _aaveToken) external onlyMaster {

42  function setAaveTokenAddressBulk(address[] calldata _underlyingAssets, address[] calldata _aaveTokens)
        external
        onlyMaster
    {    
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L33

```solidity
File:  contracts/WrapperHub/Declarations.sol
72  function setWiseSecurity(address _securityAddress) external onlyMaster {
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L72






## [G‑14] State variables only set in the constructor should be declared immutable
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas). 

```solidity 
File:  contracts/DerivativeOracles/PendleChildLpOracle.sol
14    IPriceFeed public priceFeedPendleLpOracle;
15    IPendleChildToken public pendleChildToken;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleChildLpOracle.sol#L14-L15

```solidity
File:  contracts/DerivativeOracles/PendleLpOracle.sol
68  string public name;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L68

```solidity
File:  contracts/DerivativeOracles/PtOracleDerivative.sol
71  string public name;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L71

```solidity
File:  contracts/DerivativeOracles/PtOraclePure.sol
57   string public name;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L57

```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol
43  uint256 public collateralFactor;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L43

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol
68  IArbRewardsClaimer public ARB_REWARDS;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L68

## [G-15] Use Modifiers Instead of Functions To Save Gas




```solidity
File:  contracts/FeeManager/FeeManager.sol
47  _checkValue(_percent);

84  _checkValue(_newFee);
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L47

```solidity
File:  contracts/WiseSecurity/WiseSecurityHelper.sol
485   checkOwnerPosition(_nftId, _caller);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L485

```solidity
File:  contracts/WrapperHub/AaveHub.sol
126   _checkOwner(_nftId);

140   _checkOwner(_nftId);

163   _checkOwner(_nftId);

177   _checkOwner(_nftId);

203   _checkOwner(_nftId);

220   _checkOwner(_nftId);

243   _checkPositionLocked(_nftId);

263   _checkPositionLocked(_nftId);

300   _checkPositionLocked(_nftId);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L126


## [G-16] Use selfbalance() instead of address(this).balance
it's recommended to use the selfbalance() function instead of address(this).balance. The selfbalance() function is a built-in Solidity function that returns the balance of the current contract in Wei and is considered more gas-efficient and secure.


```solidity
File:  contracts/TransferHub/SendValueHelper.sol
12  if (address(this).balance < _amount) {
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/SendValueHelper.sol#L12

```solidity
File:  contracts/WrapperHub/AaveHelper.sol
109   if (address(this).balance < _amount) {
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L109


## [G-17] Replace state variable reads and writes within loops with local variable reads and writes.
Reading and writing local variables is cheap, whereas reading and writing state variables that are stored in contract storage is expensive.





```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
166     while (i < l) {
            UserReward memory userReward = _getUserReward(rewardTokens[i], PENDLE_POWER_FARM_CONTROLLER);

            if (userReward.accrued > 0) {
                PENDLE_MARKET.redeemRewards(PENDLE_POWER_FARM_CONTROLLER);

                userReward = _getUserReward(rewardTokens[i], PENDLE_POWER_FARM_CONTROLLER);
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L166-L172


## [G‑18] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

Note: missed from bots

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
79   uint256 withdrawnAmount = IPendlePowerFarmToken(pendleChild).withdrawExactShares(_pendleChildShares);

97   uint256 totalAssets = IPendlePowerFarmToken(childMarket).totalLpAssets();

185  IPendlePowerFarmToken(child).changeMintFee(_newFee);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L79

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol
175  IPendlePowerFarmToken(child).manualSync();
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L175

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol
79  return IPendleMarket(_pendleMarket).getRewardTokens();

87  return IPendleMarket(_pendleMarket).userReward(_rewardToken, _user);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L79

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol
59  PendlePowerFarmToken(pendlePowerFarmTokenAddress).initialize(
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L59

```solidity
File:  contracts/WiseOracleHub/OracleHelper.sol
149   (int56[] memory tickCumulatives,) = IUniswapV3Pool(_oracle).observe(secondsAgo);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L149


## [G-19] Refactor external/internal function to avoid unnecessary External Calls
The functions below perform external calls that are previously performed in the functions that invoke them. We can refactor the external/internal functions so we could pass the cached external calls into them and avoid the extra external calls that would otherwise take place in the internal functions.

```solidity
File:  contracts/WiseSecurity/WiseSecurity.sol
100          _safeApprove(ICurve(curvePool).coins(tokenIndexForApprove), curvePool, 0);

102         _safeApprove(ICurve(curvePool).coins(tokenIndexForApprove), curvePool, UINT256_MAX);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L100-L102

- `ICurve(curveMetaPool).coins(tokenIndexForApprove)` is external call cahce external call then use to save gas
```solidity
File:  contracts/WiseSecurity/WiseSecurity.sol
112        _safeApprove(ICurve(curveMetaPool).coins(tokenIndexForApprove), curveMetaPool, 0);

114        _safeApprove(ICurve(curveMetaPool).coins(tokenIndexForApprove), curveMetaPool, UINT256_MAX);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L112-L114


## [G-20] Optimize External Calls with Assembly for Memory Efficiency
Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.
Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.
Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.

There are 16 instances of this issue:


```solidity
File:  contracts/FeeManager/FeeManager.sol
86  WISE_LENDING.setPoolFee(_poolToken, _newFee);

490 WISE_LENDING.corePaybackFeeManager(_paybackToken, _nftId, paybackAmount, _shares);

526 WISE_LENDING.corePaybackFeeManager(_paybackToken, _nftId, paybackAmount, _shares);
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L86

```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol
77  WISE_LENDING.paybackExactShares(_nftId, poolAddress, _paybackShares);

161 WISE_LENDING.setRegistrationIsolationPool(_nftId, true);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L77


```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol
200  AAVE_HUB.paybackExactShares(_nftId, WETH_ADDRESS, _borrowShares);

205  WISE_LENDING.paybackExactShares(_nftId, WETH_ADDRESS, _borrowShares);

291  WISE_LENDING.depositExactAmount(_nftId, PENDLE_CHILD, receivedShares);

304  AAVE_HUB.borrowExactAmount(_nftId, WETH_ADDRESS, _totalDebtBalancer);

309  WISE_LENDING.borrowExactAmount(_nftId, WETH_ADDRESS, _totalDebtBalancer);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L200

```solidity
File:  contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol
116   POSITION_NFT.approve(AAVE_HUB_ADDRESS, nftId);

138   FARMS_NFTS.burnKey(_keyId);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L116


```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
52  IPendlePowerFarmToken(pendleChildAddress[_pendleMarket]).addCompoundRewards(sendingAmount);

185 IPendlePowerFarmToken(child).changeMintFee(_newFee);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L52

```solidity
File:  contracts/TransferHub/WrapperHelper.sol
19  WETH.deposit{value: _value}();

27  WETH.withdraw(_value);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/WrapperHelper.sol#L19


## [G-21] Pre-increment and pre-decrement are cheaper thanĀ +1 ,-1


```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
99  if (balance < totalAssets + 1) {

103  uint256 difference = balance - totalAssets + 1;    
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L99

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
243   PENDLE_MARKET.increaseObservationsCardinalityNext(storageMarket.observationCardinalityNext + 1);

320  : product / _underlyingLpAssetsCurrent + 1;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L243

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
451  return numerator % denominator == 0 ? numerator / denominator : numerator / denominator + 1;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L451

```solidity
File:  contracts/FeeManager/FeeManager.sol
296   uint256 lastEntry = len - 1;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L296

## [G-22] Consider merging sequential loops
Merging multiple loops within a function in Solidity can enhance efficiency and reduce gas costs, especially when they share a common iterating variable or perform related operations. By minimizing redundant iterations over the same data set, execution becomes more cost-effective. However, while merging can optimize gas usage and simplify logic, it may also increase code complexity. Therefore, careful balance between optimization and maintainability is essential, along with thorough testing to ensure the refactored code behaves as expected.

```solidity
File:  contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol
        while (j != 0) {
            length++;
            j /= 10;
        }

        bytes memory bstr = new bytes(length);

        uint256 k = length;
        j = _tokenId;

        while (j != 0) {
            bstr[--k] = bytes1(uint8(48 + j % 10));
            j /= 10;
        }
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L126-L129


## [G-23] Move storage pointer to top of function to avoid offset calculation
We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
127  CompoundStruct storage childInfo = pendleChildCompoundInfo[_pendleMarket];
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L127


## [G-24] Consider using OZ EnumerateSet in place of nested mappings             
Nested mappings and multi-dimensional arrays in Solidity operate through a process of double hashing, wherein the original storage slot and the first key are concatenated and hashed, and then this hash is again concatenated with the second key and hashed. This process can be quite gas expensive due to the double-hashing operation and subsequent storage operation (sstore).
A possible optimization involves manually concatenating the keys followed by a single hash operation and an sstore. However, this technique introduces the risk of storage collision, especially when there are other nested hash maps in the contract that use the same key types. Because Solidity is unaware of the number and structure of nested hash maps in a contract, it follows a conservative approach in computing the storage slot to avoid possible collisions.
OpenZeppelin's EnumerableSet provides a potential solution to this problem. It creates a data structure that combines the benefits of set operations with the ability to enumerate stored elements, which is not natively available in Solidity. EnumerableSet handles the element uniqueness internally and can therefore provide a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays in certain scenarios.

```solidity
File:  contracts/FeeManager/DeclarationsFeeManager.sol
143  mapping(address => mapping(address => bool)) public allowedTokens;

146  mapping(address => mapping(address => uint256)) public gatheredIncentiveToken;
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L143


## [G-25] Shorten the array rather than copying to a new one
ASSEMBLY can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File:  contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
131        childInfo.lastIndex = new uint128[](rewardTokensLength);

133        childInfo.reservedForCompound = new uint256[](rewardTokensLength);

307        childInfo.reservedForCompound = new uint256[](childInfo.rewardTokens.length);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L131

```solidity
File:  contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol
70  uint256[] memory tokenIds = new uint256[](ownerTokenCount + reservedCount);
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L70


