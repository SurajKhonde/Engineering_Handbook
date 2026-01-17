# Basic Nodejs 
The Node event loop (from libuv) is the scheduler that runs callbacks when the JS call stack is empty, but it’s built around server work like I/O. Node’s loop has multiple phases (timers → I/O callbacks → poll → check → close callbacks), and it also has a thread pool for some async tasks (fs/crypto/dns). The browser event loop is focused on UI + rendering (events, DOM callbacks, paint), and it has browser-specific queues like rendering steps and Web APIs.

Does Node.js have Web APIs?

Node does not have browser Web APIs like DOM (window, document) because there’s no webpage.

But Node does have runtime APIs that are similar in purpose:

Timers: setTimeout ✅ (looks like browser)

Some web-standard APIs exist in Node too (like fetch, URL, streams, crypto in many setups) ✅

Node-specific APIs: fs, http, net, process, Buffer ✅

Key point: If an API is called “Web API”, it’s a browser term. In Node, it’s a Node runtime API (even if it looks the same).

Ingredients: Browser vs Node (easy to remember)
Browser = “Webpage machine”

JS Engine (executes JS)

Web APIs (DOM, fetch, timers, events, storage)

Event loop (schedules callbacks)

Renderer (paint UI)

Node = “Server machine”

V8 (executes JS)

Node core APIs (fs, http, net, crypto, Buffer, process)

libuv (event loop + async I/O + thread pool)

C++ bindings (bridge JS ↔ OS)


browsers don’t use libuv.

libuv is a Node.js library that implements Node’s event loop + async I/O integration for Node.

Browsers have their own event loop implementation inside the browser engine (Chromium/Blink, WebKit, Gecko). They expose async through Web APIs (timers, fetch, DOM events, etc.), but it’s not libuv.

Think of JavaScript as just the language, and Browser / Node as two different “machines” that run JS and give it extra powers.

What is “Web API”?

Web APIs are extra features the browser provides to JavaScript that are not part of the JS language itself—examples:

DOM: document, window, events

Network: fetch, XMLHttpRequest

Timers: setTimeout, setInterval

Storage: localStorage, IndexedDB

Why we use Web APIs

Because JS runs on a single main thread, and you can’t block it for:

network calls

timers

user clicks

reading files (in browser sandbox)

So the browser does those tasks in the background and later queues a callback / resolves a promise.
Does Node.js have Web APIs?

Node does not have browser Web APIs like DOM (window, document) because there’s no webpage.

But Node does have runtime APIs that are similar in purpose:

Timers: setTimeout ✅ (looks like browser)

Some web-standard APIs exist in Node too (like fetch, URL, streams, crypto in many setups) ✅

Node-specific APIs: fs, http, net, process, Buffer ✅

Key point: If an API is called “Web API”, it’s a browser term. In Node, it’s a Node runtime API (even if it looks the same).

Ingredients: Browser vs Node (easy to remember)
Browser = “Webpage machine”

JS Engine (executes JS)

Web APIs (DOM, fetch, timers, events, storage)

Event loop (schedules callbacks)

Renderer (paint UI)

Node = “Server machine”

V8 (executes JS)

Node core APIs (fs, http, net, crypto, Buffer, process)

libuv (event loop + async I/O + thread pool)

C++ bindings (bridge JS ↔ OS)

The libuv thread pool is a small pool of background threads Node uses to run certain blocking/slow operations so the main JS thread stays free. It’s mainly used when the OS doesn’t provide a fast non-blocking way, so libuv runs the work on worker threads and then sends the callback back to the event loop.

Examples that use the thread pool (common):

File system operations (fs.readFile, fs.writeFile, etc.)

DNS lookups using dns.lookup() (not dns.resolve which uses network async)

Crypto operations (crypto.pbkdf2, crypto.scrypt, some crypto functions)

Compression (zlib like gzip/deflate)

Top idea: JS doesn’t “pick” async events. JS only registers them. After that, libuv + OS handle the waiting/work. When work is ready, libuv schedules a JS callback, and V8 schedules Promise handlers (microtasks).

Here’s the from-scratch flow in Node, end-to-end:

1) Start: JS runs sync code (main thread)

V8 executes your JS on the call stack.

2) JS hits an async API → it “registers” work

Depending on the API, Node does one of these:

A) Timers (setTimeout)

Node/libuv registers a timer with the event loop.

OS helps with timing (internally).

B) Network I/O (http, sockets)

libuv asks the OS to watch sockets (epoll/kqueue/IOCP).

OS will later say “socket is readable/writable”.

C) File system / crypto / dns.lookup

libuv sends the job to the libuv thread pool (background threads).

When done, thread pool reports completion back to the loop.

✅ So: OS + libuv do the waiting/work (not JS).

3) When the work finishes, who “picks” it?

OS signals readiness (network/timers), OR

Thread pool finishes (fs/crypto/dns.lookup),

libuv receives that completion and queues the corresponding JS callback to run in the event loop (this is the “macrotask side”).

4) Who decides microtask vs macrotask?

This is the key:

Macrotasks (event loop callbacks)

Scheduled by libuv:

setTimeout/setInterval

I/O callbacks (fs, network)

setImmediate

Microtasks

Scheduled by V8 when a Promise settles:

.then/.catch/.finally

queueMicrotask

Promises themselves don’t run on another thread. Only the underlying work might. When that work finishes, Node resolves the Promise, and V8 enqueues the .then as a microtask.

5) Execution order (what runs first)

After Node runs one callback (a macrotask), it does:

process.nextTick queue (Node special, highest)

Microtasks (Promise handlers)

Then goes back and picks the next macrotask from the event loop phases