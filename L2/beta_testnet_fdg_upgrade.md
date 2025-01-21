1. Upgrade ASR：
    ```bash
    forge script --sig 'run(address,address,address,address,address,uint32,uint256,bytes32)' \
        scripts/deploy/UpgradeAnchorStateRegistry.s.sol:UpgradeAnchorStateRegistry \
        $DISPUTE_GAME_FACTORY_PROXY_ADDRESS $OP_PROXY_ADMIN_ADDRESS \
        $ANCHOR_STATE_REGISTRY_PROXY_ADDRESS $SUPERCHAIN_CONFIG_PROXY_ADDRESS \
        0x0000000000000000000000000000000000000000 \
        0 0 0x723104efca73c6fc662a784365f6874a13dc1f781c1968a4240fb56a8212218a \
        --rpc-url $L1_RPC_URL --private-key $GS_ADMIN_PRIVATE_KEY --broadcast
    ```
    1.  fetch genesis output root：`0x723104efca73c6fc662a784365f6874a13dc1f781c1968a4240fb56a8212218a`
        ```bash
        $ curl -X POST -H "Content-Type: application/json" --data  \
        '{"jsonrpc":"2.0","method":"optimism_outputAtBlock","params":["0x0"],"id":1}' \
        http://5.9.87.214:8547
        ```  
2. Ensure these files exist：
```
op-program/chainconfig/configs/3335-genesis-l2.json
op-program/chainconfig/configs/3335-rollup.json
```
3. Run `make reproducible-prestate` to get correct absolute prestate：`0x03972e7f423d6ad3a7cdc9537ac452f1e3ebe3bd2e4c6bb5c5ca3edeec5c2908`
4. Attributes from the permissioned FDG can be queried like below：
```bash
# cast call $DISPUTE_GAME_FACTORY_PROXY_ADDRESS 'gameImpls(uint32)(address)' 1 -r $L1_RPC_URL
0xdE5e5470c67CAEA0A9bd376D6b26511eaB31EaEF
# cast call 0xdE5e5470c67CAEA0A9bd376D6b26511eaB31EaEF "maxGameDepth()(uint256)" -r $L1_RPC_URL
73
# cast call 0xdE5e5470c67CAEA0A9bd376D6b26511eaB31EaEF "splitDepth()(uint256)" -r $L1_RPC_URL
30
# cast call 0xdE5e5470c67CAEA0A9bd376D6b26511eaB31EaEF "clockExtension()(uint64)" -r $L1_RPC_URL
10800
# cast call 0xdE5e5470c67CAEA0A9bd376D6b26511eaB31EaEF "maxClockDuration()(uint64)" -r $L1_RPC_URL
302400
# cast call 0xdE5e5470c67CAEA0A9bd376D6b26511eaB31EaEF "vm()(address)" -r $L1_RPC_URL
0xd2CD9D02fD5F58a42e2496F82254F0a1CcC3FA29
```
5. Run the command below to deploy a permission-less FDG with correct absolute prestate：
```bash
op-deployer/bin/op-deployer bootstrap disputegame --l1-rpc-url $L1_RPC_URL --private-key $GS_ADMIN_PRIVATE_KEY \
    --artifacts-locator "file:///root/xu/beta_testnet/optimism/packages/contracts-bedrock/forge-artifacts/" \
    --vm 0xd2cd9d02fd5f58a42e2496f82254f0a1ccc3fa29 --game-kind FaultDisputeGame --game-type 0 \
    --absolute-prestate 0x03972e7f423d6ad3a7cdc9537ac452f1e3ebe3bd2e4c6bb5c5ca3edeec5c2908 \
    --max-game-depth 73 --split-depth 30 --clock-extension 10800 --max-clock-duration 302400 \
    --delayed-weth-proxy $DELAYED_WETHPERMISSIONED_GAME_PROXY_ADDRESS \
    --anchor-state-registry-proxy $ANCHOR_STATE_REGISTRY_PROXY_ADDRESS --l2-chain-id 3335 
(Note the address of the deployed contract as DisputeGameImpl)
```
6. Run the command below to update the above FDG to DisputeGameFactory：
```bash
cast send $DISPUTE_GAME_FACTORY_PROXY_ADDRESS "setImplementation(uint32,address)" 0 <DisputeGameImpl> -r $L1_RPC_URL
```
7. Run the command below to set portal's `respectedGameType` to permission-less FDG:
```bash
cast send $OPTIMISM_PORTAL_PROXY_ADDRESS "setRespectedGameType(uint32)" 0 -r $L1_RPC_URL
```
8. Set the game-type of the op-proposer to permission-less FDG：
```bash
 ./bin/op-proposer --poll-interval=12s --rpc.port=8560 --rollup-rpc=http://localhost:8547 \
                              --game-factory-address=$DISPUTE_GAME_FACTORY_PROXY_ADDRESS \
                              --proposal-interval 12h --game-type 0 \
                              --private-key=$GS_PROPOSER_PRIVATE_KEY --l1-eth-rpc=$L1_RPC_URL 2>&1 | tee -a proposer.log -i
```
9. Start `op-challenger`:
```bash
cd op-challenger
mkdir datadir
bin/op-challenger --l1-eth-rpc $L1_RPC_URL --l1-beacon $L1_BEACON_URL \ 
    --l2-eth-rpc http://localhost:8545 --rollup-rpc http://localhost:8547 \
    --datadir ./datadir --cannon-server ../op-program/bin/op-program --cannon-bin ../cannon/bin/cannon \ 
    --cannon-prestate $(realpath ../op-program/bin/prestate.bin.gz) --private-key $GS_CHALLENGER_PRIVATE_KEY \ 
    --cannon-rollup-config $(realpath ../op-program/chainconfig/configs/3335-rollup.json) \
    --cannon-l2-genesis $(realpath ../op-program/chainconfig/configs/3335-genesis-l2.json) \ 
    --game-factory-address 0x4b2215d682208b2a598cb04270f96562f5ab225f --trace-type cannon

```
10. Make sure that `make verify-beta-testnet` under op-program passes.
