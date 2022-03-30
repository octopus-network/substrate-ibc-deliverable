# Substrate-IBC Milestone 2 Deliverable

This milestone includes a runnable Substrate chain integrated with the following modules in [ibc-rs](https://github.com/informalsystems/ibc-rs),  
* ICS002 Client Semantics
* ICS003 Connection Semantics
* ICS004 Channel and Packet Semantics
* ICS023 Vector Commitments 
* ICS024 Host Requirements

| Number | Deliverable       | Comments                                                |
| ------ | ----------------- | ------------------------------------------------------------ |
| 1.     | Substrate chain  |     [repo](https://github.com/octopus-network/substrate/tree/feature/beefy)   |
| 2.     | Relayer |   [repo](https://github.com/octopus-network/ibc-rs/tree/feature/beefy)   |
| 3.     | Substrate-Ibc Pallet | [repo](https://github.com/octopus-network/substrate-ibc) |
| 4.     | Design Spec       |  [Design](./design.md)                                                            |
| 5.     | Operation Guide, including testing script       |  Right below                                                            |
| 6.     | Video Demo           | [Video](https://www.youtube.com/watch?v=MLdwqpAu_ZA) |

# Operation Guide
* Please head for [this section](#slow-process) to work around the slow process problem.
## Launch 2 IBC Enabled Substrate Chains Locally
```shell script
git clone --branch feature/beefy https://github.com/octopus-network/substrate.git
cd substrate
git submodule update --init
rm bin/node-template/octopus-pallets/Cargo.toml
rm -rf .ibc-*
cargo build -p node-template # generate ./target/debug/node-template

# in terminal 1: lanch a chain to be recognized as ibc-0 by the relayer
./target/debug/node-template --dev -d .ibc-0 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true

# in terminal 2: lanch a chain to be recognized as ibc-1 by the relayer
 ./target/debug/node-template --dev -d .ibc-1 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true --port 2033 --ws-port 8844
```

## Prepare the Relayer and Run E2E Test
### Requirement
* python3.8+
* `pip install toml`

### Compile the Relayer
```bash
git clone --branch ibc-m2-rc1 https://github.com/octopus-network/ibc-rs.git
cd ibc-rs
cargo build
```

### Run e2e Test
```bash
cd e2e-ibc-m2
python run.py -c ../config.toml --cmd ../target/debug/hermes # Run automatic e2e testing
view README.md  # More details of the testing 
```

After the packet transfer completes, events below will be detected by polkadot.js in sequence and displayed on the frontend(e.g., https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer). Whereas `SendPacket` & `AcknowledgementPacket` events are emitted from the chain initiated the packet transfer, and `WriteAcknowledgement` & `ReceivePacket` are from the other chain. It takes over 20 mins.

![SendPacket](assets/SendPacket.png)

![RecvPacket](assets/RecvPacket.png)

![AckPacket](assets/AckPacket.png)

## Commands in [Video Demo](https://www.youtube.com/watch?v=MLdwqpAu_ZA)
```bash
# in terminal 1: lanch a chain to be recognized as ibc-0 by the relayer
./target/debug/node-template --dev -d .ibc-0 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true

# in terminal 2: lanch a chain to be recognized as ibc-1 by the relayer
 ./target/debug/node-template --dev -d .ibc-1 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true --port 2033 --ws-port 8844

# in terminal 3: establish IBC clients, connections, and channels
 RUST_BACKTRACE=full  ./target/debug/hermes -c  config.toml create channel ibc-0 ibc-1 --port-a transfer --port-b transfer -o unordered

# in terminal 4: start a relayer
RUST_BACKTRACE=full ./target/debug/hermes -c config.toml start

# in terminal 3: trigger packet transfer
./target/debug/hermes -c config.toml tx raw ft-transfer ibc-1 ibc-0 transfer channel-0 9999 -o 9999 -n 1 -t 9999
```


# Pending Issues
## Relevant Issues in Github
* [Client update based on Beefy protocol](https://github.com/informalsystems/ibc-rs/issues/1775)
* [new variant needed in the enum of commitment_proof](https://github.com/informalsystems/ibc-rs/issues/1945)
* [height.increment() not needed for Beefy](https://github.com/informalsystems/ibc-rs/issues/1845)
* [Allow Timeout UNORDERED channel without proof of absence](https://github.com/cosmos/ibc/issues/620): Some chains do not have the ability to provide proof of absence, like Substrate based chains. Therefore, the proof of [timeout](Some chains do not have the ability to provide proof of absence) is pending in code [here](https://github.com/octopus-network/ibc-rs/blob/6e5f6c196dad0acde4aafb379b39bd01ba5a0724/relayer/src/chain/substrate.rs#L1518) and [here](https://github.com/octopus-network/ibc-rs/blob/6e5f6c196dad0acde4aafb379b39bd01ba5a0724/relayer/src/chain/substrate.rs#L1521).
* [commitment_proof for non-Cosmos chain](https://github.com/confio/ics23/issues/80)

## Other Issues
### Slow Process
* Issue: By tag `ibc-m2-rc1` of `ibc-rs`, it takes about 15 min to establish IBC client, connection, and channel to link the 2 chains, and about 10 min to complete a packet transfer.
* Cause: The integration of update MMR root function causes the slowness; [current integration](https://github.com/octopus-network/ibc-rs/blob/330b1a554c3223b07121ca83af5eccffc3f56a2b/relayer/src/foreign_client.rs#L917) results in [relayer waiting a long time for MMR root to be updated](https://github.com/octopus-network/ibc-rs/blob/330b1a554c3223b07121ca83af5eccffc3f56a2b/relayer/src/foreign_client.rs#L927). The complete fix depends on the Github issue [Client update based on Beefy protocol](https://github.com/informalsystems/ibc-rs/issues/1775).
* Workaround: Running the MMR update service as an individual process away from the relayer reduces the duration significantly. About 6 min to establish paths to link to the 2 chains; 2 min to complete a packet transfer. You may refer to the [video demo of the workaround](https://www.youtube.com/watch?v=yDLtsGGU9Mw) and the commands in the video below.
```shell script
git clone --branch feature/beefy https://github.com/octopus-network/substrate.git
cd substrate
git submodule update --init
rm bin/node-template/octopus-pallets/Cargo.toml
rm -rf .ibc-*
cargo build -p node-template # generate ./target/debug/node-template

# in terminal 1: lanch a chain to be recognized as ibc-0 by the relayer
./target/debug/node-template --dev -d .ibc-0 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true

# in terminal 2: lanch a chain to be recognized as ibc-1 by the relayer
./target/debug/node-template --dev -d .ibc-1 --rpc-methods=unsafe --ws-external --enable-offchain-indexing true --port 2033 --ws-port 8844

# in terminal 3: establish IBC clients, connections, and channels
git clone --branch ibc-m2-rc2 https://github.com/octopus-network/ibc-rs.git # Without MMR update service
cd ibc-rs
cargo build # generate ./target/debug/hermes
RUST_BACKTRACE=full  ./target/debug/hermes -c  config.toml create channel ibc-0 ibc-1 --port-a transfer --port-b transfer -o unordered 

# in terminal 4: start mmr root update service
sleep 10
git clone https://github.com/octopus-network/octopusxt.git
cd octopusxt
sleep 10;cargo test test_update_client_state_service -- --nocapture 

# in terminal 5: start a relayer
RUST_BACKTRACE=full ./target/debug/hermes -c config.toml start  # Without MMR update service

# in terminal 3: trigger packet transfer
sleep 20;./target/debug/hermes -c config.toml tx raw ft-transfer ibc-1 ibc-0 transfer channel-0 9999 -o 9999 -n 1 -t 9999
```
