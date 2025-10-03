
# Reading and Writing Bytes

Now for the good stuff. We've got a connection—let's actually communicate! Reading and writing with `TcpStream` is refreshingly straightforward. If you've ever worked with files in Rust, this will feel familiar.

## The Read and Write Traits

A `TcpStream` implements both the `Read` and `Write` traits from `std::io`. This means you get access to methods like:

- `read()` - Read bytes into a buffer
- `read_exact()` - Read exactly n bytes (or error)
- `write()` - Write bytes from a buffer
- `write_all()` - Write all bytes (or error)

These are the same traits that files, stdin/stdout, and other I/O types use. One of Rust's nice design decisions—network I/O and file I/O have the same interface.

## Reading From a Stream

To read data, you need a buffer—a chunk of memory where the incoming bytes will be stored. Think of it like a notepad where you write down what someone's saying.

```rust
use std::io::Read;

let mut stream = /* ... */;
let mut buffer = [0; 512];  // 512-byte buffer

let bytes_read = stream.read(&mut buffer)?;
println!("Read {} bytes", bytes_read);
```

The `read()` method returns the number of bytes actually read. This is important: it might read fewer bytes than your buffer can hold. If the client only sends 10 bytes, `bytes_read` will be 10, even if your buffer is 512 bytes.

### What Does read() Return?

- **Some positive number** - That many bytes were read successfully
- **0** - The connection was closed by the client (EOF)
- **Error** - Something went wrong (network error, connection reset, etc.)

The zero case is crucial. When `read()` returns 0, it means the client has closed their end of the connection. You should stop reading and close your end too.

```rust
let bytes_read = stream.read(&mut buffer)?;

if bytes_read == 0 {
    println!("Client closed connection");
    return Ok(());
}
```

## Writing to a Stream

Writing is even simpler:

```rust
use std::io::Write;

let mut stream = /* ... */;
stream.write_all(b"Hello, client!")?;
```

The `b"..."` prefix creates a byte slice (`&[u8]`), which is what `write_all()` expects.

### write() vs write_all()

There are two writing methods:

- **write()** - Writes *some* bytes, returns how many
- **write_all()** - Writes *all* bytes, or returns an error

For most cases, you want `write_all()`. It handles the case where the OS can't write all your data at once (rare, but possible). The `write()` method is lower-level and requires you to handle partial writes yourself.

```rust
// This might only write part of the message
let bytes_written = stream.write(b"Hello, client!")?;

// This guarantees the whole message is sent (or errors)
stream.write_all(b"Hello, client!")?;
```

For an echo server, `write_all()` is what we want.

## Building the Echo Logic

Let's put it together. The echo logic is simple:

1. Read data from the client
2. Write that same data back
3. Repeat until the client closes the connection

Here's the core loop:

```rust
use std::net::TcpListener;
use std::io::{Read, Write};

fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Echo server listening on port 8080");
    
    for stream in listener.incoming() {
        let mut stream = stream?;
        let addr = stream.peer_addr()?;
        println!("[{}] Connection established", addr);
        
        let mut buffer = [0; 512];
        
        loop {
            // Read from the client
            let bytes_read = stream.read(&mut buffer)?;
            
            // If read returns 0, client closed the connection
            if bytes_read == 0 {
                println!("[{}] Connection closed", addr);
                break;
            }
            
            println!("[{}] Received {} bytes", addr, bytes_read);
            
            // Echo it back
            stream.write_all(&buffer[..bytes_read])?;
        }
    }
    
    Ok(())
}
```

That's it! A fully functional echo server in about 30 lines of code.

## Understanding the Buffer Slice

Look at this line carefully:

```rust
stream.write_all(&buffer[..bytes_read])?;
```

We're not writing the entire buffer—we're writing `&buffer[..bytes_read]`, which is a slice of only the bytes we actually read. This is important! If the client sends 10 bytes but our buffer is 512 bytes, we only want to echo back those 10 bytes, not all 512 (with 502 zeros).

The slice `&buffer[..bytes_read]` means "from the start of the buffer up to index `bytes_read`."

## Choosing a Buffer Size

We used 512 bytes, but why? It's a trade-off:

- **Smaller buffers** (64, 128 bytes) - Less memory per connection, but more system calls
- **Larger buffers** (4KB, 8KB) - Fewer system calls, but more memory per connection

For an echo server where we're only handling one connection at a time, it doesn't matter much. 512 bytes is a reasonable middle ground.

Many network programs use 4KB (4096 bytes) or 8KB (8192 bytes) because these match common page sizes and network MTUs. But don't overthink it—start with something reasonable and optimize later if needed.

```rust
let mut buffer = [0; 4096];  // Also a fine choice
```

## Testing the Echo Server

Let's test this! Run your server:

```bash
cargo run
```

In another terminal, use `curl` with the `--data` flag to send some data:

```bash
curl --data "Hello, echo server!" localhost:8080
```

You should see your message echoed back:
```
Hello, echo server!
```

And your server logs:
```
[127.0.0.1:54321] Connection established
[127.0.0.1:54321] Received 21 bytes
[127.0.0.1:54321] Connection closed
```

Beautiful! The server received your message, echoed it back, and closed cleanly when `curl` finished.

## Multiple Reads

What if the client sends a lot of data? More than our buffer can hold? The `read()` call will only read up to 512 bytes, and you'll need multiple reads to get it all.

Try sending more data:

```bash
curl --data "$(yes "Hello " | head -n 100 | tr -d '\n')" localhost:8080
```

This sends "Hello " repeated 100 times (600 bytes). Your server will log:

```
[127.0.0.1:54322] Received 512 bytes
[127.0.0.1:54322] Received 88 bytes
[127.0.0.1:54322] Connection closed
```

It took two reads—the first got 512 bytes (the full buffer), and the second got the remaining 88 bytes. Each read was echoed back immediately. This is perfect for an echo server!

## Handling Errors

What happens if something goes wrong during read or write? The `?` operator will propagate the error up and exit the program. For a production server, you'd want more granular error handling:

```rust
loop {
    let bytes_read = match stream.read(&mut buffer) {
        Ok(0) => {
            println!("[{}] Connection closed", addr);
            break;
        }
        Ok(n) => n,
        Err(e) => {
            eprintln!("[{}] Read error: {}", addr, e);
            break;
        }
    };
    
    if let Err(e) = stream.write_all(&buffer[..bytes_read]) {
        eprintln!("[{}] Write error: {}", addr, e);
        break;
    }
}
```

This way, if one connection fails, it doesn't crash the entire server. The error is logged, the connection closes, and the server keeps accepting new connections.

## Why TCP Streams Are Like Files

Notice how similar this is to file I/O? You could almost replace `TcpStream` with a `File` and the code would still make sense:

```rust
// Reading from a file
let mut file = File::open("data.txt")?;
let mut buffer = [0; 512];
let bytes_read = file.read(&mut buffer)?;

// Reading from a network stream
let mut stream = /* TcpStream */;
let mut buffer = [0; 512];
let bytes_read = stream.read(&mut buffer)?;
```

This is one of Rust's strengths. By using traits like `Read` and `Write`, the language provides a consistent interface across different I/O types. Skills you learn working with files transfer directly to network programming.

## Flushing

One more thing: sometimes you want to ensure data is actually sent immediately, not buffered by the OS:

```rust
stream.write_all(b"Important message")?;
stream.flush()?;
```

The `flush()` method forces any buffered data to be sent right away. For our echo server, it's not necessary (data gets sent when we read again or when the connection closes), but it's useful when you need real-time responsiveness.

## What We've Built

We now have a complete, working echo server! It:

- Accepts connections
- Reads data from clients
- Echoes that data back immediately  
- Handles connection closes gracefully
- Processes multiple messages from the same client

The only limitation is that it handles one connection at a time. While it's busy with one client, other clients have to wait. We'll solve this with concurrency in Part III, but for now, you've got a solid foundation in TCP programming.

In the next section, we'll look at the various errors that can occur and how to handle them gracefully. Because in network programming, things *will* go wrong, and your server needs to be resilient.