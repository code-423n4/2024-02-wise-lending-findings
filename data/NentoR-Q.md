## Low Issues

### [L-1] `PendlePowerManager::receive()` doesn't forward ETH as specified in natspec

```solidity
/**
  * @dev Standard receive functions forwarding
  * directly send ETH to the master address.
  */
receive()
	external
	payable
{
	emit ETHReceived(
		msg.value,
		msg.sender
	);
}
```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarm/PendlePowerManager.sol#L21-L29

This leads to funds sent to this contract becoming inaccessible.

### [L-2] Small risk of collision with already used address in `PendlePowerFarmTokenFactory::_clone()`.

In `PendlePowerFarmTokenFactory::_clone()`, there's a small risk that the `create2` might create an address that already exists. Since the salt is not sender controlled, there'll be no way to deploy the particular Pendle market. Make the salt controllable by the deployer (in `PendlePowerFarmController::addPendleMarket()`).

```solidity
bytes32 salt = keccak256(
		abi.encodePacked(
				_underlyingPendleMarket
		)
);

```

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmTokenFactory.sol#L65-L69

### [L-3] Immutable feeds in `PendleLpOracle`, `PtOracleDerivative`, and `PtOraclePure`

Chainlink feeds in `PendleLpOracle`, `PtOracleDerivative`, and `PtOraclePure` are immutable and cannot be changed beyond deployment. There is slight risk to it because Chainlink could decide to deprecate a particular feed and deploy a new one at a different address. The contracts have no way to change to that feed unless they all get re-deployed. It's recommended to add a way for the master address to change the feeds.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PendleLpOracle.sol#L63
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOracleDerivative.sol#L49-L53
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/DerivativeOracles/PtOraclePure.sol#L47

## Informational

### [I-1] Various typos and non-existent words in the codebase

The codebase has typos and non-existent words in various places that should be fixed:

<details>
<summary>Click here to expand the list</summary>

<p>./contracts/MainHelper.sol:12:38 - Unknown word (devison)</p>
<p>./contracts/MainHelper.sol:50:37 - Unknown word (devison)</p>
<p>./contracts/MainHelper.sol:74:37 - Unknown word (devison)</p>
<p>./contracts/MainHelper.sol:79:14 - Unknown word (cashout)</p>
<p>./contracts/MainHelper.sol:87:17 - Unknown word (cashout)</p>
<p>./contracts/MainHelper.sol:93:15 - Unknown word (cashout)</p>
<p>./contracts/MainHelper.sol:109:36 - Unknown word (devison)</p>
<p>./contracts/MainHelper.sol:348:36 - Unknown word (reasoanbly)</p>
<p>./contracts/MainHelper.sol:369:25 - Unknown word (entrancy)</p>
<p>./contracts/MainHelper.sol:580:22 - Unknown word (increas)</p>
<p>./contracts/MainHelper.sol:581:28 - Unknown word (postion) fix: (position)</p>
<p>./contracts/MainHelper.sol:596:28 - Unknown word (postion) fix: (position)</p>
<p>./contracts/MainHelper.sol:645:14 - Unknown word (keccak)</p>
<p>./contracts/MainHelper.sol:655:16 - Unknown word (keccak)</p>
<p>./contracts/MainHelper.sol:731:11 - Unknown word (postion) fix: (position)</p>
<p>./contracts/MainHelper.sol:792:11 - Unknown word (postion) fix: (position)</p>
<p>./contracts/MainHelper.sol:812:11 - Unknown word (uncollateralized)</p>
<p>./contracts/MainHelper.sol:814:16 - Unknown word (Uncollateralized)</p>
<p>./contracts/MainHelper.sol:822:54 - Unknown word (Collateralized)</p>
<p>./contracts/MainHelper.sol:874:17 - Unknown word (intervall)</p>
<p>./contracts/MainHelper.sol:907:36 - Unknown word (maximise)</p>
<p>./contracts/MainHelper.sol:1004:8 - Unknown word (unorganic)</p>
<p>./contracts/MainHelper.sol:1076:27 - Unknown word (decresing)</p>
<p>./contracts/MainHelper.sol:1120:31 - Unknown word (decresing)</p>
<p>./contracts/PoolManager.sol:301:40 - Unknown word (NORMALISATION)</p>
<p>./contracts/PositionNFTs.sol:390:22 - Unknown word (bstr)</p>
<p>./contracts/PositionNFTs.sol:398:13 - Unknown word (bstr)</p>
<p>./contracts/PositionNFTs.sol:407:13 - Unknown word (bstr)</p>
<p>./contracts/WiseCore.sol:187:25 - Unknown word (postion) fix: (position)</p>
<p>./contracts/WiseCore.sol:425:8 - Unknown word (caluclating)</p>
<p>./contracts/WiseCore.sol:472:17 - Unknown word (cashout)</p>
<p>./contracts/WiseCore.sol:474:35 - Unknown word (cashout)</p>
<p>./contracts/WiseCore.sol:476:13 - Unknown word (cashout)</p>
<p>./contracts/WiseCore.sol:487:17 - Unknown word (cashout)</p>
<p>./contracts/WiseCore.sol:501:35 - Unknown word (cashout)</p>
<p>./contracts/WiseCore.sol:539:46 - Unknown word (functionallity)</p>
<p>./contracts/WiseCore.sol:560:35 - Unknown word (Cashout)</p>
<p>./contracts/WiseCore.sol:565:31 - Unknown word (Cashout)</p>
<p>./contracts/WiseCore.sol:565:69 - Unknown word (Cashout)</p>
<p>./contracts/WiseCore.sol:572:31 - Unknown word (Cashout)</p>
<p>./contracts/WiseCore.sol:572:64 - Unknown word (Cashout)</p>
<p>./contracts/WiseCore.sol:592:55 - Unknown word (Collateralized)</p>
<p>./contracts/WiseCore.sol:637:26 - Unknown word (Recieve) fix: (Receive)</p>
<p>./contracts/WiseCore.sol:661:26 - Unknown word (Recieve) fix: (Receive)</p>
<p>./contracts/WiseCore.sol:682:26 - Unknown word (Recieve) fix: (Receive)</p>
<p>./contracts/WiseLending.sol:32:26 - Unknown word (nside)</p>
<p>./contracts/WiseLending.sol:342:47 - Unknown word (Collateralized)</p>
<p>./contracts/WiseLending.sol:369:47 - Unknown word (Collateralized)</p>
<p>./contracts/WiseLending.sol:371:28 - Unknown word (Uncollateralized)</p>
<p>./contracts/WiseLending.sol:439:43 - Unknown word (collateralized)</p>
<p>./contracts/WiseLending.sol:458:43 - Unknown word (collateralized)</p>
<p>./contracts/WiseLending.sol:1243:37 - Unknown word (postion) fix: (position)</p>
<p>./contracts/WiseLending.sol:1270:21 - Unknown word (Recieve) fix: (Receive)</p>
<p>./contracts/WiseLending.sol:1312:34 - Unknown word (liqudaiton)</p>
<p>./contracts/WiseLending.sol:1337:21 - Unknown word (Recieve) fix: (Receive)</p>
<p>./contracts/WiseLending.sol:1354:25 - Unknown word (Recieve) fix: (Receive)</p>
<p>./contracts/WiseLending.sol:1410:13 - Unknown word (Mananger)</p>
<p>./contracts/WiseLending.sol:1534:35 - Unknown word (cashout)</p>
<p>./contracts/WiseLending.sol:1585:42 - Unknown word (reservating)</p>
<p>./contracts/WiseLendingDeclaration.sol:182:8 - Unknown word (Orace)</p>
<p>./contracts/WiseLendingDeclaration.sol:193:16 - Unknown word (Collateralized)</p>
<p>./contracts/WiseLendingDeclaration.sol:243:24 - Unknown word (Recieve) fix: (Receive)</p>
<p>./contracts/WiseLendingDeclaration.sol:305:8 - Unknown word (Norming)</p>
<p>./contracts/WiseLendingDeclaration.sol:307:31 - Unknown word (NORMALISATION)</p>
<p>./contracts/WiseLendingDeclaration.sol:317:13 - Unknown word (THRESHHOLD) fix: (THRESHOLD)</p>

</details>

## Recommendations

### [R-1] Add `onlyMaster` modifier to `PendlePowerFarmController::claimVoteRewards()` and `PendlePowerFarmController::claimArb()`

`PendlePowerFarmController::claimVoteRewards()` and `PendlePowerFarmController::claimArb()` should apply the `onlyMaster` modifier since they deal with sending funds to that address.

https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L581-L598
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/PowerFarms/PendlePowerFarmController/PendlePowerFarmController.sol#L431-L449
