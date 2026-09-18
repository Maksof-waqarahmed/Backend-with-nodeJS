# 📘 MongoDB with Express.js

---

## 🧠 Introduction to MongoDB: NoSQL Databases and Document-Based Storage

**MongoDB** is a **NoSQL (non-relational), document-oriented database**.

Unlike SQL databases:

* Data is stored as **documents** (JSON-like objects)
* Each document can have **different fields** (schema-less)
* Data is organized into **collections**, not tables

**Advantages of MongoDB:**

* Flexible schema for dynamic applications
* High performance for large-scale apps
* Horizontal scalability (sharding)
* Supports complex queries and aggregation

**Example Document:**

```json
{
  "_id": "64f567c9e87d95",
  "name": "Waqar Rana",
  "role": "Software Engineer",
  "skills": ["JavaScript", "ReactJS", "Node.js", "TypeScript"],
  "experience": 2
}
```

---

## ⚙️ Why MongoDB?

| Feature                    | Explanation                                            |
| -------------------------- | ------------------------------------------------------ |
| **Schema-less**            | Documents can have different fields                    |
| **JSON-like Documents**    | Data stored as BSON (Binary JSON)                      |
| **High Scalability**       | Horizontal scaling via sharding                        |
| **Fast Performance**       | Indexes & in-memory operations speed up queries        |
| **Flexible Relationships** | Embedding or referencing documents                     |
| **Cloud-native**           | Works well with MongoDB Atlas or other cloud platforms |

---

## 🧱 SQL vs MongoDB

| Concept       | SQL           | MongoDB                     |
| ------------- | ------------- | --------------------------- |
| Database Type | Relational    | Non-relational              |
| Data Format   | Tables & Rows | JSON-like Documents         |
| Schema        | Fixed Schema  | Dynamic Schema              |
| Relationships | JOINs         | Embedding or Referencing    |
| Scalability   | Vertical      | Horizontal (Sharding)       |
| Transactions  | ACID          | Multi-document ACID (v4.0+) |

---

## 🛠 Setting Up MongoDB: Installation and Basic Configuration

### 1️⃣ MongoDB Atlas (Cloud)

1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) and sign up.
2. Create a free cluster (M0 tier).
3. Add a database user with read/write permissions.
4. Allow network access (0.0.0.0/0 for testing).
5. Copy the connection URI:

```text
mongodb+srv://username:password@cluster.mongodb.net/myDatabase?retryWrites=true&w=majority
```

### 2️⃣ Local MongoDB Installation

1. Download MongoDB Community Edition: [MongoDB Download](https://www.mongodb.com/try/download/community)
2. Install MongoDB and start the `mongod` service.
3. Use **Mongo Shell** or **MongoDB Compass** to manage your database.

---

## ⚡ Using MongoDB with Express.js

### Step 1 — Install Dependencies

```bash
npm install express mongoose dotenv
```

* `express` → Node.js framework
* `mongoose` → ODM (Object Data Modeling)
* `dotenv` → Manage environment variables

### Step 2 — Environment Variables

Create `.env`:

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/myDatabase
SECRET_KEY=mySecret123
```

> Keeps sensitive data secure and configurable per environment.

---

### Step 3 — Connecting Express.js to MongoDB

**File:** `server.ts`

```ts
import express, { Request, Response } from "express";
import mongoose from "mongoose";
import dotenv from "dotenv";

dotenv.config();

const app = express();
app.use(express.json());

mongoose.connect(process.env.MONGO_URI as string)
  .then(() => console.log("MongoDB connected successfully"))
  .catch(err => console.error("MongoDB connection error:", err));

app.get("/", (req: Request, res: Response) => res.send("Hello MongoDB with Express!"));

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

> ⚠️ **Note:** `process.env.MONGO_URI` has the type `string | undefined` in TypeScript's strict mode, so it must be cast with `as string` (or validated first) before passing it to `mongoose.connect()`. Also, older tutorials show `useNewUrlParser`/`useUnifiedTopology` options — these are **deprecated and unnecessary** since Mongoose 6+ (this course uses Mongoose 9) and will cause a TypeScript error if included.

---

## 🧱 Data Modeling and Schema Design with Mongoose

### Mongoose Schema Example

**File:** `models/User.ts`

```ts
import mongoose from "mongoose";

const userSchema = new mongoose.Schema(
  {
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    role: { type: String, default: "user" },
    skills: [String],
    experience: { type: Number, min: 0 }
  },
  {
    timestamps: true
  }
);

export default mongoose.model("User", userSchema);
```

**Schema Types in Mongoose:**

| Type         | Use                    |
| ------------ | ---------------------- |
| `String`     | Name, email, title     |
| `Number`     | Age, price, experience |
| `Boolean`    | true / false           |
| `Date`       | createdAt, updatedAt   |
| `Buffer`     | Binary data (rare)     |
| `ObjectId`   | Reference (relations)  |
| `Mixed`      | Any type of data       |
| `Array`      | List of values         |
| `Map`        | Key-value pairs        |
| `Decimal128` | High precision numbers |

---

### 🔹 Examples 👇

```js
name: String
age: Number
isActive: Boolean
createdAt: Date
userId: mongoose.Schema.Types.ObjectId
skills: [String]
meta: mongoose.Schema.Types.Mixed
```

---

**Schema Attributes / Validators in Mongoose:**

| Attribute                 | Purpose              |
| ------------------------- | -------------------- |
| `type`                    | Data type            |
| `required`                | Mandatory field      |
| `default`                 | Default value        |
| `unique`                  | Unique value         |
| `min` / `max`             | Number limits        |
| `minlength` / `maxlength` | String length        |
| `enum`                    | Allowed values       |
| `match`                   | Regex validation     |
| `trim`                    | Remove spaces        |
| `lowercase`               | Convert to lowercase |
| `uppercase`               | Convert to uppercase |
| `select`                  | Hide/show field      |
| `index`                   | Fast searching       |

---

### 🔸 Attribute Example

```js
email: {
  type: String,
  required: true,
  unique: true,
  lowercase: true,
  trim: true,
  match: /^[^\s@]+@[^\s@]+\.[^\s@]+$/
}
```

---

---

## 3️⃣ Advanced Schema Features 🚀

### 🔹 Nested Object

```js
address: {
  city: String,
  country: String
}
```

### 🔹 Array of Objects

```js
projects: [
  {
    title: String,
    duration: Number
  }
]
```

### 🔹 References (Relations)

```js
roleId: {
  type: mongoose.Schema.Types.ObjectId,
  ref: "Role"
}
```

---

## 4️⃣ Schema-level Options (second argument)

```js
new mongoose.Schema({}, {
  timestamps: true,
  collection: "users",
  versionKey: false
})
```

| Option       | Kaam                       |
| ------------ | -------------------------- |
| `timestamps` | createdAt, updatedAt       |
| `collection` | Custom collection name     |
| `versionKey` | __v disable                |
| `strict`     | Extra fields allow / block |

---

### 🔗 Official Mongoose Documentation

👉 **Schema Types**
[https://mongoosejs.com/docs/schematypes.html](https://mongoosejs.com/docs/schematypes.html)

👉 **Schema Options**
[https://mongoosejs.com/docs/guide.html](https://mongoosejs.com/docs/guide.html)

👉 **Validators**
[https://mongoosejs.com/docs/validation.html](https://mongoosejs.com/docs/validation.html)

👉 **Population / References**
[https://mongoosejs.com/docs/populate.html](https://mongoosejs.com/docs/populate.html)

---

## 📄 Defining TypeScript Interfaces for Mongoose Schemas

**File:** `types/User.ts`

```typescript
export interface IUser {
  name: string;
  email: string;
  role?: "user" | "admin";
  skills?: string[];
  experience?: number;
  createdAt?: Date;
  updatedAt?: Date;
}
```

**Usage with Mongoose:**

```typescript
import mongoose from "mongoose";
import { IUser } from "../types/User";

const userSchema = new mongoose.Schema<IUser>(
  {
    name: { type: String, required: true, trim: true },
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true
    },
    role: {
      type: String,
      enum: ["user", "admin"],
      default: "user"
    },
    skills: [String],
    experience: { type: Number, min: 0 }
  },
  {
    timestamps: true
  }
)

export const User = mongoose.model<IUser>("User", userSchema)
```

> ✅ This ensures **type safety** in TypeScript projects.
>
> ⚠️ **Note:** Older Mongoose + TypeScript tutorials have your interface `extend Document` (e.g. `interface IUser extends Document`). Since Mongoose 6+ (this course uses Mongoose 9), this is **no longer necessary** — `mongoose.Schema<IUser>` and `mongoose.model<IUser>(...)` already infer all of `Document`'s properties (`_id`, `.save()`, etc.) automatically. This is exactly the pattern used in this repo's own `Backend/src/user/user.model.ts`.

---

## 🏗 CRUD Operations in MongoDB

### 1️⃣ Create (Insert)

```ts
import { Request, Response } from "express";
import { User } from "./models/User";

app.post("/users", async (req: Request, res: Response) => {
  try {
    const newUser = new User(req.body);
    const savedUser = await newUser.save();
    res.status(201).json(savedUser);
  } catch (error) {
    res.status(400).json({ message: "Error creating user: " + error });
  }
});
```

---

### 2️⃣ Read (Find)

**Get all users:**

```ts
app.get("/users", async (req: Request, res: Response) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (error) {
    res.status(500).json({ message: "Error fetching users: " + error });
  }
});
```

**Get user by ID:**

```ts
app.get("/users/:id", async (req: Request, res: Response) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ message: "User not found" });
    res.json(user);
  } catch (error) {
    res.status(500).json({ message: "Error fetching user: " + error });
  }
});
```

---

### 3️⃣ Update

```ts
app.put("/users/:id", async (req: Request, res: Response) => {
  try {
    const updatedUser = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );
    if (!updatedUser) return res.status(404).json({ message: "User not found" });
    res.json(updatedUser);
  } catch (error) {
    res.status(400).json({ message: "Error updating user: " + error });
  }
});
```

---

### 4️⃣ Delete

```ts
app.delete("/users/:id", async (req: Request, res: Response) => {
  try {
    const deletedUser = await User.findByIdAndDelete(req.params.id);
    if (!deletedUser) return res.status(404).json({ message: "User not found" });
    res.json({ message: "User deleted successfully" });
  } catch (error) {
    res.status(500).json({ message: "Error deleting user: " + error });
  }
});
```

> 💡 **Why `"..." + error` instead of `error.message`?** In TypeScript's strict mode, a `catch` block's error variable has the type `unknown` by default — you can't access `.message` on it directly without first narrowing its type (e.g. `error instanceof Error`). String-concatenating it (as this repo's own controllers do throughout `Backend/src/`) sidesteps that without extra boilerplate.

---

## 🧩 Advanced Data Modeling Concepts

### 1️⃣ Embedding (Nested Documents)

```json
{
  "name": "Waqar",
  "email": "waqar@example.com",
  "address": {
    "city": "Karachi",
    "zip": "75500"
  }
}
```

✅ Pros: Faster reads, no joins required
🚫 Cons: Document size limit (16MB)

---

### 2️⃣ Referencing (Normalization)

```json
// Users
{ "_id": 1, "name": "Waqar" }

// Posts
{ "title": "Hello MongoDB", "author_id": 1 }
```

✅ Pros: Better for large datasets
🚫 Cons: Requires multiple queries

---

## ⚡ Indexing

```javascript
db.users.createIndex({ name: 1 });
```

* `1` → ascending
* `-1` → descending

Improves performance for **search queries**.

---

## 📦 BSON (Binary JSON)

MongoDB stores documents in **BSON**, extending JSON with:

* `_id` → unique identifier
* `Date` → timestamp
* `Binary` → files or binary data

```json
{
  "_id": ObjectId("64f567c9e87d95"),
  "name": "Waqar Rana",
  "createdAt": ISODate("2025-11-02T12:00:00Z")
}
```

---

## ✅ Best Practices

1. Use `.env` for credentials.
2. Index frequently queried fields.
3. Use Mongoose validation.
4. Choose embedding vs referencing wisely.
5. Avoid storing large files directly (use GridFS/S3).
6. Use **TypeScript interfaces** for type safety.

---