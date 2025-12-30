# Chapter 11: Helpful NullPointerExceptions & More

> **Java Version:** 14-21 | **Difficulty:** Beginner | **Time:** 30 minutes

This chapter covers various quality-of-life improvements that make Java development more pleasant.

---

## Helpful NullPointerExceptions (Java 14)

### The Problem

Old NPE messages were frustrating:

```java
var user = getUser();
var street = user.getAddress().getCity().getStreet();
// Exception: NullPointerException: null
// Which one was null? user? address? city? 🤷
```

### The Solution

Java 14 provides detailed NPE messages:

```java
// Now you get:
// NullPointerException: Cannot invoke "Address.getCity()"
// because the return value of "User.getAddress()" is null
```

Create `src/chapter11/HelpfulNPE.java`:

```java
package chapter11;

public class HelpfulNPE {
    public static void main(String[] args) {
        // Enabled by default since Java 15
        // (Java 14 required -XX:+ShowCodeDetailsInExceptionMessages)

        System.out.println("=== Helpful NullPointerException Messages ===\n");

        // Example 1: Method chain
        System.out.println("Example 1: Method chain NPE");
        try {
            User user = new User("Alice", null);  // No address
            String street = user.address().city().street();
        } catch (NullPointerException e) {
            System.out.println("  Message: " + e.getMessage());
        }

        // Example 2: Array access
        System.out.println("\nExample 2: Array access NPE");
        try {
            String[] names = null;
            String first = names[0];
        } catch (NullPointerException e) {
            System.out.println("  Message: " + e.getMessage());
        }

        // Example 3: Field access
        System.out.println("\nExample 3: Field access NPE");
        try {
            User user = null;
            String name = user.name;
        } catch (NullPointerException e) {
            System.out.println("  Message: " + e.getMessage());
        }

        // Example 4: Complex expression
        System.out.println("\nExample 4: Complex expression NPE");
        try {
            var users = new User[]{null};
            String city = users[0].address().city().name;
        } catch (NullPointerException e) {
            System.out.println("  Message: " + e.getMessage());
        }
    }

    record Address(City city) {}
    record City(String name) {
        String street() { return "Main St"; }
    }

    static class User {
        String name;
        Address address;

        User(String name, Address address) {
            this.name = name;
            this.address = address;
        }

        Address address() { return address; }
    }
}
```

**Run it:**
```bash
java src/chapter11/HelpfulNPE.java
```

---

## String Methods (Java 11-15)

Create `src/chapter11/StringMethods.java`:

```java
package chapter11;

public class StringMethods {
    public static void main(String[] args) {
        // isBlank() - Java 11
        System.out.println("=== isBlank() (Java 11) ===");
        System.out.println("\"  \".isBlank() = " + "  ".isBlank());        // true
        System.out.println("\"\".isBlank() = " + "".isBlank());            // true
        System.out.println("\"hi\".isBlank() = " + "hi".isBlank());        // false
        System.out.println("\" hi \".isBlank() = " + " hi ".isBlank());    // false

        // lines() - Java 11
        System.out.println("\n=== lines() (Java 11) ===");
        String multiline = "Line 1\nLine 2\nLine 3";
        multiline.lines().forEach(line -> System.out.println("  " + line));

        // strip(), stripLeading(), stripTrailing() - Java 11
        System.out.println("\n=== strip() methods (Java 11) ===");
        String padded = "  \t Hello World \n  ";
        System.out.println("Original: [" + padded + "]");
        System.out.println("strip(): [" + padded.strip() + "]");
        System.out.println("stripLeading(): [" + padded.stripLeading() + "]");
        System.out.println("stripTrailing(): [" + padded.stripTrailing() + "]");
        // Note: strip() is Unicode-aware, trim() only handles ASCII whitespace

        // repeat() - Java 11
        System.out.println("\n=== repeat() (Java 11) ===");
        System.out.println("\"Hi \".repeat(3) = " + "Hi ".repeat(3));
        System.out.println("\"=\".repeat(20) = " + "=".repeat(20));

        // indent() - Java 12
        System.out.println("\n=== indent() (Java 12) ===");
        String code = "if (x) {\n    return y;\n}";
        System.out.println("Original:\n" + code);
        System.out.println("\nIndented by 4:\n" + code.indent(4));
        System.out.println("Dedented by 2:\n" + "    Hello\n    World".indent(-2));

        // transform() - Java 12
        System.out.println("=== transform() (Java 12) ===");
        String result = "  hello world  "
            .transform(String::strip)
            .transform(String::toUpperCase)
            .transform(s -> s.replace(" ", "_"));
        System.out.println("Transformed: " + result);

        // formatted() - Java 15 (already covered in text blocks chapter)
        System.out.println("\n=== formatted() (Java 15) ===");
        String template = "Hello, %s! You have %d messages.";
        System.out.println(template.formatted("Alice", 5));

        // stripIndent() and translateEscapes() - Java 15
        System.out.println("\n=== stripIndent() (Java 15) ===");
        String indented = "    line1\n    line2\n    line3";
        System.out.println("Before:\n" + indented);
        System.out.println("\nAfter stripIndent():\n" + indented.stripIndent());

        System.out.println("\n=== translateEscapes() (Java 15) ===");
        String escaped = "Hello\\nWorld\\tTab";
        System.out.println("Before: " + escaped);
        System.out.println("After:  " + escaped.translateEscapes());
    }
}
```

---

## Files Methods (Java 11-12)

Create `src/chapter11/FilesMethods.java`:

```java
package chapter11;

import java.nio.file.*;
import java.io.IOException;

public class FilesMethods {
    public static void main(String[] args) throws IOException {
        Path tempFile = Files.createTempFile("demo", ".txt");

        try {
            // writeString() and readString() - Java 11
            System.out.println("=== writeString/readString (Java 11) ===");

            // Old way:
            // Files.write(path, "content".getBytes());
            // String content = new String(Files.readAllBytes(path));

            // New way:
            Files.writeString(tempFile, "Hello, Java 11!\nSecond line.");
            String content = Files.readString(tempFile);
            System.out.println("Read: " + content);

            // isSameFile() - already existed but worth mentioning
            System.out.println("\n=== Path operations ===");
            Path path1 = Path.of("/tmp/../tmp/file.txt");
            Path path2 = Path.of("/tmp/file.txt");
            System.out.println("path1: " + path1);
            System.out.println("path2: " + path2);
            System.out.println("Normalized path1: " + path1.normalize());

            // mismatch() - Java 12
            System.out.println("\n=== mismatch() (Java 12) ===");
            Path file1 = Files.createTempFile("file1", ".txt");
            Path file2 = Files.createTempFile("file2", ".txt");

            Files.writeString(file1, "Hello World");
            Files.writeString(file2, "Hello World");

            long mismatch = Files.mismatch(file1, file2);
            System.out.println("Identical files mismatch: " + mismatch);  // -1 means identical

            Files.writeString(file2, "Hello Java!");
            mismatch = Files.mismatch(file1, file2);
            System.out.println("Different files mismatch at byte: " + mismatch);

            Files.delete(file1);
            Files.delete(file2);

        } finally {
            Files.deleteIfExists(tempFile);
        }
    }
}
```

---

## NumberFormat Improvements (Java 12)

Create `src/chapter11/NumberFormatting.java`:

```java
package chapter11;

import java.text.NumberFormat;
import java.util.Locale;

public class NumberFormatting {
    public static void main(String[] args) {
        // Compact number formatting - Java 12
        System.out.println("=== Compact Number Format (Java 12) ===\n");

        NumberFormat shortFormat = NumberFormat.getCompactNumberInstance(
            Locale.US, NumberFormat.Style.SHORT);

        NumberFormat longFormat = NumberFormat.getCompactNumberInstance(
            Locale.US, NumberFormat.Style.LONG);

        long[] numbers = {1, 100, 1_000, 10_000, 100_000, 1_000_000, 1_000_000_000};

        System.out.printf("%-15s %-10s %-15s%n", "Number", "Short", "Long");
        System.out.println("-".repeat(40));

        for (long n : numbers) {
            System.out.printf("%-15d %-10s %-15s%n",
                n,
                shortFormat.format(n),
                longFormat.format(n));
        }

        // Different locales
        System.out.println("\n=== Different Locales ===");

        long bigNumber = 1_500_000;
        String[] locales = {"en-US", "de-DE", "ja-JP", "zh-CN"};

        for (String localeStr : locales) {
            Locale locale = Locale.forLanguageTag(localeStr);
            NumberFormat fmt = NumberFormat.getCompactNumberInstance(
                locale, NumberFormat.Style.SHORT);
            System.out.printf("%s: %s%n", localeStr, fmt.format(bigNumber));
        }

        // With fractions
        System.out.println("\n=== With Fraction Digits ===");
        shortFormat.setMaximumFractionDigits(2);

        double[] decimals = {1234.5, 12345.67, 1_234_567.89};
        for (double d : decimals) {
            System.out.printf("%.2f -> %s%n", d, shortFormat.format(d));
        }
    }
}
```

---

## Process API Improvements (Java 9)

Create `src/chapter11/ProcessAPI.java`:

```java
package chapter11;

import java.time.Duration;
import java.time.Instant;

public class ProcessAPI {
    public static void main(String[] args) throws Exception {
        System.out.println("=== Process API (Java 9) ===\n");

        // Get current process info
        ProcessHandle current = ProcessHandle.current();
        ProcessHandle.Info info = current.info();

        System.out.println("Current Process:");
        System.out.println("  PID: " + current.pid());
        System.out.println("  Command: " + info.command().orElse("N/A"));
        System.out.println("  Arguments: " + info.arguments().map(java.util.Arrays::toString).orElse("N/A"));
        System.out.println("  User: " + info.user().orElse("N/A"));
        System.out.println("  Start time: " + info.startInstant().orElse(null));
        System.out.println("  CPU time: " + info.totalCpuDuration().orElse(Duration.ZERO));

        // List all processes (limited for demo)
        System.out.println("\n=== All Processes (first 5) ===");
        ProcessHandle.allProcesses()
            .limit(5)
            .forEach(ph -> {
                ProcessHandle.Info i = ph.info();
                System.out.printf("  PID %d: %s%n",
                    ph.pid(),
                    i.command().map(c -> c.substring(c.lastIndexOf('/') + 1)).orElse("?"));
            });

        // Start a process and get its handle
        System.out.println("\n=== Starting a Process ===");
        Process process = new ProcessBuilder("sleep", "1").start();
        ProcessHandle handle = process.toHandle();

        System.out.println("Started process PID: " + handle.pid());
        System.out.println("Is alive: " + handle.isAlive());

        // Wait for completion with timeout
        handle.onExit().thenAccept(ph ->
            System.out.println("Process " + ph.pid() + " exited"));

        process.waitFor();
        System.out.println("Process finished");

        // Parent/children relationships
        System.out.println("\n=== Process Hierarchy ===");
        current.parent().ifPresent(parent ->
            System.out.println("Parent PID: " + parent.pid()));

        System.out.println("Direct children count: " + current.children().count());
        System.out.println("All descendants count: " + current.descendants().count());
    }
}
```

---

## instanceof Pattern in switch null (Java 21)

Create `src/chapter11/ModernPatterns.java`:

```java
package chapter11;

public class ModernPatterns {
    public static void main(String[] args) {
        // null handling in switch - Java 21
        System.out.println("=== Null in Switch (Java 21) ===\n");

        Object[] values = {"hello", 42, null, 3.14, true};

        for (Object value : values) {
            String result = categorize(value);
            System.out.printf("%s -> %s%n",
                value == null ? "null" : value, result);
        }

        // Combined null and pattern
        System.out.println("\n=== Combined Patterns ===");
        testNullCases(null);
        testNullCases("");
        testNullCases("Hello");
    }

    static String categorize(Object obj) {
        return switch (obj) {
            case null -> "It's null!";
            case String s -> "String: " + s;
            case Integer i -> "Integer: " + i;
            case Double d -> "Double: " + d;
            default -> "Other: " + obj.getClass().getSimpleName();
        };
    }

    static void testNullCases(String s) {
        String result = switch (s) {
            case null -> "null input";
            case "" -> "empty string";
            case String str when str.length() < 5 -> "short: " + str;
            case String str -> "long: " + str;
        };
        System.out.printf("Input: %-10s -> %s%n",
            s == null ? "null" : "\"" + s + "\"", result);
    }
}
```

---

## Math Methods (Java 9-18)

Create `src/chapter11/MathMethods.java`:

```java
package chapter11;

import java.math.BigInteger;

public class MathMethods {
    public static void main(String[] args) {
        // Exact arithmetic methods (throw on overflow)
        System.out.println("=== Exact Arithmetic ===");

        try {
            int result = Math.multiplyExact(Integer.MAX_VALUE, 2);
        } catch (ArithmeticException e) {
            System.out.println("multiplyExact overflow: " + e.getMessage());
        }

        // floorDiv and floorMod (better behavior for negative numbers)
        System.out.println("\n=== floorDiv/floorMod ===");
        System.out.println("-7 / 3 = " + (-7 / 3));           // -2 (truncated)
        System.out.println("Math.floorDiv(-7, 3) = " + Math.floorDiv(-7, 3));  // -3
        System.out.println("-7 % 3 = " + (-7 % 3));           // -1
        System.out.println("Math.floorMod(-7, 3) = " + Math.floorMod(-7, 3));  // 2

        // fma - fused multiply-add (Java 9)
        System.out.println("\n=== Fused Multiply-Add (Java 9) ===");
        double a = 1.0, b = 2.0, c = 3.0;
        System.out.println("a*b + c = " + (a * b + c));
        System.out.println("Math.fma(a, b, c) = " + Math.fma(a, b, c));
        // More accurate for floating point, especially near limits

        // clamp (Java 21)
        System.out.println("\n=== clamp (Java 21) ===");
        int value = 150;
        int min = 0;
        int max = 100;

        int clamped = Math.clamp(value, min, max);
        System.out.printf("clamp(%d, %d, %d) = %d%n", value, min, max, clamped);

        System.out.println("clamp(50, 0, 100) = " + Math.clamp(50, 0, 100));   // 50
        System.out.println("clamp(-10, 0, 100) = " + Math.clamp(-10, 0, 100)); // 0
        System.out.println("clamp(150, 0, 100) = " + Math.clamp(150, 0, 100)); // 100

        // Works with long and double too
        System.out.println("clamp(1.5, 0.0, 1.0) = " + Math.clamp(1.5, 0.0, 1.0));

        // BigInteger improvements
        System.out.println("\n=== BigInteger Improvements ===");
        BigInteger big = BigInteger.valueOf(1000);
        System.out.println("sqrt(1000) = " + big.sqrt());  // Java 9
        System.out.println("1000^0.5 approximation = " + Math.sqrt(1000));
    }
}
```

---

## Misc Improvements Summary

| Feature | Version | Description |
|---------|---------|-------------|
| Helpful NPE | 14 | Detailed null pointer messages |
| `String.isBlank()` | 11 | Check for whitespace-only |
| `String.lines()` | 11 | Stream of lines |
| `String.strip()` | 11 | Unicode-aware trim |
| `String.repeat()` | 11 | Repeat string n times |
| `String.indent()` | 12 | Add/remove indentation |
| `String.transform()` | 12 | Function application |
| `Files.readString()` | 11 | Read file as String |
| `Files.writeString()` | 11 | Write String to file |
| `Files.mismatch()` | 12 | Find first difference |
| Compact NumberFormat | 12 | 1K, 1M formatting |
| Process API | 9 | Process info and control |
| `Math.clamp()` | 21 | Constrain to range |

---

## Practice Exercises

### Exercise 11.1: Debug Helper
Create a method that catches any exception and prints a helpful debug message including the stack trace with line numbers.

<details>
<summary>Click for solution</summary>

```java
static void debugException(Exception e) {
    System.out.println("Exception: " + e.getClass().getSimpleName());
    System.out.println("Message: " + e.getMessage());
    System.out.println("Stack trace:");

    for (StackTraceElement element : e.getStackTrace()) {
        System.out.printf("  at %s.%s(%s:%d)%n",
            element.getClassName(),
            element.getMethodName(),
            element.getFileName(),
            element.getLineNumber());
    }
}
```
</details>

### Exercise 11.2: Text Processor
Write a method that takes a multi-line string and:
1. Removes blank lines
2. Trims each line
3. Numbers each line
4. Returns as a single string

<details>
<summary>Click for solution</summary>

```java
static String processText(String input) {
    var counter = new int[]{0};  // Mutable counter for lambda

    return input.lines()
        .filter(line -> !line.isBlank())
        .map(String::strip)
        .map(line -> ++counter[0] + ": " + line)
        .reduce((a, b) -> a + "\n" + b)
        .orElse("");
}
```
</details>

---

## Key Takeaways

1. **Helpful NPE messages** save debugging time
2. **String methods** reduce boilerplate (`isBlank`, `repeat`, `indent`)
3. **Files methods** simplify file I/O (`readString`, `writeString`)
4. **Compact NumberFormat** for user-friendly numbers
5. **Process API** for better process management
6. **Math.clamp()** for value constraint

---

## What's Next?

In [Chapter 12: Putting It All Together](12-mini-project.md), you'll build a complete application using all the modern Java features you've learned.

---

[← Previous Chapter](10-virtual-threads.md) | [Back to Contents](../README.md) | [Next Chapter →](12-mini-project.md)
