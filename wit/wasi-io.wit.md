# WASI I/O

```wit
/// WASI I/O is an I/O abstraction API which is currently focused on providing
/// stream types.
///
/// In the future, the component model is expected to add built-in stream types;
/// when it does, they are expected to subsume this API.
default interface wasi-io {
```

## Imports
```wit
use pkg.wasi-poll.{pollable}
```

## `stream-error`
```wit
/// An error type returned from a stream operation. Currently this
/// doesn't provide any additional information.
record stream-error {}
```

## `input-stream`
```wit
/// An input bytestream. In the future, this will be replaced by handle
/// types.
///
/// This conceptually represents a `stream<u8, _>`. It's temporary
/// scaffolding until component-model's async features are ready.
///
/// `input-stream`s are *non-blocking* to the extent practical on underlying
/// platforms. I/O operations always return promptly; if fewer bytes are
/// promptly available than requested, they return the number of bytes promptly
/// available, which could even be zero. To wait for data to be available,
/// use the `subscribe-to-input-stream` function to obtain a `pollable` which
/// can be polled for using `wasi_poll`.
///
/// And at present, it is a `u32` instead of being an actual handle, until
/// the wit-bindgen implementation of handles and resources is ready.
// TODO(resource input-stream {)
type input-stream = u32
```

## `read`
```wit
    /// Read bytes from a stream.
    ///
    /// This function returns a list of bytes containing the data that was
    /// read, along with a bool indicating whether the end of the stream
    /// was reached. The returned list will contain up to `len` bytes; it
    /// may return fewer than requested, but not more.
    ///
    /// Once a stream has reached the end, subsequent calls to read or
    /// `skip` will always report end-of-stream rather than producing more
    /// data.
    ///
    /// If `len` is 0, it represents a request to read 0 bytes, which should
    /// always succeed, assuming the stream hasn't reached its end yet, and
    /// return an empty list.
    ///
    /// The len here is a `u64`, but some callees may not be able to allocate
    /// a buffer as large as that would imply.
    /// FIXME: describe what happens if allocation fails.
    read: func(
        this: input-stream,
        /// The maximum number of bytes to read
        len: u64
    ) -> result<tuple<list<u8>, bool>, stream-error>
```

## `skip`
```wit
    /// Skip bytes from a stream.
    ///
    /// This is similar to the `read` function, but avoids copying the
    /// bytes into the instance.
    ///
    /// Once a stream has reached the end, subsequent calls to read or
    /// `skip` will always report end-of-stream rather than producing more
    /// data.
    ///
    /// This function returns the number of bytes skipped, along with a bool
    /// indicating whether the end of the stream was reached. The returned
    /// value will be at most `len`; it may be less.
    skip: func(
        this: input-stream,
        /// The maximum number of bytes to skip.
        len: u64,
    ) -> result<tuple<u64, bool>, stream-error>
```

## `subscribe-to-input-stream`
```wit
/// Create a `pollable` which will resolve once either the specified stream has bytes
/// available to read or the other end of the stream has been closed.
subscribe-to-input-stream: func(this: input-stream) -> pollable
```

# `drop-input-stream`
```wit
/// Dispose of the specified `input-stream`, after which it may no longer
/// be used.
// TODO(} /* resource input-stream */)
drop-input-stream: func(this: input-stream)
```

## `output-stream`
```wit
/// An output bytestream. In the future, this will be replaced by handle
/// types.
///
/// This conceptually represents a `stream<u8, _>`. It's temporary
/// scaffolding until component-model's async features are ready.
///
/// `output-stream`s are *non-blocking* to the extent practical on
/// underlying platforms. Except where specified otherwise, I/O operations also
/// always return promptly, after the number of bytes that can be written
/// promptly, which could even be zero. To wait for the stream to be ready to
/// accept data, the `subscribe-to-output-stream` function to obtain a
/// `pollable` which can be polled for using `wasi_poll`.
///
/// And at present, it is a `u32` instead of being an actual handle, until
/// the wit-bindgen implementation of handles and resources is ready.
// TODO(resource output-stream {)
type output-stream = u32
```

## `write`
```wit
    /// Write bytes to a stream.
    ///
    /// This function returns a `u64` indicating the number of bytes from
    /// `buf` that were written; it may be less than the full list.
    write: func(
        this: output-stream,
        /// Data to write
        buf: list<u8>
    ) -> result<u64, stream-error>
```

## `write-zeroes`
```wit
    /// Write multiple zero bytes to a stream.
    ///
    /// This function returns a `u64` indicating the number of zero bytes
    /// that were written; it may be less than `len`.
    write-zeroes: func(
        this: output-stream,
        /// The number of zero bytes to write
        len: u64
    ) -> result<u64, stream-error>
```

## `splice`
```wit
    /// Read from one stream and write to another.
    ///
    /// This function returns the number of bytes transferred; it may be less
    /// than `len`.
    ///
    /// Unlike other I/O functions, this function blocks until all the data
    /// read from the input stream has been written to the output stream.
    splice: func(
        this: output-stream,
        /// The stream to read from
        src: input-stream,
        /// The number of bytes to splice
        len: u64,
    ) -> result<tuple<u64, bool>, stream-error>
```

## `forward`
```wit
    /// Forward the entire contents of an input stream to an output stream.
    ///
    /// This function repeatedly reads from the input stream and writes
    /// the data to the output stream, until the end of the input stream
    /// is reached, or an error is encountered.
    ///
    /// Unlike other I/O functions, this function blocks until the end
    /// of the input stream is seen and all the data has been written to
    /// the output stream.
    ///
    /// This function returns the number of bytes transferred.
    forward: func(
        this: output-stream,
        /// The stream to read from
        src: input-stream
    ) -> result<u64, stream-error>
```

# `subscribe-to-output-stream`
```wit
/// Create a `pollable` which will resolve once either the specified stream is ready
/// to accept bytes or the other end of the stream has been closed.
subscribe-to-output-stream: func(this: output-stream) -> pollable
```

# `drop-output-stream`
```wit
/// Dispose of the specified `output-stream`, after which it may no longer
/// be used.
// TODO(} /* resource output-stream */)
drop-output-stream: func(this: output-stream)
```

```wit
}
```
