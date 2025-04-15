###  [QA-1] Missing return value check when making external call
The return value of an external transfer/transferFrom call is not checked

## Exploit Scenario:
```solidity
contract Token {
    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success);
}
contract MyBank{  
    mapping(address => uint) balances;
    Token token;
    function deposit(uint amount) public{
        token.transferFrom(msg.sender, address(this), amount);
        balances[msg.sender] += amount;
    }
}
```
Several tokens do not revert in case of failure and return false. If one of these tokens is used in MyBank, deposit will not revert if the transfer fails, and an attacker can call deposit for free..

## Transfer and TransferFrom
*Instances (3)*:
```solidity
File: ./src/StakingManager.sol

280:         kHYPE.transferFrom(msg.sender, address(this), kHYPEAmount);

406:         kHYPE.transfer(treasury, kHYPEFee);

940:         kHYPE.transfer(user, kHYPEAmount + kHYPEFee);
```

## external call

*Instances (17)*:
```solidity
File: ./src/OracleManager.sol

102:         authorizedOracles.add(adapter);

109:         authorizedOracles.remove(adapter);
```

```solidity
File: ./src/PauserRegistry.sol

80:          _authorizedContracts.add(contracts[i]);

135:         _authorizedContracts.add(contractAddress);

146:         _authorizedContracts.remove(contractAddress);
```

```solidity
File: ./src/StakingAccountant.sol

87:          _authorizedManagers.set(manager, kHYPEToken);

90:          _uniqueTokens.add(kHYPEToken);

104:         _authorizedManagers.remove(manager);

111:         (, address otherToken) = _authorizedManagers.at(i);

120:          _uniqueTokens.remove(token);

```
```solidity
File: ./src/StakingManager.sol

847:          _whitelist.add(accounts[i]);
```
```solidity
File: ./src/ValidatorManager.sol

146:          _validatorIndexes.set(validator, _validators.length - 1);

231:          (bool exists /* uint256 index */, ) = _validatorIndexes.tryGet(validator);

239:          _validatorsWithPendingRebalance.add(validator);

271:          _validatorsWithPendingRebalance.remove(validator);

395:          (validator, ) = _validatorIndexes.at(index);
```
```solidity
File: ./src/oracles/OracleAdapter.sol

42:          return IDefaultOracle(defaultOracle).getValidatorMetrics(validator);

```

###  [QA-2] Local variable shadowing in `KHYPE:initialize`

## Description

Detection of shadowing using local variables.

*Instances (4)*:

KHYPE.initialize(string,string,address,address,address,address).name (src/KHYPE.sol#49) shadows:
        - ERC20Upgradeable.name() (lib/openzeppelin-contracts-upgradeable/contracts/token/ERC20/ERC20Upgradeable.sol#71-74) (function)
        - IERC20Metadata.name() (lib/openzeppelin-contracts/contracts/token/ERC20/extensions/IERC20Metadata.sol#15) (function)

KHYPE.initialize(string,string,address,address,address,address).symbol (src/KHYPE.sol#50) shadows:
        - ERC20Upgradeable.symbol() (lib/openzeppelin-contracts-upgradeable/contracts/token/ERC20/ERC20Upgradeable.sol#80-83) (function)
        - IERC20Metadata.symbol() (lib/openzeppelin-contracts/contracts/token/ERC20/extensions/IERC20Metadata.sol#20) (function)

```solidity
function initialize(
@>      string calldata name,
@>      string calldata symbol,
        address admin,
        address minter,
        address burner,
        address _pauserRegistry
    ) public initializer {...}
```
###  [QA-3] Loop contains `require`/`revert` statements

Avoid `require` / `revert` statements in a loop because a single bad item can cause the whole transaction to fail. It's better to forgive on fail and return failed elements post processing of the loop

*Instances (7)*:

- Found in src/StakingManager.sol [Line: 346](src/StakingManager.sol#L346)

	```solidity
	        for (uint256 i = 0; i < validators.length; ) {
	```

- Found in src/StakingManager.sol [Line: 567](src/StakingManager.sol#L567)

	```solidity
	        for (uint256 i = 0; i < validators.length; ) {
	```

- Found in src/StakingManager.sol [Line: 688](src/StakingManager.sol#L688)

	```solidity
	        for (uint256 i = _withdrawalProcessingIndex; i < endIndex; i++) {
	```

- Found in src/StakingManager.sol [Line: 736](src/StakingManager.sol#L736)

	```solidity
	        for (uint256 i = _depositProcessingIndex; i < endIndex; i++) {
	```

- Found in src/StakingManager.sol [Line: 840](src/StakingManager.sol#L840)

	```solidity
	        for (uint256 i = 0; i < accounts.length; i++) {
	```

- Found in src/ValidatorManager.sol [Line: 207](src/ValidatorManager.sol#L207)

	```solidity
	        for (uint256 i = 0; i < validators.length; ) {
	```

- Found in src/ValidatorManager.sol [Line: 259](src/ValidatorManager.sol#L259)

	```solidity
	        for (uint256 i = 0; i < validators.length; ) {
	```
###  [QA-4] Large literal values multiples of 10000 can be replaced with scientific notation
Use `e` notation, for example: `1e18`, instead of its full numeric value.
*Instances (4)*:
- Found in src/StakingManager.sol [Line: 50](src/StakingManager.sol#L50)

	```solidity
	    uint256 public constant BASIS_POINTS = 10000; // 100% in basis points
	```

- Found in src/ValidatorManager.sol [Line: 38](src/ValidatorManager.sol#L38)

	```solidity
	    uint256 public constant BASIS_POINTS = 10000;
	```

- Found in src/oracles/DefaultOracle.sol [Line: 29](src/oracles/DefaultOracle.sol#L29)

	```solidity
	    uint256 public constant BASIS_POINTS = 10000; // 100% in basis points
	```

- Found in src/validators/ValidatorSanityChecker.sol [Line: 17](src/validators/ValidatorSanityChecker.sol#L17)

	```solidity
	    uint256 public constant BASIS_POINTS = 10000; // 100% in basis points
	```
###  [QA-5] Modifiers invoked only once can be shoe-horned into the function
*Instances (2)*:
- Found in src/StakingManager.sol [Line: 112](src/StakingManager.sol#L112)

	```solidity
	    modifier whenStakingNotPaused() {
	```
- Found in src/StakingManager.sol [Line: 117](src/StakingManager.sol#L117)

	```solidity
	    modifier whenWithdrawalNotPaused() {
	```

###  [QA-6] The `nonReentrant` `modifier` should occur before all other modifiers

This is a best-practice to protect against reentrancy in other modifiers.

*Instances (4)*:


- Found in src/ValidatorManager.sol [Line: 155](src/ValidatorManager.sol#L155)

	```solidity
	    function deactivateValidator(address validator) external whenNotPaused nonReentrant validatorExists(validator) {
	```

- Found in src/ValidatorManager.sol [Line: 177](src/ValidatorManager.sol#L177)

	```solidity
	    ) external whenNotPaused nonReentrant onlyRole(MANAGER_ROLE) validatorExists(validator) {
	```

- Found in src/ValidatorManager.sol [Line: 203](src/ValidatorManager.sol#L203)

	```solidity
	    ) external whenNotPaused nonReentrant onlyRole(MANAGER_ROLE) {
	```

- Found in src/ValidatorManager.sol [Line: 253](src/ValidatorManager.sol#L253)

	```solidity
	    ) external whenNotPaused nonReentrant onlyRole(MANAGER_ROLE) {
	```

###  [QA-7] Event is missing `indexed` fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.


*Instances (53)*:

- Found in src/interfaces/IOracleManager.sol [Line: 13](src/interfaces/IOracleManager.sol#L13)

	```solidity
	    event OracleActiveStateChanged(address indexed oracle, bool active);
	```

- Found in src/interfaces/IOracleManager.sol [Line: 14](src/interfaces/IOracleManager.sol#L14)

	```solidity
	    event MaxPerformanceBoundUpdated(uint256 newBound);
	```

- Found in src/interfaces/IOracleManager.sol [Line: 15](src/interfaces/IOracleManager.sol#L15)

	```solidity
	    event PerformanceUpdated(address indexed validator, uint256 timestamp);
	```

- Found in src/interfaces/IOracleManager.sol [Line: 17](src/interfaces/IOracleManager.sol#L17)

	```solidity
	    event ValidatorBehaviorCheckFailed(address indexed validator, string reason);
	```

- Found in src/interfaces/IOracleManager.sol [Line: 19](src/interfaces/IOracleManager.sol#L19)

	```solidity
	    event MaxOracleStalenessUpdated(uint256);
	```

- Found in src/interfaces/IOracleManager.sol [Line: 20](src/interfaces/IOracleManager.sol#L20)

	```solidity
	    event OracleDataStale(address indexed oracle, address indexed validator, uint256 timestamp, uint256 currentTime);
	```

- Found in src/interfaces/IOracleManager.sol [Line: 23](src/interfaces/IOracleManager.sol#L23)

	```solidity
	    event MinValidOraclesUpdated(uint256 newMinimum);
	```

- Found in src/interfaces/IStakingAccountant.sol [Line: 8](src/interfaces/IStakingAccountant.sol#L8)

	```solidity
	    event StakeRecorded(address indexed manager, uint256 amount);
	```

- Found in src/interfaces/IStakingAccountant.sol [Line: 9](src/interfaces/IStakingAccountant.sol#L9)

	```solidity
	    event ClaimRecorded(address indexed manager, uint256 amount);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 32](src/interfaces/IStakingManager.sol#L32)

	```solidity
	    event StakeReceived(address indexed staking, address indexed staker, uint256 amount);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 41](src/interfaces/IStakingManager.sol#L41)

	```solidity
	    event WithdrawalConfirmed(address indexed user, uint256 indexed withdrawalId, uint256 amount);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 42](src/interfaces/IStakingManager.sol#L42)

	```solidity
	    event WithdrawalCancelled(
	```

- Found in src/interfaces/IStakingManager.sol [Line: 48](src/interfaces/IStakingManager.sol#L48)

	```solidity
	    event StakingLimitUpdated(uint256 newStakingLimit);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 49](src/interfaces/IStakingManager.sol#L49)

	```solidity
	    event MinStakeAmountUpdated(uint256 newMinStakeAmount);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 50](src/interfaces/IStakingManager.sol#L50)

	```solidity
	    event MaxStakeAmountUpdated(uint256 newMaxStakeAmount);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 51](src/interfaces/IStakingManager.sol#L51)

	```solidity
	    event WithdrawalDelayUpdated(uint256 newDelay);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 52](src/interfaces/IStakingManager.sol#L52)

	```solidity
	    event Delegate(address indexed staking, address indexed validator, uint256 amount);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 53](src/interfaces/IStakingManager.sol#L53)

	```solidity
	    event TargetBufferUpdated(uint256 newTargetBuffer);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 54](src/interfaces/IStakingManager.sol#L54)

	```solidity
	    event BufferIncreased(uint256 amountAdded, uint256 newTotal);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 55](src/interfaces/IStakingManager.sol#L55)

	```solidity
	    event BufferDecreased(uint256 amountRemoved, uint256 newTotal);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 56](src/interfaces/IStakingManager.sol#L56)

	```solidity
	    event ValidatorWithdrawal(address indexed staking, address indexed validator, uint256 amount);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 66](src/interfaces/IStakingManager.sol#L66)

	```solidity
	    event WithdrawalRedelegated(uint256 amount);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 67](src/interfaces/IStakingManager.sol#L67)

	```solidity
	    event UnstakeFeeRateUpdated(uint256 newRate);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 71](src/interfaces/IStakingManager.sol#L71)

	```solidity
	    event L1DelegationQueued(
	```

- Found in src/interfaces/IStakingManager.sol [Line: 77](src/interfaces/IStakingManager.sol#L77)

	```solidity
	    event L1DelegationProcessed(
	```

- Found in src/interfaces/IStakingManager.sol [Line: 83](src/interfaces/IStakingManager.sol#L83)

	```solidity
	    event SpotWithdrawn(uint256 amount);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 91](src/interfaces/IStakingManager.sol#L91)

	```solidity
	    event TokenWithdrawnFromSpot(uint64 indexed tokenId, uint64 amount, address indexed recipient);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 98](src/interfaces/IStakingManager.sol#L98)

	```solidity
	    event TokenRescued(address indexed token, uint256 amount, address indexed recipient);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 105](src/interfaces/IStakingManager.sol#L105)

	```solidity
	    event L1OperationsQueued(address[] validators, uint256[] amounts, OperationType[] operationTypes);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 111](src/interfaces/IStakingManager.sol#L111)

	```solidity
	    event L1OperationsBatchProcessed(uint256 processedCount, uint256 remainingCount);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 116](src/interfaces/IStakingManager.sol#L116)

	```solidity
	    event L1OperationsQueueReset(uint256 queueLength);
	```

- Found in src/interfaces/IStakingManager.sol [Line: 122](src/interfaces/IStakingManager.sol#L122)

	```solidity
	    event EmergencyWithdrawalExecuted(address indexed validator, uint256 amount);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 39](src/interfaces/IValidatorManager.sol#L39)

	```solidity
	    event EmergencyWithdrawalLimitUpdated(uint256 newLimit);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 42](src/interfaces/IValidatorManager.sol#L42)

	```solidity
	    event ValidatorScoreUpdated(address indexed validator, uint256 totalScore);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 43](src/interfaces/IValidatorManager.sol#L43)

	```solidity
	    event StakeRebalanced(address indexed validator, uint256 newStake);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 44](src/interfaces/IValidatorManager.sol#L44)

	```solidity
	    event StakeLimitsUpdated(uint256 minLimit, uint256 maxLimit);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 45](src/interfaces/IValidatorManager.sol#L45)

	```solidity
	    event EmergencyWithdrawalRequested(address indexed validator, uint256 amount);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 46](src/interfaces/IValidatorManager.sol#L46)

	```solidity
	    event EmergencyWithdrawalProcessed(address indexed validator, uint256 amount);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 47](src/interfaces/IValidatorManager.sol#L47)

	```solidity
	    event SlashingEventReported(address indexed validator, uint256 amount);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 48](src/interfaces/IValidatorManager.sol#L48)

	```solidity
	    event PerformanceReportGenerated(address indexed validator, uint256 timestamp);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 49](src/interfaces/IValidatorManager.sol#L49)

	```solidity
	    event RebalanceRequestAdded(address indexed validator, uint256 amount);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 50](src/interfaces/IValidatorManager.sol#L50)

	```solidity
	    event RebalanceBatchProcessed(uint256 startIndex, uint256 endIndex, uint256 timestamp);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 51](src/interfaces/IValidatorManager.sol#L51)

	```solidity
	    event RebalanceRequestClosed(address indexed validator, uint256 amount);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 52](src/interfaces/IValidatorManager.sol#L52)

	```solidity
	    event AllRebalanceRequestsClosed(uint256 count, uint256 timestamp);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 53](src/interfaces/IValidatorManager.sol#L53)

	```solidity
	    event EmergencyCooldownUpdated(uint256 cooldown);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 54](src/interfaces/IValidatorManager.sol#L54)

	```solidity
	    event RebalanceCooldownUpdated(uint256 cooldown);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 58](src/interfaces/IValidatorManager.sol#L58)

	```solidity
	    event ValidatorPerformanceUpdated(address indexed validator, uint256 timestamp, uint256 blockNumber);
	```

- Found in src/interfaces/IValidatorManager.sol [Line: 59](src/interfaces/IValidatorManager.sol#L59)

	```solidity
	    event RewardEventReported(address indexed validator, uint256 amount);
	```

- Found in src/oracles/DefaultOracle.sol [Line: 47](src/oracles/DefaultOracle.sol#L47)

	```solidity
	    event MetricsUpdated(
	```

- Found in src/validators/IValidatorSanityChecker.sol [Line: 11](src/validators/IValidatorSanityChecker.sol#L11)

	```solidity
	    event SlashingToleranceUpdated(uint256 newTolerance);
	```

- Found in src/validators/IValidatorSanityChecker.sol [Line: 12](src/validators/IValidatorSanityChecker.sol#L12)

	```solidity
	    event RewardsToleranceUpdated(uint256 newTolerance);
	```

- Found in src/validators/IValidatorSanityChecker.sol [Line: 13](src/validators/IValidatorSanityChecker.sol#L13)

	```solidity
	    event ScoreToleranceUpdated(uint256 newTolerance);
	```

- Found in src/validators/IValidatorSanityChecker.sol [Line: 14](src/validators/IValidatorSanityChecker.sol#L14)

	```solidity
	    event MaxScoreBoundUpdated(uint256 newBound);
	```
###  [QA-8] `public` functions not used internally could be marked `external`
Instead of marking a function as `public`, consider marking it as `external` if it is not used internally.
*Instances (14):*
- Found in src/KHYPE.sol [Line: 48](src/KHYPE.sol#L48)

	```solidity
	    function initialize(
	```

- Found in src/KHYPE.sol [Line: 107](src/KHYPE.sol#L107)

	```solidity
	    function supportsInterface(
	```

- Found in src/OracleManager.sol [Line: 67](src/OracleManager.sol#L67)

	```solidity
	    function initialize(
	```

- Found in src/OracleManager.sol [Line: 120](src/OracleManager.sol#L120)

	```solidity
	    function isAuthorizedOracle(address adapter) public view returns (bool) {
	```

- Found in src/OracleManager.sol [Line: 124](src/OracleManager.sol#L124)

	```solidity
	    function isActiveOracle(address adapter) public view returns (bool) {
	```

- Found in src/PauserRegistry.sol [Line: 49](src/PauserRegistry.sol#L49)

	```solidity
	    function initialize(
	```

- Found in src/StakingAccountant.sol [Line: 62](src/StakingAccountant.sol#L62)

	```solidity
	    function initialize(address admin, address manager, address _validatorManager) public initializer {
	```

- Found in src/StakingAccountant.sol [Line: 181](src/StakingAccountant.sol#L181)

	```solidity
	    function kHYPEToHYPE(uint256 kHYPEAmount) public view override returns (uint256) {
	```

- Found in src/StakingAccountant.sol [Line: 185](src/StakingAccountant.sol#L185)

	```solidity
	    function HYPEToKHYPE(uint256 HYPEAmount) public view override returns (uint256) {
	```

- Found in src/StakingManager.sol [Line: 139](src/StakingManager.sol#L139)

	```solidity
	    function initialize(
	```

- Found in src/ValidatorManager.sol [Line: 387](src/ValidatorManager.sol#L387)

	```solidity
	    function validatorCount() public view returns (uint256) {
	```

- Found in src/ValidatorManager.sol [Line: 391](src/ValidatorManager.sol#L391)

	```solidity
	    function validatorAt(uint256 index) public view returns (address validator, Validator memory data) {
	```

- Found in src/ValidatorManager.sol [Line: 400](src/ValidatorManager.sol#L400)

	```solidity
	    function validatorInfo(address validator) public view validatorExists(validator) returns (Validator memory) {
	```

- Found in src/oracles/DefaultAdapter.sol [Line: 55](src/oracles/DefaultAdapter.sol#L55)

	```solidity
	    function supportsInterface(bytes4 interfaceId) public pure override returns (bool) {
	```

### [QA-9] Missing checks for `address(0)` when assigning values to address state variables

Check for `address(0)` when assigning values to address state variables.

*Instances (1):*

- Found in src/OracleManager.sol [Line: 310](src/OracleManager.sol#L310)

	```solidity
	        sanityChecker = IValidatorSanityChecker(newChecker);
	```

### [QA-10] Unsafe ERC20 Operations should not be used

ERC20 functions may not behave as expected. For example: return values are not always meaningful. It is recommended to use OpenZeppelin's SafeERC20 library.

*Instances (3):*

- Found in src/StakingManager.sol [Line: 277](src/StakingManager.sol#L277)

	```solidity
	        kHYPE.transferFrom(msg.sender, address(this), kHYPEAmount);
	```

- Found in src/StakingManager.sol [Line: 402](src/StakingManager.sol#L402)

	```solidity
	        kHYPE.transfer(treasury, kHYPEFee);
	```

- Found in src/StakingManager.sol [Line: 935](src/StakingManager.sol#L935)

	```solidity
	        kHYPE.transfer(user, kHYPEAmount + kHYPEFee);
	```

### [QA-11] Centralization Risk for trusted owners

Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (52):*

- Found in src/KHYPE.sol [Line: 87](src/KHYPE.sol#L87)

	```solidity
	    function mint(address to, uint256 amount) external onlyRole(MINTER_ROLE) {
	```

- Found in src/KHYPE.sol [Line: 96](src/KHYPE.sol#L96)

	```solidity
	    function burn(address from, uint256 amount) external onlyRole(BURNER_ROLE) {
	```

- Found in src/OracleManager.sol [Line: 98](src/OracleManager.sol#L98)

	```solidity
	    function authorizeOracleAdapter(address adapter) external whenNotPaused onlyRole(MANAGER_ROLE) {
	```

- Found in src/OracleManager.sol [Line: 108](src/OracleManager.sol#L108)

	```solidity
	    function deauthorizeOracle(address adapter) external whenNotPaused onlyRole(MANAGER_ROLE) {
	```

- Found in src/OracleManager.sol [Line: 114](src/OracleManager.sol#L114)

	```solidity
	    function setOracleActive(address adapter, bool active) external whenNotPaused onlyRole(MANAGER_ROLE) {
	```

- Found in src/OracleManager.sol [Line: 142](src/OracleManager.sol#L142)

	```solidity
	    function generatePerformance(address validator) external whenNotPaused onlyRole(OPERATOR_ROLE) returns (bool) {
	```

- Found in src/OracleManager.sol [Line: 290](src/OracleManager.sol#L290)

	```solidity
	    function setMaxPerformanceBound(uint256 newBound) external onlyRole(OPERATOR_ROLE) {
	```

- Found in src/OracleManager.sol [Line: 296](src/OracleManager.sol#L296)

	```solidity
	    function setMinUpdateInterval(uint256 newInterval) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/OracleManager.sol [Line: 302](src/OracleManager.sol#L302)

	```solidity
	    function setMaxOracleStaleness(uint256 newStaleness) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/OracleManager.sol [Line: 309](src/OracleManager.sol#L309)

	```solidity
	    function setSanityChecker(address newChecker) external onlyRole(DEFAULT_ADMIN_ROLE) {
	```

- Found in src/OracleManager.sol [Line: 314](src/OracleManager.sol#L314)

	```solidity
	    function setMinValidOracles(uint256 newMinimum) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/PauserRegistry.sol [Line: 88](src/PauserRegistry.sol#L88)

	```solidity
	    function pauseContract(address contractAddress) external onlyRole(PAUSER_ROLE) {
	```

- Found in src/PauserRegistry.sol [Line: 100](src/PauserRegistry.sol#L100)

	```solidity
	    function unpauseContract(address contractAddress) external onlyRole(UNPAUSER_ROLE) {
	```

- Found in src/PauserRegistry.sol [Line: 111](src/PauserRegistry.sol#L111)

	```solidity
	    function emergencyPauseAll() external onlyRole(PAUSE_ALL_ROLE) {
	```

- Found in src/PauserRegistry.sol [Line: 129](src/PauserRegistry.sol#L129)

	```solidity
	    function authorizeContract(address contractAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {
	```

- Found in src/PauserRegistry.sol [Line: 141](src/PauserRegistry.sol#L141)

	```solidity
	    function deauthorizeContract(address contractAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {
	```

- Found in src/StakingAccountant.sol [Line: 82](src/StakingAccountant.sol#L82)

	```solidity
	    function authorizeStakingManager(address manager, address kHYPEToken) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingAccountant.sol [Line: 99](src/StakingAccountant.sol#L99)

	```solidity
	    function deauthorizeStakingManager(address manager) external override onlyRole(DEFAULT_ADMIN_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 561](src/StakingManager.sol#L561)

	```solidity
	    ) external nonReentrant whenNotPaused onlyRole(OPERATOR_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 627](src/StakingManager.sol#L627)

	```solidity
	    function processL1Operations(uint256 batchSize) public onlyRole(OPERATOR_ROLE) whenNotPaused {
	```

- Found in src/StakingManager.sol [Line: 773](src/StakingManager.sol#L773)

	```solidity
	    function setTargetBuffer(uint256 newTargetBuffer) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 778](src/StakingManager.sol#L778)

	```solidity
	    function setStakingLimit(uint256 newStakingLimit) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 791](src/StakingManager.sol#L791)

	```solidity
	    function setMinStakeAmount(uint256 newMinStakeAmount) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 799](src/StakingManager.sol#L799)

	```solidity
	    function setMaxStakeAmount(uint256 newMaxStakeAmount) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 814](src/StakingManager.sol#L814)

	```solidity
	    function setWithdrawalDelay(uint256 newDelay) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 822](src/StakingManager.sol#L822)

	```solidity
	    function enableWhitelist() external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 830](src/StakingManager.sol#L830)

	```solidity
	    function disableWhitelist() external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 839](src/StakingManager.sol#L839)

	```solidity
	    function addToWhitelist(address[] calldata accounts) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 852](src/StakingManager.sol#L852)

	```solidity
	    function removeFromWhitelist(address[] calldata accounts) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 885](src/StakingManager.sol#L885)

	```solidity
	    function pauseStaking() external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 893](src/StakingManager.sol#L893)

	```solidity
	    function unpauseStaking() external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 901](src/StakingManager.sol#L901)

	```solidity
	    function pauseWithdrawal() external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 909](src/StakingManager.sol#L909)

	```solidity
	    function unpauseWithdrawal() external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 919](src/StakingManager.sol#L919)

	```solidity
	    function cancelWithdrawal(address user, uint256 withdrawalId) external onlyRole(MANAGER_ROLE) whenNotPaused {
	```

- Found in src/StakingManager.sol [Line: 946](src/StakingManager.sol#L946)

	```solidity
	    function redelegateWithdrawnHYPE() external onlyRole(MANAGER_ROLE) whenNotPaused {
	```

- Found in src/StakingManager.sol [Line: 963](src/StakingManager.sol#L963)

	```solidity
	    function resetL1OperationsQueue() external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 979](src/StakingManager.sol#L979)

	```solidity
	    function setUnstakeFeeRate(uint256 newRate) external onlyRole(MANAGER_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 986](src/StakingManager.sol#L986)

	```solidity
	    function setTreasury(address newTreasury) external onlyRole(DEFAULT_ADMIN_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 994](src/StakingManager.sol#L994)

	```solidity
	    function withdrawFromSpot(uint64 amount) external onlyRole(OPERATOR_ROLE) {
	```

- Found in src/StakingManager.sol [Line: 1007](src/StakingManager.sol#L1007)

	```solidity
	    function withdrawTokenFromSpot(uint64 tokenId, uint64 amount) external onlyRole(TREASURY_ROLE) whenNotPaused {
	```

- Found in src/StakingManager.sol [Line: 1028](src/StakingManager.sol#L1028)

	```solidity
	    function rescueToken(address token, uint256 amount) external onlyRole(TREASURY_ROLE) whenNotPaused {
	```

- Found in src/StakingManager.sol [Line: 1049](src/StakingManager.sol#L1049)

	```solidity
	    ) external nonReentrant whenNotPaused onlyRole(SENTINEL_ROLE) {
	```

- Found in src/ValidatorManager.sol [Line: 131](src/ValidatorManager.sol#L131)

	```solidity
	    function activateValidator(address validator) external whenNotPaused onlyRole(MANAGER_ROLE) {
	```

- Found in src/ValidatorManager.sol [Line: 177](src/ValidatorManager.sol#L177)

	```solidity
	    ) external whenNotPaused nonReentrant onlyRole(MANAGER_ROLE) validatorExists(validator) {
	```

- Found in src/ValidatorManager.sol [Line: 203](src/ValidatorManager.sol#L203)

	```solidity
	    ) external whenNotPaused nonReentrant onlyRole(MANAGER_ROLE) {
	```

- Found in src/ValidatorManager.sol [Line: 253](src/ValidatorManager.sol#L253)

	```solidity
	    ) external whenNotPaused nonReentrant onlyRole(MANAGER_ROLE) {
	```

- Found in src/ValidatorManager.sol [Line: 311](src/ValidatorManager.sol#L311)

	```solidity
	    ) external whenNotPaused onlyRole(ORACLE_MANAGER_ROLE) validatorActive(validator) {
	```

- Found in src/ValidatorManager.sol [Line: 412](src/ValidatorManager.sol#L412)

	```solidity
	    ) external onlyRole(ORACLE_MANAGER_ROLE) validatorActive(validator) {
	```

- Found in src/ValidatorManager.sol [Line: 430](src/ValidatorManager.sol#L430)

	```solidity
	    ) external onlyRole(ORACLE_MANAGER_ROLE) validatorActive(validator) {
	```

- Found in src/ValidatorManager.sol [Line: 450](src/ValidatorManager.sol#L450)

	```solidity
	    ) external whenNotPaused onlyRole(MANAGER_ROLE) validatorActive(validator) {
	```

- Found in src/oracles/DefaultOracle.sol [Line: 11](src/oracles/DefaultOracle.sol#L11)

	```solidity
	contract DefaultOracle is AccessControl {
	```

- Found in src/oracles/DefaultOracle.sol [Line: 94](src/oracles/DefaultOracle.sol#L94)

	```solidity
	    ) external onlyRole(OPERATOR_ROLE) {
	```
