# Class

## What is a class?

A class is a blueprint for creating objects with:
data (properties)
behavior (methods)
Example: BadRequestError is a class. When you do new BadRequestError("msg"), you create an object representing that error.
**Class** = `blueprint / design.`
**Object** = `us design ka real piece.`

Example:

- “Car” class = design
- “MyCar” object = actual car.
  
```ts
class Car {
  model = "Swift";
}
const c = new Car();
new Car() = ek naya object ban gaya.
```

## `constructor` kya hota hai?

- Constructor = object banate hi automatically call hota hai.

```ts
class User {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}
```

- `constructor(name)` ko data milta hai.
- `this.name` me store ho jata hai.
  
## this kya hota hai?

`this` = current object.

```ts
class User {
  name: string;
  constructor(name: string) { this.name = name; }

  sayHi() {
    return `Hi ${this.name}`;
  }
}
```

## Methods kaise banate?

**Method = class ke andar function.**

```ts
class Calc {
  add(a: number, b: number) {
    return a + b;
  }
}

```

## Public vs Private vs Protected (very important)

### public (default)

**Baher se access allowed.**

```ts
class A { public x = 1; }
new A().x // ✅


```

### private

**Sirf class ke andar.**

```ts
class A { private x = 1; }
new A().x // ❌

```

### protected

**Class + child class ke andar, baher se nahi.**

```ts
class A { protected x = 1; }
class B extends A { show(){ return this.x } } // ✅
new A().x // ❌

```

## Inheritance: `extends` kya hota hai?

**Inheritance: extends kya hota hai?**

```ts
class Animal {
  eat() { return "eating"; }
}

class Dog extends Animal {
  bark() { return "woof"; }
}

```

**Dog ko `eat()` bhi mil gaya + `bark()` bhi.**

## `super` kya hota hai?

**Jab child class ka constructor ho, to parent constructor ko call karne ke liye `super()`**

```ts
class Animal {
  name: string;
  constructor(name: string) { this.name = name; }
}

class Dog extends Animal {
  constructor(name: string) {
    super(name); // parent constructor
  }
}

```

Rule: `super()` pehle call hota hai, tabhi this use kar sakte ho.

## `abstract` class kya hoti hai?

- Abstract class = incomplete base class.
- Iska object nahi bana sakte:
  - ❌ new CustomError(...)

- Sirf extend karke use karte ho.

```ts
abstract class Shape {
  abstract area(): number; // must be implemented by child
}
```

**Child:**

```ts
class Circle extends Shape {
  area() { return 123; }
}

```

**Point:** abstract ensures child class required things implement kare.

## `abstract` property/method ka matlab

`abstract statusCode: number;`

**Matlab:`Child class ko ye property define karni hi padegi.`**

## `static` kya hota hai?

`static` = class ka property/method, object ka nahi.

```ts
class MathUtil {
  static pi = 3.14;
}
MathUtil.pi // ✅

```

**Object se:**

`new MathUtil().pi // ❌`

## `readonly` = immutability (can it change?)

- Property read ho sakti hai, but assign/update nahi ho sakta (after initialization).
- Accessible can be public or private — readonly is about “change”, not “access”.

```ts
class User {
  readonly id: string;
  constructor(id: string) {
    this.id = id;
  }
}

const u = new User("u1");
u.id        // ✅ read
// u.id = "u2" ❌ Error (readonly)

```
