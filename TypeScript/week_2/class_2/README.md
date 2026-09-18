# ⚙️ Functions in TypeScript

## 🌐 Overview

Functions are the heart of any program. In this class, you'll learn how TypeScript adds **type safety to function parameters and return values**, plus advanced patterns like optional/default/rest parameters, function types, and overloads.

---

## 🎯 Learning Goals

* Type function parameters and return values
* Understand optional, default, and rest parameters
* Type arrow functions
* Define reusable function types
* Understand function overloading
* Understand the `this` keyword in TypeScript

---

## 🧩 Basic Function Typing

```ts
function add(a: number, b: number): number {
  return a + b;
}

add(5, 10);     // ✅ 15
add("5", 10);   // ❌ Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

The `: number` after the parentheses is the **return type**.

---

## 🏹 Arrow Functions

```ts
const multiply = (a: number, b: number): number => a * b;

const greet = (name: string): string => `Hello, ${name}!`;
```

---

## 🔓 Optional Parameters (`?`)

A parameter marked `?` is not required. It must come **after** all required parameters.

```ts
function greetUser(name: string, title?: string): string {
  return title ? `Hello, ${title} ${name}` : `Hello, ${name}`;
}

greetUser("Ali");            // ✅ "Hello, Ali"
greetUser("Ali", "Dr.");     // ✅ "Hello, Dr. Ali"
```

---

## 🎯 Default Parameters

Give a parameter a default value — it becomes optional automatically.

```ts
function calculateTotal(price: number, taxRate: number = 0.1): number {
  return price + price * taxRate;
}

calculateTotal(100);       // ✅ uses default taxRate 0.1 → 110
calculateTotal(100, 0.15); // ✅ 115
```

---

## 📦 Rest Parameters

Collect an unlimited number of arguments into a typed array.

```ts
function sumAll(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sumAll(1, 2, 3);       // 6
sumAll(10, 20, 30, 40); // 100
```

---

## 🧠 Function Types (Typing a Variable That Holds a Function)

You can describe the **shape of a function** — its parameters and return type — as a type.

```ts
let operation: (a: number, b: number) => number;

operation = (a, b) => a + b; // ✅ matches the shape
operation = (a, b) => `${a}${b}`; // ❌ Error: returns string, not number
```

### Reusable Function Type with `type`

```ts
type MathOperation = (a: number, b: number) => number;

const add: MathOperation = (a, b) => a + b;
const subtract: MathOperation = (a, b) => a - b;
```

### Function as a Parameter (Callback)

```ts
function calculate(a: number, b: number, operation: MathOperation): number {
  return operation(a, b);
}

console.log(calculate(10, 5, add));      // 15
console.log(calculate(10, 5, subtract)); // 5
```

---

## 🚫 `void` Return Type

Use `void` for functions that perform an action but return nothing.

```ts
function logOrder(orderId: number): void {
  console.log(`Order #${orderId} received.`);
}
```

---

## 🧮 Function Overloading

Sometimes a function needs to behave differently depending on the **types** of arguments passed. **Overloading** lets you define multiple valid call signatures for one function.

```ts
// Overload signatures
function formatInput(input: string): string;
function formatInput(input: number): string;

// Implementation (must handle all overloads)
function formatInput(input: string | number): string {
  if (typeof input === "number") {
    return `Number: ${input.toFixed(2)}`;
  }
  return `Text: ${input.toUpperCase()}`;
}

console.log(formatInput("hello")); // "Text: HELLO"
console.log(formatInput(42));       // "Number: 42.00"
```

> 💡 Overload signatures tell callers exactly what combinations are valid, while the single implementation underneath handles all cases safely.

---

## 🧩 Typing Object Parameters (Destructuring)

Instead of passing many separate parameters, pass a typed object — very common in real backend code.

```ts
interface CreateUserInput {
  name: string;
  email: string;
  age?: number;
}

function createUser({ name, email, age }: CreateUserInput): void {
  console.log(`Creating user ${name} (${email}), age: ${age ?? "N/A"}`);
}

createUser({ name: "Sara", email: "sara@mail.com" });
createUser({ name: "Ali", email: "ali@mail.com", age: 25 });
```

> Notice `age ?? "N/A"` — the **nullish coalescing operator** — returns `"N/A"` only when `age` is `null` or `undefined`.

---

## 🌀 Functions That Return Promises (Async Functions)

Backend code constantly deals with asynchronous operations (database calls, API requests). TypeScript types the resolved value inside `Promise<T>`.

```ts
function fetchUserById(id: number): Promise<{ id: number; name: string }> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({ id, name: "Ali" });
    }, 1000);
  });
}

async function main() {
  const user = await fetchUserById(1);
  console.log(user.name); // ✅ TypeScript knows this is a string
}
```

---

## 🧾 Full Example

```ts
interface Product {
  title: string;
  price: number;
  quantity: number;
}

function calculateInvoiceTotal(products: Product[], discount: number = 0): number {
  const subtotal = products.reduce(
    (total, product) => total + product.price * product.quantity,
    0
  );
  return subtotal - subtotal * (discount / 100);
}

const cart: Product[] = [
  { title: "Keyboard", price: 50, quantity: 2 },
  { title: "Mouse", price: 20, quantity: 1 },
];

console.log(calculateInvoiceTotal(cart));       // 120
console.log(calculateInvoiceTotal(cart, 10));   // 108 (10% discount)
```

---

## 🧾 Summary

| Concept | Syntax | Example |
|---|---|---|
| Typed parameters/return | `(a: T): R` | `function add(a: number): number` |
| Optional parameter | `param?: T` | `title?: string` |
| Default parameter | `param: T = value` | `taxRate: number = 0.1` |
| Rest parameter | `...param: T[]` | `...numbers: number[]` |
| Function type | `(a: T) => R` | `type Op = (a: number) => number` |
| No return value | `void` | `function log(): void` |
| Overloading | multiple signatures + 1 implementation | `formatInput(string)` / `formatInput(number)` |
| Async return | `Promise<T>` | `Promise<User>` |

---

## 🧩 Hands-On Practice

1. Write `isEven(num: number): boolean`.
2. Write `buildFullName(first: string, last: string, middle?: string): string`.
3. Write `applyDiscount(price: number, percent: number = 5): number`.
4. Write `sumOfAll(...values: number[]): number` using rest parameters.
5. Create a `type Validator = (value: string) => boolean` and write two validator functions matching that type (e.g. `isNotEmpty`, `isEmail`).
6. Write an overloaded function `combine` that concatenates two strings OR adds two numbers.

Next class (Week 3): **Union & Intersection Types, Literal Types, and Type Narrowing.** 🚀
