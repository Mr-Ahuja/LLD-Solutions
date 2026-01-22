# Spotify / Music Player - Low Level Design

## Problem Statement

Design a music streaming app with playlists, queue management, playback control, search, and recommendations.

---

## Key Classes

```java
public class Song {
    private final String songId;
    private final String title;
    private final List<Artist> artists;
    private final Album album;
    private final int durationSeconds;
    private final String genre;
    private final String audioUrl;
}

public class Artist {
    private final String artistId;
    private final String name;
    private final List<Album> albums;
    private final List<Song> songs;
}

public class Album {
    private final String albumId;
    private final String title;
    private final Artist artist;
    private final List<Song> songs;
    private final int releaseYear;
}

public class Playlist {
    private final String playlistId;
    private final String name;
    private final User owner;
    private final List<Song> songs;
    private boolean isPublic;

    public void addSong(Song song) {
        songs.add(song);
    }

    public void removeSong(Song song) {
        songs.remove(song);
    }
}

public class Player {
    private Queue<Song> queue;
    private Song currentSong;
    private int currentPosition;  // in seconds
    private PlayerState state;
    private boolean shuffle;
    private RepeatMode repeatMode;

    public void play() {
        state = PlayerState.PLAYING;
    }

    public void pause() {
        state = PlayerState.PAUSED;
    }

    public void next() {
        if (!queue.isEmpty()) {
            currentSong = queue.poll();
            currentPosition = 0;
            play();

            // Handle repeat
            if (repeatMode == RepeatMode.ONE) {
                queue.offer(currentSong);
            }
        }
    }

    public void previous() {
        // Go to previous song in history
    }

    public void seek(int seconds) {
        currentPosition = seconds;
    }

    public void addToQueue(Song song) {
        queue.offer(song);
    }

    public void shuffleQueue() {
        List<Song> list = new ArrayList<>(queue);
        Collections.shuffle(list);
        queue = new LinkedList<>(list);
    }
}

public enum PlayerState {
    PLAYING,
    PAUSED,
    STOPPED
}

public enum RepeatMode {
    OFF,
    ALL,
    ONE
}

public class User {
    private final String userId;
    private final String email;
    private Subscription subscription;
    private final List<Playlist> playlists;
    private final List<Song> likedSongs;
    private final Player player;

    public void createPlaylist(String name) {
        Playlist playlist = new Playlist(name, this);
        playlists.add(playlist);
    }

    public void likeSong(Song song) {
        likedSongs.add(song);
    }
}

public class SpotifySystem {
    private final Map<String, Song> songs;
    private final Map<String, Artist> artists;
    private final Map<String, Album> albums;
    private final Map<String, User> users;
    private final Map<String, Playlist> playlists;

    public List<Song> searchSongs(String query) {
        return songs.values().stream()
            .filter(s -> s.getTitle().toLowerCase().contains(query.toLowerCase()) ||
                        s.getArtists().stream().anyMatch(a -> a.getName().toLowerCase().contains(query.toLowerCase())))
            .collect(Collectors.toList());
    }

    public void playPlaylist(String userId, String playlistId) {
        User user = users.get(userId);
        Playlist playlist = playlists.get(playlistId);

        user.getPlayer().getQueue().clear();
        for (Song song : playlist.getSongs()) {
            user.getPlayer().addToQueue(song);
        }

        user.getPlayer().next();  // Start playing first song
    }

    public void playSong(String userId, String songId) {
        User user = users.get(userId);
        Song song = songs.get(songId);

        user.getPlayer().getQueue().clear();
        user.getPlayer().addToQueue(song);
        user.getPlayer().next();
    }

    public List<Song> getRecommendations(String userId) {
        User user = users.get(userId);

        // Simplified: recommend based on liked songs' genres
        Map<String, Long> genreFrequency = user.getLikedSongs().stream()
            .collect(Collectors.groupingBy(Song::getGenre, Collectors.counting()));

        String topGenre = genreFrequency.entrySet().stream()
            .max(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey)
            .orElse(null);

        if (topGenre != null) {
            return songs.values().stream()
                .filter(s -> s.getGenre().equals(topGenre))
                .filter(s -> !user.getLikedSongs().contains(s))
                .limit(20)
                .collect(Collectors.toList());
        }

        return new ArrayList<>();
    }
}
```

---

## Summary

Demonstrates: Playlist management, queue/playback control, shuffle/repeat modes, search, recommendations.
