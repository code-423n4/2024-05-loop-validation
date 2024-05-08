## Typo In NatSpec could be misleading
**Line of Code**
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L154

**Correction:**
```diff
/**
     * @notice Locks a valid token for a given address
     * @param _token     address of token to lock
     * @param _amount    amount of token to lock
+    * @param _for       address for which token is locked
-    * @param _for       address for which ETH is locked
     * @param _referral  info of the referral. This value will be processed in the backend.
     */
```