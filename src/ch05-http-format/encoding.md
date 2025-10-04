# Line Endings and Encoding

We've covered the structure of HTTP messages—request lines, status lines, headers, and bodies. But there's a layer beneath all of this: the actual bytes on the wire. Getting line endings and encoding right is critical. These are the details that separate a working implementation from one that mysteriously breaks.

## CRLF: The HTTP Line Ending

HTTP uses **CRLF** (Carriage Return + Line Feed) to end every line. This is `\r\n` in most programming languages.

In raw bytes:
- `\r` is byte `0x0D` (13 in decimal) - Carriage Return
- `\n` is byte `0x0A` (10 in decimal) - Line Feed

**Every line in an HTTP message ends with both:**

```
GET /index.html HTTP/1.1\r\n
Host: example.com\r\n
\r\n
```

Not `\n` alone (Unix/Linux). Not `\r` alone (old Mac). Both: `\r\n`.

## Why CRLF?

This convention comes from the early days of computing. Teletype machines had two operations:
- **Carriage Return**: Move the print head to the start of the line
- **Line Feed**: Advance the paper to the next line

To start a new line, you needed both operations: return to the start (`\r`), then advance down (`\n`).

HTTP inherited this from earlier internet protocols (SMTP, FTP, etc.), which themselves inherited it from teletype machines. It's anachronistic, but it's part of the spec.

## The Empty Line Separator

The boundary between headers and body is an empty line—which is actually `\r\n\r\n`:

```
POST /api/data HTTP/1.1\r\n
Host: example.com\r\n
Content-Length: 13\r\n
\r\n                        ← This is \r\n (empty line)
Hello, World!
```

Breaking it down:
- `Content-Length: 13\r\n` - Last header line
- `\r\n` - Empty line (no content between the two CRLFs)

Together: `13\r\n\r\n` (the last header's CRLF, then the empty line's CRLF).

## Common Line Ending Mistakes

### Using Only `\n`

```
GET /index.html HTTP/1.1\n
Host: example.com\n
\n
```

**Result:** Most HTTP parsers will reject this. Some might accept it, but you can't rely on it.

**Why it fails:** HTTP parsers specifically look for `\r\n`. When they see just `\n`, they often:
- Treat it as part of the header value (breaking parsing)
- Reject the message as malformed
- Hang waiting for the "real" line ending

### Using Only `\r`

```
GET /index.html HTTP/1.1\r
Host: example.com\r
\r
```

**Result:** Even worse than `\n` alone. Virtually no HTTP implementation will accept this.

### Mixed Line Endings

```
GET /index.html HTTP/1.1\r\n
Host: example.com\n
Content-Length: 0\r\n
\r\n
```

**Result:** The second line has only `\n`, breaking the parser. Inconsistent line endings are a recipe for mysterious bugs.

### Forgetting the Empty Line

```
POST /api/data HTTP/1.1\r\n
Content-Length: 5\r\n
Hello
```

**Result:** The parser never sees the empty line separator, so it keeps reading lines as headers. It might treat "Hello" as a malformed header, or hang waiting for the separator.

## Platform Differences

Different operating systems use different line endings in text files:

| Platform | Line Ending | Bytes | Name |
|----------|-------------|-------|------|
| Windows | `\r\n` | `0D 0A` | CRLF |
| Unix/Linux | `\n` | `0A` | LF |
| Old Mac (pre-OSX) | `\r` | `0D` | CR |
| Mac OSX+ | `\n` | `0A` | LF |

**HTTP always uses CRLF** regardless of platform. When writing HTTP code:

```rust
// DON'T use platform-specific line endings
stream.write_all(b"GET / HTTP/1.1\n")?;  // Wrong on Windows

// DO use explicit CRLF
stream.write_all(b"GET / HTTP/1.1\r\n")?;  // Correct everywhere
```

In Rust, `"\n"` in a string literal produces LF only. You must write `"\r\n"` explicitly for CRLF.

## Character Encoding in HTTP

HTTP messages have two distinct parts with different encoding rules:

### 1. The Envelope (Headers)

Everything before the body—the request/status line and headers—must be **ASCII**.

```
GET /index.html HTTP/1.1\r\n
Host: example.com\r\n
User-Agent: MyClient/1.0\r\n
\r\n
```

Every character here is ASCII (bytes 0-127). No Unicode, no UTF-8 multi-byte sequences.

**Why ASCII?**
- Simplicity: ASCII is universal and unambiguous
- Compatibility: All systems understand ASCII
- Efficiency: Single-byte characters, easy to parse

### 2. The Body

The body can use **any encoding**, specified by the `Content-Type` header:

```
HTTP/1.1 200 OK\r\n
Content-Type: text/html; charset=utf-8\r\n
\r\n
<!DOCTYPE html>
<html>
<head><title>Hello 世界</title></head>
</html>
```

The body contains UTF-8 encoded text (notice the Chinese characters "世界"). The `charset=utf-8` parameter tells the client how to decode it.

**Common encodings:**
- `utf-8` - Universal, handles all Unicode (recommended)
- `iso-8859-1` - Latin-1, Western European characters
- `windows-1252` - Windows Western European
- Binary (no charset) - For images, videos, etc.

## Percent Encoding (URL Encoding)

Since HTTP headers must be ASCII, how do we include non-ASCII characters or special characters in URLs?

**Answer:** Percent encoding.

### How Percent Encoding Works

1. Take the character
2. Encode it as UTF-8 bytes
3. Represent each byte as `%XX` where `XX` is hexadecimal

**Example: Space**
```
Character: ' ' (space)
UTF-8 byte: 0x20
Percent encoded: %20
```

**Example: Non-ASCII**
```
Character: é
UTF-8 bytes: 0xC3 0xA9
Percent encoded: %C3%A9
```

### Where Percent Encoding Is Used

**In URLs (paths and query parameters):**

```
GET /search?q=hello%20world&city=S%C3%A3o%20Paulo HTTP/1.1
```

This is asking for:
- Query parameter `q` with value "hello world"
- Query parameter `city` with value "São Paulo"

**Common encodings:**
- Space → `%20` (or `+` in query strings only)
- `?` → `%3F`
- `#` → `%23`
- `&` → `%26`
- `=` → `%3D`
- `/` → `%2F` (when you want a literal `/` in a path segment)
- `%` → `%25` (must encode the percent sign itself!)

### Reserved Characters

Certain characters have special meaning in URLs and must be encoded when used literally:

```
: / ? # [ ] @ ! $ & ' ( ) * + , ; =
```

**Example:**
If you want a path that literally contains a question mark:

```
GET /what%3F HTTP/1.1
```

This is asking for a resource named "what?", not starting a query string.

### Unreserved Characters

These characters never need encoding:

```
A-Z a-z 0-9 - _ . ~
```

You can use them freely in URLs without encoding.

### Encoding Spaces: %20 vs +

Two ways to encode spaces:
- `%20` - Works everywhere in a URL
- `+` - Only in query parameter values (a shorthand)

```
/search?q=hello+world    (Valid - + becomes space)
/search?q=hello%20world  (Valid - %20 is space)
/hello+world             (Invalid - + in path is literal +)
/hello%20world           (Valid - %20 is space)
```

**Rule of thumb:** Use `%20` everywhere, or use `+` only in query values.

## Header Value Encoding

Headers must be ASCII, but what if you need non-ASCII in a header value?

### Option 1: Percent Encoding (Limited)

Some headers allow percent encoding:

```
Content-Disposition: attachment; filename="document%20%281%29.pdf"
```

But this isn't universally supported for all headers.

### Option 2: RFC 2047 Encoding

For headers like `Subject` in email (also used in some HTTP contexts):

```
Subject: =?UTF-8?B?SGVsbG8gV29ybGQ=?=
```

This is Base64-encoded "Hello World". Format:
```
=?charset?encoding?encoded-text?=
```

Where encoding is:
- `B` for Base64
- `Q` for Quoted-Printable

### Option 3: Avoid It

**Best practice:** Keep header values ASCII when possible. Use the body for non-ASCII data.

## Content-Type and Charset

For text bodies, always specify the charset:

```
Content-Type: text/html; charset=utf-8
Content-Type: application/json; charset=utf-8
Content-Type: text/plain; charset=utf-8
```

**Without charset:**
The receiver guesses, often defaulting to `ISO-8859-1` or `UTF-8`. Guessing can lead to mojibake (garbled text).

**Example of mojibake:**
- Server sends UTF-8: `"Hello 世界"` as bytes `48 65 6C 6C 6F 20 E4 B8 96 E7 95 8C`
- Client assumes ISO-8859-1: Interprets those UTF-8 bytes as individual characters
- Result: `"Hello ä¸–ç•Œ"` (garbage)

## Binary vs Text

HTTP distinguishes between text and binary content:

**Text:**
- Has a `charset` parameter
- Line endings might be interpreted or transformed
- Subject to encoding issues

**Binary:**
- No `charset` parameter
- Raw bytes, no interpretation
- Examples: `image/jpeg`, `application/octet-stream`, `video/mp4`

```
Content-Type: image/jpeg        (Binary - no charset)
Content-Type: text/html         (Text - but which charset?)
Content-Type: text/html; charset=utf-8  (Text - UTF-8 specified)
```

## Debugging Line Ending Issues

When your HTTP implementation mysteriously breaks, line endings are often the culprit.

### Hexdump Is Your Friend

View the actual bytes being sent:

```bash
# Capture HTTP traffic
tcpdump -i any -w capture.pcap port 8080

# View as hex
xxd capture.pcap
```

Look for:
- `0D 0A` (CRLF) - Correct
- `0A` alone - Wrong
- `0D` alone - Wrong

### Common Symptoms

**Symptom:** Server returns 400 Bad Request for seemingly valid requests
**Cause:** Wrong line endings (probably `\n` instead of `\r\n`)

**Symptom:** Request appears to hang, never completes
**Cause:** Missing empty line separator (`\r\n\r\n`)

**Symptom:** Header values include unexpected characters
**Cause:** Line ending bytes interpreted as part of the value

**Symptom:** Unicode characters display as garbage
**Cause:** Charset mismatch or not specified

### Testing with Netcat

Send raw HTTP manually:

```bash
# Use \r\n explicitly
printf "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n" | nc example.com 80
```

This helps you verify what the server expects.

### Testing with Telnet

```bash
telnet example.com 80
```

Then type:
```
GET / HTTP/1.1
Host: example.com

```

Press Enter twice for the empty line. Telnet might not send CRLF properly—this is a test to see how forgiving the server is.

## Best Practices

### 1. Always Use CRLF Explicitly

```rust
// Good
stream.write_all(b"HTTP/1.1 200 OK\r\n")?;

// Bad - relies on platform line endings
stream.write_all(b"HTTP/1.1 200 OK\n")?;
```

### 2. Use Raw Byte Strings

In Rust:

```rust
// This is binary-safe
let request = b"GET / HTTP/1.1\r\nHost: example.com\r\n\r\n";

// This might have encoding issues depending on source file encoding
let request = "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n".as_bytes();
```

### 3. Validate Line Endings

When parsing HTTP:

```rust
if !line.ends_with("\r\n") {
    return Err("Invalid line ending");
}
```

### 4. Always Specify Charset for Text

```rust
"Content-Type: text/html; charset=utf-8"
"Content-Type: application/json; charset=utf-8"
```

### 5. Percent-Encode URLs Properly

Use a library—don't roll your own:

```rust
use url::form_urlencoded;

let encoded = form_urlencoded::byte_serialize("hello world".as_bytes())
    .collect::<String>();
// Result: "hello+world" or "hello%20world"
```

### 6. Test with Binary Tools

Use `xxd`, `hexdump`, or Wireshark to see actual bytes:

```bash
echo -ne "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n" | xxd
```

Output:
```
00000000: 4745 5420 2f20 4854 5450 2f31 2e31 0d0a  GET / HTTP/1.1..
00000010: 486f 7374 3a20 6578 616d 706c 652e 636f  Host: example.co
00000020: 6d0d 0a0d 0a                             m....
```

See those `0d0a` sequences? That's CRLF. If you see only `0a`, your line endings are wrong.

## The Devil Is in the Details

Line endings and encoding are the unsexy parts of HTTP. They're easy to get wrong and hard to debug. But getting them right is essential:

- Wrong line endings → 400 Bad Request or mysterious hangs
- Missing charset → garbled text and user complaints
- Wrong percent encoding → 404 errors or security issues
- Mixed encodings → data corruption

The good news? Once you implement these correctly, they work reliably. The bad news? You have to get them right the first time.

## Real-World Example

A complete, correctly-formatted HTTP request with all encoding details correct:

```
GET /search?q=hello%20world&city=S%C3%A3o%20Paulo HTTP/1.1\r\n
Host: api.example.com\r\n
User-Agent: MyClient/1.0\r\n
Accept: application/json\r\n
Accept-Charset: utf-8\r\n
\r\n
```

And a response:

```
HTTP/1.1 200 OK\r\n
Content-Type: application/json; charset=utf-8\r\n
Content-Length: 67\r\n
\r\n
{"results":["Hello World in São Paulo"],"count":1,"query":"hello world"}
```

Every line ends with `\r\n`. The headers are ASCII. The body is UTF-8 (specified by charset). The URL parameters are percent-encoded. Everything is precise.

## Chapter 5 Complete

We've now covered every aspect of HTTP message format:

1. **Overall structure** - The anatomy of requests and responses
2. **Request line** - Methods, paths, and versions
3. **Status line** - HTTP version, status codes, and reason phrases
4. **Headers** - Metadata that makes HTTP flexible
5. **Message body** - The actual payload
6. **Line endings and encoding** - The byte-level details that make it all work

You now understand not just what HTTP messages look like, but *why* they look that way, and exactly how they're encoded on the wire.

In the next chapter, we'll dive deep into **HTTP methods**—GET, POST, PUT, DELETE, and more. We'll explore what each method means, when to use it, and the semantics that make HTTP a truly RESTful protocol.

The foundation is solid. Now let's build on it.
