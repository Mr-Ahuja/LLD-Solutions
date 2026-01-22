# Order Management System - Low Level Design

## Problem Statement

Design an order management system tracking order lifecycle from creation to delivery with status transitions.

---

## Key Implementation

```java
public class Order {
    private final String orderId;
    private final String customerId;
    private final List<OrderItem> items;
    private final Address shippingAddress;
    private OrderStatus status;
    private final double totalAmount;
    private final LocalDateTime createdAt;
    private LocalDateTime deliveredAt;

    public void updateStatus(OrderStatus newStatus) {
        if (!isValidTransition(status, newStatus)) {
            throw new IllegalStateException("Invalid status transition");
        }
        this.status = newStatus;

        if (newStatus == OrderStatus.DELIVERED) {
            this.deliveredAt = LocalDateTime.now();
        }
    }

    private boolean isValidTransition(OrderStatus from, OrderStatus to) {
        switch (from) {
            case PENDING:
                return to == OrderStatus.CONFIRMED || to == OrderStatus.CANCELLED;
            case CONFIRMED:
                return to == OrderStatus.SHIPPED || to == OrderStatus.CANCELLED;
            case SHIPPED:
                return to == OrderStatus.DELIVERED;
            case DELIVERED:
                return to == OrderStatus.RETURNED;
            default:
                return false;
        }
    }
}

public enum OrderStatus {
    PENDING,
    CONFIRMED,
    SHIPPED,
    DELIVERED,
    CANCELLED,
    RETURNED
}

public class OrderItem {
    private final Product product;
    private final int quantity;
    private final double priceAtPurchase;

    public double getTotal() {
        return priceAtPurchase * quantity;
    }
}

public class OrderManagementSystem {
    private final Map<String, Order> orders;

    public Order createOrder(String customerId, List<OrderItem> items, Address address) {
        double total = items.stream().mapToDouble(OrderItem::getTotal).sum();
        Order order = new Order(customerId, items, address, total);
        orders.put(order.getOrderId(), order);
        return order;
    }

    public void confirmOrder(String orderId) {
        Order order = orders.get(orderId);
        order.updateStatus(OrderStatus.CONFIRMED);
        // Process payment, reserve inventory
    }

    public void shipOrder(String orderId) {
        Order order = orders.get(orderId);
        order.updateStatus(OrderStatus.SHIPPED);
        // Generate tracking number, notify customer
    }

    public void deliverOrder(String orderId) {
        Order order = orders.get(orderId);
        order.updateStatus(OrderStatus.DELIVERED);
    }

    public void cancelOrder(String orderId) {
        Order order = orders.get(orderId);
        order.updateStatus(OrderStatus.CANCELLED);
        // Refund payment, release inventory
    }
}
```

---

## Summary

Demonstrates: State machine for order lifecycle, status transition validation, timestamp tracking, order processing workflow.
