##

## [L-1] Risks of disabling locking ETHs after contract deployment 

### Impact

The setLoopAddresses function, which is guarded by the onlyAuthorized modifier, allows an authorized account (typically the contract owner) to set the addresses for the lpETH and lpETHVault contracts. Significantly, this function also resets the loopActivation timestamp to the current block timestamp (block.timestamp). 

The contract relies on the loopActivation timestamp to control the locking of ETH. The original intent appears to be setting a future date (120 days post-deployment) for when certain actions, like token locking, can begin. However, calling setLoopAddresses resets this timestamp to the current time

This functionality allows the authorized user to arbitrarily disabling the ``locking of ETH immediately`` after contract deployment.

```solidity
FILE: 2024-05-loop/src/PrelaunchPoints.sol

172: function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
        internal
        onlyBeforeDate(loopActivation)


348: function setLoopAddresses(address _loopAddress, address _vaultAddress)
        external
        onlyAuthorized
        onlyBeforeDate(loopActivation)
    {
        lpETH = ILpETH(_loopAddress);
        lpETHVault = ILpETHVault(_vaultAddress);
        loopActivation = uint32(block.timestamp);

        emit LoopAddressesUpdated(_loopAddress, _vaultAddress);
    }

525: modifier onlyBeforeDate(uint256 limitDate) {
        if (block.timestamp >= limitDate) {
            revert NoLongerPossible();
        }
        _;
    }

```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L342-L358

### POC 

- Contract deployed the with ``loopActivation = uint32(block.timestamp + 120 days)`` which means its possible to lock ETH and ERC20 Tokens till ``block.timestamp + 120 days`` 

- In normal operations ``setLoopAddresses`` called before 120 days ends once contract deployment.

- But if ``onlyAuthorized`` decided to set immediately after contract deployment technically the value of loopActivation set to ``loopActivation = uint32(block.timestamp) ``

- After loopActivation value set the onlyBeforeDate() modifiers reverts because this check become true if (block.timestamp >= limitDate) .

So technically not possible to lock tokens after  ``loopActivation `` set. Reverts all locking process

### Recommended Mitigation
Exactly define when ``setLoopAddresses()`` called 

##

## [L-2] Locking Mechanism Bypass via Direct ETH Transfers

The contract has a receive function that allows it to receive Ether directly, but this does not interact with any of the contract's state-managing functions such as lockETH or lockETHFor. This opens up a pathway where Ether could be sent to the contract without being accounted for in the totalSupply or balances, breaking the contract's internal accounting logic.

Untracked Ether in the contract could lead to discrepancies between the recorded and actual balances, affecting the integrity of the contract and potentially enabling theft or loss of funds.

```solidity
FILE: 2024-05-loop/src/PrelaunchPoints.sol

392: receive() external payable {}

```

##

## [L-3] ``if (block.timestamp <= loopActivation) { `` this check is not implemented as per docs 

The docs states that withdrawal only allowed after 7 Days once addresses are set

```
Once these addresses are set, all deposits are paused and users have 7 days to withdraw their tokens in case they changed their mind, or they detected a malicious contract being set. On withdrawal, users loose all their points.

```

```solidity
FILE: 2024-05-loop/src/PrelaunchPoints.sol

276: if (block.timestamp <= loopActivation) {

```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L276

If the Eth is not converted in time after 7 days technically this will allow 
withdrawal even after 7 days completed.

### Recommended Mitigation

The check should be 

```solidity

if (block.timestamp - loopActivation >= TIMELOCK) {

```

##

## [L-4] State Update After External Calls vulnerable to Reentrancy attacks

 In the _processLock function, the contract interacts with external tokens (via safeTransferFrom) after updating the internal state. This generally contradicts the recommended practice in Solidity to prevent reentrancy attacks, which advises updating the contract's state before making external calls.

Relying on this without strict adherence to the checks-effects-interactions pattern could expose the contract to risks if the external called contracts behave unexpectedly.

```solidity
FILE: 2024-05-loop/src/PrelaunchPoints.sol

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

```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L182-L194

### Recommended Mitigation
Implement the Check-Effects-Intraction patteren (CEI)

##

## 

## [L-5] Uncertainty in calling ``setLoopAddresses() `` function

The uncertainty surrounding when the setLoopAddresses() function can be executed—whether immediately after deployment or just before the 120-day threshold—is indeed a significant point of concern. 

The function can technically be called at any moment before the 120-day mark, which adds an element of unpredictability. Users and stakeholders might not be aware of when these addresses could change, affecting their strategies and interactions with the contract.

```solidity
FILE: 2024-05-loop/src/PrelaunchPoints.sol

/**
     * @notice Sets the lpETH contract address
     * @param _loopAddress address of the lpETH contract
     * @dev Can only be set once before 120 days have passed from deployment.
     *      After that users can only withdraw ETH.
     */
    function setLoopAddresses(address _loopAddress, address _vaultAddress)
        external
        onlyAuthorized
        onlyBeforeDate(loopActivation)
    {
        lpETH = ILpETH(_loopAddress);
        lpETHVault = ILpETHVault(_vaultAddress);
        loopActivation = uint32(block.timestamp);

        emit LoopAddressesUpdated(_loopAddress, _vaultAddress);
    }


```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L342-L358

##

## [L-6] Hardcoded Time Delay

Having a hardcoded time delay like uint32 public constant TIMELOCK = 7 days; in  code can be limiting.

If you need to adjust the time delay in the future, you'll have to recompile and redeploy your contract, which can be a cumbersome process.

```solidity
FILE: 2024-05-loop/src/PrelaunchPoints.sol

47: uint32 public constant TIMELOCK = 7 days;

```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L47

##

## [L-7] Missing the token uniqueness check in ``isTokenAllowed`` mapping

The code doesn't explicitly check for duplicate entries in the ``_allowedTokens`` array. This could allow to add the same token address multiple times, and updates the same value multiple times

```solidity
FILE: 2024-05-loop/src/PrelaunchPoints.sol

 // Allow intital list of tokens
        uint256 length = _allowedTokens.length;
        for (uint256 i = 0; i < length;) {
            isTokenAllowed[_allowedTokens[i]] = true;
            unchecked {
                i++;
            }
        }

```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L105-L112

##

## [L-8] Inconsistent Data Types in Date Modifiers (``onlyBeforeDate``, ``onlyAfterDate``)

The loopActivation and startClaimDate variable is declared as uint32, which can only represent values up to around 4.29 billion seconds (roughly 42.9 years). However, the modifiers onlyBeforeDate and onlyAfterDate are defined to take uint256 arguments, which can hold much larger values. This mismatch could lead to unexpected behavior and potentially creates the confusion

```solidity
FILE: 2024-05-loop/src/PrelaunchPoints.sol


    uint32 public loopActivation;
    uint32 public startClaimDate;

172: function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
        internal
        onlyBeforeDate(loopActivation)

226: function claimAndStake(address _token, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        external
        onlyAfterDate(startClaimDate)

  modifier onlyAfterDate(uint256 limitDate) {
        if (block.timestamp <= limitDate) {
            revert CurrentlyNotPossible();
        }
        _;
    }

    modifier onlyBeforeDate(uint256 limitDate) {
        if (block.timestamp >= limitDate) {
            revert NoLongerPossible();
        }
        _;
    }

``` 
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L518-L530

##

## [L-9] 

















## [L-1] Missing contract-existence checks before low-level calls

Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

```solidity
FILE: 2024-05-loop/src/PrelaunchPoints.sol   

296: (bool sent,) = msg.sender.call{value: lockedAmount}("");

```
https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L296C12-L296C69
