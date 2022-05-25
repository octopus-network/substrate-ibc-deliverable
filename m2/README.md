# Substrate-IBC Milestone 2 Deliverable

## This milestone includes the following goals:
* Testnet involving two IBC-enabled Substrate chains capable of transferring a token following ICS 020
* CLI and UI supporting the testnet


## Related artifacts

| Number | Deliverable       | Comments                                                |
| ------ | ----------------- | ------------------------------------------------------------ |
| 1.     | Substrate chain  |     [repo](https://github.com/octopus-network/substrate/)   |
| 2.     | Relayer |   [repo](https://github.com/octopus-network/ibc-rs/)   |
| 3.     | Substrate-Ibc Pallet | [repo](https://github.com/octopus-network/substrate-ibc) |
| 4.     | ibc frontend demo | [repo](https://github.com/octopus-network/ibc-frontend-demo/) |
| 5.     | Design Spec       |  [Design](./design.md)                                                            |
| 6.     | Operation Guide    |  Right below                                                            |
| 7.     | Video Demo           | [Video]() |

## Operation Guide

### Transfer tokens by CLI
*  Launch 2 IBC Enabled Substrate Chains Locally
```bash
git clone --branch feature/beefy https://github.com/octopus-network/substrate.git
cd substrate
git submodule update --init
git pull
rm bin/node-template/octopus-pallets/Cargo.toml
rm -rf .ibc-*
cargo build -p node-template # generate ./target/debug/node-template

# in terminal 1: lanch a chain to be recognized as ibc-0 by the relayer
./target/debug/node-template --dev -d .ibc-0 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true

# in terminal 2: lanch a chain to be recognized as ibc-1 by the relayer
 ./target/debug/node-template --dev -d .ibc-1 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true --port 2033 --ws-port 8844


```
* explore the chains info and events via polkadot.js:   
    https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer  
    https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A8844#/explorer


* Prepare the Relayer
```bash
git clone --branch ibc-m2-rc1 https://github.com/octopus-network/ibc-rs.git
cd ibc-rs
git checkout feature/ics20
cargo build
hermes version
```
* Start mmr root update service
```bash
git clone https://github.com/octopus-network/octopusxt.git
cd octopusxt
sleep 10;cargo test test_update_client_state_service -- --nocapture 

```
* Create channel between two substrate nodes
```bash
RUST_BACKTRACE=full  
hermes -c  config.toml create channel ibc-0 ibc-1 --port-a transfer --port-b transfer -o unordered 

# in terminal 4: start mmr root update service
git clone https://github.com/octopus-network/octopusxt.git
cd octopusxt
sleep 10;
cargo test test_update_client_state_service -- --nocapture 

# in terminal 5: start a relayer
RUST_BACKTRACE=full 
hermes -c config.toml start  # Without MMR update service

```

* transfer a token
```bash
# in terminal 3: trigger packet transfer
hermes -c config.toml tx raw ft-transfer ibc-1 ibc-0 transfer channel-0 9999 -o 9999 -n 1 -t 9999
# transfer back to the src chain

```
* Screen shot

### Transfer a token by UI
* Install UI demo
```bash
git clone git@github.com:octopus-network/ibc-frontend-demo.git
cd substrate-front-end-template
yarn install
yarn start
```
* Open your browser and visit http://localhost:8000/
* Transfer tokens by ui
* 附截图和说明


### Video Demo
  Pls refer to [Youtube](https://www.youtube.com/watch?v=MLdwqpAu_ZA)
### Issues

