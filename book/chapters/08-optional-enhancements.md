# Chapter 8: Optional Enhancements

> **Java Version:** 9-11 | **Difficulty:** Beginner-Intermediate | **Time:** 30 minutes

## The Problem

Java 8 introduced `Optional` to handle the absence of values, but some operations were still clunky:

```java
// Java 8: No good way to execute different actions for present/empty
Optional<String> opt = findUser();
if (opt.isPresent()) {
    System.out.println("Found: " + opt.get());
} else {
    System.out.println("Not found");
}

// Java 8: No way to fall back to another Optional
Optional<String> primary = findPrimary();
Optional<String> secondary = findSecondary();
// How to try primary first, then secondary?
```

## The Solution: Enhanced Optional API

Java 9-11 added several methods to make Optional more powerful and expressive.

---

## Hands-On Exercise 1: ifPresentOrElse (Java 9)

Create `src/chapter08/IfPresentOrElse.java`:

```java
package chapter08;

import java.util.Optional;

public class IfPresentOrElse {
    public static void main(String[] args) {
        // Java 8 way
        System.out.println("=== Java 8 Way ===");
        Optional<String> user1 = Optional.of("Alice");
        Optional<String> user2 = Optional.empty();

        // Clunky: need if-else
        if (user1.isPresent()) {
            System.out.println("Found: " + user1.get());
        } else {
            System.out.println("Not found");
        }

        // Java 9 way: ifPresentOrElse
        System.out.println("\n=== Java 9 Way: ifPresentOrElse ===");

        user1.ifPresentOrElse(
            name -> System.out.println("Found: " + name),
            () -> System.out.println("Not found")
        );

        user2.ifPresentOrElse(
            name -> System.out.println("Found: " + name),
            () -> System.out.println("Not found")
        );

        // Practical example: Database lookup
        System.out.println("\n=== Practical: User Lookup ===");

        findUserById(1).ifPresentOrElse(
            user -> {
                System.out.println("Welcome, " + user.name() + "!");
                System.out.println("Email: " + user.email());
            },
            () -> System.out.println("User not found. Please register.")
        );

        findUserById(999).ifPresentOrElse(
            user -> System.out.println("Welcome, " + user.name()),
            () -> System.out.println("User 999 not found. Please register.")
        );
    }

    record User(int id, String name, String email) {}

    static Optional<User> findUserById(int id) {
        // Simulated database
        if (id == 1) {
            return Optional.of(new User(1, "Alice", "alice@example.com"));
        }
        return Optional.empty();
    }
}
```

**Run it:**
```bash
java src/chapter08/IfPresentOrElse.java
```

---

## Hands-On Exercise 2: or() - Fallback Optional (Java 9)

Create `src/chapter08/OrFallback.java`:

```java
package chapter08;

import java.util.Optional;

public class OrFallback {
    public static void main(String[] args) {
        // Java 8 problem: chaining Optionals is awkward
        System.out.println("=== Java 8 Problem ===");

        Optional<String> primary = Optional.empty();
        Optional<String> secondary = Optional.of("Secondary Value");

        // Ugly workaround in Java 8:
        String result = primary.isPresent() ? primary.get() : secondary.orElse("default");
        System.out.println("Result (Java 8 way): " + result);

        // Java 9: or() method
        System.out.println("\n=== Java 9: or() ===");

        Optional<String> final1 = primary.or(() -> secondary);
        System.out.println("primary.or(secondary): " + final1);

        // Chain multiple fallbacks
        System.out.println("\n=== Chaining Fallbacks ===");

        Optional<String> empty1 = Optional.empty();
        Optional<String> empty2 = Optional.empty();
        Optional<String> value = Optional.of("Found it!");

        Optional<String> chained = empty1
            .or(() -> empty2)
            .or(() -> value)
            .or(() -> Optional.of("Last resort"));

        System.out.println("Chained result: " + chained);

        // Practical: Configuration lookup
        System.out.println("\n=== Practical: Config Lookup ===");

        String dbUrl = getEnvVar("DATABASE_URL")
            .or(() -> getSystemProperty("db.url"))
            .or(() -> getConfigFile("database.url"))
            .orElse("jdbc:h2:mem:default");

        System.out.println("Database URL: " + dbUrl);
    }

    static Optional<String> getEnvVar(String name) {
        System.out.println("  Checking env var: " + name);
        return Optional.ofNullable(System.getenv(name));
    }

    static Optional<String> getSystemProperty(String name) {
        System.out.println("  Checking system property: " + name);
        return Optional.ofNullable(System.getProperty(name));
    }

    static Optional<String> getConfigFile(String key) {
        System.out.println("  Checking config file: " + key);
        // Simulated - return empty
        return Optional.empty();
    }
}
```

---

## Hands-On Exercise 3: stream() - Optional to Stream (Java 9)

Create `src/chapter08/OptionalStream.java`:

```java
package chapter08;

import java.util.*;
import java.util.stream.*;

public class OptionalStream {
    public static void main(String[] args) {
        // Java 8 problem: filtering Optionals in a stream
        System.out.println("=== Java 8 Problem ===");

        List<Optional<String>> optionals = List.of(
            Optional.of("apple"),
            Optional.empty(),
            Optional.of("banana"),
            Optional.empty(),
            Optional.of("cherry")
        );

        // Java 8: Ugly
        List<String> java8Way = optionals.stream()
            .filter(Optional::isPresent)
            .map(Optional::get)
            .toList();
        System.out.println("Java 8 way: " + java8Way);

        // Java 9: Optional.stream()
        System.out.println("\n=== Java 9: Optional.stream() ===");

        List<String> java9Way = optionals.stream()
            .flatMap(Optional::stream)  // Empty optional = empty stream
            .toList();
        System.out.println("Java 9 way: " + java9Way);

        // How it works
        System.out.println("\n=== How stream() works ===");
        Optional.of("present").stream().forEach(s -> System.out.println("Present: " + s));
        Optional.empty().stream().forEach(s -> System.out.println("Empty: " + s));  // Nothing printed

        // Practical: Getting all successful results
        System.out.println("\n=== Practical: Process Results ===");

        List<Integer> ids = List.of(1, 2, 3, 4, 5);

        List<User> users = ids.stream()
            .map(id -> findUserById(id))  // Returns Optional<User>
            .flatMap(Optional::stream)     // Keep only present values
            .toList();

        System.out.println("Found users:");
        users.forEach(u -> System.out.println("  " + u));

        // Combining with other stream operations
        System.out.println("\n=== Combined Operations ===");

        var activeUsernames = ids.stream()
            .map(OptionalStream::findUserById)
            .flatMap(Optional::stream)
            .filter(User::active)
            .map(User::name)
            .toList();

        System.out.println("Active usernames: " + activeUsernames);
    }

    record User(int id, String name, boolean active) {}

    static Optional<User> findUserById(int id) {
        return switch (id) {
            case 1 -> Optional.of(new User(1, "Alice", true));
            case 2 -> Optional.empty();  // User 2 doesn't exist
            case 3 -> Optional.of(new User(3, "Charlie", false));
            case 4 -> Optional.of(new User(4, "Diana", true));
            default -> Optional.empty();
        };
    }
}
```

---

## Hands-On Exercise 4: isEmpty() (Java 11)

Create `src/chapter08/IsEmpty.java`:

```java
package chapter08;

import java.util.Optional;

public class IsEmpty {
    public static void main(String[] args) {
        Optional<String> present = Optional.of("Hello");
        Optional<String> empty = Optional.empty();

        // Java 8: !isPresent() - double negative, awkward to read
        System.out.println("=== Java 8: !isPresent() ===");
        if (!empty.isPresent()) {
            System.out.println("It's empty (using !isPresent)");
        }

        // Java 11: isEmpty() - cleaner!
        System.out.println("\n=== Java 11: isEmpty() ===");
        if (empty.isEmpty()) {
            System.out.println("It's empty (using isEmpty)");
        }

        // Comparison
        System.out.println("\n=== Comparison ===");
        System.out.println("present.isPresent(): " + present.isPresent());
        System.out.println("present.isEmpty(): " + present.isEmpty());
        System.out.println("empty.isPresent(): " + empty.isPresent());
        System.out.println("empty.isEmpty(): " + empty.isEmpty());

        // Practical use in guards
        System.out.println("\n=== Practical: Guard Clauses ===");
        processRequest(Optional.of("data"));
        processRequest(Optional.empty());
    }

    static void processRequest(Optional<String> data) {
        // Clean guard clause with isEmpty
        if (data.isEmpty()) {
            System.out.println("No data provided - returning early");
            return;
        }

        System.out.println("Processing: " + data.get());
    }
}
```

---

## Hands-On Exercise 5: orElseThrow() without arguments (Java 10)

Create `src/chapter08/OrElseThrowSimple.java`:

```java
package chapter08;

import java.util.*;

public class OrElseThrowSimple {
    public static void main(String[] args) {
        // Java 8: orElseThrow requires exception supplier
        System.out.println("=== Java 8: orElseThrow(Supplier) ===");

        Optional<String> value = Optional.of("Hello");

        // Must provide exception supplier
        String result1 = value.orElseThrow(() -> new NoSuchElementException("Value not found"));
        System.out.println("Result: " + result1);

        // Java 10: orElseThrow() with no arguments
        System.out.println("\n=== Java 10: orElseThrow() ===");

        // Throws NoSuchElementException by default
        String result2 = value.orElseThrow();
        System.out.println("Result: " + result2);

        // Same as get() but with better semantics
        // get() implies the value is there
        // orElseThrow() makes it clear an exception may occur

        // Demonstrate the exception
        System.out.println("\n=== Exception Demo ===");
        Optional<String> empty = Optional.empty();
        try {
            String fail = empty.orElseThrow();
        } catch (NoSuchElementException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName());
        }

        // Practical: When you know value should exist
        System.out.println("\n=== Practical Use ===");
        var users = Map.of(1, "Alice", 2, "Bob");

        // When absence is truly exceptional
        int requiredUserId = 1;
        String user = Optional.ofNullable(users.get(requiredUserId))
            .orElseThrow();  // Clean - throws if user doesn't exist
        System.out.println("Required user: " + user);
    }
}
```

---

## Hands-On Exercise 6: Combining All New Methods

Create `src/chapter08/OptionalCombined.java`:

```java
package chapter08;

import java.util.*;
import java.util.stream.*;

public class OptionalCombined {
    public static void main(String[] args) {
        // Scenario: User authentication and session handling

        System.out.println("=== User Authentication Flow ===\n");

        // Test different scenarios
        authenticateUser("alice", "password123");
        System.out.println();
        authenticateUser("bob", "wrongpassword");
        System.out.println();
        authenticateUser("unknown", "password");

        // Scenario: Data pipeline with fallbacks
        System.out.println("\n=== Data Pipeline ===\n");

        List<String> userIds = List.of("U001", "U002", "U003", "U004");

        var userData = userIds.stream()
            .map(id -> fetchFromCache(id)
                .or(() -> fetchFromDatabase(id))
                .or(() -> fetchFromExternalApi(id)))
            .flatMap(Optional::stream)
            .toList();

        System.out.println("Retrieved user data: " + userData);

        // Scenario: Configuration with multiple sources
        System.out.println("\n=== Configuration Chain ===\n");
        loadConfiguration();
    }

    // Authentication flow
    static void authenticateUser(String username, String password) {
        System.out.println("Authenticating: " + username);

        findUser(username)
            .filter(user -> user.password().equals(password))
            .ifPresentOrElse(
                user -> {
                    createSession(user).ifPresentOrElse(
                        session -> System.out.println("  Login successful! Session: " + session),
                        () -> System.out.println("  Failed to create session")
                    );
                },
                () -> System.out.println("  Invalid credentials")
            );
    }

    record User(String username, String password, String role) {}

    static Optional<User> findUser(String username) {
        var users = Map.of(
            "alice", new User("alice", "password123", "admin"),
            "bob", new User("bob", "bobpass", "user")
        );
        return Optional.ofNullable(users.get(username));
    }

    static Optional<String> createSession(User user) {
        // Simulate session creation
        return Optional.of("SESSION-" + user.username().toUpperCase() + "-" +
            System.currentTimeMillis());
    }

    // Data pipeline
    static Optional<String> fetchFromCache(String id) {
        System.out.println("  Checking cache for " + id);
        // Simulate cache hit for U001
        return id.equals("U001") ? Optional.of("Cached: " + id) : Optional.empty();
    }

    static Optional<String> fetchFromDatabase(String id) {
        System.out.println("  Checking database for " + id);
        // Simulate DB hit for U002
        return id.equals("U002") ? Optional.of("FromDB: " + id) : Optional.empty();
    }

    static Optional<String> fetchFromExternalApi(String id) {
        System.out.println("  Checking external API for " + id);
        // Simulate API hit for U003
        return id.equals("U003") ? Optional.of("FromAPI: " + id) : Optional.empty();
    }

    // Configuration
    record Config(String dbUrl, int maxConnections, boolean debugMode) {}

    static void loadConfiguration() {
        String dbUrl = getConfig("DB_URL")
            .or(() -> getEnv("DATABASE_URL"))
            .or(() -> getDefault("db.url"))
            .orElseThrow(() -> new RuntimeException("No database URL configured!"));

        int maxConn = getConfig("MAX_CONNECTIONS")
            .or(() -> getEnv("MAX_CONN"))
            .map(Integer::parseInt)
            .orElse(10);

        boolean debug = getConfig("DEBUG")
            .map(Boolean::parseBoolean)
            .orElse(false);

        var config = new Config(dbUrl, maxConn, debug);
        System.out.println("Loaded configuration: " + config);
    }

    static Optional<String> getConfig(String key) {
        var config = Map.of("DB_URL", "jdbc:postgresql://localhost/mydb");
        return Optional.ofNullable(config.get(key));
    }

    static Optional<String> getEnv(String key) {
        return Optional.ofNullable(System.getenv(key));
    }

    static Optional<String> getDefault(String key) {
        var defaults = Map.of("db.url", "jdbc:h2:mem:default");
        return Optional.ofNullable(defaults.get(key));
    }
}
```

---

## Optional Best Practices

### Do:

```java
// Return Optional from methods that might not have a result
Optional<User> findById(int id) { ... }

// Chain operations fluently
findById(1)
    .filter(User::isActive)
    .map(User::getEmail)
    .ifPresent(this::sendNotification);

// Use or() for fallbacks
getFromCache(key).or(() -> getFromDb(key)).orElse(defaultValue);

// Use ifPresentOrElse for side effects
result.ifPresentOrElse(
    value -> log.info("Found: {}", value),
    () -> log.warn("Not found")
);
```

### Don't:

```java
// Don't use Optional for fields
class User {
    Optional<String> middleName;  // BAD - use nullable instead
}

// Don't use Optional for parameters
void process(Optional<String> data) { }  // BAD - use @Nullable

// Don't use isPresent() + get() when you can map/flatMap
if (opt.isPresent()) {
    return opt.get().transform();  // BAD
}
return opt.map(v -> v.transform()).orElse(default);  // GOOD

// Don't use Optional.of() with nullable values
Optional.of(nullableValue);  // Throws NPE if null!
Optional.ofNullable(nullableValue);  // GOOD
```

---

## Summary: Optional Methods by Version

| Version | Method | Purpose |
|---------|--------|---------|
| Java 8 | `of()`, `ofNullable()`, `empty()` | Creation |
| Java 8 | `isPresent()`, `get()` | Basic access |
| Java 8 | `ifPresent()` | Conditional action |
| Java 8 | `map()`, `flatMap()`, `filter()` | Transformation |
| Java 8 | `orElse()`, `orElseGet()`, `orElseThrow(Supplier)` | Default values |
| Java 9 | `ifPresentOrElse()` | Action for both cases |
| Java 9 | `or()` | Fallback Optional |
| Java 9 | `stream()` | Convert to Stream |
| Java 10 | `orElseThrow()` | Throw without supplier |
| Java 11 | `isEmpty()` | Check for empty |

---

## Practice Exercises

### Exercise 8.1: Safe Chain
Write a method that safely navigates a chain of optional values:
```java
// Get street name from: Company -> CEO -> Address -> Street
Optional<String> getCeoStreet(Company company)
```

<details>
<summary>Click for solution</summary>

```java
record Street(String name) {}
record Address(Street street) {}
record CEO(Address address) {}
record Company(CEO ceo) {}

static Optional<String> getCeoStreet(Company company) {
    return Optional.ofNullable(company)
        .map(Company::ceo)
        .map(CEO::address)
        .map(Address::street)
        .map(Street::name);
}
```
</details>

### Exercise 8.2: First Available
Write a method that returns the first available value from multiple sources:

```java
Optional<String> firstAvailable(
    Supplier<Optional<String>>... sources)
```

<details>
<summary>Click for solution</summary>

```java
@SafeVarargs
static Optional<String> firstAvailable(Supplier<Optional<String>>... sources) {
    Optional<String> result = Optional.empty();
    for (var source : sources) {
        result = result.or(source);
        if (result.isPresent()) break;  // Short-circuit
    }
    return result;
}

// Or with streams:
@SafeVarargs
static Optional<String> firstAvailableStream(Supplier<Optional<String>>... sources) {
    return Arrays.stream(sources)
        .map(Supplier::get)
        .flatMap(Optional::stream)
        .findFirst();
}
```
</details>

### Exercise 8.3: Validated Optional
Create a method that validates and transforms an optional value, logging the outcome:

<details>
<summary>Click for solution</summary>

```java
static Optional<Integer> parseAndValidate(Optional<String> input) {
    return input
        .filter(s -> !s.isBlank())
        .map(String::trim)
        .flatMap(s -> {
            try {
                return Optional.of(Integer.parseInt(s));
            } catch (NumberFormatException e) {
                return Optional.empty();
            }
        })
        .filter(n -> n > 0 && n < 1000);
}

// Usage with new methods
parseAndValidate(Optional.of("42")).ifPresentOrElse(
    n -> System.out.println("Valid: " + n),
    () -> System.out.println("Invalid input")
);
```
</details>

---

## Key Takeaways

1. **`ifPresentOrElse()`** - handle both present and absent cases cleanly
2. **`or()`** - chain fallback Optionals (not just values)
3. **`stream()`** - seamlessly integrate with Stream operations
4. **`isEmpty()`** - cleaner than `!isPresent()`
5. **`orElseThrow()`** - cleaner when exception is fine
6. **Don't use Optional for fields or parameters** - use nullable
7. **Use Optional for return types** when absence is valid

---

## What's Next?

In [Chapter 9: HTTP Client API](09-http-client.md), you'll learn about Java's modern HTTP client for making web requests.

---

[← Previous Chapter](07-collections-streams.md) | [Back to Contents](../README.md) | [Next Chapter →](09-http-client.md)
