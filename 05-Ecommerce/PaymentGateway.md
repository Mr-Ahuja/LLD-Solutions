# Payment Gateway - Low Level Design

## Problem Statement

Design a payment gateway integrating multiple payment providers with retry logic and transaction management.

---

## Key Implementation

```java
public interface PaymentProvider {
    PaymentResponse process(PaymentRequest request);
}

public class StripeProvider implements PaymentProvider {
    @Override
    public PaymentResponse process(PaymentRequest request) {
        // Stripe API integration
        return new PaymentResponse(true, generateTransactionId());
    }
}

public class PaymentRequest {
    private final String orderId;
    private final double amount;
    private final String currency;
    private final PaymentMethod method;
}

public class PaymentResponse {
    private final boolean success;
    private final String transactionId;
    private final String message;
}

public class PaymentGateway {
    private final Map<String, PaymentProvider> providers;
    private final int maxRetries = 3;

    public PaymentTransaction processPayment(PaymentRequest request, String providerName) {
        PaymentTransaction transaction = new PaymentTransaction(request);

        for (int attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                PaymentProvider provider = providers.get(providerName);
                PaymentResponse response = provider.process(request);

                if (response.isSuccess()) {
                    transaction.setStatus(PaymentStatus.SUCCESS);
                    return transaction;
                }

                Thread.sleep(1000 * attempt);  // Exponential backoff
            } catch (Exception e) {
                if (attempt == maxRetries) {
                    transaction.setStatus(PaymentStatus.FAILED);
                }
            }
        }

        return transaction;
    }
}
```

---

## Summary

Demonstrates: Multiple payment providers, retry with exponential backoff, transaction state management.
