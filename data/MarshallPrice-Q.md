Absence of event in allowToken() function.

Events make contract operations more transparent and available for analysis. They help developers and auditors track changes in contract state and analyze contract behavior

Adding an event when authorized tokens change makes the process more consistent and transparent. This helps maintain good contract state management practices.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L364-L366

```solidity
function allowToken(address _token) external onlyAuthorized {
        isTokenAllowed[_token] = true;
    }
```

Recommendations:
Add specific events to the contract. 

```solidity
function allowToken(address _token) external onlyAuthorized {
         isTokenAllowed[_token] = true;
         emit TokenAllowed(_token);
    }
```
