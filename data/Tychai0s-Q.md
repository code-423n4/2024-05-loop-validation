## Audit Report for PrelaunchPoints.sol

### Title
L-01 Improper Handling of `_percentage` in `_claim` Function Leading to Potential Underflow and Zero Token Operations

### Impact
This issue can lead to an underflow error when `_percentage` exceeds 100, causing a revert due to attempting to subtract a larger number from a smaller one. Additionally, if `_percentage` is 0, the function proceeds to execute token swaps and deposits with a zero value, which is inefficient and could lead to unnecessary gas expenditure without any effective operation.

### Proof of Concept
In the `_claim` function within `PrelaunchPoints.sol`, the `_percentage` parameter is used to calculate the amount of tokens a user wishes to claim based on their stake. The function does not validate whether `_percentage` is within a logical range (0 < `_percentage` <= 100). This oversight can lead to two main issues:
1. **Underflow Error**: If `_percentage` is greater than 100, the calculation `userStake * _percentage / 100` results in `userClaim` being greater than `userStake`. This leads to an underflow in the line `balances[msg.sender][_token] = userStake - userClaim;` when attempting to update the user's balance.
2. **Zero Token Operations**: If `_percentage` is 0, the calculation results in a `userClaim` of 0. The function still proceeds to validate and attempt token swaps and deposits with this zero value, leading to unnecessary gas usage and potential confusion or errors in the swap functions.

### Tools Used
- Manual Review

### Recommended Mitigation Steps
1. **Input Validation**: Implement a check to ensure `_percentage` is within the range 1 to 100. This can be done by adding a require statement at the beginning of the function:
   ```solidity
   require(_percentage > 0 && _percentage <= 100, "Percentage must be between 1 and 100");
   ```
2. **Error Handling**: Improve error messages to clearly indicate the nature of the error, aiding in debugging and user comprehension. Specifically, handle the case where `_percentage` is 0 to prevent unnecessary operations.

### Issue Type
**Invalid Validation**

### Title
Non-Standard Token Behavior Handling in [_fillQuote](https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L495) Function

### Impact
The function [_fillQuote](https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L491) may fail or behave unpredictably when interacting with non-standard ERC20 tokens that return unexpected data from the approve function. This could lead to transaction failures.

### Proof of Concept
In `PrelaunchPoints.sol:497`, the function [_fillQuote](https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L491) method of an ERC20 token:
```solidity
require(_sellToken.approve(exchangeProxy, _amount));
```
This implementation assumes that the `approve` function will always return a boolean value, which is not guaranteed for non-standard tokens. Non-standard tokens might return irrelevant or malformed data, causing the `require` statement to fail or behave incorrectly.

### Tools Used
- Manual Review

### Recommended Mitigation Steps
1. **Integration of SafeERC20**: Replace the direct use of `_sellToken.approve` with `SafeERC20.safeApprove` from the OpenZeppelin library, which is designed to handle ERC20 tokens safely, including those that do not conform to the standard return values.
2. **Consistent Use of SafeERC20**: Ensure that all ERC20 token interactions within the contract utilize SafeERC20 methods to prevent similar issues in other parts of the contract.

### Issue Type
- **ERC20**