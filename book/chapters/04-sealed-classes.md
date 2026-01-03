# Chapter 4: Sealed Classes

> **Java Version:** 17+ | **Difficulty:** Intermediate | **Time:** 35 minutes

## The Problem

You're modeling a domain where you need to restrict inheritance. Consider a payment system:

```java
// Old approach: Anyone can extend PaymentMethod
public abstract class PaymentMethod {
    public abstract void process(double amount);
}

// These are legitimate
class CreditCard extends PaymentMethod { ... }
class BankTransfer extends PaymentMethod { ... }

// But what about this?
class BitcoinPayment extends PaymentMethod { ... }  // Do we support this?
class CashPayment extends PaymentMethod { ... }     // Is this valid?
```

Before Java 17, you had two options:
1. Make the class `final` - no inheritance at all
2. Leave it open - anyone can extend it

There was no middle ground.

## The Solution: Sealed Classes

Sealed classes let you **explicitly declare** which classes are permitted to extend them:

```java
public sealed class PaymentMethod
    permits CreditCard, BankTransfer, DigitalWallet {
    // Only these three can extend PaymentMethod
}
```

The compiler enforces this - no other class can extend `PaymentMethod`.

---

## Hands-On Exercise 1: Your First Sealed Class

Create `src/chapter04/SealedBasics.java`:

```java
package chapter04;

// A sealed class must declare its permitted subclasses
sealed class Shape permits Circle, Rectangle, Triangle {
    public abstract double area();
}

// Permitted subclass 1: 'final' means no further subclasses
final class Circle extends Shape {
    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }

    @Override
    public String toString() {
        return "Circle(radius=" + radius + ")";
    }
}

// Permitted subclass 2: 'final'
final class Rectangle extends Shape {
    private final double width, height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() {
        return width * height;
    }

    @Override
    public String toString() {
        return "Rectangle(" + width + "x" + height + ")";
    }
}

// Permitted subclass 3: 'final'
final class Triangle extends Shape {
    private final double base, height;

    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }

    @Override
    public double area() {
        return 0.5 * base * height;
    }

    @Override
    public String toString() {
        return "Triangle(base=" + base + ", height=" + height + ")";
    }
}

// This would NOT compile:
// class Pentagon extends Shape { }  // ERROR: not in permits clause

public class SealedBasics {
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle(5),
            new Rectangle(4, 6),
            new Triangle(3, 4)
        };

        System.out.println("=== Shape Areas ===");
        for (Shape shape : shapes) {
            System.out.printf("%s -> area = %.2f%n", shape, shape.area());
        }

        // The compiler knows all possible subtypes!
        System.out.println("\n=== Pattern Matching (Preview of Ch. 6) ===");
        for (Shape shape : shapes) {
            String description = switch (shape) {
                case Circle c -> "A circle with radius " + c.toString();
                case Rectangle r -> "A rectangle: " + r.toString();
                case Triangle t -> "A triangle: " + t.toString();
                // No default needed - compiler knows all cases are covered!
            };
            System.out.println(description);
        }
    }
}
```

**Run it:**
```bash
java src/chapter04/SealedBasics.java
```

---

## The Three Modifiers for Permitted Subclasses

Every direct subclass of a sealed class must be one of:

| Modifier | Meaning |
|----------|---------|
| `final` | Cannot be extended further |
| `sealed` | Can only be extended by its own permitted subclasses |
| `non-sealed` | Reopens the hierarchy - any class can extend it |

---

## Hands-On Exercise 2: Hierarchical Sealing

Create `src/chapter04/HierarchicalSealing.java`:

```java
package chapter04;

// Top-level sealed class
sealed class Vehicle permits Car, Truck, Motorcycle {
    private final String brand;

    public Vehicle(String brand) {
        this.brand = brand;
    }

    public String brand() { return brand; }
}

// 'sealed' subclass - restricts its own subclasses
sealed class Car extends Vehicle permits Sedan, SUV, SportsCar {
    public Car(String brand) {
        super(brand);
    }
}

// 'final' - end of the line
final class Sedan extends Car {
    private final int doors;

    public Sedan(String brand, int doors) {
        super(brand);
        this.doors = doors;
    }

    @Override
    public String toString() {
        return brand() + " Sedan (" + doors + " doors)";
    }
}

final class SUV extends Car {
    private final boolean fourWheelDrive;

    public SUV(String brand, boolean fourWheelDrive) {
        super(brand);
        this.fourWheelDrive = fourWheelDrive;
    }

    @Override
    public String toString() {
        return brand() + " SUV" + (fourWheelDrive ? " 4WD" : "");
    }
}

final class SportsCar extends Car {
    private final int horsePower;

    public SportsCar(String brand, int horsePower) {
        super(brand);
        this.horsePower = horsePower;
    }

    @Override
    public String toString() {
        return brand() + " Sports Car (" + horsePower + " HP)";
    }
}

// 'non-sealed' - opens up for extension
non-sealed class Truck extends Vehicle {
    public Truck(String brand) {
        super(brand);
    }
}

// Since Truck is non-sealed, anyone can extend it
class PickupTruck extends Truck {
    public PickupTruck(String brand) {
        super(brand);
    }

    @Override
    public String toString() {
        return brand() + " Pickup Truck";
    }
}

class SemiTruck extends Truck {
    public SemiTruck(String brand) {
        super(brand);
    }

    @Override
    public String toString() {
        return brand() + " Semi Truck";
    }
}

// 'final' subclass
final class Motorcycle extends Vehicle {
    private final String type;

    public Motorcycle(String brand, String type) {
        super(brand);
        this.type = type;
    }

    @Override
    public String toString() {
        return brand() + " " + type + " Motorcycle";
    }
}

public class HierarchicalSealing {
    public static void main(String[] args) {
        Vehicle[] vehicles = {
            new Sedan("Toyota", 4),
            new SUV("Jeep", true),
            new SportsCar("Ferrari", 710),
            new PickupTruck("Ford"),
            new SemiTruck("Volvo"),
            new Motorcycle("Harley-Davidson", "Cruiser")
        };

        System.out.println("=== Vehicle Fleet ===");
        for (Vehicle v : vehicles) {
            System.out.println(v);
        }

        // Hierarchy:
        // Vehicle (sealed)
        // ├── Car (sealed)
        // │   ├── Sedan (final)
        // │   ├── SUV (final)
        // │   └── SportsCar (final)
        // ├── Truck (non-sealed)
        // │   ├── PickupTruck (can extend freely)
        // │   └── SemiTruck (can extend freely)
        // └── Motorcycle (final)
    }
}
```

---

## Hands-On Exercise 3: Sealed Interfaces

Interfaces can also be sealed:

Create `src/chapter04/SealedInterfaces.java`:

```java
package chapter04;

import java.math.BigDecimal;

// Sealed interface
sealed interface Payment permits CashPayment, CardPayment, DigitalPayment {
    BigDecimal amount();
    String description();
}

// Record implementing sealed interface
record CashPayment(BigDecimal amount) implements Payment {
    @Override
    public String description() {
        return "Cash payment of $" + amount;
    }
}

// Another sealed interface extending the parent
sealed interface CardPayment extends Payment permits CreditCardPayment, DebitCardPayment {
    String cardNumber();  // last 4 digits only
}

record CreditCardPayment(BigDecimal amount, String cardNumber, String cardNetwork)
    implements CardPayment {
    @Override
    public String description() {
        return cardNetwork + " credit card (****" + cardNumber + "): $" + amount;
    }
}

record DebitCardPayment(BigDecimal amount, String cardNumber, String bankName)
    implements CardPayment {
    @Override
    public String description() {
        return bankName + " debit card (****" + cardNumber + "): $" + amount;
    }
}

// Non-sealed to allow future digital payment types
non-sealed interface DigitalPayment extends Payment {
    String provider();
}

record PayPalPayment(BigDecimal amount, String email) implements DigitalPayment {
    @Override
    public String provider() { return "PayPal"; }

    @Override
    public String description() {
        return "PayPal (" + email + "): $" + amount;
    }
}

record ApplePayPayment(BigDecimal amount, String deviceId) implements DigitalPayment {
    @Override
    public String provider() { return "Apple Pay"; }

    @Override
    public String description() {
        return "Apple Pay: $" + amount;
    }
}

// Anyone can create new DigitalPayment types
record CryptoPayment(BigDecimal amount, String walletAddress, String currency)
    implements DigitalPayment {
    @Override
    public String provider() { return currency; }

    @Override
    public String description() {
        return currency + " (" + walletAddress.substring(0, 8) + "...): $" + amount;
    }
}

public class SealedInterfaces {
    public static void main(String[] args) {
        Payment[] payments = {
            new CashPayment(new BigDecimal("50.00")),
            new CreditCardPayment(new BigDecimal("125.99"), "4242", "Visa"),
            new DebitCardPayment(new BigDecimal("75.50"), "1234", "Chase"),
            new PayPalPayment(new BigDecimal("200.00"), "user@example.com"),
            new ApplePayPayment(new BigDecimal("15.00"), "device-123"),
            new CryptoPayment(new BigDecimal("500.00"), "0x1234567890abcdef", "Ethereum")
        };

        System.out.println("=== Payment Processing ===\n");

        BigDecimal total = BigDecimal.ZERO;
        for (Payment payment : payments) {
            System.out.println(payment.description());
            total = total.add(payment.amount());
        }

        System.out.println("\nTotal: $" + total);

        // Process by type
        System.out.println("\n=== Processing Fees ===");
        for (Payment payment : payments) {
            BigDecimal fee = calculateFee(payment);
            System.out.printf("%s -> Fee: $%.2f%n",
                payment.getClass().getSimpleName(), fee);
        }
    }

    static BigDecimal calculateFee(Payment payment) {
        return switch (payment) {
            case CashPayment p -> BigDecimal.ZERO;  // No fee for cash
            case CreditCardPayment p -> p.amount().multiply(new BigDecimal("0.029"));  // 2.9%
            case DebitCardPayment p -> new BigDecimal("0.50");  // Flat fee
            case DigitalPayment p -> p.amount().multiply(new BigDecimal("0.025"));  // 2.5%
        };
    }
}
```

---

## Hands-On Exercise 4: Domain Modeling with Sealed Types

Create `src/chapter04/DomainModeling.java`:

```java
package chapter04;

import java.time.*;
import java.util.*;

// Order state machine using sealed classes
sealed interface OrderState
    permits OrderState.Pending, OrderState.Confirmed, OrderState.Shipped,
            OrderState.Delivered, OrderState.Cancelled {

    String status();
    Instant timestamp();

    record Pending(Instant timestamp) implements OrderState {
        @Override public String status() { return "PENDING"; }
    }

    record Confirmed(Instant timestamp, String confirmationCode) implements OrderState {
        @Override public String status() { return "CONFIRMED"; }
    }

    record Shipped(Instant timestamp, String trackingNumber, String carrier) implements OrderState {
        @Override public String status() { return "SHIPPED"; }
    }

    record Delivered(Instant timestamp, String signedBy) implements OrderState {
        @Override public String status() { return "DELIVERED"; }
    }

    record Cancelled(Instant timestamp, String reason) implements OrderState {
        @Override public String status() { return "CANCELLED"; }
    }
}

// Result type - functional error handling
sealed interface Result<T> permits Result.Success, Result.Failure {
    record Success<T>(T value) implements Result<T> {}
    record Failure<T>(String error, Exception cause) implements Result<T> {
        public Failure(String error) {
            this(error, null);
        }
    }

    default boolean isSuccess() {
        return this instanceof Success;
    }

    default T getOrThrow() {
        return switch (this) {
            case Success<T> s -> s.value();
            case Failure<T> f -> throw new RuntimeException(f.error(), f.cause());
        };
    }

    default T getOrDefault(T defaultValue) {
        return switch (this) {
            case Success<T> s -> s.value();
            case Failure<T> f -> defaultValue;
        };
    }
}

// Expression AST (Abstract Syntax Tree)
sealed interface Expr permits Expr.Num, Expr.Add, Expr.Mul, Expr.Neg {
    record Num(double value) implements Expr {}
    record Add(Expr left, Expr right) implements Expr {}
    record Mul(Expr left, Expr right) implements Expr {}
    record Neg(Expr operand) implements Expr {}

    static double evaluate(Expr expr) {
        return switch (expr) {
            case Num n -> n.value();
            case Add a -> evaluate(a.left()) + evaluate(a.right());
            case Mul m -> evaluate(m.left()) * evaluate(m.right());
            case Neg n -> -evaluate(n.operand());
        };
    }
}

public class DomainModeling {
    public static void main(String[] args) {
        // Order State Machine
        System.out.println("=== Order State Machine ===");
        List<OrderState> orderHistory = List.of(
            new OrderState.Pending(Instant.now().minusSeconds(3600)),
            new OrderState.Confirmed(Instant.now().minusSeconds(3000), "CONF-12345"),
            new OrderState.Shipped(Instant.now().minusSeconds(1800), "TRK-9876", "FedEx"),
            new OrderState.Delivered(Instant.now(), "John Smith")
        );

        for (OrderState state : orderHistory) {
            System.out.println(state.status() + " at " + state.timestamp());
            printStateDetails(state);
        }

        // Result type
        System.out.println("\n=== Result Type ===");
        Result<Integer> success = new Result.Success<>(42);
        Result<Integer> failure = new Result.Failure<>("Division by zero");

        System.out.println("Success value: " + success.getOrDefault(-1));
        System.out.println("Failure value: " + failure.getOrDefault(-1));

        // Expression evaluation
        System.out.println("\n=== Expression AST ===");
        // (3 + 4) * 2
        Expr expr = new Expr.Mul(
            new Expr.Add(new Expr.Num(3), new Expr.Num(4)),
            new Expr.Num(2)
        );
        System.out.println("(3 + 4) * 2 = " + Expr.evaluate(expr));

        // -(5 + 3)
        Expr negExpr = new Expr.Neg(
            new Expr.Add(new Expr.Num(5), new Expr.Num(3))
        );
        System.out.println("-(5 + 3) = " + Expr.evaluate(negExpr));
    }

    static void printStateDetails(OrderState state) {
        String details = switch (state) {
            case OrderState.Pending p -> "  Waiting for confirmation";
            case OrderState.Confirmed c -> "  Code: " + c.confirmationCode();
            case OrderState.Shipped s -> "  Tracking: " + s.trackingNumber() + " via " + s.carrier();
            case OrderState.Delivered d -> "  Signed by: " + d.signedBy();
            case OrderState.Cancelled c -> "  Reason: " + c.reason();
        };
        System.out.println(details);
    }
}
```

---

## Rules Summary

### Sealed Class Rules

1. Use `sealed` modifier and `permits` clause
2. Permitted classes must be in the same module (or package if unnamed module)
3. Every permitted class must extend the sealed class
4. Each permitted class must be `final`, `sealed`, or `non-sealed`

### When to Omit `permits`

If all subclasses are in the same file, you can omit `permits`:

```java
// All in the same file - permits clause is optional
sealed class Animal { }
final class Dog extends Animal { }
final class Cat extends Animal { }
// Compiler infers: permits Dog, Cat
```

---

## Sealed vs. Other Approaches

| Approach | Use Case |
|----------|----------|
| `final class` | No inheritance at all |
| Regular `class` | Open inheritance |
| `abstract class` | Force inheritance, no restriction on who |
| `sealed class` | Controlled inheritance with known subtypes |

---

## Practice Exercises

### Exercise 4.1: HTTP Response
Create a sealed hierarchy for HTTP responses:
- `HttpResponse` (sealed)
  - `Success` with status code and body
  - `Redirect` with status code and location URL
  - `ClientError` with status code and error message
  - `ServerError` with status code and error message

<details>
<summary>Click for solution</summary>

```java
sealed interface HttpResponse
    permits HttpResponse.Success, HttpResponse.Redirect,
            HttpResponse.ClientError, HttpResponse.ServerError {

    int statusCode();

    record Success(int statusCode, String body) implements HttpResponse {}

    record Redirect(int statusCode, String location) implements HttpResponse {}

    record ClientError(int statusCode, String message) implements HttpResponse {
        public boolean isNotFound() { return statusCode == 404; }
        public boolean isUnauthorized() { return statusCode == 401; }
    }

    record ServerError(int statusCode, String message) implements HttpResponse {
        public boolean isInternalError() { return statusCode == 500; }
    }
}
```
</details>

### Exercise 4.2: JSON Value
Model JSON values as a sealed type hierarchy:
- `JsonValue` (sealed)
  - `JsonNull`
  - `JsonBoolean` (with boolean value)
  - `JsonNumber` (with double value)
  - `JsonString` (with String value)
  - `JsonArray` (with List of JsonValue)
  - `JsonObject` (with Map of String to JsonValue)

<details>
<summary>Click for solution</summary>

```java
import java.util.*;

sealed interface JsonValue
    permits JsonValue.JsonNull, JsonValue.JsonBoolean, JsonValue.JsonNumber,
            JsonValue.JsonString, JsonValue.JsonArray, JsonValue.JsonObject {

    record JsonNull() implements JsonValue {}

    record JsonBoolean(boolean value) implements JsonValue {}

    record JsonNumber(double value) implements JsonValue {}

    record JsonString(String value) implements JsonValue {}

    record JsonArray(List<JsonValue> elements) implements JsonValue {
        public JsonArray {
            elements = List.copyOf(elements);
        }
    }

    record JsonObject(Map<String, JsonValue> members) implements JsonValue {
        public JsonObject {
            members = Map.copyOf(members);
        }
    }
}
```
</details>

### Exercise 4.3: State Machine
Create a sealed type for a document workflow:
- Draft (can be submitted)
- Submitted (can be approved or rejected)
- Approved (final state)
- Rejected (can be revised back to Draft)

Add methods to each state that return the valid next states.

<details>
<summary>Click for solution</summary>

```java
import java.time.Instant;

sealed interface DocumentState
    permits DocumentState.Draft, DocumentState.Submitted,
            DocumentState.Approved, DocumentState.Rejected {

    String name();

    record Draft(String author, Instant createdAt) implements DocumentState {
        @Override public String name() { return "DRAFT"; }

        public Submitted submit(String submittedBy) {
            return new Submitted(submittedBy, Instant.now());
        }
    }

    record Submitted(String submittedBy, Instant submittedAt) implements DocumentState {
        @Override public String name() { return "SUBMITTED"; }

        public Approved approve(String approvedBy) {
            return new Approved(approvedBy, Instant.now());
        }

        public Rejected reject(String rejectedBy, String reason) {
            return new Rejected(rejectedBy, reason, Instant.now());
        }
    }

    record Approved(String approvedBy, Instant approvedAt) implements DocumentState {
        @Override public String name() { return "APPROVED"; }
        // Final state - no transitions
    }

    record Rejected(String rejectedBy, String reason, Instant rejectedAt)
        implements DocumentState {
        @Override public String name() { return "REJECTED"; }

        public Draft revise(String author) {
            return new Draft(author, Instant.now());
        }
    }
}
```
</details>

---

## Key Takeaways

1. **Sealed classes restrict inheritance** - only `permits` classes can extend
2. **Three modifiers for subclasses**: `final`, `sealed`, `non-sealed`
3. **Compiler knows all subtypes** - enables exhaustive pattern matching
4. **Works with interfaces too** - `sealed interface`
5. **Great for domain modeling** - state machines, ADTs, result types
6. **Permits clause optional** if subclasses are in the same file

---

## What's Next?

In [Chapter 5: Pattern Matching for instanceof](05-pattern-matching-instanceof.md), you'll see how sealed classes combine beautifully with pattern matching to eliminate type casting boilerplate.

---

[← Previous Chapter](03-records.md) | [Back to Contents](../README.md) | [Next Chapter →](05-pattern-matching-instanceof.md)
