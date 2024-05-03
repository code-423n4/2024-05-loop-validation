User that deposited WETH, can't withdraw in the same currency

**Description:** 
User that locked WETH into the contract it's not able to withdraw WETH due 
to conversion to ETH.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L189

**Impact:** 
User are not able to withdraw their funds using the same currency that 
they deposited. 

**Recommended Mitigation:** 
Consider documenting it or consider moving the exchange of WETH to ETH to convertAllETH!