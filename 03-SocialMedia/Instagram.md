# Instagram - Low Level Design

## Problem Statement

Design Instagram with posts, stories, likes, comments, following, and feed generation.

---

## Key Classes

```java
public class User {
    private final String userId;
    private final String username;
    private final Set<String> followers;
    private final Set<String> following;
    private final List<Post> posts;
    private final List<Story> stories;
}

public class Post {
    private final String postId;
    private final User author;
    private final List<String> imageUrls;
    private final String caption;
    private final List<String> hashtags;
    private final Set<String> likedBy;
    private final List<Comment> comments;
    private final LocalDateTime createdAt;

    public void like(User user) {
        likedBy.add(user.getUserId());
    }

    public void unlike(User user) {
        likedBy.remove(user.getUserId());
    }

    public void addComment(Comment comment) {
        comments.add(comment);
    }

    public int getLikeCount() {
        return likedBy.size();
    }
}

public class Comment {
    private final String commentId;
    private final User author;
    private final String text;
    private final LocalDateTime createdAt;
    private final Set<String> likedBy;
}

public class Story {
    private final String storyId;
    private final User author;
    private final String mediaUrl;
    private final LocalDateTime createdAt;
    private final LocalDateTime expiresAt;

    public Story(User author, String mediaUrl) {
        this.storyId = UUID.randomUUID().toString();
        this.author = author;
        this.mediaUrl = mediaUrl;
        this.createdAt = LocalDateTime.now();
        this.expiresAt = createdAt.plusHours(24);  // 24-hour expiry
    }

    public boolean isExpired() {
        return LocalDateTime.now().isAfter(expiresAt);
    }
}

public class Feed {
    private final User user;

    public List<Post> generateFeed() {
        List<Post> feedPosts = new ArrayList<>();

        // Get posts from followed users
        for (String followedId : user.getFollowing()) {
            User followed = getUserById(followedId);
            feedPosts.addAll(followed.getPosts());
        }

        // Ranking algorithm (simplified)
        feedPosts.sort((p1, p2) -> {
            // Consider recency, engagement, relationship
            int engagementScore1 = p1.getLikeCount() + p1.getComments().size();
            int engagementScore2 = p2.getLikeCount() + p2.getComments().size();

            return Integer.compare(engagementScore2, engagementScore1);
        });

        return feedPosts;
    }
}

public class InstagramSystem {
    private final Map<String, User> users;
    private final Map<String, Post> posts;

    public Post createPost(String userId, List<String> imageUrls, String caption) {
        User user = users.get(userId);

        List<String> hashtags = extractHashtags(caption);
        Post post = new Post(user, imageUrls, caption, hashtags);

        posts.put(post.getPostId(), post);
        user.getPosts().add(post);

        return post;
    }

    public Story createStory(String userId, String mediaUrl) {
        User user = users.get(userId);

        Story story = new Story(user, mediaUrl);
        user.getStories().add(story);

        return story;
    }

    public void likePost(String userId, String postId) {
        User user = users.get(userId);
        Post post = posts.get(postId);

        post.like(user);
    }

    public void commentOnPost(String userId, String postId, String text) {
        User user = users.get(userId);
        Post post = posts.get(postId);

        Comment comment = new Comment(user, text);
        post.addComment(comment);
    }

    public List<Post> getFeed(String userId) {
        User user = users.get(userId);
        Feed feed = new Feed(user);
        return feed.generateFeed();
    }

    public List<Story> getStories(String userId) {
        User user = users.get(userId);

        List<Story> stories = new ArrayList<>();

        // Get stories from followed users
        for (String followedId : user.getFollowing()) {
            User followed = users.get(followedId);
            stories.addAll(followed.getStories().stream()
                .filter(s -> !s.isExpired())
                .collect(Collectors.toList()));
        }

        return stories;
    }

    private List<String> extractHashtags(String caption) {
        List<String> hashtags = new ArrayList<>();
        String[] words = caption.split("\\s+");

        for (String word : words) {
            if (word.startsWith("#")) {
                hashtags.add(word.substring(1));
            }
        }

        return hashtags;
    }
}
```

---

## Summary

Demonstrates: Post/story lifecycle, engagement (likes/comments), feed ranking algorithm, 24-hour story expiry.
