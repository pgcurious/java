# Chapter 2: Text Blocks

> **Java Version:** 15+ | **Difficulty:** Beginner | **Time:** 25 minutes

## The Problem

You're building an application that needs to work with multi-line strings: JSON payloads, SQL queries, HTML templates, or configuration files. Here's what you had to write in Java 8:

```java
String json = "{\n" +
    "  \"name\": \"John Doe\",\n" +
    "  \"email\": \"john@example.com\",\n" +
    "  \"age\": 30,\n" +
    "  \"address\": {\n" +
    "    \"street\": \"123 Main St\",\n" +
    "    \"city\": \"Springfield\"\n" +
    "  }\n" +
    "}";
```

Escape characters everywhere! Hard to read, easy to make mistakes.

## The Solution: Text Blocks

Java 15 introduced **text blocks** - multi-line string literals that preserve formatting and reduce escape character noise.

```java
String json = """
    {
      "name": "John Doe",
      "email": "john@example.com",
      "age": 30,
      "address": {
        "street": "123 Main St",
        "city": "Springfield"
      }
    }
    """;
```

Clean, readable, and exactly what you'd expect!

---

## Hands-On Exercise 1: Basic Text Blocks

Create `src/chapter02/TextBlockBasics.java`:

```java
package chapter02;

public class TextBlockBasics {
    public static void main(String[] args) {
        // Old way: Concatenation nightmare
        String oldWay = "Line 1\n" +
                        "Line 2\n" +
                        "Line 3";

        // New way: Text block
        String newWay = """
                Line 1
                Line 2
                Line 3""";

        System.out.println("=== Old Way ===");
        System.out.println(oldWay);

        System.out.println("\n=== New Way (Text Block) ===");
        System.out.println(newWay);

        // They produce the same result!
        System.out.println("\n=== Are they equal? ===");
        System.out.println("Equal: " + oldWay.equals(newWay));

        // Text blocks are just Strings
        System.out.println("\n=== Text blocks are Strings ===");
        String textBlock = """
                Hello, World!""";
        System.out.println("Type: " + textBlock.getClass().getName());
        System.out.println("Length: " + textBlock.length());
        System.out.println("Uppercase: " + textBlock.toUpperCase());
    }
}
```

**Run it:**
```bash
java src/chapter02/TextBlockBasics.java
```

---

## The Rules of Text Blocks

### Rule 1: Opening delimiter must be followed by a newline

```java
// CORRECT: Content starts on the next line
String valid = """
    content here
    """;

// WRONG: Content cannot be on the same line as opening """
String invalid = """content here""";  // COMPILE ERROR!
```

### Rule 2: Closing delimiter position controls trailing newline

```java
package chapter02;

public class TrailingNewlines {
    public static void main(String[] args) {
        // No trailing newline - closing """ on last content line
        String noTrailing = """
                Hello World""";

        // Trailing newline - closing """ on its own line
        String withTrailing = """
                Hello World
                """;

        System.out.println("Without trailing newline:");
        System.out.println("[" + noTrailing + "]");
        System.out.println("Length: " + noTrailing.length());

        System.out.println("\nWith trailing newline:");
        System.out.println("[" + withTrailing + "]");
        System.out.println("Length: " + withTrailing.length());
    }
}
```

---

## Hands-On Exercise 2: Indentation Magic

Create `src/chapter02/IndentationDemo.java`:

```java
package chapter02;

public class IndentationDemo {
    public static void main(String[] args) {
        // The closing """ position determines the "margin"
        // Everything to the left of it is incidental indentation (removed)
        // Everything to the right is essential indentation (kept)

        // Example 1: Closing at column 0 - all indentation is kept
        String keepAll = """
    Line with 4 spaces
        Line with 8 spaces
""";
        System.out.println("=== Keep All Indentation ===");
        System.out.println(keepAll);
        showSpaces(keepAll);

        // Example 2: Closing aligned with content - no leading spaces
        String noLeading = """
                Line 1
                Line 2
                Line 3
                """;
        System.out.println("=== No Leading Spaces ===");
        System.out.println(noLeading);
        showSpaces(noLeading);

        // Example 3: Some lines indented relative to others
        String mixedIndent = """
                {
                    "nested": true,
                    "items": [
                        "one",
                        "two"
                    ]
                }
                """;
        System.out.println("=== Mixed Indentation (JSON) ===");
        System.out.println(mixedIndent);
    }

    static void showSpaces(String s) {
        String visual = s.replace(" ", "·").replace("\n", "↵\n");
        System.out.println("Visualization: " + visual);
        System.out.println();
    }
}
```

**Key Insight:** The compiler determines "incidental whitespace" by finding the leftmost non-whitespace character across all lines (including the closing `"""`). Everything to the left of that column is removed.

---

## Hands-On Exercise 3: Real-World Use Cases

Create `src/chapter02/RealWorldTextBlocks.java`:

```java
package chapter02;

public class RealWorldTextBlocks {
    public static void main(String[] args) {
        // Use Case 1: SQL Queries
        String sql = """
                SELECT u.id, u.name, u.email, o.total
                FROM users u
                JOIN orders o ON u.id = o.user_id
                WHERE o.status = 'completed'
                  AND o.created_at > '2024-01-01'
                ORDER BY o.total DESC
                LIMIT 10
                """;
        System.out.println("=== SQL Query ===");
        System.out.println(sql);

        // Use Case 2: JSON
        String jsonTemplate = """
                {
                    "apiVersion": "v1",
                    "kind": "ConfigMap",
                    "metadata": {
                        "name": "app-config",
                        "namespace": "production"
                    },
                    "data": {
                        "database.url": "jdbc:postgresql://localhost:5432/mydb",
                        "cache.enabled": "true"
                    }
                }
                """;
        System.out.println("=== JSON ===");
        System.out.println(jsonTemplate);

        // Use Case 3: HTML
        String html = """
                <!DOCTYPE html>
                <html>
                <head>
                    <title>Welcome</title>
                    <style>
                        body { font-family: Arial, sans-serif; }
                        .greeting { color: #333; }
                    </style>
                </head>
                <body>
                    <h1 class="greeting">Hello, World!</h1>
                </body>
                </html>
                """;
        System.out.println("=== HTML ===");
        System.out.println(html);

        // Use Case 4: ASCII Art / Documentation
        String asciiArt = """

                    ╔═══════════════════════════════════╗
                    ║     WELCOME TO MODERN JAVA!       ║
                    ║   Text blocks make life easier    ║
                    ╚═══════════════════════════════════╝

                """;
        System.out.println("=== ASCII Art ===");
        System.out.println(asciiArt);

        // Use Case 5: Regular Expressions (no double escaping!)
        // Old way: String pattern = "\\d{3}-\\d{2}-\\d{4}";
        // New way (same, but cleaner for complex patterns):
        String regexPattern = """
                ^(?<areaCode>\\d{3})-(?<exchange>\\d{3})-(?<subscriber>\\d{4})$""";
        System.out.println("=== Regex Pattern ===");
        System.out.println(regexPattern);
    }
}
```

---

## Hands-On Exercise 4: Escape Sequences in Text Blocks

Create `src/chapter02/EscapeSequences.java`:

```java
package chapter02;

public class EscapeSequences {
    public static void main(String[] args) {
        // Most characters don't need escaping in text blocks
        String noEscapeNeeded = """
                Quotes are "easy" now!
                Backslash \\ still needs escaping
                Tab	character works naturally
                """;
        System.out.println("=== Natural Characters ===");
        System.out.println(noEscapeNeeded);

        // Triple quotes inside text blocks need escaping
        String tripleQuotes = """
                To write text blocks, use:
                String s = \"""
                    content
                    \""";
                """;
        System.out.println("=== Escaped Triple Quotes ===");
        System.out.println(tripleQuotes);

        // New escape sequences (Java 14+)

        // \s = space that prevents trailing whitespace stripping
        String preserveSpaces = """
                Name:     \s
                Address:  \s
                """;
        System.out.println("=== Preserved Trailing Spaces (\\s) ===");
        System.out.println("[" + preserveSpaces.replace(" ", "·") + "]");

        // \<newline> = line continuation (no newline in output)
        String singleLine = """
                This is a very long line that we want to \
                break in the source code for readability, \
                but it should appear as one line in output.""";
        System.out.println("=== Line Continuation (\\<newline>) ===");
        System.out.println(singleLine);
        System.out.println("Contains newline: " + singleLine.contains("\n"));
    }
}
```

---

## Hands-On Exercise 5: String Formatting with Text Blocks

Create `src/chapter02/FormattedTextBlocks.java`:

```java
package chapter02;

public class FormattedTextBlocks {
    public static void main(String[] args) {
        String name = "Alice";
        int age = 28;
        String city = "New York";

        // Method 1: String.format()
        String template1 = """
                User Profile
                ============
                Name: %s
                Age: %d years old
                City: %s
                """;
        String result1 = String.format(template1, name, age, city);
        System.out.println("=== Using String.format() ===");
        System.out.println(result1);

        // Method 2: formatted() method (Java 15+) - more readable!
        String result2 = """
                User Profile
                ============
                Name: %s
                Age: %d years old
                City: %s
                """.formatted(name, age, city);
        System.out.println("=== Using .formatted() ===");
        System.out.println(result2);

        // Method 3: String concatenation/replacement
        String result3 = """
                User Profile
                ============
                Name: ${name}
                Age: ${age} years old
                City: ${city}
                """
                .replace("${name}", name)
                .replace("${age}", String.valueOf(age))
                .replace("${city}", city);
        System.out.println("=== Using .replace() ===");
        System.out.println(result3);

        // Practical example: Building JSON
        record User(String name, String email, int age) {}

        User user = new User("Bob", "bob@example.com", 35);

        String json = """
                {
                    "name": "%s",
                    "email": "%s",
                    "age": %d,
                    "active": true
                }
                """.formatted(user.name(), user.email(), user.age());
        System.out.println("=== Dynamic JSON ===");
        System.out.println(json);
    }
}
```

---

## Text Block Methods

Java added helpful methods to the String class:

```java
package chapter02;

public class TextBlockMethods {
    public static void main(String[] args) {
        String text = "  Hello World  ";

        // stripIndent() - removes incidental whitespace (like text block processing)
        String indented = "    line 1\n    line 2\n    line 3";
        System.out.println("stripIndent():");
        System.out.println(indented.stripIndent());

        // translateEscapes() - processes escape sequences in a string
        String withEscapes = "Hello\\nWorld\\tTab";
        System.out.println("\ntranslateEscapes():");
        System.out.println("Before: " + withEscapes);
        System.out.println("After: " + withEscapes.translateEscapes());

        // indent(n) - adds n spaces to each line (or removes if negative)
        String lines = "A\nB\nC";
        System.out.println("\nindent(4):");
        System.out.println(lines.indent(4));

        System.out.println("indent(-2) on indented text:");
        System.out.println("    X\n    Y\n    Z".indent(-2));
    }
}
```

---

## Common Mistakes

### Mistake 1: Content on same line as opening delimiter

```java
// WRONG
String bad = """Hello""";

// CORRECT
String good = """
        Hello""";
```

### Mistake 2: Inconsistent indentation

```java
// Confusing - mixed tabs and spaces
String confusing = """
		Tab indented
        Space indented
        """;

// Better - consistent spaces
String clear = """
        Space indented
        Space indented
        """;
```

### Mistake 3: Forgetting that text blocks are still Strings

```java
// Text blocks support ALL String operations
String upper = """
        hello""".toUpperCase();

String repeated = """
        Hi! """.repeat(3);

boolean empty = """
        """.isEmpty();  // false - contains a newline!
```

---

## Practice Exercises

### Exercise 2.1: Convert to Text Block
Convert this old-style string to a text block:

```java
String email = "Dear " + customerName + ",\n\n" +
    "Thank you for your order #" + orderId + ".\n" +
    "Your items will be shipped within 3-5 business days.\n\n" +
    "Best regards,\n" +
    "The Store Team";
```

<details>
<summary>Click for solution</summary>

```java
String email = """
        Dear %s,

        Thank you for your order #%s.
        Your items will be shipped within 3-5 business days.

        Best regards,
        The Store Team
        """.formatted(customerName, orderId);
```
</details>

### Exercise 2.2: Fix the Text Block
This code has issues. Fix them:

```java
String broken = """SELECT * FROM users WHERE name = "John"""";
```

<details>
<summary>Click for solution</summary>

```java
String fixed = """
        SELECT * FROM users WHERE name = "John"
        """;
// Or without trailing newline:
String fixed2 = """
        SELECT * FROM users WHERE name = "John\"""";
```
</details>

### Exercise 2.3: Create an HTML Template
Create a text block for an HTML email template with placeholders for: `{recipient}`, `{subject}`, and `{body}`. Then use `replace()` to fill in the values.

<details>
<summary>Click for solution</summary>

```java
String template = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>{subject}</title>
        </head>
        <body>
            <h1>Hello, {recipient}!</h1>
            <div class="content">
                {body}
            </div>
        </body>
        </html>
        """;

String email = template
    .replace("{recipient}", "Alice")
    .replace("{subject}", "Welcome!")
    .replace("{body}", "Thanks for signing up.");

System.out.println(email);
```
</details>

---

## Key Takeaways

1. **Text blocks start with `"""`** followed by a newline, content, then closing `"""`
2. **Indentation is smart** - incidental whitespace is automatically removed
3. **Closing delimiter position matters** - controls indentation and trailing newline
4. **New escape sequences** - `\s` preserves spaces, `\<newline>` continues lines
5. **Use `.formatted()`** for dynamic content (cleaner than `String.format()`)
6. **Text blocks are just Strings** - all String methods work on them

---

## What's Next?

In [Chapter 3: Records](03-records.md), you'll discover how to create immutable data classes with minimal boilerplate.

---

[← Previous Chapter](01-var-type-inference.md) | [Back to Contents](../README.md) | [Next Chapter →](03-records.md)
