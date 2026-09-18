# 🧪 Testing Node.js APIs with Jest & Supertest

## 🧠 Introduction

You've built validation, security, and are about to deploy — but how do you **know** your API actually works correctly, and keeps working as you add new features? Manually clicking through Postman every time doesn't scale. This class covers **automated testing**: unit tests, integration tests, and testing a real Express + MongoDB API with **Jest** and **Supertest**.

---

## 🎯 Learning Goals

* Understand the difference between unit and integration tests
* Set up Jest with TypeScript
* Write unit tests for pure functions
* Write integration tests for Express routes using Supertest
* Mock dependencies (database calls)
* Use a real in-memory MongoDB for realistic integration tests
* Understand test coverage

---

## ❓ Why Automated Testing?

| Without Tests | With Tests |
|---|---|
| Every change requires manual Postman clicking | Run one command: `npm test` |
| Bugs discovered by users in production | Bugs caught before you even commit |
| Scary to refactor — might break something silently | Confident refactoring — tests immediately flag breakage |
| No proof the API behaves as documented | Tests double as living documentation |

---

## 🧩 Unit Tests vs Integration Tests

| | Unit Test | Integration Test |
|---|---|---|
| Tests | One isolated function/unit of logic | Multiple parts working together (e.g. a full HTTP request → controller → database) |
| Speed | Very fast | Slower (involves I/O) |
| Dependencies | Mocked/faked | Real (or realistic, e.g. in-memory DB) |
| Example | "Does `calculateDiscount(100, 10)` return `90`?" | "Does `POST /api/users/register` actually create a user in the database?" |

A healthy project uses **both** — many fast unit tests for logic, and a smaller number of integration tests for critical user flows.

---

## 🧰 Step 1: Project Setup

```bash
npm install --save-dev jest ts-jest @types/jest supertest @types/supertest
npx ts-jest config:init
```

This generates a `jest.config.js`:

```js
/** @type {import('ts-jest').JestConfigWithTsJest} */
module.exports = {
  preset: "ts-jest",
  testEnvironment: "node",
};
```

Add a test script to `package.json`:

```json
"scripts": {
  "test": "jest",
  "test:watch": "jest --watch",
  "test:coverage": "jest --coverage"
}
```

Jest automatically finds files matching `*.test.ts` or inside a `__tests__/` folder.

---

## 🧪 Step 2: Your First Unit Test

Let's test a pure function — no database, no HTTP, just logic.

```ts
// src/utils/discount.ts
export function calculateDiscountedPrice(price: number, discountPercent: number): number {
  if (discountPercent < 0 || discountPercent > 100) {
    throw new Error("Discount percent must be between 0 and 100");
  }
  return price - price * (discountPercent / 100);
}
```

```ts
// src/utils/discount.test.ts
import { calculateDiscountedPrice } from "./discount";

describe("calculateDiscountedPrice", () => {
  it("applies a 10% discount correctly", () => {
    expect(calculateDiscountedPrice(100, 10)).toBe(90);
  });

  it("returns the original price when discount is 0", () => {
    expect(calculateDiscountedPrice(200, 0)).toBe(200);
  });

  it("returns 0 when discount is 100%", () => {
    expect(calculateDiscountedPrice(50, 100)).toBe(0);
  });

  it("throws an error for a negative discount", () => {
    expect(() => calculateDiscountedPrice(100, -5)).toThrow(
      "Discount percent must be between 0 and 100"
    );
  });
});
```

Run it:

```bash
npm test
```

**Output:**

```
 PASS  src/utils/discount.test.ts
  calculateDiscountedPrice
    ✓ applies a 10% discount correctly
    ✓ returns the original price when discount is 0
    ✓ returns 0 when discount is 100%
    ✓ throws an error for a negative discount

Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
```

### 🔍 Anatomy of a Jest Test

| Function | Purpose |
|---|---|
| `describe("name", () => {...})` | Groups related tests together |
| `it(...)` / `test(...)` | Defines a single test case |
| `expect(value)` | Wraps the actual value you want to check |
| `.toBe(x)` | Strict equality (`===`) — best for primitives |
| `.toEqual(x)` | Deep equality — best for objects/arrays |
| `.toThrow(message)` | Asserts a function throws an error |
| `.toBeTruthy()` / `.toBeFalsy()` | Checks truthiness |
| `.toContain(item)` | Checks if an array/string contains something |

---

## 🌐 Step 3: Integration Testing an Express Route with Supertest

**Supertest** lets you send real HTTP requests to your Express app **in-memory** (no need to actually start a server on a port).

```ts
// src/app.ts — export the app WITHOUT calling .listen() here
import express from "express";
import userRoutes from "./user/user.routes";

const app = express();
app.use(express.json());
app.use("/api/users", userRoutes);

export default app; // exported for testing
```

```ts
// src/index.ts — the actual entry point that starts the server
import app from "./app";

app.listen(4000, () => console.log("Server running on port 4000"));
```

> 💡 **Why split `app.ts` from `index.ts`?** Tests need access to the Express `app` object to send requests to it, but they must NOT actually start a real server listening on a port (that would conflict across test runs and slow things down).

```ts
// src/user/user.routes.test.ts
import request from "supertest";
import app from "../app";

describe("GET /api/users/all", () => {
  it("requires authentication", async () => {
    const response = await request(app).get("/api/users/all");

    expect(response.status).toBe(401);
    expect(response.body.success).toBe(false);
  });
});

describe("POST /api/users/register", () => {
  it("rejects registration with an invalid email", async () => {
    const response = await request(app).post("/api/users/register").send({
      name: "Ali",
      email: "not-an-email",
      password: "Password1!",
    });

    expect(response.status).toBe(400);
    expect(response.body.success).toBe(false);
  });

  it("rejects a weak password", async () => {
    const response = await request(app).post("/api/users/register").send({
      name: "Ali",
      email: "ali@example.com",
      password: "weak",
    });

    expect(response.status).toBe(400);
  });
});
```

> 💡 This directly tests the real validation logic in this repo's `Backend/src/user/user.schema.ts` (Zod schema) and `Backend/src/helpers/auth.ts` (auth middleware) — end-to-end, exactly as a real client would experience it.

---

## 🗄️ Step 4: Testing Against a Real (In-Memory) Database

For integration tests, you don't want to pollute your real MongoDB database — instead, use **`mongodb-memory-server`**, which spins up a temporary, real MongoDB instance purely in memory for the duration of your tests.

```bash
npm install --save-dev mongodb-memory-server
```

```ts
// src/test/setup.ts
import { MongoMemoryServer } from "mongodb-memory-server";
import mongoose from "mongoose";

let mongoServer: MongoMemoryServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  await mongoose.connect(mongoServer.getUri());
});

afterEach(async () => {
  // Clean all collections between tests so tests don't affect each other
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});
```

```ts
// src/user/user.integration.test.ts
import request from "supertest";
import app from "../app";
import "../test/setup"; // runs the beforeAll/afterEach/afterAll hooks above

describe("User Registration Flow", () => {
  it("successfully registers a new user with valid data", async () => {
    const response = await request(app).post("/api/users/register").send({
      name: "Ali Khan",
      email: "ali@example.com",
      password: "SecurePass1!",
    });

    expect(response.status).toBe(201);
    expect(response.body.success).toBe(true);
    expect(response.body.data.email).toBe("ali@example.com");
  });

  it("rejects a duplicate email", async () => {
    // Register once
    await request(app).post("/api/users/register").send({
      name: "Ali Khan",
      email: "ali@example.com",
      password: "SecurePass1!",
    });

    // Try registering again with the same email
    const response = await request(app).post("/api/users/register").send({
      name: "Someone Else",
      email: "ali@example.com",
      password: "AnotherPass1!",
    });

    expect(response.status).toBe(400);
    expect(response.body.message).toMatch(/already exists/i);
  });
});
```

This test suite exercises the **real** controller logic, **real** Mongoose schema validation, and a **real** (temporary) database — giving you strong confidence the whole flow actually works, not just isolated pieces.

---

## 🎭 Step 5: Mocking (Faking Dependencies for Pure Unit Tests)

Sometimes you want to test a function's logic **without** actually hitting a database or sending a real email. Jest lets you **mock** (fake) modules.

```ts
// src/user/user.controllers.ts (simplified)
import { sendEmail } from "../config/smtp";

export async function notifyUser(email: string) {
  await sendEmail({ email, subject: "Hi", template: "<p>Hello</p>" });
  return true;
}
```

```ts
// src/user/user.controllers.test.ts
import { notifyUser } from "./user.controllers";
import * as smtp from "../config/smtp";

jest.mock("../config/smtp"); // replaces the real module with an auto-mocked version

describe("notifyUser", () => {
  it("calls sendEmail with the correct arguments", async () => {
    const sendEmailMock = smtp.sendEmail as jest.Mock;
    sendEmailMock.mockResolvedValue(undefined); // pretend it succeeded instantly

    const result = await notifyUser("ali@example.com");

    expect(sendEmailMock).toHaveBeenCalledWith(
      expect.objectContaining({ email: "ali@example.com" })
    );
    expect(result).toBe(true);
  });
});
```

> 💡 Mocking is essential here because you don't want your test suite to **actually send real emails** every time you run `npm test`!

---

## 📊 Step 6: Test Coverage

Coverage tells you **what percentage of your code is actually exercised by tests**.

```bash
npm run test:coverage
```

**Example output:**

```
--------------------|---------|----------|---------|---------|
File                | % Stmts | % Branch | % Funcs | % Lines |
--------------------|---------|----------|---------|---------|
All files           |   78.4  |   65.2   |   80.0  |   77.9  |
 user.controllers.ts|   85.0  |   70.0   |   90.0  |   84.5  |
 product.controllers|   60.0  |   45.0   |   65.0  |   59.0  |
--------------------|---------|----------|---------|---------|
```

> 💡 **100% coverage doesn't mean bug-free** — it just means every line *ran* during tests, not that every edge case was *checked*. Aim for high coverage on critical business logic (auth, payments, orders), but don't chase 100% everywhere at the cost of writing meaningless tests.

---

## 🧾 Summary

| Concept | Tool/Pattern |
|---|---|
| Test runner | Jest (`describe`, `it`, `expect`) |
| Unit tests | Test pure functions in isolation |
| Integration tests | Supertest — sends real HTTP requests to your Express `app` |
| Splitting `app.ts`/`index.ts` | Lets tests import the app without starting a real server |
| In-memory database | `mongodb-memory-server` — realistic DB tests without touching production data |
| Mocking | `jest.mock(...)` — fake external dependencies (email, payment APIs) |
| Coverage | `npm run test:coverage` — measure how much code your tests exercise |

---

## 🧩 Hands-On Practice

1. Write unit tests for a `calculateInvoiceTotal` function (from Week 2's TypeScript functions class) covering normal cases, zero items, and a discount edge case.
2. Split this repo's `Backend/src/index.ts` into `app.ts` (Express setup) and `index.ts` (the `.listen()` call).
3. Write a Supertest integration test for `GET /api/product` confirming it returns `200` and a `data` array.
4. Set up `mongodb-memory-server` and write an integration test for user registration + login (register a user, then log in with the same credentials, and confirm you receive an `accessToken`).
5. Mock the `sendEmail` function and confirm it's called exactly once when a user registers.
6. Run `npm run test:coverage` and identify the least-tested file in your project.

🎉 **This wraps up the full backend course** — from HTTP fundamentals all the way to automated testing. You're ready to build, secure, and confidently ship production Node.js APIs.
