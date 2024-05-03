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

# [L-02] Handling of Unintended ETH Deposits in PrelaunchPoints Contract
The PrelaunchPoints contract currently faces challenges with unintended ETH deposits, particularly in how these are handled during different operational phases of the contract. In the locking phase, stray ETH contributions are added to `totalLpETH`, fairly increasing the total pool available for distribution among users. However, during the claiming phase, any ETH inadvertently sent can be unintentionally claimed by the next user converting a token other than ETH, leading to potential unfair advantages and disruptions in intended token distributions.

To address these issues, it is proposed to modify the `_claim` function to ensure that only ETH intended for conversion during a claim is used by tracking the ETH balance before and after conversion. Additionally, an authorized recovery function similar to `recoverERC20` should be implemented for ETH to manage and potentially return unintended deposits after the claiming phase has begun. These changes aim to enhance fairness, prevent manipulation, and increase trust in the contract’s operations by providing a clear and controlled method for handling stray ETH deposits.

Proposed solutions:
1. Track Received ETH During Claiming: 

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L259-L262

```diff
+            uint256 initialBalance = address(this).balance;
            _fillQuote(IERC20(_token), userClaim, _data);
+            uint256 newBalance = address(this).balance;

            // Convert swapped ETH to lpETH (1 to 1 conversion)
-            claimedAmount = address(this).balance;
+            claimedAmount = newBalance - initialBalance;
```
2. Authorized Retrieval Function: 

```solidity
function recoverETH(uint256 amount) external onlyAuthorized onlyAfterDate(startClaimDate){
    require(address(this).balance >= amount, "Insufficient ETH balance");
    (bool sent,) = owner.call{value: amount}("");
    require(sent, "Failed to send ETH");
    emit RecoveredETH(amount);
}
```
## [L-03] Discrepancy in Staking Timelines: A Possibly Associated Issue in PrelaunchPoints Contract
The PrelaunchPoints contract exhibits a significant design flaw related to the staking mechanism timing due to its substantial impact on fairness and user trust. The issue arises because, after the [setLoopAddresses](https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L348-L358) function is called, non-lockers can deposit ETH in LpETH and stake ETH in lpETHVault during a [7-day waiting period](https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L316), while lockers must wait until this period elapses to claim and stake their lpETH. This discrepancy not only provides non-lockers with an undue advantage in earning potential but also risks filling staking quotas (if ever present) before lockers can participate. Such a situation could demotivate users from participating in future locking events and undermine trust in the platform. Addressing this flaw by synchronizing staking opportunities, possibly through a reserve mechanism or governance adjustments, is crucial to maintaining fairness and ensuring the platform's integrity and attractiveness to all users.
