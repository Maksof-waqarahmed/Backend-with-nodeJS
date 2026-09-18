# 📘 Role-Based Authentication (RBAC) & Email Sending

---

## 🧠 Introduction

In modern applications, **login alone is not enough**.
We also need to control:

* **Who can access what**
* What **Admins** are allowed to do
* What **normal users** are restricted from doing

This concept is called:

👉 **Role-Based Authentication & Authorization (RBAC)**

In this README, we will cover:

* Authentication vs Authorization
* JWT-based login system
* Role-based access (Admin, User, Manager, etc.)
* Protected routes
* Email sending (welcome, verification, password reset)
* MongoDB + Mongoose + TypeScript integration

---

## 🔐 Authentication vs Authorization

| Concept            | Meaning                                               |
| ------------------ | ----------------------------------------------------- |
| **Authentication** | Identifying **who the user is** (login/signup)        |
| **Authorization**  | Defining **what the user can do** (roles/permissions) |

**Example:**

* Logging in → Authentication
* Accessing admin panel → Authorization

---

## 🧩 What is Role-Based Authentication (RBAC)?

In RBAC, each user is assigned a **role**:

```text
Admin → full access
Manager → limited control
User → basic access
```

The server checks:

> “Does this user’s role have permission to perform this action?”

---

## 🏗 User Schema with Roles (MongoDB + Mongoose)

### TypeScript Interface

**File:** `types/User.ts`

```ts
export interface IUser {
  name: string;
  email: string;
  password: string;
  role: "user" | "admin";
  isVerified: boolean;
  createdAt: Date;
}
```

> 💡 No need to `extend Document` — `mongoose.Schema<IUser>` and `mongoose.model<IUser>(...)` (Mongoose 6+) infer `Document`'s properties automatically, matching this repo's own `Backend/src/user/user.model.ts`.

---

### Mongoose Schema

**File:** `models/User.ts`

```ts
import mongoose from "mongoose";
import { IUser } from "../types/User";

const userSchema = new mongoose.Schema<IUser>({
  name: {
    type: String,
    required: true,
  },

  email: {
    type: String,
    required: true,
    unique: true,
  },

  password: {
    type: String,
    required: true,
  },

  role: {
    type: String,
    enum: ["user", "admin"],
    default: "user",
  },

  isVerified: {
    type: Boolean,
    default: false,
  },

  createdAt: {
    type: Date,
    default: Date.now,
  },
});

export default mongoose.model<IUser>("User", userSchema);
```

---

## 🔑 Password Hashing (Security)

Passwords should **never be stored as plain text**.

### Install bcrypt

```bash
npm install bcrypt
```

---

### Hash Password Before Saving

```ts
import bcrypt from "bcrypt";

userSchema.pre("save", async function (next) {
  if (!this.isModified("password")) return next();

  this.password = await bcrypt.hash(this.password, 10);
  next();
});
```

---

## 🔐 JWT Authentication (Login System)

### Install JWT

```bash
npm install jsonwebtoken
```

---

### Generate JWT Token

```ts
import jwt from "jsonwebtoken";

export const generateToken = (userId: string, role: string) => {
  return jwt.sign(
    { userId, role },
    process.env.JWT_SECRET as string,
    { expiresIn: "7d" }
  );
};
```

---

### Login Route

```ts
import bcrypt from "bcrypt";
import User from "../models/User";

app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email });
  if (!user) return res.status(404).json({ message: "User not found" });

  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) return res.status(401).json({ message: "Invalid credentials" });

  const token = generateToken(user._id.toString(), user.role);

  res.json({
    token,
    user: {
      id: user._id,
      role: user.role,
    },
  });
});
```

---

## 🛡 Authentication Middleware (JWT Verification)

```ts
import jwt from "jsonwebtoken";
import { Request, Response, NextFunction } from "express";

export const authenticate = (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const token = req.headers.authorization?.split(" ")[1];

  if (!token)
    return res.status(401).json({ message: "No token provided" });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET as string);
    req.user = decoded;
    next();
  } catch {
    res.status(401).json({ message: "Invalid token" });
  }
};
```

> ⚠️ **Compile error waiting to happen:** `req.user = decoded` will fail with `Property 'user' does not exist on type 'Request'` — Express's built-in `Request` type has no `user` property by default. You must **augment** the type first via a declaration file:
>
> ```ts
> // types/express.d.ts
> import "express";
>
> declare global {
>   namespace Express {
>     interface Request {
>       user?: string | import("jsonwebtoken").JwtPayload;
>     }
>   }
> }
> ```
>
> This technique (called **module augmentation**) is covered in depth in the [TypeScript course's Week 6, Class 1](../../../TypeScript/week_6/class_1/README.md).

---

## 🎭 Role-Based Authorization Middleware

```ts
export const authorizeRoles = (...roles: string[]) => {
  return (req: any, res: Response, next: NextFunction) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ message: "Access denied" });
    }
    next();
  };
};
```

---

### Protected Routes Example

```ts
app.get(
  "/admin/dashboard",
  authenticate,
  authorizeRoles("admin"),
  (req, res) => {
    res.json({ message: "Welcome Admin" });
  }
);
```

✔ Role = admin → Access allowed
❌ Role = user → Access denied

---

## 📧 Email Sending (NodeMailer)

Emails are commonly used for:

* Welcome emails
* Email verification
* Password reset

---

### Install Nodemailer

```bash
npm install nodemailer
```

---

### Email Configuration

```ts
import nodemailer from "nodemailer";

export const transporter = nodemailer.createTransport({
  service: "gmail",
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS,
  },
});
```

`.env` file:

```env
EMAIL_USER=yourgmail@gmail.com
EMAIL_PASS=app_password_here
```

---

### Send Welcome Email

```ts
import { transporter } from "../utils/mailer";

export const sendWelcomeEmail = async (email: string, name: string) => {
  await transporter.sendMail({
    from: process.env.EMAIL_USER,
    to: email,
    subject: "Welcome to Our Platform 🎉",
    html: `
      <h2>Hello ${name}</h2>
      <p>Welcome to our platform. We're glad to have you!</p>
    `,
  });
};
```

---

### Call Email After Signup

```ts
app.post("/register", async (req, res) => {
  const user = await User.create(req.body);

  await sendWelcomeEmail(user.email, user.name);

  res.status(201).json({ message: "User registered successfully" });
});
```

---

## 🔁 Email Verification (Advanced)

**Steps:**

1. Generate verification token
2. Send verification link via email
3. Verify user when link is clicked

```ts
const verifyToken = jwt.sign(
  { userId: user._id },
  process.env.JWT_SECRET!,
  { expiresIn: "1d" }
);
```

Email link:

```html
<a href="https://yourapp.com/verify/${verifyToken}">
Verify Email
</a>
```

---

## ✅ Best Practices

* Use **JWT expiration**
* Store **roles inside JWT**
* Always protect admin routes
* Never expose passwords
* Use **TypeScript types everywhere**
* Keep email credentials inside `.env`

---

## 🏁 Summary

This README demonstrates:

✔ JWT-based Authentication
✔ Role-Based Authorization (RBAC)
✔ Secure password hashing
✔ Admin/User role separation
✔ Email sending (Welcome & Verification)
✔ MongoDB + Mongoose + TypeScript integration

This system is used in **real-world production applications** such as:

* Admin dashboards
* SaaS platforms
* E-commerce systems
* Learning management systems