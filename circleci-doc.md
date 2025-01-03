# CircleCI Memo

This document provides an overview of the changes made to the configuration and environment to enable the forked `Optimism` monorepo to successfully execute CircleCI tests on the new runner.

More importantly, it also outlines the necessary steps to locally verify code changes, whether from `ethstorage` or an upstream sync. Additionally, for certain tests, commands are provided to resolve any identified issues. 

---

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

---

### Repo Name Change

The repository name was updated from `develop` to `op-es` in the `packages/contracts-bedrock/scripts/checks/check-semver-diff.sh` file.

---

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

---

### Environment Variables

The following **Environment Variables** must be set in CircleCI under **Project Settings > Environment Variables** to enable tests that depend on forked testnet/mainnet environments (e.g., archive nodes):

- **`SEPOLIA_RPC_URL`**
- **`MAINNET_RPC_URL`**

> Archive nodes with a free BlockPI API key should be sufficient for these RPC URLs.

---

### Submodules

For repositories configured as submodules (e.g., `da-server`), code can be automatically synced during workflows due to an additional step is added to update submodules.

---

## Local Checks After Syncing With Upstream



### Contract Checks and Fixes

The `just pre-pr` command can be run to clean, build, lint, and validate all checks on the contracts:

```bash
cd packages/contracts-bedrock && just pre-pr
```

The following checks are included:

1. **Gas Snapshot Check (`gas-snapshot-check-no-build`):**  

   Ensures the gas snapshot is up-to-date. If not, you can update it using:

   ```bash
   just gas-snapshot-no-build
   ```

2. **Semgrep Test Validity Check (`semgrep-test-validity-check`):**  

   Ensures that semgrep tests are properly configured.

3. **Unused Imports Check (`unused-imports-check-no-build`):**  

   Flags unused imports in Solidity contracts.

4. **Snapshots Check (`snapshots-check-no-build`):**  

   Ensures that all snapshots are up-to-date. To fix, regenerate snapshots:

   ```bash
   just snapshots-no-build
   ```

5. **Lint Check (`lint-check`):**  

   Ensures contracts are properly linted. If there are lint errors, auto-fix them:

   ```bash
   just lint-fix
   ```

6. **Semver Diff Check (`semver-diff-check-no-build`):**  

   Ensures that contracts with modified `semver-lock` also have updated semver versions. To fix:

   ```bash
   just semver-lock
   ```

7. **Deploy Config Validation (`validate-deploy-configs`):**  

   Ensures that deploy configurations are valid.

8. **Spacer Variable Check (`validate-spacers-no-build`):**  

   Validates that spacer variables are inserted correctly without requiring a build.

9. **Interface Check (`interfaces-check-no-build`):**  

   Validates interfaces without requiring a build.

10. **Forge Test Linting (`lint-forge-tests-check-no-build`):**  

    Validates that Forge test names adhere to the correct format. 

### Go Tests


Firstly, check that go code is correctly linted.

```bash
make lint-go
```

Go tests also requires the contracts to be built:

```bash
make build
```
Build other necessary components:

```bash
cd op-program && make op-program-client && cd ..
cd cannon && make elf && cd ..
cd op-e2e && make pre-test && cd ..
```

Generate allocs:

```bash
make devnet-allocs
```


Make sure kurtosis is running before go tests:

```bash
kurtosis engine status
```

Run go tests with some environment variables provided:

```bash

export ENABLE_KURTOSIS=true
export OP_E2E_CANNON_ENABLED="false"
export OP_E2E_SKIP_SLOW_TEST=true
export OP_E2E_USE_HTTP=true
export ENABLE_ANVIL=true
# set the following rpc urls of your own
# export SEPOLIA_RPC_URL=
# export MAINNET_RPC_URL=

gotestsum --no-summary=skipped,output \
   --format=testname \
   --rerun-fails=2
```

Now check the result and make fixes if necessary. A success result may look like:

```bash
DONE 1764 tests, 2 skipped in 173.122s
```