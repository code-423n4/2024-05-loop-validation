# Inconsistent handling on `limitDate`

Both `onlyAfterDate` and `onlyBeforeDate` will consider the `block.timestamp == limitDate` as a revert condition.
```solidity
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
