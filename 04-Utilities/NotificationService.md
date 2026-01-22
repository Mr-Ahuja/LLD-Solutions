# Notification Service - Low Level Design

## Problem Statement

Design a notification service supporting multiple channels (email, SMS, push) with retry logic and templates.

---

## Key Implementation

```java
public interface NotificationChannel {
    boolean send(Notification notification);
}

public class EmailChannel implements NotificationChannel {
    @Override
    public boolean send(Notification notification) {
        System.out.println("Sending email to: " + notification.getRecipient());
        // Email sending logic
        return true;
    }
}

public class SMSChannel implements NotificationChannel {
    @Override
    public boolean send(Notification notification) {
        System.out.println("Sending SMS to: " + notification.getRecipient());
        // SMS sending logic
        return true;
    }
}

public class PushChannel implements NotificationChannel {
    @Override
    public boolean send(Notification notification) {
        System.out.println("Sending push to: " + notification.getRecipient());
        // Push notification logic
        return true;
    }
}

public class Notification {
    private final String id;
    private final String recipient;
    private final String subject;
    private final String content;
    private final NotificationType type;
    private NotificationStatus status;
    private int retryCount;

    public Notification(String recipient, String subject, String content, NotificationType type) {
        this.id = UUID.randomUUID().toString();
        this.recipient = recipient;
        this.subject = subject;
        this.content = content;
        this.type = type;
        this.status = NotificationStatus.PENDING;
        this.retryCount = 0;
    }
}

public enum NotificationType {
    EMAIL,
    SMS,
    PUSH
}

public enum NotificationStatus {
    PENDING,
    SENT,
    FAILED
}

public class NotificationService {
    private final Map<NotificationType, NotificationChannel> channels;
    private final Queue<Notification> queue;
    private final int maxRetries = 3;

    public NotificationService() {
        this.channels = new HashMap<>();
        this.queue = new ConcurrentLinkedQueue<>();

        channels.put(NotificationType.EMAIL, new EmailChannel());
        channels.put(NotificationType.SMS, new SMSChannel());
        channels.put(NotificationType.PUSH, new PushChannel());

        startWorker();
    }

    public void sendNotification(String recipient, String subject, String content, NotificationType type) {
        Notification notification = new Notification(recipient, subject, content, type);
        queue.offer(notification);
    }

    private void startWorker() {
        Thread worker = new Thread(() -> {
            while (true) {
                Notification notification = queue.poll();

                if (notification != null) {
                    processNotification(notification);
                }

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    break;
                }
            }
        });
        worker.setDaemon(true);
        worker.start();
    }

    private void processNotification(Notification notification) {
        NotificationChannel channel = channels.get(notification.getType());

        try {
            boolean success = channel.send(notification);

            if (success) {
                notification.setStatus(NotificationStatus.SENT);
            } else {
                retry(notification);
            }
        } catch (Exception e) {
            retry(notification);
        }
    }

    private void retry(Notification notification) {
        if (notification.getRetryCount() < maxRetries) {
            notification.incrementRetryCount();
            queue.offer(notification);
        } else {
            notification.setStatus(NotificationStatus.FAILED);
            System.err.println("Failed to send notification after " + maxRetries + " retries");
        }
    }
}
```

---

## Summary

Demonstrates: Multi-channel notifications, strategy pattern, retry logic, asynchronous processing, queue-based delivery.
