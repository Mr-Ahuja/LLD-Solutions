# Stack Overflow / Quora - Low Level Design

## Problem Statement

Design a Q&A platform with questions, answers, comments, voting, reputation system, and tagging.

---

## Key Classes

```java
public class User {
    private final String userId;
    private final String username;
    private int reputation;
    private final List<Badge> badges;

    public void upvote(int points) {
        reputation += points;
    }

    public void downvote(int points) {
        reputation -= points;
    }

    public boolean canPerformAction(Action action) {
        return reputation >= action.getMinReputation();
    }
}

public class Question {
    private final String questionId;
    private final String title;
    private final String content;
    private final User author;
    private final List<String> tags;
    private final List<Answer> answers;
    private final List<Comment> comments;
    private int votes;
    private final LocalDateTime createdAt;
    private boolean isClosed;

    public void addAnswer(Answer answer) {
        answers.add(answer);
    }

    public void vote(int value) {
        votes += value;
        author.upvote(value * 5);  // 5 reputation per upvote
    }
}

public class Answer {
    private final String answerId;
    private final String content;
    private final User author;
    private final Question question;
    private final List<Comment> comments;
    private int votes;
    private boolean isAccepted;

    public void accept() {
        if (!isAccepted) {
            isAccepted = true;
            author.upvote(15);  // 15 reputation for accepted answer
        }
    }

    public void vote(int value) {
        votes += value;
        author.upvote(value * 10);  // 10 reputation per upvote
    }
}

public class Comment {
    private final String commentId;
    private final String content;
    private final User author;
    private final LocalDateTime createdAt;
}

public class Tag {
    private final String name;
    private final String description;
    private int usageCount;

    public void incrementUsage() {
        usageCount++;
    }
}

public enum Action {
    ASK_QUESTION(1),
    ANSWER_QUESTION(1),
    COMMENT(50),
    UPVOTE(15),
    DOWNVOTE(125),
    EDIT_POST(2000),
    CLOSE_QUESTION(3000),
    DELETE_POST(10000);

    private final int minReputation;

    Action(int minReputation) {
        this.minReputation = minReputation;
    }

    public int getMinReputation() {
        return minReputation;
    }
}

public class Badge {
    private final String name;
    private final BadgeType type;
    private final String description;
}

public enum BadgeType {
    GOLD,
    SILVER,
    BRONZE
}

public class StackOverflowSystem {
    private final Map<String, User> users;
    private final Map<String, Question> questions;
    private final Map<String, Tag> tags;

    public Question askQuestion(String userId, String title, String content, List<String> tagNames) {
        User user = users.get(userId);

        if (!user.canPerformAction(Action.ASK_QUESTION)) {
            throw new IllegalStateException("Insufficient reputation");
        }

        List<String> questionTags = new ArrayList<>();
        for (String tagName : tagNames) {
            Tag tag = tags.computeIfAbsent(tagName, Tag::new);
            tag.incrementUsage();
            questionTags.add(tagName);
        }

        Question question = new Question(title, content, user, questionTags);
        questions.put(question.getQuestionId(), question);

        return question;
    }

    public Answer postAnswer(String userId, String questionId, String content) {
        User user = users.get(userId);
        Question question = questions.get(questionId);

        if (question.isClosed()) {
            throw new IllegalStateException("Question is closed");
        }

        Answer answer = new Answer(content, user, question);
        question.addAnswer(answer);

        return answer;
    }

    public void upvote(String userId, String postId, PostType type) {
        User user = users.get(userId);

        if (!user.canPerformAction(Action.UPVOTE)) {
            throw new IllegalStateException("Insufficient reputation");
        }

        if (type == PostType.QUESTION) {
            Question question = questions.get(postId);
            question.vote(1);
        } else if (type == PostType.ANSWER) {
            // Find and upvote answer
        }
    }

    public List<Question> searchByTag(String tag) {
        return questions.values().stream()
            .filter(q -> q.getTags().contains(tag))
            .sorted(Comparator.comparingInt(Question::getVotes).reversed())
            .collect(Collectors.toList());
    }

    public List<Question> searchByKeyword(String keyword) {
        return questions.values().stream()
            .filter(q -> q.getTitle().toLowerCase().contains(keyword.toLowerCase()) ||
                        q.getContent().toLowerCase().contains(keyword.toLowerCase()))
            .collect(Collectors.toList());
    }
}

public enum PostType {
    QUESTION,
    ANSWER,
    COMMENT
}
```

---

## Summary

Demonstrates: Reputation system, voting mechanism, tagging, privilege levels, content hierarchy (question→answer→comment).
