# Run [cannon](https://github.com/ethstorage/optimism/tree/develop/cannon)

Replace the `{APIKEY}` to yours in the following commands.

```
git clone https://github.com/ethstorage/optimism.git
# Build op-program server-mode and MIPS-client binaries.
cd op-program
make op-program # build

# Switch back to cannon, and build the CLI
cd ../cannon
make cannon

# Transform MIPS op-program client binary into first VM state.
# This outputs state.json (VM state) and meta.json (for debug symbols).
./bin/cannon load-elf --path=../op-program/bin/op-program-client.elf

# Run cannon emulator (with example inputs)
# Note that the server-mode op-program command is passed into cannon (after the --),
# it runs as sub-process to provide the pre-image data.
#
# Note:
#  - The L2 RPC is an archive L2 node on OP goerli.
#  - The L1 RPC is a non-archive RPC, also change `--l1.rpckind` to reflect the correct L1 RPC type.
./bin/cannon run \
    --pprof.cpu \
    --info-at '%10000000' \
    --proof-at never \
    --input ./state.json \
    -- \
    ../op-program/bin/op-program \
    --l2 https://optimism-goerli.infura.io/v3/{APIKEY} \
    --l1 https://goerli.infura.io/v3/{APIKEY} \
    --l1.trustrpc \
    --l1.rpckind debug_geth \
    --log.format terminal \
    --l2.head 0xedc79de4d616a9100fdd42192224580daee81ea3d6303de8089d48a6c1bf4816 \
    --network goerli \
    --l1.head 0x204f815790ca3bb43526ad60ebcc64784ec809bdc3550e82b54a0172f981efab \
    --l2.claim 0x530658ab1b1b3ff4829731fc8d5955f0e6b8410db2cd65b572067ba58df1f2b9 \
    --l2.blocknumber 8813570 \
    --datadir /tmp/fpp-database \
    --server

# Add --proof-at '=12345' (or pick other pattern, see --help)
# to pick a step to build a proof for (e.g. exact step, every N steps, etc.)

# Also see `./bin/cannon run --help` for more options

```

# Arb reference
- [replay code](https://github.com/OffchainLabs/nitro/blob/master/cmd/replay/main.go)
- Build: 
    1. follow [arbitrum build nitro tutorial](https://docs.arbitrum.io/node-running/how-tos/build-nitro-locally to install requirements
    2. `make build-wasm-bin`