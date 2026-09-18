# 🧯 Centralized Error Handling & Logging (Winston + Morgan)

## 🧠 Introduction

Two things separate a hobby backend from a production-ready one: **consistent error handling** and **proper logging**. Without them, bugs are nearly impossible to trace once your app is live, and one unhandled crash can take down your entire server. This class covers both — using the exact tools listed in this course's tech stack (`Winston` and `Morgan`).

---

## 🎯 Learning Goals

* Understand why scattered `try/catch` blocks aren't enough
* Build a centralized Express error-handling middleware
* Create custom error classes for predictable, typed errors
* Handle async route errors without repetitive `try/catch`
* Set up request logging with **Morgan**
* Set up structured application logging with **Winston**
* Handle **uncaught exceptions** and **unhandled promise rejections**

---

## ⚠️ Step 1: The Problem With Repetitive `try/catch`

Every controller in this repo's `Backend/src/` follows this pattern:

```ts
export const getById = async (req: Request, res: Response) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ success: false, message: "User not found" });
    }
    return res.status(200).json({ success: true, data: user });
  } catch (error) {
    return res.status(500).json({ success: false, message: "Internal server error " + error });
  }
};
```

This works, but notice the `catch` block is **nearly identical in every single controller** — repeated dozens of times across `user`, `product`, `cart`, `order`, `payment`, and `address` controllers. That's a lot of duplicated code to maintain.

---

## 🧱 Step 2: Custom Error Classes

Instead of throwing generic `Error` objects, create a custom `AppError` class that carries an HTTP status code with it.

```ts
// src/utils/AppError.ts
export class AppError extends Error {
  statusCode: number;
  isOperational: boolean;

  constructor(message: string, statusCode: number) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true; // marks this as an expected, "safe" error
    Error.captureStackTrace(this, this.constructor);
  }
}
```

Now you can throw **meaningful, typed errors** anywhere in your code:

```ts
if (!user) {
  throw new AppError("User not found", 404);
}

if (!isPasswordValid) {
  throw new AppError("Invalid email or password", 401);
}
```

---

## 🌀 Step 3: An Async Handler Wrapper (Removing Repetitive `try/catch`)

Express doesn't automatically catch errors thrown inside `async` functions — an unhandled rejection can crash your server. A common solution is a wrapper function:

```ts
// src/utils/catchAsync.ts
import { Request, Response, NextFunction, RequestHandler } from "express";

export const catchAsync = (fn: RequestHandler) => {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next); // forwards errors to Express's error middleware
  };
};
```

Now your controllers become much shorter — no more `try/catch` in every function:

```ts
import { catchAsync } from "../utils/catchAsync";
import { AppError } from "../utils/AppError";

export const getById = catchAsync(async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    throw new AppError("User not found", 404);
  }
  res.status(200).json({ success: true, data: user });
});
```

Any thrown error (or rejected promise) is automatically caught and passed to `next(error)`, which forwards it straight to our centralized error-handling middleware.

---

## 🛡️ Step 4: The Centralized Error-Handling Middleware

Express recognizes a middleware as an **error handler** specifically because it has **4 parameters**: `(err, req, res, next)`. It must be registered **last**, after all your routes.

```ts
// src/middleware/errorHandler.ts
import { Request, Response, NextFunction } from "express";
import { AppError } from "../utils/AppError";
import { logger } from "../config/logger";

export function errorHandler(
  err: Error | AppError,
  req: Request,
  res: Response,
  next: NextFunction
) {
  const statusCode = err instanceof AppError ? err.statusCode : 500;
  const message = err.message || "Internal Server Error";

  logger.error(`${req.method} ${req.url} → ${statusCode}: ${message}`);

  res.status(statusCode).json({
    success: false,
    message,
    // Only expose the stack trace in development — never in production!
    ...(process.env.NODE_ENV === "development" && { stack: err.stack }),
  });
}
```

Register it in `index.ts`, **after** all routes:

```ts
import { errorHandler } from "./middleware/errorHandler";

app.use("/api", allAPIRoutes);

// Must be registered LAST
app.use(errorHandler);
```

### 🚏 Handling Unknown Routes (404 Fallback)

Add this **before** the error handler, but **after** all valid routes:

```ts
app.use((req: Request, res: Response) => {
  res.status(404).json({ success: false, message: `Route ${req.originalUrl} not found` });
});
```

---

## 📝 Step 5: Request Logging with Morgan

**Morgan** logs every incoming HTTP request — method, URL, status code, and response time — directly to your console. It's essential for seeing what's actually happening on your server in real time.

```bash
npm install morgan
npm install --save-dev @types/morgan
```

```ts
import morgan from "morgan";

// "dev" format — colored, concise, great for development
app.use(morgan("dev"));
```

**Example output:**

```
GET /api/products 200 12.345 ms - 348
POST /api/users/register 201 45.102 ms - 156
GET /api/products/123 404 3.221 ms - 42
```

### Common Morgan Formats

| Format | Description |
|---|---|
| `"dev"` | Concise, colored — best for development |
| `"combined"` | Apache-style detailed log — best for production |
| `"tiny"` | Minimal output |

---

## 🪵 Step 6: Structured Application Logging with Winston

While Morgan logs **HTTP requests**, **Winston** is a general-purpose logger for your **application's own events** — startup messages, database connection status, business logic warnings, and errors — with support for log **levels**, timestamps, and writing to files.

```bash
npm install winston
```

```ts
// src/config/logger.ts
import winston from "winston";

export const logger = winston.createLogger({
  level: "info",
  format: winston.format.combine(
    winston.format.timestamp({ format: "YYYY-MM-DD HH:mm:ss" }),
    winston.format.printf(({ timestamp, level, message }) => {
      return `[${timestamp}] ${level.toUpperCase()}: ${message}`;
    })
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: "logs/error.log", level: "error" }),
    new winston.transports.File({ filename: "logs/combined.log" }),
  ],
});
```

### Winston Log Levels (Highest to Lowest Priority)

```
error → warn → info → http → verbose → debug → silly
```

### Using the Logger

```ts
import { logger } from "./config/logger";

logger.info("Server started on port 4000");
logger.warn("Cache miss for product ID 42");
logger.error("Failed to connect to MongoDB");
```

**Example output:**

```
[2026-09-18 14:22:01] INFO: Server started on port 4000
[2026-09-18 14:22:05] WARN: Cache miss for product ID 42
[2026-09-18 14:22:10] ERROR: Failed to connect to MongoDB
```

> 💡 In `Backend/src/config/db.ts`, the `connectDB()` function currently uses `console.log`/`console.error`. In a production setup, you'd replace these with `logger.info(...)` and `logger.error(...)` so logs are timestamped, leveled, and saved to files for later inspection.

---

## 💥 Step 7: Handling Uncaught Exceptions & Unhandled Rejections

Even with try/catch everywhere, two categories of errors can still crash your entire Node.js process if left unhandled:

```ts
// Catches synchronous errors that were never wrapped in try/catch anywhere
process.on("uncaughtException", (error) => {
  logger.error(`Uncaught Exception: ${error.message}`);
  process.exit(1); // best practice: exit and let a process manager (PM2, Docker) restart the app
});

// Catches rejected Promises that were never caught with .catch() or try/catch
process.on("unhandledRejection", (reason) => {
  logger.error(`Unhandled Rejection: ${reason}`);
  process.exit(1);
});
```

> ⚠️ **Why exit instead of continuing?** After an uncaught exception, your application may be in an unpredictable, corrupted state (e.g. a half-completed database transaction). The safest move is to **log it, exit cleanly, and let a process manager restart the app fresh** — rather than risk silent data corruption.

---

## 🧾 Full Example — Putting It All Together

```ts
// src/index.ts
import express from "express";
import morgan from "morgan";
import { logger } from "./config/logger";
import { errorHandler } from "./middleware/errorHandler";
import allAPIRoutes from "./routes/routes";

const app = express();

app.use(express.json());
app.use(morgan("dev"));

app.use("/api", allAPIRoutes);

// 404 handler
app.use((req, res) => {
  res.status(404).json({ success: false, message: "Route not found" });
});

// Centralized error handler — always last
app.use(errorHandler);

process.on("uncaughtException", (error) => {
  logger.error(`Uncaught Exception: ${error.message}`);
  process.exit(1);
});

process.on("unhandledRejection", (reason) => {
  logger.error(`Unhandled Rejection: ${reason}`);
  process.exit(1);
});

app.listen(4000, () => logger.info("Server running on port 4000"));
```

---

## 🧾 Summary

| Tool/Pattern | Purpose |
|---|---|
| `AppError` class | Typed, predictable errors carrying an HTTP status code |
| `catchAsync` wrapper | Eliminates repetitive `try/catch` in every controller |
| Centralized error middleware `(err, req, res, next)` | One place to format and log every error |
| 404 fallback middleware | Catches requests to routes that don't exist |
| **Morgan** | Logs every HTTP request (method, URL, status, timing) |
| **Winston** | Structured application logging with levels, timestamps, and file output |
| `process.on("uncaughtException")` | Safety net for synchronous errors that slip through |
| `process.on("unhandledRejection")` | Safety net for rejected Promises that were never caught |

---

## 🧩 Hands-On Practice

1. Create an `AppError` class and a `catchAsync` wrapper, then refactor one controller from this repo's `Backend/src/product/product.controllers.ts` to use both.
2. Add a centralized error-handling middleware and a 404 fallback to a small Express app.
3. Install and configure Morgan with the `"dev"` format; make a few requests and observe the console output.
4. Install and configure Winston to log to both the console and a `logs/error.log` file; trigger an error and confirm it's written to the file.
5. Add `uncaughtException`/`unhandledRejection` handlers, then intentionally throw an error outside of any try/catch to confirm they catch it.

Next class: **Redis Caching Strategies.** 🚀
