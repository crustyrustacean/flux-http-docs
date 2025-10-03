# Quick Reference

Here's a handy cheatsheet for everything we covered in this chapter. Bookmark this page—you'll refer back to it often.

## Essential Imports

```rust
use std::net::{TcpListener, TcpStream, SocketAddr};
use std::io::{Read, Write, ErrorKind};
```

## Core Types

| Type | Purpose | Created By |
|------|---------|------------|
| `TcpListener` | Listening socket that accepts connections | `TcpListener::bind()` |
| `TcpStream` | Active connection to a client | `listener.accept()` or `listener.incoming()` |
| `SocketAddr` | IP address + port (e.g., `127.0.0.1:8080`) | Returned by `accept()` or `peer_addr()` |

## Creating a Listener

```rust
// Basic binding
let listener = TcpListener::bind("127.0.0.1:8080")?;

// Bind to all interfaces (accessible from network)
let listener = TcpListener::bind("0.0.0.0:8080")?;

// Let OS assign a port
let listener = TcpListener::bind("127.0.0.1:0")?;
let port = listener.local_addr()?.port();

// With better error handling
let listener = TcpListener::bind("127.0.0.1:8080")
    .expect("Failed to bind - port already in use?");
```

## Accepting Connections

```rust
// Accept one connection
let (stream, addr) = listener.accept()?;

// Accept multiple connections (manual loop)
loop {
    let (stream, addr) = listener.accept()?;
    // Handle stream...
}

// Accept multiple connections (iterator style - preferred)
for stream in listener.incoming() {
    match stream {
        Ok(stream) => { /* handle connection */ }
        Err(e) => { /* log error, continue */ }
    }
}
```

## Reading Data

```rust
let mut buffer = [0; 4096];

// Read some bytes (returns number read)
let n = stream.read(&mut buffer)?;

// Zero means client closed connection
if n == 0 {
    println!("Connection closed");
    return Ok(());
}

// Read exactly n bytes (errors if connection closes first)
stream.read_exact(&mut buffer[..100])?;
```

## Writing Data

```rust
// Write all bytes (preferred - handles partial writes)
stream.write_all(b"Hello, client!")?;
stream.write_all(&buffer[..n])?;  // Write first n bytes

// Write some bytes (returns number written - low level)
let n = stream.write(b"Hello")?;

// Force data to be sent immediately
stream.flush()?;
```

## Buffer Patterns

```rust
// Fixed-size buffer
let mut buffer = [0; 4096];
let n = stream.read(&mut buffer)?;
stream.write_all(&buffer[..n])?;  // Only write what we read!

// Vector buffer (growable)
let mut buffer = vec![0; 4096];
let n = stream.read(&mut buffer)?;
buffer.truncate(n);  // Shrink to actual size

// String buffer (if you know it's UTF-8)
let mut buffer = String::new();
stream.read_to_string(&mut buffer)?;
```

## Connection Information

```rust
// Client's address
let addr = stream.peer_addr()?;
println!("Client: {}", addr);

// Our local address
let local = stream.local_addr()?;
println!("Server: {}", local);

// Check if addresses are available
if let Ok(addr) = stream.peer_addr() {
    println!("Connected to {}", addr);
}
```

## Error Handling

```rust
// Match specific error kinds
match stream.read(&mut buffer) {
    Ok(0) => { /* Connection closed */ }
    Ok(n) => { /* Got n bytes */ }
    Err(e) => match e.kind() {
        ErrorKind::ConnectionReset => { /* Client crashed */ }
        ErrorKind::BrokenPipe => { /* Already closed */ }
        ErrorKind::Interrupted => { /* Retry */ }
        ErrorKind::TimedOut => { /* Timeout expired */ }
        ErrorKind::WouldBlock => { /* Non-blocking mode */ }
        _ => return Err(e),  // Unexpected error
    }
}
```

## Common Error Kinds

| Error | When It Happens | How to Handle |
|-------|-----------------|---------------|
| `ConnectionReset` | Client abruptly closed | Log and close connection (normal) |
| `BrokenPipe` | Write to closed connection | Log and close connection (normal) |
| `Interrupted` | System call interrupted | Retry the operation |
| `TimedOut` | Operation exceeded timeout | Close connection or retry |
| `WouldBlock` | Non-blocking I/O not ready | Wait and retry (or use async) |
| `ConnectionRefused` | Nothing listening on port | Service is down (client-side error) |
| `AddrInUse` | Port already bound | Choose different port or stop other process |
| `PermissionDenied` | No permission (e.g., port < 1024) | Use higher port or run with sudo |

## Timeouts

```rust
use std::time::Duration;

// Set timeouts (prevents hanging forever)
stream.set_read_timeout(Some(Duration::from_secs(30)))?;
stream.set_write_timeout(Some(Duration::from_secs(30)))?;

// Disable timeout (default)
stream.set_read_timeout(None)?;

// Check current timeout
if let Some(timeout) = stream.read_timeout()? {
    println!("Timeout: {:?}", timeout);
}
```

## Complete Server Pattern

```rust
fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    
    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                // Isolate each client's errors
                if let Err(e) = handle_client(stream) {
                    eprintln!("Client error: {}", e);
                }
            }
            Err(e) => eprintln!("Accept error: {}", e),
        }
    }
    
    Ok(())
}

fn handle_client(mut stream: TcpStream) -> std::io::Result<()> {
    let addr = stream.peer_addr()?;
    let mut buffer = [0; 4096];
    
    loop {
        match stream.read(&mut buffer) {
            Ok(0) => return Ok(()),  // Clean close
            Ok(n) => {
                stream.write_all(&buffer[..n])?;
            }
            Err(e) if e.kind() == ErrorKind::ConnectionReset => {
                return Ok(());  // Client disconnected
            }
            Err(e) => return Err(e),
        }
    }
}
```

## Common Port Numbers

| Range | Category | Examples | Access |
|-------|----------|----------|--------|
| 0-1023 | Privileged | 80 (HTTP), 443 (HTTPS), 22 (SSH) | Requires root/admin |
| 1024-49151 | Registered | 3000, 8080, 5432 (PostgreSQL) | Normal user |
| 49152-65535 | Dynamic | Ephemeral ports | Normal user |

**For development:** Use `3000`, `8000`, `8080`, or `9000`—unlikely to conflict.

## Address Formats

```rust
// Localhost only
"127.0.0.1:8080"
"localhost:8080"

// All interfaces (accessible from network)
"0.0.0.0:8080"

// Specific interface
"192.168.1.100:8080"

// IPv6
"[::1]:8080"          // IPv6 localhost
"[::]:8080"           // All IPv6 interfaces

// Let OS choose port
"127.0.0.1:0"
```

## Testing Your Server

```bash
# Basic test
curl localhost:8080

# Send data
curl --data "Hello!" localhost:8080

# Send file
curl --data-binary @file.txt localhost:8080

# Keep connection open (interactive)
nc localhost 8080

# Multiple concurrent connections
for i in {1..10}; do curl --data "Client $i" localhost:8080 & done

# Check if port is in use
lsof -i :8080          # macOS/Linux
netstat -an | grep 8080  # Windows/Linux
```

## Common Pitfalls

### ❌ Writing entire buffer instead of what you read
```rust
let n = stream.read(&mut buffer)?;
stream.write_all(&buffer)?;  // WRONG - sends garbage!
```

### ✅ Only write what you read
```rust
let n = stream.read(&mut buffer)?;
stream.write_all(&buffer[..n])?;  // RIGHT
```

### ❌ Not checking for zero-length reads
```rust
loop {
    let n = stream.read(&mut buffer)?;
    stream.write_all(&buffer[..n])?;  // Infinite loop if n=0!
}
```

### ✅ Check for connection close
```rust
loop {
    let n = stream.read(&mut buffer)?;
    if n == 0 {
        break;  // Client closed
    }
    stream.write_all(&buffer[..n])?;
}
```

### ❌ Letting one client crash the server
```rust
for stream in listener.incoming() {
    let stream = stream?;  // WRONG - error crashes server
    handle_client(stream)?;  // WRONG - crashes server
}
```

### ✅ Isolate client errors
```rust
for stream in listener.incoming() {
    match stream {
        Ok(stream) => {
            if let Err(e) = handle_client(stream) {
                eprintln!("Client error: {}", e);  // Log and continue
            }
        }
        Err(e) => eprintln!("Accept error: {}", e),
    }
}
```

## Performance Tips

- **Buffer size:** Start with 4KB (4096 bytes) - matches page size
- **Timeouts:** Always set them in production (prevents hanging)
- **Read loops:** Check for zero-length reads to detect closes
- **Error handling:** Treat `ConnectionReset` and `BrokenPipe` as normal
- **Concurrency:** This chapter's server handles one client at a time—see Part III for solutions

## What's Next?

This chapter covered raw TCP. Now you can:
- Create listening sockets
- Accept connections
- Read and write bytes
- Handle errors gracefully

**Coming up in Part II:** We'll learn the HTTP protocol and build an HTTP server on top of this TCP foundation. Same socket code, different data format!

---

**Pro tip:** Keep this page open while coding. These patterns will become second nature, but the error kinds and edge cases are easy to forget.
