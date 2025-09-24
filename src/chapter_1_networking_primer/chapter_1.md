# Chapter 1 - Networking Primer

Building network aware applications in Rust, even at a low level, is pretty straightforward. The standard library gives you the building blocks you need.

### Introduction

Rust is a low level language, but you can choose your level of low. The standard library affords several tools that make your networking journey easy to start. In this quick tour, we're going to:

- learn what elements from the standard library we can use
- learn how to make a socket
- learn how to accept incoming connections on this socket
- learn how to handle errors, at least in a rudimentary way
- understand where to go to build out from this foundation

Let's go!

### The Code

Here is the code, then I'll break it down step by step:

```Rust
// src/main.rs

// dependencies
use std::net::{Ipv4Addr, TcpListener};

// main function
fn main() {
    // create an IPv4 address
    let address = Ipv4Addr::new(127, 0, 0, 1);

    // create port
    let port: u16 = 8000;

    // create a socket from the address and port we just made
    let socket = format!("{}:{}", address, port);

    // create a listener and give it the socket we just made to listen on, panic and crash out if there's an error
    // returned by the bind method
    let listener = match TcpListener::bind(socket) {
        Ok(ltnr) => ltnr,
        Err(e) => panic!(
            "Unable to obtain a listener on which to receive incoming connections: {}",
            e
        ),
    };

    // start a loop and accept incoming connections, match on an errors and print them to stderr
    for stream in listener.incoming() {
        match stream {
            Ok(_stream) => {
                println!("Connection established!");
            }
            Err(e) => eprintln!("Connection failed: {}", e),
        }
    }
}
```

Alright, what are we doing here.

- bring into scope some dependencies from the standard library, namely:
  - the struct type Ipv4Addr, gives us tools to create an IpV4 address
  - the TcpListener type
- we the start up a main function and create an Ipv4 address which we bind to a variable called `address`
- we create a `u16` type to represent a port, with the value 8000, and bind to a variable called `port`
- a socket can be created by combining the address and port, using the `format!` macro which creates a new string type from the two separate values
- armed with a socket, we create a `listener` by using the `bind()` method available for the `Tcplistener` type, the `bind` method takes our socket as an argument
- the bind method can fail, se we match on it
  - success (the `Ok` arm of the match statement) binds the returned listener to a variable called `ltnr`
  - the error path (the `Err` arm of the match statement) causes the program to panic with an error message. In reality, you'll want to handle errors potentially more gracefully. Although there's something to be said from just quitting, because with out a listener to receive data, there's not much point in continuing
- now that we have something to listen on, we start up a loop and use the `incoming()` method on our listener, which accepts connections in a blocking manner
- we bind the incoming data to a variable called `stream`
- we have to match here again, because the `incoming()` method could fail
  - success (the `Ok` arm) we just print out a message confirming we've established a connnection
  - the error path (the `Err` arm) we print out a message to `stderr` giving a reason why the connection failed

### Where to Go Next

This program we've created isn't particularly useful. Yes, we accept incoming connections and we somewhat handle errors, but we don't do anything with the information coming to us. Where we can we go from here?

Well, we can build out some functions which take the stream of data and do something with it. In a future article, I'll illustrate how to do that.

I alluded above to the fact this code is blocking. This means every request will block the next until the first is fully processed. You can extend this code by creating a simple event loop, using the `tokio` crate, to handle connections in an asynchronous manner.

In the next chapters we're going to see how to do all that, but for now, let's build out what we've done so far and try to make it a little more useful.


