1. `claimedAmount` in the `Claimed` event can be increased by sending ETH before a `claim()` call (with a flashloan, for example). The funds will be returned back at the end of the call as lpETH tokens. Potentially a medium/high if there's some reliance on the event in the offline part of the points mechanism.

https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L261-L265
```solidity
            claimedAmount = address(this).balance;
            lpETH.deposit{value: claimedAmount}(_receiver);
        }
        emit Claimed(msg.sender, _token, claimedAmount);
```

2. `convertAllETH()` will revert with an underflow if the function is called when `loopActivation` >= `block.timestamp` (e.g., right after the deployment) because of the subtraction. Consider adding another custom error:

https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L316

```diff
+       if (block.timestamp < loopActivation) {
+           revert CurrentlyNotPossible();
+       }
        if (block.timestamp - loopActivation <= TIMELOCK) {
            revert LoopNotActivated();
        }
```

3. Calculations in `_claim()` (see below) will revert with an underflow if the `_percentage` argument is > 100. Consider adding adequate validation with a custom error. 

https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L253-L255

```solidity
            uint256 userClaim = userStake * _percentage / 100;
            _validateData(_token, userClaim, _exchange, _data);
            balances[msg.sender][_token] = userStake - userClaim;
```

4. The conversion to `payable` and `{value: 0}` are not necessary during the call in `_fillQuote` because they have no effect on the call:

https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L497

```solidity
        (bool success,) = payable(exchangeProxy).call{value: 0}(_swapCallData);
```

5. Both exchange routes during the claim process offer no deadline parameter for the swap. There's the slippage protection in place, so the impact is low.

https://github.com/code-423n4/2024-05-loop/blob/35f272b8f60c9b1d8c5c04e63db172c42e2b6e97/src/PrelaunchPoints.sol#L211-L216

```solidity
    function claim(address _token, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        external
        onlyAfterDate(startClaimDate)
    {
        _claim(_token, msg.sender, _percentage, _exchange, _data);
    }
```