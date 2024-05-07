
``L1 - user will receive all the balance of contract instead of userClaim``
claim function is supposed to claim the staked amount but instead will claim balance of the contract.
 ```solidity
    function _claim(address _token, address _receiver, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        internal
        returns (uint256 claimedAmount)
    {
      ////  
        } else {
            uint256 userClaim = userStake * _percentage / 100;
            _validateData(_token, userClaim, _exchange, _data);
            balances[msg.sender][_token] = userStake - userClaim;

            // At this point there should not be any ETH in the contract
            // Swap token to ETH
            _fillQuote(IERC20(_token), userClaim, _data);

            // Convert swapped ETH to lpETH (1 to 1 conversion)
            claimedAmount = address(this).balance;
            lpETH.deposit{value: claimedAmount}(_receiver); // here @audit
        }
        emit Claimed(msg.sender, _token, claimedAmount);
    }
```
Recommendation
```solidity
  claimedAmount = userClaim;
            lpETH.deposit{value: claimedAmount}(_receiver); 

```
``L2 - in _claim function both the if statements is intended to do the same thing but when the ``_token== ETH`` it uses ``   lpETH.safeTransfer`` instead of ``  lpETH.deposit`` 
```solidity
        if (_token == ETH) {
           ////
            lpETH.safeTransfer(_receiver, claimedAmount);
        } else {
           ////
            lpETH.deposit{value: claimedAmount}(_receiver);
        }
        emit Claimed(msg.sender, _token, claimedAmount);
    }

```
Recpmmendation
```solidity
 
   if (_token == ETH) {
           ////
            lpETH.deposit{value: claimedAmount}(_receiver);
        } else {
           ////
            lpETH.deposit{value: claimedAmount}(_receiver);
        }
        emit Claimed(msg.sender, _token, claimedAmount);
    }
```
``L3 - wrong variable set in claimed event in function _claim``
```solidity
            // Convert swapped ETH to lpETH (1 to 1 conversion)
            claimedAmount = address(this).balance;
            lpETH.deposit{value: claimedAmount}(_receiver);
        }
        emit Claimed(msg.sender, _token, claimedAmount); //@audit should be userClaim.
    }
```
Recommendation
```solidity
 emit Claimed(msg.sender, _token, userClaim);
```


``NC-1 - 0x address check in function setOwner``
Recommendation
```solidity
unction setOwner(address _owner) external onlyAuthorized {
        if(owner == address(0){
        revert():
         }
        owner = _owner; 
        emit OwnerUpdated(_owner);
    }
```
``NC-2 make code more redeable ``
```solidity
 totalSupply = totalSupply + _amount; 
```
Recommendation
```solidity
 totalSupply += _amount; 
```