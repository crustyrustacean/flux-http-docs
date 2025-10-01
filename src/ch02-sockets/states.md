
# Socket States

A TCP socket isn't just "open" or "closed." Throughout its lifetime, it transitions through various states as it establishes connections, transfers data, and shuts down. Understanding these states is like understanding the gears in a car—you don't need to think about them during normal driving, but when something goes wrong, knowing what gear you're in makes all the difference.

## Why States Matter

You might be thinking: "Can't I just connect, send data, and disconnect? Why do I need to know about states?"

Here's why states matter:
- **Debugging**: When `netstat` shows sockets in `TIME_WAIT` or `CLOSE_WAIT`, you need to know what that means
- **Resource management**: Sockets in certain states consume resources even though they're not actively transferring data
- **Connection problems**: "Connection refused" vs. "Connection reset" vs. "Connection timeout" all relate to different socket states
- **Performance tuning**: Understanding states helps you optimize connection pooling and reuse

Think of socket states like relationship statuses. You're not just "single" or "married"—there's "talking," "dating," "engaged," "married," "separated," and "divorced." Each state has different rules and expectations. Sockets are the same way.

## The Socket State Diagram

TCP defines 11 different states a socket can be in. Don't worry—we'll break them down in a way that makes sense. Here's a simplified view of the main states you'll encounter:

```
CLOSED → LISTEN → SYN_RECEIVED → ESTABLISHED → FIN_WAIT → TIME_WAIT → CLOSED
         ↓                              ↓
    SYN_SENT ────────────────────→ ESTABLISHED
```

Let's walk through these states with the journey of a typical connection.

## Server-Side States: Waiting for Guests

When you create a server, it goes through these states:

### CLOSED
This is the starting state. The socket exists, but nothing has happened yet. It's like having a phone that's not plugged in.

### LISTEN
When you call `bind()` and `listen()`, the socket enters the `LISTEN` state. It's now waiting for incoming connection requests. This is like your phone being plugged in and ready to receive calls.

```rust
use std::net::TcpListener;

fn main() -> std::io::Result<()> {
    // Socket is created and immediately enters LISTEN state
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Server socket is in LISTEN state");
    
    // The socket stays in LISTEN state, waiting for clients
    // It never leaves LISTEN state—it creates NEW sockets when clients connect
    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                // This NEW socket (not the listener) is in ESTABLISHED state
                println!("New socket created in ESTABLISHED state");
            }
            Err(e) => {
                eprintln!("Connection attempt failed: {}", e);
            }
        }
    }
    
    Ok(())
}
```

Here's an important point: the listening socket **stays in LISTEN state**. When a client connects, the OS creates a *new* socket that handles that specific connection.

### SYN_RECEIVED
When a client sends a SYN packet to your listening socket, the OS creates a new socket in `SYN_RECEIVED` state to handle the handshake. This socket is waiting for the final ACK from the client.

You never directly see this state in application code—the OS manages it. If the handshake completes, the socket moves to `ESTABLISHED`. If it times out, the socket is destroyed.

### ESTABLISHED
Once the three-way handshake completes, the socket enters `ESTABLISHED` state. This is where actual data transfer happens. Both sides can send and receive data freely.

```rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write};

fn handle_connection(mut stream: TcpStream) -> std::io::Result<()> {
    // We're in ESTABLISHED state here
    println!("Connection is ESTABLISHED, ready for data transfer");
    
    let mut buffer = [0; 1024];
    
    // Read and write while in ESTABLISHED state
    loop {
        let bytes_read = stream.read(&mut buffer)?;
        
        if bytes_read == 0 {
            // Client initiated close, we're transitioning out of ESTABLISHED
            println!("Client closed connection, leaving ESTABLISHED state");
            break;
        }
        
        stream.write_all(&buffer[..bytes_read])?;
    }
    
    Ok(())
}

fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    
    for stream in listener.incoming() {
        if let Ok(stream) = stream {
            handle_connection(stream)?;
        }
    }
    
    Ok(())
}
```

## Client-Side States: Making the Call

Clients go through a slightly different set of states:

### CLOSED
Starting state, same as the server.

### SYN_SENT
When you call `connect()`, the client sends a SYN packet and enters `SYN_SENT` state. It's waiting for the SYN-ACK from the server.

```rust
use std::net::TcpStream;
use std::time::Instant;

fn main() -> std::io::Result<()> {
    let start = Instant::now();
    
    println!("Entering SYN_SENT state (sending SYN)...");
    
    // During this call, we're in SYN_SENT state
    // If this times out, we never got the SYN-ACK
    match TcpStream::connect("example.com:80") {
        Ok(stream) => {
            let elapsed = start.elapsed();
            println!("Handshake completed in {:?}", elapsed);
            println!("Now in ESTABLISHED state");
            
            // We can now use the stream
        }
        Err(e) => {
            println!("Failed to reach ESTABLISHED state: {}", e);
            // Could be:
            // - Connection refused (server sent RST)
            // - Connection timeout (no response to SYN)
            // - Network unreachable
        }
    }
    
    Ok(())
}
```

If the server responds with SYN-ACK and the client sends the final ACK, the socket moves to `ESTABLISHED`.

### ESTABLISHED
Same as server—this is where data flows.

## Closing States: The Goodbye Dance

Closing a TCP connection is more complex than opening it because both sides need to gracefully shut down. This is where many states come in.

### FIN_WAIT_1
When one side decides to close the connection, it sends a FIN (finish) packet and enters `FIN_WAIT_1` state. It's waiting for an ACK of its FIN.

### FIN_WAIT_2
After receiving the ACK, the socket moves to `FIN_WAIT_2`. It's waiting for the other side to send its own FIN.

### CLOSE_WAIT
When you receive a FIN from the other side, your socket enters `CLOSE_WAIT` state. The other side is done sending, but you can still send data if you want. When you're done, you send your own FIN.

This is important: `CLOSE_WAIT` sockets stick around if you don't properly close them. Let's see an example:

```rust
use std::net::TcpStream;
use std::io::{Read, Write};
use std::time::Duration;
use std::thread;

fn main() -> std::io::Result<()> {
    let mut stream = TcpStream::connect("example.com:80")?;
    
    // Send an HTTP request
    stream.write_all(b"GET / HTTP/1.1\r\nHost: example.com\r\n\r\n")?;
    
    let mut buffer = [0; 4096];
    let bytes_read = stream.read(&mut buffer)?;
    println!("Received {} bytes", bytes_read);
    
    // At this point, if the server closes its side, we enter CLOSE_WAIT
    // But we haven't closed our side yet!
    
    println!("Socket might be in CLOSE_WAIT now");
    thread::sleep(Duration::from_secs(5));
    
    // When stream goes out of scope, Rust closes it properly
    // This sends our FIN and completes the close
    Ok(())
    
    // If we had forgotten to close (kept the stream alive forever),
    // we'd have a socket stuck in CLOSE_WAIT leaking resources!
}
```

### CLOSING
If both sides send FIN simultaneously (rare but possible), they enter `CLOSING` state. They're each waiting for an ACK of their FIN.

### LAST_ACK
After sending your FIN in response to the other side's FIN, you enter `LAST_ACK` state, waiting for the ACK of your FIN.

### TIME_WAIT
This is the most infamous state. After sending the final ACK in the close handshake, the socket enters `TIME_WAIT` and stays there for a while (typically 30-120 seconds, depending on the OS).

Why? Two reasons:
1. **Lost ACK handling**: If the final ACK gets lost, the other side will retransmit its FIN, and we need to be able to ACK it again
2. **Old duplicate packets**: Ensures that old duplicate packets from this connection have died off before we reuse the same port numbers

Here's where you'll notice `TIME_WAIT` in practice:

```rust
use std::net::TcpListener;
use std::thread;
use std::time::Duration;

fn main() -> std::io::Result<()> {
    // Start a server
    {
        let listener = TcpListener::bind("127.0.0.1:8080")?;
        println!("Server running on port 8080");
        
        // Accept one connection
        if let Ok((stream, addr)) = listener.accept() {
            println!("Accepted connection from {}", addr);
            drop(stream); // Close the connection
        }
        
        // Drop the listener, closing it
        drop(listener);
        println!("Server shut down");
    }
    
    // Try to immediately start a new server on the same port
    println!("Trying to bind to port 8080 again...");
    
    // This often fails! The old socket is in TIME_WAIT
    match TcpListener::bind("127.0.0.1:8080") {
        Ok(_) => println!("Success! No TIME_WAIT issue."),
        Err(e) => {
            println!("Failed: {}", e);
            println!("The port is likely in TIME_WAIT state");
            println!("Wait 30-120 seconds and try again, or use a different port");
        }
    }
    
    Ok(())
}
```

You can work around `TIME_WAIT` by setting the `SO_REUSEADDR` socket option:

```rust
use std::net::{TcpListener, TcpStream};
use socket2::{Socket, Domain, Type};

fn create_reusable_listener(addr: &str) -> std::io::Result<TcpListener> {
    // Parse the address
    let addr: std::net::SocketAddr = addr.parse()
        .map_err(|e| std::io::Error::new(std::io::ErrorKind::InvalidInput, e))?;
    
    // Create a socket with socket2 for more control
    let socket = Socket::new(Domain::IPV4, Type::STREAM, None)?;
    
    // Set SO_REUSEADDR to allow binding even if the port is in TIME_WAIT
    socket.set_reuse_address(true)?;
    
    // Bind and listen
    socket.bind(&addr.into())?;
    socket.listen(128)?;
    
    // Convert to std::net::TcpListener
    Ok(socket.into())
}

fn main() -> std::io::Result<()> {
    let listener = create_reusable_listener("127.0.0.1:8080")?;
    println!("Server bound with SO_REUSEADDR set");
    println!("Can rebind immediately even if port was in TIME_WAIT");
    
    // Use listener...
    
    Ok(())
}
```

Note: This example uses the `socket2` crate for socket options not exposed in `std::net`. Add `socket2 = "0.5"` to your `Cargo.toml` to use it.

### CLOSED
Finally, after `TIME_WAIT` expires (or immediately from other states), the socket returns to `CLOSED` and all resources are freed.

## Special State: RST (Reset)

There's one more important thing: the RST (reset) flag. This isn't a state, but a way to immediately abort a connection from any state.

When you receive a RST:
- The connection is immediately dead
- No graceful close handshake
- Any data in flight is lost

You'll see RST in several situations:
- Connecting to a port with no listener: "Connection refused"
- Sending data to a closed connection: "Connection reset by peer"
- Firewall dropping packets

```rust
use std::net::TcpStream;
use std::io::{Write, Read};
use std::time::Duration;
use std::thread;

fn main() -> std::io::Result<()> {
    let mut stream = TcpStream::connect("127.0.0.1:8080")?;
    
    println!("Connected successfully");
    
    // Forcefully close the connection on our side
    drop(stream);
    
    // Give it a moment
    thread::sleep(Duration::from_millis(100));
    
    // Try to connect again and send data
    let mut stream = TcpStream::connect("127.0.0.1:8080")?;
    
    // Close the connection forcefully AGAIN
    drop(stream);
    
    // If we had tried to send data to the dropped connection,
    // we would receive a RST and get "Connection reset by peer" error
    
    Ok(())
}
```

## Viewing Socket States

Want to see these states in action? Use command-line tools:

**Linux/macOS:**
```bash
netstat -an | grep 8080
ss -tan | grep 8080
```

**Windows:**
```bash
netstat -an | findstr 8080
```

You'll see output like:
```
tcp   0   0   127.0.0.1:8080    0.0.0.0:*       LISTEN
tcp   0   0   127.0.0.1:8080    127.0.0.1:45678 ESTABLISHED
tcp   0   0   127.0.0.1:8080    127.0.0.1:45679 TIME_WAIT
tcp   0   0   127.0.0.1:8080    127.0.0.1:45680 CLOSE_WAIT
```

## Common Problems and Their States

**"Address already in use"**: You're trying to bind to a port that has a socket in `LISTEN` or `TIME_WAIT` state.

**Too many CLOSE_WAIT sockets**: Your application received FINs but never closed its side. You're leaking sockets! Make sure to properly drop your `TcpStream` objects.

**Too many TIME_WAIT sockets**: Normal for high-traffic servers. You're closing connections faster than TIME_WAIT expires. Consider connection pooling or increasing the TIME_WAIT timeout.

**Connection hangs**: Might be stuck in `SYN_SENT` (server not responding) or `FIN_WAIT_2` (waiting for the other side to close).

## State Transitions Summary

Here's a quick reference of the happy path:

**Server:**
```
CLOSED → LISTEN → SYN_RECEIVED → ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED
```

**Client:**
```
CLOSED → SYN_SENT → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED
```

## What's Next?

You now understand the lifecycle of a TCP socket—from birth (CLOSED) through adolescence (connecting states), adulthood (ESTABLISHED), and death (closing states).

In the next chapter, we'll put all this knowledge to use and build our first TCP server from scratch. We'll handle connections, read data, write responses, and properly manage socket lifecycles. You'll see these states in action and understand why proper error handling and resource management matter.

The theory is done. Now it's time to build something real.