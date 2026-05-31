# JavaScript Classes — Complete Interview Guide

> Everything you need to crack JS OOP interviews

---

## 1. What is a Class?

A class is a **blueprint** for creating objects. It bundles data (properties) and behaviour (methods) together.  
Under the hood, JS classes are **syntactic sugar** over prototype-based inheritance.

> ⚠️ Classes are NOT hoisted like functions. Declare before use.

```js
class Person {
  constructor(name, age) {
    this.name = name; // instance property
    this.age = age;
  }
  greet() {           // instance method
    return `Hello, I am ${this.name}`;
  }
}

const p = new Person('Suraj', 25);
console.log(p.greet()); // Hello, I am Suraj
```

🎯 **Interview:** What is the difference between a class and an object?  
**Answer:** Class = blueprint. Object = instance created from that blueprint using `new`.

---

## 2. constructor()

A special method called automatically when you do `new ClassName()`.  
Used to initialise properties. Every class can have **only ONE constructor**.

```js
class Car {
  constructor(brand, model) {
    this.brand = brand;
    this.model = model;
  }
}
const c = new Car('Toyota', 'Camry');
```

🎯 **Interview:** What happens if you don't write a constructor?  
**Answer:** JavaScript provides a default empty constructor automatically.

---

## 3. Instance vs Static

### Instance Methods & Properties

Belong to each individual object. Every `new` call creates its own copy of properties.

```js
class Counter {
  constructor() {
    this.count = 0; // each object has its own
  }
  increment() {
    this.count++;
  }
}

const a = new Counter();
const b = new Counter();
a.increment();
console.log(a.count); // 1
console.log(b.count); // 0 ← separate from a
```

### Static Methods & Properties

Belong to the **CLASS itself**, not any instance. Called on the class directly.  
Use for utility functions or shared data.

```js
class MathHelper {
  static PI = 3.14159;        // static property

  static add(a, b) {          // static method
    return a + b;
  }
}

console.log(MathHelper.PI);        // 3.14159
console.log(MathHelper.add(2, 3)); // 5

// ❌ Cannot call on instance:
const m = new MathHelper();
m.add(1, 2); // TypeError
```

🎯 **Interview:** When would you use a static method?  
**Answer:** For utility/helper functions that don't need instance data — e.g., factory methods, validators, `Date.now()`, `Array.isArray()`.

---

## 4. Getters & Setters

Special methods that let you define **computed properties** and add validation when reading or writing a value.

```js
class Temperature {
  constructor(celsius) {
    this._celsius = celsius;
  }

  get fahrenheit() {                    // getter
    return this._celsius * 9 / 5 + 32;
  }

  set celsius(value) {                  // setter with validation
    if (value < -273.15) throw new Error('Below absolute zero!');
    this._celsius = value;
  }
}

const t = new Temperature(100);
console.log(t.fahrenheit); // 212 ← called like a property, not t.fahrenheit()
t.celsius = 25;            // uses setter
```

> 💡 Prefix private-by-convention properties with `_` (underscore). For truly private, use `#` (see section 5).

🎯 **Interview:** Difference between a getter and a regular method?  
**Answer:** A getter is accessed like a property (no parentheses). Ideal for computed/derived values. A regular method needs explicit `()`.

---

## 5. Private Fields & Methods (`#`)

Introduced in **ES2022**. Fields prefixed with `#` are truly private — cannot be accessed outside the class, not even by subclasses.

```js
class BankAccount {
  #balance = 0;         // private field
  #pin;

  constructor(initialBalance, pin) {
    this.#balance = initialBalance;
    this.#pin = pin;
  }

  #validatePin(pin) {   // private method
    return this.#pin === pin;
  }

  withdraw(amount, pin) {
    if (!this.#validatePin(pin)) throw new Error('Wrong PIN');
    if (amount > this.#balance) throw new Error('Insufficient funds');
    this.#balance -= amount;
  }

  get balance() { return this.#balance; }
}

const acc = new BankAccount(1000, 1234);
acc.withdraw(200, 1234);
console.log(acc.balance);  // 800
console.log(acc.#balance); // ❌ SyntaxError — truly private
```

🎯 **Interview:** Difference between `_` prefix and `#` private?  
**Answer:** `_` is just a naming convention — field is still publicly accessible. `#` is enforced by the JS engine — accessing it outside throws a `SyntaxError`.

---

## 6. Inheritance (`extends` & `super`)

A child class inherits all properties and methods from a parent using `extends`.  
Use `super()` to call the parent constructor.

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound.`;
  }
}

class Dog extends Animal {       // Dog inherits from Animal
  constructor(name, breed) {
    super(name);                 // MUST call super() before this.*
    this.breed = breed;
  }
  speak() {                      // method overriding
    return `${this.name} barks!`;
  }
  info() {
    return `${super.speak()} Breed: ${this.breed}`; // call parent method
  }
}

const d = new Dog('Bruno', 'Labrador');
console.log(d.speak()); // Bruno barks!
console.log(d.info());  // Bruno makes a sound. Breed: Labrador
```

> ⚠️ Forgetting `super()` throws: `ReferenceError: Must call super constructor before accessing 'this'`

🎯 **Interview:** Can a child class access `#` private fields of the parent?  
**Answer:** No. Private fields are scoped strictly to the class they're defined in. Subclasses cannot access them.

---

## 7. Method Overriding & Polymorphism

A child class can redefine a parent method. The child's version runs when called on a child instance.

```js
class Shape {
  area() { return 0; }
  toString() { return `Area: ${this.area()}`; }
}

class Circle extends Shape {
  constructor(r) { super(); this.r = r; }
  area() { return Math.PI * this.r ** 2; }  // override
}

class Rectangle extends Shape {
  constructor(w, h) { super(); this.w = w; this.h = h; }
  area() { return this.w * this.h; }         // override
}

const shapes = [new Circle(5), new Rectangle(4, 6)];
shapes.forEach(s => console.log(s.toString()));
// Area: 78.53...
// Area: 24
```

---

## 8. `instanceof` & `typeof`

```js
const d = new Dog('Rex', 'Pug');

console.log(d instanceof Dog);    // true
console.log(d instanceof Animal); // true  ← inherits
console.log(d instanceof Object); // true  ← everything is Object

console.log(typeof d);            // 'object'   (not 'Dog'!)
console.log(typeof Dog);          // 'function' ← classes are functions
```

🎯 **Interview:** `typeof` an instance gives `'object'` — how do you get the class name?  
**Answer:**
```js
console.log(d.constructor.name); // 'Dog'
```

---

## 9. Mixins (Multiple Behaviour)

JS supports only **single inheritance**. To reuse behaviour from multiple sources, use **mixins** — functions that add methods to a class.

```js
const Serializable = (Base) => class extends Base {
  serialize() { return JSON.stringify(this); }
};

const Timestamped = (Base) => class extends Base {
  constructor(...args) {
    super(...args);
    this.createdAt = new Date();
  }
};

class User extends Timestamped(Serializable(class {})) {
  constructor(name) { super(); this.name = name; }
}

const u = new User('Suraj');
console.log(u.serialize());  // {"name":"Suraj","createdAt":"..."}
console.log(u.createdAt);    // Date object
```

> 💡 "Favour composition over inheritance" — mixins are that composition pattern.

---

## 10. Abstract Classes (Pattern)

JavaScript has no built-in `abstract` keyword. Simulate it using `new.target`.

```js
class AbstractPayment {
  constructor() {
    if (new.target === AbstractPayment) {
      throw new Error('Cannot instantiate abstract class');
    }
  }
  process() {  // abstract method
    throw new Error('process() must be implemented');
  }
}

class UPIPayment extends AbstractPayment {
  process(amount) {
    return `Paid ₹${amount} via UPI`;
  }
}

// new AbstractPayment(); ❌ Error
const pay = new UPIPayment();
console.log(pay.process(500)); // Paid ₹500 via UPI
```

🎯 **Interview:** What is `new.target`?  
**Answer:** `new.target` refers to the constructor directly invoked with `new`. Inside a parent constructor, if `new.target === Parent`, someone tried to directly instantiate the abstract base — which you can block.

---

## 11. Prototype Chain

Methods defined in a class go on the **prototype**, not on each instance — saving memory.

```js
class Animal {
  speak() { return 'sound'; }
}
class Dog extends Animal {}

const d = new Dog();

// Prototype chain:
// d --> Dog.prototype --> Animal.prototype --> Object.prototype --> null

console.log(d.__proto__ === Dog.prototype);                // true
console.log(Dog.prototype.__proto__ === Animal.prototype); // true

// speak lives on Animal.prototype, NOT on d
console.log(d.hasOwnProperty('speak')); // false
```

🎯 **Interview:** Why store methods on the class (prototype) instead of in the constructor?  
**Answer:** Defining methods inside the constructor with `this.method = function(){}` gives every instance its own copy — wasting memory. Class methods live on the shared prototype, so all instances share one copy.

---

## 12. Quick-Reference Cheat Sheet

| Concept | Syntax | Key Point |
|---|---|---|
| Instance method | `greet() {}` | Called on object: `obj.greet()` |
| Static method | `static add(a,b) {}` | Called on class: `Class.add()` |
| Static property | `static count = 0` | Shared across all instances |
| Private field | `#secret = 'x'` | `#` enforced by JS engine |
| Getter | `get name() {}` | Access like property: `obj.name` |
| Setter | `set name(v) {}` | Assign like property: `obj.name = v` |
| Inheritance | `class B extends A {}` | B gets all of A's methods |
| Super constructor | `super(args)` | Must be first line in child constructor |
| Super method | `super.method()` | Call parent's version of a method |
| Override | Redefine method in child | Child version runs on child instances |
| instanceof | `obj instanceof Class` | Checks class + parents |
| Abstract pattern | `if(new.target===Base) throw` | Prevent direct instantiation |
| Get class name | `obj.constructor.name` | Returns class name as string |

---

*JS Classes Interview Guide — Complete OOP Reference*
