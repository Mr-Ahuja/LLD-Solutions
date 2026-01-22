# Rate Limiter - Low Level Design

## Problem Statement

Design a rate limiter to control request rates using token bucket or sliding window algorithms.

---

## Key Implementations

### Token Bucket Algorithm

```java
public class TokenBucketRateLimiter {
    private final long capacity;
    private final long refillRate;  // tokens per second
    private long availableTokens;
    private long lastRefillTime;

    public TokenBucketRateLimiter(long capacity, long refillRate) {
        this.capacity = capacity;
        this.refillRate = refillRate;
        this.availableTokens = capacity;
        this.lastRefillTime = System.currentTimeMillis();
    }

    public synchronized boolean allowRequest() {
        refill();

        if (availableTokens > 0) {
            availableTokens--;
            return true;
        }

        return false;
    }

    private void refill() {
        long now = System.currentTimeMillis();
        long elapsedTime = now - lastRefillTime;
        long tokensToAdd = (elapsedTime / 1000) * refillRate;

        if (tokensToAdd > 0) {
            availableTokens = Math.min(capacity, availableTokens + tokensToAdd);
            lastRefillTime = now;
        }
    }
}
```

### Sliding Window Log Algorithm

```java
public class SlidingWindowLogRateLimiter {
    private final int maxRequests;
    private final long windowSizeMs;
    private final Queue<Long> requestTimestamps;

    public SlidingWindowLogRateLimiter(int maxRequests, long windowSizeMs) {
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSizeMs;
        this.requestTimestamps = new LinkedList<>();
    }

    public synchronized boolean allowRequest() {
        long now = System.currentTimeMillis();

        // Remove old requests outside window
        while (!requestTimestamps.isEmpty() &&
               now - requestTimestamps.peek() > windowSizeMs) {
            requestTimestamps.poll();
        }

        if (requestTimestamps.size() < maxRequests) {
            requestTimestamps.offer(now);
            return true;
        }

        return false;
    }
}
```

### Fixed Window Counter

```java
public class FixedWindowRateLimiter {
    private final int maxRequests;
    private final long windowSizeMs;
    private int requestCount;
    private long windowStart;

    public FixedWindowRateLimiter(int maxRequests, long windowSizeMs) {
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSizeMs;
        this.requestCount = 0;
        this.windowStart = System.currentTimeMillis();
    }

    public synchronized boolean allowRequest() {
        long now = System.currentTimeMillis();

        // Check if new window
        if (now - windowStart >= windowSizeMs) {
            requestCount = 0;
            windowStart = now;
        }

        if (requestCount < maxRequests) {
            requestCount++;
            return true;
        }

        return false;
    }
}
```

---

## Comparison

| Algorithm | Pros | Cons | Use Case |
|-----------|------|------|----------|
| Token Bucket | Smooth rate limiting, allows bursts | Complex refill logic | API rate limiting |
| Sliding Window Log | Precise, no boundary issues | Memory intensive | Analytics, strict limits |
| Fixed Window | Simple, low memory | Burst at window boundaries | Simple scenarios |

---

## Summary

Demonstrates: Rate limiting algorithms, token bucket, sliding window, time-based access control.
