# Async Patterns & Error Handling
## 2) Async Patterns & Error Handling

1. Compare **callbacks**, **promises**, and **async/await** in Node. When would you still see callbacks?

**Ans**
In Node, callbacks, promises, and async/await are just different async styles. Callbacks are the old style (usually err, data) and are common in event-based APIs, but nesting makes code messy. Promises improve this by giving chaining and one place for errors using .catch(). async/await is the same as promises but more readable, using try/catch. You still see callbacks a lot in Node for streams/events (req.on, stream.on) and in older/core APIs or libraries.
2. What is **error‑first callback** convention? Why does it exist?
   In Node callbacks usually follow (err, data). If something fails, err is an Error and data is ignored; if success, err is null and data is valid. This convention makes error handling consistent, easy to compose, and compatible with utilities like util.promisify, and it prevents ambiguity (you always know where to check errors first).
```js
   fs.readFile("a.txt", (err, data) => {
  if (err) return console.error(err);
  console.log(data.toString());
});

```
3. Explain **promise chaining** rules: return a value vs return a promise vs throw.
**Ans**
Every .then() returns a new Promise. If you return a value, the next .then() receives that value. If you return a Promise, the chain waits for it and the next .then() receives its resolved value. If you throw (or return a rejected Promise), the chain becomes rejected and control jumps to the nearest .catch().

```js
Promise.resolve(1)
  .then(x => x + 1)                 // return value -> next gets 2
  .then(x => Promise.resolve(x*2))  // return promise -> waits -> next gets 4
  .then(() => { throw new Error("boom"); }) // throw -> goes to catch
  .catch(err => console.log(err.message));

```
4. What is an **unhandled promise rejection**? How do you prevent it in production code?
**Ans**
An unhandled rejection happens when a Promise rejects but no .catch() handles it (and it isn’t awaited inside try/catch). This is dangerous because errors get lost or can crash the process depending on settings. Prevent it by: always return/await promises, end chains with .catch(), wrap awaits in try/catch, and in Node add process.on('unhandledRejection') to log/alert (and typically restart safely).

```js
process.on("unhandledRejection", (reason) => {
  console.error("UNHANDLED:", reason);
});

```
5. How do you handle errors in an Express async route reliably (without repeating try/catch everywhere)?
Wrap async handlers with a small helper that catches rejections and calls next(err), so your global error middleware handles it. This keeps routes clean and ensures no rejected Promise is missed.
```js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get("/users/:id", asyncHandler(async (req, res) => {
  const user = await db.getUser(req.params.id);
  res.json(user);
}));

app.use((err, req, res, next) => {
  res.status(500).json({ message: "Internal Error" });
});

```


6. When should you use `Promise.all` vs `allSettled` vs `race` vs `any`?
all: “need all succeed” → fails fast on first rejection.
allSettled: “collect all results” → never fails, gives success/fail for each.
race: “first settled wins” → first resolve OR reject.
any: “first success wins” → ignores rejects until one resolves; fails only if all reject.

```js
await Promise.all([a(), b()]);              // all must succeed
await Promise.allSettled([a(), b()]);       // get both outcomes
await Promise.race([slow(), timeout(100)]); // first settled
await Promise.any([maybeFail1(), maybeOk()]); // first fulfilled

```


1. How do you implement **timeouts** and **retries** for async operations safely?
For timeouts, race the operation with a timer that rejects. For retries, retry only on retryable errors (timeouts, 5xx, network), add exponential backoff + jitter, and cap attempts to avoid thundering herd. Also consider idempotency (don’t retry non-idempotent operations unless you have an idempotency key).

```js
const sleep = (ms) => new Promise(r => setTimeout(r, ms));

function withTimeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, rej) => setTimeout(() => rej(new Error("timeout")), ms))
  ]);
}

async function retry(fn, attempts = 3) {
  let delay = 200;
  for (let i = 0; i < attempts; i++) {
    try { return await fn(); }
    catch (e) {
      if (i === attempts - 1) throw e;
      await sleep(delay + Math.random() * 100); // jitter
      delay *= 2;
    }
  }
}

```
2. How do you avoid “callback hell” without over‑engineering?
Use small named functions, keep nesting shallow, and prefer Promises/async-await for sequential logic. If callbacks are required (events/streams), keep them event-style and don’t build deep pyramids—extract logic to functions and return early on errors.
```js
// instead of nested callbacks, use async/await
async function handler() {
  const a = await stepA();
  const b = await stepB(a);
  return stepC(b);
}

```

3.  What is **backpressure** in async pipelines and why does it matter for reliability?
   Backpressure means the consumer is slower than the producer, so you must slow down production or you’ll buffer unlimited data and crash (memory blow, latency spikes). In Node streams, backpressure is built in: write() returns false when the buffer is full, and you wait for 'drain'. This keeps systems stable under load.

```js
if (!writable.write(chunk)) {
  readable.pause();
  writable.once("drain", () => readable.resume());
}

```
4.  Scenario: A batch job should continue even if some tasks fail. Which promise combinator pattern do you use and why?

Use Promise.allSettled because it returns outcomes for every task without failing early. Then you can count successes, log failures, and decide whether to retry failures or proceed. Use all only if one failure should stop the whole batch.
```js
const results = await Promise.allSettled(tasks.map(t => t()));
const ok = results.filter(r => r.status === "fulfilled");
const fail = results.filter(r => r.status === "rejected");
```