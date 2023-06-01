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

Imagine you have an HTTP handler that needs to persist some data to a key-value store. The handler needs to be able to retrieve, delete, and update the data in the key-value store. The following Rust and Go code shows how you can use the WASI Key-Value Store API to in the handler.

```rust
wit_bindgen::generate!("outbound-keyvalue" in "wit/world.wit");
use types::{ open_bucket, drop_bucket, payload_consume_sync };
use readwrite::{ get, set, exists };
{
  let my_bucket = open_bucket("my-bucket")?;
  set(my_bucket, "my-key", "my-value")?;
  if exists(my_bucket, "my-key")? {
    let payload = get(my_bucket, "my-key")?;
    let body: Vec<u8> = payload_consume_sync(payload)?;
    let body = String::from_utf8(body)?;
    println!("body: {}", body);
  }
  drop_bucket(my_bucket);
}
```

```go
bucket, err := types.OpenBucket("my-bucket")
if err != nil {
  panic(err)
}
defer types.DropBucket(bucket)

err := readwrite.Set(bucket, "my-key", "my-value")
if err != nil {
  panic(err)
}

exists, err := readwrite.Exists(bucket, "my-key")
if exists {
  payload, err := readwrite.Get(bucket, "my-key")
  if err != nil {
    panic(err)
  }
  body, err := types.PayloadConsumeSync(payload)
  if err != nil {
    panic(err)
  }
  fmt.Println("body: ", string(body))
}
```

#### Use case 2

If you want to watch for changes to a key-value store, you can write a wasm component that uses the inbound-keyvalue interface this API provides. The following Rust code shows how you can use the WASI Key-Value Store API to watch for changes to a key-value store.

```rust
wit_bindgen::generate!("inbound-keyvalue" in "wit/world.wit");
struct Handler;

impl inbound_keyvalue::HandleWatch for Handler {
  fn on_set(&mut self, bucket: Bucket, key: Key, value: Payload) {
    let payload = get(my_bucket, "my-key").unwrap();
    let body: Vec<u8> = payload_consume_sync(payload).unwrap();
    let body = String::from_utf8(body).unwrap();
    println!("bucket: {}, key: {}, value: {}", bucket, key, body);
  }
  fn on_delete(&mut self, bucket: Bucket, key: Key) {
    println!("bucket: {}, key: {}", bucket, key);
  }
}

```

### Detailed design discussion

```go
/// A keyvalue interface that provides simple read and write operations.
interface readwrite {
	/// A keyvalue interface that provides simple read and write operations.
	use types.{bucket, error, incoming-value, key, outgoing-value}
	
	/// Get the value associated with the key in the bucket. It returns a incoming-value
	/// that can be consumed to get the value.
	///
	/// If the key does not exist in the bucket, it returns an error.
	get: func(bucket: bucket, key: key) -> result<incoming-value, error>

	/// Set the value associated with the key in the bucket. If the key already
	/// exists in the bucket, it overwrites the value.
	///
	/// If the key does not exist in the bucket, it creates a new key-value pair.
	/// If any other error occurs, it returns an error.
	set: func(bucket: bucket, key: key, outgoing-value: outgoing-value) -> result<_, error>

	/// Delete the key-value pair associated with the key in the bucket.
	///
	/// If the key does not exist in the bucket, it returns an error.
	delete: func(bucket: bucket, key: key) -> result<_, error>

	/// Check if the key exists in the bucket.
	///
	/// If the key does not exist in the bucket, it returns an error.
	exists: func(bucket: bucket, key: key) -> result<bool, error>
}
/// A keyvalue interface that provides atomic operations.
interface atomic {
	/// A keyvalue interface that provides atomic operations.
	use types.{bucket, error, key}

	/// Atomically increment the value associated with the key in the bucket by the 
	/// given delta. It returns the new value.
	///
	/// If the key does not exist in the bucket, it creates a new key-value pair
	/// with the value set to the given delta. 
	///
	/// If any other error occurs, it returns an error.
	increment: func(bucket: bucket, key: key, delta: u64) -> result<u64, error>
	
	/// Atomically compare and swap the value associated with the key in the bucket.
	/// It returns a boolean indicating if the swap was successful.
	///
	/// If the key does not exist in the bucket, it returns an error.
	compare-and-swap: func(bucket: bucket, key: key, old: u64, new: u64) -> result<bool, error>
}

/// A keyvalue interface that provides batch operations.
interface batch {
	/// A keyvalue interface that provides batch get operations.
	use types.{bucket, error, key, keys, incoming-value, outgoing-value}

	/// Get the values associated with the keys in the bucket. It returns a list of
	/// incoming-values that can be consumed to get the values.
	///
	/// If any of the keys do not exist in the bucket, it returns an error.
	get-many: func(bucket: bucket, keys: keys) -> result<list<incoming-value>, error>

	/// Get all the keys in the bucket. It returns a list of keys.
	get-keys: func(bucket: bucket) -> keys

	/// Set the values associated with the keys in the bucket. If the key already
	/// exists in the bucket, it overwrites the value.
	///
	/// If any of the keys do not exist in the bucket, it creates a new key-value pair.
	/// If any other error occurs, it returns an error.
	set-many: func(bucket: bucket, keys: keys, values: list<tuple<key, outgoing-value>>) -> result<_, error>

	/// Delete the key-value pairs associated with the keys in the bucket.
	///
	/// If any of the keys do not exist in the bucket, it skips the key.
	/// If any other error occurs, it returns an error.
	delete-many: func(bucket: bucket, keys: keys) -> result<_, error>
}

// A generic keyvalue interface for WASI.
interface types {
	/// A bucket is a collection of key-value pairs. Each key-value pair is stored
	/// as a entry in the bucket, and the bucket itself acts as a collection of all
	/// these entries. 
	///
	/// It is worth noting that the exact terminology for bucket in key-value stores
	/// can very depending on the specific implementation. For example,
	/// 1. Amazon DynamoDB calls a collection of key-value pairs a table
	/// 2. Redis has hashes, sets, and sorted sets as different types of collections
	/// 3. Cassandra calls a collection of key-value pairs a column family
	/// 4. MongoDB calls a collection of key-value pairs a collection
	/// 5. Riak calls a collection of key-value pairs a bucket
	/// 6. Memcached calls a collection of key-value pairs a slab
	/// 7. Azure Cosmos DB calls a collection of key-value pairs a container
	///
	/// In this interface, we use the term `bucket` to refer to a collection of key-value
	// Soon: switch to `resource bucket { ... }`
	type bucket = u32
	drop-bucket: func(bucket: bucket)
	open-bucket: func(name: string) -> result<bucket, error>

	/// A key is a unique identifier for a value in a bucket. The key is used to
	/// retrieve the value from the bucket.
	type key = string

	/// A list of keys
	type keys = list<key>

	use wasi:io/streams.{input-stream, output-stream}
	use wasi-cloud-error.{ error }
	/// A value is the data stored in a key-value pair. The value can be of any type
	/// that can be represented in a byte array. It provides a way to write the value
	/// to the output-stream defined in the `wasi-io` interface.
	// Soon: switch to `resource value { ... }`
	type outgoing-value = u32
	drop-outgoing-value: func(outgoing-value: outgoing-value)
	new-outgoing-value: func() -> outgoing-value
	outgoing-value-write-body: func(outgoing-value: outgoing-value) -> result<output-stream>

	/// A incoming-value is a wrapper around a value. It provides a way to read the value
	/// from the input-stream defined in the `wasi-io` interface.
	///
	/// The incoming-value provides two ways to consume the value:
	/// 1. `incoming-value-consume-sync` consumes the value synchronously and returns the
	///    value as a list of bytes.
	/// 2. `incoming-value-consume-async` consumes the value asynchronously and returns the
	///    value as an input-stream.
	// Soon: switch to `resource incoming-value { ... }`
	type incoming-value = u32
	type incoming-value-async-body = input-stream
	type incoming-value-sync-body = list<u8>
	drop-incoming-value: func(incoming-value: incoming-value)
	incoming-value-consume-sync: func(incoming-value: incoming-value) -> result<incoming-value-sync-body, error>
	incoming-value-consume-async: func(incoming-value: incoming-value) -> result<incoming-value-async-body, error>
	size: func(incoming-value: incoming-value) -> u64
}
```

- The `get-many` and `set-many` are atomic.
- The `increment` is atomic, in a way that it is a small transaction of get, increment, and set operations on the same key.

The following interfaces are still under discussion:

```go
interface transaction {
  // transaction is an atomic operation that groups multiple operations together.
  // If any operation fails, all operations in the transaction are rolled back.
  use types.{ Error, payload, key }

  get-multi: func(keys: keys) -> result<list<payload>, Error>
  
  set-multi: func(key_values: list<(key, list<u8>)>) -> result<_, Error>
}

interface ttl {
  use types.{ Error, key }
  
  set-with-ttl: func(key: key, value: list<u8>, ttl: u64) -> result<_, Error>
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

### Change log

- 2023-05-17: Updated batch example to use one interface instead of 2
- 2023-05-25: Change the WITs to the newest syntax.
- 2023-02-13: The following changes were made to the API:
  - Added `bucket` type to the `types` interface.
  - Uses pseudo-stream and pseudo-resource types inspired from [wasi-io ](https://github.com/WebAssembly/wasi-io)
  - Added *.wit files for the interfaces and worlds, which are verified by the [wasm-tools](https://github.com/bytecodealliance/wasm-tools)
  - Added a inbound-keyvalue interface that handles the `watch` operation.
- 2023-01-28: The following changes were made to the API:
  - Changed `stream<T>` type to `list<T>` type because `stream<T>` is not supported by the current version of [*.wit](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md).
  - Removed the worlds section because the star import syntax is not supported by the current version of [*.wit](https://github.com/WebAssembly/component-model/blob/main/design/mvp/WIT.md)
- 2022-11-29: Renamed `bulk-get` to `get-multi` and `bulk-set` to `set-multi` to be consistent with the naming convention of the other interfaces.
- 2022-10-31: Initial draft
