# Message Body

> "Are you of the body?"
>
> — *Star Trek: The Original Series, "The Return of the Archons"*

The **message body** is where HTTP carries its payload—the actual data being transmitted. While the request/status line and headers provide structure and metadata, the body contains the content: HTML pages, JSON data, images, videos, form submissions, file uploads, and more.

## What Is the Body?

The body is everything that comes after the empty line separator (`\r\n\r\n`). It's optional—many HTTP messages don't have a body at all.

```
POST /api/users HTTP/1.1\r\n
Host: api.example.com\r\n
Content-Type: application/json\r\n
Content-Length: 45\r\n
\r\n                                    ← Empty line separator
{"username":"alice","email":"a@example.com"}  ← Body starts here
```

The body is just bytes. What those bytes *mean* depends on the `Content-Type` header.

## When Messages Have Bodies

**Requests with bodies:**
- **POST** - Creating resources, submitting forms
- **PUT** - Replacing resources
- **PATCH** - Updating resources
- **Other methods** - Rarely, but technically allowed

**Requests without bodies:**
- **GET** - Just requesting data
- **DELETE** - Usually just identifying what to delete
- **HEAD** - Only wants headers back
- **OPTIONS** - Asking about available methods

**Responses with bodies:**
- **200 OK** - Successful GET returns the resource
- **201 Created** - Often returns the created resource
- **400 Bad Request** - Often includes error details
- **500 Internal Server Error** - Often includes error message

**Responses without bodies:**
- **204 No Content** - Explicitly means no body
- **304 Not Modified** - Use your cached version
- **HEAD responses** - Never have bodies (by definition)
- **1xx responses** - Informational, no body

## How Big Is the Body?

The body can be any size—from zero bytes to gigabytes. The server or client needs to know where the body ends. Two mechanisms handle this:

### 1. Content-Length Header

The most common approach: explicitly state the body size in bytes.

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 27

{"status":"success","id":5}
```

The body is exactly 27 bytes. The receiver reads 27 bytes and knows it's done.

**Advantages:**
- Simple and straightforward
- Receiver knows total size upfront
- Can allocate memory accordingly
- Can show progress bars

**Disadvantages:**
- Must know size before sending
- Can't stream dynamically-generated content easily

### 2. Chunked Transfer Encoding

When the sender doesn't know the size in advance, it can send the body in chunks:

```
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

7\r\n
Hello, \r\n
6\r\n
World!\r\n
0\r\n
\r\n
```

Each chunk has:
1. Chunk size in hexadecimal
2. `\r\n`
3. The chunk data
4. `\r\n`

The final chunk has size `0`, indicating the end.

**Advantages:**
- Can stream data as it's generated
- Don't need to buffer entire response
- Good for long-running responses

**Disadvantages:**
- More complex to parse
- Receiver doesn't know total size upfront
- Slightly more overhead

**Note:** You cannot use both `Content-Length` and `Transfer-Encoding: chunked` in the same message.

## Content-Type Determines Meaning

The `Content-Type` header tells the receiver how to interpret the body bytes.

### application/json

JSON data—the lingua franca of modern APIs:

```
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 82

{"username":"alice","email":"alice@example.com","password":"secure_pass_123"}
```

The receiver parses the body as JSON and extracts the fields.

### application/x-www-form-urlencoded

Traditional HTML form submissions:

```
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

username=alice&password=secret123
```

This is URL-encoded key-value pairs (like query parameters, but in the body):
- Spaces become `+` or `%20`
- Special characters are percent-encoded
- Parameters separated by `&`

### multipart/form-data

For forms with file uploads:

```
POST /upload HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Length: 256

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="username"

alice
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="photo.jpg"
Content-Type: image/jpeg

[binary image data]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

Each part has:
- A boundary delimiter
- Headers for that part
- The part's data

The boundary (specified in Content-Type) separates parts. The final boundary has `--` appended.

### text/html

HTML content:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 137

<!DOCTYPE html>
<html>
<head><title>Example</title></head>
<body><h1>Hello World</h1></body>
</html>
```

Browsers render this as a web page.

### text/plain

Plain text:

```
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 13

Hello, World!
```

No formatting, just raw text.

### application/octet-stream

Binary data—any file type:

```
HTTP/1.1 200 OK
Content-Type: application/octet-stream
Content-Length: 1048576

[binary data...]
```

This is the generic "I don't know what this is, just raw bytes" type.

### image/jpeg, image/png, video/mp4, etc.

Media files:

```
HTTP/1.1 200 OK
Content-Type: image/jpeg
Content-Length: 524288

[JPEG binary data...]
```

The browser or client knows how to display the image based on the Content-Type.

## Character Encoding

For text bodies, character encoding matters. UTF-8 is standard:

```
Content-Type: text/html; charset=utf-8
```

This tells the receiver that text in the body is encoded as UTF-8.

**Without explicit charset:**
- Receivers may assume a default (often UTF-8 or ISO-8859-1)
- Can lead to mojibake (garbled text) if assumption is wrong

**Always specify charset for text content:**
```
Content-Type: application/json; charset=utf-8
Content-Type: text/plain; charset=utf-8
Content-Type: text/html; charset=utf-8
```

## Empty Bodies

A body can be completely absent:

```
GET /api/users HTTP/1.1
Host: api.example.com

```

Notice: no body, not even an empty line after the headers' empty line.

Or explicitly zero-length:

```
DELETE /api/users/123 HTTP/1.1
Host: api.example.com
Content-Length: 0

```

Both mean "no body."

## Reading the Body

When implementing an HTTP server or client, reading the body requires careful handling:

### With Content-Length

```rust
// Conceptual approach
let content_length: usize = headers.get("content-length")
    .unwrap()
    .parse()
    .unwrap();

let mut body = vec![0u8; content_length];
stream.read_exact(&mut body)?;

// Now you have the complete body
```

Read exactly `Content-Length` bytes from the socket.

### With Chunked Encoding

```rust
// Conceptual approach
let mut body = Vec::new();

loop {
    // Read chunk size line (hex number followed by \r\n)
    let size_line = read_line(&mut stream)?;
    let chunk_size = usize::from_str_radix(size_line.trim(), 16)?;

    if chunk_size == 0 {
        // Last chunk
        read_line(&mut stream)?; // Read final \r\n
        break;
    }

    // Read chunk data
    let mut chunk = vec![0u8; chunk_size];
    stream.read_exact(&mut chunk)?;
    body.extend_from_slice(&chunk);

    // Read trailing \r\n after chunk data
    read_line(&mut stream)?;
}

// Now you have the complete body
```

More complex, but necessary for streamed content.

### Without Content-Length or Chunked

If neither is present:
- **Requests**: No body (body requires explicit length or chunking)
- **Responses**: Read until connection closes (HTTP/1.0 style, rarely used)

## Compression

Bodies can be compressed to save bandwidth:

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Encoding: gzip
Content-Length: 128

[gzipped data - 128 bytes compressed]
```

The receiver must:
1. Read the compressed body (128 bytes)
2. Decompress it (might become 500 bytes)
3. Parse the decompressed data as JSON

**Common encodings:**
- `gzip` - Most common
- `deflate` - Less common
- `br` (Brotli) - Modern, better compression
- `identity` - No compression (default if not specified)

**Important:** `Content-Length` is the size of the *compressed* body, not the decompressed size.

## Body in GET Requests

Technically, GET requests *can* have bodies, but it's strongly discouraged and rarely supported:

```
GET /search HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 25

{"query":"search term"}
```

**Problems:**
- Semantically wrong (GET should only retrieve, not send data)
- Many servers ignore GET request bodies
- Proxies and caches may drop the body
- Many HTTP libraries don't support it

**Instead, use query parameters:**
```
GET /search?query=search+term HTTP/1.1
```

Or use POST if you need to send complex data.

## Large Bodies

When bodies are large (megabytes or gigabytes):

**Streaming is essential:**
Don't buffer the entire body in memory. Process it as it arrives:

```rust
// Read and write in chunks
let mut buffer = [0u8; 8192]; // 8KB buffer

loop {
    let bytes_read = stream.read(&mut buffer)?;
    if bytes_read == 0 { break; }

    // Process this chunk
    process_chunk(&buffer[..bytes_read]);
}
```

**Progress indication:**
With `Content-Length`, you can show progress:
```
Downloaded 4,567,890 of 10,000,000 bytes (45.7%)
```

**Memory considerations:**
A server handling 1000 concurrent requests, each with a 10MB body, would need 10GB just for request bodies. Streaming prevents this.

## Body Edge Cases

### Mismatched Content-Length

```
Content-Length: 100

[only 50 bytes of data]
```

The receiver will hang waiting for the remaining 50 bytes until timeout. This is a common bug.

### Content-Length Too Large

```
Content-Length: 999999999

[small amount of data]
```

The receiver allocates a huge buffer or hangs waiting. Servers should reject requests with unreasonably large Content-Length values.

### Both Content-Length and Transfer-Encoding

```
Content-Length: 100
Transfer-Encoding: chunked
```

This is invalid. If both are present, Transfer-Encoding takes precedence and Content-Length should be ignored.

### No Content-Length or Transfer-Encoding (Response)

In HTTP/1.0, responses could omit both and just close the connection when done:

```
HTTP/1.0 200 OK
Content-Type: text/html

[HTML content...]
[server closes connection - that's how you know it's done]
```

This prevents connection reuse. HTTP/1.1 requires either Content-Length or Transfer-Encoding.

## Security Considerations

Bodies can be attack vectors:

**Denial of Service:**
```
Content-Length: 999999999999
```
Attacker claims to send terabytes. Server allocates memory and crashes.

**Solution:** Limit maximum body size (e.g., 10MB for APIs, 100MB for file uploads).

**Request Smuggling:**
Mismatched Content-Length or malformed chunking can confuse proxies and servers:
```
Content-Length: 10
Transfer-Encoding: chunked

[malicious data]
```

**Solution:** Strict parsing. Reject ambiguous requests.

**Oversized Uploads:**
Users upload 5GB files to your 1GB storage quota.

**Solution:** Check Content-Length *before* reading the body. Reject if too large.

## The Body Contains the Payload

While the request/status line and headers provide the *metadata* and *instructions*, the body contains the *actual data*:

- The HTML page you're viewing
- The JSON response from an API
- The image you're downloading
- The form data you're submitting
- The file you're uploading

The body is the reason HTTP exists—to transfer content across the internet.

## Real-World Example

A complete request with a JSON body:

```
POST /api/v1/posts HTTP/1.1
Host: blog.example.com
User-Agent: MyBlogClient/1.0
Accept: application/json
Content-Type: application/json; charset=utf-8
Content-Length: 156
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

{"title":"Building HTTP Servers in Rust","content":"In this post, we'll explore how to build a production-ready HTTP server...","tags":["rust","http","servers"]}
```

And the response:

```
HTTP/1.1 201 Created
Date: Wed, 04 Oct 2025 16:45:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 234
Location: /api/v1/posts/42
X-Request-ID: a1b2c3d4-e5f6-7890-abcd-ef1234567890

{"id":42,"title":"Building HTTP Servers in Rust","slug":"building-http-servers-rust","author_id":15,"created_at":"2025-10-04T16:45:00Z","url":"/posts/building-http-servers-rust","view_count":0}
```

## Looking Ahead

We've now covered all the main components of HTTP messages:
- Request line (what you want)
- Status line (what happened)
- Headers (metadata about the message)
- Body (the actual data)

But there's one more crucial detail that ties everything together: **line endings and encoding**. In the next section, we'll explore the nitty-gritty details that can make or break your HTTP implementation—the stuff that trips up even experienced developers.

Getting these details right is the difference between a working HTTP server and one that mysteriously fails in production.
