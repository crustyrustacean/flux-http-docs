# What is HTTP?

HTTP stands for **Hypertext Transfer Protocol**. At its core, it's an application-layer protocol that defines how messages are formatted and transmitted between clients and servers, and what actions should be taken in response to various commands.

Let's unpack that definition.

## A Protocol for Applications

Remember the OSI model and TCP/IP stack we covered in Chapter 1? HTTP sits at the top of that stack—the application layer. While TCP handles reliable, ordered delivery of bytes between two machines, HTTP defines *what those bytes mean* and *how they should be interpreted*.

Think of it this way: TCP is like a telephone line that guarantees your voice reaches the other person clearly. HTTP is the language you speak over that phone line—the grammar, vocabulary, and conversation rules that let you actually communicate ideas.

## The "Hypertext" Part

When HTTP was created by Tim Berners-Lee at CERN in 1989, the "hypertext" in its name referred to text documents that contained links to other documents. The original vision was simple: scientists needed a way to share and link research papers across different computers.

```
Client: "Please give me the document at /research/particle-physics.html"
Server: "Here it is, and by the way, it contains links to other documents"
```

While HTTP was born to serve hypertext documents, it quickly evolved beyond its original purpose. Today, HTTP transfers all kinds of data: JSON from APIs, images, videos, streaming data, WebSocket upgrades, and much more. The "hypertext" name remains for historical reasons, but HTTP has become the universal protocol for client-server communication on the internet.

## How HTTP Uses TCP

HTTP doesn't work alone—it builds on top of TCP (and occasionally UDP in newer versions, but we'll focus on TCP for now). Here's how they work together:

1. **TCP provides the connection**: Before any HTTP communication happens, a TCP connection is established using the three-way handshake we learned about in Chapter 2.

2. **HTTP uses that connection**: Once the TCP connection exists, the client sends HTTP-formatted messages as bytes through the socket, and the server responds with HTTP-formatted bytes.

3. **TCP ensures delivery**: All the reliability, ordering, and error-checking happens at the TCP layer. HTTP doesn't worry about lost packets or retransmission—TCP handles that.

This separation of concerns is elegant. TCP focuses on *getting bytes from A to B reliably*, while HTTP focuses on *what those bytes represent and what they mean*.

## Text-Based and Human-Readable

One of HTTP's defining characteristics is that it's a text-based protocol. Unlike binary protocols where messages are compact but opaque, HTTP messages are designed to be readable by humans. Here's an actual HTTP request:

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html

```

You can read that! It's asking for the document at `/index.html` from the server at `example.com`. This human-readability was a deliberate design choice that made HTTP easier to debug, easier to implement, and easier to teach.

Of course, there's a trade-off. Text takes more bytes than binary, and parsing text is slower than parsing binary. But in 1989, when network bandwidth was precious but developer time was preciou*er*, this was the right choice. Even today, with HTTP/2 and HTTP/3 introducing binary encodings for performance, the fundamental concepts remain text-based and readable.

## Versions of HTTP

HTTP has evolved over the decades:

- **HTTP/0.9** (1991): The original version—incredibly simple, only supported GET requests
- **HTTP/1.0** (1996): Added headers, status codes, and more methods
- **HTTP/1.1** (1997): The workhorse version, added persistent connections, chunked transfer, and much more. Still widely used today.
- **HTTP/2** (2015): Binary protocol, multiplexing, server push
- **HTTP/3** (2022): Runs over QUIC (UDP-based) instead of TCP

In this book, we'll focus primarily on **HTTP/1.1**. It's the version you need to understand deeply because:
- It's still the most widely deployed version
- Later versions are optimizations of the same fundamental concepts
- Building an HTTP/1.1 server teaches you everything you need to know about the protocol's core principles

Understanding HTTP/1.1 means understanding HTTP. The newer versions improve performance but don't change the fundamental request-response model.

## What HTTP Defines

Specifically, HTTP defines:

1. **Message format**: Exactly how requests and responses should be structured
2. **Methods**: The actions clients can request (GET, POST, PUT, DELETE, etc.)
3. **Status codes**: How servers indicate success, failure, or other outcomes (200, 404, 500, etc.)
4. **Headers**: Metadata about the request or response (content type, length, caching, etc.)
5. **Semantics**: What each method and status code *means* and how they should be used

These aren't just suggestions—they're specifications that implementations must follow for interoperability. A browser and a server can communicate because they both implement the same HTTP specification.

## Why HTTP Succeeded

HTTP's success isn't accidental. Several design decisions made it the protocol of choice for the internet:

- **Simplicity**: Easy to understand and implement
- **Extensibility**: Headers allow new features without breaking old clients
- **Statelessness**: Each request is independent (we'll explore this more in a later section)
- **Layered architecture**: Works over TCP without caring about the underlying network
- **Text-based**: Easy to debug and learn

These principles are worth internalizing. As you build systems, whether they use HTTP or not, these design patterns—simplicity, extensibility, independence—are fundamental to creating protocols that last.

## From Theory to Practice

Now that we understand *what* HTTP is, we need to understand *how* it works in practice. In the next sections, we'll explore:

- The client-server model that structures HTTP communication
- The request-response cycle that governs every HTTP interaction
- The stateless nature of HTTP and its implications

By understanding these concepts, you'll be ready to dive into the actual format of HTTP messages and start building your own HTTP server.
