# Headers

After the request or status line comes the most flexible part of HTTP messages: **headers**. Headers are key-value pairs that provide metadata about the request or response. They're the workhorses of HTTP, carrying everything from authentication credentials to caching instructions.

## What Are Headers?

Headers provide **context** about the HTTP message. While the request/status line tells you *what* is happening, headers tell you *how*, *why*, and *under what conditions*.

A header is a simple key-value pair:

```
Header-Name: Header-Value
```

For example:
```
Content-Type: application/json
Content-Length: 1234
Authorization: Bearer token123abc
```

Every header follows this format: name, colon, space, value, line ending.

## Header Format

Headers must follow specific formatting rules:

### Basic Structure

```
Header-Name: Header-Value\r\n
```

Breaking it down:
- **Header name**: Case-insensitive, typically uses `Title-Case-With-Hyphens`
- **Colon**: Separates name from value
- **Space**: Single space after the colon (technically optional but conventional)
- **Header value**: Can contain almost anything
- **Line ending**: `\r\n` like all HTTP lines

### Multiple Headers

Requests and responses typically have many headers:

```
GET /api/users HTTP/1.1\r\n
Host: api.example.com\r\n
User-Agent: Mozilla/5.0\r\n
Accept: application/json\r\n
Authorization: Bearer abc123\r\n
\r\n
```

Each header is on its own line. The empty line (`\r\n\r\n`) signals the end of headers.

### Header Names Are Case-Insensitive

By specification, header names are case-insensitive:

```
Content-Type: application/json
content-type: application/json
CONTENT-TYPE: application/json
```

These are all equivalent. However, by convention:
- Use `Title-Case-With-Hyphens` when writing headers
- Parse them case-insensitively

### Header Values Are Case-Sensitive

While names are case-insensitive, **values** are case-sensitive (unless the specific header defines otherwise):

```
Content-Type: application/json    ✓ Correct
Content-Type: Application/JSON    ✗ Wrong (most parsers would reject this)
```

### Whitespace Rules

**After the colon:**
Optional whitespace is allowed (and conventional):

```
Content-Type: application/json     (Standard - space after colon)
Content-Type:application/json      (Valid but unconventional)
Content-Type:  application/json    (Valid - multiple spaces)
```

**In the value:**
Leading and trailing whitespace in values should be trimmed:

```
Content-Type:   application/json
              ^^^                ^^^
              Should be trimmed
```

**Line folding (deprecated):**
Historically, long header values could be split across lines:

```
Content-Type: application/json;
              charset=utf-8
```

This is **deprecated** in HTTP/1.1 and should not be used. Keep each header on a single line.

## Common Request Headers

Let's explore headers you'll frequently see in requests:

### Host

```
Host: www.example.com
```

**Purpose:** Specifies the domain name of the server.

**Required:** Yes, in HTTP/1.1. This is the only mandatory header.

**Why it exists:** Multiple websites can share the same IP address (virtual hosting). The Host header tells the server which website you want.

```
GET /index.html HTTP/1.1
Host: example.com
```

Without the Host header, the server wouldn't know which site to serve if it hosts multiple domains.

### User-Agent

```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
```

**Purpose:** Identifies the client software making the request.

**Typical values:**
- Browser: `Mozilla/5.0...`
- Mobile app: `MyApp/1.0 (iOS 15.0)`
- Script: `curl/7.68.0`
- Bot: `Googlebot/2.1`

Servers use this to:
- Serve different content to different devices
- Block or allow specific user agents
- Gather analytics
- Detect bots

### Accept

```
Accept: text/html,application/json,*/*
```

**Purpose:** Tells the server what content types the client can handle.

**Format:** Comma-separated list of MIME types, optionally with quality values:

```
Accept: text/html, application/json;q=0.9, */*;q=0.8
```

The `q` parameter (quality value) indicates preference, from 0 to 1. Higher is more preferred.

**Related headers:**
- `Accept-Language: en-US,en;q=0.9,es;q=0.8`
- `Accept-Encoding: gzip, deflate, br`
- `Accept-Charset: utf-8, iso-8859-1;q=0.5`

### Content-Type

```
Content-Type: application/json
```

**Purpose:** Tells the server what type of data is in the request body.

**Common values:**
- `application/json` - JSON data
- `application/x-www-form-urlencoded` - Form data
- `multipart/form-data` - File uploads
- `text/plain` - Plain text
- `application/xml` - XML data

**With charset:**
```
Content-Type: application/json; charset=utf-8
```

### Content-Length

```
Content-Length: 1234
```

**Purpose:** Specifies the size of the request body in bytes.

**Critical for:** Knowing when the body ends. Without this (or chunked encoding), the server doesn't know how much data to read.

**Example:**
```
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 45

{"username":"alice","password":"secret123"}
```

The body is exactly 45 bytes.

### Authorization

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Purpose:** Provides authentication credentials.

**Common schemes:**
- `Basic base64(username:password)`
- `Bearer <token>` (OAuth 2.0, JWT)
- `Digest <credentials>` (digest authentication)

**Example:**
```
Authorization: Basic dXNlcjpwYXNz
```

### Cookie

```
Cookie: session_id=abc123; user_pref=dark_mode
```

**Purpose:** Sends cookies from the client to the server.

**Format:** Semicolon-separated name-value pairs:

```
Cookie: name1=value1; name2=value2; name3=value3
```

Cookies are how HTTP maintains state across requests (remember, HTTP itself is stateless).

### Referer (sic)

```
Referer: https://example.com/page1
```

**Purpose:** Tells the server which page linked to the current request.

**Note:** Yes, it's misspelled ("Referer" instead of "Referrer"). This typo was in the original spec and is now permanent.

**Privacy:** Browsers sometimes omit this header for privacy reasons.

### Connection

```
Connection: keep-alive
```

**Purpose:** Controls whether the connection should stay open after the request.

**Values:**
- `keep-alive` - Keep connection open for reuse (default in HTTP/1.1)
- `close` - Close connection after response

## Common Response Headers

Now let's look at headers in responses:

### Content-Type

```
Content-Type: text/html; charset=utf-8
```

**Purpose:** Tells the client what type of data is in the response body.

The client uses this to know how to process the response (render HTML, parse JSON, display image, etc.).

### Content-Length

```
Content-Length: 2048
```

**Purpose:** Size of the response body in bytes.

Helps the client know when it has received the complete response.

### Set-Cookie

```
Set-Cookie: session_id=abc123; Path=/; HttpOnly; Secure
```

**Purpose:** Tells the client to store a cookie.

**Attributes:**
- `Path=/` - Cookie applies to all paths
- `Domain=example.com` - Cookie valid for domain
- `Expires=Wed, 21 Oct 2025 07:28:00 GMT` - When cookie expires
- `Max-Age=3600` - Cookie lifetime in seconds
- `Secure` - Only send over HTTPS
- `HttpOnly` - JavaScript cannot access (security)
- `SameSite=Strict` - CSRF protection

**Multiple cookies:**
```
Set-Cookie: session_id=abc123; HttpOnly
Set-Cookie: user_pref=dark; Path=/
```

Each cookie needs its own `Set-Cookie` header.

### Location

```
Location: https://example.com/new-location
```

**Purpose:** Used with 3xx redirects to tell the client where to go.

**Example:**
```
HTTP/1.1 301 Moved Permanently
Location: https://example.com/new-url
```

The client should make a new request to the URL in the Location header.

### Cache-Control

```
Cache-Control: max-age=3600, public
```

**Purpose:** Directives for caching mechanisms.

**Common directives:**
- `no-cache` - Must revalidate before using cache
- `no-store` - Don't cache at all
- `public` - Any cache can store
- `private` - Only client cache, not proxies
- `max-age=3600` - Cache for 3600 seconds
- `must-revalidate` - Must check with server when stale

### ETag

```
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

**Purpose:** A unique identifier for a specific version of a resource.

Used with caching:

```
Client: GET /resource HTTP/1.1
Server: HTTP/1.1 200 OK
        ETag: "abc123"
        [resource data]

[Later, client checks if still valid]

Client: GET /resource HTTP/1.1
        If-None-Match: "abc123"

Server: HTTP/1.1 304 Not Modified
        [No body - use your cached version]
```

### Server

```
Server: nginx/1.18.0
```

**Purpose:** Identifies the server software.

**Examples:**
- `Apache/2.4.41`
- `nginx/1.18.0`
- `Caddy/2.4.6`

Often omitted or obscured for security reasons.

### Date

```
Date: Wed, 21 Oct 2025 07:28:00 GMT
```

**Purpose:** When the response was generated.

**Format:** HTTP-date format (RFC 7231):
```
Date: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT
```

Always in GMT timezone.

### Content-Encoding

```
Content-Encoding: gzip
```

**Purpose:** What encoding/compression is applied to the body.

**Common values:**
- `gzip` - Gzip compression
- `deflate` - Deflate compression
- `br` - Brotli compression
- `identity` - No encoding (default)

The client must decompress the body before using it.

### Transfer-Encoding

```
Transfer-Encoding: chunked
```

**Purpose:** How the message body is encoded for transfer.

**Chunked transfer:**
When the server doesn't know the content length in advance, it can send the body in chunks:

```
HTTP/1.1 200 OK
Transfer-Encoding: chunked

5\r\n
Hello\r\n
6\r\n
 World\r\n
0\r\n
\r\n
```

Each chunk has its size in hex, followed by the data.

## Custom Headers

You can define custom headers for your application:

```
X-Request-ID: 550e8400-e29b-41d4-a716-446655440000
X-API-Version: v2
X-Rate-Limit-Remaining: 99
```

**Convention:** Custom headers traditionally started with `X-`, but this is deprecated. Modern practice is to use application-specific names without the prefix:

```
Request-ID: 550e8400-e29b-41d4-a716-446655440000
API-Version: v2
Rate-Limit-Remaining: 99
```

## Header Ordering

Headers can appear in any order:

```
GET /api/data HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer token123
```

Is equivalent to:

```
GET /api/data HTTP/1.1
Authorization: Bearer token123
Accept: application/json
Host: api.example.com
```

However, `Host` is often placed first by convention.

## Duplicate Headers

What if the same header appears twice?

```
Cache-Control: no-cache
Cache-Control: no-store
```

**Behavior depends on the header:**

- **Most headers:** Values are combined with commas:
  ```
  Accept: text/html
  Accept: application/json
  ```
  Equivalent to: `Accept: text/html, application/json`

- **Set-Cookie:** Each is treated separately (you can set multiple cookies)

- **Some headers:** Last value wins (implementation-specific)

Generally, avoid duplicate headers except where explicitly allowed (like `Set-Cookie`).

## Empty Header Values

Headers can have empty values:

```
X-Custom-Header:
```

This is valid but uncommon. The header exists but has no value.

## Very Long Headers

Headers can be long:

```
Cookie: session_id=abc123; preferences=font:14px,theme:dark,language:en-US,notifications:enabled,layout:grid,sidebar:collapsed
```

There's no formal limit, but practical limits exist:
- Servers often reject requests with headers > 8KB total
- Individual header lines > 8KB may be rejected
- Very long headers impact performance

## Header Parsing Strategy

When building an HTTP parser:

```rust
// Conceptual approach
for line in headers_section.lines() {
    if line.is_empty() {
        break; // Empty line means headers are done
    }

    // Split on first colon
    if let Some(colon_pos) = line.find(':') {
        let name = &line[..colon_pos];
        let value = &line[colon_pos + 1..].trim();

        // Store header (case-insensitive name)
        headers.insert(name.to_lowercase(), value.to_string());
    }
}
```

Key considerations:
- Split on first colon only (values can contain colons)
- Trim whitespace from values
- Store names case-insensitively
- Handle duplicate headers appropriately

## Security Considerations

Headers can be attack vectors:

**Header injection:**
If you blindly trust user input in headers:
```
X-User-Input: user_data\r\nMalicious-Header: evil
```

Always validate and sanitize.

**Sensitive data:**
Don't log Authorization headers or cookies—they contain secrets.

**Size limits:**
Enforce maximum header sizes to prevent memory exhaustion.

**Host header attacks:**
Validate the Host header to prevent host header injection attacks.

## Headers Make HTTP Flexible

Headers are why HTTP is so adaptable. Need authentication? Add an Authorization header. Need caching? Add Cache-Control. Need custom application data? Add your own headers.

The request/status line tells you *what*. Headers tell you *everything else*.

## Real-World Example

Here's a complete request with realistic headers:

```
POST /api/v1/users HTTP/1.1
Host: api.example.com
User-Agent: MyApp/2.1.0 (iOS 15.0)
Accept: application/json
Content-Type: application/json
Content-Length: 82
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
X-Request-ID: 7b8d3c4e-5f6a-4b2c-8d9e-1a2b3c4d5e6f
Accept-Encoding: gzip, deflate
Connection: keep-alive

{"username":"alice","email":"alice@example.com","password":"secure_password_123"}
```

And a complete response:

```
HTTP/1.1 201 Created
Date: Wed, 04 Oct 2025 15:30:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 156
Location: /api/v1/users/12345
X-Request-ID: 7b8d3c4e-5f6a-4b2c-8d9e-1a2b3c4d5e6f
X-Rate-Limit-Remaining: 99
X-Rate-Limit-Reset: 1696435800
Cache-Control: no-store
Connection: keep-alive

{"id":12345,"username":"alice","email":"alice@example.com","created_at":"2025-10-04T15:30:00Z","profile_url":"/api/v1/users/12345"}
```

## Looking Ahead

We've now covered the first three parts of HTTP messages:
- Request line / Status line
- Headers

Next, we'll explore the **message body**—where the actual data lives. Whether it's HTML, JSON, images, or files, the body carries the payload of HTTP communication.
