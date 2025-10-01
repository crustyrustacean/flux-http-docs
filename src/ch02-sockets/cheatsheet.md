
# Chapter 2 Cheatsheet: Sockets and Connections

## What is a Socket?

**Definition**: A socket is your program's interface to the network—the programming API that lets you send and receive data over network connections.

**Key Concepts:**
- Sockets are like telephones for your application
- On Unix-like systems, sockets are treated as file descriptors
- You can read from and write to sockets like files
- The OS handles the complexity of packets, retransmission, and reassembly

**Basic Socket Operations:**
- **Create**: Allocate a new socket
- **Bind**: Associate socket with an address and port
- **Listen**: Prepare to accept incoming connections (server)
- **Accept**: Accept an incoming connection (server)
- **Connect**: Establish connection to a server (client)
- **Read/Write**: Exchange data
- **Close**: Terminate the connection

## TCP vs UDP Quick Reference

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Required (connection-oriented) | Not required (connectionless) |
| **Reliability** | Guaranteed delivery | No guarantees, packets can be lost |
| **Ordering** | Packets arrive in order | Packets may arrive out of order |
| **Speed** | Slower (overhead) | Faster (minimal overhead) |
| **Header Size** | 20+ bytes | 8 bytes |
| **Use When** | Correctness matters | Speed matters more than completeness |
| **Error Handling** | Automatic retransmission | No retransmission |
| **Flow Control** | Yes | No |

**TCP Use Cases:**
- Web servers (HTTP/HTTPS)
- Email (SMTP, IMAP)
- File transfer (FTP, SSH)
- Database connections
- REST APIs

**UDP Use Cases:**
- Online gaming
- Video/audio streaming
- VoIP calls
- DNS queries
- IoT sensor data

## The Three-Way Handshake

**Purpose**: Establish a TCP connection with sequence numbers and agreed parameters.

**The Steps:**

1. **SYN** (Client → Server)
   - Client sends SYN with initial sequence number X
   - Client enters `SYN_SENT` state
   - "Hey! Want to chat?"

2. **SYN-ACK** (Server → Client)
   - Server sends SYN-ACK with its sequence number Y, acknowledges X
   - Server enters `SYN_RECEIVED` state
   - "Sure! I heard you. Ready when you are!"

3. **ACK** (Client → Server)
   - Client sends ACK, acknowledges Y
   - Both enter `ESTABLISHED` state
   - "Great, I heard you too. Let's start!"

**Why Three Steps?**
- Ensures both sides confirm they've heard each other
- Prevents half-open connections if packets are lost
- Synchronizes sequence numbers for reliable data transfer

**Timing:**
- Local connection: microseconds
- Remote connection: 100-300ms (1× round-trip time)
- Includes timeouts and retries if packets are lost

## Socket States Reference

### Main States (Simplified Path)

**Server Path:**
```
CLOSED → LISTEN → SYN_RECEIVED → ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED
```

**Client Path:**
```
CLOSED → SYN_SENT → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED
```

### State Descriptions

| State | Description | Who | Duration |
|-------|-------------|-----|----------|
| **CLOSED** | Initial state, no connection | Both | Starting point |
| **LISTEN** | Waiting for connection requests | Server | Until shutdown |
| **SYN_SENT** | Sent SYN, waiting for SYN-ACK | Client | 1-3 seconds per retry |
| **SYN_RECEIVED** | Received SYN, sent SYN-ACK | Server | Milliseconds |
| **ESTABLISHED** | Connection open, data flowing | Both | Until someone closes |
| **FIN_WAIT_1** | Sent FIN, waiting for ACK | Closer | Brief |
| **FIN_WAIT_2** | Got ACK, waiting for FIN | Closer | Until other side closes |
| **CLOSE_WAIT** | Received FIN, can still send | Receiver | Until app closes |
| **CLOSING** | Both sent FIN simultaneously | Both | Brief (rare) |
| **LAST_ACK** | Waiting for final ACK of FIN | Receiver | Brief |
| **TIME_WAIT** | Waiting to ensure clean close | Closer | 30-120 seconds |

### Critical States to Watch

**TIME_WAIT:**
- Lasts 30-120 seconds after closing
- Prevents port reuse issues
- Can cause "Address already in use" errors
- Workaround: Set `SO_REUSEADDR` socket option

**CLOSE_WAIT:**
- Indicates you received a FIN but didn't close your side
- Too many = socket leak in your application
- Fix: Ensure all `TcpStream` objects are properly dropped

**SYN_SENT:**
- Connection attempt in progress
- Timeout = server not responding or unreachable
- Retry automatically happens at TCP layer

## Common Rust Patterns

### Creating a TCP Server
```rust
let listener = TcpListener::bind("127.0.0.1:8080")?;
for stream in listener.incoming() {
    match stream {
        Ok(stream) => { /* handle connection */ }
        Err(e) => { /* handle error */ }
    }
}
```

### Creating a TCP Client
```rust
let mut stream = TcpStream::connect("127.0.0.1:8080")?;
stream.write_all(b"Hello")?;
let mut buffer = [0; 512];
let n = stream.read(&mut buffer)?;
```

### Creating a UDP Socket
```rust
let socket = UdpSocket::bind("127.0.0.1:8080")?;
let (bytes, src) = socket.recv_from(&mut buffer)?;
socket.send_to(&buffer[..bytes], src)?;
```

### Setting Socket Options
```rust
use socket2::{Socket, Domain, Type};

let socket = Socket::new(Domain::IPV4, Type::STREAM, None)?;
socket.set_reuse_address(true)?;
socket.set_nodelay(true)?; // Disable Nagle's algorithm
socket.bind(&addr.into())?;
```

## Error Handling Quick Reference

| Error | Meaning | Common Cause |
|-------|---------|--------------|
| "Connection refused" | Server sent RST | No listener on that port |
| "Connection reset" | Received RST during transfer | Other side crashed/closed abruptly |
| "Connection timeout" | No response | Server down, network issue, firewall |
| "Address already in use" | Port unavailable | Another process or TIME_WAIT socket |
| "Broken pipe" | Write to closed socket | Other side closed connection |
| "Network unreachable" | Cannot route to address | Network/routing problem |

## Key Takeaways

✓ **Sockets are your network interface** – abstracts away packet complexity

✓ **TCP = reliable, UDP = fast** – choose based on your requirements

✓ **Three-way handshake = connection setup** – happens automatically, costs 1 RTT

✓ **Socket states matter** – especially TIME_WAIT and CLOSE_WAIT

✓ **Always handle errors** – network operations can fail in many ways

✓ **Drop sockets properly** – prevents resource leaks and stuck CLOSE_WAIT states

✓ **One listening socket, many connection sockets** – the listener creates new sockets for each client

## Debugging Commands

**View socket states:**
```bash
# Linux/macOS
netstat -an | grep 8080
ss -tan | grep 8080

# Windows
netstat -an | findstr 8080
```

**Count sockets by state:**
```bash
# Linux
ss -tan | awk '{print $1}' | sort | uniq -c

# macOS
netstat -an | grep ESTABLISHED | wc -l
```