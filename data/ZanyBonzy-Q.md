## 1. Tokens cannot be disallowed

Links to affected code *

https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L364

### Impact
Tokens can be allowed but cannot be disallowed. In case of an admin error, and the wrong token being allowed, there's no way for this to be rectified. In cases of depeg or blackswan, slashing events, worthless tokens will be exchangable for `lpETH`.

```solidity
    function allowToken(address _token) external onlyAuthorized {
        isTokenAllowed[_token] = true;
    }
```
### Recommended Mitigation Steps
Consider introducing a function to disallow tokens.

***

## 2. Users being able to donate to `ETH` or `lpETH` before conversion one of protocol's main invariant 

Links to affected code *

https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L392

### Impact
From the readme, there's a main invariant - 

> Users that deposit ETH/WETH get the correct amount of lpETH on claim (1 to 1 conversion)

This 1 to 1 invariant can be easily broken through donations, before the `convertAllETH` function is called. The function queries the balance of ETH before conerting all to lpETH, and queries the contract's lpETH balance to set the `totalLpETH` parameter. The conversion rate is 1 to 1.

```solidity
    function convertAllETH() external onlyAuthorized onlyBeforeDate(startClaimDate) {
        if (block.timestamp - loopActivation <= TIMELOCK) {
            revert LoopNotActivated();
        }

        // deposits all the ETH to lpETH contract. Receives lpETH back
        uint256 totalBalance = address(this).balance;
        lpETH.deposit{value: totalBalance}(address(this));

        totalLpETH = lpETH.balanceOf(address(this));

        // Claims of lpETH can start immediately after conversion.
        startClaimDate = uint32(block.timestamp);

        emit Converted(totalBalance, totalLpETH);
    }
```
Note the `totalSupply` tracks `ETH` deposits only through the [`_processLock`](https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L180-L181) function, meaning it isn't affected by donations. So if tokens had been donated, they'll be converted to `lpETH` without actually changing the totalSupply.

When claiming, the claimed amount a user is entitled to is calculated as `userStake` * `totalLpETH`/`totalSupply`. And as it has been established that `totalLpETH` was a 1 to 1 exchange, this means that it will be more than `totalSupply` (since donations had increased `address(this).balance` but didn't affect `totalSupply`). As `totalLpETH` is more than `totalSupply`, a user's claimed amount will not be in a 1 to 1 ratio with his staked amount, but rather more.

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
        }
        ...
```
The same also applies if `lpETH` is donated to the contract before ETH is converted, due to the use of `lpETH.balanceOf(address(this));` to set `totalLpETH`.

Important to note that this doesn't affect ERC20 stakers.

### Recommended Mitigation Steps
Best way to fix it is to convert only `totalSupply` of ETH, and setting `totalLpETH` to `totalSupply`. That way, the 1 to 1 ratio will be protected. Donations will not be taken into consideration.

```solidity
    function convertAllETH() external onlyAuthorized onlyBeforeDate(startClaimDate) {
        if (block.timestamp - loopActivation <= TIMELOCK) {
            revert LoopNotActivated();
        }

        // deposits all the ETH to lpETH contract. Receives lpETH back
        lpETH.deposit{value: totalSupply}(address(this)); //++++

        totalLpETH = totalSupply; //++++ 

        // Claims of lpETH can start immediately after conversion.
        startClaimDate = uint32(block.timestamp);

        emit Converted(totalBalance, totalLpETH);
    }
```

***