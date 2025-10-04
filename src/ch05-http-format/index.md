# HTTP Message Format

In Chapter 4, we learned the fundamental concepts: HTTP is a stateless, request-response protocol built on the client-server model. Now it's time to get concrete. What does an HTTP message actually *look like*?

## Messages Are Just Text

Remember that HTTP is a text-based protocol. An HTTP message is just a string of characters—plain text that you could read with your own eyes. When we send an HTTP request or response, we're literally writing text through a TCP socket.

Let's look at a real HTTP request:

```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html

```

And a real HTTP response:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 137

<!DOCTYPE html>
<html>
<head><title>Example</title></head>
<body><h1>Hello World</h1></body>
</html>
```

That's it. No magic, no binary encoding (in HTTP/1.1), just text. These are the actual bytes that flow through the TCP connection.

## Two Types of Messages

There are exactly two kinds of HTTP messages:

1. **Requests** (sent by clients to servers)
2. **Responses** (sent by servers back to clients)

They have similar but distinct structures. Both follow a pattern of:
- A special first line
- Headers
- An empty line
- An optional message body

## Request Structure

An HTTP request has this structure:

```
[Request Line]
[Header 1]
[Header 2]
...
[Header N]
[Empty Line]
[Optional Message Body]
```

Breaking it down:

```
GET /api/users HTTP/1.1          ← Request Line
Host: api.example.com             ← Headers start here
Accept: application/json
Authorization: Bearer token123
                                  ← Empty line (CRLF CRLF)
                                  ← Body would go here (if present)
```

The request line tells the server **what** you want. The headers provide **metadata** about the request. The body (if present) contains **data** you're sending.

## Response Structure

An HTTP response has a similar structure:

```
[Status Line]
[Header 1]
[Header 2]
...
[Header N]
[Empty Line]
[Optional Message Body]
```

Breaking it down:

```
HTTP/1.1 200 OK                   ← Status Line
Content-Type: application/json    ← Headers start here
Content-Length: 45
Cache-Control: no-cache
                                  ← Empty line (CRLF CRLF)
{"users": ["alice", "bob"]}      ← Body starts here
```

The status line tells the client **what happened**. The headers provide **metadata** about the response. The body contains **data** the server is sending back.

## The First Line Is Special

The first line of an HTTP message is critically important—it identifies what kind of message this is:

**For requests**, the first line is the **request line**:
```
METHOD /path/to/resource HTTP/VERSION
```
Example: `GET /api/users HTTP/1.1`

**For responses**, the first line is the **status line**:
```
HTTP/VERSION STATUS_CODE REASON_PHRASE
```
Example: `HTTP/1.1 200 OK`

The first line is the only way to distinguish a request from a response. There's no "message type" field—you know it's a request because it starts with a method, and you know it's a response because it starts with `HTTP/`.

## Headers Provide Metadata

After the first line come the headers—key-value pairs that provide information about the message:

```
Header-Name: Header-Value
Another-Header: Another-Value
Yet-Another: Value
```

Headers are incredibly flexible. They can specify:
- Content type and length
- Caching behavior
- Authentication credentials
- Acceptable response formats
- Cookies
- Compression
- And much more

We'll explore headers in detail in section 5.3.

## The Empty Line Separator

There's always an empty line between the headers and the body. This empty line is **mandatory** and serves as the boundary:

```
GET /api/data HTTP/1.1
Host: example.com
Accept: application/json
                              ← This empty line is CRUCIAL
{"query": "search term"}
```

Without this empty line, the server wouldn't know where headers end and the body begins. The empty line is the signal: "Headers are done, body (if any) starts now."

## The Optional Body

Many HTTP messages include a body—the actual data being transmitted:

**Requests with bodies:**
- POST requests submitting form data
- PUT requests uploading files
- PATCH requests updating resources

**Requests without bodies:**
- GET requests (you're just asking for data)
- DELETE requests (usually just identifying what to delete)
- HEAD requests (you only want headers back)

**Responses with bodies:**
- HTML pages
- JSON data
- Images, videos, files
- Error messages

**Responses without bodies:**
- 204 No Content responses
- 304 Not Modified responses
- Responses to HEAD requests

The `Content-Length` header tells you how many bytes the body contains. If there's no body, this header might be absent or set to 0.

## Character Encoding

HTTP messages use **ASCII** for the request/status line and headers. The body can use any encoding (specified by the Content-Type header), but the "envelope"—everything before the empty line—must be ASCII.

This is why you'll see encoded characters in URLs and headers:
- Space becomes `%20`
- Non-ASCII characters are percent-encoded
- Headers with special characters are encoded

## Line Endings: CRLF

HTTP uses **CRLF** (Carriage Return + Line Feed, `\r\n`) to end each line. Not just `\n` (Unix-style) or just `\r` (old Mac-style), but both: `\r\n`.

In bytes, this is:
- `\r` = 0x0D (13 in decimal)
- `\n` = 0x0A (10 in decimal)

So every line in an HTTP message ends with these two bytes:

```
GET /index.html HTTP/1.1\r\n
Host: example.com\r\n
\r\n
```

The empty line separator is actually `\r\n\r\n`—two CRLFs in a row.

This might seem pedantic, but it's crucial. If you use the wrong line endings, HTTP parsers will fail or behave unpredictably. We'll explore this more in section 5.5.

## Visualizing the Structure

Let's see a complete request with all parts labeled:

```
POST /api/login HTTP/1.1\r\n          ← Request Line
Host: api.example.com\r\n              ← Headers
Content-Type: application/json\r\n
Content-Length: 45\r\n
Authorization: Bearer abc123\r\n
\r\n                                   ← Empty Line (separator)
{"username":"alice","password":"pw"}  ← Body (45 bytes)
```

And a complete response:

```
HTTP/1.1 200 OK\r\n                    ← Status Line
Content-Type: application/json\r\n     ← Headers
Content-Length: 27\r\n
Set-Cookie: session=xyz789\r\n
\r\n                                   ← Empty Line (separator)
{"status":"login successful"}         ← Body (27 bytes)
```

## Why This Format?

This format might seem verbose compared to binary protocols, but it has advantages:

**Human-readable**: You can read and debug HTTP messages with your eyes. Try doing that with a binary protocol!

**Simple to parse**: Split on `\r\n`, parse the first line specially, then parse key-value pairs. It's straightforward.

**Extensible**: New headers can be added without breaking old parsers (they just ignore unknown headers).

**Text-based tools work**: You can use `telnet`, `netcat`, or any text editor to craft HTTP messages.

**Easy to test**: Writing test cases is simple when you can just write strings.

The downside? It's verbose and slower to parse than binary. This is why HTTP/2 moved to binary framing—but the *concepts* remain the same. Understanding HTTP/1.1's text format gives you the foundation to understand all HTTP versions.

## What's Next

Now that we've seen the overall structure, we'll dive into each component:

- **Request Line** (5.1): How to specify what you want
- **Status Line** (5.2): How servers indicate what happened
- **Headers** (5.3): All the metadata you can send
- **Message Body** (5.4): Sending and receiving data
- **Line Endings and Encoding** (5.5): The nitty-gritty details that trip people up

By the end of this chapter, you'll be able to craft HTTP messages by hand and understand exactly what every byte means. This knowledge is essential for building our HTTP server in Chapter 9.

Let's start with the request line—the most important line in any HTTP request.
