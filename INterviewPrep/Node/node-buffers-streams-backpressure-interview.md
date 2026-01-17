# Buffers, Streams & Backpressure (Node.js) — Interview Answers

> Quick, memorable interview-ready answers with mental models + key follow-ups.

---

## 1) What is a **Buffer** in Node? Why isn’t a normal JS string/array enough for binary data?

**Answer (interview):**  
A **Buffer** is Node’s way to represent **raw bytes** (binary data) in memory. JS **strings are text** (encoding-dependent) and can’t safely represent *arbitrary bytes* without encoding/decoding; JS arrays store **boxed numbers/objects** and are too heavy/slow for byte-level I/O.

**Mental model:**  
**String = letters**, **Buffer = a box of bytes**. Files/sockets speak bytes, not letters.

**Good follow-ups to mention:**
- `Buffer` is effectively a **Uint8Array** (bytes 0–255) and is **mutable**.
- Binary ↔ text requires an **encoding** (`utf8`, `base64`, `hex`).

---

## 2) When do you use a **Buffer** vs a **Stream**?

**Answer (interview):**  
Use a **Buffer** when the data is **small and you need it all at once** (e.g., parsing a small header, hashing a small payload). Use a **Stream** when data is **large or continuous** (files, uploads, network) so you process **chunk-by-chunk** and keep memory flat.

**Mental model:**  
**Buffer = full meal on a plate.**  
**Stream = buffet line.** You take small portions as they come.

---

## 3) Explain the 4 stream types: **Readable, Writable, Duplex, Transform**.

**Answer (interview):**
- **Readable**: produces data (`fs.createReadStream`, HTTP `req` body).  
- **Writable**: consumes data (`fs.createWriteStream`, HTTP `res`).  
- **Duplex**: both readable + writable, **independent directions** (`net.Socket`).  
- **Transform**: duplex **that modifies** data (gzip, encryption) (`zlib.createGzip()`).

**Mental model:**  
Readable = **faucet**, Writable = **drain**  
Duplex = **two-way pipe**  
Transform = **water filter** (changes what flows through)

---

## 4) What is **backpressure** in streams? How does Node signal it?

**Answer (interview):**  
**Backpressure** is when the consumer (Writable) is slower than the producer (Readable). Node signals it mainly via **`writable.write(chunk)` returning `false`**, meaning “my internal buffer is full—stop pushing.” When the buffer drains, Writable emits **`drain`**.

**Mental model:**  
Sink/drain analogy: if the drain is clogged, you stop pouring.  
`write() === false` → “buffer is full.”

**Keywords:** `highWaterMark`, `write() => false`, `drain`, `pause()/resume()`

---

## 5) Why is `fs.readFile()` risky for very large files? What’s the stream alternative?

**Answer (interview):**  
`fs.readFile()` loads the **entire file into RAM** before you can process it. For huge files it can cause **high memory usage, GC pressure, or OOM** and slow the process. The stream alternative is **`fs.createReadStream()`** so you process **chunk-by-chunk**.

---

## 6) What does `.pipe()` do under the hood conceptually? (flow + backpressure)

**Answer (interview):**  
`source.pipe(dest)` wires the Readable to push chunks into `dest.write(chunk)`. If `dest.write()` returns **false**, pipe tells the source to **pause**. When `dest` emits **`drain`**, pipe **resumes** the source. On `end`, it typically calls `dest.end()`.

**Mental model:**  
`pipe()` is a **flow controller** + **backpressure coordinator** between producer and consumer.

---

## 7) How do you handle stream errors correctly? Where do errors surface?

**Answer (interview):**  
Streams emit errors via the **`'error'` event** on *the stream that failed*. In a chain (read → transform → write), **any stage** can error, so you handle errors for **all streams** or use `pipeline()` which wires error handling + cleanup correctly.

**Interview gold:**  
“I use `stream.pipeline()` (or `stream/promises` pipeline) because it forwards errors and closes all streams reliably.”

**Where errors can surface:**
- Readable: disk permissions, ENOENT, read failures  
- Transform: gzip/crypto failures  
- Writable: disk full, broken connection

---

## 8) What is the difference between **flowing** and **paused** mode in Readable streams?

**Answer (interview):**
- **Paused mode (pull):** you pull data via `read()` / listen to `'readable'`.  
- **Flowing mode (push):** data is pushed via `'data'` events or when you `pipe()`.

**How it switches:**
- Add `'data'` listener or call `.resume()` → **flowing**  
- Use `'readable'` + `.read()` → stays **paused/pull**

**Mental model:**  
Paused = **you ask for data**. Flowing = **data is thrown at you**.

---

## 9) Scenario: Implement a large file upload API. How do you avoid high memory usage?

**Answer (interview):**  
I treat the request body as a **Readable stream** and **never buffer the whole file**. I stream it directly to storage (disk/S3) and rely on **backpressure** so Node doesn’t accumulate chunks in memory.

**Checklist I mention:**
- Use a **streaming multipart parser** (busboy/formidable) so file parts are streamed.
- Set limits: **max file size**, timeouts, validate content type; use `Content-Length` if present.
- If client disconnects, **destroy streams** and cleanup partial uploads.
- Use `pipeline()` for safe error handling and cleanup.

**Mental model:**  
Upload endpoint should behave like a **conveyor belt**, not a warehouse.

---

## 10) Scenario: Compress a file on the fly and upload to S3. Which stream type(s) would you use?

**Answer (interview):**  
Use a **Transform stream** for compression in the middle:  
**Readable (file)** → **Transform (gzip)** → **Writable (S3 upload stream / request body)**

**Mental model:**  
Bytes go through a **filter (Transform)**, then into the **delivery pipe (Writable to S3)**.

---

## Tiny code sketch (optional in interview)

```js
const fs = require("node:fs");
const zlib = require("node:zlib");
const { pipeline } = require("node:stream/promises");

await pipeline(
  fs.createReadStream("big.log"),
  zlib.createGzip(),          // Transform
  fs.createWriteStream("big.log.gz")
);
```

---

### One-liners to sound senior (optional)

- “I use **streams** for large data to keep memory **O(1)** with backpressure.”
- “I prefer **`pipeline()`** over manual `.pipe()` + multiple error handlers.”
- “Backpressure is signaled by **`write()` returning false** and recovered via **`drain`**.”
