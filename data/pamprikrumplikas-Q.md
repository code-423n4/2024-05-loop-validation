# [01] `onlyAuthorized` modifier introduces unnecessary centralization via  in `PrelaunchPoints::convertAllETH`

`PrelaunchPoints::convertAllETH` is protected by the `onlyAuthorized` modifier, restricting access to this function to the protocol owner:

```javascript
    function convertAllETH() external onlyAuthorized onlyBeforeDate(startClaimDate) {...}
```

## Impact
The restriction deepens centralization unnecessarily, exposing users to associated risks, and making the flow of events less predictable to the users.

Users should be able to start claiming LpETH right after the `TIMELOCK` period passes following the succesfull execution of a call to `PrelaunchPoints::setLoopAddresses`. 

Claims are enabled upon a successful call to `PrelaunchPoints::convertAllETH`, but as long as it is protected by the `onlyAuthorized` modifier, the authorized account can chose to delay to call this function or can decide to never call it at all, introducing uncertainty and risk for the users.

## Recommended Mitigation Steps
Remove the `onlyAuthorized` modifier in `PrelaunchPoints::convertAllETH`.


# [02] Not allowed tokens cannot be recovered if they get allowed after having been sent to the contract

`PrelaunchPoints::recoverERC20` is designed to enable the protocol owner to recover any not allowed tokens that users mistakeanly send to the protocol. However, `PrelaunchPoints::allowToken` does not check whether the protocol, by user mistake, has previously received any funds in the token to be allowed.

## Impact

ERC20 funds are lost in the following scenario:

1. ERC20 token `tokenA` is not allowed by the protocol.
2. `UserA` mistakeanly transfers some `tokenA` to the protocol.
3. Protocol owner allows `tokenA` in the protocol, and does not check whether the contract already has any balance of the token.
4. When trying to recover the now allowed `tokenA` for `UserA`, `PrelaunchPoints::recoverERC20` will revert with `NotValidToken`.

## Recommended Mitigation Steps

Implement a check so that a token cannot be allowed if the contract has any balance in that token - i.e. before allowing `token X`, the protocol owner will need to recover any funds of `token X`  from the protocol.

