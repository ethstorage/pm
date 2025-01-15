Run Fault Proof on Devnet
=========================

# Setup Helper Functions
```
function json_to_env() {
  for key in $( jq -r 'to_entries|map("\(.key)")|.[]' $1 ); do
    skey=$(echo $key | sed -r 's/([a-z0-9])([A-Z])/\1_\L\2/g' | sed -e 's/\(.*\)/\U\1/')
    value=$(jq \.$key $1 | tr -d \")
    echo $skey=$value
    export $skey=$value
  done
}

function setup_devnet_prestate() {
  cd $OP_HOME
  cp .devnet/rollup.json op-program/chainconfig/configs/901-rollup.json
  cp .devnet/genesis-l2.json op-program/chainconfig/configs/901-genesis-l2.json
  cd op-program
  make reproducible-prestate
}
```

# Download Repo

```
git clone https://github.com/ethstorage/optimism.git
cd optimism
git checkout fp-devnet
OP_HOME=$(realpath .)
```

# Run Devnet and op-challenger
(Note: op-proposer and op-challenger are not started on `fp-devnet` branch)

```
make devnet-up
```
In other branches instead of fp-devnet, the following changed are required:
```
# Because devnet's default gameType is FastGame and it's maxClockDuration is hardcoded to 0, sending attack tx to
# the dispute game contact would fail. So, we should enforce devnet's faultGameClockExtension to use the config
# (already done in the fp-devnet branch) in `packages/contracts-bedrock/deploy-config/devnetL1-template.json`
sed -i '913s/0/uint64(cfg.faultGameMaxClockDuration())/' packages/contracts-bedrock/scripts/deploy/Deploy.s.sol
# (already done in the fp-devnet branch) change the following fields in packages/contracts-bedrock/deploy-config/devnetL1-template.json
  "faultGameClockExtension": 80,  # must be > 0, all units are second
  "faultGameMaxClockDuration": 200, # 2 * faultGameClockExtension + preimageOracleChallengePeriod <= faultGameMaxClockDuration
  "preimageOracleChallengePeriod": 40,
```

> `SplitDepth` must be greater than the `op-proposal`'s proposal interval (in seconds) divided by 2 (block time) because the `op-challenger` can only attack blocks within the range [offset, offset + 2^SplitDepth]. If `SplitDepth` is too small, the honest output root will always be attacked, preventing the anchor state from being updated. Additionally, `MaxGameDepth - SplitDepth` must be greater than the number of cannon instructions to meet the `_verifyExecBisectionRoot` requirement of the FaultDisputeGame contract.

## (Optional) Restart a Clean Devnet

```
make devnet-down
make devnet-clean
```

# Deploy Fault Proof with Devnet Absolute Prestate

## Build op-program with Devnet Config

```
$ setup_devnet_prestate
...
Cannon Absolute prestate hash:
0x0384fd1b3e1f41288e21b315f67a691c24bf420d7fb53f9b41f6cad80dc78a50
```

## Verify Without Cannon
```
$ make verify-devnet
...
t=2024-10-15T17:46:52+0000 lvl=info msg="Validating claim" head=0x0a33510461b206f7e178d152809477bdb7336399a5f12d6dd71f7b7de58bf8ca:397 output=0x73b217962cdcdf5b878389222d79ff5c07b0ec8c7749d5bed8ead353836e7faa claim=0x73b217962cdcdf5b878389222d79ff5c07b0ec8c7749d5bed8ead353836e7faa
```
which will output the claim (output root) and its l2 block number(head=xx:blocknumber)

## Update Config

Replace the prestate hash / genesis block / genesis output root in `deploy-config/devnetL1.json`.

## Deploy Fault Proof Implementations

```
export IMPL_SALT=$(openssl rand -hex 32)
DEPLOY_CONFIG_PATH=deploy-config/devnetL1.json DEPLOYMENT_INFILE=deployments/devnetL1/.deploy forge script scripts/deploy/Deploy.s.sol:Deploy --sig deployFaultProofImplementations --private-key 0x2a871d0798f97d79848a013d4936a73bf4cc922c825d33c1cf7073dff6d409c6 --rpc-url http://localhost:8545  --broadcast
```

## Verify Deployments
```
json_to_env $OP_HOME/.devnet/addresses.json
## verify correct gameType(0) is set
cast call $OPTIMISM_PORTAL_PROXY "respectedGameType()"
FP_IMPL=$(cast call $DISPUTE_GAME_FACTORY_PROXY 'gameImpls(uint32)' 0 | cut -b 1,2,27-66 )
echo "Fault Proof Impl" $FP_IMPL
cast call $FP_IMPL 'absolutePrestate()'
ASR=$(cast call $FP_IMPL 'anchorStateRegistry()' | cut -b 1,2,27-66)
echo "Anchor State Registry" $ASR
echo "Anchor root" $(cast call $ASR 'anchors(uint32)' 0)
```

Make sure the prestate and anchor output root are correct.

# Run op-proposer & op-challenger
- build op-proposer and op-challenger (make sure your `just` is latest version):
```
make op-proposer && make op-challenger && make cannon
```
- run
```
op-proposer/bin/op-proposer --l1-eth-rpc http://localhost:8545  --rollup-rpc http://localhost:7545 --proposal-interval 60s --poll-interval 1s --num-confirmations 1 --private-key 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d --game-factory-address $DISPUTE_GAME_FACTORY_PROXY --rpc.port 10545

op-challenger/bin/op-challenger --l1-eth-rpc http://localhost:8545 --l1-beacon http://localhost:5052 --l2-eth-rpc http://localhost:9545 --rollup-rpc http://localhost:7545 --datadir /tmp/op-chg-db --cannon-server op-program/bin/op-program --cannon-bin cannon/bin/cannon --cannon-prestate $(realpath op-program/bin/prestate.bin.gz) --private-key 0x47e179ec197488593b187f80a00eb0da91f1b9d0b13f8733639f19c30a34926a --cannon-rollup-config $(realpath op-program/chainconfig/configs/901-rollup.json) --cannon-l2-genesis $(realpath op-program/chainconfig/configs/901-genesis-l2.json) --game-factory-address $DISPUTE_GAME_FACTORY_PROXY --trace-type cannon
```

# Verify with op-challenger and cannon
```
op-challenger/bin/op-challenger run-trace --l1-eth-rpc http://localhost:8545 --l1-beacon http://localhost:5052 --l2-eth-rpc http://localhost:9545 --rollup-rpc http://localhost:7545 --datadir /tmp/op-chg-db --cannon-server op-program/bin/op-program --cannon-bin cannon/bin/cannon --cannon-prestate $(realpath op-program/bin/prestate.bin.gz) --private-key 0x47e179ec197488593b187f80a00eb0da91f1b9d0b13f8733639f19c30a34926a --cannon-rollup-config $(realpath op-program/chainconfig/configs/901-rollup.json) --cannon-l2-genesis $(realpath op-program/chainconfig/configs/901-genesis-l2.json) --game-factory-address $DISPUTE_GAME_FACTORY_PROXY --trace-type cannon
```

# Do something bad
## attack with bad claim
- [simple python script](https://github.com/dajuguan/op-notes/blob/main/play-op-challenger.py)
  1. Create a game with a false output root for the latest L2 block number.
  2. Attack with random false claims and wait for the `op-challenger` to counter until reaching `maxGameDepth`.
  3. Wait for the `op-challenger` to call step when `maxGameDepth` is even.
## With wrong prestate hash (or rollup config)
## With wrong anchor output root
op-challenger will throw the following error:
```
failed to create job for game 0xd5FeCA89e82973B1eb337A2082C59fB248F8e7d8: failed to validate prestate: failed to validate prestate: output root absolute prestate does not match:
Provider: 0x9d0979edf7b21a77848048c9377cc077cfa5f8fd035393ede118a77a7db35e8e |
Contract: 0x000000000000000000000000000000000000000000000000000000000000ffff
```
