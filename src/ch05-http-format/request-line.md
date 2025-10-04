# Request Line

The **request line** is the first line of every HTTP request. It's the most important line in the message because it tells the server exactly what the client wants to do.

## Anatomy of the Request Line

Every request line has exactly three parts, separated by single spaces:

```
METHOD /path/to/resource HTTP/VERSION
```

For example:
```
GET /api/users HTTP/1.1
```

Let's break down each component.

## The HTTP Method

The first part is the **HTTP method** (also called the HTTP verb). It tells the server what action you want to perform.

```
GET /index.html HTTP/1.1
^^^
This is the method
```

Common methods include:
- **GET** - Retrieve a resource
- **POST** - Create a resource or submit data
- **PUT** - Update/replace a resource
- **PATCH** - Partially update a resource
- **DELETE** - Remove a resource
- **HEAD** - Get headers only (no body)
- **OPTIONS** - Get allowed methods for a resource

We'll explore methods in depth in Chapter 6. For now, just know that the method is always:
- Uppercase (by convention, though servers should accept any case)
- A single word
- The first thing in the request line

**Examples:**
```
GET /products HTTP/1.1
POST /api/login HTTP/1.1
DELETE /users/123 HTTP/1.1
```

## The Request Target (Path)

The second part is the **request target**—what resource you want to interact with. In most cases, this is a path on the server.

```
GET /api/users HTTP/1.1
    ^^^^^^^^^^
    This is the request target
```

The request target typically includes:

### The Path

The hierarchical path to the resource:
```
/
/index.html
/api/users
/products/123
/blog/2024/01/post-title
```

Paths always start with a forward slash `/`. The root path is just `/`.

### Query Parameters (Optional)

Query parameters come after a `?` and provide additional data:

```
GET /search?q=rust&category=books&sort=price HTTP/1.1
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
            Query parameters
```

Multiple parameters are separated by `&`:
- `q=rust` (search query)
- `category=books` (filter by category)
- `sort=price` (sort order)

Query parameters are commonly used with GET requests to filter, sort, or paginate results.

### Fragment (Not Sent)

URLs can have fragments (the part after `#`), but these are **never sent to the server**:

```
Browser URL: https://example.com/docs/api#authentication
Request Line: GET /docs/api HTTP/1.1
                           ^
                           No #authentication - browser keeps it
```

Fragments are purely client-side. They tell the browser where to scroll on the page, but the server never sees them.

## The HTTP Version

The third part specifies which version of HTTP you're using:

```
GET /api/users HTTP/1.1
               ^^^^^^^^
               This is the version
```

Common versions:
- **HTTP/1.0** - Legacy, rarely used
- **HTTP/1.1** - The standard, what we're focusing on
- **HTTP/2** - Binary protocol, backwards compatible
- **HTTP/3** - Over QUIC instead of TCP

For our purposes, we'll always use `HTTP/1.1`. The version must be written exactly as shown:
- `HTTP/` prefix
- Major version
- `.` (period)
- Minor version

**Valid:** `HTTP/1.1`
**Invalid:** `http/1.1` (wrong case), `HTTP/1` (missing minor version), `1.1` (missing HTTP prefix)

## Format Rules

The request line must follow strict formatting:

### Single Spaces

Components are separated by exactly **one space** (ASCII character 0x20):

```
GET /api/users HTTP/1.1
   ^          ^
   Single spaces here
```

Not tabs, not multiple spaces—exactly one space.

### No Trailing Whitespace

The request line ends with `\r\n`, with no spaces before it:

```
GET /api/users HTTP/1.1\r\n
                       ^
                       CRLF immediately after version
```

### Case Sensitivity

- **Methods**: Conventionally uppercase, but should be case-sensitive
- **Paths**: Case-sensitive (most servers treat `/Users` and `/users` as different)
- **Query parameters**: Case-sensitive (depends on server implementation)
- **HTTP version**: Case-sensitive (`HTTP` must be uppercase)

### Special Characters in Paths

Certain characters must be percent-encoded in the request target:

**Must be encoded:**
- Space → `%20` or `+` (in query parameters)
- Non-ASCII characters → UTF-8 bytes, percent-encoded
- Reserved characters when used literally: `? # [ ] @ ! $ & ' ( ) * + , ; =`

**Examples:**
```
/search?q=hello world    (Invalid - space not encoded)
/search?q=hello%20world  (Valid - space encoded as %20)
/search?q=hello+world    (Valid - space as + in query string)

/users/josé              (Invalid - non-ASCII)
/users/jos%C3%A9         (Valid - é encoded as UTF-8 bytes)

/path/with/question?     (Invalid - literal ? in path)
/path/with/question%3F   (Valid - ? encoded as %3F)
```

## Real-World Examples

Let's see how different types of requests look:

### Simple Resource Request
```
GET /index.html HTTP/1.1
```
"Give me the file at /index.html"

### API Endpoint
```
GET /api/v1/users/123 HTTP/1.1
```
"Give me user with ID 123 from API version 1"

### Search with Parameters
```
GET /search?q=rust+programming&page=2&limit=20 HTTP/1.1
```
"Search for 'rust programming', give me page 2, 20 results per page"

### Form Submission
```
POST /api/login HTTP/1.1
```
"I'm submitting data to the login endpoint"

### Resource Update
```
PUT /users/123 HTTP/1.1
```
"Replace the entire user 123 with new data"

### Partial Update
```
PATCH /users/123 HTTP/1.1
```
"Update just some fields of user 123"

### Resource Deletion
```
DELETE /users/123 HTTP/1.1
```
"Delete user 123"

### Root Path
```
GET / HTTP/1.1
```
"Give me the root resource" (typically the home page)

## Parsing the Request Line

When building an HTTP server, parsing the request line is the first step. Here's the conceptual approach:

```rust
// Given: "GET /api/users HTTP/1.1\r\n"

// 1. Split by the first \r\n to isolate the request line
let request_line = "GET /api/users HTTP/1.1";

// 2. Split by spaces
let parts: Vec<&str> = request_line.split(' ').collect();

// 3. Extract components
let method = parts[0];      // "GET"
let target = parts[1];      // "/api/users"
let version = parts[2];     // "HTTP/1.1"

// 4. Validate
assert_eq!(parts.len(), 3, "Request line must have exactly 3 parts");
assert!(version.starts_with("HTTP/"), "Invalid HTTP version");
```

We'll implement this properly in Chapter 8 when we build our parser.

## Edge Cases and Validation

When parsing request lines, watch out for:

**Missing components:**
```
GET /path           (Missing HTTP version - invalid)
/path HTTP/1.1      (Missing method - invalid)
GET HTTP/1.1        (Missing path - invalid)
```

**Extra spaces:**
```
GET  /path HTTP/1.1     (Double space - technically invalid)
GET /path  HTTP/1.1     (Double space - technically invalid)
```

**Invalid methods:**
```
get /path HTTP/1.1      (Lowercase - should reject or normalize)
G3T /path HTTP/1.1      (Typo - invalid method)
```

**Malformed paths:**
```
GET path HTTP/1.1       (No leading slash - invalid)
GET //path HTTP/1.1     (Double slash - technically valid but weird)
```

**Wrong HTTP version format:**
```
GET /path HTTP/2        (Missing minor version)
GET /path http/1.1      (Wrong case)
GET /path HTTP 1.1      (Space instead of /)
```

A robust HTTP server must validate the request line and return appropriate error responses (like `400 Bad Request`) when it's malformed.

## The Request Line Sets Everything in Motion

The request line is the single most important line in an HTTP request. It answers three critical questions:

1. **What do you want to do?** (Method)
2. **What resource?** (Request target)
3. **How should we communicate?** (HTTP version)

Every other part of the request—headers, body—provides additional context. But the request line is where it all begins.

## Looking Ahead

Now that we understand request lines, we'll look at their counterpart: **status lines** in HTTP responses. Status lines answer the client's request line, telling them what happened and whether their request succeeded.
