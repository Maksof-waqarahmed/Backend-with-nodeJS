# 🧱 Objects, Interfaces & Type Aliases

## 🌐 Overview

Real applications are built around **structured data** — users, products, orders. This class teaches you how to describe the *shape* of that data using **object types**, **interfaces**, and **type aliases** — the most-used TypeScript features in everyday backend/frontend code.

---

## 🎯 Learning Goals

* Type object literals directly
* Create reusable structures with `interface`
* Create reusable structures with `type`
* Understand optional (`?`) and `readonly` properties
* Know the real difference between `interface` and `type`
* Extend interfaces and combine types

---

## 🧩 Typing an Object Directly

```ts
let user: { name: string; age: number; isAdmin: boolean } = {
  name: "Waqar",
  age: 25,
  isAdmin: true,
};
```

This works, but writing the shape inline every time gets repetitive. That's where **interfaces** and **type aliases** come in.

---

## 📘 Interfaces

An **interface** defines a reusable contract — the exact shape an object must follow.

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

const user1: User = {
  id: 1,
  name: "Ali",
  email: "ali@example.com",
};
```

If you're missing a property, or you add one that's not defined, TypeScript throws a compile-time error:

```ts
const user2: User = { id: 2, name: "Sara" };
// ❌ Error: Property 'email' is missing
```

---

## 🔓 Optional Properties (`?`)

Use `?` when a property is not always required.

```ts
interface Person {
  name: string;
  age: number;
  address?: string; // optional
}

const p1: Person = { name: "Ali", age: 20 };               // ✅ valid
const p2: Person = { name: "Sara", age: 22, address: "Karachi" }; // ✅ valid
```

---

## 🔒 Readonly Properties

`readonly` prevents a property from being changed **after** the object is created — useful for IDs, creation dates, etc.

```ts
interface Product {
  readonly id: number;
  title: string;
  price: number;
}

const product: Product = { id: 101, title: "Laptop", price: 1200 };

product.title = "Gaming Laptop"; // ✅ allowed
product.id = 999;                 // ❌ Error: Cannot assign to 'id' because it is a read-only property
```

---

## 🧱 Type Aliases

A **type alias** does almost the same job, using the `type` keyword:

```ts
type User = {
  name: string;
  email: string;
  age?: number;
};

const u1: User = { name: "Ali", email: "ali@gmail.com" };
```

### Type Aliases Can Also Represent Non-Object Types

This is something interfaces **cannot** do:

```ts
type ID = string | number;          // union
type Status = "active" | "inactive"; // literal union
type Point = [number, number];       // tuple
```

---

## ⚔️ Interface vs Type Alias — The Real Difference

| Feature | `interface` | `type` |
|---|---|---|
| Object shapes | ✅ Yes | ✅ Yes |
| Primitives / Unions / Tuples | ❌ No | ✅ Yes |
| Can be extended (`extends`) | ✅ Yes | ✅ Yes (via `&`) |
| Declaration merging (same name adds properties) | ✅ Yes | ❌ No |
| Common convention | Object / API models | Unions, function types, utility combos |

### Declaration Merging Example (only interfaces can do this)

```ts
interface Car {
  brand: string;
}

interface Car {
  year: number;
}

const myCar: Car = { brand: "Toyota", year: 2023 }; // ✅ merged automatically
```

> 💡 **Practical rule of thumb:** Use `interface` for objects/models (users, products, API responses). Use `type` for unions, function signatures, and combining multiple types together.

---

## 🔄 Extending Interfaces

```ts
interface User {
  id: number;
  name: string;
}

interface Admin extends User {
  role: string;
  permissions: string[];
}

const admin: Admin = {
  id: 1,
  name: "Waqar",
  role: "Super Admin",
  permissions: ["create", "delete", "update"],
};
```

You can extend from **multiple interfaces** too:

```ts
interface Timestamped {
  createdAt: Date;
}

interface Post extends User, Timestamped {
  title: string;
  content: string;
}
```

---

## 🧬 Combining Types with Intersections (`&`)

Type aliases achieve the same result using `&`:

```ts
type User = {
  id: number;
  name: string;
};

type Timestamped = {
  createdAt: Date;
};

type Post = User & Timestamped & {
  title: string;
  content: string;
};
```

---

## 🧩 Nested Objects

Real-world data is rarely flat. You can nest interfaces inside other interfaces:

```ts
interface Address {
  city: string;
  country: string;
}

interface Customer {
  name: string;
  email: string;
  address: Address; // nested object
}

const customer: Customer = {
  name: "Ayesha",
  email: "ayesha@example.com",
  address: {
    city: "Lahore",
    country: "Pakistan",
  },
};
```

---

## 🧮 Index Signatures (Dynamic Keys)

Sometimes you don't know the exact key names in advance — like a dictionary/map.

```ts
interface Scores {
  [studentName: string]: number;
}

const marks: Scores = {
  Ali: 85,
  Sara: 92,
  Ahmed: 78,
};

marks["Hina"] = 88; // ✅ allowed — any string key maps to a number
```

---

## 🧾 Full Example

```ts
interface Address {
  city: string;
  country: string;
}

interface Student {
  readonly rollNo: number;
  name: string;
  age: number;
  address: Address;
  hobbies?: string[];
}

function printStudent(student: Student): void {
  console.log(`Roll #${student.rollNo}: ${student.name}, Age ${student.age}`);
  console.log(`Lives in ${student.address.city}, ${student.address.country}`);
  if (student.hobbies) {
    console.log(`Hobbies: ${student.hobbies.join(", ")}`);
  }
}

const student1: Student = {
  rollNo: 101,
  name: "Bilal",
  age: 20,
  address: { city: "Karachi", country: "Pakistan" },
  hobbies: ["cricket", "coding"],
};

printStudent(student1);
```

**Output:**

```
Roll #101: Bilal, Age 20
Lives in Karachi, Pakistan
Hobbies: cricket, coding
```

---

## 🧾 Summary

| Concept | Keyword | Example |
|---|---|---|
| Object shape (reusable) | `interface` | `interface User { name: string }` |
| Object shape / union / tuple | `type` | `type ID = string \| number` |
| Optional property | `?` | `age?: number` |
| Immutable property | `readonly` | `readonly id: number` |
| Extend an interface | `extends` | `interface Admin extends User` |
| Combine types | `&` | `type Post = User & Timestamped` |
| Dynamic keys | `[key: string]: T` | `interface Scores { [name: string]: number }` |

---

## 🧩 Hands-On Practice

1. Create an interface `Book` with `title`, `author`, `pages`, and an optional `isBestseller` field.
2. Make `isbn` a `readonly` property on `Book` and try to reassign it (observe the error).
3. Create a `type Coordinates = [number, number]` and a function `getDistance` that accepts two coordinates.
4. Create an interface `Employee extends Person` where `Person` has `name`/`age`, and `Employee` adds `salary` and `department`.
5. Create an index signature interface `Inventory` where keys are product names (`string`) and values are stock counts (`number`).

Next class: **Functions in TypeScript** — parameters, return types, optional/default/rest params, and overloads. 🚀
