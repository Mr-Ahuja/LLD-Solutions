# Hotel Management System - Low Level Design

## Problem Statement

Design a hotel management system that handles room bookings, guest check-in/check-out, room service, billing, and housekeeping management.

---

## Requirements

### Functional Requirements
1. Search available rooms by date, type, and occupancy
2. Book rooms with guest details
3. Check-in and check-out processing
4. Room service requests
5. Housekeeping status tracking
6. Bill generation with itemized charges
7. Payment processing

---

## Key Classes

```java
public enum RoomType {
    SINGLE(100.0),
    DOUBLE(150.0),
    SUITE(300.0),
    DELUXE(500.0);

    private final double basePrice;
    RoomType(double basePrice) { this.basePrice = basePrice; }
    public double getBasePrice() { return basePrice; }
}

public enum RoomStatus {
    AVAILABLE,
    OCCUPIED,
    MAINTENANCE,
    CLEANING
}

public class Room {
    private final String roomNumber;
    private final RoomType type;
    private final int floor;
    private final int maxOccupancy;
    private RoomStatus status;

    public Room(String roomNumber, RoomType type, int floor, int maxOccupancy) {
        this.roomNumber = roomNumber;
        this.type = type;
        this.floor = floor;
        this.maxOccupancy = maxOccupancy;
        this.status = RoomStatus.AVAILABLE;
    }

    public boolean isAvailable() {
        return status == RoomStatus.AVAILABLE;
    }

    public void occupy() {
        this.status = RoomStatus.OCCUPIED;
    }

    public void vacate() {
        this.status = RoomStatus.CLEANING;
    }

    public void markClean() {
        this.status = RoomStatus.AVAILABLE;
    }
}

public class Guest {
    private final String guestId;
    private final String name;
    private final String email;
    private final String phone;
    private final String address;

    public Guest(String guestId, String name, String email, String phone, String address) {
        this.guestId = guestId;
        this.name = name;
        this.email = email;
        this.phone = phone;
        this.address = address;
    }
}

public class Booking {
    private final String bookingId;
    private final Guest guest;
    private final Room room;
    private final LocalDate checkInDate;
    private final LocalDate checkOutDate;
    private BookingStatus status;
    private double totalAmount;

    public Booking(Guest guest, Room room, LocalDate checkIn, LocalDate checkOut) {
        this.bookingId = UUID.randomUUID().toString();
        this.guest = guest;
        this.room = room;
        this.checkInDate = checkIn;
        this.checkOutDate = checkOut;
        this.status = BookingStatus.CONFIRMED;
        this.totalAmount = calculateAmount();
    }

    private double calculateAmount() {
        long nights = ChronoUnit.DAYS.between(checkInDate, checkOutDate);
        return nights * room.getType().getBasePrice();
    }

    public void checkIn() {
        if (status != BookingStatus.CONFIRMED) {
            throw new IllegalStateException("Booking not confirmed");
        }
        room.occupy();
        status = BookingStatus.CHECKED_IN;
    }

    public void checkOut() {
        if (status != BookingStatus.CHECKED_IN) {
            throw new IllegalStateException("Not checked in");
        }
        room.vacate();
        status = BookingStatus.CHECKED_OUT;
    }

    public void cancel() {
        if (status == BookingStatus.CHECKED_IN) {
            throw new IllegalStateException("Cannot cancel after check-in");
        }
        status = BookingStatus.CANCELLED;
    }
}

public enum BookingStatus {
    CONFIRMED,
    CHECKED_IN,
    CHECKED_OUT,
    CANCELLED
}

public class RoomService {
    private final String serviceId;
    private final Room room;
    private final String itemName;
    private final double price;
    private final LocalDateTime orderTime;

    public RoomService(Room room, String itemName, double price) {
        this.serviceId = UUID.randomUUID().toString();
        this.room = room;
        this.itemName = itemName;
        this.price = price;
        this.orderTime = LocalDateTime.now();
    }
}

public class Bill {
    private final String billId;
    private final Booking booking;
    private final List<RoomService> services;
    private double roomCharges;
    private double serviceCharges;
    private double taxes;
    private double totalAmount;

    public Bill(Booking booking) {
        this.billId = UUID.randomUUID().toString();
        this.booking = booking;
        this.services = new ArrayList<>();
        this.roomCharges = booking.getTotalAmount();
        calculate();
    }

    public void addService(RoomService service) {
        services.add(service);
        calculate();
    }

    private void calculate() {
        serviceCharges = services.stream()
            .mapToDouble(RoomService::getPrice)
            .sum();

        double subtotal = roomCharges + serviceCharges;
        taxes = subtotal * 0.1;  // 10% tax
        totalAmount = subtotal + taxes;
    }

    public void printBill() {
        System.out.println("\n=== HOTEL BILL ===");
        System.out.println("Bill ID: " + billId);
        System.out.println("Guest: " + booking.getGuest().getName());
        System.out.println("Room: " + booking.getRoom().getRoomNumber());
        System.out.println("\nRoom Charges: $" + roomCharges);
        System.out.println("Service Charges: $" + serviceCharges);
        System.out.println("Taxes: $" + taxes);
        System.out.println("Total: $" + totalAmount);
    }
}

public class Hotel {
    private final String name;
    private final Map<String, Room> rooms;
    private final Map<String, Booking> bookings;
    private final List<Guest> guests;

    public Hotel(String name) {
        this.name = name;
        this.rooms = new ConcurrentHashMap<>();
        this.bookings = new ConcurrentHashMap<>();
        this.guests = new ArrayList<>();
    }

    public void addRoom(Room room) {
        rooms.put(room.getRoomNumber(), room);
    }

    public List<Room> searchAvailableRooms(LocalDate checkIn, LocalDate checkOut, RoomType type) {
        return rooms.values().stream()
            .filter(room -> room.getType() == type)
            .filter(room -> isRoomAvailable(room, checkIn, checkOut))
            .collect(Collectors.toList());
    }

    private boolean isRoomAvailable(Room room, LocalDate checkIn, LocalDate checkOut) {
        if (!room.isAvailable()) return false;

        // Check for overlapping bookings
        return bookings.values().stream()
            .filter(b -> b.getRoom().equals(room))
            .filter(b -> b.getStatus() != BookingStatus.CANCELLED)
            .noneMatch(b -> datesOverlap(checkIn, checkOut,
                                        b.getCheckInDate(), b.getCheckOutDate()));
    }

    private boolean datesOverlap(LocalDate start1, LocalDate end1,
                                  LocalDate start2, LocalDate end2) {
        return !start1.isAfter(end2) && !end1.isBefore(start2);
    }

    public Booking createBooking(Guest guest, Room room, LocalDate checkIn, LocalDate checkOut) {
        if (!isRoomAvailable(room, checkIn, checkOut)) {
            throw new IllegalStateException("Room not available");
        }

        Booking booking = new Booking(guest, room, checkIn, checkOut);
        bookings.put(booking.getBookingId(), booking);
        return booking;
    }

    public void checkIn(String bookingId) {
        Booking booking = bookings.get(bookingId);
        if (booking == null) {
            throw new IllegalArgumentException("Booking not found");
        }
        booking.checkIn();
        System.out.println("Check-in successful. Room: " + booking.getRoom().getRoomNumber());
    }

    public Bill checkOut(String bookingId) {
        Booking booking = bookings.get(bookingId);
        if (booking == null) {
            throw new IllegalArgumentException("Booking not found");
        }

        booking.checkOut();
        Bill bill = new Bill(booking);
        bill.printBill();
        return bill;
    }
}
```

---

## Summary

Hotel Management System demonstrates:
- **Booking Management**: Date-based availability checking
- **Room Status Tracking**: Lifecycle from available → occupied → cleaning
- **Bill Generation**: Itemized charges with taxes
- **Conflict Resolution**: Overlapping booking prevention
- **Service Integration**: Room service additions to bill
