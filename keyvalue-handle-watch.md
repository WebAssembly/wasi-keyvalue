# <a name="keyvalue_handle_watch">World keyvalue-handle-watch</a>


 - Imports:
    - interface `wasi:poll/poll`
    - interface `wasi:io/streams`
    - interface `wasi:keyvalue/wasi-cloud-error`
    - interface `wasi:keyvalue/types`
    - interface `wasi:keyvalue/readwrite`
    - interface `wasi:keyvalue/atomic`
    - interface `wasi:keyvalue/batch`
 - Exports:
    - interface `wasi:keyvalue/handle-watch`

## <a name="wasi:poll_poll">Import interface wasi:poll/poll</a>

A poll API intended to let users wait for I/O events on multiple handles
at once.

----

### Types

#### <a name="pollable">`type pollable`</a>
`u32`
<p>A "pollable" handle.

This is conceptually represents a `stream<_, _>`, or in other words,
a stream that one can wait on, repeatedly, but which does not itself
produce any data. It's temporary scaffolding until component-model's
async features are ready.

And at present, it is a `u32` instead of being an actual handle, until
the wit-bindgen implementation of handles and resources is ready.

`pollable` lifetimes are not automatically managed. Users must ensure
that they do not outlive the resource they reference.

This [represents a resource](https://github.com/WebAssembly/WASI/blob/main/docs/WitInWasi.md#Resources).

----

### Functions

#### <a name="drop_pollable">`drop-pollable: func`</a>

Dispose of the specified `pollable`, after which it may no longer
be used.

##### Params

- <a name="drop_pollable.this">`this`</a>: [`pollable`](#pollable)

#### <a name="poll_oneoff">`poll-oneoff: func`</a>

Poll for completion on a set of pollables.

This function takes a list of pollables, which identify I/O sources of
interest, and waits until one or more of the events is ready for I/O.

The result `list<bool>` is the same length as the argument
`list<pollable>`, and indicates the readiness of each corresponding
element in that list, with true indicating ready. A single call can
return multiple true elements.

A timeout can be implemented by adding a pollable from the
wasi-clocks API to the list.

This function does not return a `result`; polling in itself does not
do any I/O so it doesn't fail. If any of the I/O sources identified by
the pollables has an error, it is indicated by marking the source as
ready in the `list<bool>`.

The "oneoff" in the name refers to the fact that this function must do a
linear scan through the entire list of subscriptions, which may be
inefficient if the number is large and the same subscriptions are used
many times. In the future, this is expected to be obsoleted by the
component model async proposal, which will include a scalable waiting
facility.

##### Params

- <a name="poll_oneoff.in">`in`</a>: list<[`pollable`](#pollable)>

##### Return values

- <a name="poll_oneoff.0"></a> list<`bool`>

## <a name="wasi:io_streams">Import interface wasi:io/streams</a>

WASI I/O is an I/O abstraction API which is currently focused on providing
stream types.

In the future, the component model is expected to add built-in stream types;
when it does, they are expected to subsume this API.

----

### Types

#### <a name="pollable">`type pollable`</a>
[`pollable`](#pollable)
<p>
#### <a name="stream_error">`record stream-error`</a>

An error type returned from a stream operation. Currently this
doesn't provide any additional information.

##### Record Fields

#### <a name="stream_status">`enum stream-status`</a>

Streams provide a sequence of data and then end; once they end, they
no longer provide any further data.

For example, a stream reading from a file ends when the stream reaches
the end of the file. For another example, a stream reading from a
socket ends when the socket is closed.

##### Enum Cases

- <a name="stream_status.open">`open`</a>
  <p>The stream is open and may produce further data.
  
- <a name="stream_status.ended">`ended`</a>
  <p>The stream has ended and will not produce any further data.
  
#### <a name="input_stream">`type input-stream`</a>
`u32`
<p>An input bytestream. In the future, this will be replaced by handle
types.

This conceptually represents a `stream<u8, _>`. It's temporary
scaffolding until component-model's async features are ready.

`input-stream`s are *non-blocking* to the extent practical on underlying
platforms. I/O operations always return promptly; if fewer bytes are
promptly available than requested, they return the number of bytes promptly
available, which could even be zero. To wait for data to be available,
use the `subscribe-to-input-stream` function to obtain a `pollable` which
can be polled for using `wasi_poll`.

And at present, it is a `u32` instead of being an actual handle, until
the wit-bindgen implementation of handles and resources is ready.

This [represents a resource](https://github.com/WebAssembly/WASI/blob/main/docs/WitInWasi.md#Resources).

#### <a name="output_stream">`type output-stream`</a>
`u32`
<p>An output bytestream. In the future, this will be replaced by handle
types.

This conceptually represents a `stream<u8, _>`. It's temporary
scaffolding until component-model's async features are ready.

`output-stream`s are *non-blocking* to the extent practical on
underlying platforms. Except where specified otherwise, I/O operations also
always return promptly, after the number of bytes that can be written
promptly, which could even be zero. To wait for the stream to be ready to
accept data, the `subscribe-to-output-stream` function to obtain a
`pollable` which can be polled for using `wasi_poll`.

And at present, it is a `u32` instead of being an actual handle, until
the wit-bindgen implementation of handles and resources is ready.

This [represents a resource](https://github.com/WebAssembly/WASI/blob/main/docs/WitInWasi.md#Resources).

----

### Functions

#### <a name="read">`read: func`</a>

Read bytes from a stream.

This function returns a list of bytes containing the data that was
read, along with a `stream-status` which indicates whether the end of
the stream was reached. The returned list will contain up to `len`
bytes; it may return fewer than requested, but not more.

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

- <a name="read.this">`this`</a>: [`input-stream`](#input_stream)
- <a name="read.len">`len`</a>: `u64`

##### Return values

- <a name="read.0"></a> result<(list<`u8`>, [`stream-status`](#stream_status)), [`stream-error`](#stream_error)>

#### <a name="blocking_read">`blocking-read: func`</a>

Read bytes from a stream, with blocking.

This is similar to `read`, except that it blocks until at least one
byte can be read.

##### Params

- <a name="blocking_read.this">`this`</a>: [`input-stream`](#input_stream)
- <a name="blocking_read.len">`len`</a>: `u64`

##### Return values

- <a name="blocking_read.0"></a> result<(list<`u8`>, [`stream-status`](#stream_status)), [`stream-error`](#stream_error)>

#### <a name="skip">`skip: func`</a>

Skip bytes from a stream.

This is similar to the `read` function, but avoids copying the
bytes into the instance.

Once a stream has reached the end, subsequent calls to read or
`skip` will always report end-of-stream rather than producing more
data.

This function returns the number of bytes skipped, along with a
`stream-status` indicating whether the end of the stream was
reached. The returned value will be at most `len`; it may be less.

##### Params

- <a name="skip.this">`this`</a>: [`input-stream`](#input_stream)
- <a name="skip.len">`len`</a>: `u64`

##### Return values

- <a name="skip.0"></a> result<(`u64`, [`stream-status`](#stream_status)), [`stream-error`](#stream_error)>

#### <a name="blocking_skip">`blocking-skip: func`</a>

Skip bytes from a stream, with blocking.

This is similar to `skip`, except that it blocks until at least one
byte can be consumed.

##### Params

- <a name="blocking_skip.this">`this`</a>: [`input-stream`](#input_stream)
- <a name="blocking_skip.len">`len`</a>: `u64`

##### Return values

- <a name="blocking_skip.0"></a> result<(`u64`, [`stream-status`](#stream_status)), [`stream-error`](#stream_error)>

#### <a name="subscribe_to_input_stream">`subscribe-to-input-stream: func`</a>

Create a `pollable` which will resolve once either the specified stream
has bytes available to read or the other end of the stream has been
closed.

##### Params

- <a name="subscribe_to_input_stream.this">`this`</a>: [`input-stream`](#input_stream)

##### Return values

- <a name="subscribe_to_input_stream.0"></a> [`pollable`](#pollable)

#### <a name="drop_input_stream">`drop-input-stream: func`</a>

Dispose of the specified `input-stream`, after which it may no longer
be used.

##### Params

- <a name="drop_input_stream.this">`this`</a>: [`input-stream`](#input_stream)

#### <a name="write">`write: func`</a>

Write bytes to a stream.

This function returns a `u64` indicating the number of bytes from
`buf` that were written; it may be less than the full list.

##### Params

- <a name="write.this">`this`</a>: [`output-stream`](#output_stream)
- <a name="write.buf">`buf`</a>: list<`u8`>

##### Return values

- <a name="write.0"></a> result<`u64`, [`stream-error`](#stream_error)>

#### <a name="blocking_write">`blocking-write: func`</a>

Write bytes to a stream, with blocking.

This is similar to `write`, except that it blocks until at least one
byte can be written.

##### Params

- <a name="blocking_write.this">`this`</a>: [`output-stream`](#output_stream)
- <a name="blocking_write.buf">`buf`</a>: list<`u8`>

##### Return values

- <a name="blocking_write.0"></a> result<`u64`, [`stream-error`](#stream_error)>

#### <a name="write_zeroes">`write-zeroes: func`</a>

Write multiple zero bytes to a stream.

This function returns a `u64` indicating the number of zero bytes
that were written; it may be less than `len`.

##### Params

- <a name="write_zeroes.this">`this`</a>: [`output-stream`](#output_stream)
- <a name="write_zeroes.len">`len`</a>: `u64`

##### Return values

- <a name="write_zeroes.0"></a> result<`u64`, [`stream-error`](#stream_error)>

#### <a name="blocking_write_zeroes">`blocking-write-zeroes: func`</a>

Write multiple zero bytes to a stream, with blocking.

This is similar to `write-zeroes`, except that it blocks until at least
one byte can be written.

##### Params

- <a name="blocking_write_zeroes.this">`this`</a>: [`output-stream`](#output_stream)
- <a name="blocking_write_zeroes.len">`len`</a>: `u64`

##### Return values

- <a name="blocking_write_zeroes.0"></a> result<`u64`, [`stream-error`](#stream_error)>

#### <a name="splice">`splice: func`</a>

Read from one stream and write to another.

This function returns the number of bytes transferred; it may be less
than `len`.

Unlike other I/O functions, this function blocks until all the data
read from the input stream has been written to the output stream.

##### Params

- <a name="splice.this">`this`</a>: [`output-stream`](#output_stream)
- <a name="splice.src">`src`</a>: [`input-stream`](#input_stream)
- <a name="splice.len">`len`</a>: `u64`

##### Return values

- <a name="splice.0"></a> result<(`u64`, [`stream-status`](#stream_status)), [`stream-error`](#stream_error)>

#### <a name="blocking_splice">`blocking-splice: func`</a>

Read from one stream and write to another, with blocking.

This is similar to `splice`, except that it blocks until at least
one byte can be read.

##### Params

- <a name="blocking_splice.this">`this`</a>: [`output-stream`](#output_stream)
- <a name="blocking_splice.src">`src`</a>: [`input-stream`](#input_stream)
- <a name="blocking_splice.len">`len`</a>: `u64`

##### Return values

- <a name="blocking_splice.0"></a> result<(`u64`, [`stream-status`](#stream_status)), [`stream-error`](#stream_error)>

#### <a name="forward">`forward: func`</a>

Forward the entire contents of an input stream to an output stream.

This function repeatedly reads from the input stream and writes
the data to the output stream, until the end of the input stream
is reached, or an error is encountered.

Unlike other I/O functions, this function blocks until the end
of the input stream is seen and all the data has been written to
the output stream.

This function returns the number of bytes transferred.

##### Params

- <a name="forward.this">`this`</a>: [`output-stream`](#output_stream)
- <a name="forward.src">`src`</a>: [`input-stream`](#input_stream)

##### Return values

- <a name="forward.0"></a> result<`u64`, [`stream-error`](#stream_error)>

#### <a name="subscribe_to_output_stream">`subscribe-to-output-stream: func`</a>

Create a `pollable` which will resolve once either the specified stream
is ready to accept bytes or the other end of the stream has been closed.

##### Params

- <a name="subscribe_to_output_stream.this">`this`</a>: [`output-stream`](#output_stream)

##### Return values

- <a name="subscribe_to_output_stream.0"></a> [`pollable`](#pollable)

#### <a name="drop_output_stream">`drop-output-stream: func`</a>

Dispose of the specified `output-stream`, after which it may no longer
be used.

##### Params

- <a name="drop_output_stream.this">`this`</a>: [`output-stream`](#output_stream)

## <a name="wasi:keyvalue_wasi_cloud_error">Import interface wasi:keyvalue/wasi-cloud-error</a>


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

## <a name="wasi:keyvalue_types">Import interface wasi:keyvalue/types</a>


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

#### <a name="key">`type key`</a>
`string`
<p>A key is a unique identifier for a value in a bucket. The key is used to
retrieve the value from the bucket.

#### <a name="keys">`type keys`</a>
[`keys`](#keys)
<p>A list of keys

#### <a name="outgoing_value">`type outgoing-value`</a>
`u32`
<p>A value is the data stored in a key-value pair. The value can be of any type
that can be represented in a byte array. It provides a way to write the value
to the output-stream defined in the `wasi-io` interface.

#### <a name="outgoing_value_body_async">`type outgoing-value-body-async`</a>
[`output-stream`](#output_stream)
<p>
#### <a name="outgoing_value_body_sync">`type outgoing-value-body-sync`</a>
[`outgoing-value-body-sync`](#outgoing_value_body_sync)
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

#### <a name="incoming_value_async_body">`type incoming-value-async-body`</a>
[`input-stream`](#input_stream)
<p>
#### <a name="incoming_value_sync_body">`type incoming-value-sync-body`</a>
[`incoming-value-sync-body`](#incoming_value_sync_body)
<p>
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

#### <a name="outgoing_value_write_body_async">`outgoing-value-write-body-async: func`</a>


##### Params

- <a name="outgoing_value_write_body_async.outgoing_value">`outgoing-value`</a>: [`outgoing-value`](#outgoing_value)

##### Return values

- <a name="outgoing_value_write_body_async.0"></a> result<[`outgoing-value-body-async`](#outgoing_value_body_async), [`error`](#error)>

#### <a name="outgoing_value_write_body_sync">`outgoing-value-write-body-sync: func`</a>


##### Params

- <a name="outgoing_value_write_body_sync.outgoing_value">`outgoing-value`</a>: [`outgoing-value`](#outgoing_value)
- <a name="outgoing_value_write_body_sync.value">`value`</a>: [`outgoing-value-body-sync`](#outgoing_value_body_sync)

##### Return values

- <a name="outgoing_value_write_body_sync.0"></a> result<_, [`error`](#error)>

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

## <a name="wasi:keyvalue_readwrite">Import interface wasi:keyvalue/readwrite</a>

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

##### Params

- <a name="exists.bucket">`bucket`</a>: [`bucket`](#bucket)
- <a name="exists.key">`key`</a>: [`key`](#key)

##### Return values

- <a name="exists.0"></a> result<`bool`, [`error`](#error)>

## <a name="wasi:keyvalue_atomic">Import interface wasi:keyvalue/atomic</a>

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

## <a name="wasi:keyvalue_batch">Import interface wasi:keyvalue/batch</a>

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
- <a name="set_many.key_values">`key-values`</a>: list<([`key`](#key), [`outgoing-value`](#outgoing_value))>

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

## <a name="wasi:keyvalue_handle_watch">Export interface wasi:keyvalue/handle-watch</a>

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

