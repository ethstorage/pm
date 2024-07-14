
# Basic Info

```bash
Entrance: N/A
Explorer：http://65.109.20.29/
RPC: http://65.109.20.29:8545
Custom Gas Token: 0xe6ABD81D16a20606a661D4e075cdE5734AB62519
```

# Get Custom Gas Token On L1

First, ensure you've some sepolia gas, otherwise go [here](https://www.alchemy.com/faucets/ethereum-sepolia) for faucet.

Then invoke the `drop` function on etherscan [here](https://sepolia.etherscan.io/address/0x274a6990dE7AaE06452cbEFa266c0C6a568F0D5B#writeContract).

Or simply run this:
```bash
export L1_RPC_URL='http://88.99.30.186:8545'
export PRIVATE_KEY=''# input your own pk

cast send 0x274a6990dE7AaE06452cbEFa266c0C6a568F0D5B 'drop()' --private-key $PRIVATE_KEY -r $L1_RPC_URL
```

After that you can deposit the claimed `Custom Gas Token` via `entrance` or follow the instructions [here](https://github.com/ethereum-optimism/specs/discussions/140#discussioncomment-9426636).

# Get Soul Gas Token On L2

```bash
export SOUL_GAS_TOKEN=0x4200000000000000000000000000000000000800
export L2_RPC_URL='http://65.109.20.29:8545'
export PRIVATE_KEY=''# input your own pk

cast send --value 10ether $SOUL_GAS_TOKEN 'deposit()' --private-key $PRIVATE_KEY -r $L2_RPC_URL
```


Then if you import `0x4200000000000000000000000000000000000800` into metamask, you'll see your balance of `Soul Gas Token`.

# Get L2 Blob From DA Server


DA Server Info:
```bash
DA Server: http://65.109.20.29:8888
```

Get L2 Blob in two steps:

1. [Construct](https://github.com/ethstorage/da-server/blob/ed2ee4ff52d9f08231708b0a88c23838a39e3c27/pkg/da/client/client.go#L22) a DA Client.
2. Get L2 blobs with [GetBlobs](https://github.com/ethstorage/da-server/blob/ed2ee4ff52d9f08231708b0a88c23838a39e3c27/pkg/da/client/client.go#L92).

Or by [this](https://github.com/ethstorage/da-server) cli tool like following:


```bash
git clone https://github.com/ethstorage/da-server
cd da-server
go run main.go da download --rpc http://65.109.20.29:8888 --blob_hash 01314c3f1d37db90fed33fc52516505cbfa37bfc704963dfef776ef4ef52ab4f 
```
(Replace `blob_hash` parameter accordingly.)
