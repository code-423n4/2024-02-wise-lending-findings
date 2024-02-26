# Enhancing Gas Efficiency Through Refined _checkReentrancy Trimming

# Description

* There are two if statements check in _checkReentrancy(). 1.sendingProgress, 2._sendingProgressAaveHub(). `Line: 343 WiseLowLevelHelper.sol#L358` second one is call another private function checks. this method check are consume lot of gas ( more then 2162 gas) . 

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
