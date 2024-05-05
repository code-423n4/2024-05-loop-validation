### [L-1] Checking the Rest of the Call Data for Security

**Description:** 

On line https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L470
of the provided code snippet, the `PrelaunchPoints::callData` variable potentially contains additional parameters like "transformers" and "minOutputTokenAmount" as described in the 0x Protocol documentation on ERC20 transformations. If these parameters are not properly validated, it could lead to malicious activity. Especially, if the minimum output token amount is set to zero, it might not be properly asserted, potentially leading to undesirable outcomes.

**Recommended Mitigation:** 

It's crucial to thoroughly inspect the rest of the `PrelaunchPoints::callData` for any potentially malicious content. Implement robust validation mechanisms to ensure that all parameters, including transformers and minimum output token amounts, are within acceptable ranges and adhere to expected formats. This could involve parsing and validating each parameter individually, checking for any unexpected or suspicious values, and reverting the transaction if any anomalies are detected.

### [L-2] Inability to Withdraw Same Currency Deposited

**Description:** 

Users who deposit WETH into the contract are unable to withdraw WETH directly due to the conversion to ETH during the lock process.

**Impact:** 

This limitation affects users who prefer to withdraw their funds in the same currency they deposited, causing inconvenience and potentially confusion.

**Recommended Mitigation:** 

Consider documenting this behavior clearly to inform users about the withdrawal process. Alternatively, if allowing withdrawals in the same currency is a priority, consider adjusting the contract logic to enable direct withdrawal of WETH. One solution could involve modifying the `PrelaunchPoints::convertAllETH` function to handle the conversion of WETH to ETH there.

### [L-3] Potential DoS Attack via External Exchange Proxy

**Description:** 

There is a potential vulnerability on line https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L497 where the contract calls an external exchange proxy. If this external proxy is compromised or malicious, it could be exploited to execute a denial-of-service (DoS) attack.

**Impact:** 

A successful DoS attack could disrupt the functionality of the contract, leading to service unavailability and financial losses for users. This could result in reputational damage and undermine user trust in the platform.

**Recommended Mitigation:** 

It's essential to thoroughly evaluate the security and reliability of any external services or proxies used by the contract. Consider conducting security audits or due diligence on the external exchange proxy to ensure its integrity and robustness. Additionally, document this potential risk to inform users and stakeholders about the associated security considerations. Alternatively, if feasible, explore options to mitigate reliance on external services or proxies to reduce the vulnerability to potential DoS attacks.


[L-4] Validating `PrelaunchPoints::_percentage` Range
Description:

The `PrelaunchPoints::_percentage` variable is defined as a uint8, allowing it to hold values between 0 and 255. However, in the context of representing percentages, valid values should fall within the range of 0 to 100.

Impact:

Allowing `PrelaunchPoints::_percentage` to accept values outside the valid percentage range could lead to invalid requests or calculations. Users might inadvertently input values outside the expected range, potentially resulting in unexpected behavior or incorrect outcomes.

Recommended Mitigation:

To ensure that `PrelaunchPoints::_percentage` remains within the valid percentage range, implement a validation check to verify that the value falls within the expected bounds. If the value is outside this range, revert the transaction with a clear error message indicating that the provided percentage is invalid. This helps maintain the integrity of the contract's functionality and prevents erroneous inputs from affecting the system's behavior.