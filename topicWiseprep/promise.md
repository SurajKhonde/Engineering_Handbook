# Promises + Methods — Clean Notes (Correct Formatting + More Explanation)

These notes cover the Promise topics you mentioned:

- Promise fundamentals (what a Promise is)
- Promise states (pending/fulfilled/rejected) + “settle once”
- `.then()` chaining rules (return value / return promise / throw)
- `.catch()` as `.then(null, onRejected)` + error propagation
- `.finally()` rules + override pitfall
- Recovery vs rethrow patterns
- Implementing `sleep(ms)`
- Microtasks vs macrotasks (why `.then` runs before `setTimeout`)
- Promise utility methods: `all`, `allSettled`, `race`, `any`, `resolve`, `reject`
- `async/await` with Promise methods
- Edge cases: `AggregateError`, order preservation in `all`
- Unhandled rejections (Node/browser behavior)
- Sequential vs parallel execution
- Common anti-patterns
- Promisify (callback → promise)
- Cancellation using `AbortController` (fetch)

---

## 1) Promise fundamentals

A **Promise** is an object that represents the **eventual result** of an asynchronous operation.

- You don’t have the value immediately.
- You get it later via `.then(...)` (success) or `.catch(...)` (error).

### Promise states (core)

A Promise is always in **one** of these states:

1. **pending** → initial state (still running)
2. **fulfilled** → finished successfully (has a value)
3. **rejected** → finished with error (has a reason / error)

### “Settles only once” rule

A Promise **settles only once** (either fulfilled or rejected).  
After that, it never changes — later `resolve/reject` calls are ignored.

Why this matters:
- Predictable chaining: once a promise is done, it can’t flip again.

---

## 2) Basic Promise examples

### Example 1 — resolve vs reject

```js
const promise = new Promise((resolve, reject) => {
  if (true) return resolve("Ok");
  reject(new Error("we have issue here"));
});

promise
  .then((res) => console.log(res))
  .catch((err) => console.log(err));
```

**What happens?**
- Since `true`, it resolves with `"Ok"`.
- `.then` runs, `.catch` is skipped.

### Example 2 — resolve after 1 second

```js
const promise = new Promise((resolve, reject) => {
  if (true) return setTimeout(() => resolve("ok"), 1000);
  reject(new Error("we have issue here"));
});

promise
  .then((res) => console.log(res))
  .catch((err) => console.log(err));
```

**Key idea:** `setTimeout` delays `resolve`, but the promise still “settles once”.

---

## 3) `.then()` chaining (most important topic)

`.then(onFulfilled, onRejected)` **returns a NEW Promise**.

That new promise resolves based on what you do inside the handler:

### Rules inside `.then()`

- **Return a value** → next `.then` receives that value  
- **Return a promise** → chain waits for it  
- **Throw an error** → chain becomes rejected → next `.catch` handles it  

```js
Promise.resolve(1)
  .then((v) => v + 1)                        // returns 2
  .then((v) => Promise.resolve(v + 1))       // returns promise -> waits -> 3
  .then((v) => {
    throw new Error("boom");                 // throw -> rejection
  })
  .catch((err) => console.log(err.message)); // boom
```

### Why this is powerful
This is how we build clean async flows **without nested callbacks**.

---

## 4) `.catch()` is just `.then(null, handler)`

```js
p.catch(handleError);

// same as:
p.then(null, handleError);
```

### What `.catch` will catch
- Rejections from earlier promises
- Errors thrown inside any previous `.then(...)` handlers

---

## 5) `.finally()` behavior (very common interview question)

`finally` runs **no matter what** (success or error).

Key rules:
- It doesn’t receive the value/error by default.
- It **passes through** the original value/error unless it fails.

```js
Promise.resolve("data")
  .finally(() => console.log("cleanup"))
  .then((v) => console.log(v)); // cleanup, then data

Promise.reject("err")
  .finally(() => console.log("cleanup"))
  .catch((e) => console.log(e)); // cleanup, then err
```

### `finally` can override success if it throws (pitfall)

```js
Promise.resolve("ok")
  .finally(() => {
    throw new Error("fail in finally");
  })
  .then(console.log)
  .catch((e) => console.log(e.message)); // fail in finally
```

**Why?**
- Cleanup failed, so that failure becomes the final result.

---

## 6) Error propagation (how errors “bubble” in promises)

Errors flow forward until a `.catch` handles them.

```js
Promise.resolve()
  .then(() => {
    throw new Error("bad");
  })
  .then(() => console.log("never runs"))
  .catch((err) => console.log("caught:", err.message));
// caught: bad
```

### “Recovering” vs “rethrowing”

#### Recover (convert failure into success by returning a fallback value)

```js
Promise.reject(new Error("network"))
  .catch((err) => {
    return "fallback"; // recovery
  })
  .then((v) => console.log(v)); // fallback
```

#### Rethrow (keep the chain failed)

```js
Promise.reject(new Error("network"))
  .catch((err) => {
    throw err; // keep it rejected
  })
  .then(() => console.log("no"))
  .catch((err) => console.log("still failed:", err.message));
```

**Real-world meaning**
- Recover when you can safely continue (fallback UI, cache miss, optional feature).
- Rethrow when failure must stop the flow (payments, auth, data correctness).

---

## 7) Implement `sleep(ms)`

```js
function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

// Example
sleep(1000).then(() => console.log("1 second passed"));
```

> Fix applied: `setTimeout` (your draft had a small typo `setTimout`).

---

## 8) Order matters in chains (left → right)

A Promise chain flows **left to right**:

- `.then(...)` handles **success** from the previous step
- `.catch(...)` handles **failure** from the previous step (and errors thrown above)
- If `.catch(...)` returns a value → it **recovers** and the chain becomes fulfilled

```js
Promise.reject(new Error("network"))
  .then((v) => console.log("then:", v)) // skipped
  .catch((err) => "we are surpassing the issue")
  .then((v) => console.log("after recovery:", v));
```

---

## 9) Microtasks vs macrotasks (Promise vs setTimeout)

Promise callbacks (`then/catch/finally`) run in the **microtask queue**.  
`setTimeout` runs in the **task (macrotask) queue**.

**Rule:** after the current synchronous code finishes, the runtime drains **all microtasks** before taking the next macrotask.

```js
setTimeout(() => console.log("T"), 0);
Promise.resolve().then(() => console.log("P"));
console.log("S");

// Output:
// S
// P
// T
```

---

# 10) Promise methods (utility APIs)

We’ll reuse your dummy async helpers:

```js
function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function fetchDummy(name, ms, shouldFail = false) {
  await sleep(ms);

  if (shouldFail) {
    throw new Error(`${name} failed`);
  }

  return { name, ms, data: `data-for-${name}` };
}
```

---

## 10.1) `Promise.all()` — “all must succeed”

- Runs in parallel
- Resolves with results array (same order as input)
- Rejects immediately if any promise rejects (**fail-fast**)

```js
Promise.all([
  fetchDummy("A", 300),
  fetchDummy("B", 100),
  fetchDummy("C", 200),
])
  .then((results) => {
    console.log("ALL success:", results);
  })
  .catch((err) => {
    console.log("ALL failed:", err.message);
  });
```

### Order is preserved (not completion order)
Even if `B` finishes first, result order is still `[A, B, C]` because it matches input order.

### Fail-fast example
```js
Promise.all([
  fetchDummy("A", 300),
  fetchDummy("B", 100, true), // fails
  fetchDummy("C", 200),
])
  .then(console.log)
  .catch((err) => console.log("ALL failed:", err.message));
// ALL failed: B failed
```

---

## 10.2) `Promise.allSettled()` — “collect success + failure”

- Never rejects
- Returns per-promise outcome objects:
  - `{ status: "fulfilled", value: ... }`
  - `{ status: "rejected", reason: ... }`

```js
Promise.allSettled([
  fetchDummy("A", 300),
  fetchDummy("B", 100, true),
  fetchDummy("C", 200),
]).then((results) => {
  console.log(results);
});
```

**Use cases**
- Batch processing
- “Try everything, show partial results”
- Reporting which tasks failed without crashing

---

## 10.3) `Promise.race()` — “first to settle wins”

Returns the first promise that **settles** (fulfilled OR rejected).

```js
Promise.race([
  fetchDummy("A", 300),
  fetchDummy("B", 100),
  fetchDummy("C", 200),
]).then((winner) => {
  console.log("RACE winner:", winner);
});
```

### Real-world: timeout wrapper with `race`
```js
function timeout(ms) {
  return new Promise((_, reject) =>
    setTimeout(() => reject(new Error("Timed out")), ms)
  );
}

Promise.race([fetchDummy("API", 500), timeout(200)])
  .then((v) => console.log("Got:", v))
  .catch((e) => console.log("Error:", e.message));
// Error: Timed out
```

---

## 10.4) `Promise.any()` — “first success wins”

- Resolves as soon as **any one fulfills**
- Rejects only if **all reject** (with `AggregateError`)

```js
Promise.any([
  fetchDummy("A", 300, true),
  fetchDummy("B", 100, true),
  fetchDummy("C", 200), // success
])
  .then((v) => console.log("ANY success:", v))
  .catch((e) => console.log("ANY failed:", e));
```

### `AggregateError` edge case
```js
Promise.any([Promise.reject("a"), Promise.reject("b")])
  .catch((e) => console.log(e.name, e.errors)); // AggregateError ["a","b"]
```

**Use cases**
- Multiple mirrors/CDNs: take the first successful response
- Fallback servers

---

## 10.5) `Promise.resolve()` / `Promise.reject()` — quick helpers

```js
Promise.resolve(123).then(console.log); // 123
Promise.reject(new Error("nope")).catch((e) => console.log(e.message)); // nope
```

---

## 10.6) Using `async/await` with these methods

```js
async function run() {
  const results = await Promise.all([
    fetchDummy("A", 200),
    fetchDummy("B", 100),
  ]);
  console.log(results);
}

run();
```

---

# 11) Unhandled rejections (real-world debugging)

If a promise rejects and nobody handles it, you can get warnings/crashes depending on environment.

```js
Promise.reject(new Error("boom")); // unhandled
```

In Node you can listen:

```js
process.on("unhandledRejection", (err) => console.error(err));
```

**Practical advice**
- Always attach `.catch` OR wrap in `try/catch` with `await`.

---

# 12) Sequential vs parallel (big interview topic)

### ❌ Sequential (slower)
```js
const a = await fetchA();
const b = await fetchB();
```

### ✅ Parallel (faster)
```js
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

Why: both start at the same time, total time is closer to the max(duration), not sum(duration).

---

# 13) Promise constructor anti-patterns

### Don’t wrap a promise unnecessarily

❌ Bad (unneeded wrapper):
```js
return new Promise((res, rej) => {
  fetch(url).then(res).catch(rej);
});
```

✅ Good:
```js
return fetch(url);
```

### Avoid `new Promise(async (res, rej) => ...)`
Mixing `new Promise` + `async` often creates confusing flows and potential unhandled errors.
Prefer writing an `async function` instead.

---

# 14) Promisify callbacks (Node-style)

Convert Node callbacks `(err, data)` to promises:

```js
function promisify(fn) {
  return (...args) =>
    new Promise((resolve, reject) => {
      fn(...args, (err, data) => (err ? reject(err) : resolve(data)));
    });
}
```

---

# 15) Cancellation / AbortController (modern real-world)

Promises don’t cancel by default. For `fetch`, you cancel via `AbortController`.

```js
const controller = new AbortController();

fetch("/api/data", { signal: controller.signal })
  .then((r) => r.json())
  .then(console.log)
  .catch((e) => console.log("fetch error:", e.name)); // AbortError if aborted

controller.abort(); // cancel request
```

**Where it’s used**
- Cancel search requests when user types fast
- Cancel requests on route change/unmount in React

---

## Final quick interview summary

- Promises settle once: pending → fulfilled/rejected.
- `.then()` returns a new promise:
  - return value → next success
  - return promise → wait
  - throw → rejection
- `.catch()` handles earlier failures and can recover by returning a value.
- `.finally()` is cleanup; passes through unless it fails.
- `then/catch/finally` are microtasks (run before `setTimeout`).
- `all` = “all required”, `allSettled` = “collect everything”, `race` = “first settle”, `any` = “first success”.

## 1) What are Promise states? When does it change?

A Promise has **three states**:

- **pending**: initial state (not finished yet)
- **fulfilled**: finished successfully with a **value**
- **rejected**: finished with a **reason** (usually an `Error`)

### When does it change?
It changes **exactly once** when it “**settles**”:
- `resolve(value)` ⇒ becomes **fulfilled**
- `reject(reason)` ⇒ becomes **rejected**
- If you resolve with another promise/thenable, it adopts that promise’s final state.

### Why it matters
- A promise is **immutable after settlement**: later `resolve/reject` calls are ignored.
- This guarantees predictable async composition.

```js
const p = new Promise((resolve, reject) => {
  resolve("ok");
  reject("ignored"); // ignored
});

p.then(console.log); // ok
```

---

## 2) What does `.then()` return?

`.then()` **always returns a NEW Promise**.

### Why
This is what enables **chaining**:
each `.then()` creates a new step whose result depends on what you return/throw.

```js
const p2 = Promise.resolve(1).then(v => v + 1);
p2.then(console.log); // 2
```

---

## 3) If a `.then()` returns a value, what happens?

If you return a **non-promise value** inside `.then`, the promise returned by `.then` becomes **fulfilled** with that value.

### Why
The returned value becomes the input to the next step.

```js
Promise.resolve(10)
  .then(v => v * 2)       // returns 20
  .then(v => console.log(v)); // 20
```

---

## 4) If a `.then()` returns a Promise, what happens?

If you return a **promise** (or thenable) inside `.then`, the next promise **waits** and adopts its final state:

- if returned promise fulfills ⇒ chain fulfills with its value
- if returned promise rejects ⇒ chain rejects with its reason

### Why
This is the foundation of composing async work **without nested callbacks**.

```js
function fetchUser() {
  return new Promise(res => setTimeout(() => res({ id: 1 }), 100));
}

Promise.resolve()
  .then(() => fetchUser())     // returns a Promise
  .then(user => console.log(user.id)); // 1
```

---

## 5) If you throw inside `.then()`, where does it go?

If you `throw` inside `.then` (or `.catch`), that becomes a **rejection** of the promise returned by that handler.

So it “goes” to the **next `.catch()`** downstream (or next `then` with rejection handler).

### Why
Promises treat thrown exceptions in async callbacks the same way as rejecting a promise.

```js
Promise.resolve()
  .then(() => {
    throw new Error("boom");
  })
  .catch(err => console.log("caught:", err.message)); // caught: boom
```

---

## 6) Difference between `Promise.resolve(x)` and `new Promise(res => res(x))`?

For most values they behave similarly, but there are practical differences.

### `Promise.resolve(x)`
- If `x` is a **promise**, it returns **the same promise** (or a promise that follows it).
- If `x` is a **thenable**, it will be **assimilated** (followed).
- Typically used to “normalize” a value into a promise.

### `new Promise(res => res(x))`
- Always creates a **new promise** object.
- Runs the executor immediately (sync), although resolving still schedules reactions.

### Why it matters
- `Promise.resolve(existingPromise)` avoids unnecessary wrapping.
- `new Promise(...)` is heavier and often an anti-pattern if you already have a promise.

```js
const p = Promise.resolve(1);

Promise.resolve(p) === p; // true (usually)
new Promise(res => res(p)) === p; // false (new wrapper)
```

---

## 7) What does `.catch()` actually equal internally?

`.catch(onRejected)` is equivalent to:

```js
.then(null, onRejected)
```

### Why
Promises support two handlers in `.then`:
- first for fulfillment
- second for rejection

`.catch` is just convenience + readability.

---

## 8) What does `.finally()` receive as argument?

`finally` does **not** receive the value or error by default (it receives no arguments).

```js
Promise.resolve("x").finally(v => console.log(v)); // v is undefined
```

### Why
`finally` is meant for **cleanup** that should run regardless of success/failure:
- stop loader
- close DB connection
- release lock
- hide spinner

To access the value/error, use `.then/.catch` around it.

---

## 9) Can `.finally()` change the resolved value? When?

Normally, **no**: it passes through the previous value/error unchanged.

It **can** change the outcome only if:
- it **throws** an error, or
- it returns a **rejecting promise**

### Why
Because the promise returned from `finally` follows these rules:
- if cleanup succeeds ⇒ original result continues
- if cleanup fails ⇒ cleanup error overrides original result

```js
Promise.resolve("OK")
  .finally(() => { throw new Error("cleanup failed"); })
  .then(console.log)
  .catch(e => console.log(e.message)); // cleanup failed
```

---

## 10) Difference between `Promise.all` and `Promise.allSettled`?

### `Promise.all(promises)`
- **Fulfilled** when **all** fulfill ⇒ returns array of values
- **Rejected immediately** when **any one** rejects (fail-fast)
- Output order matches input order

### `Promise.allSettled(promises)`
- **Always fulfills**
- Returns array of objects describing each result:
  - `{ status: "fulfilled", value }`
  - `{ status: "rejected", reason }`

### Why it matters
- Use `all` when **all are required** (e.g., load critical data)
- Use `allSettled` when you want **partial success** (e.g., batch jobs, best-effort requests)

---

## 11) What happens if one promise rejects in `Promise.all`?

`Promise.all` **rejects immediately** with that rejection reason.

- You do **not** get partial results from `all`.
- Other promises may still continue running in the background, but `all` is already rejected.

```js
Promise.all([
  Promise.resolve("A"),
  Promise.reject("B failed"),
  Promise.resolve("C"),
]).catch(console.log); // B failed
```

### Why
Fail-fast is useful when the combined result is useless without every part.

---

## 12) What does `await` do under the hood (microtask)?

`await` is syntax sugar over promise chaining.

Conceptually:

```js
const v = await p;
```

behaves like:

```js
return p.then(v => { /* continue */ });
```

### Microtask detail
When the awaited value is already a resolved promise/value, the continuation (code after `await`) runs in a **microtask** (after current sync code finishes).

```js
async function f() {
  console.log("A");
  await null;
  console.log("B");
}
console.log("S");
f();
console.log("E");
// S A E B
```

### Why
This ensures async functions don’t “interrupt” the current call stack and keeps ordering consistent.

---

## 13) Why is `new Promise(async (res)=>...)` usually a bad idea?

Because it mixes two async models and can cause subtle problems:

- The Promise constructor expects you to call `res/rej`.  
- `async` functions already return a promise and use implicit rejection on throw.
- If you throw inside an `async` executor, it becomes a rejection of the **async function’s returned promise**, which may not properly link to the outer promise if you forget to handle it.
- It’s easy to accidentally create **double-wrapping** and **unhandled rejections**.

### Safer alternatives
- Use `async function` directly:
```js
async function work() { ... }
```
- Or use a normal executor (not async) and call `resolve/reject` explicitly.

**Bad (avoid):**
```js
new Promise(async (resolve) => {
  // await stuff...
  resolve(123);
});
```

**Good:**
```js
async function getValue() {
  // await stuff...
  return 123;
}
```

---

## 14) How do you convert callback-based async to Promise (promisify)?

Classic Node-style callbacks are: `(err, data) => ...`

A `promisify` wrapper converts that into a promise:

```js
function promisify(fn) {
  return (...args) =>
    new Promise((resolve, reject) => {
      fn(...args, (err, data) => {
        if (err) reject(err);
        else resolve(data);
      });
    });
}
```

Usage example:
```js
// Suppose readFileCb(path, cb) uses (err, data)
const readFileAsync = promisify(readFileCb);

readFileAsync("a.txt")
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

### Why it matters
It allows you to use `.then/.catch` or `await` instead of nested callbacks.

---

## 15) What is “error propagation” in promise chains?

Error propagation means:
- If a promise is rejected, or a handler throws, the error **travels down the chain** until something handles it (a `.catch` or rejection handler).

### Key behaviors
- If a `.catch` **returns a value**, it “recovers” and the chain becomes fulfilled after that.
- If a `.catch` **throws** or returns a rejecting promise, it stays rejected.

```js
Promise.resolve()
  .then(() => { throw new Error("fail"); })
  .then(() => console.log("never"))
  .catch(err => {
    console.log("caught:", err.message);
    return "recovered"; // recovery
  })
  .then(v => console.log("after:", v));
// caught: fail
// after: recovered
```

### Why
This is the async equivalent of try/catch that “bubbles” errors outward, but in a controlled, composable way.

---

# Quick Interview Summary (1-minute)
- Promises settle once: pending → fulfilled/rejected.
- `.then` returns a new promise; return value passes forward; return promise waits; throw becomes rejection.
- `.catch` = `.then(null, onRejected)` and can recover by returning a value.
- `.finally` is cleanup; doesn’t get value; passes through unless it fails.
- `await` is promise chaining; continuation runs as microtask.
- Use `all` for “all required”, `allSettled` for “collect outcomes”.
