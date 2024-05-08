## 1. `uint32` TIMESTAMPS WILL OVERFLOW AFTER YEAR 2107 THUS BREAKING THE PROTOCOL

The `timestamp` state variables which are used by the `PrelaunchPoints.sol` contract, such as `loopActivation` and `startClaimDate` are declared as `uint32` variables. As a result these variables can only be used till year 2107 (approximately) since after that the `uint32` variables will overflow and the transactions will revert thus breaking the protocol.

```solidity
    uint32 public loopActivation;
    uint32 public startClaimDate;
```

Hence it is recommended to declare the `timestamp` state variables to higher uint data type such as `uint64`, thus allowing them to store long periods beyond year 2107.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L45-L46

## 2. RETURN BOOL VALUE OF THE `lpETH.approve` IS NOT CHECKED FOR SUCCESS

In the `PrelaunchPoints.claimAndStake` function the `PrelaunchPoints` contract approves the `claimedAmount` of `lpETH` tokens to the `lpETHVault` contract as shown below:

```solidity
        lpETH.approve(address(lpETHVault), claimedAmount);
```

The `lpETH` token contract inherits from the openzeppelin `IERC20` interface and the `approve` function returns a `bool` value to indicate the success of the operation. But as it is evident from the above code snippet of `lpETH.approve` function call, the return `bool` value is not checked for success.

Hence as a result even if the `approval` is failed, the transaction proceeds with the execution without revert, thus breaking the internal accounting of `lpETH` token transfers and token approval.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L231

## 3. NO RECOVER FUNCTION TO RETRIEVE THE `ETH` TOKEN MISTAKENLY SENT TO THE CONTRACT 

In the `PrelaunchPoints` contract the `recoverERC20` function is used to recover other `ERC20s` mistakingly sent to this contract. But there is no such function to recover the `native ETH` tokens direclty sent to the `PrelaunchPoints` contract.

After the `convertAllETH` function is called and entire balance of the `ETH` is converted to `lpETH`, there is no method to convert or recover the `ETH` directly sent to this contract.

Hence it is recommended to add a `recover` function for `ETH` which is mistakenly sent to `PrelaunchPoints` contract. This function should only be callable by the `owner` of the contract (Controlled by the `onlyAuthorized` modifier). To compute `the mistakenly sent ETH`, it is recommended to keep an internal mapping of `ETH` token amount which is transacted during the contract operations.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L379-L386
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L392

## 4. `PrelaunchPoints.claim` TRANSACTION COULD REVERT IF THE `approve` FUNCTION OF THE `_sellToken` DOES NOT RETURN A `BOOL` VALUE

The `PrelaunchPoints.claim` and `PrelaunchPoints.claimAndStake` functions call the `_claim` function, to get the vested `lpETH` tokens of the `msg.sender`. If the `_token` is an `ERC20` token then the `_fillQuote` function is called to swap the `ERC20` into `ETH` to calculate the `claimedAmount`.

The `_fillQuote` function performs the following `approve` transaction to let the `PrelaunchPoints` contract to approve `_amount` of `_sellToken` to the `exchangeProxy` contract as shown below:

```solidity
        require(_sellToken.approve(exchangeProxy, _amount)); 
```

But the issue here is that not all `tokens` follow the `ERC20 standard` and some tokens might not return a `bool` value when `approve` is called. As a result the `require` statement will always revert even if the `approval` was succesfuly executed.

Hence it is recommended to check whether the `approve` function returns a `bool` value before checking the `success` of the return value.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L495

## 5. RECOMMENDED TO USE THE `safeApprove` INPLACE OF `approve` FUNCTION FOR EXTERNAL ERC20 CONTRACT CALLS

The `PrelaunchPoints._fillQuote` function calls on the `approve` function of the `_sellToken` as shown below:

```solidity
        require(_sellToken.approve(exchangeProxy, _amount));
```

But the issue here is that there are certain `ERC20 tokens` which will revert the `approve` transaction if the existing `allowance > 0` (Eg: USDT). Hence the `_fillQuote` function will not be compatible with such `ERC20` tokens.

Hence it is recommended to use the `safeApprove` function of the `openzeppelin safeERC20 library`. The `safeApprove` function call will make the `current allowance to 0` if it is already non-zero before calling on the `new approve` transaction.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L495

## 6. TOKENS WITH MULTIPLE ADDRESSES COULD BREAK THE INTERNAL ACCOUNTING OF THE CONTRACT IF TRANSFERRED VIA THE `PrelaunchPoints.recoverERC20` FUNCTION BY THE OWNER

The `PrelaunchPoints.recoverERC20` function is used to recover other ERC20s mistakingly sent to this contract. If the token is not equal to the `address(lpETH)` or one of the allowed tokens of the contract state, then that respective token can be transferred to the `owner` address.

But the issue here is that there are `ERC20 tokens` with muliple addresses as well. Let's consider a token with 2 addresses pointing to the same token. Hence if the `first address` is `whitelisted` and included in the `isTokenAllowed mapping`, an owner might mistakenly use the `second address` of the same token and call the `PrelaunchPoints.recoverERC20` function thus transferring the tokens to the `owner's` address.

Hence this will break the internal accounting of the contract. Since the token balances of that token in the `balances` mapping is not changed but the actual contract balance of that token is not less than the value stored in the state of the `PrelaunchPoints.balances` contract mapping.

Hence it is recommended to ensure that tokens with `two addresses` are not whitelisted in the `PrelaunchPoints` contract.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L379-L386

## 7. `PrelaunchPoints._processLock` IS NOT COMPATIBLE WITH THE `fee-on-transfer` TOKENS

The `PrelaunchPoints._processLock` function is used to lock the `_token` amounts in the contract and to update the respective balances of the contract. 

The `_processLock` function uses the `safeTransferFrom` function to transfer the `_amount` of assets from the `msg.sender` to the `PrelaunchPoints` contract. And then the transferred amount is used to update the `balances` mapping for the respective `_receiver` and the `_token` as shown below:

```solidity
            IERC20(_token).safeTransferFrom(msg.sender, address(this), _amount);
```

```solidity
            } else {
                balances[_receiver][_token] += _amount;
            }
```

But the issue here is that if the `_token` is `fee-on-transfer` then the actual transferred `_token` amount to the contract will be less than the `_amount` which is added on to the existing balance. Hence this will break the internal accounting of the `balances` mapping for the respective `receiver` and `_token` combination.

Hence it is recommended to get the `before transfer` and `after transfer` balance of the `_token` and add the difference of the above balances to the `balances` mapping of the respective `_receiver` and `_token` combination.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L186
https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L193

## 8. RECOMMENDED TO USE TWO STEP OWNERSHIP TRANSFER PROCESS IN THE `PrelaunchPoints.setOwner` FUNCTION

The `PrelaunchPoints.setOwner` function is used to change the `owner` of the contract as shown below:

```solidity
    function setOwner(address _owner) external onlyAuthorized {
        owner = _owner;

        emit OwnerUpdated(_owner);
    } 
```

But the issue here is that if the new `_owner` is set to an `erroneous address` it is irrecoverable and the contract's `onlyAuthorized` modifier controlled function will not be callable. 

Hence it is recommended to use the `two step ownership transfer` process in the `setOwner` function as it is done in the `Ownable2Step.sol` contract of `openzeppelin`. This will ensure that the new owner is elected first and then that `new owner` will trigger the transaction of `ownership transfer`.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L336-L340

## 9. NO INPUT VALIDATION CHECKS ARE PERFORMED FOR `address(0)` ON THE `PrelaunchPoints.setLoopAddresses` FUNCTION WHICH CAN BE CALLED ONLY ONCE

The `PrelaunchPoints.setLoopAddresses` function can only be called only once, before 120 days have passed from deployment. The `setLoopAddresses` function is used to set the `lpETH` and `lpETHVault` contract addresses as shown below:

```solidity
        lpETH = ILpETH(_loopAddress);
        lpETHVault = ILpETHVault(_vaultAddress);
```

But the issue is that there is no input validation check of `address(0)` on the `_loopAddress` and `_vaultAddress` addresses. Since `setLoopAddresses` can only be called once, the protocol will end up in a broken state if either `lpETH` or `lpETHVault` is set to `address(0)`.

Hence it is recommended to perform the input validation checks on the `lpETH` and `lpETHVault` contracts for `address(0)`.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L348-L358

## 10. THE `outputToken` VALUE RETRIEVAL IN THE `PrelaunchPoints._decodeUniswapV3Data` FUNCTION IS ERRORNEOUS SINCE WRONG OFFSET IS USED

The `PrelaunchPoints._decodeUniswapV3Data` function is used to decode the data sent from `0x API` when `UniswapV3` is used. 

The `outputToken` in the passed in `_data` is retrieved as shown below:

```solidity
            outputToken := shr(96, calldataload(add(p, add(encodedPathLength, 108))))
```

But there seems to be an error in the `offset` used for the `retrieval` of the `outputToken` since `108` is used inplace of the `128`. Hence the returned `outputToken` address is errorneous and will revert the `execution of the claim` transaction since `_validateData` function call will fail.

Hence it is recommended to update the `offset` to `128` in when retriving the `outputToken` value using assmbly.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L462

## 11. THE `ETH` CAN BE LOST AS IT IS TRANSFERRED TO `addresss(0)` AFTER SWAPS, SINCE THE `PrelaunchPoints._validateData` ALLOWS THE `recipient` ADDRESS TO BE `address(0)`

The `PrelaunchPoints._validateData` function is used to validate the data for a `claim` transaction. In this function the `recipient` address is retrived by calling the `_decodeUniswapV3Data` function or the `_decodeTransformERC20Data` based on the `exchange type`.

There is a check to validate the `recipient` address as shown below:

```solidity
        if (recipient != address(this) && recipient != address(0)) {
            revert WrongRecipient(recipient);
        }
```

The above check allows the `address(0)` as a valid `recipient` address. This is because the `_decodeTransformERC20Data` function call does not return the `recipient` address and hence by default is address(0).

But since the above check is common to both the `_decodeUniswapV3Data` and `_decodeTransformERC20Data` function calls, the transaction will execute succesfully without revert even if the `_decodeUniswapV3Data` function call returns `address(0)` for the `recipient` address. 

Hence if the swap happens in the `UniswapV3` with the `recipient` being address(0), this will result in loss of funds since the returned `ETH` tokens from the swap will be transferred to the `address(0)`.

Hence it is recommended to revert if the `recipient is address(0)` when the `recipient` is retrieved via the `_decodeUniswapV3Data` function call.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L439-L441

## 12. THE `PrelaunchPoints.withdraw` FUNCTION IS VULNERABLE TO REENTRANCY ATTACKS

The `PrelaunchPoints.withdraw` function is called by a staker to withdraw all their ETH or LRT. During the `ETH` withdrawal the `lockedAmount` is transferred to the `msg.sender` as shown below:

```solidity
            (bool sent,) = msg.sender.call{value: lockedAmount}("");
```

But the issue here is that the `msg.sender` can be an external contract with a malicious code in its `recieve` function, thus can reenter the `PrelaunchPoints.withdraw` function or any other state changing function in the `PrelaunchPoints` contract during `native ETH transfers`, since the reentrancy guards are not used in the critical functions of the `PrelaunchPoints` contract.

This makes the code vulnerable to reentrancy attacks. 

Hence it is recommended to use `reentrancy guards` in the critical functions of the `PrelaunchPoints` contract such as on the `PrelaunchPoints.withdraw` function.

https://github.com/code-423n4/2024-05-loop/blob/main/src/PrelaunchPoints.sol#L296