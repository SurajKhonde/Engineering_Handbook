# JavaScript Async — Deep Interview Guide

> Callback → Promise → Async/Await — full evolution with internals

---

## The Core Problem: JavaScript is Single-Threaded

Before anything else, understand **why** async exists.

JS runs on **one thread** — it can only do one thing at a time. If you make a network request and wait for it, the entire page freezes. Async patterns solve this.

```js
// Without async thinking — this would freeze the browser for 3 seconds
const data = readFileSync("big-file.txt"); // blocks everything
console.log("This waits for the file");
console.log("So does this");
```

The **Event Loop** is what makes async JS possible — it picks up completed async tasks from a queue and runs their callbacks when the call stack is empty.

---

## 1. Callback — The Foundation

### What it actually is

A callback is a function you hand to another function and say: *"call this when you're done."*

```js
function greet(name, callback) {
  console.log("Hello " + name);
  callback(); // call it when done
}

greet("Suraj", function() {
  console.log("Callback ran!");
});
// Hello Suraj
// Callback ran!
```

### Why it exists — async use case

```js
const fs = require("fs");

// Node doesn't block here. It registers the callback and moves on.
fs.readFile("data.txt", function(err, data) {
  // This runs LATER, when file is ready
  if (err) {
    console.log("Error:", err);
    return;
  }
  console.log(data.toString());
});

console.log("This runs BEFORE the file is read"); // ← runs first
```

### The Error-First Convention (Node.js pattern)

Node.js always follows this pattern for callbacks:

```js
function doSomething(input, callback) {
  if (!input) {
    callback(new Error("Input required"), null); // error first
    return;
  }
  callback(null, "result"); // null error = success
}

doSomething("hello", function(err, result) {
  if (err) {
    console.log("Failed:", err.message);
    return;
  }
  console.log("Success:", result);
});
```

> 💡 **Rule:** First argument is always the error. If no error, pass `null`. This is a convention, not enforced by JS.

---

## 2. Callback Hell — The Real Problem

When you need results from multiple async operations in sequence, callbacks nest deeply. This is called **Callback Hell** or the **Pyramid of Doom**.

```js
// Real scenario: get user → get their orders → get order details → get product info
fs.readFile("user.txt", function(err, userData) {
  if (err) return console.log(err);

  fs.readFile("orders.txt", function(err, orderData) {
    if (err) return console.log(err);

    fs.readFile("details.txt", function(err, detailData) {
      if (err) return console.log(err);

      fs.readFile("products.txt", function(err, productData) {
        if (err) return console.log(err);

        // Finally have everything — but look at this mess
        console.log("Done");
      });
    });
  });
});
```

### Why Callback Hell is bad

```
Problem 1 — Readability:   Code grows rightward, hard to follow
Problem 2 — Error handling: Each level needs its own err check
Problem 3 — Reusability:   Logic is deeply buried, hard to extract
Problem 4 — Debugging:     Stack traces are confusing
Problem 5 — Control flow:  No clean way to run things in parallel
```

### The "fix" before Promises — Named functions

```js
// Better, but still messy
function handleProducts(err, productData) {
  if (err) return console.log(err);
  console.log("Done");
}
function handleDetails(err, detailData) {
  if (err) return console.log(err);
  fs.readFile("products.txt", handleProducts);
}
function handleOrders(err, orderData) {
  if (err) return console.log(err);
  fs.readFile("details.txt", handleDetails);
}

fs.readFile("orders.txt", handleOrders);
// Still callback-based — Promises were the real solution
```

---

## 3. Promise — The Real Fix

### What a Promise actually is

A Promise is an **object** representing the eventual completion or failure of an async operation. Think of it as a receipt: "I promise to give you the result later."

### Three states — very important for interviews

```
Pending   → operation not done yet
Fulfilled → operation succeeded (resolve was called)
Rejected  → operation failed (reject was called)

Once settled (fulfilled or rejected), a Promise CANNOT change state.
```

### Creating a Promise

```js
const promise = new Promise(function(resolve, reject) {
  // Do async work here
  const success = true;

  if (success) {
    resolve("Here is your data"); // fulfills the promise
  } else {
    reject(new Error("Something went wrong")); // rejects it
  }
});
```

### Consuming a Promise

```js
promise
  .then(function(data) {
    console.log(data); // "Here is your data"
  })
  .catch(function(err) {
    console.log(err.message);
  })
  .finally(function() {
    console.log("Always runs — success or failure");
  });
```

### Real async Promise

```js
function fetchUser(id) {
  return new Promise(function(resolve, reject) {
    setTimeout(function() {     // simulate network delay
      if (id === 1) {
        resolve({ id: 1, name: "Suraj" });
      } else {
        reject(new Error("User not found"));
      }
    }, 1000);
  });
}

fetchUser(1)
  .then(user => console.log(user.name)) // Suraj (after 1 second)
  .catch(err => console.log(err.message));
```

---

## 4. Promise Chaining

`.then()` always returns a **new Promise** — so you can chain them. This is the fix for callback hell.

```js
function getUser(id) {
  return new Promise(resolve => {
    setTimeout(() => resolve({ id, name: "Suraj" }), 500);
  });
}

function getOrders(user) {
  return new Promise(resolve => {
    setTimeout(() => resolve([{ id: 101, item: "Laptop" }]), 500);
  });
}

function getOrderDetail(order) {
  return new Promise(resolve => {
    setTimeout(() => resolve({ ...order, price: 80000 }), 500);
  });
}

// Clean chain — no nesting
getUser(1)
  .then(user => {
    console.log("User:", user.name);
    return getOrders(user);         // return next promise
  })
  .then(orders => {
    console.log("Orders:", orders);
    return getOrderDetail(orders[0]); // return next promise
  })
  .then(detail => {
    console.log("Detail:", detail);
  })
  .catch(err => {
    // ONE catch handles errors from the ENTIRE chain
    console.log("Error:", err.message);
  });
```

### Key rule: always return inside `.then()`

```js
// ❌ Wrong — not returning the promise, chain breaks
fetchUser(1)
  .then(user => {
    fetchOrders(user); // forgot return — next .then gets undefined
  })
  .then(orders => {
    console.log(orders); // undefined!
  });

// ✅ Correct
fetchUser(1)
  .then(user => {
    return fetchOrders(user); // return it
  })
  .then(orders => {
    console.log(orders); // works
  });
```

---

## 5. Promise.all() — Run in Parallel, Wait for All

When you need **multiple independent promises** to all succeed.

```js
const p1 = fetch("/api/users").then(r => r.json());
const p2 = fetch("/api/products").then(r => r.json());
const p3 = fetch("/api/orders").then(r => r.json());

// Runs all 3 at the SAME TIME — not one by one
Promise.all([p1, p2, p3])
  .then(([users, products, orders]) => {
    // All three completed successfully
    console.log(users, products, orders);
  })
  .catch(err => {
    // If ANY ONE fails, this catch runs immediately
    // The other promises are ignored
    console.log("One failed:", err.message);
  });
```

### The critical behaviour

```js
const fast   = new Promise(resolve => setTimeout(() => resolve("fast"),   100));
const slow   = new Promise(resolve => setTimeout(() => resolve("slow"),  3000));
const broken = new Promise((_, reject) => setTimeout(() => reject(new Error("broken")), 500));

// ❌ If any fails, Promise.all rejects immediately
Promise.all([fast, slow, broken])
  .then(results => console.log(results))
  .catch(err => console.log("Failed:", err.message)); // Failed: broken

// "slow" was still running but we don't get its result
```

### When to use

```
✅ Load dashboard data — users + stats + notifications all at once
✅ Multiple independent API calls
✅ Processing array of items in parallel

❌ Don't use when one failure should not stop the others
```

---

## 6. Promise.allSettled() — Run in Parallel, Wait for All Results

Unlike `Promise.all()`, this **never rejects**. It waits for every promise to finish — success or failure — and gives you all results.

```js
const p1 = Promise.resolve("Users loaded");
const p2 = Promise.reject(new Error("Products API down"));
const p3 = Promise.resolve("Orders loaded");

Promise.allSettled([p1, p2, p3])
  .then(results => {
    results.forEach(result => {
      if (result.status === "fulfilled") {
        console.log("✅", result.value);
      } else {
        console.log("❌", result.reason.message);
      }
    });
  });

// ✅ Users loaded
// ❌ Products API down
// ✅ Orders loaded
```

### Promise.all vs Promise.allSettled — interview comparison

```
Promise.all()
  - Fails fast: ONE rejection = immediate rejection
  - Result: array of values (if all succeed)
  - Use when: all must succeed, else abort

Promise.allSettled()
  - Never rejects: waits for everything
  - Result: array of { status, value/reason }
  - Use when: partial success is acceptable
```

### Real use case

```js
// Send notifications to multiple users — don't stop if one fails
const userIds = [1, 2, 3, 4, 5];

const notifications = userIds.map(id =>
  sendNotification(id)
);

Promise.allSettled(notifications).then(results => {
  const failed  = results.filter(r => r.status === "rejected");
  const success = results.filter(r => r.status === "fulfilled");
  console.log(`Sent: ${success.length}, Failed: ${failed.length}`);
});
```

---

## 7. Other Promise Methods (Bonus for Senior Interviews)

### Promise.race() — First one wins

```js
const p1 = new Promise(resolve => setTimeout(() => resolve("slow"), 3000));
const p2 = new Promise(resolve => setTimeout(() => resolve("fast"), 500));

Promise.race([p1, p2])
  .then(result => console.log(result)); // "fast"
// Whoever resolves/rejects first wins — others are ignored
```

**Use case:** Timeout pattern

```js
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error("Timed out")), ms)
  );
  return Promise.race([promise, timeout]);
}

withTimeout(fetchUser(1), 2000)
  .then(user => console.log(user))
  .catch(err => console.log(err.message)); // "Timed out" if > 2s
```

### Promise.any() — First success wins

```js
// Like race(), but ignores rejections — settles on first FULFILLMENT
Promise.any([
  Promise.reject("fail 1"),
  Promise.resolve("success"),
  Promise.resolve("also success"),
])
.then(result => console.log(result)); // "success"
```

---

## 8. Async/Await — Cleaner Syntax on Top of Promises

### What it actually is

`async/await` is **not** a new async model. It is syntactic sugar over Promises. Every `async` function returns a Promise under the hood.

```js
// These are identical
function getUser() {
  return Promise.resolve("Suraj");
}

async function getUser() {
  return "Suraj"; // async automatically wraps in Promise.resolve()
}

// Both: getUser().then(u => console.log(u))
```

### await pauses execution inside the function only

```js
async function loadDashboard() {
  console.log("start");

  const user = await fetchUser();       // pauses HERE — non-blocking
  console.log("user done");

  const orders = await fetchOrders();   // pauses HERE
  console.log("orders done");

  return { user, orders };
}

loadDashboard();
console.log("this runs immediately"); // ← doesn't wait for loadDashboard
```

Output:
```
start
this runs immediately    ← runs before user/orders
user done
orders done
```

### Error handling with try/catch

```js
async function getUser(id) {
  try {
    const res  = await fetch(`/api/users/${id}`);

    if (!res.ok) throw new Error(`HTTP error: ${res.status}`);

    const data = await res.json();
    return data;
  } catch (err) {
    console.log("Failed:", err.message);
    throw err; // re-throw if you want callers to handle it too
  }
}
```

### Sequential vs Parallel with async/await

```js
// ❌ Sequential — slow! Waits for each one before starting the next
async function loadSlow() {
  const users    = await fetchUsers();    // 1s
  const products = await fetchProducts(); // 1s — starts after users done
  const orders   = await fetchOrders();   // 1s — starts after products done
  // Total: ~3 seconds
}

// ✅ Parallel — fast! All start at the same time
async function loadFast() {
  const [users, products, orders] = await Promise.all([
    fetchUsers(),
    fetchProducts(),
    fetchOrders(),
  ]);
  // Total: ~1 second (they run simultaneously)
}
```

> 💡 This is one of the most common real-world mistakes — accidentally making parallel work sequential with async/await.

### async/await with loops

```js
const ids = [1, 2, 3, 4, 5];

// ❌ forEach does NOT work with await
ids.forEach(async (id) => {
  const user = await fetchUser(id); // doesn't wait between iterations
  console.log(user);
});

// ✅ Use for...of for sequential
for (const id of ids) {
  const user = await fetchUser(id); // waits for each one
  console.log(user);
}

// ✅ Use Promise.all for parallel
const users = await Promise.all(ids.map(id => fetchUser(id)));
```

---

## 9. Complete Real-World Example

### Dashboard loader — combines everything

```js
async function loadDashboard(userId) {
  try {
    // Step 1: Get user first (need userId for next calls)
    const user = await fetchUser(userId);

    // Step 2: Load user's data in parallel (independent of each other)
    const [orders, notifications, settings] = await Promise.all([
      fetchOrders(user.id),
      fetchNotifications(user.id),
      fetchSettings(user.id),
    ]);

    // Step 3: Load order details — partial failure is OK
    const detailResults = await Promise.allSettled(
      orders.map(order => fetchOrderDetail(order.id))
    );

    const details = detailResults
      .filter(r => r.status === "fulfilled")
      .map(r => r.value);

    return { user, orders, notifications, settings, details };

  } catch (err) {
    console.error("Dashboard load failed:", err.message);
    throw err;
  }
}
```

---

## 10. Interview Cheat Sheet

### The evolution

```
Callback
  ↓ (solved: async operations)
  Problem: Callback Hell, error handling per-level

Promise
  ↓ (solved: nesting, chaining, centralised error handling)
  Problem: verbose .then().then().then()

Async/Await
  ↓ (solved: readability, looks synchronous)
  Still Promises underneath — same model, better syntax
```

### Quick comparison table

| | Callback | Promise | Async/Await |
|---|---|---|---|
| Syntax | `cb(err, data)` | `.then().catch()` | `await / try-catch` |
| Error handling | Per callback | `.catch()` at end | `try/catch` block |
| Nesting | Deep pyramid | Flat chain | Linear code |
| Parallel | Manual tracking | `Promise.all()` | `await Promise.all()` |
| Returns | Nothing | Promise object | Promise object |
| Cancellable | No | No (native) | No (native) |

### Promise methods summary

| Method | Behaviour | Use when |
|---|---|---|
| `Promise.all()` | Fails if ANY fails | All must succeed |
| `Promise.allSettled()` | Never fails, gives all results | Partial success OK |
| `Promise.race()` | First to settle wins | Timeout pattern |
| `Promise.any()` | First to SUCCEED wins | Fallback sources |

### Common interview questions

**Q: Is async/await better than Promises?**  
A: It's not "better" — it's the same thing with different syntax. `async/await` improves readability for sequential async logic. For parallel work you still use `Promise.all()` inside async/await.

**Q: Can you use await outside an async function?**  
A: Yes — top-level await is supported in ES modules (`.mjs` files or `"type": "module"` in package.json). In CommonJS you cannot.

**Q: What happens if you don't catch a rejected Promise?**  
A: You get an `UnhandledPromiseRejectionWarning` in Node.js. In modern Node (v15+) it crashes the process. Always handle rejections.

**Q: Does await block the entire thread?**  
A: No. It only pauses the current async function. The event loop continues running other code.

**Q: Difference between Promise.all and running awaits in sequence?**  
A: Sequential awaits run one after another (slower). `Promise.all` starts all simultaneously (faster). Use `Promise.all` for independent operations.

---

*JS Async Deep Guide — Callbacks · Promises · Async/Await*
