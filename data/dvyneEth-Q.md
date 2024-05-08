## Missing check for `msg.sender` in `lockEthFor` and  `lockFor`

when inputing value in `lockFor` or `lockEthFor` there should be a check that prevents  `msg.sender` from being the value of `_for`  
- found in `src/PrelaunchPoints.sol` [Ln :139] and [ Ln :165]

``` solidity
function lockFor(address _token, uint256 _amount, address _for, bytes32 _referral) external {
        if (_token == ETH) {
            revert InvalidToken();
        }
        _processLock(_token, _amount, _for, _referral);
    }
```
and 

``` solidity
function lockETHFor(address _for, bytes32 _referral) external payable {
        _processLock(ETH, msg.value, _for, _referral);
    }
```

**Recommended Mitigation** 
add a check that prevents msg.sender from being the value of `_for` 