						
## Summary
|Impact|Title|
|:-:|:----|
|[LOW-01](#DoS-if-fail-on-eth-receive)|DoS if fail on eth receive|
|[NC-01](#allow-user-to-withdraw-in-case-of-loop-is-not-set)|Allow User to withdraw in case of loop is not set|
|[NC-02](#use-two-step-ownership)|User Two step Ownership|
|[NC-03](#follow-cei-pattern)|Follow CEI pattern|

--- 

## DoS if fail on eth receive
The user could not be able to with their `ETH` if the receiver address is contract and does not have `receive() external payable` or `fallback` function. the best approach would be to convert the ETH into WETH and transfer to receiver.
The POC test taken from test file.
### POC
```solidity
function testWithdrawETHFailNotReceive(uint256 lockAmount) public {
        vm.assume(lockAmount > 0);
        vm.deal(address(lpETHVault), lockAmount);
        vm.prank(address(lpETHVault)); // Contract without receive
        prelaunchPoints.lockETH{value: lockAmount}(referral);

        prelaunchPoints.setLoopAddresses(address(lpETH), address(lpETHVault));
        vm.warp(prelaunchPoints.loopActivation() + 1);

        vm.prank(address(lpETHVault));
        vm.expectRevert(PrelaunchPoints.FailedToSendEther.selector);
        prelaunchPoints.withdraw(ETH);
    }
```
Run it with `forge test --mt testWithdrawETHFailNotReceive`
### Recommendation
On Fail convert the ETH into WETH and transfer.
```solidity
if (!sent) {
    uint256 balanceBefore = address(this).balance;
WETH.deposit{value:lockedAmount}();
WETH.transfer(msg.sender , address(this).balance-balanceBefore);
    }
```
---

## Allow User to withdraw in case of loop is not set
The Protocol set the `loopActivation` to 120 days in future at the time of `PrelaunchPoints` deployment. It got updated if the owner of `PrelaunchPoints` set the loop addresses. there is a case if the owner of `PrelaunchPoints` does not set the loop addresses then the user must wait for 120 days if `emergencyMode` is false. 
The issue here is that the user would be allowed to withdraw their assets if claimDate is not started it.
Change this code :
```solidity
function withdraw(address _token) external {
        if (!emergencyMode) {
            if (block.timestamp <= loopActivation) {
                revert CurrentlyNotPossible();
            }
            if (block.timestamp >= startClaimDate) {
                revert NoLongerPossible();
            }
        }
        ...

```
To this:
```solidity
function withdraw(address _token) external {
        if (!emergencyMode) {
            if (block.timestamp >= startClaimDate) {
                revert NoLongerPossible();
            }
        }
```
---

## Use Two step Ownership
The `PrelaunchPoints` use single step to set newOwnership which is not recommended. The standard is to used two step ownership transfer . 
add pendingOwner variable which will store the newOwner address temporally when newOwner accept/claim ownership then transfer the ownership.

---
## Follow CEI pattern
The function `convertAllETH` does not follow CEI pattern Because it uses `onlyBeforeDate(startClaimDate)` modifier to call `convertAllETH` function moe then once. 
`convertAllETH` function:
```solidity
function convertAllETH() external onlyAuthorized onlyBeforeDate(startClaimDate) {
        if (block.timestamp - loopActivation <= TIMELOCK) {
            revert LoopNotActivated();
        }

        // deposits all the ETH to lpETH contract. Receives lpETH back
        uint256 totalBalance = address(this).balance;
    I@>    lpETH.deposit{value: totalBalance}(address(this));

    E@>    totalLpETH = lpETH.balanceOf(address(this)); // @audit : overide totalLpETH it would be appended, balanceOf valuerable to donation attack

        // Claims of lpETH can start immediately after conversion.
    E@>    startClaimDate = uint32(block.timestamp);

        emit Converted(totalBalance, totalLpETH);
    }

```
also change this `totalLpETH=` to `totalLpETH+=`.
this function could be changed to following :

```solidity
function convertAllETH() external onlyAuthorized onlyBeforeDate(startClaimDate) {
        if (block.timestamp - loopActivation <= TIMELOCK) {
            revert LoopNotActivated();
        }
        startClaimDate = uint32(block.timestamp);

        // deposits all the ETH to lpETH contract. Receives lpETH back
        uint256 totalBalance = address(this).balance;
       lpETH.deposit{value: totalBalance}(address(this));

        totalLpETH += lpETH.balanceOf(address(this)); // @audit : overide totalLpETH it would be appended, balanceOf valuerable to donation attack

        // Claims of lpETH can start immediately after conversion.

        emit Converted(totalBalance, totalLpETH);
    }

```