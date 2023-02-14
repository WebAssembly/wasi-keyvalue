# <a name="inbound_keyvalue">World inbound-keyvalue</a>


 - Imports:
    - interface `wasi-io`
    - interface `types`
 - Exports:
    - interface `handle-watch`

## <a name="wasi_io">Import interface wasi-io</a>


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

## <a name="types">Import interface types</a>


----

### Types

#### <a name="input_stream">`type input-stream`</a>
[`input-stream`](#input_stream)
<p>
#### <a name="output_stream">`type output-stream`</a>
[`output-stream`](#output_stream)
<p>
#### <a name="value">`type value`</a>
`u32`
<p>A value is the data stored in a key-value pair. The value can be of any type
that can be represented in a byte array. It provides a way to write the value
to the output-stream defined in the `wasi-io` interface.

#### <a name="payload_sync_body">`type payload-sync-body`</a>
[`payload-sync-body`](#payload_sync_body)
<p>
#### <a name="payload">`type payload`</a>
`u32`
<p>A payload is a wrapper around a value. It provides a way to read the value
from the input-stream defined in the `wasi-io` interface.

The payload provides two ways to consume the value:
1. `payload-consume-sync` consumes the value synchronously and returns the
value as a list of bytes.
2. `payload-consume-async` consumes the value asynchronously and returns the
value as an input-stream.

#### <a name="key">`type key`</a>
`string`
<p>A key is a unique identifier for a value in a bucket. The key is used to
retrieve the value from the bucket.

#### <a name="keys">`type keys`</a>
[`keys`](#keys)
<p>A list of keys

#### <a name="payload_async_body">`type payload-async-body`</a>
[`input-stream`](#input_stream)
<p>
#### <a name="error">`type error`</a>
`u32`
<p>An error resource type for keyvalue operations.
Currently, this provides only one function to return a string representation
of the error. In the future, this will be extended to provide more information
about the error.

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

#### <a name="drop_error">`drop-error: func`</a>


##### Params

- <a name="drop_error.error">`error`</a>: [`error`](#error)

#### <a name="new_error">`new-error: func`</a>


##### Return values

- <a name="new_error.0"></a> [`error`](#error)

#### <a name="trace">`trace: func`</a>


##### Params

- <a name="trace.error">`error`</a>: [`error`](#error)

##### Return values

- <a name="trace.0"></a> `string`

#### <a name="drop_value">`drop-value: func`</a>


##### Params

- <a name="drop_value.value">`value`</a>: [`value`](#value)

#### <a name="new_value">`new-value: func`</a>


##### Return values

- <a name="new_value.0"></a> [`value`](#value)

#### <a name="value_write_body">`value-write-body: func`</a>


##### Params

- <a name="value_write_body.value">`value`</a>: [`value`](#value)

##### Return values

- <a name="value_write_body.0"></a> result<[`output-stream`](#output_stream)>

#### <a name="drop_payload">`drop-payload: func`</a>


##### Params

- <a name="drop_payload.payload">`payload`</a>: [`payload`](#payload)

#### <a name="payload_consume_sync">`payload-consume-sync: func`</a>


##### Params

- <a name="payload_consume_sync.payload">`payload`</a>: [`payload`](#payload)

##### Return values

- <a name="payload_consume_sync.0"></a> result<[`payload-sync-body`](#payload_sync_body), [`error`](#error)>

#### <a name="payload_consume_async">`payload-consume-async: func`</a>


##### Params

- <a name="payload_consume_async.payload">`payload`</a>: [`payload`](#payload)

##### Return values

- <a name="payload_consume_async.0"></a> result<[`payload-async-body`](#payload_async_body), [`error`](#error)>

#### <a name="size">`size: func`</a>


##### Params

- <a name="size.payload">`payload`</a>: [`payload`](#payload)

##### Return values

- <a name="size.0"></a> `u64`

## <a name="handle_watch">Export interface handle-watch</a>

----

### Types

#### <a name="bucket">`type bucket`</a>
[`bucket`](#bucket)
<p>
#### <a name="key">`type key`</a>
[`key`](#key)
<p>
#### <a name="payload">`type payload`</a>
[`payload`](#payload)
<p>
----

### Functions

#### <a name="on_set">`on-set: func`</a>

Handle the set event for the given bucket and key.
It returns a payload that can be consumed to get the value.

##### Params

- <a name="on_set.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="on_set.key">`key`</a>: [`key`](#key)
- <a name="on_set.payload">`payload`</a>: [`payload`](#payload)

#### <a name="on_delete">`on-delete: func`</a>

Handle the delete event for the given bucket and key.

##### Params

- <a name="on_delete.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="on_delete.key">`key`</a>: [`key`](#key)

