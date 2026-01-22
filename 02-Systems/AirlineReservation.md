# Airline Reservation System - Low Level Design

## Problem Statement

Design an airline reservation system for flight booking, seat selection, passenger management, and ticket generation.

---

## Key Classes

```java
public class Flight {
    private final String flightNumber;
    private final String airline;
    private final Airport source;
    private final Airport destination;
    private final LocalDateTime departureTime;
    private final LocalDateTime arrivalTime;
    private final Map<SeatClass, List<Seat>> seats;
    private final double basePrice;

    public List<Seat> getAvailableSeats(SeatClass seatClass) {
        return seats.get(seatClass).stream()
            .filter(Seat::isAvailable)
            .collect(Collectors.toList());
    }
}

public class Seat {
    private final String seatNumber;
    private final SeatClass seatClass;
    private SeatStatus status;
    private Passenger passenger;

    public synchronized boolean book(Passenger passenger) {
        if (status == SeatStatus.AVAILABLE) {
            this.passenger = passenger;
            this.status = SeatStatus.BOOKED;
            return true;
        }
        return false;
    }
}

public enum SeatClass {
    ECONOMY(1.0),
    PREMIUM_ECONOMY(1.5),
    BUSINESS(3.0),
    FIRST_CLASS(5.0);

    private final double priceMultiplier;
    SeatClass(double multiplier) { this.priceMultiplier = multiplier; }
}

public class Passenger {
    private final String passengerId;
    private final String name;
    private final String passportNumber;
    private final LocalDate dateOfBirth;
    private final String email;
}

public class Booking {
    private final String PNR;  // Passenger Name Record
    private final List<Passenger> passengers;
    private final Flight flight;
    private final List<Seat> seats;
    private final double totalFare;
    private BookingStatus status;

    public String generatePNR() {
        return "PNR" + System.currentTimeMillis();
    }

    public double calculateFare() {
        return seats.stream()
            .mapToDouble(s -> flight.getBasePrice() * s.getSeatClass().getPriceMultiplier())
            .sum();
    }
}

public class AirlineReservationSystem {
    private final Map<String, Flight> flights;
    private final Map<String, Booking> bookings;

    public List<Flight> searchFlights(String source, String dest, LocalDate date) {
        return flights.values().stream()
            .filter(f -> f.getSource().getCode().equals(source))
            .filter(f -> f.getDestination().getCode().equals(dest))
            .filter(f -> f.getDepartureTime().toLocalDate().equals(date))
            .collect(Collectors.toList());
    }

    public Booking bookFlight(Flight flight, List<Passenger> passengers, SeatClass seatClass) {
        List<Seat> availableSeats = flight.getAvailableSeats(seatClass);

        if (availableSeats.size() < passengers.size()) {
            throw new IllegalStateException("Insufficient seats");
        }

        List<Seat> selectedSeats = availableSeats.subList(0, passengers.size());

        for (int i = 0; i < passengers.size(); i++) {
            selectedSeats.get(i).book(passengers.get(i));
        }

        Booking booking = new Booking(passengers, flight, selectedSeats);
        bookings.put(booking.getPNR(), booking);
        return booking;
    }

    public void cancelBooking(String PNR) {
        Booking booking = bookings.get(PNR);
        if (booking != null) {
            booking.cancel();
            for (Seat seat : booking.getSeats()) {
                seat.release();
            }
        }
    }
}
```

---

## Summary

Demonstrates: Flight search, seat booking, PNR generation, fare calculation, multi-passenger booking.
