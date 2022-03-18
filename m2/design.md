# Design of Substrate-IBC Milestone 2

The document doesn't cover the overall design of Substrate-IBC(which is [here](https://github.com/octopus-network/substrate-ibc/tree/7b36d984b5393492e51f35cf3c1be95be074ce52#design-overview)), but some key modifications in this milestone.

## Substrate Chain Support in Relayer
### Add Chain Type and Consensus
* Add a client type: Grandpa
```rust
    pub fn prefix(client_type: ClientType) -> &'static str {
        match client_type {
            ClientType::Tendermint => ClientType::Tendermint.as_str(),
            ClientType::Grandpa => ClientType::Grandpa.as_str(),
// --snip--
        }
    }
```

* Spawn chain runtime for Substrate type chain
```rust
    let handle = match account_prefix.as_str()  {
        "cosmos" => {
// --snip--
        },
        "substrate" => {
            let rt = Arc::new(TokioRuntime::new().unwrap());
            let handle = ChainRuntime::<SubstrateChain>::spawn(chain_config, rt).map_err(SpawnError::relayer)?;
            handle
        }
    };
```

* Add Grandpa consensus type
```rust
pub enum AnyConsensusState {
    Tendermint(consensus_state::ConsensusState),
    Grandpa(ics10_grandpa::consensus_state::ConsensusState),
// --snip--
}
```
### Invoking RPC to Communicate with Substrate Chain 
RPC communication with Substrate chains is packed in crate [Octopusxt](https://github.com/octopus-network/octopusxt), which is a wrapper of [Subxt](https://github.com/paritytech/subxt).

* Implement monitor for events from Substrate chain
```rust
    fn init_event_monitor(
        &self,
        rt: Arc<TokioRuntime>,
    ) -> Result<(EventReceiver, TxMonitorCmd), Error> {
        tracing::info!("in Substrate: [init_event_mointor]");

        tracing::info!(
            "In Substrate: [init_event_mointor] >> websocket addr: {:?}",
            self.config.websocket_addr.clone()
        );
// --snip--
        thread::spawn(move || event_monitor.run());
    }
```
* Implement transaction interface to submit transactions to Substrate chains
```rust
    fn send_messages_and_wait_commit(
        &mut self,
        proto_msgs: Vec<Any>,
    ) -> Result<Vec<IbcEvent>, Error> {
// --snip--
        let client = async {
            let client = ClientBuilder::new()
                .set_url(&self.websocket_url.clone())
                .build::<ibc_node::DefaultConfig>().await.unwrap();
            sleep(Duration::from_secs(4)).await;
            let result = self.deliever(proto_msgs, client).await.unwrap();
            result
        };
        let _ = self.block_on(client);
// --snip--
    }
```
* Implement client query of Substrate chains
```rust
    fn query_client_state(
        &self,
        client_id: &ClientId,
        height: ICSHeight,
    ) -> Result<Self::ClientState, Error> {
// --snip--
        let client_state = async {
            let client = ClientBuilder::new()
                .set_url(&self.websocket_url.clone())
                .build::<ibc_node::DefaultConfig>().await.unwrap();
// --snip--
    }
```
* Implement connection query of Substrate chains
```rust
    fn query_connection(
        &self,
        connection_id: &ConnectionId,
        height: ICSHeight,
    ) -> Result<ConnectionEnd, Error> {
// --snip--
        let connection_end = async {
            let client = ClientBuilder::new()
                .set_url(&self.websocket_url.clone())
                .build::<ibc_node::DefaultConfig>().await.unwrap();
// --snip--
    }
```
* Implement the channel query of Substrate chains
```rust
    fn query_channel(
        &self,
        port_id: &PortId,
        channel_id: &ChannelId,
        height: ICSHeight,
    ) -> Result<ChannelEnd, Error> {
// --snip--
        let channel_end = async {
            let client = ClientBuilder::new()
                .set_url(&self.websocket_url.clone())
                .build::<ibc_node::DefaultConfig>().await.unwrap();
// --snip--
    }
```
* Implement the packet query of Substrate chains
```rust
    fn query_packet_commitments(
        &self,
        request: QueryPacketCommitmentsRequest,
    ) -> Result<(Vec<PacketState>, ICSHeight), Error> {
// --snip--
        let packet_commitments = async {
            let client = ClientBuilder::new()
                .set_url(&self.websocket_url.clone())
                .build::<ibc_node::DefaultConfig>().await.unwrap();
// --snip--
    }
```

## Beefy Integration
[Beefy](https://github.com/paritytech/grandpa-bridge-gadget/blob/8e7d82917e988d7e54ff2ffecdf8461bb9f65c53/docs/beefy.md) is integrated in this milestone. [ClientState](https://github.com/octopus-network/ibc-rs/blob/4c0a6919c284811d6e435bff2d249891a91e40fd/modules/src/clients/ics10_grandpa/client_state.rs#L24) is updated to include MMR root.

```rust
#[derive(Clone, Debug, PartialEq, Eq, Serialize, Deserialize)]
pub struct ClientState {
    pub chain_id: ChainId,
    /// block_number is height?
    pub block_number: u32,
    /// Block height when the client was frozen due to a misbehaviour
    pub frozen_height: Option<Height>,
    pub block_header: BlockHeader,
    pub latest_commitment: Commitment,  // <== MMR root is included
    pub validator_set: ValidatorSet,
}
```

When implementing Beefy protocol based on Grandpa consensus, there are 3 kinds of verifications in a hierarchy.
* The top tier verification is MMR root verification, by collecting sufficient validators' valid signatures. 
```rust
// in relayer
Ok(ChainRequest::UpdateMmrRoot { src_chain_websocket_url, dst_chain_websocket_url, reply_to }) => {
    self.update_mmr_root(src_chain_websocket_url,dst_chain_websocket_url,reply_to,)?
},
```

```rust
// in substrate-ibc
pub fn update_client_state(
    origin: OriginFor<T>,
    client_id: Vec<u8>,
    mmr_root: Vec<u8>,
) -> DispatchResultWithPostInfo {
    // --snip--
}
```

* Based on a valid MMR root, the 2nd tier is [block header verification](https://github.com/informalsystems/ibc-rs/issues/1775#issuecomment-1016043500).
```rust
// prepare header proof in relayer
fn build_header(
    &self,
    trusted_height: ICSHeight,
    target_height: ICSHeight,
    client_state: &AnyClientState,
    light_client: &mut Self::LightClient,
) -> Result<(Self::Header, Vec<Self::Header>), Error> {
    // --snip--
}
```

```rust
// verify header in ibc-rs/modules
fn check_header_and_update_state(
    &self,
    ctx: &dyn ClientReader,
    client_id: ClientId,
    client_state: Self::ClientState,
    header: Self::Header,
) -> Result<(Self::ClientState, Self::ConsensusState), Error> {
    // --snip--
}
```

* Based on a valid block header,  the 3rd tier is [on-chain storage verification](https://github.com/octopus-network/substrate/blob/33c518ebbe43d38228ac47e793e4d1c76738a56d/primitives/state-machine/src/lib.rs?_pjax=%23js-repo-pjax-container%2C%20div%5Bitemtype%3D%22http%3A%2F%2Fschema.org%2FSoftwareSourceCode%22%5D%20main%2C%20%5Bdata-pjax-container%5D#L1078), which calculates the correct on-chain storage by the state root, the storage proof, and the storage key. 
```rust
// prepare on-chain storage proof in relayer
fn generate_storage_proof<F: StorageEntry>(
    &self,
    storage_entry: &F,
    height: &Height,
    storage_name: &str
) -> MerkleProof
where
    <F as StorageEntry>::Value: serde::Serialize + core::fmt::Debug,
{
    // --snip--
}
```

```rust
// verify on-chain storage in ibc-rs/modules

// Reconstruct on-chain storage value by proof, key(path), and state root
let on_chain_storage = fn get_storage_via_proof(
    _client_state: &ClientState,
    _height: Height,
    _proof: &CommitmentProofBytes,
    _keys: Vec<Vec<u8>>,
    _storage_name: &str,
) -> Result<Vec<u8>, Error> {
// --snip--
}

// compare on_chain_storage with the expected on-chain storage value
    // --snip--
```
