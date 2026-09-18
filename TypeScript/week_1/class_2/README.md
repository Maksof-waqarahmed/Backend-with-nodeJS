# 🧩 Basic Types & Type Annotations

## 🌐 Overview

Now that your TypeScript environment is ready, it's time to learn the **building blocks** of the language: types.

In this class we cover every basic type TypeScript offers — from simple `string`/`number` to the special types like `any`, `unknown`, `never`, and `void` — with lots of examples.

---

## 🎯 Learning Goals

* Understand **type annotation** vs **type inference**
* Learn all **primitive types**
* Learn `arrays` and `tuples`
* Understand the special types: `any`, `unknown`, `void`, `never`
* Know when (and when not) to use each one

---

## ✍️ Type Annotation vs Type Inference

**Type annotation** = you explicitly write the type.

```ts
let city: string = "Karachi";
```

**Type inference** = TypeScript automatically figures out the type from the assigned value — you don't have to write it.

```ts
let city = "Karachi"; // TypeScript infers this is a `string`
city = 123; // ❌ Error: Type 'number' is not assignable to type 'string'
```

> 💡 **Best Practice:** Let TypeScript infer types for simple local variables. Use explicit annotations for function parameters, return types, and anything that isn't obvious.

---

## 🔤 Primitive Types

| Type | Description | Example |
|---|---|---|
| `string` | Text values | `"Ali"`, `'Hello'` |
| `number` | Integers & floats (TS has only one number type) | `25`, `3.14`, `-10` |
| `boolean` | `true` / `false` | `true` |
| `bigint` | Very large integers | `9007199254740993n` |
| `symbol` | Unique, immutable identifiers | `Symbol("id")` |
| `null` | Intentional "no value" | `null` |
| `undefined` | A variable that hasn't been assigned yet | `undefined` |

```ts
let firstName: string = "Waqar";
let age: number = 22;
let isDeveloper: boolean = true;
let bigNumber: bigint = 1234567890123456789n;
let id: symbol = Symbol("user-id");
let empty: null = null;
let notAssigned: undefined = undefined;
```

---

## 📦 Arrays

An array's element type can be declared in two equivalent ways:

```ts
let scores: number[] = [90, 85, 77];
let names: Array<string> = ["Ali", "Sara", "Ahmed"];
```

### Array of a Union Type

```ts
let mixed: (string | number)[] = ["Ali", 22, "Sara", 30];
```

### Array of Objects

```ts
let students: { name: string; age: number }[] = [
  { name: "Ali", age: 20 },
  { name: "Sara", age: 22 },
];
```

If you try to push something that doesn't match, TypeScript blocks it:

```ts
scores.push("100"); // ❌ Error: string is not assignable to number
```

---

## 🎯 Tuples

A **tuple** is a fixed-length array where **each position has its own specific type**.

```ts
let user: [string, number, boolean] = ["Ali", 22, true];
//          name      age     isActive
```

Unlike a regular array, order and count matter:

```ts
let point: [number, number] = [10, 20]; // x, y
point = [10, "20"]; // ❌ Error
point = [10, 20, 30]; // ❌ Error: too many elements
```

### 🧠 Real Use Case: `useState` in React

React's `useState` returns a tuple: `[value, setterFunction]`. That's exactly the same pattern:

```ts
type UseStateResult = [number, (newVal: number) => void];
```

---

## ⚠️ The `any` Type

`any` **disables type checking completely** for that variable — it can become anything.

```ts
let randomValue: any = "Hello";
randomValue = 42;
randomValue = true;
randomValue.someRandomMethod(); // No error, even though it doesn't exist!
```

> 🚫 **Avoid `any` whenever possible.** It defeats the entire purpose of using TypeScript. Only use it as a last resort — e.g. while gradually migrating old JavaScript code.

---

## 🛡️ The `unknown` Type (Safer Alternative)

`unknown` is like `any`, but **TypeScript forces you to check the type before using it**.

```ts
let value: unknown = "Hello";

value.toUpperCase(); // ❌ Error: Object is of type 'unknown'

if (typeof value === "string") {
  value.toUpperCase(); // ✅ Safe now — TypeScript knows it's a string here
}
```

| | `any` | `unknown` |
|---|---|---|
| Type checking | ❌ Off completely | ✅ Still enforced |
| Must verify before use | ❌ No | ✅ Yes |
| Recommended | ❌ Avoid | ✅ Prefer this |

---

## 🚫 The `never` Type

`never` represents a value that **can never happen** — used for functions that always throw an error or never finish (infinite loops).

```ts
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}
```

---

## 🕳️ The `void` Type

`void` means a function **doesn't return anything meaningful**.

```ts
function logMessage(message: string): void {
  console.log(message);
  // no return statement
}
```

| Type | Used For |
|---|---|
| `void` | Functions that don't return a useful value |
| `never` | Functions that never successfully complete (always throw / infinite loop) |

---

## 🧮 `null` and `undefined` in Strict Mode

With `"strict": true` in `tsconfig.json`, TypeScript treats `null` and `undefined` as **separate, specific types** — they are NOT automatically allowed inside other types.

```ts
let username: string = null; // ❌ Error in strict mode
let username: string | null = null; // ✅ You must explicitly allow it
```

This forces you to **handle missing values on purpose**, preventing the infamous *"Cannot read property of undefined"* runtime crash.

---

## 🧾 Full Example Program

```ts
// src/index.ts

let name: string = "Hina";
let age: number = 21;
let isStudent: boolean = true;
let hobbies: string[] = ["reading", "coding", "painting"];
let coordinates: [number, number] = [24.86, 67.01];

function describePerson(
  name: string,
  age: number,
  isStudent: boolean
): string {
  return `${name} is ${age} years old and is ${
    isStudent ? "a student" : "not a student"
  }.`;
}

console.log(describePerson(name, age, isStudent));
console.log("Hobbies:", hobbies.join(", "));
console.log("Location:", coordinates);
```

**Output:**

```
Hina is 21 years old and is a student.
Hobbies: reading, coding, painting
Location: [ 24.86, 67.01 ]
```

---

## 🧾 Summary Table

| Type | Use Case |
|---|---|
| `string`, `number`, `boolean` | Basic values |
| `bigint`, `symbol` | Rare, special-purpose values |
| `array` (`T[]`) | List of same-type items |
| `tuple` (`[T, U]`) | Fixed-length list with different types per position |
| `any` | Avoid — disables all type safety |
| `unknown` | Safer alternative to `any` |
| `void` | Function returns nothing |
| `never` | Function never returns (throws / infinite loop) |
| `null` / `undefined` | Explicit "no value", must be declared with `\| null` in strict mode |

---

## 🧩 Hands-On Practice

1. Declare variables for a `Product`: `title` (string), `price` (number), `inStock` (boolean), `tags` (string array).
2. Create a tuple `[string, number]` representing `[productName, quantity]`.
3. Write a function `getDiscountedPrice(price: number, discountPercent: number): number`.
4. Write a function that accepts `unknown` input and safely checks if it's a `number` before doubling it.
5. Try assigning `null` to a `string` variable with `strict: true` and observe the error.

Next class (Week 2): **Objects, Interfaces & Type Aliases** — where we start modeling real-world data structures. 🚀
