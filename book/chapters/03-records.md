# Chapter 3: Records

> **Java Version:** 16+ | **Difficulty:** Beginner-Intermediate | **Time:** 45 minutes

## The Problem

You need a simple class to hold data - a user, a point, a configuration. In Java 8, you write:

```java
public class User {
    private final String name;
    private final String email;
    private final int age;

    public User(String name, String email, int age) {
        this.name = name;
        this.email = email;
        this.age = age;
    }

    public String getName() { return name; }
    public String getEmail() { return email; }
    public int getAge() { return age; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return age == user.age &&
               Objects.equals(name, user.name) &&
               Objects.equals(email, user.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, email, age);
    }

    @Override
    public String toString() {
        return "User{name='" + name + "', email='" + email + "', age=" + age + "}";
    }
}
```

That's **40+ lines** for a simple data holder!

## The Solution: Records

```java
public record User(String name, String email, int age) {}
```

**One line.** Same functionality. Records are immutable data carriers with automatically generated:
- Constructor
- Accessor methods (not getXxx, just the field name)
- `equals()`, `hashCode()`, and `toString()`

---

## Hands-On Exercise 1: Your First Record

Create `src/chapter03/RecordBasics.java`:

```java
package chapter03;

// Define a record - it's this simple!
record Point(int x, int y) {}

record User(String name, String email, int age) {}

public class RecordBasics {
    public static void main(String[] args) {
        // Create record instances
        var p1 = new Point(10, 20);
        var p2 = new Point(10, 20);
        var p3 = new Point(30, 40);

        // Access components (not getX(), just x())
        System.out.println("=== Accessing Components ===");
        System.out.println("p1.x() = " + p1.x());
        System.out.println("p1.y() = " + p1.y());

        // Automatic toString()
        System.out.println("\n=== Automatic toString() ===");
        System.out.println("p1 = " + p1);
        System.out.println("p3 = " + p3);

        // Automatic equals() - compares all components
        System.out.println("\n=== Automatic equals() ===");
        System.out.println("p1.equals(p2) = " + p1.equals(p2));  // true
        System.out.println("p1.equals(p3) = " + p1.equals(p3));  // false
        System.out.println("p1 == p2 = " + (p1 == p2));          // false (different objects)

        // Automatic hashCode() - consistent with equals()
        System.out.println("\n=== Automatic hashCode() ===");
        System.out.println("p1.hashCode() = " + p1.hashCode());
        System.out.println("p2.hashCode() = " + p2.hashCode());
        System.out.println("Same hash: " + (p1.hashCode() == p2.hashCode()));

        // Works great in collections
        var user = new User("Alice", "alice@example.com", 30);
        System.out.println("\n=== User Record ===");
        System.out.println(user);
        System.out.println("Name: " + user.name());
        System.out.println("Email: " + user.email());
    }
}
```

**Run it:**
```bash
java src/chapter03/RecordBasics.java
```

---

## Records are Immutable

Once created, a record's values cannot change:

```java
record Person(String name, int age) {}

var person = new Person("Bob", 25);
// person.name = "Alice";  // ERROR: Cannot assign to a component
// person.age++;           // ERROR: Cannot modify

// To "change" a record, create a new one
var olderPerson = new Person(person.name(), person.age() + 1);
```

This immutability makes records:
- **Thread-safe** by default
- **Safe as Map keys** (hashCode won't change)
- **Predictable** (no unexpected mutations)

---

## Hands-On Exercise 2: Custom Constructors

Create `src/chapter03/RecordConstructors.java`:

```java
package chapter03;

// Compact constructor - for validation
record Email(String address) {
    // Compact constructor - no parameter list, modifies 'address' directly
    public Email {
        if (address == null || !address.contains("@")) {
            throw new IllegalArgumentException("Invalid email: " + address);
        }
        // You can transform the value
        address = address.toLowerCase().trim();
    }
}

// Record with validation and normalization
record PhoneNumber(String countryCode, String number) {
    public PhoneNumber {
        // Validation
        if (countryCode == null || !countryCode.startsWith("+")) {
            throw new IllegalArgumentException("Country code must start with +");
        }
        if (number == null || number.length() < 7) {
            throw new IllegalArgumentException("Number too short");
        }
        // Normalization - remove spaces and dashes
        number = number.replaceAll("[\\s-]", "");
    }
}

// Custom canonical constructor (explicit version)
record Temperature(double celsius) {
    // Explicit canonical constructor
    public Temperature(double celsius) {
        if (celsius < -273.15) {
            throw new IllegalArgumentException("Below absolute zero!");
        }
        this.celsius = celsius;
    }

    // Additional constructors must delegate to canonical
    public Temperature(double value, String unit) {
        this(convertToCelsius(value, unit));
    }

    private static double convertToCelsius(double value, String unit) {
        return switch (unit.toUpperCase()) {
            case "C", "CELSIUS" -> value;
            case "F", "FAHRENHEIT" -> (value - 32) * 5 / 9;
            case "K", "KELVIN" -> value - 273.15;
            default -> throw new IllegalArgumentException("Unknown unit: " + unit);
        };
    }

    // Convenience method
    public double fahrenheit() {
        return celsius * 9 / 5 + 32;
    }

    public double kelvin() {
        return celsius + 273.15;
    }
}

public class RecordConstructors {
    public static void main(String[] args) {
        // Email validation
        System.out.println("=== Email Validation ===");
        var email = new Email("  Alice@Example.COM  ");
        System.out.println("Normalized: " + email.address());

        try {
            new Email("invalid-email");
        } catch (IllegalArgumentException e) {
            System.out.println("Caught: " + e.getMessage());
        }

        // Phone number
        System.out.println("\n=== Phone Number ===");
        var phone = new PhoneNumber("+1", "555-123-4567");
        System.out.println("Phone: " + phone);
        System.out.println("Normalized number: " + phone.number());

        // Temperature conversion
        System.out.println("\n=== Temperature ===");
        var tempC = new Temperature(100);
        System.out.println("100°C = " + tempC.fahrenheit() + "°F");

        var tempF = new Temperature(32, "F");
        System.out.println("32°F = " + tempF.celsius() + "°C");

        var tempK = new Temperature(0, "K");
        System.out.println("0K = " + tempK.celsius() + "°C");
    }
}
```

---

## Hands-On Exercise 3: Records with Methods

Records can have instance methods, static methods, and static fields:

Create `src/chapter03/RecordMethods.java`:

```java
package chapter03;

import java.time.LocalDate;
import java.time.Period;

record Person(String firstName, String lastName, LocalDate birthDate) {

    // Static field (allowed)
    public static final String SPECIES = "Homo Sapiens";

    // Instance method
    public String fullName() {
        return firstName + " " + lastName;
    }

    // Another instance method
    public int age() {
        return Period.between(birthDate, LocalDate.now()).getYears();
    }

    // Static factory method
    public static Person of(String fullName, LocalDate birthDate) {
        String[] parts = fullName.split(" ", 2);
        return new Person(
            parts[0],
            parts.length > 1 ? parts[1] : "",
            birthDate
        );
    }

    // Method with complex logic
    public boolean isAdult() {
        return age() >= 18;
    }

    // Methods can use other methods
    public String greeting() {
        return "Hello, I'm " + fullName() + " and I'm " + age() + " years old.";
    }
}

record Rectangle(double width, double height) {
    // Computed properties as methods
    public double area() {
        return width * height;
    }

    public double perimeter() {
        return 2 * (width + height);
    }

    public boolean isSquare() {
        return width == height;
    }

    public Rectangle scale(double factor) {
        return new Rectangle(width * factor, height * factor);
    }

    // Static factory methods
    public static Rectangle square(double side) {
        return new Rectangle(side, side);
    }
}

public class RecordMethods {
    public static void main(String[] args) {
        // Person with methods
        System.out.println("=== Person Record ===");
        var person = new Person("John", "Doe", LocalDate.of(1990, 5, 15));
        System.out.println("Full name: " + person.fullName());
        System.out.println("Age: " + person.age());
        System.out.println("Is adult: " + person.isAdult());
        System.out.println("Greeting: " + person.greeting());
        System.out.println("Species: " + Person.SPECIES);

        // Static factory
        var person2 = Person.of("Jane Smith", LocalDate.of(1985, 3, 20));
        System.out.println("\nCreated from factory: " + person2);

        // Rectangle with computed properties
        System.out.println("\n=== Rectangle Record ===");
        var rect = new Rectangle(10, 5);
        System.out.println("Rectangle: " + rect);
        System.out.println("Area: " + rect.area());
        System.out.println("Perimeter: " + rect.perimeter());
        System.out.println("Is square: " + rect.isSquare());

        var scaled = rect.scale(2);
        System.out.println("Scaled 2x: " + scaled);

        var square = Rectangle.square(7);
        System.out.println("\nSquare: " + square);
        System.out.println("Is square: " + square.isSquare());
    }
}
```

---

## Hands-On Exercise 4: Records with Interfaces

Create `src/chapter03/RecordInterfaces.java`:

```java
package chapter03;

import java.util.*;

// Records can implement interfaces
interface Drawable {
    void draw();
}

interface Measurable {
    double measure();
}

record Circle(double radius) implements Drawable, Measurable {
    @Override
    public void draw() {
        System.out.println("Drawing circle with radius " + radius);
    }

    @Override
    public double measure() {
        return Math.PI * radius * radius; // area
    }

    public double circumference() {
        return 2 * Math.PI * radius;
    }
}

record Square(double side) implements Drawable, Measurable {
    @Override
    public void draw() {
        System.out.println("Drawing square with side " + side);
    }

    @Override
    public double measure() {
        return side * side;
    }
}

// Comparable interface
record Product(String name, double price) implements Comparable<Product> {
    @Override
    public int compareTo(Product other) {
        return Double.compare(this.price, other.price);
    }
}

public class RecordInterfaces {
    public static void main(String[] args) {
        // Polymorphism with records
        System.out.println("=== Drawable Shapes ===");
        List<Drawable> shapes = List.of(
            new Circle(5),
            new Square(4),
            new Circle(3)
        );

        for (var shape : shapes) {
            shape.draw();
        }

        // Measurable
        System.out.println("\n=== Measurable Shapes ===");
        List<Measurable> measurables = List.of(
            new Circle(5),
            new Square(4)
        );

        for (var m : measurables) {
            System.out.printf("Area: %.2f%n", m.measure());
        }

        // Sorting with Comparable
        System.out.println("\n=== Sortable Products ===");
        var products = new ArrayList<>(List.of(
            new Product("Laptop", 999.99),
            new Product("Mouse", 29.99),
            new Product("Keyboard", 79.99)
        ));

        System.out.println("Before sorting: " + products);
        Collections.sort(products);
        System.out.println("After sorting: " + products);
    }
}
```

---

## Hands-On Exercise 5: Nested and Local Records

Create `src/chapter03/NestedRecords.java`:

```java
package chapter03;

import java.util.*;

// Records can be nested in classes
class Order {
    // Nested record
    public record Item(String productId, String name, int quantity, double price) {
        public double total() {
            return quantity * price;
        }
    }

    public record ShippingAddress(String street, String city, String zipCode, String country) {}

    private final String orderId;
    private final List<Item> items;
    private final ShippingAddress shippingAddress;

    public Order(String orderId, List<Item> items, ShippingAddress shippingAddress) {
        this.orderId = orderId;
        this.items = List.copyOf(items);
        this.shippingAddress = shippingAddress;
    }

    public double total() {
        return items.stream().mapToDouble(Item::total).sum();
    }

    public void printOrder() {
        System.out.println("Order: " + orderId);
        System.out.println("Ship to: " + shippingAddress);
        System.out.println("Items:");
        for (var item : items) {
            System.out.printf("  - %s x%d @ $%.2f = $%.2f%n",
                item.name(), item.quantity(), item.price(), item.total());
        }
        System.out.printf("Total: $%.2f%n", total());
    }
}

public class NestedRecords {
    public static void main(String[] args) {
        // Using nested records
        var items = List.of(
            new Order.Item("P001", "Widget", 3, 19.99),
            new Order.Item("P002", "Gadget", 1, 49.99),
            new Order.Item("P003", "Gizmo", 2, 9.99)
        );

        var address = new Order.ShippingAddress(
            "123 Main St", "Springfield", "12345", "USA"
        );

        var order = new Order("ORD-2024-001", items, address);
        order.printOrder();

        // Local records (defined inside a method)
        System.out.println("\n=== Local Records ===");
        processData();
    }

    static void processData() {
        // Local record - only visible in this method
        record Statistics(double min, double max, double average, int count) {
            public double range() {
                return max - min;
            }
        }

        var numbers = List.of(10.5, 20.3, 15.7, 8.2, 25.1);

        var stats = new Statistics(
            numbers.stream().mapToDouble(d -> d).min().orElse(0),
            numbers.stream().mapToDouble(d -> d).max().orElse(0),
            numbers.stream().mapToDouble(d -> d).average().orElse(0),
            numbers.size()
        );

        System.out.println("Statistics: " + stats);
        System.out.println("Range: " + stats.range());
    }
}
```

---

## Hands-On Exercise 6: Records in Real Applications

Create `src/chapter03/RealWorldRecords.java`:

```java
package chapter03;

import java.time.*;
import java.util.*;
import java.util.stream.*;

// API Response modeling
record ApiResponse<T>(T data, boolean success, String message, Instant timestamp) {
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(data, true, "OK", Instant.now());
    }

    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(null, false, message, Instant.now());
    }
}

// Configuration
record DatabaseConfig(String host, int port, String database, String username) {
    public DatabaseConfig {
        if (port < 1 || port > 65535) {
            throw new IllegalArgumentException("Invalid port: " + port);
        }
    }

    public String connectionString() {
        return "jdbc:postgresql://%s:%d/%s".formatted(host, port, database);
    }
}

// Event sourcing
sealed interface DomainEvent permits UserCreated, UserUpdated, UserDeleted {}

record UserCreated(String userId, String name, String email, Instant createdAt)
    implements DomainEvent {}

record UserUpdated(String userId, Map<String, Object> changes, Instant updatedAt)
    implements DomainEvent {}

record UserDeleted(String userId, String reason, Instant deletedAt)
    implements DomainEvent {}

// DTO for HTTP requests
record CreateUserRequest(String name, String email, String password) {
    public CreateUserRequest {
        Objects.requireNonNull(name, "name is required");
        Objects.requireNonNull(email, "email is required");
        Objects.requireNonNull(password, "password is required");

        if (password.length() < 8) {
            throw new IllegalArgumentException("Password must be at least 8 characters");
        }
    }
}

// Pagination
record Page<T>(List<T> content, int pageNumber, int pageSize, long totalElements) {
    public int totalPages() {
        return (int) Math.ceil((double) totalElements / pageSize);
    }

    public boolean hasNext() {
        return pageNumber < totalPages() - 1;
    }

    public boolean hasPrevious() {
        return pageNumber > 0;
    }

    public boolean isEmpty() {
        return content.isEmpty();
    }

    public <R> Page<R> map(java.util.function.Function<T, R> mapper) {
        var mappedContent = content.stream().map(mapper).collect(Collectors.toList());
        return new Page<>(mappedContent, pageNumber, pageSize, totalElements);
    }
}

public class RealWorldRecords {
    public static void main(String[] args) {
        // API Responses
        System.out.println("=== API Responses ===");
        var successResponse = ApiResponse.success(Map.of("id", 123, "status", "active"));
        System.out.println("Success: " + successResponse);

        var errorResponse = ApiResponse.<Map<String, Object>>error("User not found");
        System.out.println("Error: " + errorResponse);

        // Database Config
        System.out.println("\n=== Database Config ===");
        var dbConfig = new DatabaseConfig("localhost", 5432, "myapp", "admin");
        System.out.println("Config: " + dbConfig);
        System.out.println("Connection: " + dbConfig.connectionString());

        // Domain Events
        System.out.println("\n=== Domain Events ===");
        List<DomainEvent> events = List.of(
            new UserCreated("U001", "Alice", "alice@example.com", Instant.now()),
            new UserUpdated("U001", Map.of("email", "alice.new@example.com"), Instant.now()),
            new UserDeleted("U002", "Account closed by user", Instant.now())
        );

        for (var event : events) {
            System.out.println(event);
        }

        // Pagination
        System.out.println("\n=== Pagination ===");
        var users = List.of("Alice", "Bob", "Charlie", "David", "Eve");
        var page = new Page<>(users, 0, 5, 23);

        System.out.println("Page: " + page);
        System.out.println("Total pages: " + page.totalPages());
        System.out.println("Has next: " + page.hasNext());
        System.out.println("Has previous: " + page.hasPrevious());

        // Mapping
        var upperPage = page.map(String::toUpperCase);
        System.out.println("Mapped: " + upperPage.content());

        // Request validation
        System.out.println("\n=== Request Validation ===");
        try {
            var request = new CreateUserRequest("John", "john@example.com", "short");
        } catch (IllegalArgumentException e) {
            System.out.println("Validation failed: " + e.getMessage());
        }

        var validRequest = new CreateUserRequest("John", "john@example.com", "securePassword123");
        System.out.println("Valid request: " + validRequest);
    }
}
```

---

## What Records Cannot Do

```java
// Records CANNOT:

// 1. Extend other classes (they implicitly extend java.lang.Record)
// record Child(int x) extends Parent {}  // ERROR!

// 2. Be extended (they are implicitly final)
// class SubRecord extends MyRecord {}  // ERROR!

// 3. Have instance fields other than record components
// record Bad(int x) {
//     private int y;  // ERROR: Cannot declare instance fields
// }

// 4. Have mutable components (well, you can, but you shouldn't)
// record Mutable(List<String> items) {}  // Compiles but dangerous!

// Records CAN:

// 1. Implement interfaces
record Good(int x) implements Comparable<Good> {
    public int compareTo(Good other) { return Integer.compare(x, other.x); }
}

// 2. Have static fields
record WithStatic(int x) {
    public static final int DEFAULT = 0;
}

// 3. Be generic
record Pair<A, B>(A first, B second) {}

// 4. Be nested or local
class Outer {
    record Inner(int x) {}
}
```

---

## Practice Exercises

### Exercise 3.1: Create a Money Record
Create a `Money` record with `amount` (BigDecimal) and `currency` (String). Add:
- Validation that amount is not negative and currency is 3 characters
- An `add(Money other)` method that returns a new Money (must be same currency)
- A static factory `Money.of(String amount, String currency)`

<details>
<summary>Click for solution</summary>

```java
import java.math.BigDecimal;

record Money(BigDecimal amount, String currency) {
    public Money {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
        }
        if (currency == null || currency.length() != 3) {
            throw new IllegalArgumentException("Currency must be 3 characters");
        }
        currency = currency.toUpperCase();
    }

    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }

    public static Money of(String amount, String currency) {
        return new Money(new BigDecimal(amount), currency);
    }
}
```
</details>

### Exercise 3.2: Range Record
Create a `Range` record representing a numeric range with `start` and `end`. Add:
- Validation that `start <= end`
- `contains(int value)` method
- `overlaps(Range other)` method
- `length()` method

<details>
<summary>Click for solution</summary>

```java
record Range(int start, int end) {
    public Range {
        if (start > end) {
            throw new IllegalArgumentException("Start must be <= end");
        }
    }

    public boolean contains(int value) {
        return value >= start && value <= end;
    }

    public boolean overlaps(Range other) {
        return this.start <= other.end && this.end >= other.start;
    }

    public int length() {
        return end - start;
    }
}
```
</details>

### Exercise 3.3: Immutable Tree Node
Create a `TreeNode<T>` record that has a value and a list of children. Ensure the children list is truly immutable.

<details>
<summary>Click for solution</summary>

```java
import java.util.*;

record TreeNode<T>(T value, List<TreeNode<T>> children) {
    public TreeNode {
        // Create a defensive copy to ensure immutability
        children = children == null ? List.of() : List.copyOf(children);
    }

    public TreeNode(T value) {
        this(value, List.of());
    }

    public boolean isLeaf() {
        return children.isEmpty();
    }

    public TreeNode<T> withChild(TreeNode<T> child) {
        var newChildren = new ArrayList<>(children);
        newChildren.add(child);
        return new TreeNode<>(value, newChildren);
    }
}
```
</details>

---

## Key Takeaways

1. **Records are immutable data carriers** - components are final
2. **Automatic generation** of constructor, accessors, `equals()`, `hashCode()`, `toString()`
3. **Accessor methods use component names** - `user.name()` not `user.getName()`
4. **Compact constructors** for validation and normalization
5. **Can have methods, static fields, and implement interfaces**
6. **Cannot extend classes or be extended** (implicitly final, extends Record)
7. **Great for DTOs, value objects, API responses, and configuration**

---

## What's Next?

In [Chapter 4: Sealed Classes](04-sealed-classes.md), you'll learn how to control which classes can extend or implement your types.

---

[← Previous Chapter](02-text-blocks.md) | [Back to Contents](../README.md) | [Next Chapter →](04-sealed-classes.md)
