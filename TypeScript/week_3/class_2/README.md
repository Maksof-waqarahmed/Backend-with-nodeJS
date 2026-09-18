# 🎯 Enums & Tuples Deep Dive

## 🌐 Overview

You already saw tuples briefly in Week 1. This class goes deeper into **tuples** (labeled tuples, optional elements) and introduces **enums** — a dedicated TypeScript feature for representing a fixed set of named constants, plus a comparison with literal unions (which many teams prefer today).

---

## 🎯 Learning Goals

* Master advanced tuple patterns
* Understand numeric vs string enums
* Understand `const enum`
* Compare enums vs literal union types
* Know when to use each one

---

## 🎯 Tuples — A Recap

```ts
let point: [number, number] = [10, 20];
let user: [string, number, boolean] = ["Ali", 25, true];
```

### 🏷️ Labeled Tuples (Better Readability)

```ts
let coordinate: [x: number, y: number] = [24.86, 67.01];
```

Labels don't change behavior — they just make the tuple **self-documenting** in your editor's tooltips.

### 🔓 Optional Tuple Elements

```ts
let userInfo: [name: string, age: number, email?: string];

userInfo = ["Ali", 25];                       // ✅ email omitted
userInfo = ["Sara", 22, "sara@mail.com"];     // ✅
```

### 📦 Rest Elements in Tuples

```ts
let scores: [subject: string, ...marks: number[]];

scores = ["Math", 90, 85, 95];       // ✅
scores = ["Science", 88];             // ✅
```

### 🧠 Real-World Tuple Example: A Function That Returns Multiple Values

```ts
function divide(a: number, b: number): [result: number, remainder: number] {
  return [Math.floor(a / b), a % b];
}

const [result, remainder] = divide(17, 5);
console.log(result, remainder); // 3 2
```

---

## 🔢 Enums — Named Constants

An **enum** ("enumerated type") gives friendly names to a fixed set of related values.

### Numeric Enum (default)

```ts
enum OrderStatus {
  Pending,   // 0
  Shipped,   // 1
  Delivered, // 2
  Cancelled, // 3
}

let status: OrderStatus = OrderStatus.Shipped;
console.log(status); // 1
console.log(OrderStatus[1]); // "Shipped" — reverse lookup!
```

By default, the first member is `0` and each next one increments by `1`. You can also set a custom starting number:

```ts
enum OrderStatus {
  Pending = 1,
  Shipped,   // 2
  Delivered, // 3
  Cancelled, // 4
}
```

### String Enum (Recommended — More Readable & Debuggable)

```ts
enum Role {
  Admin = "ADMIN",
  Editor = "EDITOR",
  Viewer = "VIEWER",
}

let userRole: Role = Role.Admin;
console.log(userRole); // "ADMIN"
```

> 💡 **String enums are usually preferred** because `console.log`/debugging shows a meaningful value (`"ADMIN"`) instead of a confusing number (`0`).

---

## 🧠 Using Enums in Functions

```ts
enum PaymentMethod {
  Card = "CARD",
  Cash = "CASH",
  PayPal = "PAYPAL",
}

function processOrder(amount: number, method: PaymentMethod): void {
  console.log(`Processing $${amount} via ${method}`);
}

processOrder(500, PaymentMethod.Card); // "Processing $500 via CARD"
```

---

## ⚡ `const enum` (Performance Optimized)

A `const enum` is completely **removed during compilation** and replaced directly with its value — producing smaller, faster JavaScript. Use it when you don't need reverse lookups.

```ts
const enum Direction {
  Up,
  Down,
  Left,
  Right,
}

let move: Direction = Direction.Up;
```

Compiles down to just:

```js
let move = 0;
```

---

## ⚔️ Enums vs Literal Union Types

Modern TypeScript codebases (and this repo's own backend!) often prefer **literal string unions** over enums for simple cases:

```ts
// Enum approach
enum Status {
  Pending = "PENDING",
  Paid = "PAID",
}

// Literal union approach (used throughout Backend/src models in this project)
type Status = "pending" | "paid";
```

| | Enum | Literal Union |
|---|---|---|
| Extra JS output | ✅ Yes (an object is generated) | ❌ No (types vanish after compile) |
| Reverse lookup (`Enum[0]`) | ✅ Yes (numeric enums) | ❌ No |
| Works great with Mongoose `enum: [...]` | ⚠️ Needs `Object.values(Enum)` | ✅ Very natural |
| Simplicity | Slightly more ceremony | Very lightweight |
| Team preference today | Large, formal codebases | Most modern TS/Node projects |

**Example from this repository:** `Backend/src/order/order.model.ts` uses a literal union, not an enum:

```ts
status: "pending" | "paid" | "shipped" | "completed" | "cancelled";
```

> 💡 **General guideline:** Prefer literal union types for simple fixed-value fields (like this repo does). Reach for `enum` when you need reverse lookups, or when the set of values is shared and referenced by name across many files in a large team codebase.

---

## 🧾 Full Example

```ts
enum UserRole {
  Admin = "ADMIN",
  Customer = "CUSTOMER",
}

interface User {
  id: number;
  name: string;
  role: UserRole;
}

function canDeleteProduct(user: User): boolean {
  return user.role === UserRole.Admin;
}

const admin: User = { id: 1, name: "Waqar", role: UserRole.Admin };
const customer: User = { id: 2, name: "Ali", role: UserRole.Customer };

console.log(canDeleteProduct(admin));    // true
console.log(canDeleteProduct(customer)); // false
```

---

## 🧾 Summary

| Concept | Example | Notes |
|---|---|---|
| Labeled tuple | `[x: number, y: number]` | Improves readability in tooltips |
| Optional tuple element | `[string, number?]` | Must come after required elements |
| Rest tuple element | `[string, ...number[]]` | Variable-length tail |
| Numeric enum | `enum Status { Pending, Paid }` | Auto-increments from 0 |
| String enum | `enum Status { Pending = "PENDING" }` | More readable when debugging |
| `const enum` | `const enum Direction { Up, Down }` | Fully inlined, no runtime object |
| Literal union (alternative) | `type Status = "pending" \| "paid"` | Lighter weight, very common in Node/Express backends |

---

## 🧩 Hands-On Practice

1. Create a labeled tuple type `[isbn: string, title: string, price: number]` for a `Book`.
2. Create a numeric enum `enum Weekday { Monday, Tuesday, ..., Sunday }` and write a function that returns `"Weekend"` or `"Weekday"` based on it.
3. Create a string enum `enum HttpStatus { OK = "200", NotFound = "404", ServerError = "500" }`.
4. Rewrite question 3 as a literal union type instead, and compare readability.
5. Write a function returning a tuple `[isValid: boolean, errorMessage?: string]` from a simple password validator.

Next week (Week 4): **Classes in TypeScript** — constructors, access modifiers, inheritance, and abstract classes. 🚀
