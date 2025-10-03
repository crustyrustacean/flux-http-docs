# A Complete Echo Server

We've covered all the pieces—now let's put them together into a complete, production-quality echo server. This is the culmination of everything we've learned in this chapter.

## The Final Code

Here's our echo server with proper error handling, logging, and comments:

```rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write, ErrorKind};

fn main() -> std::io::Result<()> {
    // Bind to localhost on port 8080
    // This creates a socket, binds it, and puts it in listening mode
    let listener = TcpListener::bind("127.0.0.1:8080")
        .expect("Failed to bind to port 8080 - is it already in use?");
    
    println!("Echo server listening on 127.0.0.1:8080");
    println!("Test with: curl --data 'Hello!' localhost:8080");
    
    // Accept connections in an infinite loop
    // Each connection is handled sequentially (one at a time)
    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                // Handle this client connection
                // Errors here only affect this client, not the whole server
                if let Err(e) = handle_client(stream) {
                    eprintln!("Error handling client: {}", e);
                }
            }
            Err(e) => {
                // Failed to accept a connection - log it but keep running
                eprintln!("Failed to accept connection: {}", e);
            }
        }
    }
    
    Ok(())
}

/// Handles a single client connection
/// Reads data from the client and echoes it back until the connection closes
fn handle_client(mut stream: TcpStream) -> std::io::Result<()> {
    // Get the client's address for logging
    let addr = match stream.peer_addr() {
        Ok(addr) => addr.to_string(),
        Err(_) => "unknown".to_string(),
    };
    
    println!("[{}] Connection established", addr);
    
    // Buffer for reading data - 4KB is a good size for most purposes
    let mut buffer = [0; 4096];
    
    // Keep echoing until the client closes the connection
    loop {
        // Read data from the client
        let bytes_read = match stream.read(&mut buffer) {
            Ok(0) => {
                // Zero bytes means the client closed the connection cleanly
                println!("[{}] Connection closed", addr);
                return Ok(());
            }
            Ok(n) => n,
            Err(e) => {
                // Handle different error types appropriately
                return match e.kind() {
                    ErrorKind::ConnectionReset => {
                        // Client abruptly closed - this is normal
                        println!("[{}] Connection reset by client", addr);
                        Ok(())
                    }
                    ErrorKind::BrokenPipe => {
                        // Tried to read from a closed connection
                        println!("[{}] Broken pipe", addr);
                        Ok(())
                    }
                    ErrorKind::Interrupted => {
                        // System call was interrupted - just retry
                        println!("[{}] Read interrupted, retrying", addr);
                        continue;
                    }
                    _ => {
                        // Unexpected error - log and propagate
                        eprintln!("[{}] Read error: {}", addr, e);
                        Err(e)
                    }
                };
            }
        };
        
        println!("[{}] Received {} bytes, echoing back", addr, bytes_read);
        
        // Echo the data back to the client
        // We only write the bytes we actually read, not the whole buffer
        if let Err(e) = stream.write_all(&buffer[..bytes_read]) {
            return match e.kind() {
                ErrorKind::ConnectionReset | ErrorKind::BrokenPipe => {
                    // Client disconnected while we were writing
                    println!("[{}] Client disconnected during write", addr);
                    Ok(())
                }
                ErrorKind::Interrupted => {
                    // Interrupted - we should retry, but for simplicity we'll just close
                    println!("[{}] Write interrupted", addr);
                    Ok(())
                }
                _ => {
                    eprintln!("[{}] Write error: {}", addr, e);
                    Err(e)
                }
            };
        }
    }
}
```

## What We've Built

This server does everything a basic TCP server needs to:

1. **Binds to an address** - Claims port 8080 on localhost
2. **Accepts connections** - Handles incoming clients one at a time
3. **Reads data** - Gets bytes from the client into a buffer
4. **Echoes data back** - Sends the same bytes back to the client
5. **Handles errors gracefully** - Doesn't crash when things go wrong
6. **Logs activity** - Shows what's happening for debugging

Each function has a clear, single responsibility. The separation between `main()` and `handle_client()` means errors in one connection don't affect others.

## Running the Server

Compile and run:

```bash
cargo run
```

You should see:
```
Echo server listening on 127.0.0.1:8080
Test with: curl --data 'Hello!' localhost:8080
```

## Testing It

### Basic Echo Test

In another terminal:

```bash
curl --data "Hello, echo server!" localhost:8080
```

You should see your message echoed back:
```
Hello, echo server!
```

And the server logs:
```
[127.0.0.1:54321] Connection established
[127.0.0.1:54321] Received 21 bytes, echoing back
[127.0.0.1:54321] Connection closed
```

### Multiple Messages

`curl` closes the connection after one request, but you can use `nc` (netcat) to keep the connection open and send multiple messages:

```bash
nc localhost 8080
```

Type something and press Enter—you'll see it echoed back immediately. Each line you type gets echoed. Press Ctrl+C to disconnect.

### Large Data Transfer

Send a bigger chunk of data:

```bash
curl --data "$(head -c 10000 < /dev/urandom | base64)" localhost:8080
```

The server will read it in chunks (up to 4096 bytes at a time) and echo each chunk back. You'll see multiple "Received X bytes" messages in the server logs.

### Multiple Clients

Try opening several `curl` connections simultaneously:

```bash
curl --data "Client 1" localhost:8080 &
curl --data "Client 2" localhost:8080 &
curl --data "Client 3" localhost:8080 &
```

They'll be handled sequentially—the second client waits for the first to finish, the third waits for the second, and so on. This is the limitation of our single-threaded approach, which we'll fix in Part III.

## What's Missing?

This echo server is fully functional, but it has one major limitation: **concurrency**. It handles one client at a time. While it's processing one connection, all other clients have to wait.

For an echo server, this is usually fine—echoing is fast! But for real servers that do complex work (database queries, file I/O, computation), this becomes a bottleneck quickly.

Here's what we'll add later:

- **Threads** (Chapter 11) - Handle multiple clients simultaneously
- **Thread pools** (Chapter 12) - Limit resource usage with a fixed pool
- **Async I/O** (Chapter 14-16) - Handle thousands of concurrent connections efficiently

But for now, you have a working TCP server. You understand sockets, you can read and write bytes, and you handle errors properly. That's the foundation everything else is built on.

## Key Takeaways

Let's recap what we learned in this chapter:

1. **Rust's `std::net`** provides clean abstractions over system sockets
2. **`TcpListener::bind()`** creates a listening socket in one call
3. **`accept()`** completes the three-way handshake and returns a `TcpStream`
4. **Reading and writing** use the same `Read` and `Write` traits as file I/O
5. **Zero-length reads** indicate the client closed the connection
6. **Error handling** separates connection failures from server crashes
7. **Blocking I/O** is simple but limits concurrency

You've gone from theory to practice. You understand how TCP works at the protocol level (Part I), and you can now build a real server that uses it. The rest of this book builds on this foundation—first by adding concurrency, then by implementing the HTTP protocol on top of TCP, and finally by building a web framework.

But take a moment to appreciate what you've built. This is a real, working network server. It's the same basic structure that web servers, database servers, and chat servers use. The only difference is what you *do* with the data between reading and writing.

Next up: HTTP. We'll learn what makes HTTP special, how to parse HTTP messages, and how to build an HTTP server on top of our TCP foundation. Ready to level up?
