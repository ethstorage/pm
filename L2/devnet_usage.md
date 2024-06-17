
# Basic Info

```bash
Entrance: https://quarkchain-b1ac26e1bc5a3c1f.testnets.rollbridge.app/
Explorer：http://142.132.154.16/
RPC: http://142.132.154.16:8545
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

After that you can deposit the claimed `Custom Gas Token` via [entrance](https://quarkchain-b1ac26e1bc5a3c1f.testnets.rollbridge.app/).

# Get Soul Gas Token On L2

```bash
export SOUL_GAS_TOKEN=0x4200000000000000000000000000000000000800
export L2_RPC_URL='http://142.132.154.16:8545'
export PRIVATE_KEY=''# input your own pk

cast send --value 10ether $SOUL_GAS_TOKEN 'deposit()' --private-key $PRIVATE_KEY -r $L2_RPC_URL
```


Then if you import `0x4200000000000000000000000000000000000800` into metamask, you'll see your balance of `Soul Gas Token`.

# Get L2 Blob from DA Server


DA Server Info:
```bash
DA Server: http://142.132.154.16:8888
DA Signer: 0x9268Dca89E7B1cD35F92d04DdC1De60Fb695696F
```

Get L2 Blob in two steps:

1. [Construct](https://github.com/ethstorage/da-server/blob/431639da87c37816293c2d4ca67e614c2dc372db/pkg/da/client/client.go#L25) a DA Client.
2. Get L2 blobs with [GetBlobs](https://github.com/ethstorage/da-server/blob/431639da87c37816293c2d4ca67e614c2dc372db/pkg/da/client/client.go#L68).