# Based Sequencing Preconfirmation research

## High-level design
- [Ethereum sequencing](https://notes.ethereum.org/WLuNFaliQiqw7Zhd-7AnmQ)

This design mentions no need hard fork. There is a video [here](https://www.youtube.com/watch?v=eycLQCaqDsk) that explains the design.

## Details of the design resource

- [Vanilla Based Rollups](https://hackmd.io/@Perseverance/SyYA47CZ0?utm_source=preview-mode&utm_medium=rec) and [GitHub](https://github.com/LimeChain/based-preconfirmations-research)

- [L1 Preconf API](https://right-knife-e11.notion.site/Aligning-Preconfirmation-APIs-db7907d9e66e41718e6bc2cff19604e4#21cd6f7f864d417b9d9727bd8c29fc6e)

## Features
[POC Scope](https://github.com/LimeChain/based-preconfirmations-research/blob/main/docs/suggested-poc-scope.md)

1. [Sequencer Opt-In, Discovery and Communication](https://github.com/LimeChain/based-preconfirmations-research/blob/main/docs/optin-mechanics.md) [taiko sequencer registry](https://github.com/taikoxyz/taiko-mono/blob/preconfirmations/packages/protocol/contracts/L1/ISequencerRegistry.sol)
1. [Preconfirmation Transaction](https://github.com/LimeChain/based-preconfirmations-research/blob/main/docs/preconfirmations-for-vanilla-based-rollups.md)
   1. preconf api design differences? [taiko preconf api](https://github.com/taikoxyz/taiko-mono/tree/preconfirmations/packages/taiko-client/preconfapi)
1. [Pipeline](https://github.com/LimeChain/based-preconfirmations-research/blob/main/docs/pipelines.md)


## Issues
- [taiko mev](https://ethresear.ch/t/based-rollups-can-reward-proposers-first-come-first-serve/18317)

## Tasks

### Preconfirmation Infrastructures
- [x] [bolt devnet](https://github.com/chainbound/bolt/issues/218)
- [x] [bolt demo flow](./bolt.md)
- [ ] bolt opt-in/challenge

### Preconfirmation Users
- [ ] taiko nethermind demo