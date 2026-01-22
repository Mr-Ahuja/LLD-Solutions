# LRU Cache - Low Level Design

## Problem Statement

Design an LRU (Least Recently Used) cache with O(1) get and put operations.

---

## Key Implementation

```java
public class LRUCache<K, V> {
    private class Node {
        K key;
        V value;
        Node prev;
        Node next;

        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }

    private final int capacity;
    private final Map<K, Node> cache;
    private final Node head;  // Dummy head
    private final Node tail;  // Dummy tail

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new HashMap<>();
        this.head = new Node(null, null);
        this.tail = new Node(null, null);
        head.next = tail;
        tail.prev = head;
    }

    public V get(K key) {
        Node node = cache.get(key);
        if (node == null) {
            return null;
        }

        // Move to front (most recently used)
        moveToFront(node);
        return node.value;
    }

    public void put(K key, V value) {
        Node node = cache.get(key);

        if (node != null) {
            // Update existing
            node.value = value;
            moveToFront(node);
        } else {
            // Add new node
            Node newNode = new Node(key, value);
            cache.put(key, newNode);
            addToFront(newNode);

            // Evict if over capacity
            if (cache.size() > capacity) {
                Node lru = removeLast();
                cache.remove(lru.key);
            }
        }
    }

    private void moveToFront(Node node) {
        removeNode(node);
        addToFront(node);
    }

    private void addToFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    private void removeNode(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private Node removeLast() {
        Node lru = tail.prev;
        removeNode(lru);
        return lru;
    }
}
```

---

## Design Decisions

### 1. **HashMap + Doubly Linked List**
**Decision**: Combine HashMap for O(1) lookup and DLL for O(1) removal/insertion
**Reasoning**:
- HashMap provides O(1) access
- DLL allows O(1) move-to-front and LRU eviction
- Together achieve all O(1) operations

### 2. **Dummy Head/Tail**
**Decision**: Use sentinel nodes to simplify edge cases
**Reasoning**:
- No null checks for head/tail
- Uniform insertion/deletion logic
- Cleaner code

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| get() | O(1) | - |
| put() | O(1) | - |
| Total | - | O(capacity) |

---

## Summary

Demonstrates: Data structure combination (HashMap + DLL), cache eviction policy, O(1) operations.
