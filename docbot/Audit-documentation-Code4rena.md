# System overview

Recall leverages the IPC (InterPlanetary Consensus) framework to create an L2 on Filecoin. This is facilitated by two main components: (1) a set of smart contracts deployed to Filecoin, and (2) a FVM (Filecoin Virtual Machine) based blockchain built using CometBFT. These type of L2s are referred to as ‘subnets’, and will eventually allow us to orchestrate multiple blockchains hierarchically and in parallel, but for our mainnet v1 we will only operate one such subnet. This audit will be primarily focused on the IPC framework implemented in Rust. A general description of the IPC framework can be [found here](https://docs.ipc.space/). 

## Code walkthrough

https://www.loom.com/share/964b9d722ec24ef7902bdc802318287d

## L1 smart contracts

The contracts listed below starts with the Hoku ERC20 token which is used by operators to stake and become validators in the network. 

### Repo: [hokunet/ipc](https://github.com/hokunet/ipc)`/contracts`

This repo contains the general IPC related solidity code. These contracts are provided by the IPC framework

**GatewayDiamond.sol**

Implementation of the IPC GatewayActor within the Diamond pattern.

**SubnetActorDiamond.sol**

Reference implementation of an IPC SubnetActor within the Diamond pattern.

**SubnetRegistry.sol**

Registry contract for seamlessly deploying subnet actors.

## IPC blockchain

A general description of the IPC framework can be [found here](https://docs.ipc.space/). This section will outline the most important components, what they do, and their priority in the audit.

### [Fendermint](https://github.com/hokunet/ipc/tree/develop/fendermint)

Fendermint is an implementation of IPC as a CometBFT ABCI++ application written in Rust. It leverages the FVM as it’s virtual machine, but has custom logic for verifying transactions. The main differentiator of IPC compared to other blockchain frameworks is the concept of *hierarchical consensus*, which enables an hierarchy of blockchains to be spun up and down dynamically. However, when Recall launches there will only be one level, e.g. *subnet*.

### Key components

**`ipc/fendermint/vm/topdown`**

Part of the hierarchical network topology, e.g. we attach to Filecoin L1 through the gateway contracts.

**`ipc/fendermint/vm/interpreter`**

How checkpoints are managed and how to make sense of IPC messages

**`ipc/fendermint/actors`**

Contains FVM actors that ship by default in all new deployments of an IPC subnet

### [IPC](https://github.com/hokunet/ipc/tree/develop/ipc)

This is the main CLI used to interact with an IPC subnet as well as providing some basic wallet functionality.

## Node architecture

Below is an architecture diagram for a Recall node. The parts that are purely IPC are:

- RootEVM
- CometBFT
- Fendermint
- Ethereum RPC

See image linked [here](https://flat-agustinia-3f3.notion.site/Audit-documentation-Code4rena-17ddfc9427de80a7849fe2144b56dffb#17ddfc9427de81348304e62cb815e446).

---

# Design choices

## Why we are using IPC

We often frame blockchain scalability in terms of throughput and latency, but scaling data presents additional challenges beyond just processing transactions. The volume, type, and structure of data can vary drastically. The Interplanetary Consensus Protocol (IPC) helps Hoku address scalability challenges through a hierarchical architecture where subnets operate in parallel, significantly enhancing throughput. These subnets function practically independently, under their own consensus, while borrowing security from their parent subnet. This hierarchical structure also means subnets themselves can optimize for reducing *latency*, while the overall network can scale horizontally to accommodate greater *throughput*.

Most L2 scaling solutions either inherit L1 security without their own consensus algorithms (e.g., rollups) or have their own consensus but lack L1 security (e.g., sidechains). In contrast, the [hierarchical subnet strategy](https://docs.ipc.space/overview/how-ipc-compares) employed by Hoku (in theory) operates as a hybrid of these approaches, allowing subnets to have their own consensus algorithms while also inheriting security from parent subnets (though in practice, we aren’t there yet).

Validators on Recall may choose to join (multiple) subnets, or form a consortium with other validators to launch a new subnet. The root subnet anchors to a L1 blockchain. In our initial implementation, we don’t yet support multiple subnets, though the infrastructure to do so is in place.

---

# Known restrictions/limitations

We are still relatively far from the vision for Hoku articulated in the litepaper linked above. As such, we thought it would be useful to enumerate some of the current known limitations to help guide reviewers and auditors.

### Asymmetric storage commitments

Hoku does not yet support asymmetric storage commitments. This means that all validators store all data, and they are all storing the same amount of data for the same length of time, even though the data is being erasure coded. This limitation affects staking, recoverability, storage efficiency, and validator rewards. While moving towards asymmetric storage commitments is a high priority post-audit, we have deprioritized it because the network acts safely and correctly even without this feature in place. This does affect consensus operations.

### Collateral slashing

Like many early proof of stake networks, Hoku does not yet support a fraud proof system, collateral slashing, or other dis-incentive structures. We operate on a good-faith basis with a set of well-vetted validators, and so have knowingly deprioritized slashing in an effort to move quickly and validate correct operation of the consensus logic happy path. This does affect staking, validator rewards, and overall tokenomics.

### Delegated staking

Hoku is designed to support delegated proof of stake. This is standard practice across many proof of state blockchain systems. However, delegated staking does not make a lot of sense in a network without asymmetric staking commitments and collateral slashing, so we have temporarily deprioritized delegated staking. This does affect staking, validator rewards, and overall tokenomics.

### Token issuance flow

To incentivize ongoing network participation, the Hoku blockchain is designed to leverage a dynamic inflation model that is a function of total committed stake and other network parameters. Inflation is supposed to be based on a [stable yield mechanism](https://forum.threshold.network/t/tip-003-threshold-network-reward-mechanisms-proposal-i-stable-yield-for-non-institutional-staker-welfare/82), and is parameterized to strike a balance between incentivizing validators and preserving network utility over time. However, due to some temporary limitations at the IPC layer, we have moved to a fixed issuance rate. This is reflected in the current rewards flow. This does affect validator rewards, and overall tokenomics. For the time being, the issuance rate is parameterized and configurable via permissioned network updates.

---

# Dependencies

**Filecoin**

We rely on Filecoin as an L1 and consider our network an L2 in the context of Filecoin. We have our ERC20 token deployed there as well as all of the IPC specific Solidity contracts which allow our validators to coordinate.

**InterPlanetary Consensus (IPC)**

IPC is the main blockchain framework we are building on. It is a combination of the FVM and CometBFT and is intended to be used as a L2 framework for Filecoin. However, we are currently maintaining a *fork* of the [upstream repo](https://github.com/consensus-shipyard/ipc). This is because the framework doesn’t currently allow us to implement custom actors and in particular not custom actors that have out of consensus functionality.

---

### Previous audits

The `ipc/contracts` solidity code has gotten a light audit already. Results here: https://github.com/hokunet/ipc/pull/437

# Running a node

See the documentation here: https://github.com/hokunet/ipc/blob/main/scripts/README.md
