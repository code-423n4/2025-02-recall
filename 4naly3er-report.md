# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings) | 19 |
| [GAS-2](#GAS-2) | Use assembly to check for `address(0)` | 33 |
| [GAS-3](#GAS-3) | Using bools for storage incurs overhead | 1 |
| [GAS-4](#GAS-4) | Cache array length outside of loop | 2 |
| [GAS-5](#GAS-5) | Use calldata instead of memory for function arguments that do not get mutated | 17 |
| [GAS-6](#GAS-6) | For Operations that will not overflow, you could use unchecked | 435 |
| [GAS-7](#GAS-7) | Use Custom Errors instead of Revert Strings to save Gas | 9 |
| [GAS-8](#GAS-8) | Avoid contract existence checks by using low level calls | 7 |
| [GAS-9](#GAS-9) | Functions guaranteed to revert when called by normal users can be marked `payable` | 3 |
| [GAS-10](#GAS-10) | `++i` costs less gas compared to `i++` or `i += 1` (same for `--i` vs `i--` or `i -= 1`) | 5 |
| [GAS-11](#GAS-11) | Using `private` rather than `public` for constants, saves gas | 5 |
| [GAS-12](#GAS-12) | Use shift right/left instead of division/multiplication if possible | 2 |
| [GAS-13](#GAS-13) | Increments/decrements can be unchecked in for-loops | 8 |
| [GAS-14](#GAS-14) | Use != 0 instead of > 0 for unsigned integer comparison | 6 |
| [GAS-15](#GAS-15) | `internal` functions not called by the contract should be removed | 133 |
### <a name="GAS-1"></a>[GAS-1] `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings)
This saves **16 gas per instance.**

*Instances (19)*:
```solidity
File: contracts/gateway/GatewayManagerFacet.sol

55:         s.totalSubnets += 1;

81:         subnet.stake += amount;

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

138:                 totalValue += msgs[i].value;

```

```solidity
File: contracts/lib/LibGateway.sol

216:             totalValidatorsWeight += self.validators[i].weight;

252:         subnet.circSupply += crossMessage.value;

269:         s.bottomUpNonce += 1;

433:             subnet.appliedBottomUpNonce += 1;

448:             s.appliedTopDownNonce += 1;

```

```solidity
File: contracts/lib/LibQuorum.sol

60:         info.currentWeight += weight;

```

```solidity
File: contracts/lib/LibStaking.sol

46:             amount += release.amount;

164:             collateral += getPower(validators, validator);

176:             collateral += getConfirmedCollateral(validators, validator);

181:         collateral += getTotalConfirmedCollateral(validators);

220:         validators.validators[validator].totalCollateral += amount;

252:         self.totalConfirmedCollateral += amount;

```

```solidity
File: contracts/lib/LibSubnetActor.sol

165:         msgValue += s.supplySource.makeAvailable(s.ipcGatewayAddr, genesisCircSupply);

166:         msgValue += s.collateralSource.makeAvailable(s.ipcGatewayAddr, collateral);

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

48:         s.genesisBalance[msg.sender] += amount;

49:         s.genesisCircSupply += amount;

```

### <a name="GAS-2"></a>[GAS-2] Use assembly to check for `address(0)`
*Saves 6 gas per instance*

*Instances (33)*:
```solidity
File: contracts/GatewayDiamond.sol

91:         if (facet == address(0)) {

```

```solidity
File: contracts/SubnetActorDiamond.sol

46:         if (params.ipcGatewayAddr == address(0)) {

105:         if (params.validatorGater != address(0)) {

109:         if (params.validatorRewarder != address(0)) {

124:         if (facet == address(0)) {

```

```solidity
File: contracts/SubnetRegistryDiamond.sol

42:         if (params.gateway == address(0)) {

45:         if (params.getterFacet == address(0)) {

48:         if (params.managerFacet == address(0)) {

51:         if (params.rewarderFacet == address(0)) {

54:         if (params.checkpointerFacet == address(0)) {

57:         if (params.pauserFacet == address(0)) {

60:         if (params.diamondCutFacet == address(0)) {

63:         if (params.diamondLoupeFacet == address(0)) {

66:         if (params.ownershipFacet == address(0)) {

69:         if (params.activityFacet == address(0)) {

116:         if (facet == address(0)) {

```

```solidity
File: contracts/lib/AssetHelper.sol

26:             require(asset.tokenAddress != address(0), "Invalid ERC20 address");

```

```solidity
File: contracts/lib/LibActivity.sol

137:         if (Consensus.MerkleHash.unwrap(commitment) == bytes32(0)) {

```

```solidity
File: contracts/lib/LibDiamond.sol

49:         if (newOwner == address(0)) {

114:         if (_facetAddress == address(0)) {

124:             if (oldFacetAddress != address(0)) {

141:         if (_facetAddress == address(0)) {

156:             if (oldFacetAddress == address(0)) {

170:         if (_facetAddress != address(0)) {

178:             if (oldFacetAddressAndSelectorPosition.facetAddress == address(0)) {

204:         if (_init == address(0)) {

```

```solidity
File: contracts/lib/LibGateway.sol

325:         if (actor == address(0)) {

```

```solidity
File: contracts/lib/LibSubnetActor.sol

51:         if (s.validatorGater == address(0)) {

64:         if (s.validatorGater == address(0)) {

```

```solidity
File: contracts/subnetregistry/SubnetGetterFacet.sol

20:         if (subnet == address(0)) {

33:         if (subnet == address(0)) {

117:         if (newGetterFacet == address(0)) {

120:         if (newManagerFacet == address(0)) {

```

### <a name="GAS-3"></a>[GAS-3] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (1)*:
```solidity
File: contracts/lib/LibDiamond.sol

39:         mapping(bytes4 => bool) supportedInterfaces;

```

### <a name="GAS-4"></a>[GAS-4] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (2)*:
```solidity
File: contracts/lib/LibActivity.sol

66:         for (uint256 i = 0; i < proof.length; i++) {

110:             for (uint256 j = 0; j < heights.length; j++) {

```

### <a name="GAS-5"></a>[GAS-5] Use calldata instead of memory for function arguments that do not get mutated
When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. Each iteration of this for-loop costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly bypasses this loop. 

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gas-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one. 

 *Saves 60 gas per instance*

*Instances (17)*:
```solidity
File: contracts/gateway/GatewayGetterFacet.sol

102:     function getSubnetTopDownMsgsLength(SubnetID memory subnetId) external view returns (uint256) {

```

```solidity
File: contracts/gateway/GatewayMessengerFacet.sol

41:         IpcEnvelope memory envelope

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

108:         bytes32[] memory membershipProof,

110:         bytes memory signature

```

```solidity
File: contracts/interfaces/IValidatorGater.sol

13:     function interceptPowerDelta(SubnetID memory id, address validator, uint256 prevPower, uint256 newPower) external;

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

30:         IPCAddress memory from,

31:         IPCAddress memory to,

47:         IPCAddress memory from,

48:         IPCAddress memory to,

51:         bytes memory params

72:         bytes memory ret

143:     function toHash(IpcEnvelope[] memory crossMsgs) public pure returns (bytes32) {

183:         Asset memory supplySource

```

```solidity
File: contracts/lib/SubnetIDHelper.sol

16:     function getAddress(SubnetID memory subnet) public pure returns (address) {

25:     function getParentSubnet(SubnetID memory subnet) public pure returns (SubnetID memory) {

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

65:         address[] memory signatories,

67:         bytes[] memory signatures

```

### <a name="GAS-6"></a>[GAS-6] For Operations that will not overflow, you could use unchecked

*Instances (435)*:
```solidity
File: contracts/GatewayDiamond.sol

4: import {GatewayActorStorage} from "./lib/LibGatewayActorStorage.sol";

5: import {IDiamond} from "./interfaces/IDiamond.sol";

6: import {IDiamondCut} from "./interfaces/IDiamondCut.sol";

7: import {IDiamondLoupe} from "./interfaces/IDiamondLoupe.sol";

8: import {IERC165} from "./interfaces/IERC165.sol";

9: import {Validator, Membership} from "./structs/Subnet.sol";

10: import {InvalidCollateral, InvalidSubmissionPeriod, InvalidMajorityPercentage} from "./errors/IPCErrors.sol";

11: import {LibDiamond} from "./lib/LibDiamond.sol";

12: import {LibGateway} from "./lib/LibGateway.sol";

13: import {SubnetID} from "./structs/Subnet.sol";

14: import {LibStaking} from "./lib/LibStaking.sol";

15: import {BATCH_PERIOD, MAX_MSGS_PER_BATCH} from "./structs/CrossNet.sol";

```

```solidity
File: contracts/OwnershipFacet.sol

4: import {LibDiamond} from "./lib/LibDiamond.sol";

```

```solidity
File: contracts/SubnetActorDiamond.sol

4: import {SubnetActorStorage} from "./lib/LibSubnetActorStorage.sol";

5: import {ConsensusType} from "./enums/ConsensusType.sol";

6: import {IDiamond} from "./interfaces/IDiamond.sol";

7: import {IDiamondCut} from "./interfaces/IDiamondCut.sol";

8: import {IDiamondLoupe} from "./interfaces/IDiamondLoupe.sol";

9: import {IERC165} from "./interfaces/IERC165.sol";

10: import {GatewayCannotBeZero, NotGateway, InvalidSubmissionPeriod, InvalidCollateral, InvalidMajorityPercentage, InvalidPowerScale} from "./errors/IPCErrors.sol";

11: import {BATCH_PERIOD, MAX_MSGS_PER_BATCH} from "./structs/CrossNet.sol";

12: import {LibDiamond} from "./lib/LibDiamond.sol";

13: import {PermissionMode, SubnetID, AssetKind, Asset} from "./structs/Subnet.sol";

14: import {SubnetIDHelper} from "./lib/SubnetIDHelper.sol";

15: import {LibStaking} from "./lib/LibStaking.sol";

16: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

17: import {AssetHelper} from "./lib/AssetHelper.sol";

18: import {LibActivity} from "./lib/LibActivity.sol";

```

```solidity
File: contracts/SubnetRegistryDiamond.sol

4: import {IDiamond} from "./interfaces/IDiamond.sol";

5: import {IDiamondCut} from "./interfaces/IDiamondCut.sol";

6: import {IDiamondLoupe} from "./interfaces/IDiamondLoupe.sol";

7: import {IERC165} from "./interfaces/IERC165.sol";

8: import {SubnetRegistryActorStorage} from "./lib/LibSubnetRegistryStorage.sol";

9: import {GatewayCannotBeZero, FacetCannotBeZero} from "./errors/IPCErrors.sol";

10: import {LibDiamond} from "./lib/LibDiamond.sol";

11: import {SubnetCreationPrivileges} from "./structs/Subnet.sol";

```

```solidity
File: contracts/diamond/DiamondCutFacet.sol

9: import {IDiamondCut} from "../interfaces/IDiamondCut.sol";

10: import {LibDiamond} from "../lib/LibDiamond.sol";

```

```solidity
File: contracts/diamond/DiamondLoupeFacet.sol

11: import {LibDiamond} from "../lib/LibDiamond.sol";

12: import {IDiamondLoupe} from "../interfaces/IDiamondLoupe.sol";

13: import {IERC165} from "../interfaces/IERC165.sol";

36:         for (uint256 selectorIndex; selectorIndex < selectorCount; ++selectorIndex) {

41:             for (uint256 facetIndex; facetIndex < numFacets; ++facetIndex) {

44:                     ++numFacetSelectors[facetIndex];

59:             ++numFacets;

70:                 ++facetIndex;

91:         for (uint256 selectorIndex; selectorIndex < selectorCount; ++selectorIndex) {

96:                 ++numSelectors;

115:         for (uint256 selectorIndex; selectorIndex < selectorCount; ++selectorIndex) {

120:             for (uint256 facetIndex; facetIndex < numFacets; ++facetIndex) {

133:             ++numFacets;

```

```solidity
File: contracts/gateway/GatewayGetterFacet.sol

4: import {BottomUpCheckpoint, BottomUpMsgBatch, IpcEnvelope, ParentFinality} from "../structs/CrossNet.sol";

5: import {QuorumInfo} from "../structs/Quorum.sol";

6: import {SubnetID, Subnet} from "../structs/Subnet.sol";

7: import {Membership} from "../structs/Subnet.sol";

8: import {LibGateway} from "../lib/LibGateway.sol";

9: import {LibStaking} from "../lib/LibStaking.sol";

10: import {LibQuorum} from "../lib/LibQuorum.sol";

11: import {GatewayActorStorage} from "../lib/LibGatewayActorStorage.sol";

12: import {SubnetIDHelper} from "../lib/SubnetIDHelper.sol";

13: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

160:                 ++i;

217:                 ++i;

```

```solidity
File: contracts/gateway/GatewayManagerFacet.sol

4: import {GatewayActorModifiers} from "../lib/LibGatewayActorStorage.sol";

5: import {SubnetActorGetterFacet} from "../subnet/SubnetActorGetterFacet.sol";

6: import {BURNT_FUNDS_ACTOR} from "../constants/Constants.sol";

7: import {IpcEnvelope} from "../structs/CrossNet.sol";

8: import {FvmAddress} from "../structs/FvmAddress.sol";

9: import {SubnetID, Subnet, Asset} from "../structs/Subnet.sol";

10: import {Membership, AssetKind} from "../structs/Subnet.sol";

11: import {AlreadyRegisteredSubnet, CannotReleaseZero, MethodNotAllowed, NotEnoughFunds, NotEnoughFundsToRelease, NotEnoughCollateral, NotEmptySubnetCircSupply, NotRegisteredSubnet, InvalidXnetMessage, InvalidXnetMessageReason} from "../errors/IPCErrors.sol";

12: import {LibGateway} from "../lib/LibGateway.sol";

13: import {SubnetIDHelper} from "../lib/SubnetIDHelper.sol";

14: import {CrossMsgHelper} from "../lib/CrossMsgHelper.sol";

15: import {FilAddress} from "fevmate/contracts/utils/FilAddress.sol";

16: import {ReentrancyGuard} from "../lib/LibReentrancyGuard.sol";

17: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

18: import {Address} from "@openzeppelin/contracts/utils/Address.sol";

19: import {AssetHelper} from "../lib/AssetHelper.sol";

20: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

38:         if (s.networkName.route.length + 1 >= s.maxTreeDepth) {

55:         s.totalSubnets += 1;

81:         subnet.stake += amount;

101:         subnet.stake -= amount;

125:         s.totalSubnets -= 1;

```

```solidity
File: contracts/gateway/GatewayMessengerFacet.sol

4: import {GatewayActorModifiers} from "../lib/LibGatewayActorStorage.sol";

5: import {IpcEnvelope, CallMsg, IpcMsgKind} from "../structs/CrossNet.sol";

6: import {IPCMsgType} from "../enums/IPCMsgType.sol";

7: import {Subnet, SubnetID, AssetKind, IPCAddress, Asset} from "../structs/Subnet.sol";

8: import {InvalidXnetMessage, InvalidXnetMessageReason, MethodNotAllowed} from "../errors/IPCErrors.sol";

9: import {SubnetIDHelper} from "../lib/SubnetIDHelper.sol";

10: import {LibGateway} from "../lib/LibGateway.sol";

11: import {FilAddress} from "fevmate/contracts/utils/FilAddress.sol";

12: import {AssetHelper} from "../lib/AssetHelper.sol";

13: import {CrossMsgHelper} from "../lib/CrossMsgHelper.sol";

14: import {FvmAddressHelper} from "../lib/FvmAddressHelper.sol";

15: import {ISubnetActor} from "../interfaces/ISubnetActor.sol";

17: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

19: string constant ERR_GENERAL_CROSS_MSG_DISABLED = "Support for general-purpose cross-net messages is disabled";

20: string constant ERR_MULTILEVEL_CROSS_MSG_DISABLED = "Support for multi-level cross-net messages is disabled";

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

4: import {GatewayActorModifiers} from "../../lib/LibGatewayActorStorage.sol";

5: import {BottomUpCheckpoint} from "../../structs/CrossNet.sol";

6: import {LibGateway} from "../../lib/LibGateway.sol";

7: import {LibQuorum} from "../../lib/LibQuorum.sol";

8: import {Subnet} from "../../structs/Subnet.sol";

9: import {QuorumObjKind} from "../../structs/Quorum.sol";

10: import {Address} from "@openzeppelin/contracts/utils/Address.sol";

12: import {InvalidBatchSource, NotEnoughBalance, MaxMsgsPerBatchExceeded, InvalidCheckpointSource, CheckpointAlreadyExists} from "../../errors/IPCErrors.sol";

13: import {NotRegisteredSubnet, SubnetNotActive, SubnetNotFound, InvalidSubnet, CheckpointNotCreated} from "../../errors/IPCErrors.sol";

14: import {BatchNotCreated, InvalidBatchEpoch, BatchAlreadyExists, NotEnoughSubnetCircSupply, InvalidCheckpointEpoch} from "../../errors/IPCErrors.sol";

16: import {CrossMsgHelper} from "../../lib/CrossMsgHelper.sol";

17: import {IpcEnvelope, SubnetID, IpcMsgKind} from "../../structs/CrossNet.sol";

18: import {SubnetIDHelper} from "../../lib/SubnetIDHelper.sol";

20: import {ActivityRollupRecorded, FullActivityRollup} from "../../structs/Activity.sol";

93:                 ++h;

138:                 totalValue += msgs[i].value;

141:                 ++i;

151:         subnet.circSupply -= totalAmount;

```

```solidity
File: contracts/gateway/router/TopDownFinalityFacet.sol

4: import {GatewayActorModifiers} from "../../lib/LibGatewayActorStorage.sol";

5: import {ParentFinality} from "../../structs/CrossNet.sol";

6: import {PermissionMode, Validator, ValidatorInfo, StakingChangeRequest, Membership} from "../../structs/Subnet.sol";

7: import {LibGateway} from "../../lib/LibGateway.sol";

9: import {FilAddress} from "fevmate/contracts/utils/FilAddress.sol";

11: import {ParentValidatorsTracker, ValidatorSet} from "../../structs/Subnet.sol";

12: import {LibValidatorTracking, LibValidatorSet} from "../../lib/LibStaking.sol";

51:         uint64 configurationNumber = s.validatorsTracker.changes.nextConfigurationNumber - 1;

55:             (configurationNumber + 1) == s.validatorsTracker.changes.startConfigurationNumber

72:             uint256 weight = info.confirmedCollateral + info.federatedPower;

76:                 ++i;

```

```solidity
File: contracts/gateway/router/XnetMessagingFacet.sol

4: import {GatewayActorModifiers} from "../../lib/LibGatewayActorStorage.sol";

5: import {IpcEnvelope, SubnetID} from "../../structs/CrossNet.sol";

6: import {LibGateway} from "../../lib/LibGateway.sol";

7: import {IPCMsgType} from "../../enums/IPCMsgType.sol";

8: import {SubnetActorGetterFacet} from "../../subnet/SubnetActorGetterFacet.sol";

9: import {Subnet} from "../../structs/Subnet.sol";

11: import {FilAddress} from "fevmate/contracts/utils/FilAddress.sol";

12: import {SubnetIDHelper} from "../../lib/SubnetIDHelper.sol";

13: import {CrossMsgHelper} from "../../lib/CrossMsgHelper.sol";

14: import {AssetHelper} from "../../lib/AssetHelper.sol";

15: import {Asset} from "../../structs/Subnet.sol";

17: import {NotRegisteredSubnet} from "../../errors/IPCErrors.sol";

```

```solidity
File: contracts/interfaces/IDiamondCut.sol

4: import {IDiamond} from "./IDiamond.sol";

```

```solidity
File: contracts/interfaces/IGateway.sol

4: import {BottomUpCheckpoint, BottomUpMsgBatch, IpcEnvelope, ParentFinality} from "../structs/CrossNet.sol";

5: import {FullActivityRollup} from "../structs/Activity.sol";

6: import {SubnetID} from "../structs/Subnet.sol";

7: import {FvmAddress} from "../structs/FvmAddress.sol";

```

```solidity
File: contracts/interfaces/ISubnetActor.sol

4: import {Asset} from "../structs/Subnet.sol";

```

```solidity
File: contracts/interfaces/IValidatorGater.sol

4: import {SubnetID} from "../structs/Subnet.sol";

```

```solidity
File: contracts/interfaces/IValidatorRewarder.sol

4: import {SubnetID} from "../structs/Subnet.sol";

5: import {Consensus} from "../structs/Activity.sol";

```

```solidity
File: contracts/lib/AccountHelper.sol

4: import {FilAddress} from "fevmate/contracts/utils/FilAddress.sol";

```

```solidity
File: contracts/lib/AssetHelper.sol

4: import {NotEnoughBalance, InvalidSubnetActor} from "../errors/IPCErrors.sol";

5: import {Asset, AssetKind} from "../structs/Subnet.sol";

6: import {EMPTY_BYTES} from "../constants/Constants.sol";

7: import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

8: import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

9: import {ISubnetActor} from "../interfaces/ISubnetActor.sol";

57:             return finalBalance - initialBalance;

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

4: import {METHOD_SEND, EMPTY_BYTES} from "../constants/Constants.sol";

5: import {IpcEnvelope, ResultMsg, CallMsg, IpcMsgKind, OutcomeType} from "../structs/CrossNet.sol";

6: import {IPCMsgType} from "../enums/IPCMsgType.sol";

7: import {SubnetID, IPCAddress} from "../structs/Subnet.sol";

8: import {SubnetIDHelper} from "../lib/SubnetIDHelper.sol";

9: import {FvmAddressHelper} from "../lib/FvmAddressHelper.sol";

10: import {LibGateway} from "../lib/LibGateway.sol";

11: import {FvmAddress} from "../structs/FvmAddress.sol";

12: import {FilAddress} from "fevmate/contracts/utils/FilAddress.sol";

13: import {Address} from "@openzeppelin/contracts/utils/Address.sol";

14: import {Asset} from "../structs/Subnet.sol";

15: import {AssetHelper} from "./AssetHelper.sol";

16: import {IIpcHandler} from "../../sdk/interfaces/IIpcHandler.sol";

18: import "../errors/IPCErrors.sol";

226:                 ++i;

```

```solidity
File: contracts/lib/FvmAddressHelper.sol

4: import {FvmAddress, DelegatedAddress} from "../structs/FvmAddress.sol";

```

```solidity
File: contracts/lib/LibActivity.sol

4: import {IValidatorRewarder} from "../interfaces/IValidatorRewarder.sol";

5: import {Consensus, CompressedActivityRollup} from "../structs/Activity.sol";

6: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

7: import {MerkleProof} from "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

8: import {SubnetID} from "../structs/Subnet.sol";

9: import {SubnetIDHelper} from "../lib/SubnetIDHelper.sol";

10: import {InvalidActivityProof, MissingActivityCommitment, ValidatorAlreadyClaimed} from "../errors/IPCErrors.sol";

66:         for (uint256 i = 0; i < proof.length; i++) {

106:         for (uint256 i = 0; i < size; i++) {

110:             for (uint256 j = 0; j < heights.length; j++) {

```

```solidity
File: contracts/lib/LibDiamond.sol

4: import {IDiamondCut} from "../interfaces/IDiamondCut.sol";

5: import {IDiamond} from "../interfaces/IDiamond.sol";

6: import {NotOwner} from "../errors/IPCErrors.sol";

106:                 ++facetIndex;

132:             ++selectorCount;

134:                 ++selectorIndex;

162:                 ++selectorIndex;

187:             --selectorCount;

198:                 ++selectorIndex;

209:         (bool success, bytes memory error) = _init.delegatecall(_calldata); // solhint-disable-line avoid-low-level-calls

```

```solidity
File: contracts/lib/LibGateway.sol

4: import {IPCMsgType} from "../enums/IPCMsgType.sol";

5: import {GatewayActorStorage, LibGatewayActorStorage} from "../lib/LibGatewayActorStorage.sol";

6: import {BURNT_FUNDS_ACTOR} from "../constants/Constants.sol";

7: import {SubnetID, Subnet, AssetKind, Asset} from "../structs/Subnet.sol";

8: import {SubnetActorGetterFacet} from "../subnet/SubnetActorGetterFacet.sol";

9: import {CallMsg, IpcMsgKind, IpcEnvelope, OutcomeType, BottomUpMsgBatch, BottomUpMsgBatch, BottomUpCheckpoint, ParentFinality} from "../structs/CrossNet.sol";

10: import {Membership} from "../structs/Subnet.sol";

11: import {CrossMsgHelper} from "../lib/CrossMsgHelper.sol";

12: import {FilAddress} from "fevmate/contracts/utils/FilAddress.sol";

13: import {SubnetIDHelper} from "../lib/SubnetIDHelper.sol";

14: import {AssetHelper} from "../lib/AssetHelper.sol";

15: import {ISubnetActor} from "../interfaces/ISubnetActor.sol";

16: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

18: import "../errors/IPCErrors.sol";

105:                 ++i;

124:                 ++i;

195:                 ++i;

205:                     ++i;

216:             totalValidatorsWeight += self.validators[i].weight;

218:                 ++i;

251:         subnet.topDownNonce = topDownNonce + 1;

252:         subnet.circSupply += crossMessage.value;

269:         s.bottomUpNonce += 1;

297:                     ++i;

346:         return ((uint64(blockNumber) / checkPeriod) + 1) * checkPeriod;

358:                 ++i;

371:                 ++i;

417:                 revert("Expecting top-down messages only");

433:             subnet.appliedBottomUpNonce += 1;

448:             s.appliedTopDownNonce += 1;

496:         (success, result) = address(CrossMsgHelper).delegatecall( // solhint-disable-line avoid-low-level-calls

704:                 ++i;

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

4: import {NotSystemActor, NotEnoughFunds} from "../errors/IPCErrors.sol";

5: import {QuorumMap} from "../structs/Quorum.sol";

6: import {BottomUpCheckpoint, BottomUpMsgBatch, IpcEnvelope, ParentFinality} from "../structs/CrossNet.sol";

7: import {SubnetID, Subnet, ParentValidatorsTracker} from "../structs/Subnet.sol";

8: import {Membership} from "../structs/Subnet.sol";

9: import {AccountHelper} from "../lib/AccountHelper.sol";

10: import {FilAddress} from "fevmate/contracts/utils/FilAddress.sol";

11: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

```

```solidity
File: contracts/lib/LibMultisignatureChecker.sol

4: import {ECDSA} from "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

60:             weight = weight + weights[i];

62:                 ++i;

```

```solidity
File: contracts/lib/LibQuorum.sol

4: import {QuorumMap, QuorumInfo, QuorumObjKind} from "../structs/Quorum.sol";

5: import {InvalidRetentionHeight, QuorumAlreadyProcessed, FailedAddSignatory, InvalidSignature, SignatureReplay, NotAuthorized, FailedRemoveIncompleteQuorum, ZeroMembershipWeight, FailedAddIncompleteQuorum} from "../errors/IPCErrors.sol";

6: import {MerkleProof} from "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

7: import {ECDSA} from "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

8: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

60:         info.currentWeight += weight;

148:                     ++i;

156:                 ++h;

172:         return (weight * majorityPercentage) / 100;

189:                 ++i;

```

```solidity
File: contracts/lib/LibStaking.sol

4: import {IGateway} from "../interfaces/IGateway.sol";

5: import {LibSubnetActorStorage, SubnetActorStorage} from "./LibSubnetActorStorage.sol";

6: import {LibMaxPQ, MaxPQ} from "./priority/LibMaxPQ.sol";

7: import {LibMinPQ, MinPQ} from "./priority/LibMinPQ.sol";

8: import {LibStakingChangeLog} from "./LibStakingChangeLog.sol";

9: import {AssetHelper} from "./AssetHelper.sol";

10: import {PermissionMode, StakingReleaseQueue, StakingChangeLog, StakingChange, StakingChangeRequest, StakingOperation, StakingRelease, ValidatorSet, AddressStakingReleases, ParentValidatorsTracker, Validator, Asset} from "../structs/Subnet.sol";

11: import {WithdrawExceedingCollateral, NotValidator, CannotConfirmFutureChanges, NoCollateralToWithdraw, AddressShouldBeValidator, InvalidConfigurationNumber} from "../errors/IPCErrors.sol";

12: import {Address} from "@openzeppelin/contracts/utils/Address.sol";

19:         uint16 nextIdx = self.startIdx + length;

22:         self.length = length + 1;

46:             amount += release.amount;

50:                 ++i;

51:                 --newLength;

75:         uint256 releaseAt = block.number + self.lockingDuration;

139:             addresses[i - 1] = validators.activeValidators.getAddress(i);

141:                 ++i;

151:             addresses[i - 1] = validators.waitingValidators.getAddress(i);

153:                 ++i;

164:             collateral += getPower(validators, validator);

166:                 ++i;

176:             collateral += getConfirmedCollateral(validators, validator);

178:                 ++i;

181:         collateral += getTotalConfirmedCollateral(validators);

199:                 ++i;

220:         validators.validators[validator].totalCollateral += amount;

230:         total -= amount;

249:         uint256 newCollateral = self.validators[validator].confirmedCollateral + amount;

252:         self.totalConfirmedCollateral += amount;

258:         uint256 newCollateral = self.validators[validator].confirmedCollateral - amount;

269:         self.totalConfirmedCollateral -= amount;

437:         return s.validatorSet.waitingValidators.getSize() + s.validatorSet.activeValidators.getSize();

502:                     ++i;

613:                 ++i;

617:         changeSet.startConfigurationNumber = configurationNumber + 1;

653:                 ++i;

690:                 ++i;

693:         self.changes.startConfigurationNumber = configurationNumber + 1;

```

```solidity
File: contracts/lib/LibStakingChangeLog.sol

4: import {StakingChangeLog, StakingChange, StakingOperation} from "../structs/Subnet.sol";

102:         changes.nextConfigurationNumber = configurationNumber + 1;

```

```solidity
File: contracts/lib/LibSubnetActor.sol

4: import {VALIDATOR_SECP256K1_PUBLIC_KEY_LENGTH} from "../constants/Constants.sol";

5: import {ERR_PERMISSIONED_AND_BOOTSTRAPPED} from "../errors/IPCErrors.sol";

6: import {NotEnoughGenesisValidators, DuplicatedGenesisValidator, NotOwnerOfPublicKey, MethodNotAllowed} from "../errors/IPCErrors.sol";

7: import {IGateway} from "../interfaces/IGateway.sol";

8: import {IValidatorGater} from "../interfaces/IValidatorGater.sol";

9: import {Validator, ValidatorSet, PermissionMode, SubnetID, Asset} from "../structs/Subnet.sol";

10: import {SubnetActorModifiers} from "../lib/LibSubnetActorStorage.sol";

11: import {LibValidatorSet, LibStaking} from "../lib/LibStaking.sol";

12: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

13: import {LibSubnetActorStorage, SubnetActorStorage} from "./LibSubnetActorStorage.sol";

14: import {SubnetIDHelper} from "../lib/SubnetIDHelper.sol";

15: import {AssetHelper} from "../lib/AssetHelper.sol";

77:                 i++;

148:                 ++i;

165:         msgValue += s.supplySource.makeAvailable(s.ipcGatewayAddr, genesisCircSupply);

166:         msgValue += s.collateralSource.makeAvailable(s.ipcGatewayAddr, collateral);

197:                 ++i;

210:                 s.genesisBalanceKeys[i] = s.genesisBalanceKeys[length - 1];

216:                 ++i;

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

4: import {ConsensusType} from "../enums/ConsensusType.sol";

5: import {NotGateway, SubnetAlreadyKilled} from "../errors/IPCErrors.sol";

6: import {BottomUpCheckpoint, BottomUpMsgBatchInfo} from "../structs/CrossNet.sol";

7: import {SubnetID, ValidatorSet, StakingChangeLog, StakingReleaseQueue, Asset, Validator, PermissionMode} from "../structs/Subnet.sol";

8: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

```

```solidity
File: contracts/lib/LibSubnetRegistryStorage.sol

4: import {SubnetCreationPrivileges} from "../structs/Subnet.sol";

```

```solidity
File: contracts/lib/SubnetIDHelper.sol

4: import {SubnetID} from "../structs/Subnet.sol";

5: import {Strings} from "@openzeppelin/contracts/utils/Strings.sol";

22:         return subnet.route[length - 1];

30:         address[] memory route = new address[](subnet.route.length - 1);

35:                 ++i;

43:         string memory route = string.concat("/r", Strings.toString(subnet.root));

46:             route = string.concat(route, "/");

49:                 ++i;

63:         newSubnet.route = new address[](subnetRouteLength + 1);

67:                 ++i;

71:         newSubnet.route[newSubnet.route.length - 1] = actor;

79:         return subnet.route[subnet.route.length - 1];

109:                 ++i;

120:                 ++j;

144:                 ++i;

148:         ++i;

155:                 ++j;

```

```solidity
File: contracts/lib/priority/LibMaxPQ.sol

4: import {LibValidatorSet} from "../LibStaking.sol";

5: import {ValidatorSet} from "../../structs/Subnet.sol";

6: import {PQ, LibPQ} from "./LibPQ.sol";

33:         uint16 size = self.inner.size + 1;

53:         self.inner.size = size - 1;

69:         self.inner.size = size - 1;

119:             parentPos = pos >> 1; // parentPos = pos / 2

133:         uint16 childPos = pos << 1; // childPos = pos * 2

145:                     pos2: childPos + 1

```

```solidity
File: contracts/lib/priority/LibMinPQ.sol

4: import {LibValidatorSet} from "../LibStaking.sol";

5: import {ValidatorSet} from "../../structs/Subnet.sol";

6: import {PQ, LibPQ} from "./LibPQ.sol";

32:         uint16 size = self.inner.size + 1;

51:         self.inner.size = size - 1;

66:         self.inner.size = size - 1;

129:         uint16 childPos = pos * 2;

141:                     pos2: childPos + 1

154:             childPos = pos * 2;

```

```solidity
File: contracts/lib/priority/LibPQ.sol

4: import {LibValidatorSet} from "../LibStaking.sol";

5: import {ValidatorSet} from "../../structs/Subnet.sol";

6: import {PQEmpty, PQDoesNotContainAddress} from "../../errors/IPCErrors.sol";

```

```solidity
File: contracts/structs/Activity.sol

4: import {SubnetID} from "../structs/Subnet.sol";

```

```solidity
File: contracts/structs/CrossNet.sol

4: import {SubnetID, IPCAddress} from "./Subnet.sol";

5: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

6: import {CompressedActivityRollup} from "../structs/Activity.sol";

```

```solidity
File: contracts/structs/Quorum.sol

4: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

```

```solidity
File: contracts/structs/Subnet.sol

4: import {FvmAddress} from "./FvmAddress.sol";

5: import {MaxPQ} from "../lib/priority/LibMaxPQ.sol";

6: import {MinPQ} from "../lib/priority/LibMinPQ.sol";

```

```solidity
File: contracts/subnet/SubnetActorActivityFacet.sol

4: import {Consensus} from "../structs/Activity.sol";

5: import {LibActivity} from "../lib/LibActivity.sol";

6: import {LibDiamond} from "../lib/LibDiamond.sol";

7: import {NotAuthorized} from "../errors/IPCErrors.sol";

8: import {Pausable} from "../lib/LibPausable.sol";

9: import {ReentrancyGuard} from "../lib/LibReentrancyGuard.sol";

10: import {SubnetIDHelper} from "../lib/SubnetIDHelper.sol";

11: import {SubnetID} from "../structs/Subnet.sol";

29:                 i++;

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

4: import {InvalidBatchEpoch, MaxMsgsPerBatchExceeded, InvalidSignatureErr, BottomUpCheckpointAlreadySubmitted, CannotSubmitFutureCheckpoint, InvalidCheckpointEpoch} from "../errors/IPCErrors.sol";

5: import {IGateway} from "../interfaces/IGateway.sol";

6: import {BottomUpCheckpoint, BottomUpMsgBatch, BottomUpMsgBatchInfo} from "../structs/CrossNet.sol";

7: import {Validator, ValidatorSet} from "../structs/Subnet.sol";

8: import {MultisignatureChecker} from "../lib/LibMultisignatureChecker.sol";

9: import {ReentrancyGuard} from "../lib/LibReentrancyGuard.sol";

10: import {SubnetActorModifiers} from "../lib/LibSubnetActorStorage.sol";

11: import {LibValidatorSet, LibStaking} from "../lib/LibStaking.sol";

12: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

13: import {LibSubnetActor} from "../lib/LibSubnetActor.sol";

14: import {Pausable} from "../lib/LibPausable.sol";

15: import {LibGateway} from "../lib/LibGateway.sol";

16: import {LibActivity} from "../lib/LibActivity.sol";

73:         uint256 threshold = (activeCollateral * s.majorityPercentage) / 100;

```

```solidity
File: contracts/subnet/SubnetActorGetterFacet.sol

4: import {ConsensusType} from "../enums/ConsensusType.sol";

5: import {BottomUpCheckpoint, IpcEnvelope} from "../structs/CrossNet.sol";

6: import {SubnetID, Asset} from "../structs/Subnet.sol";

7: import {SubnetID, ValidatorInfo, Validator, PermissionMode} from "../structs/Subnet.sol";

8: import {SubnetActorStorage} from "../lib/LibSubnetActorStorage.sol";

9: import {SubnetIDHelper} from "../lib/SubnetIDHelper.sol";

10: import {Address} from "@openzeppelin/contracts/utils/Address.sol";

11: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

12: import {LibStaking} from "../lib/LibStaking.sol";

79:                 ++i;

211:                 ++i;

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

4: import {VALIDATOR_SECP256K1_PUBLIC_KEY_LENGTH} from "../constants/Constants.sol";

5: import {ERR_VALIDATOR_JOINED, ERR_VALIDATOR_NOT_JOINED} from "../errors/IPCErrors.sol";

6: import {InvalidFederationPayload, SubnetAlreadyBootstrapped, NotEnoughFunds, CollateralIsZero, CannotReleaseZero, NotOwnerOfPublicKey, EmptyAddress, NotEnoughBalance, NotEnoughCollateral, NotValidator, NotAllValidatorsHaveLeft, InvalidPublicKeyLength, MethodNotAllowed, SubnetNotBootstrapped} from "../errors/IPCErrors.sol";

7: import {IGateway} from "../interfaces/IGateway.sol";

8: import {Validator, ValidatorSet, Asset, SubnetID} from "../structs/Subnet.sol";

9: import {SubnetIDHelper} from "../lib/SubnetIDHelper.sol";

10: import {LibDiamond} from "../lib/LibDiamond.sol";

11: import {ReentrancyGuard} from "../lib/LibReentrancyGuard.sol";

12: import {SubnetActorModifiers} from "../lib/LibSubnetActorStorage.sol";

13: import {LibValidatorSet, LibStaking} from "../lib/LibStaking.sol";

14: import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

15: import {Address} from "@openzeppelin/contracts/utils/Address.sol";

16: import {LibSubnetActor} from "../lib/LibSubnetActor.sol";

17: import {Pausable} from "../lib/LibPausable.sol";

18: import {AssetHelper} from "../lib/AssetHelper.sol";

48:         s.genesisBalance[msg.sender] += amount;

49:         s.genesisCircSupply += amount;

71:         s.genesisBalance[msg.sender] -= amount;

72:         s.genesisCircSupply -= amount;

196:         LibSubnetActor.gateValidatorPowerDelta(msg.sender, collateral, collateral + amount);

229:         LibSubnetActor.gateValidatorPowerDelta(msg.sender, collateral, collateral - amount);

267:                 s.genesisCircSupply -= genesisBalance;

```

```solidity
File: contracts/subnet/SubnetActorPauseFacet.sol

4: import {LibDiamond} from "../lib/LibDiamond.sol";

5: import {Pausable} from "../lib/LibPausable.sol";

```

```solidity
File: contracts/subnet/SubnetActorRewardFacet.sol

4: import {QuorumObjKind} from "../structs/Quorum.sol";

5: import {Pausable} from "../lib/LibPausable.sol";

6: import {ReentrancyGuard} from "../lib/LibReentrancyGuard.sol";

7: import {SubnetActorModifiers} from "../lib/LibSubnetActorStorage.sol";

8: import {LibStaking} from "../lib/LibStaking.sol";

9: import {LibSubnetActor} from "../lib/LibSubnetActor.sol";

10: import {AssetHelper} from "../lib/AssetHelper.sol";

11: import {Asset} from "../structs/Subnet.sol";

```

```solidity
File: contracts/subnetregistry/RegisterSubnetFacet.sol

4: import {IDiamond} from "../interfaces/IDiamond.sol";

5: import {SubnetActorDiamond} from "../SubnetActorDiamond.sol";

6: import {SubnetRegistryActorStorage} from "../lib/LibSubnetRegistryStorage.sol";

8: import {ReentrancyGuard} from "../lib/LibReentrancyGuard.sol";

9: import {WrongGateway} from "../errors/IPCErrors.sol";

11: import {SubnetCreationPrivileges} from "../structs/Subnet.sol";

12: import {LibDiamond} from "../lib/LibDiamond.sol";

93:         ++s.userNonces[msg.sender];

```

```solidity
File: contracts/subnetregistry/SubnetGetterFacet.sol

3: import {SubnetRegistryActorStorage} from "../lib/LibSubnetRegistryStorage.sol";

4: import {CannotFindSubnet, FacetCannotBeZero} from "../errors/IPCErrors.sol";

5: import {LibDiamond} from "../lib/LibDiamond.sol";

```

### <a name="GAS-7"></a>[GAS-7] Use Custom Errors instead of Revert Strings to save Gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas

Additionally, custom errors can be used inside and outside of contracts (including interfaces and libraries).

Source: <https://blog.soliditylang.org/2021/04/21/custom-errors/>:

> Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., `revert("Insufficient funds.");`), but they are rather expensive, especially when it comes to deploy cost, and it is difficult to use dynamic information in them.

Consider replacing **all revert strings** with custom errors in the solution, and particularly those that have multiple occurrences:

*Instances (9)*:
```solidity
File: contracts/lib/AssetHelper.sol

26:             require(asset.tokenAddress != address(0), "Invalid ERC20 address");

37:         require(asset.kind == kind, "Unexpected asset");

55:             require(finalBalance > initialBalance, "No balance increase");

62:             require(msg.value >= value, "Insufficient funds");

```

```solidity
File: contracts/lib/LibActivity.sol

88:         require(added, "duplicate checkpoint height");

```

```solidity
File: contracts/lib/LibGateway.sol

417:                 revert("Expecting top-down messages only");

718:             revert("Message not found in postbox");

```

```solidity
File: contracts/lib/LibStaking.sol

607:                     revert("Unknown staking operation");

```

```solidity
File: contracts/subnet/SubnetActorActivityFacet.sol

24:         require(checkpointHeights.length == claims.length, "length mismatch");

```

### <a name="GAS-8"></a>[GAS-8] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including `EXTCODESIZE` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

*Instances (7)*:
```solidity
File: contracts/lib/AssetHelper.sol

31:             token.balanceOf(address(0));

52:             uint256 initialBalance = token.balanceOf(address(this));

54:             uint256 finalBalance = token.balanceOf(address(this));

213:             ret = IERC20(asset.tokenAddress).balanceOf(address(this));

222:             ret = IERC20(asset.tokenAddress).balanceOf(holder);

```

```solidity
File: contracts/lib/LibDiamond.sol

209:         (bool success, bytes memory error) = _init.delegatecall(_calldata); // solhint-disable-line avoid-low-level-calls

```

```solidity
File: contracts/lib/LibGateway.sol

496:         (success, result) = address(CrossMsgHelper).delegatecall( // solhint-disable-line avoid-low-level-calls

```

### <a name="GAS-9"></a>[GAS-9] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (3)*:
```solidity
File: contracts/SubnetActorDiamond.sol

160:     function _onlyGateway() private view {

```

```solidity
File: contracts/lib/LibDiamond.sol

48:     function transferOwnership(address newOwner) internal onlyOwner {

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

82:     function _onlyGateway() private view {

```

### <a name="GAS-10"></a>[GAS-10] `++i` costs less gas compared to `i++` or `i += 1` (same for `--i` vs `i--` or `i -= 1`)
Pre-increments and pre-decrements are cheaper.

For a `uint256 i` variable, the following is true with the Optimizer enabled at 10k:

**Increment:**

- `i += 1` is the most expensive form
- `i++` costs 6 gas less than `i += 1`
- `++i` costs 5 gas less than `i++` (11 gas less than `i += 1`)

**Decrement:**

- `i -= 1` is the most expensive form
- `i--` costs 11 gas less than `i -= 1`
- `--i` costs 5 gas less than `i--` (16 gas less than `i -= 1`)

Note that post-increments (or post-decrements) return the old value before incrementing or decrementing, hence the name *post-increment*:

```solidity
uint i = 1;  
uint j = 2;
require(j == i++, "This will be false as i is incremented after the comparison");
```
  
However, pre-increments (or pre-decrements) return the new value:
  
```solidity
uint i = 1;  
uint j = 2;
require(j == ++i, "This will be true as i is incremented before the comparison");
```

In the pre-increment case, the compiler has to create a temporary variable (when used) for returning `1` instead of `2`.

Consider using pre-increments and pre-decrements where they are relevant (meaning: not where post-increments/decrements logic are relevant).

*Saves 5 gas per instance*

*Instances (5)*:
```solidity
File: contracts/lib/LibActivity.sol

66:         for (uint256 i = 0; i < proof.length; i++) {

106:         for (uint256 i = 0; i < size; i++) {

110:             for (uint256 j = 0; j < heights.length; j++) {

```

```solidity
File: contracts/lib/LibSubnetActor.sol

77:                 i++;

```

```solidity
File: contracts/subnet/SubnetActorActivityFacet.sol

29:                 i++;

```

### <a name="GAS-11"></a>[GAS-11] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (5)*:
```solidity
File: contracts/lib/FvmAddressHelper.sol

9:     uint8 public constant SECP256K1 = 1;

10:     uint8 public constant PAYLOAD_HASH_LEN = 20;

13:     uint8 public constant DELEGATED = 4;

14:     uint64 public constant EAM_ACTOR = 10;

```

```solidity
File: contracts/lib/LibDiamond.sol

9:     bytes32 public constant DIAMOND_STORAGE_POSITION = keccak256("libdiamond.lib.diamond.storage");

```

### <a name="GAS-12"></a>[GAS-12] Use shift right/left instead of division/multiplication if possible
While the `DIV` / `MUL` opcode uses 5 gas, the `SHR` / `SHL` opcode only uses 3 gas. Furthermore, beware that Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting. Eventually, overflow checks are never performed for shift operations as they are done for arithmetic operations. Instead, the result is always truncated, so the calculation can be unchecked in Solidity version `0.8+`
- Use `>> 1` instead of `/ 2`
- Use `>> 2` instead of `/ 4`
- Use `<< 3` instead of `* 8`
- ...
- Use `>> 5` instead of `/ 2^5 == / 32`
- Use `<< 6` instead of `* 2^6 == * 64`

TL;DR:
- Shifting left by N is like multiplying by 2^N (Each bits to the left is an increased power of 2)
- Shifting right by N is like dividing by 2^N (Each bits to the right is a decreased power of 2)

*Saves around 2 gas + 20 for unchecked per instance*

*Instances (2)*:
```solidity
File: contracts/lib/priority/LibMinPQ.sol

129:         uint16 childPos = pos * 2;

154:             childPos = pos * 2;

```

### <a name="GAS-13"></a>[GAS-13] Increments/decrements can be unchecked in for-loops
In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

[ethereum/solidity#10695](https://github.com/ethereum/solidity/issues/10695)

The change would be:

```diff
- for (uint256 i; i < numIterations; i++) {
+ for (uint256 i; i < numIterations;) {
 // ...  
+   unchecked { ++i; }
}  
```

These save around **25 gas saved** per instance.

The same can be applied with decrements (which should use `break` when `i == 0`).

The risk of overflow is non-existent for `uint256`.

*Instances (8)*:
```solidity
File: contracts/diamond/DiamondLoupeFacet.sol

36:         for (uint256 selectorIndex; selectorIndex < selectorCount; ++selectorIndex) {

41:             for (uint256 facetIndex; facetIndex < numFacets; ++facetIndex) {

91:         for (uint256 selectorIndex; selectorIndex < selectorCount; ++selectorIndex) {

115:         for (uint256 selectorIndex; selectorIndex < selectorCount; ++selectorIndex) {

120:             for (uint256 facetIndex; facetIndex < numFacets; ++facetIndex) {

```

```solidity
File: contracts/lib/LibActivity.sol

66:         for (uint256 i = 0; i < proof.length; i++) {

106:         for (uint256 i = 0; i < size; i++) {

110:             for (uint256 j = 0; j < heights.length; j++) {

```

### <a name="GAS-14"></a>[GAS-14] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (6)*:
```solidity
File: contracts/gateway/GatewayManagerFacet.sol

57:         if (genesisCircSupply > 0) {

60:         if (collateral > 0) {

```

```solidity
File: contracts/lib/AssetHelper.sol

139:             if (ret.length > 0) {

159:         return size > 0;

```

```solidity
File: contracts/lib/LibSubnetActor.sol

138:             if (LibStaking.getPower(validators[i]) > 0) {

```

```solidity
File: contracts/subnet/SubnetActorRewardFacet.sol

19:         if (amount > 0) {

```

### <a name="GAS-15"></a>[GAS-15] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (133)*:
```solidity
File: contracts/lib/AssetHelper.sol

18:     function hasSupplyOfKind(address subnetActor, AssetKind compare) internal view returns (bool) {

24:     function validate(Asset memory asset) internal view {

36:     function expect(Asset memory asset, AssetKind kind) internal pure {

40:     function equals(Asset memory asset, Asset memory asset2) internal pure returns (bool) {

47:     function lock(Asset memory asset, uint256 value) internal returns (uint256) {

69:     function transferFunds(

101:     function performCall(

209:     function balance(Asset memory asset) internal view returns (uint256 ret) {

218:     function balanceOf(Asset memory asset, address holder) internal view returns (uint256 ret) {

229:     function makeAvailable(Asset memory self, address spender, uint256 amount) internal returns (uint256 msgValue) {

240:     function native() internal pure returns (Asset memory) {

244:     function erc20(address token) internal pure returns (Asset memory) {

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

139:     function toHash(IpcEnvelope memory crossMsg) internal pure returns (bytes32) {

233:     function validateCrossMessage(

```

```solidity
File: contracts/lib/FvmAddressHelper.sol

19:     function from(address addr) internal pure returns (FvmAddress memory fvmAddress) {

28:     function toHash(FvmAddress memory fvmAddress) internal pure returns (bytes32) {

33:     function equal(FvmAddress memory a, FvmAddress memory b) internal pure returns (bool) {

40:     function extractEvmAddress(FvmAddress memory fvmAddress) internal pure returns (address addr) {

```

```solidity
File: contracts/lib/LibActivity.sol

77:     function recordActivityRollup(

95:     function listPendingConsensus(

124:     function processConsensusClaim(

159:     function setRewarder(address rewarder) internal {

```

```solidity
File: contracts/lib/LibDiamond.sol

48:     function transferOwnership(address newOwner) internal onlyOwner {

70:     function contractOwner() internal view returns (address contractOwner_) {

81:     function enforceIsContractOwner() internal view {

87:     function diamondCut(IDiamond.FacetCut[] memory _diamondCut, address _init, bytes memory _calldata) internal {

```

```solidity
File: contracts/lib/LibGateway.sol

46:     function getCurrentBottomUpCheckpoint()

58:     function getBottomUpCheckpoint(

68:     function getBottomUpMsgBatch(uint256 epoch) internal view returns (bool exists, BottomUpMsgBatch storage batch) {

76:     function bottomUpCheckpointExists(uint256 epoch) internal view returns (bool) {

82:     function bottomUpBatchMsgsExists(uint256 epoch) internal view returns (bool) {

88:     function storeBottomUpCheckpoint(BottomUpCheckpoint memory checkpoint) internal {

111:     function storeBottomUpMsgBatch(BottomUpMsgBatch memory batch) internal {

137:     function getLatestParentFinality() internal view returns (ParentFinality memory) {

144:     function commitParentFinality(

161:     function updateMembership(Membership memory membership) internal {

243:     function commitTopDownMsg(Subnet storage subnet, IpcEnvelope memory crossMessage) internal {

259:     function commitBottomUpMsg(IpcEnvelope memory crossMessage) internal {

353:     function applyMessages(SubnetID memory arrivingFrom, IpcEnvelope[] memory crossMsgs) internal {

366:     function applyTopDownMessages(SubnetID memory arrivingFrom, IpcEnvelope[] memory crossMsgs) internal {

567:     function crossMsgSideEffects(uint256 v, bool shouldBurn) internal {

575:     function checkMsgLength(IpcEnvelope[] calldata msgs) internal view {

692:     function propagateAllPostboxMessages() internal {

713:     function propagatePostboxMessage(bytes32 msgCid) internal {

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

81:     function appStorage() internal pure returns (GatewayActorStorage storage ds) {

```

```solidity
File: contracts/lib/LibMultisignatureChecker.sol

30:     function isValidWeightedMultiSignature(

```

```solidity
File: contracts/lib/LibQuorum.sol

23:     function addQuorumSignature(

93:     function createQuorumInfo(

133:     function pruneQuorums(QuorumMap storage self, uint256 newRetentionHeight) internal {

163:     function isHeightAlreadyProcessed(QuorumMap storage self, uint256 height) internal view {

```

```solidity
File: contracts/lib/LibStaking.sol

17:     function push(AddressStakingReleases storage self, StakingRelease memory release) internal {

28:     function compact(AddressStakingReleases storage self) internal returns (uint256, uint16) {

69:     function setLockDuration(StakingReleaseQueue storage self, uint256 blocks) internal {

74:     function addNewRelease(StakingReleaseQueue storage self, address validator, uint256 amount) internal {

84:     function claim(StakingReleaseQueue storage self, address validator) internal returns (uint256) {

123:     function totalActiveValidators(ValidatorSet storage validators) internal view returns (uint16 total) {

135:     function listActiveValidators(ValidatorSet storage validators) internal view returns (address[] memory addresses) {

147:     function listWaitingValidators(ValidatorSet storage validators) internal view returns (address[] memory addresses) {

160:     function getTotalActivePower(ValidatorSet storage validators) internal view returns (uint256 collateral) {

172:     function getTotalCollateral(ValidatorSet storage validators) internal view returns (uint256 collateral) {

186:     function getTotalPowerOfValidators(

210:     function setMetadata(ValidatorSet storage validators, address validator, bytes calldata metadata) internal {

219:     function recordDeposit(ValidatorSet storage validators, address validator, uint256 amount) internal {

224:     function recordWithdraw(ValidatorSet storage validators, address validator, uint256 amount) internal {

235:     function confirmFederatedPower(ValidatorSet storage self, address validator, uint256 power) internal {

248:     function confirmDeposit(ValidatorSet storage self, address validator, uint256 amount) internal {

257:     function confirmWithdraw(ValidatorSet storage self, address validator, uint256 amount) internal {

395:     function getPower(address validator) internal view returns (uint256 power) {

401:     function isActiveValidator(address validator) internal view returns (bool) {

407:     function isWaitingValidator(address validator) internal view returns (bool) {

415:     function isValidator(address addr) internal view returns (bool) {

429:     function totalActiveValidators() internal view returns (uint16) {

435:     function totalValidators() internal view returns (uint16) {

441:     function listActiveValidators() internal view returns (address[] memory addresses) {

447:     function listWaitingValidators() internal view returns (address[] memory addresses) {

452:     function getTotalConfirmedCollateral() internal view returns (uint256) {

457:     function getTotalCollateral() internal view returns (uint256) {

463:     function totalValidatorCollateral(address validator) internal view returns (uint256) {

471:     function setFederatedPowerWithConfirm(address validator, uint256 power) internal {

477:     function setMetadataWithConfirm(address validator, bytes calldata metadata) internal {

483:     function depositWithConfirm(address validator, uint256 amount) internal {

520:     function withdrawWithConfirm(address validator, uint256 amount) internal {

531:     function setFederatedPower(address validator, bytes calldata metadata, uint256 amount) internal {

537:     function setValidatorMetadata(address validator, bytes calldata metadata) internal {

543:     function deposit(address validator, uint256 amount) internal {

551:     function withdraw(address validator, uint256 amount) internal {

561:     function claimCollateral(address validator) internal returns (uint256 amount) {

567:     function getConfigurationNumbers() internal view returns (uint64, uint64) {

573:     function confirmChange(uint64 configurationNumber) internal {

641:     function batchStoreChange(

659:     function confirmChange(ParentValidatorsTracker storage self, uint64 configurationNumber) internal {

```

```solidity
File: contracts/lib/LibStakingChangeLog.sol

11:     function metadataRequest(StakingChangeLog storage changes, address validator, bytes calldata metadata) internal {

28:     function federatedPowerRequest(

54:     function withdrawRequest(StakingChangeLog storage changes, address validator, uint256 amount) internal {

73:     function depositRequest(StakingChangeLog storage changes, address validator, uint256 amount) internal {

106:     function getChange(

113:     function purgeChange(StakingChangeLog storage changes, uint64 configurationNumber) internal {

```

```solidity
File: contracts/lib/LibSubnetActor.sol

26:     function enforceCollateralValidation() internal view {

37:     function enforceFederatedValidation() internal view {

47:     function gateValidatorPowerDelta(address validator, uint256 oldPower, uint256 newPower) internal {

60:     function gateValidatorNewPowers(address[] calldata validators, uint256[] calldata newPowers) internal {

84:     function bootstrapSubnetIfNeeded() internal {

114:     function preBootstrapSetFederatedPower(

176:     function postBootstrapSetFederatedPower(

204:     function rmAddressFromBalanceKey(address addr) internal {

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

71:     function appStorage() internal pure returns (SubnetActorStorage storage ds) {

```

```solidity
File: contracts/lib/priority/LibMaxPQ.sol

18:     function getSize(MaxPQ storage self) internal view returns (uint16) {

22:     function getAddress(MaxPQ storage self, uint16 i) internal view returns (address) {

26:     function contains(MaxPQ storage self, address validator) internal view returns (bool) {

32:     function insert(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

46:     function pop(MaxPQ storage self, ValidatorSet storage validators) internal {

62:     function deleteReheapify(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

87:     function increaseReheapify(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

95:     function decreaseReheapify(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

103:     function max(MaxPQ storage self, ValidatorSet storage validators) internal view returns (address, uint256) {

```

```solidity
File: contracts/lib/priority/LibMinPQ.sol

17:     function getSize(MinPQ storage self) internal view returns (uint16) {

21:     function getAddress(MinPQ storage self, uint16 i) internal view returns (address) {

25:     function contains(MinPQ storage self, address validator) internal view returns (bool) {

31:     function insert(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

44:     function pop(MinPQ storage self, ValidatorSet storage validators) internal {

59:     function deleteReheapify(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

83:     function increaseReheapify(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

90:     function decreaseReheapify(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

98:     function min(MinPQ storage self, ValidatorSet storage validators) internal view returns (address, uint256) {

```

```solidity
File: contracts/lib/priority/LibPQ.sol

23:     function isEmpty(PQ storage self) internal view returns (bool) {

27:     function requireNotEmpty(PQ storage self) internal view {

33:     function getSize(PQ storage self) internal view returns (uint16) {

37:     function contains(PQ storage self, address validator) internal view returns (bool) {

41:     function getPosOrRevert(PQ storage self, address validator) internal view returns (uint16 pos) {

48:     function del(PQ storage self, uint16 pos) internal {

54:     function getPower(PQ storage self, ValidatorSet storage validators, uint16 pos) internal view returns (uint256) {

59:     function getConfirmedCollateral(

68:     function exchange(PQ storage self, uint16 pos1, uint16 pos2) internal {

```


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Replace `abi.encodeWithSignature` and `abi.encodeWithSelector` with `abi.encodeCall` which keeps the code typo/type safe | 5 |
| [NC-2](#NC-2) | Array indices should be referenced via `enum`s rather than via numeric literals | 12 |
| [NC-3](#NC-3) | `require()` should be used instead of `assert()` | 3 |
| [NC-4](#NC-4) | Use `string.concat()` or `bytes.concat()` instead of `abi.encodePacked` | 3 |
| [NC-5](#NC-5) | `constant`s should be defined rather than using magic numbers | 12 |
| [NC-6](#NC-6) | Control structures do not follow the Solidity Style Guide | 25 |
| [NC-7](#NC-7) | Critical Changes Should Use Two-step Procedure | 1 |
| [NC-8](#NC-8) | Default Visibility for constants | 16 |
| [NC-9](#NC-9) | Unused `error` definition | 3 |
| [NC-10](#NC-10) | Event missing indexed field | 25 |
| [NC-11](#NC-11) | Events that mark critical parameter changes should contain both the old and the new value | 3 |
| [NC-12](#NC-12) | Function ordering does not follow the Solidity style guide | 4 |
| [NC-13](#NC-13) | Functions should not be longer than 50 lines | 304 |
| [NC-14](#NC-14) | Change int to int256 | 20 |
| [NC-15](#NC-15) | Lack of checks in setters | 33 |
| [NC-16](#NC-16) | Lines are too long | 6 |
| [NC-17](#NC-17) | Missing Event for critical parameters change | 2 |
| [NC-18](#NC-18) | NatSpec is completely non-existent on functions that should have them | 2 |
| [NC-19](#NC-19) | Incomplete NatSpec: `@param` is missing on actually documented functions | 5 |
| [NC-20](#NC-20) | Incomplete NatSpec: `@return` is missing on actually documented functions | 2 |
| [NC-21](#NC-21) | Use a `modifier` instead of a `require/if` statement for a special `msg.sender` actor | 15 |
| [NC-22](#NC-22) | Constant state variables defined more than once | 2 |
| [NC-23](#NC-23) | Consider using named mappings | 23 |
| [NC-24](#NC-24) | Adding a `return` statement when the function defines a named return variable, is redundant | 45 |
| [NC-25](#NC-25) | Take advantage of Custom Error's return value property | 113 |
| [NC-26](#NC-26) | Contract does not follow the Solidity style guide's suggested layout ordering | 10 |
| [NC-27](#NC-27) | Internal and private variables and functions names should begin with an underscore | 197 |
| [NC-28](#NC-28) | Event is missing `indexed` fields | 27 |
| [NC-29](#NC-29) | Constants should be defined rather than using magic numbers | 1 |
| [NC-30](#NC-30) | `public` functions not called by the contract should be declared `external` instead | 17 |
| [NC-31](#NC-31) | Variables need not be initialized to zero | 6 |
### <a name="NC-1"></a>[NC-1] Replace `abi.encodeWithSignature` and `abi.encodeWithSelector` with `abi.encodeCall` which keeps the code typo/type safe
When using `abi.encodeWithSignature`, it is possible to include a typo for the correct function signature.
When using `abi.encodeWithSignature` or `abi.encodeWithSelector`, it is also possible to provide parameters that are not of the correct type for the function.

To avoid these pitfalls, it would be best to use [`abi.encodeCall`](https://solidity-by-example.org/abi-encode/) instead.

*Instances (5)*:
```solidity
File: contracts/lib/LibGateway.sol

404:                 abi.encodeWithSelector(InvalidXnetMessage.selector, InvalidXnetMessageReason.DstSubnet)

429:                     abi.encodeWithSelector(InvalidXnetMessage.selector, InvalidXnetMessageReason.Nonce)

444:                     abi.encodeWithSelector(InvalidXnetMessage.selector, InvalidXnetMessageReason.Nonce)

466:                     abi.encodeWithSelector(InvalidXnetMessage.selector, reason)

497:                 abi.encodeWithSelector(CrossMsgHelper.execute.selector, crossMsg, supplySource)

```

### <a name="NC-2"></a>[NC-2] Array indices should be referenced via `enum`s rather than via numeric literals

*Instances (12)*:
```solidity
File: contracts/diamond/DiamondLoupeFacet.sol

57:             facets_[numFacets].functionSelectors[0] = selector;

```

```solidity
File: contracts/lib/priority/LibMaxPQ.sol

106:         address addr = self.inner.posToAddress[1];

```

```solidity
File: contracts/lib/priority/LibMinPQ.sol

101:         address addr = self.inner.posToAddress[1];

```

```solidity
File: contracts/subnetregistry/RegisterSubnetFacet.sol

34:         diamondCut[0] = IDiamond.FacetCut({

41:         diamondCut[1] = IDiamond.FacetCut({

47:         diamondCut[2] = IDiamond.FacetCut({

53:         diamondCut[3] = IDiamond.FacetCut({

59:         diamondCut[4] = IDiamond.FacetCut({

65:         diamondCut[5] = IDiamond.FacetCut({

71:         diamondCut[6] = IDiamond.FacetCut({

77:         diamondCut[7] = IDiamond.FacetCut({

83:         diamondCut[8] = IDiamond.FacetCut({

```

### <a name="NC-3"></a>[NC-3] `require()` should be used instead of `assert()`
Prior to solidity version 0.8.0, hitting an assert consumes the **remainder of the transaction's available gas** rather than returning it, as `require()`/`revert()` do. `assert()` should be avoided even past solidity version 0.8.0 as its [documentation](https://docs.soliditylang.org/en/v0.8.14/control-structures.html#panic-via-assert-and-error-via-require) states that "The assert function creates an error of type Panic(uint256). ... Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix. Additionally, a require statement (or a custom error) are more friendly in terms of understanding what happened."

*Instances (3)*:
```solidity
File: contracts/lib/LibSubnetActor.sol

104:         assert(publicKey.length == VALIDATOR_SECP256K1_PUBLIC_KEY_LENGTH);

```

```solidity
File: contracts/lib/priority/LibPQ.sol

69:         assert(pos1 <= self.size);

70:         assert(pos2 <= self.size);

```

### <a name="NC-4"></a>[NC-4] Use `string.concat()` or `bytes.concat()` instead of `abi.encodePacked`
Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

Solidity version 0.8.12 introduces `string.concat()` (vs `abi.encodePacked(<str>,<str>), which catches concatenation errors (in the event of a `bytes` data mixed in the concatenation)`)

*Instances (3)*:
```solidity
File: contracts/lib/AssetHelper.sol

96:                 abi.encodePacked(IERC20.transfer.selector, abi.encode(recipient, value))

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

53:         CallMsg memory message = CallMsg({method: abi.encodePacked(method), params: params});

```

```solidity
File: contracts/lib/FvmAddressHelper.sol

21:             DelegatedAddress({namespace: EAM_ACTOR, length: 20, buffer: abi.encodePacked(addr)})

```

### <a name="NC-5"></a>[NC-5] `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*Instances (12)*:
```solidity
File: contracts/GatewayDiamond.sol

41:         if (params.majorityPercentage < 51 || params.majorityPercentage > 100) {

```

```solidity
File: contracts/SubnetActorDiamond.sol

56:         if (params.majorityPercentage < 51 || params.majorityPercentage > 100) {

59:         if (params.powerScale > 18) {

```

```solidity
File: contracts/lib/FvmAddressHelper.sol

21:             DelegatedAddress({namespace: EAM_ACTOR, length: 20, buffer: abi.encodePacked(addr)})

50:         if (delegated.length != 20) {

53:         if (delegated.buffer.length != 20) {

63:             addr := mload(add(bys, 20))

```

```solidity
File: contracts/lib/LibActivity.sol

111:                 uint64 height = uint64((uint256(heights[j]) << 192) >> 192);

```

```solidity
File: contracts/lib/LibQuorum.sol

172:         return (weight * majorityPercentage) / 100;

```

```solidity
File: contracts/lib/priority/LibMinPQ.sol

129:         uint16 childPos = pos * 2;

154:             childPos = pos * 2;

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

73:         uint256 threshold = (activeCollateral * s.majorityPercentage) / 100;

```

### <a name="NC-6"></a>[NC-6] Control structures do not follow the Solidity Style Guide
See the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) section of the Solidity Style Guide

*Instances (25)*:
```solidity
File: contracts/errors/IPCErrors.sol

98: string constant ERR_PERMISSIONED_AND_BOOTSTRAPPED = "Method not allowed if permissioned is enabled and subnet bootstrapped";

99: string constant ERR_VALIDATOR_JOINED = "Method not allowed if validator has already joined";

100: string constant ERR_VALIDATOR_NOT_JOINED = "Method not allowed if validator has not joined";

```

```solidity
File: contracts/gateway/GatewayManagerFacet.sol

4: import {GatewayActorModifiers} from "../lib/LibGatewayActorStorage.sol";

```

```solidity
File: contracts/gateway/GatewayMessengerFacet.sol

4: import {GatewayActorModifiers} from "../lib/LibGatewayActorStorage.sol";

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

4: import {GatewayActorModifiers} from "../../lib/LibGatewayActorStorage.sol";

```

```solidity
File: contracts/gateway/router/TopDownFinalityFacet.sol

4: import {GatewayActorModifiers} from "../../lib/LibGatewayActorStorage.sol";

53:         if (

```

```solidity
File: contracts/gateway/router/XnetMessagingFacet.sol

4: import {GatewayActorModifiers} from "../../lib/LibGatewayActorStorage.sol";

```

```solidity
File: contracts/interfaces/IValidatorRewarder.sol

19:     function notifyValidClaim(

```

```solidity
File: contracts/lib/LibActivity.sol

149:         IValidatorRewarder(s.validatorRewarder).notifyValidClaim(subnet, checkpointHeight, data);

```

```solidity
File: contracts/lib/LibReentrancyGuard.sol

20:         if (s.status == _ENTERED) revert ReentrancyError();

```

```solidity
File: contracts/lib/LibStaking.sol

275:             self.activeValidators.increaseReheapify(self, maybeActive);

304:                 self.waitingValidators.deleteReheapify(self, maybeActive);

315:             self.waitingValidators.increaseReheapify(self, maybeActive);

328:                 self.waitingValidators.deleteReheapify(self, validator);

332:             self.waitingValidators.decreaseReheapify(self, validator);

345:             self.activeValidators.deleteReheapify(self, validator);

358:         self.activeValidators.decreaseReheapify(self, validator);

```

```solidity
File: contracts/lib/LibSubnetActor.sol

10: import {SubnetActorModifiers} from "../lib/LibSubnetActorStorage.sol";

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

10: import {SubnetActorModifiers} from "../lib/LibSubnetActorStorage.sol";

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

12: import {SubnetActorModifiers} from "../lib/LibSubnetActorStorage.sol";

170:             LibSubnetActor.bootstrapSubnetIfNeeded();

202:             LibSubnetActor.bootstrapSubnetIfNeeded();

```

```solidity
File: contracts/subnet/SubnetActorRewardFacet.sol

7: import {SubnetActorModifiers} from "../lib/LibSubnetActorStorage.sol";

```

### <a name="NC-7"></a>[NC-7] Critical Changes Should Use Two-step Procedure
The critical procedures should be two step process.

See similar findings in previous Code4rena contests for reference: <https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure>

**Recommended Mitigation Steps**

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

*Instances (1)*:
```solidity
File: contracts/lib/LibDiamond.sol

62:     function setContractOwner(address _newOwner) internal {

```

### <a name="NC-8"></a>[NC-8] Default Visibility for constants
Some constants are using the default visibility. For readability, consider explicitly declaring them as `internal`.

*Instances (16)*:
```solidity
File: contracts/GatewayDiamond.sol

19: bool constant FEATURE_MULTILEVEL_CROSSMSG = true;

20: bool constant FEATURE_GENERAL_PUPRPOSE_CROSSMSG = true;

21: uint8 constant FEATURE_SUBNET_DEPTH = 10;

```

```solidity
File: contracts/constants/Constants.sol

4: address constant BURNT_FUNDS_ACTOR = address(99);

5: bytes32 constant EMPTY_HASH = bytes32("");

6: bytes constant EMPTY_BYTES = bytes("");

7: bytes4 constant METHOD_SEND = bytes4(0);

10: uint256 constant VALIDATOR_SECP256K1_PUBLIC_KEY_LENGTH = 65;

```

```solidity
File: contracts/errors/IPCErrors.sol

98: string constant ERR_PERMISSIONED_AND_BOOTSTRAPPED = "Method not allowed if permissioned is enabled and subnet bootstrapped";

99: string constant ERR_VALIDATOR_JOINED = "Method not allowed if validator has already joined";

100: string constant ERR_VALIDATOR_NOT_JOINED = "Method not allowed if validator has not joined";

```

```solidity
File: contracts/gateway/GatewayManagerFacet.sol

22: string constant ERR_CHILD_SUBNET_NOT_ALLOWED = "Subnet does not allow child subnets";

```

```solidity
File: contracts/gateway/GatewayMessengerFacet.sol

19: string constant ERR_GENERAL_CROSS_MSG_DISABLED = "Support for general-purpose cross-net messages is disabled";

20: string constant ERR_MULTILEVEL_CROSS_MSG_DISABLED = "Support for multi-level cross-net messages is disabled";

```

```solidity
File: contracts/structs/CrossNet.sol

8: uint64 constant MAX_MSGS_PER_BATCH = 10;

9: uint256 constant BATCH_PERIOD = 100;

```

### <a name="NC-9"></a>[NC-9] Unused `error` definition
Note that there may be cases where an error superficially appears to be used, but this is only because there are multiple definitions of the error in different files. In such cases, the error definition should be moved into a separate file. The instances below are the unused definitions.

*Instances (3)*:
```solidity
File: contracts/lib/LibDiamond.sol

18:     error NoSelectorsGivenToAdd();

19:     error NotContractOwner(address _user, address _contractOwner);

```

```solidity
File: contracts/lib/SubnetIDHelper.sol

14:     error EmptySubnet();

```

### <a name="NC-10"></a>[NC-10] Event missing indexed field
Index event fields make the field more quickly accessible [to off-chain tools](https://ethereum.stackexchange.com/questions/40396/can-somebody-please-explain-the-concept-of-event-indexing) that parse events. This is especially useful when it comes to filtering based on an address. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Where applicable, each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three applicable fields, all of the applicable fields should be indexed.

*Instances (25)*:
```solidity
File: contracts/gateway/GatewayManagerFacet.sol

30:     event SubnetDestroyed(SubnetID id);

```

```solidity
File: contracts/interfaces/IDiamond.sol

18:     event DiamondCut(FacetCut[] _diamondCut, address _init, bytes _calldata);

```

```solidity
File: contracts/lib/LibDiamond.sol

28:     event OwnershipTransferred(address oldOwner, address newOwner);

```

```solidity
File: contracts/lib/LibGateway.sol

29:     event MembershipUpdated(Membership);

40:     event MessagePropagatedFromPostbox(bytes32 id);

```

```solidity
File: contracts/lib/LibPausable.sol

16:     event Paused(address account);

21:     event Unpaused(address account);

```

```solidity
File: contracts/lib/LibQuorum.sol

14:     event QuorumReached(QuorumObjKind objKind, uint256 height, bytes32 objHash, uint256 quorumWeight);

15:     event QuorumWeightUpdated(QuorumObjKind objKind, uint256 height, bytes32 objHash, uint256 newWeight);

```

```solidity
File: contracts/lib/LibStaking.sol

67:     event NewCollateralRelease(address validator, uint256 amount, uint256 releaseBlock);

100:     event ActiveValidatorCollateralUpdated(address validator, uint256 newPower);

101:     event WaitingValidatorCollateralUpdated(address validator, uint256 newPower);

102:     event NewActiveValidator(address validator, uint256 power);

103:     event NewWaitingValidator(address validator, uint256 power);

104:     event ActiveValidatorReplaced(address oldValidator, address newValidator);

105:     event ActiveValidatorLeft(address validator);

106:     event WaitingValidatorLeft(address validator);

391:     event ConfigurationNumberConfirmed(uint64 number);

392:     event CollateralClaimed(address validator, uint256 amount);

```

```solidity
File: contracts/lib/LibStakingChangeLog.sol

8:     event NewStakingChangeRequest(StakingOperation op, address validator, bytes payload, uint64 configurationNumber);

```

```solidity
File: contracts/lib/LibSubnetActor.sol

22:     event SubnetBootstrapped(Validator[]);

```

```solidity
File: contracts/structs/Activity.sol

7: event ActivityRollupRecorded(uint64 checkpointHeight, FullActivityRollup rollup);

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

27:     event ValidatorGaterUpdated(address oldGater, address newGater);

28:     event NewBootstrapNode(string netAddress, address owner);

```

```solidity
File: contracts/subnetregistry/RegisterSubnetFacet.sol

18:     event SubnetDeployed(address subnetAddr);

```

### <a name="NC-11"></a>[NC-11] Events that mark critical parameter changes should contain both the old and the new value
This should especially be done if the new value is not required to be different from the old value

*Instances (3)*:
```solidity
File: contracts/lib/LibDiamond.sol

62:     function setContractOwner(address _newOwner) internal {
            DiamondStorage storage ds = diamondStorage();
    
            address oldOwner = ds.contractOwner;
            ds.contractOwner = _newOwner;
            emit OwnershipTransferred(oldOwner, _newOwner);

```

```solidity
File: contracts/lib/LibGateway.sol

161:     function updateMembership(Membership memory membership) internal {
             emit MembershipUpdated(membership);

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

81:     function setValidatorGater(address gater) external notKilled {
            LibDiamond.enforceIsContractOwner();
    
            emit ValidatorGaterUpdated(s.validatorGater, gater);

```

### <a name="NC-12"></a>[NC-12] Function ordering does not follow the Solidity style guide
According to the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern

*Instances (4)*:
```solidity
File: contracts/lib/CrossMsgHelper.sol

1: 
   Current order:
   public createTransferMsg
   public createCallMsg
   public createResultMsg
   public createReleaseMsg
   public createFundMsg
   public applyType
   internal toHash
   public toHash
   internal toTracingId
   internal isEmpty
   public execute
   external isSorted
   internal validateCrossMessage
   
   Suggested order:
   external isSorted
   public createTransferMsg
   public createCallMsg
   public createResultMsg
   public createReleaseMsg
   public createFundMsg
   public applyType
   public toHash
   public execute
   internal toHash
   internal toTracingId
   internal isEmpty
   internal validateCrossMessage

```

```solidity
File: contracts/lib/LibQuorum.sol

1: 
   Current order:
   internal addQuorumSignature
   internal createQuorumInfo
   internal pruneQuorums
   internal isHeightAlreadyProcessed
   internal weightNeeded
   external getSignatureBundle
   
   Suggested order:
   external getSignatureBundle
   internal addQuorumSignature
   internal createQuorumInfo
   internal pruneQuorums
   internal isHeightAlreadyProcessed
   internal weightNeeded

```

```solidity
File: contracts/lib/LibSubnetActor.sol

1: 
   Current order:
   internal enforceCollateralValidation
   internal enforceFederatedValidation
   internal gateValidatorPowerDelta
   internal gateValidatorNewPowers
   internal bootstrapSubnetIfNeeded
   internal publicKeyToAddress
   internal preBootstrapSetFederatedPower
   internal registerInGateway
   internal postBootstrapSetFederatedPower
   internal rmAddressFromBalanceKey
   
   Suggested order:
   internal publicKeyToAddress
   internal enforceCollateralValidation
   internal enforceFederatedValidation
   internal gateValidatorPowerDelta
   internal gateValidatorNewPowers
   internal bootstrapSubnetIfNeeded
   internal publicKeyToAddress
   internal preBootstrapSetFederatedPower
   internal registerInGateway
   internal postBootstrapSetFederatedPower
   internal rmAddressFromBalanceKey

```

```solidity
File: contracts/subnet/SubnetActorGetterFacet.sol

1: 
   Current order:
   external getParent
   external permissionMode
   external ipcGatewayAddr
   external minValidators
   external majorityPercentage
   external activeValidatorsLimit
   external getConfigurationNumbers
   external genesisValidators
   external genesisCircSupply
   external genesisBalances
   external bottomUpCheckPeriod
   external lastBottomUpCheckpointHeight
   external consensus
   external bootstrapped
   external killed
   external minActivationCollateral
   external getValidator
   external getActiveValidators
   external getWaitingValidators
   external getTotalValidatorsNumber
   external getActiveValidatorsNumber
   external getTotalConfirmedCollateral
   external getTotalCollateral
   external getTotalValidatorCollateral
   external getPower
   external isActiveValidator
   external isWaitingValidator
   public bottomUpCheckpointAtEpoch
   external bottomUpCheckpointHashAtEpoch
   external powerScale
   external getBootstrapNodes
   external crossMsgsHash
   external supplySource
   external collateralSource
   
   Suggested order:
   external getParent
   external permissionMode
   external ipcGatewayAddr
   external minValidators
   external majorityPercentage
   external activeValidatorsLimit
   external getConfigurationNumbers
   external genesisValidators
   external genesisCircSupply
   external genesisBalances
   external bottomUpCheckPeriod
   external lastBottomUpCheckpointHeight
   external consensus
   external bootstrapped
   external killed
   external minActivationCollateral
   external getValidator
   external getActiveValidators
   external getWaitingValidators
   external getTotalValidatorsNumber
   external getActiveValidatorsNumber
   external getTotalConfirmedCollateral
   external getTotalCollateral
   external getTotalValidatorCollateral
   external getPower
   external isActiveValidator
   external isWaitingValidator
   external bottomUpCheckpointHashAtEpoch
   external powerScale
   external getBootstrapNodes
   external crossMsgsHash
   external supplySource
   external collateralSource
   public bottomUpCheckpointAtEpoch

```

### <a name="NC-13"></a>[NC-13] Functions should not be longer than 50 lines
Overly complex code can make understanding functionality more difficult, try to further modularize your code to ensure readability 

*Instances (304)*:
```solidity
File: contracts/OwnershipFacet.sol

7:     function transferOwnership(address _newOwner) external {

11:     function owner() external view returns (address owner_) {

```

```solidity
File: contracts/diamond/DiamondCutFacet.sol

19:     function diamondCut(FacetCut[] calldata _diamondCut, address _init, bytes calldata _calldata) external override {

```

```solidity
File: contracts/diamond/DiamondLoupeFacet.sol

26:     function facets() external view override returns (Facet[] memory facets_) {

43:                     facets_[facetIndex].functionSelectors[numFacetSelectors[facetIndex]] = selector;

108:     function facetAddresses() external view override returns (address[] memory facetAddresses_) {

146:     function facetAddress(bytes4 _functionSelector) external view override returns (address facetAddress_) {

152:     function supportsInterface(bytes4 _interfaceId) external view override returns (bool) {

```

```solidity
File: contracts/gateway/GatewayGetterFacet.sol

26:     function getValidatorConfigurationNumbers() external view returns (uint64, uint64) {

31:     function getCommitSha() external view returns (bytes32) {

36:     function bottomUpNonce() external view returns (uint64) {

41:     function totalSubnets() external view returns (uint64) {

46:     function maxMsgsPerBottomUpBatch() external view returns (uint64) {

51:     function bottomUpCheckPeriod() external view returns (uint256) {

56:     function getNetworkName() external view returns (SubnetID memory) {

62:     function bottomUpCheckpoint(uint256 e) external view returns (BottomUpCheckpoint memory) {

68:     function bottomUpMsgBatch(uint256 e) external view returns (BottomUpMsgBatch memory) {

74:     function getParentFinality(uint256 blockNumber) external view returns (ParentFinality memory) {

79:     function getLatestParentFinality() external view returns (ParentFinality memory) {

87:     function getSubnet(SubnetID calldata subnetId) external view returns (bool, Subnet memory) {

95:     function subnets(bytes32 h) external view returns (Subnet memory subnet) {

102:     function getSubnetTopDownMsgsLength(SubnetID memory subnetId) external view returns (uint256) {

112:     function getTopDownNonce(SubnetID calldata subnetId) external view returns (bool, uint64) {

123:     function getAppliedBottomUpNonce(SubnetID calldata subnetId) external view returns (bool, uint64) {

132:     function appliedTopDownNonce() external view returns (uint64) {

138:     function postbox(bytes32 id) external view returns (IpcEnvelope memory storableMsg) {

142:     function postboxMsgs() external view returns (bytes32[] memory) {

147:     function majorityPercentage() external view returns (uint64) {

153:     function listSubnets() external view returns (Subnet[] memory) {

167:     function getSubnetKeys() external view returns (bytes32[] memory) {

172:     function getLastMembership() external view returns (Membership memory) {

177:     function getLastConfigurationNumber() external view returns (uint64) {

182:     function getCurrentMembership() external view returns (Membership memory) {

187:     function getCurrentConfigurationNumber() external view returns (uint64) {

194:     function getCheckpointInfo(uint256 h) external view returns (QuorumInfo memory) {

199:     function getCheckpointCurrentWeight(uint256 h) external view returns (uint256) {

204:     function getIncompleteCheckpointHeights() external view returns (uint256[] memory) {

209:     function getIncompleteCheckpoints() external view returns (BottomUpCheckpoint[] memory) {

224:     function getCheckpointRetentionHeight() external view returns (uint256) {

232:     function getQuorumThreshold(uint256 totalWeight) external view returns (uint256) {

```

```solidity
File: contracts/gateway/GatewayManagerFacet.sol

35:     function register(uint256 genesisCircSupply, uint256 collateral) external payable {

66:     function addStake(uint256 amount) external payable {

87:     function releaseStake(uint256 amount) external nonReentrant {

141:     function fund(SubnetID calldata subnetId, FvmAddress calldata to) external payable {

174:     function fundWithToken(SubnetID calldata subnetId, FvmAddress calldata to, uint256 amount) external nonReentrant {

212:     function release(FvmAddress calldata to) external payable {

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

33:     function commitCheckpoint(BottomUpCheckpoint calldata checkpoint) external {

86:     function pruneBottomUpCheckpoints(uint256 newRetentionHeight) external systemActorOnly {

132:     function execBottomUpMsgs(IpcEnvelope[] calldata msgs, Subnet storage subnet) internal {

```

```solidity
File: contracts/gateway/router/TopDownFinalityFacet.sol

34:     function storeValidatorChanges(StakingChangeRequest[] calldata changeRequests) external systemActorOnly {

40:     function getTrackerConfigurationNumbers() external view returns (uint64, uint64) {

49:     function applyFinalityChanges() external systemActorOnly returns (uint64) {

```

```solidity
File: contracts/gateway/router/XnetMessagingFacet.sol

29:     function applyCrossMessages(IpcEnvelope[] calldata crossMsgs) external systemActorOnly {

```

```solidity
File: contracts/interfaces/IDiamondCut.sol

13:     function diamondCut(IDiamond.FacetCut[] calldata _diamondCut, address _init, bytes calldata _calldata) external;

```

```solidity
File: contracts/interfaces/IDiamondLoupe.sol

22:     function facets() external view returns (Facet[] memory facets_);

27:     function facetFunctionSelectors(address _facet) external view returns (bytes4[] memory facetFunctionSelectors_);

31:     function facetAddresses() external view returns (address[] memory facetAddresses_);

37:     function facetAddress(bytes4 _functionSelector) external view returns (address facetAddress_);

```

```solidity
File: contracts/interfaces/IERC165.sol

11:     function supportsInterface(bytes4 interfaceId) external view returns (bool);

```

```solidity
File: contracts/interfaces/IGateway.sol

14:     function register(uint256 genesisCircSupply, uint256 collateral) external payable;

17:     function addStake(uint256 amount) external payable;

27:     function commitCheckpoint(BottomUpCheckpoint calldata bottomUpCheckpoint) external;

36:     function fund(SubnetID calldata subnetId, FvmAddress calldata to) external payable;

50:     function fundWithToken(SubnetID calldata subnetId, FvmAddress calldata to, uint256 amount) external;

57:     function release(FvmAddress calldata to) external payable;

69:     function commitParentFinality(ParentFinality calldata finality) external;

```

```solidity
File: contracts/interfaces/ISubnetActor.sol

8:     function supplySource() external view returns (Asset memory);

```

```solidity
File: contracts/interfaces/IValidatorGater.sol

13:     function interceptPowerDelta(SubnetID memory id, address validator, uint256 prevPower, uint256 newPower) external;

```

```solidity
File: contracts/lib/AccountHelper.sol

9:     function isSystemActor(address _address) external pure returns (bool) {

```

```solidity
File: contracts/lib/AssetHelper.sol

18:     function hasSupplyOfKind(address subnetActor, AssetKind compare) internal view returns (bool) {

24:     function validate(Asset memory asset) internal view {

36:     function expect(Asset memory asset, AssetKind kind) internal pure {

40:     function equals(Asset memory asset, Asset memory asset2) internal pure returns (bool) {

47:     function lock(Asset memory asset, uint256 value) internal returns (uint256) {

115:             (success, ret) = functionCallWithValue({target: target, data: data, value: value});

117:             (success, ret) = functionCallWithERC20Value({asset: asset, target: target, data: data, value: value});

154:     function isContract(address account) internal view returns (bool) {

200:     function sendValue(address payable recipient, uint256 value) internal returns (bool) {

209:     function balance(Asset memory asset) internal view returns (uint256 ret) {

218:     function balanceOf(Asset memory asset, address holder) internal view returns (uint256 ret) {

229:     function makeAvailable(Asset memory self, address spender, uint256 amount) internal returns (uint256 msgValue) {

240:     function native() internal pure returns (Asset memory) {

244:     function erc20(address token) internal pure returns (Asset memory) {

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

124:     function applyType(IpcEnvelope calldata message, SubnetID calldata currentSubnet) public pure returns (IPCMsgType) {

139:     function toHash(IpcEnvelope memory crossMsg) internal pure returns (bytes32) {

143:     function toHash(IpcEnvelope[] memory crossMsgs) public pure returns (bytes32) {

149:     function toTracingId(IpcEnvelope memory crossMsg) internal pure returns (bytes32) {

164:     function isEmpty(IpcEnvelope memory crossMsg) internal pure returns (bool) {

211:     function isSorted(IpcEnvelope[] calldata crossMsgs) external pure returns (bool) {

```

```solidity
File: contracts/lib/FvmAddressHelper.sol

19:     function from(address addr) internal pure returns (FvmAddress memory fvmAddress) {

28:     function toHash(FvmAddress memory fvmAddress) internal pure returns (bytes32) {

33:     function equal(FvmAddress memory a, FvmAddress memory b) internal pure returns (bool) {

40:     function extractEvmAddress(FvmAddress memory fvmAddress) internal pure returns (address addr) {

60:     function _bytesToAddress(bytes memory bys) private pure returns (address addr) {

```

```solidity
File: contracts/lib/LibActivity.sol

166:     function consensusStorage() internal pure returns (ConsensusStorage storage ds) {

```

```solidity
File: contracts/lib/LibDiamond.sol

15:     error CannotAddFunctionToDiamondThatAlreadyExists(bytes4 _selector);

17:     error InitializationFunctionReverted(address _initializationContractAddress, bytes _calldata);

20:     error CannotReplaceFunctionsFromFacetWithZeroAddress(bytes4[] _selectors);

22:     error CannotReplaceFunctionWithTheSameFunctionFromTheSameFacet(bytes4 _selector);

48:     function transferOwnership(address newOwner) internal onlyOwner {

55:     function diamondStorage() internal pure returns (DiamondStorage storage ds) {

62:     function setContractOwner(address _newOwner) internal {

70:     function contractOwner() internal view returns (address contractOwner_) {

87:     function diamondCut(IDiamond.FacetCut[] memory _diamondCut, address _init, bytes memory _calldata) internal {

90:             bytes4[] memory functionSelectors = _diamondCut[facetIndex].functionSelectors;

113:     function addFunctions(address _facetAddress, bytes4[] memory _functionSelectors) internal {

139:     function replaceFunctions(address _facetAddress, bytes4[] memory _functionSelectors) internal {

142:             revert CannotReplaceFunctionsFromFacetWithZeroAddress(_functionSelectors);

154:                 revert CannotReplaceFunctionWithTheSameFunctionFromTheSameFacet(selector);

167:     function removeFunctions(address _facetAddress, bytes4[] memory _functionSelectors) internal {

203:     function initializeDiamondCut(address _init, bytes memory _calldata) internal {

225:     function enforceHasContractCode(address _contract, string memory _errorMessage) internal view {

```

```solidity
File: contracts/lib/LibGateway.sol

68:     function getBottomUpMsgBatch(uint256 epoch) internal view returns (bool exists, BottomUpMsgBatch storage batch) {

76:     function bottomUpCheckpointExists(uint256 epoch) internal view returns (bool) {

82:     function bottomUpBatchMsgsExists(uint256 epoch) internal view returns (bool) {

88:     function storeBottomUpCheckpoint(BottomUpCheckpoint memory checkpoint) internal {

111:     function storeBottomUpMsgBatch(BottomUpMsgBatch memory batch) internal {

131:     function getParentFinality(uint256 blockNumber) internal view returns (ParentFinality memory) {

137:     function getLatestParentFinality() internal view returns (ParentFinality memory) {

161:     function updateMembership(Membership memory membership) internal {

212:     function membershipTotalWeight(Membership memory self) internal pure returns (uint256) {

225:     function membershipEqual(Membership memory mb1, Membership memory mb2) internal pure returns (bool) {

243:     function commitTopDownMsg(Subnet storage subnet, IpcEnvelope memory crossMessage) internal {

259:     function commitBottomUpMsg(IpcEnvelope memory crossMessage) internal {

323:     function getSubnet(address actor) internal view returns (bool found, Subnet storage subnet) {

337:     function getSubnet(SubnetID memory subnetId) internal view returns (bool found, Subnet storage subnet) {

345:     function getNextEpoch(uint256 blockNumber, uint256 checkPeriod) internal pure returns (uint256) {

353:     function applyMessages(SubnetID memory arrivingFrom, IpcEnvelope[] memory crossMsgs) internal {

366:     function applyTopDownMessages(SubnetID memory arrivingFrom, IpcEnvelope[] memory crossMsgs) internal {

384:     function applyMsg(SubnetID memory arrivingFrom, IpcEnvelope memory crossMsg) internal {

397:     function applyMsg(SubnetID memory arrivingFrom, IpcEnvelope memory crossMsg, bool expectTopDownOnly) internal {

513:     function sendReceipt(IpcEnvelope memory original, OutcomeType outcomeType, bytes memory ret) internal {

538:     function commitValidatedCrossMessage(IpcEnvelope memory crossMessage) internal returns (bool shouldBurn) {

567:     function crossMsgSideEffects(uint256 v, bool shouldBurn) internal {

575:     function checkMsgLength(IpcEnvelope[] calldata msgs) internal view {

624:     function validateCrossMessage(IpcEnvelope memory envelope) internal view returns (bool, InvalidXnetMessageReason) {

713:     function propagatePostboxMessage(bytes32 msgCid) internal {

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

81:     function appStorage() internal pure returns (GatewayActorStorage storage ds) {

```

```solidity
File: contracts/lib/LibPausable.sol

110:     function pausableStorage() private pure returns (PausableStorage storage ds) {

```

```solidity
File: contracts/lib/LibQuorum.sol

133:     function pruneQuorums(QuorumMap storage self, uint256 newRetentionHeight) internal {

163:     function isHeightAlreadyProcessed(QuorumMap storage self, uint256 height) internal view {

171:     function weightNeeded(uint256 weight, uint256 majorityPercentage) internal pure returns (uint256) {

```

```solidity
File: contracts/lib/LibReentrancyGuard.sol

27:     function reentrancyStorage() private pure returns (ReentrancyStorage storage ds) {

```

```solidity
File: contracts/lib/LibStaking.sol

17:     function push(AddressStakingReleases storage self, StakingRelease memory release) internal {

28:     function compact(AddressStakingReleases storage self) internal returns (uint256, uint16) {

69:     function setLockDuration(StakingReleaseQueue storage self, uint256 blocks) internal {

74:     function addNewRelease(StakingReleaseQueue storage self, address validator, uint256 amount) internal {

84:     function claim(StakingReleaseQueue storage self, address validator) internal returns (uint256) {

109:     function getPower(ValidatorSet storage validators, address validator) internal view returns (uint256 power) {

118:     function getTotalConfirmedCollateral(ValidatorSet storage validators) internal view returns (uint256 collateral) {

123:     function totalActiveValidators(ValidatorSet storage validators) internal view returns (uint16 total) {

135:     function listActiveValidators(ValidatorSet storage validators) internal view returns (address[] memory addresses) {

147:     function listWaitingValidators(ValidatorSet storage validators) internal view returns (address[] memory addresses) {

160:     function getTotalActivePower(ValidatorSet storage validators) internal view returns (uint256 collateral) {

172:     function getTotalCollateral(ValidatorSet storage validators) internal view returns (uint256 collateral) {

205:     function isActiveValidator(ValidatorSet storage self, address validator) internal view returns (bool) {

210:     function setMetadata(ValidatorSet storage validators, address validator, bytes calldata metadata) internal {

219:     function recordDeposit(ValidatorSet storage validators, address validator, uint256 amount) internal {

224:     function recordWithdraw(ValidatorSet storage validators, address validator, uint256 amount) internal {

235:     function confirmFederatedPower(ValidatorSet storage self, address validator, uint256 power) internal {

248:     function confirmDeposit(ValidatorSet storage self, address validator, uint256 amount) internal {

257:     function confirmWithdraw(ValidatorSet storage self, address validator, uint256 amount) internal {

273:     function increaseReshuffle(ValidatorSet storage self, address maybeActive, uint256 newPower) internal {

325:     function reduceReshuffle(ValidatorSet storage self, address validator, uint256 newPower) internal {

395:     function getPower(address validator) internal view returns (uint256 power) {

401:     function isActiveValidator(address validator) internal view returns (bool) {

407:     function isWaitingValidator(address validator) internal view returns (bool) {

415:     function isValidator(address addr) internal view returns (bool) {

422:     function hasStaked(address validator) internal view returns (bool) {

429:     function totalActiveValidators() internal view returns (uint16) {

435:     function totalValidators() internal view returns (uint16) {

441:     function listActiveValidators() internal view returns (address[] memory addresses) {

447:     function listWaitingValidators() internal view returns (address[] memory addresses) {

452:     function getTotalConfirmedCollateral() internal view returns (uint256) {

457:     function getTotalCollateral() internal view returns (uint256) {

463:     function totalValidatorCollateral(address validator) internal view returns (uint256) {

471:     function setFederatedPowerWithConfirm(address validator, uint256 power) internal {

477:     function setMetadataWithConfirm(address validator, bytes calldata metadata) internal {

483:     function depositWithConfirm(address validator, uint256 amount) internal {

520:     function withdrawWithConfirm(address validator, uint256 amount) internal {

531:     function setFederatedPower(address validator, bytes calldata metadata, uint256 amount) internal {

537:     function setValidatorMetadata(address validator, bytes calldata metadata) internal {

543:     function deposit(address validator, uint256 amount) internal {

551:     function withdraw(address validator, uint256 amount) internal {

561:     function claimCollateral(address validator) internal returns (uint256 amount) {

567:     function getConfigurationNumbers() internal view returns (uint64, uint64) {

573:     function confirmChange(uint64 configurationNumber) internal {

629:     function storeChange(ParentValidatorsTracker storage self, StakingChangeRequest calldata changeRequest) internal {

659:     function confirmChange(ParentValidatorsTracker storage self, uint64 configurationNumber) internal {

```

```solidity
File: contracts/lib/LibStakingChangeLog.sol

11:     function metadataRequest(StakingChangeLog storage changes, address validator, bytes calldata metadata) internal {

54:     function withdrawRequest(StakingChangeLog storage changes, address validator, uint256 amount) internal {

73:     function depositRequest(StakingChangeLog storage changes, address validator, uint256 amount) internal {

113:     function purgeChange(StakingChangeLog storage changes, uint64 configurationNumber) internal {

```

```solidity
File: contracts/lib/LibSubnetActor.sol

26:     function enforceCollateralValidation() internal view {

37:     function enforceFederatedValidation() internal view {

47:     function gateValidatorPowerDelta(address validator, uint256 oldPower, uint256 newPower) internal {

60:     function gateValidatorNewPowers(address[] calldata validators, uint256[] calldata newPowers) internal {

103:     function publicKeyToAddress(bytes calldata publicKey) internal pure returns (address) {

159:     function registerInGateway(uint256 collateral) internal {

204:     function rmAddressFromBalanceKey(address addr) internal {

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

71:     function appStorage() internal pure returns (SubnetActorStorage storage ds) {

```

```solidity
File: contracts/lib/SubnetIDHelper.sol

16:     function getAddress(SubnetID memory subnet) public pure returns (address) {

25:     function getParentSubnet(SubnetID memory subnet) public pure returns (SubnetID memory) {

42:     function toString(SubnetID calldata subnet) public pure returns (string memory) {

56:     function toHash(SubnetID calldata subnet) public pure returns (bytes32) {

60:     function createSubnetId(SubnetID calldata subnet, address actor) public pure returns (SubnetID memory newSubnet) {

74:     function getActor(SubnetID calldata subnet) public pure returns (address) {

82:     function isRoot(SubnetID calldata subnet) public pure returns (bool) {

87:     function equals(SubnetID calldata subnet1, SubnetID calldata subnet2) public pure returns (bool) {

99:     function commonParent(SubnetID calldata subnet1, SubnetID calldata subnet2) public pure returns (SubnetID memory) {

132:     function down(SubnetID calldata subnet1, SubnetID calldata subnet2) public pure returns (bool, SubnetID memory) {

162:     function isEmpty(SubnetID calldata subnetId) public pure returns (bool) {

```

```solidity
File: contracts/lib/priority/LibMaxPQ.sol

18:     function getSize(MaxPQ storage self) internal view returns (uint16) {

22:     function getAddress(MaxPQ storage self, uint16 i) internal view returns (address) {

26:     function contains(MaxPQ storage self, address validator) internal view returns (bool) {

32:     function insert(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

46:     function pop(MaxPQ storage self, ValidatorSet storage validators) internal {

62:     function deleteReheapify(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

87:     function increaseReheapify(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

95:     function decreaseReheapify(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

103:     function max(MaxPQ storage self, ValidatorSet storage validators) internal view returns (address, uint256) {

114:     function swim(MaxPQ storage self, ValidatorSet storage validators, uint16 pos, uint256 value) internal {

132:     function sink(MaxPQ storage self, ValidatorSet storage validators, uint16 pos, uint256 value) internal {

178:     function firstValueSmaller(uint256 v1, uint256 v2) internal pure returns (bool) {

```

```solidity
File: contracts/lib/priority/LibMinPQ.sol

17:     function getSize(MinPQ storage self) internal view returns (uint16) {

21:     function getAddress(MinPQ storage self, uint16 i) internal view returns (address) {

25:     function contains(MinPQ storage self, address validator) internal view returns (bool) {

31:     function insert(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

44:     function pop(MinPQ storage self, ValidatorSet storage validators) internal {

59:     function deleteReheapify(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

83:     function increaseReheapify(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

90:     function decreaseReheapify(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

98:     function min(MinPQ storage self, ValidatorSet storage validators) internal view returns (address, uint256) {

109:     function swim(MinPQ storage self, ValidatorSet storage validators, uint16 pos, uint256 value) internal {

128:     function sink(MinPQ storage self, ValidatorSet storage validators, uint16 pos, uint256 value) internal {

174:     function firstValueLarger(uint256 v1, uint256 v2) internal pure returns (bool) {

```

```solidity
File: contracts/lib/priority/LibPQ.sol

23:     function isEmpty(PQ storage self) internal view returns (bool) {

27:     function requireNotEmpty(PQ storage self) internal view {

33:     function getSize(PQ storage self) internal view returns (uint16) {

37:     function contains(PQ storage self, address validator) internal view returns (bool) {

41:     function getPosOrRevert(PQ storage self, address validator) internal view returns (uint16 pos) {

48:     function del(PQ storage self, uint16 pos) internal {

54:     function getPower(PQ storage self, ValidatorSet storage validators, uint16 pos) internal view returns (uint256) {

68:     function exchange(PQ storage self, uint16 pos1, uint16 pos2) internal {

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

91:     function ensureValidCheckpoint(BottomUpCheckpoint calldata checkpoint) internal view {

```

```solidity
File: contracts/subnet/SubnetActorGetterFacet.sol

23:     function getParent() external view returns (SubnetID memory) {

28:     function permissionMode() external view returns (PermissionMode) {

33:     function ipcGatewayAddr() external view returns (address) {

38:     function minValidators() external view returns (uint64) {

43:     function majorityPercentage() external view returns (uint8) {

48:     function activeValidatorsLimit() external view returns (uint16) {

53:     function getConfigurationNumbers() external view returns (uint64, uint64) {

58:     function genesisValidators() external view returns (Validator[] memory) {

63:     function genesisCircSupply() external view returns (uint256) {

68:     function genesisBalances() external view returns (address[] memory, uint256[] memory) {

86:     function bottomUpCheckPeriod() external view returns (uint256) {

91:     function lastBottomUpCheckpointHeight() external view returns (uint256) {

96:     function consensus() external view returns (ConsensusType) {

101:     function bootstrapped() external view returns (bool) {

111:     function minActivationCollateral() external view returns (uint256) {

117:     function getValidator(address validatorAddress) external view returns (ValidatorInfo memory validator) {

122:     function getActiveValidators() external view returns (address[] memory) {

127:     function getWaitingValidators() external view returns (address[] memory) {

132:     function getTotalValidatorsNumber() external view returns (uint16) {

137:     function getActiveValidatorsNumber() external view returns (uint16) {

142:     function getTotalConfirmedCollateral() external view returns (uint256) {

147:     function getTotalCollateral() external view returns (uint256) {

153:     function getTotalValidatorCollateral(address validator) external view returns (uint256) {

159:     function getPower(address validator) external view returns (uint256) {

164:     function isActiveValidator(address validator) external view returns (bool) {

170:     function isWaitingValidator(address validator) external view returns (bool) {

190:     function bottomUpCheckpointHashAtEpoch(uint256 epoch) external view returns (bool, bytes32) {

196:     function powerScale() external view returns (int8) {

201:     function getBootstrapNodes() external view returns (string[] memory) {

221:     function crossMsgsHash(IpcEnvelope[] calldata messages) external pure returns (bytes32) {

226:     function supplySource() external view returns (Asset memory supply) {

231:     function collateralSource() external view returns (Asset memory supply) {

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

33:     function preFund(uint256 amount) external payable {

56:     function preRelease(uint256 amount) external nonReentrant {

81:     function setValidatorGater(address gater) external notKilled {

132:     function join(bytes calldata publicKey, uint256 amount) external payable nonReentrant whenNotPaused notKilled {

183:     function stake(uint256 amount) external payable whenNotPaused notKilled {

211:     function unstake(uint256 amount) external nonReentrant whenNotPaused notKilled {

240:     function leave() external nonReentrant whenNotPaused notKilled {

295:     function addBootstrapNode(string calldata netAddress) external whenNotPaused {

```

```solidity
File: contracts/subnet/SubnetActorRewardFacet.sol

17:     function claim() external nonReentrant whenNotPaused {

```

```solidity
File: contracts/subnetregistry/RegisterSubnetFacet.sol

56:             functionSelectors: s.subnetActorCheckpointerSelectors

68:             functionSelectors: s.subnetActorDiamondCutSelectors

74:             functionSelectors: s.subnetActorDiamondLoupeSelectors

80:             functionSelectors: s.subnetActorOwnershipSelectors

```

```solidity
File: contracts/subnetregistry/SubnetGetterFacet.sol

13:     function latestSubnetDeployed(address owner) external view returns (address subnet) {

28:     function getSubnetDeployedByNonce(address owner, uint64 nonce) external view returns (address subnet) {

40:     function getUserLastNonce(address user) external view returns (uint64 nonce) {

48:     function getGateway() external view returns (address) {

53:     function getSubnetActorGetterFacet() external view returns (address) {

58:     function getSubnetActorManagerFacet() external view returns (address) {

63:     function getSubnetActorRewarderFacet() external view returns (address) {

68:     function getSubnetActorCheckpointerFacet() external view returns (address) {

73:     function getSubnetActorPauserFacet() external view returns (address) {

78:     function getSubnetActorGetterSelectors() external view returns (bytes4[] memory) {

83:     function getSubnetActorManagerSelectors() external view returns (bytes4[] memory) {

88:     function getSubnetActorRewarderSelectors() external view returns (bytes4[] memory) {

93:     function getSubnetActorCheckpointerSelectors() external view returns (bytes4[] memory) {

98:     function getSubnetActorPauserSelectors() external view returns (bytes4[] memory) {

```

### <a name="NC-14"></a>[NC-14] Change int to int256
Throughout the code base, some variables are declared as `int`. To favor explicitness, consider changing all instances of `int` to `int256`

*Instances (20)*:
```solidity
File: contracts/gateway/GatewayGetterFacet.sol

62:     function bottomUpCheckpoint(uint256 e) external view returns (BottomUpCheckpoint memory) {

247:             BottomUpCheckpoint memory ch,

266:         returns (bool exists, uint256 epoch, BottomUpCheckpoint memory checkpoint)

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

33:     function commitCheckpoint(BottomUpCheckpoint calldata checkpoint) external {

59:         BottomUpCheckpoint calldata checkpoint,

```

```solidity
File: contracts/interfaces/IGateway.sol

27:     function commitCheckpoint(BottomUpCheckpoint calldata bottomUpCheckpoint) external;

73:         BottomUpCheckpoint calldata checkpoint,

```

```solidity
File: contracts/lib/LibActivity.sol

88:         require(added, "duplicate checkpoint height");

```

```solidity
File: contracts/lib/LibGateway.sol

49:         returns (bool exists, uint256 epoch, BottomUpCheckpoint memory checkpoint)

53:         checkpoint = s.bottomUpCheckpoints[epoch];

60:     ) internal view returns (bool exists, BottomUpCheckpoint storage checkpoint) {

63:         checkpoint = s.bottomUpCheckpoints[epoch];

88:     function storeBottomUpCheckpoint(BottomUpCheckpoint memory checkpoint) internal {

91:         BottomUpCheckpoint storage b = s.bottomUpCheckpoints[checkpoint.blockHeight];

```

```solidity
File: contracts/structs/CrossNet.sol

18: struct BottomUpCheckpoint {

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

29:         BottomUpCheckpoint calldata checkpoint,

91:     function ensureValidCheckpoint(BottomUpCheckpoint calldata checkpoint) internal view {

```

```solidity
File: contracts/subnet/SubnetActorGetterFacet.sol

180:     ) public view returns (bool exists, BottomUpCheckpoint memory checkpoint) {

181:         checkpoint = s.committedCheckpoints[epoch];

191:         (bool exists, BottomUpCheckpoint memory checkpoint) = bottomUpCheckpointAtEpoch(epoch);

```

### <a name="NC-15"></a>[NC-15] Lack of checks in setters
Be it sanity checks (like checks against `0`-values) or initial setting checks: it's best for Setter functions to have them

*Instances (33)*:
```solidity
File: contracts/lib/LibActivity.sol

159:     function setRewarder(address rewarder) internal {
             ConsensusStorage storage ds = consensusStorage();
             ds.validatorRewarder = rewarder;

```

```solidity
File: contracts/lib/LibDiamond.sol

62:     function setContractOwner(address _newOwner) internal {
            DiamondStorage storage ds = diamondStorage();
    
            address oldOwner = ds.contractOwner;
            ds.contractOwner = _newOwner;
            emit OwnershipTransferred(oldOwner, _newOwner);

```

```solidity
File: contracts/lib/LibStaking.sol

69:     function setLockDuration(StakingReleaseQueue storage self, uint256 blocks) internal {
            self.lockingDuration = blocks;

69:     function setLockDuration(StakingReleaseQueue storage self, uint256 blocks) internal {
            self.lockingDuration = blocks;

69:     function setLockDuration(StakingReleaseQueue storage self, uint256 blocks) internal {
            self.lockingDuration = blocks;

69:     function setLockDuration(StakingReleaseQueue storage self, uint256 blocks) internal {
            self.lockingDuration = blocks;

69:     function setLockDuration(StakingReleaseQueue storage self, uint256 blocks) internal {
            self.lockingDuration = blocks;

210:     function setMetadata(ValidatorSet storage validators, address validator, bytes calldata metadata) internal {
             validators.validators[validator].metadata = metadata;

210:     function setMetadata(ValidatorSet storage validators, address validator, bytes calldata metadata) internal {
             validators.validators[validator].metadata = metadata;

210:     function setMetadata(ValidatorSet storage validators, address validator, bytes calldata metadata) internal {
             validators.validators[validator].metadata = metadata;

210:     function setMetadata(ValidatorSet storage validators, address validator, bytes calldata metadata) internal {
             validators.validators[validator].metadata = metadata;

210:     function setMetadata(ValidatorSet storage validators, address validator, bytes calldata metadata) internal {
             validators.validators[validator].metadata = metadata;

471:     function setFederatedPowerWithConfirm(address validator, uint256 power) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.validatorSet.confirmFederatedPower(validator, power);

471:     function setFederatedPowerWithConfirm(address validator, uint256 power) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.validatorSet.confirmFederatedPower(validator, power);

471:     function setFederatedPowerWithConfirm(address validator, uint256 power) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.validatorSet.confirmFederatedPower(validator, power);

471:     function setFederatedPowerWithConfirm(address validator, uint256 power) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.validatorSet.confirmFederatedPower(validator, power);

471:     function setFederatedPowerWithConfirm(address validator, uint256 power) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.validatorSet.confirmFederatedPower(validator, power);

477:     function setMetadataWithConfirm(address validator, bytes calldata metadata) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.validatorSet.setMetadata(validator, metadata);

477:     function setMetadataWithConfirm(address validator, bytes calldata metadata) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.validatorSet.setMetadata(validator, metadata);

477:     function setMetadataWithConfirm(address validator, bytes calldata metadata) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.validatorSet.setMetadata(validator, metadata);

477:     function setMetadataWithConfirm(address validator, bytes calldata metadata) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.validatorSet.setMetadata(validator, metadata);

477:     function setMetadataWithConfirm(address validator, bytes calldata metadata) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.validatorSet.setMetadata(validator, metadata);

531:     function setFederatedPower(address validator, bytes calldata metadata, uint256 amount) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.changeSet.federatedPowerRequest({validator: validator, metadata: metadata, power: amount});

531:     function setFederatedPower(address validator, bytes calldata metadata, uint256 amount) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.changeSet.federatedPowerRequest({validator: validator, metadata: metadata, power: amount});

531:     function setFederatedPower(address validator, bytes calldata metadata, uint256 amount) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.changeSet.federatedPowerRequest({validator: validator, metadata: metadata, power: amount});

531:     function setFederatedPower(address validator, bytes calldata metadata, uint256 amount) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.changeSet.federatedPowerRequest({validator: validator, metadata: metadata, power: amount});

531:     function setFederatedPower(address validator, bytes calldata metadata, uint256 amount) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.changeSet.federatedPowerRequest({validator: validator, metadata: metadata, power: amount});

537:     function setValidatorMetadata(address validator, bytes calldata metadata) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.changeSet.metadataRequest(validator, metadata);

537:     function setValidatorMetadata(address validator, bytes calldata metadata) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.changeSet.metadataRequest(validator, metadata);

537:     function setValidatorMetadata(address validator, bytes calldata metadata) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.changeSet.metadataRequest(validator, metadata);

537:     function setValidatorMetadata(address validator, bytes calldata metadata) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.changeSet.metadataRequest(validator, metadata);

537:     function setValidatorMetadata(address validator, bytes calldata metadata) internal {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             s.changeSet.metadataRequest(validator, metadata);

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

81:     function setValidatorGater(address gater) external notKilled {
            LibDiamond.enforceIsContractOwner();
    
            emit ValidatorGaterUpdated(s.validatorGater, gater);
    
            s.validatorGater = gater;

```

### <a name="NC-16"></a>[NC-16] Lines are too long
Usually lines in source code are limited to [80](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width) characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over [164](https://github.com/aizatto/character-length) characters, the lines below should be split when they reach that length

*Instances (6)*:
```solidity
File: contracts/gateway/GatewayManagerFacet.sol

11: import {AlreadyRegisteredSubnet, CannotReleaseZero, MethodNotAllowed, NotEnoughFunds, NotEnoughFundsToRelease, NotEnoughCollateral, NotEmptySubnetCircSupply, NotRegisteredSubnet, InvalidXnetMessage, InvalidXnetMessageReason} from "../errors/IPCErrors.sol";

```

```solidity
File: contracts/lib/LibQuorum.sol

5: import {InvalidRetentionHeight, QuorumAlreadyProcessed, FailedAddSignatory, InvalidSignature, SignatureReplay, NotAuthorized, FailedRemoveIncompleteQuorum, ZeroMembershipWeight, FailedAddIncompleteQuorum} from "../errors/IPCErrors.sol";

```

```solidity
File: contracts/lib/LibStaking.sol

10: import {PermissionMode, StakingReleaseQueue, StakingChangeLog, StakingChange, StakingChangeRequest, StakingOperation, StakingRelease, ValidatorSet, AddressStakingReleases, ParentValidatorsTracker, Validator, Asset} from "../structs/Subnet.sol";

11: import {WithdrawExceedingCollateral, NotValidator, CannotConfirmFutureChanges, NoCollateralToWithdraw, AddressShouldBeValidator, InvalidConfigurationNumber} from "../errors/IPCErrors.sol";

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

4: import {InvalidBatchEpoch, MaxMsgsPerBatchExceeded, InvalidSignatureErr, BottomUpCheckpointAlreadySubmitted, CannotSubmitFutureCheckpoint, InvalidCheckpointEpoch} from "../errors/IPCErrors.sol";

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

6: import {InvalidFederationPayload, SubnetAlreadyBootstrapped, NotEnoughFunds, CollateralIsZero, CannotReleaseZero, NotOwnerOfPublicKey, EmptyAddress, NotEnoughBalance, NotEnoughCollateral, NotValidator, NotAllValidatorsHaveLeft, InvalidPublicKeyLength, MethodNotAllowed, SubnetNotBootstrapped} from "../errors/IPCErrors.sol";

```

### <a name="NC-17"></a>[NC-17] Missing Event for critical parameters change
Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

*Instances (2)*:
```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

94:     function setFederatedPower(
            address[] calldata validators,
            bytes[] calldata publicKeys,
            uint256[] calldata powers
        ) external notKilled {
            LibDiamond.enforceIsContractOwner();
    
            LibSubnetActor.enforceFederatedValidation();
    
            if (validators.length != powers.length) {
                revert InvalidFederationPayload();
            }
    
            if (validators.length != publicKeys.length) {
                revert InvalidFederationPayload();
            }
    
            if (s.bootstrapped) {

```

```solidity
File: contracts/subnetregistry/SubnetGetterFacet.sol

108:     function updateReferenceSubnetContract(
             address newGetterFacet,
             address newManagerFacet,
             bytes4[] calldata newSubnetGetterSelectors,
             bytes4[] calldata newSubnetManagerSelectors
         ) external {
             LibDiamond.enforceIsContractOwner();
     
             // Validate addresses are not zero
             if (newGetterFacet == address(0)) {
                 revert FacetCannotBeZero();
             }
             if (newManagerFacet == address(0)) {
                 revert FacetCannotBeZero();
             }
     
             // Update the storage variables
             s.SUBNET_ACTOR_GETTER_FACET = newGetterFacet;
             s.SUBNET_ACTOR_MANAGER_FACET = newManagerFacet;
     
             s.subnetActorGetterSelectors = newSubnetGetterSelectors;
             s.subnetActorManagerSelectors = newSubnetManagerSelectors;

```

### <a name="NC-18"></a>[NC-18] NatSpec is completely non-existent on functions that should have them
Public and external functions that aren't view or pure should have NatSpec comments

*Instances (2)*:
```solidity
File: contracts/OwnershipFacet.sol

7:     function transferOwnership(address _newOwner) external {

```

```solidity
File: contracts/subnet/SubnetActorActivityFacet.sol

19:     function batchSubnetClaim(

```

### <a name="NC-19"></a>[NC-19] Incomplete NatSpec: `@param` is missing on actually documented functions
The following functions are missing `@param` NatSpec comments.

*Instances (5)*:
```solidity
File: contracts/gateway/GatewayManagerFacet.sol

32:     /// @notice register a subnet in the gateway. It is called by a subnet when it reaches the threshold stake
        /// @dev The subnet can optionally pass a genesis circulating supply that would be pre-allocated in the
        /// subnet from genesis (without having to wait for the subnet to be spawned to propagate the funds).
        function register(uint256 genesisCircSupply, uint256 collateral) external payable {

65:     /// @notice addStake - add collateral for an existing subnet
        function addStake(uint256 amount) external payable {

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

173:     /// @notice Executes a cross message envelope.
         ///
         /// This function doesn't revert except if the envelope is empty.
         /// It returns a success flag and the return data for the success or
         /// the error so it can be returned to the sender through a cross-message receipt.
         /// NOTE: Execute assumes that the fund it is handling have already been
         /// released for their use so they can be conveniently included in the
         /// forwarded message, or the receipt in the case of failure.
         function execute(
             IpcEnvelope calldata crossMsg,
             Asset memory supplySource

```

```solidity
File: contracts/subnet/SubnetActorActivityFacet.sol

34:     /// Entrypoint for validators to claim their reward for doing work in the child subnet.
        function claim(
            SubnetID calldata subnet,
            uint64 checkpointHeight,
            Consensus.ValidatorData calldata data,
            Consensus.MerkleHash[] calldata proof

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

30:     /// @notice method to add some initial balance into a subnet that hasn't yet bootstrapped.
        /// @dev This balance is added to user addresses in genesis, and becomes part of the genesis
        /// circulating supply.
        function preFund(uint256 amount) external payable {

```

### <a name="NC-20"></a>[NC-20] Incomplete NatSpec: `@return` is missing on actually documented functions
The following functions are missing `@return` NatSpec comments.

*Instances (2)*:
```solidity
File: contracts/lib/CrossMsgHelper.sol

173:     /// @notice Executes a cross message envelope.
         ///
         /// This function doesn't revert except if the envelope is empty.
         /// It returns a success flag and the return data for the success or
         /// the error so it can be returned to the sender through a cross-message receipt.
         /// NOTE: Execute assumes that the fund it is handling have already been
         /// released for their use so they can be conveniently included in the
         /// forwarded message, or the receipt in the case of failure.
         function execute(
             IpcEnvelope calldata crossMsg,
             Asset memory supplySource
         ) public returns (bool success, bytes memory ret) {

```

```solidity
File: contracts/subnetregistry/RegisterSubnetFacet.sol

20:     /// @notice Deploys a new subnet actor.
        /// @param _params The constructor params for Subnet Actor Diamond.
        function newSubnetActor(
            SubnetActorDiamond.ConstructorParams calldata _params
        ) external nonReentrant returns (address subnetAddr) {

```

### <a name="NC-21"></a>[NC-21] Use a `modifier` instead of a `require/if` statement for a special `msg.sender` actor
If a function is supposed to be access-controlled, a `modifier` should be used instead of a `require/if` statement for more readability.

*Instances (15)*:
```solidity
File: contracts/SubnetActorDiamond.sol

161:         if (msg.sender != s.ipcGatewayAddr) {

```

```solidity
File: contracts/gateway/GatewayMessengerFacet.sol

48:         if (msg.sender.code.length == 0) {

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

35:         if (checkpoint.subnetID.getActor() != msg.sender) {

```

```solidity
File: contracts/lib/LibDiamond.sol

75:         if (msg.sender != diamondStorage().contractOwner) {

82:         if (msg.sender != diamondStorage().contractOwner) {

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

97:         if (!msg.sender.isSystemActor()) {

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

83:         if (msg.sender != s.ipcGatewayAddr) {

```

```solidity
File: contracts/subnet/SubnetActorActivityFacet.sol

54:         if (msg.sender != detail.validator) {

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

44:         if (s.genesisBalance[msg.sender] == 0) {

67:         if (s.genesisBalance[msg.sender] < amount) {

74:         if (s.genesisBalance[msg.sender] == 0) {

143:         if (LibStaking.isValidator(msg.sender)) {

153:         if (convertedAddress != msg.sender) {

191:         if (!LibStaking.isValidator(msg.sender)) {

296:         if (!s.validatorSet.isActiveValidator(msg.sender)) {

```

### <a name="NC-22"></a>[NC-22] Constant state variables defined more than once
Rather than redefining state variable constant, consider using a library to store all constants as this will prevent data redundancy

*Instances (2)*:
```solidity
File: contracts/lib/LibPausable.sol

7:     bytes32 private constant NAMESPACE = keccak256("pausable.lib.diamond.storage");

```

```solidity
File: contracts/lib/LibReentrancyGuard.sol

7:     bytes32 private constant NAMESPACE = keccak256("reentrancyguard.lib.diamond.storage");

```

### <a name="NC-23"></a>[NC-23] Consider using named mappings
Consider moving to solidity version 0.8.18 or later, and using [named mappings](https://ethereum.stackexchange.com/questions/51629/how-to-name-the-arguments-in-mapping/145555#145555) to make it easier to understand the purpose of each mapping

*Instances (23)*:
```solidity
File: contracts/lib/LibActivity.sol

28:         mapping(uint64 => ConsensusPendingAtHeight) pending;

47:         mapping(SubnetKey => ConsensusTracker) tracker;

```

```solidity
File: contracts/lib/LibDiamond.sol

37:         mapping(bytes4 => FacetAddressAndSelectorPosition) facetAddressAndSelectorPosition;

39:         mapping(bytes4 => bool) supportedInterfaces;

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

61:     mapping(bytes32 => Subnet) subnets;

63:     mapping(uint256 => ParentFinality) finalitiesMap;

67:     mapping(bytes32 => IpcEnvelope) postbox;

72:     mapping(uint256 => BottomUpCheckpoint) bottomUpCheckpoints;

75:     mapping(uint256 => BottomUpMsgBatch) bottomUpMsgBatches;

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

55:     mapping(address => string) bootstrapNodes;

59:     mapping(uint256 => BottomUpCheckpoint) committedCheckpoints;

63:     mapping(address => uint256) genesisBalance;

```

```solidity
File: contracts/lib/LibSubnetRegistryStorage.sol

51:     mapping(address => mapping(uint64 => address)) subnets;

55:     mapping(address => uint64) userNonces;

```

```solidity
File: contracts/lib/priority/LibPQ.sol

15:     mapping(address => uint16) addressToPos;

17:     mapping(uint16 => address) posToAddress;

```

```solidity
File: contracts/structs/Quorum.sol

35:     mapping(uint256 => QuorumInfo) quorumInfo;

40:     mapping(uint256 => EnumerableSet.AddressSet) quorumSignatureSenders;

42:     mapping(uint256 => mapping(address => bytes)) quorumSignatures;

```

```solidity
File: contracts/structs/Subnet.sol

54:     mapping(uint64 => StakingChange) changes;

71:     mapping(uint16 => StakingRelease) releases;

79:     mapping(address => AddressStakingReleases) releases;

135:     mapping(address => ValidatorInfo) validators;

```

### <a name="NC-24"></a>[NC-24] Adding a `return` statement when the function defines a named return variable, is redundant

*Instances (45)*:
```solidity
File: contracts/gateway/GatewayGetterFacet.sol

92:     /// @notice Returns information about a specific subnet using its hash identifier.
        /// @param h The hash identifier of the subnet to be queried.
        /// @return subnet The subnet information corresponding to the given hash.
        function subnets(bytes32 h) external view returns (Subnet memory subnet) {
            return s.subnets[h];

136:     /// @notice Returns the storable message and its wrapped status from the postbox by a given identifier.
         /// @param id The unique identifier of the message in the postbox.
         function postbox(bytes32 id) external view returns (IpcEnvelope memory storableMsg) {
             return (s.postbox[id]);

236:     /// @notice Retrieves a bundle of information and signatures for a specified bottom-up checkpoint.
         /// @param h The height of the checkpoint for which information is requested.
         /// @return ch The checkpoint information at the specified height.
         /// @return info Quorum information related to the checkpoint.
         /// @return signatories An array of addresses of signatories who have signed the checkpoint.
         function getCheckpointSignatureBundle(
             uint256 h
         )
             external
             view
             returns (
                 BottomUpCheckpoint memory ch,
                 QuorumInfo memory info,
                 address[] memory signatories,
                 bytes[] memory signatures
             )
         {
             ch = s.bottomUpCheckpoints[h];
             (info, signatories, signatures) = LibQuorum.getSignatureBundle(s.checkpointQuorumMap, h);
     
             return (ch, info, signatories, signatures);

259:     /// @notice Returns the current bottom-up checkpoint.
         /// @return exists - whether the checkpoint exists
         /// @return epoch - the epoch of the checkpoint
         /// @return checkpoint - the checkpoint struct
         function getCurrentBottomUpCheckpoint()
             external
             view
             returns (bool exists, uint256 epoch, BottomUpCheckpoint memory checkpoint)
         {
             (exists, epoch, checkpoint) = LibGateway.getCurrentBottomUpCheckpoint();
             return (exists, epoch, checkpoint);

```

```solidity
File: contracts/gateway/GatewayMessengerFacet.sol

29:     /**
         * @dev Sends a general-purpose cross-message from the local subnet to the destination subnet.
         * IMPORTANT: Native tokens via msg.value are treated as a contribution toward gas costs associated with message propagation.
         * There is no strict enforcement of the exact gas cost, and any msg.value provided will be accepted.
         *
         * IMPORTANT: Only smart contracts are allowed to trigger these cross-net messages. User wallets can send funds
         * from their address to the destination subnet and then run the transaction in the destination normally.
         *
         * @param envelope - the original envelope, which will be validated, stamped, and committed during the send.
         * @return committed envelope.
         */
        function sendContractXnetMessage(
            IpcEnvelope memory envelope
        ) external payable returns (IpcEnvelope memory committed) {
            if (!s.generalPurposeCrossMsg) {
                revert MethodNotAllowed(ERR_GENERAL_CROSS_MSG_DISABLED);
            }
    
            // We prevent the sender from being an EoA.
            if (msg.sender.code.length == 0) {
                revert InvalidXnetMessage(InvalidXnetMessageReason.Sender);
            }
    
            // Will revert if the message won't deserialize into a CallMsg.
            abi.decode(envelope.message, (CallMsg));
    
            committed = IpcEnvelope({
                kind: IpcMsgKind.Call,
                from: IPCAddress({subnetId: s.networkName, rawAddress: FvmAddressHelper.from(msg.sender)}),
                to: envelope.to,
                value: envelope.value,
                message: envelope.message,
                // nonce and originalNonce will be updated by LibGateway.commitValidatedCrossMessage
                originalNonce: 0,
                localNonce: 0
            });
    
            (bool valid, InvalidXnetMessageReason reason, IPCMsgType applyType) = committed.validateCrossMessage();
            if (!valid) {
                revert InvalidXnetMessage(reason);
            }
    
            if (applyType == IPCMsgType.TopDown) {
                (, SubnetID memory nextHop) = committed.to.subnetId.down(s.networkName);
                // lock funds on the current subnet gateway for the next hop
                ISubnetActor(nextHop.getActor()).supplySource().lock(envelope.value);
            }
    
            // Commit xnet message for dispatch.
            bool shouldBurn = LibGateway.commitValidatedCrossMessage(committed);
    
            // Apply side effects, such as burning funds.
            LibGateway.crossMsgSideEffects({v: committed.value, shouldBurn: shouldBurn});
    
            // Return a copy of the envelope, which was updated when it was committed.
            // Updates are visible to us because commitCrossMessage takes the envelope with memory scope,
            // which passes the struct by reference.
            return committed;

```

```solidity
File: contracts/lib/AssetHelper.sol

68:     /// @notice Transfers the specified amount out of our treasury to the recipient address.
        function transferFunds(
            Asset memory asset,
            address payable recipient,
            uint256 value
        ) internal returns (bool success, bytes memory ret) {
            if (asset.kind == AssetKind.Native) {
                success = sendValue(payable(recipient), value);
                return (success, EMPTY_BYTES);

68:     /// @notice Transfers the specified amount out of our treasury to the recipient address.
        function transferFunds(
            Asset memory asset,
            address payable recipient,
            uint256 value
        ) internal returns (bool success, bytes memory ret) {
            if (asset.kind == AssetKind.Native) {
                success = sendValue(payable(recipient), value);
                return (success, EMPTY_BYTES);
            } else if (asset.kind == AssetKind.ERC20) {
                return ierc20Transfer(asset, recipient, value);

82:     /// @notice Wrapper for an IERC20 transfer that bubbles up the success or failure
        /// and the return value instead of reverting so a cross-message receipt can be
        /// triggered from the execution.
        /// This function is the same as `safeTransfer` function used before.
        function ierc20Transfer(
            Asset memory asset,
            address recipient,
            uint256 value
        ) internal returns (bool success, bytes memory ret) {
            return

100:     /// @notice Calls the target with the specified data, ensuring it receives the specified value.
         function performCall(
             Asset memory asset,
             address payable target,
             bytes memory data,
             uint256 value
         ) internal returns (bool success, bytes memory ret) {
             // If value is zero, we can just go ahead and call the function.
             if (value == 0) {
                 return functionCallWithValue(target, data, 0);
             }
     
             // Otherwise, we need to do something different.
             if (asset.kind == AssetKind.Native) {
                 // Use the optimized path to send value along with the call.
                 (success, ret) = functionCallWithValue({target: target, data: data, value: value});
             } else if (asset.kind == AssetKind.ERC20) {
                 (success, ret) = functionCallWithERC20Value({asset: asset, target: target, data: data, value: value});
             }
             return (success, ret);

100:     /// @notice Calls the target with the specified data, ensuring it receives the specified value.
         function performCall(
             Asset memory asset,
             address payable target,
             bytes memory data,
             uint256 value
         ) internal returns (bool success, bytes memory ret) {
             // If value is zero, we can just go ahead and call the function.
             if (value == 0) {
                 return functionCallWithValue(target, data, 0);

122:     /// @dev Performs the function call with ERC20 value atomically
         function functionCallWithERC20Value(
             Asset memory asset,
             address target,
             bytes memory data,
             uint256 value
         ) internal returns (bool success, bytes memory ret) {
             // Transfer the tokens first, _then_ perform the call.
             (success, ret) = ierc20Transfer(asset, target, value);
     
             if (success) {
                 // Perform the call only if the ERC20 was successful.
                 (success, ret) = functionCallWithValue(target, data, 0);
             }
     
             if (!success) {
                 // following the implementation of `openzeppelin-contracts/utils/Address.sol`
                 if (ret.length > 0) {
                     assembly {
                         let returndata_size := mload(ret)
                         // see https://ethereum.stackexchange.com/questions/133748/trying-to-understand-solidity-assemblys-revert-function
                         revert(add(32, ret), returndata_size)
                     }
                 }
                 // disable solhint as the failing call does not have return data as well.
                 /* solhint-disable reason-string */
                 revert();
             }
             return (success, ret);

162:     /// @dev Adaptation from implementation `openzeppelin-contracts/utils/Address.sol`
         /// that doesn't revert immediately in case of failure and merely notifies of the outcome.
         function functionCallWithValue(
             address target,
             bytes memory data,
             uint256 value
         ) internal returns (bool success, bytes memory) {
             if (address(this).balance < value) {
                 revert NotEnoughBalance();
             }
     
             if (!isContract(target)) {
                 revert InvalidSubnetActor();
             }
     
             return target.call{value: value}(data);

229:     function makeAvailable(Asset memory self, address spender, uint256 amount) internal returns (uint256 msgValue) {
             if (self.kind == AssetKind.Native) {
                 msgValue = amount;
             } else if (self.kind == AssetKind.ERC20) {
                 IERC20 token = IERC20(self.tokenAddress);
                 token.safeIncreaseAllowance(spender, amount);
                 msgValue = 0;
             }
             return msgValue;

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

173:     /// @notice Executes a cross message envelope.
         ///
         /// This function doesn't revert except if the envelope is empty.
         /// It returns a success flag and the return data for the success or
         /// the error so it can be returned to the sender through a cross-message receipt.
         /// NOTE: Execute assumes that the fund it is handling have already been
         /// released for their use so they can be conveniently included in the
         /// forwarded message, or the receipt in the case of failure.
         function execute(
             IpcEnvelope calldata crossMsg,
             Asset memory supplySource
         ) public returns (bool success, bytes memory ret) {
             if (isEmpty(crossMsg)) {
                 revert CannotExecuteEmptyEnvelope();
             }
     
             address recipient = crossMsg.to.rawAddress.extractEvmAddress().normalize();
             if (crossMsg.kind == IpcMsgKind.Transfer) {
                 return supplySource.transferFunds({recipient: payable(recipient), value: crossMsg.value});
             } else if (crossMsg.kind == IpcMsgKind.Call || crossMsg.kind == IpcMsgKind.Result) {
                 // For a Result message, the idea is to perform a call as this returns control back to the caller.
                 // If it's an account, there will be no code to invoke, so this will be have like a bare transfer.
                 // But if the original caller was a contract, this give it control so it can handle the result
     
                 // send the envelope directly to the entrypoint
                 // use supplySource so the tokens in the message are handled successfully
                 // and by the right supply source
                 return
                     supplySource.performCall(
                         payable(recipient),
                         abi.encodeCall(IIpcHandler.handleIpcMessage, (crossMsg)),
                         crossMsg.value
                     );
             }
             return (false, EMPTY_BYTES);

173:     /// @notice Executes a cross message envelope.
         ///
         /// This function doesn't revert except if the envelope is empty.
         /// It returns a success flag and the return data for the success or
         /// the error so it can be returned to the sender through a cross-message receipt.
         /// NOTE: Execute assumes that the fund it is handling have already been
         /// released for their use so they can be conveniently included in the
         /// forwarded message, or the receipt in the case of failure.
         function execute(
             IpcEnvelope calldata crossMsg,
             Asset memory supplySource
         ) public returns (bool success, bytes memory ret) {
             if (isEmpty(crossMsg)) {
                 revert CannotExecuteEmptyEnvelope();
             }
     
             address recipient = crossMsg.to.rawAddress.extractEvmAddress().normalize();
             if (crossMsg.kind == IpcMsgKind.Transfer) {
                 return supplySource.transferFunds({recipient: payable(recipient), value: crossMsg.value});

173:     /// @notice Executes a cross message envelope.
         ///
         /// This function doesn't revert except if the envelope is empty.
         /// It returns a success flag and the return data for the success or
         /// the error so it can be returned to the sender through a cross-message receipt.
         /// NOTE: Execute assumes that the fund it is handling have already been
         /// released for their use so they can be conveniently included in the
         /// forwarded message, or the receipt in the case of failure.
         function execute(
             IpcEnvelope calldata crossMsg,
             Asset memory supplySource
         ) public returns (bool success, bytes memory ret) {
             if (isEmpty(crossMsg)) {
                 revert CannotExecuteEmptyEnvelope();
             }
     
             address recipient = crossMsg.to.rawAddress.extractEvmAddress().normalize();
             if (crossMsg.kind == IpcMsgKind.Transfer) {
                 return supplySource.transferFunds({recipient: payable(recipient), value: crossMsg.value});
             } else if (crossMsg.kind == IpcMsgKind.Call || crossMsg.kind == IpcMsgKind.Result) {
                 // For a Result message, the idea is to perform a call as this returns control back to the caller.
                 // If it's an account, there will be no code to invoke, so this will be have like a bare transfer.
                 // But if the original caller was a contract, this give it control so it can handle the result
     
                 // send the envelope directly to the entrypoint
                 // use supplySource so the tokens in the message are handled successfully
                 // and by the right supply source
                 return

```

```solidity
File: contracts/lib/LibActivity.sol

94:     /// A view accessor to query the pending consensus summaries for a given subnet.
        function listPendingConsensus(
            SubnetID calldata subnet
        ) internal view returns (ListPendingReturnEntry[] memory result) {
            ConsensusStorage storage s = consensusStorage();
    
            SubnetKey subnetKey = SubnetKey.wrap(subnet.toHash());
    
            uint256 size = s.tracker[subnetKey].pendingHeights.length();
            result = new ListPendingReturnEntry[](size);
    
            // Ok to not optimize with unchecked increments, since we expect this to be used off-chain only, for introspection.
            for (uint256 i = 0; i < size; i++) {
                ConsensusTracker storage tracker = s.tracker[subnetKey];
                bytes32[] memory heights = tracker.pendingHeights.values();
    
                for (uint256 j = 0; j < heights.length; j++) {
                    uint64 height = uint64((uint256(heights[j]) << 192) >> 192);
                    ConsensusPendingAtHeight storage pending = tracker.pending[height];
                    result[i] = ListPendingReturnEntry({
                        height: height,
                        summary: pending.summary,
                        claimed: pending.claimed.values()
                    });
                }
            }
    
            return result;

166:     function consensusStorage() internal pure returns (ConsensusStorage storage ds) {
             bytes32 position = CONSENSUS_NAMESPACE;
             assembly {
                 ds.slot := position
             }
             return ds;

```

```solidity
File: contracts/lib/LibGateway.sol

319:     /// @notice returns the subnet created by a validator
         /// @param actor the validator that created the subnet
         /// @return found whether the subnet exists
         /// @return subnet -  the subnet struct
         function getSubnet(address actor) internal view returns (bool found, Subnet storage subnet) {
             GatewayActorStorage storage s = LibGatewayActorStorage.appStorage();
             if (actor == address(0)) {
                 revert InvalidActorAddress();
             }
             SubnetID memory subnetId = s.networkName.createSubnetId(actor);
     
             return getSubnet(subnetId);

488:     /// @dev Execute the cross message using low level `call` method. This way ipc will
         ///      catch contract revert messages as well. We need this because in `CrossMsgHelper.execute`
         ///      there are `require` and `revert` calls, without reflexive call, the execution will
         ///      revert and block the checkpoint submission process.
         function executeCrossMsg(
             IpcEnvelope memory crossMsg,
             Asset memory supplySource
         ) internal returns (bool success, bytes memory result) {
             (success, result) = address(CrossMsgHelper).delegatecall( // solhint-disable-line avoid-low-level-calls
                     abi.encodeWithSelector(CrossMsgHelper.execute.selector, crossMsg, supplySource)
                 );
     
             if (success) {
                 return abi.decode(result, (bool, bytes));
             }
     
             return (success, result);

488:     /// @dev Execute the cross message using low level `call` method. This way ipc will
         ///      catch contract revert messages as well. We need this because in `CrossMsgHelper.execute`
         ///      there are `require` and `revert` calls, without reflexive call, the execution will
         ///      revert and block the checkpoint submission process.
         function executeCrossMsg(
             IpcEnvelope memory crossMsg,
             Asset memory supplySource
         ) internal returns (bool success, bytes memory result) {
             (success, result) = address(CrossMsgHelper).delegatecall( // solhint-disable-line avoid-low-level-calls
                     abi.encodeWithSelector(CrossMsgHelper.execute.selector, crossMsg, supplySource)
                 );
     
             if (success) {
                 return abi.decode(result, (bool, bytes));

531:     /**
          * @notice Commit the cross message to storage.
          *
          * @dev It does not make any validations. They are assumed to be done before calling this function.
          *  @param crossMessage The cross-network message to commit.
          *  @return shouldBurn A Boolean that indicates if the input amount should be burned.
          */
         function commitValidatedCrossMessage(IpcEnvelope memory crossMessage) internal returns (bool shouldBurn) {
             GatewayActorStorage storage s = LibGatewayActorStorage.appStorage();
     
             SubnetID memory to = crossMessage.to.subnetId;
             IPCMsgType applyType = crossMessage.applyType(s.networkName);
             bool isLCA = to.commonParent(crossMessage.from.subnetId).equals(s.networkName);
     
             // If the directionality is top-down, or if we're inverting the direction
             // because we're the LCA, commit a top-down message.
             if (applyType == IPCMsgType.TopDown || isLCA) {
                 (, SubnetID memory subnetId) = to.down(s.networkName);
                 (, Subnet storage subnet) = getSubnet(subnetId);
                 LibGateway.commitTopDownMsg(subnet, crossMessage);
                 return (shouldBurn = false);
             }
     
             // Else, commit a bottom up message.
             LibGateway.commitBottomUpMsg(crossMessage);
             // gas-opt: original check: value > 0
             return (shouldBurn = crossMessage.value != 0);

531:     /**
          * @notice Commit the cross message to storage.
          *
          * @dev It does not make any validations. They are assumed to be done before calling this function.
          *  @param crossMessage The cross-network message to commit.
          *  @return shouldBurn A Boolean that indicates if the input amount should be burned.
          */
         function commitValidatedCrossMessage(IpcEnvelope memory crossMessage) internal returns (bool shouldBurn) {
             GatewayActorStorage storage s = LibGatewayActorStorage.appStorage();
     
             SubnetID memory to = crossMessage.to.subnetId;
             IPCMsgType applyType = crossMessage.applyType(s.networkName);
             bool isLCA = to.commonParent(crossMessage.from.subnetId).equals(s.networkName);
     
             // If the directionality is top-down, or if we're inverting the direction
             // because we're the LCA, commit a top-down message.
             if (applyType == IPCMsgType.TopDown || isLCA) {
                 (, SubnetID memory subnetId) = to.down(s.networkName);
                 (, Subnet storage subnet) = getSubnet(subnetId);
                 LibGateway.commitTopDownMsg(subnet, crossMessage);
                 return (shouldBurn = false);

629:     /// @notice Validates a cross message and returns the applyType if the message is valid
         function checkCrossMessage(
             IpcEnvelope memory envelope
         ) internal view returns (bool valid, InvalidXnetMessageReason reason, IPCMsgType applyType) {
             SubnetID memory toSubnetId = envelope.to.subnetId;
             if (toSubnetId.isEmpty()) {
                 return (false, InvalidXnetMessageReason.DstSubnet, applyType);
             }
     
             GatewayActorStorage storage s = LibGatewayActorStorage.appStorage();
             SubnetID memory currentNetwork = s.networkName;
     
             // We cannot send a cross message to the same subnet.
             if (toSubnetId.equals(currentNetwork)) {
                 return (false, InvalidXnetMessageReason.ReflexiveSend, applyType);
             }
     
             // Lowest common ancestor subnet
             bool isLCA = toSubnetId.commonParent(envelope.from.subnetId).equals(currentNetwork);
             applyType = envelope.applyType(currentNetwork);
     
             // If the directionality is top-down, or if we're inverting the direction
             // else we need to check if the common parent exists.
             if (applyType == IPCMsgType.TopDown || isLCA) {
                 (bool foundChildSubnetId, SubnetID memory childSubnetId) = toSubnetId.down(currentNetwork);
                 if (!foundChildSubnetId) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);
                 }
     
                 (bool foundSubnet, ) = LibGateway.getSubnet(childSubnetId);
                 if (!foundSubnet) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);
                 }
             } else {
                 SubnetID memory commonParent = toSubnetId.commonParent(currentNetwork);
                 if (commonParent.isEmpty()) {
                     return (false, InvalidXnetMessageReason.NoRoute, applyType);
                 }
             }
     
             // starting/ending subnet, no need check supply sources
             if (envelope.from.subnetId.equals(currentNetwork) || envelope.to.subnetId.equals(currentNetwork)) {
                 return (true, reason, applyType);
             }
     
             bool supplySourcesCompatible = checkSubnetsSupplyCompatible({
                 isLCA: isLCA,
                 applyType: applyType,
                 incoming: envelope.from.subnetId,
                 outgoing: envelope.to.subnetId,
                 current: currentNetwork
             });
     
             if (!supplySourcesCompatible) {
                 return (false, InvalidXnetMessageReason.IncompatibleSupplySource, applyType);
             }
     
             return (true, reason, applyType);

629:     /// @notice Validates a cross message and returns the applyType if the message is valid
         function checkCrossMessage(
             IpcEnvelope memory envelope
         ) internal view returns (bool valid, InvalidXnetMessageReason reason, IPCMsgType applyType) {
             SubnetID memory toSubnetId = envelope.to.subnetId;
             if (toSubnetId.isEmpty()) {
                 return (false, InvalidXnetMessageReason.DstSubnet, applyType);

629:     /// @notice Validates a cross message and returns the applyType if the message is valid
         function checkCrossMessage(
             IpcEnvelope memory envelope
         ) internal view returns (bool valid, InvalidXnetMessageReason reason, IPCMsgType applyType) {
             SubnetID memory toSubnetId = envelope.to.subnetId;
             if (toSubnetId.isEmpty()) {
                 return (false, InvalidXnetMessageReason.DstSubnet, applyType);
             }
     
             GatewayActorStorage storage s = LibGatewayActorStorage.appStorage();
             SubnetID memory currentNetwork = s.networkName;
     
             // We cannot send a cross message to the same subnet.
             if (toSubnetId.equals(currentNetwork)) {
                 return (false, InvalidXnetMessageReason.ReflexiveSend, applyType);

629:     /// @notice Validates a cross message and returns the applyType if the message is valid
         function checkCrossMessage(
             IpcEnvelope memory envelope
         ) internal view returns (bool valid, InvalidXnetMessageReason reason, IPCMsgType applyType) {
             SubnetID memory toSubnetId = envelope.to.subnetId;
             if (toSubnetId.isEmpty()) {
                 return (false, InvalidXnetMessageReason.DstSubnet, applyType);
             }
     
             GatewayActorStorage storage s = LibGatewayActorStorage.appStorage();
             SubnetID memory currentNetwork = s.networkName;
     
             // We cannot send a cross message to the same subnet.
             if (toSubnetId.equals(currentNetwork)) {
                 return (false, InvalidXnetMessageReason.ReflexiveSend, applyType);
             }
     
             // Lowest common ancestor subnet
             bool isLCA = toSubnetId.commonParent(envelope.from.subnetId).equals(currentNetwork);
             applyType = envelope.applyType(currentNetwork);
     
             // If the directionality is top-down, or if we're inverting the direction
             // else we need to check if the common parent exists.
             if (applyType == IPCMsgType.TopDown || isLCA) {
                 (bool foundChildSubnetId, SubnetID memory childSubnetId) = toSubnetId.down(currentNetwork);
                 if (!foundChildSubnetId) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);
                 }
     
                 (bool foundSubnet, ) = LibGateway.getSubnet(childSubnetId);
                 if (!foundSubnet) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);
                 }
             } else {
                 SubnetID memory commonParent = toSubnetId.commonParent(currentNetwork);
                 if (commonParent.isEmpty()) {
                     return (false, InvalidXnetMessageReason.NoRoute, applyType);
                 }
             }
     
             // starting/ending subnet, no need check supply sources
             if (envelope.from.subnetId.equals(currentNetwork) || envelope.to.subnetId.equals(currentNetwork)) {
                 return (true, reason, applyType);

629:     /// @notice Validates a cross message and returns the applyType if the message is valid
         function checkCrossMessage(
             IpcEnvelope memory envelope
         ) internal view returns (bool valid, InvalidXnetMessageReason reason, IPCMsgType applyType) {
             SubnetID memory toSubnetId = envelope.to.subnetId;
             if (toSubnetId.isEmpty()) {
                 return (false, InvalidXnetMessageReason.DstSubnet, applyType);
             }
     
             GatewayActorStorage storage s = LibGatewayActorStorage.appStorage();
             SubnetID memory currentNetwork = s.networkName;
     
             // We cannot send a cross message to the same subnet.
             if (toSubnetId.equals(currentNetwork)) {
                 return (false, InvalidXnetMessageReason.ReflexiveSend, applyType);
             }
     
             // Lowest common ancestor subnet
             bool isLCA = toSubnetId.commonParent(envelope.from.subnetId).equals(currentNetwork);
             applyType = envelope.applyType(currentNetwork);
     
             // If the directionality is top-down, or if we're inverting the direction
             // else we need to check if the common parent exists.
             if (applyType == IPCMsgType.TopDown || isLCA) {
                 (bool foundChildSubnetId, SubnetID memory childSubnetId) = toSubnetId.down(currentNetwork);
                 if (!foundChildSubnetId) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);
                 }
     
                 (bool foundSubnet, ) = LibGateway.getSubnet(childSubnetId);
                 if (!foundSubnet) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);
                 }
             } else {
                 SubnetID memory commonParent = toSubnetId.commonParent(currentNetwork);
                 if (commonParent.isEmpty()) {
                     return (false, InvalidXnetMessageReason.NoRoute, applyType);
                 }
             }
     
             // starting/ending subnet, no need check supply sources
             if (envelope.from.subnetId.equals(currentNetwork) || envelope.to.subnetId.equals(currentNetwork)) {
                 return (true, reason, applyType);
             }
     
             bool supplySourcesCompatible = checkSubnetsSupplyCompatible({
                 isLCA: isLCA,
                 applyType: applyType,
                 incoming: envelope.from.subnetId,
                 outgoing: envelope.to.subnetId,
                 current: currentNetwork
             });
     
             if (!supplySourcesCompatible) {
                 return (false, InvalidXnetMessageReason.IncompatibleSupplySource, applyType);

629:     /// @notice Validates a cross message and returns the applyType if the message is valid
         function checkCrossMessage(
             IpcEnvelope memory envelope
         ) internal view returns (bool valid, InvalidXnetMessageReason reason, IPCMsgType applyType) {
             SubnetID memory toSubnetId = envelope.to.subnetId;
             if (toSubnetId.isEmpty()) {
                 return (false, InvalidXnetMessageReason.DstSubnet, applyType);
             }
     
             GatewayActorStorage storage s = LibGatewayActorStorage.appStorage();
             SubnetID memory currentNetwork = s.networkName;
     
             // We cannot send a cross message to the same subnet.
             if (toSubnetId.equals(currentNetwork)) {
                 return (false, InvalidXnetMessageReason.ReflexiveSend, applyType);
             }
     
             // Lowest common ancestor subnet
             bool isLCA = toSubnetId.commonParent(envelope.from.subnetId).equals(currentNetwork);
             applyType = envelope.applyType(currentNetwork);
     
             // If the directionality is top-down, or if we're inverting the direction
             // else we need to check if the common parent exists.
             if (applyType == IPCMsgType.TopDown || isLCA) {
                 (bool foundChildSubnetId, SubnetID memory childSubnetId) = toSubnetId.down(currentNetwork);
                 if (!foundChildSubnetId) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);
                 }
     
                 (bool foundSubnet, ) = LibGateway.getSubnet(childSubnetId);
                 if (!foundSubnet) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);
                 }
             } else {
                 SubnetID memory commonParent = toSubnetId.commonParent(currentNetwork);
                 if (commonParent.isEmpty()) {
                     return (false, InvalidXnetMessageReason.NoRoute, applyType);

629:     /// @notice Validates a cross message and returns the applyType if the message is valid
         function checkCrossMessage(
             IpcEnvelope memory envelope
         ) internal view returns (bool valid, InvalidXnetMessageReason reason, IPCMsgType applyType) {
             SubnetID memory toSubnetId = envelope.to.subnetId;
             if (toSubnetId.isEmpty()) {
                 return (false, InvalidXnetMessageReason.DstSubnet, applyType);
             }
     
             GatewayActorStorage storage s = LibGatewayActorStorage.appStorage();
             SubnetID memory currentNetwork = s.networkName;
     
             // We cannot send a cross message to the same subnet.
             if (toSubnetId.equals(currentNetwork)) {
                 return (false, InvalidXnetMessageReason.ReflexiveSend, applyType);
             }
     
             // Lowest common ancestor subnet
             bool isLCA = toSubnetId.commonParent(envelope.from.subnetId).equals(currentNetwork);
             applyType = envelope.applyType(currentNetwork);
     
             // If the directionality is top-down, or if we're inverting the direction
             // else we need to check if the common parent exists.
             if (applyType == IPCMsgType.TopDown || isLCA) {
                 (bool foundChildSubnetId, SubnetID memory childSubnetId) = toSubnetId.down(currentNetwork);
                 if (!foundChildSubnetId) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);

629:     /// @notice Validates a cross message and returns the applyType if the message is valid
         function checkCrossMessage(
             IpcEnvelope memory envelope
         ) internal view returns (bool valid, InvalidXnetMessageReason reason, IPCMsgType applyType) {
             SubnetID memory toSubnetId = envelope.to.subnetId;
             if (toSubnetId.isEmpty()) {
                 return (false, InvalidXnetMessageReason.DstSubnet, applyType);
             }
     
             GatewayActorStorage storage s = LibGatewayActorStorage.appStorage();
             SubnetID memory currentNetwork = s.networkName;
     
             // We cannot send a cross message to the same subnet.
             if (toSubnetId.equals(currentNetwork)) {
                 return (false, InvalidXnetMessageReason.ReflexiveSend, applyType);
             }
     
             // Lowest common ancestor subnet
             bool isLCA = toSubnetId.commonParent(envelope.from.subnetId).equals(currentNetwork);
             applyType = envelope.applyType(currentNetwork);
     
             // If the directionality is top-down, or if we're inverting the direction
             // else we need to check if the common parent exists.
             if (applyType == IPCMsgType.TopDown || isLCA) {
                 (bool foundChildSubnetId, SubnetID memory childSubnetId) = toSubnetId.down(currentNetwork);
                 if (!foundChildSubnetId) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);
                 }
     
                 (bool foundSubnet, ) = LibGateway.getSubnet(childSubnetId);
                 if (!foundSubnet) {
                     return (false, InvalidXnetMessageReason.DstSubnet, applyType);

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

81:     function appStorage() internal pure returns (GatewayActorStorage storage ds) {
            assembly {
                ds.slot := 0
            }
            return ds;

```

```solidity
File: contracts/lib/LibPausable.sol

109:     /// @notice get the storage slot
         function pausableStorage() private pure returns (PausableStorage storage ds) {
             bytes32 position = NAMESPACE;
             // solhint-disable-next-line no-inline-assembly
             assembly {
                 ds.slot := position
             }
             return ds;

```

```solidity
File: contracts/lib/LibQuorum.sol

175:     /// @notice get quorum signature bundle consisting of the info, signatories and the corresponding signatures.
         function getSignatureBundle(
             QuorumMap storage self,
             uint256 h
         ) external view returns (QuorumInfo memory info, address[] memory signatories, bytes[] memory signatures) {
             info = self.quorumInfo[h];
             signatories = self.quorumSignatureSenders[h].values();
             uint256 n = signatories.length;
     
             signatures = new bytes[](n);
     
             for (uint256 i; i < n; ) {
                 signatures[i] = self.quorumSignatures[h][signatories[i]];
                 unchecked {
                     ++i;
                 }
             }
     
             return (info, signatories, signatures);

```

```solidity
File: contracts/lib/LibReentrancyGuard.sol

26:     /// @dev fetch local storage
        function reentrancyStorage() private pure returns (ReentrancyStorage storage ds) {
            bytes32 position = NAMESPACE;
            // solhint-disable-next-line no-inline-assembly
            assembly {
                ds.slot := position
            }
            return ds;

```

```solidity
File: contracts/lib/LibStaking.sol

135:     function listActiveValidators(ValidatorSet storage validators) internal view returns (address[] memory addresses) {
             uint16 size = validators.activeValidators.getSize();
             addresses = new address[](size);
             for (uint16 i = 1; i <= size; ) {
                 addresses[i - 1] = validators.activeValidators.getAddress(i);
                 unchecked {
                     ++i;
                 }
             }
             return addresses;

147:     function listWaitingValidators(ValidatorSet storage validators) internal view returns (address[] memory addresses) {
             uint16 size = validators.waitingValidators.getSize();
             addresses = new address[](size);
             for (uint16 i = 1; i <= size; ) {
                 addresses[i - 1] = validators.waitingValidators.getAddress(i);
                 unchecked {
                     ++i;
                 }
             }
             return addresses;

395:     function getPower(address validator) internal view returns (uint256 power) {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             return s.validatorSet.getPower(validator);

440:     /// @notice Returns all active validators.
         function listActiveValidators() internal view returns (address[] memory addresses) {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             return s.validatorSet.listActiveValidators();

446:     /// @notice Returns all waiting validators.
         function listWaitingValidators() internal view returns (address[] memory addresses) {
             SubnetActorStorage storage s = LibSubnetActorStorage.appStorage();
             return s.validatorSet.listWaitingValidators();

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

71:     function appStorage() internal pure returns (SubnetActorStorage storage ds) {
            assembly {
                ds.slot := 0
            }
            return ds;

```

```solidity
File: contracts/subnet/SubnetActorGetterFacet.sol

174:     /// @notice returns the committed bottom-up checkpoint at specific epoch.
         /// @param epoch - the epoch to check.
         /// @return exists - whether the checkpoint exists.
         /// @return checkpoint - the checkpoint struct.
         function bottomUpCheckpointAtEpoch(
             uint256 epoch
         ) public view returns (bool exists, BottomUpCheckpoint memory checkpoint) {
             checkpoint = s.committedCheckpoints[epoch];
             exists = !checkpoint.subnetID.isEmpty();
             return (exists, checkpoint);

225:     /// @notice Returns the supply strategy for the subnet.
         function supplySource() external view returns (Asset memory supply) {
             return s.supplySource;

230:     /// @notice Returns the collateral asset kind for the subnet.
         function collateralSource() external view returns (Asset memory supply) {
             return s.collateralSource;

```

```solidity
File: contracts/subnetregistry/RegisterSubnetFacet.sol

20:     /// @notice Deploys a new subnet actor.
        /// @param _params The constructor params for Subnet Actor Diamond.
        function newSubnetActor(
            SubnetActorDiamond.ConstructorParams calldata _params
        ) external nonReentrant returns (address subnetAddr) {
            if (_params.ipcGatewayAddr != s.GATEWAY) {
                revert WrongGateway();
            }
    
            ensurePrivileges();
    
            IDiamond.FacetCut[] memory diamondCut = new IDiamond.FacetCut[](9);
    
            // set the diamond cut for subnet getter
            diamondCut[0] = IDiamond.FacetCut({
                facetAddress: s.SUBNET_ACTOR_GETTER_FACET,
                action: IDiamond.FacetCutAction.Add,
                functionSelectors: s.subnetActorGetterSelectors
            });
    
            // set the diamond cut for subnet manager
            diamondCut[1] = IDiamond.FacetCut({
                facetAddress: s.SUBNET_ACTOR_MANAGER_FACET,
                action: IDiamond.FacetCutAction.Add,
                functionSelectors: s.subnetActorManagerSelectors
            });
    
            diamondCut[2] = IDiamond.FacetCut({
                facetAddress: s.SUBNET_ACTOR_REWARD_FACET,
                action: IDiamond.FacetCutAction.Add,
                functionSelectors: s.subnetActorRewarderSelectors
            });
    
            diamondCut[3] = IDiamond.FacetCut({
                facetAddress: s.SUBNET_ACTOR_CHECKPOINTING_FACET,
                action: IDiamond.FacetCutAction.Add,
                functionSelectors: s.subnetActorCheckpointerSelectors
            });
    
            diamondCut[4] = IDiamond.FacetCut({
                facetAddress: s.SUBNET_ACTOR_PAUSE_FACET,
                action: IDiamond.FacetCutAction.Add,
                functionSelectors: s.subnetActorPauserSelectors
            });
    
            diamondCut[5] = IDiamond.FacetCut({
                facetAddress: s.SUBNET_ACTOR_DIAMOND_CUT_FACET,
                action: IDiamond.FacetCutAction.Add,
                functionSelectors: s.subnetActorDiamondCutSelectors
            });
    
            diamondCut[6] = IDiamond.FacetCut({
                facetAddress: s.SUBNET_ACTOR_LOUPE_FACET,
                action: IDiamond.FacetCutAction.Add,
                functionSelectors: s.subnetActorDiamondLoupeSelectors
            });
    
            diamondCut[7] = IDiamond.FacetCut({
                facetAddress: s.SUBNET_ACTOR_OWNERSHIP_FACET,
                action: IDiamond.FacetCutAction.Add,
                functionSelectors: s.subnetActorOwnershipSelectors
            });
    
            diamondCut[8] = IDiamond.FacetCut({
                facetAddress: s.VALIDATOR_REWARD_FACET,
                action: IDiamond.FacetCutAction.Add,
                functionSelectors: s.subnetActorActivitySelectors
            });
    
            // slither-disable-next-line reentrancy-benign
            subnetAddr = address(new SubnetActorDiamond(diamondCut, _params, msg.sender));
    
            //nonces start with 1, similar to eip 161
            ++s.userNonces[msg.sender];
            s.subnets[msg.sender][s.userNonces[msg.sender]] = subnetAddr;
    
            emit SubnetDeployed(subnetAddr);
    
            return subnetAddr;

```

### <a name="NC-25"></a>[NC-25] Take advantage of Custom Error's return value property
An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

*Instances (113)*:
```solidity
File: contracts/GatewayDiamond.sol

38:             revert InvalidSubmissionPeriod();

42:             revert InvalidMajorityPercentage();

106:                 revert(0, returndatasize())

```

```solidity
File: contracts/SubnetActorDiamond.sol

47:             revert GatewayCannotBeZero();

51:             revert InvalidSubmissionPeriod();

54:             revert InvalidCollateral();

57:             revert InvalidMajorityPercentage();

60:             revert InvalidPowerScale();

139:                 revert(0, returndatasize())

162:             revert NotGateway();

```

```solidity
File: contracts/SubnetRegistryDiamond.sol

43:             revert GatewayCannotBeZero();

46:             revert FacetCannotBeZero();

49:             revert FacetCannotBeZero();

52:             revert FacetCannotBeZero();

55:             revert FacetCannotBeZero();

58:             revert FacetCannotBeZero();

61:             revert FacetCannotBeZero();

64:             revert FacetCannotBeZero();

67:             revert FacetCannotBeZero();

70:             revert FacetCannotBeZero();

131:                 revert(0, returndatasize())

```

```solidity
File: contracts/gateway/GatewayManagerFacet.sol

46:             revert AlreadyRegisteredSubnet();

68:             revert NotEnoughFunds();

78:             revert NotRegisteredSubnet();

89:             revert CannotReleaseZero();

95:             revert NotRegisteredSubnet();

98:             revert NotEnoughFundsToRelease();

115:             revert NotRegisteredSubnet();

119:             revert NotEmptySubnetCircSupply();

149:             revert NotRegisteredSubnet();

182:             revert NotRegisteredSubnet();

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

36:             revert InvalidCheckpointSource();

40:             revert SubnetNotFound();

43:             revert InvalidSubnet();

65:             revert CheckpointAlreadyExists();

119:             revert CheckpointNotCreated();

148:             revert NotEnoughSubnetCircSupply();

```

```solidity
File: contracts/lib/AssetHelper.sol

148:             revert();

170:             revert NotEnoughBalance();

174:             revert InvalidSubnetActor();

202:             revert NotEnoughBalance();

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

186:             revert CannotExecuteEmptyEnvelope();

```

```solidity
File: contracts/lib/FvmAddressHelper.sol

42:             revert NotDelegatedEvmAddress();

48:             revert NotDelegatedEvmAddress();

51:             revert NotDelegatedEvmAddress();

54:             revert NotDelegatedEvmAddress();

```

```solidity
File: contracts/lib/LibActivity.sol

71:             revert InvalidActivityProof();

138:             revert MissingActivityCommitment();

145:             revert ValidatorAlreadyClaimed();

```

```solidity
File: contracts/lib/LibDiamond.sol

50:             revert InvalidAddress();

76:             revert NotOwner();

83:             revert NotOwner();

```

```solidity
File: contracts/lib/LibGateway.sol

151:             revert ParentFinalityAlreadyCommitted();

173:                 revert OldConfigurationNumber();

326:             revert InvalidActorAddress();

579:             revert MaxMsgsPerBatchExceeded();

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

98:             revert NotSystemActor();

```

```solidity
File: contracts/lib/LibPausable.sol

62:             revert EnforcedPause();

71:             revert ExpectedPause();

```

```solidity
File: contracts/lib/LibQuorum.sol

36:             revert InvalidSignature();

41:             revert SignatureReplay();

57:             revert FailedAddSignatory();

68:                     revert FailedRemoveIncompleteQuorum();

102:             revert QuorumAlreadyProcessed();

106:             revert ZeroMembershipWeight();

114:             revert FailedAddIncompleteQuorum();

137:             revert InvalidRetentionHeight();

165:             revert QuorumAlreadyProcessed();

```

```solidity
File: contracts/lib/LibReentrancyGuard.sol

20:         if (s.status == _ENTERED) revert ReentrancyError();

```

```solidity
File: contracts/lib/LibStaking.sol

31:             revert NoCollateralToWithdraw();

227:             revert WithdrawExceedingCollateral();

339:             revert AddressShouldBeValidator();

578:             revert CannotConfirmFutureChanges();

637:             revert InvalidConfigurationNumber();

661:             revert CannotConfirmFutureChanges();

```

```solidity
File: contracts/lib/LibSubnetActor.sol

124:             revert NotEnoughGenesisValidators();

133:                 revert NotOwnerOfPublicKey();

139:                 revert DuplicatedGenesisValidator();

189:                 revert NotOwnerOfPublicKey();

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

84:             revert NotGateway();

90:             revert SubnetAlreadyKilled();

```

```solidity
File: contracts/lib/SubnetIDHelper.sol

20:             revert NoAddressForRoot();

27:             revert NoParentForSubnet();

```

```solidity
File: contracts/lib/priority/LibPQ.sol

29:             revert PQEmpty();

44:             revert PQDoesNotContainAddress();

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

94:             revert MaxMsgsPerBatchExceeded();

102:             revert BottomUpCheckpointAlreadySubmitted();

108:             revert CannotSubmitFutureCheckpoint();

121:         revert InvalidCheckpointEpoch();

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

35:             revert NotEnoughFunds();

39:             revert SubnetAlreadyBootstrapped();

58:             revert NotEnoughFunds();

62:             revert SubnetAlreadyBootstrapped();

68:             revert NotEnoughBalance();

104:             revert InvalidFederationPayload();

108:             revert InvalidFederationPayload();

140:             revert CollateralIsZero();

149:             revert InvalidPublicKeyLength();

154:             revert NotOwnerOfPublicKey();

188:             revert CollateralIsZero();

217:             revert CannotReleaseZero();

226:             revert NotEnoughCollateral();

284:             revert NotAllValidatorsHaveLeft();

287:             revert SubnetNotBootstrapped();

300:             revert EmptyAddress();

```

```solidity
File: contracts/subnetregistry/RegisterSubnetFacet.sol

26:             revert WrongGateway();

```

```solidity
File: contracts/subnetregistry/SubnetGetterFacet.sol

16:             revert CannotFindSubnet();

21:             revert CannotFindSubnet();

30:             revert CannotFindSubnet();

34:             revert CannotFindSubnet();

43:             revert CannotFindSubnet();

118:             revert FacetCannotBeZero();

121:             revert FacetCannotBeZero();

```

### <a name="NC-26"></a>[NC-26] Contract does not follow the Solidity style guide's suggested layout ordering
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be:

1) Type declarations
2) State variables
3) Events
4) Modifiers
5) Functions

However, the contract(s) below do not follow this ordering

*Instances (10)*:
```solidity
File: contracts/SubnetActorDiamond.sol

1: 
   Current order:
   VariableDeclaration.s
   UsingForDirective.SubnetID
   UsingForDirective.Asset
   StructDefinition.ConstructorParams
   FunctionDefinition.constructor
   FunctionDefinition._fallback
   FunctionDefinition.fallback
   FunctionDefinition.receive
   FunctionDefinition._onlyGateway
   ModifierDefinition.onlyGateway
   
   Suggested order:
   UsingForDirective.SubnetID
   UsingForDirective.Asset
   VariableDeclaration.s
   StructDefinition.ConstructorParams
   ModifierDefinition.onlyGateway
   FunctionDefinition.constructor
   FunctionDefinition._fallback
   FunctionDefinition.fallback
   FunctionDefinition.receive
   FunctionDefinition._onlyGateway

```

```solidity
File: contracts/gateway/GatewayGetterFacet.sol

1: 
   Current order:
   VariableDeclaration.s
   UsingForDirective.SubnetID
   UsingForDirective.EnumerableSet.UintSet
   UsingForDirective.EnumerableSet.AddressSet
   UsingForDirective.EnumerableSet.Bytes32Set
   FunctionDefinition.getValidatorConfigurationNumbers
   FunctionDefinition.getCommitSha
   FunctionDefinition.bottomUpNonce
   FunctionDefinition.totalSubnets
   FunctionDefinition.maxMsgsPerBottomUpBatch
   FunctionDefinition.bottomUpCheckPeriod
   FunctionDefinition.getNetworkName
   FunctionDefinition.bottomUpCheckpoint
   FunctionDefinition.bottomUpMsgBatch
   FunctionDefinition.getParentFinality
   FunctionDefinition.getLatestParentFinality
   FunctionDefinition.getSubnet
   FunctionDefinition.subnets
   FunctionDefinition.getSubnetTopDownMsgsLength
   FunctionDefinition.getTopDownNonce
   FunctionDefinition.getAppliedBottomUpNonce
   FunctionDefinition.appliedTopDownNonce
   FunctionDefinition.postbox
   FunctionDefinition.postboxMsgs
   FunctionDefinition.majorityPercentage
   FunctionDefinition.listSubnets
   FunctionDefinition.getSubnetKeys
   FunctionDefinition.getLastMembership
   FunctionDefinition.getLastConfigurationNumber
   FunctionDefinition.getCurrentMembership
   FunctionDefinition.getCurrentConfigurationNumber
   FunctionDefinition.getCheckpointInfo
   FunctionDefinition.getCheckpointCurrentWeight
   FunctionDefinition.getIncompleteCheckpointHeights
   FunctionDefinition.getIncompleteCheckpoints
   FunctionDefinition.getCheckpointRetentionHeight
   FunctionDefinition.getQuorumThreshold
   FunctionDefinition.getCheckpointSignatureBundle
   FunctionDefinition.getCurrentBottomUpCheckpoint
   
   Suggested order:
   UsingForDirective.SubnetID
   UsingForDirective.EnumerableSet.UintSet
   UsingForDirective.EnumerableSet.AddressSet
   UsingForDirective.EnumerableSet.Bytes32Set
   VariableDeclaration.s
   FunctionDefinition.getValidatorConfigurationNumbers
   FunctionDefinition.getCommitSha
   FunctionDefinition.bottomUpNonce
   FunctionDefinition.totalSubnets
   FunctionDefinition.maxMsgsPerBottomUpBatch
   FunctionDefinition.bottomUpCheckPeriod
   FunctionDefinition.getNetworkName
   FunctionDefinition.bottomUpCheckpoint
   FunctionDefinition.bottomUpMsgBatch
   FunctionDefinition.getParentFinality
   FunctionDefinition.getLatestParentFinality
   FunctionDefinition.getSubnet
   FunctionDefinition.subnets
   FunctionDefinition.getSubnetTopDownMsgsLength
   FunctionDefinition.getTopDownNonce
   FunctionDefinition.getAppliedBottomUpNonce
   FunctionDefinition.appliedTopDownNonce
   FunctionDefinition.postbox
   FunctionDefinition.postboxMsgs
   FunctionDefinition.majorityPercentage
   FunctionDefinition.listSubnets
   FunctionDefinition.getSubnetKeys
   FunctionDefinition.getLastMembership
   FunctionDefinition.getLastConfigurationNumber
   FunctionDefinition.getCurrentMembership
   FunctionDefinition.getCurrentConfigurationNumber
   FunctionDefinition.getCheckpointInfo
   FunctionDefinition.getCheckpointCurrentWeight
   FunctionDefinition.getIncompleteCheckpointHeights
   FunctionDefinition.getIncompleteCheckpoints
   FunctionDefinition.getCheckpointRetentionHeight
   FunctionDefinition.getQuorumThreshold
   FunctionDefinition.getCheckpointSignatureBundle
   FunctionDefinition.getCurrentBottomUpCheckpoint

```

```solidity
File: contracts/lib/LibActivity.sol

1: 
   Current order:
   VariableDeclaration.CONSENSUS_NAMESPACE
   UsingForDirective.SubnetID
   UsingForDirective.EnumerableSet.AddressSet
   UsingForDirective.EnumerableSet.Bytes32Set
   UserDefinedValueTypeDefinition.SubnetKey
   StructDefinition.ConsensusTracker
   StructDefinition.ConsensusPendingAtHeight
   StructDefinition.ConsensusStorage
   StructDefinition.ListPendingReturnEntry
   FunctionDefinition.ensureValidProof
   FunctionDefinition.recordActivityRollup
   FunctionDefinition.listPendingConsensus
   FunctionDefinition.processConsensusClaim
   FunctionDefinition.setRewarder
   FunctionDefinition.consensusStorage
   
   Suggested order:
   UsingForDirective.SubnetID
   UsingForDirective.EnumerableSet.AddressSet
   UsingForDirective.EnumerableSet.Bytes32Set
   VariableDeclaration.CONSENSUS_NAMESPACE
   StructDefinition.ConsensusTracker
   StructDefinition.ConsensusPendingAtHeight
   StructDefinition.ConsensusStorage
   StructDefinition.ListPendingReturnEntry
   FunctionDefinition.ensureValidProof
   FunctionDefinition.recordActivityRollup
   FunctionDefinition.listPendingConsensus
   FunctionDefinition.processConsensusClaim
   FunctionDefinition.setRewarder
   FunctionDefinition.consensusStorage

```

```solidity
File: contracts/lib/LibDiamond.sol

1: 
   Current order:
   VariableDeclaration.DIAMOND_STORAGE_POSITION
   ErrorDefinition.InvalidAddress
   ErrorDefinition.NoBytecodeAtAddress
   ErrorDefinition.IncorrectFacetCutAction
   ErrorDefinition.NoSelectorsProvidedForFacetForCut
   ErrorDefinition.CannotAddFunctionToDiamondThatAlreadyExists
   ErrorDefinition.CannotAddSelectorsToZeroAddress
   ErrorDefinition.InitializationFunctionReverted
   ErrorDefinition.NoSelectorsGivenToAdd
   ErrorDefinition.NotContractOwner
   ErrorDefinition.CannotReplaceFunctionsFromFacetWithZeroAddress
   ErrorDefinition.CannotReplaceImmutableFunction
   ErrorDefinition.CannotReplaceFunctionWithTheSameFunctionFromTheSameFacet
   ErrorDefinition.CannotReplaceFunctionThatDoesNotExists
   ErrorDefinition.RemoveFacetAddressMustBeZeroAddress
   ErrorDefinition.CannotRemoveFunctionThatDoesNotExist
   ErrorDefinition.CannotRemoveImmutableFunction
   EventDefinition.OwnershipTransferred
   StructDefinition.FacetAddressAndSelectorPosition
   StructDefinition.DiamondStorage
   FunctionDefinition.transferOwnership
   FunctionDefinition.diamondStorage
   FunctionDefinition.setContractOwner
   FunctionDefinition.contractOwner
   ModifierDefinition.onlyOwner
   FunctionDefinition.enforceIsContractOwner
   FunctionDefinition.diamondCut
   FunctionDefinition.addFunctions
   FunctionDefinition.replaceFunctions
   FunctionDefinition.removeFunctions
   FunctionDefinition.initializeDiamondCut
   FunctionDefinition.enforceHasContractCode
   
   Suggested order:
   VariableDeclaration.DIAMOND_STORAGE_POSITION
   StructDefinition.FacetAddressAndSelectorPosition
   StructDefinition.DiamondStorage
   ErrorDefinition.InvalidAddress
   ErrorDefinition.NoBytecodeAtAddress
   ErrorDefinition.IncorrectFacetCutAction
   ErrorDefinition.NoSelectorsProvidedForFacetForCut
   ErrorDefinition.CannotAddFunctionToDiamondThatAlreadyExists
   ErrorDefinition.CannotAddSelectorsToZeroAddress
   ErrorDefinition.InitializationFunctionReverted
   ErrorDefinition.NoSelectorsGivenToAdd
   ErrorDefinition.NotContractOwner
   ErrorDefinition.CannotReplaceFunctionsFromFacetWithZeroAddress
   ErrorDefinition.CannotReplaceImmutableFunction
   ErrorDefinition.CannotReplaceFunctionWithTheSameFunctionFromTheSameFacet
   ErrorDefinition.CannotReplaceFunctionThatDoesNotExists
   ErrorDefinition.RemoveFacetAddressMustBeZeroAddress
   ErrorDefinition.CannotRemoveFunctionThatDoesNotExist
   ErrorDefinition.CannotRemoveImmutableFunction
   EventDefinition.OwnershipTransferred
   ModifierDefinition.onlyOwner
   FunctionDefinition.transferOwnership
   FunctionDefinition.diamondStorage
   FunctionDefinition.setContractOwner
   FunctionDefinition.contractOwner
   FunctionDefinition.enforceIsContractOwner
   FunctionDefinition.diamondCut
   FunctionDefinition.addFunctions
   FunctionDefinition.replaceFunctions
   FunctionDefinition.removeFunctions
   FunctionDefinition.initializeDiamondCut
   FunctionDefinition.enforceHasContractCode

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

1: 
   Current order:
   FunctionDefinition.appStorage
   VariableDeclaration.s
   UsingForDirective.FilAddress
   UsingForDirective.FilAddress
   UsingForDirective.AccountHelper
   FunctionDefinition._systemActorOnly
   ModifierDefinition.systemActorOnly
   
   Suggested order:
   UsingForDirective.FilAddress
   UsingForDirective.FilAddress
   UsingForDirective.AccountHelper
   VariableDeclaration.s
   ModifierDefinition.systemActorOnly
   FunctionDefinition.appStorage
   FunctionDefinition._systemActorOnly

```

```solidity
File: contracts/lib/LibPausable.sol

1: 
   Current order:
   VariableDeclaration.NAMESPACE
   StructDefinition.PausableStorage
   EventDefinition.Paused
   EventDefinition.Unpaused
   ErrorDefinition.EnforcedPause
   ErrorDefinition.ExpectedPause
   ModifierDefinition.whenNotPaused
   ModifierDefinition.whenPaused
   FunctionDefinition._requireNotPaused
   FunctionDefinition._requirePaused
   FunctionDefinition._paused
   FunctionDefinition._pause
   FunctionDefinition._unpause
   FunctionDefinition.pausableStorage
   
   Suggested order:
   VariableDeclaration.NAMESPACE
   StructDefinition.PausableStorage
   ErrorDefinition.EnforcedPause
   ErrorDefinition.ExpectedPause
   EventDefinition.Paused
   EventDefinition.Unpaused
   ModifierDefinition.whenNotPaused
   ModifierDefinition.whenPaused
   FunctionDefinition._requireNotPaused
   FunctionDefinition._requirePaused
   FunctionDefinition._paused
   FunctionDefinition._pause
   FunctionDefinition._unpause
   FunctionDefinition.pausableStorage

```

```solidity
File: contracts/lib/LibReentrancyGuard.sol

1: 
   Current order:
   VariableDeclaration.NAMESPACE
   StructDefinition.ReentrancyStorage
   ErrorDefinition.ReentrancyError
   VariableDeclaration._NOT_ENTERED
   VariableDeclaration._ENTERED
   ModifierDefinition.nonReentrant
   FunctionDefinition.reentrancyStorage
   
   Suggested order:
   VariableDeclaration.NAMESPACE
   VariableDeclaration._NOT_ENTERED
   VariableDeclaration._ENTERED
   StructDefinition.ReentrancyStorage
   ErrorDefinition.ReentrancyError
   ModifierDefinition.nonReentrant
   FunctionDefinition.reentrancyStorage

```

```solidity
File: contracts/lib/LibStaking.sol

1: 
   Current order:
   FunctionDefinition.push
   FunctionDefinition.compact
   UsingForDirective.Address
   UsingForDirective.AddressStakingReleases
   EventDefinition.NewCollateralRelease
   FunctionDefinition.setLockDuration
   FunctionDefinition.addNewRelease
   FunctionDefinition.claim
   UsingForDirective.MinPQ
   UsingForDirective.MaxPQ
   EventDefinition.ActiveValidatorCollateralUpdated
   EventDefinition.WaitingValidatorCollateralUpdated
   EventDefinition.NewActiveValidator
   EventDefinition.NewWaitingValidator
   EventDefinition.ActiveValidatorReplaced
   EventDefinition.ActiveValidatorLeft
   EventDefinition.WaitingValidatorLeft
   FunctionDefinition.getPower
   FunctionDefinition.getTotalConfirmedCollateral
   FunctionDefinition.totalActiveValidators
   FunctionDefinition.getConfirmedCollateral
   FunctionDefinition.listActiveValidators
   FunctionDefinition.listWaitingValidators
   FunctionDefinition.getTotalActivePower
   FunctionDefinition.getTotalCollateral
   FunctionDefinition.getTotalPowerOfValidators
   FunctionDefinition.isActiveValidator
   FunctionDefinition.setMetadata
   FunctionDefinition.recordDeposit
   FunctionDefinition.recordWithdraw
   FunctionDefinition.confirmFederatedPower
   FunctionDefinition.confirmDeposit
   FunctionDefinition.confirmWithdraw
   FunctionDefinition.increaseReshuffle
   FunctionDefinition.reduceReshuffle
   UsingForDirective.StakingReleaseQueue
   UsingForDirective.StakingChangeLog
   UsingForDirective.ValidatorSet
   UsingForDirective.Asset
   UsingForDirective.MaxPQ
   UsingForDirective.MinPQ
   UsingForDirective.Address
   VariableDeclaration.INITIAL_CONFIGURATION_NUMBER
   EventDefinition.ConfigurationNumberConfirmed
   EventDefinition.CollateralClaimed
   FunctionDefinition.getPower
   FunctionDefinition.isActiveValidator
   FunctionDefinition.isWaitingValidator
   FunctionDefinition.isValidator
   FunctionDefinition.hasStaked
   FunctionDefinition.totalActiveValidators
   FunctionDefinition.totalValidators
   FunctionDefinition.listActiveValidators
   FunctionDefinition.listWaitingValidators
   FunctionDefinition.getTotalConfirmedCollateral
   FunctionDefinition.getTotalCollateral
   FunctionDefinition.totalValidatorCollateral
   FunctionDefinition.setFederatedPowerWithConfirm
   FunctionDefinition.setMetadataWithConfirm
   FunctionDefinition.depositWithConfirm
   FunctionDefinition.withdrawWithConfirm
   FunctionDefinition.setFederatedPower
   FunctionDefinition.setValidatorMetadata
   FunctionDefinition.deposit
   FunctionDefinition.withdraw
   FunctionDefinition.claimCollateral
   FunctionDefinition.getConfigurationNumbers
   FunctionDefinition.confirmChange
   UsingForDirective.ValidatorSet
   UsingForDirective.StakingChangeLog
   FunctionDefinition.storeChange
   FunctionDefinition.batchStoreChange
   FunctionDefinition.confirmChange
   
   Suggested order:
   UsingForDirective.Address
   UsingForDirective.AddressStakingReleases
   UsingForDirective.MinPQ
   UsingForDirective.MaxPQ
   UsingForDirective.StakingReleaseQueue
   UsingForDirective.StakingChangeLog
   UsingForDirective.ValidatorSet
   UsingForDirective.Asset
   UsingForDirective.MaxPQ
   UsingForDirective.MinPQ
   UsingForDirective.Address
   UsingForDirective.ValidatorSet
   UsingForDirective.StakingChangeLog
   VariableDeclaration.INITIAL_CONFIGURATION_NUMBER
   EventDefinition.NewCollateralRelease
   EventDefinition.ActiveValidatorCollateralUpdated
   EventDefinition.WaitingValidatorCollateralUpdated
   EventDefinition.NewActiveValidator
   EventDefinition.NewWaitingValidator
   EventDefinition.ActiveValidatorReplaced
   EventDefinition.ActiveValidatorLeft
   EventDefinition.WaitingValidatorLeft
   EventDefinition.ConfigurationNumberConfirmed
   EventDefinition.CollateralClaimed
   FunctionDefinition.push
   FunctionDefinition.compact
   FunctionDefinition.setLockDuration
   FunctionDefinition.addNewRelease
   FunctionDefinition.claim
   FunctionDefinition.getPower
   FunctionDefinition.getTotalConfirmedCollateral
   FunctionDefinition.totalActiveValidators
   FunctionDefinition.getConfirmedCollateral
   FunctionDefinition.listActiveValidators
   FunctionDefinition.listWaitingValidators
   FunctionDefinition.getTotalActivePower
   FunctionDefinition.getTotalCollateral
   FunctionDefinition.getTotalPowerOfValidators
   FunctionDefinition.isActiveValidator
   FunctionDefinition.setMetadata
   FunctionDefinition.recordDeposit
   FunctionDefinition.recordWithdraw
   FunctionDefinition.confirmFederatedPower
   FunctionDefinition.confirmDeposit
   FunctionDefinition.confirmWithdraw
   FunctionDefinition.increaseReshuffle
   FunctionDefinition.reduceReshuffle
   FunctionDefinition.getPower
   FunctionDefinition.isActiveValidator
   FunctionDefinition.isWaitingValidator
   FunctionDefinition.isValidator
   FunctionDefinition.hasStaked
   FunctionDefinition.totalActiveValidators
   FunctionDefinition.totalValidators
   FunctionDefinition.listActiveValidators
   FunctionDefinition.listWaitingValidators
   FunctionDefinition.getTotalConfirmedCollateral
   FunctionDefinition.getTotalCollateral
   FunctionDefinition.totalValidatorCollateral
   FunctionDefinition.setFederatedPowerWithConfirm
   FunctionDefinition.setMetadataWithConfirm
   FunctionDefinition.depositWithConfirm
   FunctionDefinition.withdrawWithConfirm
   FunctionDefinition.setFederatedPower
   FunctionDefinition.setValidatorMetadata
   FunctionDefinition.deposit
   FunctionDefinition.withdraw
   FunctionDefinition.claimCollateral
   FunctionDefinition.getConfigurationNumbers
   FunctionDefinition.confirmChange
   FunctionDefinition.storeChange
   FunctionDefinition.batchStoreChange
   FunctionDefinition.confirmChange

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

1: 
   Current order:
   FunctionDefinition.appStorage
   VariableDeclaration.s
   FunctionDefinition._onlyGateway
   FunctionDefinition._notKilled
   ModifierDefinition.onlyGateway
   ModifierDefinition.notKilled
   
   Suggested order:
   VariableDeclaration.s
   ModifierDefinition.onlyGateway
   ModifierDefinition.notKilled
   FunctionDefinition.appStorage
   FunctionDefinition._onlyGateway
   FunctionDefinition._notKilled

```

```solidity
File: contracts/structs/Activity.sol

1: 
   Current order:
   UserDefinedValueTypeDefinition.MerkleHash
   StructDefinition.AggregatedStats
   StructDefinition.FullSummary
   StructDefinition.CompressedSummary
   StructDefinition.ValidatorData
   StructDefinition.ValidatorClaim
   
   Suggested order:
   StructDefinition.AggregatedStats
   StructDefinition.FullSummary
   StructDefinition.CompressedSummary
   StructDefinition.ValidatorData
   StructDefinition.ValidatorClaim

```

### <a name="NC-27"></a>[NC-27] Internal and private variables and functions names should begin with an underscore
According to the Solidity Style Guide, Non-`external` variable and function names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)

*Instances (197)*:
```solidity
File: contracts/GatewayDiamond.sol

24:     GatewayActorStorage internal s;

```

```solidity
File: contracts/SubnetActorDiamond.sol

23:     SubnetActorStorage internal s;

```

```solidity
File: contracts/SubnetRegistryDiamond.sol

16:     SubnetRegistryActorStorage internal s;

```

```solidity
File: contracts/gateway/GatewayGetterFacet.sol

17:     GatewayActorStorage internal s;

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

132:     function execBottomUpMsgs(IpcEnvelope[] calldata msgs, Subnet storage subnet) internal {

```

```solidity
File: contracts/lib/AssetHelper.sol

18:     function hasSupplyOfKind(address subnetActor, AssetKind compare) internal view returns (bool) {

24:     function validate(Asset memory asset) internal view {

36:     function expect(Asset memory asset, AssetKind kind) internal pure {

40:     function equals(Asset memory asset, Asset memory asset2) internal pure returns (bool) {

47:     function lock(Asset memory asset, uint256 value) internal returns (uint256) {

69:     function transferFunds(

86:     function ierc20Transfer(

101:     function performCall(

123:     function functionCallWithERC20Value(

154:     function isContract(address account) internal view returns (bool) {

164:     function functionCallWithValue(

200:     function sendValue(address payable recipient, uint256 value) internal returns (bool) {

209:     function balance(Asset memory asset) internal view returns (uint256 ret) {

218:     function balanceOf(Asset memory asset, address holder) internal view returns (uint256 ret) {

229:     function makeAvailable(Asset memory self, address spender, uint256 amount) internal returns (uint256 msgValue) {

240:     function native() internal pure returns (Asset memory) {

244:     function erc20(address token) internal pure returns (Asset memory) {

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

139:     function toHash(IpcEnvelope memory crossMsg) internal pure returns (bytes32) {

149:     function toTracingId(IpcEnvelope memory crossMsg) internal pure returns (bytes32) {

164:     function isEmpty(IpcEnvelope memory crossMsg) internal pure returns (bool) {

233:     function validateCrossMessage(

```

```solidity
File: contracts/lib/FvmAddressHelper.sol

19:     function from(address addr) internal pure returns (FvmAddress memory fvmAddress) {

28:     function toHash(FvmAddress memory fvmAddress) internal pure returns (bytes32) {

33:     function equal(FvmAddress memory a, FvmAddress memory b) internal pure returns (bool) {

40:     function extractEvmAddress(FvmAddress memory fvmAddress) internal pure returns (address addr) {

```

```solidity
File: contracts/lib/LibActivity.sol

57:     function ensureValidProof(

77:     function recordActivityRollup(

95:     function listPendingConsensus(

124:     function processConsensusClaim(

159:     function setRewarder(address rewarder) internal {

166:     function consensusStorage() internal pure returns (ConsensusStorage storage ds) {

```

```solidity
File: contracts/lib/LibDiamond.sol

48:     function transferOwnership(address newOwner) internal onlyOwner {

55:     function diamondStorage() internal pure returns (DiamondStorage storage ds) {

62:     function setContractOwner(address _newOwner) internal {

70:     function contractOwner() internal view returns (address contractOwner_) {

81:     function enforceIsContractOwner() internal view {

87:     function diamondCut(IDiamond.FacetCut[] memory _diamondCut, address _init, bytes memory _calldata) internal {

113:     function addFunctions(address _facetAddress, bytes4[] memory _functionSelectors) internal {

139:     function replaceFunctions(address _facetAddress, bytes4[] memory _functionSelectors) internal {

167:     function removeFunctions(address _facetAddress, bytes4[] memory _functionSelectors) internal {

203:     function initializeDiamondCut(address _init, bytes memory _calldata) internal {

225:     function enforceHasContractCode(address _contract, string memory _errorMessage) internal view {

```

```solidity
File: contracts/lib/LibGateway.sol

46:     function getCurrentBottomUpCheckpoint()

58:     function getBottomUpCheckpoint(

68:     function getBottomUpMsgBatch(uint256 epoch) internal view returns (bool exists, BottomUpMsgBatch storage batch) {

76:     function bottomUpCheckpointExists(uint256 epoch) internal view returns (bool) {

82:     function bottomUpBatchMsgsExists(uint256 epoch) internal view returns (bool) {

88:     function storeBottomUpCheckpoint(BottomUpCheckpoint memory checkpoint) internal {

111:     function storeBottomUpMsgBatch(BottomUpMsgBatch memory batch) internal {

131:     function getParentFinality(uint256 blockNumber) internal view returns (ParentFinality memory) {

137:     function getLatestParentFinality() internal view returns (ParentFinality memory) {

144:     function commitParentFinality(

161:     function updateMembership(Membership memory membership) internal {

212:     function membershipTotalWeight(Membership memory self) internal pure returns (uint256) {

225:     function membershipEqual(Membership memory mb1, Membership memory mb2) internal pure returns (bool) {

243:     function commitTopDownMsg(Subnet storage subnet, IpcEnvelope memory crossMessage) internal {

259:     function commitBottomUpMsg(IpcEnvelope memory crossMessage) internal {

323:     function getSubnet(address actor) internal view returns (bool found, Subnet storage subnet) {

337:     function getSubnet(SubnetID memory subnetId) internal view returns (bool found, Subnet storage subnet) {

345:     function getNextEpoch(uint256 blockNumber, uint256 checkPeriod) internal pure returns (uint256) {

353:     function applyMessages(SubnetID memory arrivingFrom, IpcEnvelope[] memory crossMsgs) internal {

366:     function applyTopDownMessages(SubnetID memory arrivingFrom, IpcEnvelope[] memory crossMsgs) internal {

384:     function applyMsg(SubnetID memory arrivingFrom, IpcEnvelope memory crossMsg) internal {

397:     function applyMsg(SubnetID memory arrivingFrom, IpcEnvelope memory crossMsg, bool expectTopDownOnly) internal {

492:     function executeCrossMsg(

513:     function sendReceipt(IpcEnvelope memory original, OutcomeType outcomeType, bytes memory ret) internal {

538:     function commitValidatedCrossMessage(IpcEnvelope memory crossMessage) internal returns (bool shouldBurn) {

567:     function crossMsgSideEffects(uint256 v, bool shouldBurn) internal {

575:     function checkMsgLength(IpcEnvelope[] calldata msgs) internal view {

585:     function checkSubnetsSupplyCompatible(

624:     function validateCrossMessage(IpcEnvelope memory envelope) internal view returns (bool, InvalidXnetMessageReason) {

630:     function checkCrossMessage(

692:     function propagateAllPostboxMessages() internal {

713:     function propagatePostboxMessage(bytes32 msgCid) internal {

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

81:     function appStorage() internal pure returns (GatewayActorStorage storage ds) {

90:     GatewayActorStorage internal s;

```

```solidity
File: contracts/lib/LibMultisignatureChecker.sol

30:     function isValidWeightedMultiSignature(

```

```solidity
File: contracts/lib/LibPausable.sol

110:     function pausableStorage() private pure returns (PausableStorage storage ds) {

```

```solidity
File: contracts/lib/LibQuorum.sol

23:     function addQuorumSignature(

93:     function createQuorumInfo(

133:     function pruneQuorums(QuorumMap storage self, uint256 newRetentionHeight) internal {

163:     function isHeightAlreadyProcessed(QuorumMap storage self, uint256 height) internal view {

171:     function weightNeeded(uint256 weight, uint256 majorityPercentage) internal pure returns (uint256) {

```

```solidity
File: contracts/lib/LibReentrancyGuard.sol

27:     function reentrancyStorage() private pure returns (ReentrancyStorage storage ds) {

```

```solidity
File: contracts/lib/LibStaking.sol

17:     function push(AddressStakingReleases storage self, StakingRelease memory release) internal {

28:     function compact(AddressStakingReleases storage self) internal returns (uint256, uint16) {

69:     function setLockDuration(StakingReleaseQueue storage self, uint256 blocks) internal {

74:     function addNewRelease(StakingReleaseQueue storage self, address validator, uint256 amount) internal {

84:     function claim(StakingReleaseQueue storage self, address validator) internal returns (uint256) {

109:     function getPower(ValidatorSet storage validators, address validator) internal view returns (uint256 power) {

118:     function getTotalConfirmedCollateral(ValidatorSet storage validators) internal view returns (uint256 collateral) {

123:     function totalActiveValidators(ValidatorSet storage validators) internal view returns (uint16 total) {

128:     function getConfirmedCollateral(

135:     function listActiveValidators(ValidatorSet storage validators) internal view returns (address[] memory addresses) {

147:     function listWaitingValidators(ValidatorSet storage validators) internal view returns (address[] memory addresses) {

160:     function getTotalActivePower(ValidatorSet storage validators) internal view returns (uint256 collateral) {

172:     function getTotalCollateral(ValidatorSet storage validators) internal view returns (uint256 collateral) {

186:     function getTotalPowerOfValidators(

205:     function isActiveValidator(ValidatorSet storage self, address validator) internal view returns (bool) {

210:     function setMetadata(ValidatorSet storage validators, address validator, bytes calldata metadata) internal {

219:     function recordDeposit(ValidatorSet storage validators, address validator, uint256 amount) internal {

224:     function recordWithdraw(ValidatorSet storage validators, address validator, uint256 amount) internal {

235:     function confirmFederatedPower(ValidatorSet storage self, address validator, uint256 power) internal {

248:     function confirmDeposit(ValidatorSet storage self, address validator, uint256 amount) internal {

257:     function confirmWithdraw(ValidatorSet storage self, address validator, uint256 amount) internal {

273:     function increaseReshuffle(ValidatorSet storage self, address maybeActive, uint256 newPower) internal {

325:     function reduceReshuffle(ValidatorSet storage self, address validator, uint256 newPower) internal {

395:     function getPower(address validator) internal view returns (uint256 power) {

401:     function isActiveValidator(address validator) internal view returns (bool) {

407:     function isWaitingValidator(address validator) internal view returns (bool) {

415:     function isValidator(address addr) internal view returns (bool) {

422:     function hasStaked(address validator) internal view returns (bool) {

429:     function totalActiveValidators() internal view returns (uint16) {

435:     function totalValidators() internal view returns (uint16) {

441:     function listActiveValidators() internal view returns (address[] memory addresses) {

447:     function listWaitingValidators() internal view returns (address[] memory addresses) {

452:     function getTotalConfirmedCollateral() internal view returns (uint256) {

457:     function getTotalCollateral() internal view returns (uint256) {

463:     function totalValidatorCollateral(address validator) internal view returns (uint256) {

471:     function setFederatedPowerWithConfirm(address validator, uint256 power) internal {

477:     function setMetadataWithConfirm(address validator, bytes calldata metadata) internal {

483:     function depositWithConfirm(address validator, uint256 amount) internal {

520:     function withdrawWithConfirm(address validator, uint256 amount) internal {

531:     function setFederatedPower(address validator, bytes calldata metadata, uint256 amount) internal {

537:     function setValidatorMetadata(address validator, bytes calldata metadata) internal {

543:     function deposit(address validator, uint256 amount) internal {

551:     function withdraw(address validator, uint256 amount) internal {

561:     function claimCollateral(address validator) internal returns (uint256 amount) {

567:     function getConfigurationNumbers() internal view returns (uint64, uint64) {

573:     function confirmChange(uint64 configurationNumber) internal {

629:     function storeChange(ParentValidatorsTracker storage self, StakingChangeRequest calldata changeRequest) internal {

641:     function batchStoreChange(

659:     function confirmChange(ParentValidatorsTracker storage self, uint64 configurationNumber) internal {

```

```solidity
File: contracts/lib/LibStakingChangeLog.sol

11:     function metadataRequest(StakingChangeLog storage changes, address validator, bytes calldata metadata) internal {

28:     function federatedPowerRequest(

54:     function withdrawRequest(StakingChangeLog storage changes, address validator, uint256 amount) internal {

73:     function depositRequest(StakingChangeLog storage changes, address validator, uint256 amount) internal {

92:     function recordChange(

106:     function getChange(

113:     function purgeChange(StakingChangeLog storage changes, uint64 configurationNumber) internal {

```

```solidity
File: contracts/lib/LibSubnetActor.sol

26:     function enforceCollateralValidation() internal view {

37:     function enforceFederatedValidation() internal view {

47:     function gateValidatorPowerDelta(address validator, uint256 oldPower, uint256 newPower) internal {

60:     function gateValidatorNewPowers(address[] calldata validators, uint256[] calldata newPowers) internal {

84:     function bootstrapSubnetIfNeeded() internal {

103:     function publicKeyToAddress(bytes calldata publicKey) internal pure returns (address) {

114:     function preBootstrapSetFederatedPower(

159:     function registerInGateway(uint256 collateral) internal {

176:     function postBootstrapSetFederatedPower(

204:     function rmAddressFromBalanceKey(address addr) internal {

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

71:     function appStorage() internal pure returns (SubnetActorStorage storage ds) {

80:     SubnetActorStorage internal s;

```

```solidity
File: contracts/lib/priority/LibMaxPQ.sol

18:     function getSize(MaxPQ storage self) internal view returns (uint16) {

22:     function getAddress(MaxPQ storage self, uint16 i) internal view returns (address) {

26:     function contains(MaxPQ storage self, address validator) internal view returns (bool) {

32:     function insert(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

46:     function pop(MaxPQ storage self, ValidatorSet storage validators) internal {

62:     function deleteReheapify(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

87:     function increaseReheapify(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

95:     function decreaseReheapify(MaxPQ storage self, ValidatorSet storage validators, address validator) internal {

103:     function max(MaxPQ storage self, ValidatorSet storage validators) internal view returns (address, uint256) {

114:     function swim(MaxPQ storage self, ValidatorSet storage validators, uint16 pos, uint256 value) internal {

132:     function sink(MaxPQ storage self, ValidatorSet storage validators, uint16 pos, uint256 value) internal {

163:     function largerPosition(

178:     function firstValueSmaller(uint256 v1, uint256 v2) internal pure returns (bool) {

```

```solidity
File: contracts/lib/priority/LibMinPQ.sol

17:     function getSize(MinPQ storage self) internal view returns (uint16) {

21:     function getAddress(MinPQ storage self, uint16 i) internal view returns (address) {

25:     function contains(MinPQ storage self, address validator) internal view returns (bool) {

31:     function insert(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

44:     function pop(MinPQ storage self, ValidatorSet storage validators) internal {

59:     function deleteReheapify(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

83:     function increaseReheapify(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

90:     function decreaseReheapify(MinPQ storage self, ValidatorSet storage validators, address validator) internal {

98:     function min(MinPQ storage self, ValidatorSet storage validators) internal view returns (address, uint256) {

109:     function swim(MinPQ storage self, ValidatorSet storage validators, uint16 pos, uint256 value) internal {

128:     function sink(MinPQ storage self, ValidatorSet storage validators, uint16 pos, uint256 value) internal {

159:     function smallerPosition(

174:     function firstValueLarger(uint256 v1, uint256 v2) internal pure returns (bool) {

```

```solidity
File: contracts/lib/priority/LibPQ.sol

23:     function isEmpty(PQ storage self) internal view returns (bool) {

27:     function requireNotEmpty(PQ storage self) internal view {

33:     function getSize(PQ storage self) internal view returns (uint16) {

37:     function contains(PQ storage self, address validator) internal view returns (bool) {

41:     function getPosOrRevert(PQ storage self, address validator) internal view returns (uint16 pos) {

48:     function del(PQ storage self, uint16 pos) internal {

54:     function getPower(PQ storage self, ValidatorSet storage validators, uint16 pos) internal view returns (uint256) {

59:     function getConfirmedCollateral(

68:     function exchange(PQ storage self, uint16 pos1, uint16 pos2) internal {

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

91:     function ensureValidCheckpoint(BottomUpCheckpoint calldata checkpoint) internal view {

```

```solidity
File: contracts/subnet/SubnetActorGetterFacet.sol

20:     SubnetActorStorage internal s;

```

```solidity
File: contracts/subnetregistry/RegisterSubnetFacet.sol

15:     SubnetRegistryActorStorage internal s;

101:     function ensurePrivileges() internal view {

```

```solidity
File: contracts/subnetregistry/SubnetGetterFacet.sol

9:     SubnetRegistryActorStorage internal s;

```

### <a name="NC-28"></a>[NC-28] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (27)*:
```solidity
File: contracts/gateway/GatewayManagerFacet.sol

30:     event SubnetDestroyed(SubnetID id);

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

27:     event CheckpointCommitted(address indexed subnet, uint256 subnetHeight);

```

```solidity
File: contracts/interfaces/IDiamond.sol

18:     event DiamondCut(FacetCut[] _diamondCut, address _init, bytes _calldata);

```

```solidity
File: contracts/lib/LibDiamond.sol

28:     event OwnershipTransferred(address oldOwner, address newOwner);

```

```solidity
File: contracts/lib/LibGateway.sol

29:     event MembershipUpdated(Membership);

31:     event NewTopDownMessage(address indexed subnet, IpcEnvelope message, bytes32 indexed id);

40:     event MessagePropagatedFromPostbox(bytes32 id);

```

```solidity
File: contracts/lib/LibPausable.sol

16:     event Paused(address account);

21:     event Unpaused(address account);

```

```solidity
File: contracts/lib/LibQuorum.sol

14:     event QuorumReached(QuorumObjKind objKind, uint256 height, bytes32 objHash, uint256 quorumWeight);

15:     event QuorumWeightUpdated(QuorumObjKind objKind, uint256 height, bytes32 objHash, uint256 newWeight);

```

```solidity
File: contracts/lib/LibStaking.sol

67:     event NewCollateralRelease(address validator, uint256 amount, uint256 releaseBlock);

100:     event ActiveValidatorCollateralUpdated(address validator, uint256 newPower);

101:     event WaitingValidatorCollateralUpdated(address validator, uint256 newPower);

102:     event NewActiveValidator(address validator, uint256 power);

103:     event NewWaitingValidator(address validator, uint256 power);

104:     event ActiveValidatorReplaced(address oldValidator, address newValidator);

105:     event ActiveValidatorLeft(address validator);

106:     event WaitingValidatorLeft(address validator);

391:     event ConfigurationNumberConfirmed(uint64 number);

392:     event CollateralClaimed(address validator, uint256 amount);

```

```solidity
File: contracts/lib/LibStakingChangeLog.sol

8:     event NewStakingChangeRequest(StakingOperation op, address validator, bytes payload, uint64 configurationNumber);

```

```solidity
File: contracts/lib/LibSubnetActor.sol

22:     event SubnetBootstrapped(Validator[]);

```

```solidity
File: contracts/structs/Activity.sol

7: event ActivityRollupRecorded(uint64 checkpointHeight, FullActivityRollup rollup);

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

27:     event ValidatorGaterUpdated(address oldGater, address newGater);

28:     event NewBootstrapNode(string netAddress, address owner);

```

```solidity
File: contracts/subnetregistry/RegisterSubnetFacet.sol

18:     event SubnetDeployed(address subnetAddr);

```

### <a name="NC-29"></a>[NC-29] Constants should be defined rather than using magic numbers

*Instances (1)*:
```solidity
File: contracts/constants/Constants.sol

4: address constant BURNT_FUNDS_ACTOR = address(99);

```

### <a name="NC-30"></a>[NC-30] `public` functions not called by the contract should be declared `external` instead

*Instances (17)*:
```solidity
File: contracts/lib/CrossMsgHelper.sol

46:     function createCallMsg(

69:     function createResultMsg(

95:     function createReleaseMsg(

110:     function createFundMsg(

124:     function applyType(IpcEnvelope calldata message, SubnetID calldata currentSubnet) public pure returns (IPCMsgType) {

143:     function toHash(IpcEnvelope[] memory crossMsgs) public pure returns (bytes32) {

181:     function execute(

```

```solidity
File: contracts/lib/SubnetIDHelper.sol

16:     function getAddress(SubnetID memory subnet) public pure returns (address) {

25:     function getParentSubnet(SubnetID memory subnet) public pure returns (SubnetID memory) {

42:     function toString(SubnetID calldata subnet) public pure returns (string memory) {

60:     function createSubnetId(SubnetID calldata subnet, address actor) public pure returns (SubnetID memory newSubnet) {

74:     function getActor(SubnetID calldata subnet) public pure returns (address) {

82:     function isRoot(SubnetID calldata subnet) public pure returns (bool) {

87:     function equals(SubnetID calldata subnet1, SubnetID calldata subnet2) public pure returns (bool) {

99:     function commonParent(SubnetID calldata subnet1, SubnetID calldata subnet2) public pure returns (SubnetID memory) {

132:     function down(SubnetID calldata subnet1, SubnetID calldata subnet2) public pure returns (bool, SubnetID memory) {

162:     function isEmpty(SubnetID calldata subnetId) public pure returns (bool) {

```

### <a name="NC-31"></a>[NC-31] Variables need not be initialized to zero
The default value for variables is zero, so initializing them to zero is superfluous.

*Instances (6)*:
```solidity
File: contracts/lib/LibActivity.sol

66:         for (uint256 i = 0; i < proof.length; i++) {

106:         for (uint256 i = 0; i < size; i++) {

110:             for (uint256 j = 0; j < heights.length; j++) {

```

```solidity
File: contracts/lib/LibGateway.sol

699:         for (uint256 i = 0; i < keysLength; ) {

```

```solidity
File: contracts/lib/LibSubnetActor.sol

164:         uint256 msgValue = 0;

```

```solidity
File: contracts/subnet/SubnetActorActivityFacet.sol

26:         for (uint256 i = 0; i < len; ) {

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Some tokens may revert when zero value transfers are made | 1 |
| [L-2](#L-2) | `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 1 |
| [L-3](#L-3) | Division by zero not prevented | 1 |
| [L-4](#L-4) | Duplicate import statements | 33 |
| [L-5](#L-5) | Empty `receive()/payable fallback()` function does not authenticate requests | 1 |
| [L-6](#L-6) | External calls in an un-bounded `for-`loop may result in a DOS | 3 |
| [L-7](#L-7) | External call recipient may consume all transaction gas | 3 |
| [L-8](#L-8) | Fallback lacking `payable` | 8 |
| [L-9](#L-9) | Solidity version 0.8.20+ may not work on other chains due to `PUSH0` | 48 |
| [L-10](#L-10) | Use `Ownable2Step.transferOwnership` instead of `Ownable.transferOwnership` | 3 |
| [L-11](#L-11) | Consider using OpenZeppelin's SafeCast library to prevent unexpected overflows when downcasting | 1 |
| [L-12](#L-12) | Upgradeable contract not initialized | 2 |
### <a name="L-1"></a>[L-1] Some tokens may revert when zero value transfers are made
Example: https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers.

In spite of the fact that EIP-20 [states](https://github.com/ethereum/EIPs/blob/46b9b698815abbfa628cd1097311deee77dd45c5/EIPS/eip-20.md?plain=1#L116) that zero-valued transfers must be accepted, some tokens, such as LEND will revert if this is attempted, which may cause transactions that involve other tokens (such as batch operations) to fully revert. Consider skipping the transfer if the amount is zero, which will also save gas.

*Instances (1)*:
```solidity
File: contracts/lib/AssetHelper.sol

53:             token.safeTransferFrom({from: msg.sender, to: address(this), value: value});

```

### <a name="L-2"></a>[L-2] `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (1)*:
```solidity
File: contracts/lib/AssetHelper.sol

96:                 abi.encodePacked(IERC20.transfer.selector, abi.encode(recipient, value))

```

### <a name="L-3"></a>[L-3] Division by zero not prevented
The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.

*Instances (1)*:
```solidity
File: contracts/lib/LibGateway.sol

346:         return ((uint64(blockNumber) / checkPeriod) + 1) * checkPeriod;

```

### <a name="L-4"></a>[L-4] Duplicate import statements

*Instances (33)*:
```solidity
File: contracts/GatewayDiamond.sol

9: import {Validator, Membership} from "./structs/Subnet.sol";

13: import {SubnetID} from "./structs/Subnet.sol";

```

```solidity
File: contracts/gateway/GatewayGetterFacet.sol

6: import {SubnetID, Subnet} from "../structs/Subnet.sol";

7: import {Membership} from "../structs/Subnet.sol";

```

```solidity
File: contracts/gateway/GatewayManagerFacet.sol

9: import {SubnetID, Subnet, Asset} from "../structs/Subnet.sol";

10: import {Membership, AssetKind} from "../structs/Subnet.sol";

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

5: import {BottomUpCheckpoint} from "../../structs/CrossNet.sol";

12: import {InvalidBatchSource, NotEnoughBalance, MaxMsgsPerBatchExceeded, InvalidCheckpointSource, CheckpointAlreadyExists} from "../../errors/IPCErrors.sol";

12: import {InvalidBatchSource, NotEnoughBalance, MaxMsgsPerBatchExceeded, InvalidCheckpointSource, CheckpointAlreadyExists} from "../../errors/IPCErrors.sol";

13: import {NotRegisteredSubnet, SubnetNotActive, SubnetNotFound, InvalidSubnet, CheckpointNotCreated} from "../../errors/IPCErrors.sol";

13: import {NotRegisteredSubnet, SubnetNotActive, SubnetNotFound, InvalidSubnet, CheckpointNotCreated} from "../../errors/IPCErrors.sol";

13: import {NotRegisteredSubnet, SubnetNotActive, SubnetNotFound, InvalidSubnet, CheckpointNotCreated} from "../../errors/IPCErrors.sol";

14: import {BatchNotCreated, InvalidBatchEpoch, BatchAlreadyExists, NotEnoughSubnetCircSupply, InvalidCheckpointEpoch} from "../../errors/IPCErrors.sol";

14: import {BatchNotCreated, InvalidBatchEpoch, BatchAlreadyExists, NotEnoughSubnetCircSupply, InvalidCheckpointEpoch} from "../../errors/IPCErrors.sol";

14: import {BatchNotCreated, InvalidBatchEpoch, BatchAlreadyExists, NotEnoughSubnetCircSupply, InvalidCheckpointEpoch} from "../../errors/IPCErrors.sol";

17: import {IpcEnvelope, SubnetID, IpcMsgKind} from "../../structs/CrossNet.sol";

17: import {IpcEnvelope, SubnetID, IpcMsgKind} from "../../structs/CrossNet.sol";

```

```solidity
File: contracts/gateway/router/TopDownFinalityFacet.sol

6: import {PermissionMode, Validator, ValidatorInfo, StakingChangeRequest, Membership} from "../../structs/Subnet.sol";

11: import {ParentValidatorsTracker, ValidatorSet} from "../../structs/Subnet.sol";

```

```solidity
File: contracts/gateway/router/XnetMessagingFacet.sol

9: import {Subnet} from "../../structs/Subnet.sol";

15: import {Asset} from "../../structs/Subnet.sol";

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

7: import {SubnetID, IPCAddress} from "../structs/Subnet.sol";

14: import {Asset} from "../structs/Subnet.sol";

```

```solidity
File: contracts/lib/LibGateway.sol

7: import {SubnetID, Subnet, AssetKind, Asset} from "../structs/Subnet.sol";

10: import {Membership} from "../structs/Subnet.sol";

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

7: import {SubnetID, Subnet, ParentValidatorsTracker} from "../structs/Subnet.sol";

8: import {Membership} from "../structs/Subnet.sol";

```

```solidity
File: contracts/lib/LibSubnetActor.sol

5: import {ERR_PERMISSIONED_AND_BOOTSTRAPPED} from "../errors/IPCErrors.sol";

6: import {NotEnoughGenesisValidators, DuplicatedGenesisValidator, NotOwnerOfPublicKey, MethodNotAllowed} from "../errors/IPCErrors.sol";

```

```solidity
File: contracts/subnet/SubnetActorGetterFacet.sol

6: import {SubnetID, Asset} from "../structs/Subnet.sol";

7: import {SubnetID, ValidatorInfo, Validator, PermissionMode} from "../structs/Subnet.sol";

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

5: import {ERR_VALIDATOR_JOINED, ERR_VALIDATOR_NOT_JOINED} from "../errors/IPCErrors.sol";

6: import {InvalidFederationPayload, SubnetAlreadyBootstrapped, NotEnoughFunds, CollateralIsZero, CannotReleaseZero, NotOwnerOfPublicKey, EmptyAddress, NotEnoughBalance, NotEnoughCollateral, NotValidator, NotAllValidatorsHaveLeft, InvalidPublicKeyLength, MethodNotAllowed, SubnetNotBootstrapped} from "../errors/IPCErrors.sol";

```

### <a name="L-5"></a>[L-5] Empty `receive()/payable fallback()` function does not authenticate requests
If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.

*Instances (1)*:
```solidity
File: contracts/SubnetActorDiamond.sol

154:     receive() external payable onlyGateway {

```

### <a name="L-6"></a>[L-6] External calls in an un-bounded `for-`loop may result in a DOS
Consider limiting the number of iterations in for-loops that make external calls

*Instances (3)*:
```solidity
File: contracts/lib/LibQuorum.sol

141:             address[] memory oldValidators = self.quorumSignatureSenders[h].values();

146:                 self.quorumSignatureSenders[h].remove(oldValidators[i]);

```

```solidity
File: contracts/lib/SubnetIDHelper.sol

47:             route = string.concat(route, subnet.route[i].toHexString());

```

### <a name="L-7"></a>[L-7] External call recipient may consume all transaction gas
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use `addr.call{gas: <amount>}("")` or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

*Instances (3)*:
```solidity
File: contracts/lib/AssetHelper.sol

92:             asset.tokenAddress.call(

177:         return target.call{value: value}(data);

204:         (bool success, ) = recipient.call{value: value}("");

```

### <a name="L-8"></a>[L-8] Fallback lacking `payable`

*Instances (8)*:
```solidity
File: contracts/GatewayDiamond.sol

81:     function _fallback() internal {

116:         _fallback();

121:         _fallback();

```

```solidity
File: contracts/SubnetActorDiamond.sol

114:     function _fallback() internal {

149:         _fallback();

```

```solidity
File: contracts/SubnetRegistryDiamond.sol

106:     function _fallback() internal {

141:         _fallback();

146:         _fallback();

```

### <a name="L-9"></a>[L-9] Solidity version 0.8.20+ may not work on other chains due to `PUSH0`
The compiler for Solidity 0.8.20 switches the default target EVM version to [Shanghai](https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/#important-note), which includes the new `PUSH0` op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier [EVM](https://docs.soliditylang.org/en/v0.8.20/using-the-compiler.html?ref=zaryabs.com#setting-the-evm-version-to-target) [version](https://book.getfoundry.sh/reference/config/solidity-compiler#evm_version). While the project itself may or may not compile with 0.8.20, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.

*Instances (48)*:
```solidity
File: contracts/GatewayDiamond.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/OwnershipFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/SubnetActorDiamond.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/SubnetRegistryDiamond.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/constants/Constants.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/diamond/DiamondCutFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/diamond/DiamondLoupeFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/enums/ConsensusType.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/enums/IPCMsgType.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/errors/IPCErrors.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/gateway/GatewayGetterFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/gateway/GatewayManagerFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/gateway/GatewayMessengerFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/gateway/router/TopDownFinalityFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/gateway/router/XnetMessagingFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/AccountHelper.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/AssetHelper.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/FvmAddressHelper.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibActivity.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibDiamond.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibGateway.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibGatewayActorStorage.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibMultisignatureChecker.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibQuorum.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibStaking.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibStakingChangeLog.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibSubnetActor.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibSubnetActorStorage.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/LibSubnetRegistryStorage.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/SubnetIDHelper.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/priority/LibMaxPQ.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/priority/LibMinPQ.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/lib/priority/LibPQ.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/structs/Activity.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/structs/CrossNet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/structs/FvmAddress.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/structs/Quorum.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/structs/Subnet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/subnet/SubnetActorActivityFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/subnet/SubnetActorGetterFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/subnet/SubnetActorManagerFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/subnet/SubnetActorPauseFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/subnet/SubnetActorRewardFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/subnetregistry/RegisterSubnetFacet.sol

2: pragma solidity ^0.8.23;

```

```solidity
File: contracts/subnetregistry/SubnetGetterFacet.sol

2: pragma solidity ^0.8.23;

```

### <a name="L-10"></a>[L-10] Use `Ownable2Step.transferOwnership` instead of `Ownable.transferOwnership`
Use [Ownable2Step.transferOwnership](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) which is safer. Use it as it is more secure due to 2-stage ownership transfer.

**Recommended Mitigation Steps**

Use <a href="https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol">Ownable2Step.sol</a>
  
  ```solidity
      function acceptOwnership() external {
          address sender = _msgSender();
          require(pendingOwner() == sender, "Ownable2Step: caller is not the new owner");
          _transferOwnership(sender);
      }
```

*Instances (3)*:
```solidity
File: contracts/OwnershipFacet.sol

7:     function transferOwnership(address _newOwner) external {

8:         LibDiamond.transferOwnership(_newOwner);

```

```solidity
File: contracts/lib/LibDiamond.sol

48:     function transferOwnership(address newOwner) internal onlyOwner {

```

### <a name="L-11"></a>[L-11] Consider using OpenZeppelin's SafeCast library to prevent unexpected overflows when downcasting
Downcasting from `uint256`/`int256` in Solidity does not revert on overflow. This can result in undesired exploitation or bugs, since developers usually assume that overflows raise errors. [OpenZeppelin's SafeCast library](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) restores this intuition by reverting the transaction when such an operation overflows. Using this library eliminates an entire class of bugs, so it's recommended to use it always. Some exceptions are acceptable like with the classic `uint256(uint160(address(variable)))`

*Instances (1)*:
```solidity
File: contracts/lib/LibDiamond.sol

118:         uint16 selectorCount = uint16(ds.selectors.length);

```

### <a name="L-12"></a>[L-12] Upgradeable contract not initialized
Upgradeable contracts are initialized via an initializer function rather than by a constructor. Leaving such a contract uninitialized may lead to it being taken over by a malicious user

*Instances (2)*:
```solidity
File: contracts/lib/LibDiamond.sol

110:         initializeDiamondCut(_init, _calldata);

203:     function initializeDiamondCut(address _init, bytes memory _calldata) internal {

```


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Contracts are vulnerable to fee-on-transfer accounting-related issues | 1 |
| [M-2](#M-2) | `block.number` means different things on different L2s | 6 |
| [M-3](#M-3) | Centralization Risk for trusted owners | 1 |
| [M-4](#M-4) | `increaseAllowance/decreaseAllowance` won't work on mainnet for USDT | 1 |
| [M-5](#M-5) | Lack of EIP-712 compliance: using `keccak256()` directly on an array or struct variable | 12 |
| [M-6](#M-6) | Library function isn't `internal` or `private` | 22 |
### <a name="M-1"></a>[M-1] Contracts are vulnerable to fee-on-transfer accounting-related issues
Consistently check account balance before and after transfers for Fee-On-Transfer discrepancies. As arbitrary ERC20 tokens can be used, the amount here should be calculated every time to take into consideration a possible fee-on-transfer or deflation.
Also, it's a good practice for the future of the solution.

Use the balance before and after the transfer to calculate the received amount instead of assuming that it would be equal to the amount passed as a parameter. Or explicitly document that such tokens shouldn't be used and won't be supported

*Instances (1)*:
```solidity
File: contracts/lib/AssetHelper.sol

53:             token.safeTransferFrom({from: msg.sender, to: address(this), value: value});

```

### <a name="M-2"></a>[M-2] `block.number` means different things on different L2s
On Optimism, `block.number` is the L2 block number, but on Arbitrum, it's the L1 block number, and `ArbSys(address(100)).arbBlockNumber()` must be used. Furthermore, L2 block numbers often occur much more frequently than L1 block numbers (any may even occur on a per-transaction basis), so using block numbers for timing results in inconsistencies, especially when voting is involved across multiple chains. As of version 4.9, OpenZeppelin has [modified](https://blog.openzeppelin.com/introducing-openzeppelin-contracts-v4.9#governor) their governor code to use a clock rather than block numbers, to avoid these sorts of issues, but this still requires that the project [implement](https://docs.openzeppelin.com/contracts/4.x/governance#token_2) a [clock](https://eips.ethereum.org/EIPS/eip-6372) for each L2.

*Instances (6)*:
```solidity
File: contracts/gateway/GatewayManagerFacet.sol

51:         subnet.genesisEpoch = block.number;

```

```solidity
File: contracts/lib/LibGateway.sol

52:         epoch = LibGateway.getNextEpoch(block.number, s.bottomUpCheckPeriod);

261:         uint256 epoch = getNextEpoch(block.number, s.bottomUpCheckPeriod);

286:             uint256 epochCut = block.number;

```

```solidity
File: contracts/lib/LibStaking.sol

42:             if (release.releaseAt > block.number) {

75:         uint256 releaseAt = block.number + self.lockingDuration;

```

### <a name="M-3"></a>[M-3] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (1)*:
```solidity
File: contracts/lib/LibDiamond.sol

48:     function transferOwnership(address newOwner) internal onlyOwner {

```

### <a name="M-4"></a>[M-4] `increaseAllowance/decreaseAllowance` won't work on mainnet for USDT
On mainnet, the mitigation to be compatible with `increaseAllowance/decreaseAllowance` isn't applied: https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#code, meaning it reverts on setting a non-zero & non-max allowance, unless the allowance is already zero.

*Instances (1)*:
```solidity
File: contracts/lib/AssetHelper.sol

234:             token.safeIncreaseAllowance(spender, amount);

```

### <a name="M-5"></a>[M-5] Lack of EIP-712 compliance: using `keccak256()` directly on an array or struct variable
Directly using the actual variable instead of encoding the array values goes against the EIP-712 specification https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md#definition-of-encodedata. 
**Note**: OpenSea's [Seaport's example with offerHashes and considerationHashes](https://github.com/ProjectOpenSea/seaport/blob/a62c2f8f484784735025d7b03ccb37865bc39e5a/reference/lib/ReferenceGettersAndDerivers.sol#L130-L131) can be used as a reference to understand how array of structs should be encoded.

*Instances (12)*:
```solidity
File: contracts/gateway/router/CheckpointingFacet.sol

71:             objHash: keccak256(abi.encode(checkpoint)),

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

140:         return keccak256(abi.encode(crossMsg));

144:         return keccak256(abi.encode(crossMsgs));

155:                     crossMsg.to,

156:                     crossMsg.from,

```

```solidity
File: contracts/lib/FvmAddressHelper.sol

29:         return keccak256(abi.encode(fvmAddress));

```

```solidity
File: contracts/lib/LibGateway.sol

235:         bytes32 h1 = keccak256(abi.encode(mb1.validators));

236:         bytes32 h2 = keccak256(abi.encode(mb2.validators));

```

```solidity
File: contracts/lib/SubnetIDHelper.sol

57:         return keccak256(abi.encode(subnet));

```

```solidity
File: contracts/subnet/SubnetActorCheckpointingFacet.sol

35:         bytes32 checkpointHash = keccak256(abi.encode(checkpoint));

```

```solidity
File: contracts/subnet/SubnetActorGetterFacet.sol

192:         return (exists, keccak256(abi.encode(checkpoint)));

222:         return keccak256(abi.encode(messages));

```

### <a name="M-6"></a>[M-6] Library function isn't `internal` or `private`
In a library, using an external or public visibility means that we won't be going through the library with a DELEGATECALL but with a CALL. This changes the context and should be done carefully.

*Instances (22)*:
```solidity
File: contracts/lib/AccountHelper.sol

9:     function isSystemActor(address _address) external pure returns (bool) {

```

```solidity
File: contracts/lib/CrossMsgHelper.sol

29:     function createTransferMsg(

46:     function createCallMsg(

69:     function createResultMsg(

95:     function createReleaseMsg(

110:     function createFundMsg(

124:     function applyType(IpcEnvelope calldata message, SubnetID calldata currentSubnet) public pure returns (IPCMsgType) {

143:     function toHash(IpcEnvelope[] memory crossMsgs) public pure returns (bytes32) {

181:     function execute(

211:     function isSorted(IpcEnvelope[] calldata crossMsgs) external pure returns (bool) {

```

```solidity
File: contracts/lib/LibQuorum.sol

176:     function getSignatureBundle(

```

```solidity
File: contracts/lib/SubnetIDHelper.sol

16:     function getAddress(SubnetID memory subnet) public pure returns (address) {

25:     function getParentSubnet(SubnetID memory subnet) public pure returns (SubnetID memory) {

42:     function toString(SubnetID calldata subnet) public pure returns (string memory) {

56:     function toHash(SubnetID calldata subnet) public pure returns (bytes32) {

60:     function createSubnetId(SubnetID calldata subnet, address actor) public pure returns (SubnetID memory newSubnet) {

74:     function getActor(SubnetID calldata subnet) public pure returns (address) {

82:     function isRoot(SubnetID calldata subnet) public pure returns (bool) {

87:     function equals(SubnetID calldata subnet1, SubnetID calldata subnet2) public pure returns (bool) {

99:     function commonParent(SubnetID calldata subnet1, SubnetID calldata subnet2) public pure returns (SubnetID memory) {

132:     function down(SubnetID calldata subnet1, SubnetID calldata subnet2) public pure returns (bool, SubnetID memory) {

162:     function isEmpty(SubnetID calldata subnetId) public pure returns (bool) {

```
