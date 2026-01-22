# Twitter - Low Level Design

## Problem Statement

Design Twitter with tweeting, following, timeline generation, likes, retweets, and trending topics.

---

## Key Classes

```java
public class User {
    private final String userId;
    private final String username;
    private final Set<String> followers;
    private final Set<String> following;
    private final List<Tweet> tweets;

    public void follow(User other) {
        following.add(other.getUserId());
        other.followers.add(this.userId);
    }

    public void unfollow(User other) {
        following.remove(other.getUserId());
        other.followers.remove(this.userId);
    }
}

public class Tweet {
    private final String tweetId;
    private final String content;
    private final User author;
    private final LocalDateTime createdAt;
    private final Set<String> likedBy;
    private int retweetCount;
    private final List<Tweet> replies;
    private final List<String> hashtags;

    public void like(User user) {
        likedBy.add(user.getUserId());
    }

    public void unlike(User user) {
        likedBy.remove(user.getUserId());
    }

    public void retweet() {
        retweetCount++;
    }

    public void reply(Tweet replyTweet) {
        replies.add(replyTweet);
    }
}

public class Timeline {
    private final User user;
    private final PriorityQueue<Tweet> feed;

    public List<Tweet> generateFeed() {
        List<Tweet> tweets = new ArrayList<>();

        // Get tweets from followed users
        for (String followedId : user.getFollowing()) {
            User followed = getUserById(followedId);
            tweets.addAll(followed.getTweets());
        }

        // Sort by recency
        tweets.sort(Comparator.comparing(Tweet::getCreatedAt).reversed());

        return tweets.stream().limit(50).collect(Collectors.toList());
    }
}

public class TwitterSystem {
    private final Map<String, User> users;
    private final Map<String, Tweet> tweets;
    private final Map<String, Integer> trendingHashtags;

    public Tweet postTweet(String userId, String content) {
        User user = users.get(userId);

        if (content.length() > 280) {
            throw new IllegalArgumentException("Tweet too long");
        }

        List<String> hashtags = extractHashtags(content);
        Tweet tweet = new Tweet(content, user, hashtags);

        tweets.put(tweet.getTweetId(), tweet);
        user.getTweets().add(tweet);

        // Update trending
        for (String hashtag : hashtags) {
            trendingHashtags.merge(hashtag, 1, Integer::sum);
        }

        return tweet;
    }

    public List<Tweet> getTimeline(String userId) {
        User user = users.get(userId);
        Timeline timeline = new Timeline(user);
        return timeline.generateFeed();
    }

    public List<String> getTrendingHashtags(int count) {
        return trendingHashtags.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .limit(count)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }

    private List<String> extractHashtags(String content) {
        List<String> hashtags = new ArrayList<>();
        String[] words = content.split("\\s+");

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

Demonstrates: Follow/unfollow, timeline generation, hashtag extraction, trending topics, social graph.
