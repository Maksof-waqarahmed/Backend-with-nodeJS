# 🧩 Generics — Writing Flexible, Reusable, Type-Safe Code

## 🌐 Overview

So far, every function/interface we've written has worked with **one specific type**. But what if you want a function that works safely with **any type**, while still keeping full type safety? That's exactly what **Generics** solve — one of the most powerful (and most confusing at first!) features of TypeScript.

---

## 🎯 Learning Goals

* Understand the problem generics solve
* Write generic functions
* Write generic interfaces and types
* Write generic classes
* Use generic constraints (`extends`)
* Use default generic types
* Recognize generics in built-in types you already use (`Array<T>`, `Promise<T>`)

---

## ❓ The Problem Generics Solve

Imagine a function that returns whatever you pass into it:

```ts
function identity(value: any): any {
  return value;
}

const result = identity("Hello");
result.toUpperCase(); // ✅ works, but TypeScript has NO idea result is a string — no safety!

const result2 = identity(42);
result2.toUpperCase(); // 😱 no compile error, but this will crash at runtime!
```

Using `any` throws away all type safety. We need a way to say: *"Whatever type goes in, the exact same type comes out"* — without hardcoding a specific type.

---

## ✅ The Solution: Generics

```ts
function identity<T>(value: T): T {
  return value;
}

const result = identity<string>("Hello");
result.toUpperCase(); // ✅ TypeScript KNOWS result is a string

const result2 = identity(42); // TypeScript infers T = number automatically
result2.toFixed(2);            // ✅ Safe
result2.toUpperCase();         // ❌ Error: number has no method toUpperCase
```

`T` is a **type parameter** — a placeholder for "whatever type is used at call time". You could name it anything (`T`, `U`, `Item`, etc.), but `T` is the common convention for "Type".

> 💡 You usually don't need to write `identity<string>(...)` explicitly — TypeScript **infers** the type from the argument automatically.

---

## 🧩 Generic Functions with Arrays

```ts
function getFirstElement<T>(arr: T[]): T {
  return arr[0];
}

const firstNum = getFirstElement<number>([1, 2, 3]);        // 1 (number)
const firstName = getFirstElement<string>(["Ali", "Sara"]); // "Ali" (string)
```

### Multiple Type Parameters

```ts
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const result = pair<string, number>("age", 25);
console.log(result); // ["age", 25]
```

---

## 🧱 Generic Interfaces

```ts
interface ApiResponse<T> {
  success: boolean;
  data: T;
  message: string;
}

interface User {
  id: number;
  name: string;
}

const userResponse: ApiResponse<User> = {
  success: true,
  data: { id: 1, name: "Ali" },
  message: "User fetched successfully",
};

const productListResponse: ApiResponse<string[]> = {
  success: true,
  data: ["Laptop", "Mouse"],
  message: "Products fetched successfully",
};
```

> 💡 This is **exactly** the shape used throughout this repo's `Backend/src/*/**.controllers.ts` files — every response is `{ success, message, data }`. Making it generic (`ApiResponse<T>`) lets you reuse ONE type for every endpoint, instead of writing a new interface for each one.

---

## 🧱 Generic Type Aliases

```ts
type Pair<T, U> = {
  first: T;
  second: U;
};

const coordinate: Pair<number, number> = { first: 10, second: 20 };
const entry: Pair<string, boolean> = { first: "isActive", second: true };
```

---

## 🏛️ Generic Classes

```ts
class Box<T> {
  private contents: T;

  constructor(value: T) {
    this.contents = value;
  }

  getContents(): T {
    return this.contents;
  }

  setContents(value: T): void {
    this.contents = value;
  }
}

const numberBox = new Box<number>(42);
console.log(numberBox.getContents()); // 42

const stringBox = new Box<string>("Hello");
console.log(stringBox.getContents()); // "Hello"
```

### 🧠 Real Example: A Generic Repository Pattern (common in backend apps)

```ts
class Repository<T extends { id: number }> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  findById(id: number): T | undefined {
    return this.items.find((item) => item.id === id);
  }

  getAll(): T[] {
    return this.items;
  }
}

interface Product {
  id: number;
  title: string;
}

const productRepo = new Repository<Product>();
productRepo.add({ id: 1, title: "Laptop" });
productRepo.add({ id: 2, title: "Mouse" });

console.log(productRepo.findById(1)); // { id: 1, title: 'Laptop' }
```

---

## 🔒 Generic Constraints (`extends`)

Sometimes you need to limit **what kinds of types** are allowed — not truly "anything". Use `extends` to add a constraint.

```ts
function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}

getLength("Hello");        // ✅ strings have .length
getLength([1, 2, 3]);      // ✅ arrays have .length
getLength(42);              // ❌ Error: number doesn't have .length
```

### Constraining to Object Keys with `keyof`

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Ali", age: 25 };

getProperty(user, "name"); // ✅ "Ali"
getProperty(user, "age");  // ✅ 25
getProperty(user, "email"); // ❌ Error: 'email' is not a key of user
```

---

## 🎯 Default Generic Types

You can give a type parameter a fallback, used when none is specified:

```ts
interface ApiResponse<T = unknown> {
  success: boolean;
  data: T;
}

const response: ApiResponse = { success: true, data: "anything" }; // T defaults to unknown
```

---

## 🧠 Generics You're Already Using

You've been using generics since Week 1 without realizing it!

```ts
let numbers: Array<number> = [1, 2, 3];     // Array<T>
let user: Promise<User>;                     // Promise<T>
let map: Map<string, number> = new Map();    // Map<K, V>
```

| Built-in | Generic Meaning |
|---|---|
| `Array<T>` | An array containing items of type `T` |
| `Promise<T>` | A promise that eventually resolves to a value of type `T` |
| `Map<K, V>` | A key-value map where keys are `K` and values are `V` |
| `Record<K, V>` | An object type with keys `K` mapped to values `V` (covered next class) |

---

## 🧾 Full Example

```ts
interface ApiResponse<T> {
  success: boolean;
  message: string;
  data: T;
}

function createResponse<T>(data: T, message: string = "OK"): ApiResponse<T> {
  return { success: true, message, data };
}

interface Product {
  id: number;
  title: string;
  price: number;
}

const singleProduct = createResponse<Product>(
  { id: 1, title: "Laptop", price: 1200 },
  "Product fetched successfully"
);

const productList = createResponse<Product[]>([
  { id: 1, title: "Laptop", price: 1200 },
  { id: 2, title: "Mouse", price: 20 },
]);

console.log(singleProduct.data.title); // "Laptop"
console.log(productList.data.length);  // 2
```

---

## 🧾 Summary

| Concept | Syntax | Meaning |
|---|---|---|
| Generic function | `function fn<T>(x: T): T` | Works with any type, preserving type safety |
| Generic interface | `interface Box<T> { value: T }` | Reusable shape for any data type |
| Generic class | `class Repo<T> { ... }` | Reusable class logic for any data type |
| Multiple parameters | `<T, U>` | More than one placeholder type |
| Constraint | `<T extends SomeType>` | Restrict which types are allowed |
| `keyof` constraint | `<K extends keyof T>` | Restrict to valid property names of `T` |
| Default type | `<T = unknown>` | Fallback type when none is provided |

---

## 🧩 Hands-On Practice

1. Write a generic function `wrapInArray<T>(value: T): T[]` that returns `[value]`.
2. Write a generic interface `Pair<T, U>` and a function `swap<T, U>(pair: Pair<T, U>): Pair<U, T>`.
3. Write a generic class `Stack<T>` with `push(item: T)`, `pop(): T | undefined`, and `peek(): T | undefined`.
4. Write a generic function `getRandomItem<T>(items: T[]): T`.
5. Write a constrained generic function `hasProperty<T extends object, K extends keyof T>(obj: T, key: K): boolean`.
6. Create a generic `ApiResponse<T>` interface (like the example above) and use it for two different endpoints: `User` and `Product`.

Next class: **Utility Types, Mapped Types & Conditional Types.** 🚀
