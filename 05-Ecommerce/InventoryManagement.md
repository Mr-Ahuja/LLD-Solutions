# Inventory Management System - Low Level Design

## Problem Statement

Design an inventory system managing stock levels, reservations, replenishment, and warehouse locations.

---

## Key Implementation

```java
public class InventoryItem {
    private final String productId;
    private int availableQuantity;
    private int reservedQuantity;
    private int reorderLevel;
    private final String warehouseId;

    public synchronized boolean reserve(int quantity) {
        if (availableQuantity >= quantity) {
            availableQuantity -= quantity;
            reservedQuantity += quantity;
            return true;
        }
        return false;
    }

    public synchronized void releaseReservation(int quantity) {
        reservedQuantity -= quantity;
        availableQuantity += quantity;
    }

    public synchronized void confirmReservation(int quantity) {
        reservedQuantity -= quantity;
        // Quantity removed from inventory
    }

    public synchronized void addStock(int quantity) {
        availableQuantity += quantity;
    }

    public boolean needsReplenishment() {
        return availableQuantity + reservedQuantity < reorderLevel;
    }
}

public class InventoryManagementSystem {
    private final Map<String, InventoryItem> inventory;
    private final Map<String, List<String>> warehouseInventory;

    public boolean checkAvailability(String productId, int quantity) {
        InventoryItem item = inventory.get(productId);
        return item != null && item.getAvailableQuantity() >= quantity;
    }

    public boolean reserveStock(String productId, int quantity) {
        InventoryItem item = inventory.get(productId);

        if (item != null && item.reserve(quantity)) {
            if (item.needsReplenishment()) {
                triggerReplenishment(productId);
            }
            return true;
        }

        return false;
    }

    public void releaseReservation(String productId, int quantity) {
        InventoryItem item = inventory.get(productId);
        if (item != null) {
            item.releaseReservation(quantity);
        }
    }

    public void fulfillOrder(String productId, int quantity) {
        InventoryItem item = inventory.get(productId);
        if (item != null) {
            item.confirmReservation(quantity);
        }
    }

    public void addStock(String productId, int quantity, String warehouseId) {
        InventoryItem item = inventory.get(productId);
        if (item != null) {
            item.addStock(quantity);
        }
    }

    private void triggerReplenishment(String productId) {
        System.out.println("Replenishment triggered for product: " + productId);
        // Send replenishment request to supplier
    }

    public int getAvailableQuantity(String productId) {
        InventoryItem item = inventory.get(productId);
        return item != null ? item.getAvailableQuantity() : 0;
    }
}
```

---

## Summary

Demonstrates: Stock reservation, concurrent inventory updates, reorder level monitoring, warehouse management.
