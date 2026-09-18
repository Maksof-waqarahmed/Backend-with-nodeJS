# 🏛️ Classes in TypeScript — Basics

## 🌐 Overview

TypeScript gives JavaScript classes a huge upgrade: typed properties, access modifiers (`public`/`private`/`protected`), constructor shortcuts, and more. This class covers everything you need to model real-world entities like `User`, `Product`, or `Order` using **object-oriented programming (OOP)**.

---

## 🎯 Learning Goals

* Declare typed class properties
* Understand constructors
* Understand `public`, `private`, and `protected`
* Use the **parameter property shortcut**
* Understand `readonly` class properties
* Use getters and setters

---

## 🧩 Basic Class Syntax

```ts
class Person {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet(): string {
    return `Hi, I'm ${this.name} and I'm ${this.age} years old.`;
  }
}

const p1 = new Person("Ali", 25);
console.log(p1.greet()); // "Hi, I'm Ali and I'm 25 years old."
```

### 🔍 Breakdown

| Part | Meaning |
|---|---|
| `name: string; age: number;` | Declares the class's **properties** and their types |
| `constructor(...)` | Runs automatically when you create a new instance (`new Person(...)`) |
| `this.name = name` | Assigns the passed-in value to the instance |
| `greet()` | A regular **method** on the class |

---

## 🔐 Access Modifiers

TypeScript adds three keywords to control **who can access a property/method**:

| Modifier | Accessible From |
|---|---|
| `public` (default) | Anywhere — inside the class, outside, and in subclasses |
| `private` | Only inside the **same class** |
| `protected` | Inside the class **and its subclasses** |

```ts
class BankAccount {
  public accountHolder: string;
  private balance: number;
  protected accountType: string;

  constructor(accountHolder: string, balance: number) {
    this.accountHolder = accountHolder;
    this.balance = balance;
    this.accountType = "Savings";
  }

  deposit(amount: number): void {
    this.balance += amount; // ✅ allowed, we're inside the class
  }

  getBalance(): number {
    return this.balance;
  }
}

const account = new BankAccount("Sara", 1000);
account.deposit(500);
console.log(account.getBalance()); // 1500

account.balance;   // ❌ Error: 'balance' is private
account.accountType; // ❌ Error: 'accountType' is protected
```

> 💡 **Why this matters:** `private` protects sensitive internal state (like `balance`) from being changed directly from outside — forcing consumers to go through safe methods like `deposit()`.

---

## ⚡ Parameter Properties (Shortcut Syntax)

Writing `this.x = x` for every constructor parameter is repetitive. TypeScript lets you **declare and assign in one step** by adding an access modifier directly to a constructor parameter.

```ts
class BankAccount {
  constructor(
    public accountHolder: string,
    private balance: number,
    protected accountType: string = "Savings"
  ) {}

  deposit(amount: number): void {
    this.balance += amount;
  }

  getBalance(): number {
    return this.balance;
  }
}

const acc = new BankAccount("Ahmed", 2000);
console.log(acc.getBalance()); // 2000
```

This is **identical** to the longer version above — just far less boilerplate. This shortcut is used constantly in real-world TypeScript (NestJS, Angular, etc.).

---

## 🔒 `readonly` Class Properties

```ts
class Product {
  readonly id: number;
  title: string;

  constructor(id: number, title: string) {
    this.id = id;
    this.title = title;
  }
}

const p = new Product(1, "Laptop");
p.title = "Gaming Laptop"; // ✅ allowed
p.id = 999;                 // ❌ Error: Cannot assign to 'id' because it is read-only
```

---

## 🎛️ Getters and Setters

Getters/setters let you control how a property is **read** or **written**, while callers still use normal property syntax.

```ts
class Circle {
  private _radius: number;

  constructor(radius: number) {
    this._radius = radius;
  }

  get radius(): number {
    return this._radius;
  }

  set radius(value: number) {
    if (value <= 0) {
      throw new Error("Radius must be positive");
    }
    this._radius = value;
  }

  get area(): number {
    return Math.PI * this._radius ** 2;
  }
}

const circle = new Circle(5);
console.log(circle.radius); // 5 (calls the getter)
console.log(circle.area);   // 78.53...

circle.radius = 10; // calls the setter
circle.radius = -5; // ❌ throws: "Radius must be positive"
```

---

## 🧱 Static Properties and Methods

`static` members belong to the **class itself**, not to individual instances.

```ts
class MathHelper {
  static PI: number = 3.14159;

  static square(x: number): number {
    return x * x;
  }
}

console.log(MathHelper.PI);        // 3.14159
console.log(MathHelper.square(4)); // 16
```

You don't need `new MathHelper()` to use static members — you call them directly on the class.

### 🧠 Real Use Case: Counting Instances

```ts
class User {
  static totalUsers: number = 0;

  constructor(public name: string) {
    User.totalUsers++;
  }
}

new User("Ali");
new User("Sara");
console.log(User.totalUsers); // 2
```

---

## 🧾 Full Example

```ts
class Product {
  constructor(
    public readonly id: number,
    public title: string,
    private price: number,
    private stock: number
  ) {}

  get isInStock(): boolean {
    return this.stock > 0;
  }

  sell(quantity: number): string {
    if (quantity > this.stock) {
      return `Not enough stock for ${this.title}`;
    }
    this.stock -= quantity;
    return `Sold ${quantity} of ${this.title}. Remaining: ${this.stock}`;
  }

  getPrice(): number {
    return this.price;
  }
}

const laptop = new Product(1, "Laptop", 1200, 5);

console.log(laptop.isInStock);       // true
console.log(laptop.sell(2));          // "Sold 2 of Laptop. Remaining: 3"
console.log(laptop.getPrice());       // 1200
```

---

## 🧾 Summary

| Concept | Keyword | Meaning |
|---|---|---|
| Instance creation | `constructor` | Runs when `new ClassName()` is called |
| Public access | `public` (default) | Accessible from anywhere |
| Private access | `private` | Accessible only inside the same class |
| Protected access | `protected` | Accessible in the class and its subclasses |
| Shortcut | parameter properties | `constructor(public name: string)` |
| Immutable property | `readonly` | Can only be set once, in the constructor |
| Computed read | `get` | Runs custom logic when reading a property |
| Controlled write | `set` | Runs custom logic (e.g. validation) when writing |
| Class-level member | `static` | Belongs to the class, not an instance |

---

## 🧩 Hands-On Practice

1. Create a `Student` class with `name` (public), `rollNo` (readonly), and `grades` (private array of numbers).
2. Add a method `addGrade(grade: number): void` and a getter `average` that computes the average grade.
3. Create a `Car` class using the parameter-property shortcut with `brand`, `model`, and a `private fuel: number`.
4. Add `drive(distance: number): void` that reduces fuel, and throws an error if fuel would go below 0.
5. Add a `static totalCars` counter that increments every time a new `Car` is created.

Next class: **Inheritance, Abstract Classes & Interfaces with Classes.** 🚀
