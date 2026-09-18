# 🧩 Node.js Core Fundamentals — Modules, Built-in APIs & Asynchronous Programming

## 🧠 Introduction

Before diving deeper into Express and MongoDB, every backend developer needs a solid grip on **Node.js itself** — how modules work, the built-in modules you'll use constantly, and the three generations of asynchronous programming (**callbacks → Promises → async/await**). This class fills that foundation.

---

## 🎯 Learning Goals

* Understand the CommonJS module system (`require`/`module.exports`) and how it differs from ES Modules (`import`/`export`)
* Use Node's essential built-in modules: `fs`, `path`, `os`, `events`
* Understand `process`, `__dirname`, and `__filename`
* Master the evolution of async code: callbacks → Promises → async/await
* Understand `package.json` and npm dependency management in depth

---

## 📦 Step 1: The Module System

Every file in Node.js is treated as its own **module** — variables and functions declared in one file are **not** automatically available in another. You must explicitly `export` and `import`/`require` them.

### CommonJS (Node's traditional/default system)

```js
// math.js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

module.exports = { add, subtract };
```

```js
// index.js
const { add, subtract } = require("./math");

console.log(add(5, 3)); // 8
```

### ES Modules (modern JavaScript standard)

To use `import`/`export` syntax directly in Node.js (without TypeScript), add `"type": "module"` to `package.json`:

```json
{
  "type": "module"
}
```

```js
// math.mjs (or math.js, with "type": "module" set)
export function add(a, b) {
  return a + b;
}
```

```js
import { add } from "./math.js";
```

### CommonJS vs ES Modules

| | CommonJS | ES Modules |
|---|---|---|
| Syntax | `require()` / `module.exports` | `import` / `export` |
| Loading | Synchronous | Can be asynchronous |
| Default in Node.js | ✅ Yes (`.js` files by default) | ❌ Needs `"type": "module"` or `.mjs` |
| Used by | Older Node.js code, most npm packages | Modern JS, browsers, TypeScript (compiles to either) |

> 💡 In this repo's `Backend/` folder, you write `import`/`export` syntax in `.ts` files, but `tsconfig.json` sets `"module": "commonjs"` — so TypeScript **compiles your modern syntax down to CommonJS** for Node.js to run. You get to write modern syntax either way!

---

## 🗂️ Step 2: Essential Built-in Modules

Node.js ships with many modules you never need to install — just `require`/`import` them.

### 📁 `fs` (File System)

Read, write, and manage files.

```js
const fs = require("fs");

// Synchronous (blocks the event loop — avoid in servers!)
const data = fs.readFileSync("notes.txt", "utf-8");

// Asynchronous with callback (non-blocking)
fs.readFile("notes.txt", "utf-8", (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Asynchronous with Promises (modern, preferred)
const fsPromises = require("fs/promises");

async function readNotes() {
  const data = await fsPromises.readFile("notes.txt", "utf-8");
  console.log(data);
}
```

> ⚠️ **Never use the `Sync` versions (`readFileSync`, `writeFileSync`) inside an Express route handler** — they block the single event loop thread for every connected user. See the [Event Loop guide](../event_loop/README.md) for why.

### 🛣️ `path` (File & Directory Paths)

Handles file paths correctly across operating systems (Windows uses `\`, Linux/Mac use `/`).

```js
const path = require("path");

path.join("uploads", "images", "photo.png");
// → "uploads/images/photo.png" (or "uploads\images\photo.png" on Windows)

path.extname("photo.png");   // ".png"
path.basename("/uploads/photo.png"); // "photo.png"
path.dirname("/uploads/photo.png");  // "/uploads"
path.resolve("uploads", "photo.png");
// → absolute path, e.g. "/home/user/project/uploads/photo.png"
```

> 💡 This repo's own upload code (`Backend/src/helpers/upload.ts`) relies on correct path handling for storing uploaded files.

### 🖥️ `os` (Operating System Info)

```js
const os = require("os");

os.platform();     // "win32", "linux", "darwin"
os.cpus().length;   // number of CPU cores (useful for clustering)
os.totalmem();       // total system memory in bytes
os.freemem();         // free system memory in bytes
```

### 📡 `events` (EventEmitter — The Foundation of Node.js)

Node.js's asynchronous, event-driven core is built on the **EventEmitter** pattern. Many built-in objects (HTTP servers, streams) are EventEmitters under the hood.

```js
const EventEmitter = require("events");

const orderEmitter = new EventEmitter();

// Listen for an event
orderEmitter.on("orderPlaced", (orderId) => {
  console.log(`📦 Order ${orderId} placed! Sending confirmation email...`);
});

orderEmitter.on("orderPlaced", (orderId) => {
  console.log(`📊 Logging order ${orderId} for analytics...`);
});

// Trigger the event — both listeners above run
orderEmitter.emit("orderPlaced", 101);
```

**Output:**

```
📦 Order 101 placed! Sending confirmation email...
📊 Logging order 101 for analytics...
```

> 💡 This pattern (multiple independent listeners reacting to one event) is how you decouple side effects — e.g., "when an order is placed, send an email AND update inventory AND log analytics" — without cramming all that logic into one function.

### 🌍 Global Objects: `process`, `__dirname`, `__filename`

```js
console.log(__dirname);  // absolute path to the current file's directory
console.log(__filename); // absolute path to the current file itself

console.log(process.env.NODE_ENV);   // reads an environment variable
console.log(process.argv);            // command-line arguments passed to the script
console.log(process.platform);         // "win32", "linux", "darwin"

process.exit(1); // forcefully stops the Node.js process (1 = error exit code)
```

`process.env` is how **environment variables** (like database URLs and secrets) are accessed — this is exactly how `Backend/src/config/db.ts` reads `process.env.MongoDB_URI` in this repository.

---

## ⏳ Step 3: The Evolution of Asynchronous Code

Node.js is asynchronous by nature (see the [Event Loop guide](../event_loop/README.md)). Over the years, JavaScript introduced three patterns to *write* async code — you should recognize and understand all three, since real codebases mix them.

### 1️⃣ Callbacks (The Original Pattern)

A function passed as an argument, called once an operation finishes.

```js
fs.readFile("data.txt", "utf-8", (err, data) => {
  if (err) {
    console.error("Error:", err);
    return;
  }
  console.log(data);
});
```

**Problem: "Callback Hell"** — nesting multiple async steps becomes unreadable:

```js
getUser(userId, (err, user) => {
  getOrders(user.id, (err, orders) => {
    getOrderDetails(orders[0].id, (err, details) => {
      sendEmail(user.email, details, (err, result) => {
        console.log("Finally done!"); // deeply nested, hard to read/maintain
      });
    });
  });
});
```

### 2️⃣ Promises (The Fix for Callback Hell)

A **Promise** represents a value that will be available **eventually** — either resolved (success) or rejected (failure).

```js
function getUser(userId) {
  return new Promise((resolve, reject) => {
    if (!userId) {
      reject(new Error("User ID is required"));
    } else {
      resolve({ id: userId, name: "Ali" });
    }
  });
}

getUser(1)
  .then((user) => console.log(user))
  .catch((error) => console.error(error));
```

**Chaining fixes the nesting problem:**

```js
getUser(userId)
  .then((user) => getOrders(user.id))
  .then((orders) => getOrderDetails(orders[0].id))
  .then((details) => sendEmail(details))
  .then(() => console.log("Finally done!"))
  .catch((error) => console.error("Something failed:", error));
```

### 3️⃣ Async/Await (Modern, Cleanest Syntax)

`async`/`await` is **syntactic sugar over Promises** — it lets asynchronous code read like synchronous code.

```js
async function processOrder(userId) {
  try {
    const user = await getUser(userId);
    const orders = await getOrders(user.id);
    const details = await getOrderDetails(orders[0].id);
    await sendEmail(details);
    console.log("Finally done!");
  } catch (error) {
    console.error("Something failed:", error);
  }
}
```

> 💡 This is exactly the pattern used throughout this repo's `Backend/src/*/**.controllers.ts` files — every controller is an `async` function using `try/catch` and `await`, e.g. `Backend/src/user/user.controllers.ts`.

### 🔁 Running Promises in Parallel

If async operations **don't depend on each other**, run them **together** instead of one-by-one for much better performance.

```js
// ❌ Slow — runs one at a time (sequential)
const user = await getUser(1);
const products = await getProducts();
const orders = await getOrders(1);

// ✅ Fast — runs all three simultaneously
const [user, products, orders] = await Promise.all([
  getUser(1),
  getProducts(),
  getOrders(1),
]);
```

| Method | Behavior |
|---|---|
| `Promise.all([...])` | Waits for **all** to succeed; rejects immediately if **any one** fails |
| `Promise.allSettled([...])` | Waits for **all** to finish (success or failure), never rejects |
| `Promise.race([...])` | Resolves/rejects as soon as the **first** promise settles |

---

## 📦 Step 4: npm & `package.json` In Depth

### What is npm?

**npm (Node Package Manager)** is the tool used to install, manage, and share reusable JavaScript/TypeScript packages.

```bash
npm install express         # add to "dependencies" (needed in production)
npm install --save-dev nodemon  # add to "devDependencies" (only needed for development)
npm install -g typescript    # install globally, available as a CLI command anywhere
npm uninstall express          # remove a package
npm update                      # update packages to their latest allowed version
```

### Anatomy of `package.json`

```json
{
  "name": "backend",
  "version": "1.0.0",
  "main": "index.ts",
  "scripts": {
    "dev": "nodemon",
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "express": "^5.2.1",
    "mongoose": "^9.1.3"
  },
  "devDependencies": {
    "typescript": "^5.9.3",
    "nodemon": "^3.1.11"
  }
}
```

| Field | Meaning |
|---|---|
| `dependencies` | Packages needed to **run** the app in production |
| `devDependencies` | Packages only needed **while developing** (compilers, test tools, linters) |
| `scripts` | Custom commands runnable via `npm run <name>` |

### Understanding Semantic Versioning (SemVer)

```
^5.2.1
│ │ │ └── PATCH — bug fixes, no new features
│ │ └──── MINOR — new features, backward-compatible
│ └────── MAJOR — breaking changes
```

| Symbol | Meaning | Example | Allows |
|---|---|---|---|
| `^5.2.1` | Compatible with | Accepts `5.x.x`, but not `6.0.0` | Minor + patch updates |
| `~5.2.1` | Approximately | Accepts `5.2.x` only | Patch updates only |
| `5.2.1` (no symbol) | Exact version | Only `5.2.1` | No updates at all |

### `package-lock.json`

This file **locks the exact version** of every package (including nested dependencies) that was actually installed. It ensures that **every machine and every deployment** installs the exact same dependency tree — critical for avoiding "works on my machine" bugs. **Always commit this file to git.**

---

## 🧾 Summary

| Concept | Key Takeaway |
|---|---|
| CommonJS | `require()` / `module.exports` — Node's default module system |
| ES Modules | `import` / `export` — modern standard, needs `"type": "module"` or TypeScript |
| `fs` | File system access — always prefer async/Promise versions in servers |
| `path` | Cross-platform-safe file path handling |
| `os` | Read info about the operating system/hardware |
| `events` (EventEmitter) | The event-driven pattern underlying Node.js itself |
| `process.env` | How environment variables/secrets are read |
| Callbacks → Promises → async/await | The evolution of asynchronous code — know all three |
| `Promise.all()` | Run independent async operations in parallel for speed |
| `package.json` | Declares dependencies, scripts, and project metadata |
| SemVer (`^`, `~`) | Controls which package updates npm is allowed to install |

---

## 🧩 Hands-On Practice

1. Create a `math.js` module using CommonJS (`module.exports`) with `add`/`subtract`/`multiply` functions, and import it into another file.
2. Write a script using `fs.promises` to read a `.txt` file and log its word count.
3. Create a custom `EventEmitter` that emits a `"userRegistered"` event, with two separate listeners: one that logs a welcome message, and one that simulates "sending an email."
4. Rewrite a callback-based function (using `setTimeout` to simulate delay) into: (a) a Promise-based version, then (b) an `async/await` version.
5. Write three functions that each resolve after a random delay, and use `Promise.all()` to run them in parallel — time how much faster this is than running them sequentially with `await`.

Next class (Week 2): **HTTP Status Codes & RESTful APIs.** 🚀
