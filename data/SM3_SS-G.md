## Gas Optimizations

| Number | Issue | Instences |
|--------|-------|-----------|
|[G-01]| Using bytes32 is cheaper than using string | 28 |
|[G-02]| Store Data in calldata Instead of Memory for Certain Function Parameters  | 19 |
|[G-03]| Use assembly to emit events | 75 |
|[G-04]| Use the external Visibility Modifier | 1 |
|[G-05]| Avoid Unnecessary Use of Storage | 7 |
|[G-06]| Precompute Off-Chain When Possible | 2 |
|[G-07]| State variables only set in the constructor should be declared immutable | 10 |
|[G-08]| Do not calculate constants  | 23 |
|[G-09]| Redundant state variable getters | 1 |
|[G-10]| Check if amount is greater than zero before performing safeTransfer | 10 |
|[G-11]| State variables should be cached in stack variables rather than re-reading them from storage | 2 |
|[G-12]| Use constants instead of type(uintX).max  | 3 |
|[G-13]| Calculation(addition, subtruction) which won’t underflow can be unchecked in | 13 |
|[G-14]| Using Storage Instead of Memory for structs/arrays Saves Gas | 13 |
|[G-15]| Use named returns for local variables of pure functions where it is possible | 6 |
|[G-16]| Using assembly to revert with an error message | 4 |
|[G-17]| Refactor functions to combine events to reduce gas cost | 2 |
|[G-18]| Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4 | 6 |
|[G-19]| Using > 0 costs more gas than != 0 when used on a uint in a if() statement | 21 |
|[G-20]| Functions guaranteed to revert when called by normal users can be marked payable | 4 |
|[G-21]| require()/revert() strings longer than 32 bytes cost extra gas | 4 |

## [G-01] Using bytes32 is cheaper than using string. 

Storing and manipulating data in bytes32 is more gas-efficient than using string, which involves dynamic storage allocation. bytes32 is a fixed-size type, whereas string can have variable length, resulting in higher gas costs.

code snapet

<details>

```solidity
file: blob/main/contracts/DerivativeOracles/PtOraclePure.sol

60  string public name;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L60

```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

26   string public baseURI;

27   string public baseExtension;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L26

```solidity
file: blob/main/contracts/PositionNFTs.sol

13  string public baseURI;

14  string public baseExtension;

26  string memory _name,

27  string memory _symbol,

28  string memory _baseURI

320 string memory _newBaseURI

329 string memory _newBaseExtension

353 string memory currentBaseURI = baseURI;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L13


```solidity
file: blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol

74  string public name;

29  string memory _oracleName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L74

```solidity
file: blob/main/contracts/DerivativeOracles/PendleLpOracle.sol

76  string public name;

36  string memory _oracleName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L76


```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

39   string memory _name,

40   string memory _symbol

162  string calldata _newBaseURI

171  string calldata _newBaseExtension

195  string memory currentBaseURI = baseURI;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L39

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

213   string memory _tokenName,

214   string memory _symbolName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L213

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

685   string memory _tokenName,

686   string memory _symbolName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L685


```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

37   string memory _tokenName,

38   string memory _symbolName,

58   string memory _tokenName,

59   string memory _symbolName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L37

</details>

## [G-02] Store Data in calldata Instead of Memory for Certain Function Parameters 

Instead of copying variables to memory, it is typically more cost-effective to load them immediately from calldata. If all you need to do is read data, you can conserve gas by saving the data in calldata.

code snapet

<details>

```solidity
file: blob/main/contracts/MainHelper.sol

417   function _prepareTokens(
        address _poolToken,
        address[] memory _tokens
    ){

454   function _curveSecurityChecks(
        address[] memory _lendTokens,
        address[] memory _borrowTokens
    )

473   function _whileLoopCurveSecurity(
        address[] memory _tokens
    )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L417-L420

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

59  function receiveFlashLoan(
        IERC20[] memory _flashloanToken,
        uint256[] memory _flashloanAmounts,
        uint256[] memory _feeAmounts,
        bytes memory _userData
    )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L59-L64

```solidity
file: blob/main/contracts/PositionNFTs.sol

25  constructor(
        string memory _name,
        string memory _symbol,
        string memory _baseURI
    )

319 function setBaseURI(
        string memory _newBaseURI
    )

328  function setBaseExtension(
        string memory _newBaseExtension
    )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L25-L29

```solidity
file: blob/main/contracts/DerivativeOracles/PendleLpOracle.sol

36   string memory _oracleName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L36

```solidity
file: blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol

29  string memory _oracleName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L29

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

219  address[] memory rewardTokensToCompare

238  address[] memory _rewardTokens

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol#L219

```solidity
file: blob/main/contracts/DerivativeOracles/PtOraclePure.sol

28  string memory _oracleName,

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L28

```solidity
file: blob/main/contracts/FeeManager/FeeManager.sol
 
573  address[] memory _feeTokens

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L573

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

211  function addPendleMarket(
        address _pendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L211-L216

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

35  function deploy(
        address _underlyingPendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )

56   function _clone(
        address _underlyingPendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L35-L40

```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

38  constructor(
        string memory _name,
        string memory _symbol
    )

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L38-L41

</details>

## [G-03] Use assembly to emit events

This detector checks for instances where a contract uses assembly code to emit events. While it’s technically possible to emit events using inline assembly in Solidity, it’s generally discouraged due to readability concerns and potential for errors. Events should usually be emitted using higher-level Solidity constructs.

code snapet

<details>

```solidity
file: blob/main/contracts/OwnableMaster.sol

82   emit MasterProposed(

110  emit RenouncedOwnership(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol#L82


```solidity 
file: blob/main/contracts/WiseCore.sol

77  emit FundsWithdrawnOnBehalf(

86  emit FundsWithdrawn(

156 emit FundsDeposited(

398 emit FundsBorrowedOnBehalf(

407 emit FundsBorrowed(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L77


```solidity
file: blob/main/contracts/WiseLending.sol

136  emit FundsSolelyWithdrawn(

153  emit FundsSolelyDeposited(

1450 emit FundsReturned(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L136

```solidity
file: blob/main/contracts/FeeManager/FeeManager.sol

124  emit PoolFeeChanged(

156  emit PoolFeeChanged(

185  emit IncentiveMasterProposed(

204  emit ClaimedOwnershipIncentiveMaster(

222  emit IncentiveIncreasedA(

240  emit IncentiveIncreasedB(

276  emit ClaimedIncentivesBulk(

297  emit ClaimedIncentives(

342  emit IncentiveOwnerAChanged(

379  emit IncentiveOwnerBChanged(

402  emit PoolTokenAdded(

428  emit PoolTokenAdded(

478  emit PoolTokenRemoved(

504  emit BadDebtIncreasedLiquidation(

526  emit SetBadDebtPosition(

560  emit SetBeneficial(

593  emit RevokeBeneficial(

619  emit ClaimedFeesWiseBulk(

677  emit ClaimedFeesWise(

716  emit ClaimedFeesBeneficial(

801  emit PayedBackBadDebt(

852  emit PayedBackBadDebtFree(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L124

```solidity
file: blob/main/contracts/FeeManager/FeeManagerHelper.sol

96   emit TotalBadDebtIncreased(

112  emit TotalBadDebtDecreased(

161  emit UpdateBadDebtPosition(

183  emit UpdateBadDebtPosition(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L86

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol

284  emit RegistrationFarm(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L284


```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

25   emit ETHReceived(

71   emit MinDepositChange(

90   emit FarmStatus(

131  emit FarmEntry(

174  emit FarmEntry(

246  emit FarmExit(

266  emit ManualPaybackShares(

296  emit ManualWithdrawShares(
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L25

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

46   emit WithdrawLp(

103  emit ExchangeRewardsForCompounding(

159  emit ExchangeLpFeesForPendle(

287  emit AddPendleMarket(

313  emit UpdateRewardTokens(

329  emit ChangeExchangeIncentive(

355  emit ChangeMintFee(

421  emit LockPendle(

444  emit ClaimArb(

467  emit WithdrawLock(

491  emit WithdrawLock(

520  emit IncreaseReservedForCompound(

593  emit ClaimVoteRewards(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L46


```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

129   emit ETHReceived(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L129

```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

169  emit ERC721Received(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L169


```solidity
file: blob/main/contracts/WiseSecurity/WiseSecurity.sol

1022  emit SecurityShutdown(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1022


```solidity
file: blob/main/contracts/WrapperHub/AaveHub.sol

145  emit IsDepositAave(

190  emit IsDepositAave(

226  emit IsWithdrawAave(

269  emit IsWithdrawAave(

302  emit IsWithdrawAave(

342  emit IsWithdrawAave(

378  emit IsBorrowAave(

421  emit IsBorrowAave(

470  emit IsPaybackAave(

544  emit IsPaybackAave(

603  emit IsPaybackAave(

688  emit SetAaveTokenAddress(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L145

</details>

## [G-04] Use the external Visibility Modifier

Use the external function visibility(https://www.alchemy.com/overviews/solidity-function-visibility) for gas optimization because the public visibility modifier is equivalent to using the external and internal visibility modifier, meaning both public and external can be called from outside of your contract, which requires more gas.

Remember that of these two visibility modifiers, only the public modifier can be called from other functions inside of your contract.

```solidity
function one() public view returns (string memory){
         return message;
    }

 
    function two() external view returns  (string memory){
         return message;
    }

```

```solidity
file: blob/main/contracts/FeeManager/FeeManager.sol

872  function getPoolTokenAddressesLength()
        public
        view
        returns (uint256)
    {
        return poolTokenAddresses.length;
    }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L872-L878

## [G-05] Avoid Unnecessary Use of Storage

Writing data to storage is one of the most expensive operations in Solidity. Reduce gas costs by avoiding unnecessary writes. For example, move constant values to memory:

```solidity
// Expensive
string public constant NAME = "MyContract"; 

// Cheaper
string memory NAME = "MyContract";
```
Also be careful about storing large pieces of data on-chain. Use alternative patterns like IPFS for larger data.

1. Storing data costs 20,000 gas versus 3 gas for memory.
2. Minimize storage by using memory for fixed constants and temporary values.

```solidity
file: blob/main/contracts/PositionNFTs.sol
 
13   string public baseURI;

14   string public baseExtension;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L13

```solidity
file: blob/main/contracts/DerivativeOracles/PendleLpOracle.sol

76  string public name;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L76


```solidity
file: blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol

76  string public name;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L76

```solidity
file: blob/main/contracts/DerivativeOracles/PtOraclePure.sol

60  string public name;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L60


```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

26    string public baseURI;

27    string public baseExtension;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L26


## [G-06] Precompute Off-Chain When Possible

Computing values in advance outside the blockchain is cheaper than doing it inside smart contracts:

```solidity
// On-chain computation
function verify(uint numA, uint numB) {
  require(numA * numB < 1000);
}

// Precomputed off-chain
function verify(uint result) {
  require(result < 1000); 
}
```
Do computations like hashes, signatures outside.
Pass in precomputed values to save on-chain gas.

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

194  if (balance < totalAssets + 1) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L194

```solidity
file: blob/main/contracts/WiseOracleHub/OracleHelper.sol

439  if (tickCumulativesDelta < 0 && (tickCumulativesDelta % twapPeriodInt56 != 0)) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L439

## [G-07] State variables only set in the constructor should be declared immutable

State variables only set in the constructor should be declared immutable as it avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

The paybackIncentive state variable could be made immutable

```solidity
file: blob/main/contracts/FeeManager/DeclarationsFeeManager.sol

120  uint256 public paybackIncentive;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L120

```diff
-  uint256 public paybackIncentive;
+  uint256 public immutable paybackIncentive;
```

```solidity
file: blob/main/contracts/FeeManager/DeclarationsFeeManager.sol

132  address public incentiveOwnerA;

135  address public incentiveOwnerB;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L132

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

49  uint256 public collateralFactor;

52  uint256 public minDepositEthAmount;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L49

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol

66  uint256 public exchangeIncentive;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.sol#L66

```solidity
file: blob/main/contracts/FeeManager/DeclarationsFeeManager.sol

146   mapping(address => uint256) public incentiveUSD;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L146

The name state variable could be made immutable

```solidity
file: blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol

74  string public name;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L74

Recommandation:

```diff
-   string public name;
+   string public immutable name;
```

```solidity
file: blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol

247   mapping(address => bool) public securityWorker;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L247

The name state variable could be made immutable

```solidity
file:  blob/main/contracts/DerivativeOracles/PtOraclePure.sol

60  string public name;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L60

## [G-08] Do not calculate constants 

While constants have known value, to save more gas, calculate their values before compile-time.

```solidity
file: blob/main/contracts/WiseLendingDeclaration.sol

170  uint256 internal constant MIN_BORROW_SHARE_PRICE = 5 * PRECISION_FACTOR_E18
        / 10;

297  uint256 internal constant PRECISION_FACTOR_E36 = PRECISION_FACTOR_E18 * PRECISION_FACTOR_E18;

302  uint256 internal constant PRECISION_FACTOR_YEAR = PRECISION_FACTOR_E18 * ONE_YEAR;

310  uint256 internal constant LOWER_BOUND_MAX_RATE = 100 * PRECISION_FACTOR_E16;

311  uint256 internal constant UPPER_BOUND_MAX_RATE = 300 * PRECISION_FACTOR_E16;

314  uint256 internal constant THRESHOLD_SWITCH_DIRECTION = 90 * PRECISION_FACTOR_E16;

315  uint256 internal constant THRESHOLD_RESET_RESONANCE_FACTOR = 75 * PRECISION_FACTOR_E16;

319  uint256 internal constant MAX_COLLATERAL_FACTOR = 85 * PRECISION_FACTOR_E16;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L170-L171


```solidity
file: blob/main/contracts/DerivativeOracles/PendleLpOracle.sol

67  uint256 internal constant POW_FEED_DECIMALS = 10 ** 18;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L67

```solidity
file: blob/main/contracts/FeeManager/DeclarationsFeeManager.sol

172  uint256 public constant INCENTIVE_PORTION = 5 * PRECISION_FACTOR_E15;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L172

```solidity 
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

62  uint256 internal constant MAX_COLLATERAL_FACTOR = 95 * PRECISION_FACTOR_E16;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L62

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

58  uint256 internal constant PRECISION_FACTOR_E36 = PRECISION_FACTOR_E18 * PRECISION_FACTOR_E18;

59  uint256 internal constant PRECISION_FACTOR_YEAR = PRECISION_FACTOR_E18 * ONE_YEAR;

63   uint256 internal constant RESTRICTION_FACTOR = 10
        * PRECISION_FACTOR_E36
        / PRECISION_FACTOR_YEAR;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L58

```solidity
file: blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol

185   uint256 public constant BORROW_PERCENTAGE_CAP = 95 * PRECISION_FACTOR_E16;

225   uint256 internal constant LIQUIDATION_INCENTIVE_MAX = 11 * PRECISION_FACTOR_E16;

226   uint256 internal constant LIQUIDATION_INCENTIVE_MIN = 2 * PRECISION_FACTOR_E16;

227   uint256 internal constant LIQUIDATION_INCENTIVE_POWERFARM_MAX = 4 * PRECISION_FACTOR_E16;

230   uint256 internal constant LIQUIDATION_FEE_MIN_ETH = 3 * PRECISION_FACTOR_E18;

231   uint256 internal constant LIQUIDATION_FEE_MAX_ETH = 100 * PRECISION_FACTOR_E18;

232   uint256 internal constant LIQUIDATION_FEE_MAX_NON_ETH = 10 * PRECISION_FACTOR_E18;

233   uint256 internal constant LIQUIDATION_FEE_MIN_NON_ETH = 30 * PRECISION_FACTOR_E16;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L185

## [G-09] Redundant state variable getters

Getters for public state variables are automatically generated by the solidity compiler so there is no need to code them manually as this increases deployment cost.

```solidity
file: blob/main/contracts/WiseLowLevelHelper.sol

39  function getTotalPool(
        address _poolToken
    )
        public
        view
        returns (uint256)
    {
        return globalPoolData[_poolToken].totalPool;
    }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L39-L47

## [G-10] Check if amount is greater than zero before performing safeTransfer

safeTransfer‘ing when amount = 0 is a waste of gas, since there will be no funds transferred. To save gas, check if amount is greater than 0, before calling safeTransfer.


```solidity
file: blob/main/contracts/FeeManager/FeeManager.sol

109  _safeTransferFrom(
            WETH_ADDRESS,
            msg.sender,
            address(this),
            _amount
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L109-L114


```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

607  payable(_to).transfer(
            _amount
        );

40  _safeTransfer(
            _pendleMarket,
            _to,
            _amount
        );
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L607-L609

```solidity
file: blob/main/contracts/WiseLending.sol

476  _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        );

622  _safeTransferFrom(
            _poolToken,
            msg.sender,
            address(this),
            _amount
        );

1042  _safeTransfer(
            _poolToken,
            msg.sender,
            _amount
        );

1073  _safeTransfer(
            _poolToken,
            msg.sender,
            _amount
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L476-L481

```solidity
file: blob/main/contracts/WrapperHub/AaveHub.sol

132  _safeTransferFrom(
            _underlyingAsset,
            msg.sender,
            address(this),
            _amount
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L132-L137

```solidity
file: blob/main/contracts/FeeManager/FeeManager.sol

304   _safeTransfer(
            _feeToken,
            msg.sender,
            amount
        );

710  _safeTransfer(
            _feeToken,
            caller,
            _amount
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L304-L308

## [G-11] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

Please note these instances were not included in the bot reports.

### cache the totalLpAssetsToDistribute state variable in local variable in PendlePowerFarmToken.previewDistribution

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

386     function previewDistribution()
        public
        view
        returns (uint256)
    {
        if (totalLpAssetsToDistribute == 0) {
            return 0;
        }

        if (block.timestamp == lastInteraction) {
            return 0;
        }

        if (totalLpAssetsToDistribute < ONE_WEEK) {
            return totalLpAssetsToDistribute;
        }

        uint256 currentRate = totalLpAssetsToDistribute
            / ONE_WEEK;

        uint256 additonalAssets = currentRate
            * (block.timestamp - lastInteraction);

        if (additonalAssets > totalLpAssetsToDistribute) {
            return totalLpAssetsToDistribute;
        }

        return additonalAssets;
    }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L386-L414

### cache the feeManager state variable in local variable in PositionNFTs.forwardFeeManagerNFT


```solidity
file: blob/main/contracts/PositionNFTs.sol

71   function forwardFeeManagerNFT(
        address _feeManagerContract
    )
        external
        onlyMaster
    {
        if (feeManager > ZERO_ADDRESS) {
            revert NotPermitted();
        }

        feeManager = _feeManagerContract;

        _transfer(
            address(this),
            _feeManagerContract,
            FEE_MANAGER_NFT
        );
    }
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L71-L88

## [G-12] Use constants instead of type(uintX).max 

When you use type(uintX).max, it may result in higher gas costs because it involves a runtime operation to calculate the type(uintX).max at runtime. This calculation is performed every time the expression is evaluated.

To save gas, it is recommended to use constants to represent the maximum value. Declaration of a constant with the maximum value means that the value is known at compile-time and does not require any runtime calculations.

```solidity
file: blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol

209   uint256 internal constant UINT256_MAX = type(uint256).max;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityDeclarations.sol#L209

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol

98  uint256 internal constant MAX_AMOUNT = type(uint256).max;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L98

```solidity
file: blob/main/contracts/WiseOracleHub/Declarations.sol

39   uint256 internal constant MAX_AMOUNT = type(uint256).max;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/Declarations.sol#L39

## [G-13] Calculation(addition, subtruction) which won’t underflow can be unchecked in

```solidity
file: blob/main/contracts/MainHelper.sol

321  totalPool + bareToken

1106  uint256 sum = delta
            + borrowData.pole;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L321


```solidity
file: blob/main/contracts/PositionNFTs.sol

125  totalReserved + 1;

293  ownerTokenCount + reservedCount

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L125

```solidity
file: blob/main/contracts/WiseSecurity/WiseSecurity.sol

812  withdrawAmount = withdrawAmount - _solelyWithdrawAmount;

848  tokenAmount = tokenAmount - _poolWithdrawAmount;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L812

```solidity
file: blob/main/contracts/WiseCore.sol

585  return receiveAmount + potentialPureExtraCashout;

501  uint256 shareDifference = cashoutShares
            - totalPoolInShares;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L585 

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

385   storageMarket.observationCardinalityNext + 1

439   return underlyingLpAssetsCurrent
            + totalLpAssetsToDistribute;

448   return previewDistribution()
            + underlyingLpAssetsCurrent;

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L385

```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

135  ownerTokenCount + reservedCount

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L135 


## [G-14] Using Storage Instead of Memory for structs/arrays Saves Gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct / array to be read from storage. This incurs a Gcoldsload (2100 gas) for each field of the struct / array. If the fields are read from the new memory variable, they incur an additional MLOAD, which is more expensive than a simple stack read.

Instead of declaring the variable with the memory keyword, a more cost-effective approach is to declare the variable with the storage keyword. In this case, you can cache any fields that need to be re-read in stack variables. This approach significantly reduces gas costs, as you only incur the Gcoldsload for the fields actually read.

The only scenario where it makes sense to read the whole struct / array into a memory variable is if the full struct / array is being returned by the function or passed to a function that requires memory. Additionally, if the array / struct is being read from another memory array / struct, using a memory variable may be appropriate.

By carefully considering the storage and memory usage in your contract, you can optimize gas costs and improve the efficiency of your smart contract operations

```solidity
file: blob/main/contracts/MainHelper.sol


399  address[] memory tokens = _userTokenData[

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L399

```solidity
file: blob/main/contracts/PositionNFTs.sol

292  uint256[] memory tokenIds = new uint256[](

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L292

```solidity
file: blob/main/contracts/WiseLending.sol

1373  address[] memory tokens = new address[](1);

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1373

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

27    IERC20[] memory tokens = new IERC20[](1);

28    uint256[] memory amount = new uint256[](1);

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L27

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

300  address[] memory rewardTokens = _getRewardTokens(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L300

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

199  uint256[] memory rewardsOutsideArray = _calculateRewardsClaimedOutside();

222  address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(

226  uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(

231  uint256[] memory rewardsOutsideArray = new uint256[](l);

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L199

```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

134  uint256[] memory tokenIds = new uint256[](

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L134

```solidity
file: blob/main/contracts/WiseOracleHub/OracleHelper.sol

413  uint32[] memory secondsAgo = new uint32[](

421  int56[] memory tickCumulatives

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L413

## [G-15] Use named returns for local variables of pure functions where it is possible

Proof of Concept

<details>

```solidity

library NoNamedReturnArithmetic {
    
    function sum(uint256 num1, uint256 num2) internal pure returns(uint256){
        return num1 + num2;
    }
}
contract NoNamedReturn {
    using NoNamedReturnArithmetic for uint256;
    uint256 public stateVar;
    function add2State(uint256 num) public {
        stateVar = stateVar.sum(num);
    }
}
```
```solidity
test for test/NoNamedReturn.t.sol:NamedReturnTest
[PASS] test_Increment() (gas: 27639)
```

```solidity
library NamedReturnArithmetic {
    
    function sum(uint256 num1, uint256 num2) internal pure returns(uint256 theSum){
        theSum = num1 + num2;
    }
}
contract NamedReturn {
    using NamedReturnArithmetic for uint256;
    uint256 public stateVar;
    function add2State(uint256 num) public {
        stateVar = stateVar.sum(num);
    }
}
```
```solidity

test for test/NamedReturn.t.sol:NamedReturnTest
[PASS] test_Increment() (gas: 27613)
```

</details>

6 Instances

### Use named returns in the PositionNFTs.getNextExpectedId() function

```solidity
file: blob/main/contracts/PositionNFTs.sol

130   function getNextExpectedId()
        public
        view
        returns (uint256)
    {
        return totalReserved + totalSupply();
    }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#130-L136

### Use named returns in the MainHelper.paybackAmount() function

```solidity
file: blob/main/contracts/MainHelper.sol

114   function paybackAmount(
        address _poolToken,
        uint256 _shares
    )
        public
        view
        returns (uint256)
    {
        uint256 product = _shares
            * borrowPoolData[_poolToken].pseudoTotalBorrowAmount;

        uint256 totalBorrowShares = borrowPoolData[_poolToken].totalBorrowShares;

        return product / totalBorrowShares + 1;
    }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L114-L128

### Use named returns in the PoolManager._getDeltaPole() function

```solidity
file: blob/main/contracts/PoolManager.sol

293  function _getDeltaPole(
        uint256 _maxPole,
        uint256 _minPole
    )
        private
        pure
        returns (uint256)
    {
        return (_maxPole - _minPole) / NORMALISATION_FACTOR;
    }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L293-L302

### Use named returns in the PendlePowerFarmToken.previewUnderlyingLpAssets() function

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

443  function previewUnderlyingLpAssets()
        public
        view
        returns (uint256)
    {
        return previewDistribution()
            + underlyingLpAssetsCurrent;
    }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L443-L450

### Use named returns in the MinterReserver._getNextReserveKey() function


```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol

70  function _getNextReserveKey()
        internal
        returns (uint256)
    {
        return totalMinted + _incrementReserved();
    }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L70-L75

### Use named returns in the PoolManager._getStartValue() function

```solidity
file: blob/main/contracts/PoolManager.sol

304   function _getStartValue(
        uint256 _maxPole,
        uint256 _minPole
    )
        private
        pure
        returns (uint256)
    {
        return (_maxPole + _minPole) / 2;
    }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L304-L313


## [G-16] Using assembly to revert with an error message

Issue Description - When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Estimated Gas Savings - Here’s an example:

<Details>

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}
/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```
</Details>

From the example above, we can see that we get a gas saving of over 300 gas when reverting with the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

```solidity
file: blob/main/contracts/PositionNFTs.sol

348  require(
            _exists(_tokenId) == true,
            "PositionNFTs: WRONG_TOKEN"
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L348-L351

```solidity
file: blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol

15  require(
            msg.sender == master,
            "CustomOracleSetup: NOT_MASTER"
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L15-L18

```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

31  require(
            msg.sender == farmContract,
            "PowerFarmsNFTs: INVALID_FARM"
        );
190  require(
            _exists(_tokenId) == true,
            "PowerFarmsNFTs: WRONG_TOKEN"
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L31-L34


## [G-17] Refactor functions to combine events to reduce gas cost

In scenarios where a function emits multiple event but these events are only emitted by that function we can combine these events to a single event in doing this we would save at least 1 Glog0 375 gas units


### Combine the events in WiseCore._coreWithdrawToken() to reduce gas cost


The WiseCore._coreWithdrawToken() function emits two events FundsWithdrawnOnBehalf and FundsWithdrawn but these events are only emitted in this function. We can make the WiseCore._coreWithdrawToken() function more gas efficient if we combine these FundsWithdrawnOnBehalf and FundsWithdrawn events to a single event.

```solidity
file: blob/main/contracts/WiseCore.sol

76   if (_onBehalf == true) {
            emit FundsWithdrawnOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
        } else {
            emit FundsWithdrawn(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
        }
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L76-L94

### Combine the events in WiseCore._coreBorrowTokens() to reduce gas cost

The WiseCore._coreBorrowTokens() function emits two events FundsBorrowedOnBehalf and FundsBorrowed but these events are only emitted in this function. We can make the WiseCore._coreBorrowTokens() function more gas efficient if we combine these FundsBorrowedOnBehalf and FundsBorrowed events to a single event.

```solidity
file: blob/main/contracts/WiseCore.sol

397  if (_onBehalf == true) {
            emit FundsBorrowedOnBehalf(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
        } else {
            emit FundsBorrowed(
                _caller,
                _nftId,
                _poolToken,
                _amount,
                _shares,
                block.timestamp
            );
        }

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L397-L415


## [G-18] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

Issue Description - The abi.encodePacked() function is used to concatenate multiple arguments into a byte array prior to 0.8.4. However, since 0.8.4 the bytes.concat() function was introduced which performs the same role but is preferred since it avoids ABI encoding overhead.

Proposed Optimization - Replace uses of abi.encodePacked() with bytes.concat() where multiple arguments need to be concatenated into a byte array.

Estimated Gas Savings - Using bytes.concat() instead of abi.encodePacked() saves approximately 300-500 gas per concatenation by avoiding ABI encoding.

```solidity
file: blob/main/contracts/MainHelper.sol

656   abi.encodePacked(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L656


```solidity
file:  blob/main/contracts/PositionNFTs.sol

360    abi.encodePacked(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L360

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

24   abi.encodePacked(

66   abi.encodePacked(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L24

```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

202  abi.encodePacked(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L202 

```solidity
file: blob/main/contracts/WiseOracleHub/WiseOracleHub.sol

48   abi.encodePacked(

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L48


## [G‑19] Using > 0 costs more gas than != 0 when used on a uint in a if() statement


```solidity
file: blob/main/contracts/PoolManager.sol 

108   if (_maximumDeposit > 0) {

112   if (_collateralFactor > 0) {

268   if (fetchBalance > 0) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol#L108

```solidity
file: blob/main/contracts/PositionNFTs.sol

117  if (reserved[_user] > 0) {

203  if (nftId > 0) {

288  if (reservedId > 0) {

309  if (reservedId > 0) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L117


```solidity
file: blob/main/contracts/WiseCore.sol

564   if (pureCollateral > 0 && userShares > 0) {

572   if (potentialPureExtraCashout > 0 && potentialPureExtraCashout <= pureCollateral) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L564

```solidity
file: blob/main/contracts/WiseLending.sol

1147   if (refundAmount > 0) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1147

```solidity
file: blob/main/contracts/WiseLowLevelHelper.sol

32   if (_parameterValue > _parameterLimit) {

464  if (_value > 0) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L32

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

107  if (initialAmount > 0) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L107


```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol

103   if (positionBorrowShares > 0) {

186   if (borrowShares > 0) {

198   if (borrowSharesAave > 0) {

206   if (borrowShares > 0) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L103

```solidity
file: blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

388   if (_amount > 0) {

417   if (_amount > 0) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L388


```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

130   if (reservedId > 0) {

151   if (reservedId > 0) {

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L130

## [G‑20] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided

```solidity
file: blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol

26     function renounceOwnership()
        external
        onlyOwner
    {

33  function setLastUpdateGlobal(
        uint256 _time
    )
        external
        onlyOwner
    {

42  function setRoundData(
        uint80 _roundId,
        uint256 _updateTime
    )
        external
        onlyOwner
    {

52   function setGlobalAggregatorRoundId(
        uint80 _aggregatorRoundId
    )
        external
        onlyOwner
    {
```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L26-L29



## [G‑21] require()/revert() strings longer than 32 bytes cost extra gas


```solidity
file: blob/main/contracts/PositionNFTs.sol

348  require(
            _exists(_tokenId) == true,
            "PositionNFTs: WRONG_TOKEN"
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L348-L351

```solidity
file: blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol

15  require(
            msg.sender == master,
            "CustomOracleSetup: NOT_MASTER"
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol#L15-L18

```solidity
file: blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

31  require(
            msg.sender == farmContract,
            "PowerFarmsNFTs: INVALID_FARM"
        );
190  require(
            _exists(_tokenId) == true,
            "PowerFarmsNFTs: WRONG_TOKEN"
        );

```
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L31-L34
