# Chapter 10: Virtual Threads

> **Java Version:** 21+ | **Difficulty:** Intermediate-Advanced | **Time:** 50 minutes

## The Problem

Traditional Java threads (platform threads) are expensive:
- Each thread maps 1:1 to an OS thread
- Default stack size: ~1MB per thread
- Creating thousands of threads exhausts memory
- Context switching is costly

```java
// This can crash your JVM - don't run!
for (int i = 0; i < 1_000_000; i++) {
    new Thread(() -> {
        try { Thread.sleep(10000); }
        catch (InterruptedException e) { }
    }).start();
}
// OutOfMemoryError: unable to create native thread
```

As a result, we use thread pools to limit concurrency:

```java
var executor = Executors.newFixedThreadPool(200);  // Only 200 threads!
```

But 200 threads can only handle 200 concurrent blocking operations.

## The Solution: Virtual Threads

Java 21 introduced **virtual threads** - lightweight threads managed by the JVM:

- **Cheap to create**: Millions of virtual threads are possible
- **Small footprint**: ~1KB vs ~1MB for platform threads
- **Automatic scheduling**: JVM handles mounting/unmounting to carrier threads
- **Same API**: Use existing Thread API

```java
// Create a million virtual threads - no problem!
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1_000_000; i++) {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            return "Done";
        });
    }
}
```

---

## Hands-On Exercise 1: Creating Virtual Threads

Create `src/chapter10/VirtualThreadBasics.java`:

```java
package chapter10;

import java.time.Duration;
import java.time.Instant;

public class VirtualThreadBasics {
    public static void main(String[] args) throws Exception {
        // Method 1: Thread.startVirtualThread()
        System.out.println("=== Starting Virtual Threads ===\n");

        Thread vt1 = Thread.startVirtualThread(() -> {
            System.out.println("Virtual thread 1 running on: " + Thread.currentThread());
        });
        vt1.join();

        // Method 2: Thread.ofVirtual()
        Thread vt2 = Thread.ofVirtual()
            .name("my-virtual-thread")
            .start(() -> {
                System.out.println("Virtual thread 2: " + Thread.currentThread().getName());
            });
        vt2.join();

        // Method 3: Unstarted thread
        Thread vt3 = Thread.ofVirtual()
            .name("lazy-virtual-thread")
            .unstarted(() -> {
                System.out.println("Virtual thread 3 started later");
            });
        // Start when ready
        vt3.start();
        vt3.join();

        // Comparing with platform thread
        System.out.println("\n=== Virtual vs Platform ===");

        Thread platform = Thread.ofPlatform()
            .name("platform-thread")
            .start(() -> {
                System.out.println("Platform thread: " + Thread.currentThread());
                System.out.println("Is virtual: " + Thread.currentThread().isVirtual());
            });
        platform.join();

        Thread virtual = Thread.ofVirtual()
            .name("virtual-thread")
            .start(() -> {
                System.out.println("Virtual thread: " + Thread.currentThread());
                System.out.println("Is virtual: " + Thread.currentThread().isVirtual());
            });
        virtual.join();

        // Show the scale difference
        System.out.println("\n=== Scale Test ===");
        testScale();
    }

    static void testScale() throws Exception {
        int numThreads = 100_000;

        Instant start = Instant.now();

        Thread[] threads = new Thread[numThreads];
        for (int i = 0; i < numThreads; i++) {
            threads[i] = Thread.startVirtualThread(() -> {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        // Wait for all to complete
        for (Thread t : threads) {
            t.join();
        }

        Duration duration = Duration.between(start, Instant.now());
        System.out.printf("Created and ran %,d virtual threads in %s%n",
            numThreads, duration);
        System.out.printf("That's %.0f threads/second%n",
            numThreads / (duration.toMillis() / 1000.0));
    }
}
```

**Run it:**
```bash
java src/chapter10/VirtualThreadBasics.java
```

---

## Hands-On Exercise 2: ExecutorService with Virtual Threads

Create `src/chapter10/VirtualThreadExecutor.java`:

```java
package chapter10;

import java.time.Duration;
import java.time.Instant;
import java.util.concurrent.*;
import java.util.stream.IntStream;

public class VirtualThreadExecutor {
    public static void main(String[] args) throws Exception {
        // newVirtualThreadPerTaskExecutor - creates a new virtual thread per task
        System.out.println("=== Virtual Thread Per Task Executor ===\n");

        // This is the recommended way to use virtual threads
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            // Submit tasks - each gets its own virtual thread
            var futures = IntStream.range(0, 10)
                .mapToObj(i -> executor.submit(() -> {
                    Thread.sleep(100);
                    return "Task " + i + " completed on " +
                        Thread.currentThread().getName();
                }))
                .toList();

            // Get results
            for (var future : futures) {
                System.out.println(future.get());
            }
        }

        // Compare with fixed thread pool
        System.out.println("\n=== Comparison: Fixed Pool vs Virtual ===");

        int numTasks = 10_000;
        Duration blockingTime = Duration.ofMillis(100);

        // Fixed thread pool (limited concurrency)
        System.out.println("\nFixed thread pool (200 threads)...");
        Instant start = Instant.now();
        try (var executor = Executors.newFixedThreadPool(200)) {
            var futures = IntStream.range(0, numTasks)
                .mapToObj(i -> executor.submit(() -> {
                    Thread.sleep(blockingTime);
                    return i;
                }))
                .toList();

            for (var f : futures) {
                f.get();
            }
        }
        System.out.printf("Time: %s%n", Duration.between(start, Instant.now()));

        // Virtual threads (unlimited concurrency)
        System.out.println("\nVirtual threads...");
        start = Instant.now();
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            var futures = IntStream.range(0, numTasks)
                .mapToObj(i -> executor.submit(() -> {
                    Thread.sleep(blockingTime);
                    return i;
                }))
                .toList();

            for (var f : futures) {
                f.get();
            }
        }
        System.out.printf("Time: %s%n", Duration.between(start, Instant.now()));
    }
}
```

---

## Hands-On Exercise 3: How Virtual Threads Work

Create `src/chapter10/VirtualThreadInternals.java`:

```java
package chapter10;

import java.util.concurrent.*;

public class VirtualThreadInternals {
    public static void main(String[] args) throws Exception {
        System.out.println("=== Virtual Thread Internals ===\n");

        // Virtual threads are "mounted" on carrier (platform) threads
        System.out.println("Available processors: " +
            Runtime.getRuntime().availableProcessors());

        // Demo: See carrier thread changes during blocking
        System.out.println("\n=== Carrier Thread Switching ===");

        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            var futures = new java.util.ArrayList<Future<String>>();

            for (int i = 0; i < 5; i++) {
                int taskNum = i;
                futures.add(executor.submit(() -> {
                    StringBuilder sb = new StringBuilder();
                    sb.append("Task ").append(taskNum).append(":\n");

                    // Before blocking
                    sb.append("  Before sleep: ").append(carrierInfo()).append("\n");

                    // Blocking operation - virtual thread yields
                    Thread.sleep(100);

                    // After blocking - might be on different carrier
                    sb.append("  After sleep:  ").append(carrierInfo()).append("\n");

                    return sb.toString();
                }));
            }

            for (var f : futures) {
                System.out.println(f.get());
            }
        }

        // Demo: Pinning (when virtual thread can't yield)
        System.out.println("=== Pinning Demo ===");
        System.out.println("Virtual threads get 'pinned' when:");
        System.out.println("  - Inside synchronized block/method");
        System.out.println("  - During native method call");
        System.out.println("\nUsing ReentrantLock instead of synchronized allows yielding.\n");

        // Good: ReentrantLock
        var lock = new java.util.concurrent.locks.ReentrantLock();
        Thread.startVirtualThread(() -> {
            lock.lock();
            try {
                System.out.println("ReentrantLock - thread can yield while waiting");
            } finally {
                lock.unlock();
            }
        }).join();

        // Be careful: synchronized can cause pinning
        Object monitor = new Object();
        Thread.startVirtualThread(() -> {
            synchronized (monitor) {
                // While in synchronized, the virtual thread is "pinned"
                // It cannot yield even during blocking operations
                System.out.println("Synchronized - thread is pinned");
            }
        }).join();
    }

    static String carrierInfo() {
        // Get info about underlying platform thread
        Thread current = Thread.currentThread();
        return String.format("VT[%s] on carrier", current.threadId());
    }
}
```

---

## Hands-On Exercise 4: Practical Example - Concurrent HTTP Requests

Create `src/chapter10/ConcurrentHttpClient.java`:

```java
package chapter10;

import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;
import java.time.*;
import java.util.*;
import java.util.concurrent.*;

public class ConcurrentHttpClient {
    private static final HttpClient httpClient = HttpClient.newBuilder()
        .connectTimeout(Duration.ofSeconds(10))
        .build();

    public static void main(String[] args) throws Exception {
        // Simulate fetching data from multiple URLs
        List<String> urls = List.of(
            "https://httpbin.org/delay/1",  // 1 second delay
            "https://httpbin.org/delay/1",
            "https://httpbin.org/delay/1",
            "https://httpbin.org/delay/1",
            "https://httpbin.org/delay/1",
            "https://httpbin.org/delay/1",
            "https://httpbin.org/delay/1",
            "https://httpbin.org/delay/1",
            "https://httpbin.org/delay/1",
            "https://httpbin.org/delay/1"
        );

        System.out.println("=== Concurrent HTTP Requests ===\n");
        System.out.println("Fetching " + urls.size() + " URLs (each with 1s delay)...\n");

        // Sequential - one at a time
        System.out.println("Sequential approach...");
        Instant start = Instant.now();
        for (String url : urls) {
            fetchUrl(url);
        }
        System.out.printf("Sequential time: %s%n%n", Duration.between(start, Instant.now()));

        // Concurrent with virtual threads
        System.out.println("Virtual threads approach...");
        start = Instant.now();

        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            List<Future<FetchResult>> futures = urls.stream()
                .map(url -> executor.submit(() -> fetchUrl(url)))
                .toList();

            for (Future<FetchResult> future : futures) {
                FetchResult result = future.get();
                System.out.printf("  %s -> %d (%dms)%n",
                    result.url(), result.statusCode(), result.durationMs());
            }
        }
        System.out.printf("Concurrent time: %s%n", Duration.between(start, Instant.now()));

        // Large scale demo
        System.out.println("\n=== Large Scale Demo ===");
        largeScaleDemo();
    }

    record FetchResult(String url, int statusCode, long durationMs) {}

    static FetchResult fetchUrl(String url) throws Exception {
        Instant start = Instant.now();

        var request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .GET()
            .build();

        var response = httpClient.send(request, BodyHandlers.discarding());

        long duration = Duration.between(start, Instant.now()).toMillis();
        return new FetchResult(url, response.statusCode(), duration);
    }

    static void largeScaleDemo() throws Exception {
        // Simulate handling many concurrent requests
        int numRequests = 1000;

        System.out.printf("Simulating %,d concurrent 'requests'...%n", numRequests);

        Instant start = Instant.now();

        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            List<Future<?>> futures = new ArrayList<>();

            for (int i = 0; i < numRequests; i++) {
                int requestId = i;
                futures.add(executor.submit(() -> {
                    // Simulate I/O bound work (database query, API call, etc.)
                    Thread.sleep(Duration.ofMillis(100 + (requestId % 50)));
                    return "Request " + requestId + " completed";
                }));
            }

            // Wait for all
            for (var f : futures) {
                f.get();
            }
        }

        Duration duration = Duration.between(start, Instant.now());
        System.out.printf("Completed %,d requests in %s%n", numRequests, duration);
        System.out.printf("Throughput: %.0f requests/second%n",
            numRequests / (duration.toMillis() / 1000.0));
    }
}
```

---

## Hands-On Exercise 5: Structured Concurrency (Preview)

Create `src/chapter10/StructuredConcurrency.java`:

```java
package chapter10;

import java.time.Duration;
import java.util.concurrent.*;

public class StructuredConcurrency {
    public static void main(String[] args) throws Exception {
        System.out.println("=== Structured Concurrency Patterns ===\n");

        // Pattern 1: Wait for all tasks
        System.out.println("Pattern 1: Wait for all tasks");
        waitForAllExample();

        // Pattern 2: First successful result
        System.out.println("\nPattern 2: First successful result");
        firstSuccessExample();

        // Pattern 3: Collect all results
        System.out.println("\nPattern 3: Collect all results");
        collectResultsExample();

        // Note: Java 21 has StructuredTaskScope in preview
        // The examples below show the patterns without using preview features
    }

    static void waitForAllExample() throws Exception {
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            var future1 = executor.submit(() -> {
                Thread.sleep(100);
                System.out.println("  Task 1 completed");
                return "Result 1";
            });

            var future2 = executor.submit(() -> {
                Thread.sleep(150);
                System.out.println("  Task 2 completed");
                return "Result 2";
            });

            var future3 = executor.submit(() -> {
                Thread.sleep(50);
                System.out.println("  Task 3 completed");
                return "Result 3";
            });

            // Wait for all
            String r1 = future1.get();
            String r2 = future2.get();
            String r3 = future3.get();

            System.out.println("  All results: " + r1 + ", " + r2 + ", " + r3);
        }
    }

    static void firstSuccessExample() throws Exception {
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            var completionService = new ExecutorCompletionService<String>(executor);

            completionService.submit(() -> {
                Thread.sleep(200);
                return "Slow server response";
            });

            completionService.submit(() -> {
                Thread.sleep(50);
                return "Fast server response";
            });

            completionService.submit(() -> {
                Thread.sleep(100);
                return "Medium server response";
            });

            // Get the first completed result
            String firstResult = completionService.take().get();
            System.out.println("  First result: " + firstResult);
        }
    }

    static void collectResultsExample() throws Exception {
        record UserData(String name, int age) {}
        record OrderData(String orderId, double total) {}
        record CombinedData(UserData user, OrderData order) {}

        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            // Fetch user and order data concurrently
            Future<UserData> userFuture = executor.submit(() -> {
                Thread.sleep(100);  // Simulate DB call
                return new UserData("Alice", 30);
            });

            Future<OrderData> orderFuture = executor.submit(() -> {
                Thread.sleep(150);  // Simulate API call
                return new OrderData("ORD-123", 99.99);
            });

            // Combine results
            UserData user = userFuture.get();
            OrderData order = orderFuture.get();
            CombinedData combined = new CombinedData(user, order);

            System.out.println("  Combined data: " + combined);
        }
    }
}
```

---

## When to Use Virtual Threads

### Good Use Cases

```java
// I/O bound operations - EXCELLENT
executor.submit(() -> {
    String data = httpClient.send(request, handler).body();
    String result = database.query(sql);
    fileSystem.write(path, content);
});

// High-concurrency servers - EXCELLENT
ServerSocket server = new ServerSocket(8080);
while (true) {
    Socket socket = server.accept();
    Thread.startVirtualThread(() -> handleConnection(socket));
}

// Parallel data fetching - EXCELLENT
List<Future<Data>> futures = urls.stream()
    .map(url -> executor.submit(() -> fetch(url)))
    .toList();
```

### Not Ideal Use Cases

```java
// CPU-bound work - use platform thread pools
// Virtual threads don't help with pure computation
executor.submit(() -> {
    return fibonacci(1000000);  // CPU intensive
});

// Using synchronized heavily - causes pinning
synchronized (lock) {
    Thread.sleep(1000);  // Pinned! Use ReentrantLock instead
}

// Long-running computations without blocking
Thread.startVirtualThread(() -> {
    while (true) {
        computeNextPrime();  // Never blocks, never yields
    }
});
```

---

## Migration Guide: Platform to Virtual

```java
// Before: Fixed thread pool
ExecutorService executor = Executors.newFixedThreadPool(200);

// After: Virtual thread per task
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

// Before: Cached thread pool with limits
ExecutorService executor = new ThreadPoolExecutor(
    10, 200, 60L, TimeUnit.SECONDS, new LinkedBlockingQueue<>());

// After: Just use virtual threads
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

// Before: synchronized for thread safety
synchronized (this) {
    // critical section
}

// After: Use ReentrantLock for virtual thread compatibility
private final ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

---

## Summary: Virtual Threads

| Aspect | Platform Threads | Virtual Threads |
|--------|------------------|-----------------|
| Cost | ~1MB stack | ~1KB |
| Count | Thousands | Millions |
| Managed by | OS | JVM |
| Blocking | Expensive | Cheap (yields) |
| CPU-bound work | Good | No benefit |
| I/O-bound work | Limited | Excellent |

### Key APIs

```java
// Create virtual thread
Thread.startVirtualThread(runnable)
Thread.ofVirtual().start(runnable)

// Check if virtual
thread.isVirtual()

// ExecutorService
Executors.newVirtualThreadPerTaskExecutor()

// Thread factory
Thread.ofVirtual().factory()
```

---

## Practice Exercises

### Exercise 10.1: Concurrent File Processing
Process 100 files concurrently, each taking 100ms to "process". Compare virtual threads vs fixed pool.

<details>
<summary>Click for solution</summary>

```java
void processFiles() throws Exception {
    int numFiles = 100;

    // Virtual threads
    Instant start = Instant.now();
    try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
        var futures = IntStream.range(0, numFiles)
            .mapToObj(i -> executor.submit(() -> {
                Thread.sleep(100);  // Simulate file processing
                return "file-" + i;
            }))
            .toList();
        for (var f : futures) f.get();
    }
    System.out.println("Virtual: " + Duration.between(start, Instant.now()));

    // Fixed pool
    start = Instant.now();
    try (var executor = Executors.newFixedThreadPool(10)) {
        var futures = IntStream.range(0, numFiles)
            .mapToObj(i -> executor.submit(() -> {
                Thread.sleep(100);
                return "file-" + i;
            }))
            .toList();
        for (var f : futures) f.get();
    }
    System.out.println("Fixed(10): " + Duration.between(start, Instant.now()));
}
```
</details>

### Exercise 10.2: Web Scraper
Create a simple web scraper that fetches multiple URLs concurrently using virtual threads.

<details>
<summary>Click for solution</summary>

```java
record ScrapedPage(String url, int length, int status) {}

List<ScrapedPage> scrapeUrls(List<String> urls) throws Exception {
    var client = HttpClient.newHttpClient();

    try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
        var futures = urls.stream()
            .map(url -> executor.submit(() -> {
                var response = client.send(
                    HttpRequest.newBuilder().uri(URI.create(url)).build(),
                    BodyHandlers.ofString()
                );
                return new ScrapedPage(url, response.body().length(), response.statusCode());
            }))
            .toList();

        return futures.stream()
            .map(f -> {
                try { return f.get(); }
                catch (Exception e) { return null; }
            })
            .filter(Objects::nonNull)
            .toList();
    }
}
```
</details>

---

## Key Takeaways

1. **Virtual threads are lightweight** - create millions without worry
2. **Best for I/O-bound tasks** - blocking is cheap
3. **Use `newVirtualThreadPerTaskExecutor()`** - one thread per task
4. **Avoid `synchronized` for long operations** - causes pinning
5. **Use `ReentrantLock` instead** - allows yielding
6. **Same Thread API** - easy migration
7. **Not for CPU-bound work** - use platform threads for computation

---

## What's Next?

In [Chapter 11: Helpful NullPointerExceptions & More](11-misc-improvements.md), you'll discover various quality-of-life improvements in modern Java.

---

[← Previous Chapter](09-http-client.md) | [Back to Contents](../README.md) | [Next Chapter →](11-misc-improvements.md)
