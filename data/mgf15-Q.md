L-1 `_decodeUniswapV3Data` and `_decodeTransformERC20Data` do not validate `_data` size

these functions do not validate whether the size of the `_data` provided is sufficient or valid. This can lead to potential issues like reading out-of-bound data or encountering unexpected behavior if `_data` does not match the expected size.

```solidity
function _decodeUniswapV3Data(bytes calldata _data)
        internal
        pure
        returns (address inputToken, address outputToken, uint256 inputTokenAmount, address recipient, bytes4 selector)
    {
        uint256 encodedPathLength;
        assembly {
            let p := _data.offset
            selector := calldataload(p)
            p := add(p, 36) // Data: selector 4 + lenght data 32
            inputTokenAmount := calldataload(p)
            recipient := calldataload(add(p, 64))
            encodedPathLength := calldataload(add(p, 96)) // Get length of encodedPath (obtained through abi.encodePacked)
            inputToken := shr(96, calldataload(add(p, 128))) // Shift to the Right with 24 zeroes (12 bytes = 96 bits) to get address
            outputToken := shr(96, calldataload(add(p, add(encodedPathLength, 108)))) // Get last address of the hop
        }
    }
```
```solidity
function _decodeTransformERC20Data(bytes calldata _data)
        internal
        pure
        returns (address inputToken, address outputToken, uint256 inputTokenAmount, bytes4 selector)
    {
        assembly {
            let p := _data.offset
            selector := calldataload(p)
            inputToken := calldataload(add(p, 4)) // Read slot, selector 4 bytes
            outputToken := calldataload(add(p, 36)) // Read slot
            inputTokenAmount := calldataload(add(p, 68)) // Read slot
        }
    }
```

L-2 contract can't disallow token 

there no function to set isTokenAllowed[_token] to false if any thing happen in the future.

```solidity
    function allowToken(address _token) external onlyAuthorized {
        isTokenAllowed[_token] = true;
    }
```
L-3 withdraw function dos not update total supply if the token is weth

in `_processLock` function when the token is ETH or WETH the totalSupply got increased but `withdraw` it decrease the totalSupply
only if 

`if (_token == ETH) {`

```solidity
function withdraw(address _token) external {
        if (!emergencyMode) {
            if (block.timestamp <= loopActivation) {
                revert CurrentlyNotPossible();
            }
            if (block.timestamp >= startClaimDate) {
                revert NoLongerPossible();
            }
        }

        uint256 lockedAmount = balances[msg.sender][_token];
        balances[msg.sender][_token] = 0;

        if (lockedAmount == 0) {
            revert CannotWithdrawZero();
        }
        if (_token == ETH) {
            if (block.timestamp >= startClaimDate){
                revert UseClaimInstead();
            }
            totalSupply = totalSupply - lockedAmount;

            (bool sent,) = msg.sender.call{value: lockedAmount}("");

            if (!sent) {
                revert FailedToSendEther();
            }
        } else {
            IERC20(_token).safeTransfer(msg.sender, lockedAmount);
        }

        emit Withdrawn(msg.sender, _token, lockedAmount);
    }
```

N-1 `claimAndStake` should emit token address

```solidity
function claimAndStake(address _token, uint8 _percentage, Exchange _exchange, bytes calldata _data)
        external
        onlyAfterDate(startClaimDate)
    {
        uint256 claimedAmount = _claim(_token, address(this), _percentage, _exchange, _data);
        lpETH.approve(address(lpETHVault), claimedAmount);
        lpETHVault.stake(claimedAmount, msg.sender);
        
        emit StakedVault(msg.sender, claimedAmount);
    }
```

N-2 typo 
```
     * @notice Validates the data sent from 0x API to match desired behaviour //@audit typo behavior

```
```
            p := add(p, 36) // Data: selector 4 + lenght data 32 //@audit typo length 

```
