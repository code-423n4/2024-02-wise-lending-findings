# QA Report: WiseLending

## L-01 Malicious wise security admin can hide escalating risks

The setBadDebtUserLiquidation function is restricted to be called by only WISE_SECURITY. This however, does not safeguard it against accidental or misuse by bad acting admins. The lack of safeguards for invalid bad debt updates opens this function to attak vectors that can be abused by insiders who are aware of this loophole. For instance an admin acting in bad faith could abuse this to hide escalating risks in the system by making zero-value adjustments. This could also be abused to flood the event logs with misleading event that delay timely analysis of bad debt and associated risks.


```solidity
function setBadDebtUserLiquidation(
        uint256 _nftId,
        uint256 _amount
    )
        external
        onlyWiseSecurity
    {
        _setBadDebtPosition(
            _nftId,
            _amount
        );

        emit SetBadDebtPosition(
            _nftId,
            _amount,
            block.timestamp
        );
    }
```

```solidity
File: FeeManagerHelper.sol
function _setBadDebtPosition(
        uint256 _nftId,
        uint256 _amount
    )
        internal
    {
        badDebtPosition[_nftId] = _amount;
    }
```

### scenario

- Wiselending has experienced a surge in defaulted loans, leading to a significant increase in bad debt positions.
- Alice holds the WISE_SECURITY admin privileges
- Alice, being aware of this issue, decides to hide the escalating risks by manipulating the bad debt positions.
- Alice calls the setBadDebtUserLiquidation function repeatedly with different NFT IDs and a zero _amount value
- This creates a flood of events in the logs, obfuscating the true extent of the bad debt crisis.
- Alice effectively masks the growing bad debt problem
- This also increases the noise-to-signal ratio, making it challenging for analysts to identify and investigate legitimate bad debt events.


### Mitigation

Add safeguards for _amount to ensure it is non zero.

### Poc 

```python
class Contract:
    def __init__(self):
        self.bad_debt_position = {}

    def set_bad_debt_user_liquidation(self, nft_id, amount):
        self._set_bad_debt_position(nft_id, amount)
        print(f"SetBadDebtPosition event: nft_id={nft_id}, amount={amount}")

    def _set_bad_debt_position(self, nft_id, amount):
        self.bad_debt_position[nft_id] = amount

# Instantiate the contract
contract = Contract()

# Scenario 1: Hiding escalating risks by making zero-value adjustments
contract.set_bad_debt_user_liquidation(123, 0)
contract.set_bad_debt_user_liquidation(456, 0)
contract.set_bad_debt_user_liquidation(789, 0)

# Scenario 2: Flooding the event logs with misleading events
for i in range(1000):
    contract.set_bad_debt_user_liquidation(i, 0)
```

### Loc: 

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L514


## L-02: Inaccurate share price calculations leading to unfair distribution of shares or incorrect valuations.

The in WiseLending contract has insufficient time validation. It's not checking for the edge case when time difference may be very small during pool initialization or timestamp updates. This can lead to calculations that do not accurately reflect the intended restrictions on share price increases.

```solidity
File: WiseLending.sol

function _getCurrentSharePriceMax(
        address _poolToken
    )
        private
        view
        returns (uint256)
    {
        uint256 timeDifference = block.timestamp
            - timestampsPoolData[_poolToken].initialTimeStamp;

        return timeDifference
            * RESTRICTION_FACTOR
            + PRECISION_FACTOR_E18;
    }
```

### Mitigation

```solidity
require(block.timestamp >= timestampsPoolData[_poolToken].initialTimeStamp, "Current time is before the initial timestamp.");
```

### Loc: 

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L255C5-L268C6


## L-03 Adopt a fail fast approach in removePoolTokenManual function

It is solidity security best practices to adopt a fail fast approach for removePoolTokenManual function and other functions as well. This ensures the function execution is halted early if the precondition is not met therefore avoiding any further execution or state changes that are doomed to revert eventually.

```solidity
File: FeeManager.sol

uint256 i;
        uint256 len = getPoolTokenAddressesLength();
        uint256 lastEntry = len - 1;
        bool found;

        if (poolTokenAdded[_poolToken] == false) {
            revert PoolNotPresent();
        }
```

### LOC: 
https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L444C9-L451C10

## L-04 Insufficient Batch Operation Failure Handling 

The setAaveFlagBulk function does not provide granular feedback on which specific operation or operations might fail within the batch. If _setAaveFlag reverts for any pair, the whole transaction reverts. This applies to other functions perfoming bulk operations as well.

```solidity
File: FeeManager.sol

function setAaveFlagBulk(
        address[] calldata _poolTokens,
        address[] calldata _underlyingTokens
    )
        externals
        onlyMaster
    {
        uint256 i;
        uint256 l = _poolTokens.length;

        while (i < l) {
            _setAaveFlag(
                _poolTokens[i],
                _underlyingTokens[i]
            );

            unchecked {
                ++i;
            }
        }
    }
```

### Mitigation
Introduce granular error handling for batch operations in FeeManager to provide better feedback.

### LOC:
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L135
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L135
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L249
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L603
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L898

## L-05 Batch size not explicitly limited could potentially lead to DOS and bad user experience

Not limiting batch size explicitly can lead to failed transactions due to the transaction exceeding the block gas limit. 

```solidity
File: FeeManager.sol

function setAaveFlagBulk(
        address[] calldata _poolTokens,
        address[] calldata _underlyingTokens
    )
        externals
        onlyMaster
    {
        uint256 i;
        uint256 l = _poolTokens.length;

        while (i < l) {
            _setAaveFlag(
                _poolTokens[i],
                _underlyingTokens[i]
            );

            unchecked {
                ++i;
            }
        }
    }
```


### Mitigation

```solidity
require(_poolTokens.length <= MAX_BATCH_SIZE, "Batch size exceeds limit");
```

 
### LOC:
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L135
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L135
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L249
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L603
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L898


L-06 No circuit breakers for critical functions

For functions that directly impact liquidity in WiseLending add circuit breakers to better safeguard against high market volatility scenarios where platform is not able to facilitate user withdrawals or borrowing.

For instance, Excessive borrowing, especially in volatile market conditions, can significantly impact the platform's overall health and risk profile. A circuit breaker could be triggered by abnormal spikes in borrowing activity, drastic price movements of collateral assets, or when the system's overall health metrics like the total collateralization ratio fall below safe thresholds. 

### Mitigation

Apply circuit breakers to the following functions:

File: WiseLending.sol

- withdrawExactAmount
- paybackExactAmount
- withdrawExactAmountETH
- solelyWithdraw
- solelyWithdrawETH
- borrowExactAmount
- borrowExactAmountETH
- paybackExactShares
- paybackExactAmount
- paybackExactAmountETH
- paybackExactShares
- liquidatePartiallyFromTokens





## L-07 User can game system to affect the position of a borrower without actually providing any value to the protocol

The liquidatePartiallyFromTokens function in WiseLending can execute without any actual repayment of debt. This allows for malicious users to target specific borrowers and affecting the health status of a borrower's position without making a significant contribution to debt reduction.

### Poc
- An attacker observes that the liquidatePartiallyFromTokens function lacks a check for _shareAmountToPay > 0.

- The attacker identifies a target borrower's position that they wish to manipulate for potential gain or harm.

- The attacker calls liquidatePartiallyFromTokens, specifying a _shareAmountToPay of 0. This is done with the intention to affect the borrower's position.

- The function executes without any actual repayment of debt due to the zero _shareAmountToPay.
        
- Despite no value being added or debt reduced, the attacker could potentially trigger unwarranted liquidation processes or affecting the health status of the borrower's position.

- The attacker manages to game the system, impacting a borrower's position negatively without contributing to the protocol's health or taking on the intended risk associated with liquidation.

### Mitigation

Add this at the beginning of the liquidatePartiallyFromTokens function

```solidity
require(_shareAmountToPay > 0, "Liquidation amount must be greater than zero.");
```

### LOC

- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L1250

## NC-01 Opt for `For` loops instead of while loops for readability

> for is a keyword in Solidity and it informs the compiler that it contains information about looping a set of instructions. It is very similar to the while loop; however it is more succinct and readable since all information can be viewed in a single line.

source: https://subscription.packtpub.com/book/programming/9781788831383/5/ch05lvl1sec61/the-for-loop

### LOC:
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L298
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L385
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L397
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L92
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L145
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L257
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L453
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L548
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L581
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L609
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L904
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L32
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L79
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L121
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L160
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L372
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L408
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L444
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L501
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L1065
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManagerHelper.sol#L25
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L204
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L258
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L428
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L227

## NC-02 Function name should describe functionality accurately to avoid confusions

Change function name from setBlacklistToken to toggleBlacklistToken to better describe the functionality. Same goes for setSecurityWorker though it applies more to the former.

```solidity
function setBlacklistToken(
        address _tokenAddress,
        bool _state
    )
        external
        onlyMaster()
    {
        wasBlacklisted[_tokenAddress] = _state;
    }

```

```solidity
function setSecurityWorker(
        address _entitiy,
        bool _state
    )
        external
        onlyMaster
    {
        securityWorker[_entitiy] = _state;
    }
```

LOC:
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L980
- https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L995

## NC-03 Pack the arguments into a single code line to improve readability

To improve readability even further pack function arguments into a single line of code.

```solidity
File: FeeManagerHelper.sol

function updatePositionCurrentBadDebt(
        uint256 _nftId
    )
        public
    {
        _prepareCollaterals(
            _nftId
        );

        _prepareBorrows(
            _nftId
        );

        _updateUserBadDebt(
            _nftId
        );
    }
```

LOC:
> Almost all files in scope. 

## NC-04 Add information to ClaimedIncentivesBulk event

Adding more information like number of incentives claimed and addresses for which incentives were claimed can benefit offchain monitoring.

LOC
- https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L276