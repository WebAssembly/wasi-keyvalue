<h1><a id="imports"></a>World imports</h1>
<p>The <code>wasi:keyvalue/imports</code> world provides common APIs for interacting with key-value stores.
Components targeting this world will be able to do:</p>
<ol>
<li>CRUD (create, read, update, delete) operations on key-value stores.</li>
<li>Atomic <a href="#increment"><code>increment</code></a> and CAS (compare-and-swap) operations.</li>
<li>Batch operations that can reduce the number of round trips to the network.</li>
</ol>
<ul>
<li>Imports:
<ul>
<li>interface <a href="#wasi_keyvalue_store_0_2_0_draft2"><code>wasi:keyvalue/store@0.2.0-draft2</code></a></li>
<li>interface <a href="#wasi_keyvalue_atomics_0_2_0_draft2"><code>wasi:keyvalue/atomics@0.2.0-draft2</code></a></li>
<li>interface <a href="#wasi_keyvalue_batch_0_2_0_draft2"><code>wasi:keyvalue/batch@0.2.0-draft2</code></a></li>
</ul>
</li>
</ul>
<h2><a id="wasi_keyvalue_store_0_2_0_draft2"></a>Import interface wasi:keyvalue/store@0.2.0-draft2</h2>
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
<h4><a id="error"></a><code>variant error</code></h4>
<p>The set of errors which may be raised by functions in this package</p>
<h5>Variant Cases</h5>
<ul>
<li>
<p><a id="error.no_such_store"></a><code>no-such-store</code></p>
<p>The host does not recognize the store identifier requested.
</li>
<li>
<p><a id="error.access_denied"></a><code>access-denied</code></p>
<p>The requesting component does not have access to the specified store
(which may or may not exist).
</li>
<li>
<p><a id="error.other"></a><code>other</code>: <code>string</code></p>
<p>Some implementation-specific error has occurred (e.g. I/O)
</li>
</ul>
<h4><a id="key_response"></a><code>record key-response</code></h4>
<p>A response to a <code>list-keys</code> operation.</p>
<h5>Record Fields</h5>
<ul>
<li>
<p><a id="key_response.keys"></a><code>keys</code>: list&lt;<code>string</code>&gt;</p>
<p>The list of keys returned by the query.
</li>
<li>
<p><a id="key_response.cursor"></a><code>cursor</code>: option&lt;<code>string</code>&gt;</p>
<p>The continuation token to use to fetch the next page of keys. If this is `null`, then
there are no more keys to fetch.
</li>
</ul>
<h4><a id="bucket"></a><code>resource bucket</code></h4>
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
<h2>In this interface, we use the term <a href="#bucket"><code>bucket</code></a> to refer to a collection of key-value pairs</h2>
<h3>Functions</h3>
<h4><a id="open"></a><code>open: func</code></h4>
<p>Get the bucket with the specified identifier.</p>
<p><code>identifier</code> must refer to a bucket provided by the host.</p>
<p><a href="#error.no_such_store"><code>error::no-such-store</code></a> will be raised if the <code>identifier</code> is not recognized.</p>
<h5>Params</h5>
<ul>
<li><a id="open.identifier"></a><code>identifier</code>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="open.0"></a> result&lt;own&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a id="method_bucket_get"></a><code>[method]bucket.get: func</code></h4>
<p>Get the value associated with the specified <code>key</code></p>
<p>The value is returned as an option. If the key-value pair exists in the
store, it returns <code>Ok(value)</code>. If the key does not exist in the
store, it returns <code>Ok(none)</code>.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a id="method_bucket_get.self"></a><code>self</code>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a id="method_bucket_get.key"></a><code>key</code>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="method_bucket_get.0"></a> result&lt;option&lt;list&lt;<code>u8</code>&gt;&gt;, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a id="method_bucket_set"></a><code>[method]bucket.set: func</code></h4>
<p>Set the value associated with the key in the store. If the key already
exists in the store, it overwrites the value.</p>
<p>If the key does not exist in the store, it creates a new key-value pair.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a id="method_bucket_set.self"></a><code>self</code>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a id="method_bucket_set.key"></a><code>key</code>: <code>string</code></li>
<li><a id="method_bucket_set.value"></a><code>value</code>: list&lt;<code>u8</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="method_bucket_set.0"></a> result&lt;_, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a id="method_bucket_delete"></a><code>[method]bucket.delete: func</code></h4>
<p>Delete the key-value pair associated with the key in the store.</p>
<p>If the key does not exist in the store, it does nothing.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a id="method_bucket_delete.self"></a><code>self</code>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a id="method_bucket_delete.key"></a><code>key</code>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="method_bucket_delete.0"></a> result&lt;_, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a id="method_bucket_exists"></a><code>[method]bucket.exists: func</code></h4>
<p>Check if the key exists in the store.</p>
<p>If the key exists in the store, it returns <code>Ok(true)</code>. If the key does
not exist in the store, it returns <code>Ok(false)</code>.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a id="method_bucket_exists.self"></a><code>self</code>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a id="method_bucket_exists.key"></a><code>key</code>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="method_bucket_exists.0"></a> result&lt;<code>bool</code>, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a id="method_bucket_list_keys"></a><code>[method]bucket.list-keys: func</code></h4>
<p>Get all the keys in the store with an optional cursor (for use in pagination). It
returns a list of keys. Please note that for most KeyValue implementations, this is a
can be a very expensive operation and so it should be used judiciously. Implementations
can return any number of keys in a single response, but they should never attempt to
send more data than is reasonable (i.e. on a small edge device, this may only be a few
KB, while on a large machine this could be several MB). Any response should also return
a cursor that can be used to fetch the next page of keys. See the <a href="#key_response"><code>key-response</code></a> record
for more information.</p>
<p>Note that the keys are not guaranteed to be returned in any particular order.</p>
<p>If the store is empty, it returns an empty list.</p>
<p>MAY show an out-of-date list of keys if there are concurrent writes to the store.</p>
<p>If any error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a id="method_bucket_list_keys.self"></a><code>self</code>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a id="method_bucket_list_keys.cursor"></a><code>cursor</code>: option&lt;<code>string</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="method_bucket_list_keys.0"></a> result&lt;<a href="#key_response"><a href="#key_response"><code>key-response</code></a></a>, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h2><a id="wasi_keyvalue_atomics_0_2_0_draft2"></a>Import interface wasi:keyvalue/atomics@0.2.0-draft2</h2>
<p>A keyvalue interface that provides atomic operations.</p>
<p>Atomic operations are single, indivisible operations. When a fault causes an atomic operation to
fail, it will appear to the invoker of the atomic operation that the action either completed
successfully or did nothing at all.</p>
<p>Please note that this interface is bare functions that take a reference to a bucket. This is to
get around the current lack of a way to &quot;extend&quot; a resource with additional methods inside of
wit. Future version of the interface will instead extend these methods on the base <a href="#bucket"><code>bucket</code></a>
resource.</p>
<hr />
<h3>Types</h3>
<h4><a id="bucket"></a><code>type bucket</code></h4>
<p><a href="#bucket"><a href="#bucket"><code>bucket</code></a></a></p>
<p>
#### <a id="error"></a>`type error`
[`error`](#error)
<p>
#### <a id="cas"></a>`resource cas`
<p>A handle to a CAS (compare-and-swap) operation.</p>
<h4><a id="cas_error"></a><code>variant cas-error</code></h4>
<p>The error returned by a CAS operation</p>
<h5>Variant Cases</h5>
<ul>
<li>
<p><a id="cas_error.store_error"></a><code>store-error</code>: <a href="#error"><a href="#error"><code>error</code></a></a></p>
<p>A store error occurred when performing the operation
</li>
<li>
<p><a id="cas_error.cas_failed"></a><code>cas-failed</code>: own&lt;<a href="#cas"><a href="#cas"><code>cas</code></a></a>&gt;</p>
<p>The CAS operation failed because the value was too old. This returns a new CAS handle
for easy retries. Implementors MUST return a CAS handle that has been updated to the
latest version or transaction.
</li>
</ul>
<hr />
<h3>Functions</h3>
<h4><a id="static_cas_new"></a><code>[static]cas.new: func</code></h4>
<p>Construct a new CAS operation. Implementors can map the underlying functionality
(transactions, versions, etc) as desired.</p>
<h5>Params</h5>
<ul>
<li><a id="static_cas_new.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a id="static_cas_new.key"></a><code>key</code>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="static_cas_new.0"></a> result&lt;own&lt;<a href="#cas"><a href="#cas"><code>cas</code></a></a>&gt;, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a id="method_cas_current"></a><code>[method]cas.current: func</code></h4>
<p>Get the current value of the key (if it exists). This allows for avoiding reads if all
that is needed to ensure the atomicity of the operation</p>
<h5>Params</h5>
<ul>
<li><a id="method_cas_current.self"></a><code>self</code>: borrow&lt;<a href="#cas"><a href="#cas"><code>cas</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="method_cas_current.0"></a> result&lt;option&lt;list&lt;<code>u8</code>&gt;&gt;, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a id="increment"></a><code>increment: func</code></h4>
<p>Atomically increment the value associated with the key in the store by the given delta. It
returns the new value.</p>
<p>If the key does not exist in the store, it creates a new key-value pair with the value set
to the given delta.</p>
<p>The result of using both <a href="#increment"><code>increment</code></a> and any other operation in this spec with the same key
is implementation-dependent.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a id="increment.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a id="increment.key"></a><code>key</code>: <code>string</code></li>
<li><a id="increment.delta"></a><code>delta</code>: <code>s64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="increment.0"></a> result&lt;<code>s64</code>, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a id="swap"></a><code>swap: func</code></h4>
<p>Perform the swap on a CAS operation. This consumes the CAS handle and returns an error if
the CAS operation failed.</p>
<h5>Params</h5>
<ul>
<li><a id="swap.cas"></a><a href="#cas"><code>cas</code></a>: own&lt;<a href="#cas"><a href="#cas"><code>cas</code></a></a>&gt;</li>
<li><a id="swap.value"></a><code>value</code>: list&lt;<code>u8</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="swap.0"></a> result&lt;_, <a href="#cas_error"><a href="#cas_error"><code>cas-error</code></a></a>&gt;</li>
</ul>
<h2><a id="wasi_keyvalue_batch_0_2_0_draft2"></a>Import interface wasi:keyvalue/batch@0.2.0-draft2</h2>
<p>A keyvalue interface that provides batch operations.</p>
<p>A batch operation is an operation that operates on multiple keys at once.</p>
<p>Batch operations are useful for reducing network round-trip time. For example, if you want to
get the values associated with 100 keys, you can either do 100 get operations or you can do 1
batch get operation. The batch operation is faster because it only needs to make 1 network call
instead of 100.</p>
<p>A batch operation does not guarantee atomicity, meaning that if the batch operation fails, some
of the keys may have been modified and some may not.</p>
<p>This interface does has the same consistency guarantees as the <code>store</code> interface, meaning that
you should be able to &quot;read your writes.&quot;</p>
<p>Please note that this interface is bare functions that take a reference to a bucket. This is to
get around the current lack of a way to &quot;extend&quot; a resource with additional methods inside of
wit. Future version of the interface will instead extend these methods on the base <a href="#bucket"><code>bucket</code></a>
resource.</p>
<hr />
<h3>Types</h3>
<h4><a id="bucket"></a><code>type bucket</code></h4>
<p><a href="#bucket"><a href="#bucket"><code>bucket</code></a></a></p>
<p>
#### <a id="error"></a>`type error`
[`error`](#error)
<p>
----
<h3>Functions</h3>
<h4><a id="get_many"></a><code>get-many: func</code></h4>
<p>Get the key-value pairs associated with the keys in the store. It returns a list of
key-value pairs.</p>
<p>If any of the keys do not exist in the store, it returns a <code>none</code> value for that pair in the
list.</p>
<p>MAY show an out-of-date value if there are concurrent writes to the store.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a id="get_many.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a id="get_many.keys"></a><code>keys</code>: list&lt;<code>string</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="get_many.0"></a> result&lt;list&lt;option&lt;(<code>string</code>, list&lt;<code>u8</code>&gt;)&gt;&gt;, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a id="set_many"></a><code>set-many: func</code></h4>
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
<li><a id="set_many.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a id="set_many.key_values"></a><code>key-values</code>: list&lt;(<code>string</code>, list&lt;<code>u8</code>&gt;)&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="set_many.0"></a> result&lt;_, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h4><a id="delete_many"></a><code>delete-many: func</code></h4>
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
<li><a id="delete_many.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a id="delete_many.keys"></a><code>keys</code>: list&lt;<code>string</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a id="delete_many.0"></a> result&lt;_, <a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
