# Substrate-IBC Contract 2 Milestone 2 Deliverable

## This milestone includes the following goals:
1. Updates to Chinese translation of IBC spec (pending finalization of IBC 1.0 to be released in conjunction with
Cosmos Hub Stargate upgrade)
2. Tutorial and workshop introducing IBC and IBC on Substrate
a.Host workshop online (recorded and made available for Cosmos Code With Us at the discretion of All in Bits)
b. Written tutorial version of the workshop, available in English and Chinese
3. Live workshops about Substrate IBC module (pallet-ibc)
4. Testnet bug fixes
5. PR to upstream Substrate or releasing a customized Substrate, depending on
upstream


## Related artifacts

| Number | Deliverable       | Comments                                                |
| ------ | ----------------- | ------------------------------------------------------------ |
| 1.     | Substrate chain(ibc-0) |     [repo](https://github.com/octopus-network/substrate/tree/feature/ics20-ibc-0)   |
| 2.     | Substrate chain(ibc-1) |     [repo](https://github.com/octopus-network/substrate/tree/feature/ics20-ibc-1)   |
| 3.     | Ibc rs |   [repo](https://github.com/octopus-network/ibc-rs/tree/feature/ics20)   |
| 4.     | Substrate-Ibc Pallet | [repo](https://github.com/octopus-network/substrate-ibc/tree/feature/ics20) |
| 5.     | Chinese Translation of IBC Spec       |  [CN-Spec](https://github.com/octopus-network/ibc-spec-cn)                                                            |
| 6.     | Workshop introducing IBC and IBC on Substrate    |  [Link](https://drive.google.com/drive/u/2/folders/1jBUafbKlWvbPpzxnuCZadYr6i9uEcrHD)                                                            |
| 7.     | Written tutorial of the workshop           | [CN](https://docs.google.com/presentation/d/1xiSZJJ0pVONVSJi1v4yaso3leYCk6tYw2MqRvif4ZwM/edit?usp=sharing), [EN](https://docs.google.com/presentation/d/1ZOjHToR0EFJrng-hvR5tlFP2YUPVJfNmNHhRAqGIoJQ/edit?usp=sharing) |
| 8.     | Live workshops about Substrate IBC module |   [LiveDemo](https://drive.google.com/drive/u/2/folders/1rFUOIyW1HMWYf5f_J8E7uJ_RA9Srbo2I)   |
| 9.     | Tutorial of live workshop | [CN](./guide-cn.md), [EN](./guide.md) |
| 10.    | Testnet Bug Fix | [Testnet Bug Fix](#testnet-bug-fix) |

## Testnet Bug Fix

Bugs reported:
* It takes a long time to create a channel. (Took 6.5 minutes on my laptop)

    Answer: Furthermore, in the BEEFY protocol set in the deliverable, the MMR root is updated every 8 blocks. So the cross-chain event needs at least 8 blocks' time to be verified. While Cosmos(tendermint) needs only 1~2 block time for cross-chain event verification.

* Create channel doesn’t work if the MMR update service isn’t running (although this might be the intended behaviour).

    Answer: It's an open issue logged [here](https://github.com/informalsystems/ibc-rs/issues/1775), which is up to the architecture decision from ibc-rs. Individual MMR root update service is a temporary solution.

* Hermes seems to query the chains very often, more than once per second. (query_client_state)

    Answer: The querying frequency is configured here, It's set to query every 200 ms until the substrate chain replies with a positive value , at most querying 100 times. Because after the relayer submits a tx to the substrate chain, it will take a few seconds for the tx to be accepted, if no error. And the time a substrate chain accepts a tx is variant.

    In the Cosmos handling logic in the relayer,  the querying interval is [300 ms](https://github.com/informalsystems/ibc-rs/blob/d9c6f897097315a6b67df063ec682f13b4cffbba/relayer/src/chain/cosmos/wait.rs#L16). We set the same interval for Substrate. 

