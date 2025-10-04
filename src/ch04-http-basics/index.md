# Chapter 4: Understanding HTTP

In the previous chapters, we built a solid foundation in networking fundamentals. We explored how the internet works, learned about TCP sockets, and even built our own TCP server capable of accepting connections and exchanging raw bytes with clients. However, exchanging raw bytes is only the beginning—we need a common language for communication.

This is where HTTP enters the picture.

**HTTP (Hypertext Transfer Protocol)** is the application-layer protocol that powers the World Wide Web. Every time you visit a website, stream a video, submit a form, or interact with a web API, HTTP is working behind the scenes. It's the protocol that turns raw TCP connections into meaningful conversations between clients and servers.

But HTTP is more than just a web browser protocol. It's become the universal language of distributed systems, powering REST APIs, microservices, mobile applications, and IoT devices. Understanding HTTP deeply isn't just about building websites—it's about understanding how modern networked applications communicate.

In this chapter, we'll explore the fundamental concepts that make HTTP work:

- **What HTTP actually is** and why it was designed the way it was
- The **client-server model** that defines the relationship between browsers and web servers
- The **request-response cycle** that structures every HTTP interaction
- Why HTTP is a **stateless protocol** and what that means for our applications

By the end of this chapter, you'll understand not just *how* to use HTTP, but *why* it works the way it does. This foundation will be crucial as we move forward to parsing HTTP messages, building our own HTTP server, and eventually implementing the Flux HTTP framework.

Let's begin by answering a deceptively simple question: what exactly is HTTP?
