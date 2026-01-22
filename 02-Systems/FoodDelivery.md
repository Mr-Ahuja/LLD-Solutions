# Food Delivery System (Zomato/Swiggy) - Low Level Design

## Problem Statement

Design a food delivery system with restaurant management, order processing, delivery assignment, and tracking.

---

## Key Classes

```java
public class Restaurant {
    private final String id;
    private final String name;
    private final Address address;
    private final List<MenuItem> menu;
    private boolean isOpen;
    private double rating;

    public List<MenuItem> getAvailableItems() {
        return menu.stream().filter(MenuItem::isAvailable).collect(Collectors.toList());
    }
}

public class MenuItem {
    private final String id;
    private final String name;
    private final double price;
    private final String category;
    private boolean isAvailable;
}

public class Customer {
    private final String customerId;
    private final String name;
    private final List<Address> savedAddresses;
    private Cart cart;
}

public class Cart {
    private final Map<MenuItem, Integer> items;

    public void addItem(MenuItem item, int quantity) {
        items.put(item, items.getOrDefault(item, 0) + quantity);
    }

    public double calculateTotal() {
        return items.entrySet().stream()
            .mapToDouble(e -> e.getKey().getPrice() * e.getValue())
            .sum();
    }

    public void clear() {
        items.clear();
    }
}

public class Order {
    private final String orderId;
    private final Customer customer;
    private final Restaurant restaurant;
    private final List<OrderItem> items;
    private final Address deliveryAddress;
    private OrderStatus status;
    private DeliveryAgent deliveryAgent;
    private double totalAmount;
    private final LocalDateTime orderTime;

    public void assignDeliveryAgent(DeliveryAgent agent) {
        this.deliveryAgent = agent;
        this.status = OrderStatus.ASSIGNED;
    }

    public void updateStatus(OrderStatus newStatus) {
        this.status = newStatus;
        notifyCustomer();
    }

    private void notifyCustomer() {
        System.out.println("Order " + orderId + " status: " + status);
    }
}

public class OrderItem {
    private final MenuItem menuItem;
    private final int quantity;
    private final double price;

    public double getTotal() {
        return price * quantity;
    }
}

public enum OrderStatus {
    PLACED,
    CONFIRMED,
    PREPARING,
    READY_FOR_PICKUP,
    ASSIGNED,
    PICKED_UP,
    OUT_FOR_DELIVERY,
    DELIVERED,
    CANCELLED
}

public class DeliveryAgent {
    private final String agentId;
    private final String name;
    private Location currentLocation;
    private AgentStatus status;
    private Vehicle vehicle;

    public void updateLocation(Location location) {
        this.currentLocation = location;
    }
}

public enum AgentStatus {
    AVAILABLE,
    BUSY,
    OFFLINE
}

public class FoodDeliverySystem {
    private final Map<String, Restaurant> restaurants;
    private final Map<String, Customer> customers;
    private final Map<String, DeliveryAgent> deliveryAgents;
    private final Map<String, Order> activeOrders;

    public List<Restaurant> searchRestaurants(String city, String cuisine) {
        return restaurants.values().stream()
            .filter(r -> r.getAddress().getCity().equals(city))
            .filter(r -> r.isOpen())
            .collect(Collectors.toList());
    }

    public Order placeOrder(String customerId, Cart cart, Address deliveryAddress) {
        Customer customer = customers.get(customerId);

        List<OrderItem> items = cart.getItems().entrySet().stream()
            .map(e -> new OrderItem(e.getKey(), e.getValue(), e.getKey().getPrice()))
            .collect(Collectors.toList());

        Order order = new Order(customer, cart.getRestaurant(), items, deliveryAddress);
        order.setTotalAmount(cart.calculateTotal());

        activeOrders.put(order.getOrderId(), order);
        cart.clear();

        // Assign delivery agent
        assignDeliveryAgent(order);

        return order;
    }

    private void assignDeliveryAgent(Order order) {
        Location restaurantLocation = order.getRestaurant().getAddress().getLocation();

        DeliveryAgent agent = deliveryAgents.values().stream()
            .filter(a -> a.getStatus() == AgentStatus.AVAILABLE)
            .min(Comparator.comparingDouble(a ->
                a.getCurrentLocation().distanceTo(restaurantLocation)))
            .orElse(null);

        if (agent != null) {
            order.assignDeliveryAgent(agent);
            agent.setStatus(AgentStatus.BUSY);
        }
    }

    public void updateOrderStatus(String orderId, OrderStatus status) {
        Order order = activeOrders.get(orderId);
        if (order != null) {
            order.updateStatus(status);

            if (status == OrderStatus.DELIVERED) {
                order.getDeliveryAgent().setStatus(AgentStatus.AVAILABLE);
                activeOrders.remove(orderId);
            }
        }
    }
}
```

---

## Summary

Demonstrates: Restaurant management, cart handling, order lifecycle, delivery agent assignment, real-time tracking.
