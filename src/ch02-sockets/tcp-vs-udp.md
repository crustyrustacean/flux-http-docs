
# TCP vs UDP

Now that you understand what a socket is, let's talk about the two main types of sockets you'll actually use: TCP sockets and UDP sockets. They both let you send data over a network, but they work in fundamentally different ways—and choosing the wrong one for your use case can lead to some serious headaches.

## The Postal Service vs. The Phone Call

Here's an analogy that captures the essence of the difference:

**TCP (Transmission Control Protocol)** is like making a phone call. Before you can talk, you need to establish a connection—the phone rings, someone answers, you say hello. Once connected, everything you say arrives in order. If the line gets garbled, you ask the person to repeat themselves. When you're done, you say goodbye and hang up. It's reliable, orderly, but has some overhead.

**UDP (User Datagram Protocol)** is like sending postcards. You write a message, slap on an address, and drop it in the mailbox. You don't know if it arrived. You don't know what order multiple postcards will arrive in. You don't even know if the recipient is home. But it's fast and simple—you don't need to establish anything before sending.

## TCP: The Reliable Choice

TCP is a **connection-oriented** protocol. This means before any data is exchanged, a connection must be established between the client and server. Once that connection exists, TCP provides several guarantees:

**Reliability**: Every byte you send will arrive, or you'll get an error telling you the connection failed. TCP handles packet loss by detecting missing packets and requesting retransmission.

**Ordering**: Data arrives in the exact order you sent it. If packet 3 arrives before packet 2, TCP will hold packet 3 until packet 2 shows up, then deliver them in order.

**Flow Control**: TCP prevents a fast sender from overwhelming a slow receiver by adjusting the sending rate.

**Error Checking**: Built-in checksums ensure data isn't corrupted in transit.

The tradeoff? TCP has overhead. That initial connection setup (the three-way handshake), the acknowledgment packets flying back and forth, the buffering for reordering—it all takes time and bandwidth.

Here's what a simple TCP server looks like in Rust:

```rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write};

fn handle_client(mut stream: TcpStream) -> std::io::Result<()> {
    // Allocate a buffer to read data into
    let mut buffer = [0; 1024];
    
    // Read data from the client
    // This blocks until data arrives or the connection closes
    let bytes_read = stream.read(&mut buffer)?;
    
    if bytes_read == 0 {
        // Connection was closed by the client
        println!("Client disconnected");
        return Ok(());
    }
    
    // Echo the data back to the client
    // write_all ensures all bytes are sent, retrying if needed
    stream.write_all(&buffer[..bytes_read])?;
    
    Ok(())
}

fn main() -> std::io::Result<()> {
    // Bind to localhost on port 8080
    // This can fail if the port is already in use
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("TCP server listening on port 8080");
    
    // Accept connections in a loop
    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                println!("New connection from: {}", 
                    stream.peer_addr().unwrap_or_else(|_| "unknown".parse().unwrap()));
                
                // Handle this client
                // In production, you'd spawn a thread or async task here
                if let Err(e) = handle_client(stream) {
                    eprintln!("Error handling client: {}", e);
                }
            }
            Err(e) => {
                eprintln!("Connection failed: {}", e);
            }
        }
    }
    
    Ok(())
}
```

Notice how we're using TCP? The connection is established before any data flows, and we can read and write like it's a two-way pipe.

## UDP: The Fast and Loose Choice

UDP is **connectionless**. There's no handshake, no connection state, no promises. You just send packets (called **datagrams**) into the void and hope they arrive.

What UDP provides:
- **Speed**: No connection setup, no acknowledgments, no waiting
- **Simplicity**: Less state to manage
- **Low overhead**: Smaller packet headers

What UDP doesn't provide:
- **No reliability**: Packets can get lost, and UDP won't tell you
- **No ordering**: Packet 2 might arrive before packet 1
- **No flow control**: You can overwhelm the receiver
- **No connection state**: Each packet is independent

UDP is perfect for situations where:
- You can tolerate some data loss (video streaming, online games)
- Speed matters more than reliability
- You're sending small, self-contained messages
- You're implementing your own reliability on top (some protocols do this)

Here's a UDP server:

```rust
use std::net::UdpSocket;

fn main() -> std::io::Result<()> {
    // Bind to localhost on port 8080
    // Unlike TCP, this doesn't "listen"—it just reserves the address
    let socket = UdpSocket::bind("127.0.0.1:8080")?;
    println!("UDP server bound to port 8080");
    
    // Buffer to receive data into
    let mut buffer = [0; 1024];
    
    loop {
        // Receive a datagram
        // This blocks until a packet arrives
        // Returns the number of bytes and the sender's address
        match socket.recv_from(&mut buffer) {
            Ok((bytes_received, source_addr)) => {
                println!("Received {} bytes from {}", bytes_received, source_addr);
                
                // Echo the data back to whoever sent it
                // send_to might fail if the network is unreachable
                // Note: there's no guarantee this packet will arrive!
                if let Err(e) = socket.send_to(&buffer[..bytes_received], source_addr) {
                    eprintln!("Failed to send response: {}", e);
                }
            }
            Err(e) => {
                eprintln!("Error receiving data: {}", e);
            }
        }
    }
}
```

See the difference? No connection acceptance, no stream. We just receive datagrams and send datagrams. Each datagram includes the sender's address, so we know where to reply.

And here's a UDP client:

```rust
use std::net::UdpSocket;

fn main() -> std::io::Result<()> {
    // Bind to any available port on localhost
    // The OS will assign us a random port
    let socket = UdpSocket::bind("0.0.0.0:0")?;
    
    // Set a timeout so we don't wait forever for a response
    socket.set_read_timeout(Some(std::time::Duration::from_secs(5)))?;
    
    let message = b"Hello, UDP server!";
    let server_addr = "127.0.0.1:8080";
    
    // Send the message
    // This doesn't establish a connection—it just fires off a packet
    socket.send_to(message, server_addr)?;
    println!("Sent message to {}", server_addr);
    
    // Wait for a response
    let mut buffer = [0; 1024];
    match socket.recv_from(&mut buffer) {
        Ok((bytes_received, source)) => {
            println!("Received response from {}: {}", 
                source,
                String::from_utf8_lossy(&buffer[..bytes_received]));
        }
        Err(e) => {
            // This could be a timeout or a network error
            eprintln!("No response received: {}", e);
        }
    }
    
    Ok(())
}
```

Notice we set a timeout? With UDP, there's no guarantee we'll get a response. Our packet might get lost, or the server's response might get lost. Without a timeout, we'd wait forever.

## The Real Difference: Guarantees vs. Speed

Let's make this concrete with a table:

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Required (three-way handshake) | Not required |
| Reliability | Guaranteed delivery | No guarantees |
| Ordering | Packets arrive in order | Packets may arrive out of order |
| Speed | Slower (more overhead) | Faster (minimal overhead) |
| Use case | Web, email, file transfer | Streaming, gaming, DNS |
| Header size | 20+ bytes | 8 bytes |
| Error checking | Yes, with retransmission | Yes, but no retransmission |

## When to Use TCP

Use TCP when:
- **Correctness matters**: You're transferring files, financial data, or anything where losing even a single byte is unacceptable
- **Order matters**: You're sending a sequence of commands that must be processed in order
- **You want simplicity**: TCP's guarantees mean you can write simpler application code—you don't need to handle retransmission, reordering, or detecting lost packets yourself

**Real-world TCP applications:**
- Web servers (HTTP/HTTPS)
- Email (SMTP, IMAP)
- File transfer (FTP, SFTP)
- SSH connections
- Database connections
- REST APIs

## When to Use UDP

Use UDP when:
- **Speed matters more than completeness**: You're streaming video or audio where a dropped frame is better than a delayed frame
- **Low latency is critical**: Online games where a 50ms delay matters
- **Small messages**: DNS queries that fit in a single packet
- **Broadcast/multicast**: Sending the same data to multiple recipients
- **You'll implement your own reliability**: Some protocols use UDP but add their own reliability layer (like QUIC)

**Real-world UDP applications:**
- Online gaming
- Video/audio streaming
- VoIP (voice calls)
- DNS queries
- Network management (SNMP)
- IoT sensor data

## A Common Misconception

People sometimes think UDP is "unreliable" in the sense that it's buggy or bad. That's not true at all. UDP is *deliberately* unreliable—it's designed that way for good reasons. It's a tool for specific jobs, just like TCP is a tool for different jobs.

Think of it this way: a race car and a cargo truck are both vehicles, but you wouldn't use a race car to move furniture, and you wouldn't race a cargo truck. TCP is the cargo truck: slower but carries everything safely. UDP is the race car: fast but carries less and might skip some stops.

## Can You Mix Them?

Absolutely! Many applications use both. For example:
- A video chat app might use UDP for the audio/video streams (where dropped packets cause minor glitches) and TCP for chat messages (where every message must arrive)
- An online game might use UDP for player position updates (constant stream, old data becomes irrelevant) and TCP for in-game purchases (critical, must be reliable)

In Rust, you can have both socket types running simultaneously—they're just different types (`TcpStream` vs `UdpSocket`).

## What's Next?

Now you understand the fundamental difference between TCP and UDP. In the rest of this chapter, we'll focus primarily on TCP because:
1. It's what you'll use for most applications
2. It's what HTTP is built on (and we're building an HTTP server)
3. Understanding TCP connections deeply will make you a better network programmer

In the next section, we'll dive into exactly how TCP establishes a connection with the famous three-way handshake. It's more interesting than it sounds, I promise.