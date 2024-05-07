# [N-01] There is no need for an unchecked statement in a constructor

variable "i" which is used in the for loop is initialized with 0 and is only incremented by one additionally there is no other operation that is happening on "i".

 "i" is uint256 which means it can store a number up to 11579208923731619542357098500868790785326998466564056403945758400791 or 2^256-1 and is likely impossible(because of gas price) to reach this number by starting from 0 and incrementing it by 1.

## link to code

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L107C1-L111C14