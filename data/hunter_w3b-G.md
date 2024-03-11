## [G-01] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
File:  contracts/DerivativeOracles/CustomOracleSetup.sol
22  function renounceOwnership() external onlyOwner {
23        master = address(0x0);
24    }
25
26    function setLastUpdateGlobal(uint256 _time) external onlyOwner {
27        lastUpdateGlobal = _time;
28    }
29
30    function setRoundData(uint80 _roundId, uint256 _updateTime) external onlyOwner {
31        timeStampByRoundId[_roundId] = _updateTime;
32    }
33
34    function setGlobalAggregatorRoundId(uint80 _aggregatorRoundId) external onlyOwner {
35        globalRoundId = _aggregatorRoundId;
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
24        external
25        onlyChildContract(_pendleMarket)
26    {


110    function addPendleMarket(
111        address _pendleMarket,
112        string memory _tokenName,
113        string memory _symbolName,
114        uint16 _maxCardinality
115    ) external onlyMaster {

158  function updateRewardTokens(address _pendleMarket) external onlyChildContract(_pendleMarket) returns (bool) {

172  function changeExchangeIncentive(uint256 _newExchangeIncentive) external onlyMaster {

178  function changeMintFee(address _pendleMarket, uint256 _newFee) external onlyMaster {

193    function lockPendle(uint256 _amount, uint128 _weeks, bool _fromInside, bool _sameExpiry)
194        external
195        onlyMaster
196        returns (uint256 newVeBalance)
197    {

231  function claimArb(uint256 _accrued, bytes32[] calldata _proof) external onlyArbitrum {

237  function withdrawLock() external onlyMaster returns (uint256 amount) {

264    function increaseReservedForCompound(address _pendleMarket, uint256[] calldata _amounts)
265        external
266        onlyChildContract(_pendleMarket)
267    {

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

## [G-03] Use calldata instead of memory for function arguments that do not get mutated

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

## [G-04] Using storage instead of memory for structs/arrays saves gas

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

## [G-05] With assembly, .call (bool success)  transfer can be done gas-optimized

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

## [G-06] Use constants instead of type(uintx).max

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

## [G-07] Use nested if and, avoid multiple check combinations

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

## [G-08] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.

Note: this instances missed from bots

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

## [G-09] Change public state variable visibility to private

it's generally a good practice to limit the visibility of state variables to the minimum necessary level. This means that you should use the private visibility modifier for state variables whenever possible, and avoid using the public modifier unless it is absolutely necessary.

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

## [G-10] Use of emit inside a loop

Emitting an event inside a loop performs a LOG op N times, where N is the loop length. Consider refactoring the code to emit the event only once at the end of loop. Gas savings should be multiplied by the average loop length.

```solidity
File:  contracts/FeeManager/FeeManager.sol
104  emit PoolFeeChanged(_poolTokens[i], _newFees[i], block.timestamp);
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L104

## [G-11] multiplications of powers of 2 can be replaced by a left shift operation to save gas

Replace such found multiplications with left shift operations when ensured it is safe to do so. NOTE: This only applies to uint variables!

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

## [G‑12] State variables only set in the constructor should be declared immutable

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

## [G-13] Shorten the array rather than copying to a new one

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

## [G-14] Use selfbalance() instead of address(this).balance

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

## [G-15] Replace state variable reads and writes within loops with local variable reads and writes.

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

## [G-16] Consider merging sequential loops

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

## [G-17] Consider using OZ EnumerateSet in place of nested mappings

Nested mappings and multi-dimensional arrays in Solidity operate through a process of double hashing, wherein the original storage slot and the first key are concatenated and hashed, and then this hash is again concatenated with the second key and hashed. This process can be quite gas expensive due to the double-hashing operation and subsequent storage operation (sstore).
A possible optimization involves manually concatenating the keys followed by a single hash operation and an sstore. However, this technique introduces the risk of storage collision, especially when there are other nested hash maps in the contract that use the same key types. Because Solidity is unaware of the number and structure of nested hash maps in a contract, it follows a conservative approach in computing the storage slot to avoid possible collisions.
OpenZeppelin's EnumerableSet provides a potential solution to this problem. It creates a data structure that combines the benefits of set operations with the ability to enumerate stored elements, which is not natively available in Solidity. EnumerableSet handles the element uniqueness internally and can therefore provide a more gas-efficient and collision-resistant alternative to nested mappings or multi-dimensional arrays in certain scenarios.

```solidity
File:  contracts/FeeManager/DeclarationsFeeManager.sol
143  mapping(address => mapping(address => bool)) public allowedTokens;

146  mapping(address => mapping(address => uint256)) public gatheredIncentiveToken;
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L143
