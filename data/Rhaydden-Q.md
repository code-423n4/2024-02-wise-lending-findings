## Issue 1: Typo
(https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmDeclarations.sol#L102)
Correct `FIVTY` to `FIFTY`


## Issue 2: No Event Logging Statements in Key Functions 

(https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L77-L139)

Mitigation Step:
- Include `emit` statements in the `_reserveKey` and `_mintKeyForUser` functions to log key reservations and minting activities.

## Issue 3: The `onERC721Received` Function
(https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PowerFarmNFTs/MinterReserver.sol#L160-L178)
Mitigation Step:
- Clarify that the `onERC721Received` function is an ERC721 callback function and is not used within the contract itself for logging key reservations.


## Issue 4: The `setFarmContract` function does not update the `farmContract` variable if it is already set to a non-zero address. 
(https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L51-L60)

Mitigation step:
The `setFarmContract` function is designed to set the `farmContract` address only if it hasn't been set before (i.e., if it's equal to ZERO_ADDRESS). This is a common pattern to prevent the contract from being reconfigured after it has been initialized. However, if the intention is to allow the farmContract to be updated, this check should be removed or modified to allow updates under certain conditions.


## Issue 5: Lack of events emission

https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/CustomOracleSetup.sol#L1-L68

For a contract like CustomOracleSetup, which involves setting and updating various parameters (e.g., lastUpdateGlobal, globalRoundId, timeStampByRoundId), it would be beneficial to emit events to notify external observers about these changes.

Here's how it can be corrected:

```solidity
// SPDX-License-Identifier: -- WISE --

pragma solidity =0.8.24;

contract CustomOracleSetup {

    address public master;
    uint256 public lastUpdateGlobal;

    uint80 public globalRoundId;

    mapping(uint80 => uint256) public timeStampByRoundId;

    // Event declarations
    event OwnershipRenounced(address indexed previousOwner);
    event LastUpdateGlobalSet(uint256 indexed newTime);
    event RoundDataSet(uint80 indexed roundId, uint256 indexed updateTime);
    event GlobalAggregatorRoundIdSet(uint80 indexed newAggregatorRoundId);

    modifier onlyOwner() {
        require(
            msg.sender == master,
            "CustomOracleSetup: NOT_MASTER"
        );
        _;
    }

    constructor() {
        master = msg.sender;
    }

    function renounceOwnership()
        external
        onlyOwner
    {
        master = address(0x0);
        emit OwnershipRenounced(msg.sender);
    }

    function setLastUpdateGlobal(
        uint256 _time
    )
        external
        onlyOwner
    {
        lastUpdateGlobal = _time;
        emit LastUpdateGlobalSet(_time);
    }

    function setRoundData(
        uint80 _roundId,
        uint256 _updateTime
    )
        external
        onlyOwner
    {
        timeStampByRoundId[_roundId] = _updateTime;
        emit RoundDataSet(_roundId, _updateTime);
    }

    function setGlobalAggregatorRoundId(
        uint80 _aggregatorRoundId
    )
        external
        onlyOwner
    {
        globalRoundId = _aggregatorRoundId;
        emit GlobalAggregatorRoundIdSet(_aggregatorRoundId);
    }

    function getTimeStamp()
        external
        view
        returns (uint256)
    {
        return block.timestamp;
    }
}
```
In this modified contract, I've added four events:

**OwnershipRenounced:** Emitted when the ownership is renounced.
**LastUpdateGlobalSet:** Emitted when the lastUpdateGlobal is set.
**RoundDataSet:** Emitted when round data is set.
**GlobalAggregatorRoundIdSet:** Emitted when the globalRoundId is set.
These events are emitted at the end of the functions that modify the state, providing a clear and transparent log of the contract's activity.



## Issue 6:Incorrect Event Emission in Ownership Claim

(https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/OwnableMaster.sol#L92-L114)

The issue is related to the `claimOwnership` function in the `OwnableMaster.sol` contract. The function is intended to allow the proposed master to claim ownership, but it does not emit an event that accurately reflects this change. Specifically, the event should emit the address of the new master and the address of the previous master, but it incorrectly emits the address of the new master for both parameters.

To fix this issue, a new event ClaimedOwnership should be introduced, and the claimOwnership function should be modified to emit this event with the correct parameters. The new master's address should emit as msg.sender, and the previous master's address should be stored in a variable before the ownership is transferred and emitted as previousMaster.

Here is how to modify to modify it:

```solidity

// Define the new event
event ClaimedOwnership(
    address indexed newMaster,
    address indexed previousMaster
);

function claimOwnership()
    external
    onlyProposed
{
    address previousMaster = master;
    master = proposedMaster;

    emit ClaimedOwnership(
        msg.sender,
        previousMaster
    );
}
```

