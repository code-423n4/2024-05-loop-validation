## L1  `_validateData` fail to validate uniswap recipient address is zero 
 
in current  `_validateData`  function , two  exechange type are accecpteble , one is `uniswap` and another is `transfer`,
param `recipient` only used in `uniswap` exchange type, so it's required to equal to `address(this)`
another exchange type `transfer` , `recipient` should be `address(0)`, because it's not used in `transfer` exchange type.

The issue is if user  pass `recipient` as `address(0)` in `uniswap` exchange type, it will send the token to `address(0)`,causing the loss of token.
 



```solidity
function _validateData(address _token, uint256 _amount, Exchange _exchange, bytes calldata _data) internal view {
  
  ...
@>     if (recipient != address(this) && recipient != address(0)) {
      revert WrongRecipient(recipient);
    }
   } 
```



## L2   In `emergencyMode`  user should trasnfer erc20 token via `claim`.

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



