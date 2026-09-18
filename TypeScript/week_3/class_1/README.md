# 🔀 Union & Intersection Types, Literal Types, and Type Narrowing

## 🌐 Overview

Real data isn't always one fixed type — a value might be "either this or that". This class covers how TypeScript expresses **flexibility with safety** using unions, intersections, literal types, and the powerful concept of **type narrowing**.

---

## 🎯 Learning Goals

* Understand **union types** (`|`)
* Understand **intersection types** (`&`)
* Understand **literal types**
* Master **type narrowing** techniques (`typeof`, `instanceof`, `in`, truthiness)
* Understand **discriminated unions** — a key real-world pattern

---

## 🔀 Union Types (`|`)

A union type means a value **can be one of several types**.

```ts
let id: string | number;

id = 101;      // ✅
id = "A-101";  // ✅
id = true;     // ❌ Error
```

### Union in Function Parameters

```ts
function printId(id: string | number): void {
  console.log(`Your ID is: ${id}`);
}

printId(101);     // ✅
printId("A-101"); // ✅
```

---

## ⚠️ The Problem: You Can't Use Type-Specific Methods Right Away

```ts
function printId(id: string | number): void {
  console.log(id.toUpperCase()); // ❌ Error: 'toUpperCase' does not exist on type 'number'
}
```

TypeScript only allows operations that are **valid for every type in the union**. This is where **narrowing** comes in.

---

## 🔍 Type Narrowing

Narrowing means **checking the type first**, so TypeScript "narrows down" which specific type you're working with inside that block.

### 1️⃣ `typeof` Narrowing

```ts
function printId(id: string | number): void {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // ✅ Safe — TS knows id is a string here
  } else {
    console.log(id.toFixed(2));    // ✅ Safe — TS knows id is a number here
  }
}
```

### 2️⃣ `instanceof` Narrowing (for classes)

```ts
class Dog {
  bark() { console.log("Woof!"); }
}
class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark(); // ✅
  } else {
    animal.meow(); // ✅
  }
}
```

### 3️⃣ `in` Narrowing (checking if a property exists)

```ts
interface Car {
  drive(): void;
}
interface Boat {
  sail(): void;
}

function travel(vehicle: Car | Boat) {
  if ("drive" in vehicle) {
    vehicle.drive(); // ✅
  } else {
    vehicle.sail();  // ✅
  }
}
```

### 4️⃣ Truthiness Narrowing

```ts
function printLength(value?: string) {
  if (value) {
    console.log(value.length); // ✅ value is definitely a string here, not undefined
  }
}
```

### 5️⃣ Array.isArray() Narrowing

```ts
function process(input: string | string[]) {
  if (Array.isArray(input)) {
    console.log(input.join(", ")); // ✅ input is string[]
  } else {
    console.log(input.toUpperCase()); // ✅ input is string
  }
}
```

---

## 🏷️ Literal Types

A **literal type** restricts a value to one **exact, specific value**, instead of an entire type category.

```ts
let direction: "left" | "right" | "up" | "down";

direction = "left";  // ✅
direction = "sideways"; // ❌ Error
```

### Literal Types + Functions (very common backend pattern)

```ts
type OrderStatus = "pending" | "shipped" | "delivered" | "cancelled";

function updateOrderStatus(orderId: number, status: OrderStatus): void {
  console.log(`Order ${orderId} is now ${status}`);
}

updateOrderStatus(1, "shipped");  // ✅
updateOrderStatus(1, "on hold");  // ❌ Error — not one of the allowed values
```

> 💡 This exact pattern is used in this repository's own backend — see `Backend/src/order/order.model.ts`, where `status` is typed as `"pending" | "paid" | "shipped" | "completed" | "cancelled"`.

---

## 🧬 Intersection Types (`&`)

While a **union** means "one OR the other", an **intersection** means "this AND that **combined**" — the result must satisfy **all** the combined types.

```ts
type Person = {
  name: string;
};

type Employee = {
  salary: number;
};

type StaffMember = Person & Employee;

const staff: StaffMember = {
  name: "Bilal",
  salary: 50000,
}; // must have BOTH name and salary
```

---

## 🧩 Discriminated Unions (Powerful Real-World Pattern)

A **discriminated union** is a union of object types that all share a common **literal property** (the "discriminant") — which lets TypeScript automatically narrow the exact shape inside `if`/`switch` blocks.

```ts
interface SuccessResponse {
  status: "success";
  data: { id: number; name: string };
}

interface ErrorResponse {
  status: "error";
  message: string;
}

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse) {
  if (response.status === "success") {
    console.log(response.data.name); // ✅ TS knows this is SuccessResponse
  } else {
    console.log(response.message);   // ✅ TS knows this is ErrorResponse
  }
}
```

This pattern is extremely common in API response handling, Redux actions, and form validation results.

---

## 🧾 Full Example

```ts
type PaymentMethod = "card" | "cash" | "paypal";

interface Order {
  id: number;
  amount: number;
  paymentMethod: PaymentMethod;
}

function processPayment(order: Order): string {
  switch (order.paymentMethod) {
    case "card":
      return `Charging card for order #${order.id}`;
    case "cash":
      return `Marking order #${order.id} as Cash on Delivery`;
    case "paypal":
      return `Redirecting to PayPal for order #${order.id}`;
    default:
      // If a new PaymentMethod is ever added and not handled here,
      // TypeScript will flag this line as unreachable — a great safety net!
      const _exhaustiveCheck: never = order.paymentMethod;
      return _exhaustiveCheck;
  }
}

const order1: Order = { id: 1, amount: 500, paymentMethod: "card" };
console.log(processPayment(order1));
```

**Output:**

```
Charging card for order #1
```

> 💡 The `never` trick in the `default` case is called an **exhaustiveness check** — if someone adds `"bank-transfer"` to `PaymentMethod` later but forgets to handle it in the switch, TypeScript will immediately show a compile error.

---

## 🧾 Summary

| Concept | Symbol | Meaning |
|---|---|---|
| Union | `\|` | Value can be one of several types |
| Intersection | `&` | Value must satisfy all combined types |
| Literal type | `"exact value"` | Restrict to specific fixed values |
| Narrowing | `typeof`, `instanceof`, `in`, truthiness | Safely access type-specific features inside a union |
| Discriminated union | shared literal field | Auto-narrows object shape based on one property |
| Exhaustiveness check | `never` | Compiler warns if a new case isn't handled |

---

## 🧩 Hands-On Practice

1. Create `type Role = "admin" | "editor" | "viewer"` and a function that returns different permission messages per role.
2. Write a function `describe(value: string | number | boolean)` that uses `typeof` narrowing to handle all three cases differently.
3. Create two interfaces `Circle { kind: "circle"; radius: number }` and `Square { kind: "square"; side: number }`. Build a discriminated union `Shape` and write a function `getArea(shape: Shape): number`.
4. Create an intersection type `type AdminUser = User & { permissions: string[] }`.
5. Add an exhaustiveness check (`never`) to your `getArea` function's switch statement.

Next class: **Enums & Tuples Deep Dive.** 🚀
