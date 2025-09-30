# Chapter 3 - Single Threaded Blocking HTTP Server

```rust
// src/main.rs

// dependencies
use std::collections::HashMap;
use std::env;
use std::io::{Read, Write};
use std::net::{TcpListener, TcpStream};

// Windows only dependencies
#[cfg(windows)]
use std::os::raw::c_int;
use std::sync::atomic::{AtomicBool, Ordering};

// configure the server's buffer, for now it's 1KB
const BUFFER_SIZE: usize = 1024;

// enum type to represent the HTTP method
#[allow(dead_code)]
#[derive(Debug)]
enum Method {
    Get,
    Post,
    Put,
    Delete,
}

// enum type to represent a parsing error
#[derive(Debug)]
enum ParseError {
    InvalidRequest,
    InvalidMethod,
    MissingRequestLine,
}

// struct type to model an incoming HTTP request
#[allow(dead_code)]
#[derive(Debug)]
struct Request {
    method: Method,
    path: String,
    version: String,
    headers: HashMap<String, String>,
    body: Vec<u8>,
}

// struct type to model an outgoing HTTP response
#[allow(dead_code)]
#[derive(Debug)]
struct Response {
    status_code: u16,
    status_text: String,
    headers: HashMap<String, String>,
    body: Vec<u8>,
}

// methods for the Response type
#[allow(dead_code)]
impl Response {
    fn new(status_code: u16, status_text: &str) -> Self {
        Response {
            status_code,
            status_text: status_text.to_string(),
            headers: HashMap::new(),
            body: Vec::new(),
        }
    }

    fn ok() -> Self {
        Self::new(200, "OK")
    }

    fn not_found() -> Self {
        Self::new(404, "Not Found")
    }

    fn with_header(mut self, key: &str, value: &str) -> Self {
        self.headers.insert(key.to_string(), value.to_string());
        self
    }

    fn with_body(mut self, body: Vec<u8>) -> Self {
        self.body = body;
        self
    }

    fn with_text_body(mut self, text: &str) -> Self {
        self.body = text.as_bytes().to_vec();
        self
    }

    fn to_bytes(&self) -> Vec<u8> {
        let mut response = format!("HTTP/1.1 {} {}\r\n", self.status_code, self.status_text);

        // Add Content-Length header automatically
        response.push_str(&format!("Content-Length: {}\r\n", self.body.len()));

        // Add all headers
        for (key, value) in &self.headers {
            response.push_str(&format!("{}: {}\r\n", key, value));
        }

        // Blank line separates headers from body
        response.push_str("\r\n");

        // Combine header string with body bytes
        let mut bytes = response.into_bytes();
        bytes.extend_from_slice(&self.body);

        bytes
    }
}

// function to parse the incoming buffer of bytes into our Request type
fn parse_request(buffer: &[u8]) -> Result<Request, ParseError> {
    let request_str = std::str::from_utf8(buffer).map_err(|_| ParseError::InvalidRequest)?;
    let (headers_part, body_part) = request_str
        .split_once("\r\n\r\n")
        .ok_or(ParseError::InvalidRequest)?;

    let mut lines = headers_part.lines();
    let request_line = lines.next().ok_or(ParseError::MissingRequestLine)?;

    let mut parts = request_line.split_whitespace();

    let method_str = parts.next().ok_or(ParseError::InvalidRequest)?;
    let path = parts.next().ok_or(ParseError::InvalidRequest)?;
    let version = parts.next().ok_or(ParseError::InvalidRequest)?;

    let method = match method_str {
        "GET" => Method::Get,
        "POST" => Method::Post,
        "PUT" => Method::Put,
        "DELETE" => Method::Delete,
        _ => return Err(ParseError::InvalidMethod),
    };

    let mut headers = HashMap::new();
    for line in lines {
        if let Some((key, value)) = line.split_once(": ") {
            headers.insert(key.to_lowercase(), value.to_string());
        }
    }

    Ok(Request {
        method,
        path: path.to_string(),
        version: version.to_string(),
        headers,
        body: body_part.as_bytes().to_vec(),
    })
}

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
    stream.set_read_timeout(Some(std::time::Duration::from_millis(500)))?;
    let mut buffer = [0u8; BUFFER_SIZE];
    // Retry reading on timeout
    let bytes_read = loop {
        match stream.read(&mut buffer) {
            Ok(n) => break n,
            Err(ref e) if e.kind() == std::io::ErrorKind::TimedOut => {
                // Check shutdown flag
                #[cfg(windows)]
                if !RUNNING.load(Ordering::SeqCst) {
                    return Ok(()); // Exit gracefully during shutdown
                }
                continue; // Retry read
            }
            Err(e) => return Err(e),
        }
    };

    println!("Connection established.");
    println!("Bytes read: {}", bytes_read);

    let response = match parse_request(&buffer[..bytes_read]) {
        Ok(request) => {
            println!("Parsed request: {:#?}", request);
            Response::ok()
                .with_header("Content-Type", "text/html")
                .with_text_body("<h1>Hello from Flux!</h1>")
        }
        Err(e) => {
            println!("Parse error: {:?}", e);
            Response::new(400, "Bad Request").with_text_body("Invalid HTTP request")
        }
    };

    stream.write_all(&response.to_bytes())?;
    stream.flush()?;

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
