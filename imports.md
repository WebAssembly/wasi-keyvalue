<h1><a name="imports">World imports</a></h1>
<p>The <code>wasi:keyvalue/imports</code> world provides common APIs for interacting
with key-value stores. Components targeting this world will be able to
do</p>
<ol>
<li>CRUD (create, read, update, delete) operations on key-value stores.</li>
<li>Atomic <a href="#increment"><code>increment</code></a> and CAS (compare-and-swap) operations.</li>
<li>Batch operations that can reduce the number of round trips to the network.</li>
</ol>
<ul>
<li>Imports:
<ul>
<li>interface <a href="#wasi:io_error_0.2.0"><code>wasi:io/error@0.2.0</code></a></li>
<li>interface <a href="#wasi:io_poll_0.2.0"><code>wasi:io/poll@0.2.0</code></a></li>
<li>interface <a href="#wasi:io_streams_0.2.0"><code>wasi:io/streams@0.2.0</code></a></li>
<li>interface <a href="#wasi:keyvalue_wasi_keyvalue_error_0.2.0_draft"><code>wasi:keyvalue/wasi-keyvalue-error@0.2.0-draft</code></a></li>
<li>interface <a href="#wasi:keyvalue_types_0.2.0_draft"><code>wasi:keyvalue/types@0.2.0-draft</code></a></li>
<li>interface <a href="#wasi:keyvalue_eventual_0.2.0_draft"><code>wasi:keyvalue/eventual@0.2.0-draft</code></a></li>
<li>interface <a href="#wasi:keyvalue_atomic_0.2.0_draft"><code>wasi:keyvalue/atomic@0.2.0-draft</code></a></li>
<li>interface <a href="#wasi:keyvalue_eventual_batch_0.2.0_draft"><code>wasi:keyvalue/eventual-batch@0.2.0-draft</code></a></li>
</ul>
</li>
</ul>
<h2><a name="wasi:io_error_0.2.0"></a>Import interface wasi:io/error@0.2.0</h2>
<hr />
<h3>Types</h3>
<h4><a name="error"></a><code>resource error</code></h4>
<p>A resource which represents some error information.</p>
<p>The only method provided by this resource is <code>to-debug-string</code>,
which provides some human-readable information about the error.</p>
<p>In the <code>wasi:io</code> package, this resource is returned through the
<code>wasi:io/streams/stream-error</code> type.</p>
<p>To provide more specific error information, other interfaces may
provide functions to further &quot;downcast&quot; this error into more specific
error information. For example, <a href="#error"><code>error</code></a>s returned in streams derived
from filesystem types to be described using the filesystem's own
error-code type, using the function
<code>wasi:filesystem/types/filesystem-error-code</code>, which takes a parameter
<code>borrow&lt;error&gt;</code> and returns
<code>option&lt;wasi:filesystem/types/error-code&gt;</code>.</p>
<h2>The set of functions which can &quot;downcast&quot; an <a href="#error"><code>error</code></a> into a more
concrete type is open.</h2>
<h3>Functions</h3>
<h4><a name="method_error.to_debug_string"></a><code>[method]error.to-debug-string: func</code></h4>
<p>Returns a string that is suitable to assist humans in debugging
this error.</p>
<p>WARNING: The returned string should not be consumed mechanically!
It may change across platforms, hosts, or other implementation
details. Parsing this string is a major platform-compatibility
hazard.</p>
<h5>Params</h5>
<ul>
<li><a name="method_error.to_debug_string.self"></a><code>self</code>: borrow&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_error.to_debug_string.0"></a> <code>string</code></li>
</ul>
<h2><a name="wasi:io_poll_0.2.0"></a>Import interface wasi:io/poll@0.2.0</h2>
<p>A poll API intended to let users wait for I/O events on multiple handles
at once.</p>
<hr />
<h3>Types</h3>
<h4><a name="pollable"></a><code>resource pollable</code></h4>
<h2><a href="#pollable"><code>pollable</code></a> represents a single I/O event which may be ready, or not.</h2>
<h3>Functions</h3>
<h4><a name="method_pollable.ready"></a><code>[method]pollable.ready: func</code></h4>
<p>Return the readiness of a pollable. This function never blocks.</p>
<p>Returns <code>true</code> when the pollable is ready, and <code>false</code> otherwise.</p>
<h5>Params</h5>
<ul>
<li><a name="method_pollable.ready.self"></a><code>self</code>: borrow&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_pollable.ready.0"></a> <code>bool</code></li>
</ul>
<h4><a name="method_pollable.block"></a><code>[method]pollable.block: func</code></h4>
<p><code>block</code> returns immediately if the pollable is ready, and otherwise
blocks until ready.</p>
<p>This function is equivalent to calling <code>poll.poll</code> on a list
containing only this pollable.</p>
<h5>Params</h5>
<ul>
<li><a name="method_pollable.block.self"></a><code>self</code>: borrow&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;</li>
</ul>
<h4><a name="poll"></a><code>poll: func</code></h4>
<p>Poll for completion on a set of pollables.</p>
<p>This function takes a list of pollables, which identify I/O sources of
interest, and waits until one or more of the events is ready for I/O.</p>
<p>The result <code>list&lt;u32&gt;</code> contains one or more indices of handles in the
argument list that is ready for I/O.</p>
<p>If the list contains more elements than can be indexed with a <code>u32</code>
value, this function traps.</p>
<p>A timeout can be implemented by adding a pollable from the
wasi-clocks API to the list.</p>
<p>This function does not return a <code>result</code>; polling in itself does not
do any I/O so it doesn't fail. If any of the I/O sources identified by
the pollables has an error, it is indicated by marking the source as
being reaedy for I/O.</p>
<h5>Params</h5>
<ul>
<li><a name="poll.in"></a><code>in</code>: list&lt;borrow&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="poll.0"></a> list&lt;<code>u32</code>&gt;</li>
</ul>
<h2><a name="wasi:io_streams_0.2.0"></a>Import interface wasi:io/streams@0.2.0</h2>
<p>WASI I/O is an I/O abstraction API which is currently focused on providing
stream types.</p>
<p>In the future, the component model is expected to add built-in stream types;
when it does, they are expected to subsume this API.</p>
<hr />
<h3>Types</h3>
<h4><a name="error"></a><code>type error</code></h4>
<p><a href="#error"><a href="#error"><code>error</code></a></a></p>
<p>
#### <a name="pollable"></a>`type pollable`
[`pollable`](#pollable)
<p>
#### <a name="stream_error"></a>`variant stream-error`
<p>An error for input-stream and output-stream operations.</p>
<h5>Variant Cases</h5>
<ul>
<li>
<p><a name="stream_error.last_operation_failed"></a><code>last-operation-failed</code>: own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;</p>
<p>The last operation (a write or flush) failed before completion.
<p>More information is available in the <a href="#error"><code>error</code></a> payload.</p>
</li>
<li>
<p><a name="stream_error.closed"></a><code>closed</code></p>
<p>The stream is closed: no more input will be accepted by the
stream. A closed output-stream will return this error on all
future operations.
</li>
</ul>
<h4><a name="input_stream"></a><code>resource input-stream</code></h4>
<p>An input bytestream.</p>
<p><a href="#input_stream"><code>input-stream</code></a>s are <em>non-blocking</em> to the extent practical on underlying
platforms. I/O operations always return promptly; if fewer bytes are
promptly available than requested, they return the number of bytes promptly
available, which could even be zero. To wait for data to be available,
use the <code>subscribe</code> function to obtain a <a href="#pollable"><code>pollable</code></a> which can be polled
for using <code>wasi:io/poll</code>.</p>
<h4><a name="output_stream"></a><code>resource output-stream</code></h4>
<p>An output bytestream.</p>
<h2><a href="#output_stream"><code>output-stream</code></a>s are <em>non-blocking</em> to the extent practical on
underlying platforms. Except where specified otherwise, I/O operations also
always return promptly, after the number of bytes that can be written
promptly, which could even be zero. To wait for the stream to be ready to
accept data, the <code>subscribe</code> function to obtain a <a href="#pollable"><code>pollable</code></a> which can be
polled for using <code>wasi:io/poll</code>.</h2>
<h3>Functions</h3>
<h4><a name="method_input_stream.read"></a><code>[method]input-stream.read: func</code></h4>
<p>Perform a non-blocking read from the stream.</p>
<p>When the source of a <code>read</code> is binary data, the bytes from the source
are returned verbatim. When the source of a <code>read</code> is known to the
implementation to be text, bytes containing the UTF-8 encoding of the
text are returned.</p>
<p>This function returns a list of bytes containing the read data,
when successful. The returned list will contain up to <code>len</code> bytes;
it may return fewer than requested, but not more. The list is
empty when no bytes are available for reading at this time. The
pollable given by <code>subscribe</code> will be ready when more bytes are
available.</p>
<p>This function fails with a <a href="#stream_error"><code>stream-error</code></a> when the operation
encounters an error, giving <code>last-operation-failed</code>, or when the
stream is closed, giving <code>closed</code>.</p>
<p>When the caller gives a <code>len</code> of 0, it represents a request to
read 0 bytes. If the stream is still open, this call should
succeed and return an empty list, or otherwise fail with <code>closed</code>.</p>
<p>The <code>len</code> parameter is a <code>u64</code>, which could represent a list of u8 which
is not possible to allocate in wasm32, or not desirable to allocate as
as a return value by the callee. The callee may return a list of bytes
less than <code>len</code> in size while more bytes are available for reading.</p>
<h5>Params</h5>
<ul>
<li><a name="method_input_stream.read.self"></a><code>self</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_input_stream.read.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_input_stream.read.0"></a> result&lt;list&lt;<code>u8</code>&gt;, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_input_stream.blocking_read"></a><code>[method]input-stream.blocking-read: func</code></h4>
<p>Read bytes from a stream, after blocking until at least one byte can
be read. Except for blocking, behavior is identical to <code>read</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_input_stream.blocking_read.self"></a><code>self</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_input_stream.blocking_read.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_input_stream.blocking_read.0"></a> result&lt;list&lt;<code>u8</code>&gt;, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_input_stream.skip"></a><code>[method]input-stream.skip: func</code></h4>
<p>Skip bytes from a stream. Returns number of bytes skipped.</p>
<p>Behaves identical to <code>read</code>, except instead of returning a list
of bytes, returns the number of bytes consumed from the stream.</p>
<h5>Params</h5>
<ul>
<li><a name="method_input_stream.skip.self"></a><code>self</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_input_stream.skip.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_input_stream.skip.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_input_stream.blocking_skip"></a><code>[method]input-stream.blocking-skip: func</code></h4>
<p>Skip bytes from a stream, after blocking until at least one byte
can be skipped. Except for blocking behavior, identical to <code>skip</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_input_stream.blocking_skip.self"></a><code>self</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_input_stream.blocking_skip.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_input_stream.blocking_skip.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_input_stream.subscribe"></a><code>[method]input-stream.subscribe: func</code></h4>
<p>Create a <a href="#pollable"><code>pollable</code></a> which will resolve once either the specified stream
has bytes available to read or the other end of the stream has been
closed.
The created <a href="#pollable"><code>pollable</code></a> is a child resource of the <a href="#input_stream"><code>input-stream</code></a>.
Implementations may trap if the <a href="#input_stream"><code>input-stream</code></a> is dropped before
all derived <a href="#pollable"><code>pollable</code></a>s created with this function are dropped.</p>
<h5>Params</h5>
<ul>
<li><a name="method_input_stream.subscribe.self"></a><code>self</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_input_stream.subscribe.0"></a> own&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream.check_write"></a><code>[method]output-stream.check-write: func</code></h4>
<p>Check readiness for writing. This function never blocks.</p>
<p>Returns the number of bytes permitted for the next call to <code>write</code>,
or an error. Calling <code>write</code> with more bytes than this function has
permitted will trap.</p>
<p>When this function returns 0 bytes, the <code>subscribe</code> pollable will
become ready when this function will report at least 1 byte, or an
error.</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream.check_write.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream.check_write.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream.write"></a><code>[method]output-stream.write: func</code></h4>
<p>Perform a write. This function never blocks.</p>
<p>When the destination of a <code>write</code> is binary data, the bytes from
<code>contents</code> are written verbatim. When the destination of a <code>write</code> is
known to the implementation to be text, the bytes of <code>contents</code> are
transcoded from UTF-8 into the encoding of the destination and then
written.</p>
<p>Precondition: check-write gave permit of Ok(n) and contents has a
length of less than or equal to n. Otherwise, this function will trap.</p>
<p>returns Err(closed) without writing if the stream has closed since
the last call to check-write provided a permit.</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream.write.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream.write.contents"></a><code>contents</code>: list&lt;<code>u8</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream.write.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream.blocking_write_and_flush"></a><code>[method]output-stream.blocking-write-and-flush: func</code></h4>
<p>Perform a write of up to 4096 bytes, and then flush the stream. Block
until all of these operations are complete, or an error occurs.</p>
<p>This is a convenience wrapper around the use of <code>check-write</code>,
<code>subscribe</code>, <code>write</code>, and <code>flush</code>, and is implemented with the
following pseudo-code:</p>
<pre><code class="language-text">let pollable = this.subscribe();
while !contents.is_empty() {
  // Wait for the stream to become writable
  pollable.block();
  let Ok(n) = this.check-write(); // eliding error handling
  let len = min(n, contents.len());
  let (chunk, rest) = contents.split_at(len);
  this.write(chunk  );            // eliding error handling
  contents = rest;
}
this.flush();
// Wait for completion of `flush`
pollable.block();
// Check for any errors that arose during `flush`
let _ = this.check-write();         // eliding error handling
</code></pre>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream.blocking_write_and_flush.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream.blocking_write_and_flush.contents"></a><code>contents</code>: list&lt;<code>u8</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream.blocking_write_and_flush.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream.flush"></a><code>[method]output-stream.flush: func</code></h4>
<p>Request to flush buffered output. This function never blocks.</p>
<p>This tells the output-stream that the caller intends any buffered
output to be flushed. the output which is expected to be flushed
is all that has been passed to <code>write</code> prior to this call.</p>
<p>Upon calling this function, the <a href="#output_stream"><code>output-stream</code></a> will not accept any
writes (<code>check-write</code> will return <code>ok(0)</code>) until the flush has
completed. The <code>subscribe</code> pollable will become ready when the
flush has completed and the stream can accept more writes.</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream.flush.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream.flush.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream.blocking_flush"></a><code>[method]output-stream.blocking-flush: func</code></h4>
<p>Request to flush buffered output, and block until flush completes
and stream is ready for writing again.</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream.blocking_flush.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream.blocking_flush.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream.subscribe"></a><code>[method]output-stream.subscribe: func</code></h4>
<p>Create a <a href="#pollable"><code>pollable</code></a> which will resolve once the output-stream
is ready for more writing, or an error has occured. When this
pollable is ready, <code>check-write</code> will return <code>ok(n)</code> with n&gt;0, or an
error.</p>
<p>If the stream is closed, this pollable is always ready immediately.</p>
<p>The created <a href="#pollable"><code>pollable</code></a> is a child resource of the <a href="#output_stream"><code>output-stream</code></a>.
Implementations may trap if the <a href="#output_stream"><code>output-stream</code></a> is dropped before
all derived <a href="#pollable"><code>pollable</code></a>s created with this function are dropped.</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream.subscribe.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream.subscribe.0"></a> own&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream.write_zeroes"></a><code>[method]output-stream.write-zeroes: func</code></h4>
<p>Write zeroes to a stream.</p>
<p>This should be used precisely like <code>write</code> with the exact same
preconditions (must use check-write first), but instead of
passing a list of bytes, you simply pass the number of zero-bytes
that should be written.</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream.write_zeroes.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream.write_zeroes.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream.write_zeroes.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream.blocking_write_zeroes_and_flush"></a><code>[method]output-stream.blocking-write-zeroes-and-flush: func</code></h4>
<p>Perform a write of up to 4096 zeroes, and then flush the stream.
Block until all of these operations are complete, or an error
occurs.</p>
<p>This is a convenience wrapper around the use of <code>check-write</code>,
<code>subscribe</code>, <code>write-zeroes</code>, and <code>flush</code>, and is implemented with
the following pseudo-code:</p>
<pre><code class="language-text">let pollable = this.subscribe();
while num_zeroes != 0 {
  // Wait for the stream to become writable
  pollable.block();
  let Ok(n) = this.check-write(); // eliding error handling
  let len = min(n, num_zeroes);
  this.write-zeroes(len);         // eliding error handling
  num_zeroes -= len;
}
this.flush();
// Wait for completion of `flush`
pollable.block();
// Check for any errors that arose during `flush`
let _ = this.check-write();         // eliding error handling
</code></pre>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream.blocking_write_zeroes_and_flush.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream.blocking_write_zeroes_and_flush.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream.blocking_write_zeroes_and_flush.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream.splice"></a><code>[method]output-stream.splice: func</code></h4>
<p>Read from one stream and write to another.</p>
<p>The behavior of splice is equivelant to:</p>
<ol>
<li>calling <code>check-write</code> on the <a href="#output_stream"><code>output-stream</code></a></li>
<li>calling <code>read</code> on the <a href="#input_stream"><code>input-stream</code></a> with the smaller of the
<code>check-write</code> permitted length and the <code>len</code> provided to <code>splice</code></li>
<li>calling <code>write</code> on the <a href="#output_stream"><code>output-stream</code></a> with that read data.</li>
</ol>
<p>Any error reported by the call to <code>check-write</code>, <code>read</code>, or
<code>write</code> ends the splice and reports that error.</p>
<p>This function returns the number of bytes transferred; it may be less
than <code>len</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream.splice.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream.splice.src"></a><code>src</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream.splice.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream.splice.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream.blocking_splice"></a><code>[method]output-stream.blocking-splice: func</code></h4>
<p>Read from one stream and write to another, with blocking.</p>
<p>This is similar to <code>splice</code>, except that it blocks until the
<a href="#output_stream"><code>output-stream</code></a> is ready for writing, and the <a href="#input_stream"><code>input-stream</code></a>
is ready for reading, before performing the <code>splice</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream.blocking_splice.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream.blocking_splice.src"></a><code>src</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream.blocking_splice.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream.blocking_splice.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h2><a name="wasi:keyvalue_wasi_keyvalue_error_0.2.0_draft"></a>Import interface wasi:keyvalue/wasi-keyvalue-error@0.2.0-draft</h2>
<hr />
<h3>Types</h3>
<h4><a name="error"></a><code>resource error</code></h4>
<p>An error resource type for keyvalue operations.</p>
<p>Common errors:</p>
<ul>
<li>Connectivity errors (e.g. network errors): when the client cannot establish
a connection to the keyvalue service.</li>
<li>Authentication and Authorization errors: when the client fails to authenticate
or does not have the required permissions to perform the operation.</li>
<li>Data errors: when the client sends incompatible or corrupted data.</li>
<li>Resource errors: when the system runs out of resources (e.g. memory).</li>
<li>Internal errors: unexpected errors on the server side.</li>
</ul>
<h2>Currently, this provides only one function to return a string representation
of the error. In the future, this will be extended to provide more information
about the error.
Soon: switch to <code>resource error { ... }</code></h2>
<h3>Functions</h3>
<h4><a name="method_error.trace"></a><code>[method]error.trace: func</code></h4>
<h5>Params</h5>
<ul>
<li><a name="method_error.trace.self"></a><code>self</code>: borrow&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_error.trace.0"></a> <code>string</code></li>
</ul>
<h2><a name="wasi:keyvalue_types_0.2.0_draft"></a>Import interface wasi:keyvalue/types@0.2.0-draft</h2>
<p>A generic keyvalue interface for WASI.</p>
<hr />
<h3>Types</h3>
<h4><a name="input_stream"></a><code>type input-stream</code></h4>
<p><a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a></p>
<p>
#### <a name="output_stream"></a>`type output-stream`
[`output-stream`](#output_stream)
<p>
#### <a name="error"></a>`type error`
[`error`](#error)
<p>
#### <a name="bucket"></a>`resource bucket`
<p>A bucket is a collection of key-value pairs. Each key-value pair is stored
as a entry in the bucket, and the bucket itself acts as a collection of all
these entries.</p>
<p>It is worth noting that the exact terminology for bucket in key-value stores
can very depending on the specific implementation. For example,</p>
<ol>
<li>Amazon DynamoDB calls a collection of key-value pairs a table</li>
<li>Redis has hashes, sets, and sorted sets as different types of collections</li>
<li>Cassandra calls a collection of key-value pairs a column family</li>
<li>MongoDB calls a collection of key-value pairs a collection</li>
<li>Riak calls a collection of key-value pairs a bucket</li>
<li>Memcached calls a collection of key-value pairs a slab</li>
<li>Azure Cosmos DB calls a collection of key-value pairs a container</li>
</ol>
<p>In this interface, we use the term <a href="#bucket"><code>bucket</code></a> to refer to a collection of key-value
Soon: switch to <code>resource bucket { ... }</code></p>
<h4><a name="key"></a><code>type key</code></h4>
<p><code>string</code></p>
<p>A key is a unique identifier for a value in a bucket. The key is used to
retrieve the value from the bucket.
<h4><a name="outgoing_value"></a><code>resource outgoing-value</code></h4>
<p>A value is the data stored in a key-value pair. The value can be of any type
that can be represented in a byte array. It provides a way to write the value
to the output-stream defined in the <code>wasi-io</code> interface.
Soon: switch to <code>resource value { ... }</code></p>
<h4><a name="outgoing_value_body_async"></a><code>type outgoing-value-body-async</code></h4>
<p><a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a></p>
<p>
#### <a name="outgoing_value_body_sync"></a>`type outgoing-value-body-sync`
[`outgoing-value-body-sync`](#outgoing_value_body_sync)
<p>
#### <a name="incoming_value"></a>`resource incoming-value`
<p>A incoming-value is a wrapper around a value. It provides a way to read the value
from the <a href="#input_stream"><code>input-stream</code></a> defined in the <code>wasi-io</code> interface.</p>
<p>The incoming-value provides two ways to consume the value:</p>
<ol>
<li><code>incoming-value-consume-sync</code> consumes the value synchronously and returns the
value as a <code>list&lt;u8&gt;</code>.</li>
<li><code>incoming-value-consume-async</code> consumes the value asynchronously and returns the
value as an <a href="#input_stream"><code>input-stream</code></a>.
In addition, it provides a <code>incoming-value-size</code> function to get the size of the value.
This is useful when the value is large and the caller wants to allocate a buffer of
the right size to consume the value.
Soon: switch to <code>resource incoming-value { ... }</code></li>
</ol>
<h4><a name="incoming_value_async_body"></a><code>type incoming-value-async-body</code></h4>
<p><a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a></p>
<p>
#### <a name="incoming_value_sync_body"></a>`type incoming-value-sync-body`
[`incoming-value-sync-body`](#incoming_value_sync_body)
<p>
----
<h3>Functions</h3>
<h4><a name="static_bucket.open_bucket"></a><code>[static]bucket.open-bucket: func</code></h4>
<p>Opens a bucket with the given name.</p>
<p>If any error occurs, including if the bucket does not exist, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="static_bucket.open_bucket.name"></a><code>name</code>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="static_bucket.open_bucket.0"></a> result&lt;own&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="static_outgoing_value.new_outgoing_value"></a><code>[static]outgoing-value.new-outgoing-value: func</code></h4>
<h5>Return values</h5>
<ul>
<li><a name="static_outgoing_value.new_outgoing_value.0"></a> own&lt;<a href="#outgoing_value"><a href="#outgoing_value"><code>outgoing-value</code></a></a>&gt;</li>
</ul>
<h4><a name="method_outgoing_value.outgoing_value_write_body_async"></a><code>[method]outgoing-value.outgoing-value-write-body-async: func</code></h4>
<p>Writes the value to the output-stream asynchronously.
If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_outgoing_value.outgoing_value_write_body_async.self"></a><code>self</code>: borrow&lt;<a href="#outgoing_value"><a href="#outgoing_value"><code>outgoing-value</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_outgoing_value.outgoing_value_write_body_async.0"></a> result&lt;own&lt;<a href="#outgoing_value_body_async"><a href="#outgoing_value_body_async"><code>outgoing-value-body-async</code></a></a>&gt;, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="method_outgoing_value.outgoing_value_write_body_sync"></a><code>[method]outgoing-value.outgoing-value-write-body-sync: func</code></h4>
<p>Writes the value to the output-stream synchronously.
If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_outgoing_value.outgoing_value_write_body_sync.self"></a><code>self</code>: borrow&lt;<a href="#outgoing_value"><a href="#outgoing_value"><code>outgoing-value</code></a></a>&gt;</li>
<li><a name="method_outgoing_value.outgoing_value_write_body_sync.value"></a><code>value</code>: <a href="#outgoing_value_body_sync"><a href="#outgoing_value_body_sync"><code>outgoing-value-body-sync</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_outgoing_value.outgoing_value_write_body_sync.0"></a> result&lt;_, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="static_incoming_value.incoming_value_consume_sync"></a><code>[static]incoming-value.incoming-value-consume-sync: func</code></h4>
<p>Consumes the value synchronously and returns the value as a list of bytes.
If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="static_incoming_value.incoming_value_consume_sync.this"></a><code>this</code>: own&lt;<a href="#incoming_value"><a href="#incoming_value"><code>incoming-value</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="static_incoming_value.incoming_value_consume_sync.0"></a> result&lt;<a href="#incoming_value_sync_body"><a href="#incoming_value_sync_body"><code>incoming-value-sync-body</code></a></a>, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="static_incoming_value.incoming_value_consume_async"></a><code>[static]incoming-value.incoming-value-consume-async: func</code></h4>
<p>Consumes the value asynchronously and returns the value as an <a href="#input_stream"><code>input-stream</code></a>.
If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="static_incoming_value.incoming_value_consume_async.this"></a><code>this</code>: own&lt;<a href="#incoming_value"><a href="#incoming_value"><code>incoming-value</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="static_incoming_value.incoming_value_consume_async.0"></a> result&lt;own&lt;<a href="#incoming_value_async_body"><a href="#incoming_value_async_body"><code>incoming-value-async-body</code></a></a>&gt;, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="method_incoming_value.incoming_value_size"></a><code>[method]incoming-value.incoming-value-size: func</code></h4>
<p>The size of the value in bytes.
If the size is unknown or unavailable, this function returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="method_incoming_value.incoming_value_size.self"></a><code>self</code>: borrow&lt;<a href="#incoming_value"><a href="#incoming_value"><code>incoming-value</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_incoming_value.incoming_value_size.0"></a> result&lt;<code>u64</code>, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h2><a name="wasi:keyvalue_eventual_0.2.0_draft"></a>Import interface wasi:keyvalue/eventual@0.2.0-draft</h2>
<p>A keyvalue interface that provides eventually consistent CRUD operations.</p>
<p>A CRUD operation is an operation that acts on a single key-value pair.</p>
<p>The value in the key-value pair is defined as a <code>u8</code> byte array and the intention
is that it is the common denominator for all data types defined by different
key-value stores to handle data, ensuring compatibility between different
key-value stores. Note: the clients will be expecting serialization/deserialization overhead
to be handled by the key-value store. The value could be a serialized object from
JSON, HTML or vendor-specific data types like AWS S3 objects.</p>
<p>Data consistency in a key value store refers to the gaurantee that once a
write operation completes, all subsequent read operations will return the
value that was written.</p>
<p>The level of consistency in readwrite interfaces is <strong>eventual consistency</strong>,
which means that if a write operation completes successfully, all subsequent
read operations will eventually return the value that was written. In other words,
if we pause the updates to the system, the system eventually will return
the last updated value for read.</p>
<hr />
<h3>Types</h3>
<h4><a name="bucket"></a><code>type bucket</code></h4>
<p><a href="#bucket"><a href="#bucket"><code>bucket</code></a></a></p>
<p>
#### <a name="error"></a>`type error`
[`error`](#error)
<p>
#### <a name="incoming_value"></a>`type incoming-value`
[`incoming-value`](#incoming_value)
<p>
#### <a name="key"></a>`type key`
[`key`](#key)
<p>
#### <a name="outgoing_value"></a>`type outgoing-value`
[`outgoing-value`](#outgoing_value)
<p>
----
<h3>Functions</h3>
<h4><a name="get"></a><code>get: func</code></h4>
<p>Get the value associated with the key in the bucket.</p>
<p>The value is returned as an option. If the key-value pair exists in the
bucket, it returns <code>Ok(value)</code>. If the key does not exist in the
bucket, it returns <code>Ok(none)</code>.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="get.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a name="get.key"></a><a href="#key"><code>key</code></a>: <a href="#key"><a href="#key"><code>key</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="get.0"></a> result&lt;option&lt;own&lt;<a href="#incoming_value"><a href="#incoming_value"><code>incoming-value</code></a></a>&gt;&gt;, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="set"></a><code>set: func</code></h4>
<p>Set the value associated with the key in the bucket. If the key already
exists in the bucket, it overwrites the value.</p>
<p>If the key does not exist in the bucket, it creates a new key-value pair.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="set.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a name="set.key"></a><a href="#key"><code>key</code></a>: <a href="#key"><a href="#key"><code>key</code></a></a></li>
<li><a name="set.outgoing_value"></a><a href="#outgoing_value"><code>outgoing-value</code></a>: borrow&lt;<a href="#outgoing_value"><a href="#outgoing_value"><code>outgoing-value</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="set.0"></a> result&lt;_, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="delete"></a><code>delete: func</code></h4>
<p>Delete the key-value pair associated with the key in the bucket.</p>
<p>If the key does not exist in the bucket, it does nothing.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="delete.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a name="delete.key"></a><a href="#key"><code>key</code></a>: <a href="#key"><a href="#key"><code>key</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="delete.0"></a> result&lt;_, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="exists"></a><code>exists: func</code></h4>
<p>Check if the key exists in the bucket.</p>
<p>If the key exists in the bucket, it returns <code>Ok(true)</code>. If the key does
not exist in the bucket, it returns <code>Ok(false)</code>.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="exists.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a name="exists.key"></a><a href="#key"><code>key</code></a>: <a href="#key"><a href="#key"><code>key</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="exists.0"></a> result&lt;<code>bool</code>, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h2><a name="wasi:keyvalue_atomic_0.2.0_draft"></a>Import interface wasi:keyvalue/atomic@0.2.0-draft</h2>
<p>A keyvalue interface that provides atomic operations.</p>
<p>Atomic operations are single, indivisible operations. When a fault causes
an atomic operation to fail, it will appear to the invoker of the atomic
operation that the action either completed successfully or did nothing
at all.</p>
<hr />
<h3>Types</h3>
<h4><a name="bucket"></a><code>type bucket</code></h4>
<p><a href="#bucket"><a href="#bucket"><code>bucket</code></a></a></p>
<p>
#### <a name="error"></a>`type error`
[`error`](#error)
<p>
#### <a name="key"></a>`type key`
[`key`](#key)
<p>
----
<h3>Functions</h3>
<h4><a name="increment"></a><code>increment: func</code></h4>
<p>Atomically increment the value associated with the key in the bucket by the
given delta. It returns the new value.</p>
<p>If the key does not exist in the bucket, it creates a new key-value pair
with the value set to the given delta.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="increment.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a name="increment.key"></a><a href="#key"><code>key</code></a>: <a href="#key"><a href="#key"><code>key</code></a></a></li>
<li><a name="increment.delta"></a><code>delta</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="increment.0"></a> result&lt;<code>u64</code>, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="compare_and_swap"></a><code>compare-and-swap: func</code></h4>
<p>Compare-and-swap (CAS) atomically updates the value associated with the key
in the bucket if the value matches the old value. This operation returns
<code>Ok(true)</code> if the swap was successful, <code>Ok(false)</code> if the value did not match,</p>
<p>A successful CAS operation means the current value matched the <code>old</code> value
and was replaced with the <code>new</code> value.</p>
<p>If the key does not exist in the bucket, it returns <code>Ok(false)</code>.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="compare_and_swap.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a name="compare_and_swap.key"></a><a href="#key"><code>key</code></a>: <a href="#key"><a href="#key"><code>key</code></a></a></li>
<li><a name="compare_and_swap.old"></a><code>old</code>: <code>u64</code></li>
<li><a name="compare_and_swap.new"></a><code>new</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="compare_and_swap.0"></a> result&lt;<code>bool</code>, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h2><a name="wasi:keyvalue_eventual_batch_0.2.0_draft"></a>Import interface wasi:keyvalue/eventual-batch@0.2.0-draft</h2>
<p>A keyvalue interface that provides eventually consistent batch operations.</p>
<p>A batch operation is an operation that operates on multiple keys at once.</p>
<p>Batch operations are useful for reducing network round-trip time. For example,
if you want to get the values associated with 100 keys, you can either do 100 get
operations or you can do 1 batch get operation. The batch operation is
faster because it only needs to make 1 network call instead of 100.</p>
<p>A batch operation does not guarantee atomicity, meaning that if the batch
operation fails, some of the keys may have been modified and some may not.
Transactional operations are being worked on and will be added in the future to
provide atomicity.</p>
<p>Data consistency in a key value store refers to the gaurantee that once a
write operation completes, all subsequent read operations will return the
value that was written.</p>
<p>The level of consistency in batch operations is <strong>eventual consistency</strong>, the same
with the readwrite interface. This interface does not guarantee strong consistency,
meaning that if a write operation completes, subsequent read operations may not return
the value that was written.</p>
<hr />
<h3>Types</h3>
<h4><a name="bucket"></a><code>type bucket</code></h4>
<p><a href="#bucket"><a href="#bucket"><code>bucket</code></a></a></p>
<p>
#### <a name="error"></a>`type error`
[`error`](#error)
<p>
#### <a name="key"></a>`type key`
[`key`](#key)
<p>
#### <a name="incoming_value"></a>`type incoming-value`
[`incoming-value`](#incoming_value)
<p>
#### <a name="outgoing_value"></a>`type outgoing-value`
[`outgoing-value`](#outgoing_value)
<p>
----
<h3>Functions</h3>
<h4><a name="get_many"></a><code>get-many: func</code></h4>
<p>Get the values associated with the keys in the bucket. It returns a list of
incoming-value that can be consumed to get the value associated with the key.</p>
<p>If any of the keys do not exist in the bucket, it returns a <code>none</code> value for
that key in the list.</p>
<p>Note that the key-value pairs are guaranteed to be returned in the same order</p>
<p>MAY show an out-of-date value if there are concurrent writes to the bucket.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="get_many.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a name="get_many.keys"></a><a href="#keys"><code>keys</code></a>: list&lt;<a href="#key"><a href="#key"><code>key</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="get_many.0"></a> result&lt;list&lt;option&lt;own&lt;<a href="#incoming_value"><a href="#incoming_value"><code>incoming-value</code></a></a>&gt;&gt;&gt;, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="keys"></a><code>keys: func</code></h4>
<p>Get all the keys in the bucket. It returns a list of keys.</p>
<p>Note that the keys are not guaranteed to be returned in any particular order.</p>
<p>If the bucket is empty, it returns an empty list.</p>
<p>MAY show an out-of-date list of keys if there are concurrent writes to the bucket.</p>
<p>If any error occurs, it returns an <code>Err(error)</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="keys.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="keys.0"></a> result&lt;list&lt;<a href="#key"><a href="#key"><code>key</code></a></a>&gt;, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="set_many"></a><code>set-many: func</code></h4>
<p>Set the values associated with the keys in the bucket. If the key already
exists in the bucket, it overwrites the value.</p>
<p>Note that the key-value pairs are not guaranteed to be set in the order
they are provided.</p>
<p>If any of the keys do not exist in the bucket, it creates a new key-value pair.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>. When an error occurs, it
does not rollback the key-value pairs that were already set. Thus, this batch operation
does not guarantee atomicity, implying that some key-value pairs could be
set while others might fail.</p>
<p>Other concurrent operations may also be able to see the partial results.</p>
<h5>Params</h5>
<ul>
<li><a name="set_many.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a name="set_many.key_values"></a><code>key-values</code>: list&lt;(<a href="#key"><a href="#key"><code>key</code></a></a>, borrow&lt;<a href="#outgoing_value"><a href="#outgoing_value"><code>outgoing-value</code></a></a>&gt;)&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="set_many.0"></a> result&lt;_, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
<h4><a name="delete_many"></a><code>delete-many: func</code></h4>
<p>Delete the key-value pairs associated with the keys in the bucket.</p>
<p>Note that the key-value pairs are not guaranteed to be deleted in the order
they are provided.</p>
<p>If any of the keys do not exist in the bucket, it skips the key.</p>
<p>If any other error occurs, it returns an <code>Err(error)</code>. When an error occurs, it
does not rollback the key-value pairs that were already deleted. Thus, this batch operation
does not guarantee atomicity, implying that some key-value pairs could be
deleted while others might fail.</p>
<p>Other concurrent operations may also be able to see the partial results.</p>
<h5>Params</h5>
<ul>
<li><a name="delete_many.bucket"></a><a href="#bucket"><code>bucket</code></a>: borrow&lt;<a href="#bucket"><a href="#bucket"><code>bucket</code></a></a>&gt;</li>
<li><a name="delete_many.keys"></a><a href="#keys"><code>keys</code></a>: list&lt;<a href="#key"><a href="#key"><code>key</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="delete_many.0"></a> result&lt;_, own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;&gt;</li>
</ul>
