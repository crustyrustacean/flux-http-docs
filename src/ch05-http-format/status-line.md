# Status Line

The **status line** is the first line of every HTTP response. It's the server's answer to the client's request line, telling the client what happened with their request.

## Anatomy of the Status Line

Every status line has exactly three parts, separated by single spaces:

```
HTTP/VERSION STATUS_CODE REASON_PHRASE
```

For example:
```
HTTP/1.1 200 OK
```

Let's break down each component.

## The HTTP Version

The first part of the status line is the **HTTP version** that the server is using to respond:

```
HTTP/1.1 200 OK
^^^^^^^^
This is the version
```

This tells the client which version of HTTP the server is speaking. It's formatted exactly like in the request line:
- `HTTP/` prefix
- Major version
- `.` (period)
- Minor version

**Examples:**
```
HTTP/1.0 200 OK
HTTP/1.1 404 Not Found
HTTP/2.0 500 Internal Server Error
```

### Version Negotiation

The version in the response doesn't have to match the request. The server can downgrade if it doesn't support the client's requested version:

```
Client Request:  GET /page HTTP/1.1
Server Response: HTTP/1.0 200 OK
```

Here, the client requested HTTP/1.1, but the server only supports HTTP/1.0, so it responds with the older version. The client should adapt to the server's version.

In practice, HTTP/1.1 is nearly universal, so you'll almost always see `HTTP/1.1` in both requests and responses.

## The Status Code

The second part is the **status code**—a three-digit number that tells the client what happened:

```
HTTP/1.1 200 OK
         ^^^
         This is the status code
```

Status codes are grouped into five categories by their first digit:

- **1xx (Informational)**: Request received, continuing process
  - `100 Continue`, `101 Switching Protocols`

- **2xx (Success)**: Request was successfully received, understood, and accepted
  - `200 OK`, `201 Created`, `204 No Content`

- **3xx (Redirection)**: Further action needs to be taken to complete the request
  - `301 Moved Permanently`, `302 Found`, `304 Not Modified`

- **4xx (Client Error)**: Request contains bad syntax or cannot be fulfilled
  - `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

- **5xx (Server Error)**: Server failed to fulfill an apparently valid request
  - `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`

The status code is the programmatic result of the request. Clients should make decisions based on this number, not the human-readable reason phrase.

We'll explore status codes in detail in Chapter 7. For now, remember:
- Always exactly three digits
- First digit indicates the category
- Most important part of the status line for automated processing

## The Reason Phrase

The third part is the **reason phrase**—a human-readable description of the status code:

```
HTTP/1.1 200 OK
             ^^
             This is the reason phrase
```

The reason phrase is meant for humans, not machines. It's a short text description of what the status code means.

**Common examples:**
```
HTTP/1.1 200 OK
HTTP/1.1 404 Not Found
HTTP/1.1 500 Internal Server Error
HTTP/1.1 201 Created
HTTP/1.1 403 Forbidden
```

### Reason Phrases Are Optional (Sort of)

Technically, the HTTP specification defines standard reason phrases for each status code:
- `200` → "OK"
- `404` → "Not Found"
- `500` → "Internal Server Error"

But servers can use custom reason phrases:

```
HTTP/1.1 200 Everything is Awesome
HTTP/1.1 404 Could Not Find That Thing
HTTP/1.1 500 Oops, Something Broke
```

**However**, clients should **never** parse or make decisions based on reason phrases. They're purely informational. Always use the status code for logic.

In HTTP/2, reason phrases were removed entirely—only the status code remains. This reinforces that reason phrases are just decoration.

### Reason Phrases Can Have Spaces

Unlike methods and paths, reason phrases can contain spaces:

```
HTTP/1.1 404 Not Found
             ^^^^^^^^^
             Multiple words are fine
```

The reason phrase extends from after the status code to the end of the line (`\r\n`).

## Format Rules

The status line follows strict formatting rules:

### Single Spaces Between Components

The version and status code are separated by exactly one space. The status code and reason phrase are also separated by exactly one space:

```
HTTP/1.1 200 OK
        ^   ^
        Single spaces
```

### Line Ending

Like all HTTP lines, the status line ends with `\r\n`:

```
HTTP/1.1 200 OK\r\n
```

### No Extra Whitespace

No trailing spaces after the reason phrase:

```
HTTP/1.1 200 OK\r\n         (Correct)
HTTP/1.1 200 OK  \r\n       (Incorrect - extra spaces)
```

## Real-World Examples

Let's see status lines you'll encounter in practice:

### Successful GET Request
```
HTTP/1.1 200 OK
```
"I successfully retrieved what you asked for"

### Successful POST (Resource Created)
```
HTTP/1.1 201 Created
```
"I successfully created the resource you submitted"

### Successful DELETE (No Content to Return)
```
HTTP/1.1 204 No Content
```
"I successfully deleted it, and there's nothing to send back"

### Permanent Redirect
```
HTTP/1.1 301 Moved Permanently
```
"That resource has moved to a new location forever"

### Temporary Redirect
```
HTTP/1.1 302 Found
```
"That resource is temporarily at a different location"

### Not Modified (Caching)
```
HTTP/1.1 304 Not Modified
```
"You already have the current version, use your cached copy"

### Bad Request
```
HTTP/1.1 400 Bad Request
```
"Your request was malformed or invalid"

### Unauthorized
```
HTTP/1.1 401 Unauthorized
```
"You need to authenticate to access this"

### Forbidden
```
HTTP/1.1 403 Forbidden
```
"You're authenticated, but you don't have permission"

### Not Found
```
HTTP/1.1 404 Not Found
```
"That resource doesn't exist"

### Method Not Allowed
```
HTTP/1.1 405 Method Not Allowed
```
"You can't use that HTTP method on this resource"

### Internal Server Error
```
HTTP/1.1 500 Internal Server Error
```
"Something went wrong on the server side"

### Service Unavailable
```
HTTP/1.1 503 Service Unavailable
```
"The server is temporarily unable to handle requests"

## Matching Status to Request

The status line is the server's direct response to the request line. Let's see some request-response pairs:

### Successful Retrieval
```
Request:  GET /users/123 HTTP/1.1
Response: HTTP/1.1 200 OK
```

### Resource Not Found
```
Request:  GET /users/999 HTTP/1.1
Response: HTTP/1.1 404 Not Found
```

### Successful Creation
```
Request:  POST /users HTTP/1.1
Response: HTTP/1.1 201 Created
```

### Invalid Method
```
Request:  DELETE / HTTP/1.1
Response: HTTP/1.1 405 Method Not Allowed
```

### Server Error
```
Request:  GET /api/data HTTP/1.1
Response: HTTP/1.1 500 Internal Server Error
```

## Parsing the Status Line

When building an HTTP client or debugging responses, you'll need to parse status lines. Here's the conceptual approach:

```rust
// Given: "HTTP/1.1 200 OK\r\n"

// 1. Split by the first \r\n to isolate the status line
let status_line = "HTTP/1.1 200 OK";

// 2. Split by spaces, but only the first two spaces
//    (reason phrase might contain spaces)
let parts: Vec<&str> = status_line.splitn(3, ' ').collect();

// 3. Extract components
let version = parts[0];      // "HTTP/1.1"
let code = parts[1];         // "200"
let reason = parts[2];       // "OK"

// 4. Validate
assert_eq!(parts.len(), 3, "Status line must have 3 parts");
assert!(version.starts_with("HTTP/"), "Invalid HTTP version");
assert_eq!(code.len(), 3, "Status code must be 3 digits");
assert!(code.chars().all(|c| c.is_numeric()), "Status code must be numeric");

// 5. Parse status code as integer
let status_code: u16 = code.parse().expect("Invalid status code");
```

Note that we use `splitn(3, ' ')` to split on only the first two spaces. This is important because the reason phrase can contain spaces:

```
HTTP/1.1 404 Not Found
         ^^^^^^^^^^^^^^
         Split only here and here, not in "Not Found"
```

## Edge Cases and Validation

When parsing status lines, watch out for:

**Invalid status codes:**
```
HTTP/1.1 2000 OK        (Too many digits)
HTTP/1.1 20 OK          (Too few digits)
HTTP/1.1 ABC OK         (Not numeric)
HTTP/1.1 600 OK         (Outside valid range)
```

**Missing components:**
```
HTTP/1.1 200            (Missing reason phrase - technically allowed)
HTTP/1.1 OK             (Missing status code - invalid)
200 OK                  (Missing version - invalid)
```

**Wrong version format:**
```
http/1.1 200 OK         (Wrong case)
HTTP/1 200 OK           (Missing minor version)
HTTP 1.1 200 OK         (Space instead of /)
```

**Extra spaces:**
```
HTTP/1.1  200 OK        (Double space - invalid)
HTTP/1.1 200  OK        (Double space - technically invalid)
```

A robust HTTP client should:
- Validate the version format
- Ensure the status code is exactly 3 digits and numeric
- Handle missing reason phrases gracefully (they're optional in practice)
- Return errors for malformed status lines

## Custom Reason Phrases

While standard reason phrases exist, servers can customize them:

**Standard:**
```
HTTP/1.1 418 I'm a teapot
```

**Custom:**
```
HTTP/1.1 418 Short and stout
```

Status code 418 is a fun one—it was defined in an April Fools' RFC about the Hyper Text Coffee Pot Control Protocol. Some servers still implement it!

**Important:** Never rely on reason phrases in your code. They're for humans debugging with tools like `curl`, not for programmatic logic. Always use the status code.

## The Status Line Tells the Story

The status line is the server's verdict on the client's request. It answers the essential question: **"What happened?"**

- **2xx**: "Success! I did what you asked"
- **3xx**: "The resource is elsewhere, go there"
- **4xx**: "You made a mistake in your request"
- **5xx**: "I made a mistake trying to fulfill your request"

Every status line falls into one of these categories, and the three-digit code tells you exactly what happened.

## Comparing Request and Status Lines

Let's see how they correspond:

| Request Line | Status Line |
|--------------|-------------|
| `GET /users HTTP/1.1` | `HTTP/1.1 200 OK` |
| Method → What you want | Status Code → What happened |
| Path → Which resource | Reason Phrase → Why |
| Version → How to talk | Version → How server responds |

The request line asks a question. The status line provides the answer.

## Looking Ahead

Now we understand both:
- **Request lines**: What clients ask for
- **Status lines**: How servers respond

Next, we'll explore **headers**—the metadata that accompanies both requests and responses. Headers provide crucial context: authentication, content type, caching instructions, cookies, and much more.

Headers are where HTTP's real flexibility shines.
