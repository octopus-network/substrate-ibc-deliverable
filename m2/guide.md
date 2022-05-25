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

mv substrate substrate0
# copy substrate0 as substrate1
cp -rvf substrate0 substrate1

# in terminal 1: build and lanch a chain to be recognized as ibc-0 by the relayer
cd substrate0
cargo build -p node-template 
./target/debug/node-template --dev -d .ibc-0 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true

# in terminal 2: build and lanch a chain to be recognized as ibc-1 by the relayer
cd substrate1
cargo build -p node-template
./target/debug/node-template --dev -d .ibc-1 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true --port 2033 --ws-port 8844


```
* (Option)explore the chains info and events via polkadot.js:   
    https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer  
    https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A8844#/explorer


* Start mmr root update service
```bash
git clone https://github.com/octopus-network/octopusxt.git
# in terminal 3: start mmr root update serivce
cd octopusxt
cargo test test_update_client_state_service -- --nocapture 

```

* Build the Relayer
```bash
# in terminal 4
git clone  https://github.com/octopus-network/ibc-rs.git
cd ibc-rs
git checkout feature/ics20
cargo build
# check version
./target/debug/hermes -c  config.toml --version
```


* Create channel between two substrate nodes
```bash
# in terminal 4
RUST_BACKTRACE=full  
./target/debug/hermes -c   config.toml create channel ibc-0 ibc-1 --port-a transfer --port-b transfer -o unordered 

RUST_BACKTRACE=full 
./target/debug/hermes -c  -c config.toml start  # Without MMR update service

```

* Transfer fungible tokens
```bash
# in terminal 5
# transfer fungible tokens from ibc-0 to ibc-1
./target/debug/hermes -c config.toml tx raw ft-transfer ibc-1 ibc-0 transfer channel-0 100000000000000000000 -o 9999 -d ATOM

# todo: get denom trace

# transfer fungible tokens from ibc-1 back to ibc-0
./target/debug/hermes -c config.toml tx raw ft-transfer ibc-0 ibc-1 transfer channel-0 100000000000000000000 -o 9999 -d ibc/04C1A8B4EC211C89630916F8424F16DC9611148A5F300C122464CE8E996AABD0
```


### Transfer fungible tokens by UI
* Install UI demo
```bash
# in terminal 6: start UI demo
git clone git@github.com:octopus-network/ibc-frontend-demo.git
cd substrate-front-end-template
# install dependencies
yarn install
# start service
yarn start
```
* Open your browser and visit http://localhost:8000/
* Transfer tokens by ui



### Video Demo
  Pls refer to [Youtube]()
### Issues
  N/A