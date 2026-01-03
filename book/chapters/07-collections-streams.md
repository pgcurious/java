# Chapter 7: Enhanced Collections & Streams

> **Java Version:** 9-21 | **Difficulty:** Beginner-Intermediate | **Time:** 45 minutes

## The Problem

Creating small, immutable collections in Java 8 was verbose:

```java
// Old way - verbose!
List<String> list = Collections.unmodifiableList(Arrays.asList("a", "b", "c"));
Set<String> set = Collections.unmodifiableSet(new HashSet<>(Arrays.asList("a", "b", "c")));
Map<String, Integer> map = Collections.unmodifiableMap(new HashMap<String, Integer>() {{
    put("one", 1);
    put("two", 2);
}});
```

And Stream had some operations missing that you often needed.

## The Solution: Factory Methods & Stream Enhancements

Java 9+ introduced convenient factory methods and new stream operations:

```java
// New way - clean!
var list = List.of("a", "b", "c");
var set = Set.of("a", "b", "c");
var map = Map.of("one", 1, "two", 2);
```

---

## Part 1: Collection Factory Methods

### Hands-On Exercise 1: List, Set, and Map Factories

Create `src/chapter07/CollectionFactories.java`:

```java
package chapter07;

import java.util.*;

public class CollectionFactories {
    public static void main(String[] args) {
        // List.of() - creates immutable list
        System.out.println("=== List.of() ===");
        var fruits = List.of("apple", "banana", "cherry");
        System.out.println("Fruits: " + fruits);
        System.out.println("Type: " + fruits.getClass().getSimpleName());

        // Immutable - these would throw UnsupportedOperationException:
        // fruits.add("date");
        // fruits.set(0, "apricot");
        // fruits.remove("apple");

        // Set.of() - creates immutable set
        System.out.println("\n=== Set.of() ===");
        var colors = Set.of("red", "green", "blue");
        System.out.println("Colors: " + colors);

        // Duplicates throw exception:
        // var invalid = Set.of("a", "a");  // IllegalArgumentException!

        // Map.of() - creates immutable map (up to 10 entries)
        System.out.println("\n=== Map.of() ===");
        var capitals = Map.of(
            "France", "Paris",
            "Germany", "Berlin",
            "Italy", "Rome"
        );
        System.out.println("Capitals: " + capitals);

        // Map.ofEntries() - for more than 10 entries
        System.out.println("\n=== Map.ofEntries() ===");
        var moreCapitals = Map.ofEntries(
            Map.entry("France", "Paris"),
            Map.entry("Germany", "Berlin"),
            Map.entry("Italy", "Rome"),
            Map.entry("Spain", "Madrid"),
            Map.entry("Portugal", "Lisbon"),
            Map.entry("Netherlands", "Amsterdam"),
            Map.entry("Belgium", "Brussels"),
            Map.entry("Austria", "Vienna"),
            Map.entry("Poland", "Warsaw"),
            Map.entry("Czech Republic", "Prague"),
            Map.entry("Hungary", "Budapest")
        );
        System.out.println("More capitals: " + moreCapitals.size() + " entries");

        // Empty collections
        System.out.println("\n=== Empty Collections ===");
        var emptyList = List.of();
        var emptySet = Set.of();
        var emptyMap = Map.of();
        System.out.println("Empty list: " + emptyList);
        System.out.println("Empty set: " + emptySet);
        System.out.println("Empty map: " + emptyMap);

        // Single element
        var singleList = List.of("only one");
        var singleSet = Set.of("only one");
        var singleMap = Map.of("key", "value");
        System.out.println("Single list: " + singleList);

        // Null is NOT allowed!
        // List.of(null);           // NullPointerException
        // Set.of(null);            // NullPointerException
        // Map.of("key", null);     // NullPointerException
    }
}
```

**Run it:**
```bash
java src/chapter07/CollectionFactories.java
```

---

### Hands-On Exercise 2: Copying Collections

Create `src/chapter07/CopyingCollections.java`:

```java
package chapter07;

import java.util.*;

public class CopyingCollections {
    public static void main(String[] args) {
        // copyOf() creates immutable copy (Java 10+)
        System.out.println("=== copyOf() Methods ===\n");

        var mutableList = new ArrayList<>(List.of("a", "b", "c"));
        var immutableCopy = List.copyOf(mutableList);

        System.out.println("Original: " + mutableList);
        System.out.println("Copy: " + immutableCopy);

        // Modify original - copy is unaffected
        mutableList.add("d");
        mutableList.set(0, "X");

        System.out.println("\nAfter modifying original:");
        System.out.println("Original: " + mutableList);
        System.out.println("Copy: " + immutableCopy);

        // Set.copyOf()
        var mutableSet = new HashSet<>(Set.of(1, 2, 3));
        var immutableSetCopy = Set.copyOf(mutableSet);
        System.out.println("\nSet copy: " + immutableSetCopy);

        // Map.copyOf()
        var mutableMap = new HashMap<>(Map.of("a", 1, "b", 2));
        var immutableMapCopy = Map.copyOf(mutableMap);
        System.out.println("Map copy: " + immutableMapCopy);

        // toList() collector (Java 16+) - creates unmodifiable list
        System.out.println("\n=== Stream.toList() ===");
        var numbers = List.of(1, 2, 3, 4, 5);
        var doubled = numbers.stream()
            .map(n -> n * 2)
            .toList();  // Shorter than .collect(Collectors.toList())

        System.out.println("Doubled: " + doubled);
        // doubled.add(12);  // UnsupportedOperationException!
    }
}
```

---

### Hands-On Exercise 3: Sequenced Collections (Java 21)

Java 21 introduced `SequencedCollection`, `SequencedSet`, and `SequencedMap`:

Create `src/chapter07/SequencedCollections.java`:

```java
package chapter07;

import java.util.*;

public class SequencedCollections {
    public static void main(String[] args) {
        // SequencedCollection methods
        System.out.println("=== SequencedCollection (Java 21) ===\n");

        // ArrayList, LinkedList, etc. implement SequencedCollection
        var list = new ArrayList<>(List.of("first", "second", "third", "last"));

        System.out.println("Original: " + list);
        System.out.println("getFirst(): " + list.getFirst());
        System.out.println("getLast(): " + list.getLast());

        // Add at beginning/end
        list.addFirst("new first");
        list.addLast("new last");
        System.out.println("After addFirst/addLast: " + list);

        // Remove from beginning/end
        String removedFirst = list.removeFirst();
        String removedLast = list.removeLast();
        System.out.println("Removed first: " + removedFirst);
        System.out.println("Removed last: " + removedLast);
        System.out.println("After removals: " + list);

        // Reversed view
        System.out.println("\n=== Reversed View ===");
        SequencedCollection<String> reversed = list.reversed();
        System.out.println("Reversed: " + reversed);

        // The reversed view is a VIEW - changes reflect in original
        reversed.addFirst("from reversed");  // Adds to END of original
        System.out.println("Original after reversed.addFirst: " + list);

        // SequencedSet
        System.out.println("\n=== SequencedSet ===");
        var linkedSet = new LinkedHashSet<>(List.of("A", "B", "C", "D"));
        System.out.println("Set: " + linkedSet);
        System.out.println("First: " + linkedSet.getFirst());
        System.out.println("Last: " + linkedSet.getLast());
        System.out.println("Reversed: " + linkedSet.reversed());

        // SequencedMap
        System.out.println("\n=== SequencedMap ===");
        var linkedMap = new LinkedHashMap<String, Integer>();
        linkedMap.put("one", 1);
        linkedMap.put("two", 2);
        linkedMap.put("three", 3);

        System.out.println("Map: " + linkedMap);
        System.out.println("First entry: " + linkedMap.firstEntry());
        System.out.println("Last entry: " + linkedMap.lastEntry());

        // Put at beginning (Java 21)
        linkedMap.putFirst("zero", 0);
        linkedMap.putLast("four", 4);
        System.out.println("After putFirst/putLast: " + linkedMap);

        // Poll (remove and return) first/last entry
        Map.Entry<String, Integer> polled = linkedMap.pollFirstEntry();
        System.out.println("Polled first: " + polled);
        System.out.println("Map after poll: " + linkedMap);

        // Reversed view
        System.out.println("Reversed map: " + linkedMap.reversed());

        // SequencedMap.sequencedKeySet(), sequencedValues(), sequencedEntrySet()
        System.out.println("\nSequenced key set: " + linkedMap.sequencedKeySet());
        System.out.println("Sequenced values: " + linkedMap.sequencedValues());
    }
}
```

---

## Part 2: Stream API Enhancements

### Hands-On Exercise 4: New Stream Operations

Create `src/chapter07/StreamEnhancements.java`:

```java
package chapter07;

import java.util.*;
import java.util.stream.*;

public class StreamEnhancements {
    public static void main(String[] args) {
        // takeWhile() - take elements while predicate is true (Java 9)
        System.out.println("=== takeWhile() ===");
        var numbers = List.of(2, 4, 6, 8, 9, 10, 12);  // 9 breaks the pattern

        var evenPrefix = numbers.stream()
            .takeWhile(n -> n % 2 == 0)
            .toList();
        System.out.println("Numbers: " + numbers);
        System.out.println("Take while even: " + evenPrefix);

        // dropWhile() - drop elements while predicate is true (Java 9)
        System.out.println("\n=== dropWhile() ===");
        var afterOdd = numbers.stream()
            .dropWhile(n -> n % 2 == 0)
            .toList();
        System.out.println("Drop while even: " + afterOdd);

        // Practical example: parsing log entries
        System.out.println("\n=== Practical: Log Parsing ===");
        var logLines = List.of(
            "# Configuration file",
            "# Version 1.0",
            "",
            "server.host=localhost",
            "server.port=8080",
            "# Comment in middle",
            "database.url=jdbc:mysql://localhost/db"
        );

        var configLines = logLines.stream()
            .dropWhile(line -> line.startsWith("#") || line.isEmpty())
            .filter(line -> !line.startsWith("#"))
            .toList();
        System.out.println("Config lines: " + configLines);

        // Stream.ofNullable() - 0 or 1 element stream (Java 9)
        System.out.println("\n=== Stream.ofNullable() ===");
        String value = "hello";
        String nullValue = null;

        long count1 = Stream.ofNullable(value).count();
        long count2 = Stream.ofNullable(nullValue).count();
        System.out.println("ofNullable(\"hello\").count() = " + count1);
        System.out.println("ofNullable(null).count() = " + count2);

        // Useful with flatMap
        var optionalValues = List.of(
            Optional.of("a"),
            Optional.empty(),
            Optional.of("b"),
            Optional.empty(),
            Optional.of("c")
        );

        var presentValues = optionalValues.stream()
            .flatMap(opt -> opt.stream())  // Optional.stream() is Java 9
            .toList();
        System.out.println("Present values: " + presentValues);

        // Stream.iterate() with predicate (Java 9)
        System.out.println("\n=== Stream.iterate() with Predicate ===");

        // Old way (Java 8): generate infinite stream, then limit
        // Stream.iterate(1, n -> n * 2).limit(10).forEach(System.out::println);

        // New way (Java 9): built-in termination condition
        var powersOfTwo = Stream.iterate(1, n -> n < 1000, n -> n * 2)
            .toList();
        System.out.println("Powers of 2 < 1000: " + powersOfTwo);

        // Like a for loop: for (int i = 0; i < 10; i += 2)
        var evens = Stream.iterate(0, i -> i < 10, i -> i + 2)
            .toList();
        System.out.println("Even numbers 0-8: " + evens);
    }
}
```

---

### Hands-On Exercise 5: Collectors Enhancements

Create `src/chapter07/CollectorsEnhancements.java`:

```java
package chapter07;

import java.util.*;
import java.util.stream.*;
import static java.util.stream.Collectors.*;

public class CollectorsEnhancements {
    public static void main(String[] args) {
        var people = List.of(
            new Person("Alice", 30, "Engineering"),
            new Person("Bob", 25, "Engineering"),
            new Person("Charlie", 35, "Sales"),
            new Person("Diana", 28, "Sales"),
            new Person("Eve", 32, "Engineering")
        );

        // Collectors.toUnmodifiableList() (Java 10)
        System.out.println("=== toUnmodifiableList/Set/Map ===");
        var names = people.stream()
            .map(Person::name)
            .collect(toUnmodifiableList());
        System.out.println("Names (unmodifiable): " + names);

        var departments = people.stream()
            .map(Person::department)
            .collect(toUnmodifiableSet());
        System.out.println("Departments (unmodifiable): " + departments);

        // Collectors.filtering() (Java 9)
        System.out.println("\n=== Collectors.filtering() ===");

        // Group by department, but only include people over 28
        var seniorByDept = people.stream()
            .collect(groupingBy(
                Person::department,
                filtering(p -> p.age() > 28, toList())
            ));
        System.out.println("Seniors by department: " + seniorByDept);

        // Collectors.flatMapping() (Java 9)
        System.out.println("\n=== Collectors.flatMapping() ===");

        var projects = Map.of(
            "Alice", List.of("ProjectA", "ProjectB"),
            "Bob", List.of("ProjectB", "ProjectC"),
            "Charlie", List.of("ProjectA")
        );

        // Group people by department and flatten their projects
        record PersonProjects(String name, String department, List<String> projects) {}

        var peopleWithProjects = people.stream()
            .filter(p -> projects.containsKey(p.name()))
            .map(p -> new PersonProjects(p.name(), p.department(), projects.get(p.name())))
            .toList();

        var projectsByDept = peopleWithProjects.stream()
            .collect(groupingBy(
                PersonProjects::department,
                flatMapping(pp -> pp.projects().stream(), toSet())
            ));
        System.out.println("Projects by department: " + projectsByDept);

        // Collectors.teeing() (Java 12)
        System.out.println("\n=== Collectors.teeing() ===");

        // Calculate min and max age in one pass
        record MinMax(int min, int max) {}

        var minMaxAge = people.stream()
            .collect(teeing(
                minBy(Comparator.comparingInt(Person::age)),
                maxBy(Comparator.comparingInt(Person::age)),
                (min, max) -> new MinMax(
                    min.map(Person::age).orElse(0),
                    max.map(Person::age).orElse(0)
                )
            ));
        System.out.println("Age range: " + minMaxAge);

        // Calculate sum and count in one pass
        record SumCount(long sum, long count) {
            double average() { return count == 0 ? 0 : (double) sum / count; }
        }

        var stats = people.stream()
            .collect(teeing(
                summingInt(Person::age),
                counting(),
                SumCount::new
            ));
        System.out.println("Age stats: sum=" + stats.sum() +
            ", count=" + stats.count() + ", avg=" + stats.average());
    }

    record Person(String name, int age, String department) {}
}
```

---

### Hands-On Exercise 6: More Stream Operations

Create `src/chapter07/MoreStreamOps.java`:

```java
package chapter07;

import java.util.*;
import java.util.stream.*;

public class MoreStreamOps {
    public static void main(String[] args) {
        // mapMulti() - Java 16: flexible flatMap alternative
        System.out.println("=== mapMulti() ===");

        var numbers = List.of(1, 2, 3, 4, 5);

        // Expand each number to itself and its square
        var expanded = numbers.stream()
            .<Integer>mapMulti((n, consumer) -> {
                consumer.accept(n);
                consumer.accept(n * n);
            })
            .toList();
        System.out.println("Original: " + numbers);
        System.out.println("Expanded (n, n^2): " + expanded);

        // Conditional expansion
        var filtered = numbers.stream()
            .<String>mapMulti((n, consumer) -> {
                if (n % 2 == 0) {
                    consumer.accept("even: " + n);
                }
                if (n > 3) {
                    consumer.accept("large: " + n);
                }
            })
            .toList();
        System.out.println("Conditional: " + filtered);

        // toList() - Java 16: shorthand for collect(Collectors.toList())
        System.out.println("\n=== toList() ===");
        var doubled = numbers.stream()
            .map(n -> n * 2)
            .toList();  // Cleaner than .collect(Collectors.toList())
        System.out.println("Doubled: " + doubled);

        // Note: toList() returns unmodifiable list!
        // doubled.add(100);  // UnsupportedOperationException

        // Gatherers (Java 22 Preview - showing concept)
        System.out.println("\n=== Stream processing patterns ===");

        // Sliding window (manual implementation)
        var data = List.of(1, 2, 3, 4, 5, 6, 7);
        var windows = slidingWindow(data, 3);
        System.out.println("Sliding windows of size 3:");
        windows.forEach(w -> System.out.println("  " + w));

        // Running total
        System.out.println("\nRunning totals:");
        var runningTotals = new ArrayList<Integer>();
        int total = 0;
        for (int n : data) {
            total += n;
            runningTotals.add(total);
        }
        System.out.println("Data: " + data);
        System.out.println("Totals: " + runningTotals);
    }

    // Helper: create sliding windows
    static <T> List<List<T>> slidingWindow(List<T> source, int size) {
        if (size > source.size()) return List.of();

        return IntStream.rangeClosed(0, source.size() - size)
            .mapToObj(start -> source.subList(start, start + size))
            .map(ArrayList::new)  // Create copies
            .collect(Collectors.toList());
    }
}
```

---

### Hands-On Exercise 7: Practical Collection Operations

Create `src/chapter07/PracticalOperations.java`:

```java
package chapter07;

import java.util.*;
import java.util.stream.*;

public class PracticalOperations {
    public static void main(String[] args) {
        // Map.of and computations
        System.out.println("=== Map Operations ===\n");

        var wordCounts = new HashMap<String, Integer>();
        var words = List.of("apple", "banana", "apple", "cherry", "banana", "apple");

        // Count words using compute
        for (String word : words) {
            wordCounts.compute(word, (k, v) -> v == null ? 1 : v + 1);
        }
        System.out.println("Word counts: " + wordCounts);

        // Or using merge (cleaner)
        var wordCounts2 = new HashMap<String, Integer>();
        for (String word : words) {
            wordCounts2.merge(word, 1, Integer::sum);
        }
        System.out.println("Word counts (merge): " + wordCounts2);

        // getOrDefault
        int appleCount = wordCounts.getOrDefault("apple", 0);
        int orangeCount = wordCounts.getOrDefault("orange", 0);
        System.out.println("Apple: " + appleCount + ", Orange: " + orangeCount);

        // putIfAbsent
        wordCounts.putIfAbsent("date", 5);
        wordCounts.putIfAbsent("apple", 100);  // Won't change - already exists
        System.out.println("After putIfAbsent: " + wordCounts);

        // computeIfAbsent - great for lazy initialization
        System.out.println("\n=== computeIfAbsent ===");
        var cache = new HashMap<String, List<String>>();

        // Old way:
        // if (!cache.containsKey("users")) {
        //     cache.put("users", new ArrayList<>());
        // }
        // cache.get("users").add("Alice");

        // New way:
        cache.computeIfAbsent("users", k -> new ArrayList<>()).add("Alice");
        cache.computeIfAbsent("users", k -> new ArrayList<>()).add("Bob");
        cache.computeIfAbsent("admins", k -> new ArrayList<>()).add("Root");

        System.out.println("Cache: " + cache);

        // forEach with BiConsumer
        System.out.println("\n=== Map.forEach ===");
        wordCounts.forEach((word, count) ->
            System.out.println(word + " -> " + "*".repeat(count)));

        // replaceAll
        System.out.println("\n=== replaceAll ===");
        var prices = new HashMap<>(Map.of("apple", 1.0, "banana", 0.5, "cherry", 2.0));
        System.out.println("Before discount: " + prices);

        prices.replaceAll((item, price) -> price * 0.9);  // 10% discount
        System.out.println("After 10% discount: " + prices);

        // List.replaceAll (different from Map!)
        var names = new ArrayList<>(List.of("alice", "bob", "charlie"));
        names.replaceAll(String::toUpperCase);
        System.out.println("\nUppercase names: " + names);

        // removeIf
        System.out.println("\n=== removeIf ===");
        var scores = new ArrayList<>(List.of(85, 92, 45, 78, 33, 95, 67));
        System.out.println("Before: " + scores);
        scores.removeIf(score -> score < 60);  // Remove failing scores
        System.out.println("After removing < 60: " + scores);
    }
}
```

---

## Summary: What's New by Version

| Version | Feature |
|---------|---------|
| Java 9 | `List.of()`, `Set.of()`, `Map.of()`, `Map.ofEntries()` |
| Java 9 | `Stream.takeWhile()`, `dropWhile()`, `ofNullable()` |
| Java 9 | `Stream.iterate()` with predicate |
| Java 9 | `Collectors.filtering()`, `flatMapping()` |
| Java 10 | `List.copyOf()`, `Set.copyOf()`, `Map.copyOf()` |
| Java 10 | `Collectors.toUnmodifiableList/Set/Map()` |
| Java 12 | `Collectors.teeing()` |
| Java 16 | `Stream.toList()` |
| Java 16 | `Stream.mapMulti()` |
| Java 21 | `SequencedCollection`, `SequencedSet`, `SequencedMap` |

---

## Practice Exercises

### Exercise 7.1: Word Frequency
Write a method that takes a list of sentences and returns a map of word frequencies (case-insensitive).

<details>
<summary>Click for solution</summary>

```java
static Map<String, Long> wordFrequency(List<String> sentences) {
    return sentences.stream()
        .flatMap(s -> Arrays.stream(s.toLowerCase().split("\\W+")))
        .filter(w -> !w.isEmpty())
        .collect(Collectors.groupingBy(
            w -> w,
            Collectors.counting()
        ));
}
```
</details>

### Exercise 7.2: Top N by Group
Given a list of products with categories and ratings, find the top 3 rated products in each category.

<details>
<summary>Click for solution</summary>

```java
record Product(String name, String category, double rating) {}

static Map<String, List<Product>> topNByCategory(List<Product> products, int n) {
    return products.stream()
        .collect(Collectors.groupingBy(
            Product::category,
            Collectors.collectingAndThen(
                Collectors.toList(),
                list -> list.stream()
                    .sorted(Comparator.comparingDouble(Product::rating).reversed())
                    .limit(n)
                    .toList()
            )
        ));
}
```
</details>

### Exercise 7.3: Partition and Summarize
Given a list of numbers, partition them into even and odd, and calculate the sum of each partition.

<details>
<summary>Click for solution</summary>

```java
static Map<Boolean, Integer> partitionAndSum(List<Integer> numbers) {
    return numbers.stream()
        .collect(Collectors.partitioningBy(
            n -> n % 2 == 0,
            Collectors.summingInt(Integer::intValue)
        ));
}
```
</details>

---

## Key Takeaways

1. **Factory methods** (`List.of`, `Set.of`, `Map.of`) create immutable collections
2. **No nulls allowed** in factory-created collections
3. **`copyOf()`** creates immutable copies of mutable collections
4. **`toList()`** (Java 16) is shorter than `collect(Collectors.toList())`
5. **`takeWhile`/`dropWhile`** enable prefix/suffix stream operations
6. **`teeing()`** combines two collectors into one result
7. **Sequenced collections** (Java 21) add first/last access methods
8. **Map methods** like `compute`, `merge`, `computeIfAbsent` simplify updates

---

## What's Next?

In [Chapter 8: Optional Enhancements](08-optional-enhancements.md), you'll learn about the new Optional methods that make null handling even more elegant.

---

[← Previous Chapter](06-switch-expressions.md) | [Back to Contents](../README.md) | [Next Chapter →](08-optional-enhancements.md)
