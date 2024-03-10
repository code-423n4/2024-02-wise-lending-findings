Typographical Errors: The PayedBackBadDebtFree event has a typo in the timestamp parameter (timestampp), which could lead to issues if the event is emitted with the wrong parameter name.

 https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/FeeManager/FeeManagerEvents.sol#L129C4-L135C7
event PayedBackBadDebtFree(
        uint256 indexed nftId,
        address indexed sender,
        address indexed paybackToken,
        uint256  paybackAmount,
        uint256 timestampp
    );