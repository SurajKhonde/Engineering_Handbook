# Node.js Deep Interview Q&A (3+ Years) — Buffers, Streams, Event Loop, HTTP, Perf, Internals

---

## How to use
- Do **15 Q/day**.
- For each: explain out loud + write a tiny snippet or draw the flow.
- Star the ones you can’t explain without reading.


---

## Contents
1. Runtime & Event Loop (1–30)
2. Buffers & Binary Data (31–50)
3. Streams & Backpressure (51–80)
4. Files, FS, Upload/Download (81–105)
5. HTTP, Networking, TLS, WebSockets (106–135)
6. Performance, Memory, GC, Debugging (136–165)
7. Modules, Packaging, Security, Architecture (166–200)


---


# 1. Runtime & Event Loop


## 1. What does “Node.js is single-threaded” actually mean?

**What:** Your **JavaScript execution** runs on a single main thread (one call stack).
**Why it matters:** To understand concurrency vs parallelism and why CPU work can hurt all requests.
**How (intuition):** Node uses **libuv** to manage non-blocking I/O. JS runs on one thread, but many I/O operations are handled by the OS (kernel async). Some operations run in a libuv **thread pool** (fs, crypto, zlib...).
**Pitfalls:** Thinking Node can’t use multiple cores. Use **cluster**, **worker_threads**, or multiple processes/pods.
**Interview line:** “JS is single-threaded; Node’s runtime uses OS async I/O + threadpool.”


## 2. What is libuv?

**What:** A C library providing the event loop + async I/O abstraction used by Node.
**Why it matters:** Explains Node’s scalable I/O and cross-platform behavior.
**How (intuition):** Wraps epoll/kqueue/IOCP and schedules callbacks into Node’s event loop phases.
**Interview line:** “libuv is Node’s portability layer for async I/O + loop.”


## 3. Draw the Node event loop phases (classic view).

**What:** The order Node processes callbacks.
**How (simplified):**
```
Timers (setTimeout/setInterval)
  ↓
Pending callbacks (some system ops)
  ↓
Idle/prepare (internal)
  ↓
Poll (I/O callbacks, waits here)
  ↓
Check (setImmediate)
  ↓
Close callbacks (socket 'close', etc)
```
**Extra:** Microtasks (`process.nextTick`, Promises) run at specific checkpoints between phases.


## 4. Promise microtasks vs process.nextTick — ordering?

**What:** In Node, `process.nextTick` callbacks run **before** Promise microtasks.
**Why it matters:** This affects ordering and can cause starvation if abused.
**How (intuition):** After each callback, Node drains `nextTick` queue first, then Promise microtasks, then continues loop.
**Example:** ```js
setTimeout(()=>console.log('timeout'));
Promise.resolve().then(()=>console.log('promise'));
process.nextTick(()=>console.log('nextTick'));
// usually: nextTick, promise, timeout

**Pitfalls:** Recursive `nextTick` can starve I/O; prefer Promises/queueMicrotask for yielding.


## 5. setImmediate vs setTimeout(0) — difference?

**What:** `setImmediate` runs in **check** phase; `setTimeout(0)` runs in **timers** phase after delay.
**Why it matters:** Ordering matters especially around I/O callbacks.
**How (intuition):** After poll (I/O), Node typically goes to check → setImmediate fires quickly; timers depend on time + phase.
**Pitfalls:** Ordering at top-level can vary based on timing; don’t rely on it blindly.
**Interview line:** “After an I/O callback, setImmediate tends to run before timers.”


## 6. What operations use the libuv thread pool?

**What:** A fixed-size pool (default 4) for tasks that would block.
**Why it matters:** Threadpool saturation causes latency spikes and weird slowdowns.
**How (intuition):** Common users: many `fs` calls, `crypto` (pbkdf2/scrypt), `zlib`, some DNS operations.
**Pitfalls:** 4 heavy crypto tasks can block fs reads. Tune `UV_THREADPOOL_SIZE` carefully and limit concurrency.


## 7. How do you scale CPU-bound work in Node?

**What:** Move CPU-heavy work off the event loop via `worker_threads` or run multiple processes/pods.
**Why it matters:** CPU work blocks the loop and increases latency for everyone.
**How (intuition):** Processes scale across cores; workers enable shared memory/message passing within one process.
**Pitfalls:** Doing image processing, big JSON parse, heavy crypto in request path.


## 8. What is event-loop blocking? How do you detect it?

**What:** Main thread is busy → it can’t process I/O callbacks/timers.
**Why it matters:** Direct cause of p95/p99 spikes.
**How (intuition):** Use `perf_hooks.monitorEventLoopDelay()`, CPU profiles, flamegraphs; correlate with latency.
**Pitfalls:** Sync loops, sync crypto/fs, huge JSON stringify/parse, catastrophic regex.


## 9. Why can one slow request affect all users in Node?

**What:** Because callbacks run serially on one thread; long sync code delays all callbacks.
**Why it matters:** Explains how a single hot path can cause global latency spikes.
**Interview line:** “Concurrency isn’t parallelism: Node is concurrent with async I/O, but JS runs serially.”


## 10. What is AsyncLocalStorage and why used?

**What:** Request-scoped storage that persists across async boundaries.
**Why it matters:** Tracing/logging (requestId, userId) without passing context everywhere.
**Pitfalls:** Overhead and context leaks if mis-scoped; use carefully.
**Interview line:** “Like thread-locals, but for async.”


## 11. What is graceful shutdown in Node servers?

**Goal:** Stop accepting new traffic, finish in-flight, close resources, then exit.
**Steps:**
- Handle SIGTERM/SIGINT.
- `server.close()` (stop accepting).
- Track in-flight requests; set a hard shutdown timeout.
- Close DB/Redis connections; stop consumers; flush logs.
**Pitfalls:** keep-alive sockets, long requests, forgotten intervals/handles.


## 12. What is keep-alive and why does it matter?

**What:** Reuse TCP connections for multiple HTTP requests.
**Why it matters:** Reduces latency but increases open sockets and resource usage.
**How (intuition):** Tune client Agents (`keepAlive`, `maxSockets`) and set server timeouts.
**Pitfalls:** No timeouts → idle sockets leak; too many sockets overload upstream.


## 13. What is threadpool starvation and how do you spot it?

**What:** Long threadpool tasks block other threadpool tasks.
**Why it matters:** Can slow unrelated operations like fs while crypto runs.
**How (intuition):** Symptoms: fs latency spikes during crypto; look at CPU, libuv metrics, and request traces.
**Pitfalls:** Fix by limiting concurrency, using workers, or increasing pool cautiously.


## 14. Unhandled promise rejection — what should your service do?

**What:** It indicates a bug; handle locally, but decide policy for process-level handling.
**Why it matters:** Silent async failures are dangerous.
**How (intuition):** Use try/catch around awaits; log; for truly unknown state, prefer graceful shutdown + restart.
**Pitfalls:** Swallowing errors globally; continuing in corrupted state.
**Interview line:** “Crash-only design is safer than limping along after unknown exceptions.”


## 15. What is `process.nextTick` starvation?

**What:** `nextTick` runs before I/O; heavy use can prevent the loop from reaching poll/check.
**Why it matters:** Can freeze the server under load.
**Pitfalls:** Recursive nextTick; use `setImmediate`/Promises to yield.


## 16. What is the difference between concurrency and parallelism (Node context)?

**What:** Concurrency: many tasks in progress (interleaving). Parallelism: multiple tasks executing at the same time on multiple cores.
**Why it matters:** Node excels at I/O concurrency; needs processes/workers for CPU parallelism.


## 17. Timers (setTimeout) — are they exact?

**What:** No. They run **not earlier than** the delay, but later if the loop is busy.
**Why it matters:** Explains timer drift and scheduling issues under load.
**Pitfalls:** Using timers for precise scheduling; use cron/queue schedulers for reliability.


## 18. What are active handles and why does Node sometimes not exit?

**What:** Open sockets/timers/servers keep the loop alive.
**Why it matters:** Common in tests and shutdown bugs.
**How (intuition):** Close servers, clear intervals, close pools; use unref for non-essential timers.


## 19. What is `unref()` and when to use it?

**What:** Allows process to exit even if timer/socket exists.
**Why it matters:** Useful for background maintenance timers in CLIs.
**Pitfalls:** Unref-ing critical resources can exit early unexpectedly.


## 20. Why is `async/await` not multi-threading?

**What:** It yields control while waiting; it doesn’t run JS in parallel.
**Why it matters:** Common misconception.


## 21. What is an EventEmitter memory leak pattern?

**What:** Adding listeners per request and never removing them.
**Why it matters:** Causes memory growth and warnings (`MaxListenersExceededWarning`).
**Pitfalls:** Just increasing max listeners hides the leak; fix the pattern.


## 22. Why can console.log be slow in production?

**What:** stdout can be blocking/buffered; huge logs add CPU and I/O contention.
**Why it matters:** Can increase tail latency.
**Pitfalls:** Logging large objects/buffers per request; prefer structured logs + sampling.


# 2. Buffers & Binary Data


## 23. What is a Buffer in Node.js?

**What:** A Buffer represents raw binary data (bytes) in Node.
**Why it matters:** Network, files, crypto operate on bytes, not JS strings.
**How (intuition):** Backed by memory outside V8 heap; Node adds helpful APIs on top of Uint8Array.
**Pitfalls:** Confusing string length vs byte length (UTF‑8).


## 24. Buffer vs ArrayBuffer vs Uint8Array — differences?

**What:** ArrayBuffer = raw memory block; Uint8Array = typed view; Buffer = Node’s Uint8Array subclass with extra APIs.
**Why it matters:** Interoperability with browser code and binary protocols.
**Pitfalls:** Copy vs view differences; some conversions share memory.


## 25. Why do Unicode strings cause bugs with buffers?

**What:** Strings are UTF‑16 code units; buffers are bytes; UTF‑8 is variable-length.
**Why it matters:** Incorrect size limits, truncation, and corrupted text.
**Example:** `'€'.length === 1` but `Buffer.byteLength('€') === 3`.


## 26. What is endianness and where you see it?

**What:** Byte order for multi-byte integers (LE/BE).
**Why it matters:** Binary protocols, file formats.
**How (intuition):** Use `readUInt32LE/BE`, `writeInt16LE/BE`.
**Pitfalls:** Wrong endianness gives wrong values silently.


## 27. What is Buffer pooling and why does it exist?

**What:** Node reuses a slab for small buffers to reduce allocations.
**Why it matters:** Performance under many small allocations.
**Pitfalls:** Slices may share underlying memory; mutate carefully and copy when needed.


## 28. alloc vs from vs allocUnsafe — explain + when to use.

**What:** `Buffer.alloc(n)` = zero-filled; `Buffer.from(data)` = from existing data; `allocUnsafe(n)` = fast uninitialized.
**Why it matters:** Security and correctness.
**Pitfalls:** allocUnsafe can leak old memory if not fully overwritten before use.
**Interview line:** “Unsafe is okay only if you overwrite all bytes before reading/sending.”


## 29. How does HTTP request body arrive in Node?

**What:** As a stream of Buffer chunks on `req` (Readable stream).
**Why it matters:** You must handle backpressure and size limits.
**Pitfalls:** Collecting whole body with no limit → DoS risk.


## 30. How do you implement request body size limit safely?

**What:** Track bytes and abort if exceeds maximum (respond 413).
**Why it matters:** Prevents memory abuse and slowloris variants.
**Pitfalls:** Not destroying the stream/socket properly can leave open connections.


## 31. Why is converting huge buffers to strings dangerous?

**What:** Decoding allocates large strings on the heap and burns CPU.
**Why it matters:** Causes memory pressure and GC pauses.
**Pitfalls:** Logging `buffer.toString()` for big files.


## 32. What is base64 and when should you avoid it?

**What:** Binary-to-text encoding.
**Why it matters:** Adds ~33% size overhead and CPU cost.
**Pitfalls:** Don’t send large files as base64 JSON; use streaming/multipart or direct upload.


## 33. What are ‘views’ vs ‘copies’ with Buffer slicing?

**What:** Slicing often creates a view sharing same memory; copying allocates new memory.
**Pitfalls:** Mutating a view changes original; use explicit copy if needed.


## 34. What is `TextDecoder/TextEncoder` and why use it?

**What:** Standard encoding/decoding APIs for string↔bytes.
**Why it matters:** More consistent across Node and browsers.


## 35. How do you validate file type safely?

**What:** Check magic bytes / signatures and do not trust extension or client Content-Type.
**Why it matters:** Security: attackers can rename malicious files.


## 36. Explain zero-copy file send at OS level (sendfile).

**What:** Kernel can send file bytes from page cache to socket without copying into user-space.
**Why it matters:** High performance static file serving.
**Interview line:** “Zero-copy reduces CPU/memory bandwidth.”


## 37. What is `highWaterMark` for buffers/streams?

**What:** Threshold controlling internal buffering before applying backpressure.
**Why it matters:** Tune between throughput and memory.


## 38. How do you compute checksum while streaming upload?

**What:** Hash chunks as they arrive and finalize at end.
**Why it matters:** Integrity checks and dedup.
**Example:** Use `crypto.createHash('sha256')` + `hash.update(chunk)` in a pipeline.


## 39. Why do buffers often show up as ‘external’ memory?

**What:** Buffers allocate outside V8-managed heap.
**Why it matters:** Heap graphs can look stable while RSS grows; must monitor RSS/external memory.


## 40. When do you prefer Buffer over streams?

**What:** Small payloads needing random access; otherwise streams for large data.
**Interview line:** “For > few MB or unknown size, default to streaming.”


## 41. What is a binary protocol and why you might use it?

**What:** A compact byte format (protobuf, msgpack) vs JSON.
**Why it matters:** Smaller, faster parse, stronger schemas (protobuf).
**Pitfalls:** Harder debugging, compatibility rules needed.


# 3. Streams & Backpressure


## 42. What is a stream in Node?

**What:** An interface to read/write data incrementally as chunks.
**Why it matters:** Memory-bounded processing and built-in backpressure.
**How (intuition):** Types: Readable, Writable, Duplex, Transform.
**Interview line:** “Streams are Node’s backpressure primitive.”


## 43. Readable vs Writable vs Duplex vs Transform — define with examples.

**Readable:** produces data (`req`, `fs.createReadStream`).
**Writable:** consumes data (`res`, `fs.createWriteStream`).
**Duplex:** both directions (`net.Socket`).
**Transform:** duplex that transforms (`gzip`, encryption, parsing).


## 44. What is backpressure in streams?

**What:** Consumer signals producer to slow down when its buffer is full.
**Why it matters:** Prevents memory blowups under slow clients or slow downstream.
**How (intuition):** Writable `.write()` returns false → pause source; resume on `'drain'`. `pipe()` handles it.
**Pitfalls:** Ignoring `.write()` return value causes unbounded buffering.


## 45. How does `pipe()` manage backpressure?

**What:** Automatically pauses/resumes readable based on writable buffer state.
**Why it matters:** Safer than manual loops.
**Pitfalls:** You must still handle errors; use `pipeline()` for robust cleanup.


## 46. What is `stream.pipeline()` and why is it preferred?

**What:** Connects streams and ensures errors/close propagate correctly (with cleanup).
**Why it matters:** Prevents FD leaks and half-open pipes.
**Example:** 
```js
import { pipeline } from 'node:stream/promises';
await pipeline(src, transform, dest);
```
**Interview line:** “Use pipeline for production streaming.”


## 47. Flowing mode vs paused mode — what changes?

**What:** Flowing emits `data` events; paused requires `.read()` calls.
**Why it matters:** Affects parser implementation and control.
**Pitfalls:** Switching modes unintentionally and missing data.


## 48. What is objectMode and when is it useful?

**What:** Stream chunks are objects instead of bytes.
**Why it matters:** ETL: stream rows/records not raw bytes.
**Pitfalls:** highWaterMark counts objects, not bytes; large objects can still OOM.


## 49. How do you manually implement backpressure?

```js
src.on('data', (chunk) => {
  if (!dest.write(chunk)) src.pause();
});
dest.on('drain', () => src.resume());
```
**Pitfall:** must handle `error` + `close` on both sides.


## 50. What do `end`, `finish`, and `close` mean?

**Readable `end`:** no more data.
**Writable `finish`:** all writes flushed (after `end()` called).
**`close`:** underlying resource closed.
**Pitfall:** wait for correct event depending on what you need.


## 51. How do you stream large JSON safely?

**What:** Prefer NDJSON or a streaming parser; avoid parsing giant arrays at once.
**Why it matters:** Huge JSON parse blocks event loop and uses huge memory.
**Interview line:** “For large datasets, design format for streaming (NDJSON).”


## 52. What is NDJSON and why is it great?

**What:** Newline-delimited JSON: one JSON object per line.
**Why it matters:** Parse line-by-line; easy streaming.
**Pitfalls:** Handle partial lines across chunk boundaries.


## 53. Streams vs async iterators — how to consume?

**What:** Readable streams support `for await...of` with backpressure.
**Example:** 
```js
for await (const chunk of readable) {
  // process chunk
}
```


## 54. What is `highWaterMark` tuning tradeoff?

**What:** Higher = more buffering (memory) but fewer syscalls; lower = less memory but more overhead.
**Pitfalls:** Raising it to ‘fix speed’ can crash memory under many concurrent streams.


## 55. How to implement throttling (bytes/sec) in a pipeline?

**What:** Use a Transform that delays pushing based on elapsed time and chunk size.
**Why it matters:** Protect downstream and control bandwidth.


## 56. What are Range requests and why needed?

**What:** Partial content retrieval with `Range: bytes=...` (status 206).
**Why it matters:** Video seeking and resumable downloads.
**Pitfalls:** Incorrect `Content-Range` breaks clients.


## 57. How do you stop streaming work when client disconnects?

**What:** Listen to `res.close` / `req.aborted` and destroy upstream streams/requests.
**Why it matters:** Avoid wasted work and resource leaks.


## 58. How to stream request to upstream + stream response back (proxy)?

**Pattern:** `pipeline(req, upstreamReq)` and `pipeline(upstreamRes, res)` with abort propagation.
**Pitfalls:** timeouts, header forwarding, backpressure, and disconnect handling.


## 59. What is a decompression bomb and how to protect?

**What:** Compressed input expands massively (zip bomb).
**Why it matters:** Can OOM or spike CPU.
**Pitfalls:** Always enforce decompressed size limits and timeouts.


## 60. When do you prefer SSE over WebSockets?

**What:** When you only need server→client updates (notifications, feed updates).
**Why it matters:** SSE is simpler, plays well with HTTP infra.


## 61. How do you broadcast WS messages across multiple Node instances?

**What:** Use Redis pub/sub or Kafka; each instance forwards to its local sockets.
**Why it matters:** Without shared bus, broadcasts only reach one instance.
**Pitfalls:** Redis pub/sub has no replay; Kafka can replay with offsets.


## 62. Common stream bugs that interviewers love?

**What:** Ignoring errors, ignoring backpressure, collecting whole stream in memory, leaking FDs, not handling disconnect.
**Why it matters:** These map directly to real outages.


## 63. How do you upload to S3 without buffering whole file in Node?

**What:** Stream upload, often with multipart upload support.
**Why it matters:** Memory safety for large files.
**Pitfalls:** Need retries and part size management; enforce timeouts and max size.


## 64. What is `AbortController` in Node and why use it?

**What:** Standard cancellation mechanism for fetch and some APIs.
**Why it matters:** Cancel upstream calls on timeouts/client abort.


## 65. When should you choose streams over buffers?

**What:** If payload can be large or unknown size, stream. If small and random access needed, buffer.


# 4. Files, FS, Upload/Download


## 66. fs.readFile vs fs.createReadStream — which and why?

**What:** `readFile` loads entire file; stream reads in chunks.
**Why it matters:** Streams keep memory bounded and support backpressure.


## 67. Sync fs methods — why are they dangerous on servers?

**What:** They block the event loop.
**Why it matters:** Increases latency for all concurrent requests.
**Pitfalls:** Using `readFileSync` or `existsSync` in request handlers.


## 68. How does Node handle file I/O internally?

**What:** Many fs operations use libuv threadpool because file APIs are often blocking.
**Why it matters:** Threadpool saturation can slow fs and crypto together.


## 69. Best practice: send a file in Node (with pipeline).

```js
import { createReadStream } from 'node:fs';
import { pipeline } from 'node:stream/promises';

export async function sendFile(res, path) {
  res.setHeader('Content-Type', 'application/octet-stream');
  await pipeline(createReadStream(path), res);
}
```
**Why:** bounded memory + proper backpressure + cleanup on errors.


## 70. How do you handle file uploads at scale (real design)?

**Best pattern:** client uploads directly to object storage using a pre-signed URL.
- API creates upload session + returns pre-signed URL and constraints.
- Client uploads to S3/GCS directly (Node not in the hot path).
- Storage event triggers async processing worker (virus scan, thumbnails, transcoding).
- DB stores metadata + status transitions.
**Why:** removes huge bandwidth/memory load from Node and reduces outages.


## 71. Multipart/form-data — why is it tricky?

**What:** Boundaries and parts can split across chunks; must parse incrementally.
**Why it matters:** Naive string parsing breaks and can be exploited.
**Interview line:** “Use a streaming multipart parser (Busboy).”


## 72. How do you prevent path traversal when serving files?

**What:** Ensure user input cannot escape the allowed directory or map IDs to stored paths.
**How (intuition):** Resolve path and confirm it starts with your root; don’t concatenate raw user input.
**Pitfalls:** `../` attacks or URL-encoded traversal.


## 73. DB vs object storage for files — which and why?

**What:** Store blobs in object storage; store metadata in DB.
**Why it matters:** DB backups/replication become heavy if you store blobs; object storage scales and is cheaper.


## 74. What is content-addressable storage (CAS)?

**What:** Use content hash as key (sha256).
**Why it matters:** Dedup and integrity checks.
**Pitfalls:** Need reference counting/GC for orphaned blobs.


## 75. How to compute checksum while streaming upload?

**What:** Hash chunks as they stream and finalize at end.
**Why it matters:** Detect corruption and enable dedup.


## 76. What is a file descriptor leak? How does it happen in Node?

**What:** Not closing file streams/sockets leads to EMFILE errors.
**Why it matters:** Causes outages under concurrency.
**How (intuition):** Use `pipeline`, handle errors, destroy streams on abort.


## 77. EMFILE/ENFILE errors — what do they mean?

**What:** Too many open files (per process or system).
**Why it matters:** Often indicates leaks or unbounded concurrency.
**How (intuition):** Fix leaks, limit concurrency, raise ulimit carefully.


## 78. What happens if client disconnects during file download?

**What:** Writable stream closes; you should stop reading file and close fd.
**Why it matters:** Avoid wasted I/O and leaked resources.


## 79. Static files: serve from Node or Nginx/CDN?

**What:** Prefer CDN/Nginx; Node for dynamic APIs.
**Why it matters:** CDN edge caching + optimized file serving reduces load and improves latency.


## 80. Range requests: how do you implement for local files?

**What:** Parse Range header, validate bounds, create stream with start/end, respond 206 with Content-Range.
**Why it matters:** Video seeking/resume downloads.
**Pitfalls:** Wrong ranges or missing headers break clients.


## 81. How do you store file metadata?

**What:** DB table: fileId, ownerId, storageKey, size, mime, checksum, status, createdAt.
**Why it matters:** Enables auth, lifecycle, and auditing.


## 82. How do you do safe temp file usage?

- Randomized filenames in OS temp dir, restricted permissions.
- Cleanup on success/failure and periodic sweeps on startup.
- Prefer direct-to-object-store to reduce temp disk dependency.


## 83. fs.watch vs fs.watchFile — difference?

**What:** watch uses OS events; watchFile polls.
**Why it matters:** Used in dev tools and watchers.
**Pitfalls:** watch reliability varies across platforms; debouncing needed.


## 84. Why is reading directory contents per request a bad idea?

**What:** FS calls are slow and threadpool-bound.
**Why it matters:** Adds latency and can starve threadpool under load.
**Pitfalls:** Cache directory index or precompute.


## 85. What headers matter for downloads?

- `Content-Type`
- `Content-Length` (if known)
- `Content-Disposition` (inline/attachment; filename)
- `Accept-Ranges` (bytes) when supported
- Cache headers (ETag/Cache-Control)


# 5. HTTP, Networking, TLS, WebSockets


## 86. In Node `http` server, what are `req` and `res` really?

**What:** `req` is Readable stream (IncomingMessage); `res` is Writable stream (ServerResponse).
**Why it matters:** Understanding streaming and backpressure depends on this.


## 87. Why is streaming response good for performance?

**What:** You can send bytes early; memory stays bounded.
**Why it matters:** Improves TTFB and prevents buffering large payloads.


## 88. How do you tune outbound HTTP keep-alive?

**What:** Use an Agent with keepAlive and set maxSockets/timeouts.
**Why it matters:** Without reuse you pay handshake cost; with too many sockets you overload upstream.
**Pitfalls:** No timeouts leads to socket leaks; too high maxSockets can DDoS your own dependency.


## 89. What is head-of-line blocking (HOL) and where it appears?

**What:** One slow item blocks others behind it.
**Why it matters:** Affects latency and throughput.
**How (intuition):** HTTP/1.1 pipelining issues; TCP loss can affect HTTP/2 multiplexing; HTTP/3 reduces transport HOL with QUIC.


## 90. TLS termination — why at a proxy/load balancer?

**What:** Edge handles TLS; backend gets plain HTTP (or re-encrypted).
**Why it matters:** Centralized cert management and security policies.
**Pitfalls:** Internal traffic may be unencrypted unless you enable mTLS/internal TLS.


## 91. What is mTLS?

**What:** Mutual TLS: both sides authenticate using certificates.
**Why it matters:** Strong service identity in microservices/zero-trust.
**Pitfalls:** Operational complexity (rotation, trust chains).


## 92. WebSocket handshake — what’s the flow?

**What:** HTTP request with Upgrade headers; server responds 101 Switching Protocols.
**Why it matters:** Helps debug proxy/LB issues and auth.


## 93. Why do WebSockets stress servers differently than HTTP?

**What:** Long-lived connections consume memory/CPU; fanout costs; scaling requires shared pub/sub.
**Why it matters:** You hit memory and connection limits before CPU sometimes.


## 94. How to scale WebSockets across many Node instances?

**What:** LB distributes connections; Redis/Kafka pubsub for broadcast; shared session/presence store; connection limits and backpressure.
**Pitfalls:** No shared bus means cross-instance broadcast fails.


## 95. SSE vs WebSocket vs polling — choose which?

**What:** SSE for server→client; WS for two-way; polling for low frequency simplicity.
**Why it matters:** Choosing right tool reduces complexity.


## 96. What is chunked transfer encoding?

**What:** Send response as chunks without Content-Length.
**Why it matters:** Enables streaming.
**Pitfalls:** Some proxies buffer; disable buffering for streaming endpoints.


## 97. How do you implement rate limiting in Node with Redis?

**What:** Use counters with TTL or token bucket; key by IP/user/token.
**Why it matters:** Protects from abuse and smooths load.
**Pitfalls:** In-memory limiters break with multiple instances; must use shared store.


## 98. How do you ensure timeouts for outbound calls?

**What:** Set connect/read/total timeouts; abort via AbortController.
**Pitfalls:** Defaults can be infinite causing leaked sockets and hanging requests.


## 99. What is DNS caching risk in long-lived Node processes?

**What:** Stale DNS entries can keep routing to dead IPs.
**Why it matters:** Microservices with service discovery may change endpoints.
**Pitfalls:** Some clients resolve once and never refresh; configure resolver/agent appropriately.


## 100. dns.lookup vs dns.resolve — difference?

**What:** lookup uses OS resolver; resolve performs actual DNS queries.
**Why it matters:** Caching behavior differs.


## 101. How do you protect against slowloris?

**What:** Set header/body timeouts, min data rate, connection limits, use proxy mitigations.
**Pitfalls:** Not setting `headersTimeout`/`requestTimeout` or proxy timeouts.


## 102. What is HTTP/2 in Node and what changes?

**What:** Multiplexing streams and header compression.
**Why it matters:** Fewer connections; different performance patterns.
**Pitfalls:** TCP loss still affects all streams; not a magic bullet.


## 103. Why sit behind a reverse proxy (nginx/envoy) in production?

**What:** Proxy handles TLS, buffering, static, WAF, rate limits, routing.
**Why it matters:** Stability and security; Node focuses on app logic.


## 104. How do you handle client aborts in Node request handlers?

**What:** Listen to `req.aborted`/`res.close` and cancel DB/upstream work when possible.
**Why it matters:** Prevents wasted work and reduces tail latency.


## 105. What is `res.flushHeaders()` and when useful?

**What:** Send headers immediately.
**Why it matters:** Useful for streaming responses (SSE) and early TTFB.
**Pitfalls:** After flush you can’t change status/headers.


## 106. What is `SO_REUSEPORT` and why mention it?

**What:** Multiple processes can bind same port; kernel distributes accepts.
**Why it matters:** Alternative multi-process scaling pattern.
**Pitfalls:** OS-dependent behavior; deployment must be consistent.


## 107. What are the most important server timeouts to set in Node?

- `headersTimeout` (protects from slow headers)
- `requestTimeout` (overall request)
- keep-alive timeouts
- socket timeouts
- upstream client timeouts


# 6. Performance, Memory, GC, Debugging


## 108. List the top causes of p99 latency spikes in Node.

**What:** Event loop blocking, GC pauses, threadpool starvation, downstream slowness/timeouts, connection pool contention, excessive logging.
**Why it matters:** Interviewers want root cause thinking.


## 109. How does GC impact Node latency?

**What:** GC pauses stop JS briefly; bigger heaps can increase pause time.
**Why it matters:** Tail latency is often GC-related.
**How (intuition):** Reduce allocations, stream data, avoid huge objects, tune memory limits, reuse buffers carefully.


## 110. Heap vs RSS vs external memory — explain.

**What:** Heap = V8 JS objects; RSS = total process memory; external = native/buffers outside heap.
**Why it matters:** You can OOM while heap looks fine (buffers/external).
**Interview line:** “Monitor RSS + heap + external; buffers can inflate RSS.”


## 111. How do you debug memory leaks in Node?

**What:** Take heap snapshots over time and compare; check retained paths.
**Why it matters:** Leaks are usually references held in Maps/caches/listeners.
**How (intuition):** Use `--inspect`, DevTools, heapdump, clinic.js.
**Pitfalls:** Misdiagnosing external memory growth as heap leak; watch RSS.


## 112. How do you profile CPU in Node?

**What:** Use flamegraphs/profilers to find hotspots.
**How (intuition):** Chrome DevTools profiler, `node --prof`, `0x`, clinic flame.
**Pitfalls:** Profiling in prod needs sampling to reduce overhead.


## 113. How do you measure event loop delay?

**What:** `perf_hooks.monitorEventLoopDelay()` and track percentiles.
**Why it matters:** Direct signal for blocking on main thread.


## 114. Why is regex sometimes a DoS vector (ReDoS)?

**What:** Catastrophic backtracking consumes CPU.
**Why it matters:** Attackers can craft inputs to freeze server.
**Pitfalls:** Unsafe regex on user input; validate length and use safer patterns.


## 115. Why is `Promise.all` risky on large arrays?

**What:** It fires everything at once (unbounded concurrency).
**Why it matters:** Can overload DB/services and blow memory.
**How (intuition):** Use concurrency limiting (semaphore/p-limit).


## 116. What is a semaphore/concurrency limiter and where to use it?

**What:** Limits simultaneous tasks (DB queries, fs reads, upstream calls).
**Why it matters:** Protect dependencies and stabilize tail latency.


## 117. How to debug ‘process won’t exit’ after tests?

**What:** Active handles keep loop alive (open servers, intervals, sockets, DB pools).
**How (intuition):** Close pools, clear timers, ensure `await server.close()`, inspect active handles.


## 118. Why is big logging a performance issue?

**What:** Serialization + I/O overhead; can block or contend.
**Why it matters:** Often increases p99 under load.
**Pitfalls:** Logging huge objects/buffers; logging per request at info.


## 119. What should you monitor in Node production?

- p50/p95/p99 latency, error rate, QPS
- event loop delay
- heap used + GC pauses
- RSS/external memory
- threadpool queue delay
- DB pool saturation
- upstream timeouts/retries
- open sockets/FD usage


## 120. How do you pick Node memory settings in containers?

**What:** Set `--max-old-space-size` to match container limits minus headroom for native/external memory.
**Why it matters:** Avoid OOM kills and excessive GC.
**Pitfalls:** Setting too high causes container OOM; too low causes frequent GC and slowdowns.


## 121. What is circuit breaker and bulkhead for Node services?

**What:** Circuit breaker stops calling failing dependency; bulkhead isolates resource pools per dependency.
**Why it matters:** Prevents cascading failures and protects tail latency.


## 122. Threadpool size tuning — when to increase UV_THREADPOOL_SIZE?

**What:** When you have many concurrent fs/crypto/zlib tasks and observe starvation.
**Why it matters:** Can reduce queueing delay.
**Pitfalls:** Too high can cause CPU contention and reduce throughput; always measure.


## 123. What’s the difference between optimizing throughput vs latency?

**What:** Throughput: total QPS; latency: time per request + tail behavior.
**Why it matters:** Different strategies: batching helps throughput; timeouts/isolation helps tail; caching helps both.


## 124. How do you cancel downstream work when client disconnects?

**What:** Use abort signals; cancel fetch requests; stop streaming; avoid continuing heavy DB work when possible.
**Why it matters:** Saves capacity and reduces tail latency.


## 125. What is `--inspect` and when do you use it?

**What:** Enables debugging/profiling via Chrome DevTools.
**Why it matters:** CPU profile, heap snapshot, async stack traces.
**Pitfalls:** Don’t leave open in production without access controls.


## 126. What is a ‘crash-only’ philosophy and why used?

**What:** On unexpected state, log and exit; rely on supervisor to restart.
**Why it matters:** Safer than running in corrupted state.
**Interview line:** “Fail fast, restart clean, keep state external.”


## 127. Why does JSON.parse/stringify show up in CPU profiles?

**What:** Serialization is CPU-heavy and allocates many objects.
**Why it matters:** Common bottleneck in high QPS APIs.
**How (intuition):** Avoid huge payloads; use binary codecs; stream where possible.


## 128. What is external memory pressure and how to reduce it?

**What:** Buffers/native allocations increase RSS beyond heap.
**Why it matters:** Can trigger OOM while heap is fine.
**How (intuition):** Stream, avoid buffering, release references, tune caching and connection limits.


## 129. How do you implement safe retries in Node clients?

**What:** Retry only idempotent ops; use exponential backoff+jitter; cap retries; enforce timeouts.
**Why it matters:** Avoid retry storms.
**Pitfalls:** Retrying POST without idempotency key creates duplicates.


# 7. Modules, Packaging, Security, Architecture


## 130. CommonJS vs ESM — core differences?

**What:** CJS uses require/exports; ESM uses import/export with static analysis.
**Why it matters:** Affects tree-shaking, top-level await, and package export behavior.
**Pitfalls:** Interop issues with default exports; mixed module type errors.


## 131. High-level Node module resolution algorithm?

**What:** Resolve relative paths directly; for packages search node_modules up directory tree; respect package `exports` (ESM).
**Why it matters:** Debug ‘cannot find module’ and deep-import issues.


## 132. What is `package.json` `exports` and why it can break deep imports?

**What:** Exports restrict what paths can be imported from the package.
**Why it matters:** Better encapsulation but breaks `pkg/lib/internal` imports if not exported.


## 133. What is a lockfile and why is it critical in production?

**What:** Pins exact dependency versions for reproducible builds.
**Why it matters:** Prevents surprise breaking changes on deploy.


## 134. Semver: why can `^` still break you?

**What:** Allows minor updates; some packages break semver in practice.
**Why it matters:** Lockfiles + controlled upgrades are safer.


## 135. Security checklist for Node APIs (must-know).

- Input validation (schema)
- Body size limits
- Paramized SQL
- Authn/Authz
- Rate limiting
- Timeouts
- Secure cookies (HttpOnly/SameSite)
- Secrets management
- Dependency audits
- Avoid ReDoS regex
- SSRF protections
- Don’t log secrets/PII


## 136. What is SSRF and how to prevent it (Node)?

**What:** Attacker makes server call internal URLs via a feature like `fetch?url=`.
**Why it matters:** Can leak metadata/credentials (cloud).
**How (intuition):** Allowlist domains, block private IP ranges after DNS resolution, block redirects, enforce timeouts and size limits.


## 137. What is prototype pollution and why does it matter?

**What:** Unsafe deep merges let attacker modify object prototypes, changing behavior globally.
**Why it matters:** Can lead to auth bypass or crashes.
**Pitfalls:** Using vulnerable deep-merge utilities on untrusted JSON.


## 138. Design a file upload service (best architecture).

1) `POST /uploads/init` → returns uploadId + pre-signed URL + constraints
2) Client uploads to S3 directly
3) `POST /uploads/complete` → mark done
4) Worker processes file (scan/thumbnail) via queue
5) Store metadata/status in DB
**Benefits:** Node avoids huge streaming load, better reliability, cheaper bandwidth.


## 139. Design a file download service (best architecture).

- Auth check
- Prefer CDN for public/cacheable
- For protected: signed URL with short TTL OR authenticated streaming endpoint
- Support Range requests for media
- Cancel work on disconnect
- Observability: bytes sent, aborts, latency


## 140. Idempotency keys: how to implement in Node API?

**What:** Client sends unique key; server stores key→response and returns same response on retries.
**Why it matters:** Prevents duplicate orders/payments.
**How (intuition):** Store in DB/Redis with TTL; atomic check+set (transaction/unique constraint).


## 141. Why do you need timeouts everywhere?

**What:** Without timeouts, requests hang and consume sockets/workers forever.
**Why it matters:** A single stuck dependency can exhaust capacity and cause outage.
**Interview line:** “Timeouts are the seatbelts of distributed systems.”


## 142. Why should you avoid in-memory sessions in multi-instance Node?

**What:** Sessions tied to a single instance require sticky load balancing and break failover.
**Why it matters:** Stateless scaling is simpler: use Redis or JWT (with refresh).


## 143. How to avoid N+1 network calls in Node microservices?

**What:** Batching, caching, dataloaders, or change API to fetch aggregates.
**Why it matters:** RTT dominates; N+1 kills tail latency.


## 144. Job queue in Node (BullMQ): what problems does it solve?

**What:** Offloads slow tasks to workers with retries/delays/concurrency controls.
**Why it matters:** Protects request latency and increases reliability.
**Pitfalls:** Jobs must be idempotent; monitor retries; DLQ for poison jobs.


## 145. How to answer Node ‘buffers vs streams’ in one crisp line?

**What:** Buffers hold the whole data in memory; streams process data chunk-by-chunk with backpressure.
**Interview line:** “Use Buffer for small; stream for large/unknown.”


## 146. How to answer Node ‘how do files get sent’ question (crisp)?

**Answer:** File is read as a stream (chunks) and piped to the HTTP response, so memory stays bounded and backpressure prevents buffering if client is slow.


## 147. What is `res.write()` return value and why important?

**What:** It returns false when internal buffer is full.
**Why it matters:** Signal for backpressure; wait for `drain`.


## 148. What is a ‘slow consumer’ and how do you protect server?

**What:** Client reads response slowly so server buffers.
**How (intuition):** Backpressure + timeouts + max buffering + terminate very slow clients.


## 149. What is `headersTimeout` vs `requestTimeout`?

**What:** headersTimeout limits time to receive headers; requestTimeout limits full request duration.
**Why it matters:** Protects from slowloris and long-hanging requests.


## 150. What is `Content-Disposition` and why used for downloads?

**What:** Controls inline vs attachment and filename.
**Why it matters:** Better UX and correct browser behavior.


## 151. Why is `existsSync` discouraged in request path?

**What:** It blocks the event loop and is race-prone.
**How (intuition):** Attempt open and handle errors instead.


## 152. What is `stream.finished()`?

**What:** A helper that resolves/calls back when a stream ends or errors.
**Why it matters:** Useful for awaiting completion when not using pipeline.


## 153. Why do you need DLQ in background processing?

**What:** Some jobs will fail repeatedly (poison). DLQ isolates them for inspection.
**Why it matters:** Prevents infinite retries from burning resources.


## 154. How do you ensure upload endpoints aren’t used for DoS?

**What:** Enforce size limits, timeouts, auth, rate limits; store directly to object storage; validate content type.


## 155. What is `AbortSignal` used for in Node fetch?

**What:** Cancel in-flight requests on timeout or client abort.
**Why it matters:** Frees sockets and reduces resource waste.


## 156. What is connection pool ‘queue time’?

**What:** Time waiting to acquire a connection from pool.
**Why it matters:** Signal of saturation; causes latency spikes.


## 157. What is ‘keep-alive mismatch’ between LB and Node?

**What:** LB closes idle sockets earlier/later than Node expects causing resets.
**How (intuition):** Align keep-alive and idle timeout settings across layers.


## 158. What is ‘backpressure’ at HTTP client side?

**What:** When streaming request body to upstream, respect upstream writable buffer to avoid OOM.


## 159. How to prevent decompression bombs?

**What:** Limit decompressed output size and time; don’t trust compressed inputs.


## 160. Why should you avoid logging request bodies?

**What:** PII/secrets leak and huge payloads cause CPU/memory overhead.


## 161. What is ‘hot path’ vs ‘cold path’?

**What:** Hot path runs per request; cold path is startup/rare operations.
**Why it matters:** Optimize hot path aggressively; parse env/config once at startup.


## 162. What is `undici` and why it matters?

**What:** High-performance HTTP client used by Node’s `fetch` in modern versions.
**Why it matters:** Better pooling and performance than naive clients; still needs timeouts and agents.


## 163. What is `res.write()` return value and why important?

**What:** It returns false when internal buffer is full.
**Why it matters:** Signal for backpressure; wait for `drain`.


## 164. What is a ‘slow consumer’ and how do you protect server?

**What:** Client reads response slowly so server buffers.
**How (intuition):** Backpressure + timeouts + max buffering + terminate very slow clients.


## 165. What is `headersTimeout` vs `requestTimeout`?

**What:** headersTimeout limits time to receive headers; requestTimeout limits full request duration.
**Why it matters:** Protects from slowloris and long-hanging requests.


## 166. What is `Content-Disposition` and why used for downloads?

**What:** Controls inline vs attachment and filename.
**Why it matters:** Better UX and correct browser behavior.


## 167. Why is `existsSync` discouraged in request path?

**What:** It blocks the event loop and is race-prone.
**How (intuition):** Attempt open and handle errors instead.


## 168. What is `stream.finished()`?

**What:** A helper that resolves/calls back when a stream ends or errors.
**Why it matters:** Useful for awaiting completion when not using pipeline.


## 169. Why do you need DLQ in background processing?

**What:** Some jobs will fail repeatedly (poison). DLQ isolates them for inspection.
**Why it matters:** Prevents infinite retries from burning resources.


## 170. How do you ensure upload endpoints aren’t used for DoS?

**What:** Enforce size limits, timeouts, auth, rate limits; store directly to object storage; validate content type.


## 171. What is `AbortSignal` used for in Node fetch?

**What:** Cancel in-flight requests on timeout or client abort.
**Why it matters:** Frees sockets and reduces resource waste.


## 172. What is connection pool ‘queue time’?

**What:** Time waiting to acquire a connection from pool.
**Why it matters:** Signal of saturation; causes latency spikes.


## 173. What is ‘keep-alive mismatch’ between LB and Node?

**What:** LB closes idle sockets earlier/later than Node expects causing resets.
**How (intuition):** Align keep-alive and idle timeout settings across layers.


## 174. What is ‘backpressure’ at HTTP client side?

**What:** When streaming request body to upstream, respect upstream writable buffer to avoid OOM.


## 175. How to prevent decompression bombs?

**What:** Limit decompressed output size and time; don’t trust compressed inputs.


## 176. Why should you avoid logging request bodies?

**What:** PII/secrets leak and huge payloads cause CPU/memory overhead.


## 177. What is ‘hot path’ vs ‘cold path’?

**What:** Hot path runs per request; cold path is startup/rare operations.
**Why it matters:** Optimize hot path aggressively; parse env/config once at startup.


## 178. What is `undici` and why it matters?

**What:** High-performance HTTP client used by Node’s `fetch` in modern versions.
**Why it matters:** Better pooling and performance than naive clients; still needs timeouts and agents.


## 179. What is `res.write()` return value and why important?

**What:** It returns false when internal buffer is full.
**Why it matters:** Signal for backpressure; wait for `drain`.


## 180. What is a ‘slow consumer’ and how do you protect server?

**What:** Client reads response slowly so server buffers.
**How (intuition):** Backpressure + timeouts + max buffering + terminate very slow clients.


## 181. What is `headersTimeout` vs `requestTimeout`?

**What:** headersTimeout limits time to receive headers; requestTimeout limits full request duration.
**Why it matters:** Protects from slowloris and long-hanging requests.


## 182. What is `Content-Disposition` and why used for downloads?

**What:** Controls inline vs attachment and filename.
**Why it matters:** Better UX and correct browser behavior.


## 183. Why is `existsSync` discouraged in request path?

**What:** It blocks the event loop and is race-prone.
**How (intuition):** Attempt open and handle errors instead.


## 184. What is `stream.finished()`?

**What:** A helper that resolves/calls back when a stream ends or errors.
**Why it matters:** Useful for awaiting completion when not using pipeline.


## 185. Why do you need DLQ in background processing?

**What:** Some jobs will fail repeatedly (poison). DLQ isolates them for inspection.
**Why it matters:** Prevents infinite retries from burning resources.


## 186. How do you ensure upload endpoints aren’t used for DoS?

**What:** Enforce size limits, timeouts, auth, rate limits; store directly to object storage; validate content type.


## 187. What is `AbortSignal` used for in Node fetch?

**What:** Cancel in-flight requests on timeout or client abort.
**Why it matters:** Frees sockets and reduces resource waste.


## 188. What is connection pool ‘queue time’?

**What:** Time waiting to acquire a connection from pool.
**Why it matters:** Signal of saturation; causes latency spikes.


## 189. What is ‘keep-alive mismatch’ between LB and Node?

**What:** LB closes idle sockets earlier/later than Node expects causing resets.
**How (intuition):** Align keep-alive and idle timeout settings across layers.


## 190. What is ‘backpressure’ at HTTP client side?

**What:** When streaming request body to upstream, respect upstream writable buffer to avoid OOM.


## 191. How to prevent decompression bombs?

**What:** Limit decompressed output size and time; don’t trust compressed inputs.


## 192. Why should you avoid logging request bodies?

**What:** PII/secrets leak and huge payloads cause CPU/memory overhead.


## 193. What is ‘hot path’ vs ‘cold path’?

**What:** Hot path runs per request; cold path is startup/rare operations.
**Why it matters:** Optimize hot path aggressively; parse env/config once at startup.


## 194. What is `undici` and why it matters?

**What:** High-performance HTTP client used by Node’s `fetch` in modern versions.
**Why it matters:** Better pooling and performance than naive clients; still needs timeouts and agents.


## 195. What is `res.write()` return value and why important?

**What:** It returns false when internal buffer is full.
**Why it matters:** Signal for backpressure; wait for `drain`.


## 196. What is a ‘slow consumer’ and how do you protect server?

**What:** Client reads response slowly so server buffers.
**How (intuition):** Backpressure + timeouts + max buffering + terminate very slow clients.


## 197. What is `headersTimeout` vs `requestTimeout`?

**What:** headersTimeout limits time to receive headers; requestTimeout limits full request duration.
**Why it matters:** Protects from slowloris and long-hanging requests.


## 198. What is `Content-Disposition` and why used for downloads?

**What:** Controls inline vs attachment and filename.
**Why it matters:** Better UX and correct browser behavior.


## 199. Why is `existsSync` discouraged in request path?

**What:** It blocks the event loop and is race-prone.
**How (intuition):** Attempt open and handle errors instead.


## 200. What is `stream.finished()`?

**What:** A helper that resolves/calls back when a stream ends or errors.
**Why it matters:** Useful for awaiting completion when not using pipeline.
