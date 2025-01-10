## Developer Guide to Optimism in EthStorage

This document provides instructions for contributing to EthStorage's Optimism Monorepo. It outlines a practical process for running tests locally to verify code changes and ensure successful CircleCI tests.

It is good practice to conduct static checks and tests locally every time before merging code changes into the default branch. 

## Local Checks and Tests: Run with One Command

This section provides a very convenient way to perform all checks and tests with just one command.

If you are a first-time contributor to EthStorage's Optimism repository or are not yet using `mise` to manage your software dependencies, please complete [the steps to set up your development environment](#environment-setup).

Now, in the root directory of your Optimism repo, execute this command, with the placeholders replaced with your RPC endpoints prepared in [this step](#prepare-rpc-endpoints).

```bash
# Set your own RPC URLs below. 
export SEPOLIA_RPC_URL=<YOUR_SEPOLIA_RPC_URL>
export MAINNET_RPC_URL=<YOUR_MAINNET_RPC_URL>
mise run dt
```

Basically the above command does all the job needed to do the local tests. If you find any problem or need further explore, please refer to [the step-by-step instruction](#local-checks-and-tests-step-by-step-instructions), which have the same effect but more detailed explainations.

## Local Checks and Tests: Step-by-Step Instructions

This section provides a step-by-step guide based on the way how programming languages are organized in the repo: [Solidity](#solidity) and [Go](#go). If you are not the first time to run the tests, a more convinient way to conduct the tests can be found [here](#local-checks-and-tests-run-with-one-command).

### Environment Check
Before going forward, check if `mise` is activated in your current prompt:

```bash
mise doctor | grep 'activated:'
```

In a correct output you should see:

```bash
bashactivated: yes
```

If the answer is `no`, please check [this step](#activate-mise) is properly executed.
Sometimes in the senario of IDE, you may need to manually activate `mise` each time after open the terminal according to your shell type:


```bash
eval "$(mise activate zsh)" 
# or
eval "$(mise activate bash)"
```

### Solidity

Navigate to the `contracts-bedrock` directory:

```bash
cd packages/contracts-bedrock
```

#### Static Checks

Run the following command to clean, build, lint, and check all contracts:

```bash
just pre-pr
```

For detailed explanations and possible fixes for certain errors, refer to [this section](#contract-checks-and-fixes-in-detail).

#### Run Unit Tests

The following command runs contract tests using Forge:

```bash
just test
```

You may need to check the result and make necessary fixes if there is any error, and re-running the tests. 

A successful result may look like this:

```bash
Ran 282 test suites in 396.48s (1890.89s CPU time): 1763 tests passed, 0 failed, 1 skipped (1764 total tests)
```

### Go

Navigate to the root directory of your Optimism Monorepo.

#### Lint

Before proceeding with tests, ensure that your Go code is correctly linted:

```bash
make lint-go
```

If any lint errors are detected, attempt to fix them:

```bash
make lint-go-fix
```
Note that some errors may need manual fixing.

#### Build

Go tests require Go components as well as contracts to be built. Execute:

```bash
# Builds op-node, op-proposer, op-batcher, and contracts-bedrock
make build

# Build other essential components
cd op-program && make op-program-client && cd ..
cd cannon && make elf && cd ..
cd op-e2e && make pre-test && cd ..
```

#### Generate Allocations

Allocations are also required by the following tests. To generate allocations, use:

```bash
make devnet-allocs
```

#### Set Environment Variables

Prepare your environment by setting necessary variables like below. Replace the placeholders with the RPC endpoints prepared in [this step](#prepare-rpc-endpoints).

```bash
export ENABLE_KURTOSIS=true
export OP_E2E_CANNON_ENABLED="false"
export OP_E2E_SKIP_SLOW_TEST=true
export OP_E2E_USE_HTTP=true
export ENABLE_ANVIL=true

# Set your own RPC URLs below. 
export SEPOLIA_RPC_URL=<YOUR_SEPOLIA_RPC_URL>
export MAINNET_RPC_URL=<YOUR_MAINNET_RPC_URL>
```

#### Run Go Tests

Execute the Go tests with:

```bash
gotestsum --no-summary=skipped,output \
   --format=testname \
   --rerun-fails=2
```

After the tests have finished, check the console for any errors or failures. A successful result should look like this:

```bash
DONE 8567 tests, 95 skipped in 189.396s
```

#### Run End-to-End Tests

Run end-to-end tests with:

```bash
cd op-e2e
make test-actions
make test-ws
```
For detailed information on end-to-end testing, refer to [this README](https://github.com/ethstorage/optimism/blob/op-es/op-e2e/README.md).

## Contract Checks and Fixes in Detail

The `just pre-pr` command includes several checks:

- **Gas Snapshot Check (`gas-snapshot-check-no-build`)**: Ensures gas snapshots are up-to-date. Update using:
  ```bash
  just gas-snapshot-no-build
  ```

- **Semgrep Test Validity Check (`semgrep-test-validity-check`)**: Ensures semgrep tests are properly configured.

- **Unused Imports Check (`unused-imports-check-no-build`)**: Flags unused imports in Solidity contracts.

- **Snapshots Check (`snapshots-check-no-build`)**: Ensures all snapshots are current. Regenerate snapshots with:
  ```bash
  just snapshots-no-build
  ```

- **Lint Check (`lint-check`)**: Ensures contracts are properly linted. Auto-fix lint errors with:
  ```bash
  just lint-fix
  ```

- **Semver Diff Check (`semver-diff-check-no-build`)**: Ensures modified contracts have updated semver versions. Fix with:
  ```bash
  just semver-lock
  ```

- **Deploy Config Validation (`validate-deploy-configs`)**: Validates deploy configurations.

- **Spacer Variable Check (`validate-spacers-no-build`)**: Validates spacer variables without requiring a build.

- **Interface Check (`interfaces-check-no-build`)**: Validates interfaces without requiring a build.

- **Forge Test Linting (`lint-forge-tests-check-no-build`)**: Validates Forge test names adhere to correct formats.

## Environment Setup

This section will guide you through setting up your local development environment step-by-step for a beginner.

### Clone the Repository

Start by cloning the repository:

```bash
git clone https://github.com/ethstorage/optimism.git
cd optimism
```

### Install Software Dependencies

Optimism uses [`mise`](https://mise.jdx.dev/) as a dependency manager to install and maintain various software dependencies necessary for development and testing. 
Once properly installed, `mise` will provide the correct versions for each tool.

1. #### Install the `mise` CLI
   Execute the following command to install `mise` with the latest version:
   ```bash
   curl https://mise.run | sh
   ```

2. #### Activate mise
   The `mise activate` command updates your environment variables and PATH every time your prompt is run to ensure you use the correct versions.
   
   Choose a command to execute based on your shell type:
   ```bash
   # for bash
   echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc

   # for zsh
   echo 'eval "$(~/.local/bin/mise activate zsh)"' >> ~/.zshrc
   ```

3. #### Verify `mise` Installation
   Check that `mise` is installed correctly:
   ```bash
   mise --version
   # Expected output: mise 2025.x.x
   ```

4. #### Trust the `mise.toml` File
   The `mise.toml` file lists the dependencies that this repository uses:
   ```bash
   mise trust mise.toml
   ```

5. #### Install Dependencies
   Use `mise` to install the required tools:
   ```bash
   mise install
   ```
   Note: You may see an error message after installation (e.g., `failed to install cargo:svm-rs@0.5.8`). This can be ignored temporarily, as it does not affect subsequent operations.

For more information on `mise` command, please refer to https://mise.jdx.dev/cli/.

### Check the Environment

Verify that the tools dependent are correctly installed with the required versions:
```bash
mise ls
```

### Check Kurtosis Status

Executing Go tests requires that the Kurtosis engine is running. Check its status with:

```bash
kurtosis engine status
```

Use the following command to start Kurtosis if it is not running:

```bash
kurtosis engine start
```

### Prepare RPC Endpoints

You will need access to Sepolia and Mainnet during the upcoming Go tests. Therefore, you should set the RPC endpoints as environment variables as in [this step](#step-5-set-environment-variables). 

Fortunately, RPC URLs with a free BlockPI API key will be sufficient for the tests. If you need assistance, please refer to [this link](https://docs.ethstorage.io/storage-provider-guide/tutorials#applying-for-ethereum-api-endpoints) for detailed instructions.

## Summary

By following this guide, you will be well-equipped to contribute effectively to EthStorage's Optimism Monorepo.
