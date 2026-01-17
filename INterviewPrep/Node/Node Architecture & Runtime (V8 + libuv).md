1. What parts of Node run on the **JS thread** and what parts run **outside** it?
**Ans**
Node runs **JavaScript on one main (JS) thread**—that thread executes **sync code + callbacks + Promise** handlers. Heavy/slow work like file **I/O, DNS, crypto, compression, and network waiting** happens outside the JS thread using **libuv + the OS (and thread pool).** When that work finishes, Node puts the **callback back into the event loop**, and it runs again on the **JS thread.**
```js
const fs = require("fs/promises");
fs.readFile("a.txt")              // file reading happens outside JS thread
  .then(() => console.log("then")) // this runs on JS thread (microtask)
  .catch(console.error);
```
```js
+---------------------+                           +----------------------+
|     CALL STACK      |                           |   Runtime APIs       |
|   (JS Main Thread)  |                           | (Browser APIs / OS)  |
| - sync code         |   async request           |  Timers / Network    |
| - callbacks         |-------------------------->|  File I/O / DNS etc  |
| - promise handlers  |                           +----------+-----------+
+----------+----------+                                      |
           ^                                                 | when done
           |                                                 v
           |                                   +----------------------+
           |                                   |   Queues (Ready work)|
           |                                   +----------+-----------+
           |                                              |
           |      +-------------------+                   |
           |      |  MICROTASK QUEUE  |<------------------+
           |      | (Promises,        |   resolve/reject
           |      |  queueMicrotask)  |
           |      +---------+---------+
           |                |
           |                | drain ALL microtasks first
           |                v
           |      +-------------------+
           |      |  EVENT LOOP       |
           |      | (scheduler)       |
           |      +---------+---------+
           |                |
           |                | then pick ONE macrotask
           |                v
           |      +-------------------+
           |      |  MACROTASK QUEUE  |
           |      | (setTimeout, I/O, |
           |      |  setImmediate)    |
           |      +---------+---------+
           |                |
           +----------------+   push callback back to CALL STACK


Memory rule:
CALL STACK empty?  -> run ALL MICROTASKS  -> run ONE MACROTASK  -> repeat
```

```js
+-----------------------------+            async request            +---------------------------+
|        CALL STACK           |----------------------------------->|     libuv + OS runtime    |
|      (JS Main Thread)       |                                    |  - network sockets (OS)   |
|  - sync JS                  |                                    |  - fs/dns/crypto (pool)   |
|  - callbacks                |                                    |  - timers                 |
|  - Promise handlers         |                                    +-------------+-------------+
+--------------+--------------+                                                  |
               ^                                                                 | when ready
               |                                                                 v
               |                                                   +---------------------------+
               |                                                   |      Ready Queues         |
               |                                                   +-----------+---------------+
               |                                                               |
               |                      resolve/reject Promise                   |
               |                     +-----------------------+                 |
               |                     |   MICROTASK QUEUE     |<----------------+
               |                     | (Promise.then/catch,  |
               |                     |  queueMicrotask)      |
               |                     +-----------+-----------+
               |                                 |
               |              drain ALL microtasks first
               |                                 v
               |                     +-----------------------+
               |                     |      EVENT LOOP       |
               |                     |  (Node scheduling)    |
               |                     +-----------+-----------+
               |                                 |
               |                                 | then ONE macrotask/callback
               |                                 v
               |                     +-----------------------+
               |                     |   MACROTASK QUEUE     |
               |                     | (timers, I/O callbacks|
               |                     |  setImmediate)         |
               |                     +-----------+-----------+
               |                                 |
               +---------------------------------+  push callback to CALL STACK


```

1. Explain the roles of **V8**, **libuv**, and the **C++ bindings** at a high level.
**Ans**
**V8** is the JavaScript engine that compiles and runs your JS code (it also manages memory/GC). 
**libuv** is the library that gives Node its event loop and handles async I/O by talking to the OS (and using a thread pool for things like fs/crypto/dns). 
**C++** bindings are the “bridge” between JS and native code: they connect JS APIs like fs, net, crypto to the underlying C/C++ implementations in Node and libuv/OS.

1. What is the **event loop** in Node and how is it different from the browser event loop?
**Ans**:-
The Node event loop (from libuv) is the scheduler that runs callbacks when the JS call stack is empty, but it’s built around server work like I/O. Node’s loop has multiple phases (timers → I/O callbacks → poll → check → close callbacks), and it also has a thread pool for some async tasks (fs/crypto/dns). The browser event loop is focused on UI + rendering (events, DOM callbacks, paint), and it has browser-specific queues like rendering steps and Web APIs.

2. What is the libuv **thread pool**? Which operations use it (examples)?
The libuv thread pool is a small set of background threads Node uses to run blocking tasks so the main JS thread doesn’t get stuck. It’s used for work that can’t be handled efficiently by the OS’s non-blocking event system.

Common operations that use the thread pool:

File system: fs.readFile, fs.writeFile, etc.

Crypto: crypto.pbkdf2, crypto.scrypt, some hashing/encryption work

Compression: zlib.gzip, zlib.deflate, etc.

DNS: dns.lookup() (not dns.resolve, which is different)

3. What happens when you run CPU‑heavy JS code in Node? How does it affect throughput?
CPU-heavy JS runs on the single main JS thread, so it blocks the event loop. While that code is running, Node can’t process other requests, timers, or callbacks, so latency spikes and throughput drops (requests queue up). Fixes are to move CPU work off the main thread using worker_threads, split work into smaller chunks (yield back to loop), or scale across cores using cluster/multiple processes.
```js
// CPU heavy blocks everything
app.get("/slow", (req, res) => {
  let sum = 0;
  for (let i = 0; i < 2e9; i++) sum += i; // blocks event loop
  res.send(String(sum));
});

```
4. How do **native addons** fit into the runtime? When would you consider them?

Native addons are Node modules written in C/C++ (or Rust etc.) that plug into Node through C++ bindings (N-API). They let JS call native code for speed or to access system/legacy libraries. You consider addons when you have CPU-heavy work that’s too slow in JS (image/video processing, encryption, ML), need to use an existing native library, or need performance-critical code paths. Downsides: harder builds, platform issues, and more maintenance—so prefer pure JS or worker_threads first unless you truly need native speed or native integration.


5. Explain what “Node is single‑threaded” means and what it **does NOT** mean.

What “Node is single-threaded” actually means

When people say Node is single-threaded, they mean:

Your JavaScript runs on one main thread (one call stack).

That one thread executes:

synchronous JS code

your callbacks (HTTP handlers, timers callbacks, etc.)

Promise handlers (microtasks)

most application logic

So: only one piece of JS can run at a time in a single Node process.

That’s the “single-threaded” part.

What it does NOT mean
1) It does NOT mean Node can’t handle many users at once

Node handles high concurrency because it uses non-blocking I/O:

It can keep 10k sockets open

The OS + libuv watch them

When data is ready, Node runs the callback on the JS thread

So Node is concurrent even though JS is single-threaded.

✅ Concurrency = lots of tasks in progress (waiting).
❌ Not necessarily parallel JS execution.

2) It does NOT mean Node uses only one thread internally

Even though JS runs on one thread, Node has other threads doing work:

libuv thread pool runs blocking tasks like:

fs (many file ops)

crypto (pbkdf2/scrypt etc.)

compression (zlib)

dns.lookup()

The OS has its own threads handling networking and scheduling.

Some native libraries can spin threads too.

So the process can be multi-threaded internally, but your JS still runs on one main thread.

3) It does NOT mean async code runs on “another JS thread”

Promises/async-await do not create threads.

The work may happen outside JS (OS/thread pool/DB server)

But your .then() / code after await runs back on the JS thread

So: async ≠ multi-threaded JS

4) It does NOT mean you can’t use multiple CPU cores

A single Node process uses one JS thread → one core for JS work.
But you can scale CPU usage via:

cluster (multiple processes, one per core)

worker_threads (true parallelism for CPU-heavy tasks)

running multiple Node instances behind a load balancer

The real consequence (important)

If you run CPU-heavy JS on the main thread:

event loop is blocked

callbacks can’t run

latency goes up, throughput goes down

So Node is amazing for I/O-heavy apps, and for CPU-heavy you offload work to workers or other services.


6. How do you inspect or reason about **memory usage** in a Node process (high-level tools/approach)?

To reason about Node memory, I first separate heap vs non-heap and look for growth over time (leak) vs stable usage. I check quick metrics using process.memoryUsage() (heapUsed/heapTotal/rss), then reproduce the issue under load and take heap snapshots to find what objects keep growing. For deeper analysis I use Chrome DevTools / Node inspector (node --inspect) or tools like heapdump to compare snapshots, and I watch GC behavior (frequent GC + rising heapUsed usually means retention). I also review common leak sources: global caches, unbounded maps, event listeners, closures holding big objects, and not closing resources (sockets/streams).
```js
// quick runtime view
setInterval(() => {
  const m = process.memoryUsage();
  console.log({
    rssMB: (m.rss / 1024 / 1024).toFixed(1),
    heapUsedMB: (m.heapUsed / 1024 / 1024).toFixed(1),
  });
}, 2000);

```
7. What is a **module** in Node runtime terms (loading, caching, resolution)?

In Node, a module is a file/package that Node loads using its module loader. When you require() (CommonJS) or import (ESM), Node resolves the specifier to an actual file (built-in like fs, local ./x, or package in node_modules), then loads & executes it once, and finally caches the exported result. Because of caching, repeated require('x') returns the same exported object instance without re-running the file, unless you clear the cache. Resolution also follows rules: relative paths resolve from the current file, packages resolve via node_modules lookup, and Node reads package.json (like main / exports / type) to decide entry points and whether it’s CJS or ESM.

```js
// cache behavior (CommonJS)
const a1 = require("./a");
const a2 = require("./a");
console.log(a1 === a2); // true (same cached exports)

```
8.  Scenario: Your API has high latency spikes under load. List 5 Node runtime causes you’d investigate first.
re are 5 Node runtime causes I’d check first for latency spikes under load:

Event loop blocking (CPU-heavy JS)
Big loops, JSON heavy parsing, sync crypto, expensive serialization → blocks the main thread.

Thread pool saturation (libuv pool busy)
Too many fs/crypto/zlib/dns.lookup tasks → queue builds up → requests wait.

GC pressure / memory growth
High allocations or leaks → frequent/long GC pauses → jitter spikes.

Too many microtasks / promise chains (microtask starvation)
Extreme Promise churn can delay timers/I/O callbacks.

Connection/resource saturation
HTTP keep-alive misconfig, too many open sockets, not closing streams, too many concurrent in-flight requests → queueing delays.