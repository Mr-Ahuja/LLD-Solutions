# BookMyShow (Movie Ticket Booking) - Low Level Design

## Problem Statement

Design a movie ticket booking system supporting theaters, movies, showtimes, seat selection, booking, and payment with concurrency handling.

---

## Requirements

### Functional Requirements
1. Browse movies by city, theater, language
2. View available showtimes
3. Select seats with real-time availability
4. Book tickets with payment
5. Cancel bookings (within time limit)
6. Prevent double-booking (concurrency control)

---

## Key Classes

```java
public class Movie {
    private final String id;
    private final String title;
    private final String language;
    private final Genre genre;
    private final int durationMinutes;
    private final double rating;
}

public class Theater {
    private final String id;
    private final String name;
    private final String city;
    private final List<Screen> screens;
    private final Address address;
}

public class Screen {
    private final String id;
    private final String name;
    private final int totalSeats;
    private final List<Seat> seats;
}

public class Seat {
    private final String seatNumber;
    private final SeatType type;
    private SeatStatus status;

    public synchronized boolean book() {
        if (status == SeatStatus.AVAILABLE) {
            status = SeatStatus.BOOKED;
            return true;
        }
        return false;
    }

    public synchronized boolean lock(String userId) {
        if (status == SeatStatus.AVAILABLE) {
            status = SeatStatus.LOCKED;
            // Start timer to auto-release after 10 minutes
            return true;
        }
        return false;
    }

    public synchronized void release() {
        if (status == SeatStatus.LOCKED || status == SeatStatus.BOOKED) {
            status = SeatStatus.AVAILABLE;
        }
    }
}

public enum SeatStatus {
    AVAILABLE,
    LOCKED,      // Temporarily locked during booking process
    BOOKED       // Confirmed booking
}

public enum SeatType {
    REGULAR(150.0),
    PREMIUM(250.0),
    VIP(400.0);

    private final double basePrice;
    SeatType(double basePrice) { this.basePrice = basePrice; }
    public double getBasePrice() { return basePrice; }
}

public class Show {
    private final String id;
    private final Movie movie;
    private final Screen screen;
    private final LocalDateTime showTime;
    private final Map<String, Seat> seats;

    public Show(Movie movie, Screen screen, LocalDateTime showTime) {
        this.id = UUID.randomUUID().toString();
        this.movie = movie;
        this.screen = screen;
        this.showTime = showTime;
        this.seats = new ConcurrentHashMap<>();
        initializeSeats();
    }

    private void initializeSeats() {
        for (Seat seat : screen.getSeats()) {
            seats.put(seat.getSeatNumber(), new Seat(
                seat.getSeatNumber(),
                seat.getType(),
                SeatStatus.AVAILABLE
            ));
        }
    }

    public List<Seat> getAvailableSeats() {
        return seats.values().stream()
            .filter(s -> s.getStatus() == SeatStatus.AVAILABLE)
            .collect(Collectors.toList());
    }

    public synchronized boolean lockSeats(List<String> seatNumbers, String userId) {
        // Check all seats available
        for (String seatNum : seatNumbers) {
            Seat seat = seats.get(seatNum);
            if (seat == null || !seat.lock(userId)) {
                // Rollback
                unlockSeats(seatNumbers, userId);
                return false;
            }
        }
        return true;
    }

    public void unlockSeats(List<String> seatNumbers, String userId) {
        for (String seatNum : seatNumbers) {
            Seat seat = seats.get(seatNum);
            if (seat != null) {
                seat.release();
            }
        }
    }

    public synchronized boolean confirmBooking(List<String> seatNumbers) {
        for (String seatNum : seatNumbers) {
            Seat seat = seats.get(seatNum);
            if (seat == null || !seat.book()) {
                return false;
            }
        }
        return true;
    }
}

public class Booking {
    private final String bookingId;
    private final User user;
    private final Show show;
    private final List<Seat> seats;
    private final double totalAmount;
    private final LocalDateTime bookingTime;
    private BookingStatus status;

    public Booking(User user, Show show, List<Seat> seats) {
        this.bookingId = generateBookingId();
        this.user = user;
        this.show = show;
        this.seats = seats;
        this.totalAmount = calculateAmount();
        this.bookingTime = LocalDateTime.now();
        this.status = BookingStatus.PENDING;
    }

    private double calculateAmount() {
        return seats.stream()
            .mapToDouble(s -> s.getType().getBasePrice())
            .sum();
    }

    private String generateBookingId() {
        return "BMS" + System.currentTimeMillis();
    }

    public void confirm() {
        this.status = BookingStatus.CONFIRMED;
    }

    public void cancel() {
        if (canCancel()) {
            this.status = BookingStatus.CANCELLED;
            // Release seats
            for (Seat seat : seats) {
                seat.release();
            }
        } else {
            throw new IllegalStateException("Cannot cancel - show time passed");
        }
    }

    private boolean canCancel() {
        // Allow cancellation up to 1 hour before show
        return LocalDateTime.now().plusHours(1).isBefore(show.getShowTime());
    }
}

public enum BookingStatus {
    PENDING,
    CONFIRMED,
    CANCELLED,
    EXPIRED
}

public class Payment {
    private final String paymentId;
    private final Booking booking;
    private final PaymentMethod method;
    private final double amount;
    private PaymentStatus status;

    public Payment(Booking booking, PaymentMethod method) {
        this.paymentId = UUID.randomUUID().toString();
        this.booking = booking;
        this.method = method;
        this.amount = booking.getTotalAmount();
        this.status = PaymentStatus.PENDING;
    }

    public boolean process() {
        // Simulate payment gateway
        try {
            // Payment processing logic
            this.status = PaymentStatus.SUCCESS;
            booking.confirm();
            return true;
        } catch (Exception e) {
            this.status = PaymentStatus.FAILED;
            return false;
        }
    }
}

public enum PaymentMethod {
    CREDIT_CARD,
    DEBIT_CARD,
    UPI,
    NET_BANKING
}

public enum PaymentStatus {
    PENDING,
    SUCCESS,
    FAILED,
    REFUNDED
}

public class BookMyShowSystem {
    private final Map<String, Movie> movies;
    private final Map<String, Theater> theaters;
    private final Map<String, Show> shows;
    private final Map<String, Booking> bookings;

    public BookMyShowSystem() {
        this.movies = new ConcurrentHashMap<>();
        this.theaters = new ConcurrentHashMap<>();
        this.shows = new ConcurrentHashMap<>();
        this.bookings = new ConcurrentHashMap<>();
    }

    public List<Show> searchShows(String city, String movieId, LocalDate date) {
        return shows.values().stream()
            .filter(s -> s.getMovie().getId().equals(movieId))
            .filter(s -> s.getScreen().getTheater().getCity().equalsIgnoreCase(city))
            .filter(s -> s.getShowTime().toLocalDate().equals(date))
            .collect(Collectors.toList());
    }

    public Booking bookSeats(String userId, String showId, List<String> seatNumbers) {
        Show show = shows.get(showId);
        if (show == null) {
            throw new IllegalArgumentException("Show not found");
        }

        // Lock seats
        if (!show.lockSeats(seatNumbers, userId)) {
            throw new IllegalStateException("Seats not available");
        }

        try {
            // Create booking
            List<Seat> seats = seatNumbers.stream()
                .map(show::getSeat)
                .collect(Collectors.toList());

            User user = getUserById(userId);
            Booking booking = new Booking(user, show, seats);
            bookings.put(booking.getBookingId(), booking);

            // Payment process (10 min window)
            return booking;
        } catch (Exception e) {
            // Release seats on error
            show.unlockSeats(seatNumbers, userId);
            throw e;
        }
    }

    public boolean confirmBooking(String bookingId, PaymentMethod paymentMethod) {
        Booking booking = bookings.get(bookingId);
        if (booking == null) {
            return false;
        }

        Payment payment = new Payment(booking, paymentMethod);
        if (payment.process()) {
            Show show = booking.getShow();
            List<String> seatNumbers = booking.getSeats().stream()
                .map(Seat::getSeatNumber)
                .collect(Collectors.toList());

            show.confirmBooking(seatNumbers);
            System.out.println("Booking confirmed: " + bookingId);
            return true;
        }

        return false;
    }

    public void cancelBooking(String bookingId) {
        Booking booking = bookings.get(bookingId);
        if (booking != null) {
            booking.cancel();
            System.out.println("Booking cancelled: " + bookingId);
        }
    }
}
```

---

## Design Decisions

### 1. **Seat Locking Mechanism**
**Decision**: Three-state seat status (AVAILABLE → LOCKED → BOOKED)
**Reasoning**:
- LOCKED: Temporary hold during payment (10 min)
- Prevents double-booking during checkout
- Auto-expires if payment not completed

### 2. **Synchronized Seat Operations**
**Decision**: `synchronized` on seat lock/book methods
**Reasoning**:
- Prevents race conditions in concurrent bookings
- Ensures atomicity of seat state changes
- Critical for data consistency

### 3. **Rollback on Partial Failure**
**Decision**: If any seat unavailable, release all locked seats
**Reasoning**:
- All-or-nothing booking semantics
- Prevents partial bookings
- Clean error handling

---

## Summary

BookMyShow demonstrates:
- **Concurrency Control**: Thread-safe seat booking
- **State Management**: Seat locking and booking flow
- **Transaction Handling**: Payment integration
- **Real-Time Availability**: Concurrent seat selection
- **Business Rules**: Cancellation policies, time-based logic
