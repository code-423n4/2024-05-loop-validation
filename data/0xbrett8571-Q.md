## Overview
PrelaunchPoints.sol suffers from a centralized ownership and governance vulnerability, where a single address has complete control over critical contract functions and parameters.

## Description
The contract has several privileged functions that can only be called by the `owner` address. These functions include:

- [setOwner(address _owner)](https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L336-L340): Sets a new owner address.
- [setLoopAddresses(address _loopAddress, address _vaultAddress)](https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L348-L358): Sets the addresses of the `lpETH` and `lpETHVault` contracts.
- [allowToken(address _token)](https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L364-L366): Allows a new token to be used for locking.
- [setEmergencyMode(bool _mode)](https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L372-L374): Enables or disables the emergency mode.
- [recoverERC20(address tokenAddress, uint256 tokenAmount)](https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L379-L386): Recovers any ERC20 token sent to the contract by mistake.

These functions are protected by the [onlyAuthorized modifier](https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L511-L516):

```solidity
modifier onlyAuthorized() {
    if (msg.sender != owner) {
        revert NotAuthorized();
    }
    _;
}
```

## Vulnerability Details
Lies in the centralized control granted to the `owner` address. Only the `owner` can call the privileged functions mentioned above, allowing them to make critical changes to the contract's behavior and parameters.

The `owner` variable is set to `msg.sender` during the contract's construction: https://github.com/code-423n4/2024-05-loop/blob/40167e469edde09969643b6808c57e25d1b9c203/src/PrelaunchPoints.sol#L97-L114

```solidity
constructor(address _exchangeProxy, address _wethAddress, address[] memory _allowedTokens) {
    owner = msg.sender;
    // ...
}
```

Once set, the `owner` address has complete control over the contract's critical functions, and there is no mechanism to change or revoke this ownership in a decentralized or transparent manner.

> In the PrelaunchPoints contract, a single `owner` address has complete control over critical functions, allowing them to make unilateral decisions that can significantly impact the contract's behavior and the users' funds.

## Impact
The centralized ownership and governance vulnerability can have the following impacts:

- If the `owner` address is compromised or becomes malicious, it can lead to the abuse of privileged functions, potentially causing loss of user funds or unauthorized changes to the contract's behavior.
- Decisions made by the `owner` address are not transparent to the users or the community, leading to a lack of trust and accountability.
- The interests of the `owner` address may not always align with the interests of the users or the broader community, potentially leading to decisions that favor the `owner` at the expense of others.

## Mitigation
The PrelaunchPoints contract should implement a decentralized governance mechanism. This can be achieved by introducing a governance token and a voting system, where critical decisions and parameter changes are proposed and voted on by token holders.

1. Introduce a governance token (e.g., `GovernanceToken`) that represents voting power in the system.
2. Create a `GovernanceContract` that manages proposals, voting, and execution of approved proposals.
3. Add a function in the `PrelaunchPoints` contract that allows the `GovernanceContract` to make privileged calls, such as `setOwner`, `setLoopAddresses`, `allowToken`, and `setEmergencyMode`.
4. Implement a proposal and voting mechanism in the `GovernanceContract`, where token holders can create proposals, vote on them, and execute approved proposals by calling the privileged functions in the `PrelaunchPoints` contract.
5. Introduce mechanisms for delegation and vote weighting based on token holdings to ensure fair representation and prevent centralization of voting power.