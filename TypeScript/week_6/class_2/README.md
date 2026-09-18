# 🚀 TypeScript with Express.js — Practical Mini Project & Best Practices

## 🌐 Overview

This is our **final class**! We'll bring together everything you've learned — types, interfaces, generics, utility types, classes — and apply it to build a small, real, typed **Express.js REST API**. We'll close with a set of **best practices** used by professional TypeScript backend teams (and followed throughout this repo's own `Backend/` folder).

---

## 🎯 Learning Goals

* Type Express `Request`, `Response`, and route params/body/query
* Build a small typed REST API end-to-end
* Apply validation with Zod (as used in this repo)
* Organize a scalable folder structure
* Learn professional TypeScript best practices

---

## 🧰 Step 1: Project Setup

```bash
mkdir ts-express-api && cd ts-express-api
npm init -y
npm install express
npm install --save-dev typescript ts-node-dev @types/node @types/express
npx tsc --init
```

`tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "rootDir": "src",
    "outDir": "dist",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

`package.json` scripts:

```json
"scripts": {
  "dev": "ts-node-dev --respawn src/index.ts",
  "build": "tsc",
  "start": "node dist/index.js"
}
```

---

## 🧱 Step 2: Basic Typed Express Server

```ts
// src/index.ts
import express, { Request, Response } from "express";

const app = express();
app.use(express.json());

app.get("/", (req: Request, res: Response) => {
  res.send("API is running 🚀");
});

const PORT = 4000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

* `Request` and `Response` come from `@types/express` — they give you full autocomplete for `req.body`, `req.params`, `res.json()`, etc.

---

## 🎯 Step 3: Typing Route Params, Query, and Body

Express's `Request` type accepts **4 generic parameters**: `Request<Params, ResBody, ReqBody, ReqQuery>`.

```ts
import { Request, Response } from "express";

interface CreateProductBody {
  title: string;
  price: number;
}

interface ProductParams {
  id: string;
}

app.post(
  "/products",
  (req: Request<{}, {}, CreateProductBody>, res: Response) => {
    const { title, price } = req.body; // fully typed — no "any"!
    res.status(201).json({ title, price });
  }
);

app.get(
  "/products/:id",
  (req: Request<ProductParams>, res: Response) => {
    const { id } = req.params; // typed as string
    res.json({ id });
  }
);
```

> 💡 **This is exactly the pattern used throughout this repo.** Compare with `Backend/src/product/product.controllers.ts`:
> ```ts
> export const getProductById = async (
>   req: Request<{ id: string }>,
>   res: Response
> ) => { ... };
> ```

---

## 🧩 Step 4: Defining a Model & In-Memory "Database"

```ts
// src/models/product.model.ts
export interface Product {
  id: number;
  title: string;
  price: number;
  stock: number;
}

export const products: Product[] = [];
```

---

## ✅ Step 5: Validation with Zod (Same Library This Repo Uses)

Runtime validation is critical — TypeScript types **only exist at compile time** and vanish once your code runs. A malicious or buggy client can still send bad data over HTTP. That's why this repo's real backend pairs TypeScript with **Zod** for runtime validation.

```bash
npm install zod
```

```ts
// src/product/product.schema.ts
import { z } from "zod";

export const createProductSchema = z.object({
  title: z.string().min(1),
  price: z.number().min(0),
  stock: z.number().min(0),
});

export type CreateProductInput = z.infer<typeof createProductSchema>;
```

`z.infer<typeof schema>` **automatically generates the TypeScript type** from your validation schema — you define validation rules ONCE, and get both runtime checks AND compile-time types for free.

---

## 🧾 Step 6: Full Controller with Validation

```ts
// src/product/product.controller.ts
import { Request, Response } from "express";
import { createProductSchema, CreateProductInput } from "./product.schema";
import { products, Product } from "../models/product.model";

let nextId = 1;

export const createProduct = (
  req: Request<{}, {}, CreateProductInput>,
  res: Response
) => {
  const parsed = createProductSchema.safeParse(req.body);

  if (!parsed.success) {
    return res.status(400).json({
      success: false,
      errors: parsed.error.issues.map((issue) => issue.message),
    });
  }

  const newProduct: Product = { id: nextId++, ...parsed.data };
  products.push(newProduct);

  return res.status(201).json({ success: true, data: newProduct });
};

export const getAllProducts = (req: Request, res: Response) => {
  return res.status(200).json({ success: true, data: products });
};

export const getProductById = (
  req: Request<{ id: string }>,
  res: Response
) => {
  const product = products.find((p) => p.id === Number(req.params.id));

  if (!product) {
    return res.status(404).json({ success: false, message: "Product not found" });
  }

  return res.status(200).json({ success: true, data: product });
};
```

---

## 🛣️ Step 7: Routes

```ts
// src/product/product.routes.ts
import { Router } from "express";
import { createProduct, getAllProducts, getProductById } from "./product.controller";

const router = Router();

router.post("/", createProduct);
router.get("/", getAllProducts);
router.get("/:id", getProductById);

export default router;
```

```ts
// src/index.ts
import express from "express";
import productRoutes from "./product/product.routes";

const app = express();
app.use(express.json());
app.use("/api/products", productRoutes);

app.listen(4000, () => console.log("Server running on port 4000"));
```

---

## 🧩 Step 8: Typed Middleware (e.g. Auth Guard)

```ts
import { Request, Response, NextFunction } from "express";

export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.split(" ")[1];

  if (!token) {
    return res.status(401).json({ success: false, message: "Token not found" });
  }

  // in real apps, verify the JWT here (see this repo's helpers/auth.ts)
  next();
}
```

---

## 📂 Recommended Scalable Folder Structure

This mirrors exactly how this repo's own `Backend/src/` is organized, feature-by-feature (not type-by-type):

```
src/
├── product/
│   ├── product.controller.ts
│   ├── product.model.ts
│   ├── product.routes.ts
│   └── product.schema.ts
├── user/
│   ├── user.controller.ts
│   ├── user.model.ts
│   ├── user.routes.ts
│   └── user.schema.ts
├── helpers/
│   ├── auth.ts
│   └── jwt.ts
├── config/
│   └── db.ts
├── routes/
│   └── routes.ts      👈 combines all feature routers
└── index.ts             👈 app entry point
```

> 💡 This "feature folder" pattern (`user/`, `product/`, `cart/`, `order/`...) scales much better than grouping by file type (`controllers/`, `models/`, `routes/` folders each containing everything) because everything related to one feature lives together.

---

## 🏆 Best Practices Checklist

| ✅ Practice | Why |
|---|---|
| Always keep `"strict": true` | Catches the most bugs; the entire point of TypeScript |
| Never use `any` — prefer `unknown` + narrowing | Keeps type safety intact |
| Validate all external input at runtime (Zod, Joi, etc.) | Types disappear at runtime — bad data can still arrive |
| Use interfaces/types for every function's params & return | Self-documenting, safer refactors |
| Use `Partial<T>`/`Omit<T>`/`Pick<T>` for DTOs instead of duplicating interfaces | DRY — one source of truth per model |
| Organize code by **feature**, not by file type | Easier to navigate as the app grows |
| Never expose passwords/tokens in API responses | Use `Omit<User, "password">` or `.select("-password")` |
| Keep environment secrets in `.env`, never hardcoded | Security |
| Use generic `ApiResponse<T>` for consistent API shapes | Predictable frontend integration |
| Compile with `tsc --noEmit` in CI before deploying | Catches type errors before they reach production |

---

## 🧾 Course Recap

| Week | Topic |
|---|---|
| 1 | Setup, basic types, type annotations |
| 2 | Objects, interfaces, type aliases, functions |
| 3 | Unions, intersections, literal types, narrowing, enums, tuples |
| 4 | Classes, inheritance, abstract classes, interfaces with classes |
| 5 | Generics, utility types, mapped & conditional types |
| 6 | Modules, declaration files, tsconfig, real Express + Zod project |

🎉 **Congratulations!** You've gone from "What is TypeScript?" to building a fully-typed, validated Express API — the exact same stack used in this repository's `Backend/` folder.

---

## 🧩 Final Hands-On Project

Build a small **typed Task Manager API** using everything from this course:

1. `Task` interface: `id`, `title`, `isCompleted`, `priority: "low" | "medium" | "high"`.
2. Zod schema for creating a task (`title` required, `priority` optional, default `"medium"`).
3. Routes: `POST /tasks`, `GET /tasks`, `GET /tasks/:id`, `PATCH /tasks/:id` (using `Partial<Task>`), `DELETE /tasks/:id`.
4. A generic `ApiResponse<T>` interface used consistently across every response.
5. A `TaskRepository` **generic class** (like Week 5's `Repository<T>`) to store tasks in memory.
6. Bonus: Add an `authMiddleware` that checks for a fake `Authorization` header before allowing `POST`/`DELETE`.
