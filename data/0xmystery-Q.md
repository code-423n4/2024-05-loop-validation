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

For this reason, it's recommended having `lpETHVault` deployed and set when/after `convertAllETH()` is called:

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L348-L358

```diff
    function setLoopAddresses(address _loopAddress, address _vaultAddress)
        external
        onlyAuthorized
        onlyBeforeDate(loopActivation)
    {
        lpETH = ILpETH(_loopAddress);
-        lpETHVault = ILpETHVault(_vaultAddress);
        loopActivation = uint32(block.timestamp);

        emit LoopAddressesUpdated(_loopAddress, _vaultAddress);
    }
``` 
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L315-L330

```diff
-    function convertAllETH() external onlyAuthorized onlyBeforeDate(startClaimDate) {
+    function convertAllETH() external onlyAuthorized onlyBeforeDate(startClaimDate, address _vaultAddress) {
        if (block.timestamp - loopActivation <= TIMELOCK) {
            revert LoopNotActivated();
        }

        // deposits all the ETH to lpETH contract. Receives lpETH back
        uint256 totalBalance = address(this).balance;
        lpETH.deposit{value: totalBalance}(address(this));

        totalLpETH = lpETH.balanceOf(address(this));

        // Claims of lpETH can start immediately after conversion.
        startClaimDate = uint32(block.timestamp);

+        lpETHVault = ILpETHVault(_vaultAddress);

        emit Converted(totalBalance, totalLpETH);
    }
```
Note: The related events will have to be revised to cater to the above changes though.

## [L-04] Enhanced Flexibility with Conditional Referral Verification in PrelaunchPoints Contract
To improve user experience and flexibility, the `_processLock` function in the PrelaunchPoints contract should be enhanced to conditionally verify referral codes. This update allows users to choose whether to participate in the referral program by providing a referral code along with a corresponding Merkle proof. If no referral code is provided (i.e., the referral code is set to zero), the function skips the verification step, allowing the transaction to proceed without any referral-related checks. This change ensures that the referral system remains secure and robust for users who opt to use it, while also providing simplicity and inclusivity for those who prefer not to engage with referrals.

The function now checks if a referral code is non-zero before executing the referral verification logic. If a referral is provided, the verifyReferral function validates it against a stored Merkle root in the contract. This ensures that only valid referrals influence the system's behavior and rewards distribution in the backend. If the verification fails, the transaction reverts with an "Invalid referral" error message, preventing unauthorized actions. 

By implementing this optional referral verification, the PrelaunchPoints contract caters to a wider array of user preferences and situations, enhancing the overall usability of the system. This approach not only boosts user autonomy by allowing them to decide on the use of referrals but also maintains high security standards for referral processing.

Here're the suggested fixes:

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L172-L198

```diff
-    function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
+    function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral, bytes32[] memory _proof)
        internal
        onlyBeforeDate(loopActivation)
    {
+        // Verify the referral only if provided
+        if (_referral != 0) {
+            require(verifyReferral(_referral, _proof), "Invalid referral");
+        }

        if (_amount == 0) {
            revert CannotLockZero();
        }
        if (_token == ETH) {
            totalSupply = totalSupply + _amount;
            balances[_receiver][ETH] += _amount;
        } else {
            if (!isTokenAllowed[_token]) {
                revert TokenNotAllowed();
            }
            IERC20(_token).safeTransferFrom(msg.sender, address(this), _amount);

            if (_token == address(WETH)) {
                WETH.withdraw(_amount);
                totalSupply = totalSupply + _amount;
                balances[_receiver][ETH] += _amount;
            } else {
                balances[_receiver][_token] += _amount;
            }
        }

        emit Locked(_receiver, _amount, _token, _referral);
    }

+    // The verifyReferral function, as defined earlier
+    function verifyReferral(bytes32 _referral, bytes32[] memory proof) public view returns (bool) {
+        bytes32 computedHash = _referral;
+        for (uint256 i = 0; i < proof.length; i++) {
+            computedHash = keccak256(abi.encodePacked(computedHash, proof[i]));
+        }
+        return computedHash == merkleRoot;
+    }
```