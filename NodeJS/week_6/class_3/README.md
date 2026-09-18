# ⚡ Caching Strategies with Redis

## 🧠 Introduction

Every database query costs time — reading from disk (or even from a remote MongoDB Atlas cluster over the network) is **orders of magnitude slower** than reading from memory. **Redis** is an in-memory data store used almost universally alongside Node.js backends to **cache** frequently-requested data, dramatically speeding up APIs and reducing database load.

This is the "Caching Strategies with Redis" topic promised in this course's outline — covering the **cache-aside pattern**, **cache invalidation**, and practical Express + Redis integration.

---

## 🎯 Learning Goals

* Understand what Redis is and why it's fast
* Set up Redis locally and connect it to a Node.js app
* Implement the **cache-aside** pattern
* Understand **TTL (Time To Live)** and cache expiration
* Understand **cache invalidation** strategies
* Use Redis for rate limiting (bonus real-world use case)

---

## ❓ What Is Redis?

**Redis** (REmote DIctionary Server) is an **in-memory key-value data store**. Unlike MongoDB, which persists data to disk, Redis keeps data primarily in **RAM** — making reads and writes extremely fast (sub-millisecond).

| | MongoDB | Redis |
|---|---|---|
| Storage | Disk-based (with memory caching) | Primarily in-memory (RAM) |
| Speed | Fast, but slower than RAM | Extremely fast |
| Data model | Documents (JSON-like) | Simple key-value pairs (+ lists, sets, hashes, sorted sets) |
| Best for | Primary, durable data storage | Caching, sessions, rate limiting, pub/sub, leaderboards |
| Persistence | Always persisted | Optional (can persist to disk, but optimized for speed, not durability) |

> 💡 **Redis is not a replacement for MongoDB** — it's a companion. MongoDB remains your permanent "source of truth"; Redis is a fast, temporary layer in front of it.

---

## 🖥️ Setting Up Redis

### Option 1: Local Installation (via Docker — easiest cross-platform option)

```bash
docker run --name redis-server -p 6379:6379 -d redis
```

### Option 2: Cloud (Redis Cloud — free tier available)

1. Sign up at [Redis Cloud](https://redis.io/try-free/)
2. Create a free database
3. Copy the connection URL (e.g. `redis://default:password@host:port`)

### Install the Node.js Redis Client

```bash
npm install redis
npm install --save-dev @types/redis
```

### Connecting to Redis in TypeScript

```ts
// src/config/redis.ts
import { createClient } from "redis";

export const redisClient = createClient({
  url: process.env.REDIS_URL || "redis://localhost:6379",
});

redisClient.on("error", (err) => console.error("Redis Client Error", err));

export async function connectRedis() {
  await redisClient.connect();
  console.log("✅ Redis connected");
}
```

```ts
// index.ts
import { connectRedis } from "./config/redis";

connectRedis();
```

---

## 🔑 Basic Redis Commands (via the Node.js Client)

```ts
// Store a value
await redisClient.set("greeting", "Hello Redis!");

// Retrieve a value
const value = await redisClient.get("greeting");
console.log(value); // "Hello Redis!"

// Store with expiration (TTL) — auto-deletes after N seconds
await redisClient.set("otp:1234", "5678", { EX: 300 }); // expires in 5 minutes

// Delete a key
await redisClient.del("greeting");

// Check if a key exists
const exists = await redisClient.exists("greeting"); // 1 or 0

// Store objects — Redis only stores strings, so JSON.stringify first
await redisClient.set("user:1", JSON.stringify({ id: 1, name: "Ali" }));
const userStr = await redisClient.get("user:1");
const user = userStr ? JSON.parse(userStr) : null;
```

---

## 🎯 The Cache-Aside Pattern (Most Common Strategy)

**Cache-aside** (a.k.a. "lazy loading") is the most widely used caching pattern:

```
1. Request comes in for data (e.g. GET /products)
2. Check the CACHE first
   ├── Cache HIT  → return cached data immediately (fast! no DB query)
   └── Cache MISS → query the DATABASE
                     → store the result in the cache (for next time)
                     → return the data
```

```
   ┌────────┐   1. Check cache    ┌────────┐
   │ Client │ ───────────────────>│  Redis │
   └────────┘                      └───┬────┘
                                        │
                          2. Cache MISS │  2. Cache HIT
                                        ▼         │
                                 ┌───────────┐    │
                                 │  MongoDB  │    │
                                 └─────┬─────┘    │
                                       │           │
                          3. Store in cache        │
                                       ▼           ▼
                                 ┌──────────────────┐
                                 │  Return to Client  │
                                 └──────────────────┘
```

### Implementation Example: Caching a Product List

```ts
import { Request, Response } from "express";
import Product from "../product/product.model";
import { redisClient } from "../config/redis";

const CACHE_KEY = "products:all";
const CACHE_TTL_SECONDS = 60; // cache expires after 1 minute

export const getAllProducts = async (req: Request, res: Response) => {
  try {
    // 1️⃣ Check the cache first
    const cached = await redisClient.get(CACHE_KEY);

    if (cached) {
      console.log("✅ Cache HIT");
      return res.status(200).json({
        success: true,
        source: "cache",
        data: JSON.parse(cached),
      });
    }

    // 2️⃣ Cache MISS — fetch from MongoDB
    console.log("❌ Cache MISS — querying database");
    const products = await Product.find({ isActive: true });

    // 3️⃣ Store the result in the cache for next time
    await redisClient.set(CACHE_KEY, JSON.stringify(products), {
      EX: CACHE_TTL_SECONDS,
    });

    return res.status(200).json({
      success: true,
      source: "database",
      data: products,
    });
  } catch (error) {
    return res.status(500).json({ success: false, message: "Internal server error " + error });
  }
};
```

> 💡 **Real impact:** The first request queries MongoDB (slower). Every request in the next 60 seconds is served instantly from Redis (much faster), without touching the database at all — until the cache expires.

---

## ⏰ TTL (Time To Live) — Why Caches Must Expire

If cached data never expired, users would keep seeing **stale (outdated) data** forever, even after the underlying database changes. TTL solves this by automatically deleting a cached key after N seconds.

```ts
await redisClient.set("products:all", JSON.stringify(products), {
  EX: 60, // expires automatically after 60 seconds
});
```

| Data Type | Typical TTL |
|---|---|
| Product listings | 30s – 5 min |
| User session data | Minutes to hours |
| Rarely-changing config | Hours to a day |
| OTP codes | 1–5 minutes |

---

## 🧹 Cache Invalidation — "The Hardest Problem in Computer Science"

There's a famous programming joke: *"There are only two hard things in Computer Science: cache invalidation and naming things."* TTL alone isn't always enough — if a product's price changes RIGHT NOW, you don't want users seeing the old cached price for the next 60 seconds.

### Strategy: Invalidate (Delete) the Cache on Write

Whenever data changes (create/update/delete), **manually delete** the related cache key so the **next** read is forced to fetch fresh data from the database.

```ts
export const updateProduct = async (req: Request, res: Response) => {
  try {
    const updated = await Product.findByIdAndUpdate(req.params.id, req.body, { new: true });

    if (!updated) {
      return res.status(404).json({ success: false, message: "Product not found" });
    }

    // 🧹 Invalidate the cache — force fresh data on next read
    await redisClient.del("products:all");

    return res.status(200).json({ success: true, data: updated });
  } catch (error) {
    return res.status(500).json({ success: false, message: "Internal server error " + error });
  }
};
```

> 💡 **This same pattern should be applied to `createProduct` and `deleteProduct`** in this repo's `Backend/src/product/product.controllers.ts` — any write operation that changes product data should delete the `products:all` cache key.

### Common Invalidation Strategies

| Strategy | How It Works | Trade-off |
|---|---|---|
| **TTL only** | Cache auto-expires after N seconds | Simple, but data can be stale for up to N seconds |
| **Delete-on-write** | Explicitly delete the cache key whenever the data changes | Always fresh, but requires remembering to invalidate everywhere data changes |
| **Write-through** | Update the cache AND the database together, in the same operation | Cache is always fresh, but adds complexity to every write |

---

## 🔒 Bonus Real-World Use Case: Rate Limiting with Redis

Redis's speed and atomic operations make it perfect for tracking "how many requests has this user made in the last minute?" — used by the `express-rate-limit` package (covered in Week 8) under the hood when configured with a Redis store, so limits work correctly even across multiple server instances.

```ts
async function isRateLimited(userIp: string): Promise<boolean> {
  const key = `rate-limit:${userIp}`;
  const requests = await redisClient.incr(key); // increments and returns the new count

  if (requests === 1) {
    await redisClient.expire(key, 60); // set a 60-second window on the first request
  }

  return requests > 100; // block after 100 requests per minute
}
```

---

## ✅ Best Practices

| Practice | Why |
|---|---|
| Always set a **TTL** | Prevents permanently stale data if you forget to invalidate |
| Use **descriptive, namespaced keys** | `products:all`, `user:123:profile` — avoids key collisions |
| Invalidate on write | Keeps cache accurate after create/update/delete |
| Cache expensive, frequently-read data | Product lists, dashboards, aggregation results — not data that changes every second |
| Don't cache user-specific sensitive data carelessly | Be mindful of what's shared across users vs. per-user |
| Monitor cache hit/miss ratio | Tells you if caching is actually helping |

---

## 🧾 Summary

| Concept | Key Idea |
|---|---|
| Redis | Fast, in-memory key-value store, used alongside (not instead of) MongoDB |
| Cache-aside pattern | Check cache first → on miss, query DB → store result in cache |
| TTL | Automatic cache expiration to prevent permanently stale data |
| Cache invalidation | Manually deleting cache keys when underlying data changes |
| Rate limiting | Another common Redis use case, using atomic counters |

---

## 🧩 Hands-On Practice

1. Run Redis locally with Docker and connect to it from a small Express app.
2. Implement cache-aside caching for this repo's `GET /api/product` endpoint (`Backend/src/product/product.controllers.ts`), with a 60-second TTL.
3. Add cache invalidation (`redisClient.del(...)`) to the `createProduct`, `updateProduct`, and `deleteProduct` controllers.
4. Add a `source: "cache" | "database"` field to your API response (like the example above) and verify in Postman that the second request within 60 seconds returns `"cache"`.
5. Implement a simple Redis-based rate limiter middleware that blocks an IP after 10 requests within 60 seconds.

Next class (Week 7): **File Uploads with Multer & Cloudinary.** 🚀
