# 🛠️ Utility Types, Mapped Types & Conditional Types

## 🌐 Overview

TypeScript ships with a set of **built-in utility types** that transform existing types into new ones — saving you from rewriting interfaces over and over. This class covers the utilities you'll use **every single day** in real projects, plus a look under the hood at how they're built (mapped & conditional types).

---

## 🎯 Learning Goals

* Master the most common built-in utility types
* Understand `keyof` and `typeof` operators
* Understand mapped types (how utilities are actually built)
* Understand conditional types (`T extends U ? X : Y`)
* Apply these patterns to real backend code (like this repo's Zod + Mongoose models)

---

## 🔑 `keyof` Operator

`keyof` extracts all the **property names** of a type as a union of string literals.

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

type UserKeys = keyof User; // "id" | "name" | "email"

function getValue(user: User, key: UserKeys) {
  return user[key];
}
```

---

## 🪞 `typeof` Operator (in type positions)

`typeof` lets you **reuse the shape of an existing variable/object** as a type.

```ts
const config = {
  port: 4000,
  dbUrl: "mongodb://localhost/shop",
};

type Config = typeof config;
// equivalent to: { port: number; dbUrl: string }
```

---

## ✅ Built-in Utility Types

### 1️⃣ `Partial<T>` — Makes All Properties Optional

Perfect for **update/edit** endpoints where you only send the fields that changed.

```ts
interface Product {
  title: string;
  price: number;
  stock: number;
}

function updateProduct(id: number, updates: Partial<Product>) {
  // updates can contain ANY subset of Product's fields
}

updateProduct(1, { price: 999 }); // ✅ only updating price
```

> 💡 This is exactly what `Backend/src/product/product.schema.ts` does with Zod: `updateProductSchema = createProductSchema.partial();`

### 2️⃣ `Required<T>` — Makes All Properties Mandatory (opposite of `Partial`)

```ts
interface Profile {
  name: string;
  bio?: string;
}

type CompleteProfile = Required<Profile>;
// { name: string; bio: string }  — bio is no longer optional
```

### 3️⃣ `Readonly<T>` — Makes All Properties Immutable

```ts
interface Settings {
  theme: string;
}

const settings: Readonly<Settings> = { theme: "dark" };
settings.theme = "light"; // ❌ Error: Cannot assign to 'theme' because it's read-only
```

### 4️⃣ `Pick<T, Keys>` — Select Only Specific Properties

```ts
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type PublicUser = Pick<User, "id" | "name" | "email">;
// { id: number; name: string; email: string }  — password excluded!
```

### 5️⃣ `Omit<T, Keys>` — Remove Specific Properties (opposite of `Pick`)

```ts
type UserWithoutPassword = Omit<User, "password">;
// { id: number; name: string; email: string }
```

> 💡 `Pick` and `Omit` are perfect for building **safe API responses** — this repo's `getAllUsers` controller does the same thing manually with `.select("-password -token -__v")` on the Mongoose query; `Omit` is the TypeScript-level equivalent.

### 6️⃣ `Record<Keys, ValueType>` — Build an Object Type with Specific Keys

```ts
type Role = "admin" | "editor" | "viewer";

type RolePermissions = Record<Role, string[]>;

const permissions: RolePermissions = {
  admin: ["create", "read", "update", "delete"],
  editor: ["create", "read", "update"],
  viewer: ["read"],
};
```

### 7️⃣ `Exclude<T, U>` and `Extract<T, U>` — Filter Union Types

```ts
type Status = "pending" | "paid" | "shipped" | "cancelled";

type ActiveStatus = Exclude<Status, "cancelled">;
// "pending" | "paid" | "shipped"

type FinalStatus = Extract<Status, "paid" | "cancelled">;
// "paid" | "cancelled"
```

### 8️⃣ `ReturnType<T>` — Extract a Function's Return Type

```ts
function createUser(name: string, age: number) {
  return { id: 1, name, age, createdAt: new Date() };
}

type NewUser = ReturnType<typeof createUser>;
// { id: number; name: string; age: number; createdAt: Date }
```

### 9️⃣ `Parameters<T>` — Extract a Function's Parameter Types as a Tuple

```ts
type CreateUserParams = Parameters<typeof createUser>;
// [name: string, age: number]
```

---

## 🧾 Quick Reference Table

| Utility | Purpose | Example |
|---|---|---|
| `Partial<T>` | All properties optional | Update/PATCH endpoints |
| `Required<T>` | All properties mandatory | Enforce completeness |
| `Readonly<T>` | All properties immutable | Config objects, constants |
| `Pick<T, K>` | Keep only selected keys | Public-safe DTOs |
| `Omit<T, K>` | Remove selected keys | Hide sensitive fields (passwords) |
| `Record<K, V>` | Build object type from keys+value type | Permission maps, lookup tables |
| `Exclude<T, U>` | Remove members from a union | Filter out a status |
| `Extract<T, U>` | Keep only matching members of a union | Narrow down allowed values |
| `ReturnType<T>` | Get a function's return type | Reuse inferred shapes |
| `Parameters<T>` | Get a function's parameter types | Wrapping/proxying functions |

---

## 🧬 Mapped Types (How These Utilities Are Actually Built)

A **mapped type** loops over the keys of an existing type to build a new one. This is literally how `Partial`, `Readonly`, etc. are implemented internally.

```ts
type MyPartial<T> = {
  [Key in keyof T]?: T[Key];
};

type MyReadonly<T> = {
  readonly [Key in keyof T]: T[Key];
};
```

### Custom Mapped Type Example

```ts
interface Product {
  title: string;
  price: number;
}

type Nullable<T> = {
  [Key in keyof T]: T[Key] | null;
};

type NullableProduct = Nullable<Product>;
// { title: string | null; price: number | null }
```

---

## 🔀 Conditional Types

A **conditional type** picks between two types based on a condition — like a ternary operator, but for types.

```ts
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>; // "yes"
type B = IsString<number>; // "no"
```

### Real Example: Extracting a Promise's Resolved Type

```ts
type Unwrap<T> = T extends Promise<infer U> ? U : T;

type A = Unwrap<Promise<string>>; // string
type B = Unwrap<number>;           // number (not a promise, so returned as-is)
```

`infer` lets TypeScript **capture** a type from within another type — this is how TypeScript's own built-in `Awaited<T>` and `ReturnType<T>` are implemented.

---

## 🧾 Full Example — Applying Utilities to a Real Backend Model

```ts
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  role: "user" | "admin";
}

// For creating a new user — no id yet (DB generates it)
type CreateUserInput = Omit<User, "id">;

// For sending user data back to the client — never expose password
type PublicUser = Omit<User, "password">;

// For updating a user — every field optional, but never allow changing the id
type UpdateUserInput = Partial<Omit<User, "id">>;

function createUser(input: CreateUserInput): User {
  return { id: Date.now(), ...input };
}

function toPublicUser(user: User): PublicUser {
  const { password, ...publicUser } = user;
  return publicUser;
}

const newUser = createUser({
  name: "Ali",
  email: "ali@mail.com",
  password: "secret123",
  role: "user",
});

console.log(toPublicUser(newUser));
// { id: ..., name: 'Ali', email: 'ali@mail.com', role: 'user' }
```

---

## 🧾 Summary

| Concept | Key Idea |
|---|---|
| `keyof` | Get a union of a type's property names |
| `typeof` | Reuse the type of an existing value |
| Utility types | Pre-built type transformations (`Partial`, `Pick`, `Omit`, etc.) |
| Mapped types | `{ [K in keyof T]: ... }` — loop over keys to build new types |
| Conditional types | `T extends U ? X : Y` — type-level if/else |
| `infer` | Capture/extract a type from within another type |

---

## 🧩 Hands-On Practice

1. Given `interface Order { id: number; total: number; status: string; }`, create `CreateOrderInput` (no `id`) and `UpdateOrderInput` (all optional, no `id`) using `Omit` and `Partial`.
2. Create a `PublicProfile` type from a `Profile` interface that excludes `password` and `ssn` using `Omit`.
3. Build a `Record<"admin" | "user", string>` mapping roles to a welcome message.
4. Write your own mapped type `MyReadonly<T>` from scratch (don't use the built-in `Readonly`).
5. Write a conditional type `ElementType<T>` that extracts the item type from an array type (hint: `T extends (infer U)[] ? U : T`).

Next week (Week 6): **Modules, tsconfig deep dive, and building a real TypeScript + Express project.** 🚀
