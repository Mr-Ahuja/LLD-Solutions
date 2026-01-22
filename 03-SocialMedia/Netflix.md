# Netflix / Video Streaming Platform - Low Level Design

## Problem Statement

Design a video streaming platform with content catalog, recommendations, watchlist, subscriptions, and viewing history.

---

## Key Classes

```java
public class Content {
    private final String contentId;
    private final String title;
    private final ContentType type;
    private final List<String> genres;
    private final int releaseYear;
    private final double rating;
    private final int durationMinutes;
    private final String videoUrl;
}

public enum ContentType {
    MOVIE,
    TV_SHOW,
    DOCUMENTARY,
    SHORT_FILM
}

public class TVShow extends Content {
    private final List<Season> seasons;
}

public class Season {
    private final int seasonNumber;
    private final List<Episode> episodes;
}

public class Episode {
    private final int episodeNumber;
    private final String title;
    private final int durationMinutes;
    private final String videoUrl;
}

public class User {
    private final String userId;
    private final String email;
    private Subscription subscription;
    private final List<Profile> profiles;
    private final List<Content> watchlist;
}

public class Profile {
    private final String profileId;
    private final String name;
    private final boolean isKids;
    private final List<WatchHistory> watchHistory;
    private final List<Content> favorites;
}

public class WatchHistory {
    private final Content content;
    private int lastWatchedPosition;  // in seconds
    private final LocalDateTime lastWatched;

    public int getProgress() {
        return (lastWatchedPosition * 100) / content.getDurationMinutes() / 60;
    }
}

public class Subscription {
    private final SubscriptionPlan plan;
    private final LocalDate startDate;
    private LocalDate endDate;
    private SubscriptionStatus status;

    public boolean isActive() {
        return status == SubscriptionStatus.ACTIVE &&
               LocalDate.now().isBefore(endDate);
    }
}

public enum SubscriptionPlan {
    BASIC(8.99, 1, "480p"),
    STANDARD(12.99, 2, "1080p"),
    PREMIUM(15.99, 4, "4K");

    private final double monthlyPrice;
    private final int maxScreens;
    private final String quality;

    SubscriptionPlan(double monthlyPrice, int maxScreens, String quality) {
        this.monthlyPrice = monthlyPrice;
        this.maxScreens = maxScreens;
        this.quality = quality;
    }
}

public enum SubscriptionStatus {
    ACTIVE,
    CANCELLED,
    EXPIRED,
    PAUSED
}

public class NetflixSystem {
    private final Map<String, Content> contentCatalog;
    private final Map<String, User> users;
    private final RecommendationEngine recommendationEngine;

    public List<Content> searchContent(String query) {
        return contentCatalog.values().stream()
            .filter(c -> c.getTitle().toLowerCase().contains(query.toLowerCase()) ||
                        c.getGenres().stream().anyMatch(g -> g.toLowerCase().contains(query.toLowerCase())))
            .collect(Collectors.toList());
    }

    public void startWatching(String profileId, String contentId) {
        Profile profile = getProfile(profileId);
        Content content = contentCatalog.get(contentId);

        if (!profile.getUser().getSubscription().isActive()) {
            throw new IllegalStateException("Subscription not active");
        }

        WatchHistory history = new WatchHistory(content);
        profile.getWatchHistory().add(history);
    }

    public void updateProgress(String profileId, String contentId, int position) {
        Profile profile = getProfile(profileId);

        WatchHistory history = profile.getWatchHistory().stream()
            .filter(wh -> wh.getContent().getContentId().equals(contentId))
            .findFirst()
            .orElse(null);

        if (history != null) {
            history.setLastWatchedPosition(position);
        }
    }

    public List<Content> getRecommendations(String profileId) {
        Profile profile = getProfile(profileId);
        return recommendationEngine.recommend(profile);
    }

    public void addToWatchlist(String userId, String contentId) {
        User user = users.get(userId);
        Content content = contentCatalog.get(contentId);

        user.getWatchlist().add(content);
    }
}

public class RecommendationEngine {
    public List<Content> recommend(Profile profile) {
        // Simplified recommendation logic
        List<String> favoriteGenres = profile.getWatchHistory().stream()
            .map(wh -> wh.getContent().getGenres())
            .flatMap(List::stream)
            .collect(Collectors.groupingBy(g -> g, Collectors.counting()))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
            .limit(3)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());

        // Recommend content from favorite genres
        return findContentByGenres(favoriteGenres);
    }
}
```

---

## Summary

Demonstrates: Content catalog, subscription tiers, profiles, watch history, recommendations, progress tracking.
