# Shopping Cart System - Low Level Design

## Problem Statement

Design a shopping cart for e-commerce with add/remove items, quantity management, pricing, and checkout.

---

## Key Implementation

```java
public class Product {
    private final String productId;
    private final String name;
    private final double price;
    private final int availableQuantity;

    public boolean isInStock(int quantity) {
        return availableQuantity >= quantity;
    }
}

public class CartItem {
    private final Product product;
    private int quantity;

    public CartItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }

    public void updateQuantity(int newQuantity) {
        if (!product.isInStock(newQuantity)) {
            throw new IllegalStateException("Insufficient stock");
        }
        this.quantity = newQuantity;
    }

    public double getSubtotal() {
        return product.getPrice() * quantity;
    }
}

public class ShoppingCart {
    private final String cartId;
    private final String userId;
    private final Map<String, CartItem> items;
    private final LocalDateTime createdAt;

    public ShoppingCart(String userId) {
        this.cartId = UUID.randomUUID().toString();
        this.userId = userId;
        this.items = new ConcurrentHashMap<>();
        this.createdAt = LocalDateTime.now();
    }

    public void addItem(Product product, int quantity) {
        if (!product.isInStock(quantity)) {
            throw new IllegalStateException("Product out of stock");
        }

        CartItem existing = items.get(product.getProductId());

        if (existing != null) {
            existing.updateQuantity(existing.getQuantity() + quantity);
        } else {
            items.put(product.getProductId(), new CartItem(product, quantity));
        }
    }

    public void removeItem(String productId) {
        items.remove(productId);
    }

    public void updateQuantity(String productId, int quantity) {
        CartItem item = items.get(productId);
        if (item != null) {
            if (quantity == 0) {
                removeItem(productId);
            } else {
                item.updateQuantity(quantity);
            }
        }
    }

    public double calculateTotal() {
        return items.values().stream()
            .mapToDouble(CartItem::getSubtotal)
            .sum();
    }

    public void clear() {
        items.clear();
    }

    public boolean isEmpty() {
        return items.isEmpty();
    }

    public List<CartItem> getItems() {
        return new ArrayList<>(items.values());
    }
}
```

---

## Summary

Demonstrates: Cart management, quantity updates, stock validation, total calculation, concurrent operations.
