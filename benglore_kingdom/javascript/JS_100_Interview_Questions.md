# JavaScript Mastery — 100 Interview Questions
### For 3+ Year Experience Interviews

---

## Section 1: Memory & Types (Q1–15)

**Q1. How is memory allocated for primitives vs objects?**

Primitives (string, number, boolean, null, undefined, symbol, bigint) → stored in **Stack** — fixed size, by value.
Objects, arrays, functions → stored in **Heap** — dynamic size, by reference. Variable on stack holds a pointer.

```js
let a = 10;
let b = a;       // copy of VALUE
b = 20;
console.log(a);  // 10 — unaffected

let obj1 = { x: 1 };
let obj2 = obj1; // copy of REFERENCE
obj2.x = 99;
console.log(obj1.x); // 99 — same heap object!
```

---

**Q2. What is shallow copy vs deep copy?**

Shallow copy = top level copied, nested objects still shared.
Deep copy = fully independent clone, nothing shared.

```js
const orig = { name: 'Suraj', addr: { city: 'Bengaluru' } };

// Shallow
const s = { ...orig };
s.addr.city = 'Mumbai';
console.log(orig.addr.city); // 'Mumbai' — nested still shared!

// Deep
const d = structuredClone(orig); // best modern way
const d2 = JSON.parse(JSON.stringify(orig)); // simple, loses functions/Date
```

---

**Q3. What is typeof and what are its quirks?**

```js
typeof 42           // 'number'
typeof 'hello'      // 'string'
typeof true         // 'boolean'
typeof undefined    // 'undefined'
typeof Symbol()     // 'symbol'
typeof function(){} // 'function'
typeof {}           // 'object'
typeof []           // 'object'  ← array is object!
typeof null         // 'object'  ← famous bug, never fixed

// Better checks:
Array.isArray([])         // true
obj === null              // true
Object.prototype.toString.call([]) // '[object Array]'
```

---

**Q4. What is type coercion? Give dangerous examples.**

JS automatically converts types. Loose equality `==` triggers coercion.

```js
0 == false       // true  ← coercion
'' == false      // true
null == undefined // true
null == 0        // false ← surprise!

1 + '2'          // '12'  ← number becomes string
'5' - 2          // 3     ← string becomes number
[] + []          // ''
[] + {}          // '[object Object]'
{} + []          // 0     ← browser parses {} as block!

// Rule: always use === to avoid coercion
```

---

**Q5. What is NaN and how do you check for it?**

NaN = "Not a Number" — result of invalid math operations. Type is 'number'. NaN !== NaN (only value not equal to itself).

```js
typeof NaN        // 'number' ← weird
NaN === NaN       // false
isNaN('hello')    // true  ← coerces first, unreliable
Number.isNaN('hello') // false ← correct, no coercion
Number.isNaN(NaN)     // true  ← use this always
```

---

**Q6. What is the difference between null and undefined?**

```js
// undefined = variable declared but not assigned
let x;
console.log(x); // undefined

// null = intentional absence of value, must be assigned
let y = null;

typeof undefined // 'undefined'
typeof null      // 'object' ← bug

null == undefined  // true  (loose)
null === undefined // false (strict)

// Practical: use null to intentionally clear a value
// undefined means "not set yet"
```

---

**Q7. What are Symbol and BigInt used for?**

```js
// Symbol: unique identifier, never collides
const id1 = Symbol('id');
const id2 = Symbol('id');
id1 === id2 // false — always unique

// Use: private-ish object keys
const KEY = Symbol('key');
obj[KEY] = 'hidden';

// BigInt: integers beyond Number.MAX_SAFE_INTEGER
const big = 9007199254740991n + 1n; // works!
typeof big // 'bigint'
// Cannot mix BigInt and Number: 1n + 1 throws TypeError
```

---

**Q8. What is the difference between == and ===?**

`==` loose equality — coerces types before comparing.
`===` strict equality — no coercion, checks type AND value.

```js
1 == '1'   // true  (string coerced to number)
1 === '1'  // false (different types)
0 == false // true
0 === false // false

// Rule: always use ===
// Exception: null == undefined is a clean way to check both
if (val == null) { } // catches both null and undefined
```

---

**Q9. How does JavaScript handle integer precision?**

JS uses IEEE 754 double-precision floating point. Max safe integer is 2^53 - 1.

```js
Number.MAX_SAFE_INTEGER // 9007199254740991
0.1 + 0.2              // 0.30000000000000004 ← famous bug!
0.1 + 0.2 === 0.3      // false

// Fix:
+(0.1 + 0.2).toFixed(2) === 0.3 // true
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON // true

// For money: use integers (paise, cents) or BigInt or a library
```

---

**Q10. What is variable hoisting?**

JS moves declarations (not initializations) to top of their scope before executing.

Hoisting is JavaScript's behavior during the memory creation phase where declarations are registered before code execution begins. The behavior depends on the type of declaration:

- var is hoisted and initialized with undefined.
- let and const are hoisted but remain in the - Temporal Dead Zone until their declaration is reached.
Function declarations are fully hoisted and can be called before their declaration line.

```js
console.log(a); // undefined — hoisted but not initialized
var a = 5;

console.log(b); // ReferenceError: Cannot access before initialization
let b = 5;      // let/const hoisted but in TDZ (Temporal Dead Zone)

hello(); // works! function declarations are fully hoisted
function hello() { console.log('hi'); }

greet(); // TypeError: greet is not a function
var greet = function() { }; // only var declaration hoisted, not the function
```

---

what is scope ? 
Scope determines where a variable can be accessed in your code.

**Q11. What is Temporal Dead Zone (TDZ)?**

The period between entering scope and the let/const declaration being evaluated. Accessing the variable in TDZ throws ReferenceError.

```js
{
  // TDZ for x starts here
  console.log(x); // ReferenceError
  let x = 10;     // TDZ ends here
  console.log(x); // 10
}

// Why it exists: prevents using variables before they're ready
// var didn't have this protection
```

---

**Q12. What is the difference between var, let, const?**

```
           var          let          const
Scope:     Function     Block        Block
Hoisted:   Yes(undef)   Yes(TDZ)     Yes(TDZ)
Re-declare: Yes         No           No
Re-assign:  Yes         Yes          No
Global obj: Yes         No           No
```

```js
// const doesn't mean immutable!
const obj = { x: 1 };
obj.x = 2;      // works — object contents can change
obj = {};       // TypeError — can't reassign the binding

// Use const by default, let when you need to reassign, never var
```

---

**Q13. What is pass by value vs pass by reference in JS?**

JS is always pass by value. But for objects, the value IS a reference.

```js
// Primitive — pass by value (copy)
function double(n) { n = n * 2; }
let x = 5;
double(x);
console.log(x); // 5 — unchanged

// Object — pass by value of reference
function rename(obj) { obj.name = 'Changed'; }
let user = { name: 'Suraj' };
rename(user);
console.log(user.name); // 'Changed' — same reference modified

// But reassigning doesn't affect original
function replace(obj) { obj = { name: 'New' }; }
replace(user);
console.log(user.name); // 'Changed' — original reference intact
```

---

**Q14. What are wrapper objects for primitives?**

```js
// Primitives have no methods... so how does this work?
'hello'.toUpperCase() // 'HELLO'

// JS auto-wraps primitive in object temporarily, calls method, discards wrapper
// String('hello') → .toUpperCase() → unwrap → 'HELLO'

// Explicit wrappers (avoid in practice):
new String('hello') // object, not primitive
typeof new String('x') // 'object', not 'string'
```

---

**Q15. What is the difference between Object.freeze() and const?**

```js
const obj = { x: 1 };
obj.x = 2;        // works — const only prevents rebinding
obj = {};         // TypeError

Object.freeze(obj);
obj.x = 99;       // silently fails (or throws in strict mode)
obj.y = 5;        // silently fails
console.log(obj); // { x: 2 } — unchanged

// freeze is shallow — nested objects not frozen
const a = Object.freeze({ inner: { z: 1 } });
a.inner.z = 99;   // works! inner not frozen
```

---

## Section 2: Functions & Scope (Q16–30)

**Q16. What is a closure? Real use case.**

A closure is a function that remembers variables from its outer scope even after the outer function has returned.

```js
function makeCounter() {
  let count = 0;          // this variable is "closed over"
  return function() {
    count++;
    return count;
  };
}

const counter = makeCounter();
counter(); // 1
counter(); // 2
counter(); // 3 — count persists!

// Real use cases:
// 1. Data privacy / encapsulation
// 2. Function factories
// 3. Memoization
// 4. setTimeout loops (classic interview trap)

for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3 3 3 — var shares scope!
}
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0 1 2 — let has block scope
}
// Fix with var: wrap in IIFE or use let
```

---

**Q17. What is scope chain?**

When JS can't find a variable in current scope, it looks in outer scope, then outer's outer, all the way to global. This chain is called the scope chain.

```js
let global = 'G';
function outer() {
  let outerVar = 'O';
  function inner() {
    let innerVar = 'I';
    console.log(innerVar);  // found in own scope
    console.log(outerVar);  // found in outer scope
    console.log(global);    // found in global scope
  }
  inner();
}
// Scope chain is determined at WRITE time (lexical scope), not call time
```

---

**Q18. What is lexical scope?**

Scope is determined by where the function is WRITTEN in the code, not where it is called from.

```js
let x = 'global';

function outer() {
  let x = 'outer';
  function inner() {
    console.log(x); // 'outer' — looks at where inner was WRITTEN
  }
  return inner;
}

const fn = outer();
fn(); // 'outer' — NOT global, even though called from global scope
```

---

**Q19. What is the difference between function declaration and function expression?**

```js
// Declaration — hoisted completely
sayHi(); // works!
function sayHi() { console.log('hi'); }

// Expression — only var is hoisted, not the function
greet(); // TypeError: greet is not a function
var greet = function() { console.log('hello'); };

// Named function expression — name only accessible inside
const factorial = function fact(n) {
  return n <= 1 ? 1 : n * fact(n - 1); // fact available here
};
factorial(5); // 120
fact(5);      // ReferenceError — not outside
```

---

**Q20. What is an IIFE and why use it?**

Immediately Invoked Function Expression — runs immediately, creates private scope.

```js
(function() {
  var private = 'no one can see me';
  console.log('runs immediately');
})();

// Before ES6 modules, used to avoid polluting global scope
// Still useful for: initialization code, avoiding variable leaks
const result = (function() {
  const x = 10;
  return x * 2;
})(); // 20
```

---

**Q21. What is this keyword? How does it behave in different contexts?**

`this` refers to the object that is executing the current function. It depends on HOW the function is called.

```js
// 1. Global context
console.log(this); // window (browser) or {} (Node.js strict)

// 2. Object method
const obj = {
  name: 'Suraj',
  greet() { console.log(this.name); } // this = obj
};
obj.greet(); // 'Suraj'

// 3. Regular function (not method)
function fn() { console.log(this); }
fn(); // undefined (strict) or window (non-strict)

// 4. Arrow function — no own 'this', inherits from enclosing scope
const obj2 = {
  name: 'Suraj',
  greet: () => console.log(this.name) // this = outer scope (global)
};
obj2.greet(); // undefined!

// 5. Constructor
function Person(name) { this.name = name; }
const p = new Person('Suraj'); // this = new object

// 6. Explicit binding
fn.call(obj, arg1);    // sets this = obj, calls immediately
fn.apply(obj, [args]); // same but args as array
const bound = fn.bind(obj); // returns new function with this = obj
```

---

**Q22. What is the difference between call, apply, bind?**

All three explicitly set `this`. Difference is WHEN the function is called.

```js
function introduce(city, country) {
  console.log(`${this.name} from ${city}, ${country}`);
}
const user = { name: 'Suraj' };

introduce.call(user, 'Bengaluru', 'India');   // calls immediately, args spread
introduce.apply(user, ['Bengaluru', 'India']); // calls immediately, args array
const fn = introduce.bind(user, 'Bengaluru'); // returns new fn, call later
fn('India'); // call whenever
```

---

**Q23. What are arrow functions and how are they different from regular functions?**

```js
// Arrow function differences:
// 1. No own 'this' — inherits from enclosing scope
// 2. No 'arguments' object
// 3. Cannot be used as constructor (no 'new')
// 4. No 'prototype' property
// 5. Cannot be used as generator

class Timer {
  constructor() { this.count = 0; }
  start() {
    // Regular function: this = undefined (lost in setTimeout)
    // Arrow function: this = Timer instance (inherited)
    setInterval(() => {
      this.count++; // works because arrow inherits this
    }, 1000);
  }
}

// When NOT to use arrow functions:
const obj = {
  name: 'Suraj',
  greet: () => console.log(this.name) // wrong! use regular function
};
```

---

**Q24. What is currying?**

Transforming a function with multiple arguments into a sequence of functions each taking one argument.

```js
// Normal function
function add(a, b, c) { return a + b + c; }

// Curried version
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}
curriedAdd(1)(2)(3); // 6

// Practical use: reusable partial functions
const add10 = curriedAdd(10);
const add10and5 = add10(5);
add10and5(3); // 18

// Arrow syntax:
const curry = a => b => c => a + b + c;
```

---

**Q25. What is memoization?**

Caching the result of expensive function calls so repeated calls with same input return cached result.

```js
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      console.log('cache hit');
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const slowSquare = (n) => { /* expensive */ return n * n; };
const fastSquare = memoize(slowSquare);
fastSquare(100); // computed
fastSquare(100); // cache hit!
```

---

**Q26. What are higher-order functions?**

Functions that take a function as argument OR return a function (or both).

```js
// Takes function as argument
[1,2,3].map(x => x * 2);    // map is HOF
[1,2,3].filter(x => x > 1); // filter is HOF
[1,2,3].reduce((acc, x) => acc + x, 0); // reduce is HOF

// Returns a function
function multiplier(factor) {
  return n => n * factor; // HOF — returns function
}
const double = multiplier(2);
const triple = multiplier(3);
double(5); // 10
triple(5); // 15
```

---

**Q27. What is function composition?**

Combining multiple functions where output of one is input of next.

```js
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);
const pipe    = (...fns) => x => fns.reduce((v, f) => f(v), x);

const double  = x => x * 2;
const addOne  = x => x + 1;
const square  = x => x * x;

const transform = pipe(double, addOne, square);
transform(3); // double(3)=6, addOne(6)=7, square(7)=49
```

---

**Q28. What is the arguments object?**

Available inside regular functions (not arrow), contains all passed arguments.

```js
function sum() {
  let total = 0;
  for (let arg of arguments) total += arg;
  return total;
}
sum(1, 2, 3, 4); // 10

// arguments is array-like but NOT an array
arguments.forEach // undefined! can't use array methods
Array.from(arguments) // convert to real array

// Modern replacement: rest parameters
function sum(...nums) {
  return nums.reduce((a, b) => a + b, 0); // nums is real array
}
```

---

**Q29. What is a pure function?**

A function that: (1) given same input always returns same output, (2) has no side effects.

```js
// Pure
function add(a, b) { return a + b; }

// Impure — depends on external state
let tax = 0.1;
function price(amount) { return amount + amount * tax; } // depends on tax

// Impure — side effect (modifies external state)
function addToCart(item) { cart.push(item); } // mutates cart

// Benefits of pure functions:
// - Predictable, easy to test
// - Safe to memoize
// - Easy to parallelize
// - Foundation of functional programming
```

---

**Q30. What is a generator function?**

A function that can pause execution and resume later, yielding multiple values.

```js
function* counter() {
  let i = 0;
  while (true) {
    yield i++;  // pause here, return i
  }
}

const gen = counter();
gen.next(); // { value: 0, done: false }
gen.next(); // { value: 1, done: false }
gen.next(); // { value: 2, done: false }

// Practical: lazy sequences, infinite lists, async control flow
function* range(start, end) {
  for (let i = start; i <= end; i++) yield i;
}
[...range(1, 5)]; // [1, 2, 3, 4, 5]
```

---

## Section 3: Prototypes & OOP (Q31–45)

**Q31. What is a prototype in JavaScript?**

Every object has a hidden `__proto__` property pointing to another object (its prototype). When you access a property not found on the object, JS looks up the prototype chain.

```js
const animal = {
  breathe() { return 'breathing'; }
};

const dog = Object.create(animal); // dog's prototype = animal
dog.bark = function() { return 'woof'; };

dog.bark();    // found on dog itself
dog.breathe(); // not on dog → found on animal (prototype)
dog.toString() // not on dog or animal → found on Object.prototype

// __proto__ vs prototype:
// __proto__ — on every object, points to its prototype
// prototype — only on functions, used when function is used as constructor
```

---

**Q32. What is the prototype chain?**

The chain of prototype links JS traverses when looking up a property.

```js
function Animal(name) { this.name = name; }
Animal.prototype.breathe = function() { return 'breathing'; };

function Dog(name) {
  Animal.call(this, name); // inherit properties
}
Dog.prototype = Object.create(Animal.prototype); // inherit methods
Dog.prototype.constructor = Dog;
Dog.prototype.bark = function() { return 'woof'; };

const rex = new Dog('Rex');
rex.bark();    // Dog.prototype
rex.breathe(); // Animal.prototype
rex.toString() // Object.prototype
// Chain: rex → Dog.prototype → Animal.prototype → Object.prototype → null
```

---

**Q33. How does class work in JavaScript? Is it real OOP?**

`class` is syntactic sugar over prototype-based inheritance. Under the hood it's still prototypes.

```js
class Animal {
  constructor(name) {
    this.name = name;         // own property
  }
  breathe() {                  // added to Animal.prototype
    return `${this.name} breathes`;
  }
  static kingdom() {           // on Animal itself, not instances
    return 'Animalia';
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);              // must call before using this
    this.breed = breed;
  }
  bark() { return 'woof'; }
  breathe() {                  // override
    return super.breathe() + ' heavily';
  }
}

const d = new Dog('Rex', 'Lab');
d instanceof Dog    // true
d instanceof Animal // true — prototype chain
Animal.kingdom()    // 'Animalia' — static method
```

---

**Q34. What is the difference between Object.create() and new?**

```js
// Object.create(proto) — creates object with proto as prototype
const proto = { greet() { return 'hi'; } };
const obj = Object.create(proto);
obj.greet(); // 'hi' — found on prototype
Object.getPrototypeOf(obj) === proto; // true

// new Constructor() — 4 things happen:
// 1. New empty object created
// 2. __proto__ set to Constructor.prototype
// 3. Constructor runs with this = new object
// 4. Returns the new object (unless constructor returns an object)

function User(name) { this.name = name; }
const u = new User('Suraj');
// same as:
const u2 = Object.create(User.prototype);
User.call(u2, 'Suraj');
```

---

**Q35. What are private class fields?**

```js
class BankAccount {
  #balance = 0;       // private — only accessible inside class
  #owner;

  constructor(owner, initial) {
    this.#owner = owner;
    this.#balance = initial;
  }

  deposit(amount) { this.#balance += amount; }
  get balance() { return this.#balance; } // public getter

  #validate(amount) { return amount > 0; } // private method
}

const acc = new BankAccount('Suraj', 1000);
acc.deposit(500);
acc.balance;    // 1500
acc.#balance;   // SyntaxError — truly private
```

---

**Q36. What is method chaining?**

Pattern where methods return `this`, allowing multiple method calls in sequence.

```js
class QueryBuilder {
  constructor() { this.query = {}; }

  where(condition) {
    this.query.where = condition;
    return this;  // ← key: return this
  }
  select(fields) {
    this.query.select = fields;
    return this;
  }
  limit(n) {
    this.query.limit = n;
    return this;
  }
  build() { return this.query; }
}

const q = new QueryBuilder()
  .where({ active: true })
  .select(['name', 'email'])
  .limit(10)
  .build();
```

---

**Q37. What is mixins pattern in JavaScript?**

A way to share methods between classes without inheritance.

```js
const Serializable = (Base) => class extends Base {
  serialize() { return JSON.stringify(this); }
  static deserialize(json) { return Object.assign(new this(), JSON.parse(json)); }
};

const Validatable = (Base) => class extends Base {
  validate() { return Object.keys(this).every(k => this[k] !== null); }
};

class User { constructor(name) { this.name = name; } }

class EnhancedUser extends Serializable(Validatable(User)) {}

const u = new EnhancedUser('Suraj');
u.serialize();  // works
u.validate();   // works
```

---

**Q38. What is the difference between hasOwnProperty and 'in' operator?**

```js
const parent = { inherited: true };
const child = Object.create(parent);
child.own = true;

'own' in child              // true — checks own + prototype chain
'inherited' in child        // true — found in prototype
child.hasOwnProperty('own')         // true
child.hasOwnProperty('inherited')   // false — not own property

// Modern replacement for hasOwnProperty:
Object.hasOwn(child, 'own') // true — safer, works on objects with no prototype
```

---

**Q39. What is Object.defineProperty?**

Defines a property with fine-grained control over its behavior.

```js
const obj = {};
Object.defineProperty(obj, 'name', {
  value: 'Suraj',
  writable: false,    // can't change value
  enumerable: true,   // shows in for...in and Object.keys
  configurable: false // can't delete or redefine
});

obj.name = 'Other'; // silently fails (TypeError in strict mode)
delete obj.name;    // fails

// Getters and setters
Object.defineProperty(obj, 'fullName', {
  get() { return `${obj.first} ${obj.last}`; },
  set(val) { [obj.first, obj.last] = val.split(' '); }
});
```

---

**Q40. What are WeakMap and WeakSet?**

Like Map/Set but keys must be objects and are weakly held — they don't prevent garbage collection.

```js
const cache = new WeakMap();

function process(element) {
  if (cache.has(element)) return cache.get(element);
  const result = expensiveOperation(element);
  cache.set(element, result);
  return result;
}
// When element is removed from DOM, its cache entry is GC'd automatically
// Regular Map would hold a reference and prevent GC — memory leak!

// WeakMap is not iterable — no .keys(), .values(), .size
// Perfect for: private data, memoization, DOM metadata
```

---

**Q41. What is the difference between Map and Object?**

```
                  Object            Map
Key types:        String/Symbol     Any value (objects, functions)
Key order:        Not guaranteed    Insertion order
Size:             Manual count      .size property
Iteration:        for...in(proto)   for...of (own only)
Performance:      Good for known    Better for frequent add/delete
Default keys:     Yes (prototype)   No
JSON support:     Yes               No (need custom)
```

```js
const map = new Map();
map.set({ id: 1 }, 'user data');  // object as key!
map.set(42, 'number key');
map.size; // 2

// Converting
Object.fromEntries(map); // Map → Object
new Map(Object.entries(obj)); // Object → Map
```

---

**Q42. What is Proxy?**

Wraps an object to intercept and redefine operations on it.

```js
const handler = {
  get(target, prop) {
    console.log(`Getting ${prop}`);
    return prop in target ? target[prop] : `Property ${prop} doesn't exist`;
  },
  set(target, prop, value) {
    if (typeof value !== 'string') throw new TypeError('Only strings!');
    target[prop] = value;
    return true;
  }
};

const user = new Proxy({}, handler);
user.name = 'Suraj';    // triggers set
user.name;              // triggers get — 'Suraj'
user.age = 25;          // TypeError — not a string
user.unknown;           // "Property unknown doesn't exist"
// Used in: Vue 3 reactivity, validation, logging, ORM
```

---

**Q43. What is Reflect API?**

Complement to Proxy — provides methods for interceptable JS operations. Better than using operators directly in Proxy handlers.

```js
Reflect.get(obj, 'name')           // same as obj.name
Reflect.set(obj, 'name', 'Suraj') // same as obj.name = 'Suraj'
Reflect.has(obj, 'name')           // same as 'name' in obj
Reflect.deleteProperty(obj, 'name') // same as delete obj.name
Reflect.ownKeys(obj)               // own + Symbol keys

// In Proxy handlers, use Reflect to forward default behavior:
const proxy = new Proxy(obj, {
  set(target, prop, value) {
    console.log(`Setting ${prop}`);
    return Reflect.set(target, prop, value); // don't break default behavior
  }
});
```

---

**Q44. What is the difference between for...in and for...of?**

```js
const arr = [10, 20, 30];
const obj = { a: 1, b: 2 };

// for...in — iterates KEYS (enumerable), including prototype chain
for (let key in arr) console.log(key); // '0' '1' '2' (string keys!)
for (let key in obj) console.log(key); // 'a' 'b'
// Avoid for arrays — can pick up prototype properties

// for...of — iterates VALUES of iterables (arrays, strings, Maps, Sets)
for (let val of arr) console.log(val); // 10 20 30
for (let char of 'hi') console.log(char); // 'h' 'i'
// Cannot use on plain objects — not iterable by default
for (let [k,v] of Object.entries(obj)) console.log(k, v); // a 1, b 2
```

---

**Q45. What is Symbol.iterator?**

Protocol that makes any object iterable (usable in for...of, spread, destructuring).

```js
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    return {
      next() {
        if (current <= end) return { value: current++, done: false };
        return { value: undefined, done: true };
      }
    };
  }
}

const range = new Range(1, 5);
[...range];              // [1, 2, 3, 4, 5]
for (let n of range) {} // works!
const [a, b, c] = range; // destructuring works!
```

---

## Section 4: Async JavaScript (Q46–62)

**Q46. What is the event loop?**

JS is single-threaded. The event loop is the mechanism that lets it handle async operations without blocking.

```
Call Stack → runs synchronous code
Web APIs    → handle async (setTimeout, fetch, DOM events)
Task Queue  → callbacks from Web APIs wait here
Microtask Queue → Promise callbacks wait here (higher priority)

Event loop: when call stack is empty →
  1. Drain microtask queue completely first
  2. Take ONE task from task queue
  3. Repeat
```

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// Output: 1, 4, 3, 2
// Microtask (Promise) runs before macrotask (setTimeout) always!
```

---

**Q47. What is the difference between microtasks and macrotasks?**

```
Macrotasks (Task Queue):    setTimeout, setInterval, setImmediate, I/O, UI rendering
Microtasks (Microtask Queue): Promise.then/.catch/.finally, queueMicrotask, MutationObserver

Priority: Microtasks always drain completely before next macrotask runs.
```

```js
setTimeout(() => console.log('macro'), 0);
queueMicrotask(() => console.log('micro 1'));
Promise.resolve().then(() => console.log('micro 2'));
queueMicrotask(() => console.log('micro 3'));
// Output: micro 1, micro 2, micro 3, macro
```

---

**Q48. What is a callback and what is callback hell?**

Callback: a function passed as argument to another function, called when async operation completes.

```js
// Callback pattern
fs.readFile('a.txt', (err, data) => {
  if (err) handle(err);
  fs.readFile('b.txt', (err2, data2) => {  // nested
    if (err2) handle(err2);
    fs.writeFile('c.txt', data + data2, (err3) => {  // deeper nested
      // callback hell — pyramid of doom
    });
  });
});

// Problems: hard to read, error handling at each level, hard to parallelize
```

---

**Q49. What is a Promise?**

An object representing a future value. Has three states: pending, fulfilled, rejected.

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    Math.random() > 0.5 ? resolve('success') : reject('failed');
  }, 1000);
});

promise
  .then(value => console.log(value))    // fulfilled
  .catch(err => console.error(err))     // rejected
  .finally(() => console.log('done')); // always runs

// Promise is resolved — value is wrapped and doesn't change
// then/catch always return a NEW promise — enables chaining
```

---

**Q50. What is Promise chaining?**

Each `.then()` returns a new Promise, enabling sequential async operations cleanly.

```js
// Instead of nested callbacks:
fetchUser(id)
  .then(user => fetchPosts(user.id))
  .then(posts => fetchComments(posts[0].id))
  .then(comments => console.log(comments))
  .catch(err => console.error(err)); // ONE catch handles all errors above

// If you return a value from .then(), it's wrapped in Promise.resolve()
// If you return a Promise, chaining waits for it to resolve
Promise.resolve(1)
  .then(n => n + 1)    // returns 2
  .then(n => n * 3)    // returns 6
  .then(console.log);  // 6
```

---

**Q51. What is Promise.all vs Promise.race vs Promise.allSettled vs Promise.any?**

```js
const p1 = fetch('/api/user');
const p2 = fetch('/api/posts');
const p3 = fetch('/api/comments');

// Promise.all — waits for ALL, rejects if ANY rejects
const [user, posts] = await Promise.all([p1, p2]);

// Promise.race — resolves/rejects as soon as FIRST settles
const fastest = await Promise.race([p1, p2, p3]);

// Promise.allSettled — waits for ALL, never rejects, gives status of each
const results = await Promise.allSettled([p1, p2, p3]);
// [{status:'fulfilled',value:...}, {status:'rejected',reason:...}]

// Promise.any — resolves with FIRST fulfilled, rejects only if ALL reject
const first = await Promise.any([p1, p2, p3]); // AggregateError if all fail
```

---

**Q52. What is async/await?**

Syntactic sugar over Promises. Makes async code look synchronous.

```js
// Promise version
function getUser(id) {
  return fetch(`/api/users/${id}`)
    .then(res => res.json())
    .then(user => user.name);
}

// async/await version — same thing
async function getUser(id) {
  const res = await fetch(`/api/users/${id}`);
  const user = await res.json();
  return user.name; // automatically wrapped in Promise.resolve()
}

// Error handling
async function safe() {
  try {
    const data = await riskyOperation();
    return data;
  } catch (err) {
    console.error(err);
  }
}
```

---

**Q53. Common async/await mistakes**

```js
// ❌ Sequential when should be parallel
async function bad() {
  const user = await getUser();    // 1s
  const posts = await getPosts();  // 1s — waits for user, but shouldn't!
}                                   // total: 2s

// ✅ Parallel
async function good() {
  const [user, posts] = await Promise.all([getUser(), getPosts()]); // 1s
}

// ❌ async in forEach doesn't work as expected
async function bad2() {
  items.forEach(async item => {
    await process(item); // forEach doesn't wait for these!
  });
}

// ✅ Use for...of or Promise.all
async function good2() {
  for (const item of items) { await process(item); } // sequential
  await Promise.all(items.map(process));              // parallel
}
```

---

**Q54. What is the difference between Promise.resolve() and new Promise()?**

```js
// Promise.resolve(val) — creates already-resolved Promise
const p = Promise.resolve(42);
p.then(console.log); // 42 immediately (as microtask)

// new Promise() — for wrapping callback-based APIs
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Promisifying a callback API
function readFile(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}
// Node.js has util.promisify for this
```

---

**Q55. What is async iteration?**

Iterate over async data sources with `for await...of`.

```js
async function* generateNumbers() {
  for (let i = 0; i < 5; i++) {
    await delay(100); // wait 100ms between each
    yield i;
  }
}

async function main() {
  for await (const num of generateNumbers()) {
    console.log(num); // 0, 1, 2, 3, 4 with 100ms gaps
  }
}

// Real use: streaming API responses, paginated data, reading large files
```

---

**Q56. How do you handle errors in Promise chains properly?**

```js
// ❌ Missing error handling — unhandled rejection
fetch('/api/data')
  .then(res => res.json())
  .then(data => process(data));

// ✅ Catch at end
fetch('/api/data')
  .then(res => {
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
  })
  .then(data => process(data))
  .catch(err => handleError(err));

// ❌ catch only catches errors BEFORE it in chain
promise
  .catch(err => console.log('caught')) // this catches above
  .then(() => { throw new Error('new error') }) // this is NOT caught!

// ✅ catch at the very end
promise
  .then(...)
  .then(...)
  .catch(err => console.error(err)) // catches everything above
  .finally(() => cleanup());
```

---

**Q57. What is an AbortController?**

Cancel fetch requests or any async operation.

```js
const controller = new AbortController();

// Cancel after 5 seconds
const timeout = setTimeout(() => controller.abort(), 5000);

try {
  const res = await fetch('/api/data', { signal: controller.signal });
  clearTimeout(timeout);
  return await res.json();
} catch (err) {
  if (err.name === 'AbortError') {
    console.log('Request cancelled');
  }
}
```

---

**Q58. What is the difference between setTimeout(fn, 0) and Promise.resolve()?**

Both defer execution, but Promise.resolve() is a microtask (higher priority).

```js
setTimeout(() => console.log('timeout'), 0);  // macrotask queue
Promise.resolve().then(() => console.log('promise')); // microtask queue

// Output: promise, timeout
// Microtasks drain before any macrotask runs

// Use case: deferring work to after current synchronous code finishes,
// but Promise.resolve() is slightly faster (fewer queue hops)
```

---

**Q59. What is a memory leak in async code?**

```js
// ❌ Event listeners never removed — memory leak
class Component {
  init() {
    this.handler = () => this.update();
    window.addEventListener('resize', this.handler); // registered
  }
  // never calls removeEventListener!
}

// ✅ Always clean up
class Component {
  init() {
    this.handler = () => this.update();
    window.addEventListener('resize', this.handler);
  }
  destroy() {
    window.removeEventListener('resize', this.handler); // clean up
  }
}

// ❌ Interval never cleared
const id = setInterval(sendHeartbeat, 1000);
// ✅
clearInterval(id); // when done
```

---

**Q60. What are the phases of a Promise?**

```
Pending   → initial state, neither fulfilled nor rejected
Fulfilled → operation completed successfully, .then() callbacks run
Rejected  → operation failed, .catch() callbacks run
Settled   → either fulfilled OR rejected (not pending)

Key rules:
- A settled Promise CANNOT change state
- Handlers (.then/.catch) are always called asynchronously (as microtask)
- Even Promise.resolve(42).then(fn) — fn runs asynchronously!
- Returning a Promise from .then() unwraps it (no nested Promises)
```

---

**Q61. What is the difference between throw in async function vs rejected promise?**

```js
// These are equivalent in async functions:
async function a() { throw new Error('oops'); }
async function b() { return Promise.reject(new Error('oops')); }

// Both return a rejected Promise
// Both are caught with .catch() or try/catch with await

// But outside async — you CANNOT throw from a callback
setTimeout(() => {
  throw new Error('uncaught!'); // NOT caught by outer try/catch
  // Use process.on('uncaughtException') or always return rejected Promises
}, 0);
```

---

**Q62. What is queueMicrotask?**

Schedule a microtask without creating a Promise.

```js
queueMicrotask(() => {
  console.log('runs as microtask');
});
// Same priority as Promise.then(), but more explicit and lighter weight

// Practical use: deferring work to end of current task without full Promise overhead
function notify(callback) {
  queueMicrotask(callback); // run after current sync code, before any macrotask
}
```

---

## Section 5: Modern JavaScript Features (Q63–78)

**Q63. What is destructuring and its advanced uses?**

```js
// Array destructuring
const [a, b, ...rest] = [1, 2, 3, 4, 5];
const [, second] = [1, 2, 3]; // skip first

// Object destructuring
const { name, age = 25, address: { city } } = user; // rename, default, nested
const { name: userName } = user; // rename to userName

// Function params
function greet({ name, age = 0 }) { return `${name} is ${age}`; }

// Swap variables
let x = 1, y = 2;
[x, y] = [y, x];

// Destructure in loops
for (const [key, value] of Object.entries(obj)) { }
```

---

**Q64. What is the spread operator vs rest parameter?**

Same syntax (`...`), different context.

```js
// Spread — EXPANDS an iterable
const arr = [1, 2, 3];
Math.max(...arr);            // expands array to arguments
const copy = [...arr];       // shallow copy array
const merged = [...a, ...b]; // merge arrays
const objCopy = { ...obj };  // shallow copy object
const merged2 = { ...defaults, ...overrides }; // merge objects

// Rest — COLLECTS remaining into array
function fn(first, second, ...others) { } // others is array
const { a, ...remaining } = obj;          // remaining has all except a
const [head, ...tail] = arr;              // head = 1, tail = [2,3]
```

---

**Q65. What is optional chaining?**

Access deeply nested properties without throwing if intermediate value is null/undefined.

```js
const user = { address: null };

// Old way
const city = user && user.address && user.address.city;

// Optional chaining
const city = user?.address?.city;   // undefined, no error
const len  = arr?.length;            // undefined if arr is null
const val  = obj?.method?.();        // call method only if it exists
const item = arr?.[0];               // access index safely

// With nullish coalescing
const city = user?.address?.city ?? 'Unknown';
```

---

**Q66. What is nullish coalescing (??) vs OR (||)?**

```js
// || returns right side if left is FALSY (0, '', false, null, undefined)
const port = config.port || 3000;
// Problem: config.port = 0 → returns 3000 (wrong! 0 is valid)

// ?? returns right side only if left is NULL or UNDEFINED
const port = config.port ?? 3000;
// config.port = 0 → returns 0 (correct)
// config.port = '' → returns '' (correct)
// config.port = null → returns 3000 (correct)
// config.port = undefined → returns 3000 (correct)

// ??= nullish assignment
user.settings ??= {}; // only assigns if null or undefined
```

---

**Q67. What are template literals and tagged templates?**

```js
// Template literals
const name = 'Suraj';
const greeting = `Hello, ${name}! ${1 + 1} things.`; // expression inside ${}

// Multi-line without \n
const html = `
  <div>
    <p>${content}</p>
  </div>
`;

// Tagged templates — process template before output
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] ? `<mark>${values[i]}</mark>` : '');
  }, '');
}

const city = 'Bengaluru';
highlight`Hello from ${city} in ${'India'}`
// 'Hello from <mark>Bengaluru</mark> in <mark>India</mark>'
// Used in: styled-components CSS, GraphQL gql``, SQL sanitization
```

---

**Q68. What is Object.keys vs Object.values vs Object.entries?**

```js
const obj = { a: 1, b: 2, c: 3 };

Object.keys(obj)    // ['a', 'b', 'c']
Object.values(obj)  // [1, 2, 3]
Object.entries(obj) // [['a',1], ['b',2], ['c',3]]

// Transform object values
const doubled = Object.fromEntries(
  Object.entries(obj).map(([k, v]) => [k, v * 2])
); // { a: 2, b: 4, c: 6 }

// All three only return OWN ENUMERABLE properties
// None include prototype chain
```

---

**Q69. What is a Set?**

Collection of unique values.

```js
const set = new Set([1, 2, 2, 3, 3, 3]);
set.size; // 3 — duplicates removed

set.add(4);
set.has(2);   // true
set.delete(1);

// Remove duplicates from array
const unique = [...new Set([1,2,2,3,3])]; // [1,2,3]

// Set operations
const a = new Set([1,2,3]);
const b = new Set([2,3,4]);
const union        = new Set([...a, ...b]);           // {1,2,3,4}
const intersection = new Set([...a].filter(x => b.has(x))); // {2,3}
const difference   = new Set([...a].filter(x => !b.has(x))); // {1}
```

---

**Q70. What is the difference between structuredClone and JSON methods for deep copy?**

```js
// JSON.parse(JSON.stringify(obj))
// ✅ Works for plain objects, arrays, primitives
// ❌ Loses: functions, undefined, Symbol, Date (converts to string), Map, Set, circular refs

const orig = { date: new Date(), fn: () => {}, val: undefined };
const copy = JSON.parse(JSON.stringify(orig));
// copy = { date: '2024-01-01T...' (string!) } — fn and val lost!

// structuredClone(obj) — modern, available in Node 17+ and all modern browsers
// ✅ Works for: Date, Map, Set, ArrayBuffer, RegExp
// ❌ Loses: functions, DOM nodes, class instances (lose prototype)
const d = structuredClone({ date: new Date(), map: new Map() }); // works!
structuredClone({ fn: () => {} }); // DataCloneError!
```

---

**Q71. What are Iterators and the Iterator Protocol?**

An object is an iterator if it has a `next()` method returning `{ value, done }`.

```js
function createIterator(arr) {
  let index = 0;
  return {
    next() {
      if (index < arr.length) {
        return { value: arr[index++], done: false };
      }
      return { value: undefined, done: true };
    }
  };
}

const it = createIterator([10, 20, 30]);
it.next(); // { value: 10, done: false }
it.next(); // { value: 20, done: false }
it.next(); // { value: 30, done: false }
it.next(); // { value: undefined, done: true }
```

---

**Q72. What is logical assignment operators?**

```js
// ||=  assign if left is falsy
a ||= b;  // same as: a = a || b;

// &&=  assign if left is truthy
a &&= b;  // same as: a = a && b;

// ??=  assign if left is nullish
a ??= b;  // same as: a = a ?? b;

// Practical
user.settings ??= {};          // init if not set
arr.length &&= 0;              // clear if non-empty
config.debug ||= isDev;        // set default if falsy
```

---

**Q73. What is the Intl API?**

Built-in internationalization — format numbers, dates, currencies for different locales.

```js
// Number formatting
new Intl.NumberFormat('en-IN', { style: 'currency', currency: 'INR' })
  .format(1234567); // '₹12,34,567.00' (Indian numbering system!)

// Date formatting
new Intl.DateTimeFormat('en-IN', { dateStyle: 'long' })
  .format(new Date()); // '31 May 2026'

// Relative time
new Intl.RelativeTimeFormat('en').format(-2, 'days'); // '2 days ago'

// Pluralization
new Intl.PluralRules('en').select(1); // 'one'
new Intl.PluralRules('en').select(2); // 'other'
```

---

**Q74. What is globalThis?**

Unified way to access global object across environments.

```js
// Different environments have different global objects:
// Browser: window
// Node.js: global
// Web Worker: self

// Before:
const global = typeof window !== 'undefined' ? window :
               typeof global !== 'undefined' ? global : self;

// After:
globalThis.setTimeout;   // works everywhere
globalThis.fetch;        // works everywhere
```

---

**Q75. What is Error handling best practices?**

```js
// Custom error classes
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
    Error.captureStackTrace(this, ValidationError);
  }
}

try {
  throw new ValidationError('Invalid email', 'email');
} catch (err) {
  if (err instanceof ValidationError) {
    console.log(err.field); // 'email'
  } else if (err instanceof NetworkError) {
    // handle differently
  } else {
    throw err; // re-throw unknown errors!
  }
}

// Error cause (ES2022)
catch (err) {
  throw new Error('Failed to load user', { cause: err }); // wrap with context
}
```

---

**Q76. What is Object.fromEntries?**

Creates object from array of [key, value] pairs. Inverse of Object.entries.

```js
const entries = [['a', 1], ['b', 2], ['c', 3]];
Object.fromEntries(entries); // { a: 1, b: 2, c: 3 }

// Transform object values (common interview question)
const prices = { apple: 100, banana: 50, mango: 200 };
const discounted = Object.fromEntries(
  Object.entries(prices).map(([item, price]) => [item, price * 0.9])
);
// { apple: 90, banana: 45, mango: 180 }

// Convert Map to Object
const map = new Map([['a', 1], ['b', 2]]);
Object.fromEntries(map); // { a: 1, b: 2 }
```

---

**Q77. What is Array.flat and Array.flatMap?**

```js
// flat — flatten nested arrays
[1, [2, [3, [4]]]].flat();    // [1, 2, [3, [4]]] — one level
[1, [2, [3, [4]]]].flat(2);   // [1, 2, 3, [4]] — two levels
[1, [2, [3, [4]]]].flat(Infinity); // [1, 2, 3, 4] — all levels

// flatMap — map then flat (one level)
const sentences = ['Hello World', 'Foo Bar'];
sentences.flatMap(s => s.split(' ')); // ['Hello', 'World', 'Foo', 'Bar']

// More efficient than .map().flat():
arr.flatMap(fn); // one pass
arr.map(fn).flat(); // two passes
```

---

**Q78. What is at() method on arrays?**

```js
const arr = [1, 2, 3, 4, 5];

// Old way to get last element:
arr[arr.length - 1]; // 5

// New way:
arr.at(-1);  // 5 — negative index from end!
arr.at(-2);  // 4
arr.at(0);   // 1
arr.at(1);   // 2

// Also works on strings:
'hello'.at(-1); // 'o'
'hello'.at(0);  // 'h'
```

---

## Section 6: Hard Concepts (Q79–100)

**Q79. Explain the output of this code:**

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// Output: 3, 3, 3

// Why? var is function-scoped, one shared i. By the time setTimeout runs,
// loop is done and i = 3.

// Fix 1: use let (block-scoped, new i each iteration)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0); // 0, 1, 2
}

// Fix 2: IIFE to capture i
for (var i = 0; i < 3; i++) {
  ((i) => setTimeout(() => console.log(i), 0))(i);
}
```

---

**Q80. What is the output and why?**

```js
console.log(1);
setTimeout(() => console.log(2), 0);
Promise.resolve().then(() => console.log(3));
setTimeout(() => console.log(4), 0);
Promise.resolve().then(() => console.log(5));
console.log(6);

// Output: 1, 6, 3, 5, 2, 4
// 1, 6 → synchronous
// 3, 5 → microtasks (both Promise.then) — drain completely before macrotasks
// 2, 4 → macrotasks (both setTimeout) — in order
```

---

**Q81. What happens when you use new with an arrow function?**

```js
const Foo = () => {};
new Foo(); // TypeError: Foo is not a constructor

// Arrow functions:
// - Have no prototype property
// - Have no [[Construct]] internal method
// - Cannot be used with new
// - This is by design — arrow functions are for callbacks, not constructors
```

---

**Q82. What is the output?**

```js
const obj = {
  name: 'Suraj',
  greet: function() {
    console.log(this.name);
    const inner = function() {
      console.log(this.name); // what is this here?
    };
    inner();
  }
};
obj.greet();
// Output:
// 'Suraj'       ← greet called as method, this = obj
// undefined     ← inner called as plain function, this = global/undefined(strict)

// Fix: use arrow function for inner
const inner = () => console.log(this.name); // inherits this from greet
```

---

**Q83. What is prototype pollution?**

A security vulnerability where attacker modifies Object.prototype, affecting all objects.

```js
// Vulnerable code
function merge(target, source) {
  for (let key in source) {
    target[key] = source[key]; // dangerous!
  }
}

const malicious = JSON.parse('{"__proto__": {"isAdmin": true}}');
merge({}, malicious);

const victim = {};
console.log(victim.isAdmin); // true! polluted via Object.prototype

// Fix: use Object.hasOwn() to check, or Object.create(null) for no-prototype objects
function safeMerge(target, source) {
  for (let key of Object.keys(source)) { // Object.keys — no prototype
    if (key === '__proto__') continue;    // explicit guard
    target[key] = source[key];
  }
}
```

---

**Q84. What is the difference between Object.assign and spread for objects?**

```js
// Mostly same — both shallow copy, both copy enumerable own properties
const a = { x: 1 };
const copy1 = { ...a };
const copy2 = Object.assign({}, a);

// Difference 1: Object.assign triggers setters
// Difference 2: spread copies own enumerable, Object.assign uses [[Set]] which triggers setters

// Difference 3: Object.assign mutates target
Object.assign(target, source); // modifies target!
const copy = { ...source };    // creates new object

// For merging: spread is preferred (immutable, readable)
const merged = { ...defaults, ...overrides };
```

---

**Q85. What is Function.prototype.toString?**

```js
function add(a, b) { return a + b; }
add.toString(); // "function add(a, b) { return a + b; }"

// Actual use cases:
// 1. Serializing functions (limited — won't capture closure scope)
// 2. Checking if native function
console.log(Array.isArray.toString()); // "function isArray() { [native code] }"
// 3. Some frameworks use it to extract parameter names for DI
```

---

**Q86. What is tail call optimization?**

In strict mode, if last thing a function does is call another function (tail call), JS engine can reuse the current stack frame instead of creating a new one — preventing stack overflow in deep recursion.

```js
'use strict';

// Not tail call — must return and add 1, so stack frame kept
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // multiply after recursive call returns
}

// Tail call optimized — last operation is just the function call
function factorial(n, acc = 1) {
  if (n <= 1) return acc;
  return factorial(n - 1, n * acc); // nothing to do after this returns
}
// Note: TCO support in JS engines is inconsistent (V8 removed it)
```

---

**Q87. What is the difference between microtask starvation and task starvation?**

```js
// Microtask starvation — if you keep adding microtasks, macrotasks never run
function forever() {
  Promise.resolve().then(forever); // infinite microtask chain
}
forever();
// setTimeout callbacks (macrotasks) will NEVER run!
// The microtask queue drains completely before each macrotask

// Task starvation — long-running sync code blocks everything
for (let i = 0; i < 1e9; i++) { } // blocks for seconds — nothing else runs
```

---

**Q88. What is lazy evaluation?**

Delaying computation until result is actually needed.

```js
// Eager — computed immediately even if not used
const result = heavyComputation(); // runs even if result never used

// Lazy — only computed when accessed
const lazy = () => heavyComputation(); // wraps in function
// ... later:
if (needResult) {
  const result = lazy(); // computed only now
}

// Practical: lazy property with getter
const obj = {
  get expensive() {
    const val = heavyCompute();
    Object.defineProperty(this, 'expensive', { value: val }); // cache it
    return val;
  }
};
obj.expensive; // computed and cached on first access
obj.expensive; // returns cached value
```

---

**Q89. What is the difference between synchronous and asynchronous iteration?**

```js
// Synchronous iterator — values available immediately
const syncIter = [1, 2, 3][Symbol.iterator]();
syncIter.next(); // { value: 1, done: false } — immediately

// Asynchronous iterator — each value may require waiting
async function* asyncIter() {
  yield await fetch('/api/1').then(r => r.json());
  yield await fetch('/api/2').then(r => r.json());
}

// Consume:
for await (const item of asyncIter()) {
  console.log(item); // waits for each fetch
}

// Real use: streaming HTTP responses, file reading, database cursors
const stream = response.body.getReader();
// stream is async iterable in modern browsers
```

---

**Q90. What are labeled statements?**

```js
// Labels let you break/continue outer loops from inner loops
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) break outer; // breaks OUTER loop, not just inner
    console.log(i, j);
  }
}
// 0 0, 0 1, 0 2, 1 0 — stops at i=1, j=1

// continue outer
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) continue outer; // skip rest of inner, go to next outer iteration
  }
}
```

---

**Q91. What happens when toString and valueOf are defined?**

```js
const money = {
  amount: 100,
  currency: 'INR',
  toString() { return `${this.amount} ${this.currency}`; },
  valueOf()  { return this.amount; }
};

`I have ${money}`;  // 'I have 100 INR' — toString used in string context
money + 50;         // 150 — valueOf used in numeric context
money > 50;         // true — valueOf used in comparison
+money;             // 100 — valueOf
```

---

**Q92. What is the void operator?**

```js
void 0       // undefined — always returns undefined
void 'hello' // undefined
void(0)      // undefined

// Uses:
// 1. href="javascript:void(0)" — prevent page reload, deprecated
// 2. Check for original undefined (before ES5, undefined was writable!)
// 3. Intentionally discard return value of async functions
void someAsyncFunction(); // explicitly discarding the Promise
```

---

**Q93. What is string interning?**

JS engines may reuse the same memory for identical string literals (implementation detail, not spec).

```js
const a = 'hello';
const b = 'hello';
a === b; // true — same value (and likely same memory in engine)

// Strings created dynamically may or may not be interned:
const c = 'hel' + 'lo'; // engines usually optimize this at compile time
const d = ['h','e','l','l','o'].join(''); // may be different allocation
c === d; // true — because strings are compared by value, not reference
// Unlike objects: {} === {} is false!
```

---

**Q94. What is the difference between Array.from and spread for converting iterables?**

```js
const str = 'hello';
[...str];                  // ['h','e','l','l','o']
Array.from(str);           // ['h','e','l','l','o']

// Array.from has a map function:
Array.from({length: 5}, (_, i) => i * 2); // [0, 2, 4, 6, 8]
Array.from('hello', c => c.toUpperCase()); // ['H','E','L','L','O']

// Spread doesn't work on non-iterables:
Array.from({length: 3}); // [undefined, undefined, undefined] — works!
[...{length: 3}];        // TypeError — not iterable

// Array.from also converts NodeLists (DOM):
Array.from(document.querySelectorAll('div')); // proper array
```

---

**Q95. What are the gotchas with object property shorthand?**

```js
const name = 'Suraj';
const age = 25;

// Shorthand
const user = { name, age }; // same as { name: name, age: age }

// Method shorthand
const obj = {
  greet() { return 'hi'; },            // shorthand method
  greet: function() { return 'hi'; },  // same
  greet: () => 'hi',                   // arrow — no own this!
};

// Computed property names
const key = 'dynamic';
const obj2 = {
  [key]: 'value',           // { dynamic: 'value' }
  [`prefix_${key}`]: true,  // { prefix_dynamic: true }
};
```

---

**Q96. What is the difference between delete and setting to undefined?**

```js
const obj = { a: 1, b: 2, c: 3 };

obj.b = undefined;
console.log('b' in obj);    // true — property still EXISTS, value is undefined
Object.keys(obj);            // ['a', 'b', 'c'] — b is still there

delete obj.c;
console.log('c' in obj);    // false — property REMOVED
Object.keys(obj);            // ['a', 'b'] — c is gone

// delete returns true for successful deletion, false if property non-configurable
delete obj.a; // true
// delete cannot delete variables or function declarations
delete name;  // false (or error in strict mode)
```

---

**Q97. What is structural sharing in immutable updates?**

```js
// When you update a nested object immutably, JS creates new objects
// only for the changed path — unchanged parts are shared (same reference)

const state = {
  user: { name: 'Suraj', age: 25 },
  posts: [1, 2, 3],       // this is NOT changed
  settings: { theme: 'dark' } // this is NOT changed
};

// Update only user.age
const newState = {
  ...state,                           // spreads same reference to posts and settings!
  user: { ...state.user, age: 26 }   // only user is new object
};

newState.posts === state.posts;       // true — same array reference (shared)
newState.settings === state.settings; // true — same object reference (shared)
newState.user === state.user;         // false — new object created
// This is how Redux/Immer work efficiently
```

---

**Q98. What are the rules of the new keyword?**

```js
// new does 4 things:
function Person(name) {
  // 1. New empty object created: this = {}
  // 2. this.__proto__ = Person.prototype
  this.name = name; // 3. Constructor runs with this = new object
  // 4. Returns this (unless constructor explicitly returns an object)
}

// If constructor returns a primitive, it's ignored (new object returned)
function Weird() {
  this.x = 1;
  return 42; // ignored — primitive
}
new Weird(); // { x: 1 }

// If constructor returns an object, THAT is returned!
function Weird2() {
  this.x = 1;
  return { y: 2 }; // returned instead of 'this'!
}
new Weird2(); // { y: 2 } ← not { x: 1 }!
```

---

**Q99. What is the difference between enumerable, configurable, writable property attributes?**

```js
const obj = {};
Object.defineProperty(obj, 'x', {
  value: 42,
  writable: false,      // can't change value
  enumerable: false,    // won't appear in for...in or Object.keys()
  configurable: false   // can't delete, can't redefine attributes
});

obj.x = 99;           // silently fails (TypeError in strict mode)
Object.keys(obj);      // [] — x not enumerable!
'x' in obj;            // true — 'in' checks regardless of enumerable
delete obj.x;          // fails — not configurable

// Default when you do obj.y = 5:
// writable: true, enumerable: true, configurable: true
```

---

**Q100. What is tail recursion, trampolining, and how to avoid stack overflow?**

```js
// Problem: deep recursion causes stack overflow
function sum(n) {
  if (n <= 0) return 0;
  return n + sum(n - 1); // stack grows with each call
}
sum(100000); // RangeError: Maximum call stack size exceeded

// Solution 1: Iteration
function sumIter(n) {
  let total = 0;
  for (let i = n; i > 0; i--) total += i;
  return total;
}

// Solution 2: Trampolining — convert recursion to iteration automatically
function trampoline(fn) {
  return function(...args) {
    let result = fn(...args);
    while (typeof result === 'function') {
      result = result();
    }
    return result;
  };
}

function sum(n, acc = 0) {
  if (n <= 0) return acc;
  return () => sum(n - 1, acc + n); // return thunk instead of recursive call
}

const safeSum = trampoline(sum);
safeSum(100000); // works! no stack overflow
```

---

## Quick Reference: Most Asked Topics

```
Closures:          Q16, Q17, Q18, Q79
Prototype:         Q31, Q32, Q33, Q34
this keyword:      Q21, Q22, Q23, Q82
Memory:            Q1, Q2, Q3
Async:             Q46-Q62
ES6+ Features:     Q63-Q78
Tricky Output Qs:  Q79, Q80, Q81, Q82, Q87
```

---

*100 questions done. Read one section per day. For each question — explain it out loud like you're teaching a junior. That's your interview voice.*
