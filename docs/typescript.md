# TypeScript Cheatsheet

A comprehensive guide to TypeScript - JavaScript with static typing for better development experience and code quality.

## Table of Contents

- [Installation & Setup](#installation--setup)
- [Basic Types](#basic-types)
- [Interfaces](#interfaces)
- [Type Aliases](#type-aliases)
- [Classes](#classes)
- [Generics](#generics)
- [Utility Types](#utility-types)
- [Type Guards](#type-guards)
- [Advanced Types](#advanced-types)
- [Decorators](#decorators)
- [Configuration](#configuration)
- [Best Practices](#best-practices)

## Installation & Setup

### Install TypeScript

```bash
# Global installation
npm install -g typescript

# Project installation
npm install --save-dev typescript

# Check version
tsc --version
```

### Initialize TypeScript Project

```bash
# Create tsconfig.json
tsc --init

# Compile TypeScript file
tsc file.ts

# Watch mode
tsc --watch

# Compile all files in project
tsc
```

### Quick Setup

```bash
# Create new project
mkdir my-project && cd my-project
npm init -y
npm install --save-dev typescript @types/node

# Create tsconfig.json
npx tsc --init

# Create src directory
mkdir src
echo 'console.log("Hello TypeScript");' > src/index.ts

# Compile and run
npx tsc
node dist/index.js

# Using ts-node for development
npm install --save-dev ts-node
npx ts-node src/index.ts
```

## Basic Types

### Primitive Types

```typescript
// Boolean
let isDone: boolean = false;

// Number
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;

// String
let color: string = "blue";
let fullName: string = `Bob Smith`;
let sentence: string = `Hello, my name is ${fullName}`;

// Null and Undefined
let u: undefined = undefined;
let n: null = null;

// Void (for functions that don't return)
function warn(): void {
  console.log("Warning!");
}

// Any (avoid when possible)
let notSure: any = 4;
notSure = "maybe a string";
notSure = false;

// Unknown (safer than any)
let value: unknown = 4;
value = "hello";
// value.toUpperCase(); // Error
if (typeof value === "string") {
  value.toUpperCase(); // OK
}

// Never (for functions that never return)
function error(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}
```

### Arrays

```typescript
// Array type
let list: number[] = [1, 2, 3];
let list2: Array<number> = [1, 2, 3];

// String array
let names: string[] = ["Alice", "Bob", "Charlie"];

// Mixed types (tuple-like but not strict)
let mixed: (string | number)[] = [1, "two", 3];

// Readonly array
let ro: ReadonlyArray<number> = [1, 2, 3];
// ro.push(4); // Error
```

### Tuples

```typescript
// Tuple: fixed-length array with known types
let x: [string, number];
x = ["hello", 10]; // OK
// x = [10, "hello"]; // Error

// Accessing elements
console.log(x[0]); // "hello"
console.log(x[1]); // 10

// Tuple with optional elements
let optional: [string, number?];
optional = ["hello"];
optional = ["hello", 42];

// Rest elements in tuples
let rest: [string, ...number[]];
rest = ["hello"];
rest = ["hello", 1, 2, 3];

// Named tuples
type Point = [x: number, y: number];
type Range = [start: number, end: number];
```

### Enums

```typescript
// Numeric enum
enum Direction {
  Up = 1,
  Down,
  Left,
  Right,
}
let dir: Direction = Direction.Up;

// String enum
enum Status {
  Active = "ACTIVE",
  Inactive = "INACTIVE",
  Pending = "PENDING",
}
let status: Status = Status.Active;

// Const enum (more efficient)
const enum HttpStatus {
  OK = 200,
  NotFound = 404,
  InternalServerError = 500,
}
let code: HttpStatus = HttpStatus.OK;

// Computed and constant members
enum FileAccess {
  None,
  Read = 1 << 1,
  Write = 1 << 2,
  ReadWrite = Read | Write,
}
```

### Object Types

```typescript
// Object type
let obj: { name: string; age: number } = {
  name: "John",
  age: 30,
};

// Optional properties
let person: { name: string; age?: number } = {
  name: "Alice",
};

// Readonly properties
let point: { readonly x: number; readonly y: number } = {
  x: 10,
  y: 20,
};
// point.x = 5; // Error

// Index signatures
let dict: { [key: string]: number } = {
  age: 30,
  height: 180,
};
```

## Interfaces

### Basic Interface

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // Optional
  readonly createdAt: Date; // Readonly
}

const user: User = {
  id: 1,
  name: "John Doe",
  email: "john@example.com",
  createdAt: new Date(),
};
```

### Function Interfaces

```typescript
// Function type
interface SearchFunc {
  (source: string, subString: string): boolean;
}

const mySearch: SearchFunc = (src, sub) => {
  return src.includes(sub);
};

// Method signature
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

const calc: Calculator = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
};
```

### Extending Interfaces

```typescript
interface Animal {
  name: string;
  age: number;
}

interface Dog extends Animal {
  breed: string;
  bark(): void;
}

const myDog: Dog = {
  name: "Buddy",
  age: 3,
  breed: "Golden Retriever",
  bark() {
    console.log("Woof!");
  },
};

// Multiple inheritance
interface Flyable {
  fly(): void;
}

interface Swimmable {
  swim(): void;
}

interface Duck extends Animal, Flyable, Swimmable {}
```

### Interface Merging (Declaration Merging)

```typescript
interface Box {
  height: number;
  width: number;
}

interface Box {
  depth: number;
}

// Merged into one interface
const box: Box = {
  height: 10,
  width: 20,
  depth: 5,
};
```

### Index Signatures

```typescript
interface StringMap {
  [key: string]: string;
}

const colors: StringMap = {
  primary: "blue",
  secondary: "green",
};

// With additional properties
interface Config {
  name: string;
  [key: string]: any;
}

const config: Config = {
  name: "app",
  debug: true,
  timeout: 5000,
};
```

## Type Aliases

### Basic Type Alias

```typescript
// Primitive type alias
type ID = string | number;

// Object type
type Point = {
  x: number;
  y: number;
};

// Function type
type Callback = (data: string) => void;

// Union type
type Result = "success" | "error" | "pending";

// Using type aliases
let userId: ID = "abc123";
let point: Point = { x: 10, y: 20 };
let status: Result = "success";
```

### Intersection Types

```typescript
type Name = {
  firstName: string;
  lastName: string;
};

type Contact = {
  email: string;
  phone: string;
};

// Intersection (AND)
type Person = Name & Contact;

const person: Person = {
  firstName: "John",
  lastName: "Doe",
  email: "john@example.com",
  phone: "123-456-7890",
};
```

### Union Types

```typescript
// Union (OR)
type StringOrNumber = string | number;

function print(value: StringOrNumber): void {
  console.log(value);
}

print("hello");
print(42);

// Discriminated unions
type Success = {
  type: "success";
  data: string;
};

type Error = {
  type: "error";
  message: string;
};

type Response = Success | Error;

function handleResponse(response: Response): void {
  if (response.type === "success") {
    console.log(response.data);
  } else {
    console.error(response.message);
  }
}
```

### Literal Types

```typescript
// String literals
type Direction = "north" | "south" | "east" | "west";
let dir: Direction = "north";

// Numeric literals
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;
let roll: DiceRoll = 4;

// Boolean literals
type SuccessResponse = { success: true; data: string };
type ErrorResponse = { success: false; error: string };
type ApiResponse = SuccessResponse | ErrorResponse;
```

## Classes

### Basic Class

```typescript
class Person {
  // Properties
  name: string;
  age: number;

  // Constructor
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  // Method
  greet(): void {
    console.log(`Hello, I'm ${this.name}`);
  }
}

const person = new Person("Alice", 30);
person.greet();
```

### Access Modifiers

```typescript
class BankAccount {
  public accountNumber: string; // Accessible anywhere (default)
  private balance: number; // Only accessible within class
  protected owner: string; // Accessible within class and subclasses

  constructor(accountNumber: string, initialBalance: number, owner: string) {
    this.accountNumber = accountNumber;
    this.balance = initialBalance;
    this.owner = owner;
  }

  public deposit(amount: number): void {
    this.balance += amount;
  }

  public getBalance(): number {
    return this.balance;
  }
}

// Parameter properties (shorthand)
class User {
  constructor(
    public id: number,
    public name: string,
    private password: string
  ) {}
}
```

### Readonly Properties

```typescript
class Point {
  readonly x: number;
  readonly y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

const p = new Point(10, 20);
// p.x = 5; // Error
```

### Getters and Setters

```typescript
class Employee {
  private _fullName: string = "";

  get fullName(): string {
    return this._fullName;
  }

  set fullName(newName: string) {
    if (newName && newName.length > 0) {
      this._fullName = newName;
    }
  }
}

const emp = new Employee();
emp.fullName = "John Doe";
console.log(emp.fullName);
```

### Static Members

```typescript
class MathUtils {
  static PI: number = 3.14159;

  static calculateCircumference(diameter: number): number {
    return diameter * MathUtils.PI;
  }
}

console.log(MathUtils.PI);
console.log(MathUtils.calculateCircumference(10));
```

### Abstract Classes

```typescript
abstract class Animal {
  abstract makeSound(): void;

  move(): void {
    console.log("Moving...");
  }
}

class Dog extends Animal {
  makeSound(): void {
    console.log("Woof!");
  }
}

// const animal = new Animal(); // Error: cannot instantiate abstract class
const dog = new Dog();
dog.makeSound();
dog.move();
```

### Inheritance

```typescript
class Vehicle {
  constructor(public brand: string) {}

  drive(): void {
    console.log(`Driving a ${this.brand}`);
  }
}

class Car extends Vehicle {
  constructor(brand: string, public model: string) {
    super(brand);
  }

  drive(): void {
    super.drive();
    console.log(`Model: ${this.model}`);
  }
}

const car = new Car("Toyota", "Camry");
car.drive();
```

### Implementing Interfaces

```typescript
interface Drawable {
  draw(): void;
}

interface Resizable {
  resize(width: number, height: number): void;
}

class Rectangle implements Drawable, Resizable {
  constructor(private width: number, private height: number) {}

  draw(): void {
    console.log(`Drawing rectangle ${this.width}x${this.height}`);
  }

  resize(width: number, height: number): void {
    this.width = width;
    this.height = height;
  }
}
```

## Generics

### Generic Functions

```typescript
// Basic generic function
function identity<T>(arg: T): T {
  return arg;
}

let output1 = identity<string>("hello");
let output2 = identity<number>(42);
// Type inference
let output3 = identity("world"); // TypeScript infers T as string

// Generic with array
function getFirstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

const first = getFirstElement([1, 2, 3]); // number
const firstStr = getFirstElement(["a", "b"]); // string

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const result = pair<string, number>("age", 30);
```

### Generic Interfaces

```typescript
interface Container<T> {
  value: T;
  getValue(): T;
  setValue(value: T): void;
}

class Box<T> implements Container<T> {
  constructor(public value: T) {}

  getValue(): T {
    return this.value;
  }

  setValue(value: T): void {
    this.value = value;
  }
}

const numberBox = new Box<number>(42);
const stringBox = new Box<string>("hello");
```

### Generic Classes

```typescript
class DataStore<T> {
  private data: T[] = [];

  add(item: T): void {
    this.data.push(item);
  }

  get(index: number): T | undefined {
    return this.data[index];
  }

  getAll(): T[] {
    return this.data;
  }
}

const numberStore = new DataStore<number>();
numberStore.add(1);
numberStore.add(2);

const stringStore = new DataStore<string>();
stringStore.add("hello");
stringStore.add("world");
```

### Generic Constraints

```typescript
// Constraint with extends
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(arg: T): void {
  console.log(arg.length);
}

logLength("hello"); // OK
logLength([1, 2, 3]); // OK
// logLength(42); // Error: number doesn't have length

// Multiple constraints
interface HasId {
  id: number;
}

interface HasName {
  name: string;
}

function printInfo<T extends HasId & HasName>(obj: T): void {
  console.log(`${obj.id}: ${obj.name}`);
}

// Using keyof constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "Alice", age: 30 };
const name = getProperty(person, "name"); // string
const age = getProperty(person, "age"); // number
```

## Utility Types

### Partial

```typescript
// Makes all properties optional
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// Same as: { id?: number; name?: string; email?: string; }

function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates };
}
```

### Required

```typescript
// Makes all properties required
interface Config {
  host?: string;
  port?: number;
}

type RequiredConfig = Required<Config>;
// Same as: { host: string; port: number; }
```

### Readonly

```typescript
// Makes all properties readonly
interface Point {
  x: number;
  y: number;
}

type ReadonlyPoint = Readonly<Point>;
// Same as: { readonly x: number; readonly y: number; }

const point: ReadonlyPoint = { x: 10, y: 20 };
// point.x = 5; // Error
```

### Pick

```typescript
// Pick specific properties
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

type UserPreview = Pick<User, "id" | "name">;
// Same as: { id: number; name: string; }
```

### Omit

```typescript
// Omit specific properties
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type PublicUser = Omit<User, "password">;
// Same as: { id: number; name: string; email: string; }
```

### Record

```typescript
// Create object type with specific keys and value type
type Role = "admin" | "user" | "guest";
type Permissions = Record<Role, string[]>;

const permissions: Permissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"],
};

// Another example
type PageInfo = {
  title: string;
  url: string;
};

type Pages = Record<"home" | "about" | "contact", PageInfo>;
```

### Exclude and Extract

```typescript
// Exclude: Remove types from union
type T1 = Exclude<"a" | "b" | "c", "a">; // "b" | "c"
type T2 = Exclude<string | number | boolean, string>; // number | boolean

// Extract: Extract types from union
type T3 = Extract<"a" | "b" | "c", "a" | "f">; // "a"
type T4 = Extract<string | number | boolean, string | boolean>; // string | boolean
```

### ReturnType

```typescript
// Get return type of function
function getUser() {
  return { id: 1, name: "John" };
}

type User = ReturnType<typeof getUser>;
// Same as: { id: number; name: string; }

type T1 = ReturnType<() => string>; // string
type T2 = ReturnType<(x: number) => number>; // number
```

### Parameters

```typescript
// Get parameters as tuple type
function createUser(name: string, age: number, email: string) {
  return { name, age, email };
}

type CreateUserParams = Parameters<typeof createUser>;
// Same as: [name: string, age: number, email: string]
```

### NonNullable

```typescript
// Remove null and undefined from type
type T1 = NonNullable<string | number | null | undefined>; // string | number
```

## Type Guards

### typeof Type Guards

```typescript
function print(value: string | number): void {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  } else {
    console.log(value.toFixed(2));
  }
}
```

### instanceof Type Guards

```typescript
class Dog {
  bark() {
    console.log("Woof!");
  }
}

class Cat {
  meow() {
    console.log("Meow!");
  }
}

function makeSound(animal: Dog | Cat): void {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}
```

### in Type Guards

```typescript
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

function move(animal: Bird | Fish): void {
  if ("fly" in animal) {
    animal.fly();
  } else {
    animal.swim();
  }
}
```

### Custom Type Guards

```typescript
interface User {
  id: number;
  name: string;
}

interface Admin extends User {
  role: "admin";
  permissions: string[];
}

// Type predicate
function isAdmin(user: User | Admin): user is Admin {
  return (user as Admin).role === "admin";
}

function checkPermissions(user: User | Admin): void {
  if (isAdmin(user)) {
    console.log(user.permissions); // OK
  }
}
```

## Advanced Types

### Conditional Types

```typescript
// T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type T1 = IsString<string>; // true
type T2 = IsString<number>; // false

// Practical example
type NonNullable<T> = T extends null | undefined ? never : T;

type T3 = NonNullable<string | null>; // string
type T4 = NonNullable<number | undefined>; // number
```

### Mapped Types

```typescript
// Create new type by transforming properties
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

interface User {
  id: number;
  name: string;
}

type NullableUser = Nullable<User>;
// Same as: { id: number | null; name: string | null; }

// With modifiers
type ReadonlyPartial<T> = {
  readonly [P in keyof T]?: T[P];
};
```

### Template Literal Types

```typescript
// String manipulation in types
type EventName = "click" | "focus" | "blur";
type Handler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"

// More examples
type Getter<T> = `get${Capitalize<T & string>}`;
type Setter<T> = `set${Capitalize<T & string>}`;

type PropertyGetters = Getter<"name" | "age">;
// "getName" | "getAge"

// Pattern matching
type ExtractRouteParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractRouteParams<`/${Rest}`>
    : T extends `${string}:${infer Param}`
    ? Param
    : never;

type Params = ExtractRouteParams<"/user/:id/post/:postId">;
// "id" | "postId"
```

### Index Access Types

```typescript
interface Person {
  name: string;
  age: number;
  address: {
    street: string;
    city: string;
  };
}

type Name = Person["name"]; // string
type Age = Person["age"]; // number
type Address = Person["address"]; // { street: string; city: string; }
type City = Person["address"]["city"]; // string

// Using with unions
type NameOrAge = Person["name" | "age"]; // string | number
```

## Decorators

### Enable Decorators

```json
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

### Class Decorators

```typescript
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
}
```

### Method Decorators

```typescript
function log(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with args:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };

  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}
```

### Property Decorators

```typescript
function readonly(target: any, propertyKey: string) {
  Object.defineProperty(target, propertyKey, {
    writable: false,
  });
}

class User {
  @readonly
  id: number = 1;
}
```

## Configuration

### tsconfig.json

```json
{
  "compilerOptions": {
    // Target JavaScript version
    "target": "ES2020",

    // Module system
    "module": "commonjs",

    // Module resolution strategy
    "moduleResolution": "node",

    // Output directory
    "outDir": "./dist",

    // Root directory of source files
    "rootDir": "./src",

    // Enable all strict type-checking options
    "strict": true,

    // Individual strict options
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,

    // Additional checks
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,

    // Module options
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,

    // Source map for debugging
    "sourceMap": true,

    // Skip lib check for faster compilation
    "skipLibCheck": true,

    // Emit decorator metadata
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,

    // Path mapping
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    },

    // Include type definitions
    "types": ["node", "jest"],

    // Library files
    "lib": ["ES2020", "DOM"]
  },

  // Files to include
  "include": ["src/**/*"],

  // Files to exclude
  "exclude": ["node_modules", "dist", "**/*.spec.ts"]
}
```

## Best Practices

### Use Strict Mode

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

### Avoid `any`

```typescript
// Bad
function process(data: any) {
  return data;
}

// Good
function process<T>(data: T): T {
  return data;
}

// Or use unknown if type is truly unknown
function process(data: unknown) {
  if (typeof data === "string") {
    return data.toUpperCase();
  }
}
```

### Use Type Inference

```typescript
// Redundant type annotation
const numbers: number[] = [1, 2, 3];

// Better: let TypeScript infer
const numbers = [1, 2, 3];

// But DO annotate function parameters and return types
function add(a: number, b: number): number {
  return a + b;
}
```

### Use Const Assertions

```typescript
// Object is mutable
const config = {
  endpoint: "https://api.example.com",
  timeout: 5000,
};

// Object is deeply readonly
const config = {
  endpoint: "https://api.example.com",
  timeout: 5000,
} as const;

// Array becomes readonly tuple
const colors = ["red", "green", "blue"] as const;
type Color = (typeof colors)[number]; // "red" | "green" | "blue"
```

### Prefer Interfaces for Objects

```typescript
// For object shapes, prefer interfaces
interface User {
  id: number;
  name: string;
}

// Use type aliases for unions, intersections, etc.
type ID = string | number;
type Result = Success | Error;
```

### Use Discriminated Unions

```typescript
// Good pattern for handling different states
type State =
  | { status: "loading" }
  | { status: "success"; data: string }
  | { status: "error"; error: Error };

function handleState(state: State) {
  switch (state.status) {
    case "loading":
      console.log("Loading...");
      break;
    case "success":
      console.log(state.data);
      break;
    case "error":
      console.error(state.error);
      break;
  }
}
```

### Type-safe Event Handlers

```typescript
type EventMap = {
  click: { x: number; y: number };
  keypress: { key: string };
  change: { value: string };
};

class EventEmitter {
  on<K extends keyof EventMap>(
    event: K,
    handler: (data: EventMap[K]) => void
  ): void {
    // Implementation
  }

  emit<K extends keyof EventMap>(event: K, data: EventMap[K]): void {
    // Implementation
  }
}

const emitter = new EventEmitter();
emitter.on("click", (data) => {
  console.log(data.x, data.y); // Type-safe
});
```

## Additional Resources

- [Official TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [TypeScript Playground](https://www.typescriptlang.org/play)
- [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) - Type definitions for JavaScript libraries

---

*Last updated: 2025-11-16*
