In the `lockETHFor` and `lockFor` functions, there is no validation to prevent users from locking funds for the address(0). Additionally, the `_processLock` function, which underpins these lock functions, lacks such a check as well. This oversight means that users can inadvertently lock funds for the null address, resulting in irreversible loss of funds.

LockFor
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L157-L162
lockETHFor
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L133-L135
_processLock
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L172-L198