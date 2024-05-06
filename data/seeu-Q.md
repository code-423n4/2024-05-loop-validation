## Low Issues

| ID | Issues | Contexts | Instances |
|----|--------|----------|-----------|
| [L-01](#L-01-timestamp-dependency-use-of-blocktimestamp-or-now) | Timestamp dependency: use of `block.timestamp` (or `now`) | 1 | 9 |

### [L-01] Timestamp dependency: use of `block.timestamp` (or `now`)

The timestamp of a block is provided by the miner who mined the block. As a result, the timestamp is not guaranteed to be accurate or to be the same across different nodes in the network. In particular, an attacker can potentially mine a block with a timestamp that is favorable to them, known as "selective packing". For example, an attacker could mine a block with a timestamp that is slightly in the future, allowing them to bypass a time-based restriction in a smart contract that relies on `block.timestamp`. This could potentially allow the attacker to execute a malicious action that would otherwise be blocked by the restriction. It is reccomended to, instead, use an alternative timestamp source, such as an oracle, that is not susceptible to manipulation by a miner. References: [Timestamp dependence | Solidity Best Practices for Smart Contract Security](https://consensys.net/blog/developers/solidity-best-practices-for-smart-contract-security/), [What Is Timestamp Dependence?](https://halborn.com/what-is-timestamp-dependence/).

#### Instances (9)

```JavaScript
src/PrelaunchPoints.sol
::115 =>         loopActivation = uint32(block.timestamp + 120 days);
::309 =>             if (block.timestamp <= loopActivation) {
::312 =>             if (block.timestamp >= startClaimDate) {
::324 =>             if (block.timestamp >= startClaimDate) {
::353 =>         if (block.timestamp - loopActivation <= TIMELOCK) {
::364 =>         startClaimDate = uint32(block.timestamp);
::391 =>         loopActivation = uint32(block.timestamp);
::596 =>         if (block.timestamp <= limitDate) {
::603 =>         if (block.timestamp >= limitDate) {
```
