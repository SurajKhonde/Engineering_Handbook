# JavaScript & TypeScript (25)
[!note]All Ans are used 
1. Explain `var` vs `let` vs `const` (scope, hoisting, TDZ).
    **ans**: var, let, and const are ways to declare variables in JavaScript. The main difference is scope and how they behave before declaration. var is function-scoped and it’s hoisted in a way that it becomes undefined if you access it before the line where it’s declared. let and const are block-scoped, and even though they are also hoisted, they stay in the temporal dead zone, so accessing them before declaration throws a ReferenceError. In practice I use const by default, let when I need reassignment, and I avoid var to prevent subtle bugs.

2. What is closure? Give a real use-case.

**ans**
 A closure is when a function remembers and can access variables from its outer lexical scope even after the outer function has finished executing. This is used to keep private state, like counters, caching, or encapsulating data.
  ```js
 function createBankAccount() {
  let balance = 0;

  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      balance -= amount;
      return balance;
    },
    getBalance() {
      return balance;
    }
  };
}
const acc = createBankAccount();
acc.deposit(100);
acc.withdraw(30); 
acc.getBalance();

function makeCounter() {
  let count = 0; 
  return function () {
    count++;
    return count;
  };
}

const c1 = makeCounter();
c1();
c1();

const c2 = makeCounter();
c2();
 ```
3. What is Lexical? Give a real use-case.
  Lexical scope means: scope is decided by where you write the code (in the source), not by where you call it.
So a function can access variables from the blocks/functions it is physically inside.
```js
const x = 10;
function outer() {
  const x = 20;
  function inner() {
    console.log(x);
  }

  return inner;
}
const fn = outer();
fn();

```
Why 20?
Because inner() was defined inside outer(), so its lexical scope includes outer’s x. Even when you call it later, it still uses the scope it was born in.
So when you see a variable in a function, JavaScript answers:

“Where was this function defined in the code?
That’s where it can look for variables.”

Not:

“Where was this function called from?”

That’s the whole idea.
 
**5. Explain prototypal inheritance in JavaScript.**

JavaScript uses prototypal inheritance, meaning every object has a hidden link called [[Prototype]] to another object. When you access a property/method (like arr.push), JS first checks the object itself; if not found, it looks up the prototype chain (e.g., Array.prototype, then Object.prototype) until it finds it or returns undefined. This lets many objects share the same methods without copying them.

```js
const arr = [10, 20, 30];
console.log(arr.hasOwnProperty("push"));                 // false
console.log(Object.getPrototypeOf(arr) === Array.prototype); // true
arr.push(40);
console.log(arr); // [10, 20, 30, 40]
```

1. Explain `this` binding rules (call/apply/bind, arrow functions).

 this means “who is calling this function?” In JS, this is decided when you call the function. If you call with dot `obj.fn()` then `this = obj`. If you call normally fn() then this = undefined (in strict mode). If you use new `new Fn()` then this becomes the new object. If you use call/apply/bind, you force what this should be. Arrow functions don’t have their own this—they just use this from the outer function.
 ```js
"use strict";
function show() { return this?.name; }
const a = { name: "A" };
const b = { name: "B" };
a.show = show // must  
console.log(a.show()); // "A"  direct Result: TypeError: a.show is not a function
// 2) explicit (call/apply)
console.log(show.call(b));      // "B"
console.log(show.apply(b));     // "B"
// 3) bind (returns new function with fixed this)
const bound = show.bind(a);
console.log(bound());           // "A"
// 4) default (plain call)
console.log(show());            // undefined (strict mode)
// Arrow: lexical this
const obj = {
  name: "OBJ",
  normal() { return (() => this.name)(); }, // arrow captures this from normal()
  arrow: () => this?.name,                  // arrow has NO own this (captures from outer scope)
};
console.log(obj.normal()); // "OBJ"
console.log(obj.arrow());  // undefined (in strict/module scope), not "OBJ"

 ```
 Arrow function takes this from the surrounding (outer) function, so this doesn’t change when the arrow is called.
1. What is the event loop? Explain microtasks vs macrotasks.
**Ans**
JavaScript runs sync code on a single call stack. Async work (timers, network, I/O) is handled by the runtime and its callbacks are put into queues. The event loop is the mechanism that, when the call stack becomes empty, moves queued callbacks onto the stack to run. Microtasks have higher priority: after the current stack finishes, JS runs all microtasks first (Promise .then/.catch/finally, queueMicrotask), and only then runs the next macrotask (setTimeout/setInterval, I/O, UI events).
Memory line: `Stack → Microtasks (all) → Macrotask (one) → repeat.`”

What happens when JS sees async code?

- JS starts running your code on the call stack.
- When it hits an async API (like setTimeout, fetch, file I/O):
- JS registers the request with the runtime (browser APIs / Node libuv).
- Then JS immediately continues to the next line (so stack stays free).
- When the async work finishes, the runtime puts the callback into:
- Microtask queue (Promises), or
- Macrotask queue (timers, I/O events).
- When the call stack becomes empty:
- Event loop pushes all microtasks to the stack (run them),
- Then pushes one macrotask to the stack,

```ts
                (Your JS code runs here)
        +-------------------------------+
        |          CALL STACK           |
        |  sync functions execute here  |
        +---------------+---------------+
                        |
                        | hits async API (setTimeout/fetch/I-O)
                        v
        +-------------------------------+
        |     RUNTIME (Browser/Node)    |
        |   Web APIs / libuv handles    |
        |   timer, network, file I/O    |
        +---------------+---------------+
                        |
                        | when async completes
                        v
        +--------------------+   +--------------------+
        |   MICROTASK QUEUE  |   |   MACROTASK QUEUE  |
        | Promise.then/catch |   | setTimeout, I/O     |
        | queueMicrotask     |   | events, intervals   |
        +---------+----------+   +---------+----------+
                  |                        |
                  | (priority)             |
                  +-----------+------------+
                              |
                              v
                    +------------------+
                    |    EVENT LOOP    |
                    | if stack empty:  |
                    | 1) run ALL micro |
                    | 2) run ONE macro |
                    +--------+---------+
                             |
                             v
                        (back to)
                     +---CALL STACK---+
```

1.  Explain Promises: states, chaining, error propagation.
   **Promise**
   A Promise is an object representing a future result of async work. It has 3 states: pending → fulfilled (success value) or rejected (error). A promise settles only once—after fulfilled/rejected it can’t change. Chaining works because .then() returns a new Promise, so you can return a value (passes to next .then) or return another promise (waits for it). Error propagation is automatic: if any .then() throws or returns a rejected promise, the chain skips to the nearest .catch(), and after handling, the chain can continue.
   
   ```js
      const p = new Promise((resolve, reject) => {
  if (true) resolve("ok");
  else reject(new Error("we stuck here"));
   });
   p.then((x) => {
  console.log(x);          // ok
  return x.toUpperCase();  // return value -> next then gets it  
    })
   .then((y) => {
  console.log(y);          // OK
  throw new Error("boom"); // throw -> goes to catch
   })
   .catch((err) => {
  console.error(err.message); // boom
  return "recovered";         // continue chain
   })
  .then((z) => {
  console.log(z);          // recovered
   });

   ```
2.  `async/await` under the hood: how does it relate to promises?
   **ans**
   async/await is syntax sugar on top of Promises. An async function always returns a Promise: if you return value, it becomes a resolved Promise, and if you throw, it becomes a rejected Promise. await works like .then()—it waits for a Promise to settle and gives you its fulfilled value; if the Promise rejects, await throws, which is why we handle it with try/catch. Important: await pauses only that async function, not the whole JS thread—the rest of the program keeps running.

3.  How do you handle errors in async code reliably?
   **ans **
To handle async errors reliably, I use one rule: every async operation must be awaited or returned, and errors must flow to one central handler. In async/await, wrap awaited code in try/catch and either handle it or rethrow a clean error. In Promise chains, end with .catch(). In Node/Express, I forward errors to a global error middleware (next(err)), and I also monitor unhandledRejection and uncaughtException to log/alert (and usually restart the process), so errors never get silently ignored.
1.  What is “unhandled promise rejection” and how to prevent it?
   **ans**
  An unhandled promise rejection happens when a Promise rejects but no error handler is attached—meaning there’s no .catch() in the chain and it isn’t awaited inside a try/catch. This is dangerous because the app may log warnings or even crash (Node versions can treat it strictly). To prevent it: always return/await Promises, end promise chains with .catch(), wrap awaited code in try/catch, and in Node add a process-level unhandledRejection handler for logging/alerting (and usually restart safely). 

```js
// ❌ Unhandled rejection (no catch)
Promise.reject(new Error("fail"));

// ✅ Handled
Promise.reject(new Error("fail")).catch(console.error);

// ✅ async/await handled
(async () => {
  try {
    await Promise.reject(new Error("fail"));
  } catch (e) {
    console.error(e);
  }
})(); 
```

2.  Explain `==` vs `===` and type coercion pitfalls.
 **ans**
`==` is loose equality: it may convert (coerce) types before comparing, which can create surprising results. `===` is strict equality: it compares type + value with no coercion, so it’s safer for predictable code. Main pitfalls of `==` come from coercion rules like strings to numbers, booleans to numbers, and the special case where null `==` undefined is true but 
**null is not equal to anything else.**
```js
console.log(10 == "10");   // true  (coercion)
console.log(10 === "10");  // false
console.log("" == 0);      // true  ("" -> 0)
console.log(false == 0);   // true  (false -> 0)
console.log(null == undefined);  // true (special rule)
console.log(null === undefined); // false
const obj = {};
const obj2 = obj;
console.log(obj == obj2);   // true  (same reference)
console.log(obj === obj2);  // true
```

3.  What is deep copy vs shallow copy? Common ways to clone objects.
    A shallow copy clones only the top-level properties—if the object contains nested objects/arrays, the nested ones are still shared references, so changes inside them affect both copies. A deep copy clones everything recursively, so nested objects are independent. Common shallow clone methods are spread {...obj} and Object.assign({}, obj); for deep clone, use structuredClone(obj) (best built-in) or JSON clone for simple data (but it loses functions/Date/undefined).
        
4.  How does garbage collection work in JS at a high level?
**ans**
JavaScript uses automatic garbage collection based on reachability (most engines use mark-and-sweep style). The GC starts from roots (global objects, current call stack variables, active closures, etc.), marks everything reachable by following references, and then sweeps (frees) the unmarked memory. So memory is reclaimed when an object becomes unreachable, not simply when a function finishes—if something is still referenced (like via a closure, cache, or event listener), it stays alive.

1. Explain pass-by-value vs pass-by-reference behavior in JS.

JavaScript is pass-by-value: when you pass a variable to a function, JS copies its value. For primitives (number, string, boolean, null, undefined, symbol, bigint) the value is the actual data, so changes inside the function don’t affect the outside. For objects/arrays/functions, the copied value is a reference to the same heap object, so mutating the object inside the function affects the original. But reassigning the parameter to a new object does not change the original reference outside.

```js
   function changePrimitive(x) {
  x = x + 1;
}
let a = 10;
changePrimitive(a);
console.log(a); // 10

function mutateObj(o) {
  o.count = 2;      // mutation affects original
  o = { count: 99 } // reassignment does NOT affect original
}
const obj = { count: 1 };
mutateObj(obj);
console.log(obj); // { count: 2 }

```

2.  What are modules? CommonJS vs ES Modules.

Modules let us split code into separate files with private scope and explicit exports, so code is reusable and maintainable. In CommonJS (CJS) (classic Node), we use `require() and module.exports`, and it loads modules synchronously. In ES Modules (ESM) (modern JS), we use import/export, and it supports static imports (better tooling/tree-shaking) and async loading in the spec. Today Node supports both, but ESM is the modern standard for new code.

```js
// CommonJS (Node)
const fs = require("fs");
module.exports = { sum: (a, b) => a + b };

// ES Modules
import fs from "fs";
export const sum = (a, b) => a + b;
export default function add(a, b) { return a + b; }

```

```js
console.log(0 == "0"); // true
console.log(0 === "0"); // false
console.log("" == 0);   // true
console.log(false == 0); // true
console.log(false == "");// true
console.log(null == undefined); // true
console.log(null === undefined); //false
console.log(null == 0); // true
console.log([] == ""); //true
console.log([] == 0); // true
console.log([1] == 1); //true
console.log([] == ![]); //true
console.log(+""); //0
console.log(+"  \t\n"); //0
console.log(Number("  \t\n")); //0
console.log(parseInt("1e2")); //1
console.log(Number("1e2"));// 100
console.log(NaN == NaN); //false
console.log(Object.is(NaN, NaN)); true
console.log(0 || "A"); // "A"
console.log(0 ?? "A"); // 0
console.log("" ?? "A"); // 
console.log("" || "A"); // "A"
```

### B) Scope, hoisting, closures (Guess output)
```js
console.log(a); //undefined
var a = 10;
console.log(b);  // referceError
let b = 10;
foo(); //foo
function foo() { console.log("foo"); }

bar(); //TypeError
var bar = function () { console.log("bar"); };

for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log("var", i), 0);
}
// 3,3,3
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log("let", j), 0);
}
// 0 ,1,2

(function () {
  console.log(typeof x);
  let x = 1;
})(); // ReferenceError

"use strict";
function show() { return this?.name; }

const obj = { name: "A", show };
const f = obj.show;

console.log(obj.show()); // undefined
console.log(f()); // "A"

"use strict";
function make() {
  return {
    name: "INNER",
    normal() { return this.name; },
    arrow: () => this.name,
  };
}
const o = make.call({ name: "OUTER" });

console.log(o.normal()); //INNER
console.log(o.arrow()); // OUTER


console.log(1);
setTimeout(() => console.log(2), 0);
Promise.resolve().then(() => console.log(3));
console.log(4);
// 1,4,3,2 

Promise.resolve()
  .then(() => console.log("A"))
  .then(() => console.log("B"));
console.log("C");
 // C A,B

Promise.resolve()
  .then(() => { throw new Error("boom"); })
  .then(() => console.log("after then"))
  .catch(() => console.log("caught"));
// caught
Promise.resolve("x")
  .finally(() => "y")
  .then((v) => console.log(v));
// X

Promise.reject("err")
  .finally(() => console.log("finally"))
  .catch((e) => console.log("catch", e));
// "catch" err ,finally 

async function run() {
  console.log("a");
  await Promise.resolve();
  console.log("b");
}
console.log("c");
run();
console.log("d");
 // c, d ,a,b
setTimeout(() => console.log("t1"), 0);
Promise.resolve().then(() => console.log("p1"));
Promise.resolve().then(() => console.log("p2"));
setTimeout(() => console.log("t2"), 0);
 //  p1,p2,t1,t2

```

**1) [] == ![] ✅ result: true**

Step-by-step:

- `[]` is an object → objects are truthy
- ![] → false
- Now it becomes: [] == false
- With ==, if one side is boolean → JS converts boolean to number
false → 0
- Now: `[] == 0`
- [] (object) is converted to primitive: [].toString() → ""
- Now: `"" == 0`
- String to number: `"" → 0`
`0 == 0 → true`

**Memory line**: ![] => false => 0, and [] => "" => 0

**2) [] == 0 ✅ result: true**

- Object vs number → convert object to primitive
`[] -> ""`
`"" == 0 → ""` converts to 0
`0 == 0` → true

**3) [1] == 1 ✅ result: true**

1. `[1]` is object → convert to primitive
`[1].toString()` → `"1"`

2. `"1" == 1 `→ `"1"` converts to number 1

3. `1 == 1` → true

**Memory line**: [1] -> "1" -> 1

**4) NaN == NaN ✅ result: false**

`NaN` means “Not a Number” and by IEEE rules **NaN is never equal to anything**, even itself.

✅ checks:

- `Number.isNaN(NaN)` → true
- `Object.is(NaN, NaN)` → true

**5) "" ?? "A" ✅ result: "" (empty string)**

?? only uses the right side when the left side is null or undefined.

"" is not null/undefined → so it stays ""

Difference:

`"" || "A" → "A"` (because "" is falsy)

`"" ?? "A" → ""` (because "" is not nullish)

Memory line: ?? = “only null/undefined fallback”, not “falsy fallback”.

## TypeScript
1.  What is TypeScript? What problems does it solve?
2.  `any` vs `unknown` vs `never` — when to use each?
3.  Union vs intersection types with real examples.
4.  Generics in TypeScript — show an example you used.
5.  Type narrowing and type guards — how do you do it?
6.  Optional chaining and nullish coalescing — common mistakes.
7.  `readonly` and immutability patterns in TS.
8.  How do you type Express request/response handlers correctly?
9.  How do you validate runtime input even with TypeScript?
10. How do you structure types in a large codebase (dto, domain, api)?
