# Chapter 5: Pattern Matching for instanceof

> **Java Version:** 16+ | **Difficulty:** Beginner-Intermediate | **Time:** 30 minutes

## The Problem

Type checking in Java has always been verbose. You check the type, then cast:

```java
// Old way - check and cast separately
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}
```

The cast feels redundant - you just checked that `obj` is a `String`!

## The Solution: Pattern Matching for instanceof

Java 16 introduced **pattern matching** for `instanceof`. Now you can check and bind in one expression:

```java
// New way - check, cast, and bind in one step
if (obj instanceof String s) {
    System.out.println(s.length());  // s is already a String!
}
```

The variable `s` is automatically typed as `String` and available in the `if` block.

---

## Hands-On Exercise 1: Basic Pattern Matching

Create `src/chapter05/PatternMatchingBasics.java`:

```java
package chapter05;

public class PatternMatchingBasics {
    public static void main(String[] args) {
        Object[] items = {
            "Hello, World!",
            42,
            3.14159,
            true,
            new int[]{1, 2, 3},
            null
        };

        System.out.println("=== Old Way vs New Way ===\n");

        for (Object item : items) {
            // Old way
            if (item instanceof String) {
                String s = (String) item;
                System.out.println("Old: String of length " + s.length());
            }

            // New way - pattern matching
            if (item instanceof String s) {
                System.out.println("New: String of length " + s.length());
            }

            if (item instanceof Integer i) {
                System.out.println("New: Integer value is " + i);
            }

            if (item instanceof Double d) {
                System.out.println("New: Double value is " + d);
            }

            if (item instanceof Boolean b) {
                System.out.println("New: Boolean value is " + b);
            }

            if (item instanceof int[] arr) {
                System.out.println("New: int[] of length " + arr.length);
            }

            if (item == null) {
                System.out.println("New: It's null");
            }

            System.out.println();
        }
    }
}
```

**Run it:**
```bash
java src/chapter05/PatternMatchingBasics.java
```

---

## Variable Scope: Flow-Sensitive Typing

The pattern variable is in scope **where the compiler can prove the pattern matched**:

Create `src/chapter05/PatternScope.java`:

```java
package chapter05;

public class PatternScope {
    public static void main(String[] args) {
        Object obj = "Hello";

        // Pattern variable 's' is in scope inside the if block
        if (obj instanceof String s) {
            System.out.println("Length: " + s.length());  // OK
        }
        // System.out.println(s);  // ERROR: 's' not in scope here

        // With negation - variable is in scope in the else branch
        if (!(obj instanceof String s)) {
            System.out.println("Not a string");
        } else {
            System.out.println("String: " + s);  // OK - we know it's a String here
        }

        // With && operator
        if (obj instanceof String s && s.length() > 5) {
            System.out.println("Long string: " + s);
        }

        // With || operator - CAREFUL!
        // This would NOT compile:
        // if (obj instanceof String s || s.isEmpty()) { }
        // Because 's' might not be defined if first condition is false

        // This pattern is useful
        if (!(obj instanceof String s) || s.isEmpty()) {
            System.out.println("Not a string or empty");
        } else {
            System.out.println("Non-empty string: " + s);
        }

        // Early return pattern
        String result = processObject(obj);
        System.out.println("Result: " + result);
    }

    static String processObject(Object obj) {
        // Guard clause with pattern matching
        if (!(obj instanceof String s)) {
            return "Not a string";
        }
        // Here, 's' is in scope for the rest of the method!
        return "String of length " + s.length() + ": " + s.toUpperCase();
    }
}
```

---

## Hands-On Exercise 2: Real-World Patterns

Create `src/chapter05/RealWorldPatterns.java`:

```java
package chapter05;

import java.util.*;

public class RealWorldPatterns {
    public static void main(String[] args) {
        // Example 1: Processing mixed collections
        System.out.println("=== Processing Mixed Collection ===");
        List<Object> mixedList = List.of(
            "apple",
            42,
            List.of(1, 2, 3),
            Map.of("key", "value"),
            3.14
        );

        for (Object item : mixedList) {
            describeItem(item);
        }

        // Example 2: Equals implementation
        System.out.println("\n=== Custom Equals ===");
        var point1 = new Point(3, 4);
        var point2 = new Point(3, 4);
        var point3 = new Point(5, 6);

        System.out.println("point1.equals(point2): " + point1.equals(point2));
        System.out.println("point1.equals(point3): " + point1.equals(point3));
        System.out.println("point1.equals(\"hello\"): " + point1.equals("hello"));
        System.out.println("point1.equals(null): " + point1.equals(null));

        // Example 3: Parsing input
        System.out.println("\n=== Parsing Input ===");
        Object[] inputs = {123, "456", "hello", 78.9, null};
        for (Object input : inputs) {
            int value = parseToInt(input);
            System.out.println(input + " -> " + value);
        }
    }

    static void describeItem(Object item) {
        String description;

        if (item instanceof String s) {
            description = "String: \"" + s + "\" (length=" + s.length() + ")";
        } else if (item instanceof Integer i) {
            description = "Integer: " + i + " (even=" + (i % 2 == 0) + ")";
        } else if (item instanceof List<?> list) {
            description = "List with " + list.size() + " elements";
        } else if (item instanceof Map<?, ?> map) {
            description = "Map with " + map.size() + " entries";
        } else if (item instanceof Double d) {
            description = "Double: " + String.format("%.2f", d);
        } else {
            description = "Unknown type: " + item.getClass().getSimpleName();
        }

        System.out.println("  " + description);
    }

    static int parseToInt(Object input) {
        if (input instanceof Integer i) {
            return i;
        } else if (input instanceof String s && s.matches("\\d+")) {
            return Integer.parseInt(s);
        } else if (input instanceof Number n) {
            return n.intValue();
        }
        return 0;  // default
    }
}

// Custom class to demonstrate equals()
class Point {
    private final int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object obj) {
        // Old way:
        // if (obj == null || getClass() != obj.getClass()) return false;
        // Point other = (Point) obj;
        // return x == other.x && y == other.y;

        // New way with pattern matching:
        return obj instanceof Point other
            && this.x == other.x
            && this.y == other.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
}
```

---

## Hands-On Exercise 3: Combining with Sealed Types

Create `src/chapter05/SealedPatternMatching.java`:

```java
package chapter05;

sealed interface Shape permits Circle, Rectangle, Triangle {}

record Circle(double radius) implements Shape {}
record Rectangle(double width, double height) implements Shape {}
record Triangle(double base, double height) implements Shape {}

public class SealedPatternMatching {
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle(5.0),
            new Rectangle(4.0, 6.0),
            new Triangle(3.0, 4.0)
        };

        System.out.println("=== Shape Calculations ===\n");

        for (Shape shape : shapes) {
            System.out.println("Shape: " + shape);
            System.out.printf("  Area: %.2f%n", calculateArea(shape));
            System.out.printf("  Perimeter: %.2f%n", calculatePerimeter(shape));
            System.out.println();
        }

        // Example: Getting specific data
        System.out.println("=== Extracting Specific Data ===");
        for (Shape shape : shapes) {
            if (shape instanceof Circle c) {
                System.out.println("Circle diameter: " + (c.radius() * 2));
            } else if (shape instanceof Rectangle r) {
                System.out.println("Rectangle diagonal: " +
                    Math.sqrt(r.width() * r.width() + r.height() * r.height()));
            } else if (shape instanceof Triangle t) {
                System.out.println("Triangle hypotenuse (if right): " +
                    Math.sqrt(t.base() * t.base() + t.height() * t.height()));
            }
        }
    }

    static double calculateArea(Shape shape) {
        if (shape instanceof Circle c) {
            return Math.PI * c.radius() * c.radius();
        } else if (shape instanceof Rectangle r) {
            return r.width() * r.height();
        } else if (shape instanceof Triangle t) {
            return 0.5 * t.base() * t.height();
        }
        throw new IllegalArgumentException("Unknown shape");
    }

    static double calculatePerimeter(Shape shape) {
        if (shape instanceof Circle c) {
            return 2 * Math.PI * c.radius();
        } else if (shape instanceof Rectangle r) {
            return 2 * (r.width() + r.height());
        } else if (shape instanceof Triangle t) {
            // Assuming right triangle for simplicity
            double hypotenuse = Math.sqrt(t.base() * t.base() + t.height() * t.height());
            return t.base() + t.height() + hypotenuse;
        }
        throw new IllegalArgumentException("Unknown shape");
    }
}
```

---

## Hands-On Exercise 4: Event Handling

Create `src/chapter05/EventHandling.java`:

```java
package chapter05;

import java.time.Instant;
import java.util.*;

// Event hierarchy
sealed interface Event permits ClickEvent, KeyEvent, ScrollEvent, CustomEvent {}

record ClickEvent(int x, int y, int button, Instant timestamp) implements Event {
    public static final int LEFT = 1;
    public static final int RIGHT = 3;
}

record KeyEvent(char key, boolean shift, boolean ctrl, Instant timestamp) implements Event {}

record ScrollEvent(int deltaX, int deltaY, Instant timestamp) implements Event {}

// Allow custom events to be extended
non-sealed class CustomEvent implements Event {
    private final String name;
    private final Map<String, Object> data;
    private final Instant timestamp;

    public CustomEvent(String name, Map<String, Object> data) {
        this.name = name;
        this.data = Map.copyOf(data);
        this.timestamp = Instant.now();
    }

    public String name() { return name; }
    public Map<String, Object> data() { return data; }
    public Instant timestamp() { return timestamp; }
}

public class EventHandling {
    public static void main(String[] args) {
        List<Event> events = List.of(
            new ClickEvent(100, 200, ClickEvent.LEFT, Instant.now()),
            new KeyEvent('A', false, true, Instant.now()),
            new ScrollEvent(0, -50, Instant.now()),
            new ClickEvent(150, 300, ClickEvent.RIGHT, Instant.now()),
            new CustomEvent("drag", Map.of("startX", 0, "startY", 0, "endX", 100, "endY", 100))
        );

        System.out.println("=== Event Processing ===\n");

        for (Event event : events) {
            handleEvent(event);
        }

        // Filtering specific event types
        System.out.println("\n=== Click Events Only ===");
        events.stream()
            .filter(e -> e instanceof ClickEvent)
            .forEach(e -> {
                if (e instanceof ClickEvent click) {
                    System.out.printf("Click at (%d, %d)%n", click.x(), click.y());
                }
            });

        // Counting by type
        System.out.println("\n=== Event Counts ===");
        Map<String, Long> counts = new HashMap<>();
        for (Event event : events) {
            String type = getEventType(event);
            counts.merge(type, 1L, Long::sum);
        }
        counts.forEach((type, count) -> System.out.println(type + ": " + count));
    }

    static void handleEvent(Event event) {
        if (event instanceof ClickEvent click) {
            String button = click.button() == ClickEvent.LEFT ? "left" : "right";
            System.out.printf("🖱️ %s click at (%d, %d)%n",
                button, click.x(), click.y());

        } else if (event instanceof KeyEvent key) {
            StringBuilder mods = new StringBuilder();
            if (key.ctrl()) mods.append("Ctrl+");
            if (key.shift()) mods.append("Shift+");
            System.out.printf("⌨️ Key pressed: %s%c%n",
                mods, key.key());

        } else if (event instanceof ScrollEvent scroll) {
            String direction = scroll.deltaY() < 0 ? "down" : "up";
            System.out.printf("📜 Scroll %s by %d%n",
                direction, Math.abs(scroll.deltaY()));

        } else if (event instanceof CustomEvent custom) {
            System.out.printf("🔧 Custom event '%s': %s%n",
                custom.name(), custom.data());
        }
    }

    static String getEventType(Event event) {
        if (event instanceof ClickEvent) return "ClickEvent";
        if (event instanceof KeyEvent) return "KeyEvent";
        if (event instanceof ScrollEvent) return "ScrollEvent";
        if (event instanceof CustomEvent) return "CustomEvent";
        return "Unknown";
    }
}
```

---

## Pattern Variable Shadowing

Be careful with variable names:

```java
class Example {
    String s = "field";

    void method(Object obj) {
        // Pattern variable 's' shadows the field 's'
        if (obj instanceof String s) {
            System.out.println(s);       // Pattern variable
            System.out.println(this.s);  // Field
        }
        System.out.println(s);  // Field (pattern variable out of scope)
    }
}
```

**Best Practice:** Use distinct names to avoid confusion:

```java
if (obj instanceof String str) {  // Clear: str is the pattern variable
    System.out.println(str);
}
```

---

## Common Patterns and Idioms

### Guard Clauses

```java
// Old way
void process(Object obj) {
    if (obj == null) return;
    if (!(obj instanceof String)) return;
    String s = (String) obj;
    // work with s
}

// New way
void process(Object obj) {
    if (!(obj instanceof String s)) return;
    // s is available here
    System.out.println(s.toUpperCase());
}
```

### Null-Safe Processing

```java
// Pattern matching with null check
void processNullSafe(Object obj) {
    if (obj instanceof String s && !s.isEmpty()) {
        System.out.println("Non-empty string: " + s);
    }
    // null is never an instance of anything, so the above handles null safely
}
```

### Chained Type Checks

```java
String describe(Object obj) {
    // Most specific first
    if (obj instanceof ArrayList<?> list) {
        return "ArrayList with " + list.size() + " elements";
    } else if (obj instanceof List<?> list) {
        return "List with " + list.size() + " elements";
    } else if (obj instanceof Collection<?> col) {
        return "Collection with " + col.size() + " elements";
    } else if (obj instanceof Iterable<?> iter) {
        return "Iterable";
    }
    return "Something else";
}
```

---

## Practice Exercises

### Exercise 5.1: Number Formatter
Write a method that takes an `Object` and formats it as a number string:
- If it's an `Integer`, format with commas (1,234,567)
- If it's a `Double`, format with 2 decimal places
- If it's a `String` that looks like a number, parse and format it
- Otherwise return "Not a number"

<details>
<summary>Click for solution</summary>

```java
import java.text.NumberFormat;

static String formatAsNumber(Object obj) {
    NumberFormat nf = NumberFormat.getInstance();

    if (obj instanceof Integer i) {
        return nf.format(i);
    } else if (obj instanceof Double d) {
        return String.format("%.2f", d);
    } else if (obj instanceof Long l) {
        return nf.format(l);
    } else if (obj instanceof String s) {
        try {
            double value = Double.parseDouble(s);
            if (s.contains(".")) {
                return String.format("%.2f", value);
            } else {
                return nf.format((long) value);
            }
        } catch (NumberFormatException e) {
            return "Not a number";
        }
    }
    return "Not a number";
}
```
</details>

### Exercise 5.2: Safe Cast Utility
Create a utility method `safeCast` that safely casts an object to a given type, returning an `Optional`:

```java
Optional<String> str = safeCast(obj, String.class);
```

<details>
<summary>Click for solution</summary>

```java
import java.util.Optional;

@SuppressWarnings("unchecked")
static <T> Optional<T> safeCast(Object obj, Class<T> type) {
    if (type.isInstance(obj)) {
        return Optional.of((T) obj);
    }
    return Optional.empty();
}

// Usage:
Optional<String> str = safeCast("hello", String.class);  // Optional[hello]
Optional<Integer> num = safeCast("hello", Integer.class); // Optional.empty
```
</details>

### Exercise 5.3: Expression Evaluator
Given this expression hierarchy, implement an `evaluate` method:

```java
sealed interface Expr permits Num, Add, Sub, Mul, Div {}
record Num(double value) implements Expr {}
record Add(Expr left, Expr right) implements Expr {}
record Sub(Expr left, Expr right) implements Expr {}
record Mul(Expr left, Expr right) implements Expr {}
record Div(Expr left, Expr right) implements Expr {}
```

<details>
<summary>Click for solution</summary>

```java
static double evaluate(Expr expr) {
    if (expr instanceof Num n) {
        return n.value();
    } else if (expr instanceof Add a) {
        return evaluate(a.left()) + evaluate(a.right());
    } else if (expr instanceof Sub s) {
        return evaluate(s.left()) - evaluate(s.right());
    } else if (expr instanceof Mul m) {
        return evaluate(m.left()) * evaluate(m.right());
    } else if (expr instanceof Div d) {
        double divisor = evaluate(d.right());
        if (divisor == 0) throw new ArithmeticException("Division by zero");
        return evaluate(d.left()) / divisor;
    }
    throw new IllegalArgumentException("Unknown expression type");
}
```
</details>

---

## Key Takeaways

1. **Pattern matching combines check and cast** - no redundant casts needed
2. **The pattern variable is scoped by flow analysis** - available where match is guaranteed
3. **Works with any type** - classes, interfaces, arrays, generics
4. **Null is never an instance** - pattern matching is null-safe
5. **Great for sealed types** - handle all cases cleanly
6. **Use in equals()** - cleaner implementations
7. **Leads to switch patterns** - next chapter!

---

## What's Next?

In [Chapter 6: Switch Expressions & Pattern Matching](06-switch-expressions.md), you'll see how pattern matching becomes even more powerful when combined with switch expressions.

---

[← Previous Chapter](04-sealed-classes.md) | [Back to Contents](../README.md) | [Next Chapter →](06-switch-expressions.md)
