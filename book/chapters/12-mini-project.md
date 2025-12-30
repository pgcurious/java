# Chapter 12: Putting It All Together - Mini Project

> **Java Version:** 21 | **Difficulty:** Intermediate | **Time:** 60 minutes

In this final chapter, we'll build a complete application that demonstrates all the modern Java features you've learned. We'll create a **Weather Dashboard CLI** that fetches weather data from multiple cities concurrently.

## Project Overview

**Weather Dashboard** - A command-line application that:
- Fetches weather data from a public API
- Processes multiple cities concurrently using virtual threads
- Uses records for data modeling
- Uses sealed classes for response types
- Demonstrates pattern matching and switch expressions
- Uses text blocks for formatted output

---

## The Complete Application

Create `src/chapter12/WeatherDashboard.java`:

```java
package chapter12;

import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.util.*;
import java.util.concurrent.*;
import java.util.stream.*;

public class WeatherDashboard {

    // ============================================================
    // PART 1: Data Models using Records (Chapter 3)
    // ============================================================

    // Immutable data carriers for weather information
    record Coordinates(double latitude, double longitude) {}

    record Temperature(double celsius) {
        // Custom method to convert
        double fahrenheit() {
            return celsius * 9 / 5 + 32;
        }

        String formatted() {
            return String.format("%.1f°C (%.1f°F)", celsius, fahrenheit());
        }
    }

    record Wind(double speedKmh, String direction) {
        String formatted() {
            return String.format("%.1f km/h %s", speedKmh, direction);
        }
    }

    record WeatherCondition(String description, String icon) {}

    record WeatherData(
        String cityName,
        Coordinates coordinates,
        Temperature temperature,
        Temperature feelsLike,
        int humidity,
        Wind wind,
        WeatherCondition condition,
        Instant timestamp
    ) {
        // Compact constructor for validation (Chapter 3)
        public WeatherData {
            Objects.requireNonNull(cityName, "City name cannot be null");
            if (humidity < 0 || humidity > 100) {
                throw new IllegalArgumentException("Humidity must be 0-100");
            }
        }
    }

    // ============================================================
    // PART 2: Sealed Types for Results (Chapter 4)
    // ============================================================

    // Result type using sealed classes - handles success and failure
    sealed interface FetchResult permits FetchResult.Success, FetchResult.Failure {

        record Success(WeatherData data) implements FetchResult {}

        record Failure(String city, String error, Exception cause) implements FetchResult {
            Failure(String city, String error) {
                this(city, error, null);
            }
        }

        // Helper methods using pattern matching (Chapter 5)
        default boolean isSuccess() {
            return this instanceof Success;
        }

        default Optional<WeatherData> getData() {
            return switch (this) {
                case Success(var data) -> Optional.of(data);
                case Failure(_, _, _) -> Optional.empty();
            };
        }
    }

    // ============================================================
    // PART 3: HTTP Client for API Calls (Chapter 9)
    // ============================================================

    private final HttpClient httpClient;
    private static final String API_BASE = "https://wttr.in";

    public WeatherDashboard() {
        // Configured HTTP client
        this.httpClient = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_2)
            .connectTimeout(Duration.ofSeconds(10))
            .followRedirects(HttpClient.Redirect.NORMAL)
            .build();
    }

    // Fetch weather for a single city
    FetchResult fetchWeather(String city) {
        try {
            // Build request with text blocks (Chapter 2) for query format
            var url = "%s/%s?format=j1".formatted(API_BASE, city.replace(" ", "+"));

            var request = HttpRequest.newBuilder()
                .uri(URI.create(url))
                .header("Accept", "application/json")
                .GET()
                .build();

            var response = httpClient.send(request, BodyHandlers.ofString());

            if (response.statusCode() != 200) {
                return new FetchResult.Failure(city,
                    "HTTP " + response.statusCode());
            }

            // Parse the response (simplified parsing)
            var data = parseWeatherJson(city, response.body());
            return new FetchResult.Success(data);

        } catch (Exception e) {
            return new FetchResult.Failure(city, e.getMessage(), e);
        }
    }

    // Simplified JSON parsing (in real app, use a JSON library)
    private WeatherData parseWeatherJson(String city, String json) {
        // Extract values using simple string operations
        // (In production, use Jackson or Gson)

        var temp = extractDouble(json, "\"temp_C\"");
        var feelsLike = extractDouble(json, "\"FeelsLikeC\"");
        var humidity = (int) extractDouble(json, "\"humidity\"");
        var windSpeed = extractDouble(json, "\"windspeedKmph\"");
        var windDir = extractString(json, "\"winddir16Point\"");
        var description = extractString(json, "\"weatherDesc\"");
        var lat = extractDouble(json, "\"latitude\"");
        var lon = extractDouble(json, "\"longitude\"");

        return new WeatherData(
            city,
            new Coordinates(lat, lon),
            new Temperature(temp),
            new Temperature(feelsLike),
            humidity,
            new Wind(windSpeed, windDir != null ? windDir : "N/A"),
            new WeatherCondition(description != null ? description : "Unknown", ""),
            Instant.now()
        );
    }

    private double extractDouble(String json, String key) {
        try {
            int idx = json.indexOf(key);
            if (idx == -1) return 0.0;
            int start = json.indexOf(":", idx) + 1;
            int end = start;
            while (end < json.length() &&
                   (Character.isDigit(json.charAt(end)) ||
                    json.charAt(end) == '.' ||
                    json.charAt(end) == '-' ||
                    json.charAt(end) == '"' ||
                    json.charAt(end) == ' ')) {
                end++;
            }
            String value = json.substring(start, end).trim().replace("\"", "");
            return value.isEmpty() ? 0.0 : Double.parseDouble(value);
        } catch (Exception e) {
            return 0.0;
        }
    }

    private String extractString(String json, String key) {
        try {
            int idx = json.indexOf(key);
            if (idx == -1) return null;
            // Find the value after the key
            int colonIdx = json.indexOf(":", idx);
            if (colonIdx == -1) return null;

            // Look for quoted value
            int startQuote = json.indexOf("\"", colonIdx + 1);
            if (startQuote == -1) return null;
            int endQuote = json.indexOf("\"", startQuote + 1);
            if (endQuote == -1) return null;

            return json.substring(startQuote + 1, endQuote);
        } catch (Exception e) {
            return null;
        }
    }

    // ============================================================
    // PART 4: Concurrent Fetching with Virtual Threads (Chapter 10)
    // ============================================================

    List<FetchResult> fetchAllConcurrently(List<String> cities) {
        // Using virtual threads for concurrent HTTP requests
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {

            // Submit all fetch tasks
            List<Future<FetchResult>> futures = cities.stream()
                .map(city -> executor.submit(() -> fetchWeather(city)))
                .toList();

            // Collect results
            return futures.stream()
                .map(future -> {
                    try {
                        return future.get(30, TimeUnit.SECONDS);
                    } catch (Exception e) {
                        return new FetchResult.Failure("Unknown", e.getMessage(), e);
                    }
                })
                .toList();
        }
    }

    // ============================================================
    // PART 5: Display using Pattern Matching & Text Blocks
    // ============================================================

    void displayResults(List<FetchResult> results) {
        // Header using text block (Chapter 2)
        System.out.println("""

            ╔═══════════════════════════════════════════════════════════════╗
            ║                    🌤️  WEATHER DASHBOARD  🌤️                   ║
            ╠═══════════════════════════════════════════════════════════════╣
            """);

        // Process each result using pattern matching (Chapters 5 & 6)
        for (FetchResult result : results) {
            displayResult(result);
        }

        // Summary
        displaySummary(results);
    }

    void displayResult(FetchResult result) {
        // Switch expression with pattern matching (Chapter 6)
        String output = switch (result) {
            case FetchResult.Success(var data) -> formatWeatherData(data);
            case FetchResult.Failure(var city, var error, _) -> formatError(city, error);
        };
        System.out.println(output);
    }

    String formatWeatherData(WeatherData data) {
        // Using text block with formatted() (Chapter 2)
        return """
            ║  📍 %s
            ║     🌡️  Temperature: %s
            ║     🤔 Feels like:  %s
            ║     💧 Humidity:    %d%%
            ║     💨 Wind:        %s
            ║     ☁️  Condition:   %s
            ╟───────────────────────────────────────────────────────────────╢
            """.formatted(
                padRight(data.cityName(), 20),
                data.temperature().formatted(),
                data.feelsLike().formatted(),
                data.humidity(),
                data.wind().formatted(),
                data.condition().description()
            );
    }

    String formatError(String city, String error) {
        return """
            ║  ❌ %s
            ║     Error: %s
            ╟───────────────────────────────────────────────────────────────╢
            """.formatted(padRight(city, 20), error);
    }

    void displaySummary(List<FetchResult> results) {
        // Using stream operations with Optional (Chapters 7 & 8)
        var successfulResults = results.stream()
            .flatMap(r -> r.getData().stream())  // Optional.stream() (Chapter 8)
            .toList();

        var failedCount = results.size() - successfulResults.size();

        // Calculate statistics using streams (Chapter 7)
        var avgTemp = successfulResults.stream()
            .mapToDouble(d -> d.temperature().celsius())
            .average()
            .orElse(0);

        var hottestCity = successfulResults.stream()
            .max(Comparator.comparingDouble(d -> d.temperature().celsius()))
            .map(WeatherData::cityName)
            .orElse("N/A");

        var coldestCity = successfulResults.stream()
            .min(Comparator.comparingDouble(d -> d.temperature().celsius()))
            .map(WeatherData::cityName)
            .orElse("N/A");

        // Summary text block
        System.out.println("""
            ║                         📊 SUMMARY                            ║
            ╟───────────────────────────────────────────────────────────────╢
            ║  ✅ Successful: %d    ❌ Failed: %d
            ║  🌡️  Average Temperature: %.1f°C
            ║  🔥 Hottest: %s
            ║  ❄️  Coldest: %s
            ║  📅 Generated: %s
            ╚═══════════════════════════════════════════════════════════════╝
            """.formatted(
                successfulResults.size(),
                failedCount,
                avgTemp,
                hottestCity,
                coldestCity,
                LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"))
            ));
    }

    // Helper method
    private String padRight(String s, int length) {
        return String.format("%-" + length + "s", s);
    }

    // ============================================================
    // PART 6: Main Application Entry Point
    // ============================================================

    public static void main(String[] args) {
        // Default cities or use command line arguments
        List<String> cities = args.length > 0
            ? List.of(args)
            : List.of("London", "Tokyo", "New York", "Sydney", "Paris",
                      "Berlin", "Singapore", "Dubai", "Mumbai", "Toronto");

        System.out.println("Fetching weather data for " + cities.size() + " cities...");

        var dashboard = new WeatherDashboard();

        // Measure time using Instant (Chapter 11)
        var start = Instant.now();

        // Fetch all weather data concurrently
        var results = dashboard.fetchAllConcurrently(cities);

        var duration = Duration.between(start, Instant.now());

        // Display results
        dashboard.displayResults(results);

        System.out.printf("%nCompleted in %d ms using virtual threads!%n",
            duration.toMillis());
    }
}
```

---

## Running the Application

```bash
# Run with default cities
java src/chapter12/WeatherDashboard.java

# Run with custom cities
java src/chapter12/WeatherDashboard.java London Paris Tokyo
```

---

## Features Demonstrated

| Feature | Chapter | Where Used |
|---------|---------|------------|
| `var` | 1 | Throughout for local variables |
| Text Blocks | 2 | Output formatting, API URL |
| Records | 3 | All data models |
| Sealed Classes | 4 | `FetchResult` type |
| Pattern Matching | 5 | Result handling |
| Switch Expressions | 6 | `displayResult()` method |
| Collections/Streams | 7 | Statistics calculation |
| Optional | 8 | Safe data extraction |
| HTTP Client | 9 | API calls |
| Virtual Threads | 10 | Concurrent fetching |
| Misc Improvements | 11 | `Duration`, `Instant` |

---

## Extension Challenges

### Challenge 1: Add Caching
Add a simple in-memory cache to avoid repeated API calls for the same city.

<details>
<summary>Hint</summary>

```java
private final Map<String, CacheEntry> cache = new ConcurrentHashMap<>();

record CacheEntry(WeatherData data, Instant expiry) {
    boolean isValid() {
        return Instant.now().isBefore(expiry);
    }
}

FetchResult fetchWeather(String city) {
    var cached = cache.get(city.toLowerCase());
    if (cached != null && cached.isValid()) {
        return new FetchResult.Success(cached.data());
    }
    // ... fetch from API
}
```
</details>

### Challenge 2: Add Weather Alerts
Create a sealed hierarchy for weather alerts based on conditions.

<details>
<summary>Hint</summary>

```java
sealed interface WeatherAlert permits HeatWarning, ColdWarning, StormWarning, NoAlert {
    String message();
}

record HeatWarning(double temperature) implements WeatherAlert {
    public String message() {
        return "⚠️ Heat warning! Temperature is %.1f°C".formatted(temperature);
    }
}

// Generate alerts based on weather data
WeatherAlert checkForAlerts(WeatherData data) {
    return switch (data) {
        case WeatherData d when d.temperature().celsius() > 35 ->
            new HeatWarning(d.temperature().celsius());
        case WeatherData d when d.temperature().celsius() < -10 ->
            new ColdWarning(d.temperature().celsius());
        // ... more conditions
        default -> new NoAlert();
    };
}
```
</details>

### Challenge 3: Save to File
Add functionality to save the results to a JSON file.

<details>
<summary>Hint</summary>

```java
void saveToFile(List<FetchResult> results, Path outputPath) throws IOException {
    var json = results.stream()
        .flatMap(r -> r.getData().stream())
        .map(this::toJson)
        .collect(Collectors.joining(",\n  ", "[\n  ", "\n]"));

    Files.writeString(outputPath, json);
}

String toJson(WeatherData data) {
    return """
        {
          "city": "%s",
          "temperature": %.1f,
          "humidity": %d
        }""".formatted(data.cityName(), data.temperature().celsius(), data.humidity());
}
```
</details>

### Challenge 4: Add Unit Tests
Create tests using records for test data.

<details>
<summary>Hint</summary>

```java
// Test data using records
record TestCase(String input, String expectedOutput) {}

List<TestCase> testCases = List.of(
    new TestCase("London", "London"),
    new TestCase("New York", "New York"),
    new TestCase("", "Error")
);

void runTests() {
    for (var test : testCases) {
        var result = dashboard.fetchWeather(test.input());
        // Assert expected behavior
    }
}
```
</details>

---

## Key Learnings

1. **Modern Java is expressive** - Records, sealed classes, and pattern matching make domain modeling cleaner
2. **Virtual threads simplify concurrency** - No thread pools to tune, just spawn threads
3. **Text blocks improve readability** - Multi-line strings are now pleasant to work with
4. **Switch expressions are powerful** - Combined with patterns, they handle complex logic elegantly
5. **The HTTP Client is production-ready** - No need for third-party libraries for basic HTTP

---

## Congratulations!

You've completed the Modern Java hands-on guide! You now have practical experience with:

- ✅ `var` for type inference
- ✅ Text blocks for multi-line strings
- ✅ Records for data classes
- ✅ Sealed classes for restricted hierarchies
- ✅ Pattern matching for type-safe checks
- ✅ Switch expressions for cleaner control flow
- ✅ Enhanced collections and streams
- ✅ Optional improvements
- ✅ The HTTP Client API
- ✅ Virtual threads for scalable concurrency

**Keep practicing, and enjoy writing modern Java!**

---

## Further Reading

- [Java Language Updates](https://docs.oracle.com/en/java/javase/21/language/java-language-changes.html)
- [JEP Index](https://openjdk.org/jeps/0) - All Java Enhancement Proposals
- [Inside Java](https://inside.java/) - Official Java blog
- [Dev.java](https://dev.java/) - Official Java developer portal

---

[← Previous Chapter](11-misc-improvements.md) | [Back to Contents](../README.md)
