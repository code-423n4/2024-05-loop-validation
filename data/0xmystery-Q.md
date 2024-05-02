## [L-01] Inability to Invalidate Erroneously Allowed LRT Tokens
The lack of functionality to invalidate erroneously allowed LRT tokens, identified within the `allowToken` function, permits users to lock these unsupported tokens without the ability to subsequently utilize them for intended functions such as converting to lpETH via `claim` and `claimAndStake` functions. Notably, these tokens become irretrievable after the 7-day TIMELOCK passes when `startClaimDate` is assigned `uint32(block.timestamp)` in `convertAllETH()`. Turning on emergency mode just to withdraw these tokens could be seen as an excessive measure, especially since it affects the entire contract and not just the erroneous tokens.

Consider having `allowToken()` refactored as follows:

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L364-L366

```diff
-    function allowToken(address _token) external onlyAuthorized {
+    function allowToken(address _token, bool _mode) external onlyAuthorized {
-        isTokenAllowed[_token] = true;
+        isTokenAllowed[_token] = _mode;
    }
```
Where possible, consider refining the emergency mode too to allow selective operations rather than a blanket override. This could help manage specific issues without impacting the entire system's integrity.
