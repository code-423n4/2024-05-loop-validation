## Impact
When user runs `clamAndStake` function, unnecessary token transfer(self-transfer) will be running.
This can cause users to consume additional gas. 

## Vulnerability Detail
`claimAndStake` calls `_claim` internal function, and if `_token` is ETH, `_receiver` is contract itself.
Therefore it causes self-transfer that is no need to run in this case. 

```solidity
function claimAndStake(address _token, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        external
        onlyAfterDate(startClaimDate)
    {
@>      uint256 claimedAmount = _claim(_token, address(this), _percentage, _exchange, _data);
        lpETH.approve(address(lpETHVault), claimedAmount);
        lpETHVault.stake(claimedAmount, msg.sender);

        emit StakedVault(msg.sender, claimedAmount);
    }

function _claim(address _token, address _receiver, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        internal
        returns (uint256 claimedAmount)
    {
        uint256 userStake = balances[msg.sender][_token];
        if (userStake == 0) {
            revert NothingToClaim();
        }
        if (_token == ETH) {
            claimedAmount = userStake.mulDiv(totalLpETH, totalSupply);
            balances[msg.sender][_token] = 0;
@>          lpETH.safeTransfer(_receiver, claimedAmount); 
        } else {
            __SNIP__
        }
        emit Claimed(msg.sender, _token, claimedAmount);
    }
```

## Recommendation
```solidity
function _claim(address _token, address _receiver, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        internal
        returns (uint256 claimedAmount)
    {
        uint256 userStake = balances[msg.sender][_token];
        if (userStake == 0) {
            revert NothingToClaim();
        }
        if (_token == ETH) {
            claimedAmount = userStake.mulDiv(totalLpETH, totalSupply);
            balances[msg.sender][_token] = 0;
--          lpETH.safeTransfer(_receiver, claimedAmount); 
++          if (_receiver != address(this))
++                lpETH.safeTransfer(_receiver, claimedAmount); 
        } else {
            __SNIP__
        }
        emit Claimed(msg.sender, _token, claimedAmount);
    }
```