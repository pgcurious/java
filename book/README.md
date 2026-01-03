# Modern Java: A Hands-On Guide (Java 9-21+)

> **For developers who know Java basics but want to master modern Java features**

## About This Book

This book is designed for Java developers who are comfortable with Java 8 fundamentals but haven't kept up with the rapid evolution of the language. Each chapter presents **real problems** and shows how modern Java features provide elegant solutions.

### How to Use This Book

1. **Read the problem statement** - Understand what we're trying to solve
2. **Study the "Before" code** - See how you might have solved it in Java 8
3. **Implement the "After" code** - Type it yourself in your IDE
4. **Complete the exercises** - Reinforce your learning
5. **Build the mini-projects** - Apply multiple concepts together

### Prerequisites

- Java 17+ installed (Java 21 recommended for all features)
- Basic understanding of Java 8 (lambdas, streams, Optional)
- An IDE (IntelliJ IDEA, Eclipse, or VS Code with Java extensions)

---

## Table of Contents

### Part I: Language Enhancements

| Chapter | Topic | Java Version | Key Concepts |
|---------|-------|--------------|--------------|
| [1](chapters/01-var-type-inference.md) | Local Variable Type Inference | Java 10 | `var` keyword, when to use it |
| [2](chapters/02-text-blocks.md) | Text Blocks | Java 15 | Multi-line strings, formatting |
| [3](chapters/03-records.md) | Records | Java 16 | Immutable data carriers, compact constructors |
| [4](chapters/04-sealed-classes.md) | Sealed Classes | Java 17 | Restricted inheritance, domain modeling |
| [5](chapters/05-pattern-matching-instanceof.md) | Pattern Matching for instanceof | Java 16 | Type patterns, eliminating casts |
| [6](chapters/06-switch-expressions.md) | Switch Expressions & Patterns | Java 21 | Arrow syntax, pattern matching in switch |

### Part II: API Improvements

| Chapter | Topic | Java Version | Key Concepts |
|---------|-------|--------------|--------------|
| [7](chapters/07-collections-streams.md) | Enhanced Collections & Streams | Java 9-21 | Factory methods, new Stream operations |
| [8](chapters/08-optional-enhancements.md) | Optional Enhancements | Java 9-11 | `ifPresentOrElse`, `or`, `stream` |
| [9](chapters/09-http-client.md) | HTTP Client API | Java 11 | Modern HTTP, async requests |

### Part III: Concurrency Revolution

| Chapter | Topic | Java Version | Key Concepts |
|---------|-------|--------------|--------------|
| [10](chapters/10-virtual-threads.md) | Virtual Threads | Java 21 | Project Loom, scalable concurrency |

### Part IV: Developer Experience

| Chapter | Topic | Java Version | Key Concepts |
|---------|-------|--------------|--------------|
| [11](chapters/11-misc-improvements.md) | Helpful NullPointerExceptions & More | Java 14+ | Better errors, small improvements |
| [12](chapters/12-mini-project.md) | Putting It All Together | All | Real-world application |

---

## Quick Reference: Java Version Features

```
Java 9  (2017): Modules, JShell, Collection factories, Stream improvements
Java 10 (2018): var keyword
Java 11 (2018): HTTP Client, String methods, single-file execution (LTS)
Java 12 (2019): Switch expressions (preview)
Java 13 (2019): Text blocks (preview)
Java 14 (2020): Records (preview), Pattern matching (preview), Helpful NPE
Java 15 (2020): Text blocks (final), Sealed classes (preview)
Java 16 (2021): Records (final), Pattern matching instanceof (final)
Java 17 (2021): Sealed classes (final), Pattern matching switch (preview) (LTS)
Java 18 (2022): Simple web server, code snippets in Javadoc
Java 19 (2022): Virtual threads (preview), Record patterns (preview)
Java 20 (2023): Scoped values (preview)
Java 21 (2023): Virtual threads (final), Pattern matching switch (final) (LTS)
```

---

## Setting Up Your Workspace

Create a new directory for the exercises:

```bash
mkdir modern-java-practice
cd modern-java-practice
```

Each chapter has its own package. Create this structure:

```
modern-java-practice/
├── src/
│   ├── chapter01/
│   ├── chapter02/
│   └── ...
└── README.md
```

**Compile and run** (Java 11+):

```bash
# Single file execution (no explicit compile needed)
java src/chapter01/VarDemo.java

# Or traditional approach
javac -d out src/chapter01/*.java
java -cp out chapter01.VarDemo
```

---

## Let's Begin!

Start with [Chapter 1: Local Variable Type Inference](chapters/01-var-type-inference.md) and work through each chapter sequentially. The concepts build on each other, with the final chapter bringing everything together in a practical project.

**Happy coding!**
