# 🚀 Introduction to TypeScript & Environment Setup

## 🌐 Overview

Welcome to your first TypeScript class! Before we write a single line of code, it's important to understand **what TypeScript is**, **why the industry uses it**, and **how to set it up** on your machine correctly.

By the end of this class, you'll have a fully working TypeScript environment and you'll have written and run your very first `.ts` file.

---

## 🎯 Learning Goals

After this class, you will be able to:

* Explain what TypeScript is and how it relates to JavaScript
* Understand **why** companies prefer TypeScript for large projects
* Install and configure TypeScript on your computer
* Understand `tsconfig.json` basics
* Write, compile, and run your first TypeScript program

---

## 💡 What Is TypeScript?

> **TypeScript = JavaScript + Types**

TypeScript is an **open-source programming language** developed by **Microsoft (2012)**, created by **Anders Hejlsberg** (also the creator of C#).

It is a **superset of JavaScript** — this means:

* Every valid JavaScript file is also a valid TypeScript file
* TypeScript **adds extra features** on top of JavaScript: static types, interfaces, generics, and compile-time error checking
* Browsers and Node.js **cannot run TypeScript directly** — it must first be **compiled (transpiled)** into plain JavaScript

```
 TypeScript (.ts)  --->  TypeScript Compiler (tsc)  --->  JavaScript (.js)  --->  Runs in Browser / Node.js
```

---

## ❓ Why Do We Need TypeScript If JavaScript Already Works?

JavaScript is **dynamically typed** — a variable's type can change at any time, and mistakes are only caught **while the app is running** (often in production, in front of real users 😬).

```js
// JavaScript
let age = 25;
age = "twenty five"; // No error! JS allows this silently
console.log(age * 2); // NaN — bug discovered too late
```

```ts
// TypeScript
let age: number = 25;
age = "twenty five"; // ❌ Compile-time error — caught immediately
```

### 🧠 Key Benefits

| Benefit | Description |
|---|---|
| ✅ **Catches bugs early** | Errors are shown while typing, not after deployment |
| 💬 **Better autocomplete** | Editors like VS Code understand your data shapes |
| 🧱 **Self-documenting code** | Types describe what a function expects and returns |
| 🔄 **Safer refactoring** | Renaming/moving code shows every broken reference |
| 🏢 **Industry standard** | Used by Google, Microsoft, Airbnb, Netflix, Meta |

---

## 🖥️ How TypeScript Fits Into the Workflow

1. You write code in a `.ts` file
2. The **TypeScript Compiler (`tsc`)** checks your types and converts the file to `.js`
3. Node.js (or the browser) runs the plain JavaScript output

```ts
// greet.ts
const greet = (name: string): string => {
  return `Hello, ${name}!`;
};

console.log(greet("Ali"));
```

```bash
npx tsc greet.ts     # produces greet.js
node greet.js         # runs the compiled file
```

> 💡 TypeScript **never runs directly** — it always becomes JavaScript first. Types exist only to help *you* while coding; they disappear after compilation.

---

## 🧰 Step 1: Prerequisites

Before installing TypeScript, make sure you have:

* [Node.js](https://nodejs.org/) (v18 or higher) installed
* npm (comes bundled with Node.js)
* A code editor — **VS Code** is strongly recommended (best TypeScript support)

Verify your installation:

```bash
node -v
npm -v
```

---

## ⚙️ Step 2: Create a Project Folder

```bash
mkdir typescript-basics
cd typescript-basics
npm init -y
```

`npm init -y` creates a `package.json` file instantly with default values — the "identity card" of your project.

---

## 📦 Step 3: Install TypeScript

```bash
npm install typescript --save-dev
```

* `typescript` gives you the `tsc` (TypeScript Compiler) command
* `--save-dev` means it's only needed during development, not in production

Optionally, also install a fast runner so you don't need to compile manually every time:

```bash
npm install tsx --save-dev
```

| Tool | Purpose |
|---|---|
| `tsc` | Compiles `.ts` → `.js` (official compiler) |
| `tsx` | Runs `.ts` files directly, instantly, no manual compile step |
| `ts-node` | Older alternative to `tsx`, also runs `.ts` directly |

Verify installation:

```bash
npx tsc --version
```

---

## 📝 Step 4: Create `tsconfig.json`

This file tells the compiler **how** to behave.

```bash
npx tsc --init
```

This generates a big file with many commented-out options. For now, open it and set these core options:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "rootDir": "src",
    "outDir": "dist",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

### 🔍 What Each Option Means

| Option | Meaning |
|---|---|
| `target` | Which JS version to output (`ES2020` supports modern syntax like async/await) |
| `module` | Module system — `commonjs` is what Node.js uses (`require`/`module.exports`) |
| `rootDir` | Folder where your `.ts` source files live |
| `outDir` | Folder where compiled `.js` files will be placed |
| `esModuleInterop` | Lets you use `import x from "y"` even with older CommonJS packages |
| `strict` | Turns on **all** strict type-checking rules — always keep this `true` |
| `skipLibCheck` | Skips checking type files inside `node_modules` (faster builds) |

> 💡 **Golden Rule:** Always keep `"strict": true`. It's the entire reason to use TypeScript — it forces you to handle `null`, `undefined`, and type mismatches properly.

---

## 📂 Recommended Project Structure

```
typescript-basics/
│
├── src/
│   └── index.ts       👈 you write code here
│
├── dist/               👈 compiled JS appears here (auto-created)
│
├── package.json
└── tsconfig.json
```

---

## ✍️ Step 5: Write Your First TypeScript Program

Create `src/index.ts`:

```ts
// src/index.ts

const studentName: string = "Ayesha";
const rollNumber: number = 42;
const isPresent: boolean = true;

function markAttendance(name: string, present: boolean): string {
  return present
    ? `${name} is marked Present ✅`
    : `${name} is marked Absent ❌`;
}

console.log(markAttendance(studentName, isPresent));
console.log(`Roll Number: ${rollNumber}`);
```

### 🔍 Code Explanation

| Line | Explanation |
|---|---|
| `const studentName: string = "Ayesha";` | Declares a variable and explicitly says it must always be text |
| `function markAttendance(name: string, present: boolean): string` | The function accepts a `string` and a `boolean`, and must return a `string` |
| `present ? ... : ...` | A ternary operator — shorthand for if/else |

---

## ⚙️ Step 6: Compile and Run

### Option 1 — Using `tsc` (compile, then run)

```bash
npx tsc
node dist/index.js
```

### Option 2 — Using `tsx` (run directly, no compile step)

```bash
npx tsx src/index.ts
```

### 💡 Add Convenience Scripts

In `package.json`:

```json
"scripts": {
  "dev": "tsx src/index.ts",
  "build": "tsc",
  "start": "node dist/index.js"
}
```

Now you can simply run:

```bash
npm run dev
```

---

## 🎉 Expected Output

```
Ayesha is marked Present ✅
Roll Number: 42
```

---

## 🧠 What Happens If You Break a Type Rule?

Try this on purpose:

```ts
markAttendance(123, true); // ❌ Error
```

You'll immediately see:

```
Argument of type 'number' is not assignable to parameter of type 'string'.
```

This is TypeScript **protecting you before the code ever runs** — this is the entire point of the language.

---

## 🧾 Summary

| Concept | Key Takeaway |
|---|---|
| TypeScript | JavaScript + static types, compiled with `tsc` |
| Setup | `npm init -y` → `npm i typescript --save-dev` → `npx tsc --init` |
| `tsconfig.json` | Controls how your code compiles — always enable `strict` |
| Running code | `tsc` + `node`, or directly with `tsx` |
| Core benefit | Catches mistakes **before** the app runs |

---

## 🧩 Hands-On Practice

1. Set up a new TypeScript project from scratch following every step above.
2. Write a function `calculateAge(birthYear: number): number` that returns the current age.
3. Intentionally call it with a string argument and observe the compiler error.
4. Add a `dev` script to `package.json` and run your project with `npm run dev`.

Next class: **Basic Types & Type Annotations** — we'll go deep into every type TypeScript offers. 🚀
