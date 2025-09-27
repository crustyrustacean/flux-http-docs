# Chapter 2 - Raw TCP Server

```rust
// src/main.rs

// dependencies
use std::env;
use std::io::Read;
use std::net::{TcpListener, TcpStream};

// Windows only dependencies
#[cfg(windows)]
use std::os::raw::c_int;
use std::sync::atomic::{AtomicBool, Ordering};

// configure the server's buffer, for now it's 1KB
const BUFFER_SIZE: usize = 1024;

#[cfg(windows)]
#[link(name = "kernel32")]
unsafe extern "system" {
    fn SetConsoleCtrlHandler(
        handler: Option<unsafe extern "system" fn(c_int) -> c_int>,
        add: c_int,
    ) -> c_int;
}

#[cfg(windows)]
static RUNNING: AtomicBool = AtomicBool::new(true);

#[cfg(windows)]
unsafe extern "system" fn ctrl_handler(_: c_int) -> c_int {
    RUNNING.store(false, Ordering::SeqCst);
    1
}

fn build_listener(addr: &str, port: u16) -> std::io::Result<TcpListener> {
    TcpListener::bind(format!("{}:{}", addr, port))
}

fn handle_connection(mut stream: TcpStream) -> std::io::Result<()> {
    stream.set_nonblocking(false)?;
    let mut buffer = [0u8; BUFFER_SIZE];
    let bytes_read = stream.read(&mut buffer)?;
    println!("Connection established.");
    println!("Bytes read: {}", bytes_read);
    println!(
        "Received data:\n{}",
        String::from_utf8_lossy(&buffer[..bytes_read])
    );
    Ok(())
}

fn event_loop<F>(listener: TcpListener, handler: F) -> std::io::Result<()>
where
    F: Fn(TcpStream) -> std::io::Result<()>,
{
    listener.set_nonblocking(true)?;

    loop {
        #[cfg(windows)]
        if !RUNNING.load(Ordering::SeqCst) {
            println!("Shutting down gracefully...");
            break;
        }

        match listener.accept() {
            Ok((stream, _addr)) => handler(stream)?,
            Err(ref e) if e.kind() == std::io::ErrorKind::WouldBlock => {
                std::thread::sleep(std::time::Duration::from_millis(100));
                continue;
            }
            Err(e) => return Err(e),
        }
    }

    Ok(())
}

fn main() -> std::io::Result<()> {
    // register a shutdown handler
    #[cfg(windows)]
    unsafe {
        SetConsoleCtrlHandler(Some(ctrl_handler), 1);
    }

    // get the host and port from the command line arguments
    let args: Vec<String> = env::args().collect();
    let host = &args[1];
    let port: u16 = args[2]
        .parse()
        .expect("Unable to parse a port value from the command line input");
    
    // get a listener
    let listener = build_listener(host, port)?;

    // run an event loop
    event_loop(listener, handle_connection)?;

    Ok(())
}
```