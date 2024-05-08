# [L-01] totalLpETH can be bigger than totalSupply
The `totalSupply` variable represents the sum of all ETH and WETH deposits in the contract. When the `convertAllETH` function is called, `totalLpETH` equals the balance of the smart contract. Currently, the smart contract can receive ETH through the receive function. In cases where a mistake or malicious user sends ETH to the contract and the `convertAllETH` function is called, `totalLpETH` can be greater than `totalSupply`. This could pose problems when calculating the claimAmount."


`claimedAmount = userStake.mulDiv(totalLpETH, totalSupply);`

The claimed amount will not be in a 1:1 ratio of ETH to lpETH; as a result, the last user who claims their reward will receive less lpETH than expected.

# [I-01] Data can have dirty bytes

The `calldataload` opcode returns 32 bytes from a given offset. The data decoded for `TransformERC20` and `UniswapV3` can contain dirty bytes. In the case of dirty bytes, the encoding will be incorrect and can lead to unexpected result.

```solidity
        assembly {
            //@audit-issue clean dirty bytes
            let p := _data.offset
            selector := calldataload(p)
            inputToken := calldataload(add(p, 4)) // Read slot, selector 4 bytes
            outputToken := calldataload(add(p, 36)) // Read slot
            inputTokenAmount := calldataload(add(p, 68)) // Read slot
        }
```

## Make the following changes:
```diff
        assembly {
-            let p := _data.offset
-            selector := calldataload(p)
-            inputToken := calldataload(add(p, 4)) // Read slot, selector 4 bytes
-            outputToken := calldataload(add(p, 36)) // Read slot
            inputTokenAmount := calldataload(add(p, 68)) // Read slot

+            selector := shl(224, shr(224, calldataload(add(p, 4))))
+            inputToken := shr(96, shl(96, calldataload(add(p, 4))))
+            outputToken := shr(96, shl(96,calldataload(add(p, 36))))
        }
```
