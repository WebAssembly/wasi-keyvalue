<h1><a name="imports">World imports</a></h1>
<p>The <code>wasi:keyvalue/imports</code> world provides common APIs for interacting with key-value stores.
Components targeting this world will be able to do:</p>
<ol>
<li>CRUD (create, read, update, delete) operations on key-value stores.</li>
<li>Atomic <code>increment</code> and CAS (compare-and-swap) operations.</li>
<li>Batch operations that can reduce the number of round trips to the network.</li>
</ol>
<ul>
<li>Imports:
<ul>
<li>interface <a href="#wasi:keyvalue_types_0.2.0_draft"><code>wasi:keyvalue/types@0.2.0-draft</code></a></li>
<li>interface <a href="#wasi:keyvalue_store_0.2.0_draft"><code>wasi:keyvalue/store@0.2.0-draft</code></a></li>
<li>interface <a href="#wasi:keyvalue_atomics_0.2.0_draft"><code>wasi:keyvalue/atomics@0.2.0-draft</code></a></li>
<li>interface <a href="#wasi:keyvalue_batch_0.2.0_draft"><code>wasi:keyvalue/batch@0.2.0-draft</code></a></li>
</ul>
</li>
</ul>
<h2><a name="wasi:keyvalue_types_0.2.0_draft"></a>Import interface wasi:keyvalue/types@0.2.0-draft</h2>
<p>A generic keyvalue interface for WASI.</p>
<hr />
<h3>Types</h3>
<h4><a name="bucket"></a><code>resource bucket</code></h4>
<p>A bucket is a collection of key-value pairs. Each key-value pair is stored as a entry in the
bucket, and the bucket itself acts as a collection of all these entries.</p>
<p>It is worth noting that the exact terminology for bucket in key-value stores can very
depending on the specific implementation. For example:</p>
<ol>
<li>Amazon DynamoDB calls a collection of key-value pairs a table</li>
<li>Redis has hashes, sets, and sorted sets as different types of collections</li>
<li>Cassandra calls a collection of key-value pairs a column family</li>
<li>MongoDB calls a collection of key-value pairs a collection</li>
<li>Riak calls a collection of key-value pairs a bucket</li>
<li>Memcached calls a collection of key-value pairs a slab</li>
<li>Azure Cosmos DB calls a collection of key-value pairs a container</li>
</ol>
<p>In this interface, we use the term <a href="#bucket"><code>bucket</code></a> to refer to a collection of key-value pairs</p>
<h4><a name="error"></a><code>variant error</code></h4>
<p>The set of errors which may be raised by functions in this interface</p>
<h5>Variant Cases</h5>
<ul>
<li>
<p><a name="error.no_such_store"></a><code>no-such-store</code></p>
<p>The host does not recognize the store identifier requested.
</li>
<li>
<p><a name="error.access_denied"></a><code>access-denied</code></p>
<p>The requesting component does not have access to the specified store
(which may or may not exist).
</li>
<li>
<p><a name="error.other"></a><code>other</code>: <code>string</code></p>
<p>Some implementation-specific error has occurred (e.g. I/O)
</li>
</ul>
<hr />
<h3>Functions</h3>
<h4><a name="static_bucket.get"></a><code>[static]bucket.get: func</code></h4>
<p>Get the store with the specified identifier.</p>
<p><code>identifier</code> must refer to a store provided by the host.</p>
<p><a href="#error.no_such_store"><code>error::no-such-store</code></a> will be raised if the <code>identifier</code> is not recognized.</p>
<h5>Params</h5>
<ul>
<li><a name="static_bucket.get.identifier"></a><code>identifier</code>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="static_bucket.get.0"></a> result&lt;own&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h2><a name="wasi:keyvalue_store_0.2.0_draft"></a>Import interface wasi:keyvalue/store@0.2.0-draft</h2>
<p>A keyvalue interface that provides eventually consistent key-value operations.</p>
<p>Each of these operations acts on a single key-value pair.</p>
<p>The value in the key-value pair is defined as a <code>u8</code> byte array and the intention is that it is
the common denominator for all data types defined by different key-value stores to handle data,
ensuring compatibility between different key-value stores. Note: the clients will be expecting
serialization/deserialization overhead to be handled by the key-value store. The value could be
a serialized object from JSON, HTML or vendor-specific data types like AWS S3 objects.</p>
<p>Data consistency in a key value store refers to the guarantee that once a write operation
completes, all subsequent read operations will return the value that was written.</p>
<p>Any implementation of this interface must have enough consistency to guarantee &quot;reading your
writes.&quot; In particular, this means that the client should never get a value that is older than
the one it wrote, but it MAY get a newer value if one was written around the same time. These
guarantees only apply to the same client (which will likely be provided by the host or an
external capability of some kind). In this context a &quot;client&quot; is referring to the caller or
guest that is consuming this interface. Once a write request is committed by a specific client,
all subsequent read requests by the same client will reflect that write or any subsequent
writes. Another client running in a different context may or may not immediately see the result
due to the replication lag. As an example of all of this, if a value at a given key is A, and
the client writes B, then immediately reads, it should get B. If something else writes C in
quick succession, then the client may get C. However, a client running in a separate context may
still see A or B</p>
<hr />
<h3>Types</h3>
<h4><a name="bucket"></a><code>type bucket</code></h4>
<p><a href="#bucket"><a href="#bucket"><code>bucket</code></a></a></p>
<p>
#### <a name="error"></a>`type error`
[`error`](#error)
<p>
#### <a name="store"></a>`resource store`
<h2>The crud resource contains the common create, read, update, and delete operations for
key-value pairs in a key-value store</h2>
<h3>Functions</h3>
<h4><a name="static_store.open"></a><code>[static]store.open: func</code></h4>
<p>Create a new crud resource, using the given bucket to access the data</p>
<h5>Params</h5>
<ul>
<li><a name="static_store.open.store"></a><a href="#store"><code>store</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="static_store.open.0"></a> own&lt;<a href="#store"><a href="#store"><code>store</code></a></a>&gt;</li>
</ul>
<h4><a name="method_store.get"></a><code>[method]store.get: func</code></h4>
<p>Get the value associated with the specified <code>key</code></p>
<p>The value is returned as an option. If the key-value pair exists in the
store, it returns <code>Ok(value)</code>. If the key does not exist in the
store, it returns <code>Ok(none)</code>.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_store.get.self"></a><code>self</code>: borrow&lt;<a href="#store"><a href="#store"><code>store</code></a></a>&gt;</li>
<li><a name="method_store.get.key"></a><code>key</code>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_store.get.0"></a> result&lt;option&lt;list&lt;<code>u8</code>&gt;&gt;, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_store.set"></a><code>[method]store.set: func</code></h4>
<p>Set the value associated with the key in the store. If the key already
exists in the store, it overwrites the value.</p>
<p>If the key does not exist in the store, it creates a new key-value pair.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_store.set.self"></a><code>self</code>: borrow&lt;<a href="#store"><a href="#store"><code>store</code></a></a>&gt;</li>
<li><a name="method_store.set.key"></a><code>key</code>: <code>string</code></li>
<li><a name="method_store.set.value"></a><code>value</code>: list&lt;<code>u8</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_store.set.0"></a> result&lt;_, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_store.delete"></a><code>[method]store.delete: func</code></h4>
<p>Delete the key-value pair associated with the key in the store.</p>
<p>If the key does not exist in the store, it does nothing.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_store.delete.self"></a><code>self</code>: borrow&lt;<a href="#store"><a href="#store"><code>store</code></a></a>&gt;</li>
<li><a name="method_store.delete.key"></a><code>key</code>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_store.delete.0"></a> result&lt;_, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_store.exists"></a><code>[method]store.exists: func</code></h4>
<p>Check if the key exists in the store.</p>
<p>If the key exists in the store, it returns <code>Ok(true)</code>. If the key does
not exist in the store, it returns <code>Ok(false)</code>.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_store.exists.self"></a><code>self</code>: borrow&lt;<a href="#store"><a href="#store"><code>store</code></a></a>&gt;</li>
<li><a name="method_store.exists.key"></a><code>key</code>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_store.exists.0"></a> result&lt;<code>bool</code>, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_store.list_keys"></a><code>[method]store.list-keys: func</code></h4>
<p>Get all the keys in the store. It returns a list of keys.</p>
<p>Note that the keys are not guaranteed to be returned in any particular order.</p>
<p>If the store is empty, it returns an empty list.</p>
<p>MAY show an out-of-date list of keys if there are concurrent writes to the store.</p>
<p>If any error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_store.list_keys.self"></a><code>self</code>: borrow&lt;<a href="#store"><a href="#store"><code>store</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_store.list_keys.0"></a> result&lt;list&lt;<code>string</code>&gt;, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h2><a name="wasi:keyvalue_atomics_0.2.0_draft"></a>Import interface wasi:keyvalue/atomics@0.2.0-draft</h2>
<p>A keyvalue interface that provides atomic operations.</p>
<p>Atomic operations are single, indivisible operations. When a fault causes an atomic operation to
fail, it will appear to the invoker of the atomic operation that the action either completed
successfully or did nothing at all.</p>
<hr />
<h3>Types</h3>
<h4><a name="bucket"></a><code>type bucket</code></h4>
<p><a href="#bucket"><a href="#bucket"><code>bucket</code></a></a></p>
<p>
#### <a name="error"></a>`type error`
[`error`](#error)
<p>
#### <a name="atomics"></a>`resource atomics`
<h2>The atomics resource contains atomic operations that can be executed against a store</h2>
<h3>Functions</h3>
<h4><a name="static_atomics.open"></a><code>[static]atomics.open: func</code></h4>
<p>Create a new atomic resource, using the given bucket to access the data</p>
<h5>Params</h5>
<ul>
<li><a name="static_atomics.open.store"></a><a href="#store"><code>store</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="static_atomics.open.0"></a> own&lt;<a href="#atomics"><a href="#atomics"><code>atomics</code></a></a>&gt;</li>
</ul>
<h4><a name="method_atomics.increment"></a><code>[method]atomics.increment: func</code></h4>
<p>Atomically increment the value associated with the key in the store by the given delta. It
returns the new value.</p>
<p>If the key does not exist in the store, it creates a new key-value pair with the value set
to the given delta.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_atomics.increment.self"></a><code>self</code>: borrow&lt;<a href="#atomics"><a href="#atomics"><code>atomics</code></a></a>&gt;</li>
<li><a name="method_atomics.increment.key"></a><code>key</code>: <code>string</code></li>
<li><a name="method_atomics.increment.delta"></a><code>delta</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_atomics.increment.0"></a> result&lt;<code>u64</code>, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h2><a name="wasi:keyvalue_batch_0.2.0_draft"></a>Import interface wasi:keyvalue/batch@0.2.0-draft</h2>
<p>A keyvalue interface that provides batch operations.</p>
<p>A batch operation is an operation that operates on multiple keys at once.</p>
<p>Batch operations are useful for reducing network round-trip time. For example, if you want to
get the values associated with 100 keys, you can either do 100 get operations or you can do 1
batch get operation. The batch operation is faster because it only needs to make 1 network call
instead of 100.</p>
<p>A batch operation does not guarantee atomicity, meaning that if the batch operation fails, some
of the keys may have been modified and some may not.</p>
<p>This interface does has the same consistency guarantees as the <a href="#store"><code>store</code></a> interface, meaning that
you should be able to &quot;read your writes.&quot;</p>
<hr />
<h3>Types</h3>
<h4><a name="bucket"></a><code>type bucket</code></h4>
<p><a href="#bucket"><a href="#bucket"><code>bucket</code></a></a></p>
<p>
#### <a name="error"></a>`type error`
[`error`](#error)
<p>
#### <a name="batch"></a>`resource batch`
<h2>The batch resource contains functions for fetching or updating many keys at once.</h2>
<h3>Functions</h3>
<h4><a name="static_batch.open"></a><code>[static]batch.open: func</code></h4>
<p>Create a new batch resource, using the given bucket to access the data</p>
<h5>Params</h5>
<ul>
<li><a name="static_batch.open.store"></a><a href="#store"><code>store</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="static_batch.open.0"></a> own&lt;<a href="#batch"><a href="#batch"><code>batch</code></a></a>&gt;</li>
</ul>
<h4><a name="method_batch.get_many"></a><code>[method]batch.get-many: func</code></h4>
<p>Get the key-value pairs associated with the keys in the store. It returns a list of
key-value pairs.</p>
<p>If any of the keys do not exist in the store, it returns a <code>none</code> value for that pair in the
list.</p>
<p>MAY show an out-of-date value if there are concurrent writes to the store.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_batch.get_many.self"></a><code>self</code>: borrow&lt;<a href="#batch"><a href="#batch"><code>batch</code></a></a>&gt;</li>
<li><a name="method_batch.get_many.keys"></a><code>keys</code>: list&lt;<code>string</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_batch.get_many.0"></a> result&lt;list&lt;option&lt;(<code>string</code>, list&lt;<code>u8</code>&gt;)&gt;&gt;, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_batch.set_many"></a><code>[method]batch.set-many: func</code></h4>
<p>Set the values associated with the keys in the store. If the key already exists in the
store, it overwrites the value.</p>
<p>Note that the key-value pairs are not guaranteed to be set in the order they are provided.</p>
<p>If any of the keys do not exist in the store, it creates a new key-value pair.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>. When an error occurs, it does not
rollback the key-value pairs that were already set. Thus, this batch operation does not
guarantee atomicity, implying that some key-value pairs could be set while others might
fail.</p>
<p>Other concurrent operations may also be able to see the partial results.</p>
<h5>Params</h5>
<ul>
<li><a name="method_batch.set_many.self"></a><code>self</code>: borrow&lt;<a href="#batch"><a href="#batch"><code>batch</code></a></a>&gt;</li>
<li><a name="method_batch.set_many.key_values"></a><code>key-values</code>: list&lt;(<code>string</code>, list&lt;<code>u8</code>&gt;)&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_batch.set_many.0"></a> result&lt;_, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_batch.delete_many"></a><code>[method]batch.delete-many: func</code></h4>
<p>Delete the key-value pairs associated with the keys in the store.</p>
<p>Note that the key-value pairs are not guaranteed to be deleted in the order they are
provided.</p>
<p>If any of the keys do not exist in the store, it skips the key.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>. When an error occurs, it does not
rollback the key-value pairs that were already deleted. Thus, this batch operation does not
guarantee atomicity, implying that some key-value pairs could be deleted while others might
fail.</p>
<p>Other concurrent operations may also be able to see the partial results.</p>
<h5>Params</h5>
<ul>
<li><a name="method_batch.delete_many.self"></a><code>self</code>: borrow&lt;<a href="#batch"><a href="#batch"><code>batch</code></a></a>&gt;</li>
<li><a name="method_batch.delete_many.keys"></a><code>keys</code>: list&lt;<code>string</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_batch.delete_many.0"></a> result&lt;_, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
