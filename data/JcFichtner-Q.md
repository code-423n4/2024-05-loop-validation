## [C-01] Centralization Risks in the PrelaunchPoints Contract

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol

The provided contract has several points of centralization that could pose risks:

###### 1. Owner Authority:

- **Setting Loop Addresses:** The owner can set the `lpETH` and `lpETHVault` addresses only once within 120 days of deployment. This decision significantly impacts users as it determines where their staked funds are directed and how rewards are generated.

- **Allowing Tokens:** The owner decides which tokens are allowed for locking, potentially excluding certain tokens or favoring others, thereby influencing user choices.

- **Emergency Mode:** The owner can activate/deactivate emergency mode, which alters withdrawal rules and could be misused to restrict access to funds.

- **Recovering ERC20 Tokens:** The owner can recover mistakenly sent tokens, which while intended for legitimate use, could be misused.

- **Setting a New Owner:** While crucial for succession planning, the owner choosing their successor concentrates power.

###### 2. Exchange Proxy:

- The contract relies on a single `exchangeProxy` for token swaps, creating a dependence on a specific external service. If the `exchangeProxy` experiences issues or becomes compromised, it could disrupt the claim and stake functionality and impact user experience.

###### 3. Timelock Period:

- While the 7-day timelock after setting Loop addresses adds a layer of security, it also delays user actions and limits their flexibility during that period.

###### Centralization Score: High

Given the significant control concentrated in the owner's hands and the reliance on a single exchange proxy, the centralization score for this contract is `high`.

The owner's ability to make critical decisions without community involvement and the dependence on a single external service introduce notable risks and vulnerabilities.

###### Recommendations for Mitigation:

- **Decentralized Governance:** Implement a DAO structure where token holders vote on critical decisions such as setting Loop addresses, allowing tokens, and activating emergency mode.

- **Multiple Exchange Proxies:** Integrate multiple exchange proxies to diversify risk and avoid dependence on a single service.

- **Timelock Adjustments:** Explore options for adjusting the timelock period based on community consensus, balancing security with user flexibility.

- **Transparency and Communication:** Maintain open communication with the community about any changes and the rationale behind them, fostering trust and accountability.








