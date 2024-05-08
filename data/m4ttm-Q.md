### Separate `WrongDataTokens` into `WrongDataInputToken` and `WrongDataOutputToken`

In `_validateData`, the same custom error is thrown with the input and output token as arguments in the case that either one is invalid, with no information on which of the two was invalid. Separating this into two separate errors would increase clarity to users.

```solidity
    error WrongDataTokens(address inputToken, address outputToken);
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L79

### Name Ownable function and modifier `transferOwner` and `onlyOwner`

The function and modifier `setOwner` and `onlyAuthorized` implement standard ownership functionality. These are typically called `transferOwner` and `onlyOwner`. Using standard names for standard functionality increases clarity for readers familiar with Solidity. This is often implemented by inheriting from a trusted contract such as OpenZeppelin and EIPs have been created on the subject such as (EIP-173)\[https://eips.ethereum.org/EIPS/eip-173\].

```solidity
    function setOwner(address _owner) external onlyAuthorized {
        owner = _owner;

        emit OwnerUpdated(_owner);
    }
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L336-L340

### Use `limitTime` as modifier argument names rather than `limitDate`

The modifiers `onlyBeforeDate` and `onlyAfterDate` take a single argument `limitDate`, which is a timestamp. Calling this `limitTime` would increase clarity that this is in fact a timestamp and represents a point in time rather than a specific calendar date.

```solidity
    modifier onlyAfterDate(uint256 limitDate) {
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L518

```solidity
    modifier onlyBeforeDate(uint256 limitDate) {
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L525

### Avoid repeated code to check timestamp constraints with internal functions

Timestamp constraints are checked with the modifiers `onlyBeforeDate` and `onlyAfterDate` but these checks are also implemented conditionally in some functions so modifiers are not appropriate. In these cases, the checks are implemented with if statements and custom errors. Using internal functions would prevent the need to duplicate this code and increase freedom around their usage.

```solidity
    modifier onlyAfterDate(uint256 limitDate) {
        if (block.timestamp <= limitDate) {
            revert CurrentlyNotPossible();
        }
        _;
    }
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L518-L523

```solidity
    modifier onlyBeforeDate(uint256 limitDate) {
        if (block.timestamp >= limitDate) {
            revert NoLongerPossible();
        }
        _;
    }
```

https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L525-L530