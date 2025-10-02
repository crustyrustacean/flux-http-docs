# The Three-Way Handshake

When you connect to a website, send an email, or SSH into a server, something magical happens in the first few milliseconds: the three-way handshake. It's the process by which two computers establish a TCP connection, and it happens so fast you never notice it. But understanding it will make you a better network programmer and help you debug connection issues when they inevitably arise.

## Why Do We Need a Handshake?

Remember, the Internet is chaos. Packets get lost, duplicated, delayed, or arrive out of order. Networks are unreliable, and that's okay—TCP is designed to work despite this chaos.

But before TCP can start sending your data reliably, it needs to establish some ground rules with the other side:
- "Are you there and ready to talk?"
- "Here's my starting sequence number for tracking packets"
- "I'm ready to receive data from you"
- "Let's agree on some parameters (window size, maximum segment size, etc.)"

The three-way handshake is how TCP accomplishes all of this before any actual application data flows.

## The Handshake: A Conversation

Think of it like meeting someone for coffee:

**You (Client)**: "Hey! Want to chat?" *(SYN)*

**Them (Server)**: "Sure! I heard you. Ready when you are!" *(SYN-ACK)*

**You (Client)**: "Great, I heard you too. Let's start!" *(ACK)*

At this point, you both know the other person is there, listening, and ready. Now you can have your actual conversation (exchange data).

## The Technical Details

Let's break down what's actually happening at the packet level. Each step sends a TCP segment with specific flags set.

### Step 1: SYN (Synchronize)

The client sends a TCP segment with the **SYN** (synchronize) flag set. This packet says:
- "I want to establish a connection"
- "My initial sequence number is X" (a random number)
- "Here are my TCP options" (maximum segment size, window scaling, etc.)

The client then enters the **SYN_SENT** state, waiting for a response.

### Step 2: SYN-ACK (Synchronize-Acknowledge)

If the server is listening and willing to accept the connection, it responds with a segment that has both the **SYN** and **ACK** flags set:
- "I acknowledge your SYN with sequence number X"
- "My initial sequence number is Y" (a different random number)
- "Here are my TCP options"

The server enters the **SYN_RECEIVED** state. The client now knows the server received its request and is ready.

### Step 3: ACK (Acknowledge)

The client sends back a segment with just the **ACK** flag set:
- "I acknowledge your SYN with sequence number Y"
- "We're connected, let's go!"

Both sides are now in the **ESTABLISHED** state. The connection is open, and data can flow in both directions.

## Why Three Steps? Why Not Two?

You might wonder: why does the client need to send that final ACK? Couldn't we just stop after the SYN-ACK?

The problem is that TCP operates over an unreliable network. Imagine this scenario with a two-way handshake:

1. Client sends SYN
2. Server sends SYN-ACK and immediately considers the connection established
3. The SYN-ACK gets lost in the network

Now the server thinks there's a connection and might allocate resources for it, but the client has no idea. The client will eventually timeout and retry, but the server is left hanging.

With three steps, both sides explicitly confirm they've heard each other before committing resources. The final ACK ensures the client received the server's SYN-ACK.

## Sequence Numbers: The Secret Sauce

Those random sequence numbers we mentioned? They're crucial. They're how TCP tracks every single byte in the connection.

When you send data, each byte gets a sequence number. If you send 100 bytes starting at sequence number 1000, those bytes are numbered 1000 through 1099. The receiver acknowledges by sending back "I received up to byte 1099."

Using random starting sequence numbers (rather than always starting at 0) prevents attacks and avoids confusion if old duplicate packets from previous connections show up.

## Watching the Handshake in Rust

While Rust's standard library abstracts away the handshake (it happens automatically when you connect), we can observe the process. Here's code that connects to a server and shows timing:

```rust
use std::net::TcpStream;
use std::time::Instant;

fn main() -> std::io::Result<()> {
    let server_addr = "example.com:80";
    
    println!("Attempting to connect to {}", server_addr);
    println!("The three-way handshake happens now...");
    
    // The handshake happens inside this connect() call
    let start = Instant::now();
    let stream = TcpStream::connect(server_addr)?;
    let duration = start.elapsed();
    
    println!("Connected in {:?}", duration);
    println!("Connection established! Both sides are in ESTABLISHED state.");
    
    // At this point, the handshake is complete and we can send data
    println!("Local address: {}", stream.local_addr()?);
    println!("Peer address: {}", stream.peer_addr()?);
    
    Ok(())
}
```

On a local connection, this might take microseconds. Connecting to a server across the world? Maybe 100-300 milliseconds—and that's just for the handshake, before any data is sent!

## The Handshake From the Server Side

On the server side, the handshake is handled automatically by the operating system. When you call `accept()`, the handshake has already completed. Here's what's happening behind the scenes:

```rust
use std::net::TcpListener;

fn main() -> std::io::Result<()> {
    // Start listening - this doesn't complete any handshakes yet
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Server listening on port 8080");
    println!("Waiting for clients to initiate three-way handshakes...");
    
    loop {
        // accept() blocks until a complete three-way handshake finishes
        // The OS handles the SYN, SYN-ACK, ACK exchange for us
        match listener.accept() {
            Ok((stream, addr)) => {
                // By the time we get here, the three-way handshake is DONE
                // The connection is in ESTABLISHED state
                println!("New connection established with {}", addr);
                
                // We can now read and write immediately
                // No additional handshaking needed
            }
            Err(e) => {
                // This might happen if the handshake was interrupted
                // or if there was a network error during connection setup
                eprintln!("Connection failed during handshake: {}", e);
            }
        }
    }
}
```

The accept() call blocks until a complete three-way handshake has finished. The operating system maintains a queue of pending connections—clients that have sent SYN but haven't completed the handshake yet. Once the handshake completes, that connection moves to the "ready" queue, and accept() returns it to you.

*(Note: Rust also provides listener.incoming(), which is a convenient iterator that repeatedly calls accept() for you. Both approaches work—we're using accept() directly here to make it explicit.)*

## Timeouts and Retries

What happens if a packet in the handshake gets lost?

TCP automatically retries. If the client sends a SYN and doesn't get a SYN-ACK back within a timeout period (usually 1-3 seconds), it sends the SYN again. It will typically retry several times with increasing delays before giving up.

Here's how you can set a connection timeout:

```rust
use std::net::{TcpStream, SocketAddr};
use std::time::Duration;

fn connect_with_timeout(addr: &str, timeout: Duration) -> std::io::Result<TcpStream> {
    // Parse the address
    let socket_addr: SocketAddr = addr.parse()
        .map_err(|e| std::io::Error::new(std::io::ErrorKind::InvalidInput, e))?;
    
    println!("Connecting with {}s timeout...", timeout.as_secs());
    
    // This will give up if the handshake doesn't complete within the timeout
    TcpStream::connect_timeout(&socket_addr, timeout)
}

fn main() -> std::io::Result<()> {
    // Try to connect with a 5-second timeout
    match connect_with_timeout("example.com:80", Duration::from_secs(5)) {
        Ok(stream) => {
            println!("Handshake completed successfully!");
            println!("Connected to {}", stream.peer_addr()?);
        }
        Err(e) => {
            // Could be timeout, connection refused, or network unreachable
            eprintln!("Handshake failed: {}", e);
        }
    }
    
    Ok(())
}
```

## Security: The SYN Flood Attack

Understanding the handshake helps you understand certain attacks. A **SYN flood** attack exploits the handshake process:

1. Attacker sends thousands of SYN packets with fake source addresses
2. Server responds with SYN-ACK to each one
3. Server waits for the final ACK that will never come (fake addresses!)
4. Server's connection queue fills up with half-open connections
5. Legitimate clients can't connect because the queue is full

This is why servers often limit how many half-open connections they'll maintain and implement SYN cookies—a clever technique to avoid allocating resources until the final ACK arrives.

## The Four-Way Handshake (Closing Connections)

While we're at it, let's mention that TCP also has a handshake for closing connections—but it's four steps instead of three:

1. **FIN**: "I'm done sending data"
2. **ACK**: "I acknowledge you're done sending"
3. **FIN**: "I'm done sending too"
4. **ACK**: "I acknowledge you're done"

This allows both sides to independently signal they're finished. It's possible for one side to close its sending side while still receiving (called a half-closed connection).

In Rust, closing happens automatically when your `TcpStream` goes out of scope:

```rust
use std::net::TcpStream;

fn main() -> std::io::Result<()> {
    {
        let stream = TcpStream::connect("example.com:80")?;
        println!("Connected!");
        // Use stream...
    } // stream goes out of scope here
    
    // At this point, Rust has dropped the stream
    // This triggered the four-way close handshake
    println!("Connection closed gracefully");
    
    Ok(())
}
```

## Why This Matters

Understanding the three-way handshake helps you:
- **Debug connection issues**: "The connection times out" often means the SYN isn't reaching the server or the SYN-ACK isn't coming back
- **Optimize performance**: Every connection requires a round trip. That's why HTTP/2 and HTTP/3 reuse connections—to avoid repeated handshakes
- **Understand latency**: A connection to a server 100ms away requires at least 100ms for the handshake alone, before any data is sent
- **Secure your server**: Knowing how SYN floods work helps you implement proper defenses

## What's Next?

Now that you understand how TCP establishes connections, you might be wondering: what happens to the connection after it's established? How does the connection know what state it's in?

That's where socket states come in, which we'll cover in the next section. The socket lifecycle includes many states beyond just "connected" and "closed," and understanding these states will help you write more robust networked applications.

The handshake is just the beginning of the connection's journey.
