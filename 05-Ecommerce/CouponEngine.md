# Coupon/Discount Engine - Low Level Design

## Problem Statement

Design a coupon system with various discount types, validation rules, and application logic.

---

## Key Implementation

```java
public abstract class Coupon {
    protected final String code;
    protected final LocalDate validFrom;
    protected final LocalDate validTo;
    protected final double minOrderValue;
    protected final int usageLimit;
    protected int usageCount;

    public abstract double calculateDiscount(double orderAmount);

    public boolean isValid() {
        LocalDate now = LocalDate.now();
        return now.isAfter(validFrom) && now.isBefore(validTo) && usageCount < usageLimit;
    }

    public boolean canApply(double orderAmount) {
        return isValid() && orderAmount >= minOrderValue;
    }
}

public class PercentageCoupon extends Coupon {
    private final double percentage;
    private final double maxDiscount;

    @Override
    public double calculateDiscount(double orderAmount) {
        if (!canApply(orderAmount)) {
            return 0;
        }

        double discount = orderAmount * (percentage / 100.0);
        return Math.min(discount, maxDiscount);
    }
}

public class FixedAmountCoupon extends Coupon {
    private final double discountAmount;

    @Override
    public double calculateDiscount(double orderAmount) {
        if (!canApply(orderAmount)) {
            return 0;
        }

        return Math.min(discountAmount, orderAmount);
    }
}

public class BuyXGetYCoupon extends Coupon {
    private final int buyQuantity;
    private final int getQuantity;
    private final String productId;

    @Override
    public double calculateDiscount(double orderAmount) {
        // Calculate based on product quantity in cart
        return 0;  // Simplified
    }
}

public class CouponEngine {
    private final Map<String, Coupon> coupons;
    private final Map<String, Set<String>> userCoupons;

    public void createCoupon(Coupon coupon) {
        coupons.put(coupon.getCode(), coupon);
    }

    public double applyCoupon(String couponCode, double orderAmount, String userId) {
        Coupon coupon = coupons.get(couponCode);

        if (coupon == null) {
            throw new IllegalArgumentException("Invalid coupon code");
        }

        if (!coupon.canApply(orderAmount)) {
            throw new IllegalStateException("Coupon cannot be applied");
        }

        // Check if user already used this coupon
        if (hasUserUsedCoupon(userId, couponCode)) {
            throw new IllegalStateException("Coupon already used");
        }

        double discount = coupon.calculateDiscount(orderAmount);
        coupon.incrementUsage();
        markCouponUsed(userId, couponCode);

        return discount;
    }

    private boolean hasUserUsedCoupon(String userId, String couponCode) {
        return userCoupons.getOrDefault(userId, new HashSet<>()).contains(couponCode);
    }

    private void markCouponUsed(String userId, String couponCode) {
        userCoupons.computeIfAbsent(userId, k -> new HashSet<>()).add(couponCode);
    }
}
```

---

## Summary

Demonstrates: Strategy pattern for discount types, validation rules, usage tracking, percentage/fixed/BOGO discounts.
