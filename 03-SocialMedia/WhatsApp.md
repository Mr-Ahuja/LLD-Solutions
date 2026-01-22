# WhatsApp / Messenger - Low Level Design

## Problem Statement

Design a messaging app with one-on-one chats, group chats, message delivery, read receipts, and media sharing.

---

## Key Classes

```java
public class User {
    private final String userId;
    private final String phoneNumber;
    private final String name;
    private UserStatus status;
    private LocalDateTime lastSeen;
}

public enum UserStatus {
    ONLINE,
    OFFLINE,
    TYPING
}

public class Message {
    private final String messageId;
    private final User sender;
    private final String content;
    private final MessageType type;
    private final LocalDateTime timestamp;
    private MessageStatus status;

    public void markDelivered() {
        this.status = MessageStatus.DELIVERED;
    }

    public void markRead() {
        this.status = MessageStatus.READ;
    }
}

public enum MessageType {
    TEXT,
    IMAGE,
    VIDEO,
    AUDIO,
    DOCUMENT
}

public enum MessageStatus {
    SENT,
    DELIVERED,
    READ
}

public class Chat {
    private final String chatId;
    private final ChatType type;
    private final List<User> participants;
    private final List<Message> messages;

    public void sendMessage(Message message) {
        messages.add(message);
        notifyParticipants(message);
    }

    private void notifyParticipants(Message message) {
        for (User user : participants) {
            if (!user.equals(message.getSender())) {
                user.receiveNotification(message);
            }
        }
    }
}

public enum ChatType {
    PRIVATE,
    GROUP
}

public class GroupChat extends Chat {
    private String groupName;
    private User admin;
    private final List<User> admins;

    public void addMember(User user, User addedBy) {
        if (isAdmin(addedBy)) {
            participants.add(user);
        }
    }

    public void removeMember(User user, User removedBy) {
        if (isAdmin(removedBy) && !user.equals(admin)) {
            participants.remove(user);
        }
    }

    private boolean isAdmin(User user) {
        return admins.contains(user);
    }
}

public class WhatsAppSystem {
    private final Map<String, User> users;
    private final Map<String, Chat> chats;

    public Chat createPrivateChat(String user1Id, String user2Id) {
        User user1 = users.get(user1Id);
        User user2 = users.get(user2Id);

        Chat chat = new Chat(ChatType.PRIVATE, Arrays.asList(user1, user2));
        chats.put(chat.getChatId(), chat);

        return chat;
    }

    public GroupChat createGroupChat(String creatorId, String groupName, List<String> memberIds) {
        User creator = users.get(creatorId);
        List<User> members = memberIds.stream()
            .map(users::get)
            .collect(Collectors.toList());

        members.add(creator);

        GroupChat group = new GroupChat(groupName, creator, members);
        chats.put(group.getChatId(), group);

        return group;
    }

    public void sendMessage(String chatId, String senderId, String content, MessageType type) {
        Chat chat = chats.get(chatId);
        User sender = users.get(senderId);

        Message message = new Message(sender, content, type);
        chat.sendMessage(message);
    }

    public void markMessageAsRead(String messageId, String userId) {
        // Find message and mark as read
        for (Chat chat : chats.values()) {
            for (Message msg : chat.getMessages()) {
                if (msg.getMessageId().equals(messageId)) {
                    msg.markRead();
                    return;
                }
            }
        }
    }
}
```

---

## Summary

Demonstrates: Chat types (private/group), message status tracking, notifications, group administration.
