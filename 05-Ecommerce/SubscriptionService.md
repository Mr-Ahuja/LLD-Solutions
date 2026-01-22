# Subscription Service - Low Level Design

## Problem Statement

Design a subscription service with multiple tiers, billing cycles, payment processing, and automatic renewals.

---

## Key Implementation

```java
public class SubscriptionPlan {
    private final String planId;
    private final String name;
    private final double monthlyPrice;
    private final double annualPrice;
    private final List<String> features;
    private final int maxUsers;

    public double getPrice(BillingCycle cycle) {
        return cycle == BillingCycle.MONTHLY ? monthlyPrice : annualPrice;
    }
}

public enum BillingCycle {
    MONTHLY,
    ANNUAL
}

public class Subscription {
    private final String subscriptionId;
    private final User user;
    private final SubscriptionPlan plan;
    private final BillingCycle billingCycle;
    private SubscriptionStatus status;
    private LocalDate startDate;
    private LocalDate nextBillingDate;
    private LocalDate endDate;

    public void renew() {
        if (status != SubscriptionStatus.ACTIVE) {
            return;
        }

        boolean paymentSuccess = processPayment();

        if (paymentSuccess) {
            updateBillingDates();
        } else {
            status = SubscriptionStatus.PAYMENT_FAILED;
            // Retry logic or grace period
        }
    }

    private boolean processPayment() {
        double amount = plan.getPrice(billingCycle);
        // Process payment through gateway
        return true;  // Simplified
    }

    private void updateBillingDates() {
        if (billingCycle == BillingCycle.MONTHLY) {
            nextBillingDate = nextBillingDate.plusMonths(1);
        } else {
            nextBillingDate = nextBillingDate.plusYears(1);
        }
    }

    public void cancel() {
        status = SubscriptionStatus.CANCELLED;
        endDate = nextBillingDate;  // Cancel at end of billing period
    }

    public void pause() {
        status = SubscriptionStatus.PAUSED;
    }

    public void resume() {
        if (status == SubscriptionStatus.PAUSED) {
            status = SubscriptionStatus.ACTIVE;
        }
    }
}

public enum SubscriptionStatus {
    ACTIVE,
    CANCELLED,
    PAUSED,
    EXPIRED,
    PAYMENT_FAILED
}

public class SubscriptionService {
    private final Map<String, SubscriptionPlan> plans;
    private final Map<String, Subscription> subscriptions;

    public Subscription subscribe(User user, String planId, BillingCycle cycle) {
        SubscriptionPlan plan = plans.get(planId);

        Subscription subscription = new Subscription(user, plan, cycle);
        subscriptions.put(subscription.getSubscriptionId(), subscription);

        return subscription;
    }

    public void processRenewals() {
        LocalDate today = LocalDate.now();

        subscriptions.values().stream()
            .filter(s -> s.getStatus() == SubscriptionStatus.ACTIVE)
            .filter(s -> s.getNextBillingDate().equals(today))
            .forEach(Subscription::renew);
    }

    public void cancelSubscription(String subscriptionId) {
        Subscription subscription = subscriptions.get(subscriptionId);
        if (subscription != null) {
            subscription.cancel();
        }
    }
}
```

---

## Summary

Demonstrates: Subscription tiers, billing cycles, auto-renewal, payment retry, status management, pause/resume functionality.
