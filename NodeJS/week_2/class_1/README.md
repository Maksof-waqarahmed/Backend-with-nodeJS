# 🌐 Understanding HTTP Status Codes in Node.js & TypeScript

HTTP status codes are **three-digit numbers** returned by a server to tell the client (like a browser, mobile app, or API) **what happened** after making a request.

Every time you send a request — for example, visiting a page, submitting a form, or fetching API data — the server replies with a **status code** that helps describe the **result** of that operation.

---

## ⚙️ What is an HTTP Status Code?

An **HTTP Status Code** is part of the **response message** sent from the server to the client.
It informs the client whether the request was successful, redirected, failed, or caused an error.

### Example:

```bash
GET /home HTTP/1.1
Host: localhost:3000
```

Server response:

```bash
HTTP/1.1 200 OK
Content-Type: text/html
```

Here:

* `200` → is the **status code**
* `OK` → is the **status message**

---

## 🧭 The 5 Categories of HTTP Status Codes

| Category | Range   | Meaning                                                      |
| -------- | ------- | ------------------------------------------------------------ |
| **1xx**  | 100–199 | Informational – Request received, continuing process         |
| **2xx**  | 200–299 | Success – Request was successfully received and processed    |
| **3xx**  | 300–399 | Redirection – Further action needed (e.g., new URL)          |
| **4xx**  | 400–499 | Client Error – The request had an issue from client-side     |
| **5xx**  | 500–599 | Server Error – The server failed to complete a valid request |

---

## 🧩 1xx – Informational Responses

These are **rarely used** in modern applications.
They simply mean *“the request has started processing”*.

| Code | Name                | Brief Meaning                                                                               |
| ---- | ------------------- | ------------------------------------------------------------------------------------------- |
| 100  | Continue            | Server received initial part of request; client should continue.            |
| 101  | Switching Protocols | Server will switch protocols (e.g., to WebSocket) as requested.             |
| 102  | Processing          | Server received request and is processing it, but not yet a final response. |
| 103  | Early Hints         | Used to send preliminary headers (e.g., Link) before full response.        |

---

## ✅ 2xx – Success Codes

These indicate that the client’s request was **successfully received, understood, and accepted**.

| Code | Name                          | Brief Meaning                                                                                                                                                      |
| ---- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 200  | OK                            | Standard success for GET, POST etc.                                                                                                                |
| 201  | Created                       | Request succeeded and a new resource was created.                                                                                                  |
| 202  | Accepted                      | Request accepted but processing not yet complete.                                                                                                  |
| 203  | Non-Authoritative Information | Response from a proxy rather than the origin server.                                                                                               |
| 204  | No Content                    | Success but no content to send back.                                                                                                               |
| 205  | Reset Content                 | Success; client should reset view.                                                                                                                 |
| 206  | Partial Content               | Partial data sent in response to a “Range” request.                                                                                                |
| 207  | Multi-Status                  | WebDAV: multiple responses for multiple operations.                                                                                                |
| 208  | Already Reported              | WebDAV: members of a DAV binding list already reported.                                                                                            |
| 226  | IM Used                       | The server has fulfilled a GET request for the resource, and the response is a representation of the result of one or more instance-manipulations. |

### 🧠 Example:

```ts
res.writeHead(201, { "Content-Type": "application/json" });
res.end(JSON.stringify({ message: "User created successfully ✅" }));
```

---

## 🔁 3xx – Redirection Codes

These codes tell the client that the **requested resource has moved** to another URL or location.

| Code | Name                                 | Brief Meaning                                                               |
| ---- | ------------------------------------ | --------------------------------------------------------------------------- |
| 300  | Multiple Choices                     | Multiple options for the resource; user or agent to choose. |
| 301  | Moved Permanently                    | Resource permanently moved to new URI.                     |
| 302  | Found (formerly “Moved Temporarily”) | Resource temporarily at a different URI.                    |
| 303  | See Other                            | Response to request can be found under another URI by GET.  |
| 304  | Not Modified                         | Resource not modified since last request; use cache.        |
| 305  | Use Proxy                            | Requested resource available only via proxy.                |
| 306  | (Unused)                             | Previously “Switch Proxy”, now unused.                     |
| 307  | Temporary Redirect                   | Like 302 but method must not change.                        |
| 308  | Permanent Redirect                   | Like 301 but method cannot change.                          |

### 🧠 Example:

```ts
res.writeHead(301, { Location: "/new-page" });
res.end();
```

This sends a **redirect** to the client, taking them automatically to `/new-page`.

---

## ⚠️ 4xx – Client Error Codes

These indicate problems with the **client’s request** — such as wrong data, missing authorization, or invalid routes.

| Code | Name                                         | Brief Meaning                                                                                               |
| ---- | -------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| 400  | Bad Request                                  | Client sent invalid request.                                                       |
| 401  | Unauthorized                                 | Authentication required and has failed or not yet provided.                                 |
| 402  | Payment Required                             | Reserved for future use / digital payments.                                                 |
| 403  | Forbidden                                    | Server understands request but refuses to authorize.                                       |
| 404  | Not Found                                    | Requested resource not found.                                                      |
| 405  | Method Not Allowed                           | Method not supported for the resource.                                             |
| 406  | Not Acceptable                               | Server cannot generate content acceptable by client.                               |
| 407  | Proxy Authentication Required                | Client must authenticate with proxy.                                               |
| 408  | Request Timeout                              | Server timed out waiting for request.                                                       |
| 409  | Conflict                                     | Request conflicts with current state of server.                                             |
| 410  | Gone                                         | Resource no longer available and no forwarding address.                            |
| 411  | Length Required                              | Server refuses to accept request without Content-Length header.                             |
| 412  | Precondition Failed                          | One or more conditions given in request headers failed.                                     |
| 413  | Payload Too Large / Request Entity Too Large | Request is larger than server is willing/able to process.                          |
| 414  | URI Too Long                                 | URI requested is too long.                                                                  |
| 415  | Unsupported Media Type                       | Request entity has a media type the server does not support.                       |
| 416  | Range Not Satisfiable                        | Requested range cannot be fulfilled.                                                        |
| 417  | Expectation Failed                           | Server cannot meet the requirements of the Expect header.                                   |
| 418  | I’m a Teapot (RFC 2324)                      | Easter-egg code; server refuses to brew coffee.                                            |
| 421  | Misdirected Request                          | Request directed at a server that is not able to produce a response.                        |
| 422  | Unprocessable Entity                         | WebDAV: request was well-formed but unable to process.                                      |
| 423  | Locked                                       | WebDAV: resource is locked.                                                                 |
| 424  | Failed Dependency                            | WebDAV: failure in previous request dependency.                                             |
| 425  | Too Early                                    | Indicates that the server is unwilling to risk processing a request that might be replayed. |
| 426  | Upgrade Required                             | Client should switch to a different protocol.                                               |
| 428  | Precondition Required                        | Request must be conditional.                                                                |
| 429  | Too Many Requests                            | Client sent too many requests in a given amount of time ("rate limiting").                  |
| 431  | Request Header Fields Too Large              | Server refuses because headers are too large.                                               |
| 451  | Unavailable For Legal Reasons                | Resource unavailable for legal/censorship reason.                                              |

### 🧠 Example:

```ts
if (req.url !== "/home") {
  res.writeHead(404, { "Content-Type": "text/plain" });
  res.end("404 - Page Not Found ❌");
}
```

---

## 💣 5xx – Server Error Codes

These indicate that the **server failed** to fulfill a valid request due to internal issues.

| Code | Name                            | Brief Meaning                                                                                            |
| ---- | ------------------------------- | -------------------------------------------------------------------------------------------------------- |
| 500  | Internal Server Error           | Generic server error when no more specific code fits.                                    |
| 501  | Not Implemented                 | Server does not support functionality required to fulfill request.                       |
| 502  | Bad Gateway                     | Server, while acting as gateway/proxy, received invalid response from upstream server.  |
| 503  | Service Unavailable             | Server currently unavailable (overloaded/maintenance).                                   |
| 504  | Gateway Timeout                 | Gateway or proxy did not get response in time.                                           |
| 505  | HTTP Version Not Supported      | Server does not support HTTP protocol version used in request.                           |
| 506  | Variant Also Negotiates         | Internal server error; transparent content negotiation loop.                             |
| 507  | Insufficient Storage            | WebDAV: Server unable to store representation needed to complete request.                |
| 508  | Loop Detected                   | WebDAV: Server detected infinite loop while processing a request with “Depth: infinity”. |
| 510  | Not Extended                    | Further extensions to the request are required for server to fulfil it.                  |
| 511  | Network Authentication Required | Client needs network authentication (e.g., captive portal) before accessing network.     |

### 🧠 Example:

```ts
try {
  throw new Error("Unexpected failure");
} catch (error) {
  res.writeHead(500, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ message: "Internal Server Error 💥" }));
}
```

---

### 📌 Notes:

* For most typical API/back-end work, you’ll mainly use a subset: **200, 201, 204, 301, 400, 401, 403, 404, 422, 429, 500, 503** etc.

---

## 💡 Why Status Codes Are Important

| Benefit                    | Explanation                                                         |
| -------------------------- | ------------------------------------------------------------------- |
| 🧠 **Clear Communication** | Helps the client know whether the request was successful or failed. |
| ⚙️ **Debugging**           | Developers can easily identify what went wrong.                     |
| 🔒 **Security**            | Proper use of codes prevents exposing sensitive info.               |
| 🌍 **Standardization**     | Every system (browser, mobile, backend) understands these codes.    |

---

## 🧩 Complete Example in TypeScript

Here’s a small **Node.js + TypeScript** server demonstrating multiple status codes:

```ts
import http, { IncomingMessage, ServerResponse } from "http";

const server = http.createServer((req: IncomingMessage, res: ServerResponse) => {
  if (req.url === "/") {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("✅ Home Page - Status: 200 OK");
  } 
  else if (req.url === "/create") {
    res.writeHead(201, { "Content-Type": "application/json" });
    res.end(JSON.stringify({ message: "Resource created successfully" }));
  } 
  else if (req.url === "/redirect") {
    res.writeHead(301, { Location: "/new-location" });
    res.end();
  } 
  else if (req.url === "/error") {
    res.writeHead(500, { "Content-Type": "text/plain" });
    res.end("💥 Internal Server Error");
  } 
  else {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("❌ Page Not Found");
  }
});

server.listen(3000, () => {
  console.log("🚀 Server running at http://localhost:3000");
});
```

---

## 🧭 Output Summary

| URL         | Status Code | Response                     |
| ----------- | ----------- | ---------------------------- |
| `/`         | 200         | ✅ Home Page - OK             |
| `/create`   | 201         | JSON: Resource created       |
| `/redirect` | 301         | Redirects to `/new-location` |
| `/error`    | 500         | 💥 Internal Server Error     |
| `/random`   | 404         | ❌ Page Not Found             |

---

## 🧱 Quick Tips for Professionals

✅ Always use **meaningful status codes** in APIs (never send 200 for all cases).
✅ Combine with proper **JSON responses** like `{ "success": false, "message": "Not Found" }`.
✅ Log `500` errors on the server — they usually mean code bugs.
✅ Use tools like **Postman** or **curl** to test different response scenarios.
✅ Keep your API consistent — always return standard formats.

---

## 🧩 Practice Tasks (For Students)

1. Create a simple server that:

   * Returns `200` for `/`
   * Returns `201` for `/create`
   * Returns `404` for `/notfound`
   * Returns `500` for `/error`
2. Log the status code and URL in the console each time a request is made.
3. Try returning JSON responses instead of plain text.

---

# 🌐 Introduction to RESTful APIs and HTTP Methods

## 📘 Overview

Modern web and mobile apps need a way to **communicate** with servers to exchange data — that’s where **APIs** come in.
In backend development, the most common type of API is the **RESTful API**.

This guide covers:

* What an API is 🧠
* What REST means ⚙️
* HTTP methods (GET, POST, PUT, DELETE)
* How REST APIs work in Node.js
* Example with TypeScript

---

## 🧠 What is an API?

**API** stands for **Application Programming Interface**.

In simple terms:

> It’s a way for two applications (or systems) to talk to each other.

### Visual Representation

<img src="./images/api-works" alt="API Communication" />

Example:
When you open Instagram:

* The mobile app calls the **Instagram API** on the backend.
* That API sends back your profile data, images, and likes.

In backend development:

> APIs allow frontend apps (React, mobile, etc.) to **communicate** with your Node.js server.

---

## 🔁 How API Communication Works

### Visual Representation

<img src="./images/rest3.png" alt="API Flow" />

---

## ⚙️ What is REST?

**REST** stands for **Representational State Transfer**.
It’s not a programming language or a framework — it’s an **architectural style** or **set of rules** for building scalable, maintainable, and efficient **web APIs**.

A **RESTful API** is an API that **follows REST principles** and communicates over the **HTTP protocol**.

---

## 🧠 Simple Understanding

Imagine a restaurant 🍽️:

* You (the client) request a dish from the waiter.
* The waiter (the API) takes your request to the chef.
* The chef (the server/database) prepares your food and sends it back through the waiter.

Similarly:

* The **client** sends an HTTP request.
* The **server** processes it and sends back a **response**.
* The **API** is the medium connecting both.

That’s **REST in action** — simple, consistent, and stateless communication between client and server.

---

A **RESTful API** follows these principles:

| Principle             | Description                                                                 |
| --------------------- | --------------------------------------------------------------------------- |
| **Client-Server**     | Frontend (client) and backend (server) are separate                         |
| **Stateless**         | Each request contains all info needed — server doesn’t store previous state |
| **Uniform Interface** | Consistent endpoints and formats                                            |
| **Resource-Based**    | Data (like users, posts, products) are called *resources*                   |
| **HTTP Methods**      | Uses standard HTTP methods for CRUD operations                              |
  
---

### Visual Representation

<img src="./images/rest4.png" alt="RESTful API" />

<img src="./images/rest2.jpg" alt="RESTful API" />


## 🌐 Understanding HTTP Methods in Backend Development

HTTP methods (also known as **HTTP verbs**) define the **type of action** you want to perform on a resource (data) in a web server.

These methods tell the server **what you want to do** — such as:

* Fetch data
* Create new data
* Update existing data
* Delete data

---

## ⚙️ What is HTTP?

**HTTP (HyperText Transfer Protocol)** is the foundation of data communication on the web.
It defines **how clients (like browsers or apps)** and **servers** communicate.

When you visit a website or call an API:

1. Your browser (client) sends an **HTTP request**.
2. The server processes it and sends back an **HTTP response**.

Each request includes:

* **Method** (e.g., GET, POST)
* **URL/Endpoint**
* **Headers**
* **Body** (for methods like POST or PUT)

### Visual Representation

<img src="./images/method.gif" alt="RESTful API" />

## 🧩 Common HTTP Methods Explained

### 🟩 1. **GET — Retrieve Data**

**Purpose:** Used to **fetch or read data** from the server.
It **never modifies** data — just requests information.

**Key Points:**

* Safe and idempotent (can be called multiple times safely)
* Sends data through the **URL** (query parameters)
* Usually used to **display or fetch lists, details, or search results**

**Example:**

```bash
GET /books
```

**Response:**

```json
[
  { "id": 1, "title": "Atomic Habits" },
  { "id": 2, "title": "Deep Work" }
]
```

---

### 🟦 2. **POST — Create Data**

**Purpose:** Used to **add new data** to the server.

**Key Points:**

* Sends data inside the **request body**.
* The server processes and stores it.
* Returns confirmation or the created object.

**Example:**

```bash
POST /books
```

**Request Body:**

```json
{ "title": "The Alchemist" }
```

**Response:**

```json
{ "message": "Book added successfully!" }
```

---

### 🟨 3. **PUT — Full Update**

**Purpose:** Used to **update an existing resource completely**.
If some fields are missing, they will be **replaced** or **overwritten**.

**Key Points:**

* Idempotent (same result no matter how many times you send it)
* Replaces the entire resource with new data

**Example:**

```bash
PUT /books/2
```

**Request Body:**

```json
{ "title": "Deep Work (Updated Edition)" }
```

**Response:**

```json
{ "message": "Book updated successfully!" }
```

---

### 🟥 4. **DELETE — Remove Data**

**Purpose:** Used to **delete** data from the server.

**Key Points:**

* Removes the specified resource
* Usually requires an ID or identifier in the URL

**Example:**

```bash
DELETE /books/2
```

**Response:**

```json
{ "message": "Book deleted successfully!" }
```

---

### 🟧 5. **PATCH — Partial Update**

**Purpose:** Used to **update only specific fields** of an existing resource.
Unlike PUT, it doesn’t replace the whole object.

**Example:**

```bash
PATCH /books/1
```

**Request Body:**

```json
{ "author": "James Clear" }
```

**Response:**

```json
{ "message": "Book partially updated!" }
```

---

## 💬 PUT vs PATCH — Key Difference

| Method    | Purpose        | Behavior                      |
| --------- | -------------- | ----------------------------- |
| **PUT**   | Full update    | Replaces the entire resource  |
| **PATCH** | Partial update | Updates only specified fields |

**Example:**

✅ PUT replaces everything

```json
{ "title": "New Title" }
```

✅ PATCH changes only one field

```json
{ "author": "New Author" }
```

---

## 🌍 How a Typical API Uses HTTP Methods

| Method     | Example Endpoint | Purpose                  |
| ---------- | ---------------- | ------------------------ |
| **GET**    | `/users`         | Fetch all users          |
| **GET**    | `/users/1`       | Fetch a single user      |
| **POST**   | `/users`         | Add a new user           |
| **PUT**    | `/users/1`       | Replace an existing user |
| **PATCH**  | `/users/1`       | Update specific fields   |
| **DELETE** | `/users/1`       | Delete a user            |

---

## 🧠 Summary

| Method     | Action           | Safe? | Idempotent? | Request Body |
| ---------- | ---------------- | ----- | ----------- | ------------ |
| **GET**    | Read             | ✅ Yes | ✅ Yes       | ❌ No         |
| **POST**   | Create           | ❌ No  | ❌ No        | ✅ Yes        |
| **PUT**    | Update (Full)    | ❌ No  | ✅ Yes       | ✅ Yes        |
| **PATCH**  | Update (Partial) | ❌ No  | ✅ Yes       | ✅ Yes        |
| **DELETE** | Delete           | ❌ No  | ✅ Yes       | ❌ No         |

---

## 🎯 Key Takeaways

* HTTP methods define **how you interact** with a resource (CRUD operations).
* Each method has a **specific purpose** — don’t mix them up.
* Always return **meaningful status codes** (e.g., 200 OK, 201 Created, 404 Not Found).
* **TypeScript** helps enforce type safety when handling requests and responses.

---

# 1) What is Express.js?

**Express.js** is a **minimal, flexible** web framework for Node.js that simplifies building web servers and APIs.
It provides a thin layer of features — routing, middleware, request/response helpers — so you don't have to implement low-level HTTP handling manually.

**Key idea:** Express gives you a structured, readable API to create endpoints while staying unopinionated (you choose how to structure the app).

---

# 2) Why do we need Express?

Without a framework, creating a web server requires manual handling of many common tasks:

* Parsing request bodies (JSON, form data)
* Routing (matching URLs and HTTP methods)
* Handling headers and status codes consistently
* Middleware chains (authentication, logging, body parsing)
* Error handling, central logging, and static file serving

**Express solves these** by offering:

* `app.get()`, `app.post()` etc. for routing
* `express.json()` and `express.urlencoded()` for parsing the body
* Middleware system (`app.use()`) for reusable logic
* Clear separation of concerns (routes, middleware, controllers)
* Easy integration with community middleware (CORS, helmet, helmet for security, morgan for logging, etc.)

---

# 3) Node.js vs Express.js — quick comparison

|      Feature | Raw Node.js (http module)             | Express.js                                |
| -----------: | ------------------------------------- | ----------------------------------------- |
|      Routing | Manual `if (req.url ...)`             | `app.get`, `app.post`, routers            |
| Parsing body | Manual `req.on('data')` & parse       | `express.json()` / `express.urlencoded()` |
|   Middleware | Ad-hoc implementation needed          | Built-in middleware pipeline (`next()`)   |
|  Readability | Verbose & low-level                   | Cleaner, declarative                      |
|    Ecosystem | You write more code                   | Rich middleware ecosystem                 |
|     Use case | Very small scripts / custom protocols | Most web APIs and apps                    |

---

# 4) Who made Express & when?

* **Created by:** TJ Holowaychuk
* **First release:** around **2010**.
* Over time it became the de-facto standard web framework for Node.js and is now maintained by the Express.js core team and the community.

---

# 5) Other popular Node.js frameworks

* **Fastify** — high performance, schema-based validation.
* **Koa** — by the creators of Express, minimal and modern (uses async/await middleware).
* **NestJS** — full-featured, opinionated framework built with TypeScript (good for large apps).
* **Hapi** — configuration-first framework.
* **Sails.js** — MVC-style, useful for data-driven apps.

---

# 6) Basic Express + TypeScript server

We’ll build a simple server, then extend it with a small in-memory CRUD for `books`.

### Prerequisites

* Node.js installed (v14+ recommended)
* Basic familiarity with npm and TypeScript

---

## 6.1 Install & initialize

```bash
mkdir express-ts-server
cd express-ts-server
npm init -y
```

Install packages:

```bash
# production dependency
npm install express

# dev dependencies (TypeScript and types)
npm install -D typescript tsx @types/node @types/express nodemon
```

* `express` — framework
* `typescript` — compiler
* `tsx` — run TS files directly (fast)
* `@types/*` — TypeScript definitions for Node and Express
* `nodemon` — restarts server in dev when files change

---

## 6.2 tsconfig.json

Create `tsconfig.json` (you can run `npx tsc --init` then edit). Example minimal:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

**Key fields:**

* `target`: language version of emitted JS
* `module`: `CommonJS` for Node.js
* `rootDir` / `outDir`: source and compiled dirs
* `strict`: enables strict typing
* `esModuleInterop`: compatibility with `import express from "express"`

---

## 6.3 package.json scripts

Edit `package.json` scripts:

```json
"scripts": {
  "dev": "npx tsx src/server.ts",
  "start": "node dist/server.js",
  "build": "tsc"
}
```

* `npm run dev` — development server with auto restart (uses tsx to run TypeScript without compiling to dist)
* `npm run build` — compile to `dist/`
* `npm start` — run compiled JS in production

---

## 6.4 Create the basic server: `src/server.ts`

Create `src/server.ts` with detailed comments:

```ts
import express, { Request, Response, NextFunction } from "express";

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware: parse incoming JSON bodies
app.use(express.json());

// Basic GET route (home)
app.get("/", (req: Request, res: Response) => {
  // res.status(200) is default for successful responses from GET
  res.status(200).send("🚀 Welcome to Express + TypeScript Server");
});

// Start the server
app.listen(PORT, () => {
  console.log(`✅ Server running at http://localhost:${PORT}`);
});
```

### Explanation (line by line)

* `import express, { Request, Response, NextFunction } from "express";`
  Imports Express and TypeScript types for request/response.

* `const app = express();`
  Creates an Express application instance. This `app` will register routes and middleware.

* `app.use(express.json());`
  Built-in middleware to parse `application/json` request bodies and populate `req.body`. Without this, `req.body` is `undefined` for JSON POSTs.

* `app.get("/", (req, res) => { ... });`
  Defines a GET endpoint at `/`. `req` and `res` are typed. `res.send()` sets appropriate headers and sends the content.

* `app.listen(PORT, ...)`
  Starts listening on the given port. In development you’ll see the console message.

---

## 🧭 Understanding Express.js Core Concepts

---

### 1️⃣ Routing in Express

Routing defines how your app responds to client requests at specific URLs.

**Syntax:**

```ts
app.METHOD(PATH, HANDLER)
```

| Term      | Meaning                                      |
| --------- | -------------------------------------------- |
| `METHOD`  | HTTP method (`get`, `post`, `put`, `delete`) |
| `PATH`    | URL endpoint (`"/users"`, `"/books/:id"`)    |
| `HANDLER` | Function handling request & response         |

---

### 🔹 Example: Basic Routes

```ts
app.get("/users", (req: Request, res: Response) => {
  res.json([{ id: 1, name: "Rana" }]);
});
```

---

### `req.body`

Use `req.body` to access JSON body sent in POST/PUT requests

```ts
app.post("/data", (req: Request, res: Response) => {
  const data = req.body;
  res.json({ message: "Data received", data });
});
```

---

### 🔸 Dynamic Routes with Parameters

Use `req.params` to handle dynamic IDs.

```ts
app.get("/users/:id", (req: Request, res: Response) => {
  const id = req.params.id;
  res.json({ message: `Fetching user with ID: ${id}` });
});
```

---

### ⚙️ Sending Responses

Express provides multiple methods to send responses:

| Method             | Description            |
| ------------------ | ---------------------- |
| `res.send()`       | Sends text/HTML        |
| `res.json()`       | Sends JSON             |
| `res.status()`     | Sets HTTP status code  |
| `res.sendStatus()` | Sends only status code |

**Example:**

```ts
res.status(200).json({ message: "Success" });
```

---

# 7) Hands-on CRUD with static (in-memory) data

We’ll create a small `books` resource stored in memory (array). This demonstrates typical REST CRUD (Create / Read / Update / Delete).

Create `src/routes/books.ts` (or keep in `server.ts` for simplicity). Below is a single-file example with full explanation.

### 7.1 Full code: `src/server.ts` (CRUD example)

```ts
import express, { Request, Response, NextFunction } from "express";

const app = express();
const PORT = process.env.PORT || 3000;

// Use middleware to parse JSON
app.use(express.json());

// --- In-memory data store (static for demo purposes) ---
type Book = {
  id: number;
  title: string;
  author?: string;
};

let books: Book[] = [
  { id: 1, title: "Atomic Habits", author: "James Clear" },
  { id: 2, title: "Deep Work", author: "Cal Newport" }
];

// Utility to find next ID (simple)
const getNextId = (): number => (books.length ? Math.max(...books.map(b => b.id)) + 1 : 1);

// --- Routes ---

// GET /books - list all books
app.get("/books", (req: Request, res: Response) => {
  res.status(200).json(books);
});

// GET /books/:id - get a single book
app.get("/books/:id", (req: Request, res: Response) => {
  const id = Number(req.params.id);
  const book = books.find(b => b.id === id);
  if (!book) {
    return res.status(404).json({ message: "Book not found" });
  }
  res.json(book);
});

// POST /books - create a new book
app.post("/books", (req: Request, res: Response) => {
  const { title, author } = req.body as Partial<Book>;
  if (!title || typeof title !== "string") {
    return res.status(400).json({ message: "Title is required and must be a string" });
  }
  const newBook: Book = { id: getNextId(), title, author };
  books.push(newBook);
  res.status(201).json(newBook);
});

// PUT /books/:id - full replace (must provide title)
app.put("/books/:id", (req: Request, res: Response) => {
  const id = Number(req.params.id);
  const { title, author } = req.body as Partial<Book>;
  if (!title || typeof title !== "string") {
    return res.status(400).json({ message: "Title is required and must be a string" });
  }

  const idx = books.findIndex(b => b.id === id);
  if (idx === -1) return res.status(404).json({ message: "Book not found" });

  const updated: Book = { id, title, author };
  books[idx] = updated;
  res.json(updated);
});

// PATCH /books/:id - partial update
app.patch("/books/:id", (req: Request, res: Response) => {
  const id = Number(req.params.id);
  const { title, author } = req.body as Partial<Book>;
  const book = books.find(b => b.id === id);
  if (!book) return res.status(404).json({ message: "Book not found" });

  if (title !== undefined) {
    if (typeof title !== "string") return res.status(400).json({ message: "Title must be a string" });
    book.title = title;
  }
  if (author !== undefined) book.author = author;

  res.json(book);
});

// DELETE /books/:id - delete a book
app.delete("/books/:id", (req: Request, res: Response) => {
  const id = Number(req.params.id);
  const idx = books.findIndex(b => b.id === id);
  if (idx === -1) return res.status(404).json({ message: "Book not found" });

  const deleted = books.splice(idx, 1)[0];
  res.json({ message: "Book deleted", deleted });
});

// Simple global error handler (this example doesn't throw errors, but good practice)
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error("Unhandled error:", err);
  res.status(500).json({ message: "Internal Server Error" });
});

// Start server
app.listen(PORT, () => {
  console.log(`✅ Server running at http://localhost:${PORT}`);
});
```

---

### 7.2 Explanation of CRUD routes

**GET /books**

* Returns the whole array of books.
* `res.status(200).json(books)` — 200 OK with JSON payload.

**GET /books/:id**

* URL param `:id` available at `req.params.id`. Convert to number and find the book. If not found, return 404.

**POST /books**

* Reads JSON body via `req.body` (requires `express.json()` middleware).
* Validates required `title`. If invalid, returns 400. On success, creates new book, pushes to array, and returns `201 Created` with the created object.

**PUT /books/:id**

* Full replacement: requires `title`. Replace the entire resource with the supplied object. Idempotent — repeated calls produce same result.

**PATCH /books/:id**

* Partial update: only update fields present in body. Validate types when needed.

**DELETE /books/:id**

* Remove item from array and return confirmation.

**Global error handler**

* Express calls handlers with `(err, req, res, next)` when `next(err)` is used or exceptions happen in async code (with proper handling). Useful for production to avoid leaking stack traces.

---

# 8) Testing with Postman and curl

### 8.1 Start your server (dev)

```bash
npm run dev
# or
npx tsx src/server.ts
```

### 8.2 GET all books (curl & Postman)

**curl:**

```bash
curl -i http://localhost:3000/books
```

**Response:**

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

[{"id":1,"title":"Atomic Habits","author":"James Clear"}, ...]
```

**Postman:**

* Method: GET
* URL: `http://localhost:3000/books`
* Send → view JSON array in Body tab.

---

### 8.3 GET single book

**curl:**

```bash
curl -i http://localhost:3000/books/1
```

**Postman:**

* GET `http://localhost:3000/books/1`

---

### 8.4 POST create book

**curl:**

```bash
curl -i -X POST http://localhost:3000/books \
  -H "Content-Type: application/json" \
  -d '{"title":"The Alchemist","author":"Paulo Coelho"}'
```

**Postman:**

* Method: POST
* URL: `http://localhost:3000/books`
* Body → raw → JSON → `{"title":"The Alchemist","author":"Paulo Coelho"}`

**Response:** 201 Created with created object.

---

### 8.5 PUT replace book

**curl:**

```bash
curl -i -X PUT http://localhost:3000/books/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"Atomic Habits (2nd Edition)","author":"James Clear"}'
```

**Note:** PUT expects full resource (title required). If you omit fields, they’ll be replaced/overwritten.

---

### 8.6 PATCH partial update

**curl:**

```bash
curl -i -X PATCH http://localhost:3000/books/1 \
  -H "Content-Type: application/json" \
  -d '{"author":"J. Clear"}'
```

**Response:** 200 with updated object.

---

### 8.7 DELETE

**curl:**

```bash
curl -i -X DELETE http://localhost:3000/books/1
```

**Response:** 200 with message and deleted resource.

---