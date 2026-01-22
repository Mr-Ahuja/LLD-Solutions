# Payment Gateway System - Low Level Design

## Problem Statement

Design a payment gateway that processes payments through multiple providers, handles retries, and manages transaction states.

---

## Key Classes

```java
public interface PaymentProvider {
    PaymentResponse processPayment(PaymentRequest request);
    boolean refund(String transactionId, double amount);
}

public class StripeProvider implements PaymentProvider {
    @Override
    public PaymentResponse processPayment(PaymentRequest request) {
        // Stripe API integration
        try {
            // Simulate processing
            return new PaymentResponse(true, generateTransactionId(), "Success");
        } catch (Exception e) {
            return new PaymentResponse(false, null, e.getMessage());
        }
    }

    @Override
    public boolean refund(String transactionId, double amount) {
        // Stripe refund API
        return true;
    }
}

public class RazorpayProvider implements PaymentProvider {
    @Override
    public PaymentResponse processPayment(PaymentRequest request) {
        // Razorpay API integration
        return new PaymentResponse(true, generateTransactionId(), "Success");
    }

    @Override
    public boolean refund(String transactionId, double amount) {
        return true;
    }
}

public class PaymentRequest {
    private final String orderId;
    private final double amount;
    private final String currency;
    private final PaymentMethod method;
    private final Map<String, String> metadata;
}

public class PaymentResponse {
    private final boolean success;
    private final String transactionId;
    private final String message;
}

public class PaymentTransaction {
    private final String transactionId;
    private final PaymentRequest request;
    private PaymentStatus status;
    private int retryCount;
    private final LocalDateTime createdAt;
    private LocalDateTime completedAt;

    public void updateStatus(PaymentStatus newStatus) {
        this.status = newStatus;
        if (newStatus == PaymentStatus.SUCCESS || newStatus == PaymentStatus.FAILED) {
            this.completedAt = LocalDateTime.now();
        }
    }
}

public enum PaymentStatus {
    PENDING,
    PROCESSING,
    SUCCESS,
    FAILED,
    REFUNDED
}

public class PaymentGateway {
    private final Map<String, PaymentProvider> providers;
    private final Map<String, PaymentTransaction> transactions;
    private final int maxRetries = 3;

    public PaymentGateway() {
        this.providers = new HashMap<>();
        this.transactions = new ConcurrentHashMap<>();

        // Register providers
        providers.put("STRIPE", new StripeProvider());
        providers.put("RAZORPAY", new RazorpayProvider());
    }

    public PaymentTransaction initiatePayment(PaymentRequest request, String providerName) {
        PaymentTransaction transaction = new PaymentTransaction(request);
        transactions.put(transaction.getTransactionId(), transaction);

        processPaymentWithRetry(transaction, providerName);

        return transaction;
    }

    private void processPaymentWithRetry(PaymentTransaction transaction, String providerName) {
        PaymentProvider provider = providers.get(providerName);

        if (provider == null) {
            transaction.updateStatus(PaymentStatus.FAILED);
            return;
        }

        transaction.updateStatus(PaymentStatus.PROCESSING);

        while (transaction.getRetryCount() < maxRetries) {
            try {
                PaymentResponse response = provider.processPayment(transaction.getRequest());

                if (response.isSuccess()) {
                    transaction.updateStatus(PaymentStatus.SUCCESS);
                    return;
                }

                transaction.incrementRetryCount();

                if (transaction.getRetryCount() >= maxRetries) {
                    transaction.updateStatus(PaymentStatus.FAILED);
                    return;
                }

                // Exponential backoff
                Thread.sleep(1000 * (long) Math.pow(2, transaction.getRetryCount()));

            } catch (Exception e) {
                transaction.incrementRetryCount();

                if (transaction.getRetryCount() >= maxRetries) {
                    transaction.updateStatus(PaymentStatus.FAILED);
                    return;
                }
            }
        }
    }

    public boolean refundPayment(String transactionId, double amount) {
        PaymentTransaction transaction = transactions.get(transactionId);

        if (transaction == null || transaction.getStatus() != PaymentStatus.SUCCESS) {
            return false;
        }

        // Find provider and process refund
        for (PaymentProvider provider : providers.values()) {
            if (provider.refund(transactionId, amount)) {
                transaction.updateStatus(PaymentStatus.REFUNDED);
                return true;
            }
        }

        return false;
    }

    public PaymentStatus getPaymentStatus(String transactionId) {
        PaymentTransaction transaction = transactions.get(transactionId);
        return transaction != null ? transaction.getStatus() : null;
    }
}
```

---

## Summary

Demonstrates: Multiple payment providers, retry logic with exponential backoff, transaction state management, refund processing.
