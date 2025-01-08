# Developer Guide to Optimism in EthStorage

This document provides instructions for contributing to EthStorage's Optimism Monorepo. It provides a practical process for running tests locally to verify code changes and ensure successful CircleCI tests before merging into the default branch. 

## Local Checks and Tests

It is good practice to do static checks and tests every time before merge code changes into the default branch. This section provides a step by step guide for this based on the two main progamming languages used: Solidity and Go.

If you are a first-time contributor to EthStorage's Optimism repository, or not yet using `mise` to manage your software dependencies, please complete [the steps to set up your development environment](#environment-setup).

### Solidity

Navigate to the `contracts-bedrock` directory:

```bash
cd packages/contracts-bedrock
```

#### Step 1: Static Checks

Run the following command to clean, build, lint, and check all contracts:

```bash
just pre-pr
```

For detailed explanations and possible fixes for certain errors, refer to [this section](#contract-checks-and-fixes-in-detail).

#### Step 2: Run unit tests:

The following command runs contracts tests using Forge:

```bash
just test
```

You may need to check the result and make necessary fixes and re-run the tests.

A success result may look like this:

```bash
Ran 282 test suites in 396.48s (1890.89s CPU time): 1763 tests passed, 0 failed, 1 skipped (1764 total tests)
```

### Go

Navigate to the root directory of your Optimism Monorepo.

#### Step 1: Lint

Before proceeding with tests, ensure that your Go code is correctly linted:

```bash
make lint-go
```

If any lint errors are detected, attempt to fix them:

```bash
make lint-go-fix
```
Note that some errors may need manual fixing.

#### Step 2: Build

Go tests require Go components as well as contracts to be built. Execute:

```bash
# Builds op-node, op-proposer, op-batcher, and contracts-bedrock
make build

# Build other essential components
cd op-program && make op-program-client && cd ..
cd cannon && make elf && cd ..
cd op-e2e && make pre-test && cd ..
```

#### Step 3: Generate Allocations

Allocations are also required by the following tests. To generate allocations, use:

```bash
make devnet-allocs
```

#### Step 5: Set Environment Variables

Prepare your environment by setting necessary variables like below. Replace the place holders with the RPC endpoints prepared in [this step](#prepare-rpc-endpoints).

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

#### Step 6: Run Go Tests

Execute the Go tests with:

```bash
gotestsum --no-summary=skipped,output \
   --format=testname \
   --rerun-fails=2
```

After the tests have been finished, check the console for any error or failures. A successful result should look like this:

```bash
DONE 8567 tests, 95 skipped in 189.396s
```

#### Step 7: End-to-End Tests

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

Optimism uses [`mise`](https://mise.jdx.dev/) as a dependency manager to install and maintain various software dependencies necessary for development and testing. Once properly installed, `mise` will provide the correct versions for each tool.

1. **Install the mise CLI**  
   Execute the following command to install `mise` with the latest version:
   ```bash
   curl https://mise.run | sh
   ```

2. **Activate mise**  
   The `mise activate` command updates your environment variables and PATH every time your prompt is run to ensure you use the correct versions:
   ```bash
   echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc
   ```

3. **Verify mise Installation**  
   Check that `mise` is installed correctly:
   ```bash
   mise --version
   # Expected output: mise 2025.x.x
   ```

4. **Trust the mise.toml File**  
   The `mise.toml` file lists the dependencies that this repository uses:
   ```bash
   mise trust mise.toml
   ```

5. **Install Dependencies**  
   Use `mise` to install the required tools:
   ```bash
   mise install
   ```
   Note: You may see an error message after installation (e.g., `failed to install cargo:svm-rs@0.5.8`). This can be ignored temporarily, as it does not affect subsequent operations.

6. **Build the Monorepo**

To verify that your environment setup is correct, especially regarding software dependencies, build the repository:

```bash
make build
```

7. **Check Kurtosis Status**

Executing Go tests requires that the Kurtosis engine is running. Check its status with:

```bash
kurtosis engine status
```
### Prepare RPC Endpoints

You will need access to Sepolia and Mainnet during the upcoming Go tests. Therefore, you should set the RPC endpoints as environment variables as in [this step](#step-5-set-environment-variables). 

Fortunately, RPC URLs with a free BlockPI API key will be sufficient for the tests. If you need assistance, please refer to [this link](https://docs.ethstorage.io/storage-provider-guide/tutorials#applying-for-ethereum-api-endpoints) for detailed instructions.


## Summary

By following this guide, you will be well-equipped to contribute effectively to EthStorage's Optimism Monorepo.
