# Chapter 9: HTTP Client API

> **Java Version:** 11+ | **Difficulty:** Intermediate | **Time:** 40 minutes

## The Problem

Before Java 11, making HTTP requests required either the old, clunky `HttpURLConnection` or third-party libraries:

```java
// Old way with HttpURLConnection - verbose and error-prone
URL url = new URL("https://api.example.com/users");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
conn.setRequestProperty("Accept", "application/json");

int status = conn.getResponseCode();
BufferedReader reader = new BufferedReader(
    new InputStreamReader(conn.getInputStream()));
String line;
StringBuilder response = new StringBuilder();
while ((line = reader.readLine()) != null) {
    response.append(line);
}
reader.close();
```

## The Solution: Java HTTP Client

Java 11 introduced a modern, fluent HTTP client that supports:
- HTTP/1.1 and HTTP/2
- Synchronous and asynchronous requests
- WebSocket support
- Automatic redirect handling
- Request/response body handlers

```java
// New way - clean and modern
var client = HttpClient.newHttpClient();
var request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .GET()
    .build();

var response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());
```

---

## Hands-On Exercise 1: Basic GET Request

Create `src/chapter09/BasicHttpClient.java`:

```java
package chapter09;

import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;

public class BasicHttpClient {
    public static void main(String[] args) throws Exception {
        // Create HTTP client (reusable)
        var client = HttpClient.newHttpClient();

        // Build a GET request
        var request = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/get"))
            .header("Accept", "application/json")
            .GET()  // Optional - GET is the default
            .build();

        System.out.println("=== Sending GET Request ===");
        System.out.println("URL: " + request.uri());
        System.out.println("Method: " + request.method());

        // Send synchronously and get response as String
        var response = client.send(request, BodyHandlers.ofString());

        System.out.println("\n=== Response ===");
        System.out.println("Status: " + response.statusCode());
        System.out.println("Headers: " + response.headers().map());
        System.out.println("\nBody (first 500 chars):");
        String body = response.body();
        System.out.println(body.substring(0, Math.min(500, body.length())));

        // Check if successful
        if (response.statusCode() >= 200 && response.statusCode() < 300) {
            System.out.println("\n✓ Request successful!");
        }
    }
}
```

**Run it:**
```bash
java src/chapter09/BasicHttpClient.java
```

---

## Hands-On Exercise 2: Different Request Types

Create `src/chapter09/HttpMethods.java`:

```java
package chapter09;

import java.net.URI;
import java.net.http.*;
import java.net.http.HttpRequest.BodyPublishers;
import java.net.http.HttpResponse.BodyHandlers;

public class HttpMethods {
    public static void main(String[] args) throws Exception {
        var client = HttpClient.newHttpClient();
        var baseUrl = "https://httpbin.org";

        // GET request
        System.out.println("=== GET Request ===");
        var getRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/get?name=Java&version=21"))
            .GET()
            .build();

        var getResponse = client.send(getRequest, BodyHandlers.ofString());
        System.out.println("GET Status: " + getResponse.statusCode());

        // POST request with JSON body
        System.out.println("\n=== POST Request ===");
        String jsonBody = """
            {
                "name": "John Doe",
                "email": "john@example.com",
                "age": 30
            }
            """;

        var postRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/post"))
            .header("Content-Type", "application/json")
            .POST(BodyPublishers.ofString(jsonBody))
            .build();

        var postResponse = client.send(postRequest, BodyHandlers.ofString());
        System.out.println("POST Status: " + postResponse.statusCode());
        System.out.println("Response body:\n" + postResponse.body().substring(0, 300));

        // PUT request
        System.out.println("\n=== PUT Request ===");
        var putRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/put"))
            .header("Content-Type", "application/json")
            .PUT(BodyPublishers.ofString("""
                {"id": 1, "name": "Updated Name"}
                """))
            .build();

        var putResponse = client.send(putRequest, BodyHandlers.ofString());
        System.out.println("PUT Status: " + putResponse.statusCode());

        // DELETE request
        System.out.println("\n=== DELETE Request ===");
        var deleteRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/delete"))
            .DELETE()
            .build();

        var deleteResponse = client.send(deleteRequest, BodyHandlers.ofString());
        System.out.println("DELETE Status: " + deleteResponse.statusCode());

        // Custom method (PATCH)
        System.out.println("\n=== PATCH Request ===");
        var patchRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/patch"))
            .method("PATCH", BodyPublishers.ofString("""
                {"field": "value"}
                """))
            .header("Content-Type", "application/json")
            .build();

        var patchResponse = client.send(patchRequest, BodyHandlers.ofString());
        System.out.println("PATCH Status: " + patchResponse.statusCode());
    }
}
```

---

## Hands-On Exercise 3: Async Requests

Create `src/chapter09/AsyncHttpClient.java`:

```java
package chapter09;

import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;
import java.util.List;
import java.util.concurrent.CompletableFuture;

public class AsyncHttpClient {
    public static void main(String[] args) throws Exception {
        var client = HttpClient.newHttpClient();

        // Single async request
        System.out.println("=== Single Async Request ===");

        var request = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/delay/1"))  // 1 second delay
            .build();

        System.out.println("Sending async request...");
        long start = System.currentTimeMillis();

        CompletableFuture<HttpResponse<String>> future = client
            .sendAsync(request, BodyHandlers.ofString());

        System.out.println("Request sent, doing other work...");

        // Do other work while waiting
        for (int i = 0; i < 5; i++) {
            Thread.sleep(200);
            System.out.println("  Working... (" + i + ")");
        }

        // Wait for response
        HttpResponse<String> response = future.join();
        System.out.println("\nResponse received after " +
            (System.currentTimeMillis() - start) + "ms");
        System.out.println("Status: " + response.statusCode());

        // Multiple concurrent requests
        System.out.println("\n=== Multiple Concurrent Requests ===");

        var urls = List.of(
            "https://httpbin.org/get?id=1",
            "https://httpbin.org/get?id=2",
            "https://httpbin.org/get?id=3",
            "https://httpbin.org/get?id=4",
            "https://httpbin.org/get?id=5"
        );

        start = System.currentTimeMillis();

        // Send all requests concurrently
        List<CompletableFuture<HttpResponse<String>>> futures = urls.stream()
            .map(url -> HttpRequest.newBuilder()
                .uri(URI.create(url))
                .build())
            .map(req -> client.sendAsync(req, BodyHandlers.ofString()))
            .toList();

        // Wait for all and collect results
        List<HttpResponse<String>> responses = futures.stream()
            .map(CompletableFuture::join)
            .toList();

        System.out.println("All " + responses.size() + " requests completed in " +
            (System.currentTimeMillis() - start) + "ms");

        for (var resp : responses) {
            System.out.println("  " + resp.uri().getQuery() + " -> " + resp.statusCode());
        }

        // Chaining async operations
        System.out.println("\n=== Chained Async Operations ===");

        client.sendAsync(
            HttpRequest.newBuilder()
                .uri(URI.create("https://httpbin.org/get"))
                .build(),
            BodyHandlers.ofString()
        )
        .thenApply(HttpResponse::body)
        .thenApply(body -> body.length())
        .thenAccept(length -> System.out.println("Response body length: " + length))
        .join();
    }
}
```

---

## Hands-On Exercise 4: Configuring the Client

Create `src/chapter09/ConfiguredHttpClient.java`:

```java
package chapter09;

import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;
import java.time.Duration;

public class ConfiguredHttpClient {
    public static void main(String[] args) throws Exception {
        // Configure client with custom settings
        var client = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_2)          // Prefer HTTP/2
            .followRedirects(HttpClient.Redirect.NORMAL) // Follow redirects
            .connectTimeout(Duration.ofSeconds(10))      // Connection timeout
            .build();

        System.out.println("=== Client Configuration ===");
        System.out.println("HTTP Version: " + client.version());
        System.out.println("Redirect Policy: " + client.followRedirects());
        System.out.println("Connect Timeout: " + client.connectTimeout());

        // Test redirect following
        System.out.println("\n=== Testing Redirects ===");
        var redirectRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/redirect/3"))  // 3 redirects
            .build();

        var response = client.send(redirectRequest, BodyHandlers.ofString());
        System.out.println("Final URL: " + response.uri());
        System.out.println("Status: " + response.statusCode());
        System.out.println("Previous responses: " + response.previousResponse());

        // Request with timeout
        System.out.println("\n=== Request Timeout ===");
        var timeoutRequest = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/delay/2"))
            .timeout(Duration.ofSeconds(5))  // Request timeout
            .build();

        try {
            var resp = client.send(timeoutRequest, BodyHandlers.ofString());
            System.out.println("Request completed: " + resp.statusCode());
        } catch (java.net.http.HttpTimeoutException e) {
            System.out.println("Request timed out!");
        }

        // HTTP/2 demonstration
        System.out.println("\n=== HTTP/2 Info ===");
        var http2Request = HttpRequest.newBuilder()
            .uri(URI.create("https://httpbin.org/get"))
            .build();

        var http2Response = client.send(http2Request, BodyHandlers.ofString());
        System.out.println("Protocol: " + http2Response.version());
    }
}
```

---

## Hands-On Exercise 5: Body Publishers and Handlers

Create `src/chapter09/BodyHandling.java`:

```java
package chapter09;

import java.io.*;
import java.net.URI;
import java.net.http.*;
import java.net.http.HttpRequest.BodyPublishers;
import java.net.http.HttpResponse.BodyHandlers;
import java.nio.file.*;
import java.util.stream.Stream;

public class BodyHandling {
    public static void main(String[] args) throws Exception {
        var client = HttpClient.newHttpClient();
        var baseUrl = "https://httpbin.org";

        // BodyPublishers: Different ways to send data

        // 1. String body
        System.out.println("=== String Body ===");
        var stringRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/post"))
            .POST(BodyPublishers.ofString("Hello, Server!"))
            .build();
        System.out.println("Sent string body");

        // 2. Form data
        System.out.println("\n=== Form Data ===");
        var formData = "username=john&password=secret123";
        var formRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/post"))
            .header("Content-Type", "application/x-www-form-urlencoded")
            .POST(BodyPublishers.ofString(formData))
            .build();

        var formResponse = client.send(formRequest, BodyHandlers.ofString());
        System.out.println("Form response status: " + formResponse.statusCode());

        // 3. File body
        System.out.println("\n=== File Body ===");
        Path tempFile = Files.createTempFile("upload", ".txt");
        Files.writeString(tempFile, "Content to upload!");

        var fileRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/post"))
            .POST(BodyPublishers.ofFile(tempFile))
            .build();
        System.out.println("Would send file: " + tempFile);
        Files.delete(tempFile);

        // 4. Byte array body
        System.out.println("\n=== Byte Array Body ===");
        byte[] data = "Binary data here".getBytes();
        var byteRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/post"))
            .POST(BodyPublishers.ofByteArray(data))
            .build();
        System.out.println("Sent " + data.length + " bytes");

        // 5. No body (empty)
        System.out.println("\n=== No Body ===");
        var noBodyRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/post"))
            .POST(BodyPublishers.noBody())
            .build();

        // BodyHandlers: Different ways to receive data

        // 1. As String
        System.out.println("\n=== Response as String ===");
        var strResponse = client.send(
            HttpRequest.newBuilder()
                .uri(URI.create(baseUrl + "/get"))
                .build(),
            BodyHandlers.ofString()
        );
        System.out.println("Body length: " + strResponse.body().length());

        // 2. As byte array
        System.out.println("\n=== Response as Bytes ===");
        var byteResponse = client.send(
            HttpRequest.newBuilder()
                .uri(URI.create(baseUrl + "/bytes/100"))  // Get 100 random bytes
                .build(),
            BodyHandlers.ofByteArray()
        );
        System.out.println("Received bytes: " + byteResponse.body().length);

        // 3. As lines (Stream<String>)
        System.out.println("\n=== Response as Lines ===");
        var linesResponse = client.send(
            HttpRequest.newBuilder()
                .uri(URI.create(baseUrl + "/get"))
                .build(),
            BodyHandlers.ofLines()
        );

        Stream<String> lines = linesResponse.body();
        System.out.println("First 5 lines:");
        lines.limit(5).forEach(line -> System.out.println("  " + line));

        // 4. Discard body
        System.out.println("\n=== Discard Body ===");
        var discardResponse = client.send(
            HttpRequest.newBuilder()
                .uri(URI.create(baseUrl + "/get"))
                .build(),
            BodyHandlers.discarding()
        );
        System.out.println("Status: " + discardResponse.statusCode());
        System.out.println("Body (discarded): " + discardResponse.body());

        // 5. Save to file
        System.out.println("\n=== Save to File ===");
        Path outputFile = Files.createTempFile("response", ".json");
        var fileResponse = client.send(
            HttpRequest.newBuilder()
                .uri(URI.create(baseUrl + "/json"))
                .build(),
            BodyHandlers.ofFile(outputFile)
        );
        System.out.println("Saved to: " + fileResponse.body());
        System.out.println("Content: " + Files.readString(outputFile));
        Files.delete(outputFile);
    }
}
```

---

## Hands-On Exercise 6: Headers and Authentication

Create `src/chapter09/HeadersAndAuth.java`:

```java
package chapter09;

import java.net.URI;
import java.net.http.*;
import java.net.http.HttpResponse.BodyHandlers;
import java.util.Base64;

public class HeadersAndAuth {
    public static void main(String[] args) throws Exception {
        var client = HttpClient.newHttpClient();
        var baseUrl = "https://httpbin.org";

        // Setting headers
        System.out.println("=== Custom Headers ===");
        var request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/headers"))
            .header("X-Custom-Header", "CustomValue")
            .header("Accept", "application/json")
            .header("User-Agent", "Java HttpClient")
            .headers("X-Header-1", "Value1", "X-Header-2", "Value2")  // Multiple
            .build();

        var response = client.send(request, BodyHandlers.ofString());
        System.out.println("Response:\n" + response.body());

        // Reading response headers
        System.out.println("\n=== Reading Response Headers ===");
        var headers = response.headers();

        System.out.println("Content-Type: " +
            headers.firstValue("content-type").orElse("not set"));

        System.out.println("All headers:");
        headers.map().forEach((name, values) ->
            System.out.println("  " + name + ": " + values));

        // Basic Authentication
        System.out.println("\n=== Basic Authentication ===");
        String username = "testuser";
        String password = "testpass";
        String auth = username + ":" + password;
        String encodedAuth = Base64.getEncoder().encodeToString(auth.getBytes());

        var authRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/basic-auth/" + username + "/" + password))
            .header("Authorization", "Basic " + encodedAuth)
            .build();

        var authResponse = client.send(authRequest, BodyHandlers.ofString());
        System.out.println("Auth Status: " + authResponse.statusCode());
        System.out.println("Response: " + authResponse.body());

        // Bearer token (JWT style)
        System.out.println("\n=== Bearer Token ===");
        String token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";

        var tokenRequest = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/bearer"))
            .header("Authorization", "Bearer " + token)
            .build();

        var tokenResponse = client.send(tokenRequest, BodyHandlers.ofString());
        System.out.println("Token Auth Status: " + tokenResponse.statusCode());
    }
}
```

---

## Hands-On Exercise 7: Building a Simple REST Client

Create `src/chapter09/RestClient.java`:

```java
package chapter09;

import java.net.URI;
import java.net.http.*;
import java.net.http.HttpRequest.BodyPublishers;
import java.net.http.HttpResponse.BodyHandlers;
import java.time.Duration;
import java.util.*;

public class RestClient {
    private final HttpClient client;
    private final String baseUrl;

    public RestClient(String baseUrl) {
        this.baseUrl = baseUrl;
        this.client = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .followRedirects(HttpClient.Redirect.NORMAL)
            .build();
    }

    // GET request
    public ApiResponse get(String path) throws Exception {
        var request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + path))
            .header("Accept", "application/json")
            .GET()
            .build();

        return executeRequest(request);
    }

    // POST request
    public ApiResponse post(String path, String jsonBody) throws Exception {
        var request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + path))
            .header("Content-Type", "application/json")
            .header("Accept", "application/json")
            .POST(BodyPublishers.ofString(jsonBody))
            .build();

        return executeRequest(request);
    }

    // PUT request
    public ApiResponse put(String path, String jsonBody) throws Exception {
        var request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + path))
            .header("Content-Type", "application/json")
            .header("Accept", "application/json")
            .PUT(BodyPublishers.ofString(jsonBody))
            .build();

        return executeRequest(request);
    }

    // DELETE request
    public ApiResponse delete(String path) throws Exception {
        var request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + path))
            .DELETE()
            .build();

        return executeRequest(request);
    }

    private ApiResponse executeRequest(HttpRequest request) throws Exception {
        var response = client.send(request, BodyHandlers.ofString());
        return new ApiResponse(
            response.statusCode(),
            response.body(),
            response.headers().map()
        );
    }

    // Response record
    public record ApiResponse(
        int statusCode,
        String body,
        Map<String, List<String>> headers
    ) {
        public boolean isSuccess() {
            return statusCode >= 200 && statusCode < 300;
        }

        public boolean isClientError() {
            return statusCode >= 400 && statusCode < 500;
        }

        public boolean isServerError() {
            return statusCode >= 500;
        }
    }

    public static void main(String[] args) throws Exception {
        var api = new RestClient("https://httpbin.org");

        // Test GET
        System.out.println("=== GET /get ===");
        var getResponse = api.get("/get?name=Java");
        System.out.println("Status: " + getResponse.statusCode());
        System.out.println("Success: " + getResponse.isSuccess());

        // Test POST
        System.out.println("\n=== POST /post ===");
        var postBody = """
            {
                "title": "Test Post",
                "content": "Hello from Java HTTP Client!"
            }
            """;
        var postResponse = api.post("/post", postBody);
        System.out.println("Status: " + postResponse.statusCode());

        // Test PUT
        System.out.println("\n=== PUT /put ===");
        var putBody = """
            {
                "id": 1,
                "title": "Updated Title"
            }
            """;
        var putResponse = api.put("/put", putBody);
        System.out.println("Status: " + putResponse.statusCode());

        // Test DELETE
        System.out.println("\n=== DELETE /delete ===");
        var deleteResponse = api.delete("/delete");
        System.out.println("Status: " + deleteResponse.statusCode());

        // Test error handling
        System.out.println("\n=== Error Response ===");
        var errorResponse = api.get("/status/404");
        System.out.println("Status: " + errorResponse.statusCode());
        System.out.println("Is Client Error: " + errorResponse.isClientError());
    }
}
```

---

## Summary: HTTP Client API

| Class/Interface | Purpose |
|-----------------|---------|
| `HttpClient` | Send requests, configurable (timeouts, version, redirects) |
| `HttpRequest` | Represents a request (URI, method, headers, body) |
| `HttpResponse` | Represents a response (status, headers, body) |
| `BodyPublishers` | Create request bodies (string, file, bytes) |
| `BodyHandlers` | Handle response bodies (string, file, bytes, lines) |

### Key Methods

```java
// Client creation
HttpClient.newHttpClient()
HttpClient.newBuilder().build()

// Request building
HttpRequest.newBuilder().uri(...).GET/POST/PUT/DELETE().build()

// Sending
client.send(request, bodyHandler)        // Synchronous
client.sendAsync(request, bodyHandler)   // Async -> CompletableFuture
```

---

## Practice Exercises

### Exercise 9.1: JSON API Client
Create a client that fetches user data from JSONPlaceholder API (https://jsonplaceholder.typicode.com/users).

<details>
<summary>Click for solution</summary>

```java
var client = HttpClient.newHttpClient();
var request = HttpRequest.newBuilder()
    .uri(URI.create("https://jsonplaceholder.typicode.com/users"))
    .GET()
    .build();

var response = client.send(request, BodyHandlers.ofString());
System.out.println(response.body());
```
</details>

### Exercise 9.2: Retry Logic
Implement a method that retries failed requests up to 3 times with exponential backoff.

<details>
<summary>Click for solution</summary>

```java
HttpResponse<String> sendWithRetry(HttpClient client, HttpRequest request, int maxRetries)
    throws Exception {

    int attempt = 0;
    while (true) {
        try {
            var response = client.send(request, BodyHandlers.ofString());
            if (response.statusCode() < 500) {
                return response;
            }
            throw new IOException("Server error: " + response.statusCode());
        } catch (IOException e) {
            attempt++;
            if (attempt >= maxRetries) throw e;
            Thread.sleep((long) Math.pow(2, attempt) * 1000);
        }
    }
}
```
</details>

---

## Key Takeaways

1. **HttpClient is immutable and thread-safe** - create once, reuse
2. **HttpRequest.newBuilder()** - fluent API for building requests
3. **Synchronous and async** - `send()` vs `sendAsync()`
4. **BodyPublishers** - various ways to send data
5. **BodyHandlers** - various ways to receive data
6. **HTTP/2 support** - automatic protocol negotiation
7. **Built-in redirect handling** - configurable policy

---

## What's Next?

In [Chapter 10: Virtual Threads](10-virtual-threads.md), you'll learn about Java 21's revolutionary approach to concurrency.

---

[← Previous Chapter](08-optional-enhancements.md) | [Back to Contents](../README.md) | [Next Chapter →](10-virtual-threads.md)
