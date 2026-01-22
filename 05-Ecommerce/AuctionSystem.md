# Auction System (eBay-like) - Low Level Design

## Problem Statement

Design an online auction system with bidding, time-based auctions, automatic winner selection, and notifications.

---

## Key Implementation

```java
public class AuctionItem {
    private final String itemId;
    private final String title;
    private final double startingPrice;
    private final LocalDateTime startTime;
    private final LocalDateTime endTime;
    private double currentBid;
    private User currentWinner;
    private AuctionStatus status;
    private final List<Bid> bidHistory;

    public synchronized boolean placeBid(User bidder, double amount) {
        if (status != AuctionStatus.ACTIVE) {
            return false;
        }

        if (LocalDateTime.now().isAfter(endTime)) {
            endAuction();
            return false;
        }

        if (amount <= currentBid) {
            return false;  // Bid must be higher
        }

        Bid bid = new Bid(bidder, amount);
        bidHistory.add(bid);
        currentBid = amount;
        currentWinner = bidder;

        return true;
    }

    public void endAuction() {
        this.status = AuctionStatus.ENDED;
        if (currentWinner != null) {
            notifyWinner();
        }
    }

    private void notifyWinner() {
        System.out.println("Winner: " + currentWinner.getName() + " - $" + currentBid);
    }
}

public class Bid {
    private final User bidder;
    private final double amount;
    private final LocalDateTime timestamp;

    public Bid(User bidder, double amount) {
        this.bidder = bidder;
        this.amount = amount;
        this.timestamp = LocalDateTime.now();
    }
}

public enum AuctionStatus {
    PENDING,
    ACTIVE,
    ENDED,
    CANCELLED
}

public class AuctionSystem {
    private final Map<String, AuctionItem> auctions;

    public void createAuction(AuctionItem item) {
        auctions.put(item.getItemId(), item);
        scheduleAuctionEnd(item);
    }

    public boolean placeBid(String itemId, User bidder, double amount) {
        AuctionItem item = auctions.get(itemId);
        return item != null && item.placeBid(bidder, amount);
    }

    private void scheduleAuctionEnd(AuctionItem item) {
        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                item.endAuction();
            }
        }, Date.from(item.getEndTime().atZone(ZoneId.systemDefault()).toInstant()));
    }
}
```

---

## Summary

Demonstrates: Time-based auctions, bid validation, winner selection, automatic auction ending, bid history.
