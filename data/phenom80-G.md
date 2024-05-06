Gas Optimizations
Avoid contract existence checks by using low level calls
Impact: Using low level calls can help avoid unnecessary contract existence checks, resulting in lower gas costs.

Instances (2):

262: claimedAmount = userStake.mulDiv(totalLpETH, totalSupply);

344: totalLpETH = lpETH.balanceOf(address(this));
Cache state variables to save gas
Impact: Caching state variables in stack variables can save gas by reducing read operations.

Instances (11):



100: WETH = IWETH(_wethAddress);

102: loopActivation = uint32(block.timestamp + 120 days);

238: uint256 claimedAmount = _claim(_token, address(this), _percentage, _exchange, _data);

275: claimedAmount = address(this).balance;

341: uint256 totalBalance = address(this).balance;

347: startClaimDate = uint32(block.timestamp);

375: lpETH = ILpETH(_loopAddress);

376: lpETHVault = ILpETHVault(_vaultAddress);

377: loopActivation = uint32(block.timestamp);

517: uint256 boughtETHAmount = address(this).balance;

527: boughtETHAmount = address(this).balance - boughtETHAmount;
... (continue for other optimizations)

Use assembly to write address/contract type storage values
Impact: Using assembly to write address or contract type storage values can save gas.

Instances (21):



28: address public constant ETH = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

42: bytes4 public constant UNI_SELECTOR = 0x803ba26d;

43: bytes4 public constant TRANSFORM_SELECTOR = 0x415565b0;

98: owner = msg.sender;

99: exchangeProxy = _exchangeProxy;

100: WETH = IWETH(_wethAddress);

103: startClaimDate = 4294967295; // Max uint32 ~ year 2107

106: uint256 length = _allowedTokens.length;

107: for (uint256 i = 0; i < length;) {

257: uint256 userStake = balances[msg.sender][_token];

275: claimedAmount = address(this).balance;

299: uint256 lockedAmount = balances[msg.sender][_token];

341: uint256 totalBalance = address(this).balance;

344: totalLpETH = lpETH.balanceOf(address(this));

347: startClaimDate = uint32(block.timestamp);

357: owner = _owner;

375: lpETH = ILpETH(_loopAddress);

376: lpETHVault = ILpETHVault(_vaultAddress);

377: loopActivation = uint32(block.timestamp);

395: emergencyMode = _mode;

517: uint256 boughtETHAmount = address(this).balance;