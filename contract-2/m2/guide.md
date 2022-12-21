# Operation Guide

* Note: Browser refresh may be needed if you fail to transact on browser.

## Requirements

- Install [polkadot wallet extension](https://polkadot.js.org/extension/)
- Create test accounts: David and Davirain
  - create accounts
  ```bash
    gaiad keys add david --output json > david_seed.json
    gaiad keys add davidrain --output json > davidrain_seed.json
  ```

  - The mnemonic used in this guide :
    - David   
    >ecology agent adjust admit raw castle rather travel asthma good say field away vote timber miss demand mandate rib print sport vault warfare thrive
    - Davirian   
    >lawn lunch neutral task waste pioneer grape setup stumble copy squirrel wasp language speed slam output blur three squeeze network awful spoon monkey spare  
- Import the accounts to Polkadot.js
  ![image](assets/import_step1.jpeg)  
  ![image](assets/import_step2.jpeg)  
  ![image](assets/import_step3.jpeg)  
  ![image](assets/import_step4.jpeg)  
  
- install rust
```bash
rustup default stable
rustup update
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```
  
## Launch 2 IBC Enabled Substrate Chains Locally

```bash
# in terminal 1: build and lanch a chain to be recognized as ibc-0 by the relayer
git clone -b feature/m4v4-hermes1.0 git@github.com:octopus-network/substrate.git ibc-0
cd ibc-0
git submodule update --init
git pull
rm bin/node-template/octopus-pallets/Cargo.toml
rm -rf .ibc-*
cargo build -p node-template 
./target/debug/node-template --dev -d .ibc-0 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true


# in terminal 2: build and lanch a chain to be recognized as ibc-1 by the relayer
git clone -b feature/m4v4-hermes1.0-ibc1 git@github.com:octopus-network/substrate.git  ibc-1
cd ibc-1
git submodule update --init
git pull
rm bin/node-template/octopus-pallets/Cargo.toml
rm -rf .ibc-*
cargo build -p node-template
./target/debug/node-template --dev -d .ibc-1 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true --port 2033 --ws-port 8844

```
* (Option)explore the chains info and events via polkadot.js:   
    https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer  
    https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A8844#/explorer


## Create channel between two substrate nodes
* Build the Relayer
```bash
# in terminal 4
git clone --branch feature/m4v4-hermes1.0 git@github.com:octopus-network/hermes.git ibc-rs
cd ibc-rs
cargo build
```
* Add keys to the Relayer
```bash
# in terminal 4
./target/debug/hermes --config  config.toml keys add -n david -f david_seed.json ibc-0
./target/debug/hermes --config  config.toml keys add -n davirain -f davirain_seed.json ibc-1

```
* Create channel 
```bash
# in terminal 4
./target/debug/hermes --config config.toml create channel --a-chain ibc-0 --b-chain ibc-1 --a-port transfer --b-port transfer --order unordered --new-client-connection 

# After the channel is created for both parties
# start the relayer
./target/debug/hermes --config config.toml start
```

## Top-up for test account
- Deposit a certain amount of assets to the David account on ibc-0
  1. access ibc-0 extrinsics module via polkadot.js  
  > https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/extrinsics
  2. transfer 5000000000000000000 to David from Alice
  ![image](assets/d2d.jpeg)
  3. query the balance of David on ibc-0  
  > https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/accounts
  ![image](assets/d_account.jpeg)

- Deposit a certain amount of assets to the Davirian account on ibc-1
  1. access ibc-1 extrinsics module via polkadot.js  
  > https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A8844#/extrinsics
  2. transfer 5000000000000000000 to Davirian from Alice
  ![image](assets/d2dr.jpeg)
  3. query the balance of Davirian on ibc-1
  > https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A8844#/accounts 

  ![image](assets/dr_account.jpeg)

## Create cross-chain assets 
>In order to identify assets for cross-chain transfers, we need to create each other's asset classes on the target chain separately
> Current,ibc-0 native token is OCT and ibc-1 native token is ATOM.

- Create assets ATOM on ibc-0 chain for ibc-1 chain  
  1. access ibc-0 sudo module via polkadot.js  
  > https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/sudo   
  2. create asset class
![image](assets/ibc-0-cf.jpeg)
  
- Create assets OCT on ibc-1 chain for ibc-0 chain
  1. access ibc-0 sudo module via polkadot.js  
  > https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A8844#/sudo 
  2. create asset class
![image](assets/ibc-1-cf.jpeg)

## Transfer fungible tokens by CLI
- Transfer OCT from ibc-0(david) to ibc-1(davirain)
```bash
# in terminal 5
 ./target/debug/hermes --config config.toml tx ft-transfer --receiver-chain ibc-1 --sender-chain ibc-0 --sender-port transfer --sender-channel channel-0 --amount 1000000000000000000 --denom OCT
```
- query the transfer events via polkadot.js  
> https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer 
![image](assets/ibc-0-se.jpeg)  

> https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A8844#/explorer   
![image](assets/ibc-1-re.png) 
- query the transfer result via polkadot.js
> https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A8844#/chainstate 

![image](assets/ibc-1-result.jpeg) 


- Transfer OCT back to ibc-0(david) from ibc-1(davirain)
```bash
# in terminal 5
./target/debug/hermes --config config.toml tx ft-transfer --receiver-chain ibc-0 --sender-chain ibc-1 --sender-port transfer --sender-channel channel-0 --amount 1000000000000000000 --denom transfer/channel-0/OCT
```
- query the transfer events and result via polkadot.js
![image](assets/ibc-1-back.jpeg)
