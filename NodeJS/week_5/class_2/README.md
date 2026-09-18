# 🔐 Authentication & Authorization (Complete Guide)

## 📌 1. What is Authentication?

**Authentication** is the process of **verifying who a user is**.

> ❓ “Are you really who you claim to be?”

### Examples:

* Logging in with **email & password**
* Logging in with **Google / GitHub**
* Using **OTP**
* Using **biometrics**

### Real-life example:

Showing your **CNIC** to prove your identity.

---

## 📌 2. What is Authorization?

**Authorization** is the process of **deciding what a user can access**.

> ❓ “What are you allowed to do?”

### Examples:

* Admin can delete users
* User can only view profile
* Teacher can upload content
* Student can only view content

### Real-life example:

You may enter a building (authenticated),
but only some rooms are allowed (authorized).

---

## 🔁 Authentication vs Authorization

| Feature      | Authentication | Authorization       |
| ------------ | -------------- | ------------------- |
| Purpose      | Who are you?   | What can you do?    |
| Happens when | Login          | After login         |
| Example      | Login success  | Admin-only access   |
| Depends on   | Credentials    | Roles / permissions |

---

<img src="./images/1.png" alt="Authentication vs Authorization">
---

## 🔑 Authentication Methods (Types)

### 1️⃣ Username & Password

* Most common
* Password stored as **hashed**
* Example: bcrypt, argon2

```ts
bcrypt.compare(password, hashedPassword)
```

---

### 2️⃣ OAuth (Google, GitHub)

* Third-party login
* User trusts Google
* App never sees password

✔ Secure
✔ Fast
✔ Scalable

---

### 3️⃣ OTP (One Time Password)

* Sent via SMS / Email
* Short-lived
* Used in banking apps

---

### 4️⃣ Token-based Authentication

* JWT
* Access token
* Refresh token

---

## 🍪 Sessions (Stateful Authentication)

### 📌 What is a Session?

A **session** stores user data on the **server**.

### Flow:

1. User logs in
2. Server creates session
3. Session ID stored in **cookie**
4. Cookie sent with every request
5. Server verifies session

---

### 📦 Where is data stored?

| Location | Stored Data         |
| -------- | ------------------- |
| Server   | Session data        |
| Client   | Session ID (cookie) |

---

### Example:

```txt
Browser → sends cookie
Server → finds session → allows access
```

---

### Pros:

✔ Secure
✔ Easy to revoke
✔ Good for traditional apps

### Cons:

❌ Not scalable
❌ Server memory heavy
❌ Hard with mobile apps

---

## 🔐 JWT (JSON Web Token) – Stateless Authentication

### What is JWT?

JWT is a **self-contained token** used for authentication.

It has **3 parts**:

```txt
HEADER.PAYLOAD.SIGNATURE
```

---

### JWT Structure

```json
Header: {
  "alg": "HS256",
  "typ": "JWT"
}

Payload: {
  "id": "123",
  "role": "admin"
}

Signature:
HMACSHA256(base64(header + payload), SECRET_KEY)
```

---

### How JWT Works

1. User logs in
2. Server generates JWT
3. JWT sent to client
4. Client stores JWT
5. Client sends JWT in headers
6. Server verifies signature

```http
Authorization: Bearer <token>
```

---

### Where is JWT Stored?

| Storage         | Safe? | Notes       |
| --------------- | ----- | ----------- |
| localStorage    | ❌     | XSS risk    |
| sessionStorage  | ❌     | XSS risk    |
| httpOnly cookie | ✅     | Recommended |

---

### Example: JWT Creation

```ts
jwt.sign({ id: user._id }, process.env.JWT_SECRET, {
  expiresIn: "1h"
});
```

---

### JWT Verification

```ts
jwt.verify(token, process.env.JWT_SECRET);
```

---

### JWT Pros:

✔ Stateless
✔ Scalable
✔ Fast
✔ Good for APIs & mobile apps

### JWT Cons:

❌ Hard to revoke
❌ Token leakage risk
❌ Requires refresh strategy

---

## 🔄 Access Token & Refresh Token

### Why?

Short-lived tokens increase security.

| Token         | Expiry | Use                  |
| ------------- | ------ | -------------------- |
| Access Token  | 15 min | API access           |
| Refresh Token | Days   | Get new access token |

---

### Flow:

1. Access token expires
2. Client sends refresh token
3. Server issues new access token

---

## 🔐 Authorization (Roles & Permissions)

### Role-based Authorization (RBAC)

```ts
if (user.role !== "admin") {
  return res.status(403).json({ message: "Forbidden" });
}
```

### Common Roles:

* admin
* user
* moderator
* teacher
* student

---

## 🔥 Common Security Threats

| Threat      | Prevention       |
| ----------- | ---------------- |
| XSS         | httpOnly cookies |
| CSRF        | sameSite cookies |
| Token theft | Short expiry     |
| Brute force | Rate limiting    |

---

# 🔐 Authentication & Security in Express

This guide covers:

1. Password hashing with **bcrypt**
2. Security best practices
3. JWT generation & verification with **TypeScript**
4. Refresh token implementation
5. Authentication & Authorization in **Express**
6. Error handling & middleware chaining

---

## 📌 1. Password Hashing with bcrypt

### ❓ Why Hash Passwords?

❌ Never store passwords in plain text
✔ If DB is leaked, hashed passwords protect users

---

## 🔑 What is bcrypt?

**bcrypt** is a password-hashing algorithm that:

* Uses **salt**
* Is **slow by design** (prevents brute-force attacks)
* Automatically handles salt generation

---

## 📦 Install bcrypt

```bash
npm install bcrypt
npm install --save-dev @types/bcrypt
```

---

## 🔐 Hashing a Password (Signup)

```ts
import bcrypt from "bcrypt";

const saltRounds = 10;

const hashedPassword = await bcrypt.hash(password, saltRounds);
```

### What happens internally?

1. Random **salt** is generated
2. Password + salt → hashed
3. Stored safely in DB

---

## 🔍 Verifying Password (Login)

```ts
const isMatch = await bcrypt.compare(password, user.password);

if (!isMatch) {
  throw new Error("Invalid credentials");
}
```

✔ bcrypt automatically extracts salt
✔ Secure comparison

---

## 🔒 Password Security Best Practices

✔ Minimum 8–12 characters
✔ Use bcrypt or argon2
✔ Never log passwords
✔ Rate-limit login attempts
✔ Enforce strong password rules
✔ Hash again if algorithm changes

---

## 📌 2. JWT (JSON Web Token) with TypeScript

### ❓ What is JWT?

JWT is a **stateless authentication token**.

It contains:

```
HEADER.PAYLOAD.SIGNATURE
```

---

## 📦 Install JWT

```bash
npm install jsonwebtoken
npm install --save-dev @types/jsonwebtoken
```

---

## 🔐 Generate JWT (Login)

```ts
import jwt from "jsonwebtoken";

const accessToken = jwt.sign(
  { id: user._id, role: user.role },
  process.env.JWT_SECRET as string,
  { expiresIn: "15m" }
);
```

### Why short expiry?

✔ Limits damage if token leaks

---

## 🔍 Verify JWT (Middleware)

```ts
const verifyToken = (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader)
    return res.status(401).json({ message: "No token provided" });

  const token = authHeader.split(" ")[1];

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET as string);
    req.user = decoded;
    next();
  } catch {
    res.status(403).json({ message: "Invalid or expired token" });
  }
};
```

---

## 📌 3. Refresh Token Implementation

### ❓ Why Refresh Tokens?

Access tokens expire quickly
Refresh tokens allow getting new access tokens **without login**

---

## 🔁 Token Strategy

| Token         | Lifetime  | Stored Where         |
| ------------- | --------- | -------------------- |
| Access Token  | 15 min    | Memory / cookie      |
| Refresh Token | 7–30 days | httpOnly cookie / DB |

---

## 🔐 Generate Refresh Token

```ts
const refreshToken = jwt.sign(
  { id: user._id },
  process.env.REFRESH_SECRET as string,
  { expiresIn: "7d" }
);
```

---

## 🍪 Store Refresh Token Securely

```ts
res.cookie("refreshToken", refreshToken, {
  httpOnly: true,
  secure: true,
  sameSite: "strict"
});
```

---

## 🔄 Refresh Token Endpoint

```ts
import { Request, Response } from "express";
import jwt, { JwtPayload } from "jsonwebtoken";

app.post("/refresh", (req: Request, res: Response) => {
  const token = req.cookies?.refreshToken;

  if (!token)
    return res.status(401).json({ message: "No refresh token" });

  jwt.verify(token, process.env.REFRESH_SECRET as string, (err, decoded) => {
    if (err || !decoded) return res.status(403).json({ message: "Invalid refresh token" });

    // `decoded` is typed as `string | JwtPayload` — narrow it before accessing `.id`
    const payload = decoded as JwtPayload;

    const newAccessToken = jwt.sign(
      { id: payload.id },
      process.env.JWT_SECRET as string,
      { expiresIn: "15m" }
    );

    res.json({ accessToken: newAccessToken });
  });
});
```

> ⚠️ **Two gotchas to remember:**
> 1. `req.cookies` requires the `cookie-parser` middleware (`npm install cookie-parser` + `app.use(cookieParser())`) — it doesn't exist on `req` by default.
> 2. `jwt.verify`'s callback gives you `decoded: string | JwtPayload | undefined`, so accessing `decoded.id` directly is a TypeScript error — cast it to `JwtPayload` first (as shown above) after confirming it isn't `undefined`.

---

## 📌 4. Middleware Chaining Explained

### ❓ What is Middleware?

Middleware is a function that runs:
➡ Before controller
➡ After request
➡ Before response

---

### Middleware Flow

```txt
Request
 ↓
Middleware 1
 ↓
Middleware 2
 ↓
Controller
 ↓
Response
```

---

### Example Chain

```ts
app.get(
  "/profile",
  verifyToken,
  authenticate,
  (req, res) => {
    res.json(req.user);
  }
);
```

---

## 🔥 Common Security Best Practices

✔ Use HTTPS
✔ Hash passwords
✔ Use httpOnly cookies
✔ Short token expiry
✔ Refresh token rotation
✔ Rate limiting
✔ Input validation (Zod)
✔ Do not expose stack traces

---

## 🧾 Summary Table

| Feature       | Purpose                  |
| ------------- | ------------------------ |
| bcrypt        | Secure password hashing  |
| JWT           | Stateless authentication |
| Refresh Token | Long-term login          |
| Middleware    | Auth & authorization     |
| Error Handler | Centralized errors       |

---