
# Steps To Setup A New Node For Devnet

First, prepare [rollup.json](./assets/beta_testnet_rollup.json) and [genesis.json](./assets/beta_testnet_genesis.json).

Then, follow the steps [here](https://docs.optimism.io/builders/chain-operators/tutorials/create-l2-rollup) to build `op-node` and `op-geth`, the only difference is: use the `beta_testnet` branch of both [optimism](https://github.com/ethstorage/optimism/tree/beta_testnet) and [op-geth](https://github.com/ethstorage/op-geth/tree/beta_testnet) instead.

Finally, setup `op-node` and `op-geth` following the steps below:

```bash
# assume optimism and op-geth repo are located at ./optimism and ./op-geth

cd op-geth

./build/bin/geth init --datadir=datadir --state.scheme hash genesis.json

openssl rand -hex 32 > jwt.txt

# We don't specify `--rollup.sequencerhttp` since it's for testing blob archiver only.
# The rpc port is the default one: 8545.
./build/bin/geth   --datadir ./datadir   --http   --http.corsdomain="*"   --http.vhosts="*"   --http.addr=0.0.0.0   --http.api=web3,debug,eth,txpool,net,engine   --ws   --ws.addr=0.0.0.0   --ws.port=8546   --ws.origins="*"   --ws.api=debug,eth,txpool,net,engine   --syncmode=full   --gcmode=archive   --nodiscover   --maxpeers=0   --networkid=3335   --authrpc.vhosts="*"   --authrpc.addr=0.0.0.0   --authrpc.port=8551   --authrpc.jwtsecret=./jwt.txt   --rollup.disabletxpoolgossip=true

 cp jwt.txt ../optimism/op-node 
 cd ../optimism/op-node

 export L1_RPC_KIND=basic

 export L1_RPC_URL=http://88.99.30.186:8545

 export L1_BEACON_URL=http://88.99.30.186:3500


# Ensure to replace --p2p.static with the sequencer's address.
# Note: p2p is enabled for unsafe block.
 ./bin/op-node   --l2=http://localhost:8551   --l2.jwt-secret=./jwt.txt   --verifier.l1-confs=4   --rollup.config=./rollup.json   --rpc.addr=0.0.0.0   --rpc.port=8547   --p2p.static=/ip4/65.109.20.29/tcp/9003/p2p/16Uiu2HAmP3KorAMS1DC5SdDEcNGwhMFKuoyvZzBSWXdqysZgrxQ7 --p2p.listen.ip=0.0.0.0 --p2p.listen.tcp=9003 --p2p.listen.udp=9003  --p2p.no-discovery --p2p.sync.onlyreqtostatic --rpc.enable-admin   --l1=$L1_RPC_URL   --l1.rpckind=$L1_RPC_KIND --l1.beacon=$L1_BEACON_URL --l1.beacon-archiver=http://65.108.236.27:9645

```