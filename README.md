# Design Patterns in .NET / C#

A practical guide to **Design Patterns in C# and .NET**, with simple explanations, real-world use cases, class diagrams, implementation examples, advantages, disadvantages, and interview questions.

This repository is designed for **.NET developers, backend developers, full-stack developers, and interview preparation**.

---

## 📚 Design Patterns

| # | Design Pattern | Category | Documentation |
|---:|---|---|---|
| 1 | Singleton | Creational | [Singleton Pattern](./Design-Patterns/Creational/Singleton.md) |
| 2 | Factory Method | Creational | [Factory Method](./Design-Patterns/Creational/Factory-Method.md) |
| 3 | Abstract Factory | Creational | [Abstract Factory](./Design-Patterns/Creational/Abstract-Factory.md) |
| 4 | Builder | Creational | [Builder Pattern](./Design-Patterns/Creational/Builder.md) |
| 5 | Prototype | Creational | [Prototype Pattern](./Design-Patterns/Creational/Prototype.md) |
| 6 | Adapter | Structural | [Adapter Pattern](./Design-Patterns/Structural/Adapter.md) |
| 7 | Bridge | Structural | [Bridge Pattern](./Design-Patterns/Structural/Bridge.md) |
| 8 | Composite | Structural | [Composite Pattern](./Design-Patterns/Structural/Composite.md) |
| 9 | Decorator | Structural | [Decorator Pattern](./Design-Patterns/Structural/Decorator.md) |
| 10 | Facade | Structural | [Facade Pattern](./Design-Patterns/Structural/Facade.md) |
| 11 | Flyweight | Structural | [Flyweight Pattern](./Design-Patterns/Structural/Flyweight.md) |
| 12 | Proxy | Structural | [Proxy Pattern](./Design-Patterns/Structural/Proxy.md) |
| 13 | Chain of Responsibility | Behavioral | [Chain of Responsibility](./Design-Patterns/Behavioral/Chain-of-Responsibility.md) |
| 14 | Command | Behavioral | [Command Pattern](./Design-Patterns/Behavioral/Command.md) |
| 15 | Interpreter | Behavioral | [Interpreter Pattern](./Design-Patterns/Behavioral/Interpreter.md) |
| 16 | Iterator | Behavioral | [Iterator Pattern](./Design-Patterns/Behavioral/Iterator.md) |
| 17 | Mediator | Behavioral | [Mediator Pattern](./Design-Patterns/Behavioral/Mediator.md) |
| 18 | Memento | Behavioral | [Memento Pattern](./Design-Patterns/Behavioral/Memento.md) |
| 19 | Observer | Behavioral | [Observer Pattern](./Design-Patterns/Behavioral/Observer.md) |
| 20 | State | Behavioral | [State Pattern](./Design-Patterns/Behavioral/State.md) |
| 21 | Strategy | Behavioral | [Strategy Pattern](./Design-Patterns/Behavioral/Strategy.md) |
| 22 | Template Method | Behavioral | [Template Method](./Design-Patterns/Behavioral/Template-Method.md) |
| 23 | Visitor | Behavioral | [Visitor Pattern](./Design-Patterns/Behavioral/Visitor.md) |

---

# 📖 What Are Design Patterns?

Design patterns are **proven, reusable solutions to commonly occurring software design problems**.

They are not ready-made pieces of code. Instead, they provide a **general approach for structuring classes, objects, and their interactions**.

For example, instead of creating a completely new solution every time you need to create objects dynamically, you can use a **Factory Pattern**.

---

# 🎯 Why Use Design Patterns?

Design patterns help developers:

- Write maintainable code
- Reduce code duplication
- Improve code organization
- Follow SOLID principles
- Reduce tight coupling
- Improve extensibility
- Make systems easier to test
- Improve readability
- Provide proven solutions to common problems
- Make communication between developers easier

---

# 🏗️ Design Pattern Categories

Design patterns are commonly divided into three major categories.

## 1. Creational Patterns

Creational patterns deal with **object creation**.

Examples:

- Singleton
- Factory Method
- Abstract Factory
- Builder
- Prototype

---

## 2. Structural Patterns

Structural patterns deal with **how classes and objects are composed**.

Examples:

- Adapter
- Bridge
- Composite
- Decorator
- Facade
- Flyweight
- Proxy

---

## 3. Behavioral Patterns

Behavioral patterns deal with **communication and responsibility between objects**.

Examples:

- Chain of Responsibility
- Command
- Interpreter
- Iterator
- Mediator
- Memento
- Observer
- State
- Strategy
- Template Method
- Visitor

---

# 📂 Repository Structure

```text
Design-Patterns/
│
├── README.md
│
├── Design-Patterns/
│   │
│   ├── Creational/
│   │   ├── Singleton.md
│   │   ├── Factory-Method.md
│   │   ├── Abstract-Factory.md
│   │   ├── Builder.md
│   │   └── Prototype.md
│   │
│   ├── Structural/
│   │   ├── Adapter.md
│   │   ├── Bridge.md
│   │   ├── Composite.md
│   │   ├── Decorator.md
│   │   ├── Facade.md
│   │   ├── Flyweight.md
│   │   └── Proxy.md
│   │
│   └── Behavioral/
│       ├── Chain-of-Responsibility.md
│       ├── Command.md
│       ├── Interpreter.md
│       ├── Iterator.md
│       ├── Mediator.md
│       ├── Memento.md
│       ├── Observer.md
│       ├── State.md
│       ├── Strategy.md
│       ├── Template-Method.md
│       └── Visitor.md
│
└── Examples/
    └── ...
```

---

# 💻 Technology

The examples in this repository primarily use:

- C#
- .NET
- ASP.NET Core
- Object-Oriented Programming
- SOLID Principles
- Dependency Injection
- Clean Architecture concepts
- Microservices concepts

---

# 🔥 Design Patterns in Real-World .NET Applications

Design patterns are already used extensively in modern .NET applications.

For example:

| Design Pattern | Common .NET Usage |
|---|---|
| Singleton | Configuration, caching, shared services |
| Factory | Creating different implementations |
| Builder | Building complex objects |
| Adapter | Integrating third-party APIs |
| Decorator | Adding logging, caching, authorization |
| Facade | Simplifying complex subsystems |
| Proxy | Lazy loading, authorization, remote calls |
| Strategy | Payment methods, pricing algorithms |
| Observer | Event notifications |
| Mediator | Decoupling components |
| Command | CQRS and request processing |
| Chain of Responsibility | Middleware and request pipelines |

---

# 🧩 Design Pattern Documentation Format

Each design pattern in this repository follows a consistent structure.

For example:

```text
1. Introduction
2. What Problem Does It Solve?
3. When Should We Use It?
4. When Should We Avoid It?
5. Real-World Example
6. C# Implementation
7. Project Structure
8. Step-by-Step Explanation
9. Advantages
10. Disadvantages
11. Real-World .NET Usage
12. Interview Questions
```

---

# 🎓 Recommended Learning Order

If you are new to design patterns, don't try to learn all patterns at once.

A recommended learning path is:

### Step 1 — Creational

1. Singleton
2. Factory Method
3. Abstract Factory
4. Builder
5. Prototype

### Step 2 — Structural

6. Adapter
7. Decorator
8. Facade
9. Proxy
10. Composite

### Step 3 — Behavioral

11. Strategy
12. Observer
13. Command
14. Chain of Responsibility
15. Mediator
16. State
17. Template Method
18. Iterator
19. Memento
20. Visitor
21. Interpreter

---

# ⭐ Most Important Patterns for .NET Developers

If your primary goal is **ASP.NET Core / .NET development and interviews**, pay special attention to:

- Singleton
- Factory
- Builder
- Adapter
- Decorator
- Facade
- Proxy
- Strategy
- Observer
- Command
- Mediator
- Chain of Responsibility

These patterns frequently appear in enterprise application architecture and framework design.

---

# 🎯 Design Patterns vs SOLID

Design patterns and SOLID principles are related but they are **not the same thing**.

### SOLID

SOLID provides **principles for designing maintainable object-oriented software**.

### Design Patterns

Design patterns provide **proven approaches for solving recurring design problems**.

For example:

```text
SOLID Principles
       ↓
Better Object-Oriented Design
       ↓
Design Patterns
       ↓
Maintainable Architecture
```

---

# 📌 Important Note

A design pattern should not be used simply because it exists.

Always ask:

> **What problem am I trying to solve?**

Then determine whether a design pattern actually makes the solution simpler, more maintainable, and easier to extend.

Using too many patterns can introduce unnecessary complexity.

---

# 🚀 Goals of This Repository

This repository aims to provide:

- Simple explanations
- Practical C# examples
- Enterprise-oriented examples
- Real-world scenarios
- .NET-specific implementations
- Interview preparation
- Advantages and disadvantages
- Clear project structures
- Easy-to-understand diagrams
- Pattern comparison

---

# 🤝 Contributing

Contributions are welcome.

If you find an issue, improvement, or missing example:

1. Fork the repository
2. Create a new branch
3. Make your changes
4. Commit your changes
5. Create a Pull Request

---

# 📜 License

This repository is intended for **learning and educational purposes**.

---

## ⭐ If This Repository Helps You

If you find this repository useful for learning **C#, .NET, ASP.NET Core, Design Patterns, and Software Architecture**, consider giving it a ⭐ on GitHub.