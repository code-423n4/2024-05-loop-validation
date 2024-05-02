## Inefficiency in Function `_claim`

The function `_claim` in the PrelaunchPoints contract has an inefficiency where the `claimedAmount` for non-ETH tokens is only set after calling `_fillQuote`. It didn't explicitly make use of the amount of ETH received from the swap (tracked in `_fillQuote`) for further processing. Instead, it implicitly depended on the assumption that all ETH held by the contract after the swap was a result of the swap. This can be risky if the contract holds ETH from other sources or processes.

### Original Code:

```
function _claim(
    address _token,
    address _receiver,
    uint8 _percentage,
    Exchange _exchange,
    bytes calldata _data
) internal returns (uint256 claimedAmount) {
    uint256 userStake = balances[msg.sender][_token];
    if (userStake == 0) {
        revert NothingToClaim();
    }
    if (_token == ETH) {
        claimedAmount = userStake.mulDiv(totalLpETH, totalSupply);
        balances[msg.sender][_token] = 0;
        lpETH.safeTransfer(_receiver, claimedAmount);
    } else {
        uint256 userClaim = (userStake * _percentage) / 100;
        _validateData(_token, userClaim, _exchange, _data);
        balances[msg.sender][_token] = userStake - userClaim;

        // At this point there should not be any ETH in the contract
        // Swap token to ETH
        _fillQuote(IERC20(_token), userClaim, _data);

        // Convert swapped ETH to lpETH (1 to 1 conversion)
        claimedAmount = address(this).balance;
        lpETH.deposit{value: claimedAmount}(_receiver);
    }
    emit Claimed(msg.sender, _token, claimedAmount);
}
```

### Suggested Change:
```
function _claim(
    address _token,
    address _receiver,
    uint8 _percentage,
    Exchange _exchange,
    bytes calldata _data
) internal returns (uint256 claimedAmount) {
    uint256 userStake = balances[msg.sender][_token];
    if (userStake == 0) {
        revert NothingToClaim();
    }

    uint256 userClaim = (userStake * _percentage) / 100;
    balances[msg.sender][_token] = userStake - userClaim;

    if (_token == ETH) {
        claimedAmount = userStake.mulDiv(totalLpETH, totalSupply);
        lpETH.safeTransfer(_receiver, claimedAmount);
    } else {
        _validateData(_token, userClaim, _exchange, _data);
        
        // Swap token to ETH
        uint256 initialETHBalance = address(this).balance;
        _fillQuote(IERC20(_token), userClaim, _data);
        uint256 newETHBalance = address(this).balance;
        uint256 ethReceived = newETHBalance - initialETHBalance;

        // Convert received ETH to lpETH
        claimedAmount = ethReceived;
        lpETH.deposit{value: ethReceived}(_receiver);
    }

    emit Claimed(msg.sender, _token, claimedAmount);
}
```
**Changes Made:**

- Added tracking of ETH balance before and after the swap to ensure only swapped ETH is used.

- Updated the balance adjustment logic to reflect changes before making external calls.

### Optimization Details:

- **Efficiency Enhancement:** The original implementation first swaps the token to ETH and then calculates the claimed amount using the entire ETH balance of the contract. This is inefficient and risky as it assumes all ETH in the contract came from the swap.

- **Risk Mitigation:** The suggested change refactors the function to track the exact amount of ETH received from the swap by measuring the ETH balance before and after the swap. This ensures accuracy in determining the claimed amount, mitigating the risk of overestimation or misallocation of funds.