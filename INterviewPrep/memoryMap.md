# Memory web 

## 1. Explain `var` vs `let` vs `const` (scope, hoisting, TDZ).

The best mnemonic: V = Function + Undefined

- V-A-R → “Var = Function + Undefined”
- Function scope
- Hoisted → gives undefined before declaration
- Risky → avoid

>[!note]Just remembering “Function + Undefined” is enough to speak.

The second mnemonic: L/C = **Block + Bang**
   Let/Const = Block scope + Bang error

- Block scope
- Hoisted but TDZ → accessing early BANG! ReferenceError

**Reassignment rule: Let = Re-Let, Const = Constant**

- let → “let it change” (reassign allowed)
- const → “constant binding” (reassign not allowed)

Extra mini-hook:

- Const locks the variable name, not the object.


## 2. What is closure?
3-word hook: Function + Memory + Scope
Closure = function carrying memory of outer scope.

## Lexical = Location in code.
Lexical scope = scope decided at write-time (code position), not run-time (call position).


## prototypal inheritance
“Me → Parent → Grandparent → Stop”
When you do obj.prop JS checks:

- Me (own properties)
- Parent (obj.[[Prototype]])
- Grandparent (prototype’s prototype…)
- Stop at null (not found → undefined)

Shortcut line: **Own → Proto → Proto… → null**

## event loop

The event loop is the mechanism that, when the call stack becomes empty, moves queued callbacks onto the stack to run.
JS only registers async work; runtime completes it; event loop schedules the callback back onto the stack.
**Memory line**: `Stack → Microtasks (all) → Macrotask (one) → repeat.`
Memory line:
`Stack → Runtime → Queue → Event loop → Stack`
`Priority: Microtasks (all) > Macrotask (one)`

## async/await

async = returns Promise, await = then + unwrap, reject = throw, try/catch handles it.

## error Handling

Await/return everything → errors flow to one place (catch / middleware) → log + respond + monitor.

## Garbage Collector

GC = “Start from roots → mark reachable → delete the rest.”

Memory line: `JS thread runs code; libuv/OS does I/O; callback returns to JS thread.`
**Memory line:**
`JS runs on one thread → libuv/OS does I/O → callbacks queued → stack empty?→ ALL microtasks (Promises) → ONE macrotask (timers/I-O/setImmediate) → repeat`

`process.nextTick()` is even higher priority than Promise microtasks in Node, and can starve the loop if abused.

**V8 runs JS → libuv does async I/O → C++ bindings connect JS ↔ native/OS.**

## Node js 
Browser gives JS “Web APIs + DOM”; Node gives JS “server APIs + libuv”; both run JS on one main thread and use an event loop to schedule callbacks.

Register async → OS/threadpool completes → libuv queues callback (macro) → V8 runs Promise handlers (micro) before next macro.

You’re thinking correctly: JS doesn’t do the waiting. But also: libuv doesn’t “do the database work” either.
For a DB call, the real “work” happens on the DB server, and the “waiting” happens in the OS networking layer. libuv’s main job is to watch for readiness and then run your JS callback.

Who actually “makes the DB call”?

Your Node DB driver/library (mysql2/pg/mongoose etc.) does:

Open a TCP connection to the DB (socket).

Write bytes (your query) to that socket.

After sending the bytes, Node is basically waiting for response bytes.

Who does the waiting?

The OS kernel waits for network data to arrive (socket becomes readable).

libuv asks the OS: “tell me when this socket has data.”

So yes: OS is doing the waiting, and DB server is doing the query processing.

Then what does “libuv schedules a JS callback” mean?

When the DB response arrives:

OS says: “socket has data”

libuv notices that (via epoll/kqueue/IOCP)

libuv puts the related JS callback into the event loop queue

When the JS call stack is free, Node runs that callback on the JS main thread

If you’re using Promises, the driver resolves the Promise, and V8 queues .then as a microtask

Full flow in 6 steps (DB call)

JS calls db.query(...)

Driver sends query over socket (JS thread)

DB server executes query (DB machine)

OS receives response packets (your machine kernel)

libuv sees socket ready and queues callback (event loop macro task)

JS runs callback / Promise .then runs (JS thread)


CPU-heavy JS = event loop blocked → fewer requests handled → throughput down, latency up.

Callback = events, Promise = chain, Async/Await = readable chain.