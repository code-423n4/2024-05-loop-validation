### **[[ 1 ]]** 
In order to avoid losing control of the contract (renouncing ownership), a two-step ownership transfer should be used like `OZ OwnableTwoSteps` instead of `setOwner`. That would allow to fix 2 flaws in the current `setOwner` function:
1) Ensure the new owner is not address(0)
2) Ensure the new owner is a valid account and actually controlled by someone (EOA or smart contract)

Losing control of the contract would means all the functions behind the `onlyAuthorized` modifier would become inaccessible, which can bring all sort of side effects.

```solidity
    function setOwner(address _owner) external onlyAuthorized {
        owner = _owner;

        emit OwnerUpdated(_owner);
    }
```
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L336-L340


### **[[ 2 ]]** 
`lockFor` and `lockETHFor` have no validation for the `_for` parameter which is used as is inside `_processLock`. Consequently, this means the same flaws as setOwner are being exposed, and if such scenario happen would cause `user fund loss` as will not revert. I would advice the protocol to be a little bit more strick on this one. 

```diff
    function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
        internal
        onlyBeforeDate(loopActivation)
    {
        if (_amount == 0) {
            revert CannotLockZero();
        }
+       if (_receiver == address(0)) {
+           revert InvalidReceiver();
+       }
+	
        if (_token == ETH) {
            totalSupply = totalSupply + _amount;
            balances[_receiver][ETH] += _amount;
        } else {
            if (!isTokenAllowed[_token]) {
                revert TokenNotAllowed();
            }
            IERC20(_token).safeTransferFrom(msg.sender, address(this), _amount);

            if (_token == address(WETH)) {
                WETH.withdraw(_amount);
                totalSupply = totalSupply + _amount;
                balances[_receiver][ETH] += _amount;
            } else {
                balances[_receiver][_token] += _amount;
            }
        }

        emit Locked(_receiver, _amount, _token, _referral);
    }
```

**PoC**
Add the following test in `PrelaunchPointsTest.t.sol` which confirm locking ETH for address(0) doesn't revert, so funds are lost.
```solidity
    function testLockETHForToAddressZero(uint256 lockAmount) public {
        vm.assume(lockAmount > 0);
        address recipient = address(0);

        vm.deal(address(this), lockAmount);
        prelaunchPoints.lockETHFor{value: lockAmount}(recipient, referral);

        assertEq(prelaunchPoints.balances(recipient, ETH), lockAmount);
        assertEq(prelaunchPoints.totalSupply(), lockAmount);
    }
```
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L133-L135
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L157-L162


### **[[ 3 ]]**
Would it make sense to ensure the `totalLpETH >= totalBalance` when calling convertAllETH, which ensure 1:1 invariant hold?
```diff
    function convertAllETH() external onlyAuthorized onlyBeforeDate(startClaimDate) {
        if (block.timestamp - loopActivation <= TIMELOCK) { // @audit (L) will revert when done too early?
            revert LoopNotActivated();
        }

        // deposits all the ETH to lpETH contract. Receives lpETH back
        uint256 totalBalance = address(this).balance;
        lpETH.deposit{value: totalBalance}(address(this));

        totalLpETH = lpETH.balanceOf(address(this));

+       if (totalLpETH < totalBalance) {
+	     revert LoopNotActivated();
+       }

        // Claims of lpETH can start immediately after conversion.
        startClaimDate = uint32(block.timestamp);

        emit Converted(totalBalance, totalLpETH);
    }
```
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L315-L330