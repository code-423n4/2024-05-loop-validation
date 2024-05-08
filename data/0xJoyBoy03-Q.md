## [L-1] Users who sent and locked `WETH` can't claim their corresponding token
### Description
after the `startClaimDate` Users who locked `WETH`, can't claim their `lpETH` tokens. 
The protocol unwraps WETH to ETH when locking but doesn't handle the `WETH` pretty well at the claim function
```java
function _claim(address _token, address _receiver, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        internal
        returns (uint256 claimedAmount)
    {
        // @audit-low: it gets reverted with `WETH` 
@>        uint256 userStake = balances[msg.sender][_token];
        if (userStake == 0) {
            revert NothingToClaim();
        // ...
}
```
Since the protocol doesn't have any `balances[msg.sender][WETH]` when a user calls the claim function with the `WETH` as `_token` parameter, the function gets reverted with the `NothingToClaim()` error but the user does have something to claim but the protocol doesn't handle that 

### Recommended Mitigation you can modify the `claim` function to something like this:
```git
    function claim(address _token, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        external
        onlyAfterDate(startClaimDate)
    {
+        _claim(_token == ETH || _token == WETH ? ETH : _token,
            msg.sender,
           _percentage,
           _exchange,
           _data);
    }
```
you should also do this for the `claimAndStake` function either


