## Low Risk Findings

| Number | issue                                                                 | Instances |
| ------ | --------------------------------------------------------------------- | --------: |
| [L-01] | Low level calls to account with no code will not fail                 |         1 |
| [L-02] | Lack of Check if amount == ETH in Locking Amount                      |         1 |
| [L-03] | Prevent gas griefing attacks that’s possible with custom address.call |         1 |
| [L-04] | Streamlining token approvals                                          |         1 |
| [L-05] | Enforce a minimum claimable amount                                    |         1 |
| [L-06] | Reentrancy in Withdraw Function                                       |         1 |

## [L-01] : Low level calls to account with no code will not fail

Low level calls to account with no code will not fail

Low level calls `(i.e. address.call(...))` to account with no code will silently succeed without reverting or throwing any error. Quoting the reference for the CALL opcode in evm.codes:

Creates a new sub context and execute the code of the given account, then resumes the current one. Note that an account with no code will return success as true.

**_Instances_**

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L296

## [L-02] : Lack of Check if amount == ETH in Locking Amount

The `_processLock` function does not have a check to ensure that the amount of ETH being locked is the same as `msg.value`. This could lead to unexpected behavior or vulnerabilities.

For ETH deposits, `msg.value` must be checked if it is not less than the amount specified.

POC
In the `_processLock` function, add a check to ensure that the amount of ETH being locked is equal to `msg.value`:

```javascript
 function _processLock(address _token, uint256 _amount, address _receiver, bytes32 _referral)
        internal
        onlyBeforeDate(loopActivation)
    {
        if (_amount == 0) {
            revert CannotLockZero();
        }
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

This ensures that amount specified is equal to the amount of ETH sent, preventing any potential issues.

Recommendation:
Implement a check to ensure that the amount of ETH being locked is equal to `msg.value` in the `_processLock` function,

```javascript
if (_token == ETH) {
  Require`msg.value == amount`;
  totalSupply = totalSupply + _amount;
  balances[_receiver][ETH] += _amount;
}
```

## [L-03] : Prevent gas griefing attacks that’s possible with custom address.call

Whenever the returned bytes data is not required, using the .call() function with non TRUSTED addresses opens the transaction to unnecessary gas griefing by return huge bytes data.

Note that this:

```javascript
(bool sent,) = msg.sender.call{value: lockedAmount}("");

            if (!sent) {
                revert FailedToSendEther();
            }

```

So in both cases, the bytes data is returned and copied to memory. Malicious target address can return huge bytes data to cause gas grief to the sender.

Impact:
Malicious target address can gas grief the sender making the sender waste more gas than necessary.

Recommendation:
Short term: When returned data is not required, use a low level call:

```javascript
(bool sent, ) = msg.sender.call{ value: feeCollected }("");
if (!success) {
    assembly {
        revert(0, 0)
    }
}
```

Long Term: Consider using https://github.com/nomad-xyz/ExcessivelySafeCall

**_Instances_**

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L296

## [L-04] : Streamlining token approvals

Streamlining token approvals in

Incorporating forceApprove(), could offer a more streamlined approach to managing ERC20 token allowances; particularly in complex interactions involving asset transfers . By enabling the contract to set precise allowances in a single step, this method could eliminate the need for conditional checks and subsequent resetting of allowances, thus simplifying the logic and potentially enhancing contract efficiency.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L231

```javascript
lpETH.approve(address(lpETHVault), claimedAmount);
```

## [L-05] : Enforce a minimum claimable amount

The current implementation of the `_claim` function in the contract exposes it to a potential denial-of-service (DoS) attack.An attacker could exploit this vulnerability by pushing numerous claims with very small amounts, overwhelming the system and causing it to become unusable. This attack is possible because of gas costs.

Proof of Concept:

Attack Setup:
The attacker, Alice, spends a significant amount on gas to push many claims with very small amounts.
Gas Consumption:
Each claim, incurs gas costs for processing.

Recommended Mitigation Steps:
To mitigate this vulnerability, a minimum claim amount should be enforced in the `_claim` function. By setting a minimum claim amount, the contract can prevent potential DoS attacks.

## [L-06] : Reentrancy in Withdraw Function

The withdraw function in the contract is vulnerable to reentrancy attacks. Reentrancy occurs when an external contract is able to make repeated calls back into the vulnerable contract before the initial call completes. In this case, a malicious contract could exploit this vulnerability to manipulate the contract's state and potentially drain its funds.

Proof of Concept (PoC):

In the withdraw function, there is a risk of reentrancy attacks due to the order of operations. The function updates the total supply before transferring ETH to the user, which can lead to reentrant calls. To address this, we propose implementing a nonReentrant modifier and rearranging the order of operations to update the state before interacting with external contracts. This will help safeguard the contract against reentrancy attacks and ensure the integrity of its state transitions.

```javascript
 /**
     * @dev Called by a staker to withdraw all their ETH or LRT
     * Note Can only be called after the loop address is set and before claiming lpETH,
     * i.e. for at least TIMELOCK. In emergency mode can be called at any time.
     * @param _token      Address of the token to withdraw
     */
    function withdraw(address _token) external { //@audit----info add nonReentrant modifier
        if (!emergencyMode) {
            if (block.timestamp <= loopActivation) {
                revert CurrentlyNotPossible();
            }
            if (block.timestamp >= startClaimDate) {
                revert NoLongerPossible();
            }
        }

        uint256 lockedAmount = balances[msg.sender][_token];
        balances[msg.sender][_token] = 0;

        if (lockedAmount == 0) {
            revert CannotWithdrawZero();
        }
        if (_token == ETH) {
            if (block.timestamp >= startClaimDate){
                revert UseClaimInstead();
            }
            totalSupply = totalSupply - lockedAmount;

            (bool sent,) = msg.sender.call{value: lockedAmount}("");

            if (!sent) {
                revert FailedToSendEther();
            }
        } else {
            IERC20(_token).safeTransfer(msg.sender, lockedAmount);
        }

        emit Withdrawn(msg.sender, _token, lockedAmount);
    }
```

Recommendation:
To mitigate the reentrancy vulnerability, follow these steps:

Implement a nonReentrant modifier to prevent reentrant calls during the execution of critical functions.
Ensure that state changes are made before any external calls are made to prevent manipulation by malicious contracts.

## Non-Critical Issues

| Number  |                      issue                       | Instances |
| ------- | :----------------------------------------------: | --------: |
| [NC-01] |              Activate the Optimizer              |         1 |
| [NC-02] | Navigating new frontiers in transaction fairness |         1 |
| [NC-03] |    Use constants for literal or magic values     |         3 |
| [NC-04] |        Withdrawal Process Not Pull-Based         |         1 |
| [NC-05] |       Missing checks for duplicate tokens        |         1 |
| [NC-06] |        block.timestamp can be manipulated        |         4 |

## [NC-01] : Activate the Optimizer

Before deploying your contract, activate the optimizer when compiling using “solc —optimize —bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ —optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “—optimize-runs” to a high number.

module.exports = {
solidity: {
version: "0.8.24",
settings: {
optimizer: {
enabled: true,
runs: 1000,
},
},
},
};
Please visit this site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here’s one example of instance on opcode comparison that delineates the gas saving mechanism:

for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI
after optimization
DUP1
PUSH1 [revert offset]
JUMPI
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past.

## [NC-02] : Navigating new frontiers in transaction fairness

Navigating new frontiers in transaction fairness
As Layer 2, like Arbitrum or Base, considers moving towards a more decentralized sequencer model, the platform faces the challenge of maintaining its current mitigation of frontrunning risks inherent in a “first come, first served” system.

The transition could reintroduce vulnerabilities to transaction ordering manipulation, demanding innovative solutions to uphold transaction fairness. Strategies such as commit-reveal schemes, submarine sends, Fair Sequencing Services (FSS), decentralized MEV mitigation techniques, and the incorporation of time-locks and randomness could play pivotal roles. These measures aim to preserve the integrity of transaction sequencing, ensuring that the L2’s evolution towards decentralization enhances its ecosystem without compromising the security and fairness that are crucial for user trust and platform reliability.

## [NC-03] : Use constants for literal or magic values

Use constants for literal or magic values

Consider defining constants for literal or magic values as it improves readability and prevents duplication of config values.

**_Instances_**

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L103

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L102

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L253


## [NC-04]  : Withdrawal Process Not Pull-Based 

The current withdrawal process in the contract is not following the pull-based approach.
Contract proactively sends funds to users instead of allowing users to pull funds themselves.
Proof of Concept:

Users cannot initiate the withdrawal process; instead, the contract initiates the transfer of funds to users.
Funds are sent to users using msg.sender.call{value: lockedAmount}("") for ETH and IERC20(_token).safeTransfer(msg.sender, lockedAmount) for tokens other than ETH.
Recommendation:

Implement a pull-based approach where users initiate the withdrawal process and pull funds from the contract themselves.
Modify the withdrawal function to deduct the requested amount from the user's balance and hold it until the user requests to withdraw it.

## [NC-05] :  Missing checks for duplicate tokens


Implement a check in the constructor to ensure that the token addresses provided in the _allowedTokens array are unique. This can be done by using a mapping to track unique tokens and reverting if a token address is encountered more than once.

Updated Constructor with Unique Token Check:

```javascript
constructor(address _exchangeProxy, address _wethAddress, address[] memory _allowedTokens) {
    // Other initialization code...

    // Allow initial list of tokens with unique addresses
    uint256 length = _allowedTokens.length;
    for (uint256 i = 0; i < length; i++) {
        address token = _allowedTokens[i];
        require(token != address(0), "Invalid token address");
        require(!isTokenAllowed[token], "Duplicate token address");
        isTokenAllowed[token] = true;
    }
}
```

## [NC-06] : block.timestamp can be manipulated 


`block.timestamp` can be manipulated by miners to a small extent, so relying on it for precise timing might be risky.
Remediation:
Use `block.timestamp` only where a slight inaccuracy is acceptable, such as for longer intervals.

***instances***
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L316

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L291

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L276

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L102
