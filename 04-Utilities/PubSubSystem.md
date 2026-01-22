# Pub-Sub System - Low Level Design

## Problem Statement

Design a publish-subscribe messaging system where publishers send messages to topics and subscribers receive them.

---

## Key Implementation

```java
public interface Subscriber {
    void onMessage(Message message);
    String getSubscriberId();
}

public class Message {
    private final String topic;
    private final String content;
    private final LocalDateTime timestamp;

    public Message(String topic, String content) {
        this.topic = topic;
        this.content = content;
        this.timestamp = LocalDateTime.now();
    }
}

public class Topic {
    private final String name;
    private final Set<Subscriber> subscribers;

    public Topic(String name) {
        this.name = name;
        this.subscribers = new CopyOnWriteArraySet<>();
    }

    public void subscribe(Subscriber subscriber) {
        subscribers.add(subscriber);
    }

    public void unsubscribe(Subscriber subscriber) {
        subscribers.remove(subscriber);
    }

    public void publish(Message message) {
        for (Subscriber subscriber : subscribers) {
            try {
                subscriber.onMessage(message);
            } catch (Exception e) {
                System.err.println("Error delivering message to " + subscriber.getSubscriberId());
            }
        }
    }
}

public class PubSubSystem {
    private final Map<String, Topic> topics;

    public PubSubSystem() {
        this.topics = new ConcurrentHashMap<>();
    }

    public void createTopic(String topicName) {
        topics.putIfAbsent(topicName, new Topic(topicName));
    }

    public void subscribe(String topicName, Subscriber subscriber) {
        Topic topic = topics.get(topicName);
        if (topic != null) {
            topic.subscribe(subscriber);
        }
    }

    public void unsubscribe(String topicName, Subscriber subscriber) {
        Topic topic = topics.get(topicName);
        if (topic != null) {
            topic.unsubscribe(subscriber);
        }
    }

    public void publish(String topicName, String content) {
        Topic topic = topics.get(topicName);
        if (topic != null) {
            Message message = new Message(topicName, content);
            topic.publish(message);
        }
    }
}

// Example Subscriber
public class EmailSubscriber implements Subscriber {
    private final String email;

    public EmailSubscriber(String email) {
        this.email = email;
    }

    @Override
    public void onMessage(Message message) {
        System.out.println("Email to " + email + ": " + message.getContent());
    }

    @Override
    public String getSubscriberId() {
        return email;
    }
}
```

---

## Summary

Demonstrates: Observer pattern, topic-based messaging, asynchronous communication, error handling.
