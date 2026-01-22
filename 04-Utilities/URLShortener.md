# URL Shortener (bit.ly) - Low Level Design

## Problem Statement

Design a URL shortening service that converts long URLs to short codes and redirects users.

---

## Key Classes

```java
public class URLShortener {
    private final Map<String, String> shortToLong;
    private final Map<String, String> longToShort;
    private final Map<String, URLMetrics> metrics;
    private static final String BASE_URL = "https://short.ly/";
    private static final String ALPHABET = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    private static final int SHORT_CODE_LENGTH = 7;

    public String shortenURL(String longURL) {
        if (longToShort.containsKey(longURL)) {
            return BASE_URL + longToShort.get(longURL);
        }

        String shortCode = generateShortCode();
        shortToLong.put(shortCode, longURL);
        longToShort.put(longURL, shortCode);
        metrics.put(shortCode, new URLMetrics());

        return BASE_URL + shortCode;
    }

    public String expandURL(String shortCode) {
        String longURL = shortToLong.get(shortCode);

        if (longURL != null) {
            // Track click
            URLMetrics metric = metrics.get(shortCode);
            metric.incrementClicks();
        }

        return longURL;
    }

    private String generateShortCode() {
        // Base62 encoding or random generation
        StringBuilder sb = new StringBuilder();
        Random random = new Random();

        for (int i = 0; i < SHORT_CODE_LENGTH; i++) {
            sb.append(ALPHABET.charAt(random.nextInt(ALPHABET.length())));
        }

        // Check for collision
        if (shortToLong.containsKey(sb.toString())) {
            return generateShortCode();  // Retry
        }

        return sb.toString();
    }
}

public class URLMetrics {
    private int clickCount;
    private final LocalDateTime createdAt;
    private final Map<String, Integer> clicksByCountry;

    public void incrementClicks() {
        clickCount++;
    }
}
```

---

## Design Decisions

### 1. **Base62 Encoding**
**Decision**: Use [a-zA-Z0-9] (62 characters) for short codes
**Reasoning**:
- 62^7 ≈ 3.5 trillion URLs
- URL-safe characters
- Case-sensitive for more combinations

### 2. **Bidirectional Mapping**
**Decision**: Store both shortToLong and longToShort
**Reasoning**:
- Avoid duplicate shortened URLs for same long URL
- O(1) lookup in both directions
- Space trade-off worth it

---

## Summary

Demonstrates: Hash collision handling, base62 encoding, bidirectional mapping, click tracking.
