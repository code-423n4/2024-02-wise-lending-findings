## 1. Use named returns for local variables of pure functions where it is possible

## Description

* Streamline return values: A simple yet effective optimisation technique is to name the return value in a function, eliminating the need for a separate local variable. For instance, in a function that calculates a product, you can directly name the return value, streamlining the process



##  Proof of Concept


```
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

test for test/NamedReturn.t.sol:NamedReturnTest
[PASS] test_Increment() (gas: 27613)


```

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
test for test/NoNamedReturn.t.sol:NamedReturnTest
[PASS] test_Increment() (gas: 27639)




* There are 3 instances of this issue:


https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseCore.sol#L611



https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WrapperHub/AaveHub.sol#L640



https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/DerivativeOracles/CustomOracleSetup.sol#L64



https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PowerFarmNFTs/PowerFarmNFTs.sol#L114



https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/PowerFarms/PendlePowerFarm/PendlePowerFarmMathLogic.sol#L179


https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLending.sol#L170



###################################################################################

## 2. Enhancing Gas Efficiency Through Refined _checkReentrancy Trimming

# Description

* There are two if statements check in _checkReentrancy(). 1.sendingProgress, 2. _sendingProgressAaveHub(). `Line: 343 WiseLowLevelHelper.sol#L358` second one is call another private function checks. this method check are consume lot of gas ( more then 2162 gas) . 

* https://github.com/code-423n4/2024-02-wise-lending/blob/79186b243d8553e66358c05497e5ccfd9488b5e2/contracts/WiseLowLevelHelper.sol#L354-L369

``` 

function _checkReentrancy()
        internal
        view
    {
+        if (sendingProgress ||  IAaveHubLite(AAVE_HUB_ADDRESS).sendingProgressAaveHub() ) {
+            revert InvalidAction();
+        }

-		if (sendingProgress == true) {
-           revert InvalidAction();
-        }

-        if (_sendingProgressAaveHub() == true) {
-            revert InvalidAction();
-        }
    }



-  function _sendingProgressAaveHub()  private view returns (bool)   { 
-        return IAaveHubLite(AAVE_HUB_ADDRESS).sendingProgressAaveHub();
-    }

```

##################################################################################

## 3. Multiple accesses of a msg.value should use a local variable cache


* Begin by verifying that msg.value is not equal to zero initially, as this precaution can result in significant gas savings. Additionally, there is no need to perform a check within the _validateNonZero(paybackShares) function.


* https://github.com/code-423n4/2024-02-wise-lending/blob/main/contracts/WiseLending.sol#L1088-L1155

```
function paybackExactAmountETH(
        uint256 _nftId
    )
 ...
    {
...
+		require(msg.value != 0, "Empty value," )
+		uint256 senderAmount = msg.value;	

        uint256 paybackShares = calculateBorrowShares(
            {
                _poolToken: WETH_ADDRESS,
-                _amount: msg.value,
+              _amount: senderAmount,
                _maxSharePrice: false
            }
        );

-        _validateNonZero(
-           paybackShares
-        );

-        uint256 requiredAmount = msg.value;

-        if (msg.value > maxPaybackAmount) {
+        if (msg.value > senderAmount) {

            unchecked {
-                refundAmount = msg.value
+ 				 refundAmount = senderAmount
                    - maxPaybackAmount;
            }
...
...
    }

```