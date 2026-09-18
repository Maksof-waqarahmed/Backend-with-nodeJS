# ⚙️ Building a Simple Node.js Server with TypeScript

## 📖 Before You Start

This class assumes you already know TypeScript fundamentals (types, interfaces, functions). If you need to learn or review those first, go through the **[TypeScript course](../../../TypeScript/README.md)** — specifically [Week 1: Introduction to TypeScript & Environment Setup](../../../TypeScript/week_1/class_1/README.md) and [Basic Types & Type Annotations](../../../TypeScript/week_1/class_2/README.md).

Here, we focus purely on **Node.js itself** — creating your first real backend server using Node's built-in `http` module.

---

In this section, you’ll learn how to create and run your **first backend server** using **Node.js** and **TypeScript** — from zero to a running application.

We’ll cover:

* 🧠 What a Server Actually Is
* ⚙️ How Node.js Handles Requests & Responses
* 🪜 Step-by-Step Server Setup Using TypeScript
* 💻 Example Code with Complete Explanation
* 📡 Routing, Paths, and JSON Responses
* 🧩 Bonus: Understanding Headers, Ports, and Status Codes

---

## 🌍 What Is a Server?

> ❓ Common Confusion: “Is a server a computer or a program?”

✅ **Answer:**
A **server** can refer to both:

1. The **hardware (computer)** that stores and delivers data.
2. The **software (program)** that listens for and responds to requests.

So when we say “create a server in Node.js,” we mean **creating a server program** that runs on a machine and responds to client requests.

### 🧠 Simple Definition:

A **server** is a program that:

1. **Listens** for client requests (like from browsers, mobile apps, or APIs)
2. **Processes** those requests
3. **Sends back** a response — data, HTML, JSON, or error messages

**Example:**
When you visit `https://banoqabil.pk/`:

* Your browser sends a **request** to the server.
* The server processes it.
* Then sends back an **HTML page**, **JSON data**, or **error response**.

---

## ⚡ Why Build a Server with Node.js?

Node.js allows you to build **server-side applications** using **JavaScript or TypeScript** — the same language used for frontend.

Here’s why it’s so powerful:

| Feature                      | Description                                                          |
| ----------------------------- | ---------------------------------------------------------------------- |
| 🚀 **Fast Performance**      | Runs on Google Chrome’s V8 JavaScript engine.                        |
| ⚙️ **Non-blocking I/O**      | Handles multiple requests simultaneously without waiting.            |
| 🔁 **Asynchronous Nature**   | Doesn’t freeze when waiting for tasks like file reads or DB queries. |
| 💬 **Real-time Support**     | Ideal for chat apps, APIs, and live dashboards.                      |
| 🧩 **TypeScript Compatible** | Gives type safety and better scalability.                            |

---

## 🧩 Step 1: Prerequisites

Before creating your server, ensure you have:

* ✅ **Node.js** installed → [https://nodejs.org/](https://nodejs.org/)
* ✅ **TypeScript** initialized

  ```bash
  npm init -y
  npm install typescript @types/node --save-dev
  npx tsc --init
  ```

This creates a `tsconfig.json` file for TypeScript configuration.

---

## 🧠 Step 2: Project Structure

Organize your project like this:

```
project-folder/
│
├── src/
│   └── index.ts        # Main TypeScript file (server code)
│
├── dist/               # Compiled JavaScript files (auto-generated)
│
├── package.json
└── tsconfig.json
```

---

## 🧱 Step 3: Create a Basic Server

Create a new file:
`src/index.ts`

Add this code:

```ts
import http, { IncomingMessage, ServerResponse } from "http";

// Create a server
const server = http.createServer((req: IncomingMessage, res: ServerResponse) => {
  // Set response header
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Welcome to TypeScript Server 🚀");
});

// Start the server and listen on port 3000
server.listen(3000, () => {
  console.log("✅ Server running at http://localhost:3000");
});
```
When using TypeScript with Node.js, both `req` (request) and `res` (response) objects have specific **TypeScript types** that describe their structure and available properties.
These types come from Node.js’ built-in `@types/node` package.

---

### 🔹 `IncomingMessage`

```ts
import { IncomingMessage } from "http";
```

#### 🧾 Definition:

`IncomingMessage` is the **TypeScript type** that represents the **data coming from the client** — i.e., the **HTTP request**.

Whenever a client (like a browser or Postman) sends a request to your server, Node.js automatically wraps that request in an object of type `IncomingMessage`.

#### 🧩 Common Properties:

| Property                  | Type                  | Description                                                                                   |                                                         |
| -------------------------- | --------------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `req.url`                 | `string               | undefined`                                                                                    | The URL path requested (e.g., `/about`, `/home`).       |
| `req.method`              | `string               | undefined`                                                                                    | The HTTP method (e.g., `GET`, `POST`, `PUT`, `DELETE`). |
| `req.headers`             | `IncomingHttpHeaders` | The request headers (like `Content-Type`, `Authorization`, etc.).                             |                                                         |
| `req.statusCode`          | `number               | undefined`                                                                                    | Status code (usually used internally).                  |
| `req.on(event, listener)` | Function              | Allows listening to request events like `"data"` and `"end"` (used for reading request body). |                                                         |

#### 🧠 Example:

```ts
if (req.method === "GET" && req.url === "/") {
  console.log("Received a GET request at the home page!");
}
```

So `IncomingMessage` ensures TypeScript knows **which properties exist** and **what types they hold** — giving you **autocompletion and error checking**.

---

### 🔹 `ServerResponse`

```ts
import { ServerResponse } from "http";
```

#### 🧾 Definition:

`ServerResponse` is the **TypeScript type** for the **response object** that the server sends back to the client.
It represents the **outgoing HTTP response**.

This object allows you to:

* Set headers
* Define a status code
* Send text or JSON data
* End the response

#### 🧩 Common Methods:

| Method                               | Description                                                                                   |
| ------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `res.writeHead(statusCode, headers)` | Sets HTTP status and headers. Example: `res.writeHead(200, { "Content-Type": "text/html" })`. |
| `res.statusCode = 200`               | Sets the response status manually.                                                            |
| `res.setHeader(name, value)`         | Adds or modifies a single HTTP header.                                                        |
| `res.write(data)`                    | Sends a chunk of the response body.                                                           |
| `res.end(data?)`                     | Signals that the response is complete (must be called once).                                  |

#### 🧠 Example:

```ts
res.writeHead(200, { "Content-Type": "application/json" });
res.end(JSON.stringify({ message: "Hello from TypeScript Server 🚀" }));
```

Here, `ServerResponse` ensures you can only call methods that exist on a valid Node.js response object — protecting you from typos or misuse.

---

## 🔍 Step 4: Explanation of Each Part

### 1️⃣ `import http, { IncomingMessage, ServerResponse } from "http";`

* Node.js provides a **built-in `http` module** — no installation required.
* It allows you to **create servers, send requests, and manage responses**.

---

### 2️⃣ `http.createServer((req: IncomingMessage, res: ServerResponse) => {...})`

* This method **creates an HTTP server** that listens for requests.
* It takes a callback with two parameters:

  * `req` → represents the **incoming request**
  * `res` → represents the **response** object

Example:

```ts
req.url    // The requested path, e.g., "/about"
req.method // The HTTP method, e.g., "GET" or "POST"
```

---

### 3️⃣ `res.writeHead(200, { "Content-Type": "text/plain" });`

* This sets the **status code** and **headers** for the response.

| Concept         | Description                                                                   |
| ---------------- | ------------------------------------------------------------------------------- |
| **Status Code** | Tells the client whether the request succeeded (e.g., 200 OK, 404 Not Found). |
| **Headers**     | Provide meta-information about the response (like format, length, encoding).  |

Example:

```ts
"Content-Type": "text/plain"      // plain text
"Content-Type": "application/json" // JSON data
"Content-Type": "text/html"        // HTML content
```

**💡 Why headers are important?**
If you don’t specify the content type, the browser won’t know how to interpret the data — it might display it incorrectly or even trigger download behavior.

---

### 4️⃣ `res.end("Welcome...")`

* Ends the response and sends data back to the client.
* Without `res.end()`, the server keeps waiting and the page won’t load.

---

### 5️⃣ `server.listen(3000, callback)`

* Starts the server on **port 3000** (like your app’s “door number”).
* You can access it via:
  `http://localhost:3000`

**Common Ports:**

| Port | Usage                       |
| ---- | ---------------------------- |
| 3000 | React / Node.js development |
| 4000 | API servers                 |
| 5000 | Custom backend              |

---

## 🧮 Step 5: Compile and Run

### Option 1 – Manual Compilation

```bash
npx tsc
node dist/index.js
```

### Option 2 – Using ts-node (direct execution)

```bash
npx ts-node src/index.ts
```

### Option 3 – Using tsx (recommended)

```bash
npx tsx src/index.ts
```

---

## 📡 Step 6: Handling Multiple Routes

Now let’s serve different pages based on the request URL.

```ts
import http, { IncomingMessage, ServerResponse } from "http";

const server = http.createServer((req: IncomingMessage, res: ServerResponse) => {
  if (req.url === "/") {
    res.end("🏠 Home Page");
  } else if (req.url === "/about") {
    res.end("ℹ️ About Page");
  } else {
    res.statusCode = 404;
    res.end("❌ Page Not Found");
  }
});

server.listen(3000, () => {
  console.log("✅ Server running at http://localhost:3000");
});
```

---

### 🧭 Understanding Path, URL, and Endpoint

| Term         | Meaning                     | Example                     |
| ------------ | ----------------------------- | ------------------------------ |
| **URL**      | Full address of a resource  | `https://example.com/about` |
| **Path**     | The part after the domain   | `/about`                    |

So in the above example:

* `/` → Home route
* `/about` → About route

---

## ⚙️ Sending JSON Responses

If you want to send structured data (like APIs), use **JSON** format.

```ts
if (req.url === "/user") {
  res.writeHead(200, { "Content-Type": "application/json" });
  const user = { name: "Waqar Rana", role: "Developer" };
  res.end(JSON.stringify(user));
}
```

### ❓Why use `JSON.stringify()`?

Because the `res.end()` method only sends **text data**, not objects.
`JSON.stringify()` converts a JavaScript object into a **JSON string** that the client can understand.

Example:

```json
{
  "name": "Waqar Rana",
  "role": "Developer"
}
```

---

## 🧰 Step 7: Add Scripts in package.json

To simplify running your app, edit your `package.json`:

```json
"scripts": {
  "start": "node dist/index.js",
  "dev": "npx tsx src/index.ts"
}
```

Now you can run:

```bash
npm run dev
```

---

## 🧾 Summary

| Concept               | Description                                   |
| ---------------------- | ------------------------------------------------ |
| `http.createServer()` | Creates a new Node.js HTTP server             |
| `req` / `res`         | Handle incoming request and outgoing response |
| `res.writeHead()`     | Set status code and headers                   |
| `res.end()`           | Send and finish the response                  |
| `server.listen()`     | Starts the server and listens on a port       |
| `tsx` / `ts-node`     | Run TypeScript directly without compiling     |

---

## 🚀 Final Output

Run the server:

```bash
npm run dev
```

Terminal Output:

```
✅ Server running at http://localhost:3000
```

Browser Output:

```
Welcome to TypeScript Server 🚀
```

---

## 🧩 Bonus Tip: From Native to Frameworks

Once you understand **native Node.js servers**, you can easily move to frameworks like:

| Framework      | Description                                      |
| --------------- | --------------------------------------------------- |
| **Express.js** | Most popular Node.js framework for APIs          |
| **NestJS**     | Enterprise-grade framework built with TypeScript |
| **Fastify**    | Lightweight and performance-focused alternative  |

They build on top of what you just learned — handling routes, requests, and responses automatically.

---

## 🧩 Hands On Practice:

1. Hands-on: Create a TypeScript-powered Node.js server including different routes and responses.
2. Add a `/user` route that returns a JSON object instead of plain text.
3. Add a 404 handler for any route that isn't `/`, `/about`, or `/user`.

Next class (Week 1, Class 3): **[Node.js Core Fundamentals — Modules, Built-in APIs & Asynchronous Programming](../class_3/README.md)**. 🚀
