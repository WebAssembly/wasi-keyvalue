# WASI Key-Value Store

A proposed [WebAssembly System Interface](https://github.com/WebAssembly/WASI) API.

### Current Phase

Phase 1

### Champions

- Dan Chiarlone
- David Justice
- Jiaxiao Zhou

### Phase 4 Advancement Criteria

At least two independent production implementations.

At least two cloud provider implementations.

Implementations available for at least Windows, Linux & MacOS.

A testsuite that passes on the platforms and implementations mentioned above.

## Table of Contents

- [WASI Key-Value Store](#wasi-key-value-store)
    - [Current Phase](#current-phase)
    - [Champions](#champions)
    - [Phase 4 Advancement Criteria](#phase-4-advancement-criteria)
  - [Table of Contents](#table-of-contents)
    - [Introduction](#introduction)
    - [Goals](#goals)
    - [Non-goals](#non-goals)
    - [API walk-through](#api-walk-through)
      - [Use case 1](#use-case-1)
    - [Detailed design discussion](#detailed-design-discussion)
      - [Tricky design choice 1](#tricky-design-choice-1)
      - [Tricky design choice 2](#tricky-design-choice-2)
    - [Considered alternatives](#considered-alternatives)
      - [Alternative 1](#alternative-1)
      - [Alternative 2](#alternative-2)
    - [Stakeholder Interest & Feedback](#stakeholder-interest--feedback)
    - [References & acknowledgements](#references--acknowledgements)

### Introduction

WASI Key-Value Store is a WASI API primarily for accessing a key-value datastore. It has functions to:

1. retrieve the value stored and associated with a key,
2. delete a key-value pair, and
3. create and update a key-value pair.

The API is designed to hide data-plane differences between different key-value stores. The control-plane behavior/functionality (e.g., including cluster management, data consistency, replication, and sharding) are not specified, and are provider specific.

### Goals

The primary goal of this API is to allow users to use WASI programs to access key-value stores. It abstracts away the complexity of building stateful services that need to access key-value stores. In other words, it decouples the application code from the specific key-value store APIs. This allows WASI programs to be portable across different key-value store that supports this interface, be it on-prem, in the cloud, or in edge devices.

The second goal of this API is to abstract away the network stack, allowing applications that need to access key-value stores to not worry about what network protocol is used to access the key-value store.

### Non-goals

- Cover all application requirements for a key-value store. Instead, it focuses on the most common use cases. It allows providers to extend the API to support more use cases.
- Transactional semantics.
- Data consistency.
- Data replication.
- Data sharding.
- Cluster management.
- Monitoring

### API walk-through

#### Use case 1

Imagine you have an HTTP handler that needs to persist some data to a key-value store. The handler needs to be able to retrieve, delete, and update the data in the key-value store. The following Rust code shows how you can use the WASI Key-Value Store API to in the handler.

```rust
let value = b"value1";
kv1::set("key", value)?;
let val = kv1::get("key")?;
assert_eq!(val, value);

kv1::delete("key")?;
```

### Detailed design discussion

```go
interface "wasi:kv/data/readwrite" {
  use { Error, payload, key } from "wasi:kv/types"

  get: func(key: key) -> result<payload, Error>

  set: func(key: key, value: stream<u8>) -> result<_, Error>

  delete: func(key: key) -> result<_, Error>

  exists: func(key: key) -> result<bool, Error>
}

interface "wasi:kv/data/atomic" {
  use { Error, key } from "wasi:kv/types"
  
  // atomically increments the value at key by one. 
  // The value must be an integer.
  increment: func(key: key) -> result<u64, Error>

  // compare and swap (CAS) is an atomic operation that compares the value of a key 
  // with a given value and, if they are equal, updates that key to a new value.
  // This is useful for achieving synchronization between multiple processes.
  compare_and_swap: func(key: key, old: u64, new: u64) -> result<bool, Error>
}

interface "wasi:kv/data/get-many" {
  // get_many returns a stream of key-value pairs.
  // Notice hat this interface is non-atomic, meaning that the key-value pairs
  // returned may not be consistent.

  use { Error, payload } from "wasi:kv/types"
  
  get-many: func(keys: keys) -> result<stream<payload>, Error>

  keys: func() -> result<keys, Error>
}

interface "wasi:kv/data/set-many" {
  // set_many sets multiple key-value pairs.
  // Notice that this interface is non-atomic, meaning that the key-value pairs
  // may not be consistent.
  use { Error, key } from "wasi:kv/types"
  
  set-many: func(key_values: stream<(key, stream<u8>)>) -> result<_, Error>
}

interface "wasi:kv/types" {
  resource Error { 
    trace: func() -> string

    // possibly more methods
  }
  resource payload {
    consume_async: func() -> result<stream<u8>, Error>
    consume_sync: func() -> result<list<u8>, Error>

    // possibly more methods, like consume_async_with_key that returns a key payload pair `(key, stream<u8>)`
  }

  type key = string
  type keys = stream<key>
}
```

- The `bulk-get` and `bulk-set` are atomic.
- The `increment` is atomic, in a way that it is a small transaction of get, increment, and set operations on the same key.

```go
world "wasi:cloud/services" {
  import kv: {*: "wasi:cloud/kv"}
  import mq: {*: "wasi:cloud/mq"}
  ...

  export http: "wasi:http/handler"
}

world "wasi:cloud/kv" {
  import kv: {*: "wasi:kv/data/crud"}
  
  export http: "wasi:http/handler"
}
```

The following interfaces are still under discussion:

```go
interface "wasi:kv/data/transaction" {
  // transaction is an atomic operation that groups multiple operations together.
  // If any operation fails, all operations in the transaction are rolled back.
  use { Error, payload, key } from "wasi:kv/types"

  bulk-get: func(keys: keys) -> result<stream<payload>, Error>
  
  bulk-set: func(key_values: stream<(key, stream<u8>)>) -> result<_, Error>
}

interface "wasi:kv/data/ttl" {
  use { Error, key } from "wasi:kv/types"
  
  set-with-ttl: func(key: key, value: stream<u8>, ttl: u64) -> result<_, Error>
}

interface "wasi:kv/data/query" { 
  ...
}
```

#### Tricky design choice 1

TODO

#### Tricky design choice 2

TODO

### Considered alternatives

TODO

#### Alternative 1

TODO
#### Alternative 2

TODO

### Stakeholder Interest & Feedback

TODO before entering Phase 3.

### References & acknowledgements

Many thanks for valuable feedback and advice from:

- [Person 1]
- [Person 2]
- [etc.]
