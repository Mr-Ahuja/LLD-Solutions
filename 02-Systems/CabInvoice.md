# Cab Invoice Generator - Low Level Design

## Problem Statement

Design a system to generate invoices for cab rides with variable pricing based on distance, time, and ride type.

---

## Key Classes

```java
public class Ride {
    private final double distance;  // in km
    private final int duration;     // in minutes
    private final RideType type;

    public double calculateFare() {
        return type.getBaseFare() +
               (distance * type.getPerKmRate()) +
               (duration * type.getPerMinuteRate());
    }
}

public enum RideType {
    NORMAL(20, 10, 1),
    PREMIUM(30, 15, 2);

    private final double baseFare;
    private final double perKmRate;
    private final double perMinuteRate;

    RideType(double baseFare, double perKmRate, double perMinuteRate) {
        this.baseFare = baseFare;
        this.perKmRate = perKmRate;
        this.perMinuteRate = perMinuteRate;
    }

    public double getBaseFare() { return baseFare; }
    public double getPerKmRate() { return perKmRate; }
    public double getPerMinuteRate() { return perMinuteRate; }
}

public class Invoice {
    private final List<Ride> rides;
    private double totalFare;
    private double averageFarePerRide;
    private int totalRides;

    public void generateInvoice(List<Ride> rides) {
        this.rides = rides;
        this.totalRides = rides.size();
        this.totalFare = rides.stream().mapToDouble(Ride::calculateFare).sum();
        this.averageFarePerRide = totalFare / totalRides;
    }

    public void printInvoice() {
        System.out.println("Total Rides: " + totalRides);
        System.out.println("Total Fare: $" + totalFare);
        System.out.println("Average Fare: $" + averageFarePerRide);
    }
}
```

---

## Summary

Demonstrates: Fare calculation, invoice generation, pricing strategies.
