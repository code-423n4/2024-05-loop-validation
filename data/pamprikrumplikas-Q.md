# [01] Unneccessary centralization via `onlyAuthorized` modifier in `PrelaunchPoints::convertAllETH`

`PrelaunchPoints::convertAllETH` is protected by the `onlyAuthorized` modifier, restricting access to this function to the protocol owner:

```javascript
    function convertAllETH() external onlyAuthorized onlyBeforeDate(startClaimDate) {...}
```

## Impact
The restriction deepens centralization unneccesarily, exposing users to associated risks, and making the flow of events less predictable to the users.

Users should be able to start claiming LpETH right after the `TIMELOCK` period passes following the succesfull execution of a call to `PrelaunchPoints::setLoopAddresses`. 

Claims are enabled upon a successful call to `PrelaunchPoints::convertAllETH`, but as long as it is protected by the `onlyAuthorized` modifier, the authorized account can chose to delay to call this function or can decide to never call it at all, introducing uncertainty and risk for the users.

## Recommended Mitigation Steps
Remove the `onlyAuthorized` modifier in `PrelaunchPoints::convertAllETH`.


