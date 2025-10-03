# Accepting Connections

We have a listener, and it's sitting there waiting. Now we need to actually accept connections when clients knock on our door. This is where `accept()` comes in—it's the method that completes the three-way handshake and gives us a `TcpStream` to communicate with the client.

## The accept() Method

Here's the simplest possible version:

```rust
use std::net::TcpListener;

fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Server listening on port 8080");
    
    let (stream, addr) = listener.accept()?;
    println!("New connection from: {}", addr);
    
    Ok(())
}
```

When you call `accept()`, you get back a tuple with two things:

1. **TcpStream** - The actual connection to the client
2. **SocketAddr** - The client's IP address and port number

The `TcpStream` is what you'll use to read and write data. The `SocketAddr` tells you who connected—useful for logging, security checks, or just being friendly and saying hello.

## What Happens During accept()

Remember the three-way handshake from Chapter 2? Here's what's happening:

1. Your program calls `accept()` and **blocks** (stops and waits)
2. A client sends a SYN packet to your server
3. Your OS sends back a SYN-ACK
4. The client sends an ACK
5. The handshake is complete, and `accept()` returns with a connected `TcpStream`

By the time you have a `TcpStream` in your hands, the TCP connection is fully established. You're ready to send and receive data immediately.

## Blocking Behavior

Let's talk about that blocking thing. When you call `accept()`, your entire program stops and waits until a client connects. It's like a shopkeeper standing at the door, staring outside, doing absolutely nothing until a customer walks in.

Try running the code above:

```bash
cargo run
```

You'll see:
```
Server listening on port 8080
```

And then... nothing. The program is frozen, waiting. Open another terminal and connect with `curl`:

```bash
curl localhost:8080
```

Immediately, your server will print:
```
New connection from: 127.0.0.1:54321
```

(The port number will be different—that's the client's ephemeral port assigned by the OS.)

The `curl` command will hang too, waiting for a response we're not sending yet. That's fine! We'll implement the echo logic in the next section.

## Accepting Multiple Connections

The code above only accepts one connection and then exits. That's not very useful for a server. We need a loop:

```rust
use std::net::TcpListener;

fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Server listening on port 8080");
    
    loop {
        let (stream, addr) = listener.accept()?;
        println!("New connection from: {}", addr);
        
        // TODO: Handle the connection
        // For now, it just immediately closes
    }
}
```

Now your server will accept a connection, print a message, and immediately go back to waiting for the next one. The `TcpStream` goes out of scope at the end of each loop iteration, which automatically closes the connection.

This is the basic structure of most TCP servers: an infinite loop that accepts connections one at a time.

## The Problem With Sequential Processing

There's a catch with this approach. Because `accept()` blocks, you can only handle one client at a time. While you're processing one connection (reading data, doing computation, writing a response), no other clients can connect. They have to wait in line.

For our simple echo server, this is actually fine! We'll address it properly when we get to concurrency in Part III. For now, we're focusing on the fundamentals.

## Using incoming() Iterator

Rust provides a nicer way to write this loop using the `incoming()` method, which returns an iterator:

```rust
use std::net::TcpListener;

fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Server listening on port 8080");
    
    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                let addr = stream.peer_addr()?;
                println!("New connection from: {}", addr);
                // TODO: Handle the connection
            }
            Err(e) => {
                eprintln!("Connection failed: {}", e);
            }
        }
    }
    
    Ok(())
}
```

This is more idiomatic Rust. The `incoming()` method returns an iterator that yields `Result<TcpStream>`, so you get error handling built in. If accepting a connection fails (rare, but possible), you can log the error and keep accepting other connections.

Note that `incoming()` only gives you the stream, not the address. If you need the client's address, call `stream.peer_addr()` to get it.

## Getting Connection Information

Once you have a `TcpStream`, you can ask it questions about the connection:

```rust
let (stream, _) = listener.accept()?;

// Client's address and port
let peer_addr = stream.peer_addr()?;
println!("Client: {}", peer_addr);

// Our server's local address (usually what we bound to)
let local_addr = stream.local_addr()?;
println!("Server: {}", local_addr);
```

This is useful for logging. In a real server, you'd probably log every connection:

```rust
for stream in listener.incoming() {
    match stream {
        Ok(stream) => {
            match stream.peer_addr() {
                Ok(addr) => println!("[{}] Connection established", addr),
                Err(_) => println!("Connection established (unknown address)"),
            }
            // Handle connection...
        }
        Err(e) => {
            eprintln!("Failed to accept connection: {}", e);
        }
    }
}
```

## What Can Go Wrong?

### Too Many Open Files

Every connection uses a file descriptor (remember, sockets are just special files to the OS). If you accept thousands of connections without closing them, you might run out:

```
Error: Too many open files (os error 24)
```

This shouldn't happen with our echo server because connections close quickly. But it's something to be aware of in production systems. Operating systems have limits (often around 1024 file descriptors per process by default).

### Interrupted System Calls

Sometimes `accept()` can be interrupted by a signal (like Ctrl+C):

```
Error: Interrupted system call (os error 4)
```

In production code, you might want to retry if this happens. For learning purposes, letting the program exit is fine.

## Testing It Out

Let's write a version that's actually useful to test with:

```rust
use std::net::TcpListener;
use std::io::{Read, Write};

fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Server listening on port 8080");
    
    for stream in listener.incoming() {
        match stream {
            Ok(mut stream) => {
                let addr = stream.peer_addr()?;
                println!("New connection from: {}", addr);
                
                // Send a quick response so curl doesn't hang
                stream.write_all(b"Connection accepted!\n")?;
                
                println!("Connection closed");
            }
            Err(e) => {
                eprintln!("Connection failed: {}", e);
            }
        }
    }
    
    Ok(())
}
```

Run this and test it with `curl`:

```bash
curl localhost:8080
```

You should see:
```
Connection accepted!
```

And your server logs:
```
New connection from: 127.0.0.1:54123
Connection closed
```

Perfect! We're accepting connections, sending a tiny response, and closing gracefully.

## What's Next

We can now accept connections, but we're not doing anything interesting with them yet. We send a hardcoded message and close immediately. That's not much of an echo server.

In the next section, we'll learn how to actually read data from the client and write it back—the core of the echo functionality. That's where `TcpStream` really shines.
