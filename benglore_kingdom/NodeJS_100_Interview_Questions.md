# Node.js Deep Dive — 100 Interview Questions
### Explained like you're 10, remembered forever

---

## Section 1: What is Node.js & Why Single Threaded (Q1–12)

**Q1. What is Node.js? Explain like I'm 5.**

Imagine a restaurant kitchen.

Old kitchens (like Apache/Java): Every customer gets their OWN personal chef. 1000 customers = 1000 chefs standing around. Most chefs are just WAITING (for the oven, for the fridge). Huge waste.

Node.js kitchen: ONE super-fast chef. But this chef never waits. He puts the pizza in oven, immediately starts cutting vegetables, then checks if pizza is done, moves to next task. Never stands idle.

That one chef is Node.js. The trick is — he never does slow work himself. He delegates (to the oven, the fridge, the dishwasher) and keeps moving.

```
Traditional Server:
Request 1 → Thread 1 (waits for DB... waits... waits... responds)
Request 2 → Thread 2 (waits for DB... waits... waits... responds)
1000 requests = 1000 threads = OUT OF MEMORY

Node.js:
Request 1 → "Go fetch from DB, call me when done" → moves on
Request 2 → "Go fetch from DB, call me when done" → moves on
ONE thread handles all! Callbacks fire when data is ready.
```

---

**Q2. Why is Node.js single-threaded? Isn't that slow?**

JavaScript was born in the browser. Browsers have ONE main thread (can't have two JS scripts modifying the DOM simultaneously — chaos!). Node.js kept that same model.

But "single-threaded" is misleading. It means ONE thread runs YOUR JavaScript code. But Node.js is NOT fully single-threaded internally.

```
What you see (single-threaded):
Your JS code → runs on V8 engine → one thread

What's actually happening (multi-threaded behind the scenes):
Your JS code → one thread
File I/O     → libuv thread pool (4 threads by default)
DNS lookup   → libuv thread pool
Crypto ops   → libuv thread pool
OS network   → OS kernel async I/O (no threads needed!)
```

So Node.js is single-threaded for YOUR code, but multi-threaded for heavy work behind the scenes. Best of both worlds.

---

**Q3. What is libuv? Why does it matter?**

libuv is the C library that gives Node.js its superpowers. It's the engine under the hood.

Think of Node.js as a car:
- V8 = the engine that runs JavaScript
- libuv = the gearbox, wheels, and fuel system that connects to the real world

```
libuv provides:
1. Event Loop          — the heartbeat of Node.js
2. Thread Pool         — for CPU-heavy async work (file I/O, crypto, DNS)
3. Async I/O           — network, pipes, TTY
4. Timers              — setTimeout, setInterval
5. Child Processes     — spawn other programs
6. Signal handling     — SIGTERM, SIGINT etc.
7. File System         — async file operations
```

Without libuv, Node.js would just be V8 with no way to talk to the operating system.

---

**Q4. What is the Event Loop? Explain with a simple story.**

The Event Loop is like a waiter in a restaurant who keeps checking: "Is there anything ready for me to serve?"

```
1. Your Node.js program starts
2. ALL synchronous code runs first (top to bottom)
3. When async operations complete (timers, file reads, network),
   their callbacks are placed in queues
4. When the call stack is EMPTY, the Event Loop wakes up
5. It checks the queues in a specific ORDER (phases)
6. Picks the next callback, puts it on the call stack, runs it
7. Goes back to checking queues
8. Repeats until no more callbacks
9. Program exits
```

```js
console.log('A');                              // sync
setTimeout(() => console.log('B'), 0);        // macro task
Promise.resolve().then(() => console.log('C')); // micro task
console.log('D');                              // sync

// Output: A, D, C, B
// A and D are sync (run first)
// C is microtask (runs before any macrotask)
// B is macrotask (runs last)
```

---

**Q5. What are the phases of the Event Loop? Most asked question.**

The Event Loop has 6 phases. Think of it as a merry-go-round visiting 6 stations in order.

```
Phase 1: TIMERS         → setTimeout, setInterval callbacks
Phase 2: PENDING I/O    → I/O callbacks deferred from previous loop
Phase 3: IDLE/PREPARE   → internal use only
Phase 4: POLL           → fetch new I/O events (waits here if nothing)
Phase 5: CHECK          → setImmediate callbacks
Phase 6: CLOSE          → socket.on('close') callbacks

Between EVERY phase: drain microtask queue completely
(process.nextTick first, then Promise.then)
```

---

**Q6. Explain each Event Loop phase in detail.**

**Phase 1: Timers** — Runs callbacks from `setTimeout` and `setInterval` whose delay has EXPIRED. `setTimeout(fn, 0)` doesn't mean "run immediately" — it means "run as soon as possible after 0ms".

**Phase 2: Pending Callbacks** — Runs I/O callbacks deferred from the previous iteration (like TCP errors). You rarely interact with this directly.

**Phase 3: Idle, Prepare** — Internal to libuv. You never touch this.

**Phase 4: Poll (most important)** — Two jobs: calculate how long to wait for I/O, then process events in the poll queue. If poll queue empty and setImmediate callbacks waiting → move to check phase. If nothing → sit here and wait (saves CPU).

**Phase 5: Check** — Runs `setImmediate` callbacks. Always runs after I/O in current iteration.

**Phase 6: Close Callbacks** — Runs `.on('close', ...)` callbacks. Example: socket closed abruptly.

---

**Q7. Difference between setTimeout(fn,0), setImmediate, and process.nextTick?**

Most common tricky Node.js interview question.

```js
// In a normal script:
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('Promise'));

// Output: nextTick, Promise, setTimeout, setImmediate
// (setTimeout vs setImmediate order may vary when NOT inside I/O)

// Inside an I/O callback (always deterministic):
fs.readFile('file.txt', () => {
  setTimeout(() => console.log('setTimeout'), 0);
  setImmediate(() => console.log('setImmediate'));
});
// Output: setImmediate, setTimeout — ALWAYS
// After I/O, we're in poll phase. Next phase is CHECK (setImmediate).
// Timers phase comes in next loop iteration.
```

```
process.nextTick:
- NOT part of event loop phases
- Runs after CURRENT operation, before event loop continues
- Higher priority than Promise.then
- Danger: recursive nextTick can starve I/O!

setImmediate:
- Runs in CHECK phase (after poll/I/O phase)
- "Run after current I/O events"

setTimeout(fn, 0):
- Runs in TIMERS phase
- After setImmediate when inside I/O callback
```

---

**Q8. What is the call stack? How does it relate to Node.js?**

The call stack tracks what function is currently executing. Last in, first out.

```js
function greet(name) { return `Hello ${name}`; }
function main() {
  const msg = greet('Suraj');
  console.log(msg);
}
main();

// Stack progression:
// [main]
// [main, greet]     ← greet called
// [main]            ← greet returns
// [main, console.log]
// [main]
// []                ← empty! Event Loop checks queues
```

If the call stack is NEVER empty (infinite sync loop), the Event Loop never runs callbacks. This is why blocking the event loop is critical to avoid.

---

**Q9. What does "blocking the event loop" mean? Why is it dangerous?**

```js
// BLOCKING — never do this in a route handler!
app.get('/bad', (req, res) => {
  let sum = 0;
  for (let i = 0; i < 10_000_000_000; i++) {
    sum += i;   // takes 10 seconds, blocks ALL other requests!
  }
  res.json({ sum });
});

// BLOCKING file read
const data = fs.readFileSync('huge-file.txt'); // blocks event loop!

// NON-BLOCKING
app.get('/good', async (req, res) => {
  const data = await fs.promises.readFile('file.txt'); // non-blocking
  res.json({ data });
});

// For CPU-heavy work → use worker threads
```

---

**Q10. Difference between synchronous and asynchronous in Node.js?**

```js
// Synchronous — blocks everything until done
const data = fs.readFileSync('file.txt', 'utf8'); // WAIT HERE
console.log('after');                              // runs after file read

// Asynchronous — registers callback, moves on immediately
fs.readFile('file.txt', 'utf8', (err, data) => {
  console.log(data);  // runs LATER
});
console.log('after'); // runs IMMEDIATELY, before file is read!

// Output: 'after' first, then file data
```

Use `readFileSync` only in startup scripts (reading config at boot). Never in request handlers.

---

**Q11. How does Node.js handle 10,000 concurrent connections with one thread?**

```
Secret: Node.js NEVER waits. Every slow operation is handed off.

Client 1 → "fetchData from DB, callback when done" → Node moves on
Client 2 → "fetchData from DB, callback when done" → Node moves on
... 10,000 clients registered ...

Node.js is now free. Just waiting.

DB responds for Client 1 → callback runs → response sent
DB responds for Client 4 → callback runs → response sent

Memory per connection: ~few KB (just the callback and data)
Thread-per-request: ~1-2MB per thread × 10000 = 10-20GB RAM!
```

Node.js is perfect for I/O-heavy apps. Bad for CPU-heavy apps (video processing, ML inference).

---

**Q12. What is the thread pool in Node.js?**

libuv maintains a thread pool for operations that can't be done async at OS level.

```
Thread pool handles:
- File system operations (fs.readFile, fs.writeFile)
- DNS lookups (dns.lookup — but NOT dns.resolve!)
- Crypto operations (crypto.pbkdf2, crypto.randomBytes)
- Compression (zlib)

Default pool size: 4 threads
Change with: UV_THREADPOOL_SIZE=8 node app.js (max 128)

Network I/O (HTTP, TCP, UDP) does NOT use thread pool.
OS handles networking asynchronously at the kernel level.
That's why Node.js handles thousands of network connections
without a big thread pool.
```

```js
// Demonstrating thread pool limit
const crypto = require('crypto');
for (let i = 0; i < 8; i++) {
  crypto.pbkdf2('password', 'salt', 100000, 64, 'sha512', () => {
    console.log(i, Date.now(), 'ms');
  });
}
// With pool 4: first 4 complete together, then next 4
// With UV_THREADPOOL_SIZE=8: all 8 complete at similar time
```

---

## Section 2: Modules System (Q13–22)

**Q13. What is the module system in Node.js?**

```
CommonJS (CJS) — original, default in Node.js
  require() to import
  module.exports to export
  Synchronous loading
  .js files by default

ES Modules (ESM) — modern standard
  import/export keywords
  Asynchronous loading
  .mjs files, or "type":"module" in package.json
```

```js
// CommonJS
const express = require('express');
const { readFile } = require('fs');
module.exports = { myFunction };

// ES Modules
import express from 'express';
import { readFile } from 'fs/promises';
export const myFunction = () => {};
export default myFunction;
```

---

**Q14. How does require() work internally? Step by step.**

```
Step 1: RESOLVE — Find the actual file path
  './myModule' tries: .js → .json → .node → /index.js

Step 2: LOAD — Read the file content

Step 3: WRAP — Node wraps your code in this function:
  (function(exports, require, module, __filename, __dirname) {
    // YOUR CODE HERE
  });
  This gives every file its own scope!

Step 4: EVALUATE — Run the wrapped function

Step 5: CACHE — Store in require.cache
  Next require() of same file → returns cached exports
  Module code runs ONLY ONCE ever!
```

```js
// Prove caching:
// counter.js
let count = 0;
module.exports = { increment: () => ++count, getCount: () => count };

// app.js
const c1 = require('./counter');
const c2 = require('./counter'); // same reference from cache!
c1.increment();
console.log(c2.getCount()); // 1 — same object!
c1 === c2; // true
```

---

**Q15. Difference between module.exports and exports?**

```js
// exports is just a shortcut reference to module.exports initially
// exports === module.exports === {} at start

// SAFE: add properties
exports.greet = function() { return 'hi'; };       // works
module.exports.greet = function() { return 'hi'; }; // works

// DANGER: reassign exports — breaks the reference!
exports = function() { };  // WRONG — module.exports still {}
                            // require() gets {} not your function!

// CORRECT: reassign module.exports
module.exports = function() { };  // CORRECT
module.exports = { greet, name }; // CORRECT
```

---

**Q16. What is __dirname and __filename?**

```js
console.log(__filename); // /home/suraj/project/app.js — full path of THIS file
console.log(__dirname);  // /home/suraj/project — directory of THIS file

// These are injected by Node's module wrapper (the IIFE)
// Each file has its own values — NOT global

// Common use: safe path building
const path = require('path');
const configPath = path.join(__dirname, 'config', 'db.json');
// Always relative to THIS FILE, not to where you ran node

// In ES Modules (__dirname doesn't exist):
import { fileURLToPath } from 'url';
import { dirname } from 'path';
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

---

**Q17. What is the path module? Why not string concatenation?**

```js
const path = require('path');

// Never build paths with + strings (breaks on Windows which uses \)

path.join('/users', 'suraj', 'file.txt');   // /users/suraj/file.txt
path.join(__dirname, 'config', 'db.json');  // safe relative path
path.resolve('config', 'db.json');          // absolute path from cwd
path.extname('file.txt');                   // '.txt'
path.basename('/users/suraj/file.txt');     // 'file.txt'
path.dirname('/users/suraj/file.txt');      // '/users/suraj'
path.parse('/home/suraj/file.txt');
// { root: '/', dir: '/home/suraj', base: 'file.txt', ext: '.txt', name: 'file' }
```

---

**Q18. What is the os module?**

```js
const os = require('os');

os.cpus().length  // number of CPU cores — use for worker/cluster count
os.totalmem()     // total RAM in bytes
os.freemem()      // free RAM in bytes
os.platform()     // 'linux', 'darwin', 'win32'
os.arch()         // 'x64', 'arm64'
os.hostname()     // machine name
os.homedir()      // '/home/suraj'
os.tmpdir()       // '/tmp'
os.uptime()       // seconds since boot
```

---

**Q19. What is EventEmitter?**

EventEmitter is the backbone of Node.js. Almost everything in Node (streams, HTTP, fs) extends EventEmitter.

```js
const EventEmitter = require('events');

class PaymentProcessor extends EventEmitter {
  processPayment(amount) {
    if (amount > 10000) {
      this.emit('fraud-alert', { amount });
    } else {
      this.emit('success', { amount });
    }
  }
}

const processor = new PaymentProcessor();

processor.on('success', (data) => console.log(`Payment ₹${data.amount} ok`));
processor.on('fraud-alert', (data) => console.log(`ALERT! ₹${data.amount}`));
processor.once('ready', () => console.log('Only fires once'));

processor.processPayment(500);    // success event
processor.processPayment(15000);  // fraud-alert event

// Always remove listeners to prevent memory leaks!
processor.off('success', handler);
processor.removeAllListeners('success');
processor.setMaxListeners(20); // default 10, increase if needed
```

---

**Q20. What is the Buffer module?**

Buffer handles binary data directly — raw bytes, not strings.

```js
// Why Buffer? JS strings are UTF-16.
// Network data, files, images — these are RAW BYTES.

const buf1 = Buffer.alloc(10);           // 10 bytes, filled with zeros
const buf2 = Buffer.from('Hello');       // from string
const buf3 = Buffer.from([72, 101, 108, 108, 111]); // from byte array

buf2.toString();          // 'Hello'
buf2.toString('hex');     // '48656c6c6f'
buf2.toString('base64');  // 'SGVsbG8='
buf2.length;              // 5 bytes

// When you receive TCP data, HTTP body, file read — you get a Buffer.
// Convert to string when you're sure it's text data.
```

---

**Q21. What is the crypto module?**

```js
const crypto = require('crypto');

// Hash (one way — can't reverse)
const hash = crypto.createHash('sha256').update('password123').digest('hex');

// HMAC (hash with secret key — used in JWT signature)
const hmac = crypto.createHmac('sha256', 'secret').update('data').digest('hex');

// Random token (for session IDs, reset tokens)
const token = crypto.randomBytes(32).toString('hex'); // 64 char random string

// Slow password hash (prevents brute force)
crypto.pbkdf2('password', 'salt', 100000, 64, 'sha512', (err, key) => {
  const hash = key.toString('hex');
});

// UUID
crypto.randomUUID(); // '9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d'
```

---

**Q22. What is the util module?**

```js
const util = require('util');

// promisify — convert callback-style function to Promise
const readFile = util.promisify(require('fs').readFile);
const data = await readFile('file.txt', 'utf8');

// util.inspect — deep print objects (better than JSON.stringify)
util.inspect(obj, { depth: null, colors: true });

// util.format — like printf
util.format('Hello %s, you are %d years old', 'Suraj', 25);
// 'Hello Suraj, you are 25 years old'

// util.isDeepStrictEqual
util.isDeepStrictEqual({ a: 1 }, { a: 1 }); // true
```

---

## Section 3: File System & Streams (Q23–32)

**Q23. What is the fs module? All ways to read a file.**

```js
const fs = require('fs');
const fsp = require('fs/promises');

// 1. Synchronous (BLOCKS — only at startup)
const data = fs.readFileSync('file.txt', 'utf8');

// 2. Callback (old way)
fs.readFile('file.txt', 'utf8', (err, data) => { });

// 3. Promise (modern, preferred)
const data = await fsp.readFile('file.txt', 'utf8');

// 4. Stream (for LARGE files — never loads full file in memory)
fs.createReadStream('file.txt').pipe(process.stdout);

// Writing
await fsp.writeFile('out.txt', 'content');
await fsp.appendFile('out.txt', '\nmore');

// Other
await fsp.mkdir('newFolder', { recursive: true });
await fsp.readdir('./');
await fsp.unlink('file.txt');
await fsp.rename('a.txt', 'b.txt');
const stats = await fsp.stat('file.txt'); // size, mtime, isFile(), isDirectory()
```

---

**Q24. What are Streams? Why are they important?**

Imagine moving a lake of water:
- Option 1: Fill entire lake in ONE huge tank, carry it, pour it. (readFile — works for small files, crashes for big ones)
- Option 2: Connect a pipe. Water flows in small chunks. (Streams — memory efficient, starts immediately)

```js
// Problem without streams (2GB video):
const data = fs.readFileSync('video.mp4');  // 2GB in RAM!
res.send(data);

// With streams (never loads full 2GB):
fs.createReadStream('video.mp4').pipe(res);

// Types of streams:
// Readable  — you can READ (fs.createReadStream, http.IncomingMessage)
// Writable  — you can WRITE (fs.createWriteStream, http.ServerResponse)
// Duplex    — both (net.Socket)
// Transform — read, modify, write (zlib.createGzip)
```

---

**Q25. How do you use streams practically?**

```js
const { pipeline } = require('stream/promises');
const fs = require('fs');
const zlib = require('zlib');

// Read → compress → write (constant memory regardless of file size!)
await pipeline(
  fs.createReadStream('big-file.txt'),
  zlib.createGzip(),
  fs.createWriteStream('big-file.txt.gz')
);

// HTTP streaming response
app.get('/download', (req, res) => {
  res.setHeader('Content-Type', 'video/mp4');
  fs.createReadStream('movie.mp4').pipe(res);
});

// Transform stream — custom processing
const { Transform } = require('stream');
const upperCase = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});
await pipeline(
  fs.createReadStream('input.txt'),
  upperCase,
  fs.createWriteStream('output.txt')
);
```

---

**Q26. What is backpressure in streams?**

Imagine filling a bucket with a fire hose — the bucket overflows. Backpressure is protection against this.

```js
// WRONG — no backpressure
readable.on('data', chunk => {
  writable.write(chunk); // what if writable is slower? buffer overflows!
});

// CORRECT — .pipe() handles backpressure automatically
readable.pipe(writable);
// When writable buffer is full → pipe PAUSES readable
// When writable drains → pipe RESUMES readable

// Manual backpressure:
readable.on('data', chunk => {
  const canContinue = writable.write(chunk);
  if (!canContinue) {
    readable.pause();
    writable.once('drain', () => readable.resume());
  }
});
```

---

**Q27. Difference between pipe and pipeline?**

```js
// pipe — old way, poor error handling
r.pipe(g).pipe(w);
// If r errors, w is NOT automatically closed → resource leak!

// pipeline — modern, proper error handling, cleans up all streams
const { pipeline } = require('stream/promises');
try {
  await pipeline(
    fs.createReadStream('file.txt'),
    zlib.createGzip(),
    fs.createWriteStream('file.gz')
  );
} catch (err) {
  // ALL streams automatically destroyed on error
}
```

Always use `pipeline` in production. `pipe` causes resource leaks on errors.

---

**Q28. Difference between readFile and createReadStream?**

```js
// readFile — loads ENTIRE file into memory
// Good for: config files, small JSON data (< 1MB)
const data = await fs.promises.readFile('config.json', 'utf8');

// createReadStream — reads in chunks (default 64KB)
// Good for: large files, HTTP downloads, log processing (> 1MB)
const stream = fs.createReadStream('access.log', { highWaterMark: 64 * 1024 });

// Rule:
// File < 1MB   → readFile
// File > 1MB   → streams
// File > 10MB  → MUST use streams
```

---

**Q29. How do you watch files for changes?**

```js
// fs.watch — efficient, uses OS file system events
const watcher = fs.watch('config.json', (eventType, filename) => {
  console.log(`${filename} changed: ${eventType}`);
});

fs.watch('./src', { recursive: true }, (eventType, filename) => {
  console.log(`${filename} ${eventType}`);
});

watcher.close(); // stop watching

// This is what nodemon uses internally!
```

---

**Q30. What are the important stream events?**

```js
// Readable stream events:
stream.on('data', (chunk) => { });    // chunk available
stream.on('end', () => { });          // no more data
stream.on('error', (err) => { });     // error occurred
stream.on('readable', () => { });     // data ready to read manually

// Writable stream events:
stream.on('drain', () => { });        // buffer emptied, safe to write
stream.on('finish', () => { });       // all data flushed
stream.on('error', (err) => { });     // error

// Readable modes:
stream.pause();   // paused mode — must call .read() manually
stream.resume();  // flowing mode — 'data' events fire automatically
```

---

**Q31. How do you handle large file uploads in Node.js?**

```js
// Never load entire upload into memory!
const multer = require('multer');

// Stream to disk (memory efficient)
const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, './uploads'),
  filename: (req, file, cb) => cb(null, `${Date.now()}-${file.originalname}`)
});

const upload = multer({ storage, limits: { fileSize: 100 * 1024 * 1024 } }); // 100MB
app.post('/upload', upload.single('file'), (req, res) => res.json({ path: req.file.path }));

// Stream directly to S3 (never touches disk)
app.post('/upload-s3', (req, res) => {
  const upload = s3.upload({
    Bucket: 'my-bucket',
    Key: `uploads/${Date.now()}`,
    Body: req,  // req IS a readable stream!
    ContentType: req.headers['content-type']
  });
  upload.send((err, data) => res.json(data));
});
```

---

**Q32. Difference between fs.stat and fs.access?**

```js
const fs = require('fs/promises');

// fs.access — check if file is accessible (exists + permissions)
try {
  await fs.access('file.txt', fs.constants.R_OK);
  console.log('readable');
} catch { console.log('not readable or not found'); }

// fs.stat — get file metadata
const stats = await fs.stat('file.txt');
stats.size;          // file size in bytes
stats.mtime;         // last modified date
stats.isFile();      // true
stats.isDirectory(); // false
```

---

## Section 4: Child Processes (Q33–44)

**Q33. What is child_process and why do we need it?**

Node.js is single-threaded. What if you need to:
- Run a shell command (git, ffmpeg, python)
- Do heavy CPU work without blocking the event loop
- Run multiple Node.js scripts in parallel

That's what `child_process` is for. It lets you spawn SEPARATE OS processes.

```js
const { exec, execFile, spawn, fork } = require('child_process');

// exec     — shell command, buffer output (small output)
// execFile — run executable directly, safer (no shell injection)
// spawn    — stream output (large output / real-time)
// fork     — special spawn for Node.js scripts (has IPC channel)
```

---

**Q34. What is exec? When to use it?**

```js
const { exec } = require('child_process');
const { promisify } = require('util');
const execAsync = promisify(exec);

// Promise style
const { stdout } = await execAsync('git log --oneline -10');
console.log(stdout);

// DANGER — shell injection! Never pass user input to exec directly
const userInput = 'file.txt; rm -rf /';
exec(`cat ${userInput}`); // ATTACK! runs both cat AND rm -rf /!

// SAFE — use execFile (no shell interpretation)
execFile('cat', [userInput], (err, stdout) => {
  // userInput treated as ONE filename argument, not shell command
});
```

---

**Q35. What is spawn? How is it different from exec?**

```js
const { spawn } = require('child_process');

// spawn streams output in real-time — perfect for long-running commands
const ffmpeg = spawn('ffmpeg', ['-i', 'input.mp4', '-vcodec', 'libx264', 'output.mp4']);

ffmpeg.stdout.on('data', (data) => process.stdout.write(data));
ffmpeg.stderr.on('data', (data) => process.stderr.write(data)); // ffmpeg progress → stderr
ffmpeg.on('close', (code) => console.log(`ffmpeg exited: ${code}`));
ffmpeg.kill('SIGTERM'); // kill if needed

// exec vs spawn:
// exec:  buffers ALL output → bad for large output (memory!)
// spawn: streams output → good for large/real-time output
// exec:  goes through shell → supports pipes, redirects
// spawn: no shell by default → safer, faster
```

---

**Q36. What is fork? How is it different from spawn?**

fork is specifically for spawning another Node.js script. It creates an IPC channel for parent-child messaging.

```js
// parent.js
const { fork } = require('child_process');
const child = fork('./worker.js');

child.send({ task: 'compute', data: [1, 2, 3, 4, 5] });

child.on('message', (result) => {
  console.log('Result from child:', result);
  child.kill();
});

// worker.js
process.on('message', (msg) => {
  const result = msg.data.reduce((sum, n) => sum + n, 0);
  process.send({ result }); // send back to parent
});

// Use: CPU-heavy computation — fork to child so parent event loop stays free!
```

---

**Q37. What is execFile? When is it safer than exec?**

```js
const { execFile } = require('child_process');

// execFile runs file directly — NO SHELL
// Shell injection attacks don't work
const userFile = 'user_provided_filename.txt';
execFile('cat', [userFile], (error, stdout) => {
  // Even if userFile = 'file; rm -rf /', treated as ONE filename arg
});

// exec vs execFile:
// exec('cat file.txt')       — runs: sh -c 'cat file.txt'
// execFile('cat', ['file'])  — runs: cat file  (no shell)

// When user input involved → always use execFile or spawn
// When you need shell features (pipes, &&) → exec (sanitize input!)
```

---

**Q38. How do you run Python script from Node.js?**

```js
const { spawn } = require('child_process');

function runPython(scriptPath, args = []) {
  return new Promise((resolve, reject) => {
    const python = spawn('python3', [scriptPath, ...args]);
    let result = '';
    let error = '';
    python.stdout.on('data', data => result += data.toString());
    python.stderr.on('data', data => error += data.toString());
    python.on('close', code => {
      if (code !== 0) reject(new Error(error));
      else resolve(result.trim());
    });
  });
}

const output = await runPython('./ml_model.py', ['--input', 'data.json']);
const prediction = JSON.parse(output);

// Two-way communication via stdin:
const python = spawn('python3', ['process.py']);
python.stdin.write(JSON.stringify({ data: [1,2,3] }));
python.stdin.end();
python.stdout.on('data', d => console.log('Result:', d.toString()));
```

---

**Q39. What are child process events?**

```js
const child = spawn('node', ['worker.js']);

child.on('spawn', () => console.log('Started'));
child.on('message', (msg) => console.log('From child:', msg)); // fork only
child.on('error', (err) => console.error('Error:', err));
child.on('exit', (code, signal) => {
  // code = 0 means success, non-zero = error
  console.log(`Exited code=${code} signal=${signal}`);
});
child.on('close', (code, signal) => {
  // stdio streams closed (fires after exit)
});
child.on('disconnect', () => {
  // IPC channel closed
});
```

---

**Q40. Difference between process.exit() and process.exitCode?**

```js
// process.exit(code) — immediately terminates
// ALL cleanup is skipped (pending I/O, timers)
process.exit(0);  // success
process.exit(1);  // error

// process.exitCode — set exit code, let process exit naturally
process.exitCode = 1;
// All cleanup WILL run before exit

// Last sync cleanup:
process.on('exit', (code) => {
  // SYNC only! No async here!
  fs.writeFileSync('shutdown.log', `Shutdown at ${new Date()}`);
});

// Async cleanup (for SIGTERM from PM2, Docker, Kubernetes):
process.on('SIGTERM', async () => {
  await db.close();
  await server.close();
  process.exit(0);
});
```

---

**Q41. How do you handle uncaught errors in Node.js?**

```js
process.on('uncaughtException', (err, origin) => {
  console.error('UNCAUGHT EXCEPTION:', err);
  fs.writeFileSync('crash.log', err.stack);
  process.exit(1); // must exit — app is in unknown state!
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('UNHANDLED REJECTION:', reason);
  // Node 15+: crashes process by default!
  // Node 14-: just a warning
});

// Best practice:
// - Use PM2 to auto-restart on crash
// - Always handle errors in async code
// - Use express error middleware
// - These handlers are LAST RESORT, not primary error handling
```

---

**Q42. What is process.env? How to use it properly?**

```js
// process.env — all environment variables as object
const port = process.env.PORT || 3000;
const nodeEnv = process.env.NODE_ENV; // 'development', 'production', 'test'

// Validate required vars at startup — fail fast!
const required = ['DATABASE_URL', 'JWT_SECRET', 'REDIS_URL'];
for (const key of required) {
  if (!process.env[key]) throw new Error(`Missing required env var: ${key}`);
}

// dotenv — load .env file into process.env
require('dotenv').config();
// .env file:
// DATABASE_URL=mongodb://localhost:27017/mydb
// JWT_SECRET=supersecret123

// NEVER commit .env to git!
// Add .env to .gitignore
// Commit .env.example with placeholder values
```

---

**Q43. What is the process object?**

```js
process.pid          // current process ID
process.platform     // 'linux', 'darwin', 'win32'
process.version      // 'v20.9.0'
process.env          // environment variables
process.argv         // command line arguments
process.cwd()        // current working directory
process.memoryUsage() // { rss, heapTotal, heapUsed, external }
process.cpuUsage()   // { user, system } microseconds
process.uptime()     // seconds since process started
process.hrtime.bigint() // nanoseconds (high precision timing)

// argv example:
// node app.js --port 3000 --env production
process.argv // ['node', '/path/app.js', '--port', '3000', '--env', 'production']

process.stdout.write('hello without newline');
process.stdin.on('data', data => console.log(data.toString()));
```

---

**Q44. How do you pass data between parent and child processes efficiently?**

```js
// Small data: JSON messages via IPC (fork)
child.send({ type: 'TASK', data: smallData });

// Large data: SharedArrayBuffer with Worker Threads
const { Worker } = require('worker_threads');
const sharedBuffer = new SharedArrayBuffer(1024 * 1024); // 1MB shared RAM
const worker = new Worker(__filename, { workerData: { sharedBuffer } });
const arr = new Int32Array(sharedBuffer);
arr[0] = 42; // worker can read this immediately — zero copy!

// Streams for large data via stdin/stdout
const child = spawn('node', ['process.js']);
largeReadableStream.pipe(child.stdin);   // send large input
child.stdout.pipe(largeWritableStream);  // receive large output
```

## Section 5: Worker Threads (Q45–52)

**Q45. What are Worker Threads? How are they different from child processes?**

Worker Threads are lighter than child processes. They share memory with the main thread.

```
                    Child Process       Worker Thread
Memory:             Separate            Can share (SharedArrayBuffer)
Startup time:       Slower (~30ms)      Faster (~1ms)
Communication:      IPC (serialized)    Shared memory + messages
Use case:           Any program         Node.js CPU tasks only
Isolation:          Full                Shares Node.js instance
```

```js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename, {
    workerData: { numbers: [1,2,3,4,5,6,7,8,9,10] }
  });
  worker.on('message', result => console.log('Sum:', result));
  worker.on('error', err => console.error(err));
} else {
  const sum = workerData.numbers.reduce((a, b) => a + b, 0);
  parentPort.postMessage(sum);
}
```

---

**Q46. When should you use Worker Threads vs child_process?**

```
Use Worker Threads when:
- CPU-heavy Node.js work (image processing, parsing huge JSON, encryption)
- Need shared memory (large datasets)
- Same Node.js codebase
- Frequent communication needed (low overhead)

Use child_process when:
- Running non-Node programs (Python, shell scripts, ffmpeg, binaries)
- Need complete process isolation
- Running untrusted code
- One-off commands (git, ls, etc.)

Use cluster when:
- Want multiple instances of your HTTP server
- Need to use all CPU cores for network handling (scale Express)

Real examples:
Worker Threads: resize 1000 images, process large CSV, parse large JSON
child_process:  call Python ML model, run ffmpeg, execute shell commands
cluster:        scale Express API to use all 8 CPU cores
```

---

**Q47. What is transferList in Worker Threads?**

```js
// Normally postMessage COPIES data (serialize → send → deserialize)
// For large ArrayBuffers, this is slow and uses double memory

const buffer = new ArrayBuffer(1024 * 1024 * 100); // 100MB

// Without transfer: copies 100MB (slow, double memory used)
worker.postMessage({ data: buffer });

// With transfer: moves ownership, zero-copy!
worker.postMessage({ data: buffer }, [buffer]);
// After this, buffer is empty in main thread — ownership transferred!

// Types that can be transferred:
// ArrayBuffer, SharedArrayBuffer, MessagePort, ReadableStream, WritableStream
```

---

**Q48. What is SharedArrayBuffer?**

```js
// SharedArrayBuffer — memory shared between threads without copying
// All threads read/write the SAME memory location

if (isMainThread) {
  const sharedBuffer = new SharedArrayBuffer(4); // 4 bytes
  const arr = new Int32Array(sharedBuffer);
  arr[0] = 0; // initial counter = 0
  
  for (let i = 0; i < 4; i++) {
    new Worker(__filename, { workerData: { sharedBuffer } });
  }
  
  setTimeout(() => console.log('Counter:', arr[0]), 1000);
} else {
  const arr = new Int32Array(workerData.sharedBuffer);
  for (let i = 0; i < 1000; i++) {
    Atomics.add(arr, 0, 1); // thread-safe increment
  }
}
// Counter: 4000 (4 workers × 1000 increments)
```

---

**Q49. What is Atomics? Why needed with SharedArrayBuffer?**

```js
// Problem without Atomics: race conditions
// Thread 1: read arr[0]=5, compute 5+1=6, write 6
// Thread 2: read arr[0]=5 (SAME TIME!), compute 5+1=6, write 6
// Result: 6 (should be 7!) — lost update!

// Atomics.add — thread-safe, cannot be interrupted
Atomics.add(arr, 0, 1);    // atomic increment
Atomics.sub(arr, 0, 1);    // atomic decrement
Atomics.exchange(arr, 0, newValue); // atomic swap
Atomics.compareExchange(arr, 0, expected, newValue); // CAS

// Wait/notify (mutex-like)
Atomics.wait(arr, 0, 0);   // sleep until arr[0] !== 0
Atomics.notify(arr, 0, 1); // wake up 1 waiting thread
```

---

**Q50. When should you use Worker Threads for CPU tasks?**

```js
// WITHOUT worker threads — blocks event loop!
app.get('/heavy', (req, res) => {
  const result = expensiveComputation(); // blocks for 5 seconds!
  res.json(result); // ALL other requests wait during these 5 seconds
});

// WITH worker threads — event loop stays free
app.get('/heavy', (req, res) => {
  const worker = new Worker('./compute.js', {
    workerData: { input: req.query.data }
  });
  worker.on('message', result => res.json(result));
  worker.on('error', err => res.status(500).json({ error: err.message }));
});

// Better: worker pool (reuse workers — avoid create/destroy overhead per request)
const Piscina = require('piscina');
const pool = new Piscina({ filename: './compute.js', maxThreads: 4 });

app.get('/heavy', async (req, res) => {
  const result = await pool.run({ input: req.query.data });
  res.json(result);
});
```

---

**Q51. What is the cluster module?**

Cluster lets you run multiple instances of your Node.js server to use ALL CPU cores.

```js
const cluster = require('cluster');
const os = require('os');
const numCPUs = os.cpus().length; // e.g. 8

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} running`);
  
  for (let i = 0; i < numCPUs; i++) cluster.fork();
  
  // Auto-restart crashed workers
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died. Restarting...`);
    cluster.fork();
  });
} else {
  // Each worker runs its own Express server on same port
  const app = express();
  app.get('/', (req, res) => res.json({ workerPid: process.pid }));
  app.listen(3000, () => console.log(`Worker ${process.pid} started`));
}
// OS load balances incoming connections across all workers
// This is what PM2 cluster mode does under the hood!
```

---

**Q52. Worker Threads vs Cluster — which one?**

```
Cluster:
- Multiple NODE.JS PROCESSES (each with own memory)
- Each runs a full copy of your HTTP server
- OS distributes network connections between processes
- Best for: scaling HTTP server throughput across CPU cores
- Example: 8 cores → 8 server processes → 8x throughput

Worker Threads:
- Multiple THREADS inside ONE process (shared memory)
- No HTTP server duplication
- Best for: one-off CPU tasks while keeping main thread free
- Example: image resize request → worker thread handles it

In practice: use BOTH
- PM2 cluster mode for HTTP throughput
- Worker threads for CPU-intensive tasks within each cluster worker
```

---

## Section 6: PM2 — Process Manager (Q53–64)

**Q53. What is PM2? Why do we need it?**

PM2 = Production Process Manager for Node.js. It's the guardian of your app.

```
WITHOUT PM2:
- App crashes → server DOWN until you manually restart
- Server reboots → must manually start app
- Can't use multiple CPU cores
- No log management
- No monitoring

WITH PM2:
- App crashes → PM2 auto-restarts immediately
- Server reboots → PM2 auto-starts your app
- One command uses all CPU cores
- Centralised log management
- Built-in monitoring dashboard
```

```bash
npm install -g pm2
pm2 start app.js --name "my-api"
pm2 list          # see all running apps
pm2 stop my-api
pm2 restart my-api
pm2 delete my-api
```

---

**Q54. What is ecosystem.config.js?**

```js
// ecosystem.config.js — PM2 config file (like docker-compose for PM2)
module.exports = {
  apps: [{
    name: 'dawa-saathi-api',
    script: './src/index.js',
    instances: 'max',           // use all CPU cores
    exec_mode: 'cluster',       // load balanced cluster mode
    watch: false,               // don't watch in production
    max_memory_restart: '500M', // restart if RAM > 500MB
    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 80
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    time: true,                 // timestamps in logs
    autorestart: true,          // restart on crash
    max_restarts: 10,           // give up after 10 restarts
    restart_delay: 4000,        // wait 4s between restarts
  }]
};
```

```bash
pm2 start ecosystem.config.js
pm2 start ecosystem.config.js --env production
```

---

**Q55. Difference between cluster mode and fork mode in PM2?**

```bash
# Fork mode (default) — ONE instance
pm2 start app.js --name api
# Single process. Good for development.

# Cluster mode — MULTIPLE instances (one per CPU core)
pm2 start app.js -i max --name api  # max = all CPU cores
pm2 start app.js -i 4 --name api    # exactly 4 instances

# PM2 load balances incoming requests across all instances
# All instances share the same PORT
# If one crashes, others keep running, PM2 restarts the crashed one

# Performance difference:
# Fork mode:   1 process × 1 core = X requests/sec
# Cluster mode: 8 processes × 8 cores = ~8X requests/sec
```

---

**Q56. How do you do zero-downtime deployment with PM2?**

```bash
# Graceful reload — NO downtime (use in production)
pm2 reload app

# How pm2 reload works:
# 1. Send SIGINT to one worker
# 2. Worker finishes current requests
# 3. Worker shuts down gracefully
# 4. PM2 starts new worker with new code
# 5. New worker ready → PM2 routes traffic to it
# 6. Repeat for all workers
# → Zero requests dropped!

# Hard restart (drops ongoing requests — avoid in production)
pm2 restart app
```

```js
// Your app MUST handle SIGINT for graceful shutdown:
process.on('SIGINT', async () => {
  server.close(() => {
    db.close(() => {
      process.exit(0);
    });
  });
});
```

---

**Q57. How do you use PM2 logs?**

```bash
pm2 logs              # all apps live
pm2 logs my-api       # specific app
pm2 logs my-api --lines 100  # last 100 lines
pm2 flush             # clear all logs

# Log rotation
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 30
pm2 set pm2-logrotate:compress true

# Default log locations:
# ~/.pm2/logs/app-out.log
# ~/.pm2/logs/app-error.log
```

---

**Q58. How do you monitor apps with PM2?**

```bash
pm2 monit     # real-time terminal dashboard (CPU, memory, restarts, logs)
pm2 list      # quick status of all apps
pm2 show api  # detailed info: pid, uptime, restarts, env vars, log paths

# What pm2 list shows:
# NAME    MODE      PID    STATUS  ↺    CPU  MEM
# api     cluster   1234   online  0    0.5% 45MB
# api     cluster   1235   online  0    0.3% 43MB
```

---

**Q59. How do you make PM2 start on system boot?**

```bash
pm2 startup                    # generates startup command for your OS
# Copy and run the output command, e.g.:
sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u ubuntu --hp /home/ubuntu

pm2 save                       # save current running processes to dump file

# After reboot: PM2 auto-starts saved processes
pm2 list                       # verify apps running

pm2 unstartup systemd          # remove startup
```

---

**Q60. PM2 environment variables?**

```js
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'api',
    script: 'app.js',
    env: {                     // default
      NODE_ENV: 'development',
      PORT: 3000,
    },
    env_staging: {
      NODE_ENV: 'staging',
      PORT: 3001,
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 80,
    }
  }]
};
```

```bash
pm2 start ecosystem.config.js --env production
pm2 restart ecosystem.config.js --env staging
pm2 show api | grep NODE_ENV   # check current env
```

---

**Q61. How does PM2 handle crashes and restarts?**

```js
// ecosystem.config.js
{
  autorestart: true,              // default true
  max_restarts: 10,               // give up after 10 restarts
  min_uptime: '10s',              // stable if up for 10s
  restart_delay: 4000,            // wait 4s between restarts
  exp_backoff_restart_delay: 100, // exponential backoff
  max_memory_restart: '500M',     // restart if RAM > 500MB
  cron_restart: '0 2 * * *',      // restart every night at 2am (clear memory leaks)
}
```

```bash
pm2 list  # check ↺ column for restart count
```

---

**Q62. PM2 vs systemd?**

```
PM2:
- Easy to use, no sysadmin knowledge needed
- Built-in cluster mode
- Built-in monitoring
- Log management with rotation
- Zero-downtime reload
- Node-specific features

systemd:
- OS-level service manager (any program)
- More reliable on long-running servers
- Better OS tool integration (journalctl)
- No built-in clustering
- More configuration

In practice:
- Single VM/EC2 → PM2
- Docker + Kubernetes → no PM2 needed (k8s handles restarts)
```

---

**Q63. How do you do graceful shutdown in Node.js?**

```js
const server = require('http').createServer(app);

async function gracefulShutdown(signal) {
  console.log(`Received ${signal}. Shutting down...`);

  server.close(async () => {
    await mongoose.connection.close();
    await redis.quit();
    await queue.close();
    process.exit(0);
  });

  // Force kill after 30 seconds
  setTimeout(() => {
    console.error('Forced shutdown — timeout');
    process.exit(1);
  }, 30000);
}

process.on('SIGTERM', () => gracefulShutdown('SIGTERM')); // PM2/k8s sends this
process.on('SIGINT', () => gracefulShutdown('SIGINT'));   // Ctrl+C
```

---

**Q64. PM2 commands cheat sheet.**

```bash
# Starting
pm2 start app.js --name api
pm2 start app.js -i max           # cluster mode
pm2 start ecosystem.config.js --env production

# Managing
pm2 list                          # show all
pm2 stop api / restart api / delete api
pm2 reload api                    # zero-downtime restart
pm2 stop all / restart all

# Logs
pm2 logs [app] [--lines 50]
pm2 flush                         # clear logs

# Monitoring
pm2 monit                         # live dashboard
pm2 show api                      # app details

# Startup
pm2 startup → pm2 save            # auto-start on boot

# Updates
pm2 update                        # update PM2 in-place
```

---

## Section 7: HTTP, Networking & Internals (Q65–80)

**Q65. How does Node.js handle HTTP requests?**

```js
const http = require('http');

const server = http.createServer((req, res) => {
  // req = IncomingMessage (Readable Stream)
  // res = ServerResponse (Writable Stream)
  
  console.log(req.method);   // 'GET', 'POST'
  console.log(req.url);      // '/users?page=1'
  console.log(req.headers);  // { 'content-type': 'application/json', ... }
  
  // Reading body (it's a stream!)
  let body = '';
  req.on('data', chunk => body += chunk.toString());
  req.on('end', () => {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ received: JSON.parse(body) }));
  });
});
server.listen(3000);

// Flow:
// 1. OS receives TCP connection on port 3000
// 2. Node.js http module parses HTTP headers
// 3. Creates IncomingMessage + ServerResponse
// 4. Emits 'request' → your callback
// 5. You read body stream, build response, call res.end()
// 6. Node sends HTTP response over TCP
```

---

**Q66. What is keep-alive in HTTP?**

```js
// HTTP/1.0: each request opens new TCP connection (3-way handshake each time — slow)
// HTTP/1.1: keep-alive by default — reuse TCP connection for multiple requests

const http = require('http');
const agent = new http.Agent({
  keepAlive: true,       // reuse connections
  maxSockets: 10,        // max 10 connections per host
  maxFreeSockets: 5,     // keep 5 idle connections in pool
  timeout: 60000         // idle timeout
});

http.get({ hostname: 'api.example.com', agent }, (res) => {
  // connection reused from pool — faster!
});
// In production: Nginx handles keep-alive between client and Nginx
// Nginx → Node.js also uses keep-alive internally
```

---

**Q67. Difference between http and https modules?**

```js
// http — plain text, port 80
const server = require('http').createServer(app);
server.listen(80);

// https — TLS encrypted, port 443
const https = require('https');
const server = https.createServer({
  key: fs.readFileSync('/etc/ssl/private/server.key'),
  cert: fs.readFileSync('/etc/ssl/certs/server.crt'),
}, app);
server.listen(443);

// In production: Nginx handles SSL termination
// Clients → HTTPS to Nginx → HTTP to Node.js (internal, no encryption needed)
// So your Node app can just use http module!
```

---

**Q68. What is CORS and how to handle it?**

```js
// CORS = Cross-Origin Resource Sharing
// Browser blocks requests from frontend.com to api.different.com
// Server must explicitly allow it via headers

const cors = require('cors');

// Allow all (dangerous in production):
app.use(cors());

// Allow specific origins (production):
app.use(cors({
  origin: ['https://myapp.com', 'https://www.myapp.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,   // allow cookies
  maxAge: 86400        // preflight cache for 24 hours
}));

// Handle preflight (browser sends OPTIONS first):
app.options('*', cors());

// What Node actually sends:
// Access-Control-Allow-Origin: https://myapp.com
// Access-Control-Allow-Methods: GET, POST, ...
// Access-Control-Allow-Headers: Content-Type, Authorization
```

---

**Q69. How do you make HTTP requests from Node.js?**

```js
// Built-in fetch (Node 18+) — best for simple requests
const res = await fetch('https://api.example.com/users');
const data = await res.json();

// axios — most popular, works everywhere
const axios = require('axios');
const { data } = await axios.get('https://api.example.com/users');
await axios.post('/api/users', { name: 'Suraj' }, {
  headers: { Authorization: `Bearer ${token}` }
});

// Built-in https module (no dependencies)
const https = require('https');
function get(url) {
  return new Promise((resolve, reject) => {
    https.get(url, (res) => {
      let data = '';
      res.on('data', chunk => data += chunk);
      res.on('end', () => resolve(JSON.parse(data)));
    }).on('error', reject);
  });
}
```

---

**Q70. What is the zlib module?**

```js
const zlib = require('zlib');
const { pipeline } = require('stream/promises');

// Compress file
await pipeline(
  fs.createReadStream('file.txt'),
  zlib.createGzip(),
  fs.createWriteStream('file.txt.gz')
);

// Decompress
await pipeline(
  fs.createReadStream('file.txt.gz'),
  zlib.createGunzip(),
  fs.createWriteStream('file.txt')
);

// HTTP gzip responses (Express)
const compression = require('compression');
app.use(compression());
// Reduces response size by 60-80%! Must-have in production.
```

---

**Q71. What is the net module?**

```js
const net = require('net');

// net = raw TCP (below HTTP)
// Used for: game servers, custom protocols, microservice communication

const server = net.createServer((socket) => {
  socket.on('data', (data) => {
    socket.write('Echo: ' + data);
  });
  socket.on('end', () => console.log('Client disconnected'));
});
server.listen(8080);

// Redis, MySQL, MongoDB — all built on net.createConnection internally!
// require('mysql2').connect() creates a net.Socket to MySQL port 3306
// and speaks the MySQL protocol over raw TCP
```

---

**Q72. What is the DNS module?**

```js
const dns = require('dns');

// dns.lookup — uses OS resolver (uses thread pool)
dns.lookup('google.com', (err, address) => {
  console.log(address); // '142.250.195.14'
});

// dns.resolve — Node's own DNS resolver (not OS)
dns.resolve4('google.com', (err, addresses) => {
  console.log(addresses); // ['142.250.195.14', ...]
});

// Reverse lookup
dns.reverse('8.8.8.8', (err, hostnames) => {
  console.log(hostnames); // ['dns.google']
});

// Promise API
const { promises: dnsPromises } = require('dns');
const addresses = await dnsPromises.resolve4('google.com');
```

---

**Q73. What is HTTP/2 in Node.js?**

```js
// HTTP/2 advantages:
// 1. Multiplexing — multiple requests over ONE TCP connection
// 2. Header compression (HPACK) — smaller headers
// 3. Server push — send resources before client asks
// 4. Binary protocol — faster than text HTTP/1.1

const http2 = require('http2');
const server = http2.createSecureServer({ key, cert });

server.on('stream', (stream, headers) => {
  // Server push — send CSS before browser asks
  stream.pushStream({ ':path': '/style.css' }, (err, pushStream) => {
    pushStream.respond({ ':status': 200 });
    pushStream.end('body { color: red; }');
  });
  stream.respond({ ':status': 200 });
  stream.end('<html>Hello HTTP/2!</html>');
});

// In production: Nginx handles HTTP/2, proxies HTTP/1.1 to Node
```

---

**Q74. What is the REPL?**

```bash
# REPL = Read-Evaluate-Print Loop — interactive Node.js shell
node        # start REPL
> 1 + 1     # 2
> const x = 10
> x * 2     # 20

# REPL commands:
.break    — exit multiline input
.exit     — exit (or Ctrl+D)
.help     — show commands
.editor   — multiline editor mode

# Custom REPL for debugging/admin:
const repl = require('repl');
const r = repl.start({ prompt: 'myapp> ' });
r.context.db = database;     // inject db — now you can query DB interactively!
r.context.models = models;
```

---

**Q75. How do you debug Node.js?**

```bash
# Method 1: console.log (everyone's default)
console.dir(obj, { depth: null }); # deep print
console.time('label'); ... console.timeEnd('label'); # timing
console.trace(); # print call stack

# Method 2: Chrome DevTools
node --inspect app.js
# Open chrome://inspect → click "inspect"
# Full Chrome DevTools for Node.js! Breakpoints, watch variables, call stack

node --inspect-brk app.js  # pause at first line

# Method 3: VS Code debugger
# .vscode/launch.json → F5 to run with breakpoints

# Method 4: DEBUG environment variable
DEBUG=express:* node app.js  # all Express internal logs
DEBUG=* node app.js           # ALL debug logs from any module using 'debug' package
```

---

**Q76. What is V8 engine and how does it optimise JS?**

```
V8 compilation pipeline:

JavaScript source
      → Parser (builds AST — Abstract Syntax Tree)
      → Ignition (interpreter — runs bytecode)
      → TurboFan (JIT compiler — compiles HOT functions to machine code)
      → Deoptimisation (if type assumptions are wrong, back to Ignition)
```

```js
// V8 optimisation tips:

// 1. Consistent object shapes (hidden classes)
// BAD — shape changes conditionally
function bad(a) {
  const obj = {};
  if (a) obj.x = 1; // sometimes has x, sometimes not
  return obj;
}

// GOOD — same shape always
function good(a) {
  return { x: a ? 1 : undefined }; // V8 can optimise this
}

// 2. Avoid delete (changes object shape)
// 3. Use TypedArrays for numeric data
// 4. Avoid try/catch in hot loops (prevents optimisation)
```

---

**Q77. What is Node.js garbage collection?**

```js
// V8 uses generational GC:
// New Space (Young gen) — newly created objects, ~1-8MB, fast GC (Scavenge)
// Old Space (Old gen)   — survived multiple GCs, hundreds of MB, slower GC

const mem = process.memoryUsage();
// {
//   rss: 45MB,       // total process memory (physical RAM)
//   heapTotal: 30MB, // total heap allocated by V8
//   heapUsed: 25MB,  // heap actually in use
//   external: 1MB,   // C++ objects (Buffer, etc.)
// }

// Common memory leaks:
// 1. Global variables accumulating data
// 2. Event listeners never removed
// 3. Closures holding large object references
// 4. setInterval callbacks never cleared
// 5. Cached data with no expiry
```

---

**Q78. How do you profile Node.js?**

```bash
# Built-in profiler
node --prof app.js       # generate isolate-*.log
node --prof-process isolate-*.log > profile.txt

# clinic.js (best)
npm install -g clinic
clinic doctor -- node app.js     # overall diagnostics
clinic flame -- node app.js      # flame graph (slow functions)
clinic bubbleprof -- node app.js # async profiling

# Reading flame graphs:
# X-axis = time spent (wider bar = more time)
# Y-axis = call stack (bottom = main, top = what's being called)
# Wide bars at the top = HOT functions to optimise
```

---

**Q79. What are Node.js performance best practices?**

```js
// 1. Never block the event loop
// BAD: synchronous heavy work in request handlers
// GOOD: worker threads for CPU tasks, async for I/O

// 2. Use streams for large data
// BAD: fs.readFile('1GB-file.csv')
// GOOD: fs.createReadStream('1GB-file.csv').pipe(...)

// 3. Use compression
app.use(require('compression')());

// 4. Cache expensive operations (Redis or in-memory)
const cache = new Map();
async function getCached(key) {
  if (cache.has(key)) return cache.get(key);
  const data = await fetchFromDB(key);
  cache.set(key, data);
  return data;
}

// 5. Use cluster mode (PM2) for multi-core
// pm2 start app.js -i max

// 6. Set NODE_ENV=production
// Express disables debug features, enables caching

// 7. Use connection pooling for DB
// Never open/close connection per request!

// 8. Keep Node.js version updated
// Newer versions = better V8 = faster JS execution
```

---

**Q80. What is the difference between require and dynamic import()?**

```js
// require() — synchronous, loads at runtime
const module = require('./module');

// import (static) — ES Modules, parsed at file load time
import { fn } from './module.js';

// import() dynamic — asynchronous, lazy loading, conditional
const module = await import('./heavy-module.js'); // loads ONLY when called

// Use cases for dynamic import:
// 1. Lazy loading — load heavy library only when needed
async function loadChart() {
  if (needsChart) {
    const { Chart } = await import('chart.js');
    return new Chart(...);
  }
}

// 2. Conditional imports based on runtime values
const locale = await import(`./locales/${language}.js`);

// 3. Code splitting in webpack/vite
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));
```

---

## Section 8: Advanced & Tricky Questions (Q81–100)

**Q81. What is the difference between process.nextTick and setImmediate? (Deep)**

```js
process.nextTick(() => console.log('nextTick'));
setImmediate(() => console.log('setImmediate'));
Promise.resolve().then(() => console.log('promise'));

// Output: nextTick, promise, setImmediate

// process.nextTick:
// - NOT part of event loop
// - Runs after current operation, before ANYTHING else
// - Even before Promise callbacks!
// - Queue drains completely before event loop continues
// - Danger: recursive nextTick can starve I/O!

// setImmediate:
// - Runs in CHECK phase (after I/O)
// - Yields to event loop between calls
// - Safe for recursive use

// When to use:
// process.nextTick — emit events after constructor, error propagation patterns
// setImmediate — break up CPU work without blocking I/O
```

---

**Q82. Why can recursive process.nextTick starve I/O?**

```js
// nextTick queue drains COMPLETELY before event loop moves
// If nextTick keeps adding more nextTick → event loop NEVER progresses!

function recursive() {
  process.nextTick(recursive); // adds itself infinitely
}
recursive();
// setTimeout callback NEVER runs — completely starved!
// I/O events never fire!

// Fix: use setImmediate instead
function recursive() {
  setImmediate(recursive); // yields to event loop between calls
}
// setImmediate runs in CHECK phase
// Event loop still processes I/O between calls
```

---

**Q83. What is AsyncLocalStorage?**

```js
const { AsyncLocalStorage } = require('async_hooks');

// AsyncLocalStorage = request-scoped storage
// Like "thread-local storage" but for async operations
// Data in one request is NOT visible to other requests

const requestContext = new AsyncLocalStorage();

// Middleware: start a context for this request
app.use((req, res, next) => {
  requestContext.run({
    requestId: crypto.randomUUID(),
    userId: req.user?.id
  }, next);
});

// Anywhere in your code (even deeply nested async functions):
function logQuery(query) {
  const ctx = requestContext.getStore();
  console.log(`[${ctx?.requestId}] Query: ${query}`);
}

async function fetchUser(id) {
  const ctx = requestContext.getStore();
  // ctx.requestId available here WITHOUT passing it as a parameter!
  return db.query('SELECT * FROM users WHERE id = $1', [id]);
}
// This is how pino, winston implement per-request tracing
```

---

**Q84. How does Express middleware work under the hood?**

```js
// Express is basically this:
function createApp() {
  const stack = [];

  function use(fn) { stack.push(fn); }

  function handle(req, res) {
    let index = 0;
    function next(err) {
      const fn = stack[index++];
      if (!fn) return; // end of stack — request hangs if no response sent!
      try { fn(req, res, next); }
      catch(e) { next(e); }
    }
    next();
  }

  return { use, handle };
}

// That's it! Express is a sophisticated version of this.
// app.use(fn) adds to stack
// Incoming request triggers handle() which runs stack in order
// next() moves to next middleware
// Not calling next() AND not sending response = hanging request!
```

---

**Q85. What is the correct order of middleware in Express?**

```js
app.use(logger);           // 1. logging — runs for everything
app.use(express.json());   // 2. parse JSON body
app.use('/api', cors);     // 3. CORS for API routes
app.use(rateLimit);        // 4. rate limiting

app.get('/api/users', auth, validate, handler); // 5, 6, 7 — specific route

app.use(notFoundHandler);  // runs if no route matched (404)
app.use(errorHandler);     // error handler — MUST be LAST, MUST have 4 params!

// Common mistake: error handler NOT last
app.use(errorHandler);         // WRONG — registered before routes!
app.get('/users', getUsers);   // never reaches error handler properly

// Error handler signature: (err, req, res, next)
// If only 3 params, Express doesn't treat it as error handler!
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});
```

---

**Q86. What is the difference between app.listen and http.createServer?**

```js
const app = express();

// app.listen — simple, common
const server = app.listen(3000);
// Internally: http.createServer(app).listen(3000)

// http.createServer — when you need the server instance
const http = require('http');
const server = http.createServer(app);
server.listen(3000);

// Why use createServer?
// 1. WebSocket (attach ws to same server)
const wss = new WebSocket.Server({ server });

// 2. HTTPS
const httpsServer = https.createServer({ key, cert }, app);
httpsServer.listen(443);

// 3. Graceful shutdown
process.on('SIGTERM', () => server.close(() => process.exit(0)));

// app itself is just a function: (req, res) => {}
typeof app; // 'function'
```

---

**Q87. How do you implement rate limiting in Node.js?**

```js
// Simple in-memory (doesn't work across multiple instances!):
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  message: 'Too many requests',
});
app.use('/api/', limiter);

// Redis-based (works across ALL instances — production):
const RedisStore = require('rate-limit-redis');
const redis = require('ioredis');
const client = new redis();

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  store: new RedisStore({
    sendCommand: (...args) => client.call(...args),
  }),
});
// Now works across all PM2 workers!
```

---

**Q88. How do you implement caching in Node.js?**

```js
// Level 1: In-memory (single instance, fastest)
const cache = new Map();
async function getCached(key, fetcher, ttl = 60000) {
  const cached = cache.get(key);
  if (cached && Date.now() < cached.expiresAt) return cached.value;
  const value = await fetcher();
  cache.set(key, { value, expiresAt: Date.now() + ttl });
  return value;
}

// Level 2: Redis (distributed, survives restarts)
async function getCachedRedis(key, fetcher, ttlSeconds = 60) {
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);
  const value = await fetcher();
  await redis.setex(key, ttlSeconds, JSON.stringify(value));
  return value;
}

// Usage
app.get('/users/:id', async (req, res) => {
  const user = await getCachedRedis(
    `user:${req.params.id}`,
    () => db.findUser(req.params.id),
    300 // 5 minutes
  );
  res.json(user);
});
```

---

**Q89. How do you implement WebSockets in Node.js?**

```js
const http = require('http');
const WebSocket = require('ws');
const app = express();
const server = http.createServer(app);
const wss = new WebSocket.Server({ server });
const clients = new Map();

wss.on('connection', (ws, req) => {
  const clientId = crypto.randomUUID();
  clients.set(clientId, ws);

  ws.on('message', (message) => {
    // Broadcast to all connected clients
    clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify({ from: clientId, data: message.toString() }));
      }
    });
  });

  ws.on('close', () => clients.delete(clientId));
  ws.on('error', () => clients.delete(clientId));
  ws.send(JSON.stringify({ type: 'connected', id: clientId }));
});

// Heartbeat — detect dead connections
setInterval(() => {
  clients.forEach((ws, id) => {
    if (ws.readyState !== WebSocket.OPEN) clients.delete(id);
    else ws.ping();
  });
}, 30000);

server.listen(3000);
```

---

**Q90. How do you handle circular dependencies in CommonJS?**

```js
// a.js
const b = require('./b');
module.exports = { a: 1, useB: () => b.value };

// b.js
const a = require('./a');
// PROBLEM: a.js required b.js, b.js requires a.js back
// Node returns PARTIAL exports of a.js!
// a might be {} at this point!
module.exports = { b: 2, value: a.a }; // a.a = undefined!

// Solution 1: restructure to avoid circular deps
// Solution 2: require inside function (lazy)
// b.js
module.exports = {
  getValue() {
    const a = require('./a'); // loaded lazily, a is complete by now
    return a.a;
  }
};

// Tools to detect circular deps:
// npx madge --circular src/
```

---

**Q91. What is the difference between synchronous and async error handling in Express?**

```js
// Sync error — Express catches automatically
app.get('/sync', (req, res, next) => {
  throw new Error('sync error'); // Express catches this
});

// Async error — MUST pass to next()!
app.get('/async', async (req, res, next) => {
  try {
    const data = await fetchData();
    res.json(data);
  } catch (err) {
    next(err); // MUST call next(err), throw won't work in async!
  }
});

// Cleaner wrapper:
const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/clean', asyncHandler(async (req, res) => {
  const data = await fetchData(); // if throws, auto-passed to error handler
  res.json(data);
}));

// Express 5 (beta): handles async errors automatically
// Express 4 (current): need the wrapper pattern
```

---

**Q92. What is the Node.js module resolution algorithm?**

```
When you require('something'), Node checks in this ORDER:

1. Is it a core module? (fs, http, path, etc.)
   → return immediately

2. Does it start with ./ or ../? (relative path)
   → Try: ./something.js
   → Try: ./something.json
   → Try: ./something.node
   → Try: ./something/index.js
   → Try: ./something/package.json main field

3. Otherwise (package name):
   → Check node_modules/ in current directory
   → Check node_modules/ in parent directory
   → Walk up to filesystem root
   → Check NODE_PATH environment variable
   → Return MODULE_NOT_FOUND error

4. Resolution for scoped packages (@org/pkg):
   → node_modules/@org/pkg/package.json main field
```

```js
// Check where a module resolves to:
require.resolve('express'); // '/node_modules/express/index.js'
require.resolve('./utils'); // '/home/suraj/project/utils.js'
```

---

**Q93. What is the difference between spawn vs execFile vs fork in terms of security?**

```
Most secure → Least secure:
execFile > spawn (shell:false) > fork > spawn (shell:true) > exec

exec:
- Runs: sh -c 'command'
- Shell injection possible with user input
- Convenient for shell features (pipes, redirects)
- AVOID with user input

execFile:
- Runs: /path/to/binary arg1 arg2
- No shell — injection impossible
- Use whenever running external binaries with user data

spawn (shell: false — default):
- Like execFile but streaming
- Safe with user input as args

fork:
- Only for Node.js scripts
- IPC channel available
- Safe (no external binary)

spawn (shell: true):
- Same risk as exec — avoid with user input
```

---

**Q94. What are environment-specific configs in Node.js?**

```js
// config.js
const baseConfig = {
  port: 3000,
  logLevel: 'info',
};

const envConfig = {
  development: {
    db: 'mongodb://localhost/dev',
    logLevel: 'debug',
    corsOrigin: 'http://localhost:3000'
  },
  production: {
    db: process.env.DATABASE_URL,
    logLevel: 'warn',
    corsOrigin: process.env.FRONTEND_URL
  },
  test: {
    db: 'mongodb://localhost/test',
    logLevel: 'error'
  }
};

const env = process.env.NODE_ENV || 'development';
const config = { ...baseConfig, ...envConfig[env] };
module.exports = config;

// Usage:
const { db, port, corsOrigin } = require('./config');
```

---

**Q95. What is connection pooling in Node.js?**

```js
// BAD: create new connection per request
app.get('/users', async (req, res) => {
  const conn = await mysql.createConnection(config); // new connection each time!
  const [rows] = await conn.query('SELECT * FROM users');
  await conn.end();
  res.json(rows);
}); // Creating connection takes ~20-50ms. Wasted on every request!

// GOOD: connection pool — reuse connections
const pool = mysql.createPool({
  host: 'localhost',
  database: 'mydb',
  connectionLimit: 10,  // max 10 simultaneous connections
  waitForConnections: true,
  queueLimit: 0
});

app.get('/users', async (req, res) => {
  // Gets connection from pool (already open, instant!)
  const [rows] = await pool.query('SELECT * FROM users');
  res.json(rows); // connection returned to pool automatically
});

// Mongoose also uses pooling:
mongoose.connect(uri, { maxPoolSize: 10 });
```

---

**Q96. What is the difference between JSON.stringify and fast-json-stringify?**

```js
// JSON.stringify — built-in, flexible, slower
JSON.stringify({ name: 'Suraj', age: 25 }); // works on any object

// fast-json-stringify — 2x faster, requires schema upfront
const fastStringify = require('fast-json-stringify');
const stringify = fastStringify({
  type: 'object',
  properties: {
    name: { type: 'string' },
    age: { type: 'integer' }
  }
});
stringify({ name: 'Suraj', age: 25 }); // 2x faster!

// Why faster? Uses schema to generate optimised serialization code
// No need to dynamically check types at runtime

// Used by: Fastify (HTTP framework), very high throughput APIs
// Not worth it for most apps — premature optimisation

// JSON.stringify limitations (always remember!):
// - functions → omitted
// - undefined → omitted
// - Date → ISO string (loses Date type)
// - Map/Set → {} (empty object!)
// - Circular reference → throws TypeError
```

---

**Q97. What is graceful shutdown and why is it important?**

```
Without graceful shutdown:
- Request in progress → DROPPED mid-way (client gets error)
- DB transactions → ROLLED BACK (data inconsistency possible)
- Job queue → LOST JOBS (messages never processed)
- File writes → CORRUPTED FILES

With graceful shutdown:
1. Stop accepting NEW connections
2. Wait for IN-PROGRESS requests to complete
3. Close DB connections properly
4. Drain job queues
5. Exit cleanly

Triggers for graceful shutdown:
- PM2 reload (sends SIGINT)
- Kubernetes rolling update (sends SIGTERM)
- Docker stop (sends SIGTERM, then SIGKILL after 30s)
- Ctrl+C in terminal (sends SIGINT)
```

```js
// Always implement this pattern in production:
process.on('SIGTERM', gracefulShutdown);
process.on('SIGINT', gracefulShutdown);

async function gracefulShutdown() {
  server.close(async () => {
    await db.close();
    await redis.quit();
    await queue.close();
    process.exit(0);
  });
  setTimeout(() => process.exit(1), 30000); // force after 30s
}
```

---

**Q98. What is the difference between setTimeout and setInterval? Edge cases?**

```js
// setTimeout — run ONCE after delay
const id = setTimeout(() => console.log('once'), 1000);
clearTimeout(id); // cancel

// setInterval — run REPEATEDLY every delay
const id = setInterval(() => console.log('every second'), 1000);
clearInterval(id); // cancel

// EDGE CASE 1: Timer drift with setInterval
// If callback takes 200ms and interval is 1000ms,
// next call happens at 1200ms, not 1000ms — drift accumulates!

// Fix: recursive setTimeout (self-scheduling)
function accurate() {
  const start = Date.now();
  doWork(); // takes variable time
  setTimeout(accurate, 1000 - (Date.now() - start)); // adjust for drift
}

// EDGE CASE 2: setInterval that never clears = memory leak!
// Always store the ID and clearInterval when done

// EDGE CASE 3: Minimum delay
// setTimeout(fn, 0) is NOT 0ms
// Minimum is ~1ms in Node.js, 4ms in browsers after 5 nested calls
// Actual execution depends on event loop state
```

---

**Q99. How do you test Node.js applications?**

```js
const { describe, it, beforeEach, afterEach } = require('@jest/globals');
const request = require('supertest');
const app = require('./app');

// Unit test (pure function)
describe('calculateTax', () => {
  it('should calculate 18% GST correctly', () => {
    expect(calculateTax(100, 0.18)).toBe(18);
  });
});

// Integration test (with DB)
describe('UserService', () => {
  beforeEach(async () => {
    await db.migrate.latest();
  });
  afterEach(async () => {
    await db.migrate.rollback();
  });
  it('should create user', async () => {
    const user = await UserService.create({ name: 'Suraj', email: 'test@test.com' });
    expect(user.id).toBeDefined();
  });
});

// HTTP test (with supertest)
describe('GET /api/users', () => {
  it('should return 401 without token', async () => {
    const res = await request(app).get('/api/users');
    expect(res.status).toBe(401);
  });
  it('should return users with valid token', async () => {
    const res = await request(app)
      .get('/api/users')
      .set('Authorization', `Bearer ${testToken}`);
    expect(res.status).toBe(200);
    expect(Array.isArray(res.body)).toBe(true);
  });
});
```

---

**Q100. How would you architect a production-ready Node.js backend?**

```
Production Architecture (what interviewers want to hear):

Internet → Route 53 (DNS) → CloudFront (CDN for static)
         → Nginx (SSL termination, rate limiting, reverse proxy)
         → PM2 cluster (Node.js × all CPU cores)
         → Express app
              ├── Middleware (CORS, auth, rate limit, logging)
              ├── Routes (thin — just call services)
              ├── Services (business logic)
              └── Data layer
                   ├── PostgreSQL/MySQL (connection pool of 10)
                   ├── Redis (cache, sessions, rate limits, queue backing)
                   └── S3 (files)
         → BullMQ Workers (background jobs, separate processes)
              ├── EmailWorker
              ├── ClaudeAPIWorker (your Dawa Saathi!)
              └── NotificationWorker

Key decisions to explain:
1. PM2 cluster — use all CPU cores for HTTP throughput
2. Redis cache — avoid DB hits for hot data
3. BullMQ — async background jobs, HTTP responds instantly
4. Connection pooling — never open/close DB per request
5. Graceful shutdown — SIGTERM → drain requests → close DB → exit
6. Rate limiting — Redis-based, works across all PM2 workers
7. Nginx — SSL termination, serves static files, protects Node
8. Error handling — try/catch + asyncHandler + global handlers
9. Environment config — .env + dotenv, validated at startup
10. Health check endpoint — /health returns 200 if app is ok (for load balancer)
```

---

## Quick Reference Map

```
Event Loop & Phases:       Q4, Q5, Q6, Q7, Q81, Q82
Single-threaded internals: Q1, Q2, Q3, Q8, Q9, Q11, Q12
Thread pool:               Q12, Q48, Q49, Q50
Modules:                   Q13, Q14, Q15, Q16, Q80, Q90, Q92
File System & Streams:     Q23-Q32
child_process:             Q33-Q44
Worker Threads:            Q45-Q52
PM2:                       Q53-Q64
HTTP & Networking:         Q65-Q74
Debugging & Performance:   Q75-Q79
Tricky/Advanced:           Q81-Q100
```

---

*Study one section per day. For each question — explain it out loud as if teaching a teammate. That's the exact voice you need in the interview.*
