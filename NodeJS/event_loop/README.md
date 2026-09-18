# 🔄 The Node.js Event Loop — Deep Dive

## 🧠 Introduction

Node.js is famous for being **"single-threaded, non-blocking, and asynchronous."** But what does that actually mean, and how does a single thread handle thousands of simultaneous connections without freezing? The answer is the **Event Loop** — the single most important concept to truly understand backend performance in Node.js.

This guide explains the event loop **from the ground up**, with diagrams, phase-by-phase breakdowns, and real code examples whose output order might surprise you (until you understand *why*).

---

## ⚙️ Step 1: Node.js Is Single-Threaded — But Not Alone

JavaScript itself runs on a **single thread** — one line of code executes at a time, in one call stack. This is true both in the browser and in Node.js.

But Node.js is **not just the V8 JavaScript engine**. It's built on top of several components working together:

| Component | Responsibility |
|---|---|
| **V8 Engine** | Compiles and executes your JavaScript code (also used by Chrome) |
| **libuv** | A C++ library that provides the **event loop**, a **thread pool**, and handles async I/O (file system, DNS, some crypto operations) |
| **Node.js APIs** | Bindings that expose OS-level features (`fs`, `net`, `http`, timers) to JavaScript |

> 💡 **Key insight:** Your JavaScript code runs on ONE thread. But slow operations (reading a file, querying a database, making an HTTP request) are handed off to **libuv**, which uses the operating system's async capabilities (and a background **thread pool** for things the OS can't do async, like some file system calls) — freeing the main thread to keep running other code.

<img src="./eventLoop.PNG" alt="Node.js Event Loop Architecture">

---

## 🧩 Step 2: The Call Stack

The **call stack** is where your JavaScript code actually executes, one function at a time, in a **LIFO** (Last In, First Out) order.

```js
function greet() {
  console.log("Hello");
}

function main() {
  greet();
}

main();
```

Execution order on the call stack:

```
1. main() is pushed onto the stack
2. greet() is pushed onto the stack (called inside main)
3. console.log("Hello") runs
4. greet() is popped off the stack (finished)
5. main() is popped off the stack (finished)
```

If a function takes a long time to run **synchronously**, it **blocks the call stack** — nothing else can run until it finishes. This is why a single huge `for` loop, or a synchronous file read, can freeze your entire Node.js server for every single user.

```js
// ❌ Blocks everything for ~3 seconds — no other request can be handled!
function blockEventLoop() {
  const start = Date.now();
  while (Date.now() - start < 3000) {} // busy-wait loop
}
```

---

## 📬 Step 3: Where Do Async Operations Go?

When you call an asynchronous function — `setTimeout`, `fs.readFile`, an HTTP request, a database query — Node.js does **not** run it directly on the call stack. Instead:

1. The async operation is **handed off** to libuv (or the OS/thread pool)
2. Your JavaScript code **keeps running** — it doesn't wait
3. When the operation **completes**, its callback function is placed into a **queue**
4. The **Event Loop** constantly checks: *"Is the call stack empty? If yes, take the next callback from the queue and push it onto the stack."*

```
┌───────────────────────────┐
│         Call Stack         │  ← your code executes here, one thing at a time
└──────────────┬──────────────┘
               │  (empty? → pull from queue)
               ▼
┌───────────────────────────┐
│         Event Loop         │  ← constantly checking: is the stack empty?
└──────────────┬──────────────┘
               │
    ┌──────────┴──────────┐
    ▼                      ▼
Macrotask Queue      Microtask Queue
(setTimeout,          (Promises,
 setInterval, I/O)     process.nextTick)
```

---

## 🌀 Step 4: The Event Loop's 6 Phases

The event loop doesn't just run in one giant loop — it cycles through **distinct phases**, each with its own callback queue. One full cycle through all phases is called a **"tick"**.

```
   ┌───────────────────────┐
┌─>│        timers         │  → setTimeout(), setInterval() callbacks
│  └──────────┬────────────┘
│  ┌──────────▼────────────┐
│  │   pending callbacks    │  → I/O callbacks deferred from previous cycle
│  └──────────┬────────────┘
│  ┌──────────▼────────────┐
│  │     idle, prepare      │  → internal use only
│  └──────────┬────────────┘
│  ┌──────────▼────────────┐
│  │          poll          │  → fetch new I/O events (file reads, network); executes their callbacks
│  └──────────┬────────────┘
│  ┌──────────▼────────────┐
│  │          check         │  → setImmediate() callbacks run here
│  └──────────┬────────────┘
│  ┌──────────▼────────────┐
└──┤     close callbacks    │  → e.g. socket.on('close', ...)
   └────────────────────────┘
```

| Phase | What Runs Here |
|---|---|
| **timers** | Callbacks scheduled by `setTimeout()` and `setInterval()` whose time has elapsed |
| **pending callbacks** | Certain system-level callbacks deferred from the previous loop iteration |
| **idle, prepare** | Used internally by Node.js — not something you interact with directly |
| **poll** | Retrieves new I/O events (reads a file, gets a network response) and runs their callbacks; if nothing is scheduled, it may wait here |
| **check** | Runs `setImmediate()` callbacks, always right after the poll phase |
| **close callbacks** | Runs cleanup callbacks, e.g. `socket.on('close', callback)` |

---

## ⚡ Step 5: Microtasks — They Jump the Queue!

Not all async callbacks wait for their phase. **Microtasks** run **immediately after the current operation finishes**, **before the event loop moves to the next phase** — even before timers or I/O callbacks that were "ready" earlier.

There are two microtask queues, checked in this priority order:

1. **`process.nextTick()` queue** — highest priority, Node.js-specific
2. **Promise microtask queue** — `.then()`, `.catch()`, `.finally()`, and `await` continuations

> 💡 **Rule:** After **every single** synchronous operation completes (and after every phase-callback runs), Node.js **fully drains** the `process.nextTick()` queue, then **fully drains** the Promise microtask queue, **before** moving on to the next macrotask or event loop phase.

---

## 🧪 Step 6: Execution Order — Worked Examples

### Example 1: Sync vs `setTimeout` vs Promise

```js
console.log("1: Start");

setTimeout(() => {
  console.log("2: setTimeout callback");
}, 0);

Promise.resolve().then(() => {
  console.log("3: Promise callback");
});

console.log("4: End");
```

**Output:**

```
1: Start
4: End
3: Promise callback
2: setTimeout callback
```

**Why?**
1. `console.log("1: Start")` runs immediately (synchronous).
2. `setTimeout` is handed off to libuv's timer phase — even with `0ms`, it must wait for the **current** synchronous code to finish AND for the microtask queue to drain.
3. `Promise.resolve().then()` schedules a **microtask**.
4. `console.log("4: End")` runs immediately (synchronous).
5. Call stack is now empty → Node.js drains the **microtask queue** first → `"3: Promise callback"` prints.
6. Only now does the event loop move into the **timers phase** → `"2: setTimeout callback"` prints.

### Example 2: `process.nextTick()` vs `Promise`

```js
console.log("Start");

Promise.resolve().then(() => console.log("Promise"));

process.nextTick(() => console.log("nextTick"));

console.log("End");
```

**Output:**

```
Start
End
nextTick
Promise
```

`process.nextTick()` **always** runs before Promise microtasks, no matter the order they were scheduled in.

### Example 3: `setTimeout` vs `setImmediate`

```js
setTimeout(() => console.log("setTimeout"), 0);
setImmediate(() => console.log("setImmediate"));
```

**Output: not guaranteed! ⚠️**

When called from the **main module** (top-level code), the order between `setTimeout(fn, 0)` and `setImmediate(fn)` is **not deterministic** — it depends on system performance and timing.

However, inside an **I/O callback**, `setImmediate()` is **always guaranteed to run before** `setTimeout(fn, 0)`:

```js
const fs = require("fs");

fs.readFile(__filename, () => {
  setTimeout(() => console.log("setTimeout"), 0);
  setImmediate(() => console.log("setImmediate"));
});
```

**Output (guaranteed):**

```
setImmediate
setTimeout
```

This happens because after an I/O callback runs (in the **poll** phase), the event loop moves to the **check** phase (`setImmediate`) **before** it loops back around to the **timers** phase (`setTimeout`).

### Example 4: A Realistic Mixed Example

```js
console.log("1");

setTimeout(() => console.log("2: timeout"), 0);

setImmediate(() => console.log("3: immediate"));

Promise.resolve().then(() => console.log("4: promise"));

process.nextTick(() => console.log("5: nextTick"));

console.log("6");
```

**Output:**

```
1
6
5: nextTick
4: promise
2: timeout   (or 3: immediate — order may vary at top level)
3: immediate (or 2: timeout)
```

Synchronous code (`1`, `6`) always runs first, then `nextTick`, then Promises, and only then do we enter the event loop's phases for timers/immediates.

---

## 🚫 Step 7: Don't Block the Event Loop!

Because everything shares **one thread**, any CPU-heavy synchronous code freezes your **entire server** for **every connected user** — not just the current request.

```js
// ❌ BAD: blocks the event loop for everyone
app.get("/heavy", (req, res) => {
  let sum = 0;
  for (let i = 0; i < 20_000_000_000; i++) {
    sum += i; // synchronous CPU-bound work — freezes the server
  }
  res.json({ sum });
});
```

### ✅ Solutions for CPU-Heavy Work

| Technique | When to Use |
|---|---|
| **`worker_threads`** | Run CPU-intensive JS code on a separate thread |
| **Child processes** (`child_process`) | Offload work to a separate Node.js process |
| **Break work into chunks** | Use `setImmediate()` between chunks to let the event loop breathe |
| **External services / queues** | Offload heavy jobs (image processing, PDF generation) to a background worker service (e.g. BullMQ + Redis) |

> 💡 I/O operations (database queries, file reads, HTTP calls) are **already non-blocking** in Node.js — they don't freeze the event loop. The danger is specifically **synchronous, CPU-bound** code.

---

## 🧾 Summary

| Concept | Key Idea |
|---|---|
| Single-threaded | Your JS code runs on ONE thread, one operation at a time |
| libuv | Handles async I/O and the thread pool behind the scenes |
| Call stack | Where synchronous code actually executes (LIFO) |
| Macrotask queue | `setTimeout`, `setInterval`, I/O callbacks, `setImmediate` |
| Microtask queue | `process.nextTick()` (highest priority) and Promise callbacks |
| Event loop phases | timers → pending callbacks → idle/prepare → poll → check → close callbacks |
| Microtasks vs macrotasks | ALL microtasks drain completely before the next macrotask/phase runs |
| Blocking the loop | Synchronous CPU-heavy code freezes the server for everyone — avoid it |

---

## 🧩 Hands-On Practice

1. Write a script mixing `console.log`, `setTimeout(fn, 0)`, `Promise.resolve().then()`, and `process.nextTick()`. Predict the output on paper first, then run it and compare.
2. Write a synchronous blocking loop inside an Express route, and confirm (using two browser tabs) that it blocks a *second* unrelated request while it runs.
3. Fix the blocking example using `setImmediate()` to break the loop into smaller chunks, and observe that other requests can now be handled in between.
4. Research and explain, in your own words, why `fs.readFileSync` should almost never be used inside an Express route handler.
