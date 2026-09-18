# 🧬 Inheritance, Abstract Classes & Interfaces with Classes

## 🌐 Overview

This class builds on last time's OOP basics. You'll learn how classes can **inherit** behavior from one another, how to define **abstract classes** that force subclasses to implement certain methods, and how to make a class **implement an interface** as a contract.

---

## 🎯 Learning Goals

* Use `extends` for class inheritance
* Override parent methods and use `super`
* Create `abstract` classes and methods
* Use `implements` to enforce an interface contract
* Understand polymorphism in practice

---

## 🧬 Inheritance with `extends`

A **subclass** (child) can inherit properties and methods from a **superclass** (parent) using `extends`.

```ts
class Animal {
  constructor(public name: string) {}

  makeSound(): void {
    console.log(`${this.name} makes a sound.`);
  }
}

class Dog extends Animal {
  bark(): void {
    console.log(`${this.name} says Woof!`);
  }
}

const dog = new Dog("Rex");
dog.makeSound(); // inherited from Animal → "Rex makes a sound."
dog.bark();       // defined in Dog → "Rex says Woof!"
```

---

## 🔁 Overriding Methods + `super`

A subclass can **override** (replace) a parent method, and use `super` to still call the parent's original version if needed.

```ts
class Animal {
  constructor(public name: string) {}

  makeSound(): void {
    console.log(`${this.name} makes a generic sound.`);
  }
}

class Cat extends Animal {
  // Override
  makeSound(): void {
    super.makeSound(); // calls Animal's version first
    console.log(`${this.name} says Meow!`);
  }
}

const cat = new Cat("Whiskers");
cat.makeSound();
// "Whiskers makes a generic sound."
// "Whiskers says Meow!"
```

### `super()` in Constructors

If a subclass defines its own constructor, it **must** call `super(...)` before using `this`.

```ts
class Employee {
  constructor(public name: string, public salary: number) {}
}

class Manager extends Employee {
  constructor(name: string, salary: number, public teamSize: number) {
    super(name, salary); // must call parent constructor first
  }
}

const manager = new Manager("Waqar", 90000, 8);
console.log(manager.name, manager.salary, manager.teamSize);
// "Waqar 90000 8"
```

---

## 🏛️ Abstract Classes

An **abstract class** is a class that **cannot be instantiated directly** — it exists only to be extended. It can define both:

* **Abstract methods** — no implementation, subclasses MUST provide one
* **Regular methods** — shared logic every subclass inherits as-is

```ts
abstract class Shape {
  abstract getArea(): number; // no body — every subclass must implement this

  describe(): string {
    return `This shape has an area of ${this.getArea()}`; // shared logic
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  getArea(): number {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  constructor(private width: number, private height: number) {
    super();
  }

  getArea(): number {
    return this.width * this.height;
  }
}

const shapes: Shape[] = [new Circle(5), new Rectangle(4, 6)];

shapes.forEach((shape) => console.log(shape.describe()));
// "This shape has an area of 78.53981633974483"
// "This shape has an area of 24"
```

```ts
new Shape(); // ❌ Error: Cannot create an instance of an abstract class
```

> 💡 **Why use abstract classes?** They guarantee that every subclass implements the required behavior (`getArea()`), while still sharing common logic (`describe()`) — avoiding duplicated code.

---

## 📘 Implementing an Interface (`implements`)

While `extends` inherits actual **implementation**, `implements` only enforces a **contract/shape** — the class must provide its own implementation for everything listed.

```ts
interface Payable {
  amount: number;
  pay(): void;
}

class Invoice implements Payable {
  constructor(public amount: number) {}

  pay(): void {
    console.log(`Paid $${this.amount}`);
  }
}

const invoice = new Invoice(500);
invoice.pay(); // "Paid $500"
```

### A Class Can Implement Multiple Interfaces

```ts
interface Printable {
  print(): void;
}
interface Payable {
  pay(): void;
}

class Order implements Printable, Payable {
  print(): void {
    console.log("Printing order...");
  }
  pay(): void {
    console.log("Processing payment...");
  }
}
```

### `extends` vs `implements`

| | `extends` | `implements` |
|---|---|---|
| Inherits actual code | ✅ Yes | ❌ No — only enforces shape |
| Can extend/implement multiple | ❌ Only one class | ✅ Multiple interfaces |
| Use case | Share behavior between related classes | Guarantee a class follows a specific contract |

---

## 🎭 Polymorphism in Practice

**Polymorphism** means objects of different classes can be treated through a **common interface/parent type**, while each behaves according to its own implementation.

```ts
abstract class PaymentMethod {
  abstract process(amount: number): string;
}

class CreditCard extends PaymentMethod {
  process(amount: number): string {
    return `Charged $${amount} to credit card`;
  }
}

class PayPal extends PaymentMethod {
  process(amount: number): string {
    return `Paid $${amount} via PayPal`;
  }
}

function checkout(amount: number, method: PaymentMethod): void {
  console.log(method.process(amount)); // works no matter which subclass is passed
}

checkout(100, new CreditCard()); // "Charged $100 to credit card"
checkout(50, new PayPal());       // "Paid $50 via PayPal"
```

This is the same design idea used by real payment systems (and mirrors the `paymentMethod` union used in this repo's `Backend/src/payment` module, just expressed with classes instead of literal strings).

---

## 🧾 Full Example

```ts
interface Notifiable {
  notify(message: string): void;
}

abstract class User implements Notifiable {
  constructor(public name: string, public email: string) {}

  abstract getRole(): string;

  notify(message: string): void {
    console.log(`📧 To ${this.email}: ${message}`);
  }
}

class Customer extends User {
  getRole(): string {
    return "Customer";
  }
}

class Admin extends User {
  constructor(name: string, email: string, public permissions: string[]) {
    super(name, email);
  }

  getRole(): string {
    return "Admin";
  }
}

const users: User[] = [
  new Customer("Ali", "ali@mail.com"),
  new Admin("Sara", "sara@mail.com", ["delete", "edit"]),
];

users.forEach((user) => {
  console.log(`${user.name} is a ${user.getRole()}`);
  user.notify("Welcome to the platform!");
});
```

**Output:**

```
Ali is a Customer
📧 To ali@mail.com: Welcome to the platform!
Sara is a Admin
📧 To sara@mail.com: Welcome to the platform!
```

---

## 🧾 Summary

| Concept | Keyword | Purpose |
|---|---|---|
| Inheritance | `extends` | Reuse a parent class's properties/methods |
| Override | same method name in child | Replace parent behavior |
| Parent call | `super()` / `super.method()` | Call the parent constructor/method |
| Abstract class | `abstract class` | Cannot be instantiated; defines shared + required behavior |
| Abstract method | `abstract method(): T` | Must be implemented by every subclass |
| Contract | `implements` | Enforces an interface's shape without inheriting code |
| Polymorphism | — | Treat different subclasses through one common type |

---

## 🧩 Hands-On Practice

1. Create an abstract class `Employee` with an abstract method `calculateSalary(): number` and a shared method `getDetails()`.
2. Create `FullTimeEmployee` and `PartTimeEmployee` subclasses that implement `calculateSalary()` differently.
3. Create an interface `Drivable { drive(): void }` and a class `Car implements Drivable`.
4. Build a small polymorphism example: an array of `Shape[]` (Circle, Square, Triangle) where you call `.getArea()` on each in a loop.
5. Create a `Manager extends Employee` class that calls `super()` in its constructor and adds a `teamSize` property.

Next week (Week 5): **Generics** — writing flexible, reusable, type-safe code. 🚀
