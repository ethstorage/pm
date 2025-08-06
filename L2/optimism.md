# Optimism Deep Dive

The new implementation is event-driven which causes the code not readable. This document is to help understand the
codebase and the flow of the code.

[TOC]

## Rollup Node

### Driver

The `Driver` is the entry point of derivation. The core function is `eventLoop` which is a loop to handle channels, the
external inputs will pass through the channels.
Once any channel is ready, the `eventLoop` will call the corresponding function to handle the input.

The inputs are actually L1 blocks and L2 unsafe blocks.

- `OnNewL1Head` is to subscribing the latest L1 header.
- `OnNewL1Safe` is to polling the latest L1 safe block.
- `OnNewL1Finalized` is to polling the latest L1 finalized block.
- `OnUnsafeL2Payload` is to handle the unsafe L2 block.

`Driver` use channels to communicate with other components. However, the internal components are communicating with
events. There are several internal drivers to handle different functionalities.

### Derivation

The derivation is the core driver to handle the L1 blocks to derive the L2 blocks.
It is a pipeline which consists of several stages. Each stage is a function to handle the L1 block and derive the L2
block.

#### How to trigger the pipeline

When `PipelineDeriver` receives the `PipelineStepEvent`, it will call the `Step` function to trigger the pipeline.
But how the `PipelineStepEvent` is emitted? We go back to the `eventLoop` function of the `Driver` to find the answer.

- There is a `reqStep` function to request a derivation step. It will first emit the `StepReqEvent`, through the
  `StepSchedulingDeriver` to transform the request to the `StepAttemptEvent`, and eventually to the `StepEvent`.
  The `StepSchedulingDeriver` will schedule the step request based on the backoff strategy.
- The `StepEvent` will be handled by the `SyncDeriver` which responsible for the synchronization of the `opNode` and
  `execution engine` status which are header, safe, finalized block.
  It will call `SyncStep` function, first drain the remaining events, then synchronization with `execution engine`. If
  `execution engine` is ready, it will emit the `PendingSafeRequestEvent` which notify `AttributesHandler` to handle.
  Otherwise, it will trigger `ResetStepBackoffEvent` and some error events will be triggered.
  `SyncDeriver` will handle the error events and trigger the `StepReqEvent` again.
- The `EngDeriver` will transform the `PendingSafeRequestEvent` to the `PendingSafeUpdateEvent`, `AttributesHandler`
  will handle the `PendingSafeUpdateEvent` and when no attributes are handling, it will emit `PipelineStepEvent`.

#### Stages

##### `L1Traversal`
TODO:


### Recovery

#### Sequencer Outages
https://docs.optimism.io/stack/protocol/outages

[dc discussion](https://discord.com/channels/1244729134312198194/1277998107287748649)

#### Key point

- There is **no depositTx** in batch, `op-batcher` will filter deposit txs before submitting to L1
  - [code](https://github.com/ethereum-optimism/optimism/blob/develop/op-node/rollup/derive/channel_out.go#L233)

- Both verifier and sequencer get deposit txs from L1 contract.
  - [code](https://github.com/ethereum-optimism/optimism/blob/develop/op-node/rollup/derive/attributes.go#L46)

- If sequencer downtime is enough long, the verifier will force produce empty batch, and add deposit txs to generate L2 block
  - [code](https://github.com/ethereum-optimism/optimism/blob/develop/op-node/rollup/derive/batch_queue.go#L303)

- When sequencer is backup from a long downtime(verifier already create empty batch), it will also catch up verifier first(create empty batch), after that, it can sequence new batch again. If it create new batch before catch up verifier, the verifier will drop these batches.
  - [code](https://github.com/ethereum-optimism/optimism/blob/develop/op-node/rollup/sequencing/sequencer.go#L516)

  - [code](https://github.com/ethereum-optimism/optimism/blob/develop/op-node/rollup/derive/batches.go#L128)