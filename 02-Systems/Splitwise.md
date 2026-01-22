# Splitwise (Expense Sharing) - Low Level Design

## Problem Statement

Design an expense-sharing application that tracks shared expenses, calculates balances, and suggests optimal settlements.

---

## Key Classes

```java
public class User {
    private final String userId;
    private final String name;
    private final String email;
    private final Map<String, Double> balances;  // userId -> amount (positive = owed to me)

    public void updateBalance(String otherUserId, double amount) {
        balances.put(otherUserId, balances.getOrDefault(otherUserId, 0.0) + amount);
    }

    public double getBalanceWith(String otherUserId) {
        return balances.getOrDefault(otherUserId, 0.0);
    }
}

public class Expense {
    private final String expenseId;
    private final String description;
    private final double totalAmount;
    private final User paidBy;
    private final List<Split> splits;
    private final LocalDateTime createdAt;

    public void addSplit(Split split) {
        splits.add(split);
    }

    public void validateSplits() {
        double totalSplit = splits.stream().mapToDouble(Split::getAmount).sum();
        if (Math.abs(totalSplit - totalAmount) > 0.01) {
            throw new IllegalStateException("Splits don't match total amount");
        }
    }
}

public abstract class Split {
    private final User user;
    protected double amount;

    public abstract void calculateAmount(double totalAmount, int totalUsers);

    public double getAmount() {
        return amount;
    }

    public User getUser() {
        return user;
    }
}

public class EqualSplit extends Split {
    @Override
    public void calculateAmount(double totalAmount, int totalUsers) {
        this.amount = totalAmount / totalUsers;
    }
}

public class ExactSplit extends Split {
    public ExactSplit(User user, double amount) {
        super(user);
        this.amount = amount;
    }

    @Override
    public void calculateAmount(double totalAmount, int totalUsers) {
        // Amount already set
    }
}

public class PercentSplit extends Split {
    private final double percentage;

    public PercentSplit(User user, double percentage) {
        super(user);
        this.percentage = percentage;
    }

    @Override
    public void calculateAmount(double totalAmount, int totalUsers) {
        this.amount = (totalAmount * percentage) / 100.0;
    }
}

public class Group {
    private final String groupId;
    private final String name;
    private final List<User> members;
    private final List<Expense> expenses;

    public void addExpense(Expense expense) {
        expense.validateSplits();

        User paidBy = expense.getPaidBy();

        for (Split split : expense.getSplits()) {
            User user = split.getUser();
            double amount = split.getAmount();

            if (!user.equals(paidBy)) {
                // User owes paidBy this amount
                user.updateBalance(paidBy.getUserId(), -amount);
                paidBy.updateBalance(user.getUserId(), amount);
            }
        }

        expenses.add(expense);
    }

    public Map<String, Double> getBalances() {
        Map<String, Double> balances = new HashMap<>();

        for (User user : members) {
            double totalBalance = user.getBalances().values().stream()
                .mapToDouble(Double::doubleValue)
                .sum();
            balances.put(user.getName(), totalBalance);
        }

        return balances;
    }
}

public class Settlement {
    private final User from;
    private final User to;
    private final double amount;

    public Settlement(User from, User to, double amount) {
        this.from = from;
        this.to = to;
        this.amount = amount;
    }

    @Override
    public String toString() {
        return from.getName() + " pays " + to.getName() + ": $" + String.format("%.2f", amount);
    }
}

public class SplitwiseSystem {
    private final Map<String, User> users;
    private final Map<String, Group> groups;
    private final List<Expense> expenses;

    public SplitwiseSystem() {
        this.users = new ConcurrentHashMap<>();
        this.groups = new ConcurrentHashMap<>();
        this.expenses = new ArrayList<>();
    }

    public void addEqualExpense(String paidById, List<String> participantIds,
                                 double amount, String description) {
        User paidBy = users.get(paidById);
        List<User> participants = participantIds.stream()
            .map(users::get)
            .collect(Collectors.toList());

        Expense expense = new Expense(description, amount, paidBy);

        for (User user : participants) {
            Split split = new EqualSplit(user);
            split.calculateAmount(amount, participants.size());
            expense.addSplit(split);
        }

        processExpense(expense);
    }

    public void addExactExpense(String paidById, Map<String, Double> splits,
                                String description) {
        User paidBy = users.get(paidById);
        double totalAmount = splits.values().stream().mapToDouble(Double::doubleValue).sum();

        Expense expense = new Expense(description, totalAmount, paidBy);

        for (Map.Entry<String, Double> entry : splits.entrySet()) {
            User user = users.get(entry.getKey());
            expense.addSplit(new ExactSplit(user, entry.getValue()));
        }

        processExpense(expense);
    }

    private void processExpense(Expense expense) {
        expense.validateSplits();
        User paidBy = expense.getPaidBy();

        for (Split split : expense.getSplits()) {
            User user = split.getUser();
            double amount = split.getAmount();

            if (!user.equals(paidBy)) {
                user.updateBalance(paidBy.getUserId(), -amount);
                paidBy.updateBalance(user.getUserId(), amount);
            }
        }

        expenses.add(expense);
    }

    public List<Settlement> simplifyDebts() {
        // Calculate net balances
        Map<String, Double> netBalances = new HashMap<>();

        for (User user : users.values()) {
            double netBalance = user.getBalances().values().stream()
                .mapToDouble(Double::doubleValue)
                .sum();
            if (Math.abs(netBalance) > 0.01) {
                netBalances.put(user.getUserId(), netBalance);
            }
        }

        // Greedy approach: match largest debtor with largest creditor
        List<Settlement> settlements = new ArrayList<>();

        while (!netBalances.isEmpty()) {
            String maxCreditor = netBalances.entrySet().stream()
                .max(Map.Entry.comparingByValue())
                .map(Map.Entry::getKey)
                .orElse(null);

            String maxDebtor = netBalances.entrySet().stream()
                .min(Map.Entry.comparingByValue())
                .map(Map.Entry::getKey)
                .orElse(null);

            if (maxCreditor == null || maxDebtor == null) break;

            double creditAmount = netBalances.get(maxCreditor);
            double debtAmount = Math.abs(netBalances.get(maxDebtor));

            double settleAmount = Math.min(creditAmount, debtAmount);

            settlements.add(new Settlement(
                users.get(maxDebtor),
                users.get(maxCreditor),
                settleAmount
            ));

            netBalances.put(maxCreditor, creditAmount - settleAmount);
            netBalances.put(maxDebtor, netBalances.get(maxDebtor) + settleAmount);

            netBalances.entrySet().removeIf(e -> Math.abs(e.getValue()) < 0.01);
        }

        return settlements;
    }

    public void showBalances(String userId) {
        User user = users.get(userId);
        System.out.println("\n=== Balances for " + user.getName() + " ===");

        for (Map.Entry<String, Double> entry : user.getBalances().entrySet()) {
            User otherUser = users.get(entry.getKey());
            double amount = entry.getValue();

            if (Math.abs(amount) > 0.01) {
                if (amount > 0) {
                    System.out.println(otherUser.getName() + " owes you: $" +
                                     String.format("%.2f", amount));
                } else {
                    System.out.println("You owe " + otherUser.getName() + ": $" +
                                     String.format("%.2f", Math.abs(amount)));
                }
            }
        }
    }
}
```

---

## Summary

Demonstrates: Expense splitting strategies (equal, exact, percent), balance tracking, debt simplification algorithm, group management.
