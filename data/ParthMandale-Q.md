# LoopFi QA Report

## [L-01] Inconsistent Token Balance Handling for WETH Claims

### Context

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L245

### Description

In `function _processLock` there is a separate condition for `WETH` to be
locked - `if (_token == address(WETH))` and in that condition the `WETH` is been
converted into `ETH` and added to the `totalSupply`, and user's balance of `ETH`
(not ERC20 WETH _token)

`PrelaunchPoints.sol#L188`
```
if (_token == address(WETH)) {
WETH.withdraw(_amount);
totalSupply = totalSupply + _amount;
@> balances[_receiver][ETH] += _amount;
}
```

Issue comes when user calls `function _claim` with input param to be WETH token
(because that's what user locked, so now he will claim that token only) But here
user will be getting a revert because line #245 in `function _claim` because
user's WETH balance is not updated instead user's ETH balance is updated in
`function _processLock` -

`PrelaunchPoints.sol#L245`
```
// fetch user stake
uint256 userStake = balances[msg.sender][_token];
// if user has nothing to claim revert the tx
@> if (userStake == 0) {
revert NothingToClaim();
}
```

### Recommendation

To avoid getting revert with WETH token while claiming for locked WETH token,
considering implementing this in `function _claim`

```
uint256 userStake;

if ( _token == address(WETH)) {
 userStake = balances[msg.sender][ETH];
}

userStake = balances[msg.sender][_token];

  if (userStake == 0) {
            revert NothingToClaim();
        }

if (_token == ETH || _token == address(WETH)) {
            claimedAmount = userStake.mulDiv(totalLpETH, totalSupply);
            balances[msg.sender][_token] = 0;
            lpETH.safeTransfer(_receiver, claimedAmount);
        }

```


## [L-02] Missing Percentage Check for ERC20 Token Claims

### Context

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L252

### Description

In `function -_claim` else block, which handles ERC20 token claims, it lacks a check to ensure that the claim percentage (`_percentage` input param) is greater than 0. failing to validate inputs can lead to unexpected behavior and could potentially be exploited in more complex contracts or in combination with other oversights.

### Recommendation

``` 
 // ... existing code ...

else {
      @>    require(_percentage > 0, "Percentage must be greater than 0");    
            uint256 userClaim = userStake * _percentage / 100;
            _validateData(_token, userClaim, _exchange, _data);
            balances[msg.sender][_token] = userStake - userClaim;

 // ... existing code ...
```

## [L-03] 