# Chapter 1: Local Variable Type Inference (`var`)

> **Java Version:** 10+ | **Difficulty:** Beginner | **Time:** 30 minutes

## The Problem

You're writing code that processes a map of user data. Look at this Java 8 style code:

```java
Map<String, List<UserTransaction>> userTransactions = new HashMap<String, List<UserTransaction>>();
```

The type information is repeated twice. It's verbose, and when types get complex, the code becomes hard to read.

## The Solution: `var`

Java 10 introduced **local variable type inference** with the `var` keyword. The compiler infers the type from the right-hand side of the assignment.

---

## Hands-On Exercise 1: Your First `var`

Create a file `src/chapter01/VarBasics.java`:

```java
package chapter01;

import java.util.*;

public class VarBasics {
    public static void main(String[] args) {
        // Before Java 10: Explicit type declaration
        String message = "Hello, Modern Java!";
        List<String> names = new ArrayList<String>();
        Map<String, Integer> scores = new HashMap<String, Integer>();

        // After Java 10: Type inference with var
        var modernMessage = "Hello, Modern Java!";
        var modernNames = new ArrayList<String>();
        var modernScores = new HashMap<String, Integer>();

        // The types are the same!
        System.out.println("message type: " + message.getClass().getName());
        System.out.println("modernMessage type: " + modernMessage.getClass().getName());

        // var works with any type
        var number = 42;              // int
        var decimal = 3.14;           // double
        var flag = true;              // boolean
        var character = 'A';          // char

        System.out.println("\nInferred types:");
        System.out.println("number: " + ((Object) number).getClass().getName());
        System.out.println("decimal: " + ((Object) decimal).getClass().getName());
        System.out.println("flag: " + ((Object) flag).getClass().getName());
    }
}
```

**Run it:**
```bash
java src/chapter01/VarBasics.java
```

**Expected Output:**
```
message type: java.lang.String
modernMessage type: java.lang.String

Inferred types:
number: java.lang.Integer
decimal: java.lang.Double
flag: java.lang.Boolean
```

---

## Key Insight: `var` is NOT Dynamic Typing

This is crucial to understand:

```java
var text = "Hello";    // text is String, determined at compile time
text = 123;            // COMPILE ERROR! Cannot assign int to String
```

The type is **inferred at compile time** and **fixed**. Java remains statically typed.

---

## Hands-On Exercise 2: Where `var` Shines

Create `src/chapter01/VarShines.java`:

```java
package chapter01;

import java.util.*;
import java.util.stream.*;
import java.io.*;
import java.nio.file.*;

public class VarShines {
    public static void main(String[] args) {
        // Example 1: Complex generic types
        // Before: Type is repeated and verbose
        Map<String, List<Map<String, Integer>>> complexBefore =
            new HashMap<String, List<Map<String, Integer>>>();

        // After: Clean and readable
        var complexAfter = new HashMap<String, List<Map<String, Integer>>>();

        // Example 2: Stream intermediate results
        var numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // Without var, you'd write:
        // Stream<Integer> evenStream = numbers.stream().filter(n -> n % 2 == 0);

        // With var:
        var evenStream = numbers.stream().filter(n -> n % 2 == 0);
        var evenSquared = evenStream.map(n -> n * n);
        var result = evenSquared.collect(Collectors.toList());

        System.out.println("Even numbers squared: " + result);

        // Example 3: Try-with-resources (cleaner)
        var path = Path.of("test.txt");
        try {
            Files.writeString(path, "Hello from var!");

            // var in try-with-resources
            try (var reader = Files.newBufferedReader(path)) {
                var line = reader.readLine();
                System.out.println("Read from file: " + line);
            }
        } catch (IOException e) {
            System.out.println("File operations: " + e.getMessage());
        } finally {
            try { Files.deleteIfExists(path); } catch (IOException ignored) {}
        }

        // Example 4: For loops
        var items = List.of("apple", "banana", "cherry");

        System.out.println("\nIterating with var:");
        for (var item : items) {
            System.out.println("  - " + item.toUpperCase());
        }

        // Traditional for loop
        for (var i = 0; i < items.size(); i++) {
            System.out.println("  [" + i + "] " + items.get(i));
        }
    }
}
```

**Run it:**
```bash
java src/chapter01/VarShines.java
```

---

## Hands-On Exercise 3: Where NOT to Use `var`

Create `src/chapter01/VarPitfalls.java`:

```java
package chapter01;

import java.util.*;

public class VarPitfalls {
    public static void main(String[] args) {
        // PITFALL 1: Unclear types from method calls
        // Bad: What type is 'result'?
        var result = processData();  // Reader can't tell the type

        // Better: Be explicit when the type isn't obvious
        String processedResult = processData();

        // PITFALL 2: Numeric literals can be confusing
        var amount = 100;        // int
        var price = 100L;        // long
        var rate = 100.0;        // double
        var percentage = 100.0f; // float

        System.out.println("amount type: " + ((Object) amount).getClass().getSimpleName());
        System.out.println("price type: " + ((Object) price).getClass().getSimpleName());
        System.out.println("rate type: " + ((Object) rate).getClass().getSimpleName());
        System.out.println("percentage type: " + ((Object) percentage).getClass().getSimpleName());

        // PITFALL 3: Diamond operator ambiguity
        var list1 = new ArrayList<>();           // ArrayList<Object> - probably not what you want!
        var list2 = new ArrayList<String>();     // ArrayList<String> - explicit is better here

        list1.add("string");
        list1.add(123);  // This compiles! list1 is ArrayList<Object>

        // list2.add(123);  // This would NOT compile - list2 is ArrayList<String>

        System.out.println("\nlist1 (ArrayList<?>): " + list1);
        System.out.println("list2 (ArrayList<String>): " + list2);

        // PITFALL 4: Reduced readability with long method chains
        var data = fetchUsers().stream()
            .filter(u -> u.isActive())
            .map(User::getName)
            .collect(Collectors.toList());  // What's the type? Hard to tell without IDE

        // Sometimes explicit is better for documentation:
        List<String> activeUserNames = fetchUsers().stream()
            .filter(u -> u.isActive())
            .map(User::getName)
            .collect(Collectors.toList());  // Clear: it's a List<String>
    }

    static String processData() {
        return "processed";
    }

    static List<User> fetchUsers() {
        return List.of(
            new User("Alice", true),
            new User("Bob", false),
            new User("Charlie", true)
        );
    }

    record User(String name, boolean active) {
        String getName() { return name; }
        boolean isActive() { return active; }
    }
}
```

---

## Where `var` CANNOT Be Used

```java
public class VarRestrictions {
    // var count = 0;              // ERROR: Cannot use var for fields

    // public void process(var input) {}  // ERROR: Cannot use var for parameters

    // public var getData() {}     // ERROR: Cannot use var for return types

    public void examples() {
        // var x;                  // ERROR: Must have initializer
        // var nothing = null;     // ERROR: Cannot infer type from null
        // var arr = {1, 2, 3};    // ERROR: Cannot use with array initializers

        // These are all valid:
        var x = 10;
        var arr = new int[]{1, 2, 3};
        Object nothing = null;  // Use explicit type for null
    }
}
```

---

## Best Practices Summary

| Do Use `var` | Don't Use `var` |
|--------------|-----------------|
| When the type is obvious from the right side | When the type isn't clear |
| For complex generic types | For primitive types where the literal is ambiguous |
| In for-each loops | When it reduces code readability |
| With try-with-resources | For fields, parameters, or return types (not allowed) |
| When the variable name is descriptive | With diamond operator `<>` alone |

### The Golden Rule

> **Use `var` when it improves readability; avoid it when it obscures intent.**

---

## Practice Exercises

### Exercise 1.1: Refactor with `var`
Refactor this code to use `var` where appropriate:

```java
public class Exercise1 {
    public static void main(String[] args) {
        HashMap<String, ArrayList<Integer>> scores = new HashMap<String, ArrayList<Integer>>();
        ArrayList<Integer> aliceScores = new ArrayList<Integer>();
        aliceScores.add(95);
        aliceScores.add(87);
        aliceScores.add(92);
        scores.put("Alice", aliceScores);

        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader("data.txt"));
            String line = reader.readLine();
            System.out.println(line);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### Exercise 1.2: Identify the Types
What types will the compiler infer for each variable?

```java
var a = List.of(1, 2, 3);
var b = new ArrayList<>();
var c = "Hello".length();
var d = Map.entry("key", 42);
var e = 1 + 2.0;
```

<details>
<summary>Click for answers</summary>

```
a: List<Integer>
b: ArrayList<Object>
c: int
d: Map.Entry<String, Integer>
e: double (int promoted to double for addition)
```
</details>

### Exercise 1.3: Fix the Errors
This code has errors. Fix them:

```java
public class BrokenVar {
    var instanceField = "hello";  // Error 1

    public var getData() {        // Error 2
        return "data";
    }

    public void process() {
        var x;                    // Error 3
        var y = null;             // Error 4
    }
}
```

<details>
<summary>Click for solution</summary>

```java
public class FixedVar {
    String instanceField = "hello";  // var not allowed for fields

    public String getData() {        // var not allowed for return types
        return "data";
    }

    public void process() {
        var x = 0;                   // var requires initializer
        Object y = null;             // Use explicit type for null
    }
}
```
</details>

---

## Key Takeaways

1. **`var` is for local variables only** - not fields, parameters, or return types
2. **Type is inferred at compile time** - Java remains statically typed
3. **Requires an initializer** - the compiler needs to see what type to infer
4. **Use when it improves readability** - especially for complex generic types
5. **Avoid when it obscures intent** - especially for primitive literals or unclear method returns

---

## What's Next?

In [Chapter 2: Text Blocks](02-text-blocks.md), you'll learn how to write multi-line strings elegantly without escape character nightmares.

---

[← Back to Table of Contents](../README.md) | [Next Chapter →](02-text-blocks.md)
