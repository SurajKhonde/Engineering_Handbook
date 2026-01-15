# What are APIs
APIs, or Application Programming Interfaces, provide a manner in which software applications communicate with each other. They abstract the complexity of applications to allow developers to use only the essentials of the software they are working with. They define the methods and data formats an application should use in order to perform tasks, like sending, retrieving, or modifying data. Understanding APIs is integral to mastering modern software development, primarily because they allow applications to exchange data and functionality with ease, thus enabling integration and convergence of technological services. Therefore, a solid understanding of what APIs are forms the basic cornerstone of API design.

## HTTP in API Design

[HTTP] HTTP

CORS = a browser security rule.
It controls whether a frontend website (origin) can call an API hosted on a different origin.

Why CORS is needed

Browsers enforce Same-Origin Policy to stop malicious sites from reading your data using your logged-in cookies/tokens.

Example threat:
You’re logged into bank.com. You open evil.com. Without CORS rules, evil.com could call https://bank.com/api/balance and read the response.

So CORS protects users in the browser.

What exactly is “Origin”?

Origin = protocol + domain + port
http://localhost:3000 and http://localhost:4000 are different origins.

How CORS works (what happens)

When browser calls API from different origin:

Simple request

Browser sends request, server must include:

Access-Control-Allow-Origin: <allowed-origin>

Non-simple request (preflight)

Browser sends OPTIONS first (preflight) asking “allowed?”
Server must respond with headers like:

Access-Control-Allow-Origin

Access-Control-Allow-Methods

Access-Control-Allow-Headers
Then actual request happens.

Node.js (Express) CORS 해결 (with code)
1) Allow all (public API) — *

⚠️ Works for APIs that don’t use cookies.

import express from "express";
import cors from "cors";

const app = express();

app.use(cors({ origin: "*" })); // allow all origins

app.get("/api/health", (req, res) => res.json({ ok: true }));

app.listen(4000, () => console.log("Server running"));

2) Allow only specific domains (recommended)
import express from "express";
import cors from "cors";

const app = express();

const allowedOrigins = [
  "http://localhost:3000",
  "https://myfrontend.com",
];

app.use(
  cors({
    origin: (origin, cb) => {
      // allow server-to-server / Postman (no Origin header)
      if (!origin) return cb(null, true);

      if (allowedOrigins.includes(origin)) return cb(null, true);
      return cb(new Error("Not allowed by CORS"));
    },
  })
);

app.get("/api/data", (req, res) => res.json({ data: "ok" }));

app.listen(4000);

3) Allow only specific IPs? (important truth)

CORS does NOT whitelist IP addresses reliably, because CORS checks the Origin header, not the client IP.

If you truly want “only this IP can call my API”, do IP allowlisting on server (or load balancer / firewall), like this:

import express from "express";

const app = express();

const allowIPs = new Set(["203.0.113.10", "127.0.0.1"]);

app.set("trust proxy", true); // if behind proxy (nginx/load balancer)

app.use((req, res, next) => {
  const ip = req.ip; // Express resolved client IP (with trust proxy)
  if (!allowIPs.has(ip)) return res.status(403).json({ error: "IP blocked" });
  next();
});

app.get("/api/private", (req, res) => res.json({ ok: true }));

app.listen(4000);


✅ Best practice: CORS for browsers + IP allowlist for network security.

4) Cookies / sessions case (needs extra settings)

If frontend sends cookies (JWT in cookie / session), you must do:

origin cannot be *

credentials: true

import express from "express";
import cors from "cors";

const app = express();

app.use(
  cors({
    origin: "http://localhost:3000",
    credentials: true,
  })
);

app.get("/api/me", (req, res) => res.json({ user: "suraj" }));

app.listen(4000);

One-liner interview wrap-up

CORS is a browser-side security gate controlled by server headers.
We fix it in Express using cors() middleware with allowed origins, and if we need IP restriction, we add IP allowlisting separately.

1) What is a Path Parameter?

Path param = part of the URL path that identifies a specific resource.
It’s usually required.

Example:

GET /users/123

123 is the path param (user id)

Think: /users/:id → “Give me THIS user”.

2) What is a Query Parameter?

Query param = extra options after ? used for filtering, sorting, pagination, searching, etc.
It’s usually optional.

Example:

GET /users?role=admin&sort=createdAt&page=2

role, sort, page are query params

Think: “Give me users, but with conditions”.

3) Main difference (interview answer)
Use Path params when:

You are targeting one specific resource

Resource identity is part of the URL

✅ /users/123
✅ /orders/987/items

Use Query params when:

You are querying a collection with filters/options

Sorting, pagination, search, optional flags

✅ /users?role=admin
✅ /users?page=2&limit=20
✅ /products?minPrice=100&maxPrice=500

Rule of thumb:

Path = “who/which resource”

Query = “how to filter/view the list”

4) “Can I fetch user data without query param?”

Yes.

GET /users → returns all users (usually paginated)

GET /users/123 → returns one user

GET /users?name=suraj → returns filtered users

So query param is not required for fetching data. It’s for filtering/options.

Express.js code (Path + Query)
A) Path param example: /users/:id
import express from "express";

const app = express();

app.get("/users/:id", (req, res) => {
  const { id } = req.params; // path params
  res.json({ message: "User by id", id });
});

app.listen(4000, () => console.log("Server running on 4000"));


Call:

GET http://localhost:4000/users/123

Output:

{ "message": "User by id", "id": "123" }

B) Query param example: /users?role=admin&page=2
import express from "express";

const app = express();

app.get("/users", (req, res) => {
  const { role, page = "1", limit = "10" } = req.query; // query params
  res.json({
    message: "Users list",
    filters: { role },
    pagination: { page: Number(page), limit: Number(limit) },
  });
});

app.listen(4000, () => console.log("Server running on 4000"));


Call:

GET http://localhost:4000/users?role=admin&page=2&limit=5

C) Combine both: /users/:id/posts?sort=desc
import express from "express";

const app = express();

app.get("/users/:id/posts", (req, res) => {
  const { id } = req.params;        // path param
  const { sort = "asc" } = req.query; // query param
  res.json({ message: "User posts", userId: id, sort });
});

app.listen(4000);

Common real-world API patterns (best practice)
1) Single resource

GET /users/:id

PUT /users/:id

DELETE /users/:id

2) List + filter

GET /users?page=1&limit=20

GET /users?role=admin&status=active

3) Nested resources

GET /users/:id/orders

GET /orders/:orderId/items

One super important tip (security)

Never trust params directly.
Validate id (numeric/uuid), validate query values.

Example quick validation:

app.get("/users/:id", (req, res) => {
  const { id } = req.params;
  if (!/^\d+$/.test(id)) return res.status(400).json({ error: "Invalid id" });
  res.json({ id });
});

1) OSI 7 layers (what they do) + where your API lives

OSI layers (top → bottom):

Application (L7): HTTP, HTTPS, WebSocket, DNS (protocols apps use)

Presentation (L6): encoding/format, encryption concepts (TLS often discussed here)

Session (L5): connection/session management (often merged in real life)

Transport (L4): TCP / UDP, ports, reliability

Network (L3): IP, routing between networks

Data Link (L2): MAC address, switching within local network

Physical (L1): wires, Wi-Fi radio, signals

Reality for developers

Your Express API is at L7 (HTTP/HTTPS).

TCP/UDP is L4.

IP routing is L3.

2) When user sends request → how it “flows” (simple story)

Let’s say browser calls:
https://api.myapp.com/users/123

Step-by-step (what happens)

DNS lookup (L7)
Browser asks “what IP is api.myapp.com?” → gets 203.x.x.x.

TCP connection (L4) (if HTTPS over TCP)
Browser connects to server IP on port 443 using TCP 3-way handshake.

TLS handshake (HTTPS security)
Certificates, encryption keys, then encrypted connection starts.

HTTP request (L7)
GET /users/123 is created by browser/app.

Encapsulation (packing down layers)

L7 HTTP message goes into

L4 TCP segments (with ports)

L3 IP packets (with src/dst IP)

L2 frames (with MAC in local network)

L1 signals over wire/Wi-Fi

On server side: unpacking (reverse)

Network card receives (L1/L2)

OS processes IP routing (L3)

OS delivers to TCP port 443 (L4)

TLS decrypts

HTTP server (Node/Express) receives request (L7)

✅ Your Express app finally sees it at L7.

3) What is IP (for you)?

IP address = “where” the machine is on the network (routing address).
Example: 203.0.113.10

In Express: how to get client IP
import express from "express";
const app = express();

// IMPORTANT when behind a proxy/load balancer (nginx, AWS ALB, Cloudflare, etc.)
app.set("trust proxy", true);

app.get("/ip", (req, res) => {
  // req.ip is best when trust proxy is set correctly
  res.json({
    ip: req.ip,
    ips: req.ips, // array if multiple proxies
    forwardedFor: req.headers["x-forwarded-for"] || null,
  });
});

app.listen(4000);


Why trust proxy matters:
If your app is behind a proxy, without it you may only see the proxy IP (not real client).

4) What is TCP? Why we use it for APIs?

TCP (Transport layer) gives:

Reliable delivery (retries lost packets)

Ordered delivery (keeps correct sequence)

Congestion control (avoids network collapse)

Connection-based (handshake)

✅ Great for APIs because API calls need correct + complete data.

Important correction

“why TCP is better than HTTP”
This comparison is wrong because:

HTTP is an application protocol (L7)

TCP is a transport protocol (L4)

HTTP usually runs on top of TCP (HTTP/1.1, HTTP/2)
So TCP isn’t “better than HTTP” — it’s the layer underneath that HTTP uses.

5) What is UDP?

UDP (L4) is:

No connection

No guaranteed delivery

No ordering

Lower overhead → faster + lower latency

Used for:

Live video/audio, gaming, DNS

And modern web: HTTP/3 uses QUIC over UDP (gives speed + reliability in a different way)
DNS basics (how it works)

DNS = phonebook of the internet. It converts a domain name (like api.example.com) into an IP address (like 203.0.113.10).

When you type a URL, your device typically does this:

Browser cache check
“Do I already know the IP for this domain?”

OS cache check
Your operating system keeps its own DNS cache.

Hosts file check (local override)
If you’ve mapped a domain manually, it uses that.

Ask a DNS resolver (recursive resolver)
Usually your ISP resolver, Google (8.8.8.8), Cloudflare (1.1.1.1), or your company DNS.

If resolver doesn’t know, it does the DNS chain:

Root DNS: “Who knows about .com?”

TLD DNS (.com): “Who knows example.com?”

Authoritative DNS (for example.com): “Here’s the answer for api.example.com”
Then it returns A record (IPv4) or AAAA record (IPv6) (or follows CNAME to another name).

Is the IP cached by the browser?

✅ Yes — usually. And not only the browser.

DNS answers are cached at multiple levels:

Browser DNS cache

OS DNS cache

Router / local network cache (sometimes)

ISP / public DNS resolver cache

How long it stays cached = TTL (Time To Live) in the DNS record.
Example: TTL 300 seconds → cache for ~5 minutes, then re-check.

What REST is (1 line)

REST = resources + standard HTTP semantics + stateless server + cache + layered system + uniform interface.

REST principles

1) Client–Server separation

Frontend and backend are separate concerns.

Client handles UI/UX

Server handles data + business rules

Why it matters: you can change UI without rewriting backend, and scale them independently.

2) Statelessness (most important)

Every request must contain all info needed to process it.
Server does not remember client session state between requests.

✅ Good:

Client sends token each time: Authorization: Bearer <jwt>

Server can handle any request on any node (easy load balancing)

❌ Not REST-ish:

Server stores “where you are in workflow” in memory and expects next request to continue from that state.

Important nuance:
Stateless ≠ “no auth” and ≠ “no sessions ever”.
You can use sessions/cookies, but the server should treat each request independently (shared session store, not in-memory per node).

3) Cacheability

Responses should define if they can be cached, using HTTP caching headers.

Cache-Control, ETag, Last-Modified

Why it matters: huge performance boost, reduces server load.

Example patterns:

GET /products can be cached

POST /orders usually not cached

4) Uniform Interface (the core REST “feel”)

This is what makes REST predictable.

4.1 Resource identification via URI

Use nouns (resources), not verbs (actions).
✅ /users/123
✅ /orders/987/items
❌ /getUser ❌ /createOrder

4.2 Manipulation via representations

Client sends/receives representations like JSON.

POST /users with JSON body creates user

PUT /users/123 with JSON replaces user

4.3 Self-descriptive messages

Request/response should be understandable by itself:

correct method (GET/POST/PUT…)

headers (Content-Type: application/json)

status codes (200/201/404/400/401/409/500)

4.4 HATEOAS (often asked, rarely implemented)

Response includes links to next actions.

Response includes links to next action
{
  "id": "123",
  "name": "Suraj",
  "links": [
    { "rel": "self", "href": "/users/123" },
    { "rel": "orders", "href": "/users/123/orders" }
  ]
}


5) Layered System

Client doesn’t care if it’s talking to:

API gateway

load balancer

cache layer (CDN)

microservices behind it

Why it matters: you can add CDN, auth gateway, rate limiter without breaking clients.

6) Code-on-Demand (optional)

Server can send executable code to client (like JS).
Most APIs ignore this, but it’s part of REST constraints.

REST in practice: what interviewers expect you to apply
Resource + HTTP method mapping (CRUD)

GET /users → list

GET /users/:id → read one

POST /users → create

PUT /users/:id → replace

PATCH /users/:id → partial update

DELETE /users/:id → delete

Status codes (must know)

200 OK (success)

201 Created (created + often return resource)

204 No Content (success but no body, often delete)

400 Bad Request (validation)

401 Unauthorized (not logged in)

403 Forbidden (logged in but not allowed)

404 Not Found

409 Conflict (duplicate, version conflict)

429 Too Many Requests (rate limit)

500 / 503 (server/down)

Idempotency (very common question)

GET ✅

PUT ✅

DELETE ✅

POST ❌ (usually)

Versioning (common in real APIs)

URL: /v1/users

Header: Accept: application/vnd.myapp.v1+json


Quick “speakable” interview answer

“REST is an architectural style where we expose resources via URLs, use standard HTTP methods and status codes, keep the server stateless, support caching, and allow layered architecture like gateways/CDNs. Uniform interface makes APIs predictable; in strict REST, HATEOAS adds discoverable links, though many production APIs are REST-like.”



When do you need a new version?

Create a new version for breaking changes, like:

rename/remove fields

change response shape

change meaning of fields

change auth behavior

change URL semantics

Non-breaking changes (usually no new version):

add optional fields

add new endpoints

add new query params (optional)

performance improvements

1) URI Versioning (most common)

Example

/api/v1/users

/api/v2/users

✅ Pros

Super easy to understand and test (works in browser too)

Easy routing in gateways/load balancers/CDNs

Clear docs, clear separations

❌ Cons

“Version is part of resource URL” (some people call it not pure REST)

If you have many services, versions can spread everywhere

Best when: public APIs, mobile clients, many consumers, simple operations.