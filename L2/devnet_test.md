# Devnet Manual Test

Helper code

```bash
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

function setup_op_geth() {
	cd $OP_GETH_HOME
  CGO_ENABLED=0 make geth # build geth without glibc dependency
  cp build/bin/geth $OP_HOME/ops-bedrock
  $OP_HOME/ops-bedrock/geth --version
}

export L1=http://localhost:8545
export L2=http://localhost:9545
export OP_HOME=<op repo dir>
export OP_GETH_HOME<op-geth repo dir>
```

Wallet code

```bash
# Addresses used in devnet
# 	Mnemonic:     "test test test test test test test test test test test junk",
#	CliqueSigner: "m/44'/60'/0'/0/0",
#	Proposer:     "m/44'/60'/0'/0/1",
#	Batcher:      "m/44'/60'/0'/0/2",
#	Deployer:     "m/44'/60'/0'/0/3",
#	Alice:        "m/44'/60'/0'/0/4",
#	SequencerP2P: "m/44'/60'/0'/0/5",
#	Bob:          "m/44'/60'/0'/0/7",
#	Mallory:      "m/44'/60'/0'/0/8",
#	SysCfgOwner:  "m/44'/60'/0'/0/9",
# See https://github.com/ethstorage/optimism/blob/6e133b884c7a1295fcf09184f0db23cac35d0922/op-e2e/e2eutils/secrets.go#L21
# Note that m/44'/60'/0'/0/x (x <= 29) are pre-filled with some tokens 

# Address conversion from HD wallet
cast wallet private-key "test test test test test test test test test test test junk"  "m/44'/60'/0'/0/1" # 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d 
cast wallet address 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d # 0x70997970C51812dc3A010C7d01b50e0d17dc79C8

# Create test accounts
PK=$(cast wallet private-key "test test test test test test test test test test test junk"  "m/44'/60'/0'/0/30")
ADDR=$(cast wallet address $PK)
cast balance $ADDR
cast balance $ADDR -r $L2
# Fund the test account
cast send $ADDR --value 10000000000000000000000 --private-key 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d
# Check balance
cast balance $ADDR

# Another account with zero balance
PK1=$(cast wallet private-key "test test test test test test test test test test test junk"  "m/44'/60'/0'/0/31")
ADDR1=$(cast wallet address $PK1)
cast balance $ADDR1
cast balance $ADDR1 -r $L2

# Another account with zero balance
PK2=$(cast wallet private-key "test test test test test test test test test test test junk"  "m/44'/60'/0'/0/32")
ADDR2=$(cast wallet address $PK2)
cast balance $ADDR2
cast balance $ADDR2 -r $L2
```

# Prepare op-geth

```bash
setup_op_geth

# edit l2-op-geth.Dockerfile
# unmark COPY geth /usr/local/bin/geth
```

# Run Devnet

```bash
make devnet-up
json_to_env .devnet/addresses.json
```

# Test Soul Gas Token

- Check code is deployed

```bash
# Check SGT code is deployed
cast codesize 0x4200000000000000000000000000000000000800 -r $L2

# Send token to L2
cast balance $ADDR -r $L2
cast send $OPTIMISM_PORTAL_PROXY --value 100000000000000000000 --private-key $PK

# Wait a few seconds and check
sleep 5
cast balance $ADDR -r $L2
```

- Deposit SGT for another user

```bash
cast call 0x4200000000000000000000000000000000000800 "balanceOf(address)" $ADDR1 -r $L2

# Deposit SGT to another account
cast send 0x4200000000000000000000000000000000000800 "batchDepositFor(address[],uint256[])" "[$ADDR1]" "[100000000000000000]" --value 100000000000000000 --private-key $PK -r $L2

cast call 0x4200000000000000000000000000000000000800 "balanceOf(address)" $ADDR1 -r $L2 # make sure the balance is 100000000000000000
```

- Spend SGT without native gas token

```bash
cast balance $ADDR1 -r $L2 # make sure it is zero
cast send $ADDR1 -r $L2 --private-key $PK1
cast call 0x4200000000000000000000000000000000000800 "balanceOf(address)" $ADDR1 -r $L2
```

- Spend SGT with native gas token

```bash
cast balance $ADDR1 -r $L2 # should be zero
cast send $ADDR1 --value 1000000000000000000 -r $L2 --private-key $PK
cast balance $ADDR1 -r $L2 # should be 1ETH

# record SGT balance
cast call 0x4200000000000000000000000000000000000800 "balanceOf(address)" $ADDR1 -r $L2 

# self send a tx
cast send $ADDR1 -r $L2 --private-key $PK1

cast balance $ADDR1 -r $L2 # should be 1ETH (unchanged)
cast call 0x4200000000000000000000000000000000000800 "balanceOf(address)" $ADDR1 -r $L2 # the balance should be reduced

# send all balance
cast send $ADDR2 --value $(cast balance $ADDR1 -r $L2) --private-key $PK1 -r $L2
cast balance $ADDR1 -r $L2 # should be zero
cast call 0x4200000000000000000000000000000000000800 "balanceOf(address)" $ADDR1 -r $L2 # the balance should be reduced
```

# Test FP

Submit an SGT tx and wait if the tx (and the block) is not reverted.

```bash
cd $OP_HOME
cp .devnet/rollup.json op-program/chainconfig/configs/901-rollup.json
cp .devnet/genesis-l2.json op-program/chainconfig/configs/901-genesis-l2.json
  
make verify-devnet
```

# Test L2 BLOB

```bash
# (optional) 
# run an extra da server. A da server is already included in the Docker container and is hosted at http://localhost:8888
git clone git@github.com:ethstorage/da-server.git

# prepare config
# run
go run main.go da start --config config.json

# add the server in docker-compose

# edit devnetL1-template.json 
# add l2GenesisBlobTimeOffset 0 

# run make devnet-clean

make devnet-up

# make sure .devnet/rollup.json has 
#   "l2_blob_config": {
#     "l2BlobTime": 0
#   }
  
```

```bash
# send a blob tx
cast send $ADDR --private-key $PK --blob --path <file> -r $L2
# get the blob datahash
cast tx <tx-hash> -r $L2
# download the blob (under da-server)
cd da-server
go run main.go da download --rpc http://localhost:8888 --blob_hash <blob-data-hash>
```
