## NC-01: Event is missing `indexed` fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

- Found in src/PrelaunchPoints.sol [Line: 56](https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L56)

	```solidity
	    event Locked(address indexed user, uint256 amount, address token, bytes32 indexed referral);
	```

- Found in src/PrelaunchPoints.sol [Line: 57](https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L57)

	```solidity
	    event StakedVault(address indexed user, uint256 amount);
	```

- Found in src/PrelaunchPoints.sol [Line: 59](https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L59)

	```solidity
	    event Withdrawn(address indexed user, address token, uint256 amount);
	```

- Found in src/PrelaunchPoints.sol [Line: 60](https://github.com/code-423n4/2024-05-loop/blob/0dc8467ccff27230e7c0530b619524cc8401e22a/src/PrelaunchPoints.sol#L60)

	```solidity
	    event Claimed(address indexed user, address token, uint256 reward);
	```

