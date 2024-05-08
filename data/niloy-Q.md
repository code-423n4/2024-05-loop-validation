# Low Severity Issues:

## Issue 1: Inadequate Input Validation for Percentage Paramet

### Description:
The `claim` and `claimAndStake` functions allow users to specify a `_percentage` parameter to determine the proportion of tokens to withdraw. However, the contract lacks proper validation to ensure that the provided `_percentage` value falls within a valid range, such as between 0 and 100. This oversight could lead to unexpected behavior or errors in the token claiming process.

### Impact:
Passing invalid or out-of-range values for `_percentage` could result in incorrect token calculations and withdrawals.

### Proof of Concept:
```solidity
function claim(address _token, uint8 _percentage, Exchange _exchange, bytes calldata _data)
    external
    onlyAfterDate(startClaimDate)
{
    _claim(_token, msg.sender, _percentage, _exchange, _data);
}
```

### Recommended Mitigation Steps:
```diff
function claim(address _token, uint8 _percentage, Exchange _exchange, bytes calldata _data)
    external
    onlyAfterDate(startClaimDate)
{
+   require(_percentage >= 0 && _percentage <= 100, "Invalid percentage value");
    _claim(_token, msg.sender, _percentage, _exchange, _data);
}
```

## Issue 2: Absence of Events for Critical State Changes

### Description:
The contract lacks event emission for certain critical state changes, such as updating the `emergencyMode` or adding allowed tokens via the `allowToken` function. Emitting events is crucial for maintaining transparency and facilitating event-based monitoring and integration with external systems.

### Impact:
The lack of events hinders the ability to monitor and react to critical state changes in the contract.

### Proof of Concept:
```solidity
function setEmergencyMode(bool _mode) external onlyAuthorized {
    emergencyMode = _mode;
}

function allowToken(address _token) external onlyAuthorized {
    isTokenAllowed[_token] = true;
}
```

### Recommended Mitigation Steps:
```diff
function setEmergencyMode(bool _mode) external onlyAuthorized {
    emergencyMode = _mode;
+   emit EmergencyModeUpdated(_mode);
}

function allowToken(address _token) external onlyAuthorized {
    isTokenAllowed[_token] = true;
+   emit TokenAllowed(_token);
}
```

## Issue 3: Inconsistent Error Handling Approach

### Description:
The contract employs a mix of custom errors and revert statements with string error messages for error handling. This inconsistency can make the code harder to read, understand, and maintain.

### Impact:
Inconsistent error handling can lead to confusion and make the code harder to maintain.

### Proof of Concept:
```solidity
error InvalidToken();
// ...
revert("FailedToSendEther");
```

### Recommended Mitigation Steps:
```diff
error InvalidToken();
+ error FailedToSendEther();
// ...
- revert("FailedToSendEther");
+ revert FailedToSendEther();
```

## Issue 4: Unbounded Token Lock Duration

### Description:
The contract allows users to lock their tokens for an arbitrary duration without imposing any upper limit. While this flexibility may be desirable in certain scenarios, it can also lead to tokens being locked for an excessively long period, potentially contradicting the intended token economics or liquidity goals of the project.

### Impact:
Users could inadvertently lock their tokens for an unreasonably long time, affecting token liquidity and circulation.

### Proof of Concept:
```solidity
function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
    internal
    onlyBeforeDate(loopActivation)
{
    // ...
}
```

### Recommended Mitigation Steps:
```diff
+ uint256 public constant MAX_LOCK_DURATION = 365 days; // Example: 1 year

function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
    internal
    onlyBeforeDate(loopActivation)
{
+   require(lockDuration <= MAX_LOCK_DURATION, "Exceeds maximum lock duration");
    // ...
}
```

## Issue 5: Inadequate Checks for Token Transfer Success

### Description:
The contract uses `safeTransfer` and `safeTransferFrom` functions from the SafeERC20 library to transfer tokens. However, it does not consistently check the return value of these functions to ensure the success of the token transfers.

### Impact:
Unsuccessful token transfers may go unnoticed, resulting in discrepancies in token balances and contract state.

### Proof of Concept:
```solidity
IERC20(_token).safeTransfer(msg.sender, lockedAmount);
```

### Recommended Mitigation Steps:
```diff
- IERC20(_token).safeTransfer(msg.sender, lockedAmount);
+ bool success = IERC20(_token).safeTransfer(msg.sender, lockedAmount);
+ require(success, "Token transfer failed");
```

# Non-Critical Issues:

## Issue 1: Inconsistent Function Visibility Specifiers

### Proof of Concept:
```solidity
function lock(address _token, uint256 _amount, bytes32 _referral) external {
    // ...
}

function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
    internal
    onlyBeforeDate(loopActivation)
{
    // ...
}
```

### Recommended Mitigation Steps:
```diff
- function lock(address _token, uint256 _amount, bytes32 _referral) external {
+ function lock(address _token, uint256 _amount, bytes32 _referral) public {
    // ...
}

function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
    internal
    onlyBeforeDate(loopActivation)
{
    // ...
}
```

## Issue 2: Redundant Payable Keyword for Exchange Proxy

### Proof of Concept:
```solidity
function _fillQuote(IERC20 _sellToken, uint256 _amount, bytes calldata _swapCallData) internal {
    // ...
    (bool success,) = payable(exchangeProxy).call{value: 0}(_swapCallData);
    // ...
}
```

### Recommended Mitigation Steps:
```diff
function _fillQuote(IERC20 _sellToken, uint256 _amount, bytes calldata _swapCallData) internal {
    // ...
-   (bool success,) = payable(exchangeProxy).call{value: 0}(_swapCallData);
+   (bool success,) = exchangeProxy.call{value: 0}(_swapCallData);
    // ...
}
```

## Issue 3: Duplicate Token Type Check in Withdraw Function


### Proof of Concept:
```solidity
function withdraw(address _token) external {
    // ...
    if (_token == ETH) {
        if (block.timestamp >= startClaimDate){
            revert UseClaimInstead();
        }
        // ...
    }
    // ...
}
```

### Recommended Mitigation Steps:
```diff
function withdraw(address _token) external {
    // ...
    if (_token == ETH) {
-       if (block.timestamp >= startClaimDate){
-           revert UseClaimInstead();
-       }
        // ...
    }
    // ...
}
```

## Issue 4: Unnecessary Zero Amount Check in Process Lock Function


### Proof of Concept:
```solidity
function _processLock(address _token, uint256 _amount, address _receiver, bytes

32 _referral)
    internal
    onlyBeforeDate(loopActivation)
{
    if (_amount == 0) {
        revert CannotLockZero();
    }
    // ...
}
```

### Recommended Mitigation Steps:
```diff
function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
    internal
    onlyBeforeDate(loopActivation)
{
-   if (_amount == 0) {
-       revert CannotLockZero();
-   }
    // ...
}
```

## Issue 5: Inconsistent Error Message Formatting

### Proof of Concept:
```solidity
error InvalidToken();
// ...
revert("FailedToSendEther");
```

### Recommended Mitigation Steps:
```diff
error InvalidToken();
// ...
- revert("FailedToSendEther");
+ revert("Failed to send Ether");
```
