1. Run a Local L1 Devnet
```bash
make devnet-up
make devnet-down
docker compose up -d l1 l1-bn l1-vc
cast chain-id
```
2. Run L2 Components (op-geth, op-node, batcher, proposer)
 - Follow the [opup setup guide](https://github.com/zhiqiangxu/private_notes/blob/main/misc/beta_testnet_local_l1.md).

3. Restart op-node with Additional Options
```bash
export OP_NODE_CONDUCTOR_ENABLED=true
export OP_NODE_CONDUCTOR_RPC=<conductor-rpc-endpoint> # for example http://conductor:8545
```

4. Run op-conductor
```bash
# Build op-conductor
just op-conductor

# Example configuration
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

# Start op-conductor
./bin/op-conductor
```
5. Configure and Run [op-conductor-ops](https://github.com/ethereum-optimism/infra/tree/main/op-conductor-ops)
```bash
# Install dependencies. Follow readme.md to install poetry first
poetry install

# Configure op-conductor-ops
cp example.config.toml config.toml

# Example configuration
[networks.op-network-N]
sequencers = [
    "sequencer-0",
]
[sequencers.sequencer-0]
raft_addr = "<sequencer-0-ip-address>:50050"
conductor_rpc_url = "http://<sequencer-0-ip-address>:6660"
node_rpc_url = "http://<sequencer-0-ip-address>:8547"
voting = true

# Check the status of the first sequencer
./op-conductor-ops status op-network-N
```

6. Add Additional Sequencers
 - Deploy two sequencers (op-geth and op-node) on separate machines.
     - Enable sequencer mode for op-node: OP_NODE_SEQUENCER_ENABLED=true
     - Use the same sequencer key for op-node: OP_NODE_P2P_SEQUENCER_KEY
     - Add op-node options:
        ```bash
        export OP_NODE_CONDUCTOR_ENABLED=true
        export OP_NODE_CONDUCTOR_RPC=<conductor-rpc-endpoint>
        ```
     - To sync data from other sequencers, we need to add three options accroding to this [doc](https://github.com/ethstorage/pm/blob/main/L2/beta_testnet_new_node.md)
        ```bash
         --p2p.static # other two sequencers' address
         --p2p.no-discovery
         --p2p.sync.onlyreqtostatic
         ```
  - Wait for the new sequencer to start and get synced up with the rest of the nodes
  - Once the new sequencer is synced up, manually add it to the cluster by calling `conductor_addServerAsVoter` json rpc method on the leader sequencer. You can also use the `update_cluster_membership` command from `op-conductor-ops` to automate this task.
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

7. Update op-conductor-ops Configuration
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

10. Transfer Leadership using op-conductor-ops
```bash
./op-conductor-ops transfer-leader op-network-N <sequencer-id>
```

11. Demo:
 - [Recording](https://meeting.tencent.com/crm/2kkdpwDG32)
 - [Slides](https://docs.google.com/presentation/d/1x5Dpq67og7q7rHqT9YhA1Bf35QtvrHir0FBvgcuogyA/edit?usp=sharing)

12. Reference:
 - [op-conductor repo](https://github.com/ethstorage/optimism/tree/op-es/op-conductor)
 - [op-conductor runbook](https://github.com/ethstorage/optimism/blob/op-es/op-conductor/RUNBOOK.md)
 - [op-conductor doc](https://docs.optimism.io/builders/chain-operators/tools/op-conductor)
 - [op-conductor-ops repo](https://github.com/ethereum-optimism/infra/tree/main/op-conductor-ops)
 - [OP mainnet runbooks](https://oplabs.notion.site/OP-Mainnet-Runbooks-120f153ee1628045b230d5cd3df79f63)
 - [Discord discusstion](https://discord.com/channels/1244729134312198194/1299101976671420507/1325785755624407100)
