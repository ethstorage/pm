
# Developer Guide to Optimism in EthStorage

  
This document outlines the instructions to contribute to EthStorage's Optimism Monorepo. 

As a quick starter, this doc will guide you setup local developement environment step by step, and also provide a standard process to run tests locally to verify code changes and help to pass CircleCI tests before merge to the default branch.

## Environment Setup

If you are the first time being a contributer to ethstorage's Optimism repo, please complete the following steps to set up your development environment.

### Clone the Repo

```bash
git clone https://github.com/ethstorage/optimism.git
cd optimism
```
### Install Software Dependencies

 Optimism uses [`mise`](https://mise.jdx.dev/) as a dependency manager to install and maintain  a number of software dependencies necessary to the development. Once properly installed, `mise` will provide the correct versions for each tool.

1. Install mise CLI
Execute the following command to install `mise` with latest version.
```bash
curl https://mise.run | sh
```
2. Activate mise
`mise activate` method updates your environment variable and PATH every time your prompt is run to ensure you use the correct versions.

```bash
echo 'eval "$(~/.local/bin/mise activate bash)"' >> ~/.bashrc
```
3. verify mise installation

```bash
mise --version
# mise 2025.x.x
```
4. Trust the mise.toml file
`mise.toml` is the config file which lists the dependencies that this repository uses.
```bash
mise trust mise.toml
```
5. Install dependencies
Use mise to install the correct versions for all of the required tools:
```bash
mise install
```
Note: You may see the following error on the console after the installation finished. You can ignore it temporarily because there is no indication that this error has any impact on subsequent operations.

```bash
   0: failed to install cargo:svm-rs@0.5.8
   1: cargo exited with non-zero status: exit code 101
```

`mise`  will notify you if any dependencies are outdated. Simply run  `mise install`  again to install the latest versions of the dependencies if you receive these notifications.

### Build the Monorepo

As a way to verify the correctness of environment setup, especially the installation of software dependency, build the repo:

```bash
make build
```

## Local Checks and Tests

### Solidity

Enter `contracts-bedrock` directory:

```bash
cd packages/contracts-bedrock
```

The following command cleans, builds, lints, and runs all checks on the contracts:

```bash
just pre-pr
```
Check [here]() for detailed explainations and possible fixes of certain errors.



Then run unit-tests using forge:

```bash
just test
```

#### Go

#### Step 1: Lint the Go Code

Before proceeding with tests, ensure that your Go code is correctly linted. Run the following command:

```bash
make lint-go
```

If any lint errors are detected, try to fix them using:

```bash
make lint-go-fix
```

Some of the errors may need to fix manually.

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
DONE 8567 tests, 95 skipped in 189.396s
```

#### Step 9: e2e Tests

```bash
cd op-e2e
make test-actions
make test-ws
```
For detailed information of e2e, please refer to https://github.com/ethstorage/optimism/blob/op-es/op-e2e/README.md.

### Contract Checks and Fixes In Detail

The following checks are included in the `just pre-pr` command:

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

**Forge Test Linting (`lint-forge-tests-check-no-build`):**  

    Validates that Forge test names adhere to the correct format. 