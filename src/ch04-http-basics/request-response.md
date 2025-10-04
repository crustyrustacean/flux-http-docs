# Request-Response Cycle

Now that we understand the client-server model, let's examine the fundamental pattern that governs all HTTP communication: the **request-response cycle**.

## The Basic Cycle

Every HTTP interaction follows the same simple pattern:

1. **Client sends a request** → "I want this resource" or "I want to perform this action"
2. **Server processes the request** → Reads data, runs code, accesses databases, etc.
3. **Server sends a response** → "Here's what you asked for" or "Here's what happened"
4. **Client receives and processes the response** → Renders page, stores data, shows error, etc.

That's it. This cycle is the heartbeat of HTTP. Whether you're loading a web page, posting a form, or calling an API—it's always request, then response.

## One Request, One Response

A crucial characteristic of HTTP is that **every request gets exactly one response**. Not zero, not two—one.

```
Client: "GET /users"
Server: "200 OK, here are the users"
[cycle complete]

Client: "POST /login"
Server: "200 OK, you're logged in"
[cycle complete]
```

This might seem obvious, but it's a defining constraint of HTTP. The server cannot:
- Send a response without first receiving a request
- Send multiple responses to a single request
- Send a response and then update it later

Once the server sends a response, that particular request-response cycle is complete. If the client wants more data, it must send another request, starting a new cycle.

## The Synchronous Nature

HTTP/1.1 is fundamentally **synchronous** from the client's perspective. When a client sends a request, it waits for the response before that exchange is complete.

Think of it like a conversation where you must wait for an answer:

```
You: "What time is it?"
[waiting...]
Friend: "It's 3 PM"
You: "Thanks. What's the weather like?"
[waiting...]
Friend: "It's sunny"
```

You can't ask your next question until you get an answer. HTTP/1.1 works the same way on a single connection—one request-response cycle must complete before the next begins.

(Note: HTTP/1.1 does support **pipelining** where multiple requests can be sent without waiting for responses, but it's rarely used in practice due to head-of-line blocking issues. HTTP/2 improved on this significantly, but we're focusing on HTTP/1.1's fundamental model.)

## Anatomy of the Cycle

Let's break down what actually happens during a request-response cycle:

### Step 1: Client Prepares Request

The client (browser, app, script) decides it needs something:
- A web page to display
- Data from an API
- To submit a form
- To upload a file

It constructs an HTTP request message:
```
GET /api/weather?city=London HTTP/1.1
Host: api.example.com
User-Agent: MyApp/1.0
Accept: application/json

```

### Step 2: Client Sends Request

The client writes this text through the TCP socket to the server. Remember, the TCP connection was already established (or is established now if this is the first request).

```rust
// Conceptually, what the client does:
stream.write_all(b"GET /api/weather?city=London HTTP/1.1\r\n")?;
stream.write_all(b"Host: api.example.com\r\n")?;
stream.write_all(b"\r\n")?;
stream.flush()?;
```

The bytes flow across the network, TCP ensuring they arrive in order and without corruption.

### Step 3: Server Receives Request

The server's accept loop has already accepted this connection and spawned a handler. That handler reads bytes from the socket:

```rust
// Conceptually, what the server does:
let mut buffer = [0; 1024];
stream.read(&mut buffer)?;
```

The server receives: `GET /api/weather?city=London HTTP/1.1\r\nHost: api.example.com\r\n\r\n`

### Step 4: Server Parses Request

The server must interpret these bytes as an HTTP request:
- Parse the request line: method is GET, path is `/api/weather?city=London`, version is HTTP/1.1
- Parse headers: Host is `api.example.com`
- Extract query parameters: city is "London"

We'll implement this parsing logic in Chapter 8.

### Step 5: Server Processes Request

Now the server does whatever work is necessary:
- Routes the request to the appropriate handler
- Queries a weather database for London
- Retrieves: `{"temp": 15, "condition": "cloudy"}`
- Prepares to send this data back

This is where your application logic lives. The server might:
- Read from a database
- Call other APIs
- Run calculations
- Generate HTML
- Process uploaded files
- Anything your application needs to do

### Step 6: Server Generates Response

The server constructs an HTTP response message:
```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 35

{"temp": 15, "condition": "cloudy"}
```

### Step 7: Server Sends Response

The server writes this response through the TCP socket back to the client:

```rust
stream.write_all(b"HTTP/1.1 200 OK\r\n")?;
stream.write_all(b"Content-Type: application/json\r\n")?;
stream.write_all(b"Content-Length: 35\r\n")?;
stream.write_all(b"\r\n")?;
stream.write_all(b"{\"temp\": 15, \"condition\": \"cloudy\"}")?;
stream.flush()?;
```

### Step 8: Client Receives Response

The client reads bytes from its socket:
```rust
let mut buffer = [0; 1024];
stream.read(&mut buffer)?;
```

It receives the complete response.

### Step 9: Client Processes Response

The client parses the response:
- Status code: 200 (success!)
- Content-Type: JSON
- Body: Weather data

Then does whatever it needs with this data:
- A browser might update the page
- An app might store the data
- A script might process it further

### Cycle Complete

The request-response cycle is now complete. If the client needs more data, it starts a new cycle.

## Timing and Waiting

A critical aspect of this cycle is that **time passes** during each step:

```
Time →
0ms:    Client sends request
        [network latency - request traveling]
50ms:   Server receives request
        [parsing - usually very fast]
51ms:   Server begins processing
        [could be fast or slow depending on work]
200ms:  Server finishes processing
        [response generation - usually fast]
201ms:  Server sends response
        [network latency - response traveling]
251ms:  Client receives response
        [total cycle time: 251ms]
```

During this time, the client is **blocked** waiting for the response (unless it's doing async work). The server is **occupied** handling this request (unless it's properly concurrent, which we'll cover in Part III).

If the server is slow—maybe it's doing a complex database query or calling a slow external API—the client just waits. There's no way for the server to say "hold on, this is taking longer than expected." It either completes and sends a response, or the request times out.

## Multiple Cycles

Most real-world interactions involve many request-response cycles:

Loading a typical web page might involve:

```
Cycle 1:  GET /index.html → HTML document
Cycle 2:  GET /style.css → Stylesheet
Cycle 3:  GET /script.js → JavaScript
Cycle 4:  GET /logo.png → Image
Cycle 5:  GET /api/user → User data (from JavaScript)
Cycle 6:  GET /api/notifications → Notifications (from JavaScript)
```

Each is an independent request-response cycle. The browser might run some of these in parallel by opening multiple TCP connections, but each individual cycle follows the same pattern.

## Connection Reuse

In HTTP/1.0, each request-response cycle typically used a new TCP connection:

```
[Open connection] → Request → Response → [Close connection]
[Open connection] → Request → Response → [Close connection]
```

This was wasteful because establishing TCP connections is expensive (three-way handshake every time).

HTTP/1.1 introduced **persistent connections** (keep-alive) by default:

```
[Open connection] → Request 1 → Response 1
                 → Request 2 → Response 2
                 → Request 3 → Response 3
                 → [Eventually close connection]
```

The connection stays open, and multiple request-response cycles can use it. But each cycle still completes before the next begins on that connection—it's still sequential.

## What Can Go Wrong?

Several things can disrupt the request-response cycle:

**Network failure**: The request or response gets lost. Eventually the client times out.

**Server crash**: The server dies mid-processing. The client never gets a response.

**Client disconnect**: The client closes the connection before receiving the full response. The server might keep processing or might detect the closed connection.

**Timeout**: Either side decides the other is taking too long and gives up.

**Malformed messages**: The server can't parse the request, or the client can't parse the response.

All these scenarios break the cycle. HTTP itself doesn't have built-in recovery—it's up to the application layer (retry logic, error handling, etc.) to deal with failures.

## The Cycle Is Always the Same

Whether you're:
- Loading a simple text file
- Streaming a 4K video
- Submitting a complex form
- Uploading gigabytes of data
- Making a real-time API call

The fundamental cycle never changes: request, process, respond. The complexity varies, the data varies, the processing time varies—but the pattern is constant.

This predictability is one of HTTP's great strengths. Once you understand this cycle, you understand the skeleton of every HTTP interaction.

## Looking Ahead

Now we understand *how* HTTP communication flows. But there's one more crucial concept we need to explore: **statelessness**.

In the next section, we'll discover that HTTP doesn't remember anything between request-response cycles—and why this seemingly limiting design choice is actually a powerful feature.
