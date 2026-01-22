# Key-Value Store (Redis-like) - Low Level Design

## Problem Statement

Design an in-memory key-value store with support for strings, lists, sets, and expiration.

---

## Key Implementation

```java
public class KeyValueStore {
    private final Map<String, Value> store;
    private final Map<String, Long> expirations;

    public KeyValueStore() {
        this.store = new ConcurrentHashMap<>();
        this.expirations = new ConcurrentHashMap<>();
        startExpirationChecker();
    }

    public void set(String key, String value) {
        store.put(key, new StringValue(value));
    }

    public void set(String key, String value, long ttlSeconds) {
        set(key, value);
        expirations.put(key, System.currentTimeMillis() + ttlSeconds * 1000);
    }

    public String get(String key) {
        if (isExpired(key)) {
            delete(key);
            return null;
        }

        Value value = store.get(key);
        return value instanceof StringValue ? ((StringValue) value).getValue() : null;
    }

    public void delete(String key) {
        store.remove(key);
        expirations.remove(key);
    }

    public boolean exists(String key) {
        if (isExpired(key)) {
            delete(key);
            return false;
        }
        return store.containsKey(key);
    }

    private boolean isExpired(String key) {
        Long expiration = expirations.get(key);
        return expiration != null && System.currentTimeMillis() > expiration;
    }

    private void startExpirationChecker() {
        Thread checker = new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(1000);
                    List<String> expiredKeys = expirations.entrySet().stream()
                        .filter(e -> System.currentTimeMillis() > e.getValue())
                        .map(Map.Entry::getKey)
                        .collect(Collectors.toList());

                    expiredKeys.forEach(this::delete);
                } catch (InterruptedException e) {
                    break;
                }
            }
        });
        checker.setDaemon(true);
        checker.start();
    }
}

interface Value {}

class StringValue implements Value {
    private final String value;

    StringValue(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}
```

---

## Summary

Demonstrates: In-memory storage, TTL management, background expiration checker, type-safe values.
