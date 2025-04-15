### [GAS-1] Use Error to check for `address(0)`
*Saves 80 gas per instance*

[Details Link](https://medium.com/@kalexotsu/solidity-assembly-checking-if-an-address-is-0-efficiently-d2bfe071331)

*Example *
Use error ZeroAddress() to save around 80 gas per call
```solidity
error ZeroAddress();
function solidity_notZero(address toCheck) public pure returns(bool success) {
    if(toCheck == address(0)) revert ZeroAddress();
    return true;
}
```

*Instances (44)*:
```solidity
File: ./src/KHYPE.sol

57:         require(admin != address(0), "Invalid admin address");

58:         require(_pauserRegistry != address(0), "Invalid pauser registry address");

59:         require(minter != address(0), "Invalid minter address");

60:         require(burner != address(0), "Invalid burner address");

```
[Link to code](https://github.com/code-423n4/2025-04-kinetiq/blob/main/./src/KHYPE.sol)

```solidity
File: ./src/OracleManager.sol

77:         require(_pauserRegistry != address(0), "Invalid pauser registry");

78:         require(_validatorManager != address(0), "Invalid validator manager");

99:         require(adapter != address(0), "Invalid oracle address");

148:         require(validator != address(0), "Invalid validator address");

242:         if (address(sanityChecker) != address(0)) {

```
[Link to code](https://github.com/code-423n4/2025-04-kinetiq/blob/main/./src/OracleManager.sol)

```solidity
File: ./src/PauserRegistry.sol

57:         require(admin != address(0), "Invalid admin address");

58:         require(pauser != address(0), "Invalid pauser address");

59:         require(unpauser != address(0), "Invalid unpauser address");

60:         require(pauseAll != address(0), "Invalid pauseAll address");

130:         require(contractAddress != address(0), "Invalid contract address");

```
[Link to code](https://github.com/code-423n4/2025-04-kinetiq/blob/main/./src/PauserRegistry.sol)

```solidity
File: ./src/StakingAccountant.sol

63:         require(admin != address(0), "Invalid admin address");

64:         require(manager != address(0), "Invalid manager address");

65:         require(_validatorManager != address(0), "Invalid validator manager");

83:         require(manager != address(0), "Invalid manager address");

84:         require(kHYPEToken != address(0), "Invalid kHYPE token address");

```
[Link to code](https://github.com/code-423n4/2025-04-kinetiq/blob/main/./src/StakingAccountant.sol)

```solidity
File: ./src/StakingManager.sol

154:         require(_pauserRegistry != address(0), "Invalid pauser registry");

155:         require(_kHYPE != address(0), "Invalid kHYPE token");

156:         require(_validatorManager != address(0), "Invalid validator manager");

157:         require(_stakingAccountant != address(0), "Invalid staking accountant");

158:         require(admin != address(0), "Invalid admin address");

159:         require(operator != address(0), "Invalid operator address");

160:         require(manager != address(0), "Invalid manager address");

161:         require(_treasury != address(0), "Invalid treasury address");

292:         require(currentDelegation != address(0), "No delegation set");

515:         require(validator != address(0), "Invalid validator address");

841:             require(accounts[i] != address(0), "Invalid address");

987:         require(newTreasury != address(0), "Invalid treasury address");

1050:         require(validator != address(0), "Invalid validator address");

```
[Link to code](https://github.com/code-423n4/2025-04-kinetiq/blob/main/./src/StakingManager.sol)

```solidity
File: ./src/ValidatorManager.sol

110:         require(admin != address(0), "Invalid admin address");

111:         require(manager != address(0), "Invalid manager address");

112:         require(_oracle != address(0), "Invalid oracle address");

113:         require(_pauserRegistry != address(0), "Invalid pauser registry");

132:         require(validator != address(0), "Invalid validator address");

208:             require(validators[i] != address(0), "Invalid validator address");

451:         require(stakingManager != address(0), "Invalid staking manager");

465:         require(validator != address(0), "No delegation set");

```
[Link to code](https://github.com/code-423n4/2025-04-kinetiq/blob/main/./src/ValidatorManager.sol)

```solidity
File: ./src/oracles/DefaultAdapter.sol

18:         require(_defaultOracle != address(0), "Invalid oracle address");

```
[Link to code](https://github.com/code-423n4/2025-04-kinetiq/blob/main/./src/oracles/DefaultAdapter.sol)

```solidity
File: ./src/oracles/DefaultOracle.sol

61:         require(admin != address(0), "Invalid admin");

62:         require(operator != address(0), "Invalid operator");

91:         require(validator != address(0), "Invalid validator");

```
[Link to code](https://github.com/code-423n4/2025-04-kinetiq/blob/main/./src/oracles/DefaultOracle.sol)


###  [G-2] Costly operations inside a loop 

## Description

Costly operations inside a loop might waste gas, so optimizations are justified.

*Instances (4)*:
```solidity
File: ./src/StakingManager.sol

398:          totalQueuedWithdrawals -= hypeAmount;

399:          totalClaimed += hypeAmount;

529:          hypeBuffer = currentBuffer - amountFromBuffer;
```
[Link to code](https://github.com/code-423n4/2025-04-kinetiq/blob/main/./src/KHYPE.sol)
```solidity
File: ./src/StakingManager.sol

270:          delete validatorRebalanceRequests[validator];

```
[Link to code](https://github.com/code-423n4/2025-04-kinetiq/blob/main/./src/StakingManager.sol)

## Recommendation
Use a local variable to hold the loop computation result.
