# Recall audit details
- Total Prize Pool: $100,000 in USDC
  - HM awards: $88,000 in USDC
  - QA awards: $3,300 in USDC
  - Judge awards: $5,000 in USDC
  - Validator awards: $3.200 in USDC
  - Scout awards: $500 in USDC
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts February 19, 2025 20:00 UTC
- Ends March 19, 2025 20:00 UTC

**Note re: risk level upgrades/downgrades**

Two important notes about judging phase risk adjustments: 
- High- or Medium-risk submissions downgraded to Low-risk (QA) will be ineligible for awards.
- Upgrading a Low-risk finding from a QA report to a Medium- or High-risk finding is not supported.

As such, wardens are encouraged to select the appropriate risk level carefully during the submission phase.

## Automated Findings / Publicly Known Issues

The 4naly3er report can be found [here](https://github.com/code-423n4/2025-02-3box/blob/main/4naly3er-report.md).

_Note for C4 wardens: Anything included in this `Automated Findings / Publicly Known Issues` section is considered a publicly known issue and is ineligible for awards._

### Light Audit

- Any issues outlined in the light audit are considered out-of-scope for the purposes of this contest.

### IPC Known Issues

- Any issues that are known in the original [IPC repository](https://github.com/consensus-shipyard/ipc/issues) are considered to be known issues in the Recall implementation and are thus ineligible for a reward

### TODO / FIXME Comments

- Any issue that arises from a documented `TODO` or `FIXME` comment is considered a known issue and is therefore ineligible for a reward

### Gateway

- Gateway manages funds for all registered subnets (cf. “omnibus account”). Funds sit in the subnet actor until the subnet is bootstrapped, at which point the collateral and supply is transferred to the gateway. Gateway will manage the fund and disperse to the subnet upon subnet’s request. This design will cause some security threats as one subnet can potentially impact other subnets. Please check the `Problems` section of the refactoring [proposal](https://github.com/consensus-shipyard/ipc/blob/main/specs/drafts/contract-redesign.md). The mitigation under the current design is for gateways to be single-tenant, i.e. to restrict who can create subnets under it, so all risk is contained under a single owner. In the future, the design will segregate liquidity per subnet into their dedicated actors/contracts.

### Topdown Checkpointing Issues

#### Reliability Issues

- Entirely reliant on a single RPC endpoint to query and observe the parent chain. Blind trust on a single RPC node for parent network events/states; possibility of attacks. RPCs can malfunction and serve missing or incorrect data. (For the Filecoin L1: Lotus event indices are optional, and Lotus does not reason about data availability. So it happily returns empty events for an unindexed epoch, instead of an error, misleading the caller to think there were truly no events. Ideally would use trustless light clients, but the Filecoin L1 doesn’t support them yet.)
 
#### Availability and Catch-up Challenges

- Dependent on RPC data availability, rendering catch-ups problematic. If the subnet has fallen behind its parent and needs to perform a long-range catch-up, queries may exceed the lookback window of the RPC endpoint (16-24h for popular hosted Filecoin L1 RPC endpoints). Furthermore, queries may slow down as we query historical portions of the parent chain, the deeper the queries go.
- Imported parent data is transient and does not survive restarts. We engage in unnecessary work upon restarts to reload data that IPC had already acquired (and was immutable, to begin with). In case of long-range catch-ups, the relevant chain history may be unfetchable as a result of it.

#### Condition and Liveness Problems

- Race conditions during start, potentially deriving in network-wide liveness problems. If Fendermint stops or crashes unexpectedly while processing proposed topdown finality, and we are forced to restart, we will need to fill up our cache before we can validate proposals. A problematic edge case occurs if we had accepted a proposal containing parent finality or prevoted on it just before crashing, as we will enter a crash loop during WAL replay because we will now reject a previously accepted proposal, and this is perceived by CometBFT as a fatal consensus violation from the app. To break this cycle, we will need to clear the local CometBFT state and possibly the WAL.

#### Consensus and Side Effect Inconsistencies

- Topdown voting covers observed height and block hash, but not imported side effects. Due to the Dependency Issues, various nodes may see different side effects (especially if using different RPCs), causing rejections at consensus time, resulting in chain liveness problems, e.g. A/B split brain scenario leading to cohort A rejecting cohort B’s proposals, and vice versa.

#### Tolerance Issues

- Lack of fault tolerance preventing self-healing and automatic failure recovery. Topdown components do not backoff nor reset their state machine after repeated failures, causing us to enter infinite loops when things get stuck in a bad state, and requiring manual intervention to recover. To remedy this, we could gracefully degrade and back off, skip proposing parent finality, and optionally reset some state (circuit breaker pattern), if we repeatedly fail to commit top-down finality during a particular height.

#### Brittle Vote Broadcast and Tallying Mechanisms

- Nodes may restart anytime, wiping their state (Condition and Liveness Problems) and appearing to others as if they’re going back when they gossip, which is not tolerated by peers. Similarly, nodes only gossip new parent views but never re-broadcast older pending ones, preventing restarted nodes from learning about intermediate states they could make partial progress towards.

#### Casuality Awareness

- No notion of a causality. Even though voting for A in a chain A’’←A’←A implicitly votes for A’’, topdown finality does not leverage this fact at all.

#### Query Patterns

- Inefficient parent RPC query patterns, querying individual blocks and events instead of performing range queries when catching up.
- ProcessProposal is non-deterministic, as validators with different views of parent finality (or data availability issues) will reject benevolently.

### Validators 

- Known bug for mapping subnet genesis validators to the initial Fendermint power table. There is an [issue](https://github.com/consensus-shipyard/ipc/pull/1166) describing the cause and and a hot fix.
- Bottom-up checkpoints are injected in the child ledger during block proposal (no gas required), and are subsequently signed by a quorum via transactions in the child ledger (requires gas). This means that validators need to monitor their account balance to be able to expend gas for this system operation. While this is not a bug, it is an inconvenience. We are fixing this by making gas charges conditional (exempted in valid system  as well.

### Subnet Interactions

- Individual subnet stalls cause progress stalls for all child subnets, which opens DoS vectors at the weakest parent.
- A subnet stall causes funds locked on the parent and/or root chains to become inaccessible, but can be recovered through a contract upgrade.
- When a network does not run a relayer and is unable to confirm its changes to the parent, the parent cannot confirm the validator set changes from the subnet, effectively rendering them ineffective.
- A subnet in the parent needs to be activated either by setting federated power or assigning minimum collateral; otherwise, it cannot be used yet. This is not a bug, but it can be confusing.

### Message Relays

- Setting a non-standard IPC address in a cross-message might lead to a panic during message parsing in Fendermint.
- Known open issues regarding transaction acceptance, gossip, and inclusion itself, even in the presence of invalid gas parameters. [Patch is in progress](https://github.com/consensus-shipyard/ipc/pull/1239), but is blocked on some deeper refactors needed.

# Overview

Recall leverages the IPC (InterPlanetary Consensus) framework to create an L2 on Filecoin. This is facilitated by two main components: (1) a set of smart contracts deployed to Filecoin, and (2) a FVM (Filecoin Virtual Machine) based blockchain built using CometBFT. These type of L2s are referred to as ‘subnets’, and will eventually allow us to orchestrate multiple blockchains hierarchically and in parallel, but for our mainnet v1 we will only operate one such subnet. This audit will be primarily focused on the IPC framework implemented in Rust. A general description of the IPC framework can be [found here](https://docs.ipc.space/). 

## Links

- **Previous audits:** [Light Audit](https://github.com/recallnet/ipc/blob/e35cb08f31d12da586afa64abd9838692c6d5034/audits/ip-audit-prep.md)
- **Documentation:** https://flat-agustinia-3f3.notion.site/Audit-documentation-Code4rena-17ddfc9427de80a7849fe2144b56dffb?pvs=4
- **Website:** https://recall.network/
- **X/Twitter:** https://x.com/recallnet

---

# Scope

### Files in scope

| File | Logic Contracts | Interfaces | nSLOC | Purpose | Libraries Used |
| ---- | --------------- | ---------- | ----- | ------- | -------------- | 
| ./contracts/binding/build.rs   | N/A  | ****       | 132   |         | N/A|
| ./contracts/binding/src/convert.rs   | N/A   | ****       | 107   |         | N/A|
| ./contracts/binding/src/lib.rs   | N/A  | ****       | 62   |         | N/A|
| ./contracts/contracts/GatewayDiamond.sol   | 1   | ****       | 85   |         | N/A|
| ./contracts/contracts/OwnershipFacet.sol   | 1   | ****       | 10   |         | N/A|
| ./contracts/contracts/SubnetActorDiamond.sol   | 1   | ****       | 124   |         | "@openzeppelin/contracts/token/ERC20/IERC20.sol"|
| ./contracts/contracts/SubnetRegistryDiamond.sol   | 1   | ****       | 122   |         | N/A|
| ./contracts/contracts/constants/Constants.sol   | 1   | ****       | 6   |         | N/A|
| ./contracts/contracts/diamond/DiamondCutFacet.sol   | 1   | ****       | 9   |         | N/A|
| ./contracts/contracts/diamond/DiamondLoupeFacet.sol   | 1   | ****       | 101   |         | N/A|
| ./contracts/contracts/enums/ConsensusType.sol   | 1   | ****       | 4   |         | N/A|
| ./contracts/contracts/enums/IPCMsgType.sol   | 1   | ****       | 5   |         | N/A|
| ./contracts/contracts/errors/IPCErrors.sol   | 1   | ****       | 96   |         | N/A|
| ./contracts/contracts/gateway/GatewayGetterFacet.sol   | 1   | ****       | 165   |         | "@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/gateway/GatewayManagerFacet.sol   | 1   | ****       | 139   |         | "fevmate/contracts/utils/FilAddress.sol","@openzeppelin/contracts/token/ERC20/IERC20.sol","@openzeppelin/contracts/utils/Address.sol","@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/gateway/GatewayMessengerFacet.sol   | 1   | ****       | 57   |         | "fevmate/contracts/utils/FilAddress.sol","@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/gateway/router/CheckpointingFacet.sol   | 1   | ****       | 102   |         | "@openzeppelin/contracts/utils/Address.sol"|
| ./contracts/contracts/gateway/router/TopDownFinalityFacet.sol   | 1   | ****       | 51   |         | "fevmate/contracts/utils/FilAddress.sol"|
| ./contracts/contracts/gateway/router/XnetMessagingFacet.sol   | 1   | ****       | 22   |         | "fevmate/contracts/utils/FilAddress.sol"|
| ./contracts/contracts/interfaces/IDiamond.sol   | 1   | ****       | 14   |         | N/A|
| ./contracts/contracts/interfaces/IDiamondCut.sol   | 1   | ****       | 5   |         | N/A|
| ./contracts/contracts/interfaces/IDiamondLoupe.sol   | 1   | ****       | 11   |         | N/A|
| ./contracts/contracts/interfaces/IERC165.sol   | 1   | ****       | 4   |         | N/A|
| ./contracts/contracts/interfaces/IGateway.sol   | 1   | ****       | 26   |         | N/A|
| ./contracts/contracts/interfaces/ISubnetActor.sol   | 1   | ****       | 5   |         | N/A|
| ./contracts/contracts/interfaces/IValidatorGater.sol   | 1   | ****       | 5   |         | N/A|
| ./contracts/contracts/interfaces/IValidatorRewarder.sol   | 1   | ****       | 10   |         | N/A|
| ./contracts/contracts/lib/AccountHelper.sol   | 1   | ****       | 7   |         | "fevmate/contracts/utils/FilAddress.sol"|
| ./contracts/contracts/lib/AssetHelper.sol   | 1   | ****       | 155   |         | "@openzeppelin/contracts/token/ERC20/IERC20.sol","@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol"|
| ./contracts/contracts/lib/CrossMsgHelper.sol   | 1   | ****       | 183   |         | "fevmate/contracts/utils/FilAddress.sol","@openzeppelin/contracts/utils/Address.sol"|
| ./contracts/contracts/lib/FvmAddressHelper.sol   | 1   | ****       | 45   |         | N/A           |
| ./contracts/contracts/lib/LibActivity.sol   | 1   | ****       | 118   |         | "@openzeppelin/contracts/utils/structs/EnumerableSet.sol","@openzeppelin/contracts/utils/cryptography/MerkleProof.sol"|
| ./contracts/contracts/lib/LibDiamond.sol   | 1   | ****       | 198   |         | N/A           |
| ./contracts/contracts/lib/LibGateway.sol   | 1   | ****       | 460   |         | "fevmate/contracts/utils/FilAddress.sol","@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/lib/LibGatewayActorStorage.sol   | 1   | ****       | 58   |         | "fevmate/contracts/utils/FilAddress.sol","@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/lib/LibMultisignatureChecker.sol   | 1   | ****       | 50   |         | "@openzeppelin/contracts/utils/cryptography/ECDSA.sol"|
| ./contracts/contracts/lib/LibPausable.sol   | 1   | ****       | 52   |         | N/A           |
| ./contracts/contracts/lib/LibQuorum.sol   | 1   | ****       | 136   |         | "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol","@openzeppelin/contracts/utils/cryptography/ECDSA.sol","@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/lib/LibReentrancyGuard.sol   | 1   | ****       | 24   |         | N/A           |
| ./contracts/contracts/lib/LibStaking.sol   | 1   | ****       | 498   |         | "@openzeppelin/contracts/utils/Address.sol"|
| ./contracts/contracts/lib/LibStakingChangeLog.sol   | 1   | ****       | 88   |         | N/A           |
| ./contracts/contracts/lib/LibSubnetActor.sol   | 1   | ****       | 143   |         | "@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/lib/LibSubnetActorStorage.sol   | 1   | ****       | 63   |         | "@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/lib/LibSubnetRegistryStorage.sol   | 1   | ****       | 26   |         | N/A           |
| ./contracts/contracts/lib/SubnetIDHelper.sol   | 1   | ****       | 126   |         | "@openzeppelin/contracts/utils/Strings.sol"|
| ./contracts/contracts/lib/priority/LibMaxPQ.sol   | 1   | ****       | 119   |         | N/A           |
| ./contracts/contracts/lib/priority/LibMinPQ.sol   | 1   | ****       | 119   |         | N/A           |
| ./contracts/contracts/lib/priority/LibPQ.sol   | 1   | ****       | 59   |         | N/A           |
| ./contracts/contracts/structs/Activity.sol   | 1   | ****       | 32   |         | N/A           |
| ./contracts/contracts/structs/CrossNet.sol   | 1   | ****       | 55   |         | "@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/structs/FvmAddress.sol   | 1   | ****       | 10   |         | N/A           |
| ./contracts/contracts/structs/Quorum.sol   | 1   | ****       | 21   |         | "@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/structs/Subnet.sol   | 1   | ****       | 97   |         | N/A           |
| ./contracts/contracts/subnet/SubnetActorActivityFacet.sol   | 1   | ****       | 44   |         | N/A           |
| ./contracts/contracts/subnet/SubnetActorCheckpointingFacet.sol   | 1   | ****       | 74   |         | "@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/subnet/SubnetActorGetterFacet.sol   | 1   | ****       | 146   |         | "@openzeppelin/contracts/utils/Address.sol","@openzeppelin/contracts/utils/structs/EnumerableSet.sol"|
| ./contracts/contracts/subnet/SubnetActorManagerFacet.sol   | 1   | ****       | 200   |         | "@openzeppelin/contracts/utils/structs/EnumerableSet.sol","@openzeppelin/contracts/utils/Address.sol"|
| ./contracts/contracts/subnet/SubnetActorPauseFacet.sol   | 1   | ****       | 16   |         | N/A           |
| ./contracts/contracts/subnet/SubnetActorRewardFacet.sol   | 1   | ****       | 18   |         | N/A           |
| ./contracts/contracts/subnetregistry/RegisterSubnetFacet.sol   | 1   | ****       | 77   |         | N/A           |
| ./contracts/contracts/subnetregistry/SubnetGetterFacet.sol   | 1   | ****       | 83   |         | N/A           |
| ./contracts/sdk/IpcContract.sol   | 1   | ****       | 79   |         | "@openzeppelin/contracts/utils/ReentrancyGuard.sol","@openzeppelin/contracts/access/Ownable.sol"|
| ./contracts/sdk/IpcContractUpgradeable.sol   | 1   | ****       | 85   |         | "@openzeppelin/contracts-upgradeable/utils/ReentrancyGuardUpgradeable.sol","@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol","@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol"|
| ./contracts/sdk/interfaces/IIpcHandler.sol   | 1   | ****       | 8   |         | N/A           |
| ./fendermint/abci/examples/kvstore.rs   | N/A  | ****       | 160   |         | N/A           |
| ./fendermint/abci/src/application.rs   | N/A  | ****       | 137   |         | N/A           |
| ./fendermint/abci/src/lib.rs   | N/A  | ****       | 3   |         | N/A           |
| ./fendermint/abci/src/util.rs   | N/A  | ****       | 13   |         | N/A           |
| ./fendermint/actors/activity-tracker/src/lib.rs   | N/A  | ****       | 82   |         | N/A           |
| ./fendermint/actors/activity-tracker/src/state.rs   | N/A  | ****       | 43   |         | N/A           |
| ./fendermint/actors/activity-tracker/src/types.rs   | N/A  | ****       | 21   |         | N/A           |
| ./fendermint/actors/api/src/gas_market.rs   | N/A  | ****       | 28   |         | N/A           |
| ./fendermint/actors/api/src/lib.rs   | N/A  | ****       | 1   |         | N/A           |
| ./fendermint/actors/build.rs   | N/A  | ****       | 111   |         | N/A           |
| ./fendermint/actors/chainmetadata/src/actor.rs   | N/A  | ****       | 75   |         | N/A           |
| ./fendermint/actors/chainmetadata/src/lib.rs   | N/A  | ****       | 4   |         | N/A           |
| ./fendermint/actors/chainmetadata/src/shared.rs   | N/A  | ****       | 75   |         | N/A           |
| ./fendermint/actors/eam/src/lib.rs   | N/A  | ****       | 416   |         | N/A           |
| ./fendermint/actors/eam/src/state.rs   | N/A  | ****       | 72   |         | N/A           |
| ./fendermint/actors/gas_market/eip1559/src/lib.rs   | N/A  | ****       | 274   |         | N/A           |
| ./fendermint/actors/src/lib.rs   | N/A  | ****       | 2   |         | N/A           |
| ./fendermint/actors/src/manifest.rs   | N/A  | ****       | 45   |         | N/A           |
| ./fendermint/app/options/examples/network.rs   | N/A  | ****       | 11   |         | N/A           |
| ./fendermint/app/options/src/config.rs   | N/A  | ****       | 3   |         | N/A           |
| ./fendermint/app/options/src/debug.rs   | N/A  | ****       | 40   |         | N/A           |
| ./fendermint/app/options/src/eth.rs   | N/A  | ****       | 28   |         | N/A           |
| ./fendermint/app/options/src/genesis.rs   | N/A  | ****       | 158   |         | N/A           |
| ./fendermint/app/options/src/key.rs   | N/A  | ****       | 69   |         | N/A           |
| ./fendermint/app/options/src/lib.rs   | N/A  | ****       | 165   |         | N/A           |
| ./fendermint/app/options/src/materializer.rs   | N/A  | ****       | 51   |         | N/A           |
| ./fendermint/app/options/src/parse.rs   | N/A  | ****       | 88   |         | N/A           |
| ./fendermint/app/options/src/rpc.rs   | N/A  | ****       | 129   |         | N/A           |
| ./fendermint/app/options/src/run.rs   | N/A  | ****       | 3   |         | N/A           |
| ./fendermint/app/settings/src/eth.rs   | N/A  | ****       | 41   |         | N/A           |
| ./fendermint/app/settings/src/fvm.rs   | N/A  | ****       | 15   |         | N/A           |
| /fendermint/app/settings/src/lib.rs   | N/A  | ****       | 401   |         | N/A           |
| ./fendermint/app/settings/src/resolver.rs   | N/A  | ****       | 58   |         | N/A           |
| ./fendermint/app/settings/src/testing.rs   | N/A  | ****       | 5   |         | N/A           |
| ./fendermint/app/settings/src/utils.rs   | N/A  | ****       | 147   |         | N/A           |
| ./fendermint/app/src/app.rs   | N/A  | ****       | 843   |         | N/A           |
| ./fendermint/app/src/cmd/config.rs   | N/A  | ****       | 11   |         | N/A           |
| ./fendermint/app/src/cmd/debug.rs   | N/A  | ****       | 56   |         | N/A           |
| ./fendermint/app/src/cmd/eth.rs   | N/A  | ****       | 59   |         | N/A           |
| ./fendermint/app/src/cmd/genesis.rs   | N/A  | ****       | 332   |         | N/A           |
| ./fendermint/app/src/cmd/key.rs   | N/A  | ****       | 166   |         | N/A           |
| ./fendermint/app/src/cmd/materializer.rs   | N/A  | ****       | 96   |         | N/A           |
| ./fendermint/app/src/cmd/mod.rs   | N/A  | ****       | 92   |         | N/A           |
| ./fendermint/app/src/cmd/rpc.rs   | N/A  | ****       | 331   |         | N/A           |
| ./fendermint/app/src/cmd/run.rs   | N/A  | ****       | 479   |         | N/A           |
| ./fendermint/app/src/ipc.rs   | N/A  | ****       | 74   |         | N/A           |
| ./fendermint/app/src/lib.rs   | N/A  | ****       | 18   |         | N/A           |
| ./fendermint/app/src/main.rs   | N/A  | ****       | 64   |         | N/A           |
| ./fendermint/app/src/metrics/mod.rs   | N/A  | ****       | 2   |         | N/A           |
| ./fendermint/app/src/metrics/prometheus.rs   | N/A  | ****       | 15   |         | N/A           |
| ./fendermint/app/src/observe.rs   | N/A  | ****       | 164   |         | N/A           |
| ./fendermint/app/src/store.rs   | N/A  | ****       | 87   |         | N/A           |
| ./fendermint/app/src/tmconv.rs   | N/A  | ****       | 353   |         | N/A           |
| ./fendermint/app/src/validators.rs   | N/A  | ****       | 38   |         | N/A           |
| ./fendermint/crypto/src/lib.rs   | N/A  | ****       | 72   |         | N/A           |
| ./fendermint/eth/api/examples/common/mod.rs   | N/A  | ****       | 120   |         | N/A           |
| /fendermint/eth/api/examples/ethers.rs   | N/A  | ****       | 525   |         | N/A           |
| ./fendermint/eth/api/examples/greeter.rs   | N/A  | ****       | 126   |         | N/A           |
| ./fendermint/eth/api/examples/query_blockhash.rs   | N/A  | ****       | 112   |         | N/A           |
| ./fendermint/eth/api/src/apis/eth.rs   | N/A  | ****       | 1038   |         | N/A           |
| ./fendermint/eth/api/src/apis/mod.rs   | N/A  | ****       | 105   |         | N/A           |
| ./fendermint/eth/api/src/apis/net.rs   | N/A  | ****       | 37   |         | N/A           |
| ./fendermint/eth/api/src/apis/web3.rs   | N/A  | ****       | 31   |         | N/A           |
| ./fendermint/eth/api/src/cache.rs   | N/A  | ****       | 224   |         | N/A           |
| ./fendermint/eth/api/src/client.rs   | N/A  | ****       | 155   |         | N/A           |
| ./fendermint/eth/api/src/conv/from_eth.rs   | N/A  | ****       | 109   |         | N/A           |
| ./fendermint/eth/api/src/conv/from_fvm.rs   | N/A  | ****       | 1   |         | N/A           |
| ./fendermint/eth/api/src/conv/from_tm.rs   | N/A  | ****       | 470   |         | N/A           |
| ./fendermint/eth/api/src/conv/mod.rs   | N/A  | ****       | 3   |         | N/A           |
| ./fendermint/eth/api/src/error.rs   | N/A  | ****       | 169   |         | N/A           |
| ./fendermint/eth/api/src/filters.rs   | N/A  | ****       | 607   |         | N/A           |
| ./fendermint/eth/api/src/gas/mod.rs   | N/A  | ****       | 69   |         | N/A           |
| ./fendermint/eth/api/src/gas/output.rs   | N/A  | ****       | 130   |         | N/A           |
| ./fendermint/eth/api/src/handlers/http.rs   | N/A  | ****       | 81   |         | N/A           |
| ./fendermint/eth/api/src/handlers/mod.rs   | N/A  | ****       | 2   |         | N/A           |
| ./fendermint/eth/api/src/handlers/ws.rs   | N/A  | ****       | 186   |         | N/A           |
| ./fendermint/eth/api/src/lib.rs   | N/A  | ****       | 92   |         | N/A           |
| ./fendermint/eth/api/src/mpool.rs   | N/A  | ****       | 160   |         | N/A           |
| ./fendermint/eth/api/src/state.rs   | N/A  | ****       | 512   |         | N/A           |
| ./fendermint/eth/hardhat/src/lib.rs   | N/A  | ****       | 287   |         | N/A           |
| ./fendermint/rocksdb/src/blockstore.rs   | N/A  | ****       | 70   |         | N/A           |
| ./fendermint/rocksdb/src/kvstore.rs   | N/A  | ****       | 318   |         | N/A           |
| ./fendermint/rocksdb/src/lib.rs   | N/A  | ****       | 7   |         | N/A           |
| ./fendermint/rocksdb/src/namespaces.rs   | N/A  | ****       | 20   |         | N/A           |
| ./fendermint/rocksdb/src/rocks/config.rs   | N/A  | ****       | 189   |         | N/A           |
| ./fendermint/rocksdb/src/rocks/error.rs   | N/A  | ****       | 8   |         | N/A           |
| ./fendermint/rocksdb/src/rocks/mod.rs   | N/A  | ****       | 113 |         | N/A           |
| ./fendermint/rpc/examples/simplecoin.rs   | N/A  | ****       | 231   |         | N/A           |
| ./fendermint/rpc/examples/transfer.rs   | N/A  | ****       | 90   |         | N/A           |
| ./fendermint/rpc/src/client.rs   | N/A  | ****       | 304   |         | N/A           |
| ./fendermint/rpc/src/lib.rs   | N/A  | ****       | 18   |         | N/A           |
| ./fendermint/rpc/src/message.rs   | N/A  | ****       | 207   |         | N/A           |
| ./fendermint/rpc/src/query.rs   | N/A  | ****       | 168   |         | N/A           |
| ./fendermint/rpc/src/response.rs   | N/A  | ****       | 38   |         | N/A           |
| ./fendermint/rpc/src/tx.rs   | N/A  | ****       | 154   |         | N/A           |
| ./fendermint/storage/src/im.rs   | N/A  | ****       | 248   |         | N/A           |
| ./fendermint/storage/src/lib.rs   | N/A  | ****       | 122   |         | N/A           |
| ./fendermint/storage/src/testing.rs   | N/A  | ****       | 371   |         | N/A           |
| ./fendermint/tracing/src/lib.rs   | N/A  | ****       | 33   |         | N/A           |
| ./fendermint/vm/actor_interface/src/account.rs   | N/A  | ****       | 7   |         | N/A           |
| ./fendermint/vm/actor_interface/src/activity.rs   | N/A  | ****       | 1   |         | N/A           |
| ./fendermint/vm/actor_interface/src/burntfunds.rs   | N/A  | ****       | 1   |         | N/A           |
| ./fendermint/vm/actor_interface/src/chainmetadata.rs   | N/A  | ****       | 1   |         | N/A           |
| ./fendermint/vm/actor_interface/src/cron.rs   | N/A  | ****       | 19   |         | N/A           |
| ./fendermint/vm/actor_interface/src/diamond.rs   | N/A  | ****       | 15   |         | N/A           |
| ./fendermint/vm/actor_interface/src/eam.rs   | N/A  | ****       | 123   |         | N/A           |
| ./fendermint/vm/actor_interface/src/ethaccount.rs   | N/A  | ****       | 1   |         | N/A           |
| ./fendermint/vm/actor_interface/src/evm.rs   | N/A  | ****       | 86   |         | N/A           |
| ./fendermint/vm/actor_interface/src/gas.rs    | N/A  | ****       | 1   |         | N/A           |
| ./fendermint/vm/actor_interface/src/gas_market.rs   | N/A  | ****       | 1   |         | N/A           |
| ./fendermint/vm/actor_interface/src/init.rs   | N/A  | ****       | 89   |         | N/A           |
| ./fendermint/vm/actor_interface/src/ipc.rs   | N/A  | ****       | 491   |         | N/A           |
| ./fendermint/vm/actor_interface/src/lib.rs   | N/A  | ****       | 38   |         | N/A           |
| ./fendermint/vm/actor_interface/src/multisig.rs   | N/A  | ****       | 42   |         | N/A           |
| ./fendermint/vm/actor_interface/src/placeholder.rs   | N/A  | ****       | 1   |         | N/A           |
| ./fendermint/vm/actor_interface/src/reward.rs   | N/A  | ****       | 1   |         | N/A           |
| ./fendermint/vm/actor_interface/src/system.rs   | N/A  | ****       | 16   |         | N/A           |
| ./fendermint/vm/core/src/chainid.rs   | N/A  | ****       | 135   |         | N/A           |
| ./fendermint/vm/core/src/lib.rs   | N/A  | ****       | 3   |         | N/A           |
| ./fendermint/vm/core/src/timestamp.rs   | N/A  | ****       | 15   |         | N/A           |
| ./fendermint/vm/encoding/src/lib.rs   | N/A  | ****       | 135   |         | N/A           |
| ./fendermint/vm/event/src/lib.rs   | N/A  | ****       | 19   |         | N/A           |
| ./fendermint/vm/genesis/src/arb.rs   | N/A  | ****       | 19   |         | N/A           |
| ./fendermint/vm/genesis/src/lib.rs   | N/A  | ****       | 245   |         | N/A           |
| ./fendermint/vm/genesis/tests/golden.rs   | N/A  | ****       | 12   |         | N/A           |
| ./fendermint/vm/interpreter/src/arb.rs   | N/A  | ****       | 22   |         | N/A           |
| ./fendermint/vm/interpreter/src/bytes.rs   | N/A  | ****       | 215   |         | N/A           |
| ./fendermint/vm/interpreter/src/chain.rs   | N/A  | ****       | 445   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/activity/actor.rs   | N/A  | ****       | 54   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/activity/mod.rs   | N/A  | ****       | 136   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/broadcast.rs   | N/A  | ****       | 171   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/bundle.rs   | N/A  | ****       | 40   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/check.rs   | N/A  | ****       | 131   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/checkpoint.rs   | N/A  | ****       | 428   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/exec.rs   | N/A  | ****       | 214   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/externs.rs   | N/A  | ****       | 107   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/gas.rs   | N/A  | ****       | 137   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/mod.rs   | N/A  | ****       | 104   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/observe.rs   | N/A  | ****       | 164   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/query.rs   | N/A  | ****       | 234   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/state/check.rs   | N/A  | ****       | 50   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/state/exec.rs   | N/A  | ****       | 297   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/state/fevm.rs   | N/A  | ****       | 261   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/state/genesis.rs   | N/A  | ****       | 458   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/state/ipc.rs   | N/A  | ****       | 283   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/state/mod.rs   | N/A  | ****       | 15   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/state/priority.rs   | N/A  | ****       | 62   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/state/query.rs   | N/A  | ****       | 202   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/state/snapshot.rs   | N/A  | ****       | 326   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/store/memory.rs   | N/A  | ****       | 31   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/store/mod.rs   | N/A  | ****       | 25   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/topdown.rs   | N/A  | ****       | 49   |         | N/A           |
| ./fendermint/vm/interpreter/src/fvm/upgrades.rs   | N/A  | ****       | 134   |         | N/A           |
| ./fendermint/vm/interpreter/src/genesis.rs   | N/A  | ****       | 634   |         | N/A           |
| ./fendermint/vm/interpreter/src/lib.rs   | N/A  | ****       | 58   |         | N/A           |
| ./fendermint/vm/interpreter/src/selector.rs   | N/A  | ****       | 32   |         | N/A           |
| ./fendermint/vm/interpreter/src/signed.rs   | N/A  | ****       | 186   |         | N/A           |
| ./fendermint/vm/interpreter/tests/golden.rs   | N/A  | ****       | 12   |         | N/A           |
| ./fendermint/vm/message/src/chain.rs   | N/A  | ****       | 32   |         | N/A           |
| ./fendermint/vm/message/src/conv/from_eth.rs   | N/A  | ****       | 138   |         | N/A           |
| ./fendermint/vm/message/src/conv/from_fvm.rs   | N/A  | ****       | 214   |         | N/A           |
| ./fendermint/vm/message/src/conv/mod.rs   | N/A  | ****       | 66   |         | N/A           |
| ./fendermint/vm/message/src/ipc.rs   | N/A  | ****       | 136   |         | N/A           |
| ./fendermint/vm/message/src/lib.rs   | N/A  | ****       | 14   |         | N/A           |
| ./fendermint/vm/message/src/query.rs   | N/A  | ****       | 105   |         | N/A           |
| ./fendermint/vm/message/src/signed.rs   | N/A  | ****       | 389   |         | N/A           |
| ./fendermint/vm/message/tests/golden.rs   | N/A  | ****       | 72   |         | N/A           |
| ./fendermint/vm/resolver/src/ipld.rs   | N/A  | ****       | 103   |         | N/A           |
| ./fendermint/vm/resolver/src/lib.rs   | N/A  | ****       | 2   |         | N/A           |
| ./fendermint/vm/resolver/src/pool.rs   | N/A  | ****       | 211   |         | N/A           |
| ./fendermint/vm/snapshot/src/car/chunker.rs   | N/A  | ****       | 179   |         | N/A           |
| ./fendermint/vm/snapshot/src/car/mod.rs   | N/A  | ****       | 87   |         | N/A           |
| ./fendermint/vm/snapshot/src/car/streamer.rs   | N/A  | ****       | 114   |         | N/A           |
| ./fendermint/vm/snapshot/src/client.rs   | N/A  | ****       | 140   |         | N/A           |
| ./fendermint/vm/snapshot/src/error.rs   | N/A  | ****       | 14   |         | N/A           |
| ./fendermint/vm/snapshot/src/lib.rs   | N/A  | ****       | 14   |         | N/A           |
| ./fendermint/vm/snapshot/src/manager.rs   | N/A  | ****       | 357   |         | N/A           |
| ./fendermint/vm/snapshot/src/manifest.rs   | N/A  | ****       | 168   |         | N/A           |
| ./fendermint/vm/snapshot/src/state.rs   | N/A  | ****       | 119   |         | N/A           |
| ./fendermint/vm/snapshot/tests/golden.rs   | N/A  | ****       | 12   |         | N/A           |
| ./fendermint/vm/topdown/src/cache.rs   | N/A  | ****       | 224   |         | N/A           |
| ./fendermint/vm/topdown/src/convert.rs   | N/A  | ****       | 34   |         | N/A           |
| ./fendermint/vm/topdown/src/error.rs   | N/A  | ****       | 13   |         | N/A           |
| ./fendermint/vm/topdown/src/finality/fetch.rs   | N/A  | ****       | 372   |         | N/A           |
| ./fendermint/vm/topdown/src/finality/mod.rs   | N/A  | ****       | 140   |         | N/A           |
| ./fendermint/vm/topdown/src/finality/null.rs   | N/A  | ****       | 464   |         | N/A           |
| ./fendermint/vm/topdown/src/lib.rs   | N/A  | ****       | 146   |         | N/A           |
| ./fendermint/vm/topdown/src/observe.rs   | N/A  | ****       | 203   |         | N/A           |
| ./fendermint/vm/topdown/src/proxy.rs   | N/A  | ****       | 168   |         | N/A           |
| ./fendermint/vm/topdown/src/sync/mod.rs   | N/A  | ****       | 157   |         | N/A           |
| ./fendermint/vm/topdown/src/sync/syncer.rs   | N/A  | ****       | 499   |         | N/A           |
| ./fendermint/vm/topdown/src/sync/tendermint.rs   | N/A  | ****       | 36   |         | N/A           |
| ./fendermint/vm/topdown/src/toggle.rs   | N/A  | ****       | 107   |         | N/A           |
| ./fendermint/vm/topdown/src/voting.rs   | N/A  | ****       | 322   |         | N/A           |
| ./fendermint/vm/topdown/tests/smt_voting.rs   | N/A  | ****       | 371   |         | N/A           |
| ./ipc/api/src/address.rs   | N/A  | ****       | 131   |         | N/A           |
| ./ipc/api/src/checkpoint.rs   | N/A  | ****       | 147   |         | N/A           |
| ./ipc/api/src/cross.rs   | N/A  | ****       | 181   |         | N/A           |
| ./ipc/api/src/error.rs   | N/A  | ****       | 24   |         | N/A           |
| ./ipc/api/src/evm.rs   | N/A  | ****       | 337   |         | N/A           |
| ./ipc/api/src/gateway.rs   | N/A  | ****       | 17   |         | N/A           |
| ./ipc/api/src/lib.rs   | N/A  | ****       | 129   |         | N/A           |
| ./ipc/api/src/merkle.rs   | N/A  | ****       | 29   |         | N/A           |
| ./ipc/api/src/runtime.rs   | N/A  | ****       | 22   |         | N/A           |
| ./ipc/api/src/staking.rs   | N/A  | ****       | 85   |         | N/A           |
| ./ipc/api/src/subnet.rs   | N/A  | ****       | 74   |         | N/A           |
| ./ipc/api/src/subnet_id.rs   | N/A  | ****       | 313   |         | N/A           |
| ./ipc/api/src/validator.rs   | N/A  | ****       | 46   |         | N/A           |
| ./ipc/cli/src/commands/checkpoint/bottomup_bundles.rs   | N/A  | ****       | 36   |         | N/A           |
| ./ipc/cli/src/commands/checkpoint/bottomup_height.rs   | N/A  | ****       | 29   |         | N/A           |
| ./ipc/cli/src/commands/checkpoint/list_validator_changes.rs   | N/A  | ****       | 33   |         | N/A           |
| ./ipc/cli/src/commands/checkpoint/mod.rs   | N/A  | ****       | 50   |         | N/A           |
| ./ipc/cli/src/commands/checkpoint/quorum_reached.rs   | N/A  | ****       | 35   |         | N/A           |
| ./ipc/cli/src/commands/checkpoint/relayer.rs   | N/A  | ****       | 106   |         | N/A           |
| ./ipc/cli/src/commands/config/init.rs   | N/A  | ****       | 30   |         | N/A           |
| ./ipc/cli/src/commands/config/mod.rs   | N/A  | ****       | 23   |         | N/A           |
| ./ipc/cli/src/commands/crossmsg/fund.rs   | N/A  | ****       | 150   |         | N/A           |
| ./ipc/cli/src/commands/crossmsg/mod.rs   | N/A  | ****       | 49   |         | N/A           |
| ./ipc/cli/src/commands/crossmsg/propagate.rs   | N/A  | ****       | 24   |         | N/A           |
| ./ipc/cli/src/commands/crossmsg/release.rs   | N/A  | ****       | 92   |         | N/A           |
| ./ipc/cli/src/commands/crossmsg/topdown_cross.rs   | N/A  | ****       | 65   |         | N/A           |
| ./ipc/cli/src/commands/daemon.rs   | N/A  | ****       | 52   |         | N/A           |
| ./ipc/cli/src/commands/mod.rs   | N/A  | ****       | 128   |         | N/A           |
| ./ipc/cli/src/commands/subnet/bootstrap.rs   | N/A  | ****       | 59   |         | N/A           |
| ./ipc/cli/src/commands/subnet/create.rs   | N/A  | ****       | 158   |         | N/A           |
| ./ipc/cli/src/commands/subnet/genesis_epoch.rs   | N/A  | ****       | 25   |         | N/A           |
| ./ipc/cli/src/commands/subnet/join.rs   | N/A  | ****       | 115   |         | N/A           |
| ./ipc/cli/src/commands/subnet/kill.rs   | N/A  | ****       | 28   |         | N/A           |
| ./ipc/cli/src/commands/subnet/leave.rs   | N/A  | ****       | 59   |         | N/A           |
| ./ipc/cli/src/commands/subnet/list_subnets.rs   | N/A  | ****       | 39   |         | N/A           |
| ./ipc/cli/src/commands/subnet/list_validators.rs   | N/A  | ****       | 30   |         | N/A           |
| ./ipc/cli/src/commands/subnet/mod.rs   | N/A  | ****       | 91   |         | N/A           |
| ./ipc/cli/src/commands/subnet/rpc.rs   | N/A  | ****       | 51   |         | N/A           |
| ./ipc/cli/src/commands/subnet/send_value.rs   | N/A  | ****       | 42   |         | N/A           |
| ./ipc/cli/src/commands/subnet/set_federated_power.rs   | N/A  | ****       | 56   |         | N/A           |
| ./ipc/cli/src/commands/subnet/show_gateway_contract_commit_sha.rs   | N/A  | ****       | 33   |         | N/A           |
| ./ipc/cli/src/commands/subnet/validator.rs   | N/A  | ****       | 35   |         | N/A           |
| ./ipc/cli/src/commands/util/eth.rs   | N/A  | ****       | 24   |         | N/A           |
| ./ipc/cli/src/commands/util/f4.rs   | N/A  | ****       | 23   |         | N/A           |
| ./ipc/cli/src/commands/util/mod.rs   | N/A  | ****       | 26   |         | N/A           |
| ./ipc/cli/src/commands/validator/batch_claim.rs   | N/A  | ****       | 44   |         | N/A           |
| ./ipc/cli/src/commands/validator/list.rs   | N/A  | ****       | 40   |         | N/A           |
| ./ipc/cli/src/commands/validator/mod.rs   | N/A  | ****       | 26   |         | N/A           |
| ./ipc/cli/src/commands/wallet/balances.rs   | N/A  | ****       | 98   |         | N/A           |
| ./ipc/cli/src/commands/wallet/default.rs   | N/A  | ****       | 68   |         | N/A           |
| ./ipc/cli/src/commands/wallet/export.rs   | N/A  | ****       | 149   |         | N/A           |
| ./ipc/cli/src/commands/wallet/import.rs   | N/A  | ****       | 66   |         | N/A           |
| ./ipc/cli/src/commands/wallet/list.rs   | N/A  | ****       | 55   |         | N/A           |
| ./ipc/cli/src/commands/wallet/mod.rs   | N/A  | ****       | 52   |         | N/A           |
| ./ipc/cli/src/commands/wallet/new.rs   | N/A  | ****       | 43   |         | N/A           |
| ./ipc/cli/src/commands/wallet/remove.rs   | N/A  | ****       | 37   |         | N/A           |
| ./ipc/cli/src/lib.rs   | N/A  | ****       | 53   |         | N/A           |
| ./ipc/cli/src/main.rs   | N/A  | ****       | 13   |         | N/A           |
| ./ipc/observability/src/config.rs   | N/A  | ****       | 58   |         | N/A           |
| ./ipc/observability/src/lib.rs   | N/A  | ****       | 48   |         | N/A           |
| ./ipc/observability/src/macros.rs   | N/A  | ****       | 51   |         | N/A           |
| ./ipc/observability/src/observe.rs   | N/A  | ****       | 23   |         | N/A           |
| ./ipc/observability/src/serde.rs   | N/A  | ****       | 8   |         | N/A           |
| ./ipc/observability/src/traces.rs   | N/A  | ****       | 114   |         | N/A           |
| ./ipc/observability/src/tracing_layers.rs   | N/A  | ****       | 85   |         | N/A           |
| ./ipc/provider/src/checkpoint.rs   | N/A  | ****       | 212   |         | N/A           |
| ./ipc/provider/src/config/deserialize.rs   | N/A  | ****       | 179   |         | N/A           |
| ./ipc/provider/src/config/mod.rs   | N/A  | ****       | 84   |         | N/A           |
| ./ipc/provider/src/config/serialize.rs   | N/A  | ****       | 117   |         | N/A           |
| ./ipc/provider/src/config/subnet.rs   | N/A  | ****       | 78   |         | N/A           |
| ./ipc/provider/src/config/tests.rs   | N/A  | ****       | 52   |         | N/A           |
| ./ipc/provider/src/jsonrpc/mod.rs   | N/A  | ****       | 159   |         | N/A           |
| ./ipc/provider/src/jsonrpc/tests.rs   | N/A  | ****       | 45   |         | N/A           |
| ./ipc/provider/src/lib.rs   | N/A  | ****       | 733   |         | N/A           |
| ./ipc/provider/src/lotus/client.rs   | N/A  | ****       | 410   |         | N/A           |
| ./ipc/provider/src/lotus/message/chain.rs   | N/A  | ****       | 44   |         | N/A           |
| ./ipc/provider/src/lotus/message/deserialize.rs   | N/A  | ****       | 157   |         | N/A           |
| ./ipc/provider/src/lotus/message/ipc.rs   | N/A  | ****       | 109   |         | N/A           |
| ./ipc/provider/src/lotus/message/mod.rs   | N/A  | ****       | 57   |         | N/A           |
| ./ipc/provider/src/lotus/message/mpool.rs   | N/A  | ****       | 98   |         | N/A           |
| ./ipc/provider/src/lotus/message/serialize.rs   | N/A  | ****       | 15   |         | N/A           |
| ./ipc/provider/src/lotus/message/state.rs   | N/A  | ****       | 59   |         | N/A           |
| ./ipc/provider/src/lotus/message/tests.rs   | N/A  | ****       | 114   |         | N/A           |
| ./ipc/provider/src/lotus/message/wallet.rs   | N/A  | ****       | 45   |         | N/A           |
| ./ipc/provider/src/lotus/mod.rs   | N/A  | ****       | 48   |         | N/A           |
| ./ipc/provider/src/manager/evm/gas_estimator_middleware.rs   | N/A  | ****       | 126   |         | N/A           |
| ./ipc/provider/src/manager/evm/manager.rs   | N/A  | ****       | 1421   |         | N/A           |
| ./ipc/provider/src/manager/evm/mod.rs   | N/A  | ****       | 27   |         | N/A           |
| ./ipc/provider/src/manager/mod.rs   | N/A  | ****       | 8   |         | N/A           |
| ./ipc/provider/src/manager/subnet.rs   | N/A  | ****       | 177   |         | N/A           |
| ./ipc/provider/src/observe.rs   | N/A  | ****       | 38   |         | N/A           |
| ./ipc/types/src/actor_error.rs   | N/A  | ****       | 282   |         | N/A           |
| ./ipc/types/src/amt.rs   | N/A  | ****       | 51   |         | N/A           |
| ./ipc/types/src/ethaddr.rs   | N/A  | ****       | 153   |         | N/A           |
| ./ipc/types/src/hamt.rs   | N/A  | ****       | 56   |         | N/A           |
| ./ipc/types/src/lib.rs   | N/A  | ****       | 246   |         | N/A           |
| ./ipc/types/src/link.rs   | N/A  | ****       | 62   |         | N/A           |
| ./ipc/types/src/taddress.rs   | N/A  | ****       | 118   |         | N/A           |
| ./ipc/types/src/uints.rs   | N/A  | ****       | 268   |         | N/A           |
| ./ipc/wallet/src/evm/memory.rs   | N/A  | ****       | 53   |         | N/A           |
| ./ipc/wallet/src/evm/mod.rs   | N/A  | ****       | 105   |         | N/A           |
| ./ipc/wallet/src/evm/persistent.rs   | N/A  | ****       | 227   |         | N/A           |
| ./ipc/wallet/src/fvm/errors.rs   | N/A  | ****       | 24   |         | N/A           |
| ./ipc/wallet/src/fvm/keystore.rs   | N/A  | ****       | 503   |         | N/A           |
| ./ipc/wallet/src/fvm/mod.rs   | N/A  | ****       | 11   |         | N/A           |
| ./ipc/wallet/src/fvm/serialization.rs   | N/A  | ****       | 131   |         | N/A           |
| ./ipc/wallet/src/fvm/utils.rs   | N/A  | ****       | 13   |         | N/A           |
| ./ipc/wallet/src/fvm/wallet.rs   | N/A  | ****       | 329   |         | N/A           |
| ./ipc/wallet/src/fvm/wallet_helpers.rs   | N/A  | ****       | 329   |         | N/A           |
| ./ipc/wallet/src/lib.rs   | N/A  | ****       | 28   |         | N/A           |
| ./ipld/resolver/src/arb.rs   | N/A  | ****       | 21   |         | N/A           |
| ./ipld/resolver/src/behaviour/content.rs   | N/A  | ****       | 254   |         | N/A           |
| ./ipld/resolver/src/behaviour/discovery.rs   | N/A  | ****       | 314   |         | N/A           |
| ./ipld/resolver/src/behaviour/membership.rs   | N/A  | ****       | 488   |         | N/A           |
| ./ipld/resolver/src/behaviour/mod.rs   | N/A  | ****       | 87   |         | N/A           |
| ./ipld/resolver/src/client.rs   | N/A  | ****       | 73   |         | N/A           |
| ./ipld/resolver/src/hash.rs   | N/A  | ****       | 21   |         | N/A           |
| ./ipld/resolver/src/lib.rs   | N/A  | ****       | 20   |         | N/A           |
| ./ipld/resolver/src/limiter.rs   | N/A  | ****       | 60   |         | N/A           |
| ./ipld/resolver/src/missing_blocks.rs   | N/A  | ****       | 22   |         | N/A           |
| ./ipld/resolver/src/observe.rs   | N/A  | ****       | 401   |         | N/A           |
| ./ipld/resolver/src/provider_cache.rs   | N/A  | ****       | 279   |         | N/A           |
| ./ipld/resolver/src/provider_record.rs   | N/A  | ****       | 81   |         | N/A           |
| ./ipld/resolver/src/service.rs   | N/A  | ****       | 430   |         | N/A           |
| ./ipld/resolver/src/signed_record.rs   | N/A  | ****       | 83   |         | N/A           |
| ./ipld/resolver/src/timestamp.rs   | N/A  | ****       | 38   |         | N/A           |
| ./ipld/resolver/src/vote_record.rs   | N/A  | ****       | 120   |         | N/A           |
| ./ipld/resolver/tests/smoke.rs   | N/A  | ****       | 279   |         | N/A           |
| ./ipld/resolver/tests/store/mod.rs   | N/A  | ****       | 41   |         | N/A           |
|Total| 61| | 47,988 | | | |

### Files out of scope

Any file that is not explicitly included in the aforementioned list is to be considered out-of-scope. 

## Scoping Q &amp; A

### General questions

| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| ERC20 used by the protocol              |       None             |
| Test coverage (Solidity)                           | 83.76%                          |
| ERC721 used  by the protocol            |            None              |
| ERC777 used by the protocol             |           None                |
| ERC1155 used by the protocol            |              None            |
| Chains the protocol will be deployed on | Filecoin, IPC  |

### ERC20 token behaviors in scope

| Question                                                                                                                                                   | Answer |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| [Missing return values](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#missing-return-values)                                                      | No   |
| [Fee on transfer](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#fee-on-transfer)                                                                  |  No |
| [Balance changes outside of transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#balance-modifications-outside-of-transfers-rebasingairdrops) |  No  |
| [Upgradeability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#upgradable-tokens)                                                                 | No   |
| [Flash minting](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#flash-mintable-tokens)                                                              | No   |
| [Pausability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#pausable-tokens)                                                                      |  No  |
| [Approval race protections](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#approval-race-protections)                                              |  No  |
| [Revert on approval to zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-approval-to-zero-address)                            |  No  |
| [Revert on zero value approvals](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-approvals)                                    |  No  |
| [Revert on zero value transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                    |  No  |
| [Revert on transfer to the zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-transfer-to-the-zero-address)                    |  No  |
| [Revert on large approvals and/or transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-large-approvals--transfers)                  |  No  |
| [Doesn't revert on failure](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure)                                                   |  No  |
| [Multiple token addresses](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                          |  No  |
| [Low decimals ( < 6)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#low-decimals)                                                                 |  No  |
| [High decimals ( > 18)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#high-decimals)                                                              |  No  |
| [Blocklists](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#tokens-with-blocklists)                                                                |  No  |

### External integrations (e.g., Uniswap) behavior in scope:


| Question                                                  | Answer |
| --------------------------------------------------------- | ------ |
| Enabling/disabling fees (e.g. Blur disables/enables fees) | No   |
| Pausability (e.g. Uniswap pool gets paused)               |  No   |
| Upgradeability (e.g. Uniswap gets upgraded)               |   No  |


### EIP compliance checklist

Although some contracts do inherit and implement certain EIP functionality, none of the contracts in scope are expected to be strictly compliant with any EIP standard.

# Additional context

## Main Invariants

### Funds

- The total supply of a subnet must match the total funds locked for that specific subnet in the gateway
- Funds bridged via the gateway for a particular address to a subnet should result in the same amount being credited for that address within the subnet 

### Consensus

- The current membership in the child subnet gateway must be the same to the validators in  CometBFT, i.e. public key and power (power scale applied).
- The active validators in the parent subnet must have the same public key, power (with power scale applied) to the validators in the child subnet after bottom up checkpoint is submitted to the parent subnet.

### Topdown Finality

- The topdown finality block hash in the child subnet must be equal to the parent’s block hash for the same height.
- The nonces in cross network messages to be applied in topdown finality must be sequential and no gaps.
- The configuration number for validator change request must be sequential and no gaps.
- The child subnet’s `appliedTopDownNonce` must be equal the cross network message’s `localNonce`.
- The topdown finality’s block height must be larger than that of the previous committed topdown finality’s block height.
- The cross network messages and validator changes to be committed from the parent finality must be the same compared to the parent subnet.

### Bottom-Up Checkpoint

- The block height in the checkpoint submitted must be more than the last submitted bottom up checkpoint height.
- The block height in the checkpoint submitted must be less than the last submitted bottom up checkpoint height + bottom up checkpoint period.
- The `configurationNumber` in the bottom up checkpoint to be submitted must be within the `startConfigurationNumber`(inclusive) and `nextConfigurationNumber` (exclusive).
- The parent subnet’s `appliedBottomUpNonce` must be equal the cross network message’s `localNonce`.
- The cross network messages to be committed from the bottom up checkpoint must be the same compared to the child subnet.

### Cross-Chain Messages

- Cross xnet messages can only by send from contract to contract that implements specific interface

## Attack ideas (where to focus for bugs)

**Repo: ipc/contracts**

This repo contains the general IPC related solidity code. These contracts are provided by the IPC framework

**GatewayDiamond.sol**

Implementation of the IPC GatewayActor within the Diamond pattern.

**SubnetActorDiamond.sol**

Reference implementation of an IPC SubnetActor within the Diamond pattern.

**SubnetRegistry.sol**

Registry contract for seamlessly deploying subnet actors.

This is the entrypoint to cross network messaging: https://github.com/consensus-shipyard/ipc/blob/main/contracts/contracts/gateway/GatewayMessengerFacet.sol. The documentation can be found: https://github.com/consensus-shipyard/ipc/blob/main/docs/fendermint/general_cross_messages.md.

**IPC blockchain**

A general description of the IPC framework can be [found here](https://docs.ipc.space/). This section will outline the most important components, what they do, and their priority in the audit.

**Fendermint**

Fendermint is an implementation of IPC as a CometBFT ABCI++ application written in Rust. It leverages the FVM as it’s virtual machine, but has custom logic for verifying transactions. The main differentiator of IPC compared to other blockchain frameworks is the concept of *hierarchical consensus*, which enables an hierarchy of blockchains to be spun up and down dynamically. However, when Recall launches there will only be one level, e.g. *subnet*.

**Key components**

**`ipc/fendermint/vm/topdown`**

Part of the hierarchical network topology, e.g. we attach to Filecoin L1 through the gateway contracts. Please find the high level overview here: https://github.com/consensus-shipyard/ipc/blob/main/specs/topdown.md.

For bottom up checkpointing, please find the documentation here: https://github.com/consensus-shipyard/ipc/blob/main/specs/bottom-up-interaction.md.

**`ipc/fendermint/vm/interpreter`**

How checkpoints are managed and how to make sense of IPC messages

**`ipc/fendermint/actors`**

Contains FVM actors that ship by default in all new deployments of an IPC subnet

**IPC - `ipc/ipc`**

This is the main CLI used to interact with an IPC subnet, providing basic wallet functionality as well as interaction with the parent network. It also includes the provider, which manages the sender's wallet and is responsible for calling contracts or submitting transactions on the parent or subnet.

## All trusted roles in the protocol

| Role                                | Description                       |
| --------------------------------------- | ---------------------------- |
| Gateway Owner                          | Able to upgrade contracts without on-chain governance |
| Subnet Actor | Able to handle and disperse fund withdrawals without validation |
| Subnet Validator | Fully trusted without collusion check or validator penalty |
| Validators | Reduce the risk of trusting a single RPC node for parent chain data by confirming observations |

## Describe any novel or unique curve logic or mathematical models implemented in the contracts:

N/A

## Running tests

Build dependencies are identical to the [Consensus Shipyard IPC](https://github.com/consensus-shipyard/ipc/tree/main?tab=readme-ov-file#prerequisites) implementation.

**Building**

```bash
# make sure that rust has the wasm32 target
rustup target add wasm32-unknown-unknown

# add your user to the docker group
sudo usermod -aG docker $USER && newgrp docker

# build
make

# building will generate the following binaries
./target/release/ipc-cli --version
./target/release/fendermint --version
```

To run tests

```bash
make test
```

To run code coverage (Solidity)

```bash
cd contracts/
make coverage
```

### Coverage Table (Solidity)

| File                                                | % Lines            | % Statements       | % Branches       | % Funcs          |
|-----------------------------------------------------|--------------------|--------------------|------------------|------------------|
| contracts/GatewayDiamond.sol                        | 37.21% (16/43)     | 36.36% (16/44)     | 33.33% (1/3)     | 50.00% (2/4)     |
| contracts/OwnershipFacet.sol                        | 100.00% (4/4)      | 100.00% (2/2)      | 100.00% (0/0)    | 100.00% (2/2)    |
| contracts/SubnetActorDiamond.sol                    | 32.79% (20/61)     | 28.57% (18/63)     | 10.00% (1/10)    | 80.00% (4/5)     |
| contracts/SubnetRegistryDiamond.sol                 | 92.31% (60/65)     | 93.75% (60/64)     | 72.73% (8/11)    | 75.00% (3/4)     |
| contracts/diamond/DiamondCutFacet.sol               | 100.00% (3/3)      | 100.00% (2/2)      | 100.00% (0/0)    | 100.00% (1/1)    |
| contracts/diamond/DiamondLoupeFacet.sol             | 77.61% (52/67)     | 77.92% (60/77)     | 80.00% (4/5)     | 60.00% (3/5)     |
| contracts/gateway/GatewayGetterFacet.sol            | 87.78% (79/90)     | 86.30% (63/73)     | 50.00% (1/2)     | 91.18% (31/34)   |
| contracts/gateway/GatewayManagerFacet.sol           | 96.00% (72/75)     | 96.34% (79/82)     | 81.25% (13/16)   | 100.00% (7/7)    |
| contracts/gateway/GatewayMessengerFacet.sol         | 88.89% (16/18)     | 89.47% (17/19)     | 50.00% (2/4)     | 100.00% (2/2)    |
| contracts/gateway/router/CheckpointingFacet.sol     | 95.12% (39/41)     | 95.12% (39/41)     | 71.43% (5/7)     | 100.00% (5/5)    |
| contracts/gateway/router/TopDownFinalityFacet.sol   | 91.67% (22/24)     | 95.83% (23/24)     | 100.00% (1/1)    | 75.00% (3/4)     |
| contracts/gateway/router/XnetMessagingFacet.sol     | 100.00% (3/3)      | 100.00% (2/2)      | 100.00% (0/0)    | 100.00% (1/1)    |
| contracts/lib/AccountHelper.sol                     | 100.00% (2/2)      | 100.00% (2/2)      | 100.00% (0/0)    | 100.00% (1/1)    |
| contracts/lib/AssetHelper.sol                       | 78.82% (67/85)     | 81.01% (64/79)     | 66.67% (22/33)   | 82.35% (14/17)   |
| contracts/lib/CrossMsgHelper.sol                    | 95.00% (57/60)     | 96.61% (57/59)     | 80.00% (8/10)    | 92.31% (12/13)   |
| contracts/lib/FvmAddressHelper.sol                  | 52.17% (12/23)     | 44.00% (11/25)     | 0.00% (0/4)      | 60.00% (3/5)     |
| contracts/lib/LibActivity.sol                       | 67.31% (35/52)     | 64.71% (44/68)     | 66.67% (4/6)     | 66.67% (4/6)     |
| contracts/lib/LibDiamond.sol                        | 79.44% (85/107)    | 77.88% (81/104)    | 28.00% (7/25)    | 100.00% (12/12)  |
| contracts/lib/LibGateway.sol                        | 89.64% (251/280)   | 89.31% (284/318)   | 73.91% (34/46)   | 93.75% (30/32)   |
| contracts/lib/LibGatewayActorStorage.sol            | 75.00% (6/8)       | 60.00% (3/5)       | 100.00% (1/1)    | 100.00% (3/3)    |
| contracts/lib/LibMultisignatureChecker.sol          | 100.00% (19/19)    | 100.00% (22/22)    | 100.00% (5/5)    | 100.00% (1/1)    |
| contracts/lib/LibPausable.sol                       | 92.59% (25/27)     | 95.45% (21/22)     | 100.00% (2/2)    | 87.50% (7/8)     |
| contracts/lib/LibQuorum.sol                         | 89.06% (57/64)     | 89.86% (62/69)     | 69.23% (9/13)    | 100.00% (6/6)    |
| contracts/lib/LibReentrancyGuard.sol                | 100.00% (9/9)      | 88.89% (8/9)       | 0.00% (0/1)      | 100.00% (2/2)    |
| contracts/lib/LibStaking.sol                        | 90.62% (290/320)   | 90.72% (313/345)   | 78.00% (39/50)   | 91.84% (45/49)   |
| contracts/lib/LibStakingChangeLog.sol               | 100.00% (23/23)    | 100.00% (23/23)    | 100.00% (0/0)    | 100.00% (7/7)    |
| contracts/lib/LibSubnetActor.sol                    | 97.59% (81/83)     | 97.87% (92/94)     | 90.91% (10/11)   | 100.00% (10/10)  |
| contracts/lib/LibSubnetActorStorage.sol             | 30.77% (4/13)      | 14.29% (1/7)       | 0.00% (0/2)      | 60.00% (3/5)     |
| contracts/lib/SubnetIDHelper.sol                    | 98.68% (75/76)     | 98.89% (89/90)     | 88.89% (8/9)     | 100.00% (11/11)  |
| contracts/lib/priority/LibMaxPQ.sol                 | 97.40% (75/77)     | 98.73% (78/79)     | 100.00% (6/6)    | 92.31% (12/13)   |
| contracts/lib/priority/LibMinPQ.sol                 | 100.00% (77/77)    | 100.00% (79/79)    | 100.00% (6/6)    | 100.00% (13/13)  |
| contracts/lib/priority/LibPQ.sol                    | 71.88% (23/32)     | 70.37% (19/27)     | 0.00% (0/2)      | 66.67% (6/9)     |
| contracts/subnet/SubnetActorActivityFacet.sol       | 91.67% (11/12)     | 90.00% (9/10)      | 33.33% (1/3)     | 100.00% (3/3)    |
| contracts/subnet/SubnetActorCheckpointingFacet.sol  | 96.97% (32/33)     | 97.22% (35/36)     | 83.33% (5/6)     | 100.00% (3/3)    |
| contracts/subnet/SubnetActorGetterFacet.sol         | 85.06% (74/87)     | 83.33% (60/72)     | 100.00% (1/1)    | 82.35% (28/34)   |
| contracts/subnet/SubnetActorManagerFacet.sol        | 94.07% (111/118)   | 93.52% (101/108)   | 85.71% (30/35)   | 100.00% (10/10)  |
| contracts/subnet/SubnetActorPauseFacet.sol          | 100.00% (8/8)      | 100.00% (6/6)      | 100.00% (0/0)    | 100.00% (3/3)    |
| contracts/subnet/SubnetActorRewardFacet.sol         | 100.00% (4/4)      | 100.00% (4/4)      | 100.00% (1/1)    | 100.00% (1/1)    |
| contracts/subnetregistry/RegisterSubnetFacet.sol    | 100.00% (23/23)    | 100.00% (22/22)    | 100.00% (2/2)    | 100.00% (2/2)    |
| contracts/subnetregistry/SubnetGetterFacet.sol      | 59.18% (29/49)     | 61.76% (21/34)     | 28.57% (2/7)     | 53.33% (8/15)    |
| Total                                               | 83.76% (2754/3288) | 85.67% (2887/3370) | 68.60% (319/465) | 79.09% (454/574) |

## Miscellaneous
Employees of Recall and employees' family members are ineligible to participate in this audit.

Code4rena's rules cannot be overridden by the contents of this README. In case of doubt, please check with C4 staff.

