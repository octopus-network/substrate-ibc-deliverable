# Substrate-IBC Milestone 2 Deliverable

This milestone includes a runnable Substrate chain integrated with the following modules in [ibc-rs](https://github.com/informalsystems/ibc-rs),  
* ICS002 Client Semantics
* ICS003 Connection Semantics
* ICS004 Channel and Packet Semantics
* ICS023 Vector Commitments 
* ICS024 Host Requirements

| Number | Deliverable       | Commnets                                                |
| ------ | ----------------- | ------------------------------------------------------------ |
| 1.     | Substrate chain  |     ???   |
| 2.     | Relayer |   ???   |
| 3.     | Vedio Demo           | ??? |
| 4.     | Design Spec       |  [design](./design.md)                                                            |
| 5.     | Operation Guide, including testing script       |  Right below                                                            |

# Operation Guide
## Launch 2 IBC Enabled Substrate Chains Locally
```bash
git clone ??? 
cd ???
rm -rf .ibc-*
cargo build -p node-template

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
git clone ???
cd ibc-rs
cargo build
```

### Run e2e Test
```bash
cd e2e-ibc-m2
python run.py -c ../config.toml --cmd ../target/debug/hermes # Run automatic e2e testing
view README.md  # More details of the testing 
```

Wait until you see events of `WriteAcknowledgement` & `ReceivePacket` on the explorer of parachain like below. It takes over 20 mins.

???

# Pending Issues
## Relevant Issues in Github
* [Client update based on Beefy protocol](https://github.com/informalsystems/ibc-rs/issues/1775)
* [new variant needed in the enum of commitment_proof](https://github.com/informalsystems/ibc-rs/issues/1945)
* [height.increment() not needed for Beefy](https://github.com/informalsystems/ibc-rs/issues/1845)
* [Allow Timeout UNORDERED channel without proof of absence](https://github.com/cosmos/ibc/issues/620): Some chains do not have ability to provide proof of absence, like Substrate based chains. Therefore, the proof of [timeout](Some chains do not have ability to provide proof of absence) is pending in code [here](https://github.com/octopus-network/ibc-rs/blob/6e5f6c196dad0acde4aafb379b39bd01ba5a0724/relayer/src/chain/substrate.rs#L1518) and [here](https://github.com/octopus-network/ibc-rs/blob/6e5f6c196dad0acde4aafb379b39bd01ba5a0724/relayer/src/chain/substrate.rs#L1521).

## Other Issues
* Slow process: It takes about 15 min to establish IBC client, connection, and channel to link the 2 chains, and about 10 min to complete a packet transfer.