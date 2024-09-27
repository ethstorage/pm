
# Basic Info

```bash
Entrance: https://bridge.alpha.testnet.worldsuper.computer
Faucet: https://wsc-faucet.eth.sep.w3link.io/
Explorer：https://explorer.alpha.testnet.worldsuper.computer or http://65.109.20.29/
RPC: https://rpc.alpha.testnet.worldsuper.computer:8545 or http://65.109.20.29:8545 
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

After that you can cross the claimed `Custom Gas Token` to L2 via `entrance` or follow the instructions [here](https://github.com/ethereum-optimism/specs/discussions/140#discussioncomment-9426636).

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

# EthStorage
```bash
Storage contract: 0x64003adbdf3014f7E38FC6BE752EB047b95da89A
RPC: https://rpc.testnet.l2.ethstorage.io:9540 or http://65.109.115.36:9540
Chain id: 3336
```
```bash
// upgrade ethfs-cli to the latest first
$ npm install -g ethfs-cli

// deploy a FlatDirectory contract
$ ethfs-cli create -p <private key> -c 43069

// upload your application using file upload type 2
$ ethfs-cli upload -f <your application folder> -a <flat directory address> -c 43069 -p <private key> -t 2

// visit it using gateway
https://<flat_directory_address>.3336.w3link.io/index.html

// set the default page path
$ ethfs-cli default -a <flat_directory_address> -f <index.html> -p <private_key> -c 43069

// visit it using gateway like this
https://<flat_directory_address>.3336.w3link.io/

// register an ENS name like <my-dapp.eth> on sepolia
// and add a text record: name is "contentcontract", and value is "esl2-t:<flat_directory_address>"
// you can visit it by the following link:
https://my-dapp.eth.sep.w3link.io/
```
