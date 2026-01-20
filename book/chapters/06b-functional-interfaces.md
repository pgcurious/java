# Chapter 6B: Functional Interfaces Deep Dive

> **Java Version:** 8+ (with modern enhancements) | **Difficulty:** Beginner-Intermediate | **Time:** 40 minutes

## The Problem

You're working with streams, Optional, and modern Java APIs, but the various functional interfaces (`Predicate`, `Function`, `Consumer`, `Supplier`, etc.) feel like a grab bag of random types. You know they're important, but you're not confident about:
- When to use which interface
- How to compose them together
- What the specialized variants are for
- How to write clean, readable functional code

## The Solution: Master the java.util.function Package

Java's `java.util.function` package provides **43 functional interfaces** that cover virtually every use case. Understanding the core patterns makes the entire API click into place.

---

## Part 1: The Core Four Functional Interfaces

### The Essential Pattern

Every functional interface in Java follows one of four patterns:

| Interface | Input | Output | Method | Use Case |
|-----------|-------|--------|--------|----------|
| `Predicate<T>` | T | boolean | `test(T)` | Filtering, validation |
| `Function<T, R>` | T | R | `apply(T)` | Transformation |
| `Consumer<T>` | T | void | `accept(T)` | Side effects, actions |
| `Supplier<T>` | none | T | `get()` | Lazy generation |

### Hands-On Exercise 1: The Core Four in Action

Create `src/chapter06b/CoreFunctionalInterfaces.java`:

```java
package chapter06b;

import java.util.function.*;
import java.util.*;

public class CoreFunctionalInterfaces {
    public static void main(String[] args) {
        // ============================================
        // PREDICATE<T> - Test a condition (T -> boolean)
        // ============================================
        System.out.println("=== Predicate<T> ===");

        Predicate<String> isLongEnough = s -> s.length() >= 5;
        Predicate<String> startsWithA = s -> s.startsWith("A");

        var words = List.of("Apple", "Ant", "Banana", "Avocado", "Cat");

        System.out.println("Words: " + words);
        System.out.println("Long enough (>=5): " +
            words.stream().filter(isLongEnough).toList());
        System.out.println("Starts with A: " +
            words.stream().filter(startsWithA).toList());

        // ============================================
        // FUNCTION<T, R> - Transform T to R
        // ============================================
        System.out.println("\n=== Function<T, R> ===");

        Function<String, Integer> wordLength = String::length;
        Function<String, String> toUpperCase = String::toUpperCase;

        System.out.println("Lengths: " +
            words.stream().map(wordLength).toList());
        System.out.println("Uppercase: " +
            words.stream().map(toUpperCase).toList());

        // ============================================
        // CONSUMER<T> - Perform action on T (returns void)
        // ============================================
        System.out.println("\n=== Consumer<T> ===");

        Consumer<String> printWithBullet = s -> System.out.println("  • " + s);
        Consumer<String> logWord = s -> System.out.println("  [LOG] Processing: " + s);

        System.out.println("Words with bullets:");
        words.forEach(printWithBullet);

        // ============================================
        // SUPPLIER<T> - Generate/provide T (takes no input)
        // ============================================
        System.out.println("\n=== Supplier<T> ===");

        Supplier<Double> randomNumber = Math::random;
        Supplier<List<String>> emptyListSupplier = ArrayList::new;
        Supplier<String> greetingSupplier = () -> "Hello, World!";

        System.out.println("Random 1: " + randomNumber.get());
        System.out.println("Random 2: " + randomNumber.get());
        System.out.println("Empty list: " + emptyListSupplier.get());
        System.out.println("Greeting: " + greetingSupplier.get());

        // Lazy evaluation with Supplier
        System.out.println("\n=== Lazy Evaluation ===");
        Supplier<String> expensiveOperation = () -> {
            System.out.println("  Computing expensive value...");
            return "Expensive Result";
        };

        System.out.println("Supplier created but not called yet");
        System.out.println("Now calling supplier:");
        String result = expensiveOperation.get();
        System.out.println("Result: " + result);
    }
}
```

**Run it:**
```bash
java src/chapter06b/CoreFunctionalInterfaces.java
```

---

## Part 2: Predicate In Depth

### Hands-On Exercise 2: Predicate Composition

Predicates can be combined using `and()`, `or()`, and `negate()`:

Create `src/chapter06b/PredicateComposition.java`:

```java
package chapter06b;

import java.util.function.*;
import java.util.*;

public class PredicateComposition {
    public static void main(String[] args) {
        var products = List.of(
            new Product("Laptop", 999.99, "Electronics", true),
            new Product("Mouse", 29.99, "Electronics", true),
            new Product("Desk", 249.99, "Furniture", false),
            new Product("Monitor", 399.99, "Electronics", true),
            new Product("Chair", 199.99, "Furniture", true),
            new Product("Keyboard", 79.99, "Electronics", false),
            new Product("Lamp", 49.99, "Furniture", true)
        );

        // Define basic predicates
        Predicate<Product> isElectronics = p -> p.category().equals("Electronics");
        Predicate<Product> isExpensive = p -> p.price() > 100;
        Predicate<Product> isInStock = Product::inStock;

        System.out.println("=== Basic Predicates ===");
        System.out.println("Electronics: " +
            products.stream().filter(isElectronics).map(Product::name).toList());
        System.out.println("Expensive (>$100): " +
            products.stream().filter(isExpensive).map(Product::name).toList());
        System.out.println("In Stock: " +
            products.stream().filter(isInStock).map(Product::name).toList());

        // Combine with and()
        System.out.println("\n=== Predicate.and() ===");
        Predicate<Product> expensiveElectronics = isElectronics.and(isExpensive);
        System.out.println("Expensive Electronics: " +
            products.stream().filter(expensiveElectronics).map(Product::name).toList());

        // Combine with or()
        System.out.println("\n=== Predicate.or() ===");
        Predicate<Product> electronicsOrExpensive = isElectronics.or(isExpensive);
        System.out.println("Electronics OR Expensive: " +
            products.stream().filter(electronicsOrExpensive).map(Product::name).toList());

        // Negate with negate()
        System.out.println("\n=== Predicate.negate() ===");
        Predicate<Product> notElectronics = isElectronics.negate();
        System.out.println("NOT Electronics: " +
            products.stream().filter(notElectronics).map(Product::name).toList());

        // Complex composition
        System.out.println("\n=== Complex Composition ===");
        // In stock AND (electronics OR expensive)
        Predicate<Product> complexFilter = isInStock.and(isElectronics.or(isExpensive));
        System.out.println("In Stock AND (Electronics OR Expensive): " +
            products.stream().filter(complexFilter).map(Product::name).toList());

        // Predicate.not() - static method (Java 11)
        System.out.println("\n=== Predicate.not() (Java 11) ===");
        var outOfStock = products.stream()
            .filter(Predicate.not(Product::inStock))
            .map(Product::name)
            .toList();
        System.out.println("Out of Stock: " + outOfStock);

        // Predicate.isEqual() - for equality checking
        System.out.println("\n=== Predicate.isEqual() ===");
        Predicate<String> isApple = Predicate.isEqual("Apple");
        var fruits = List.of("Apple", "Banana", "Apple", "Cherry");
        long appleCount = fruits.stream().filter(isApple).count();
        System.out.println("Apple count: " + appleCount);
    }

    record Product(String name, double price, String category, boolean inStock) {}
}
```

---

## Part 3: Function In Depth

### Hands-On Exercise 3: Function Composition

Functions can be chained using `andThen()` and `compose()`:

Create `src/chapter06b/FunctionComposition.java`:

```java
package chapter06b;

import java.util.function.*;
import java.util.*;

public class FunctionComposition {
    public static void main(String[] args) {
        // Basic function
        Function<String, String> trim = String::trim;
        Function<String, String> toUpper = String::toUpperCase;
        Function<String, Integer> length = String::length;

        // andThen: f.andThen(g) means g(f(x))
        System.out.println("=== Function.andThen() ===");
        Function<String, String> trimThenUpper = trim.andThen(toUpper);
        System.out.println("'  hello  ' -> trimThenUpper -> '" +
            trimThenUpper.apply("  hello  ") + "'");

        // Chain multiple
        Function<String, Integer> trimUpperLength = trim.andThen(toUpper).andThen(length);
        System.out.println("'  hello  ' -> trim -> upper -> length = " +
            trimUpperLength.apply("  hello  "));

        // compose: f.compose(g) means f(g(x)) - opposite order!
        System.out.println("\n=== Function.compose() ===");
        Function<String, String> upperAfterTrim = toUpper.compose(trim);
        System.out.println("'  hello  ' -> upperAfterTrim -> '" +
            upperAfterTrim.apply("  hello  ") + "'");

        // Identity function
        System.out.println("\n=== Function.identity() ===");
        Function<String, String> identity = Function.identity();
        System.out.println("identity.apply(\"test\") = " + identity.apply("test"));

        // Useful for grouping: map keys to themselves
        var words = List.of("apple", "banana", "apple", "cherry");
        var countByWord = words.stream()
            .collect(java.util.stream.Collectors.groupingBy(
                Function.identity(),
                java.util.stream.Collectors.counting()
            ));
        System.out.println("Word counts: " + countByWord);

        // Real-world example: Data transformation pipeline
        System.out.println("\n=== Data Transformation Pipeline ===");

        record RawUser(String name, String email) {}
        record CleanUser(String name, String email, String domain) {}

        Function<RawUser, RawUser> trimFields = u ->
            new RawUser(u.name().trim(), u.email().trim());

        Function<RawUser, RawUser> lowercaseEmail = u ->
            new RawUser(u.name(), u.email().toLowerCase());

        Function<RawUser, CleanUser> extractDomain = u -> {
            String domain = u.email().substring(u.email().indexOf('@') + 1);
            return new CleanUser(u.name(), u.email(), domain);
        };

        Function<RawUser, CleanUser> cleanUserPipeline =
            trimFields.andThen(lowercaseEmail).andThen(extractDomain);

        var rawUser = new RawUser("  John Doe  ", "  JOHN@EXAMPLE.COM  ");
        var cleanUser = cleanUserPipeline.apply(rawUser);

        System.out.println("Raw: " + rawUser);
        System.out.println("Clean: " + cleanUser);
    }
}
```

---

## Part 4: Binary Variants (Two Arguments)

### Hands-On Exercise 4: BiPredicate, BiFunction, BiConsumer

Create `src/chapter06b/BinaryFunctionalInterfaces.java`:

```java
package chapter06b;

import java.util.function.*;
import java.util.*;

public class BinaryFunctionalInterfaces {
    public static void main(String[] args) {
        // ============================================
        // BiPredicate<T, U> - Test condition with two inputs
        // ============================================
        System.out.println("=== BiPredicate<T, U> ===");

        BiPredicate<String, Integer> hasMinLength = (s, min) -> s.length() >= min;
        BiPredicate<Integer, Integer> isInRange = (val, max) -> val >= 0 && val <= max;

        System.out.println("'Hello' has min length 3: " + hasMinLength.test("Hello", 3));
        System.out.println("'Hi' has min length 3: " + hasMinLength.test("Hi", 3));
        System.out.println("5 is in range [0, 10]: " + isInRange.test(5, 10));
        System.out.println("15 is in range [0, 10]: " + isInRange.test(15, 10));

        // BiPredicate composition
        BiPredicate<String, Integer> notHasMinLength = hasMinLength.negate();
        System.out.println("'Hi' does NOT have min length 3: " +
            notHasMinLength.test("Hi", 3));

        // ============================================
        // BiFunction<T, U, R> - Transform two inputs to one output
        // ============================================
        System.out.println("\n=== BiFunction<T, U, R> ===");

        BiFunction<String, String, String> concat = (a, b) -> a + " " + b;
        BiFunction<Integer, Integer, Integer> max = Math::max;
        BiFunction<String, Integer, String> repeat = String::repeat;

        System.out.println("concat('Hello', 'World'): " + concat.apply("Hello", "World"));
        System.out.println("max(5, 10): " + max.apply(5, 10));
        System.out.println("repeat('Hi', 3): " + repeat.apply("Hi", 3));

        // BiFunction with andThen (chains a Function, not BiFunction)
        BiFunction<String, String, Integer> concatLength =
            concat.andThen(String::length);
        System.out.println("concat then length('Hello', 'World'): " +
            concatLength.apply("Hello", "World"));

        // Real-world: Map.compute uses BiFunction
        var scores = new HashMap<String, Integer>();
        scores.put("Alice", 100);
        scores.put("Bob", 85);

        BiFunction<String, Integer, Integer> addBonus = (name, score) ->
            score != null ? score + 10 : 10;

        scores.compute("Alice", addBonus);
        scores.compute("Charlie", addBonus);  // New entry
        System.out.println("Scores after bonus: " + scores);

        // ============================================
        // BiConsumer<T, U> - Action with two inputs (no return)
        // ============================================
        System.out.println("\n=== BiConsumer<T, U> ===");

        BiConsumer<String, Integer> printRepeated = (s, n) ->
            System.out.println("  " + s.repeat(n));

        BiConsumer<String, Integer> logEntry = (key, value) ->
            System.out.println("  Key: " + key + ", Value: " + value);

        System.out.println("Repeated strings:");
        printRepeated.accept("*", 5);
        printRepeated.accept("->", 3);

        // BiConsumer with Map.forEach
        System.out.println("\nMap entries:");
        scores.forEach(logEntry);

        // BiConsumer.andThen - chain two BiConsumers
        BiConsumer<String, Integer> logAndRepeat = logEntry.andThen(printRepeated);
        System.out.println("\nLog then repeat:");
        logAndRepeat.accept("=-", 4);
    }
}
```

---

## Part 5: Operators (Specialized Functions)

### Hands-On Exercise 5: UnaryOperator and BinaryOperator

Operators are specialized functions where input and output types are the same:

Create `src/chapter06b/Operators.java`:

```java
package chapter06b;

import java.util.function.*;
import java.util.*;

public class Operators {
    public static void main(String[] args) {
        // ============================================
        // UnaryOperator<T> - Function<T, T>
        // ============================================
        System.out.println("=== UnaryOperator<T> ===");

        UnaryOperator<Integer> doubleIt = n -> n * 2;
        UnaryOperator<Integer> square = n -> n * n;
        UnaryOperator<String> addExclamation = s -> s + "!";

        System.out.println("double(5) = " + doubleIt.apply(5));
        System.out.println("square(5) = " + square.apply(5));
        System.out.println("addExclamation('Hello') = " + addExclamation.apply("Hello"));

        // UnaryOperator composition
        UnaryOperator<Integer> doubleThenSquare = doubleIt.andThen(square)::apply;
        // Note: andThen returns Function<T,T>, cast back to UnaryOperator
        System.out.println("double then square(5) = " + doubleThenSquare.apply(5));

        // UnaryOperator.identity()
        UnaryOperator<String> noChange = UnaryOperator.identity();
        System.out.println("identity('test') = " + noChange.apply("test"));

        // Used with List.replaceAll
        System.out.println("\n=== replaceAll with UnaryOperator ===");
        var names = new ArrayList<>(List.of("alice", "bob", "charlie"));
        System.out.println("Before: " + names);
        names.replaceAll(String::toUpperCase);  // UnaryOperator<String>
        System.out.println("After toUpperCase: " + names);

        // Used with Stream.iterate
        System.out.println("\n=== Stream.iterate with UnaryOperator ===");
        var powersOfTwo = java.util.stream.Stream
            .iterate(1, n -> n < 100, n -> n * 2)  // UnaryOperator for next
            .toList();
        System.out.println("Powers of 2: " + powersOfTwo);

        // ============================================
        // BinaryOperator<T> - BiFunction<T, T, T>
        // ============================================
        System.out.println("\n=== BinaryOperator<T> ===");

        BinaryOperator<Integer> add = (a, b) -> a + b;
        BinaryOperator<Integer> multiply = (a, b) -> a * b;
        BinaryOperator<String> concatWithSpace = (a, b) -> a + " " + b;

        System.out.println("add(3, 4) = " + add.apply(3, 4));
        System.out.println("multiply(3, 4) = " + multiply.apply(3, 4));
        System.out.println("concatWithSpace('Hello', 'World') = " +
            concatWithSpace.apply("Hello", "World"));

        // BinaryOperator.minBy and maxBy
        System.out.println("\n=== BinaryOperator.minBy/maxBy ===");
        BinaryOperator<String> shorterString =
            BinaryOperator.minBy(Comparator.comparingInt(String::length));
        BinaryOperator<String> longerString =
            BinaryOperator.maxBy(Comparator.comparingInt(String::length));

        System.out.println("Shorter of 'cat' and 'elephant': " +
            shorterString.apply("cat", "elephant"));
        System.out.println("Longer of 'cat' and 'elephant': " +
            longerString.apply("cat", "elephant"));

        // Used with Stream.reduce
        System.out.println("\n=== Stream.reduce with BinaryOperator ===");
        var numbers = List.of(1, 2, 3, 4, 5);
        int sum = numbers.stream().reduce(0, add);
        int product = numbers.stream().reduce(1, multiply);
        System.out.println("Sum of " + numbers + " = " + sum);
        System.out.println("Product of " + numbers + " = " + product);

        // reduce with Optional (no identity)
        var words = List.of("Hello", "functional", "programming");
        String concatenated = words.stream()
            .reduce(concatWithSpace)
            .orElse("");
        System.out.println("Concatenated: " + concatenated);

        // Find longest word
        String longest = words.stream()
            .reduce(longerString)
            .orElse("");
        System.out.println("Longest word: " + longest);
    }
}
```

---

## Part 6: Primitive Specializations

### Hands-On Exercise 6: Avoiding Boxing Overhead

Create `src/chapter06b/PrimitiveSpecializations.java`:

```java
package chapter06b;

import java.util.function.*;
import java.util.*;
import java.util.stream.*;

public class PrimitiveSpecializations {
    public static void main(String[] args) {
        // Problem: Boxing/unboxing overhead with generics
        // Solution: Primitive specializations

        System.out.println("=== Primitive Predicates ===");

        // IntPredicate, LongPredicate, DoublePredicate
        IntPredicate isEven = n -> n % 2 == 0;
        IntPredicate isPositive = n -> n > 0;
        LongPredicate isLargeNumber = n -> n > 1_000_000_000L;
        DoublePredicate isFinite = Double::isFinite;

        System.out.println("isEven(4): " + isEven.test(4));
        System.out.println("isEven(5): " + isEven.test(5));
        System.out.println("isLargeNumber(999): " + isLargeNumber.test(999));
        System.out.println("isFinite(1.5): " + isFinite.test(1.5));
        System.out.println("isFinite(INFINITY): " + isFinite.test(Double.POSITIVE_INFINITY));

        // Primitive predicate composition
        IntPredicate isPositiveEven = isEven.and(isPositive);
        System.out.println("isPositiveEven(-4): " + isPositiveEven.test(-4));
        System.out.println("isPositiveEven(4): " + isPositiveEven.test(4));

        System.out.println("\n=== Primitive Functions ===");

        // IntFunction<R>, LongFunction<R>, DoubleFunction<R>
        IntFunction<String> intToString = n -> "Number: " + n;
        System.out.println("intToString(42): " + intToString.apply(42));

        // ToIntFunction<T>, ToLongFunction<T>, ToDoubleFunction<T>
        ToIntFunction<String> stringLength = String::length;
        System.out.println("stringLength('Hello'): " + stringLength.applyAsInt("Hello"));

        // IntToLongFunction, IntToDoubleFunction, etc.
        IntToDoubleFunction intToDouble = n -> n * 1.5;
        System.out.println("intToDouble(10): " + intToDouble.applyAsDouble(10));

        // IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator
        IntUnaryOperator triple = n -> n * 3;
        IntUnaryOperator addTen = n -> n + 10;
        System.out.println("triple(5): " + triple.applyAsInt(5));

        // Composition
        IntUnaryOperator tripleThenAddTen = triple.andThen(addTen);
        System.out.println("tripleThenAddTen(5): " + tripleThenAddTen.applyAsInt(5));

        // IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator
        IntBinaryOperator multiply = (a, b) -> a * b;
        DoubleBinaryOperator average = (a, b) -> (a + b) / 2.0;
        System.out.println("multiply(6, 7): " + multiply.applyAsInt(6, 7));
        System.out.println("average(3.0, 5.0): " + average.applyAsDouble(3.0, 5.0));

        System.out.println("\n=== Primitive Consumers and Suppliers ===");

        // IntConsumer, LongConsumer, DoubleConsumer
        IntConsumer printSquare = n -> System.out.println("  " + n + "^2 = " + (n * n));
        System.out.println("Squares:");
        IntStream.rangeClosed(1, 5).forEach(printSquare);

        // IntSupplier, LongSupplier, DoubleSupplier
        var random = new Random(42);
        IntSupplier diceRoll = () -> random.nextInt(6) + 1;
        System.out.println("\nDice rolls:");
        for (int i = 0; i < 5; i++) {
            System.out.println("  Roll " + (i + 1) + ": " + diceRoll.getAsInt());
        }

        DoubleSupplier randomDouble = Math::random;
        System.out.println("\nRandom double: " + randomDouble.getAsDouble());

        System.out.println("\n=== Performance Benefit ===");

        // Primitive streams avoid boxing
        var numbers = IntStream.rangeClosed(1, 1_000_000);
        long sum = numbers.filter(isEven).sum();  // No boxing!
        System.out.println("Sum of even numbers 1-1,000,000: " + sum);

        // ObjIntConsumer, ObjLongConsumer, ObjDoubleConsumer
        ObjIntConsumer<String> printNTimes = (s, n) -> {
            for (int i = 0; i < n; i++) System.out.print(s);
            System.out.println();
        };
        System.out.println("\nObjIntConsumer example:");
        printNTimes.accept("*", 10);
    }
}
```

---

## Part 7: Practical Applications

### Hands-On Exercise 7: Building Reusable Validators

Create `src/chapter06b/ValidatorPattern.java`:

```java
package chapter06b;

import java.util.function.*;
import java.util.*;
import java.util.regex.*;

public class ValidatorPattern {
    public static void main(String[] args) {
        // Build composable validators using Predicate

        // String validators
        Predicate<String> notNull = Objects::nonNull;
        Predicate<String> notEmpty = s -> !s.isEmpty();
        Predicate<String> notBlank = s -> !s.isBlank();
        Predicate<String> minLength(int min) {
            return s -> s != null && s.length() >= min;
        }
        Predicate<String> maxLength(int max) {
            return s -> s != null && s.length() <= max;
        }
        Predicate<String> matchesPattern(String regex) {
            Pattern pattern = Pattern.compile(regex);
            return s -> s != null && pattern.matcher(s).matches();
        }

        // Compose validators
        Predicate<String> validUsername = notNull
            .and(notBlank)
            .and(minLength(3))
            .and(maxLength(20))
            .and(matchesPattern("^[a-zA-Z0-9_]+$"));

        Predicate<String> validEmail = notNull
            .and(notBlank)
            .and(matchesPattern("^[\\w.-]+@[\\w.-]+\\.[a-zA-Z]{2,}$"));

        // Test usernames
        System.out.println("=== Username Validation ===");
        var usernames = List.of("alice", "bob123", "a", "valid_user",
            "way_too_long_username_here", "invalid user", "test@email");

        for (String username : usernames) {
            String status = validUsername.test(username) ? "valid" : "INVALID";
            System.out.printf("  %-30s : %s%n", "'" + username + "'", status);
        }

        // Test emails
        System.out.println("\n=== Email Validation ===");
        var emails = List.of("test@example.com", "invalid", "user@domain",
            "valid.name@company.org", "@bad.com");

        for (String email : emails) {
            String status = validEmail.test(email) ? "valid" : "INVALID";
            System.out.printf("  %-30s : %s%n", "'" + email + "'", status);
        }

        // Numeric validators
        System.out.println("\n=== Numeric Validation ===");

        Predicate<Integer> positive = n -> n > 0;
        Predicate<Integer> lessThan(int max) { return n -> n < max; }
        Predicate<Integer> between(int min, int max) {
            return n -> n >= min && n <= max;
        }

        Predicate<Integer> validAge = positive.and(lessThan(150));
        Predicate<Integer> validPercentage = between(0, 100);

        var ages = List.of(-5, 0, 25, 100, 200);
        System.out.println("Age validation:");
        for (int age : ages) {
            System.out.printf("  %-5d : %s%n", age,
                validAge.test(age) ? "valid" : "INVALID");
        }

        // Using with Optional
        System.out.println("\n=== With Optional ===");
        Optional<String> maybeUsername = Optional.of("valid_user");

        boolean isValid = maybeUsername.filter(validUsername).isPresent();
        System.out.println("Optional username valid: " + isValid);

        // Using with Stream filtering
        System.out.println("\n=== Filtering Collection ===");
        var validUsers = usernames.stream()
            .filter(validUsername)
            .toList();
        System.out.println("Valid usernames: " + validUsers);
    }

    // Helper methods (defined as static for use in predicates)
    static Predicate<String> minLength(int min) {
        return s -> s != null && s.length() >= min;
    }

    static Predicate<String> maxLength(int max) {
        return s -> s != null && s.length() <= max;
    }

    static Predicate<String> matchesPattern(String regex) {
        Pattern pattern = Pattern.compile(regex);
        return s -> s != null && pattern.matcher(s).matches();
    }

    static Predicate<Integer> lessThan(int max) {
        return n -> n < max;
    }

    static Predicate<Integer> between(int min, int max) {
        return n -> n >= min && n <= max;
    }
}
```

---

### Hands-On Exercise 8: Strategy Pattern with Functional Interfaces

Create `src/chapter06b/StrategyPattern.java`:

```java
package chapter06b;

import java.util.function.*;
import java.util.*;

public class StrategyPattern {
    public static void main(String[] args) {
        // Pricing strategies using Function
        System.out.println("=== Pricing Strategies ===");

        record Item(String name, double basePrice) {}

        // Define pricing strategies as Functions
        Function<Item, Double> standardPricing = Item::basePrice;

        Function<Item, Double> premiumPricing = item ->
            item.basePrice() * 1.2;  // 20% markup

        Function<Item, Double> discountPricing = item ->
            item.basePrice() * 0.8;  // 20% off

        Function<Item, Double> tieredPricing = item -> {
            if (item.basePrice() > 100) return item.basePrice() * 0.85;  // 15% off
            if (item.basePrice() > 50) return item.basePrice() * 0.90;   // 10% off
            return item.basePrice();
        };

        var items = List.of(
            new Item("Book", 25.00),
            new Item("Headphones", 75.00),
            new Item("Laptop", 999.00)
        );

        // Apply different strategies
        printPrices("Standard", items, standardPricing);
        printPrices("Premium", items, premiumPricing);
        printPrices("Discount", items, discountPricing);
        printPrices("Tiered", items, tieredPricing);

        // Sorting strategies using Comparator (which is a functional interface!)
        System.out.println("\n=== Sorting Strategies ===");

        record Person(String name, int age, String city) {}

        var people = new ArrayList<>(List.of(
            new Person("Alice", 30, "New York"),
            new Person("Bob", 25, "Chicago"),
            new Person("Charlie", 35, "Boston"),
            new Person("Diana", 28, "New York")
        ));

        // Different sorting strategies
        Comparator<Person> byName = Comparator.comparing(Person::name);
        Comparator<Person> byAge = Comparator.comparingInt(Person::age);
        Comparator<Person> byCityThenName = Comparator
            .comparing(Person::city)
            .thenComparing(Person::name);

        System.out.println("By name:");
        people.stream().sorted(byName).forEach(p ->
            System.out.println("  " + p));

        System.out.println("\nBy age:");
        people.stream().sorted(byAge).forEach(p ->
            System.out.println("  " + p));

        System.out.println("\nBy city, then name:");
        people.stream().sorted(byCityThenName).forEach(p ->
            System.out.println("  " + p));

        // Processing strategies using Consumer
        System.out.println("\n=== Processing Strategies ===");

        Consumer<Person> logProcessor = p ->
            System.out.println("  [LOG] Processing: " + p.name());

        Consumer<Person> welcomeProcessor = p ->
            System.out.println("  Welcome, " + p.name() + " from " + p.city() + "!");

        Consumer<Person> fullProcessor = logProcessor.andThen(welcomeProcessor);

        System.out.println("Processing with full pipeline:");
        people.forEach(fullProcessor);
    }

    static void printPrices(String strategy, List<Item> items,
                           Function<Item, Double> pricer) {
        System.out.println("\n" + strategy + " pricing:");
        for (var item : items) {
            System.out.printf("  %-12s: $%.2f -> $%.2f%n",
                item.name(), item.basePrice(), pricer.apply(item));
        }
    }

    record Item(String name, double basePrice) {}
}
```

---

## Summary: Functional Interfaces Cheat Sheet

### Core Interfaces

| Interface | Method | Signature | Common Uses |
|-----------|--------|-----------|-------------|
| `Predicate<T>` | `test` | `T -> boolean` | filter, validation |
| `Function<T,R>` | `apply` | `T -> R` | map, transform |
| `Consumer<T>` | `accept` | `T -> void` | forEach, side effects |
| `Supplier<T>` | `get` | `() -> T` | lazy values, factories |

### Binary Interfaces

| Interface | Method | Signature | Common Uses |
|-----------|--------|-----------|-------------|
| `BiPredicate<T,U>` | `test` | `(T,U) -> boolean` | two-arg testing |
| `BiFunction<T,U,R>` | `apply` | `(T,U) -> R` | two-arg transform |
| `BiConsumer<T,U>` | `accept` | `(T,U) -> void` | Map.forEach |

### Operators

| Interface | Method | Signature | Common Uses |
|-----------|--------|-----------|-------------|
| `UnaryOperator<T>` | `apply` | `T -> T` | replaceAll, iterate |
| `BinaryOperator<T>` | `apply` | `(T,T) -> T` | reduce, merge |

### Composition Methods

| Method | On | Description |
|--------|-----|-------------|
| `and()` | Predicate | AND composition |
| `or()` | Predicate | OR composition |
| `negate()` | Predicate | NOT |
| `andThen()` | Function, Consumer | Chain (f then g) |
| `compose()` | Function | Reverse chain (g then f) |

---

## Practice Exercises

### Exercise 6B.1: Chain Transformations
Create a data processing pipeline using Function composition that:
1. Trims whitespace
2. Converts to lowercase
3. Replaces spaces with underscores
4. Adds a prefix "user_"

<details>
<summary>Click for solution</summary>

```java
Function<String, String> pipeline = ((Function<String, String>) String::trim)
    .andThen(String::toLowerCase)
    .andThen(s -> s.replace(" ", "_"))
    .andThen(s -> "user_" + s);

String result = pipeline.apply("  John Doe  ");  // "user_john_doe"
```
</details>

### Exercise 6B.2: Complex Predicate
Create a password validator that checks:
- At least 8 characters
- Contains at least one digit
- Contains at least one uppercase letter
- No spaces

<details>
<summary>Click for solution</summary>

```java
Predicate<String> minLength = s -> s.length() >= 8;
Predicate<String> hasDigit = s -> s.chars().anyMatch(Character::isDigit);
Predicate<String> hasUpper = s -> s.chars().anyMatch(Character::isUpperCase);
Predicate<String> noSpaces = s -> !s.contains(" ");

Predicate<String> validPassword = minLength
    .and(hasDigit)
    .and(hasUpper)
    .and(noSpaces);
```
</details>

### Exercise 6B.3: Custom Collector
Use `BinaryOperator` with `reduce` to find the product with the highest rating:

<details>
<summary>Click for solution</summary>

```java
record Product(String name, double rating) {}

BinaryOperator<Product> higherRated = (p1, p2) ->
    p1.rating() >= p2.rating() ? p1 : p2;

Optional<Product> best = products.stream().reduce(higherRated);
```
</details>

---

## Key Takeaways

1. **Four core patterns**: Predicate (test), Function (transform), Consumer (action), Supplier (generate)
2. **Composition is powerful**: Use `and()`, `or()`, `negate()`, `andThen()`, `compose()` to build complex logic from simple parts
3. **Binary variants**: BiPredicate, BiFunction, BiConsumer handle two arguments
4. **Operators**: UnaryOperator and BinaryOperator are specialized Functions with same input/output types
5. **Primitive specializations**: IntPredicate, DoubleFunction, etc. avoid boxing overhead
6. **Real-world patterns**: Validators, strategies, pipelines all benefit from functional interfaces
7. **Method references**: Use `String::length`, `Objects::nonNull`, etc. for cleaner code

---

## What's Next?

In [Chapter 7: Enhanced Collections & Streams](07-collections-streams.md), you'll see these functional interfaces in action with the Stream API and modern collection methods.

---

[← Previous Chapter](06-switch-expressions.md) | [Back to Contents](../README.md) | [Next Chapter →](07-collections-streams.md)
