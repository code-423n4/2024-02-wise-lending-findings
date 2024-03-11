## [G-01] Functions guaranteed to revert when called by normal users can be marked *payable*

If a function modifier such as *onlyOwner* is used, the function will revert if a normal user tries to pay the function. Marking the function as *payable* will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are *CALLVALUE*(2),*DUP1*(3),*ISZERO*(3),*PUSH2*(3),*JUMPI*(10),*PUSH1*(3),*DUP1*(3),*REVERT*(0),*JUMPDEST*(1),*POP*(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

There are 72 instances of this issue in 16 files:

    File: WiseLending.sol	

    859: function withdrawOnBehalfExactAmount(
    860:     uint256 _nftId,
    861:     address _poolToken,
    862:     uint256 _withdrawAmount
    863: )
    864:     external
    865:     onlyAaveHub
    866:     syncPool(_poolToken)
    867:     healthStateCheck(_nftId)
    868:     returns (uint256)

    939: function withdrawOnBehalfExactShares(
    940:     uint256 _nftId,
    941:     address _poolToken,
    942:     uint256 _shares
    943: )
    944:     external
    945:     onlyAaveHub
    946:     syncPool(_poolToken)
    947:     healthStateCheck(_nftId)
    948:     returns (uint256)

    1055: function borrowOnBehalfExactAmount(
    1056:     uint256 _nftId,
    1057:     address _poolToken,
    1058:     uint256 _amount
    1059: )
    1060:     external
    1061:     onlyAaveHub
    1062:     syncPool(_poolToken)
    1063:     healthStateCheck(_nftId)
    1064:     returns (uint256)

    1412: function corePaybackFeeManager(
    1413:     address _poolToken,
    1414:     uint256 _nftId,
    1415:     uint256 _amount,
    1416:     uint256 _shares
    1417: )
    1418:     external
    1419:     onlyFeeManager
    1420:     syncPool(_poolToken)

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol

    File: WiseSecurity.sol	

    85: function setLiquidationSettings(
    86:     uint256 _baseReward,
    87:     uint256 _baseRewardFarm,
    88:     uint256 _newMaxFeeETH,
    89:     uint256 _newMaxFeeFarmETH
    90: )
    91:     external
    92:     onlyMaster

    142: function prepareCurvePools(
    143:     address _poolToken,
    144:     CurveSwapStructData calldata _curveSwapStructData,
    145:     CurveSwapStructToken calldata _curveSwapStructToken
    146: )
    147:     external
    148:     onlyWiseLending

    193: function curveSecurityCheck(
    194:     address _poolToken
    195: )
    196:     external
    197:     onlyWiseLending

    405: function checkBadDebtLiquidation(
    406:     uint256 _nftId
    407: )
    408:     external
    409:     onlyWiseLending

    980: function setBlacklistToken(
    981:     address _tokenAddress,
    982:     bool _state
    983: )
    984:     external
    985:     onlyMaster()

    995: function setSecurityWorker(
    996:     address _entitiy,
    997:     bool _state
    998: )
    999:     external
    1000:     onlyMaster

    1032: function revokeShutdown()
    1033:     external
    1034:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol

    File: PendlePowerFarmToken.sol	

    592: function changeMintFee(
    593:     uint256 _newFee
    594: )
    595:     external
    596:     onlyController

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

    File: FeeManager.sol	

    49: function setRepayBadDebtIncentive(
    50:     uint256 _percent
    51: )
    52:     external
    53:     onlyMaster

    66: function setAaveFlag(
    67:     address _poolToken,
    68:     address _underlyingToken
    69: )
    70:     external
    71:     onlyMaster

    82: function setAaveFlagBulk(
    83:     address[] calldata _poolTokens,
    84:     address[] calldata _underlyingTokens
    85: )
    86:     external
    87:     onlyMaster

    108: function setPoolFee(
    109:     address _poolToken,
    110:     uint256 _newFee
    111: )
    112:     external
    113:     onlyMaster

    135: function setPoolFeeBulk(
    136:     address[] calldata _poolTokens,
    137:     uint256[] calldata _newFees
    138: )
    139:     external
    140:     onlyMaster

    173: function proposeIncentiveMaster(
    174:     address _proposedIncentiveMaster
    175: )
    176:     external
    177:     onlyIncentiveMaster

    214: function increaseIncentiveA(
    215:     uint256 _value
    216: )
    217:     external
    218:     onlyIncentiveMaster

    232: function increaseIncentiveB(
    233:     uint256 _value
    234: )
    235:     external
    236:     onlyIncentiveMaster

    390: function addPoolTokenAddress(
    391:     address _poolToken
    392: )
    393:     external
    394:     onlyWiseLending

    412: function addPoolTokenAddressManual(
    413:     address _poolToken
    414: )
    415:     external
    416:     onlyMaster

    438: function removePoolTokenManual(
    439:     address _poolToken
    440: )
    441:     external
    442:     onlyMaster

    494: function increaseTotalBadDebtLiquidation(
    495:     uint256 _amount
    496: )
    497:     external
    498:     onlyWiseSecurity

    514: function setBadDebtUserLiquidation(
    515:     uint256 _nftId,
    516:     uint256 _amount
    517: )
    518:     external
    519:     onlyWiseSecurity

    538: function setBeneficial(
    539:     address _user,
    540:     address[] calldata _feeTokens
    541: )
    542:     external
    543:     onlyMaster

    571: function revokeBeneficial(
    572:     address _user,
    573:     address[] memory _feeTokens
    574: )
    575:     external
    576:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol

    File: PendlePowerFarmController.sol	

    211: function addPendleMarket(
    212:     address _pendleMarket,
    213:     string memory _tokenName,
    214:     string memory _symbolName,
    215:     uint16 _maxCardinality
    216: )
    217:     external
    218:     onlyMaster

    321: function changeExchangeIncentive(
    322:     uint256 _newExchangeIncentive
    323: )
    324:     external
    325:     onlyMaster

    334: function changeMintFee(
    335:     address _pendleMarket,
    336:     uint256 _newFee
    337: )
    338:     external
    339:     onlyMaster

    364: function lockPendle(
    365:     uint256 _amount,
    366:     uint128 _weeks,
    367:     bool _fromInside,
    368:     bool _sameExpiry
    369: )
    340:     external
    341:     onlyMaster
    342:     returns (uint256 newVeBalance)

    431: function claimArb(
    432:     uint256 _accrued,
    433:     bytes32[] calldata _proof
    434: )
    435:     external
    436:     onlyArbitrum

    451: function withdrawLock()
    452:     external
    453:     onlyMaster
    454:     returns (uint256 amount)

    601: function forwardETH(
    602:     address _to,
    603:     uint256 _amount
    604: )
    605:     external
    606:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

    File: WiseLendingDeclaration.sol	

    140: function setSecurity(
    141:     address _wiseSecurity
    142: )
    143:     external
    144:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol

    File: WiseOracleHub.sol	

    218: function addTwapOracle(
    219:     address _tokenAddress,
    220:     address _uniPoolAddress,
    221:     address _token0,
    222:     address _token1,
    223:     uint24 _fee
    224: )
    225:     external
    226:     onlyMaster

    266: function addTwapOracleDerivative(
    267:     address _tokenAddress,
    268:     address _partnerTokenAddress,
    269:     address[2] calldata _uniPools,
    270:     address[2] calldata _token0Array,
    271:     address[2] calldata _token1Array,
    272:     uint24[2] calldata _feeArray
    273: )
    274:     external
    275:     onlyMaster

    359: function addOracle(
    360:     address _tokenAddress,
    361:     IPriceFeed _priceFeedAddress,
    362:     address[] calldata _underlyingFeedTokens
    363: )
    364:     external
    365:     onlyMaster

    379: function addOracleBulk(
    380:     address[] calldata _tokenAddresses,
    381:     IPriceFeed[] calldata _priceFeedAddresses,
    382:     address[][] calldata _underlyingFeedTokens
    383: )
    384:     external
    385:     onlyMaster

    403: function addAggregator(
    404:     address _tokenAddress
    405: )
    406:     external
    407:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol

    File: AaveHub.sol	

    44: function setAaveTokenAddress(
    45:     address _underlyingAsset,
    46:     address _aaveToken
    47: )
    48:     external
    49:     onlyMaster

    64: function setAaveTokenAddressBulk(
    65:     address[] calldata _underlyingAssets,
    66:     address[] calldata _aaveTokens
    67: )
    68:     external
    69:     onlyMaster

    611: function skimAave(
    612:     address _underlyingAsset,
    613:     bool _isAave
    614: )
    615:     external
    616:     validToken(_underlyingAsset)
    617:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol

    File: WiseLowLevelHelper.sol	

    424: function setPoolFee(
    425:     address _poolToken,
    426:     uint256 _newFee
    427: )
    428:     external
    429:     onlyFeeManager

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol

    File: PositionNFTs.sol	

    54: function assignReserveRole(
    55:     address _reserverForOthers,
    56:     bool _state
    57: )
    58:     external
    59:     onlyMaster

    64: function blockReservePublic()
    65:     external
    66:     onlyMaster
    67: {

    71: function forwardFeeManagerNFT(
    72:     address _feeManagerContract
    73: )
    74:     external
    75:     onlyMaster

    99: function reservePositionForUser(
    101:     address _user
    102: )
    103:     onlyReserveRole
    104:     external
    105:     returns (uint256)

    319: function setBaseURI(
    320:     string memory _newBaseURI
    321: )
    322:     external
    323:     onlyMaster

    328: function setBaseExtension(
    329:     string memory _newBaseExtension
    330: )
    331:     external
    332:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol

    File: PoolManager.sol	

    25: function setParamsLASA(
    26:     address _poolToken,
    27:     uint256 _poolMulFactor,
    28:     uint256 _upperBoundMaxRate,
    29:     uint256 _lowerBoundMaxRate,
    30:     bool _steppingDirection,
    31:     bool _isFinal
    32: )
    33:     external
    34:     onlyMaster

    100: function setPoolParameters(
    101:     address _poolToken,
    102:     uint256 _collateralFactor,
    103:     uint256 _maximumDeposit
    104: )
    105:     external
    106:     onlyMaster

    125: function setVerifiedIsolationPool(
    126:     address _isolationPool,
    127:     bool _state
    128: )
    129:     external
    130:     onlyMaster

    135: function createPool(
    136:     CreatePool calldata _params
    137: )
    138:     external
    139:     onlyMaster

    146: function createCurvePool(
    147:     CreatePool calldata _params,
    148:     CurvePoolSettings calldata _settings
    149: )
    150:     external
    151:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PoolManager.sol

    File: PendlePowerManager.sol	

    63: function changeMinDeposit(
    64:     uint256 _newMinDeposit
    65: )
    66:     external
    67:     onlyMaster

    82: function shutDownFarm(
    83:     bool _state
    84: )
    85:     external
    86:     onlyMaster

    210: function exitFarm(
    211:     uint256 _keyId,
    212:     uint256 _allowedSpread,
    213:     bool _ethBack
    214: )
    215:     external
    216:     updatePools
    217:     onlyKeyOwner(_keyId)

    274: function manuallyWithdrawShares(
    275:     uint256 _keyId,
    276:     uint256 _withdrawShares
    277: )
    278:     external
    279:     updatePools
    280:     onlyKeyOwner(_keyId)

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol

    File: PowerFarmNFTs.sol	

    51: function setFarmContract(
    52:     address _farmContract
    53: )
    54:     external
    55:     onlyMaster

    66: function mintKey(
    67:     address _keyOwner,
    68:     uint256 _keyId
    69: )
    70:     external
    71:     onlyFarmContract

    82: function burnKey(
    83:     uint256 _keyId
    84: )
    85:     external
    86:     onlyFarmContract

    161: function setBaseURI(
    162:     string calldata _newBaseURI
    163: )
    164:     external
    165:     onlyMaster

    170: function setBaseExtension(
    171:     string calldata _newBaseExtension
    172: )
    173:     external
    174:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol

    File: Declarations.sol	

    105: function setWiseSecurity(
    106:     address _securityAddress
    107: )
    108:     external
    109:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/Declarations.sol

    File: OwnableMaster.sol	

    70: function proposeOwner(
    71:     address _proposedOwner
    72: )
    73:     external
    74:     onlyMaster

    92: function claimOwnership()
    93:     external
    94:     onlyProposed

    103: function renounceOwnership()
    104:     external
    105:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/OwnableMaster.sol

    File: CustomOracleSetup.sol	

    26: function renounceOwnership()
    27:     external
    28:     onlyOwner

    33: function setLastUpdateGlobal(
    34:     uint256 _time
    35: )
    36:     external
    37:     onlyOwner

    42: function setRoundData(
    43:     uint80 _roundId,
    44:     uint256 _updateTime
    45: )
    46:     external
    47:     onlyOwner

    52: function setGlobalAggregatorRoundId(
    53:     uint80 _aggregatorRoundId
    54: )
    56:     external
    57:     onlyOwner

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/CustomOracleSetup.sol

## [G-02] Using *calldata* instead of *memory* for read-only arguments in *external* functions saves gas
When a function with a *memory* array is called externally, the *abi.decode()* step has to use a for-loop to copy each index of the *calldata* to the *memory* index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * *<mem_array>.length*). Using *calldata* directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having *memory* arguments, it’s still valid for implementation contracs to use *calldata* arguments instead.

If the array is passed to an *internal* function which passes the array to another internal function where the array is modified and therefore *memory* is used in the *external* call, it’s still more gass-efficient to use *calldata* when the *external* function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is *public* but can be marked as *external* since it’s not called by the contract, and cases where a constructor is involved

There are 17 instances of this issue in 11 files:

    File: MainHelper.sol	

    417: function _prepareTokens(
    418:     address _poolToken,
    419:     address[] memory _tokens
    420: )
    421:     private
    
    454: function _curveSecurityChecks(
    455:     address[] memory _lendTokens,
    456:     address[] memory _borrowTokens
    457: )
    458:     internal

    473: function _whileLoopCurveSecurity(
    474:     address[] memory _tokens
    475: )
    476:     private

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol

    File: PendlePowerFarmToken.sol	

    682: function initialize(
    683:     address _underlyingPendleMarket,
    684:     address _pendleController,
    685:     string memory _tokenName,
    686:     string memory _symbolName,
    687:     uint16 _maxCardinality
    688: )
    689:     external

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

    File: FeeManager.sol	

    571: function revokeBeneficial(
    572:     address _user,
    573:     address[] memory _feeTokens
    574: )
    575:     external
    576:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol

    File: PendlePowerFarmLeverageLogic.sol	

    59: function receiveFlashLoan(
    60:     IERC20[] memory _flashloanToken,
    61:     uint256[] memory _flashloanAmounts,
    62:     uint256[] memory _feeAmounts,
    63:     bytes memory _userData
    64: )
    65:     external

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

    File: OracleHelper.sol	

    460: function _getTwapDerivatePrice(
    461:     address _tokenAddress,
    462:     UniTwapPoolInfo memory _uniTwapPoolInfo
    463: )
    464:     internal
    465:     view
    466:     returns (uint256)

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol

    File: PendlePowerFarmController.sol	

    211: function addPendleMarket(
    212:     address _pendleMarket,
    213:     string memory _tokenName,
    214:     string memory _symbolName,
    215:     uint16 _maxCardinality
    216: )
    217:     external
    218:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

    File: WiseCore.sol	

    625: function _coreLiquidation(
    626:     CoreLiquidationStruct memory _data
    627: )
    628:     internal
    629:     returns (uint256 receiveAmount)

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol

    File: PositionNFTs.sol	

    319: function setBaseURI(
    320:     string memory _newBaseURI
    321: )
    322:     external
    323:     onlyMaster

    328: function setBaseExtension(
    329:     string memory _newBaseExtension
    330: )
    331:     external
    332:     onlyMaster

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol

    File: PendlePowerFarmControllerHelper.sol	

    9: function _findIndex(
    10:     address[] memory _array,
    11:     address _value
    12: )
    13:     internal
    14:     pure
    15:     returns (uint256)

    217: function _compareHashes(
    218:     address _pendleMarket,
    219:     address[] memory rewardTokensToCompare
    220: )
    221:     internal
    222:     view
    223:     returns (bool)

    236: function _setRewardTokens(
    237:     address _pendleMarket,
    238:     address[] memory _rewardTokens
    239: )
    240:     internal

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerHelper.sol

    File: PendlePowerFarmTokenFactory.sol	

    35: function deploy(
    36:     address _underlyingPendleMarket,
    37:     string memory _tokenName,
    38:     string memory _symbolName,
    39:     uint16 _maxCardinality
    40: )
    41:     external
    42:     returns (address)

    56: function _clone(
    57:     address _underlyingPendleMarket,
    58:     string memory _tokenName,
    59:     string memory _symbolName,
    60:     uint16 _maxCardinality
    61: )
    62:     private
    63:     returns (address pendlePowerFarmTokenAddress)

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

    File: CallOptionalReturn.sol	

    12: function _callOptionalReturn(
    13:     address token,
    14:     bytes memory data
    15: )
    16:     internal
    17:     returns (bool call)

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol
