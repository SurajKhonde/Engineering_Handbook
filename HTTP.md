## HTTP AND REST API Best Practices 
### **1. What is HTTP?**
HTTP = HyperText Transfer Protocol
It’s a set of rules for how clients (like browsers, apps, or other servers) and servers talk to each other over the internet.
It runs on top of **TCP/IP** (the transport layer that actually moves data between machines).
Think of HTTP like a language that both the client and server agree to use.
#### HTTP solved this by:
✅ Standardizing how to request things (GET, POST, etc.).
✅ Standardizing how to respond (status codes, headers, body).
✅ Making the Web universal — any browser can talk to any server if both understand HTTP.

#### What an HTTP method is

An HTTP method tells the server what action the client wants to perform on a resource (the URL). Examples: **GET**, **POST**, **PUT**, **PATCH**, **DELETE**, **HEAD**, **OPTIONS**.
- Two important properties to keep in mind:
Safe: method does not change server state (e.g., GET, HEAD, OPTIONS are safe).
**Idempotent**: performing the exact same request multiple times has the same effect as doing it once (e.g., GET, PUT, DELETE are idempotent; 
POST is not; 
PATCH usually isn’t).

2) The common methods (what they mean + when to use)
**GET** — read data (safe + idempotent)
- Use to fetch a resource or list.
- Should not change server state.
Example:

```js
GET /users/42

// Curl:
curl -X GET http://localhost:3000/users/42
```
**POST** — create or trigger (not idempotent)

- Use to create resources (or trigger server-side actions).
- Not idempotent: calling it twice may create two resources.
- Should return 201 Created when a new resource is created (include Location header).

Example (create user):
```js
curl -X POST -H "Content-Type: application/json" -d '{"name":"Suraj"}' http://localhost:3000/users
```

- Typical Express route:

```js
app.post('/users', (req, res) => { /* create and return new user */ });
```

**PUT** — replace / upsert (idempotent)
upsert = update + insert.
In the context of PUT:
- Normally, PUT /resource/:id means replace the entire resource at that ID.
- But what if that resource (say, id = 42) doesn’t exist yet?
  - Some APIs will return 404 Not Found.
  - Others allow upsert → they will   create a new resource with that ID instead.
- Use to replace the entire resource at a known URL. If the resource doesn’t exist, many APIs allow PUT to create it (upsert).
- Idempotent: repeating the same PUT yields same result.

Example (replace user with id 42):
```js
curl -X PUT -H "Content-Type: application/json" -d '{"name":"New","age":30}' http://localhost:3000/users/42
```
- Express:

```js
app.put('/users/:id', (req, res) => { /* replace whole user */ });
```

**PATCH**  — partial update (not necessarily idempotent)
- Use to modify part of a resource (only changed fields).
- Not guaranteed idempotent (depends on operation), but often safe to treat as partial update.
Two common patch formats: application/merge-patch+json and application/json-patch+json (JSON Patch).

Example (change only name):

curl -X PATCH -H "Content-Type: application/json" -d '{"name":"Suraj Updated"}' http://localhost:3000/users/42


Express:

app.patch('/users/:id', (req, res) => { /* update only provided fields */ });


**When to prefer PUT vs PATCH**
- If client has complete new representation → use PUT.
- If client updates only some fields → use PATCH.
DELETE — remove resource (idempotent)

Remove the resource at the URL.

Idempotent: deleting an already-deleted resource usually returns 204 or 404 but has no further effect.

Example:

curl -X DELETE http://localhost:3000/users/42

HEAD — same as GET but no body (safe + idempotent)

Fetches headers only (useful for checking existence, content-length, caching).

Example: HEAD /users/42

- **OPTIONS** — describe supported actions / CORS preflight (safe + idempotent)

Returns allowed methods for a resource (used for CORS preflight).
Example: browser sends OPTIONS during cross-origin requests.

3) Status codes commonly used with these methods
- **POST** (create) → 201 Created (Location header), or 200/202 for other actions.
- **GET** → 200 OK, 404 Not Found
- **PUT/PATCH** → 200 OK (returns resource) or 204 No Content (no body) or 201 Created if created.

- **DELETE** → 204 No Content (successful, no body) or 404 if not found.
- Bad request / validation → 400
- Unauthorized → 401 / 403
- Conflict / duplicate → 409
- Server error → 500

### What is an HTTP header?
- Headers are key-value pairs sent along with the HTTP request or response.
- They carry metadata about the request/response, such as type of content, authentication, caching, etc.
- Think of them as the “envelope info” about the message.
#### 2️⃣ Why headers are needed
Headers solve these problems:
Problem	Header                       Solution
- What type of data is being sent  :--Content-Type
- How to authorize the request	 :--Authorization
- How long to cache the response	 :--Cache-Control
- Which browser/client made the request :--User-Agent
- Tell server/client what to accept :-	Accept / Accept-Language
- Cookies / session tracking	:- Cookie / Set-Cookie
So basically, headers help client and server communicate additional info without polluting the body.

### 3️⃣ Common request headers
  **Header	Use**
- Authorization	Pass credentials / token (e.g., Bearer <token>)
- Content-Type	Tell server the type of body being sent (e.g., application/json)
- Accept	Tell server what content types client can handle
- User-Agent	Browser or app info
- Cookie	Session tracking
- Host	Server domain

### 4️⃣ Setting headers
Client-side (browser / fetch / axios)
```js
fetch("http://localhost:3000/users", {
  method: "GET",
  headers: {
    "Authorization": "Bearer abc123",
    "Accept": "application/json"
  }
});

```
Axios example:
```js
import axios from "axios";

axios.get("/users", {
  headers: {
    Authorization: "Bearer abc123",
    "Content-Type": "application/json"
  }
});
```
### Server-side (Express) — reading headers
```js
import express, { Request, Response } from "express";

const app = express();

app.get("/users", (req: Request, res: Response) => {
  const authHeader = req.headers["authorization"]; // all headers lowercase
  const contentType = req.headers["content-type"];
  console.log("Auth:", authHeader);
  console.log("Content-Type:", contentType);
  res.json({ message: "Headers received" });
});

```
**Note: In Express, req.headers is an object with lowercase keys.**
#### Server-side — sending headers
 ```js
 app.get("/users", (req, res) => {
  res.setHeader("Content-Type", "application/json");
  res.setHeader("X-Custom-Header", "my-value");
  res.json({ success: true });
});
 ```

 ### Header
- Headers = metadata, body = actual data.
- Use headers for auth, content-type, caching, cookies, etc.
- Client sets headers; server reads or sets headers in response.
- They are optional but critical for secure & correct communication.

## HTTP Request
The client sends a request. It has 3 main parts:
```bash
METHOD PATH VERSION
//expamle
GET /users/42 HTTP/1.1 
```
- Method: what action you want (GET, POST, PUT, PATCH, DELETE, etc.)
- Path: resource /users/42
- Version: HTTP/1.1, HTTP/2, HTTP/3
**2. Headers**
Metadata about the request.
Examples:
- Authorization: Bearer <token> → auth info
- Content-Type: application/json → body format
- Accept: application/json → response format client can handle

**3. Body (optional)**
- Data sent to the server (for POST/PUT/PATCH).
- Format usually defined by Content-Type (JSON, form, XML).
### HTTP Response
- Server replies with:
**1. Status line**
- HTTP/1.1 200 OK
- 200 OK → success
- 404 Not Found → resource missing
- 500 Internal Server Error → server failed

**2. Headers**
Metadata about the response.

Examples:
- Content-Type: application/json → body type
- Set-Cookie: session=abc123 → set session cookie
- Cache-Control: max-age=3600 → caching rules

**3. Body**
- Actual data (JSON, HTML, images, etc.)
- Optional for some responses (like 204 No Content)

5️⃣ URL Components

Example URL:

```JS
https://example.com:8080/users/42?role=admin#section
```

- Protocol: https://
- Host: example.com
- Port: 8080
- Path: /users/42

**Query params**: ?role=admin → req.query

Fragment: #section → browser only, not sent to server

**Parameters**
- Route params (req.params) → from URL path /users/:id
- Query params (req.query) → after ?, optional filters /users?role=admin
- Body (req.body) → POST/PUT/PATCH data

#### Headers
- Already explained in detail, but key takeaway:
- Client → server: Authorization, Content-Type, Accept, Cookie
- Server → client: Content-Type, Set-Cookie, Cache-Control, ETag

Headers = metadata about the request/response

**Some of the most commonly used request headers are:**


#### Status Codes
- 1xx → Informational
- 2xx → Success (200 OK, 201 Created, 204 No Content)
- 3xx → Redirects (301 Moved Permanently, 302 Found)
- 4xx → Client errors (400 Bad Request, 401 Unauthorized, 404 Not Found)
- 5xx → Server errors (500 Internal Server Error, 503 Service Unavailable)

#### Caching & Conditional Requests
ETag, If-None-Match, Cache-Control → control browser/server caching
Helps reduce network load and improve performance

#### Cookies & Sessions
- Cookies: small pieces of data stored in browser → sent in Cookie header
- Session: server stores user state, linked to cookie

Example: login persistence

#### Security-related headers
- Authorization → API tokens
- X-Frame-Options, Content-Security-Policy → protect against XSS/Clickjacking
- Strict-Transport-Security → enforce HTTPS
#####  Practical Tips for Node.js

### Map HTTP concepts to Express:
- req.params → route params
- req.query → query string
- req.body → POST/PUT/PATCH data
- req.headers → request headers
- res.setHeader(...) → send headers
- res.status(...) → send status codes
Always return meaningful status codes.
Keep API consistent (method, URL, response shape).
Use headers for metadata, body for actual data.
Learn how CORS works for cross-origin requests.

## API 
An API, or application programming interface, is a set of rules or protocols that enables software applications to communicate with each other to exchange data, features and functionality. 
It’s useful to think about API communication in terms of a **request** and **response** between a client and server.
 **types of APIs**, 
 - REST
 - SOAP
 - GraphQL
### HTTP response status codes
- Informational responses (100 – 199)
- Successful responses (200 – 299)
- Redirection messages (300 – 399)
- Client error responses (400 – 499)
- Server error responses (500 – 599)

### Request headers (client → server)

| Header | Purpose | Typical Example | Quick Notes |
|---|---|---|---|
| **Accept** | Formats the client can accept | `Accept: application/json, text/html` | Server negotiates best match. |
| **User-Agent** | Identifies caller (browser/app) | `User-Agent: Mozilla/5.0 … Chrome/124` | For tailoring/logging; not for auth. |
| **Authorization** | Credentials for protected routes | `Authorization: Bearer <JWT>` | Send over HTTPS; prefer short-lived tokens. |
| **Content-Type** | Format of **request body** | `Content-Type: application/json` | Required when sending a body. |
| **Cookie** | Sends stored cookies | `Cookie: sid=abc; locale=en` | Enables sessions/personalization. |

### Response headers (server → client)

| Header | Purpose | Typical Example | Quick Notes |
|---|---|---|---|
| **Content-Type** | Format of **response body** | `Content-Type: application/json; charset=utf-8` | Must match payload. |
| **Cache-Control** | Caching policy | `Cache-Control: public, max-age=3600` | Use `no-store`/`no-cache`/`must-revalidate` as needed. |
| **Server** | Server software info | `Server: Apache/2.4.10 (Unix)` | Informational; often masked. |
| **Set-Cookie** | Store cookie on client | `Set-Cookie: sid=abc; Path=/; HttpOnly; Secure` | Add `Secure`, `HttpOnly`, `SameSite`. |
| **Content-Length** | Response size (bytes) | `Content-Length: 842` | Omit when chunked. |
0
#### Mnemonics
- **Request:** *A U ACT Cool* → Accept, User-Agent, Authorization, Content-Type, Cookie  
- **Response:** *C C S S C* → Content-Type, Cache-Control, Server, Set-Cookie, Content-Length
---

A “cookie” is always an HTTP cookie. The difference is how you use it:
Web (site) cookies → keep a user logged in on a website and remember preferences.
API cookies → carry auth/session for API requests (often from a browser SPA) instead of using an Authorization header.