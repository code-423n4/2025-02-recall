# ✨ So you want to run an audit

This `README.md` contains a set of checklists for our audit collaboration. This is your audit repo, which is used for scoping your audit and for providing information to wardens

Some of the checklists in this doc are for our scouts and some of them are for **you as the audit sponsor (⭐️)**.

---

# Repo setup

## ⭐️ Sponsor: Add code to this repo

- [ ] Create a PR to this repo with the below changes:
- [ ] Confirm that this repo is a self-contained repository with working commands that will build (at least) all in-scope contracts, and commands that will run tests producing gas reports for the relevant contracts.
- [ ] Please have final versions of contracts and documentation added/updated in this repo **no less than 48 business hours prior to audit start time.**
- [ ] Be prepared for a 🚨code freeze🚨 for the duration of the audit — important because it establishes a level playing field. We want to ensure everyone's looking at the same code, no matter when they look during the audit. (Note: this includes your own repo, since a PR can leak alpha to our wardens!)

## ⭐️ Sponsor: Repo checklist

- [ ] Modify the [Overview](#overview) section of this `README.md` file. Describe how your code is supposed to work with links to any relevant documentation and any other criteria/details that the auditors should keep in mind when reviewing. (Here are two well-constructed examples: [Ajna Protocol](https://github.com/code-423n4/2023-05-ajna) and [Maia DAO Ecosystem](https://github.com/code-423n4/2023-05-maia))
- [ ] Optional: pre-record a high-level overview of your protocol (not just specific smart contract functions). This saves wardens a lot of time wading through documentation.
- [ ] Review and confirm the details created by the Scout (technical reviewer) who was assigned to your contest. *Note: any files not listed as "in scope" will be considered out of scope for the purposes of judging, even if the file will be part of the deployed contracts.*  

---

# Textile audit details
- Total Prize Pool: $100000 in USDC
  - HM awards: up to XXX XXX USDC (Notion: HM (main) pool)
    - If no valid Highs or Mediums are found, the HM pool is $0 
  - QA awards: $3300 in USDC
  - Judge awards: $5000 in USDC
  - Validator awards: XXX XXX USDC (Notion: Triage fee - final)
  - Scout awards: $500 in USDC
  - (this line can be removed if there is no mitigation) Mitigation Review: XXX XXX USDC
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts XXX XXX XX 20:00 UTC (ex. `Starts March 22, 2023 20:00 UTC`)
- Ends XXX XXX XX 20:00 UTC (ex. `Ends March 30, 2023 20:00 UTC`)

**Note re: risk level upgrades/downgrades**

Two important notes about judging phase risk adjustments: 
- High- or Medium-risk submissions downgraded to Low-risk (QA) will be ineligible for awards.
- Upgrading a Low-risk finding from a QA report to a Medium- or High-risk finding is not supported.

As such, wardens are encouraged to select the appropriate risk level carefully during the submission phase.

## Automated Findings / Publicly Known Issues

The 4naly3er report can be found [here](https://github.com/code-423n4/2025-02-textile/blob/main/4naly3er-report.md).



_Note for C4 wardens: Anything included in this `Automated Findings / Publicly Known Issues` section is considered a publicly known issue and is ineligible for awards._
## 🐺 C4: Begin Gist paste here (and delete this line)





# Scope

*See [scope.txt](https://github.com/code-423n4/2025-02-textile/blob/main/scope.txt)*

### Files in scope


| File   | Logic Contracts | Interfaces | nSLOC | Purpose | Libraries used |
| ------ | --------------- | ---------- | ----- | -----   | ------------ |
| /contracts/contracts/GatewayDiamond.sol | 1| **** | 85 | ||
| /contracts/contracts/OwnershipFacet.sol | 1| **** | 10 | ||
| /contracts/contracts/SubnetActorDiamond.sol | 1| **** | 124 | |@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/contracts/SubnetRegistryDiamond.sol | 1| **** | 122 | ||
| /contracts/contracts/constants/Constants.sol | ****| **** | 6 | ||
| /contracts/contracts/diamond/DiamondCutFacet.sol | 1| **** | 9 | ||
| /contracts/contracts/diamond/DiamondLoupeFacet.sol | 1| **** | 99 | ||
| /contracts/contracts/enums/ConsensusType.sol | ****| **** | 4 | ||
| /contracts/contracts/enums/IPCMsgType.sol | ****| **** | 5 | ||
| /contracts/contracts/errors/IPCErrors.sol | ****| **** | 96 | ||
| /contracts/contracts/examples/CrossMessengerCaller.sol | 1| 1 | 47 | ||
| /contracts/contracts/examples/MintingValidatorRewarder.sol | 2| **** | 34 | |@openzeppelin/contracts/token/ERC20/ERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/access/Ownable.sol|
| /contracts/contracts/examples/SubnetValidatorGater.sol | 1| **** | 41 | |@openzeppelin/contracts/access/Ownable.sol|
| /contracts/contracts/examples/ValidatorRewarderMap.sol | 1| **** | 20 | |@openzeppelin/contracts/access/Ownable.sol|
| /contracts/contracts/gateway/GatewayGetterFacet.sol | 1| **** | 150 | |@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/gateway/GatewayManagerFacet.sol | 1| **** | 139 | |fevmate/contracts/utils/FilAddress.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/utils/Address.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/gateway/GatewayMessengerFacet.sol | 1| **** | 55 | |fevmate/contracts/utils/FilAddress.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/gateway/router/CheckpointingFacet.sol | 1| **** | 92 | |@openzeppelin/contracts/utils/Address.sol|
| /contracts/contracts/gateway/router/TopDownFinalityFacet.sol | 1| **** | 49 | |fevmate/contracts/utils/FilAddress.sol|
| /contracts/contracts/gateway/router/XnetMessagingFacet.sol | 1| **** | 22 | |fevmate/contracts/utils/FilAddress.sol|
| /contracts/contracts/interfaces/IDiamond.sol | ****| 1 | 14 | ||
| /contracts/contracts/interfaces/IDiamondCut.sol | ****| 1 | 4 | ||
| /contracts/contracts/interfaces/IDiamondLoupe.sol | ****| 1 | 7 | ||
| /contracts/contracts/interfaces/IERC165.sol | ****| 1 | 3 | ||
| /contracts/contracts/interfaces/IGateway.sol | ****| 1 | 7 | ||
| /contracts/contracts/interfaces/ISubnetActor.sol | ****| 1 | 4 | ||
| /contracts/contracts/interfaces/IValidatorGater.sol | ****| 1 | 4 | ||
| /contracts/contracts/interfaces/IValidatorRewarder.sol | ****| 1 | 5 | ||
| /contracts/contracts/lib/AccountHelper.sol | 1| **** | 7 | |fevmate/contracts/utils/FilAddress.sol|
| /contracts/contracts/lib/AssetHelper.sol | 1| **** | 121 | |@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol|
| /contracts/contracts/lib/CrossMsgHelper.sol | 1| **** | 154 | |fevmate/contracts/utils/FilAddress.sol<br>@openzeppelin/contracts/utils/Address.sol|
| /contracts/contracts/lib/FvmAddressHelper.sol | 1| **** | 45 | ||
| /contracts/contracts/lib/LibActivity.sol | 1| **** | 103 | |@openzeppelin/contracts/utils/structs/EnumerableSet.sol<br>@openzeppelin/contracts/utils/cryptography/MerkleProof.sol|
| /contracts/contracts/lib/LibDiamond.sol | 1| **** | 198 | ||
| /contracts/contracts/lib/LibGateway.sol | 1| **** | 441 | |fevmate/contracts/utils/FilAddress.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/lib/LibGatewayActorStorage.sol | 2| **** | 58 | |fevmate/contracts/utils/FilAddress.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/lib/LibMultisignatureChecker.sol | 1| **** | 44 | |@openzeppelin/contracts/utils/cryptography/ECDSA.sol|
| /contracts/contracts/lib/LibPausable.sol | 1| **** | 52 | ||
| /contracts/contracts/lib/LibQuorum.sol | 1| **** | 120 | |@openzeppelin/contracts/utils/cryptography/MerkleProof.sol<br>@openzeppelin/contracts/utils/cryptography/ECDSA.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/lib/LibReentrancyGuard.sol | 1| **** | 24 | ||
| /contracts/contracts/lib/LibStaking.sol | 5| **** | 489 | |@openzeppelin/contracts/utils/Address.sol|
| /contracts/contracts/lib/LibStakingChangeLog.sol | 1| **** | 75 | ||
| /contracts/contracts/lib/LibSubnetActor.sol | 1| **** | 135 | |@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/lib/LibSubnetActorStorage.sol | 2| **** | 63 | |@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/lib/LibSubnetRegistryStorage.sol | ****| **** | 26 | ||
| /contracts/contracts/lib/SubnetIDHelper.sol | 1| **** | 126 | |@openzeppelin/contracts/utils/Strings.sol|
| /contracts/contracts/lib/priority/LibMaxPQ.sol | 1| **** | 114 | ||
| /contracts/contracts/lib/priority/LibMinPQ.sol | 1| **** | 114 | ||
| /contracts/contracts/lib/priority/LibPQ.sol | 1| **** | 55 | ||
| /contracts/contracts/structs/Activity.sol | 1| **** | 32 | ||
| /contracts/contracts/structs/CrossNet.sol | ****| **** | 55 | |@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/structs/FvmAddress.sol | ****| **** | 10 | ||
| /contracts/contracts/structs/Quorum.sol | ****| **** | 21 | |@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/structs/Subnet.sol | ****| **** | 97 | ||
| /contracts/contracts/subnet/SubnetActorActivityFacet.sol | 1| **** | 30 | ||
| /contracts/contracts/subnet/SubnetActorCheckpointingFacet.sol | 1| **** | 66 | |@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/subnet/SubnetActorGetterFacet.sol | 1| **** | 144 | |@openzeppelin/contracts/utils/Address.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/contracts/subnet/SubnetActorManagerFacet.sol | 1| **** | 196 | |@openzeppelin/contracts/utils/structs/EnumerableSet.sol<br>@openzeppelin/contracts/utils/Address.sol|
| /contracts/contracts/subnet/SubnetActorPauseFacet.sol | 1| **** | 16 | ||
| /contracts/contracts/subnet/SubnetActorRewardFacet.sol | 1| **** | 18 | ||
| /contracts/contracts/subnetregistry/RegisterSubnetFacet.sol | 1| **** | 75 | ||
| /contracts/contracts/subnetregistry/SubnetGetterFacet.sol | 1| **** | 78 | ||
| /contracts/sdk/IpcContract.sol | 1| **** | 66 | |@openzeppelin/contracts/utils/ReentrancyGuard.sol<br>@openzeppelin/contracts/access/Ownable.sol|
| /contracts/sdk/IpcContractUpgradeable.sol | 1| **** | 72 | |@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol<br>@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol|
| /contracts/sdk/interfaces/IIpcHandler.sol | ****| 1 | 7 | ||
| /demos/axelar-token/contracts/IpcTokenHandler.sol | 1| 2 | 66 | |@axelar-network/interchain-token-service/contracts/executable/InterchainTokenExecutable.sol<br>@openzeppelin/contracts/interfaces/IERC20.sol<br>@openzeppelin/contracts/access/Ownable.sol<br>@consensus-shipyard/ipc-contracts/contracts/structs/Subnet.sol<br>@consensus-shipyard/ipc-contracts/contracts/structs/FvmAddress.sol<br>@consensus-shipyard/ipc-contracts/sdk/interfaces/IIpcHandler.sol<br>@consensus-shipyard/ipc-contracts/contracts/structs/CrossNet.sol<br>@consensus-shipyard/ipc-contracts/contracts/lib/FvmAddressHelper.sol<br>@consensus-shipyard/ipc-contracts/contracts/lib/SubnetIDHelper.sol<br>@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol|
| /demos/axelar-token/contracts/IpcTokenSender.sol | 1| **** | 34 | |@axelar-network/interchain-token-service/contracts/interfaces/IInterchainTokenService.sol<br>@axelar-network/axelar-gmp-sdk-solidity/contracts/libs/AddressBytes.sol<br>@openzeppelin/contracts/interfaces/IERC20.sol<br>@consensus-shipyard/ipc-contracts/contracts/structs/Subnet.sol|
| /demos/axelar-token/script/Deploy.s.sol | 1| **** | 52 | |forge-std/Script.sol|
| /demos/axelar-token/script/Deposit.s.sol | 1| **** | 41 | |forge-std/Script.sol<br>@openzeppelin/contracts/interfaces/IERC20.sol<br>@consensus-shipyard/ipc-contracts/contracts/structs/Subnet.sol|
| /demos/axelar-token/test/DummyERC20.sol | 1| **** | 15 | |@openzeppelin/contracts/token/ERC20/ERC20.sol<br>@openzeppelin/contracts/access/Ownable.sol|
| /demos/axelar-token/test/TestHandler.sol | 1| **** | 77 | |forge-std/Test.sol<br>@consensus-shipyard/ipc-contracts/contracts/lib/FvmAddressHelper.sol|
| /demos/axelar-token/test/TestSender.sol | 1| **** | 9 | |forge-std/Test.sol<br>@consensus-shipyard/ipc-contracts/contracts/lib/FvmAddressHelper.sol|
| /demos/linked-token/contracts/LinkedToken.sol | 1| **** | 168 | |@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol<br>@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@ipc/contracts/lib/FvmAddressHelper.sol<br>@ipc/contracts/structs/FvmAddress.sol<br>@ipc/sdk/IpcContractUpgradeable.sol<br>@ipc/contracts/structs/CrossNet.sol<br>@ipc/contracts/structs/Subnet.sol<br>@ipc/contracts/lib/CrossMsgHelper.sol<br>@ipc/contracts/lib/SubnetIDHelper.sol|
| /demos/linked-token/contracts/LinkedTokenController.sol | 1| **** | 24 | |@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@ipc/contracts/structs/Subnet.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol|
| /demos/linked-token/contracts/LinkedTokenReplica.sol | 1| **** | 31 | |@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@ipc/contracts/structs/Subnet.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol<br>@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol|
| /demos/linked-token/contracts/USDCTest.sol | 1| **** | 9 | |@openzeppelin/contracts/token/ERC20/ERC20.sol<br>@openzeppelin/contracts/access/Ownable.sol|
| /demos/linked-token/contracts/v2/LinkedTokenControllerV2.sol | 1| **** | 24 | |@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@ipc/contracts/structs/Subnet.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol|
| /demos/linked-token/contracts/v2/LinkedTokenReplicaV2.sol | 1| **** | 35 | |@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@ipc/contracts/structs/Subnet.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol<br>@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol|
| /demos/linked-token/script/ConfigManager.sol | 1| **** | 36 | |forge-std/Script.sol|
| /demos/linked-token/script/DeployIpcTokenController.s.sol | 1| **** | 46 | |@ipc/contracts/structs/Subnet.sol<br>@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol|
| /demos/linked-token/script/DeployIpcTokenReplica.s.sol | 1| **** | 54 | |@ipc/contracts/structs/Subnet.sol<br>@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol|
| /demos/linked-token/script/DeployUSDCTest.s.sol | 1| **** | 11 | ||
| /demos/linked-token/test/LinkedTokenController.t.sol | 1| **** | 128 | |forge-std/Test.sol<br>@ipc/test/IntegrationTestBase.sol<br>@ipc/contracts/GatewayDiamond.sol<br>@ipc/contracts/lib/SubnetIDHelper.sol<br>@ipc/contracts/structs/Subnet.sol<br>@ipc/contracts/lib/FvmAddressHelper.sol<br>@ipc/contracts/structs/FvmAddress.sol<br>@ipc/contracts/structs/CrossNet.sol<br>@ipc/contracts/SubnetActorDiamond.sol<br>@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol|
| /demos/linked-token/test/LinkedTokenControllerV2Extension.sol | 1| **** | 7 | ||
| /demos/linked-token/test/LinkedTokenReplica.t.sol | 1| **** | 112 | |forge-std/Test.sol<br>@ipc/test/IntegrationTestBase.sol<br>@ipc/contracts/GatewayDiamond.sol<br>@ipc/contracts/lib/SubnetIDHelper.sol<br>@ipc/contracts/structs/Subnet.sol<br>@ipc/contracts/lib/FvmAddressHelper.sol<br>@ipc/contracts/structs/FvmAddress.sol<br>@ipc/contracts/structs/CrossNet.sol<br>@ipc/contracts/SubnetActorDiamond.sol<br>@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol|
| /demos/linked-token/test/LinkedTokenReplicaV2Extension.sol | 1| **** | 7 | ||
| /demos/linked-token/test/MultiSubnetTest.t.sol | 1| **** | 334 | |@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@ipc/test/IntegrationTestBase.sol<br>@ipc/test/helpers/ERC20PresetFixedSupply.sol<br>@ipc/test/helpers/TestUtils.sol<br>@ipc/test/helpers/MerkleTreeHelper.sol<br>@ipc/test/helpers/GatewayFacetsHelper.sol<br>@ipc/test/helpers/SubnetActorFacetsHelper.sol<br>@ipc/test/helpers/ActivityHelper.sol<br>@ipc/contracts/structs/Subnet.sol<br>@ipc/contracts/SubnetActorDiamond.sol<br>@ipc/contracts/GatewayDiamond.sol<br>@ipc/contracts/gateway/router/TopDownFinalityFacet.sol<br>@ipc/contracts/gateway/router/XnetMessagingFacet.sol<br>@ipc/contracts/subnet/SubnetActorManagerFacet.sol<br>@ipc/contracts/gateway/GatewayGetterFacet.sol<br>@ipc/contracts/subnet/SubnetActorCheckpointingFacet.sol<br>@ipc/contracts/gateway/router/CheckpointingFacet.sol<br>@ipc/contracts/lib/FvmAddressHelper.sol<br>@ipc/contracts/structs/Activity.sol<br>@ipc/contracts/structs/CrossNet.sol<br>@ipc/contracts/lib/SubnetIDHelper.sol<br>@ipc/contracts/lib/CrossMsgHelper.sol<br>@ipc/sdk/interfaces/IIpcHandler.sol<br>fevmate/contracts/utils/FilAddress.sol<br>forge-std/console.sol<br>@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol|
| /ext/frc42_dispatch/hasher/src/lib.rs | ****| **** | 1 | ||
| /ext/libp2p-bitswap/src/lib.rs | ****| **** | 12 | ||
| /fendermint/actors/api/src/lib.rs | ****| **** | 1 | ||
| /fendermint/eth/api/src/conv/mod.rs | ****| **** | 3 | ||
| /fendermint/eth/api/src/handlers/mod.rs | ****| **** | 2 | ||
| /fendermint/testing/contract-test/src/ipc/mod.rs | ****| **** | 2 | ||
| /fendermint/testing/contracts/Greeter.sol | 1| **** | 16 | ||
| /fendermint/testing/contracts/QueryBlockhash.sol | 1| **** | 7 | ||
| /fendermint/testing/contracts/SimpleCoin.sol | 1| **** | 25 | ||
| /fendermint/testing/graph-test/src/lib.rs | ****| **** | **** | ||
| /fendermint/testing/materializer/tests/docker_tests/mod.rs | ****| **** | 3 | ||
| /fendermint/testing/smoke-test/src/lib.rs | ****| **** | **** | ||
| /fendermint/testing/snapshot-test/src/lib.rs | ****| **** | **** | ||
| /fendermint/vm/resolver/src/lib.rs | ****| **** | 2 | ||
| /ipc/types/src/taddress.rs | ****| **** | 118 | ||
| /ipc/types/src/uints.rs | ****| **** | 268 | ||
| **Totals** | **79** | **12** | **6584** | | |

### Files out of scope

*See [out_of_scope.txt](https://github.com/code-423n4/2025-02-textile/blob/main/out_of_scope.txt)*

| File         |
| ------------ |
| ./contracts/test/IntegrationTestBase.sol |
| ./contracts/test/IntegrationTestPresets.sol |
| ./contracts/test/examples/SubnetValidatorGater.sol |
| ./contracts/test/helpers/ActivityHelper.sol |
| ./contracts/test/helpers/DiamondFacetsHelper.sol |
| ./contracts/test/helpers/ERC20Deflationary.sol |
| ./contracts/test/helpers/ERC20Helper.sol |
| ./contracts/test/helpers/ERC20Inflationary.sol |
| ./contracts/test/helpers/ERC20Nil.sol |
| ./contracts/test/helpers/ERC20PresetFixedSupply.sol |
| ./contracts/test/helpers/FvmAddressHelper.sol |
| ./contracts/test/helpers/GatewayFacetsHelper.sol |
| ./contracts/test/helpers/MerkleTreeHelper.sol |
| ./contracts/test/helpers/RegistryFacetsHelper.sol |
| ./contracts/test/helpers/SelectorLibrary.sol |
| ./contracts/test/helpers/SubnetActorFacetsHelper.sol |
| ./contracts/test/helpers/TestUtils.sol |
| ./contracts/test/helpers/contracts/NumberContractFacetEight.sol |
| ./contracts/test/helpers/contracts/NumberContractFacetSeven.sol |
| ./contracts/test/integration/GatewayDiamond.t.sol |
| ./contracts/test/integration/GatewayDiamondToken.t.sol |
| ./contracts/test/integration/L2GatewayDiamond.t.sol |
| ./contracts/test/integration/L2PlusSubnet.t.sol |
| ./contracts/test/integration/L2PlusXNet.t.sol |
| ./contracts/test/integration/MultiSubnet.t.sol |
| ./contracts/test/integration/SubnetActorDiamond.t.sol |
| ./contracts/test/integration/SubnetRegistry.t.sol |
| ./contracts/test/invariants/GatewayActorInvariantTests.t.sol |
| ./contracts/test/invariants/GatewayActorProperties.sol |
| ./contracts/test/invariants/SubnetActorInvariants.t.sol |
| ./contracts/test/invariants/SubnetRegistryInvariants.t.sol |
| ./contracts/test/invariants/handlers/GatewayActorHandler.sol |
| ./contracts/test/invariants/handlers/SubnetActorHandler.sol |
| ./contracts/test/invariants/handlers/SubnetRegistryHandler.sol |
| ./contracts/test/mocks/AssetHelperMock.sol |
| ./contracts/test/mocks/LibGatewayMock.sol |
| ./contracts/test/mocks/SubnetActorMock.sol |
| ./contracts/test/sdk/IpcContract.t.sol |
| ./contracts/test/unit/AccountHelper.t.sol |
| ./contracts/test/unit/CrossMsgHelper.t.sol |
| ./contracts/test/unit/FvmAddressHelper.t.sol |
| ./contracts/test/unit/GenericTokenHelper.t.sol |
| ./contracts/test/unit/LibGateway.t.sol |
| ./contracts/test/unit/LibMaxPQ.t.sol |
| ./contracts/test/unit/LibMinPQ.t.sol |
| ./contracts/test/unit/LibMultisignatureChecker.t.sol |
| ./contracts/test/unit/LibValidatorSet.t.sol |
| ./contracts/test/unit/MerkleTree.t.sol |
| ./contracts/test/unit/SubnetIDHelper.t.sol |
| Totals: 49 |

