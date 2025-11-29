## TypeScript
```bash
npm install typescript --save-dev
```
- The compiler is installed in the node_modules directory and can be run with: npx tsc.
```bash
npx tsc
```
- **Configuring the compiler**
By default the TypeScript compiler will print a help message when run in an empty project.

- The compiler can be configured using a `tsconfig.json` file.

You can have TypeScript create `tsconfig.json` with the recommended settings with:
```bash
npx tsc --init
```

**Key Concepts & Explanations**
- compilerOptions: Controls how TypeScript compiles your code (e.g., target, module, strictness).
- include: Files or folders to include in the compilation.
- exclude: Files or folders to exclude.
- files: Explicit list of files to include (rarely used with include).
- extends: Inherit options from another config file.
- references: Enable project references for monorepos or multi-package setups.
**`Advanced tsconfig.json`**\
```bash
{
  "compilerOptions": {
    "target": "es2020",
    "module": "esnext",
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@app/*": ["src/app/*"]
    },
    "outDir": "dist",
    "esModuleInterop": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```
### Your First TypeScript Program

**1.Create a new file named `hello.ts` with the following content:**
```ts
function greet(name: string): string {
  return `Hello, ${name}!`;
}

const message: string = greet("World");
console.log(message);
```
```bash
npx tsc hello.ts
```
#### TypeScript Primitives:
```ts
let isActive: boolean = true;
let hasPermission = false;
//Number
let decimal: number = 6;
let hex: number = 0xf00d;       //Hexadecimal
let binary: number = 0b1010;     // Binary
let octal: number = 0o744;      // Octal
let float: number = 3.14; 
//String
let color: string = "blue";
let fullName: string = 'John Doe';
//BigInt (ES2020+)
const bigNumber: bigint = 9007199254740991n;
const hugeNumber = BigInt(9007199254740991);
//Symbol 
const uniqueKey: symbol = Symbol('description');
const obj = {
  [uniqueKey]: 'This is a unique property'
};
console.log(obj[uniqueKey]); 
```
#### Type Annotations and Inference.
TypeScript offers two ways to work with types:

1.Explicit Typing: You explicitly declare the type of a variable
2.Type Inference: TypeScript automatically determines the type based on the assigned value
###### When to Use Each Approach
Use explicit types for:
- Function parameters and return types
Object literals
- When the initial value might not be the final type
Use type inference for:
- Simple variable declarations with immediate assignment
- When the type is obvious from the context
**Explicit Type Annotations**
```ts
// String
greeting: string = "Hello, TypeScript!";

// Number
userCount: number = 42;

// Boolean
isLoading: boolean = true;

// Array of numbers
scores: number[] = [100, 95, 98];
```
>[!note]Best Practice: Use explicit types for function parameters and return types to make your code more maintainable and self-documenting.

```ts
//Function with Explicit Types
function greet(name: string): string {
return `Hello, ${name}!`;
}

// TypeScript will ensure you pass the correct argument type
greet("Alice"); // OK
greet(42);     // Error: Argument of type '42' is not assignable to parameter of type 'string'
```
##### Type Inference
```ts
// TypeScript infers 'string'
let username = "alice";

// TypeScript infers 'number'
let score = 100;

// TypeScript infers 'boolean[]'
let flags = [true, false, true];

// TypeScript infers return type as 'number'
function add(a: number, b: number) {
return a + b;
}
```
>[!note]Note: Type inference works best when variables are initialized at declaration.Uninitialized variables have type 'any' by default unless you enable strictNullChecks in your tsconfig.json.

##### Type Safety in Action.
```ts
//Explicit Type Mismatch
let username: string = "alice";
username = 42; // Error: Type 'number' is not assignable to type 'string'
//Implicit Type Mismatch
let score = 100;  // TypeScript infers 'number'
score = "high";  // Error: Type 'string' is not assignable to type 'number'
```
#### **When TypeScript Can't Infer Types**
- While TypeScript's type inference is powerful, there are cases where it can't determine the correct type.

- In these situations, TypeScript falls back to the any type, which disables type checking.
>[!note]Note: any and other Special Types are covered in more detail in the next chapter.
```ts
/ 1. JSON.parse returns 'any' because the structure isn't known at compile time
const data = JSON.parse('{ "name": "Alice", "age": 30 }');

// 2. Variables declared without initialization
let something;  // Type is 'any'
something = 'hello';
something = 42;  // No error
```
**Avoid `any` When Possible**
Using any disables TypeScript's type checking.

Instead, consider these alternatives:

- Use type annotations
- Create interfaces for complex objects
- Use type guards for runtime type checking
- Enable noImplicitAny in your tsconfig.json

##### TypeScript Special Types
These types are used in various scenarios to handle cases where the type might not be known in advance or when you need to work with JavaScript primitives in a type-safe way.
>[!Note]These special types are part of TypeScript's type system and help make your code more type-safe and self-documenting.

###### **Type: any**
- The any type is the most flexible type in TypeScript.
- It essentially tells the compiler to skip type checking for a particular variable.
- While this can be useful in certain situations, it should be used sparingly as it bypasses TypeScript's type safety features.
>[!any]When to use `any`:
- When migrating JavaScript code to TypeScript.
- When working with dynamic content where the type is unknown.
- When you need to opt out of type checking for a specific case.
>[!warning]Remember, it should be avoided at "any" cost...

###### **Type: unknown**
The `unknown` type is a type-safe counterpart of `any`.
It's the type-safe way to say "this could be anything, so you must perform some type of checking before you use it".
>[!note]Key differences between unknown and any:
>unknown must be type-checked before use.
>You can't access properties on an unknown type without type assertion.
>You can't call or construct values of type unknown.

**When to use unknown:**
- When working with data from external sources (APIs, user input, etc.)
- When you want to ensure type safety while still allowing flexibility
- When migrating from JavaScript to TypeScript in a type-safe way.
  
**Type: never**
- The never type represents the type of values that never occur.
- It's used to indicate that something never happens or should never happen.
**Common use cases for never:**
- Functions that never return (always throw an error or enter an infinite loop).
- Type guards that never pass type checking.
- Exhaustiveness checking in discriminated unions.
######  Examples of never in action:
**1. Function that never returns**
```js
function throwError(message: string): never {
  throw new Error(message);
}
```
#### TypeScript Arrays
```ts
const names: string[] = [];
names.push("Dylan");
// names.push(3); // Error: Argument of type 'number' is not assignable to parameter of type 'string'.
```
**Readonly**
- The `readonly` keyword can prevent arrays from being changed.

#### TypeScript Tuples
###### Typed Arrays
- A `tuple` is a typed array with a pre-defined length and types for each index.Tuples are great because they allow each element in the array to be a known type of value.
```ts
// define our tuple
let ourTuple: [number, boolean, string];
// initialize correctly
ourTuple = [5, false, 'Coding God was here'];
```
>[!note]Even though we have a boolean, string, and number the order matters in our tuple and will throw an error.
###### Destructuring Tuples
```ts
const graph: [number, number] = [55.2, 41.3];
const [x, y] = graph;
```
#### TypeScript Object Types 
```ts
const car: { type: string, model: string, year: number } = {
  type: "Toyota",
  model: "Corolla",
  year: 2009
};
```
###### Optional Properties
```ts
const car: { type: string, mileage?: number } = { // no error
  type: "Toyota"
};
car.mileage = 2000;
```
##### Numeric Enums - Fully Initialized
- You can assign unique number values for each enum value.
- Then the values will not be incremented automatically:
```ts
enum StatusCodes {
  NotFound = 404,
  Success = 200,
  Accepted = 202,
  BadRequest = 400
}
// logs 404
console.log(StatusCodes.NotFound);
// logs 200
console.log(StatusCodes.Success);
```
#### **TypeScript Type Aliases and Interfaces**
- Type Alias in TypeScript means: you give a type a new name, so you can reuse it and keep code clean.
**1) Alias for primitive types**
```js
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
**1) Intersection type (&) = “combine / merge types”**
```ts
type Animal = { name: string };
type Bear = Animal & { honey: boolean };
const bear: Bear = { name: "Winnie", honey: true };
```
**Meaning:**
- Bear must have everything from Animal AND everything from { honey: boolean }
so Bear becomes: { name: string; honey: boolean }

**2) Union type (|) = “either this OR that”**
```ts
type Status = "success" | "error";
let response: Status = "success";
```
**Meaning:**
- response can be only "success" or "error"
- anything else is 
#### Interfaces
Interfaces in TypeScript are a way to **define the “shape” (structure) of an object.** They are used a lot in real projects (React props, API responses, service contracts).
- 1) Basic interface (object shape)
```ts
  interface User {
  id: string;
  name: string;
  isActive: boolean;
}
const u1: User = { id: "1", name: "Suraj", isActive: true };
```
- 2) Optional properties (?)
```ts
interface User {
  id: string;
  name: string;
  phone?: string; // optional
}
const u2: User = { id: "2", name: "Amit" }; // ok

```
-3) Readonly properties:
```ts
interface User {
  readonly id: string;
  name: string;
}
const u: User = { id: "1", name: "Suraj" };
// u.id = "2" ❌ not allowed

```
**4) Function types in interface**
```ts
interface Logger {
  log(message: string): void;
}

const consoleLogger: Logger = {
  log(msg) {
    console.log(msg);
  },
};
```
**Extending interface (extends)**
```ts
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}

const b: Bear = { name: "Winnie", honey: true };
```
```ts
interface Rectangle {
  height: number,
  width: number
}
interface ColoredRectangle extends Rectangle {
  color: string
}
const coloredRectangle: ColoredRectangle = {
  height: 20,
  width: 10,
  color: "red"
};
```
##### TypeScript Union Types
**Union types** are used when a value can be more than a single type.
Such as when a property would be `string` or `number`.

**Union | (OR)**
- Using the | we are saying our parameter is a `string` or `number`:
```ts
function printStatusCode(code: string | number) {
  console.log(`My status code is ${code}.`)
}
printStatusCode(404);
printStatusCode('404');
```
##### TypeScript Functions
**Return Type**
```ts
// the `: number` here specifies that this function returns a number
function getTime(): number {
  return new Date().getTime();
}
```
**Void Return Type**
- The type `void` can be used to indicate a function doesn't return any value.
```ts
function printHello(): void {
  console.log('Hello!');
}
```
**Parameters**
```ts
function multiply(a: number, b: number) {
  return a * b;
}
```
**Optional Parameters**
- By default TypeScript will assume all parameters are required, but they can be explicitly marked as optional.
```ts
// the `?` operator here marks parameter `c` as optional
function add(a: number, b: number, c?: number) {
  return a + b + (c || 0);
}
```
**Default Parameters**
- For parameters with default values, the default value goes after the type annotation:
```ts
function pow(value: number, exponent: number = 10) {
  return value ** exponent;
}
```
##### TypeScript Casting
**Casting with `as`**
A straightforward way to cast a variable is using the as keyword, which will directly change the type of the given variable.
```ts
let x: unknown = 'hello';
console.log((x as string).length);
```
**Casting with `<>`**
Using `<>` works the same as casting with as.
```ts
let x: unknown = 'hello';
console.log((<string>x).length);
```
#### TypeScript Classes
- The members of a class (properties & methods) are typed using type annotations, similar to variables.
```ts
class Person {
  name: string;
}
const person = new Person();
person.name = "Jane";
```
##### Members: Visibility
>[!note]There are three main visibility modifiers in TypeScript.
 - public - (default) allows access to the class member from anywhere
 - private - only allows access to the class member from within the class
 - protected - allows access to the class member from itself and any classes that inherit it, which is covered in the inheritance section below.
```ts
 class Person {
  private name: string;

  public constructor(name: string) {
    this.name = name;
  }

  public getName(): string {
    return this.name;
  }
}

const person = new Person("Jane");
console.log(person.getName()); // person.name isn't accessible from outside the class since it's private
```
#### Parameter Properties
TypeScript provides a convenient way to define class members in the constructor, by adding a visibility modifier to the parameter.
```ts
class Person {
  // name is a private member variable
  public constructor(private name: string) {}

  public getName(): string {
    return this.name;
  }
}

const person = new Person("Jane");
console.log(person.getName());
```

#### TypeScript Basic Generics
Generics allow creating 'type variables' which can be used to create classes, functions & type aliases that don't need to explicitly define the types that they use.
Generics make it easier to write reusable code.
**Functions**
- Generics with functions help create more general functions that accurately represent the input and return types.
```ts
function createPair<S, T>(v1: S, v2: T): [S, T] {
  return [v1, v2];
}
console.log(createPair<string, number>('hello', 42)); // ['hello', 42]
```
**Classes**
Generics can be used to create generalized classes, like Map.
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

let value = new NamedValue<number>('myNumber');
value.setValue(10);
console.log(value.toString()); // myNumber: 10
```
**Type Aliases**
```ts
type Wrapped<T> = { value: T };

const wrappedValue: Wrapped<number> = { value: 10 };
```
#### TypeScript Utility Types
- TypeScript comes with a large number of types that can help with some common type manipulation, usually referred to as utility types.
  **Partial**
  `Partial` changes all the properties in an object to be optional.
```ts
interface Point {
  x: number;
  y: number;
}

let pointPart: Partial<Point> = {}; // `Partial` allows x and y to be optional
pointPart.x = 10;
```
**Required**
`Required` changes all the properties in an object to be required.
```ts
interface Car {
  make: string;
  model: string;
  mileage?: number;
}

let myCar: Required<Car> = {
  make: 'Ford',
  model: 'Focus',
  mileage: 12000 // `Required` forces mileage to be defined
}
```
**Record**
`Record` is a shortcut to defining an object type with a specific key type and value type.
```ts
const nameAgeMap: Record<string, number> = {
  'Alice': 21,
  'Bob': 25
};
```
>[!warning]Record<string, number> is equivalent to { [key: string]: number }

**Omit**
`Omit` removes keys from an object type.
```ts
interface Person {
  name: string;
  age: number;
  location?: string;
}

const bob: Omit<Person, 'age' | 'location'> = {
  name: 'Bob'
  // `Omit` has removed age and location from the type and they can't be defined here
};
```
**Pick**
`Pick` removes all but the specified keys from an object type.
```ts
interface Person {
  name: string;
  age: number;
  location?: string;
}

const bob: Pick<Person, 'name'> = {
  name: 'Bob'
  // `Pick` has only kept name, so age and location were removed from the type and they can't be defined here
};
```
**Exclude**
`Exclude` removes types from a union.
```ts
type Primitive = string | number | boolean
const value: Exclude<Primitive, string> = true; // a string cannot be used here since Exclude removed it from the type.
```
**ReturnType**
`ReturnType` extracts the return type of a function type.
```ts
type PointGenerator = () => { x: number; y: number; };
const point: ReturnType<PointGenerator> = {
  x: 10,
  y: 20
};
```
**Parameters**
`Parameters` extracts the parameter types of a function type as an array.
```ts
type PointPrinter = (p: { x: number; y: number; }) => void;
const point: Parameters<PointPrinter>[0] = {
  x: 10,
  y: 20
};
```
**Readonly**
`Readonly` is used to create a new type where all properties are readonly, meaning they cannot be modified once assigned a value.
```ts
interface Person {
  name: string;
  age: number;
}
const person: Readonly<Person> = {
  name: "Dylan",
  age: 35,
};
person.name = 'Israel'; // prog.ts(11,8): error TS2540: Cannot assign to 'name' because it is a read-only property.
```

### **TypeScript Configuration**
- The `tsconfig.json` file is the heart of every TypeScript project.
- It tells the TypeScript compiler how to process your code, which files to include, and which features to enable or disable.
- A well-configured `tsconfig.json` ensures a smooth developer experience and reliable builds.

**Key Concepts & Explanations**
- **compilerOptions**: Controls how TypeScript compiles your code (e.g., target, module, strictness).
- **include**: Files or folders to include in the compilation.
- **exclude**: Files or folders to exclude.
- **files**: Explicit list of files to include (rarely used with include).
- **extends**: Inherit options from another config file.
- **references**: Enable project references for monorepos or multi-package setups.

**To generate a `tsconfig.json` file, run:**
`tsc --init`
###### 1.Initialize a New Project
```bash
mkdir my-ts-node-app
cd my-ts-node-app
npm init -y
npm install typescript @types/node --save-dev
npx tsc --init
```
###### 2. Create a Source Folder
Keep source code in `src/ `and compiled output in `dist/.`
```ts
mkdir src
# later add files like: src/server.ts, src/middleware/auth.ts
```