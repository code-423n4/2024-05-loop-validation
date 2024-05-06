## L1   In `emergencyMode`  user should trasnfer erc20 token via `claim`.

 In `emergencyMode`,when  `block.timestamp >= startClaimDate` , user can withdraw erc20 token by `wihtdraw` or `claim` ,
  we should have the erc20 withdraw limit like logic in eth withdraw.
  If  `block.timestamp >= startClaimDate`, user should have only one way to withdraw  erc20 token.

  
```solidity
    function withdraw(address _token) external {
       if (!emergencyMode) {
            if (block.timestamp <= loopActivation) {
                revert CurrentlyNotPossible();
            }
            if (block.timestamp >= startClaimDate) {
                revert NoLongerPossible();
            }
        }

        uint256 lockedAmount = balances[msg.sender][_token];
        balances[msg.sender][_token] = 0;

        if (lockedAmount == 0) {
            revert CannotWithdrawZero();
        }

             if (_token == ETH) {
  @>          if (block.timestamp >= startClaimDate){
                revert UseClaimInstead();
            }
            totalSupply = totalSupply - lockedAmount;

            (bool sent,) = msg.sender.call{value: lockedAmount}("");

            if (!sent) {
                revert FailedToSendEther();
            }
        } else {
@>            IERC20(_token).safeTransfer(msg.sender, lockedAmount);
        }
```



Recommandation:

 else {
+       if (block.timestamp >= startClaimDate){
+                revert UseClaimInstead();
+            }
         IERC20(_token).safeTransfer(msg.sender, lockedAmount);
        }


