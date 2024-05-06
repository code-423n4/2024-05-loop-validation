# L[1] Users can get more LPETH than the locked assets.
## Detail
```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L262C10-L263C60
```
Users can deposit ETH into the contract in advance, so they can claim LpETH without locking the assets.
## Suggestion
```solidity
+       BeforeSwapAmount = address(this).balance;

        uint256 userClaim = userStake * _percentage / 100;
        _validateData(_token, userClaim, _exchange, _data);
        balances[msg.sender][_token] = userStake - userClaim;

        // At this point there should not be any ETH in the contract
        // Swap token to ETH
        _fillQuote(IERC20(_token), userClaim, _data);

+       AfterSwapAmount = address(this).balance; 
        // Convert swapped ETH to lpETH (1 to 1 conversion)
-       claimedAmount = address(this).balance;
+       claimedAmount = AfterSwapAmount - BeforeSwapAmount;
        lpETH.deposit{value: claimedAmount}(_receiver);
```
# NC[1] Missing check for _receiver

## Detail
```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L172
```
Check _receiver to prevent it from being 0x00

## Suggestion
```solidity

    function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
        internal
        onlyBeforeDate(loopActivation)
    {
        if (_amount == 0) {
            revert CannotLockZero();
        }
+       if(_receiver == address(0)) {
+           revert CannotLockForInvalidReceiver(address(0));
+       }
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
```
# NC[2]  Missing events

## Detail
```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L364

https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L372
```

## Suggestion
```solidity
    function allowToken(address _token) external onlyAuthorized {
        isTokenAllowed[_token] = true;
+       emit AllowToken(_token);    
    }

    function setEmergencyMode(bool _mode) external onlyAuthorized {
        emergencyMode = _mode;
+       emit SetEmergencyMode();
    }
```
# NC[3] Incomplete event parameters
## Detail
```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L379
```
I suggest adding onwer’s address to the event.
## Suggestion
```solidity
    function recoverERC20(address tokenAddress, uint256 tokenAmount) external onlyAuthorized {
        if (tokenAddress == address(lpETH) || isTokenAllowed[tokenAddress]) {
            revert NotValidToken();
        }
        IERC20(tokenAddress).safeTransfer(owner, tokenAmount);

-       emit Recovered(tokenAddress, tokenAmount);
+       emit Recovered(tokenAddress, onwer,tokenAmount);
    }
```