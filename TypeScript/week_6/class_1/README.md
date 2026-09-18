# 📦 Modules, Declaration Files & tsconfig Deep Dive

## 🌐 Overview

You've been using `import`/`export` since Week 1 without a deep explanation. This class covers how **modules** really work in TypeScript, how to add types for JavaScript libraries that don't ship their own (`.d.ts` files), and a deeper look at the `tsconfig.json` options that matter most in real projects.

---

## 🎯 Learning Goals

* Understand ES Modules (`import`/`export`) vs CommonJS (`require`/`module.exports`)
* Use named exports vs default exports
* Understand declaration files (`.d.ts`) and `@types` packages
* Understand key `tsconfig.json` compiler options in depth
* Understand path aliases

---

## 📦 ES Modules — Named Exports

```ts
// mathUtils.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export const PI = 3.14159;
```

```ts
// index.ts
import { add, subtract, PI } from "./mathUtils";

console.log(add(2, 3));      // 5
console.log(PI);              // 3.14159
```

You can also rename during import:

```ts
import { add as sum } from "./mathUtils";
```

---

## 📦 Default Exports

Each file can have **one** default export — commonly used for the "main thing" a file provides.

```ts
// userModel.ts
interface User {
  id: number;
  name: string;
}

export default User;
```

```ts
// index.ts
import User from "./userModel"; // no curly braces needed for default imports
```

> 💡 **Real example from this repo:** `Backend/src/user/user.model.ts` does `export default mongoose.model<UserModel>("User", userSchema);`, and other files import it with `import User from "./user.model";`.

### Named vs Default — When to Use Which

| | Named Export | Default Export |
|---|---|---|
| Count per file | Many | Only one |
| Import syntax | `import { x } from "./file"` | `import x from "./file"` |
| Renaming on import | Must use `as` | Automatic (any name works) |
| Common use | Utility functions, multiple related items | The main class/model/component of a file |

---

## 🔄 CommonJS vs ES Modules

Node.js traditionally used **CommonJS**:

```js
// CommonJS
const express = require("express");
module.exports = router;
```

Modern TypeScript/Node projects prefer **ES Modules** syntax:

```ts
// ES Modules
import express from "express";
export default router;
```

The `"module"` option in `tsconfig.json` controls which one your compiled output uses. This repo's `Backend/tsconfig.json` targets `commonjs` (Node's traditional format), even though the source is written with `import`/`export` syntax — TypeScript compiles the modern syntax down to CommonJS automatically.

---

## 📄 Declaration Files (`.d.ts`)

TypeScript needs to know the **types** of everything you import — including plain JavaScript libraries that were never written in TypeScript.

* `.ts` files → contain actual code + types
* `.d.ts` files → contain **only type declarations**, no actual logic — they just describe the shape of existing JavaScript

### Where do `.d.ts` files come from?

1. **Bundled with the library itself** (e.g. modern packages like `zod`, `mongoose` ship their own types)
2. **Separately from the `@types` organization** on npm (community-maintained), e.g.:

```bash
npm install --save-dev @types/express
npm install --save-dev @types/node
npm install --save-dev @types/bcrypt
```

> 💡 **This is exactly what you see in this repo's `Backend/package.json`** — devDependencies like `@types/cors`, `@types/jsonwebtoken`, `@types/multer` exist because those libraries (`cors`, `jsonwebtoken`, `multer`) are written in plain JavaScript and don't include their own types.

### Writing Your Own Declaration File

If a library has **no types at all** (not even on `@types`), you can declare it yourself:

```ts
// types/my-untyped-lib.d.ts
declare module "my-untyped-lib" {
  export function doSomething(value: string): void;
}
```

### Extending Express's `Request` Type (Common Real-World Need)

This repo's `Backend/src/helpers/auth.ts` does `req.user = decoded;` — but Express's built-in `Request` type doesn't have a `user` property! It works because somewhere a declaration file extends it:

```ts
// types/express.d.ts
import "express";

declare global {
  namespace Express {
    interface Request {
      user?: any;
    }
  }
}
```

This is called **module augmentation** — adding new properties to an existing library's types.

---

## ⚙️ tsconfig.json — Deep Dive on Key Options

We touched on the basics in Week 1. Here's a deeper look at options that matter in real projects:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "rootDir": "src",
    "outDir": "dist",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

| Option | What It Does |
|---|---|
| `resolveJsonModule` | Lets you `import data from "./data.json"` directly |
| `moduleResolution: "node"` | Uses Node's normal module lookup rules (checks `node_modules`, etc.) |
| `baseUrl` + `paths` | Enables **path aliases** (see below) |
| `noUnusedLocals` | Errors on variables you declared but never used |
| `noUnusedParameters` | Errors on function parameters you never used |
| `noImplicitReturns` | Ensures every code path in a function explicitly returns a value |
| `forceConsistentCasingInFileNames` | Prevents import bugs from mismatched file name casing (`User.ts` vs `user.ts`) — critical on Windows/macOS which are case-insensitive but Linux servers are not! |
| `include` / `exclude` | Which files/folders to compile |

---

## 🧭 Path Aliases (Cleaner Imports)

Without aliases, deeply nested files end up with messy relative imports:

```ts
import { connectDB } from "../../../config/db";
```

With a path alias configured in `tsconfig.json`:

```json
"baseUrl": ".",
"paths": {
  "@/*": ["src/*"]
}
```

You can write instead:

```ts
import { connectDB } from "@/config/db";
```

> ⚠️ **Note:** `tsconfig.json` path aliases only affect **type-checking**, not the actual compiled JavaScript. To make aliases work at runtime with plain `tsc`/Node, you also need a tool like `tsc-alias` or a bundler (esbuild, Vite, webpack) that understands them. Frameworks like Next.js (used in this repo's `Frontend/`) handle this automatically — that's how `@/components/Header` works in `Frontend/app/page.tsx`.

---

## 🧾 Full Example — Small Multi-File Project

```ts
// src/types/user.ts
export interface User {
  id: number;
  name: string;
  email: string;
}
```

```ts
// src/services/userService.ts
import { User } from "../types/user";

const users: User[] = [];

export function addUser(user: User): void {
  users.push(user);
}

export function getAllUsers(): User[] {
  return users;
}

export default users;
```

```ts
// src/index.ts
import { addUser, getAllUsers } from "./services/userService";

addUser({ id: 1, name: "Ali", email: "ali@mail.com" });
addUser({ id: 2, name: "Sara", email: "sara@mail.com" });

console.log(getAllUsers());
```

---

## 🧾 Summary

| Concept | Key Point |
|---|---|
| Named export/import | `export function x` / `import { x }` — multiple per file |
| Default export/import | `export default x` / `import x` — one per file |
| `.d.ts` files | Type-only files describing a JS library's shape |
| `@types/*` packages | Community types for JS libraries without built-in types |
| Module augmentation | Extending an existing library's types (e.g. adding `req.user`) |
| `strict` | Always keep enabled — the core value of TypeScript |
| Path aliases | Cleaner imports via `baseUrl` + `paths` (needs extra tooling for runtime) |

---

## 🧩 Hands-On Practice

1. Split a small project into 3 files: `types.ts` (interfaces), `utils.ts` (functions), `index.ts` (entry point). Use named exports/imports throughout.
2. Convert one function to a default export and update its import accordingly.
3. Install `@types/node` in a fresh project and inspect what it adds (try using `process.env` without it first, and observe the error).
4. Write a tiny `.d.ts` file that declares types for a fictional untyped npm package.
5. Configure a path alias (`@/*` → `src/*`) in a new project's `tsconfig.json` and use it in an import.

Next class (final class!): **TypeScript with Express.js — a real practical mini project + best practices.** 🚀
