# CircleCI Memo

This document provides an overview of the changes made to the configuration and environment to enable the forked `Optimism` monorepo to successfully execute CircleCI tests on the new runner.

More importantly, it also outlines the necessary steps to locally verify code changes, whether from `ethstorage` or an upstream sync. Additionally, for certain tests, commands are provided to resolve any identified issues. 

## Changes Made to Downstream

### Runner 

The most time-consuming jobs now run on the new runner machine `AX101` (65.108.230.142), including:

- `contracts-bedrock-checks`
- `fuzz-golang`
- `go-tests`

The following jobs **still run on Docker containers**, but may be moved to `AX101` later after further evaluation:

- `check-generated-mocks-op-node`
- `check-generated-mocks-op-service`
- `cannon-build-test-vectors`
- `todo-issues`



### Repo Name Change

The repository name was updated from `develop` to `op-es` in the `packages/contracts-bedrock/scripts/checks/check-semver-diff.sh` file.



### Tools

The following tools must be manually installed on the runner machine (`AX101`) for CircleCI workflows to run properly:

- `python3`
- `yq`
- `anvil`
- `semgrep`
- `kurtosis`
- `ripgrep` (`rg`)
- `golangci-lint`

> **Note**: All tools must be installed in `/usr/local/bin`, or their commands will not be found during execution.



### Environment Variables

The following **Environment Variables** must be set in CircleCI under **Project Settings > Environment Variables** to enable tests that depend on forked testnet/mainnet environments (e.g., archive nodes):

- **`SEPOLIA_RPC_URL`**
- **`MAINNET_RPC_URL`**

> Archive nodes with a free BlockPI API key should be sufficient for these RPC URLs.



### Submodules

For repositories configured as submodules (e.g., `da-server`), code can be automatically synced during workflows due to an additional step is added to update submodules.


## Local Checks After Syncing With Upstream

### Contract Checks and Fixes

The `just pre-pr` command can be run to clean, build, lint, and validate all checks on the contracts:

```bash
cd packages/contracts-bedrock && just pre-pr
```

The following checks are included:

**Gas Snapshot Check (`gas-snapshot-check-no-build`):**  

   Ensures the gas snapshot is up-to-date. If not, you can update it using:

   ```bash
   just gas-snapshot-no-build
   ```

**Semgrep Test Validity Check (`semgrep-test-validity-check`):**  

   Ensures that semgrep tests are properly configured.

**Unused Imports Check (`unused-imports-check-no-build`):**  

   Flags unused imports in Solidity contracts.

**Snapshots Check (`snapshots-check-no-build`):**  

   Ensures that all snapshots are up-to-date. To fix, regenerate snapshots:

   ```bash
   just snapshots-no-build
   ```

**Lint Check (`lint-check`):**  

   Ensures contracts are properly linted. If there are lint errors, auto-fix them:

   ```bash
   just lint-fix
   ```

**Semver Diff Check (`semver-diff-check-no-build`):**  

   Ensures that contracts with modified `semver-lock` also have updated semver versions. To fix:

   ```bash
   just semver-lock
   ```

**Deploy Config Validation (`validate-deploy-configs`):**  

   Ensures that deploy configurations are valid.

**Spacer Variable Check (`validate-spacers-no-build`):**  

   Validates that spacer variables are inserted correctly without requiring a build.

**Interface Check (`interfaces-check-no-build`):**  

   Validates interfaces without requiring a build.

1**Forge Test Linting (`lint-forge-tests-check-no-build`):**  

    Validates that Forge test names adhere to the correct format. 

### Go Tests

#### Step 1: Lint the Go Code

Before proceeding with tests, ensure that your Go code is correctly linted. Run the following command:

```bash
make lint-go
```

If any lint errors are detected, fix them using:

```bash
make lint-go-fix
```

#### Step 2: Build Contracts

Go tests require the contracts to be built. Execute the following command:

```bash
make build
```

#### Step 3: Build Necessary Components

Next, build other essential components by running these commands in sequence:

```bash
cd op-program && make op-program-client && cd ..
cd cannon && make elf && cd ..
cd op-e2e && make pre-test && cd ..
```

#### Step 4: Generate Allocations

To generate allocations, use:

```bash
make devnet-allocs
```

#### Step 5: Check Kurtosis Status

Ensure that the Kurtosis engine is running before executing Go tests. Check its status with:

```bash
kurtosis engine status
```

#### Step 6: Set Environment Variables

Prepare your environment by setting the necessary variables. You can do this by running:

```bash
export ENABLE_KURTOSIS=true
export OP_E2E_CANNON_ENABLED="false"
export OP_E2E_SKIP_SLOW_TEST=true
export OP_E2E_USE_HTTP=true
export ENABLE_ANVIL=true
# Set your own RPC URLs below:
export SEPOLIA_RPC_URL=<YOUR_SEPOLIA_RPC_URL>
export MAINNET_RPC_URL=<YOUR_MAINNET_RPC_URL>
```

#### Step 7: Run Go Tests

Now you are ready to execute the Go tests. Use the following command:

```bash
gotestsum --no-summary=skipped,output \
   --format=testname \
   --rerun-fails=2
```

#### Step 8: Review Test Results

After running the tests, check the results for any issues. A successful result should look like this:

```bash
DONE 1764 tests, 2 skipped in 173.122s
```

If there are any failures, make the necessary fixes and rerun the tests.

### All In One Script

```bash
#!/bin/bash

make lint-go-fix

cd packages/contracts-bedrock && just pre-pr && cd ../..

cd op-program && make op-program-client && cd ..
cd cannon && make elf && cd ..
cd op-e2e && make pre-test && cd ..

make devnet-allocs

export ENABLE_KURTOSIS=true
export OP_E2E_CANNON_ENABLED="false"
export OP_E2E_SKIP_SLOW_TEST=true
export OP_E2E_USE_HTTP=true
export ENABLE_ANVIL=true
export SEPOLIA_RPC_URL=<YOUR_SEPOLIA_RPC_URL>
export MAINNET_RPC_URL=<YOUR_MAINNET_RPC_URL>

gotestsum --no-summary=skipped,output \
   --format=testname \
   --rerun-fails=2
```