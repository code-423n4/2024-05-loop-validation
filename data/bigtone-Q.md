# The `_percentage` doesn't support decimals.

## Impact
The claim percentage parameter currently restricts values to integers from 0 to 100.
To enhance the functionality, I suggest expanding it to support decimals.
It will provide users with more flexibility percentage.

## Proof of Concept
The `_percentage` doesn't support decimals like 5.5%.
```solidity PrelaunchPoints.sol#L253
=>    uint256 userClaim = userStake * _percentage / 100; // @audit doesn't support decimals
    _validateData(_token, userClaim, _exchange, _data);
    balances[msg.sender][_token] = userStake - userClaim;
```


## Tools Used
Manual review

## Recommended Mitigation Steps
To enhance the functionality, I suggest expanding it to support decimals.
This would allow for a wider range of possible claims, with a format like 1e6 representing 100%.

```solidity PrelaunchPoints.sol#L253
=>    uint256 userClaim = userStake * _percentage / 1e6; // @audit doesn't support decimals
    _validateData(_token, userClaim, _exchange, _data);
    balances[msg.sender][_token] = userStake - userClaim;
```