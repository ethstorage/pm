# Steps To Setup A New Node For Mainnet

## Geth
```bash
./go-ethereum/build/bin/geth \
  --datadir ./data \
  --port 30303 \
  --http --http.addr 0.0.0.0 --http.port 8545 \
  --http.corsdomain="*" \
  --http.vhosts="*" \
  --http.api eth,net,web3,txpool \
  --ws --ws.addr 0.0.0.0 --ws.port 8546 \
  --ws.origins="*" \
  --ws.api eth,net,web3,txpool \
  --authrpc.port 8551 \
  --authrpc.jwtsecret ./jwtmain.hex \
  --syncmode snap \
  --gcmode=archive \
```

## Prysm
```bash
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
cp ../go-ethereum/jwtmain.hex ./
```

```bash

 ./prysm.sh beacon-chain --execution-endpoint=http://localhost:8551   --jwt-secret=./jwtmain.hex   --datadir=./data   --rpc-host=127.0.0.1 --rpc-port=4201   --grpc-gateway-host=0.0.0.0 --grpc-gateway-port=4200   --p2p-tcp-port=13100 --p2p-udp-port=12105 --checkpoint-sync-url=https://beaconstate.info
```