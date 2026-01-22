# LFU Cache - Low Level Design

## Problem Statement

Design an LFU (Least Frequently Used) cache with O(1) operations, evicting items with lowest access frequency.

---

## Key Implementation

```java
public class LFUCache<K, V> {
    private class Node {
        K key;
        V value;
        int frequency;

        Node(K key, V value) {
            this.key = key;
            this.value = value;
            this.frequency = 1;
        }
    }

    private final int capacity;
    private int minFrequency;
    private final Map<K, Node> cache;
    private final Map<Integer, LinkedHashSet<K>> frequencyMap;

    public LFUCache(int capacity) {
        this.capacity = capacity;
        this.minFrequency = 0;
        this.cache = new HashMap<>();
        this.frequencyMap = new HashMap<>();
    }

    public V get(K key) {
        Node node = cache.get(key);
        if (node == null) {
            return null;
        }

        updateFrequency(node);
        return node.value;
    }

    public void put(K key, V value) {
        if (capacity == 0) return;

        Node node = cache.get(key);

        if (node != null) {
            node.value = value;
            updateFrequency(node);
        } else {
            if (cache.size() >= capacity) {
                evict();
            }

            Node newNode = new Node(key, value);
            cache.put(key, newNode);
            frequencyMap.computeIfAbsent(1, k -> new LinkedHashSet<>()).add(key);
            minFrequency = 1;
        }
    }

    private void updateFrequency(Node node) {
        int oldFreq = node.frequency;
        int newFreq = oldFreq + 1;

        // Remove from old frequency list
        frequencyMap.get(oldFreq).remove(node.key);
        if (frequencyMap.get(oldFreq).isEmpty()) {
            frequencyMap.remove(oldFreq);
            if (minFrequency == oldFreq) {
                minFrequency = newFreq;
            }
        }

        // Add to new frequency list
        node.frequency = newFreq;
        frequencyMap.computeIfAbsent(newFreq, k -> new LinkedHashSet<>()).add(node.key);
    }

    private void evict() {
        LinkedHashSet<K> keys = frequencyMap.get(minFrequency);
        K keyToEvict = keys.iterator().next();
        keys.remove(keyToEvict);

        if (keys.isEmpty()) {
            frequencyMap.remove(minFrequency);
        }

        cache.remove(keyToEvict);
    }
}
```

---

## Summary

Demonstrates: Frequency tracking, LinkedHashSet for LRU within same frequency, O(1) operations.
