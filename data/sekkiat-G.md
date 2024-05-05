## Ttile
Should use UINT256 instead of UINT8

## Impact
Increase Gas 

## Affected Code
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L211
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L226

## Useful Link
https://ethereum.stackexchange.com/questions/3067/why-does-uint8-cost-more-gas-than-uint256

## Tools Used
Manual Review

## Recommended Mitigation Steps
Use UINT256