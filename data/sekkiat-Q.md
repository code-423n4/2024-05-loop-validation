## Title
Token in isTokenAllowed list not able to remove

## Affected Code
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L365

## Impact
The owner mistakenly added an incorrect token that cannot be removed.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Add a function that can remove the token in isTokenAllowed list.


