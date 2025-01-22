1. Run an local L1 devnet
```bash
make devnet-up
make devnet-down
docker compose up -d l1 l1-bn l1-vc
cast chain-id
```
2. Run an L2 op-geth/op-node/batcher/proposer using [opup](https://github.com/zhiqiangxu/private_notes/blob/main/misc/beta_testnet_local_l1.md).

3. Restart op-node with two more options
```bash
export OP_NODE_CONDUCTOR_ENABLED=true
export OP_NODE_CONDUCTOR_RPC=<conductor-rpc-endpoint> # for example http://conductor:8545
```

4. Run op-conductor
```bash
just op-conductor # build

# An example about how to set the options
export OP_CONDUCTOR_CONSENSUS_ADDR=<Your_IP_Address>
export OP_CONDUCTOR_CONSENSUS_PORT=50050
export OP_CONDUCTOR_EXECUTION_RPC=http://localhost:8545
export OP_CONDUCTOR_HEALTHCHECK_INTERVAL=1
export OP_CONDUCTOR_HEALTHCHECK_MIN_PEER_COUNT=1  # set based on your internal p2p network peer count 
export OP_CONDUCTOR_HEALTHCHECK_UNSAFE_INTERVAL=7200 # recommend a 2-3x multiple of your network block time to account for temporary performance issues
export OP_CONDUCTOR_LOG_FORMAT=logfmt
export OP_CONDUCTOR_LOG_LEVEL=debug
export OP_CONDUCTOR_METRICS_ADDR=0.0.0.0
export OP_CONDUCTOR_METRICS_ENABLED=true
export OP_CONDUCTOR_METRICS_PORT=7300
export OP_CONDUCTOR_ROLLUP_CONFIG=./rollup.json
export OP_CONDUCTOR_NODE_RPC=http://localhost:8547 # your op-node RPC
export OP_CONDUCTOR_RAFT_SERVER_ID=sequencer-0
export OP_CONDUCTOR_RAFT_STORAGE_DIR=./conductor/raft
export OP_CONDUCTOR_RPC_ADDR=0.0.0.0
export OP_CONDUCTOR_RPC_ENABLE_ADMIN=true
export OP_CONDUCTOR_RPC_ENABLE_PROXY=true
export OP_CONDUCTOR_RPC_PORT=6660
export OP_CONDUCTOR_PAUSED=false
export OP_CONDUCTOR_RAFT_BOOTSTRAP=true

./bin/op-conductor # start
```
5. Run [op-conductor-ops](https://github.com/ethereum-optimism/infra/tree/main/op-conductor-ops)
```bash
# 1 install poetry and run "poetry install" first by following the readme.md

# 2
cp example.config.toml config.toml

# 3 config the op-network-N in config.toml
[networks.op-network-N]
sequencers = [
    "sequencer-0",
]
[sequencers.sequencer-0]
raft_addr = "<sequencer-0-ip-address>:50050"
conductor_rpc_url = "http://<sequencer-0-ip-address>:6660"
node_rpc_url = "http://<sequencer-0-ip-address>:8547"
voting = true

# 4. Run op-conductor-ops to check the status of the first sequencer
./op-conductor-ops status op-network-N
```

6. Now you have a single HA sequencer which treats itself as the cluster leader!

7. Follow the same process to add 2 more sequencers
 - Run an op-geth and op-node on a seperate machine.
     - op-node need also to be started as sequencer: OP_NODE_SEQUENCER_ENABLED=true
     - The sequcener key should be the same as the first sequencer: OP_NODE_P2P_SEQUENCER_KEY
     - The two conductor options are also needed to be specified: OP_NODE_CONDUCTOR_ENABLED and OP_NODE_CONDUCTOR_RPC
     - To sync data more quickly with other sequencers, we need to add three options accroding to this [doc](https://github.com/ethstorage/pm/blob/main/L2/beta_testnet_new_node.md)
         - --p2p.static # other two sequencers' address
         - --p2p.no-discovery
         - --p2p.sync.onlyreqtostatic
  - Wait for the new sequencer to start and get synced up with the rest of the nodes
  - Once the new sequencer is synced up, manually or use automation to add it to the cluster by calling `conductor_addServerAsVoter` json rpc method on the leader sequencer
      - ```bash
        curl -X POST -H "Content-Type: application/json" --data \
            '{"jsonrpc":"2.0","method":"conductor_addServerAsVoter","params":[<raft-server-id>, <raft-consensus-addr>, <raft-config-version>],"id":1}'  \
            http://127.0.0.1:8547
        ```
  - Resume the conductor on the new sequencer by calling `conductor_resume` json rpc method on op-conductor
      - ```bash
        curl -X POST -H "Content-Type: application/json" --data \
            '{"jsonrpc":"2.0","method":"conductor_resume","params":[],"id":1}'  \
            http://127.0.0.1:8547
        ```

8. Once finished, you should have a 3-node HA sequencer cluster.

9. Add the two new sequcencers into the op-conductor-ops toml config, and run op-conductor-ops
```bash
# install poetry and run "poetry install" first

cp example.config.toml config.toml

# config the op-network-N in config.toml
[networks.op-network-N]
sequencers = [
    "sequencer-0",
    "sequencer-1",
    "sequencer-2",
]
[sequencers.sequencer-0]
raft_addr = "<sequencer-0-ip-address>:50050"
conductor_rpc_url = "http://<sequencer-0-ip-address>:6660"
node_rpc_url = "http://<sequencer-0-ip-address>:8547"
voting = true

[sequencers.sequencer-1]
raft_addr = "<sequencer-1-ip-address>:50050"
conductor_rpc_url = "http://sequencer-1-ip-address:6660"
node_rpc_url = "http://sequencer-1-ip-address:8547"
voting = true

[sequencers.sequencer-2]
raft_addr = "<sequencer-2-ip-address>:50050"
conductor_rpc_url = "http://sequencer-2-ip-address:6660"
node_rpc_url = "http://sequencer-2-ip-address:8547"
voting = true
# Run op-conductor-ops to check the status of the first sequencer
./op-conductor-ops status op-network-N
```

10. Transfer leadership using op-conductor-ops
```bash
./op-conductor-ops transfer-leader op-network-N <sequencer-id>
```

