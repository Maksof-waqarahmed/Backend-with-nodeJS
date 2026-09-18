# 📤 File Uploads in Backend

---

## 🧠 Introduction

File uploading is a **very important backend feature**.
Almost every real-world application needs file uploads, for example:

* Profile pictures
* Product images
* Documents (PDF, CV, Resume)
* Certificates
* Blog images

---

## 🌐 What is Multipart Form Data?

Normally, backend APIs accept **JSON data**.
But JSON **cannot send files**.

When we want to send:

* Text data + Files

We use:

👉 **multipart/form-data**

### Example (Frontend → Backend)

```
name = "Ali"
email = "ali@gmail.com"
profileImage = image.jpg
```

This data is sent in **multiple parts**.
That is why it is called **multipart**.

📌 **Multer** is used to handle this multipart data in backend.

---

## 📦 What is Multer?

**Multer** is an **Express middleware** that handles file uploads.

Express by default:
❌ Does NOT understand files

Multer:
✅ Reads files
✅ Processes them
✅ Saves them (or keeps them in memory)

---

## 🧠 Real-Life Example

Think like this:

1. User fills a form
2. User selects a file (image, PDF, CV)
3. Browser sends file as multipart/form-data

👉 Express gets confused 😵
👉 Multer becomes a **translator** 🧑‍🏫

---

## 🔄 Request Flow

```
Frontend (File)
   ↓
Multer (Reads & processes file)
   ↓
Express Route
   ↓
Server / Database / Cloud
```

---

## ⚙️ What Multer Does

* Receives files
* Renames files
* Saves files to folder
* Limits file size
* Checks file type
* Creates `req.file` or `req.files`

---

## ❌ Without Multer

```ts
app.post("/upload", (req, res) => {
  console.log(req.body); // ❌ file not available
});
```

👉 File is missing 😬

---

## ✅ With Multer

```ts
app.post("/upload", upload.single("file"), (req, res) => {
  console.log(req.file); // ✅ file available
});
```

👉 File received successfully 🎉

---

## 🗂 What is inside `req.file`?

```ts
{
  fieldname: 'file',
  originalname: 'photo.png',
  mimetype: 'image/png',
  destination: 'uploads/',
  filename: '170000000.png',
  path: 'uploads/170000000.png',
  size: 24567
}
```

---

## 🧩 Multer Upload Methods

### 1️⃣ Single File

```ts
upload.single("file")
```

---

### 2️⃣ Multiple Files (same field)

```ts
upload.array("files", 5)
```

---

### 3️⃣ Multiple Fields

```ts
upload.fields([
  { name: "image", maxCount: 1 },
  { name: "cv", maxCount: 1 }
])
```

---

## 🧠 Multer Storage Types

Multer has **two main storage types**:

1. Disk Storage
2. Memory Storage

---

# 📁 Disk Storage (Save File to Folder)

## When to use Disk Storage?

* Save files on server
* Images, PDFs, documents
* Simple projects

---

## 📁 Disk Storage Code Example

### `upload.middleware.ts`

```ts
import multer from "multer";
import path from "path";

const diskStorage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/");
  },

  filename: (req, file, cb) => {
    const uniqueName = Date.now() + path.extname(file.originalname);
    cb(null, uniqueName);
  },
});

export const uploadDisk = multer({ storage: diskStorage });
```

## 🧩 Disk Storage Flow

1. User uploads file
2. Multer receives file
3. File saved in `uploads/` folder
4. File info available in `req.file`

---

# 🧠 Memory Storage (Save File in RAM)

## When to use Memory Storage?

* Upload directly to cloud (Cloudinary, AWS S3)
* Do NOT save file on server

⚠️ Not recommended for large files

---

## 🧠 Memory Storage Code Example

```ts
import multer from "multer";

const memoryStorage = multer.memoryStorage();

export const uploadMemory = multer({
  storage: memoryStorage,
  limits: { fileSize: 2 * 1024 * 1024 }, // 2MB
});
```

---

## 🧩 Memory Storage Flow

1. File uploaded
2. Multer keeps file in RAM
3. File available as `req.file.buffer`
4. Send buffer to cloud

---

## 📏 File Size Validation

```ts
limits: {
  fileSize: 2 * 1024 * 1024 // 2MB
}
```

❌ Files larger than 2MB are rejected

---

## 🖼 File Type Validation (Images Only)

```ts
const fileFilter = (req, file, cb) => {
  const allowedTypes = ["image/png", "image/jpeg", "image/jpg"];

  if (!allowedTypes.includes(file.mimetype)) {
    return cb(new Error("Only images allowed"), false);
  }

  cb(null, true);
};
```

---

## ✅ Final Multer Configuration

```ts
export const upload = multer({
  storage: diskStorage,
  limits: { fileSize: 2 * 1024 * 1024 },
  fileFilter,
});
```

---

## 🚏 Upload Route Example

```ts
router.post(
  "/upload",
  upload.single("image"),
  uploadImage
);
```

---

## 🎯 Upload Controller Example

```ts
export const uploadImage = (req, res) => {
  if (!req.file) {
    return res.status(400).json({ message: "No file uploaded" });
  }

  res.json({
    message: "File uploaded successfully",
    file: req.file,
  });
};
```

---

## 📂 Multiple File Upload Example

```ts
router.post(
  "/upload-multiple",
  upload.array("images", 5),
  (req, res) => {
    res.json({ files: req.files });
  }
);
```

---

## 🧪 Testing with Postman

1. Method: `POST`
2. URL: `/upload`
3. Body → `form-data`
4. Key: `image` (type = File)
5. Select image
6. Send request

✔ Image uploads successfully

---

## ☁️ Cloud Storage (Why Needed?)

Local storage ❌ is not recommended for production because:

* Server crash → files lost
* Scaling issues
* Performance problems

👉 Solution: **Cloud Storage**

---

## ☁️ Cloudinary (Beginner-Friendly)

Cloudinary is an **image hosting service**.

### Features

* Image upload
* Auto optimization
* CDN delivery
* Secure URLs

---

## 📦 Install Cloudinary

```bash
npm install cloudinary
```

---

## ⚙️ Cloudinary Configuration

### `.env`

```env
CLOUD_NAME=your_cloud_name
CLOUD_API_KEY=your_api_key
CLOUD_API_SECRET=your_secret
```

---

### File: `utils/cloudinary.ts`

```ts
import { v2 as cloudinary } from "cloudinary";

cloudinary.config({
  cloud_name: process.env.CLOUD_NAME,
  api_key: process.env.CLOUD_API_KEY,
  api_secret: process.env.CLOUD_API_SECRET,
});

export default cloudinary;
```

---

## ☁️ Upload Image to Cloudinary

### Controller Update

```ts
import cloudinary from "../utils/cloudinary";

export const uploadImage = async (
  req: Request,
  res: Response
) => {
  if (!req.file) {
    return res.status(400).json({ message: "No file uploaded" });
  }

  const result = await cloudinary.uploader.upload(
    req.file.path,
    { folder: "uploads" }
  );

  res.json({
    message: "Uploaded to cloud",
    imageUrl: result.secure_url,
  });
};
```

✔ Image is now stored on **Cloudinary**
✔ You get a **public URL**

---

## ⚡ A Better Approach: Direct Upload with `multer-storage-cloudinary`

The two-step flow above (Multer saves to local disk → then manually upload that file to Cloudinary) works, but it means every uploaded file briefly touches your server's disk before being pushed to the cloud — unnecessary I/O, and a problem on serverless/read-only filesystems.

**This repository's own code** (`Backend/src/helpers/upload.ts` and `Backend/src/config/cloudinary.ts`) uses a cleaner approach: `multer-storage-cloudinary`, a Multer storage engine that uploads **directly** to Cloudinary — no local disk step at all.

```bash
npm install multer-storage-cloudinary
```

```ts
// config/cloudinary.ts
import { v2 as cloudinary } from "cloudinary";

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME!,
  api_key: process.env.CLOUDINARY_API_KEY!,
  api_secret: process.env.CLOUDINARY_API_SECRET!,
});

export default cloudinary;
```

```ts
// helpers/upload.ts
import multer from "multer";
import { CloudinaryStorage } from "multer-storage-cloudinary";
import cloudinary from "../config/cloudinary";

const storage = new CloudinaryStorage({
  cloudinary,
  params: {
    folder: "products",
    allowed_formats: ["jpg", "png", "jpeg", "webp"],
  } as any,
});

export const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 },
});
```

Now in your route, `req.file.path` (or `req.files[i].path`) is **already the final Cloudinary URL** — no separate `cloudinary.uploader.upload(...)` call needed:

```ts
// product.controllers.ts
export const createProduct = async (req: Request, res: Response) => {
  const images = req.files && Array.isArray(req.files)
    ? req.files.map((file: any) => ({ url: file.path })) // already a Cloudinary URL!
    : [];

  const product = await Product.create({ ...req.body, images });
  res.status(201).json({ success: true, data: product });
};
```

| | Two-step (disk → cloud) | Direct (`multer-storage-cloudinary`) |
|---|---|---|
| Touches local disk | ✅ Yes, briefly | ❌ No |
| Works on serverless/read-only filesystems | ❌ Often breaks | ✅ Yes |
| Extra `cloudinary.uploader.upload()` call needed | ✅ Yes | ❌ No — handled automatically |
| Good for learning the underlying steps | ✅ Yes | Less transparent, but more production-ready |

> 💡 Learning the manual two-step version first is valuable because it shows you **what's actually happening** underneath. In real projects (like this repo's `Backend/`), prefer the direct `multer-storage-cloudinary` approach.

---

## ☁️ AWS S3 (Concept Only – Beginner)

Note for students:

* AWS S3 works similarly to Cloudinary
* Ideal for large files (videos, documents)
* Requires IAM setup, bucket, access keys
* Slightly advanced → can be taught later

---

## 🔐 Security Best Practices

✔ Always validate file type
✔ Limit file size
✔ Never trust filenames from the user
✔ Use cloud storage in production
✔ Protect upload routes with authentication

---

## 📌 Real-World Use Cases

* User profile pictures
* Product image uploads
* Resume uploads
* Certificates
* Blog thumbnails

---

## 🏁 Summary

This README covers:

✔ Multipart form data
✔ Multer basics
✔ Image upload
✔ File size & type validation
✔ Local storage
✔ Cloudinary integration
✔ Production best practices

This topic is **mandatory** for every backend beginner and is widely used in:

* Social media apps
* E-commerce platforms
* Admin dashboards
* SaaS systems