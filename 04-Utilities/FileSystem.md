# File System - Low Level Design

## Problem Statement

Design an in-memory file system with directories, files, and operations like mkdir, ls, addContentToFile.

---

## Key Implementation

```java
public abstract class FileSystemEntity {
    protected String name;
    protected Directory parent;

    public abstract int getSize();
    public abstract boolean isDirectory();
}

public class File extends FileSystemEntity {
    private StringBuilder content;

    public File(String name) {
        this.name = name;
        this.content = new StringBuilder();
    }

    public void addContent(String newContent) {
        content.append(newContent);
    }

    public String getContent() {
        return content.toString();
    }

    @Override
    public int getSize() {
        return content.length();
    }

    @Override
    public boolean isDirectory() {
        return false;
    }
}

public class Directory extends FileSystemEntity {
    private final Map<String, FileSystemEntity> children;

    public Directory(String name) {
        this.name = name;
        this.children = new HashMap<>();
    }

    public void addEntity(FileSystemEntity entity) {
        children.put(entity.name, entity);
        entity.parent = this;
    }

    public FileSystemEntity getEntity(String name) {
        return children.get(name);
    }

    public List<String> list() {
        return new ArrayList<>(children.keySet());
    }

    @Override
    public int getSize() {
        return children.values().stream()
            .mapToInt(FileSystemEntity::getSize)
            .sum();
    }

    @Override
    public boolean isDirectory() {
        return true;
    }
}

public class FileSystem {
    private final Directory root;

    public FileSystem() {
        this.root = new Directory("");
    }

    public List<String> ls(String path) {
        FileSystemEntity entity = navigate(path);

        if (entity.isDirectory()) {
            return ((Directory) entity).list();
        } else {
            return Arrays.asList(entity.name);
        }
    }

    public void mkdir(String path) {
        String[] parts = path.split("/");
        Directory current = root;

        for (String part : parts) {
            if (part.isEmpty()) continue;

            FileSystemEntity entity = current.getEntity(part);

            if (entity == null) {
                Directory newDir = new Directory(part);
                current.addEntity(newDir);
                current = newDir;
            } else if (entity.isDirectory()) {
                current = (Directory) entity;
            } else {
                throw new IllegalArgumentException("Path contains file");
            }
        }
    }

    public void addContentToFile(String filePath, String content) {
        int lastSlash = filePath.lastIndexOf('/');
        String dirPath = filePath.substring(0, lastSlash);
        String fileName = filePath.substring(lastSlash + 1);

        Directory dir = (Directory) navigate(dirPath);
        File file = (File) dir.getEntity(fileName);

        if (file == null) {
            file = new File(fileName);
            dir.addEntity(file);
        }

        file.addContent(content);
    }

    public String readContentFromFile(String filePath) {
        File file = (File) navigate(filePath);
        return file.getContent();
    }

    private FileSystemEntity navigate(String path) {
        if (path.equals("/")) return root;

        String[] parts = path.split("/");
        Directory current = root;

        for (String part : parts) {
            if (part.isEmpty()) continue;

            FileSystemEntity entity = current.getEntity(part);

            if (entity == null) {
                throw new IllegalArgumentException("Path not found: " + path);
            }

            if (entity.isDirectory()) {
                current = (Directory) entity;
            } else {
                return entity;
            }
        }

        return current;
    }
}
```

---

## Summary

Demonstrates: Composite pattern (File/Directory hierarchy), tree navigation, path parsing.
