# WASI Key-Value Store

A proposed [WebAssembly System Interface](https://github.com/WebAssembly/WASI) API.

### Current Phase

Phase 1

### Champions

- Dan Chiarlone
- David Justice
- Jiaxiao Zhou
- Taylor Thomas

### Portability Criteria

`wasi:keyvalue` must have at least two implementations in terms of relevant mainstream key-value stores in each of the following categories:

1. Open-source key-value stores like Redis, Memcached, Etcd etc.
2. Proprietary key-value stores like AWS DynamoDB, Azure CosmosDB, Google Cloud Firestore etc.

The implementations must be able to run on Linux, MacOS, and Windows.

A testsuite that passes on the platforms and implementations mentioned above.

## Table of Contents

- [WASI Key-Value Store](#wasi-key-value-store) - [Current Phase](#current-phase) - [Champions](#champions) - [Portability Criteria](#portability-criteria)
  - [Table of Contents](#table-of-contents)
    - [Introduction](#introduction)
    - [Goals](#goals)
    - [Non-goals](#non-goals)
    - [API walk-through](#api-walk-through)
      - [Use case 1](#use-case-1)
      - [Use case 2](#use-case-2)
    - [Detailed design discussion](#detailed-design-discussion)
      - [Tricky design choice 1](#tricky-design-choice-1)
      - [Tricky design choice 2](#tricky-design-choice-2)
    - [Considered alternatives](#considered-alternatives)
      - [Alternative 1](#alternative-1)
      - [Alternative 2](#alternative-2)
    - [Stakeholder Interest \& Feedback](#stakeholder-interest--feedback)
    - [References \& acknowledgements](#references--acknowledgements)
    - [Change log](#change-log)

### Introduction

This `wasi:keyvalue` proposal defines a collection of [interfaces] for
interacting with key-value stores. It additionally defines a [world],
`wasi:keyvalue/keyvalue`, that groups together common interfaces including

1. CRUD operations on single key-value pairs
2. Atomic operations on single key-value pairs
3. Batch operations on multiple key-value pairs

The API is designed to hide data-plane differences between different key-value stores. The control-plane behavior/functionality (e.g., including cluster management, data consistency, replication, and sharding) are not specified, and are provider specific.

[Interfaces]: https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#wit-interfaces
[World]: https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md#wit-worlds

### Goals

The primary goal of this API is to allow WASI programs to access a wide variety of key-value stores. It meant the `wasi:keyvalue` interfaces to be implementable by a wide variety of key-value stores, including but not limited to: in-memory key-value stores, on-disk key-value stores, document databases, relational databases, and either single-node or distributed key-value stores.

The second goal of `wasi:keyvalue` interfaces is to abstract away the network stack, allowing applications that need to access key-value stores to not worry about what network protocol is used to access the key-value store.

A third design goal of `wasi:keyvalue` interfaces is to allow performance optimizations without compromising the portability of the API. This includes minimizing the number of copies of data, minimizing the number of requests and round-trips, and support for asynchronous operations.

### Non-goals

- The `wasi:keyvalue` interfaces only focus on the most common use cases for key-value stores. It does not attempt to cover all possible use cases.
- The `wasi:keyvalue` interfaces do not guarantee data consistency. Data consistency models (eventual, strong, casual etc.) are diverse and are specific to each key-value store's architecture. The implication is that components importing `wasi:keyvalue` interfaces would need to be aware of the data consistency model of the key-value store they are using.
- The `wasi:keyvalue` interfaces do not handle data replication and sharding. These are control-plane behaviors that are specific to each key-value store's architecture. Replication needs and sharding management are left to the outside of the `wasi:keyvalue` interfaces.
- No cluster management is provided. Operational aspects like scaling, node management, and cluster health are to be managed separately from the wasi:keyvalue interface `wasi:keyvalue` interfaces.
- No built-in monitoring. Monitoring of key-value store performance and health should be done using external tools and not be expected as part of the `wasi:keyvalue` interfaces.

### API walk-through

The proposal can be understood by first reading the comments of `world.wit`, then `eventual.wit`, `atomic.wit`, `eventual-batch.wit`, `caching.wit` and finally `types.wit`

[`world.wit`](./wit/world.wit)
[`eventual.wit`](./wit/eventual.wit)
[`atomic.wit`](./wit/atomic.wit)
[`eventual-batch.wit`](./wit/eventual-batch.wit)
[`caching.wit`](./wit/caching.wit)
[`types.wit`](./wit/types.wit)

### Working with the WIT

Bindings can be generated from the `wit` directory via:

```
wit-bindgen c wit/ --world proxy
```

and can be validated and otherwise manipulated via:

```
wasm-tools component wit wit/ ...
```

The `wit/deps` directory contains a live snapshot of the contents of several
other WASI proposals upon which this proposal depends. It is automatically
updated by running [`wit-deps update`](https://crates.io/crates/wit-deps-cli)
in the root directory, which fetches the live contents of the `main` branch of
each proposal. As things stabilize, `wit/deps.toml` will be updated to refer to
versioned releases.

### References & acknowledgements

This proposal was inspired by Dapr's [State API]

[State API](https://docs.dapr.io/developing-applications/building-blocks/state-management/)

### Change log

- 2024-01-16:
  - Changed the `readwrite` and `batch` interface names to `eventual` and `eventual-batch`. See more details [here](https://github.com/WebAssembly/wasi-keyvalue/pull/30#discussion_r1442282650).
- 2023-12-19:
  - Changed the `size` to `incoming-value-size` and it's signature
- 2023-11-30:
  - Changed the `get` and `get-many` and `keys` signatures
  - Updated comments in all the interfaces.
  - Renamed `wasi-cloud-error` to `wasi-keyvalue-error`
- 2023-05-17: Updated batch example to use one interface instead of 2
- 2023-05-25: Change the WITs to the newest syntax.
- 2023-02-13: The following changes were made to the API:
  - Added `bucket` type to the `types` interface.
  - Uses pseudo-stream and pseudo-resource types inspired from [wasi-io ](https://github.com/WebAssembly/wasi-io)
  - Added \*.wit files for the interfaces and worlds, which are verified by the [wasm-tools](https://github.com/bytecodealliance/wasm-tools)
  - Added a inbound-keyvalue interface that handles the `watch` operation.
- 2023-01-28: The following changes were made to the API:
  - Changed `stream<T>` type to `list<T>` type because `stream<T>` is not supported by the current version of [\*.wit](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md).
  - Removed the worlds section because the star import syntax is not supported by the current version of [\*.wit](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md)
- 2022-11-29: Renamed `bulk-get` to `get-multi` and `bulk-set` to `set-multi` to be consistent with the naming convention of the other interfaces.
- 2022-10-31: Initial draft
