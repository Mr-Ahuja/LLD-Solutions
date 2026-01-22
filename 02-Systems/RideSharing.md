# Ride Sharing System (Uber/Ola) - Low Level Design

## Problem Statement

Design a ride-sharing system with driver-rider matching, dynamic pricing, ride tracking, and payment processing.

---

## Key Classes

```java
public class Rider {
    private final String riderId;
    private final String name;
    private Location currentLocation;
    private double rating;
}

public class Driver {
    private final String driverId;
    private final String name;
    private final Vehicle vehicle;
    private Location currentLocation;
    private DriverStatus status;
    private double rating;

    public void updateLocation(Location location) {
        this.currentLocation = location;
    }
}

public enum DriverStatus {
    AVAILABLE,
    ON_RIDE,
    OFFLINE
}

public class Vehicle {
    private final String licensePlate;
    private final VehicleType type;
    private final String model;
    private final int capacity;
}

public enum VehicleType {
    BIKE, SEDAN, SUV, LUXURY
}

public class Location {
    private final double latitude;
    private final double longitude;

    public double distanceTo(Location other) {
        // Haversine formula
        return calculateDistance(this.latitude, this.longitude,
                                other.latitude, other.longitude);
    }
}

public class Ride {
    private final String rideId;
    private final Rider rider;
    private Driver driver;
    private final Location pickup;
    private final Location dropoff;
    private final RideType type;
    private RideStatus status;
    private double fare;
    private final LocalDateTime requestTime;
    private LocalDateTime startTime;
    private LocalDateTime endTime;

    public void assignDriver(Driver driver) {
        this.driver = driver;
        this.status = RideStatus.DRIVER_ASSIGNED;
    }

    public void startRide() {
        this.status = RideStatus.IN_PROGRESS;
        this.startTime = LocalDateTime.now();
    }

    public void completeRide() {
        this.status = RideStatus.COMPLETED;
        this.endTime = LocalDateTime.now();
        this.fare = calculateFare();
    }

    private double calculateFare() {
        double distance = pickup.distanceTo(dropoff);
        double baseFare = type.getBaseFare();
        double perKmRate = type.getPerKmRate();

        // Dynamic pricing
        double surgeFactor = getSurgeFactor();

        return (baseFare + distance * perKmRate) * surgeFactor;
    }

    private double getSurgeFactor() {
        // Simplified - could be based on demand/supply
        return 1.0;
    }
}

public enum RideStatus {
    REQUESTED,
    DRIVER_ASSIGNED,
    DRIVER_ARRIVED,
    IN_PROGRESS,
    COMPLETED,
    CANCELLED
}

public enum RideType {
    BIKE(20, 10),
    SEDAN(50, 15),
    SUV(80, 20),
    LUXURY(150, 30);

    private final double baseFare;
    private final double perKmRate;

    RideType(double baseFare, double perKmRate) {
        this.baseFare = baseFare;
        this.perKmRate = perKmRate;
    }

    public double getBaseFare() { return baseFare; }
    public double getPerKmRate() { return perKmRate; }
}

public class RideSharingSystem {
    private final Map<String, Driver> drivers;
    private final Map<String, Rider> riders;
    private final Map<String, Ride> activeRides;

    public Ride requestRide(String riderId, Location pickup, Location dropoff, RideType type) {
        Rider rider = riders.get(riderId);

        Ride ride = new Ride(rider, pickup, dropoff, type);

        // Find nearest available driver
        Driver driver = findNearestDriver(pickup, type);

        if (driver != null) {
            ride.assignDriver(driver);
            driver.setStatus(DriverStatus.ON_RIDE);
            activeRides.put(ride.getRideId(), ride);
            System.out.println("Ride assigned to driver: " + driver.getName());
        } else {
            System.out.println("No drivers available");
        }

        return ride;
    }

    private Driver findNearestDriver(Location pickup, RideType type) {
        return drivers.values().stream()
            .filter(d -> d.getStatus() == DriverStatus.AVAILABLE)
            .filter(d -> d.getVehicle().getType() == type.getCorrespondingVehicleType())
            .min(Comparator.comparingDouble(d ->
                d.getCurrentLocation().distanceTo(pickup)))
            .orElse(null);
    }

    public void completeRide(String rideId) {
        Ride ride = activeRides.get(rideId);
        if (ride != null) {
            ride.completeRide();
            ride.getDriver().setStatus(DriverStatus.AVAILABLE);
            System.out.println("Ride completed. Fare: $" + ride.getFare());
            activeRides.remove(rideId);
        }
    }

    public void cancelRide(String rideId) {
        Ride ride = activeRides.get(rideId);
        if (ride != null) {
            ride.cancel();
            if (ride.getDriver() != null) {
                ride.getDriver().setStatus(DriverStatus.AVAILABLE);
            }
            activeRides.remove(rideId);
        }
    }
}
```

---

## Summary

Demonstrates: Location-based matching, dynamic pricing, driver assignment, ride lifecycle, real-time tracking.
