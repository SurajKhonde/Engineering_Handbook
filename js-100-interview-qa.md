# JavaScript Interview Prep — 100 Questions & Answers (with “surprise” examples)

> Focus areas: primitives vs objects, `typeof`, `==` vs `===`, **ToPrimitive coercion**, scope chain, global/function/block scope, closures, async/promises, and advanced JS.

---

## Table of contents
- [A. Types & Coercion](#a-types--coercion-1--25)
- [B. Scope, Hoisting, Closures](#b-scope-hoisting-closures-26--45)
- [C. Functions, `this`, Prototypes, Classes](#c-functions-this-prototypes-classes-46--70)
- [D. Async, Event Loop, Promises](#d-async-event-loop-promises-71--90)
- [E. Practical/Advanced Patterns](#e-practicaladvanced-patterns-91--100)

---

## A. Types & Coercion (1–25)

### 1) What are **primitives** in JS?
**Answer:** Primitives are immutable, non-object values: `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, `null`.
- “Immutable” means operations create new values rather than changing the primitive in-place.

```js
let s = "hi";
s[0] = "H";
console.log(s); // "hi" (still)
```

---

### 2) What are **objects** (non-primitives)?
**Answer:** Everything that’s not a primitive is an object: `{}`, `[]`, functions, dates, regex, maps, sets, etc. Objects are mutable and stored by reference.

```js
const a = { x: 1 };
const b = a;
b.x = 2;
console.log(a.x); // 2
```

---

### 3) What’s the difference between **pass-by-value** and **pass-by-reference** in JS?
**Answer:** JS passes **values**. For objects, the “value” is a **reference** to the object.
- Primitives: copied as values.
- Objects: reference value copied (two variables point to same object).

---

### 4) Why does `typeof null` return `"object"`?
**Answer:** It’s a historical bug kept for backward compatibility.

```js
typeof null; // "object" (surprising)
```

---

### 5) What does `typeof` return for common values?
**Answer:**
- `typeof 123` → `"number"`
- `typeof "x"` → `"string"`
- `typeof true` → `"boolean"`
- `typeof undefined` → `"undefined"`
- `typeof 10n` → `"bigint"`
- `typeof Symbol()` → `"symbol"`
- `typeof function(){}` → `"function"`
- `typeof []` → `"object"`
- `typeof null` → `"object"` (quirk)

---

### 6) How to correctly check “array” vs “object”?
**Answer:** Use `Array.isArray(x)` for arrays.

```js
Array.isArray([]); // true
Array.isArray({}); // false
```

---

### 7) What’s the difference between `==` and `===`?
**Answer:**
- `===` is **strict equality**: no type coercion.
- `==` is **loose equality**: tries coercion using spec rules.

---

### 8) Give 5 `==` examples that surprise people.
**Answer (and why):**
```js
0 == "0"        // true  ("0" -> 0)
0 == ""         // true  "" -> 0
0 == " 	
"    // true  whitespace string -> 0
null == undefined // true (special rule)
[] == ""        // true  [] -> "" -> 0? actually "" == "" true
```
**Explanation:** `==` may trigger **ToPrimitive** then string/number conversions.

---

### 9) Why is `[] == ![]` true?
**Answer:**  
- `![]` is `false` because any object is truthy.
- Then `[] == false` triggers coercion: `[]` → `""`, `false` → `0`, `""` → `0`, so `0 == 0` → `true`.

```js
[] == ![] // true (classic surprise)
```

---

### 10) What is **ToPrimitive**?
**Answer:** It’s an internal conversion: when JS needs a primitive (e.g., in `+`, `==`, template strings), it tries:
1. `obj[Symbol.toPrimitive](hint)` if present
2. otherwise `valueOf()` then `toString()` (order depends on hint / type)

---

### 11) Show ToPrimitive with `Symbol.toPrimitive`.
```js
const obj = {
  [Symbol.toPrimitive](hint) {
    return hint === "number" ? 10 : "ten";
  }
};

+obj;        // 10  (number hint)
`${obj}`;    // "ten" (string hint)
obj + 1;     // "ten1" (default hint acts like string-ish here)
```
**Answer:** Engines pass a “hint” and the object decides what primitive to return.

---

### 12) Why does `Date` behave differently in `+` or string contexts?
**Answer:** `Date`’s default ToPrimitive is **string-preferred**.

```js
const d = new Date(0);
d + 1;     // like "Thu Jan..." + "1" (string concat)
+d;        // 0 (number conversion)
```

---

### 13) What’s the difference between `Object.is` and `===`?
**Answer:** `Object.is` handles:
- `NaN` equals itself
- `+0` and `-0` are different

```js
NaN === NaN;          // false
Object.is(NaN, NaN);  // true

0 === -0;             // true
Object.is(0, -0);     // false
```

---

### 14) Why is `NaN` weird?
**Answer:** `NaN` is “not a number” and is not equal to anything, including itself.

```js
Number.isNaN(NaN); // true
NaN === NaN;       // false
```

---

### 15) What does `+` do with strings and numbers?
**Answer:** If either operand becomes a string, `+` concatenates; otherwise it adds numbers.

```js
1 + "2"; // "12"
"1" + 2; // "12"
1 + 2;   // 3
```

---

### 16) Surprise: `{} + []` vs `[] + {}`
**Answer:** Parsing matters.
- In many consoles, `{}` at start is treated as a block, so `{}` is ignored and you get `+[]` → `0`.
- `[] + {}` → `"" + "[object Object]"` → `"[object Object]"`

```js
[] + {} // "[object Object]"
```
**Note:** The exact output of `{} + []` depends on where/how you run it.

---

### 17) What’s the difference between `Number("10")` and `parseInt("10")`?
**Answer:**
- `Number("10")` parses the whole string strictly.
- `parseInt("10")` parses until it can’t.

```js
Number("10px");   // NaN
parseInt("10px"); // 10
```

---

### 18) Surprise: `parseInt` with radix.
**Answer:** Always pass radix.
```js
parseInt("08", 10); // 8
parseInt("1e2", 10);// 1 (stops at 'e')
```

---

### 19) What’s `undefined` vs `null`?
**Answer:**
- `undefined`: “not assigned / missing”
- `null`: “intentionally empty”
`null == undefined` is true, but `null === undefined` is false.

---

### 20) What is **boxing** of primitives?
**Answer:** When you access properties on primitives, JS temporarily wraps them in objects.

```js
"abc".toUpperCase(); // works via temporary String object wrapper
```

---

### 21) What are Symbols used for?
**Answer:** Unique keys to avoid collisions; also used for well-known behaviors like `Symbol.iterator`.

```js
const id = Symbol("id");
const o = { [id]: 123 };
```

---

### 22) What are BigInts?
**Answer:** Integers of arbitrary size, written with `n`.

```js
10n + 20n; // 30n
// 10n + 20 -> TypeError (cannot mix BigInt and Number)
```

---

### 23) What does “truthy/falsy” mean?
**Answer:** Falsy values: `false, 0, -0, 0n, "", null, undefined, NaN`. Everything else is truthy.

---

### 24) Surprise: empty array is truthy.
```js
if ([]) console.log("truthy"); // prints
```
**Answer:** Objects are always truthy, even “empty” ones.

---

### 25) What is `??` (nullish coalescing) vs `||`?
**Answer:**  
- `||` uses falsy check.
- `??` uses only `null` or `undefined`.

```js
0 || 10;   // 10
0 ?? 10;   // 0
"" || "x"; // "x"
"" ?? "x"; // ""
```

---

## B. Scope, Hoisting, Closures (26–45)

### 26) What is scope?
**Answer:** Scope determines where variables are accessible.

---

### 27) Global scope vs function scope vs block scope
**Answer:**
- **Global**: accessible everywhere (careful in Node modules vs browser).
- **Function scope**: `var` lives inside function.
- **Block scope**: `let/const` live inside `{}` blocks.

---

### 28) `var` vs `let` vs `const`
**Answer:**
- `var`: function-scoped, hoisted, can be re-declared.
- `let`: block-scoped, hoisted but in TDZ, re-assignable.
- `const`: block-scoped, TDZ, not re-assignable (object contents can still mutate).

---

### 29) What is hoisting?
**Answer:** Declarations are processed before execution.
- Function declarations are hoisted with their body.
- `var` is hoisted and initialized to `undefined`.
- `let/const` are hoisted but not initialized (TDZ).

---

### 30) What is TDZ (Temporal Dead Zone)?
**Answer:** The period where a `let/const` exists but can’t be accessed until its declaration line runs.

```js
console.log(x); // ReferenceError
let x = 10;
```

---

### 31) What is the scope chain?
**Answer:** When JS looks up a variable, it checks:
1) local scope
2) outer scopes (parents)
3) global
This linked structure is the **scope chain**.

---

### 32) Trace variable lookup in nested functions (scope chain example)
```js
const a = "global";

function outer() {
  const b = "outer";

  function inner() {
    const c = "inner";
    console.log(a, b, c);
  }

  inner();
}
outer();
```

**Answer (lookup trace):**
- `c` found in `inner` (local)
- `b` not in `inner`, found in `outer`
- `a` not in `inner` or `outer`, found in global

---

### 33) What is a closure?
**Answer:** A function that remembers variables from its outer scope even after that outer function has finished.

```js
function makeCounter() {
  let count = 0;
  return () => ++count;
}
const c = makeCounter();
c(); // 1
c(); // 2
```

---

### 34) Classic closure interview trap with loops (`var`)
```js
const fns = [];
for (var i = 0; i < 3; i++) {
  fns.push(() => i);
}
console.log(fns[0](), fns[1](), fns[2]()); // 3 3 3
```

**Answer:** `var` is function-scoped; all closures share the same `i`.

---

### 35) Fix the loop trap with `let`
```js
const fns = [];
for (let i = 0; i < 3; i++) {
  fns.push(() => i);
}
console.log(fns[0](), fns[1](), fns[2]()); // 0 1 2
```
**Answer:** `let` creates a new block-scoped `i` per iteration.

---

### 36) Fix the loop trap with IIFE (older style)
```js
for (var i = 0; i < 3; i++) {
  ((j) => fns.push(() => j))(i);
}
```
**Answer:** IIFE captures current value as `j`.

---

### 37) What is shadowing?
**Answer:** Inner scope variable with same name as outer scope variable.

```js
let x = 1;
function f() {
  let x = 2; // shadows outer x
  return x;
}
```

---

### 38) What is illegal shadowing?
**Answer:** `let/const` cannot be re-declared in the same scope; also mixing `let` and `var` can cause issues in nested blocks.

```js
let x = 1;
// let x = 2; // SyntaxError (same scope)
```

---

### 39) What does `"use strict"` change?
**Answer:** Strict mode removes some silent errors and changes `this` behavior in functions.
Example: assigning to undeclared var throws error.

```js
"use strict";
x = 10; // ReferenceError
```

---

### 40) How does `this` behave in strict vs non-strict (preview)?
**Answer:** In strict mode, `this` in a plain function call is `undefined` (not `window`).

---

### 41) What is “lexical environment”?
**Answer:** Internal structure storing variables + reference to outer environment; closures keep it alive.

---

### 42) What are modules’ top-level scopes like in Node?
**Answer:** In CommonJS/ESM modules, top-level variables are module-scoped, not truly global.

---

### 43) What is “global object” in browser vs Node?
**Answer:**
- Browser: `window` (and `globalThis`)
- Node: `global` (and `globalThis`)

---

### 44) What is `globalThis`?
**Answer:** Standard way to access the global object across environments.

---

### 45) Why is `let` preferred over `var` in modern code?
**Answer:** Safer scoping, avoids accidental re-declarations, avoids hoisting surprises.

---

## C. Functions, `this`, Prototypes, Classes (46–70)

### 46) Function declaration vs function expression
**Answer:**
- Declaration is hoisted with body.
- Expression behaves like a value; hoisting depends on `var/let/const`.

---

### 47) What is an arrow function?
**Answer:** Concise function with **lexical `this`** (doesn’t have its own `this`, `arguments`, `new`).

---

### 48) Arrow function `this` surprise
```js
const obj = {
  x: 1,
  getX: () => this.x
};
obj.getX(); // usually undefined (or window/global), not 1
```
**Answer:** Arrow `this` comes from where it was defined, not from call-site.

---

### 49) How does `this` get its value?
**Answer:** Mostly by call-site:
- `obj.fn()` → `this = obj`
- `fn()` → `this = global` (non-strict) or `undefined` (strict)
- `fn.call(x)` / `fn.apply(x)` → `this = x`
- `new Fn()` → `this = new object`

---

### 50) `call` vs `apply` vs `bind`
**Answer:**
- `call(thisArg, a, b)`
- `apply(thisArg, [a, b])`
- `bind(thisArg)` returns a new function with fixed `this`.

---

### 51) What is a “higher-order function”?
**Answer:** A function that takes functions as arguments or returns a function (`map`, `filter`, `compose`).

---

### 52) What is currying?
**Answer:** Transforming `f(a,b,c)` into `f(a)(b)(c)`.

---

### 53) What is a pure function?
**Answer:** Same inputs → same outputs, no side effects.

---

### 54) What is immutability and why it matters?
**Answer:** Avoids unexpected changes; helps debugging and predictable state updates (Redux).

---

### 55) What is prototype in JS?
**Answer:** Objects have an internal link to another object (prototype) used for property lookup.

---

### 56) Explain prototype chain lookup
**Answer:** If property not found on object, JS checks its prototype, then prototype’s prototype, etc, until `null`.

---

### 57) `__proto__` vs `prototype`
**Answer:**
- `obj.__proto__` (or `Object.getPrototypeOf(obj)`) is the actual prototype of an object.
- `Fn.prototype` is the object assigned as prototype for instances created with `new Fn()`.

---

### 58) What does `new` do?
**Answer:** It:
1) creates a new empty object
2) sets its prototype to `Fn.prototype`
3) calls `Fn` with `this` bound to that object
4) returns the object (unless Fn returns another object)

---

### 59) What is a constructor function?
**Answer:** Any function used with `new` is a constructor.

---

### 60) What are ES6 classes really?
**Answer:** Syntax sugar over prototypes (with nicer semantics and `super`).

---

### 61) Class field vs prototype method
**Answer:**
- Methods in `class { method(){} }` go on the prototype.
- Fields like `x = 1` live per instance.

---

### 62) What is `super`?
**Answer:** Used in subclass to call parent constructor/methods. Must call `super()` before using `this` in derived constructors.

---

### 63) Private fields `#x`
**Answer:** Truly private, not accessible outside class.

```js
class A { #x = 1; getX(){ return this.#x; } }
```

---

### 64) What is a getter/setter?
**Answer:** Property access that runs functions.

```js
const o = {
  get x() { return 1; },
  set x(v) { console.log("set", v); }
};
```

---

### 65) What is `Object.freeze`?
**Answer:** Prevents adding/removing/changing properties at top level (shallow freeze).

---

### 66) What is `Object.seal`?
**Answer:** Prevents adding/removing properties, but existing properties can change (if writable).

---

### 67) What are `Map` and `Set`?
**Answer:**
- `Map`: key-value with any key type.
- `Set`: unique values.

---

### 68) `Map` vs plain object
**Answer:** `Map` preserves insertion order, supports non-string keys, better for frequent add/remove.

---

### 69) What are WeakMap/WeakSet?
**Answer:** Keys (WeakMap) or values (WeakSet) are weakly held; allows GC. Not iterable.

---

### 70) What are iterators and iterables?
**Answer:** Iterables implement `Symbol.iterator` returning an iterator with `.next()`.

---

## D. Async, Event Loop, Promises (71–90)

### 71) What is the event loop in one line?
**Answer:** It runs queued callbacks when the call stack is empty, coordinating async tasks.

---

### 72) Microtask queue vs task (macrotask) queue
**Answer:** Microtasks (Promises) run **before** the next task (timers, IO callbacks).

---

### 73) Predict the output (microtasks win)
```js
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
```
**Answer:** `A D C B` (promise `.then` microtask runs before timer task).

---

### 74) What is a Promise?
**Answer:** An object representing a value that may be available now, later, or never. States: `pending`, `fulfilled`, `rejected`.

---

### 75) What does `.then` return?
**Answer:** A **new Promise**, enabling chaining.

---

### 76) Promise chaining vs nesting
**Answer:** Return promises from `.then` to avoid “callback pyramid”.

---

### 77) What is `async/await`?
**Answer:** Syntax over promises. `await` pauses within async function until promise settles, without blocking the whole thread.

---

### 78) Does `await` block the CPU thread?
**Answer:** It **does not block** the event loop; it yields, and the rest resumes later as a microtask.

---

### 79) What is “unhandled rejection”?
**Answer:** A promise rejection that isn’t caught; browsers/Node may warn or crash depending on settings.

---

### 80) `Promise.all` vs `Promise.allSettled`
**Answer:**
- `all`: fails fast on first rejection.
- `allSettled`: waits for all, returns statuses.

---

### 81) `Promise.race` vs `Promise.any`
**Answer:**
- `race`: first settled (resolve OR reject) wins.
- `any`: first fulfilled wins; rejects only if all reject (AggregateError).

---

### 82) How to run async tasks in parallel with a limit?
**Answer:** Use a concurrency limiter (queue) or run batches with `Promise.all` on slices.

---

### 83) What is a callback?
**Answer:** Function passed to another function to run later (often after async work).

---

### 84) What are common async sources in JS?
**Answer:** Timers, network requests, file IO (Node), UI events, promises.

---

### 85) What is “callback hell” and how to avoid it?
**Answer:** Deep nesting of callbacks. Avoid with promises, async/await, or composing functions.

---

### 86) What is an AbortController (web/fetch)?
**Answer:** A way to cancel fetch/async operations.

```js
const ac = new AbortController();
fetch(url, { signal: ac.signal });
ac.abort();
```

---

### 87) What is throttling vs debouncing?
**Answer:**
- **Debounce**: run after user stops triggering (e.g., search input).
- **Throttle**: run at most once per interval (e.g., scroll handler).

---

### 88) Why are CPU-heavy loops bad in Node/JS?
**Answer:** They block the single JS thread; no other callbacks can run → server feels “stuck”.

---

### 89) How to handle CPU-heavy work in Node?
**Answer:** Use `worker_threads`, `cluster`, or move heavy computation to another service/native module.

---

### 90) What is the libuv threadpool in Node?
**Answer:** Background worker threads (default commonly 4) used for certain operations like fs/crypto/zlib; JS still runs on main thread.

---

## E. Practical/Advanced Patterns (91–100)

### 91) What is deep clone and why is it hard?
**Answer:** Objects can include cycles, dates, maps, sets, functions—JSON cloning loses types.

---

### 92) What are common pitfalls of `JSON.stringify` cloning?
**Answer:** Loses `Date`, `Map`, `Set`, `undefined`, `Infinity`, functions; fails on circular references.

---

### 93) How to safely check “plain object”?
**Answer:** Use `Object.prototype.toString.call(x) === "[object Object]"` plus prototype checks if needed.

---

### 94) What is memoization?
**Answer:** Cache function results by inputs to speed up repeated calls.

---

### 95) What is event delegation?
**Answer:** Attach one handler to a parent, handle events from children using `event.target`.

---

### 96) What is a module (ESM) and what are named/default exports?
**Answer:**
- `export default ...` (one default)
- `export const x = ...` (named)
- `import x from` vs `import { x } from`

---

### 97) What is tree-shaking?
**Answer:** Bundlers remove unused ESM exports during build.

---

### 98) What is “prototype pollution”?
**Answer:** Security issue where user input modifies `Object.prototype`, impacting all objects.

---

### 99) What are common ways to avoid mutation bugs in state?
**Answer:** Use spread/rest, `Array.map`, `Object.assign`, or libraries like Immer.

---

### 100) Interview: Explain `==` in one sentence and what you do in real code.
**Answer:** `==` performs coercion via ToPrimitive + conversion rules; in production I mostly use `===` and only use `== null` intentionally to match `null` or `undefined`.

```js
if (x == null) { /* matches null or undefined */ }
```

---

## Extra: 20 “Rapid-fire” mini questions (bonus)
If you want, I can generate an additional **20 rapid-fire** Q&As focused only on:
- tricky coercion
- event loop ordering
- `this` binding puzzles
- prototype chain questions

