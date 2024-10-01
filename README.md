# Guide to setting up execution clients

**All ETH Execution Clients Are Supported!**

However, some may require more configuration and fine tweaking than others to target our blocktime. For that reason, we currently most recommend the following:

- [Reth](https://github.com/paradigmxyz/reth)
- [Geth](https://github.com/ethereum/go-ethereum)
- [Erigon](https://github.com/ledgerwatch/erigon)
- [Besu](https://github.com/hyperledger/besu)
- [Nethermind](https://github.com/NethermindEth/nethermind)
- [EthereumJS](https://github.com/ethereumjs/ethereumjs-monorepo)

## System Requirements

Before starting the setup process, please check that your machine meets the following requirements.

**Minimum Requirements:**

- OS: Linux
- CPU Architecture: AMD64 or ARM64
- CPU: 4 Physical Cores
- RAM: 24GB
- Storage: 2TB (SSD recommended), more (4TB) for an archive node.

## Installation

Each client comes with its own configuration and setup process. Follow the specific guidelines provided by the client's documentation. Overall, you are going to install the client, initialize it, start it up, and make sure it is accessible through its engine RPC for the beacond node to connect to.

### Step 1 - Genesis.json

There will be two separate genesis files as there are two different formats used. You will only need one genesis file based on your respective client, and each will be clearly labeled.


- `eth-genesis.json` - [https://github.com/berachain/beacon-kit/blob/main/testing/networks/80084/eth-genesis.json](https://github.com/berachain/beacon-kit/blob/main/testing/networks/80084/eth-genesis.json)
  - Reth
  - Geth
  - Erigon
  - Besu
  - EthereumJS
- `eth-nethermind-genesis.json` - [https://github.com/berachain/beacon-kit/blob/main/testing/networks/80084/eth-nether-genesis.json](https://github.com/berachain/beacon-kit/blob/main/testing/networks/80084/eth-nether-genesis.json)
  - Nethermind

### Step 2 - Initialize the chain

Some clients require you to initialize the chain first. Reth and Geth are the only ones that require this.

### Step 3 - Get the bootnode enode

```sh
bootnodes_url="https://raw.githubusercontent.com/berachain/beacon-kit/main/testing/networks/80084/el-bootnodes.txt";
bootnodes=$(curl -s "$bootnodes_url" | grep '^enode://' | tr '\n' ',' | sed 's/,$//');
```

### Step 3 - Create the service file

Create a new service file for the respective client you are using.

#### Reth

```sh
reth init --chain /path/to/eth-genesis.json
```

```ini
[Unit]
Description=Ethereum client
After=syslog.target network.target

[Service]
User=reth
Group=reth
Environment=HOME=/home/reth
Type=simple
ExecStart=reth node --authrpc.jwtsecret=/root/.beacond/config/jwt.hex --chain=/root/berachain-execution/eth-genesis.json --datadir=/root/berachain-execution --config=/data/config.toml --port=30303 --http --http.addr=0.0.0.0 --http.api="eth,net,web3,txpool,debug" --http.port=8545 --http.corsdomain=* --bootnodes=$bootnodes --trusted-peers=$bootnodes --ws --ws.addr=0.0.0.0 --ws.port=8546 --ws.origins=* --authrpc.addr=0.0.0.0 --authrpc.port=8551 --log.file.directory=/data/logs --metrics=0.0.0.0:6060
KillMode=process
KillSignal=SIGINT
TimeoutStopSec=90
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

#### Geth

Init the chain first:

```sh
 geth init --datadir /PATH/TO/ETH/DIR /PATH/TO/ETH/DIR/eth-genesis.json
```

```ini
[Unit]
Description=Ethereum client
After=syslog.target network.target

[Service]
User=geth
Group=geth
Environment=HOME=/home/geth
Type=simple
ExecStart=/home/geth/go-ethereum/build/bin/geth --datadir=/data --config=/config/geth.toml --bootnodes=$bootnodes --networkid=80084 --ipcpath=/data/geth.ipc --snapshot=false --syncmode=snap --http --http.addr=0.0.0.0 --http.api eth,net,web3,txpool,debug --http.port=8545 --http.vhosts=* --http.corsdomain=* --ws --ws.addr=0.0.0.0 --ws.port=8546 --ws.origins=* --authrpc.jwtsecret=/data/jwt.hex --authrpc.addr=0.0.0.0 --authrpc.port=8551 --authrpc.vhosts=* --metrics --metrics.addr=0.0.0.0 --metrics.port=6060
KillMode=process
KillSignal=SIGINT
TimeoutStopSec=90
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

*Static peers in geth.toml*
```toml
[Node.P2P]
StaticNodes = [
"enode://0401e494dbd0c84c5c0f72adac5985d2f2525e08b68d448958aae218f5ac8198a80d1498e0ebec2ce38b1b18d6750f6e61a56b4614c5a6c6cf0981c39aed47dc@34.159.32.127:30303","enode://9b6c1eb143c9e3af0c7283262a9a38fe8bf844114b1f304673c2ac1c23e6bccfdaa8f4e9cb8c460bded495933fd92eeff30e6ab2e0538b56e249beea2c512906@35.234.88.149:30303","enode://e9675164b5e17b9d9edf0cc2bd79e6b6f487200c74d1331c220abb5b8ee80c2eefbf18213989585e9d0960683e819542e11d4eefb5f2b4019e1e49f9fd8fff18@berav2-bootnode.staketab.org:30303","enode://16e21c20f670d9e88570b8d3c580c7ef54f3515bffab864f1f3047c4125c3e7d98e782b990165808363a1b54ddca51c9dafaca9d6cd7ecca93e2e809ba522cae@berachain-testnet-v2.enode.l0vd.com:30304","enode://e31aa249638083d34817eed2b499ccd4b0718a332f0ea530e3062e13f624cb03a7d6b6e0460193ee87b5fc12e73a726070a3126ef53492ffbdc5e6c102f6dfb3@34.64.198.56:30303","enode://3f2f85e2e711f198fb7324b74fab6a0599b2534774f3aa26241dbbabe870b650574324da01aa98ee24ce97c8d76362a2db03034a6ddff43119ccfdc269663cbf@34.47.79.13:30303","enode://7a2f67d22b12e10c6ba9cd951866dda6471604be5fbd5102217dbad1cc56e590befd2009ecc99958a468a5b8e0dc28e14d9b6822491719c93199be6aa0319077@34.124.220.31:30303","enode://a96aac0b81c7e75fecc2ae613eaf13b27b2aaf3d46a90db904f94797d1746aa31e6593ae4cd476f81d5c6d1d2228ca60c885727978c369586c38871c63a330ee@35.240.182.27:30303","enode://dc44744074ac2dd76db0e0f9d95eb86cd558f6ba75e4a4af1303f2259624c8ce041198f976862a284165253b6dc6b2fa91b995cbca3ef2683879b6247e05e553@34.95.61.239:30303","enode://bf5364e1cf7ecd11646ccaea5c06b56622c04d52200d9cd141e01db9c9661237ceebecde1616e66e390a968ffd1c07e027531cad23044517b7bf36caa8b97f5f@34.152.41.26:30303","enode://f61e51c18fdb6ddf5e520209c53a0e60b2864d168eb0d3c02541050de9fee003b61818c7f70b32b61adee082280e7de4811fd3da47d87c87b3d17bf44e3bb76c@beacond-testnet.blacknodes.net:30303","enode://f24b54da77cf604e92aeb5ee5e79401fd3e66111563ca630e72330ccab6f385ccbbde5eba4577ee7bfb5e83347263d0e4cad042fd4c10468d0e38906fc82ba31@bera-testnet-seeds.nodeinfra.com:30303","enode://2e44e8e12b4666632dd2d4d555cfca5ceac4ca6cf6f45c46fc0ba27d1f9f7578dd598c74ae8b4189430a85b15d103c215a63cdbeafd41895fee1405a094fa77a@135.125.188.10:30303"]
```

### Nethermind

```ini
[Unit]
Description=Ethereum client
After=syslog.target network.target

[Service]
User=nethermind
Group=nethermind
Environment=HOME=/home/nethermind
Type=simple
ExecStart=nethermind --datadir=/root/berachain-nethermind --baseDbPath=/root/berachain-nethermind --JsonRpc.JwtSecretFile=/root/.beacond/config/jwt.hex --Network.Bootnodes=$bootnodes --Network.StaticPeers=$bootnodes --JsonRpc.Enabled=true --JsonRpc.EnabledModules=eth,net,web3,engine,txpool,debug --JsonRpc.Host=0.0.0.0 --JsonRpc.Port=8545 --Init.WebSocketsEnabled=true --JsonRpc.WebSocketsPort=8546 --JsonRpc.EngineHost=0.0.0.0 --JsonRpc.EnginePort=8551 --Init.ChainSpecPath=/root/berachain-nethermind/eth-nether-genesis.json --Network.MaxActivePeers=300 --Sync.PivotNumber=0
KillMode=process
KillSignal=SIGINT
TimeoutStopSec=90
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

#### Besu

```ini
[Unit]
Description=Ethereum client
After=syslog.target network.target

[Service]
User=besu
Group=besu
Environment=HOME=/home/besu
Type=simple
ExecStart=besu --data-path=/root/berachain-besu --genesis-file=/root/berachain-besu/eth-genesis.json --static-nodes-file=/root/berachain-besu/static-nodes.json --max-peers=300 --rpc-http-enabled --rpc-http-api=ETH,NET,ENGINE,DEBUG,NET,WEB3 --host-allowlist="*" --rpc-http-cors-origins="all" --engine-rpc-port=8551 --engine-rpc-enabled --engine-host-allowlist="*" --engine-jwt-secret=/root/.beacon/config/jwt.hex
KillMode=process
KillSignal=SIGINT
TimeoutStopSec=90
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

Example config file `static-nodes.json`:

```
[
"enode://0401e494dbd0c84c5c0f72adac5985d2f2525e08b68d448958aae218f5ac8198a80d1498e0ebec2ce38b1b18d6750f6e61a56b4614c5a6c6cf0981c39aed47dc@34.159.32.127:30303",
"enode://9b6c1eb143c9e3af0c7283262a9a38fe8bf844114b1f304673c2ac1c23e6bccfdaa8f4e9cb8c460bded495933fd92eeff30e6ab2e0538b56e249beea2c512906@35.234.88.149:30303",
"enode://e9675164b5e17b9d9edf0cc2bd79e6b6f487200c74d1331c220abb5b8ee80c2eefbf18213989585e9d0960683e819542e11d4eefb5f2b4019e1e49f9fd8fff18@berav2-bootnode.staketab.org:30303",
"enode://16e21c20f670d9e88570b8d3c580c7ef54f3515bffab864f1f3047c4125c3e7d98e782b990165808363a1b54ddca51c9dafaca9d6cd7ecca93e2e809ba522cae@berachain-testnet-v2.enode.l0vd.com:30304","enode://7a2f67d22b12e10c6ba9cd951866dda6471604be5fbd5102217dbad1cc56e590befd2009ecc99958a468a5b8e0dc28e14d9b6822491719c93199be6aa0319077@34.124.220.31:30303","enode://a96aac0b81c7e75fecc2ae613eaf13b27b2aaf3d46a90db904f94797d1746aa31e6593ae4cd476f81d5c6d1d2228ca60c885727978c369586c38871c63a330ee@35.240.182.27:30303"
]
```

#### Erigon

```ini
[Unit]
Description=Ethereum client
After=syslog.target network.target

[Service]
User=erigon
Group=erigon
Environment=HOME=/home/erigon
Type=simple
ExecStart=erigon --authrpc.jwtsecret=/data/jwt.hex --datadir=/data --http=false --authrpc.addr=0.0.0.0 --authrpc.port=8551 --authrpc.vhosts=* --ws --ws.addr=0.0.0.0 --ws.port=8546 --ws.origins=* --authrpc.addr=0.0.0.0 --authrpc.port=8551 --log.file.directory=/data/logs --metrics=0.0.0.0:6060
KillMode=process
KillSignal=SIGINT
TimeoutStopSec=90
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

#### EthereumJS

TODO
