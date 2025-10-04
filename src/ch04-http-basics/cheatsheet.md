# Chapter 4 Cheatsheet

## What is HTTP?

**Definition**
- HTTP = Hypertext Transfer Protocol
- Application-layer protocol that defines how messages are formatted and transmitted
- Built on top of TCP (which handles reliable byte delivery)

**Key Characteristics**
- Text-based and human-readable
- Request-response pattern
- Stateless by design
- Extensible through headers

**HTTP Versions**
- **HTTP/1.1**: The workhorse version we'll focus on (1997-present)
- **HTTP/2**: Binary, multiplexed (2015)
- **HTTP/3**: Over QUIC/UDP instead of TCP (2022)

**What HTTP Defines**
- Message format (how requests/responses are structured)
- Methods (GET, POST, PUT, DELETE, etc.)
- Status codes (200, 404, 500, etc.)
- Headers (metadata about messages)
- Semantics (what everything means)

---

## Client-Server Model

**Two Distinct Roles**

| Client | Server |
|--------|--------|
| Initiates communication | Waits passively for requests |
| Sends requests | Sends responses |
| Consumes resources | Provides resources |
| Runs on-demand | Runs continuously |

**Key Rules**
- **Clients always initiate**—servers never spontaneously send data
- **One server, many clients**—servers handle multiple concurrent connections
- **Clients must know server address**—servers don't need to know about clients until they connect
- **Asymmetric relationship**—not peer-to-peer

**In Practice**
```
Browser (client) → Initiates connection
Web Server (server) → Waits on port 80/443
                    → Responds to requests
```

---

## Request-Response Cycle

**The Fundamental Pattern**
```
1. Client sends request
2. Server processes request
3. Server sends response
4. Client processes response
[Cycle complete]
```

**Critical Properties**
- **One request, one response**—always exactly one response per request
- **Synchronous**—client waits for response (in HTTP/1.1)
- **Sequential on a connection**—one cycle completes before next begins
- **Independent cycles**—each is self-contained

**Lifecycle**
```
Client: Prepare → Send → Wait → Receive → Process
Server: Listen → Receive → Parse → Process → Generate → Send
```

**Connection Reuse (HTTP/1.1)**
- HTTP/1.0: New TCP connection per request (wasteful)
- HTTP/1.1: Persistent connections (keep-alive) by default
- Multiple request-response cycles can share one TCP connection

**What Can Go Wrong**
- Network failure → request/response lost
- Server crash → no response
- Client disconnect → incomplete cycle
- Timeout → either side gives up
- Malformed messages → parsing fails

---

## Stateless Protocol

**Core Concept**
- **Stateless**: Server doesn't remember anything between requests
- Each request is processed independently
- Server has no memory, no context, no history

**Visualization**
```
Request 1 → Process → Response → [FORGET EVERYTHING]
Request 2 → Process → Response → [FORGET EVERYTHING]
Request 3 → Process → Response → [FORGET EVERYTHING]
```

**Why Stateless?**

| Advantage | Benefit |
|-----------|---------|
| **Simplicity** | No session tracking, timeout management, or cleanup |
| **Scalability** | Any server can handle any request—easy load balancing |
| **Reliability** | Server crashes don't lose state (nothing to lose) |
| **Caching** | Responses can be cached aggressively |
| **Independence** | Each request is self-contained and debuggable |

**The Golden Rule**
> Every request must contain ALL information the server needs to process it

**How Applications Maintain State**

The application layer creates the illusion of state while HTTP remains stateless:

**Client-side state:**
- Cookies
- Local Storage
- URL parameters
- Hidden form fields

**Server-side state (external):**
- Databases (session stores)
- Caches (Redis, Memcached)
- File systems

**Common Pattern: Sessions with Cookies**
```
1. User logs in
2. Server creates session in database (session_id → user_data)
3. Server sends session_id to client via Set-Cookie
4. Client includes session_id in Cookie header on every request
5. Server looks up session_id to retrieve state

→ HTTP stays stateless; state lives in database
→ Each request includes session_id (self-contained)
```

---

## Key Takeaways

✓ **HTTP is a text-based protocol** that builds on TCP
✓ **Clients initiate, servers respond**—never the other way around
✓ **Every request gets exactly one response**—always
✓ **Each request-response cycle is independent**—no memory between them
✓ **Statelessness enables simplicity and scalability**—but requires external state management
✓ **Understanding these fundamentals is essential**—everything in HTTP builds on these concepts

---

## Quick Reference

**What makes a protocol HTTP-compliant?**
1. Follows the message format (we'll learn in Chapter 5)
2. Uses standard methods and status codes
3. Maintains statelessness
4. Follows request-response pattern

**Remember:**
- HTTP = language, TCP = delivery mechanism
- Client-server = who does what
- Request-response = how communication flows
- Stateless = no memory between requests

---

**Next Up**: Chapter 5 - HTTP Message Format

Now that we understand the *concepts* behind HTTP, we'll dive into the actual *bytes* that make up HTTP messages. We'll see exactly how requests and responses are structured, line by line, byte by byte.
