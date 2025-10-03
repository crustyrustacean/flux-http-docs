# Rust's std::net

Before we start writing code, let's take a look at what Rust gives us out of the box. The [`std::net`](https://doc.rust-lang.org/std/net/index.html) module is your networking toolkit—it wraps all the low-level system calls for creating sockets, binding to addresses, and sending data over the network.

Remember back in Chapter 2 when we talked about sockets being endpoints for communication? Well, Rust doesn't make you deal with raw file descriptors or cryptic system calls. Instead, it gives you clean, type-safe abstractions that handle the messy details for you.

## The Core Types

Here are the main players we'll be working with:

### TcpListener

This is your server's front door. A [`TcpListener`](https://doc.rust-lang.org/std/net/struct.TcpListener.html) binds to an address and port, then listens for incoming connections. Think of it as the receptionist at a hotel—it sits at the entrance and waits for guests to arrive.

When you create a `TcpListener`, you're doing what we described in Chapter 2: creating a socket, binding it to an address, and putting it in listening mode. Rust just wraps all three steps into a nice ergonomic API.

```rust
use std::net::TcpListener;

let listener = TcpListener::bind("127.0.0.1:8080")?;
```

That one line does a lot: it creates a socket, binds it to localhost on port 8080, and starts listening. Under the hood, this is calling `socket()`, `bind()`, and `listen()` from the operating system, but you don't have to think about that.

### TcpStream

A [`TcpStream`](https://doc.rust-lang.org/std/net/struct.TcpStream.html) represents an active TCP connection between your server and a client. This is where the actual data flows. When a client connects to your `TcpListener`, you get a `TcpStream` that you can read from and write to.

Remember the three-way handshake from Chapter 2? By the time you have a `TcpStream` in your hands, that handshake has already completed. The connection is established, and you're ready to communicate.

```rust
// Reading from a stream
let mut buffer = [0; 512];
stream.read(&mut buffer)?;

// Writing to a stream
stream.write(b"Hello, client!")?;
```

`TcpStream` implements both `Read` and `Write` traits, which means you can use all the familiar I/O operations you'd use with files. This is one of Rust's nice design choices—network streams and file streams use the same interface.

### SocketAddr and IpAddr

These types represent network addresses. A [`SocketAddr`](https://doc.rust-lang.org/std/net/enum.SocketAddr.html) is an IP address plus a port number—exactly what we learned about in Chapter 1 when we covered IP addresses and ports.

```rust
use std::net::{SocketAddr, IpAddr, Ipv4Addr};

// Create an IPv4 socket address
let addr = SocketAddr::from(([127, 0, 0, 1], 8080));

// Or parse from a string
let addr: SocketAddr = "127.0.0.1:8080".parse()?;
```

You'll mostly use these when you want to know who connected to your server or when you need to bind to a specific address.

## How It All Fits Together

Here's the basic flow we'll be implementing in our echo server:

1. **Create a TcpListener** and bind it to an address (like `127.0.0.1:8080`)
2. **Accept incoming connections** - this gives us a `TcpStream` for each client
3. **Read data** from the `TcpStream` into a buffer
4. **Write data** back to the `TcpStream` (echoing what we received)
5. **Handle errors** gracefully when things go wrong

Each of these steps maps directly to the networking concepts we covered earlier. The `TcpListener` is our listening socket from Chapter 2. The `accept()` call completes the three-way handshake. The `TcpStream` is the established connection, and reading/writing sends data through the TCP/IP stack we learned about in Chapter 1.

## What Rust Handles For You

One of the best things about `std::net` is what you *don't* have to worry about:

- **Platform differences** - The same code works on Linux, macOS, Windows, and more
- **Socket creation** - No manual `socket()` system calls
- **Memory safety** - No buffer overflows or use-after-free bugs
- **Resource cleanup** - Sockets are automatically closed when they go out of scope
- **Endianness** - Network byte order is handled automatically for port numbers

Rust's ownership system means that when a `TcpStream` goes out of scope, it automatically closes the underlying socket. No resource leaks, no forgetting to call `close()`. This is huge for writing reliable server code.

## A Quick Note on Blocking

By default, operations on `TcpListener` and `TcpStream` are **blocking**. When you call `accept()` on a listener, your program stops and waits until a client connects. When you call `read()` on a stream, it waits until data arrives.

This is actually fine for our echo server! We'll talk about the problems with blocking I/O in Chapter 10, and we'll explore solutions like threads in Chapter 11 and async in Chapter 14. But for now, blocking behavior keeps things simple while we learn the basics.

---

Alright, we've got our toolkit. In the next section, we'll use `TcpListener` to set up our server and start accepting connections. Let's write some code.
