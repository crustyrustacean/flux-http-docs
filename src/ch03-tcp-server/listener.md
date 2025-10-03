# Creating a Listener

Time to write our first real networking code. We're going to create a `TcpListener` and bind it to an address. This is the foundation of every TCP server—without a listener, you can't accept connections.

## The Simplest Listener

Here's the absolute minimum code to create a listener:

```rust
use std::net::TcpListener;

fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Server listening on port 8080");
    
    Ok(())
}
```

That's it. Two lines of actual code, and you've got a server listening on port 8080. Of course, it doesn't *do* anything yet—we haven't told it to accept connections. But it's listening, and you've successfully claimed port 8080 on your machine.

Let's break down what's happening here.

## Understanding bind()

The `bind()` method is doing three things behind the scenes:

1. **Creating a socket** - Asks the operating system for a new TCP socket
2. **Binding to an address** - Claims a specific IP address and port
3. **Setting it to listen mode** - Tells the OS you want to accept incoming connections

Remember from Chapter 2 that these are the three steps to set up a listening socket? Rust bundles them into one convenient method call.

The string `"127.0.0.1:8080"` is an address in the format `IP:PORT`. Let's talk about what these mean.

## Choosing an Address

### Localhost: 127.0.0.1

Using `127.0.0.1` means your server only accepts connections from the same machine. This is perfect for development—you can test your server locally without exposing it to the network.

Think of it like putting a "Staff Only" sign on your shop's back door. Only people already inside the building (your computer) can knock on that door.

```rust
let listener = TcpListener::bind("127.0.0.1:8080")?;
```

You can also use the hostname `localhost`, which resolves to `127.0.0.1`:

```rust
let listener = TcpListener::bind("localhost:8080")?;
```

### All Interfaces: 0.0.0.0

If you want your server to accept connections from other machines on the network, bind to `0.0.0.0`:

```rust
let listener = TcpListener::bind("0.0.0.0:8080")?;
```

This special address means "listen on all available network interfaces." If your computer has multiple network connections (Wi-Fi, Ethernet, virtual interfaces), this binds to all of them. It's like opening your shop to the whole street instead of just the back alley.

For our echo server, we'll stick with `127.0.0.1` since we're just learning. When you deploy a real server to production, you'll typically use `0.0.0.0`.

## Choosing a Port

The port number comes after the colon. A few things to know:

- **Ports below 1024** are "privileged ports" and typically require administrator/root access. Ports like 80 (HTTP) and 443 (HTTPS) live here.
- **Ports 1024-49151** are "registered ports" that are loosely associated with specific services
- **Ports 49152-65535** are "dynamic" or "private" ports, free for anyone to use

For development, pick something above 1024 that's unlikely to conflict with other services. `8080`, `3000`, and `8000` are popular choices. I like `8080` because it's easy to remember and it's a common convention for development HTTP servers.

```rust
let listener = TcpListener::bind("127.0.0.1:8080")?;  // Common choice
let listener = TcpListener::bind("127.0.0.1:3000")?;  // Also popular
let listener = TcpListener::bind("127.0.0.1:9999")?;  // Less likely to conflict
```

## What Can Go Wrong?

Notice the `?` operator at the end of `bind()`? That's because binding can fail. The most common reasons:

### Port Already in Use

If another program (or another instance of your server) is already using that port, you'll get an error:

```
Error: Address already in use (os error 48)
```

This is like trying to open a shop in a building that already has a tenant. Only one program can bind to a specific address and port combination at a time.

If this happens, either:
- Stop the other program using that port
- Choose a different port number
- Wait a moment—sometimes the OS takes a few seconds to release a port after a program closes

### Permission Denied

If you try to bind to a privileged port (below 1024) without proper permissions:

```rust
let listener = TcpListener::bind("127.0.0.1:80")?;  // Might fail!
```

You'll get:
```
Error: Permission denied (os error 13)
```

On Linux/macOS, you'd need to run with `sudo`. On Windows, you'd need administrator privileges. For learning, just use ports above 1024.

## Better Error Handling

Instead of letting the error propagate with `?`, you might want to print a helpful message:

```rust
use std::net::TcpListener;

fn main() {
    let listener = match TcpListener::bind("127.0.0.1:8080") {
        Ok(listener) => listener,
        Err(e) => {
            eprintln!("Failed to bind to port 8080: {}", e);
            eprintln!("Make sure the port isn't already in use.");
            return;
        }
    };
    
    println!("Server listening on 127.0.0.1:8080");
    
    // ... rest of the server code
}
```

This gives your users (or future you) a much friendlier experience than a raw panic.

## Parsing Addresses Explicitly

You can also construct a `SocketAddr` explicitly if you need more control:

```rust
use std::net::{TcpListener, SocketAddr};

fn main() -> std::io::Result<()> {
    let addr: SocketAddr = "127.0.0.1:8080".parse()
        .expect("Invalid address");
    
    let listener = TcpListener::bind(addr)?;
    println!("Server listening on {}", addr);
    
    Ok(())
}
```

The `parse()` method can fail if you give it a malformed address, so you need to handle that too. For simple cases, passing a string directly to `bind()` is cleaner.

## Checking Where We're Bound

Once you've successfully created a listener, you can ask it what address it actually bound to:

```rust
let listener = TcpListener::bind("127.0.0.1:8080")?;
let local_addr = listener.local_addr()?;
println!("Listening on {}", local_addr);
```

This is especially useful if you bind to port `0`, which asks the OS to assign you any available port:

```rust
let listener = TcpListener::bind("127.0.0.1:0")?;
let local_addr = listener.local_addr()?;
println!("OS assigned us port {}", local_addr.port());
```

This is handy for testing—you don't have to worry about port conflicts because the OS picks an unused port for you.

## What We've Built

At this point, we have a server that's listening but not accepting connections. It's like a shop with an "Open" sign in the window, but no employee at the counter to greet customers.

If you run this code and then try to connect to it with `curl localhost:8080`, your connection attempt will just hang. The listener is there, but nobody's calling `accept()` yet. (You'll eventually see cURL timeout with an error like "Empty reply from server"—that's expected! We haven't written the code to respond yet.)

In the next section, we'll fix that by actually accepting incoming connections and getting our hands on those `TcpStream` objects we talked about. That's where the real action happens.
