# Textile audit details
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

From the sponsors: 

- Gateway manages funds for all registered subnets (cf. “omnibus account”). Funds sit in the subnet actor until the subnet is bootstrapped, at which point the collateral and supply is transferred to the gateway. Gateway will manage the fund and disperse to the subnet upon subnet’s request. This design will cause some security threats as one subnet can potentially impact other subnets. Please check the `Problems` section of the refactoring [proposal](https://github.com/consensus-shipyard/ipc/blob/main/specs/drafts/contract-redesign.md). The mitigation under the current design is for gateways to be single-tenant, i.e. to restrict who can create subnets under it, so all risk is contained under a single owner. In the future, the design will segregate liquidity per subnet into their dedicated actors/contracts.
- Current topdown checkpointing issues (these are addressed in PRs pending merge, but not yet integrated by Recall):
    - Entirely reliant on a single RPC endpoint to query and observe the parent chain. We deposit blind trust on a single RPC node for parent network events/states; possibility of attacks. RPCs can malfunction and serve missing or incorrect data. (For the Filecoin L1: Lotus event indices are optional, and Lotus does not reason about data availability. So it happily returns empty events for an unindexed epoch, instead of an error, misleading the caller to think there were truly no events. Ideally would use trustless light clients, but the Filecoin L1 doesn’t support them yet.)
    1. Dependent on RPC data availability, rendering catch-ups problematic. If the subnet has fallen behind its parent and needs to perform a long-range catch-up, queries may exceed the lookback window of the RPC endpoint (16-24h for popular hosted Filecoin L1 RPC endpoints). Furthermore, queries may slow down as we query historical portions of the parent chain, the deeper the queries go.
    2. Imported parent data is transient and does not survive restarts. We engage in unnecessary work upon restarts to reload data that IPC had already acquired (and was immutable, to begin with). In case of long-range catch-ups, the relevant chain history may be unfetchable as a result of (2).
    3. Race conditions during start, potentially deriving in network-wide liveness problems. If Fendermint stops or crashes unexpectedly while processing proposed topdown finality, and we are forced to restart, we will need to fill up our cache before we can validate proposals. A problematic edge case occurs if we had accepted a proposal containing parent finality or prevoted on it just before crashing, as we will enter a crash loop during WAL replay because we will now reject a previously accepted proposal, and this is perceived by CometBFT as a fatal consensus violation from the app. To break this cycle, we will need to clear the local CometBFT state and possibly the WAL.
    4. Topdown voting covers observed height and block hash, but not imported side effects. Due to problems (1) and (2), various nodes may see different side effects (especially if using different RPCs), causing rejections at consensus time, resulting in chain liveness problems, e.g. A/B split brain scenario leading to cohort A rejecting cohort B’s proposals, and vice versa.
    5. Lack of fault tolerance preventing self-healing and automatic failure recovery. Topdown components do not backoff nor reset their state machine after repeated failures, causing us to enter infinite loops when things get stuck in a bad state, and requiring manual intervention to recover. To remedy this, we could gracefully degrade and back off, skip proposing parent finality, and optionally reset some state (circuit breaker pattern), if we repeatedly fail to commit top-down finality during a particular height.
    6. Brittle vote broadcast and tallying mechanisms. Nodes may restart anytime, wiping their state (see problem 3) and appearing to others as if they’re going back when they gossip, which is not tolerated by peers. Similarly, nodes only gossip new parent views but never re-broadcast older pending ones, preventing restarted nodes from learning about intermediate states they could make partial progress towards.
    7. No notion of a causality. Even though voting for A in a chain A’’←A’←A implicitly votes for A’’, topdown finality does not leverage this fact at all.
    8. Inefficient parent RPC query patterns, querying individual blocks and events instead of performing range queries when catching up.
    9. ProcessProposal is non-deterministic, as validators with different views of parent finality (or data availability issues) will reject benevolently.
- Known bug for mapping subnet genesis validators to the initial Fendermint power table. There is an [issue](https://github.com/consensus-shipyard/ipc/pull/1166) describing the cause and and a hot fix.
- Bottom-up checkpoints are injected in the child ledger during block proposal (no gas required), and are subsequently signed by a quorum via transactions in the child ledger (requires gas). This means that validators need to monitor their account balance to be able to expend gas for this system operation. While this is not a bug, it is an inconvenience. We are fixing this by making gas charges conditional (exempted in valid system  as well.
- Individual subnet stalls cause progress stalls for all child subnets, which opens DoS vectors at the weakest parent.
- A subnet stall causes funds locked on the parent and/or root chains to become inaccessible, but can be recovered through a contract upgrade.
- When a network does not run a relayer and is unable to confirm its changes to the parent, the parent cannot confirm the validator set changes from the subnet, effectively rendering them ineffective.
- A subnet in the parent needs to be activated either by setting federated power or assigning minimum collateral; otherwise, it cannot be used yet. This is not a bug, but it can be confusing.
- Setting a non-standard IPC address in a cross-message might lead to a panic during message parsing in Fendermint.
- Known open issues regarding transaction acceptance, gossip, and inclusion itself, even in the presence of invalid gas parameters. [Patch is in progress](https://github.com/consensus-shipyard/ipc/pull/1239), but is blocked on some deeper refactors needed.

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

# Overview

[ ⭐️ SPONSORS: add info here ]

## Links

- **Previous audits:**  
  - ✅ SCOUTS: If there are multiple report links, please format them in a list.
- **Documentation:** https://flat-agustinia-3f3.notion.site/Audit-documentation-Code4rena-17ddfc9427de80a7849fe2144b56dffb?pvs=4
- **Website:** https://linktr.ee/textileio
- **X/Twitter:** https://x.com/textileio

---

# Scope

[ ✅ SCOUTS: add scoping and technical details here ]

### Files in scope
- ✅ This should be completed using the `metrics.md` file
- ✅ Last row of the table should be Total: SLOC
- ✅ SCOUTS: Have the sponsor review and and confirm in text the details in the section titled "Scoping Q amp; A"

*For sponsors that don't use the scoping tool: list all files in scope in the table below (along with hyperlinks) -- and feel free to add notes to emphasize areas of focus.*

| Contract | SLOC | Purpose | Libraries used |  
| ----------- | ----------- | ----------- | ----------- |
| [contracts/folder/sample.sol](https://github.com/code-423n4/repo-name/blob/contracts/folder/sample.sol) | 123 | This contract does XYZ | [`@openzeppelin/*`](https://openzeppelin.com/contracts/) |

### Files out of scope
✅ SCOUTS: List files/directories out of scope

## Scoping Q &amp; A

### General questions
### Are there any ERC20's in scope?: No

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".




### Are there any ERC777's in scope?: 

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".



### Are there any ERC721's in scope?: No

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".



### Are there any ERC1155's in scope?: No

✅ SCOUTS: If the answer above 👆 is "Yes", please add the tokens below 👇 to the table. Otherwise, update the column with "None".



✅ SCOUTS: Once done populating the table below, please remove all the Q/A data above.

| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| ERC20 used by the protocol              |       🖊️             |
| Test coverage                           | ✅ SCOUTS: Please populate this after running the test coverage command                          |
| ERC721 used  by the protocol            |            🖊️              |
| ERC777 used by the protocol             |           🖊️                |
| ERC1155 used by the protocol            |              🖊️            |
| Chains the protocol will be deployed on | OtherFilecoin  |

### ERC20 token behaviors in scope

| Question                                                                                                                                                   | Answer |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| [Missing return values](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#missing-return-values)                                                      |    |
| [Fee on transfer](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#fee-on-transfer)                                                                  |   |
| [Balance changes outside of transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#balance-modifications-outside-of-transfers-rebasingairdrops) |    |
| [Upgradeability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#upgradable-tokens)                                                                 |    |
| [Flash minting](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#flash-mintable-tokens)                                                              |    |
| [Pausability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#pausable-tokens)                                                                      |    |
| [Approval race protections](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#approval-race-protections)                                              |    |
| [Revert on approval to zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-approval-to-zero-address)                            |    |
| [Revert on zero value approvals](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-approvals)                                    |    |
| [Revert on zero value transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                    |    |
| [Revert on transfer to the zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-transfer-to-the-zero-address)                    |    |
| [Revert on large approvals and/or transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-large-approvals--transfers)                  |    |
| [Doesn't revert on failure](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure)                                                   |    |
| [Multiple token addresses](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                          |    |
| [Low decimals ( < 6)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#low-decimals)                                                                 |    |
| [High decimals ( > 18)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#high-decimals)                                                              |    |
| [Blocklists](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#tokens-with-blocklists)                                                                |    |

### External integrations (e.g., Uniswap) behavior in scope:


| Question                                                  | Answer |
| --------------------------------------------------------- | ------ |
| Enabling/disabling fees (e.g. Blur disables/enables fees) | No   |
| Pausability (e.g. Uniswap pool gets paused)               |  No   |
| Upgradeability (e.g. Uniswap gets upgraded)               |   No  |


### EIP compliance checklist
no

✅ SCOUTS: Please format the response above 👆 using the template below👇

| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| src/Token.sol                           | ERC20, ERC721                |
| src/NFT.sol                             | ERC721                       |


# Additional context

## Main invariants

- The total circulating supply in the child subnet must be equal to the total funds locked in the gateway for that specific subnet.
- The fund bridged to the child subnet for an address must be the same to the native token balance of that address in the child subnet blockchain (assuming no other native token transfers).
- The current membership in the child subnet gateway must be the same to the validators in the cometbft, i.e. public key and power (power scale applied).
- The active validators in the parent subnet must have the same public key, power (with power scale applied) to the validators in the child subnet after bottom up checkpoint is submitted to the parent subnet.
- Topdown finality:
    - The topdown finality block hash in the child subnet must be equal to the parent’s block hash for the same height.
    - The nonces in cross network messages to be applied in topdown finality must be sequential and no gaps.
    - The configuration number for validator change request must be sequential and no gaps.
    - The child subnet’s `appliedTopDownNonce` must be equal the cross network message’s `localNonce`.
    - The topdown finality’s block height must be larger than that of the previous committed topdown finality’s block height.
    - The cross network messages and validator changes to be committed from the parent finality must be the same compared to the parent subnet.
- Bottom up checkpoint:
    - The block height in the checkpoint submitted must be more than the last submitted bottom up checkpoint height.
    - The block height in the checkpoint submitted must be less than the last submitted bottom up checkpoint height + bottom up checkpoint period.
    - The `configurationNumber` in the bottom up checkpoint to be submitted must be within the `startConfigurationNumber`(inclusive) and `nextConfigurationNumber` (exclusive).
    - The parent subnet’s `appliedBottomUpNonce` must be equal the cross network message’s `localNonce`.
    - The cross network messages to be committed from the bottom up checkpoint must be the same compared to the child subnet.
- Cross xnet messages can only by send from contract to contract that implements specific interface

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

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

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## All trusted roles in the protocol

The owner of the gateway can perform contract upgrades without any on-chain governance.

Gateway trusts the subnet actor completely with current design. This means fund withdraw calls are not validated by the gateway and directly dispersed.

Currently, the entire set of subnet validators is fully trusted. There is no collusion check or validator penalty.

For topdown, we entirely reliant on a single RPC endpoint to query and observe the parent chain. We deposit blind trust on a single RPC node for parent network events/states. To reduce the risk, a quorum of validators are required to confirm the observations.

✅ SCOUTS: Please format the response above 👆 using the template below👇

| Role                                | Description                       |
| --------------------------------------- | ---------------------------- |
| Owner                          | Has superpowers                |
| Administrator                             | Can change fees                       |

## Describe any novel or unique curve logic or mathematical models implemented in the contracts:

n/a

✅ SCOUTS: Please format the response above 👆 so its not a wall of text and its readable.

## Running tests

> from github consensus-shipyard/ipc:
> 

**Building**

```
# make sure that rust has the wasm32 target
rustup target add wasm32-unknown-unknown

# add your user to the docker group
sudo usermod -aG docker $USER && newgrp docker

# clone this repo and build
git clone https://github.com/consensus-shipyard/ipc.git
cd ipc
make

# building will generate the following binaries
./target/release/ipc-cli --version
./target/release/fendermint --version
```

**Run tests**

```
make test
```

✅ SCOUTS: Please format the response above 👆 using the template below👇

```bash
git clone https://github.com/code-423n4/2023-08-arbitrum
git submodule update --init --recursive
cd governance
foundryup
make install
make build
make sc-election-test
```
To run code coverage
```bash
make coverage
```
To run gas benchmarks
```bash
make gas
```

✅ SCOUTS: Add a screenshot of your terminal showing the gas report
✅ SCOUTS: Add a screenshot of your terminal showing the test coverage

## Miscellaneous
Employees of Textile and employees' family members are ineligible to participate in this audit.

Code4rena's rules cannot be overridden by the contents of this README. In case of doubt, please check with C4 staff.

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

