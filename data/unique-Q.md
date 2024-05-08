## \[L-01\] Critical Address Changes Should Use Two-step Procedure

The critical procedures should be a two-step process.

```
function setOwner(address _owner) external onlyAuthorized {
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L336

**Recommended Mitigation Steps**  
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two-step procedure on the critical functions.

&nbsp;

## \[L-02\] **Missing check for valid data**

&nbsp;in \`\_claim\` function \`\_percentage\` parameters should checked for invalid value

```
uint256 userClaim = userStake * _percentage / 100;
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L253

&nbsp;check \`\_amount\` parameter for invalid value

```
_processLock(_token, _amount, msg.sender, _referral);
```

## \[L-03\] Insufficient coverage

Description  
The test coverage rate of the project is ~<span style="color: #adabb2;">78.95%</span>%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

&nbsp;

## \[N-04\] constants should be defined rather than using magic numbers

A magic number is a numeric literal that is used in the code without any explanation of its meaning. The use of magic numbers makes programs less readable and hence more difficult to maintain and update.

&nbsp;

```
uint256 userClaim = userStake * _percentage / 100;
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L253

Even assembly can benefit from using readable constants instead of hex/numeric literals.

## \[N-05\] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

&nbsp;

```
address public constant ETH = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;
```

**https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L28**

```
bytes4 public constant UNI_SELECTOR = 0x803ba26d;
    bytes4 public constant TRANSFORM_SELECTOR = 0x415565b0;
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L41C1-L43C60

constants.sol  
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

&nbsp;

## \[N-06\] ORDERS OF FUNCTIONS DO NOT FOLLOW OFFICIAL STYLE GUIDE

Context  
All Contracts

Description  
[Order of Functions;](https://docs.soliditylang.org/en/v0.8.17/style-guide.html) ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Functions should be grouped according to their visibility and ordered:

constructor  
receive function (if exists)  
fallback function (if exists)  
external  
public  
internal  
private  
within a grouping, place the view and pure functions last

## \[N‑7\] Event is missing `indexed` fields

Index event fields make the field more quickly accessible <ins>to off-chain tools</ins> that parse events. However, note that each index field costs extra gas during emission, so it’s not necessarily best to index the maximum allowed per event (three fields). Each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

```
event Recovered(address token, uint256 amount);
    event OwnerUpdated(address newOwner);
    event LoopAddressesUpdated(address loopAddress, address vaultAddress);
    event SwappedTokens(address sellToken, uint256 sellAmount, uint256 buyETHAmount);
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L61C4-L64C86

&nbsp;

## \[N-8\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-9\] NatSpec comments should be increased in contracts

> all contest

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity Official documentation

in complext project such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.  
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[S-01\] You can explain the operation of critical functions in NatSpec with an infographic.