[L-1] `PrelaunchPoint::allowToken` - If the wrong token is allowed it cannot be undone

**Description:**

In the `PrelaunchPoint` the function `allowToken` allows the owner to set an allowed token. However if for any reason the owner sets the wrong token, it can not be undone.

**Impact:**

1. This can potentially lead to a token being allowed that is not intended to be.

**Tools Used**

Manual Review

**Recommended Mitigation:**

In the function `allowToken` pass the token and a bool as a parameter.

```diff
- function allowToken(address _token) external onlyAuthorized {
+ function allowToken(address _token, bool isAllowed) external onlyAuthorized { 
         isTokenAllowed[_token] = isAllowed;
     }
```