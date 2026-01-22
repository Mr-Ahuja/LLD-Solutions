# Library Management System - Low Level Design

## Problem Statement

Design a library management system that handles book cataloging, member registration, book borrowing/returning, fines, and reservations.

---

## Requirements

### Functional Requirements
1. Add/remove books and members
2. Search books (title, author, ISBN)
3. Borrow and return books
4. Reserve books
5. Calculate fines for overdue books
6. Track borrowing history

### Non-Functional Requirements
1. Concurrent borrowing support
2. Efficient search (indexes)
3. Fine calculation automation
4. Reservation queue management

---

## Key Classes

```java
public class Book {
    private final String ISBN;
    private final String title;
    private final List<String> authors;
    private final String publisher;
    private int totalCopies;
    private int availableCopies;

    public synchronized boolean borrow() {
        if (availableCopies > 0) {
            availableCopies--;
            return true;
        }
        return false;
    }

    public synchronized void returnBook() {
        if (availableCopies < totalCopies) {
            availableCopies++;
        }
    }

    public boolean isAvailable() {
        return availableCopies > 0;
    }
}

public class Member {
    private final String memberId;
    private final String name;
    private final String email;
    private final List<BookLoan> activeLoans;
    private final List<BookLoan> loanHistory;
    private double totalFines;

    public boolean canBorrow() {
        return activeLoans.size() < 5 && totalFines == 0;
    }

    public void borrowBook(Book book, LocalDate dueDate) {
        if (!canBorrow()) {
            throw new IllegalStateException("Cannot borrow - limit reached or fines pending");
        }
        activeLoans.add(new BookLoan(book, this, dueDate));
    }

    public void returnBook(Book book) {
        BookLoan loan = activeLoans.stream()
            .filter(l -> l.getBook().equals(book))
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("Book not borrowed"));

        loan.markReturned();
        activeLoans.remove(loan);
        loanHistory.add(loan);

        double fine = loan.calculateFine();
        if (fine > 0) {
            totalFines += fine;
            System.out.println("Fine: $" + fine);
        }
    }

    public void payFine(double amount) {
        totalFines = Math.max(0, totalFines - amount);
    }
}

public class BookLoan {
    private final Book book;
    private final Member member;
    private final LocalDate borrowDate;
    private final LocalDate dueDate;
    private LocalDate returnDate;

    private static final double FINE_PER_DAY = 0.50;

    public BookLoan(Book book, Member member, LocalDate dueDate) {
        this.book = book;
        this.member = member;
        this.borrowDate = LocalDate.now();
        this.dueDate = dueDate;
    }

    public void markReturned() {
        this.returnDate = LocalDate.now();
    }

    public double calculateFine() {
        if (returnDate == null || !returnDate.isAfter(dueDate)) {
            return 0;
        }
        long daysOverdue = ChronoUnit.DAYS.between(dueDate, returnDate);
        return daysOverdue * FINE_PER_DAY;
    }

    public boolean isOverdue() {
        return LocalDate.now().isAfter(dueDate) && returnDate == null;
    }
}

public class BookReservation {
    private final Book book;
    private final Member member;
    private final LocalDateTime reservationTime;
    private ReservationStatus status;

    public BookReservation(Book book, Member member) {
        this.book = book;
        this.member = member;
        this.reservationTime = LocalDateTime.now();
        this.status = ReservationStatus.WAITING;
    }

    public void fulfill() {
        status = ReservationStatus.FULFILLED;
    }

    public void cancel() {
        status = ReservationStatus.CANCELLED;
    }
}

public enum ReservationStatus {
    WAITING,
    FULFILLED,
    CANCELLED,
    EXPIRED
}

public class Library {
    private final Map<String, Book> booksByISBN;
    private final Map<String, Member> membersById;
    private final Map<String, Queue<BookReservation>> reservations;

    public Library() {
        this.booksByISBN = new ConcurrentHashMap<>();
        this.membersById = new ConcurrentHashMap<>();
        this.reservations = new ConcurrentHashMap<>();
    }

    public void addBook(Book book) {
        booksByISBN.put(book.getISBN(), book);
    }

    public void registerMember(Member member) {
        membersById.put(member.getMemberId(), member);
    }

    public synchronized boolean borrowBook(String memberId, String ISBN) {
        Member member = membersById.get(memberId);
        Book book = booksByISBN.get(ISBN);

        if (member == null || book == null) {
            return false;
        }

        if (!member.canBorrow()) {
            System.out.println("Member cannot borrow");
            return false;
        }

        if (book.borrow()) {
            LocalDate dueDate = LocalDate.now().plusWeeks(2);
            member.borrowBook(book, dueDate);
            System.out.println("Book borrowed. Due: " + dueDate);
            return true;
        } else {
            System.out.println("Book not available");
            return false;
        }
    }

    public synchronized void returnBook(String memberId, String ISBN) {
        Member member = membersById.get(memberId);
        Book book = booksByISBN.get(ISBN);

        if (member == null || book == null) {
            throw new IllegalArgumentException("Invalid member or book");
        }

        member.returnBook(book);
        book.returnBook();

        // Check reservations
        processReservation(ISBN);
    }

    public void reserveBook(String memberId, String ISBN) {
        Member member = membersById.get(memberId);
        Book book = booksByISBN.get(ISBN);

        if (member == null || book == null) {
            throw new IllegalArgumentException("Invalid member or book");
        }

        reservations.putIfAbsent(ISBN, new LinkedList<>());
        reservations.get(ISBN).offer(new BookReservation(book, member));
        System.out.println("Book reserved");
    }

    private void processReservation(String ISBN) {
        Queue<BookReservation> queue = reservations.get(ISBN);
        if (queue != null && !queue.isEmpty()) {
            BookReservation reservation = queue.poll();
            reservation.fulfill();
            System.out.println("Reservation fulfilled for " + reservation.getMember().getName());
        }
    }

    public List<Book> searchByTitle(String title) {
        return booksByISBN.values().stream()
            .filter(book -> book.getTitle().toLowerCase().contains(title.toLowerCase()))
            .collect(Collectors.toList());
    }

    public List<Book> searchByAuthor(String author) {
        return booksByISBN.values().stream()
            .filter(book -> book.getAuthors().stream()
                .anyMatch(a -> a.toLowerCase().contains(author.toLowerCase())))
            .collect(Collectors.toList());
    }
}
```

---

## Summary

Library Management System demonstrates:
- **Concurrent Borrowing**: Thread-safe book availability management
- **Fine Calculation**: Automatic overdue fine computation
- **Reservation Queue**: FIFO booking system
- **Search Functionality**: Multiple search criteria
- **Transaction History**: Loan tracking
