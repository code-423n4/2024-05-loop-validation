# [C-01] Owner can skip calling `convertAllETH`

## Summary

Once users have locked their desired amount of tokens, and `setLoopAddresses` has been called, to set the needed addresses, the last step remaining is to wait out the 7 day period in which users can withdraw their locked tokens. After that, `convertAllETH` has to be called by the `Owner` for the users to be able to claim their `lpETH`, however there's no guarantee that the Owner would even call the function.

## Impact

If the owner never calls `convertAllETH` after the 7 day time period has passed, all of the users' locked deposits just remain locked.

## Recommended Mitigation Steps

Consider adding an additional period, for example of 2 weeks, which would allow any of the depositors to call `convertAllETH`.