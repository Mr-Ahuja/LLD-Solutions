# Low-Level Design (LLD) Solutions - Comprehensive Interview Preparation

A curated collection of 54 comprehensive Low-Level Design solutions with Java implementations, UML diagrams, and detailed design reasoning. Perfect for preparing for software engineering interviews at FAANG and top tech companies.

## Table of Contents
- [About This Repository](#about-this-repository)
- [How to Use](#how-to-use)
- [Problem Categories](#problem-categories)
- [Design Patterns Index](#design-patterns-index)
- [Difficulty Levels](#difficulty-levels)
- [Interview Company Tags](#interview-company-tags)

## About This Repository

This repository contains production-quality Low-Level Design solutions focusing on:

- **Object-Oriented Design Principles** (SOLID, DRY, KISS)
- **Design Patterns** (23 GoF patterns + architectural patterns)
- **Clean Code** (Java best practices, naming conventions)
- **System Thinking** (Scalability, extensibility, maintainability)
- **Visual Documentation** (Mermaid class & sequence diagrams)

### What Each Solution Includes

1. **Problem Statement** - Clear description with real-world context
2. **Requirements Analysis** - Functional, non-functional, and scope
3. **Class Diagrams** - Visual representation of system architecture
4. **Complete Java Implementation** - Production-ready code with comments
5. **Design Patterns** - Explanation of patterns used and why
6. **SOLID Principles** - How each principle is applied
7. **Workflow Diagrams** - Sequence diagrams for key operations
8. **Design Decisions** - Reasoning, trade-offs, alternatives
9. **Extensibility** - How to extend and enhance the system
10. **Complexity Analysis** - Time & space complexity of operations
11. **Testing Strategy** - Test cases, edge cases, unit test structure

## How to Use

### For Interview Preparation
1. **Start with Easy problems** - Build confidence with foundational concepts
2. **Practice Medium problems** - Most common interview difficulty
3. **Challenge with Hard problems** - Demonstrate advanced design skills

### For Learning
1. **Study by Category** - Master one domain at a time
2. **Study by Pattern** - Learn when and how to apply specific patterns
3. **Study by Principle** - Understand SOLID principles in practice

### Practice Approach
1. Try solving the problem yourself first
2. Compare your solution with the provided one
3. Understand the reasoning behind design decisions
4. Implement the code from scratch
5. Extend the system with additional features

---

## Problem Categories

### 01. Game-Oriented LLD (11 Problems)
Tests OOP fundamentals, design patterns, and rule-based systems.

| # | Problem | Difficulty | Key Patterns | Companies |
|---|---------|------------|--------------|-----------|
| 1 | [Tic Tac Toe](01-Games/TicTacToe.md) | Easy | Strategy, Factory | Amazon, Microsoft |
| 2 | [Snake and Ladder](01-Games/SnakeAndLadder.md) | Easy | Factory, Command | Flipkart, Google |
| 3 | [Connect Four](01-Games/ConnectFour.md) | Easy | Strategy | Meta, Apple |
| 4 | [Minesweeper](01-Games/Minesweeper.md) | Medium | Observer, State | Amazon, Google |
| 5 | [Sudoku Solver](01-Games/Sudoku.md) | Medium | Backtracking, Singleton | Adobe, Microsoft |
| 6 | [Ludo](01-Games/Ludo.md) | Medium | State, Command, Observer | Zomato, PhonePe |
| 7 | [Battleship](01-Games/Battleship.md) | Medium | Strategy, Observer | Amazon, Uber |
| 8 | [Chess](01-Games/Chess.md) | Hard | Factory, Strategy, Command | Google, Microsoft |
| 9 | [Card Game Engine](01-Games/CardGameEngine.md) | Medium | Abstract Factory, Template | Amazon, Salesforce |
| 10 | [Poker / Blackjack](01-Games/Poker.md) | Hard | Strategy, Chain of Responsibility | DraftKings, Zynga |
| 11 | [Cricket Scoreboard](01-Games/CricketScoreboard.md) | Medium | Observer, Singleton | Dream11, MPL |

### 02. System / Platform LLD (14 Problems)
Focus on real-world systems with complex entity relationships.

| # | Problem | Difficulty | Key Patterns | Companies |
|---|---------|------------|--------------|-----------|
| 1 | [Parking Lot](02-Systems/ParkingLot.md) | Medium | Factory, Strategy, Singleton | Amazon, Walmart |
| 2 | [Elevator System](02-Systems/ElevatorSystem.md) | Hard | State, Strategy, Observer | Microsoft, Uber |
| 3 | [Vending Machine](02-Systems/VendingMachine.md) | Medium | State, Factory | Amazon, Google |
| 4 | [ATM Machine](02-Systems/ATM.md) | Medium | State, Chain of Responsibility | Banks, Fintech |
| 5 | [Library Management](02-Systems/LibraryManagement.md) | Easy | Factory, Observer | Amazon, Flipkart |
| 6 | [Hotel Management](02-Systems/HotelManagement.md) | Medium | Factory, Strategy, Observer | Airbnb, OYO |
| 7 | [BookMyShow](02-Systems/BookMyShow.md) | Hard | Factory, Observer, Singleton | BookMyShow, Paytm |
| 8 | [Airline Reservation](02-Systems/AirlineReservation.md) | Hard | Factory, Strategy, Observer | Airlines, MakeMyTrip |
| 9 | [Ride Sharing (Uber/Ola)](02-Systems/RideSharing.md) | Hard | Strategy, Observer, Factory | Uber, Lyft, Ola |
| 10 | [Food Delivery](02-Systems/FoodDelivery.md) | Hard | Observer, Strategy, Factory | Zomato, Swiggy |
| 11 | [Splitwise](02-Systems/Splitwise.md) | Hard | Strategy, Observer | Splitwise, PayPal |
| 12 | [Cab Invoice Generator](02-Systems/CabInvoice.md) | Medium | Strategy, Factory | Uber, Ola |
| 13 | [Payment Wallet](02-Systems/PaymentWallet.md) | Medium | Singleton, Strategy, Observer | Paytm, PhonePe |
| 14 | [Online Payment Gateway](02-Systems/PaymentGateway.md) | Hard | Strategy, Chain of Responsibility | Stripe, Razorpay |

### 03. Social Media / Content Platforms (6 Problems)
Design systems with user interactions, feeds, and content management.

| # | Problem | Difficulty | Key Patterns | Companies |
|---|---------|------------|--------------|-----------|
| 1 | [Stack Overflow / Quora](03-SocialMedia/StackOverflow.md) | Hard | Observer, Strategy, Composite | Stack Overflow, Quora |
| 2 | [Twitter](03-SocialMedia/Twitter.md) | Hard | Observer, Strategy, Factory | Twitter, Meta |
| 3 | [WhatsApp / Messenger](03-SocialMedia/WhatsApp.md) | Hard | Observer, Singleton, Factory | Meta, WhatsApp |
| 4 | [Instagram](03-SocialMedia/Instagram.md) | Hard | Observer, Strategy, Composite | Meta, Instagram |
| 5 | [Netflix / YouTube](03-SocialMedia/Netflix.md) | Hard | Strategy, Decorator, Proxy | Netflix, Google |
| 6 | [Spotify / Music Player](03-SocialMedia/Spotify.md) | Medium | Strategy, Observer, Command | Spotify, Apple |

### 04. Utility-Based LLD (10 Problems)
Data structures, caching, and infrastructure components.

| # | Problem | Difficulty | Key Patterns | Companies |
|---|---------|------------|--------------|-----------|
| 1 | [LRU Cache](04-Utilities/LRUCache.md) | Medium | HashMap + DLL | Google, Amazon, Meta |
| 2 | [LFU Cache](04-Utilities/LFUCache.md) | Hard | HashMap + Priority Queue | Amazon, Google |
| 3 | [URL Shortener](04-Utilities/URLShortener.md) | Medium | Factory, Strategy | Bitly, Google |
| 4 | [Logging Framework](04-Utilities/LoggingFramework.md) | Medium | Singleton, Factory, Strategy | All Companies |
| 5 | [Rate Limiter](04-Utilities/RateLimiter.md) | Hard | Strategy, Decorator | Amazon, Stripe |
| 6 | [File System](04-Utilities/FileSystem.md) | Hard | Composite, Command | Google, Dropbox |
| 7 | [Key-Value Store](04-Utilities/KeyValueStore.md) | Hard | Strategy, Singleton | Redis, Amazon |
| 8 | [Pub-Sub System](04-Utilities/PubSubSystem.md) | Hard | Observer, Mediator | Kafka, RabbitMQ |
| 9 | [Notification Service](04-Utilities/NotificationService.md) | Medium | Strategy, Observer, Factory | All Companies |
| 10 | [Search Autocomplete](04-Utilities/SearchAutocomplete.md) | Medium | Trie, Strategy | Google, Amazon |

### 05. E-commerce / Business LLD (7 Problems)
Shopping, payments, and business workflow systems.

| # | Problem | Difficulty | Key Patterns | Companies |
|---|---------|------------|--------------|-----------|
| 1 | [Shopping Cart](05-Ecommerce/ShoppingCart.md) | Easy | Factory, Strategy, Observer | Amazon, Flipkart |
| 2 | [Order Management](05-Ecommerce/OrderManagement.md) | Medium | State, Observer, Factory | Amazon, Walmart |
| 3 | [Inventory Management](05-Ecommerce/InventoryManagement.md) | Medium | Observer, Singleton, Factory | Flipkart, Shopify |
| 4 | [Coupon / Discount Engine](05-Ecommerce/CouponEngine.md) | Medium | Strategy, Decorator, Chain | Amazon, Myntra |
| 5 | [Payment Gateway](05-Ecommerce/PaymentGateway.md) | Hard | Strategy, Chain of Responsibility | Stripe, PayPal |
| 6 | [Auction System](05-Ecommerce/AuctionSystem.md) | Hard | Observer, Strategy, State | eBay, Sotheby's |
| 7 | [Subscription Service](05-Ecommerce/SubscriptionService.md) | Medium | Strategy, State, Observer | Netflix, Spotify |

### 06. Analytics & Others (6 Problems)
Metrics, scheduling, and task management systems.

| # | Problem | Difficulty | Key Patterns | Companies |
|---|---------|------------|--------------|-----------|
| 1 | [Leaderboard System](06-Analytics/Leaderboard.md) | Medium | Strategy, Observer | Gaming Companies |
| 2 | [Job Scheduler](06-Analytics/JobScheduler.md) | Hard | Strategy, Command, Observer | Cron, Airflow |
| 3 | [Task Management (Trello/Jira)](06-Analytics/TaskManagement.md) | Hard | State, Observer, Composite | Atlassian, Asana |
| 4 | [News Feed System](06-Analytics/NewsFeed.md) | Hard | Strategy, Observer | Meta, LinkedIn |
| 5 | [Logging & Metrics Aggregator](06-Analytics/LoggingMetrics.md) | Hard | Observer, Strategy, Singleton | DataDog, Splunk |
| 6 | [Analytics Dashboard](06-Analytics/AnalyticsDashboard.md) | Medium | Observer, Strategy, Decorator | Tableau, PowerBI |

---

## Design Patterns Index

Find which problems demonstrate specific design patterns:

### Creational Patterns
- **Factory Method**: Parking Lot, BookMyShow, Chess, Card Games
- **Abstract Factory**: Card Game Engine, Payment Gateway
- **Builder**: Order Management, Query Builder
- **Singleton**: Logger, Database Connection Pool, Configuration Manager
- **Prototype**: Document Cloning

### Structural Patterns
- **Adapter**: Payment Gateway, Notification Service
- **Decorator**: Coupon Engine, Rate Limiter, Coffee Shop
- **Facade**: Complex subsystem interfaces
- **Composite**: File System, Organization Hierarchy
- **Proxy**: Virtual Proxy for heavy objects, Protection Proxy

### Behavioral Patterns
- **Strategy**: Pricing strategies, Sorting algorithms, Payment methods
- **Observer**: Event systems, Notifications, Pub-Sub
- **State**: Vending Machine, ATM, Order Status
- **Command**: Undo/Redo, Transaction Management
- **Chain of Responsibility**: ATM cash dispensing, Approval workflows
- **Template Method**: Game framework, Algorithm skeletons
- **Iterator**: Collection traversal
- **Mediator**: Chat systems, Air traffic control
- **Memento**: Undo functionality, Snapshots

---

## Difficulty Levels

### Easy (Beginner Friendly)
Good for understanding basic OOP principles and simple patterns.
- Tic Tac Toe
- Snake and Ladder
- Library Management
- Shopping Cart

### Medium (Interview Common)
Most frequently asked in interviews, tests multiple concepts.
- Parking Lot
- Hotel Management
- LRU Cache
- Vending Machine
- ATM
- Notification Service
- Leaderboard

### Hard (Advanced)
Complex systems requiring deep design thinking and multiple patterns.
- Chess
- Elevator System
- BookMyShow
- Splitwise
- Twitter/Instagram
- Rate Limiter
- Job Scheduler

---

## Interview Company Tags

### FAANG Companies
- **Amazon**: Parking Lot, LRU Cache, Shopping Cart, Locker System
- **Google**: Snake & Ladder, URL Shortener, Search Autocomplete, File System
- **Meta**: Instagram, WhatsApp, News Feed System
- **Netflix**: Video Streaming Platform, Subscription Service
- **Apple**: Music Player, Notification System

### Indian Unicorns
- **Flipkart**: E-commerce, Inventory Management, Order System
- **Zomato/Swiggy**: Food Delivery, Restaurant Management
- **Paytm/PhonePe**: Payment Wallet, UPI System
- **Ola/Uber**: Ride Sharing, Cab Booking
- **Dream11**: Cricket Scoreboard, Leaderboard

### Specialized Companies
- **Booking.com / Airbnb**: Hotel Management, Booking System
- **Atlassian**: Jira/Trello Task Management
- **Stripe**: Payment Gateway, Subscription Billing
- **Zoom**: Meeting Scheduler, Video Call System

---

## Learning Path Recommendations

### Week 1-2: Fundamentals
Start with easy problems to build OOP foundation
1. Tic Tac Toe → Learn Strategy Pattern
2. Library Management → Learn Factory & Observer
3. Shopping Cart → Learn basic e-commerce design
4. Parking Lot → Classic LLD problem

### Week 3-4: Intermediate Patterns
Tackle medium difficulty with focus on patterns
1. LRU Cache → Data structures + Design
2. Vending Machine → State Pattern mastery
3. Hotel Management → Booking system design
4. Notification Service → Multi-channel handling

### Week 5-6: Complex Systems
Advanced problems with multiple components
1. BookMyShow → Concurrent booking, transactions
2. Elevator System → Scheduling algorithms
3. Splitwise → Graph-based debt settlement
4. Rate Limiter → Distributed systems concept

### Week 7-8: Real-World Applications
Full-stack thinking with scalability
1. Twitter/Instagram → Social media architecture
2. Food Delivery → Multi-actor coordination
3. Payment Gateway → Security & reliability
4. Job Scheduler → Task orchestration

---

## Contributing

This is a learning resource. If you find improvements or want to add alternative solutions:
1. Focus on clear reasoning and educational value
2. Follow the existing template structure
3. Include proper diagrams and documentation
4. Test code for correctness

---

## Additional Resources

### Books
- "Head First Design Patterns" by Freeman & Robson
- "Design Patterns: Elements of Reusable Object-Oriented Software" by Gang of Four
- "Clean Code" by Robert C. Martin
- "Refactoring" by Martin Fowler

### Online Resources
- [Refactoring Guru - Design Patterns](https://refactoring.guru/design-patterns)
- [GeeksforGeeks LLD](https://www.geeksforgeeks.org/low-level-design-lld/)
- [GitHub LLD Repositories](https://github.com/topics/low-level-design)

### Practice Platforms
- LeetCode (Design Problems)
- InterviewBit (System Design)
- Educative.io (Grokking the Object Oriented Design Interview)

---

## License

This repository is for educational purposes. Feel free to use, modify, and share for learning.

---

## Quick Navigation

**[Games](#01-game-oriented-lld-11-problems)** | **[Systems](#02-system--platform-lld-14-problems)** | **[Social Media](#03-social-media--content-platforms-6-problems)** | **[Utilities](#04-utility-based-lld-10-problems)** | **[E-commerce](#05-e-commerce--business-lld-7-problems)** | **[Analytics](#06-analytics--others-6-problems)**

---

**Happy Learning! Good luck with your interviews!**
