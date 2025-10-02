# Summary

[Introduction](./introduction.md)

# Part I: Networking Fundamentals

- [How the Internet Works](./ch01-how-internet-works/index.md)
  - [The OSI Model](./ch01-how-internet-works/osi-model.md)
  - [TCP/IP Stack](./ch01-how-internet-works/tcp-ip-stack.md)
  - [IP Addresses and Ports](./ch01-how-internet-works/addresses-ports.md)
  - [DNS and Hostnames](./ch01-how-internet-works/dns.md)
  - [Cheatsheet](./ch01-how-internet-works/cheat-sheet.md)

- [Sockets and Connections](./ch02-sockets/index.md)
  - [What is a Socket?](./ch02-sockets/what-is-socket.md)
  - [TCP vs UDP](./ch02-sockets/tcp-vs-udp.md)
  - [The Three-Way Handshake](./ch02-sockets/handshake.md)
  - [Socket States](./ch02-sockets/states.md)
  - [Cheatsheet](./ch02-sockets/cheatsheet.md)

- [Building a TCP Server](./ch03-tcp-server/index.md)
  - [Rust's std::net](./ch03-tcp-server/std-net.md)
  - [Creating a Listener](./ch03-tcp-server/listener.md)
  - [Accepting Connections](./ch03-tcp-server/accepting.md)
  - [Reading and Writing Bytes](./ch03-tcp-server/read-write.md)
  - [Error Handling](./ch03-tcp-server/errors.md)
  - [Final Code](./ch03-tcp-server/final-code.md)

# Part II: The HTTP Protocol

- [Understanding HTTP](./ch04-http-basics/index.md)
  - [What is HTTP?](./ch04-http-basics/what-is-http.md)
  - [Client-Server Model](./ch04-http-basics/client-server.md)
  - [Request-Response Cycle](./ch04-http-basics/request-response.md)
  - [Stateless Protocol](./ch04-http-basics/stateless.md)

- [HTTP Message Format](./ch05-http-format/index.md)
  - [Request Line](./ch05-http-format/request-line.md)
  - [Status Line](./ch05-http-format/status-line.md)
  - [Headers](./ch05-http-format/headers.md)
  - [Message Body](./ch05-http-format/body.md)
  - [Line Endings and Encoding](./ch05-http-format/encoding.md)

- [HTTP Methods](./ch06-http-methods/index.md)
  - [GET](./ch06-http-methods/get.md)
  - [POST](./ch06-http-methods/post.md)
  - [PUT and PATCH](./ch06-http-methods/put-patch.md)
  - [DELETE](./ch06-http-methods/delete.md)
  - [HEAD and OPTIONS](./ch06-http-methods/head-options.md)
  - [Safe and Idempotent Methods](./ch06-http-methods/safe-idempotent.md)

- [HTTP Status Codes](./ch07-status-codes/index.md)
  - [1xx Informational](./ch07-status-codes/1xx.md)
  - [2xx Success](./ch07-status-codes/2xx.md)
  - [3xx Redirection](./ch07-status-codes/3xx.md)
  - [4xx Client Errors](./ch07-status-codes/4xx.md)
  - [5xx Server Errors](./ch07-status-codes/5xx.md)

- [Parsing HTTP Manually](./ch08-parsing/index.md)
  - [Parsing the Request Line](./ch08-parsing/request-line.md)
  - [Parsing Headers](./ch08-parsing/headers.md)
  - [Handling the Body](./ch08-parsing/body.md)
  - [Edge Cases and Pitfalls](./ch08-parsing/edge-cases.md)
  - [Why Use a Parser Library](./ch08-parsing/use-library.md)

- [Building an HTTP Server](./ch09-http-server/index.md)
  - [Request Struct](./ch09-http-server/request-struct.md)
  - [Response Struct](./ch09-http-server/response-struct.md)
  - [Parsing Implementation](./ch09-http-server/parsing.md)
  - [Response Generation](./ch09-http-server/response.md)
  - [A Working Server](./ch09-http-server/complete.md)

# Part III: Concurrency

- [The Single-Threaded Problem](./ch10-single-threaded/index.md)
  - [Blocking I/O](./ch10-single-threaded/blocking.md)
  - [Sequential Processing](./ch10-single-threaded/sequential.md)
  - [Performance Bottlenecks](./ch10-single-threaded/bottlenecks.md)

- [Threads](./ch11-threads/index.md)
  - [What are Threads?](./ch11-threads/what-are-threads.md)
  - [Spawning Threads](./ch11-threads/spawning.md)
  - [Thread Safety](./ch11-threads/safety.md)
  - [One Thread Per Connection](./ch11-threads/per-connection.md)

- [Thread Pools](./ch12-thread-pools/index.md)
  - [Why Thread Pools?](./ch12-thread-pools/why.md)
  - [Worker Pattern](./ch12-thread-pools/workers.md)
  - [Channel Communication](./ch12-thread-pools/channels.md)
  - [Implementation](./ch12-thread-pools/implementation.md)

- [Thread Pool Limitations](./ch13-thread-limits/index.md)
  - [Resource Exhaustion](./ch13-thread-limits/exhaustion.md)
  - [Context Switching](./ch13-thread-limits/context-switching.md)
  - [Waiting vs Working](./ch13-thread-limits/waiting.md)
  - [The C10K Problem](./ch13-thread-limits/c10k.md)

# Part IV: Async Programming

- [Introduction to Async](./ch14-async-intro/index.md)
  - [The Problem Async Solves](./ch14-async-intro/problem.md)
  - [Cooperative Multitasking](./ch14-async-intro/cooperative.md)
  - [Event Loops](./ch14-async-intro/event-loops.md)
  - [Futures](./ch14-async-intro/futures.md)

- [Async/Await in Rust](./ch15-async-rust/index.md)
  - [The async Keyword](./ch15-async-rust/async-keyword.md)
  - [The await Keyword](./ch15-async-rust/await-keyword.md)
  - [Tokio Runtime](./ch15-async-rust/tokio.md)
  - [Async Traits and Lifetimes](./ch15-async-rust/traits-lifetimes.md)

- [Migrating to Async](./ch16-async-migration/index.md)
  - [Converting the Server](./ch16-async-migration/converting.md)
  - [Async Accept Loop](./ch16-async-migration/accept-loop.md)
  - [Spawning Tasks](./ch16-async-migration/spawning.md)
  - [Performance Comparison](./ch16-async-migration/performance.md)

- [Async + Threads](./ch17-async-threads/index.md)
  - [When to Use Both](./ch17-async-threads/when.md)
  - [CPU-Bound vs I/O-Bound](./ch17-async-threads/bound.md)
  - [spawn_blocking](./ch17-async-threads/spawn-blocking.md)
  - [Best Practices](./ch17-async-threads/practices.md)

# Part V: Production HTTP

- [Using Hyper](./ch18-hyper/index.md)
  - [Why Not Parse Manually?](./ch18-hyper/why-hyper.md)
  - [HTTP/1.1 and HTTP/2](./ch18-hyper/http-versions.md)
  - [Setting Up Hyper](./ch18-hyper/setup.md)
  - [Service Trait](./ch18-hyper/service-trait.md)
  - [Request and Response Types](./ch18-hyper/types.md)

- [Web Framework Patterns](./ch19-frameworks/index.md)
  - [What Frameworks Provide](./ch19-frameworks/what-frameworks.md)
  - [Routing](./ch19-frameworks/routing.md)
  - [Middleware](./ch19-frameworks/middleware.md)
  - [Extractors](./ch19-frameworks/extractors.md)
  - [Axum vs Actix vs Warp](./ch19-frameworks/comparison.md)

- [Static Files](./ch20-static/index.md)
  - [Serving Files](./ch20-static/serving.md)
  - [MIME Types](./ch20-static/mime-types.md)
  - [Caching Headers](./ch20-static/caching.md)
  - [Security Considerations](./ch20-static/security.md)

- [HTML Templating](./ch21-templates/index.md)
  - [Why Templates?](./ch21-templates/why.md)
  - [Template Engines](./ch21-templates/engines.md)
  - [Tera Basics](./ch21-templates/tera-basics.md)
  - [Context and Data](./ch21-templates/context.md)

# Part VI: Building Flux HTTP

- [Designing a Framework](./ch22-design/index.md)
  - [Design Goals](./ch22-design/goals.md)
  - [Express as Inspiration](./ch22-design/express.md)
  - [API Design](./ch22-design/api.md)
  - [Trade-offs](./ch22-design/tradeoffs.md)

- [Core Architecture](./ch23-architecture/index.md)
  - [Request/Response Types](./ch23-architecture/types.md)
  - [Handler Trait](./ch23-architecture/handler.md)
  - [Router Implementation](./ch23-architecture/router.md)
  - [App Structure](./ch23-architecture/app.md)

- [Routing](./ch24-routing/index.md)
  - [Exact Path Matching](./ch24-routing/exact.md)
  - [Path Parameters](./ch24-routing/params.md)
  - [Query Strings](./ch24-routing/query.md)
  - [Route Priority](./ch24-routing/priority.md)

- [Middleware](./ch25-middleware/index.md)
  - [Middleware Pattern](./ch25-middleware/pattern.md)
  - [Request Pipeline](./ch25-middleware/pipeline.md)
  - [Common Middleware](./ch25-middleware/common.md)
  - [Error Handling](./ch25-middleware/errors.md)

- [Testing](./ch26-testing/index.md)
  - [Unit Tests](./ch26-testing/unit.md)
  - [Integration Tests](./ch26-testing/integration.md)
  - [Testing Strategy](./ch26-testing/strategy.md)

- [Production Considerations](./ch27-production/index.md)
  - [TLS/HTTPS](./ch27-production/tls.md)
  - [Reverse Proxies](./ch27-production/proxy.md)
  - [Logging](./ch27-production/logging.md)
  - [Error Handling](./ch27-production/errors.md)
  - [Graceful Shutdown](./ch27-production/shutdown.md)
  - [Why Use Existing Frameworks](./ch27-production/use-frameworks.md)

[Conclusion](./conclusion.md)
[References](./references.md)