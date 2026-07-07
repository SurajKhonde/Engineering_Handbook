# Node.js & Backend Interview Prep
**Format:** Simple Answer → 🧠 Memory Trick → 🎤 Say This in Interview

---

## 1️⃣ Node.js Basics

### 1. What is the Event Loop?
It's a loop that keeps checking "any task done? any callback to run?" — lets Node do non-blocking I/O on a single thread.
🧠 **Trick:** Event Loop = "Waiter in a restaurant" — takes orders (requests), doesn't wait at your table, serves whoever's food is ready.
🎤 Say: "It's the mechanism that lets Node handle async operations on a single thread by offloading I/O and picking up callbacks when they're ready, cycling through phases like timers, I/O callbacks, and check."

### 2. What is Buffer in Node.js?
Temporary storage for raw binary data (before it's fully processed), used since JS has no native binary type.
🧠 **Trick:** Buffer = "Bucket for raw water (bytes) before it reaches the tap (your app)."
🎤 Say: "Buffer is Node's way of handling raw binary data outside the V8 heap — used heavily in streams and file/network I/O."

### 3. What are Streams?
A way to read/write data piece-by-piece (chunks) instead of loading it all into memory at once.
🧠 **Trick:** Streams = "Watching a movie online (chunks arrive) vs downloading the whole movie first."
🎤 Say: "Streams let you process data incrementally — Readable, Writable, Duplex, Transform — great for large files or network data, saves memory."

### 4. Difference between Buffer and Stream?
Buffer = a chunk of raw data sitting in memory. Stream = the pipeline that moves data (often as buffers) piece by piece.
🧠 **Trick:** Buffer = the water bucket. Stream = the pipe carrying the water.
🎤 Say: "Buffer is a data container; Stream is the flow mechanism that often uses buffers internally to move data efficiently."

### 5. What is the `fs` module?
Built-in module to read/write/delete files — has sync and async versions of every method.
🧠 **Trick:** `fs` = "File System remote control."
🎤 Say: "I use `fs` for file I/O, always preferring async (`fs.promises` or callbacks) over sync in production to avoid blocking the event loop."

### 6. What is the `path` module?
Helps build/manipulate file paths correctly across OS (Windows uses `\`, Linux uses `/`).
🧠 **Trick:** `path` = "GPS for files — works regardless of which country's roads (OS) you're on."
🎤 Say: "`path` gives OS-independent path handling — `path.join`, `path.resolve` — avoids hardcoding slashes."

### 7. What is the `os` module used for?
Gives info about the machine — CPU cores, memory, platform, uptime.
🧠 **Trick:** `os` = "Asking the computer 'how are you feeling today?'"
🎤 Say: "I use `os` mainly for things like determining CPU count to size a cluster or thread pool."

### 8. What is the `crypto` module?
Built-in module for hashing, encryption, decryption, generating secure random tokens.
🧠 **Trick:** `crypto` = "The vault and lock-maker of Node."
🎤 Say: "I use `crypto` for hashing (SHA-256), generating secure tokens, and HMAC signatures — e.g., verifying webhook payloads."

### 9. What is the `cluster` module?
Lets you spawn multiple Node processes (one per CPU core) to share the load, since Node is single-threaded.
🧠 **Trick:** `cluster` = "Franchise model — one shop (process) per branch (CPU core), same menu."
🎤 Say: "`cluster` forks multiple worker processes that share the same server port, letting me use all CPU cores for a single Node app."

### 10. What are Worker Threads?
A way to run actual multi-threaded JS code for CPU-heavy tasks, without blocking the main event loop.
🧠 **Trick:** Worker Threads = "Hiring a helper to do heavy lifting while you keep serving customers."
🎤 Say: "Worker Threads are for CPU-bound tasks (image processing, heavy computation) — unlike cluster (multi-process), these are true threads sharing memory via SharedArrayBuffer."

---

## 2️⃣ Asynchronous Programming

### 1. Sync vs Async functions?
Sync = blocks execution until done. Async = starts task, moves on, comes back when result is ready.
🧠 **Trick:** Sync = "Standing in line at the bank." Async = "Taking a token number and sitting down."
🎤 Say: "Sync blocks the thread; async lets Node continue processing other work while I/O completes in the background."

### 2. What is callback hell?
Nested callbacks inside callbacks — code becomes a pyramid, hard to read/maintain/debug.
🧠 **Trick:** Callback Hell = "Russian nesting dolls — you need to open 5 to reach the real one."
🎤 Say: "It's deeply nested async callbacks causing the 'pyramid of doom' — solved using Promises or async/await for flatter, readable code."

### 3. How do Promises solve callback hell?
They let you chain `.then()` instead of nesting, and centralize error handling with `.catch()`.
🧠 **Trick:** Promise = "IOU note — 'I promise to give you the result later,' chainable instead of nested."
🎤 Say: "Promises flatten async chains via `.then()`/`.catch()` and give a single point for error handling instead of scattered callbacks."

### 4. What is async/await?
Syntax sugar over Promises — lets async code look and read like synchronous code.
🧠 **Trick:** async/await = "Promises wearing a synchronous costume."
🎤 Say: "`async/await` makes asynchronous code read top-to-bottom like sync code, while still being non-blocking under the hood — always wrap in try/catch for errors."

### 5. What happens if you forget to `await` a Promise?
Code moves on immediately; you get a pending Promise object instead of the actual value — can cause race conditions or unhandled rejections.
🧠 **Trick:** Forgetting `await` = "Ordering food but walking away before it arrives."
🎤 Say: "The function continues without waiting for resolution — you might use an unresolved Promise as if it were data, causing bugs, and if it rejects, you get an unhandled promise rejection."

### 6. What is `Promise.all()`?
Runs multiple Promises in parallel, waits for ALL to succeed — if even one fails, the whole thing rejects.
🧠 **Trick:** `Promise.all` = "All students must pass — one fails, whole class fails."
🎤 Say: "Use `Promise.all` when all async tasks are independent and you need every result — but one rejection fails the whole batch."

### 7. Promise.all() vs Promise.allSettled()?
`all()` fails fast on first rejection. `allSettled()` waits for everyone and returns status (fulfilled/rejected) for each.
🧠 **Trick:** `allSettled` = "Report card for everyone, pass or fail, no one is skipped."
🎤 Say: "I use `allSettled` when I want results from all promises regardless of individual failures — e.g., calling 5 independent APIs where partial failure is okay."

### 8. What is the microtask queue?
Queue for Promise callbacks (`.then`, `async/await`, `queueMicrotask`) — runs BEFORE the next macrotask (like setTimeout), after each synchronous block.
🧠 **Trick:** Microtask queue = "VIP line — always served before the general queue (macrotasks)."
🎤 Say: "Microtasks (Promises) have higher priority than macrotasks (setTimeout/setImmediate) and are drained completely before the event loop moves to the next phase."

### 9. setTimeout() vs setImmediate()?
`setTimeout(fn, 0)` runs after a minimum delay in the timers phase. `setImmediate()` runs in the check phase, right after I/O callbacks — order can vary outside I/O cycles, but inside an I/O callback, `setImmediate` always runs first.
🧠 **Trick:** setImmediate = "Do it right after I finish today's mail (I/O)." setTimeout = "Do it after X minutes, whenever that lands."
🎤 Say: "Inside an I/O callback, `setImmediate` always fires before `setTimeout(fn,0)` because of event loop phase order; outside I/O, it's non-deterministic."

### 10. What is process.nextTick()?
Runs a callback immediately after the current operation, BEFORE even the microtask/Promise queue and before the event loop continues.
🧠 **Trick:** `nextTick` = "Cutting the line completely — goes before even the VIPs."
🎤 Say: "`process.nextTick` has the highest priority — it runs before Promises and before the event loop moves to the next phase. Overusing it can starve I/O."

---

## 3️⃣ Node Internals

### 1. How does the Event Loop work? (Detailed)
It cycles through phases: **Timers → Pending Callbacks → Idle/Prepare → Poll (I/O) → Check (setImmediate) → Close Callbacks**. Microtasks (Promises, `nextTick`) run between every phase.
🧠 **Trick:** Remember "**T-P-I-P-C-C**" like a train stopping at each station.
🎤 Say: "Each phase has its own callback queue; the loop processes one phase fully (with microtasks drained after each callback) before moving to the next."

### 2. What is libuv?
The C++ library that gives Node its event loop, thread pool, and async I/O capability across OS platforms.
🧠 **Trick:** libuv = "The engine under Node's hood — Node is the car body, libuv is the engine."
🎤 Say: "libuv abstracts OS-level async I/O and provides the thread pool used for file system, DNS, and crypto operations."

### 3. Why is Node.js single-threaded?
Simplifies programming (no locks/race conditions on the main thread) and is efficient for I/O-heavy apps since it doesn't need a thread per request.
🧠 **Trick:** Single-threaded = "One chef, but he doesn't wait at the oven — he starts the next dish while one bakes."
🎤 Say: "The main JS thread is single-threaded by design to avoid concurrency complexity, while I/O-heavy work is offloaded to libuv's thread pool or the OS kernel."

### 4. If single-threaded, how does it handle thousands of requests?
Non-blocking I/O — the main thread delegates I/O work to the OS/thread pool and just handles callbacks when results are ready, so it's never stuck waiting.
🧠 **Trick:** "One receptionist, but she doesn't wait for your call to finish before taking the next one."
🎤 Say: "Because I/O is non-blocking and delegated, the single thread can rapidly context-switch between many in-flight requests instead of blocking on any one."

### 5. What operations use the thread pool?
File system operations (`fs`), DNS lookups (`dns.lookup`), some `crypto` functions (like `pbkdf2`), and compression (`zlib`).
🧠 **Trick:** Remember "**F-D-C-Z**" — File, DNS, Crypto, Zlib.
🎤 Say: "libuv's thread pool (default size 4) handles these because they can't be made non-blocking at the OS level easily."

### 6. How do you increase thread pool size?
Set environment variable `UV_THREADPOOL_SIZE` (max 128) before the process starts.
🧠 **Trick:** "It's a knob you turn before starting the engine, not while driving."
🎤 Say: "`process.env.UV_THREADPOOL_SIZE = 8` set at the very top of the entry file, or via env var before `node index.js` runs."

### 7. What is blocking I/O?
Operation halts the thread until it completes — nothing else can run meanwhile.
🧠 **Trick:** Blocking = "Standing at the ATM until your transaction fully finishes, no one else can use it."
🎤 Say: "Blocking I/O (like `fs.readFileSync`) freezes the event loop — should be avoided in production Node code."

### 8. What is non-blocking I/O?
Operation starts, thread moves on to other work, and a callback/Promise resolves when the result is ready.
🧠 **Trick:** Non-blocking = "Drop your clothes at the laundromat, come back later — you didn't wait there."
🎤 Say: "Non-blocking I/O is the core of Node's performance model — it lets a single thread serve many concurrent operations."

---

## 4️⃣ Express.js

### 1. What is middleware?
A function that sits between the request and the response — can modify `req`/`res`, or end the cycle, or call `next()`.
🧠 **Trick:** Middleware = "Security checkpoints at the airport — each one checks something before you board."
🎤 Say: "Middleware functions have access to `req, res, next` and form a chain — used for logging, auth, validation, error handling."

### 2. Types of middleware?
Application-level, Router-level, Error-handling, Built-in (e.g. `express.json()`), Third-party (e.g. `cors`, `helmet`).
🧠 **Trick:** "**A-R-E-B-T**" — Application, Router, Error, Built-in, Third-party.
🎤 Say: "I categorize them by scope and purpose — e.g., `helmet` for security headers is third-party, my JWT check is application-level."

### 3. app.use() vs app.get()?
`app.use()` runs for ALL HTTP methods and matches path prefixes. `app.get()` only runs for GET requests on an exact/pattern route.
🧠 **Trick:** `use` = "General security guard at the gate (any visitor)." `get` = "Specific receptionist only for GET-type visitors."
🎤 Say: "`app.use` is for cross-cutting middleware regardless of method; `app.get` is for defining a specific route handler."

### 4. How does Express route matching work?
Routes are matched top-to-bottom in the order they're defined — first match wins (unless `next()` is called to pass through).
🧠 **Trick:** "Race — whoever's route is written first and matches, wins, unless they pass the baton (`next()`)."
🎤 Say: "Express checks middleware/routes sequentially in registration order, so route order matters — put specific routes before generic wildcard ones."

### 5. What is error-handling middleware?
Special middleware with 4 params `(err, req, res, next)` — Express detects it by the arg count and routes errors to it.
🧠 **Trick:** Trick to remember: "4 params = error handler, always last in the chain."
🎤 Say: "I define it as the last middleware with signature `(err, req, res, next)`, and call `next(err)` anywhere to trigger it."

### 6. How do you handle async errors in Express?
Wrap route handlers in try/catch and call `next(err)`, or use a wrapper utility (like `express-async-handler`) to auto-catch rejected Promises.
🧠 **Trick:** "Async errors don't automatically reach Express's safety net — you must throw them into it manually."
🎤 Say: "Since Express doesn't catch async rejections by default, I wrap async routes in a try/catch that forwards errors via `next(err)`, or use a async-wrapper helper."

### 7. What is next()?
Function to pass control to the next middleware in the stack; `next(err)` skips to error-handling middleware.
🧠 **Trick:** `next()` = "Passing the baton in a relay race."
🎤 Say: "Calling `next()` continues the middleware chain; forgetting to call it (when not sending a response) hangs the request."

---

## 5️⃣ API Design

### 1. What is REST?
An architectural style using stateless HTTP requests, resource-based URLs, and standard verbs (GET/POST/PUT/DELETE) to manipulate resources.
🧠 **Trick:** REST = "Treat everything as a noun (resource) + standard verb (HTTP method)."
🎤 Say: "REST principles: statelessness, resource-based URLs, uniform interface, and using HTTP methods/status codes meaningfully."

### 2. PUT vs PATCH?
PUT replaces the ENTIRE resource. PATCH updates only specific fields.
🧠 **Trick:** PUT = "Repaint the whole car." PATCH = "Just fix the scratch."
🎤 Say: "PUT is idempotent full replacement; PATCH is a partial update, and may or may not be idempotent depending on implementation."

### 3. POST vs PUT?
POST creates a NEW resource (server decides ID, not idempotent — calling twice creates two). PUT creates/replaces at a KNOWN URI (idempotent).
🧠 **Trick:** POST = "Drop a suggestion box — new entry every time." PUT = "Label a specific shelf — same shelf every time."
🎤 Say: "POST is for creation where the server assigns identity; PUT is for creating/updating a resource at a client-known URI and is idempotent."

### 4. What are idempotent APIs?
Calling the same request multiple times produces the same result/state as calling it once (GET, PUT, DELETE are idempotent; POST is not).
🧠 **Trick:** Idempotent = "Pressing an elevator button 5 times — elevator still comes once."
🎤 Say: "Idempotency matters for retry-safety — if a network call fails and I retry, an idempotent endpoint won't cause duplicate side effects."

### 5. Query params vs path params?
Path params identify a SPECIFIC resource (`/users/123`). Query params filter/sort/paginate (`/users?age=25&sort=name`).
🧠 **Trick:** Path = "Address of the house." Query = "Instructions on what to do once you're there."
🎤 Say: "I use path params for resource identity and query params for optional filtering, sorting, or pagination."

---

## 6️⃣ Authentication

### 1. How does JWT work?
Server creates a signed token (Header.Payload.Signature) after login; client sends it in every request (usually `Authorization: Bearer`); server verifies the signature without a DB lookup.
🧠 **Trick:** JWT = "A sealed, tamper-proof movie ticket — anyone can read it, but no one can forge it without the key."
🎤 Say: "JWT is stateless auth — the signature (HMAC/RSA) proves the token wasn't tampered with, so the server can trust the payload without a session store."

### 2. Authentication vs Authorization?
Authentication = "Who are you?" (login/identity). Authorization = "What are you allowed to do?" (permissions/roles).
🧠 **Trick:** AuthN = "ID card check at the door." AuthZ = "Which rooms your ID card can unlock."
🎤 Say: "Authentication verifies identity; authorization checks permissions — they're separate concerns, often handled by different middleware layers."

### 3. Why not store passwords in plain text?
If the DB is breached, all passwords are instantly exposed — hashing (one-way) protects users even after a leak.
🧠 **Trick:** "Plain text password = leaving your house key under the doormat with your address written on it."
🎤 Say: "Plaintext storage means a single DB breach compromises every user account — hashing with a strong algorithm and salt prevents that."

### 4. What is bcrypt?
A slow, salted hashing algorithm designed specifically for passwords — deliberately slow to resist brute-force attacks.
🧠 **Trick:** bcrypt = "A lock that takes deliberately long to pick, on purpose."
🎤 Say: "bcrypt auto-generates a salt per password and has a configurable cost factor, making brute-force attacks computationally expensive."

### 5. Refresh token vs access token?
Access token = short-lived, used for actual API calls. Refresh token = long-lived, used only to get a new access token without re-login.
🧠 **Trick:** Access token = "Day pass." Refresh token = "Membership card you use to get a new day pass."
🎤 Say: "Short-lived access tokens limit damage if leaked; the refresh token (stored securely, often httpOnly cookie) lets the user stay logged in without re-authenticating."

---

## 7️⃣ Database

### 1. How do you optimize slow queries?
Add proper indexes, use `EXPLAIN ANALYZE` to see the query plan, avoid `SELECT *`, avoid N+1 queries, paginate large results.
🧠 **Trick:** Remember "**I-E-S-N-P**" — Index, Explain, Select-fields, N+1, Paginate.
🎤 Say: "First I run `EXPLAIN ANALYZE` to find the bottleneck — usually a missing index or an N+1 query pattern — then fix that specifically instead of guessing."

### 2. What are indexes?
Data structures (usually B-trees) that let the DB find rows fast without scanning the whole table — like a book's index page.
🧠 **Trick:** Index = "Book index — jump straight to the page instead of reading cover to cover."
🎤 Say: "Indexes trade write-speed and storage for faster reads — I index columns used often in `WHERE`, `JOIN`, and `ORDER BY`."

### 3. What is connection pooling?
Reusing a fixed set of open DB connections instead of opening/closing a new one per request — much faster and avoids exhausting DB connections.
🧠 **Trick:** Pooling = "Shared office cabs instead of buying a new car for every trip."
🎤 Say: "Connection pooling maintains a set of reusable DB connections, which reduces the overhead of the TCP+auth handshake on every query."

### 4. SQL vs NoSQL?
SQL = structured schema, relational, strong consistency, joins (Postgres, MySQL). NoSQL = flexible schema, document/key-value/graph, built for scale/speed (MongoDB, Redis).
🧠 **Trick:** SQL = "Excel sheet with strict columns." NoSQL = "A flexible folder where each file can look different."
🎤 Say: "I choose SQL when relationships and consistency matter (payments, users); NoSQL when schema flexibility or horizontal scale matters (logs, catalogs)."

### 5. What is a transaction?
A group of DB operations that all succeed together or all fail together (atomicity) — follows ACID properties.
🧠 **Trick:** Transaction = "Bank transfer — money must leave AND arrive, or neither happens."
🎤 Say: "Transactions ensure atomicity — e.g., in Pilooopu's payment flow, deducting credits and confirming an event booking either both commit or both roll back."

---

## 8️⃣ Production

### 1. How do you log errors?
Use structured logging (Winston/Pino) with log levels (info/warn/error), send to a centralized system (CloudWatch, Datadog, Sentry), never `console.log` in prod.
🧠 **Trick:** "Logs are your black box recorder — useless if scattered, powerful if centralized."
🎤 Say: "I use structured JSON logging with correlation IDs so I can trace a single request across services, plus alerts on error-level logs."

### 2. What is rate limiting?
Restricting how many requests a client can make in a time window, to prevent abuse/overload.
🧠 **Trick:** Rate limiting = "Bouncer at the club — only lets in X people per minute."
🎤 Say: "I implement rate limiting (e.g. `express-rate-limit` + Redis) to protect auth/payment endpoints from brute-force and abuse."

### 3. What is CORS?
Browser security feature — a server must explicitly allow which origins (domains) can call its API from the browser.
🧠 **Trick:** CORS = "Bouncer checking your ID's home address before letting you make requests."
🎤 Say: "CORS is enforced by the browser, not the server — I configure allowed origins/headers via the `cors` middleware to control cross-origin access."

### 4. What is compression middleware?
Middleware (e.g. `compression` in Express) that gzips response bodies, reducing payload size over the network.
🧠 **Trick:** Compression = "Vacuum-sealing your luggage before the flight."
🎤 Say: "It reduces bandwidth and speeds up response times, especially for large JSON/HTML payloads — trade-off is a small CPU cost."

### 5. What is caching?
Storing a computed/fetched result temporarily so future requests are served faster without redoing the work.
🧠 **Trick:** Caching = "Keeping leftovers in the fridge instead of cooking from scratch every time."
🎤 Say: "I cache expensive or frequently-read data (e.g., in Redis) with a sensible TTL and invalidation strategy to keep it from going stale."

### 6. When would you use Redis?
Caching, session storage, rate limiting counters, pub/sub, and as a fast queue/lock store (used in BullMQ).
🧠 **Trick:** Redis = "The Swiss Army knife of speed — anything that needs to be read/written in microseconds."
🎤 Say: "I use Redis wherever I need sub-millisecond reads — caching, session store, and as the backing store for BullMQ job queues in Pilooopu."

---

## 9️⃣ Backend Architecture

### 1. What is a message queue?
A system that holds tasks/messages so producers and consumers don't need to talk directly or in real-time — decouples and buffers work.
🧠 **Trick:** Queue = "Restaurant order tickets on a spike — kitchen processes them at its own pace."
🎤 Say: "Message queues decouple services and smooth out traffic spikes — producer pushes a job, consumer processes it independently, often with retries."

### 2. Why use BullMQ?
Redis-backed job queue for Node — gives retries, delays, priorities, concurrency control, and job persistence out of the box.
🧠 **Trick:** BullMQ = "A reliable to-do list manager that never forgets a task, even if the app crashes."
🎤 Say: "I used BullMQ in Pilooopu for a 7-stage crash-resumable pipeline — each stage is its own queue, so a crash mid-way doesn't lose progress, it resumes from the last completed stage."

### 3. What is WebSocket?
A protocol that keeps a persistent, full-duplex connection open between client and server, so either side can push data anytime.
🧠 **Trick:** WebSocket = "An open phone call" vs HTTP's "sending a letter each time."
🎤 Say: "WebSocket upgrades an HTTP connection to a persistent two-way channel — ideal for chat, live notifications, or real-time dashboards."

### 4. WebSocket vs HTTP?
HTTP = request-response, connection closes after each exchange (stateless). WebSocket = one connection stays open, both sides can send anytime.
🧠 **Trick:** HTTP = "Knock, ask, get answer, door closes." WebSocket = "Door stays open all day."
🎤 Say: "HTTP is best for typical CRUD APIs; WebSocket is best when the server needs to push updates to the client without the client asking first."

---

## 🔟 Debugging Scenarios (STAR-style answers)

### 1. API suddenly becomes slow — how do you debug?
Check: recent deploys → APM/traces (which endpoint/query is slow) → DB slow query log → CPU/memory on server → external API latency.
🧠 **Trick:** Remember "**D-A-D-C-E**" — Deploys, APM, DB, CPU, External.
🎤 Say: "I'd first check if it correlates with a recent deploy or traffic spike, then use APM tracing to isolate whether it's DB, CPU, or a downstream API call — narrow from macro to micro."

### 2. Memory usage keeps increasing — how do you investigate?
Suspect a memory leak — take heap snapshots (`--inspect` + Chrome DevTools) at intervals, compare, look for growing objects (unclosed listeners, growing caches/arrays, unresolved promises).
🧠 **Trick:** Memory leak = "A bathtub filling faster than it drains — find what's not being cleaned up."
🎤 Say: "I'd take heap snapshots over time using `node --inspect`, diff them to find growing object types, and check for common leak sources like uncleaned event listeners or unbounded in-memory caches."

### 3. Redis is down — what happens?
Depends on design: if used only as cache, app should fall back to DB (degraded but working); if used for sessions/queues, that functionality breaks — so always design for graceful degradation.
🧠 **Trick:** "Redis down should mean 'slower,' not 'broken' — that's good architecture."
🎤 Say: "I design cache-reads with a fallback to the source of truth and circuit-breaker logic, so a Redis outage degrades performance rather than causing a full outage."

### 4. Database responding slowly — what would you check?
Check active connections/connection pool exhaustion, slow query logs, missing indexes, table locks, disk I/O, and whether a migration or bad query was recently deployed.
🧠 **Trick:** Remember "**C-S-I-L-D**" — Connections, Slow queries, Indexes, Locks, Disk.
🎤 Say: "I'd check connection pool saturation first (a common quick cause), then look at slow query logs and locking, before assuming it's a hardware/disk issue."

### 5. A third-party API is timing out — how would you handle it?
Set explicit timeouts, add retries with exponential backoff, use a circuit breaker to stop hammering a failing service, and have a fallback/cached response if possible.
🧠 **Trick:** Remember "**T-R-C-F**" — Timeout, Retry, Circuit breaker, Fallback.
🎤 Say: "I always set a hard timeout so it doesn't hang my request thread, add retry with backoff for transient failures, and wrap it in a circuit breaker so repeated failures don't cascade into my own service being slow."

---

## 📌 Quick Interview Technique Reminders
- **Structure every answer:** What it is → Why it matters → Real example from your project (Pilooopu/Dawa Saathi).
- **Use the memory tricks above as your mental index** — recall the analogy, then the technical answer follows naturally.
- **For debugging questions:** always narrate a funnel (broad → narrow), interviewers care more about your process than the "correct" final answer.
- **Never say "I don't know" flat** — say "I haven't hit that exact case, but based on [related concept], I'd approach it by..."
