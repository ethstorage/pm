# CircleCI Memo

This document provides an overview of the changes made to the configuration and environment to enable the forked `Optimism` monorepo to successfully execute CircleCI tests on the new runner.

## CircleCI Server Environment Setup

### Software Installations

The following tools must be manually installed on the runner machine (`AX101`) for CircleCI workflows to run properly:

- `python3`
- `yq`
- `anvil`
- `semgrep`
- `kurtosis`
- `ripgrep` (`rg`)
- `golangci-lint`

> **Note**: All tools must be installed in `/usr/local/bin`, or their commands will not be found during execution.

If any dependencies are not installed, please follow the corresponding instructions below to install them.

#### Installing Python3

Most Linux OSs have Python pre-installed. Following [this instruction](https://www.geeksforgeeks.org/download-and-install-python-3-latest-version/) otherwise.

#### Installing yq

```bash
wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/local/bin/yq && chmod +x /usr/local/bin/yq
yq -V
```

#### Installing golangci-lint

```bash
curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b /usr/local/bin v1.63.3

```
####  Installing kurtosis

```
echo "deb [trusted=yes] https://apt.fury.io/kurtosis-tech/ /" | sudo tee /etc/apt/sources.list.d/kurtosis.list
sudo apt update
sudo apt install kurtosis-cli

# This command should start the Kurtosis Engine Server. 
kurtosis engine start

# After starting the engine, you can check its status with:
kurtosis engine status
```


### Environment Variables

The following **Environment Variables** must be set in CircleCI under **Project Settings > Environment Variables** to enable tests that depend on forked testnet/mainnet environments (e.g., archive nodes):

- **`SEPOLIA_RPC_URL`**
- **`MAINNET_RPC_URL`**

> Archive nodes with a free BlockPI API key should be sufficient for these RPC URLs.


### Configuration Updates

#### Jobs

The most time-consuming jobs now run on the new runner machine `AX101` (65.108.230.142), including:

- `contracts-bedrock-checks`
- `fuzz-golang`
- `go-tests`

The following jobs **still run on Docker containers**, but may be moved to `AX101` later after further evaluation:

- `check-generated-mocks-op-node`
- `check-generated-mocks-op-service`
- `cannon-build-test-vectors`
- `todo-issues`

#### Repo Name

The repository name was updated from `develop` to `op-es` in the `packages/contracts-bedrock/scripts/checks/check-semver-diff.sh` file.

#### Submodules

For repositories configured as submodules (e.g., `da-server`), code can be automatically synced during workflows due to an additional step is added to update submodules.

