# Cheatsheet
# Chapter 5 Cheatsheet

## HTTP Message Structure

**Two Types of Messages:**
1. **Request** (client → server)
2. **Response** (server → client)

**Common Structure:**
```
[First Line: Request Line OR Status Line]
[Header 1: value]
[Header 2: value]
...
[Empty Line: \r\n]
[Optional Body]
```

---

## Request Line

**Format:**
```
METHOD /path/to/resource HTTP/VERSION
```

**Three Components:**

| Component | Example | Notes |
|-----------|---------|-------|
| **Method** | `GET`, `POST`, `DELETE` | What action to perform |
| **Request Target** | `/api/users?page=2` | Which resource (path + query) |
| **HTTP Version** | `HTTP/1.1` | Protocol version |

**Examples:**
```
GET /index.html HTTP/1.1
POST /api/login HTTP/1.1
DELETE /users/123 HTTP/1.1
GET /search?q=rust&page=2 HTTP/1.1
```

**Rules:**
- Components separated by single spaces
- Path must start with `/`
- Special characters must be percent-encoded
- Ends with `\r\n`

**Fragment Note:** `#fragment` never sent to server (client-only)

---

## Status Line

**Format:**
```
HTTP/VERSION STATUS_CODE REASON_PHRASE
```

**Three Components:**

| Component | Example | Notes |
|-----------|---------|-------|
| **HTTP Version** | `HTTP/1.1` | Protocol version |
| **Status Code** | `200`, `404`, `500` | Numeric result (3 digits) |
| **Reason Phrase** | `OK`, `Not Found` | Human-readable description |

**Examples:**
```
HTTP/1.1 200 OK
HTTP/1.1 404 Not Found
HTTP/1.1 500 Internal Server Error
HTTP/1.1 201 Created
```

**Status Code Categories:**

| Range | Category | Meaning |
|-------|----------|---------|
| **1xx** | Informational | Request received, continuing |
| **2xx** | Success | Request successful |
| **3xx** | Redirection | Further action needed |
| **4xx** | Client Error | Request has errors |
| **5xx** | Server Error | Server failed |

**Important:** Always use status code for logic, never the reason phrase!

---

## Headers

**Format:**
```
Header-Name: Header-Value\r\n
```

**Key Properties:**
- Header names are **case-insensitive** (but conventionally `Title-Case`)
- Header values are **case-sensitive** (usually)
- Optional space after colon (conventional)
- Multiple headers allowed
- Order doesn't matter (except Host is often first)

### Common Request Headers

| Header | Purpose | Example |
|--------|---------|---------|
| `Host` | Target server (REQUIRED in HTTP/1.1) | `Host: www.example.com` |
| `User-Agent` | Client identification | `User-Agent: Mozilla/5.0...` |
| `Accept` | Acceptable response types | `Accept: application/json` |
| `Content-Type` | Body type (in request) | `Content-Type: application/json` |
| `Content-Length` | Body size in bytes | `Content-Length: 1234` |
| `Authorization` | Authentication credentials | `Authorization: Bearer token123` |
| `Cookie` | Send cookies to server | `Cookie: session=abc123` |
| `Referer` | Previous page URL | `Referer: https://example.com/page1` |

### Common Response Headers

| Header | Purpose | Example |
|--------|---------|---------|
| `Content-Type` | Body type (in response) | `Content-Type: text/html; charset=utf-8` |
| `Content-Length` | Body size in bytes | `Content-Length: 2048` |
| `Set-Cookie` | Tell client to store cookie | `Set-Cookie: session=abc; HttpOnly` |
| `Location` | Redirect destination | `Location: https://example.com/new` |
| `Cache-Control` | Caching directives | `Cache-Control: max-age=3600` |
| `ETag` | Resource version identifier | `ETag: "abc123"` |
| `Server` | Server software | `Server: nginx/1.18.0` |
| `Date` | When response generated | `Date: Wed, 04 Oct 2025 15:30:00 GMT` |

### Accept Headers Family

```
Accept: text/html,application/json;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9,es;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Charset: utf-8, iso-8859-1;q=0.5
```

`q` values indicate preference (0.0 to 1.0, higher is better)

---

## Message Body

**When Present:**

| Requests | Responses |
|----------|-----------|
| POST (creating) | 200 OK (successful GET) |
| PUT (replacing) | 201 Created (with created resource) |
| PATCH (updating) | 400/500 (error details) |
| ~~GET~~ (discouraged) | |

**When Absent:**

| Requests | Responses |
|----------|-----------|
| GET | 204 No Content |
| DELETE | 304 Not Modified |
| HEAD | HEAD responses (never have body) |
| OPTIONS | 1xx Informational |

### Specifying Body Length

**Option 1: Content-Length**
```
Content-Length: 45

{"username":"alice","email":"a@example.com"}
```
Body is exactly 45 bytes.

**Option 2: Chunked Transfer Encoding**
```
Transfer-Encoding: chunked

7\r\n
Hello, \r\n
6\r\n
World!\r\n
0\r\n
\r\n
```
Size in hex, then data, repeat until size is 0.

**Never use both together!**

### Common Content Types

| Content-Type | Usage |
|--------------|-------|
| `application/json` | JSON data (APIs) |
| `application/x-www-form-urlencoded` | Form submissions |
| `multipart/form-data` | File uploads |
| `text/html` | HTML pages |
| `text/plain` | Plain text |
| `application/octet-stream` | Binary/unknown |
| `image/jpeg`, `image/png` | Images |
| `video/mp4` | Video |

**Always specify charset for text:**
```
Content-Type: text/html; charset=utf-8
Content-Type: application/json; charset=utf-8
```

---

## Line Endings and Encoding

### CRLF - The HTTP Line Ending

**Always use `\r\n` (CRLF):**
- `\r` = Carriage Return = byte `0x0D` (13)
- `\n` = Line Feed = byte `0x0A` (10)

```
GET / HTTP/1.1\r\n
Host: example.com\r\n
\r\n
```

**Empty line separator:** `\r\n\r\n`

**Platform Line Endings (HTTP IGNORES THESE):**
| Platform | Native | HTTP Uses |
|----------|--------|-----------|
| Windows | `\r\n` | `\r\n` ✓ |
| Unix/Linux | `\n` | `\r\n` (must convert!) |
| Mac OSX | `\n` | `\r\n` (must convert!) |

### Character Encoding

**Headers (Envelope):**
- Must be **ASCII only** (bytes 0-127)
- No Unicode, no multi-byte characters
- Use percent encoding for special characters

**Body:**
- Can be any encoding
- Specified by `charset` parameter
- UTF-8 recommended

```
HTTP/1.1 200 OK\r\n
Content-Type: text/html; charset=utf-8\r\n
\r\n
<!DOCTYPE html>
<html><title>Hello 世界</title></html>
```

### Percent Encoding (URL Encoding)

**How it works:**
1. Encode character as UTF-8 bytes
2. Represent each byte as `%XX` (hex)

**Examples:**
```
Space     → %20
é         → %C3%A9
?         → %3F
#         → %23
São Paulo → S%C3%A3o%20Paulo
```

**Reserved characters (must encode when literal):**
```
: / ? # [ ] @ ! $ & ' ( ) * + , ; =
```

**Unreserved (never need encoding):**
```
A-Z a-z 0-9 - _ . ~
```

**Special case - Spaces:**
- `%20` - Works everywhere
- `+` - Only in query string values

```
/search?q=hello+world     ✓ (+ in query value)
/search?q=hello%20world   ✓ (always works)
/hello+world              ✗ (+ in path is literal)
/hello%20world            ✓ (works in path)
```

---

## Complete Examples

### Request with Body

```
POST /api/users HTTP/1.1\r\n
Host: api.example.com\r\n
User-Agent: MyApp/1.0\r\n
Accept: application/json\r\n
Content-Type: application/json; charset=utf-8\r\n
Content-Length: 56\r\n
Authorization: Bearer abc123\r\n
\r\n
{"username":"alice","email":"alice@example.com","age":30}
```

### Response with Body

```
HTTP/1.1 200 OK\r\n
Date: Wed, 04 Oct 2025 16:00:00 GMT\r\n
Content-Type: application/json; charset=utf-8\r\n
Content-Length: 89\r\n
Cache-Control: no-cache\r\n
Set-Cookie: session=xyz789; HttpOnly; Secure\r\n
\r\n
{"id":42,"username":"alice","email":"alice@example.com","created_at":"2025-10-04T16:00:00Z"}
```

### GET Request (No Body)

```
GET /api/users?page=2&limit=20 HTTP/1.1\r\n
Host: api.example.com\r\n
Accept: application/json\r\n
Authorization: Bearer token123\r\n
\r\n
```

---

## Common Mistakes

❌ **Wrong line ending:**
```
GET / HTTP/1.1\n          (Only \n - WRONG)
```

✓ **Correct:**
```
GET / HTTP/1.1\r\n        (CRLF)
```

---

❌ **Missing empty line:**
```
POST /api HTTP/1.1\r\n
Content-Length: 5\r\n
Hello
```

✓ **Correct:**
```
POST /api HTTP/1.1\r\n
Content-Length: 5\r\n
\r\n
Hello
```

---

❌ **Non-ASCII in headers:**
```
User-Agent: MyApp™ 2.0
```

✓ **Correct:**
```
User-Agent: MyApp 2.0
```

---

❌ **Unencoded URL:**
```
GET /search?q=hello world HTTP/1.1
```

✓ **Correct:**
```
GET /search?q=hello%20world HTTP/1.1
```

---

❌ **Missing charset:**
```
Content-Type: text/html
```

✓ **Correct:**
```
Content-Type: text/html; charset=utf-8
```

---

## Parsing Strategy

**Reading a Message:**
1. Read first line → Determine if request or response
2. Read headers line by line until empty line
3. Check `Content-Length` or `Transfer-Encoding`
4. Read body (if present)

**Request Line Parsing:**
```rust
let parts: Vec<&str> = line.split(' ').collect();
let method = parts[0];      // "GET"
let target = parts[1];      // "/path"
let version = parts[2];     // "HTTP/1.1"
```

**Status Line Parsing:**
```rust
let parts: Vec<&str> = line.splitn(3, ' ').collect();
let version = parts[0];     // "HTTP/1.1"
let code = parts[1];        // "200"
let reason = parts[2];      // "OK" (may contain spaces)
```

**Header Parsing:**
```rust
if let Some(colon_pos) = line.find(':') {
    let name = &line[..colon_pos];
    let value = &line[colon_pos + 1..].trim();
    headers.insert(name.to_lowercase(), value.to_string());
}
```

---

## Debugging Tips

**View actual bytes:**
```bash
xxd file.txt              # Hexdump
tcpdump -i any -w cap.pcap port 8080  # Capture traffic
```

**Look for CRLF:**
```
0D 0A = \r\n ✓ Correct
0A    = \n   ✗ Wrong
0D    = \r   ✗ Wrong
```

**Test with netcat:**
```bash
printf "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n" | nc example.com 80
```

---

## Quick Reference

**Valid HTTP Message Checklist:**
- ✓ First line has correct format
- ✓ All lines end with `\r\n`
- ✓ Empty line (`\r\n\r\n`) separates headers from body
- ✓ Headers are ASCII
- ✓ Host header present (requests only)
- ✓ Content-Length or Transfer-Encoding for bodies
- ✓ Charset specified for text bodies
- ✓ URLs properly percent-encoded

**Remember:**
- CRLF (`\r\n`) for ALL line endings
- ASCII for headers, any encoding for body
- Percent-encode special characters in URLs
- Always specify charset for text
- Content-Length = size of body in bytes
- Empty line separates headers from body
