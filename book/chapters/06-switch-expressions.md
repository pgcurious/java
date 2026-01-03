# Chapter 6: Switch Expressions & Pattern Matching

> **Java Version:** 14+ (expressions), 21+ (pattern matching) | **Difficulty:** Intermediate | **Time:** 45 minutes

## The Problem

Classic switch statements are verbose and error-prone:

```java
// Old switch - verbose and risky
String dayType;
switch (day) {
    case MONDAY:
    case TUESDAY:
    case WEDNESDAY:
    case THURSDAY:
    case FRIDAY:
        dayType = "Weekday";
        break;
    case SATURDAY:
    case SUNDAY:
        dayType = "Weekend";
        break;
    default:
        dayType = "Unknown";  // Forgot break? Silent fall-through!
}
```

Problems:
- Verbose `break` statements everywhere
- Easy to forget `break` (silent fall-through)
- Can't use switch as an expression
- Limited to primitives, enums, and strings

## The Solution: Switch Expressions

Java 14 introduced **switch expressions** with arrow syntax:

```java
String dayType = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Weekday";
    case SATURDAY, SUNDAY -> "Weekend";
};
```

And Java 21 added **pattern matching in switch**, letting you match types:

```java
String result = switch (obj) {
    case Integer i -> "Integer: " + i;
    case String s -> "String: " + s;
    case null -> "It's null";
    default -> "Something else";
};
```

---

## Hands-On Exercise 1: Switch Expressions Basics

Create `src/chapter06/SwitchExpressionBasics.java`:

```java
package chapter06;

public class SwitchExpressionBasics {
    public static void main(String[] args) {
        // Example 1: Classic switch statement
        System.out.println("=== Classic Switch Statement ===");
        int day = 3;
        String dayNameOld;
        switch (day) {
            case 1:
                dayNameOld = "Monday";
                break;
            case 2:
                dayNameOld = "Tuesday";
                break;
            case 3:
                dayNameOld = "Wednesday";
                break;
            case 4:
                dayNameOld = "Thursday";
                break;
            case 5:
                dayNameOld = "Friday";
                break;
            case 6:
                dayNameOld = "Saturday";
                break;
            case 7:
                dayNameOld = "Sunday";
                break;
            default:
                dayNameOld = "Invalid";
        }
        System.out.println("Day " + day + " = " + dayNameOld);

        // Example 2: Switch expression with arrow syntax
        System.out.println("\n=== Switch Expression (Arrow Syntax) ===");
        String dayNameNew = switch (day) {
            case 1 -> "Monday";
            case 2 -> "Tuesday";
            case 3 -> "Wednesday";
            case 4 -> "Thursday";
            case 5 -> "Friday";
            case 6 -> "Saturday";
            case 7 -> "Sunday";
            default -> "Invalid";
        };
        System.out.println("Day " + day + " = " + dayNameNew);

        // Example 3: Multiple labels per case
        System.out.println("\n=== Multiple Labels ===");
        for (int d = 1; d <= 7; d++) {
            String type = switch (d) {
                case 1, 2, 3, 4, 5 -> "Weekday";
                case 6, 7 -> "Weekend";
                default -> "Invalid";
            };
            System.out.println("Day " + d + " is a " + type);
        }

        // Example 4: Switch expression as a value
        System.out.println("\n=== Switch as Value ===");
        int month = 2;
        int year = 2024;
        int daysInMonth = switch (month) {
            case 1, 3, 5, 7, 8, 10, 12 -> 31;
            case 4, 6, 9, 11 -> 30;
            case 2 -> isLeapYear(year) ? 29 : 28;
            default -> throw new IllegalArgumentException("Invalid month: " + month);
        };
        System.out.println("Days in month " + month + ": " + daysInMonth);
    }

    static boolean isLeapYear(int year) {
        return (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
    }
}
```

**Run it:**
```bash
java src/chapter06/SwitchExpressionBasics.java
```

---

## Hands-On Exercise 2: yield Keyword for Blocks

When you need multiple statements in a case, use a block with `yield`:

Create `src/chapter06/SwitchYield.java`:

```java
package chapter06;

public class SwitchYield {
    public static void main(String[] args) {
        String[] grades = {"A", "B", "C", "D", "F", "X"};

        System.out.println("=== Grade Point Values ===\n");

        for (String grade : grades) {
            // Using yield for multi-line case blocks
            double gpa = switch (grade) {
                case "A" -> 4.0;
                case "B" -> 3.0;
                case "C" -> 2.0;
                case "D" -> 1.0;
                case "F" -> 0.0;
                default -> {
                    System.out.println("Warning: Unknown grade '" + grade + "'");
                    yield -1.0;  // yield returns value from block
                }
            };
            System.out.printf("Grade %s -> GPA: %.1f%n", grade, gpa);
        }

        // More complex example with yield
        System.out.println("\n=== Complex Processing ===\n");

        int score = 85;
        String feedback = switch (score / 10) {
            case 10, 9 -> {
                String base = "Excellent!";
                if (score == 100) {
                    yield base + " Perfect score!";
                }
                yield base + " Score: " + score;
            }
            case 8 -> {
                yield "Good job! Score: " + score;
            }
            case 7 -> {
                yield "Satisfactory. Score: " + score;
            }
            case 6 -> {
                yield "Passing. Score: " + score + ". Consider studying more.";
            }
            default -> {
                if (score < 0) {
                    yield "Invalid score!";
                }
                yield "Failing. Score: " + score + ". Please seek help.";
            }
        };
        System.out.println(feedback);
    }
}
```

**Key Point:** `yield` is like `return` but for switch expressions. Use it when a case block has multiple statements.

---

## Hands-On Exercise 3: Pattern Matching in Switch (Java 21)

Create `src/chapter06/PatternMatchingSwitch.java`:

```java
package chapter06;

import java.util.*;

public class PatternMatchingSwitch {
    public static void main(String[] args) {
        Object[] values = {
            42,
            "Hello",
            3.14,
            List.of(1, 2, 3),
            new int[]{1, 2, 3},
            null,
            true
        };

        System.out.println("=== Type Pattern Matching ===\n");

        for (Object value : values) {
            String result = describe(value);
            System.out.println(result);
        }

        // Pattern matching with records
        System.out.println("\n=== Record Pattern Matching ===\n");

        Shape[] shapes = {
            new Circle(5),
            new Rectangle(4, 6),
            new Triangle(3, 4)
        };

        for (Shape shape : shapes) {
            System.out.println(describeShape(shape));
        }
    }

    static String describe(Object obj) {
        return switch (obj) {
            case null -> "It's null!";
            case Integer i -> "Integer: " + i + " (even: " + (i % 2 == 0) + ")";
            case String s -> "String: \"" + s + "\" (length: " + s.length() + ")";
            case Double d -> "Double: " + String.format("%.2f", d);
            case List<?> list -> "List with " + list.size() + " elements: " + list;
            case int[] arr -> "int[] with " + arr.length + " elements";
            case Boolean b -> "Boolean: " + b;
            default -> "Unknown type: " + obj.getClass().getSimpleName();
        };
    }

    static String describeShape(Shape shape) {
        return switch (shape) {
            case Circle c -> String.format("Circle with radius %.1f, area = %.2f",
                c.radius(), Math.PI * c.radius() * c.radius());
            case Rectangle r -> String.format("Rectangle %.1fx%.1f, area = %.2f",
                r.width(), r.height(), r.width() * r.height());
            case Triangle t -> String.format("Triangle with base %.1f and height %.1f, area = %.2f",
                t.base(), t.height(), 0.5 * t.base() * t.height());
        };  // No default needed - sealed type!
    }
}

sealed interface Shape permits Circle, Rectangle, Triangle {}
record Circle(double radius) implements Shape {}
record Rectangle(double width, double height) implements Shape {}
record Triangle(double base, double height) implements Shape {}
```

**Run it:**
```bash
java src/chapter06/PatternMatchingSwitch.java
```

---

## Hands-On Exercise 4: Guarded Patterns (when clause)

Add conditions to patterns using `when`:

Create `src/chapter06/GuardedPatterns.java`:

```java
package chapter06;

public class GuardedPatterns {
    public static void main(String[] args) {
        Object[] values = {
            15,
            -5,
            0,
            100,
            "hello",
            "",
            "  ",
            null
        };

        System.out.println("=== Guarded Patterns ===\n");

        for (Object value : values) {
            String category = categorize(value);
            System.out.printf("%-10s -> %s%n", value == null ? "null" : "'" + value + "'", category);
        }

        // Number ranges
        System.out.println("\n=== Number Ranges ===\n");
        int[] scores = {-10, 0, 45, 65, 75, 85, 95, 100, 110};

        for (int score : scores) {
            String grade = getGrade(score);
            System.out.printf("Score %3d -> %s%n", score, grade);
        }
    }

    static String categorize(Object obj) {
        return switch (obj) {
            case null -> "null value";

            // Guarded patterns with 'when'
            case Integer i when i < 0 -> "negative integer";
            case Integer i when i == 0 -> "zero";
            case Integer i when i > 0 && i <= 50 -> "small positive integer";
            case Integer i when i > 50 -> "large positive integer";

            case String s when s.isEmpty() -> "empty string";
            case String s when s.isBlank() -> "blank string";
            case String s when s.length() < 5 -> "short string";
            case String s -> "long string";

            default -> "other type";
        };
    }

    static String getGrade(int score) {
        return switch (score) {
            case Integer s when s < 0 -> "Invalid (negative)";
            case Integer s when s > 100 -> "Invalid (too high)";
            case Integer s when s >= 90 -> "A";
            case Integer s when s >= 80 -> "B";
            case Integer s when s >= 70 -> "C";
            case Integer s when s >= 60 -> "D";
            default -> "F";
        };
    }
}
```

---

## Hands-On Exercise 5: Record Patterns (Java 21)

Deconstruct records directly in patterns:

Create `src/chapter06/RecordPatterns.java`:

```java
package chapter06;

public class RecordPatterns {
    public static void main(String[] args) {
        // Simple record patterns
        System.out.println("=== Simple Record Patterns ===\n");

        Point[] points = {
            new Point(0, 0),
            new Point(5, 0),
            new Point(0, 3),
            new Point(3, 4)
        };

        for (Point p : points) {
            String description = switch (p) {
                case Point(int x, int y) when x == 0 && y == 0 -> "Origin";
                case Point(int x, int y) when x == 0 -> "On Y-axis at y=" + y;
                case Point(int x, int y) when y == 0 -> "On X-axis at x=" + x;
                case Point(int x, int y) -> "Point at (" + x + ", " + y + ")";
            };
            System.out.println(p + " -> " + description);
        }

        // Nested record patterns
        System.out.println("\n=== Nested Record Patterns ===\n");

        Line[] lines = {
            new Line(new Point(0, 0), new Point(5, 5)),
            new Line(new Point(0, 0), new Point(10, 0)),
            new Line(new Point(0, 0), new Point(0, 10)),
            new Line(new Point(1, 1), new Point(4, 5))
        };

        for (Line line : lines) {
            String lineType = classifyLine(line);
            System.out.println(line + " -> " + lineType);
        }

        // Complex example with colors
        System.out.println("\n=== Colored Shapes ===\n");

        ColoredShape[] shapes = {
            new ColoredShape(new Circle(5), new Color(255, 0, 0)),
            new ColoredShape(new Circle(3), new Color(0, 255, 0)),
            new ColoredShape(new Rect(4, 6), new Color(0, 0, 255)),
            new ColoredShape(new Rect(5, 5), new Color(128, 128, 128))
        };

        for (ColoredShape cs : shapes) {
            describeColoredShape(cs);
        }
    }

    static String classifyLine(Line line) {
        return switch (line) {
            // Nested deconstruction!
            case Line(Point(int x1, int y1), Point(int x2, int y2))
                when x1 == x2 && y1 == y2 -> "A point (zero length)";

            case Line(Point(int x1, int y1), Point(int x2, int y2))
                when x1 == x2 -> "Vertical line";

            case Line(Point(int x1, int y1), Point(int x2, int y2))
                when y1 == y2 -> "Horizontal line";

            case Line(Point(int x1, int y1), Point(int x2, int y2))
                when (x2 - x1) == (y2 - y1) -> "Diagonal (45°)";

            case Line(Point p1, Point p2) -> "General line from " + p1 + " to " + p2;
        };
    }

    static void describeColoredShape(ColoredShape cs) {
        switch (cs) {
            case ColoredShape(Circle(double r), Color(int red, int g, int b))
                when red > 200 && g < 50 && b < 50 ->
                System.out.printf("Red circle with radius %.1f%n", r);

            case ColoredShape(Circle(double r), Color c) ->
                System.out.printf("Circle with radius %.1f in color %s%n", r, c);

            case ColoredShape(Rect(double w, double h), Color c) when w == h ->
                System.out.printf("Square %.1fx%.1f in color %s%n", w, h, c);

            case ColoredShape(Rect(double w, double h), Color c) ->
                System.out.printf("Rectangle %.1fx%.1f in color %s%n", w, h, c);
        }
    }
}

record Point(int x, int y) {}
record Line(Point start, Point end) {}
record Color(int red, int green, int blue) {}
record ColoredShape(ShapeType shape, Color color) {}

sealed interface ShapeType permits Circle, Rect {}
record Circle(double radius) implements ShapeType {}
record Rect(double width, double height) implements ShapeType {}
```

---

## Hands-On Exercise 6: Exhaustiveness

The compiler ensures switch expressions are exhaustive:

Create `src/chapter06/Exhaustiveness.java`:

```java
package chapter06;

public class Exhaustiveness {
    public static void main(String[] args) {
        // Enum exhaustiveness
        System.out.println("=== Enum Exhaustiveness ===\n");

        for (Status status : Status.values()) {
            // No default needed - all enum values covered!
            String message = switch (status) {
                case PENDING -> "Waiting...";
                case PROCESSING -> "Working on it...";
                case COMPLETED -> "Done!";
                case FAILED -> "Oops!";
                case CANCELLED -> "Nevermind.";
            };
            System.out.println(status + " -> " + message);
        }

        // Sealed type exhaustiveness
        System.out.println("\n=== Sealed Type Exhaustiveness ===\n");

        PaymentMethod[] methods = {
            new CreditCard("1234", "Visa"),
            new DebitCard("5678", "Chase"),
            new Cash(100.00)
        };

        for (PaymentMethod method : methods) {
            // No default needed - sealed type!
            String description = switch (method) {
                case CreditCard(String last4, String network) ->
                    network + " credit ending in " + last4;
                case DebitCard(String last4, String bank) ->
                    bank + " debit ending in " + last4;
                case Cash(double amount) ->
                    String.format("Cash: $%.2f", amount);
            };
            System.out.println(description);
        }

        // What happens if we add a new subtype?
        // The compiler will flag all switch expressions that
        // need to be updated!
    }
}

enum Status { PENDING, PROCESSING, COMPLETED, FAILED, CANCELLED }

sealed interface PaymentMethod permits CreditCard, DebitCard, Cash {}
record CreditCard(String last4Digits, String network) implements PaymentMethod {}
record DebitCard(String last4Digits, String bank) implements PaymentMethod {}
record Cash(double amount) implements PaymentMethod {}
```

---

## Hands-On Exercise 7: Null Handling

Java 21 allows explicit null handling in switch:

Create `src/chapter06/NullHandling.java`:

```java
package chapter06;

public class NullHandling {
    public static void main(String[] args) {
        String[] inputs = {"hello", "WORLD", "", null, "  ", "Java21"};

        System.out.println("=== Null Handling in Switch ===\n");

        for (String input : inputs) {
            String result = processString(input);
            System.out.printf("Input: %-10s -> %s%n",
                input == null ? "null" : "\"" + input + "\"", result);
        }

        // Combined null and type patterns
        System.out.println("\n=== Null with Type Patterns ===\n");

        Object[] objects = {42, "test", null, 3.14, true};

        for (Object obj : objects) {
            String type = getTypeName(obj);
            System.out.printf("%-10s -> %s%n",
                obj == null ? "null" : obj, type);
        }
    }

    static String processString(String s) {
        return switch (s) {
            case null -> "null value - handle specially";
            case "" -> "empty string";
            case String str when str.isBlank() -> "blank string";
            case String str when str.equals(str.toUpperCase()) -> "ALL CAPS: " + str;
            case String str -> "normal string: " + str;
        };
    }

    static String getTypeName(Object obj) {
        return switch (obj) {
            case null -> "null";
            case Integer i -> "Integer";
            case String s -> "String";
            case Double d -> "Double";
            case Boolean b -> "Boolean";
            default -> obj.getClass().getSimpleName();
        };
    }
}
```

**Note:** Before Java 21, passing `null` to a switch would throw `NullPointerException`. Now you can handle it explicitly.

---

## Switch Expression vs Statement

```java
// Switch EXPRESSION - produces a value
String result = switch (x) {
    case 1 -> "one";
    case 2 -> "two";
    default -> "other";
};

// Switch STATEMENT with arrow syntax - no value produced
switch (x) {
    case 1 -> System.out.println("one");
    case 2 -> System.out.println("two");
    default -> System.out.println("other");
}

// Classic switch STATEMENT still works
switch (x) {
    case 1:
        System.out.println("one");
        break;
    case 2:
        System.out.println("two");
        break;
    default:
        System.out.println("other");
}
```

---

## Common Patterns

### Return early with switch expression

```java
String process(Object obj) {
    return switch (obj) {
        case null -> "null";
        case String s when s.isEmpty() -> "empty";
        case String s -> s.toUpperCase();
        case Integer i -> String.valueOf(i * 2);
        default -> obj.toString();
    };
}
```

### Combining with Optional

```java
import java.util.Optional;

Optional<String> findType(Object obj) {
    return switch (obj) {
        case null -> Optional.empty();
        case String s -> Optional.of("string");
        case Integer i -> Optional.of("integer");
        default -> Optional.empty();
    };
}
```

### Error handling

```java
record Result<T>(T value, String error) {
    static <T> Result<T> success(T value) {
        return new Result<>(value, null);
    }

    static <T> Result<T> failure(String error) {
        return new Result<>(null, error);
    }
}

Result<Integer> parse(String input) {
    return switch (input) {
        case null -> Result.failure("Input is null");
        case String s when s.isBlank() -> Result.failure("Input is blank");
        case String s -> {
            try {
                yield Result.success(Integer.parseInt(s));
            } catch (NumberFormatException e) {
                yield Result.failure("Not a number: " + s);
            }
        }
    };
}
```

---

## Practice Exercises

### Exercise 6.1: HTTP Status Handler
Write a switch expression that takes an HTTP status code and returns a category:
- 100-199: "Informational"
- 200-299: "Success"
- 300-399: "Redirect"
- 400-499: "Client Error"
- 500-599: "Server Error"

<details>
<summary>Click for solution</summary>

```java
static String categorizeStatus(int code) {
    return switch (code) {
        case Integer c when c >= 100 && c < 200 -> "Informational";
        case Integer c when c >= 200 && c < 300 -> "Success";
        case Integer c when c >= 300 && c < 400 -> "Redirect";
        case Integer c when c >= 400 && c < 500 -> "Client Error";
        case Integer c when c >= 500 && c < 600 -> "Server Error";
        default -> "Unknown";
    };
}
```
</details>

### Exercise 6.2: Expression Calculator
Given this expression hierarchy, implement `evaluate` using switch patterns:

```java
sealed interface Expr {}
record Num(int value) implements Expr {}
record Add(Expr left, Expr right) implements Expr {}
record Mul(Expr left, Expr right) implements Expr {}
record Neg(Expr expr) implements Expr {}
```

<details>
<summary>Click for solution</summary>

```java
static int evaluate(Expr expr) {
    return switch (expr) {
        case Num(int value) -> value;
        case Add(var left, var right) -> evaluate(left) + evaluate(right);
        case Mul(var left, var right) -> evaluate(left) * evaluate(right);
        case Neg(var inner) -> -evaluate(inner);
    };
}
```
</details>

### Exercise 6.3: Command Parser
Create a sealed hierarchy for commands and use switch patterns to execute them:
- `PrintCommand(String message)` - prints the message
- `AddCommand(int a, int b)` - prints the sum
- `RepeatCommand(String text, int times)` - prints text n times
- `QuitCommand` - prints "Goodbye"

<details>
<summary>Click for solution</summary>

```java
sealed interface Command {}
record PrintCommand(String message) implements Command {}
record AddCommand(int a, int b) implements Command {}
record RepeatCommand(String text, int times) implements Command {}
record QuitCommand() implements Command {}

static void execute(Command cmd) {
    switch (cmd) {
        case PrintCommand(String msg) -> System.out.println(msg);
        case AddCommand(int a, int b) -> System.out.println(a + " + " + b + " = " + (a + b));
        case RepeatCommand(String text, int times) -> {
            for (int i = 0; i < times; i++) {
                System.out.println(text);
            }
        }
        case QuitCommand() -> System.out.println("Goodbye!");
    }
}
```
</details>

---

## Key Takeaways

1. **Switch expressions return values** - assign directly to variables
2. **Arrow syntax (`->`)** eliminates break statements and fall-through
3. **Multiple labels** can share one case: `case 1, 2, 3 ->`
4. **`yield`** returns values from block expressions
5. **Pattern matching** allows type checking and destructuring
6. **Guarded patterns (`when`)** add conditions to patterns
7. **Record patterns** deconstruct records in place
8. **Exhaustiveness** - compiler ensures all cases are covered
9. **Null handling** - explicit `case null` since Java 21
10. **Sealed types + switch** = powerful, safe pattern matching

---

## What's Next?

In [Chapter 7: Enhanced Collections & Streams](07-collections-streams.md), you'll discover the new collection factory methods and stream operations added since Java 9.

---

[← Previous Chapter](05-pattern-matching-instanceof.md) | [Back to Contents](../README.md) | [Next Chapter →](07-collections-streams.md)
