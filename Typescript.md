# TypeScript Notes (Quickstart → Core Concepts)

> These notes are written for **TypeScript + Node.js** projects, but most concepts apply everywhere.

## Setup

Install TypeScript as a dev dependency:

```bash
npm install --save-dev typescript
```

Run the compiler (installed in `node_modules`) with:

```bash
npx tsc
```

### Create a `tsconfig.json`

In an empty project, `tsc` prints help by default. Create a config file:

```bash
npx tsc --init
```

## `tsconfig.json` basics

### Key fields

- **compilerOptions**: how TypeScript compiles your code (target, module, strictness, etc.)
- **include**: files/folders to include (commonly `["src"]`)
- **exclude**: files/folders to ignore (commonly `["node_modules", "dist"]`)
- **files**: explicit list of files to include (rare; usually use `include`)
- **extends**: inherit from another tsconfig (useful for monorepos)
- **references**: project references (monorepos / multi-package builds)

### Example: practical `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020"],
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "noImplicitAny": true,
    "baseUrl": ".",
    "paths": {
      "@app/*": ["src/app/*"]
    },
    "rootDir": "src",
    "outDir": "dist",
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

> [!NOTE]
> `moduleResolution` depends on your runtime/bundler. For Node.js (CommonJS) projects you may prefer `"module": "CommonJS"` and `"moduleResolution": "NodeNext"`.

---

## Your first TypeScript program

Create `hello.ts`:

```ts
function greet(name: string): string {
  return `Hello, ${name}!`;
}

const message: string = greet("World");
console.log(message);
```

Compile it:

```bash
npx tsc hello.ts
```

---

## TypeScript primitives

```ts
// Boolean
let isActive: boolean = true;
let hasPermission = false; // inferred as boolean

// Number
let decimal: number = 6;
let hex: number = 0xf00d;      // Hexadecimal
let binary: number = 0b1010;   // Binary
let octal: number = 0o744;     // Octal
let float: number = 3.14;

// String
let color: string = "blue";
let fullName: string = "John Doe";

// BigInt (ES2020+)
const bigNumber: bigint = 9007199254740991n;
const hugeNumber = BigInt(9007199254740991);

// Symbol
const uniqueKey: symbol = Symbol("description");
const obj = {
  [uniqueKey]: "This is a unique property"
};
console.log(obj[uniqueKey]);
```

---

## Type annotations vs type inference

TypeScript offers two ways to work with types:

1. **Explicit typing**: you declare the type yourself  
2. **Type inference**: TypeScript infers the type from the assigned value

### When to use each

Use **explicit types** for:
- Function parameters and return types
- Object literals (especially public-facing shapes)
- When the initial value might not be the final type

Use **type inference** for:
- Simple variables declared with immediate assignment
- Cases where the type is obvious

### Explicit type annotations

```ts
// String
let greeting: string = "Hello, TypeScript!";

// Number
let userCount: number = 42;

// Boolean
let isLoading: boolean = true;

// Array of numbers
let scores: number[] = [100, 95, 98];
```

> [!NOTE]
> Best practice: use explicit types for **function parameters and return types** to make code more maintainable and self-documenting.

```ts
function greet(name: string): string {
  return `Hello, ${name}!`;
}

greet("Alice"); // OK
// greet(42);   // ❌ Error: Argument of type 'number' is not assignable to parameter of type 'string'
```

### Type inference

```ts
let username = "alice"; // inferred as string
let score = 100;        // inferred as number
let flags = [true, false, true]; // inferred as boolean[]

function add(a: number, b: number) {
  return a + b; // inferred return type: number
}
```

> [!NOTE]
> Inference works best when variables are initialized at declaration.  
> Uninitialized variables become `any` unless `noImplicitAny` is enabled.

### Type safety in action

```ts
let username2: string = "alice";
// username2 = 42; // ❌ Error: Type 'number' is not assignable to type 'string'

let score2 = 100; // inferred as number
// score2 = "high"; // ❌ Error: Type 'string' is not assignable to type 'number'
```

---

## When TypeScript can't infer types (and falls back to `any`)

```ts
// 1) JSON.parse returns 'any' because the structure isn't known at compile time
const data = JSON.parse('{ "name": "Alice", "age": 30 }');

// 2) Variable declared without initialization
let something; // type: any (unless noImplicitAny is enabled)
something = "hello";
something = 42; // no error if it's 'any'
```

### Avoid `any` when possible

Instead consider:
- Add type annotations
- Create interfaces/types for complex objects
- Use type guards for runtime checks
- Enable `noImplicitAny` in your tsconfig

---

## Special types: `any`, `unknown`, `never`

### `any`

- Opts out of type checking for a value.
- Useful during migration from JS → TS, but should be used sparingly.

> [!WARNING]
> `any` disables TypeScript’s protection—avoid it unless you truly must.

### `unknown`

`unknown` is the type-safe counterpart of `any`.

Key rules:
- You **must** narrow or assert the type before using it.
- You can’t access properties/call it without narrowing.

```ts
function handleInput(input: unknown) {
  if (typeof input === "string") {
    console.log(input.toUpperCase());
  }
}
```

### `never`

Represents values that never occur.

Common uses:
- Functions that never return (throw / infinite loop)
- Exhaustiveness checking in unions

```ts
function throwError(message: string): never {
  throw new Error(message);
}
```

---

## Arrays

```ts
const names: string[] = [];
names.push("Dylan");
// names.push(3); // ❌ Error
```

### Readonly arrays

```ts
const nums: readonly number[] = [1, 2, 3];
// nums.push(4); // ❌ Property 'push' does not exist on type 'readonly number[]'
```

---

## Tuples

A **tuple** is a typed array with a fixed length and known types per index.

```ts
let ourTuple: [number, boolean, string];
ourTuple = [5, false, "Coding God was here"];
// ourTuple = [false, 5, "x"]; // ❌ order matters
```

### Destructuring tuples

```ts
const graph: [number, number] = [55.2, 41.3];
const [x, y] = graph;
```

---

## Object types

```ts
const car: { type: string; model: string; year: number } = {
  type: "Toyota",
  model: "Corolla",
  year: 2009
};
```

### Optional properties

```ts
const car2: { type: string; mileage?: number } = {
  type: "Toyota"
};

car2.mileage = 2000;
```

---

## Enums (numeric example)

```ts
enum StatusCodes {
  NotFound = 404,
  Success = 200,
  Accepted = 202,
  BadRequest = 400
}

console.log(StatusCodes.NotFound); // 404
console.log(StatusCodes.Success);  // 200
```

> [!NOTE]
> Many projects prefer string unions (`"success" | "error"`) over enums for simpler JS output.

---

## Type aliases and interfaces

### Type aliases

**1) Alias for primitives**

```ts
type UserId = string;
type Age = number;

const id: UserId = "u-101";
const age: Age = 25;
```

**2) Alias for object types**

```ts
type User = {
  id: string;
  name: string;
  isActive: boolean;
};

const u1: User = { id: "1", name: "Suraj", isActive: true };
```

**Intersection (`&`) = combine types**

```ts
type Animal = { name: string };
type Bear = Animal & { honey: boolean };

const bear: Bear = { name: "Winnie", honey: true };
```

**Union (`|`) = either/or**

```ts
type Status = "success" | "error";
let response: Status = "success";
// response = "pending"; // ❌ not allowed
```

### Interfaces

Interfaces define the **shape** of an object. Common in real projects (API types, React props, service contracts).

**1) Basic interface**

```ts
interface IUser {
  id: string;
  name: string;
  isActive: boolean;
}

const userA: IUser = { id: "1", name: "Suraj", isActive: true };
```

**2) Optional properties**

```ts
interface IUser2 {
  id: string;
  name: string;
  phone?: string;
}

const userB: IUser2 = { id: "2", name: "Amit" };
```

**3) Readonly properties**

```ts
interface IUser3 {
  readonly id: string;
  name: string;
}

const userC: IUser3 = { id: "1", name: "Suraj" };
// userC.id = "2"; // ❌ not allowed
```

**4) Function types in interfaces**

```ts
interface Logger {
  log(message: string): void;
}

const consoleLogger: Logger = {
  log(msg) {
    console.log(msg);
  }
};
```

**Extending interfaces (`extends`)**

```ts
interface Animal2 {
  name: string;
}

interface Bear2 extends Animal2 {
  honey: boolean;
}

const b: Bear2 = { name: "Winnie", honey: true };
```

---

## Union types (more examples)

```ts
function printStatusCode(code: string | number) {
  console.log(`My status code is ${code}.`);
}

printStatusCode(404);
printStatusCode("404");
```

---

## Functions

### Return type

```ts
function getTime(): number {
  return new Date().getTime();
}
```

### `void` return type

```ts
function printHello(): void {
  console.log("Hello!");
}
```

### Optional parameters

```ts
function add2(a: number, b: number, c?: number) {
  return a + b + (c ?? 0);
}
```

### Default parameters

```ts
function pow(value: number, exponent: number = 10) {
  return value ** exponent;
}
```

---

## Casting (type assertions)

### `as` casting

```ts
let x: unknown = "hello";
console.log((x as string).length);
```

### `<>` casting

```ts
let y: unknown = "hello";
console.log((<string>y).length);
```

> [!NOTE]
> In TSX/React files, prefer `as` because `<>` conflicts with JSX syntax.

---

## Classes

```ts
class Person {
  name: string = "";
}

const person = new Person();
person.name = "Jane";
```

### Visibility modifiers

- `public` (default): accessible everywhere
- `private`: accessible only inside the class
- `protected`: accessible in the class and subclasses

```ts
class Person2 {
  private name: string;

  public constructor(name: string) {
    this.name = name;
  }

  public getName(): string {
    return this.name;
  }
}

const p2 = new Person2("Jane");
console.log(p2.getName());
// console.log(p2.name); // ❌ private
```

### Parameter properties

```ts
class Person3 {
  public constructor(private name: string) {}

  public getName(): string {
    return this.name;
  }
}
```

---

## Generics (basic)

Generics let you write reusable code with “type variables”.

### Generic functions

```ts
function createPair<S, T>(v1: S, v2: T): [S, T] {
  return [v1, v2];
}

console.log(createPair<string, number>("hello", 42));
```

### Generic classes

```ts
class NamedValue<T> {
  private _value: T | undefined;

  constructor(private name: string) {}

  public setValue(value: T) {
    this._value = value;
  }

  public getValue(): T | undefined {
    return this._value;
  }

  public toString(): string {
    return `${this.name}: ${this._value}`;
  }
}

const value = new NamedValue<number>("myNumber");
value.setValue(10);
console.log(value.toString());
```

### Generic type aliases

```ts
type Wrapped<T> = { value: T };

const wrappedValue: Wrapped<number> = { value: 10 };
```

---

## Utility types

### `Partial<T>`

Makes all properties optional.

```ts
interface Point {
  x: number;
  y: number;
}

let pointPart: Partial<Point> = {};
pointPart.x = 10;
```

### `Required<T>`

Makes all properties required.

```ts
interface Car3 {
  make: string;
  model: string;
  mileage?: number;
}

let myCar: Required<Car3> = {
  make: "Ford",
  model: "Focus",
  mileage: 12000
};
```

### `Record<K, V>`

```ts
const nameAgeMap: Record<string, number> = {
  Alice: 21,
  Bob: 25
};
```

### `Omit<T, K>`

```ts
interface Person4 {
  name: string;
  age: number;
  location?: string;
}

const bob: Omit<Person4, "age" | "location"> = {
  name: "Bob"
};
```

### `Pick<T, K>`

```ts
const bob2: Pick<Person4, "name"> = {
  name: "Bob"
};
```

### `Exclude<T, U>`

```ts
type Primitive = string | number | boolean;
const val: Exclude<Primitive, string> = true; // string excluded
```

### `ReturnType<T>`

```ts
type PointGenerator = () => { x: number; y: number };
const p: ReturnType<PointGenerator> = { x: 10, y: 20 };
```

### `Parameters<T>`

```ts
type PointPrinter = (p: { x: number; y: number }) => void;
const arg: Parameters<PointPrinter>[0] = { x: 10, y: 20 };
```

### `Readonly<T>`

```ts
interface Person5 {
  name: string;
  age: number;
}

const personR: Readonly<Person5> = { name: "Dylan", age: 35 };
// personR.name = "Israel"; // ❌ Cannot assign to 'name'
```

---

## Node.js + TypeScript project skeleton

### 1) Initialize

```bash
mkdir my-ts-node-app
cd my-ts-node-app
npm init -y
npm install --save-dev typescript @types/node
npx tsc --init
```

### 2) Create folders

Keep source in `src/` and output in `dist/`.

```bash
mkdir src
mkdir dist
# later add files like: src/server.ts, src/middleware/auth.ts
```

### 3) Useful `package.json` scripts

```json
{
  "scripts": {
    "build": "tsc -p tsconfig.json",
    "start": "node dist/server.js",
    "dev": "tsc -w"
  }
}
```

---

### Next topics to add (optional)

- Narrowing & type guards (`in`, `typeof`, `instanceof`, custom predicates)
- Discriminated unions
- `as const`, literal inference, template literal types
- Zod / runtime validation patterns for API inputs
- TS with Express / Fastify / NestJS
