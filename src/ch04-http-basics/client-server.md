# Client-Server Model

HTTP is built on a **client-server architecture**. This is a fundamental pattern in distributed computing, and understanding it clearly is essential to understanding how HTTP works.

## Clients and Servers: Two Distinct Roles

In the client-server model, there are two distinct roles:

**The Server:**
- Waits passively for incoming requests
- Listens on a specific IP address and port (like `192.168.1.10:80`)
- Provides resources or services
- Responds to requests
- Typically runs continuously, always ready to handle requests

**The Client:**
- Initiates communication
- Sends requests to the server
- Consumes resources or services
- Waits for responses
- Typically runs on-demand when a user needs something

This is an **asymmetric relationship**. The server and client have different responsibilities, different life cycles, and different behaviors. They're not peers—they have clearly defined roles.

## A Restaurant Analogy

Think of a restaurant:

- **The server** (the restaurant itself) is always there, waiting at a specific location. It has a kitchen, staff, and resources ready to prepare food.
- **The client** (you, the customer) decides when you're hungry and walks into the restaurant. You make requests ("I'd like the pasta"), and the restaurant fulfills them.

The restaurant doesn't come to you—you go to the restaurant. The restaurant doesn't decide when you eat—you decide when to visit. This asymmetry is the essence of client-server architecture.

## In HTTP Terms

When you type `https://example.com` into your browser:

1. **Your browser is the client**. It initiates the connection and sends an HTTP request.
2. **The web server at example.com is the server**. It was already running, waiting for requests on port 443 (HTTPS).
3. The server receives your request, processes it, and sends back a response.
4. Your browser receives the response and renders the page.

The server didn't wake up your browser and say "Hey, want to see this web page?" Your browser initiated everything. This is always true in HTTP: **clients initiate, servers respond**.

## One Server, Many Clients

A critical aspect of the client-server model is that **one server typically serves many clients**.

Consider a web server for a popular website:
- The server runs on one machine (or cluster of machines)
- Thousands or millions of clients might connect to it
- Each client gets its own TCP connection
- The server handles all these connections concurrently (we'll explore how in the Concurrency chapters)

```
         Client 1 (Browser in Tokyo)
                    ↓
         Client 2 (Mobile app in London) → [Server at example.com:80]
                    ↓
         Client 3 (API call from NYC)
```

From the server's perspective, all these clients are independent. It doesn't matter if one client is a web browser, another is a mobile app, and another is an automated script—they all speak HTTP, so the server can handle them all the same way.

## Who Knows About Whom?

Here's an important asymmetry: **servers don't need to know clients exist until they connect**.

When you start a web server:
```rust
let listener = TcpListener::bind("0.0.0.0:8080")?;
println!("Server running on port 8080");
```

The server doesn't have a list of clients. It doesn't know who will connect or when. It just waits. Clients might connect immediately, or hours later, or never. The server's job is simply to be ready.

Conversely, **clients must know the server's address**. You can't just randomly browse the web hoping to stumble upon websites—you need to know (or discover through DNS) where the server is located.

## Request Origination

In HTTP, **only clients send requests**. This is a hard rule.

- A client sends: "GET /api/users"
- The server responds: "200 OK, here's the data"

The server cannot spontaneously send data to a client. Even if the server has urgent information for a client, it must wait for the client to ask. This might seem limiting (and it is—hence technologies like WebSockets and Server-Sent Events for bidirectional communication), but it greatly simplifies the protocol.

This means:
- Servers are reactive, not proactive
- Clients are in control of when communication happens
- Every HTTP message is either a request (from client) or a response (from server)

## Not Peer-to-Peer

It's worth contrasting HTTP's client-server model with **peer-to-peer (P2P)** architectures like BitTorrent or blockchain networks.

In P2P systems:
- All nodes are equal peers
- Any node can initiate communication with any other node
- Nodes are both clients and servers simultaneously
- There's no central authority or single point of failure

HTTP explicitly rejects this model. There's always a clear distinction between client and server. This makes HTTP simpler to reason about and implement, but it also creates centralization—if the server goes down, all clients are affected.

## The Connection Dance

Here's what happens when a client wants to communicate with a server:

```
1. Server starts and binds to 192.168.1.10:8080
   └─> Listening, waiting...

2. Client decides it needs something from that server
   └─> Initiates TCP connection to 192.168.1.10:8080

3. TCP three-way handshake (from Chapter 2)
   └─> Connection established

4. Client sends HTTP request through the connection
   └─> "GET /data HTTP/1.1"

5. Server receives request, processes it
   └─> Reads from socket, parses HTTP, generates response

6. Server sends HTTP response back
   └─> "HTTP/1.1 200 OK\r\n..."

7. Client receives response
   └─> Reads from socket, parses HTTP, uses data

8. Connection closes (or kept alive for reuse)
```

Notice how the server never decides when this happens—the client drives the entire interaction.

## Multiple Servers, One Client

While one server typically serves many clients, it's also common for one client to talk to many servers:

Your web browser might load a single web page that requires:
- HTML from `example.com`
- JavaScript from `cdn.example.com`
- Images from `images.example.com`
- API data from `api.example.com`
- Ads from `ads.thirdparty.com`

Your browser is the client in all these interactions, talking to five different servers. Each connection follows the same client-server model—your browser initiates, the servers respond.

## Implications for Building Servers

Understanding the client-server model has practical implications when building HTTP servers:

1. **Design for multiple concurrent clients**: Your server must handle many simultaneous connections. One slow client shouldn't block others.

2. **Don't assume client availability**: Clients come and go. Never assume a client will stick around or reconnect.

3. **Be passive and reactive**: Don't try to "push" to clients unless you're using specific technologies designed for that (WebSockets, etc.).

4. **Trust nothing from clients**: Clients can send malicious or malformed requests. Validate everything.

5. **Make your server stateless** (usually): We'll explore this more in the next section, but the server shouldn't need to remember individual clients between requests.

## Why This Model Works

The client-server model has powered the web for over three decades because:

- **Clear separation of concerns**: Clients handle presentation and user interaction; servers handle data and business logic.
- **Scalability**: Servers can be scaled independently by adding more machines behind load balancers.
- **Security boundaries**: Servers can enforce access control and protect sensitive data.
- **Simplicity**: The asymmetric relationship is easier to reason about than peer-to-peer systems.

## Looking Ahead

Now that we understand the roles of clients and servers, we can explore exactly how they communicate. In the next section, we'll dive into the **request-response cycle**—the fundamental pattern that governs every HTTP interaction.

Remember: in HTTP, clients ask, servers answer. Always.
