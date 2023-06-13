# ICF M3 deliverable

## Related Repos

| Repo Name | Branch | Note |
| --- | --- | --- |
| Hermes | https://github.com/octopus-network/hermes/tree/icf-m3 | relayer |
|  |  |  |
| substrate-ibc | https://github.com/octopus-network/substrate-ibc/tree/icf-m3 | based on ibc-rs 0.28 |
| polkadot | https://github.com/octopus-network/polkadot/tree/icf-m3 | demo chain |
|  |  |  |
| ibc-go | https://github.com/octopus-network/ibc-go/tree/icf-m3 | include ics10 grandpa light client |
| oct-planet | https://github.com/octopus-network/oct-planet/tree/icf-m3 | demo chain |
|  |  |  |

### pre-requisites

rust 1.69 +

go 1.19+

python 3.10+

[zombienet latest](https://github.com/paritytech/zombienet/releases/)

[ignite latest](https://ignite.com/)

## spin up substrate/polkadot

```bash
# Clone polkadot
git clone https://github.com/octopus-network/polkadot
cd polkadot
git checkout icf-m3

# build polkadot 
cargo build --release --bin polkadot
# optional put polkadot cli into your BIN PATH

# spin up polkadot nodes
zombienet spawn zombienet_tests/ibc/rococo.toml

```

## spin up cosmos chain

```bash
# clone Octopus ignite chain: oct-planet
git clone https://github.com/octopus-network/oct-planet.git

git checkout icf-m3

# launch a cosmos chain: earth 
ignite chain serve -f -v -c earth.yml

```

## compile and config hermes

```bash
$ git clone https://github.com/octopus-network/hermes.git
$ cd hermes
$ git checkout icf-m3

# build
$ cargo build -p ibc-relayer-cli
# check hermes version
$ ./target/debug/hermes version

# optional,export PATH
# export PATH="$PWD/./target/release/:$PATH"

# add key
hermes --config config/cos_sub.toml keys add --chain earth-0 --key-file config/alice_cosmos_key.json --key-name alice
hermes --config config/cos_sub.toml keys add --chain rococo-0 --key-file config/bob_substrate_key.json --key-name Bob

```

## Test

```bash

# create channel
cd hermes
hermes --config config/cos_sub.toml create channel --a-chain earth-0 --b-chain rococo-0 --a-port transfer --b-port transfer --new-client-connection --yes

# start hermes service 
hermes --config config/cos_sub.toml start

# create cross asset on substrate
# first install substrate-interface
pip install substrate-interface
./scripts/sub-cli --sudo //Alice tx --signer //Alice --module Assets --method force_create --params '{"id":666,"owner":"5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty","is_sufficient":true,"min_balance": 10}'

# transfer from earth to rococo
hermes --config config/cos_sub.toml tx ft-transfer --timeout-height-offset 1000 --number-msgs 1 --dst-chain rococo-0 --src-chain earth-0 --src-port transfer --src-channel channel-0 --amount 999000 --denom ERT

# wait for auto relay by hermes,about 30s

# query cosmos account(Alice) change 
earth --node tcp://localhost:26657 query bank balances $(earth --home .earth keys --keyring-backend="test" show alice -a)
# query substrate account(Bob) change
./scripts/sub-cli query-balances --account 5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty

# transfer back to earth from rococo
hermes --config config/cos_sub.toml tx ft-transfer --timeout-height-offset 1000 --denom ibc/972368C2A53AAD83A3718FD4A43522394D4B5A905D79296BF04EE80565B595DF  --dst-chain earth-0 --src-chain rococo-0 --src-port transfer --src-channel channel-0 --amount 999000

# query cosmos account(Alice) change 
earth --node tcp://localhost:26657 query bank balances $(earth --home .earth keys --keyring-backend="test" show alice -a)
# query substrate account(Bob) change
./scripts/sub-cli query-balances --account 5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty

```

Optional : browser info via polkadotjs

N/A