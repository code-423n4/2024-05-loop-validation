## QA-01: Users who lock in WETH cannot withdraw if they input WETH as token 
If `WETH` is locked , it is converted to ETH and users ETH balance increases. Users have to withdraw in ETH which is an intended behavior . However, the issue is if a user calls `withdraw` with `WETH` as token , it'll revert as users balance is not tracked in WETH . 

#recommendation:
Users should be able to withdraw their funds with `WETH` as argument . Adding this line to `withdraw ` will fix this :
```diff
function withdraw(address _token) external {//@todo users who deposit in weth will withdraw in eth ! 
       //.....
+       if (_token == WETH) _token =ETH ;
        uint256 lockedAmount = balances[msg.sender][_token];
        balances[msg.sender][_token] = 0;

        if (lockedAmount == 0) {
            revert CannotWithdrawZero();
        }
        if (_token == ETH) {
            if (block.timestamp >= startClaimDate){
                revert UseClaimInstead();
            }
```
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L274
## QA-02:Typos in comments 
 `lockFor` function is supposed to lock token for a receiver . But in-line comments says `ETH` . 
```solidity
    * @param _for  address for which ETH is locked
```
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L154C47-L154C48
#recommendation:
`ETH` should be changed to `Token ` . 


## QA-03:Typos !  
Spelling Mistake in in-line comments 
Change `mistakingly ` to `mistakenly ` . 
```solidity
  * @dev Allows the owner to recover other ERC20s mistakingly sent to this contract
```
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L377

