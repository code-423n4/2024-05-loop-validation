| Risk    | Issues Details                                                                                          | Number        |
|---------|---------------------------------------------------------------------------------------------------------|---------------|
| [01]    | Missing checks for `address(0)` could loss the user funds.                                                                        |      1        |



## Missing checks for `address(0)` could loss the user funds.
#### Description
In the `lockETHFor` and `lockFor` external functions if the User accidentally sent the funds to `address(0)` there is no check for this scenario.
#### Lines of code
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L133
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L157
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L172
#### Recommended Mitigation Steps
Add this error and check in the `_processLock` internal function.
```solidity
    error cannotLockForAddress0();

    if (_receiver == address(0)) {
        revert cannotLockForAddress0();
    }
```