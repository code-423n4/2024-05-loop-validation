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

Proof of code:

```javascript
    function test_cannotRecoverTokenIfWasAllowedMeanwhile() public {
        address user = makeAddr("user");
        uint256 balance = 2e18;

        // token that is not allowed yet
        LRToken lrtB = new LRToken();
        lrtB.mint(address(this), INITIAL_SUPPLY);
        lrtB.transfer(user, balance);

        vm.startPrank(user);
        lrtB.approve(address(prelaunchPoints), balance);
        lrtB.transfer(address(prelaunchPoints), balance);
        vm.stopPrank();

        // when it is allowed, it is possible to recover
        prelaunchPoints.recoverERC20(address(lrtB), balance / 2);

        // when it gets allowed, it becomes impossible to recover
        prelaunchPoints.allowToken(address(lrtB));
        vm.expectRevert(PrelaunchPoints.NotValidToken.selector);
        prelaunchPoints.recoverERC20(address(lrtB), balance / 2);
    }
```

## Recommended Mitigation Steps

Implement a check so that a token cannot be allowed if the contract has any balance in that token - i.e. before allowing `token X`, the protocol owner will need to recover any funds of `token X`  from the protocol.

```diff
    function allowToken(address _token) external onlyAuthorized {
+        if (_token.balanceOf(address(this)) != 0) {
+            recoverERC20(_token, _token.balanceOf(address(this)));
+        }
        isTokenAllowed[_token] = true;
    }

-    function recoverERC20(address tokenAddress, uint256 tokenAmount) external onlyAuthorized{...}
+    function recoverERC20(address tokenAddress, uint256 tokenAmount) public onlyAuthorized{...}
```

