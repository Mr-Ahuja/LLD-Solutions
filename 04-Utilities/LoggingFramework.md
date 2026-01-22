# Logging Framework - Low Level Design

## Problem Statement

Design a flexible logging framework supporting multiple log levels, outputs (console, file), and formats.

---

## Key Classes

```java
public enum LogLevel {
    DEBUG(1),
    INFO(2),
    WARN(3),
    ERROR(4);

    private final int priority;

    LogLevel(int priority) {
        this.priority = priority;
    }

    public int getPriority() {
        return priority;
    }
}

public class LogMessage {
    private final LogLevel level;
    private final String message;
    private final LocalDateTime timestamp;
    private final String className;

    public LogMessage(LogLevel level, String message, String className) {
        this.level = level;
        this.message = message;
        this.timestamp = LocalDateTime.now();
        this.className = className;
    }
}

public interface LogAppender {
    void append(LogMessage message);
}

public class ConsoleAppender implements LogAppender {
    @Override
    public void append(LogMessage message) {
        System.out.println(formatMessage(message));
    }

    private String formatMessage(LogMessage message) {
        return String.format("[%s] [%s] %s: %s",
            message.getTimestamp(),
            message.getLevel(),
            message.getClassName(),
            message.getMessage());
    }
}

public class FileAppender implements LogAppender {
    private final String filePath;
    private final BufferedWriter writer;

    public FileAppender(String filePath) throws IOException {
        this.filePath = filePath;
        this.writer = new BufferedWriter(new FileWriter(filePath, true));
    }

    @Override
    public synchronized void append(LogMessage message) {
        try {
            writer.write(formatMessage(message));
            writer.newLine();
            writer.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

public class Logger {
    private static Logger instance;
    private LogLevel currentLevel;
    private final List<LogAppender> appenders;

    private Logger() {
        this.currentLevel = LogLevel.INFO;
        this.appenders = new ArrayList<>();
        this.appenders.add(new ConsoleAppender());
    }

    public static Logger getInstance() {
        if (instance == null) {
            synchronized (Logger.class) {
                if (instance == null) {
                    instance = new Logger();
                }
            }
        }
        return instance;
    }

    public void setLevel(LogLevel level) {
        this.currentLevel = level;
    }

    public void addAppender(LogAppender appender) {
        appenders.add(appender);
    }

    public void log(LogLevel level, String message, String className) {
        if (level.getPriority() >= currentLevel.getPriority()) {
            LogMessage logMessage = new LogMessage(level, message, className);
            for (LogAppender appender : appenders) {
                appender.append(logMessage);
            }
        }
    }

    public void debug(String message, String className) {
        log(LogLevel.DEBUG, message, className);
    }

    public void info(String message, String className) {
        log(LogLevel.INFO, message, className);
    }

    public void warn(String message, String className) {
        log(LogLevel.WARN, message, className);
    }

    public void error(String message, String className) {
        log(LogLevel.ERROR, message, className);
    }
}
```

---

## Summary

Demonstrates: Singleton logger, multiple appenders, log levels, filtering, thread-safe file writing.
