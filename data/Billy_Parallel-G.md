check token.code.length first before exec the call to save gas.

***Lines of code***
https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/TransferHub/CallOptionalReturn.sol#L12-L38

***Vulnerability details***
if token.code.length == 0 , exec the call will lose some gas.

***Recommended Mitigation***

    function _callOptionalReturn(
        address token,
        bytes memory data
    )
        internal
        returns (bool callResults)
    {
        if (token.code.length > 0) {
            (
                bool success,
                bytes memory returndata
            ) = token.call(
                data
            );
            if (success == false) {
                revert();
            }
            callResults = returndata.length == 0 || abi.decode(returndata,(bool));
        }
    }