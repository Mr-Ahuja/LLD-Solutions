# Payment Wallet System (Paytm/PhonePe) - Low Level Design

## Problem Statement

Design a digital wallet system supporting balance management, transactions, peer-to-peer transfers, and transaction history.

---

## Key Classes

```java
public class Wallet {
    private final String walletId;
    private final User owner;
    private double balance;
    private final List<Transaction> transactions;

    public synchronized boolean debit(double amount) {
        if (amount > balance) {
            return false;
        }
        balance -= amount;
        recordTransaction(TransactionType.DEBIT, amount);
        return true;
    }

    public synchronized void credit(double amount) {
        balance += amount;
        recordTransaction(TransactionType.CREDIT, amount);
    }

    private void recordTransaction(TransactionType type, double amount) {
        transactions.add(new Transaction(type, amount, balance));
    }

    public List<Transaction> getTransactionHistory() {
        return new ArrayList<>(transactions);
    }
}

public class Transaction {
    private final String transactionId;
    private final TransactionType type;
    private final double amount;
    private final double balanceAfter;
    private final LocalDateTime timestamp;
    private TransactionStatus status;

    public Transaction(TransactionType type, double amount, double balanceAfter) {
        this.transactionId = UUID.randomUUID().toString();
        this.type = type;
        this.amount = amount;
        this.balanceAfter = balanceAfter;
        this.timestamp = LocalDateTime.now();
        this.status = TransactionStatus.SUCCESS;
    }
}

public enum TransactionType {
    CREDIT,
    DEBIT,
    TRANSFER_IN,
    TRANSFER_OUT,
    REFUND
}

public enum TransactionStatus {
    PENDING,
    SUCCESS,
    FAILED,
    REVERSED
}

public class WalletSystem {
    private final Map<String, Wallet> wallets;

    public synchronized boolean transfer(String fromWalletId, String toWalletId, double amount) {
        Wallet fromWallet = wallets.get(fromWalletId);
        Wallet toWallet = wallets.get(toWalletId);

        if (fromWallet == null || toWallet == null) {
            return false;
        }

        if (fromWallet.debit(amount)) {
            toWallet.credit(amount);
            System.out.println("Transfer successful: $" + amount);
            return true;
        }

        System.out.println("Insufficient balance");
        return false;
    }

    public void addMoney(String walletId, double amount, PaymentMethod method) {
        Wallet wallet = wallets.get(walletId);
        if (wallet != null) {
            // Process payment through gateway
            wallet.credit(amount);
            System.out.println("Added $" + amount + " via " + method);
        }
    }
}

public enum PaymentMethod {
    CREDIT_CARD,
    DEBIT_CARD,
    NET_BANKING,
    UPI
}
```

---

## Summary

Demonstrates: Wallet balance management, P2P transfers, transaction logging, thread-safe operations.
