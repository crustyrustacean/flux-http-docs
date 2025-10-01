# Introduction

## What This Book Is About

This book documents my journey building a web framework in Rust, starting from absolute first principles. We begin with "what is a network packet?" and end with "how do I deploy this to production?" Everything in between—TCP/IP, sockets, HTTP, parsing, threads, async, frameworks—we build ourselves or understand deeply before using libraries.

Most programming books assume you already understand the layers below what they're teaching. A web framework book assumes you know HTTP. An HTTP book assumes you know TCP. A TCP book assumes you know networking fundamentals. This book assumes none of that. We start at the bottom and work up, layer by layer, until we've built a complete web framework called Flux HTTP.

## Who Should Read This

You should read this if you:
- Know basic Rust (ownership, borrowing, common types)
- Have built something with a web framework before
- Want to understand how everything actually works
- Are willing to invest significant time in deep learning
- Learn best by building things yourself

You don't need to know networking, systems programming, or concurrency. We'll build that knowledge from scratch. But you need patience—this is not a quick tutorial. We're going deep.

## What Makes This Different

Most resources either stay too high-level (just use the framework) or too low-level (here's the RFC specification). This book occupies the middle ground: we implement enough to understand deeply, then use production tools when appropriate.

We don't skip steps. Before writing async code, we write blocking code and feel its pain. Before using Hyper, we parse HTTP manually and discover why it's hard. Before building a framework, we understand what problems frameworks solve.

The progression is deliberate: each chapter reveals a limitation that the next chapter solves. You'll understand *why* async exists because you struggled with threads. You'll appreciate Hyper because you wrote a buggy parser. You'll know what frameworks provide because you felt their absence.

## The Journey

This book has six parts, each building on the previous:

### Part I: Networking Fundamentals (Chapters 1-3)

We start with "how does the internet work?" The OSI model, TCP/IP stack, what IP addresses and ports are, how DNS functions. Then we dive into sockets: what they are, TCP vs UDP, the three-way handshake, socket states.

Finally, we build a TCP server using Rust's standard library. We learn to create listeners, accept connections, read and write bytes. No HTTP yet—just raw TCP.

**Why start here?** You can't understand HTTP without understanding what's underneath. When we encounter "connection refused" or "address already in use," you'll know exactly what's happening.

### Part II: The HTTP Protocol (Chapters 4-9)

We study HTTP thoroughly before implementing it. What is HTTP? How does the request-response cycle work? Why is it stateless? We dissect the message format: request lines, status lines, headers, bodies, line endings.

We explore HTTP methods (GET, POST, PUT, DELETE) and when to use each. We categorize status codes (2xx success, 4xx client error, 5xx server error). We examine headers and their purposes.

Then we try to parse HTTP manually. We implement request and response structs. We write parsing code. We discover edge cases, encoding issues, security problems. We learn why parsing HTTP is deceptively hard.

By the end, we have a working HTTP server built entirely from scratch—and we understand why we shouldn't have done it this way.

**Why so thorough?** HTTP is the foundation of the web. Shallow understanding leads to bugs. Deep understanding lets you debug production issues and make informed architectural decisions.

### Part III: Concurrency (Chapters 10-13)

Our single-threaded server can only handle one request at a time. We explore why this matters, what blocking I/O means, where performance bottlenecks occur.

We add threads: what they are, how to spawn them, thread safety concerns. We implement "one thread per connection" and see its limitations.

We build a thread pool from scratch: the worker pattern, channel communication, handling jobs. We see how thread pools improve resource usage.

Then we hit thread pool limitations: resource exhaustion, context switching overhead, the difference between waiting and working. We learn about the C10K problem and why threads don't scale to thousands of connections.

**Why threads before async?** You can't appreciate async's value until you've tried to solve concurrency with threads and hit their limits. The frustration is the lesson.

### Part IV: Async Programming (Chapters 14-17)

We introduce async as a solution to thread limitations. What problem does async solve? How does cooperative multitasking differ from preemptive? What are event loops and futures?

We learn Rust's async/await syntax: the `async` keyword, the `await` keyword, the Tokio runtime, async traits and lifetime challenges.

We migrate our server to async and measure the performance difference. We learn when to combine async and threads: CPU-bound vs I/O-bound work, `spawn_blocking`, best practices.

**Why async last?** Async is complex. Understanding it requires understanding what came before. Now you'll know exactly what async does and why it exists.

### Part V: Production HTTP (Chapters 18-21)

We stop reinventing wheels. We adopt Hyper for HTTP and see what production implementations handle: HTTP/1.1, HTTP/2, the Service trait, proper request and response types.

We study web framework patterns: what frameworks provide, routing implementations, middleware architectures, extractors. We compare Axum, Actix-Web, and Warp.

We add static file serving: proper MIME types, caching headers, security considerations. We add HTML templating with Tera: why templates exist, how template engines work, context and data flow.

**Why use libraries now?** Because we understand what they do. We're not cargo-culting—we know exactly what Hyper handles and why we shouldn't write our own HTTP parser.

### Part VI: Building Flux Web (Chapters 22-27)

We design and build our own Express-like framework on top of Tokio and Hyper. Not because the world needs another framework, but because building one cements everything we've learned.

We define design goals, draw inspiration from Express, make API design choices, accept trade-offs. We implement core architecture: request/response types, handler traits, router logic, the app structure.

We add routing features: exact path matching, path parameters (`/users/:id`), query strings, route priority. We implement middleware: the middleware pattern, request pipelines, common middleware, error handling.

We write tests: unit tests for components, integration tests for the whole system, testing strategies for frameworks.

Finally, we consider production: TLS/HTTPS, reverse proxies, logging, error handling, graceful shutdown. And we discuss honestly why you should probably use existing frameworks instead of your own.

**Why build a framework?** Because it's the ultimate test of understanding. If you can build a framework, you truly understand all the layers beneath it.

## Time Commitment

This is not a weekend project. Depending on your background and how deeply you engage:

- **Reading through:** 20-40 hours
- **Typing all code:** 40-60 hours  
- **Experimenting and exploring:** 60-100+ hours

This is a semester-long self-study course, not a tutorial. Budget accordingly.

## Prerequisites

**Required:**
- Basic Rust: ownership, borrowing, pattern matching, common types
- Any web framework experience (Express, Flask, Rails—any language)
- Comfort with the terminal and command-line tools
- Willingness to struggle, get stuck, and debug

**Not required:**
- Networking knowledge
- Systems programming experience
- Understanding of async or threads
- Production server experience

We build all of this from scratch.

## How to Use This Book

**Don't skip chapters.** Each assumes knowledge from previous chapters. The order is carefully designed.

**Type the code yourself.** Don't copy-paste. Typing forces you to read every line and think about what it does.

**When stuck, struggle first.** The confusion is where learning happens. Try to debug before checking solutions.

**Experiment freely.** Break things. Change things. See what happens. The code is yours to modify.

**Take breaks.** This is dense material. Let concepts settle. Come back refreshed.

## What You'll Know by the End

By the final chapter, you'll have:

- Built a TCP server from scratch
- Parsed HTTP manually and understood the specification
- Implemented threading and thread pools
- Written async code with Tokio
- Used Hyper for production HTTP
- Created a complete web framework
- Deployed a working application

More importantly, you'll understand:

- Why async exists and when to use it
- What frameworks actually do
- How HTTP really works
- When threads make sense and when they don't
- Trade-offs in API design
- What production deployment requires

You'll have built intuition. When you use Express or Axum, you'll know what's happening under the hood. When something breaks, you'll know where to look. When making architectural decisions, you'll understand the implications.

## A Word About Ambition

This is an ambitious book. Twenty-seven chapters covering six major topics. If that seems overwhelming, remember: we're building incrementally. Each chapter is digestible. Each concept builds on the last. The journey is long, but the path is clear.

You don't have to finish it all at once. Read what interests you. Skip ahead if you already know a topic (though be aware later chapters assume earlier knowledge). Use it as a reference. Come back to it over time.

But if you do work through it completely, you'll have something few developers possess: genuine understanding of the entire web stack, from TCP packets to production deployment.

## Philosophy: Build to Understand

We're not building production software. We're building teaching software. The goal isn't creating perfect code—it's creating perfect understanding.

That means:
- We make mistakes intentionally to learn from them
- We build "wrong" solutions before "right" ones
- We prioritize clarity over performance
- We show reasoning, not just results

When code seems simple, trust it. When it seems complex, ask "what problem does this solve?" Usually an earlier chapter demonstrated that problem.

## Let's Begin

We start with a simple question: "How does the internet work?" From that question, we'll build everything else. TCP servers, HTTP parsers, thread pools, async runtimes, web frameworks, production deployments.

The journey is long. The concepts are deep. The code is extensive. But at the end, you'll understand web development from the ground up.

Ready? Let's write some code.

---

**Note:** This book uses mdBook for publishing. All code is available in the companion repository. Each chapter includes complete, runnable examples.