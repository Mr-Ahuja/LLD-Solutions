# Search Autocomplete / Typeahead - Low Level Design

## Problem Statement

Design a search autocomplete system using Trie data structure for efficient prefix-based search suggestions.

---

## Key Implementation

```java
public class TrieNode {
    private final Map<Character, TrieNode> children;
    private boolean isEndOfWord;
    private int frequency;  // For popularity-based ranking

    public TrieNode() {
        this.children = new HashMap<>();
        this.isEndOfWord = false;
        this.frequency = 0;
    }
}

public class AutocompleteSystem {
    private final TrieNode root;
    private static final int MAX_SUGGESTIONS = 10;

    public AutocompleteSystem() {
        this.root = new TrieNode();
    }

    public void insert(String word) {
        TrieNode current = root;

        for (char ch : word.toLowerCase().toCharArray()) {
            current.children.putIfAbsent(ch, new TrieNode());
            current = current.children.get(ch);
        }

        current.isEndOfWord = true;
        current.frequency++;
    }

    public List<String> getSuggestions(String prefix) {
        TrieNode node = findNode(prefix.toLowerCase());

        if (node == null) {
            return new ArrayList<>();
        }

        List<SuggestionWithFrequency> suggestions = new ArrayList<>();
        collectSuggestions(node, prefix.toLowerCase(), suggestions);

        // Sort by frequency (descending) and limit
        return suggestions.stream()
            .sorted(Comparator.comparingInt(SuggestionWithFrequency::getFrequency).reversed())
            .limit(MAX_SUGGESTIONS)
            .map(SuggestionWithFrequency::getWord)
            .collect(Collectors.toList());
    }

    private TrieNode findNode(String prefix) {
        TrieNode current = root;

        for (char ch : prefix.toCharArray()) {
            current = current.children.get(ch);
            if (current == null) {
                return null;
            }
        }

        return current;
    }

    private void collectSuggestions(TrieNode node, String prefix, List<SuggestionWithFrequency> suggestions) {
        if (node.isEndOfWord) {
            suggestions.add(new SuggestionWithFrequency(prefix, node.frequency));
        }

        for (Map.Entry<Character, TrieNode> entry : node.children.entrySet()) {
            collectSuggestions(entry.getValue(), prefix + entry.getKey(), suggestions);
        }
    }

    public boolean search(String word) {
        TrieNode node = findNode(word.toLowerCase());
        return node != null && node.isEndOfWord;
    }

    public boolean startsWith(String prefix) {
        return findNode(prefix.toLowerCase()) != null;
    }

    public void delete(String word) {
        delete(root, word.toLowerCase(), 0);
    }

    private boolean delete(TrieNode node, String word, int index) {
        if (index == word.length()) {
            if (!node.isEndOfWord) {
                return false;
            }
            node.isEndOfWord = false;
            return node.children.isEmpty();
        }

        char ch = word.charAt(index);
        TrieNode child = node.children.get(ch);

        if (child == null) {
            return false;
        }

        boolean shouldDeleteChild = delete(child, word, index + 1);

        if (shouldDeleteChild) {
            node.children.remove(ch);
            return node.children.isEmpty() && !node.isEndOfWord;
        }

        return false;
    }
}

class SuggestionWithFrequency {
    private final String word;
    private final int frequency;

    public SuggestionWithFrequency(String word, int frequency) {
        this.word = word;
        this.frequency = frequency;
    }

    public String getWord() {
        return word;
    }

    public int getFrequency() {
        return frequency;
    }
}
```

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Insert | O(m) | O(m) |
| Search | O(m) | O(1) |
| AutoComplete | O(m + n) | O(n) |

Where m = word length, n = total suggestions

---

## Summary

Demonstrates: Trie data structure, prefix search, frequency-based ranking, DFS for suggestion collection.
