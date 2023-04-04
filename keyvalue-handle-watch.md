# <a name="keyvalue_handle_watch">World keyvalue-handle-watch</a>


 - Imports:
    - interface `streams`
    - interface `wasi-cloud-error`
    - interface `types`
    - interface `readwrite`
    - interface `atomic`
    - interface `batch`
 - Exports:
    - interface `handle-watch`

## <a name="streams">Import interface streams</a>


----

### Types

#### <a name="stream_error">`record stream-error`</a>

An error type returned from a stream operation. Currently this
doesn't provide any additional information.

##### Record Fields

#### <a name="output_stream">`type output-stream`</a>
`u32`
<p>An output bytestream. In the future, this will be replaced by handle
types.

This conceptually represents a `stream<u8, _>`. It's temporary
scaffolding until component-model's async features are ready.

And at present, it is a `u32` instead of being an actual handle, until
the wit-bindgen implementation of handles and resources is ready.

#### <a name="input_stream">`type input-stream`</a>
`u32`
<p>An input bytestream. In the future, this will be replaced by handle
types.

This conceptually represents a `stream<u8, _>`. It's temporary
scaffolding until component-model's async features are ready.

And at present, it is a `u32` instead of being an actual handle, until
the wit-bindgen implementation of handles and resources is ready.

----

### Functions

#### <a name="read">`read: func`</a>

Read bytes from a stream.

This function returns a list of bytes containing the data that was
read, along with a bool indicating whether the end of the stream
was reached. The returned list will contain up to `len` bytes; it
may return fewer than requested, but not more.

Once a stream has reached the end, subsequent calls to read or
`skip` will always report end-of-stream rather than producing more
data.

If `len` is 0, it represents a request to read 0 bytes, which should
always succeed, assuming the stream hasn't reached its end yet, and
return an empty list.

The len here is a `u64`, but some callees may not be able to allocate
a buffer as large as that would imply.
FIXME: describe what happens if allocation fails.

##### Params

- <a name="read.src">`src`</a>: [`input-stream`](#input_stream)
- <a name="read.len">`len`</a>: `u64`

##### Return values

- <a name="read.0"></a> result<(list<`u8`>, `bool`), [`stream-error`](#stream_error)>

#### <a name="skip">`skip: func`</a>

Skip bytes from a stream.

This is similar to the `read` function, but avoids copying the
bytes into the instance.

Once a stream has reached the end, subsequent calls to read or
`skip` will always report end-of-stream rather than producing more
data.

This function returns the number of bytes skipped, along with a bool
indicating whether the end of the stream was reached. The returned
value will be at most `len`; it may be less.

##### Params

- <a name="skip.src">`src`</a>: [`input-stream`](#input_stream)
- <a name="skip.len">`len`</a>: `u64`

##### Return values

- <a name="skip.0"></a> result<(`u64`, `bool`), [`stream-error`](#stream_error)>

#### <a name="write">`write: func`</a>

Write bytes to a stream.

This function returns a `u64` indicating the number of bytes from
`buf` that were written; it may be less than the full list.

##### Params

- <a name="write.dst">`dst`</a>: [`output-stream`](#output_stream)
- <a name="write.buf">`buf`</a>: list<`u8`>

##### Return values

- <a name="write.0"></a> result<`u64`, [`stream-error`](#stream_error)>

#### <a name="write_repeated">`write-repeated: func`</a>

Write a single byte multiple times to a stream.

This function returns a `u64` indicating the number of copies of
`byte` that were written; it may be less than `len`.

##### Params

- <a name="write_repeated.dst">`dst`</a>: [`output-stream`](#output_stream)
- <a name="write_repeated.byte">`byte`</a>: `u8`
- <a name="write_repeated.len">`len`</a>: `u64`

##### Return values

- <a name="write_repeated.0"></a> result<`u64`, [`stream-error`](#stream_error)>

#### <a name="splice">`splice: func`</a>

Read from one stream and write to another.

This function returns the number of bytes transferred; it may be less
than `len`.

##### Params

- <a name="splice.dst">`dst`</a>: [`output-stream`](#output_stream)
- <a name="splice.src">`src`</a>: [`input-stream`](#input_stream)
- <a name="splice.len">`len`</a>: `u64`

##### Return values

- <a name="splice.0"></a> result<(`u64`, `bool`), [`stream-error`](#stream_error)>

#### <a name="forward">`forward: func`</a>

Forward the entire contents of an input stream to an output stream.

This function repeatedly reads from the input stream and writes
the data to the output stream, until the end of the input stream
is reached, or an error is encountered.

This function returns the number of bytes transferred.

##### Params

- <a name="forward.dst">`dst`</a>: [`output-stream`](#output_stream)
- <a name="forward.src">`src`</a>: [`input-stream`](#input_stream)

##### Return values

- <a name="forward.0"></a> result<`u64`, [`stream-error`](#stream_error)>

#### <a name="drop_input_stream">`drop-input-stream: func`</a>

Dispose of the specified input-stream, after which it may no longer
be used.

##### Params

- <a name="drop_input_stream.f">`f`</a>: [`input-stream`](#input_stream)

#### <a name="drop_output_stream">`drop-output-stream: func`</a>

Dispose of the specified output-stream, after which it may no longer
be used.

##### Params

- <a name="drop_output_stream.f">`f`</a>: [`output-stream`](#output_stream)

## <a name="wasi_cloud_error">Import interface wasi-cloud-error</a>


----

### Types

#### <a name="error">`type error`</a>
`u32`
<p>An error resource type for keyvalue operations.
Currently, this provides only one function to return a string representation
of the error. In the future, this will be extended to provide more information
about the error.

----

### Functions

#### <a name="drop_error">`drop-error: func`</a>


##### Params

- <a name="drop_error.error">`error`</a>: [`error`](#error)

#### <a name="trace">`trace: func`</a>


##### Params

- <a name="trace.error">`error`</a>: [`error`](#error)

##### Return values

- <a name="trace.0"></a> `string`

## <a name="types">Import interface types</a>


----

### Types

#### <a name="input_stream">`type input-stream`</a>
[`input-stream`](#input_stream)
<p>
#### <a name="output_stream">`type output-stream`</a>
[`output-stream`](#output_stream)
<p>
#### <a name="error">`type error`</a>
[`error`](#error)
<p>
#### <a name="outgoing_value">`type outgoing-value`</a>
`u32`
<p>A value is the data stored in a key-value pair. The value can be of any type
that can be represented in a byte array. It provides a way to write the value
to the output-stream defined in the `wasi-io` interface.

#### <a name="key">`type key`</a>
`string`
<p>A key is a unique identifier for a value in a bucket. The key is used to
retrieve the value from the bucket.

#### <a name="keys">`type keys`</a>
[`keys`](#keys)
<p>A list of keys

#### <a name="incoming_value_sync_body">`type incoming-value-sync-body`</a>
[`incoming-value-sync-body`](#incoming_value_sync_body)
<p>
#### <a name="incoming_value_async_body">`type incoming-value-async-body`</a>
[`input-stream`](#input_stream)
<p>
#### <a name="incoming_value">`type incoming-value`</a>
`u32`
<p>A incoming-value is a wrapper around a value. It provides a way to read the value
from the input-stream defined in the `wasi-io` interface.

The incoming-value provides two ways to consume the value:
1. `incoming-value-consume-sync` consumes the value synchronously and returns the
value as a list of bytes.
2. `incoming-value-consume-async` consumes the value asynchronously and returns the
value as an input-stream.

#### <a name="bucket">`type bucket`</a>
`u32`
<p>A bucket is a collection of key-value pairs. Each key-value pair is stored
as a entry in the bucket, and the bucket itself acts as a collection of all
these entries.

It is worth noting that the exact terminology for bucket in key-value stores
can very depending on the specific implementation. For example,
1. Amazon DynamoDB calls a collection of key-value pairs a table
2. Redis has hashes, sets, and sorted sets as different types of collections
3. Cassandra calls a collection of key-value pairs a column family
4. MongoDB calls a collection of key-value pairs a collection
5. Riak calls a collection of key-value pairs a bucket
6. Memcached calls a collection of key-value pairs a slab
7. Azure Cosmos DB calls a collection of key-value pairs a container

In this interface, we use the term `bucket` to refer to a collection of key-value

----

### Functions

#### <a name="drop_bucket">`drop-bucket: func`</a>


##### Params

- <a name="drop_bucket.bucket">`bucket`</a>: [`bucket`](#bucket)

#### <a name="open_bucket">`open-bucket: func`</a>


##### Params

- <a name="open_bucket.name">`name`</a>: `string`

##### Return values

- <a name="open_bucket.0"></a> result<[`bucket`](#bucket), [`error`](#error)>

#### <a name="drop_outgoing_value">`drop-outgoing-value: func`</a>


##### Params

- <a name="drop_outgoing_value.outgoing_value">`outgoing-value`</a>: [`outgoing-value`](#outgoing_value)

#### <a name="new_outgoing_value">`new-outgoing-value: func`</a>


##### Return values

- <a name="new_outgoing_value.0"></a> [`outgoing-value`](#outgoing_value)

#### <a name="outgoing_value_write_body">`outgoing-value-write-body: func`</a>


##### Params

- <a name="outgoing_value_write_body.outgoing_value">`outgoing-value`</a>: [`outgoing-value`](#outgoing_value)

##### Return values

- <a name="outgoing_value_write_body.0"></a> result<[`output-stream`](#output_stream)>

#### <a name="drop_incoming_value">`drop-incoming-value: func`</a>


##### Params

- <a name="drop_incoming_value.incoming_value">`incoming-value`</a>: [`incoming-value`](#incoming_value)

#### <a name="incoming_value_consume_sync">`incoming-value-consume-sync: func`</a>


##### Params

- <a name="incoming_value_consume_sync.incoming_value">`incoming-value`</a>: [`incoming-value`](#incoming_value)

##### Return values

- <a name="incoming_value_consume_sync.0"></a> result<[`incoming-value-sync-body`](#incoming_value_sync_body), [`error`](#error)>

#### <a name="incoming_value_consume_async">`incoming-value-consume-async: func`</a>


##### Params

- <a name="incoming_value_consume_async.incoming_value">`incoming-value`</a>: [`incoming-value`](#incoming_value)

##### Return values

- <a name="incoming_value_consume_async.0"></a> result<[`incoming-value-async-body`](#incoming_value_async_body), [`error`](#error)>

#### <a name="size">`size: func`</a>


##### Params

- <a name="size.incoming_value">`incoming-value`</a>: [`incoming-value`](#incoming_value)

##### Return values

- <a name="size.0"></a> `u64`

## <a name="readwrite">Import interface readwrite</a>

A keyvalue interface that provides simple read and write operations.

----

### Types

#### <a name="bucket">`type bucket`</a>
[`bucket`](#bucket)
<p>
#### <a name="error">`type error`</a>
[`error`](#error)
<p>
#### <a name="incoming_value">`type incoming-value`</a>
[`incoming-value`](#incoming_value)
<p>
#### <a name="key">`type key`</a>
[`key`](#key)
<p>
#### <a name="outgoing_value">`type outgoing-value`</a>
[`outgoing-value`](#outgoing_value)
<p>
----

### Functions

#### <a name="get">`get: func`</a>

Get the value associated with the key in the bucket. It returns a incoming-value
that can be consumed to get the value.

If the key does not exist in the bucket, it returns an error.

##### Params

- <a name="get.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="get.key">`key`</a>: [`key`](#key)

##### Return values

- <a name="get.0"></a> result<[`incoming-value`](#incoming_value), [`error`](#error)>

#### <a name="set">`set: func`</a>

Set the value associated with the key in the bucket. If the key already
exists in the bucket, it overwrites the value.

If the key does not exist in the bucket, it creates a new key-value pair.
If any other error occurs, it returns an error.

##### Params

- <a name="set.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="set.key">`key`</a>: [`key`](#key)
- <a name="set.outgoing_value">`outgoing-value`</a>: [`outgoing-value`](#outgoing_value)

##### Return values

- <a name="set.0"></a> result<_, [`error`](#error)>

#### <a name="delete">`delete: func`</a>

Delete the key-value pair associated with the key in the bucket.

If the key does not exist in the bucket, it returns an error.

##### Params

- <a name="delete.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="delete.key">`key`</a>: [`key`](#key)

##### Return values

- <a name="delete.0"></a> result<_, [`error`](#error)>

#### <a name="exists">`exists: func`</a>

Check if the key exists in the bucket.

If the key does not exist in the bucket, it returns an error.

##### Params

- <a name="exists.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="exists.key">`key`</a>: [`key`](#key)

##### Return values

- <a name="exists.0"></a> result<`bool`, [`error`](#error)>

## <a name="atomic">Import interface atomic</a>

A keyvalue interface that provides atomic operations.

----

### Types

#### <a name="bucket">`type bucket`</a>
[`bucket`](#bucket)
<p>
#### <a name="error">`type error`</a>
[`error`](#error)
<p>
#### <a name="key">`type key`</a>
[`key`](#key)
<p>
----

### Functions

#### <a name="increment">`increment: func`</a>

Atomically increment the value associated with the key in the bucket by the
given delta. It returns the new value.

If the key does not exist in the bucket, it creates a new key-value pair
with the value set to the given delta.

If any other error occurs, it returns an error.

##### Params

- <a name="increment.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="increment.key">`key`</a>: [`key`](#key)
- <a name="increment.delta">`delta`</a>: `u64`

##### Return values

- <a name="increment.0"></a> result<`u64`, [`error`](#error)>

#### <a name="compare_and_swap">`compare-and-swap: func`</a>

Atomically compare and swap the value associated with the key in the bucket.
It returns a boolean indicating if the swap was successful.

If the key does not exist in the bucket, it returns an error.

##### Params

- <a name="compare_and_swap.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="compare_and_swap.key">`key`</a>: [`key`](#key)
- <a name="compare_and_swap.old">`old`</a>: `u64`
- <a name="compare_and_swap.new">`new`</a>: `u64`

##### Return values

- <a name="compare_and_swap.0"></a> result<`bool`, [`error`](#error)>

## <a name="batch">Import interface batch</a>

A keyvalue interface that provides batch operations.

----

### Types

#### <a name="bucket">`type bucket`</a>
[`bucket`](#bucket)
<p>
#### <a name="error">`type error`</a>
[`error`](#error)
<p>
#### <a name="key">`type key`</a>
[`key`](#key)
<p>
#### <a name="keys">`type keys`</a>
[`keys`](#keys)
<p>
#### <a name="incoming_value">`type incoming-value`</a>
[`incoming-value`](#incoming_value)
<p>
#### <a name="outgoing_value">`type outgoing-value`</a>
[`outgoing-value`](#outgoing_value)
<p>
----

### Functions

#### <a name="get_many">`get-many: func`</a>

Get the values associated with the keys in the bucket. It returns a list of
incoming-values that can be consumed to get the values.

If any of the keys do not exist in the bucket, it returns an error.

##### Params

- <a name="get_many.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="get_many.keys">`keys`</a>: [`keys`](#keys)

##### Return values

- <a name="get_many.0"></a> result<list<[`incoming-value`](#incoming_value)>, [`error`](#error)>

#### <a name="get_keys">`get-keys: func`</a>

Get all the keys in the bucket. It returns a list of keys.

##### Params

- <a name="get_keys.bucket">`bucket`</a>: [`bucket`](#bucket)

##### Return values

- <a name="get_keys.0"></a> [`keys`](#keys)

#### <a name="set_many">`set-many: func`</a>

Set the values associated with the keys in the bucket. If the key already
exists in the bucket, it overwrites the value.

If any of the keys do not exist in the bucket, it creates a new key-value pair.
If any other error occurs, it returns an error.

##### Params

- <a name="set_many.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="set_many.keys">`keys`</a>: [`keys`](#keys)
- <a name="set_many.values">`values`</a>: list<([`key`](#key), [`outgoing-value`](#outgoing_value))>

##### Return values

- <a name="set_many.0"></a> result<_, [`error`](#error)>

#### <a name="delete_many">`delete-many: func`</a>

Delete the key-value pairs associated with the keys in the bucket.

If any of the keys do not exist in the bucket, it skips the key.
If any other error occurs, it returns an error.

##### Params

- <a name="delete_many.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="delete_many.keys">`keys`</a>: [`keys`](#keys)

##### Return values

- <a name="delete_many.0"></a> result<_, [`error`](#error)>

## <a name="handle_watch">Export interface handle-watch</a>

----

### Types

#### <a name="bucket">`type bucket`</a>
[`bucket`](#bucket)
<p>
#### <a name="key">`type key`</a>
[`key`](#key)
<p>
#### <a name="incoming_value">`type incoming-value`</a>
[`incoming-value`](#incoming_value)
<p>
----

### Functions

#### <a name="on_set">`on-set: func`</a>

Handle the set event for the given bucket and key.
It returns a incoming-value that can be consumed to get the value.

##### Params

- <a name="on_set.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="on_set.key">`key`</a>: [`key`](#key)
- <a name="on_set.incoming_value">`incoming-value`</a>: [`incoming-value`](#incoming_value)

#### <a name="on_delete">`on-delete: func`</a>

Handle the delete event for the given bucket and key.

##### Params

- <a name="on_delete.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="on_delete.key">`key`</a>: [`key`](#key)

