in CallOptionalReturn.sol , in function _callOptionalReturn:
check token.code.length first before exec the call to save gas.

org code :
    function _callOptionalReturn(
        address token,
        bytes memory data
    )
        internal
        returns (bool call)
    {
        (
            bool success,
            bytes memory returndata
        ) = token.call(
            data
        );

        bool results = returndata.length == 0 || abi.decode(
            returndata,
            (bool)
        );

        if (success == false) {
            revert();
        }

        call = success
            && results
            && token.code.length > 0;
    }

Gas Optimizations :
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