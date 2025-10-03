# Error Handling

Network programming is full of things that can go wrong. Clients disconnect suddenly. Networks get flaky. Ports are already in use. If you don't handle errors properly, your server will crash at the first hiccup. Let's make our echo server resilient.

## The Reality of Network Errors

Here's the uncomfortable truth: in file I/O, errors are the exception. Most file reads succeed. But in network programming, errors are *normal*. Clients disconnect mid-transfer. Connections timeout. Packets get lost. Your server needs to shrug these off and keep running.

Think of it like running a shop. If one customer leaves without buying anything, you don't close the store. You just help the next customer.

## Types of Errors

Rust's networking functions return `std::io::Result<T>`, which is really `Result<T, std::io::Error>`. The `std::io::Error` type wraps different kinds of errors, each with an `ErrorKind`:

```rust
use std::io::ErrorKind;

match stream.read(&mut buffer) {
    Ok(n) => { /* handle n bytes */ }
    Err(e) => {
        match e.kind() {
            ErrorKind::ConnectionReset => {
                println!("Client reset the connection");
            }
            ErrorKind::BrokenPipe => {
                println!("Connection broken");
            }
            ErrorKind::WouldBlock => {
                println!("Operation would block");
            }
            _ => {
                eprintln!("Unknown error: {}", e);
            }
        }
    }
}
```

Let's go through the common ones you'll encounter.

## Connection Errors

### ConnectionRefused

This happens when you try to connect to a port where nothing is listening:

```rust
// If no server is running on port 9999...
let stream = TcpStream::connect("127.0.0.1:9999");
// Error: Connection refused (os error 61)
```

For a server, you rarely see this. But if you're building a client that connects to other services, this is how you know the service is down.

### ConnectionReset

The client (or an intermediary router) forcibly closed the connection. This is the TCP equivalent of hanging up the phone mid-conversation:

```rust
// Client closes their end abruptly
let bytes_read = stream.read(&mut buffer)?;
// Error: Connection reset by peer (os error 54)
```

This is **normal** in network programming. Maybe the user closed their browser. Maybe their Wi-Fi cut out. Your server should log it and move on.

### BrokenPipe

You tried to write to a connection that's already closed:

```rust
// Connection is closed...
stream.write_all(b"Hello")?;
// Error: Broken pipe (os error 32)
```

This usually means the client disconnected and you didn't notice before trying to write. Check for zero-length reads before writing.

## Graceful Error Handling

Here's our echo server with proper error handling that keeps it running despite problems:

```rust
use std::net::TcpListener;
use std::io::{Read, Write, ErrorKind};

fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Echo server listening on port 8080");
    
    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                if let Err(e) = handle_client(stream) {
                    eprintln!("Error handling client: {}", e);
                    // Don't crash - just log and continue with next client
                }
            }
            Err(e) => {
                eprintln!("Failed to accept connection: {}", e);
                // Keep accepting other connections
            }
        }
    }
    
    Ok(())
}

fn handle_client(mut stream: std::net::TcpStream) -> std::io::Result<()> {
    let addr = stream.peer_addr()?;
    println!("[{}] Connection established", addr);
    
    let mut buffer = [0; 512];
    
    loop {
        let bytes_read = match stream.read(&mut buffer) {
            Ok(0) => {
                println!("[{}] Connection closed cleanly", addr);
                return Ok(());
            }
            Ok(n) => n,
            Err(e) => {
                return match e.kind() {
                    ErrorKind::ConnectionReset => {
                        println!("[{}] Connection reset by client", addr);
                        Ok(())  // Not really an error - just log it
                    }
                    ErrorKind::BrokenPipe => {
                        println!("[{}] Broken pipe", addr);
                        Ok(())  // Client disconnected
                    }
                    _ => {
                        eprintln!("[{}] Read error: {}", addr, e);
                        Err(e)  // Propagate unexpected errors
                    }
                };
            }
        };
        
        println!("[{}] Received {} bytes", addr, bytes_read);
        
        if let Err(e) = stream.write_all(&buffer[..bytes_read]) {
            return match e.kind() {
                ErrorKind::ConnectionReset | ErrorKind::BrokenPipe => {
                    println!("[{}] Client disconnected during write", addr);
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

Notice the structure: we've extracted connection handling into a separate function. This way, errors in `handle_client()` don't crash the server—they just close that one connection.

## The Power of Separation

By separating `handle_client()` from the main loop, we create a firewall. Each client connection is isolated:

```rust
for stream in listener.incoming() {
    match stream {
        Ok(stream) => {
            if let Err(e) = handle_client(stream) {
                // This error only affects this one client
                eprintln!("Error with client: {}", e);
            }
            // Server keeps running!
        }
        Err(e) => {
            eprintln!("Accept failed: {}", e);
            // Even accept failures don't crash us
        }
    }
}
```

This is a fundamental pattern in server design. One bad connection shouldn't bring down the whole system.

## Timeouts

By default, TCP operations block forever. If a client connects but never sends data, `read()` will wait... and wait... and wait. Forever.

You can set timeouts to prevent this:

```rust
use std::time::Duration;

let mut stream = /* ... */;

// Set a 30-second read timeout
stream.set_read_timeout(Some(Duration::from_secs(30)))?;

// Now reads will fail with TimedOut after 30 seconds
match stream.read(&mut buffer) {
    Ok(n) => { /* ... */ }
    Err(e) if e.kind() == ErrorKind::TimedOut => {
        println!("Client took too long to send data");
        return Ok(());
    }
    Err(e) => return Err(e),
}
```

There's also `set_write_timeout()` for writes. Timeouts are essential in production—without them, a slow or malicious client can tie up your server indefinitely.

## The Interrupted Error

On Unix systems, system calls can be interrupted by signals (like Ctrl+C). When this happens, you get `ErrorKind::Interrupted`:

```rust
loop {
    match stream.read(&mut buffer) {
        Ok(n) => { /* ... */ }
        Err(e) if e.kind() == ErrorKind::Interrupted => {
            // Just retry - this is a temporary condition
            continue;
        }
        Err(e) => return Err(e),
    }
}
```

For `Interrupted`, the right thing to do is usually just retry the operation. It's a temporary glitch, not a real error.

## Testing Error Conditions

How do you test error handling? Here are some tricks:

### Test Connection Reset

Connect with `curl` and then kill it mid-transfer:

```bash
curl localhost:8080 &
PID=$!
sleep 0.1
kill -9 $PID
```

Your server should log "Connection reset" and keep running.

### Test Timeouts

Connect with `telnet` or `nc` (netcat) and just... wait:

```bash
nc localhost 8080
# Don't type anything - just wait
```

If you have timeouts configured, the server should disconnect after the timeout expires.

### Test Large Data

Send more data than the buffer can hold to ensure multiple reads work:

```bash
# Send 10KB of data
dd if=/dev/zero bs=1024 count=10 | curl --data-binary @- localhost:8080
```

## Logging Best Practices

Good error messages make debugging easier. Include context:

```rust
// Bad
eprintln!("Error: {}", e);

// Good
eprintln!("[{}] Read error: {}", addr, e);

// Better
eprintln!("[{}] Read error after {} bytes: {}", addr, total_bytes_read, e);
```

In production, you'd use a proper logging library like `log` or `tracing`:

```rust
use log::{info, warn, error};

info!("[{}] Connection established", addr);
warn!("[{}] Connection reset by peer", addr);
error!("[{}] Unexpected error: {}", addr, e);
```

But for learning, `println!` and `eprintln!` work fine.

## When to Panic vs Return Err

Use this rule of thumb:

**Return `Err`** for:
- Network errors (connection reset, timeout, etc.)
- Expected failures (client disconnects)
- Recoverable problems

**Panic** for:
- Programming bugs (unwrap on None when it shouldn't be)
- Invariant violations
- Unrecoverable errors (out of memory)

For our echo server, almost all errors should be returned, not panicked. The only exception might be binding errors—if you can't bind to the port, there's nothing the server can do:

```rust
let listener = TcpListener::bind("127.0.0.1:8080")
    .expect("Failed to bind to port 8080");
```

This will panic if binding fails, which is reasonable for a server that can't function without its listening socket.

## What We've Learned

Network errors are normal, not exceptional. Your server needs to:

1. **Isolate failures** - One bad connection doesn't crash the server
2. **Log informatively** - Include context in error messages
3. **Distinguish error types** - Handle expected errors gracefully, propagate unexpected ones
4. **Use timeouts** - Don't let slow clients hang forever
5. **Keep running** - The server stays up even when things go wrong

With proper error handling, your echo server is now production-ready from a stability standpoint. It won't crash when clients misbehave. That's a huge achievement!

Of course, it still only handles one client at a time. We'll fix that limitation when we get to concurrency. But first, let's step back and look at what we've built as a complete, working system.
