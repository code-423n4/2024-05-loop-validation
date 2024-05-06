https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L244-L255

The ```_claim()``` function first calculates``` uint256 userStake = balances[msg.sender][_token]```.  Further, if the first 2 conditions do not pass, it goes to the else section and here it calculates ```balances[msg.sender][_token] = userStake - userClaim```, but it makes no sense to use the userStake variable, because it wastes more gas, however, at code execution time, the ```userStake``` variable and ```balances[_account][_token]``` have the same value, but the reduced subtraction will be less costly than using ```userStake```.

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
            lpETH.safeTransfer(_receiver, claimedAmount);
        } else {
            uint256 userClaim = userStake * _percentage / 100;
            _validateData(_token, userClaim, _exchange, _data);
            balances[msg.sender][_token] = userStake - userClaim;
           ...
```

It is recommended to change to an abbreviated form of entry.

```balances[_account][_token] -= userClaim```