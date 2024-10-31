<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [SuperchainERC20 Starter Kit](#superchainerc20-starter-kit)
  - [What is SuperchainERC20?](#what-is-superchainerc20)
    - [`ICrosschainERC20`](#icrosschainerc20)
  - [Getting Started](#getting-started)
    - [1. Install prerequisites: `foundry`](#1-install-prerequisites-foundry)
    - [2. Clone the repository:](#2-clone-the-repository)
    - [3. Navigate to the project directory:](#3-navigate-to-the-project-directory)
    - [4. Install project dependencies using pnpm:](#4-install-project-dependencies-using-pnpm)
    - [5. Install smart contracts dependencies:](#5-install-smart-contracts-dependencies)
    - [6. Start the development environment:](#6-start-the-development-environment)
  - [Deploying SuperchainERC20s](#deploying-superchainerc20s)
    - [Configuring RPC urls](#configuring-rpc-urls)
    - [Deployment config](#deployment-config)
      - [`[deploy-config]`](#deploy-config)
      - [`[token]`](#token)
      - [`[single_chain_deploy_config]`](#single_chain_deploy_config)
      - [`[multi_chain_deploy_config]`](#multi_chain_deploy_config)
    - [Deploying to multiple chains](#deploying-to-multiple-chains)
    - [Deploying to single chain](#deploying-to-single-chain)
    - [Best practices for deploying SuperchainERC20](#best-practices-for-deploying-superchainerc20)
      - [Use Create3 to deploy](#use-create3-to-deploy)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# SuperchainERC20 Starter Kit

## What is SuperchainERC20?

The full spec for the SuperchainERC20 standard can be found at: https://specs.optimism.io/interop/token-bridging.html#superchainerc20-standard. 

The `SuperchainERC20` standard is a cross-chain-compatible ERC20 token design that ensures token fungibility across chains within the Superchain ecosystem. Built with interoperability in mind, `SuperchainERC20` tokens can be fungible across the Superchain using the `SuperchainERC20Bridge`.

**Notice**: ERC20s that do not use `SuperchainERC20Bridge` can still be fungible across the Superchain with interop message passing using a custom bridge and implementing `ICrosschainERC20`.

### `ICrosschainERC20`

To achieve cross-chain functionality, the `SuperchainERC20` standard incorporates the `ICrosschainERC20` interface, defining essential functions and events:

- **`crosschainMint`**: Mints tokens on the destination chain as part of a cross-chain transfer.
- **`crosschainBurn`**: Burns tokens on the source chain to facilitate the transfer.
- **Events (`CrosschainMint` and `CrosschainBurn`)**: Emit when tokens are minted or burned, enabling transparent tracking of cross-chain transactions.

## Getting Started

### 1. Install prerequisites: `foundry`

`supersim` requires `anvil` to be installed.

Follow [this guide](https://book.getfoundry.sh/getting-started/installation) to install Foundry.

### 2. Clone the repository:

```sh
git clone git@github.com:ethereum-optimism/superchainerc20-starter.git
```

### 3. Navigate to the project directory:

```sh
cd superchainerc20-starter
```

### 4. Install project dependencies using pnpm:

```sh
pnpm i
```

### 5. Install smart contracts dependencies:

```sh
pnpm install:contracts
```

### 6. Start the development environment:

This command will:

- Start the `supersim` local development environment
- Deploy the smart contracts to the test networks
- Launch the example frontend application

```sh
pnpm dev
```

## Deploying SuperchainERC20s

### Configuring RPC urls

This repository includes a script to automatically fetch the public RPC URLs for each chain listed in the [Superchain Registry](https://github.com/ethereum-optimism/superchain-registry/blob/main/chainList.json) and add them to the `[rpc_endpoints]` configuration section of `foundry.toml`.

The script ensures that only new RPC URLs are appended, preserving any URLs already present in `foundry.toml`. To execute this script, run:
```sh
pnpm contracts:update:rpcs
```

### Deployment config

The deployment configuration for token deployments is managed through the `deploy-config.toml` file. The options available in this file allow you to customize both single and multi-chain deployments. Below is a detailed breakdown of each configuration section:

#### `[deploy-config]`

This section defines parameters for deploying token contracts across both single and multi-chain environments.

- `salt`: A unique identifier used for deploying token contracts via [`Create3`](https://github.com/pcaversaccio/createx/blob/8c91357af5eb3454eb84103863d8a49a15613883/src/CreateX.sol#L630). This value ensures that contract deployments are deterministic.
    - example: `salt = "ethers phoenix"`

#### `[token]`

Deployment configuration for the token that will be deployed.

- `owner_address`: the address designated as the owner of the token.
    - The `L2NativeSuperchainERC20.sol` contract included in this repo extends the [`Ownable`](https://github.com/Vectorized/solady/blob/c3b2ffb4a3334ea519555c5ea11fb0e666f8c2bc/src/auth/Ownable.sol) contract
    - example: `owner_address = "0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266"`
- `name`: the token's name.
    - example: `name = "TestSuperchainERC20"`
- `symbol`: the token's symbol.
    - example: `symbol = "TSU"`
- `decimals`: the number of decimal places the token supports.
    - example: `decimals = 18`

#### `[single_chain_deploy_config]`

This section contains configuration settings specific to single chain deployments via the `SingleChainSuperchainERC20Deployment.s.sol` script.

- `chain`: specifies the chain where the token will be deployed. This value must correspond to a chain in the `[rpc_endpoints]` section of `foundry.toml`.
  - example: `chain = "op/mainnet"`

#### `[multi_chain_deploy_config]`

This section contains configuration settings specific to multi-chain deployments via the `MultiChainSuperchainERC20Deployment.s.sol` script.

- `chains`: Lists the chains where the token will be deployed. Each chain must correspond to an entry in the `[rpc_endpoints]` section of `foundry.toml`.
  - example: `chains = ["op_chain_a","op_chain_b"]`

### Deploying to multiple chains

Before proceeding with this section, ensure that your `deploy-config.toml` file is fully configured (see the [Deployment config](#deployment-config) section for more details on setup). Additionally, confirm that the `[rpc_endpoints]` section in `foundry.toml` is properly set up by following the instructions in [Configuring RPC urls](#configuring-rpc-urls).

Multi-chain deployments are executed through the `MultiChainSuperchainERC20Deployment.s.sol` script. This script deploys tokens across each specified chain in the deployment configuration using [`Create3`](https://github.com/pcaversaccio/createx/blob/8c91357af5eb3454eb84103863d8a49a15613883/src/CreateX.sol#L630), ensuring deterministic contract addresses for each deployment. The script targets the `L2NativeSuperchainERC20.sol` contract by default. If you need to modify the token being deployed, either update this file directly or point the script to a custom token contract of your choice.

To execute a multi-chain deployment run:

```sh
pnpm contracts:deploy:multichain

```

### Deploying to single chain

Before proceeding with this section, ensure that your `deploy-config.toml` file is fully configured (see the [Deployment config](#deployment-config) section for more details on setup). Additionally, confirm that the `[rpc_endpoints]` section in `foundry.toml` is properly set up by following the instructions in [Configuring RPC urls](#configuring-rpc-urls).

A single chain deployment is executed through the `SingleChainSuperchainERC20Deployment.s.sol` script. This script deploys a token on the specified chain in the deployment configuration using [`Create3`](https://github.com/pcaversaccio/createx/blob/8c91357af5eb3454eb84103863d8a49a15613883/src/CreateX.sol#L630), ensuring deterministic contract addresses for the deployment. The script targets the `L2NativeSuperchainERC20.sol` contract by default. If you need to modify the token being deployed, either update this file directly or point the script to a custom token contract of your choice.

To execute a single chain deployment run:

```sh
pnpm contracts:deploy:singlechain

```

### Best practices for deploying SuperchainERC20

#### Use Create3 to deploy

[`Create3`](https://github.com/pcaversaccio/createx/blob/8c91357af5eb3454eb84103863d8a49a15613883/src/CreateX.sol#L630) ensures that the address is deterministically deterimined by the deployer address and the provided salt. Unlike `CREATE2` the address is not dependent on the init code of the contract, which makes it easy to ensure that the contract is deployed to the same address across all chains. This is crucial because in order for cross-chain transfers of `SuperchainERC20`s to work, the tokens must be deployed at the same address across all chains.

The `CreateX` library used in this repository further strengthens deployment security with permissioned deploy protection (see [details here](https://github.com/pcaversaccio/createx/blob/058bc3b07e082711457d8ea20d8767a37a5a0021/src/CreateX.sol#L922)), which prevents unauthorized entities from deploying a contract at the same address. Only the original deployer using the same salt can deploy to that address, ensuring that your SuperchainERC20 token's address is uniquely tied to your deployment setup across chains.