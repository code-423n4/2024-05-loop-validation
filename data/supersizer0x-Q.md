### When a user is withdrawing according to the spec they should be able to withdraw right after `loopActivation` but that is not true in the code 
```solidity
  if (block.timestamp <= loopActivation) {
```
[CodeLink](https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L269C1-L283C1)
As we can see above when a user calls this function they will have to wait until `block.timestamp+ 12(one block)` after `loopAcitivation` which goes against what the spec says!
```solidity 
  * Note Can only be called after the loop address is set and before claiming lpETH,
```
Furthermore the choice of using the word `after` implies that a user can withdraw right which is not something the code confirms.
