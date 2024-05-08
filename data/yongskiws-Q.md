### QA-1 there is no checking of the user's balance before adding the amount of locked tokens to their balance

lock and lockFor allow users to lock any ERC20 token that is in the allowedTokens mapping. However, the contract does not check the balance of the token in the user's account before allowing the lock. This could potentially allow an attacker to lock tokens that they do not actually own.

Impact:
```
    function lockFor(address _token, uint256 _amount, address _for, bytes32 _referral) external {
        if (_token == ETH) {
            revert InvalidToken();
        }
        _processLock(_token, _amount, _for, _referral);
    }

    function lock(address _token, uint256 _amount, bytes32 _referral) external {
        if (_token == ETH) {
            revert InvalidToken();
        }
        _processLock(_token, _amount, msg.sender, _referral);
    }

```

Recommendation:
Before adding the number of locked tokens to the user's balance, be sure to check that the user has a sufficient balance of tokens to be locked and can do this by using the balanceOf function of the appropriate ERC20 contract.


### QA-2 no validation of the user's LP token balance before making a claim.

The claim and claimAndStake allow users to claim their vested LP tokens and convert them to ERC20 LP tokens. However, the contract does not check the balance of the LP tokens in the user's account before allowing the claim. This could potentially allow an attacker to claim LP tokens that they do not actually own.

Impact:

```
   function claim(address _token, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        external
        onlyAfterDate(startClaimDate)
    {
        _claim(_token, msg.sender, _percentage, _exchange, _data);
    }

    function claimAndStake(address _token, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        external
        onlyAfterDate(startClaimDate)
    {
        uint256 claimedAmount = _claim(_token, address(this), _percentage, _exchange, _data);
        lpETH.approve(address(lpETHVault), claimedAmount);
        lpETHVault.stake(claimedAmount, msg.sender);

        emit StakedVault(msg.sender, claimedAmount);
    }
```

Recommendation:
consider checking the user's LP token balance before allowing a claim

### QA-3 Add checks to the withdraw function to ensure that the contract has a sufficient balance of the token before allowing the withdrawal.

Impact:
```
    function withdraw(address _token) external {
```

Recommendation: ensure that there is sufficient token balance before allowing a withdrawal

```
uint256 lockedAmount = balances[msg.sender][_token];
require(lockedAmount > 0, "No tokens to withdraw");
```

### QA-4 Provide documentation or comments in the contract that explain what happens in emergency mode and how it can be used.

```
    function recoverERC20(address tokenAddress, uint256 tokenAmount) external onlyAuthorized {
        if (tokenAddress == address(lpETH) || isTokenAllowed[tokenAddress]) {
            revert NotValidToken();
        }
        IERC20(tokenAddress).safeTransfer(owner, tokenAmount);

        emit Recovered(tokenAddress, tokenAmount);
    }
```