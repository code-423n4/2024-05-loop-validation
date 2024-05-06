
### [L-1] Only an underflow protects the protocol from users setting `_percentage` parameter to be greater than `100`

If the malicious user tries to make their own `_data` with `inputTokenAmount` greater than their `userStake` together with using the `_percentage` parameter with value greater than 100, the call reverts with an underflow error which is not an ideal form of protection. In `_claim()` function only the following line ensures that the user does not set the `_percentage` parameter to be more than `100`:

```javascript
balances[msg.sender][_token] = userStake - userClaim;
```

because in this case `userClaim > userStake`.

This could be easily mitigated with a simple check in `_claim()` to ensure `_percentage` parameter is within bounds:

```diff
 uint256 userStake = balances[msg.sender][_token];
 if (userStake == 0) {
         revert NothingToClaim();
 }
+if (_percentage > 100) {
+        revert PercentageGreaterThanMax();
+}
```

To run a PoC, create a file `./test/PercentageOver100.test.ts` and paste there code from this [gist](https://gist.github.com/bbl4de/09489f41257e42c130ab0d79088eae44), then run the test using `yarn hardhat test ./test/PercentageOver100.test.ts`.

### [I-1] The owner making a mistake in calling `setLoopAddresses()` is irreversible

In case the owner by mistake passes the wrong addresses as parameters to the `setLoopAddresses()` function, they're unusable and the protocol is forced to deploy the contract again, which is an issue for all users that have already locked some stake.

### [I-2] Two different custom errors with exactly the same meaning

There are two very similar error messages: `InvalidToken()` and `NotValidToken()`. Consider removing one of these for simplicity, unless their distinction is essential for the off-chain backend.
