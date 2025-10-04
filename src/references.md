# References

This section provides authoritative sources and recommended reading for the topics covered in this book. The primary references are IETF RFCs (Request for Comments), which are the official specifications for internet protocols.

## Primary Specifications

### HTTP Protocol

**RFC 9110 - HTTP Semantics** (June 2022)  
Fielding, R., et al.  
*https://www.rfc-editor.org/rfc/rfc9110.html*  
The core HTTP specification defining the semantics of HTTP messages, methods, status codes, and headers. This is the authoritative source for HTTP/1.1 and later versions.

**RFC 9112 - HTTP/1.1** (June 2022)  
Fielding, R., et al.  
*https://www.rfc-editor.org/rfc/rfc9112.html*  
Defines the HTTP/1.1 message syntax, connection management, and framing. Essential for understanding the wire format of HTTP messages.

**RFC 7230 - HTTP/1.1: Message Syntax and Routing** (June 2014)  
Fielding, R., et al.  
*https://www.rfc-editor.org/rfc/rfc7230.html*  
Obsoleted by RFC 9112, but still useful for historical context and understanding HTTP/1.1 evolution.

**RFC 7231 - HTTP/1.1: Semantics and Content** (June 2014)  
Fielding, R., et al.  
*https://www.rfc-editor.org/rfc/rfc7231.html*  
Obsoleted by RFC 9110, but valuable for understanding HTTP semantics and content negotiation.

**RFC 7232 - HTTP/1.1: Conditional Requests** (June 2014)  
Fielding, R., et al.  
*https://www.rfc-editor.org/rfc/rfc7232.html*  
Covers ETags, conditional requests, and caching validation.

**RFC 7233 - HTTP/1.1: Range Requests** (June 2014)  
Fielding, R., et al.  
*https://www.rfc-editor.org/rfc/rfc7233.html*  
Defines partial content retrieval and byte ranges.

**RFC 7234 - HTTP/1.1: Caching** (June 2014)  
Fielding, R., et al.  
*https://www.rfc-editor.org/rfc/rfc7234.html*  
Comprehensive specification for HTTP caching mechanisms.

**RFC 7235 - HTTP/1.1: Authentication** (June 2014)  
Fielding, R., et al.  
*https://www.rfc-editor.org/rfc/rfc7235.html*  
Defines HTTP authentication framework and schemes.

**RFC 2616 - Hypertext Transfer Protocol -- HTTP/1.1** (June 1999)  
Fielding, R., et al.  
*https://www.rfc-editor.org/rfc/rfc2616.html*  
The original HTTP/1.1 specification. Obsoleted by RFC 7230-7235 series, then by RFC 9110-9114.

### URI and URL Specifications

**RFC 3986 - Uniform Resource Identifier (URI): Generic Syntax** (January 2005)  
Berners-Lee, T., et al.  
*https://www.rfc-editor.org/rfc/rfc3986.html*  
Defines URI syntax, including URLs, percent-encoding, and path structures.

**RFC 3987 - Internationalized Resource Identifiers (IRIs)** (January 2005)  
Duerst, M., et al.  
*https://www.rfc-editor.org/rfc/rfc3987.html*  
Extends URIs to support international characters.

### Transport Layer Protocols

**RFC 793 - Transmission Control Protocol** (September 1981)  
Postel, J.  
*https://www.rfc-editor.org/rfc/rfc793.html*  
The foundational TCP specification defining reliable, ordered byte streams.

**RFC 9293 - Transmission Control Protocol (TCP)** (August 2022)  
Eddy, W., ed.  
*https://www.rfc-editor.org/rfc/rfc9293.html*  
Updated TCP specification consolidating amendments and clarifications.

**RFC 768 - User Datagram Protocol** (August 1980)  
Postel, J.  
*https://www.rfc-editor.org/rfc/rfc768.html*  
Defines UDP, the connectionless transport protocol.

### Internet Protocol

**RFC 791 - Internet Protocol** (September 1981)  
Postel, J.  
*https://www.rfc-editor.org/rfc/rfc791.html*  
The original IPv4 specification.

**RFC 8200 - Internet Protocol, Version 6 (IPv6) Specification** (July 2017)  
Deering, S., et al.  
*https://www.rfc-editor.org/rfc/rfc8200.html*  
The IPv6 protocol specification.

### Domain Name System (DNS)

**RFC 1034 - Domain Names - Concepts and Facilities** (November 1987)  
Mockapetris, P.  
*https://www.rfc-editor.org/rfc/rfc1034.html*  
Introduces DNS concepts and architecture.

**RFC 1035 - Domain Names - Implementation and Specification** (November 1987)  
Mockapetris, P.  
*https://www.rfc-editor.org/rfc/rfc1035.html*  
Defines DNS protocol details and message formats.

### Character Encoding

**RFC 3629 - UTF-8, a transformation format of ISO 10646** (November 2003)  
Yergeau, F.  
*https://www.rfc-editor.org/rfc/rfc3629.html*  
Official UTF-8 encoding specification.

**RFC 2047 - MIME Part Three: Message Header Extensions for Non-ASCII Text** (November 1996)  
Moore, K.  
*https://www.rfc-editor.org/rfc/rfc2047.html*  
Defines encoded-word syntax for non-ASCII text in headers.

### MIME and Content Types

**RFC 2045 - Multipurpose Internet Mail Extensions (MIME) Part One** (November 1996)  
Freed, N., et al.  
*https://www.rfc-editor.org/rfc/rfc2045.html*  
Defines MIME message format and Content-Type header.

**RFC 2046 - MIME Part Two: Media Types** (November 1996)  
Freed, N., et al.  
*https://www.rfc-editor.org/rfc/rfc2046.html*  
Defines standard MIME media types.

**RFC 2049 - MIME Part Five: Conformance Criteria** (November 1996)  
Freed, N., et al.  
*https://www.rfc-editor.org/rfc/rfc2049.html*  
MIME conformance requirements.

### HTTP/2 and HTTP/3

**RFC 9113 - HTTP/2** (June 2022)  
Thomson, M., et al.  
*https://www.rfc-editor.org/rfc/rfc9113.html*  
The binary framing HTTP/2 protocol specification.

**RFC 9114 - HTTP/3** (June 2022)  
Bishop, M., ed.  
*https://www.rfc-editor.org/rfc/rfc9114.html*  
HTTP over QUIC (UDP-based transport).

**RFC 9000 - QUIC: A UDP-Based Multiplexed and Secure Transport** (May 2021)  
Iyengar, J., et al.  
*https://www.rfc-editor.org/rfc/rfc9000.html*  
The QUIC transport protocol underlying HTTP/3.

### Security

**RFC 8446 - The Transport Layer Security (TLS) Protocol Version 1.3** (August 2018)  
Rescorla, E.  
*https://www.rfc-editor.org/rfc/rfc8446.html*  
Current TLS specification for securing HTTP connections (HTTPS).

**RFC 6265 - HTTP State Management Mechanism** (April 2011)  
Barth, A.  
*https://www.rfc-editor.org/rfc/rfc6265.html*  
Defines HTTP cookies and their security considerations.

## Books

**Stevens, W. Richard.** *TCP/IP Illustrated, Volume 1: The Protocols*, 2nd Edition.  
Addison-Wesley Professional, 2011.  
ISBN: 978-0321336316  
The definitive guide to TCP/IP protocols with detailed packet analysis and protocol behavior.

**Stevens, W. Richard.** *Unix Network Programming, Volume 1: The Sockets Networking API*, 3rd Edition.  
Addison-Wesley Professional, 2003.  
ISBN: 978-0131411555  
Essential reference for socket programming and network application development.

**Gourley, David and Totty, Brian.** *HTTP: The Definitive Guide*.  
O'Reilly Media, 2002.  
ISBN: 978-1565925090  
Comprehensive coverage of HTTP/1.1 protocol details and web architecture.

**Tanenbaum, Andrew S. and Wetherall, David J.** *Computer Networks*, 5th Edition.  
Pearson, 2010.  
ISBN: 978-0132126953  
Thorough treatment of networking fundamentals including the OSI model and TCP/IP stack.

**Kurose, James F. and Ross, Keith W.** *Computer Networking: A Top-Down Approach*, 8th Edition.  
Pearson, 2020.  
ISBN: 978-0135928615  
Accessible introduction to computer networking with application-layer focus.

**Fall, Kevin R. and Stevens, W. Richard.** *TCP/IP Illustrated, Volume 2: The Implementation*.  
Addison-Wesley Professional, 1995.  
ISBN: 978-0201633542  
Deep dive into TCP/IP implementation details.

## Rust Programming Resources

**The Rust Programming Language** (Official Book)  
Klabnik, Steve and Nichols, Carol.  
*https://doc.rust-lang.org/book/*  
The official Rust book covering language fundamentals.

**Rust Standard Library Documentation - std::net Module**  
*https://doc.rust-lang.org/std/net/*  
Official documentation for Rust's networking primitives including TcpListener and TcpStream.

**Tokio Documentation**  
*https://tokio.rs/*  
Official documentation for Tokio, Rust's asynchronous runtime.

**Hyper Documentation**  
*https://hyper.rs/*  
Documentation for Hyper, a fast HTTP implementation in Rust.

**Blandy, Jim, Orendorff, Jason, and Tindall, Leonora.** *Programming Rust: Fast, Safe Systems Development*, 2nd Edition.  
O'Reilly Media, 2021.  
ISBN: 978-1492052593  
Comprehensive guide to Rust programming including networking and concurrency.

## Web Resources

**Mozilla Developer Network (MDN) - HTTP**  
*https://developer.mozilla.org/en-US/docs/Web/HTTP*  
Comprehensive HTTP documentation with practical examples and browser compatibility notes.

**Internet Assigned Numbers Authority (IANA) - HTTP**  
*https://www.iana.org/assignments/http-parameters/http-parameters.xhtml*  
Official registry of HTTP status codes, methods, and header fields.

**W3C - World Wide Web Consortium**  
*https://www.w3.org/*  
Web standards organization with specifications for HTML, URLs, and web APIs.

**HTTP Working Group**  
*https://httpwg.org/*  
IETF HTTP Working Group developing HTTP specifications.

## Historical and Context

**Berners-Lee, Tim.** "Information Management: A Proposal."  
CERN, March 1989.  
*https://www.w3.org/History/1989/proposal.html*  
The original proposal for the World Wide Web and HTTP.

**Berners-Lee, Tim, Fielding, Roy T., and Frystyk, Henrik.** *Hypertext Transfer Protocol -- HTTP/1.0*.  
RFC 1945, May 1996.  
*https://www.rfc-editor.org/rfc/rfc1945.html*  
The first standardized HTTP specification (HTTP/1.0).

**Fielding, Roy Thomas.** *Architectural Styles and the Design of Network-based Software Architectures*.  
Doctoral dissertation, University of California, Irvine, 2000.  
*https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm*  
Defines REST (Representational State Transfer) architectural style and HTTP design principles.

## Additional Technical References

**POSIX.1-2017 (IEEE Std 1003.1-2017)**  
*https://pubs.opengroup.org/onlinepubs/9699919799/*  
POSIX specification for socket APIs and networking functions.

**Berkeley Sockets API**  
Original socket interface specification from BSD Unix, now part of POSIX.

**RFC 2119 - Key words for use in RFCs to Indicate Requirement Levels** (March 1997)  
Bradner, S.  
*https://www.rfc-editor.org/rfc/rfc2119.html*  
Defines MUST, SHOULD, MAY keywords used in technical specifications.

---

## Note on RFC Access

All RFCs are freely available at:
- **IETF RFC Editor**: https://www.rfc-editor.org/
- **IETF Datatracker**: https://datatracker.ietf.org/

RFCs are living documents. Newer RFCs often obsolete or update older ones. When referencing RFCs, check for the most current version on the IETF website.

## Further Reading

For readers interested in diving deeper into specific topics:

**Concurrency and Async Programming:**
- *Rust Atomics and Locks* by Mara Bos (O'Reilly, 2023)
- Tokio tutorial: https://tokio.rs/tokio/tutorial

**Web Security:**
- *The Tangled Web* by Michal Zalewski (No Starch Press, 2011)
- OWASP Web Security Testing Guide: https://owasp.org/

**Protocol Design:**
- *Designing Data-Intensive Applications* by Martin Kleppmann (O'Reilly, 2017)
- *Site Reliability Engineering* edited by Betsy Beyer et al. (O'Reilly, 2016)

**HTTP/2 and HTTP/3:**
- *HTTP/2 in Action* by Barry Pollard (Manning, 2019)
- *Learning HTTP/2* by Stephen Ludin and Javier Garza (O'Reilly, 2017)

---

*Note: This reference list reflects authoritative sources for HTTP, networking, and related protocols. While writing this book, content was synthesized from general knowledge of these specifications and best practices in the field.*